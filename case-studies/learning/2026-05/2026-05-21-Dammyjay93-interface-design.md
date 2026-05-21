# Dammyjay93/interface-design — 学习案例

**仓库**：https://github.com/Dammyjay93/interface-design
**Stars**：4630 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-05-21（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `manifest-discipline`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

interface-design 是一个面向 Claude Code 的纯语言插件，专注于 UI 界面设计的全流程辅助。它不含任何脚本、钩子或 MCP 配置，整个执行面为零——所有逻辑都用自然语言写成。

核心文件结构：
- `.claude-plugin/plugin.json` + `marketplace.json`：插件注册与发布元数据
- `.claude/commands/`：5 个命令（audit、critique、extract、init、status）
- `.claude/skills/interface-design/SKILL.md`：主技能文件（含 references/ 子目录）
- `.claude/skills/interface-design/references/`：4 个参考文件（critique.md、example.md、principles.md、validation.md）
- `reference/`：system-template.md 以及 2 个样本（system-precision.md、system-warmth.md）
- `.githooks/pre-commit`：提交前质量校验钩子
- `website/`：Vercel 部署的项目落地页

版本号为 `2026.2.9.1212`——这是一个 CalVer（日历版本号）格式：年.月.日.时分，粒度精细到分钟级。

### 1.2 架构剖析

该插件的核心设计轴是**设计记忆文件 system.md**。每次开始新项目时，`interface-design:init` 命令会在用户的工作目录写入一个 system.md，记录当前项目的设计系统（颜色 token、字体规范、间距体系）。之后的每一条命令都以 system.md 为读取基础：

- `init`：写入 system.md，加载 SKILL.md，建立设计上下文
- `status`：读取 system.md，展示当前设计系统状态
- `audit`：对照 system.md 审查已有界面的一致性
- `critique`：加载 references/critique.md，以"Correct vs Crafted"哲学对构建结果进行精工级点评
- `extract`：从现有代码中提炼并更新 system.md

SKILL.md 定义了 5 个操作阶段：Understand → Design System → Build → Validate → Maintain，以及 3 个设计支柱：Craft（匠心）、Memory（记忆）、Consistency（一致性）。

`references/principles.md` 涵盖 Surface & Token Architecture——前景/背景/边框/品牌/语义原语（primitives），构成整个设计 token 体系的理论骨架。

### 1.3 设计思路 / 方法论

**核心洞见："跨会话设计记忆"**

没有 system.md，每次 Claude 会话都从设计零点出发：不知道主色是什么、间距单位是多少、字体规范是什么。`init` 命令把这套记忆物化为文件，让 Claude 在后续任何会话中都能"记住"项目的设计语言。

**"Correct vs Crafted"哲学**

`critique` 命令建立在一个前提上：AI 首轮产出的界面是"功能正确的"，但不是"精工之作"。`references/critique.md` 的第一段即点明这一哲学——从 functional 到 beautiful 是一个需要刻意施压的过程，要求 Claude "退后一步，审视整体"，检查节奏感、密度、呼吸空间。

4630 颗星表明这个插件触达了大量设计师用户——Claude Code 生态中的非程序员群体比想象的更庞大。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**持久化上下文模式（Persistent Context Pattern）**：通过 init 命令在项目目录写入一个结构化 markdown 文件，后续所有命令读取该文件以获取领域上下文。这是一种"外部记忆"架构，不依赖 Claude 的上下文窗口，可跨会话持久。

**分层参考模式（Tiered Reference Pattern）**：SKILL.md 作为主技能文件，references/ 子目录存放深度参考。这避免了单一巨型 SKILL.md，也避免了过度碎片化——主文件负责流程，子文件负责深度知识。

**零执行面插件模式（Zero Execution Surface Pattern）**：整个插件不含任何可运行代码，消除了供应链攻击面。与 BayramAnnakov 等含 Python 脚本的插件相比，这种纯语言插件的安全审查成本为零。

### 2.2 适用场景

持久化上下文模式特别适合：
- 任何需要跨会话维护一致性的领域（设计系统、品牌规范、API 约定）
- 团队共享项目：system.md 可以提交进版本控制，让所有成员共享同一份"Claude 记忆"
- 存在"首次设置代价高"的场景——一次 init，终身受益

### 2.3 与其他架构对比

| 维度 | interface-design | 典型脚本型插件 |
|------|-----------------|--------------|
| 执行面 | 零 | 有（shell/Python） |
| 安全审查成本 | 极低 | 需要完整代码审查 |
| 跨会话记忆 | system.md（外部文件） | 通常无 |
| 参考文件分层 | SKILL.md + references/ | 多为单文件 |
| 版本语义 | CalVer（时间戳） | SemVer（语义） |

### 2.4 改进空间

尽管架构整体优秀，仍存在两处横向可见的弱点：

1. **allowed-tools 缺席**：5 个命令的 frontmatter 均未声明 `allowed-tools`。这意味着 Claude 可以在命令执行过程中调用任何工具，降低了可预期性，也让工具权限无法被最小化。

2. **SKILL.md 的幻象跨引用**：Scope 段落（第 14 行）将非界面设计请求重定向到 `/frontend-design`，但该命令在本仓库和注册表中均不存在。这是一个"指向虚空的链接"——对于真实用户来说，会产生困惑。

---

## 三、过去审查发现（历史快照）

### 3.1 当时质量评分（NLPM）

**总分：94/100**，Security：CLEAR（0 个安全发现）

| 文件 | 分数 | 主要扣分项 |
|------|------|----------|
| `.claude/commands/critique.md` | 85/100 | 无空输入处理 -10，缺 allowed-tools -5 |
| `.claude-plugin/plugin.json` | 95/100 | CalVer 格式——信息性提示 |
| `.claude/commands/audit.md` | 95/100 | 缺 allowed-tools -5 |
| `.claude/commands/extract.md` | 95/100 | 缺 allowed-tools -5 |
| `.claude/commands/init.md` | 95/100 | 缺 allowed-tools -5 |
| `.claude/commands/status.md` | 95/100 | 缺 allowed-tools -5 |
| `.claude/skills/interface-design/SKILL.md` | 98/100 | 含糊词"some" -2 |

### 3.2 当时值得借鉴的模式

- **命名空间规范**：所有命令均以 `interface-design:` 为前缀（`interface-design:init`、`interface-design:audit` 等），避免多插件冲突
- **plugin.json 路径匹配**：manifest 中声明的 commands/skills 路径与实际文件位置完全对应，零漂移
- **SKILL.md 分层**：SKILL.md 通过 `references/` 子目录组织深度知识，主文件保持简洁

### 3.3 当时的缺陷

**Bug（影响实际用户）**：

1. `SKILL.md` 第 381 行引用了不存在的 `references/principles.md`（深色模式细节、代码示例）
2. `SKILL.md` 第 382 行引用了不存在的 `references/validation.md`（内存管理、system.md 更新流程）
3. `SKILL.md` 第 383 行引用了不存在的 `references/critique.md`（构建后批评协议）

这 3 个 404 参考意味着：凡是想深入了解某个话题的用户，点击链接后均会遭遇"文件不存在"错误。

**质量问题**：
- 全部 5 个命令缺少 `allowed-tools` frontmatter
- `critique.md` 假设文件刚刚构建完成，无空输入处理
- SKILL.md 含糊词："some decisions are creative and others are structural"中的"some"无法量化

**跨组件问题**：
- SKILL.md 底部的 Commands 列表（第 388-391 行）遗漏了 `interface-design:init`（主入口命令竟然不在清单里）
- Scope 段落重定向至不存在的 `/frontend-design` 命令

### 3.4 当时的优化机会

- 补全 `references/` 目录下的 3 个缺失文件
- 为所有命令添加 `allowed-tools` 声明
- 将 `interface-design:init` 补入 SKILL.md 底部命令列表
- 将"some"替换为可量化描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | Audit 时（2026-04-27） | HEAD 时（2026-05-21） | 状态 |
|------|----------------------|---------------------|------|
| `references/principles.md` 缺失 | ❌ 不存在 | ✅ 已创建，含 Surface & Token Architecture 内容 | **已修复** |
| `references/validation.md` 缺失 | ❌ 不存在 | ✅ 已创建 | **已修复** |
| `references/critique.md` 缺失 | ❌ 不存在 | ✅ 已创建，含 frontmatter `name: interface-design:critique` | **已修复** |
| `allowed-tools` 缺失（5 个命令） | ❌ 全部缺席 | ❌ 仍全部缺席 | **未修复** |

额外奖励：作者还新建了 `references/example.md`，提供了设计 token 和间距值的具体 system.md 示例——这是 Audit 时未要求的额外工作。

### 4.2 架构演进

3 个 Bug 参考文件在约 3 周内全部修复完毕。这个修复速度说明有真实用户在使用该插件并报告了 404 错误，同时也证明作者在积极维护项目。

命令 frontmatter 现在有了正确的 `name:` 字段——`critique.md` 含 `name: interface-design:critique`，`status.md` 含 `name: interface-design:status`。这在 Audit 时未被特别标注，说明这是作者主动的质量改进。

`allowed-tools` 持续缺席这一现象值得关注：在修复了其他问题的同时，`allowed-tools` 始终未被补充。可能的原因有两个：作者不知道这个 frontmatter 字段的存在，或者作者有意选择不限制工具范围（认为"让 Claude 自由选择工具"更灵活）。

### 4.3 新增的可学习模式

**内置示例教学（Teaching by Example Within the Plugin）**：`references/example.md` 提供了具体的 system.md 写法样本（精准系统、温暖系统两种风格，对应 `reference/examples/system-precision.md` 和 `reference/examples/system-warmth.md`）。这是"文档自包含"的体现——用户无需离开插件就能看到规范的实例。

**提交钩子作为 NL 质量门（Git Hook as NL Quality Gate）**：`.githooks/pre-commit` 在提交前校验插件质量。这是把 NL 规范执行内嵌到开发流程的最简单方式，不依赖 CI/CD，开发者本地就能获得即时反馈。

---

## 五、校准

### 5.1 我已经在做对的

- **单一职责命令**：每条命令做一件事（audit 只审查，critique 只批评，extract 只提炼），符合 R01 单一目的原则
- **清洁 plugin.json 结构**：manifest 路径与实际文件位置对应，无漂移
- **命名空间规范**：`interface-design:*` 前缀一致，避免多插件命名冲突
- **零执行面意识**：纯语言插件的安全优势是真实的，不是理论上的

### 5.2 挑战 / 验证

**认知挑战**：4630 颗星说明 Claude Code 用户群体中有大量非程序员（设计师、产品经理）。这意味着 NL plugin 生态的宽度远超我的预期——"Claude Code 主要是给开发者用的"这个假设需要被修正。

**验证（已被 HEAD 证实）**：`references/` 里的 3 个 Bug 在约 3 周内就被修复，证明有真实用户在使用并反馈问题。这验证了一个规律：**高星仓库的 Bug 修复速度更快，因为用户压力更大**。

**新认知**：CalVer 版本号（`2026.2.9.1212` = 2026年2月9日12时12分）说明作者把每次变更都视为一次"发布"，不需要语义化地思考"这是 major 还是 minor"。对于频繁迭代的 NL 插件来说，CalVer 可能比 SemVer 更自然——时间戳本身就传达了"这是最新版本"的信息。

---

## 六、行动

### 6.1 自查动作

1. **检查所有命令的 frontmatter**：确认每个命令文件是否含有 `allowed-tools`。如果没有，决策：是刻意留空（全工具）还是应该补全（最小权限）？

2. **审查跨引用完整性**：所有 SKILL.md 中的 `references/` 内链是否指向实际存在的文件？用 grep 搜索 `references/` 的所有引用，逐一验证文件存在。

3. **检查命令列表完整性**：SKILL.md 底部的命令列表是否包含了所有命令（尤其是主入口命令）？

4. **审查重定向链接**：所有"如需 X，请使用 /Y 命令"的语句，确认 /Y 实际存在于本仓库或已知注册表中。

### 6.2 灵感 → 实施路径

**路径 A：为自己的插件添加 system.md 持久化上下文**

如果你的插件需要跨会话维护领域知识（API 约定、品牌规范、测试策略），可以仿照 interface-design 的 init + system.md 模式：
1. 编写 `init.md` 命令，负责在用户项目目录生成一个结构化 context 文件
2. 在其他命令的 frontmatter 中声明加载该 context 文件
3. 在 SKILL.md 中明确说明 context 文件的格式规范

**路径 B：用 references/ 分层你的知识结构**

如果你的 SKILL.md 超过 200 行，考虑：
1. 主 SKILL.md 保留流程和原则（概述层）
2. 将深度专题知识移入 `references/<topic>.md`
3. 在主 SKILL.md 用明确路径引用（`references/<topic>.md`）
4. 先写引用行，再创建文件——这样 Audit 工具能在文件缺失时给出警告（反面教材即 interface-design 的 Bug 1-3）

**路径 C：添加 .githooks/pre-commit 作为 NL 质量门**

在自己的 NL 插件仓库中添加提交前钩子，调用 `bin/nlpm-check` 或类似工具验证 frontmatter 完整性，在问题进入代码库之前拦截。

---

## 七、术语表

| 术语 | 解释 |
|------|------|
| **CalVer（日历版本号）** | Calendar Versioning 的缩写。版本号以时间戳编码而非语义，例如 `2026.2.9.1212` = 2026年2月9日12:12。适合频繁发布、无需向后兼容承诺的项目。 |
| **allowed-tools** | Claude Code 命令 frontmatter 中的字段，声明该命令执行期间允许调用的工具列表（如 `[Read, Write, Bash]`）。缺席时 Claude 可使用任何工具，最小权限原则要求显式声明。 |
| **system.md 设计记忆文件** | 由 `interface-design:init` 写入用户项目目录的持久化上下文文件，记录设计 token、字体规范、间距体系等。使 Claude 在后续会话中"记住"项目设计语言。 |
| **纯语言插件（Pure NL Plugin）** | 不含任何可执行代码（脚本、MCP 配置、包依赖）的 Claude Code 插件。全部逻辑以 markdown 自然语言编写。执行面为零，安全审查成本极低。 |
| **Correct vs Crafted（正确 vs 精工）** | interface-design 的核心设计哲学：AI 首轮产出的界面是"功能正确的（Correct）"，但距离"精工之作（Crafted）"仍有差距。critique 命令专门弥补这一差距。 |
| **零执行面（Zero Execution Surface）** | 插件中不存在任何可被操系统执行的代码表面，包括 shell 脚本、Python 文件、MCP 服务器配置等。减少供应链攻击向量的最彻底方式。 |
| **Surface & Token Architecture** | interface-design 的设计 token 分层体系，将颜色分为前景（Foreground）、背景（Background）、边框（Border）、品牌色（Brand）、语义色（Semantic）五类原语（primitives）。 |
| **持久化上下文模式** | 通过 init 命令在项目目录写入结构化文件，后续所有命令读取该文件以获取领域上下文的设计模式。克服了 AI 会话间记忆丢失的根本限制。 |
