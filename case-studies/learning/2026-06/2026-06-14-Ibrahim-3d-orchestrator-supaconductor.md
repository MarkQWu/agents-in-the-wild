# Ibrahim-3d/orchestrator-supaconductor — 学习案例

**仓库**：https://github.com/Ibrahim-3d/orchestrator-supaconductor
**Stars**：336 | **来源**：xiaolai upstream（<500 星，已在 upstream 池中）
**Audit 日期**：2026-04-30（历史快照）| **生成日期**：2026-06-14（基于当前 HEAD）
**主题标签**：`cross-reference`, `router-channels`, `examples-driven`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`orchestrator-supaconductor` 是一个**多 agent 编排框架**，专注于把复杂开发任务分解为「规划→执行→评估→修复」的循环，并支持并行 agent 调度。作者将其定位为「软件项目的 AI 指挥层」。

关键事实：
1. 作者 Ibrahim-3d 是全栈开发者，该仓库是其个人工具集的核心部件，已经在生产项目中实际使用
2. 336 stars，规模中等：107 个 NL 工件，audit 时 89/100，是本批次中质量最高的单体框架
3. 核心概念：**「轨道（Track）」执行模型** —— 每个开发任务运行在一条 track 上，支持多 track 并行，类似 Git 的多分支工作流
4. 包含「董事会（Board of Directors）」决策层：CEO、CMO、CTO 等 agent 会在关键决策点召开「董事会会议」进行跨职能评审
5. Audit 安全评级为「REVIEW」（非 CLEAR）：board-meeting agent 使用了 `read_file`/`write_file` 等非标准工具名，而非 Claude Code 的标准工具

### 1.2 架构剖析

**目录结构**（关键层级）：
```
orchestrator-supaconductor/
├── agents/
│   ├── board-meeting.md          # 董事会 agent（CEO + CMO + CTO 联合表决）
│   ├── conductor-orchestrator.md # 主编排者：分配 track
│   ├── loop-planner.md           # 规划 agent：生成执行计划
│   ├── loop-executor.md          # 执行 agent：按计划执行
│   ├── loop-execution-evaluator.md # 评估 agent
│   ├── loop-fixer.md             # 修复 agent
│   ├── parallel-dispatcher.md    # 并行调度 agent
│   ├── ceo.md / cmo.md / cto.md  # 业务角色 agent
│   └── ...（共约 15 个 agent）
├── commands/                     # 约 39 个命令（全缺 allowed-tools）
│   ├── conductor-setup.md
│   ├── loop-planner.md           # 指向 agents/loop-planner
│   └── ...
├── skills/
│   ├── subagent-driven-development/SKILL.md  # 子 agent 驱动开发核心
│   ├── test-driven-development/SKILL.md      # TDD：Iron Law + rationalization table
│   ├── systematic-debugging/SKILL.md         # 四阶段调试
│   ├── board-of-directors/SKILL.md           # 董事会议 skill
│   ├── loop-planner/SKILL.md                 # 规划 skill
│   ├── loop-executor/SKILL.md                # 执行 skill
│   └── ...（共约 30 个 skill）
```

- **文件类型分布**：约 15 个 agent（avg 76/100）、约 39 个命令（avg 91/100）、约 30 个 skill（avg 88/100）
- **编排关系**：用户 → 命令（入口）→ conductor-orchestrator 分配 track → loop-planner 生成计划 → loop-executor 执行 → loop-execution-evaluator 评估 → loop-fixer 修复（循环直到评估通过）。关键决策点：board-meeting agent 代表业务视角介入
- **跨件契约**：命令文件通过 `@agent-name` 引用 agent（但 audit 时有几个用了过时路径），skill 之间通过文件命名约定相互引用，`track-manager/SKILL.md` 管理 track 状态

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「循环即工作流」—— 不依赖一次性的线性指令，而是让 AI 规划、执行、自我评估、修复，直到达到质量标准为止。这个循环可以多轮运行
- **解决什么问题**：单次 AI 指令难以保证复杂任务的质量，作者通过「执行→评估→修复」的多轮循环把质量保证内置到工作流中
- **Trade-off**：循环模式增加了 token 消耗，但换来了更高的任务完成率；`board-of-directors` 引入了业务视角，但增加了流程复杂度
- **认知模型**：把软件开发视为「规划和执行相互验证的迭代过程」，评估 agent 是内置的 QA 角色，修复 agent 是内置的纠错机制

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多轮循环编排（Plan-Execute-Evaluate-Fix Loop）+ 并行轨道」**

关键特征：
- 明确的四步循环：plan → execute → evaluate → fix，每步都有专属 agent
- 支持多 track 并行，每条 track 独立运行同一循环
- 业务层（Board of Directors）与工程层（Loop agents）解耦
- 评估 agent 基于预定义的「评估标准」输出通过/失败，而非主观判断

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 复杂工程任务（需要多轮迭代的功能开发） | ✅ 高度适用 | 循环机制自动处理第一次执行的不完美 |
| 需要多角色决策的产品功能 | ✅ 适用 | Board of Directors 提供业务 + 技术双视角 |
| 简单单次任务（修一个 typo） | ❌ 过度设计 | 引入 conductor + loop + evaluator 成本远高于直接执行 |
| 实时响应场景（聊天机器人） | ❌ 不适用 | 多 agent 循环响应延迟过高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：Plan-Execute-Evaluate-Fix 循环 | Ibrahim-3d/supaconductor | 自我纠错、质量有保障 | token 消耗高、loop 可能不收敛 |
| 备选 A：单次调度（Router + 专家 agent） | 大多数 orchestrator 框架 | 响应快、成本低 | 质量取决于第一次就对 |
| 备选 B：人工审查在中间 | 传统 CI/CD 流水线 | 人工把控质量 | 失去 AI 自动化优势 |

### 2.4 改进空间

1. **当前问题**：`board-meeting` agent 使用 `read_file`/`write_file` 非标准工具名，实际会调用失败 **改进做法**：改为 Claude Code 标准工具 `Read`/`Write`，并在 frontmatter `tools` 字段声明 `- Read` / `- Write` **预期收益**：board-meeting 会议记录写入和读取正常工作，安全评级从 REVIEW 降为 CLEAR
2. **当前问题**：约 39 个命令全部缺 `allowed-tools` **改进做法**：大多数命令只需要声明 `allowed-tools: []`（纯文本 agent 调度命令），少数需要 `Bash` 的加上对应声明；可以批量处理 **预期收益**：命令注册更规范，工具访问意图更清晰
3. **当前问题**：`business-docs-sync/SKILL.md` 无任何 frontmatter **改进做法**：补充最小 frontmatter（name + description），`allowed-tools: []`（纯文档，不需要工具） **预期收益**：技能可以被正确注册和发现

---

## 三、过去审查发现（2026-04-30 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-30 当时得分 **89/100**（加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `agents/board-meeting.md` | 60 | BUG：`run_shell_command` 未在 frontmatter 声明；零示例 |
| `agents/conductor-orchestrator.md` | 65 | 零示例；description 描述工作流（CSO 反模式） |
| `skills/business-docs-sync/SKILL.md` | 50 | BUG：完全无 frontmatter |
| `commands/UI-UX-AUDIT-REPORT.md` | 50 | BUG：模板文件误放在 commands/ 目录 |
| `skills/test-driven-development/SKILL.md` | 92 | 优秀：Iron Law + rationalization table + red flag 清单 |

### 3.2 当时值得借鉴的模式

1. **`test-driven-development/SKILL.md` 的「Iron Law + Rationalization Table」** → 为 TDD 的「先写测试」建立了不可违反的原则，并提供了一张「常见合理化借口 vs 正确应对」对照表，极具教学价值 → 路径：`skills/test-driven-development/SKILL.md` → 借鉴：规则类 skill 不只写「该怎么做」，还要写「常见的错误借口是什么、为什么它们是错的」
2. **`systematic-debugging/SKILL.md` 的四阶段调试** → Phase 1 复现、Phase 2 定位、Phase 3 修复、Phase 4 验证，每阶段有明确的退出条件 → 借鉴：流程类 skill 要为每个 phase 定义「完成条件」而不只是「做什么」
3. **`subagent-driven-development/SKILL.md` 的并行调度图示** → 用 ASCII 流程图展示子 agent 的并行/串行关系，比纯文字描述清晰 10 倍 → 借鉴：复杂编排 skill 用图示替代文字描述的编排逻辑
4. **Plan-Execute-Evaluate-Fix 循环对称设计** → planner / executor / evaluator / fixer 四个 agent 的命名完全对称，职责边界一眼可见 → 借鉴：一组协作 agent 应该有系统性的命名约定，让读者从名字就能理解每个 agent 在整体流程中的位置
5. **`conductor-orchestrator` 作为无状态路由层** → conductor 本身不执行任务，只负责分析请求、分配 track、启动 loop → 借鉴：编排 agent 应该是「无状态路由器」，不要在它里面积累业务逻辑

### 3.3 当时的缺陷

1. **`board-meeting` agent 使用了 `read_file`/`write_file`** → 根本原因：这些是某些 Python AI 框架的工具名（如 LangChain），而 Claude Code 的标准工具是 `Read`/`Write`；作者可能从其他 AI 框架迁移过来，携带了旧框架的工具命名惯例 → 自查：我有没有在 agent 的 tools 字段里用非标准工具名？
2. **命令文件通过已过时的路径引用 agent** → `commands/loop-planner.md` 中写了 `.claude/agents/orchestrator-supaconductor:loop-planner.md`，但当前仓库的 agent 路径是 `agents/loop-planner.md` → 根本原因：仓库重构时更改了目录结构，但命令文件没有同步更新 → 自查：当我重命名或移动 agent 文件时，有没有全局搜索所有引用该路径的命令和 skill？
3. **`business-docs-sync/SKILL.md` 完全没有 frontmatter** → 根本原因：该 skill 可能是后来补充的文档，作者写成了纯 markdown 文档格式而忘记加 frontmatter，导致 Claude Code 无法识别为 skill → 自查：我的 skills/ 目录下有没有缺 frontmatter 的 SKILL.md？

### 3.4 当时的优化机会

1. 给 `board-meeting` agent 的 9 个 agent（CEO、CMO、CTO 等）各补 1 个具体示例：「当我接收到什么请求时，我会输出什么格式的决策」
2. `conductor-orchestrator` 的 description 字段描述的是工作流（CSO 反模式），应改为触发条件描述：「当需要分配新的执行 track 时」
3. 把 `commands/UI-UX-AUDIT-REPORT.md`（一个模板文件）移出 `commands/` 目录，放到 `templates/` 或 `docs/`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `board-meeting.md` 使用非标准工具名 | `grep "tools:" agents/board-meeting.md` | **仍存在**：frontmatter 仍有 `- read_file` 和 `- write_file` | 作者未修复；会议记录写入在 Claude Code 中仍会失败 |
| `business-docs-sync/SKILL.md` 无 frontmatter | `head -3 skills/business-docs-sync/SKILL.md` | **仍存在**：文件以 `# Business Document Sync Strategy` 开头，无 frontmatter | 该 skill 仍无法被注册 |
| `commands/loop-planner.md` 中的过时路径 | `grep "orchestrator-supaconductor:" commands/loop-planner.md` | **仍存在**：命中 3 处旧路径引用 | 命令仍引用不存在的路径，实际调用会失败 |

### 4.2 架构演进

从 audit（2026-04-30）到当前 HEAD，主要变化：
- `skills/` 目录有新增，特别是 `skills/using-supaconductor/` —— 这是一个「框架使用说明」skill，说明作者开始考虑框架的可入门性
- `skills/brainstorming/` 重命名自 `skills/brainstorm`（原 audit 无此命名变更记录，但 skills 目录新有此文件）
- 核心 loop agent 结构稳定

### 4.3 新增的可学习模式

`skills/using-supaconductor/SKILL.md` 是一个值得关注的新增：它是「框架的自解释 skill」—— 让框架本身告诉用户怎么使用框架。这种「用框架的语言写框架的文档」是一种元级别的 dogfooding，值得借鉴到复杂插件的 onboarding 设计中。

---

## 五、校准

### 5.1 我已经在做对的

1. **使用 Claude Code 标准工具名**：echo-sleuth 的所有 agent/skill 均使用 `Read`、`Write`、`Edit`、`Bash` 等标准名称，没有从其他框架迁移过来的非标准名称
2. **skill 的 frontmatter 完整**：echo-sleuth 的 4 个 SKILL.md 均有 frontmatter，与 `business-docs-sync` 的问题不同
3. **避免路径引用在重构后不同步**：echo-sleuth 结构简单，尚未遇到这个问题；但这提醒我在重构时要做全局搜索

### 5.2 挑战 / 验证

- **挑战**：本案例挑战了「打 89 分说明质量很好」的假设。89/100 的仓库依然有 3 个未修复的「功能性」bug（board-meeting 工具名、business-docs-sync 无 frontmatter、过时命令路径），并且这些 bug 在两个月后仍未修复。这提醒我：**NL 分数高不等于没有 bug，工具兼容性问题（非标准工具名）是一类容易被忽视的致命问题**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否使用了非 Claude Code 标准的工具名
# 标准工具名：Read, Write, Edit, Bash, Glob, Grep, Task, AskUserQuestion, WebFetch, WebSearch
grep -rn "read_file\|write_file\|list_files\|execute_bash\|create_file" \
  /tmp/my-repos/ 2>/dev/null

# 命中后：将非标准工具名替换为 Claude Code 的标准工具名
```

```bash
# 检查 SKILL.md 文件是否都有 frontmatter（以 --- 开头）
find /tmp/my-repos -name "SKILL.md" | while read f; do
  first=$(head -1 "$f")
  if [ "$first" != "---" ]; then
    echo "NO FRONTMATTER: $f"
  fi
done

# 命中后：在文件开头补充最小 frontmatter（name + description + allowed-tools）
```

```bash
# 检查命令文件中是否有引用已不存在的 agent 路径
grep -rn "agents/.*\.md" /tmp/my-repos/*/commands/*.md 2>/dev/null | \
  while read line; do
    path=$(echo "$line" | grep -o "agents/[^'\"]*\.md")
    [ -n "$path" ] && [ ! -f "$(dirname $(echo $line | cut -d: -f1))/../$path" ] && echo "BROKEN: $line"
  done

# 命中后：更新引用路径或确认引用是否仍有效
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 引入类似「执行→评估」的两步验证，让 `extract` 命令在提取后自动触发 `memory-auditor` agent 评估质量  
   **为何可行**：echo-sleuth 的 `extract` 命令目前是单步操作，加一个自动评估步骤可以过滤低质量提取结果，提高记忆库质量  
   **第一步**：在 `commands/extract.md` 末尾加「Step 6: 提取完成后，召唤 memory-auditor agent 对本次提取的条目做质量评分，低于 0.6 的条目标记为 `draft`」→ 15 分钟

2. **想法**：给 echo-sleuth 的 `agents/memory-auditor.md` 加一个「自解释 skill」，说明如何使用整个 echo-sleuth 框架  
   **为何可行**：`using-supaconductor` skill 的做法说明复杂插件需要有一个「我是什么、怎么用」的入口 skill；echo-sleuth 目前只有 README，没有 Claude Code 可以读取的使用说明  
   **第一步**：创建 `skills/using-echo-sleuth/SKILL.md`，写入框架用途、推荐调用顺序（extract → audit → prune → dashboard）→ 20 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 Ibrahim-3d/orchestrator-supaconductor 的核心目的**：为复杂开发任务提供「规划→执行→评估→修复」的多 agent 编排框架

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为有编排逻辑的 Claude Code 插件（多 agent 协作） | echo-sleuth 的任务是挖掘对话而非开发任务；编排复杂度远低于 supaconductor | 高 |
| MarkQWu/claude-for-legal | 低 | 同为多 agent 架构 | claude-for-legal 的 agent 是被动触发型，supaconductor 是主动循环型 | 低 |
| MarkQWu/drama-workshop-skills | 无 | — | 目的完全不同 | 无 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 非标准工具名 | `grep -rn "read_file\|write_file" /tmp/my-repos/` | **未命中** | 无 |
| SKILL.md 无 frontmatter | `find /tmp/my-repos -name "SKILL.md" -exec grep -L "^---" {} \;` | **未命中**（echo-sleuth 4 个 SKILL.md 均有 frontmatter） | 无 |
| agent 零示例 | `find /tmp/my-repos -path "*/agents/*.md" -exec grep -L "## Example" {} \;` | **命中全部 5 个 echo-sleuth agents + 5 个 claude-for-legal agents** | 高 |

**具体行动建议**：
- `echo-sleuth/agents/file-historian.md` → 在末尾加 `## Example` 段，描述「触发词：『分析我的 bash 历史』→ 输出：最近 7 天的 git 提交分类报告（表格格式）」→ 10 分钟

### 8.3 别人的更优方案

1. **领域**：TDD Skill 的规则+反例对照  
   - **本案例做法**：`test-driven-development/SKILL.md` 的「Rationalization Table」列出 6 种常见的「为什么我没先写测试的借口」和逐条反驳，配合「Iron Law: 红灯才提交实现」  
   - **我的项目现状**：echo-sleuth 无类似的「原则 + 常见违反模式」对照文档  
   - **如何借鉴**：在 `skills/git-mining/SKILL.md` 中加一个「常见失误」段，列出「用 git log 挖掘时的 3 个常见误操作及正确做法」

2. **领域**：「框架自解释 skill」  
   - **本案例做法**：`skills/using-supaconductor/SKILL.md` 是框架的使用手册，Claude Code 可以在对话中直接 load 这个 skill 来了解如何使用框架  
   - **我的项目现状**：echo-sleuth 只有 README.md（人读的），没有 Claude Code 可以 load 的「使用手册 skill」  
   - **如何借鉴**：新建 `skills/echo-sleuth-guide/SKILL.md`，说明 extract → audit → prune → dashboard 的推荐工作流

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：工具名称规范性  
  - **我的做法**：echo-sleuth 全程使用 Claude Code 标准工具名（Read、Write、Edit、Bash 等），无跨框架污染  
  - **本案例做法（弱在哪）**：`board-meeting.md` 中的 `read_file`/`write_file` 是从其他 AI 框架迁移来的遗留命名，导致安全评级为 REVIEW，且功能实际不可用  
  - **意义**：工具名称规范性是一个容易被忽视的坑；我目前保持了良好的习惯，未来在参考其他 AI 框架（LangChain、AutoGen 等）的设计时要注意不要带入它们的工具名称

---

## 八、术语表

### <a name="CSO-anti-pattern"></a>CSO 反模式（Context-Summarizing in Owner）
> 当一个 agent 或 skill 的 `description` 字段写成了「本 skill 执行 X 步骤，完成 Y 任务，然后 Z」这样对工作流的描述，而不是「当出现什么情况时，调用我」这样的触发条件描述。前者让调用方（命令/上游 agent）无法判断「什么时候该用我」；正确做法是 description 写触发条件，body 写实现细节。

### <a name="iron-law-tdd"></a>Iron Law（TDD 铁律）
> Ibrahim-3d 仓库的 `test-driven-development/SKILL.md` 中定义的不可违反原则：「红灯（测试失败）才能提交实现代码。在测试通过之前，不允许合并实现。」配套「Rationalization Table」列出常见借口（「时间紧」「这个逻辑太简单」「先出功能再补测试」等）并逐条反驳。

### <a name="track"></a>轨道（Track）
> Ibrahim-3d 框架中的核心概念：一条「轨道」代表一个开发任务的完整执行上下文（目标、计划、执行状态、历史记录）。类似 Git 分支，多条轨道可以并行运行，`conductor-orchestrator` 负责为每个新任务分配轨道，`track-manager` 负责轨道状态管理。

### <a name="非标准工具名"></a>非标准工具名
> Claude Code 有一套固定的工具名称（`Read`、`Write`、`Edit`、`Bash`、`Glob`、`Grep`、`Task`、`AskUserQuestion` 等）。如果在 agent frontmatter 的 `tools` 字段或 skill body 中写了其他 AI 框架的工具名（如 LangChain 的 `read_file`、`write_file`，或 AutoGen 的 `execute_code`），Claude Code 不会识别这些名称，调用时会失败。这是跨 AI 框架迁移时最常见的一类 bug。
