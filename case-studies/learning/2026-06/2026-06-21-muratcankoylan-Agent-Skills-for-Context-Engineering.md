# muratcankoylan/Agent-Skills-for-Context-Engineering — 学习案例

**仓库**：https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering
**Stars**：15277 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-21（基于当前 HEAD）
**主题标签**：cross-reference, manifest-discipline, examples-driven, template-design, experience-accumulation

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

本仓库是一个专注于"上下文工程"（Context Engineering）的 Claude Code 技能库，收录 15 个核心技能（`skills/` 目录）和 5 个完整示例项目（`examples/` 目录）。截至 2026-06-21，仓库共有 15277 Stars，是目前公开的 Claude Code 生态中最系统化的上下文管理技能集合之一。

核心内容分布：
- `skills/context-fundamentals/SKILL.md`：上下文管理基础理论（其他技能的参照锚点）
- `skills/context-degradation/SKILL.md`：研究支撑的上下文退化阈值分析
- `skills/context-compression/SKILL.md`：基于探针（probe-based）的压缩评估方法论
- `skills/context-optimization/SKILL.md`：含具体性能目标的优化策略
- `skills/memory-systems/SKILL.md`：记忆系统框架对比表
- `skills/multi-agent-patterns/SKILL.md`：多智能体模式（含"电话游戏反模式"代码示例）
- `skills/filesystem-context/SKILL.md`：文件系统上下文管理
- `skills/latent-briefing/SKILL.md`：注意力匹配公式化方法
- `skills/harness-engineering/SKILL.md`（**新增**）：测试框架工程（审计后新增，15 技能 vs CLAUDE.md 中仍声称 14 个）
- 4 个 llm-as-judge-skills 下的 Agent 文件（`evaluator-agent.md`、`orchestrator-agent.md`、`research-agent.md`、`index.md`）：严重缺陷，均无 YAML frontmatter

### 1.2 架构剖析

仓库采用两层结构：

**核心技能层（`skills/`）**：每个子目录对应一个独立上下文工程主题，全部使用 `SKILL.md` 命名规范。经审计的 7 个技能文件得分均为 95/100（exemplary 级），另有 4 个得分 93/100（strong 级）。高分技能的共同特征：有完整 YAML frontmatter、清晰的 `## When to Use` 和 `## Examples` 章节、具体可操作的代码片段或决策框架。

**示例项目层（`examples/`）**：5 个完整子项目演示技能的实际集成方式：
- `examples/digital-brain-skill/`：个人知识管理原型，含自定义 Agent
- `examples/llm-as-judge-skills/`：LLM 评判系统（Agent 文件存在严重缺陷）
- `examples/book-sft-pipeline/`：书籍微调数据管道
- `examples/x-to-book-system/`：社交媒体内容转书籍系统
- `examples/interleaved-thinking/`：交织式思考模式示例

**Python 脚本层**：各技能目录下的 `scripts/` 子目录提供 10+ 个可执行辅助脚本（如 `skills/hosted-agents/scripts/sandbox_manager.py`），实现"技能文档 + 可运行代码"的双轨交付。

### 1.3 设计思路 / 方法论

本仓库最核心的设计哲学是**研究驱动（research-backed）的技能编写**：每个技能不停留在"建议"层面，而是给出具体阈值、基准对比数据和可量化的性能目标。例如：

- `context-degradation/SKILL.md` 给出研究文献支撑的上下文退化临界点（如超过 X tokens 后质量下降 Y%）
- `context-optimization/SKILL.md` 给出具体性能目标（"压缩率 >40%，质量损失 <5%"之类的可测量指标）
- `context-compression/SKILL.md` 提出基于探针（probe-based evaluation）的压缩质量评估方法——在压缩前后植入探针问题，通过答案准确率量化信息保留程度

第二个设计哲学是**反模式与正模式同等权重**：`multi-agent-patterns/SKILL.md` 专设"电话游戏模式"（telephone-game pattern）——用代码示例说明"坏"写法如何让上下文在多智能体传递中逐步损耗，并给出对应修复版本。这是"以失败案例为教学材料"的高价值模式。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

本仓库体现了以下可迁移的架构模式：

**模式 A：主题树形技能组织（Thematic Skill Tree）**
14-15 个技能围绕单一主题域（上下文工程）形成从基础到进阶的知识树：`context-fundamentals` → `context-degradation` → `context-compression` → `context-optimization`，后续技能可引用前序技能的概念。

**模式 B：技能 + 脚本双轨交付**
每个技能目录不只有 `SKILL.md`（理论文档），还附带 `scripts/` 下的可运行 Python 脚本，形成"读懂 → 跑通"的闭环。这降低了"知道但不会用"的鸿沟。

**模式 C：示例项目作为整合验证**
`examples/` 下的项目不只是演示代码，而是将多个技能整合到一个真实场景（如 digital-brain-skill 整合了 memory-systems + filesystem-context）。这是"单元技能"向"集成实践"的桥梁层。

**模式 D：同类实体同名约定**
所有核心技能统一命名 `SKILL.md`（全大写）；所有 Agent 文件预期以 `agent-name.md` 或 `AGENTS.md` 命名。统一命名约定让机械扫描和引用都更可靠。

### 2.2 适用场景

- 适合内容体量大、需要分层组织的技能库（10 个以上技能）
- 适合"教学型"插件：面向学习者，不只提供工具，还解释原理
- 适合有配套可执行代码的技能：技能文档 + 脚本形成互证

不适合：
- 单一功能的小型插件（两三个技能）——层级带来的维护开销超过收益
- 纯工具型插件（不需要理论背景的场景）

### 2.3 与其他架构对比

与 `forrestchang/andrej-karpathy-skills`（纯平铺技能，无示例项目层）相比，本仓库多了"示例项目层"这一集成验证层，代价是维护复杂度更高，且示例项目中的 Agent 文件更容易出现规范缺失（如本案中的 llm-as-judge-skills 全军覆没）。

与 `EveryInc/compound-engineering-plugin` 的"命令 + 智能体 + 技能"三层架构相比，本仓库没有命令层（没有 `/commands/` 目录），是"技能库"而非"功能插件"，使用方通过 `@` 引用技能，而非通过斜杠命令触发流程。

### 2.4 改进空间

1. **示例项目层的 Agent 文件质量管控缺位**：`examples/` 下的 Agent 文件没有纳入与核心 `skills/` 相同的质量标准，导致 llm-as-judge-skills 的 4 个 Agent 全部缺失 YAML frontmatter，审计快照得分仅 40-50/100。改进建议：为 `examples/` 下的 Agent 文件单独设立 CI 校验或文档模板。

2. **CLAUDE.md 技能计数未与实际同步**：新增 `skills/harness-engineering/SKILL.md` 后，CLAUDE.md 仍声称"14 skills"。这是清单纪律（manifest discipline）缺失的典型表现。

3. **root SKILL.md 与 CLAUDE.md 的引用规范冲突**：root SKILL.md 使用括号链接（`[技能名](路径)`），CLAUDE.md 建议使用纯文本技能名。两者互相矛盾，说明"规则"文档本身未被规则约束。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

总分 **83/100**，安全状态：**CLEAR**。
- 26 个 NL 制品，4 个 Bug，11 个质量问题，3 个安全发现（0 Critical、0 High、1 Medium、2 Low）
- 7 个 exemplary 级技能（95/100）：`filesystem-context`、`context-degradation`、`context-fundamentals`、`context-optimization`、`memory-systems`、`context-compression`、`multi-agent-patterns`
- 4 个 strong 级技能（93/100）：`advanced-evaluation`、`tool-design`、`project-development`、`evaluation`
- 问题文件拉低了整体均分：`examples/llm-as-judge-skills/agents/index.md`（40 分）、`evaluator-agent.md`（50 分）、`orchestrator-agent.md`（50 分）、`research-agent.md`（50 分）、`examples/digital-brain-skill/agents/AGENTS.md`（70 分）

### 3.2 当时值得借鉴的模式

**探针式评估方法论**（`skills/context-compression/SKILL.md`）：在压缩前后植入诊断性探针问题，通过回答准确率量化信息保留程度。这是将"感性判断"（"这个压缩好不好？"）转化为"可测量指标"的工程思维。

**电话游戏反模式 + 代码修复**（`skills/multi-agent-patterns/SKILL.md`）：用具体代码演示多智能体链路中上下文逐步失真的问题，并给出对应修复版本。"正反对照 + 可运行代码"的组合是技能编写的高信号模式。

**研究支撑的阈值声明**（`skills/context-degradation/SKILL.md`）：不只说"上下文过长会出问题"，而是给出文献支撑的具体临界值，让模型（和使用者）能做出数值化的决策。

**框架对比表**（`skills/memory-systems/SKILL.md`）：将多种记忆系统方案整理为结构化对比表（适用场景、性能特征、实现复杂度），帮助读者根据自身约束快速选型。

### 3.3 当时的缺陷

**Bug 1-4（同类缺陷，均为 Critical 级别功能失效）**：`examples/llm-as-judge-skills/` 下的全部 4 个 Agent 文件均缺失 YAML frontmatter：
- `evaluator-agent/evaluator-agent.md`：以 `# Evaluator Agent` 开头，无 `name:` 和 `description:` 字段
- `orchestrator-agent/orchestrator-agent.md`：同上
- `research-agent/research-agent.md`：同上
- `agents/index.md`：同上

没有 YAML frontmatter，这些 Agent 无法被 Claude Code 注册和发现，等同于失效的死代码。

**Medium 安全风险**：`skills/hosted-agents/scripts/sandbox_manager.py` 第 186 行，将 GitHub Token 嵌入 git clone URL 字符串格式：`x-access-token:{token}@github.com/...`。Token 会出现在进程列表（`ps aux`）和系统日志中，形成凭证泄露风险。

**Low 安全风险 1**：`examples/llm-as-judge-skills/package.json` 所有依赖使用 `^` 锁定（`ai ^4.0.0`、`@ai-sdk/openai ^1.0.0` 等），允许非预期的小版本自动升级，存在供应链风险。

**Low 安全风险 2**：`examples/digital-brain-skill/scripts/install.sh` 通过 `read -p` 读取用户提供的自定义路径，直接传入 `mkdir -p` 和 `cp -r`，未做路径净化，存在路径遍历风险。

**跨组件不一致**：`orchestrator-agent.md` 引用了不存在的 `writer` 和 `analyst` 两个 Agent（无对应目录 `writer-agent/` 或 `analyst-agent/`）——悬空引用（dangling reference）。

### 3.4 当时的优化机会

- `examples/digital-brain-skill/agents/AGENTS.md` 缺失 `model:` 字段（−5）、无 Agent I/O 示例（−15）、无输出格式说明（−10），降至 70 分，但并非不可修复
- root `SKILL.md` 与 `CLAUDE.md` 的引用规范冲突：前者用括号链接，后者要求纯文本——可通过统一 root SKILL.md 的引用格式修复
- `skills/latent-briefing/SKILL.md` 和 `skills/hosted-agents/SKILL.md` 可能缺少 `## Examples` 章节（参照其他技能的满分模式推断）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

通过对当前 HEAD（2026-06-21）的现场核查：

| 缺陷 | 审计时状态 | 当前状态 |
|------|-----------|---------|
| `evaluator-agent/evaluator-agent.md` 无 frontmatter | Bug | **仍未修复**，以 `# Evaluator Agent` 开头 |
| `orchestrator-agent/orchestrator-agent.md` 无 frontmatter | Bug | **仍未修复**，以 `# Orchestrator Agent` 开头 |
| `research-agent/research-agent.md` 无 frontmatter | Bug | **仍未修复**，以 `# Research Agent` 开头 |
| `agents/index.md` 无 frontmatter | Bug | **仍未修复**，以 `# Agents Index` 开头 |
| `orchestrator-agent.md` 悬空引用 writer/analyst | 质量问题 | 未核查（Agent 文件本身仍损坏） |

**结论**：4 个核心 Agent Bug 在至少 57 天内（2026-04-26 至 2026-06-21）未被修复，说明维护者可能不知晓或暂未优先处理示例项目层的规范问题。

### 4.2 架构演进

审计后新增了 `skills/harness-engineering/SKILL.md`，将技能总数从 14 增至 15。新技能聚焦于"测试框架工程"（harness engineering），与原有技能体系方向吻合，是架构的有机延伸。

但 CLAUDE.md 中的技能计数（"14 skills"）未同步更新——这是清单纪律问题，新技能在声明文件中隐形，降低了可发现性。

### 4.3 新增的可学习模式

`skills/harness-engineering/SKILL.md` 的新增演示了"主题技能库的有机生长"：核心技能体系建立后，可根据实践反馈增补新主题（测试框架是上下文工程实践中自然产生的需求）。这验证了"主题树形技能组织"（模式 A）的可扩展性。

反面教训：新技能的加入没有触发对 CLAUDE.md 技能计数的更新检查，说明该仓库缺少"新增技能时的清单同步检查"机制（无对应的 CI 规则或 pre-commit hook）。

---

## 五、校准

### 5.1 我已经在做对的

**Agent frontmatter 纪律（相对优势）**：本案中 4 个 Agent 文件全部缺失 YAML frontmatter，是评分崩塌的主因（40-50/100）。反观我的 `MarkQWu/echo-sleuth-for-claude`，5 个 Agent 文件（`memory-auditor`、`file-historian`、`schema-scout`、`recall`、`analyze`）全部有正确的 `name:` 和 `description:` frontmatter——这一点比本案做得好。

**单一功能聚焦**：`echo-sleuth-for-claude` 专注于 session 挖掘这一具体场景，没有过度延伸到"示例项目"层。避免了本案中"示例项目质量管控缺位"的问题。

### 5.2 挑战 / 验证

**技能的 Examples 章节缺失是共性弱点**：本案 exemplary 技能的高分（95/100）部分来自完整的 `## Examples` 章节。我的 `echo-sleuth-for-claude` 的 4 个 SKILL.md 文件（`0/4` 有 `## Examples` 章节）在这一维度上全面落后。这是需要重点补强的方向。

**输出格式声明缺失**：`echo-sleuth` 的 4 个 SKILL.md 中有 3 个缺少输出格式说明（output format section），而本案的高分技能均有明确的输出格式描述。

**研究支撑 vs 经验支撑**：本案最高分技能的核心优势是"研究支撑的具体阈值"（如 context-degradation 的退化临界值）。我的技能更多依赖经验性描述，缺少量化支撑。这不是所有场景的必要条件（legal workflow 技能未必有学术研究可引），但在技术性技能上是值得追求的。

---

## 六、行动

### 6.1 自查动作

1. **为 `echo-sleuth-for-claude` 的所有 4 个 SKILL.md 文件补写 `## Examples` 章节**
   - 目标文件：`skills/experience-synthesis/SKILL.md`、`skills/session-mining/SKILL.md`、`skills/pattern-recognition/SKILL.md`、`skills/context-bridging/SKILL.md`（路径待核实）
   - 格式参照：`skills/multi-agent-patterns/SKILL.md`（正反对照 + 代码片段）

2. **为缺少输出格式声明的 3 个 SKILL.md 补充 `## Output Format` 章节**
   - 使用结构化描述：返回什么类型的内容、格式（JSON/Markdown/纯文本）、必填字段

3. **核查所有仓库的 CLAUDE.md 技能计数是否与实际 SKILL.md 文件数一致**
   - 命令：`find skills/ -name "SKILL.md" | wc -l` 并与 CLAUDE.md 声明对比

4. **在 `echo-sleuth` 和其他仓库中检查是否存在悬空引用（dangling reference）**
   - 核查 Agent 文件中所引用的其他 Agent 名称是否有对应目录

5. **检查 `MarkQWu/claude-for-legal` 中是否存在类似的 Agent frontmatter 缺失问题**

### 6.2 灵感 → 实施路径

**灵感：探针式评估方法（来自 `context-compression/SKILL.md`）**
- 适用场景：在 `echo-sleuth` 的 `experience-synthesis` 技能中，可以引入"经验保留探针"——在会话挖掘前后植入特定问题，量化跨会话记忆质量
- 实施路径：在 `experience-synthesis/SKILL.md` 的 `## How to Use` 章节新增"探针验证步骤"，描述如何构造探针问题集并量化挖掘质量

**灵感：反模式 + 代码修复（来自 `multi-agent-patterns/SKILL.md`）**
- 适用场景：`echo-sleuth` 中的 `session-mining` 技能可以增加"反模式"章节，展示"直接转储 session 而不提炼关键信息"导致上下文爆炸的失败案例，并给出对应的精炼策略
- 实施路径：在 SKILL.md 中新增 `## Anti-patterns` 章节，包含一个"错误用法"代码块和一个"修复版本"代码块

**灵感：清单同步检查机制**
- 本案缺少的：新增技能后自动检查 CLAUDE.md 计数
- 实施路径：在 `.claude/hooks/` 中添加一个 PostToolUse hook，当写入 `SKILL.md` 文件时，提示检查 CLAUDE.md 中的技能计数声明

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 仓库 | 与本案主题的对齐度 | 说明 |
|------|------------------|------|
| `MarkQWu/echo-sleuth-for-claude` | **高**（结构对齐） | 同为多技能 Claude Code 插件；echo-sleuth 的 `experience-synthesis/SKILL.md` 处理跨会话上下文积累，与本案的 `context-degradation` / `memory-systems` 主题高度相邻 |
| `MarkQWu/drama-workshop-skills` | **低**（主题不同） | 中文微剧创作，仅 2 个 SKILL.md，规模和主题均与本案差异较大 |
| `MarkQWu/claude-for-legal` | **中**（结构参考） | 法律工作流插件，有子域组织需求，可参照本案的主题树形技能组织模式 |

### 8.2 在我的项目里复现的同类问题

**`echo-sleuth-for-claude`：SKILL.md 缺少 `## Examples` 章节（4/4 均缺失）**
- 本案对应：`context-fundamentals/SKILL.md` 等高分技能的 95 分与 `## Examples` 章节完整性直接相关
- 现状：`echo-sleuth` 的所有 4 个技能文件均无 Examples 章节，预计在 NLPM 评分中损失 10-15 分

**`echo-sleuth-for-claude`：3/4 SKILL.md 缺少输出格式声明**
- 本案对应：高分技能均有明确的 `## Output Format` 或相当的输出规范描述
- 影响：输出格式不明确时，模型生成结果的一致性下降

**CLAUDE.md 清单纪律（潜在风险）**
- 本案已暴露：新增第 15 个技能后 CLAUDE.md 未更新技能计数
- 在 `echo-sleuth` 中尚未触发（技能数量少，但随着增长可能出现）

### 8.3 别人的更优方案

**正反对照式技能描述**（`multi-agent-patterns/SKILL.md`）：本案明确展示"错误做法（电话游戏模式）→ 后果 → 修复代码"的三段式结构。这比 `echo-sleuth` 当前只有"建议做法"的单一视角更有教学价值和说服力。

**量化性能目标**（`context-optimization/SKILL.md`）：给出"压缩率 >40%，质量损失 <5%"等可测量指标。`echo-sleuth` 的技能描述目前停留在定性层面（"提高效率"之类），缺少可量化的验收标准。

**技能 + 脚本双轨交付**（`skills/*/scripts/` 目录）：每个技能附配套可运行 Python 脚本，实现"读懂 → 跑通"闭环。`echo-sleuth` 目前仅有 Agent 和 SKILL.md，没有脚本层。

**框架对比表**（`memory-systems/SKILL.md`）：用结构化表格帮助选型决策。在 `echo-sleuth` 的 `experience-synthesis` 技能中，可以加入"session 挖掘策略对比表"（全量扫描 vs 增量索引 vs 向量检索），类似本案的记忆系统对比表。

### 8.4 反向：我的项目做得比他们好的地方

**Agent frontmatter 纪律**：`echo-sleuth` 的 5 个 Agent 文件全部有正确的 `name:` 和 `description:` YAML frontmatter。本案 `examples/llm-as-judge-skills/` 下的 4 个 Agent 文件全部缺失，且至少 57 天未修复。这是 Agent 可发现性的基础，`echo-sleuth` 在此保持了零缺陷。

**功能聚焦 + 示例层质量管控**：`echo-sleuth` 没有"示例项目"层，避免了本案中示例项目成为质量短板的问题。当仓库没有能力维护多个子项目的规范时，不引入示例项目层是更明智的选择。

**无悬空引用**：`echo-sleuth` 的 Agent 之间没有相互引用不存在组件的问题（本案 `orchestrator-agent.md` 引用了不存在的 `writer` 和 `analyst` Agent）。规模小的好处是引用图更容易保持一致。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **YAML frontmatter** | Markdown 文件顶部以 `---` 包裹的元数据块，Claude Code 用于注册和发现 Agent / Skill 的必要字段（`name:`、`description:` 等） |
| **Exemplary 级**（95/100） | NLPM 评分体系中的最高质量区间，表示该制品在所有评分维度均接近满分 |
| **Strong 级**（93/100） | NLPM 评分体系中的高质量区间，存在少量可改进点但不影响核心功能 |
| **探针式评估（probe-based evaluation）** | 在处理前后植入诊断性问题（探针），通过答案准确率量化信息保留程度的评估方法 |
| **电话游戏反模式（telephone-game pattern）** | 多智能体链路中，上下文在逐步传递过程中因摘要/截断而失真，最终输出与原始信息偏离的反面模式 |
| **悬空引用（dangling reference）** | 文档中引用了不存在的组件（如 Agent、目录），导致读者或工具无法找到被引用对象 |
| **清单纪律（manifest discipline）** | 每次新增/删除组件后同步更新所有声明该组件计数或列表的文档（如 CLAUDE.md）的习惯 |
| **主题树形技能组织（Thematic Skill Tree）** | 围绕单一主题域构建从基础到进阶的分层技能体系，后续技能可引用前序技能的概念 |
| **技能 + 脚本双轨交付** | 技能文档（SKILL.md）与可运行辅助脚本（`scripts/`）同时提供，形成"读懂 → 跑通"闭环的交付模式 |
| **上下文工程（Context Engineering）** | 系统性管理大语言模型上下文窗口的方法论，包括压缩、优化、退化检测、跨会话记忆等子领域 |
| **caret 锁定（^ caret range）** | npm 依赖版本格式，`^4.0.0` 表示允许自动升级到 `4.x.x` 的最新版本，存在非预期的破坏性变更风险 |
| **路径遍历风险（path traversal）** | 用户提供的路径字符串经拼接后用于文件操作，若未净化可导致操作目标超出预期目录范围的安全问题 |
