# wanshuiyin/Auto-claude-code-research-in-sleep — 学习案例

**仓库**：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
**Stars**：6387 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-07-04（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `cross-reference`, `single-purpose`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
**ARIS（AI Research In Sleep）** 是一套为 Claude Code、Codex CLI、Cursor 等多平台设计的学术研究自动化 skill 套件，让 AI 在用户「睡觉时」自主完成文献综述、论文写作、实验规划、审稿、重写等科研全流程。

关键事实：
- HuggingFace Daily Paper 排名第一（2026 年某日），有 arXiv 技术报告（arXiv:2605.03042）
- 100 个 NL 工件（全部是 skill），分布在 4 个 skill 树：`skills/`、`skills-codex/`、`skills-codex-claude-review/`、`skills-codex-gemini-review/`
- 目前总 skill 文件夹数增至 83 个（audit 时约 100 个工件，当前树结构有所调整）
- 安全状态：**BLOCKED**——`tools/save_trace.sh` 中存在 Python 代码注入漏洞（CRITICAL）
- 多语言（中英双语文档），面向中国学术社区为主

### 1.2 架构剖析
```
Auto-claude-code-research-in-sleep/
├── skills/                          ← 主 skill 树（Claude Code，完整 frontmatter）
│   ├── research-refine/SKILL.md     ← 研究方案打磨（最高质量，明确常量/循环/状态持久化）
│   ├── paper-plan/
│   ├── paper-write/
│   ├── experiment-plan/
│   ├── auto-review-loop/
│   └── ... （83 个总计，含专利/授权申请类）
├── skills/skills-codex/             ← Codex CLI 版本（缺 allowed-tools）
├── skills/skills-codex-claude-review/   ← Claude review 变体（8 个）
├── skills/skills-codex-gemini-review/   ← Gemini review 变体（3 个）
├── tools/                           ← Python 工具脚本
│   ├── save_trace.sh                ← ⚠️ CRITICAL：shell var 插入 Python 字符串
│   └── experiment_queue/
│       └── queue_manager.py         ← ⚠️ HIGH：shell=True subprocess
├── mcp-servers/                     ← 多个 MCP 服务（Feishu、Gemini、MiniMax 等）
└── AGENT_GUIDE.md                   ← AI 专用文档（结构化供 LLM 消费）
```

- **文件类型分布**：100 个 Skill（全部）；无 command/agent；多个 Python MCP 服务器
- **编排关系**：单 skill 平级，无内置路由层。复杂工作流（如 `research-refine-pipeline`、`dse-loop`）通过 skill body 内的顺序 phase 实现编排，不依赖独立 orchestrator
- **跨件契约**：skills-codex 变体引用 `../shared-references/` 路径，若单独分发会断链；主 `skills/` 树是参考源

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「状态持久化 + 多轮循环 + 跨模型审查」——ARIS 的核心思想是把单次 LLM 调用变成有状态、有审查、可恢复的长程工作流。skill 内部管理循环（`while score < threshold`）、状态文件（`REFINE_STATE.json`）、外部审查器（GPT-5.5 via Codex MCP）
- **解决什么问题**：学术研究需要多轮迭代（想法→审查→修改→再审查），单次 LLM 调用质量不可靠；同时 session 可能中途中断，需要恢复机制
- **做了什么 trade-off**：技术复杂度（状态文件、MCP、multi-LLM 审查）换来研究自动化深度。用于日常开发会是严重过度设计
- **反映什么认知模型**：把 AI agent 当「可恢复的研究流水线」——不是单次调用，而是带状态、可中断、可恢复、可外部审查的长程任务

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「自驱迭代研究流水线（Self-Driving Research Pipeline）」**

skill 内部封装完整的迭代循环逻辑：初始化 → Phase 循环 → 状态持久化 → 外部审查 → 收敛判断，整个工作流无需用户干预。

模式特征清单：
- 特征 1：skill 顶部声明显式常量（`REVIEWER_MODEL`、`MAX_ROUNDS`、`SCORE_THRESHOLD`），可 override
- 特征 2：循环逻辑在 skill body 内（`while score < SCORE_THRESHOLD or rounds < MAX_ROUNDS`）
- 特征 3：每个 phase 边界写入状态文件（`REFINE_STATE.json`），支持 checkpoint 恢复
- 特征 4：引入外部审查器（GPT-5.5 via Codex MCP）做盲审，避免 self-evaluation 偏差
- 特征 5：`OUTPUT_DIR` 显式声明，所有中间产物和最终报告写入固定目录

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 学术研究自动化（综述/论文/实验） | ✅ 核心用途 | 多轮迭代、长时间运行的场景天然需要这种结构 |
| 工程代码生成/review | ⚠️ 需要简化 | 状态文件和外部审查对一般工程任务是过度设计 |
| 日常 CLI 小工具 | ❌ 完全不适用 | 维护成本高于收益 |
| 多模型互审系统 | ✅ 适用 | cross-model auditing 是 ARIS 的核心优势之一 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 自驱迭代流水线（本仓库） | wanshuiyin/ARIS | 深度自动化、可恢复、多轮审查 | 复杂度高，维护困难，安全面广 |
| 简单自主迭代（autoresearch） | uditgoenka/autoresearch | 多子命令，用户可控 | 无跨模型审查，无状态持久化 |
| 单次 skill 调用 | 多数 skill | 简单可靠 | 无迭代能力 |

### 2.4 改进空间
1. **当前问题**：`tools/save_trace.sh` 中 `python3 -c "... '${PURPOSE}' ..."` 把 bash 变量插入 Python 字符串，用户控制的 CLI 参数可以逃逸执行任意 Python 代码。**改进做法**：用 `python3 -c "import sys; purpose = sys.argv[1]; ..."` 传参，完全消除字符串注入路径。**预期收益**：消除 CRITICAL 漏洞，解除 BLOCKED 状态。
2. **当前问题**：31 个 skills-codex 文件（audit 后仍有 31/79 缺 `allowed-tools`）。**改进做法**：写一个 CI 检查，要求所有 `skills-codex/*/SKILL.md` 必须有 `allowed-tools` 字段，否则 PR 不通过。**预期收益**：自动阻止漏写。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **81/100**（100 个文件加权平均）。

| 文件组 | 当时分数区间 | 主要问题 |
|---|---|---|
| `skills-codex/idea-discovery-robot/SKILL.md` | 74/100 | 缺 allowed-tools + 5 个模糊量词（-10-10） |
| `skills-codex/*` 大多数文件（40 个）| 74-85/100 | 缺 allowed-tools（-5）+ 模糊量词 |
| `skills/research-refine/SKILL.md` | 85/100 | 最高分，结构最完整 |
| `skills/system-profile/SKILL.md` | 75/100 | 缺 allowed-tools（-5）+ 3 个模糊量词 |

### 3.2 当时值得借鉴的模式
1. **显式常量声明** → `REVIEWER_MODEL = gpt-5.5`、`MAX_ROUNDS = 5`、`SCORE_THRESHOLD = 9` 集中在 skill 顶部，可 override → `skills/research-refine/SKILL.md` `## Constants` 节 → 任何有「上限/阈值/外部依赖」的 skill 都应该这样声明常量
2. **状态持久化与 checkpoint 恢复** → 每个 phase 边界写入 `REFINE_STATE.json`，记录 `{phase, round, threadId, last_score, status}` → `research-refine/SKILL.md` `## State Persistence` 节 → 长时间运行（>5 轮）的 skill 应该有状态文件
3. **Phase 边界明确命名** → `Phase 0: Freeze Problem Anchor`、`Phase 1: Scan grounding papers...`，每个阶段目的清晰 → research-refine skill 第 40 行起的 Overview 图 → 多步骤 skill 应该用 Phase 而非无序 step 序列
4. **`AGENT_GUIDE.md` 作为 AI 友好文档** → 专门为 LLM 消费写的结构化文档，与 README（给人看）分离 → 顶层 `AGENT_GUIDE.md` → 复杂项目可以分开写「人类文档」和「AI 文档」
5. **多平台分发声明** → skill body 开头明确告知适用平台（Claude Code/Codex CLI/Cursor/Trae/Antigravity/...），减少用户困惑

### 3.3 当时的缺陷
1. **Python 代码注入（CRITICAL）**：`tools/save_trace.sh` 把 `--purpose`、`--model`、`--skill` CLI 参数直接插入 `python3 -c "... '${PURPOSE}' ..."` 字符串 → 根本原因：shell 脚本的字符串拼接无类型系统保护，作者没有意识到用户可以传入 `'; __import__("os").system("id"); x='` 这样的值 → 自查：我的任何脚本是否有 `python3 -c "... $VAR ..."` 模式？
2. **51 个文件缺少 `allowed-tools`**：skills-codex 变体全线缺失 → 根本原因：作者为 Claude Code 版本（`skills/`）正确填写了 allowed-tools，但在生成 Codex 变体时没有同步这个字段（可能认为 Codex 不需要） → 自查：我的多平台分发是否漏掉了 allowed-tools 同步？
3. **模糊量词高密度**：`appropriate`、`ensure`、`various`、`relevant` 在几乎所有 skill 中高频出现（部分文件 10 次以上） → 根本原因：学术写作风格习惯了这类「修饰性程度副词」，但 AI 需要边界清晰的指令 → 这与人类对学术写作「体面感」的期望和 AI 执行「可操作性」之间的张力是本质矛盾

### 3.4 当时的优化机会
1. 修复 `save_trace.sh` Python 代码注入（用 sys.argv 传参）
2. 为 51 个 skills-codex/claude-review/gemini-review 文件补全 `allowed-tools`
3. 建立 CI 规则阻止模糊量词超过阈值的文件合并

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `save_trace.sh` Python 代码注入（CRITICAL） | `grep -n "python3 -c" tools/save_trace.sh` | ❌ **仍存在**（第 113、130、146、162 行全部命中） | CRITICAL 未修复，BLOCKED 状态持续 |
| `skills-codex/` 缺 allowed-tools | `find skills/skills-codex -name "SKILL.md" \| xargs grep -L "allowed-tools" \| wc -l` | **部分修复**：79 个文件中 48 个已有 allowed-tools，**仍有 31 个缺失** | 修了 28 个（audit 时约 40 缺失→现在 31 缺失），但未完成 |
| 模糊量词 | `grep -c "appropriate\|ensure\|relevant" skills/research-refine/SKILL.md` | **未系统修复**，部分文件仍有大量命中 | 高密度模糊量词是此仓库的「性格特征」，不像是未来会修复的 |

### 4.2 架构演进
从 audit 时的约 100 个工件到现在 83 个 skill 文件夹（`skills/` 下）：

- 新增了专利相关 skill 类（`patent-review`、`patent-pipeline`、`patent-novelty-check`、`claims-drafting`、`invention-structuring`）——说明 ARIS 从纯学术研究扩展到了知识产权保护场景
- 新增 `jurisdiction-format`（专利司法格式化）、`kill-argument`（反驳论证）、`prior-art-search`（现有技术检索）
- `tools/smart_update.sh` 继续存在（部分自动化同步），但 skills-codex 仍有 31 个文件未同步 `allowed-tools`

这说明作者在持续扩展功能范围（专利 → 学术），但 NL 质量债务（allowed-tools 缺失、模糊量词）在积累而非消减。

### 4.3 新增的可学习模式
**专利 + 学术双轨 skill 系统**：新增的专利类 skill（`patent-pipeline`、`claims-drafting`）与学术 skill 遵守完全相同的结构约定，说明 ARIS 的 skill 模板有足够的通用性可以跨领域复用。这验证了「统一的 skill 结构约定 + 领域专用知识」的组合是可扩展的。

---

## 五、校准

### 5.1 我已经在做对的
1. **显式常量声明**：bureau 的 `compile/SKILL.md` 有信任等级常量（`TRUST_TIER: 1|2|3`）集中声明
2. **Phase 边界命名**：gstack 的多个 skill 有明确 phase 序列（如 `scout → analyze → report`）
3. **无 `save_trace.sh` 类型脚本**：我的脚本不把用户输入直接插入 python3 -c 命令
4. **AGENT_GUIDE.md 模式**：gstack 的 `AGENTS.md`（面向 AI 的项目文档）与 ARIS 的 `AGENT_GUIDE.md` 同一思路

### 5.2 挑战 / 验证
**验证**：我一直觉得「学术写作风格」的模糊量词在专业语境下是必要的（`comprehensive review`、`appropriate methodology`）。但 ARIS 案例证明：即便是 arXiv 发表、HuggingFace 第一的顶级项目，这些词同样被评为 NL 质量问题，而且作者在 18 个月内没有修复——说明这不是「写得对不对」的问题，而是「领域惯例 vs AI 可执行性」之间有根本张力，无论项目多受欢迎，这个张力都存在。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的脚本是否有 python3 -c + bash 变量插值的危险模式
grep -rn "python3 -c.*\$" /tmp/my-repos/MarkQWu-gstack \
  /tmp/my-repos/MarkQWu-bureau --include="*.sh" 2>/dev/null
```
命中后：立即改为 `sys.argv` 或临时文件传参，这是 CRITICAL 级别漏洞。

```bash
# 检查研究/分析类 skill 是否缺少状态持久化机制
grep -rn "REFINE_STATE\|state\.json\|checkpoint" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude \
  /tmp/my-repos/MarkQWu-bureau --include="SKILL.md" 2>/dev/null
```
命中后：为长时间运行的 skill（预计运行 >5 轮的分析）添加状态文件写入步骤。

### 6.2 灵感 → 实施路径
1. **想法**：为 echo-sleuth-for-claude 的长分析 skill 引入「状态持久化 + checkpoint 恢复」
   - **为何可行**：echo-sleuth 可能分析大量历史对话，session 中断是常见场景；ARIS 的 `REFINE_STATE.json` 模式可以直接借鉴
   - **第一步**：在最长的 skill（`skills/mine/SKILL.md`？）末尾加 `## State Persistence` 节，定义 `MINE_STATE.json` 的结构（`{phase, processed_count, last_session_id}`）→ 预计 20 分钟

2. **想法**：在 gstack 的 SKILL.md 末尾加「显式常量」节
   - **为何可行**：gstack 的几个 skill（如 `plan-devex-review`）有隐式阈值（评分标准），用 Constants 节显式化可以让用户按需 override
   - **第一步**：参考 ARIS 的 `## Constants` 格式，在 `gstack/plan-devex-review/SKILL.md` 中把 review 阈值、评分维度权重提取为 constants → 预计 15 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 wanshuiyin/ARIS 的核心目的**：学术研究全流程自动化（综述、写作、实验、审稿），多模型互审，睡眠中运行

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是多轮分析、挖掘深层洞察 | echo-sleuth 分析对话历史，ARIS 写新论文 | 中 |
| MarkQWu/bureau | 低 | 都有知识积累机制（research-wiki vs canon） | bureau 是知识管理，不做研究自动化 | 低 |
| MarkQWu/shiji-kb | 中 | 都涉及大规模文本分析（古诗 vs 学术文献） | shiji-kb 是知识库，无自动化研究流水线 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词高密度 | `grep -c "appropriate\|ensure\|relevant\|various" gstack/*/SKILL.md` | **gstack 多个 SKILL.md 每个命中 5-20 次**（ios-design-review: 8 次，document-release: 12 次） | 中 |
| 无状态持久化 | `grep -rL "state.json\|REFINE_STATE" echo-sleuth/skills/*/SKILL.md` | echo-sleuth 的分析 skill 无状态文件 | 中（仅在长任务场景下高严重度） |
| Python 代码注入（shell var + python3 -c） | `grep -rn "python3 -c.*\$" my-repos --include="*.sh"` | **未命中**（bureau、gstack 无此模式） | 无 |

**命中后的具体行动建议**：
- `gstack/ios-design-review/SKILL.md` → 把 `appropriate`, `relevant` 替换为具体的 Apple HIG 版本号或像素/pt 数值 → 10 分钟
- `echo-sleuth/skills/mine/SKILL.md`（或同等文件）→ 加 State Persistence 节 → 20 分钟

### 8.3 别人的更优方案

1. **领域**：显式常量 + override 机制
   - **本案例做法**：`skills/research-refine/SKILL.md` 的 `## Constants` 节：`REVIEWER_MODEL = gpt-5.5`、`MAX_ROUNDS = 5`（可 override：`-- max rounds: 3`）
   - **我的项目现状**：gstack 的 `plan-devex-review/SKILL.md` 有隐式的评分标准，但没有 Constants 节，用户无法 override
   - **如何借鉴**：在有阈值/模型选择/轮数上限的 skill 中加 `## Constants` 节 → 更改时只需改一处

2. **领域**：`AGENT_GUIDE.md`（AI 专用文档）
   - **本案例做法**：顶层 `AGENT_GUIDE.md` 按「结构化 LLM 消费」格式编写，与 README 完全分离
   - **我的项目现状**：gstack 的 `AGENTS.md` 已经有这个概念，但 bureau 和 echo-sleuth 没有
   - **如何借鉴**：为 bureau 创建 `AGENT_GUIDE.md`，用 h2/h3 结构化说明 skill 触发时机和返回格式 → 30 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全的脚本参数传递
- **我的做法**：bureau 和 gstack 的脚本使用 `"$1"`、`"$2"` 位置参数传递路径，不把 bash 变量插入字符串命令
- **本案例做法**：`tools/save_trace.sh` 的 `python3 -c "... '${PURPOSE}' ..."` 是 CRITICAL 级别漏洞，至今未修复（audit 后 18 个月）
- **意义**：6387 stars 的知名项目也会有这样的基础安全问题，说明「流行」和「安全」不相关。我的脚本在这个维度上更安全，这是值得保持的实践。

---

## 八、术语表

### <a name="checkpoint-recovery"></a>状态持久化与 checkpoint 恢复
> 在多轮、长时间运行的任务中，每完成一个阶段（phase）就把当前状态（已完成哪些阶段、当前分数、外部服务的线程 ID）写入 JSON 文件。session 中断后，下次运行时先读取状态文件，从中断点继续，而非从头开始。ARIS 用 `REFINE_STATE.json` 实现这一机制。

### <a name="cross-model-audit"></a>跨模型互审（cross-model audit）
> 用不同于生成者的 AI 模型来审查生成结果，避免自我评价偏差（self-evaluation bias）。例如：Claude 生成研究方案 → GPT-5.5 盲审打分 → Claude 根据分数修改 → 循环直到分数达标。ARIS 的核心创新之一是把这种跨模型审查结构化为 skill 内的 Phase 循环。

### <a name="python-injection"></a>Python 代码注入（Python code injection）
> 把外部输入（用户参数、环境变量）直接插入 `python3 -c "..."` 命令字符串中执行的漏洞。攻击者可以通过构造特殊的参数值（如 `'; import os; os.system("rm -rf /"); x='`）来执行任意 Python 代码。修复方法：改用 `sys.argv` 传参，完全避免字符串拼接。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md frontmatter 字段，声明该 skill 运行时被允许调用的工具列表。在 Codex CLI 等受限环境中，缺少此字段可能导致工具调用失败或触发额外确认弹窗。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> 在 AI 指令中使用的不可测量修饰词，如 `appropriate`（合适的）、`ensure`（确保）、`relevant`（相关的）、`comprehensive`（全面的）。问题：对人类直觉清晰，但 AI 没有操作边界来判断何时算「足够合适」或「够全面」，导致每次执行标准不一致。NL 编程规范（R01）要求用可测量表达替代（如「覆盖 9 个维度」替代「全面覆盖」）。
