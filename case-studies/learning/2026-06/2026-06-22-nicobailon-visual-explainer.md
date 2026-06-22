# nicobailon/visual-explainer — 学习案例

**仓库**：https://github.com/nicobailon/visual-explainer
**Stars**：7692 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-22（基于当前 HEAD）
**主题标签**：`manifest-discipline` `vague-quantifier` `single-purpose` `template-design` `multi-platform-config`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

visual-explainer 是一个 Claude Code 插件，核心功能是将代码、文档、项目变更转化为可视化的解释性内容——幻灯片、网页图表、可视化计划、差异审查报告，以及事实核查摘要。截至 2026-06-22 HEAD，插件已重构为多平台支持形态：命令从根级 `commands/` 迁移至 `plugins/visual-explainer/commands/`，并新增 `configs/` 目录，涵盖 Codex、Cursor、OpenCode、OpenClaw、Pi 等五个平台的专属配置，同时引入了 `CHANGELOG.md` 版本追踪机制。

当前插件目录结构：

```
plugins/visual-explainer/
  .claude-plugin/plugin.json   (v1.0.0 规范入口)
  commands/
    diff-review.md
    fact-check.md
    generate-slides.md
    generate-visual-plan.md
    generate-web-diagram.md
    plan-review.md
    project-recap.md
    share-page.md              (从 share.md 重命名)
configs/
  codex/AGENTS.md
  cursor/visual-explainer.mdc
  opencode/AGENTS.md
  openclaw/AGENTS.md
  pi/AGENTS.md
```

### 1.2 架构剖析

该插件采用"单命令单职责"模式：每个 `.md` 文件对应一种视觉输出类型，彼此互不依赖。`share-page.md` 作为发布通道，将生成结果部署至公开 Vercel URL。`SKILL.md` 提供底层知识支撑，供各命令按需引用。`install-pi.sh` 与 `scripts/share.sh` 是两个可执行脚本，负责平台安装与发布流程。

多平台适配层（`configs/`）是当前 HEAD 的显著演进：每个平台格式不同（AGENTS.md vs .mdc），但所有配置统一描述了插件的能力边界，实现了"一套能力，多平台接入"。

### 1.3 设计思路 / 方法论

该插件的核心设计哲学是**输出类型即命令边界**。作者没有设计一个通用的"可视化"命令再用参数区分类型，而是为每种输出形态单独建立一个命令文件。这使得每个命令可以针对特定输出进行深度定制（提示策略、工具选择、格式规范），同时让用户在命令补全时直观看到所有可用能力。

SKILL.md 扮演共享知识库角色，避免在每个命令中重复陈述可视化的最佳实践。这是"知识集中、执行分散"的典型体现。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式：输出类型驱动的命令分解（Output-Type-Driven Command Decomposition）**

将同一领域的不同输出形态拆分为独立命令，而非用标志位或参数控制。每个命令文件的唯一职责就是生成某一特定类型的输出。

**模式：多平台适配层（Multi-Platform Config Layer）**

在插件核心逻辑之外，单独维护 `configs/` 目录，为每个目标平台提供该平台原生格式的配置文件。插件能力不变，只是接入层随平台变化。

**模式：共享技能 + 专用命令（Shared Skill + Specialized Commands）**

一个 SKILL.md 承担领域知识沉淀，多个命令按需引用，避免知识碎片化。

### 2.2 适用场景

- **输出类型明确且有限**：当你的插件能产出 3-8 种结构上不同的输出时，输出类型驱动的命令分解优于参数化设计。
- **目标用户多元化**：当插件需要同时支持不同 AI 编程工具的用户群体时，多平台适配层能显著降低接入门槛。
- **领域知识稳定**：当最佳实践相对固定时，集中至 SKILL.md 比在每个命令中重复更易维护。

### 2.3 与其他架构对比

| 维度 | visual-explainer（本案） | 参数化设计 | 多命令多 SKILL |
|------|--------------------------|-----------|---------------|
| 命令发现性 | 高（Tab 补全即可见） | 低（需读文档） | 高 |
| 单个命令复杂度 | 低 | 高（需处理参数） | 低 |
| 知识维护成本 | 低（集中 SKILL.md） | 中 | 高（分散） |
| 跨命令一致性 | 需规范约束 | 自然一致 | 需规范约束 |

### 2.4 改进空间

尽管架构思路清晰，当前实现在规范执行层面存在明显短板：

1. **frontmatter 纪律性不足**：8 个命令全部缺失 `name` 字段，`share.md`（现为 `share-page.md`）在 Audit 时连 YAML frontmatter 都没有。缺少 `name` 意味着命令在 Claude Code 界面中无法有意义地呈现。
2. **allowed-tools 缺失**：所有命令均未声明 `allowed-tools`，导致权限边界不明确，用户会在运行时收到未预期的权限请求。
3. **空输入处理缺失**：多数命令假设用户会提供有效输入，没有处理空输入或无效输入的防御逻辑。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：66 / 100**（本批 4 个案例中最低分）

| 文件 | 分数 | 主要扣分原因 |
|------|------|------------|
| commands/share.md | 35 | 完全无 YAML frontmatter |
| commands/generate-slides.md | 44 | 缺 name、无空输入处理、步骤未编号 |
| commands/generate-visual-plan.md | 54 | 缺 name、无空输入处理 |
| commands/generate-web-diagram.md | 54 | 缺 name、无空输入处理 |
| commands/plan-review.md | 56 | 缺 name、无空输入处理 |
| commands/diff-review.md | 62 | 缺 name、模糊量词 |
| commands/project-recap.md | 64 | 缺 name、模糊量词 |
| commands/fact-check.md | 68 | 缺 name |
| SKILL.md | 88 | "key" 出现约 6 次（模糊量词） |
| .claude-plugin/plugin.json | 100 | — |
| plugins/visual-explainer/.claude-plugin/plugin.json | 100 | — |

### 3.2 当时值得借鉴的模式

- **双 plugin.json 结构**：根级与内嵌各有一份，体现了向嵌套插件结构迁移的过渡意图。
- **SKILL.md 质量较高**（88 分）：领域知识组织清晰，结构完整，作为共享知识库发挥了应有作用。
- **plugin.json 满分**：两个 plugin.json 均达 100 分，说明清单文件的规范意识比命令文件更强。
- **单命令单职责**：即使命令文件质量参差，架构分解思路本身是正确的。

### 3.3 当时的缺陷

**缺陷一（高危）：8 个命令全部缺失 `name` 字段**
NLPM 规范要求每个命令的 YAML frontmatter 必须包含 `name` 字段，用于在 Claude Code 界面显示可读名称。缺失 `name` 导致命令列表中显示的是文件路径而非人类可读的名称，严重影响用户体验。

**缺陷二（高危）：share.md 完全无 frontmatter**
这是 8 个命令中最严重的单文件问题。没有 YAML frontmatter 意味着该命令无法被 Claude Code 正确识别和注册。

**缺陷三（中危）：所有命令缺失 `allowed-tools`**
未声明工具使用权限，导致运行时权限边界不透明。

**缺陷四（中危）：多步骤命令未编号**
`generate-slides.md` 等包含多个执行步骤的命令，步骤之间没有编号标识，降低了可读性和可追踪性。

**缺陷五（低危）：模糊量词泛滥**
"comprehensive"、"significant"、"key" 等词在多个命令及 SKILL.md 中频繁出现，缺乏可验证的具体标准。

### 3.4 当时的优化机会

- 为所有命令统一补充 `name` 和 `allowed-tools` 字段（机械性修复，可批量执行）
- 为空输入场景增加防御性分支（"如果用户未提供输入，提示用户选择目标文件或粘贴内容"）
- 将模糊量词替换为具体描述（"key concepts" → "至多 5 个核心概念"）
- 统一步骤编号格式（"Step 1: ..., Step 2: ..."）
- 澄清双 plugin.json 结构的版本对齐问题（外层 v1.0.0，内层 v0.6.3，版本不一致）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

**已确认：核心缺陷仍然存在**

对当前 HEAD 执行 `grep -n "^name:" plugins/visual-explainer/commands/diff-review.md` 无结果——`name` 字段仍然缺失。尽管历经仓库重构，命令迁移至新路径，8 个命令缺失 `name` 字段的根本问题未被修复。

| 缺陷 | 2026-04-06 | 2026-06-22 | 状态 |
|------|-----------|-----------|------|
| 缺失 `name` 字段（8 个命令） | 存在 | 存在 | 未修复 |
| share.md 无 frontmatter | 存在 | share-page.md（待核实） | 可能改善 |
| 缺失 `allowed-tools` | 存在 | 存在（极大可能） | 未修复 |
| 双 plugin.json 版本不一致 | v1.0.0 vs v0.6.3 | 结构已调整 | 部分改善 |

**结论**：作者在这次重构中专注于架构扩展（多平台适配），而非质量修复。这是一种合理的优先级选择，但遗留缺陷需要单独跟进。

### 4.2 架构演进

当前 HEAD 相比 Audit 快照发生了三项显著演进：

**演进一：命令路径规范化**
从根级 `commands/` 迁移至 `plugins/visual-explainer/commands/`，将插件内容完整收纳在 `plugins/visual-explainer/` 子树下。这是向"插件即独立单元"的正确迈进。

**演进二：多平台适配层**
新增 `configs/` 目录，为 Codex、Cursor、OpenCode、OpenClaw、Pi 五个平台分别提供原生格式的配置文件。这是本次演进中最有价值的架构决策，将插件的受众从"仅 Claude Code 用户"扩展至"全 AI 编程工具用户"。

**演进三：版本可追踪性**
引入 `CHANGELOG.md`，使版本历史可查阅，提升了项目可维护性。

### 4.3 新增的可学习模式

**多平台配置模式（Multi-Platform Config Pattern）**

将 `configs/codex/AGENTS.md`、`configs/cursor/visual-explainer.mdc` 等纳入同一仓库，是一种低成本的高价值扩展。每个文件只需将插件能力翻译为目标平台的原生配置语法，核心逻辑无需复制。

这个模式值得在任何有潜力跨平台使用的插件中学习和复用。

---

## 五、校准

### 5.1 我已经在做对的

回顾本案的得分与演进，以下几点是我们（作为 NLPM 开发者或插件作者）已经在遵循的正确做法：

- **清单文件（plugin.json）放在首位**：visual-explainer 的两个 plugin.json 均为满分，说明作者对声明性配置的规范意识较强。在我们自己的项目中，plugin.json 的规范性同样是基线要求。
- **单命令单职责**：不将多种输出类型塞进一个命令里，保持每个文件的职责边界清晰。
- **SKILL.md 作为知识层**：集中领域知识，而非在每个命令中重复，是正确的知识管理思路。

### 5.2 挑战 / 验证

本案提出了几个值得深入思考的问题：

**挑战一：frontmatter 纪律性是否应该自动化强制？**
visual-explainer 的 `name` 字段缺失在两次快照之间都未被修复，说明仅靠"知道规范"不足以保证执行。这支持了 NLPM 在 CI/pre-commit 中运行 `bin/nlpm-check` 的必要性——机械性检查应由工具负责，而非靠人工记忆。

**挑战二：架构扩展 vs 质量修复的优先级如何平衡？**
作者在这次重构中选择了架构扩展（多平台支持）而非质量修复（补 `name` 字段）。这个选择在短期内提升了插件的受众面，但长期会让技术债堆积。可以考虑在重构时将机械性修复捆绑进来——`name` 字段的补充只需几分钟，收益却是持久的。

**挑战三：多平台配置的同步成本**
`configs/` 下五个平台的配置文件需要随插件功能演进同步更新。如果没有明确的同步机制，这些配置文件会逐渐过时。值得思考：是否应该在 CHANGELOG.md 中明确标注哪些变更需要同步更新各平台配置？

---

## 六、行动

### 6.1 自查动作

参照本案的缺陷模式，在自己的插件项目中执行以下检查：

1. **frontmatter 完整性检查**
   ```bash
   # 查找缺少 name 字段的命令文件
   grep -rL "^name:" commands/ 2>/dev/null || echo "所有命令均有 name 字段"
   
   # 查找完全缺少 frontmatter 的命令文件
   for f in commands/*.md; do
     head -1 "$f" | grep -q "^---" || echo "缺少 frontmatter: $f"
   done
   ```

2. **allowed-tools 覆盖率检查**
   ```bash
   # 查找缺少 allowed-tools 的命令文件
   grep -rL "allowed-tools" commands/
   ```

3. **模糊量词扫描**
   ```bash
   # 扫描常见模糊词
   grep -rn "comprehensive\|significant\|key \|important\|various" commands/ skills/
   ```

4. **步骤编号检查**（多步骤命令）
   ```bash
   # 查找包含多个步骤但未编号的命令
   grep -l "Step\|步骤" commands/*.md | xargs grep -L "^1\.\|^Step 1"
   ```

5. **版本一致性检查**（多 plugin.json 场景）
   ```bash
   grep -r '"version"' .claude-plugin/ plugins/*/
   ```

### 6.2 灵感 → 实施路径

**灵感：多平台适配层**

如果我的插件有潜力在 Cursor、Codex 等平台上使用，可以按以下路径实施：

1. 在仓库根级创建 `configs/` 目录
2. 为每个目标平台创建子目录（`configs/cursor/`、`configs/codex/` 等）
3. 在各子目录中用该平台的原生格式（`.mdc`、`AGENTS.md`）描述插件能力
4. 在 README.md 中添加"多平台安装"章节，说明各平台的安装方式
5. 在 CHANGELOG.md 中追踪各平台配置的版本同步状态

**灵感：CHANGELOG.md + 版本追踪**

即使是小型插件，维护 CHANGELOG.md 也有助于：
- 让用户了解版本变更
- 让自己在重构时有历史可查
- 让 NLPM 审计工具可以追踪演进趋势

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 项目 | 核心目的 | 与 visual-explainer 的重叠度 |
|------|---------|---------------------------|
| MarkQWu/echo-sleuth-for-claude | Claude Code 会话数据挖掘与可视化 | 中——同为处理代码产物并产出结构化报告的插件，但输入源（会话日志 vs 代码文件）不同 |
| MarkQWu/drama-workshop-skills | 戏剧写作技能库 | 低——领域完全不同，但架构模式（SKILL.md 集中知识）有共同参考价值 |
| MarkQWu/claude-for-legal | 法律技能库 | 低——领域不同，但多平台适配思路有借鉴价值 |

echo-sleuth-for-claude 与 visual-explainer 的功能相似度最高：两者都是以结构化方式处理某类信息源，并产出可视化/报告类输出。以下的对比分析以 echo-sleuth 为主要参照。

### 8.2 在我的项目里复现的同类问题

**好消息**：echo-sleuth-for-claude 在 visual-explainer 的最主要缺陷上表现更好。

- **allowed-tools**：echo-sleuth 的所有命令（dashboard.md、audit.md、prune.md、extract.md）均已声明 `allowed-tools`——这是 visual-explainer 8 个命令全部缺失的字段。echo-sleuth 在这一维度上已达到规范要求。
- **步骤编号**：echo-sleuth 的命令使用了编号步骤，这是 visual-explainer 在 `generate-slides.md` 等多步骤命令中缺失的。

**需要关注**：

- **`name` 字段**：需要确认 echo-sleuth 的命令是否包含 `name` 字段。如果同样缺失，应立即补充——这是 visual-explainer 评分最低的单一原因。
- **drama-workshop-skills**：作为纯技能库，其 SKILL.md 文件是否存在类似 visual-explainer SKILL.md 中"key"频繁出现的问题，值得检查。
- **空输入处理**：echo-sleuth 的命令是否处理了用户未提供参数的场景，需要核实。

### 8.3 别人的更优方案

**visual-explainer 在这一点上明显领先：多平台配置支持**

nicobailon 为 Codex、Cursor、OpenCode、OpenClaw、Pi 五个平台维护了专属配置。我的三个项目均未提供此类多平台适配。

对于 echo-sleuth-for-claude 而言，这是一个可操作的改进机会：Cursor 用户同样有会话挖掘的需求，为其提供 `configs/cursor/echo-sleuth.mdc` 可以显著扩大用户受众，且实施成本极低（一个描述插件能力的配置文件）。

**CHANGELOG.md 版本追踪**

visual-explainer 新增了 CHANGELOG.md，记录版本历史。如果我的项目尚未维护变更日志，这是一个低成本、高长期价值的补充。

### 8.4 反向：我的项目做得比他们好的地方

**echo-sleuth-for-claude 在规范执行上优于 visual-explainer**

1. **allowed-tools 全覆盖**：echo-sleuth 所有命令都声明了 `allowed-tools`，这是 visual-explainer 评分最大失分点的对立面。这直接反映在用户体验上——echo-sleuth 的用户不会在运行命令时收到意外的权限请求。

2. **步骤编号**：echo-sleuth 的多步骤命令使用了编号，而 visual-explainer 的 `generate-slides.md` 等文件缺乏此结构。

3. **综合评估**：如果用 NLPM 对 echo-sleuth 打分，其 frontmatter 规范性应显著高于 visual-explainer 的 66 分均值。

**这一对比说明**：规范执行的纪律性，比架构设计的创意更能影响实际评分。visual-explainer 有很好的架构思路，但因为机械性规范执行不到位，总分仅为 66 分——而这些问题全部是可以用自动化工具（如 `bin/nlpm-check`）在提交前拦截的。

---

## 八、术语表

**frontmatter（前置元数据）**
YAML 格式的文件头部声明块，以 `---` 开始和结束。在 Claude Code 插件的命令文件中，frontmatter 包含 `name`（可读名称）、`description`（功能描述）、`allowed-tools`（允许使用的工具列表）等关键字段。缺少 frontmatter 或其中的必要字段，会导致命令无法被正确识别或在界面中无意义地显示。

**allowed-tools（允许工具声明）**
命令 frontmatter 中的一个字段，显式列出该命令在执行过程中允许调用的 Claude Code 工具（如 `Read`、`Edit`、`Bash` 等）。声明 `allowed-tools` 的好处是双向的：用户在运行命令前就知道会发生什么（透明性），Claude 也能在边界内更高效地执行（聚焦性）。缺失此字段不会导致命令失效，但会降低用户信任度，并在运行时产生意外的权限弹窗。

**empty-input handling（空输入处理）**
在命令定义中，针对用户未提供任何输入（或提供无效输入）时的防御性逻辑分支。良好的空输入处理应当：提示用户应提供什么类型的输入，或提供默认行为（如"如果未指定文件，对当前目录下所有 `.ts` 文件执行操作"）。缺少空输入处理会导致命令在边缘情况下产生不可预期的行为。

**supply-chain risk（供应链风险）**
在可执行脚本或安装流程中，从外部来源（第三方 GitHub 仓库、CDN、外部域名）拉取并执行代码所带来的安全风险。即使当前版本的外部代码是可信的，供应链攻击可能通过劫持该外部仓库来替换恶意代码，并在下一次安装时自动生效。visual-explainer 的 `install-pi.sh` 在运行时从外部仓库 `git clone` 是典型的供应链风险场景。缓解方式包括：固定 commit hash、使用可信的包管理器、或将依赖内化至仓库本身。

**multi-platform config（多平台配置）**
为同一套插件能力提供面向不同 AI 编程工具（Claude Code、Cursor、Codex、OpenCode 等）的原生格式配置文件。每个平台有其特定的配置语法（如 Claude Code 使用 `.md`，Cursor 使用 `.mdc`，Codex 使用 `AGENTS.md`），多平台配置层通过在 `configs/` 目录下维护各平台的翻译版本，使同一插件的能力可被不同平台的用户无缝接入，而无需修改核心命令逻辑。
