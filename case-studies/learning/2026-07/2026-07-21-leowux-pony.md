# leowux/pony — 学习案例

**仓库**：https://github.com/leowux/pony
**Stars**：1（exemplar_published=true 覆盖星数门槛）| **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-21（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `router-channels`, `fallback-chain`, `model-pinning`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[pony](#pony-tool) 是阿里巴巴内部（Alipay）开发者 wuwenbang 发布到 GitHub 的轻量级任务管理 CLI + Claude Code 插件。核心功能：文件存储的任务追踪（带依赖关系和状态流转）+ 实时 HUD（Heads-Up Display）侧边栏渲染 + 四代理编排流水线（planner → explorer → executor → verifier）。

关键事实：
- 仓库根目录存在 `.orphaned_at` 文件（时间戳 `1775389404680` ≈ 2026-04），意味着该项目已被**标记为废弃/孤儿状态**，可能已被合并入 Alipay 内部工具或停止维护
- 即便如此，所有 NL 工件在 HEAD 中保持完整，是学习「NL 表皮 + 原生二进制核心」架构的绝佳标本
- 内部注册表指向 Alipay 私有 npm registry（`registry.antgroup-inc.cn`），外部用户无法直接 `npm install`
- 有 TypeScript 源码（`src/`）和完整的构建配置（vite、oxlint、Vitest），是真正的「开源可运行」工具

### 1.2 架构剖析
```
pony/
├── agents/                     # 4 个 NL 代理
│   ├── planner.md              # 任务规划（分解 feature → tasks）
│   ├── explorer.md             # 代码探索（定位相关文件/函数）
│   ├── executor.md             # 代码执行（实现具体 task）
│   └── verifier.md             # 验证（检查实现是否满足需求）
├── commands/                   # 4 个 slash commands（用户入口）
│   ├── run.md                  # 编排循环：循环取任务 → 路由 → 代理
│   ├── task.md                 # 任务 CRUD 操作
│   ├── hud.md                  # HUD 管理（启动/停止/配置）
│   └── init.md                 # 初始化 pony 工作区
├── src/                        # TypeScript 原生核心
│   ├── tasks/                  # 任务 CRUD + 文件操作
│   ├── todos/                  # Todo 追踪
│   ├── hud/                    # 实时渲染引擎（elements/ + render.ts）
│   └── cli/                    # CLI 入口
├── .claude-plugin/
│   └── plugin.json             # NL 层注册（版本 0.0.2，repository 指向内部 git）
├── .claude-plugin.json         # 根级插件配置（指向 commands/ 和 agents/）
└── package.json                # npm 配置（指向 Alipay 私有 registry）
```

- **文件类型分布**：4 commands + 4 agents + TypeScript 源码 + 2 个插件配置文件
- **编排关系**：`commands/run.md` 是主编排器，通过任务 tag 路由到 4 个 agents 之一；agents 内部使用 `pony` CLI 命令（由 TypeScript 核心实现）更新任务状态
- **跨件契约**：NL 层（commands/）通过 CLI 接口调用原生层（`pony next --json`、`pony update <id> -s running`），任务状态机在 TypeScript 里维护，NL 层只触发状态变更

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」——Claude 的 NL 能力负责理解任务、路由代理、判断完成；任务数据持久化、状态流转、HUD 渲染等精确操作全部由 TypeScript CLI 处理
- **解决什么问题**：Claude Code 会话本身是无状态的——任务做到一半关掉，下次不知道从哪里继续。pony 用文件持久化 + CLI 状态机解决这个问题
- **做了什么 trade-off**：TypeScript 核心让数据操作精确可靠，但外部用户无法通过标准 npm install（指向私有 registry），这是开源与内部工具双轨的典型代价
- **反映什么认知模型**：作者认为 AI 善于「推理和路由」，不擅长「精确的状态管理」；把两者职责明确分开，各做各的强项

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「NL 表皮 + 原生二进制核心」（nl-binary-hybrid）

这个模式的核心是：Claude Code NL 工件（commands/agents）只负责理解用户意图、路由到正确代理、监控循环进度；所有需要精确执行的操作（文件 I/O、状态流转、渲染）都交给一个编译好的原生工具（CLI/binary）。

模式特征清单：
- **特征 1**：NL 层通过 CLI 接口（而非直接文件操作）与原生层通信，接口是稳定契约
- **特征 2**：状态机和数据结构完全在原生层定义，NL 层只知道状态名（`pending`/`running`/`completed`）
- **特征 3**：NL 层可以被替换（换一套 commands/agents），原生核心无需变动
- **特征 4**：原生层提供 JSON 输出（`pony next --json`），NL 层结构化消费，不做字符串解析
- **特征 5**：HUD 等「持续渲染」功能只有原生层能高效实现，NL 层不尝试实现

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要持久化状态 + 精确数据操作的工具 | ✅ 高度适用 | 任务管理、知识库、版本控制等 |
| 需要高性能实时渲染（HUD/statusline） | ✅ 适用 | TypeScript/Go 可实现 NL 层无法实现的帧率 |
| 团队内部工具（可控制 npm registry） | ✅ 适用 | 私有 registry 问题可控 |
| 面向公开用户、希望零依赖安装的工具 | ❌ 不适用 | 需要用户安装 CLI，摩擦高 |
| 纯 NL 交互、无需持久化的一次性任务 | ❌ 不适用 | 原生层是 overkill |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制核心（本仓库）| pony | 状态操作精确，HUD 可能，CLI 可单独测试 | 外部用户需安装 CLI，双栈维护成本 |
| 纯 NL（Claude 读写文件） | agent-sh/agentsys | 零依赖，任何地方可用 | 文件操作不精确，无法实现实时渲染 |
| MCP 后端 + NL 前端 | pe-menezes/fin-claude-plugin | 后端隔离，可独立部署 | 需要 MCP server 持续运行 |

### 2.4 改进空间

1. **当前问题**：`commands/run.md` 使用了 `Agent` 工具但未在 `allowed-tools` 里声明。**改进做法**：在 frontmatter 加 `allowed-tools: [Agent, Bash]`。**预期收益**：runtime enforcement 下不再报错，planner→executor→verifier 流水线可靠执行。

2. **当前问题**：4 个 agents 均无具体 invocation 示例（只有模板，没有真实 input/output 对）。**改进做法**：每个 agent 加一个「sample_input → sample_output」对。**预期收益**：新用户理解 agent 行为的时间从「读完整个 body」降至「看一个例子」。

3. **当前问题**：`package.json` 的 `publishConfig.registry` 指向 Alipay 私有 registry，外部用户 `npm install` 报错。**改进做法**：将公开的 GitHub 发布版本改为发布到 public npm registry，私有版本走 fork。**预期收益**：真正的开源可用。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时整体得分 **91/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/task.md | 75 | 无输出格式(-10)，无空输入处理(-10)，无 allowed-tools(-5) |
| commands/run.md | 83 | 无输出格式(-10)，无 allowed-tools(-5)，"appropriate"模糊(-2) |
| commands/hud.md | 85 | 无输出格式(-10)，无 allowed-tools(-5) |
| commands/init.md | 85 | 无输出格式(-10)，无 allowed-tools(-5) |
| agents/explorer.md | 93 | 无具体示例(-5)，"relevant"模糊(-2) |
| agents/executor.md | 95 | 无具体示例(-5) |
| agents/planner.md | 95 | 无具体示例(-5) |
| agents/verifier.md | 95 | 无具体示例(-5) |
| CLAUDE.md | 100 | 满分 |
| .claude-plugin/plugin.json | 100 | 满分 |

### 3.2 当时值得借鉴的模式

1. **四角色代理流水线（planner/explorer/executor/verifier）**：分工明确，每个代理专注一件事。为什么好：对复杂编程任务，先规划再探索再执行再验证，比「一个 agent 做所有事」出错率低。如何借鉴：在需要多步骤的任务里考虑 planner+executor+verifier 三角结构，不一定需要 explorer。

2. **Tag 路由表（task tags → agent 映射）**：`commands/run.md` 里有明确的路由表，tag → subagent_type，决策透明。为什么好：不需要 AI 推断「这个任务该交给谁」，用 tag 确定性路由，可调试。如何借鉴：在任何多代理编排里，先定义一个显式路由表，而非让主 agent 自由发挥。

3. **HUD 实时渲染（由原生层驱动）**：TypeScript 实现的 `pony hud` 在终端实时显示当前任务、token 消耗、skill 列表。为什么好：Claude Code 会话期间，用户始终知道当前状态，不迷失。如何借鉴：工具类插件值得投入一个实时状态显示机制，哪怕只是 bash 脚本定期 echo。

4. **CLAUDE.md 满分设计**：CLAUDE.md 内容清晰、完整、无模糊表达。为什么好：这是 AI 理解整个项目的第一个文件，质量影响所有后续操作。如何借鉴：每个项目的 CLAUDE.md 都要当成「AI 入职说明书」来写，不要省略关键命令和约定。

### 3.3 当时的缺陷

1. **Bug：`commands/run.md` 使用 Agent 但未声明 allowed-tools**：运行时 `Agent()` 调用会被 Claude Code 工具权限检查拦截，导致整个 planner→executor→verifier 流水线无法执行。根本原因：command 文件是按「描述应该做什么」的方式写的，而不是按「Claude 运行时的约束要求」写的——两种心智模型之间的脱节。自查：我的 commands 有没有类似的「用了工具但没声明」的情况？

2. **Commands 全部缺少输出格式声明**（每个 -10 分）：4 个 commands 都没有说「Claude 执行后应该输出什么」。根本原因：作者的注意力放在「怎么做」而非「做完输出什么」，但对 Claude 来说输出格式声明是它知道自己完成了的信号。自查：我的 commands 有没有输出格式声明？

3. **executor.md 声明了 TaskCreate/TaskUpdate/TaskList 但这些不是标准 Claude Code 工具名**：Claude Code 里实际工具是 `TodoWrite`/`TodoRead`，不是 `Task*`，会运行时静默失败。根本原因：作者把 Claude Code 的内部 tool API 名字记错了（或版本过时）。自查：我的 agents 有没有引用了不存在的工具名？

### 3.4 当时的优化机会

1. **Bug 修复（最高优先级）**：`commands/run.md` 加 `allowed-tools: [Agent, Bash]`，一行修复
2. **Commands 加 Output Format**：4 个 commands 各加一个「执行完后 Claude 应该输出什么」的描述（3-5 行）
3. **Agents 加具体示例**：4 个 agents 各加一个真实的 input → output 对

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug：run.md 使用 Agent 未声明 allowed-tools | `grep "allowed-tools" commands/run.md` | **持续**：命令文件存在，bug 原封不动 | .orphaned_at 存在，作者已停止维护 |
| Commands 缺输出格式 | `grep "## Output" commands/task.md` | **持续**：4 个 commands 均无输出格式 | 与 bug 一同被冻结 |
| executor.md 非标 Tool 名 | `grep "TaskCreate" agents/executor.md` | 无法验证（executor.md 里是 `<Tool_Usage>` 模板块，需实际查看） | 需要读完整文件才能判断 |

### 4.2 架构演进

`.orphaned_at` 文件说明这是 2026 年 4 月左右就停止维护的项目快照。实质性变化：**无**。仓库完全冻结在 audit 时的状态，这反而使它成为一个完美的「学习标本」——所有 audit 时发现的问题都还在，可以逐一核实。

新增的可观察现象：
- 根目录新增 `.orphaned_at` 时间戳文件和 `.oxfmtrc.json`（oxlint 配置），说明弃用前做过最后一次 lint 配置整理

### 4.3 新增的可学习模式

「有意识地归档」模式：`.orphaned_at` 文件本身是一个值得借鉴的做法——在废弃一个项目时，明确标记弃用时间而非直接删仓库，让历史快照可搜索可引用。

---

## 五、校准

### 5.1 我已经在做对的

1. **四角色分工思路**：bureau 的 `crew/auditor/agent.md` 使用了类似「read → analyze → write」分阶段的设计，与 pony 的 planner/explorer/executor/verifier 层次相似
2. **CLAUDE.md 优先级**：我的仓库都有结构良好的 CLAUDE.md，pony 得 100 分的 CLAUDE.md 做法我已经在践行
3. **CLI 接口作为 NL-Native 契约**：graphify 的 skill 通过 CLI 命令调用 Python 核心，与 pony 的 `pony next --json` 模式相同

### 5.2 挑战 / 验证

pony 验证了一个我一直知道但没有严格执行的原则：**命令文件里不只要写「做什么」，还要写「Claude 工具运行时约束」**。Tag 路由表写得多好都没用，如果 `allowed-tools` 缺失，整个流水线在运行时就会崩。这是「编写 NL 代码」和「编写实现说明」最大的区别。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 里有没有「使用了工具但没在 allowed-tools 声明」的情况
# 先找所有 command 文件里用到 Agent/Bash/Read 等工具的
grep -rn "Agent\|Bash\|Read\|Write\|Grep" /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/ --include="*.md" -l 2>/dev/null | head -20
# 然后对每个文件检查是否有 allowed-tools 声明
# 命中（有工具调用但无 allowed-tools）后：在 frontmatter 加 allowed-tools 行

# 检查我的 commands 有没有输出格式
find /tmp/my-repos/MarkQWu-bureau/commands -name "*.md" 2>/dev/null | \
  xargs grep -L "## Output\|output format\|## 输出" 2>/dev/null
# 命中后：加 3-5 行「执行完成后 Claude 的输出格式」

# 验证 agent 里引用的工具名是否是 Claude Code 实际支持的
grep -rn "TaskCreate\|TaskUpdate\|TaskList\|TodoCreate" \
  /tmp/my-repos/MarkQWu-gstack/ /tmp/my-repos/MarkQWu-bureau/ --include="*.md" 2>/dev/null
# 命中后：把 TaskCreate 改为 TodoWrite，TaskList 改为 TodoRead（Claude Code 实际 API）
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 里实现轻量版 planner+executor+verifier 流水线（不需要 HUD，但需要 task 状态追踪）
   - **为何可行**：bureau 已有 agent 层，缺的是「任务循环编排」command
   - **第一步**：写 `bureau/commands/run.md`，从 `BUREAU.md` 中的待办事项读取下一个任务，路由到对应 skill（1-2 小时）

2. **想法**：为 gstack 缺少 allowed-tools 的 commands 做一次系统性修复
   - **为何可行**：pony 案例清楚说明了这是最高优先级 bug，影响 runtime
   - **第一步**：列出所有 gstack command 文件，对每个检查 allowed-tools 声明，批量补充（30 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **leowux/pony 的核心目的**：Claude Code 会话内任务持久化 + 四代理编排流水线 + 实时 HUD

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都解决 AI 会话跨回合状态持久化；都有 agent 层做编排 | bureau 更通用（知识管理），pony 专注任务管理；bureau 无 HUD | **高** |
| MarkQWu/gstack | 中 | 都有多 skill 工具集；都通过 CLI 工具与系统交互 | gstack 偏工程操作，无任务状态追踪 | 中 |
| MarkQWu/graphify | 低 | 都有 CLI 作为 NL 与原生层的桥梁 | 领域不同（代码图谱 vs 任务管理） | 低 |
| MarkQWu/drama-workshop-skills | 无 | - | 领域完全不同 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 缺 allowed-tools 声明 | `find /tmp/my-repos/MarkQWu-bureau/commands -name "*.md" \| xargs grep -L "allowed-tools"` | bureau commands 目录下命令文件需逐一检查 | **高**（若有命中是 runtime bug） |
| Commands 缺输出格式声明 | `find /tmp/my-repos/MarkQWu-bureau/commands -name "*.md" \| xargs grep -L "## Output"` | 需检查 | 中 |
| Agent 零示例 | `find /tmp/my-repos/MarkQWu-bureau/crew -name "agent.md" \| xargs grep -L "example"` | bureau 的 auditor/agent.md 需确认 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的任何 command 文件若缺 `allowed-tools` → 立即加上，5 分钟
- `MarkQWu/bureau/crew/auditor/agent.md` 若无示例 → 加 1 个读入文件路径 → 输出摘要的 input/output 对，10 分钟

### 7.3 别人的更优方案

1. **领域**：任务路由表（tag → agent 映射）
   - **本案例做法**：`commands/run.md` 里有一个明确的表格，列出每种 task tag 对应的 agent 和 model
   - **我的项目现状**：`MarkQWu/bureau` 的 agent 调度是隐式的，主 agent 自己判断
   - **如何借鉴**：在 bureau 的 `crew/agent.md`（若有）里加一个「决策表」，明确列出哪类请求路由到哪个 skill

2. **领域**：CLI JSON 输出作为 NL 消费接口（`pony next --json`）
   - **本案例做法**：CLI 输出结构化 JSON，NL 层用 `--json` flag 消费，不做字符串解析
   - **我的项目现状**：`MarkQWu/graphify` 的 CLI 输出是人类可读格式，NL 层需要解析文本
   - **如何借鉴**：给 graphify CLI 加 `--json` flag 输出结构化结果，skill 里用 `| python3 -c "import json,sys; d=json.load(sys.stdin)"` 消费

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Agent 模型声明
- **我的做法**：`MarkQWu/bureau/crew/auditor/agent.md` 声明 `model: sonnet`
- **本案例做法**：pony 的 4 个 agents 均无 `model:` 声明，随 Claude Code 版本漂移
- **意义**：bureau 的 agent 行为在版本升级时更稳定可预测，这是值得在 pony 的 PR 里提的改进点（虽然 pony 已废弃）

---

## 八、术语表

### <a name="pony-tool"></a>pony
> wuwenbang 开发的任务管理 CLI 工具，同时也是一个 Claude Code 插件。「pony」是项目的内部代号。它的「任务」是 Claude Code 会话内部的工作单元（类似「子任务」），不是系统级的 TODO 应用。Pony CLI 在终端运行，维护任务的状态机（pending → running → completed）；Claude Code 插件部分（agents/commands）让 Claude 能够调用 pony CLI 并根据任务内容路由到合适的代理。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 Claude Code 插件设计模式：自然语言（NL）工件（agents/skills/commands）作为用户与 AI 的交互界面（「表皮」），而所有需要精确执行的操作（文件 I/O、状态管理、实时渲染）由一个编译好的原生程序（TypeScript 编译为 .js，或 Go/Rust 编译为二进制）完成（「核心」）。两者通过命令行接口（CLI）通信。pony 是典型代表：Claude 调用 `pony next --json` 获取下一个任务，状态机逻辑在 TypeScript 里保证精确性。

### <a name="HUD"></a>HUD（Heads-Up Display）
> 抬头显示器。借用战斗机飞行员「不低头看仪表板，用透明屏幕叠加信息」的概念。pony 的 HUD 是一个在终端侧边栏实时显示当前任务、token 消耗、可用 skill 列表的面板，让开发者在 Claude Code 会话进行时始终能「抬头看状态」，不需要把注意力移出对话窗口。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包围的 YAML 配置块。对 command 文件来说，`allowed-tools` 是关键字段——告诉 Claude Code「这个命令可以调用哪些工具」。缺了这个字段，Claude 在运行时想调用 `Agent()` 会被权限检查拦截，整个编排流水线崩溃。

### <a name="状态机"></a>状态机
> 描述一个事物可以处于哪些状态、从一个状态如何转到另一个状态的规则系统。pony 的任务状态机包含：`pending`（待处理）→ `running`（执行中）→ `completed`（已完成），以及可能的 `failed`、`blocked` 等状态。状态机保证了「每个任务的生命周期是可追踪的」，不会出现「做到一半消失了不知道哪里去了」的情况。
