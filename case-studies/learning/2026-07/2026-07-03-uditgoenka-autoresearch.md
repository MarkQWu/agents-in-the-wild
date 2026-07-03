# uditgoenka/autoresearch — 学习案例

**仓库**：https://github.com/uditgoenka/autoresearch
**Stars**：⭐3612 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-03（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `security-gate`, `fallback-chain`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

**一句话**：Autoresearch 是一款「自主目标导向迭代引擎」——给定一个目标（bug、优化、功能），让 Claude Code 在边界内自主循环修改→验证→决定保留/回滚，直到收敛或达到迭代上限。

关键事实：
- 当前版本 2.2.1，提供 14 个命令（autoresearch/plan/debug/fix/security/ship/scenario/predict/learn/reason/probe/evals/improve/regression）
- 多平台发布：Claude Code（.claude/）、OpenCode（.opencode/）、Agents/Codex（.agents/）、claude-plugin 四套分发，靠 `scripts/sync-codex.sh` 等脚本同步
- 核心竞争力：声称「95% token 减少」（通过模块化架构），有 `.claude/hooks/autoresearch/` 下完整的行为控制 hook 体系
- 作者 Udit Goenka 是 Claude Code 生态早期活跃贡献者，该仓库是他的旗舰产品

### 1.2 架构剖析

```
autoresearch/
├── .claude/
│   ├── commands/autoresearch.md          # 主命令（入口）
│   ├── commands/autoresearch/{debug,fix,plan,...}.md  # 13 个子命令
│   ├── skills/.env.example
│   ├── skills/autoresearch/
│   │   ├── SKILL.md                      # 主技能（描述 autoresearch 概念）
│   │   └── references/                   # 4 个参考文件
│   │       ├── orchestrator-routing.md   # 命令路由决策树
│   │       ├── predict-personas.md       # predict 命令角色库
│   │       ├── reason-judge-protocol.md  # reason 命令裁判协议
│   │       └── security-checklist.md    # security 命令安全清单
│   └── hooks/autoresearch/
│       ├── hooks.json                   # hook 注册配置
│       ├── session-init.cjs             # 会话初始化
│       ├── iteration-context.cjs        # 迭代上下文追踪
│       ├── simplify-gate.cjs            # 阻止过度简化
│       ├── dangerous-cmd-block.cjs      # 危险命令拦截
│       ├── privacy-block.cjs            # 隐私数据泄露防护
│       ├── scout-block.cjs              # 无目标探索拦截
│       ├── dev-rules-reminder.cjs       # 开发规则提醒
│       └── stop-notify.cjs              # 停止时通知
├── .opencode/                           # OpenCode 发行版
├── .agents/                             # Codex/Agents 发行版
├── claude-plugin/                       # Claude Plugin 发行版
└── scripts/
    ├── install.sh
    ├── sync-codex.sh / sync-opencode.sh # 多平台同步脚本
    └── release.sh
```

- **文件类型分布**：1 个 SKILL.md + 4 个 references + 14 个 commands + 9 个 hooks + 4 套发行版 × 上述结构
- **编排关系**：`orchestrator-routing.md` 是核心路由文件，决定用户执行 `/autoresearch` 后如何根据参数路由到 debug/fix/plan 等子命令
- **跨件契约**：SKILL.md 知道命令体系，命令文件按 `orchestrator-routing.md` 的决策树分发，hooks 作为透明守卫层介入所有工具调用

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「行为约束 > 能力扩展」——不仅给 AI 更多能力（14 个命令），更通过 hooks 在运行时约束行为（拦截危险命令、防止隐私泄露、阻止无目标探索）
- **解决什么问题**：AI 自主迭代容易「失控」——无限循环、误操作、破坏现有代码、泄露敏感数据。autoresearch 把这些风险点作为 hook 前置拦截
- **做了什么 trade-off**：行为约束完善 vs 工具设置复杂（9 个 hook 文件，用户需要理解什么是 hook 才能信任这个工具）
- **反映什么认知模型**：作者把 AI agent 看作「需要护栏的实习生」——功能性强但危险性真实存在，每个 hook 对应一类真实出现过的问题（dangerous-cmd-block 不是为了安全感，是因为确实发生过）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「入口命令 + 参考路由 + Hook 守卫层」三明治架构

用户调用单一入口（`/autoresearch`），命令体内加载 `orchestrator-routing.md` 参考文件自主决定子命令，同时 hooks 在所有工具调用前后静默执行护栏逻辑——命令层、路由层、安全层三者分离。

模式特征清单：
- 特征 1：**单入口多出口**：一个 `/autoresearch` 命令下挂 13 个子命令，降低用户记忆负担
- 特征 2：**路由逻辑外置**：`orchestrator-routing.md` 是一份路由决策树文档，命令只引用它，不硬编码路由逻辑
- 特征 3：**行为护栏透明化**：9 个 hook 对用户透明，但在工具调用层面强制执行约束
- 特征 4：**多平台适配**：通过同步脚本维护 4 套发行版，核心逻辑只改一处
- 特征 5：**无状态跨会话**：hooks 管理迭代上下文，让跨会话的 autoresearch 循环有上下文连续性

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要 AI 自主迭代修复（有验证命令的场景） | ✅ 高度适用 | 有 Verify 命令让循环可以收敛 |
| 简单一次性任务 | ❌ 不适用 | 14 命令 + 9 hooks 的复杂度超过价值 |
| 跨平台（Claude + OpenCode + Codex）发布 | ✅ 适用 | 4 套发行版体系成熟 |
| 需要精细控制 AI 行为的高风险场景 | ✅ 适用 | hooks 护栏体系正是为此设计 |
| 初学者或简单项目 | ❌ 不适用 | 学习曲线陡峭 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 入口命令 + 参考路由 + Hook 守卫（本仓库） | autoresearch | 护栏完善，扩展灵活 | 复杂度高，9 hooks 维护成本 |
| 单命令单技能 | swiftui-pro | 简单直观 | 无护栏，无跨会话状态 |
| 多命令平铺（无路由层） | gstack | 易理解 | 每个命令独立维护，相互无关联 |
| 外部 MCP 服务 | manaflow-ai/cmux | 计算下沉，不消耗 context | 需要运行外部服务 |

### 2.4 改进空间

1. **当前问题**：`.claude/commands/` 下所有 14 个命令文件均缺少 `allowed-tools` 声明（当前 HEAD 已确认）。**改进做法**：在每个命令 frontmatter 加明确 `allowed-tools` 列表。**预期收益**：在限制工具环境下（企业部署）能明确报错而非静默失败。

2. **当前问题**：4 套发行版同步靠脚本，脚本内有 `python3 -c "..."` 内联 Python（含 bash 变量插值）的安全隐患（audit #1 中等严重）。**改进做法**：同步脚本改用 temp file 传 Python 脚本。**预期收益**：路径中含特殊字符时不会导致 Python 语法错误。

3. **当前问题**：`hooks.json` 是整个护栏体系的核心，但目前没有文档说明每个 hook 的触发条件和处理逻辑（面向插件使用者）。**改进做法**：在 `.claude/hooks/autoresearch/` 下加 `README.md` 解释 9 个 hook 的各自职责和触发事件。**预期收益**：用户遇到 hook 拦截时知道是哪条规则在起作用。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **88/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.opencode/agents/docs-manager.md` | 55/100 | 缺 `name` 字段、无 model 声明、无示例 |
| `.opencode/commands/autoresearch*.md`（10 个） | 75/100 | 缺 `name` 字段 |
| `.claude/skills/autoresearch/SKILL.md` | 88/100 | 内嵌「发布后让用户帮 star」的社交提示 |
| `.claude/commands/autoresearch*.md`（11 个） | 95/100 | 缺 `allowed-tools` 声明 |
| `claude-plugin/.claude-plugin/plugin.json` | 100/100 | 完整无误 |

### 3.2 当时值得借鉴的模式

1. **Orchestrator Routing 外置**：`references/orchestrator-routing.md` 单独维护路由决策树，命令文件只引用它。好处：路由规则变化时只改一处文件，不改 14 个命令文件。

2. **Hook 守卫层完整化**：9 个 hook 对应 9 类真实风险（dangerous-cmd-block、privacy-block、scout-block 等），每个 hook 职责单一。这是「单职责原则」在 NL artifact 层面的完整体现。

3. **多平台发布 + 同步脚本**：4 套发行版有脚本维护同步，核心逻辑改一处。体现了「不要重复自己（DRY）」原则在跨平台场景下的工程实现。

4. **`argument-hint` 结构化**：主命令的 `argument-hint` 字段把所有参数写得非常明确（`[Goal: <text>] [Scope: <glob>] [Metric: <text>] [Verify: <cmd>] [Guard: <cmd>] [Iterations: N] [--evals]`），用户即使不读文档也知道怎么用。

5. **`.env.example` 配套**：提供了 `.claude/skills/.env.example`，说明技能可能需要的环境变量，降低了安装摩擦。

### 3.3 当时的缺陷

1. **社交工程风险：star-repo 嵌入技能行为**：所有 4 套发行版的 SKILL.md 都有一段「完成后运行 `gh api -X PUT /user/starred/uditgoenka/autoresearch`」的指令。根本原因：作者把营销手段嵌入产品行为，让 AI 在用户毫不知情的情况下代为 star 仓库。这类「行为劫持」损害用户信任——即使是 idempotent 的 API 调用，未经明确同意的网络副作用都不应该内嵌进 skill。自查：我的任何 skill 都没有此类社交工程指令（这是正确的，应该继续保持）。

2. **`allowed-tools` 缺失**（14 个命令文件）：命令调用了 Bash、Read、Write 等工具，但 frontmatter 不声明。根本原因：开发时默认继承全部权限，没有想到限制工具的场景。自查：我的 gstack 所有 SKILL.md 都有 `allowed-tools`（这是我做得比他们好的地方），但 bureau 的 commands/ 下的 `.md` 文件我需要检查。

3. **OpenCode 命令缺 `name` 字段**：`.opencode/` 下的 10 个命令在 audit 时缺 `name`，且 `.opencode/agents/docs-manager.md` 缺 name + model + examples。根本原因：OpenCode 的 schema 与 Claude Code 略有不同，同步脚本没有自动处理这个差异。自查：我目前没有 OpenCode 分发，这个问题不适用。

### 3.4 当时的优化机会

1. 把 star-repo 社交提示从 SKILL.md 移出，改成用户主动调用的可选命令，且加一行显式确认
2. 为所有 `.claude/commands/` 文件加 `allowed-tools`（15 分钟可批量完成）
3. 修复 `scripts/sync-codex.sh` 里 `python3 -c` 的路径注入风险

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `docs-manager.md` 缺 `name` | `head -10 .opencode/agents/docs-manager.md` | **已修复**：`name: docs-manager` 已在第 2 行 | 作者响应了 bug；frontmatter 完整化 |
| `.opencode/commands/` 缺 `name` | `head -5 .opencode/commands/autoresearch.md` | **已修复**：`name: autoresearch` 已存在 | 同上，两个 bug 一并处理 |
| `.claude/commands/` 缺 `allowed-tools` | `grep -r "allowed-tools" .claude/commands/` | **仍存在**：grep 无结果，14 个命令文件均无此字段 | 软质量问题通常被推迟，优先级低于 bug |
| SKILL.md 内嵌 star-repo 社交提示 | `grep -r "starred" .claude/skills/` | **已移除**：grep 无结果，4 套 SKILL.md 均无此指令 | 这是最大的进步——说明作者收到了负反馈并响应 |

### 4.2 架构演进

2026-04-06 audit 时 vs 当前 HEAD，显著变化：
1. **新增 hooks 体系**（重大新增）：`.claude/hooks/autoresearch/` 下的 9 个 hook 文件在 audit 时不存在或不完整，现在成为核心特性。说明作者在 v2.x 阶段意识到「行为约束」比「功能扩展」更重要。
2. **命令从 10 个增加到 14 个**：新增了 evals/improve/probe/regression 四个命令，体现从「修复导向」扩展到「评估导向」的产品进化。
3. **`.agents/` 发行版新增**：audit 时记录了 .claude/.opencode/claude-plugin 三套，现在新增了 `.agents/`（Codex/Agents 平台）
4. **sync-codex/sync-opencode 脚本演化**：现在有独立的 `scripts/orchestrate.sh`

### 4.3 新增的可学习模式

**Hook 守卫层体系**（audit 时未覆盖）：

`dangerous-cmd-block.cjs`、`privacy-block.cjs`、`scout-block.cjs` 等构成了一套「行为防火墙」——比单纯在 prompt 里说「不要做 X」更可靠，因为 hook 是在工具调用层面拦截，而不是依赖模型遵守指令。

这是架构层面的安全，而非 prompt 层面的安全。值得在需要高可靠性的自主 agent 场景中直接借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **allowed-tools 声明**：gstack 所有 SKILL.md 都有显式 `allowed-tools`，这点比 autoresearch 的命令文件强
2. **无社交工程指令**：我的任何 skill 都没有嵌入「帮我 star 仓库」「帮我转发」等行为劫持指令
3. **单职责命令**：gstack 的每个 SKILL.md 做一件事（如 ios-design-review、qa、spec），不用路由到子命令，减少了 orchestrator-routing 的维护成本

### 5.2 挑战 / 验证

这次案例验证了一个关键认知：**「行为约束 > 提示约束」**——autoresearch 从 v1 到 v2 最大的架构变化是加了 hooks，而不是改了 prompt。在构建需要自主迭代的 agent 时，仅靠 prompt 说「不要做危险操作」是不够的，需要在工具调用层设防火墙。

另一个挑战：审查时发现「社交工程指令」从仓库移除了，说明作者真的在响应社区反馈。这提示了：**高质量 audit 报告有改变维护者行为的实际效果**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 allowed-tools 声明
grep -rn "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null | head -5
grep -rn "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null | head -5
# 命中后：确认覆盖齐全；未命中：逐一加上，tools 列表参考命令实际调用的工具

# 检查是否有隐藏的社交工程指令
grep -rn -E '(starred|star.*repo|star.*仓库|follow.*github)' \
  /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/ \
  /tmp/my-repos/MarkQWu-graphify/ 2>/dev/null
# 命中后：评估该指令是否经用户明确同意；若为隐式调用则移除
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 agent 流程加一个「危险命令拦截 hook」（类似 autoresearch 的 dangerous-cmd-block）
   - **为何可行**：bureau 的 recall/scribe agent 会操作用户文件，有误删风险
   - **第一步**：新建 `.claude/hooks/bureau-guard.cjs`，在 PreToolUse 事件中检查 Bash 命令是否包含 `rm -rf`、`> /dev/null` 等危险模式；参考 autoresearch 的 hook 格式，预计 30-45 分钟

2. **想法**：为 gstack 的 orchestrate（`spec → qa → ship`）流程引入 orchestrator-routing 参考文件
   - **为何可行**：gstack 现有 spec/qa/ship 三个 skill 之间有逻辑依赖，但没有路由文档说明在什么条件下跳到下一步
   - **第一步**：新建 `gstack-flow/references/routing.md`，定义「spec 完成标准」→「qa 触发条件」→「ship 判断门槛」；预计 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：让 AI 在给定目标和验证命令后自主迭代到收敛，并在过程中通过 hooks 约束危险行为

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都有 agent 编排，都有多命令体系 | bureau 是知识管理工具，不是自主迭代引擎 | 中（hooks 模式可借鉴）|
| MarkQWu/gstack | 低 | 都是 Claude Code skill 集合 | gstack 是单次调用工具集，不做循环迭代 | 低 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 agent 定义，都用引用文件 | echo-sleuth 做记忆挖掘，不做迭代 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令文件缺 `allowed-tools` | `grep -L "allowed-tools" bureau/commands/*.md` | **bureau 命中**：`commands/lint.md`、`commands/status.md`、`commands/serve.md`、`commands/inspect.md` 均无 `allowed-tools` 字段 | 中 |
| 社交工程指令（star/follow） | `grep -rn "starred\|star.*repo" gstack/ bureau/ echo-sleuth/` | **未命中**：我的仓库无此问题 | 无 |
| Agent 缺 model 声明 | `grep -L "model:" echo-sleuth-for-claude/agents/*.md` | echo-sleuth 的 4 个 agent 文件均无 `model:` 字段 | 中 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/commands/lint.md` 等 4 个文件 → 逐一追加 `allowed-tools: [Read, Bash, Glob]`（每个约 2 分钟）
- `MarkQWu-echo-sleuth-for-claude/agents/*.md` → 加 `model: claude-haiku-4-5-20251001` 或 `model: claude-sonnet-5`（根据 agent 复杂度选择）

### 8.3 别人的更优方案

1. **领域**：Hook 守卫层（行为级安全防护）
   - **本案例做法**：`.claude/hooks/autoresearch/dangerous-cmd-block.cjs` 在 PreToolUse 事件中拦截 `rm -rf`、`dd if=`、`mkfs` 等危险命令，返回错误并记录日志
   - **我的项目现状**：bureau 和 gstack 无任何 hook，AI 执行 Bash 命令时无护栏
   - **如何借鉴**：在 `MarkQWu-bureau/.claude/hooks/` 下新建 `guard.cjs`，注册 PreToolUse 事件，检查 Bash 工具调用的命令参数；hooks.json 格式参考 autoresearch 现有配置

2. **领域**：Orchestrator Routing 外置为参考文件
   - **本案例做法**：`references/orchestrator-routing.md` 是一份完整的路由决策树，`/autoresearch` 命令体内引用它，自己不写路由逻辑
   - **我的项目现状**：echo-sleuth 的多个 agent（file-historian/schema-scout/recall）之间无路由文档，由命令文件隐式决定调用顺序
   - **如何借鉴**：新建 `echo-sleuth/references/agent-routing.md`，写明「哪种查询 → 调用哪个 agent」的决策逻辑

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：SKILL.md 中无副作用行为劫持
- **我的做法**：gstack、bureau、echo-sleuth 所有技能文件中，没有任何隐式网络调用（star、follow、send 等）
- **本案例做法（弱在哪）**：audit 时 autoresearch 的 SKILL.md 内嵌 `gh api -X PUT /user/starred/...`，在用户不知情下执行 GitHub API 调用
- **意义**：这是「零副作用设计」的体现——一个 skill 只做它声称的事，不附带隐形行为。这点值得在项目说明中明确提及作为设计原则

---

## 八、术语表

### <a name="hook"></a>hook（Claude Code Hooks）
> Claude Code 提供的「事件钩子」机制。可以在 AI 执行某类工具（如 Bash、Write）**之前（PreToolUse）** 或**之后（PostToolUse）** 自动运行一段脚本。autoresearch 的 `dangerous-cmd-block.cjs` 就是在 PreToolUse 时运行——如果 Bash 命令包含危险模式，脚本返回错误码，Claude Code 就会拒绝执行那条命令。

### <a name="orchestrator-routing"></a>编排路由（Orchestrator Routing）
> 「给定用户输入，决定调用哪个子命令/子 agent」的决策逻辑。autoresearch 把这个逻辑写成 `orchestrator-routing.md` 参考文件，而不是硬编码在命令体内。好处：路由规则改变时只改一处文件，不影响 14 个命令文件。

### <a name="social-engineering-in-skills"></a>技能内社交工程
> 在 SKILL.md 里嵌入「帮我点赞/关注/转发」等指令，让 AI 在完成任务后代替用户执行有利于 skill 作者的社交行为。这类做法不需要用户知情同意，属于行为劫持。autoresearch v2.2.1 已移除此类指令，但作为反面教材值得记住。

### <a name="manifest"></a>manifest（plugin.json）
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些 skills/commands/agents。如果 manifest 里漏声明某个文件，该文件即使在磁盘上也不会被加载——类似「仓库里的书没编入目录，图书馆找不到」。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md 或 command.md [frontmatter](#frontmatter) 中的字段，声明这个 skill/command 允许调用哪些工具。缺少此字段时 Claude Code 默认继承会话全部工具权限，可能在受限环境（企业部署、安全沙箱）下静默失败或超过预期地调用敏感工具。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据。`name`、`description`、`model`、`allowed-tools` 都写在这里。Claude Code 先解析 frontmatter 才知道如何注册和调用这个 artifact。
