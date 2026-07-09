# 2389-research/simmer — 学习案例

**仓库**：https://github.com/2389-research/simmer
**Stars**：4 | **来源**：upstream（exemplar_published，SECURITY CLEAR，status=contributed）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-09（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `template-design`, `cross-reference`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Simmer 是一个「迭代精炼」插件：输入任意产物（单文件、多文件工作区、流水线、配置），根据用户定义的标准反复打磨改进。评估模式可以是主观的（评审团投票）、客观的（可运行脚本打分）或混合的（两者结合）。

关键事实：
- 作者 2389-research 同时维护 review-squad（本次 Case 1）
- 采用"路由 + 6 个子技能"分层架构，orchestrator 技能调度所有子技能
- 当前 HEAD 把主 SKILL.md 从 `skills/SKILL.md` 重命名为 `skills/simmer/SKILL.md`，架构有轻微重组
- 有 `tests/integration/simmer-scenario.md` 集成测试场景文件

### 1.2 架构剖析
```
simmer/
├── CLAUDE.md                       ← 插件总说明（无安装命令，无 Prerequisites）
├── .claude-plugin/plugin.json      ← manifest
├── skills/
│   ├── simmer/SKILL.md             ← 路由层：550 行 orchestrator（超限）
│   ├── simmer-setup/SKILL.md       ← 子技能：识别产物、引出标准
│   ├── simmer-generator/SKILL.md   ← 子技能：生成改进版候选
│   ├── simmer-judge/SKILL.md       ← 子技能：单评审打分
│   ├── simmer-judge-board/SKILL.md ← 子技能：多评审委员会模式
│   └── simmer-reflect/SKILL.md     ← 子技能：反思与总结
├── docs/specs/                     ← 设计文档
└── tests/integration/              ← 集成测试场景
```

- **文件类型分布**：6 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：`simmer` 作为 orchestrator，按 setup → generate → judge（单或委员会）→ reflect 调度子技能；每轮迭代结束后根据分数决定是否继续
- **跨件契约**：`simmer-judge-board` 是 `simmer-judge` 的"下沉替换"（JUDGE_MODE=board 时生效），orchestrator 中有明确切换逻辑

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「评估驱动生成」——先构建问题专属的评审员，让评审员读代码理解问题，再基于 ASI（Action-Score-Insight）反馈生成下一版
- **解决什么问题**：LLM 的"一次生成"质量天花板——通过多轮迭代+评分反馈突破单次生成的极限
- **Trade-off**：主观评审（灵活但不可重现）vs. 客观脚本（可重现但需要提前编写评估器）→ 作者选择支持两者，由用户在 setup 阶段选择
- **反映什么认知模型**：把"LLM 写代码"类比为"工匠打磨工件"——评审团不是外部裁判，而是内嵌于流程的质量门

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「Orchestrator + 专用子技能」模式**：一个主技能负责调度流程，每个子技能只做一件事，orchestrator 通过 JUDGE_MODE 等状态参数在子技能之间路由。

模式特征清单：
- 特征 1：orchestrator 是唯一入口，用户只需知道一个触发词（"simmer this"）
- 特征 2：子技能之间有明确的数据流：setup 的输出是 generate 的输入，judge 的输出是下一轮 generate 的输入
- 特征 3：orchestrator 持有状态（迭代计数、JUDGE_MODE、最佳候选）
- 特征 4：子技能可以被替换（simmer-judge ↔ simmer-judge-board）而不影响主流程
- 特征 5：有集成测试场景文件验证整体流程

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多轮迭代的产物（代码、文案、提示词） | ✅ 高度适用 | 正是 simmer 的核心用例 |
| 一次生成即可的简单任务 | ❌ 不适用 | 过度设计，路由层带来不必要的复杂度 |
| 需要客观可重现评分的场景 | ✅ 适用 | 支持脚本评估器，分数可追溯 |
| 轻量快速体验 | ❌ 不适用 | setup 阶段需要引出标准，门槛较高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Orchestrator + 子技能（本仓库） | simmer | 流程自动化，子技能可替换 | orchestrator 超长（550 行），维护成本高 |
| 角色面板（本次 Case 1） | review-squad | 零学习成本，视角独立 | 无法流程化，无数据流 |
| 单技能全流程 | 大多数 MVP 插件 | 实现简单 | 难以测试，难以迭代子步骤 |

### 2.4 改进空间
1. **当前问题**：CLAUDE.md 无安装命令 **改进做法**：加一行 `claude plugin install simmer@2389-research --scope project` **预期收益**：用户在 audit 报告 -20 分的 CLAUDE.md 短板消除
2. **当前问题**：orchestrator SKILL.md 550 行超过 500 行上限 **改进做法**：将"单智能体模式"相关 ~35 行提取到 `docs/single-agent-mode.md` 并链接 **预期收益**：分数 -10 penalty 消除
3. **当前问题**：`simmer-judge-board` 在 CLAUDE.md 目录树中缺失 **改进做法**：在 Directory Structure 段落补充该目录项 **预期收益**：降低新读者困惑

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 得分 93/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 75 | 无安装/测试/Prerequisites 说明（-20），描述性内容过多 |
| skills/SKILL.md（当时路径） | 88 | 550 行超限，"appropriate" 模糊量词 |
| skills/simmer-judge-board/SKILL.md | 90 | "relevant" ×5 |
| skills/simmer-judge/SKILL.md | 96 | "relevant" ×2 |
| skills/simmer-setup/SKILL.md | 98 | "some" 模糊量词 |
| plugin.json | 100 | 无 |
| simmer-generator/reflect | 100 | 无 |

### 3.2 当时值得借鉴的模式
1. **评估器先于生成** → simmer-setup 先引出评估标准，generator 才执行改进 → 把"评审标准"当一等公民 → 借鉴：我的自动化任务里，应先明确成功标准再开始执行
2. **JUDGE_MODE 参数化路由** → 单评审 vs 委员会通过一个参数切换，不需要修改主流程 → `skills/simmer/SKILL.md` 路由段落 → 借鉴：设计时预留扩展点，而不是硬编码流程
3. **ASI 格式化反馈** → 每轮 judge 输出 Action-Score-Insight 结构，generator 依赖此结构作下一版 → 子技能之间有明确的数据契约 → 借鉴：子技能间传递的数据应有格式规范
4. **集成测试场景** → `tests/integration/simmer-scenario.md` 记录了端到端测试用例 → 对 NL 系统做端到端测试 → 借鉴：为我的多技能流程写测试场景

### 3.3 当时的缺陷
1. **CLAUDE.md 无安装命令（-10 分）** → 为什么失败：用户找到插件后第一件事是想用，没有 `claude plugin install` 命令=要自己摸索 → 自查：bureau 的 CLAUDE.md 有 Prerequisites 但无安装命令
2. **orchestrator 550 行超限** → 为什么失败：超过 500 行的 SKILL.md 会触发 NLPM R05 penalty；更深层原因是单文件塞入了过多角色——路由逻辑、状态管理、单智能体模式说明混在一起 → 自查：我的 skills 有没有超 500 行的？
3. **"relevant" ×5 无定义（-10 分）** → 为什么失败：judge-board 的评审员需要知道"读哪些文件"，"relevant artifacts" 是 undefined behavior——不同 LLM、不同上下文会有不同判断 → 自查：我的技能文件里"relevant"使用频率

### 3.4 当时的优化机会
1. CLAUDE.md 加安装/测试/Prerequisites（3 个 penalty 合计 -20 分，只需加约 5 行文字）
2. 把 orchestrator 单智能体模式段落抽出到 docs/（-10 分 penalty 消除）
3. simmer-judge-board 的 5 处 "relevant" 逐一替换为具体文件/条件列表（-10 分消除）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 无安装命令 | `grep -n "plugin install\|Usage\|Prerequisites" CLAUDE.md` → 无结果 | **持续** | 此项未修，仍缺 -20 分 |
| orchestrator 550 行超限 | `wc -l skills/simmer/SKILL.md` → 550 行 | **持续**（路径变更但行数不变） | 内容未被提取，仍 550 行 |
| "relevant" ×5 in judge-board | `grep -n "relevant" skills/simmer-judge-board/SKILL.md` → L53/125/267/271/275 仍存在 | **持续** | 4 个月未修 |
| CLAUDE.md 目录树缺 simmer-judge-board | `grep "simmer-judge-board" CLAUDE.md` | **待验证**（CLAUDE.md 有 Directory Structure 段落） | 已有目录结构段落，但具体内容待核查 |

### 4.2 架构演进
主要变化：`skills/SKILL.md` 重命名为 `skills/simmer/SKILL.md`，使技能目录结构与 skill 名称对齐（`simmer:simmer` 对应 `skills/simmer/SKILL.md`）。这是一个纯命名规范改进，未见内容变化。docs 目录新增了 `specs/2026-03-16-simmer-v2-design.md` 设计文档，说明作者有版本化设计决策的习惯。

### 4.3 新增的可学习模式
当前 HEAD 有 `docs/specs/` 目录存放设计决策文档——这是 audit 时未覆盖的模式。把设计决策以带日期的 spec 文件记录，未来修改时能追溯当时的权衡。

---

## 五、校准

### 5.1 我已经在做对的
1. **集成测试思路**：echo-sleuth 和 bureau 均有测试目录，与 simmer 的 `tests/integration/` 做法对齐
2. **明确的成功标准**：bureau 的 capture 技能有明确的"什么算捕获成功"判断条件
3. **子技能职责边界**：bureau 的 compile/review 技能有清晰的职责边界，不互相跨界
4. **docs/ 目录存放设计文档**：bureau 有 `docs/` 目录，与 simmer 的 spec 文档习惯一致

### 5.2 挑战 / 验证
这个案例**挑战了**我对"orchestrator 越长越好"的隐性假设——550 行的 orchestrator 被 NLPM 判为缺陷，不是优点。关键洞察：orchestrator 的职责是**路由决策**，不是**知识文档**；把使用手册、边缘模式说明都塞进 orchestrator 会降低可维护性，应该链接到 docs/ 而不是内联。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 有没有超 500 行的
find ~/.claude/plugins/ -name "SKILL.md" -exec wc -l {} \; | sort -rn | head -10
```
命中后：把超出部分（通常是使用示例、背景说明）提取到 `docs/` 或 `references/` 并用 Markdown 链接引用。

```bash
# 检查 CLAUDE.md 是否有安装命令
grep -rn "plugin install\|Usage" ~/projects/*/CLAUDE.md
```
命中后（无结果即命中）：加一行 `claude plugin install <name>@<user> --scope project`

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 加 ASI 格式化反馈约定——compile 技能生成的 spec 文件应有固定格式，供 review 技能消费
   - **为何可行**：simmer 的 ASI 协议证明子技能间的结构化数据流可以显著减少 LLM 推断错误
   - **第一步**：在 `bureau/skills/compile/SKILL.md` 加 `## Output Format` 段落，定义 `## Bureau Spec` 格式，15 分钟
2. **想法**：给 gstack 的迭代类技能（如 retro、qa）加"迭代轮数上限"参数
   - **为何可行**：simmer 的 orchestrator 有明确的终止条件（分数达到阈值 or 轮数上限）；gstack 的迭代技能缺这个终止逻辑
   - **第一步**：在 `gstack/retro/SKILL.md` 加 `MAX_ROUNDS=3` 常量说明，5 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：通过多轮评审-生成迭代，将任意产物打磨至达标

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 也是对"内容产物"做流水线处理（capture→compile→review） | bureau 是知识管理，simmer 是通用迭代精炼；simmer 有评分，bureau 无量化评审 | 高 |
| MarkQWu/gstack | 中 | 有 retro/qa 等迭代类技能 | gstack 偏工作流管理，simmer 是单点精炼工具 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都处理历史内容 | echo-sleuth 是静态挖掘，simmer 是动态迭代 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 无安装命令 | `grep "plugin install" bureau/CLAUDE.md echo-sleuth-for-claude/CLAUDE.md` | **两者均无** | 高 |
| 技能文件超 500 行 | `find . -name "SKILL.md" -exec wc -l {} \; \| awk '$1>500'` | 需本地验证 | 中 |
| "relevant" 无定义标准 | `grep -rn "relevant" */skills/*/SKILL.md` | 需本地验证 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/CLAUDE.md` → 加 `claude plugin install bureau@MarkQWu --scope project` 一行 → 5 分钟
- `MarkQWu/echo-sleuth-for-claude/CLAUDE.md` → 同上 → 5 分钟

### 8.3 别人的更优方案

1. **领域**：子技能间结构化数据契约
   - **本案例做法**：judge 技能输出固定的 ASI 格式（Action + Score + Insight），generator 明确依赖此格式 → `skills/simmer-judge/SKILL.md` 末尾格式声明
   - **我的项目现状**：bureau 的 compile→review 数据流靠 LLM 自由推断格式，无明确契约
   - **如何借鉴**：在 `bureau/skills/compile/SKILL.md` 的 `## Output Format` 中定义固定输出格式；在 `review/SKILL.md` 的 `## When to Use` 中声明依赖 compile 的输出

2. **领域**：带日期的 spec 设计文档
   - **本案例做法**：`docs/specs/2026-03-16-simmer-v2-design.md` 记录架构演进决策
   - **我的项目现状**：bureau 的 docs/ 目录有文档但无 spec 规范
   - **如何借鉴**：每次大版本改动后，在 `docs/specs/` 下写一份带日期的决策记录

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：CLAUDE.md 有 Prerequisites 段落
- **我的做法**：`bureau/CLAUDE.md` 有 `## Prerequisites` 段落说明 Node ≥ 18 要求
- **本案例（弱在哪）**：simmer 的 CLAUDE.md 完全没有 Prerequisites 或安装说明，是 audit 里扣分最多的单项（-20 分）
- **意义**：我在这个维度做得更好；如果给 simmer 贡献 PR，这是最容易被接受的改进点

---

## 八、术语表

### <a name="orchestrator"></a>orchestrator（编排器）
> 在多技能插件中担任"指挥官"角色的主技能。它不直接执行任务，而是决定顺序、传递参数、在子技能之间路由，最后汇总结果。好的 orchestrator 应该只做路由决策，不应该内联大量业务逻辑或文档。

### <a name="ASI"></a>ASI（Action-Score-Insight）
> Simmer 评审技能输出的固定格式：Action（推荐的改进动作）+ Score（当前版本的量化分数）+ Insight（为什么这么评分）。这个三元组是 judge → generator 数据流的合同，生成器依赖 Action 和 Score 决定下一步改进方向。

### <a name="JUDGE_MODE"></a>JUDGE_MODE
> Simmer orchestrator 的一个状态参数，决定调用 `simmer-judge`（单评审）还是 `simmer-judge-board`（委员会）。这是一个"策略参数"——改变参数值就切换了评审策略，无需修改主流程。

### <a name="vague-quantifier"></a>模糊量词
> 见 Case 1 术语表：[模糊量词](#模糊量词)。在 simmer 中最典型的例子是 "relevant artifacts" ——没有指定"哪些文件算 relevant"，导致评审员每次判断不同。
