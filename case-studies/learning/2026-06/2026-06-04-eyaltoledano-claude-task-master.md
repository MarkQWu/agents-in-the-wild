# eyaltoledano/claude-task-master — 学习案例

**仓库**：https://github.com/eyaltoledano/claude-task-master
**Stars**：26807 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-04（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `vague-quantifier`, `curl-pipe-bash-risk`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-task-master（简称 TaskMaster）是一个基于 AI 的任务管理系统，通过 MCP 服务器 + Claude Code 插件双路径提供任务拆解、分配和追踪功能。拥有 26807 星，是 Claude Code 生态内下载量最高的第三方插件之一。核心价值：把一个大型开发需求自动拆解为有依赖关系的子任务树，并驱动 Claude 按序执行。

关键事实：
- 采用三智能体协作模型：`task-orchestrator`（拆解）→ `task-executor`（执行）→ `task-checker`（验证）
- 47 个 slash commands 覆盖任务 CRUD 的全生命周期
- [MCP 服务器](#MCP)提供持久化任务存储，Claude Code 插件提供 slash commands 触发界面
- 安全扫描结果：**BLOCKED**——两个 Critical 级别安全发现，audit 后至今未修复

### 1.2 架构剖析

**目录结构**（关键部分）：
```
packages/claude-code-plugin/
  .claude-plugin/plugin.json          # 插件清单（缺 version 字段）
  commands/                           # 47 个 slash commands（全部缺 name frontmatter）
    init-project.md, parse-prd.md, add-task.md, next-task.md ... (47 files)
  agents/
    task-orchestrator.md              # 主编排智能体（89/100）
    task-executor.md                  # 执行智能体（92/100）
    task-checker.md                   # 验证智能体（96/100）
scripts/
  modules/prompt-manager.js           # 包含 new Function() 安全漏洞
.claude/
  commands/                           # 本地命令（非插件分发）
    dedupe.md, go/ham.md, go/pr-comments.md
.mcp.json                             # MCP 服务器配置（task-master-ai 无版本锁定）
```

**文件类型分布**：3 个 agent / 47 个 command / 1 个 plugin.json / 55 个脚本文件

**编排关系**：task-orchestrator 拆解需求为任务树 → task-executor 执行单个任务 → task-checker 验证结果并决定是否迭代。是明确的分层 Orchestrator-Executor-Checker 三层架构，task-checker 通过 MCP 工具 `mcp__task-master-ai__get_task` 访问任务数据。

**跨件契约**：task-checker 依赖 `.mcp.json` 中配置的 MCP 服务器，如果 MCP 未启动，task-checker 的工具调用会静默失败。`learn.md` command 里的跨引用使用了错误的命名空间（`/project:task-master:*` 而非 `/taskmaster:*`），导致 6 个命令引用全部失效。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「任务即数据，AI 即驱动」——任务用 JSON 结构存储（通过 MCP），AI 通过 slash commands 操作任务数据，实现「自然语言描述需求 → 结构化任务树 → 自动执行」的闭环。

**解决什么问题**：大型 AI 辅助开发项目中，Claude 的上下文窗口有限，无法同时记住所有待做事项；TaskMaster 把任务持久化到 MCP 服务器，让 Claude 每次只需处理「当前任务」，通过 `/taskmaster:next-task` 获取下一步。

**做了什么 trade-off**：功能宽度 vs NL 质量——47 个命令覆盖了任务管理的每一个细节操作（add-dependency、remove-subtask、to-deferred……），但代价是每个 command 的 [frontmatter](#frontmatter) 质量极低（全部缺 `name` 字段）。选择了「功能齐全」而非「规范齐全」。

**反映什么认知模型**：作者把 NL artifacts（command files）视为「配置文件」而不是「可被工具链审查的代码」——功能能跑就行，frontmatter 字段是次要的。这在快速迭代阶段合理，但忽视了工具链（NLPM scanner、plugin registry）对 frontmatter 完整性的依赖。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「MCP 存储 + NL 命令 + 多智能体编排」三层架构**：MCP 服务器负责状态持久化，slash commands 提供用户触发界面，三个专职 agent 分别负责拆解、执行和验证——职责严格分离。

模式特征清单：
- 特征 1：状态持久化在 MCP 服务器（不在文件系统/对话上下文）
- 特征 2：Orchestrator→Executor→Checker 三层 agent，每层职责单一
- 特征 3：用户通过 slash commands 触发工作流，不需要理解内部 agent 调用关系
- 特征 4：commands 数量多（≥20），覆盖任务对象的完整 CRUD 操作
- 特征 5：MCP 和 Claude Code 插件并存（双渠道访问同一数据）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要跨会话持久化状态的工作流（任务/issue/笔记） | ✅ 高度适用 | MCP 提供持久化，优于文件系统方案 |
| 一次性任务（生成文档、代码审查等无需持久状态） | ❌ 不适用 | MCP 依赖引入不必要的运行时复杂度 |
| 团队协作场景（多人共享任务列表） | ✅ 适用 | MCP 服务器可共享，slash commands 是统一接口 |
| 高安全要求环境 | ❌ 不适用 | `.mcp.json` 的 `npx -y` 无版本锁定是供应链风险 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP + NL 命令 + 多 Agent（本仓库） | eyaltoledano/claude-task-master | 状态持久、工作流完整 | 安全漏洞多；frontmatter 质量差；运行依赖 MCP 服务 |
| 纯文件持久化 + 少量命令 | MarkQWu/echo-sleuth-for-claude | 零外部依赖，可移植 | 状态散落文件系统，跨 session 查找麻烦 |
| 无持久化，纯会话内任务管理 | 大多数 agent 插件 | 零配置 | 上下文超限后任务丢失 |

### 2.4 改进空间

1. **当前问题**：47 个 command 全部缺 `name` frontmatter **改进做法**：写一个 shell 脚本遍历所有 command 文件，从文件名提取 name 并自动追加 frontmatter（20分钟一劳永逸） **预期收益**：plugin registry 能正确注册所有命令；NLPM 分数从 67 → ~75

2. **当前问题**：`new Function()` 执行动态代码字符串（Critical 安全漏洞）**改进做法**：重构 `scripts/modules/prompt-manager.js:280`，用静态条件映射表替换动态代码评估（把 condition 字符串改成枚举键值对） **预期收益**：消除代码注入风险，这是 BlockED 状态的直接原因

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 67/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| 47 个 command 文件 | 60–70 | 无 name frontmatter；部分缺 allowed-tools、output format |
| plugin.json | 75 | 缺 `version` 字段 |
| CLAUDE.md | 78 | 项目说明文档，非 NL artifact |
| task-orchestrator.md | 89 | 部分 output format 不完整；模糊量词 |
| task-executor.md | 92 | output format 定义模糊 |
| task-checker.md | 96 | 轻微模糊量词 |

### 3.2 当时值得借鉴的模式

1. **Orchestrator-Executor-Checker 三层 agent 分工** → 每个 agent 做且只做一件事：拆（orchestrator）、执（executor）、验（checker）→ 任何有「分解→执行→验证」循环的工作流都可以套用这个模式 → `agents/task-orchestrator.md`、`agents/task-executor.md`、`agents/task-checker.md`

2. **`task-checker.md` 作为质量门** → checker 在 executor 完成后评估结果，决定是继续还是迭代；把「质量验证」显式化为一个独立角色，而不是让 executor 自我评估（自我评估有偏差） → `agents/task-checker.md`

3. **MCP 服务器 + Claude 插件双渠道** → 同一套任务数据可以通过 MCP API 也可以通过 Claude slash commands 访问，适配不同的集成场景

4. **47 个命令覆盖完整任务 CRUD** → 从「初始化项目」到「移动到不同状态」（to-done、to-deferred、to-in-progress）都有专门命令，用户不需要手写任何操作任务的指令

### 3.3 当时的缺陷

1. **`new Function()` 执行动态代码（Critical 安全漏洞）** → `prompt-manager.js:280` 把模板条件字符串直接传入 `new Function()` 执行，等同于 eval()；如果 tasks.json 里的 condition 字段被恶意用户控制，就可以在 Claude Code 进程里执行任意代码 → 根本原因：作者用「JS 动态表达式」实现条件模板，选择了最简单但最危险的方案 → **自查：我的项目里是否有类似的动态代码执行？** echo-sleuth 全是 NL，无此风险；claude-for-legal 无脚本

2. **`install-taskmaster.md` 包含 `curl | bash` 指令（Critical 安全漏洞）** → 当用户执行 `/taskmaster:install-taskmaster` 时，Claude 会被指示运行 `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/.../install.sh | bash`；如果 GitHub CDN 或 install.sh 被篡改，就会执行恶意代码 → 根本原因：作者把「安装指南」（面向人类执行）直接放进了「可被 Claude 自动执行」的 command 里，混淆了两种文档的用途 → **自查：我的 command 里有没有包含 curl|bash 或 sudo 指令？** 没有。

3. **47 个命令全部缺 `name` frontmatter** → plugin registry 只能靠文件名推断命令名，没有正式注册 → 根本原因：前期快速开发时没有建立 frontmatter 标准，后期 47 个文件一次性补全代价很高，所以一直拖 → **自查：echo-sleuth 8 个命令全部有 name，是亮点。**

### 3.4 当时的优化机会

1. **Critical 优先**：修复 prompt-manager.js 的 `new Function()` → 用 switch-case 或条件映射表替代动态代码执行
2. **Critical 优先**：修改 install-taskmaster.md，把 `curl|bash` 指令改为「建议用户手动执行」的警告文本（不作为 Claude 自动执行的指令）
3. **NL 规范**：一次性给所有 47 个 command 补全 `name` frontmatter（脚本化，20分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `new Function()` 动态代码执行（Critical） | `grep -n "new Function" scripts/modules/prompt-manager.js` | **仍存在**（L280） | Audit 后 2 个月 Critical 漏洞未修，说明作者选择接受这个风险 |
| `curl \| bash` 在 install-taskmaster.md（Critical） | `grep -n "curl.*bash" packages/claude-code-plugin/commands/install-taskmaster.md` | **仍存在**（L95） | 同上；用户每次运行 `/taskmaster:install-taskmaster` 时风险仍在 |
| plugin.json 缺 `version` 字段（Bug #1） | `cat packages/claude-code-plugin/.claude-plugin/plugin.json` | **仍存在**（无 `version` 字段） | 简单一行修复都未完成，说明这个 repo 对 NLPM 报告响应度很低 |

### 4.2 架构演进

Audit 时（2026-04-06）已存在的 47 命令 + 3 个 agent 结构基本未变。值得注意的变化：
- 仓库根目录出现了 `manifest.json`、`CLAUDE_CODE_PLUGIN.md` 等文件，说明 TaskMaster 在积极适配不同平台（Cursor、Claude Code、其他 IDE）
- `turbo.json`、`tsdown.config.ts` 等构建工具配置新增，说明项目正在从「脚本 + Markdown」往「monorepo 构建工程」演进
- `apps/` 目录出现，说明 TaskMaster 在拓展为一个更大的生态系统（不只是 Claude Code 插件）

这说明作者意识到：TaskMaster 的未来是「多平台任务管理基础设施」，Claude Code 插件只是入口之一。

### 4.3 新增的可学习模式

`manifest.json`（根目录）出现：与 `.claude-plugin/plugin.json` 并存，说明 TaskMaster 在往「平台无关的 manifest」方向演进——一份 manifest 能让不同的 AI IDE 发现这个工具。这是「跨 IDE 可移植性」的新兴模式，值得关注。

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth 所有命令都有 `name` frontmatter**：TaskMaster 的 47 个命令全部缺这个字段，而我的 8 个命令全部有——这是正向对比
2. **不在 command 文件里包含 curl|bash 或 sudo 指令**：我的 command 文件只有 NL 指令，没有危险的 shell 命令嵌入
3. **三层 agent 模式有意识地在 echo-sleuth 里部分实现**：recall（查询）→ analyze（分析）→ prune（修改）大致对应 TaskMaster 的 Executor→Checker 两层

### 5.2 挑战 / 验证

这次案例挑战了「26000 星的项目一定质量可靠」的假设——TaskMaster 的 NL Score 只有 67/100，且有 2 个 Critical 安全漏洞 2 个月未修。

验证了：**用户星数 ≠ NL artifact 质量**。用户数量反映功能价值，NLPM score 反映 AI 执行可靠性。高星数项目甚至更可能「快速发展、NL 规范跑后」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否包含危险的 shell 命令
grep -rn "curl.*bash\|curl.*sh\|sudo\|eval\|new Function" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/ \
  /tmp/my-repos/MarkQWu-claude-for-legal 2>/dev/null | grep ".md" | head -10
```
命中后：把 command 里的 shell 命令改为「提示用户手动执行」的说明文字，而不是 Claude 可以自动执行的步骤。

```bash
# 检查我的 command 是否全部有 allowed-tools
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands -name "*.md" \
  | xargs grep -L "allowed-tools" 2>/dev/null
```
命中后：为每个命令补充 `allowed-tools`（对照命令体里用到的工具：Read、Write、Bash 等）。

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 里实现一个轻量的 Orchestrator→Executor 两层模式，让 `recall` command 作为 orchestrator 分派给 `file-historian` 或 `schema-scout` agent  
   **为何可行**：当前 echo-sleuth 的 recall command 和 recall agent 是分开的，已经具备这个结构  
   **第一步**：在 `commands/recall.md` 里明确写 "如果查询需要深度 git 分析，dispatch file-historian agent"（10分钟）

2. **想法**：给 echo-sleuth 建立一个 `quality-checker.md` agent，在 analyze/recall 完成后验证输出是否达标  
   **为何可行**：TaskMaster 的 task-checker 模式直接可借用  
   **第一步**：用 task-checker.md 为模板，写一个评估「经验提取结果是否有操作建议、是否有时间参考、是否有来源文件」的 checker（30分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 eyaltoledano/claude-task-master 的核心目的**：通过 MCP 持久化任务树，驱动 Claude 按序完成大型开发项目中的多步骤工作
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都有多命令 + 多 agent 协作；都有跨会话状态跟踪的需求 | TaskMaster 是任务管理（面向未来）；echo-sleuth 是历史挖掘（面向过去） | 高（Agent 架构参考） |
| MarkQWu/claude-for-legal | 低 | 都是多 plugin 结构 | 法律工作流无需任务状态机 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 `allowed-tools` | `find commands -name "*.md" \| xargs grep -L "allowed-tools"` | echo-sleuth 命中 4/8（recap、timeline、recall、lessons） | 中 |
| plugin.json 缺 `version` 字段 | `cat .claude-plugin/plugin.json \| grep version` | 需手动验证（echo-sleuth 的 plugin.json 未在本次检查范围内） | 低 |

**命中后的具体行动建议**：
- `echo-sleuth/commands/recall.md` → 加 `allowed-tools: [Read]`，因为 recall 只读 `.jsonl` 文件
- `echo-sleuth/commands/lessons.md` → 加 `allowed-tools: [Read, Write]`，因为 lessons 可能写入 MEMORY.md

### 8.3 别人的更优方案

1. **领域**：Orchestrator-Executor-Checker 三层 agent 架构
   - **本案例做法**：三个明确分工的 agent（orchestrator 拆解、executor 执行、checker 验证），agent 之间有明确的调用关系和触发条件
   - **我的项目现状**：echo-sleuth 有 5 个 agents（memory-auditor、file-historian、schema-scout、recall、analyze），但没有明确的层级结构，每个 agent 相对独立
   - **如何借鉴**：在 echo-sleuth CLAUDE.md 里为 5 个 agent 画出调用关系图，明确哪个是 orchestrator（recall/analyze），哪个是 executor（file-historian、schema-scout），哪个是 checker（memory-auditor）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Command frontmatter 规范性
- **我的做法**：echo-sleuth 8 个命令全部有 `name` frontmatter，4 个有 `allowed-tools`（50% 覆盖率）
- **本案例做法**（弱在哪）：47 个命令全部缺 `name`，全部缺 `allowed-tools`（0% 覆盖率）
- **意义**：在 Claude Code 生态里，command 的 frontmatter 完整性直接影响 plugin registry 的注册质量；我的 echo-sleuth 在这一点上比 26807 星的 TaskMaster 更规范

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置，声明 command 的 `name`、`description`、`allowed-tools` 等元数据。缺少 `name` 字段时，Claude Code 只能用文件名推断命令名，不会正式注册。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 定义的工具通信协议。MCP 服务器是一个独立运行的进程，通过标准化接口向 Claude 提供「工具」（读写数据库、调 API 等）。TaskMaster 用 MCP 服务器存储任务数据，Claude 通过 `mcp__task-master-ai__get_task` 等工具读写任务，实现跨会话的状态持久化。

### <a name="eval"></a>eval / new Function
> JavaScript 里两种把「字符串」当作「代码」执行的方式。`eval("1+1")` 和 `new Function("return 1+1")()` 都会执行传入的字符串作为 JS 代码。危险在于：如果字符串来自用户输入或外部文件，攻击者可以构造恶意代码字符串，实现任意代码执行。

### <a name="monorepo"></a>monorepo
> 把多个相关项目（包、应用）放在同一个 Git 仓库里管理的做法。TaskMaster 用 `turbo.json` 管理 monorepo，包含 `packages/claude-code-plugin`（NL 插件）、`mcp-server`（MCP 服务端）等多个子包。
