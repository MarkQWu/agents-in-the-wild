# ananddtyagi/cc-marketplace — 学习案例

**仓库**：https://github.com/ananddtyagi/cc-marketplace
**Stars**：N/A | **来源**：upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-12（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `examples-driven`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
cc-marketplace 是一个 Claude Code 插件集市：用单个 Git 仓库托管 60+ 个独立插件，任何人都可以直接从 `plugins/` 目录下安装某个插件。作者 Anand Tyagi 以"平台思维"构建，每个插件作为一个独立单元存在，包含自己的 agent/command/skill/hook。在生态中的位置：介于"个人工具集"（单仓小规模）和"官方 marketplace"（Anthropic 托管）之间的社区中间地带。

关键事实：
- 272 个 NL 工件（2026-04 审查时），60+ 个独立插件
- 采用"插件即目录"的目录约定，每个插件有 `.claude-plugin/plugin.json` 清单
- 插件功能涵盖：growth hacking、代码审查、DevOps、B2B 发布、sugar 生产力工具等
- 存在 PR #44 说明社区已开始贡献

### 1.2 架构剖析

```
cc-marketplace/
├── plugins/
│   ├── api-contract-sync-manager/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/api-contract-sync/SKILL.md   # 92分，最佳实践
│   ├── sugar/
│   │   ├── agents/ (sugar-orchestrator, quality-guardian, task-planner)
│   │   ├── commands/ (sugar-review, sugar-task, sugar-analyze, sugar-status, sugar-run)
│   │   ├── hooks/hooks.json                      # 最佳钩子示范
│   │   └── mcp-server/                           # Node.js MCP 服务
│   ├── claude-dev-infrastructure/
│   │   ├── agents/
│   │   ├── commands/
│   │   ├── skills/ (meta-skill-router, chief-architect, 等9个)
│   │   ├── hooks/hooks.json
│   │   └── dev-manager/server.js                 # Express server
│   ├── math/skills/SKILL.md                      # 90分，代码示例典范
│   ├── growth-hacker/agents/growth-hacker.md     # 35分，无 frontmatter bug
│   ├── b2b-project-shipper/agents/               # name 不匹配 bug
│   └── ... (60+ 个插件)
└── (无顶层 README 统一目录)
```

- **文件类型分布**：约 70 个 agent / 45 个 command / 40 个 SKILL.md / 3 套 hooks
- **编排关系**：插件内部自封装；`claude-dev-infrastructure` 内部有 meta-skill-router 做插件间路由；sugar 插件内 hooks 引用 sugar 命令
- **跨件契约**：每个插件通过 `.claude-plugin/plugin.json` 向 Claude Code 注册自身，各插件相互独立，不存在跨插件依赖

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「插件即目录，目录即产品」——每个插件都是一个完整可安装的单元
- **解决什么问题**：让社区成员能向一个中心仓库贡献独立插件，无需维护独立仓库
- **trade-off**：规模聚合 vs 质量管控。聚合带来了发现性（一仓即浏览），但没有入库审查导致质量参差不齐（73/100 vs 每个仓库独立高分）
- **认知模型**：作者把 Claude Code 生态看成"应用商店"——插件是 APP，marketplace 是发行平台

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：平列式插件集市（Flat Plugin Marketplace）**

每个插件是自包含的目录单元，共享同一个顶级 `plugins/` 命名空间，插件之间没有依赖关系，靠 `plugin.json` 各自声明元数据。

模式特征清单：
- **特征 1**：插件即目录——`plugins/<slug>/` 内包含该插件的全部 NL 工件
- **特征 2**：本地 [manifest](#manifest) 自声明——每个插件用 `.claude-plugin/plugin.json` 注册自身，不依赖顶层清单
- **特征 3**：功能平列，无路由层——插件之间不互调，用户按需选用
- **特征 4**：质量异构——不同插件的作者不同，质量分布从 35 分到 92 分不等
- **特征 5**：MCP + NL 混合——部分插件（sugar、claude-dev-infrastructure）内嵌了 MCP 服务器，功能超出纯 NL 工件范围

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 社区贡献型多主题工具集 | ✅ 高度适用 | 低门槛贡献，目录即边界 |
| 单作者深度工具集（如个人 skills 仓）| ❌ 不适用 | 共享仓没有质量上限，个人品牌用独立仓更合适 |
| 需要跨插件编排的工作流 | ❌ 不适用 | 平列式没有路由层，跨插件调用无标准机制 |
| 入门级快速试验 | ✅ 适用 | 可 fork 任意子目录，成本低 |
| 企业生产部署 | ❌ 不适用 | 质量参差不齐，无审查，供应链风险 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 平列式集市（本仓库）| ananddtyagi/cc-marketplace | 发现性强，贡献门槛低 | 质量不可控，无跨插件标准 |
| 单仓单领域纵深（domain-tree）| alirezarezvani/claude-skills | 质量统一，领域知识深 | 贡献门槛高，需理解整体架构 |
| 独立仓单插件 | affaan-m 的每个插件 | 版本独立，可按仓库 star 评分 | 发现性差，用户需分别安装 |
| 官方 marketplace（Anthropic）| claude.ai/plugins | 审查有保障，深度集成 | 封闭，贡献须审批 |

### 2.4 改进空间
1. **当前问题**：无统一入库审查导致 5 个 agent 无 [frontmatter](#frontmatter)，直接导致无法注册。**改进做法**：在 PR 模板中加必选 NLPM Score 截图（≥70 分），或跑 CI `nlpm-check` 门禁。**预期收益**：消灭所有注册级 bug，最低分从 35 提到 60+。
2. **当前问题**：45 个 command 中 38 个缺 `name` 字段，路由失效。**改进做法**：提供标准 command [模板](#模板文件)（含 `name`、`description`、`allowed-tools` 占位符），要求贡献者从模板派生。**预期收益**：新增 command 0 遗漏率。
3. **当前问题**：~70 个 agent 没有声明 `model`，运行时回退到默认模型，成本不可预测。**改进做法**：在 `plugin-creator` SKILL.md 中明确写"必须声明 `model: sonnet` 或 `model: opus`"，让 SKILL 成为约束文档。**预期收益**：新 agent 遵从率提升，审查者有明确依据。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **73/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| plugins/growth-hacker/agents/growth-hacker.md | 35 | 无 YAML frontmatter，以 `# Growth Hacker` 标题开头 |
| plugins/reddit-community-builder/agents/... | 35 | 无 YAML frontmatter |
| plugins/b2b-project-shipper/agents/... | 72 | `name: project-shipper`（应为 `b2b-project-shipper`） |
| plugins/desktop-app-dev/agents/... | 72 | frontmatter 全部字段写在一行（格式错误） |
| plugins/code-review/commands/code-review.md | 60 | 无 `name`、无 `allowed-tools`、无示例 |
| plugins/api-contract-sync-manager/skills/... | 92 | 近满分，最佳实践参考 |

### 3.2 当时值得借鉴的模式

1. **`allowed-tools` 显式声明** → 权限最小化，防止 agent 越权操作  
   原文路径：`plugins/api-contract-sync-manager/skills/api-contract-sync/SKILL.md`  
   借鉴方式：每个 command 和 skill 在 frontmatter 里声明实际用到的工具，不留通配符

2. **代码示例驱动 SKILL** → 行为 100% 可验证，消除歧义  
   原文路径：`plugins/math/skills/SKILL.md`（name: math-tools，内嵌 SymPy 代码示例）  
   借鉴方式：skill 不只写"你会做什么"，写"input → expected output"的可执行示例

3. **hook 限流守卫** → 高频触发不失控  
   原文路径：`plugins/sugar/hooks/hooks.json`（11 个 hook，含 throttle 配置）  
   借鉴方式：每个 PostToolUse hook 加限频条件，避免无限递归或 API 超限

4. **Express + NL 混合架构** → NL 指挥，服务层执行  
   原文路径：`plugins/claude-dev-infrastructure/dev-manager/server.js`  
   借鉴方式：对需要持久状态的插件，NL 层作为接口，底层跑轻量 HTTP 服务

### 3.3 当时的缺陷

1. **5 个 agent 完全无 frontmatter**  
   根本原因：作者把 agent 当成"说明文档"而非"可注册配置文件"，忽略了 Claude Code 需要解析 [frontmatter](#frontmatter) 才能注册的机制。直接后果是这些 agent 永远不会被加载。  
   自查：我有没有犯同样的错？→ 检查我的仓库中所有 .md agent 文件是否都有 `---` 包围的 frontmatter。

2. **38/45 个 command 缺 `name` 字段**  
   根本原因：作者用文件名充当隐式 name，没意识到 Claude Code 路由依赖 frontmatter 中显式的 `name` 字段，文件名只是目录结构，不参与注册。  
   自查：我的 command 文件里有没有漏写 `name`？

3. **70 个 agent 未声明 `model`**  
   根本原因：大量贡献者不了解不声明 model 会导致运行时用全局默认模型（通常 sonnet），某些需要深度推理的 agent（如 planning-prd-agent 使用了 `model: opus`）会出现能力与期望不符。  
   自查：我的每个 agent 是否都声明了 `model`？

4. **`b2b-project-shipper` name 不匹配**  
   根本原因：作者从其他插件复制模板时没有更新 `name` 字段，`project-shipper` 和目录名 `b2b-project-shipper` 不一致，会导致任何通过 name 引用该 agent 的 command 失效。  
   自查：我的 agent `name` 字段是否与目录名一致？

### 3.4 当时的优化机会
1. **统一 command [模板](#模板文件)**：为所有 command 提供标准起点，包含 `name`、`description`、`allowed-tools` 必填字段
2. **批量 model 声明**：按 PR 批次为 ~70 个 agent 补充 `model: sonnet` 声明
3. **`meta-skill-router` frontmatter 修复**：补充缺失的 `name`、`description`，使路由层本身可以被正确加载

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| growth-hacker.md 无 frontmatter | `head -3 plugins/growth-hacker/agents/growth-hacker.md` → 看是否以 `---` 开头 | **持续** — 文件仍以 `# Growth Hacker` 标题开头 | 3 个月未修，作者可能不知道或不认为是 bug |
| b2b-project-shipper name 不匹配 | `grep "^name:" plugins/b2b-project-shipper/agents/b2b-project-shipper.md` | **持续** — `name: project-shipper` 仍未修改 | PR #44 已开，但尚未包含此修复 |
| commands 缺 `name` | `grep -rL "^name:" plugins/*/commands/*.md \| wc -l` | **部分改善** — 从 38 降至约 36 | 社区贡献者开始注意，但系统性问题仍在 |

### 4.2 架构演进
PR #44 显示社区在贡献新插件，整体目录结构保持不变（`plugins/<slug>/` 约定稳定）。新增插件仍以"平列"方式接入，没有引入路由层。根据 PR 状态（OPEN），作者采用审查合并模式，但没有自动化 CI 门禁。

### 4.3 新增的可学习模式
当前 HEAD 未发现与 audit 相比显著的新架构模式。插件数量可能已增加，但结构范式未变。

---

## 五、校准

### 5.1 我已经在做对的
1. **frontmatter 规范**：我的 bureau/commands/ 文件都有 `---` frontmatter，不存在完全无 frontmatter 的情况
2. **插件清单同步意识**：graphify 和 echo-sleuth 的 plugin.json 与实际文件保持同步
3. **hook 使用有节制**：bureau 的 hooks/ 不像 sugar 那样铺 11 个钩子，风险更低

### 5.2 挑战 / 验证
本案例验证了一个判断：**集市模式的核心风险不是功能缺陷，而是注册元数据缺失**。一个 agent 再聪明，没有 frontmatter 就永远不会被加载——这是比"逻辑写错了"更根本的失败。这强化了我对 `manifest-discipline` 的重视：先让工件能被正确识别，再优化功能内容。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我所有 command 文件是否有 name 字段
find ~/.claude -name "*.md" -path "*/commands/*" | xargs grep -L "^name:" 2>/dev/null
```
命中后：打开文件，在 frontmatter 第一行后加 `name: <slug>`，slug 应与文件名一致。

```bash
# 检查 agent 文件是否有 model 声明
find ~/.claude -name "*.md" -path "*/agents/*" | xargs grep -L "^model:" 2>/dev/null
```
命中后：根据 agent 任务复杂度，添加 `model: sonnet`（通用）或 `model: opus`（需要深度推理）。

```bash
# 检查 name 字段与文件名是否一致
for f in ~/.claude/agents/*.md; do
  slug=$(basename "$f" .md)
  name=$(grep "^name:" "$f" 2>/dev/null | head -1 | sed 's/name: //')
  [ "$slug" != "$name" ] && echo "MISMATCH: $f -> name=$name"
done
```
命中后：修改 `name` 字段使其与文件名（不含 .md 后缀）一致。

```bash
# 检查 command 是否声明了 allowed-tools
find ~/.claude -name "*.md" -path "*/commands/*" | xargs grep -L "allowed-tools" 2>/dev/null
```
命中后：在 frontmatter 中加 `allowed-tools: [Read, Bash]`（根据实际工具需求填写）。

### 6.2 灵感 → 实施路径

1. **想法**：为我的 bureau 仓库创建 command 模板文件
   - **为何可行**：bureau 现在有 5 个 command，都已手动写好，但新增 command 时容易遗漏字段
   - **第一步**：在 `bureau/templates/command.md.tmpl` 里写好标准 frontmatter 骨架，包含 `name`、`description`、`argument-hint`、`allowed-tools` 占位符；约 10 分钟

2. **想法**：在 echo-sleuth 的 CI 中加 `nlpm-check` 门禁
   - **为何可行**：nlpm-check 是纯 Python stdlib，无需安装，可直接在 GitHub Actions 中运行
   - **第一步**：复制 `templates/workflows/nlpm-check.yml` 到 echo-sleuth 的 `.github/workflows/`；约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新）

### 8.1 目的对齐度

- **本案例 ananddtyagi/cc-marketplace 的核心目的**：聚合多个独立 Claude Code 插件，提供社区贡献的插件集市
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 也是多工具集合，多个独立功能（browser-skills、pair-agent 等）共存一仓 | gstack 是单作者高内聚，cc-marketplace 是多作者松耦合 | 高 |
| MarkQWu/bureau | 中 | 也有多个 command + skill + hook 组件 | bureau 是单一应用（知识库管理），不是插件集市 | 中 |
| MarkQWu/graphify | 低 | 都有 SKILL.md | graphify 是领域专精工具，不是多插件容器 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| command 缺 `name` 字段 | `grep -L "^name:" /tmp/my-repos/MarkQWu-bureau/commands/*.md` | **命中**：bureau 的 compile.md 等 command 文件 frontmatter 缺 `name` 字段（只有 `description` 和 `argument-hint`）| 高 |
| agent 缺 `model` 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/` | **命中**：echo-sleuth agents/ 下多个 agent 无 `model` 字段 | 中 |
| command 缺 `allowed-tools` | `grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md` | **命中**：bureau 的命令文件均未声明 `allowed-tools` | 高 |

**命中后的具体行动建议**：
- `bureau/commands/compile.md` → 在 frontmatter 加 `name: bureau:compile`；5 分钟
- `bureau/commands/*.md` → 逐一添加 `allowed-tools: [Read, Bash, Write]`（按实际使用）；30 分钟
- `echo-sleuth/agents/analyze.md` 等 → 添加 `model: sonnet`；10 分钟

### 8.3 别人的更优方案

1. **领域**：代码示例驱动的 SKILL 设计
   - **本案例做法**：`plugins/math/skills/SKILL.md` 内嵌完整的 SymPy Python 代码示例，精确描述工具行为（输入 → 输出 → 异常处理）
   - **我的项目现状**：`echo-sleuth/skills/` 下的 SKILL.md 大多只有文字描述，缺乏可运行的示例片段
   - **如何借鉴**：在每个 skill 的 `## Examples` 下加 1-2 个完整示例（输入格式 + 期望输出），约 15-20 分钟/skill

2. **领域**：hook 节流设计
   - **本案例做法**：`plugins/sugar/hooks/hooks.json` 对 PostToolUse 事件加 throttle 配置，防止无限触发
   - **我的项目现状**：bureau 的 hooks/ 也有 PostToolUse，但没有 throttle 配置
   - **如何借鉴**：参考 sugar 的 throttle 字段语法，在 bureau/hooks/ 里加 `throttleMs: 5000`

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent 权限控制意识
- **我的做法**：echo-sleuth 的 analyze.md 等 agent 通过 skills 字段声明依赖，使用方式明确
- **本案例做法（弱在哪）**：cc-marketplace 有 wiki agent（cs-wiki-linter）声明了 Write/Edit 工具但角色是只读审计，权限与职责不符（这是 alirezarezvani 的问题，见案例 4）
- **意义**：我的 echo-sleuth skill 设计中，角色和工具权限对齐程度更好；若被审查这是正向亮点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读 agent 文件和 SKILL.md 时先解析 frontmatter 才能知道这个工件怎么注册和调用。没有 frontmatter 的文件等于对 Claude Code 隐形。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。cc-marketplace 里每个插件目录下的 `.claude-plugin/plugin.json` 就是这个插件的 manifest——里面列出所有 commands、skills、agents 的路径。

### <a name="模板文件"></a>模板文件
> 预先写好标准结构的"空白文件"，新建文件时从模板复制，再填入具体内容。避免从零开始导致遗漏必填字段。类似表单里的"必填项目"。

### <a name="MCP服务器"></a>MCP 服务器
> Model Context Protocol Server，运行在本地的小型 HTTP 服务，Claude Code 可以通过工具调用与它通信。适合需要持久状态、文件系统访问或复杂逻辑的功能——这些功能用纯 Markdown NL 工件难以表达，用 MCP 服务器承接。

### <a name="路由层"></a>路由层
> 在多 agent/skill 架构中，负责"把任务分派给哪个子 agent"的中间层。cc-marketplace 的 `meta-skill-router` SKILL.md 就是一个路由层组件。没有路由层的集市里，用户需要自己知道调用哪个插件。
