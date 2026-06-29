# softaworks/agent-toolkit — 学习案例

**仓库**：https://github.com/softaworks/agent-toolkit
**Stars**：1629 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-29（目标仓库不可访问，仅基于 audit 快照）
**主题标签**：`examples-driven`, `manifest-discipline`, `single-purpose`, `vague-quantifier`, `template-design`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
softaworks/agent-toolkit 是一套通用 AI agent 工具集，覆盖软件开发全流程（代码分析、文档、UI/UX、数据库设计、沟通协作等），包含 6 个 agent、9 个 command、43 个 skill，共 58 个制品。该仓库最大的特色是 **skill 质量极高（平均 95.1 分）**，但命令质量极低（平均 67 分），形成显著的内部不均衡。

关键事实：
- 58 个制品（6 agents, 9 commands, 43 skills）
- NL 评分 90/100（安全 PASS，2 Medium 告警）
- Skill 平均分 95.1，Agent 平均分 90.2，Command 平均分 67.0——三类制品质量差距极大
- 所有 9 个 command 都缺少 `name:` 字段，是系统性问题（非个别疏漏）
- `dist/plugins/` 目录包含所有 skill/agent/command 的精确副本（分发用）

### 1.2 架构剖析
- **目录结构**：
```
agent-toolkit/
├── .claude/
│   └── commands/
│       ├── add-skill.md           ← 缺 name（Bug #2）
│       └── sync-skills-readme.md  ← 缺 name（Bug #2）
├── agents/
│   ├── ascii-ui-mockup-generator.md   # 98分，顶尖设计
│   ├── codebase-pattern-finder.md     # 95分
│   ├── communication-excellence-coach.md # 89分
│   ├── general-purpose.md             # 79分，缺 examples
│   ├── mermaid-diagram-specialist.md  # 89分，结构像 skill 非 agent
│   └── ui-ux-designer.md             # 91分
├── commands/
│   ├── codex-plan.md     ← AskUser 非标工具名（Bug #6）
│   ├── compose-email.md
│   ├── explain-changes-mental-model.md
│   ├── explain-pr-changes.md  ← 零 frontmatter（Bug #1）
│   ├── sync-branch.md
│   ├── sync-skills-readme.md
│   └── viral-tweet.md    ← 用 section headers 而非编号步骤（Bug #5）
├── skills/
│   ├── humanizer/SKILL.md     # 98分，24组 Before/After 示例
│   ├── mermaid-diagrams/SKILL.md  # 98分
│   ├── react-dev/SKILL.md     # 98分，大量 TypeScript 示例
│   ├── draw-io/SKILL.md       # 98分
│   ├── excalidraw/SKILL.md    # 98分
│   ├── naming-analyzer/SKILL.md  # 98分，多语言示例
│   ├── daily-meeting-update/SKILL.md  # 97分
│   └── ... (43 个)
├── scripts/
│   ├── build_plugins.py
│   └── bump_version.py
└── dist/plugins/   # 分发副本，与 skills/agents/commands 完全同步
```
- **文件类型分布**：43 个 SKILL.md，6 个 agent.md，9 个 command.md，14 个脚本（.sh + .py），分发副本目录
- **编排关系**：Skill 完全平铺，通过 `references/` 子目录提供深度内容（渐进披露模式）；agent 与 skill 功能有重叠（mermaid-diagram-specialist agent vs mermaid-diagrams skill）
- **跨件契约**：`dist/plugins/` 是与 `skills/` + `agents/` + `commands/` 的同步副本，通过 `scripts/build_plugins.py` 构建，audit 确认两者完全一致，无漂移

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「工具即文档，示例即合同」——每个高分 skill 都有大量的 Before/After 示例或 Input→Output 样本，让模型通过示例理解期望行为而非靠抽象描述猜测
- **解决什么问题**：通用 AI agent 的模糊性——通过工具化（tool-ified）的各类 skill 把常见开发任务变成可重复的标准化流程
- **做了什么 trade-off**：深度覆盖 skill，浅显处理 command——43 个精心打磨的 skill 和 6 个高质量 agent，但 9 个 command 显然是后期草草添加的，质量参差不齐
- **反映什么认知模型**：作者把 skill 视为「专业能力模块」，每个 skill 对应一种可量化的专业产出（24 组 AI 文本 Before/After 示例、完整的 OpenAPI→TypeScript 转换例子、100 分清晰度评分量表等）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「渐进披露 Skill 库：高密度 Examples + 引用文件分层」**

核心特征是 skill 本文件提供核心概念 + 关键示例，深度内容放在 `references/` 子目录，形成渐进披露结构。读者/模型按需深入，无需一次性加载所有内容。

模式特征清单：
- 特征 1：每个 skill 的正文都有丰富的 Before/After 或 Input→Output 示例（最高 24 组）
- 特征 2：深度内容通过 `references/` 子目录分层，SKILL.md 保持精简
- 特征 3：Skill 覆盖领域极广（代码、UI、文档、沟通、数据库），但每个 skill 聚焦单一任务
- 特征 4：`dist/plugins/` 作为分发层，通过构建脚本与源码保持同步
- 特征 5：顶尖 skill 有明确的 Output Format section（输出格式规范），而非靠模型猜测输出形式

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 通用开发工具集（广度覆盖）| ✅ 高度适用 | 43 个 skill 的宽度覆盖大多数开发场景 |
| 专一领域深度工具（如 Go 专家库）| ❌ 较不适用 | 渐进披露架构擅长宽度，不适合单领域极深的覆盖 |
| 团队共享标准化流程 | ✅ 适用 | `dist/plugins/` 分发机制适合多人团队统一安装 |
| 需要命令行 workflow 的场景 | ⚠️ 谨慎 | Command 层质量（67分）明显弱于 Skill 层，需先修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：渐进披露 Skill 库 | softaworks/agent-toolkit | Examples 密度高、skill 质量好、分发机制完整 | Command 层系统性缺陷、agent/skill 重叠 |
| 单领域深度技术栈 | samber/cc-skills-golang | 领域覆盖深、skill 集群设计好 | 覆盖面窄、无 command 层 |
| DevSecOps 全家桶 | sangrokjung/claude-forge | 安全内建、验证引擎强 | 体量大、维护成本高 |

### 2.4 改进空间
1. **当前问题**：所有 9 个 command 缺 `name:` 字段，这是系统性问题。**改进做法**：批量修复——每个 command 的 frontmatter 第一行加 `name: <slug-from-filename>`。**预期收益**：Command 整体分可从 67 升到约 85，项目整体分可从 90 升到约 93
2. **当前问题**：`mermaid-diagram-specialist` agent 用了 SKILL.md 式的字段（name/description/category/usage/input/output），而不是 agent 应有的字段（model/tools/description）。**改进做法**：重写其 frontmatter 为标准 agent 格式，或删除此 agent、将其流程整合到 `skills/mermaid-diagrams/SKILL.md`。**预期收益**：消除 agent/skill 重叠，让用户知道该用哪个

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 90/100（58 制品加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/explain-pr-changes.md | 45 | 零 frontmatter，不可注册 |
| commands/viral-tweet.md | 60 | 无 name，用主题 header 代替编号步骤 |
| .claude/commands/add-skill.md | 70 | 无 name，无 allowed-tools |
| agents/general-purpose.md | 79 | 零 examples，3 个模糊量词 |
| skills/humanizer/SKILL.md | 98 | 顶尖：24 组 Before/After AI 模式示例 |
| skills/mermaid-diagrams/SKILL.md | 98 | 顶尖：四种图类型快速入门示例 |
| skills/react-dev/SKILL.md | 98 | 顶尖：大量 TypeScript + React 19 示例 |
| agents/ascii-ui-mockup-generator.md | 98 | 顶尖：5 步流程，description 内嵌 2 个示例 |

### 3.2 当时值得借鉴的模式

1. **24 组 Before/After AI 文本示例（humanizer skill）** → 为什么好：每组示例都是「AI 原文 → 人性化改写」的对比，让模型精确理解「人性化」的含义，而不是猜测 → 路径：skills/humanizer/SKILL.md → 借鉴：任何涉及「改写/优化」的 skill 都应提供至少 5 组 Before/After 示例，而非靠自然语言描述期望效果

2. **Output Format 专节（规范化输出契约）** → 为什么好：高分 skill（如 backend-to-frontend-handoff-docs、qa-test-planner）有专门的 `## Output Format` 节，精确定义产出物的结构，防止模型自由发挥 → 路径：skills/qa-test-planner/SKILL.md → 借鉴：在所有产出物有固定格式的 skill 里加 Output Format 节

3. **渐进披露：SKILL.md + references/ 分层** → 为什么好：skill 正文简洁，深度内容（benchmark docs、eval references、security compliance reference）放 references/ 子目录，模型按需加载，避免正文臃肿 → 路径：commands/evaluating-llms-harness/ → 借鉴：超过 500 行的 SKILL.md 应考虑把深度内容拆到 references/

4. **description 内嵌 mini-examples（ascii-ui-mockup-generator）** → 为什么好：agent 的 description 字段本身就包含了两个 mini 示例，用户在 `/help` 中就能看到典型用例，不需要打开完整文档 → 路径：agents/ascii-ui-mockup-generator.md 的 description 字段 → 借鉴：高频使用的 agent 应在 description 里内嵌 1-2 个极简示例

5. **100 分量化评分量表（requirements-clarity skill）** → 为什么好：skill 用 100 分量表评估需求清晰度，把「是否清晰」这个主观判断转化为可量化、可重复的评分，防止模型给出模糊的「需求不够清晰」反馈 → 路径：skills/requirements-clarity/SKILL.md → 借鉴：评估类 skill 应提供量化评分标准而非形容词判断

### 3.3 当时的缺陷

1. **所有 9 个 command 缺 name: 字段** → 根本原因：这是系统性的遗漏，说明作者在开发 skill 时非常细心（skill 全部有完整 frontmatter），但对 command 的 frontmatter 约定认知不足，可能认为 Claude Code 会自动从文件名推导 name。事实上 name 字段决定了命令的可发现性（`/help` 显示）和注册标识。9/9 命令都缺 name 意味着所有命令可能都无法可靠注册 → 自查：我的 command 文件是否每个都有 `name:` 字段？

2. **mermaid-diagram-specialist agent 用了 SKILL 结构** → 根本原因：作者在写这个 agent 时用了 SKILL.md 的 frontmatter 格式（name/description/category/usage/input/output），但 agent 应该使用 agent 格式（model/tools/description/...）。两套格式的字段名称不同，混用导致 agent 加载器无法识别 model 和 tools 声明，agent 将使用默认 model 和无工具限制 → 与同仓库的 skills/mermaid-diagrams/SKILL.md 功能重叠，让用户迷惑 → 自查：我的 agent 文件是否误用了 SKILL 格式的 frontmatter 字段？

3. **AskUser 非标工具名（codex-plan.md）** → 根本原因：Claude Code 的正确工具名是 `AskUserQuestion`，但 codex-plan 的 allowed-tools 写了 `AskUser`。这类错误很隐蔽，因为：(a) 文件语法是合法的 YAML；(b) 工具权限检查只在实际调用时失败；(c) 失败时是静默的（工具调用被 sandbox 拒绝，但不显示错误）→ 自查：我的 allowed-tools 中有没有使用了错误的工具名？（常见错误：AskUser vs AskUserQuestion，Task vs TaskCreate）

### 3.4 当时的优化机会

1. **批量修复 9 个 command 的 name 字段**：直接收益最大（可把 command 层分从 67 升至 85+），且修法极简（每个文件加一行 `name: xxx`）
2. **viral-tweet.md 改为编号步骤**：该命令是唯一不用编号步骤的 command，与同仓库其他命令风格不一致，模型执行时无法检测步骤失败
3. **agents/general-purpose.md 加 examples 节**：这是 agent 层分最低的文件，加 2 个 worked example 可把分从 79 升到 89+

---

## 四、现在 vs 过去对比

> 目标仓库（softaworks/agent-toolkit）在本运行环境中无法访问（HTTP 403），以下分析基于 audit 快照。

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 所有 command 缺 name 字段 | `grep -L 'name:' commands/*.md` | 无法验证（仓库不可访问） | 若未修复，9 个命令可能持续在注册稳定性上有隐患 |
| AskUser 非标工具名 | `grep 'AskUser' commands/codex-plan.md` | 无法验证 | 若未修复，codex-plan 的用户交互步骤静默跳过 |
| mermaid agent 用 SKILL 格式 | `grep 'model:' agents/mermaid-diagram-specialist.md` | 无法验证 | 若未修复，该 agent 无 model 限制 |

### 4.2 架构演进
无法从当前 HEAD 对比（仓库不可访问）。从 audit 数据看，skills 层（平均 95.1 分）已相当成熟，预期重心会放在补全 command 层。

### 4.3 新增的可学习模式
暂无（无法访问当前 HEAD）。

---

## 五、校准

### 5.1 我已经在做对的
1. **Command 文件有 name 字段**：我的命令文件都有显式的 `name:` 声明
2. **使用正确的工具名**：`AskUserQuestion`（不是 `AskUser`），`TaskCreate`（不是 `Task`）
3. **高复杂度 skill 分层存储**：MarkQWu/graphify 的部分 skill 也使用了 references/ 子目录模式

### 5.2 挑战 / 验证
**被这次案例挑战的假设**：「整体高分 = 各部分均衡」。softaworks/agent-toolkit 整体 90/100，但 command 平均 67 分，skill 平均 95 分，内部差距高达 28 分。这说明 NLPM 的整体分是加权平均，高分 skill 会掩盖低分 command。学习时不应只看总分，要看各类制品的分布。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查我的命令文件是否每个都有 name: 字段
grep -rL '^name:' ~/.claude/commands/*.md 2>/dev/null
# 命中后：在 frontmatter 的 --- 块内第一行加 name: <slug>

# 2. 检查是否误用了非标工具名
grep -rn 'AskUser\b\|TaskCreate\b\|WriteFile\b\|ReadFile\b' ~/.claude/commands/*.md ~/.claude/agents/*.md 2>/dev/null \
  | grep -v 'AskUserQuestion\|TaskCreate is'
# 命中后：对照 Claude Code 工具名文档修正拼写

# 3. 检查 skill 正文是否有 Output Format 节（高分 skill 都有）
grep -rL '## Output\|## 输出格式\|## Output Format' ~/.claude/skills/*/SKILL.md 2>/dev/null | head -10
# 命中后：在 skill 末尾加 ## Output Format 节，用模板或示例定义期望输出

# 4. 检查 agent 文件是否有 ## Examples 节
grep -rL '## Examples\|## 示例' ~/.claude/agents/*.md 2>/dev/null
# 命中后：加至少 2 个 worked example（用户输入 → agent 产出）

# 5. 找出可能误用了 SKILL 格式的 agent 文件（有 category/usage/input 字段的 agent）
grep -rn 'category:\|usage:\|input:' ~/.claude/agents/*.md 2>/dev/null
# 命中后：确认是否是 agent 格式还是 skill 格式，统一使用正确格式
```

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/bureau 的核心 skill 加 Before/After 示例
   - **为何可行**：bureau 的 `compile` 和 `review` skill 描述了行为期望，但没有具体示例，导致模型执行时依赖自身理解而非确定性规范
   - **第一步**：在 skills/compile/SKILL.md 的 `## Examples` 节加 3 组「输入 session 片段 → 编译后知识条目」示例，约 20 分钟

2. **想法**：为 MarkQWu/graphify 的 skill 加 `## Output Format` 节
   - **为何可行**：graphify 的产出物（知识图谱查询结果、节点关系图）有固定格式，但当前 skill 未声明期望输出格式，导致每次产出结构不一致
   - **第一步**：在 skills/query-graph/SKILL.md 加 `## Output Format` 节，用 JSON schema 或模板定义图谱输出格式，约 15 分钟

3. **想法**：把 drama-workshop-skills 的 skill 分值从中等提升到高分（借鉴 softaworks 的 98 分 skill 特征）
   - **为何可行**：98 分 skill 的三大特征是：(a) 丰富 examples；(b) 明确 Output Format；(c) 消除模糊量词。这三点都是机械性的内容补充，不需要重新设计架构
   - **第一步**：选一个当前分数最低的 skill，按三大特征逐步补充内容，约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 注：本运行环境中无法访问外部 GitHub 仓库（HTTP 403），以下分析基于 `learning/my-repos.json` 元数据和已知结构。

### 8.1 目的对齐度

- **本案例 softaworks/agent-toolkit 的核心目的**：提供覆盖软件开发全流程的高质量通用 AI agent 工具集，以丰富 skill 示例和规范输出格式为核心竞争力

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多功能 Claude Code 工具集，覆盖多种开发角色/流程 | gstack 按角色（CEO/PM/Eng Manager）组织，agent-toolkit 按能力（UI/UX/代码/沟通）组织 | 高 |
| MarkQWu/drama-workshop-skills | 高 | 都是 skill 集合，面向特定社区 | drama-workshop-skills 偏领域垂直，agent-toolkit 偏通用 | 高 |
| MarkQWu/graphify | 中 | 都是包含多个 skill 的 Claude Code plugin | graphify 是工具型（知识图谱生成），agent-toolkit 是通用型（各类开发任务） | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 name 字段 | `grep -L '^name:' commands/*.md` | gstack 有 23 个工具命令，需逐一核查 | 高 |
| Agent 缺 Examples 节 | `grep -L '## Examples' agents/*.md` | 无法验证，但 bureau 和 gstack 的 agent 存在风险 | 中 |
| Skill 缺 Output Format | `grep -L 'Output Format\|输出格式' skills/*/SKILL.md` | 无法验证，graphify 的 skill 存在此风险 | 中 |

**命中后的具体行动建议**：
- MarkQWu/gstack 的 23 个角色命令文件 → 逐一加 `name:` 字段 → 2 分钟/文件，共约 46 分钟
- MarkQWu/drama-workshop-skills 的所有 SKILL.md → 加 `## Output Format` 节 → 10 分钟/文件

### 8.3 别人的更优方案

1. **领域**：Examples 密度（Before/After 示例）
   - **本案例做法**：skills/humanizer/SKILL.md 有 24 组 Before/After 示例（AI 原文 → 人性化改写），每组 2-3 句话，简洁且密集
   - **我的项目现状**：MarkQWu/drama-workshop-skills 的 skill 描述居多，examples 部分较少或较弱
   - **如何借鉴**：选核心 skill，整理 5-10 组真实的 Input→Output 对，加入 `## Examples` 节；每组约 2 分钟整理，总计约 20 分钟

2. **领域**：量化评分标准
   - **本案例做法**：skills/requirements-clarity/SKILL.md 有 100 分量化评分表，把「是否清晰」转成数字
   - **我的项目现状**：MarkQWu/bureau 的「review」步骤用形容词（「是否清晰」「是否准确」）而非数字判断质量
   - **如何借鉴**：在 bureau 的 review skill 中加一个 0-100 分的知识质量评分量表，定义每个分段的含义；约 30 分钟

### 8.4 反向：我的项目做得比他们好的地方

本案例未发现我的项目更优的维度（softaworks/agent-toolkit 的 skill 质量普遍高于我的当前实现；command 层虽弱，但这也是我的薄弱点）。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明 skill/command/agent 的注册元数据。`name:` 字段决定命令的唯一标识符（斜杠命令名），`allowed-tools:` 声明命令可以使用哪些工具，`description:` 提供 `/help` 中显示的说明文本。

### <a name="渐进披露"></a>渐进披露（Progressive Disclosure）
> 设计模式：把内容按深浅分层呈现。在 skill 设计中，SKILL.md 是第一层（核心逻辑 + 关键示例），`references/` 子目录是第二层（深度文档），用户/模型按需深入。避免把所有内容堆在 SKILL.md 造成「信息轰炸」。

### <a name="output-format"></a>Output Format（输出格式节）
> skill 中的专门章节，精确定义产出物的格式——是 JSON、Markdown 表格、还是代码块？有哪些必填字段？遵循什么命名约定？有了 Output Format 节，模型每次产出的结构可预期，便于自动化处理。

### <a name="allowed-tools"></a>allowed-tools
> Command/Agent frontmatter 中声明该制品可以使用的 Claude Code 工具列表。如 `allowed-tools: [Bash, Read, Edit]`。在受限权限模式下，未声明的工具调用会被 sandbox 拒绝（静默跳过或报错）。工具名必须精确匹配 Claude Code 内部名称（如 `AskUserQuestion` 而非 `AskUser`）。
