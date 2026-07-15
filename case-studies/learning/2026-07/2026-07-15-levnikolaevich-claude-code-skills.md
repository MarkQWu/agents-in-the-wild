# levnikolaevich/claude-code-skills — 学习案例

**仓库**：https://github.com/levnikolaevich/claude-code-skills
**Stars**：423 | **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-15（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `manifest-discipline`, `model-pinning`, `experience-accumulation`

> **本案例的核心价值**：三个月内发生了一次罕见的彻底架构重构——从 288 个工件的重型编排系统，缩减为 18 个干净 SKILL.md 的轻量技能库。作者用一句话解释了原因：「现代 Claude 不再需要复杂的编排脚手架」。这是关于 NL 编程演进方向的重要信号。

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-code-skills` 是 levnikolaevich 为工程师日常工作发布的一套「独立技能市场」——每个 SKILL.md 解决一个具体的工程问题，可以按需安装，相互独立。技能按功能套件分组（Review、Audit、Optimization、Testing、Product Discovery、Maintainer）。

关键事实：
- 423 颗星，定位为「高质量精品技能库」，而非「大而全的工作流系统」
- README 开篇第一句是架构宣言：「*Early coding models needed a large orchestration and evaluation harness. Modern Claude works better with concise procedural guidance, so that machinery has been removed.*」（早期模型需要复杂编排脚手架；现代 Claude 更适合简洁的程序性指导，因此那套机制已被移除）
- 当前仓库：6 个套件，18 个 SKILL.md，0 个编排脚本
- 配套有静态技能目录网站（https://levnikolaevich.github.io/claude-code-skills/）

### 1.2 架构剖析

**当前目录结构**（2026-07-15 HEAD）：
```
claude-code-skills/
├── CLAUDE.md / AGENTS.md    # 配置文件
├── plugins/
│   ├── review-suite/
│   │   └── skills/
│   │       ├── ln-11-plan-reviewer/SKILL.md
│   │       └── ln-12-delivery-reviewer/SKILL.md
│   ├── codebase-audit-suite/
│   │   └── skills/
│   │       ├── ln-21-documentation-auditor/SKILL.md
│   │       ├── ln-22-codebase-auditor/SKILL.md
│   │       ├── ln-23-test-suite-auditor/SKILL.md
│   │       ├── ln-24-architecture-auditor/SKILL.md
│   │       └── ln-25-persistence-auditor/SKILL.md
│   ├── optimization-suite/
│   │   └── skills/
│   │       ├── ln-31-performance-optimizer/SKILL.md
│   │       ├── ln-32-dependency-upgrader/SKILL.md
│   │       ├── ln-33-code-modernizer/SKILL.md
│   │       └── ln-34-benchmark-comparator/SKILL.md
│   ├── testing-suite/
│   │   └── skills/
│   │       ├── ln-41-test-strategy-planner/SKILL.md
│   │       └── ln-42-acceptance-test-builder/SKILL.md
│   ├── product-discovery-suite/
│   │   └── skills/
│   │       └── ln-51-opportunity-evaluator/SKILL.md
│   └── maintainer-suite/
│       └── skills/
│           ├── ln-61-skill-reviewer/SKILL.md
│           ├── ln-62-repository-publisher/SKILL.md
│           └── ln-63-release-publisher/SKILL.md
└── site/                    # 静态目录网站
```

**文件类型分布**（当前 HEAD）：
- 18 个 SKILL.md（6 套件 × 2-5 个技能）
- 0 个 agent
- 0 个 hook 脚本
- 0 个编排层
- 1 个静态目录网站

**编排关系**：完全无编排——每个 SKILL.md 是独立的、自包含的技能，无互相调用，无前置依赖。用户按需安装套件。

**跨件契约**：无。每个 SKILL.md 包含自己的 `## Tool Routing` 表（告诉 Claude 什么情况用什么工具）和 `## Checklist`（告诉 Claude 什么时候算完成）。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「模型够用，架构减负」——随着 Claude 能力增强，NL 框架的复杂度应该下降，而不是上升
- **解决什么问题**：工程师在 code review、架构审计、性能优化等场景下需要系统性指导，但不需要一个「完整的 AI 软件开发公司」
- **做了什么 trade-off**：
  - 功能覆盖 vs 认知负担：从「288 个工件的复杂系统」缩减到「18 个专注技能」，用户理解成本大幅下降
  - 自动化程度 vs 可控性：移除了自动编排层，把判断权还给用户（何时用哪个技能，用户决定）
  - 一体化 vs 按需安装：6 个独立套件可单独安装，用户不必装整个库
- **反映什么认知模型**：作者认为 Claude 已经足够聪明，不需要被一步步手把手引导；一个好的 skill 只需要告诉 Claude「什么情况用什么工具」+「什么时候算完成」，Claude 自己会找到最优路径

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「按功能域划分的轻量技能套件」

每个 SKILL.md 专注一个工程场景，自包含（含 Tool Routing + Checklist），按功能域归入套件（suite），套件之间完全独立。

模式特征清单：
- **自包含设计**：每个 SKILL.md 包含触发场景、工具路由表、执行清单——无需外部依赖
- **ln-XX 数字编号**：技能用数字索引（ln-11 到 ln-63），创造可排序的「技能目录」
- **套件粒度**：6 套件 = 6 个独立安装单元，用户按需组合
- **零编排层**：没有 orchestrator、没有 meta-agent、没有 router；模型智能是唯一的「编排」
- **工具路由表（Tool Routing）**：用 Markdown 表格明确告诉 Claude 「什么任务 → 用哪个工具 → 什么时候 → 失败怎么处理」，这比「你自己判断」更精确

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 工程师个人工具箱 | ✅ 高度适用 | 按需安装；技能独立；不强迫用户学整个框架 |
| 企业多团队共享 | ✅ 适用 | 套件粒度适合按团队需求分发 |
| 需要自动化完整工作流的场景 | ❌ 不适用 | 已移除编排层，需要手动串联多技能 |
| 早期模型（Claude 2 以下）环境 | ❌ 不适用 | 设计前提是「现代 Claude 足够聪明」 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 轻量技能套件（本仓库） | levnikolaevich/claude-code-skills | 低认知负担；按需安装；技能质量高 | 无自动编排；多技能场景需手动串联 |
| 重型编排框架（本仓库过去） | levnikolaevich v1（审计时） | 自动化完整工作流 | 复杂度高；维护成本大；依赖被废弃 |
| 按团队分 agent 矩阵 | LerianStudio/ring | 职能清晰；适合企业 | 学习曲线高；需要 `ring:` 命名空间知识 |
| 个人命令堆积 | aaronmaturen/claude-plugin | 快速积累 | 质量低；不可维护 |

### 2.4 改进空间

1. **当前问题**：所有 18 个 SKILL.md 缺少 `model:` 字段，用户不知道该用 Haiku 还是 Sonnet（成本差 10×）。**改进做法**：按技能复杂度分配：Delivery Reviewer（高复杂度）→ `model: claude-sonnet-4-6`；Documentation Auditor（中复杂度）→ `model: claude-sonnet-4-6`；Plan Reviewer（可用 Haiku 做初步检查）→ `model: claude-haiku-4-5`。**预期收益**：用户成本可降低 30-50%，同时明确了模型期望。

2. **当前问题**：SKILL.md 有详细的 Tool Routing 表和 Checklist，但没有一个完整的 `## Examples`（input→output 示例）。**改进做法**：为 ln-11（Plan Reviewer）添加一个 15 行的计划 → 审查报告的完整示例。**预期收益**：新用户不再需要自己测试才能理解技能的输出格式。

3. **当前问题**：`ln-61-skill-reviewer` 是用于「审查 skill 本身」的元技能，说明作者意识到 skill 质量需要工具保证——但目前还缺少与 NLPM 等外部质量工具的对接文档。**改进做法**：在 README 添加 `nlpm:check plugins/` 的用法说明，让维护者知道可以用 NLPM 评分。**预期收益**：技能库质量持续改进有工具支撑。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **77/100**。

| 文件/群组 | 当时分数 | 主要问题 |
|---|---|---|
| prompt_templates/（8 个文件） | 40–50 | 完全缺 frontmatter；无 output format |
| skills-catalog/ln-774-healthcheck-setup/SKILL.md | 75 | 无 model；无 examples；缺 Paths header |
| plugins/agile-workflow/skills/ln-4xx/（30+ 个） | 80 | 无 model；无 examples |
| plugins/agile-workflow/skills/ln-5xx/（20+ 个） | 80 | 无 model；无 examples |
| plugins/documentation-pipeline/skills/ | 80 | 无 model；无 examples |

**说明**：该仓库当时是一个有 288 个工件的重型系统，包含：`skills-catalog/`（单体目录）+ `plugins/agile-workflow/`（敏捷工作流，ln-200 到 ln-1000）+ `plugins/documentation-pipeline/`（文档流水线）+ `shared/agents/prompt_templates/`（无 frontmatter 的提示词模板）。整体架构今天已经完全不存在了。

### 3.2 当时值得借鉴的模式

1. **ln-XXX 数字编号系统** → 用数字编号给技能建立全局索引（ln-201-opportunity-discoverer、ln-1000-pipeline-orchestrator 等），即使在 288 个工件里也能快速定位。→ 借鉴：技能超过 10 个时，用数字前缀编号（10-99 单个技能，100-199 套件，等等）。

2. **adapter 层设计**（agile-workflow）→ 把「通用 skill」和「针对特定工作流的 adapter」分开：通用 skill 定义核心逻辑，adapter 为特定场景（敏捷、文档流水线）做适配。→ 借鉴：当同一个 skill 需要在不同语境下使用时，用 adapter 而非复制。

3. **prompt_templates 作为可复用基础层** → `shared/agents/prompt_templates/` 下的 general_analysis.md、review_base.md 等是跨 agent 的基础提示词组件——类似编程里的「公共函数库」。→ 借鉴：抽取跨 skill 的共同提示词结构，放入 shared/ 目录供复用。

### 3.3 当时的缺陷

1. **prompt_templates 完全缺 frontmatter（8 个文件，40-50 分）** → 根本原因：这些文件被设计为「内部组件」（由 agents 在运行时引用），不是直接给用户调用的 skills，因此作者没有添加 Claude Code 的 frontmatter。但 NLPM 按 NL 工件统一评分，不区分「内部组件」和「用户技能」。自查：我有没有写过「内部使用」的 prompt 文件，但没有加 frontmatter？

2. **100% skills/agents 缺 model 字段** → 根本原因：和其他仓库一样，model 声明规范是后来才制定的，早期文件没有回填。但 288 个文件的批量回填工程量很大，容易搁置。自查：我的 SKILL.md 文件有多少缺 model 字段？

3. **Security BLOCKED（5 项安全发现）** → 当时的重型系统包含 shell scripts、MCP 配置、Python subprocess 等执行面，触发了多个安全 flag。具体发现现在无法在当前 HEAD 核实（整个系统已被移除）。

4. **13 个 bugs（ln-XXX 适配 skill 的跨件引用错误）** → 当时的 288 工件系统中有大量 skill 之间的引用，ln-1000-pipeline-orchestrator 引用的 8 个子 skill 中有些名称不一致，导致运行时找不到。自查：我的 skill 互相引用时是否用了精确名称？

### 3.4 当时的优化机会

1. 为 prompt_templates/ 下的 8 个文件批量添加 frontmatter（即使只是 `name:` 和 `description:`）
2. 用脚本批量为 agile-workflow 的 30+ skills 添加 `model:` 字段（Haiku 用于简单任务，Sonnet 用于复杂分析）
3. 把 ln-1000-pipeline-orchestrator 的 8 个子 skill 名称做一次完整的一致性检查

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| prompt_templates 缺 frontmatter（8 个文件） | `find . -name "*.md" -path "*/prompt_templates/*"` | **目录完全不存在** | **已解决（删除）**：整个 prompt_templates 层被移除 |
| 288 个 skill/agent 缺 model 字段 | `grep -r "^model:" plugins/` | 当前 18 个 SKILL.md，**0 个有 model 字段** | **结构改变，问题持续**：工件大幅减少，但 model 缺失依然 100% |
| Security BLOCKED（5 项发现） | `find . -name "*.sh" -o -name "*.py"` | **当前仓库无任何脚本文件** | **已解决（删除）**：移除执行层后安全风险消失 |
| agile-workflow 中跨件引用错误（13 个 bugs） | `find . -path "*/agile-workflow/*"` | **目录完全不存在** | **已解决（删除）**：整个 agile-workflow 插件被移除 |

### 4.2 架构演进

这是本 routine 迄今见过的最戏剧性架构演进：

**2026-04-06 → 2026-07-15（约 3 个月）**：

| 维度 | 审计时（v1） | 当前（v2） | 变化 |
|---|---|---|---|
| 工件总数 | 288 | 18 | **-94%** |
| 目录结构 | skills-catalog/ + plugins/{agile-workflow,documentation-pipeline}/ + shared/agents/prompt_templates/ | plugins/{6 套件}/ | 彻底重组 |
| 最高 ln 编号 | ln-1000（pipeline-orchestrator） | ln-63（release-publisher） | **序号范围收窄 16×** |
| 编排层 | 存在（agile-workflow 有完整工作流编排） | **不存在** | 完全移除 |
| 安全状态 | BLOCKED（5 项发现，含 shell scripts） | 无可执行面 → 事实上 CLEAR | 大幅改善 |
| model 字段覆盖率 | 0% | 0% | 未改变 |

**作者在 README 里解释了原因**：「早期编码模型需要复杂的编排和评估脚手架才能可靠地执行复杂工作流。现代 Claude 和 Codex 模型更适合简洁的程序性指导，因此那套机制已被移除。」

这是一个关于「模型能力增长如何改变工具设计」的典型案例。当模型更聪明时，框架应该变得更简单，而不是更复杂。

### 4.3 新增的可学习模式

当前 HEAD 中，每个 SKILL.md 都有一个精心设计的 **`## Tool Routing` 表**，这是审计时（v1）没有的模式：

```markdown
| Need | Preferred tool | Use it when | Fallback |
|---|---|---|---|
| Repository instructions | Native file read plus Git | Always read AGENTS.md/CLAUDE.md first | ... |
| Symbols, references | Language server | Plan changes existing code relationships | Targeted search |
```

这种「工具路由决策表」比简单的 `allowed-tools:` 声明更进一步——它不只告诉 Claude「可以用哪些工具」，还告诉 Claude「在什么情况下优先用哪个工具，失败了用什么替代」。这是一个强大的 NL 编程模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **技能自包含设计**：drama-workshop-skills 的 short-drama/SKILL.md 是一个完整的自包含技能，和 levnikolaevich v2 的设计哲学一致。
2. **场景专注**：我的各个仓库都比较专注（drama-workshop-skills 专注于短剧，shiji-kb 专注于诗集知识库），和「轻量专注技能套件」的哲学吻合。
3. **避免过度工程**：我没有构建 288 个工件的重型系统——起点轻量，这反而可能是正确的。

### 5.2 挑战 / 验证

本案例提出了一个深刻的问题：**「我现在设计的 NL 框架，有哪些部分是『早期模型需要、现代模型不需要』的过度工程？」**

具体来说：如果我在 gstack 里为每个 skill 写了大量「步骤 1→步骤 2→步骤 3」的串行指导，现代 Claude 可能根本不需要这么细——Claude 自己就知道下一步该做什么。本案例启示我**测试「去掉一半指令，Claude 是否还能正确执行」**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否有 Tool Routing 表（levnikolaevich 的新模式）
grep -r "Preferred tool\|Use it when\|Fallback" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：已有，继续完善；未命中：参考 levnikolaevich 的 ln-11/SKILL.md 添加

# 检查我的 SKILL.md 有没有 Checklist（完成标准）
grep -r "^- \[ \]\|Checklist" ~/.claude/skills/*/SKILL.md 2>/dev/null | head -10
# 命中后：好，继续；未命中：为每个 SKILL.md 添加 3-5 项完成标准 checklist

# 识别我的框架里「过度工程」的部分（参考 levnikolaevich 的精简）
wc -l $(find . -name "SKILL.md") 2>/dev/null | sort -rn | head -10
# 命中后：行数最多的 SKILL.md 是否可以精简 30-50%？先去掉一半试试
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 的 SKILL.md 添加「工具路由表」（Tool Routing），替代现有的隐式工具使用
   - **为何可行**：levnikolaevich 证明这是比 `allowed-tools:` 更精确的工具约束方式
   - **第一步**：打开 `drama-workshop-skills/short-drama/SKILL.md`，在 `## 触发场景` 之后插入一个 `## 工具路由` 表，列出：「生成剧本 → Write 工具」、「查阅历史草稿 → Read 工具」、「统计集数 → Bash 工具」；20 分钟

2. **想法**：把 gstack 仓库里行数最多的 3 个 SKILL.md 各压缩 30%
   - **为何可行**：levnikolaevich 从 288 个文件压到 18 个，质量反而提升——「少即是多」已被验证
   - **第一步**：运行 `wc -l /tmp/my-repos/MarkQWu-gstack/*/SKILL.md | sort -rn | head -5`，找到最胖的 3 个；逐一检查哪些指令是「Claude 自己会做的」，删掉；30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 levnikolaevich/claude-code-skills 的核心目的**：工程师日常工作的精品技能套件市场，每个技能专注一个工程场景，按需安装

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 都是「专注单一场景的精品 skill」；都使用自包含设计 | drama 聚焦创作类；levnikolaevich 聚焦工程类 | 高 |
| MarkQWu/gstack | 中 | 都是 Claude Code skill 集合；都按功能分类 | gstack 更像「全能工具箱」；levnikolaevich 是专注精品库 | 中 |
| MarkQWu/bureau | 低 | 都是 Claude Code plugin | bureau 专注知识管理，不是技能市场 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 所有 SKILL.md 缺 model 字段 | `grep "^model:" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | drama-workshop-skills 的 SKILL.md **缺 model 字段** | 高 |
| 无 Tool Routing 表 | `grep "Preferred tool" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | 未找到 Tool Routing 表 | 中 |
| 无 Checklist（完成标准） | `grep "\- \[ \]" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | 未找到明确的完成 checklist | 中 |

**命中后的具体行动建议**：
- `MarkQWu/drama-workshop-skills/short-drama/SKILL.md` → 添加 `model: claude-sonnet-4-6`（5 分钟）
- 同上 → 添加 `## 工具路由` 表（20 分钟）
- 同上 → 在末尾添加 `## 完成清单` 章节，列 3-5 条 `- [ ]` 完成标准（15 分钟）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：Tool Routing 表（工具使用决策矩阵）
   - **本案例做法**：每个 SKILL.md 包含结构化的 `## Tool Routing` 表，4 列（Need / Preferred tool / Use it when / Fallback），精确到具体工具和 fallback 路径
   - **我的项目现状**：drama-workshop-skills 依赖 `allowed-tools:` 字段声明工具，但没有「什么情况用什么工具」的决策逻辑
   - **如何借鉴**：在 short-drama/SKILL.md 加入 `## 工具路由` 表，对应「写剧情 → Write」「查历史草稿 → Read」「统计集数 → Bash（wc）」

2. **领域**：以 Checklist 作为「完成标准」
   - **本案例做法**：每个 SKILL.md 末尾有明确的 `- [ ]` checklist（如「测试是否可重跑且幂等」、「覆盖所有必要证据」），Claude 把这个 checklist 当成「做完了吗」的自检标准
   - **我的项目现状**：drama-workshop-skills 没有明确的完成标准 checklist，Claude 的「完成」是模糊的
   - **如何借鉴**：为 short-drama/SKILL.md 加 `## 完成清单`，至少 3 条：□ 剧情大纲已输出、□ 集数范围在 50-100 集之内、□ 主角弧线完整

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Examples 示例覆盖（drama-workshop-skills）
- **我的做法**：drama-workshop-skills 的 short-drama/SKILL.md 在 frontmatter description 里有清晰的触发词汇列表，加上技能内容里有「快速入门」示例
- **本案例做法（弱在哪）**：levnikolaevich 的 18 个 SKILL.md 均无独立的 `## Examples` 章节（0 个完整 input→output 示例），只有 Checklist 和 Tool Routing 表
- **意义**：「有 examples 让用户知道期望的输出格式」是我胜出的维度；若对 levnikolaevich 提 PR，添加 examples 是最有价值的贡献

---

## 八、术语表

### <a name="轻量技能套件"></a>轻量技能套件（Lightweight Skill Suite）
> 把多个相关的独立技能组合成一个「套件」（suite），可以作为一个整体安装，每个技能仍然独立自包含。和「重型工作流框架」的区别在于：套件里没有统一的 orchestrator（编排器），技能之间不互相调用，用户自己决定何时用哪个。

### <a name="Tool-Routing"></a>工具路由表（Tool Routing）
> SKILL.md 里的一个结构化章节，用表格形式告诉 Claude：「这个任务 → 优先用这个工具 → 满足什么条件时用 → 如果该工具不可用用什么替代」。比 `allowed-tools:` 字段更精确，因为它不只列了「可以用的工具」，还说明了「什么时候用」。

### <a name="编排层"></a>编排层（Orchestration Layer）
> 在多个 agent/skill 之上的一层「总指挥」，负责决定哪些 agent 什么顺序执行、如何传递结果。levnikolaevich v1 有这层（`ln-1000-pipeline-orchestrator`），v2 把它完全移除了。现代 Claude 模型足够聪明，不需要被编排器一步步告知下一步做什么。

### <a name="自包含设计"></a>自包含设计（Self-contained Design）
> 一个 SKILL.md 文件包含了 Claude 执行该技能所需的一切信息：触发条件、工具路由、执行指导、完成标准。不依赖仓库里的其他文件（除了 CLAUDE.md 的基础配置）。对比：levnikolaevich v1 的 skills 依赖 `shared/agents/prompt_templates/` 下的公共模板，属于「外部依赖设计」。

### <a name="幂等性"></a>幂等性（Idempotent）
> 执行多次和执行一次结果相同的性质。levnikolaevich 的 `ln-42-acceptance-test-builder` 要求「测试可以重跑且不依赖执行顺序」，这就是幂等性要求。在 NL 编程里，幂等性意味着「同样的 skill 触发两次，产生的结果应该相同，不会因为第一次运行留下的状态而影响第二次」。
