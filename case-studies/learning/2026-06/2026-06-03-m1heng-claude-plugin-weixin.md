# m1heng/claude-plugin-weixin — 学习案例

**仓库**：https://github.com/m1heng/claude-plugin-weixin
**Stars**：556 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-03（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `single-purpose`, `template-design`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-plugin-weixin`（仓库名 `claude-channel-weixin`）是一个让 Claude Code 在终端里收发微信消息的 Claude Code channel 插件。关键事实：
- 使用微信 iLink Bot API + HTTP 长轮询，不需要公网 Webhook
- 运行时依赖 Bun 和 TypeScript，MCP 服务器是 3 个 `.ts` 文件
- 安装后在 Claude Code 里运行 `/weixin:configure login` 扫码登录
- 目前是 Claude Code channels 研究预览阶段的非官方插件
- 仓库极小（6 个文件在根目录），但 NLPM 97/100——这是"小而精"的典范

### 1.2 架构剖析
```
m1heng/claude-plugin-weixin/
├── server.ts            # MCP 服务器主体（处理消息收发）
├── login-qr.ts          # 扫码登录（生成 QR 码）
├── login-poll.ts        # 登录状态轮询
├── package.json         # Bun 包配置（版本 0.1.0）
├── .mcp.json            # MCP 配置（声明服务器命令）
└── skills/
    ├── .claude-plugin/plugin.json   # 插件清单（版本 0.4.0）
    ├── access/SKILL.md              # 收发消息技能
    └── configure/SKILL.md           # 登录配置技能
```
- **文件类型分布**：2 个 SKILL.md，0 个 agent，0 个 command；MCP 服务器 3 个 TypeScript 文件
- **编排关系**：两个技能完全平列，`configure` 先于 `access` 使用（配置后才能收发消息），但没有显式的编排约束
- **跨件契约**：`access/SKILL.md` 和 `configure/SKILL.md` 与 `server.ts`、`login-poll.ts` 共享状态目录 `~/.claude/channels/weixin/`，字段名（`dmPolicy`, `allowFrom`, `pending`, `token`, `baseUrl`）三方一致，无术语漂移

### 1.3 设计思路 / 方法论
- **核心设计哲学**："协议化状态管理"——所有配置和运行时状态用 JSON 文件落地到 `~/.claude/channels/weixin/`，技能文档和 MCP 服务器都按同一个约定读写这些文件，形成隐式但严格的契约
- **解决什么问题**：Claude Code 没有内置的即时通讯接入点；这个插件填补了"从微信消息触发 Claude Code 任务"的空白
- **做了什么 trade-off**：用长轮询代替 Webhook，牺牲了实时性（秒级延迟），换来零基础设施要求（不需要公网 IP）
- **反映什么认知模型**：作者把 MCP 服务器视为"状态管理器"，技能文档视为"用户交互脚本"，两层职责分离清晰

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：微插件 + 原生 MCP 状态层**

插件体积极小（6 个文件），但通过精心设计的状态文件约定，把 NL 技能层（SKILL.md）和 TypeScript 运行时（MCP 服务器）无缝桥接。

模式特征清单：
- 特征 1：NL 技能仅负责用户交互（解释动作语义、输出反馈），不包含业务逻辑
- 特征 2：状态落地到 JSON 文件（`~/.claude/channels/weixin/*.json`），技能和服务器约定同一路径
- 特征 3：MCP 服务器处理所有 IO（HTTP 轮询、QR 码生成、消息队列），技能调用 MCP 工具
- 特征 4：极简 manifest（`plugin.json` 只声明技能路径，无 command 列表）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 第三方 API 桥接（消息/通知类） | ✅ 高度适用 | 状态管理 + 轮询是标准模式 |
| 需要实时低延迟的推送 | ❌ 不适用 | 长轮询有秒级延迟，不适合股票行情等实时场景 |
| 纯 NL、不需要后端的技能 | ❌ 过度设计 | TypeScript MCP 服务器是额外负担 |
| 需要持久登录状态的插件 | ✅ 适用 | 状态文件模式完美支持跨会话持久化 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 微插件 + 原生 MCP 状态层（本仓库） | claude-plugin-weixin | 极简、状态清晰、易维护 | 需要运行时（Bun） |
| 纯 NL 技能（无 MCP） | addyosmani/agent-skills | 零依赖，最简单 | 无法处理 IO 密集任务 |
| 全栈平台内嵌技能 | wecode-ai/Wegent | 功能完整 | 部署复杂，技能是二等公民 |

### 2.4 改进空间
1. **当前问题**：版本号在 `plugin.json`（0.4.0）和 `package.json`（0.1.0）之间不一致，已维持 2+ 个月。**改进做法**：在 `package.json` 的 `scripts` 里加一个 `version-sync` 脚本，在 `npm version` 时同步更新 `plugin.json`。**预期收益**：发版时自动同步，彻底消灭版本漂移。
2. **当前问题**：`start` 脚本每次都运行 `bun install`，供应链风险较高。**改进做法**：改为 `bun install --frozen-lockfile && bun server.ts`，强制使用 lockfile，阻止无意的依赖升级。**预期收益**：MCP 服务器每次启动都用相同的依赖版本。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **97/100**，安全扫描结论 **CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `skills/access/SKILL.md` | 95 | `Bash(ls *)` 声明了但从未调用；"gracefully" 模糊量词 |
| `skills/configure/SKILL.md` | 95 | `Bash(mkdir *)` 声明了但 Bun 脚本实际负责目录创建；"complete picture" 模糊 |
| `.claude-plugin/plugin.json` | 100 | 完美 |

### 3.2 当时值得借鉴的模式
1. **状态协议一致性**：`server.ts`、`access/SKILL.md`、`configure/SKILL.md`、`login-poll.ts` 四个文件，对 `~/.claude/channels/weixin/` 下的字段名 100% 一致。这是"协议化 frontmatter"的极好实践。借鉴：在我的 `echo-sleuth` 里，各技能对 `~/.claude/echo-sleuth/sessions/` 的字段引用是否一致？值得 grep 验证。
2. **明确的空参数处理**：每个技能都在开头说明"如果用户不带参数调用"的行为（"No args — status and guidance"），不留歧义。
3. **安全 access control**：`server.ts` 包含 `assertSendable`（路径遍历守卫）和原子写（tmp → rename），安全思维渗透到实现层面——在 97 分的技能背后，有同样严谨的代码。

### 3.3 当时的缺陷
1. **声明了未使用的工具**（`Bash(ls *)`, `Bash(mkdir *)`）：根本原因是重构后没有同步清理 `allowed-tools` frontmatter，工具声明和实现脱节。自查：我的 `allowed-tools` 是否有类似的"历史残留"？
2. **版本号漂移**（plugin.json 0.4.0 vs package.json 0.1.0）：根本原因是两个文件独立维护，发版时没有统一更新流程。自查：我的插件仓库是否有 plugin.json 版本号？有没有和其他文件同步？
3. **运行时 `bun install`**（供应链风险）：每次启动 MCP 服务器时都拉取依赖，不固定版本。根本原因是作者优先考虑了"方便用户开箱即用"，没有充分考虑锁文件的重要性。

### 3.4 当时的优化机会
1. 从 `allowed-tools` 中移除 `Bash(ls *)` 和 `Bash(mkdir *)`（2 分钟）
2. 同步 `plugin.json` 版本到 0.1.0 或反向将 `package.json` 升到 0.4.0（5 分钟，同时建立发版流程）
3. 在 `start` 脚本中改为 `bun install --frozen-lockfile`（1 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 版本漂移：plugin.json 0.4.0 vs package.json 0.1.0 | `grep '"version"' package.json .claude-plugin/plugin.json` | **仍然存在**（package.json=0.1.0, plugin.json=0.4.0）| 2 个月未修，无版本同步机制 |
| 运行时 `bun install` | `grep "bun install" package.json` | **仍然存在**（`"start": "bun install --no-summary && bun server.ts"`）| 供应链风险持续 |
| 未使用工具声明 | `grep "Bash(ls" skills/access/SKILL.md` | **仍然存在**（access/SKILL.md line 8: `- Bash(ls *)`）| 声明和实现仍不同步 |

### 4.2 架构演进
仓库结构无明显变化——6 个根文件，2 个 SKILL.md，3 个 TypeScript 文件。这是一个高质量但低维护频率的项目，作者在 2026-04 之后似乎没有推送新提交（或变更极少）。没有发现新增的可学习模式。

### 4.3 新增的可学习模式
暂无——仓库自 audit 以来无明显架构变化。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的 plugin.json 版本号维护——`MarkQWu-claude-for-legal` 所有 plugin.json 均为 1.0.2，且没有并行的 package.json（纯 NL 仓库），所以不存在版本漂移的问题
2. 我的技能有明确的空参数处理（`echo-sleuth` 的各个 commands 均有默认行为描述）
3. 我的仓库没有运行时自动拉取依赖的脚本

### 5.2 挑战 / 验证
本案例**验证**了一个我一直模糊持有的想法：**技能的"allowed-tools"声明和实际调用要保持同步**。`claude-plugin-weixin` 97 分的高质量文件里依然有 `Bash(ls *)` 这样的过时声明——证明即使高质量作者也会在重构后忘记清理 frontmatter。这不只是初学者的错，值得在项目里建立"重构前先检查 allowed-tools"的习惯。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能是否有声明了但实际不调用的工具
# 方法：列出 allowed-tools，再在技能正文里搜索是否真的有调用
for skill in ~/.claude/skills/*/SKILL.md; do
  tools=$(grep "allowed-tools:" "$skill" | sed 's/.*: //')
  echo "=== $skill ==="; echo "声明工具: $tools"
done
# 命中（声明了 Bash 但正文没有 bash 命令）后：从 allowed-tools 中删除对应工具

# 检查我的 plugin.json 版本号是否有对应的 package.json 且版本不一致
for f in $(find . -name "plugin.json"); do
  dir=$(dirname "$f")
  pkg="$dir/../package.json"
  [ -f "$pkg" ] && echo "plugin: $(grep version $f | head -1) | package: $(grep '"version"' $pkg | head -1)"
done
# 命中后：统一两个版本号，建立发版 checklist
```

### 6.2 灵感 → 实施路径

1. **想法**：在 `echo-sleuth` 加一个版本同步检查（如果有 plugin.json 的话）
   - **为何可行**：echo-sleuth 目前没有 package.json，但未来可能加；现在就建立"plugin.json 是版本的唯一来源"的约定
   - **第一步**：在 `echo-sleuth` README 里加一行"版本管理约定：只维护 plugin.json 版本，无 package.json"，5 分钟

2. **想法**：建立 `allowed-tools` 清理 checklist，每次重构技能时走一遍
   - **为何可行**：`claude-plugin-weixin` 的教训说明这是高频遗漏点
   - **第一步**：在个人笔记里（或 CLAUDE.md）加一条"重构技能后：grep allowed-tools，对照正文每个工具调用"，2 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **m1heng/claude-plugin-weixin 的核心目的**：把微信消息接入 Claude Code，实现 IM 触发 AI 任务
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同样是"把外部数据接入 Claude"的插件 | echo-sleuth 挖掘历史对话，weixin 是实时消息通道 | 中 |
| MarkQWu/claude-for-legal | 低 | 都有技能 | claude-for-legal 是专业领域工具，weixin 是通信桥接 | 低 |

若无更接近的项目，以技术模式对照为主（状态文件约定）。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 声明未使用的工具 | `grep "allowed-tools" echo-sleuth/commands/*.md` 再核对正文 | 未专门核查（需手工比对），但这是已知风险 | 中 |
| 命令缺 allowed-tools 声明 | `for f in commands/*.md; do grep -c "allowed-tools" "$f" \|\| echo "MISSING: $f"; done` | **echo-sleuth 命中 4 个**：recall.md, recap.md, timeline.md, lessons.md 缺 allowed-tools | 中 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/commands/recall.md` → 查看正文使用了哪些工具，补充 `allowed-tools: Bash, Read`，5 分钟
- `echo-sleuth-for-claude/commands/recap.md` → 同上，预计使用 `Bash`，5 分钟
- `echo-sleuth-for-claude/commands/timeline.md` → 同上
- `echo-sleuth-for-claude/commands/lessons.md` → 同上

### 8.3 别人的更优方案

1. **领域**：MCP 服务器状态协议化
   - **本案例做法**：`server.ts`、技能文件、登录模块三方共享 `~/.claude/channels/weixin/` 下的字段定义，且 audit 验证"无术语漂移"
   - **我的项目现状**：`echo-sleuth` 的各技能对 `~/.claude/echo-sleuth/` 路径有多处引用，但没有集中的"状态协议"文档，术语一致性靠记忆维护
   - **如何借鉴**：在 `echo-sleuth` 根目录加 `STATE-SCHEMA.md`，列出所有共享的目录路径和字段名；技能文件引用时指向这个文档

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 声明完整性（部分）
- **我的做法**：`echo-sleuth` 中 `audit.md`、`dashboard.md`、`extract.md`、`prune.md` 四个命令均有完整的 `allowed-tools` 声明
- **本案例做法（弱在哪）**：`claude-plugin-weixin` 有声明了但实际不调用的工具（`Bash(ls *)`, `Bash(mkdir *)`）——声明过多
- **意义**：我的问题是"遗漏声明"（recall/recap/timeline/lessons 缺 allowed-tools），而本案例的问题是"多余声明"，两种错误方向相反，都应避免

---

## 八、术语表

### <a name="MCP服务器"></a>MCP 服务器
> MCP（Model Context Protocol）是 Anthropic 定义的协议，允许外部程序以标准化方式向 Claude 提供工具和功能。"MCP 服务器"是实现这个协议的程序（通常是本地进程），Claude Code 通过标准输入输出和它通信。本插件的 `server.ts` 就是一个 MCP 服务器，负责处理微信消息的收发。

### <a name="长轮询"></a>HTTP 长轮询
> 一种消息推送的替代方案：客户端向服务器发送请求并保持连接打开，服务器在有新消息时才返回响应，然后客户端立即再发新请求。比 Webhook（服务器主动推送）实现简单，但需要稳定的客户端连接，且有秒级延迟。

### <a name="lockfile"></a>lockfile（依赖锁文件）
> 一个记录项目所有依赖的精确版本号的文件（如 `bun.lockb` 或 `package-lock.json`）。有了 lockfile，每次 `bun install --frozen-lockfile` 都会安装完全相同的版本，避免因依赖库悄悄更新而引入未知变化。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读技能文件时先解析 frontmatter 才知道如何注册和调用这个技能。
