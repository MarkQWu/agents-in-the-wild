# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream audit
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-07-29（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `examples-driven`, `nl-binary-hybrid`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
AIRIS MCP Gateway 是一个**按需启动的本地 [MCP](#mcp) 服务端聚合层**（写在 CLAUDE.md 里的话："必要時だけ起動するlocal MCP endpoint"），让 Claude Code 在单次会话里动态连接多个 MCP 服务，而不需要全局常驻。架构上它是一个 Python FastAPI + TypeScript 控制层，用 Docker 管理 MCP 服务的生命周期。

关键事实：
- 作者：agiletec-inc（越南 SaaS 开发公司），CLAUDE.md 用日文撰写，针对 Context7 服务器场景
- 用户安装路径：官方推荐 `curl -fsSL .../install.sh | bash`（这正是本案例的核心缺陷）
- 生态位置：属于"MCP 基础设施"层，介于 Claude Code 和具体 MCP 服务之间的网关
- 发布周期：版本由 `VERSION` 文件控制，通过 `task version:sync` 同步到 [manifest](#manifest)

### 1.2 架构剖析
```
airis-mcp-gateway/
├── install.sh               ← 用户入口（curl|bash 风险点）
├── scripts/
│   ├── quick-install.sh     ← 第二个 curl|bash 风险点
│   ├── airis-mcp-gateway    ← 重命名后的网关控制脚本（含 shell 注入风险）
│   └── airis_bootstrap.py  ← 被远程拉取执行的 Python 脚本
├── apps/
│   ├── api/                 ← Python FastAPI（~55 个源文件）
│   └── gateway-control/     ← TypeScript 控制层
├── config/
│   └── bootstrap/assets/claude/commands/  ← 2 个 Claude Code 命令
├── skills/skills/           ← 4 个 SKILL.md（mcp-database/debugging/implementation/research）
└── CLAUDE.md                ← 日文，定义架构边界
```

- **文件类型分布**：2 个 command / 4 个 SKILL.md / 0 个 agent / 2 个 hook 替代路径（通过 install.sh 注入配置）
- **编排关系**：NL 表皮（commands + skills）包裹原生 Python/TS 核心。用户调用 command → skill 提供领域知识 → airis-mcp-gateway 脚本控制 Docker 容器
- **跨件契约**：`config/mcp-config.template.json` 是运行时注册表的模板；`routing-table.json` 描述 MCP 服务路由；CLAUDE.md 明确禁止手动修改 generated 文件

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「最小常驻原则」——MCP 服务按需启动，不做全局常驻。CLAUDE.md 明确写道"global MCP serverとして常設登録しない"
- **解决什么问题**：多个 MCP 服务同时运行会占用系统资源、端口冲突、且难以调试；网关统一管理生命周期
- **Trade-off**：选择了「安装简单（curl|bash）」换取了「供应链安全」——这是本案例最值得研究的决策失误
- **认知模型**：把 Claude Code 当成一个工具调用层，MCP 服务是「工具插件」，网关是「插件管理器」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 表皮 + 运行时网关核心」**：少量 NL artifacts（2 命令 + 4 技能）作为用户接口，真正的工作由 Python/TS 原生程序完成，NL 层只负责传递意图和解释上下文。

模式特征（4 条）：
- **特征 1**：NL artifacts 数量极少（≤10），但原生代码体量大（~60 源文件）
- **特征 2**：有明确的"禁止区"——CLAUDE.md 定义了 AI 不应触碰的边界（不自动启动服务、不修改 mcp-config.json）
- **特征 3**：安装脚本是用户的第一个接触点，也是最高风险点
- **特征 4**：[routing-table.json](#routing-table) 作为机器可读的服务注册表，与 NL 配置分离

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要管理多个外部进程/服务的工具 | ✅ 高度适用 | 网关模式天然适合生命周期管理 |
| 纯 Claude Code 插件/skill 集合 | ❌ 过度工程 | 没有进程管理需求时，NL 层直接够用 |
| 需要跨平台分发的开发工具 | ✅ 适用 | 但安装脚本的安全性要提升 |
| 团队内部工具（不公开分发） | ✅ 适用 | curl|bash 风险在内部网络可接受 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 运行时网关（本仓库） | airis-mcp-gateway | 进程管理能力强；NL 层薄，好维护 | 安装链路复杂；安全风险高 |
| 纯 NL 平铺 | dontbesilent2025/dbskill | 零运行时依赖；安装即用 | 无进程管理能力；无法动态调度 |
| NL 表皮 + 原生二进制 | agent-sh/agnix | 性能最优；分发简洁 | 跨平台编译复杂；调试困难 |

### 2.4 改进空间
1. **当前问题**：`install.sh` 用 curl|bash 安装，无完整性校验 **改进做法**：发布每个版本时生成 `.sha256` sidecar，安装脚本先 `curl` 下载 + 校验，再执行 **预期收益**：彻底消除供应链投毒风险，同时 README 里的"一行安装命令"体验无损
2. **当前问题**：4 个 SKILL.md 全部没有 `## Examples` section **改进做法**：每个 skill 添加 2 个端到端示例（一个"数据库查询失败→排查步骤"、一个"MCP 连接断开→恢复流程"） **预期收益**：skill 的 NLPM 得分从 83-85 提升至 95+
3. **当前问题**：`scripts/airis-mcp-gateway` 里存在 shell 变量注入到 Python heredoc **改进做法**：`python3 -c '...' "$server"` 并从 `sys.argv[1]` 读取，不做字符串插值 **预期收益**：消除 Medium 级安全缺陷，防止恶意 server 名称的 RCE 尝试

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-26 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| airis-research-first.md | 78 | 无 output format + 步骤无编号 |
| mcp-database/SKILL.md | 83 | 无示例 + "appropriate caution" 模糊量词 |
| mcp-debugging/SKILL.md | 85 | 无示例 |
| mcp-implementation/SKILL.md | 85 | 无示例 |
| mcp-research/SKILL.md | 85 | 无示例 |
| troubleshoot.md | 90 | $ARGUMENTS 空值无处理 |
| test.md / status.md 等 | 96-100 | 局部模糊量词 |

### 3.2 当时值得借鉴的模式
1. **`allowed-tools` 精准声明** → 所有命令都声明了 `Bash(curl*)` 等工具，没有"万能 Bash" → 示例：`status.md` 只声明了 curl，限制了攻击面 → 借鉴：每个命令写 allowed-tools 时逐条验证 body 里实际调用了哪些工具
2. **CLAUDE.md「禁止区」模式** → CLAUDE.md 不只写"能做什么"，还专门写"绝对不要做什么"（不自动启动服务、不修改 mcp-config.json） → 根本原因：禁止区减少了 Claude 在模糊情境下的错误推断 → 借鉴：在我的 CLAUDE.md 里增加"边界约束"小节
3. **routing-table.json 机器可读** → 服务注册表与 NL 配置分离，机器可读、可校验 → 根本原因：JSON 格式的约定比散文注释更难被误解

### 3.3 当时的缺陷
1. **所有 4 个 SKILL.md 零示例** → 根本原因：作者把 skill 当参考文档写，而不是当交互式指令写；没有"展示一次正确用法"的习惯 → 自查：MarkQWu/- 的 10 个 SKILL.md 全部缺失 `## Examples`（已验证）
2. **`curl | bash` 作为官方安装命令** → 根本原因：优先了用户体验（一行命令），忽视了供应链安全；作者可能未考虑过该仓库被攻击时的攻击面 → 自查：我的工具中如有分发脚本，是否也是 curl|bash？
3. **`$ARGUMENTS` 空值无处理（troubleshoot.md）** → `Issue Type: $ARGUMENTS` 在空调用时渲染为空白，命令没有 fallback → 根本原因：开发者以"总是传参"的心态写命令，忽视了无参调用场景 → 自查：我的命令里有没有类似的裸 `$ARGUMENTS`？

### 3.4 当时的优化机会
1. `airis-research-first.md` 将步骤改为有编号列表（-10 分项）
2. 4 个 SKILL.md 各添加 2 个 input→output 示例
3. `install.sh` 和 `scripts/airis-mcp-gateway` 中的 curl|bash 和 shell 注入修复（仅安全学习，不用于 PR）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `install.sh` 中 `curl \| bash` | `grep -n "curl.*bash" install.sh` | **持续**（第 6 行和第 266 行均存在） | 安全风险未修复 |
| 4 个 SKILL.md 无示例 | `grep -rL "## Example" skills/` | **持续**（4 个文件均无 Examples） | 质量问题未修复 |
| `scripts/airis-gateway` shell 注入 | 查找该文件 | **文件已重命名**为 `scripts/airis-mcp-gateway`，但注入模式是否保留需重审 | 文件改名，风险可能保留 |

### 4.2 架构演进
- `scripts/airis-gateway` → `scripts/airis-mcp-gateway`：脚本改名与项目名对齐，无功能变化
- 新增：`scripts/test-compose-references.sh`、`scripts/test-workflow-runners.sh`，测试基础设施有所完善
- CLAUDE.md 从通用描述演进为精确的「禁止区」文档（全日文撰写），说明作者意识到了"边界不清"会导致 Claude 误操作
- `routing-table.json` 新增：表明服务注册机制从隐式变为显式

### 4.3 新增的可学习模式
- **`.github/instructions/` 目录**：出现了 GitHub Copilot instructions 格式的辅助文档，说明项目开始双轨支持 Claude Code 和 GitHub Copilot
- **`config/profiles/`**：minimal.json + recommended.json 双配置 profile，允许用户按需选择 MCP 服务集合——这是「配置分层」模式的好示例

---

## 五、校准

### 5.1 我已经在做对的
1. 我的命令文件都在 `.claude/commands/` 目录下，且有 frontmatter 声明——这一点比 airis 的命令结构更标准
2. 我的 CLAUDE.md 有具体约束描述（bureau 项目）——方向正确，可进一步加「禁止区」
3. 我没有 curl|bash 风险——我的项目没有公开的安装脚本

### 5.2 挑战 / 验证
- **挑战**：我原以为"curl|bash 是小问题，用的人自己懂风险"。这个案例让我意识到：当你在 README 里把 `curl | bash` 写成**官方推荐安装方式**时，你就在给不了解风险的用户背书。作为工具维护者，这个责任不能推给用户。
- **验证**：CLAUDE.md 里写"禁止区"而不只写"允许区"的做法，在这个仓库里得到了验证——从 CLAUDE.md 的日文内容来看，这些约束在实际使用中显然很重要。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 MarkQWu/- 的 SKILL.md 是否有示例
grep -rL "## Example\|## Examples\|### Example" /tmp/my-repos/MarkQWu--/ --include="SKILL.md"

# 检查我的所有仓库是否有 curl|bash 模式
grep -rn "curl.*| bash\|curl.*|sh" /tmp/my-repos/MarkQWu-*/ --include="*.sh" 2>/dev/null | grep -v "node_modules"

# 检查我的命令是否有裸 $ARGUMENTS 未做空值处理
grep -rn "\$ARGUMENTS" /tmp/my-repos/MarkQWu-*/ --include="*.md" 2>/dev/null | grep -v ":-\|if\|##"
```

命中后怎么办：
- SKILL.md 缺 Examples → 为每个 SKILL.md 各添加 2 个 input→output 示例块（每个 10-20 分钟）
- curl|bash → 改为下载 + sha256 校验的两步安装
- 裸 `$ARGUMENTS` → 加 `${ARGUMENTS:-<默认动作描述>}` 或在命令头部加 empty-input guard

### 6.2 灵感 → 实施路径
1. **想法**：在 bureau 和 MarkQWu/- 的 CLAUDE.md 里增加「禁止区」小节
   - **为何可行**：本仓库验证了"边界约束"比"允许范围"更能防止 Claude 误操作
   - **第一步**：在 MarkQWu/- 的 CLAUDE.md 里加 `## 不要做什么` 小节，列出 3-5 条边界（5 分钟）

2. **想法**：为 MarkQWu/- 的 10 个 SKILL.md 批量添加 Examples section
   - **为何可行**：这是 NLPM scoring 最高权重的缺陷（-15/skill），批量修复有最大 ROI
   - **第一步**：写一个通用示例模板，然后逐个 SKILL.md 填充领域特定的 input→output（每个 15 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例的核心目的**：为 Claude Code 提供「按需启动的 MCP 服务网关」，管理多个外部进程的生命周期
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/- | 低 | 都有 NL artifacts（SKILL.md）；都是工具型产品 | 我的是情报仪表板，不管理 MCP 进程 | 低（架构差异大，但 SKILL.md 质量问题有重叠） |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件；都有 CLAUDE.md 约束 | bureau 不涉及外部进程管理 | 低 |
| MarkQWu/graphify | 中 | 都是 AI 工具插件；都需要 SKILL.md 传递复杂用法 | graphify 是知识图谱，不是进程网关 | 中（SKILL.md 示例问题高度相关） |

若全部对齐度低，架构模式仍有参照价值：「NL 表皮 + 复杂原生核心」的 CLAUDE.md 禁止区模式对我有用。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 零示例 | `grep -rL "## Example" MarkQWu--/ --include="SKILL.md"` | **命中 10/10**（MarkQWu/- 全部 10 个 SKILL.md 无 Examples） | 高 |
| $ARGUMENTS 空值无处理 | `grep -rn "\$ARGUMENTS" MarkQWu-*/ --include="*.md"` | 未发现（我的命令未使用 $ARGUMENTS 模式） | 无 |
| curl\|bash 风险 | `grep -rn "curl.*bash" MarkQWu-*/ --include="*.sh"` | 未发现（无公开安装脚本） | 无 |

**命中后的具体行动建议**：
- `MarkQWu/-` 的 `public/.well-known/agent-skills/*/SKILL.md` → 为每个 SKILL.md 在文末加 `## Examples` 块，给出 1 个具体的 API 调用示例（输入国家/事件参数 → 输出格式示例）→ 每个文件约 10 分钟

### 8.3 别人的更优方案

1. **领域**：CLAUDE.md 禁止区约束
   - **本案例做法**：CLAUDE.md 专门有「推測できない境界」小节，逐条列出 Claude 不应该做的操作（如不自动注册全局 MCP 服务、不修改 mcp-config.json）
   - **我的项目现状**：MarkQWu/- 的 CLAUDE.md 不存在；bureau 的 CLAUDE.md 偏向"能做什么"的正向描述，无明确禁止区
   - **如何借鉴**：在 bureau/CLAUDE.md 末尾加 `## 不要做什么` 小节，列出 3-5 条不可越的边界（30 分钟）

2. **领域**：机器可读服务注册表（routing-table.json）
   - **本案例做法**：把服务配置放在 JSON 而不是 CLAUDE.md 散文里，机器可校验
   - **我的项目现状**：MarkQWu/- 的服务配置散落在 markdown 注释中
   - **如何借鉴**：把 MarkQWu/- 的 API 端点配置提取为一个 `config/services.json`，SKILL.md 引用这个文件路径

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：[frontmatter](#frontmatter) 规范度
- **我的做法**：MarkQWu/graphify 和 MarkQWu/gstack 的 SKILL.md 有完整的 frontmatter（name、description、model）
- **本案例弱点**：airis 的 skills 目录下 SKILL.md 格式非标准 Claude Code plugin 格式，缺少统一 frontmatter
- **意义**：我的 frontmatter 规范度是优势，维持这个习惯

---

## 八、术语表

### <a name="mcp"></a>MCP
> Model Context Protocol 的缩写。一种让 Claude 调用外部工具（数据库、浏览器、文件系统等）的协议标准。可以把它想成"Claude 能用的 USB 接口"——MCP 服务就是插进这个接口的各种外设。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（`name`、`description`、`model` 等）。Claude Code 先读这里才知道这个 skill/agent/command 是谁、能干什么。

### <a name="manifest"></a>manifest
> 项目的"清单文件"（通常是 `plugin.json`），列出所有组件的路径。清单里没有的文件，即使在硬盘上也不会被 Claude Code 加载。

### <a name="routing-table"></a>routing-table.json
> airis-mcp-gateway 项目里描述"哪个 MCP 服务走哪个端口、哪个 Docker 容器"的 JSON 配置文件。类比为路由器的路由表——决定流量走向。
