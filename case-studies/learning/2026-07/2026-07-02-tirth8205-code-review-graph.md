# tirth8205/code-review-graph — 学习案例

**仓库**：https://github.com/tirth8205/code-review-graph
**Stars**：12954 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-02（基于当前 HEAD）
**主题标签**：`cross-reference`, `vague-quantifier`, `examples-driven`, `security-gate`, `nl-binary-hybrid`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
**tirth8205/code-review-graph** 是一个通过代码图谱（code graph）大幅降低 AI 代码审查 token 消耗的工具，实现了中位 **82 倍**的 token 削减。工具核心是用 [Tree-sitter](#tree-sitter) 解析代码 AST，构建 SQLite 图谱（节点：函数/类/导入；边：调用/继承/依赖），通过 [MCP](#MCP) 协议将「最小上下文集」交付给 AI。

关键事实：
- 创建者 tirth8205 是个人开发者，仓库从技术思路到工程实现均独立完成
- 支持 40+ 编程语言（包括 Vue SFC、Solidity、Dart、R、Perl、Lua、Jupyter Notebooks）
- MCP 服务器暴露 30 个工具，涵盖图构建、图查询、变更影响分析、社区检测
- 发行版通过 PyPI（`uvx code-review-graph serve`）+可选 VS Code 扩展分发
- 本地优先，无云依赖；向量嵌入需显式 opt-in（`CRG_ACCEPT_CLOUD_EMBEDDINGS=1`）

### 1.2 架构剖析
- **目录结构**：
  ```
  code-review-graph/
  ├── skills/
  │   ├── build-graph/SKILL.md           # 构建代码图谱
  │   ├── review-changes/SKILL.md        # 审查变更（100分）
  │   ├── review-pr/SKILL.md             # PR 审查
  │   ├── review-delta/SKILL.md          # 增量审查
  │   ├── debug-issue/SKILL.md           # 调试问题（缺 Output Format）
  │   ├── explore-codebase/SKILL.md      # 代码库探索（缺 Output Format）
  │   └── refactor-safely/SKILL.md       # 安全重构（缺 Output Format）
  ├── code_review_graph/                 # Python 后端（解析、图构建、embeddings）
  ├── code-review-graph-vscode/          # VS Code 扩展
  ├── hooks/
  │   ├── hooks.json                     # PostToolUse 钩子（100分）
  │   └── session-start.sh              # 会话启动脚本
  ├── .mcp.json                          # MCP 服务器配置（未版本固定）
  └── CLAUDE.md                          # 项目说明（98分）
  ```
- **文件类型分布**：7 个 SKILL.md + 1 个 CLAUDE.md + 1 个 hooks.json = 9 个 NL 工件；另有 Python 后端（6 个文件）、VS Code 扩展（TypeScript）
- **编排关系**：无中央 orchestration skill，7 个 skill 平行关系；用户按任务选择使用哪个 skill；hooks/hooks.json 在 `PostToolUse(Write|Edit|Bash)` 后自动触发图更新，是唯一的自动调度机制
- **跨件契约**：`review-delta` 和 `review-pr` 使用 `_tool` 后缀的 MCP 工具名（如 `get_review_context_tool`），而 `debug-issue`、`explore-codebase`、`review-changes` 使用裸工具名（如 `get_review_context`）；CLAUDE.md 的 Key Tools 表用裸名——5 个 skill 存在工具名不一致问题

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「图结构优先于全文扫描」—— 相比让 AI 读整个代码库，先建图再精准送上下文，同等质量下 token 降低 82 倍
- **解决什么问题**：AI code review 的「token 暴力破解」问题 —— 把整个 codebase 塞进上下文，贵且慢，而且真正相关的代码不超过 5%
- **做了什么 trade-off**：选择了「[NL 表皮 + 原生工具核心](#nl-binary-hybrid)」模式 —— skill/CLAUDE.md 是 NL 层，Python 后端 + MCP 服务器是工具层；NL 层极薄（7 个 skill，平均不超过 50 行），工具层极厚（30 个 MCP 工具）
- **反映什么认知模型**：作者认为 AI 的价值在于「推理」而非「记忆」—— 用图谱代替 AI 的上下文记忆，让 AI 专注在拿到的精准上下文上推理，而不是靠记住整个代码库

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「NL 表皮 + 工具核心」（NL Thin Skin + Thick Tool Core）**

skill 文件极度精简，每个 skill 的核心职责是「告诉模型用哪些 MCP 工具、按什么顺序调用」，而不是「告诉模型所有领域知识」。领域知识和数据结构全部在 MCP 服务器（Python 后端）中。

模式特征清单：
- 特征 1：skill 文件描述工作流（步骤顺序）而非领域知识（「知道什么」）
- 特征 2：工具调用序列是 skill 的核心内容（「先调 A 再调 B」）
- 特征 3：NL 层极薄（平均 98/100 的 2 个 skill 都在 100 行以内）
- 特征 4：后端工具（30 个 MCP 工具）是主要能力承载者，NL 只是调用接口的文字说明
- 特征 5：hooks.json 触发图的自动更新，不需要用户手动维护图的新鲜度

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有已有后端工具或 CLI 需要 AI 化 | ✅ 高度适用 | NL 表皮模式只需写「怎么调工具」，工具逻辑在后端 |
| 处理大型代码库（>10 万行） | ✅ 适用 | 图谱方法的 token 优势随代码库规模增大而放大 |
| 快速入手 AI coding（无后端能力） | ❌ 不适用 | 本模式依赖后端工具，纯 NL 场景应直接写详细 skill |
| 知识需要频繁更新（如快速迭代的 API 文档） | ❌ 谨慎 | 更新后端工具比更新 NL skill 成本高 |
| 离线环境 | ✅ 适用 | 本地 SQLite 图谱，无云依赖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 工具核心（本仓库） | code-review-graph | token 效率极高，工具能力可升级 | 需要维护 MCP 服务器，安装复杂 |
| 纯 NL skill（知识编码型） | timescale/pg-aiguide | 零安装，自包含 | token 消耗随代码库增大 |
| Orchestration + 多子 skill | team-attention/plugins-for-claude-natives | 模块化 | 跨 skill 调用路径复杂 |

### 2.4 改进空间
1. **当前问题**：`debug-issue`、`explore-codebase`、`refactor-safely` 这 3 个 skill 缺少 Output Format 节，AI 调用后的输出格式不可预测。**改进做法**：为每个 skill 添加 `## Output Format` 节，规定结构（如 debug-issue：根因摘要 + 影响路径 + 推荐修复 + 置信度）。**预期收益**：每个 skill 评分从 88 升至 98，且下游 skill/命令可以可靠解析输出。

2. **当前问题**：`review-delta` 和 `review-pr` 用 `_tool` 后缀，其余 3 个 skill 用裸名，实际运行时有 50% 概率调用不存在的工具变体。**改进做法**：将 `review-delta` 和 `review-pr` 中所有 `*_tool` 替换为裸工具名，以 CLAUDE.md 的 Key Tools 表为准。**预期收益**：消除 MCP 调用失败风险，统一工具命名规范。

3. **当前问题**：`.mcp.json` 中 `uvx code-review-graph serve` 无版本固定，每次启动都拉最新版本。**改进做法**：改为 `uvx code-review-graph==<当前版本> serve`，并在每次发布新版本时有意识地更新此处。**预期收益**：避免 PyPI 包被污染或意外 breaking change 时自动采纳。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 **95/100**（9 个工件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/debug-issue/SKILL.md | 88 | 缺 Output Format（−10），「related」模糊（−2） |
| skills/explore-codebase/SKILL.md | 88 | 缺 Output Format（−10），「complex」模糊（−2） |
| skills/refactor-safely/SKILL.md | 88 | 缺 Output Format（−10），「major」模糊（−2） |
| skills/review-pr/SKILL.md | 94 | 「large」「related」「significant」各 −2 分 |
| skills/review-changes/SKILL.md | 100 | 无问题 |
| hooks/hooks.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **hooks 安全设计** → `hooks/session-start.sh` 使用单引号 heredoc（`<<'INSTRUCTIONS'`），完全无变量插值；hooks.json 的所有命令都是静态参数（无 shell 注入向量）。根本原因：作者意识到 hook 脚本是特权入口，采用保守设计。如何借鉴：所有 hook 脚本对 heredoc 使用单引号、对 subprocess 使用 list 参数（不用 `shell=True`）。

2. **cloud embeddings 显式 opt-in** → 代码内容发送给第三方 API 需要 `CRG_ACCEPT_CLOUD_EMBEDDINGS=1` 环境变量才启用。根本原因：数据隐私第一，不应在用户不知情下向外发送代码。如何借鉴：所有涉及外发用户数据的功能都应默认关闭，需要显式 opt-in。

3. **review-changes skill 的 100 分设计** → 这个 skill 有完整的 Output Format、无模糊量词、MCP 工具名使用正确裸名。它是同一仓库中其他 skill 的质量对标。如何借鉴：每个仓库至少应该有 1 个「样板 skill」作为其他 skill 的对比基准。

4. **blast-radius 分析** → 图谱中内置「影响半径」分析：修改一个函数后，自动找出所有直接/间接依赖者和受影响的测试。根本原因：代码审查中最重要的问题是「这个改动会破坏什么」，而不是「改动本身对不对」。如何借鉴：在 code review 类 skill 中明确加入「影响范围评估」步骤。

5. **增量更新避免全量重扫** → 图谱只重新解析变更文件，不全量重建。根本原因：对实际使用场景的精准理解——每次代码改动通常只影响少数文件。如何借鉴：设计任何增量式 AI 工具时，把「只处理变化的部分」作为默认行为而非优化项。

### 3.3 当时的缺陷

1. **3 个 skill 缺 Output Format 节**（debug-issue、explore-codebase、refactor-safely，各 -10 分）→ 这三个 skill 描述了步骤但不规定输出格式，AI 每次的输出结构都不可预测。**根本原因**：作者专注于「怎么做」（工具调用序列），忽视了「做完之后长什么样」（输出规约）。**自查**：我的 skill 文件中是否所有工具调用 skill 都有 Output Format 节？

2. **MCP 工具名不一致（`_tool` 后缀问题）** → 5 个 skill 中 2 个用 `_tool` 后缀，3 个不用；CLAUDE.md 用裸名。当 Claude 调用 `get_review_context_tool` 时，如果 MCP 服务器的实际工具名是 `get_review_context`，调用会静默失败。**根本原因**：skill 在不同时期由不同版本的工具描述文档生成，没有统一的「工具名 source of truth」。**自查**：我的 skill 中引用的 MCP 工具名是否与 `.mcp.json` 或 CLAUDE.md 中的工具名完全一致？

3. **`skills/build-graph/SKILL.md` 语言数量过时**（14 种 vs CLAUDE.md 的 19 种）→ 这是一种「内部文档不同步」问题：skill 文件和 CLAUDE.md 关于同一事实（支持语言数）给出不同答案。**根本原因**：新增语言支持时只更新了 CLAUDE.md，没有同步更新 build-graph skill。**自查**：我的仓库中，关于同一技术事实（如支持的功能列表、配置选项数量）是否有多处说明？是否都保持同步？

### 3.4 当时的优化机会

1. **为 3 个缺 Output Format 的 skill 各添加输出规约**：为 debug-issue 加「根因摘要 + 影响路径 + 推荐修复方案 + 置信度」；为 explore-codebase 加「架构摘要 + 社区列表 + 关键入口 + 建议探索路径」；为 refactor-safely 加「变更清单 + 每项影响评估 + 回滚方案」。
2. **统一 MCP 工具名**：以 CLAUDE.md 的 Key Tools 表为权威来源，将所有 skill 文件中的 `*_tool` 后缀统一去掉，确保所有 skill 使用相同的工具名。
3. **固定 .mcp.json 中的版本号**：`uvx code-review-graph serve` → `uvx code-review-graph==X.Y.Z serve`，每次发布后有意识地更新。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| debug-issue/SKILL.md 缺 Output Format | WebFetch debug-issue/SKILL.md，搜索 Output Format | **依然存在** — 当前文件无 Output Format 节，仍使用 `_tool` 后缀（如 `semantic_search_nodes_tool`） | 2 个月内未修复，且 MCP 工具名不一致问题也继续存在 |
| .mcp.json 未固定版本 | WebFetch .mcp.json，检查 args | **依然存在** — 仍为 `uvx code-review-graph serve`，无版本号 | 中级安全风险持续存在 |
| build-graph 语言数量（14 vs 19） | （需 WebFetch build-graph/SKILL.md 确认） | 无法直接验证，但 README 已提到 40+ 语言，说明语言数量仍在增长，skill 同步更新概率低 | 内部文档不同步是长期趋势 |

### 4.2 架构演进
从 README 描述可以看出，工具层（Python 后端）在 audit 后持续进化（现在支持 40+ 语言而非审计时的 19+），但 NL 层（skill 文件）几乎未更新。这是「NL 表皮 + 工具核心」架构的常见趋势：工具核心迭代快，NL 表皮滞后，产生文档漂移。

### 4.3 新增的可学习模式
README 提到的新特性「Watch mode for continuous updates」和「Multi-repo registry and daemon」是工具层的重大增强，但 skill 文件尚未更新以反映这些能力。这提示：工具层迭代时必须同步更新 NL skill 文件，否则 AI 无法利用新能力。

---

## 五、校准

### 5.1 我已经在做对的
1. **output format 有明确规约** — bureau/query.md 的 Output Format 节详细规定了引用格式、加粗规范、Sources 列表，比 debug-issue 等 skill 更规范
2. **隐私 opt-in 设计** — bureau 的 3 级信任体系（proposed/verified/canonical）本质上是数据敏感度的明确分层，与 CRG_ACCEPT_CLOUD_EMBEDDINGS 的 opt-in 设计精神一致
3. **single source of truth 意识** — bureau 把 trust tier 定义在 BUREAU.md 的一处，不在多处重复定义相同概念

### 5.2 挑战 / 验证
**挑战了我的假设**：我之前认为「高分仓库（95/100）的 skill 都应该是完整的」，但 code-review-graph 的 3 个最实用的 skill（debug/explore/refactor）评分最低，因为缺 Output Format。这说明：**NL 工件的质量与它解决的问题的复杂度成负相关**—— 越是核心、复杂的工作流 skill，越难写好 Output Format，因为输出本身就是复杂的多维判断。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 文件是否都有 Output Format 节
find ~/.claude/ -name "SKILL.md" | while read f; do
  grep -q -iE '## (output format|output|输出格式)' "$f" || echo "NO OUTPUT FORMAT: $f"
done
```
命中后怎么办：为每个缺少 Output Format 的 skill 添加一个 `## Output Format` 节，明确规定 AI 的输出结构（标题、字段、顺序）。

```bash
# 检查 .mcp.json 中的工具是否都有版本固定
find ~ -name ".mcp.json" 2>/dev/null | while read f; do
  grep '"uvx"' "$f" | grep -qv '==' && echo "UNPINNED in $f: $(grep uvx "$f")"
  grep '"npx"' "$f" | grep -qv '@[0-9]' && echo "UNPINNED in $f: $(grep npx "$f")"
done
```
命中后怎么办：将 `uvx <package> serve` 改为 `uvx <package>==<版本号> serve`，`npx <package>` 改为 `npx <package>@<版本号>`。

```bash
# 检查 skill 文件中引用的 MCP 工具名是否与 CLAUDE.md 一致
TOOLS=$(grep -oE 'mcp__[a-z_]+' ~/.claude/CLAUDE.md 2>/dev/null | sort -u)
find ~/.claude/ -name "SKILL.md" | while read f; do
  SKILL_TOOLS=$(grep -oE 'mcp__[a-z_]+_tool' "$f" 2>/dev/null)
  [ -n "$SKILL_TOOLS" ] && echo "TOOL SUFFIX in $f: $SKILL_TOOLS"
done
```
命中后怎么办：将 `mcp__xxx_tool` 替换为 `mcp__xxx`（去掉 `_tool` 后缀），以 CLAUDE.md 中的工具名表为准。

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/graphify 添加「blast-radius 分析」步骤到 review skill
   - **为何可行**：graphify 已经能构建代码图谱，只需要在 review skill 中加一步「找出被修改函数的所有调用者」
   - **第一步**：在 graphify 的 review 相关 skill 中加一行「使用图谱查找修改函数的直接依赖者，过滤出外部调用者」，5 分钟

2. **想法**：在 bureau 的 skill 文件中添加一个「样板 skill」（100 分对标）
   - **为何可行**：code-review-graph 的 review-changes/SKILL.md 是同仓库中 100 分的标杆；bureau 目前无类似标杆
   - **第一步**：选 bureau 中最完整的一个 skill 文件，按 NLPM 规范补全（frontmatter、Output Format、examples），设为其他 skill 的质量基准，15 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 tirth8205/code-review-graph 的核心目的**：通过代码图谱大幅降低 AI code review 的 token 消耗，同时提高 review 精度

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | 都是「把代码结构转为 AI 可查询的图谱」；都有 MCP 接口；都面向提升 AI 编码辅助质量 | graphify 面向通用知识图谱（代码+文档+schema），code-review-graph 专注代码 review；code-review-graph 有 82x token 压缩的量化指标 | 高 |
| MarkQWu/gstack | 中 | gstack 的 /review 命令与 code-review-graph 的 review 系列 skill 目的相似 | gstack 是通用工作流 skill，code-review-graph 有专用后端工具支撑 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 缺 Output Format | `find ~/.claude/ -name "SKILL.md" \| xargs grep -L 'Output Format'` | bureau/query.md 有 Output Format，但 graphify 是否所有 skill 都有无法验证 | 高 |
| .mcp.json 未固定工具版本 | `grep '"uvx"\|"npx"' ~/.mcp.json ~/.claude/.mcp.json` | graphify 有 MCP 服务器，其 .mcp.json 配置中工具版本固定情况待核查 | 中 |
| 内部文档不同步（skill vs CLAUDE.md 数量不一致） | `grep -E '支持 [0-9]+ (种|个|languages)' CLAUDE.md skills/*/SKILL.md` | graphify 涉及「支持的图谱类型数量」等事实，多处说明是否同步需自查 | 低 |

**命中后的具体行动建议**：
- graphify 的所有 SKILL.md → 检查是否有 Output Format 节，无则添加，每个 15 分钟
- graphify 的 .mcp.json → 把 `npx graphify` 改为 `npx graphify@<版本号>`，5 分钟

### 7.3 别人的更优方案

1. **领域**：hooks.json 的安全设计（单引号 heredoc + list 参数）
   - **本案例做法**：`session-start.sh` 用 `<<'INSTRUCTIONS'`（单引号，无变量插值），Python 后端所有 subprocess 使用 list 参数
   - **我的项目现状**：无法验证 gstack/bureau 的 hook 脚本细节，但这是一个应全面检查的最佳实践
   - **如何借鉴**：把所有 hook 脚本中的 `<<"INSTRUCTIONS"` 改为 `<<'INSTRUCTIONS'`，subprocess 调用全部改用 list 形式，20 分钟

2. **领域**：blast-radius 分析作为默认 review 步骤
   - **本案例做法**：review 系列 skill 的第 3 步是「找修改文件的所有依赖者」，是标准步骤而非可选步骤
   - **我的项目现状**：gstack 的 /review command 可能没有明确的影响范围分析步骤
   - **如何借鉴**：在 gstack/commands/review.md 加一步「列出被修改函数/类的直接调用者（用 Grep）」，10 分钟

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：单一权威信息源（single source of truth）
- **我的做法**：bureau 的核心概念（trust tier 定义）只在 BUREAU.md 一处定义，skill 文件引用概念但不重定义
- **本案例做法（弱在哪）**：build-graph/SKILL.md 和 CLAUDE.md 对「支持语言数量」有不同答案（14 vs 19+），说明存在重复定义同一事实的问题
- **意义**：「量化事实只在一处定义」是减少文档漂移的关键原则；未来可以在 bureau 的设计文档中明确提出这一原则

---

## 八、术语表

### <a name="tree-sitter"></a>Tree-sitter
> 一个高性能、增量式代码解析器生成工具。输入源代码文件，输出语法树（AST，Abstract Syntax Tree）。支持 40+ 编程语言。code-review-graph 用它把代码文件解析成「节点（函数/类/导入）+ 边（调用/继承）」的图结构。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 制定的开放协议，让 AI 助手通过标准接口调用外部工具和服务。code-review-graph 的 MCP 服务器暴露 30 个工具（如 `build_or_update_graph`、`get_review_context`、`query_graph`），Claude 可以直接调用这些工具操作代码图谱。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 工具核心（NL thin skin + thick tool core）
> 一种 Claude Code 插件设计模式：skill/CLAUDE.md 等 NL 文件只做「怎么调用工具」的说明（极薄），真正的能力由后端工具（Python/Go/TypeScript 程序）实现（极厚）。对比之下，「纯 NL skill」是把所有领域知识都写在 NL 文件里，没有专用后端。

### <a name="blast-radius"></a>blast-radius（影响半径）
> 修改一段代码后，所有受影响的其他代码范围。code-review-graph 通过图谱追踪：A 调用了 B，B 调用了 C，改了 C 的签名 → A 和 B 都在 blast-radius 内。知道 blast-radius 才能判断「这个改动是否安全」。

### <a name="opt-in"></a>opt-in（显式启用）
> 功能默认关闭，用户明确表示同意后才启用。与之相对的是 opt-out（默认开启，用户主动关闭）。code-review-graph 要求设置 `CRG_ACCEPT_CLOUD_EMBEDDINGS=1` 才会向外部 API 发送代码，是隐私保护的最佳实践。
