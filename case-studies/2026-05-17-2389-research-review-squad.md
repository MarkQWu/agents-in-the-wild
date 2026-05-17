# 2389-research/review-squad — 学习案例

**仓库**：https://github.com/2389-research/review-squad
**分析日期**：2026-05-17
**NLPM 评分**：96/100

## 仓库概览

review-squad 是一个专注于代码/项目审查的 Claude Code 插件，通过调度四种不同"角色"的子代理来完成全面审查：experts（专家并行审查）、normies（普通用户模拟，顺序执行，使用浏览器 MCP）、regulars（经验用户视角，顺序执行）、well-actually（刁钻吹毛求疵型，顺序执行）。NL 工件只有 5 个（4 个 skill + 1 个 CLAUDE.md），却能覆盖"多视角自动化审查"这个复杂场景。整体设计极为精简，是一个展示"少即是多"的优秀范例。

## 质量评分

| 文件 | 分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 80 | 无 frontmatter，无调用示例 |
| skills/experts/SKILL.md | 98 | 第 95 行有模糊量词 "relevant" |
| skills/normies/SKILL.md | 99 | Clean |
| skills/regulars/SKILL.md | 99 | Clean |
| skills/well-actually/SKILL.md | 99 | Clean |
| .claude-plugin/plugin.json | 100 | N/A |

评分维度：触发词清晰度高，各 skill 明确定义了调用模式（并行 vs 顺序）和使用场景；四个 skill 没有任何重叠的 input→output 示例但行为定义本身很精确；plugin.json 满分。主要扣分来自 CLAUDE.md 的非 NL 格式问题。

## 值得借鉴的模式

**用角色分离解决"全能代理"陷阱** → 把一次完整的代码审查拆成四种认知视角（专家/普通用户/常规用户/吹毛求疵），每个 skill 只负责一种视角，而不是让一个代理"扮演多种角色"。这样每个 skill 的行为约束非常聚焦，不会在不同视角间漂移。→ 复杂工作流应沿"认知视角"或"输出物类型"做正交分解，而不是做功能叠加。

**调度策略显式化（并行 vs 顺序）** → CLAUDE.md 明确说明：experts 支持并行（代码分析，不用浏览器）；normies/regulars/well-actually 顺序执行（使用浏览器 MCP，顺序避免资源竞争）。这个策略在 CLAUDE.md 文档化，在各 skill 内部也有对应说明，两层一致。→ 在多代理项目中，调度策略（并发/顺序）应在协调层和执行层都明确声明，而不是只在一处。

**No-Code Guards 跨 skill 一致实施** → normies 和 regulars 都有 "No-Code Guards" 约束（禁止代理在审查过程中写代码），且该约束在 CLAUDE.md 文档化后，在两个 skill 文件里也verbatim 写入了 agent prompt。这确保约束不依赖外部文档的传导，即使单独调用某个 skill 也有效。→ 安全/限制类约束应直接内嵌在 skill 的执行层，而非只写在顶层文档里。

**plugin.json `commands: []` 的有意设计** → 该插件只暴露 skill，不暴露 slash command，`commands: []` 是有意为之。这说明设计者清楚地区分了两种接口，没有不必要地暴露命令接口。→ 在设计插件时，应明确决策哪些功能应该是 skill（被 AI 主动调用）、哪些应该是 command（被用户主动触发）。

**交叉引用外部插件** → CLAUDE.md 提到 `fresh-eyes-review` 和 `scenario-testing` 作为互补插件，这给用户提供了扩展路径。这种"生态定位"说明设计者在考虑单个插件在更大生态中的位置。→ 在自己的 skill/plugin README 里说明"不适合用这个的场景应该用什么"，避免用户误用。

## 缺陷分析

**CLAUDE.md 无 frontmatter** → 根本原因：CLAUDE.md 在 Claude Code 中是"项目规则"文件，作者将其作为人类可读文档来写，而非 NL 工件。但 NLPM 期望能够通过 frontmatter 中的 `name` 和 `description` 字段做机器化发现。没有 frontmatter 意味着扫描器无法将其纳入工件图谱，用户也无法通过 NLPM 自动发现该插件的元信息。**自查：我的 CLAUDE.md 有 frontmatter 吗？如果没有，NLPM 能扫描它吗？**

**CLAUDE.md 无调用示例** → CLAUDE.md 用散文描述了每个 skill 的用途，但没有给出一个"用户发出 X，Claude 做 Y"的具体映射示例。用户不清楚应该如何触发这四个 skill，以及期望得到什么格式的输出。根本原因：作者写的是设计文档，而非使用手册。**自查：我的 CLAUDE.md 有没有告诉用户怎么用，而不只是设计是什么？**

**experts/SKILL.md L95 "relevant" 模糊量词** → "always suggest what's relevant" 没有定义什么算相关——是技术栈匹配？还是业务场景匹配？这个词给了 AI 太大的自由裁量空间，可能导致审查建议在不同项目中质量不一致。根本原因是写 prompt 时用了自然语言直觉而非可验证标准。**自查：我的 skill 文件里有没有未定义标准的 "relevant"、"appropriate"、"suitable" 等词？**

## 优化机会

1. **问题**：CLAUDE.md 缺 YAML frontmatter，NLPM 无法机器发现 **修法**：在文件顶部添加包含 `name` 和 `description` 的 frontmatter 块 **预期影响**：NLPM 评分从 80 提升至约 90，插件元信息可被自动化工具发现

2. **问题**：CLAUDE.md 无调用示例，用户不知道如何触发 skill **修法**：为每个 skill 添加一个"触发短语 → 行为预期"的具体示例 **预期影响**：降低用户学习成本，减少误用

3. **问题**：experts/SKILL.md L95 "relevant" 无操作化定义 **修法**：将 "suggest what's relevant" 改为 "suggest reviewers that directly address gaps in the detected tech stack (e.g., i18n reviewer for multi-locale sites)" **预期影响**：减少 AI 审查结果在不同技术栈间的一致性差异

## 对照我的项目

**自查：我有没有犯同样的错？**

- **CLAUDE.md 双重定位问题**：我的 CLAUDE.md 是写给人读的设计文档，还是写给 NLPM 扫描器的机器可读工件？如果两者都想要，需要加 frontmatter + 示例块，让它同时对 AI 和 NLPM 有意义。
- **调度策略文档化**：如果我的项目有多个 skill 或 agent，调度策略（哪些可以并行、哪些必须顺序、为什么）有没有在顶层文档和执行层都写清楚？
- **约束内嵌 vs. 外挂**：安全限制和行为边界是否只写在 README/CLAUDE.md 里？如果 skill 被单独调用，这些约束还会生效吗？参考 review-squad 的 No-Code Guards 模式，把关键约束 inline 进 skill 的 prompt 体。
- **"这个 skill 不适合什么"的缺失**：我的 skill 文件有没有说明"如果你要做 X，应该用 Y 而不是我"？这种"反向定位"能防止误用。
- **模糊量词的最后一道防线**：即便是 96 分的高质量仓库也有 "relevant" 这样的漏网之鱼，说明模糊量词问题需要专门扫描，不能靠写作直觉避免。

## 灵感

- **多视角审查模板**：可以借鉴 review-squad 的四角色模式，为自己的项目设计"专家/新手/恶意用户"三视角的测试 skill，系统性地暴露不同类型的问题。
- **浏览器 MCP 顺序化原则**：使用浏览器 MCP 的代理应该顺序执行以避免资源竞争——这个工程决策值得在自己的多代理项目中采纳。
- **插件生态定位**：在下一个插件的 CLAUDE.md 里，专门写一段"何时用这个，何时用别的插件（并列举具体替代）"，帮助用户在插件生态中找到正确的工具。
