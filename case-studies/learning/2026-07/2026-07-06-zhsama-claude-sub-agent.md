# zhsama/claude-sub-agent — 学习案例

**仓库**：https://github.com/zhsama/claude-sub-agent
**Stars**：576 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-07-06（基于当前 HEAD）
**主题标签**：`cross-reference`, `vague-quantifier`, `template-design`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[zhsama/claude-sub-agent](https://github.com/zhsama/claude-sub-agent) 是一个「规格驱动开发（Spec-Driven Development）」多 agent 流水线，将「功能需求 → 规格 → 开发 → 验证」全链条拆分为 7 个专职 [subagent](#subagent)，通过一个 `/agent-workflow` 命令串联执行。作者的出发点是：把「写需求说明书 → 写代码 → 自我 review」这套手工步骤自动化，每步由专门的 AI 角色负责。仓库规模小（14 个工件），但流程设计密集——每个 spec-agent 都对应一个明确的生产阶段。

关键事实：
- **创建背景**：面向希望让 Claude Code 完成端到端开发任务的工程师
- **规模**：12 个 agent（7 个 spec-agent + 2 个 domain-agent + 3 个 utility-agent）+ 1 个 command + 1 个 CLAUDE.md
- **特殊性**：明确定义了「质量门控（Quality Gate）」机制——开发流程中设置多个检查点，每步达到阈值才进入下一步
- **语言双轨**：仓库提供 README.md（英文）和 README-zh.md（中文），面向中英文用户

### 1.2 架构剖析

```
zhsama/claude-sub-agent/
├── agents/
│   ├── spec-agents/        # 规格专项 agents（7个）
│   │   ├── spec-analyst.md     # 需求分析
│   │   ├── spec-architect.md   # 架构设计
│   │   ├── spec-planner.md     # 任务规划
│   │   ├── spec-developer.md   # 代码实现
│   │   ├── spec-tester.md      # 测试设计
│   │   ├── spec-reviewer.md    # 代码审查
│   │   ├── spec-validator.md   # 质量验证
│   │   └── spec-orchestrator.md # 流程协调
│   ├── backend/
│   │   └── senior-backend-architect.md
│   ├── frontend/
│   │   └── senior-frontend-architect.md
│   ├── ui-ux/
│   │   └── ui-ux-master.md
│   └── utility/
│       └── refactor-agent.md
├── commands/
│   └── agent-workflow.md   # 唯一 command，串联全流程
├── docs/                   # 流程说明文档
└── CLAUDE.md               # 项目配置
```

- **文件类型分布**：12 个 agent / 1 个 command / 1 个 CLAUDE.md / 3 个 docs
- **编排关系**：`/agent-workflow` 命令是唯一入口，内部按顺序调用 spec-analyst → spec-architect → spec-planner → spec-developer → spec-tester → spec-reviewer → spec-validator，每步依赖前步输出
- **跨件契约**：CLAUDE.md 描述了整个三门质量控制体系（Gate 1/2/3），spec-orchestrator 负责在运行时维持状态，但 command 文件实际上只实现了一个合并的 95% 门控

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「阶段性质量门控（Staged Quality Gates）」——把开发流程切成明确阶段，每阶段有数字化验收标准
- **解决什么问题**：让 Claude Code 不只是「写代码」，而是执行一套有纪律的规格驱动开发流程
- **Trade-off**：精细的阶段分工带来了清晰度，但也带来了跨文件一致性的维护负担——三个文件定义了三套不同的门控阈值（CLAUDE.md 是 80%/85%，spec-orchestrator 是 85%，command 是单一 95%），没有「单一来源」
- **认知模型**：作者把 Claude Code 视为可以执行「有纪律的软件工程流程」的系统，而不仅仅是代码生成工具——这是 NL 编程中少见的「流程自动化」视角

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「规格驱动流水线（Spec-Driven Pipeline）」

将软件开发拆分为严格有序的阶段，每阶段由一个专职 agent 处理，并设置数字化门控阈值来决定是否进入下一阶段。

模式特征清单：
- 特征 1：**阶段线性依赖**——每个 agent 的输入是前一个 agent 的输出，顺序不可颠倒
- 特征 2：**数字化门控**——不是「感觉差不多了继续」，而是「达到 X% 才进入下一阶段」
- 特征 3：**角色专职化**——分析、设计、规划、开发、测试、审查、验证各有专人，无一身兼多职
- 特征 4：**单一命令入口**——用户只需运行一个命令，不需要知道内部有多少个 agent

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有明确需求、希望交付生产级代码 | ✅ 高度适用 | 流水线本身就在模拟严格的开发纪律 |
| 快速原型/探索性编码 | ❌ 不适用 | 7 步流水线对「写个小脚本」来说过度设计 |
| 团队开发规范化 | ✅ 适用 | 可作为团队 review 流程的 AI 辅助 |
| 单 agent 无法完成的长任务 | ✅ 适用 | 每个阶段独立，不受上下文窗口限制 |
| 没有清晰功能边界的模糊任务 | ❌ 不适用 | spec-analyst 无法分析模糊输入 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：规格流水线 | zhsama/claude-sub-agent | 阶段清晰，门控可测量，角色专职 | 配置复杂，阈值需要跨文件同步 |
| 单 agent 全栈 | 大多数简单配置 | 零配置，灵活 | 无质量门控，输出质量不可控 |
| 路由 + 并行 agent | eyaltoledano/claude-task-master | 并行提速，可动态分配 | 并行 agent 间的上下文共享更复杂 |

### 2.4 改进空间

1. **当前问题**：三个文件定义了三套不同的门控阈值（80%/85%/95%），读者和系统都无从判断哪个是权威定义。**改进做法**：在 CLAUDE.md 中定义一个 `QUALITY_GATE_THRESHOLD` 常量，其他文件引用该常量而非硬编码数字。**预期收益**：消除矛盾，单一来源。

2. **当前问题**：`/agent-workflow` 命令在无参数时会启动完整的 7 步流水线但没有任何用户输入——等于告诉厨师「做点好吃的」但没说吃什么。**改进做法**：在命令顶部添加空参数守卫：「若 $ARGUMENTS 为空，先向用户询问功能需求，再启动流程」。**预期收益**：防止无意义的流水线执行。

3. **当前问题**：`agents/ui-ux/ui-ux-master.md` 没有 `tools:` 声明，无法读取设计文件或写输出文档，只能在对话中输出文本。**改进做法**：添加 `tools: Read, Write, Glob` 最小工具集。**预期收益**：agent 可以实际读取 Figma 导出文件、写设计规范文档。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-25 当时得分 **72/100**（14 个工件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/agent-workflow.md | 55/100 | 缺少 `name:` frontmatter（-25）+ 无空参数守卫（-10） |
| agents/frontend/senior-frontend-architect.md | 58/100 | 无 model（-5）、无示例（-15）、无输出格式（-10） |
| agents/spec-agents/spec-developer.md | 58/100 | 无 model（-5）、无示例（-15）、无输出格式（-10） |
| agents/spec-agents/spec-planner.md | 66/100 | 无 model（-5）、无示例（-15）、高模糊量词（-14） |
| agents/spec-agents/spec-orchestrator.md | 72/100 | 无 model（-5）、无示例（-15） |
| CLAUDE.md | 85/100 | 模糊量词（-8）+ 跨组件不一致 |

### 3.2 当时值得借鉴的模式

1. **流程拆分的粒度**：把「写需求」这件事拆成 analyst（理解需求）→ architect（设计方案）→ planner（分解任务），三个专职角色各管一层，职责清晰无重叠。借鉴：我的 bureau 仓库可以把「审阅」拆成「内容审阅 agent」和「格式审阅 agent」。

2. **spec-validator 的输出模板**：该 agent 有一个结构化的验证报告模板（评分项 + 通过/失败 + 具体问题），这是罕见的「可验证输出格式」——不只是「告诉你有问题」，而是给出一张结构化的检查单。

3. **双语 README**：仓库提供中英双语文档，对中文开发者友好。这个做法在英文主导的 GitHub 生态中少见但有价值。

### 3.3 当时的缺陷

1. **`commands/agent-workflow.md` 缺少 `name:` 字段**。根本原因：command 的 [frontmatter](#frontmatter) 只有 `description` 和 `allowed-tools`，没有 `name`——这等于给命令取了个别名但忘了给它注册正式名称。Claude Code 的命令注册机制依赖 `name` 字段生成斜杠命令，缺失导致 `/agent-workflow` 命令实际上无法被调用。**自查**：我的命令文件有没有 `name:` 字段？

2. **CLAUDE.md 中的 agent 引用名与 agent frontmatter 名称不匹配**。根本原因：`refactor-agent.md` 的 frontmatter 声明 `name: code-refactorer-agent`，而 CLAUDE.md 用 `refactor-agent` 引用它。这是典型的「命名漂移」——文件名和 frontmatter name 不同，外部引用又用了第三个名字，三者各不相同。**自查**：我的仓库中 CLAUDE.md 里引用的 agent 名称与其 frontmatter `name` 字段是否完全一致？

3. **质量门控阈值在三个文件中自相矛盾**。根本原因：每次需要描述门控逻辑时，作者都直接在当前文件里写了一个数字，没有单一来源。这种「每处都写、每处不同」的模式是文档维护的死亡螺旋——随着仓库演化，差异会越来越大，最终没有任何一个文档是可信的。**自查**：我的仓库中有没有在多个文件里重复声明同一个配置值？

### 3.4 当时的优化机会

1. 在 `commands/agent-workflow.md` frontmatter 中添加 `name: agent-workflow`（一行改动，解决注册失败的根本 bug）
2. 将 CLAUDE.md 第 111 行的 `refactor-agent` 改为 `code-refactorer-agent`，或将 `refactor-agent.md` 中的 `name:` 字段改回 `refactor-agent`
3. 为所有 12 个 agent 添加 `model: claude-sonnet-4-5` 声明（12 个文件，每个添加一行）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| commands/agent-workflow.md 缺 `name:` | `head -5 commands/agent-workflow.md` | ❌ **仍存在**：frontmatter 中仍只有 `description` 和 `allowed-tools`，无 `name:` 字段 | 命令注册 bug 两个月未修复，`/agent-workflow` 仍无法正常注册 |
| CLAUDE.md agent 名称不匹配 | `grep "refactor" CLAUDE.md` 和 `grep "^name:" agents/utility/refactor-agent.md` | ❌ **仍存在**：CLAUDE.md 第 110 行用 `refactor-agent`，agent frontmatter 仍是 `code-refactorer-agent` | 用户按 CLAUDE.md 安装后调用会静默失败 |
| 所有 agent 缺 model 声明 | `find agents/ -name "*.md" -exec grep -l "^model:" {} \;` | ❌ **仍存在**：12 个 agent 中 0 个有 `model:` 字段 | 质量债未动 |

### 4.2 架构演进

从 audit（2026-04-25）到当前 HEAD，目录结构基本无变化。新增了 `docs/spec-workflow-usage-guide.md`（使用指南）和 `docs/spec-workflow-system.md`（系统说明），但核心 agent 文件和命令文件未更新。

**作者意识到了什么**：增加了文档说明，说明作者意识到系统对新用户不够透明，但选择用文档补充而非修复核心 bug——这是一种「外围完善、核心暂缓」的维护策略。

### 4.3 新增的可学习模式

- `docs/Agent Directory Structure.md`：明确描述了 agent 目录的组织逻辑（frontend/backend/spec-agents/ui-ux/utility 分类），这是一种「元文档（meta-documentation）」模式——用文档记录文档的结构。这在小型仓库中少见，但对 12 个以上的 agent 集合有价值。

---

## 五、校准

### 5.1 我已经在做对的

1. bureau 仓库的命令（如果有）有 `name:` 字段——避开了 zhsama 最严重的 bug
2. bureau 的 `recall/SKILL.md` 和 `review/SKILL.md` 有 `<example>` 块——比 zhsama 所有 agent 都缺示例要好
3. 我已意识到跨文件引用一致性的重要性（agent 名称不能在多处不一致）

### 5.2 挑战 / 验证

**挑战**：这个案例让我认识到「命令注册」是一个容易被忽视的失效点。`/agent-workflow` 这个命令在 CLAUDE.md 里被当作核心功能文档化，但实际上因为一个缺失的 `name:` 字段完全无法注册——这意味着用户安装了仓库，读了文档，然后运行 `/agent-workflow`，得到的是「命令不存在」。这种「文档与功能脱节」的失效是最具误导性的失效模式。

**验证**：「门控阈值单一来源」这个设计原则被这个案例负向验证——三个文件三套数字，任何一个都不可信。这个教训可以推广到任何配置值：时区、API 端点、超时时间，都应该有且只有一个文件声明。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 command 文件是否有 name: 字段
find ~/.claude/ /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-gstack/ \
  -name "*.md" -path "*/commands/*" 2>/dev/null \
  | xargs grep -L "^name:" 2>/dev/null
# 命中后怎么办：添加 name: <command-slug> 到 frontmatter

# 检查 CLAUDE.md 中引用的 agent 名称与 frontmatter 是否一致
# 先提取 CLAUDE.md 里所有 agent 引用（大概）：
grep -n "agent" /tmp/my-repos/MarkQWu-bureau/CLAUDE.md 2>/dev/null | head -10
# 再核对每个名字：
grep "^name:" /tmp/my-repos/MarkQWu-bureau/crew/auditor/agent.md 2>/dev/null
# 命中后怎么办：选择「文件名作为 name」或「frontmatter name 作为引用」，二选一并统一

# 检查配置值是否在多处重复声明（阈值、常量）
grep -rn "95%\|80%\|85%\|threshold\|gate" /tmp/my-repos/MarkQWu-bureau/ 2>/dev/null \
  | grep -v ".git"
# 命中后怎么办：评估是否可以用单一变量替代多处硬编码
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 设计一个类似 spec-validator 的「质量检查 agent」，专门负责检查 bureau 记录的 session 是否满足结构化标准
   - **为何可行**：bureau 已经有 `skills/lint/SKILL.md`，可以基于此设计一个输出结构化检查报告的 agent
   - **第一步**：读 `bureau/skills/lint/SKILL.md`，看当前 lint skill 的输出格式，约 10 分钟

2. **想法**：参照 zhsama 的流水线设计，为 gstack 的 spec skill 增加「验证门控」——在生成 spec 之后自动验证 spec 是否满足最低质量标准
   - **为何可行**：gstack 的 `spec/SKILL.md` 已有「turn vague intent into precise spec」的核心逻辑
   - **第一步**：在 spec/SKILL.md 末尾添加「输出前自我检查：需求是否可测量、边界是否明确、排除条件是否列出」，约 20 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 zhsama/claude-sub-agent 的核心目的**：通过多 agent 规格流水线实现端到端自动化开发
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都有多阶段 AI 流程（capture→compile→review→query） | bureau 是知识管理流程，zhsama 是开发流程 | 中 |
| MarkQWu/gstack | 低 | 都用 Claude Code 作为执行环境 | gstack 是工具集合，zhsama 是单一流水线 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `name:` 字段 | `find /tmp/my-repos/MarkQWu-bureau/ -path "*/commands/*" -name "*.md" 2>/dev/null` | bureau 无独立 commands/ 目录，不适用 | N/A |
| agent 名称跨文件不一致 | `grep "name:" /tmp/my-repos/MarkQWu-bureau/crew/auditor/agent.md` vs `grep "auditor" /tmp/my-repos/MarkQWu-bureau/CLAUDE.md` | bureau 的 auditor agent 在 CLAUDE.md 中的引用需要核查 | 中 |
| agent 全部缺 model 声明 | `find /tmp/my-repos/MarkQWu-bureau -name "agent.md" -exec grep -L "^model:" {} \;` | bureau 已有 `model: sonnet`，未命中 ✅ | N/A |

**命中后具体行动建议**：
- 核查 `bureau/CLAUDE.md` 中引用的 agent 名称与 `crew/auditor/agent.md` 中的 `name:` 字段是否一致；如有差异，统一为 frontmatter 中的 `name` 值为准，约 5 分钟

### 7.3 别人的更优方案

1. **领域**：阶段化质量门控
   - **本案例做法**：spec-orchestrator 在每步完成后检查输出质量分，不达标则要求重做
   - **我的项目现状**：bureau 的 `skills/lint/SKILL.md` 有 lint 逻辑，但无数字化门控阈值，只输出「问题清单」
   - **如何借鉴**：在 bureau/skills/lint/SKILL.md 中添加一个「评分输出」部分（如「lint 评分：X/10，低于 7 需返工」），给出具体行动建议而非开放性列举

2. **领域**：多 agent 元数据一致性文档（docs/Agent Directory Structure.md）
   - **本案例做法**：专门的文档说明「哪类 agent 放哪个子目录，为什么这么分」
   - **我的项目现状**：bureau 的 `crew/` 目录无说明文档
   - **如何借鉴**：在 bureau/crew/ 下添加 `README.md`（不超过 20 行），说明 agent 命名约定和目录结构逻辑

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 有 `<example>` 块
- **我的做法**：bureau 的 `recall/SKILL.md`、`review/SKILL.md`、`scribe/SKILL.md` 都有 `<example>` 标签包裹的具体调用示例
- **本案例做法**：zhsama 的 12 个 agent 中，绝大多数无任何示例块，spec-validator 仅有 1 个结构模板（无调用示例）
- **意义**：`<example>` 块是 NLPM 评分中最高权重的质量项之一（-15 per missing），bureau 在这里是亮点

---

## 八、术语表

### <a name="subagent"></a>subagent
> Claude Code 中可被另一个 agent 或命令「调用」的子级 AI 助手。每个 subagent 有自己的 system prompt 文件，在被调用时独立执行并返回结果。zhsama 仓库里，spec-analyst、spec-architect 等 7 个 spec-agent 都是 subagent——由 `/agent-workflow` 命令按顺序调用。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。Claude Code 在注册命令时读取 frontmatter 中的 `name` 字段生成斜杠命令。若缺少 `name`，命令虽存在于文件系统，但无法被 Claude Code 识别和注册——等于写了代码但没有入口函数。

### <a name="quality-gate"></a>质量门控（Quality Gate）
> 软件开发流水线中设置的检查点——只有当前步骤的输出质量达到预设阈值（如「代码覆盖率 ≥80%」「测试通过率 ≥95%」）时，才允许进入下一步骤。zhsama 仓库的设计理念就是把传统 CI 中的质量门控引入 AI agent 的执行流程中。

### <a name="naming-drift"></a>命名漂移（Naming Drift）
> 同一个实体在不同文件中用了不同名字，随着时间推移三个名字越来越不一致的现象。zhsama 仓库中，refactor-agent 的文件名、frontmatter name、CLAUDE.md 引用三者不同就是典型的命名漂移——三个名字，没有一个是可信的单一来源。
