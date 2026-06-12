# gotalab/cc-sdd — 学习案例

**仓库**：https://github.com/gotalab/cc-sdd
**Stars**：3099 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-12（基于当前 HEAD）
**主题标签**：`template-design`, `cross-reference`, `manifest-discipline`, `vague-quantifier`, `examples-driven`

> 参考案例：[xiaolai 案例（2026-04-27）](../2026-04-27-gotalab-cc-sdd.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`cc-sdd`（Claude Code Spec-Driven Development）是 gotalab 开发的**跨平台 Kiro 式规格驱动开发工具包**，GitHub 3099 stars。它把整个 SDD 工作流——从 steering（项目引导）到 spec-init → spec-requirements → validate-gap → spec-design → validate-design → spec-tasks → spec-impl → validate-impl 共 8 个阶段——封装成 8 个 AI 编码平台的命令/提示词模板集合。

关键事实：
1. 支持 **8 个平台**：claude-code、antigravity-skills、opencode、github-copilot、windsurf、codex、gemini-cli-skills、windsurf-skills
2. 共 **102 个 NL 工件**，是典型的大规模跨平台模板项目
3. 新增了真实的 `.kiro/specs/` 示例（`customer-support-rag-backend-ja`、`photo-albums-en` 等）
4. 新增了 TypeScript 渲染器（`tools/cc-sdd/src/`、`tools/cc-sdd/test/`），说明项目在演进为可执行工具链

### 1.2 架构剖析

```
cc-sdd/
├── CLAUDE.md                          # Kiro SDD 工作流总览（当前版本已文档化完整 8 阶段流程）
├── tools/cc-sdd/
│   ├── templates/agents/<platform>/   # 8 个平台子目录，每个含 commands/ 和 docs/
│   │   ├── claude-code/
│   │   │   ├── commands/              # 8 个命令文件（spec-init.md, spec-requirements.md 等）
│   │   │   └── docs/CLAUDE.md         # 平台引导文档
│   │   ├── github-copilot/prompts/    # 9 个 prompt 文件
│   │   ├── windsurf/commands/         # 9 个命令
│   │   ├── codex/commands/            # 9 个命令
│   │   ├── opencode/commands/         # 7 个命令
│   │   ├── gemini-cli-skills/         # skill 风格（30 文件）
│   │   ├── windsurf-skills/           # skill 风格（30 文件）
│   │   └── gemini-cli/               # 含 spec-reviewer.md（存在 B3 缺陷）
│   ├── src/                           # TypeScript 渲染器源码（新增）
│   └── test/                          # TypeScript 测试（新增）
├── .kiro/
│   ├── specs/                         # 真实示例规格（新增，customer-support-rag-backend-ja 等）
│   └── settings/templates/            # steering 和 spec 模板（新增）
└── kiro-steering/                     # 4 个 steering skill（评分最高，95分）
```

- **文件类型分布**：102 个 NL 工件，含 commands、prompts、skills、docs 四类
- **编排关系**：CLAUDE.md 说明阶段顺序 → 用户依次调用对应命令 → 命令引导 AI 生成或验证对应 spec 文档 → 所有输出落地到 `.kiro/specs/<feature>/`
- **跨件契约**：模板变量系统（`{{KIRO_DIR}}`、`{{FEATURE_NAME}}`、`{{TIMESTAMP}}`、`{{PROJECT_DESCRIPTION}}`）是跨平台一致性的核心机制；各平台命令基于同一工作流模型，只是语法适配不同平台

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「把 AI 的规格驱动开发工程化」——SDD 不是一个命令，而是一条 8 阶段流水线，每个阶段有明确的输入、输出和验证标准
- **解决什么问题**：AI 编码助手缺乏系统性需求分析和设计阶段，直接跳到实现导致返工。cc-sdd 强制要求 requirements → design → tasks 三层规格在实现前锁定
- **Trade-off**：跨平台模板复制导致 102 个文件大量同质化内容，维护成本高；换取了「哪个平台的用户都能直接用」的开箱即用性
- **认知模型**：把 AI 辅助开发看作「外包团队协作」——steering 相当于项目章程，requirements 相当于需求文档，design 相当于技术方案评审，tasks 相当于工单拆解；AI 是执行者，人类是各阶段的评审者

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多阶段串行流水线 + 跨平台模板矩阵」**

关键特征：
- 8 个工作流阶段显式序列化，不可跳步（validate-gap 必须在 spec-requirements 之后，validate-design 必须在 spec-design 之后）
- 同一工作流逻辑在 8 个平台各有一套命令模板，形成「8×N 矩阵」
- [模板变量](#模板变量)（`{{KIRO_DIR}}` 等）作为平台无关的抽象层
- kiro-steering skill（评分 95 分）是最成熟的工件，用于项目级引导
- [manifest](#manifest) 层面目前尚未有统一的 plugin.json（不是单一 Claude Code 插件），各平台分别安装

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业级功能开发，需求→设计→实现全流程追溯 | ✅ 高度适用 | 8 阶段流水线强制规格化，输出可审计 |
| 跨平台团队（有人用 Claude Code，有人用 GitHub Copilot） | ✅ 适用 | 8 平台覆盖，相同工作流不同入口 |
| 个人快速原型，不需要规格文档 | ❌ 过重 | 8 阶段流程会显著增加启动摩擦 |
| 需要 AI 完全自主执行的无人值守任务 | ❌ 不适用 | validate-gap / validate-design 需要人工确认 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多阶段 SDD 流水线 | gotalab/cc-sdd | 系统化、可追溯、防止规格缺失 | 102 个工件维护成本高；大量文件缺少 frontmatter |
| 单职责 skill 集合 | wshobson/agents | 简单直观，每个 skill 独立可用 | 无阶段约束，AI 容易跳过设计直接实现 |
| 规则驱动框架 | SuperClaude | 通过 CLAUDE.md 规则约束行为 | 没有显式的阶段验证节点 |
| 多 agent 编排框架 | first-fluke/oh-my-agent | 跨厂商，自动路由 | 无 SDD 规格文档体系 |

### 2.4 改进空间

1. **当前问题**：102 个工件中约 70% 缺少 [frontmatter](#frontmatter)（无 `name`、无 `model`、无 `description`），导致 NLPM 评分 60/100，低于 70 分阈值。**改进做法**：为每个命令文件添加标准化 frontmatter（`name`、`description`、`model`、`allowed-tools`），可批量生成再人工校验。**预期收益**：从 60 分提升到 80 分以上，Claude Code 可正确注册和调用命令。
2. **当前问题**：`{{DEV_GUIDELINES}}` 模板变量在 3 个生产文档中未填充，作为占位符直接暴露给用户。**改进做法**：（a）在 README 中明确文档「安装后必填变量」清单；（b）在未填充时给出默认值或注释提示。**预期收益**：消除 B4 类型缺陷，降低用户困惑。
3. **当前问题**：大多数命令缺少具体使用示例（-15 分/文件）。**改进做法**：为每个命令添加「示例输入 → 预期输出」段落，可参考 kiro-steering skill 的高质量写法。**预期收益**：用户学习曲线降低，AI 执行时有参考锚点。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 得分 **60/100**，低于 70 分阈值。

| 文件类别 | 当时分数 | 主要问题 |
|---|---|---|
| claude-code/docs/CLAUDE.md | 20 | 无 frontmatter；`{{DEV_GUIDELINES}}` 未填充 |
| 所有平台 AGENTS.md（5 个文件） | 30 | 无 frontmatter；`{{DEV_GUIDELINES}}` 未填充 |
| gemini-cli/spec-reviewer.md | 30 | 无 frontmatter，无结构（B3 缺陷） |
| claude-code commands（8 个文件） | 45–55 | 无 name、无 model、无 examples |
| opencode commands（7 个文件） | 50 | 无 name、无 model、无 allowed-tools、无 examples |
| github-copilot prompts（9 个文件） | 50 | 无 name、无 model、无 allowed-tools、无 examples |
| windsurf commands（9 个文件） | 50 | 无 name、无 model、无 allowed-tools、无 examples |
| codex commands（9 个文件） | 50 | 无 name、无 model、无 allowed-tools、无 examples |
| steering-custom commands | 65–70 | 无 name、无 model |
| gemini/windsurf skills（30 个文件） | 80 | 无 model、无 examples |
| kiro-steering skills（4 个文件） | **95** | 无 model（唯一缺陷） |

**加权平均：60/100**

### 3.2 当时值得借鉴的模式

1. **8 阶段 SDD 工作流设计** → 为什么好：每个阶段有明确职责，validate-gap 和 validate-design 两个验证节点防止规格缺失蔓延到实现阶段，是系统性思维的体现。路径：`CLAUDE.md`。如何借鉴：为自己的工作流设计显式「验证节点」，而非一路向前直到实现才发现问题。
2. **跨平台模板一致性** → 为什么好：同一 SDD 工作流在 8 个平台保持语义一致，用户切换平台时无需重新学习工作流本身。路径：`tools/cc-sdd/templates/agents/<platform>/`。如何借鉴：设计多平台工具时，先定义平台无关的「工作流核心」，再为每个平台单独做语法适配。
3. **kiro-steering skill 的高质量写法** → 为什么好：4 个 steering skill 评分 95 分，结构完整（有描述、有步骤、有输出规范），是同仓库中对比最鲜明的优秀样本。路径：`kiro-steering/`。如何借鉴：把这 4 个文件作为仓库内部的「写法标杆」，要求其他命令文件向它对齐。

### 3.3 当时的缺陷

| 缺陷编号 | 位置 | 问题描述 |
|---|---|---|
| B1 | steering.md | 工具名错误：`glob_file_search`、`read_file`、`list_dir` → 应为 `Glob`、`Read`、`LS` |
| B2 | spec-status.md | `allowed-tools` 声明了 `Write/Edit/MultiEdit/Update`，但这是只读命令 |
| B3 | gemini-cli/spec-reviewer.md | 完全无 frontmatter，Gemini CLI 无法注册该命令 |
| B4 | 7 个 docs 文件 | `{{DEV_GUIDELINES}}` 模板占位符未填充，直接暴露给用户 |
| B5 | windsurf-skills/docs/AGENTS.md | `@` 在路径符号中的用法破坏 skill 解析 |

### 3.4 当时的优化机会

1. 为全部 102 个命令文件批量添加 `name`、`model`、`description` 字段（-25pt 的最大单项扣分项）
2. 为每个命令补充 1-2 个具体使用示例（examples-driven 原则）
3. gemini/windsurf skills（30 个文件）补充 `model` 字段即可从 80 分升至 85+ 分
4. spec-status.md 移除只读命令中的写工具声明

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查结果 | 现状 | 含义 |
|---|---|---|---|
| B1：工具名错误（glob_file_search 等） | `grep -r "glob_file_search\|read_file\|list_dir"` 无命中 | **已修复** ✅ | 工具名已更正，steering.md 中正确使用 Claude Code 工具名 |
| B4：`{{DEV_GUIDELINES}}` 未填充 | 当前仍出现在 3 个生产文档中（github-copilot/AGENTS.md, opencode-skills/AGENTS.md, claude-code/CLAUDE.md） | **部分修复** ⚠️ | 从 7 个减少到 3 个，有改进但未彻底解决 |
| B5：`@` 路径符号 | 当前 AGENTS.md 第 54 行的 `@kiro-<skill-name>` 是调用语法而非路径语法 | **实质已修复** ✅ | 上下文改变，路径层面的问题已消失 |
| Q42：无 model 声明 | steering.md frontmatter 中仍无 `model:` 字段，全平台持续 | **持续存在** ❌ | 非确定性运行时行为风险未消除 |

### 4.2 架构演进

从审计时的纯模板集合 → 现在增加了两个重要层次：
1. **`.kiro/specs/` 真实示例**：`customer-support-rag-backend-ja`、`photo-albums-en` 等真实规格示例出现，说明作者开始用自己的工具构建真实项目，这是「eating one's own dog food」的积极信号
2. **TypeScript 渲染器工具链**：`tools/cc-sdd/src/` 和 `tools/cc-sdd/test/` 说明项目从「静态模板」向「可执行生成器」演进，用户未来可能通过运行渲染器自动生成特定平台的命令文件

这两点表明 cc-sdd 正在从「模板仓库」转型为「SDD 工具平台」。

### 4.3 新增的可学习模式

- **示例即文档（`.kiro/specs/` 实例）**：真实的 `customer-support-rag-backend-ja` 规格示例比任何文字说明都更有说服力——用户看到一个完整的 requirements.md → design.md → tasks.md 链条，立刻理解 SDD 的产出形态。
- **TypeScript 渲染器分层**：模板（可读）+ 渲染器（可执行）+ 输出（平台特定）三层分离，是「约定 → 工具 → 产出」的清晰架构。

---

## 五、校准

### 5.1 我已经在做对的

1. **工作流文档化**：drama-workshop-skills 的 SKILL.md 包含明确的工作步骤描述，与 cc-sdd 的 8 阶段流水线思路一致——先定义流程，再让 AI 执行。
2. **evals 目录**：drama-workshop-skills 有 `evals/` 目录，这是 cc-sdd 完全缺失的系统性质量评估机制。我在这一点上做得更好。
3. **plugin.json 完整元数据**：echo-sleuth 的 plugin.json 包含完整元数据（name、description、version），比 cc-sdd 的平台分散安装方式更规范。

### 5.2 挑战 / 验证

本案例验证了一个关键洞察：**「模板变量的生命周期管理」是模板类项目的核心风险点**。cc-sdd 的 `{{DEV_GUIDELINES}}` 占位符从审计时的 7 个文件减少到 3 个，说明作者在有意修复，但没有机制防止新文件再次引入未填充的占位符。这提示我：如果 drama-workshop-skills 将来引入模板变量，需要同时建立「未填充占位符检测」机制（比如在 pre-commit hook 或 CI 里 grep `{{[A-Z_]+}}`）。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的项目是否有未填充的模板变量占位符
grep -rn "{{[A-Z_]\+}}" /tmp/my-repos/MarkQWu-drama-workshop-skills/ 2>/dev/null
grep -rn "{{[A-Z_]\+}}" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null
# 命中后：要么填充默认值，要么在 README 中明确说明「用户必须替换」

# 检查我的 skill 文件是否有 model 字段声明
grep -rL "^model:" /tmp/my-repos/MarkQWu-drama-workshop-skills/*.md 2>/dev/null
grep -rL "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude/skills/ 2>/dev/null
# 命中后：根据任务复杂度添加 model: haiku 或 model: sonnet

# 检查我的命令文件是否有 examples 段落
grep -rL "## Example\|## 示例\|## Usage Example" /tmp/my-repos/MarkQWu-drama-workshop-skills/ 2>/dev/null
# 命中后：为每个命令添加「示例输入 → 预期输出」段落
```

```bash
# 检查 drama-workshop-skills 的 short-drama SKILL.md 是否有空输入处理
grep -n "empty\|空输入\|no input\|if.*empty\|当.*为空" \
  /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md 2>/dev/null
# 未命中则说明缺少 Q32 类型的空输入处理
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 设计显式的多阶段创作流水线（借鉴 cc-sdd 的 8 阶段 SDD）
   - **为何可行**：短剧创作本身就有「主题 → 大纲 → 人物 → 场景 → 剧本」的自然阶段；当前 SKILL.md 把所有逻辑混在一起
   - **第一步**：在 `short-drama/SKILL.md` 中增加显式阶段标题（`## Phase 1: 主题确认`、`## Phase 2: 大纲生成` 等），每个阶段声明输入、输出和验证标准
   - **预期收益**：用户可以在任意阶段暂停和审查，减少 AI 直接跳到剧本生成导致的方向偏差

2. **想法**：为 echo-sleuth 的 `.kiro/specs/` 等效位置添加真实使用示例
   - **为何可行**：cc-sdd 新增 `.kiro/specs/customer-support-rag-backend-ja` 后可读性大幅提升；echo-sleuth 缺乏类似的「真实使用案例」展示
   - **第一步**：创建 `examples/` 目录，放一个真实的对话挖掘示例（输入：会话历史文件 → 输出：结构化洞察报告）
   - **预期收益**：新用户 5 分钟内理解插件价值，降低首次使用的认知门槛

3. **想法**：在 drama-workshop-skills 的 CI 中加入模板占位符检测
   - **为何可行**：cc-sdd 的 B4 缺陷（`{{DEV_GUIDELINES}}` 未填充）说明占位符管理是模板项目的通用痛点
   - **第一步**：在 `.github/workflows/` 中添加一步 `grep -rn "{{[A-Z_]\+}}" . && exit 1`，阻止含未填充占位符的提交合并
   - **预期收益**：防止同类问题在我的项目中出现

---

## 七、对照我的 GitHub 仓库

> 数据源：drama-workshop-skills、claude-for-legal、echo-sleuth-for-claude

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **cc-sdd 的核心目的**：通过多阶段规格化流程，让 AI 编码助手在实现前完成充分的需求和设计分析
- **最相似的我的仓库**：`drama-workshop-skills`——两者都试图把一个多步骤的「专业工作流」（SDD 规格化 / 短剧创作）用 NL 工件驱动
- **对齐分析**：drama-workshop-skills 是单平台（Claude Code）单领域（短剧）；cc-sdd 是多平台（8 个）单领域（SDD）。我的项目缺少 cc-sdd 的跨平台野心，但多了系统性质量评估（evals/）

### 8.2 在我的项目里复现的同类问题

1. **Q42 等效：drama-workshop-skills 的 SKILL.md 无 `model:` 字段**
   - 与 cc-sdd 的 Q42 完全对应：所有平台均无 model 声明，导致运行时使用的模型不可预测
   - 现状：`short-drama/SKILL.md` 和 `short-drama-remake/SKILL.md` 的 frontmatter 中均无 `model:` 字段
   - 修复成本：极低，每个文件添加 1 行 `model: sonnet`（短剧创作复杂度适合 sonnet 级别）

2. **空输入处理缺失（Q32 等效）**
   - cc-sdd 的 8 个命令中有 8/11 缺少空输入处理；drama-workshop-skills 的 SKILL.md 需要检查是否处理了「用户未提供主题/类型」的情况
   - 如果未处理：当用户只输入 `/short-drama` 不加任何参数时，AI 可能自由发挥或报错

3. **无 examples 段落**
   - cc-sdd 最大扣分项之一；drama-workshop-skills 目前也缺少具体的「示例输入 → 示例输出」展示
   - 修复：在 SKILL.md 末尾加 `## 使用示例` 段落，展示一个完整的短剧主题 → 最终剧本片段的链路

### 8.3 别人的更优方案（可以借鉴到我的项目）

1. **cc-sdd 的 8 阶段串行流水线** > drama-workshop-skills 的单一 SKILL 结构
   - gotalab 将工作流拆分为 8 个显式阶段，每个阶段有专属命令和验证节点。drama-workshop-skills 把所有逻辑压缩在 1-2 个 SKILL.md 中，没有显式的「验证」环节
   - 借鉴方法：在 short-drama SKILL 内部或作为独立命令，增加「大纲验证」（用户确认主题和人物设定）和「剧本验证」（用户确认场景安排）两个检查点，防止 AI 在方向错误时继续执行

2. **cc-sdd 的跨平台模板矩阵**
   - cc-sdd 支持 8 个 AI 平台，drama-workshop-skills 目前只支持 Claude Code
   - 借鉴方法：优先适配 1 个第二平台（建议 github-copilot，用户基数大），验证「短剧创作工作流」的平台无关性

3. **`.kiro/specs/` 形式的真实示例**
   - cc-sdd 通过真实的 `customer-support-rag-backend-ja` 示例展示完整工作流产出，极大降低理解成本
   - 借鉴方法：在 drama-workshop-skills 的 `examples/` 目录放一个完整的短剧创作案例（输入主题 → 大纲 → 最终剧本片段），让用户看到真实产出

### 8.4 反向：我的项目做得比他们好的地方

1. **系统性质量评估（evals/）**：drama-workshop-skills 有 `evals/` 目录，提供了对 SKILL 产出质量的系统性评估框架。cc-sdd 的 102 个工件中没有任何等效机制——无法知道 spec-design 命令生成的设计文档质量是否达标。这是 cc-sdd 的重大盲点。
2. **规范的 plugin.json 元数据**：echo-sleuth 的 `plugin.json` 有完整的 `name`、`description`、`version`、`author` 字段，可以通过 `claude plugin install` 一键安装。cc-sdd 没有统一的插件清单，需要用户手动复制文件到各平台目录。
3. **单仓库聚焦**：drama-workshop-skills 和 echo-sleuth 各自聚焦单一领域，文件数量在可控范围内（各约 10 个工件）。cc-sdd 的 102 个工件导致维护成本极高，`{{DEV_GUIDELINES}}` 这类问题在小规模仓库中几乎不会出现。

---

## 八、术语表

**frontmatter**
Markdown 文件顶部用 `---` 包裹的 YAML 元数据块。Claude Code 用它来注册命令（`name` 字段）、限制工具调用（`allowed-tools`）、指定模型（`model`）。缺少 frontmatter 的命令文件无法被 AI 运行时正确识别和注册。cc-sdd 约 70% 的工件缺少 frontmatter，是其低评分（60/100）的主因。

**SDD（规格驱动开发，Spec-Driven Development）**
一种软件开发方法论，要求在编码前完成需求规格（requirements）、设计规格（design）和任务规格（tasks）三层文档的编写和人工审查。Kiro 是 AWS 推出的 SDD 工具，cc-sdd 将其核心工作流移植为 AI 编码助手的命令集合。与 TDD（测试驱动开发）类比：SDD 是「规格先行」，TDD 是「测试先行」。

**manifest（清单）**
描述一个 Claude Code 插件的结构和安装路径的配置文件，通常为 `plugin.json`。它声明插件包含哪些 skills、commands、agents，以及它们在用户环境中的安装目标路径。cc-sdd 缺少统一的 plugin.json，各平台模板需要用户手动安装，增加了使用摩擦。

**模板变量**
文件中用特定符号（通常是双大括号，如 `{{KIRO_DIR}}`、`{{DEV_GUIDELINES}}`、`{{FEATURE_NAME}}`）标记的占位符，预期在安装或初始化时由用户或脚本替换为实际值。cc-sdd 的 B4 缺陷（`{{DEV_GUIDELINES}}` 未填充）展示了模板变量生命周期管理不善的典型后果：占位符原样暴露给最终用户，造成困惑或功能失效。
