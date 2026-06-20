# m1heng/claude-plugin-weixin — 学习案例

**仓库**：https://github.com/m1heng/claude-plugin-weixin
**Stars**：556 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-20（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `single-purpose`, `security-gate`, `template-design`, `nl-binary-hybrid`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

把微信接入 Claude Code 的插件——让 Claude 可以读取和发送微信消息。架构是：Claude 通过 NL skill 操作一个本地运行的 TypeScript [MCP 服务器](#MCP-服务器)，MCP 服务器通过微信 Web 协议（扫码登录）接管微信账号。556 stars，作者是 @m1heng。

关键事实：
1. 极小代码量：2 个 SKILL.md + 1 个 plugin.json + 3 个 TypeScript 文件（server.ts / login-qr.ts / login-poll.ts）
2. 状态存储在 `~/.claude/channels/weixin/`——通过 YAML 文件在 skill 和 MCP 服务器之间传递配置和令牌
3. 安全门：`server.ts` 里有 `assertSendable` 函数做路径穿越防护（realpathSync）+ 原子写（tmp→rename），这是整个仓库里最高质量的代码
4. 依赖 Bun 运行时（不是 Node.js）

### 1.2 架构剖析

```
claude-plugin-weixin/
├── .claude-plugin/
│   └── plugin.json           # version: 0.4.0 （← 与 package.json 0.1.0 不同步）
├── skills/
│   ├── access/
│   │   └── SKILL.md          # 微信消息收发（allowed-tools 声明了未使用的 Bash(ls *)）
│   └── configure/
│       └── SKILL.md          # 扫码登录 + 参数配置
├── server.ts                 # MCP 服务器主体（含 assertSendable 安全门）
├── login-qr.ts               # 二维码展示 + 自动安装依赖（← 安全隐患）
├── login-poll.ts             # 轮询登录状态
└── package.json              # version: 0.1.0; bun install 在 start script（← 安全隐患）
```

- **文件类型分布**：2 个 SKILL.md、0 个 agent、0 个 command、0 个 hook
- **编排关系**：双层——SKILL.md（Claude 的 NL 层）驱动 MCP 工具调用（server.ts 的 MCP endpoint），不经过任何 agent 路由
- **跨件契约**：`skills/access/SKILL.md`、`skills/configure/SKILL.md`、`server.ts` 和 `login-poll.ts` 四个文件在 `~/.claude/channels/weixin/` 路径和状态字段（`dmPolicy`、`allowFrom`、`pending`、`token`、`baseUrl`）上完全一致，**无术语漂移**

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「Claude 操作 MCP，MCP 操作微信」——NL 层（SKILL.md）不处理任何业务逻辑，业务逻辑全在 TypeScript MCP 服务器里。这样 Claude 的权限是有边界的（只能调用 MCP 暴露的工具），不能乱改微信配置
- **解决什么问题**：把微信这个无公开 API 的封闭系统接入 Claude Code 工作流——通过微信 Web 协议（扫码登录）模拟客户端行为
- **Trade-off**：
  - 选 Bun 运行时（不是 Node.js），利用 Bun 的 native MCP server 支持和更快的启动速度，但要求用户安装 Bun
  - 选"Claude 操作 MCP"而非"Claude 直接调用微信 API"，付出了架构复杂度，换来了权限边界清晰
  - 把 `bun install` 放在 `start` script 里，方便用户首次运行，但造成了供应链风险
- **认知模型**：作者把 Claude 看作"高级用户"，MCP 服务器看作"权限边界"——Claude 不需要知道微信 API 的细节，只需要调用 MCP 工具

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「NL 表皮 + TypeScript MCP 服务器 + 外部系统桥接」**

SKILL.md 只是 MCP 工具调用的说明书，真正的业务逻辑在 MCP 服务器里。MCP 服务器充当 Claude 和外部系统（微信）之间的权限边界和协议适配层。

模式特征清单：
- NL 层（SKILL.md）只描述工具使用方法，不含业务逻辑
- MCP 服务器是实际的权限边界（assertSendable 是例证）
- 状态通过文件系统（YAML 文件）在 NL 层和 MCP 层之间共享
- 外部系统（微信）通过逆向工程/非官方协议接入（不依赖官方 API）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 接入无官方 AI API 的服务（微信、企业内网工具）| ✅ 高度适用 | MCP 层负责协议适配，NL 层不需要知道细节 |
| 需要强权限控制的场景（只允许 Claude 做特定操作）| ✅ 适用 | MCP 服务器天然是权限边界 |
| 高并发、生产级消息系统 | ❌ 不适用 | 微信 Web 协议不稳定，随时可能被封禁 |
| 团队共享（多人同时使用同一账号）| ❌ 不适用 | 微信单账号限制，状态存在 ~/.claude 里，不支持多用户 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：NL + MCP 服务器 | m1heng/claude-plugin-weixin | 权限边界清晰，MCP 层可测试 | 需要 Bun 运行时，本地 MCP server 需保持运行 |
| 纯 NL skill（直接调用外部 API）| 假设场景 | 简单，无额外进程 | 无权限边界，Claude 可以任意调用 API |
| 完全自定义 agent | 企业级场景 | 最灵活 | 维护成本最高，需要自己管理进程生命周期 |

### 2.4 改进空间

1. **当前问题**：plugin.json（0.4.0）和 package.json（0.1.0）版本不同步。**改进做法**：用 `jq` 在发布脚本里自动同步两个版本号，或只用一个来源（如 package.json）作为权威版本，plugin.json 通过脚本生成。**预期收益**：Claude Code 和 npm 看到的版本一致，不再混乱。

2. **当前问题**：`package.json` 的 `start` script 里有 `bun install --no-summary`，MCP 服务器每次启动都执行包安装，引入供应链风险。**改进做法**：改为 `bun install --frozen-lockfile`（确保只安装 lockfile 里的版本）或完全移除，要求用户在首次安装时手动运行 `bun install`。**预期收益**：每次启动结果可预期，消除自动拉取恶意 minor 版本的风险。

3. **当前问题**：`skills/access/SKILL.md` 的 `allowed-tools` 里声明了 `Bash(ls *)` 但 SKILL.md 正文没有任何 `ls` 调用步骤。**改进做法**：从 `allowed-tools` 里删除 `Bash(ls *)`。**预期收益**：Claude 不会因为 allowed-tools 里有 `ls` 而尝试用 ls 做不该做的事。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **97/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `skills/access/SKILL.md` | 95/100 | `Bash(ls *)` 声明但未使用（−3），"gracefully" 模糊量词（−2） |
| `skills/configure/SKILL.md` | 95/100 | `Bash(mkdir *)` 声明但未使用（−3），"complete picture" 模糊量词（−2） |
| `.claude-plugin/plugin.json` | 100/100 | — |

### 3.2 当时值得借鉴的模式

1. **状态契约四文件全一致** → 为什么好：access/SKILL.md、configure/SKILL.md、server.ts、login-poll.ts 四个文件对 `~/.claude/channels/weixin/` 路径和字段名（`dmPolicy`、`allowFrom` 等）的认知完全一致，没有一个文件用了不同的名字 → 原文：audit 报告"No terminology drift"章节 → 借鉴：凡是多个文件共享的状态契约，第一次写完就该做一次全量 grep，确保命名统一

2. **`assertSendable` 安全门设计** → 为什么好：`server.ts` 里用 `realpathSync` 验证目标路径在允许的目录范围内，原子写（tmp→rename）防止部分写入 → 这是防御性编程的教科书案例 → 借鉴：任何写文件的 skill/script 都应该有路径边界检查

3. **空参数处理** → 为什么好：两个 SKILL.md 都有"No args — status and guidance"分支，Claude 在无参数时有明确的降级行为 → 原文：access/SKILL.md 和 configure/SKILL.md 各自的"No args"章节 → 借鉴：任何有参数的 skill 都需要明确处理"无参数调用"的情况

### 3.3 当时的缺陷

1. **plugin.json vs package.json 版本漂移（0.4.0 vs 0.1.0）** → 根本原因：作者在维护 Claude 插件版本和 npm 包版本时用了不同的计数方式（插件更新了，npm 包忘了同步，或者反过来）——两个版本字段没有自动化约束 → 自查：我的 claude-for-legal 各 plugin.json 版本都是 1.0.2，但有没有对应的 package.json 版本？

2. **`Bash(ls *)` 声明在 allowed-tools 但从未使用** → 根本原因：开发时测试某个子命令需要 ls，后来改了实现方式，忘了从 allowed-tools 删掉 → allowed-tools 里有幽灵声明会让 Claude 误以为可以用 ls 做任意事 → 自查：我的 SKILL.md 的 allowed-tools 里有没有已经不用的工具？

3. **`login-qr.ts` 运行时自动 `bun install`** → 根本原因：作者想减少用户的配置负担（首次运行自动安装依赖），但这把安装过程推到了运行时——任何 `bun install` 在运行时执行都可能拉取被篡改的依赖 → 自查：我的任何脚本有没有在运行时而不是安装时做包安装？

### 3.4 当时的优化机会

1. **版本同步**：用 `jq` 脚本在发版时自动同步 plugin.json 和 package.json 的版本号
2. **删除 `Bash(ls *)`** from access/SKILL.md 的 allowed-tools
3. **`start` script 改为 `bun install --frozen-lockfile && bun server.ts`**，防止每次启动拉取新版本

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（在 clone 里运行） | 现状 | 含义 |
|---|---|---|---|
| plugin.json(0.4.0) vs package.json(0.1.0) 版本漂移 | `cat .claude-plugin/plugin.json \| python3 -c "import json,sys;print(json.load(sys.stdin)['version'])"` + `cat package.json \| python3 -c "..."` | **仍存在**：plugin.json=0.4.0，package.json=0.1.0 | 2+ 个月，未修复 |
| `bun install` 在 start script | `grep "bun install" package.json` | **仍存在**：line 8 `"start": "bun install --no-summary && bun server.ts"` | 运行时安装依然触发 |
| `Bash(ls *)` 声明但未使用 | `grep "Bash(ls" skills/access/SKILL.md` | **仍存在**：frontmatter line 8 `- Bash(ls *)` | 幽灵 allowed-tool 未清理 |
| `login-qr.ts` 运行时 bun install | `grep "bun install" login-qr.ts` | **仍存在**：line 15 `Bun.spawnSync(['bun', 'install', '--no-summary'],...)` | 自动安装依然存在 |

### 4.2 架构演进

与 audit 时完全相同——目录结构、文件列表、文件内容均未变化。这是一个自 2026-04-06 以来处于"功能完成但未活跃维护"状态的仓库。所有已知 bug 和安全问题均未修复。

### 4.3 新增的可学习模式

暂无——当前 HEAD 与 audit 时完全相同。

---

## 五、校准

### 5.1 我已经在做对的

1. **状态文件契约一致性**：echo-sleuth 里所有 agent 和 skill 对 `~/.claude/projects/` 路径的引用都一致，没有术语漂移
2. **空参数分支处理**：drama-workshop-skills 的 SKILL.md 里有各种条件分支（如"如果没有给剧名则询问"）
3. **权限最小化思维**：我在写 skill 时倾向于不给太多 allowed-tools，与这个仓库的 allowed-tools 精确声明理念一致

### 5.2 挑战 / 验证

这次案例**验证**了我对"版本号要有单一权威来源"的认知。m1heng/claude-plugin-weixin 的 plugin.json 和 package.json 版本号不同步，暴露了"两处维护"的根本问题。解决方案不是提醒自己记得同步，而是用工具自动强制同步。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 的 allowed-tools 里有没有在正文中从未被调用的工具
for f in $(find . -name "SKILL.md"); do
  tools=$(sed -n '/allowed-tools:/,/^[^-]/p' "$f" | grep "- Bash\|- Write\|- Read" | grep -oP '(?<=- )\w+\([^)]+\)')
  echo "=== $f ==="
  echo "$tools"
done
```
命中后怎么办：grep 正文确认是否有对应调用，没有则删除该 allowed-tool。

```bash
# 检查我的脚本里有没有在运行时（而不是安装时）执行包安装
grep -rn "npm install\|pip install\|bun install" . --include="*.sh" --include="*.ts" --include="*.js" | grep -v "scripts/install\|postinstall\|setup.sh"
```
命中后怎么办：把运行时安装移到 `setup.sh` 或安装文档里，运行时脚本假设依赖已安装，缺失时打印错误并 exit 1。

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 里加一个 MCP-aware skill，让 Claude 知道什么时候该通过 MCP 工具操作而不是直接读文件
   - **为何可行**：参考 claude-plugin-weixin 的"SKILL.md 只是 MCP 工具使用说明书"模式，可以大幅简化 skill 内容
   - **第一步**：在 `skills/jsonl-core/SKILL.md` 里加一个 MCP 工具使用章节，说明什么情况下用 Bash 脚本 vs 什么情况下等待 MCP 工具；预计 20 分钟

2. **想法**：给 claude-for-legal 的 plugin.json 加版本同步检查脚本
   - **为何可行**：claude-for-legal 有多个 plugin.json（每个 practice area 一个），版本漂移风险高
   - **第一步**：写一个 `scripts/check-versions.sh`，对比所有 `.claude-plugin/plugin.json` 的版本号是否一致；如果不一致 exit 1；在 CI 或 pre-commit 里调用；预计 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 m1heng/claude-plugin-weixin 的核心目的**：通过 MCP 服务器把微信（无官方 API 的封闭系统）接入 Claude Code，让 Claude 能读写微信消息

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是把外部系统（法律系统/法院数据库）接入 Claude，有 MCP connectors | claude-for-legal 用官方 API；weixin 用逆向工程协议 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 TypeScript/Python 执行层 + NL skill 层 | echo-sleuth 操作本地文件；weixin 操作远程消息系统 | 低 |
| MarkQWu/drama-workshop-skills | 无 | — | 内容创作 vs 消息系统集成，无关 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 版本与 package.json 不同步 | `find . -name "plugin.json" -path "*/.claude-plugin/*" \| xargs grep version` | claude-for-legal: 所有 plugin.json 版本均 1.0.2，无 package.json，**暂无此问题** | 暂无 |
| 运行时自动 install 依赖 | `grep -rn "npm install\|bun install" . --include="*.sh"` | 未检测到运行时安装 | 暂无 |
| allowed-tools 声明了未使用的工具 | `grep -A5 "allowed-tools" */SKILL.md` | echo-sleuth skills 无 allowed-tools 声明（使用 Claude 默认工具集）| 暂无 |

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：原子写文件（tmp→rename + realpathSync 路径验证）
   - **本案例做法**：`server.ts` 里 `assertSendable` 用 `realpathSync` 验证写入路径在允许范围内，然后先写到 `.tmp` 文件再 rename，防止部分写入
   - **我的项目现状**：echo-sleuth 的 `scripts/` 里有直接写文件的操作（如 `build-index.sh`），没有路径验证和原子写
   - **如何借鉴**：在 `scripts/build-index.sh` 里加路径验证（确认目标在 `~/.claude/projects/` 内），写到临时文件再 rename；预计 10 分钟

2. **领域**：空参数的明确降级行为
   - **本案例做法**：两个 SKILL.md 都有"No args — show status"分支，Claude 在无参数时展示当前状态，不报错
   - **我的项目现状**：drama-workshop-skills 的 `/开始` 命令遇到无参数时会问用户问题，但 echo-sleuth 的 skill 遇到无参数会怎么做不明确
   - **如何借鉴**：在 echo-sleuth 各 SKILL.md 里加"无参数时展示 usage 示例"的分支说明

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：`<example>` 块
  - **我的做法**：echo-sleuth 虽然目前没有 example 块（已知待修），但 drama-workshop-skills 的 SKILL.md 有完整的使用流程描述
  - **本案例做法**：m1heng/claude-plugin-weixin 的两个 SKILL.md 里**完全没有 `<example>` 块**，用户不知道怎么用
  - **意义**：尽管我也有 example 缺失的问题，但 claude-plugin-weixin 的缺失更严重——两个 SKILL.md 一个都没有示例，对于首次使用微信集成的用户来说体验很差

---

## 八、术语表

### <a name="MCP-服务器"></a>MCP 服务器
> Model Context Protocol 服务器。这是 Claude 能调用的"工具服务器"——它暴露一组工具（函数），Claude 可以通过 MCP 协议调用这些工具。就像 Claude 的"手"：Claude（大脑）通过 MCP 工具操作外部系统，外部系统不需要直接跟 Claude 通信。本例中 `server.ts` 就是 MCP 服务器，它把微信操作包装成 Claude 可以调用的工具。

### <a name="供应链风险"></a>供应链风险
> 当你的代码依赖第三方包（如 npm 包），而第三方包被恶意篡改时，你的系统也会被攻击。例如：`bun install` 在运行时触发，如果 npm registry 里的 `qrcode-terminal` 包被替换成含恶意代码的版本，下一次启动 MCP 服务器就会执行恶意代码。防御方法：固定依赖版本 + 提交 lockfile + 用 `bun install --frozen-lockfile` 确保只安装 lockfile 里记录的精确版本。

### <a name="原子写"></a>原子写
> 先把数据写到临时文件（如 `config.tmp`），写完之后再用 rename 操作把临时文件改名为目标文件（`config.yaml`）。好处是：如果写入中途出错，原文件不受影响；rename 操作在大多数操作系统上是原子的（要么成功要么失败，不会出现"写了一半"的状态）。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 的工具权限和调用方式。
