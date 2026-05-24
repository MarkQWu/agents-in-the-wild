# KhazP/vibe-coding-prompt-template — 学习案例

**仓库**：https://github.com/KhazP/vibe-coding-prompt-template
**Stars**：2278 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-24（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `examples-driven`, `fallback-chain`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
KhazP/vibe-coding-prompt-template（2278 stars）是一套**"vibe coding" 五步工作流插件**，帮助开发者用 Claude Code 完成从需求调研到代码实现的完整产品开发周期。"vibe coding" 是社区流行词，指放松地、创意驱动地用 AI 编写代码。仓库用 6 个 SKILL.md 实现了：`vibe-research → vibe-prd → vibe-techdesign → vibe-agents → vibe-build → vibe-workflow`（最后一个作为 orchestrator），外加 1 个 `hooks.json`（带通知功能）。2278 stars 说明这套工作流模板已获得广泛认可，但安全上存在严重问题。

### 1.2 架构剖析
- **目录结构**：
  ```
  vibe-coding-prompt-template/
  └── .claude/
      ├── skills/
      │   ├── vibe-workflow/SKILL.md   # orchestrator，包含 5-step 总览
      │   ├── vibe-research/SKILL.md   # 调研阶段
      │   ├── vibe-prd/SKILL.md        # PRD 撰写阶段
      │   ├── vibe-techdesign/SKILL.md # 技术设计阶段
      │   ├── vibe-agents/SKILL.md     # 子 agent 配置阶段
      │   └── vibe-build/SKILL.md      # 构建验证阶段
      └── hooks/hooks.json              # PostToolUse + SessionStart 通知 hook
  ```
- **文件类型分布**：6 个 SKILL.md（1 个 orchestrator + 5 个专职）/ 1 个 hook 配置 / 无 command / 无 agent / 无 manifest
- **编排关系**：`vibe-workflow` 作为总览 skill，包含一个 Quick Commands 表格指向所有 5 个子 skill。子 skill 之间有明确的串行依赖：research → prd → techdesign → agents → build，每个完成后用 `/vibe-*` 触发下一步。这是一种**纯 skill 链**（没有 command 层、没有 agent 层），用户直接调用 skill。
- **跨件契约**：`vibe-workflow` 的 Quick Commands 表格正确引用了全部 5 个子 skill；`vibe-build` 声明了 `vibe-agents` 为前置依赖。所有跨件引用无断链。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「脚手架即工作流」——把产品开发流程的最佳实践写进 skill 文件，让 Claude Code 成为既懂需求分析又懂代码实现的"全栈助手"。
- **解决什么问题**：独立开发者或小团队在 vibe coding 时容易跳步骤（直接写代码，不写 PRD），导致需求不清楚、技术债积累。这套模板强制走完所有阶段。
- **做了什么 trade-off**：选择 skill 而非 command + agent，优点是轻量（无需 plugin manifest、无需 agent 注册），缺点是 skill 本身不能声明 `allowed-tools`（实际上 skill 是可以的，但作者没有声明，导致 bug）。
- **反映什么认知模型**：作者把 Claude Code 当成"遵循方法论的顾问"，不是自动代码生成机器——skill 里有大量"停下来确认"的指令，体现了对 AI 自主性的审慎边界。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[串行 Skill 链](#串行skill链)」（Linear Skill Pipeline 架构）**

无命令层、无 agent 层，用户直接触发 skill，每个 skill 完成后指向下一个 skill 的调用方式。orchestrator skill（vibe-workflow）作为"流程总览"而非实际路由器——它不执行任何步骤，只展示全局视图和快速命令表。

模式特征清单：
- **零 manifest 开销**：没有 plugin.json，直接把 skill 文件放到 `.claude/skills/` 就能使用
- **明确串行依赖**：每个 skill 明确指出"完成后调用 `/vibe-X`"
- **orchestrator 做展示不做路由**：vibe-workflow 给用户一张地图，实际执行靠用户按图索骥
- **hooks 做副作用**：通知、prettier 格式化、git status 展示这些"横切关注"交给 hooks，不污染 skill 主体逻辑
- **轻量安装**：复制 `.claude/` 目录即可，无需 `claude plugin install`

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人开发者快速搭建 AI 辅助开发流程 | ✅ 高度适用 | 零配置，fork 即用 |
| 需要强执行顺序的多阶段工作流 | ✅ 适用 | 串行链条强制顺序 |
| 需要并行子任务的复杂项目 | ❌ 不适用 | 纯 skill 架构无法并行分发 |
| 大型团队共享的标准化流程 | ⚠️ 部分适用 | hooks.json 的 shell injection 未修复，不适合生产环境 |
| 需要跨 session 状态持久化 | ❌ 不适用 | 没有状态文件设计，每次重头开始 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 串行 Skill 链（本仓库） | KhazP/vibe-coding | 零 manifest，极简安装，串行强制顺序 | 无并行能力，orchestrator skill 是"装饰性"的 |
| Command + Agent 分层 | LigphiDonk/OMP | 可注册命令，用户体验一致，支持断点续传 | 安装复杂，10 个 agent 全缺 frontmatter（OMP 的反例） |
| Router + Channels | 0xmariowu/Autosearch | 按意图路由，支持 30+ 渠道扩展 | 配置复杂，不适合小项目 |

### 2.4 改进空间
1. **当前问题**：`hooks.json` 第 31 行用字符串拼接方式把 `tool_input.file_path` 传给 `execSync`，存在 shell injection。**改进做法**：用 `execFile(cmd, [filePath])` 替代 `execSync('cmd ' + filePath)`，彻底消除 shell 解释层。**预期收益**：高危 CVE 级别的安全漏洞消除；2278 star 的项目现在有很多人用，这个风险是真实的。
2. **当前问题**：6 个 skill 全部缺 `model:` 声明，runtime 使用用户默认 model。**改进做法**：vibe-workflow（orchestrator）和 vibe-techdesign（技术设计，需要深度推理）用 `claude-sonnet-4-6`；vibe-research（信息汇聚，机械整理）可以用 `claude-haiku-4-5-20251001` 省成本。**预期收益**：用户明确知道每个阶段的 model 预期，不会因默认 model 变化而行为飘移。
3. **当前问题**：5 个 skill 全部缺 examples block（vibe-build 只有 1 个 Good/Avoid 样式例子，不是完整 input→output 例子）。**改进做法**：每个 skill 加一个"用户触发短语 → 期望的 Claude 输出结构"示例，比如 vibe-research 的例子："我要做一个 Pomodoro timer app → [竞品分析表] + [用户痛点总结]"。**预期收益**：Claude Code 的 skill 路由层用 examples 做语义匹配，有例子的 skill 被正确触发的概率更高。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 审计时得分 **80/100**，安全扫描为 BLOCKED。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| vibe-workflow/SKILL.md | 75 | 缺 allowed-tools；缺 examples |
| vibe-build/SKILL.md | 78 | 缺 allowed-tools；含 vague quantifiers |
| vibe-agents/SKILL.md | 80 | 缺 model；缺 examples |
| vibe-prd/SKILL.md | 80 | 缺 model；缺 examples |
| vibe-research/SKILL.md | 80 | 缺 model；缺 examples |
| vibe-techdesign/SKILL.md | 80 | 缺 model；缺 examples |
| hooks.json | 90 | 安全：HIGH shell injection |

### 3.2 当时值得借鉴的模式

1. **严格串行依赖声明** → 每个 skill 在末尾明确写"此阶段完成后，运行 `/vibe-X`"，用户不会迷失在工作流里。根本原因好：多步骤工作流最大的问题是"下一步是什么"，显式声明比让用户自己猜强得多。如何借鉴：echo-sleuth 的 `/lessons` 命令里可以在末尾说"如果想落地这些教训，运行 `/extract`"。

2. **跨件引用一致性极高** → vibe-workflow 的 Quick Commands 表格、vibe-build 的前置依赖声明、每个 skill 的"下一步"指引，三者完全一致，没有过期引用。根本原因好：作者在更新 skill 时同时更新了所有引用方。如何借鉴：我每次新增或修改 command 后，要检查 CLAUDE.md 和其他 command/skill 里是否有需要更新的引用。

3. **hook 做横切关注** → prettier 格式化、git status 检查、系统通知，全部用 hooks.json 实现，不污染 skill 主体。根本原因好：这些"副作用"如果写在 skill 里会让 skill 变成"做 PRD + 还要 commit + 还要格式化"的大杂烩。分离后 skill 只关注核心逻辑。如何借鉴：echo-sleuth 如果要在每次 /recall 后自动记录搜索历史，应该用 hook 而不是在命令里加额外步骤。

### 3.3 当时的缺陷

1. **[SEC-HIGH] hooks.json 第 31 行 shell injection** → `execSync('npx prettier --write "' + p + '"')` 里 `p` 来自 `tool_input.file_path`，用户只要创建一个文件名包含 `"; rm -rf .` 的文件，就会触发任意命令执行。根本原因：作者在写 hook 时用了字符串拼接而不是参数化调用，典型的初学者模式。自查：我的 echo-sleuth 没有 hook 脚本，没有这个风险。

2. **[BUG] vibe-build 和 vibe-workflow 缺 allowed-tools** → skill 在 body 里指示 Claude 去读 `AGENTS.md`、写 `agent_docs/`、运行 `npm/git` 命令，但 frontmatter 没有声明 `allowed-tools`。根本原因：作者可能不知道 skill [frontmatter](#frontmatter) 里需要声明工具权限，或者觉得"Claude 自己会判断"。自查：我的 echo-sleuth 有 4 个 command 缺 allowed-tools（recall/lessons/recap/timeline），是同类问题。

3. **[QUALITY] 5 个 skill 缺 examples** → 技能没有 input→output 例子，Claude Code 的路由层在多个 skill 并存时可能选错。根本原因：作者重视工作流描述（每个 skill 都有清晰的步骤说明），但忽视了 examples 在技术上的必要性。自查：我的 drama-workshop-skills 的 `short-drama/SKILL.md` 有多个完整例子，这一点做得比 KhazP 好。

### 3.4 当时的优化机会
1. 将 `hooks.json` 第 31 行改用 `execFile(cmd, [filePath])` 消除 shell injection（最高优先级）
2. 给 vibe-build 和 vibe-workflow 的 [frontmatter](#frontmatter) 加 `allowed-tools: Read, Write, Edit, Bash`
3. 给所有 6 个 skill 加 `model:` 声明

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| SEC HIGH: execSync+file_path shell injection | `grep -n "file_path\|execSync" .claude/hooks/hooks.json` | **仍然存在** — 第 31 行仍有 `execSync('npx prettier --write \\"'+p+'\\"')` | 高危安全漏洞约 3 周未修复，用该插件的 2000+ 用户存在风险 |
| BUG: vibe-build 缺 allowed-tools | `grep -n "allowed-tools" .claude/skills/vibe-build/SKILL.md` | **仍然存在** — 文件中无 allowed-tools | 风险依然存在 |
| BUG: vibe-workflow 缺 allowed-tools | `grep -n "allowed-tools" .claude/skills/vibe-workflow/SKILL.md` | **仍然存在** — 文件中无 allowed-tools | 同上 |
| 缺 model 声明 | `grep -n "^model:" .claude/skills/vibe-agents/SKILL.md` | **仍然存在** — 无 model 字段 | 用户默认 model 影响所有 vibe skill 的行为 |

### 4.2 架构演进
从 audit 至今（约 3 周），仓库几乎没有变化。6 个 skill 的核心内容和所有 frontmatter 问题均未改动。这说明维护者可能并不知道存在这些问题（NLPM audit 未公开报告给维护者），或者认为这些是低优先级的格式问题。高危安全漏洞未修复是最值得关注的信号。

### 4.3 新增的可学习模式
暂无明显新增。HEAD 与 audit 时一致。

---

## 五、校准

### 5.1 我已经在做对的
1. **没有 shell injection 风险**：echo-sleuth 没有使用 hooks.json 里的 execSync 字符串拼接模式；scripts/ 目录的 Python 脚本用参数化调用，没有 `subprocess(cmd, shell=True)`。
2. **串行依赖声明的思路**：echo-sleuth 的 `/extract` 命令在完成后会提示用户可以运行 `/dashboard` 查看结果，这与 KhazP 的 skill 链设计思路一致。
3. **hook 分离关注点**：echo-sleuth 的 hook 只做通知类任务（或无 hook），没有把格式化/git 操作混进 skill 里。

### 5.2 挑战 / 验证
- **验证了的判断**："缺 examples 的 skill 在有多个 skill 并存时路由可能出错"——vibe-research/prd/techdesign/agents 这 4 个 skill 都没有 examples，用户同时安装 vibe 全套后，Claude 在理解意图时只能依赖 description 和名称，路由精度下降。这验证了我在 drama-workshop-skills 里坚持写完整例子的做法是对的。
- **挑战了的假设**："高 star 的仓库安全性有保证"——2278 stars 的仓库有一个约 3 周未修复的高危 shell injection，proof 了 stars 和安全性之间没有因果关系。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的项目是否有类似的 execSync 字符串拼接 hook
find ~/.claude -name "hooks.json" -exec grep -n "execSync\|exec(" {} \; 2>/dev/null
# 命中后：检查是否有 tool_input 字段被直接拼入命令，若有则改用 execFile + 参数数组

# 检查我的 echo-sleuth 4 个缺 allowed-tools 的命令
grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
# 命中则在 frontmatter 里按实际工具使用情况添加 allowed-tools 字段

# 检查 drama-workshop-skills 的 skill 是否有 examples
grep -c "## Examples\|## 示例\|## 例子" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md
# 若为 0 则需要在 SKILL.md 加 examples section
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 KhazP 的"串行 skill 链"模式，给 echo-sleuth 的命令加"下一步"提示
   - **为何可行**：echo-sleuth 命令之间有自然的信息流（recall 找到内容 → lessons 提取教训 → extract 写入记忆），但现在每个命令是孤岛
   - **第一步**：在 `commands/lessons.md` 末尾加一行"提取完成后，运行 `/extract` 可将教训写入 CLAUDE.md 或记忆文件"（5 分钟）

2. **想法**：给 echo-sleuth 最常用的 skill（jsonl-core/SKILL.md）加 examples block
   - **为何可行**：jsonl-core 是 echo-sleuth 所有 agent 的基础知识，加 examples 让 agent 更精确地引用它
   - **第一步**：打开 `skills/jsonl-core/SKILL.md`，在文件末尾加 `## Examples` section，给出 1 个"输入：JSONL 行 → 输出：解析后的 Python dict" 的完整例子（15 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 KhazP/vibe-coding-prompt-template 的核心目的**：为"vibe coding"提供 5 步产品开发工作流模板（research→PRD→tech design→agent config→build）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 都是"多阶段工作流"的 skill 集合；都用 orchestrator skill 统领全局；都有明确的阶段串行依赖 | 短剧工坊是创作场景；vibe-coding 是软件开发场景 | 高（串行链、orchestrator 设计直接可参考） |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有清晰的阶段划分（搜索→分析→提取→存储） | echo-sleuth 用 agent 分层，vibe-coding 只用 skill | 中 |
| MarkQWu/claude-for-legal | 低 | 都有多专业 skill | 法律工作流不是串行的，是按领域并列的 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 缺 `model:` 字段 | `grep -L "^model:" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | **命中**：drama-workshop-skills/short-drama/SKILL.md 无 model 字段 | 中（不会功能失效，但 Sonnet vs Haiku 对剧本创作质量影响很大） |
| Skill 缺 examples 不够完整 | `grep -c "## Examples" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | **未命中**：drama-workshop-skills 有完整例子 | 暂无 |
| Hook 中 shell injection | `grep -rn "execSync\|exec(" /tmp/my-repos/MarkQWu-*/. claude/hooks/ 2>/dev/null` | **未命中**：我的项目无 hooks.json 中的 execSync | 暂无 |
| Command 缺 allowed-tools | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **命中**：lessons.md, recall.md, recap.md, timeline.md（4/8 命中） | 中 |

**命中后的具体行动建议**：
- `drama-workshop-skills/short-drama/SKILL.md` → frontmatter 加 `model: claude-sonnet-4-6`（剧本创作需要创意深度，不适合 Haiku）→ 5 分钟
- `echo-sleuth-for-claude/commands/recall.md` → 加 `allowed-tools: Bash, Read` → 3 分钟
- `echo-sleuth-for-claude/commands/lessons.md` → 加 `allowed-tools: Bash, Read` → 3 分钟
- `echo-sleuth-for-claude/commands/recap.md` + `timeline.md` → 各加 `allowed-tools: Bash, Read` → 5 分钟

### 7.3 别人的更优方案

1. **领域**：跨件引用的一致性维护
   - **本案例做法**：vibe-workflow 的 Quick Commands 表格、vibe-build 的前置依赖声明、每个 skill 末尾的"下一步"指引三者完全一致，即使 skill 数量变化也没有过期引用。
   - **我的项目现状**：drama-workshop-skills 的 README.md 里的命令列表有时落后于 SKILL.md 的实际命令（当新增命令时 README 没及时更新）。
   - **如何借鉴**：在 drama-workshop-skills 的 SKILL.md 末尾加一个"相关命令速查表"，与 README 合并成唯一引用源，避免双写；或者 README 的命令表从 SKILL.md 的 frontmatter 自动生成（CI 阶段）。

2. **领域**：用 hooks 实现横切关注点（prettier + git status + 通知）
   - **本案例做法**：格式化、版本控制状态检查、系统通知全部在 hooks.json 里用 PostToolUse/SessionStart 实现，不进入 skill 主体。
   - **我的项目现状**：echo-sleuth 的命令有时在主体里加"完成后运行 `git status` 检查是否有新文件"这类指令，污染了命令的核心关注点。
   - **如何借鉴**：给 echo-sleuth 加一个 `hooks.json`，在 PostToolUse:Write 后触发"更新 session 记录"，让 skill/command 主体专注知识提取而不是副作用。

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安全基线（hook 代码安全性）
- **我的做法**：echo-sleuth-for-claude 和 drama-workshop-skills 都没有 hooks.json，因此不存在 `execSync` 字符串拼接 shell injection 风险
- **本案例做法**（弱在哪）：`hooks.json` 第 31 行 `execSync` 直接拼接用户控制的 `file_path`，是高危 CVE 级别漏洞，且 3 周内未修复
- **意义**：如果我未来添加 hook 脚本，应当用 `execFile(cmd, [arg1, arg2])` 而不是字符串拼接，这个教训现在已经深刻了

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件元数据。Claude Code 的 SKILL.md 需要在 frontmatter 里声明 `name`、`description`、`model`、`allowed-tools` 等字段，让系统知道这个 skill 叫什么、能做什么、用哪个模型、可以调用哪些工具。

### <a name="串行skill链"></a>串行 Skill 链
> 一种 Claude Code 插件架构模式：多个 SKILL.md 按固定顺序串联，每个 skill 完成后显式指向下一个 skill，形成 A→B→C→D→E 的工作流链条。与 command + agent 架构相比，这种模式零 manifest 开销，但缺乏并行能力和断点续传支持。典型特征：有一个"地图 skill"（如 vibe-workflow）展示全流程但不执行，用户自己按图索骥。

### <a name="shell-injection"></a>shell injection（命令注入）
> 当程序把用户提供的内容（如文件名、参数）直接拼接进 shell 命令字符串时，攻击者可以在内容里嵌入特殊字符（`;`, `|`, `` ` ``）来注入额外命令。例如文件名 `foo.ts"; rm -rf .` 会让 shell 先执行 prettier，再执行 `rm -rf .`（删除所有文件）。防御方法：使用 `execFile(cmd, [arg])` 参数化调用，绕过 shell 解释层。
