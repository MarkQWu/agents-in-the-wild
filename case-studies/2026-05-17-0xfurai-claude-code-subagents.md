# 0xfurai/claude-code-subagents — 学习案例

**仓库**：https://github.com/0xfurai/claude-code-subagents
**分析日期**：2026-05-17
**NLPM 评分**：74/100

## 仓库概览

这是一个专为 Claude Code 设计的子代理集合库，包含 100+ 个面向主流技术栈的开发专家代理（从 React/Vue/Angular 到 Kafka/Cassandra/Prisma），覆盖前端、后端、数据库、测试、DevOps 等领域。所有代理文件结构高度一致，采用统一的四段式模板（Focus Areas / Approach / Quality Checklist / Output），并固定使用 `claude-sonnet-4-20250514` 模型。整体风格是"广而全"的工具箱，而非深度定制化的工作流。

## 质量评分

| 文件 | 分数 | 主要问题 |
|---|---|---|
| agents/sqs-expert.md | 10 | 整个文件缩进 4 个空格，`---` 未出现在第 1 列，frontmatter 完全无法解析 |
| agents/prisma-expert.md | 67 | 9 个模糊量词（appropriate×2, comprehensive×3, efficient×3 等），无示例 |
| agents/clojure-expert.md | 71 | 7 个模糊量词，无示例 |
| agents/angular-expert.md | 73 | 6 个模糊量词，无示例 |
| agents/react-expert.md | 77 | 4 个模糊量词（appropriate×2, optimal, comprehensive），无示例 |
| agents/rust-expert.md | 79 | 3 个模糊量词（comprehensive, meaningful, minimize），无示例 |
| agents/gin-expert.md | 75 | description 使用元描述语（"Create a Claude Code Agent that is…"），无示例 |
| agents/jasmine-expert.md | 79 | frontmatter 各字段间有空行（非标准格式） |

评分维度：全部 100 个代理均缺少 `## Examples` 区块（-15），且普遍存在 3–9 个模糊量词（额外 -8 到 -20）。

## 值得借鉴的模式

**模板一致性** → 100 个文件结构完全统一，Cross-component 无漂移问题，无破损引用、无术语不一致 → 可在自己的多代理项目中使用共享模板文件，靠 lint 保证结构一致性而非依赖记忆。

**模型版本固定** → 所有代理明确写入 `model: claude-sonnet-4-20250514`，避免因默认模型升级导致行为变化 → 在生产场景中，代理 frontmatter 应始终固定具体模型 ID，而非依赖"最新"。

**聚焦单一技术域** → 每个代理只负责一个框架/库（如 kafka-expert.md、swiftui-expert.md），不跨域混搭 → 写 skill 时应抵制"大而全"的诱惑，一个文件只解决一类问题。

**Focus Areas 前置** → 每个代理第一个区块就明确列出关注领域，AI 不必读完全文才知道该做什么 → 自己的 skill 文件也应把最重要的约束放在最前面。

**Quality Checklist 收尾** → 在代理 prompt 末尾加入 checklist，给 AI 一个自检锚点 → 对于复杂输出任务，在 prompt 末尾加"完成前请确认：1. xxx 2. xxx"可以提升完成质量。

## 缺陷分析

**缺少任何示例（100个文件全军覆没）** → 根本原因：模板设计从未要求 `## Examples` 区块，一旦遗漏写入模板，所有从模板复制出来的文件就都没有示例。没有示例意味着 AI 无法通过对比来校准自身输出，也意味着用户无法在安装前预判代理的实际行为。**自查：我有没有犯同样的错？** 我自己的 SKILL.md 里是否有至少一个 input → output 的具体例子？

**模糊量词泛滥（appropriate/comprehensive/robust/efficient）** → 根本原因：模板本身就用了这些词，复制时没有替换成可测量的标准。这些词对 AI 没有实际约束力——"write comprehensive tests" 比 "tests must achieve ≥80% branch coverage" 弱得多。**自查：我是否也在用 "appropriate" "effective" 这类词蒙混过关？**

**sqs-expert.md frontmatter 整体缩进 4 格** → 根本原因：复制粘贴时编辑器自动缩进，而项目缺少 frontmatter 格式校验（如 pre-commit hook）。结果是这个代理的 name/model/description 字段全部失效，不会被 Claude Code 注册。**自查：我的文件有没有被编辑器静默缩进过？**

**gin-expert.md 使用元描述语** → description 写成 "Create a Claude Code Agent that is an expert in…" 而非直接描述代理能做什么。Claude Code UI 直接展示这个 description，元描述会让用户困惑——他们看到的是"创建一个代理"，而不是"这个代理能帮你做 xxx"。**自查：我的 description 字段有没有从代理的视角而非从"如何创建"的视角描述？**

**没有 pre-commit 格式校验** → 100 个文件的质量问题完全依赖人工审查，没有任何自动化守卫。这在个人项目可以接受，但在一个被 859 颗星关注的公共库里，这是技术债。**自查：我有没有在 CI 里跑 nlpm-check？**

## 优化机会

1. **问题**：sqs-expert.md frontmatter 完全失效 **修法**：删除整个文件的 4 格前缀缩进，让 `---` 出现在第 1 列 **预期影响**：该代理从完全不可用变为可被 Claude Code 注册，高影响

2. **问题**：所有 100 个代理无示例 **修法**：在模板里增加 `## Examples` 区块，用一个具体的用户请求→代理输出映射来演示行为 **预期影响**：整体 NL 评分从 74 提升至约 85+，用户对代理行为的预期对齐度大幅提升

3. **问题**：模糊量词系统性泛滥 **修法**：在模板中把 "appropriate", "comprehensive", "robust" 替换为有数字锚点的标准（"meets project linting rules", "covers happy path, edge case, and error path" 等） **预期影响**：AI 遵循约束时有明确依据，减少歧义输出

## 对照我的项目

**自查：我有没有犯同样的错？**

- **模糊量词**：检查自己 SKILL.md 中的 "appropriate", "relevant", "comprehensive", "robust" 等词，每出现一次都问：能否替换为可验证的标准？
- **缺少示例**：检查每个 agent/skill 文件，是否有至少一个具体的 input → output 示例块？如果没有，这是最优先的补充项。
- **模板驱动的批量错误**：如果自己有多个结构相似的文件，务必在模板层面检查质量，而非逐文件审查——因为模板缺陷会被等比复制到所有文件。
- **Description 字段视角**：自己的 skill/agent description 是否从"该工件能帮用户做什么"的角度写的，而非"这个文件定义了什么"？
- **格式校验**：是否在项目里加了 `nlpm-check` 作为 pre-commit 步骤，防止 frontmatter 缩进等机械错误悄悄上线？

## 灵感

- **批量质量守卫**：在有多个同构 NL 文件的项目里，可以写一个 pre-commit 脚本检查所有 agent/skill 文件是否都有 `## Examples` 区块——这比手动审查靠谱得多。
- **模板字段强制化**：与其靠约定，不如在模板文件里把可选区块标为 `<!-- REQUIRED: 至少一个示例 -->`，让遗漏变成显眼的标记而不是静默缺失。
- **模糊词词表**：建一个项目级的"禁用词表"（appropriate, comprehensive, robust, efficient, relevant, adequate），用 grep 在 CI 里拦截新增的模糊量词。
