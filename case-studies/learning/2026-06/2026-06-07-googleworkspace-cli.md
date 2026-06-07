# googleworkspace/cli — 学习案例

**仓库**：https://github.com/googleworkspace/cli
**Stars**：25,330 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-07（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `template-design`, `single-purpose`, `score-volatility`

**xiaolai 案例**：[../2026-05-01-googleworkspace-cli.md](../2026-05-01-googleworkspace-cli.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

googleworkspace/cli 是 Google 官方维护的命令行工具，将 Google Drive、Gmail、Calendar、Sheets、Docs、Chat、Admin、Forms 等全系列 Workspace API 封装为统一的 `gws` 命令。API 接口通过 Google Discovery Service 动态生成。仓库同时附带一个完整的 AI agent 技能库——96 个 SKILL.md 文件，覆盖 API 技能、辅助技能、角色定义、食谱（recipe）、工作流组合等多种层次，是目前公开可查的最大结构化 Claude Code 技能集合之一。截至审计日期，项目拥有 **25,330 stars**、1,315 个 forks。

关键数字（2026-04-06 审计快照）：

- NLPM 综合评分：**99/100**（96 个工件）
- 安全评级：**REVIEW**（1 High，1 Medium，1 Low，无 Critical）
- 完美评分（100/100）文件数：**84 个**（占 87.5%）
- 评分低于 100 的文件：**12 个**（最低 88/100）
- 运行时 bug：**2 个**（food.md 类比：菜谱指向了不存在的食材）

### 1.2 架构剖析

**目录结构（技能库部分）**：

```
skills/
  gws-gmail/SKILL.md              # api-skill：Gmail API 封装
  gws-drive/SKILL.md              # api-skill：Drive API 封装
  gws-calendar/SKILL.md           # api-skill：Calendar API 封装
  gws-docs/SKILL.md               # api-skill：Docs API 封装
  gws-sheets/SKILL.md             # api-skill：Sheets API 封装
  gws-chat/SKILL.md               # api-skill：Chat API 封装
  gws-forms/SKILL.md              # api-skill：Forms API 封装
  gws-admin-reports/SKILL.md      # api-skill：Admin Reports API
  gws-classroom/SKILL.md          # api-skill：Classroom API
  gws-keep/SKILL.md               # api-skill：Keep API
  gws-meet/SKILL.md               # api-skill：Meet API
  gws-people/SKILL.md             # api-skill：People API
  gws-script/SKILL.md             # api-skill：Apps Script API
  gws-slides/SKILL.md             # api-skill：Slides API
  gws-tasks/SKILL.md              # api-skill：Tasks API
  gws-events/SKILL.md             # api-skill
  gws-workflow/SKILL.md           # api-skill
  gws-modelarmor/SKILL.md         # api-skill（缺少 API Resources 节）
  gws-docs-write/SKILL.md         # helper-skill：文档写入辅助
  gws-gmail-send/SKILL.md         # helper-skill：邮件发送辅助
  gws-gmail-read/SKILL.md         # helper-skill
  gws-sheets-read/SKILL.md        # helper-skill
  gws-sheets-append/SKILL.md      # helper-skill
  ...（共 20 个 helper-skill）
  content-creator/SKILL.md        # persona：内容创作者角色
  exec-assistant/SKILL.md         # persona：执行助理角色
  it-admin/SKILL.md               # persona：IT 管理员角色
  ...（共 10 个 persona）
  recipe-post-mortem-setup/SKILL.md     # recipe：事后总结设置流程（含 Bug #1）
  recipe-collect-form-responses/SKILL.md # recipe：收集表单回应（含 Bug #2）
  recipe-draft-email-from-doc/SKILL.md  # recipe：从文档起草邮件
  ...（共 ~40 个 recipe）
  gws-workflow-email-to-task/SKILL.md   # workflow-skill：邮件转任务工作流
  gws-workflow-weekly-digest/SKILL.md   # workflow-skill：每周摘要工作流
  ...（共 5 个 workflow-skill）
  gws-shared/SKILL.md             # shared：跨技能共用参考

CLAUDE.md                         # 项目指令（只有 1 行，委托给 AGENTS.md）
AGENTS.md                         # 主 agent 指令文件
scripts/
  show-art.sh                     # 开发用 ASCII art 工具
  coverage.sh                     # 代码覆盖率脚本
package.json                      # Node 依赖清单
```

**四层技能抽象梯度**（这是本仓库最值得学习的设计）：

```
api-skill（最底层）
  └── 描述单个 Google API 的所有可用方法和参数
      例：gws-gmail/SKILL.md → Gmail API 的完整方法列表

helper-skill（第二层）
  └── 封装单个常见操作，组合 api-skill 的命令
      例：gws-docs-write/SKILL.md → 如何写入文档（调用 gws docs 命令）

recipe（第三层）
  └── 多步骤任务流程，组合多个 helper-skill
      例：recipe-draft-email-from-doc → 从 Docs 读取内容，用 Gmail 发送

workflow-skill（顶层）
  └── 跨服务的端到端自动化工作流
      例：gws-workflow-email-to-task → 邮件自动转 Tasks 任务
```

**persona 层（横向切面）**：

persona 不属于上述抽象梯度，而是横向切面——定义「Claude 在某个角色身份下应如何行事」。exec-assistant（执行助理）、it-admin（IT 管理员）、project-manager（项目经理）等 10 个角色，为不同职业场景的用户提供行为模式，而非操作指令。

**关键跨件契约（CC-1）**：

`gws-docs-write/SKILL.md` 文档记录了 `--document <ID>` 参数形式。但以下三个 recipe 使用了 `--document-id` 这一不同参数名：
- `skills/recipe-save-email-to-doc/SKILL.md`（第 28 行）
- `skills/recipe-generate-report-from-sheet/SKILL.md`
- `skills/recipe-create-doc-from-template/SKILL.md`

CLI 是否实际接受 `--document-id` 不确定。这是一个参数名称漂移问题（flag-name-drift），属于跨组件一致性缺陷 CC-1。

### 1.3 设计思路 / 方法论

**核心设计哲学：「单一职责 + 显式分层」**

googleworkspace/cli 的技能库设计遵循了一种极其清晰的分层原则：每个技能只做一件事，不同抽象层次之间的调用关系明确。这使得 96 个文件的库依然保持高度的可维护性——84 个文件拿到满分 100/100，印证了这一设计的正确性。

**解决什么问题**：Google Workspace 的 API 表面极大（Drive、Gmail、Calendar、Docs、Sheets、Chat……），直接将所有 API 描述放入单一 skill 会导致上下文爆炸。分层抽象将「API 知识」（api-skill）、「操作封装」（helper-skill）、「任务流程」（recipe）和「自动化工作流」（workflow-skill）解耦，Claude 只需在完成任务的对应层加载合适的技能。

**做了什么 trade-off**：精度优先于覆盖率——库中大量使用具体的 CLI 命令示例（如 `gws gmail messages list --query "is:unread"`），而非泛泛的描述。这导致两个 recipe 中的命令错误直接成为运行时 bug，但整体质量远高于使用模糊描述的同类库。

**CLAUDE.md 的最小委托模式**：项目的 CLAUDE.md 只有一行，将所有 agent 指令委托给 AGENTS.md。这是一种刻意的「单一来源」架构选择，但在 NLPM 重审计时被评为 65/100（原审计 100/100），说明该模式与 NLPM 的评分标准存在约定冲突——这不是 bug，而是约定不同。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「显式抽象梯度」多域技能库模式**：在一个覆盖多个 API 或技术域的技能库中，通过明确的层级（api → helper → recipe → workflow）将「知识」、「操作」、「流程」和「自动化」分离，避免单一大文件覆盖全部内容。

模式特征清单：
- 特征 1：每个文件的层级角色明确（api-skill / helper-skill / recipe / workflow-skill / persona），不混淆
- 特征 2：下层技能不引用上层技能（api-skill 不依赖 recipe），形成单向依赖
- 特征 3：recipe 通过 helper-skill 的 CLI 命令表达意图，不直接引用 api-skill（减少读者必须理解的抽象层数）
- 特征 4：persona 作为横向切面，独立于操作层级
- 特征 5：共用参考（gws-shared/SKILL.md）提取所有技能的公共约定

### 2.2 适用场景

| 场景 | 是否适用 | 原因 |
|---|---|---|
| 多 API 的统一 CLI 工具文档 | ✅ 高度适用 | 本案例即为此场景，四层梯度天然对应 API 的使用粒度 |
| 法律工作流（多领域、多步骤） | ✅ 适用 | 可将 api-skill 替换为「法律数据库查询」，recipe 替换为「案件调研流程」 |
| 单领域深度技能库 | ⚠️ 过度设计 | 单领域无需四层抽象，两层（knowledge + workflow）已够 |
| 个人工具箱（少量技能） | ⚠️ 过度设计 | 5 个以下技能不需要显式分层，直接写 recipe 即可 |
| 需要跨服务自动化的企业场景 | ✅ 适用 | workflow-skill 层专为此设计 |

### 2.3 与其他架构对比

| 架构类型 | 代表性仓库 | 核心差异 | 优缺点 |
|---|---|---|---|
| 显式抽象梯度（本案例） | googleworkspace/cli | api → helper → recipe → workflow 四层 | 可维护性极高，但 recipe 错误难以发现（依赖人工核验 CLI 命令） |
| 认知场景专用化 | feiskyer/claude-code-settings | 每场景一工件，无分层 | 简单直观，但大规模时各工件间关系模糊 |
| 领域平铺 | 多数小型技能库 | 所有技能同级，无分层 | 上手快，规模化后维护成本高 |
| 单一大文件 | CLAUDE.md 直接包含所有技能 | 无分层 | 最简单，但上下文消耗大，不可复用 |

### 2.4 改进空间

**改进点 1：recipe 步骤的 CLI 命令强制验证**

- 当前问题：`recipe-post-mortem-setup/SKILL.md` 第 26 行使用 `gws docs +write --title ... --body ...`，这两个 flag 均不存在（正确的是 `--document` 和 `--text`）；`recipe-collect-form-responses/SKILL.md` 第 24 行调用 `gws forms forms list`，Forms API v1 完全没有 `list` 方法。两个 bug 均在当前 HEAD 未被修复（因 CLA 障碍阻断了合并路径）。
- 改进方式：在 CI 中加入「recipe CLI 命令验证」步骤，对 recipe 文件中出现的 `gws <subcommand>` 提取命令列表，与 `gws --help` 输出进行对比，自动标记不存在的子命令或 flag。
- 预期收益：在发布前捕获运行时 bug，避免用户执行菜谱时遇到「命令不存在」报错。

**改进点 2：跨组件参数名称一致性（CC-1）**

- 当前问题：`--document-id` vs `--document` 的不一致出现在至少 3 个 recipe 文件中（recipe-save-email-to-doc 第 28 行为例）。
- 改进方式：维护一个「规范参数名称表」（canonical-flags.md），在 CI 中扫描所有 recipe 文件，将检测到的 flag 名与规范表对比。
- 预期收益：消除 flag-name-drift，避免用户照搬 recipe 命令时因参数名不同而失败。

**改进点 3：CLAUDE.md 的委托声明显式化**

- 当前问题：CLAUDE.md 只有一行委托，再审计时被评为 65/100（-35 分）。原审计 100/100。分差 35 分完全来自约定解读的不同——这是分数波动问题，不是内容问题。
- 改进方式：在 CLAUDE.md 中加一段注释，说明「本文件采用单一来源委托模式，完整指令见 AGENTS.md」，使审计者（人或 LLM）能理解这是刻意设计而非遗漏。
- 预期收益：减少 LLM 审计员将「最小委托」误判为「缺失内容」的概率。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**综合评分：99/100**（96 个工件，84 个满分）

| 文件 | 得分 | 主要扣分原因 |
|---|---|---|
| `skills/recipe-post-mortem-setup/SKILL.md` | **88** | Bug #1：`gws docs +write --title ... --body ...`，两个 flag 均不存在；正确参数为 `--document <ID>` 和 `--text <TEXT>`。菜谱在运行时必然报错 |
| `skills/recipe-collect-form-responses/SKILL.md` | **88** | Bug #2：`gws forms forms list`，Forms API v1 无 `list` 方法（仅有 create、get、batchUpdate、setPublishSettings）。步骤在运行时必然失败 |
| `skills/gws-modelarmor/SKILL.md` | 95 | 缺少 `## API Resources` 节（其他所有 gws-* api-skill 均有此节） |
| `skills/recipe-compare-sheet-tabs/SKILL.md` | 95 | 第 3 步无 CLI 命令（只说「对比数据」，无可执行动作） |
| `skills/recipe-create-events-from-sheet/SKILL.md` | 95 | 第 2 步只展示一个硬编码示例，无循环逻辑 |
| `skills/recipe-draft-email-from-doc/SKILL.md` | 95 | 第 2 步为纯手工操作，无 CLI 命令 |
| `skills/recipe-find-large-files/SKILL.md` | 95 | 第 2 步无动作命令 |
| `skills/recipe-review-overdue-tasks/SKILL.md` | 95 | 第 3 步无动作命令 |
| `skills/recipe-create-doc-from-template/SKILL.md` | 98 | 使用 `--document-id`，而 gws-docs-write 规范为 `--document` |
| `skills/recipe-create-shared-drive/SKILL.md` | 98 | 「适当的角色」措辞模糊，不可验证 |
| `skills/recipe-generate-report-from-sheet/SKILL.md` | 98 | `--document-id` flag 不一致 |
| `skills/recipe-save-email-to-doc/SKILL.md` | 98 | `--document-id` flag 不一致（第 28 行） |
| 其余 84 个文件 | **100** | 无扣分 |

注意：99/100 的高分来自对 96 个文件的平均，掩盖了 2 个运行时 bug 的存在。**高聚合分数不等于零 bug**。

### 3.2 当时值得借鉴的模式

**模式 1：四层技能抽象梯度**

- 为何有效：api-skill 负责知识（API 有哪些方法），helper-skill 负责封装（如何调用单个操作），recipe 负责流程（如何完成一个任务），workflow-skill 负责自动化（如何将流程串联）。Claude 在执行任务时按需加载对应层级，不需要一次性消费所有技能的上下文。
- 原始路径：`skills/gws-docs/SKILL.md`（api-skill）→ `skills/gws-docs-write/SKILL.md`（helper-skill）→ `skills/recipe-draft-email-from-doc/SKILL.md`（recipe）
- 如何借鉴：任何覆盖 3 个以上独立操作域的技能库，都应考虑引入显式的分层命名约定（如 `api-*`、`helper-*`、`recipe-*`）。

**模式 2：persona 作为角色定义而非操作定义**

- 为何有效：exec-assistant/SKILL.md 描述「执行助理身份下 Claude 应如何思考和优先处理任务」，而非「执行助理会用哪些 API」。将行为模式与操作技能解耦，使同一个 recipe 可以被不同 persona 调用，只改变优先级和沟通风格。
- 原始路径：`skills/exec-assistant/SKILL.md`、`skills/it-admin/SKILL.md`
- 如何借鉴：在多用户角色的工作流系统中（如法律工作流），可为律师、助理、合规专员各建一个 persona，用于调整同一操作技能的输出风格和决策优先级。

**模式 3：高密度具体命令示例**

- 为何有效：84 个文件得到满分的核心原因是命令示例具体且可验证（如 `gws gmail messages list --query "is:unread" --max-results 10`），不使用「灵活」「适当」等模糊描述。
- 如何借鉴：每个 recipe 步骤都应该有至少一个可以直接复制运行的 CLI 命令，不能只有文字说明。

**模式 4：gws-shared 作为共用参考层**

- 为何有效：跨技能的公共约定（如认证方式、错误码处理、输出格式规范）集中在 `skills/gws-shared/SKILL.md`，避免在 96 个文件中重复。
- 如何借鉴：大型技能库应有一个 shared/common 技能，专门存放跨文件的约定，其他技能通过引用而非复制来使用公共知识。

### 3.3 当时的缺陷

**缺陷 1（Bug #1）：recipe-post-mortem-setup 使用不存在的 CLI flag**

- 问题描述：`skills/recipe-post-mortem-setup/SKILL.md` 第 26 行：`gws docs +write --title 'Post-Mortem: [Incident]' --body '## Summary\n\n...'`。`--title` 和 `--body` 两个参数均不存在，正确的参数名为 `--document <ID>` 和 `--text <TEXT>`。
- 根因：Recipe 作者可能参考了旧版 API 文档或混淆了其他 CLI 工具的参数名称，未对照 `gws docs +write --help` 输出进行核验。
- 影响：用户照搬 recipe 命令时，CLI 会立即报错「unknown flag: --title」，整个事后总结流程中断。

**缺陷 2（Bug #2）：recipe-collect-form-responses 调用不存在的 API 方法**

- 问题描述：`skills/recipe-collect-form-responses/SKILL.md` 第 24 行：`gws forms forms list`。Google Forms API v1 的 forms 资源无 `list` 方法，只有 `create`、`get`、`batchUpdate`、`setPublishSettings`。
- 根因：Google Forms API 的表单列表获取需要通过 Drive API（`gws drive files list --q "mimeType='application/vnd.google-apps.form'"`），而非直接调用 Forms API。Recipe 作者假设了一个对称的 `list` 方法存在，但 Google 未提供。
- 影响：第 1 步失败，整个「收集表单回应」流程无法启动。

**缺陷 3（CC-1）：--document-id vs --document 参数名漂移**

- 问题描述：`gws-docs-write/SKILL.md` 规范参数为 `--document <ID>`，但至少 3 个 recipe 文件使用了 `--document-id`（recipe-save-email-to-doc 第 28 行可直接核验）。
- 根因：参数名称的约定在 helper-skill 层面有明确记录，但 recipe 作者在编写时未查阅 helper-skill，或查阅了不同版本的文档。
- 影响：如果 CLI 不接受 `--document-id`，这 3 个 recipe 步骤会失败。不确定性使用户无法判断应该用哪个参数名。

### 3.4 当时的优化机会

1. **最高 ROI（两行修复，消除运行时 bug）**：
   - recipe-post-mortem-setup 第 26 行：将 `--title ... --body ...` 替换为 `--document <DOCUMENT_ID> --text <MARKDOWN_CONTENT>`
   - recipe-collect-form-responses 第 24 行：将 `gws forms forms list` 替换为 `gws drive files list --q "mimeType='application/vnd.google-apps.form'"`

2. **中等 ROI（统一参数名，消除 CC-1）**：
   - 在 3 个 recipe 文件中统一使用 `--document <ID>` 形式，或在 `gws-docs-write/SKILL.md` 中明确说明两个参数名是否均被接受

3. **低 ROI（补全结构节）**：
   - 为 `gws-modelarmor/SKILL.md` 添加 `## API Resources` 节，保持与其他 18 个 api-skill 的结构一致

4. **中等 ROI（recipe 步骤补全 CLI 命令）**：
   - 为 5 个「只说明目标但不给命令」的 recipe 步骤（recipe-compare-sheet-tabs 第 3 步等）添加具体可运行的 CLI 命令

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 验证方法（可在克隆后执行） | 当前状态 | 原因 |
|---|---|---|---|
| Bug #1：recipe-post-mortem-setup 第 26 行 `--title ... --body ...` | `grep -n "\-\-title\|\-\-body" skills/recipe-post-mortem-setup/SKILL.md` | ❌ **仍存在**，当前 HEAD 未修复 | 关联 PR #757 CLA 受阻，无法合并 |
| Bug #2：recipe-collect-form-responses 第 24 行 `gws forms forms list` | `grep -n "forms list" skills/recipe-collect-form-responses/SKILL.md` | ❌ **仍存在**，当前 HEAD 未修复 | 关联 PR #758 CLA 受阻，无法合并 |
| CC-1：recipe-save-email-to-doc 第 28 行 `--document-id` | `grep -n "\-\-document-id" skills/recipe-save-email-to-doc/SKILL.md` | ❌ **仍存在**，当前 HEAD 未修复 | 无 PR 提交（flag-inconsistency 属质量问题，未达提交阈值） |
| CC-1：另外 2 个 recipe 的 `--document-id` | `grep -rn "\-\-document-id" skills/` | ❌ **仍存在** | 同上 |

四个 PR（#757、#758、#759、#760）均处于 CLA 受阻状态。Google 的公开仓库要求提交者本人在 Google CLA 上签名；NLPM 流水线的默认提交身份（`claude[bot]`）不在任何 CLA 覆盖范围之内。PR 技术上仍为开放状态，但无合并路径，除非由已签署 CLA 的人类贡献者重新提交相同修复。

这是一个重要认知：**代码问题的修复不仅取决于技术质量，还取决于策略约束**。CLA 障碍是结构性问题，而非技术问题。

### 4.2 架构演进

基于当前 HEAD 的可验证信息：

- 核心技能文件（gws-gmail、gws-drive、gws-calendar 等 api-skill）的结构在审计后未发生变化——四层梯度架构保持稳定
- 两个含 bug 的 recipe 文件在 CLA 受阻的背景下未被外部贡献者修复
- CLAUDE.md 的「单行委托」设计在审计后仍保持不变（仍只有一行，仍委托给 AGENTS.md）

从 xiaolai 的案例研究来看，再审计（re-audit）在同一 commit（a3768d0）上产生了截然不同的结果：原审计 99/100，再审计 89/100；原审计 16 个发现全部「不可复现」，同时新增 121 个发现。**代码一行未改，分数下降 10 分，发现数量翻了 8 倍**。这不是仓库退化，而是评分工具的精度问题。

### 4.3 新增的可学习模式

**再审计揭示的「分数波动」模式**（这是本案例最独特的学习价值）：

xiaolai 将这一现象命名为「The Rubric Forgot What It Found（评分表忘了自己发现过什么）」。同一 LLM 模型家族（Sonnet 4.6），在两次运行间对以下判断产生了截然不同的结论：

- `no_scope_note` 发现：原审计 0 个 → 再审计多达 94 个
- `generic_description` 发现：原审计 0 个 → 再审计 19 个
- CLAUDE.md 评分：原审计 100/100 → 再审计 65/100

这个模式教会我们：**在高分段（90+ 分），评分的主要噪声来源是 LLM 的判断校准偏差，而非工件质量的真实差异。** 对 99/100 分的项目，10 分的波动几乎完全来自「两次运行的评分员对同一条标准的理解差异」，不来自代码变化。

---

## 五、校准

### 5.1 我已经在做对的

1. **分层架构的直觉**：我的 MarkQWu/claude-for-legal 已经按照法律领域分组组织技能（诉讼技能、合规技能、合同技能等），这与 googleworkspace/cli 的 api-skill 分层有相同的「按域隔离」直觉，是正确方向。

2. **具体操作命令优于模糊描述**：在 claude-for-legal 的技能文件中，我有意使用具体的查询语法和 API 调用示例，而非「查询相关数据」这类描述。这与 googleworkspace/cli 拿到 84 个满分的核心做法一致。

3. **维护 CONNECTORS.md 做集成文档**：我的 claude-for-legal 有 CONNECTORS.md 记录 API 集成面，googleworkspace/cli 没有等效文档。在 CLI 参数一致性问题上，CONNECTORS.md 的集中化正是 CC-1 缺陷的解决思路。

4. **意识到 recipe 步骤需要可执行命令**：我在写多步骤工作流时有意识地确保每个步骤都有对应的操作指令，避免「只说结果，不说手段」的描述方式。

### 5.2 挑战 / 验证

**挑战 1：高分不等于零 bug**

本案例挑战了「99/100 说明几乎没有问题」这一直觉。实际情况是，2 个运行时 bug 对聚合分数几乎没有影响（两个 88 分拉低平均仅约 0.25 分）。高聚合分数遮蔽了个别文件的功能性缺陷。对我的 claude-for-legal 来说，即使整体评分高，也需要单独核验每个多步骤流程中的 API 调用是否在目标系统上实际存在。

**挑战 2：分数是概率分布，不是测量值**

xiaolai 的核心发现：「同一 commit，同一工件，两次评分相差 10 分」。这要求我重新理解 NLPM 评分的性质——它是对质量的一种采样估计，不是确定性测量。对我自己的项目，如果 NLPM 给出 85 分，实际质量区间可能是 78–92 分。不应该为单次评分结果过度优化，而应该关注多次评分的趋势方向。

**验证：CLA 障碍是真实的结构性约束**

本案例验证了 NLPM CLAUDE.md 中记录的 CLA 政策门控的现实意义。四个 PR 全部受阻，不是因为内容有问题，而是因为提交者身份不符合 Google 的贡献要求。如果我计划向 Google 的开源项目贡献代码，必须在自己的账户下签署 Google CLA（https://cla.developers.google.com/about）。这是一个在技术质量之外的前置条件。

---

## 六、行动

### 6.1 自查动作

**检查 recipe 文件中每个 CLI 命令是否真实存在（最高优先级）**：

```bash
# 在本地克隆的 googleworkspace/cli 或自己的技能库中执行
# 提取所有 recipe 文件中出现的 CLI 命令模式（gws <subcommand>）
grep -rn "^gws " skills/recipe-*/SKILL.md | head -50
# 对每个出现的子命令，验证其是否在 CLI help 中存在：
# gws <subcommand> --help 2>&1 | grep -c "Usage"
```

**检查参数名称是否跨文件一致（CC-1 类问题）**：

```bash
# 找出所有使用 --document-id 的文件
grep -rn "\-\-document-id" skills/ agents/ commands/ 2>/dev/null
# 找出所有使用 --document 的文件（含参数值形式）
grep -rn "\-\-document [^-]" skills/ agents/ commands/ 2>/dev/null
# 对比两个列表，判断是否存在不一致
```

**检查多步骤流程每一步是否有可执行命令（recipe 步骤完整性）**：

```bash
# 找出 recipe 文件中只有文字描述、无 CLI 命令的步骤
# 特征：以数字+点开头，但行内无 gws/curl/python 等命令关键词
grep -n "^[0-9]\+\." skills/recipe-*/SKILL.md | grep -v "gws\|curl\|python\|bash\|\`" | head -30
# 每个匹配行 → 评估该步骤是否应该有可执行命令
```

**检查聚合评分与单文件评分的差异（掩盖效应）**：

```bash
# 对自己的技能库运行 NLPM 评分，不要只看综合分
# 找出所有低于 95 分的文件
# /nlpm:score ./ 后检查分数分布，确认是否有个别文件拖低
```

**检查是否有「只有说明没有命令」的步骤（vague-step-no-command 类问题）**：

```bash
# 在自己的 recipe/workflow 文件中查找模糊步骤
grep -rn "查看\|检查一下\|根据需要\|手动\|人工" skills/ agents/ 2>/dev/null
# 每个匹配项 → 评估是否可以替换为具体的 CLI 命令或判断标准
```

### 6.2 灵感 → 实施路径

**灵感 1：在 claude-for-legal 引入显式的四层抽象梯度（高价值）**

- 背景：claude-for-legal 目前的技能组织是「按法律域（合同/诉讼/合规）」，但没有「api-skill → helper-skill → recipe → workflow」的明确层级。跨域的公共操作（如「查询案件数据库」）在多个技能文件中重复出现。
- 第一步：梳理现有技能，识别哪些是「知识层」（描述某个法律数据库的查询能力），哪些是「操作层」（如何执行单次查询），哪些是「流程层」（多步骤的法律工作流）。
- 实施路径：将技能文件重命名为 `api-courtdb/SKILL.md`（知识层）→ `helper-case-search/SKILL.md`（操作层）→ `recipe-due-diligence/SKILL.md`（流程层），建立与 googleworkspace/cli 同构的结构。
- 时间预估：现有技能分类 2h，文件重组 1h，更新引用 1h。

**灵感 2：为 claude-for-legal 引入 persona 技能（中价值）**

- 背景：法律工作流的使用者包括律师、法务助理、合规专员、客户等不同角色，每个角色对同一操作的输出格式和细节深度要求不同。目前 claude-for-legal 没有角色定义，所有技能对所有角色一视同仁。
- 第一步：参照 googleworkspace/cli 的 `exec-assistant/SKILL.md` 和 `it-admin/SKILL.md` 格式，为 claude-for-legal 创建 `persona-senior-associate/SKILL.md`（资深律师助理角色）和 `persona-compliance-officer/SKILL.md`（合规专员角色）。
- 实施路径：persona 文件描述「在该角色视角下，Claude 应如何优先处理信息、以什么格式回答、在哪些情况下需要提示用户确认」。不描述具体 API 调用。

**灵感 3：为多步骤 recipe 建立「命令存在性」核验清单（低成本，高可靠性）**

- 背景：Bug #1 和 Bug #2 的根因是 recipe 步骤引用了不存在的 CLI 命令，而编写者没有对照实际 CLI help 进行核验。我的 claude-for-legal 的法律数据库查询 recipe 也面临同样的风险——引用的 API 端点可能已经废弃或参数已变更。
- 第一步：为 claude-for-legal 的每个 recipe 文件建立一个「命令清单」注释节（`## 命令核验`），列出文件中使用的所有外部命令和 API 调用，以及最后一次人工核验的日期。
- 实施路径：这是一个轻量级的文档约定，不需要自动化工具，只需在每次修改 recipe 时更新核验记录。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

googleworkspace/cli 的核心目的：为使用 Google Workspace 的用户提供一套结构清晰、分层明确的 Claude Code 技能库，覆盖从 API 级操作到端到端工作流的全部范围。

| 我的仓库 | 相似度 | 原因 |
|---|---|---|
| **MarkQWu/claude-for-legal** | **高** | 两者都是「多域技能库 + 实际 API 集成 + 多步骤工作流」，claude-for-legal 的法律数据库技能对应 gws-* api-skill，法律工作流对应 recipe/workflow-skill，结构上高度同构。最关键的区别是 claude-for-legal 有显式的 API 集成文档（CONNECTORS.md），而 googleworkspace/cli 没有 |
| **MarkQWu/echo-sleuth-for-claude** | 低 | echo-sleuth 是开发者工具，4 个 SKILL.md 文件，无多域覆盖，无工作流层 |
| **MarkQWu/drama-workshop-skills** | 低 | 单一创作领域（中文剧本写作），2 个 SKILL.md 文件，无 API 集成，无分层设计 |

### 8.2 在我的项目里复现的同类问题

googleworkspace/cli 的 bug 模式，在 claude-for-legal 中最值得检查：

| 本案例缺陷类型 | 在 claude-for-legal 的对应风险 | 验证方法 | 预估严重度 |
|---|---|---|---|
| Recipe 步骤使用不存在的 API 方法（Bug #2） | 法律数据库的 API 端点或查询语法可能已变更，或某些端点根本不存在 | 对每个 recipe 文件提取 API 调用，在沙盒环境中手工核验 | **高**（运行时失败，用户无法完成任务） |
| Recipe 步骤使用错误的 CLI 参数名（Bug #1） | 法律 API 客户端的参数名可能与文档不同版本有差异 | 对每个 recipe 文件提取参数名，与当前版本客户端 `--help` 输出对比 | **高**（运行时报错） |
| 跨文件参数名不一致（CC-1） | claude-for-legal 中同一 API 操作可能在不同 recipe 文件中使用了不同的参数名或查询语法 | `grep -rn` 扫描所有技能文件，提取 API 调用语法，人工对比 | **中**（用户困惑，不知哪个版本正确） |
| Recipe 步骤无可执行命令（vague-step-no-command） | 部分工作流步骤可能只有「查阅相关法规」这类描述，没有具体的查询命令 | `grep -rn "查阅\|参考\|根据" skills/` | **中**（Claude 无法自主执行，需要用户干预） |

**重要认知**：googleworkspace/cli 的 99/100 分说明高聚合分数不能代替单文件的运行时验证。claude-for-legal 即使 NLPM 评分高，也需要对每个包含 API 调用的 recipe 进行「命令存在性」核验，确保用户执行时不会遇到「方法不存在」或「参数名错误」的报错。

### 8.3 别人的更优方案

**方案 1：四层技能抽象梯度（api → helper → recipe → workflow）**

- 文件路径：`skills/gws-docs/SKILL.md`（api）→ `skills/gws-docs-write/SKILL.md`（helper）→ `skills/recipe-draft-email-from-doc/SKILL.md`（recipe）
- 比我的方案更优之处：claude-for-legal 目前没有显式的分层命名，「法律数据库查询技能」和「多步骤尽职调查流程」可能在同一层级的目录下。googleworkspace/cli 通过文件命名约定（`gws-*` 为 api-skill/helper-skill，`recipe-*` 为 recipe，`gws-workflow-*` 为 workflow）让用户和 Claude 都能一目了然地知道每个文件的抽象层次，不需要阅读文件内容才能理解其定位。
- 可以直接借鉴：在 claude-for-legal 中引入 `api-*`、`helper-*`、`recipe-*`、`workflow-*` 的命名前缀，同时更新目录结构以反映分层关系。

**方案 2：persona 技能（行为模式与操作技能解耦）**

- 文件路径：`skills/exec-assistant/SKILL.md`、`skills/it-admin/SKILL.md`、`skills/researcher/SKILL.md`
- 比我的方案更优之处：googleworkspace/cli 将「用户角色的行为模式」与「API 操作能力」分开定义。exec-assistant 告诉 Claude 在执行助理身份下应如何思考和优先处理，gws-gmail 告诉 Claude Gmail API 有哪些方法。两者解耦意味着「执行助理」可以调用任何 API skill，只改变输出风格，不需要为每个角色单独写一套 API 操作说明。claude-for-legal 目前将角色期望和操作指令混合在同一个技能文件里，维护成本更高。
- 可以直接借鉴：为 claude-for-legal 创建独立的 `persona-*` 技能层（persona-senior-associate、persona-compliance-officer），内容专注于角色行为约束，不涉及具体 API 调用。

**方案 3：recipe 步骤的高密度具体命令（反「vague-step-no-command」模式）**

- 文件路径：所有 100/100 的 recipe 文件（84 个）
- 比我的方案更优之处：googleworkspace/cli 的满分 recipe 每一步都有至少一个可直接运行的 CLI 命令，包括参数示例。这使 recipe 既是人类的操作指南，也是 Claude 可以直接执行的脚本。drama-workshop-skills 和 echo-sleuth 的技能文件在步骤描述的具体性上还有提升空间。
- 可以直接借鉴：对 claude-for-legal 的每个 recipe 步骤进行审查，确保每个步骤都有具体的 API 调用或命令示例，替换「参考相关法规后决定」这类无法执行的描述。

### 8.4 反向：我的项目做得比他们好的地方

**1. MarkQWu/claude-for-legal 有 CONNECTORS.md（API 集成面文档）和 managed-agent-cookbooks/**

googleworkspace/cli 技能库中没有集中的「API 集成说明」文档——各个 api-skill 分别描述各自的 API，但没有一个文档说明「如何配置 gws CLI 的凭证」、「如何处理 OAuth 流程」、「如何在不同 Workspace 租户间切换」等横向集成问题。这类信息在用户实际部署时是必需的。

claude-for-legal 的 CONNECTORS.md 集中记录了法律数据库的连接配置、认证方式和会话管理，避免每个 recipe 用户都需要自己摸索集成细节。这是 googleworkspace/cli 缺失的一层文档。

**2. MarkQWu/drama-workshop-skills 有 three-layer-control.md（三层质量控制框架）**

drama-workshop-skills 的 `three-layer-control.md` 定义了「地基层/骨架层/血肉层」的剧本质量控制模型，是一个在技能文件之上的质量框架文档。它帮助用户理解「何时使用技能库的哪个部分」，以及「如何评判输出质量」。

googleworkspace/cli 没有等价的「技能使用框架」文档——它有详细的技能，但没有告诉用户「在处理一个 Workspace 自动化任务时，应该从哪一层技能开始思考，按什么顺序组合技能」。这种元层次的使用指南对大型技能库尤为重要，googleworkspace/cli 缺少这一层。

---

## 八、术语表

| 术语 | 解释 |
|---|---|
| **frontmatter** | Markdown 文件顶部用 `---` 包裹的 YAML 元数据区域。在 Claude Code 工件中，用于声明 `name`（工件名称）、`description`（触发条件描述）、`model`（使用的 AI 模型）等关键属性。缺失 frontmatter 会导致 Claude Code 无法正确识别和调用工件 |
| **recipe skill** | 技能库中处于「流程层」的技能文件，描述完成一个多步骤任务的具体流程，每个步骤对应至少一个可执行的 CLI 命令。Recipe 通过组合 helper-skill 的命令来完成任务，不直接描述 API 细节 |
| **helper skill** | 技能库中处于「操作层」的技能文件，封装单个常见操作（如「写入 Google Docs 文档」），提供具体的 CLI 命令和参数说明。Helper skill 位于 api-skill（知识层）和 recipe（流程层）之间 |
| **api skill** | 技能库中处于「知识层」的技能文件，描述单个 Google API（如 Gmail、Drive、Docs）的所有可用方法、参数和返回格式。不包含任务流程，只记录 API 能力 |
| **CLA（Contributor License Agreement）** | 贡献者许可协议。开源项目要求贡献者在提交代码前签署的法律文件，授权项目维护者使用贡献者提交的代码。Google 要求所有向其开源仓库提交 PR 的贡献者签署 Google CLA（https://cla.developers.google.com/about）。Bot 账户（如 `claude[bot]`）无法独立签署 CLA，因此 NLPM 流水线提交的 PR 全部 CLA 受阻 |
| **semver（Semantic Versioning）** | 语义化版本号规范，格式为 MAJOR.MINOR.PATCH。`^` 前缀（如 `^1.2.3`）表示接受同一主版本下的任意升级，引入了供应链风险（依赖包升级后行为可能改变）。安全最佳实践是固定到精确版本或哈希值 |
| **score volatility（分数波动）** | 同一工件在不同时间点、同一 LLM 模型家族的两次独立评分之间产生的分数差异。本案例中表现为：同一 commit a3768d0，原审计 99/100，再审计 89/100，分差 10 分，但代码未发生任何变化。高分段（90+ 分）的分数波动主要来源于 LLM 评分校准偏差，而非工件质量的真实变化 |
| **SSRF（Server-Side Request Forgery）** | 服务端请求伪造攻击。攻击者通过控制服务端发出的网络请求，访问内部资源或绕过访问控制。在 Claude Code 技能和 hook 中，如果脚本接受用户输入作为 URL 或文件路径而不加验证，可能存在 SSRF 风险 |
| **flag-name-drift（参数名漂移）** | 同一 CLI 参数在不同文件中使用了不同的名称（如 `--document` vs `--document-id`），导致用户无法判断哪个是正确的参数名，recipe 步骤可能因参数名错误而在运行时失败。本案例中的 CC-1 跨件一致性缺陷即属于 flag-name-drift |
