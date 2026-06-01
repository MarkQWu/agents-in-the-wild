# obra/superpowers — 学习案例

| 字段 | 值 |
|------|-----|
| 仓库 | [obra/superpowers](https://github.com/obra/superpowers) |
| Stars | 166,779 |
| 来源 | xiaolai upstream |
| 审查日期 | 2026-04-06 |
| 生成日期 | 2026-06-01 |
| 主题标签 | `single-purpose` · `cross-reference` · `manifest-discipline` · `vague-quantifier` · `examples-driven` |

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`obra/superpowers` 是一个面向 Claude Code 的纯技能库（[skill](#skill) library），当前版本 v5.1.0（发布于 2026-04-30），拥有 166,779 颗 Star，是迄今规模最大的 Claude Code 插件之一。作者 Jesse Vincent（jesse@fsck.com）将其定位为"为 AI 代理提供超能力的技能集合"。

v5.1.0 是一次重要的架构转折点：整个 `commands/` 目录被彻底移除（PR #1188），三个旧命令（`brainstorm.md`、`write-plan.md`、`execute-plan.md`）连同它们所属的目录一并删除。`agents/code-reviewer.md` 也在同一时期被移除（PR #1299），其内容以 Task 派遣模板的形式合并进了 `skills/requesting-code-review/code-reviewer.md`。从 v5.1.0 起，该仓库是一个**无命令、无独立代理**的纯技能库。

当前核心内容：14 个技能（SKILL.md）文件，每个技能包含正文描述和若干[伴随文件](#companion-file)。配套有 `hooks/` 目录（跨平台版本：`hooks.json`、`hooks-cursor.json`、`hooks/session-start`、`hooks/run-hook.cmd`），以及 `tests/` 目录中的多套测试套件。此外，该仓库支持多平台部署：`.claude-plugin/`、`.cursor-plugin/`、`.codex-plugin/`、`.opencode/`，以及为 Gemini 扩展新增的 `GEMINI.md`。

### 1.2 架构剖析

v5.1.0 之后的目录结构如下：

```
obra/superpowers/
├── skills/
│   ├── brainstorming/
│   │   ├── SKILL.md
│   │   └── visual-companion.md
│   ├── systematic-debugging/
│   │   ├── SKILL.md
│   │   └── root-cause-tracing.md
│   ├── requesting-code-review/
│   │   ├── SKILL.md
│   │   └── code-reviewer.md      ← 原 agents/code-reviewer.md 迁入
│   ├── test-driven-development/SKILL.md
│   ├── dispatching-parallel-agents/SKILL.md
│   ├── subagent-driven-development/SKILL.md
│   ├── using-git-worktrees/SKILL.md
│   ├── executing-plans/SKILL.md
│   ├── writing-plans/SKILL.md
│   ├── writing-skills/SKILL.md
│   ├── using-superpowers/SKILL.md
│   ├── verification-before-completion/SKILL.md
│   ├── finishing-a-development-branch/SKILL.md
│   └── receiving-code-review/SKILL.md
├── hooks/
│   ├── hooks.json
│   ├── hooks-cursor.json
│   ├── session-start
│   └── run-hook.cmd
├── tests/
│   ├── brainstorm-server/
│   ├── claude-code/
│   ├── subagent-driven-dev/
│   └── ...
├── .claude-plugin/
├── .cursor-plugin/
├── .codex-plugin/
├── .opencode/
├── CLAUDE.md
├── AGENTS.md
├── GEMINI.md
└── plugin.json
```

架构的核心设计原则是**单一职责**（single-purpose）：每个技能只做一件事，通过技能间相互引用（cross-reference）形成有机网络，而不是在单个庞大文件中堆积所有内容。伴随文件（如 `visual-companion.md`、`root-cause-tracing.md`）承载了技能的辅助模板或子协议，保持主 `SKILL.md` 的简洁。

### 1.3 设计思路 / 方法论

`obra/superpowers` 的方法论建立在三个支柱之上：

1. **技能即单元**：每个技能是一个独立可组合的单元，不绑定到任何命令或代理。AI 代理可按需加载任意组合的技能，而不是被迫接受一个大型配置文件。这使得技能库具备良好的正交性——添加或删除某个技能不影响其他技能的功能。

2. **交叉引用驱动的知识网络**：技能内部会显式引用其他技能（如 `writing-plans` 引用 `executing-plans`，`test-driven-development` 引用 `verification-before-completion`），形成一个有向图。这种设计让 Claude 在处理复杂任务时能够自行导航到相关技能，而不必等待用户显式加载。

3. **反 slop 设计（anti-slop guidelines）**：v5.1.0 在 `CLAUDE.md` / `AGENTS.md` 中引入了针对 AI 代理的"反废话指南"，源于对 94% AI 生成 PR 被拒绝率的分析。这是一套元层面的约束：告诉 Claude Code 代理哪些行为模式被该仓库拒绝（如重复废话、过度格式化、无实质内容的提交），是目前为止极少数将 AI 行为偏差作为第一类问题显式处理的插件仓库。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名称 | 描述 |
|---------|------|
| **纯技能库模式** | 零命令、零独立代理，只有 SKILL.md 文件作为可组合单元，AI 按需加载 |
| **伴随文件模式** | 每个高复杂度技能配备 1-N 个伴随文件，承载子协议、视觉模板或 Task 派遣提示 |
| **技能内聚代理模式** | 将原本独立的代理定义（code-reviewer）迁入技能的伴随文件，以 Task 模板形式保留 |
| **跨技能交叉引用模式** | 技能内显式引用其他相关技能，形成可导航的知识图谱，减少冗余 |
| **多平台清单并行模式** | 同一套技能通过不同平台目录（`.claude-plugin/`、`.cursor-plugin/`、`.codex-plugin/`）支持多个 AI 客户端 |
| **反 slop 元约束模式** | 在 CLAUDE.md / AGENTS.md 中显式声明 AI 代理行为禁忌，从源头约束生成质量 |

### 2.2 适用场景

- **纯技能库模式**：适用于目标明确的领域知识沉淀，如调试方法论、代码审查规程等。当技能内容稳定、使用者不需要特定调用语法时，去掉命令层反而降低了复杂度。
- **伴随文件模式**：适用于技能内容超出单个 SKILL.md 合理容量时，或者需要提供可直接粘贴执行的模板（如代码审查者提示词）时。
- **技能内聚代理模式**：适用于代理功能与某个技能高度耦合、不存在独立调用需求的场景。相比独立 agents/ 目录，减少了原件数量，降低了维护成本。
- **反 slop 元约束模式**：适用于接受外部 AI 贡献的仓库，或希望明确告知 Claude Code 代理行为预期的个人项目。94% 拒绝率的数据表明，没有约束的 AI 代理往往产出不符合人工维护者期待的内容。

### 2.3 与其他架构对比

| 维度 | obra/superpowers（纯技能库） | 带命令层的技能+命令混合架构 | 代理+技能分层架构 |
|------|-----------------------------|-----------------------------|------------------|
| 可组合性 | **高**：技能独立，任意组合 | 中：命令与技能耦合 | 中：代理绑定技能 |
| 学习曲线 | 低：只需了解技能加载方式 | 中：需理解命令调用语法 | 高：需理解代理派遣机制 |
| 维护复杂度 | **低**：无命令/代理同步问题 | 中：命令与技能需保持一致 | 高：代理引用、命令引用、技能引用三层需同步 |
| 运行时灵活性 | **高**：任意 AI 客户端按需加载 | 低：依赖命令调用约定 | 中：依赖代理派遣约定 |
| 知识完整性 | 依赖交叉引用，有导航成本 | 命令可封装完整流程 | 代理可封装完整状态机 |

### 2.4 改进空间

1. **`hooks.json` 中 SessionStart `matcher` 字段非标准**：`matcher: "startup|clear|compact"` 的正则语法在 Claude Code 中对 SessionStart 类型的钩子无语义意义（SessionStart 不通过 matcher 过滤），属于误用或冗余配置。虽为 info 级别，但会对阅读者造成误导。
2. **跨平台清单一致性检查缺失**：四个平台目录（`.claude-plugin/`、`.cursor-plugin/`、`.codex-plugin/`、`.opencode/`）的[清单](#manifest)文件是否保持同步，目前没有自动化验证机制。若有技能新增/删除，清单漂移风险较高。
3. **测试套件与技能的映射关系不透明**：`tests/` 目录有多个测试套件，但哪个套件测试哪个技能没有明确文档，新贡献者难以定位相关测试。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：92/100**
**安全等级：CLEAR**（无安全问题，PR 贡献通道畅通）

按扣分项分解：

| 类别 | 扣分 | 描述 |
|------|------|------|
| Bug：`commands/brainstorm.md` 缺少 `name` 字段 | -25 | [frontmatter](#frontmatter) 中无 `name` 字段，命令无法被 Claude Code 识别 |
| Bug：`commands/write-plan.md` 缺少 `name` 字段 | （已含在上项合并惩罚内） | 同类问题 |
| Bug：`commands/execute-plan.md` 缺少 `name` 字段 | （已含在上项合并惩罚内） | 同类问题 |
| 质量：`agents/code-reviewer.md` 缺少输出格式章节 | -10 | 无 `## Output Format` 或等效章节 |
| 质量：[模糊量词](#vague-quantifier) `"proper"×2 / "appropriate"×1` | -6 | 位于 `agents/code-reviewer.md`，提示词意图模糊 |
| 质量：`skills/brainstorming/SKILL.md` 中 `"appropriately-scoped"` | -2 | 模糊量词，范围界定不清 |
| 信息：`hooks/hooks.json` SessionStart hook 含非标 `matcher` | 0 | info 级别，不扣分，但标记为潜在问题 |

14 个技能全部得到 **100/100**，覆盖：`using-git-worktrees`、`receiving-code-review`、`executing-plans`、`subagent-driven-development`、`test-driven-development`、`finishing-a-development-branch`、`dispatching-parallel-agents`、`using-superpowers`、`systematic-debugging`、`writing-skills`、`verification-before-completion`、`requesting-code-review`、`writing-plans`，以及另外 1 个技能。

### 3.2 当时值得借鉴的模式

1. **14 个技能全部满分（100/100）**：这是同类插件中极为罕见的成绩。每个技能都做到了目标明确、无歧义、有具体操作步骤、无模糊量词。这不是偶然——技能的"单一职责"设计天然减少了模糊表达的空间。
2. **技能交叉引用网络成熟**：技能之间的相互引用在 2026-04-06 时已经形成清晰网络，不是孤立的知识碎片。
3. **伴随文件已成常态**：`brainstorming/visual-companion.md`、`systematic-debugging/root-cause-tracing.md` 等伴随文件在当时已存在，提供了可直接使用的辅助模板。
4. **安全 CLEAR**：无任何 Critical 或 High 级别安全发现，钩子引用的脚本均真实存在，无"空槽安全风险"。

### 3.3 当时的缺陷

**Bug（三处同类）：**

- `commands/brainstorm.md`、`commands/write-plan.md`、`commands/execute-plan.md` 的 frontmatter 缺少 `name:` 字段。这意味着这三个命令在 Claude Code 中无法被正确识别和调用，实际上是失效命令。

**质量问题：**

- `agents/code-reviewer.md` 缺少 `## Output Format` 章节：Claude 不知道以何种格式输出审查结果，使用者无法建立稳定的输出预期。
- `agents/code-reviewer.md` 中三处[模糊量词](#vague-quantifier)（`proper` ×2、`appropriate` ×1）：在代码审查场景中，"恰当的注释"比"有注释"更难让 Claude 产出可预期的判断。
- `skills/brainstorming/SKILL.md` 中 `appropriately-scoped`：范围界定完全依赖 Claude 的运行时判断，无客观标准。

### 3.4 当时的优化机会

1. 为三个命令文件补充 `name:` 字段（一行修复，消除 -25 惩罚）
2. 为 `agents/code-reviewer.md` 添加 `## Output Format` 章节（消除 -10 惩罚）
3. 将模糊量词替换为可操作的具体标准（如将"proper comments"替换为"每个公开函数/方法有一句描述其行为的注释"）
4. 评估 `hooks/hooks.json` 中 SessionStart hook 的 `matcher` 字段是否需要保留

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-06 快照 | 2026-06-01 当前 HEAD | 状态 |
|------|----------------|---------------------|------|
| `brainstorm.md` 缺少 `name` 字段 | 存在 | 文件已随整个 `commands/` 目录删除（v5.1.0 PR #1188） | 已消除（文件不复存在） |
| `write-plan.md` 缺少 `name` 字段 | 存在 | 同上 | 已消除（文件不复存在） |
| `execute-plan.md` 缺少 `name` 字段 | 存在 | 同上 | 已消除（文件不复存在） |
| `agents/code-reviewer.md` 缺少输出格式章节 | 存在 | 文件已从 `agents/` 目录删除（v5.1.0 PR #1299），内容迁入 `skills/requesting-code-review/code-reviewer.md` | 已消除（原件形态变更） |
| `agents/code-reviewer.md` 模糊量词（3 处） | 存在 | 原文件已删除，迁移后的伴随文件状态待验证 | 需验证（形态变更） |
| `skills/brainstorming/SKILL.md` 中 `appropriately-scoped` | 存在 | 技能文件仍存在，词条状态未确认 | 可能持续存在 |
| `hooks/hooks.json` 非标 `matcher` | 存在 | **仍然存在**，SessionStart hook 仍有 `matcher: "startup\|clear\|compact"` | 持续存在（未修复） |

**结论**：三个 bug 通过"移除问题文件"而非"修复问题"的方式消除，这反映了 v5.1.0 的架构方向——命令层被认为是不必要的复杂性，直接砍掉。唯一在代码层面持续存在的问题是 `hooks.json` 中的非标 matcher。

### 4.2 架构演进

v5.1.0 的变化轨迹揭示了一个明确的方向性选择：

- **删除命令层**：`commands/` 目录整体移除（PR #1188）。作者认为命令的价值不及其带来的维护复杂度，纯技能库更灵活。
- **代理内聚到技能**：`agents/code-reviewer.md` 迁入 `skills/requesting-code-review/code-reviewer.md` 作为伴随文件（PR #1299）。代理不再是独立层级，而是技能的一个部分。
- **多平台扩展**：新增 `GEMINI.md`，支持 Gemini 扩展；`.codex-plugin/`、`.cursor-plugin/` 目录体现了对多 AI 客户端的主动适配。
- **反 slop 内容治理**：CLAUDE.md / AGENTS.md 新增基于 94% PR 拒绝率分析的 AI 行为约束，是从"给 AI 看的指南"到"约束 AI 本身行为"的元层面跃升。

这次演进的核心逻辑是**降低层级，提升纯粹性**：从"技能 + 命令 + 代理"的三层结构收敛到"技能 + 伴随文件"的扁平结构。

### 4.3 新增的可学习模式

- **技能内聚代理（Skill-inlined Agent）**：代理不再作为独立文件存在，而是作为 Task 派遣模板嵌入相关技能的伴随文件。这消除了"代理 frontmatter 缺字段"类型的 bug，同时保留了代理的功能。
- **反 slop 元约束**：在 CLAUDE.md / AGENTS.md 中显式枚举 AI 代理的不良行为模式，并声明拒绝接受此类贡献。这是一种基于实际数据（94% 拒绝率）的自我保护机制。
- **跨平台清单同步**：一个仓库通过不同平台目录同时服务多个 AI 客户端（Claude、Cursor、Codex、OpenCode），展示了"一次编写，多处部署"的跨平台技能库设计路径。

---

## 五、校准

### 5.1 我已经在做对的

- **安全 CLEAR**：我的三个仓库均无 Critical / High 安全发现，钩子引用的脚本均真实存在，这与 superpowers 的 CLEAR 状态对齐。
- **技能单一职责**：`echo-sleuth-for-claude` 的技能（`memory-management`、`git-mining` 等）均为单一职责，与 superpowers 的技能库理念一致。
- **无多余命令层**：`drama-workshop-skills` 是纯技能库，与 superpowers v5.1.0 之后的架构方向相同。

### 5.2 挑战 / 验证

**反直觉发现：166,779 Star 的顶级仓库，92/100 的高分，其失分完全来自后来被删掉的文件。**

这个案例反驳了"高分意味着当前无问题"的简单推断。superpowers 的 92 分是在存在三个 Bug（三个命令缺 `name` 字段）的条件下取得的——但这些 Bug 恰恰证明了"问题存在于即将被移除的代码里"。当架构方向已经确定要删除某层时，是否值得先修复那层的 Bug，是一个合理的优先级判断，而非遗漏。

**校准动作**：在评估审查结果时，应同时查看问题文件的存活状态。一个被 -25 扣分的 Bug，如果所在的文件已进入弃用路径，其实际风险远低于分数显示的严重程度。NLPM 的评分是快照静态评分，需要结合仓库演进方向来解读。

---

## 六、行动

### 6.1 自查动作

对自己的仓库执行以下检查，确认无类似问题：

```bash
# 检查所有技能 SKILL.md 是否有输出格式章节
for repo in echo-sleuth-for-claude drama-workshop-skills claude-for-legal; do
  echo "=== $repo : skills missing Output Format ==="
  find ~/$repo -name "SKILL.md" | while read f; do
    grep -q "## Output Format\|## 输出格式" "$f" || echo "MISSING: $f"
  done
done

# 检查技能文件中的模糊量词
for repo in echo-sheuth-for-claude drama-workshop-skills claude-for-legal; do
  echo "=== $repo : vague quantifiers ==="
  grep -rn "\bproper\b\|\bappropriate\b\|\brelevant\b\|\bsuitable\b" ~/$repo/skills/ 2>/dev/null
done

# 检查 frontmatter 中是否缺少 name 字段（命令和代理文件）
for repo in echo-sleuth-for-claude drama-workshop-skills claude-for-legal; do
  echo "=== $repo : commands/agents missing name field ==="
  find ~/$repo -name "*.md" \( -path "*/commands/*" -o -path "*/agents/*" \) | while read f; do
    head -5 "$f" | grep -q "^name:" || echo "MISSING name: $f"
  done
done
```

### 6.2 灵感 → 实施路径

**灵感 1：为技能添加伴随文件**

superpowers 的 `brainstorming/visual-companion.md` 和 `systematic-debugging/root-cause-tracing.md` 展示了伴随文件的价值——将"如何做"的步骤性内容（技能正文）与"可直接使用的模板"（伴随文件）分开。

实施路径：
1. 审查 `echo-sleuth-for-claude` 的四个技能，找出有子协议需求的技能（如 `git-mining`）
2. 将"输出模板"或"子流程提示词"提取为 `git-mining/output-template.md`
3. 在主 `SKILL.md` 中通过 `See also:` 或 `## 伴随文件` 段落引用

**灵感 2：反 slop 元约束**

基于 superpowers CLAUDE.md 新增的 AI 行为约束，在自己的仓库中添加类似约束，明确告知 AI 代理哪些输出模式不被接受（如过度分点、重复上下文内容、无实质改变的空提交）。

实施路径：
1. 在 `echo-sleuth-for-claude` 的 CLAUDE.md 中新增 `## AI 代理行为约束` 章节
2. 基于自身使用经验列出 2-3 条最常见的不良输出模式
3. 在相关技能的 `## 注意事项` 中交叉引用该约束

**灵感 3：技能交叉引用**

superpowers 的技能之间形成可导航的知识图谱。我的技能库目前是孤立的。

实施路径：
1. 梳理 `echo-sleuth-for-claude` 四个技能的依赖关系（如 `experience-synthesis` 依赖 `git-mining` 的输出）
2. 在每个技能的 `## 相关技能` 或 `## 参见` 章节中显式引用依赖的技能
3. 确保引用是双向的（如果 A 引用 B，B 也应在适当位置提及 A 的使用场景）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | superpowers 核心目的 | 对齐度 |
|---------|---------------------|-------|
| MarkQWu/echo-sleuth-for-claude | 为 Claude Code 代理提供可组合的方法论技能 | **中**：同样是技能库 + 代理架构，但 echo-sleuth 聚焦对话历史挖掘这一垂直场景，而非通用编程技能 |
| MarkQWu/drama-workshop-skills | 同上 | **低**：短剧制作领域与编程技能完全不同，但作为纯技能库的形态是对齐的 |
| MarkQWu/claude-for-legal | 同上 | **低**：法律工作流与编程技能场景不同，但多技能并行组织的思路对齐 |

### 8.2 在我的项目里复现的同类问题

**问题 1：多数技能缺少 `## Output Format` 章节**

与审查发现的 `agents/code-reviewer.md` 缺少输出格式章节类似，`echo-sleuth-for-claude` 的技能存在相同问题：
- `skills/jsonl-core/SKILL.md`：缺少 `## Output Format`
- `skills/memory-management/SKILL.md`：缺少 `## Output Format`
- `skills/git-mining/SKILL.md`：缺少 `## Output Format`

`drama-workshop-skills` 的 `skills/short-drama/SKILL.md` 也缺少此章节。这是 3+1=4 个技能文件的系统性遗漏，比 superpowers 当时单个代理的遗漏更严重。

**问题 2：模糊量词残留**

- `echo-sleuth-for-claude` 的 `skills/experience-synthesis/SKILL.md` 第 118 行含 `relevant`
- `echo-sleuth-for-claude` 的 `skills/git-mining/SKILL.md` 第 93 行含 `relevant`

与 superpowers 的 `appropriate` / `proper` 问题性质相同：`relevant` 在指令性文本中无法给 Claude 提供客观判断依据。修复方式：将"相关内容"替换为具体的筛选标准（如"日期在 30 天以内且包含用户提名关键词的"）。

### 8.3 别人的更优方案

**伴随文件模式（superpowers 优于我）**：

superpowers 的每个高复杂度技能都有配套的伴随文件（`visual-companion.md`、`root-cause-tracing.md`、`code-reviewer.md`）。这些文件将"技能的使用模板"与"技能的方法论描述"分开，让主 SKILL.md 保持简洁聚焦，同时为需要的用户提供即开即用的模板。

我的所有技能均为独立单文件，没有伴随文件。对于 `echo-sleuth-for-claude` 的 `git-mining` 技能，提供一个 `git-mining/query-template.md` 会让使用者更容易上手。

**技能间交叉引用（superpowers 优于我）**：

superpowers 的技能形成有向引用网络，Claude 在执行复杂任务时可以沿着引用链找到相关技能。我的技能完全孤立，没有任何技能间引用。对于 `echo-sleuth-for-claude`，`experience-synthesis` 应该显式引用 `git-mining`（它的数据来源）和 `memory-management`（它的输出存储目标）。

**测试套件（superpowers 优于我）**：

superpowers 的 `tests/` 目录有多套独立测试套件，覆盖不同技能的使用场景。我的三个仓库均无测试目录，无法验证技能是否按预期工作。

### 8.4 反向：我的项目做得比他们好的地方

1. **技能 + 代理分层架构保留（echo-sleuth-for-claude 优于 superpowers v5.1.0 后）**：superpowers 在 v5.1.0 删除了独立代理层，将代理降格为技能伴随文件中的 Task 模板。`echo-sleuth-for-claude` 保留了独立的 `agents/` 目录，技能层（what/why）与代理层（how/execute）明确分离。对于 echo-sleuth 这类需要状态化多步执行的场景（挖掘→提炼→存储），代理独立存在是正确的设计选择——不是所有仓库都适合 superpowers 的"去代理化"路径。

2. **无 hooks.json 非标配置**：我的仓库在钩子配置上比 superpowers 更规范——superpowers 在 SessionStart hook 上保留了无语义意义的 `matcher` 字段（持续至今），而我的仓库不存在此类非标配置。

---

## 八、术语表

| 术语 | 说明 |
|------|------|
| <a name="frontmatter">frontmatter</a> | Markdown 文件开头由 `---` 包围的 YAML 元数据块，Claude Code 用它解析命令/代理的 `name`、`description`、`model`、`allowed-tools` 等属性。缺少 `name:` 字段会导致命令/代理无法被识别。 |
| <a name="skill">skill（技能）</a> | 以 `SKILL.md` 文件形式存在的一段 Claude 行为规程，描述 Claude 在特定场景下应遵循的方法论。技能不直接执行，而是被命令或代理通过 `skills:` frontmatter 字段加载后作为上下文知识使用。 |
| <a name="agent">agent（代理）</a> | 以 `.md` 文件形式存在的 Claude 执行单元，包含 `name`、`model`、`description`、`tools`、`skills` 等 frontmatter 字段，定义了一个具备特定角色和能力边界的 Claude 实例。可被命令通过 Task 工具派遣。 |
| <a name="hook">hook（钩子）</a> | 在 `hooks.json` 中声明的事件响应配置，指定在特定 Claude Code 生命周期事件（如 `SessionStart`、`PostToolUse`、`Stop`）发生时执行特定命令。钩子引用的脚本必须真实存在，否则构成"空槽安全风险"。 |
| <a name="manifest">manifest（清单）</a> | 插件的 `plugin.json` 文件，声明插件 ID、版本、名称、包含的命令/代理/技能列表。Claude Code 通过清单发现插件内容。多平台场景下，各平台目录（如 `.cursor-plugin/`）可能有各自的清单文件，需保持同步。 |
| <a name="vague-quantifier">vague quantifier（模糊量词）</a> | 在指令性文本中使用的、无客观判断标准的形容词或副词，如 `proper`（恰当的）、`appropriate`（适当的）、`relevant`（相关的）、`appropriately-scoped`（适当范围的）。这类词语要求 Claude 自行界定标准，导致输出不稳定。修复方式：替换为具体可量化的标准。 |
| <a name="companion-file">companion file（伴随文件）</a> | 与 SKILL.md 放置在同一技能目录下的辅助文件，用于承载可直接使用的模板、子协议或 Task 派遣提示词。例如 `brainstorming/visual-companion.md`（视觉化提示模板）、`requesting-code-review/code-reviewer.md`（代码审查者角色 Task 模板）。伴随文件使主 SKILL.md 保持方法论聚焦，同时提供即用型工具。 |
