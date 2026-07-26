# xiaolai/loc-guardian-for-claude — 学习案例

**仓库**：https://github.com/xiaolai/loc-guardian-for-claude
**Stars**：N/A（注册表未记录）| **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-26（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `template-design`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
一个单职责的 [Claude Code 插件](#claude-code-plugin)，专注于「按文件追踪 LOC（纯代码行数），在超限时调用高级模型分析并给出精准重构建议」。由 NLPM 设计者 xiaolai 本人编写，是其插件生态中的参考实现之一。仓库规模极小（8 个 [NL 工件](#NL-工件)），但结构设计极精。

关键事实：
- 仅 2 个命令（scan / init）+ 2 个代理（counter / optimizer）+ 2 个技能（loc / loc-optimization）
- `counter` 固定使用 Haiku（便宜快），`optimizer` 固定使用 Opus（深度推理）——模型分工明确
- 支持每项目独立配置：`.claude/loc-guardian.local.md`，frontmatter 控制限额，正文存用户自定义提取规则
- 数据通过结构化的 `loc-data` 代码块在 counter → scan → optimizer 之间传递，无自然语言解析依赖

### 1.2 架构剖析

**目录结构：**
```
.claude-plugin/plugin.json   ← 插件 manifest
commands/
  scan.md                    ← /loc-guardian:scan（编排器）
  init.md                    ← /loc-guardian:init（初始化配置）
agents/
  counter.md  [haiku]        ← 统计 LOC、检查超限
  optimizer.md [opus]        ← 分析超限文件、给出提取建议
skills/
  loc/SKILL.md               ← tokei 约定、指标定义、报告格式（counter 专用）
  loc-optimization/SKILL.md  ← 配置文件格式、通用优化模式（optimizer 专用）
```

**文件类型分布：** 2 命令 / 2 代理 / 2 技能 / 1 manifest / 1 CLAUDE.md = 8 工件

**编排关系（分层设计）：**
```
用户 → scan 命令
         ├─→ counter 代理 [Haiku] → 输出 verdict + loc-data 块
         └─→（仅当有超限文件时）optimizer 代理 [Opus] ← 接收全部 counter 输出
```

**跨件契约：**
- `scan.md` 检查 counter 输出中的 `VERDICT: N over limit` 来决定是否激活 optimizer
- counter 的 `loc-data` 块格式：`OVER|WARN <path> <pure_loc>`（机器可解析，无歧义）
- optimizer 读取 `.claude/loc-guardian.local.md` 获取限额和用户提取规则
- 两个技能严格「按需分配」：loc 只给 counter 加载，loc-optimization 只给 optimizer 加载

### 1.3 设计思路 / 方法论

**核心设计哲学：** 「协议化数据流」——组件间不靠自然语言传递意图，而靠结构化数据块（verdict 行 + loc-data 块）。这消除了 LLM 解析另一个 LLM 输出时的歧义风险。

**解决什么问题：** 文件越来越大却没人管，等到被审查时已经难以拆分。loc-guardian 让「每次 scan 时都看到数字」成为一个仪式感操作，从而把 LOC 管理变成日常习惯。

**Trade-off：**
- **控制粒度 vs. 灵活性**：optimizer 强制读取 `.claude/loc-guardian.local.md`，没有该文件时用通用模式兜底。用户必须先跑 `init` 才能获得最优建议——门槛换质量。
- **模型成本控制**：Haiku 做统计（廉价高频），Opus 做分析（贵但按需）。
- **单仓极简**：8 个文件。没有路由层、没有 meta-skill，功能单一所以架构单一。

**认知模型：** xiaolai 认为 NL 工件应该是「精确的合约」，而非「模糊的指导」。组件之间的数据流必须可以被程序解析（即使实际执行者是 LLM），避免级联的「自然语言电话游戏」。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「瘦命令 + 条件激活双代理流水线」**

关键特征：
- 命令层（scan.md）本身极薄——不含任何业务逻辑，只做分派和条件判断
- 代理 A 的输出中嵌入机器可读的控制信号（`VERDICT: N over limit`）供命令层解析
- 代理 B 仅在条件满足时激活——节省成本，避免冗余调用
- 技能按「最小访问原则」分配：每个代理只加载它需要的技能，不共享一个大技能包

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 单一职责的代码质量守护工具 | ✅ 高度适用 | 结构简单，命令薄，功能聚焦 |
| 需要「廉价探测 + 昂贵分析」的两阶段场景 | ✅ 适用 | Haiku 探测 + Opus 分析的模型分层刚好契合 |
| 多步骤跨领域工作流 | ❌ 不适用 | 缺路由层，扩展多个命令后会变混乱 |
| 需要用户持续交互决策的场景 | ❌ 不适用 | 整个流程是「一次调用到底」，没有暂停点 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 瘦命令 + 双代理流水线 | 本仓库 | 结构极简，成本分层，数据流清晰 | 扩展性有限，命令多了会乱 |
| Router + Channels 分层 | xiaolai/grill-for-claude | 可扩展多命令，命令间正交 | 需要 Router 维护，复杂度高一层 |
| 单 CLAUDE.md 平铺 | 大量初级插件 | 极简 | 长指令容易冲突，无法按需加载 |

### 2.4 改进空间

1. **当前问题**：`commands/scan.md` 缺少 `allowed-tools` 声明。**改进做法**：在 frontmatter 加 `allowed-tools: Agent`（scan 只调用子代理，一个工具就够）。**预期收益**：防止未来维护者意外扩展工具范围，明确意图契约。

2. **当前问题**：`agents/counter.md` 规则区有 "Keep the output concise."，`agents/optimizer.md` 有 "Note the most obvious extraction candidate."——`concise`、`obvious` 都是模糊量词，模型会各自解读。**改进做法**：`concise` → "每个 Table 不超过 10 行；不含解释性段落"；`obvious` → "选择行数最多、能单独移出且不改变调用方 API 的代码块"。**预期收益**：减少输出格式差异，提升两个代理协作的一致性。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **97/100**，是本 audit 队列中得分最高的仓库之一。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/optimizer.md | 91 | 只有一个 example，且缺少 `user:` 输入轮次 |
| commands/scan.md | 95 | 缺 `allowed-tools` 声明 |
| CLAUDE.md | 95 | 无 frontmatter（docs 类文件可接受，minor） |
| agents/counter.md | 98 | 模糊量词 "concise" |
| commands/init.md | 98 | 无问题 |
| .claude-plugin/plugin.json | 100 | 无问题 |
| skills/loc-optimization/SKILL.md | 100 | 无问题 |
| skills/loc/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **机器可读数据契约** → `loc-data` 块格式（`OVER|WARN <path> <pure_loc>`）让命令层可以程序化解析代理输出，消除 LLM 自然语言解析的不确定性。路径：`skills/loc/SKILL.md` § Raw File Data Block。如何借鉴：在命令分派到多个代理时，让上游代理的关键决策信号以固定格式输出，而非嵌入在叙述段落中。

2. **最小技能访问原则** → counter 和 optimizer 各自只加载自己需要的技能，没有一个「大而全」的共享技能包。路径：`agents/counter.md` frontmatter `skills: [loc-guardian:loc]`；`agents/optimizer.md` frontmatter `skills: [loc-guardian:loc-optimization]`。如何借鉴：每个代理的 `skills:` 列表应该只包含该代理任务所需的知识，防止无关上下文污染输出。

3. **模型按角色分层** → 探测代理用 Haiku，分析代理用 Opus，且由命令层（scan.md）通过 `model:` 覆盖强制执行。路径：`commands/scan.md` → `model: "haiku"`、`model: "opus"` 参数。如何借鉴：在两阶段工作流中，把「是否需要昂贵模型」的决策权收回命令层，而非让代理自行决定。

4. **条件激活模式** → `scan.md` 明确写出：`VERDICT: 0 over limit` 时不调用 optimizer。这不是默认行为——是明确编码进命令逻辑的节点。如何借鉴：在命令中显式写出「何时不调用」某个代理，比隐式依赖代理判断更可靠。

5. **per-project 配置文件** → `.claude/loc-guardian.local.md` 同时承载 YAML 配置（frontmatter）和用户自定义规则（正文），一个文件解决配置 + 约定两个需求。如何借鉴：对于需要「系统约定 + 用户自定义」的工具，可以用同一个 Markdown 文件的 frontmatter + 正文分别承载两类内容。

### 3.3 当时的缺陷

1. **`optimizer.md` 只有一个 example，且缺少 `user:` 输入轮次** → 根本原因：只有 assistant 输出、没有 user 触发，模型学不到「什么场景下该用这个代理」，导致不该激活时可能激活。自查：我的代理/技能 example 有没有 `user:` 和 `assistant:` 对称出现？

2. **`commands/scan.md` 缺 `allowed-tools`** → 根本原因：命令层声明的 `allowed-tools` 是「明确的契约」，缺失意味着任何工具都可能被调用，审查者和运行时都无从约束。自查：我的命令文件有没有在 frontmatter 写 `allowed-tools`？

3. **模糊量词（"lighter"、"obvious"、"concise"）** → 根本原因：这类词语让模型在每次执行时都做一个「猜测」，同一个代理在不同上下文中的输出边界会有差异。对于需要被另一个组件（scan.md）机器解析的输出，这是严重隐患。自查：我的代理/技能里有没有 "appropriate"、"concise"、"obvious"、"comprehensive" 等词？

### 3.4 当时的优化机会

1. `optimizer.md` 补第二个 example，展示 WARN-only 场景（无 OVER 文件时的轻量通道）
2. `counter.md` Rules 部分的 "concise" 换成 "every table ≤ 10 rows; no explanatory paragraphs"
3. `scan.md` frontmatter 加 `allowed-tools: Agent`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `optimizer.md` 只有一个 example | 数当前文件中 `<example>` 标签数量 | ✅ **已修复**：现在有 2 个 example，第二个专门展示 WARN-only 场景（含完整 `user:` + `assistant:` + `<commentary>`）| 这正是 Audit 建议的 fix；作者读了报告并跟进了 |
| `scan.md` 缺 `allowed-tools` | `grep "allowed-tools" commands/scan.md` | ❌ **仍存在**：命令文件无 `allowed-tools` 声明 | 功能上无害（scan 只调用 Agent），但契约未明确 |
| `counter.md` "concise" 模糊 | `grep "concise" agents/counter.md` | ❌ **仍存在**：Rules 最后一行 "Keep the output concise." | 轻微但未修复 |
| `optimizer.md` "obvious" 模糊 | `grep "obvious" agents/optimizer.md` | ❌ **仍存在**：Step 3 "Note the most obvious extraction candidate" | 同上，轻微未修复 |

### 4.2 架构演进

从审计快照到当前 HEAD，目录结构未变（仍是 2 命令 + 2 代理 + 2 技能 + 1 manifest）。但 `optimizer.md` 有内容更新：新增了第二个 example block，且第二个 example 的 `<commentary>` 明确说明了 WARN-only 时的行为分支。这说明作者的关注点在「代理行为的情境覆盖」，而非「架构扩展」。

### 4.3 新增的可学习模式

`optimizer.md` 第二个 example 的结构值得特别注意：

```
Context: counter's loc-data block contains only WARN lines (no OVER lines)...
assistant: "No files are over the limit, but some are in the warning zone..."
<commentary>
No OVER files — optimizer runs lighter WARN-only pass...
Per Step 3, each warning-zone file gets a single-line suggestion...
</commentary>
```

这是「example 承担分支文档」的设计模式——通过不同 example 展示代理在不同输入条件下的不同行为路径。比在正文写 `if/else` 描述更直观。

---

## 五、校准

### 5.1 我已经在做对的

1. `MarkQWu/gstack` 中的所有 SKILL.md 都有 `allowed-tools` 声明——比 loc-guardian 的命令文件做得更好
2. `MarkQWu/bureau` 的命令文件（如 `commands/init.md`）用 frontmatter 声明了 `description` 和 `argument-hint`——与 loc-guardian 命令格式一致
3. `MarkQWu/gstack` 的 SKILL.md 显式列出了禁止词（"robust"、"comprehensive"、"nuanced" 等）——这是主动防范模糊量词的更高级做法
4. bureau 使用了 per-project 配置文件（`bureau.md` 配合 `BUREAU.md` import）——类似 loc-guardian 的 `.claude/loc-guardian.local.md` 模式

### 5.2 挑战 / 验证

**挑战**：我之前认为「机器可读的数据契约」只需要在数据接口层面做（API、数据库），不需要在 NL 工件之间做。这个案例改变了我的看法：当两个代理之间有控制流依赖（A 的输出决定 B 是否激活）时，把控制信号结构化是必要的，而非可选的。

**验证**：bureau 里的 `lint.md` 命令也有类似问题——它调用 `lint` 技能检查条目，但条目的检查结果是自然语言输出，没有机器可读的「通过/失败」信号。如果以后要在 lint 基础上做自动修复，这个设计会成为阻碍。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 allowed-tools 声明
grep -rL "allowed-tools" $(find ~/.claude/commands/ /tmp/my-repos/ -name "*.md" 2>/dev/null) 2>/dev/null | head -10
```
命中后：在该命令文件的 frontmatter 中添加 `allowed-tools:` 并列出实际使用的工具（只有 Agent 就写 `Agent`）。

```bash
# 检查 NL 工件中的模糊量词
grep -rn -E '\b(concise|obvious|comprehensive|robust|appropriate|sufficient|adequate)\b' \
  /tmp/my-repos/MarkQWu-bureau/commands/ \
  /tmp/my-repos/MarkQWu-bureau/skills/ \
  --include="*.md" 2>/dev/null
```
命中后：把量词替换为可验证的标准，例如"concise → 每条输出 ≤ 3 行"。

```bash
# 检查我的命令/代理文件中是否有代理间数据传递依赖，但没有结构化控制信号
grep -rn "VERDICT\|loc-data\|\[PASS\]\|\[FAIL\]" \
  /tmp/my-repos/MarkQWu-bureau/commands/ 2>/dev/null | head -5
```
命中数为 0 时：审查 bureau 的多步命令，找出「A 的输出控制 B 是否执行」的隐性依赖，考虑补充结构化信号。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 `lint.md` 命令加结构化输出信号（PASS/FAIL + 条目路径列表），使未来可以自动修复
   - **为何可行**：bureau lint 已经有明确的检查规则，只缺最后一步把结果结构化
   - **第一步**：修改 `skills/lint/SKILL.md`，在规则末尾加「输出格式」节，定义 `LINT_RESULT: PASS|FAIL` 行和 `[[path: <drawer>/<entry>.md]]` 格式

2. **想法**：在 gstack 的某个代理里，借鉴「example 承担分支文档」模式，把主要行为分支通过独立 example 而非 if/else 文本表达
   - **为何可行**：gstack 的 pair-agent/SKILL.md 有多个分支（设计评审 vs. 代码评审），但目前用文本 if 描述，改成 example 更清晰
   - **第一步**：打开 `/tmp/my-repos/MarkQWu-gstack/pair-agent/SKILL.md`，找分支逻辑部分，改写为两个对称的 `<example>` 块

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（MarkQWu/gstack, MarkQWu/bureau, MarkQWu/graphify）

### 7.1 目的对齐度

- **本案例核心目的**：监控代码文件行数、在超限时提供具体重构建议，培养「文件不超限」的工程习惯

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 同样是 NL 工件生态，有技能 + 代理分层 | gstack 是多工具集合，loc-guardian 是单一守护工具 | 高（架构模式可借鉴） |
| MarkQWu/bureau | 低 | 同样有命令 + 技能 | bureau 专注知识管理，与 LOC 分析无交叉 | 中（数据契约模式可借鉴） |
| MarkQWu/graphify | 低 | 同样做代码分析 | graphify 是代码知识图谱，不是行数守护 | 低（目的差异大） |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令文件缺 `allowed-tools` | `grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md` | **bureau 命中**：`commands/` 下所有命令均无 `allowed-tools` 声明（10 个命令文件）| 中 |
| 代理文件有模糊量词 | `grep -rn -E '\b(concise\|obvious)\b' /tmp/my-repos/ --include="*.md"` | gstack 命中：`SKILL.md` 中有 "Do not use for routine coding or obvious changes"（出现在许多 skills）| 低 |
| 单个 example 无 user: 轮次 | `grep -B2 "<example>" /tmp/my-repos/MarkQWu-bureau/commands/*.md \| grep -v "user:"` | bureau commands 中的 example 数量极少（`capture.md` 无 example，多数命令无 example）| 高 |

**命中后的具体行动建议：**
- `MarkQWu/bureau` 的 `commands/init.md` → 在 frontmatter 加 `allowed-tools: Write, Bash, Read`（init 实际调用这三个工具）→ 15 分钟可完成
- `MarkQWu/bureau` 的 `commands/capture.md` → 加至少一个 `<example>` 展示典型用法（user: + assistant: + commentary）→ 20 分钟可完成
- `MarkQWu/gstack` 的多个 SKILL.md 中 "obvious changes" → 改为 "style / rename / typo fixes"（给出具体例子）→ 30 分钟扫一遍

### 7.3 别人的更优方案

1. **领域**：机器可读的代理间数据契约
   - **本案例做法**：`loc-data` 代码块格式（`skills/loc/SKILL.md` § Raw File Data Block），每行 `OVER|WARN <path> <pure_loc>`，命令层直接字符串匹配 `VERDICT: N over limit`
   - **我的项目现状**：`MarkQWu/bureau` 的 `commands/lint.md` 调用 lint 技能，结果是自然语言段落，没有结构化信号——未来若要自动修复，需要解析 LLM 输出的叙述文字
   - **如何借鉴**：在 bureau 的 `skills/lint/SKILL.md` 末尾添加 `## Output Protocol` 节，规定 lint 技能的输出必须包含 `LINT: PASS|WARN|FAIL` 行；在 `commands/lint.md` 中通过字符串匹配提取

2. **领域**：Haiku / Opus 按代价分层的模型策略
   - **本案例做法**：counter（廉价统计）用 Haiku，optimizer（深度推理）用 Opus；模型选择在命令层通过 `model:` 参数强制执行，不依赖代理自行决定
   - **我的项目现状**：bureau 的命令文件未声明模型，全部使用会话默认模型——expensive 操作和 cheap 操作用同一个模型
   - **如何借鉴**：在 `bureau:compile`（批量编译，开销大）命令中加 `model: claude-sonnet-5`；在 `bureau:lint`（文本检查，轻量）加 `model: claude-haiku-4-5`

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：主动防范模糊量词的系统化机制
- **我的做法**：`MarkQWu/gstack` 的每个 SKILL.md 都包含一个明确的禁止词列表（"delve"、"robust"、"comprehensive"、"nuanced" 等），从规范层面预防 AI 词汇滥用
- **本案例做法**：loc-guardian 没有类似的禁止词列表，代理正文仍然保留了 "concise"、"obvious" 等模糊量词
- **意义**：gstack 的「显式禁止词列表」是比「case-by-case 修复」更系统的防御机制；若被他人审到，这是亮点

---

## 八、术语表

### <a name="claude-code-plugin"></a>Claude Code 插件
> 一个以 `plugin.json` 为入口、包含命令（commands）、代理（agents）、技能（skills）等 Markdown 文件的目录，可以通过 `claude plugin install` 安装到 Claude Code CLI 中，为用户添加新的 `/` 命令能力。

### <a name="NL-工件"></a>NL 工件（Natural Language Artifact）
> 用自然语言（Markdown）写成、供 AI 模型解读执行的「程序文件」，包括命令文件、代理文件、技能文件、CLAUDE.md 等。与传统代码文件不同，它不直接被计算机运行，而是被 LLM 理解后驱动 LLM 的行为。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，用于声明元数据（如 `name`、`description`、`model`、`tools`、`skills` 等）。Claude Code 在加载 skill/agent/command 文件时先解析 frontmatter 才知道如何注册和调用。

### <a name="allowed-tools"></a>allowed-tools
> agent/command frontmatter 中的一个字段，声明该文件在执行时**允许调用的工具列表**（如 `Read`、`Bash`、`Agent`）。缺失时工具访问无约束；声明后成为运行时契约，也帮助审查者理解该组件的访问范围。

### <a name="模糊量词"></a>模糊量词
> 在 NL 工件中使用的、没有明确可验证标准的描述词，如 "concise"（简洁）、"appropriate"（适当）、"obvious"（明显）。NLPM 的质量评分规则（R15）把这类词标记为质量扣分项，因为它们让模型在每次执行时做出不同的猜测，导致输出不稳定。
