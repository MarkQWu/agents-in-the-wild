# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream audit
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-08-01（基于当前 HEAD）
**主题标签**：`security-gate`, `curl-pipe-bash-risk`, `manifest-discipline`, `nl-binary-hybrid`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
AIRIS MCP Gateway 是一个**本地按需启动型 [MCP](#MCP) 代理网关**，将多个 MCP 服务器整合为统一端点，供 Claude Code 及其他 AI 编码客户端按需调用。作者是 agiletec-inc 团队，仓库以日文写 [CLAUDE.md](#CLAUDE.md) 为显著特色——这是罕见的非英文项目指令风格。核心功能：用一条命令（`airis enable <server-name>`）注册/注销 MCP 服务器，不需要永久后台进程，支持 Codex / Claude Desktop / Gemini 多客户端。安装路径为 `curl … | bash` 一键脚本。151 ★。

关键事实：
1. 用 Docker Compose 管理 gateway 生命周期，Context7 是唯一内置目标
2. CLAUDE.md 以日文写成，记录"无法从上下文推断的边界"
3. 历史上存在独立 `skills/` Claude Code 插件子包，当前版本已将其完全移除
4. 安装脚本 `install.sh` + `scripts/quick-install.sh` 是主要可执行面
5. 运营级设计：有 `Taskfile.yml`、`devbox.json`、`compose.yaml`，面向 DevOps 场景

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── install.sh                # 一键安装脚本（主要安全面）
  ├── scripts/
  │   ├── airis-mcp-gateway     # 主 CLI（enable/disable/status）
  │   └── airis_bootstrap.py   # Python 初始化脚本
  ├── config/
  │   └── bootstrap/assets/claude/commands/  # Claude commands
  ├── apps/                    # TypeScript API 应用
  ├── ops/                     # Linux/macOS autostart 脚本
  └── .claude/commands/        # Claude 本地命令
  ```
- **文件类型分布**：5 个 command、0 个 SKILL.md（技术上已移除整个 skills/ 子包）、0 个 agent、多个 shell script、1 个 Python bootstrap
- **编排关系**：commands 之间无互引；CLI 脚本（airis-mcp-gateway）直接调用 Docker 和本地文件；Claude commands 是调用 CLI 的入口点
- **跨件契约**：commands 调用 shell CLI，不通过 skill 体系路由；plugin.json 历史上存在，当前已不存在于仓库顶层

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「按需启动，不常驻」——违反"MCP 服务器始终在线"惯例，通过短命 session 节省资源
- **解决什么问题**：用户管理多个 MCP 服务器时的命名冲突和端口占用，以及 `mcp.json` 需要手动维护的复杂性
- **做了什么 trade-off**：选择 Docker + Shell 脚本做 gateway 层（可靠、可测），而非用 Claude skill/agent 体系内实现（更复杂、不可信）；但引入了 shell 注入和 curl|bash 风险
- **反映什么认知模型**：作者视 Claude commands 为"轻量入口"，核心逻辑在 shell/Python 层；NL 层很薄（只负责调用 CLI），执行层很厚

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 薄皮 + Shell 厚核」**

Claude commands 仅作为命令触发器，核心逻辑在 shell/Python 中实现。NL 层不承载业务逻辑，只是对 CLI 的语言包装。

模式特征清单：
- 特征 1：commands 正文只有 1-3 行，核心是 `Bash` 调用 shell CLI
- 特征 2：所有状态和逻辑在 shell/Python 层维护，而非 Claude context
- 特征 3：可测试性高——shell 逻辑可独立于 Claude 测试
- 特征 4：Claude 权限面最小——commands 只需 `Bash` 工具
- 特征 5：NL 文件的角色是"文档 + 触发器"，不是"指令集"

### 2.2 适用场景
这个架构最适合需要系统级操作（进程管理、网络配置、文件系统）的工具型项目。

| 场景 | 适不适用 | 原因 |
|---|---|---|
| MCP 服务器管理/注册工具 | ✅ 高度适用 | shell 控制 Docker/进程比 NL 更可靠 |
| 纯 NL 推理类 skill（写作、分析） | ❌ 不适用 | 这种场景需要 NL 层承载逻辑，薄皮模式浪费结构 |
| 需要跨 session 状态的工作流 | ⚠️ 谨慎 | shell 层管状态可以，但 Claude skill 体系更易组合 |
| 快速原型/个人工具 | ✅ 适用 | 开发速度快，shell 熟悉 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 薄皮 + Shell 厚核（本仓库） | airis-mcp-gateway | 可测试、权限最小、shell 生态复用 | 安全风险大、可组合性差 |
| NL 全层（skills + agents）| jnuyens/gsd-plugin | 可组合、易扩展、无 shell 依赖 | 逻辑在 NL 层，难以独立测试 |
| 混合层（NL 编排 + 原生核心）| 777genius/claude-notifications-go | 性能最优、跨平台 | 开发门槛高，需维护两套语言 |

### 2.4 改进空间
1. **当前问题**：CLI 触发入口没有幂等保护，enable 同一服务两次会出问题。**改进做法**：在 `airis-mcp-gateway enable` 检查 registry.json 已存在时幂等退出。**预期收益**：消除用户重复操作引起的状态损坏。
2. **当前问题**：commands 缺少 allowed-tools 声明（历史遗留）。**改进做法**：为每个 command 加 `allowed-tools: [Bash]`（仅需要 Bash）。**预期收益**：权限面收窄，符合 NLPM 规范。
3. **当前问题**：日文 CLAUDE.md 对国际贡献者不友好。**改进做法**：加英文对照译文或用双语块。**预期收益**：降低贡献门槛。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 当时得分 **90/100**。总体 NL 质量高，但安全被 BLOCKED（3 Critical + 1 High）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `airis-research-first.md` | 78 | 无输出格式 + 步骤用无序列表 |
| `skills/mcp-database/SKILL.md` | 83 | 无示例 + 模糊量词 |
| `skills/mcp-debugging/SKILL.md` | 85 | 无示例 |
| `skills/mcp-implementation/SKILL.md` | 85 | 无示例 |
| `skills/mcp-research/SKILL.md` | 85 | 无示例 |
| `.claude/commands/troubleshoot.md` | 90 | 无空 $ARGUMENTS 处理 |
| `.claude/commands/test.md` | 96 | 模糊量词 |
| `CLAUDE.md` | 98 | 模糊量词 |

### 3.2 当时值得借鉴的模式
1. **日文 CLAUDE.md 写「推测不到的边界」** → 聚焦于"读者不可能从代码推断的约束"，而非描述功能——根本原因是避免 CLAUDE.md 变成 README 镜像。可借鉴：我的 CLAUDE.md 应专注于"隐藏约束"，不重复 README 内容。
2. **`plugin.json` 100 分** → [manifest](#manifest) 与实际文件完全对齐。可借鉴：每次增删 skill/command 时同步更新 manifest。
3. **commands 有明确 allowed-tools 声明** → 权限声明明确。（注：当前版本已部分失去此优点。）
4. **troubleshoot command 有空 $ARGUMENTS 处理** → 空参数下给出提示而非空白渲染。可借鉴：所有需要参数的 command 加空参数守卫。

### 3.3 当时的缺陷
1. **curl|bash 安装模式（Critical）** → 文档化了 `curl … | bash` 作为标准安装路径。为什么会失败：用户执行远程脚本时无法验证完整性，攻击者一旦控制 CDN/GitHub raw 就能在用户机器上执行任意代码。自查：我的项目有没有文档化 curl|bash 安装？（有：查 README 的安装段落）
2. **Shell 变量注入（High → 降为 Critical 功能等价）** → `server = '$server'` 将 shell 变量直接插入 Python heredoc，服务器名含 `'` 或 `$(...)` 时破坏代码结构。为什么会失败：shell 字符串拼接不能安全传递不可信输入，需要参数化（`sys.argv`）代替插值。自查：我的 shell 脚本有没有直接拼接用户输入？
3. **undeclared external dependency（mcp-debugging SKILL → superpowers plugin）** → skill 要求用户先有 `superpowers:systematic-debugging`，但 plugin.json 未声明此依赖。为什么会失败：新用户 clone 后 skill 给出无效指引，调用了不存在的工具。自查：我的 skill 有没有引用未声明的外部 skill/plugin？

### 3.4 当时的优化机会
1. 为 4 个 skills 加示例块（mcp-database, mcp-debugging, mcp-implementation, mcp-research）——已通过移除整个 skills 包"解决"
2. 将 airis-research-first.md 的步骤改为有序列表 + 加输出格式段落
3. 为 troubleshoot command 加空 $ARGUMENTS 处理分支

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `install.sh` curl\|bash 文档化 | `grep -n "curl.*bash" install.sh` | **持续**（lines 6, 266）| 作者选择接受此风险或优先级低于功能开发 |
| shell 变量注入 `server = '$server'` | `grep -n "server = '\$server'" scripts/airis-mcp-gateway` | **持续**（lines 266, 314）| 脚本被重命名（airis-gateway → airis-mcp-gateway），但注入代码未修 |
| skills 无示例块 | `find skills/ -name "SKILL.md"` | **已消除**——整个 skills/ 包被移除 | 根本解法不是修 skill，而是架构决策：NL 层彻底下移到 commands |

### 4.2 架构演进
最显著变化：**独立 Claude Code plugin（skills/ 子包）被完全移除**。过去 audit 时还有 `skills/skills/mcp-database/`、`skills/skills/mcp-debugging/` 等 4 个 SKILL.md + plugin.json。现在这一层全部消失，仓库回归纯 Shell + TypeScript 实现，Claude 集成只通过 `.claude/commands/` 和 `config/bootstrap/assets/claude/commands/` 保留。

这说明作者后来意识到：**维护 NL skill 层的成本大于收益**，或者 skill 体系与其架构目标（per-session gateway，不需要持久 skill 加载）不匹配。

### 4.3 新增的可学习模式
- **日文 CLAUDE.md 边界清单格式**：当前 CLAUDE.md 仍然保留，且结构更精炼——每条边界一行，无装饰，直奔约束本体。这是"最小有效 CLAUDE.md"的极端示例。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的 CLAUDE.md 专注于开发约束，不复制 README 功能描述——与 airis 的边界清单哲学对齐
2. 我的 bureau/skills/ 有示例块（recall/SKILL.md 有 3 个示例）——比 airis 历史时期更好
3. 我的 agent（bureau/auditor）有明确的 `tools:` 声明——比 airis commands 更规范

### 5.2 挑战 / 验证
这次案例**挑战了我"NL 层一定需要厚实内容"的假设**。airis 的最终方向是把 NL 层削薄到只剩触发器，把可靠性交给 shell。这不是退化，而是面向系统工具类项目的合理选择。验证了一个判断：**NL skill 层不适合所有场景，系统级工具宁可用 shell 做核心**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的项目 README 是否文档化了 curl|bash 安装方式
grep -rn "curl.*bash\|curl.*sh.*|.*bash" ~/projects/*/README.md 2>/dev/null | grep -v "#" | head -10
```
命中后怎么办：加 SHA256 校验步骤，或改为 brew install / pip install 等包管理器方式。

```bash
# 检查我的 commands 是否有空参数守卫
grep -rL "default\|if.*-z\|ARGUMENTS.*empty" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
```
命中后怎么办：在 command 顶部加 `{% if args[0] == '' %}Please provide...{% endif %}` 或同等守卫。

```bash
# 检查我的 skills 是否引用了未声明的外部 skill/plugin
grep -rn "superpowers:\|:skill\|plugin:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：在 plugin.json 的 `dependencies` 字段声明，或在 skill 正文加"如未安装 X 则…"的降级路径。

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau commands 加 `allowed-tools` 声明
   - **为何可行**：每个 command 的工具需求都很清楚（lint → Bash, Glob; review → Read, Grep）
   - **第一步**：遍历 `commands/` 下 11 个文件，逐一加 `allowed-tools:` 行。约 15 分钟。

2. **想法**：CLAUDE.md 改为"边界清单"格式
   - **为何可行**：airis 日文 CLAUDE.md 证明一行一条边界足够清晰
   - **第一步**：把现有 CLAUDE.md 里的段落压缩为 bullet list，每条以"不要…"或"只…"开头。30 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 agiletec-inc/airis-mcp-gateway 的核心目的**：本地 MCP 服务器管理工具，供 AI coding 客户端按需使用

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件 + 管理知识/工具 | bureau 管理知识库，airis 管理 MCP 服务器 | 中（安全实践可借鉴） |
| MarkQWu/gstack | 低 | 都是 Claude Code 工具集 | gstack 是 skill 集合，airis 是 gateway 工具 | 低 |
| MarkQWu/- | 无 | — | 完全不同的应用场景 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺少 allowed-tools 声明 | `grep -rL "allowed-tools" MarkQWu-bureau/commands/*.md` | bureau 命中 11/11 个 commands | 中 |
| skill 引用未声明外部依赖 | `grep -rn "外部 skill 引用" skills/*/SKILL.md` | 未命中 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/commands/lint.md` → 加 `allowed-tools: [Bash, Glob, Read]` → 5 分钟可完成
- `MarkQWu/bureau/commands/review.md` → 加 `allowed-tools: [Read, Grep, Glob]` → 5 分钟可完成
- 批量：用一个 PR 给全部 11 个 commands 加 allowed-tools，约 20 分钟

### 7.3 别人的更优方案

1. **领域**：CLAUDE.md 边界清单写法
   - **本案例做法**：每条边界一行，以"不要…"开头，只写"读者从代码推断不出来的约束"（`CLAUDE.md`）
   - **我的项目现状**：MarkQWu/bureau/CLAUDE.md 结构较松散，混杂了功能描述和约束
   - **如何借鉴**：重写每个项目的 CLAUDE.md，只保留约束行，删除所有可以从代码/README 推断的内容

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：agent model 声明
- **我的做法**：MarkQWu/bureau/.claude/agents/auditor.md 有 `model: sonnet` 声明
- **本案例做法（弱在哪）**：airis 历史版本的 skills 无 model 声明，现版本 skills 已被移除，无法比较
- **意义**：bureau 的 agent 明确声明了模型，cost 可预测，这是比 airis 历史时期更规范的实践

---

## 八、术语表

### <a name="MCP"></a>MCP
> Model Context Protocol，Anthropic 定义的开放协议，让 AI 模型（如 Claude）能以标准化方式连接外部工具和数据源。MCP 服务器是一个实现了这个协议的小程序，Claude Code 通过它访问数据库、浏览器、代码执行环境等能力。

### <a name="CLAUDE.md"></a>CLAUDE.md
> 放在项目根目录的特殊 Markdown 文件，Claude Code 在每次会话开始时自动读取。作用类似于"给 AI 的工作说明书"——告诉 Claude 这个项目有哪些特殊约束、不能做什么、术语是什么。

### <a name="manifest"></a>manifest
> 项目的"清单文件"。Claude Code 插件的 manifest 是 `plugin.json`，里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在磁盘上也不会被加载。

### <a name="curl-pipe-bash"></a>curl|bash 模式
> 一种常见但危险的安装模式：`curl https://example.com/install.sh | bash`。含义是"从网上下载一段脚本，立刻以 root 权限执行"。危险在于用户无法在执行前验证脚本内容——如果服务器被攻击或 CDN 被劫持，用户机器就会被入侵。
