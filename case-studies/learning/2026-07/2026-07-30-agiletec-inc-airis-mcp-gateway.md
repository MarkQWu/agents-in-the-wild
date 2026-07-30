# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream audit
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-07-30（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `manifest-discipline`, `examples-driven`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`agiletec-inc/airis-mcp-gateway` 是一个 MCP（Model Context Protocol）网关，将多个 MCP 服务器统一管理在一个进程中，通过 Claude Code 插件提供命令接口（`status`、`test`、`troubleshoot`、`airis-capability-router`）。设计目标是解决用户配置多个 MCP 服务器时的启动/健康检查/排查问题，提供统一的管理平面。当前版本包含 Docker 化部署（`compose.yaml`）、自动启动配置（`ops/autostart/`）和 Python 引导脚本。

关键事实：
- 11 个 NL 工件，audit 时 90/100（最高分的待处理案例），SECURITY BLOCKED（3 个 Critical）
- Stars 151，是本批次 4 个案例中 stars 最多的
- 安装方式：`curl -fsSL .../install.sh | bash`（[curl-pipe-bash](#curl-pipe-bash) 反模式，至今未修复）
- 4 个 skill 均无 examples 块（审计后 3 个月仍未修复）
- 跨件亮点：所有命令有完整 frontmatter 和 `allowed-tools`，是本批次中「命令质量最优」的仓库

### 1.2 架构剖析
```
airis-mcp-gateway/
├── CLAUDE.md                ← 项目指令（98/100，高质量）
├── .claude/
│   └── commands/
│       ├── status.md        ← 97/100
│       ├── test.md          ← 96/100
│       └── troubleshoot.md  ← 90/100
├── config/
│   └── bootstrap/
│       └── assets/
│           └── claude/
│               └── commands/
│                   ├── airis-capability-router.md ← 98/100
│                   └── airis-research-first.md    ← 78/100（最低）
├── skills/
│   ├── .claude-plugin/
│   │   └── plugin.json      ← 100/100（完美）
│   └── skills/
│       ├── mcp-database/SKILL.md    ← 83/100
│       ├── mcp-debugging/SKILL.md   ← 85/100（有外部依赖声明问题）
│       ├── mcp-implementation/SKILL.md ← 85/100
│       └── mcp-research/SKILL.md    ← 85/100
├── install.sh               ← CRITICAL: curl|bash
├── scripts/
│   └── quick-install.sh     ← CRITICAL: curl|bash
├── apps/                    ← 主应用代码
└── ops/autostart/           ← macOS/Linux 自动启动配置
```

- **文件类型分布**：5 command / 4 skill / 1 manifest / 大量 shell/Python 脚本（系统侧）
- **编排关系**：命令通过 `task *` 调用网关底层功能（TaskFile 系统），skill 作为参考知识；`mcp-debugging/SKILL.md` 要求先调用外部 `superpowers:systematic-debugging`（未声明的外部依赖）
- **跨件契约**：plugin.json 100/100，manifest 完整；命令-skill 引用通过 Taskfile 而非直接 agent 调用，架构边界清晰

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「MCP 网关 + 统一运维接口」——将底层系统操作（Docker 启动、MCP 健康检查）封装成 Claude Code 命令，让 AI 辅助 DevOps 运维
- **解决什么问题**：MCP 生态中服务器配置复杂、调试困难、多服务器状态难以追踪的痛点
- **Trade-off**：将安全关键操作（安装、引导）交给 shell 脚本，AI 层（Claude Code 命令）保持高质量（90/100），但安装层存在严重安全隐患；换言之，「用好了」的部分做得很好，「第一次装」的部分做得很差
- **认知模型**：作者将 Claude Code 插件视为「运维助手」，命令都是查询/诊断导向（status/test/troubleshoot），而非变更导向——这是一个谨慎、只读的使用哲学，体现在 `allowed-tools` 声明上

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「轻量 NL 命令 + 重量 TaskFile 后端」**

Claude Code 命令只做查询和路由，真正的操作（启动服务、检查健康、调试连接）由 Taskfile 系统执行；NL 层专注于用户交互和结果解读。

模式特征清单：
- 特征 1：所有破坏性操作在 Taskfile 后端，NL 命令只触发 `task *` 调用
- 特征 2：命令声明了正确的 `allowed-tools: [Bash(task *)]`，精确限制了权限范围
- 特征 3：skill 层描述的是「如何在这个网关环境下使用 MCP」，不是通用知识
- 特征 4：有完整的 `capability-router` 命令，智能路由用户请求到正确工具

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| DevOps 运维辅助 | ✅ 高度适用 | TaskFile + Claude Code 命令的结合天然适合运维任务 |
| 面向终端用户的安装工具 | ❌ 不适用 | curl\|bash 安装方式对普通用户是安全隐患 |
| 纯 NL 知识管理 | ❌ 不适用 | 本仓库的命令高度依赖底层系统，无系统时无意义 |
| 安全敏感环境 | ❌ 当前不适用 | CRITICAL curl\|bash 未修复，无法在安全规范严格的环境使用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 命令 + TaskFile 后端（本仓库） | airis-mcp-gateway | 职责分离，命令简洁，权限精确 | 依赖 TaskFile 生态，移植成本高 |
| 纯 NL 命令（自包含） | aaronmaturen/claude-plugin | 零系统依赖，即开即用 | 性能受限，无法做系统级操作 |
| NL 表皮 + 原生二进制 | agent-sh/enhance | 高性能计算 | 二进制分发和安全验证复杂 |

### 2.4 改进空间
1. **当前问题**：`install.sh` 使用 `curl | bash`，无完整性验证 **改进做法**：提供 checksummed 安装方式作为主要推荐（`curl -fsSL .../install.sh -o install.sh && sha256sum -c install.sh.sha256 && bash install.sh`），保留 `curl | bash` 作为次选并标注风险 **预期收益**：消除 3 个 Critical 安全发现，允许 NLPM contribution 进行
2. **当前问题**：4 个 skills 均无 examples **改进做法**：为每个 skill 添加一个「真实查询场景」的 step-by-step 示例 **预期收益**：skill 分数从 83-85 提升到 95+，也让用户更快理解如何使用
3. **当前问题**：`mcp-debugging/SKILL.md` 声明了对外部 `superpowers` 插件的依赖，但未在 plugin.json 声明 **改进做法**：在 plugin.json 的 `dependencies` 字段声明 `superpowers`，或在 skill 中添加 `superpowers` 未安装时的降级路径 **预期收益**：无 superpowers 的用户收到明确报错而非损坏的工作流

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 得分 **90/100**，是本批次 4 个案例中最高分。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| config/.../airis-research-first.md | 78 | 无输出格式（-10）；规则未编号（-10） |
| skills/mcp-database/SKILL.md | 83 | 无 examples（-15）；"appropriate caution"（-2） |
| skills/mcp-debugging/SKILL.md | 85 | 无 examples（-15） |
| .claude/commands/troubleshoot.md | 90 | 无 `$ARGUMENTS` 守卫（-10） |
| .claude/commands/test.md | 96 | "some servers" 模糊（-2）；"actionable" 模糊（-2） |
| skills/.claude-plugin/plugin.json | 100 | 完美 |

### 3.2 当时值得借鉴的模式
1. **精确 `allowed-tools` 声明** → `status.md` 声明 `Bash(task *)` 而非 `Bash(*)`，精确限制命令只能执行 task 子命令。这是权限最小化原则的体现，值得学习。我的 bureau 命令目前无 allowed-tools，是明显改进空间
2. **`capability-router` 智能路由命令** → 根据用户描述的意图，路由到正确的 MCP 服务器或工具——相当于 NL 层的 API Gateway。这个模式可以推广到任何多工具并存的场景
3. **CLAUDE.md 的 token 节省设计** → CLAUDE.md 文档了「服务器能力 cache 化，每次 `initialize` 节省 token」——将系统优化思路直接写进 AI 可读的配置，让 AI 执行操作时自然考虑效率

### 3.3 当时的缺陷
1. **3 个 Critical curl|bash 安全问题** → 根本原因：`curl | bash` 是最方便的安装方式，作者优先考虑了安装体验，忽视了供应链安全。GitHub raw URL 上的 shell 脚本在 CDN 缓存或 DNS 劫持下会被替换——对于 151 star 的项目，这是真实攻击面。**自查：我的 gstack/bureau 是否有远程安装脚本？确认不存在。**
2. **4 个 skills 零 examples** → 根本原因：作者在写 skill 时聚焦「知识文档化」而非「行为示例化」；skill 描述了 what 和 how，但没有 before/after 的具体实例。结果：AI 读完 skill 知道「要做什么」，但不知道「做好了长什么样」。
3. **`mcp-debugging/SKILL.md` 未声明外部依赖** → 根本原因：`superpowers` 插件是作者自己常用的，他忘记了其他用户可能没有安装。在 NL 编程中，「假设环境」是常见错误——每个外部依赖都应该有 fallback 路径。

### 3.4 当时的优化机会
1. 将 `install.sh` 的 curl|bash 替换为 checksummed 安装（CRITICAL → 清除，解锁 contribution 通道）
2. 为 4 个 skills 添加 examples（从 83-85 提升到 95+，最低投入/最高回报）
3. 修复 `airis-research-first.md` 的规则编号和输出格式（78 → 95+）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| install.sh Critical curl\|bash | `grep -n "curl.*bash" install.sh` | **未修复**：第 6 行和第 266 行仍有 `curl \| bash` 文档和调用 | 3 个月后 CRITICAL 安全问题未解决；可能因 NLPM security gate 阻止了 contribution，维护者未收到 PR |
| scripts/airis-gateway 脚本 shell 注入 | `ls scripts/` | **架构变化**：`scripts/airis-gateway` 文件已不存在，被 `scripts/airis-mcp-gateway` 替代 | 原问题文件消失，但无法确认新脚本是否有同类问题 |
| AIRIS_VERSION 默认 latest | `grep "AIRIS_VERSION" install.sh` | **未修复**：`VERSION="${AIRIS_VERSION:-latest}"` 第 11 行仍在 | 未固定版本的安装风险依然存在 |
| 4 个 skills 无 examples | `grep -l "## Example" skills/skills/*.md` | **未修复**：0 个 skills 有 examples 块 | 3 个月无改进；维护者可能不知道 NLPM audit 建议 |

### 4.2 架构演进
`scripts/airis-gateway` 更名为 `scripts/airis-mcp-gateway`（命名与项目名对齐）。新增 `tests/` 目录（`test-compose-references.sh` 等），显示维护者在加强测试覆盖。`apps/` 目录保持多服务架构（gateway-control + airis-commands）。核心 NL 工件（commands/skills）无重大变化。

### 4.3 新增的可学习模式
新增的 `tests/test-compose-references.sh` 等测试脚本模式值得关注：对 shell 配置文件（compose.yaml）做交叉引用测试，确保所有引用在实际文件系统中有对应文件——这是「配置契约测试」（与 NLPM `bin/nlpm-check` 的思路类似），可以延伸到 NL 工件层：命令引用的 skill 是否都存在？

---

## 五、校准

### 5.1 我已经在做对的
1. **我的项目无 curl|bash 安装脚本** → bureau/gstack/graphify 均无分发安装脚本，不存在此类 Critical 风险
2. **bureau 的命令有精确的 `argument-hint`** → 类似于本案例的 `troubleshoot.md`；但我缺少 `allowed-tools`
3. **bureau skills 有 examples** → 本案例 4 个 skills 零 examples，我的 skills 均有 3 个——在这维度明显更好

### 5.2 挑战 / 验证
本案例确认了「安全问题是 NLPM contribution 的硬门槛」的实际效果：airis-mcp-gateway 有 3 个 Critical，导致 NLPM 完全不提 PR，维护者因此永远没有收到「你的 4 个 skills 缺少 examples」这个质量建议。**安全问题不只是安全风险，它还阻断了整个质量反馈循环。**

---

## 六、行动

### 6.1 自查动作

```bash
# 确认我的仓库无 curl|bash 安装模式（排查安全风险）
grep -rn "curl.*|.*bash\|curl.*|.*sh" /tmp/my-repos/ 2>/dev/null | grep -v ".git" | grep -v "test\|spec\|doc"
# 命中后：替换为 checksummed 安装方案或直接提供仓库地址让用户 git clone

# 检查我的 skills 是否有未声明的外部依赖
grep -rn "invoke\|call\|use.*:" /tmp/my-repos/*/skills/*/SKILL.md 2>/dev/null | grep -v ".git"
# 命中后：对每个外部 skill/plugin 引用，在 SKILL.md 中添加「如果未安装：降级做法如下」

# 检查我的命令是否有精确的 allowed-tools（权限最小化）
grep -rn "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
# 命中后：结果应非空；若为空，则补充 allowed-tools 声明
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 命令添加 `allowed-tools` 声明（借鉴本案例的权限最小化设计）
   - **为何可行**：bureau 命令功能明确，所需工具集可以精确枚举（lint 只需 Read+Grep+Glob）
   - **第一步**：逐一读取每个 bureau command 的 body，列出实际用到的工具，然后在 frontmatter 中补充 `allowed-tools: [xxx]`，约 1 小时

2. **想法**：为 gstack 添加配置契约测试（借鉴本案例的 test-compose-references.sh 思路）
   - **为何可行**：gstack 的 skills 之间有引用关系（land-and-deploy 依赖 setup-deploy），可以写一个 shell 脚本验证引用完整性
   - **第一步**：`grep -rn "skills:\|commands:" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md`，找出所有跨件引用，编写验证脚本，30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例核心目的**：MCP 服务器网关管理 + Claude Code 运维接口

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 同为 Claude Code 插件，有 skill + command | bureau 是知识管理，本案例是系统运维 | 中（命令设计模式可借鉴） |
| MarkQWu/graphify | 低 | 同为知识图谱相关 | graphify 是纯代码工具，无 Claude Code NL 接口 | 低 |
| MarkQWu/gstack | 低 | 同为技术工具 | gstack 无 MCP 管理需求 | 低 |

若全部「无相似」：我的仓库中无 MCP 网关类项目，本节主要做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skills 无 examples | `grep -L "## Example" */skills/*/SKILL.md` | **未命中**：bureau skills 均有 examples；gstack skills 待检查 | 需检查 gstack |
| 命令无 `allowed-tools` | `grep -L "allowed-tools" */commands/*.md` | **命中**：bureau 全部命令无 allowed-tools | 中 |
| 外部依赖未声明降级路径 | `grep -rn "invoke\|:systematic" */skills/*/SKILL.md` | **未命中**：我的 skills 无此类外部 plugin 引用 | 暂不适用 |

**具体行动建议**：
- `MarkQWu-bureau/commands/lint.md` → 添加 `allowed-tools: [Read, Grep, Glob, Bash(node *)]` → 5 分钟
- 检查 gstack skills 中是否有 examples（`grep -L "## Example" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md`）→ 如无，批量添加示例 → 1-2 小时

### 7.3 别人的更优方案

1. **领域**：精确 `allowed-tools` 声明（最小权限原则）
   - **本案例做法**：`status.md` 的 `allowed-tools: [Bash(task *)]` 精确到子命令前缀
   - **我的项目现状**：bureau commands 完全无 `allowed-tools` 声明
   - **如何借鉴**：为每个 bureau command 列举实际调用的工具，按 `Bash(node *)` 格式精确声明

2. **领域**：`capability-router` 路由命令模式
   - **本案例做法**：一个专用命令根据用户意图路由到合适工具，减少用户对内部架构的了解需求
   - **我的项目现状**：bureau 无此类入口路由命令
   - **如何借鉴**：考虑为 bureau 添加 `bureau:help` 路由命令，根据用户描述的问题推荐应该运行哪个子命令

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 示例覆盖率
- **我的做法**：bureau skills 每个有 3 个 examples 块，可被 NLPM tester 评估
- **本案例做法**（弱在哪）：4 个 skills 全部零 examples，audit 后 3 个月无改进
- **意义**：我的 skills 在可测试性和可信度上均优于本案例；如果进行 NLPM audit，skill 层会是明显加分项

---

## 八、术语表

### <a name="curl-pipe-bash"></a>curl-pipe-bash
> `curl -fsSL URL | bash` 这种安装模式：从网上下载 shell 脚本，直接通过管道传给 bash 执行，不经过任何本地检查。方便是方便，但安全隐患极大——如果网络在途中被篡改（CDN 被攻击、DNS 被劫持），用户实际执行的脚本可能是攻击者的恶意代码。安全的替代方案是：先 `curl -o script.sh`，再 `sha256sum -c script.sh.sha256`（校验哈希），验证通过后才 `bash script.sh`。

### <a name="TaskFile"></a>TaskFile
> 类似 Makefile 的任务运行器，用 `Taskfile.yml` 定义一系列命名任务（`task build`、`task test`、`task status`）。本案例中 Claude Code 命令通过调用 `task *` 来触发系统操作，这样 NL 层（Markdown 命令）与系统层（shell 操作）实现了清晰分离。

### <a name="权限最小化"></a>权限最小化
> Principle of Least Privilege：每个组件只拥有完成其任务所需的最小权限集。在 Claude Code 中体现为 `allowed-tools: [Bash(task *)]` 这样的精确声明——命令只被允许执行 `task` 开头的 Bash 命令，不能随意读取文件或调用其他工具。这样做的好处是：即使命令被注入恶意指令，也无法超出声明的工具范围。

### <a name="配置契约测试"></a>配置契约测试
> 验证配置文件中的引用都指向实际存在的资源。类似单元测试，但测试对象是「某 YAML 引用了某个服务名，这个服务名在 compose.yaml 里有定义」而非「某函数返回了期望值」。本案例的 `test-compose-references.sh` 就是做这种测试的脚本——NLPM 的 `bin/nlpm-check` 也有类似功能，检查 plugin.json 中注册的命令路径是否都存在对应文件。
