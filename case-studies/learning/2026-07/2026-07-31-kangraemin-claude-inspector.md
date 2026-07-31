# kangraemin/claude-inspector — 学习案例

**仓库**：https://github.com/kangraemin/claude-inspector
**Stars**：115 | **来源**：upstream audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-31（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `examples-driven`, `single-purpose`, `cross-reference`, `nl-binary-hybrid`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-inspector 是一个 Electron 桌面应用（韩语 UI），用于「捕获并可视化 Claude Code 消息传递机制」——它以代理服务器形式拦截 Claude 的所有请求，让开发者能直观地看到哪些机制（CLAUDE.md、Output Style、Slash Command、Skill、Sub-Agent）被激活了。代码库以韩语为主，README.md 提供英韩双语，面向 Claude Code 深度用户。

关键事实：
- Electron + Node.js 原生应用（`main.js` 是核心，`analytics.js` 集成了 Sentry 错误追踪）
- NL 层极轻：3 个 agents + 3 个 skills，全部用于辅助维护 inspector 本身
- 115 Stars，是调试工具品类里的高质量小仓库
- 状态：`status=contributed`，NLPM 已提交 3 个 PR（#2/#3/#4），3 个月来全部 OPEN

### 1.2 架构剖析

```
claude-inspector/
├── .claude/
│   ├── agents/
│   │   ├── proxy-analyzer.md    # 分析代理流量，诊断解析逻辑（43/100，最低分）
│   │   ├── reviewer.md          # 代码审查，引用外部 review-rules.md（55/100）
│   │   └── ui-debugger.md       # 调试 Electron UI（45/100）
│   └── skills/
│       ├── build/SKILL.md       # 构建步骤（100/100）
│       ├── deploy/SKILL.md      # 发布流程（100/100）
│       └── e2e/SKILL.md         # E2E 测试（100/100）
├── main.js                      # Electron 主进程 + 代理服务器逻辑
├── public/index.html            # UI（包含 parseClaudeMdSections 等关键函数）
├── analytics.js                 # Sentry 错误追踪
└── package.json                 # notarize.js 在 afterSign 钩子调用
```

- **文件类型分布**：3 个 agent + 3 个 SKILL.md + 0 个 command（NL 层只服务于内部维护）
- **编排关系**：agents 直接读取项目文件（main.js、public/index.html），skills 封装 npm scripts
- **跨件契约**：reviewer.md 引用 `~/.claude/rules/review-rules.md`（外部文件，非仓库内）——这是最突出的架构问题

### 1.3 设计思路 / 方法论

- **核心哲学**：「原生应用 + 轻量 NL 层」——Inspector 的核心能力在 Electron 原生代码，NL 层只是开发者自用的维护脚手架
- **解决的问题**：Claude Code 开发者的「黑盒调试困难」——各种上下文注入机制的可见性问题
- **Trade-off**：skills（build/deploy/e2e）做到了极致简洁（100 分），agents 却因为缺少 `name:` 字段全部损坏。说明作者更熟悉 skills 格式，对 agents 的 [frontmatter](#frontmatter) 要求不够了解
- **架构分层**：「高质量 skills + 低质量 agents」这种不对称本身就是一个信号：作者在两类 NL 工件上投入的精力和理解深度有明显差距

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「NL 表皮仅服务于自身维护」**模式：整个 NL 层不对外提供功能，只是项目开发者自用的「AI 辅助开发工具箱」。skills 负责运维（build/deploy/test），agents 负责诊断调试。

模式特征：
- NL 层文件数 ≤6
- 每个 NL 工件都有对应的项目内文件（main.js、package.json）作为操作对象
- NL 层不暴露给终端用户，只对仓库贡献者有用
- Skills 质量往往高于 agents（因为 skills 更容易封装确定性操作）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有固定构建/发布流程的应用仓库 | ✅ 适用 | skills 封装确定性操作效果极好 |
| 需要 AI 诊断复杂运行时行为的项目 | ✅ 适用 | agents 可以读取源码做深度分析 |
| 面向外部用户提供 AI 功能的工具 | ❌ 不适用 | NL 层无用户朝向的设计 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 仅服务内部维护（本案） | claude-inspector | 高内聚，skills 精准 | agents 容易被忽视，quality 不对称 |
| NL 既服务内部又服务用户 | c0x12c/ai-toolkit | 覆盖面广 | 维护成本高 |
| 无 NL 层的纯应用仓库 | 大多数 Electron 应用 | 无额外维护 | 无 AI 辅助维护能力 |

### 2.4 改进空间

1. **当前问题**：3 个 agents 全部缺 `name:` 字段，无法被 Claude Code 注册。**改进做法**：在每个 agent [frontmatter](#frontmatter) 加一行 `name: <agent-name>`（三个文件各 2 分钟）。**预期收益**：agents 立即可用，损坏的功能恢复。

2. **当前问题**：reviewer.md 引用 `~/.claude/rules/review-rules.md`（用户级外部文件）。**改进做法**：把 review-rules.md 提交到仓库内 `.claude/rules/review-rules.md`，或在 README 里说明「克隆后需手动创建此文件」。**预期收益**：任何新贡献者 clone 后即可使用 reviewer agent。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **76/100**，SECURITY **REVIEW**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| proxy-analyzer.md | 43 | 无 name + 无示例 + 无输出格式 |
| ui-debugger.md | 45 | 无 name + 无示例 + 无输出格式 |
| reviewer.md | 55 | 无 name + 无示例 |
| CLAUDE.md | 88 | 内容清晰，最高分的 project-config |
| e2e/SKILL.md | 100 | 无问题 |
| build/SKILL.md | 100 | 无问题 |
| deploy/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **Skills 三连 100 分** → 原因：build/deploy/e2e 每个只做一件事，步骤清晰，对应 package.json 中的真实 npm script。这是「单职责 skill」的教科书案例。

2. **CLAUDE.md 的韩英双层架构** → CLAUDE.md 简洁（88 分），把 UI 约束精确地写成代码级规则（`display:block + overflow-y:auto` 这种级别），而不是模糊的「保持响应式」。

3. **Notarize.js 注册在 afterSign 钩子** → package.json 正确注册，`afterSign` 指向存在的 `scripts/notarize.js`，没有 manifest 漂移。

### 3.3 当时的缺陷

1. **3 个 agents 缺 `name:` 字段（registration-breaking bug）** → 根本原因：Claude Code 的 agents [frontmatter](#frontmatter) 格式与 skills 不同，但作者只检查了 `description` 是否存在，忽视了 `name:` 的必须性。这是新人最常见的 agent 错误之一。**自查**：我的 agents 是否都有 `name:`？

2. **reviewer.md 引用用户级外部文件** → 根本原因：`~/.claude/rules/` 是用户的 global Claude 配置，在任何其他人的机器上都不存在。把用户级路径硬编码进仓库是 portability 反模式。**自查**：我的 agents 有没有引用 `~/.` 开头的路径？

3. **无示例（所有 3 个 agents）** → 这是与 skills 的最大反差——skills 因为步骤式结构自然而然有了示例，agents 的会话式结构需要主动添加 `<example>` block。

### 3.4 当时的优化机会

1. 每个 agent 加 `name: <agent-name>` 一行
2. review-rules.md 提交到仓库内
3. APPLE_APP_SPECIFIC_PASSWORD 改为 notarytool keychain profile

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| proxy-analyzer.md 无 `name:` | `grep "^name:" .claude/agents/proxy-analyzer.md` | **持续**：无 name 字段 | PR #2 已提交 3 个月，维护者无响应 |
| reviewer.md 无 `name:` | `grep "^name:" .claude/agents/reviewer.md` | **持续**：无 name 字段 | PR #3 同上 |
| ui-debugger.md 无 `name:` | `grep "^name:" .claude/agents/ui-debugger.md` | **持续**：无 name 字段 | PR #4 同上 |
| reviewer.md 引用外部 rules | `grep "review-rules" .claude/agents/reviewer.md` | **持续**：`~/.claude/rules/review-rules.md` 第 17 行 | 孤儿引用仍未修 |

**关键观察**：这是本批案例中 PR 被忽视情况最严重的仓库。3 个 PR 自 2026-04-22 开放至今（2026-07-31），将近 100 天，维护者零回应。每个 PR 的修改量极小（1 行代码），修复难度接近于零，但仍未被合并。

### 4.2 架构演进

从审计（2026-04-19）到现在，仓库结构无明显变化。目录树、agents/skills 组成、CLAUDE.md 内容基本稳定。活跃开发集中在 Electron 核心（main.js 的代理逻辑和 UI 调试），NL 层被完全忽视。

### 4.3 新增的可学习模式

暂无。

---

## 五、校准

### 5.1 我已经在做对的

1. **Skills 用来封装确定性操作**：我的 graphify skills 和 gstack skills 都遵循这一原则
2. **CLAUDE.md 写代码级约束**：WorldMonitor/CLAUDE.md 有具体的文件路径和架构约束，避免了模糊描述
3. **name: 字段声明**：gstack 和 WorldMonitor 的所有 skills 都有 `name:` 字段

### 5.2 挑战 / 验证

**验证了的做法**：PR 被无限期忽视（OPEN 3 个月）是现实——开源维护者的注意力有限。这验证了一个教训：给外部项目提 PR 时，要**在第一个 PR 就解释清楚背景和价值**，否则维护者无法快速判断是否合并。NLPM 的 PR 有没有附上清晰的「为什么这个 bug 是真正的 bug」的解释？

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我所有 agents 是否有 name: 字段
for f in $(find . -path "*/agents/*.md" -not -path "*/node_modules/*"); do
  name_count=$(grep -c "^name:" "$f" 2>/dev/null)
  if [ "$name_count" -eq 0 ]; then
    echo "MISSING name: field — $f"
  fi
done
```
命中后怎么办：在 frontmatter 的 `---` 内的首行加 `name: <snake-case-name>`，保证与文件名一致。

```bash
# 检查 agents 有没有引用 ~/.claude/ 路径
grep -rn "~/\.claude" .claude/agents/ 2>/dev/null && echo "WARNING: hardcoded user-level paths found"
```
命中后怎么办：把引用的文件提交到仓库内（`.claude/rules/`），或改为相对路径，或在 README 注明「需手动创建」。

### 6.2 灵感 → 实施路径

1. **想法**：每次新建 agent 文件时，用 snippet/template 保证 `name:`, `description:`, `model:` 三字段都包含
   - **为何可行**：kangraemin 的失败完全因为缺一行，snippet 可以消灭这类机械性遗漏
   - **第一步**：在编辑器里创建一个 agent frontmatter snippet 模板，包含 `name:`, `description:`, `model:`, `tools:` 占位符（5 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：Claude Code 机制可视化调试工具（Electron 桌面应用）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/- | 低 | 均为开发者工具 | WorldMonitor 是情报仪表盘，非调试工具 | 低 |
| MarkQWu/gstack | 中 | 均有 Claude Code NL 层 | gstack 面向用户而非内部维护 | 中 |
| MarkQWu/graphify | 低 | 均为开发者辅助工具 | graphify 用于知识图谱，无 Electron | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 缺 `name:` 字段 | `grep -rL "^name:" /tmp/my-repos/MarkQWu-gstack/.claude/agents/` | gstack 无 .claude/agents/ 目录，无此问题 | 无 |
| Skills 缺 `allowed_tools` | 见 §7.2 脚本 | gstack 4 个 openclaw skills 中无 `allowed_tools` 字段 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack/openclaw/skills/gstack-openclaw-ceo-review/SKILL.md` → 在 frontmatter 加 `allowed_tools: [Read, Glob, Grep, WebSearch]`（5 分钟/文件）

### 8.3 别人的更优方案

1. **领域**：Skills 封装确定性操作的极致简洁
   - **本案做法**：build/deploy/e2e 三个 SKILL.md 各得 100 分，每个只做一件事，步骤化描述，直接对应 `package.json` 的 npm scripts
   - **我的项目现状**：gstack 的 skills 相对较长，包含多个子场景，结构偏复杂
   - **如何借鉴**：审查 gstack skills，把复合步骤拆分为「一个 skill 只做一件事」，每步对应一个可验证操作

### 8.4 反向：我的项目做得比他们好的地方

MarkQWu/gstack 的 `pair-agent/SKILL.md` 和 `skillify/SKILL.md` 均包含 `allowed_tools` 字段（本案所有 agents 缺 `name:` 字段），说明我在 skill 层面的 frontmatter 规范性优于 kangraemin 在 agent 层面的规范性。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 agent.md 或 SKILL.md 时，先解析 frontmatter 才能知道这个 agent/skill 如何注册和调用。**`name:` 字段是必须的**——没有它，Claude Code 找不到这个 agent，就像一个没有名字的联系人，无法被呼叫。

### <a name="electron"></a>Electron
> 用 Web 技术（HTML/CSS/JavaScript）开发跨平台桌面应用的框架。`main.js` 是 Electron 的主进程，负责系统级操作（如启动代理服务器）；`index.html` 是渲染进程（UI 部分）。claude-inspector 用 Electron 拦截 Claude Code 的网络请求，把通信内容渲染成可视化界面。
