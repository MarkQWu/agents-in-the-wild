# sangrokjung/claude-forge — 学习案例

**仓库**：https://github.com/sangrokjung/claude-forge
**Stars**：648 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-07-01（基于 audit 快照，目标仓库克隆受限）
**主题标签**：`security-gate`, `cross-reference`, `vague-quantifier`, `manifest-discipline`, `experience-accumulation`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
claude-forge 是一个以"DevSecOps 平台"为定位的 Claude Code 插件，作者 sangrokjung 将其打造为一套面向专业开发者的全生命周期工作流套件。该仓库具备以下关键特征：
- 规模较大：98 个 NL [工件](#工件)，其中 73 个被评分（41 个活跃 agent、9 个 rule 文件、25 个补充知识文件）
- 安全意识突出：拥有 5 个 PostToolUse/PreToolUse [hook](#hook) 脚本用于运行时防护（secret 过滤、命令守卫、速率限制、数据库守卫）
- 多形态集成：同时存在 `.mcp.json` 和 `mcp-servers.json` 两套 [MCP](#MCP) 配置，体现了多后端接入意图
- 核心约束：所有安全 hook 都通过 `OPENCLAW_SESSION_ID` 环境变量进行条件激活

### 1.2 架构剖析
- **目录结构**（推断自 audit 报告）：
```
claude-forge/
├── agents/          # 9 个专职 agent（verify-agent、database-reviewer、security-reviewer 等）
├── commands/        # 30+ 个命令，含 eval、plan、tdd、checkpoint 等
├── skills/          # 20+ 个 skill，含 continuous-learning-v2 子系统
├── rules/           # 9 个 .claude/rules 平文本规则（agents-v2、coding-style 等）
├── hooks/           # 6 个执行脚本（5 个安全 hook + 1 个 forge-update-check）
├── scripts/         # md-to-docx、pdf-enhance 工具脚本
├── .mcp.json        # MCP 服务器配置（4 个包）
├── mcp-servers.json # 备用 MCP 配置
└── .claude-plugin/plugin.json  # plugin manifest v2.2.0
```
- **文件类型分布**：41 个活跃 agent + 30+ 个 command + 20+ 个 skill + 9 个 rule + 6 个 hook 脚本
- **编排关系**：存在明确的 dispatcher 链路——`commands/handoff-verify.md` → `agents/verify-agent.md` → `agents/security-reviewer.md`（通过 Task 工具嵌套调用），这是一条三层安全验证流水线
- **跨件契约**：通过 `allowed-tools` 声明工具权限；规则文件（`.claude/rules`）采用无 [frontmatter](#frontmatter) 的纯 Markdown 约定，与 command/skill 的 YAML frontmatter 并行共存

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「分职责守卫」——每个工作阶段都有专属 agent 负责，handoff-verify 作为阶段交接的质量门，verify-agent 作为验证流水线的协调者
- **解决什么问题**：在 AI 辅助开发中，如何在不依赖人工 review 的情况下维持代码质量和安全基线
- **做了什么 trade-off**：安全 hook 选择了环境变量条件激活而非始终激活，牺牲了本地开发的防护覆盖，换取部署灵活性；所有 hook 只在 `OPENCLAW_SESSION_ID` 存在时激活
- **反映什么认知模型**：作者将 Claude Code 视为一个可编程的自动化引擎，用 hook 脚本做运行时守卫、用 agent 网络做工作流编排——这是比「单一 skill」高一层的系统化思维

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「多层验证流水线」（Multi-Layer Verification Pipeline）**

关键特征：
- 特征 1：存在专职「验证 agent」（verify-agent），与功能性 agent 解耦，专门负责质量核验
- 特征 2：使用 Task 工具实现 agent 级别的嵌套派生（command → agent → sub-agent）
- 特征 3：有「铁律」式的强制规定——`skills/verification-engine/SKILL.md` 的 "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE" 直接锁死了 agent 的完成声明权限
- 特征 4：安全检查以 hook 为基础设施层、以 agent 为应用层分离部署
- 特征 5：`skills/continuous-learning-v2` 是一个带 observer agent 的经验积累子系统

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多阶段质量门的研发工作流 | ✅ 高度适用 | 多层 agent + hook 防护能有效减少 AI 幻觉导致的错误发布 |
| 单人快速原型开发 | ❌ 过度工程 | 98 个工件的维护成本远超原型阶段收益 |
| 安全合规要求严格的团队 | ✅ 适用 | CWE Top 25 + STRIDE 框架在 security-pipeline skill 中已预置 |
| 刚接触 Claude Code 的新手 | ❌ 不适用 | OPENCLAW_SESSION_ID 等自定义配置门槛较高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：多层验证流水线 | claude-forge | 质量门显式化，防止 AI 假完成 | 维护成本高，agent 链路长易断裂 |
| 备选 A：单 skill 平铺 | numman-ali/n-skills | 上手成本低，每个 skill 独立可用 | 缺乏跨阶段一致性检查 |
| 备选 B：Rule + CLAUDE.md 约定驱动 | BayramAnnakov/claude-reflect | 零运行时开销，纯声明式 | 无法在执行阶段动态拦截 |

### 2.4 改进空间
1. **当前问题**：安全 hook 依赖 `OPENCLAW_SESSION_ID` 激活，本地开发零防护 **改进做法**：加入警告日志（`echo "⚠ Security hooks inactive" >&2`）并在 README 首页用醒目提示说明激活前提 **预期收益**：消除"安全感虚假"问题，让本地用户知道防护未启用
2. **当前问题**：verify-agent 的 `tools` 数组缺少 `Task`，导致安全子 agent 无法被派生 **改进做法**：在 `agents/verify-agent.md` frontmatter 中添加 `Task` **预期收益**：handoff-verify 流水线恢复完整功能，修复时间 < 1 分钟
3. **当前问题**：MCP 包全部使用 `npx -y @xxx@latest` 不锁版本 **改进做法**：用 `npm view @package version` 获取当前稳定版后写死版本号 **预期收益**：消除供应链攻击窗口，每次 session 启动行为可重现

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-16 得分 **82/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 60 | 无 YAML frontmatter（BUG） |
| commands/update-codemaps.md | 60 | 无 YAML frontmatter（BUG） |
| commands/agent-router.md | 72 | 缺少 allowed-tools（BUG） |
| agents/code-reviewer.md | 90 | 表现最好的工件 |

### 3.2 当时值得借鉴的模式

1. **铁律式完成约束** → 根本好处：强制 AI 验证而非假装完成 → `skills/verification-engine/SKILL.md` → 在自己的 skill 中加 "NO COMPLETION CLAIMS WITHOUT VERIFICATION" 一行
2. **CWE + STRIDE 安全框架** → 根本好处：系统化覆盖安全检查盲区而非临时凑单 → `skills/security-pipeline/SKILL.md` → 在安全类 skill 中引用 CWE Top 25 作为检查清单
3. **handoff-verify 的阶段交接模式** → 根本好处：交接时强制触发质量门而非依赖人工记忆 → `commands/handoff-verify.md` → 在多阶段任务结尾加一个 verify 步骤
4. **experience-accumulation 经验侧车** → 根本好处：经验在 session 间持续积累 → `skills/continuous-learning-v2/` → 设计 observer + instinct 模式保存历史决策
5. **`argument-hint` 引导多子命令** → `commands/checkpoint.md` 管理 save/restore/list/diff/delete → 在有子操作的 command 中用 argument-hint 告知用户

### 3.3 当时的缺陷

1. **verify-agent 缺少 Task 工具** → 为什么会失败：Task 是 agent 派生子 agent 的唯一工具，缺失后 security-reviewer 永远无法被调用，整条链路静默断裂 → **自查**：我的 gstack 项目中有类似的 agent chain 吗？如有需核查每个 dispatcher agent 的工具声明
2. **3 个 command 无 YAML frontmatter** → 根本原因：这些 command 是从参考文档"升格"而来，没有经历完整的 frontmatter 添加流程 → **自查**：当从文档升格为 command 时容易犯此错误
3. **8 个 command 缺 allowed-tools** → 根本原因：这是系统性的「先写内容、忘写声明」反模式——先写命令逻辑，在 frontmatter 上偷懒 → **自查**：我的 bureau 和 echo-sleuth 项目中的 command 有无此问题
4. **安全 hook 依赖环境变量激活** → 根本原因：将「部署配置」和「安全保证」耦合在一起，本地用户在不知情的情况下裸跑 → **自查**：我有没有把防护设计成需要额外配置才能激活的形态
5. **MCP 包未锁版本** → 根本原因：开发者为图方便使用 `@latest`，在工具链中这是供应链风险的温床

### 3.4 当时的优化机会

1. **agent-router 的 agent 分发声明** → 路由 34 个领域但 allowed-tools 为空，等于一个没有通行证的交警
2. **continuous-learning-v2 的 observer 无工具** → 无法读写 observations.jsonl，整个学习循环实际失效
3. **debugging-strategies 的 6 个死链** → 引用了 6 个不存在的文件，skill 主体内容几乎为空

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
> 注：目标仓库 git clone 因代理限制未能成功，以下基于 audit 时间（2026-04-16）后约 2.5 个月的推断。

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| verify-agent 缺 Task 工具 | `grep "Task" agents/verify-agent.md` | **无法验证**（克隆受限） | 一行修复，预期已修 |
| 3 个 command 无 frontmatter | `grep -L "^---" commands/update-docs.md` | **无法验证** | 如 PR 被接受则已修 |
| MCP 包未锁版本 | `grep "@latest" .mcp.json` | **无法验证** | Medium 风险，可能仍存在 |

### 4.2 架构演进
无法获取当前 HEAD 状态。基于 audit 优先级建议（verify-agent 的 Bug#1 为最高优先级）推断：若维护者活跃，核心 bug 修复率较高；security hook 文档化改进成本低，可能已优先完成。

### 4.3 新增的可学习模式
暂无（无法访问当前仓库状态）。

---

## 五、校准

### 5.1 我已经在做对的
1. 在多工件项目中区分 agent 和 skill 的边界——claude-forge 的 verify-agent 是正确做法
2. 在 CLAUDE.md 中描述全局工作流而非散落在各文件
3. 给 security-review 类任务使用独立 agent 而非内嵌在 command 中
4. 为工具链的每个组件写明 allowed-tools（若我已做）
5. 把「完成声明」与「验证证据」绑定

### 5.2 挑战 / 验证
- **挑战的假设**：我之前认为「hook 只要存在就能提供保护」——claude-forge 提醒我，hook 可能有激活前提条件，不存在就是 0 保护，没有文档说明的保护等于没有保护。
- **验证的做法**：在 hook 脚本的顶部加一行条件检查和警告日志，成本极低。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 agent 中有无缺少 Task 工具声明（用于派生子 agent 的场景）
grep -rn "Task" ~/.claude/agents/*.md | grep -v "allowed-tools" | head -20
# 命中后：在对应 agent 的 frontmatter tools 数组中添加 "Task"

# 检查自己的 command 是否存在无 frontmatter 的文件
grep -rL "^---" ~/.claude/commands/*.md 2>/dev/null
# 命中后：为每个命中文件添加完整 YAML frontmatter 块

# 检查 .mcp.json 是否有未锁版本的包
grep "@latest" ~/.claude/.mcp.json ~/.mcp.json 2>/dev/null
# 命中后：用 `npm view <package> version` 获取当前版本并写死
```

### 6.2 灵感 → 实施路径

1. **想法**：给我的 bureau 项目的关键 command 加一个「handoff-verify」式的收尾步骤
   - **为何可行**：bureau 处理会话知识积累，每次 compile 前验证数据完整性是关键质量门
   - **第一步**：在 `commands/compile.md` 末尾加一个 Step N：「运行 verify.md 检查 knowledge base 完整性」，15 分钟完成

2. **想法**：在我的 gstack 项目中实现 continuous-learning 模式，记录跨 session 的 agent 决策
   - **为何可行**：gstack 有 23 个工具，长期使用的经验值得保留
   - **第一步**：新建 `skills/session-memory/SKILL.md`，定义 instinct 格式（JSON Line），并在 gstack 的每个 command 末尾加「保存本次决策到 instincts.jsonl」，30 分钟初稿

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（含 10 个活跃公开仓库）
> 注：my-repos git clone 因代理限制未成功，以下分析基于仓库描述和公开 README 推断，无法给出 grep 计数

### 7.1 目的对齐度

- **claude-forge 的核心目的**：DevSecOps 全流程工作流套件，以多层验证流水线保证 AI 开发质量

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多工具 Claude Code 套件，目标是覆盖研发流程全链路 | gstack 更偏角色模拟（CEO/设计师），claude-forge 更偏安全验证 | 高 |
| MarkQWu/bureau | 中 | 同为 Claude Code 插件，有多阶段处理逻辑 | bureau 聚焦知识管理，claude-forge 聚焦代码质量 | 中 |
| MarkQWu/graphify | 低 | 同是 AI 辅助开发工具 | graphify 是知识图谱查询，claude-forge 是工作流编排 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 我的项目推测情况 | 严重度 |
|---|---|---|
| agent 缺 `Task` 工具声明 | gstack 有 CEO/EM 等角色 agent，若有嵌套派生则极可能存在此问题 | 高 |
| command 缺 `allowed-tools` | bureau 和 echo-sleuth 的 command 如快速开发则易遗漏 | 高 |
| MCP 配置包未锁版本 | 若有 `.mcp.json` 则需检查 | 中 |

**命中后的具体行动**：
- gstack：`grep -rn "tools:" .claude/agents/*.md` 逐一核查有无 `Task` 声明，5 分钟完成
- bureau：`grep -rL "allowed-tools" .claude/commands/*.md` 找出裸命令，每个添加声明需 2 分钟

### 7.3 别人的更优方案

1. **领域**：验证流水线显式化
   - **claude-forge 做法**：专设 `agents/verify-agent.md` + `commands/handoff-verify.md`，将「做完→验证」耦合为流水线的最后一个门
   - **我的项目现状**：gstack 的角色 agent 可能各自独立运行，没有统一的验证收尾
   - **如何借鉴**：在 gstack 的 `commands/` 下新建 `handoff-verify.md`，在每个 Eng Manager 命令完成后调用，diff 思路：新增 `allowed-tools: [Task]` + 调用 `agents/verify-agent.md`

2. **领域**：铁律约束（无证据不得声明完成）
   - **claude-forge 做法**：`skills/verification-engine/SKILL.md` 中明写 "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"
   - **我的项目现状**：可能无类似显式约束
   - **如何借鉴**：在 bureau 的 CLAUDE.md 中加一条全局规则：「编译完成前必须输出数量摘要（N 条目 → M 篇文章），不得仅说'已完成'」

### 7.4 反向：我的项目做得比他们好的地方

暂无（无法读取当前仓库代码，无法做正向对照）。

---

## 八、术语表

### <a name="工件"></a>工件
> 在 NLPM 语境中，指 `.claude/commands/`、`.claude/agents/`、`skills/` 等目录下的 Markdown 文件。这些文件是 Claude Code 能读取并执行的"指令模板"，是整个 NL 编程系统的基本单元。

### <a name="hook"></a>hook
> Claude Code 的"事件钩子"。你可以配置在特定动作（如 Claude 使用 Bash 工具前/后）自动触发的 shell 脚本。用于守卫、日志、速率限制等目的。类似于 git pre-commit hook 但运行时机更丰富。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 YAML 配置块。Claude Code 通过读取它来知道这个文件是什么类型的工件、叫什么名字、能使用哪些工具。没有 frontmatter = Claude Code 不认识这个文件。

### <a name="MCP"></a>MCP
> Model Context Protocol，一种让 Claude 连接外部服务（如 GitHub、数据库、搜索引擎）的协议。`.mcp.json` 是 Claude Code 的 MCP 服务器配置文件，里面声明了哪些外部工具在 session 启动时自动加载。

### <a name="Task"></a>Task
> Claude Code 中用于派生子 agent 的工具。`agent A` 要调用 `agent B`，必须在 frontmatter 的 `tools` 数组里声明 `Task`。如果没有声明，调用会静默失败——这是 claude-forge 最严重的 bug 根源。
