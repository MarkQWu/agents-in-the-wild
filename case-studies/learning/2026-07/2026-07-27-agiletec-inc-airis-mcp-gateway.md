# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream audit
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-07-27（基于当前 HEAD）
**主题标签**：`security-gate`, `nl-binary-hybrid`, `manifest-discipline`, `curl-pipe-bash-risk`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

airis-mcp-gateway 是一个 **FastAPI 驱动的 MCP 多路复用器（[MCP 多路复用器](#mcp-multiplexer)）**，把多个 MCP 服务器（进程型 + Docker 网关型）通过两种传输协议对外暴露：
- `http://localhost:9400/mcp/` — Streamable HTTP，供 Claude Code / Codex 使用
- `http://localhost:9400/sse` — SSE，供 Gemini CLI / Cursor / Windsurf 使用

三个关键事实：
1. **动态发现模式（Dynamic MCP Mode）**：暴露 4 个[元工具](#meta-tool)（`airis-find`、`airis-schema`、`airis-workflow`、`airis-exec`）+ 每个可发现 COLD 服务器的懒加载[存根](#stub) schema，而不是直接把 60+ 个工具全部展开。客户端首次调用一个 COLD 工具时，网关自动发现并启用它——**一跳即达，无需元工具中转**。
2. **NL 工件是配置复印件**：`config/bootstrap/assets/claude/` 里的 commands/skills 在安装时由 `airis_bootstrap.py` 写入用户的 `.claude/` 目录——这套 NL 工件本质上是「工件模板仓库」，而不是在本仓库内独立运行的 skill。
3. **SECURITY BLOCKED（审计时）**：三个 Critical 安全发现阻止了 PR 贡献；这些发现在当前 HEAD 仍持续存在。

### 1.2 架构剖析

**目录结构（当前 HEAD）**：
```
airis-mcp-gateway/
├── apps/
│   ├── api/                     # Python 3.12 + FastAPI MCP 多路复用核心
│   │   └── src/app/core/        # behavior_compiler.py — 把 workflows/*.yaml 编译进 initialize 指令
│   ├── gateway-control/         # TypeScript MCP 服务器，暴露网关控制工具
│   └── airis-commands/          # TypeScript MCP 服务器，打包 slash-command 工具
├── config/
│   ├── bootstrap/assets/claude/ # NL 工件模板（commands: airis-research-first.md, airis-capability-router.md）
│   └── profiles/                # minimal.json / recommended.json — 工具集配置
├── scripts/
│   ├── install.sh               # 主安装脚本（含 curl|bash CRITICAL 漏洞）
│   ├── quick-install.sh         # 快速安装脚本（同上）
│   ├── airis-gateway            # CLI 主脚本（含 shell 注入 HIGH 漏洞）
│   └── airis_bootstrap.py       # 配置写入脚本
├── skills/skills/               # 4 个 SKILL.md（mcp-database/debugging/implementation/research）
├── skills/.claude-plugin/       # plugin.json
├── workflows/*.yaml              # 行为配方（compile_to: mcp_instructions 或 airis_workflow）
└── CLAUDE.md                    # 开发者指引（NLPM 98/100）
```

**文件类型分布**：2 个 command / 4 个 skill / 0 个 hook / 大量 shell 脚本和 Python 源码

**编排关系**：  
NL 工件（commands/skills）是**辅助层**，核心是 Python/TypeScript 构成的运行时网关。commands 触发 `task docker:up` 等任务；skills 引导 Claude 使用 MCP 工具（`airis-find`、`airis-schema`）来发现和调用下游服务器。

**跨件契约**：  
`skills/skills/mcp-debugging/SKILL.md` 引用了外部插件 `superpowers:systematic-debugging`，但 `plugin.json` 未声明此依赖——这是一个**未声明的外部依赖（cross-component 发现）**。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「懒加载路由代替静态展开」——COLD 工具的 schema 只在首次调用时解析，把工具上下文消耗从 O(N×schema_size) 压缩到 O(4+1)。
- **解决什么问题**：多个 MCP 服务器启动后给 AI 客户端造成「工具爆炸」（60+ 工具一次性注入）。airis-gateway 把这个问题转化为「按需发现」。
- **做了什么 trade-off**：用一次延迟（首次调用 COLD 工具多一跳）换取大幅压缩的 `tools/list` 响应体——对上下文窗口有限的客户端友好。
- **反映什么认知模型**：作者把 MCP 网关看成「路由器 + 服务注册表」，把 Claude 看成可接受元工具间接层的智能客户端。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：MCP 路由网关 + NL 配置复印模式**

核心特征：
- 特征 1：**元工具抽象层**——用少量元工具替代直接暴露所有下游工具，降低客户端上下文负担
- 特征 2：**配置复印引导**——NL 工件由脚本写入用户目录，而非原地运行；仓库是「模板源」
- 特征 3：**双传输兼容**——同时支持 Streamable HTTP 和 SSE，覆盖不同 AI 客户端
- 特征 4：**行为配方编译**——`workflows/*.yaml` 被 `behavior_compiler.py` 在启动时编译进 `initialize` 指令，而非在 NL 工件里硬编码

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要聚合 5+ 个 MCP 服务器的团队 | ✅ 高度适用 | 工具爆炸问题正是 airis 的核心靶点 |
| 单人开发 1-2 个 MCP 服务器 | ❌ 过度工程 | 直接配置 mcp.json 更简单 |
| 需要跨 Gemini/Cursor/Claude Code 兼容 | ✅ 适用 | 双传输设计专门为此 |
| 高安全要求环境 | ⚠️ 需先修安全漏洞 | curl\|bash CRITICAL 未修复前不适用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 路由网关（本仓库） | airis-mcp-gateway | 懒加载降低上下文、多客户端兼容 | 安装复杂、curl\|bash 安全风险 |
| 直接 mcp.json 配置 | 大多数个人项目 | 零学习成本、无额外进程 | 工具全量注入、无懒加载 |
| 纯 NL 工件插件（无网关） | review-squad、simmer | 安装简单、零运行时依赖 | 无法聚合下游服务器 |

### 2.4 改进空间

1. **当前问题**：NL skills 无任何 example blocks（4 个 skill 全部 -15 分）。**改进做法**：为每个 skill 加一个「用户 prompt → Claude 行为 → 工具调用序列」三段式示例。**预期收益**：NLPM 从 90 升至接近 96，新用户上手时间减半。
2. **当前问题**：`mcp-debugging/SKILL.md` 依赖未声明的 `superpowers` 插件，用户收到静默失败。**改进做法**：在 `plugin.json` 加 `dependencies` 字段，或在 skill 顶部加「需要先安装 superpowers 插件」的前置检查步骤。**预期收益**：消除跨件引用静默失败。
3. **当前问题**：`install.sh` 用 `AIRIS_VERSION:-latest` 默认值，所有新安装都从最新标签拉取，引入供应链风险。**改进做法**：把默认值改为具体版本号，如 `AIRIS_VERSION:-v2.4.1`，并在文档说明如何升级。**预期收益**：把供应链攻击面从「任意主干推送」收窄到「版本边界」。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-26 当时得分 **90/100**（11 个文件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| config/.../airis-research-first.md | 78 | 无 output format + 规则未编号 |
| skills/mcp-database/SKILL.md | 83 | 零 examples + "appropriate caution" |
| skills/mcp-debugging/SKILL.md | 85 | 零 examples |
| skills/mcp-implementation/SKILL.md | 85 | 零 examples |
| skills/mcp-research/SKILL.md | 85 | 零 examples |
| .claude/commands/troubleshoot.md | 90 | 无空 $ARGUMENTS 处理 |
| .claude/commands/test.md | 96 | "some servers"/"actionable" 模糊 |
| .claude/commands/status.md | 97 | 声明了 Bash(curl*) 但 body 未用 |
| config/.../airis-capability-router.md | 98 | "briefly" 模糊 |
| CLAUDE.md | 98 | "significant tokens" 模糊 |
| skills/.claude-plugin/plugin.json | 100 | — |

### 3.2 当时值得借鉴的模式

1. **元工具间接层** → 把 60+ 工具压缩为 4 个元工具 + 懒加载存根；根本原因：上下文消耗控制。应借鉴：若管理多个 MCP 服务器，先设计路由层而非直接展开。
2. **行为配方编译（workflow YAML → initialize 指令）** → NL 行为和安装包解耦，初始化时动态注入；根本原因：配置与实现分离。应借鉴：`compile_to: mcp_instructions` 模式值得用于需要「初始化时注入上下文」的插件。
3. **双传输设计** → 一个服务器同时支持 SSE 和 Streamable HTTP；根本原因：下游 AI 客户端标准不统一。应借鉴：设计 MCP 服务器时声明两个传输端点，未来兼容成本为零。
4. **CLAUDE.md 作为开发者合同** → 98/100 高分 CLAUDE.md 精确描述了架构、命令、测试结构；根本原因：开发者文档和用户文档分开。应借鉴：CLAUDE.md 应面向「接手代码的开发者」而非「使用工具的用户」。

### 3.3 当时的缺陷

1. **curl|bash 供应链攻击向量**：`install.sh` 第 6 行文档化了 `curl -fsSL .../install.sh | bash` 为标准安装方式，第 119 行实际下载并执行了远程 shell 脚本；两处均无哈希校验。**为什么会失败**：攻击者只需劫持 GitHub raw URL（BGP 劫持、CDN 缓存投毒、仓库密钥泄露）即可在用户机器上执行任意代码。自查：我的 install.sh 有无直接 curl|bash？→ 检查 drama-workshop-skills/install.ps1（Windows 脚本）。
2. **shell 注入（airis-gateway L264/L312）**：`server = '$server'` 把 shell 变量直接插入 Python heredoc；含单引号或 `$(...)` 的服务器名可破坏 heredoc 边界并执行任意 Python。**为什么会失败**：`$server` 来自命令行参数，完全受攻击者控制。应改用 `sys.argv[1]` 传参。自查：我的脚本有无 shell 变量插入到代码字符串？
3. **未声明外部依赖**：`mcp-debugging/SKILL.md` 要求用户先 invoke `superpowers:systematic-debugging`，但 plugin.json 无此依赖声明。**为什么会失败**：用户安装 airis 但不知道需要另装 superpowers，调试 skill 静默失效。自查：我的 skill 有无引用外部 plugin 而未在 plugin.json 声明？

### 3.4 当时的优化机会

1. 为 4 个 skills 各加一个 `## Examples` 块（最高 ROI：一次性消除 4×-15 = -60 分惩罚）
2. `airis-research-first.md` 把 unordered bullets 改为编号步骤（-10 → 0）
3. `troubleshoot.md` 加空 $ARGUMENTS guard：`如果没有参数，列出常见错误类型并请用户选择`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| curl\|bash Critical（install.sh L6/L119） | `grep -n 'curl.*bash' install.sh` + 检查 L119 附近 | **持续**：L6 仍为 `curl -fsSL ... \| bash`，L119 仍下载并执行远程脚本 | 安全风险在历经 3 个月后仍未修复；maintainer 可能优先级排在功能之后 |
| shell 注入（airis-gateway L264/L312） | `grep -n "server.*=.*'\$server'" scripts/airis-gateway` | **持续**：L264 和 L312 均发现 `server = '$server'` | HIGH 漏洞持续 |
| 4 skills 零 examples | 读取各 SKILL.md | **持续**：mcp-database/debugging/implementation/research 均无 `## Examples` | 90/100 未变化 |

### 4.2 架构演进

当前 HEAD 相比 Audit 时最显著变化：
- CLAUDE.md 新增了完整的架构说明（`Dynamic MCP Mode`、`COLD_TOOLS_IN_LIST=false` 选项）和详细的测试结构（unit/integration/e2e 层次）
- `routing-table.json` 新增（路由规则外部化）
- `assets/` 新增 demo 录制脚本和 GIF

这说明作者重心在**功能增强和演示材料**，而不是安全修复或 NL 工件质量提升。

### 4.3 新增的可学习模式

- **`COLD_TOOLS_IN_LIST=false` 降级模式**：为上下文受限客户端（旧版 Cursor 等）提供兼容路径，恢复纯元工具列表；这是一个优秀的[降级路径](#fallback-chain)设计。
- **`routing-table.json` 外部化**：路由规则从代码移入配置文件，支持运行时热更新。

---

## 五、校准

### 5.1 我已经在做对的

1. **plugin.json 声明依赖**：我的 gstack/bureau 没有引用外部插件而不声明——airis 的未声明 superpowers 依赖提示了这个反模式的危害。
2. **CLAUDE.md 面向开发者**：我的 bureau/CLAUDE.md 和 gstack 的 CLAUDE.md 都以开发者视角写作，这和 airis CLAUDE.md 98/100 的高分实践一致。
3. **不在安装脚本用 curl|bash**：我的 drama-workshop-skills 用的是 PowerShell install.ps1，没有 curl|bash 模式——这个避免本身是一个正确决策。

### 5.2 挑战 / 验证

这个案例挑战了「BLOCKED 安全的项目一定是小项目随意写的」这一假设——airis 有 151 颗星、清晰的架构、高质量的 CLAUDE.md，但仍然有 Critical 安全发现。**安全问题不是粗心的标志，而是安装脚本设计的结构性难题**：curl|bash 是业界常见模式（Homebrew、nvm 都用过），只是安全边界需要更明确的选择（哈希校验 vs 便利性）。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的安装脚本有无 curl|bash 或下载后立即执行的模式
grep -rn "curl.*|.*bash\|wget.*sh\|bash.*<(curl" ~/.claude/ ~/agents-in-the-wild/ 2>/dev/null | grep -v ".git"
```
命中后怎么办：改为先下载到临时文件、显示 SHA256、用户确认后再执行。

```bash
# 检查我的 skill/command 有无引用外部插件但未在 plugin.json 声明
grep -rn ":\|plugin:" ~/.claude/skills/*.md 2>/dev/null | grep -v "^---"
```
命中后怎么办：在对应的 plugin.json 加 `dependencies` 字段，或把 skill 改为可选引用。

### 6.2 灵感 → 实施路径

1. **想法**：在我的 gstack 中实现「元工具间接层」模式——多个 gstack skills 其实是不同工具的调度器，可以设计一个 `gstack:route` skill 来降低新用户的认知负担。
   - **为何可行**：gstack 有 20+ skills，新用户不知道从哪里开始——元工具级别的引导可以大幅降低摩擦。
   - **第一步**：在 `gstack/CLAUDE.md` 加一个「快速入口」表，按任务类型映射到具体 skill；10 分钟可完成。

2. **想法**：将 bureau 的工作流配方外部化为 YAML（参考 `workflows/*.yaml` + behavior_compiler 模式）。
   - **为何可行**：bureau 当前行为是硬编码在每个 skill 里的，修改一个行为需要改多个文件。
   - **第一步**：识别 bureau 中重复出现的行为规则（如「每次 capture 后触发 compile check」），提取到 `bureau/workflows/` 目录；30 分钟可规划，2 小时实施。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 agiletec-inc/airis-mcp-gateway 的核心目的**：聚合多个 MCP 服务器，通过懒加载路由暴露给多种 AI 客户端，降低上下文负担。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 中 | 都是 AI 编码助手技能，都面向知识图谱管理 | graphify 是单个 skill，airis 是多服务器网关 | 中 |
| MarkQWu/bureau | 低 | 都有多 skill 编排 | bureau 是知识库管理，airis 是 MCP 路由 | 低 |
| MarkQWu/gstack | 低 | 都有多工具协调 | gstack 是开发工作流，airis 是基础设施网关 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skills 零 examples（4个 skill 各 -15） | `grep -L '## Examples\|## 示例' /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | gstack 所有 SKILL.md 均无 `## Examples` 节（20+ 文件） | 高 |
| 声明了工具但 body 未用（status.md Bash(curl*)） | `grep -n "allowed-tools\|Bash" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md \| grep -B5 "Bash"` | 需要手动检查，模式有可能复现 | 中 |

**命中后的具体行动**：
- `gstack/ios-design-review/SKILL.md` → 加 `## Examples` 节，3 行示例（用户说「review the iOS design」→ Claude 行为描述）；10 分钟可完成。

### 8.3 别人的更优方案

1. **领域**：CLAUDE.md 开发者视角 vs 用户视角分离
   - **本案例做法**：`CLAUDE.md` 98/100，完整描述架构、命令、测试结构，是面向接手代码的开发者的合同文件
   - **我的项目现状**：`bureau/CLAUDE.md` 较好，但 `gstack/CLAUDE.md` 缺乏测试结构说明和具体命令列表
   - **如何借鉴**：在 `gstack/CLAUDE.md` 加 `## 测试` 和 `## 常用命令` 两个节；参考 airis CLAUDE.md 的 `task docker:up / test:e2e` 格式。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：plugin.json 依赖声明
- **我的做法**：gstack 和 bureau 的 plugin.json 没有引用外部未声明的插件
- **本案例做法弱在哪**：`mcp-debugging/SKILL.md` 引用 `superpowers:systematic-debugging` 但 plugin.json 无声明，导致跨件引用静默失败
- **意义**：若未来审计我的仓库，这一点是潜在亮点；继续保持「引用即声明」原则。

---

## 八、术语表

### <a name="mcp-multiplexer"></a>MCP 多路复用器
> Model Context Protocol（MCP）是 Anthropic 定义的一个协议，让 AI 应用通过标准接口调用外部工具。"多路复用器"是指把多个后端服务器（每个都实现 MCP 协议）的工具统一代理到一个端口，AI 客户端只需连接这一个端口就能访问所有工具。类比：路由器把多台电脑接入同一根网线。

### <a name="meta-tool"></a>元工具
> 不直接完成业务功能、而是用来「发现/查询/路由」其他工具的工具。airis 的 `airis-find`、`airis-schema`、`airis-workflow`、`airis-exec` 就是元工具——AI 先问「有没有能处理数据库的工具」（调用 `airis-find`），再得到答案后去调用具体工具。这样把工具发现延迟到运行时，节省了 context window。

### <a name="stub"></a>存根（Stub）
> 一个极简的、只有占位 schema 的工具描述（`{"type":"object"}`），用来告诉 AI 客户端「这个工具存在，你可以调用它」，但不暴露完整参数定义。第一次实际调用时，网关再去取完整 schema。节省了 `tools/list` 响应的体积。

### <a name="fallback-chain"></a>降级路径
> 当主要功能不可用时，系统退回到功能有限但仍可工作的替代方案的能力。airis 的 `COLD_TOOLS_IN_LIST=false` 就是一个降级路径——关掉懒加载工具列表，回到纯元工具模式，兼容上下文受限的旧客户端。

### <a name="curl-pipe-bash-risk"></a>curl|bash 风险
> `curl https://example.com/install.sh | bash` 模式从网络下载脚本后立即以 shell 权限执行，不给用户任何检查或取消的机会。风险在于：DNS 被劫持、CDN 被投毒、或仓库密钥泄露时，用户会在不知情的情况下执行攻击者的代码。安全替代方案：先下载到文件，显示 SHA256 哈希，用户确认后再执行。
