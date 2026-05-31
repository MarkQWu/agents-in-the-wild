# codeaholicguy/ai-devkit — 学习案例

**仓库**：https://github.com/codeaholicguy/ai-devkit
**Stars**：1128 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-31（基于当前 HEAD）
**主题标签**：`cross-reference`, `template-design`, `manifest-discipline`, `single-purpose`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

ai-devkit 是一套面向专业开发者的 **AI 辅助开发生命周期管理工具包**，当前版本 0.24.0。核心价值主张：将软件开发的全生命周期（需求 → 设计 → 编码 → 测试 → 文档）通过统一的 NL command 和 skill 体系覆盖，并同步支持 Claude Code、Codex CLI、Cursor 三个 AI 工具平台。1128 stars，活跃维护。安装方式：`npm install -g ai-devkit` 后由 CLI 脚本将 command/skill 文件注入用户本地目录。

关键事实：
- 支持 Claude / Codex / Cursor 三平台，command 文件五套并发（`.claude/`、`.codex/`、`.cursor/`、`commands/`、`packages/cli/templates/commands/`）
- 9 个核心 SKILL.md（100分满分，质量优秀）
- 12 个核心 command，覆盖需求、设计、测试、调试、文档、简化等
- 内含 memory skill 实现跨会话记忆管理

### 1.2 架构剖析

```
ai-devkit/
├── .claude-plugin/plugin.json          ← manifest（v0.24.0，与 package.json 同步）
├── skills/                             ← 10 个核心 SKILL.md（100分）
│   ├── agent-orchestration/SKILL.md
│   ├── capture-knowledge/SKILL.md
│   ├── debug/SKILL.md
│   ├── dev-lifecycle/SKILL.md
│   ├── memory/SKILL.md
│   ├── simplify-implementation/SKILL.md
│   ├── tdd/SKILL.md
│   ├── technical-writer/SKILL.md
│   ├── verify/SKILL.md
│   ├── structured-debug/SKILL.md      ← 审查后新增
│   ├── document-code/SKILL.md         ← 审查后新增
│   └── security-review/SKILL.md       ← 审查后新增
├── commands/                           ← 模板版 command（含占位符 {{docsDir}}）
├── .claude/commands/                   ← Claude 版（已实例化，硬编码 docs/ai/）
├── .codex/commands/                    ← Codex 版（同上）
├── .cursor/commands/                   ← Cursor 版（同上）
├── packages/
│   ├── cli/                            ← npm CLI 包（负责分发安装）
│   ├── memory/                         ← 记忆管理包
│   ├── agent-manager/                  ← agent 管理包
│   └── channel-connector/              ← 渠道连接包
├── skills/dev-lifecycle/scripts/
│   └── check-status.sh                 ← 唯一 shell 脚本（已修复路径注入漏洞）
└── package.json                        ← Nx monorepo 根配置
```

**文件类型分布**：12 个 SKILL.md / 12+×5 个 command（5套）/ 1 个 plugin.json / 4 个 npm 包

**编排关系**：skills 和 commands 彼此独立（平列）；`dev-lifecycle/SKILL.md` 通过 `references/` 子目录引用所有 9 个阶段 command，形成软耦合；`memory/SKILL.md` 被多个 command 调用以实现跨会话记忆。

**跨件契约**：5套 command 从同一"模板版"（`packages/cli/templates/commands/`）生成，模板用 `{{docsDir}}` 占位符；安装时 CLI 替换为具体路径（如 `docs/ai/`）。plugin.json 版本与 package.json 和 `packages/cli/package.json` 三处保持同步。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**开发生命周期协议化**。把软件开发每个阶段（需求/设计/执行/验证/文档/简化）都建模为可调用的 NL command，用 skill 提供横切关注点（记忆、调试方法论、测试驱动、技术写作）。
- **解决什么问题**：AI 辅助开发的主要痛点之一是"AI 不记得上次讨论了什么"以及"AI 无法理解项目规范"。ai-devkit 用 memory skill + 专用 command 系统性解决这两个问题。
- **做了什么 trade-off**：5套 command 并发维护带来了显著的 drift 风险（已在审查中出现）；换来的是用户零配置即可在多个 AI 工具间使用相同工作流。
- **反映什么认知模型**：作者把 AI 工具视为**协议的执行器**，而非黑盒 AI。每个 command 是一个明确的协议规范；skill 是横切关注点的知识库。这和 REST API 的接口契约设计思想同构。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「模板实例化 + 多目标分发」

核心思路：source of truth 是含占位符的模板 command，安装 CLI 按目标平台替换占位符后写入对应目录（`.claude/`, `.codex/`, `.cursor/`）。Skill 只有一套，不做平台区分。

特征清单：
- 特征 1：Command 模板用 `{{docsDir}}` 等占位符隔离平台差异
- 特征 2：Skill 完全平台无关（不含路径）
- 特征 3：`dev-lifecycle` skill 通过 `references/` 软引用阶段 command，形成可选依赖
- 特征 4：memory skill 作为全局横切关注点，被多个 command 隐式依赖
- 特征 5：npm monorepo（Nx）将 CLI、memory、agent-manager 作为独立包，减少耦合

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖多个 AI 工具平台的 devkit 类工具 | ✅ 高度适用 | 模板分发是解决多平台 drift 的正确抽象层 |
| 团队标准化 AI 工作流（企业内部推广） | ✅ 适用 | command 协议统一了团队 AI 使用方式 |
| 个人配置（单平台、单用户） | ❌ 过度工程 | npm monorepo + 5套分发对个人用户太重 |
| 需要深度平台特定功能（Claude hooks、Kiro steer） | ⚠️ 谨慎 | 模板化会掩盖平台特性 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 模板实例化（本仓库） | ai-devkit | 单一源，模板占位符隔离差异 | 5套文件仍需同步；模板本身需要维护 |
| 备选 A：平台分支仓库 | 大多数 fork 方案 | 平台完全独立、可深度定制 | N×M 的维护量，drift 不可避免 |
| 备选 B：Runtime 适配层 | agentsys transformer | 安装脚本做路径替换，skill 完全纯净 | 复杂度更高；需要 transformer 逻辑 |

### 2.4 改进空间

1. **当前问题**：5套 command 没有 `allowed-tools` 声明，Claude Code 无法做权限收敛。**改进做法**：在模板 command 的 frontmatter 中加 `allowed-tools: [Read, Write, Bash]`（按实际用途声明），CLI 安装时一起注入。**预期收益**：每个 command 的权限边界清晰，减少意外操作。

2. **当前问题**：`code-review.md` 在不同平台版本之间有内容差异（claude/codex 版缺"全局代码 review"步骤）。**改进做法**：在 CI 中加 diff 检查，对比各平台版本与模板版的差异；差异超过占位符替换范围即报错。**预期收益**：自动捕捉下一次 drift。

3. **当前问题**：skills 没有 `output:` 格式声明（审查时 memory 等 skill 100分，但 output 结构是隐式的）。**改进做法**：每个 skill 在 frontmatter 中加 `output: structured` 或内联一个 JSON schema 示例。**预期收益**：调用方可以对 skill 输出做程序化处理。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **96/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.cursor/commands/technical-writer-review.md` | 85 | 无空输入处理（-10），无 allowed-tools（-5） |
| 61 个 command 文件 | 95 | 缺 allowed-tools 声明（-5 each） |
| 9 个 SKILL.md | 100 | 无 |
| `.claude-plugin/plugin.json` | 100 | 无 |

### 3.2 当时值得借鉴的模式

1. **Skills 100分满分**：9个 skill 全部 100分，触发词、输出格式、示例完整。→ 根本原因：skill 和 command 分离，skill 只包含"方法论知识"，command 负责调用序列，职责单一。→ 如何借鉴：把"做什么"（method）和"怎么做到这一步"（invocation）分开写。

2. **`dev-lifecycle` 的 references/ 软耦合**：skill 通过 `references/*.md` 文件链接 command，而非硬编码路径。所有引用在审查时都能解析。→ 如何借鉴：在需要跨文件引用时，用 `references/` 子目录而非直接写绝对路径。

3. **版本三处同步**：plugin.json、package.json、packages/cli/package.json 都是 0.24.0。→ 如何借鉴：每次发版跑 `grep -r '"version"' plugin.json package.json packages/*/package.json` 人工核对。

4. **check-status.sh 唯一脚本**：仅一个 shell 脚本，功能聚焦（检查 feature 开发状态），其他逻辑全在 TypeScript 包中。→ 如何借鉴：不要在 NL skill 的配套脚本中堆逻辑，保持脚本单职责。

### 3.3 当时的缺陷

1. **61个 command 全部缺 `allowed-tools`**：这意味着 Claude Code 执行这些 command 时不知道会用到哪些工具，必须动态声明。→ 根本原因：作者在写 command 时把 allowed-tools 视为可选元数据，而非权限边界契约。→ 自查：`grep -L "allowed-tools" ~/.claude/commands/*.md`。

2. **多平台 command 内容 drift**：`code-review.md` 的 `.claude/` 和 `.codex/` 版本缺少"全局代码 review"步骤，该步骤只在 `commands/` 和 `.cursor/` 版本中有。`new-requirement.md` 步骤顺序在各版本间不一致。→ 根本原因：多套文件的更新没有集中校验机制，某次修改只更新了部分版本。→ 自查：如果自己也有多套 command，检查是否有 diff 检测。

3. **`technical-writer-review.md` 缺空输入处理**：command 直接开始 review 流程，没有"如果没提供文档路径，先问用户"的步骤。→ 根本原因：command 假设用户总会传参，忽略了空调用场景。→ 自查：检查每个使用 `$ARGUMENTS` 的 command 是否有 fallback。

### 3.4 当时的优化机会

1. 给所有 command 文件的 frontmatter 加 `allowed-tools` 声明（影响 61 个文件，最高优先级）
2. 修复 `technical-writer-review.md` 空输入处理（加一个开场询问步骤）
3. 将 `code-review.md` 的"全局代码 review"步骤反向传播至 `.claude/` 和 `.codex/` 版本

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 61个 command 缺 `allowed-tools` | `grep -l "allowed-tools" .claude/commands/*.md \| wc -l` | **仍存在**：0/命中（0 个文件有 allowed-tools） | 高优先级缺陷未修复 |
| check-status.sh 路径注入漏洞 | `grep "\^\[a-zA-Z0-9" skills/dev-lifecycle/scripts/check-status.sh` | **已修复**：第 14 行有 `^[a-zA-Z0-9_-]+$` 正则验证 | 安全修复已落地 |
| technical-writer-review.md 空输入 | `ls .cursor/commands/technical-writer-review.md` | **已消灭**：文件不存在 | 文件已被删除或移走 |

### 4.2 架构演进

- **Skills 层扩展**：新增 `structured-debug/SKILL.md`、`document-code/SKILL.md`、`security-review/SKILL.md` 三个 skill，覆盖面从 9 增至约 12
- **Commands 精简**：`.cursor/commands/technical-writer-review.md` 已删除，说明作者根据反馈删除了质量最差的那个 command，而非修复它
- **安全改进落地**：check-status.sh 路径注入漏洞被修复（验证正则加入），说明安全 PR 已被接受

### 4.3 新增的可学习模式

**structured-debug skill**：在原有 debug/SKILL.md 之外新增了 structured-debug，说明作者区分了"快速调试"（debug）和"系统化调试"（structured-debug）两种场景。这是一种**功能细分而非功能膨胀**的做法——不是把 structured 功能塞进原有 skill，而是新建一个独立 skill，保持单职责。

---

## 五、校准

### 5.1 我已经在做对的

1. **`echo-sleuth-for-claude` 的 command 有 allowed-tools 声明**：dashboard.md、extract.md 等 command 已有 allowed-tools，和 ai-devkit 应该达到的标准一致。
2. **`claude-for-legal` 的 skill 有 references/ 软耦合**：ip-legal/ 等区域的 skill 通过 references/ 目录引用协议文件。
3. **`drama-workshop-skills` 的 SKILL.md 职责单一**：short-drama/SKILL.md 只负责创作知识，不混入安装逻辑。

### 5.2 挑战 / 验证

**验证**：我之前不确定"是否需要为所有 command 声明 allowed-tools"。ai-devkit 的案例验证了：不声明 allowed-tools 是一个普遍性问题（61 个文件无一声明），且这个问题在审查后仍未修复——说明它的修复成本确实高（需要分析每个 command 实际调用了哪些工具）。这验证了"从项目开始就声明 allowed-tools"比"事后补"要合理得多。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 command 是否有 allowed-tools
find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/commands/*" \
  | xargs grep -L "allowed-tools" 2>/dev/null

# 检查是否有多套相同 command（潜在 drift 风险）
find /tmp/my-repos/MarkQWu-* -name "code-review.md" -o -name "debug.md" 2>/dev/null

# 检查 command 是否处理空输入（$ARGUMENTS 有无 fallback）
find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/commands/*" \
  | xargs grep -l '\$ARGUMENTS' 2>/dev/null \
  | xargs grep -L "If no\|if not provided\|如果没有\|如未提供" 2>/dev/null
```

命中 allowed-tools 缺失后：查看 command 体，找出实际用到的工具（Read/Write/Bash/WebFetch 等），在 frontmatter 中声明。
命中空输入问题后：在 command 首行加条件检查，若 `$ARGUMENTS` 为空则先询问用户。

### 6.2 灵感 → 实施路径

1. **想法**：给 `echo-sleuth-for-claude` 的命令加统一 allowed-tools 模板，避免遗漏。
   - **为何可行**：echo-sleuth 已有部分 command 有 allowed-tools，完善覆盖是增量工作。
   - **第一步**：读取 commands/ 目录所有 .md，对照命令体补全 allowed-tools。预计 20 分钟。

2. **想法**：为 `drama-workshop-skills` 新增 structured-debug 风格的功能细分——"快速策划"和"深度策划"两个 skill，而非一个大而全的 skill。
   - **为何可行**：短剧策划有明显的两种用法：快速验题（10分钟）和完整策划（2小时），当前 SKILL.md 混合了两种场景。
   - **第一步**：在 short-drama/ 中创建 quick-pitch/SKILL.md，提炼快速验题流程。预计 45 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 codeaholicguy/ai-devkit 的核心目的**：提供覆盖软件开发全生命周期的 AI 辅助工作流套件（需求→设计→编码→测试→文档），支持多 AI 工具平台

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 agent + command + skill 三层架构 | 功能聚焦（会话挖掘 vs 全生命周期）；echo-sleuth 无多平台分发 | 高 |
| MarkQWu/claude-for-legal | 中 | 都是系统性 skill 套件，覆盖多个工作场景 | 垂直领域不同；claude-for-legal 更偏知识库，ai-devkit 更偏流程协议 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都有 command 调用 skill 的模式 | 单场景 vs 多生命周期阶段 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-*/commands/*.md 2>/dev/null` | drama-workshop-skills 无 commands 目录；echo-sleuth commands 中 recap.md、timeline.md 无 allowed-tools | 中 |
| 多套 command 内容 drift | `find /tmp/my-repos/MarkQWu-* -name "code-review.md"` | 未发现多套同名 command（无 drift 风险） | 无 |
| Shell 脚本路径注入 | `grep -n '"\$1"\|'"'"'\$1'"'"'' /tmp/my-repos/MarkQWu-*/*/scripts/*.sh` | 未发现 shell 脚本（无风险） | 无 |

**命中后的具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude/commands/recap.md` 和 `timeline.md` → 查看命令体实际用到哪些工具（推测是 Read + Bash），在 frontmatter 加 `allowed-tools: [Read, Bash]`。5-10 分钟可完成。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：skill 与 command 职责分离
   - **本案例做法**：skills/ 存放"方法论"（如何调试、如何写测试），commands/ 存放"调用流程"（按步骤执行、调用哪些 skill），两层完全分离（`codeaholicguy-ai-devkit/skills/debug/SKILL.md` vs `.claude/commands/debug.md`）
   - **我的项目现状**：`echo-sleuth-for-claude` 的 agents/ 文件混合了"能力定义"和"调用逻辑"，边界不清晰
   - **如何借鉴**：将 agents/ 中的能力描述提炼为 skills/，agents 文件只保留调度逻辑

2. **领域**：功能细分而非功能膨胀（structured-debug vs debug）
   - **本案例做法**：debug + structured-debug 两个 skill 各司其职，不相互包含
   - **我的项目现状**：`drama-workshop-skills/short-drama/SKILL.md` 将快速验题和完整策划混在一个文件
   - **如何借鉴**：拆出 quick-pitch/SKILL.md（快速模式），主 SKILL.md 专注完整流程

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：allowed-tools 声明覆盖率
- **我的做法**：`MarkQWu/echo-sleuth-for-claude/commands/` 的 dashboard.md、extract.md、prune.md、audit.md 均有 allowed-tools 声明
- **本案例做法**（弱在哪）：ai-devkit 61 个 command 无一声明 allowed-tools，审查后仍未修复
- **意义**：allowed-tools 完整性是 echo-sleuth 的一个已有优势，值得继续保持并在新 command 中延续

---

## 八、术语表

### <a name="allowed-tools"></a>allowed-tools
> Command 或 agent 定义文件 frontmatter 中的字段，声明该命令在执行时允许调用哪些工具（如 `Read`、`Bash`、`Write`、`WebFetch`）。未声明的工具在执行时需用户额外确认，或被权限策略拒绝。是权限收敛的关键声明。

### <a name="monorepo"></a>monorepo（单一仓库多包）
> 一个 git 仓库里存放多个独立 npm 包（或模块）的架构。ai-devkit 用 Nx 管理：`packages/cli`、`packages/memory`、`packages/agent-manager`、`packages/channel-connector` 四个包在同一仓库，共享工具链和版本策略，但各自独立发布。

### <a name="drift"></a>drift（内容漂移）
> 同一功能在多个地方有多份副本，经过若干次独立修改后，副本之间内容出现不一致的现象。ai-devkit 的 5 套 command 文件就发生了 drift：`code-review.md` 的 `.claude/` 版本和 `commands/` 版本内容不同步。

### <a name="placeholder"></a>placeholder（占位符）
> 模板文件中待替换的变量，如 `{{docsDir}}`。安装 CLI 在"实例化"模板时把 `{{docsDir}}` 替换为用户项目中的实际目录路径（如 `docs/ai/`）。占位符把"模板"和"使用时的具体值"解耦。
