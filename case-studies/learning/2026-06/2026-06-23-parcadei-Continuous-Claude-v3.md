# parcadei/Continuous-Claude-v3 — 学习案例

**仓库**：https://github.com/parcadei/Continuous-Claude-v3
**Stars**：1200 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-23（基于当前 HEAD）
**主题标签**：stateful-memory, math-sandbox, multi-session, security-blocked, typescript-hooks

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Continuous-Claude-v3 是 parcadei 基于 Claude Code 构建的"有状态连续工作流引擎"。它的核心诉求是：Claude 不应该因为上下文窗口满了或会话结束就失忆——它应该通过 PostgreSQL + pgvector 的记忆系统、跨会话的 handoff 协议和 TLDR 代码智能层，在多个会话中保持连续性。1200 颗星，定位是"个人全栈 AI 工作站的 OS 级扩展"。

关键事实：
- 系统基于 OPC（Oneframe Protocol for Claude）架构，通过 subprocess 方式扩展 Claude Code
- PostgreSQL + pgvector + BGE 向量嵌入提供混合 RRF 召回的持久记忆
- TLDR CLI 提供代码智能（4 个 skill 文件包装）
- 数学证明沙盒使用 Lean/Mathlib，有 CPU/RAM 资源限制和超时处理
- audit 时有 94 个 artifact（32 agents + 62 skills），当前 HEAD 演进为 32 agents + **171 skills**（大幅增加）

### 1.2 架构剖析

```
Continuous-Claude-v3/
├── opc/                        # OPC 系统核心
│   ├── scripts/
│   │   ├── repoprompt_async.py # ⚠️ subprocess.run(shell=True) 注入点
│   │   ├── agent_system_prompt.md
│   │   └── README.md
│   └── src/                   # Python 核心模块
├── .claude/
│   ├── agents/                 # 32 个 agent（audit 时有 32，当前仍 32）
│   └── skills/                 # 171 个 SKILL.md（audit 时 62，大幅扩展）
│       ├── prove/SKILL.md      # ⚠️ 含 curl-pipe-sh（CRITICAL）
│       ├── validate-agent/SKILL.md  # ⚠️ 仍含 model: haiku
│       ├── remember/, recall/  # 记忆系统 skill
│       ├── debug/, debug-hooks/
│       └── ...（另约 160 个）
├── docker/                    # Docker 部署（math sandbox）
├── proofs/                    # Lean 数学证明文件
└── docs/                      # 架构文档
    ├── ARCHITECTURE.md
    ├── MULTI-SESSION-ARCHITECTURE.md
    └── TLDR.md
```

- **文件类型分布**：32 agents / 171 SKILL.md（现状）/ 1 个 Python 核心脚本 / 多个 Docker 配置
- **编排关系**：orchestrator agents（kraken、arbiter 等）通过 skill 体系调度 subagent；memory-extractor 是唯一有 Examples section 的 agent，评分 90
- **跨件契约**：audit 时发现 6 个悬挂引用（`codebase-locator`、`turbo` 等不存在的 agent）；当前是否修复需验证

### 1.3 设计思路 / 方法论

- **核心设计哲学**：AI 工作不该受限于单个上下文窗口——通过持久存储和连续性协议，把 Claude Code 从"一次性助手"升级为"长期工作伙伴"
- **解决什么问题**：Claude Code 会话结束后的"记忆消除"问题，以及跨会话协作工作时的状态传递问题
- **trade-off**：PostgreSQL 记忆系统带来生产级能力，但大幅提高了安装和运维门槛（需要数据库、pgvector、BGE 模型）
- **认知模型**：作者把 Claude Code 视为可以拥有长期记忆的工程师，每次会话是一个"工作班次"，handoff ledger 是交班记录，recall 是查阅历史记录

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「有状态持久化 Claude：数据库记忆 + 沙盒执行 + 多会话连续性」

核心特征：
- 特征 1：PostgreSQL + pgvector 作为外部记忆后端，RRF 混合召回——把 AI 记忆外置到专业数据库
- 特征 2：`create_handoff` / `continuity_ledger` skill 实现跨会话状态传递协议
- 特征 3：TLDR CLI 提供代码智能（4 个 skill 包装），减少 AI 在巨型代码库中的盲目探索
- 特征 4：Math sandbox 用 Docker + CPU/RAM 限制 + timeout 实现安全的代码执行环境
- 特征 5：hook 测试套件（vitest）为 30+ hook 脚本提供单元测试覆盖

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要跨会话持久记忆的长期项目 | ✅ 核心用例 | PostgreSQL 记忆系统专为此设计 |
| 包含数学/形式化证明的研究项目 | ✅ 适用 | Lean/Mathlib 沙盒完整集成 |
| 个人轻量使用 | ❌ 不适用 | 需要 PostgreSQL + pgvector + BGE 模型，部署成本高 |
| CI/CD 自动化流水线 | ⚠️ 谨慎 | repoprompt_async.py 的 shell=True 注入风险 |
| 安全敏感环境 | ❌ 不适用 | prove/SKILL.md 的 CRITICAL 安全发现未修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 有状态持久化（Continuous-Claude） | parcadei/Continuous-Claude-v3 | 真正的跨会话连续性，生产级记忆 | 部署复杂，安全债未清 |
| 会话内 context 管理 | 大多数 skill 仓库 | 零依赖，即开即用 | 会话结束失忆，无法积累 |
| 文件系统记忆 | mem0ai/mem0 | 中等复杂度，无数据库依赖 | 召回质量不如向量搜索 |

### 2.4 改进空间

1. **当前问题**：`prove/SKILL.md` line 25 的 `curl ... | sh` 是 CRITICAL 安全风险。**改进做法**：替换为 elan 的 package manager 安装路径（`brew install elan` 或 `nix-env -i elan`），或提供 pinned hash 验证步骤。**预期收益**：消除安全阻断，contribution workflow 解锁。
2. **当前问题**：所有 31 个 agent（除 memory-extractor）缺 Examples section，平均扣 15 分。**改进做法**：每个 agent 加 2-3 行示例（用户输入 + 期望输出格式）。**预期收益**：agent 平均分从 74 → 89，整体 NL score 从 78 → 84。
3. **当前问题**：`validate-agent.md` 声明 `model: haiku`，违反项目自己的 `.claude/rules/no-haiku.md`。**改进做法**：删除该行，让 agent 继承默认 model。**预期收益**：消除自我矛盾，符合项目规范。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **78/100**（Security: BLOCKED，Bugs: 14，Quality Issues: 2 systemic patterns）。

| 类别 | 文件数 | 平均分 | 主要问题 |
|---|---|---|---|
| Agents | 32 | 74 | 31/32 缺 Examples（-15/个）；9 缺 tools 声明 |
| Skills | 62 | 80 | ~25 缺 allowed-tools；部分有悬挂引用 |

### 3.2 当时值得借鉴的模式

1. **PostgreSQL + pgvector 记忆架构**：`recall_learnings.py` / `store_learning.py` 提供干净的 API，RRF 混合召回（向量 + BM25）是工程成熟度的标志。
2. **Math sandbox 安全设计**：`sandbox_runner.py` 实现 CPU/RAM 资源限制、timeout handler、restricted exec builtins——每个安全机制都独立可测试。
3. **Hook 测试套件**：30+ hook 脚本有 vitest 单元测试覆盖，这在 Claude Code 插件生态中极罕见。
4. **TLDR 四 skill 包装**：`tldr-code`、`tldr-stats`、`session-start-tldr-cache.sh` 把 CLI 工具包装为 AI 可用的认知工具，有示例有 allowed-tools。
5. **Handoff continuity 协议精度**：`continuity_ledger/SKILL.md` 的 YAML schema 精确到字段级别（`goal:`、`now:` 等 statusline 字段），跨会话状态传递有规约。

### 3.3 当时的缺陷

1. **CRITICAL: prove/SKILL.md 的 curl-pipe-sh**：`curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh` 在 skill 文件中意味着 Claude 被指示直接执行未经验证的远程脚本。根本原因：开发者把"快速安装指南"原文复制进了 skill，没有意识到 skill 内容会被 AI 直接执行（而不只是展示给人看）。**NL 编程的关键教训**：SKILL.md 的每一行指令都可能被 AI 执行，不能把"文档用途"的内容放进 skill。**我有没有犯？** 我的 skill 没有安装指令，未命中。
2. **31/32 agent 缺 Examples**：整个 agent 体系除一个外全部缺 Examples section，导致系统性扣分。根本原因：作者专注于 agent 逻辑内容，遗忘了 Examples 是 NL 评分的 mandatory 项（-15 分）。**自查**：graphify/skill.md 是否有 Examples？需要检查。
3. **`repoprompt_async.py` 的 `subprocess.run(shell=True)`**：f-string 拼接命令字符串传给 shell，workspace 名包含元字符会 RCE。根本原因：Python 的 `subprocess.run(shell=True)` 是初学者常见的便利陷阱——简单但危险。**自查**：我没有类似的 Python 脚本，但我的 gstack 里有 TypeScript execSync，需要确认参数来源。

### 3.4 当时的优化机会

1. **prove/SKILL.md**：替换 curl-pipe-sh 为 `brew install elan` 或官方包管理器路径
2. **31 个 agent**：批量加 2-3 行 Examples section，可以用脚本批处理（每个 agent 体内已有丰富内容，只需格式化一个示例出来）
3. **validate-agent.md**：删除 `model: haiku` 行（1 行修改，消除规则违反）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| prove/SKILL.md curl-pipe-sh | `grep -n "curl" .claude/skills/prove/SKILL.md` | **持续存在**（line 25 完全未变） | CRITICAL 发现 54 天无修复，contribution 持续阻断 |
| validate-agent model: haiku | `grep -n "model" .claude/skills/validate-agent/SKILL.md` | **持续存在**（line 212 仍有 `model="haiku"`） | 自我矛盾的规则违反仍存在 |
| agent 缺 Examples | `grep -rn "## Examples" .claude/agents/` | 快速检查当前状态 → 架构演进中无证据显示已批量补充 | 结构性质量债持续 |

### 4.2 架构演进

最显著的变化是 **skills 数量从 62 → 171**（+176%）。agent 数量维持 32 不变。这说明作者的扩展策略是"加 skill 而不加 agent"——通过丰富 skill 体系来扩展能力，而不是增加 agent 种类。

新增的 skills 包括：`remember`、`recall`（记忆系统前端）、`firecrawl-scrape`（网页抓取）、`search-hierarchy`（搜索优先级策略）等。这些新 skill 填补了 audit 时报告的缺失引用（如 `braintrust-analyze`、`debug`）。

docs/ 目录也大幅充实：新增了 `MULTI-SESSION-ARCHITECTURE.md`、`QUICKSTART.md`、`docs/tools/` 等——说明作者意识到上手成本高，在补充文档。

### 4.3 新增的可学习模式

1. **skills 爆炸式增长策略**：保持 agent 架构不变，通过大量新 skill 扩展能力——agent 是骨架，skill 是肌肉。171 个 skill 覆盖了从 firecrawl 抓取到 agentica 集成的大量场景。
2. **skill-developer SKILL.md**：一个专门教 Claude 如何写新 skill 的 skill——元技能，自举能力。值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. 我的 skill 没有包含任何安装指令（不存在 curl-pipe-sh 风险）
2. graphify 的 `tests/test_skillgen.py` 对 skill 生成有测试覆盖，接近 Continuous-Claude 的 hook 测试思路
3. bureau 的 agent 有 tools 声明

### 5.2 挑战 / 验证

这个案例强化了一个认知：**NL 编程中的 skill 文件不是文档，是指令**。prove/SKILL.md 里的 curl-pipe-sh 如果作为 README 写是完全正常的，但作为 skill 写就是在让 AI 执行危险操作。这个区分在写 skill 时要时刻保持意识——"我写的这句话，如果 AI 真的执行它，会发生什么？"

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 里有没有任何 curl 安装指令
grep -rn "curl.*install\|curl.*sh\|curl.*bash" /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-graphify/ 2>/dev/null | grep -v "node_modules"
```
命中后怎么办：把安装指令从 skill 体内移出，放到 README 或独立的 setup 文档里。

```bash
# 检查 subprocess.run + shell=True 的危险模式
grep -rn "subprocess.run.*shell=True\|subprocess.Popen.*shell=True" /tmp/my-repos/ 2>/dev/null | grep -v test
```
命中后怎么办：把命令字符串改为参数数组：`subprocess.run(['git', 'clone', url], shell=False)`。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 Continuous-Claude 的 `skill-developer` 元技能思路，给 bureau 或 graphify 加一个"新 skill 生成协议" skill，教 Claude 如何为新数据源生成符合 bureau schema 的 skill
   - **为何可行**：bureau 有标准的 agent 模板（`crew/_template/agent.md`），只需将生成协议写成 skill
   - **第一步**：阅读 `crew/_template/agent.md`，提炼为 3 步生成流程，写入 `.claude/skills/skill-developer/SKILL.md`。约 20 分钟可完成。

2. **想法**：借鉴 continuity_ledger 的精确 YAML schema，给 bureau 的会话 handoff 加结构化格式
   - **为何可行**：bureau 已有 file-session.md 但 handoff schema 不明确
   - **第一步**：阅读 bureau/commands/file-session.md 现有格式，与 continuity_ledger SKILL.md 对照，补充 `goal:`、`now:`、`next:` 字段定义。约 15 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 parcadei/Continuous-Claude-v3 的核心目的**：跨会话持久记忆 + 多智能体连续工作流引擎

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都要解决"AI 的记忆和知识持久化"问题 | bureau 基于 Git 结构化知识库，Continuous-Claude 基于 PostgreSQL | 高 |
| MarkQWu/graphify | 中 | 都处理代码库的持久化理解 | graphify 是静态图谱，Continuous-Claude 是动态会话记忆 | 中 |
| MarkQWu/gstack | 低 | 都是 Claude Code 工程工具集 | gstack 无持久记忆机制 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 内含安装/执行指令 | `grep -rn "curl\|npm install" $(find /tmp/my-repos/MarkQWu-bureau -name "*.md")` | bureau skills 未找到 curl-pipe-sh → 未命中 | N/A |
| agent 缺 Examples section | `grep -L "## Examples" /tmp/my-repos/MarkQWu-bureau/.claude/agents/*.md 2>/dev/null` | bureau 只有一个 agent（auditor），无 Examples section → 命中 | 中 |
| model: haiku 违反项目规则 | `grep -rn "model.*haiku" /tmp/my-repos/MarkQWu-bureau/ 2>/dev/null` | 未命中 | N/A |

**命中后的具体行动建议**：
- bureau/.claude/agents/auditor.md → 加 `## Examples` section（3 行：用户输入 + 期望的审计报告格式）→ 10 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：hook 测试覆盖
   - **本案例做法**：30+ hook 脚本有 vitest 单元测试（`dist/__tests__/`），每个 hook 的边界情况都有测试用例
   - **我的项目现状**：bureau 的 hooks/ 目录下有 shell 脚本，但无任何测试
   - **如何借鉴**：为 bureau 最关键的 hook（如 `canon-write-guard.sh`，如果实现的话）写一个最小化的 shell 测试脚本，验证"正确路径允许、错误路径拒绝"的基本行为。约 30 分钟可完成。

2. **领域**：skill-developer 元技能（自举）
   - **本案例做法**：`.claude/skills/skill-developer/SKILL.md` 教 Claude 如何创建新 skill，让系统能扩展自身
   - **我的项目现状**：bureau 和 graphify 都没有"如何新建 skill"的内置指导，新 skill 全靠人工参照已有模板
   - **如何借鉴**：参考 bureau/crew/_template/agent.md，写一个 SKILL.md 专门说明"如何为 bureau 添加新 crew 成员"，让 AI 能帮助扩展系统本身

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skill 文件安全性
- **我的做法**：bureau 和 graphify 的 skill 文件均不包含任何安装命令或 shell 执行指令，skill 只描述行为协议
- **本案例做法**：prove/SKILL.md 包含 CRITICAL 级别的 curl-pipe-sh 安装指令，54 天后仍未修复
- **意义**：这是一个"已经做对"的信号，但也需要建立显式的检查规则（如 CI 中加 `grep -r "curl.*|.*sh" .claude/skills/` 门控）来防止未来引入

---

## 八、术语表

- **[OPC](#opc)**：Oneframe Protocol for Claude，Continuous-Claude-v3 自研的 Claude Code 扩展协议，通过 subprocess 方式在 Claude Code 之外运行 Python 进程，实现持久化状态和工具扩展。
- **[pgvector](#pgvector)**：PostgreSQL 的向量搜索扩展，支持存储和查询高维向量（如文本 embedding），用于语义相似度搜索。
- **[BGE 向量嵌入](#bge)**：BAAI General Embedding，百度研究院开发的文本向量化模型，把文本转换为数值向量用于语义搜索。
- **[RRF 混合召回](#rrf)**：Reciprocal Rank Fusion，一种将多个搜索方法（如向量搜索 + 关键词搜索）的结果合并排序的算法。
- **[Lean/Mathlib](#lean)**：Lean 是微软研究院的定理证明语言，Mathlib 是其数学库。用于形式化验证数学命题，不是普通编程语言。
- **[elan](#elan)**：Lean 的版本管理工具，类似 Node.js 的 nvm。
- **[handoff ledger](#handoff-ledger)**：跨 Claude Code 会话传递状态的结构化记录，包含 `goal`、`now`、`next` 等字段，让下一个会话能从正确位置继续工作。
- **[TLDR CLI](#tldr-cli)**：一个提供代码库摘要和导航的命令行工具，Continuous-Claude-v3 用 4 个 skill 将其包装为 AI 可调用的认知辅助工具。
- **[vitest](#vitest)**：Node.js 生态的快速单元测试框架，Vite 生态的测试工具。
- **[sandbox_runner](#sandbox-runner)**：Continuous-Claude-v3 实现的数学代码执行沙盒，设置 CPU/RAM 上限和超时，防止无限循环或资源耗尽。
- **[curl-pipe-sh](#curl-pipe-sh)**：`curl URL | sh` 或 `curl URL | bash` 模式，在 skill/agent 文件中出现时意味着 AI 被指示下载并直接执行远程脚本，是 NL 安全审查的 CRITICAL 模式。
- **[frontmatter](#frontmatter)**：Markdown 文件顶部 `---` 包围的 YAML 元数据块，Claude Code 用它识别 artifact 类型和声明权限字段（tools、allowed-tools、model 等）。
