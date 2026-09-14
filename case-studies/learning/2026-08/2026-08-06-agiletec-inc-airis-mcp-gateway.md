# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-08-06（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `manifest-discipline`, `examples-driven`, `offline-capable`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`airis-mcp-gateway` 是一个面向本地开发者的 [MCP 网关](#mcp网关)基础设施：它在开发机器上启动多个 MCP 服务（数据库、调试、实现、研究），通过一个统一网关让 Claude Code 接入，并附带一套 Claude Code 插件（11 个 NL 工件）作为操作界面。151 颗星，属于有一定社区影响力的实用基础设施项目。安装方式是 curl pipe to bash，这也是它最大的安全隐患所在。

### 1.2 架构剖析

- **目录结构**：
  ```
  .
  ├── apps/
  │   ├── api/                 # Python FastAPI 后端（MCP 网关核心）
  │   └── gateway-control/     # Node.js 前端控制面板
  ├── ops/
  │   └── autostart/           # 系统自启动脚本（Linux/macOS）
  ├── scripts/
  │   ├── airis-gateway        # 主控制 shell 脚本
  │   ├── airis_bootstrap.py   # Python 引导脚本
  │   └── quick-install.sh     # 快速安装脚本
  ├── skills/
  │   ├── .claude-plugin/plugin.json
  │   └── skills/
  │       ├── mcp-database/SKILL.md
  │       ├── mcp-debugging/SKILL.md
  │       ├── mcp-implementation/SKILL.md
  │       └── mcp-research/SKILL.md
  ├── config/bootstrap/assets/claude/commands/
  │   ├── airis-capability-router.md   # 能力路由命令
  │   └── airis-research-first.md      # 研究优先命令
  ├── .claude/commands/
  │   ├── troubleshoot.md
  │   ├── test.md
  │   └── status.md
  └── install.sh               # 主安装脚本（含 curl|bash 风险）
  ```
- **文件类型分布**：0 个独立 agent / 4 个 SKILL.md / 5 个 command / 0 个 Claude hook（大量 shell script）
- **编排关系**：`airis-capability-router.md` 是关键路由层——用户通过它告诉 Claude "我需要什么类型的能力"，然后 Claude 选择调用对应的 MCP skill（database/debugging/implementation/research）。这是一个轻量的 router 模式。
- **跨件契约**：`mcp-debugging/SKILL.md` 第 8 行要求用户先调用 `superpowers:systematic-debugging`，但 `plugin.json` 里没有声明 `superpowers` 插件为依赖——这是一个悬空的外部依赖，没有装 superpowers 的用户会收到无法执行的指引。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「用 NL 命令封装系统级能力」——重型基础设施（MCP 服务器、Python API、Node.js 面板）由代码实现，用户操作界面（route、troubleshoot、test）由 NL 命令封装，形成「NL 表皮 + 原生代码核心」的[混合架构](#nl表皮原生代码核心)。
- **解决什么问题**：本地开发环境通常需要同时运行多个 MCP 服务，配置繁琐且各自孤立。airis-mcp-gateway 提供统一入口，解决服务管理和 Claude Code 接入的碎片化问题。
- **做了什么 trade-off**：选择 curl pipe bash 安装方式（低摩擦但高风险）vs 包管理器安装（高安全但依赖发布流程）。这个 trade-off 是整个仓库的核心安全问题。
- **反映什么认知模型**：作者把 Claude Code NL 工件作为"可编程 API"来设计——`airis-capability-router.md` 是一个路由层而不是说明文档，`troubleshoot.md` 是一个结构化的 diagnostic workflow 而不是 FAQ。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：NL 表皮 + 原生代码核心（NL Shell + Native Core）**

插件的 NL 层（commands + skills）是轻薄的，真正的工作由底层的 Python API 和 shell 脚本完成。NL 命令负责"告诉 Claude 怎么调用核心"，不负责"自己执行逻辑"。

模式特征清单：
- NL 文件（SKILL.md / command）是接口声明，不含业务逻辑
- 业务逻辑在独立的可执行程序里（Python、Go、shell script）
- 用户看到的是 NL 命令，执行的是底层程序
- 安全边界在底层程序里，而不是 NL 文件里

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要系统级操作（启停服务、文件系统写入） | ✅ 高度适用 | NL 层做决策，原生程序做执行，权限边界清晰 |
| 纯对话/内容生成场景 | ❌ 不适用 | 过度工程，NL 直接生成更快 |
| 跨平台分发（Linux/macOS/Windows） | ⚠️ 部分适用 | shell 脚本跨 Windows 困难；需要 Go 或 Python 才能真正跨平台 |
| 需要高频安装/更新的工具 | ❌ 慎用 | curl pipe bash 安装方式有供应链风险 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮+原生代码核心（本仓库） | airis-mcp-gateway | 系统能力强，安全边界清晰 | 维护两套代码栈，安装风险高 |
| 纯 NL 工件集 | alirezarezvani/claude-skills | 零依赖，纯文本，安装无风险 | 受 Claude 上下文窗口限制，无法做系统操作 |
| NL 表皮+Go 原生核心 | 777genius/claude-notifications-go | 极致性能，安全验证更容易 | Go 学习成本高，更新流程重 |

### 2.4 改进空间

1. **当前问题**：4 个 MCP skill 文件全部缺少 `## Examples` section。**改进做法**：为每个 skill 补充 2 个 input→output 示例（如"数据库连接失败时 Claude 应该调用什么"）。**预期收益**：NLPM 分数从当前 83–85 提升到 98+；用户理解 skill 用途的时间从"读文档"到"看例子"。
2. **当前问题**：`mcp-debugging/SKILL.md` 引用外部插件 `superpowers:systematic-debugging` 但未在 `plugin.json` 声明依赖。**改进做法**：在 `plugin.json` 加 `requires: ["superpowers"]` 字段，或在 skill 里加 graceful degradation（如果没有 superpowers，用如下替代方案…）。**预期收益**：消除 cross-component 悬空引用。
3. **当前问题**：`install.sh` 默认版本是 `latest`，每次安装可能拿到不同版本。**改进做法**：在 install.sh 里固定版本号为最近的 stable tag，通过环境变量允许 override。**预期收益**：消除 SEC-unpinned-semver 问题，安装可重现。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-26 当时得分 **90/100**（BLOCKED 因安全问题）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `airis-research-first.md` | 78 | 无输出格式 + 步骤未编号 |
| `skills/mcp-database/SKILL.md` | 83 | 无示例 + 模糊词"appropriate caution" |
| `skills/mcp-debugging/SKILL.md` | 85 | 无示例 |
| `skills/mcp-implementation/SKILL.md` | 85 | 无示例 |
| `skills/mcp-research/SKILL.md` | 85 | 无示例 |
| `.claude/commands/troubleshoot.md` | 90 | 无 empty $ARGUMENTS 处理 |
| `skills/.claude-plugin/plugin.json` | 100 | 满分 |

### 3.2 当时值得借鉴的模式

1. **plugin.json 满分 100/100** → `skills/.claude-plugin/plugin.json` 结构完整、描述精准、无 vague 词汇。说明作者对 [manifest](#manifest) 规范非常重视，[frontmatter](#frontmatter) 质量高。如何借鉴：把 plugin.json 当作 API 说明书来写，不写模糊形容词。
2. **`airis-capability-router.md` 路由设计** → 98/100，仅扣 2 分（"briefly" 一词）。路由命令的任务是"理解用户需求 → 选择正确的 MCP skill"，极简单职责。如何借鉴：大型插件应该有一个轻量 router 命令作为入口，而不是让用户记住 20 个命令名。
3. **`status.md` 近满分** → 97/100，结构清晰，声明了 `Bash(curl*)` 允许工具（虽然实际未用到，略扣分）。如何借鉴：status 类命令要声明具体 allowed-tools，而不是"所有工具都可以"。
4. **分层 CLAUDE.md 文档** → 主 `CLAUDE.md` 描述整体架构，各子系统有自己的文档，层次清晰。如何借鉴：大型项目不要把所有文档塞进一个 CLAUDE.md。

### 3.3 当时的缺陷

1. **4 个核心 MCP skill 零示例** → 为什么这么设计会失败：Claude 学习如何使用 skill 主要靠 `## Examples` section——没有示例，Claude 只能靠 description 猜测用法，在边缘场景（如"数据库连接超时时应该用 mcp-debugging 还是 mcp-database？"）很容易选错。自查：我的 SKILL.md 文件有没有缺少示例的？
2. **`troubleshoot.md` 缺 empty $ARGUMENTS 处理** → 当命令被无参调用时，`## Issue Type: $ARGUMENTS` 这一行会渲染成空白，模型不知道该分析什么问题，输出随机。自查：我的命令文件有没有消费 `$ARGUMENTS` 但没有空值保护的？
3. **`install.sh` curl pipe bash 安装** → 任何有权写 GitHub raw 内容的人（包括仓库被劫持的情况）都可以让用户执行任意代码。airis-gateway 的安全问题不在于 NL 文件，而在于安装脚本。自查：我的项目有没有 curl pipe bash 的安装方式？

### 3.4 当时的优化机会

1. **为 4 个 skill 补充示例**：每个 skill 加 2 个 input→output 示例，预计 NLPM 分数从 90 提升到 96+。
2. **`troubleshoot.md` 加 empty input guard**：在 `## Issue Type: $ARGUMENTS` 前加 `if [ -z "$ARGUMENTS" ]; then echo "..."; fi`。
3. **`airis-research-first.md` 步骤编号**：把当前无序列表改为 `1. ... 2. ...`，分数从 78 到 88。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 4 个 MCP skill 无示例 | `grep -c "## Example" skills/skills/mcp-*/SKILL.md` | **持续**：4 个文件全部 0 匹配 | 高优先级改进点 4 个月未修复，作者可能不知道 NLPM 对示例的评分规则 |
| `troubleshoot.md` 无 ARGUMENTS 空值保护 | `grep "ARGUMENTS\|if \[" .claude/commands/troubleshoot.md` | **持续**：第 11 行仍是 `## Issue Type: $ARGUMENTS`，无保护 | 调用无参命令时仍会产生空白渲染 |
| curl pipe bash 安装 | `grep "curl.*bash\|curl.*sh" install.sh quick-install.sh` | **持续**：install.sh 第 6 行、scripts/quick-install.sh 第 6 行均仍有该模式 | CRITICAL 安全问题未解决，BLOCKED 状态不变 |

### 4.2 架构演进

从 2026-04-26 快照到现在，11 个核心 NL 工件的结构**无变化**。主要演进在底层代码（Python API、shell 脚本）。说明 NL 层已稳定，作者的注意力在完善基础设施本身而不是 Claude 接口。

### 4.3 新增的可学习模式

从 CHANGELOG 来看（仓库内有详细版本记录），底层基础设施有持续迭代，但 NL 文件层暂无新增值得记录的设计模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau/skills 的 frontmatter 完整**：所有 7 个 skill 文件都有 `name`、`description`、`allowed-tools`，比 airis-mcp-gateway 的 skill 规范更好。
2. **gstack 的命令文件结构化程度较高**：gstack 命令有编号步骤，不是无序描述。
3. **无 curl pipe bash 安装方式**：我的项目没有使用高风险安装脚本，这个安全边界我从未踩过。

### 5.2 挑战 / 验证

**认知冲突**：我之前认为"只要 NL 文件质量高，底层代码安全问题是另一个领域的事"。这个案例告诉我：NLPM 的 BLOCKED 判定是全仓库的，底层代码的 curl pipe bash 会让整个插件被安全门拦下来，无论 NL 文件有多好。安全审查不是孤立的——NL 文件和底层代码需要一起过安全关。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 文件是否有示例 section
find ~/.claude/skills/ ~/my-projects -name "SKILL.md" | \
  xargs grep -L "## Example" 2>/dev/null
# 命中后怎么办：命中的文件缺示例，至少加 2 个 input→output 示例块
```

```bash
# 检查命令文件的 $ARGUMENTS 空值保护
grep -rn "\$ARGUMENTS" --include="*.md" . | grep -v "if\|guard\|\[\[" | head -20
# 命中后怎么办：在 $ARGUMENTS 第一次被用到之前加空值检测 if [ -z "$ARGUMENTS" ]
```

```bash
# 检查项目里的 curl pipe bash 模式
find . -name "*.sh" -o -name "install.sh" | xargs grep -l "curl.*|.*bash\|curl.*sh" 2>/dev/null
# 命中后怎么办：替换为 curl 下载 + checksum 验证的两步安装，或切换到包管理器
```

### 6.2 灵感 → 实施路径

1. **想法**：为我的 skill 文件补充结构化示例（input → expected output）。
   - **为何可行**：airis-mcp-gateway 的 4 个缺示例 skill 是最高优先级改进点但 4 个月未修复，说明作者不知道示例有这么高的评分权重（-15 分/文件）；我知道了就应该先做。
   - **第一步**：从 bureau 的 `capture/SKILL.md` 开始，加一个"输入：会话摘要文本 → 输出：canon/ 下新建的 Markdown 文件"示例。15 分钟可完成。

2. **想法**：把 `airis-capability-router.md` 的路由模式引入 gstack。
   - **为何可行**：gstack 有 23+ 个 skill，用户需要记住所有名字。一个 `gstack-route.md` 命令可以做"告诉我你想要什么，我帮你选 skill"的 router。
   - **第一步**：参考 `airis-capability-router.md`（98/100），写一个 `route/SKILL.md`，列出所有 gstack skill 的名字和一句话描述，让 Claude 根据用户意图选择调用哪个。30 分钟可写初稿。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 agiletec-inc/airis-mcp-gateway 的核心目的**：统一管理本地 MCP 服务，提供 Claude Code 的 MCP 接入层，附带操作界面。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是本地基础设施+NL 接口；都管理持久化状态 | bureau 管理知识库，airis 管理 MCP 服务器；技术栈不同 | 中（router 模式可借鉴）|
| MarkQWu/gstack | 低 | 都有 NL 表皮+代码核心的混合架构 | gstack 是个人工具套件，非网关/基础设施 | 低 |

若无直接对应项目，我的仓库中无目的完全相近的项目，本节仅做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| MCP skill 缺示例 | `grep -L "## Example" bureau/skills/*/SKILL.md` | bureau 7 个 skill 全部命中（检查过：均未找到 `## Example` section） | 高 |
| $ARGUMENTS 无空值保护 | `grep -rn "\$ARGUMENTS" bureau/commands/*.md` | bureau/commands/lint.md 和 status.md 使用 `argument-hint` 但未检查 | 中 |
| 外部依赖未声明 | `grep -rn "superpowers:\|bureau:\|gstack:" bureau/skills/*.md` | bureau/skills/recall/SKILL.md 有交叉 skill 引用，但未在 plugin.json 声明依赖 | 中 |

**命中后的具体行动建议**：
- bureau 全部 7 个 skill 缺 `## Examples`：最高优先级，从 `recall/SKILL.md`（最常用）开始，加"用户对话片段 → 生成的 recall entry"示例。每个 skill 15–20 分钟，7 个 = 2 小时内可完成。

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：capability router 命令作为插件入口
   - **本案例做法**：`airis-capability-router.md`（98/100）是一个 7 行轻量路由命令，列出所有可用 MCP skill 及其适用场景，让用户用自然语言描述需求，Claude 选择正确 skill
   - **我的项目现状**：bureau 和 gstack 没有统一的 router 入口，用户需要记住每个 skill 的名字
   - **如何借鉴**：在 bureau 的 `commands/` 下加一个 `help.md` 命令，内容是所有 skill 的功能速查 + "告诉我你想做什么，我告诉你用哪个 skill"的提示

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安全安装方式
- **我的做法**：bureau 和 gstack 通过 `claude plugin install` 安装，无 curl pipe bash 安装脚本
- **本案例做法（弱在哪）**：`install.sh` 和 `quick-install.sh` 都有 `curl | bash` 模式（CRITICAL），这使得整个插件无法通过 NLPM 安全门，被标注为 BLOCKED
- **意义**：我的项目从未有过这个反模式，说明在安全安装选择上我的直觉是正确的；这个优势需要维持。

---

## 八、术语表

### <a name="mcp网关"></a>MCP 网关
> MCP = Model Context Protocol，是 Anthropic 制定的一套标准，让外部工具（数据库、浏览器、API 等）能和 Claude 对话。MCP 网关是一个代理服务，它在本机运行，同时连接多个 MCP 服务（如数据库 MCP、调试 MCP），统一暴露给 Claude Code，让 Claude 不需要分别配置每个服务。

### <a name="nl表皮原生代码核心"></a>NL 表皮 + 原生代码核心
> 一种混合架构：用户看到的是 Markdown 写成的 Claude Code 命令（"自然语言表皮"），但实际执行的是 Python/Go/shell 等编写的程序（"原生代码核心"）。NL 层负责"告诉 Claude 怎么操作"，原生层负责"真正执行操作"。好处是两层分工明确，原生层可以访问 NL 层无法直接操作的系统资源（文件系统、网络、进程）。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉 Claude Code 这个插件包含哪些命令、skill、agent。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有组件的路径和元数据。清单里漏写的文件不会被加载，未声明的依赖在运行时会导致报错。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。Claude Code 读 NL 文件时先解析 frontmatter，才能知道这个文件是什么类型、能做什么、需要哪些工具权限。
