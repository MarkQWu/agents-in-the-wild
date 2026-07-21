# shinpr/claude-code-workflows — 学习案例

**仓库**：https://github.com/shinpr/claude-code-workflows
**Stars**：320（exemplar_published=true 覆盖星数门槛）| **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-21（基于当前 HEAD）
**主题标签**：`router-channels`, `cross-reference`, `model-pinning`, `single-purpose`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`shinpr/claude-code-workflows` 是一个专注于 AI 辅助软件开发流程的 Claude Code 插件，提供从需求分析到代码实现、测试、代码审查的完整开发流水线。目标用户是需要在 Claude Code 里完成「从 PRD 到代码合并」全流程的工程师。⭐320。

关键事实：
- Audit 时为扁平结构（直接放 agents/ + skills/），当前 HEAD 已重组为 **4 个子包**（dev-workflows/、dev-workflows-frontend/、dev-workflows-fullstack/、dev-skills/），说明作者识别并响应了 audit 指出的「跨插件依赖未声明」问题
- 共 **24 个 agents**（Audit 时也是 24 个）和 **27 个 skills**，总计 51 个 NL 工件
- 所有 24 个 agents **至今仍无 `model:` 声明**（audit 时已指出，当前持续）
- `agents/technical-designer.md`（以及 frontend 版）仍在 body 里使用 Grep 但 frontmatter tools 里**未声明 Grep**（bug 持续）
- SECURITY 评级 CLEAR，是本批次唯一零安全发现的仓库

### 1.2 架构剖析

```
claude-code-workflows/                     ← 仓库根
├── agents/                               ← 共享 agent 层（多个子包共用）
│   ├── technical-designer.md             ← Bug：tools 未声明 Grep
│   ├── technical-designer-frontend.md    ← 同上
│   ├── requirement-analyzer.md           ← TaskCreate/Update 声明但未用
│   ├── planner/solver/codebase-analyzer/ ...（共 24 个）
├── skills/                               ← 全局 skills
│   ├── testing-principles/
│   ├── coding-principles/
│   ├── typescript-rules/
│   └── subagents-orchestration-guide/ ... (5 个)
├── dev-workflows/                         ← 后端开发流水线包
│   ├── agents/                           ← 后端专用 agents 副本/变体
│   └── skills/                           ← recipe-* skills（26 个）
│       ├── recipe-plan/
│       ├── recipe-build/
│       ├── recipe-implement/
│       ├── recipe-review/ ...
├── dev-workflows-frontend/               ← 前端开发流水线包
│   └── agents/                           ← 前端专用 agents（quality-fixer-frontend 等）
├── dev-workflows-fullstack/              ← 全栈包（明确处理跨插件依赖）
│   └── agents/
├── dev-skills/                           ← 独立技能包
└── lefthook.yml / scripts/              ← 本地 git hooks 配置
```

- **文件类型分布**：24 agents + 27 skills + 4 个子包（无 commands，agent 是用户入口）
- **编排关系**：`recipe-*` skills 是编排脚本，在 skill 内部调用特定 agent；例如 `recipe-build/SKILL.md` 内部指令「调用 task-executor 处理后端任务，调用 task-executor-frontend 处理前端任务」
- **跨件契约**：recipe-fullstack-* skills 依赖 frontend/backend 两个包的 agents；Audit 时这个依赖是隐式的，当前 HEAD 通过拆分 `dev-workflows-fullstack/` 子包实现了显式化

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「[Recipe-Agent 二层编排](#recipe-agent)」——recipe skills 定义「做什么、按什么顺序」，专用 agents 负责「怎么做」，两层分离让流水线可替换、可扩展
- **解决什么问题**：Claude Code 内的软件开发缺乏标准化流程——不同人用不同方式让 Claude 写代码，结果质量差异大；该仓库提供一套经过验证的「开发流水线 recipe」
- **做了什么 trade-off**：24 个 agents 提供了覆盖开发全流程的细粒度分工，但每个 agent 都需要 model 声明、示例、tool 列表，维护成本高，audit 时发现的系统性缺失（无 model 声明、无 examples）至今仍未修复
- **反映什么认知模型**：作者把软件开发的各个角色（PRD creator、planner、designer、executor、verifier、security reviewer）都映射为独立 agent，形成一个「虚拟开发团队」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「Recipe-Agent 二层编排」

这个模式的核心是：**recipe skills 作为流程定义层**（声明做什么、调用哪个 agent、按什么顺序），**专用 agents 作为执行层**（知道怎么做具体任务）。两层之间通过 agent 名称（`code-reviewer`、`task-executor`）耦合，解耦了流程逻辑和执行逻辑。

模式特征清单：
- **特征 1**：recipe skill 是「工作流脚本」，内容是「步骤 1：调用 X agent」而非「步骤 1：检查文件」
- **特征 2**：agents 是「角色」而非「工具」，每个 agent 扮演一个开发角色（设计师/执行者/验证者）
- **特征 3**：子包按「前后端/全栈」边界拆分，避免单个包对所有 agents 的硬依赖
- **特征 4**：全局 skills（testing-principles、coding-principles）被多个 recipe 引用，是知识共享层
- **特征 5**：24 个 agents 之间无直接调用，均通过 recipe 或用户显式触发

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖完整软件开发生命周期的工具 | ✅ 高度适用 | 24 个角色 agents 覆盖从需求到测试的全流程 |
| 多前后端技术栈共存的项目 | ✅ 适用 | frontend/backend 子包分离 |
| 强调可重复、标准化流程的团队 | ✅ 适用 | recipe = SOP，每次执行路径一致 |
| 个人快速原型项目（不需要完整流程） | ❌ 不适用 | 24 个 agents 配置 overhead 太重 |
| 纯 AI 生成代码（无流程规范需求） | ❌ 不适用 | recipe 架构的价值在流程规范，非生成质量本身 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Recipe-Agent 二层（本仓库）| shinpr/claude-code-workflows | 流程可标准化，角色分工清晰 | Agent 数量多，维护成本高 |
| 单 Agent 多 Skill | pe-menezes/fin-claude-plugin | 轻量，适合聚焦场景 | 难以处理复杂开发流程 |
| NL + 原生核心 | leowux/pony | 精确状态管理 | 需要额外 CLI 工具 |

### 2.4 改进空间

1. **当前问题**：24 个 agents 全部缺少 `model:` 声明，当前运行在 Claude Code 默认 model（当前为 Sonnet 4.6），但未来版本升级时全部 agents 行为可能改变。**改进做法**：按 agent 复杂度分配 model——codebase-analyzer/document-reviewer 可用 Haiku（便宜快速），technical-designer/prd-creator 用 Sonnet，不需要全部上 Sonnet。**预期收益**：成本降低 3-5×，同时行为版本固定。

2. **当前问题**：`technical-designer.md` 和 `technical-designer-frontend.md` 的 tools frontmatter 里没有 `Grep`，但 body 里多次调用 Grep。**改进做法**：在两个文件的 frontmatter 里加 `Grep` 到 tools 列表。**预期收益**：runtime 不再因工具未声明而降级到 Bash 替代，文件搜索准确度提升。

3. **当前问题**：24 个 agents 均无 standalone invocation 示例，只有 JSON 输出格式模板（不是具体的 input → output 对）。**改进做法**：每个 agent 加一个真实场景的 input/output 对（例如「requirement-analyzer 输入：用户想加登录功能 → 输出：分解后的 task 清单 JSON」）。**预期收益**：onboarding 速度提升，agent 可测试性增加。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时整体得分 **91/100**，51 个工件。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/work-planner.md | 80 | 无 model(-5)，5 个模糊词(-10) |
| agents/technical-designer.md | 80 | 无 model(-5)，Grep 未声明(bug)，5 个模糊词(-10) |
| agents/technical-designer-frontend.md | 80 | 同上 |
| agents/codebase-analyzer.md | 82 | 无 model(-5)，4 个模糊词(-8) |
| skills/test-implement/SKILL.md | 97 | 极少模糊词 |
| skills/testing-principles/SKILL.md | 97 | 极少模糊词 |
| ... 大多数 skills | 97 | 极少问题 |

### 3.2 当时值得借鉴的模式

1. **design-sync.md 的冲突示例设计**：`agents/design-sync.md` 虽无 model 声明，但因为有「丰富的具体冲突数据示例」（设计规格与实现不一致的真实案例），audit 给了 93 分偏移量。为什么好：示例的存在弥补了声明的缺失；具体示例比任何抽象描述都更能让 AI 理解边界情况。如何借鉴：在每个 agent 里加一个「最难处理的边界情况」示例，比加 5 个普通示例更有价值。

2. **references/*.yaml 结构化知识文件**：`skills/subagents-orchestration-guide/SKILL.md` 引用了 `references/skills-index.yaml`，这是一个把知识提取到独立文件、由多个 skills 引用的设计。为什么好：知识更新时只需改一个文件，引用它的所有 skills 自动获益。如何借鉴：把「被多个 skills 重复用到的知识」提取到 `references/` 目录下的独立文件。

3. **acceptance-test-generator 的 examples 驱动设计**：该 agent 有具体的测试场景数据示例（非模板），audit 因此给了 91 分（高于其他无示例的 agents 的 82-86 分）。为什么好：测试生成 agent 的核心价值就是能从需求生成具体测试，没有示例的话 Claude 不知道「具体」到什么程度。如何借鉴：对「生成型 agent」（生成测试/生成文档/生成 RFC），必须提供生成结果的示例，不能只有格式模板。

4. **子包拆分响应跨插件依赖问题**：当前 HEAD 的 dev-workflows-fullstack/ 子包明确了「需要同时安装 backend 和 frontend 包」，解决了 audit 时指出的「用户只装了一个包就遇到 missing agent 错误」。为什么好：架构上的跨件依赖需要在「用户安装/配置层面」显式声明，不能只靠 prose 描述。如何借鉴：设计多包架构时，在 README 和 marketplace.json 里明确列出 peerDependencies。

### 3.3 当时的缺陷

1. **24 个 agents 全部无 model 声明**（系统性问题）：根本原因：作者可能把「model 选择」看成是部署配置而非代码约定——但在 NL programming 里，model 选择影响 token 成本和行为质量，应该在 artifact 里显式声明。自查：我的 bureau agents 是否都有 model 声明？

2. **technical-designer.md 使用了 Grep 但未在 tools 里声明**（Bug）：根本原因：两个 technical-designer 文件（backend/frontend 版）看起来是从一个早期模板衍生出来的，那个模板在 Grep 被加到「标准工具列表」之前就已经创建，导致更新了 body 但忘了更新 frontmatter。这是「副本漂移」问题——维护多个相似文件时，某一次局部更新没有同步到所有副本。自查：我有没有类似的「相似 agent 副本」，某次更新只改了一个？

3. **requirement-analyzer.md 声明了 TaskCreate/TaskUpdate 但 body 里没有使用**：根本原因：文件是从其他 agent 的模板复制来的，模板里有 Task Registration 指令，但 requirement-analyzer 的职责不需要创建 task（它的输出是分析报告，不是 task 列表）。unused 的工具声明不是 crash 级 bug，但增加了 agent 的权限 footprint 和理解成本。自查：我的 agents 有没有声明了但从不使用的工具？

### 3.4 当时的优化机会

1. **最高 ROI**：为所有 24 个 agents 加 `model:` 声明（haiku/sonnet 分级），一行 per file，24 次 PR 可合并为 1 个批处理 PR
2. **Bug 修复**：technical-designer.md 和 .frontend.md 的 frontmatter tools 加 `Grep`
3. **一致性**：requirement-analyzer.md 从 tools 里移除 TaskCreate/TaskUpdate，或加上相应的 body 指令

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 24 agents 无 model 声明 | `grep "^model:" agents/technical-designer.md` | **持续**：tools 行第 4 行确认无 model 字段 | 系统性缺失，3.5 个月后仍无修复 |
| technical-designer.md Grep 未声明 | `grep "^tools:" agents/technical-designer.md` | **持续**：`tools: Read, Write, Edit, MultiEdit, Glob, LS, Bash, TaskCreate, TaskUpdate, WebSearch`，无 Grep | 虽然 body 大量使用 Grep，tools 声明冻结在 audit 时状态 |
| requirement-analyzer TaskCreate 未使用 | `grep "TaskCreate\|TaskUpdate" agents/requirement-analyzer.md` | **持续**：frontmatter tools 仍声明了这两个工具 | 低优先级 bug，作者选择不修 |

### 4.2 架构演进

**最重要的变化**：仓库从扁平结构演进为「4 个子包 + 共享 agents + 全局 skills」的 monorepo 结构。

- **audit 时结构**：`agents/`（24 个）+ `skills/`（27 个）全部平铺在根目录
- **当前结构**：根目录的 `agents/` 和 `skills/` 保留为「共享层」；`dev-workflows/`、`dev-workflows-frontend/`、`dev-workflows-fullstack/`、`dev-skills/` 成为可独立安装的子包

**这个演进说明了什么**：Audit 指出的「用户只安装 dev-workflows 但需要 task-executor-frontend（来自 dev-workflows-frontend）」问题，作者通过建立 `dev-workflows-fullstack/` 子包实现了「明确告知用户需要两个包」。这是从「文档说明」到「架构强制」的升级。

### 4.3 新增的可学习模式

**「monorepo 子包显式依赖」模式**：`dev-workflows-fullstack/` 子包的存在本身就是一个「显式依赖声明」——用户看到这个包就知道「我需要同时有 backend 和 frontend 包」，不需要读文档就能理解安装要求。这比在 README 里写 "⚠️ Note: You also need to install dev-workflows-frontend" 更清晰。

---

## 五、校准

### 5.1 我已经在做对的

1. **Agent 有 model 声明**：bureau 的 `crew/auditor/agent.md` 有 `model: sonnet` 声明，这在 shinpr 案例里是 24 个 agents 都缺失的
2. **单职责 agent**：bureau 的 auditor agent 专注「读入知识 → 生成审计结论」，没有 shinpr 里 requirement-analyzer 那种「声明了工具但不使用」的冗余
3. **子包依赖显式化**：graphify 的多个 skill 文件都在 frontmatter 里明确声明了 `prerequisites:`（若有），不依赖隐式的 prose 描述

### 5.2 挑战 / 验证

shinpr 的案例挑战了我的一个假设：「24 个专精 agents 的分工比 4 个通用 agents 更好」。实际上，24 个 agents 带来的维护负担（每个都需要 model 声明、examples、tool 完整声明）已经超过了分工带来的收益——3.5 个月里，「全部加 model 声明」这个一行修复没有发生。这验证了「agent 数量应该与维护资源匹配」的原则。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 agent 文件的 tools frontmatter 是否与 body 里实际使用的工具一致
# 步骤1：找到 body 里用了 Grep 的 agent
grep -rl "Grep\|grep " /tmp/my-repos/MarkQWu-bureau/crew/ --include="*.md" 2>/dev/null | \
  xargs grep -L "^tools:.*Grep" 2>/dev/null
# 命中（body 用了 Grep 但 tools 未声明）→ 立即加 Grep 到 frontmatter

# 检查有无「声明了工具但 body 里没用」的 agent
for f in $(find /tmp/my-repos/MarkQWu-bureau/crew -name "*.md"); do
  # 如果 tools 里有 TaskCreate 但 body 里没有 TaskCreate 或 TodoWrite
  if grep -q "TaskCreate" "$f" 2>/dev/null && ! grep -q "TaskCreate\|TodoWrite" "$f" 2>/dev/null; then
    echo "UNUSED TOOLS: $f"
  fi
done
# 命中后：移除 frontmatter tools 里未使用的工具，缩小权限 footprint

# 检查 bureau agents 是否所有都有 model 声明
find /tmp/my-repos/MarkQWu-bureau/crew -name "agent.md" | \
  xargs grep -L "^model:" 2>/dev/null
# 命中后：按能力需求加 model: haiku/sonnet
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 shinpr 的 recipe-* 模式，为 bureau 设计一套「工作流 recipe」（而非让用户自己选 skill 调用顺序）
   - **为何可行**：bureau 有 7 个 skills，某些任务自然有顺序（capture → compile → review）；recipe 可以把这个顺序显式化
   - **第一步**：写 `bureau/skills/recipe-daily-review/SKILL.md`，内部指令「先 capture → 然后 compile → 最后 review」，给用户一个一键触发的入口（1-2 小时）

2. **想法**：把 shinpr 的 references/skills-index.yaml 模式引入 graphify
   - **为何可行**：graphify 有多个 skill 文件引用相同的「extraction-spec」知识，当前每个文件各自包含一份副本
   - **第一步**：把 `graphify/skills/kilo/references/extraction-spec.md` 里的公共部分提取为顶级 `references/extraction-spec.md`，其他 skill 改为引用它（30 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **shinpr/claude-code-workflows 核心目的**：Claude Code 内完整软件开发流水线（需求 → 设计 → 实现 → 测试 → 审查），24 个专精 agents 构成虚拟开发团队

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都有 agent 层做编排；都解决 AI 辅助工作流标准化 | bureau 是知识管理，shinpr 是软件开发 | **中** |
| MarkQWu/gstack | 高 | 都是工程开发辅助工具；都有 review/spec 等技能 | gstack 是个人工具集（Garry Tan 风格），shinpr 是团队流水线 | **高** |
| MarkQWu/graphify | 低 | 都涉及代码分析 | graphify 专注代码图谱构建，非开发流水线 | 低 |
| MarkQWu/drama-workshop-skills | 无 | - | 领域不同 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 工具 body/frontmatter 不一致 | `grep -rn "Grep" /tmp/my-repos/MarkQWu-gstack/ --include="*.md" -l \| xargs grep -L "Grep" 2>/dev/null` | 需检查 gstack 的 pair-agent/SKILL.md 等 | **高**（若有是 runtime bug） |
| Agent 无 model 声明 | `find /tmp/my-repos/MarkQWu-bureau/crew -name "agent.md" \| xargs grep -L "model:"` | bureau auditor/agent.md 有 model 声明，其他需确认 | 中 |
| Recipe 缺失（顺序靠用户记忆）| `find /tmp/my-repos/MarkQWu-gstack -name "recipe-*"` | gstack 无 recipe 层，spec→plan→review 顺序靠用户知道 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的任何 SKILL.md 若 body 调用了 Read 等工具但 frontmatter `allowed-tools` 里没有 → 立即补全，5 分钟
- 为 `MarkQWu/gstack` 写一个 `dev-workflow/SKILL.md` recipe，把 `spec → review → land-and-deploy` 串起来，给用户一个标准化入口（2 小时）

### 7.3 别人的更优方案

1. **领域**：`references/` 目录存放被多个 skills 共引的知识文件
   - **本案例做法**：`skills/subagents-orchestration-guide/` 引用 `references/skills-index.yaml`，`skills/(web-automation)/playwright-skill/` 引用 `API_REFERENCE.md`；知识更新只需改一处
   - **我的项目现状**：`MarkQWu/graphify` 的多个 skill 文件各自包含 `extraction-spec` 描述的副本
   - **如何借鉴**：创建 `graphify/references/extraction-spec.md`，把所有 skill 里的副本改为引用 `see references/extraction-spec.md for the full spec`；extraction-spec 更新时只改一处（30 分钟）

2. **领域**：acceptance-test-generator 的具体测试场景示例
   - **本案例做法**：agent 里提供真实的测试输入（用户需求描述）和预期输出（测试用例 JSON）对，不是抽象的格式模板
   - **我的项目现状**：`MarkQWu/bureau` 的 `skills/review/SKILL.md` 只有格式模板，没有具体的「input→output」示例
   - **如何借鉴**：在 bureau/review/SKILL.md 加一个「示例：输入一段文字 → 输出带评级的评审结论」的具体案例（20 分钟）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Agent 数量与维护资源匹配
- **我的做法**：`MarkQWu/bureau` 只有 1 个 crew agent（auditor），`MarkQWu/gstack` 无专用 agent，所有功能通过 skill 调用实现
- **本案例做法**：24 个 agents，大量系统性缺失（model 声明、examples）在 3.5 个月里没有修复，说明维护资源不足以覆盖这么多 agents
- **意义**：我的「少 agent 多 skill」策略实际上是更可维护的选择；当 agent 数量超过维护资源时，质量债务会迅速积累

---

## 八、术语表

### <a name="recipe-agent"></a>Recipe-Agent 二层编排
> shinpr 仓库的核心设计模式。「Recipe」（食谱）是一个比喻——就像做菜的食谱告诉厨师「先炒葱姜，再放肉，最后调味」，recipe skill 告诉 Claude「先调用 requirement-analyzer agent，再调用 technical-designer，最后调用 task-executor」。Recipe 层定义顺序和编排逻辑，Agent 层负责具体执行。两层解耦后，可以换一套 recipe（比如「极速版：跳过 PRD 直接实现」）而不需要改 agents。

### <a name="副本漂移"></a>副本漂移（Drift）
> 当一段代码/配置有多个副本（copies）时，某次更新只改了其中几个，其余副本没有同步更新，导致副本之间出现不一致。shinpr 的 `technical-designer.md` 和 `technical-designer-frontend.md` 是同一模板的两个副本，Grep 工具被加到 body 后，只有一部分副本更新了 frontmatter。避免副本漂移的方法：要么提取到共享文件（`references/`），要么建立「需要同步」的 CHANGELOG 记录。

### <a name="monorepo"></a>monorepo
> 「mono-repository」的缩写。指把多个相关的独立包/模块放在同一个 git 仓库里管理的策略。shinpr 当前 HEAD 是 monorepo 结构：`dev-workflows/`、`dev-workflows-frontend/`、`dev-workflows-fullstack/` 三个可独立安装的包，都在同一个 `claude-code-workflows` 仓库里。Monorepo 的优势是「共享基础设施 + 方便跨包引用」，劣势是「单个仓库越来越大」。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包围的 YAML 配置块。对 agent 文件来说，`tools:` 字段（或有些实现里是 `allowed-tools:`）声明了「这个 agent 可以调用哪些工具」。若 body 里使用了 `Grep` 但 frontmatter `tools:` 里没有声明，Claude Code 在 runtime 会拒绝或降级工具调用。这是 shinpr 的 technical-designer bug 的根本原因。

### <a name="peerDependencies"></a>peerDependencies
> npm 包系统里的「同等依赖」声明。A 包声明 `peerDependencies: {B: "^1.0"}` 的意思是「A 工作时需要用户也安装 B，但我不会自动安装 B，请用户自己安装」。shinpr 的 `dev-workflows-fullstack` 子包依赖 `dev-workflows` 和 `dev-workflows-frontend` 两个包，在 manifest 里用 peerDependencies 显式声明这个需求，防止用户只装了一个就期待全栈 recipe 能工作。
