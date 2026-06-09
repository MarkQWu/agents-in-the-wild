# upstash/context7 — 学习案例
**仓库**: https://github.com/upstash/context7
**Stars**: 53,665 | **来源**: xiaolai upstream | 本地 audit
**Audit 日期**: 2026-04-06（历史快照）| **生成日期**: 2026-06-09（基于当前 HEAD）
**主题标签**: `cross-reference`, `template-design`, `vague-quantifier`, `manifest-discipline`, `examples-driven`

## 一、理解

### 1.1 仓库概览

upstash/context7 是一个 MCP（Model Context Protocol）服务，能够将最新版本的库文档实时注入 LLM 上下文，从而解决模型训练截止日期导致的文档过时问题。该仓库以超过 5.3 万 Star 跻身 GitHub 最受关注的 Claude Code 插件之列。

仓库同时支持多种 IDE/工具集成，具有以下三条部署路径：
- **plugins/claude/context7/** — Claude Code 插件（含 commands、agents、skills）
- **plugins/cursor/context7/** — Cursor IDE 插件（含 agents、skills）
- **skills/**、**packages/cli/、packages/pi/** — 独立 skill 包和 CLI 工具

核心能力链路为：用户输入库名 → `resolve-library-id` 解析 MCP 工具 → `get-library-docs` 拉取文档 → agent 组织输出。

### 1.2 架构剖析

目录结构（当前 HEAD）：

```
plugins/
  claude/context7/
    commands/docs.md                         # 用户入口命令
    agents/docs-researcher.md               # 查文档的 agent
    skills/context7-mcp/SKILL.md            # MCP 使用说明
  cursor/context7/
    agents/docs-researcher.md               # Cursor 平台同名 agent
    skills/context7-mcp/SKILL.md            # 同内容 skill（三重复制之一）
skills/
  context7-mcp/SKILL.md                     # 根目录 skill（三重复制之一）
  context7-cli/SKILL.md                     # CLI 使用说明
  find-docs/SKILL.md                        # 文档发现 skill
  references/                               # context7-cli 的引用子目录
packages/
  pi/skills/context7-docs/SKILL.md          # 新增（audit 后加入）
```

关键观察：三条部署路径共用同一份 `context7-mcp/SKILL.md`，但以物理文件三份分散存储，而非通过符号链接或构建时生成的单一来源管理。

### 1.3 设计思路 / 方法论

context7 的设计哲学是"文档即上下文"：把实时抓取的库文档变成 Claude 的工作记忆，而非依赖训练数据中已过时的快照。这一思路体现在以下具体选择上：

1. **MCP 作为文档传递通道**：`mcp.context7.com/mcp` 提供外部 HTTP MCP 端点，两个工具（`resolve-library-id`、`get-library-docs`）各司其职，分离"库名解析"与"内容获取"两个关注点。
2. **SKILL.md 作为使用手册**：`skills/context7-mcp/SKILL.md` 得分 94/100，详细描述工具签名、调用时机和返回格式，是项目 NL 质量最高的文件。
3. **插件平台无关性**：相同的 skill 内容被封装进 Claude 和 Cursor 两套插件结构，体现了跨平台复用意图——但实现方式是文件复制，而非真正的共享机制。

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名称**：多平台插件三明治结构（Multi-Platform Plugin Sandwich）

结构定义：
```
[平台无关 skill layer]  ← 核心知识，理论上单一来源
        ↓ 复制
[平台专属 agent layer]  ← 适配平台 prompt 风格
        ↓ 调用
[平台专属 command layer] ← 暴露给用户的入口
```

context7 同时实现了这一结构的 Claude 版和 Cursor 版，但用文件复制代替了"单一来源"的理想状态。

### 2.2 适用场景

该模式适用于：
- 一个核心工具/服务需要在多个 AI 平台（Claude Code、Cursor、Windsurf 等）同时提供 NL 接口时
- skill 内容稳定、平台适配逻辑（prompt 格式、model 指令）是主要差异点时
- 团队规模小，尚未引入构建工具来管理 DRY（Don't Repeat Yourself）

不适用场景：
- skill 内容频繁迭代——三份文件同步更新成本极高，极易产生漂移

### 2.3 与其他架构对比

| 维度 | context7（文件复制） | 符号链接方案 | 模板生成方案 |
|------|-------------------|-----------|-----------| 
| 单一来源 | 否（3 份 byte-for-byte 相同）| 是 | 是 |
| Git 可追踪 | 所有文件独立 diff | 仅 target 有 diff | 仅 template 有 diff |
| CI 验证难度 | 需要 diff 三文件 | 无需 | 需要检查生成产物 |
| 新平台接入 | 再加一份拷贝 | 新增符号链接 | 新增平台变量 |
| 现有工具链成本 | 0 | 极低 | 中等 |

### 2.4 改进空间

1. **三重复制问题**：`plugins/claude/context7/skills/context7-mcp/SKILL.md`、`plugins/cursor/context7/skills/context7-mcp/SKILL.md`、`skills/context7-mcp/SKILL.md` 三文件当前仍 byte-for-byte 相同（`diff` 输出为空）。可引入 CI 检查：`diff plugins/claude/context7/skills/context7-mcp/SKILL.md skills/context7-mcp/SKILL.md || exit 1`，一旦出现漂移立即报错。
2. **Cursor agent 缺少 `model:` 声明**：`grep "^model:" plugins/cursor/context7/agents/docs-researcher.md` 返回空——当 Cursor 平台默认模型升级时，该 agent 行为将静默变更。
3. **commands/docs.md 缺少 `name:` 字段**：即使 audit 后添加了 `description:` 字段，`name:` 仍然缺失，导致该命令在 plugin manifest 中无法被正确标识。

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

整体 NL 得分：**82/100**

各文件明细：

| 文件 | 得分 | 主要扣分点 |
|------|------|----------|
| plugins/claude/context7/commands/docs.md | 46/100 | 缺少 `name` frontmatter（-25） |
| plugins/cursor/context7/agents/docs-researcher.md | 60/100 | 无 example（-15）、无 model（-5）、无输出格式（-10） |
| plugins/claude/context7/agents/docs-researcher.md | 65/100 | 无 example（-15）、无输出格式（-10） |
| plugins/claude/context7/skills/context7-mcp/SKILL.md | 94/100 | 模糊量词（-6） |
| plugins/cursor/context7/skills/context7-mcp/SKILL.md | 94/100 | 模糊量词（-6） |
| skills/context7-mcp/SKILL.md | 94/100 | 模糊量词（-6） |
| skills/find-docs/SKILL.md | 92/100 | 模糊量词（-8） |
| skills/context7-cli/SKILL.md | 96/100 | 模糊量词（-4） |
| plugins/claude/context7/.claude-plugin/plugin.json | 95/100 | 缺少 version 字段（-5） |

安全评级：**CLEAR**（中危发现 4 项，均为配置层面，无 Critical）

### 3.2 当时值得借鉴的模式

1. **SKILL.md 质量高**：`skills/context7-cli/SKILL.md` 得 96/100，接近满分；三份 `context7-mcp/SKILL.md` 均达 94/100。说明团队对 skill 文件格式有清晰认知，工具签名、调用示例、返回格式描述完整。
2. **关注点分离**：MCP 工具被拆分为两个原子操作（`resolve-library-id` + `get-library-docs`），而非合并为一个黑盒接口，便于 agent 做错误处理和局部重试。
3. **命令 argument-hint**：`docs.md` 中的 `argument-hint: <library> [query]` 为用户提供了清晰的调用语法提示，是良好 UX 写作实践。

### 3.3 当时的缺陷

1. **commands/docs.md 缺少 `name:` frontmatter**（-25 分，最重扣分项）：没有 `name:` 字段，命令无法被 plugin loader 正确识别和路由。
2. **commands/docs.md 未声明 `allowed-tools`**：该命令调用了 `resolve-library-id` 和 `get-library-docs` 两个 MCP 工具，但 frontmatter 中没有 `allowed-tools:` 声明，违反最小权限原则。
3. **两个 agent 均无 `<example>` 块**：`grep -c "<example>" plugins/claude/context7/agents/docs-researcher.md` → 0。没有示例，模型无法通过少样本学习校准输出风格。
4. **Cursor agent 缺少 `model:` 声明**：Claude agent 有 model 字段，Cursor 版没有——平台间漂移。
5. **5 个模糊量词在 docs-researcher agent 中**："concise"、"actionable"、"focused"、"relevant"、"appropriate"——全部是不可验证的形容词。
6. **三份 SKILL.md byte-for-byte 相同但独立存储**：维护时必须同时修改三处，任何一处遗漏即产生静默漂移。

### 3.4 当时的优化机会

1. 为 `docs.md` 补充 `name: docs` 和 `allowed-tools: [mcp__context7__resolve-library-id, mcp__context7__get-library-docs]`
2. 在两个 agent 中添加至少 2 个 `<example>` 块，展示典型查询和预期输出格式
3. 将三份 `context7-mcp/SKILL.md` 合并为单一来源，其余两处改为符号链接或构建时生成
4. 将模糊量词替换为可验证标准（例如"concise" → "不超过 3 句话"；"relevant" → "引用自该库官方 API 文档"）

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

以下为 Phase 3 验证（基于当前 HEAD）结果：

| 缺陷 | 2026-04-06 状态 | 2026-06-09 状态 | 是否修复 |
|------|---------------|---------------|---------|
| commands/docs.md 缺少 `name:` | 缺失 | 仍然缺失（`description:` 已添加，但 `name:` 依然不存在） | 未修复 |
| commands/docs.md 缺少 `allowed-tools` | 缺失 | 未验证到新增 | 未修复 |
| agent 无 `<example>` 块 | 0 个 | `grep -c "<example>"` → 0 | 未修复 |
| Cursor agent 缺少 `model:` | 缺失 | `grep "^model:"` → 空 | 未修复 |
| 三份 SKILL.md 三重复制 | 存在 | `diff` 输出仍为空（三文件仍 byte-for-byte 相同） | 未修复 |
| context7-mcp SKILL.md 模糊量词 | "better"(L30,L55)、"closest"(L39) | "Exact or closest name match"、"Higher benchmark scores indicate better documentation quality"、"relevant code examples"、"better results" 依然存在 | 未修复 |

**结论**：audit 后约两个月，6 个核心缺陷均未修复。团队对 NL artifact 质量的关注度低于功能代码。

### 4.2 架构演进

audit 后新增了 `packages/pi/skills/context7-docs/SKILL.md`，说明项目在 packages/ 层新增了第四个平台接入点（pi）。这一变化：
- 进一步加重了三重复制问题（现在是四重，若 pi 的 SKILL.md 内容趋同）
- 表明"插件三明治"架构正在向更多平台扩张，但单一来源问题从未被系统性解决

### 4.3 新增的可学习模式

`packages/pi/skills/context7-docs/SKILL.md` 的出现揭示了一个可学习教训：**当新平台接入时，如果没有单一来源机制，每次接入都会增加一份潜在漂移的副本**。这是技术债务的典型"滚雪球"形式——每一次扩张都加重而非减轻维护负担。

## 五、校准

### 5.1 我已经在做对的

1. **agents 均有 `name:` 字段**：我的 echo-sleuth-for-claude 中 5 个 agent 文件全部有正确的 name frontmatter，而 context7 的 commands/docs.md 在经历两个月后 `name:` 字段仍然缺失。这说明我在 manifest 纪律上做得比 53K Star 的项目更好。
2. **无 Read-only/Write 权限错配**：echo-sleuth 的 agent 没有权限声明不一致的问题，这是一项被 context7 审计发现却未修复的类似问题的镜像对比。
3. **架构上明确分离 commands 和 agents**：echo-sleuth 有 8 个 commands 和 5 个 agents 的明确分工，与 context7 的设计思路一致。

### 5.2 挑战 / 验证

1. **我的 commands 同样缺少 `allowed-tools`**：echo-sleuth 的 `lessons.md`、`recall.md`、`recap.md`、`timeline.md` 四个 commands 缺少 `allowed-tools:` 声明——这与 context7 的 `docs.md` 犯了完全相同的错误。
2. **我的 commands 同样缺少 `<example>` 块**：echo-sleuth 8 个 commands 均无示例，与 context7 两个 agent 无示例的问题同源——都是"写了功能描述，但没写使用示范"。
3. **模糊量词问题我更严重**：echo-sleuth 有 12 个模糊量词实例；claude-for-legal 有 218 个。context7 经过 audit 后仍有若干模糊量词，但数量远少于我的两个仓库。

## 六、行动

### 6.1 自查动作

针对 echo-sleuth-for-claude，按优先级排列：

1. **立即（P0）**：为 `lessons.md`、`recall.md`、`recap.md`、`timeline.md` 补充 `allowed-tools:` frontmatter。这是与 context7 最大 bug（commands/docs.md 缺 `name`）同类的 manifest 纪律问题，修复成本极低（每文件加 1-2 行）。
2. **本周（P1）**：为 8 个 commands 各添加至少 1 个 `<example>` 块，展示典型输入和预期输出。context7 两个月未修复此问题，说明"等有时间再加"的拖延极易变成永久欠债。
3. **下周（P2）**：扫描 echo-sleuth 的 12 个模糊量词实例，逐一替换为可验证标准（例如将 "comprehensive" 替换为具体字段列表）。

### 6.2 灵感 → 实施路径

**从 context7 SKILL.md（94-96/100）学习 skill 写作**：

context7 的 `skills/context7-cli/SKILL.md`（96/100）之所以接近满分，是因为它做到了：
- 明确的工具签名（输入参数、类型、可选/必填）
- 每个工具有独立的调用示例
- 返回格式有结构化描述

实施路径：打开 echo-sleuth 中得分最低的 skill 文件，对照这三个标准逐一补充缺失内容，不要全部重写，只补缺口。

**从 context7 的教训学习"单一来源"管理**：

context7 的三重复制是已知问题但两个月未解决。如果 echo-sleuth 将来需要支持多平台，在第一天就引入符号链接或 CI diff 检查，而非先复制后修复。

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

upstash/context7 与我的仓库中，**MarkQWu/echo-sleuth-for-claude** 对齐度最高：
- 都是 Claude Code 插件，使用 MCP/skills 架构
- 都有 commands + agents + skills 的三层结构
- 都服务于"扩展 Claude 能力"的核心目标（context7 扩展文档访问；echo-sleuth 扩展对话历史分析）

MarkQWu/drama-workshop-skills 和 MarkQWu/claude-for-legal 与 context7 的目的对齐度较低——前者聚焦创作领域特定工作流，后者聚焦专业垂直领域，而非通用能力扩展插件。

### 8.2 在我的项目里复现的同类问题

| context7 缺陷 | 我的仓库复现情况 |
|-------------|--------------|
| commands/docs.md 缺少 `allowed-tools` | echo-sleuth：lessons.md、recall.md、recap.md、timeline.md 四个命令均缺少 `allowed-tools` |
| agents 无 `<example>` 块 | echo-sleuth：8 个 commands 无 `<example>` 块；claude-for-legal：skills 无 `<example>` 块 |
| 模糊量词（"concise"、"relevant" 等） | echo-sleuth：12 处；claude-for-legal：218 处，均为同类问题 |
| plugin manifest 不完整 | drama-workshop-skills：reference .md 文件无 frontmatter |

**最高优先级同构问题**：`allowed-tools` 缺失。这是 manifest 纪律最基础的要求，context7 在 audit 两个月后仍未修复，印证了该问题"不紧急但很重要"的特性——越拖越容易忽视。

### 8.3 别人的更优方案

1. **SKILL.md 写作质量**：context7 的 `skills/context7-cli/SKILL.md`（96/100）和三份 `context7-mcp/SKILL.md`（94/100）都优于 echo-sleuth 中模糊量词达 12 处的 skill 文件。他们的 skill 文件有明确的工具签名和调用示例，我的 skill 文件在这方面写得不够具体。
2. **关注点分离的工具设计**：将 MCP 能力拆分为 `resolve-library-id`（解析）和 `get-library-docs`（获取）两个原子工具，比把所有功能塞入一个工具的做法更易于 agent 做条件处理。echo-sleuth 的工具边界划分可以参考这一思路。
3. **argument-hint 的使用**：context7 的 `docs.md` 使用 `argument-hint: <library> [query]` 帮助用户理解命令语法。echo-sleuth 的部分 commands 没有此字段。

### 8.4 反向：我的项目做得比他们好的地方

1. **agents 的 `name:` frontmatter 完整**：echo-sleuth 的 5 个 agent 文件全部有正确的 name 字段。context7 的 `commands/docs.md` 经过两个月的 audit 后 `name:` 仍然缺失——这是 25 分的最重扣分项，我完全避免了。
2. **无跨平台漂移**：echo-sleuth 是单平台插件（Claude Code），不存在 Cursor/Claude 之间的 agent drift 问题。context7 的 Cursor agent 缺少 `model:` 声明，是多平台插件维护复杂度的直接成本，我的单平台设计规避了这一风险。
3. **无三重复制**：echo-sleuth 的每个 skill 文件只有一份，没有 context7 这种三份 byte-for-byte 相同文件并存的情况。

## 八、术语表

| 术语 | 解释 |
|------|------|
| **frontmatter** | Markdown 文件开头的 YAML 元数据块，以 `---` 包裹。在 Claude Code NL artifacts 中，frontmatter 声明 `name`、`description`、`model`、`allowed-tools` 等关键字段。缺少必填字段会导致严重扣分（如缺 `name` 扣 25 分）。 |
| **SKILL.md** | Claude Code 插件中用于描述工具或能力的知识文件。遵循固定格式，包含工具签名、调用时机、示例和返回格式描述。NLPM 对 SKILL.md 的评分上限为 100 分，context7 的 skill 文件得分介于 92-96 分之间。 |
| **MCP** | Model Context Protocol，Anthropic 定义的 LLM 工具调用标准协议。context7 通过外部 HTTP MCP 端点（`https://mcp.context7.com/mcp`）提供 `resolve-library-id` 和 `get-library-docs` 两个工具，使 Claude 能够实时获取库文档。 |
| **allowed-tools** | command 或 agent frontmatter 中声明该 NL artifact 可以调用的工具列表。遵循最小权限原则——未在 `allowed-tools` 中声明的工具不应被调用。缺少此字段是 context7 `docs.md` 和 echo-sleuth 多个 commands 共有的 manifest 纪律问题。 |
| **vague quantifier（模糊量词）** | NL artifacts 中无法客观验证的形容词或副词，如 "concise"、"relevant"、"better"、"appropriate"。NLPM 评分规则对此扣分，因为这类词语无法指导模型产出可预期的输出。替换方法是用具体、可测量的标准（如"不超过 3 句"、"引用自 API 文档的字段名"）代替主观描述。 |
