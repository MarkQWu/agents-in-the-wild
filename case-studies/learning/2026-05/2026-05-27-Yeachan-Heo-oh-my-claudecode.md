# Yeachan-Heo/oh-my-claudecode — 学习案例

**仓库**：https://github.com/Yeachan-Heo/oh-my-claudecode
**Stars**：29671 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-27（基于当前 HEAD）
**主题标签**：`router-channels`, `security-gate`, `vague-quantifier`, `experience-accumulation`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

oh-my-claudecode（简称 OMC）是迄今为止 Claude Code 插件生态中规模最大的多 agent 编排层之一，29671 stars。版本 **4.14.4**（`plugin.json`）。它的核心定位是「一键接入专业级 AI 开发工作流」——安装后即拥有 19 个专家 agent、39 个 skill、以及一套完整的需求→规划→执行→复盘 [PDCA](#pdca) 循环。

关键事实：
1. **19 个 agent**（`agents/`），覆盖架构师、批判者、科学家、测试工程师等专业角色
2. **39 个 skill**（`skills/`），从单次任务（`ask`）到完整工作流（`ultrawork`、`ralplan`）不等
3. **11 种 [钩子(hook)](#hook)**（`hooks/hooks.json`），全部通过 Node.js `scripts/run.cjs` 分发
4. **多语言 README**：英、中、日、韩、法、德、西、意、葡、俄、土、越南语，面向全球用户
5. **CLAUDE.md** 承担双重职责：操作原则 + 关键词触发路由（keyword-trigger routing）

### 1.2 架构剖析

目录结构：

```
oh-my-claudecode/
├── agents/          # 19 个 agent .md（专家角色）
├── skills/          # 39 个 skill 目录，每个含 SKILL.md
├── hooks/           # hooks.json（11 种事件）
├── scripts/         # run.cjs（Hook 统一分发器）
├── src/             # 模板文件（exploration-template.md 等）
├── .github/         # GitHub 专用 CLAUDE.md（版本元数据）
├── AGENTS.md        # agent 调用约定
├── SECURITY.md      # 安全政策
└── plugin.json      # 插件清单，版本 4.14.4
```

文件类型分布：19 个 agent .md / 39 个 SKILL.md / 11 个 [frontmatter](#frontmatter) hook 条目

编排关系：
- **CLAUDE.md 关键词路由** → 匹配用户输入关键词 → 调用对应 SKILL.md
- **RALPLAN 框架**：`/ralph`（需求收集 PRD 循环）→ `/ralplan`（规划，支持 `--consensus`/`--direct` 等标志）→ `plan`（共识模式多 agent 规划）→ `autopilot`（自主执行循环）
- **pipeline frontmatter**：`deep-dive/SKILL.md` 声明 `pipeline: [deep-dive, plan, autopilot]` 和 `next-skill: plan`，形成自文档化 skill 链
- **Hook 分发**：所有 11 种 hook 事件均路由到 `scripts/run.cjs`，由脚本根据事件类型分发

### 1.3 设计思路 / 方法论

- **核心设计哲学**：以 skill 为原子能力单元，以 RALPLAN 框架为宏观工作流，将「需求澄清」和「执行自动化」分离为两个明确阶段
- **解决什么问题**：单纯调用 Claude Code 时，用户容易跳过需求澄清直接执行，导致返工。OMC 强制 RAL（Requirements And Learning）阶段，要求在规划前先收集充分的业务背景
- **Trade-off**：功能极其丰富，但 39 个 skill 的学习曲线陡峭；`dangerously-skip-permissions`（见[安全隐患](#dsp)）换取了自动化流畅度，但代价是引入可被滥用的执行权限
- **认知模型**：CLAUDE.md 是「大脑」（路由决策），agent 是「专家」（领域判断），skill 是「手册」（操作步骤），hook 是「自动反应」（无需用户干预的后台动作）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：CLAUDE.md 关键词路由 + [frontmatter](#frontmatter) pipeline 链**

这个仓库把「何时触发哪个 skill」完全写进 CLAUDE.md 的关键词映射表，而不是依赖用户记忆命令名称。同时，skill 通过 [frontmatter](#frontmatter) 的 `pipeline`/`next-skill`/`handoff` 字段声明自己在工作流链中的位置——这使得 skill 链是自文档化且机器可读的。

模式特征清单（4 条）：
- 特征 1：CLAUDE.md 包含明确的「关键词 → skill 路径」映射表，用户输入自然语言即可触发正确 skill
- 特征 2：skill [frontmatter](#frontmatter) 声明 `pipeline`、`next-skill`、`handoff` 字段，定义 skill 链的拓扑结构
- 特征 3：agent 文件声明 `model`（如 `opus`）、`level`、`disallowedTools`，将资源决策从运行时提前到定义时
- 特征 4：RALPLAN 框架是一个明确的宏观 [PDCA](#pdca) 循环，每个阶段（ralph/ralplan/plan/autopilot）都有对应的 skill

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多阶段开发工作流（需求→设计→实现→测试）| ✅ 高度适用 | RALPLAN 框架天然对应这个生命周期 |
| 需要不同专业视角的复杂决策（架构评审）| ✅ 适用 | 19 个专家 agent 可组合调用 |
| 单次简单查询（问一个 Python 语法问题）| ❌ 过度工程 | 39 个 skill 的认知负担远超单次任务的收益 |
| 安全要求严格的生产环境 | ⚠️ 谨慎 | `--dangerously-skip-permissions` 需人工审计后才适合生产 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| CLAUDE.md 关键词路由 + agent 矩阵（本仓库）| oh-my-claudecode | 零命令记忆成本，自然语言触发；agent 矩阵覆盖所有主流专业角色 | 规模膨胀（39 skills），安全隐患难消灭 |
| 命令驱动 + agent 链（中等规模）| echo-sleuth-for-claude | 架构清晰、作用域窄，易于理解和调试 | 功能覆盖有限，不适合通用工作流 |
| MCP 后端 + NL 包装层 | Q00/ouroboros | 状态持久化强，业务逻辑在 Python 层可测试 | 安装复杂，调试需跨两层 |

### 2.4 改进空间

1. **当前问题**：34/39 个 skill 缺少 [frontmatter](#frontmatter) 的 `triggers` 字段（`skills/*/SKILL.md` 中 `grep -l "^triggers:" skills/*/SKILL.md` 仅返回 5 个文件）。**改进做法**：为每个 skill 的 frontmatter 加 `triggers:` 数组，列出 2-3 个典型触发短语。**预期收益**：CLAUDE.md 关键词路由表可以从 skill 定义中自动生成，而不是手动维护。

2. **当前问题**：`src/agents/templates/exploration-template.md` 和 `implementation-template.md` 没有 YAML [frontmatter](#frontmatter)（文件直接以 `#` 标题开头），无法被 Claude Code 自动发现。**改进做法**：在两个模板文件顶部加入标准 frontmatter（`name`、`description`、`model`），并将 `model` 指向 sonnet（模板不需要 opus 的推理深度）。**预期收益**：模板文件成为可注册组件，且 `nlpm:score` 分数从当前的 45/100 提升到 80+。

3. **当前问题**：`.github/CLAUDE.md` 版本号仍为 `4.8.2`，而 plugin.json 已是 `4.14.4`——版本差距达 6 个小版本。**改进做法**：在 CI 发布流程中加一步同步检查（`grep -r "VERSION" .github/ | grep -v "4.14.4"` 则阻断发布）。**预期收益**：彻底消除「哪个版本是权威来源」的歧义。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 得分 **80/100**（Security: **REVIEW**）。共发现 3 个 bug、10 个质量问题、2 个 High 级安全问题、4 个 Medium 级安全问题。

| 类别 | 数量 | 主要文件 |
|---|---|---|
| Bug（功能性错误）| 3 | exploration-template.md、implementation-template.md、.github/CLAUDE.md |
| Security High | 2 | skills/project-session-manager/SKILL.md |
| Security Medium | 4 | skills/configure-notifications/SKILL.md、hooks/hooks.json（全部 11 条）|
| 质量问题（模糊量词、缺字段）| 10 | 33 个 skill 缺 triggers，code-simplifier 缺 Examples |

### 3.2 当时值得借鉴的模式

1. **agent 声明 `model: opus`**：`agents/architect.md` 和 `agents/planner.md` 显式声明 `model: opus`，把「用哪个模型」的决策从运行时提前到定义时，防止默认模型升级后的行为漂移 → `agents/architect.md` frontmatter L1-10 → 借鉴：凡是高推理深度任务（架构决策、多步规划）的 agent 都应声明 `model: opus`；轻量任务用 sonnet 或 haiku 并明确声明

2. **`disallowedTools` 作为安全边界**：`agents/architect.md` 声明 `disallowedTools: Write, Edit`——架构师只负责分析，不写代码 → 借鉴：每个 agent 的职责边界应通过 `disallowedTools` 硬约束，而非仅靠 prompt 软约束

3. **skill pipeline frontmatter**：`skills/deep-dive/SKILL.md` 的 `pipeline: [deep-dive, plan, autopilot]` 和 `handoff: .omc/specs/deep-dive-{slug}.md` 让 skill 链成为机器可读的图 → 借鉴：当 skill 之间有明确顺序关系时，用 frontmatter 声明比在 body 中用文字描述更可靠

4. **RALPLAN 宏观 [PDCA](#pdca) 框架**：ralph → ralplan → plan → autopilot 形成一个完整的需求到执行的闭环，每个阶段都有对应的 skill 作为「检查点」 → 借鉴：多步骤工作流不应设计为单个超大 skill，应拆分为明确的阶段，每阶段有独立 skill

5. **`critic.md` 的 5 阶段调查协议**：调查→自我审计→现实核查→综合→输出，每阶段有明确的问题清单 → 借鉴：批判性 agent 不应只是「提意见」，应有结构化的调查流程防止批评流于表面

### 3.3 当时的缺陷

1. **Bug 1**：`src/agents/templates/exploration-template.md` 无 YAML [frontmatter](#frontmatter)（得分 45/100）→ 根本原因：模板文件被视为「人类参考文档」而非 NL 组件，开发者复制了 Markdown 文件而没有按 Claude Code 的 agent schema 格式化

2. **Bug 2**：`src/agents/templates/implementation-template.md` 同上（得分 45/100）→ 同一根本原因

3. **Bug 3**：`.github/CLAUDE.md` 版本元数据为 `<!-- OMC:VERSION:4.8.2 -->`，而 audit 时线上版本是 4.12.1（差 4 个小版本）→ 根本原因：`.github/CLAUDE.md` 的版本号是手工维护的，发布流程没有包含这个文件的版本同步步骤

4. **Security High 1**：`skills/project-session-manager/SKILL.md` 第 255、307、337 行出现 3 次 `--dangerously-skip-permissions` 标志 → [dangerously-skip-permissions](#dsp) 会让 Claude Code 跳过所有工具调用权限确认，在自动化脚本中存在无限制执行任意命令的风险

5. **Security High 2**：同一文件中，`tmux send-keys` 命令使用了来自 GitHub API 的 PR/issue 标题作为参数，而这些标题未经任何清洗（unsanitized）→ 攻击者可以创建一个标题包含 shell 元字符的 PR，当 OMC 的 session manager 处理它时，恶意命令会被注入到 tmux 会话中执行

6. **Security Medium**：`skills/configure-notifications/SKILL.md` 指导用户将 token 明文存储到配置文件中，且使用 `curl` 带 Bearer token 发请求 → token 可能被意外提交到版本库或被进程列表泄露

### 3.4 当时的优化机会

1. 为 `src/agents/templates/*.md` 批量添加标准 YAML [frontmatter](#frontmatter)，优先级最高，直接修复两个 45 分的 bug 文件
2. 修复 `.github/CLAUDE.md` 版本同步问题，在 `package.json`/`Makefile` 的 release 步骤中加一行 sed 替换
3. 将 `project-session-manager/SKILL.md` 中的 `--dangerously-skip-permissions` 替换为按需的精细化工具权限声明（`allowedTools`），并对 `tmux send-keys` 的参数来源加引号转义（`$(printf '%q' "$TITLE")`）
4. 为所有 33 个缺少 `triggers` 字段的 skill 批量添加 [frontmatter](#frontmatter) triggers，可用脚本从 skill 描述中提取候选词

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| exploration-template.md 无 frontmatter | `head -3 src/agents/templates/exploration-template.md` | **仍未修复**（文件仍以 `# Exploration Task Template` 开头，0 行 frontmatter）| 维护者未将模板文件纳入 NL 组件管理 |
| `--dangerously-skip-permissions` 存在 | `grep -c "dangerously-skip-permissions" skills/project-session-manager/SKILL.md` | **仍存在（4 处，audit 时 3 处，新增 1 处！）** | 两个月内不减反增，说明这是有意设计选择 |
| .github/CLAUDE.md 版本落后 | `grep "OMC:VERSION" .github/CLAUDE.md` | **`4.8.2`，repo 已是 `4.14.4`，差距从 4 个小版本扩大到 6 个小版本** | 发布流程始终未包含此文件的同步 |
| code-simplifier.md 缺 Examples | `grep -c "## Examples" agents/code-simplifier.md` | **仍缺失（0 处）** | 批判性 agent 四个月内未添加示例 |
| skill triggers 不足 | `grep -rl "^triggers:" skills/*/SKILL.md \| wc -l` | **略有改善（5/39）**，audit 时约 2/37；新增 triggers 的 skill：deep-dive、deep-interview、configure-notifications、skill、learner | 正在缓慢改进，但 34/39 仍缺 |
| architect.md 缺 model 声明 | `grep "^model:" agents/architect.md` | **已修复**：现在声明 `model: opus` | ✅ 审计后修复，且是最高价值改进之一 |

### 4.2 架构演进

与 2026-04-19 audit 时相比，现在仓库的主要变化：

- **`architect.md` 声明 `model: opus`**：这是 audit 期间明确指出的缺失项，已落地，说明审计反馈被采纳
- **triggers 微幅改善**：从约 2/37 增长到 5/39，方向正确但速度慢
- **版本 4.12.1 → 4.14.4**：两个月内发布了 2 个小版本，活跃维护
- **多语言 README 扩展**：新增多个语言版本，面向更广泛的全球用户
- **`SECURITY.md` 新增**：说明维护者已意识到安全问题的重要性，但实际代码中 `--dangerously-skip-permissions` 出现次数反而增加，文档与实践存在矛盾

### 4.3 新增的可学习模式

- **`architect.md` 综合声明模式**：`model: opus` + `level: 3` + `disallowedTools: Write, Edit` + XML 输出格式要求 + 「每个结论必须引用 file:line」的强制约束 —— 这是 OMC 里迄今最完整的 agent 定义规范，可以直接作为其他高推理 agent 的模板
- **`deep-interview/SKILL.md` 的数学歧义评分**：skill 内含一套形式化的歧义评分规则，而不仅靠 prompt 描述「检查模糊性」—— 这把定性指导转化为定量判断，显著减少 Claude 的主观发挥空间

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth 的 agents 有完整 frontmatter**：5/5 个 agent 都有 `name`、`description`，这是 OMC 模板文件缺失的部分
2. **无 `--dangerously-skip-permissions` 使用**：echo-sleuth 没有任何此类标志，安全基线优于 OMC
3. **版本单一来源**：echo-sleuth 的版本信息只在 plugin.json 中维护，没有 .github/ 下的副本，不存在版本漂移问题
4. **CLAUDE.md 有清晰的命令→agent→skill 流程图**：结构文档化程度与 OMC CLAUDE.md 相当

### 5.2 挑战 / 验证

- **挑战了什么假设**：我之前认为「skill triggers 是可选的装饰性字段」。OMC 的案例表明，当 CLAUDE.md 关键词路由表需要手工维护时，triggers 字段就变成了「路由表的源数据」——如果 triggers 缺失，路由表就需要人工与 skill body 保持同步。这个认知升级改变了我对 triggers 优先级的判断。
- **验证了什么做法**：`disallowedTools` 作为硬约束（而非 prompt 软约束）的设计思路——`architect.md` 无法调用 Write/Edit 工具，这是架构师角色的边界，不依赖 Claude 的理解能力，而是通过工具列表强制执行。这个模式在 echo-sleuth 中还没有应用，值得引入。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 agents 是否声明了 model 字段（应为 opus/sonnet/haiku）
grep -rL "^model:" agents/*.md
# 命中后：对推理密集型 agent（架构/规划/审查）加 model: opus，轻量 agent 加 model: sonnet

# 检查 skills 中 triggers 字段的覆盖情况
grep -rl "^triggers:" skills/*/SKILL.md | wc -l
# 目标：至少 50% 的 skill 有 triggers；命中（< 50%）后按使用频率补充最高频 skill 的 triggers

# 检查 agent 是否声明了 disallowedTools（硬边界）
grep -rL "disallowedTools" agents/*.md
# 命中后：为每个 agent 分析其职责，添加明确禁止的工具列表（如只读 agent 禁 Write/Edit）

# 检查是否存在 dangerously-skip-permissions
grep -r "dangerously-skip-permissions" skills/ agents/ hooks/ .claude/ 2>/dev/null
# 命中后：立即审查该文件，评估是否可以用 allowedTools 精细授权替代

# 检查多处维护的版本号是否一致
VERSION=$(grep '"version"' plugin.json | grep -oP '[\d.]+')
grep -r "$VERSION" .github/ CLAUDE.md README.md 2>/dev/null | grep -v "^Binary"
# 命中空：说明其他文件版本号落后，需手动或脚本同步
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 agents 加 `disallowedTools` 声明
   - **为何可行**：echo-sleuth 的 5 个 agent 职责明确（memory-auditor 只读，file-historian 只读，analyze 只读），天然适合用 `disallowedTools: Write, Edit` 约束
   - **第一步**：读取每个 agent 文件，确认其职责，在 frontmatter 中加 `disallowedTools:` 行，约 20 分钟，5 个文件

2. **想法**：参考 `architect.md` 的「结论必须引用 file:line」模式，强化 echo-sleuth 的分析类 agent
   - **为何可行**：echo-sleuth 的 `analyze` agent 和 `recall` agent 都需要给出有依据的结论，引用具体行号可以防止幻觉
   - **第一步**：在 `agents/analyze.md` 和 `agents/recall.md` 的输出格式要求中加一行「所有结论必须引用来源文件路径和行号」，约 10 分钟

3. **想法**：为 echo-sleuth 最常用的 3 个 skill 添加 `triggers` 字段
   - **为何可行**：当前 0/4 个 skill 有 triggers，添加 triggers 使得 CLAUDE.md 路由可以从 skill 定义中推导
   - **第一步**：为 `skills/memory-management/SKILL.md`、`skills/git-mining/SKILL.md`、`skills/experience-synthesis/SKILL.md` 各加 3-5 个触发短语，约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：MarkQWu/echo-sleuth-for-claude（最相似）、MarkQWu/drama-workshop-skills、MarkQWu/claude-for-legal

### 8.1 目的对齐度

- **本案例 oh-my-claudecode 的核心目的**：通用多 agent 开发工作流编排层，覆盖需求到执行的完整 [PDCA](#pdca) 循环

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | **高** | 都是 Claude Code 插件；都有 agents/+skills/+commands/ 三层结构；都有 hooks；都面向开发者工作流 | Echo Sleuth 是「挖掘过去记忆/git 历史」，OMC 是「编排未来执行」；Echo Sleuth 无 model 声明，无 disallowedTools | **高** |
| MarkQWu/claude-for-legal | 中 | 都有多 skill 集合；CLAUDE.md 路由模式相似；claude-for-legal 有约 10 个文件声明了 model | 法律领域 vs 开发工具；claude-for-legal 的 model 声明实践甚至优于 OMC | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件形式 | 短剧创作 vs 开发工作流；无多 agent 架构 | 低 |

### 8.2 在我的项目里复现的同类问题

**问题 1：echo-sleuth agents 没有 `model` 声明**

```bash
# 检查 echo-sleuth agents 是否有 model 声明
grep -l "^model:" ~/projects/MarkQWu-echo-sleuth-for-claude/agents/*.md
# 结果：0 hits（5/5 个 agent 均无 model 字段）
```

oh-my-claudecode 在 audit 后修复了 architect.md 的 model 声明（现在是 `model: opus`）。echo-sleuth 的 5 个 agent 全部没有 `model` 声明——Claude Code 会使用默认模型，而默认模型可能随版本升级改变，导致行为不确定。

**问题 2：echo-sleuth skills 没有 `triggers` 字段**

```bash
# 检查 echo-sleuth skills 的 triggers 覆盖情况
grep -rl "^triggers:" ~/projects/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
# 结果：0 hits（4/4 个 skill 均无 triggers 字段）
```

oh-my-claudecode 虽然 34/39 仍缺 triggers，但已有 5 个 skill 添加了此字段，且有改善趋势。echo-sleuth 是 0/4，比 OMC 当前状态还差。

**问题 3：echo-sleuth agents 没有 `disallowedTools` 约束**

```bash
# 检查 echo-sleuth agents 的 disallowedTools 声明
grep -rl "disallowedTools" ~/projects/MarkQWu-echo-sleuth-for-claude/agents/*.md
# 结果：0 hits（5/5 个 agent 均无工具约束声明）
```

OMC 的 `architect.md` 通过 `disallowedTools: Write, Edit` 硬约束了角色边界。echo-sleuth 的只读 agent（memory-auditor、file-historian）没有任何工具约束，理论上 Claude 可以调用 Write 工具修改文件——这是不符合 agent 职责设计的。

### 8.3 别人的更优方案

1. **领域**：Agent 的模型精细化声明（model pinning）
   - **本案例做法**：`agents/architect.md` 声明 `model: opus`，`agents/planner.md` 同样声明 `model: opus`，轻量 agent 用默认 sonnet——在 agent 文件中直接声明，无需运行时配置
   - **我的项目现状**：echo-sleuth 的 5 个 agent 全部无 `model` 声明（0 hits）；claude-for-legal 有约 10 个文件有 model 声明（在该仓库中是最佳实践）
   - **如何借鉴**：为 echo-sleuth 的 `analyze.md` 和 `recall.md`（推理密集）加 `model: sonnet`，为 `file-historian.md` 和 `schema-scout.md`（机械扫描）加 `model: haiku`，约 10 分钟

2. **领域**：`disallowedTools` 作为 agent 职责硬边界
   - **本案例做法**：`architect.md` 和 `critic.md` 都声明 `disallowedTools: Write, Edit`——角色是「评估者」，不是「执行者」，工具列表强制这个边界
   - **我的项目现状**：echo-sleuth 所有 agent 均无 `disallowedTools` 声明（0 hits）
   - **如何借鉴**：memory-auditor、file-historian、schema-scout 都是只读分析角色，加 `disallowedTools: Write, Edit, MultiEdit` 可防止意外修改文件

3. **领域**：Skill pipeline frontmatter（技能链的自文档化）
   - **本案例做法**：`skills/deep-dive/SKILL.md` frontmatter 声明 `pipeline: [deep-dive, plan, autopilot]`、`next-skill: plan`、`handoff: .omc/specs/deep-dive-{slug}.md`——技能链的拓扑结构是机器可读的
   - **我的项目现状**：echo-sleuth 有命令→agent→skill 的调用链，但这个关系只在 CLAUDE.md 的文字说明里，没有 frontmatter 声明
   - **如何借鉴**：为 `skills/experience-synthesis/SKILL.md` 加 `next-skill: git-mining`（因为综合依赖 git 挖掘结果），建立可追溯的依赖图

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：append-only JSONL 日志架构（状态管理）
   - **我的做法**：echo-sleuth-for-claude 使用 append-only JSONL 文件（`~/.echo-sleuth/events.jsonl`）记录所有操作，支持时间线重建、幂等 replay、git diff 友好
   - **本案例做法（弱在哪）**：OMC 使用 `.omc/` 目录下的普通文件存储状态（`handoff` 字段指向 `.omc/specs/...`），没有 append-only 保证，文件可被任意覆盖，无法追溯历史
   - **意义**：对于需要审计和历史重建的工作流（如「这个 bug 是什么时候引入的」），echo-sleuth 的日志架构更可靠

2. **领域**：commands/ 目录的 `allowed-tools` 覆盖率
   - **我的做法**：echo-sleuth 的核心命令（`audit.md`、`dashboard.md`、`extract.md`、`prune.md`）都有 `allowed-tools` 声明
   - **本案例做法（弱在哪）**：OMC 没有独立的 commands/ 目录，skill 充当命令入口，但 34/39 个 skill 缺少 `triggers` 这个入口声明，对等于 commands 缺 allowed-tools
   - **意义**：echo-sleuth 的命令边界更清晰，用户可以预期每个命令会读写哪些工具

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter（前置元数据）
> Markdown 文件顶部由 `---` 包裹的 YAML 块。Claude Code 读取这个块来发现 agent、skill 的名称、描述、触发条件、模型选择等元数据。类比：就像 HTML 的 `<head>` 标签，frontmatter 是文件的「自我介绍」，没有它 Claude Code 就无法自动注册这个组件。例：
> ```yaml
> ---
> name: architect
> description: 系统架构分析专家
> model: opus
> disallowedTools: Write, Edit
> ---
> ```

### <a name="triggers"></a>triggers 字段
> skill frontmatter 中的一个数组字段，声明哪些用户输入短语应自动触发此 skill。类比：就像手机 App 的快捷指令，`triggers: ["深度分析", "deep dive", "详细调查"]` 让用户输入任意一个短语时 Claude Code 知道调用 `deep-dive` skill。当 CLAUDE.md 关键词路由表需要手工维护时，triggers 字段就是路由表的数据源。

### <a name="dsp"></a>dangerously-skip-permissions（危险权限绕过标志）
> Claude Code 的一个命令行标志 `--dangerously-skip-permissions`，启用后 Claude 执行任何工具调用（写文件、运行脚本、网络请求）时都不会弹出权限确认对话框。这在自动化脚本中可以提高流畅度，但代价是：(1) 任何包含恶意内容的外部输入（如 PR 标题）都可以直接触发未经审查的命令执行；(2) 用户失去了「最后一道人工审核关卡」。类比：相当于以 root 用户运行一个来历不明的脚本，且关闭了所有 `sudo` 密码提示。

### <a name="pdca"></a>PDCA（计划-执行-检查-行动循环）
> 质量管理中的四阶段改进循环：Plan（计划）→ Do（执行）→ Check（检查）→ Act（行动/调整）。在 OMC 的 RALPLAN 框架中对应：ralph（需求收集 = Plan 前置）→ ralplan/plan（Plan）→ autopilot（Do）→ critic/verifier（Check）→ self-improve（Act）。PDCA 是一个**闭环**，不是线性流程——检查结果应反馈到下一轮计划中。

### <a name="hook"></a>钩子（Hook）
> Claude Code 在特定生命周期事件发生时自动调用的脚本或命令。OMC 使用了 11 种 hook 类型（SessionStart、UserPromptSubmit、PostToolUse 等），全部路由到 `scripts/run.cjs`。类比：就像 Git 的 pre-commit hook——你不需要手动调用它，它在特定时机自动触发。Hook 的安全风险在于：如果 hook 脚本使用了来自外部的未清洗输入（如 PR 标题），攻击者可以构造恶意输入触发 hook 时注入命令。
