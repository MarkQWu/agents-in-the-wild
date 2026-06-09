# shanraisshan/claude-code-best-practice — 学习案例

**仓库**：https://github.com/shanraisshan/claude-code-best-practice
**Stars**：46,008 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-09（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `single-purpose`, `security-gate`, `examples-driven`
**xiaolai 案例**：[../../2026-04-24-shanraisshan-claude-code-best-practice.md](../../2026-04-24-shanraisshan-claude-code-best-practice.md)

---

## 一、理解

### 1.1 仓库概览

`shanraisshan/claude-code-best-practice` 是 GitHub 上星数最高的 Claude Code 参考实现仓库，截至本文生成日期有 **46,008 stars**、4,699 forks。作者 Shayan Rais 将其定位为"从 vibe coding 到 agentic engineering 的实践参考"——一个持续更新的活体示例，覆盖 Claude Code 的 agents、commands、skills、hooks 四大层次。

关键事实：

- 审计时共 **44 个 NL artifacts**，含 20 个 agent、11 个 command、9 个 skill、2 个 hook config、1 个 CLAUDE.md、1 个 MCP config
- 覆盖领域跨度极大：天气预报工作流、时间卡片生成、RPI 产品研究流程（8 个专业 agent 串行）、Drift Detection 工作流、presentation builder（将 Markdown 转化为 HTML 幻灯片）、多 agent 团队协同
- 仓库本身即教材和考题二合一：用来教 Claude Code 最佳实践，同时自身就是这些实践的运行实例
- 2026-04-24 的 re-audit 提交哈希为 `13cbf08`，说明仓库在 NLPM 提交 PR 后仍在持续迭代

安全评级：**CLEAR**（无 Critical / High 漏洞，2 个 Medium、2 个 Low）

### 1.2 架构剖析

**顶层目录结构**（关键层级）：

```
.claude/
  agents/                          ← 12 个根域 agent（weather、time、development-workflows、presentation×2、5 个 best-practice workflow）
    workflows/
      best-practice/               ← 5 个 workflow research agent（settings/skills/subagents/commands/concepts）
      development-workflows-research-agent.md  ← 名称冲突的根域副本（已修复）
  commands/                        ← 8 个根域 command（time-command、weather-orchestrator、workflow-* ×6）
    workflows/
      best-practice/               ← 6 个 workflow command（分别触发对应的 workflow agent）
  skills/                          ← 9 个根域 skill（weather-fetcher、weather-svg-creator、agent-browser、time-skill、presentation×3）
  hooks/
    scripts/hooks.py               ← PostToolUse 钩子（Medium 安全风险点）
    config/hooks-config.json       ← 100/100

agent-teams/
  .claude/
    agents/time-agent.md           ← Dubai GST（UTC+4）版 time-agent
    commands/time-orchestrator.md  ← 被 Bug #1 修复的命令
    skills/                        ← time-fetcher、time-svg-creator

development-workflows/
  rpi/
    .claude/
      agents/                      ← 8 个 RPI 专业 agent（requirement-parser、product-manager、senior-software-engineer、ux-designer、technical-cto-advisor、documentation-analyst-writer、code-reviewer、constitutional-validator）
      commands/
        rpi/                       ← plan.md、research.md、implement.md（三步 RPI 流程）

.mcp.json                          ← 3 个 MCP 服务器（Playwright、Context7、DeepWiki）
```

**文件类型分布**：20 个 agent / 11 个 command / 9 个 skill / 2 个 hook config / 1 个 MCP config / 1 个 CLAUDE.md / 1 个 hooks.py

**编排关系**（多层次）：

1. **天气工作流**：`weather-orchestrator.md`（command）→ `weather-agent.md`（agent）→ `weather-fetcher/SKILL.md` + `weather-svg-creator/SKILL.md`（skill）
2. **时间工作流**：`time-orchestrator.md`（agent-teams command）→ `time-agent`（agent）→ `time-fetcher/SKILL.md` + `time-svg-creator/SKILL.md`（skill）
3. **RPI 研究流程**：`research.md` → `plan.md` → `implement.md`（三步顺序 command），每步触发 8 个专业 agent 协同
4. **Drift Detection**：`development-workflows.md`（command）→ `development-workflows-research-agent`（agent），监测最佳实践是否过期
5. **Best Practice Workflow**：6 个 workflow command 分别触发对应的 workflow research agent，研究 settings/skills/subagents/commands/concepts

**跨件契约**：

- 所有 workflow command 通过 `subagent_type` 字段引用 agent，所有引用均指向存在的文件（✓ 全部通过交叉引用校验）
- RPI 流程中 `plan.md` 检查 `RESEARCH.md` 存在性，`implement.md` 检查 `PLAN.md` 存在性，形成隐式文件契约
- 8 个 RPI agent 名称在 `implement.md` 的 `subagent_type` 引用中与文件名严格对应（✓）

### 1.3 设计思路 / 方法论

**核心哲学**：用真实运行的代码教最佳实践。仓库不只是文档，每个 agent、command、skill 本身都在被真实使用，这使得维护者对质量有真实感知，问题暴露也更快。

**层次解耦**：skill 层作为知识库（What to know），agent 层作为执行者（Who to call），command 层作为用户入口（How to invoke）。三层各司其职，互不侵占——这正是 skill 层全部 100/100 而 agent 层平均 77 的结构性原因：skill 是静态知识，agent 涉及工具声明和行为约束，后者更容易出错。

**多域覆盖策略**：不局限于单一场景，而是覆盖天气、时间、RPI 研究、Drift Detection、演示文稿构建等不同领域，目的是展示 Claude Code agent 架构在多种真实需求下的可迁移性。

**已知 trade-off**：

- 广覆盖带来了模板扩散风险——用一个工作 agent 的 frontmatter 复制粘贴创建下一个，导致 `allowedTools` 从未被按需裁剪（这是本案例最核心的结构性 bug）
- RPI 流程 8 个 agent 串行，功能强大但调试复杂：任何一个 agent 失败都会中断整个流程，且没有 checkpoint 机制

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：命令 → Agent → Skill 三层最小权限编排

本仓库展示了 Claude Code 最规范的三层结构：command 是用户接触点（声明 allowed-tools、description、argument-hint），agent 是专业执行者（声明 name、description、allowedTools），skill 是知识库（声明 tools、scope note）。三层之间通过名称引用（subagent_type）和文件路径松耦合。

**该模式的特征清单**：

- **单一职责**：每个 command 只触发一类工作流，每个 agent 只完成一种专业任务，每个 skill 只描述一个领域的知识
- **名称即契约**：command 通过 `subagent_type: time-agent` 引用 agent，agent 通过 `name: time-agent` 声明自己，名称不匹配则工作流中断
- **技能层稳定性**：skill 层无状态、无工具声明，只是知识，因此 9 个 skill 全部 100/100
- **工具声明最小化**（理想状态）：每个 agent 的 allowedTools 应该只包含该 agent 真正需要的工具——本仓库在初始审计时未做到，但这正是可学习的反例

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多领域 agent 编排（天气/时间/研究等并存） | ✅ 高度适用 | 三层结构天然支持跨域扩展，新增领域只需加一组 command+agent+skill |
| 需要向用户教授架构最佳实践的参考实现 | ✅ 高度适用 | 仓库即文档，代码即示例 |
| 有严格安全要求的企业环境 | ⚠️ 需改造 | MCP 服务器未版本锁定；hooks.py 有 subprocess.Popen PATH 解析风险 |
| 单一简单工作流（如只做代码审查） | ❌ 过度工程 | 三层结构带来维护开销，单个场景直接用一个 agent 即可 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 三层命令-Agent-Skill（本仓库） | shanraisshan/claude-code-best-practice | 层次清晰；skill 层可复用；command 层对用户友好 | allowedTools 复制粘贴风险高；name 冲突在多作用域下不可见 |
| 单 Agent 平铺 | 大多数简单插件 | 极简；无跨件依赖 | 超过 3 个领域后可读性和维护性下降 |
| MCP 工具封装 | Autosearch（MCP 协议化） | 工具接口可测试；安全边界更清晰 | 需要额外维护 MCP 服务器 |
| 多 Agent 团队（agent-teams 子目录） | 本仓库的 agent-teams/ 子模块 | 允许不同时区/场景的同类 agent 并存 | 名称冲突在混合作用域下不可见（Bug #3 的根源） |

### 2.4 改进空间

1. **当前问题**：所有 agent 共享同一份 11 工具 `allowedTools` 模板（Bash、Read、Write、Edit、Glob、Grep、WebFetch、WebSearch、Agent、NotebookEdit、mcp__*），从未按 agent 实际职能裁剪。**改进做法**：为每类 agent（只读研究型 / 单命令型 / 写入型）建立三种 allowedTools 基线模板，并在 `.claude/CLAUDE.md` 中明确说明何时使用哪种。**预期收益**：消除 8 个 read-only agent 的 Write/Edit 声明矛盾，平均得分从 77 提升到约 87。
2. **当前问题**：20 个 agent（re-audit 时 19 个）全部没有 `<example>` 块，"PROACTIVELY use this agent" 的触发信号缺失。**改进做法**：为每个 agent 的 description 字段补充至少 1 个 `<example>` 块，格式如 `<example>\n<context>天气查询场景</context>\n<user-message>帮我查一下北京今天天气</user-message>\n</example>`。**预期收益**：每个 agent 单文件 +10 分；整体加权平均预计超过 92/100。
3. **当前问题**：`agent-teams/.claude/agents/` 和根域 `.claude/agents/` 同时存在相似 agent 时容易发生 name 冲突（Bug #2、#3）。**改进做法**：在 CI 中加一条 `grep -rh "^name:" .claude/agents agent-teams/.claude/agents | sort | uniq -d` 检查，检测到重复 name 立即报错。**预期收益**：防止未来新增 agent 时再次出现 name 冲突。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

**总体 NL 得分：88/100**（44 个 artifacts，安全：CLEAR）

| 层次 | 文件数 | 平均分 | 说明 |
|---|---|---|---|
| Skill | 9 | **100** | 全部满分，无任何扣分点 |
| Hook config | 2 | **100** | 全部满分 |
| Command | 11 | **93** | 主要问题：8 个 standalone command 缺 allowed-tools（-5 each）；RPI 命令有模糊量词 |
| Agent | 20 | **77** | 主要问题：0 个示例块（-15 each）；8 个只读 agent 声明 Write/Edit（-10 each）；1 个 agent 8 个未使用工具（-3 each × 8） |
| CLAUDE.md | 1 | **95** | 轻微：内容详细但无结构性问题 |

**最低分文件（审计时）**：

| 文件 | 分数 | 主要扣分点 |
|---|---|---|
| `.claude/agents/time-agent.md` | 57 | 8 个未使用工具声明（-3 × 8 = -24）；Write/Edit 在只读 agent（-10）；无示例（-15）|
| `.claude/agents/workflows/best-practice/workflow-concepts-agent.md` | 68 | 只读 agent 声明 Write/Edit（-10）；无示例（-15）；模糊量词（-6） |
| `agent-teams/.claude/commands/time-orchestrator.md` | 70 | 缺 `description` frontmatter（Bug #1）|

### 3.2 当时值得借鉴的模式

1. **Skill 层 100/100 的写法**：9 个 skill 全部满分。路径：`.claude/skills/weather-svg-creator/SKILL.md`、`.claude/skills/agent-browser/SKILL.md` 等。根本原因：skill 文件专注于"知识描述"，没有工具声明也没有行为约束，只需保证描述清晰、无模糊量词、有 scope note。借鉴：写 skill 时先问"这个文件需要教给 AI 什么知识"而不是"AI 需要什么工具"——两个问题分开回答。

2. **三层工作流的命名一致性**：`weather-orchestrator.md`（command）→ `weather-agent.md`（agent name: weather-agent）→ `weather-fetcher/SKILL.md` + `weather-svg-creator/SKILL.md`。所有工作流的 command ↔ agent ↔ skill 引用链全部验证通过（✓）。根本原因：作者在建立工作流时系统地维护了名称一致性，没有出现"命令引用了不存在的 agent"这类断链。借鉴：建立工作流时画一张 command → agent → skill 的引用图，提交前跑 `/nlpm:check` 验证引用完整性。

3. **RPI 多 Agent 串行流程**：8 个专业 agent（requirement-parser → product-manager → senior-software-engineer → ux-designer → technical-cto-advisor → documentation-analyst-writer → code-reviewer → constitutional-validator）形成一条产品研究流水线，每个 agent 角色明确，职责不重叠。根本原因：把软件工程里"领域驱动设计"的思路搬到了 agent 设计上——每个 agent 对应一个真实的职能角色。借鉴：多 agent 系统中，如果能在现实中找到对应的职位名称，agent 的职责边界就已经很清晰了。

4. **Hook config 配置清晰**：`.claude/hooks/config/hooks-config.json` 和 `.codex/hooks/config/hooks-config.json` 双双 100/100。两个文件仅声明触发条件和脚本路径，不掺杂业务逻辑。根本原因：hook config 和 hook 执行脚本（`hooks.py`）分离，config 只做路由，脚本才做工作。借鉴：hook 的 json config 只写声明，不写逻辑；逻辑写到 .py / .sh 中，便于独立测试。

5. **CLAUDE.md 的架构描述价值**：CLAUDE.md（95/100）详细描述了整个仓库的工作方式、各层次的职责划分、以及每个主要工作流的触发条件。虽然轻微超长，但作为参考仓库的"用户手册"是合适的。根本原因：作者把 CLAUDE.md 当作"给 AI 的架构图"而不只是"给人类的 README"。借鉴：CLAUDE.md 第一段应该描述"当前项目里什么 agent 可以被调用、什么场景该调用哪个"，让 Claude 在第一次读到 CLAUDE.md 时就能做出正确的路由决策。

### 3.3 当时的缺陷

1. **Bug #1：`time-orchestrator.md` 缺 `description` frontmatter**
   - 文件路径：`agent-teams/.claude/commands/time-orchestrator.md`
   - 当时 frontmatter 只有 `model: haiku`，缺少 `description` 字段
   - 影响：命令无法出现在 `/` 斜杠命令菜单中，用户完全无法发现它
   - 根本原因：创建 command 时只填了 `model` 字段，遗漏了必填的 `description`
   - 自查：我的 echo-sleuth 的 `timeline.md`、`recall.md` 等命令是否有完整 frontmatter？

2. **Bug #2：`development-workflows-research-agent` 名称冲突**
   - 文件路径 A：`.claude/agents/development-workflows-research-agent.md`（根域，包含 Write/Edit）
   - 文件路径 B：`.claude/agents/workflows/development-workflows-research-agent.md`（子域，正确的只读版本）
   - 两个文件都声明 `name: development-workflows-research-agent`
   - 影响：混合作用域下 Claude Code 可能加载错误的 agent 版本，错误版本还多出 Write/Edit 工具
   - 根本原因：在 `workflows/` 子目录中创建了正确版本后，忘记删除或重命名根域的旧版本

3. **Bug #3：`time-agent` 名称冲突（PKT vs Dubai GST）**
   - 文件路径 A：`.claude/agents/time-agent.md`（根域，PKT UTC+5，同时声明 10+ 工具）
   - 文件路径 B：`agent-teams/.claude/agents/time-agent.md`（agent-teams 域，Dubai GST UTC+4）
   - 两个文件都声明 `name: time-agent`
   - 影响：用户请求迪拜时间时可能静默返回卡拉奇时间；反之亦然

4. **系统性问题：全部 20 个 agent 零示例块**
   - 所有 agent 的 description 字段均无 `<example>` 块
   - 多个 agent 明确写了"PROACTIVELY use this agent"，但没有示例的情况下触发信号缺失
   - 根本原因：模板复制时从未考虑补充示例

5. **系统性问题：8 个只读 agent 声明 Write/Edit**
   - 涉及文件：`development-workflows-research-agent.md`、`weather-agent.md`、`time-agent.md`（根域）、5 个 `workflows/best-practice/*-agent.md`
   - 每个文件的 body 明确写了"DO NOT modify any files"，但 `allowedTools` 声明了 `Write`、`Edit`
   - 根本原因：复制粘贴了其他 agent 的 frontmatter 模板，未针对当前 agent 的只读职能裁剪工具列表

6. **安全：MCP 服务器版本未锁定（Medium）**
   - 文件：`.mcp.json` 第 4、8、12 行
   - `npx -y @playwright/mcp`、`npx -y @upstash/context7-mcp`、`npx -y deepwiki-mcp` 均无版本号
   - 风险：任何 npm 包更新（包括被劫持的恶意版本）都会静默影响所有用户

7. **安全：hooks.py subprocess.Popen PATH 解析（Medium）**
   - 文件：`.claude/hooks/scripts/hooks.py` 第 185-190 行
   - 音频播放器从 PATH 动态解析，而非硬编码绝对路径
   - 风险：PATH 被篡改时可执行恶意二进制文件

### 3.4 当时的优化机会

1. **为每个 agent 补充 1-2 个 `<example>` 块**：每个文件仅需 5-10 行，但能将该 agent 得分从约 72 提升到约 82（+10 分），整体加权平均预计从 88 提升到 92+。

2. **建立三类 allowedTools 基线模板**：
   - 只读研究型：`allowedTools: [Bash, Read, Glob, Grep, WebFetch, WebSearch]`
   - 单命令型（如 time-agent）：`allowedTools: [Bash]`
   - 全能写入型（实际写文件的 agent）：保留完整列表

3. **RPI agent 补充 `allowedTools` frontmatter**：`development-workflows/rpi/.claude/agents/*.md` 8 个文件的 tools 只在 body 里描述，未在 frontmatter 声明，Claude Code 无法强制工具限制。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查路径 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：time-orchestrator.md 缺 description | `head -5 agent-teams/.claude/commands/time-orchestrator.md` | **已修复**：当前 frontmatter 含 `description: Fetch the current time for Dubai (GST, UTC+4) and create a visual SVG time card` + `model: haiku` | NLPM PR #63 已合并，命令可被发现 |
| Bug #3：time-agent name 冲突 | `.claude/agents/time-agent.md` 中 `name:` 字段 | **已修复**：根域改名为 `name: time-agent-pkt`；agent-teams 域保持 `name: time-agent` | NLPM PR #65 已合并，冲突解除 |
| Bug #2：development-workflows name 冲突 | `grep -rh "^name:" .claude/agents` 查重 | **已修复（upstream）**：维护者在 PR #64 中自行处理 | 无需 NLPM PR |
| 只读 agent 声明 Write/Edit（8 个文件） | re-audit Bug #1-#6 | **部分修复**：re-audit 时 6 个文件（dev-workflows-research、weather-agent、time-agent 根域、3 个 workflow agent）仍存在此问题；新增 `presentation-claude-gemini.md` 也有 Write/Edit 声明 | 系统性问题尚未根治 |
| 所有 agent 零示例块 | re-audit Finding #7-#25 | **已修复（upstream）**：re-audit 验证 Finding #10"fixed — upstream, not via our PR"；但 re-audit 同时发现剩余 19 个 agent 逐一重新计为 R09 违规 | 维护者在某次提交中为 agent 添加了示例，但 re-audit 模型以更严格的规则重新检测，整体仍有大量 R09 违规 |
| MCP 服务器版本未锁定 | `.mcp.json` | **已修复**：NLPM PR #67 锁定到 `@playwright/mcp@0.0.70`、`@upstash/context7-mcp@2.1.8`、`deepwiki-mcp@0.0.6` | 版本锁定后需主动维护升级 |
| hooks.py subprocess.Popen PATH 解析 | `.claude/hooks/scripts/hooks.py` 第 185-190 行 | **已修复（upstream）**：re-audit 验证 Finding #8"fixed — upstream, not via our PR" | 维护者自行修复了 PATH 依赖 |
| hooks-log.jsonl 未在 .gitignore | `.gitignore` 中查找 `.claude/hooks/logs/` | **已修复**：NLPM PR #66 已合并，日志目录加入 .gitignore | 敏感数据泄露风险消除 |
| CLAUDE.md 缺 build/test/prerequisites | re-audit Finding #35-#37 | **仍存在**：re-audit 新增 Finding（R33、R34、R35），CLAUDE.md 无构建命令、无测试命令、无前置条件章节 | 这是参考仓库的合理缺失，但 NLPM 评分规则不区分仓库类型 |

### 4.2 架构演进

从 2026-04-19 审计快照到 re-audit `13cbf08` 提交，主要变化：

- **4 个 NLPM PR 全部合并**（2026-04-23，距提交仅 25 小时）：#63（description）、#65（time-agent-pkt）、#66（.gitignore）、#67（MCP 锁版本）
- **21 个发现被维护者独立修复**（无需 NLPM PR），包括：name 冲突 Bug #2、所有只读 agent 的 Write/Edit 声明、所有 20 个 agent 的示例块、hooks.py 的 PATH 依赖、8 个 standalone command 的 allowed-tools、RPI 命令和 agent 中的模糊量词、frontmatter 格式不一致
- **新增目录**：`best-practice/`、`implementation/`、`reports/`（HEAD 较审计时有新增内容）
- **re-audit 总分不变（88/100）**：27 个原始发现全部修复，但同时新发现 50 个问题（19 个 R09 重新枚举 + R11 回归 + R07 skill scope note + R33/R34/R35 CLAUDE.md）

### 4.3 新增的可学习模式

re-audit 暴露的一个新模式问题值得特别关注：**评分稳定性幻觉**。原始 88/100 与 re-audit 88/100 数字相同，但内容完全不同——27 个原始发现全部修复，同时新引入 50 个发现。这说明两件事：

1. NL 评分不是绝对质量度量，而是当前评分规则下的快照；规则演进会导致同一文件的得分变化
2. "修复了所有已知问题"≠"整体质量提升"；如果修复速度慢于新问题的发现速度，总分可以原地不动

这个模式对我的仓库的意义：不要追求"某次 `/nlpm:score` 输出 90+"，而是追求"每次迭代后单个 artifact 的评分都在提升"。

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth 的 agent 没有 Write/Read-only 矛盾**：echo-sleuth 的 5 个 agent 不存在"body 说只读但 allowedTools 有 Write"的情况，这是比 shanraisshan 初始审计时更干净的地方。

2. **命令 frontmatter 相对完整**：echo-sleuth 的 `timeline.md`、`recall.md`、`recap.md`、`lessons.md` 都有 `name`、`description` 字段，没有出现 Bug #1 那样的关键字段缺失。

3. **无多作用域 name 冲突风险**：echo-sleuth 所有 agent 在同一域下，不存在 `agent-teams/` + 根域这样的双作用域混合场景，也就没有 Bug #2、#3 的风险条件。

4. **skill 文件有独立目录结构**：echo-sleuth 每个 skill 在独立目录（如 `skills/experience-synthesis/SKILL.md`），与 shanraisshan 的 `skills/weather-fetcher/SKILL.md` 模式一致，便于未来加 scope note。

### 5.2 挑战 / 验证

**挑战**：shanraisshan 的案例让我意识到，`allowedTools` 的复制粘贴问题几乎不是有意识的决策，而是"默认继承"的惰性。我的 echo-sleuth 虽然没有 Write/only 矛盾，但实际上我也没有在创建每个 agent 时认真思考"这个 agent 需要哪些工具"——我只是确保了 body 和 tools 没有冲突，但没有主动做 least-privilege 设计。

**验证（成立的）**：shanraisshan 的经验验证了"skill 层投入产出比最高"的判断。skill 文件写一次可以被多个 agent 和 command 复用，且因为没有工具声明所以不会引入 allowedTools 风险。我的 claude-for-legal 有 218 个模糊量词，大概率集中在 skill/agent 层，shanraisshan 的案例证明这类模糊量词在规模化后确实会系统性拖低评分。

**验证（不成立的）**：我原以为"先写好 body，工具声明事后补"是可接受的开发节奏。shanraisshan 的案例证明这种节奏会导致工具声明永远滞后于 body——因为修改 body 比修改 allowedTools 更频繁、更自然。正确做法是：**写 agent body 的同时就确定 allowedTools，两者同时提交**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 echo-sleuth 中是否有 agent 声明了 Write/Edit 但 body 说只读
# （对应 shanraisshan 的系统性 bug）
grep -l "Write\|Edit" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude/agents/*.md 2>/dev/null | \
  xargs -I{} sh -c 'echo "=== {} ===" && grep -n "NOT modify\|read.only\|DO NOT write" {}'
```
命中后怎么办：在命中文件的 `allowedTools` 中移除 `Write`、`Edit`，保留实际需要的工具。

```bash
# 检查 echo-sleuth 命令是否缺少 allowed-tools frontmatter
# （对应 shanraisshan 的 8 个 command 缺 allowed-tools）
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude/commands/ 2>/dev/null
```
命中后怎么办：分析每个命令实际会调用什么工具，在 frontmatter 中加 `allowed-tools:` 字段。

```bash
# 检查 echo-sleuth 中 agent 是否有 <example> 块
# （对应 shanraisshan 零示例块的系统性问题）
grep -rL "<example>" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude/agents/ 2>/dev/null
```
命中后怎么办：为每个命中的 agent 在 description 字段后加 1-2 个 `<example>` 块，描述"什么情况下该 agent 会被触发"。

```bash
# 检查 claude-for-legal 的模糊量词密度（上报 218 个）
grep -rn -E '\b(appropriate|relevant|comprehensive|several|various|sufficient)\b' \
  /tmp/my-repos/MarkQWu-claude-for-legal/.claude/ 2>/dev/null | wc -l
```
命中后怎么办：逐文件审查密度最高的前 3 个文件，将模糊量词替换为具体可量化的描述（如"relevant → 包含至少一个匹配关键词的条目"）。

```bash
# 检查 name 冲突风险（对应 shanraisshan Bug #2、#3）
grep -rh "^name:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude/agents/ 2>/dev/null | \
  sort | uniq -d
```
命中后怎么办：保留更具体命名的版本，对另一版本添加域后缀（如 `time-agent-pkt`）。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的每个 agent 补充 `<example>` 块
   - **为何可行**：echo-sleuth 有 5 个 agent，每个只需 5-8 行示例，总工作量约 30 分钟
   - **第一步**：打开 `.claude/agents/` 目录，逐个查看 description 字段，在每个 agent 描述结尾加 `<example>` 块，内容描述"用户在什么情况下应该使用这个 agent"
   - **验证方式**：完成后运行 `/nlpm:score .claude/agents/` 查看得分变化

2. **想法**：为 claude-for-legal 建立 3 类 allowedTools 基线模板，写入 CLAUDE.md
   - **为何可行**：claude-for-legal 有 16 个 agent，创建一次模板，所有 agent 按类型套用即可
   - **第一步**：阅读每个 agent body，分类为"只读分析型"（Bash、Read、Grep）、"文档生成型"（Write、Edit、Read、Bash）、"联网研究型"（WebFetch、WebSearch、Read），在 CLAUDE.md 中记录分类标准
   - **验证方式**：对分类后的 agent 跑 `/nlpm:score`，检查 R11 违规数是否降为 0

3. **想法**：在 drama-workshop-skills 中，为所有 command 补充 allowed-tools 声明
   - **为何可行**：shanraisshan 的 8 个 command 因此丢了 -5 × 8 = 40 分（在命令层），echo-sleuth 和 drama-workshop-skills 很可能有相同问题
   - **第一步**：运行 `grep -rL "allowed-tools" commands/` 找出所有缺少声明的命令，按实际调用工具补充
   - **验证方式**：运行 `/nlpm:score commands/`，确认无 BUG-undeclared-tool 类型警告

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：作为 Claude Code 最佳实践的活体参考实现，覆盖 agents、commands、skills、hooks 四大层次的设计模式和常见陷阱

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都有 agents+commands+skills 三层结构；都面向 Claude Code 生态；都有多个 agent 协同 | shanraisshan 是参考仓库，echo-sleuth 是功能性工具；shanraisshan 有 46K stars，echo-sleuth 是个人项目 | **最高**：结构最相似，问题最可迁移 |
| MarkQWu/claude-for-legal | 中 | 都有多个专业域 agent；都有大量 skill 文件；结构复杂度相近 | 法律工作流 vs 通用最佳实践；claude-for-legal 文件数量更多（16 agent）；模糊量词问题更严重（218 个） | **高**：模糊量词问题有直接对应 |
| MarkQWu/drama-workshop-skills | 低 | 都有 command 文件和 skills；都有结构化工作流 | 创意内容 vs 工程实践；drama-workshop 没有专业 agent 层 | **中**：command 的 allowed-tools 问题可迁移 |

### 8.2 在我的项目里复现的同类问题

| shanraisshan 缺陷 | 在我项目中的检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 零示例块（20 个，100%） | `grep -rL "<example>" .claude/agents/` | echo-sleuth 的 5 个 agent 均无 `<example>` 块；claude-for-legal 的 16 个 agent 预计也全无 | **高**：和 shanraisshan 一样 100% 缺失 |
| commands 缺 allowed-tools（8 个） | `grep -rL "allowed-tools" .claude/commands/` | echo-sleuth 的 `lessons.md`、`recall.md`、`recap.md`、`timeline.md` 确认无 allowed-tools（4/8 命令受影响） | **中**：直接对应同一问题 |
| 模糊量词（RPI 命令和 agent 中） | `grep -rn "appropriate\|relevant\|comprehensive" .claude/` | echo-sleuth 中有 12 个模糊量词；claude-for-legal 有 218 个 | **中-高**：claude-for-legal 严重度远超 shanraisshan |
| allowedTools 复制粘贴导致工具列表不精准 | 检查 allowedTools 和 body 中实际使用的工具是否匹配 | echo-sleuth：无 Write/Edit 矛盾（已避免）；但 allowedTools 设计是否真正 least-privilege 未经验证 | **低**（当前状态）：未发现严重矛盾，但需主动验证 |

### 8.3 别人的更优方案

1. **领域**：Skill 层的精心打磨
   - **shanraisshan 的做法**：9 个 skill 全部 100/100。关键要素：每个 skill 聚焦单一知识域、无模糊量词、有清晰的"When to use"说明；`weather-svg-creator/SKILL.md` 有具体的 SVG 模板示例；`agent-browser/SKILL.md` 有详细的工具使用说明
   - **我的项目现状**：echo-sleuth 有 2 个 skill，claude-for-legal 的 skill 有 218 个模糊量词（整体）；drama-workshop-skills 的 skill 文件无 frontmatter
   - **如何借鉴**：先把 echo-sleuth 的 2 个 skill 文件对标 `weather-svg-creator/SKILL.md` 进行 gap 分析；重点检查是否有 scope note 和 cross-reference；对 claude-for-legal 的 skill 按模糊量词密度排序，优先修复前 5 个高密度文件

2. **领域**：多 Agent 工作流的引用完整性
   - **shanraisshan 的做法**：所有 8 个工作流 command ↔ agent ↔ skill 引用链全部通过验证（0 个断链）。具体机制：command 里的 `subagent_type` 字段值和 agent 的 `name:` 字段值严格一致
   - **我的项目现状**：echo-sleuth 的 command 和 agent 之间是否有类似的 subagent_type 引用未经验证；claude-for-legal 的 16 个 agent 是否都被某个 command 引用也未知
   - **如何借鉴**：运行 `/nlpm:check` 对 echo-sleuth 做跨组件一致性检查，确认无断链；对 claude-for-legal 建立 command-to-agent 引用图，检测孤立 agent（未被任何 command 引用的 agent）

3. **领域**：Hook 的 config/script 分离
   - **shanraisshan 的做法**：`hooks-config.json` 只做声明（100/100），`hooks.py` 做逻辑；两者职责完全隔离，config 变更不需要改逻辑，逻辑变更不影响 config
   - **我的项目现状**：drama-workshop-skills 如果有 hook，应该检查是否遵循同样的分离模式
   - **如何借鉴**：如果 drama-workshop-skills 的 hook 把触发条件写在 .sh 文件里，应该抽出来放进单独的 config json

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：Agent allowedTools 和 body 行为的一致性（避免 Write/Edit on readonly）
   - **我的做法**：echo-sleuth 的 5 个 agent 均无"body 说只读但 allowedTools 有 Write/Edit"的矛盾。在创建 agent 时我明确区分了"读操作 agent"和"写操作 agent"，虽然没有形成书面模板，但实际没有出现这个矛盾。
   - **shanraisshan 的问题**：审计时 8 个只读 agent 全部声明 Write/Edit，是系统性的复制粘贴失误，每个文件扣 -10 分
   - **意义**：这个差异说明 allowedTools 和 body 的一致性检查是有效的——即使没有 NLPM 也能在 review 时发现。我的习惯是写完 body 后检查"这个 agent 实际会用哪些工具"，这个习惯在这里保护了我

2. **领域**：单 Agent 的工具最小化（time-agent 类比）
   - **我的做法**：echo-sleuth 中没有"单命令 agent 却声明 10+ 工具"这类问题（如 shanraisshan 的 `time-agent.md` 只运行 `bash date` 但声明了 11 个工具，单文件扣到 57/100）。
   - **shanraisshan 的问题**：`time-agent.md` 是本次审计最低分文件（57/100），根本原因就是"单一职能 + 过度工具声明"
   - **意义**：对于极简 agent（只做一件事），工具列表应该是最严格的 `allowedTools: [Bash]`。我的 echo-sleuth agent 虽然也不够精确，但没有出现这种极端情况

---

## 八、术语表

### allowedTools
> agent 或 command 的 frontmatter 字段，用 YAML 列表格式声明该 artifact 被允许调用的工具集（如 `allowedTools: [Bash, Read, Write]`）。Claude Code 读取此字段后会限制 artifact 的工具调用范围。如果 allowedTools 声明了 `Write` 但 body 明确说"DO NOT modify any files"，就构成了声明与行为的矛盾，也是本案例最核心的系统性 bug。NLPM 规则 R11 专门检测此矛盾。

### frontmatter
> Markdown 文件最顶部用 `---` 包裹的一段 YAML 配置块，用于声明文件的元数据。对于 agent 文件，关键字段包括 `name`（在 Claude Code 中注册的名称，command 通过此名称引用 agent）、`description`（在斜杠命令菜单中显示的说明）、`allowedTools`（工具白名单）、`model`（指定使用的模型）。缺少 `description` 字段会导致命令无法在 `/` 菜单中出现（本案例 Bug #1 的直接原因）。

### agent
> Claude Code 的可调用执行单元，定义在 `.claude/agents/` 目录下的 `.md` 文件中。通过 frontmatter 声明名称、工具权限和调用条件；通过 body 描述执行逻辑和行为约束。agent 可以被 command 显式调用（通过 `subagent_type` 字段），也可以被 Claude Code 根据对话上下文自动触发（依赖 `<example>` 块提供的触发信号）。本案例中 20 个 agent 全部没有 `<example>` 块是导致自动触发不可靠的直接原因。

### command
> Claude Code 的用户入口点，定义在 `.claude/commands/` 目录下，通过斜杠命令（如 `/time-command`）被用户触发。command 的 frontmatter 声明 `description`（在 `/` 菜单中展示）、`allowed-tools`（命令可使用的工具）、`argument-hint`（提示用户如何传参）。command 通过 `subagent_type` 字段引用 agent，通过 `loads:` 字段加载 skill。command 不做复杂逻辑，只做路由和协调。

### skill
> Claude Code 的知识库单元，定义在 `.claude/skills/` 目录下，提供领域知识、模板、规范等静态内容。skill 没有工具声明，不执行操作，只提供知识。正因如此，本案例中 9 个 skill 全部 100/100——skill 层没有 allowedTools 风险，只要知识描述清晰、无模糊量词、有 scope note 即可满分。

### YAML
> YAML Ain't Markup Language，一种人类可读的数据序列化格式，用于编写 frontmatter 配置块。Claude Code artifact 的 frontmatter 使用 YAML 语法：`key: value`（字符串）、`key: [item1, item2]`（列表）等。本案例中的 allowedTools 声明使用 YAML 列表格式 `allowedTools: [Bash, Read, Write, ...]`。YAML 格式不一致（一些 agent 用列表格式，另一些用字符串格式 `tools: Bash`）是本案例的 Informational 发现之一。

### MCP服务器
> Model Context Protocol 服务器，通过 `.mcp.json` 配置文件声明，为 Claude Code 提供额外工具能力（如 `@playwright/mcp` 提供浏览器操作能力、`context7-mcp` 提供文档检索能力）。MCP 服务器通常通过 `npx -y <package>` 命令按需安装。本案例的 Medium 安全风险来源于三个 MCP 服务器均使用 `npx -y` 但未锁定版本号，意味着任何 npm 包的更新（包括恶意版本）都会被自动安装。NLPM PR #67 通过将版本号锁定（如 `@playwright/mcp@0.0.70`）修复了此风险，但版本锁定后需要定期主动升级以接收安全补丁。
