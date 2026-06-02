# disler/claude-code-hooks-mastery — 学习案例

**仓库**：https://github.com/disler/claude-code-hooks-mastery
**Stars**：3,566 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-02（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `manifest-discipline`, `experience-accumulation`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是目前 GitHub 上关注度最高的 Claude Code hooks 实战演示仓库之一，3,566 颗星。作者 disler 以「掌握 Claude Code 的所有 hook 事件」为核心命题，将 Claude Code 生命周期中的每一个可拦截点都写成了独立的 Python 脚本，并配套一套加密货币分析 multi-agent 系统作为「真实负载」来演示这些 hooks 在实际工作中的行为。

仓库的双轨结构一眼可辨：
1. **Hooks 轨**：13 个 Python hook 文件，覆盖 `PreToolUse`、`PostToolUse`、`Notification`、`Stop`、`SessionStart`、`SessionEnd`、`PreCompact`、`UserPromptSubmit`、`SubagentStart`、`SubagentStop`、`PermissionRequest`、`PostToolUseFailure` 等所有已知 Claude Code hook 事件；`settings.json` 中配置了完整的权限白名单和 hook 路由
2. **Agent 轨**：13 个薄 agent（`/` commands），每个 agent 通过 `Read and Execute` 模式委托给 `.claude/commands/agent_prompts/` 下的对应 prompt 文件；prompt 文件构成「核心知识库」，agent 只是入口层

关键事实：
1. CLAUDE.md 完全为空（0 行）——对于一个拥有复杂 hooks 架构和 13 个 agent 的仓库，这是最刺眼的矛盾
2. settings.json 是仓库里最优质的工件之一：权限白名单精细（`Bash(mkdir:*)`, `Bash(uv:*)` 等），hook 路由结构清晰，体现了作者在 hooks 领域的真实掌握
3. 多个 slash commands（cook.md, all_tools.md, update_status_line.md 等）缺少 frontmatter，导致 Claude Code 无法发现这些命令
4. 13 个加密货币 agent prompt 文件质量参差：顶部的 `crypto_research.md`（100/100）和 `crypto_research_haiku.md`（100/100）是教科书级写法，底部的薄 agent 文件则缺少示例和输出格式

### 1.2 架构剖析

目录结构（关键部分）：

```
claude-code-hooks-mastery/
  .claude/
    settings.json              # 权限白名单 + hook 路由（最优质工件）
    commands/
      cook.md                  # slash command，缺 frontmatter（BUG）
      all_tools.md             # slash command，缺 frontmatter（BUG）
      update_status_line.md    # slash command，缺 frontmatter（BUG）
      cook_research_only.md    # slash command，缺 frontmatter（BUG）
      prime.md                 # slash command，有 frontmatter（正常）
      build.md                 # slash command，有 frontmatter（正常，95/100）
      question.md              # slash command，有 frontmatter（正常，96/100）
      ...
      agent_prompts/           # prompt 文件库（无 frontmatter，纯内容）
        crypto_research.md     # 100/100
        crypto_research_haiku.md  # 100/100
        meta-agent.md          # 83/100
        crypto_investment_plays_agent_prompt.md  # 68/100
        ...
  hooks/
    pre_tool_use.py
    post_tool_use.py
    session_start.py
    session_end.py
    setup.py
    stop.py
    notification.py
    pre_compact.py
    user_prompt_submit.py
    subagent_start.py
    subagent_stop.py
    permission_request.py
    post_tool_use_failure.py
  CLAUDE.md                    # 完全为空（0 行，严重 BUG）
  package.json                 # 未锁定 semver（低风险）
```

- **文件类型分布**：0 个 skill，0 个 agent（严格意义上），14 个 slash commands，13 个 Python hook 脚本，1 个 settings.json，0 个 plugin.json（非 marketplace 插件，直接使用）
- **编排关系**：slash command → `Read and Execute: .claude/commands/agent_prompts/<file>.md` → 执行 prompt 文件内容。这是一种「薄 agent 委托」模式——command 文件是路由器，prompt 文件是实际执行者
- **Hook 架构**：settings.json 中每个 hook 事件类型（PreToolUse, PostToolUse 等）路由到对应的 Python 脚本。脚本之间通过 `hooks/utils/`、`hooks/validators/`、`hooks/llm/`、`hooks/tts/` 等工具库共享功能
- **跨件脆弱点**：agent prompt 文件路径被硬编码在各 slash command 里（如 `Read and Execute: .claude/commands/agent_prompts/crypto_research.md`）。如果 prompt 文件被重命名，调用方会静默失败，没有任何错误提示

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「用真实用例驱动 hooks 教学」——不写抽象的 hooks 示例，而是用一个完整的加密货币分析系统来承载所有 hook 事件的演示，让读者看到 hooks 在真实 agent 负载下的行为
- **解决什么问题**：Claude Code hook 事件文档化程度不足，开发者不知道每个 hook 事件的触发时机、可操作数据和实际用途。本仓库通过 13 个各司其职的 Python 脚本，给每个 hook 事件配了一个活的示例实现
- **做了什么 trade-off**：作者把精力集中在 hooks 实现的深度和广度上，代价是 slash command 的规范性（frontmatter 缺失）和 CLAUDE.md 内容（完全空置）。这是一个「技术精度 vs 规范完整性」的权衡——对于一个以「展示 hooks 能力」为目标的演示仓库，作者隐式地接受了规范层面的缺陷
- **反映什么认知模型**：作者把这个仓库定位为「hooks 工程师的参考实现」，而非「可发布的 Claude Code 插件」。settings.json 的质量（极其完整的权限白名单）和 hook 脚本的覆盖度（所有事件类型全覆盖）表明作者对 Claude Code 内部机制有深度理解；但 CLAUDE.md 空置和 frontmatter 缺失说明作者对 Claude Code 规范层（NLPM 管理的部分）关注不足

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「薄 agent 委托 + 全事件 hooks 覆盖」（Thin-Agent Delegation + Full-Event Hook Coverage）**

这个模式的核心特征：slash command 只作为入口路由器，实际任务逻辑完全委托给独立的 prompt 文件；与此同时，Python hook 脚本覆盖了 Claude Code 生命周期的所有已知事件类型，形成一张完整的「副作用拦截网」。

模式特征清单（5 条）：
- 特征 1：**薄 command 层**——slash command 文件只有寥寥几行，核心逻辑全在 `agent_prompts/` 的 prompt 文件里，command 层仅做「读取并执行」的路由
- 特征 2：**全事件 hook 覆盖**——不选择性覆盖，而是为所有已知 Claude Code hook 事件都提供实现，形成完整的生命周期感知能力
- 特征 3：**分层工具库**——hook 脚本依赖 `utils/`、`validators/`、`llm/`、`tts/` 四个子模块，而非把所有逻辑写在单文件里，体现了工程纪律
- 特征 4：**settings.json 精细白名单**——用 `Bash(mkdir:*)` 级别的细粒度路径模式（而非 `Bash(*)` 全放行）定义权限边界，把最小权限原则落实到工具调用层
- 特征 5：**质量双极分布**——同一仓库里最高质量工件（`crypto_research.md` 100/100, `settings.json`）和最低质量工件（空 CLAUDE.md, 无 frontmatter commands）并存，反映「核心功能精品化，规范层不重视」的开发取向

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 学习 Claude Code hook 事件的触发机制和数据结构 | ✅ 极高适用 | 13 个 hook 脚本覆盖所有事件，是现存最完整的参考实现 |
| 需要在 agent 运行时插入副作用（TTS 播报、状态更新、权限审查）| ✅ 高度适用 | stop.py、notification.py、permission_request.py 等提供了直接可复用的实现模板 |
| 构建 multi-agent 系统的组织模式参考 | ✅ 适用 | 薄 agent 委托模式降低了 command 文件的维护负担，prompt 文件可独立演进 |
| 作为可发布到 Claude Code marketplace 的插件 | ❌ 不适用 | 无 plugin.json，多个 commands 缺 frontmatter，CLAUDE.md 为空，安全评级 REVIEW |
| 需要零安全风险的安装环境（企业内网等）| ❌ 不适用 | 9 处 Medium 安全风险（outbound API 调用、自动包安装、rm -rf 指令）需要人工审查 |
| 作为新手了解 Claude Code 规范（frontmatter、CLAUDE.md）| ⚠️ 有限适用 | 仓库本身违反了多项规范，容易给新手形成错误示范 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 薄 agent 委托 + 全事件 hook 覆盖（本仓库）| disler/claude-code-hooks-mastery | Hook 覆盖最完整，settings.json 白名单最精细，是 hooks 工程的标杆参考 | CLAUDE.md 空置，多处 frontmatter 缺失，安全执行面大（9 Medium） |
| 单 Skill 极简原则 | forrestchang/andrej-karpathy-skills | 零安全风险，99/100 分，维护成本极低 | 无任务执行能力，无 hooks，不适合复杂 agent 系统 |
| 完整功能插件（commands + agents + skills）| MarkQWu/echo-sleuth-for-claude | 可发布 marketplace，结构规范，commands 全有 frontmatter | 无 hooks，model 字段缺失，复杂度高 |
| 规则集型仓库 | feiskyer/claude-code-settings | 规范层完整（CLAUDE.md 充实），易于理解 | 无 hooks，无 agent 编排，功能单一 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 完全为空。**改进做法**：补充至少三节内容：①项目目的（「这是 Claude Code hooks 全事件参考实现，配套加密货币分析演示负载」）；②Hook 架构概述（列出 13 个事件类型及对应脚本路径）；③使用前提（需要设置的环境变量、API keys）。**预期收益**：NLPM 评分从 65 升至 85+，消除 -25 分最大单项扣分，并为新接触仓库的用户提供导航入口。

2. **当前问题**：cook.md、all_tools.md、update_status_line.md、cook_research_only.md 缺少 frontmatter，Claude Code 无法通过 `/` 键发现这些命令。**改进做法**：为每个 command 文件补充 `---\ndescription: <一句话描述>\nallowed-tools: Bash, Read\n---` frontmatter 块。**预期收益**：命令在 Claude Code 命令面板中可见，消除「命令不可发现」的静默失败。

3. **当前问题**：cook.md 调用 `crypto-coin-analyzer` 时未指定 haiku/sonnet/opus 变体，3 个变体同时存在，选择不确定。**改进做法**：在 cook.md 的调用处明确指定 `crypto-coin-analyzer-haiku`（用于速度）或 `crypto-coin-analyzer-sonnet`（用于质量），添加注释说明选择依据。**预期收益**：消除「模糊调度」跨件一致性问题，行为可预期。

4. **当前问题**：`sentient.md` 的 description 写「Manage, organize and ships your codebase」，但实际内容包含 `rm -rf` 指令，功能描述与实际行为严重不符。**改进做法**：将 description 改为「Aggressive codebase cleanup including directory removal — review before use」，诚实反映实际风险。**预期收益**：消除安全和一致性双重问题；Medium 安全项因描述修正而降级。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 整体 NL Score：**77/100**，安全评级：**REVIEW**（9 Medium, 1 Low）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 65 | 基本为空（1 行），缺项目上下文，-25 分最大扣项 |
| crypto_investment_plays_agent_prompt.md | 68 | 无 frontmatter，[模糊量词](#vague-quantifier) |
| crypto_coin_analyzer_agent_prompt.md | 68 | 无 frontmatter，[模糊量词](#vague-quantifier) |
| cook.md | 70 | 完全无 frontmatter |
| prime_tts.md | 70 | 无 frontmatter |
| all_tools.md | 70 | 无 frontmatter |
| update_status_line.md | 70 | 无 frontmatter |
| 5 个加密 agent_prompts | 70 各 | 无 frontmatter |
| cook_research_only.md | 70 | 无 frontmatter |
| 13 个加密 agent | 72-75 | 无示例（-15 各），无输出格式（-10 各） |
| meta-agent.md | 83 | — |
| validator.md / builder.md | 85 | — |
| plan.md / plan_w_team.md | 85 | 无 allowed-tools，[模糊量词](#vague-quantifier) |
| git_status.md / prime.md | 90 | — |
| sentient.md | 90 | description 误导性（安全问题） |
| build.md | 95 | — |
| question.md | 96 | — |
| crypto_research_haiku.md | 100 | 无问题，教科书级 |
| crypto_research.md | 100 | 无问题，教科书级 |

**安全发现摘要（9 Medium, 1 Low）**：

| 级别 | 文件 | 风险描述 |
|---|---|---|
| Medium | sentient.md | 指令含 `rm -rf` 3 次；description「管理仓库」严重低估实际风险 |
| Medium | setup.py | session 启动时自动运行 `npm ci/install`、`pip install`、`uv sync`，无用户确认 |
| Medium | anth.py | 每次 Stop hook 触发时向 Anthropic API 发送出站请求 |
| Medium | oai.py | 每次 Stop hook 触发时向 OpenAI API 发送出站请求 |
| Medium | openai_tts.py | 任务完成时自动调用 OpenAI TTS 接口 |
| Medium | elevenlabs_tts.py | 任务完成时自动调用 ElevenLabs TTS 接口 |
| Medium | session_start.py | 每次 session 启动时使用环境 GitHub credentials 执行 `gh issue list` |
| Low | package.json | 依赖版本使用 `^` 未锁定（semver 浮动） |

**质量问题汇总（35 处）**：
- 6 个 agent 缺 model 声明
- 所有 13 个薄加密 agent：无示例（-15 各）+ 无输出格式（-10 各）
- 6 个 Write 声明但未使用（加密市场/币种 agent 各 -3）
- plan.md / plan_w_team.md：无 allowed-tools，含[模糊量词](#vague-quantifier)
- sentient.md：误导性 description
- CLAUDE.md：内容为空（-25）

### 3.2 当时值得借鉴的模式

1. **settings.json 细粒度权限白名单** → 为什么好：不用 `Bash(*)` 全放行，而是把每条允许的 Bash 命令都写成精确的路径模式（`Bash(mkdir:*)`, `Bash(uv:*)`, `Bash(git:*)` 等）。这意味着 Claude 无法执行白名单外的任何 Bash 命令，最小权限原则在工具调用层得到落实 → 原文路径：`.claude/settings.json` 的 `permissions.allow` 数组 → 如何借鉴：在自己的 settings.json 里逐条审查 allow 列表，把 `Bash(*)` 拆解为实际需要的最小命令集。

2. **全事件 hook 覆盖架构** → 为什么好：13 个 hook 脚本为每种事件类型都提供了一个活的参考实现，开发者可以按需取用任意事件的处理模板，而不必从头摸索触发条件和数据格式 → 原文路径：`hooks/` 目录下的 13 个 `.py` 文件 → 如何借鉴：把本仓库的 hook 文件当作「Claude Code hook 事件手册的可执行版本」，需要处理某个事件时直接参考对应脚本的结构。

3. **hook 脚本分层工具库** → 为什么好：`hooks/utils/`、`hooks/validators/`、`hooks/llm/`、`hooks/tts/` 四个子模块把共享逻辑集中管理，避免了「每个 hook 脚本各自维护重复代码」的碎片化 → 原文路径：`hooks/` 目录结构 → 如何借鉴：在自己的 hooks 实现中，把跨脚本复用的逻辑（日志格式、Claude API 调用、输入校验）提取到 `hooks/utils/` 级别，而非内联在每个事件处理脚本里。

4. **crypto_research.md 教科书级 prompt 文件** → 为什么好：100/100 分，有完整 frontmatter（虽然是 slash command 格式而非 prompt 文件格式），有清晰的步骤、工具声明、输出格式和内置校验逻辑 → 原文路径：`.claude/commands/agent_prompts/crypto_research.md` → 如何借鉴：用 `crypto_research.md` 的结构作为「我的复杂 agent prompt 应该长什么样」的参照，特别是它的工具列表声明方式和输出格式节。

### 3.3 当时的缺陷

1. **13 个 slash command 缺 frontmatter**：cook.md、all_tools.md、update_status_line.md、cook_research_only.md 等完全没有 YAML frontmatter 块。为什么会失败：Claude Code 在 `/` 触发命令面板时，需要读取 frontmatter 中的 `description` 字段来展示命令名称和说明。没有 frontmatter 的 command 文件对 Claude Code 不可见——用户无法通过 `/` 发现并调用这些命令，只能靠直接说「执行 cook」来绕过。**自查**：`find .claude/commands -name "*.md" | xargs grep -rL "^---"` → 命中文件即为缺 frontmatter 的命令。

2. **CLAUDE.md 完全为空**：一个拥有 13 个 hook 脚本、13 个 agent、复杂 settings.json 权限体系的仓库，CLAUDE.md 是空文件。为什么会失败：CLAUDE.md 在每次 Claude Code session 启动时自动加载，是「项目级上下文」的唯一载体。空文件意味着 Claude 在这个项目里工作时，完全不知道项目是什么、有哪些 hooks 在运行、有哪些约定需要遵守——而这些信息对于 hooks 这种会在后台悄悄执行的系统尤其重要。**自查**：`wc -l CLAUDE.md` → 输出 0 即为空文件。

3. **cook.md 调用 `crypto-coin-analyzer` 未指定变体**：仓库中存在 `crypto-coin-analyzer`、`crypto-coin-analyzer-haiku`、`crypto-coin-analyzer-sonnet` 三个变体，但 cook.md 在调用时没有指定具体是哪一个。为什么会失败：Claude 在模糊调度时会自行选择，选择依据不可预期，导致同样的 cook 请求可能使用不同的 agent，行为不一致。**自查**：`grep -n "crypto-coin-analyzer" .claude/commands/cook.md` → 查看调用处是否包含明确的变体后缀。

4. **薄 agent 委托的静默断裂风险**：所有 13 个加密 agent command 都用硬编码路径委托给 `agent_prompts/` 下的文件（如 `Read and Execute: .claude/commands/agent_prompts/crypto_research.md`）。没有任何机制验证被委托的文件仍然存在。为什么会失败：如果 prompt 文件被重命名或删除，agent command 会在运行时静默失败，没有错误提示，用户只会看到 Claude 「不知道该做什么」。**自查**：`grep -rn "Read and Execute" .claude/commands/*.md | grep -v "agent_prompts"` → 排查非 agent_prompts 路径的委托。

### 3.4 当时的优化机会

1. 为所有缺 frontmatter 的 command 文件补充最小 frontmatter（`description` + `allowed-tools`），13 处 slash command 恢复可发现性；约 30 分钟。
2. 补充 CLAUDE.md，3 节内容（项目目的、hooks 架构概述、环境要求），消除 -25 分最大扣项；约 20 分钟。
3. 将 cook.md 的 `crypto-coin-analyzer` 调用改为明确指定变体，消除「模糊调度」跨件一致性问题；约 5 分钟。
4. 将 sentient.md 的 description 改为诚实反映 `rm -rf` 风险的描述，同步消除安全 Medium 项和 checker 一致性警告；约 5 分钟。
5. 在 agent prompt 文件末尾增加 `## Output Format` 节，13 个薄 agent 各节约 -10 分扣项；约 2 小时。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| cook.md 缺 frontmatter | `head -5 .claude/commands/cook.md` | **持续存在**：仍以 `Run these 7 sub agent tasks...` 开头，无 `---` 块 | Audit 后 2 个月未修复，维护者接受此状态 |
| prime_tts.md 缺 frontmatter | `head -5 .claude/commands/prime_tts.md` | **已修复**：现有 `---\nallowed-tools: Bash, Read\ndescription: Load context...` | 这是 Audit 后仅有的 frontmatter 修复之一 |
| all_tools.md 缺 frontmatter | `head -3 .claude/commands/all_tools.md` | **持续存在**：以 `# List All Tools` 开头 | 未修复 |
| update_status_line.md 缺 frontmatter | `head -3 .claude/commands/update_status_line.md` | **持续存在**：以 `# Update Status Line Data` 开头 | 未修复 |
| cook_research_only.md 缺 frontmatter | `head -3 .claude/commands/cook_research_only.md` | **持续存在**：以 `Run these 7 sub agent tasks...` 开头 | 未修复 |
| llm-ai-agents-and-eng-research.md WebSearch 未声明 | `grep -n "tools\|WebSearch\|firecrawl" .claude/commands/agent_prompts/llm-ai-agents-and-eng-research.md` | **部分修复**：tools 改为 `mcp__firecrawl-mcp__firecrawl_search` 和 `WebFetch`，实际工具已切换；但正文第 21 步仍写「Use WebSearch」——文档未同步更新 | 功能修复但文档滞后，`step 21` 的 `WebSearch` 措辞会误导用户 |
| CLAUDE.md 完全为空 | `wc -l CLAUDE.md` | **持续存在**：当前仍为 0 行（完全空文件）| Audit 后 2 个月无任何内容写入，最大单项扣分项仍在 |

**修复率**：13 个已知 bug 中，1 个完全修复（prime_tts.md），1 个部分修复（WebSearch 工具声明），11 个持续存在。CLAUDE.md 空置是最直观的「维护者优先级」信号——hooks 工程精度高，规范层不在维护计划内。

### 4.2 架构演进

从 audit 时（2026-04-06）到当前 HEAD（2026-06-02），hook 架构没有变化。13 个 hook 脚本文件名和 settings.json 结构保持稳定。唯一可观察到的变化：
- `prime_tts.md` 补充了 frontmatter（唯一的规范修复）
- `llm-ai-agents-and-eng-research.md` 中工具从 WebSearch 切换到 firecrawl_search + WebFetch（功能升级，追随 MCP 生态变化）

**作者后来意识到的**：从 firecrawl 替换 WebSearch 这一变化来看，作者持续在跟踪 Claude Code 的 MCP 工具生态演进并更新工具声明，但对规范层（frontmatter、CLAUDE.md）的关注度始终低于工具层。这进一步证实了「这是一个以展示 hooks 技术能力为目标的参考仓库」，而非追求规范完整性的 marketplace 插件。

### 4.3 新增的可学习模式

**MCP 工具切换模式**：从 WebSearch 到 `mcp__firecrawl-mcp__firecrawl_search` 的迁移展示了「工具声明随 MCP 生态演进而更新」的必要性。MCP 工具名称格式为 `mcp__<server>__<tool>`，与内建工具（`Bash`, `Read`, `WebSearch`）格式不同。当 MCP 服务器版本更新或工具名变化时，所有声明了该工具的 agent/command 文件都需要同步更新——这是薄 agent 委托模式下的「工具名称漂移」风险。

**自查建议**：`grep -rn "mcp__" .claude/ | cut -d: -f3 | sort -u` → 列出所有已声明的 MCP 工具，交叉验证当前 settings.json 中实际启用的 MCP 服务器列表。

---

## 五、校准

### 5.1 我已经在做对的

1. **Commands 全部有 frontmatter**：echo-sleuth-for-claude 的 8 个 command 文件均有完整 frontmatter，`grep -rL "^---" .claude/commands/` 返回 0 命中——这是与本仓库最显著的正向差异。disler 的 4 个核心 command 因缺 frontmatter 而不可发现，而我的所有 command 都可以通过 `/` 键正常触发。

2. **有实质性 CLAUDE.md**：我的 echo-sleuth-for-claude 有内容的 CLAUDE.md，而本仓库的 CLAUDE.md 是 0 行空文件。CLAUDE.md 是 Claude 每次 session 的第一个上下文来源，空置意味着 Claude 在不知道项目是什么的情况下工作。

3. **无 sentient 级别的高风险描述问题**：我的仓库没有「description 说一件事，实际执行另一件风险更高的事」的工件。description 的准确性是用户信任的基础。

### 5.2 挑战 / 验证

- **这次案例修正了「有 hooks = 高质量插件」的直觉**：本仓库有 3,566 颗星和 13 个 hook 脚本，但 NLPM 评分仅 77/100，安全评级 REVIEW。技术实现的深度（hooks 全覆盖）和规范层的质量（frontmatter、CLAUDE.md）是两个独立维度，不能互相代替。高 star 数反映的是「hooks 教学价值」，而非「规范完整性」。

- **薄 agent 委托模式的隐患比预想的更大**：13 个 agent 通过硬编码路径委托给 prompt 文件，没有任何验证机制。这个模式在「只有作者维护」的场景下运转良好，但一旦多人协作或文件重组，静默失败的风险极高。我的 echo-sleuth 目前没有用这个模式，是一个正确的隐式决策。

- **全事件 hook 覆盖是我完全缺失的能力**：echo-sleuth-for-claude 完全没有 hooks。本仓库证明了 hooks 可以实现的价值范围（TTS 播报、session 状态管理、权限拦截、工具调用监控），这些能力在我的仓库里是白板。这不是因为我做了正确的 trade-off，而是因为我从未实现过 hooks。

- **settings.json 的细粒度白名单是我需要改进的**：我目前的 settings.json 如果有 `Bash(*)` 全放行项，应该拆解为具体命令白名单。本仓库的白名单实践是最小权限原则在 Claude Code 工具层的标准参考。

---

## 六、行动

### 6.1 自查动作

```bash
# 【检查 1】发现我的 commands 中缺 frontmatter 的文件（复现本仓库 Bug#1~5）
find .claude/commands -name "*.md" | xargs grep -rL "^---" 2>/dev/null
# 预期输出：无命中（我的 commands 已全有 frontmatter）
# 如有命中：补充最小 frontmatter（description + allowed-tools）

# 【检查 2】检查 CLAUDE.md 是否为空
wc -l CLAUDE.md
# 预期输出：>10（有实质内容）
# 如输出 0：立即补充项目概述、架构说明、使用约定三节

# 【检查 3】发现 agents/ 中缺 model 字段的文件（复现本仓库 Bug：6 agent 缺 model）
grep -rL "^model:" agents/ 2>/dev/null
# 命中后：补充 model 字段，推荐 claude-sonnet-4-5 或 claude-haiku-4-5

# 【检查 4】发现 tool 声明与正文不一致（复现本仓库 Bug#13 WebSearch 未声明）
# 步骤 1：列出所有声明了 tools 的文件及其工具列表
grep -rn "^tools:" .claude/commands/ agents/ 2>/dev/null
# 步骤 2：对每个命中文件，检查正文是否引用了 tools 列表外的工具
# 例如：文件 tools 声明了 Bash 和 Read，但正文说「用 WebSearch 搜索」
grep -rn "WebSearch\|mcp__" .claude/commands/ agents/ 2>/dev/null | grep -v "^.*tools:"

# 【检查 5】检查 settings.json 是否有过于宽泛的 Bash(*) 全放行项
python3 -c "
import json
data = json.load(open('.claude/settings.json'))
allows = data.get('permissions', {}).get('allow', [])
for item in allows:
    if item in ('Bash(*)', 'Bash') or item == 'Bash(*)':
        print(f'TOO_BROAD: {item}')
    elif item.startswith('Bash(') and '*' in item:
        print(f'CHECK: {item}')
" 2>/dev/null
# 命中 TOO_BROAD 后：拆解为实际需要的最小命令集

# 【检查 6】检查薄 agent 委托的目标文件是否都存在（复现静默断裂风险）
grep -rn "Read and Execute" .claude/commands/ 2>/dev/null | while read line; do
    file=$(echo "$line" | grep -oP '(?<=Read and Execute: )\S+')
    [ -n "$file" ] && [ ! -f "$file" ] && echo "MISSING TARGET: $file"
done

# 【检查 7】检查 description 是否与实际内容存在潜在误导（复现 sentient.md 问题）
# 查找含高风险操作关键词的文件，交叉核对其 description
grep -rln "rm -rf\|format\|delete.*all\|wipe" .claude/commands/ agents/ 2>/dev/null | while read f; do
    desc=$(grep "^description:" "$f" | head -1)
    echo "$f => $desc"
done

# 【检查 8】检查 agent prompt 中的模糊量词
grep -rn "as needed\|when appropriate\|if necessary\|where applicable\|as required\|relevant" \
  .claude/commands/ agents/ skills/ 2>/dev/null | grep -v "^Binary"
# 命中后：将「as needed」替换为具体条件，如「only when the user has not specified a format」
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth-for-claude 引入最小 hooks 架构（从 Stop 事件开始）
   - **为何可行**：本仓库证明了 Stop hook 是最低成本的起点——任务完成时触发，可以做状态日志、结果摘要播报等「完成后动作」，对主流程零干扰
   - **第一步**：创建 `hooks/stop.py`，实现「任务完成时向终端输出一行简洁的完成摘要」；在 `.claude/settings.json` 的 `hooks` 节增加 `{"event": "Stop", "command": "python3 hooks/stop.py"}`；约 30 分钟

2. **想法**：把本仓库的 settings.json 权限白名单模式迁移到自己的仓库
   - **为何可行**：echo-sleuth 目前的 settings.json 权限设置可能过于宽泛。本仓库展示了「每条 Bash 命令都写成精确路径模式」的细粒度白名单是可实现且可维护的
   - **第一步**：导出当前 settings.json 的 allow 列表，对照 echo-sleuth 实际使用的命令（git, python3, jq 等），把 `Bash(*)` 替换为具体的 `Bash(git:*)`, `Bash(python3:*)` 等；约 20 分钟

3. **想法**：为 echo-sleuth 的 5 个 agent 补充 model 字段
   - **为何可行**：本仓库的 audit 数据显示「6 个 agent 缺 model 声明」是质量扣分项之一。我的 echo-sleuth 也有同样问题（5/5 agent 缺 model 字段）。补充 model 字段可以明确 agent 的性能/成本定位，避免每次由 Claude 自行决定使用哪个模型
   - **第一步**：`grep -rL "^model:" agents/` 找出所有缺 model 的 agent，按任务复杂度分配：简单机械任务（file-historian, schema-scout）→ `claude-haiku-4-5`；复杂分析任务（analyze, memory-auditor, recall）→ `claude-sonnet-4-5`；约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：用户仓库 `MarkQWu/echo-sleuth-for-claude`、`MarkQWu/drama-workshop-skills`、`MarkQWu/claude-for-legal`

### 8.1 目的对齐度

本案例 disler/claude-code-hooks-mastery 的核心目的：展示 Claude Code 所有 hook 事件类型的参考实现，配套加密货币分析 multi-agent 系统作为演示负载。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中高 | 同为 Claude Code agent 套件；同有 commands 和 agents；同为功能型插件而非行为约束型 | echo-sleuth 无 hooks；有 CLAUDE.md；commands 全有 frontmatter；规范层质量更高 | 极高——hooks 架构是 echo-sleuth 完全缺失的能力层 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 生态产品 | drama-workshop 是单 skill 仓库，无 hooks，无 multi-agent 编排，领域（戏剧创作）完全不同 | 低——架构差异过大 |
| MarkQWu/claude-for-legal | 低 | 同为多 agent 设计；有 agents/ 目录 | claude-for-legal 面向法律 workflow，领域和使用者完全不同；安全执行面更小 | 中——settings.json 白名单模式可借鉴 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | echo-sleuth 命中情况 | 严重度 |
|---|---|---|---|
| Commands 缺 frontmatter | `find .claude/commands -name "*.md" \| xargs grep -rL "^---"` | **0 命中**（echo-sleuth 8/8 commands 均有 frontmatter）——比 disler 好 | 无 |
| CLAUDE.md 为空 | `wc -l CLAUDE.md` | **未命中**（echo-sleuth 有 CLAUDE.md 内容）——比 disler 好 | 无 |
| Agents 缺 model 字段 | `grep -rL "^model:" agents/` | **5/5 命中**：analyze.md, file-historian.md, memory-auditor.md, recall.md, schema-scout.md 全部缺 model | 中（-5 分/处，共 -25 分） |
| 模糊量词 | `grep -rn "as needed\|when appropriate\|if necessary" .claude/ agents/ skills/` | 待本地验证；过去案例数据显示 echo-sleuth 有「relevant」量词 | 中（-2 分/处） |
| 完全无 hooks | `ls hooks/ 2>/dev/null \|\| echo MISSING` | **完全缺失**：echo-sleuth 无任何 hook 脚本，也无 settings.json 的 hooks 节 | 高（功能能力空白，非规范扣分） |

**命中后的具体行动建议**：
- echo-sleuth agents 缺 model 字段（5/5）→ 按任务复杂度分配：file-historian + schema-scout 用 haiku（机械扫描任务），analyze + memory-auditor + recall 用 sonnet（推理分析任务）；`grep -rL "^model:" agents/` 确认命中列表后逐一补充；约 15 分钟
- echo-sleuth 完全无 hooks → 从 Stop 事件开始，实现「任务完成时输出一行完成摘要」；这是最低风险的 hook 起点，不需要 API key，不修改主流程

### 8.3 别人的更优方案

1. **领域**：Claude Code 全事件 hook 覆盖
   - **本案例做法**：13 个 Python 脚本覆盖 PreToolUse、PostToolUse、Stop、SessionStart、UserPromptSubmit、SubagentStart 等所有已知事件；每个脚本独立，通过共享工具库（`hooks/utils/`、`hooks/llm/`）复用公共逻辑
   - **我的项目现状**：echo-sleuth-for-claude 无任何 hook 实现，无 `hooks/` 目录，settings.json 中无 hooks 节
   - **如何借鉴**：参照本仓库 `hooks/stop.py`（最简单，无外部依赖）作为第一个 hook 实现；然后根据 echo-sleuth 的实际需求（例如：分析任务完成后自动追加到分析日志）增加针对性 hook；不必全覆盖 13 个事件

2. **领域**：settings.json 细粒度权限白名单
   - **本案例做法**：`Bash(mkdir:*)`, `Bash(uv:*)`, `Bash(git:*)`, `Bash(python3:*)` 等精确路径模式，而非 `Bash(*)` 全放行；每条规则都有实际对应的使用场景
   - **我的项目现状**：待核查；echo-sleuth 的 settings.json 可能存在过于宽泛的 Bash 权限项
   - **如何借鉴**：运行检查 5（见 §6.1），找出 `Bash(*)` 类宽泛项，拆解为实际使用的最小命令集；安全性提升，且审计时更容易理解每条权限的来源

3. **领域**：prompt 文件分层（slash command 层 + agent_prompts 内容层）
   - **本案例做法**：slash command 文件保持极简（仅含路由指令），实际执行逻辑在独立的 agent_prompts 文件中；两者分离，内容层可以独立演进
   - **我的项目现状**：echo-sleuth 的 commands 把路由和执行逻辑混在同一文件
   - **如何借鉴**：对于复杂度超过 50 行的 command 文件，考虑将核心任务描述提取到独立的 `instructions/` 目录，command 文件仅保留 frontmatter + 一行 `Read and Execute` 指令

### 8.4 反向：我的项目做得比他们好的地方

1. **Commands 全部有 frontmatter（8/8 vs disler 的 4 缺失）**：echo-sleuth 的所有命令都可以通过 Claude Code 命令面板正常发现和触发。disler 的 cook.md、all_tools.md 等核心命令因缺 frontmatter 而在 UI 中不可见。这是规范纪律的直接体现。

2. **有实质性 CLAUDE.md（vs 完全空文件）**：echo-sleuth 的 CLAUDE.md 描述了项目目的和使用约定，为每次 session 提供了最基础的项目上下文。disler 的 CLAUDE.md 是 0 行空文件——这对于一个有 13 个 hook 脚本在后台运行的复杂系统来说是显著的上下文缺失。

3. **无误导性 description（vs sentient.md 的 rm -rf 问题）**：echo-sleuth 的所有 agent 和 command 的 description 准确反映实际功能，没有「说管理仓库、实际删文件」的问题。准确的 description 是用户信任的基础，也是安全审查的第一道筛选。

---

## 八、术语表

**frontmatter**
YAML 格式的文件头部元数据块，以 `---` 开始和结束。在 Claude Code 的 slash command 文件（`.claude/commands/*.md`）中，frontmatter 包含 `description`（命令面板显示文本）和 `allowed-tools`（可使用的工具列表）等字段。缺少 frontmatter 的 command 文件对 Claude Code 的命令发现机制不可见——用户无法通过 `/` 键找到并触发该命令。

**薄 agent 委托（Thin-Agent Delegation）**
一种 slash command 设计模式：command 文件本身只有寥寥几行（通常是 `Read and Execute: <path>`），实际的任务执行逻辑完全委托给另一个独立的 prompt 文件。优点是 command 层保持稳定，内容层可独立演进；缺点是两者之间是硬编码路径依赖，目标文件被重命名或删除时会静默失败，没有任何错误提示。

**全事件 hook 覆盖（Full-Event Hook Coverage）**
Claude Code 支持在工具调用前后（PreToolUse/PostToolUse）、任务完成（Stop）、session 启动/结束（SessionStart/SessionEnd）、用户输入（UserPromptSubmit）、子 agent 生命周期（SubagentStart/SubagentStop）、权限请求（PermissionRequest）等关键时间点注入自定义 Python 脚本。全事件覆盖指为所有已知事件类型都提供实现，形成完整的「副作用拦截网」。

**模糊量词（Vague Quantifier）**
在 agent prompt 或 skill 文件中使用「as needed」「when appropriate」「if necessary」「relevant」等无法精确判断的限定词。问题在于：Claude 在解释这类词时会基于上下文自行发挥，不同上下文下会产生不一致的行为。NLPM 规范要求将模糊量词替换为可验证的具体条件，例如将「as needed」改为「only when the user's request contains a currency symbol（$, €, ¥）」。

**CLAUDE.md**
Claude Code 在每次 session 启动时自动加载的项目级配置文件。是「始终生效的项目上下文」的标准载体，每次对话开始前 Claude 都会读取它。对于拥有复杂 hooks 架构的仓库，CLAUDE.md 是告知 Claude「有哪些 hook 在后台运行、有哪些约定需要遵守」的唯一自动化通道。空置 CLAUDE.md 意味着 Claude 在不知道项目背景的情况下工作。

**settings.json 细粒度白名单（Fine-Grained Allowlist）**
在 `.claude/settings.json` 的 `permissions.allow` 数组中，将 Bash 权限写成精确的命令路径模式（如 `Bash(git:*)`, `Bash(python3:*)`, `Bash(mkdir:*)`），而非使用 `Bash(*)` 全放行。细粒度白名单落实了最小权限原则：Claude 只能执行白名单中明确列出的命令，无法执行未声明的 Bash 操作，即便有 hook 脚本在后台运行也不例外。

**安全执行面（Security Execution Surface）**
仓库中所有可以在用户系统上执行代码的工件的集合，包括 hook 脚本（.py 文件）、setup 脚本、MCP 服务器配置、package.json 依赖等。执行面越大，用户在安装前需要审查的内容越多，信任门槛越高。本仓库有 13 个 Python hook 文件，其中多个会在 session 启动或任务完成时自动向外部 API（Anthropic, OpenAI, ElevenLabs）发出网络请求，安全执行面属于同类仓库中最大的一档。

**静默断裂（Silent Failure）**
当薄 agent 委托的目标文件不存在时，Claude Code 不会抛出明显错误，而是 Claude 会「困惑地」不知道该做什么。对比「显式失败」（直接报告缺失文件路径），静默断裂更难调试，因为用户看到的只是「Claude 没有完成任务」，而不是「文件 X 不存在」。预防方法：在 CI 或预提交 hook 中验证所有硬编码路径的目标文件存在。

**模型声明（Model Declaration）**
在 agent 或 command 的 frontmatter 中通过 `model:` 字段指定执行该任务的 Claude 模型（如 `claude-haiku-4-5` 或 `claude-sonnet-4-5`）。缺少 model 声明时，由 Claude Code 自行决定使用哪个模型，可能导致：①成本不可控（简单任务使用 Sonnet 级别模型）；②性能不稳定（复杂推理任务被分配到 Haiku）；③行为不可重现（不同版本的 Claude Code 默认模型可能不同）。

**MCP 工具名漂移（MCP Tool Name Drift）**
当 MCP 服务器版本更新或工具名称发生变化时，已在 agent/command 文件中声明的工具名（格式：`mcp__<server>__<tool>`）与实际可用工具名不匹配的现象。本仓库的 `WebSearch` → `mcp__firecrawl-mcp__firecrawl_search` 切换展示了这种漂移：正文文档（step 21 仍写「Use WebSearch」）未能与工具声明同步更新，形成「声明 vs 文档」的局部不一致。

**双极分布（Bimodal Quality Distribution）**
同一仓库内不同工件之间质量分数差异极大的现象。本仓库的极端案例：`crypto_research.md` 100/100 与 `CLAUDE.md` 65/100（空文件）在同一仓库共存。双极分布通常反映作者对不同工件类型的重视程度差异——在本仓库中，作者深度掌握「如何写高质量 agent prompt」，但未将同等注意力投入到「CLAUDE.md 和 command frontmatter 的规范性」。
