# deanpeters/Product-Manager-Skills — 学习案例

**仓库**：https://github.com/deanpeters/Product-Manager-Skills
**Stars**：N/A | **来源**：本地 audit（case_study_candidate=True）
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-07-11（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `examples-driven`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是一个专为产品经理（PM）角色设计的 Claude Code 技能库，收录 56 个 SKILL.md 文件和 6 个命令（command），涵盖从用户研究到路线图规划、PRD 撰写等产品管理全流程。仓库的核心价值不在于功能本身，而在于它把产品管理方法论系统性地编码进了自然语言编程（NLP）工件。

目录结构如下：

```
Product-Manager-Skills/
├── app/               # 应用层
├── catalog/           # 技能目录索引
├── commands/          # 6 个编排命令
│   ├── discover.md
│   ├── leadership-transition.md
│   ├── plan-roadmap.md
│   ├── prioritize.md
│   ├── strategy.md
│   └── write-prd.md
├── dist/              # 分发产物
├── docs/              # 文档
├── research/          # 研究资料
├── scripts/           # 16 个脚本，含适配器
│   └── adapters/      # claude-code.sh, manual.sh 等
├── skills/            # 56 个 SKILL.md，核心内容
│   ├── component/     # 一次性组件技能
│   ├── interactive/   # 迭代对话技能
│   └── workflow/      # 多步骤编排技能
├── CLAUDE.md
└── 04JUL26.md / 08JUL26.md / 15MAY26.md / 26APR26.md  # 按日期命名的更新日志
```

根目录中按日期命名的 Markdown 文件（如 `04JUL26.md`）充当非正式更新日志，这是一种简洁但略显随意的版本追踪方式。

### 1.2 架构剖析

**技能三分类体系**是该仓库最显著的架构决策。技能按认知模式被分为三类：

| 类型 | 英文 | 含义 | 典型例子 |
|------|------|------|---------|
| 组件型 | component | 独立、一次性使用 | `jobs-to-be-done.md` |
| 交互型 | interactive | 需要多轮对话迭代 | `discovery-interview-prep.md` |
| 工作流型 | workflow | 多步骤、跨技能编排 | `skill-authoring-workflow.md` |

**命令作为编排层**：6 个命令通过 `uses:` 字段显式声明依赖的技能，例如 `discover.md` 使用：

```yaml
uses:
  - discovery-process
  - problem-framing-canvas
  - discovery-interview-prep
  - opportunity-solution-tree
  - pol-probe-advisor
```

命令还引入了 `outputs:` 和 `argument-hint:` 字段，让编排意图更加清晰可追溯。

**共享依赖技能**：`workshop-facilitation` 被多个技能通过相对路径引用（`../workshop-facilitation/SKILL.md`），形成了依赖注入式的技能复用。

**适配器模式**：`scripts/adapters/` 目录下的 `claude-code.sh` 和 `manual.sh` 让同一套技能可以在不同 AI CLI（Claude Code、手动模式）上运行，实现了运行环境解耦。

### 1.3 设计思路 / 方法论

该仓库最核心的设计思想是**教学型技能（pedagogic skills）**：每一个技能文件不仅提供操作指引，还包含三个固定子章节解释"为什么"：

- **Why This Works**（为什么有效）：从理论或实践层面说明方法的原理
- **Anti-Patterns**（反模式）：列举常见错误做法
- **Common Pitfalls**（常见陷阱）：指出容易踩坑的边界情况

这一模式让技能从"操作手册"升格为"学习材料"，使用者在执行任务的同时吸收方法论。它体现了一种元认知设计哲学：工具本身就是教师。

另一个设计思想是**自我引导（self-referential）**：仓库包含 `skill-authoring-workflow` 技能，教导如何创建新技能，实现了方法论的自我扩展。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

从该仓库提炼出以下可复用的架构模式：

**模式 A：技能类型分层（Skill-Type Stratification）**
将技能按认知复杂度分层（component → interactive → workflow），避免单一扁平目录导致使用者无法判断调用方式。

**模式 B：教学嵌入式技能（Pedagogic Skill Embedding）**
在操作内容之外固定嵌入 Why/Anti-Patterns/Pitfalls 三节，让技能文件同时充当学习材料。

**模式 C：命令显式依赖图（Command Dependency Manifest）**
命令通过 `uses:` 字段声明技能依赖，形成可读性强的依赖图，便于维护和调试。

**模式 D：适配器脚本解耦（Adapter Script Decoupling）**
通过轻量 shell 适配器将技能执行与 AI CLI 具体实现解耦，支持多平台运行。

**模式 E：自我扩展技能（Self-Extension Skill）**
将技能创作流程本身编码为一个技能，实现方法论的自我传播。

### 2.2 适用场景

| 模式 | 适用场景 |
|------|---------|
| 技能类型分层 | 技能数量超过 20 个，使用者需要快速判断调用方式 |
| 教学嵌入式技能 | 面向团队共享的技能库，新成员需要在使用中学习 |
| 命令显式依赖图 | 命令编排 3 个以上技能，需要可维护性 |
| 适配器脚本解耦 | 技能需要在多种 AI 工具或手动场景下运行 |
| 自我扩展技能 | 技能库需要持续增长，且多人参与贡献 |

### 2.3 与其他架构对比

与典型的单层技能库（如 `MarkQWu/gstack`，仅有扁平的 skills/ 目录）相比，该仓库的分层体系带来显著的可导航性提升。代价是目录维护成本：当技能从 interactive 演变为 workflow 时，需要移动文件。

与命令型插件（如 `MarkQWu/bureau`）相比，该仓库的命令更像"工作流调度器"而非"单一操作执行器"，适合复杂领域（PM 工作本身就是多步骤、高语境的）。bureau 命令风格更接近工具函数，deanpeters 命令更接近业务流程。

与剧本式技能库（如 `MarkQWu/drama-workshop-skills`）相比，两者都是面向特定社区的策划型技能集，但 deanpeters 的结构规范程度更高，技能模板更统一。

### 2.4 改进空间

1. **`allowed-tools` 缺失**：6 个命令均未声明工具白名单，运行时权限边界不清晰（详见第三节）。
2. **空输入处理缺位**：命令未处理用户不提供参数的情况，健壮性不足。
3. **根目录日期文件**：`04JUL26.md` 等文件作为更新日志是非正式做法，应迁移到结构化 CHANGELOG 或 git tag。
4. **Substack 占位符残留**：部分技能中仍有 `[Link to relevant Dean Peters' Substack articles if applicable]` 占位文本，表明发布前检查流程不严格。

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：89/100**，审查覆盖 55 个工件。

| 工件类型 | 当时分数 | 主要扣分原因 |
|---------|---------|------------|
| CLAUDE.md | 50 | 无 NL frontmatter（-50） |
| commands/README.md | 50 | 无 command frontmatter（-50） |
| 6 个命令 | 85 | 缺 `allowed-tools`（-5 每个）；无空输入处理（-10 每个） |
| ~13 个技能 | 90 | 无 `best_for`/`scenarios` 元数据；部分含 Substack 占位符 |
| ~27 个技能 | 91 | 几乎完美，细微元数据缺失 |
| 顶尖技能（如 `executive-onboarding-playbook`） | 95 | 近乎无可挑剔 |

安全评级：**CLEAR**（发现 3 个 Medium 级别问题，无 Critical/High）。

Medium 问题集中在脚本层：
- `run-pm.sh` 将用户输入传递给 claude/codex CLI，未做充分隔离
- `add-a-skill.sh` 动态 `source` 来自 `ADAPTERS_DIR` 的适配器脚本，路径未严格校验

整体运营风险低，因为这些脚本定位为本地开发工具，非服务端执行。

### 3.2 当时值得借鉴的模式

1. **教学三件套（Why / Anti-Patterns / Pitfalls）**：是当时审查所见同类技能库中罕见的设计，显著提升技能的自解释能力。
2. **技能类型分类目录**：`component/interactive/workflow` 三分类在结构清晰度上优于多数平铺式技能库。
3. **顶尖技能的完整度**：`executive-onboarding-playbook` 等技能的 frontmatter 完整、描述精确、适用场景清晰，是技能写作的示范级样本。
4. **命令编排意图明确**：即便当时 `uses:` 字段尚未普及，命令的编排逻辑已通过说明文本较清晰地表达。

### 3.3 当时的缺陷

1. **`allowed-tools` 全面缺失**：6 个命令无一声明 `allowed-tools`，累计扣分 -30（每个 -5）。
2. **空输入处理全面缺失**：6 个命令均未处理无参数调用场景，累计潜在扣分 -60（每个 -10）。
3. **CLAUDE.md 无 NL frontmatter**：导致该文件被评为 50 分，显著拉低整体均值。
4. **Substack 占位符未清理**：`user-story`、`problem-statement`、`positioning-statement`、`jobs-to-be-done` 四个技能中含有模板占位文本，表明发布流程存在遗漏。
5. **`commands/README.md` 格式不合规**：无 command frontmatter，被评为 50 分。

### 3.4 当时的优化机会

1. 为 6 个命令补充 `allowed-tools` 字段，明确声明运行时工具权限。
2. 在命令中添加空输入分支逻辑，处理用户未提供参数时的引导提示。
3. 为 CLAUDE.md 添加合规 NL frontmatter。
4. 替换或删除 Substack 占位符，填入真实链接或移除该章节。
5. 将 `commands/README.md` 改写为合规的 command 格式或改为普通文档。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

基于 2026-07-11 对当前 HEAD 的抽查：

| 缺陷 | 2026-04-07 状态 | 2026-07-11 状态 | 变化 |
|------|--------------|--------------|------|
| `allowed-tools` 缺失 | 全部 6 个命令缺失 | 仍全部缺失 | **未修复** |
| 空输入处理缺失 | 全部 6 个命令缺失 | 未见修复证据 | **未修复** |
| CLAUDE.md 无 frontmatter | 缺失 | 未单独验证 | 状态不明 |
| Substack 占位符 | 4 个技能有 | `user-story`、`problem-statement`、`positioning-statement`、`jobs-to-be-done` 仍有；`pol-probe`、`pol-probe-advisor`、`ai-shaped-readiness-advisor` 已填入真实 URL | **部分修复** |

最关键的结构性问题（`allowed-tools`）在三个月内未获修复，说明该仓库的主要精力投入在内容扩充而非合规修复。

### 4.2 架构演进

自 2026-04-07 以来，可观察到以下演进：

- **技能数量增长**：47 → 56，新增 9 个技能（增幅约 19%）
- **命令新增结构化字段**：`uses:`、`outputs:`、`argument-hint:` 三个字段现已在命令中普遍出现，这是编排成熟度的显著提升
- **Substack 内容逐步落实**：部分技能的外部参考链接从占位符变为真实 URL，说明内容持续迭代

`uses:` 字段的引入尤其值得关注——它将命令与技能的依赖关系从隐式（靠文字描述）变为显式（机器可读），为未来的依赖图分析奠定基础。

### 4.3 新增的可学习模式

**`argument-hint:` 字段**：为命令提供参数提示，减少使用者的认知负担。这是小巧但实用的 UX 改进，值得在自己的命令中借鉴。

**`outputs:` 字段**：明确声明命令的输出物，使命令在流水线中的位置和价值更清晰。这是从"命令"到"工作流节点"思维转变的体现。

---

## 五、校准

### 5.1 我已经在做对的

通过对比该仓库，以下是已经做得不错的方面：

- **技能文件结构清晰**：`MarkQWu/drama-workshop-skills` 中的技能同样有明确的操作指引，不是纯粹的文字堆砌。
- **专注特定领域**：与 deanpeters 专注 PM 技能一样，drama-workshop-skills 专注戏剧创作，领域专注度高。
- **使用命令组织工作流**：`MarkQWu/bureau` 已经采用命令层来组织功能，与 deanpeters 的编排思路一致。

### 5.2 挑战 / 验证

以下判断需要进一步验证：

- **我的技能是否真的缺少"教学三件套"**：需要逐一检查 drama-workshop-skills 和 gstack 的技能文件，确认是否含有 Why / Anti-Patterns / Pitfalls 子节。
- **bureau 命令的 `allowed-tools` 问题**：已有初步判断（缺失），需要实际运行 grep 确认（见 §六）。
- **类型分层的必要性**：gstack 技能数量是否已达到需要 component/interactive/workflow 分层的阈值？

---

## 六、行动

### 6.1 自查动作

以下 bash 命令用于检查自有仓库中的相同问题：

```bash
# 检查 bureau 命令是否缺少 allowed-tools 字段
grep -rn "allowed-tools" ~/projects/MarkQWu-bureau/commands/ 2>/dev/null \
  || echo "MISSING: bureau commands 缺少 allowed-tools"

# 检查 bureau 命令是否有空输入处理
grep -rn "empty\|no.*input\|without.*argument\|未提供\|空输入" \
  ~/projects/MarkQWu-bureau/commands/ 2>/dev/null | head -10

# 检查 drama-workshop-skills 技能是否含教学三件套
grep -rn "Why This Works\|Anti-Patterns\|Common Pitfalls" \
  ~/projects/MarkQWu-drama-workshop-skills/skills/ 2>/dev/null | head -20

# 检查 gstack 技能数量及类型分层情况
ls ~/projects/MarkQWu-gstack/skills/*.md 2>/dev/null | wc -l
ls ~/projects/MarkQWu-gstack/skills/ 2>/dev/null

# 检查 echo-sleuth-for-claude 的 agent/skill 组合是否有依赖声明
grep -rn "uses:" ~/projects/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null | head -10

# 检查自有命令中是否有 argument-hint 和 outputs 字段
grep -rn "argument-hint:\|outputs:" ~/projects/MarkQWu-bureau/commands/ 2>/dev/null
```

### 6.2 灵感 → 实施路径

**优先级 1（立即可行）：为 bureau 命令补充 `allowed-tools`**

参考 deanpeters 的缺陷，bureau 命令可能存在相同问题。修复步骤：
1. 运行上述 grep 确认缺失
2. 确定每个命令实际调用了哪些工具（Read、Edit、Bash、WebFetch 等）
3. 在命令 frontmatter 中添加 `allowed-tools:` 列表
4. 同时补充空输入处理分支

**优先级 2（下一个技能）：引入教学三件套**

在 `drama-workshop-skills` 的下一个新技能中试用 Why / Anti-Patterns / Pitfalls 结构，观察是否提升技能的自解释能力。若效果好，逐步回填到现有技能。模板：

```markdown
## Why This Works
（方法论依据，1-3 句）

## Anti-Patterns
- 反模式 1：描述 + 危害
- 反模式 2：描述 + 危害

## Common Pitfalls
- 陷阱 1：触发条件 + 识别方法
- 陷阱 2：触发条件 + 识别方法
```

**优先级 3（中期）：为命令添加 `outputs:` 和 `argument-hint:`**

借鉴 deanpeters 的新增字段，在 bureau 命令中补充这两个字段，提升命令的可读性和流水线可组合性。

**优先级 4（长期）：评估 gstack 是否需要类型分层**

当 gstack 技能超过 20 个时，考虑引入 component/interactive/workflow 分层目录。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | deanpeters 对应维度 | 对齐度 |
|---------|------------------|-------|
| `MarkQWu/bureau` | 命令编排层 | 高——同为命令驱动型插件 |
| `MarkQWu/gstack` | 技能库层 | 中——同为技能集合，但缺乏类型分层 |
| `MarkQWu/drama-workshop-skills` | 领域专用技能库 | 高——同为面向特定社区的策划型技能集 |
| `MarkQWu/echo-sleuth-for-claude` | agent + skill 组合 | 中——结构更接近 agent 驱动而非命令驱动 |

### 8.2 在我的项目里复现的同类问题

**bureau 命令缺少 `allowed-tools`（与 deanpeters 完全相同的问题）**

deanpeters 的 6 个命令在审查时因缺少 `allowed-tools` 被扣分。初步判断 bureau 的命令存在完全相同的问题——命令声明了操作意图，但没有显式列出允许调用的工具。这不仅影响 NLPM 评分，更重要的是在实际运行时会导致权限边界模糊，用户无法预期命令会访问哪些工具。

**drama-workshop-skills 技能缺少"教学三件套"**

drama-workshop-skills 的技能文件普遍缺少 Why This Works / Anti-Patterns / Common Pitfalls 三个子节。与 deanpeters 的技能相比，使用者只能知道"做什么"，不能从技能文件本身理解"为什么这样做"和"哪些做法是错的"。

**gstack 缺少类型分层**

gstack 的 skills/ 目录是单层平铺结构，没有区分一次性技能、交互式技能和工作流技能。当技能数量增长后，使用者将难以判断调用方式。

### 8.3 别人的更优方案

**deanpeters 做得更好的地方**：

1. **教学型技能设计**：每个技能都是一个微型学习单元，而不只是操作手册。这种设计让团队共享技能库时的上手成本大幅降低。
2. **命令依赖显式化**：`uses:` 字段让命令的依赖关系一目了然，维护者可以快速判断修改一个技能会影响哪些命令。
3. **技能类型分层**：component/interactive/workflow 三分类让使用者在看到技能名称时就能预判调用模式。
4. **自我扩展技能**：`skill-authoring-workflow` 技能本身就是方法论的一部分，贡献者无需查阅外部文档。
5. **适配器模式**：通过 shell 适配器支持多平台运行，解耦了技能与运行环境，值得在跨工具技能库中复用。

### 8.4 反向：我的项目做得比他们好的地方

1. **`allowed-tools` 合规性**：若 bureau 命令已声明 `allowed-tools`（待验证），则在这一维度上优于 deanpeters。
2. **发布前内容清洁度**：若自有技能库中无占位符残留，则内容质量管控优于 deanpeters（其 Substack 占位符在三个月后仍有残留）。
3. **CLAUDE.md frontmatter 合规性**：若自有项目的 CLAUDE.md 已包含合规 NL frontmatter，则在工件格式规范上优于 deanpeters（其 CLAUDE.md 评分仅 50）。
4. **echo-sleuth 的 agent 层**：`MarkQWu/echo-sleuth-for-claude` 拥有 agent 层，比 deanpeters 纯技能+命令架构有更强的自动化编排能力。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| `allowed-tools`（工具白名单） | Claude Code command frontmatter 中的一个字段，列出该命令在执行时被允许调用的工具名称（如 `Read`、`Edit`、`Bash`、`WebFetch`）。缺少此字段时，权限边界由运行时环境决定，而非命令作者明确声明，可能导致意外的工具调用或权限提升。 |
| `frontmatter`（前置元数据） | Markdown 文件开头由 `---` 包围的 YAML 块，用于声明工件的结构化元数据（如 `description:`、`argument-hint:`、`uses:` 等）。Claude Code 和 NLPM 依赖 frontmatter 理解工件的类型和能力。 |
| 适配器模式（Adapter Pattern） | 一种软件设计模式，通过引入"适配器"层将接口不兼容的组件连接起来。在该仓库中，`scripts/adapters/` 下的 shell 脚本充当适配器，让同一套技能可以在 Claude Code CLI、OpenAI Codex CLI、手动模式等不同运行环境中调用，而无需修改技能文件本身。 |
| 教学型技能（Pedagogic Skill） | 在操作指引之外，额外嵌入 Why This Works / Anti-Patterns / Common Pitfalls 三个子节的技能设计模式。目标是让技能文件同时充当操作手册和学习材料，降低团队共享技能库的认知门槛。 |
| `uses:`（依赖声明字段） | Claude Code command frontmatter 中声明该命令所依赖技能列表的字段。使依赖关系从隐式（文字描述）变为显式（机器可读），便于依赖分析和维护。 |
| `argument-hint:`（参数提示字段） | Command frontmatter 中提示用户应传入何种参数的字段，减少使用者在调用命令时的认知负担。 |
| `outputs:`（输出声明字段） | Command frontmatter 中明确列出命令产出物的字段，使命令在工作流中的位置和价值更清晰，支持流水线式组合。 |
| NL 工件（NL artifact） | 自然语言编程（Natural Language Programming）工件，指以 Markdown 格式编写、供 AI 系统（如 Claude Code）读取和执行的技能、命令、agent 等文件。 |
| component / interactive / workflow | deanpeters 仓库的技能三分类体系。component（组件型）：独立、一次性使用；interactive（交互型）：需要多轮对话迭代；workflow（工作流型）：多步骤、跨技能编排。 |
