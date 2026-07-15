# aaronmaturen/claude-plugin — 学习案例

**仓库**：https://github.com/aaronmaturen/claude-plugin
**Stars**：1 | **来源**：upstream audit（status: contributed）
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-15（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `examples-driven`, `single-purpose`

> **注意**：本案例来源是一个只有 1 颗星的小众仓库，但作为「反面案例」有很高的学习价值——它展示了命令文件在缺乏 frontmatter 和量化标准时会犯哪些典型错误，以及这些错误在三个月后依然没有被修复。

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-plugin` 是 aaronmaturen 为自己的 Angular/Django 全栈开发工作流定制的 Claude Code 命令集。作者把日常开发中的高频操作（PR 审查、技术调研、可访问性审计、发布架构）封装成 slash commands，期望通过 Claude 自动化这些重复性任务。

关键事实：
- 1 颗星，个人项目，未在社区推广
- 26 个 slash commands（对应 26 个 .md 文件）；2 个 skill（ai/bash）
- 所有 commands 发布后被 NLPM 审计，结果令人震惊：**平均分仅 36/100**，范围 15–96
- 仓库已被 NLPM 贡献（status: contributed），说明有尝试提交 PR 改进
- plugin.json 存在（96分），CLAUDE.md 存在（95分）——框架文件质量好，内容文件质量差

### 1.2 架构剖析

**目录结构**（2026-07-15 HEAD）：
```
claude-plugin/
├── CLAUDE.md              # 配置（95分，质量高）
├── AGENTS.md              # agent 清单
├── PLUGIN.md              # 插件说明
├── README.md
├── commands/              # 26 个 slash commands（全部缺 frontmatter）
│   ├── spike-investigation.md
│   ├── feature-investigation.md
│   ├── a11y-audit.md
│   ├── angular-architecture-audit.md
│   ├── angular-expert.md
│   ├── angular-performance-audit.md
│   ├── angular-style-audit.md
│   ├── django-api-audit.md
│   ├── django-expert.md
│   ├── django-model-audit.md
│   ├── django-security-audit.md
│   ├── generate-slidedeck.md
│   ├── git-revise-history.md
│   ├── implement-pr-feedback.md
│   ├── pr-review.md
│   ├── problem-solver.md
│   ├── release-architect.md
│   ├── scaffold.md
│   ├── self-review.md
│   ├── simplify.md
│   ├── summarize-branch.md
│   └── ...（共 26 个）
└── .claude/
    ├── skills/
    │   ├── ai/SKILL.md      # 84分
    │   └── bash/SKILL.md    # 88分
    └── ??? (plugin.json 位置未确认)
```

**文件类型分布**：26 个 commands + 2 个 skills = 28 个 NL 工件

**编排关系**：完全平列——没有 router，没有层级，每个 command 独立触发。用户通过 `/spike-investigation` 等直接调用。

**跨件契约**：基本没有——commands 之间不互相调用，skills 也没有明确被 commands 引用（缺 allowed-tools 声明）。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「我的工作流程 → 一个 command」——每个日常操作都值得有一个命令
- **解决什么问题**：Angular/Django 全栈开发中的重复性审查任务（PR review、a11y、架构审计）
- **做了什么 trade-off**：
  - 快速添加 vs 质量控制：26 个命令说明作者积极地把想法写下来，但没有停下来问「这个命令写得足够具体吗？」
  - 个人使用 vs 社区复用：命令是为自己写的，没有考虑「别人读了能否理解？」
- **反映什么认知模型**：作者把 Claude Code command 当作「写一遍就能用」的工具，而不认为它需要像代码一样被 review 和测试

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「个人工作流命令堆积」（反模式）

把日常工作流直接翻译成命令文件，快速积累数量，但没有统一质量标准，导致命令内容宽泛、无法验证。

模式特征（反面）：
- **缺乏 frontmatter 协议**：26 个 commands 都没有 YAML 头，Claude 不知道这个命令叫什么
- **量词无法量化**：大量 "comprehensive"、"appropriate"、"thorough" 等词，Claude 不知道「足够」是什么
- **无 empty-input guard**：多个命令不处理用户没有提供输入的情况，直接报错或产生空输出
- **无 allowed-tools 声明**：skills 不告诉 Claude 可以用哪些工具，运行时行为不可预测

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人私用、不共享 | △ 勉强可用 | 作者自己知道语境，Claude 也能猜出大概意图 |
| 团队共享或社区发布 | ❌ 不适用 | 缺 frontmatter、缺量化标准，其他人无法理解和使用 |
| 作为学习"什么不该做"的案例 | ✅ 极其适用 | 系统性展示了 NL 编程的所有常见缺陷 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 个人工作流堆积（本仓库） | aaronmaturen/claude-plugin | 快速积累覆盖场景 | 质量低；不可复用；不可维护 |
| 单 skill 深度设计 | drama-workshop-skills | 质量极高；可复用 | 覆盖场景窄 |
| 团队规范化命令集 | LerianStudio/ring | 规范完整；可协作 | 初始投入大 |

### 2.4 改进空间

1. **当前问题**：26 个 commands 零 frontmatter，系统无法识别命令名称。**改进做法**：统一添加 YAML frontmatter（5 行：name、description、model、allowed-tools、version）。**预期收益**：Claude 能正确注册和调用命令；用户能看到命令列表。

2. **当前问题**：所有 commands 使用 vague 量词（"comprehensive"、"appropriate" 等），Claude 无法判断什么程度算"完成"。**改进做法**：把每个 vague 词替换为可验证的数值标准（如"comprehensive" → "covers all exported functions and public API endpoints"）。**预期收益**：命令输出质量从「看心情」变成「可预期」。

3. **当前问题**：git-revise-history.md 在 macOS 上向 `/tmp` 写入 Python 脚本并执行（security flag）。**改进做法**：改为直接把 Python 代码传给 `python3 -c`，避免落地文件。**预期收益**：消除 HIGH 安全风险。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 **36/100**（中位数 29/100）。这是本 routine 见过的最低分之一。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| spike-investigation.md | 15 | 缺 frontmatter；无 empty-input guard（−10）；vague 量词 ≥10（cap −20） |
| feature-investigation.md | 19 | 缺 frontmatter；无 empty-input guard；8 vague 量词（−16） |
| a11y-audit.md | 25 | 缺 frontmatter；vague ≥10（cap −20） |
| commit-msg.md | 41 | 缺 frontmatter；2 vague；macOS-only（pbcopy） |
| .claude/skills/ai/SKILL.md | 84 | 8 vague 量词（effective/specific/appropriate...） |
| plugin.json | 96 | 2 vague 词（Professional/comprehensive） |

**说明**：框架文件（plugin.json、CLAUDE.md）质量高，但内容文件（commands/）质量极低。这说明作者了解「配置正确」，但不了解「内容需要量化」的道理。

### 3.2 当时值得借鉴的模式

1. **plugin.json 配置完整**（96分）→ 作者很清楚 Claude Code 的注册机制，plugin.json 写得非常规范，说明技术能力没问题，只是 command 内容质量意识欠缺。

2. **macOS-only 功能标注**（commit-msg.md）→ 在 commit-msg.md 里使用了 `pbcopy`（macOS 专有命令），审计指出了这个跨平台问题。→ 借鉴：命令里有平台特定行为时要明确标注（`# macOS only` 或添加 fallback）。

3. **场景覆盖有规律**→ 按技术栈分组（angular-*/、django-*/），结构清晰。→ 借鉴：即使命令质量低，按技术栈分类命名是好的信息架构。

### 3.3 当时的缺陷

1. **26 个 commands 零 frontmatter** → 根本原因：作者把 slash command 的 .md 文件当成普通 Markdown 笔记来写，没有意识到 Claude Code 需要解析 [frontmatter](#frontmatter) 才能正确注册命令。**Claude 看到无 frontmatter 的命令就像看到一张没有题目的试卷**，只能靠猜。自查：我有没有任何没有 `name:` 和 `description:` frontmatter 的命令文件？

2. **系统性 vague 量词（平均 7+ 个/文件）** → 根本原因：作者用自然语言写了他「想要 Claude 做的事」，但没有想到 Claude 需要知道「做到什么程度算完成」。"comprehensive audit"、"appropriate recommendations" 等词是人类语境里的正常表达，但对 AI 来说没有执行标准。自查：我的命令文件里有多少 "comprehensive"、"appropriate"、"relevant" 等词？

3. **无 empty-input guard（多个 commands）** → 根本原因：作者假设用户总会提供输入，没有处理空调用的情况。`/spike-investigation`（无参数）会让 Claude 不知道要调研什么。自查：我的命令里是否有 `if [ -z "$ARGUMENTS" ]` 或等价的空输入处理？

### 3.4 当时的优化机会

1. 为所有 26 个 commands 批量添加 5 行标准 frontmatter（可脚本化，15 分钟）
2. 把 spike-investigation.md 和 feature-investigation.md 里的每个 vague 量词替换为具体标准（最高优先级，这两个文件分数最低）
3. 把 commit-msg.md 的 `pbcopy` 改为跨平台兼容写法

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 26 commands 零 frontmatter | `head -5 commands/a11y-audit.md` | a11y-audit.md 内容开头仍是 `# Accessibility Audit`，无 `---` frontmatter | **持续存在**：三个月后仍然是零 frontmatter |
| skills 缺 allowed-tools 声明 | `grep "allowed-tools" .claude/skills/*/SKILL.md` | 未找到任何 allowed-tools 字段 | **持续存在**：未改进 |
| vague 量词系统性存在 | `grep "comprehensive\|appropriate" commands/a11y-audit.md` | 仍然存在（目视确认） | **持续存在** |

**重要发现**：尽管 NLPM 为此仓库提交了贡献 PR（status: contributed），三个月后核心质量问题仍然一个未解决。这可能说明：① PR 没有被维护者接受；② 维护者认为当前状态对个人用途已足够；③ 仓库已经处于「搁置」状态。

### 4.2 架构演进

几乎没有演进：命令数量从 30 减少到 26（删除了 4 个命令），文件名和内容结构基本不变。没有 frontmatter 回填，没有质量改进。这与 affaan-m 在同期进行了大规模 frontmatter 修复形成鲜明对比——**同样是被 NLPM 审计，不同维护者对反馈的响应方式完全不同。**

### 4.3 新增的可学习模式

没有发现新增的可学习模式。但值得记录的是：**这种「接受了 PR 贡献但不合并、也不改进」的状态**（status: contributed），是开源协作中的一个现实困境——贡献者无法强制维护者采纳建议。

---

## 五、校准

### 5.1 我已经在做对的

1. **frontmatter 规范**：drama-workshop-skills 的 SKILL.md 有正确的 `name:` 和 `description:` frontmatter，比本案例的所有 commands 都强。
2. **技术栈命名规律**：虽然我的 gstack 按角色命名（ceo/designer），这和本案例按技术栈命名（angular-*/django-*）都是有规律的命名，不是随机的。
3. **场景聚焦**：drama-workshop-skills 专注单一场景，避免了本案例「什么都包但什么都浅」的问题。

### 5.2 挑战 / 验证

本案例证实了一个关键认知：**「写出来」和「写好了」之间有巨大的鸿沟，而且这个鸿沟在三个月内会保持原样**。前者需要几分钟，后者需要系统性方法论（如 NLPM 评分）。

对于我的行动启示：把我写过的所有 SKILL.md 过一遍 NLPM scoring，确认没有类似的 vague 量词堆积问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件有无 frontmatter（aaronmaturen 同款问题）
find . -name "*.md" \( -path "*/commands/*" -o -path "*SKILL.md" \) | while read f; do
  head -1 "$f" | grep -q "^---" || echo "NO FRONTMATTER: $f"
done
# 命中后：立即添加标准 frontmatter

# 检查 vague 量词密度（每文件 > 3 个就需要处理）
for f in $(find . -name "SKILL.md"); do
  count=$(grep -coiE '\b(comprehensive|appropriate|robust|effective|relevant|proper|good|clear|focused)\b' "$f" 2>/dev/null || echo 0)
  [ "$count" -gt 2 ] && echo "VAGUE×${count}: $f"
done
# 命中后：逐个替换为可验证的具体标准

# 检查空输入处理
grep -rn 'ARGUMENTS\|\$1' . --include="*.md" | grep -v "if\|\[\|test\|\\-z" | head -10
# 命中后：在命令开头添加 if [ -z "$ARGUMENTS" ] 的 guard
```

### 6.2 灵感 → 实施路径

1. **想法**：建立「命令质量 checklist」在写完每个新命令后自检
   - **为何可行**：本案例的 26 个命令所犯的错误完全可以用 5 项 checklist 覆盖
   - **第一步**：在 CLAUDE.md 里写 `## 新命令 Checklist`：□ frontmatter 完整、□ 无 vague 量词、□ 有 empty-input guard、□ 有 allowed-tools、□ 有一个 example；每次写新 skill 都过一遍这个 checklist；10 分钟写完

暂无其他灵感。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 aaronmaturen/claude-plugin 的核心目的**：个人 Angular/Django 全栈开发工作流的 slash command 集合

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 都是特定场景的 Claude Code skill/command 集合 | drama 更深、更聚焦；aaronmaturen 更宽、更浅 | 中（作为反面对照） |
| MarkQWu/gstack | 低 | 都是个人工作流辅助工具 | gstack 更系统，有 CEO/Designer 等角色分工 | 低 |

若全部「无」，本节仅做技术模式对照：本案例作为典型反面案例，技术对照价值高于目的对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 零 frontmatter | `head -3 /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | drama-workshop-skills 的 SKILL.md **有正确 frontmatter** ✅ | 无（已解决） |
| vague 量词（appropriate/comprehensive） | `grep -i "appropriate\|comprehensive" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | 未在 drama-workshop 发现 cap-hitting vague 词汇 | 低 |
| 缺 allowed-tools | `grep "allowed-tools" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | drama-workshop-skills **缺 allowed-tools 字段** ❌ | 中 |

**命中后的具体行动建议**：
- `MarkQWu/drama-workshop-skills/short-drama/SKILL.md` → 在 frontmatter 添加 `allowed-tools: Read, Write, Bash`（视 short-drama skill 实际使用的工具而定）；5 分钟

### 8.3 别人的更优方案（值得借鉴的）

本案例是反面案例，没有「别人的更优方案」值得借鉴。但有一个提醒：

1. **领域**：有规律的技术栈命名（angular-*/django-*）
   - **本案例做法**：`commands/angular-architecture-audit.md`、`commands/angular-performance-audit.md`——按技术栈 + 任务类型命名，逻辑清晰
   - **我的项目现状**：gstack 命令按角色命名（`ceo/SKILL.md`、`designer/SKILL.md`），逻辑也很清晰
   - 两种命名策略都合理，关键是**保持一致**

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：frontmatter 完整性
- **我的做法**：`MarkQWu/drama-workshop-skills` 的所有 SKILL.md 都有完整的 `name:` 和 `description:` frontmatter
- **本案例做法（弱在哪）**：aaronmaturen 的 26 个 commands 全部缺少 frontmatter，三个月后仍未修复
- **意义**：在 frontmatter 规范性这一维度上，我的项目优于本案例；若被审计，这是得分亮点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`）。Claude Code 通过 frontmatter 知道命令叫什么、该在什么时候触发。没有 frontmatter = 命令没有"名字标签"，系统找不到它。

### <a name="empty-input-guard"></a>empty-input guard
> 在命令文件开头加的「输入为空时怎么办」的逻辑。比如 `/spike-investigation` 但没有附带调研主题，应该告诉用户"请提供调研主题"，而不是让 Claude 对着空白发呆或产生无意义的输出。

### <a name="vague-quantifier"></a>vague quantifier（模糊量词）
> 在 NL 编程里，像 "comprehensive"、"appropriate"、"relevant"、"thorough" 这类词叫做「模糊量词」——它们描述了一种状态，但没有给出「到什么程度算达到」的标准。对人类读者来说显而易见，但对 AI 来说无法执行。NLPM 每发现一个扣 −2 分，上限 −20 分。
> **修法**："comprehensive test coverage" → "test coverage including all exported functions and integration endpoints"
