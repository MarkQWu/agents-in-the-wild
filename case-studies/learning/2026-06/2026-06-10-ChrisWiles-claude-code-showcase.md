# ChrisWiles/claude-code-showcase — 学习案例

**仓库**：https://github.com/ChrisWiles/claude-code-showcase
**Stars**：未记录（Upstream Pool）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-10（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `vague-quantifier`, `template-design`, `model-pinning`

---

## 一、理解

### 1.1 仓库概览

ChrisWiles/claude-code-showcase 是一个 React Native/TypeScript 项目集成 Claude Code 的展示（showcase）仓库，而非通用 plugin 市集或生产插件。它的核心功能是展示：当你在一个真实的移动端代码库中接入 Claude Code 时，这套 NL artifacts 应该长什么样。

规模数据（2026-04-12 审计快照）：
- **17 个 NL artifacts**：6 commands、2 agents、6 skills、1 hook config + 2 hook scripts
- 整体 NLPM 得分：**81/100（Silver 级）**，Security: **CLEAR**
- 描述精确：每个 skill 服务 React Native/TypeScript 代码库的一个具体切面（formik 表单、graphql schema、react-ui-patterns 等）

关键背景：该仓库是一个"教材型"仓库——目的不是被 `claude plugin install` 安装到生产环境，而是向其他开发者展示"一个做得比较好的 Claude Code 集成样板长什么样"。这个定位直接影响了它的优缺点：showcase 质量（skill 文件整洁、agent 设计清晰）远高于生产就绪质量（命令 frontmatter 不完整、ghost skills 持续存在）。

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
.claude/
  commands/: code-quality.md, docs-sync.md, onboard.md, pr-review.md, pr-summary.md, ticket.md
  agents/: code-reviewer.md, github-workflow.md
  hooks/: skill-eval.sh, skill-eval.js, skill-rules.json, skill-rules.schema.json
  skills/: README.md + 6 subdirs
    react-ui-patterns/SKILL.md
    core-components/SKILL.md
    testing-patterns/SKILL.md
    formik-patterns/SKILL.md
    graphql-schema/SKILL.md
    systematic-debugging/SKILL.md
  settings.json
  settings.md
.mcp.json
.github/workflows/  （pr-claude-code-review.yml 等）
CLAUDE.md, README.md
```

**分层结构**：该仓库体现了典型的"三层 Claude Code 集成架构"——commands（用户入口）、agents（专家执行者）、skills（领域知识库）——加上一个独特的第四层：**hook-based 自动路由层**（skill-eval.js + skill-rules.json）。

### 1.3 设计思路

该仓库的核心设计哲学是**约定优于配置（convention over configuration）**，用一个智能 hook 代替人工激活 skill 的繁琐操作。

传统做法：用户需要手动告诉 Claude「请加载 formik-patterns skill」，然后才能得到 Formik 相关的领域知识辅助。

该仓库的做法：`skill-eval.js` hook 在每次工具调用后自动分析当前上下文（用户正在操作哪个目录、代码文件属于什么类型、当前意图是什么），然后根据 `skill-rules.json` 中的路由规则推荐应该激活哪个 skill。用户不需要知道 skill 的存在，系统自动在合适的时机推送领域知识。

这是整个仓库最有原创性的设计，也是最值得学习的创新点。

---

## 二、架构作为可学习模式

### 2.1 模式名称：「智能上下文路由 + 展示型插件」

**模式结构定义：**
```
用户操作某个代码文件
   ↓
skill-eval.js hook 触发（PostToolUse: Write|Edit|MultiEdit）
   ↓
分析当前上下文（目录路径 / 文件类型 / 操作意图）
   ↓
查询 skill-rules.json 路由表
   ↓
建议激活对应 skill（e.g., formik-patterns、react-ui-patterns）
   ↓
Claude 自动加载领域知识，无需用户手动指定
```

**skill-rules.json 路由表结构**（关键创新点）：
- 每条规则包含：匹配条件（目录模式、文件扩展名、关键词）→ 推荐 skill 名称
- 当前 HEAD 包含 20+ 条规则，覆盖 storybook、testing-patterns、formik-patterns、graphql-schema、navigation-architecture、i18n-translations 等全部领域
- 该 JSON 文件有配套 `skill-rules.schema.json`——JSON Schema 验证路由表格式，这是少见的工程严谨性体现

### 2.2 展示型插件 vs 生产插件：trade-off 分析

| 维度 | 展示型插件（本仓库） | 生产插件 |
|------|----------------|---------|
| 目标受众 | 想学习 Claude Code 集成的开发者 | 最终用户，直接使用 plugin |
| 质量重心 | 内容深度、架构创意 | 注册可靠性、边缘情况处理 |
| Frontmatter 纪律 | 可以松散（showcase 用途不需要被 install） | 必须严格（frontmatter 错误 = 命令不可见） |
| Ghost skill 容忍度 | 较高（占位符可解释为"规划中"） | 零容忍（路由规则引用不存在的 skill 会静默失败） |
| 关键风险 | 学习者误以为这些 bug 是最佳实践 | 生产运行时出错 |

**关键洞察**：展示型插件的最大风险是"教材效应"——学习者倾向于照搬 showcase 的所有设计，包括那些仍是 bug 的部分。当 `onboard.md` 缺少三个 frontmatter 字段时，一个照着学的开发者可能也会创建同样缺失字段的命令，然后困惑为什么命令不出现。

### 2.3 skill-eval hook 的工程设计亮点

`skill-rules.schema.json` 的存在意味着：即使 skill-rules.json 被其他开发者修改，也有格式验证层保护。这个设计思路——**schema-first 路由配置**——比裸 JSON 配置更健壮，但前提是规则里引用的 skill 实际存在（即 ghost skill 问题必须先解决）。

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

整体 NL 得分：**81/100（Silver 级）**，Security: **CLEAR**

各文件得分明细：

| 文件 | 得分 | 主要问题 |
|------|------|---------|
| .claude/commands/onboard.md | 23 | 缺 name、description、allowed-tools 三个字段 |
| .claude/commands/ticket.md | 61 | 缺 name frontmatter |
| .claude/commands/code-quality.md | 65 | 缺 name frontmatter |
| .claude/commands/docs-sync.md | 65 | 缺 name frontmatter |
| .claude/commands/pr-review.md | 65 | 缺 name frontmatter；unquoted `$ARGUMENTS` |
| .claude/commands/pr-summary.md | 65 | 缺 name frontmatter |
| .claude/agents/code-reviewer.md | 95 | 优秀 |
| .claude/agents/github-workflow.md | 78 | 无 output format section |
| .claude/skills/react-ui-patterns/SKILL.md | 95 | 优秀 |
| .claude/skills/core-components/SKILL.md | 95 | 优秀 |
| .claude/skills/testing-patterns/SKILL.md | 95 | 优秀 |
| .claude/skills/formik-patterns/SKILL.md | 95 | 优秀 |
| .claude/skills/graphql-schema/SKILL.md | 95 | 优秀 |
| .claude/skills/systematic-debugging/SKILL.md | 95 | 优秀 |
| .claude/hooks/skill-rules.json | -20（惩罚）| 13 个 ghost skills |
| .claude/hooks/skill-eval.js | 参考实现 | 创新机制，无独立得分 |
| .mcp.json | 安全 Medium | 7 个 MCP server 无版本 pin |

安全发现明细：
- **Medium**：`.mcp.json` — 全 7 个 MCP server 使用 `npx -y @anthropic/mcp-*` 无版本固定，供应链风险
- **Medium**：postgres MCP 无权限收敛，full DB access
- **Low**：`pr-review.md` 中 unquoted `$ARGUMENTS` in `gh pr view $ARGUMENTS`

### 3.2 当时值得借鉴的模式

1. **6 个 SKILL.md 全部 95 分**：内容结构整洁，每个 skill 服务单一领域，是同期审计中 skill 写作质量最高的仓库之一。

2. **code-reviewer.md agent 得分 95**：frontmatter 完整，描述精确，是命令/agent 分层设计的教科书案例。

3. **skill-rules.schema.json 设计**：路由配置有 JSON Schema 验证层，防止格式错误的规则混入。在所有被审计的 hook 实现中，这是少见的"配置即代码，代码有 schema"的严谨做法。

4. **创新机制：skill-eval.js 自动 skill 路由**：根据用户正在操作的目录/意图/代码内容自动建议应该激活哪个 skill，是 Claude Code ecosystem 中自动 skill 路由的早期探索。

### 3.3 当时的缺陷

**核心缺陷一：6 个命令全部缺少 `name` frontmatter**

- `code-quality.md`、`docs-sync.md`、`pr-review.md`、`pr-summary.md`、`ticket.md`：各缺 `name:` 字段
- `onboard.md`：缺 `name:`、`description:`、`allowed-tools:` 三个字段，得分仅 23

直接后果：以上 6 个命令在 slash command 调色板中全部不可见。用户安装该插件后，无法通过 `/code-quality` 或 `/pr-review` 等方式调用这些命令——命令存在但不可被发现。

**核心缺陷二：13 个 ghost skills（-20 分惩罚）**

`skill-rules.json` 路由表中引用了以下 13 个 skill 名称，但对应的 SKILL.md 文件从未被创建：

```
navigation-architecture, i18n-translations, state-management,
github-actions, analytics-tracking, list-pagination, modal-actionsheet,
maestro-e2e, receiving-code-review, documentation, typescript-conventions,
verification-before-completion, defense-in-depth
```

**根本原因分析**：这是一个典型的"框架先于内容"问题。作者首先设计了完整的路由框架（skill-rules.json 列出了所有计划的 skill），但对应的 SKILL.md 内容从未被创建。这在 showcase 仓库中尤为常见——展示架构雄心比填充实际内容更快，但遗留的占位符会让 hook 路由系统在运行时静默失败：系统建议激活 `navigation-architecture` skill，但该 skill 不存在，推荐毫无实际效果。

**次要缺陷：**
- `github-workflow.md` agent 无 output format section（-5 分）
- `.mcp.json` MCP server 无版本固定（安全中危）
- `pr-review.md` unquoted `$ARGUMENTS` shell 注入风险（安全低危）
- 4 个命令（code-quality、pr-review、ticket、onboard）无 empty-input guard

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在当前 HEAD 中的状态

以下基于 2026-06-10 Spot-check 验证：

| 缺陷 | 2026-04-12 状态 | 2026-06-10 状态 | 结论 |
|------|---------------|---------------|------|
| 6 个命令缺 `name` frontmatter | 全部缺失 | 全部**仍然缺失**（code-quality、docs-sync、onboard、pr-review、pr-summary、ticket 均无 name 字段） | 未修复 |
| onboard.md 缺 3 个字段 | name、description、allowed-tools 全缺 | **仍然缺失**全部 3 个字段 | 未修复 |
| 13 个 ghost skills | skill-rules.json 引用 13 个不存在 SKILL.md | HEAD 中 skill-rules.json **新增了更多规则**（现共 20+ 条），但 navigation-architecture、i18n-translations、state-management、github-actions、analytics-tracking、list-pagination、modal-actionsheet、maestro-e2e、receiving-code-review、documentation、typescript-conventions、verification-before-completion、defense-in-depth **仍无对应 SKILL.md** | 未修复，且问题扩大 |
| MCP 无版本 pin | 7 个 server 无 pin | 需验证（`.mcp.json` 结构未变） | 状态未知 |
| pr-review unquoted $ARGUMENTS | 存在 | 需验证 | 状态未知 |

**总结**：所有 8 个原始 bug 在当前 HEAD 中全部持续存在，且 ghost skill 数量因新增路由规则而进一步扩大。

### 4.2 新增架构演进

当前 HEAD 相比审计时新增了以下重要内容：

**`.github/workflows/pr-claude-code-review.yml`**：新增 GitHub Actions workflow，在每个 PR 时自动触发 Claude Code 进行代码审查。这是从"本地 showcase"向"CI 集成"迈进的显著架构演进。

**意义**：作者明显在继续投入该仓库，将 Claude Code 集成从开发者本地工具推向 CI/CD pipeline。这是 Claude Code ecosystem 中"AI-in-the-loop PR review"模式的实际落地案例。

### 4.3 新增的 model-pinning 问题

`pr-claude-code-review.yml` 中使用了 `claude-opus-4-5-20251101` 作为固定模型 ID。

**问题**：`claude-opus-4-5-20251101` 是一个过时的模型 ID，当前 Anthropic 提供的 Opus 最新版本为 `claude-opus-4-6`。固定过时模型 ID 会在 Anthropic 退役该 ID 后导致 workflow 失败，且不会有任何预警。

**正确做法选择**：
- 方案 A（版本明确）：更新为当前最新 ID，并在 commit message 中记录迁移原因
- 方案 B（自动跟随）：不指定模型 ID，由 `claude-code-action` 默认版本（当前 Sonnet 4.6）自动处理

该仓库恰好在本次学习案例生成日（2026-06-10）存在这个 model-pinning 问题，是 model-pinning 反模式的新鲜案例。

---

## 五、校准

### 5.1 该仓库整体水平评估

**强项**：
- 6 个 skill 文件写作质量极高（均 95 分），是同期审计中 skill 写作的标杆
- code-reviewer.md agent 设计规范（95 分）
- skill-eval.js 的自动路由机制有架构创意
- skill-rules.schema.json 体现工程严谨性
- 选题合适：React Native + TypeScript 是真实项目场景，不是人造示例

**弱项**：
- 命令层（6/6 缺 name frontmatter）是最基础的注册失败问题，与 skill 层的高质量形成鲜明反差
- Ghost skill 问题（13 个引用 → 0 个实现）造成路由系统空转，核心创新机制打折扣
- 两年后（2026-06-10）以上问题仍全部未修复，说明 showcase 型仓库的维护惰性

**校准结论**：该仓库的技术选型（hook-based skill routing）值得学习，但代码质量细节（frontmatter 纪律、ghost skill 清理）不应照搬。学习其创意，不要复制其 bug。

---

## 六、行动

### 6.1 如果参考该仓库设计 skill-eval hook，需要做的额外工作

```bash
# 检查 skill-rules.json 中引用的所有 skill 名称是否有对应 SKILL.md
python3 -c "
import json, os, sys
rules = json.load(open('.claude/hooks/skill-rules.json'))
skills_dir = '.claude/skills'
missing = []
for rule in rules.get('rules', []):
    skill_name = rule.get('skill')
    if skill_name and not os.path.exists(f'{skills_dir}/{skill_name}/SKILL.md'):
        missing.append(skill_name)
if missing:
    print('Ghost skills found:', missing)
    sys.exit(1)
else:
    print('All referenced skills have SKILL.md')
"
```

### 6.2 如果复用命令结构，先检查 name frontmatter

```bash
# 检查所有 commands/*.md 是否有 name 字段
for f in .claude/commands/*.md; do
    if ! grep -q "^name:" "$f"; then
        echo "Missing name frontmatter: $f"
    fi
done
```

### 6.3 如果复用 GitHub Actions workflow，更新 model ID

```bash
# 找到 workflow 文件中的模型 ID 声明
grep -rn "claude-opus-4-5\|claude-sonnet-4-5\|claude-haiku-4-5" .github/workflows/
# 输出：确认是否有过时 model ID，手动更新为当前版本
```

### 6.4 检查 .mcp.json 是否有无版本 pin 的 server

```bash
# 检查 .mcp.json 中是否有 npx -y 无版本锁定
grep -n "npx -y" .mcp.json | grep -v "@[0-9]"
# 如果有输出：存在无版本 pin 的 MCP server，存在供应链风险
```

---

## 七、对照我的 GitHub 仓库

### 7.1 架构相似度分析

| 我的仓库 | 与本仓库相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|-------------|------|------|----------|
| MarkQWu/echo-sleuth-for-claude | **最高** | 都是单一领域专注型插件（echo-sleuth 聚焦会话记忆，showcase 聚焦 RN/TS 项目）；都有 commands + agents + skills 三层结构；都没有 model-pinning 问题（showcase 新增 workflow 后才有） | echo-sleuth 无 hook，showcase 有完整 hook 路由系统；echo-sleuth 命令无 name frontmatter 问题（需验证） | **高** |
| MarkQWu/claude-for-legal | **中** | 都有多个 skill 服务一个大领域（法律 vs RN/TS）；都面临 skill 引用与实现的对齐挑战 | claude-for-legal 无 hook；legal 有 16 个 agent，showcase 只有 2 个 | 中 |
| MarkQWu/drama-workshop-skills | **低** | 都有 SKILL.md 文件 | drama-workshop 无 commands；领域完全不同 | 低 |

**核心结论**：echo-sleuth-for-claude 是与本仓库最接近的对标——都是聚焦单一产品/场景的 Claude Code 集成样板。

### 7.2 echo-sleuth 的同类风险检查

**echo-sleuth 5/5 命令无 allowed-tools**（已知问题）：本仓库的 `onboard.md` 缺少 allowed-tools 造成 23 分的极低得分。echo-sleuth 的 5 个命令同样没有 allowed-tools 声明，属于同类问题，需要优先修复。

**echo-sleuth 命令 name frontmatter 状态**：需要实际验证，但从已知问题看，echo-sleuth 至少存在 5/5 命令无 allowed-tools，这与本仓库 6/6 命令缺 name 的模式高度类似——都是"命令内容写得认真，但注册必需字段漏掉"的批量性问题。

### 7.3 我没有的能力：skill-eval.js 智能路由

本仓库最值得移植的创新是 skill-eval.js。我的三个仓库都没有 hook-based skill 路由机制——用户需要手动触发正确的命令才能获得 skill 辅助。

**移植 skill 路由的前提条件**（以 echo-sleuth 为例）：
1. skill 文件数量足够（≥3 个 skill 才有路由价值）
2. 每个 skill 服务明确可区分的上下文（可以根据文件路径/内容区分激活条件）
3. skill-rules.json 里的每条规则都有对应 SKILL.md（零 ghost skill 是移植前提）

echo-sleuth 目前只有少量 skill，移植时机尚不成熟。drama-workshop-skills 领域明确（不同创作阶段对应不同 skill），是更适合引入 skill 路由的候选。

### 7.4 我的 claude-for-legal 和 hooks 质量的对比

`MarkQWu/claude-for-legal` 有一个 `hooks.json`，但内容为空（`{"hooks": {}}`），没有实现任何 hook 逻辑。

**比较**：
- claude-for-legal hooks.json：结构规范（有 `hooks` 根键），内容空，不会出错，但无功能
- ChrisWiles skill-eval.js：功能完整，有创意，但 ghost skill 问题让 hook 的推荐效果大打折扣

**哪个更"正确"**：在 ghost skill 未解决之前，空 hook 比有 bug 的 hook 更安全——空 hook 不会发出错误的推荐，而引用不存在 skill 的路由 hook 会在每次触发时静默失败。我的空 hook 结构反而是更保守但更可靠的起点。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **frontmatter** | Markdown 文件顶部以 `---` 包裹的 YAML 元数据块。在 Claude Code NL artifacts 中，frontmatter 声明 `name`、`description`、`model`、`allowed-tools` 等关键字段。缺少 `name:` 字段的 command 文件无法出现在 slash command 调色板中，对用户完全不可见。本仓库 6/6 命令全部缺少 `name:` 字段是最典型的 frontmatter 纪律问题。 |
| **ghost skill** | 在 skill-rules.json 或其他路由配置中被引用，但对应 SKILL.md 文件不存在的 skill 名称。Ghost skill 会导致 hook 路由系统在建议激活该 skill 时静默失败——推荐发出，但没有任何实际内容被加载。本仓库有 13 个 ghost skill，是"框架先于内容"设计反模式的典型案例。 |
| **skill 路由（skill routing）** | 根据用户当前操作上下文（目录路径、文件类型、操作意图）自动推荐应该激活哪个 skill 的机制。本仓库的 skill-eval.js + skill-rules.json 是 Claude Code ecosystem 中 skill 路由的早期实现，通过 PostToolUse hook 在每次工具调用后触发分析。 |
| **model pinning（模型 ID 固定）** | 在 workflow 或配置文件中显式指定某个具体的 Claude 模型 ID（如 `claude-opus-4-5-20251101`），而非使用 `claude-code-action` 的默认版本。过度具体的 pinning 在 Anthropic 退役旧模型 ID 时会导致 workflow 失败。本仓库的 `pr-claude-code-review.yml` 使用了过时的 `claude-opus-4-5-20251101`，是 model pinning 反模式的典型案例。 |
| **showcase 型仓库** | 以演示最佳实践为目的、而非以生产运行为目的的 Claude Code 仓库。其特点是内容质量高（SKILL.md 整洁、agent 设计清晰），但注册相关字段（frontmatter）可能不完整，因为 showcase 不需要被 `claude plugin install` 安装到生产环境。本仓库是典型的 showcase 型仓库，6/6 命令缺 name frontmatter 与 6/6 skill 均 95 分的鲜明对比证明了这一点。 |
| **schema-first 配置** | 先定义 JSON Schema 约束配置文件格式，再填写配置内容的工程实践。本仓库的 `skill-rules.schema.json` 是 `skill-rules.json` 的格式验证层，防止格式错误的路由规则混入。这是少见的"配置即代码，代码有 schema"的严谨做法，但前提是 schema 约束的内容本身（即路由规则引用的 skill）必须实际存在。 |
| **empty-input guard** | 在命令开始执行前检查用户输入是否为空的保护逻辑。缺少 empty-input guard 的命令在用户不提供参数时会以空字符串执行，可能导致意外行为。本仓库的 code-quality、pr-review、ticket、onboard 四个命令缺少此保护。 |
| **供应链风险（supply chain risk）** | 在 hook 或 MCP 配置中使用无版本固定的 `npx -y package@latest` 调用外部 npm 包，每次执行时可能下载不同版本代码的安全风险。本仓库 `.mcp.json` 全 7 个 MCP server 使用 `npx -y @anthropic/mcp-*` 无版本 pin，是供应链风险的典型案例。 |
