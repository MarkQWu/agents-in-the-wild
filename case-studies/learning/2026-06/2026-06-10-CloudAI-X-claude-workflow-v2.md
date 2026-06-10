# CloudAI-X/claude-workflow-v2 — 学习案例

**仓库**：https://github.com/CloudAI-X/claude-workflow-v2
**Stars**：未记录（Upstream Pool）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-10（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `fallback-chain`, `examples-driven`, `cross-reference`

---

## 一、理解

### 1.1 仓库概览

CloudAI-X/claude-workflow-v2 是一个面向全栈开发团队的 Claude Code 插件，定位为"工作流编排套件"——提供从代码审查、并行分析、安全扫描到架构设计的一站式 slash command 集合，配合 7 个专用 agent 和 13 个领域 skill 共同工作。

规模数据（2026-04-12 审计快照）：
- **工件总数**：50 个（17 commands, 7 agents, 13 skills, 14 hooks 脚本 via hooks.json, 1 plugin.json, 1 CLAUDE.md）
- **NLPM 综合评分**：**93/100（Gold 级）**，审计周期内高分段代表
- **安全评级**：**CLEAR**（0 Critical，0 High，6 Medium，3 Low）
- **Bug 数**：6 个（含 3 个 Task 工具注册遗漏 + 3 个死链）
- 被 NLPM 选为 Exemplar，exemplifies R04, R05, R06, R07, R08, R12, R16, R30

当前 HEAD（2026-06-10）与历史快照相比，目录结构已有扩充，commands 从 17 个增长到 26 个；核心 BUG 已修复，部分 quality issues 仍存在。

### 1.2 架构剖析

**目录结构（当前 HEAD）：**

```
claude-workflow-v2/
  CLAUDE.md              ← 团队约定文档（含 Python hooks 说明）
  CODEBASE.md            ← 代码库规范
  CONTRIBUTING.md        ← 贡献指南
  PERMISSIONS.md         ← 权限声明
  SECURITY.md            ← 安全策略
  commands/ (26 files)   ← slash command 入口
    parallel-review.md   ← 并行代码审查（多 agent）
    parallel-analyze.md  ← 并行视角分析（多 agent）
    verify-changes.md    ← 变更验证（8 个子 agent）
    refactor-guided.md   ← 引导式重构（98/100 最高分）
    dependency-upgrade.md
    ...（其余 21 个 command）
  agents/ (7 files)
    orchestrator.md      ← 编排 agent，任务分派
    code-reviewer.md     ← 代码审查 agent
    debugger.md
    refactorer.md
    test-architect.md
    docs-writer.md
    security-auditor.md
  skills/ (13 subdirs)
    error-handling/      ← 480 行，Exemplar 核心文件（R04/R05/R06）
    parallel-execution/  ← 含 When-to-Load 结构（R07/R08）
    designing-apis/      ← 历史 BUG：OPENAPI-TEMPLATE.md 死链（已修复）
    convex-backend/      ← 历史 BUG：AGENTS.md 死链（已修复）
    vercel-react-best-practices/
    designing-tests/
    analyzing-projects/
    devops-infrastructure/
    managing-git/
    optimizing-performance/
    database-design/
    web-design-guidelines/
    security-patterns/
  hooks/
    hooks.json           ← 注册 6 个事件类别，14 个脚本
    protect-files.py
    security-check.py
    format-on-edit.py    ← Medium 安全风险（file_path 传入 subprocess）
    verify-on-complete.py ← Medium 安全风险（执行 package.json 任意脚本）
    typescript-check.py
    track-metrics.py
    validate-environment.py
    validate-prompt.py
    pre-commit-check.py
    log-commands.sh      ← Medium 安全风险（CLAUDE_PROJECT_DIR 未验证）
    branch-protection.sh
    notify-complete.sh
    notify-input.sh
  packages/add-skill/    ← Node.js CLI installer
  .claude-plugin/plugin.json
```

**两个核心设计轴：**

第一轴——「并行编排 + 子 agent 派发」：`parallel-review.md`、`parallel-analyze.md`、`verify-changes.md` 是仓库最复杂的 command，设计上依赖 Task 工具启动多个子 agent 并行工作。这三个文件是历史 BUG 的来源，也是修复后最能体现架构意图的文件。

第二轴——「技能库深度」：13 个 skill 中，`error-handling` 以 480 行做到了"每行有价值"——custom error hierarchy（3 语言）、structured logging with correlation IDs、exponential backoff、CircuitBreaker 完整实现、12 行 anti-pattern 对照表。`parallel-execution` 则以 When-to-Load 结构（Trigger + Skip 双行）解决"何时加载"问题。

### 1.3 设计思路 / 方法论

**核心设计哲学：深度技能 + 结构化输出 + 完整钩子体系。**

CloudAI-X 的选择是"少 command、深 skill"：26 个 command 覆盖了大多数开发工作流场景，但每个 skill 文件都不是描述型文档，而是可操作的参考资料（runnable code examples、named pattern templates、adversarial self-review checklist）。

**解决什么问题**：全栈团队在使用 Claude Code 时，最常见的问题是"模型知道要做什么，但不知道做到什么标准"。CloudAI-X 通过三类手段回答这个问题：

1. **输出格式模板（R16）**：`refactor-guided.md` 在变更前输出 Refactoring Scope 报告（含 Risk level 字段），在变更后输出 Refactoring Summary（含 Reverted Attempts 字段），强迫模型汇报"什么尝试了但失败了"。
2. **Adversarial Self-Review（Worth Adopting 模式）**：`code-reviewer.md` 和 `security-auditor.md` 均含自查 checklist——"Am I nitpicking?"、"Did I verify my claims?"——强迫 agent 在输出前质疑自己的结论。
3. **Effort Scaling Table（Worth Adopting 模式）**：两个 agent 均有 Instant/Light/Deep/Exhaustive 四行表格，避免单行改动触发 Exhaustive 级别的审查。

**做了什么 trade-off**：深度技能意味着上下文成本。480 行的 error-handling skill 在每次触发时都会被完整加载，对短暂的调试会话而言是过度开销。这是"质量优先 vs 效率优先"的显性选择。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

本仓库体现了两个独立可命名的模式：

**模式一：「钩子全覆盖体系」（Full-Coverage Hook System）**

结构定义：
```
hooks/hooks.json
  ├── PreToolUse     → protect-files.py, validate-prompt.py
  ├── PostToolUse    → format-on-edit.py, typescript-check.py
  ├── Notification   → notify-complete.sh, notify-input.sh
  ├── Stop           → verify-on-complete.py, track-metrics.py
  ├── SessionStart   → validate-environment.py
  └── UserPromptSubmit → security-check.py, log-commands.sh, branch-protection.sh, pre-commit-check.py
```

6 个事件类别全部覆盖，14 个脚本分工清晰，全部使用 `${CLAUDE_PLUGIN_ROOT}` 路径前缀（R30）。这是目前公开 Claude Code 插件中最完整的钩子体系之一。

**模式二：「深度技能 + 触发边界双行」（Depth Skill with Dual-Boundary Trigger）**

以 `skills/parallel-execution/SKILL.md` 为标本：
```
### When to Load
- **Trigger**: Multi-agent tasks, concurrent operations, spawning subagents, parallelizing independent work
- **Skip**: Single-step tasks or sequential workflows with no parallelization opportunity
```

Trigger 行定义"何时加载"，Skip 行定义"何时不加载"。没有 Skip 行，相邻场景会误触发技能（如顺序迁移任务错误触发并行执行技能）。

### 2.2 适用场景

**钩子全覆盖体系**适用于：
- 多人团队共享同一 Claude Code 配置，需要统一执行标准（格式化、安全检查、会话追踪）
- 有代码保护需求（protect-files.py 阻止对特定路径的写入）
- 想要指标追踪（track-metrics.py 记录会话数据）

不适用于：
- 个人工具箱（钩子体系的维护成本对单人项目而言是过度负担）
- 无 Python 环境的场景（9 个 Python 脚本均依赖 stdin JSON 解析 + subprocess）

**Dual-Boundary Trigger**适用于：
- 技能库中存在多个"相邻"技能，需要精确控制加载边界
- skill 描述已足够丰富但仍频繁被误触发的场景

### 2.3 与其他架构对比

| 架构类型 | 代表仓库 | 技能深度 | 钩子体系 | 主要优势 | 主要缺点 |
|---------|---------|---------|---------|---------|---------|
| 深度技能 + 完整钩子（本仓库）| CloudAI-X/claude-workflow-v2 | 高（480 行 skill）| 完整（6 类别 14 脚本）| 工业级完整性 | 维护成本高，安全面宽 |
| 纯 skill 精品库 | googleworkspace/cli（99/100）| 极高（四层抽象梯度）| 无 | 质量极高，上下文精准 | 无编排 command，无钩子 |
| agent 团队集合 | wshobson/agents（82/100）| 中等 | 有（但仅 protect-mcp）| 覆盖面广，插件市集效应 | manifest 纪律差，重复文件多 |
| 单一深度工具 | 2389-research/review-squad（96/100）| 高 | 无 | 质量集中，维护简单 | 覆盖面窄 |

### 2.4 改进空间

1. **安全路径验证统一化**：`log-commands.sh`、`track-metrics.py`、`validate-environment.py` 均以 `CLAUDE_PROJECT_DIR` 未经验证地构建文件路径。当前分散在三个文件中各自修复成本高。更优方案：提取一个共用的 `safe_project_path()` 工具函数（Python）或 `validate_dir()` shell 函数，在所有脚本中统一调用。

2. **`verify-on-complete.py` 的 package.json 脚本白名单**：当前读取 `package.json` 的 `scripts.test` 和 `scripts.lint` 键值后直接执行。攻击面：如果 `package.json` 被篡改，任意命令将在 session Stop 时执行。修复：硬编码允许的测试运行器列表（`npm test`、`pytest`、`go test ./...`），拒绝执行其他值。

3. **CLAUDE.md 与 hooks.json 的文档同步**：CLAUDE.md 写"Hooks: 1 files"（指 hooks.json），但实际注册了 14 个脚本。新贡献者看不到钩子体系的完整面貌。改进：在 CLAUDE.md 中列出 6 个事件类别和每类别的脚本名称。

4. **`allowed-tools` CI 门控**：当前仍有部分 command 缺少 `allowed-tools` 声明（14/26 有声明，12/26 缺失）。建议在 `.github/workflows/validate.yml` 中加入检查：
   ```bash
   grep -rL "allowed-tools:" commands/*.md && echo "Missing allowed-tools!" && exit 1
   ```

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

**综合评分：93/100（Gold 级）**，50 个工件

| 文件 | 类型 | 得分 | 主要问题 |
|------|------|------|---------|
| `commands/parallel-review.md` | command | 82 | **BUG**：使用 Task 工具但未在 `allowed-tools` 声明；无 empty-input handling |
| `commands/parallel-analyze.md` | command | 82 | **BUG**：同上；4 个并行视角 agent 无法启动 |
| `commands/quick-fix.md` | command | 88 | `$ARGUMENTS` 无 empty-input fallback；静默无目标执行 |
| `commands/lint-fix.md` | command | 88 | 无输出格式模板；"Report what was fixed" 是唯一输出指导 |
| `commands/summarize-changes.md` | command | 88 | `$ARGUMENTS` 无明确 fallback 到 today |
| `skills/convex-backend/SKILL.md` | skill | 88 | **BUG**：第 119 行 `see: AGENTS.md`，文件不存在；死链 |
| `skills/vercel-react-best-practices/SKILL.md` | skill | 88 | **BUG**：第 111 行 `see: AGENTS.md`，同一死链；45 条规则不可达 |
| `skills/designing-apis/SKILL.md` | skill | 90 | **BUG**：第 198 行 `OPENAPI-TEMPLATE.md` 死链；API 设计验证模板不可达 |
| `commands/review.md` | command | 92 | 缺 `allowed-tools`；"thorough" 模糊 |
| `commands/verify-changes.md` | command | 93 | **BUG**：8 个子 agent（5 验证 + 3 对抗）无法派发，`Task` 未声明 |
| `commands/mentor.md` | command | 93 | 缺 `allowed-tools` |
| `commands/run-tests.md` | command | 93 | 缺 `allowed-tools`；含过时安装说明 `cp templates/subagents/...` |
| `commands/code-simplifier.md` | command | 93 | 缺 `allowed-tools`；含过时 `cp templates/subagents/...` |
| `commands/validate-build.md` | command | 93 | 缺 `allowed-tools`；含过时 `cp templates/subagents/...` |
| `CLAUDE.md` | docs | 92 | 写 "Hooks: 1 files"，实际 14 个脚本；文档与实现不同步 |
| `agents/orchestrator.md` | agent | 94 | "appropriate subagents" (-2)；"relevant file paths" (-2) 轻度模糊 |
| `agents/test-architect.md` | agent | 94 | "comprehensive tests"、"appropriate data structures" 各 -2 |
| `commands/refactor-guided.md` | command | 98 | **最高分 command**；含完整 Refactoring Scope + Summary 双模板 |
| `commands/dependency-upgrade.md` | command | 98 | 最佳实践 command |
| `agents/code-reviewer.md` | agent | 97 | Adversarial Self-Review 模式标本 |
| `skills/error-handling/SKILL.md` | skill | 97 | Exemplar R04/R05/R06 核心证据文件，480 行全密度 |

安全评级：**CLEAR**（0 Critical，0 High，6 Medium，3 Low）

### 3.2 当时值得借鉴的模式

**模式 1：Skill description 打包多触发条件（R04 证据）**

`skills/error-handling/SKILL.md` 第 2-3 行：
```
description: Implements error handling patterns, structured logging, retry strategies,
circuit breakers, and graceful degradation. Use when designing error handling,
setting up logging, implementing retries, adding error tracking, or when asked
about error boundaries, log aggregation, alerting, or resilience patterns.
```

2 句话打包了 5 个不同的触发意图。每个意图对应真实的不同用户请求，没有语义重叠。这与"Helpful for errors"（空洞）或"Use when there are errors"（单一触发）形成对比——后者只能触发单一场景。

**模式 2：反模式对照表（R05 / R06 证据）**

`skills/error-handling/SKILL.md` 第 463–480 行是 12 行 AVOID / DO INSTEAD 对照表：
```
AVOID                              DO INSTEAD
Empty catch blocks                 Log and handle or re-throw
Bare `except:` in Python           Catch specific exceptions
console.log for production         Structured logger (pino, winston)
...
```

每行 AVOID 都有具体后果，每行 DO INSTEAD 都有可操作替代。这是与"don't use bare except"（只有规则无示范）的本质区别。

**模式 3：Dual-Boundary Trigger（R07 证据）**

`skills/parallel-execution/SKILL.md` 第 9–12 行：Trigger 行 + Skip 行并存。Skip 行阻止顺序工作流误触发并行执行技能。单 Trigger 行无法划定边界。

**模式 4：Named Pattern Templates（R08 证据）**

`skills/parallel-execution/SKILL.md` 用 Pattern 1/2/3 命名不同的并行化方案，每个 pattern 有 slot-fill 模板（spawn N subagents: Subagent 1:... Subagent 2:...），使 agent 能在第一次遇到时直接填空而非理解概念。

**模式 5：Effort Scaling Table（Worth Adopting）**

`agents/code-reviewer.md` 和 `agents/security-auditor.md` 均有 Instant/Light/Deep/Exhaustive 四行表格，将触发条件映射到审查深度。没有此表，agent 对所有调用默认 Exhaustive——单行改动触发完整架构审查是资源浪费。

### 3.3 当时的缺陷

1. **BUG-1/2/3：三个并行 command 的 `allowed-tools: Task` 缺失**（最高优先级）：`parallel-review.md`、`parallel-analyze.md`、`verify-changes.md` 在设计上依赖 Task 工具派发子 agent，但 frontmatter 的 `allowed-tools` 字段不包含 `Task`。根因：这三个 command 可能在 `Task` 工具注册规范确立后创建，作者没有意识到需要显式声明。影响：并行子 agent 无法启动，命令核心功能静默失效——无错误、无警告，只是不执行。

2. **BUG-4/5：两个 skill 的 `AGENTS.md` 死链**：`skills/convex-backend/SKILL.md` 第 119 行和 `skills/vercel-react-best-practices/SKILL.md` 第 111 行均写 `see: AGENTS.md`，而该文件既不在 skill 目录也不在插件根目录。根因：这两个 skill 最初可能来自另一个有 AGENTS.md 的仓库，搬迁时漏删或漏建引用目标。

3. **BUG-6：`designing-apis/SKILL.md` 的 OPENAPI-TEMPLATE.md 死链**：第 198 行的 `[OPENAPI-TEMPLATE.md](OPENAPI-TEMPLATE.md)` 指向不存在的文件。根因：API 设计验证流程依赖一个应当随 skill 一起提供的 OpenAPI 模板文件，但该文件在历史某个版本中被删除或未提交。

4. **Q-1 到 Q-9：9 个 command 缺少 `allowed-tools` 声明**：mentor, run-tests, code-simplifier, review, lint-check, rapid, security-scan, architect, validate-build 均无 `allowed-tools` 字段。

5. **Q-10/11/12：3 个 command 无 empty-input guard**：quick-fix, sync-branch, summarize-changes 均接受 `$ARGUMENTS` 但没有"如果参数为空则..."的分支，导致无目标静默执行。

6. **Q-13–Q-15：过时安装说明**：run-tests, lint-check, verify-changes, code-simplifier, validate-build 均含 `cp templates/subagents/...` 安装指引，而 `templates/` 目录不存在。这是从早期模板仓库迁移时遗留的失效指令。

### 3.4 当时的优化机会

1. **最高 ROI（30 分钟）**：为 parallel-review, parallel-analyze, verify-changes 的 frontmatter 添加 `allowed-tools: [Task, Read, Bash, ...]`。单行修复，恢复核心功能。

2. **高 ROI（2 小时）**：为 convex-backend, vercel-react-best-practices 创建对应的 `AGENTS.md`，或删除 `see:` 引用，改为内联说明。为 designing-apis 创建 OPENAPI-TEMPLATE.md 或将模板内容内联。

3. **中 ROI（半天）**：为 9 个缺少 `allowed-tools` 的 command 补充声明。逐一分析每个 command 实际需要的工具，而非一律使用最大权限。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在当前 HEAD 中的状态

以下为 2026-06-10 基于当前 HEAD 的 spot-check 结果：

| 缺陷编号 | 文件 | 验证方法 | 2026-04-12 状态 | 当前状态 |
|---------|------|---------|----------------|---------|
| BUG-1 | `commands/parallel-review.md` | `grep -n "allowed-tools" commands/parallel-review.md` | 无 `allowed-tools` 字段，Task 未声明 | **已修复**：`allowed-tools` 字段已添加，含 `Task` |
| BUG-4 | `skills/convex-backend/SKILL.md` | `ls skills/convex-backend/AGENTS.md` | 死链，`AGENTS.md` 不存在 | **已修复**：`skills/convex-backend/AGENTS.md` 文件已创建 |
| BUG-6 | `skills/designing-apis/SKILL.md` | `ls skills/designing-apis/OPENAPI-TEMPLATE.md` | 死链，`OPENAPI-TEMPLATE.md` 不存在 | **已修复**：`skills/designing-apis/OPENAPI-TEMPLATE.md` 文件已创建 |
| Q-9 中的 run-tests.md | `commands/run-tests.md` | `grep -n "templates/subagents" commands/run-tests.md` | 含 `cp templates/subagents/...` 安装说明 | **已修复**：过时安装说明已删除 |
| Q-1 到 Q-9 allowed-tools | `commands/` 全部文件 | `grep -rL "allowed-tools:" commands/` | 9/17 命令缺失 | **部分修复**：26 个 command 中 14 个有 `allowed-tools` 声明，12 个仍缺失；核心并行 command 已修复 |

**核心结论**：最高优先级的 3 个并行 command BUG 和 3 个死链 BUG 均已修复。过时安装说明已清理。`allowed-tools` 缺失问题仍以较低比率存在于新增 command 中（14/26 = 54% 有声明，vs 历史快照的 8/17 = 47%——略有改善但未彻底解决）。

### 4.2 架构演进

commands 从 17 个扩充到 26 个，增量 9 个新 command。目录增加了 `packages/add-skill/`（Node.js CLI installer）和 `SECURITY.md`、`PERMISSIONS.md`、`CONTRIBUTING.md` 等治理文档，说明项目向更正式的开源协作方向演进。

CLAUDE.md 内容显示团队已将 Python hooks 的 stdin JSON + subprocess 调用规范明确写入约定，并文档化了"如何添加新 agent / skill / hook"的标准步骤——这是历史审计中未存在的治理文档。

安全面扩展方向未变：新增 command 均使用 Python hooks，`CLAUDE_PROJECT_DIR` 路径验证问题（Medium #1/2/6）尚未被系统性修复。

### 4.3 新增的可学习模式

1. **治理文档的形式化**：`CONTRIBUTING.md` 和 `PERMISSIONS.md` 的引入，使 CloudAI-X 从"个人工具"转向"团队可维护工件"。这一转变的信号意义大于文档本身内容——一个有 CONTRIBUTING.md 的插件，意味着维护者预期外部贡献。

2. **`packages/add-skill/` 安装工具**：Node.js CLI installer 是一个有趣的新组件——它将 skill 的安装体验从"手动复制文件"变成"命令行一键"。这解决了 wshobson/agents 等大型仓库面临的"用户不知道如何只安装其中一个 skill"的问题。

---

## 五、校准

### 5.1 我已经在做对的

1. **我的 command 文件均有 frontmatter 块**：`echo-sleuth-for-claude` 的 5 个 command 和 `claude-for-legal` 的 command 全部有 `---` frontmatter 块。CloudAI-X 的历史 BUG-1/2/3 的根因是有 frontmatter 但缺关键字段（`Task` 未在 `allowed-tools` 声明）——这比完全无 frontmatter 更隐蔽，因为注册不会完全失败，而是部分功能静默失效。我的文件在结构完整性上满足最低要求。

2. **我没有死链引用**：drama-workshop-skills 和 echo-sleuth 的 skill 文件均不包含对不存在文件的 `see:` 或 `[link]()` 引用。CloudAI-X 的 BUG-4/5/6 均来自"从其他仓库迁移 skill 时漏掉引用目标"——这是一种迁移风险，我的仓库均为原创编写，不存在此类迁移残留。

3. **drama-workshop-skills 有清晰的 skill 边界**：每个 skill 聚焦单一创作阶段（策划/剧情/对话/场景），没有技能之间的交叉引用链——不存在 CloudAI-X 式的 "AGENTS.md 死链" 问题。

### 5.2 挑战

1. **echo-sleuth 的所有 5 个 command 缺少 `allowed-tools` 声明**：这与 CloudAI-X 的 Q-1 到 Q-9 是完全相同的问题——commands 有 frontmatter 但无工具权限声明。CloudAI-X 的案例证明这个问题不会"被遗漏而没有后果"：当某个 command 需要使用 Task 工具时，缺少声明直接导致核心功能静默失效（BUG-1/2/3 的机制）。即使现在 echo-sleuth 的 command 不使用 Task，未来添加并行能力时会踩到同样的坑。

2. **我的 agent 缺少 Effort Scaling Table**：CloudAI-X 的 `code-reviewer.md` 和 `security-auditor.md` 均有 Instant/Light/Deep/Exhaustive 四行表格，将触发条件映射到审查深度。我的 `claude-for-legal` 的 10 个 agent 和 `echo-sleuth` 的 5 个 agent 均无此表。结果：对任何调用均触发最大深度，在简单任务上浪费 token。

3. **我没有 Adversarial Self-Review checklist**：这是 Exemplar 文件中明确标注为"Worth Adopting"的模式。我的 agent 没有"我是否在吹毛求疵？""我是否验证了我的论断？"这类自查步骤，输出质量依赖模型的内在校准，而非结构化约束。

---

## 六、行动

### 6.1 自查动作

针对优先级从高到低：

**P0 — 立即（20 分钟）**：为 echo-sleuth 的所有 5 个 command 补充 `allowed-tools:` 声明。以 `lessons.md` 为例（其余类同）：
```yaml
---
name: lessons
description: Extract lessons and recurring patterns from past Claude Code sessions.
allowed-tools:
  - Read
  - Bash
  - mcp__claude-session__search
---
```
根本原因：CloudAI-X BUG-1/2/3 的机制不是"当前报错"，而是"未来添加 Task 时静默失效"。现在修复比将来调试容易 10 倍。

**P1 — 本周（3 小时）**：为 `claude-for-legal` 中最复杂的 3 个 agent 添加 Effort Scaling Table。判断标准：该 agent 的任务是否在不同场景下对应不同的深度（合同审查 vs 快速条款查询 vs 完整尽职调查）。示例格式：
```markdown
| 触发场景 | 深度 | 工作内容 |
|--------|------|---------|
| 单条条款快查 | Instant | 直接解释，1 分钟 |
| 合同草稿审查 | Light | 逐条检查关键风险点 |
| 完整合同谈判 | Deep | 条款分析 + 替代表述建议 |
| 跨境法律尽职调查 | Exhaustive | 多法域比对 + 风险矩阵 |
```

**P2 — 本月（半天）**：为 claude-for-legal 最高复杂度的 agent（如综合法律风险评估）添加 Adversarial Self-Review checklist。参考 CloudAI-X `agents/code-reviewer.md` 第 108–116 行的模式：
- 我是否引用了实际文件内容，还是依赖用户描述？
- 我识别的风险是否基于具体条款，还是基于泛化的法律常识？
- 我给出的建议是否可操作，还是模糊的"建议咨询律师"？

**P3 — 持续**：在 `echo-sleuth` 和 `claude-for-legal` 的 PR checklist 中加入：`- [ ] 如果 command 使用 Task 工具，`allowed-tools` 必须包含 Task`。

### 6.2 灵感 → 实施路径

**灵感 1：Dual-Boundary Trigger 引入 drama-workshop-skills**

当前 `drama-workshop-skills` 的各 skill 只有 Trigger，没有 Skip 行。在剧本创作场景中，"剧情结构"和"场景描写"是相邻技能，可能在模糊的用户请求下相互干扰。

实施路径：
1. 打开 drama-workshop-skills 每个 SKILL.md 文件
2. 在 description 字段后加"When to Load"块
3. 为每个 Trigger 编写对应的 Skip——明确"本技能不处理什么"
4. 预计每个文件 10 分钟，总计 5 个技能 50 分钟

**灵感 2：从 `refactor-guided.md` 学习双阶段输出模板**

`commands/refactor-guided.md` 的预飞行 Refactoring Scope 报告（变更前）和 Refactoring Summary（变更后，含 Reverted Attempts 字段）是目前最成熟的 command 输出模板设计。

实施路径：
- 在 `claude-for-legal` 中，找到最复杂的工作流 command（如合同起草）
- 引入"起草前：确认范围"和"起草后：输出摘要"双阶段结构
- Reverted Attempts 对应"被排除的条款选项及排除原因"

**灵感 3：从 CloudAI-X 的修复历史学习迁移安全**

BUG-4/5（AGENTS.md 死链）和 BUG-6（OPENAPI-TEMPLATE.md 死链）均来自从其他仓库迁移 skill 时漏掉引用目标。这是一个系统性风险，而非个别疏忽。

实施路径：为任何 skill 迁移操作建立两步核查：
1. 提取新文件中所有 `see:`、`[text](link)` 引用
2. 验证每个引用目标在新仓库中存在

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

| 我的仓库 | 与 CloudAI-X 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|-------------------|------|------|----------|
| MarkQWu/claude-for-legal | **高** | 都是"专业领域工作流套件"定位；都有 agent + command + skill 三层架构；都有编排 command 的需求（法律尽职调查 ≈ verify-changes 的多视角验证）| CloudAI-X 面向开发团队，claude-for-legal 面向法律场景；CloudAI-X 有完整钩子体系，claude-for-legal 的 hooks.json 内容为空 | 高 |
| MarkQWu/echo-sleuth-for-claude | **中** | 都有 commands + agents 双层结构；都面临 `allowed-tools` 缺失问题（CloudAI-X 历史 9/17，echo-sleuth 5/5）；都是 CLEAR 安全评级 | echo-sleuth 规模小（5 commands + 5 agents），无并行 agent 设计；无钩子体系 | 高（allowed-tools 是 P0 问题）|
| MarkQWu/drama-workshop-skills | **低** | 都有 skill 文件；都关注触发条件设计 | drama-workshop 无 commands 也无 agents；领域完全不同（剧本创作 vs 软件工程）| 低（仅 Dual-Boundary Trigger 可直接借鉴）|

**核心结论**：echo-sleuth 在 `allowed-tools` 缺失问题上与 CloudAI-X 历史快照高度同构，且比例更高（5/5 = 100% 缺失 vs CloudAI-X 的 9/17 = 53%）。CloudAI-X 的 BUG-1/2/3 正是在这个基础上发生的。

### 7.2 在我的项目里复现的同类问题

| CloudAI-X 缺陷 | 我的仓库中的复现情况 | 验证命令 | 当前状态 |
|--------------|-----------------|---------|---------|
| BUG-1/2/3：Task 未在 `allowed-tools` 声明 | `echo-sleuth`: 5 个 command 全无 `allowed-tools` | `grep -rL "allowed-tools:" /path/to/echo-sleuth/.claude/commands/` | 100% 缺失，P0 风险 |
| Q-1~Q-9：`allowed-tools` 完全缺失 | `claude-for-legal`: 1 个 command 无 `allowed-tools` | `grep -rL "allowed-tools:" /path/to/claude-for-legal/commands/` | 存在，需修复 |
| BUG-4/5：迁移死链 | 所有仓库均为原创，无迁移来源 | 不适用 | 不存在 |
| Q-10/11/12：无 empty-input guard | `echo-sleuth`: recall, timeline 接受 `$ARGUMENTS` 但无 fallback | `grep -n "ARGUMENTS" .claude/commands/*.md \| grep -v "if.*empty\|fallback\|default"` | 存在，需核查 |
| 过时安装说明 | 无 templates/ 目录的残留引用 | `grep -rn "templates/" .claude/` | 需核查 |

**严重性对比**：echo-sleuth 的 `allowed-tools` 问题严重性高于 CloudAI-X 历史快照——CloudAI-X 有 8/17 个 command 已正确声明（包括高复杂度的 bootstrap-repo），而 echo-sleuth 是 0/5，连最基础的工具声明都缺失。

### 7.3 别人的更优方案

1. **Adversarial Self-Review（CloudAI-X `agents/code-reviewer.md:108-116` > 我的所有 agents）**：
   - CloudAI-X 做法：agent 在输出报告前执行结构化自查 checklist（"Am I nitpicking?"、"Did I miss the forest for the trees?"、"Did I verify my claims?"）
   - 我的现状：所有 agent（echo-sleuth 的 5 个，claude-for-legal 的 10 个）均无自查步骤，输出质量完全依赖模型内在校准
   - 借鉴方法：选择高风险 agent（claude-for-legal 的合同风险评估 agent），添加 `## Adversarial Self-Review` 节，写 3–5 条领域特定自查问题

2. **Effort Scaling Table（CloudAI-X `agents/security-auditor.md:19-26` > 我的所有 agents）**：
   - CloudAI-X 做法：每个 agent 有 Instant/Light/Deep/Exhaustive 四行表，将触发条件精确映射到工作深度
   - 我的现状：agent 没有深度分层，任何调用都触发最高深度
   - 借鉴方法：claude-for-legal 的合同审查 agent 最适合引入此表——"快速条款查询"不应触发完整 SWOT 分析

3. **`$CLAUDE_PLUGIN_ROOT` 路径前缀（CloudAI-X `hooks/hooks.json:7-11` > claude-for-legal 的空 hooks）**：
   - CloudAI-X 做法：14 个脚本全部使用 `${CLAUDE_PLUGIN_ROOT}/hooks/<script>` 作路径，跨用户可移植
   - 我的现状：claude-for-legal 的 `hooks.json` 内容为 `{"hooks": {}}`，完全未利用钩子能力
   - 借鉴方法：为 claude-for-legal 添加至少一个 PostToolUse 钩子（如每次编写合同文件后触发格式检查），使用 `${CLAUDE_PLUGIN_ROOT}` 前缀

### 7.4 反向：我的项目做得比他们好的地方

1. **claude-for-legal 的 hooks.json 不携带安全风险**：空的 `{"hooks": {}}` 意味着零执行面。CloudAI-X 的 6 Medium 安全发现全部来自钩子脚本（`CLAUDE_PROJECT_DIR` 路径未验证、`verify-on-complete.py` 执行 package.json 任意脚本等）。我的仓库在这个维度上是"空白胜于有风险"。

2. **echo-sleuth 的结构聚焦**：echo-sleuth 的 5 个 command 和 5 个 agent 均专注于"分析 Claude Code 会话"这一个目标，无功能漂移。CloudAI-X 的 26 个 command 虽然也相对聚焦，但已出现少量功能边界不清晰的 command（如 `mentor.md` 和 `tutorial.md` 的目标用户与触发场景接近）。

3. **drama-workshop-skills 无安全执行面**：全部是 SKILL.md 文件，无 hooks 脚本，无 shell 执行，安全面为零。CloudAI-X 的安全评级 CLEAR 是"6 Medium 仍为 CLEAR"的最高门槛，我的仓库如果审计会获得"0 发现 CLEAR"——不同层次的 CLEAR。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **allowed-tools** | command 或 agent frontmatter 中声明该工件可以调用的工具列表（如 `Read`、`Bash`、`Task`、`mcp__toolname`）。遵循最小权限原则。CloudAI-X 的 BUG-1/2/3 揭示了缺少此字段的最严重后果：设计为并行工作的 command 在运行时静默失效——因为 `Task` 工具未被声明而无法调用，无任何错误提示。 |
| **Task 工具** | Claude Code 中用于派发子 agent 的特殊工具，允许一个 command 或 agent 在独立上下文中启动另一个 agent 并行执行。与 `Read`、`Bash` 等工具一样，必须在 `allowed-tools` 中显式声明才能使用。CloudAI-X 的并行架构（parallel-review, parallel-analyze, verify-changes）全部依赖此工具。 |
| **empty-input guard** | command 接收 `$ARGUMENTS` 参数时，对参数为空的情况进行明确处理的逻辑分支。典型形式：`如果 $ARGUMENTS 为空，则默认分析最近 24 小时的变更`。没有 empty-input guard 时，`$ARGUMENTS` 为空的调用会静默地以无目标状态执行，产生不可预测的输出。CloudAI-X 的 Q-10/11/12 均为此类问题。 |
| **Adversarial Self-Review** | agent 在生成最终输出之前对自身回答执行的结构化自查步骤。以 CloudAI-X `agents/code-reviewer.md` 第 108–116 行为标本，包含"Am I nitpicking?"、"Did I miss the forest for the trees?"、"Did I verify my claims?"等问题。目的是减少"自信但浅薄"的输出。NLPM Exemplar 文件将此命名为"Worth Adopting"模式。 |
| **Effort Scaling Table** | agent 中的四行表格，将触发场景（如 Instant / Light / Deep / Exhaustive）映射到对应的工作深度和内容范围。CloudAI-X `agents/code-reviewer.md` 和 `agents/security-auditor.md` 均使用此结构。缺少此表时，agent 对所有调用默认执行最高深度，在简单任务上浪费 token 和时间。 |
| **Dual-Boundary Trigger** | skill 的 When-to-Load 块中同时包含 Trigger（何时加载）和 Skip（何时不加载）两行的设计模式。CloudAI-X `skills/parallel-execution/SKILL.md` 第 9–12 行为标本。单 Trigger 行只能定义"什么情况下我适用"，无法阻止"什么情况下我虽然看起来适用但实际不该被加载"。 |
| **Exemplar（标本）** | NLPM 体系中对高分（≥90）且安全评级 CLEAR 的审计目标选出的"教学文件"，存放在 `auditor/exemplars/` 目录。每个 Exemplar 文件包含 frontmatter 中的 `exemplifies:` 字段（列出体现的规则编号）和逐规则的证据引用。CloudAI-X/claude-workflow-v2 exemplifies R04, R05, R06, R07, R08, R12, R16, R30。 |
| **`$CLAUDE_PLUGIN_ROOT`** | Claude Code 提供的环境变量，指向当前插件的安装根目录。在 hooks.json 的脚本路径中使用此变量，可使钩子脚本在任何用户的任何安装路径下均能正确定位。CloudAI-X 的 14 个钩子脚本全部使用此前缀（R30 合规），避免了硬编码绝对路径或相对路径导致的跨用户失效。 |
| **CLAUDE_PROJECT_DIR** | 钩子脚本中由 Claude Code 注入的环境变量，指向当前用户项目目录。CloudAI-X 的 Medium 安全发现 #1/2/6 均因直接使用此变量构建文件路径而未做路径边界验证，存在日志重定向到任意路径的风险。修复方式：在使用前调用 `os.path.realpath()` 并断言路径在已知安全前缀下。 |
