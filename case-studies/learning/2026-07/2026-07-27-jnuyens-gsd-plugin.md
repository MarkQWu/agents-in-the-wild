# jnuyens/gsd-plugin — 学习案例

**仓库**：https://github.com/jnuyens/gsd-plugin
**Stars**：9 | **来源**：upstream audit
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-07-27（基于当前 HEAD）
**主题标签**：`model-pinning`, `experience-accumulation`, `cross-reference`, `vague-quantifier`, `router-channels`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

gsd-plugin（"Buildomator"）是一个 **MCP 状态机驱动的结构化工作流插件**，在 Claude Code 之上实现了「计划 → 执行 → 验证」三阶段工程工作流，并用自定义 [MCP 服务器](#mcp-server)（`gsd`）持久化项目状态。

三个关键事实：
1. **规模最大的个人开发者插件之一**：115 个 NL 工件（33 agents + 82 skills），版本号 v4.4.0，homepage 是 `buildomator.com`——这是一个有商业意图的工程插件。
2. **`@${CLAUDE_PLUGIN_ROOT}` 跨件引用体系**：几乎每个 agent 和 skill 通过 `@${CLAUDE_PLUGIN_ROOT}/references/*.md`、`@${CLAUDE_PLUGIN_ROOT}/workflows/*.yaml` 引用共享文档；这个变量只在安装后由 Claude Code 解析，raw git clone 看到的是未展开的占位符。
3. **ALL 33 agents 零 model 声明**：这是单个最高优先级修复——33 个 agents × -5 分 = -165 分惩罚，一次批量修复就能把 agent 均分从 88.7 提升至约 93.7。

### 1.2 架构剖析

**目录结构（当前 HEAD，v4.4.0）**：
```
gsd-plugin/
├── agents/ (33 个)               # GSD 专家 agents（planner、debugger、verifier 等）
├── skills/ (82 个)               # GSD 操作入口（plan-phase、execute-phase、debug 等）
├── references/ (15 个)           # 共享参考文档（cross-reference 核心）
│   ├── mandatory-initial-read.md # 每个 agent/skill 启动时必须读取
│   ├── worktree-path-safety.md   # Git worktree 路径安全规则
│   ├── model-profiles.md         # 每个 agent 推荐的 model tier
│   ├── agent-contracts.md        # agent 间接口契约
│   ├── user-profiling.md         # 用户偏好配置
│   └── ...
├── schema/
│   ├── handoff-v1.json           # session 交接状态 JSON Schema
│   └── fixtures/                 # 测试夹具
├── mcp/
│   └── server.cjs                # GSD MCP 服务器（Node.js，项目状态持久化）
├── workflows/ (多个)             # 工作流配方（被 agents/skills @引用）
├── templates/                    # 输出模板（agent 产出的标准格式）
├── dist/bm/                      # 分发包（Buildomator 品牌）
└── .claude-plugin/
    └── plugin.json               # 插件清单（MCP 服务器声明 + skills/agents 列表）
```

**文件类型分布**：33 个 agent / 82 个 skill / 0 个 hook / 1 个 MCP 服务器 / 15 个 references

**编排关系**：  
Skills 是用户入口（`/gsd:plan-phase`、`/gsd:execute-phase`、`/gsd:debug`），skills 内部 dispatch agents（`gsd-planner`、`gsd-executor`、`gsd-debugger`）。Agents 间通过 `references/agent-contracts.md` 定义接口，通过 `mcp__gsd__*` 工具与 GSD MCP 服务器交换状态（`.planning/HANDOFF.json`）。整体是**三层编排架构**：用户 → skills → agents → MCP 状态机。

**跨件契约**：  
`gsd-project-researcher` agent 产出 6 个报告文件，`gsd-research-synthesizer` agent 消费这些文件——**生产者/消费者靠文件名称隐式耦合，缺少显式 schema 约束**，任一 agent 更名输出文件则链路静默中断。`mandatory-initial-read.md` 是全局前置读取文档，充当所有 agent 的「隐式约定层」。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「[状态驱动工作流](#state-driven-workflow)」——GSD MCP 服务器是项目状态的单一真实来源（HANDOFF.json 记录会话交接，`.planning/` 目录存储阶段状态），Claude Code 会话可以中断后恢复。
- **解决什么问题**：长时间工程任务（多天开发 sprint）在 Claude Code 会话重置后上下文丢失——`/gsd:resume-work` 能从 HANDOFF.json 恢复状态。
- **做了什么 trade-off**：引入 MCP 服务器增加了安装复杂度（需要 `node` + MCP 配置），换取了跨会话状态持久化和精确的工具访问控制（`mcp__gsd__*` 命名空间隔离）。
- **反映什么认知模型**：作者把多天工程开发看成「状态机」——每个阶段有明确的输入（当前状态）、操作（agent 执行）、输出（新状态），而非「对话串流」。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：MCP 状态机 + 三层编排架构（Buildomator 型）**

模式特征：
- 特征 1：**MCP 服务器作为状态仓库**——不把状态写进 Claude 上下文，而是写进 MCP 服务器的持久层，跨会话可恢复
- 特征 2：**三层调用链（用户 → skill → agent → MCP）**——每层职责清晰，skill 是门面，agent 是执行者，MCP 是持久化
- 特征 3：**references/ 共享知识层**——15 个共享文档被所有 agent/skill 按需 @引用，避免重复 + 保证一致性
- 特征 4：**HANDOFF.json 会话续接**——Session 中断后运行 `/gsd:resume-work` 恢复状态，MCP hook 在 SessionStart 自动触发
- 特征 5：**`@${CLAUDE_PLUGIN_ROOT}` 路径抽象**——所有跨件引用通过变量而非硬编码路径

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多天 / 多 sprint 工程开发 | ✅ 高度适用 | 跨会话状态恢复是核心价值 | 
| 单次短会话任务 | ❌ 过度工程 | HANDOFF.json + MCP 状态机的启动成本不值 |
| 需要 Git worktree 并行开发 | ✅ 适用 | `worktree-path-safety.md` 专门处理此场景 |
| 团队协作同一仓库 | ⚠️ 需定制 | 当前设计面向单人，多人共用 `.planning/` 会冲突 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 状态机 + 三层编排（本仓库） | jnuyens/gsd-plugin | 跨会话恢复、状态可观测 | 安装复杂、33 agents 无 model 声明 |
| 命令路由 + 专家 Agent（无状态） | c0x12c/ai-toolkit | 安装简单、命令覆盖面广 | 无跨会话状态、全无 allowed-tools |
| 纯 NL 工件（无运行时） | review-squad、simmer | 零依赖、安装简单 | 无持久化、每次会话重新开始 |

### 2.4 改进空间

1. **当前问题**：33 个 agents 全无 `model` 声明（最高优先级，-165 分）。**改进做法**：参照 `references/model-profiles.md`（已存在！）中的推荐，为每个 agent 加 `model:` 字段（haiku 给分类型 agent，sonnet 给执行型，opus 给调试型）。**预期收益**：agent 均分从 88.7 升至约 93.7；同时降低 50%+ 的 API 成本（haiku vs sonnet 约 20x 价差）。
2. **当前问题**：`gsd-project-researcher → gsd-research-synthesizer` 生产者/消费者靠文件名隐式耦合，无显式 schema。**改进做法**：在 `schema/` 下加 `research-output-v1.json`，在两个 agent 的 frontmatter 里各引用此 schema 作为输出/输入契约。**预期收益**：任一 agent 更新输出格式时，schema 版本碰撞会提前发现（而非运行时静默失效）。
3. **当前问题**：9 个「薄委托 skill」（audit-uat/cleanup/do 等）仅有 `@workflow` 引用无任何 process steps 或 examples（各 75/100）。**改进做法**：为每个 skill 加一行 `<output_format>` 描述（「产出文件路径 + 格式」）和一行使用示例。**预期收益**：9 个 skill 从 75 升至约 90，用户知道每个命令会产出什么。

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-28 当时得分 **90.3/100**（115 个文件，版本 v2.38.8）。

| 类别 | 当时均分 | 主要问题 |
|---|---|---|
| Agents（33 个） | 88.7/100 | 0/33 声明 model；advisor-researcher/assumptions-analyzer/eval-auditor 零 examples |
| Skills（82 个） | 约 90/100 | 9 个薄委托 skill 均 75 分；vague quantifier 系统性分布 |

### 3.2 当时值得借鉴的模式

1. **`references/` 共享知识层**：15 个 references 文件被所有 agent/skill 通过 `@${CLAUDE_PLUGIN_ROOT}/references/mandatory-initial-read.md` 等路径引用；根本原因：单一真实来源原则用于 NL 工件的「公共知识」部分。应借鉴：把多个 skill 重复的背景知识提取到 `references/` 目录，通过 `@` 引用注入。
2. **`references/model-profiles.md`（已存在）**：虽然 33 个 agents 都没有实际声明 model，但 `references/model-profiles.md` 已经写好了推荐 model tier 对照表——设计正确，执行缺失。应借鉴：设计时先写「应该用什么 model」的对照表，再批量实施。
3. **HANDOFF.json 会话续接**：`gsd-planner` body 里有 SessionStart hook 触发的 `gsd:resume-work` 指令，中断会话后无需重新描述上下文；根本原因：把状态外化到 MCP，而非依赖 LLM 记忆。应借鉴：任何多步长任务都考虑「状态外化」——存文件比依赖上下文更可靠。
4. **`schema/handoff-v1.json` 接口契约**：会话交接状态有 JSON Schema 约束；根本原因：机器可验证的约定比文字描述更健壮。应借鉴：agent 间的状态文件应有 schema，而非自由格式 JSON。
5. **安全意识的 agent 设计（gsd-debugger 的安全边界）**：`debug/SKILL.md` 包含明确的「禁止操作」列表（不读 `.env`、不修改生产配置）；根本原因：安全约束写进 skill 而非靠用户记忆。应借鉴：高权限操作的 skill/agent 要有明确的「禁止做什么」规则。

### 3.3 当时的缺陷

1. **0/33 agents 声明 model（-5/个，-165 分总计）**：这是整个插件最大的单一质量缺口。**为什么会失败**：无 model 声明时 Claude Code 用会话默认 model（通常是 Sonnet）运行所有 agent——意味着轻量分类任务（haiku 够用）也用 Sonnet 跑，API 成本约 20x 过高；同时也无法针对最复杂的调试任务（应该用 Opus）进行模型升级。自查：我的 agents 有没有 `model:` 字段，与任务复杂度是否匹配？
2. **生产者/消费者隐式耦合**：`gsd-project-researcher` 产出 6 个 section 文件，`gsd-research-synthesizer` 通过文件名消费，无显式 schema 约束。**为什么会失败**：任一 agent 版本更新改变了输出文件名，消费 agent 静默拿到空内容，产出的综合报告变成「基于空白的综合」——比失败更糟糕，是静默降级。自查：我的插件中 agent A 产出、agent B 消费的中间文件，有没有 schema 约束？
3. **模糊量词系统性分布（20+ agents 受影响）**：`appropriate`、`relevant`、`reasonable`、`intelligently` 等无法验证的词散布在 33 个 agents 里。**为什么会失败**：Claude 看到「用 appropriate 方式处理」时，需要自行判断「什么是 appropriate」——不同会话、不同 model 的判断可能完全不同，导致输出不稳定。自查：`grep -rn "appropriate\|relevant\|reasonable" agents/ | wc -l` 命中多少次？

### 3.4 当时的优化机会

1. 批量加 `model:` 到 33 个 agents（参照 `references/model-profiles.md`，一次性修复 -165 分）
2. 9 个薄委托 skills 各加一行 output format + 一行 example（75 → 90 分）
3. `gsd-advisor-researcher`、`gsd-assumptions-analyzer`、`gsd-eval-auditor` 各加 1 个 `<structured_returns>` 示例块（-15 → 0）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 0/33 agents 声明 model | `grep -rn "^model:" agents/ \| wc -l` | **持续**：0 个 model 声明（v4.4.0 仍是 0） | 3 个月、2 个大版本跨度（v2.38.8→v4.4.0）仍未修复最高优先级问题 |
| gsd-project-researcher → synthesizer 无 schema | 检查 schema/ 目录 | **部分改善**：`schema/handoff-v1.json` 存在，但 research 输出无 schema | 会话交接有 schema，但 research 链路仍无 |
| vague quantifiers 系统性分布 | `grep -rn "appropriate\|relevant" agents/ \| wc -l` | 持续（无法精确验证，需要人工抽查） | 系统性问题，修复成本高 |

### 4.2 架构演进

v2.38.8 → v4.4.0（跨度约 3 个月，版本号大幅跳跃）：
- **版本号从 v2 升至 v4**：说明有重大架构变更（可能是 API 变更或工作流重组）
- **`schema/handoff-v1.json` 新增**：会话交接有了正式 schema，与 agent-contracts.md 组合强化了 agent 间契约
- **`references/worktree-path-safety.md` 新增**：Git worktree 多分支并行开发场景获得专属安全指南
- **NL 工件数量稳定**：33 agents + 82 skills，规模未变——重心在质量而非数量扩张

这说明作者认识到了「交接/恢复」是核心痛点，专门引入了 schema 和安全指南，但对「model 声明」和「模糊量词」这两个 NLPM 明确指出的问题仍未行动。

### 4.3 新增的可学习模式

- **`schema/handoff-v1.json` JSON Schema 接口**：会话交接状态有了机器可验证的 schema，不再是自由格式 JSON。这是「约定由文字变为代码」的演进，值得在任何 multi-agent 插件中复制。
- **`references/worktree-path-safety.md`**：专门为 Git worktree 场景定义的安全规则（禁止在 worktree 外写文件等）——这种「场景专属安全 reference」的设计模式值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **状态外化而非依赖上下文**：我的 bureau 把知识存入 cabinet（文件系统），而非依赖 Claude 的记忆——与 GSD 的 HANDOFF.json 外化状态的思路一致。
2. **JSON Schema 约束（bureau 的 dossier 格式）**：bureau 的 dossier 有 YAML frontmatter schema 约束，与 gsd-plugin 的 `handoff-v1.json` 是同一种思路——agent 产出有结构化约定。
3. **skill body 没有「禁止操作」失配**：我的 bureau skills 没有要求做某事但工具声明里缺少相应工具的问题。

### 5.2 挑战 / 验证

这个案例验证了「schema 先写，实施后补」的困境：`references/model-profiles.md` 早就写好了「哪个 agent 该用什么 model」，但 33 个 agent 文件都没有执行——**这说明「有计划无实施」在 NL 工件中比代码中更难发现，因为工件不会报错**。对比代码：没写 `model` 字段不会触发编译错误，只是悄悄使用默认值，用户甚至感知不到。

教训：NL 工件的「配置正确」需要主动验证（NLPM score），而不能依赖运行时反馈。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 有没有 model 声明
grep -rn "^model:" /tmp/my-repos/MarkQWu-bureau/.claude/agents/ \
  /tmp/my-repos/MarkQWu-gstack/.claude/agents/ 2>/dev/null | head -20
# 如果 grep 无输出，说明我目前无 agent 文件
```
命中后怎么办：参考任务复杂度选择 model tier——分类/摘要用 haiku，执行/代码生成用 sonnet，复杂调试/架构审查用 opus。

```bash
# 检查我的 skills 有没有模糊量词
grep -rn -E "\b(appropriate|relevant|reasonable|intelligently|properly|key|important)\b" \
  /tmp/my-repos/MarkQWu-bureau/skills/*.md \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | grep -v CHANGELOG | wc -l
```
命中后怎么办：把「appropriate error handling」改为「处理 network failure、null input、4xx/5xx 三种情况」——用具体情况替代模糊形容词。

```bash
# 检查 agent 间状态文件是否有 JSON Schema
ls /tmp/my-repos/MarkQWu-bureau/schema/ /tmp/my-repos/MarkQWu-gstack/schema/ 2>/dev/null
```
命中后怎么办：若 agents 间通过文件交换状态，在 `schema/` 目录加对应的 JSON Schema 并在 agent frontmatter 中引用。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 引入 `references/` 共享知识层——把 bureau 的「信任层级说明」、「cabinet 格式约定」等重复出现在多个 skill 里的背景知识提取出来。
   - **为何可行**：bureau 的 `review/SKILL.md` 和 `compile/SKILL.md` 都重复描述了 trust tier（proposed/verified/canonical/stale）——这是典型的「应该进 references/ 的内容」。
   - **第一步**：在 `bureau/references/trust-tiers.md` 提取 trust tier 定义，改写 `compile/SKILL.md` 和 `review/SKILL.md` 为 `@bureau/references/trust-tiers.md` 引用；30 分钟。

2. **想法**：参考 GSD 的 HANDOFF.json 模式，为 bureau 的长时间 compile 任务加「任务续接」机制。
   - **为何可行**：bureau compile 在大型 workspace 可能需要多次会话（超出上下文窗口）——HANDOFF.json 风格的中间状态文件可以让下一次会话接续而非重做。
   - **第一步**：在 `bureau/skills/compile/SKILL.md` 最后一步加「写入 `.bureau/compile-state.json` 记录当前进度」；在 skill 开头加「检查 compile-state.json，若存在则从断点续接」；45 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 jnuyens/gsd-plugin 的核心目的**：通过 MCP 状态机持久化项目状态，支持多天/多 sprint 结构化工程工作流，会话中断后可恢复。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都把 AI session 状态写入持久化文件；都有多步骤 skill 编排 | bureau 是知识管理，gsd 是工程工作流；bureau 无 MCP 服务器 | 高 |
| MarkQWu/gstack | 中 | 都有复杂的多 skill 工程工作流 | gstack 无状态持久化，每次会话重新开始 | 中 |
| MarkQWu/graphify | 低 | 都处理「知识图谱」式数据 | graphify 是代码知识库，gsd 是工程计划状态 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agents 零 model 声明（-5/个） | `grep -rL "^model:" MarkQWu-bureau/.claude/agents/ 2>/dev/null` | bureau 无 agent 文件——暂无命中 | 暂无（提示：添加 agent 时必须加 model） |
| Vague quantifiers 系统性分布 | `grep -rn "appropriate\|relevant" MarkQWu-bureau/skills/*.md \| wc -l` | 命中约 12 次（需人工判断是否在 skill body 里） | 中 |
| 生产者/消费者无 schema 约束 | 检查 bureau 的 compile → review 状态传递 | bureau 的 compile 输出直接由 review 读取，无 JSON Schema | 中 |

**命中后的具体行动**：
- `MarkQWu-bureau/skills/compile/SKILL.md` 里的「产出文件格式」→ 在 `bureau/schema/` 加 `compiled-dossier-v1.json`，review skill 引用此 schema 作为输入契约；30 分钟。
- 逐行 review `bureau/skills/review/SKILL.md` 里的 `appropriate` 用法，替换为「处理以下情况：canonical 升级、stale 降级、contested 标注」；10 分钟。

### 8.3 别人的更优方案

1. **领域**：`references/` 共享知识层（跨 skill/agent 的 @引用机制）
   - **本案例做法**：15 个 `references/*.md` 文件，每个 skill/agent 按需 `@${CLAUDE_PLUGIN_ROOT}/references/mandatory-initial-read.md` 等引用，避免知识重复
   - **我的项目现状**：bureau 的 trust tier 定义在 `review/SKILL.md` 和 `compile/SKILL.md` 里各重复一遍；gstack 的架构说明分散在多个 SKILL.md 里
   - **如何借鉴**：创建 `bureau/references/trust-tiers.md` 和 `bureau/references/cabinet-format.md`，改写各 skill 为引用；参照 gsd 的 `@${CLAUDE_PLUGIN_ROOT}/references/` 路径约定。

2. **领域**：JSON Schema 接口契约（`schema/handoff-v1.json`）
   - **本案例做法**：会话交接状态有机器可验证的 JSON Schema，Claude Code 在产出时有结构约束
   - **我的项目现状**：bureau 的 dossier 格式有 YAML frontmatter schema（文字描述），但 compile 输出的中间状态无 schema
   - **如何借鉴**：在 `bureau/schema/` 加 `compile-state-v1.json`，compile skill 产出时验证格式，review skill 消费时 @引用 schema 作为输入规范。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Model 声明完整性
- **我的做法**：bureau 目前无 agent 文件，不存在「有 agent 但无 model 声明」的问题；若未来添加 agent，有本案例的警示
- **本案例做法弱在哪**：0/33 agents 有 model 声明，即使 `references/model-profiles.md` 已经写好推荐 tier，执行层仍然空白
- **意义**：这是一个「有设计但无实施」的典型反模式。我的项目未来添加 agents 时，`model:` 字段应该是 checklist 的第一项。

---

## 八、术语表

### <a name="mcp-server"></a>MCP 服务器
> 实现了 Model Context Protocol 的一个程序（通常是 Node.js 或 Python），提供一组工具（tools）给 AI 客户端调用。gsd-plugin 的 `mcp/server.cjs` 就是这样一个服务器——它提供 `mcp__gsd__read_project`、`mcp__gsd__write_phase` 等工具，让 Claude Code 能够读写 `.planning/` 目录里的项目状态文件。安装插件时，Claude Code 会自动启动这个服务器进程。

### <a name="state-driven-workflow"></a>状态驱动工作流
> 工作流的每一步执行前先读取「当前状态」（从文件或 MCP 服务器获取），执行后更新状态。与「对话驱动工作流」（每次会话重新开始，靠上下文记住进度）相对。GSD 的 HANDOFF.json 是状态文件——会话中断后，下一次运行 `/gsd:resume-work` 从文件读取断点，而非靠用户重新描述「之前做到哪步了」。对于多天任务，状态驱动比上下文驱动可靠得多。

### <a name="cross-reference"></a>`@${CLAUDE_PLUGIN_ROOT}` 引用
> Claude Code 插件系统提供的路径变量，在插件安装后自动解析为插件的实际目录路径。`@${CLAUDE_PLUGIN_ROOT}/references/mandatory-initial-read.md` 这样的写法让 NL 工件可以跨文件引用「兄弟文件」，而不需要硬编码绝对路径（那样在不同用户机器上会失效）。raw git clone 看到的是未展开的占位符——这不是 bug，只有通过 `claude plugin install` 安装后才能正常解析。
