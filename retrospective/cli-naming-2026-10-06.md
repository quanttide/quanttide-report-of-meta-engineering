# 命令行命名复盘：从 qtcloud-meta 的一次设计说起

读这类复盘的人只想知道两件事：上次命名错在哪，下次开工照哪几条办。2026-10-06，qtcloud-meta 的命令行设计被判「一二级命令缺乏条理、全局参数本体和范畴乱加」。本复盘从这一个案出发，把同族十个既有 CLI 的命令树与全局参数读码对照，回答那两个问题。**两处病灶同源：一级按什么切从未定过，参数抬不抬全局没有准入——缺的是规则，不是更顺口的名字。**

## 一、个案：qtcloud-meta 的命令表

设计稿实况（`src/cli/docs/api-references/` 三篇）：

```text
qtcloud-meta ingest <输入>           动作，无二级
qtcloud-meta change list|show|apply  对象，二级三个动作
qtcloud-meta category list|show      表名，二级两个动作
qtcloud-meta mapping list            表名，二级一个动作
qtcloud-meta conflict list           表名，二级一个动作
qtcloud-meta analyze --from --to     动作，无二级
```

全局参数六个：`--db`、`--ontology`、`--category`、`--json`、`--dry-run`、`--version`。四个毛病逐条走。

一级混了三种轴。六个一级命令里，`ingest` 与 `analyze` 是动作，`change` 是对象，`category`、`mapping`、`conflict` 是存储表名（对应 `category`、`category_mapping`、`conflict` 三张表）。同一层放三种切法，没有人能用一句话复述「一级是按什么分的」。

深度不对齐。`change` 与 `category` 下面是真动作，`mapping` 与 `conflict` 名下只有 `list` 一个动作——为一张表单开一级，读者得先记住哪些词后面还有一层、哪些词到此为止。

`--ontology` 只被 `ingest`（按本体对齐）与 `change apply`（写回本体文件）消费，其余五条命令收到它无事可做；`--category` 只被 `ingest` 用，`analyze` 走自己的 `--from` / `--to`。这两条在帮助里写着全局，实际是局部参数；同一个「范畴」概念还留了两套写法，读者得自己猜二者关系。

## 二、同族实况：十个既有 CLI 读码对照

| 仓 | 一级怎么切 | 二级动作 | 全局参数 |
| :-- | :-- | :-- | :-- |
| qtcloud-work | 动作叶子 search/catalog/audit/material + 名词域 workflow/order | create/show/list/check/export/import；create/show/list/next/done/journal/delete | root/data/workflows/artifacts/json/out/dry-run/server |
| qtcloud-crowd | 名词域 tasks/partners/settlements | list/review；list/certify；list/add | data |
| qtcloud-connect | 名词域 consensus/notice/mail | create/list/show/update/confirm/deprecate；send/template/log | 无 |
| qtcloud-course | 名词域 course/lesson/scene | blueprint/design/preview，三域同构 | 无 |
| qtcloud-delib | 名词域 resolutions | list/create | server |
| qtcloud-learn | 名词域 learner/completion | create/list/get | base_url |
| qtcloud-code | 动作叶子 review/audit/list-rules + 动作域 contract/refactor/scaffold/reflect | init/list/validate；rename；tests/code；slice/trace/graph/suggest | 无 |
| qtcloud-asset | 动作 run/scan/validate/config/version + 域 oss | oss：list/ls/url | 无 |
| qtcloud-pay | 混：动作 status/reconcile 与名词域 accounts/orders/… 同层 | create/get/… | server/json/quiet/no-color |
| qtcloud-execute | 混：读用名词 lists/tasks，写用动词 add/update/delete，无二级 | 无 | server/json |
| qtcloud-meta（本案） | 三种轴：动作 ingest/analyze、对象 change、表名 category/mapping/conflict | list/show/apply；list/show；list；list；无 | db/ontology/category/json/dry-run/version |

三句结论。一级有两种切法并存，`qtcloud-pay` 与 `qtcloud-execute` 在同一层内混用，十个仓没有一份文档写明该选哪种。有全局参数的六个仓，全局项一律是装载与输出——路径、端点、`json` / `quiet` / `dry-run`，没有一个把领域对象抬成全局，本案是读到的首例；`qtcloud-delib` 反过来把 `--json` 挂在 `list` 子命令上，分歧也只停在输出开关，没到领域对象。层级最整齐的是名词域到动作那一派，二级动词在仓内重复出现，`qtcloud-crowd` 与 `qtcloud-course` 只需背一张动词表。

## 三、短板：五条

1. 一级三轴并存，切法没有定语。证据是个案一级六词三种轴，同族两派并存且两仓层内混用。
2. 表名即命令，命令面暴露存储结构。证据是 `category`、`conflict` 直接取自表名，`mapping` 取自 `category_mapping`。
3. 深度与轴不对齐，一级的语义厚度不一。证据是 `mapping` 与 `conflict` 各挂一个 `list`。
4. 全局参数没有准入标准。证据是 `--ontology` 仅 2/7 条命令消费、`--category` 仅 1/7。
5. 一个概念两套参数名。证据是全局 `--category` 与 `analyze --from/--to` 并存。

## 四、约束：下次设计命令行先过这六条

1. 开工先写一句「一级按____切」，贴进命令树旁边。自查：拿这句话复述整棵命令树，复述不动就重切。
2. 二级只用一张固定动词表（`list` / `show` / `add` / `apply` 一类），新动作先查表再造词；一个一级若只挂一个动作，要么并进上一级，要么补齐动作集。自查：数每个一级下的动作个数，出现 1 的逐个问为什么。
3. 命令名不取自表名与文件名，先问用户嘴里管它叫什么。自查：拿每个命令名反查表名，重合即改。
4. 全局参数的准入是每条命令都消费，且不写会歧义；只被少数命令消费的参数挂到子命令上。自查：画命令乘参数的矩阵，出现空格就降级。
5. 一个概念一套参数名，跨命令同一拼写。自查：把同义参数列成一张表，出现两种写法就合并。
6. 提交前把命令树与参数矩阵贴进设计文档，让读者能用一句话复述规则。自查：找一个人复述，复述不出就是规则还没定。

## 五、边界与落地

数据来源是读码：提取各仓 `#[derive(Subcommand)]` 枚举与 `global = true` 参数，未逐仓实跑 `--help`；个案数据取自本仓 `src/cli/docs/api-references/` 三篇设计稿。牵连点三处：`api-references/index.md` 的命令总览与全局选项、`ontology.md`、`category.md`。

本报告只沉淀约束，不代改这三篇：第四节六条是否落成条文、落在哪个仓（本仓 AGENTS、主仓 CONTRIBUTING 还是各 CLI 仓），以及个案按哪条改，都等拍板；拍板前命令设计不动。本报告是命令行命名这条线在本仓的唯一沉淀物。
