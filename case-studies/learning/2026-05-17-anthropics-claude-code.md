# anthropics/claude-code — 学习案例

**仓库**：https://github.com/anthropics/claude-code
**Stars**：数万（未在注册表中记录，Anthropic 官方仓库）| **来源**：本地 audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD）
**主题标签**：manifest-discipline, examples-driven, cross-reference, security-gate, single-purpose

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

anthropics/claude-code 是 Claude Code CLI 工具的官方仓库，同时也是一个元级（meta-level）示范库：它不仅是工具本身，还在 `plugins/` 目录下内置了 11 个用于扩展 Claude Code 功能的正式插件。这些插件涵盖从代码审查、PR 分析、SDK 开发验证，到 hook 配置管理、安全提醒注入、循环任务编排的全谱系用例。

当前 HEAD 为 `8bdbb7296d3fa2217283d3ef94452dd64097393b`，与 audit 时的 SHA 完全一致——仓库在审查后未发生任何提交，所有 audit 时的快照发现均 100% 适用于当前状态。

本次 audit 共发现 60 个 NL 制品，分布在 11 个插件中，每个插件均为自包含单元，拥有独立的 `.claude-plugin/plugin.json` 清单。

### 1.2 架构剖析

插件按功能域分为五类：

**教学型（plugin-dev 系）**

`plugins/plugin-dev/` 是整个仓库最具架构价值的插件。它包含 7 个 skill：`plugin-structure`、`hook-development`、`skill-development`、`plugin-settings`、`agent-development`、`command-development`、`mcp-integration`，外加 1 个 `plugin-validator` agent。该插件的设计哲学是"通过示例教学"——它是一个关于如何编写插件的插件，skill 内容即是对应制品类型的最佳实践规范。

**开发辅助型**

- `plugins/feature-dev/`：包含 `code-explorer`、`code-architect`、`code-reviewer` 三个 agent，支持特性开发全流程（探索 → 架构 → 审查）。
- `plugins/agent-sdk-dev/`：包含 `agent-sdk-verifier-ts` 和 `agent-sdk-verifier-py` 两个 agent，专门验证 TypeScript 和 Python SDK 的使用合规性。

**审查型**

`plugins/pr-review-toolkit/` 是规模最大的插件，包含 6 个专职 agent：`code-reviewer`、`code-simplifier`、`type-design-analyzer`、`silent-failure-hunter`、`pr-test-analyzer`、`comment-analyzer`，分别覆盖代码质量、简化、类型设计、静默失败检测、测试充分性和评论分析等维度。

**基础设施型（hook 驱动）**

- `plugins/hookify/`：hook 配置管理元插件，内置 4 个 hook 脚本（`pretooluse.py`、`posttooluse.py`、`stop.py`、`userpromptsubmit.py`）和 `conversation-analyzer` agent。
- `plugins/security-guidance/`：通过 `security_reminder_hook.py` 将安全提醒注入每次会话——"环境安全顾问"模式。
- `plugins/ralph-wiggum/`：实现循环/后台任务编排，通过 `stop-hook.sh` 控制任务继续/中止逻辑。

**输出风格型**

- `plugins/learning-output-style/`：规范学习类输出的呈现方式。
- `plugins/explanatory-output-style/`：规范解释类输出的呈现方式。

**命令型**

`.claude/commands/` 下有 3 个用户可调用命令：`triage-issue.md`（Issue 分类）、`commit-push-pr.md`（Git 工作流）、`dedupe.md`（去重）。

### 1.3 设计思路 / 方法论

**单一职责原则（Single Responsibility）**

每个插件只做一件事。`security-guidance` 只注入安全提醒，不做代码审查；`ralph-wiggum` 只管理循环任务，不涉及代码质量。这种边界清晰的设计使插件可以独立安装、独立卸载，互不干扰。

**元级自引用设计（Meta-reference）**

`plugin-dev` 插件的核心价值在于自引用：`skill-development` skill 教你如何写 skill，`hook-development` skill 教你如何写 hook，`agent-development` skill 教你如何写 agent。Anthropic 选择用插件形式来承载插件编写知识，而不是写文档，这使得 Claude 在开发插件时可以直接加载这些 skill 作为上下文，而不是要求用户手动查阅文档。

**环境注入（Ambient Injection）**

`security-guidance` 和 `hookify` 都采用了"环境注入"而非"按需调用"的模式：它们通过 hook 机制在每次工具调用或会话开始时静默注入上下文或检查，不需要用户显式触发。这是一种适合"始终应该生效的约束"的架构选择。

**自包含清单（Self-contained Manifest）**

每个插件都有独立的 `plugin.json`，声明自己的 agent、skill、hook、command 和 MCP 服务器依赖。这一设计使插件可以作为独立单元发布和安装，无需了解其他插件的存在。

---

## 二、过去审查发现（2026-04-06 历史快照）

### 2.1 当时质量评分（NLPM）

**综合评分：88/100**（安全状态：CLEAR，0 Critical，0 High，2 Medium，3 Low）

共 60 个制品，低分制品集中在以下文件：

| 制品 | 得分 | 主要扣分原因 |
|------|------|------------|
| `plugins/agent-sdk-dev/agents/agent-sdk-verifier-ts.md` | 73 | 无 `<example>` 块，无颜色字段 |
| `plugins/agent-sdk-dev/agents/agent-sdk-verifier-py.md` | 73 | 无 `<example>` 块，无颜色字段 |
| `plugins/feature-dev/agents/code-explorer.md` | 75 | 无 `<example>` 块，模糊量词："comprehensive"×3，"specific"×2 |
| `plugins/feature-dev/agents/code-architect.md` | 75 | 无 `<example>` 块 |
| `plugins/feature-dev/agents/code-reviewer.md` | 77 | 无 `<example>` 块 |
| `plugins/pr-review-toolkit/agents/code-simplifier.md` | 82 | 缺少输出格式说明 |
| `plugins/plugin-dev/agents/plugin-validator.md` | 83 | 末尾残留创作痕迹（trailing authoring artifact） |
| `plugins/feature-dev/commands/feature-dev.md` | 83 | 无 `allowed-tools` 字段 |
| `plugins/agent-sdk-dev/commands/new-sdk-app.md` | 83 | 无 `allowed-tools` 字段 |
| `plugins/plugin-dev/commands/create-plugin.md` | 84 | 模糊量词密度偏高 |

**安全发现（CLEAR 级别，无阻断项）：**

| 级别 | 制品 | 问题描述 |
|------|------|---------|
| Medium | `plugins/security-guidance/hooks/security_reminder_hook.py` | 写入 `/tmp` 和 `~/.claude/`（预期行为，非恶意） |
| Low | `plugins/hookify/hooks/pretooluse.py` 第 60 行 | `except Exception` 捕获过宽，失败时放行（fail-open） |
| Low | `plugins/hookify/hooks/stop.py` 第 47 行 | 同上，`except Exception` fail-open |
| Low | `plugins/ralph-wiggum/scripts/setup-ralph-loop.sh` 第 140 行 | `$PROMPT` 变量在 heredoc 中未加引号，存在注入风险 |

### 2.2 当时值得借鉴的模式

**1. 单一职责插件架构**

pr-review-toolkit 用 6 个专职 agent 覆盖代码审查的不同维度，而不是把所有审查逻辑堆入一个 agent。这使得每个 agent 的 prompt 更短、更聚焦、更容易测试。

**2. 元级自引用 skill 设计**

plugin-dev 插件中的 7 个 skill 直接将 Claude Code 的插件规范编码进 NL 制品，使 Claude 在开发插件时可以实时引用这些 skill 作为权威参考，而不是依赖外部文档链接。

**3. hook 驱动的环境注入**

security-guidance 通过 `security_reminder_hook.py` 在每次工具调用前静默注入安全提醒，用户无需任何操作，约束始终生效。这是一种适合"全局约束"的架构模式，优于在每个 agent prompt 中重复写安全要求。

**4. 自包含插件清单（plugin.json）**

每个插件的 `plugin.json` 精确声明了自己依赖的所有制品类型（agents/skills/hooks/commands），使插件可以作为独立单元分发和安装，无外部依赖。

**5. 循环任务编排（ralph-wiggum）**

通过 `stop-hook.sh` 实现"任务是否继续"的决策逻辑，将循环控制从用户手动操作转移到 hook 层。这是一种适合后台自动化任务的编排模式。

### 2.3 当时的缺陷

**缺陷 1（最高影响）：plugin-validator.md 末尾残留创作痕迹**

`plugins/plugin-dev/agents/plugin-validator.md` 文件末尾保留了如下文本：
> "Excellent work! The agent-development skill is now complete…"

这是创作过程中 Claude 输出的确认语句，被意外保留在制品文件中。当 Claude Code 加载此 agent 时，这段文本会被注入 system prompt，直接污染 LLM 上下文，影响 agent 行为的可预测性。

**缺陷 2：type-design-analyzer.md 颜色值非法**

`plugins/pr-review-toolkit/agents/type-design-analyzer.md` 声明 `color: pink`，而 Claude Code 仅接受 `blue`/`cyan`/`green`/`yellow`/`magenta`/`red` 六个合法值。非法颜色值会导致 UI 渲染异常或被忽略。

**缺陷 3–7：五个 agent 完全缺少 `<example>` 块**

以下 agent 均无任何示例：
- `code-explorer.md`（feature-dev）
- `code-reviewer.md`（feature-dev）
- `code-architect.md`（feature-dev）
- `agent-sdk-verifier-ts.md`
- `agent-sdk-verifier-py.md`

无示例意味着用户和 Claude 均无法通过输入/输出对比来校准对 agent 行为的预期。

### 2.4 当时的优化机会

1. **plugin-validator.md**：删除文件末尾的 "Excellent work!…" 等创作确认语句，确保文件内容完全是 agent 指令而非创作过程的副产品。

2. **type-design-analyzer.md**：将 `color: pink` 改为任意合法颜色值（如 `color: magenta`，语义接近）。

3. **五个无示例 agent**：为每个 agent 补充至少 2 个 `<example>` 块，覆盖典型输入与预期输出格式。

4. **命令文件缺少 `allowed-tools`**：为 `feature-dev.md` 和 `new-sdk-app.md` 补充 `allowed-tools` 字段，明确命令执行时允许调用的工具集，防止权限过宽。

5. **代码审查命令的 MCP 依赖未声明**：`plugins/code-review/commands/code-review.md` 在 `allowed-tools` 中引用了 `mcp__github_inline_comment__create_inline_comment`，但插件清单中未声明对应的 MCP 服务器——运行时第 9 步将静默失败。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

基于对当前 HEAD（`8bdbb7296d3fa2217283d3ef94452dd64097393b`）的逐项核查：

| 缺陷 | Audit（2026-04-06）| 当前状态 | 结论 |
|------|-------------------|---------|------|
| plugin-validator.md 末尾残留创作痕迹 | 存在 | `tail -10` 仍显示 "Excellent work! The agent-development skill is now complete…" | **仍未修复** |
| type-design-analyzer.md `color: pink` | 存在 | 字段值仍为 `pink` | **仍未修复** |
| feature-dev code-reviewer.md 无 `<example>` | 存在 | `grep -c "<example>"` = 0 | **仍未修复** |
| code-explorer.md 无 `<example>` | 存在 | `grep -c "<example>"` = 0 | **仍未修复** |
| code-architect.md 无 `<example>` | 存在 | `grep -c "<example>"` = 0 | **仍未修复** |
| `code-reviewer` 双重命名冲突 | 存在 | feature-dev 和 pr-review-toolkit 均声明 `name: code-reviewer` | **仍未修复** |
| code-review 命令 MCP 依赖未声明 | 存在 | `allowed-tools` 中仍引用未声明的 MCP 方法 | **仍未修复** |

**关键洞察**：当前 HEAD SHA 与 audit SHA 完全相同（`8bdbb72`）。仓库在 2026-04-06 审查后未发生任何提交，所有发现均为原始状态，未经任何修复。由于 Anthropic 对外部 PR 实施全面拒绝策略（`DENY_OWNERS` 中包含 `anthropics`），这些 bug 从未被提交为 PR——这是策略性的，而非遗忘。

### 3.2 架构演进

在 audit 窗口（2026-04-06 至 2026-05-17）内，仓库零提交，架构保持完全稳定。11 个插件的结构、60 个制品的分布、各插件的 `plugin.json` 清单均未发生变化。

这种稳定性有两种解读：一是核心架构设计质量足够高，不需要频繁修正；二是该仓库作为官方参考实现，更新节奏可能受发布周期约束，而非持续迭代。

对我们而言，"官方仓库中存在未修复 bug 且仓库静止"提供了一个重要的现实校准：即使是权威参考实现，也会存在机械性的质量缺陷，而这些缺陷不会因为仓库的权威性而自动被修复。

### 3.3 新增的可学习模式

由于仓库完全静止，以下模式是在 audit 时已存在、但在"政策背景"维度下值得重新审视的：

**跨插件命名冲突的现实成本**

`plugins/feature-dev` 和 `plugins/pr-review-toolkit` 都声明 `name: code-reviewer`，当两个插件同时安装时，一个会遮蔽另一个。这不是一个假设性风险——在 Anthropic 的测试环境中，这两个插件很可能同时加载。这个案例说明：即使是官方仓库，也会在跨插件命名空间管理上出现疏漏，而且不会通过 linter 自动捕获，需要人工或 checker 专项扫描。

**policy-denied 仓库的 bug 永久化**

因为 Anthropic 拒绝所有外部 PR，这 7 个机械性 bug 在外部贡献者视角下是永久固化的。这提醒我们：对于 `DENY_OWNERS` 中的仓库，audit 的价值是学习而非贡献，不应期待上游修复。

---

## 四、校准

### 4.1 我已经在做对的

**1. 插件自包含设计**

如果你的插件已经为每个 agent/skill/hook 在 `plugin.json` 中做了完整声明，并且不依赖于其他插件的内部制品，你正在复制 claude-code 插件集中最成功的结构性决策。

**2. 单一职责边界**

如果你的 agent 职责是单一的（只做代码审查，或只做类型分析，而不是"全部审查"），你已经采用了 pr-review-toolkit 6-agent 拆分模式的精髓，这使得每个 agent 的输出质量更容易预测和评估。

**3. hook 环境注入**

如果你在使用 hook 来实现"始终生效的约束"（如格式检查、安全提醒），而不是在每个 agent prompt 中重复写相同的规则，你正在使用 security-guidance 证明有效的模式。

**4. 元级 skill 作为内部文档**

如果你的项目中有 skill 文件用于说明"如何在本项目中写 agent/skill"，而不是依赖 README，你正在实践 plugin-dev 的元级自引用设计，使 Claude 可以在创作新制品时直接加载这些规范。

### 4.2 挑战 / 验证

**挑战 1：创作痕迹污染（最容易发生的高影响缺陷）**

plugin-validator.md 的末尾残留是一类极为常见的缺陷：Claude 在帮助你生成或编辑 NL 制品时，可能会在文件末尾附上确认语句（"This is now complete."、"The skill is ready."等），而作者容易忽略这些行。

验证动作：
```bash
tail -5 agents/*.md skills/*.md
```
检查所有 NL 制品的最后 5 行，寻找任何以"Excellent"、"This is"、"The skill"、"Now you"、"I've created"、"Done!"等开头的行——这些都是创作痕迹，必须删除。

**挑战 2：颜色值合法性（机械错误，工具可检测）**

`color: pink` 是机械性错误，只要有校验工具就能 100% 捕获，但在无 linter 的项目中会静默失败。

验证动作：
```bash
grep -rn "^color:" agents/ --include="*.md"
```
对每一行确认颜色值是否为 `blue`/`cyan`/`green`/`yellow`/`magenta`/`red` 之一，其余均为非法值。

**挑战 3：跨插件命名空间冲突（checker 盲区）**

feature-dev 和 pr-review-toolkit 各自声明 `name: code-reviewer`，这类冲突在单插件检查中不会被发现，只有在跨插件全局检查时才能暴露。

验证动作（多插件项目）：
```bash
grep -rn "^name:" plugins/*/agents/*.md | awk -F: '{print $NF}' | sort | uniq -d
```
输出任何出现超过一次的 agent 名称，这些都是潜在的命名冲突。

**挑战 4：MCP 依赖的静默失败**

code-review.md 在 `allowed-tools` 中引用了一个未在 `plugin.json` 中声明的 MCP 方法。这类错误在 Claude Code 启动时不会报错，只在运行时尝试调用该工具时才失败，且错误消息可能不够明确。

验证动作：
```bash
# 提取 allowed-tools 中所有 mcp__ 前缀的工具名
grep -rn "mcp__" commands/*.md | grep -oP 'mcp__[a-z_]+__[a-z_]+'
# 对比 plugin.json 中声明的 mcpServers 键名
cat plugin.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('mcpServers', {}).keys())"
```
确认 `allowed-tools` 中引用的每个 `mcp__<server>__<method>` 都有对应的 `<server>` 在 `mcpServers` 中声明。

**挑战 5：hookify 的 fail-open 风险**

pretooluse.py 和 stop.py 中的 `except Exception` 在任何异常时放行工具调用或任务停止决策。对于安全相关的 hook（如拦截危险工具调用），fail-open 意味着当 hook 本身出错时，它的安全约束完全失效。

验证动作（适用于自己的 hook 脚本）：检查每个 hook 中的异常处理策略：`except Exception: pass` 或 `except Exception: allow` 等是 fail-open；`except Exception: deny/block` 是 fail-safe。对于安全相关的 pretooluse hook，应优先选择 fail-safe。

---

## 五、行动

### 5.1 自查动作

按优先级排列，适用于任何 Claude Code 插件作者：

**立即可做（5 分钟内）**

1. 检查所有 NL 制品文件末尾是否存在创作痕迹：
   ```bash
   for f in agents/*.md skills/*.md; do
     echo "=== $f (last 3 lines) ==="; tail -3 "$f"
   done
   ```
   删除任何以确认语句、感谢语或元评论结尾的行。

2. 验证 agent frontmatter 中的颜色值合法性：
   ```bash
   grep -rn "^color:" agents/ --include="*.md"
   ```
   对照合法值列表（blue/cyan/green/yellow/magenta/red）逐一确认。

3. 检查命令文件是否声明了 `allowed-tools`：
   ```bash
   grep -rL "allowed-tools" commands/ --include="*.md"
   ```
   输出所有缺少 `allowed-tools` 的命令文件，评估是否需要补充。

**短期任务（一周内）**

4. 对于多插件项目，运行跨插件命名冲突扫描：
   ```bash
   grep -rn "^name:" plugins/*/agents/*.md | awk -F: '{print $NF}' | sort | uniq -d
   ```
   对每个重复名称，决定是重命名其中一个（推荐加插件前缀，如 `feature-dev--code-reviewer` 和 `pr-review--code-reviewer`）还是合并两个 agent。

5. 为所有无示例 agent 补充 `<example>` 块：
   ```bash
   grep -rL "<example>" agents/ --include="*.md"
   ```
   对每个输出文件，补充至少 1 个包含"用户输入"和"预期输出格式"的具体示例。

6. 验证命令文件中引用的 MCP 工具是否均已在 `plugin.json` 中声明：
   ```bash
   grep -roh 'mcp__[a-z_]*' commands/*.md | sort -u
   ```
   对照 `plugin.json` 中的 `mcpServers` 键，确认每个引用均有对应声明。

**持续维护**

7. 在 CI 或 pre-commit hook 中加入 `nlpm-check`，自动捕获颜色值非法、frontmatter 缺失、模糊量词密度超标等机械性错误，避免人工审查的遗漏。

8. 为 hook 脚本建立异常处理策略文档：说明哪些 hook 应该 fail-open（如日志记录类 hook），哪些应该 fail-safe（如安全拦截类 hook），并在代码中添加注释说明选择理由。

### 5.2 灵感 → 实施路径

**灵感一：元级 skill 作为活文档**

plugin-dev 插件证明了一种反常识的设计：最好的插件编写指南不是 README，而是一个关于如何写插件的插件本身。当 Claude 正在帮你开发新 agent 时，它可以直接加载 `agent-development` skill 作为上下文，远比"去看 README 的第三节"更高效。

实施路径：
1. 识别你的项目中反复被问到的"如何做"问题（如："这个项目的 agent 应该怎么写 example 块？"）
2. 将答案编写为一个 skill 文件（如 `skills/how-to-write-agents.md`），包含规则、反例和 `<example>` 块
3. 在 `CLAUDE.md` 中声明"编写新 agent 前请先加载 `how-to-write-agents` skill"
4. 验证：让 Claude 在加载该 skill 后编写一个新 agent，检查是否遵循了 skill 中的规范

**灵感二：按审查维度拆分 agent**

pr-review-toolkit 的 6-agent 架构证明了一个反直觉的原则：单个"全功能代码审查 agent"通常不如 6 个专职审查 agent 的组合效果好。专职 agent 的 prompt 更短、上下文更聚焦、输出更可预测。

实施路径：
1. 列出你当前的"全功能"agent 实际执行的所有审查/分析维度（如代码质量、类型安全、测试充分性、简化机会）
2. 为每个维度创建一个独立的专职 agent，每个 agent 的 prompt 只描述该维度的标准和输出格式
3. 创建一个 orchestrator command，按顺序或并行调用这些专职 agent
4. 对比拆分前后的输出质量：专职 agent 通常在各自维度上发现更多且更精确的问题

**灵感三：hook 驱动的全局约束**

security-guidance 的 `security_reminder_hook.py` 证明了"全局约束用 hook 实现"的可行性：不需要在每个 agent prompt 中重复写安全要求，hook 会在每次工具调用前自动注入。

实施路径：
1. 识别你的项目中哪些约束是"所有 agent 都必须遵守"的（如：不能提交包含密钥的文件、所有输出必须包含置信度说明）
2. 将这些约束编写为 `pretooluse` 或 `userpromptsubmit` hook 脚本，在每次相关事件时检查或注入约束
3. 在 hook 的异常处理中明确选择 fail-open 还是 fail-safe，并在注释中记录理由
4. 在 `plugin.json` 中注册 hook，确保它在所有 agent 激活时均生效

**灵感四：循环任务的 stop-hook 编排**

ralph-wiggum 的 `stop-hook.sh` 提供了一种优雅的循环任务控制模式：通过 hook 返回值决定 Claude 是否继续任务，而不是依赖用户手动触发下一步。

实施路径：
1. 识别你的工作流中是否有"重复执行直到满足条件"的任务模式（如：持续监控 CI 状态直到通过）
2. 编写 `stop` hook 脚本，检查任务完成条件，返回"继续"或"停止"信号
3. 在 hook 中实现安全退出条件（最大迭代次数、超时时间），防止无限循环
4. 对 `$PROMPT` 变量等在 heredoc 中使用的变量进行引号保护，防止注入（参考 ralph-wiggum 的 Low 级安全发现，确保自己不重蹈覆辙）
