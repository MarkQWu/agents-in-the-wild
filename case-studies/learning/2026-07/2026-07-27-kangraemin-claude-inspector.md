# kangraemin/claude-inspector — 学习案例

**仓库**：https://github.com/kangraemin/claude-inspector
**Stars**：115 | **来源**：upstream audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-27（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `vague-quantifier`, `manifest-discipline`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-inspector 是一个 **Electron 桌面应用**，以 [MITM 代理](#mitm-proxy)方式实时拦截 Claude Code CLI 流量，让开发者看到每一个发往 API 的 JSON payload，并能用 AI 分析 session 流。

三个关键事实：
1. **工具定位**：开发者调试工具，而非 AI 编码辅助工具——它的用户是「想理解 Claude Code 内部在发什么」的开发者，而不是「想让 AI 写代码」的业务用户。
2. **韩语 NL 工件**：CLAUDE.md 和所有 3 个 agents 用韩文写作，表明这是一个以韩国开发者社区为主要受众的项目。
3. **SECURITY REVIEW（审计时）**：无 Critical/High 发现，但有 Medium 2 个（APPLE_APP_SPECIFIC_PASSWORD 明文暴露 + Sentry 数据外传无告知）和 Low 2 个。

### 1.2 架构剖析

**目录结构（当前 HEAD）**：
```
claude-inspector/
├── public/index.html        # 单文件 112KB Electron 前端（HTML+CSS+JS）
├── main.js                  # Electron 主进程 + MITM 代理服务器
├── preload.js               # Electron preload 脚本
├── assets/icon.*            # 应用图标
├── homebrew-tap/            # Homebrew 安装通道
├── scripts/
│   └── notarize.js          # macOS 代码签名脚本（含 Medium 安全问题）
├── package.json             # 依赖（含 Sentry、Playwright）
├── .claude/
│   ├── agents/
│   │   ├── proxy-analyzer.md  # 分析代理流量和解析逻辑 [BUG: 无 name]
│   │   ├── reviewer.md         # 代码审查 agent [BUG: 无 name]
│   │   └── ui-debugger.md      # UI 调试 agent [BUG: 无 name]
│   └── skills/
│       ├── build/SKILL.md      # npm run dist:mac
│       ├── deploy/SKILL.md     # 发布 workflow
│       └── e2e/SKILL.md        # Playwright e2e 测试
└── CLAUDE.md                   # 开发者指引（韩文，88/100）
```

**文件类型分布**：3 个 agent / 3 个 skill / 0 个 hook / 0 个 command

**编排关系**：  
Skills（build/deploy/e2e）是独立操作入口，无互相调用。Agents（proxy-analyzer/reviewer/ui-debugger）是按需调用的专家，各自聚焦一个诊断域——这是典型的「[单职责 agent](#single-purpose-agent) 平铺架构」。

**跨件契约**：  
`reviewer.md` 第 17 行引用 `~/.claude/rules/review-rules.md`（用户本地文件），新克隆仓库时此文件不存在，reviewer agent 的优先级规则静默丢失。Skills 调用的 npm 脚本（`dist:mac`、`test:unit`、`test:e2e`、`predist`）已在 `package.json` 中验证存在——内部一致性良好，跨件问题集中在外部引用上。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」——Electron 应用是核心（Node.js + Chromium），Claude Code 的 NL 工件只是开发辅助层（帮助开发者维护/调试这个应用）。
- **解决什么问题**：Claude Code CLI 的内部 API 通信对开发者不透明——不知道 context window 用了多少、什么机制在起作用（CLAUDE.md、skill、sub-agent 等）。
- **做了什么 trade-off**：MITM 代理需要在本地 9090 端口运行，要求用户设置 `HTTPS_PROXY=http://localhost:9090`，引入了使用门槛。简单查看 API 日志（如直接捕包）不需要这个复杂度，但 MITM 代理可以解析并关联上下文。
- **反映什么认知模型**：作者认为 Claude Code 本身是「可观测的黑盒」——通过代理拦截和 AI 分析（proxy-analyzer agent）把黑盒变成白盒。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：NL 表皮 + 原生二进制核心**

这个模式的关键特征：
- 特征 1：**核心是原生应用**——Electron/Go/Rust 等构建的实际工具，NL 工件只是开发者工具层
- 特征 2：**NL 工件与应用功能正交**——agents 帮开发者维护这个应用，而非替代应用功能
- 特征 3：**Skills 与 npm scripts 对齐**——`build/SKILL.md` 直接映射到 `npm run dist:mac`，无中间抽象层
- 特征 4：**领域专家 Agent 按需加载**——3 个 agent 各守一个调试域，而非一个万能 agent

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| Electron / macOS 原生应用 | ✅ 高度适用 | NL 工件成为开发者脚手架而非核心逻辑 |
| 纯 NL 插件 / skill 集合 | ❌ 不适用 | 无「原生核心」可包裹 |
| 需要跨平台部署的服务端工具 | ⚠️ 慎用 | notarize.js 等脚本绑定 macOS |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生核心（本仓库） | claude-inspector、leowux/pony | 充分利用原生性能、NL 辅助不妨碍核心 | agent 文件质量欠佳（本案例 3 agent 均无 name） |
| 纯 NL 插件（无原生核心） | review-squad、simmer | 零安装、跨平台 | 无系统级能力 |
| NL 工件 + Python/Node 后端 | airis-mcp-gateway、graphify | 中间道路、可部署 | 安装复杂度中等 |

### 2.4 改进空间

1. **当前问题**：3 个 agents 均缺少 `name` frontmatter 字段，Claude Code 无法注册这些 agent。**改进做法**：在每个 agent 的 `---` frontmatter 块内加 `name: proxy-analyzer`（及对应名）。**预期收益**：从无法注册变为可正常调用，NLPM 分数从 47.7 均值升至约 70+。
2. **当前问题**：`reviewer.md` 引用 `~/.claude/rules/review-rules.md`（用户本地文件），新克隆时静默丢失。**改进做法**：将 `review-rules.md` 提交到仓库 `.claude/rules/review-rules.md`，并在 `reviewer.md` 改用相对路径引用，或在 README 加「首次使用前需手动创建」的说明。**预期收益**：消除跨件引用的静默失败。
3. **当前问题**：3 个 agents 均无 examples 块（各 -15 分）。**改进做法**：为每个 agent 加一个具体调用示例（用户描述问题 → agent 输出诊断结果格式）。**预期收益**：agent 分数从 43-55 升至 70+，用户知道该说什么触发 agent。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 当时得分 **76/100**（7 个文件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/agents/proxy-analyzer.md | 43 | 无 name + 无 examples + 无 output format |
| .claude/agents/ui-debugger.md | 45 | 无 name + 无 examples + 无 output format |
| .claude/agents/reviewer.md | 55 | 无 name + 无 examples |
| CLAUDE.md | 88 | 内容清晰但极简 |
| .claude/skills/e2e/SKILL.md | 100 | 无问题 |
| .claude/skills/build/SKILL.md | 100 | 无问题 |
| .claude/skills/deploy/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **skills 与 npm scripts 精确映射** → `build/SKILL.md` 直接调用 `npm run dist:mac`，无额外抽象；根本原因：单一真实来源原则。应借鉴：skill 命令应直接调用项目已有的脚本，不要在 skill 里重复定义构建逻辑。
2. **skills 100/100 高分** → 3 个 skills 各聚焦单一操作，有 allowed-tools 声明，无模糊量词；根本原因：单职责 + 具体指令。应借鉴：每个 skill 只做一件事，触发条件越具体越好。
3. **CLAUDE.md 精炼 UI 修复原则** → 88/100，3 条韩文原则（诊断先行、确认后提交、简单解法优先）直接可操作；根本原因：CLAUDE.md 约束行为而非描述功能。应借鉴：CLAUDE.md 应写「做什么」和「不做什么」的约束，而非功能说明书。
4. **领域专家 Agent 分工** → proxy-analyzer / reviewer / ui-debugger 各守一个调试域，互不重叠；根本原因：避免通用 agent 的注意力稀释。应借鉴：把不同调试/审查任务拆成独立 agent 而非塞进一个「全能助手」。

### 3.3 当时的缺陷

1. **3 个 agents 均缺 `name` 字段**：agent frontmatter 缺少 `name` 是注册级 bug——Claude Code 无法通过名字调度这些 agent。**为什么会失败**：Claude Code 的 agent 注册机制要求 `name` 字段来索引和调用 agent；没有 `name`，agent 文件在磁盘上存在但对 Claude Code 不可见。自查：我的 agents 有没有都写了 `name:`？
2. **reviewer.md 引用用户本地文件**：`~/.claude/rules/review-rules.md` 在任何新克隆的仓库中都不存在，reviewer agent 的优先级规则（bug > security > perf 等层级）在 fresh clone 时静默丢失。**为什么会失败**：`~/.claude/` 是用户全局目录，不随 git clone 传播——任何引用这个路径的 agent 都会在协作者/CI 环境中静默失效。自查：我的 agents 有无引用 `~/.claude/` 下不在仓库中的文件？
3. **agents 无 examples 块（各 -15 分）**：用户不知道应该说什么来触发 agent，agent 描述（description frontmatter）也是韩语，非韩语用户无法理解。**为什么会失败**：缺少 examples 意味着 Claude 在决定是否调用 agent 时缺少样本锚定，容易跳过该 agent。自查：我的 agents 有没有至少一个 example 块？

### 3.4 当时的优化机会

1. 三个 agents 各加 `name:` 字段（1 分钟可完成，解锁 agent 注册）
2. 三个 agents 各加 `model:` 字段（推荐：proxy-analyzer → sonnet，reviewer → sonnet，ui-debugger → haiku 用于快速 UI 诊断）
3. 将 `review-rules.md` 提交进仓库并改用相对路径

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| proxy-analyzer 无 `name` | `grep -c "^name:" .claude/agents/proxy-analyzer.md` | **持续**：0 个 name 字段 | 注册级 bug，3 个月未修 |
| reviewer 无 `name` | `grep -c "^name:" .claude/agents/reviewer.md` | **持续**：0 个 name 字段 | 同上 |
| ui-debugger 无 `name` | `grep -c "^name:" .claude/agents/ui-debugger.md` | **持续**：0 个 name 字段 | 同上 |
| reviewer → ~/.claude/rules/review-rules.md 孤引用 | `grep -n "review-rules" .claude/agents/reviewer.md` | **持续**：L17 仍引用用户本地文件 | fresh clone 静默失效 |

### 4.2 架构演进

对比 Audit 时（2026-04-19）与当前 HEAD：
- `homebrew-tap/` 目录新增——Homebrew 安装通道上线（`brew install --cask claude-inspector`），显著降低安装门槛
- `public/screenshots/` 新增多语言截图（en-1/2/3、ko-1/2/3）——README 国际化
- NL 工件部分（`.claude/agents/`、`.claude/skills/`）**无任何变化**

这说明作者重心在**用户获取和国际化推广**，而非 NL 工件质量修复。

### 4.3 新增的可学习模式

- **Homebrew Cask 分发**：把 Electron 应用打成 Homebrew Cask，一行命令安装。对比 airis 的 curl|bash，这是更安全、更可信的分发方式。值得借鉴于任何面向 macOS 用户的原生应用。

---

## 五、校准

### 5.1 我已经在做对的

1. **Skills 单职责**：我的 gstack 和 bureau 的 skills 每个都聚焦单一操作——与 claude-inspector skills 100/100 的高分实践一致。
2. **不在 agent 引用用户本地文件**：我的 agents 不引用 `~/.claude/` 下不在仓库中的文件——避开了 reviewer.md 的静默失效反模式。
3. **CLAUDE.md 行为约束而非功能说明**：我的 bureau/CLAUDE.md 包含「不要直接修改 canonical 状态」类约束，与 claude-inspector CLAUDE.md 88/100 的写法一致。

### 5.2 挑战 / 验证

这个案例验证了「skills 和 agents 分工是两种不同工具」的判断：
- Skills 适合「用户主动触发的单步操作」（build/deploy/e2e）
- Agents 适合「需要多步推理的诊断任务」（proxy-analyzer/ui-debugger）

之前我倾向于把所有操作都写成 skill，这个案例让我重新考虑：**诊断类、探索类任务更适合 agent**，因为 agent 可以按需多步推理而不用把所有步骤硬编码在 skill 里。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否都有 name 字段
for f in ~/.claude/agents/*.md /tmp/my-repos/MarkQWu-*/.claude/agents/*.md 2>/dev/null; do
  [[ -f "$f" ]] || continue
  count=$(grep -c "^name:" "$f" 2>/dev/null || echo 0)
  [[ "$count" -eq 0 ]] && echo "MISSING name: $f"
done
```
命中后怎么办：在 frontmatter 的 `---` 块内第一行加 `name: <agent-name>`。

```bash
# 检查我的 agents 是否引用了用户本地文件（~/.claude/）
grep -rn "~/.claude/" /tmp/my-repos/MarkQWu-*/.claude/agents/*.md 2>/dev/null
```
命中后怎么办：将引用的文件提交进仓库，改用相对路径；或在 README 加「首次使用前需手动创建」说明。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 `recall` skill 设计一个专门的 `recall-debugger` agent，专门处理「recall 结果不符合预期」的调试场景。
   - **为何可行**：bureau recall 目前是通过 skill 单步调用，但调试 recall 失败需要多步推理（检查 cabinet 结构 → 检查 query → 检查 trust tier）——agent 更合适。
   - **第一步**：在 `bureau/.claude/agents/recall-debugger.md` 创建 agent，参照 claude-inspector 的 `proxy-analyzer.md` 格式（诊断对象 → 分析目标 → 输出格式）；20 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 kangraemin/claude-inspector 的核心目的**：提供 Claude Code CLI 流量的实时可观测性工具，供开发者调试 AI session。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 低 | 都涉及 AI session 的记录与分析 | bureau 是知识管理，inspector 是流量调试 | 低 |
| MarkQWu/graphify | 低 | 都是 AI 辅助工具 | graphify 是知识图谱，inspector 是代理拦截 | 低 |

我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agents 无 `name` 字段 | `grep -L "^name:" MarkQWu-gstack/.claude/agents/*.md` | gstack 目前无 `.claude/agents/` 目录——无 agent 文件 | 暂无 |
| agents 无 examples（各 -15） | `grep -L "## Example\|<example>" MarkQWu-bureau/skills/*.md` | bureau 所有 skills 均无 `<example>` 块 | 高 |

**命中后的具体行动**：
- `MarkQWu-bureau/skills/review/SKILL.md` → 加一个 `<example>` 块：「用户说"批准这批记录" → review skill 列出待审 proposed 状态记录并等待确认」；10 分钟可完成。

### 8.3 别人的更优方案

1. **领域**：skill 与 npm scripts 精确映射（skills 100/100）
   - **本案例做法**：`build/SKILL.md` 直接调用 `npm run dist:mac`，无额外封装
   - **我的项目现状**：gstack 的 `land-and-deploy/SKILL.md` 包含较长的部署流程，部分步骤在 skill 里重复定义
   - **如何借鉴**：把 gstack 中重复的构建步骤提取到 Makefile 或 npm scripts，skill 只做调用；修改 `land-and-deploy/SKILL.md` 改为引用脚本。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent frontmatter 完整性
- **我的做法**：gstack 目前无 agent 文件，但 bureau 的 skill frontmatter 均包含 `name:` 和 `description:` 字段
- **本案例做法弱在哪**：3 个 agents 均无 `name:` 字段，这是注册级 bug，导致 agents 完全无法被 Claude Code 调用
- **意义**：提醒我在将来添加 agents 时，`name:` 字段是优先级最高的必填项。

---

## 八、术语表

### <a name="mitm-proxy"></a>MITM 代理（Man-in-the-Middle Proxy）
> 在两端通信中间插入一个「代理服务器」，拦截并可修改双方的通信内容。claude-inspector 在本地 9090 端口运行这样一个代理，Claude Code CLI 发出的所有 HTTPS 请求都先经过这个代理——代理记录下 payload，再转发给真正的 Anthropic API。用户可以在 Electron 界面里看到每一条请求。

### <a name="single-purpose-agent"></a>单职责 Agent
> 每个 agent 只专注于一个特定的诊断或操作域，不试图「什么都会」。claude-inspector 的 proxy-analyzer 只分析代理流量，reviewer 只做代码审查，ui-debugger 只调试 UI——三者互不重叠。对比「通用助手 agent」：单职责 agent 在其专注域内的指令质量更高，但需要多个 agent 文件协同。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种架构模式：应用的核心功能由原生语言（Electron/Go/Rust/Node.js）实现，Claude Code 的 NL 工件（skills/agents）只是「开发者工具层」——帮助维护和调试这个应用本身，而不是替代应用的功能。好处：核心不依赖 AI，AI 辅助是加法，不是减法。
