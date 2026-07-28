# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-07-28（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `manifest-discipline`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`agiletec-inc/airis-mcp-gateway` 是一个**MCP（Model Context Protocol）网关服务**，用一条命令在本地启动一个 Docker 化的 AI 基础设施层，使 Claude Code 可以通过标准化的 MCP 接口统一访问数据库、网络、文件系统等工具。简单说：_"给 Claude Code 接一个智能路由器，所有 MCP 工具调用都走这条路"_。

关键事实：
- 作者：agiletec-inc 团队，主要面向需要在本地或企业内网部署 AI 工具基础设施的开发者
- 安装方式：一行 `curl | bash` 命令（这也是审计的核心安全问题所在）
- 技术栈：Docker Compose + TypeScript (apps/) + Python (airis_bootstrap.py) + Shell 安装脚本
- 特色能力：`airis-capability-router.md` 命令可以根据任务类型自动选择最合适的 MCP 工具
- 版本状态：使用 `latest` 标签，从不固定版本号

### 1.2 架构剖析

```
airis-mcp-gateway/
├── install.sh                    # 主安装脚本（含 curl|bash 模式）
├── scripts/
│   ├── quick-install.sh          # 快速安装（同样含 curl|bash）
│   ├── airis-gateway             # 核心管理脚本（bash，有 shell 注入漏洞）
│   └── airis_bootstrap.py        # Python 引导脚本（远程下载执行）
├── apps/
│   ├── api/                      # Python FastAPI 应用（~55 个源文件）
│   └── gateway-control/          # TypeScript 网关控制面板
├── config/bootstrap/assets/claude/
│   └── commands/
│       ├── airis-capability-router.md  # 能力路由命令（score 98）
│       └── airis-research-first.md     # 研究优先命令（score 78，最低分）
├── .claude/commands/
│   ├── status.md (97)  test.md (96)  troubleshoot.md (90)
│   └── ...
├── skills/
│   └── skills/
│       ├── mcp-database/SKILL.md (83)
│       ├── mcp-debugging/SKILL.md (85)
│       ├── mcp-implementation/SKILL.md (85)
│       └── mcp-research/SKILL.md (85)
├── skills/.claude-plugin/plugin.json (100)
├── CLAUDE.md (98)
├── manifest.toml                 # 服务清单
└── compose.yaml                  # Docker Compose 配置
```

- **文件类型分布**：4 个 skill / 5 个 command（2 个在 config/bootstrap，3 个在 .claude） / 1 个 plugin manifest
- **编排关系**：`airis-capability-router.md` 是核心路由入口，读取 `routing-table.json` 决定调度哪个 MCP 工具；其他命令（test/status/troubleshoot）是运维辅助
- **跨件契约**：`mcp-debugging/SKILL.md` 声明了对外部 `superpowers` 插件的依赖（调用 `superpowers:systematic-debugging`），但 plugin.json 里没有声明这个依赖 → **隐式外部依赖，用户不知道需要先装 superpowers**

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「基础设施即服务，Claude 是消费方」——把 MCP 工具封装成一个统一的网关，Claude Code 只需关心"我要做什么"，不需要知道底层工具是数据库驱动还是 REST API
- **解决什么问题**：在多工具 MCP 环境里，Claude 需要频繁在工具间切换、发现工具、处理工具不可用的情况。Airis 用能力路由解决了"Claude 不知道用哪个工具"的问题
- **做了什么 trade-off**：用 Docker 化部署换来了隔离性和可重复性，代价是对 Docker 有硬依赖，无法在某些受限环境中运行
- **反映什么认知模型**：作者把 MCP 工具层看作"需要被抽象的底层细节"，Claude 应该通过路由层而非直接接触工具接口来工作——这和 API Gateway 模式在 Web 开发中的角色类似

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「能力路由网关」（Capability Router + MCP Gateway）**

关键特征：
- **特征 1**：所有 MCP 工具访问经过统一网关，Claude 不直接操作工具实现
- **特征 2**：`routing-table.json` 作为配置驱动的路由规则，不需要修改代码即可调整工具选择逻辑
- **特征 3**：运维命令（status/test/troubleshoot）与业务命令（capability-router/research-first）分离
- **特征 4**：Docker Compose 保证环境隔离，install.sh 负责一键部署
- **特征 5**：4 个领域专用 skill（database/debugging/implementation/research）对应 4 种工作模式

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要同时管理多个 MCP 工具的企业内网部署 | ✅ 高度适用 | 网关集中管控，安全策略统一 |
| 单开发者本地环境，仅用 1-2 个 MCP 工具 | ❌ 过度工程 | Docker 化网关的引入成本超过收益 |
| 需要审计 AI 工具调用记录的合规场景 | ✅ 适用 | 网关层天然是日志/审计的埋点位置 |
| 对 Docker 有使用限制的受控环境（如某些 CI） | ❌ 不适用 | 强依赖 Docker，无降级方案 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 网关路由（本仓库） | airis-mcp-gateway | 统一管控、可扩展、Docker 隔离 | 部署复杂、引入延迟 |
| 直接 MCP 配置（每工具单独声明） | aaronmaturen/claude-plugin（Context7 MCP） | 简单直接、零额外服务 | 工具多时难管理，无路由能力 |
| 纯 NL skill 路由（不依赖 MCP） | agent-sh/enhance | 轻量、无网络依赖 | 无法调用系统级工具 |

### 2.4 改进空间

1. **当前问题**：4 个 skill 全部缺少 `## Examples` 块。**改进做法**：每个 skill 添加一组"用户询问 → 工具选择 → 执行结果"的端到端示例。**预期收益**：skill 分数从 83-85 → 98+，Claude 对工具使用的可预测性大幅提升

2. **当前问题**：`mcp-debugging/SKILL.md` 硬依赖 `superpowers` 插件，但 plugin.json 无声明。**改进做法**：在 plugin.json 的 `dependencies` 字段声明 `superpowers`，或在 skill 里加 graceful degradation（如果 superpowers 不存在则走备选流程）。**预期收益**：消除隐式依赖陷阱，用户安装后不会遇到莫名失败

3. **当前问题**：install.sh 的 `VERSION="${AIRIS_VERSION:-latest}"` 默认值是 latest，每次安装都可能装到不同版本。**改进做法**：把默认值改为具体版本号，保留 env var override。**预期收益**：安装行为可重复，便于排查问题

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-26 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| airis-research-first.md | 78 | 无 output format，多步骤未编号 |
| skills/mcp-database/SKILL.md | 83 | 无 examples，"appropriate caution" 模糊 |
| skills/mcp-debugging/SKILL.md | 85 | 无 examples |
| skills/mcp-implementation/SKILL.md | 85 | 无 examples |
| skills/mcp-research/SKILL.md | 85 | 无 examples |
| .claude/commands/troubleshoot.md | 90 | 无空 $ARGUMENTS 保护 |
| plugin.json | 100 | 满分 |

### 3.2 当时值得借鉴的模式

1. **plugin.json 满分（100/100）**：所有字段完整，版本固定，命令路径明确，keywords 准确 → 做到了最基础的"登记正确"，大部分仓库都没做到。借鉴：每次新建插件，先把 plugin.json 写满再开始写命令

2. **能力路由命令（airis-capability-router.md，98/100）**：用 `routing-table.json` 做配置驱动路由，命令体清晰，输出格式有结构 → 把"用什么工具"的决策从硬编码提升为可配置。借鉴：复杂工具集成可以引入路由配置文件，而非在命令体里写死条件判断

3. **CLAUDE.md 作为契约（98/100）**：包含了上下文预加载、任务声明格式、工具命名约定 → 把人机协作规范写入项目文档。借鉴：CLAUDE.md 不只是说明书，是 Claude 行为的"宪法"

4. **运维命令分离（status/test/troubleshoot）**：业务命令和运维命令分在不同目录（config/bootstrap vs .claude） → 清晰分层，用户按场景找命令。借鉴：大型插件可以把"给用户用的"和"给运维用的"命令分开放

5. **Docker 化隔离**：所有服务运行在容器里，Claude Code 通过 MCP 协议访问 → 宿主环境干净，服务版本可控。借鉴：需要运行长期服务的插件考虑容器化，而不是在宿主机直接安装

### 3.3 当时的缺陷

1. **三个 Critical 安全漏洞（curl|bash 模式）**
   - `install.sh` 的使用文档直接写 `curl -fsSL .../install.sh | bash`，没有任何完整性检查
   - `install.sh` line 119：下载远程 shell 脚本后立刻 `chmod +x` 并执行，等同于 `curl | bash`
   - `quick-install.sh` line 6：同样的模式
   - **根本原因**：作者追求安装体验的极简（一行命令），忽视了"用户信任了什么"的问题。这是业界的惯常做法但已被广泛批评
   - **自查**：我的项目有没有让用户直接 curl|bash 安装的地方？

2. **shell 注入漏洞（airis-gateway line 262）**
   - `server = '$server'` 把 shell 变量直接内插到 Python heredoc 中，包含 `'` 或 `$(...)` 的服务器名会破坏 Python 代码块
   - **根本原因**：在 bash 中生成动态代码时没有对输入进行转义，这是经典的二次求值漏洞
   - **自查**：我的脚本里有没有把 $ARGUMENTS 直接拼入命令字符串的地方？

3. **所有 4 个 skill 无 examples 块**
   - **根本原因**：作者可能认为 skill 的"使用说明"已经足够，没有意识到 examples 是 Claude 学习"如何把知识变成动作"的关键
   - **自查**：我的 skill 有没有 ## Examples 章节？

### 3.4 当时的优化机会

1. **curl|bash → 带 SHA-256 验证的安全安装脚本**：参考 `agent-sh/enhance` 的运行时二进制下载验证模式，在 install.sh 里加 checksum 验证步骤
2. **batch 为 4 个 skill 添加 examples 块**：每个 skill 写 1-2 个"用户请求 → 工具调用流程 → 预期输出"示例
3. **airis-research-first.md 添加步骤编号和 output format section**

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（在克隆目录里运行） | 现状 | 含义 |
|---|---|---|---|
| install.sh curl\|bash | `grep -n "curl.*bash\|curl.*\| bash" install.sh` | **仍存在** ❌ — line 6 仍有 `curl -fsSL .../install.sh \| bash` 的文档指引 | 3 个月后安全漏洞未修，用户仍面临供应链风险 |
| VERSION=latest（不固定版本） | `grep "AIRIS_VERSION:-latest" install.sh scripts/airis-gateway` | **仍存在** ❌ — 两处都还是 `latest` | 安装可重复性问题持续 |
| skill 无 examples | `grep -rL "## Example" skills/skills/*/SKILL.md` | **仍存在** ❌ — 4 个 skill 全无 examples | 系统性问题，需要批量处理 |

### 4.2 架构演进

与 2026-04-26 审计时相比，当前 HEAD 结构基本一致，但新增了几个运维相关文件：
- 新增了 `routing-table.json`（显式路由配置，审计时只在命令体里提及）
- 新增了 `manifest.toml`（服务清单，明确声明各个微服务的依赖关系）
- `CLAUDE.md` 内容更丰富，加入了上下文压缩策略（"Saves significant tokens per `initialize`" — 审计时就已有，但现在更突出）

作者意识到：需要一个明确的路由配置文件来替代命令体里的硬编码逻辑，`routing-table.json` 的引入体现了从"把配置写在 NL 命令里"到"把配置提取为结构化数据"的演进。

### 4.3 新增的可学习模式

- **`manifest.toml` 服务声明**：明确列出所有微服务（api、gateway-control、airis-commands）的版本、端口、依赖关系。这比 README 里散文描述要更机器可读，也是运维自动化的基础。值得借鉴：多服务组合的插件可以引入 manifest 文件（TOML/JSON 都行）作为部署文档

---

## 五、校准

### 5.1 我已经在做对的

1. **我的项目没有 curl|bash 安装模式**：bureau/gstack 都是通过 Claude Code plugin install 安装的，不涉及外部脚本下载执行
2. **我的 plugin.json 字段基本完整**：参考了 NLPM 规范，关键字段都有填写
3. **CLAUDE.md 文档化约定**：我的 bureau 和 gstack 的 CLAUDE.md 都有明确的项目约定，和 airis 的高分 CLAUDE.md 思路一致
4. **运维和业务命令没有混在一起**：bureau 的命令都是业务相关的，没有专门的 status/test 运维命令 — 但也意味着缺少了这类功能

### 5.2 挑战 / 验证

- **挑战了什么假设？** 我之前认为"NL 工件质量高等于仓库整体质量高"。这个案例打破了这个假设：airis 的 NL 分数 90/100（很高），但安全分是 BLOCKED（最低），说明 NL 质量和安全质量是两个完全独立的维度，高 NL 分不能作为整体质量的保证
- **被验证的做法**：「plugin.json 优先满分」被 airis 验证：manifest 满分是整体仓库质量的基础，可以独立于其他工件先做好

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的项目有没有 curl|bash 安装指引
grep -rn "curl.*| bash\|curl.*| sh" . --include="*.md" --include="*.sh" 2>/dev/null
# 命中后：替换为带 SHA-256 验证的安全下载流程，或改为 npm/pip install

# 检查我的 skills 是否有 examples 块
grep -rL "## Example" */skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：为每个缺少示例的 skill 补 ## Examples 章节（至少 1 个 input→output 对）

# 检查我的脚本有没有把变量直接内插进动态代码的写法
grep -rn '"\$' scripts/*.sh hooks/*.sh 2>/dev/null | grep -v "^Binary"
# 命中后：用参数传递（positional args）替代字符串插值

# 检查 troubleshoot 类命令有没有空 $ARGUMENTS 保护
grep -rn "ARGUMENTS" commands/*.md | grep -v "if.*ARGUMENTS\|-z.*ARGUMENTS"
# 命中后：在 $ARGUMENTS 首次使用前加 `[ -z "$ARGUMENTS" ] && echo "Usage: ..." && exit 1`
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 `commands/status.md` 补一个运维检查功能，类似 airis 的 status 命令
   - **为何可行**：bureau 缺少健康检查命令，用户不知道 canon 目前处于什么状态
   - **第一步**：打开 `bureau/commands/status.md`，参考 airis 的 status.md 结构，加入 workspace 健康信息输出。耗时 20 分钟

2. **想法**：为 bureau 的 4 个 skill 补 examples 块，对标 airis 的改进机会
   - **为何可行**：recall/scribe/compile/capture 每个都有明确的 input→output 场景
   - **第一步**：先为使用频率最高的 `skills/recall/SKILL.md` 补一个 query→answer 示例。耗时 15 分钟

3. **想法**：在 bureau 的 plugin.json 里声明依赖关系（如果存在跨插件调用）
   - **为何可行**：airis 的 mcp-debugging skill 因为未声明 superpowers 依赖而被扣分，我需要避免同样问题
   - **第一步**：检查 bureau 的任意 skill 里是否调用了其他插件的 command/skill，如有则在 plugin.json 里加 dependencies 声明。耗时 15 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 agiletec-inc/airis-mcp-gateway 的核心目的**：统一管理和路由 MCP 工具调用，为 Claude Code 提供稳定的工具基础设施层
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有运维视角（bureau 有 status 命令） | bureau 是知识管理；airis 是工具路由基础设施 | 中 |
| MarkQWu/- | 低 | 都有多服务架构（- 有多个 Dockerfile） | - 是情报仪表盘，不是 Claude Code 插件 | 低 |

若全部「无」——但 bureau 有中度相似性，以 bureau 为主要对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 无 examples 块 | `grep -rL "## Example" bureau/skills/*/SKILL.md` | 需检查，预计有命中 | 中 |
| 空 $ARGUMENTS 无保护 | `grep "ARGUMENTS" bureau/commands/*.md \| grep -v "if.*ARGUMENTS"` | bureau/commands/query.md 和 inspect.md 可能命中 | 中 |
| 外部插件依赖未声明 | `grep "superpowers:\|:\w*:" bureau/skills/*/SKILL.md` | 暂无跨插件调用，无风险 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的 `skills/recall/SKILL.md` → 补 `## Examples` 章节，格式：`**Query**: "..."` → `**Answer**: "..."` → 10 分钟可完成
- `MarkQWu/bureau` 的 `commands/query.md` → 在 `$ARGUMENTS` 首次使用前加空值保护 → 5 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：「routing-table.json 配置驱动路由」
   - **本案例做法**：`routing-table.json` 把工具路由规则提取为独立配置文件，命令体只引用配置，不硬编码逻辑
   - **我的项目现状**：bureau 的 commands 里的路由逻辑（如"先找 canon，找不到再走 web"）是写在 Markdown 文本里的，不可直接配置
   - **如何借鉴**：为 bureau 引入 `bureau.json` 配置文件（已有），在其中加入优先级路由规则，命令体读取配置而非硬编码

2. **领域**：「manifest.toml 多服务声明」
   - **本案例做法**：`manifest.toml` 明确列出所有服务依赖，机器可读
   - **我的项目现状**：bureau/gstack 的服务依赖散落在 README 和 CLAUDE.md 里
   - **如何借鉴**：为 bureau 加一个 `bureau-manifest.toml` 或在现有 `bureau.json` 里加 services 字段声明外部依赖

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安装安全性
- **我的做法**：bureau/gstack 通过 `claude plugin install` 安装，不涉及任何外部脚本下载执行，天然规避了 curl|bash 风险
- **本案例做法（弱在哪）**：airis 的 `install.sh` 里明确写了 `curl -fsSL .../install.sh | bash`，Critical 级别安全风险持续 3 个月未修
- **意义**：我的插件走 Claude Code 官方插件系统分发，这是一个很大的安全优势；如果未来设计需要安装脚本的工具，应优先走包管理器（npm/pip）而非 curl|bash

---

## 八、术语表

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 提出的开放协议，定义了 AI 模型和外部工具（数据库、文件系统、API）之间的标准通信接口。有了 MCP，Claude 不需要知道工具的底层实现，只需要通过标准化的调用格式（工具名 + 参数）就可以使用任意 MCP 工具。可以类比为"AI 版的 USB 接口"。

### <a name="curl-pipe-bash"></a>curl|bash
> 一种安装方式的简写：`curl -fsSL <URL> | bash`，意思是"从网络下载一个脚本并直接执行"。方便但危险：用户无法在执行前检查脚本内容，如果下载链接被劫持或 CDN 被污染，用户的机器会直接运行攻击者的代码。安全的替代方案是先下载、验证 SHA-256 校验值，再执行。

### <a name="SLSA"></a>SLSA（Supply-chain Levels for Software Artifacts）
> 一种软件供应链安全框架，通过证明"这个文件是由哪条 CI 流水线在什么条件下构建的"来防止供应链攻击。即使攻击者拿到了 GitHub 的 release token，如果没有通过 SLSA 验证，分发的二进制文件也会被下载方拒绝。

### <a name="shell-injection"></a>shell 注入
> 一种安全漏洞：当脚本把用户输入的字符串直接拼入 shell 命令时，恶意用户可以通过构造特殊字符（如 `$(command)` 或 `;rm -rf /`）让脚本执行意外的命令。正确的做法是用参数（positional args）传递输入，而不是字符串拼接。

### <a name="routing-table"></a>routing-table.json
> 配置驱动的路由规则文件。在 airis 的上下文里，它定义了"当用户的任务类型是 X 时，优先选择 MCP 工具 Y"的映射关系。相比把这个逻辑写死在 Markdown 命令体里，routing-table.json 让路由规则可以独立修改和版本控制。
