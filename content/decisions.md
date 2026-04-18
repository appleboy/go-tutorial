+++
title = '決策'
weight = 2
+++

<!--* toc_depth: 3 *-->
<!--
  本文件譯自 https://google.github.io/styleguide/go/decisions,授權條款為
  Creative Commons Attribution 3.0 (CC-BY-3.0)。原文版權屬於 Google LLC,
  本中文版本為翻譯之衍生著作,僅供學習與內部參考。
-->

# Go 風格決策

<https://google.github.io/styleguide/go/decisions>

[總覽](/) | [指南](/guide/) | [決策](/decisions/) |
[最佳實踐](/best-practices/)

<!--

-->

**注意:** 本文件屬於 Google 內部 [Go 風格](/) 系列文件之一。本份文件具 **[規範性](/#normative),但不具 [典範性](/#canonical)**,屬於[核心風格指南](/guide/)的下級文件。詳細說明請見[總覽](/#about)。

<a id="about"></a>

## 關於

本文件收錄了一系列風格決策,用以統一並提供 Go 可讀性導師所給建議的標準指引、解釋與範例。

本文件**並非詳盡無遺**,且會隨時間擴充。當[核心風格指南](/guide/)與此處內容衝突時,**以風格指南為準**,本文件應據此更新。

完整的 Go 風格文件清單請參考[總覽](https://google.github.io/styleguide/go#about)。

下列章節已從風格決策搬移至指南其他位置:

- **MixedCaps**:見 [guide#mixed-caps](/guide/#mixed-caps)
  <a id="mixed-caps"></a>

- **格式化 (Formatting)**:見 [guide#formatting](/guide/#formatting)
  <a id="formatting"></a>

- **行長度 (Line Length)**:見 [guide#line-length](/guide/#line-length)
  <a id="line-length"></a>

<a id="naming"></a>

## 命名

關於命名的整體性指引,請參考[核心風格指南](/guide/#naming)中的命名章節。下列各節對命名的特定面向作進一步澄清。

<a id="underscores"></a>

### 底線

Go 中的命名一般不應包含底線。此原則有三個例外:

1. 只被生成程式碼匯入的套件名稱可以包含底線。多字組成的套件命名請參考[套件名稱](#package-names)一節。
1. `*_test.go` 檔案中的 Test、Benchmark 與 Example 函式名稱可以包含底線。
1. 與作業系統或 cgo 互通的低階函式庫,可能會重用識別字,如 [`syscall`] 中所示。在多數程式碼庫中這非常罕見。

**注意:** 原始檔的檔名不是 Go 識別字,不必遵循上述慣例,可以包含底線。

[`syscall`]: https://pkg.go.dev/syscall#pkg-constants

<a id="package-names"></a>

### 套件名稱

<a id="TOC-PackageNames"></a>

在 Go 中,套件名稱必須精煉,只能使用小寫字母與數字 (例如 [`k8s`]、[`oauth2`])。多字組成的套件名稱應保持不分割且全部小寫 (例如 [`tabwriter`],而非 `tabWriter`、`TabWriter` 或 `tab_writer`)。

避免選擇容易被常用區域變數名稱[遮蔽][shadowed]的套件名稱。例如 `usercount` 會比 `count` 更好,因為 `count` 是個常用變數名稱。

Go 套件名稱不應有底線。如果你必須匯入名稱含底線的套件 (通常來自生成或第三方程式碼),必須在匯入時改名為適合在 Go 中使用的名稱。

例外是:只被生成程式碼匯入的套件名稱可以包含底線。具體例子包括:

- 對於只測試套件公開 API 的單元測試,使用 `_test` 後綴 (`testing` 套件稱之為「[黑箱測試](https://pkg.go.dev/testing)」)。例如,套件 `linkedlist` 必須把它的黑箱單元測試定義在名為 `linkedlist_test` 的套件中 (而非 `linked_list_test`)

- 對於指定功能或整合測試的套件,使用底線與 `_test` 後綴。例如,linked list 服務的整合測試可以命名為 `linked_list_service_test`

- 為[套件層級的文件範例](https://go.dev/blog/examples)使用 `_test` 後綴

[`tabwriter`]: https://pkg.go.dev/text/tabwriter
[`k8s`]: https://pkg.go.dev/k8s.io/client-go/kubernetes
[`oauth2`]: https://pkg.go.dev/golang.org/x/oauth2
[shadowed]: best-practices.md#shadowing

避免使用 `util`、`utility`、`common`、`helper`、`model`、`testhelper` 之類沒有資訊量的套件名稱,這會誘使套件使用者[在匯入時改名](#import-renaming)。請參考:

- [關於所謂「工具套件」的指引](/best-practices/#util-packages)
- [Go Tip #97: What's in a Name](https://google.github.io/styleguide/go/index.html#gotip)
- [Go Tip #108: The Power of a Good Package Name](https://google.github.io/styleguide/go/index.html#gotip)

當匯入的套件被改名時 (例如 `import foopb "path/to/foo_go_proto"`),其本地名稱必須符合上述規則,因為本地名稱決定了該套件中符號在檔案中如何被引用。如果同一個 import 在多份檔案中被改名,特別是同一個或相鄰的套件,應盡量在所有地方使用相同的本地名稱以維持一致性。

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

延伸閱讀:[Go blog 關於套件命名的文章](https://go.dev/blog/package-names)。

<a id="receiver-names"></a>

### 接收者名稱

<a id="TOC-ReceiverNames"></a>

[接收者][Receiver] 變數名稱必須:

- 短 (通常一到兩個字母長)
- 是型別本身的縮寫
- 對該型別的每一個接收者都一致地使用
- 不要用底線;若不會用到該變數則直接省略名稱

| 過長的名稱                  | 較佳的名稱                |
| --------------------------- | ------------------------- |
| `func (tray Tray)`          | `func (t Tray)`           |
| `func (info *ResearchInfo)` | `func (ri *ResearchInfo)` |
| `func (this *ReportWriter)` | `func (w *ReportWriter)`  |
| `func (self *Scanner)`      | `func (s *Scanner)`       |

[Receiver]: https://golang.org/ref/spec#Method_declarations

<a id="constant-names"></a>

### 常數名稱

常數名稱必須使用 [MixedCaps],與 Go 中所有其他名稱相同。([匯出][Exported]常數以大寫開頭,非匯出常數以小寫開頭。) 即使違反其他語言的慣例,Go 中仍然如此。常數名稱不應只是它的值的衍生,而應說明該值代表什麼意義。

```go
// Good:
const MaxPacketSize = 512

const (
    ExecuteBit = 1 << iota
    WriteBit
    ReadBit
)
```

[MixedCaps]: guide.md#mixed-caps
[Exported]: https://tour.golang.org/basics/3

不要使用非 MixedCaps 的常數命名,也不要使用 `K` 前綴。

```go
// Bad:
const MAX_PACKET_SIZE = 512
const kMaxBufferSize = 1024
const KMaxUsersPergroup = 500
```

依其角色而非其值來為常數命名。如果某個常數除了它的值之外沒有別的角色,那麼根本不需要把它定義為常數。

```go
// Bad:
const Twelve = 12

const (
    UserNameColumn = "username"
    GroupColumn    = "group"
)
```

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="initialisms"></a>

### 縮寫字 (Initialisms)

<a id="TOC-Initialisms"></a>

名稱中作為縮寫或字首縮略 (例如 `URL`、`NATO`) 的字應保持相同大小寫。`URL` 應寫成 `URL` 或 `url` (例如 `urlPony` 或 `URLPony`),不要寫成 `Url`。一般而言,識別字 (例如 `ID`、`DB`) 也應依其在英文文章中的用法來大小寫。

- 名稱中含多個縮寫時 (例如 `XMLAPI` 因為包含 `XML` 與 `API`),同一個縮寫中的每個字母大小寫應一致,但名稱中不同的縮寫不一定要採相同大小寫。
- 名稱中含小寫字母的縮寫 (例如 `DDoS`、`iOS`、`gRPC`) 時,該縮寫應如標準英文中所寫,除非為了[匯出][exportedness]需要改變第一個字母。在這種情況下,整個縮寫應採相同大小寫 (例如 `ddos`、`IOS`、`GRPC`)。

[exportedness]: https://golang.org/ref/spec#Exported_identifiers

<!-- Keep this table narrow. If it must grow wider, replace with a list. -->

| 英文用法 | 範圍       | 正確    | 錯誤                                   |
| -------- | ---------- | ------- | -------------------------------------- |
| XML API  | 匯出       | `XMLAPI`| `XmlApi`、`XMLApi`、`XmlAPI`、`XMLapi` |
| XML API  | 非匯出     | `xmlAPI`| `xmlapi`、`xmlApi`                     |
| iOS      | 匯出       | `IOS`   | `Ios`、`IoS`                           |
| iOS      | 非匯出     | `iOS`   | `ios`                                  |
| gRPC     | 匯出       | `GRPC`  | `Grpc`                                 |
| gRPC     | 非匯出     | `gRPC`  | `grpc`                                 |
| DDoS     | 匯出       | `DDoS`  | `DDOS`、`Ddos`                         |
| DDoS     | 非匯出     | `ddos`  | `dDoS`、`dDOS`                         |
| ID       | 匯出       | `ID`    | `Id`                                   |
| ID       | 非匯出     | `id`    | `iD`                                   |
| DB       | 匯出       | `DB`    | `Db`                                   |
| DB       | 非匯出     | `db`    | `dB`                                   |
| Txn      | 匯出       | `Txn`   | `TXN`                                  |

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="getters"></a>

### Getter

<a id="TOC-Getters"></a>

函式與方法名稱不應使用 `Get` 或 `get` 前綴,除非底層概念真的就用「get」這個詞 (例如 HTTP GET)。優先讓名稱直接以名詞起頭,例如使用 `Counts` 而非 `GetCounts`。

如果該函式涉及執行複雜運算或遠端呼叫,可以用 `Compute`、`Fetch` 等其他動詞取代 `Get`,以提示讀者該呼叫可能會花時間、可能阻塞或失敗。

<!--#include file="/go/g3doc/style/includes/special-name-exception.md"-->

<a id="variable-names"></a>

### 變數名稱

<a id="TOC-VariableNames"></a>

一般經驗法則是:名稱長度應與其 scope 大小成正比,並與其在該 scope 中被使用的次數成反比。在檔案層級宣告的變數可能需要多個字組成,而僅出現在單一內層區塊的變數可以是一個字、甚至一兩個字元,以保持程式碼清晰、避免多餘資訊。

以下是粗略的基準。這些數字並非硬性規則,請依照情境、[清晰性][clarity]、[精煉性][concision]做出判斷。

- 小 scope 通常只執行一兩個小操作,大約 1-7 行。
- 中 scope 通常執行幾個小操作或一個大操作,大約 8-15 行。
- 大 scope 通常執行一個或數個大操作,大約 15-25 行。
- 非常大的 scope 是超過一頁的範圍 (例如超過 25 行)。

[clarity]: guide.md#clarity
[concision]: guide.md#concision

在小 scope 中可能完全清楚的名稱 (例如 `c` 表示 counter),在更大的 scope 中可能就不夠用,需要進一步澄清以便讀者沿著程式碼往下讀時想起它的用途。在含有許多變數、或變數代表類似值或概念的 scope 中,可能需要比 scope 大小所暗示的更長的變數名稱。

概念的特定性也能幫助保持變數名稱簡潔。例如,假設只有一個資料庫,通常只在很小 scope 才用的短變數名稱 `db` 即使在很大 scope 仍可能完全清楚。在這種情況下,雖然依 scope 大小可能會接受 `database` 這個單字,但因為 `db` 是該詞的常見縮寫且鮮有歧義,並非必要。

區域變數的名稱應反映它所包含的內容以及它在當前情境中如何被使用,而非它的值的來源。例如,通常最佳的區域變數名稱不會與來源 struct 或 protocol buffer 的欄位名稱相同。

一般而言:

- 像 `count`、`options` 這類單字命名是好的起點。
- 為消除歧義,可以加入更多字,例如 `userCount` 與 `projectCount`。
- 不要單純為了少打字而省略字母。例如 `Sandbox` 比 `Sbx` 好,尤其是匯出名稱。
- 在多數變數名稱中省略[型別與類型別字眼][types and type-like words]。
  - 對數字而言,`userCount` 比 `numUsers` 或 `usersInt` 好。
  - 對 slice 而言,`users` 比 `userSlice` 好。
  - 如果在 scope 中同時有兩種版本的值,可以加入類似型別的修飾,例如把輸入存在 `ageString` 中、把解析後的值用 `age` 表示。
- 省略從[周圍語境][surrounding context]中已清楚的字。例如,在 `UserCount` 方法的實作中,叫 `userCount` 的區域變數可能是多餘的;`count`、`users`、甚至 `c` 都同樣易讀。

[types and type-like words]: #repetitive-with-type
[surrounding context]: #repetitive-in-context

<a id="v"></a>

#### 單字母變數名稱

單字母變數名稱可以是減少[重複](#repetition)的有用工具,但也可能讓程式碼徒增晦澀。把它的使用限制在「該詞顯而易見、且若把單字母換成完整詞會顯得重複」的情境。

一般而言:

- 對[方法接收者變數][method receiver variable],一個或兩個字母的名稱較佳。
- 對常見型別使用熟悉的變數名稱通常很有幫助:
  - `r` 用於 `io.Reader` 或 `*http.Request`
  - `w` 用於 `io.Writer` 或 `http.ResponseWriter`
- 單字母識別字適合作為整數迴圈變數,特別是索引 (例如 `i`) 與座標 (例如 `x`、`y`)。
- 縮寫在 scope 短時可作為迴圈識別字,例如 `for _, n := range nodes { ... }`。

[method receiver variable]: #receiver-names

<a id="repetition"></a>

### 重複

<!--
Note to future editors:

Do not use the term "stutter" to refer to cases when a name is repetitive.
-->

Go 原始碼應避免不必要的重複。常見來源之一是命名重複,通常包含多餘的字、或重複了上下文或型別。程式碼本身在相同或相似程式碼段重複出現於相鄰位置時,也會顯得不必要地重複。

命名重複可有多種形式,包括:

<a id="repetitive-with-package"></a>

#### 套件 vs. 匯出符號名稱

為匯出符號命名時,套件名稱永遠對外可見,因此兩者之間多餘的資訊應減少或消除。如果一個套件只匯出一種型別,而它的名字與套件本身相同,那麼如有需要,建構函式的標準名稱就是 `New`。

> **範例:** 重複的命名 -> 較佳的命名
>
> - `widget.NewWidget` -> `widget.New`
> - `widget.NewWidgetWithName` -> `widget.NewWithName`
> - `db.LoadFromDatabase` -> `db.Load`
> - `goatteleportutil.CountGoatsTeleported` -> `gtutil.CountGoatsTeleported` 或 `goatteleport.Count`
> - `myteampb.MyTeamMethodRequest` -> `mtpb.MyTeamMethodRequest` 或 `myteampb.MethodRequest`

<a id="repetitive-with-type"></a>

#### 變數名稱 vs. 型別

編譯器永遠知道變數的型別,且多數情況下從變數的使用方式,讀者也能看出它的型別。只有當該值在同一 scope 中以兩種形式出現時,才有必要在變數名稱中標出型別。

| 重複的命名                   | 較佳的命名             |
| ---------------------------- | ---------------------- |
| `var numUsers int`           | `var users int`        |
| `var nameString string`      | `var name string`      |
| `var primaryProject *Project`| `var primary *Project` |

如果該值以多種形式出現,可以用額外字 (例如 `raw` 與 `parsed`) 或底層表達來澄清:

```go
// Good:
limitRaw := r.FormValue("limit")
limit, err := strconv.Atoi(limitRaw)
```

```go
// Good:
limitStr := r.FormValue("limit")
limit, err := strconv.Atoi(limitStr)
```

<a id="repetitive-in-context"></a>

#### 外部語境 vs. 區域命名

把周圍語境的資訊也塞進命名,通常只是製造雜訊而毫無好處。套件名稱、方法名稱、型別名稱、函式名稱、import 路徑,甚至檔名,都能為其中所有名稱提供自動性的限定。

```go
// Bad:
// In package "ads/targeting/revenue/reporting"
type AdsTargetingRevenueReport struct{}

func (p *Project) ProjectName() string
```

```go
// Good:
// In package "ads/targeting/revenue/reporting"
type Report struct{}

func (p *Project) Name() string
```

```go
// Bad:
// In package "sqldb"
type DBConnection struct{}
```

```go
// Good:
// In package "sqldb"
type Connection struct{}
```

```go
// Bad:
// In package "ads/targeting"
func Process(in *pb.FooProto) *Report {
    adsTargetingID := in.GetAdsTargetingID()
}
```

```go
// Good:
// In package "ads/targeting"
func Process(in *pb.FooProto) *Report {
    id := in.GetAdsTargetingID()
}
```

重複是否多餘,通常應從符號的使用者角度評估,而非孤立來看。例如下列程式碼有許多名稱在某些情境下沒問題,但在當前語境下卻是多餘的:

```go
// Bad:
func (db *DB) UserCount() (userCount int, err error) {
    var userCountInt64 int64
    if dbLoadError := db.LoadFromDatabase("count(distinct users)", &userCountInt64); dbLoadError != nil {
        return 0, fmt.Errorf("failed to load user count: %s", dbLoadError)
    }
    userCount = int(userCountInt64)
    return userCount, nil
}
```

可從語境或使用方式推得的命名資訊,通常可以省略:

```go
// Good:
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

關於註解的慣例 (寫什麼、用什麼風格、如何提供可執行範例等),旨在協助公開 API 的文件閱讀體驗。詳情請參考 [Effective Go](http://golang.org/doc/effective_go.html#commentary)。

最佳實踐文件中關於[文件慣例][documentation conventions]一節有更多討論。

**最佳實踐:** 在開發與 code review 期間使用[文件預覽][doc preview],檢查文件與可執行範例是否有用、是否如你期望的方式呈現。

**提示:** Godoc 使用的特殊格式很少;清單與程式碼片段通常應縮排以避免折行。除了縮排外,通常應避免裝飾。

[doc preview]: best-practices.md#documentation-preview
[documentation conventions]: best-practices.md#documentation-conventions

<a id="comment-line-length"></a>

### 註解行長度

Go 中的註解沒有固定的[行長度][line length]。

[line length]: guide.md#line-length

長註解行應折行,以確保在不會自動折行的工具中也能閱讀原始碼。如果不確定要折在哪,80 或 100 欄是常見選擇。但這不是硬性截斷;在某些情況下,把長文字折開反而有害。對折行的特定欄寬不作要求。請力求在同一份檔案內保持[一致](/guide/#consistency)。

更多關於註解的內容,請參考[這篇 Go Blog 的文件文章][post from The Go Blog on documentation]。

[post from The Go Blog on documentation]: https://blog.golang.org/godoc-documenting-go-code

```text
# Good:
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

避免把大量文字塞在單一行的註解,這會讓閱讀體驗變差。

```text
# Bad:
// This is a comment paragraph. While some code editors and viewers will wrap the paragraph for the reader, others will display a very long line that will overflow most windows and require users to scroll horizontally. In addition, even on a screen capable of displaying the entire line, it is easier to read a narrower paragraph than very wide one.
//
// Don't worry too much about the long URL:
// https://supercalifragilisticexpialidocious.example.com:8080/Animalia/Chordata/Mammalia/Rodentia/Geomyoidea/Geomyidae/
```

<a id="doc-comments"></a>

### Doc 註解

<a id="TOC-DocComments"></a>

所有頂層匯出名稱都必須有 doc 註解,行為或意義不直觀的非匯出型別與函式宣告也應有。這些註解應該是[完整的句子][full sentences],並以被描述對象的名稱起頭。冠詞 (「a」「an」「the」) 可放在名稱之前,讓讀起來更自然。

```go
// Good:
// A Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

Doc 註解會出現在 [Godoc](https://pkg.go.dev/),並由 IDE 顯示,因此應該為任何使用該套件的人撰寫。

[full sentences]: #comment-sentences

文件註解套用於緊接其後的符號,如果出現在 struct 中,則套用於該欄位群。

```go
// Good:
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

**最佳實踐:** 即使是非匯出程式碼的 doc 註解,也建議遵循與匯出時相同的慣例 (即註解以該非匯出名稱起頭)。這樣未來要把它匯出時,只需要在註解與程式碼中把非匯出名稱替換成新匯出名稱即可。

<a id="comment-sentences"></a>

### 註解句子

<a id="TOC-CommentSentences"></a>

完整句子的註解應像標準英文句子一樣大寫與標點。(例外:若上下文清楚,以非大寫識別字開頭也是 OK 的。這類用法最好只在段落開頭使用。)

句子片段形式的註解則沒有這種大寫與標點要求。

[文件註解][Documentation comments] 應該永遠是完整句子,因此應一律大寫與標點。簡單的行末註解 (尤其是針對 struct 欄位) 可以是「假設欄位名稱為主詞」的簡短片語。

```go
// Good:
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

### 範例

<a id="TOC-Examples"></a>

套件應清楚記錄其預期用法。儘可能提供[可執行範例][runnable example];範例會顯示在 Godoc 中。可執行範例應放在測試檔,而非正式環境的原始檔。請參考此範例 ([Godoc]、[原始碼][source])。

[runnable example]: http://blog.golang.org/examples
[Godoc]: https://pkg.go.dev/time#example-Duration
[source]: https://cs.opensource.google/go/go/+/HEAD:src/time/example_test.go

如果無法提供可執行範例,可以在註解中放範例程式碼。如同其他在註解中出現的程式碼與命令列片段,它應遵循標準排版慣例。

<a id="named-result-parameters"></a>

### 具名回傳參數

<a id="TOC-NamedResultParameters"></a>

在為參數命名時,考慮函式簽章在 Godoc 中如何呈現。函式名稱本身與回傳參數的型別通常已足夠清楚。

```go
// Good:
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

如果一個函式回傳兩個或以上型別相同的回傳值,加上名稱會很有幫助。

```go
// Good:
func (n *Node) Children() (left, right *Node, err error)
```

如果呼叫端必須對特定回傳參數採取行動,為它們命名能暗示要做什麼:

```go
// Good:
// WithTimeout returns a context that will be canceled no later than d duration
// from now.
//
// The caller must arrange for the returned cancel function to be called when
// the context is no longer needed to prevent a resource leak.
func WithTimeout(parent Context, d time.Duration) (ctx Context, cancel func())
```

上面的程式碼中,取消 (cancel) 是呼叫端必須採取的特定動作。但若回傳參數寫成 `(Context, func())`,「cancel function」是什麼意思就會不清楚。

當具名回傳參數會造成[不必要的重複](#repetitive-with-type)時,不要使用。

```go
// Bad:
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```

不要為了避免在函式內宣告變數,而為回傳參數命名。這種做法會以微小的實作精簡換取不必要的 API 詞彙累贅。

[Naked return][Naked returns] 只在小函式中可被接受。一旦函式變成中等大小,就明確寫出回傳值。同樣地,不要只是為了能用 naked return 而為回傳參數命名。在你的函式中,[清晰性](/guide/#clarity)永遠比省幾行更重要。

如果回傳參數的值必須在 deferred closure 中被改變,為其命名永遠是可接受的。

> **提示:** 在函式簽章中,型別常常比名稱更清楚。[GoTip #38: Functions as Named Types] 示範了這一點。
>
> 在上面的 [`WithTimeout`] 中,真實程式碼用 [`CancelFunc`] 取代回傳參數列中的原始 `func()`,且不需要太多文件就能說明清楚。

[Naked returns]: https://tour.golang.org/basics/7
[GoTip #38: Functions as Named Types]: https://google.github.io/styleguide/go/index.html#gotip
[`WithTimeout`]: https://pkg.go.dev/context#WithTimeout
[`CancelFunc`]: https://pkg.go.dev/context#CancelFunc

<a id="package-comments"></a>

### 套件註解

<a id="TOC-PackageComments"></a>

套件註解必須緊鄰套件子句之上,中間不得有空行。例如:

```go
// Good:
// Package math provides basic constants and mathematical functions.
//
// This package does not guarantee bit-identical results across architectures.
package math
```

每個套件必須有且僅有一個套件註解。如果套件由多份檔案組成,應只有其中一份檔案有套件註解。

`main` 套件的註解形式略有不同,以 BUILD 檔中 `go_binary` 規則的名稱代替套件名稱。

```go
// Good:
// The seed_generator command is a utility that generates a Finch seed file
// from a set of JSON study configs.
package main
```

只要 binary 名稱與 BUILD 檔中所寫完全相同,其他註解風格也可以。當 binary 名稱出現在第一個字時,即使它與命令列實際呼叫的拼法不完全相同,也必須將其大寫。

```go
// Good:
// Binary seed_generator ...
// Command seed_generator ...
// Program seed_generator ...
// The seed_generator command ...
// The seed_generator program ...
// Seed_generator ...
```

提示:

- 命令列範例與 API 用法範例可作為有用的文件。要套用 Godoc 排版,請把含程式碼的註解行縮排。

- 如果沒有明顯的主檔案、或套件註解特別長,可以把 doc 註解放在一個叫 `doc.go` 的檔案中,內容只有註解與套件子句。

- 可使用多行註解取代多個單行註解。當文件中包含一些可能適合從原始檔複製貼上的內容 (例如 binary 的範例命令列、模板範例) 時特別好用。

  ```go
  // Good:
  /*
  The seed_generator command is a utility that generates a Finch seed file
  from a set of JSON study configs.

      seed_generator *.json | base64 > finch-seed.base64
  */
  package template
  ```

- 給維護者看且套用於整份檔案的註解,通常放在 import 宣告之後。這些註解不會出現在 Godoc 中,也不適用於上述套件註解的規則。

<a id="imports"></a>

## 匯入

<a id="TOC-Imports"></a>

<a id="import-renaming"></a>

### 匯入改名

套件匯入通常不該被改名,但有些情況必須改名,或改名能改善可讀性。

匯入套件的本地名稱必須遵循[套件命名指引](#package-names),包括禁用底線與大寫字母。請[一致地](/guide/#consistency)為相同的匯入套件採用相同的本地名稱。

當與其他匯入發生名稱衝突時,匯入套件*必須*改名。(由此延伸出:[好的套件名稱](#package-names)應該不需要改名。) 發生名稱衝突時,優先改名最區域、最專案專屬的匯入。

由 protocol buffer 生成的套件*必須*改名以移除底線,且本地名稱必須以 `pb` 結尾。詳情請參考[proto 與 stub 最佳實踐](/best-practices/#import-protos)。

```go
// Good:
import (
    foosvcpb "path/to/package/foo_service_go_proto"
)
```

最後,被匯入的非自動生成套件,如果名稱沒有資訊量 (例如 `util` 或 `v1`),*可以*改名。請謹慎使用這個方式:如果使用該套件的程式碼周圍語境已能說明上下文,就不要改名。可能時,優先重構該套件本身為一個更合適的名字。

```go
// Good:
import (
    core "github.com/kubernetes/api/core/v1"
    meta "github.com/kubernetes/apimachinery/pkg/apis/meta/v1beta1"
)
```

如果你需要匯入一個名稱與你想用的常見區域變數名稱衝突 (例如 `url`、`ssh`) 的套件,且你希望改名,建議使用 `pkg` 後綴 (例如 `urlpkg`)。注意:雖然可以用區域變數遮蔽套件,但只有當該變數仍在 scope 內時還需要使用該套件,才需要改名。

<a id="import-grouping"></a>

### 匯入分組

匯入應依下列順序分組:

1. 標準函式庫套件

1. 其他 (專案與 vendored) 套件

1. Protocol Buffer 匯入 (例如 `fpb "path/to/foo_go_proto"`)

1. [副作用 (side-effect)](https://go.dev/doc/effective_go#blank_import) 用的匯入 (例如 `_ "path/to/package"`)

```go
// Good:
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

<a id="import-blank"></a>

### 空白匯入 (`import _`)

<a id="TOC-ImportBlank"></a>

只為了副作用而匯入的套件 (使用 `import _ "package"` 語法),只能在 main 套件,或需要它們的測試中匯入。

這類套件的例子包括:

- [time/tzdata](https://pkg.go.dev/time/tzdata)

- 在影像處理程式碼中使用 [image/jpeg](https://pkg.go.dev/image/jpeg)

避免在函式庫套件中使用空白匯入,即使該函式庫間接依賴它們。把副作用匯入限制在 main 套件中,可以協助控制相依,並讓你能寫測試以使用不同的匯入,而不會衝突或浪費建置成本。

下列為此規則的唯一例外:

- 你可以用空白匯入繞過 [nogo 靜態檢查器][nogo static checker] 對禁止匯入的檢查。

- 你可以在使用 `//go:embed` 編譯指示的原始檔中,空白匯入 [embed](https://pkg.go.dev/embed) 套件。

**提示:** 如果你建立的函式庫套件在正式環境中間接依賴某個副作用匯入,請記錄預期用法。

[nogo static checker]: https://github.com/bazelbuild/rules_go/blob/master/go/nogo.rst

<a id="import-dot"></a>

### 點匯入 (`import .`)

<a id="TOC-ImportDot"></a>

`import .` 是個語言特性,可以把另一個套件匯出的識別字直接帶入當前套件,而不需要套件限定。詳情請見[語言規範](https://go.dev/ref/spec#Import_declarations)。

在 Google 程式碼庫中**不要**使用此特性;它會讓人更難看出某個功能是從哪裡來。

```go
// Bad:
package foo_test

import (
    "bar/testutil" // also imports "foo"
    . "foo"
)

var myThing = Bar() // Bar defined in package foo; no qualification needed.
```

```go
// Good:
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

### 回傳錯誤

<a id="TOC-ReturningErrors"></a>

使用 `error` 表示函式可能失敗。依慣例,`error` 是最後一個回傳參數。

```go
// Good:
func Good() error { /* ... */ }
```

回傳 `nil` 錯誤是表達「成功完成一個原本可能失敗的操作」的慣用方式。如果一個函式回傳錯誤,呼叫端必須把所有非錯誤回傳值視為未指定,除非另有明確文件。常見的情況是非錯誤回傳值就是它們的零值,但不能假設總是如此。

```go
// Good:
func GoodLookup() (*Result, error) {
    // ...
    if err != nil {
        return nil, err
    }
    return res, nil
}
```

匯出的、會回傳錯誤的函式,應使用 `error` 型別回傳錯誤。具體錯誤型別容易引發微妙的 bug:具體型別的 `nil` 指標可能被包裝進 interface,從而變成非 nil 的值 (詳見 [Go FAQ 對應條目][nil error])。

```go
// Bad:
func Bad() *os.PathError { /*...*/ }
```

**提示:** 一個接受 [`context.Context`] 引數的函式通常應回傳 `error`,讓呼叫端能判斷在函式執行期間 context 是否被取消。

[nil error]: https://golang.org/doc/faq#nil_error

<a id="error-strings"></a>

### 錯誤字串

<a id="TOC-ErrorStrings"></a>

錯誤字串不應大寫 (除非以匯出名稱、專有名詞或縮寫起頭),也不應以標點結尾。這是因為錯誤字串通常會嵌入其他語境中再印給使用者看。

```go
// Bad:
err := fmt.Errorf("Something bad happened.")
```

```go
// Good:
err := fmt.Errorf("something bad happened")
```

另一方面,完整顯示訊息 (logging、測試失敗、API 回應或其他 UI) 的風格因情境而異,但通常應大寫。

```go
// Good:
log.Infof("Operation aborted: %v", err)
log.Errorf("Operation aborted: %v", err)
t.Errorf("Op(%q) failed unexpectedly; err=%v", args, err)
```

<a id="handle-errors"></a>

### 處理錯誤

<a id="TOC-HandleErrors"></a>

遇到錯誤的程式碼應有意識地決定如何處理它。通常不適合用 `_` 變數丟棄錯誤。如果某個函式回傳錯誤,請執行下列其中之一:

- 立即處理並化解錯誤。
- 把錯誤回傳給呼叫端。
- 在特殊情況下,呼叫 [`log.Fatal`] 或 (絕對必要時) `panic`。

**注意:** `log.Fatalf` 不是標準函式庫的 log。請見 [#logging]。

在罕見情況下,可以忽略或丟棄錯誤 (例如呼叫文件保證不會失敗的 [`(*bytes.Buffer).Write`]),這時應有附註說明為何安全。

```go
// Good:
var b *bytes.Buffer

n, _ := b.Write(p) // never returns a non-nil error
```

關於錯誤處理的更多討論與範例,請見 [Effective Go](http://golang.org/doc/effective_go.html#errors) 與[最佳實踐](/best-practices/#error-handling)。

[`(*bytes.Buffer).Write`]: https://pkg.go.dev/bytes#Buffer.Write

<a id="in-band-errors"></a>

### in-band 錯誤

<a id="TOC-In-Band-Errors"></a>

在 C 與類似語言中,函式常用 -1、null 或空字串等值來表示錯誤或缺漏的結果,這稱為 in-band 錯誤處理。

```go
// Bad:
// Lookup returns the value for key or -1 if there is no mapping for key.
func Lookup(key string) int
```

如果忘了檢查 in-band 錯誤值,可能造成 bug,且把錯誤歸因到錯的函式。

```go
// Bad:
// The following line returns an error that Parse failed for the input value,
// whereas the failure was that there is no mapping for missingKey.
return Parse(Lookup(missingKey))
```

Go 對多回傳值的支援提供了更好的解法 (見 [Effective Go 中關於多回傳值的章節][Effective Go section on multiple returns])。函式不應強迫呼叫端檢查 in-band 錯誤值,而應該回傳一個額外值,表示其他回傳值是否有效。當不需要解釋時,這個回傳值可以是 error 或布林值,且應作為最後一個回傳值。

```go
// Good:
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

這個 API 可以阻止呼叫端錯寫 `Parse(Lookup(key))`,因為 `Lookup(key)` 有兩個輸出,會導致編譯期錯誤。

以這種方式回傳錯誤鼓勵更穩健、更明確的錯誤處理:

```go
// Good:
value, ok := Lookup(key)
if !ok {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

某些標準函式庫函式 (例如 `strings` 套件中的) 會回傳 in-band 錯誤值。這大大簡化了字串處理程式碼,但要求工程師更嚴謹。一般而言,Google 程式碼庫中的 Go 程式碼應為錯誤回傳額外值。

[Effective Go section on multiple returns]: http://golang.org/doc/effective_go.html#multiple-returns

<a id="indent-error-flow"></a>

### 縮排錯誤流程

<a id="TOC-IndentErrorFlow"></a>

在繼續其他程式碼之前先處理錯誤。這能改善可讀性,讓讀者快速找到正常路徑。同樣的邏輯適用於任何「測試一個條件然後以結束性條件 (例如 `return`、`panic`、`log.Fatal`) 收尾」的區塊。

如果結束性條件未成立,應出現在 `if` 區塊之後的程式碼,不應縮排在 `else` 子句中。

```go
// Good:
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

```go
// Bad:
if err != nil {
    // error handling
} else {
    // normal code that looks abnormal due to indentation
}
```

> **提示:** 如果你會在多行程式碼中使用某個變數,通常不值得使用「if 帶初始化」的風格。這種情況下通常更好的做法是把宣告搬到外面,使用標準的 `if` 寫法:
>
> ```go
> // Good:
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
> // Bad:
> if x, err := f(); err != nil {
>   // error handling
>   return
> } else {
>   // lots of code that uses x
>   // across multiple lines
> }
> ```

更多細節請參考 [Go Tip #1: Line of Sight] 與 [TotT: Reduce Code Complexity by Reducing Nesting](https://testing.googleblog.com/2017/06/code-health-reduce-nesting-reduce.html)。

[Go Tip #1: Line of Sight]: https://google.github.io/styleguide/go/index.html#gotip

<a id="language"></a>

## 語言

<a id="literal-formatting"></a>

### 字面量排版

Go 有非常強大的[複合字面量語法][composite literal syntax],可以用單一表達式表示巢狀深、複雜的值。可能時,應該使用此字面量語法而非逐欄位構造值。`gofmt` 對字面量的排版通常不錯,但仍有一些額外規則能讓字面量保持易讀與易維護。

[composite literal syntax]: https://golang.org/ref/spec#Composite_literals

<a id="literal-field-names"></a>

#### 欄位名稱

Struct literal 對於定義在當前套件之外的型別,**必須指定欄位名稱**。

- 對來自其他套件的型別,要包含欄位名稱。

  ```go
  // Good:
  // https://pkg.go.dev/encoding/csv#Reader
  r := csv.Reader{
    Comma: ',',
    Comment: '#',
    FieldsPerRecord: 4,
  }
  ```

  Struct 中欄位的位置與完整欄位集 (兩者在省略欄位名稱時都必須準確) 通常不被視為 struct 公開 API 的一部分;指定欄位名稱可避免不必要的耦合。

  ```go
  // Bad:
  r := csv.Reader{',', '#', 4, false, false, false, false}
  ```

- 對套件內部型別,欄位名稱是可選的。

  ```go
  // Good:
  okay := Type{42}
  also := internalType{4, 2}
  ```

  如果加上欄位名稱會讓程式碼更清楚,仍應使用,實務上也很常這樣做。例如,有大量欄位的 struct 幾乎總是應以欄位名稱初始化。

  <!-- TODO: Maybe a better example here that doesn't have many fields. -->

  ```go
  // Good:
  okay := StructWithLotsOfFields{
    field1: 1,
    field2: "two",
    field3: 3.14,
    field4: true,
  }
  ```

<a id="literal-matching-braces"></a>

#### 大括號對齊

成對大括號的關閉端應與開啟端有相同的縮排。單行字面量必然滿足這項。當字面量跨多行時,維持此特性能讓字面量的大括號對齊與函式、`if` 等常見 Go 結構保持一致。

最常見的錯誤是在多行 struct 字面量中,把關閉大括號放在某個值的同一行。在這些情況下,該行應以逗號結尾,關閉大括號放在下一行。

```go
// Good:
good := []*Type{{Key: "value"}}
```

```go
// Good:
good := []*Type{
    {Key: "multi"},
    {Key: "line"},
}
```

```go
// Bad:
bad := []*Type{
    {Key: "multi"},
    {Key: "line"}}
```

```go
// Bad:
bad := []*Type{
    {
        Key: "value"},
}
```

<a id="literal-cuddled-braces"></a>

#### 緊靠的大括號

在 slice 與 array 字面量中,只有當下列兩項都成立時,才允許移除大括號之間的空白 (即「緊靠」):

- [縮排對齊](#literal-matching-braces)
- 內層的值也是字面量或 proto builder (即不是變數或其他表達式)

```go
// Good:
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
// Good:
good := []*Type{{ // Cuddled correctly
    Field: "value",
}, {
    Field: "value",
}}
```

```go
// Good:
good := []*Type{
    first, // Can't be cuddled
    {Field: "second"},
}
```

```go
// Good:
okay := []*pb.Type{pb.Type_builder{
    Field: "first", // Proto Builders may be cuddled to save vertical space
}.Build(), pb.Type_builder{
    Field: "second",
}.Build()}
```

```go
// Bad:
bad := []*Type{
    first,
    {
        Field: "second",
    }}
```

<a id="literal-repeated-type-names"></a>

#### 重複的型別名稱

在 slice 與 map 字面量中,重複的型別名稱可以省略,以減少雜亂。明確重複型別名稱的合理時機,是在處理你專案中不常見的複雜型別時,或當重複的型別名稱出現在距離很遠的多行上,可以提醒讀者上下文。

```go
// Good:
good := []*Type{
    {A: 42},
    {A: 43},
}
```

```go
// Bad:
repetitive := []*Type{
    &Type{A: 42},
    &Type{A: 43},
}
```

```go
// Good:
good := map[Type1]*Type2{
    {A: 1}: {B: 2},
    {A: 3}: {B: 4},
}
```

```go
// Bad:
repetitive := map[Type1]*Type2{
    Type1{A: 1}: &Type2{B: 2},
    Type1{A: 3}: &Type2{B: 4},
}
```

**提示:** 若想移除 struct literal 中重複的型別名稱,可以執行 `gofmt -s`。

<a id="literal-zero-value-fields"></a>

#### 零值欄位

在不影響清晰度的前提下,可從 struct literal 中省略[零值][Zero-value]欄位。

設計良好的 API 常常透過零值構造提升可讀性。例如,從下列 struct 省略三個零值欄位,可讓注意力集中在唯一被指定的選項上。

[Zero-value]: https://golang.org/ref/spec#The_zero_value

```go
// Bad:
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
// Good:
import (
  "github.com/golang/leveldb"
  "github.com/golang/leveldb/db"
)

ldb := leveldb.Open("/my/table", &db.Options{
    BlockSize: 1<<16,
    ErrorIfDBExists: true,
})
```

table-driven 測試中的 struct 常常受益於[明確的欄位名稱][explicit field names],尤其是測試 struct 不單純時。這允許作者在欄位與當前 test case 無關時完全省略零值欄位。例如,成功的 test case 應省略所有與錯誤或失敗相關的欄位。在零值對於理解 test case 是必要的情況 (例如測試零或 `nil` 輸入) 下,則應指定欄位名稱。

[explicit field names]: #literal-field-names

**精煉**

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

### Nil slice

對多數情境而言,`nil` 與空 slice 在功能上沒有差別。`len`、`cap` 等內建函式對 `nil` slice 的行為符合預期。

```go
// Good:
import "fmt"

var s []int         // nil

fmt.Println(s)      // []
fmt.Println(len(s)) // 0
fmt.Println(cap(s)) // 0
for range s {...}   // no-op

s = append(s, 42)
fmt.Println(s)      // [42]
```

如果你宣告一個空的區域變數 slice (尤其是它可能被回傳),優先使用 nil 初始化,以降低呼叫端發生 bug 的風險。

```go
// Good:
var t []string
```

```go
// Bad:
t := []string{}
```

不要設計需要呼叫端區分 nil 與空 slice 的 API。

```go
// Good:
// Ping pings its targets.
// Returns hosts that successfully responded.
func Ping(hosts []string) ([]string, error) { ... }
```

```go
// Bad:
// Ping pings its targets and returns a list of hosts
// that successfully responded. Can be empty if the input was empty.
// nil signifies that a system error occurred.
func Ping(hosts []string) []string { ... }
```

設計 interface 時,避免區分 `nil` slice 與非 `nil`、長度為 0 的 slice,以免細微的程式錯誤。通常做法是用 `len` 檢查是否為空,而不是用 `== nil`。

下列實作把 `nil` 與長度為 0 的 slice 都視為「空」:

```go
// Good:
// describeInts describes s with the given prefix, unless s is empty.
func describeInts(prefix string, s []int) {
    if len(s) == 0 {
        return
    }
    fmt.Println(prefix, s)
}
```

而不是把這個區分作為 API 的一部分:

```go
// Bad:
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

更多討論請見 [in-band 錯誤][in-band errors]。

[in-band errors]: #in-band-errors

<a id="indentation-confusion"></a>

### 縮排混淆

避免引入會讓行其餘部分與已縮排的程式碼區塊對齊的換行。如果無可避免,留一個空格把區塊內程式碼與折行區隔開來。

```go
// Bad:
if longCondition1 && longCondition2 &&
    // Conditions 3 and 4 have the same indentation as the code within the if.
    longCondition3 && longCondition4 {
    log.Info("all conditions met")
}
```

詳細指引與範例,請見以下章節:

- [函式排版](#func-formatting)
- [條件式與迴圈](#conditional-formatting)
- [字面量排版](#literal-formatting)

<a id="func-formatting"></a>

### 函式排版

函式或方法宣告的簽章應保持在單行內,以避免[縮排混淆](#indentation-confusion)。

函式參數列可能是 Go 原始檔中最長的行之一。但它們之前正好會出現縮排變化,因此很難以「不會讓後續行看起來像函式內文」的方式折行:

```go
// Bad:
func (r *SomeType) SomeLongFunctionName(foo1, foo2, foo3 string,
    foo4, foo5, foo6 int) {
    foo7 := bar(foo1)
    // ...
}
```

對於原本會有許多引數的函式呼叫,參考[最佳實踐](/best-practices/#funcargs)中關於縮短呼叫端的選項。

通常可以透過抽出區域變數來縮短行長:

```go
// Good:
local := helper(some, parameters, here)
good := foo.Call(list, of, parameters, local)
```

同樣地,函式與方法呼叫不應僅因為行長就分隔。

```go
// Good:
good := foo.Call(long, list, of, parameters, all, on, one, line)
```

```go
// Bad:
bad := foo.Call(long, list, of, parameters,
    with, arbitrary, line, breaks)
```

可能時,避免對特定函式引數加入行內註解。改用[選項 struct](/best-practices/#option-structure),或在函式文件中加入更多細節。

```go
// Good:
good := server.New(ctx, server.Options{Port: 42})
```

```go
// Bad:
bad := server.New(
    ctx,
    42, // Port
)
```

如果 API 無法更動,或本地呼叫情況不尋常 (無論呼叫是否過長),只要有助於理解,加入換行永遠是允許的。

```go
// Good:
canvas.RenderHeptagon(fillColor,
    x0, y0, vertexColor0,
    x1, y1, vertexColor1,
    x2, y2, vertexColor2,
    x3, y3, vertexColor3,
    x4, y4, vertexColor4,
    x5, y5, vertexColor5,
    x6, y6, vertexColor6,
)
```

注意上例的折行不是依特定欄寬,而是依頂點座標與顏色的語意分組。

函式內的長字串字面量不應只為行長而拆開。對於這類含長字串的函式,可以在格式字串後加一個換行,然後把引數放在下一行或之後幾行。如何決定折行位置,最好依輸入的語意分組,而非單純依行長。

```go
// Good:
log.Warningf("Database key (%q, %d, %q) incompatible in transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey,
    txCustomer, txOffset, txKey)
```

```go
// Bad:
log.Warningf("Database key (%q, %d, %q) incompatible in"+
    " transaction started by (%q, %d, %q)",
    currentCustomer, currentOffset, currentKey, txCustomer,
    txOffset, txKey)
```

<a id="conditional-formatting"></a>

### 條件式與迴圈

`if` 條件式不應折行;多行 `if` 子句會導致[縮排混淆](#indentation-confusion)。

```go
// Bad:
// The second if statement is aligned with the code within the if block, causing
// indentation confusion.
if db.CurrentStatusIs(db.InTransaction) &&
    db.ValuesEqual(db.TransactionKey(), row.Key()) {
    return db.Errorf(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

如果不需要短路 (short-circuit) 行為,可以直接抽出布林運算元:

```go
// Good:
inTransaction := db.CurrentStatusIs(db.InTransaction)
keysMatch := db.ValuesEqual(db.TransactionKey(), row.Key())
if inTransaction && keysMatch {
    return db.Error(db.TransactionError, "query failed: row (%v): key does not match transaction key", row)
}
```

也可能還有其他可抽出的區域變數,尤其當條件式已重複時:

```go
// Good:
uid := user.GetUniqueUserID()
if db.UserIsAdmin(uid) || db.UserHasPermission(uid, perms.ViewServerConfig) || db.UserHasPermission(uid, perms.CreateGroup) {
    // ...
}
```

```go
// Bad:
if db.UserIsAdmin(user.GetUniqueUserID()) || db.UserHasPermission(user.GetUniqueUserID(), perms.ViewServerConfig) || db.UserHasPermission(user.GetUniqueUserID(), perms.CreateGroup) {
    // ...
}
```

含有 closure 或多行 struct literal 的 `if` 條件式,要確保[大括號對齊](#literal-matching-braces),以避免[縮排混淆](#indentation-confusion)。

```go
// Good:
if err := db.RunInTransaction(func(tx *db.TX) error {
    return tx.Execute(userUpdate, x, y, z)
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

```go
// Good:
if _, err := client.Update(ctx, &upb.UserUpdateRequest{
    ID:   userID,
    User: user,
}); err != nil {
    return fmt.Errorf("user update failed: %s", err)
}
```

同樣地,不要試圖在 `for` 條件中插入人為換行。如果沒有優雅的重構辦法,讓它就維持長行也沒關係:

```go
// Good:
for i, max := 0, collection.Size(); i < max && !collection.HasPendingWriters(); i++ {
    // ...
}
```

但通常其實是有的:

```go
// Good:
for i, max := 0, collection.Size(); i < max; i++ {
    if collection.HasPendingWriters() {
        break
    }
    // ...
}
```

`switch` 與 `case` 語句也應保持在單行。

```go
// Good:
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
// Bad:
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

如果該行過長,把所有 case 縮排並以空白行隔開,以避免[縮排混淆](#indentation-confusion):

```go
// Good:
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

在比較變數與常數的條件式中,把變數放在等號運算子的左側:

```go
// Good:
if result == "foo" {
  // ...
}
```

而不是把常數放在前面這種較不清楚的寫法 (即「[Yoda 風格條件式](https://en.wikipedia.org/wiki/Yoda_conditions)」):

```go
// Bad:
if "foo" == result {
  // ...
}
```

<a id="copying"></a>

### 複製

<a id="TOC-Copying"></a>

為了避免意外的別名 (aliasing) 與類似 bug,從另一個套件複製 struct 時要小心。例如,`sync.Mutex` 等同步物件就不應被複製。

`bytes.Buffer` 型別包含一個 `[]byte` slice,以及作為對小字串的最佳化的小 byte array,該 slice 可能引用此 array。如果你複製了一個 `Buffer`,複本中的 slice 可能與原始的 array 形成別名,導致後續方法呼叫產生意外效果。

一般而言,如果某型別 `T` 的方法是與指標型別 `*T` 關聯,就不要複製其值。

```go
// Bad:
b1 := bytes.Buffer{}
b2 := b1
```

呼叫接收 value receiver 的方法可能隱藏複製。當你設計 API 時,如果 struct 含有不應被複製的欄位,通常應接收與回傳指標型別。

下列是可接受的:

```go
// Good:
type Record struct {
  buf bytes.Buffer
  // other fields omitted
}

func New() *Record {...}

func (r *Record) Process(...) {...}

func Consumer(r *Record) {...}
```

但下列通常是錯的:

```go
// Bad:
type Record struct {
  buf bytes.Buffer
  // other fields omitted
}


func (r Record) Process(...) {...} // Makes a copy of r.buf

func Consumer(r Record) {...} // Makes a copy of r.buf
```

此指引也適用於複製 `sync.Mutex`。

<a id="dont-panic"></a>

### 不要 panic

<a id="TOC-Don-t-Panic"></a>

不要把 `panic` 用於正常的錯誤處理。改為使用 `error` 與多回傳值。請見 [Effective Go 中錯誤的章節][Effective Go section on errors]。

在 `package main` 與初始化程式碼中,對於應終止程式的錯誤 (例如無效設定),考慮使用 [`log.Exit`],因為在多數這類情況下,堆疊追蹤對讀者沒有幫助。注意 [`log.Exit`] 會呼叫 [`os.Exit`],任何 deferred 函式都不會執行。

對於代表「不可能發生」狀況、即程式碼審查與/或測試本應抓出的 bug,函式可以合理回傳錯誤或呼叫 [`log.Fatal`]。

也請參考[何時 panic 是可接受的](/best-practices/#when-to-panic)。

**注意:** `log.Fatalf` 不是標準函式庫的 log。請見 [#logging]。

[Effective Go section on errors]: http://golang.org/doc/effective_go.html#errors
[`os.Exit`]: https://pkg.go.dev/os#Exit

<a id="must-functions"></a>

### Must 函式

在失敗時讓程式停止的 setup helper 函式遵循 `MustXYZ` (或 `mustXYZ`) 命名慣例。一般而言,它們應只在程式啟動初期被呼叫,而不是在使用者輸入這類情境;後者應採用一般 Go 錯誤處理。

這常出現在僅在[套件初始化時][package initialization time]被呼叫以初始化套件層級變數的函式 (例如 [template.Must](https://golang.org/pkg/text/template/#Must) 與 [regexp.MustCompile](https://golang.org/pkg/regexp/#MustCompile))。

[package initialization time]: https://golang.org/ref/spec#Package_initialization

```go
// Good:
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

同樣的慣例可用於只停止當前測試 (使用 `t.Fatal`) 的測試 helper。這類 helper 在建立測試值時很方便,例如在[表格驅動測試](#table-driven-tests)的 struct 欄位中,因為回傳錯誤的函式不能直接賦值給 struct 欄位。

```go
// Good:
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

在這兩種情況下,這個模式的價值在於 helper 可以在「值」的語境中被呼叫。這些 helper 不應在難以保證錯誤會被攔下的地方,或在錯誤應被[檢查](#handle-errors)的語境 (例如多數 request handler) 中被呼叫。對常數輸入,這允許測試輕易地確保 `Must` 的引數格式良好;對非常數輸入,這允許測試驗證錯誤被[正確處理或傳遞](/best-practices/#error-handling)。

當 `Must` 函式用於測試時,通常應[標記為測試 helper](#mark-test-helpers),並在錯誤時呼叫 `t.Fatal` (使用上的更多考量請參考[測試 helper 中的錯誤處理](/best-practices/#test-helper-error-handling))。

當[一般錯誤處理](/best-practices/#error-handling)是可行的 (包含經過一些重構),就不應使用它們:

```go
// Bad:
func Version(o *servicepb.Object) (*version.Version, error) {
    // Return error instead of using Must functions.
    v := version.MustParse(o.GetVersionString())
    return dealiasVersion(v)
}
```

<a id="goroutine-lifetimes"></a>

### Goroutine 生命週期

<a id="TOC-GoroutineLifetimes"></a>

當你建立 goroutine 時,要讓人清楚它們何時或是否會結束。

Goroutine 可能因為阻塞在 channel 的 send 或 receive 而洩漏。即使沒有其他 goroutine 持有 channel 的引用,垃圾回收器也不會結束阻塞在 channel 上的 goroutine。

即使 goroutine 沒有洩漏,在不再需要時讓它們繼續執行,也可能造成其他細微難診斷的問題。對已關閉的 channel 進行 send 會引發 panic。

```go
// Bad:
ch := make(chan int)
ch <- 42
close(ch)
ch <- 13 // panic
```

在「結果不再需要之後」修改仍在使用中的輸入,可能導致資料競態 (data race)。讓 goroutine 任意長時間繼續執行,可能導致記憶體用量無法預測。

並行程式碼應該寫得讓 goroutine 的生命週期顯而易見。通常這代表把同步相關的程式碼限制在單一函式範圍內,並把邏輯抽成[同步函式][synchronous functions]。如果並行性仍不明顯,就要文件說明 goroutine 何時與為何結束。

遵循 context 使用最佳實踐的程式碼,通常能讓這件事變得清楚。慣例上由 [`context.Context`] 管理:

```go
// Good:
func (w *Worker) Run(ctx context.Context) error {
    var wg sync.WaitGroup
    // ...
    for item := range w.q {
        // process returns at latest when the context is cancelled.
        wg.Add(1)
        go func() {
            defer wg.Done()
            process(ctx, item)
        }()
    }
    // ...
    wg.Wait()  // Prevent spawned goroutines from outliving this function.
}
```

上面的變體還可使用原始的訊號 channel (例如 `chan struct{}`)、同步變數、[條件變數][rethinking-slides] 等。重點是讓後續維護者能看出 goroutine 何時結束。

相對地,下列程式碼對 spawn 的 goroutine 何時結束完全不在意:

```go
// Bad:
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

這段程式碼乍看 OK,但有幾個底層問題:

- 程式碼在正式環境中可能有未定義行為,程式可能無法乾淨地結束,即使作業系統最後釋放了資源。

- 由於程式碼的生命週期不確定,難以做有意義的測試。

- 程式碼可能洩漏資源,如前所述。

延伸閱讀:

- [Never start a goroutine without knowing how it will stop][cheney-stop]
- Rethinking Classical Concurrency Patterns:[投影片][rethinking-slides],[影片][rethinking-video]
- [When Go programs end]
- [文件慣例:Context][Documentation Conventions: Contexts]

[synchronous functions]: #synchronous-functions
[cheney-stop]: https://dave.cheney.net/2016/12/22/never-start-a-goroutine-without-knowing-how-it-will-stop
[rethinking-slides]: https://drive.google.com/file/d/1nPdvhB0PutEJzdCq5ms6UI58dp50fcAN/view
[rethinking-video]: https://www.youtube.com/watch?v=5zXAHh5tJqQ
[When Go programs end]: https://changelog.com/gotime/165
[Documentation Conventions: Contexts]: best-practices.md#documentation-conventions-contexts

<a id="interfaces"></a>

### Interface

<a id="TOC-Interfaces"></a>

在[真有需要](/guide/#simplicity)之前,不要建立 interface。聚焦於所需行為,而不是「服務」「儲存庫」這類抽象命名模式。

- 不要為了抽象化或測試,把 RPC 用戶端包成手寫的 interface。改為[使用真實的傳輸層](/best-practices/#use-real-transports) ([測試 RPC][testing RPC])。

- 不要為了測試而定義後門,或匯出 interface 的[測試替身][test double]實作。優先透過真正實作的[公開 API][public API] 進行測試。

設計小的 interface 以便於實作與組合 ([GoTip #78: Minimal Viable Interfaces])。為 interface 提供合適文件,包括其契約、邊界情況與預期錯誤。如果 interface 只用在套件內部,讓它保持非匯出。

interface 的消費者應該定義它 (而非實作它的套件),確保它只包含實際使用的方法。如果 interface 本身就是「產品」(共同協議),生產端套件可以匯出該 interface,以避免重複定義 interface 造成膨脹。

有句箴言:函式應接受 interface 作為引數,但回傳具體型別 ([GoTip #49: Accept Interfaces, Return Concrete Types])。回傳具體型別讓呼叫端能存取該特定實作的所有公開方法與欄位,而不只是預先選定的 interface 中的子集方法。呼叫端仍可把這個具體結果傳給任何接受 interface 的函式。有些情況下,為了封裝 (例如 `error` interface),以及 command、chaining、factory、[策略 (strategy) 模式](https://en.wikipedia.org/wiki/Strategy_pattern)等構造,回傳 interface 是可接受的。

關於 interface 的更深入討論,請見[最佳實踐中的 interface 章節](/best-practices/#interfaces)。

[GoTip #78: Minimal Viable Interfaces]: https://google.github.io/styleguide/go/index.html#gotip
[GoTip #49: Accept Interfaces, Return Concrete Types]: https://google.github.io/styleguide/go/index.html#gotip
[testing RPC]: https://codelabs.developers.google.com/grpc/getting-started-grpc-go#3
[test double]: https://abseil.io/resources/swe-book/html/ch13.html
[public API]: https://abseil.io/resources/swe-book/html/ch12.html#test_via_public_apis

<a id="generics"></a>

### 泛型 (Generics)

當泛型 (正式名稱「[Type Parameters]」) 能滿足你的業務需求時,就允許使用。在許多應用中,使用既有語言特性 (slice、map、interface 等) 的傳統做法效果同樣好,且不會帶來額外複雜度,因此要小心過早使用。請見[最小機制原則](/guide/#least-mechanism)的討論。

當引入使用泛型的匯出 API 時,確保它有合適的文件。強烈建議附上具動機的可執行[範例][examples]。

不要僅因為你正在實作的演算法或資料結構不在乎成員元素的型別,就使用泛型。如果實務上只有一種型別會被實例化,先讓你的程式碼用該型別運作,完全不用泛型。日後再加上多型,比起把已被證明不必要的抽象拿掉,容易得多。

不要用泛型發明領域特定語言 (DSL)。特別是,避免引入可能對讀者帶來顯著負擔的錯誤處理框架。請改採既定的[錯誤處理](#errors)實踐。對測試而言,要特別小心引入 [assertion 函式庫](#assert) 或框架,以免造成較沒幫助的[測試失敗訊息](#useful-test-failures)。

一般而言:

- [先寫程式碼,別先設計型別][Write code, don't design types]。出自 Robert Griesemer 與 Ian Lance Taylor 的 GopherCon 演講。
- 如果你有多個型別共享一個有用的統一 interface,考慮以該 interface 建模解法。可能不需要泛型。
- 否則,與其依賴 `any` 型別與過多的[型別 switch](https://tour.golang.org/methods/16),考慮泛型。

延伸閱讀:

- [Using Generics in Go],Ian Lance Taylor 的演講

- Go 官網的[泛型教學][Generics tutorial]

[Generics tutorial]: https://go.dev/doc/tutorial/generics
[Type Parameters]: https://go.dev/design/43651-type-parameters
[Using Generics in Go]: https://www.youtube.com/watch?v=nr8EpUO9jhw
[Write code, don't design types]: https://www.youtube.com/watch?v=Pa_e9EeCdy8&t=1250s

<a id="pass-values"></a>

### 傳值

<a id="TOC-PassValues"></a>

不要僅為了省下幾個位元組,就把指標當作函式引數傳遞。如果某函式整個只把引數 `x` 當成 `*x` 來讀,該引數就不該是指標。常見例子包括傳遞字串指標 (`*string`) 或 interface 值的指標 (`*io.Reader`)。在這兩種情況下,值本身大小固定,直接傳遞即可。

此建議不適用於大型 struct,或可能變大的小型 struct。特別是 protocol buffer 訊息一般應透過指標處理,而不是值。指標型別滿足 `proto.Message` interface (被 `proto.Marshal`、`protocmp.Transform` 等接受),且 protocol buffer 訊息可能相當大、且常隨時間變得更大。

<a id="receiver-type"></a>

### 接收者型別

<a id="TOC-ReceiverType"></a>

[方法接收者][method receiver] 可以是值或指標,就像普通函式參數一樣。在兩者之間選擇的依據,是該方法應加入哪一個[方法集][method set(s)]。

[method receiver]: https://golang.org/ref/spec#Method_declarations
[method set(s)]: https://golang.org/ref/spec#Method_sets

**正確性勝過速度或簡潔。** 有時你必須使用指標值。其他情況下,對大型型別、或對你還不確定程式碼會如何成長的情況使用指標作為未來性安排;對單純的[傳統資料][plain old data]使用值。

下列清單詳述每種情況:

- 如果接收者是 slice,且方法不會 reslice 或重新配置,使用值而非指標。

  ```go
  // Good:
  type Buffer []byte

  func (b Buffer) Len() int { return len(b) }
  ```

- 如果方法需要修改接收者,接收者必須是指標。

  ```go
  // Good:
  type Counter int

  func (c *Counter) Inc() { *c++ }

  // See https://pkg.go.dev/container/heap.
  type Queue []Item

  func (q *Queue) Push(x Item) { *q = append([]Item{x}, *q...) }
  ```

- 如果接收者是含有[不可安全複製欄位](#copying)的 struct,使用指標接收者。常見例子是 [`sync.Mutex`] 與其他同步型別。

  ```go
  // Good:
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

  **提示:** 在型別的 [Godoc] 中查看是否安全可複製的資訊。

- 如果接收者是「大型」struct 或 array,指標接收者可能更有效率。傳遞 struct 等同於把它的所有欄位或元素作為引數傳遞。如果這對[傳值](#pass-values)而言太大,指標是好的選擇。

- 對於會與其他會修改接收者的函式並行呼叫的方法,如果這些修改不該被你的方法看到,就用值;否則用指標。

- 如果接收者是 struct 或 array,而其中任一元素是指向可能被修改之物的指標,優先使用指標接收者,以讓「可變」意圖對讀者清楚。

  ```go
  // Good:
  type Counter struct {
      m *Metric
  }

  func (c *Counter) Inc() {
      c.m.Add(1)
  }
  ```

- 如果接收者是不需要修改的[內建型別][built-in type] (例如整數或字串),使用值。

  ```go
  // Good:
  type User string

  func (u User) String() { return string(u) }
  ```

- 如果接收者是 map、function 或 channel,使用值而非指標。

  ```go
  // Good:
  // See https://pkg.go.dev/net/http#Header.
  type Header map[string][]string

  func (h Header) Add(key, value string) { /* omitted */ }
  ```

- 如果接收者是「小型」array 或 struct,本質上是一個沒有可變欄位、沒有指標的值型別,通常 value receiver 是對的選擇。

  ```go
  // Good:
  // See https://pkg.go.dev/time#Time.
  type Time struct { /* omitted */ }

  func (t Time) Add(d Duration) Time { /* omitted */ }
  ```

- 拿不定主意時,使用指標接收者。

一般原則是:對某型別的所有方法,儘量全部使用指標方法或全部使用值方法。

**注意:** 關於「傳值或傳指標是否影響效能」有許多誤解。編譯器可以選擇把堆疊上的值改傳指標,也可以選擇複製堆疊上的值,但這些考量不應在多數情境下凌駕於程式碼的可讀性與正確性。當效能真的重要時,在斷言哪種寫法較快之前,以實際的 benchmark 測試兩種寫法是很重要的。

[plain old data]: https://en.wikipedia.org/wiki/Passive_data_structure
[`sync.Mutex`]: https://pkg.go.dev/sync#Mutex
[built-in type]: https://pkg.go.dev/builtin

<a id="switch-break"></a>

### `switch` 與 `break`

<a id="TOC-SwitchBreak"></a>

不要在 `switch` 子句末尾使用沒有目標 label 的 `break` 語句;它們是多餘的。與 C 與 Java 不同,Go 中的 `switch` 子句會自動 break,要達到 C 風格的行為需要 `fallthrough` 語句。如果想說明空子句的目的,用註解而非 `break`。

```go
// Good:
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
// Bad:
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

> **注意:** 如果 `switch` 子句位於 `for` 迴圈中,在 `switch` 中使用 `break` 並不會跳出外層的 `for`。
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
> 要跳出外層迴圈,在 `for` 上加 label:
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

### 同步函式

<a id="TOC-SynchronousFunctions"></a>

同步函式直接回傳結果,並在回傳前完成所有 callback 與 channel 操作。優先採用同步函式,而非非同步函式。

同步函式可以把 goroutine 限制在某次呼叫的範圍內,有助於推理它們的生命週期、避免洩漏與資料競態。同步函式也較容易測試,因為呼叫端可以直接傳入輸入並檢查輸出,而不需要輪詢或同步。

如有必要,呼叫端可以另外開一個 goroutine 來呼叫此函式以加入並行性。但要從呼叫端移除不必要的並行性卻相當困難 (有時甚至不可能)。

延伸閱讀:

- 「Rethinking Classical Concurrency Patterns」,Bryan Mills 演講:[投影片][rethinking-slides]、[影片][rethinking-video]

<a id="type-aliases"></a>

### 型別別名

<a id="TOC-TypeAliases"></a>

要定義新型別,使用「*型別定義 (type definition)*」`type T1 T2`。要在不定義新型別的情況下指向既有型別,使用「[*型別別名 (type alias)*][*type alias*]」`type T1 = T2`。型別別名很罕見;它的主要用途是協助套件遷移到新的原始碼位置。在不需要時不要使用型別別名。

[*type alias*]: http://golang.org/ref/spec#Type_declarations

<a id="use-percent-q"></a>

### 使用 %q

<a id="TOC-UsePercentQ"></a>

Go 的格式化函式 (`fmt.Printf` 等) 有一個 `%q` 動詞,會把字串以雙引號包起來印出。

```go
// Good:
fmt.Printf("value %q looks like English text", someText)
```

優先使用 `%q`,而不是用 `%s` 自行加雙引號:

```go
// Bad:
fmt.Printf("value \"%s\" looks like English text", someText)
// Avoid manually wrapping strings with single-quotes too:
fmt.Printf("value '%s' looks like English text", someText)
```

當輸出對象是人類、且輸入值可能為空或含控制字元時,建議使用 `%q`。空字串若靜悄悄出現很難被注意到,但 `""` 顯眼許多。

<a id="use-any"></a>

### 使用 any

Go 1.18 引入了 `any` 型別,作為 `interface{}` 的[別名][alias]。因為它是別名,`any` 在許多情況下與 `interface{}` 等價,在其他情況下也可以透過明確轉型輕易互換。新程式碼優先使用 `any`。

[alias]: https://go.googlesource.com/proposal/+/master/design/18130-type-alias.md

## 常用函式庫

<a id="flags"></a>

### Flag

<a id="TOC-Flags"></a>

Google 程式碼庫中的 Go 程式使用[標準 `flag` 套件][standard `flag` package]的內部變體。它的介面類似,但與 Google 內部系統互通良好。Go binary 中的 flag 名稱優先以底線分隔字詞,但保存 flag 值的變數應遵循標準 Go 命名風格 ([mixed caps])。具體來說,flag 名稱應為 snake case,變數名稱應為對應的 camel case 寫法。

```go
// Good:
var (
    pollInterval = flag.Duration("poll_interval", time.Minute, "Interval to use for polling.")
)
```

```go
// Bad:
var (
    poll_interval = flag.Int("pollIntervalSeconds", 60, "Interval to use for polling in seconds.")
)
```

Flag 只能定義在 `package main` 或同等位置。

通用套件應透過 Go API 設定,而不是穿透到命令列介面;不要讓匯入函式庫產生「匯出新 flag」的副作用。也就是說,優先使用明確的函式引數或 struct 欄位指派,或在更罕見、需要最嚴格審視的情況下使用匯出的全域變數。在極為罕見、必須打破此規則的情況下,flag 名稱必須清楚顯示是哪個套件在設定它。

如果你的 flag 是全域變數,把它們放在獨立的 `var` 群組中,緊接 imports 區塊之後。

關於建立含子命令的[複雜 CLI][complex CLIs] 的最佳實踐,還有額外討論。

延伸閱讀:

- [Tip of the Week #45: Avoid Flags, Especially in Library Code][totw-45]
- [Go Tip #10: Configuration Structs and Flags](https://google.github.io/styleguide/go/index.html#gotip)
- [Go Tip #80: Dependency Injection Principles](https://google.github.io/styleguide/go/index.html#gotip)

[standard `flag` package]: https://golang.org/pkg/flag/
[mixed caps]: guide.md#mixed-caps
[complex CLIs]: best-practices.md#complex-clis
[totw-45]: https://abseil.io/tips/45

<a id="logging"></a>

### Logging

Google 程式碼庫中的 Go 程式使用標準 [`log`] 套件的變體。它的介面類似,但更強大,且與 Google 內部系統互通良好。此函式庫的開源版本是 [`package glog`],開源 Google 專案可以使用它,但本指南通篇都稱它為 `log`。

**注意:** 對於異常的程式結束,此函式庫使用 `log.Fatal` 帶堆疊追蹤中止,使用 `log.Exit` 不帶堆疊追蹤停止。沒有像標準函式庫那樣的 `log.Panic` 函式。

**提示:** `log.Info(v)` 等同於 `log.Infof("%v", v)`,其他 logging 等級也類似。當沒有格式化需求時,優先使用非 formatting 版本。

延伸閱讀:

- 關於 [logging 錯誤](/best-practices/#error-logging) 與[自訂詳細度](/best-practices/#vlog)的最佳實踐
- 何時及如何用 log 套件[停止程式](/best-practices/#checks-and-panics)

[`log`]: https://pkg.go.dev/log
[`log/slog`]: https://pkg.go.dev/log/slog
[package `glog`]: https://pkg.go.dev/github.com/golang/glog
[`log.Exit`]: https://pkg.go.dev/github.com/golang/glog#Exit
[`log.Fatal`]: https://pkg.go.dev/github.com/golang/glog#Fatal

<a id="contexts"></a>

### Context

<a id="TOC-Contexts"></a>

[`context.Context`] 型別的值會跨 API 與 process 邊界帶著安全憑證、追蹤資訊、deadline、與取消訊號。與 C++ 與 Java 在 Google 程式碼庫中使用 thread-local storage 不同,Go 程式從進來的 RPC 與 HTTP 請求,沿著整條呼叫鏈一直到出去的請求,都明確地傳遞 context。

[`context.Context`]: https://pkg.go.dev/context

當傳給函式或方法時,[`context.Context`] 永遠是第一個參數。

```go
func F(ctx context.Context /* other arguments */) {}
```

例外:

- 在 HTTP handler 中,context 來自 [`req.Context()`](https://pkg.go.dev/net/http#Request.Context)。
- 在串流 RPC 方法中,context 來自 stream。

  使用 gRPC streaming 的程式碼,從生成的 server 型別 (實作 `grpc.ServerStream`) 的 `Context()` 方法取得 context。請見 [gRPC 生成程式碼文件](https://grpc.io/docs/languages/go/generated-code/)。

- 在測試函式 (例如 `TestXXX`、`BenchmarkXXX`、`FuzzXXX`) 中,context 來自 [`(testing.TB).Context()`](https://pkg.go.dev/testing#TB.Context)。

- 在其他進入點函式 (見下方範例) 中,使用 [`context.Background()`]。

  - 在 binary 目標中:`main`
  - 在通用程式碼與函式庫中:`init`

> **注意:** 在呼叫鏈中段的程式碼極少需要用 [`context.Background()`] 自己建立 base context。除非是錯的 context,優先取用呼叫端傳入的 context。
>
> 你可能會遇到伺服器函式庫 (Stubby、gRPC、或 Google Go server framework 中的 HTTP 實作),它們會為每個請求建立全新的 context 物件。這些 context 立即會被填入進入請求的資訊,在傳給請求 handler 時,context 上附加的值已經跨網路邊界從用戶端傳到 server。這些 context 的生命週期被綁到請求:請求結束時 context 被取消。
>
> 除非你正在實作 server framework,函式庫程式碼中不應用 [`context.Background()`] 建立 context。如果有現成 context 可用,優先使用下面提到的 context 分離。如果你認為在進入點函式之外確實需要 [`context.Background()`],在實作之前先在 Google Go 風格 mailing list 討論。

[`context.Context`] 在函式中作為第一個參數的慣例,也適用於測試 helper。

```go
// Good:
func readTestFile(ctx context.Context, t *testing.T, path string) string {}
```

不要在 struct 型別中加入 context 成員。改為在型別的每個需要傳遞 context 的方法上加入 context 參數。唯一的例外是當方法簽章必須與標準函式庫或 Google 不可控的第三方函式庫中的 interface 匹配時。這類情況非常罕見,在實作與可讀性審查之前應在 Google Go 風格 mailing list 討論。

**注意:** Go 1.24 加入了 [`(testing.TB).Context()`] 方法。在測試中,優先使用 [`(testing.TB).Context()`] 而非 [`context.Background()`] 來提供測試所需的初始 [`context.Context`]。從測試函式內呼叫、需要 context 的 helper 函式、環境或測試替身 setup 等,都應明確傳入 context。

[`(testing.TB).Context()`]: https://pkg.go.dev/testing#TB.Context

Google 程式碼庫中,如果必須在父 context 被取消後仍能執行的背景操作,可以使用內部套件做 context 分離。請追蹤 [issue #40221](https://github.com/golang/go/issues/40221) 上對於開源替代方案的討論。

由於 context 不可變,把同一個 context 傳給多個共享相同 deadline、取消訊號、憑證、父 trace 等的呼叫是 OK 的。

延伸閱讀:

- [Contexts and structs]

[`context.Background()`]: https://pkg.go.dev/context/#Background
[Contexts and structs]: https://go.dev/blog/context-and-structs

<a id="custom-contexts"></a>

#### 自訂 context

不要建立自訂 context 型別,也不要在函式簽章中使用 [`context.Context`] 以外的 interface。本規則沒有例外。

想像如果每個團隊都有自訂 context,從套件 `p` 到套件 `q` 的每一次函式呼叫,都必須決定如何把 `p.Context` 轉成 `q.Context`,而且要對所有 `p`/`q` 套件對都這麼做。這對人類而言不切實際且容易出錯,也讓增加 context 參數的自動重構幾乎不可能。

如果你有應用程式資料要傳遞,放在參數裡、放在接收者中、放在全域中,或如果它確實該放在那裡,放在 `Context` 的值中。建立你自己的 context 型別不可接受,因為它會破壞 Go 團隊讓 Go 程式在正式環境中正確運作的能力。

<a id="crypto-rand"></a>

### crypto/rand

<a id="TOC-CryptoRand"></a>

不要用 `math/rand` 套件產生金鑰,即使是用過即丟的也不可以。如果未提供 seed,該產生器是完全可預測的;若以 `time.Nanoseconds()` seed,熵也只有少少幾個 bit。改用 `crypto/rand` 的 Reader,如果需要文字,印成十六進位或 base64。

```go
// Good:
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

**注意:** `log.Fatalf` 不是標準函式庫的 log。請見 [#logging]。

<a id="useful-test-failures"></a>

## 有用的測試失敗訊息

<a id="TOC-UsefulTestFailures"></a>

應該能在不讀測試原始碼的情況下診斷測試失敗。測試應以有幫助的訊息失敗,訊息中要詳述:

- 是什麼造成失敗
- 哪些輸入導致錯誤
- 實際得到的結果
- 預期的結果

達成此目標的具體慣例如下。

<a id="assert"></a>

### Assertion 函式庫

<a id="TOC-Assert"></a>

不要建立「assertion 函式庫」作為測試的 helper。

Assertion 函式庫嘗試把驗證與失敗訊息產生結合在測試中 (儘管同樣的陷阱也可能適用於其他測試 helper)。關於測試 helper 與 assertion 函式庫的更多區別,請見[最佳實踐](/best-practices/#test-functions)。

```go
// Bad:
var obj BlogPost

assert.IsNotNil(t, "obj", obj)
assert.StringEq(t, "obj.Type", obj.Type, "blogPost")
assert.IntEq(t, "obj.Comments", obj.Comments, 2)
assert.StringNotEq(t, "obj.Body", obj.Body, "")
```

Assertion 函式庫往往會提早結束測試 (如果 `assert` 呼叫 `t.Fatalf` 或 `panic`),或省略測試實際上做對了什麼的相關資訊:

```go
// Bad:
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

複雜的 assertion 函式往往無法提供[有用的失敗訊息][useful failure messages],也沒有測試函式中本來就有的脈絡。太多的 assertion 函式與函式庫導致開發體驗碎片化:該用哪個 assertion 函式庫?輸出格式該用什麼風格?碎片化造成不必要的困惑,尤其對函式庫維護者與大型變更作者 (他們要負責修可能的下游破壞) 而言。不要為測試發明領域特定語言,使用 Go 本身。

Assertion 函式庫常常把比較與相等檢查抽出。優先使用 [`cmp`] 與 [`fmt`] 等標準函式庫:

```go
// Good:
var got BlogPost

want := BlogPost{
    Comments: 2,
    Body:     "Hello, world!",
}

if !cmp.Equal(got, want) {
    t.Errorf("Blog post = %v, want = %v", got, want)
}
```

對於更領域特定的比較 helper,優先回傳一個可以用於測試失敗訊息的值或錯誤,而不是接受 `*testing.T` 並呼叫其錯誤回報方法:

```go
// Good:
func postLength(p BlogPost) int { return len(p.Body) }

func TestBlogPost_VeritableRant(t *testing.T) {
    post := BlogPost{Body: "I am Gunnery Sergeant Hartman, your senior drill instructor."}

    if got, want := postLength(post), 60; got != want {
        t.Errorf("Length of post = %v, want %v", got, want)
    }
}
```

**最佳實踐:** 如果 `postLength` 並非 trivial,直接針對它寫測試會比較合理,獨立於使用它的測試之外。

延伸閱讀:

- [相等比較與 diff](#types-of-equality)
- [印出 diff](#print-diffs)
- 關於測試 helper 與 assertion helper 的更多區別,請見[最佳實踐](/best-practices/#test-functions)
- [Go FAQ] 中關於[測試框架][testing frameworks]及其有意缺席的章節

[useful failure messages]: #useful-test-failures
[`fmt`]: https://golang.org/pkg/fmt/
[marking test helpers]: #mark-test-helpers
[Go FAQ]: https://go.dev/doc/faq
[testing frameworks]: https://go.dev/doc/faq#testing_framework

<a id="identify-the-function"></a>

### 標識函式

在多數測試中,失敗訊息應包含失敗函式的名稱,即使從測試函式名稱看起來似乎已經很明顯。具體而言,你的失敗訊息應該是 `YourFunc(%v) = %v, want %v`,而不是只有 `got %v, want %v`。

<a id="identify-the-input"></a>

### 標識輸入

在多數測試中,如果輸入很短,失敗訊息應包含函式輸入。如果輸入的相關屬性不明顯 (例如輸入很大或不透明),你應該為 test case 取一個能描述測試內容的名稱,並把該描述印在錯誤訊息中。

<a id="got-before-want"></a>

### Got 在 want 之前

測試輸出應在印出預期值之前先印出函式實際回傳的值。一個標準的測試輸出格式是 `YourFunc(%v) = %v, want %v`。原本想寫「actual」與「expected」的地方,優先使用「got」與「want」。

對 diff 而言,方向性沒那麼明顯,因此重要的是包含一個圖例幫助解讀失敗訊息。請見[印出 diff 的章節][section on printing diffs]。無論你在失敗訊息中採用哪一種 diff 順序,都應在失敗訊息中明示,因為既有程式碼對順序不一致。

[section on printing diffs]: #print-diffs

<a id="compare-full-structures"></a>

### 整體結構比較

如果你的函式回傳一個 struct (或任何含多個欄位的資料型別,例如 slice、array 與 map),避免寫人工逐欄位比較 struct 的測試。改為構造你預期函式會回傳的資料,直接以[深度比較][deep comparison]比較。

**注意:** 如果你的資料含有與測試意圖無關的欄位,使其模糊,則本指引不適用。

如果你的 struct 需要比較的是近似 (或某種等義語意) 的相等,或它含有無法比較相等的欄位 (例如其中一個欄位是 `io.Reader`),調整 [`cmp.Diff`] 或 [`cmp.Equal`] 比較加上 [`cmpopts`] 選項 (例如 [`cmpopts.IgnoreInterfaces`]) 可能滿足需求 ([範例](https://play.golang.org/p/vrCUNVfxsvF))。

如果你的函式回傳多個值,你不需要先把它們包進一個 struct 才比較。直接逐個比較並印出。

```go
// Good:
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

### 比較穩定的結果

避免比較可能依賴你不擁有套件的輸出穩定性的結果。改為:測試應比較語意相關、穩定且能抗相依變化的資訊。對於回傳格式化字串或序列化位元組的功能,通常不能假設輸出穩定。

例如,[`json.Marshal`] 可以 (且過去曾經) 改變它輸出的具體位元組。對 JSON 字串做字串相等比較的測試,可能在 `json` 套件改變序列化方式時就失敗。較穩健的測試會解析 JSON 字串的內容,並確保它與某個預期的資料結構在語意上等價。

[`json.Marshal`]: https://golang.org/pkg/encoding/json/#Marshal

<a id="keep-going"></a>

### 繼續往下走

測試應在失敗後仍盡可能往下執行,以便在單次執行中印出所有失敗檢查。這樣修測試的工程師就不需要在修完一個 bug 後再重新執行測試找下一個。

優先用 `t.Error` 而非 `t.Fatal` 回報不相符。在比較函式輸出的多個不同屬性時,對每個比較使用 `t.Error`。

```go
// Good:
gotMean, gotVariance, err := MyDistribution(input)
if err != nil {
  t.Fatalf("MyDistribution(%v) returned unexpected error: %v", input, err)
}
if diff := cmp.Diff(wantMean, gotMean); diff != "" {
  t.Errorf("MyDistribution(%v) returned unexpected difference in mean value (-want +got):\n%s", input, diff)
}
if diff := cmp.Diff(wantVariance, gotVariance); diff != "" {
  t.Errorf("MyDistribution(%v) returned unexpected difference in variance value (-want +got):\n%s", input, diff)
}
```

呼叫 `t.Fatal` 主要用於回報意外狀況 (例如錯誤或輸出不符),且後續失敗會無意義甚至誤導調查者的情境。注意下方程式碼如何呼叫 `t.Fatalf` *再* 呼叫 `t.Errorf`:

```go
// Good:
gotEncoded := Encode(input)
if gotEncoded != wantEncoded {
  t.Fatalf("Encode(%q) = %q, want %q", input, gotEncoded, wantEncoded)
  // It doesn't make sense to decode from unexpected encoded input.
}
gotDecoded, err := Decode(gotEncoded)
if err != nil {
  t.Fatalf("Decode(%q) returned unexpected error: %v", gotEncoded, err)
}
if gotDecoded != input {
  t.Errorf("Decode(%q) = %q, want %q", gotEncoded, gotDecoded, input)
}
```

對於 table-driven 測試,考慮使用子測試,並用 `t.Fatal` 取代 `t.Error` 後接 `continue`。請見 [GoTip #25: Subtests: Making Your Tests Lean](https://google.github.io/styleguide/go/index.html#gotip)。

**最佳實踐:** 關於何時應用 `t.Fatal` 的更多討論,請見[最佳實踐](/best-practices/#t-fatal)。

<a id="types-of-equality"></a>

### 相等比較與 diff

`==` 運算子用[語言定義的比較][language-defined comparisons]評估相等性。純量值 (數字、布林等) 依其值比較,但只有部分 struct 與 interface 能以這種方式比較。指標的比較依其是否指向同一變數,而非依所指向值的相等。

[`cmp`] 套件能比較不適合 `==` 處理的更複雜資料結構,例如 slice。使用 [`cmp.Equal`] 進行相等比較,使用 [`cmp.Diff`] 取得物件之間人類可讀的 diff。

```go
// Good:
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

作為通用比較函式庫,`cmp` 可能不知道如何比較某些型別。例如,只有當傳入 [`protocmp.Transform`] 選項時,它才能比較 protocol buffer 訊息。

<!-- The order of want and got here is deliberate. See comment in #print-diffs. -->

```go
// Good:
if diff := cmp.Diff(want, got, protocmp.Transform()); diff != "" {
    t.Errorf("Foo() returned unexpected difference in protobuf messages (-want +got):\n%s", diff)
}
```

雖然 `cmp` 套件不是 Go 標準函式庫的一部分,但它由 Go 團隊維護,應能隨時間提供穩定的相等比較結果。它可由使用者設定,應能滿足多數比較需求。

[language-defined comparisons]: http://golang.org/ref/spec#Comparison_operators
[`cmp`]: https://pkg.go.dev/github.com/google/go-cmp/cmp
[`cmp.Equal`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Equal
[`cmp.Diff`]: https://pkg.go.dev/github.com/google/go-cmp/cmp#Diff
[`protocmp.Transform`]: https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp#Transform

既有程式碼可能使用以下較舊的函式庫,且為了一致性可繼續使用:

- [`pretty`] 產生視覺上美觀的差異報告。但它有意把視覺表達相同的值視為相等。特別是 `pretty` 不會抓出 nil slice 與空 slice 之間的差異、對欄位相同的不同 interface 實作不敏感,且能用巢狀 map 作為與 struct 值比較的基礎。它在產生 diff 之前還會把整個值序列化為字串,因此不適合比較大型值。預設它會比較非匯出欄位,使其對相依套件的實作細節變化敏感。基於此,在 protobuf 訊息上使用 `pretty` 並不合適。

[`pretty`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty

新程式碼優先使用 `cmp`,在實務上可行時也值得把舊程式碼更新為使用 `cmp`。

舊程式碼可能使用標準函式庫的 `reflect.DeepEqual` 函式比較複雜結構。`reflect.DeepEqual` 不應用於相等檢查,因為它對非匯出欄位與其他實作細節的變化敏感。使用 `reflect.DeepEqual` 的程式碼應更新為上述函式庫之一。

**注意:** `cmp` 套件設計用於測試,而非正式環境。基於此,當它懷疑比較被錯誤執行時,可能會 panic,以指引使用者改善測試讓它較不脆弱。鑒於 cmp 容易 panic,它並不適合用於正式環境中的程式碼,因為意外的 panic 可能致命。

<a id="level-of-detail"></a>

### 細節程度

慣例的失敗訊息 `YourFunc(%v) = %v, want %v` 適合大多數 Go 測試。但有些情況可能需要更多或更少細節:

- 執行複雜互動的測試,也應描述這些互動。例如,如果同一個 `YourFunc` 被呼叫多次,標明哪次呼叫失敗。如果系統的額外狀態很重要,把它包含在失敗輸出中 (或至少在 log 中)。
- 如果資料是含大量樣板的複雜 struct,在訊息中只描述重要部分是可接受的,但不要過度遮蔽資料。
- Setup 失敗不需要這麼多細節。如果一個測試 helper 想填入 Spanner 表,但 Spanner 掛了,你可能不需要包含原本要存到資料庫的測試輸入。`t.Fatalf("Setup: Failed to set up test database: %s", err)` 通常已足以解決問題。

**提示:** 在開發期間觸發失敗模式。檢查失敗訊息看起來如何,以及維護者能否有效處理該失敗。

有一些技巧可以清楚重現測試輸入與輸出:

- 印字串資料時,[`%q` 通常很有用](#use-percent-q),它強調該值的重要性,並讓壞值更容易被發現。
- 印 (小型) struct 時,`%+v` 通常比 `%v` 更有用。
- 當較大值的驗證失敗時,[印出 diff](#print-diffs) 可以讓失敗更容易理解。

<a id="print-diffs"></a>

### 印出 diff

如果你的函式會回傳大量輸出,讀失敗訊息的人就難以從中找出差異。與其同時印出回傳值與預期值,印出 diff。

要計算這類值的 diff,優先使用 `cmp.Diff`,尤其是新測試與新程式碼,但也可使用其他工具。請見[相等類型][types of equality]中關於各函式優缺點的指引。

- [`cmp.Diff`]

- [`pretty.Compare`]

可使用 [`diff`] 套件比較多行字串或字串清單,作為其他類型 diff 的構件。

[types of equality]: #types-of-equality
[`diff`]: https://pkg.go.dev/github.com/kylelemons/godebug/diff
[`pretty.Compare`]: https://pkg.go.dev/github.com/kylelemons/godebug/pretty#Compare

在失敗訊息中加入文字說明 diff 的方向。

<!--
The reversed order of want and got in these examples is intentional, as this is
the prevailing order across the Google codebase. The lack of a stance on which
order to use is also intentional, as there is no consensus which is
"most readable."


-->

- 使用 `cmp`、`pretty`、`diff` 套件 (傳入 `(want, got)` 給該函式) 時,類似 `diff (-want +got)` 是好的,因為你加在格式字串的 `-` 與 `+` 會與 diff 行開頭實際出現的 `-` 與 `+` 一致。如果你傳的是 `(got, want)`,正確的圖例就是 `(-got +want)`。

- `messagediff` 套件使用不同的輸出格式,因此在使用它時 (傳入 `(want, got)`) 適合用 `diff (want -> got)`,因為箭頭方向會與「modified」行中的箭頭方向一致。

Diff 會跨多行,所以在印出 diff 之前印一個換行。

<a id="test-error-semantics"></a>

### 測試錯誤語意

當單元測試對特定輸入做字串比較或用 vanilla `cmp` 檢查特定錯誤是否被回傳時,你可能會發現,如果這些錯誤訊息將來改寫,你的測試會變脆弱。由於這可能把單元測試變成變更偵測器 (見 [TotT: Change-Detector Tests Considered Harmful][tott-350]),不要用字串比較來檢查函式回傳的錯誤是哪一種型別。但用字串比較來檢查待測套件回傳的錯誤訊息是否滿足某些屬性 (例如它包含參數名稱) 是可以的。

Go 中的錯誤值通常含有給人看的部分,以及給語意控制流程用的部分。測試應只測試可被穩定觀察的語意資訊,而非為人類除錯而設的顯示資訊,因為後者經常未來會變動。關於以語意意義建構錯誤的指引,請見[最佳實踐關於錯誤的章節](/best-practices/#error-handling)。如果你不能控制的相依回傳的錯誤語意資訊不足,考慮對該套件擁有者開立 bug 改善 API,而不是依賴解析錯誤訊息。

在單元測試中,通常只關心是否發生了錯誤。若是如此,只要在你預期錯誤時測試 error 是否非 nil 就夠了。如果你想測試錯誤是否在語意上與其他錯誤相符,考慮使用 [`errors.Is`] 或加上 [`cmpopts.EquateErrors`] 的 `cmp`。

> **注意:** 如果測試使用 [`cmpopts.EquateErrors`],但其所有 `wantErr` 值都只是 `nil` 或 `cmpopts.AnyError`,那麼使用 `cmp` 就是[不必要的機制](/guide/#least-mechanism)。把 want 欄位改為 `bool` 來簡化程式碼,然後就可以用簡單的 `!=` 比較。
>
> ```go
> // Good:
> err := f(test.input)
> if gotErr := err != nil; gotErr != test.wantErr {
>     t.Errorf("f(%q) = %v, want error presence = %v", test.input, err, test.wantErr)
> }
> ```

也請參考 [GoTip #13: Designing Errors for Checking](https://google.github.io/styleguide/go/index.html#gotip)。

[tott-350]: https://testing.googleblog.com/2015/01/testing-on-toilet-change-detector-tests.html
[`cmpopts.EquateErrors`]: https://pkg.go.dev/github.com/google/go-cmp/cmp/cmpopts#EquateErrors
[`errors.Is`]: https://pkg.go.dev/errors#Is

<a id="test-structure"></a>

## 測試結構

<a id="subtests"></a>

### 子測試

Go 標準測試函式庫提供[定義子測試][define subtests]的功能。這提供 setup 與 cleanup 的彈性、控制並行性,以及測試過濾。子測試可以很有用 (尤其對 table-driven 測試),但並非強制使用。也請參考 [Go blog 關於子測試的文章](https://blog.golang.org/subtests)。

子測試不應依賴其他 case 的執行才能成功或取得初始狀態,因為子測試應能用 `go test -run` flag 或 Bazel [test filter] 表達式單獨執行。

[define subtests]: https://pkg.go.dev/testing#hdr-Subtests_and_Sub_benchmarks
[test filter]: https://bazel.build/docs/user-manual#test-filter

<a id="subtest-names"></a>

#### 子測試命名

為子測試命名時,要讓它在測試輸出中可讀,且對使用測試過濾的人在命令列中也好用。當你用 `t.Run` 建立子測試時,第一個引數作為測試的描述名稱。為確保 log 中的測試結果可讀,選擇 escape 後仍然有用且可讀的子測試名稱。把子測試名稱想成函式識別字,而不是散文描述。

測試 runner 會把空白替換為底線,並 escape 不可列印字元。為了確保 log 與原始碼之間的對應準確,建議避免在子測試名稱中使用這些字元。

如果你的測試資料受惠於較長的描述,考慮把描述放到另一個欄位 (可能用 `t.Log` 印出,或與失敗訊息一起出現)。

子測試可以用 [Go test runner] 或 Bazel [test filter] 的 flag 單獨執行,因此選擇好打字、有描述性的名字。

> **警告:** 在子測試名稱中,斜線特別不友善,因為它們在 [test filter 中有特殊意義][special meaning for test filters]。
>
> > ```sh
> > # Bad:
> > # Assuming TestTime and t.Run("America/New_York", ...)
> > bazel test :mytest --test_filter="Time/New_York"    # Runs nothing!
> > bazel test :mytest --test_filter="Time//New_York"   # Correct, but awkward.
> > ```

要[標識函式輸入][identify the inputs],把它們放在測試的失敗訊息中,那裡不會被測試 runner escape。

```go
// Good:
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

下面是一些要避免的例子:

```go
// Bad:
// Too wordy.
t.Run("check that there is no mention of scratched records or hovercrafts", ...)
// Slashes cause problems on the command line.
t.Run("AM/PM confusion", ...)
```

也請參考 [Go Tip #117: Subtest Names](https://google.github.io/styleguide/go/index.html#gotip)。

[Go test runner]: https://golang.org/cmd/go/#hdr-Testing_flags
[identify the inputs]: #identify-the-input
[special meaning for test filters]: https://blog.golang.org/subtests#:~:text=Perhaps%20a%20bit,match%20any%20tests

<a id="table-driven-tests"></a>

### 表格驅動測試

當許多不同的 test case 可以用相似的測試邏輯來測試時,使用表格驅動測試。

- 當測試函式的實際輸出是否等於預期輸出時。例如 [`fmt.Sprintf` 的眾多測試][tests of `fmt.Sprintf`] 或下方最簡單的範例。
- 當測試函式的輸出是否總是符合一組相同的不變條件時。例如 [`net.Dial` 的測試][tests for `net.Dial`]。

[tests of `fmt.Sprintf`]: https://cs.opensource.google/go/go/+/master:src/fmt/fmt_test.go
[tests for `net.Dial`]: https://cs.opensource.google/go/go/+/master:src/net/dial_test.go;l=318;drc=5b606a9d2b7649532fe25794fa6b99bd24e7697c

下面是表格驅動測試的最小結構。如有需要,可以使用不同名字、加入子測試或 setup/cleanup 函式。永遠把[有用的測試失敗訊息](#useful-test-failures)放在心上。

```go
// Good:
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

**注意:** 上例的失敗訊息符合[標識函式](#identify-the-function)與[標識輸入](#identify-the-input)的指引。不需要[以數字方式標識列](#table-tests-identifying-the-row)。

當部分 test case 需要與其他 case 不同的邏輯時,適合寫多個測試函式,如 [GoTip #50: Disjoint Table Tests] 所述。

當額外的 test case 簡單 (例如基本錯誤檢查) 且不會在表格測試的迴圈內導入有條件的程式流程,把該 case 包含在現有測試中是可接受的,但要小心使用這類邏輯。今天看似簡單,可能慢慢長成難以維護的東西。

例如:

```go
func TestDivide(t *testing.T) {
    tests := []struct {
        dividend, divisor int
        want              int
        wantErr           bool
    }{
        {
            dividend: 4,
            divisor:  2,
            want:     2,
        },
        {
            dividend: 10,
            divisor:  2,
            want:     5,
        },
        {
            dividend: 1,
            divisor:  0,
            wantErr:  true,
        },
    }

    for _, test := range tests {
        got, err := Divide(test.dividend, test.divisor)
        if (err != nil) != test.wantErr {
            t.Errorf("Divide(%d, %d) error = %v, want error presence = %t", test.dividend, test.divisor, err, test.wantErr)
        }

        // In this example, we're only testing the value result when the tested function didn't fail.
        if err != nil {
            continue
        }

        if got != test.want {
            t.Errorf("Divide(%d, %d) = %d, want %d", test.dividend, test.divisor, got, test.want)
        }
    }
}
```

當每筆表格項目根據輸入有特化邏輯時,測試程式碼中的更複雜邏輯 (例如基於測試 setup 條件式差異的複雜錯誤檢查 (常常基於 table 測試輸入參數)) 可能[難以理解](/guide/#maintainability)。如果 test case 邏輯不同但 setup 相同,在單一測試函式內使用一連串[子測試](#subtests)可能更易讀。測試 helper 也可協助簡化測試 setup 以維持測試本體的可讀性。

你可以結合表格驅動測試與多個測試函式。例如,當測試函式的輸出完全等於預期輸出,以及函式對無效輸入回傳非 nil 錯誤時,寫兩個獨立的表格驅動測試函式是最佳做法:一個給正常的非錯誤輸出,一個給錯誤輸出。

[GoTip #50: Disjoint Table Tests]: https://google.github.io/styleguide/go/index.html#gotip

<a id="table-tests-data-driven"></a>

#### 資料驅動的 test case

表格測試列有時會變複雜,列的值決定 test case 內部的條件式行為。重複帶來的額外清晰度,對可讀性而言是必要的。

```go
// Good:
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

下方反例中,注意在 case setup 中要區分每個 test case 用哪一種 `Codex` 是多麼困難。(其中的部分違反了 [TotT: Data Driven Traps!][tott-97] 中的建議。)

```go
// Bad:
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

#### 標識列

不要用測試在表格中的索引值取代命名測試或印出輸入。沒人想看你的測試表格,然後數欄位才知道哪個 test case 失敗了。

```go
// Bad:
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

在你的 test struct 加入測試描述,並在失敗訊息中印出。使用子測試時,你的子測試名稱應有效標識該列。

**重要:** 即使 `t.Run` 對輸出與執行做了 scope,你仍必須[標識輸入][identify the input]。表格測試的列名必須遵守[子測試命名][subtest naming]指引。

[identify the input]: #identify-the-input
[subtest naming]: #subtest-names

<a id="mark-test-helpers"></a>

### 測試 helper

測試 helper 是執行 setup 或 cleanup 工作的函式。在測試 helper 中發生的所有失敗,預期都是「環境」失敗 (而非受測程式碼的失敗) — 例如測試資料庫無法啟動,因為這台機器上沒有更多空閒的 port。

如果你傳入 `*testing.T`,呼叫 [`t.Helper`] 把測試 helper 中的失敗歸因到呼叫該 helper 的位置。這個參數應放在 [context](#contexts) 參數 (若有) 之後、其他參數之前。

```go
// Good:
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

當這個模式會模糊「測試失敗」與「導致失敗的條件」之間的關係時,不要使用。具體而言,關於 [assertion 函式庫](#assert) 的指引仍然適用,[`t.Helper`] 不應用來實作這類函式庫。

**提示:** 關於測試 helper 與 assertion helper 的更多區別,請見[最佳實踐](/best-practices/#test-functions)。

雖然上述提到的是 `*testing.T`,但對 benchmark 與 fuzz helper,大致建議相同。

[`t.Helper`]: https://pkg.go.dev/testing#T.Helper

<a id="test-package"></a>

### 測試套件

<a id="TOC-TestPackage"></a>

<a id="test-same-package"></a>

#### 在同一套件中測試

測試可定義在與被測試程式碼相同的套件中。

要在同一套件中寫測試:

- 把測試放在 `foo_test.go` 檔
- 測試檔使用 `package foo`
- 不要明確 import 被測試的套件

```build
# Good:
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

同一套件中的測試可以存取套件內的非匯出識別字,這可以提升測試覆蓋並讓測試更精煉。但要注意:測試中宣告的所有[範例][examples]不會帶有使用者在自己程式碼中所需的套件名稱。

[`library`]: https://github.com/bazelbuild/rules_go/blob/master/docs/go/core/rules.md#go_library
[examples]: #examples

<a id="test-different-package"></a>

#### 在不同套件中測試

並非總是適合 (甚至可行) 把測試定義在與被測試程式碼相同的套件中。在這些情況下,使用以 `_test` 結尾的套件名稱。這是「[套件名稱](#package-names)無底線」規則的例外。例如:

- 整合測試沒有明顯的歸屬函式庫

  ```go
  // Good:
  package gmailintegration_test

  import "testing"
  ```

- 在同一套件中定義測試會導致循環相依

  ```go
  // Good:
  package fireworks_test

  import (
    "fireworks"
    "fireworkstestutil" // fireworkstestutil also imports fireworks
  )
  ```

<a id="use-package-testing"></a>

### 使用 `testing` 套件

Go 標準函式庫提供 [`testing` 套件][`testing` package]。在 Google 程式碼庫中,這是 Go 程式碼唯一允許的測試框架。具體來說,不允許 [assertion 函式庫](#assert) 與第三方測試框架。

`testing` 套件提供寫好測試所需的最小但完整功能集:

- 頂層測試
- Benchmark
- [可執行範例](https://blog.golang.org/examples)
- 子測試
- Logging
- 失敗與 fatal 失敗

這些功能設計成能與[複合字面量][composite literal]、[if 帶初始化][if-with-initializer]等核心語言特性協同運作,讓測試作者能寫出[清晰、可讀且可維護的測試]。

[`testing` package]: https://pkg.go.dev/testing
[composite literal]: https://go.dev/ref/spec#Composite_literals
[if-with-initializer]: https://go.dev/ref/spec#If_statements
[清晰、可讀且可維護的測試]: guide.md#clarity

<a id="non-decisions"></a>

## 非決策

風格指南無法為所有事項都列出明示性建議,也無法列出所有它「不表態」的事項。話雖如此,以下是一些可讀性社群曾經辯論但未達成共識的事項。

- **以零值初始化區域變數**。`var i int` 與 `i := 0` 等價。也請參考[初始化最佳實踐][initialization best practices]。
- **空複合字面量 vs. `new` 或 `make`**。`&File{}` 與 `new(File)` 等價,`map[string]bool{}` 與 `make(map[string]bool)` 也等價。也請參考[複合宣告最佳實踐][composite declaration best practices]。
- **`cmp.Diff` 呼叫中 got、want 引數順序**。在區域內保持一致,並在失敗訊息中[包含圖例](#print-diffs)。
- **`errors.New` vs `fmt.Errorf` 用於非格式化字串**。`errors.New("foo")` 與 `fmt.Errorf("foo")` 可以互換使用。

如果在特殊情況下這些議題又被提出,可讀性導師可能會給出可選評論,但一般而言,作者可以在當下情境自由選擇喜歡的風格。

當然,如果風格指南未涵蓋的事項需要更多討論,作者歡迎在具體 review 中、或內部訊息板上提出來。

[composite declaration best practices]: https://google.github.io/styleguide/go/best-practices#vardeclcomposite
[initialization best practices]: https://google.github.io/styleguide/go/best-practices#vardeclinitialization

<!--

-->
