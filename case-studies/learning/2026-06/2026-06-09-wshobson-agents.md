# wshobson/agents — 学习案例
**仓库**: https://github.com/wshobson/agents
**Stars**: 33,764 | **来源**: xiaolai upstream
**Audit 日期**: 2026-04-17（历史快照）| **生成日期**: 2026-06-09（基于当前 HEAD）
**主题标签**: `manifest-discipline`, `single-purpose`, `examples-driven`, `model-pinning`, `fallback-chain`
**xiaolai 案例**: [../../2026-04-18-wshobson-agents-learnings.md](../../2026-04-18-wshobson-agents-learnings.md)

---

## 一、理解

### 1.1 仓库概览

wshobson/agents 是 GitHub 上星标数最高的 Claude Code 插件集合仓库之一，以 33,764 Star、3,708 Fork 事实上充当了"Claude Code 插件市集"的角色。仓库描述精炼："Intelligent automation and multi-agent orchestration for Claude Code"。

规模数据（2026-04-17 审计快照）：
- **40+ 插件**，覆盖 DevOps、机器学习、安全审计、SEO、前端、系统编程、量化交易等领域
- **64 个 agent 文件**，**36 个 command 文件**，共 100 个 NL artifacts 被采样
- 整体 NLPM 得分：**82/100（Silver 级）**，Agent 均分 86，Command 均分 74
- 安全评级：**CLEAR**（无 CRITICAL/HIGH 风险）

关键背景：wshobson 本人也在使用 Claude Code 辅助维护仓库——commit 历史中出现 `Co-authored-by: Claude Sonnet 4.6`，`validate.yml` CI 做 JSON 结构检查。这是一个典型的"用 AI 工具构建 AI 工具"的仓库，自动化维护痕迹明显。

### 1.2 架构剖析

**目录结构（审计快照）：**
```
plugins/
  agent-teams/                  ← 得分 90-92，最高分插件群
    agents/                     ← 每个 agent 一个 .md 文件
    commands/                   ← 对应 slash command 入口
  startup-business-analyst/     ← 唯一 command 有 allowed-tools 的非编排插件（92/100）
  full-stack-orchestration/     ← 最佳实践：分阶段工作流+检查点+恢复能力
    commands/full-stack-feature.md
    commands/tdd-cycle.md
  security-scanning/            ← 55/100（B-01: threat-modeling-expert 完全无 frontmatter）
  meigen-ai-design/             ← 72/100（B-02/03/04: 3 个 agent 缺少 name 字段）
  codebase-cleanup/             ← 57/100（3 个 command 全无 frontmatter）
  error-diagnostics/            ← 57/100（3 个 command 全无 frontmatter）
  ... （30+ 其他插件）
tools/
  requirements.txt              ← 5 个依赖全用 >= 宽松版本约束
```

**两种文化的边界**：从目录布局可以清楚看到两个截然不同的"邻区"——编排命令（orchestration commands）区和残缺插件（broken plugins）区。前者有 phased workflow、checkpoint、parallel dispatch；后者有内容丰富但 frontmatter 完全空白的 command 文件，甚至有整个插件下 3 个文件同时缺失的情况。

**19 个 Bug 的分布模式**：
| 类别 | 数量 | 受影响文件 |
|------|------|----------|
| Command 完全无 frontmatter | 15 | codebase-cleanup×3, error-diagnostics×3, tdd-refactor, framework-migration×2, c4-architecture, accessibility-compliance, systems-programming, database-cloud-optimization, javascript-typescript, deployment-validation |
| Agent 缺 name 字段（有 frontmatter 块但不完整）| 3 | meigen-ai-design 全部 3 个 agent |
| Agent 完全无 frontmatter | 1 | security-scanning/threat-modeling-expert |

### 1.3 设计思路 / 方法论

**核心设计哲学**：宽覆盖、高质量内容、但注册基础设施参差不齐。

wshobson 的仓库体现了两种明显的 authoring 文化并存：

1. **编排文化（Orchestration Culture）**：`full-stack-feature`、`tdd-cycle`、`performance-optimization` 这些命令设计精良——分阶段工作流、用户交互检查点、并行 agent 派发、显式 resume 能力。这类命令是"最佳实践的活教材"。

2. **批量创作文化（Batch Authoring Culture）**：大量插件内容本身写得详尽（部分超过 1000 行），但 YAML frontmatter 要么缺失、要么不完整。最大可能解释：这些文件在 frontmatter 规范确立之前被创建，或在批量创作时规范被遗漏应用。`meigen-ai-design` 是最清晰的证据——作者*知道* frontmatter 格式（有 `---` 块），但偏偏漏掉了 `name:` 字段。

**model tier 策略**：整体合理。Opus 用于生产代码任务，Sonnet 用于文档/调试，Haiku 用于快速/简单任务。`arm-cortex-expert` 原先用 `model: inherit` 处理深层嵌入式系统工作（后被修复）是唯一明显的 tier 错配。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

本仓库体现了两个值得独立命名的模式：

**模式一：「垂直插件市集」（Vertical Plugin Marketplace）**

结构定义：
```
一个 Git 仓库
  ↓ 包含
N 个独立插件（每个有独立 agents/ + commands/ 子目录）
  ↓ 每个插件
  agent（领域专家 persona）
  + command（用户入口 slash command）
  ↓ 共同构成
用户可按需安装的"应用商店"
```

与单一插件仓库最大的区别：插件之间彼此独立，用户可以只安装其中一个，不需要依赖整体。

**模式二：「编排命令 + 检查点」（Orchestration Command with Checkpoints）**

以 `full-stack-feature.md` 为代表：
```
Phase 1: 需求理解 → 用户确认 ✓
Phase 2: 架构设计 → 用户确认 ✓
Phase 3: 实现 → 可中断/恢复
Phase 4: 测试 → 并行 agent dispatch
Phase 5: 审查 → 汇总
```
每个阶段结束有显式 checkpoint，用户不满意可以重来，不会直接冲进下一阶段。这是与大量"一口气执行到底"的命令设计相比，最重要的质量差异。

### 2.2 适用场景

**垂直插件市集**适用于：
- 一个人或小团队维护覆盖多领域的 Claude Code 插件集合
- 想让用户按需安装，而不是一次性获取整个工具链
- 希望通过仓库星标数建立"品牌"，而不是通过发布到 npm/PyPI

不适用于：
- 插件之间有强依赖（应该用 monorepo + shared skill 而非独立插件目录）
- 需要版本化发布每个插件的独立版本

**编排命令+检查点**适用于：
- 跨越 30 分钟以上的复杂多步骤任务（全栈开发、TDD 周期、性能优化）
- 任务中间需要人类决策的分叉点
- 需要"上次做到哪里、继续"的中断恢复能力

### 2.3 与其他架构对比

| 架构 | 代表 | 优势 | 劣势 |
|------|------|------|------|
| 垂直插件市集（本仓库）| wshobson/agents | 覆盖广，用户可选择安装 | 插件间质量参差，frontmatter 纪律难以统一 |
| 单插件精品仓库 | 2389-research/review-squad | 质量高（96/100），维护专注 | 覆盖面小，靠精不靠多 |
| skill-only 仓库 | upstash/context7 skills | 平台无关，可跨插件引用 | 无 slash command 入口，靠用户自然语言触发 |

### 2.4 改进空间

1. **Frontmatter CI 门控**：`validate.yml` 当前只做 JSON 和结构检查。加一条 `grep -rL "^name:" plugins/*/commands/*.md` 即可在 PR 阶段拦截所有缺失 `name:` 的 command——成本 5 行 shell，收益覆盖 40% 命令组合注册失败问题。

2. **`startup-business-analyst` 模式复制**：该插件是唯一 command 中有正确 `allowed-tools` 声明的非编排插件，得分 92 vs 其他命令均分 74。将其 frontmatter 格式作为模板，在 `.github/CONTRIBUTING.md` 中明确要求每个 command 必须声明 `allowed-tools`。

3. **消除 3 个 verbatim 重复**：`security-auditor`、`code-reviewer`、`performance-engineer` 各存在两份 byte-for-byte 相同的副本。引入共享 skill 层，两个插件共同引用，而非各自维护一份。

4. **`protect-mcp` hooks 版本固定**：`npx protect-mcp@latest` 在每次工具调用时都可能下载不同版本的代码——这是一个持续存在的中危安全风险。固定到 `protect-mcp@1.2.3`（或当前最新稳定版本号）。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）

整体 NL 得分：**82/100（Silver 级）**

各插件得分明细：

| 插件 / 类别 | 得分 | 主要问题 |
|-----------|------|---------|
| agent-teams、startup-business-analyst、comprehensive-review、frontend-mobile、data-validation、python-development、observability-monitoring、database-design、performance-testing | 90–92 | 优秀，命令 allowed-tools 部分缺失 |
| conductor、ui-design、llm-application-dev、distributed-debugging | 86–88 | 少量 example 缺失或输出格式未声明 |
| meigen-ai-design | 72 | B-02/03/04：全部 3 个 agent 缺少 `name:` 字段 |
| security-scanning | 55 | B-01：threat-modeling-expert 完全无 YAML frontmatter |
| accessibility-compliance | 57 | command 完全无 frontmatter |
| codebase-cleanup | 57 | 全部 3 个 command 无 frontmatter |
| database-cloud-optimization | 57 | command 完全无 frontmatter |
| javascript-typescript | 57 | command 完全无 frontmatter |
| systems-programming | 57 | command 完全无 frontmatter |
| error-diagnostics | 57 | 全部 3 个 command 无 frontmatter |

**Agent 均分：86/100 | Command 均分：74/100 | 整体：82/100**

安全评级：**CLEAR**（含 1 个 Medium 风险 + 2 个 Low 风险，无 Critical/High）

安全发现明细：
- **Medium**：`plugins/protect-mcp/hooks/hooks.json` — `npx protect-mcp@latest` 在所有工具调用（matcher: `".*"`）时触发，每次都可能拉取并执行未经固定版本的 npm 代码
- **Low**：`protect-mcp@latest` 版本未固定
- **Low**：`tools/requirements.txt` — 全部 5 个依赖使用 `>=`（宽松约束），无上界限制

### 3.2 当时值得借鉴的模式

1. **编排命令三件套（最佳实践）**：`full-stack-feature`、`tdd-cycle`、`performance-optimization` 是本次审计周期中结构最佳的 Claude Code artifacts 之一。特征：分阶段工作流（Phase 1-N）、每阶段结束有用户确认检查点、并行 agent 派发、显式 resume 能力（"如果中断，从 Phase X 继续"）。这是"命令级 orchestration"的教科书示例。

2. **startup-business-analyst 的 allowed-tools 纪律**：这是唯一一个在非编排 command 中正确声明 `allowed-tools` 的插件，命令得分 92 vs 整体命令均分 74。来源不清楚（可能是不同贡献者，或后来才养成的习惯），但结果清晰：宣告工具权限让模型行为更可预测，也让 NLPM 评分更高。

3. **model tier 策略整体合理**：Opus → 复杂生产编码；Sonnet → 文档/调试；Haiku → 快速分类/简单任务。这种三级 model 分配思路与 NLPM conventions 完全对齐，说明作者对 Claude 模型能力的理解比大多数插件作者更深入。

4. **plugin 粒度**：每个插件有自己独立的 agents/ 和 commands/ 子目录，一个插件解决一个域的问题。`meigen-ai-design` 解决提示词工程，`startup-business-analyst` 解决商业分析，互不混淆。这种"单一领域单一插件"的设计使用户可以按需安装，而不是被迫接受整个仓库。

5. **validate.yml CI**：虽然只做 JSON 和结构检查，但它的存在本身是良好实践——任何 PR 必须通过 CI 才能合并，为后续加入更严格的 frontmatter 检查预留了门。

### 3.3 当时的缺陷

1. **B-01：threat-modeling-expert.md 完全无 YAML frontmatter**（最严重 bug）：文件以 `# Threat Modeling Expert` 纯 markdown 标题开头，没有任何 `---` frontmatter 块。Claude Code 无法注册该 agent，用户不知道这个 agent 存在——内容丰富但完全不可发现。

2. **B-02/03/04：meigen-ai-design 全部 3 个 agent 缺少 `name:` 字段**：这 3 个文件的 frontmatter 块本身存在（`---` 边界正确），但偏偏缺少 `name:` 字段——agent 知道格式，却无法自我介绍。说明作者曾经了解 frontmatter 格式，但在这一批次创作时漏掉了最关键字段。

3. **B-05 到 B-19：15 个 command 完全无 frontmatter**：整批命令（codebase-cleanup、error-diagnostics、tdd-refactor 等）内容详尽，部分超过 1000 行，但从未添加 `---` frontmatter 块。注册失败静默发生：no error、no warning，slash command 调色板中根本没有对应命令。

4. **Q-01/02/03：3 对 verbatim 重复文件**：
   - `comprehensive-review/security-auditor.md` = `security-scanning/security-auditor.md`
   - `comprehensive-review/code-reviewer.md` = `code-documentation/code-reviewer.md`
   - `performance-testing-review/performance-engineer.md` = `observability-monitoring/performance-engineer.md`
   修改任何一个 agent 时必须同时记得改两处，维护成本加倍，漂移风险持续存在。

5. **Q-06：30+ 个 command 缺少 `allowed-tools`**：几乎整个命令组合都没有声明工具访问权限，不符合最小权限原则，也是 Agent 均分（86）和 Command 均分（74）之间 12 分差距的主要成因。

6. **Q-07：arm-cortex-expert 的 `tools: []`**：空数组意味着该 agent 无法读取任何文件。对于深层嵌入式系统工作而言，这个限制极为严格——在无法访问源文件的情况下，agent 只能基于用户粘贴的代码片段工作，完全无法主动探索代码库。

7. **Q-10：arm-cortex-expert 使用 `model: inherit`**：深层嵌入式系统分析是高复杂度任务，应当显式指定 Opus，而非继承父命令的模型设置——`inherit` 在不同调用上下文中会产生不一致的行为。

### 3.4 当时的优化机会

1. 为 `threat-modeling-expert.md` 添加完整 YAML frontmatter（`name`、`description`、`model: opus`）— 单文件修复，5 分钟。

2. 为 meigen-ai-design 的 3 个 agent 补充 `name:` 字段 — 每文件加 1 行，10 分钟。

3. 分批为 15 个无 frontmatter 的 command 添加 `description:` 字段 — 按插件分组，每组 1 个 PR，机械操作。

4. 将 3 对重复文件统一到共享 skill 层，其他两处改为引用而非复制。

5. 在 `validate.yml` CI 中加入 `name:` 字段检查，防止同类 bug 再次混入。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

以下基于当前 HEAD（Phase 3 验证）：

| 缺陷 | 2026-04-17 状态 | 2026-06-09 状态 | 来源 |
|------|---------------|---------------|------|
| B-01: threat-modeling-expert 无 frontmatter | 完全无 frontmatter | `name: threat-modeling-expert`、`description: Expert in threat modeling...`、`model: opus` 均已添加 | xiaolai PR #488 合并 |
| B-02/03/04: meigen-ai-design 3 个 agent 缺 name | `name:` 字段缺失 | `prompt-crafter.md` 等均已有完整 name + description | xiaolai PR #489 合并 |
| B-05~B-19: 15 个 command 无 frontmatter | 完全无 frontmatter | `tech-debt.md` 有 `description: Analyze and remediate technical debt...`；`error-trace.md` 有 `description: Set up error tracking...`；其余类同 | xiaolai PR #490/#491/#492 + 维护者独立修复 |
| Q-06: 30+ command 缺 allowed-tools | 大量缺失 | re-audit 出现 99 条 R11（missing-tools）——问题仍然普遍存在 | 未修复 |
| Q-01/02/03: 3 对重复文件 | 存在 | re-audit 显示已解决（37 项原始发现全部关闭） | 维护者独立修复 |
| protect-mcp@latest 安全风险 | 中危 | 未在 re-audit 表中再次验证——状态不明 | 未知 |

**总结**：37 个原始发现全部已解决（13 个通过 xiaolai 的 5 个 PR，24 个由维护者独立处理）。re-audit（2026-04-24）得分从 82 升至 **84/100**。

### 4.2 架构演进

re-audit 发现 204 个新问题（主要是 R11 missing-tools 和 R12 missing-output-format），并有新插件出现（`developer-essentials`、`agent-orchestration`、`cicd-automation`、`deployment-strategies` 等）。说明：

1. 仓库在两次审计之间持续扩张——这是活跃维护的信号，也是"frontmatter 纪律问题随规模放大"的直观证明。
2. 1 个新文件（`plugins/developer-essentials/agents/monorepo-architect.md`）被完全无 frontmatter 提交——同类 bug 在修复后再次出现，说明没有 CI 门控的话，批量创作模式会持续产生同类错误。
3. 99 条 R11（missing-tools）发现反映了一个一致的设计选择而非反复遗忘——作者显然认为 `allowed-tools` 不是必须的。

### 4.3 新增的可学习模式

re-audit 引入的 204 个新发现揭示了一个重要教训：**规模化扩张放大了所有已知的 manifest 纪律缺陷**。每新增一个插件就可能带入 2-3 个 R11/R12 发现，但这些问题对用户来说几乎不可见——命令还是可以运行，只是工具访问权限未被显式声明。与 B-01 到 B-19 类的"完全无法注册"相比，R11/R12 是"功能可用但权限模糊"的更隐蔽问题。

---

## 五、校准

### 5.1 我已经在做对的

1. **我的 agent 文件没有缺少 `name:` 字段的情况**：`echo-sleuth-for-claude` 的 5 个 agent 和 `claude-for-legal` 的 16 个 agent 全部有正确的 `name:` 声明。wshobson/agents 在 33,764 Star 的规模下仍有 4 个 agent 因为缺少 `name:` 而无法注册——这说明规模本身不能保证基础质量，但我的仓库目前在这个最基础的指标上是干净的。

2. **无完全无 frontmatter 的文件**：echo-sleuth 的所有 command 和 agent 文件均有 frontmatter 块。wshobson 的 15 个 command 完全没有 `---` 块——这是最初级的注册失败，我完全避免了。

3. **无 verbatim 重复**：我的 3 个仓库中没有 byte-for-byte 相同的文件存在于两个位置的情况。wshobson 的 3 对重复直接导致维护成本加倍。

4. **model 声明显式**：我的 agent 没有使用 `model: inherit` 做复杂任务的情况，所有需要高质量输出的 agent 均有显式 model 声明。

### 5.2 挑战 / 验证

1. **echo-sleuth 的 4 个 command 缺少 `allowed-tools`**：`lessons.md`、`recall.md`、`recap.md`、`timeline.md` 均无 `allowed-tools:` 声明。这与 wshobson 的 Q-06（30+ command 缺失）是完全相同的问题——只是规模小得多，但性质一样。wshobson 的案例证明这个问题不会"自动消失"（re-audit 出现 99 条 R11 就是证明），越拖越难修复。

2. **claude-for-legal 的 218 个模糊量词**：这是比 wshobson 更严重的问题——wshobson 的问题更多是结构性的（缺 frontmatter），我的问题更多是语义性的（语言模糊）。两种问题都会损害 NLPM 得分，但语义问题更难批量修复。

3. **我没有编排命令**：wshobson 的最佳实践是分阶段工作流（full-stack-feature、tdd-cycle）——这类命令在我的仓库中完全缺失。这不是 bug，而是架构成熟度的差距。我的 echo-sleuth 和 claude-for-legal 的 command 都是"一次性执行"的简单命令，缺乏 checkpoint 和 resume 能力。

---

## 六、行动

### 6.1 自查动作

针对 `MarkQWu/echo-sleuth-for-claude`，按优先级：

1. **P0 — 立即（10 分钟）**：为 `lessons.md`、`recall.md`、`recap.md`、`timeline.md` 补充 `allowed-tools:` frontmatter。示例：
   ```yaml
   ---
   name: recall
   description: Search past Claude Code sessions for decisions and patterns.
   allowed-tools:
     - Read
     - Bash
   ---
   ```
   这是与 wshobson B-05~B-19 同类的问题，修复成本极低，拖延代价极高（re-audit 99 条 R11 就是代价）。

2. **P1 — 本周（2 小时）**：扫描 claude-for-legal 中的 218 个模糊量词，优先修复高频词（"comprehensive"、"appropriate"、"robust"、"relevant"）。每次替换一个词，用具体可验证标准代替。示例：`"comprehensive analysis"` → `"analysis covering [specific dimensions: jurisdiction, liability, risk level]"`。

3. **P2 — 下月（半天）**：为 echo-sleuth 的至少 2 个复杂 command 添加 checkpoint 结构（参考 wshobson 编排命令三件套）。选择最适合的候选：`recap.md`（涉及多步分析）或 `timeline.md`（跨会话状态重建）。

4. **P3 — 持续**：在 echo-sleuth 和 claude-for-legal 的 PR 模板中加一行 checklist：`- [ ] frontmatter 包含 name、description、allowed-tools`，防止新 command 漏掉 allowed-tools。

### 6.2 灵感 → 实施路径

**从 wshobson 编排命令学习 checkpoint 设计**：

`full-stack-feature.md` 的设计模式可以用以下 5 步提炼：
1. 明确列出所有 Phase（Phase 1: 理解需求、Phase 2: 设计、...）
2. 每个 Phase 结束有一个显式确认问题（"以上设计是否符合你的预期？[Y/继续/调整]"）
3. 最后一行写明 resume 方式（"如果中断，请告诉我从哪个 Phase 继续"）
4. 并行任务（如多个测试）用显式 parallel dispatch 标记
5. 所有 Phase 总计不超过一个屏幕——强迫精简

实施路径：打开 echo-sleuth 最复杂的 command（建议 `recap.md`），对照上述 5 步，分 3 个迭代添加 checkpoint 结构。每个迭代只加一步，不要全部重写。

**从 wshobson 的 bug 历史学习 CI 门控**：

startup-business-analyst 之所以是唯一有 allowed-tools 的非编排插件，大概率是因为那个插件的作者在不同时期写的，而不是因为 wshobson 有系统性的门控。如果 CI 有检查，剩余 30+ 个 command 在提交时就会被拦截，而不是在 NLPM 审计时才被发现。

实施路径：在 echo-sleuth 的 `.github/workflows/validate.yml`（如果存在）或新建 `pre-commit` hook 中加入：
```bash
# 检查所有 commands/*.md 是否有 allowed-tools
grep -rL "^allowed-tools:" .claude/commands/ && echo "Missing allowed-tools!" && exit 1
```

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 与 wshobson/agents 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|----------------------|------|------|----------|
| MarkQWu/claude-for-legal | **高** | 都是多域专业插件集合（legal 有 corporate/employment/privacy/product 等域，wshobson 有 DevOps/ML/security 等域）；都有 agent + command 双层架构；都面临"批量创作导致质量参差"的挑战 | wshobson 面向开发者，claude-for-legal 面向法律场景；wshobson 有编排命令，claude-for-legal 无 | 高 |
| MarkQWu/echo-sleuth-for-claude | **中** | 都有 commands + agents 双层结构；都有 allowed-tools 缺失问题（wshobson 30+，echo-sleuth 4 个）；安全评级都是 CLEAR | 规模差异巨大（5 vs 64 agents）；echo-sleuth 更聚焦单一能力 | 中 |
| MarkQWu/drama-workshop-skills | **低** | 都是 Claude Code 插件 | 领域完全不同（短剧创作 vs 开发工具集）；drama-workshop 以 skills 为主，无 commands | 低 |

**核心结论**：`MarkQWu/claude-for-legal` 是与 wshobson/agents 对齐度最高的仓库。两者都面临"多域专业插件集合的 manifest 纪律挑战"——区别只是 wshobson 已经被 NLPM 审计+修复，而 claude-for-legal 还没有。

### 8.2 在我的项目里复现的同类问题

| wshobson 缺陷 | 我的仓库复现情况 | 严重度 |
|-------------|--------------|------|
| 15 个 command 完全无 frontmatter（B-05~B-19）| claude-for-legal：待验证；echo-sleuth：无（全有 frontmatter）| claude-for-legal 未知风险 |
| 30+ command 缺 `allowed-tools`（Q-06）| echo-sleuth：`lessons.md`、`recall.md`、`recap.md`、`timeline.md` 均缺失（4/8 = 50%，比例高于 wshobson）| 高 |
| verbatim 重复文件（Q-01/02/03）| 3 个仓库均无重复文件 | 无 |
| 模糊量词（wshobson 少量，均已修复）| claude-for-legal：218 处（严重）；echo-sleuth：少量 | 高 |
| arm-cortex-expert `tools: []`（Q-07）| 需检查：echo-sleuth 是否有 agent 有 `tools: []` 或空数组声明 | 待查 |
| `model: inherit` 用于复杂任务（Q-10）| 需检查 claude-for-legal 的 16 个 agent 是否有 `model: inherit` | 待查 |

**最高优先级同构问题**：echo-sleuth 的 `allowed-tools` 缺失（4/8 command = 50%，比 wshobson 的 30+/36 = 83% 略好但趋势相同）。wshobson 的教训是：这个问题不会随着时间自我修复——re-audit 的 99 条 R11 就是不修复的代价。

### 8.3 别人的更优方案

1. **编排命令的 checkpoint 设计（wshobson full-stack-feature > 我的所有 commands）**：
   - wshobson 做法：Phase 1-N 分段，每段结束有用户确认，支持中断恢复，并行 dispatch 显式标记。
   - 我的现状：所有 command 都是"一次性执行"，没有阶段划分，没有 checkpoint，没有 resume 机制。
   - 借鉴方法：不需要把所有 command 都改成编排模式——只选 echo-sleuth 中最复杂的 1 个（`recap.md`：跨会话分析）和 claude-for-legal 中最复杂的 1 个（例如综合法律尽职调查 command），各自加 Phase 结构。

2. **model tier 三级策略（wshobson 整体合理 > 我的未系统化）**：
   - wshobson 做法：Opus/Sonnet/Haiku 按任务复杂度分层，每个 agent 的 model 声明与其工作性质匹配。
   - 我的现状：claude-for-legal 的 model 声明不确定是否系统化分层——需要审查。
   - 借鉴方法：列出 claude-for-legal 16 个 agent 的 model 字段，检查高复杂度任务（合同审查、风险评估）是否指定了 Opus。

3. **单插件单域的粒度控制（wshobson 清晰 > claude-for-legal 域混合）**：
   - wshobson 做法：`meigen-ai-design` 只做提示词设计，`startup-business-analyst` 只做商业分析，严格单域。
   - 我的现状：claude-for-legal 的某些 agent 可能同时覆盖多个法律子域（待审查）。
   - 借鉴方法：检查 claude-for-legal 是否有"大而全"的 agent，如有，考虑拆分为独立插件（corporate-legal 和 employment-legal 分别有独立的 agents/ 目录）。

### 8.4 反向：我的项目做得比他们好的地方

1. **零 frontmatter 完全缺失文件**：echo-sleuth 和 drama-workshop-skills 的所有 NL artifact 文件都有正确的 frontmatter 块。wshobson 在 33K Star 的体量下仍有 1 个 agent 完全无 frontmatter（B-01）、15 个 command 完全无 frontmatter（B-05~B-19）——这是最基础的注册可靠性指标，我做得更好。

2. **无 verbatim 重复**：我的 3 个仓库中没有任何文件被复制到两个不同位置。wshobson 的 3 对重复（security-auditor、code-reviewer、performance-engineer）使维护成本加倍，而且已经被 NLPM 标记为 CC-duplication 问题。

3. **插件粒度单一（drama-workshop-skills）**：drama-workshop-skills 是单一领域的精品插件，每个 skill 服务明确的创作步骤，没有"大而全"的倾向。与 wshobson 的多域集合相比，这种设计维护成本低，质量更容易控制——对应 wshobson 的 `single-purpose` 最佳实践标签。

4. **无 `tools: []` 空数组陷阱**：echo-sleuth 的 agent 都没有声明 `tools: []` — 这个隐蔽错误让 wshobson 的 `arm-cortex-expert` 完全无法读取文件，是功能性残缺。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **frontmatter** | Markdown 文件顶部以 `---` 包裹的 YAML 元数据块。在 Claude Code NL artifacts 中，frontmatter 声明 `name`、`description`、`model`、`allowed-tools` 等关键字段。缺少整个 frontmatter 块（B-01~B-19 类 bug）会导致 Claude Code 完全无法注册该 artifact；有 frontmatter 块但缺少 `name:` 字段（B-02/03/04 类 bug）同样导致注册失败，但更难被发现。 |
| **YAML** | Yet Another Markup Language，一种人类可读的数据序列化格式，在 Claude Code NL artifacts 中用于编写 frontmatter。在 `---` 边界内的 key: value 对即为 YAML 语法。缺少有效 YAML 语法的 frontmatter 无法被 Claude Code plugin loader 解析。 |
| **allowed-tools** | command 或 agent frontmatter 中声明该 NL artifact 可以调用的工具列表（如 `Read`、`Bash`、`mcp__toolname`）。遵循最小权限原则——只声明实际需要的工具。wshobson 的 Q-06 问题（30+ command 缺失此字段）和 echo-sleuth 的 4 个 command 缺失此字段是同类问题：命令仍然可以运行，但工具访问权限未被显式声明，给模型行为带来不确定性。 |
| **plugin（插件）** | Claude Code 的可安装扩展单元，通常包含一个 `.claude-plugin/plugin.json` manifest 文件，以及 `agents/`、`commands/`、`skills/` 子目录。wshobson/agents 包含 40+ 个独立插件，每个插件解决一个特定领域的问题。通过 `/plugin install` 命令安装。 |
| **orchestration（编排）** | 在 Claude Code 语境中，指协调多个 agent 或多步骤任务的命令设计模式。wshobson 的 `full-stack-feature`、`tdd-cycle` 是编排命令的最佳实践：定义明确的 Phase（阶段）、每阶段有 checkpoint（检查点）、支持 resume（恢复执行）、并行 dispatch（并行派发）多个 agent。与单步骤命令相比，编排命令适合耗时 30 分钟以上的复杂任务。 |
| **slash command（斜杠命令）** | 以 `/` 开头的 Claude Code 命令，例如 `/nlpm:score`。command 文件（`.md`）的 frontmatter 中的 `name:` 字段决定了斜杠命令的名称。缺少 `name:` 字段的 command 文件无法在 slash command 调色板中显示，对用户完全不可见——这是 wshobson B-05~B-19 的核心问题。 |
| **manifest-discipline（清单纪律）** | 确保所有 NL artifact 文件的 frontmatter 字段完整、准确、符合规范的习惯。具体包括：每个 agent/command 都有 `name:`、`description:`、`allowed-tools:`，model tier 与任务复杂度匹配，没有重复文件。wshobson/agents 案例证明：即使是 33K Star 的高关注度仓库，缺乏 manifest-discipline（没有 CI 门控）时也会出现 19 个注册失败 bug。 |
