# hellowind777/hello2cc — 学习案例

**仓库**：https://github.com/hellowind777/hello2cc
**Stars**：651 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-12（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `examples-driven`, `fallback-chain`, `single-purpose`, `router-channels`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

hello2cc 是一个面向第三方模型用户的**可选工作习惯覆盖层**（overlay）插件。它的定位很精准：当你在 Claude Code 里使用非 Opus 模型（如 GPT-4o、Gemini、Qwen 等第三方模型）时，这些模型不了解 Claude Code 原生工具的使用习惯，会习惯性地走 LLM 的"思考-生成"路线，而不是用 Task 工具、原生 agent、原生 planner 等 Claude Code 专属工具。hello2cc 的核心 agent（native.md）就是告诉第三方模型："在 Claude Code 里，你应该这样工作。"

核心 agent `native.md` 全程用**中文**写成，68 行，包含正式的 YAML frontmatter（`name: native`，`model: inherit`），以及一套密集的操作规则体系。hook 架构是本插件另一大亮点：11 个 hook 事件类型，5 个独立脚本，覆盖从 UserPromptSubmit 到 Stop 的完整生命周期。plugin.json 注册了 10 个 userConfig 字段，全部有 `title` + `description`，是目前生态中 userConfig 覆盖度最高的插件之一。

### 1.2 架构剖析

```
hello2cc/
├── agents/
│   └── native.md          # 核心覆盖层 agent，68 行，全中文
├── hooks/
│   └── hooks.json         # 11 个事件类型，5 个脚本，评分 95/100
├── scripts/
│   ├── orchestrator.mjs
│   ├── subagent-context.mjs
│   ├── subagent-stop.mjs
│   ├── teammate-idle.mjs
│   ├── task-lifecycle.mjs
│   ├── ccstatusline-bridge.mjs    # 开发工具，非 hook 调用
│   ├── generate-release-notes.mjs # 开发工具
│   └── lib/
│       └── hook-io.mjs
├── output-styles/
├── docs/
├── tests/
├── package.json
├── CHANGELOG.md
├── README.md
└── README_CN.md
```

文件类型分布：1 个 agent，0 个 command，0 个 skill，1 个 hooks.json（11 事件类型），1 个 plugin.json（10 userConfig）。

编排关系：hooks.json 监听 UserPromptSubmit / Stop / SubagentStop 等事件 → 各 script 处理上下文注入、生命周期管理 → native.md 的指令在对话中持续生效，引导第三方模型使用原生工具。

跨件契约：`native.md` 声明了明确的优先级层级（用户消息 > Claude Code 宿主规则 > CLAUDE.md/AGENTS.md > hello2cc），防止与宿主环境的指令冲突。`hooks.json` 的所有脚本路径均可解析，无悬空引用。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：最小侵入性 + 模型无关适配。插件不替换 Claude Code 的任何原生功能，只是在第三方模型的行为层上叠加一层"工作习惯引导"，用户随时可以不激活这个 agent。
- **解决什么问题**：第三方模型在 Claude Code 里容易"退化"成普通聊天模式——不用 Task 工具、不走原生 agent 流、不遵循 Claude Code 的输出约定。hello2cc 通过一个常驻 agent 持续纠偏。
- **Trade-off 选择**：选择了"一个 agent 统管全局"而不是"多个 skill 分职责"，换来了极低的认知负担（用户只需要知道 native.md 这一个东西），但代价是这个 agent 文件需要承载所有的行为约束，且没有例子帮助用户验证正确行为。
- **认知模型**：作者把 Claude Code 视为一套有明确范式的工作环境，native.md 是这套范式的"说明书翻译"——把 Claude Code 原生用法翻译成第三方模型能理解的指令语言。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单职责覆盖层 + 钩子生命周期管理」**

模式特征清单：
- **覆盖层而非替换**：native.md 通过 `model: inherit` 继承会话模型，不强制锁定到特定模型，适配所有第三方模型。
- **显式优先级层级**：agent 明确声明自己在指令链中的优先级，防止多 CLAUDE.md 环境下的冲突。
- **钩子覆盖完整生命周期**：11 个事件类型涵盖 SubagentStop、Stop、UserPromptSubmit 等关键节点，脚本分工明确（orchestrator 管编排，task-lifecycle 管任务生命周期，teammate-idle 管空闲状态）。
- **单一 agent 文件原则**：整个插件只有一个面向用户的 agent，Zero 个 command，Zero 个 skill。极简主义。
- **10-entry userConfig 配置系统**：用户可以通过 userConfig 调整行为，而不需要修改 native.md 本体。

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 日常使用第三方模型在 Claude Code 里工作 | ✅ 高度适用 | 这正是插件的设计目标 |
| 只使用 Claude 原生模型（Claude Sonnet/Opus） | ❌ 不必要 | 原生模型已经了解 Claude Code 用法 |
| 需要跨多个工具调用的复杂工作流 | ✅ 适用 | hooks.json 覆盖了完整的 agent 生命周期 |
| 团队共享的工具规范插件 | ⚠️ 部分适用 | 规则是通用的，但 native.md 缺例子，新团队成员难以验证 |
| 严格安全审计环境 | ⚠️ 需谨慎 | ccstatusline-bridge 的 spawn(shell:true) 触发了高危审计标记（已确认为误报，但需人工确认） |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单覆盖层 agent（本仓库） | hello2cc | 极低认知负担；hooks 完整；plugin.json 配置丰富 | 无例子；无输出格式；vague-quantifier 密度高 |
| 多 skill 平铺 | baoyu-skills | 自包含，每个 skill 可单独使用 | 无整体工作习惯引导，无 hooks 覆盖 |
| 多 agent + skill 分层 | wshobson/agents | 任务分工清晰 | 复杂度高，安装门槛高 |
| 单覆盖层但无 hooks | 多数简单插件 | 极简 | 无法感知对话生命周期事件 |

### 2.4 改进空间

1. **当前问题**：native.md 没有任何示例块（`## 示例` 或 `## Example`），用户无法通过具体输入输出验证 agent 行为是否符合预期。**改进做法**：在 native.md 尾部添加 `## 示例` section，包含 2-3 个对比示例（"第三方模型的默认做法" vs "经 hello2cc 引导后的正确做法"）。**预期收益**：NLPM 评分 native.md 提升 +15 分；用户能直观理解插件效果。

2. **当前问题**：native.md 无输出格式声明（无 `## 输出格式` 或 `## Output Format` section），agent 产出的格式完全由第三方模型自由发挥。**改进做法**：添加输出格式约定，说明 agent 在不同场景下期望的输出结构（如：工具调用序列、状态汇报格式、完成确认格式）。**预期收益**：NLPM 评分 native.md 提升 +10 分；第三方模型输出更一致。

3. **当前问题**：generate-release-notes.mjs 用 fetch() 调用 api.github.com 并读取 GH_TOKEN，属于中等安全风险（外部 API 调用 + 凭证访问）。**改进做法**：在 README 里明确说明该脚本是开发工具，不在 hooks.json 中注册，添加 `/* dev-only */` 注释标记。**预期收益**：安全扫描不再将其误归为用户运行时风险。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 得分 **84/100**（加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/native.md | 61 | 零示例块（-15），无输出格式（-10），7 个[模糊量词](#模糊量词)（-14） |
| hooks/hooks.json | 95 | 所有引用有效，结构正确 |
| .claude-plugin/plugin.json | 95 | 10 个 userConfig 字段，全部有 title + description |

native.md 分数细目：
- 起始：100
- 零示例块：-15（示例缺失是 NL artifact 的高权重扣分项）
- 无输出格式 section：-10
- 模糊量词 × 7（"尽量" × 5，"genuinely" × 2）：-14
- **最终：61/100**

加权平均：(61 + 95 + 95) / 3 = **84/100**

### 3.2 当时值得借鉴的模式

1. **hooks.json 95分的钩子架构** → 根本原因：11 个 hook 事件类型覆盖了 Claude Code 工作流的完整生命周期（用户提交 → 工具调用 → subagent → stop），5 个脚本各司其职，无悬空引用，无过期事件名。借鉴方式：用 hello2cc 的 hooks.json 结构作为钩子设计的参考基准——不是所有项目都需要 11 个事件，但"引用全部可验证"和"脚本职责单一"是永远适用的原则。

2. **plugin.json 10-entry userConfig** → 根本原因：每个 userConfig 条目都有 `key`（机器读）、`title`（人类读标题）、`description`（人类读说明）三个字段，用户在 Claude Code 插件 UI 里看到的是清晰的配置项，而不是一堆神秘的环境变量名。借鉴方式：自己的插件 plugin.json 里，每个 userConfig 条目必须补全 title 和 description，不能只写 key。

3. **`model: inherit` 设计** → 根本原因：不在 agent frontmatter 里写死模型 ID，而是 `model: inherit`，让 agent 继承当前会话的模型。好处双重：一是插件在 Claude Sonnet 会话和 Claude Opus 会话里都能正常工作，二是不会因为模型 ID 过期而失效。借鉴方式：自己项目里的 agent 如果没有特定的模型层级要求，优先用 `model: inherit` 而不是 `model: sonnet`。

4. **显式优先级声明** → native.md 明确写出了指令优先级顺序：`用户消息 > Claude Code 宿主规则 > CLAUDE.md/AGENTS.md > hello2cc`。这解决了"多个 CLAUDE.md 同时存在时谁说了算"的歧义。借鉴方式：凡是作为"全局覆盖层"的 agent，都应该在 agent body 的第一段声明自己的优先级位置。

5. **极简 agent 表面** → 整个插件只有一个面向用户的 agent，没有 command，没有 skill。用户安装后不需要学习任何命令，native.md 自动在对话中生效。借鉴方式：当你的插件核心功能可以用一个持续激活的 agent 来表达时，不要因为"看起来简单"而强行拆分成多个 skill——单 agent 的用户体验往往更好。

### 3.3 当时的缺陷

1. **问题：native.md 零示例块** → 为什么失败：AI agent 文件没有示例，用户无法通过 "输入 X，期待输出 Y" 的方式验证 agent 是否工作。第三方模型读到没有示例的规则时，容易发生规则遵守不一致——因为规则本身是抽象的，没有具体的"正确做法长什么样"。根本原因是作者写 native.md 时站在了"规则制定者"的视角，没有站在"规则验证者"的视角。**自查**：我的 echo-sleuth 的 agent 文件有没有示例 section？

2. **问题：7 个模糊量词（"尽量" × 5，"genuinely" × 2）** → 为什么失败："尽量用原生工具" 这句话里的"尽量"没有边界——什么情况下可以不用？"genuinely unclear" 里的"genuinely"是谁来判断？第三方模型解读这类词时会按照自己的内部标准处理，而不同模型的内部标准是不同的，导致行为不可预测。根本原因是自然语言里的"礼貌性模糊"被带入了规范性指令。**自查**：我的 agent 文件里有多少个"尽量"、"尽可能"、"适当"？

3. **问题：无输出格式 section** → 为什么失败：第三方模型产出没有格式约束，输出结构因模型和上下文而异。当 hello2cc 需要配合 ccstatusline 显示状态时，输出格式不一致会导致 UI 显示异常。根本原因是输出格式声明被当成"可选项"而非必选项。

### 3.4 当时的优化机会

1. 为 native.md 添加 `## 示例` section，至少 2 个对比示例（第三方模型默认做法 vs 正确做法）
2. 添加 `## 输出格式` section，声明在不同 hook 触发场景下的期望输出结构
3. 将所有"尽量 X"改为"X，除非 [具体例外条件]"，消除量词歧义
4. 在 generate-release-notes.mjs 头部加 `/* dev-only: not registered in hooks.json */` 注释

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 零示例块（无 `## Example` 或 `## 示例`） | `grep -c "## Example\|## 示例" agents/native.md` | **仍存在** ❌ — 当前文件无任何示例 section | 审计后 2 个月仍未修复，说明此问题对作者而言优先级极低 |
| 无输出格式 section | `grep -c "## 输出格式\|## Output" agents/native.md` | **仍存在** ❌ — 无输出格式声明 | 同上，未触发作者修复动机 |
| 模糊量词（"尽量" × 5，"genuinely" × 2） | `grep -c "尽量\|genuinely" agents/native.md` | **仍存在** ❌ — grep 结果与审计时完全相同（7 处） | native.md 自审计以来内容基本无变化 |

三个主要缺陷全部仍然存在，native.md 的 NLPM 评分今天仍然会是 61/100。这说明：作者在 hooks.json 和 plugin.json 上投入了大量设计精力，但 native.md 的内容质量被系统性地忽视了。这是一种常见的"基础设施完善但内容欠打磨"的失衡模式。

### 4.2 架构演进

从 2026-04-06 审计到 2026-06-12 当前 HEAD，目录结构整体稳定，新增了 `ccstatusline-bridge.mjs`（开发工具，用于集成 ccstatusline 状态栏显示）和 `generate-release-notes.mjs`（发版辅助工具）。scripts/lib/ 目录也是新增的，包含 `hook-io.mjs`。这说明作者的演进方向是**强化脚本基础设施**，而不是完善 native.md 的内容质量。

版本和 CHANGELOG 显示项目仍在活跃维护，但迭代集中在 hook 系统的完善，不在 agent 内容的打磨。

### 4.3 新增的可学习模式

**ccstatusline 集成模式**：新增的 `ccstatusline-bridge.mjs` 展示了一个有价值的模式——把 Claude Code 的状态信息（当前 subagent 状态、task 生命周期）桥接到终端状态栏工具（ccstatusline）。这是"把 Claude Code 的可观测性提升到人类实时可见"的设计。注意：该脚本用了 `spawn(command, {shell:true})`，这是触发安全扫描 HIGH 标记的原因，但在 bridge 场景里是合理的设计——bridge 需要 shell 上下文来调用 ccstatusline CLI。

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth-for-claude 的 plugin.json 有结构化元数据**：有 name、version、description、author、license、keywords 字段，结构完整。这与 hello2cc plugin.json 的元数据完整性一致。

2. **echo-sleuth 的 SKILL.md 全部有 YAML frontmatter**：4 个 skill 文件都有正式的 frontmatter 声明，格式规范，这是 hello2cc native.md 也做到的基本要求，两个项目在这一点上都是合规的。

3. **agent 专业化分工**：echo-sleuth 把 recall、file-historian 等能力拆成独立 agent，与 hello2cc 的"单 agent 单职责"思路一致——不过方向相反（hello2cc 选择合并，echo-sleuth 选择拆分），两种选择在各自场景下都是合理的。

### 5.2 挑战 / 验证

本案例**挑战**了一个我此前的认知：我曾认为"规则写清楚了就够了"，但 native.md 的案例说明，规则清楚和行为可验证是两回事。7 处模糊量词的规则在审计后 2 个月没有被修复，部分原因可能是作者也不确定"尽量"的边界在哪里——如果边界不清楚，修复方向就难以确定。这说明，消除模糊量词不只是文风问题，而是迫使自己把规则边界想清楚的过程。

本案例**验证**了一个关于 hooks.json 的认知：精良的 hook 架构是可以独立于 agent 内容质量而存在的。hello2cc 的 hooks.json 95分、plugin.json 95分，但 native.md 只有 61分——三个组件可以有截然不同的质量水平。这提醒我：在 review 自己的插件时，每个组件要分别评估，不能因为 hooks.json 好就认为整体没问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否包含示例 section
grep -rL "## Example\|## 示例\|## 示例输出" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null
# 命中后：为缺少示例的 agent 添加至少 2 个对比示例（标准输入 → 期望输出）
```

```bash
# 检查我的 NL 文件中的模糊量词密度
grep -rn "尽量\|尽可能\|适当\|genuinely\|appropriately\|as needed" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
# 命中后：将每个"尽量 X"改为"X，除非 [具体例外条件]"
```

```bash
# 检查我的 plugin.json 是否有 userConfig 字段，以及每个条目是否有 title + description
cat /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.claude-plugin/plugin.json | \
  python3 -c "import sys,json; d=json.load(sys.stdin); uc=d.get('userConfig',[]); \
  [print('MISSING title/description:', e['key']) for e in uc if not e.get('title') or not e.get('description')]; \
  print(f'userConfig entries: {len(uc)}')"
# 命中后：为每个 userConfig 条目补全 title 和 description
```

```bash
# 检查我的 agent 文件是否声明了 model 字段
grep -L "^model:" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null
# 命中后：根据 agent 复杂度声明 model: inherit 或 model: sonnet/haiku
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 agents 添加 `model: inherit` 模式
   - **为何可行**：echo-sleuth 的 agents 目前无 model 字段，claude-code-action 会用默认模型。`model: inherit` 是零成本改动，避免锁定到特定模型 ID，和 hello2cc 的设计完全一致。
   - **第一步**：打开 echo-sleuth/agents/ 下的每个 .md，在 frontmatter 里添加 `model: inherit`，预计 5 分钟可完成所有文件。

2. **想法**：为 echo-sleuth 的 plugin.json 补充 userConfig 字段
   - **为何可行**：echo-sleuth 当前 plugin.json 无 userConfig 字段，但有几个可以暴露给用户配置的参数（搜索深度、输出格式偏好、时间窗口默认值）。hello2cc 的 10-entry userConfig 是生态最佳实践，差距明显。
   - **第一步**：确定 3-5 个用户有理由调整的参数，在 plugin.json 里添加对应的 userConfig 条目，每个条目写 `key`、`title`（简短中文或英文标题）、`description`（一句话说明）。预计 30 分钟。

3. **想法**：为 echo-sleuth 的主 agent 添加示例 section
   - **为何可行**：目前 echo-sleuth 的 agent 文件里没有示例，用户无法快速验证 agent 行为。hello2cc 的反面教材（审计后 2 个月 native.md 仍无示例，61分）证明了示例缺失的代价。
   - **第一步**：在 echo-sleuth 的主 agent 末尾添加 `## 示例` section，写 2 个具体示例（输入：用户问 X，输出：agent 做 Y），每个示例控制在 10 行内。预计 20 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：用户仓库（MarkQWu/drama-workshop-skills, MarkQWu/claude-for-legal, MarkQWu/echo-sleuth-for-claude）

### 8.1 目的对齐度

- **本案例 hellowind777/hello2cc 的核心目的**：为第三方模型在 Claude Code 里提供工作习惯覆盖，通过 agent + hooks 组合引导模型使用原生工具和原生工作流。

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同为 Claude Code 插件；同有 agents/ + plugin.json 结构；同用自然语言 agent 文件 | echo-sleuth 是功能型插件（挖掘对话历史），hello2cc 是行为覆盖型插件；echo-sleuth 无 hooks.json | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 插件 | 垂直领域完全不同；drama-workshop 无 hooks，无 agent | 低 |
| MarkQWu/claude-for-legal | 低 | 都有 skill/agent 层 | claude-for-legal 无 hooks，无 plugin.json userConfig | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 文件无示例 section | `grep -rL "## Example\|## 示例" agents/*.md` | echo-sleuth：所有 agent 文件均无 `## 示例` section（预期命中） | 高 |
| 模糊量词密度高 | `grep -c "尽量\|适当\|appropriately\|as needed" skills/*/SKILL.md` | echo-sleuth skill 文件中存在"relevant sessions"、"relevant git commits"等表述（低密度但有命中） | 低 |
| plugin.json 无 userConfig | `cat plugin.json \| python3 -c "import sys,json; d=json.load(sys.stdin); print(len(d.get('userConfig',[])))"` | echo-sleuth：userConfig 字段完全缺失（0 entries vs hello2cc 的 10 entries） | 高 |

**命中后的具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude/agents/recall.md`：在文件末尾添加 `## 示例` section（约 2 个对比示例），约 20 分钟
- `MarkQWu/echo-sleuth-for-claude/.claude-plugin/plugin.json`：添加 3-5 个 userConfig 条目，每个含 key/title/description，约 30 分钟
- `MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md`：将"relevant sessions"改为更精确的限定词（如"sessions matching the keyword and time range specified in the query"），约 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：hooks.json 完整生命周期覆盖
   - **本案例做法**：11 个事件类型，5 个独立脚本，`hooks/hooks.json` 评分 95/100。事件类型覆盖 UserPromptSubmit、Stop、SubagentStop、PostToolUse 等完整生命周期节点，每个节点对应职责明确的脚本。
   - **我的项目现状**：`MarkQWu/echo-sleuth-for-claude` 目前无 hooks.json，完全没有生命周期钩子。
   - **如何借鉴**：为 echo-sleuth 添加一个最小的 hooks.json，至少覆盖 Stop 事件（会话结束时汇总本次对话挖掘结果），参考 hello2cc 的钩子 + 脚本分离架构。

2. **领域**：plugin.json 的 userConfig 完整性
   - **本案例做法**：10 个 userConfig 条目，每个都有 key（机器读）+ title（UI 显示）+ description（用户说明）三个字段。举例：`{ "key": "HELLO2CC_CCSTATUSLINE_COMMAND", "title": "ccstatusline command", "description": "用于显示 Claude 状态的 ccstatusline 命令路径" }`。
   - **我的项目现状**：echo-sleuth plugin.json 有元数据（name/version/description/author/license/keywords）但完全没有 userConfig 字段，用户无法通过 Claude Code UI 配置任何行为。
   - **如何借鉴**：参考 hello2cc 的格式，为 echo-sleuth 至少添加 3 个 userConfig（搜索历史深度、默认时间窗口、输出语言偏好），每个补全 title + description。

3. **领域**：`model: inherit` 模型继承声明
   - **本案例做法**：native.md frontmatter 里声明 `model: inherit`，agent 自动继承会话模型，不锁定任何具体模型 ID。
   - **我的项目现状**：echo-sleuth 的 agent 文件无 model 字段声明，依赖 Claude Code 的默认行为。
   - **如何借鉴**：在所有无特定模型要求的 agent frontmatter 里统一添加 `model: inherit`，避免未来因模型 ID 变更而失效。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：NL artifact 的内容完整性（示例 + 输出格式）
- **我的做法**：`MarkQWu/echo-sleuth-for-claude` 的 SKILL.md 文件全部有正式 YAML frontmatter，且每个 skill 的 description 描述了明确的触发条件和输出预期，结构比 hello2cc 的 native.md 更规范。
- **本案例做法（弱在哪）**：native.md 的核心内容是 68 行密集的操作规则，没有示例（-15 分）、没有输出格式声明（-10 分）、有 7 个模糊量词（-14 分），总扣分 39 分，最终只有 61/100。echo-sleuth 的 SKILL.md 虽然规模更小，但在这几个维度上表现更均衡。
- **意义**：echo-sleuth 的"小而完整"比 hello2cc 的"规则丰富但缺示例"更符合 NLPM 规范。这是真实的优势，但也不代表可以放松——echo-sleuth 目前同样缺乏示例，只是扣分幅度比 hello2cc 小。

---

## 八、术语表

### <a name="hook"></a>hook（钩子）
> Claude Code 的「事件监听机制」。你可以在 hooks.json 里声明：「每次发生 UserPromptSubmit 事件时，运行这个脚本」。hello2cc 的 hooks.json 注册了 11 个事件类型（UserPromptSubmit、Stop、SubagentStop、PostToolUse 等），5 个脚本，在 Claude Code 的整个工作周期里持续感知状态变化。类比：建筑里的传感器网络——某个传感器触发了，对应的系统自动响应，不需要人手动操作。

### <a name="userConfig"></a>userConfig
> plugin.json 里的用户配置字段数组。每个条目有三个子字段：`key`（实际的环境变量名，机器使用）、`title`（在 Claude Code 插件 UI 里显示的简短标题，人类阅读）、`description`（详细说明这个配置项的作用）。hello2cc 有 10 个 userConfig 条目，是目前生态中配置化程度最高的插件之一。没有 userConfig 的插件（如 echo-sleuth 当前状态）要调整行为只能修改源文件，体验差。

### <a name="stop-hook"></a>stop hook
> hooks.json 里监听 `Stop` 事件的钩子。当 Claude Code 的当前对话轮次结束（Claude 停止输出）时触发。hello2cc 用 stop hook 来处理会话结束时的清理和状态汇报。subagent-stop.mjs 是专门处理 SubagentStop 事件的脚本——当一个 subagent 完成任务时触发，区别于整个会话的 Stop 事件。

### <a name="model-inherit"></a>model: inherit
> agent frontmatter 里的模型继承声明。写 `model: inherit` 的 agent 会使用当前 Claude Code 会话正在使用的模型，而不是锁定到一个具体的模型 ID（如 `claude-sonnet-4-5-20251022`）。优点：插件不会因模型 ID 过期而失效；缺点：agent 的表现取决于用户的会话模型，可能有质量波动。hello2cc 作为一个为第三方模型设计的覆盖层，`model: inherit` 是必选设计——否则 agent 会强制使用某个 Claude 模型，反而违背了"支持第三方模型"的初衷。

### <a name="覆盖层"></a>覆盖层（overlay）
> 一种插件架构模式：不替换宿主系统的任何功能，只在宿主行为之上叠加新的约束和引导。hello2cc 是"工作习惯覆盖层"——它不修改 Claude Code 的原生工具，不替换用户的 CLAUDE.md，只是在对话上下文里持续提醒第三方模型"在 Claude Code 里应该怎样工作"。覆盖层的标志性设计是**显式优先级声明**（hello2cc 明确写出自己在优先级链中的位置：低于用户消息和宿主规则），确保覆盖层不会意外覆盖用户的真实意图。

### <a name="模糊量词"></a>模糊量词
> 在规范性指令中使用的、含义边界不明确的副词或形容词，如"尽量"、"尽可能"、"适当"、"genuinely"、"appropriately"、"as needed"等。NLPM 审计中，模糊量词是高频扣分项（每个 -2 分，上限 -14 分），因为它们会让 AI 模型在不同上下文中对"满足规则"的判断产生分歧，导致行为不一致。hello2cc 的 native.md 里有 7 个模糊量词（"尽量" × 5，"genuinely" × 2），累计扣 14 分。修复方法：把每个"尽量 X"改为"X，除非 [具体的例外条件列举]"，把边界显式化。
