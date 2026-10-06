# 命令行命名复盘：从 qtcloud-meta 的一次设计说起

读这类复盘的人只想知道两件事：上次错在哪，下次开工照哪几条办。2026-10-06，qtcloud-meta 的命令行先被判「一二级命令缺乏条理、全局参数本体和范畴乱加」，按六条约束重写后仍被否，再砍到一个命令，最后连这个也删掉——个案的结局是零个命令。本复盘用一句话回答那两个问题：**命名规则治得了条理，治不了「这条命令该不该存在」；缺的不是更严的规则，是从任务出发的设计和不冒充定稿的表达。** 第三、四节是命名层，那两处病灶确实修好了；第五、六节才是这次真正要留下的东西。

## 一、个案：三版设计，结局是零

三版都在 2026-10-06，原文能在提交历史里复算：

- 第一版（`eab9e48`）七个命令：`ingest`、`change list/show/apply`、`category list/show`、`mapping list`、`conflict list`、`analyze`，六个全局参数。判词是「一二级命令缺乏条理、全局参数本体和范畴乱加」。
- 第二版（`3aa8da4`）按第四节六条重写：四个名词域 `change`、`category`、`mapping`、`conflict`，二级共用动词表，全局参数从六个减到三个。
- 第三版（`919dba9`）只留 `change submit`；第四版（本轮）把它也删了，`src/cli/docs/api-references/` 只剩一篇总览。

砍的顺序本身就是证据：先砍范畴分析那三个只读域（查询不构成理由），再砍 `list`、`show`、`apply`（没有提交它们空转），最后砍掉唯一有输入的那一个——说明从头到尾没有一条命令是被谁需要的。第二版把命名改对了，被否的却是整个命令面，条理不是问题的全部。

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
| qtcloud-meta（本案） | 第一版三种轴：动作、对象、表名 | list/show/apply；list/show；list；list；无 | db/ontology/category/json/dry-run/version |

三句结论。一级有两种切法并存，`qtcloud-pay` 与 `qtcloud-execute` 在同一层内混用，十个仓没有一份文档写明该选哪种。有全局参数的六个仓，全局项一律是装载与输出——路径、端点、`json` / `quiet` / `dry-run`，没有一个把领域对象抬成全局，本案第一版是读到的首例；`qtcloud-delib` 反过来把 `--json` 挂在 `list` 子命令上，分歧也只停在输出开关，没到领域对象。层级最整齐的是名词域到动作那一派，二级动词在仓内重复出现，`qtcloud-crowd` 与 `qtcloud-course` 只需背一张动词表。

## 三、命名层的短板：五条

1. 一级三轴并存，切法没有定语。证据是第一版一级六词三种轴，同族两派并存且两仓层内混用。
2. 表名即命令，命令面暴露存储结构。证据是 `category`、`conflict` 直接取自表名，`mapping` 取自 `category_mapping`。
3. 深度与轴不对齐，一级的语义厚度不一。证据是 `mapping` 与 `conflict` 各挂一个 `list`。
4. 全局参数没有准入标准。证据是 `--ontology` 仅 2/7 条命令消费、`--category` 仅 1/7。
5. 一个概念两套参数名。证据是全局 `--category` 与 `analyze --from/--to` 并存。

第二版已按这五条逐条修掉，命令面照样被整体否掉——所以还有下面两节。

## 四、下次设计命令行先过这六条

1. 开工先写一句「一级按____切」，贴进命令树旁边。自查：拿这句话复述整棵命令树，复述不动就重切。
2. 二级只用一张固定动词表（`list` / `show` / `add` / `apply` 一类），新动作先查表再造词；一个一级若只挂一个动作，要么并进上一级，要么补齐动作集。自查：数每个一级下的动作个数，出现 1 的逐个问为什么。
3. 命令名不取自表名与文件名，先问用户嘴里管它叫什么。自查：拿每个命令名反查表名，重合即改。
4. 全局参数的准入是每条命令都消费，且不写会歧义；只被少数命令消费的参数挂到子命令上。自查：画命令乘参数的矩阵，出现空格就降级。
5. 一个概念一套参数名，跨命令同一拼写。自查：把同义参数列成一张表，出现两种写法就合并。
6. 提交前把命令树与参数矩阵贴进设计文档，让读者能用一句话复述规则。自查：找一个人复述，复述不出就是规则还没定。

这六条只管命名与参数层，管「怎么起名」，管不到「该不该有」。后者是第五节的内容。

## 五、设计问题：命令面从没被任务检验过

三版设计都是从架构图和流程图推出来的，没有一版从「谁在什么场合敲它」推出来。

命令是流程图的框。第一版六个一级命令与主文档流程图逐框对应——抽取、对齐、校验、变更、审批，用户指南那五步几乎每步都有一个动词。删的时候只能整框删，砍到最后剩下的 `change submit` 不是「最常敲的那条」，而是「流程最上游、唯一有输入的那一框」。当时回答「最有把握留哪个」给的理由是「唯一不可替代」，那是结构位置的判断，不是使用判断；没有真实调用者，这两种判断无法区分。

名词来自存储与三层架构。四个域名与四张表一一对应——`change_request` 到 `change`、`category` 到 `category`、`category_mapping` 到 `mapping`、`conflict` 到 `conflict`，命令面是架构图的镜像。而三层架构是这份方案自己的分期产物：主数据写入暂缓，范畴分析还没实现，架构还没站稳，命令面先照着它定死。

一次真实输入都没有。没有一段真实文本走过抽取到校验的链路，没有一条 `change_request` 被批准过，`--json` 的字段没有被任何脚本消费过。「该不该有这条命令」因此没有判据，只能靠「删了可不可惜」来猜；最后的答案是不可惜，一条也没留。

## 六、表达问题：写得像已经交付

文档用已交付产品的语气写不存在的东西，写得越完整，越容易把排版的成熟冒充设计的成立。

确定性是装出来的。同一页上「已实现的接口」与设计稿并排，退出码写成「0 成功、1 失败」的契约，`--json` 写「字段是脚本契约，只加不改」。一个从未跑起来的命令没有契约，这些句子把打算写成了承诺，读者分不清哪句是事实、哪句是愿望。

规则细节遮蔽了未决问题。第二版的篇幅大头是动词表、全局参数准入矩阵、「一个概念一套参数名」——都对，也都落地了，而「谁会在什么场合敲它」一句没写。命名越规范越像问题已经解决，所以第四节六条全部满足之后，命令面照样被整体否掉。

结构完整冒充设计完整。总览、本体篇、范畴篇各带设计取向与读写落点表，读第一眼是完整度；删的时候是按篇删的。分层写作是表达的选择，不是设计成立的证据。

## 七、边界与落地

留下的东西分两处归档。产品与技术判断与具体命令无关：链路止于 `change_request` 落库、人类确认是唯一关卡、生效归审批侧写本体文件、状态只有 SQLite 与 Turtle/OWL 两处、范畴分析以读为主、`--json` 字段只加不改、LLM 端点复用 `OLLAMA_HOST`、全局参数按「每条命令都消费」准入。前五条写在 `apps/qtcloud-meta/src/cli/docs/api-references/index.md` 的「设计取向」，任何一版命令设计都要在它们之内；本报告第四节的六条留在这里，管命名层。

被否定的是三版命令面：当前 CLI 没有命令设计，`src/cli/docs/api-references/` 只剩总览，何时重做未拍板。

数据来源仍是读码——各仓 `#[derive(Subcommand)]` 枚举与 `global = true` 参数，未逐仓实跑 `--help`；个案三版原文都在 qtcloud-meta 提交历史里，可复算。待拍板两件：第四节六条写成条文落在哪个仓；重新开命令树的起点，按第五节先交「谁在什么场合敲它」的答案，再动命令树。
