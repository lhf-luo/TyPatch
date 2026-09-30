\# 论文精读笔记：TyPatch



> AI 辅助研究草稿。精读检查“问题 → 方法 → 实验 → 结论”的证据链是否成立。



\## 论文身份与证据边界



\- 研究方式：全文研究（严格 12 节）。

\- 标题：TyPatch: Transforming Patches into Typestate Rules for Kernel Bug Detection。

\- 作者：Ruoyu Wang、Tuo Li、Jia Li。

\- 年份/版本：2026，arXiv:2609.13728v1，2026-09-12 预印本。

\- 个人库：namespace `default`，paper ID `arxiv-2609-13728`。

\- 原始 PDF：`TyPatch Transforming Patches into Typestate Rules for Kernel Bug Detection \[preprint].pdf`；SHA-256 `0bf5c66b35f753abba39393b8d72bf71c187bf7da29930cfe57bf30570707312`；15 个物理页。

\- 实际阅读页段：MinerU 覆盖物理页 1–15，无缺页、无截断；原 PDF 定点核验物理页 1–12、14–15；直接渲染检查第 9、15 页。

\- MinerU 材料：current，schema\_version 2，15 页 / 253 blocks；仅作导航，结论已回到 PDF 核验。

\- 主要图表核验：Figure 1（p.2）、Figure 2（p.3）、Figure 3（p.4）、Figure 4（p.4）、Figure 5（p.5）、Figure 6（p.8）、Figure 7（p.9）；Table 1（p.8）、Table 2（p.8）、Table 3（p.9）、Table 4（p.10）、Table 5（p.15）、Table 6（p.15）；Algorithm 1（p.6）、Algorithm A.1（p.14）。

\- Artifact：https://github.com/THU-Agent/TyPatch ，commit `fd613f10322b806ca44f62f87dc32709b7bddb5a`，工作树 clean，获取字节成功。本文未执行代码、未逐文件核验实现，故不把仓库存在当作结果已复现。

\- paper\_progress：MinerU 全页覆盖完整；full.md 未完整游标遍历（以全页覆盖替代）；无未续读截断；原 PDF 已定点核验。无工具失败、无来源冲突。

\- 声明限制：559 distinct bugs、121 developer-confirmed、人工 TP 判定、token 统计均为作者报告，本次未独立验证。



\## 1. 研究问题、重要性与价值



研究问题：如何把 Linux 历史补丁中隐含的缺陷知识稳定转化为可重复运行的静态检测能力，而不是停留在一次性修复里。\[论文，物理 PDF 第 1 页，Abstract、§1]



现有 LLM patch-to-checker 路线要求模型同时完成两件事：(1) 从补丁恢复缺陷语义；(2) 实现完整静态分析器（回调、对象跟踪、别名、路径状态、跨过程传播）。TyPatch 主张第 (2) 项是跨补丁重复且易错的分析器工程，不应逐补丁重新生成。\[论文，物理 PDF 第 1–3 页，§1、§2]



重要性：内核规模大、配置与硬件相关路径多，普通测试只覆盖部分相关行为；传统静态分析能扫大范围代码，但可发现类别受限于专家已实现规则。\[论文，物理 PDF 第 1 页，§1]



TyPatch 的价值主张是改变 LLM 与分析的职责边界：LLM 产出补丁特定 typestate 规则，共享后端统一执行别名感知、路径敏感分析。\[论文，物理 PDF 第 2 页，Figure 1]



适用边界（论文自身限定）：主要面向顺序性对象状态关系；数值/边界、常量或 API 变更、并发历史、跨接口协议为明确延伸方向而非已解决范围。\[论文，物理 PDF 第 5、9–10 页，§3.2、§4.3、§5]



\## 2. 之前如何解决，以及缺口在哪里



传统静态分析：Coccinelle、Smatch、Clang Static Analyzer 提供语法匹配、路径敏感分析与 checker 扩展机制，但新增长尾模式仍需专家把语义编码为规则或 checker。\[论文，物理 PDF 第 1、11 页，§1、§6.1]



typestate 体系：PATA 与 SPATA 表明执行对象状态规则需要对象身份、路径可行性与跨过程传播等专门处理；TyPatch 复用这一基础能力。\[论文，物理 PDF 第 11 页，§6.1]



规则/规范自动恢复：APHP 从补丁代码与描述提取 API 后处理规范；SEAL 从修复前后程序依赖变化推断 source-to-use value-flow 规范；SpecAuditor、BugStone 先用规则缩小范围再由 LLM 判定候选。\[论文，物理 PDF 第 11–12 页，§6]



KNighter（最直接对照）：从补丁总结 bug pattern，生成实现计划，再由 LLM 写完整 CSA checker，经编译修复后在补丁的 buggy/fixed 版本上评估。\[论文，物理 PDF 第 11–12 页，§6.2]



缺口证据：KNighter 自身失败分析显示 22 个无有效 checker 的提交中 13 个（59%）归因于实现不准确，而 bug-pattern 分析错误 2 个、计划错误 7 个。\[论文，物理 PDF 第 1 页，§1]



论文用 cgbc 补丁隔离缺口：20/20 计划都识别出 allocation→alias→unchecked dereference，20 个 checker 都能编译，但 GPT-5.5 的 10 个 checker 仅 2 个、Opus 4.8 的 10 个中 0 个能区分 buggy 与 fixed seed；失败根因是写状态与查 alias map 使用了不同 CSA region key，导致 alias 更新被跳过。\[论文，物理 PDF 第 3 页，§2.1、Figure 2] 即：编译成功与计划正确都不保证 checker 语义正确。



\## 3. 重建作者可能的思考路径



\[推断] 以下为依据论文第 1–7 页动机与设计重建的推理链，不是作者逐字陈述。



1\. 从 KNighter 的失败分布观察：模型常能理解“缺什么检查”，但在翻译成 CSA callback/region/ProgramState 时失败。\[依赖：第 1、3 页]

2\. 划分可变与不变部分：随补丁变化的是对象、source、guard、sink、状态转换；跨补丁共用的是别名传播、CFG 状态传递、跨过程映射与报告构造。\[依赖：第 2–4 页]

3\. 重新划定生成边界：不生成 checker，而生成受限、可验证的声明式中间表示。\[依赖：第 2 页 Figure 1]

4\. 选择 typestate 作为表示：多数内核错误可写成对象在 acquire/check/use/release 等动作后的合法或非法状态。\[依赖：第 4–5 页]

5\. 必须给规则附加程序绑定：只说“释放后不可用”不足，需明确函数、参数、返回值、字段与 CFG 分支。\[依赖：第 5–6 页]

6\. 把结构性错误挡在扫描前：schema、API grounding、状态可达性、后端可执行性四类校验，失败最多修两轮。\[依赖：第 5–6、14 页]

7\. 共享引擎一次实现别名与路径机制；并采用 fail-open 证据筛选以保护召回，代价是误报。\[依赖：第 6–7、15 页]

8\. 用 KNighter matched comparison 验证成本与精度收益。\[依赖：第 8–9 页]



\## 4. 核心 intuition



本质：不让 LLM 为每个补丁重新实现分析器，而只让它描述“哪个对象、经历哪些事件、进入哪个状态、什么证据构成违规”，分析机制由共享后端提供。\[论文，物理 PDF 第 2 页，Figure 1]



如何针对 challenge：把一次性的 checker 实现错误转化为集中的、可复用的后端工程，并把模型输出压缩为受限模式，使校验与统计口径可控。



具体例（cgbc）：`AllocRet` 使 `devm\_kzalloc()` 返回值进入 `MaybeNull`；`BrNonNull` 在非空分支转 `NonNull`；未检查的 `Ref` 使 `MaybeNull` 到违规状态 `NPD`；后端负责把同一对象从返回值经 `hwmon->sensors` 字段传到局部 `sensor`。\[论文，物理 PDF 第 4–5、7 页，Figure 3、Figure 5、§3.3]



规则有意省略仅运输对象的无状态操作（未绑定的 load/store、字段地址计算、cast、实参到形参传递），由共享分析处理。\[论文，物理 PDF 第 5 页，§3.1]



\## 5. 具体方法与实现



输入：commit message、code diff、修复前后相关源代码上下文。输出：通过验证的 `.ts` 规则或 `NoRule`。\[论文，物理 PDF 第 2、5–6 页，Figure 1、§3.2]



规则结构：`R = <O, E, Q, q0, δ, qb, K>`。`O` 定义起始动作如何选出被跟踪对象；`E` 为有 typestate 意义的动作字母表；`Q` 为有限抽象状态域；`q0` 初始状态；`δ: Q×E→Q` 转移；`qb` 违规状态；证据契约 `K = (ē, Φ)`，`ē` 为有序关键动作序列，`Φ` 为路径一致性/动作位置不同等报告约束。\[论文，物理 PDF 第 4–5 页，§3.1、Figure 5]



动作绑定形式：对象来源可为函数返回、实参或 out parameter、字段值、托管分配；call 动作指定被调用者及其携带对象的参数与可选字段；return 动作选择调用结果；branch 动作选择 CFG 出边；memory 动作选择 load/store/dereference；exit 动作选择函数返回。例：`Call\[of\_node\_put,arg0]` 表示释放第一个实参所传对象。\[论文，物理 PDF 第 5 页，§3.1]



构造流程（Algorithm A.1）：BuildContext → GenerateRule → ValidateSchema → GroundBindings → NormalizeRule → ValidateStateMachine（`qb` 可达且按 `ē` 从 `q0` 可达）→ ValidateBackend（需存在标识对象的 source 动作、拒绝不支持的动作组合、确认可翻译为后端表示）→ SerializeTS；诊断按 rule-field 粒度反馈，最多修订两次，否则 `NoRule`。\[论文，物理 PDF 第 5–6、14 页，§3.2、Algorithm A.1]



实现细节：构造器用 Python，共享后端用 C++ over LLVM IR。上下文最多 3 个 changed-function body、每个 ≤200 行、合计 ≤500 行；源码展开支持普通函数、多行宏、一层 wrapper、内核 `DEFINE\_FREE` cleanup 声明，并跟随一层被调用者；每个片段保留 file:line provenance 供 grounding。few-shot 最多 2 个同 family 示例，无匹配则不给示例。序列化器补齐 `J\_Q`、把 `Φ` 映射为后端 fail-open 路径筛选策略。\[论文，物理 PDF 第 14 页，§A.2]



共享后端三项机制：

\- 动作绑定：loader 按绑定形式索引动作，解析到 instruction 或 CFG edge、选中值与可选 field path。\[论文，物理 PDF 第 6 页，§3.3]

\- 对象身份：`A = <N, F, ρ>`；store 替换目标 ref 边、load 跟随、getelementptr 创建/跟随字段边、cast 保持节点、`φ`/`select` 仅当所有输入解析为同一节点才保持；解析到的调用点实参与形参共享节点，使 out parameter 写入在返回后可见；driver-data setter/getter 对由 summary 连接。CFG join 时含同一 LLVM value 的前驱节点被统一，并按相同字段标签递归统一后继，规则 slot 先重映射再 join。\[论文，物理 PDF 第 6–7 页，§3.3]

\- 路径状态传播：移除 loop 与 return backedge，在得到的无环跨过程图上按拓扑序遍历；每条分析边携带 `Γ = <A, T, H, B>`（AliasGraph、按规则索引的对象状态、supporting histories、branch facts）；节点动作先更新再传播，guard 只施加于绑定边，join 用 `J\_r`；条件状态跟踪可保留互补 null/non-null 分支状态直到被 guard 或 join 合并，每个保留状态只带一个代表性 history。\[论文，物理 PDF 第 7 页，§3.3]

\- 证据筛选：到达 `qb` 先产生候选；当 `Φ\_r` 要求时在与单条路径相关的 CFG slice 上重放 `ē\_r`，每条路径维护自己的 alias graph、typestate 与 branch facts，检查是否同一对象按序满足约束；只有能证明不存在满足 `K\_r` 的路径时才拒绝，unknown 保留。\[论文，物理 PDF 第 7 页，§3.3、Algorithm 1]



分组执行：同一 link unit 内兼容规则共享一个 AliasGraph，每个对象带按规则索引的独立状态与 history；动作索引在付出一次对象转移成本后把事件分发给所有匹配规则，因此新增规则不需要新 checker 实现，扫描期也不需要模型调用。\[论文，物理 PDF 第 7 页，§3.3]



论文到代码映射：\[未知] 本次未逐文件核对仓库实现，未建立 path:line 级映射；仅确认仓库可获取且 commit 固定。



\## 6. 核心数学与理论背景



论文没有给出 soundness/completeness 定理，也没有形式化抽象域格性质证明；这更像带事件绑定与证据契约的有限 typestate 自动机，而非新的抽象解释理论。



\- 转移：`δ: Q×E→Q`，匹配动作按 `δ\_r` 更新被跟踪对象状态，达到 `qb` 表示候选违规。\[论文，物理 PDF 第 4–5 页，§3.1]

\- CFG join：每个状态域有确定性 `J\_Q: Q×Q→Q`；规则可声明 merge case，其余由 normalization 以共享 fallback 补齐；显式 case 优先，违规状态 absorbing。标量 uninitialized-use 规则中 initial 与 non-initial 合并保留 initial；生命周期规则中同一对保留 non-initial obligation；其余未匹配对按 CFG 的确定性 predecessor order 决定。\[论文，物理 PDF 第 5 页，§3.1]

\- 证据契约：`K = (ē, Φ)`；δ 说明动作如何改变状态，K 说明报告需要什么证据。`Φ = ∅` 时无需筛选。\[论文，物理 PDF 第 5–6 页，§3.1、Algorithm 1]



关键假设与取舍（变量含义：`A` 抽象对象集合，`F` 带标签边，`ρ` 为 LLVM value 到对象的偏映射）：

\- 规则状态域是有限且 join 由归一化补全的，故 join 语义部分依赖未声明的 fallback 策略。

\- 移除 loop/return backedge 换取拓扑遍历，代价是跨迭代历史不可表达。

\- fail-open 证据筛选使 `Φ` 只能删除被证明不可能的路径。



\[推断] 这些选择符合 bug-finding 而非 sound verification 的取向，也解释了体验到的 FP 水平（见第 7 节）。依据：论文第 6–8、15 页的算法与结果。



\## 7. 实验如何验证 claim



| Challenge / claim | 机制与假设 | 实验问题与对照 | 数据、预算、指标 | 结果与证据定位 | 未排除的解释 |

| --- | --- | --- | --- | --- | --- |

| 补丁知识可跨修复点迁移，能发现真实内核 bug | 规则只编码对象状态关系，后端统一传播，故可跨 subsystem 复用 | RQ1：100 个历史修复 × 8 个 family，三种模型独立生成规则池，在 Linux v6.16 全内核扫描 | 目标 commit `98a11dadac64`；双路 12 核 Xeon Silver 4310 ×2，256 GB RAM，Ubuntu 24.04，8 workers；按 source site 与下游 sink 合并；排除 seed 站点与测试代码报告 | 559 个 distinct bugs，121 个开发者确认；GPT/DS/Opus 分别 91/74/84 条规则；196 个三方共同发现，181 个两方，182 个单方；377/559（67.4%）由 ≥2 模型发现 \[论文，p.7–8，§4.2，Table 1] | 无 ground-truth universe，bug-level recall 未测；人工判定一致性未报告；>2000 报告规则被排除 |

| 规则池的候选质量高于 KNighter | 规则构造把实现错误隔离在共享后端外，故初始池 TP 更多 | RQ1 各模型池的 source-adjudicated precision | 未设报告数 gate | GPT 3,662 报告/319 TP/8.71%；DS 4,842/384/7.93%；Opus 5,655/429/7.59% \[论文，p.8，Table 2] | 绝对 precision 仍低（约 91%–92% FP）；人工裁决口径影响精度 |

| 改变生成边界降低构造成本 | 模型只输出受限规则，无需实现分析机制 | RQ2：KNighter 公共 38 补丁、六个共享 sequential family、相同两个模型、相同内核目标 | KNighter commit `f4e834b30741` 与 published default config；token 统计含初始生成、repair、scope planning | GPT：TyPatch 57/31、88 calls、850.8K vs KNighter 167/27、596 calls、7,284.0K；DS：62/25、90 calls、979.0K vs 198/23、826 calls、9,881.7K；token 减少 88.3%–90.1%，KNighter calls 为 6.8–9.2× \[论文，p.9，Table 3(a)；p.9–10，§4.3] | 未报告 seed/方差；scope planning 计入 token 但人工审阅不计；单次运行 |

| 初始报告精度优于 KNighter | 共享后端保证规则语义被执行 | RQ2 初始池（无 report-level 后处理） | 同一 38 补丁，按 sink file/function/line 在单个 artifact 内去重，跨 artifact 不去重 | GPT：214/4,584 = 4.67% vs 15/4,804 = 0.31%；DS：158/1,735 = 9.11% vs 58/2,178 = 2.66%；TP instances 2.72–14.27×、precision 3.42–14.95× \[论文，p.9，Table 3(b)；p.9–10，§4.3] | 比较对象是 report instances 而非 distinct bugs；倍数受 KNighter 极低基数放大 |

| LLM 能可靠把补丁翻成该表示 | 受限 schema + 两级校验降低语义漂移 | RQ3：对全部 249 条生成规则与 seed commit message/diff/source 人工比对 | 定义 aligned = 保留被跟踪对象及 source/discharge/sink 角色；放宽 predicate 或 API binding 仍算 aligned | 236/249 = 94.8%（GPT 88/91=96.7%、DS 70/74=94.6%、Opus 78/84=92.9%）；159 条精确保持、77 条放宽；13 条不一致中 action binding 6、object identity 4、state topology 2、failure predicate 1；13 条不一致规则产生 340 条 dedup findings 但仅 1 TP，aligned 对应 1,131/1,132 TP \[论文，p.10，§4.4，Table 4] | 无盲审/一致性指标，判定与 TP 标注同源；相关性不等于因果 |



补充观测：RQ1 每模型规则池构造成本 2.52–3.12M tokens（Table 5：GPT 2,523K、DS 2,911K、Opus 3,116K）。\[论文，物理 PDF 第 8、15 页，§4.2、Table 5] GPT 精度低于 DS 主要由三条过宽规则支配，产生 2,849 条额外报告中的 2,729 条（95.8%）：一条把并发 UAF 近似为顺序 free–use 产生 1,753 条，两条 uninitialized-data 规则产生 545 与 431 条。\[论文，物理 PDF 第 9、15 页，§4.3、§A.4、Table 6]



\## 8. Takeaways



可靠结论（有直接实验支撑）：

\- 在 38 补丁 matched 设置下，规则构造比完整 checker 构造消耗显著更少 token 与调用，并在两模型下产出更多可扫描 artifact。\[论文，p.9，Table 3(a)]

\- 249 条规则中 236 条保留 seed 的对象与 source/discharge/sink 角色，且这些规则贡献 1,131/1,132 model-specific TP。\[论文，p.10，§4.4]

\- 规则可跨 subsystem 迁移：RDMA→WWAN 的 nullable clone 检查、ALSA→media I2C 的 all-path initialization。\[论文，p.8–9，§4.2、Figure 7]



条件性结论：

\- 559 个 distinct bugs / 121 确认是在“100 seed、三模型、Linux v6.16、按 >2000 报告规则排除”的条件下得到。\[论文，p.7、14，§4.1、§A.3]

\- token 与 precision 优势是相对 KNighter 且限定其 commit 与默认配置。\[论文，p.14–15，§A.3]



容易误读：

\- “3.42–14.95× precision”是倍数而非高绝对精度（TyPatch 自身仅 4.67%/9.11%），且单位是 report instances。\[论文，p.9，Table 3(b)]

\- “559 bugs”不是 bug-level recall，系统无真值全集。\[论文，p.14–15，§A.3]

\- Artifact 存在不等于结果可复现；本次未执行任何代码。



\## 9. 最脆弱的假设



最脆弱的核心假设：\*\*大量内核补丁的缺陷语义可完整表达为顺序性 typestate 关系，且共享后端在该表示上足够准确。\*\*



失效会破坏什么：

\- 若语义不可表达，则 rule 生成会退化为错误近似（如把并发 UAF 写成顺序 free–use），直接产生海量误报——论文已观测到单条此类规则 1,753 条报告，并占 GPT 额外报告的 95.8%。\[论文，p.9、15，§4.3、§A.4]

\- 若后端 alias/join/路径判定不准确，则错误会系统化扩散到全部规则，因为分析语义被集中实现；论文自己也把 FP 主要归因于 different-object propagation、infeasible paths、以及 over-approximating guards。\[论文，p.7、15，§3.3、§A.4]

\- 论文报告的 non-artifact 集中在数值/边界、常量或 API 变更、非 protocol misuse、缺少 use 动作的 UAF，说明表示边界真实存在。\[论文，p.9，§4.3]



现有实验对它的检验程度：RQ3 只审计规则与 seed 的对象/动作/谓词/拓扑一致性，并未验证后端在一般 alias 与跨迭代 CFG 情形下的准确性；论文未报告后端 unit-level 正确性或与 PATA/SPATA 类分析的对照。→ 该假设只被部分检验。



\## 10. 一周最小复现实验



目标：验证“规则构造边界优于 checker 构造边界”这一核心 claim，并暴露后端 FT/FP/FN 模式。以下为人工执行计划，不代表已运行。



\- Day 1 固定材料：TyPatch commit `fd613f10322b806ca44f62f87dc32709b7bddb5a`；内核 commit `98a11dadac64`；选 3–5 个 seed（cgbc null deref、一个 release/lifecycle、一个 uninitialized-use、一个最可能 `NoRule` 的 bounds 或 concurrency 反例）。检查 Docker/IR/compilation database 是否完整，不复用预置结果。

\- Day 2 规则构造：核对 context 上限（3 函数/200 行/500 行）、grounding 能否解析规则引用的函数、`ē` 是否确实从 `q0` 到达 `qb`；再验证 buggy seed 报告、fixed seed 不报告。

\- Day 3 对象传播：构造最小 IR（kzalloc 返回 → 结构字段 → 局部别名 → 分支 guard → deref），分别扰动字段、guard、wrapper/out parameter，检查同一对象身份是否保持。

\- Day 4 路径行为：加入不可行分支、循环内 acquire/use、两个 allocation 汇合、`φ` 双来源、guard 位置前后变体，区分 TP/FP/FN。

\- Day 5 matched 小比较：记录规则生成 candidate/calls/token；如环境允许，按 KNighter commit `f4e834b30741` 与默认配置生成 checker，记录 buggy/fixed 区分率与初始警报数。

\- Day 6 判定稳定性：两位审阅者独立判定 ≥50 条报告（同一对象、动作顺序、路径可行、是否属 seed obligation），计算一致性并记录分歧原因。

\- Day 7 判定：成功 = ≥3 个可表达 seed 生成并执行规则、可区分 buggy/fixed、alias/field/guard 三类传播符合规则、token 与调用明显低于完整 checker 流程。



停止条件：seed 站点报告即大量出现（说明规则过宽）；或 fixed seed 仍高频报告（说明分歧判据无效）。预算：优先复用作者 Docker 与 IR，避免自行编译内核。



\## 11. 反例设计



\- 反例 A（不同对象在 merge 后被混淆）：两分支各分配对象、仅一线被 guard，随后经 `φ` 或字段合流再 deref；观察 join/alias 是否把安全对象状态转给不安全对象。针对 object identity 与 join。

\- 反例 B（跨迭代 typestate）：第 1 次迭代 acquire、第 2 次 release、第 3 次 use；因 loop backedge 被移除，检验是否能表达跨迭代历史。\[论文，p.6–7，Algorithm 1]

\- 反例 C（并发 UAF）：缺同步的 free 与 use；顺序规则可能要么过度报告要么漏掉 happens-before。论文已观测到近似规则产生 1,753 条报告。\[论文，p.9、15]

\- 反例 D（数值/边界依赖）：修复实为 `len <= bufsize`，对象生命周期正常；正确行为应返回 `NoRule` 而非生成看似合理的 typestate。\[论文，p.5、9，§3.2、§4.3]

\- 反例 E（seed 非完整 contract）：补丁只加 null check，真实 contract 还含错误码、refcount 与 cleanup 顺序；用 patch series 与 follow-up fix 检验规则是否应变化。\[论文，p.10，§5]

\- 反例 F（scope 泛化错误）：把 subsystem-specific API 规则错误扩到 kernel scope，观察是否产生大量无关报告。论文的 scope LLM 只能保留或扩大 local scope。\[论文，p.14，§A.3]



\## 12. 非增量 follow-up idea



方向：从“单补丁生成单规则”升级为\*\*反例驱动的可验证规则综合\*\*。



\- 现状：`one patch → one rule`，校验只覆盖 schema、grounding、状态可达与后端可翻译性，语义正确性由人工 RQ3 审计。\[论文，p.5–6、10]

\- 新机制：把 buggy/fixed 版本、patch series、follow-up fix 与相似 API usage 一并作为证据；LLM 只提出候选对象/动作/状态拓扑；用符号执行或差分分析自动生成正负 witness（buggy seed 必须达到 `qb`、fixed seed 必须不能达到、已知正常调用不得报告）；由反例驱动自动收紧 object binding、predicate、scope 与 join；仅当规则在多个正负样本上一致才进入共享规则库。

\- 结构性区别：从“结构化生成 + 语法/可执行性校验”变为“语义约束下的规则综合”，直接针对论文的主要残余错误（13 条不一致中 10 条来自对象/动作选择）。\[论文，p.10，Table 4(b)]

\- 最小判伪实验：选 10 条宽泛规则，注入自动生成的反例后重跑，观察 precision 是否显著上升而 recall 不塌；若负例约束无法排除 `Φ` 允许的 unknown 路径，则该方向无收益。

\- 失败条件：反例生成成本高于收益；或收紧后 buggy seed 也无法触发违规，说明 witness 约束与 fail-open 策略冲突。



\## 复现参数表



| 参数 | 最终值或 \[未知] | 类型 | 证据位置 | 置信度与缺口 |

| --- | --- | --- | --- | --- |

| 目标内核 | Linux v6.16，commit `98a11dadac64` | 论文报告 | 论文 PDF p.7 §4.1 | 高 |

| RQ1/RQ3 seeds | 100 个历史修复，8 个 family | 论文报告 | 论文 PDF p.7–8 §4.1、Table 1 | 高；具体 commit 列表未在正文列出 |

| RQ2 seeds | KNighter 公共集合 38 个 sequential 补丁（6 null-deref、5 leak、7 UAR、8 double-free、5 uninit、7 misuse） | 论文报告 | 论文 PDF p.14 §A.3 | 高 |

| 模型 | GPT-5.5、DeepSeek-v4-pro、Claude Opus 4.8（RQ2 不含 Opus） | 论文报告 | 论文 PDF p.7 §4.1 | 高；API snapshot/temperature/seed 未报告 |

| 硬件 | 2× Intel Xeon Silver 4310（各 12 核），256 GB RAM | 论文报告 | 论文 PDF p.7 §4.1 | 高 |

| OS / workers | Ubuntu 24.04 / 8 | 论文报告 | 论文 PDF p.7 §4.1 | 高 |

| 单 TU 超时 | 2 小时 | 论文报告 | 论文 PDF p.14 §A.3 | 高 |

| 构造上下文 | ≤3 changed functions，每个 ≤200 行，合计 ≤500 行 | 论文报告 | 论文 PDF p.14 §A.2 | 高 |

| wrapper/callee 展开 | 一层 | 论文报告 | 论文 PDF p.14 §A.2 | 高 |

| few-shot | ≤2 个同 family 示例，无匹配为 0 | 论文报告 | 论文 PDF p.14 §A.2 | 高 |

| 规则修订次数 | 初始 + 最多 2 次 repair（`for i ← 0 to 2`） | 论文报告 | 论文 PDF p.6 §3.2、p.14 Algorithm A.1 | 高 |

| RQ1 报告数 gate | 单规则 >2,000 raw reports 则整条规则排除（GPT 2/91、DS 1/74、Opus 3/84） | 论文报告 | 论文 PDF p.14 §A.3 | 高 |

| RQ2 报告数 gate | 无 | 论文报告 | 论文 PDF p.14 §A.3 | 高 |

| KNighter 版本 | commit `f4e834b30741` + published default configuration | 论文报告 | 论文 PDF p.14–15 §A.3 | 高 |

| token 口径 | provider-reported input/cached-input/output；含生成、repair、scope planning | 论文报告 | 论文 PDF p.15 §A.3 | 高；不含人工审阅 |

| TP 判据 | 有序动作在一条可行路径上作用于同一抽象对象 | 论文报告 | 论文 PDF p.15 §A.3 | 高；未报告审阅者与一致性 |

| TyPatch artifact | https://github.com/THU-Agent/TyPatch | 论文报告 | 论文 PDF p.10 §5 | 高 |

| 本地获取 commit | `fd613f10322b806ca44f62f87dc32709b7bddb5a`（工作树 clean） | 代码 | 官方仓库本地浅克隆 | 高；未逐文件核验 |

| 仓库许可证 | \[未知] | 未知 | 获取快照 licenseFiles 为空 | 使用前需人工确认 |

| 代码与论文的 path:line 映射 | \[未知] | 未知 | 未执行实现核验 | 需要复现时补做 |



\## 仍然未知的问题



1\. \[未知] 三种模型的精确 API snapshot、temperature、top-p、seed、system prompt 与完整生成 prompt 是否冻结公开。

2\. \[未知] 559 个 distinct bugs 的状态分布（合并/修复/确认未修/拒绝/待处理）。

3\. \[未知] 121 个 developer-confirmed 的确认证据、对应上游链接与去重映射。

4\. \[未知] 人工报告审阅的审阅者数量、是否盲审、inter-rater agreement 与分歧解决程序；RQ3 的 alignment 判定同样缺少一致性指标。

5\. \[未知] 共享后端相对 KNighter 的运行时、峰值内存与规则数增长时的 scaling 曲线；论文只报告生成 token 与硬件。

6\. \[未知] join fallback 按 CFG predecessor order 的结果对构图顺序的敏感性及代数性质。

7\. \[未知] 移除 loop/return backedge 对跨迭代 protocol bug 的漏报程度。

8\. \[未知] 被 >2,000 报告 gate 排除的规则含多少真实 bug，故无法估计完整规则池的无条件 precision/recall。

9\. \[未知] bug-level recall：无真值全集。

10\. \[未知] Artifact 的 Docker image、Linux IR 与 compilation database 是否与论文全部实验逐字节对应；本次未运行、未安装、未构建。

11\. \[未知] 规则池长期维护的冲突处理（同一对象被多条规则赋予不同状态语义时如何合并 / 优先级）；论文讨论“规则生态”但未给机制。\[论文，p.10 §5]



\## 人工备注



<!-- 留给用户。AI 不代填人工意见。 -->



