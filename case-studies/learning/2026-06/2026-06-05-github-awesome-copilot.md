# github/awesome-copilot — 学习案例

**仓库**：https://github.com/github/awesome-copilot
**Stars**：31,146 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-06-05（基于当前 HEAD）
**主题标签**：`examples-driven`, `model-pinning`, `manifest-discipline`, `router-channels`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是 GitHub 官方维护的 GitHub Copilot agent 资源库（31,146 stars），聚合了 100 个可复用 agent 制品，覆盖从前端（Next.js、React 19 升级）到后端（Drupal、Kubernetes）、从数据库（MongoDB、Kusto）到商业系统（Salesforce、SAP）的广泛领域。

关键事实：
- 由 **GitHub 官方** 维护，是 GitHub Copilot 生态的 agents 参考库
- 包含 84 个 standalone agents（`agents/`）、15 个 plugin agents（`plugins/`）、1 个治理 agent（`.github/agents/`）
- 配套 10 个 shell hook 脚本（`hooks/`），实现自动提交、密钥扫描、许可证检查、治理审计等功能
- 这是一个可以直接 fork 使用的「生产级 agent 库」，不是教程也不是演示

### 1.2 架构剖析

```
awesome-copilot/
├── .github/
│   └── agents/
│       └── agentic-workflows.agent.md   ← 治理 agent（BUG：缺 name 字段）
├── agents/                               ← 84 个领域专家 agent
│   ├── expert-nextjs-developer.agent.md ← 93/100（最高分）
│   ├── accessibility.agent.md           ← 92/100
│   ├── task-researcher.agent.md         ← 55/100（最低分之一）
│   └── ...（共 84 个）
├── plugins/
│   ├── react19-upgrade/                 ← 4-agent 流水线（审计→依赖→迁移→测试）
│   ├── go-mcp-development/
│   ├── azure-cloud-development/
│   └── ...（共 15 个）
└── hooks/
    ├── session-auto-commit/auto-commit.sh  ← SEC-001：--no-verify 绕过密钥扫描
    ├── secrets-scanner/scan-secrets.sh
    ├── governance-audit/audit-prompt.sh    ← SEC-003：local 在函数外崩溃
    ├── license-checker/
    └── tool-guardian/guard-tool.sh
```

- **文件类型分布**：100 个 agent 文件、10 个 shell hooks、15 个 plugin 目录
- **编排关系**：`plugins/react19-upgrade/` 是最精密的多 agent 编排——4 个 agent 按「auditor → dep-surgeon → migrator → test-guardian」流水线串联，有内存协议和 GO/NO-GO 门控
- **跨件契约**：hooks 目录的各脚本之间有明确的信任链——`session-auto-commit` 本应触发 `secrets-scanner`，但 `--no-verify` 打破了这个链条

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「示例即文档」——高分 agent 全部包含完整的代码示例、before/after 对比、框架适配器示例。最好的 agent 本身就是一份教程
- **解决什么问题**：GitHub Copilot 用户需要领域专家级的 agent 配置，但不知道怎么写好——这个库提供了「拿来即用」的参考
- **做了什么 trade-off**：覆盖面（100 个 agent，84 个领域）vs 深度（每个 agent 无法做到面面俱到）。高分 agent 明显比低分 agent 内容丰富数倍
- **反映什么认知模型**：作者深度理解「agent 文件 = agent 的合同」——工具声明、模型固定、具体示例三者缺一不可

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「多层 Agent 库 + 流水线插件」

特征清单：
- **三层结构**：治理级（`.github/agents/`）→ 通用库（`agents/`）→ 场景编排（`plugins/`）
- **示例密度决定质量**：评分高低与「代码示例数量」强相关——满分 agent 有 8+ 代码块，低分 agent 一个都没有
- **hooks 作为安全层**：单独的 `hooks/` 目录承担密钥扫描、许可证检查、治理审计，与 agent 内容层解耦
- **插件内部有 pipeline**：`plugins/react19-upgrade/` 展示了多 agent 有状态流水线的完整实现模式

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业内部多领域 AI 辅助工具库 | ✅ 高度适用 | 三层结构 + hooks 安全层适合企业规模 |
| 个人垂直领域插件 | ❌ 过度设计 | 100 个 agent 的结构复杂度对 1-5 个 agent 的项目不必要 |
| 需要有状态多步骤工作流 | ✅ 适用 | react19-upgrade 的 pipeline 模式是最佳参考 |
| 快速验证一个 agent 想法 | ❌ 不适用 | 整个仓库是生产级架构，不适合原型实验 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多层 Agent 库（本仓库） | github/awesome-copilot | 覆盖广、示例充分、内置安全钩子 | 维护成本高、低质 agent 拖累整体评分 |
| 薄插件 | jarrodwatts/claude-hud | 结构简单、质量可控 | 无法覆盖多领域 |
| 精选汇编 | hesreallyhim/awesome-claude-code | 内容来自社区、更新快 | NL 质量依赖上游，不可控 |

### 2.4 改进空间

1. **当前问题**：`agents/` 和 `plugins/` 之间存在重复 agent（go-mcp-expert、terraform-azure-implement 各出现两次）  
   **改进做法**：在 `agents/` 中的 standalone 版本顶部加注释 `canonical source: plugins/…`，或直接删除重复版本，改为符号链接  
   **预期收益**：避免修复一处而另一处遗漏的维护债

2. **当前问题**：SEC-001（`--no-verify` 绕过密钥扫描）和 SEC-003（`local` 在函数外崩溃）至今未修复  
   **改进做法**：SEC-001：删除 `--no-verify`；SEC-003：把 threat 处理逻辑包进一个 bash 函数  
   **预期收益**：治理钩子实际发挥作用，而不是在最需要时崩溃

3. **当前问题**：73% 的 agent 没有声明 `model` 字段  
   **改进做法**：为每个 agent 添加 `model: gpt-4o` 或 `model: claude-sonnet-4-6`（根据领域特点选择）  
   **预期收益**：用户部署时有明确的模型期望，而不是依赖平台默认值

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

当时评分 **77/100**，100 个 artifact 加权平均。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.github/agents/agentic-workflows.agent.md` | 50 | 缺 `name` 字段（BUG），无示例 |
| `agents/task-researcher.agent.md` | 55 | 14 个模糊量词，无示例 |
| `agents/kusto-assistant.agent.md` | 62 | 声明了与 Kusto 完全无关的测试工具 |
| `agents/expert-nextjs-developer.agent.md` | 93 | 声明了 model、8+ 代码示例（最高分） |

### 3.2 当时值得借鉴的模式

1. **示例密度决定可用性**：`expert-nextjs-developer.agent.md`（93分）含 8 个代码块、async-params 迁移指南、框架适配器示例——这类 agent 对用户的实际帮助远超那些只有文字描述的 agent。规律：**具体代码示例 > 抽象能力描述**
2. **React 19 迁移 Plugin 的流水线设计**：4 个 agent 各有单一职责（审计、依赖外科手术、迁移、测试防守），通过内存状态机传递上下文，每个环节有明确的 GO/NO-GO 门控。这是我看到的最成熟的多 agent 工作流实现
3. **DAX before/after 反模式表格**：`power-bi-dax-expert.agent.md` 用「错误做法 → 正确做法」对比表格展示知识——这个格式对 AI 理解「什么是好的 DAX」极为高效

### 3.3 当时的缺陷

1. **问题**：`.github/agents/agentic-workflows.agent.md` 缺少 `name` 字段  
   **根本原因**：作者可能把 `description` 当成了 `name`，或者忘记了 `name` 是 agent 注册的必填字段。没有 `name`，这个 agent 在 Copilot 的 agent picker 里完全不可见——相当于一扇没有门牌的门  
   **自查**：我的 echo-sleuth agents 需要检查是否都有 `name` 字段

2. **问题**：`agents/kusto-assistant.agent.md` 声明了 `findTestFiles`、`runTests`、`testFailure` 这些测试工具，但 Kusto 是 Azure 数据查询服务，完全用不到测试运行器  
   **根本原因**：从模板复制后没有做工具列表清洁。这是「copy-paste 不思考」的典型问题——工具声明应该问「这个 agent 实际需要什么」，而不是「模板有什么就放什么」  
   **自查**：我需要检查自己的 agent 工具列表是否存在不匹配的声明

3. **问题**：`auto-commit.sh` 用 `--no-verify` 绕过所有 pre-commit hooks，包括密钥扫描器  
   **根本原因**：开发者想要快速提交（避免 hook 慢），但用了最危险的方式——等于在最关键的安全检查点上开了洞。等 hook 执行完再提交比泄露密钥的代价要低得多  
   **自查**：我的仓库目前没有类似的 hook bypass 模式，这是一条必须坚守的底线

### 3.4 当时的优化机会

1. 删除 `auto-commit.sh` 里的 `--no-verify`，让密钥扫描器正常运行
2. 为 `.github/agents/agentic-workflows.agent.md` 添加 `name: "Agentic Workflows"` 字段
3. 修复 `audit-prompt.sh` 里 `local evidence` 在函数外声明的 bash 语法错误（治理钩子发现威胁时会崩溃）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-001：`agentic-workflows.agent.md` 缺 `name` 字段 | `head -5 .github/agents/agentic-workflows.agent.md` — frontmatter 只有 `description` 和 `disable-model-invocation` | **PERSISTS** | 这个 agent 在 Copilot agent picker 里仍然不可见 |
| BUG-002：`laravel-expert-agent.agent.md` 管道符 model 格式 | `grep "model:" agents/laravel-expert-agent.agent.md` — 返回 `model: GPT-4.1 \| 'gpt-5' \| 'Claude Sonnet 4.5'` | **PERSISTS** | 这个 model 声明对 Copilot 没有意义，会被解析为字面量字符串 |
| SEC-001：`auto-commit.sh` 的 `--no-verify` | `grep "no-verify" hooks/session-auto-commit/auto-commit.sh` — 第 33 行命中 | **PERSISTS** | 密钥扫描器仍然会被绕过 |

### 4.2 架构演进

对比 audit 时（2026-04-25）到现在（2026-06-05），`plugins/` 目录有持续更新，新增了更多场景化 plugin。`agents/` 目录的 agent 数量稳定，核心缺陷（无 model 声明、无示例）在 40% 的 agent 上仍然存在。

3 个关键安全/Bug 问题（BUG-001、BUG-002、SEC-001）全部未修复。GitHub 自己的官方仓库存在这些问题超过 40 天，说明这些「小问题」在实际维护者的待办列表里优先级很低——即使是官方维护者也如此。

### 4.3 新增的可学习模式

当前 HEAD 中的 `plugins/react19-upgrade/` 是一个值得单独深入学习的多 agent 流水线实现，其有状态内存协议设计在 audit 时已被标注为「exemplary」——对于任何需要多步骤、有状态 AI 工作流的场景，这个 plugin 是最好的参考实现之一。

---

## 五、校准

### 5.1 我已经在做对的

1. **agent 工具列表和职责对齐**：我的 echo-sleuth agents 工具声明简洁，没有 kusto-assistant 那种「粘贴模板工具」的问题
2. **没有使用 `--no-verify`**：我从来没在 git commit 中加 `--no-verify`，这条安全底线保持完好
3. **SKILL.md frontmatter 完整**：我的所有 SKILL.md 都有正确的 `name` 和 `description` 字段

### 5.2 挑战 / 验证

本案例验证了一个之前犹豫的做法：**在 SKILL.md 和 agent 文件里放大量具体代码示例是值得的**。

我之前觉得「agent 文件里放太多示例会不会太冗长」，但看到 `expert-nextjs-developer.agent.md`（8 个代码块，93/100）和 `task-researcher.agent.md`（0 个代码块，55/100）的对比，结论是：**代码示例是 NL 制品最有价值的内容，多多益善，不怕冗长**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否都有 name 字段
find ~/.claude/ -name "*.md" -path "*/agents/*" | while read f; do
  grep -q "^name:" "$f" || echo "缺 name 字段: $f"
done
```
命中后怎么办：在 frontmatter 里添加 `name: <agent-name>` 字段。

```bash
# 检查我的 agent 文件是否有 model 声明
find ~/.claude/ -name "*.md" -path "*/agents/*" | while read f; do
  grep -q "^model:" "$f" || echo "缺 model: $f"
done
```
命中后怎么办：根据 agent 的任务类型选择合适模型。计算密集型 → `claude-opus-4-8`；标准任务 → `claude-sonnet-4-6`；轻量分类 → `claude-haiku-4-5-20251001`。

```bash
# 检查我的 agent 文件是否有具体代码示例
find ~/.claude/ -name "*.md" -path "*/agents/*" | while read f; do
  count=$(grep -c '```' "$f" 2>/dev/null || echo 0)
  echo "$count 个代码块: $f"
done
```
命中（0 个代码块）后怎么办：为该 agent 添加至少 2 个 input/output 示例，展示「好的输入」和「期望的输出格式」。

### 6.2 灵感 → 实施路径

1. **想法**：仿照 `react19-upgrade` 的 4-agent pipeline 模式，为 echo-sleuth 设计一个「experience-mining-pipeline」：收集 → 分析 → 综合 → 报告
   - **为何可行**：echo-sleuth 的 4 个 skills（git-mining、jsonl-core、experience-synthesis、memory-management）天然对应 4 个流水线阶段
   - **第一步**：在 echo-sleuth 里新建 `agents/pipeline-orchestrator.md`，定义 4 阶段状态机，每个阶段调用对应 skill，约 45 分钟

2. **想法**：为 echo-sleuth 的 agents 全部补充 model 声明和至少 2 个代码示例
   - **为何可行**：5 个 agent 文件，每个加声明 + 示例不超过 15 分钟
   - **第一步**：先处理 `agents/recall.md`（最常用），加 `model: claude-sonnet-4-6` 和一个 recall 对话示例，10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 github/awesome-copilot 的核心目的**：多领域 Copilot agent 资源库，给开发者提供领域专家级 agent 配置

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为多 agent 结构 | echo-sleuth 是单一领域（历史挖掘），awesome-copilot 是多领域通用 | 高 |
| MarkQWu/claude-for-legal | 中 | 同为多 skill/agent 套件 | claude-for-legal 有领域深度，awesome-copilot 有领域宽度 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 制品 | drama 是垂直内容创作工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 无 `model` 声明 | `grep -rL "^model:" my-repos/MarkQWu-echo-sleuth-for-claude/agents/` | **全部命中（5/5）**：echo-sleuth 所有 agent 无 model 声明 | 中 |
| SKILL.md / agent 无代码示例 | `find .../skills -name "SKILL.md"` 逐一检查 | **命中 4/4**：echo-sleuth 所有 SKILL.md 无示例块 | 高 |
| 工具声明与 agent 职责不匹配 | 手动检查 agent 的 `tools:` 列表 vs 其实际任务 | claude-for-legal 需要检查 | 中 |

**命中后的具体行动建议**：
- `echo-sleuth/skills/jsonl-core/SKILL.md` → 末尾加 `## 示例`，展示一条 JSONL 记录的解析输入输出 → 20 分钟可完成
- `echo-sleuth/agents/recall.md` → frontmatter 加 `model: claude-sonnet-4-6` → 2 分钟
- `echo-sleuth/agents/analyze.md` → 同上，加 model + 2 个 before/after 分析示例 → 15 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：多 agent 流水线编排
   - **本案例做法**：`plugins/react19-upgrade/` 里 4 个 agent 通过明确的内存状态机 + GO/NO-GO 门控串联，每个 agent 只做一件事，失败时整个流水线停止
   - **我的项目现状**：echo-sleuth 的 4 个 skills 各自独立，用户需要手动串联调用，没有自动化的工作流
   - **如何借鉴**：新建 `agents/pipeline-orchestrator.md`，定义 step 顺序、每步成功条件、失败时的 fallback 行为，让用户只需触发一个命令即可完成完整的 echo-sleuth 流程

2. **领域**：before/after 代码对比展示
   - **本案例做法**：`power-bi-dax-expert.agent.md` 用表格展示「错误 DAX → 正确 DAX」对比，直观且可执行
   - **我的项目现状**：echo-sleuth/skills 的文档是叙述性的，没有具体的 input/output 示例
   - **如何借鉴**：在每个 SKILL.md 的 `## 示例` 块里用代码块对比「调用前数据」和「调用后结果」

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent 工具声明的精确性
- **我的做法**：echo-sleuth 的 agents 工具列表简洁，只声明实际需要的工具（如 Bash、Read），没有 awesome-copilot 那种「从模板复制 + 没清洁」的冗余工具声明
- **本案例做法（弱在哪）**：`kusto-assistant.agent.md` 声明了 `findTestFiles`、`runTests` 这些测试工具，对一个查询助手毫无意义，导致扣了 12 分
- **意义**：工具声明精确 = 安全权限最小化，是一个好习惯

---

## 八、术语表

### <a name="agent-picker"></a>agent picker
> GitHub Copilot 或 Claude Code 的界面中，用户选择调用哪个 agent 的下拉菜单或列表。agent 要出现在这里，必须在 frontmatter 里有正确的 `name` 字段。没有 `name` 的 agent，用户永远无法在 picker 里找到它——就像一个没有名字的联系人，无法通过搜索找到。

### <a name="go-no-go"></a>GO/NO-GO 门控
> 流水线中的检查点：前一个阶段必须成功，才允许进入下一阶段。例如 react19-upgrade 流水线中，dep-surgeon 必须成功修改完依赖，migrator 才能开始代码迁移。如果跳过 GO/NO-GO 检查，流水线后续步骤可能在错误前提下运行，造成难以回滚的错误。

### <a name="model-pinning"></a>model pinning（模型版本固定）
> 在 agent 或 skill 的 frontmatter 里明确声明使用哪个 AI 模型（如 `model: claude-sonnet-4-6`）。如果不固定，平台会使用默认模型，而默认模型可能在平台更新后改变，导致 agent 行为漂移。固定模型 = 可预期的行为 = 更容易调试。

### <a name="no-verify"></a>--no-verify
> git commit 的一个参数，加上它之后，git 会跳过所有 pre-commit 和 commit-msg hooks 直接提交。常见的安全 hooks（如密钥扫描、代码格式检查）会因此被完全绕过。本案例的 `auto-commit.sh` 使用了这个参数，导致 `secrets-scanner` hook 形同虚设。
