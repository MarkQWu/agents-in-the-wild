# FlorianBruniaux/claude-code-ultimate-guide — 学习案例

**仓库**：https://github.com/FlorianBruniaux/claude-code-ultimate-guide
**Stars**：3604 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-21（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `curl-pipe-bash-risk`, `examples-driven`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是一个以"终极指南"为定位的 Claude Code 最佳实践百科全书。3604 stars，远超多数工具型插件，核心原因是**它不是工具，它是教材**。

目录结构体现了这种双重身份：
- `.claude/`：真正可运行的插件（agents、commands、skills、hooks）
- `examples/`：演示用的示例文件（agents、commands、scripts、skills）
- `claudedocs/`：预生成的 .txt 格式文档，供 Claude 离线读取
- `scripts/`：工程化脚本（install-templates.sh、update-cc-releases.sh）
- `CITATION.cff`：学术引用元数据——极少见于 Claude 插件，说明作者有意将其定位为可引用的软件工作

当前 HEAD（2026-05-21）共有 23 agents、52 commands、75 个 skill 相关文件。相比审查时（24/66/10）略有缩减，说明维护者在主动清理冗余。

### 1.2 架构剖析

**分层设计**：`.claude/` 是生产层，`examples/` 是教学层。两层之间有意保持镜像关系——每个生产 artifact 都有对应的教学示例。

**MCP 自集成**：`.claude/agents/guide-reviewer.md` 声明了 `mcp__claude-code-guide__search_guide` 和 `mcp__claude-code-guide__read_section` 两个工具，意味着该仓库配套了一个 MCP 服务，Claude 审查文档时可以通过 MCP 搜索自身的指南内容。这是少见的"工具审查自身文档"模式。

**claudedocs 离线缓存**：`claudedocs/audit-reviews/` 下存放 .txt 格式的预处理文档。这是避免每次让 Claude 发起 web fetch 的工程解法——将外部文档离线化，降低延迟和网络依赖。

**多语言内容**：`.claude/agents/guide-reviewer.md` 的 agent body 用法语书写（"Tu es un relecteur expert de documentation technique"），使用 `model: haiku`。作者选择母语写提示词，这在主流插件中属于例外。

### 1.3 设计思路 / 方法论

**示例驱动架构（Examples-Driven Architecture）**：每一个最佳实践都以独立文件呈现——`examples/agents/adr-writer.md`、`examples/commands/pr.md`、`examples/skills/design-patterns/SKILL.md`。这与"尽量少文件、高内聚"的工具型设计相反，是"宁可有 100 个文件，每个文件讲清一件事"的教学取向。

**代价**：100 个 artifact 平均 80/100，而工具型项目（如 BayramAnnakov 类）可能 8 个 artifact 平均 99/100。广度换深度，这是明确的设计选择，不是缺陷。

**ADR 模式**：`examples/agents/adr-writer.md` 实现了三层 ADR 格式，意图用 Claude 帮助团队记录架构决策。这表明作者不只是在演示 Claude Code 用法，而是在构建一套完整的工程协作流程。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式 | 分类 | 典型文件 |
|------|------|----------|
| 示例驱动架构 | 教学型插件 | `examples/` 全目录 |
| claudedocs 离线文档缓存 | 工程优化 | `claudedocs/audit-reviews/*.txt` |
| MCP 自集成 | 高级集成 | `.claude/agents/guide-reviewer.md` |
| command→skill 迁移 | 演进模式 | `examples/commands/diagnose.md` |
| 学术引用声明 | 定位信号 | `CITATION.cff` |
| 母语提示词 | 本地化实践 | guide-reviewer agent body（法语） |

### 2.2 适用场景

- **claudedocs 模式**适用于任何需要让 Claude 频繁读取同一批外部文档的项目（框架手册、API 参考、内部 wiki）。预生成 .txt 比每次 web fetch 稳定得多。
- **示例驱动架构**适用于教学导向的插件，或需要让团队成员快速理解"这是什么样的 agent/command"的场景。不适合追求极致质量的工具型插件。
- **MCP 自集成**适用于文档庞大、需要检索的项目。工具审查自身内容，避免 context window 塞满原始 markdown。

### 2.3 与其他架构对比

与极简型插件（单一功能、少量 artifact）相比，本仓库的优势在于覆盖场景广、入门门槛低；劣势是单个 artifact 质量参差，名称字段缺失等机械错误集中暴露。

与工具型插件（如 MCP server wrapper）相比，本仓库更接近"活文档"——它的价值随时间积累（更多示例、更多场景），而非随功能叠加。

### 2.4 改进空间

1. **name 字段补全**：5 个 ccguide commands（daily、diff-docs、init-docs、refresh-docs、search-docs）仍缺 `name:` 字段。这是 BUG，不是设计选择。
2. **example block 补全**：devops-sre、architecture-reviewer、security-auditor、planner、implementer 等 agent 缺少 example 块。
3. **README 文件归位**：3 个 README.md 被放入 agents 目录，导致被误识别为 agent 定义。目录即声明——文件放错了位置，工具就会误读意图。

---

## 三、过去审查发现（历史快照）

### 3.1 当时质量评分（NLPM）

- **总分**：80/100，Security：BLOCKED
- **Artifacts**：100 个（24 agents、66 commands、10 skills）
- **Bugs**：12 个，**Quality Issues**：19 个，**Security Findings**：10 个（1 Critical、2 High、4 Medium、3 Low）

| 类别 | 均分 | 主要拖分项 |
|------|------|-----------|
| Agents | 77.5/100 | 3 个 README.md 误分类，各扣 55–70 分 |
| Commands | 79.4/100 | 9 个缺 name 字段（BUG），2 个安全扣分 |
| Skills | 88.8/100 | 5 个缺 allowed-tools |

### 3.2 当时值得借鉴的模式

- `examples/skills/design-patterns/SKILL.md`：95/100。5 个阶段、3 种模式、完整 example block。结构清晰，可作为 skill 写作范本。
- `examples/skills/audit-agents-skills/SKILL.md`：95/100。包含行业数据、3 种模式、JSON+Markdown 双输出格式。
- `.claude/agents/guide-reviewer.md`：93/100。完整 frontmatter、指定 haiku 模型、有 example block。
- `examples/commands/pr.md`：93/100。内含复杂度评分逻辑、分拆建议、具体 bash 命令。
- `examples/agents/adr-writer.md`：92/100。三层 ADR 格式、明确的适用时机、模型选择理由。

### 3.3 当时的缺陷

**Critical（供应链攻击）**：
- `examples/commands/diagnose.md` 第 38–39 行：
  ```
  curl -sL "https://raw.githubusercontent.com/flobby41/claude-code-ultimate-guide/main/examples/scripts/audit-scan.sh" | bash -s -- --json 2>/dev/null
  ```
  该 URL 指向 `flobby41` 账号，**不是**仓库作者 `FlorianBruniaux`。这是典型供应链攻击风险：任何人控制 flobby41 账号即可替换脚本，在用户机器上执行任意代码。

**High**：
- `examples/commands/sonarqube.md`：在 /tmp 创建 fetch_sonar.sh，chmod+x 后执行。
- `scripts/install-templates.sh`：设计为 `curl -fsSL {URL} | bash` 方式运行。

**Medium**：
- `.claude/commands/update-infos-release.md`：硬编码绝对路径 `/Users/florianbruniaux/Sites/perso/...`，完全不可移植。

**Bugs（机械性缺失）**：
9 个 command 文件缺 `name:` 字段：ccguide/diff-docs.md、ccguide/init-docs.md、ccguide/refresh-docs.md、ccguide/search-docs.md、ccguide/daily.md、recipe-template.md、handoff/resume-handoff.md、handoff/create-handoff.md、handoff/update-handoff.md。

3 个 cyber-defense agent 缺 Write 工具。

### 3.4 当时的优化机会

- README.md 文件应移出 agents 目录，或改名为非 .md 格式，避免被分类为 agent。
- 技能文件补充 allowed-tools，明确权限边界。
- sonarqube.md 中的临时脚本模式应替换为直接 curl → 参数传递，或使用官方 sonarqube CLI。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 审查时 | 当前 HEAD (2026-05-21) | 状态 |
|------|--------|------------------------|------|
| C1：diagnose.md curl-pipe flobby41 | 存在，第 38–39 行 | 文件已改为迁移说明存根，curl-pipe 完全删除 | ✅ 已修复 |
| ccguide commands 缺 name 字段 | 9 个文件 | daily、diff-docs、init-docs、refresh-docs、search-docs 仍缺 | ❌ 仍存在 |
| agents 缺 example block | 6 个 | devops-sre、architecture-reviewer、security-auditor、planner、implementer 仍缺 | ❌ 仍存在 |
| H2：install-templates.sh curl-pipe | 存在 | 文件仍存在 | ⚠️ 待确认 |

**最重要的修复**：Critical C1 完全消除。`examples/commands/diagnose.md` 现在只有一行注释："# Moved to Skills / This command was migrated to a skill in Claude Code 2.1.3. See: examples/skills/diagnose/SKILL.md"。这体现了 command→skill 迁移模式：随着 Claude Code 能力演进，原来需要用 command 脚本实现的功能，可以抽象为更高层的 skill。

### 4.2 架构演进

1. **claudedocs 目录新增**：`claudedocs/audit-reviews/` 存放预生成 .txt 文档。这是审查后新增的重要模式——从"每次让 Claude 去 web fetch"进化为"预缓存关键文档"。
2. **工程化成熟**：新增 `.pre-commit-config.yaml`、GitHub Actions workflows（link-check.yml、rebuild-guide-exports.yml）。仓库从纯文档演进为有 CI 保障的工程项目。
3. **MCP 自集成**：guide-reviewer 现在使用仓库自己的 MCP server 搜索指南内容。审查时不存在此能力。
4. **学术定位强化**：新增 `CITATION.cff`，为软件引用提供标准元数据。

### 4.3 新增的可学习模式

**claudedocs 离线文档缓存**：将外部 web 文档预转换为 .txt 存入仓库，Claude 读取本地文件而非发起网络请求。优点：稳定、快速、可版本控制。适合文档内容相对稳定的场景。

**MCP 自集成（Self-Referential MCP）**：仓库内嵌 MCP server，agent 通过 MCP 工具检索自身文档。这使得 Claude 可以在有限 context 内处理大型知识库，只拉取相关片段。

**command→skill 迁移存根**：废弃的 command 文件保留为迁移说明，而非直接删除。用户升级后不会遭遇 404，而是被引导到新位置。这是对用户友好的 deprecation 策略。

---

## 五、校准

### 5.1 我已经在做对的

- **写 CLAUDE.md**：像本仓库一样，用 CLAUDE.md 声明架构、约定、开发规范。
- **使用 allowed-tools**：每个 agent 声明权限边界，不留宽泛的默认值。
- **避免 curl-pipe-bash**：不在 agent/command 中内嵌 `curl ... | bash`。本案例的 Critical C1 是最直接的反面教材。
- **name 字段完整**：每个 command 都有 `name:` 字段，避免本仓库 9 个 BUG 的重演。

### 5.2 挑战 / 验证

**挑战一：示例驱动 vs. 工具驱动的根本区别**

本仓库揭示了一个重要认知：**Claude 插件可以是教材，而不只是工具**。3604 stars 的积累靠的不是功能深度，而是覆盖广度和教学清晰度。如果我在构建教学型仓库，需要接受"更多文件、更低单文件质量"的权衡，而不是强行追求每个 artifact 满分。

**挑战二：目录即声明**

3 个 README.md 被误识别为 agent，原因是它们被放在 agents 目录下。这验证了"目录即声明"原则——文件放错位置，工具就会误读意图，不管文件内容写的是什么。我需要检查自己的仓库，确保 docs/README 类文件不在任何分类目录（agents/commands/skills）下。

**验证点**：本仓库的 Critical C1 已修复，但 name 字段缺失（BUG 级别）依然存在。这说明：安全问题修复优先级高，机械性质量问题容易被推迟。对我自己项目的启示：定期用自动化工具扫描 name 字段等机械缺失，不依赖人工记忆。

---

## 六、行动

### 6.1 自查动作

1. **扫描 name 字段**：在项目 `.claude/commands/` 下执行 `grep -rL "^name:" --include="*.md"`，确认所有 command 都有 name 字段。
2. **扫描 curl-pipe**：`grep -r "curl.*|.*bash\|curl.*|.*sh" --include="*.md" .`，确认无 curl-pipe-bash 模式。
3. **扫描目录错位**：检查 `.claude/agents/`、`.claude/commands/`、`.claude/skills/` 下是否有 README.md 或非 artifact 文件。
4. **扫描 allowed-tools**：`grep -rL "allowed-tools" .claude/agents/ --include="*.md"`，找出缺权限声明的 agent。
5. **扫描硬编码路径**：`grep -r "/Users/\|/home/[a-z]" --include="*.md" .claude/`，找出不可移植的绝对路径。

### 6.2 灵感 → 实施路径

**灵感 A：claudedocs 模式**
- 场景：NLPM 项目中，Claude 经常需要读取 Claude Code 官方文档。
- 实施：创建 `claudedocs/` 目录，将 Claude Code 官方文档关键章节预转为 .txt，加入 .gitignore 更新脚本（类似 update-cc-releases.sh）。
- 收益：稳定性 + 速度 + 版本可追溯。

**灵感 B：command→skill 迁移存根**
- 场景：废弃一个旧 command 时。
- 实施：不直接删除，保留文件并替换内容为一行迁移说明："# Deprecated: migrated to skills/xxx/SKILL.md in v0.9.0"。
- 收益：用户升级不破坏，工具分类扫描不会误报"文件消失"。

**灵感 C：CITATION.cff**
- 场景：如果 NLPM 发展到需要被学术或工程团队引用。
- 实施：添加 `CITATION.cff`，声明版本、作者、DOI（如有）。
- 收益：提升可信度，方便引用，对 stars 增长有正向信号。

---

## 七、术语表

| 术语 | 解释 |
|------|------|
| **供应链攻击（supply chain attack）** | 通过污染上游依赖（脚本、包、库）在下游用户机器执行恶意代码。本案例 C1 中，curl 拉取来自不同账号（flobby41）的脚本直接执行，是典型供应链向量。 |
| **curl-pipe-bash** | `curl <URL> \| bash` 模式的简称。将远程脚本下载后不经检查直接执行。安全风险极高，NLPM security scanner 将其列为 Critical 级别检测项。 |
| **示例驱动架构（Examples-Driven Architecture）** | 以大量独立示例文件为核心的仓库设计风格。每个最佳实践对应一个文件，牺牲单文件深度换取覆盖广度。适合教学型插件，不适合追求极致质量的工具型插件。 |
| **manifest discipline** | 在 agent/command/skill 的 frontmatter（YAML 头）中完整填写所有必要字段（name、description、allowed-tools 等）的工程纪律。缺失任何字段视为 BUG。 |
| **ADR（Architecture Decision Record）** | 架构决策记录。记录"为什么做这个决定"而非只记录"做了什么"。本仓库的 adr-writer.md 实现了三层格式的 ADR 生成 agent。 |
| **MCP server** | Model Context Protocol server。Claude 可通过 MCP 协议调用外部服务的工具。本仓库的 guide-reviewer agent 调用仓库自配的 MCP server 来搜索指南内容。 |
| **claudedocs 离线文档缓存** | 将外部 web 文档预转换为 .txt 存入仓库本地，供 Claude 直接读取，避免每次发起 web fetch。本仓库 `claudedocs/` 目录是此模式的实现。 |
| **command→skill 迁移** | 随 Claude Code 能力演进，将原本用 command 脚本实现的功能抽象为更高层的 skill 的重构模式。废弃的 command 文件保留为迁移存根（stub），引导用户到新位置。 |
| **vague quantifier（模糊量词）** | 如"very"、"many"、"some"、"better"等缺乏具体度量的词汇。NLPM 将其标记为质量扣分项，因为它们无法被 Claude 或人类验证。 |
| **security-blocked** | NLPM audit workflow 发现 Critical 级安全问题后打上的 label。带此 label 的 issue 无法进入 contribute 流程，需人工审查后方可解除。 |
