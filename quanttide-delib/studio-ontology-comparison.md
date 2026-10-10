# qtcloud-delib studio 本体与档案建模对比

读者带着这个问题进来：qtcloud-delib 的 studio 已经有一套决议模型，和档案仓 `quanttide-delib` 刚定的五类型建模（Assembly、Member、AgendaItem、Draft、Resolution）是同一件事的两种写法，还是两件事？

**答案：两边只有「决议＝结果记录、不预设执行」这半句是共识；studio 的本体只建了结果记录一个对象，过程与效力的主体在服务端 provider 里，而档案建模的主体（机构、议程、效力链）在 qtcloud-delib 里整体缺席——真正挡在接合路上的是词表：两边的「议题」指的不是一个东西。**

## studio 建了什么

studio 的本体只有一个对象。`src/studio/lib/models/resolution.dart` 的 `Resolution` 五个字段：`id`（UUID）、`name`（slug，取自文件名）、`title`（概括「决定了什么」）、`content`（决议陈述：依据、表决情况、执行安排等，当前纯文本）、`category`（治理、审计、档案、技术等分类）。类注释写明了出处与原则：「结构从实际议事档案标本中长出，不预设执行字段」「content 当前为纯文本，未来可扩展为结构化内容」。种子数据也来自同一批标本：`docs/dev-guide/seed-data.md` 写着种子＝`data/profile/resolutions/*.json` 里的真实决议。仓定位见 `docs/index.md`：「基于罗伯特议事规则管理议事和决议的 SaaS 平台」。

议题在 studio 里没有模型。`src/studio/lib/screens/topic_list.dart` 是占位页，注释写着「服务端尚未提供议题 API……后续对齐决议领域落地：服务端 internal/topic（GET/POST /topics）+ 客户端 Topic 模型」。

顺带核对了服务端，这句注释已经过时。provider 的 `internal/topic/model.go` 已有完整的 `Topic`：在 id/name/title/content/category 之外还有 `Status`、`ProposerID`（动议人）、`SeconderIDs`（附议人）、`Votes`（`VoteResult`：赞成/反对/弃权）、`ResolutionID`（通过后回填）。状态机是罗伯特议事规则的五流程：`proposed`（动议）→ `seconded`（附议）→ `debated`（辩论）→ `voted`（表决）→ `resolved`（决议）或 `rejected`（否决）；`internal/topic/service_test.go` 里「未辩论直接表决」「未表决直接归档」都判 `ErrBadState`，转移是强制的。`internal/topic/transport.go` 已挂出 `GET/POST /topics`、`/second`、`/debate`、`/vote`、`/close` 全套端点。

## 当前建模建了什么

档案仓 `data/profile/quanttide-delib/index.md` 的概念节与意图节建了五样类型加一条链：**Assembly**（议事机构，有成员、有法定人数、开会走规则）有议程，议程收 **AgendaItem**（编号、摘要、`onAgenda`，可跨机构交换），AgendaItem 下挂 **Draft**（议题编号、版本、提案国、相位 `submitted/onTable/passed/rejected`），Draft 经表决升成 **Resolution**（决议号、议题编号、机构、相位 `pending/certified/published`）。判定条件 `C(r)`＝已认证或已发布，四个动作 `review/deliberate/certify/publish` 各带前置，三条不变量管住「社区决议必过认证、官网发布必已认证、审议中草案的议题必在议程上」。参照系是安理会档案管理：先上议程才碰草案、提案国联署、表决通过发号。

## 相同的半句与一个同构点

**决议＝结果记录、不预设执行。** studio 的类注释与我们的建法从不同方向走到同一条边界：我们的决议相位只有认证与发布两格，执行不进模型；他们的 `content` 才写执行安排，且明说「不预设执行字段」。

**表决是相变，两边写法不同、时刻相同。** provider 的 `voted → resolved` 在通过时回填 `ResolutionID`，我们 `deliberate` 的 post 是「通过 ⇒ 发号产生决议」。都是在表决那一刻把过程封存成记录。

**议题是独立对象。** 他们给它留了页面、服务端包和全套 API；我们给它一等类型。承认是一回事，建出来是什么是另一回事，见下。

拿同一件事走一遍看得最清楚。种子标本里那条「周会实行记名表决制」（`service_test.go` 的建例同名）：在 qtcloud-delib，它是一个 `Topic`，从 `proposed` 走五流程到 `resolved`，`Votes` 记着 2 赞成 1 反对，然后回填 `ResolutionID`，`Resolution` 表里落一条带 `content` 散文的记录。在档案建模里，它是挂在「公司周会议事规则」这条 AgendaItem 下的一份 Draft，`review` 查议程与提案国，`deliberate` 表决通过发号，`certify` 过了 `C(r)` 才算社区决议，`publish` 才上官网——同一条内容，前一边到 `resolved` 就结束，后一边的效力才开始。

## 不同的四块

先说成因。形成这个结果的主要原因在两个来源：qtcloud-delib-studio 当前版本的五流程，是从我们内部的工作流程中抽象出的五个阶段，五阶段收在同一份文档里管理，「Topic」这个词则来自模拟联合国的资料；档案侧的当前版本参照联合国安理会，是维护者从 B 站 UP 主的视频学到的方法。两套方法本就存在很多细微的冲突与差异，下面四块是它们的具体表现，不是任何一侧的实现缺陷。

**一、词表错位，这是最要紧的一条。** provider 的 `Topic` 是「一个动议的完整生命周期」：动议人、附议、辩论、表决全挂在它身上，注释也自称「议题领域：五流程」。我们的 `AgendaItem` 是「议程上的条目」：编号、摘要、在不在议程上，动议那一截在我们的 `Draft` 上。同一个词「议题」，一边是动议的载体，一边是议程的条目。两边不对齐就接，topic API 落地出来的会是一个没有议程、没有交换概念的对象，而我们跨机构议题交换的前提——把条目挂到对方议程上——在它身上无处安放。

**二、过程的格数不同。** 他们五流程含附议与辩论两格，我们四动作里没有：我们的 `deliberate` 只有「在桌才能表决」和「通过或否决」两头，中间怎么辩没建；罗伯特的附议是程序门槛（不附议不能辩），我们只有提案国联署（sponsors），联署是提案力量，不是门槛。这两格我们不是取舍掉了，是没写——安理会同样有正式发言环节。补不补是待拍板的一件。

**三、机构与效力整块缺席，反向也缺一块。** qtcloud-delib 没有 Assembly、没有 org：`ProposerID`/`SeconderIDs` 是账号用户 ID，不是机构派出的代表；没有 `certified/published`，没有 `C(r)`，没有发布闸门，`resolved` 即到头。我们的三个核心需求（三代表大会、跨机构议题交换、联盟决议经创始人认证才成社区决议）在它那里没有挂点。反向看，我们档案侧也少一样真东西：票数。`adopted ⇔ 票数条件` 只出现在设计讨论里，没进 `index.md`，而他们有结构化的 `VoteResult`。

**四、id 的时机不同。** 他们创建即发（UUID＋slug），我们表决通过才发号。分歧的根子是 id 的语义：是对象标识，还是效力编号。

| 维度 | qtcloud-delib（studio/provider） | quanttide-delib 档案建模 |
|:--|:--|:--|
| 核心对象 | Topic（五流程）＋ Resolution（记录） | AgendaItem、Draft、Resolution，另有 Assembly、Member |
| id 时机 | 创建即发（UUID＋slug） | 表决通过才发号 |
| 议程 | 无议程概念，Topic 无处可挂 | Draft→AgendaItem→机构议程，可跨机构交换 |
| 归属 | 无机构；category 是内容标签 | org 必填，三机构之一 |
| 过程 | 动议→附议→辩论→表决→决议/否决 | review→deliberate→certify→publish（缺附议、辩论） |
| 表决 | VoteResult 赞成/反对/弃权，结构化 | 无票数载体（待补） |
| 效力 | resolved 即止 | pending→certified→published，`C(r)` 判社区决议 |
| 参照系 | 内部工作流程的五阶段，词源模拟联合国（文档自称罗伯特议事规则） | 安理会档案管理，多机构的效力与档案 |

## 边界与落地

适用范围：本报告只对到 studio 的 `models/resolution.dart`、`screens/topic_list.dart`、provider 的 `internal/topic/model.go`、`internal/topic/transport.go`、`docs/resolution.md` 与档案仓 `quanttide-delib/index.md`；CLI、部署、页面交互与数据库迁移没有查，provider 其余领域不在内。

待拍板的四件，按先后排：一、「议题」词表先对齐——topic 侧继续叫 Topic 就明写它等于我们的 Draft 加过程，还是改名让出「议题＝议程条目」这层含义；二、id 时机选对象标识还是效力编号；三、附议、辩论、票数三格进不进档案的式子；四、若 qtcloud-delib 要承载档案定义的效力链，org 归属、认证与发布相位由哪一侧补。

不适用：本报告不改规格与章程，不评价两边的实现质量；上面的对照只说明两套模型各自管什么，不说明谁该替代谁。
