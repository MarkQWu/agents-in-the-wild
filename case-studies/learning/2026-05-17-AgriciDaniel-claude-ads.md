# AgriciDaniel/claude-ads — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-ads
**Stars**：2377 | **来源**：xiaolai 上游审查
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD）
**主题标签**：vague-quantifier, template-design, manifest-discipline, cross-reference, examples-driven

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-ads 是一个专为数字广告场景设计的 Claude Code 插件，涵盖 Google Ads、Meta Ads、TikTok Ads、LinkedIn Ads、Microsoft Ads、YouTube Ads、Apple Search Ads 等主流平台的审查与创作工作流。整个插件由 32 个 NL 制品组成：10 个 agent、20 个 platform skill、1 个 orchestrator skill（ads/SKILL.md）和 1 个 plugin.json 清单。当前 HEAD 提交为 402ba63b9af56c0573766fb0ae8d7b56dc13a674。

该插件的核心价值主张是：将广告从业者在不同平台上的审查与创作经验系统化，使 Claude 能够担任具备领域知识的广告顾问角色，而不仅仅是通用写手。

### 1.2 架构剖析

插件采用三层架构：

**第一层：平台调度层**
`ads/SKILL.md` 充当 orchestrator，根据用户指定的平台将任务路由到对应的 platform skill（ads-google、ads-meta、ads-tiktok 等 20 个子技能）。这一设计将平台差异封装在子技能内部，对上层保持接口一致性。

**第二层：Agent 层（10 个 agent）**
- 审查类（6 个）：audit-google、audit-meta、audit-compliance、audit-budget、audit-creative、audit-tracking，每个 agent 对应广告运营的一个垂直审查维度
- 创作类（4 个）：creative-strategist、visual-designer、copy-writer、format-adapter，覆盖从策略到最终物料的创作链路

**第三层：知识底座**
- `ads/references/*.md`：被所有审查 agent 交叉引用的领域参考文档
- `ads/research-sources/`：包含 claude-research.md、gemini-research.md 和多份 google-*.md——将外部研究正式化为插件的领域知识
- 6 个 Python 脚本：analyze_landing.py、capture_screenshot.py、fetch_page.py、generate_image.py、generate_report.py、url_utils.py，提供落地页分析、截图、图像生成等自动化能力

### 1.3 设计思路 / 方法论

**研究先行（Research-backed design）**
与大多数 Claude Code 插件不同，claude-ads 在 `ads/research-sources/` 下保留了设计决策的原始研究依据。这意味着插件中的审查清单和创作准则不是主观猜测，而是来自对 Google、Gemini 等平台官方文档和研究报告的系统性整理。

**术语锁定（Terminology locking）**
"answer-first formatting"、"citation capsule"、"information gain marker" 等专业术语在 blog-write、blog-rewrite、blog-analyze、blog-geo 及相关 agent 中保持完全一致的拼写和语义。这种跨文件的术语一致性是人工刻意维护的结果，极大降低了 Claude 在不同上下文间切换时的语义漂移风险。

**交叉引用完整性（Cross-reference integrity）**
所有审查 agent 引用的 `ads/references/*.md` 文件均真实存在且链接有效。这一点在拥有 32 个制品的插件中尤为难得——通常规模越大，悬空引用越难避免。

---

## 二、过去审查发现（2026-04-17 历史快照）

### 2.1 当时质量评分（NLPM）

**总分：99/100**（安全状态：REVIEW，无 Critical，无 High，3 Medium，2 Low）

| 制品 | 得分 | 主要扣分原因 |
|------|------|------------|
| agents/audit-google.md | 93 | "74-check" 与"80 Checks"表头矛盾；"applicable"措辞模糊 |
| agents/audit-meta.md | 93 | "46-check" 与"50 Checks"表头矛盾；"applicable"措辞模糊 |
| agents/copy-writer.md | 95 | 仅有单一示例（-5） |
| agents/format-adapter.md | 95 | 仅有单一示例（-5） |
| CLAUDE.md | 95 | 架构树遗漏 4 个创作 agent |
| agents/audit-budget.md 至 audit-tracking.md | 98 各 | "applicable"措辞模糊 |
| ads/SKILL.md | 98 | orchestration 中"relevant"措辞模糊 |
| agents/creative-strategist.md | 100 | — |
| agents/visual-designer.md | 100 | — |
| 全部 20 个 platform skill | 100 各 | — |

**平均分位极高**：32 个制品中有 22 个（20 个 platform skill + 2 个 agent）达到满分 100，这在同等规模的插件中极为罕见。

### 2.2 当时值得借鉴的模式

1. **大规模满分覆盖**：20 个 platform skill 全部 100/100，说明作者在创建重复结构时使用了高质量的模板，并严格执行了一致性检查。
2. **交叉引用闭合**：`ads/references/*.md` 被所有审查 agent 引用，且引用全部有效——无悬空链接。
3. **领域知识正式化**：`ads/research-sources/` 目录将外部研究资料纳入插件版本控制，确保审查准则有据可查。
4. **Orchestrator 模式**：`ads/SKILL.md` 实现平台级路由，下游 skill 无需感知调度逻辑。
5. **术语一致性**：专业术语跨文件精确对齐，无同义词漂移。

### 2.3 当时的缺陷

1. **数量矛盾（Vague-quantifier 类）**：
   - audit-google.md 第 31 行标注"74-check"，但表头写"80 Checks"——差值 6，Claude 无法判断以哪个为准。
   - audit-meta.md 第 35 行标注"46-check"，但表头写"50 Checks"——差值 4，同样矛盾。

2. **模糊措辞（Vague-word 类）**：
   - 多个审查 agent 使用"applicable"（如"apply applicable rules"），未说明"适用"的判断标准。
   - ads/SKILL.md 中"relevant"同样缺乏操作化定义。

3. **示例不足（Examples-driven 类）**：
   - copy-writer.md 和 format-adapter.md 各仅提供 1 个 `<example>` 块，覆盖范围不足以展示输入多样性。

4. **清单不完整（Manifest-discipline 类）**：
   - CLAUDE.md 架构树列出了 6 个审查 agent，但未列出 creative-strategist、visual-designer、copy-writer、format-adapter 共 4 个创作 agent。

### 2.4 当时的优化机会

- 将 audit-google.md 和 audit-meta.md 中的检查项数量统一为一个权威数字（建议以实际表格行数为准，删除文字标注或同步更新）。
- 为 copy-writer.md 和 format-adapter.md 各补充至少 2 个差异化示例，覆盖不同平台、不同受众、不同文案风格的场景。
- 在 CLAUDE.md 的架构树中补全 4 个创作 agent 条目。
- 将"applicable"替换为可操作的判断条件（如"if the ad type is video, apply rules V1–V3"）。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

基于对当前 HEAD（402ba63b9af56c0573766fb0ae8d7b56dc13a674）的逐项核查：

| 缺陷 | 快照（2026-04-17）| 当前状态 | 结论 |
|------|-----------------|---------|------|
| audit-google.md "74-check" vs "80 Checks" | 存在 | 第 31 行仍为"74-check"，表头仍为"80 Checks" | **仍未修复** |
| audit-meta.md "46-check" vs "50 Checks" | 存在 | 第 35 行仍为"46-check"，表头仍为"50 Checks" | **仍未修复** |
| CLAUDE.md 遗漏 4 个创作 agent | 存在 | creative-strategist、visual-designer、copy-writer、format-adapter 仍未出现在架构树中 | **仍未修复** |
| copy-writer.md 单一示例 | 存在 | `<example>` 计数仍为 1 | **仍未修复** |
| format-adapter.md 单一示例 | 存在 | `<example>` 计数仍为 1 | **仍未修复** |

**结论**：审查发现的 5 项缺陷在一个月后的当前 HEAD 中均未得到修复。这并不影响插件的整体高质量定位（99/100 的综合得分本身已极具说服力），但说明作者对"临界值以上的小缺陷"修复优先级较低。

### 3.2 架构演进

在审查期间（2026-04-17）至生成日期（2026-05-17）之间，未观测到架构层面的重大变化。插件的 32 个制品数量、三层架构设计和脚本目录结构保持稳定。这表明 claude-ads 已进入维护期而非活跃迭代期，其核心架构决策已经固化。

从正面角度看，这种稳定性本身也是一种信号：高质量的初始设计减少了后续重构的必要性。

### 3.3 新增的可学习模式

由于架构在审查期间内未发生重大变化，当前仓库状态与快照基本一致。以下模式在审查时已存在，但值得在"当前"视角下重新强调：

- **脚本层与 NL 层的职责分离**：6 个 Python 脚本处理所有需要确定性执行的任务（截图、HTTP 请求、图像生成），NL 层的 agent 和 skill 只负责语义判断和内容生成。这种分层避免了在 prompt 中嵌入过多程序逻辑。
- **Research-sources 作为"真理来源"**：将研究文档纳入版本控制意味着插件的知识库可以像代码一样被追踪和更新，而不是依赖作者的隐性记忆。

---

## 四、校准

### 4.1 我已经在做对的

1. **交叉引用管理**：如果你的插件已经维护了有效的跨文件引用，并定期验证没有悬空链接，claude-ads 证明这一纪律在大规模（32 个制品）下同样可行。
2. **Platform skill 的模板化**：20 个 platform skill 全部 100/100 说明高质量模板 + 严格执行可以消除重复结构中的质量退化。如果你也在为多个相似场景编写 skill，这种方法值得复制。
3. **术语表维护**：如果你已经为插件建立了术语表并在多个文件中保持一致，你正在做 claude-ads 做对的最重要的事情之一。
4. **Orchestrator 路由模式**：单一入口 skill 路由到多个子 skill 的设计在可维护性和用户体验上优于让用户直接选择子 skill。

### 4.2 挑战 / 验证

1. **数量一致性是真正的盲点**：audit-google.md 和 audit-meta.md 的检查项数量矛盾在 99 分的插件中依然存在且持续了一个月。说明即使是高度专注的作者也会在"文字标注"与"表格实体"之间产生不同步。**验证动作**：检查你自己的 agent 文件中是否有任何以数字开头的描述（如"20-step process"、"15 criteria"），然后实际数一遍对应的列表项。
2. **清单（CLAUDE.md）的滞后性**：4 个创作 agent 被遗漏说明 CLAUDE.md 的更新频率落后于实际文件的增加。**验证动作**：检查你的 CLAUDE.md 或 README 中的组件列表，与实际文件数对比，确认是否有遗漏。
3. **示例多样性 vs 示例存在**：有示例（copy-writer.md 有 1 个）不等于示例充分（NLPM 认为 1 个不足以展示边界情况）。**验证动作**：检查你的 agent 文件中的示例是否覆盖了正向场景、边界场景和拒绝场景（即 Claude 应该说"不"的情况）。
4. **"applicable" 类模糊词的替换难度**：即使在高质量插件中，"applicable"和"relevant"仍然出现在条件语句里。这类词的替换需要作者真正思清楚"什么情况下适用"——这不是机械性的文字工作，而是领域知识的显式化。**验证动作**：在你的插件文件中全局搜索"applicable"、"relevant"、"appropriate"、"suitable"，对每一处问自己：我能否用一个 if 语句来替换它？

---

## 五、行动

### 5.1 自查动作

按优先级排列，适用于任何 Claude Code 插件作者：

**立即可做（5 分钟内）**

1. 在你的插件目录下运行：
   ```
   grep -rn "applicable\|relevant\|appropriate\|suitable" agents/ skills/ --include="*.md"
   ```
   记录所有命中行，评估每一处是否可以替换为具体的条件描述。

2. 运行：
   ```
   grep -rn "[0-9]\+-check\|[0-9]\+-step\|[0-9]\+ criteria\|[0-9]\+ rules" agents/ skills/ --include="*.md"
   ```
   对每一个数字标注，手动数一遍实际的列表项数，确认一致。

3. 打开你的 CLAUDE.md（或等效清单文件），对照实际的 agents/ 和 skills/ 目录内容，逐条核对是否有遗漏。

**短期任务（一周内）**

4. 检查每个 agent 文件中的 `<example>` 块数量。如果少于 2 个，补充至少一个覆盖不同场景的示例。
5. 如果你的插件有多个相似结构的 skill（如按平台分类），选出得分最高的一个作为模板，对照模板检查其余 skill 的结构完整性。

**持续维护**

6. 建立术语表文件（哪怕是一个简单的 Markdown 表格），列出你的插件中所有具有特定含义的术语，并在每次新增 skill/agent 时引用该术语表。
7. 将 `ads/research-sources/` 模式应用到自己的插件：为每个重要的设计决策（如审查清单的来源）保留一份参考文档，并将其纳入版本控制。

### 5.2 灵感 → 实施路径

**灵感一：Research-backed knowledge base**

claude-ads 将外部研究文档（claude-research.md、gemini-research.md、google-*.md）纳入插件，使审查准则有据可查。

实施路径：
1. 为你的插件创建 `references/` 或 `research-sources/` 目录
2. 将你在编写 skill/agent 时参考的外部资料（官方文档摘录、领域论文、最佳实践指南）以 Markdown 格式保存到该目录
3. 在对应的 agent 文件中用 `See: references/xxx.md` 或直接引用文件路径建立链接
4. 在 CLAUDE.md 中说明该目录的用途，使其他贡献者知道如何维护

**灵感二：Orchestrator + platform dispatch 模式**

实施路径：
1. 识别你的插件中是否存在"多个相似但有差异的执行路径"（如按平台、按语言、按受众类型区分）
2. 创建一个顶层 orchestrator skill，负责：（a）从用户输入中提取路由键；（b）根据路由键调用对应的子 skill
3. 每个子 skill 只负责自己的特化逻辑，不重复公共逻辑
4. 在 orchestrator 中替换"relevant platform skill"为显式的路由表（如"if platform is Google, load ads-google.md"）

**灵感三：Template-driven platform skill 批量生产**

claude-ads 的 20 个 platform skill 全部 100/100，背后是高质量模板的严格执行。

实施路径：
1. 为你的重复结构制品（如多个平台的 skill）先写一个"原型 skill"
2. 对原型 skill 运行 NLPM 评分，直到达到 100/100
3. 将原型 skill 作为模板，用占位符替换平台特定内容
4. 批量生成其余 skill，逐一核对是否有平台特定内容需要替换，是否有通用内容被错误保留
5. 运行 NLPM 批量评分，确认无退化

**灵感四：数量冻结机制**

audit-google.md 和 audit-meta.md 的数量矛盾说明"文字描述中的数字"与"实际内容"之间的同步是一个持续风险。

实施路径：
1. 在你的插件 CONTRIBUTING.md 或开发规范中加入规则："任何包含数字的描述（如'X-step process'）必须与实际步骤数精确匹配"
2. 在 CI 或 pre-commit hook 中加入检查脚本：提取文件中的数字描述，与实际列表项数对比，不一致则报错
3. 或者采用更简单的策略：在正式文档中避免使用具体数字，改为"multi-step process"等不带数字的描述，只在表格中保留权威数字
