# pr4xis 引入评估

决策问题：pr4xis 是否引入标准库、引到什么程度。2026-10-05 修订二：三个实验已判失败并删除，本报告补全实验过程作为存档。读者无需技术背景。

## 结论与建议

- 本报告暂不构成引入依据。三轮实验只证明了工具能运行、按它自设的判据对了答案，现实业务要回答的问题一条未测。
- 两个前置条件记录在案：许可证非商业条款先过合规；版本锁 0.29.1。
- 下一轮实验的题目从现实业务问题出——规则表能少写多少人力、会不会漏拦真实违规操作，不测工具内部属性。

## 实验一与实验二：拦截演示

题目取自 gallery 两篇文档：主数据三层归属决议、日志与档案收敛规则。过程是先把文档规则翻成检查代码，再提交两个明知会被拦的操作。运行输出：

```text
✗ DeclaredInOntology: 本体未声明「本体」归 数据工程 —— 已声明的归属见 OwnedBy 边
✗ AlignmentComplete: 未对齐的日志事件：["10月3日调整预算", "10月5日暂停B渠道"]
✗ HumanConfirmed: 候选视图尚无人类确认，停在 pending
```

判失败：题目和答案都是自己出的，运行前就知道会被拦下，只证明了工具能运转。

## 实验三：盲测

协议：只输入文档明说的关系，隐含的一条不写；四条预测先提交入库（实验室仓提交 `33c84f6`，时间戳早于运行），跑完对答案；对照组是我们自己的推导清单和同样输入的 if 规则表。

材料取自四篇文档——主数据三层归属、文档格式归文档工程、信用与支付、本体标准属性——共 18 个概念、15 条明说关系：

- 主数据拥有定义、本体、运营三层；定义归元工程、本体归业务系统、运营归数据工程；
- 文档格式属于文档工程；文档工程与叙事工程对立；
- 元工程、数据工程、信用管理是领域；信用与支付对立；
- id 属基础属性组，基础属性组和分类属性组都是属性。

四条预测在运行前写死：该推的要推（id 属属性这条两级继承要被补出来）；两条不该合成的不合成（文档格式与叙事工程、主数据与元工程都不许自动连起来）；自带检查零失败。

运行命令 `cargo run --example implicit_relations_blind_test`，完整输出：

```text
[物化] 共 36 条关系：
  18 条恒等关系（每个概念到自身，系统自带，此处不逐条列）
  Definition -[OwnedBy]-> MetaEngineering
  OntologyLayer -[OwnedBy]-> BusinessSystem
  Operations -[OwnedBy]-> DataEngineering
  DocumentFormat -[Subsumption]-> DocumentEngineering
  MetaEngineering -[Subsumption]-> Domain
  DataEngineering -[Subsumption]-> Domain
  CreditManagement -[Subsumption]-> Domain
  Id -[Subsumption]-> BasicProperty
  BasicProperty -[Subsumption]-> Property
  ClassifiedProperty -[Subsumption]-> Property
  Definition -[Parthood]-> MasterData
  OntologyLayer -[Parthood]-> MasterData
  Operations -[Parthood]-> MasterData
  DocumentEngineering -[Opposition]-> NarrativeEngineering
  NarrativeEngineering -[Opposition]-> DocumentEngineering
  Credit -[Opposition]-> Payment
  Payment -[Opposition]-> Credit
  Id -[Subsumption]-> Property
[P1] is_a 两级链会物化出 Id→Property（继承闭包）（Id → Property）预期产生，实际产生
     符合
[P2] 对立不沿 is_a 传递，文档格式→叙事工程不应物化（DocumentFormat → NarrativeEngineering）预期不产生，实际不产生
     符合
[P3] has_a 与自定义归属边不复合，主数据→元工程不应物化（MasterData → MetaEngineering）预期不产生，实际不产生
     符合
[P4] 范畴定律 + 结构公理：检查 8 条（定律 3 + 结构 5 + 领域 0），失败 0 条

[盲测总成绩] 四条预测，符合 4 条
```

白话读这份输出：15 条声明落成 17 行（对立关系被补成双向），1 行是新补出来的（id 属属性），18 行是系统自带的恒等关系。预期外出现的只有对立关系被自动补成双向、以及拥有关系的方向被翻成「定义层是主数据的一部分」，两处语义都对，已裁决为正常。该推的一条推了，声明之外一条没多推，八条自带检查零失败，四条预测全中。

判失败：题目是「它推得对不对」这种工具内部问题。业务方要的是规则表能少写多少人力、会不会漏拦真实违规操作，一个都没答。报告前两版也因技术词业务方接不住被指看不懂。实验代码已随判决删除（实验室仓提交 `e7e138d`），本报告是三个实验的唯一存档。

## 引入成本与风险

- 没有界面。规则以代码形式存在，改规则要工程师改代码、跑测试、发版，产品经理和运营都改不了它。
- 维护依赖会 Rust 的工程师。它是 Rust 程序库，配置它靠改代码，招聘或培训口径按这个来。
- 许可证含非商业条款（CC-BY-NC-SA-4.0），代码若要随产品对外分发，先过合规。
- 文档不能直接当依据。按官方简介写的第一个版本编译不过，最终以源码为准；宣传材料与实际能力有出入，后续判断同样要实测。

## 待决事项

1. 重新设计题目来自现实业务问题的实验——这决定本报告能否从「暂不构成依据」走到「支持引入」。
2. 许可证合规确认的负责人和时限。
3. 版本锁定与升级策略由工程侧确认。
