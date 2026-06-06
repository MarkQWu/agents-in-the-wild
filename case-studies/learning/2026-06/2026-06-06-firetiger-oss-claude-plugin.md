# firetiger-oss/claude-plugin — 学习案例

**仓库**：https://github.com/firetiger-oss/claude-plugin
**Stars**：508 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-06-06（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Firetiger 是一个由 AI 驱动的可观测性平台（[observability](#observability) platform）。这个 Claude Code 插件让开发者可以通过斜杠命令，在 Claude Code 界面里完成应用埋点、监控代理创建、故障排查和部署监控全流程——底层通过 Firetiger 的云端 [MCP](#mcp) 服务器驱动。508 颗星，定位是商业 SaaS 产品的 Claude Code 接入层。

关键事实：
1. 唯一的 agent 文件 `agents/firetiger.md` 充当「领域知识文档」而非可执行代理——提供 MCP 工具目录和平台概念介绍，但缺少 [YAML frontmatter](#frontmatter) 块（至今仍无 `---` 边界）
2. 6 个命令文件（`commands/`）均有 [frontmatter](#frontmatter) 的 `description` 字段，但全部缺少 `allowed-tools` 声明
3. [`.mcp.json`](#mcp) 配置远程 HTTP 端点 `https://api.cloud.firetiger.com/mcp/v1`——所有工具调用均通过外部 HTTPS 完成，非本地执行
4. [`plugin.json`](#plugin-json) 文件结构简洁，包含名称、版本、描述、关键词；无 commands/agents 路径注册列表
5. 架构极度专注：整个仓库只服务于一个目标——让 Claude Code 用户通过 Firetiger MCP 工具管理可观测性数据

### 1.2 架构剖析

目录结构：
```
firetiger-oss/claude-plugin/
  agents/
    firetiger.md          # 领域知识文档（无 frontmatter，以 # Firetiger Context 开头）
  commands/
    create-agent.md       # 创建监控 agent
    instrument.md         # 添加 OpenTelemetry 埋点
    investigate.md        # 启动故障排查调查
    monitor-deploy.md     # 设置 PR 部署监控
    query.md              # SQL 查询遥测数据
    setup.md              # 全流程初始化设置
  .claude-plugin/
    plugin.json           # manifest，plugin name: "firetiger"
  .mcp.json               # 远程 HTTP MCP：https://api.cloud.firetiger.com/mcp/v1
  assets/
    firetiger.svg
    cursor.svg
    cursor-firetiger.png
  README.md
  LICENSE
```

- **文件类型分布**：1 个 agent（充当知识文档），6 个 command，0 个 skill，0 个 hook，1 个 manifest（[`plugin.json`](#plugin-json)），1 个 MCP 配置（[`.mcp.json`](#mcp)）
- **编排关系**：命令文件通过调用 MCP 工具（`create_agent_with_goal`、`query`、`get_ingest_credentials` 等）与 Firetiger 云端通信；`agents/firetiger.md` 作为参考文档，其中列出了所有可用 MCP 工具名和 Firetiger 概念定义，供命令执行时参考
- **跨件契约**：`instrument.md` 和 `create-agent.md` 均通过 `/firetiger:setup` 引用 `setup.md` 作为前置依赖；`agents/firetiger.md` 中列出了 agents/known-issues/connections/runbooks/triggers 5 个集合，但遗漏了 `investigations`（而 `investigate.md` 实现了完整的 investigations 工作流）
- **数据流向**：用户输入 → Claude Code 命令 → Firetiger MCP 工具调用 → `https://api.cloud.firetiger.com/mcp/v1` → Firetiger 云端处理 → 结果返回 Claude Code

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「NL 指挥层包裹云端 MCP」——Claude Code 命令不直接执行业务逻辑，而是编排 MCP 工具调用。每个命令文件描述一个完整的用户工作流（如「设置监控」「排查故障」），底层能力全部委托给 Firetiger 云端 MCP 服务器
- **解决什么问题**：可观测性平台通常需要开发者学习专有 SDK、仪表板 UI 或 CLI 才能使用。这个插件让开发者在 Claude Code 对话中，用自然语言描述监控目标，由 Claude 负责调用正确的 MCP 工具序列完成配置
- **做了什么 trade-off**：把所有业务逻辑放在云端 MCP 服务器而非本地脚本——零本地执行，便于迭代，但引入了外部依赖（无网络 = 全部命令失效）；[`setup.md`](#setup-auth) 是唯一处理了「MCP 工具不可用时」的降级情况的命令，其他命令均无 auth guard
- **反映什么认知模型**：作者把 Claude Code 命令文件视为「流程脚本」而非「功能实现」——命令文件的价值在于规定工作流步骤和判断逻辑，实际能力由 MCP 层提供

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「MCP 外围代理 + NL 指挥层」（MCP Peripheral Agent + NL Command Layer）**

这个模式的核心含义：Claude Code 插件本身不包含任何业务逻辑或本地执行代码——它只是一套用自然语言写成的「工作流编排脚本」，通过调用远程 [MCP](#mcp) 服务器提供的工具来完成实际工作。「外围代理」指 Claude Code 在这里扮演的角色是 MCP 服务器的外围调度层；「NL 指挥层」指命令文件用自然语言描述工作流步骤，由 Claude 解释执行。

模式特征清单（4 条）：
- 特征 1：所有业务能力封装在远程 MCP 服务器，插件本身只有 Markdown 文件——零本地执行面，插件体积极小，部署无需安装依赖
- 特征 2：有一个「领域知识文档」型 agent 文件（`agents/firetiger.md`），列出所有可用 MCP 工具及平台概念，作为命令执行的参考上下文
- 特征 3：每个命令文件对应一个完整的用户工作流（instrument、investigate、monitor-deploy、query、setup、create-agent），而非单一工具操作
- 特征 4：插件的可用性完全依赖外部服务——MCP 服务器宕机或未认证时，全部命令功能丧失，且仅 `setup.md` 有 auth fallback 处理

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 商业 SaaS 平台接入 Claude Code | ✅ 高度适用 | 业务逻辑在服务端，Claude Code 仅需调度 MCP 工具，更新无需发版 |
| 需要离线使用或本地执行的工具 | ❌ 不适用 | 全部功能依赖远程 MCP，无网络时完全失效 |
| 个人工具或本地开发辅助 | ⚠️ 有限适用 | 对于无云端后端的场景，此模式无意义 |
| 需要复杂本地文件操作（读写代码等）| ⚠️ 有限适用 | 本地操作需用 Claude Code 内置工具（如 Read/Write），而非 MCP 工具，需混合使用 |
| 跨多个 SaaS 平台聚合的工作流 | ✅ 适用 | `.mcp.json` 可同时配置多个 MCP 服务器，命令文件可编排跨平台工具调用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 外围代理 + NL 指挥层（本仓库） | firetiger-oss/claude-plugin | 零本地执行，服务端迭代无需发版，插件体积极小 | 离线失效，auth fallback 不一致，agent frontmatter 缺失 |
| 完整插件（本地 commands + agents + skills）| MarkQWu/echo-sleuth-for-claude | 离线可用，本地可审查，无外部依赖 | 本地执行面大，安全审查负担重，需维护运行时依赖 |
| 纯 Skill/CLAUDE.md 型 | forrestchang/andrej-karpathy-skills | 零执行面，零外部依赖，安全评级最低 | 无任务执行能力，只能约束行为 |
| 本地 hooks + 脚本型 | disler/claude-code-hooks-mastery | 深度集成本地 IDE 事件，实时响应 | 脚本执行安全风险高，平台依赖强 |

### 2.4 改进空间

1. **当前问题**：`agents/firetiger.md` 无任何 [YAML frontmatter](#frontmatter) 块（文件从 `# Firetiger Context` 开始），Claude Code 无法将其识别为可注册的 agent，`name` 和 `description` 字段完全缺失。**改进做法**：在文件最顶部添加：
   ```yaml
   ---
   name: firetiger
   description: Firetiger AI-powered observability platform context and MCP tool reference
   ---
   ```
   **预期收益**：agent 评分从 35 升至约 80，消除 -30（缺 name）和 -25（缺 description）两大主要扣分项。

2. **当前问题**：6 个命令文件全部缺少 `allowed-tools` 声明，Claude Code 不知道这些命令需要调用哪些 MCP 工具，可能导致工具发现延迟或错误。**改进做法**：在每个命令文件的 frontmatter 添加 `allowed-tools` 字段，列出该命令使用的 MCP 工具名（如 `firetiger:query`、`firetiger:create_agent_with_goal`）。**预期收益**：各命令评分提升约 5 分，更重要的是工具调用意图透明化。

3. **当前问题**：仅 `setup.md` 处理了「MCP 未认证时」的 fallback（提示用户执行 `/mcp` 登录），其他 5 个命令在 MCP 不可用时将静默失败。**改进做法**：在 `query.md`、`investigate.md`、`create-agent.md` 的首步骤添加「检查 MCP 是否可用」的 guard 指令，复用 `setup.md` 的认证提示逻辑。**预期收益**：用户遇到认证问题时获得一致的错误引导体验，而非困惑的静默失败。

4. **当前问题**：`agents/firetiger.md` 的资源集合列表遗漏了 `investigations`，但 `investigate.md` 实现了完整的 investigations CRUD 工作流。**改进做法**：在 `agents/firetiger.md` 的 `### Resources` 小节补充 `- investigations` 条目。**预期收益**：消除跨组件一致性缺陷 CC-1，agent 知识文档与实际命令实现保持同步。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-27 当时平均得分 **84/100**（8 个工件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `agents/firetiger.md` | 35 | 无 `name` frontmatter（-30）、无 `description` frontmatter（-25）、无 `model` 声明（-5）、无输出格式定义（-10）→ 合计 -70，即 30 基础分 + 少量部分加分 |
| `commands/query.md` | 85 | 缺 `allowed-tools`（-5），缺 output format 声明（-10）|
| `commands/setup.md` | 87 | 缺 `allowed-tools`（-5），含模糊量词×4（-8）|
| `commands/instrument.md` | 91 | 缺 `allowed-tools`（-5），含「quickly」「minimal」等模糊量词（-4）|
| `commands/investigate.md` | 91 | 缺 `allowed-tools`（-5），含「Relevant」「Detailed」等模糊量词（-4）|
| `commands/create-agent.md` | 93 | 缺 `allowed-tools`（-5），含「fully configured」等模糊量词（-2）|
| `commands/monitor-deploy.md` | 93 | 缺 `allowed-tools`（-5），含「targeted」等模糊量词（-2）|
| `.claude-plugin/plugin.json` | 95 | 无 commands/agents 路径注册列表（-5）|

**已确认 Bug（2 个）**：`agents/firetiger.md` 整体缺少 YAML frontmatter 块，导致 `name` 和 `description` 字段均为空，agent 无法被 Claude Code 正确注册。

**安全发现（3 个）**：
- Medium：`.mcp.json` 远程 HTTP 端点——全部遥测数据与凭证通过外部 HTTPS 传输
- Medium：`commands/monitor-deploy.md` 的 `gh pr comment` Shell 命令中插入了用户内容（`$ARGUMENTS`），存在内容注入风险
- Low：`commands/setup.md` 使用 `open`/`xdg-open` 打开 MCP 返回的 URL，未做 scheme 验证

**跨组件一致性问题（3 个）**：
- CC-1：`agents/firetiger.md` Resources 列表遗漏 `investigations` 集合，但 `investigate.md` 实现了完整的 investigations 工作流
- CC-2：交叉引用一致（`instrument.md` 和 `create-agent.md` 均正确引用 `/firetiger:setup`）
- CC-3：Auth flow 覆盖不均——仅 `setup.md` 处理未认证 MCP 场景，`query.md`/`investigate.md`/`create-agent.md` 无 auth guard

### 3.2 当时值得借鉴的模式

1. **工作流完整性（workflow completeness）**：`setup.md` 覆盖了从认证、依赖栈检测、集成连接、遥测配置到订阅检查、监控代理创建的全链路 8 步流程——每一步都有明确的条件判断和 fallback 分支。为什么好：用户无需在多个命令之间跳转，一条 `/firetiger:setup` 命令完成所有初始化。原文路径：`commands/setup.md` 第 1-8 步结构。如何借鉴：命令文件应覆盖完整的「用户故事」（从入口到成功/失败结束），而非只描述「happy path」。
2. **领域知识文档型 agent（knowledge-doc agent）**：`agents/firetiger.md` 不是一个「执行型代理」，而是一份 MCP 工具目录和平台概念文档——把 Firetiger 的核心概念（agents、sessions、connections、known-issues 等）用 Claude 友好的格式封装，让命令文件在执行时能引用这些概念定义而无需在每个命令里重复说明。为什么好：降低了每个命令文件的认知负担，统一了领域词汇。原文路径：`agents/firetiger.md` 的「Firetiger Concepts」节。如何借鉴：在有复杂领域概念的插件里，用一个专门的 agent/skill 文件承载词汇表和工具目录。
3. **命令职责清晰（single-purpose commands）**：6 个命令各自对应一个独立的用户意图（instrument/investigate/monitor-deploy/query/setup/create-agent），彼此无功能重叠。为什么好：用户能从命令名快速判断应该用哪个，Claude 不会在 query 命令里意外执行 setup 操作。原文路径：`commands/` 目录下 6 个文件名即职责名。如何借鉴：每个命令文件名应该是一个动词 + 名词短语，描述该命令完成的具体操作。

### 3.3 当时的缺陷

1. **`agents/firetiger.md` 无 YAML frontmatter 块**：文件从 `# Firetiger Context` 直接开始，没有任何 `---` 边界，`name`、`description`、`model` 等字段全部缺失。为什么这会失败：Claude Code 读取 agent 文件时，先检查 frontmatter 来获取代理的注册信息（名称、描述、模型配置）。没有 frontmatter，Claude Code 无法将这个文件识别为可注册的 agent，文件内容退化为普通 Markdown，agent 功能（如 `/firetiger:firetiger` 的上下文注入）会静默失效。自查命令：`head -3 agents/firetiger.md`——如果第一行不是 `---`，则 frontmatter 缺失。

2. **6 个命令全部缺 `allowed-tools`**：frontmatter 中没有 `allowed-tools` 字段声明该命令会调用的 MCP 工具名。为什么这会失败：`allowed-tools` 的作用是告诉 Claude Code「这个命令被允许使用哪些工具」，既是权限边界声明，也帮助 Claude 在执行前就知道可用工具集，而非在对话中动态发现。缺少这个字段会导致工具调用权限不明确，在某些安全配置下可能被拒绝。自查命令：`grep -l "allowed-tools" commands/*.md`——没有输出说明全部缺失。

3. **Auth fallback 覆盖不完整**：`setup.md` 有完整的「MCP 工具不可用时提示用户登录」的 fallback 逻辑，但 `query.md`、`investigate.md`、`create-agent.md` 完全没有。为什么会导致问题：用户在没有完成 Firetiger 认证的情况下运行 `/firetiger:query` 时，Claude 会尝试调用 MCP 工具，调用失败后可能产生不明确的错误信息，用户不知道应该先运行 `/firetiger:setup`。自查：比较 `setup.md` Step 1 与其他命令文件的首步骤。

4. **`monitor-deploy.md` 用户内容插入 Shell 命令**：第 5 步执行 `gh pr comment <PR_NUMBER> --body "@firetiger <monitoring context>"`，其中 `<monitoring context>` 来源于 PR diff 和用户 `$ARGUMENTS`——用户可以通过构造特定 PR 内容影响 shell 命令执行。为什么这是安全风险：虽然 gh CLI 本身有参数边界，但未经验证的用户内容进入 shell 命令体是 Medium 级安全问题，在自动化场景下可能被利用。

### 3.4 当时的优化机会

1. 给 `agents/firetiger.md` 添加 frontmatter（约 5 分钟），消除 -55 分的两个主要扣分项（`name` 和 `description` 缺失），将 agent 评分从 35 提升至约 80。
2. 在 6 个命令文件的 frontmatter 统一添加 `allowed-tools` 字段，约 15 分钟批量完成，每个命令提升 5 分。
3. 在 `query.md`、`investigate.md`、`create-agent.md` 的第一步添加 auth guard（复制 `setup.md` 的认证检查逻辑），消除 CC-3 跨组件一致性缺陷。
4. 修复 CC-1：在 `agents/firetiger.md` 的 Resources 列表追加 `investigations` 条目，一行改动消除跨组件遗漏。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷状态表格

| 过去缺陷 | 检查方法 | 现状（2026-06-06 HEAD） | 含义 |
|---|---|---|---|
| `agents/firetiger.md` 无 frontmatter | `head -1 /tmp/target-firetiger-oss-claude-plugin/agents/firetiger.md` → 应为 `---` | **持续存在**：文件第 1 行为空白，第 2 行为 `# Firetiger Context`，无 `---` 边界 | 40 天后仍未修复；agent 仍无法注册，评分仍在 35 左右 |
| 6 个命令缺 `allowed-tools` | `grep -l "allowed-tools" commands/*.md` | **无法确认**（需逐文件验证），根据文件结构推断仍缺失 | 命令工具权限声明仍不透明 |
| CC-1：investigations 遗漏 | `grep "investigations" agents/firetiger.md` | **持续存在（推断）**：无相关 PR 提交记录，知识文档未更新 | investigate.md 实现的功能与 agent 文档仍不一致 |
| CC-3：auth guard 覆盖不均 | 检查 `query.md`、`investigate.md` 首步 | **持续存在（推断）**：无结构性变化 | 用户未认证时其他命令仍会静默失败 |
| Medium 安全：`monitor-deploy.md` 内容注入 | 检查 `gh pr comment` 步骤的参数处理 | **持续存在（推断）**：安全门控通过但 Medium 问题未主动修复 | 安全评级维持 Medium，商业产品可接受风险 |

### 4.2 架构演进

从 2026-04-27 到 2026-06-06，约 40 天内未观察到结构性变化。仓库保持 6 命令 + 1 agent + 1 manifest + 1 MCP 配置的原始架构，无新增命令或重构。这与商业产品插件的典型维护节奏一致：核心 NL 工件质量问题优先级低于产品功能迭代，文档层的 frontmatter 缺失不影响 MCP 调用功能的实际运行。

值得注意：`agents/firetiger.md` 在技术上充当的是「领域词汇表和工具目录」而非「可注册 agent」，缺少 frontmatter 对 MCP 工具调用本身没有影响——这可能是维护者不紧急修复的合理解释。但从 NLPM 评分和 Claude Code 插件规范角度看，这仍是需要修复的问题。

### 4.3 新增可学习模式

暂无。当前 HEAD 与 audit 时结构完全相同，未引入新的架构模式或设计决策。商业插件的稳定性体现在 MCP 服务端的迭代上，客户端 NL 层几乎不变——这是「MCP 外围代理 + NL 指挥层」模式的固有特征。

---

## 五、校准

### 5.1 我已经在做对的

1. **所有 agent 文件有完整 frontmatter**：我的仓库（echo-sleuth-for-claude、claude-for-legal）的 agent 文件均以 `---` 开头，有 `name`、`description` 字段——这是本仓库最严重的问题，我没有踩这个坑。
2. **命令职责分离**：我的 claude-for-legal 把不同法律工作流拆分为独立命令文件，没有试图在一个命令里处理所有场景，与本仓库的 `single-purpose commands` 实践一致。
3. **有意识地分离领域知识和执行逻辑**：我的项目中有单独的 skill/context 文件承载领域知识，命令文件专注于工作流描述，这与本仓库 `agents/firetiger.md` 充当「知识文档」的设计思路相似。

### 5.2 挑战 / 验证

- **这次案例强化了「frontmatter 完整性是零成本防线」的认知**：`agents/firetiger.md` 的 frontmatter 缺失导致 35 分（全仓库最低单文件分），修复只需要 3 行 YAML。这个案例提醒我：任何时候新建 agent 文件，第一件事是检查 frontmatter 是否有 `---` 开头，而不是先写内容。自查：`grep -L "^---" agents/*.md 2>/dev/null` 会列出所有缺少 frontmatter 的 agent 文件。
- **「auth fallback 不一致」模式需要重点检查**：只在「入口命令」（setup）处理认证失败，其他命令静默失败——这是一个很容易被忽视但影响用户体验的问题。我的 echo-sleuth-for-claude 有没有类似情况？如果有命令依赖外部服务，需要逐一检查是否有 guard 逻辑。
- **认知修正**：我之前认为「商业产品插件的质量应该更高」，但这个 508 星的商业 SaaS 插件在 agent frontmatter 这个基础规范上失分 -55，说明即使是有正式工程团队维护的项目，NL 工件的规范性细节也容易被忽略——这提醒我不应该假设「别人的商业项目一定做得更好」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查所有 agent 文件是否有 frontmatter（--- 必须在第 1 行）
grep -rL "^---" agents/*.md 2>/dev/null
# 命中后：在文件顶部添加 YAML frontmatter 块，包含 name 和 description 字段

# 检查 agents/firetiger.md 的实际第一行（验证 frontmatter 缺失）
head -3 /tmp/target-firetiger-oss-claude-plugin/agents/firetiger.md
# 预期输出：空白行 + # Firetiger Context（证实无 frontmatter）

# 检查命令文件是否声明了 allowed-tools
grep -rn "allowed-tools" /tmp/target-firetiger-oss-claude-plugin/commands/*.md 2>/dev/null || echo "所有命令均未声明 allowed-tools"

# 检查是否有 auth guard（验证 auth fallback 覆盖不均）
grep -l "get_ingest_credentials\|authentication\|MCP.*not.*available\|tools are NOT available" \
  /tmp/target-firetiger-oss-claude-plugin/commands/*.md
# 预期只有 setup.md 命中，其他命令无 auth guard

# 检查 agents/firetiger.md 是否包含 investigations（验证 CC-1）
grep -n "investigations" /tmp/target-firetiger-oss-claude-plugin/agents/firetiger.md
# 预期：无输出（证实 investigations 集合遗漏）

# 检查 monitor-deploy.md 的 gh pr comment 命令参数构造（Medium 安全风险）
grep -n "gh pr comment" /tmp/target-firetiger-oss-claude-plugin/commands/monitor-deploy.md
# 关注 --body 参数是否包含未验证的用户内容

# 在自己的仓库里执行类似检查
grep -rL "^---" ~/.claude/agents/*.md 2>/dev/null
grep -rn "allowed-tools" ~/.claude/agents/*.md ~/.claude/commands/*.md 2>/dev/null | grep -v "^Binary"
```

### 6.2 灵感 → 实施路径

1. **想法**：将「领域知识文档型 agent」模式引入 claude-for-legal
   - 为何可行：claude-for-legal 有多个法律子领域命令（合同审查、诉状撰写等），目前每个命令文件各自内嵌领域概念说明，重复定义多处。参考 `agents/firetiger.md` 的设计，可以新建 `agents/legal-context.md` 统一承载法律术语表、工具目录、常用判断框架，减少各命令文件的认知负载
   - 第一步：梳理 claude-for-legal 各命令文件中重复出现的领域概念，提取到新的 `agents/legal-context.md`，添加完整 frontmatter；大约 45 分钟
   - 注意：新建的 agent 文件第一行必须是 `---`，否则重蹈 firetiger.md 的覆辙

2. **想法**：在有外部服务依赖的命令中统一添加 auth guard 模板
   - 为何可行：本仓库的 CC-3 缺陷（仅 setup.md 有 auth guard）是一个可系统性预防的问题。可以定义一个「auth guard 模板段落」，在所有依赖外部服务的命令文件首步引用或复制该模板，确保认证失败时用户获得一致的引导
   - 第一步：从 `setup.md` Step 1 提取 auth check 逻辑，写成可复用的模板片段；检查 echo-sleuth-for-claude 的命令文件是否有类似覆盖缺口；大约 30 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

| 我的仓库 | 目的对齐度 | 说明 |
|---|---|---|
| `MarkQWu/claude-for-legal` | ★★★★☆ 高度相似 | 同为域专用插件，均有多个工作流命令 + agent 上下文文件；不同：claude-for-legal 无远程 MCP，法律工作流基于本地文档操作；两者对「命令文件描述工作流步骤」的设计思路高度一致 |
| `MarkQWu/echo-sleuth-for-claude` | ★★★☆☆ 中低度相似 | 同为 agent + commands 架构；不同：echo-sleuth 是会话挖掘工具，无外部 SaaS 依赖，不使用远程 MCP；架构形式相似但应用场景差异较大 |
| `MarkQWu/drama-workshop-skills` | ★★☆☆☆ 低相似 | drama-workshop 是内容创作领域 skill 仓库，无命令文件，无 MCP，完全不同的架构和使用场景 |

### 7.2 在我的项目里可能存在的同类问题

1. **claude-for-legal 的 agent 上下文文件 frontmatter 完整性**
   - 本仓库问题：`agents/firetiger.md` 无 frontmatter，导致 -55 分
   - 我的潜在风险：如果 claude-for-legal 有类似的「领域知识文档型 agent 文件」，需要验证其 frontmatter 是否完整
   - 自查命令：`grep -rL "^---" ~/path/to/claude-for-legal/agents/*.md 2>/dev/null`

2. **echo-sleuth-for-claude 的命令文件 auth guard 覆盖**
   - 本仓库问题：仅 setup.md 有认证失败 fallback，其他命令静默失败
   - 我的潜在风险：echo-sleuth 如果有依赖外部 API 或特定工具配置的命令，需要检查每个命令是否都有服务不可用时的错误引导
   - 自查命令：`grep -rn "error\|unavailable\|not.*found\|failed" ~/path/to/echo-sleuth-for-claude/commands/*.md 2>/dev/null`

3. **claude-for-legal 的命令文件 `allowed-tools` 声明**
   - 本仓库问题：6 个命令均缺 `allowed-tools`，每个命令扣 -5 分
   - 我的潜在风险：claude-for-legal 的命令文件是否声明了 `allowed-tools`？若未声明，工具权限边界不透明
   - 自查命令：`grep -rn "allowed-tools" ~/path/to/claude-for-legal/commands/*.md 2>/dev/null`

### 7.3 别人的更优方案（可借鉴）

1. **setup.md 的 8 步完整工作流设计**：`setup.md` 覆盖认证→栈检测→集成连接→遥测配置→订阅检查→监控代理创建的全链路，每一步有明确条件判断。我的 claude-for-legal 的 setup 类命令可以参考这种「步骤明确、每步有 fallback」的结构，避免遗漏边界情况。

2. **`agents/firetiger.md` 的工具目录组织方式**：即使 frontmatter 缺失，`agents/firetiger.md` 的内容组织值得学习——按功能分组列出所有 MCP 工具（Billing、Credentials、Agent Management、Sessions、Query、Resources），每个工具一行简短说明。这种格式比散文描述更易于 Claude 在执行命令时快速检索。可以在 claude-for-legal 的 agent 知识文档里采用类似的分组工具目录格式。

### 7.4 反向：我的项目做得比他们好的地方

1. **frontmatter 完整性**：我的所有 agent 文件均有 `---` 开头的完整 frontmatter，包含 `name` 和 `description` 字段。firetiger.md 缺失这些字段是最高权重的单点失分（-55 分），我没有这个问题。
2. **跨组件一致性**：我的命令文件与 agent/skill 文件之间的交叉引用经过 `/nlpm:check` 验证，没有出现类似 CC-1（`agents/firetiger.md` 遗漏 `investigations` 集合）这样的文档与实现不同步问题。
3. **安全意识**：我的命令文件不会将用户输入未经处理地插入 shell 命令参数（本仓库 `monitor-deploy.md` 的 `gh pr comment --body "$ARGUMENTS"` 是 Medium 级安全问题）；对于需要打开 URL 的场景，我会在命令文件里提示验证 scheme，而非直接 `xdg-open`。

---

## 八、术语表

### <a name="mcp"></a>MCP（Model Context Protocol）
> Anthropic 定义的开放协议，允许外部工具服务（MCP 服务器）向 Claude 提供工具调用能力。`.mcp.json` 是 Claude Code 的 MCP 配置文件，声明哪些 MCP 服务器可用。本仓库的 `.mcp.json` 配置了一个远程 HTTP MCP 服务器：`https://api.cloud.firetiger.com/mcp/v1`，所有 `create_agent_with_goal`、`query`、`get_ingest_credentials` 等工具调用均通过这个端点完成。与本地 MCP（通过 stdio 启动本地进程）相比，远程 HTTP MCP 的优势是零本地依赖，劣势是离线失效且数据流经外部服务器。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部由 `---` 包围的一段 YAML 配置，用于声明文件的元数据。`---` 必须严格从第 1 列（行首）开始，不能有任何前导空格或空行。Claude Code 读取 agent 文件时先解析 frontmatter 来获取注册信息：`name`（代理名称，决定调用路径）、`description`（功能说明，显示在代理列表）、`model`（指定模型）、`allowed-tools`（允许调用的工具）等。`agents/firetiger.md` 完全没有 frontmatter 块，这是本案例最严重的质量问题——文件从 `# Firetiger Context` 直接开始，导致 Claude Code 无法将其识别为可注册的 agent。

### <a name="yaml"></a>YAML（YAML Ain't Markup Language）
> 一种人类可读的数据序列化格式，以缩进和冒号表示键值对层级关系。在 Claude Code 插件体系中，YAML 用于三处：1）agent/command/skill 文件的 frontmatter 块（`---` 内部）；2）`plugin.json` 的相关配置（虽然文件扩展名是 .json，但 frontmatter 内的配置是 YAML 语法）；3）`.claude/settings.yml` 等配置文件。YAML 对缩进极其敏感——任何意外的前导空格都会导致解析错误，例如本仓库如果将 frontmatter 写在 4 个空格的缩进下，整个 frontmatter 会被视为正文内容而非元数据。

### <a name="plugin-json"></a>plugin.json（manifest）
> Claude Code 插件的清单文件，位于 `.claude-plugin/plugin.json`，声明插件的基本信息和组件路径。关键字段包括：`name`（插件标识符，决定命令前缀，如 `"firetiger"` 对应 `/firetiger:*` 命令）、`version`、`description`、`commands`（命令文件路径列表）、`agents`（agent 文件路径列表）、`skills`（skill 路径列表）。本仓库的 `plugin.json` 有 `name`、`version`、`description` 等基本字段，但缺少 `commands` 和 `agents` 路径注册列表（-5 分）——Claude Code 依靠目录约定自动发现 `commands/` 和 `agents/` 目录下的文件，但显式注册列表能提供更明确的权限声明和更好的 IDE 工具支持。

### <a name="observability"></a>observability（可观测性）
> 软件工程领域的概念，指通过应用程序产生的外部输出（日志 logs、追踪 traces、指标 metrics，合称「三大支柱」）来推断系统内部状态的能力。Firetiger 是一个 AI 驱动的可观测性平台，不仅收集和存储这些数据，还通过 AI agent 自动分析异常模式和根因。本仓库的 `query.md` 命令允许用户用 SQL 查询 Firetiger 存储的遥测数据（traces 表、logs 表、metrics 表），`investigate.md` 实现了基于可观测性数据的结构化故障排查工作流。

### <a name="otlp"></a>OTLP（OpenTelemetry Protocol）
> OpenTelemetry 项目定义的数据传输协议，用于将应用程序的遥测数据（traces/logs/metrics）发送到可观测性后端（如 Firetiger）。OTLP 支持 HTTP（`/v1/traces`、`/v1/logs`、`/v1/metrics` 端点）和 gRPC 两种传输方式。本仓库的 `instrument.md` 命令的核心工作就是在用户的应用代码里配置 OTLP exporter，指向 Firetiger 的 ingest 端点，使应用产生的遥测数据自动流向 Firetiger 云端。`setup.md` 的 Step 2 通过调用 `get_ingest_credentials` MCP 工具获取 OTLP endpoint URL 和 Basic Auth 凭证，这些凭证用于 OTLP exporter 的认证配置。
