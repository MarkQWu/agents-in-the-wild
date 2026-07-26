# deanpeters/Product-Manager-Skills — 学习案例

**仓库**：https://github.com/deanpeters/Product-Manager-Skills
**Stars**：N/A（注册表未记录）| **来源**：upstream audit（case_study_candidate=true）
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-07-26（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `vague-quantifier`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
一个以产品经理（PM）方法论为核心的大型 [NL 技能库](#NL-技能库)，由产品教练 Dean Peters 本人从自己的播客、文章和培训内容「蒸馏」而来。仓库当前（v0.83）有 **70 个技能 + 6 个命令**，覆盖 PM 的全职能生命周期：从产品发现到市场情报、财务分析到高管职业过渡。

关键事实：
- License 为 CC BY-NC-SA 4.0——开放分享但不可商用
- 技能分四类：Component（模板/工件）/ Interactive（引导式对话）/ Workflow（端到端流程）/ Meta（技能创作工具自身）
- 每个技能有 `theme:` 字段（10 个主题），支持按领域快速定位
- `catalog/` 目录提供机器可读的 YAML 索引（由 `scripts/generate-catalog.py` 生成），方便自动化处理
- 2026-07-17 刚完成「市场情报套件」（Phase 9，14 个新技能），是 audit 之后的最大一次扩展

### 1.2 架构剖析

**目录结构：**
```
CLAUDE.md                        ← 项目状态 + 技能蒸馏协议
skills/
  <skill-name>/
    SKILL.md                     ← 技能正文
    template.md                  ← 可选：提取的输出模板（v0.83 新增）
commands/
  <command>.md                   ← 命令文件（6 个）
  README.md                      ← 命令说明（非 NL 工件格式）
catalog/
  skills-index.yaml              ← 机器可读技能元数据索引
  commands-index.yaml            ← 机器可读命令索引
  skills-by-type.md              ← 按类型分类的可读视图
  commands.md                    ← 命令目录
scripts/
  add-a-skill.sh, build-a-skill.sh, ...  ← 脚本工具集（16 个）
app/main.py                      ← Streamlit 本地演示应用（beta）
```

**文件类型分布：** 70 SKILL.md + 6 命令 + 约 16 个 `template.md`（新增）+ 1 CLAUDE.md + YAML 索引

**编排关系：**
- 命令层（6 个）分别引用多个技能（通过 `uses:` 列表声明）；无路由层，命令直接加载技能
- 市场情报套件引入「链式技能」：`market-landscape-scan` → `competitive-research-snapshot` → `competitive-intel-watch` → `battle-card-builder`，每个技能消费上一个的稳定输出模式
- `workshop-facilitation` 作为「协议技能」被多个交互式技能依赖（在其 `## References` 中引用）
- `autonomous-investigation` 新增为「自主研究协议」，扮演类似 `workshop-facilitation` 在调查类技能中的角色

**跨件契约：**
- 命令通过 `uses:` 字段声明调用的技能——审计显示所有引用均解析正确，无孤儿引用
- CLAUDE.md 列出「计划中」但未创建的技能（3 个，明确标注「Planned」，非断链）
- `template.md` 与对应 SKILL.md 同目录，提供提取好的输出模板（可读工件，不是代码）

### 1.3 设计思路 / 方法论

**核心设计哲学：** 「方法论蒸馏」——把人类专家（Dean Peters 本人）的认知模型提炼成 AI 可执行的结构化知识，而不只是「AI 帮你写 PRD」。每个技能都包含「为什么这样做」（Why This Works、Anti-Patterns）的认知层面内容。

**解决什么问题：** PM 工具箱散落在书籍、播客、课程里，每次用要重新回忆框架。这个库把「需要时手头就有」的知识变成可召唤的 AI 技能。

**Trade-off：**
- **深度 vs. 广度**：技能内容非常丰富（最大的超过 10K tokens），不适合「快速调用」，更像是「深度辅助」
- **独立仓库 vs. 内嵌 CLAUDE.md**：选择独立仓库 + claude plugin install，保持可被他人安装
- **命令薄 vs. 命令厚**：命令极简（只列 `uses:` 技能，无自己的逻辑），所有行为在技能里

**认知模型：** Dean Peters 把 Claude Code 技能视为「可编程的认知伙伴」，而非「自动化代替人」。技能的首要任务是提问、对齐、引导，而非直接输出答案。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「主题化技能库 + 链式工作流」**

关键特征：
- 技能按 `theme` 聚合，每个主题构成一套协作的认知工具（如市场情报套件）
- 有「协议技能」（`workshop-facilitation`、`autonomous-investigation`）充当其他技能的隐式依赖
- 命令是技能的「组合快捷方式」，不含自己的逻辑
- 链式技能之间通过「稳定输出模式」（stable diffable schema）传递数据
- `catalog/` 机器索引是自动化消费的接口层

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域专家想把自己的方法论 AI 化 | ✅ 高度适用 | 正是这个库解决的问题 |
| 需要跨多个相关技能联动完成复杂任务 | ✅ 适用 | 链式技能 + `uses:` 声明的命令组合 |
| 希望命令层有控制流或条件逻辑 | ❌ 不适用 | 命令极薄，无法表达 if/else |
| 追求快速简洁、上下文最小化 | ❌ 慎用 | 技能文件很大，加载成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 主题化技能库 + 链式工作流 | 本仓库 | 深度方法论封装，链式技能相互传递稳定输出 | 技能体量大，命令无控制流 |
| 瘦命令 + 条件代理流水线 | xiaolai/loc-guardian-for-claude | 控制流清晰，模型分层 | 单用途，不易扩展多技能 |
| monorepo 平铺 SKILL.md | MarkQWu/gstack | 技能内嵌项目，易维护 | 跨项目共享难，无独立生命周期 |

### 2.4 改进空间

1. **当前问题**：6 个命令全无 `allowed-tools` 声明，调用工具无法被约束。**改进做法**：在每个命令 frontmatter 加 `allowed-tools: Read, Agent`（命令只读技能文件 + 分派 Agent）。**预期收益**：限定命令的工具访问范围，审计时更清晰。

2. **当前问题**：命令无空输入处理，`$ARGUMENTS` 为空时行为未定义（audit 扣 -10×6=-60 分的最大来源）。**改进做法**：每个命令开头加一行 `If called with no arguments, ask the user: ...`。**预期收益**：用户裸调用命令时获得引导而非沉默。

3. **当前问题**：`commands/README.md` 不是 NL 工件格式（无 frontmatter），自动化工具无法解析。`catalog/commands.md` 已经承担了其功能。**改进做法**：删除 `commands/README.md`，或将其迁移到 `catalog/` 目录。**预期收益**：commands/ 目录纯净，全部为可被 Claude Code 加载的命令文件。

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-07 当时得分 **89/100**，55 个工件。

| 文件类型 | 典型分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 50 | 无 NL frontmatter |
| commands/README.md | 50 | 无命令 frontmatter |
| 6 个命令文件 | 85 | 缺 `allowed-tools`、无空输入处理 |
| 11 个技能（低档） | 90 | 无 `best_for`/`scenarios` 元数据 |
| 大多数技能 | 91 | 基本无问题 |
| 高档技能 | 93-95 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **完整的认知层内容** → 每个技能都有「Why This Works」、「Anti-Patterns」、「Common Pitfalls」章节，不只是步骤清单。路径：任意 `skills/*/SKILL.md`。如何借鉴：在自己的技能里加「不要这样做」和「为什么」的反模式章节，减少模型误用。

2. **全局协议技能** → `workshop-facilitation` 作为交互式技能的共享协议，被多个技能在 `## References` 中引用。这不是代码继承，而是「文档引用即约定」。如何借鉴：当多个技能共享同一种交互模式时，提取为一个协议技能，其他技能在 References 里声明依赖。

3. **Workflow 技能链** → `prd-development` workflow 技能依赖 `proto-persona`、`user-story` 等多个 Component 技能，形成有向依赖图。技能之间的依赖关系通过 `## References` 明确声明，不是隐式假设。如何借鉴：在设计多步骤 workflow 时，把子步骤拆成独立 Component 技能，workflow 技能只做编排，不重复知识。

4. **Case-study_candidate 标志** → 注册表中 `case_study_candidate: true`，说明这个仓库在 pipeline 中被标记为「值得深度学习」。结合 PR #8 已合并、PR #7 被拒的记录，这个仓库提供了真实的「PR 命运」数据。

5. **`best_for` / `scenarios` 元数据** → 高分技能（≥91）都有这两个字段，帮助 AI 在被问「应该用哪个技能」时做出准确匹配。

### 3.3 当时的缺陷

1. **命令缺 `allowed-tools`（6 个命令，-5 × 6 = -30）** → 根本原因：命令文件的 `uses:` 声明了调用哪些技能，但没有声明技能执行时可以使用哪些工具。自查：我的 bureau/gstack 命令有没有 `allowed-tools`？

2. **命令无空输入处理（6 个命令，-10 × 6 = -60，最大扣分来源）** → 根本原因：`/pm:prioritize` 被空调用时，模型不知道该询问什么信息还是自由发挥。对交互式技能来说，引导用户提供必要上下文至关重要。自查：我的命令有没有 `If called with no argument` 的处理逻辑？

3. **占位符 Substack 引用（3 处）** → 根本原因：技能编写时为了格式完整留了占位符，但忘记填写或删除。「[Link to relevant Dean Peters' Substack articles if applicable]」这样的占位符会让用户困惑是否应该去找对应文章。自查：我的技能里有没有 TODO / [placeholder] / [Link to ...] 这样的未完成标记？

### 3.4 当时的优化机会

1. 为 6 个命令添加 `allowed-tools` 声明（`Read, Agent` 应覆盖大多数场景）
2. 为 6 个命令添加空输入兜底：一行 `If called with no arguments, ask the user for [XXX] before proceeding`
3. 删除或填写 3 个 Substack 占位符引用

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查命令 | 现状 | 含义 |
|---|---|---|---|
| 6 命令无 `allowed-tools` | `grep -rL "allowed-tools" commands/*.md` | ❌ **全部仍存在**：7 个命令文件（新增 1 个），全无 `allowed-tools` | 作者未跟进此建议 |
| 6 命令无空输入处理 | `grep -rl "\$ARGUMENTS\|no arguments" commands/*.md` | ❌ **仍存在**：`$ARGUMENTS` 只在 `argument-hint` 里出现，无业务处理逻辑 | 同上 |
| Substack 占位符（3 处） | `grep -rn "if applicable" skills/` | ⚠️ **部分改善**：`pol-probe-advisor` 已填写真实 URL；`user-story`、`problem-statement`、`positioning-statement` 仍有占位符 | 被 PR 改了一部分，其余遗留 |
| 技能缺 `best_for`/`scenarios` | `grep -rL "best_for" skills/*/SKILL.md` | ✅ **全部修复**：所有 70 个技能现在都有 `best_for` 和 `scenarios` 字段 | 作者后续做了全面补充 |
| `skills/ai-shaped-readiness-advisor/SKILL.md` >10K tokens | `wc -c` | 🔍 **无法 verify**（需要实际 token 计数）| 该文件后续是否拆分未知 |

### 4.2 架构演进

Audit 时：55 工件（47 技能 + 6 命令 + CLAUDE.md + commands/README.md）
当前 HEAD：70 技能 + 6 命令 + `catalog/` + 约 16 个 `template.md` + `app/main.py`

主要变化：
1. **Phase 9 市场情报套件**（14 新技能）：引入「链式技能」模式 + `autonomous-investigation` 协议技能
2. **`catalog/` 目录**：新的机器可读索引层（YAML + 生成脚本），使技能库可被程序消费
3. **`template.md`**：从技能文件中提取的输出模板，降低「我需要什么格式的输出」的认知负担
4. **`theme:` 字段**：从语义标签变成正式 frontmatter 字段，10 个主题已系统化
5. **命令 `uses:`/`outputs:` 字段**：命令的元数据更丰富，但仍无 `allowed-tools`

这说明 Dean Peters 意识到了「技能可发现性」和「技能可组合性」，并通过 catalog + theme + 链式技能在架构上应对。但命令层的「NLPM 规范合规性」不是他的关注点。

### 4.3 新增的可学习模式

1. **链式技能的稳定输出模式（Stable Diffable Schema）**：`autonomous-investigation` 协议要求调查技能产出可以跨次运行 diff 的输出——不只是「本次结论」，而是「每次运行可相互比较的结构化快照」。这是把 AI 输出纳入「版本控制语境」的设计思路。

2. **`template.md` 作为工件提取**：技能文件描述「做什么」，`template.md` 提供「产出什么格式」的空白框架。用户可以在 Claude 完成分析后，直接用 `template.md` 作为填写模板，而不必从输出中提取格式。

---

## 五、校准

### 5.1 我已经在做对的

1. `MarkQWu/gstack` 的 SKILL.md 有 `allowed-tools` 声明——比 deanpeters 的命令文件做得更好
2. `MarkQWu/bureau` 的技能文件有 `description` 字段，基本符合 NL frontmatter 要求
3. `MarkQWu/gstack` 的 SKILL.md 有具体的 `<example>` 块——比 deanpeters 的技能更具可测试性
4. `MarkQWu/bureau` 的 `CLAUDE.md` 有架构说明（但无 frontmatter）——与本仓库 CLAUDE.md 水平相近

### 5.2 挑战 / 验证

**验证**：我之前对「`catalog/` 目录和 YAML 索引」这类「元层次索引」的价值有些怀疑（是不是过度工程？）。这个库验证了它的价值——当技能数量超过 50 个时，如果没有机器可读的索引，「自动找到最适合当前任务的技能」会变得很难。`scripts/generate-catalog.py` 是维护这个索引的关键：索引不是手写的，是脚本生成的，所以不会跟实际文件脱节。

**挑战**：`case_study_candidate: true` 在注册表中被显式设置了。我以前认为「哪个仓库值得深度学习」是主观判断，但实际上 pipeline 是通过观察「有 PR 被合并」+「有 PR 被拒绝」来自动标记候选案例的。被拒绝的 PR（#7）和被接受的 PR（#8）放在一起，正好说明了「什么样的安全修复是维护者愿意接受的」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 bureau 和 gstack 的命令是否有空输入处理
grep -rn "no arguments\|if.*empty\|ARGUMENTS.*empty\|\$ARGUMENTS" \
  /tmp/my-repos/MarkQWu-bureau/commands/ \
  /tmp/my-repos/MarkQWu-gstack/ \
  --include="*.md" 2>/dev/null | head -10
```
命中数为 0：为命令文件添加一行「If called with no $ARGUMENTS, ask the user: [specific question]」

```bash
# 检查我的仓库有没有占位符未填写的引用
grep -rn -i "\[link to\]\|\[placeholder\]\|if applicable\]\|TODO\|FIXME" \
  /tmp/my-repos/MarkQWu-bureau/ \
  /tmp/my-repos/MarkQWu-gstack/ \
  --include="*.md" 2>/dev/null | grep -v "node_modules\|.git" | head -10
```
命中后：填写真实内容或删除占位符。

```bash
# 检查 bureau 技能是否有 best_for / scenarios 字段
grep -rL "best_for\|scenarios" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后：在缺少的技能 frontmatter 中加 `best_for:` 和 `scenarios:` 列表（各 2-3 条）。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 建一个 `catalog/` 目录，提供技能 + 命令的 YAML 索引
   - **为何可行**：bureau 现在有 7 个技能 + 10 个命令，查找最适合任务的技能已经开始依赖记忆
   - **第一步**：写一个 10 行 Python 脚本，遍历 `skills/*/SKILL.md`，提取 frontmatter 的 `name`/`description`/`type`，输出 `catalog/skills-index.yaml`

2. **想法**：为 bureau 的长流程命令（如 `compile`）提取 `template.md`，定义输出格式
   - **为何可行**：compile 命令的输出格式现在是隐式的（在技能正文里描述），用户不知道最终产物长什么样
   - **第一步**：打开 `skills/compile/SKILL.md`，找到输出格式描述，提取为 `skills/compile/template.md`

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（MarkQWu/gstack, MarkQWu/bureau, MarkQWu/graphify）

### 7.1 目的对齐度

- **本案例核心目的**：PM 方法论蒸馏为可调用技能库

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同样是 NL 技能库、命令文件编排技能 | bureau 是知识管理工具，deanpeters 是 PM 方法论库 | 高（命令/技能元数据模式直接可用） |
| MarkQWu/gstack | 中 | 同样有多个 SKILL.md + 命令 | gstack 是多工具集合，不是领域方法论库 | 中（catalog 模式可借鉴） |
| MarkQWu/graphify | 低 | 同样有 AGENTS.md | graphify 是代码知识图谱，与 PM 方法论无交叉 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令无空输入处理 | `grep -rn "\$ARGUMENTS" commands/*.md` 没有 if-empty 逻辑 | **bureau 命中**：所有 10 个命令文件均无空输入处理 | 高（多个命令调用时会产生困惑） |
| 技能缺 `best_for`/`scenarios` | `grep -rL "best_for" skills/*/SKILL.md` | **bureau 命中 7/7**：所有技能均无此字段 | 中 |
| 命令 README 不是 NL 工件格式 | 检查 `commands/README.md` 是否有 frontmatter | **bureau**：无 commands/README.md（不存在此问题） | 无 |

**命中后的具体行动建议：**
- `MarkQWu/bureau` 的 `commands/capture.md` → 在命令正文第一行加：`If no $ARGUMENTS provided, ask: "What do you want to capture? Paste the content or describe the session."` → 5 分钟可完成
- `MarkQWu/bureau` 的 `skills/recall/SKILL.md` → 在 frontmatter 加：
  ```yaml
  best_for:
    - "Finding past decisions or context from previous sessions"
  scenarios:
    - "I need to know what we decided about X"
  ```
  → 每个技能 5 分钟，7 个技能共 35 分钟

### 7.3 别人的更优方案

1. **领域**：链式技能稳定输出协议（Stable Diffable Schema）
   - **本案例做法**：`skills/autonomous-investigation/SKILL.md` 定义「每次运行产出可互相 diff 的结构化输出」——不是叙述性结论，而是固定列、固定字段的可比较快照；甚至加了 **Fact/Inference/Assumption** 三级标签区分确定性
   - **我的项目现状**：`MarkQWu/bureau` 的 `skills/compile/SKILL.md` 输出格式未明确规定，每次编译的 gazette 结构依赖技能正文描述，没有可 diff 的固定 schema
   - **如何借鉴**：在 bureau 的 `compile` 技能里加 `## Output Format` 节，规定 gazette 每条条目的必填字段（date, title, type, summary, confidence_level），这样跨会话编译结果可以被机器比较

2. **领域**：`theme:` 字段系统化组织大型技能库
   - **本案例做法**：10 个主题标签（market-intelligence, career-leadership, pm-artifacts…）让用户可以说「给我找 market-intelligence 类的技能」而不是靠记名字
   - **我的项目现状**：`MarkQWu/bureau` 和 `MarkQWu/gstack` 的技能都无 `theme:` 字段，超过 10 个技能时会依赖用户记忆
   - **如何借鉴**：在 bureau 的 skills frontmatter 加 `theme:` 字段，当前 7 个技能可分为：`ingestion`（capture/compile）/ `review`（review/lint）/ `query`（recall/guide）/ `meta`（scribe）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：NL 工件的可测试性（`<example>` 块与验证机制）
- **我的做法**：`MarkQWu/gstack` 的 SKILL.md 有具体的 `<example>` 输入/输出块，并配有 `bun test` 可运行的技能校验测试（`bun run skill:check` 提供健康仪表盘）
- **本案例做法**：deanpeters 的技能有 `scenarios:` 字段（描述性的触发场景），但没有 `<example>` 块展示具体输入→输出，也无自动化测试
- **意义**：如果对这个库提 PR，可以建议「在技能 frontmatter 里加 `<example>` 块」——这是一个从现有优质基础上改进的具体切入点

---

## 八、术语表

### <a name="NL-技能库"></a>NL 技能库（Natural Language Skill Library）
> 一组用 Markdown 写成的「技能文件」（SKILL.md），每个文件描述一类任务的知识和执行协议，供 Claude Code 通过 `claude plugin install` 安装后作为技能调用。不同于传统代码库，NL 技能库里没有编译执行的代码，全部是 AI 可以「阅读理解并遵照执行」的自然语言指令。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，用于声明元数据（`name`、`description`、`type`、`theme`、`best_for`、`scenarios` 等）。NLPM 等工具通过解析 frontmatter 来「认识」一个技能文件，没有 frontmatter 的文件对工具来说是不可见的。

### <a name="allowed-tools"></a>allowed-tools
> agent/command frontmatter 中的工具访问声明。缺失时技能执行时的工具访问是无约束的；声明后（如 `allowed-tools: Read, Agent`）则限定了该组件实际需要的工具，既是安全约束也是行为契约。

### <a name="链式技能"></a>链式技能（Chained Skills）
> 多个技能形成有序的数据流：技能 A 的输出格式是技能 B 的期望输入格式，技能 B 再将处理结果传递给 C。deanpeters 的市场情报套件是典型例子：`market-landscape-scan` → `competitive-research-snapshot` → `competitive-intel-watch` → `battle-card-builder`，每一步都消费上一步的「稳定输出模式」。

### <a name="协议技能"></a>协议技能（Protocol Skill）
> 一个本身不执行具体任务、但定义「其他技能如何行动」规范的技能文件。deanpeters 中 `workshop-facilitation` 定义了交互式技能的提问协议，`autonomous-investigation` 定义了自主研究技能的行为规范。其他技能在 `## References` 里引用它，隐式继承其约定。
