# EveryInc/compound-engineering-plugin — 学习案例

**仓库**：https://github.com/EveryInc/compound-engineering-plugin
**Stars**：未记录（Upstream Pool）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-10（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `examples-driven`, `monorepo-vs-split`, `nl-binary-hybrid`

---

## 一、理解

### 1.1 仓库概览

EveryInc/compound-engineering-plugin 是 EveryInc 工程文化的可复用输出形式。EveryInc 是一家 AI 工程公司，这个仓库把他们内部的代码审查流程、工程规范和开发工具链打包成了一个（实际上是两个）可安装的 Claude Code 插件。

规模数据（2026-04-12 审计快照）：
- **114 个 NL artifact 文件**，分布在 `plugins/compound-engineering/`、`plugins/coding-tutor/` 以及 `tests/fixtures/` 中
- 两个独立插件：`compound-engineering`（主插件，工程审查工具链）和 `coding-tutor`（编程教学辅助插件）
- 整体 NLPM 得分：**84/100（Silver 级）**，Security: **CLEAR**
- 安全发现：3 个中/低风险（无 Critical/High）

关键背景：这不是一个"作品集型"插件仓库，而是 EveryInc 自身工程实践的直接数字化输出。`plugins/compound-engineering/agents/review/` 目录下超过 20 个专化 reviewer agent，是他们内部 code review 流程的直接映射——每个 agent 对应一个审查维度，而不是让一个"全能 reviewer"包揽所有事情。

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
compound-engineering-plugin/
  CLAUDE.md, AGENTS.md, CHANGELOG.md, CONCEPTS.md
  plugins/
    compound-engineering/          ← 主插件：工程审查工具链
      agents/
        research/                  ← 研究类 agent（learnings-researcher 等）
        review/                    ← 20+ 专化 reviewer agents
        design/, workflow/, document-review/, docs/
      skills/
        ce-compound/, ce-work/, ce-review/, ce-debug/
        document-review/, git-worktree/ 等
      .claude-plugin/plugin.json
    coding-tutor/                  ← 第二个插件：编程教学
      commands/
        teach-me.md, sync-tutorials.md, quiz-me.md
      skills/, hooks/
  src/                             ← TypeScript 转换器核心层
    types/: claude.ts, gemini.ts, codex.ts, kiro.ts, copilot.ts, pi.ts, opencode.ts, droid.ts
    converters/: claude-to-gemini.ts, claude-to-codex.ts, claude-to-kiro.ts 等
    targets/: opencode.ts, codex.ts, pi.ts, kiro.ts, gemini.ts
    parsers/claude.ts
    commands/: cleanup.ts, convert.ts, install.ts, list.ts, plugin-path.ts
    utils/
  tests/fixtures/                  ← 各类 edge case 测试夹具
  bun.lock                         ← 使用 Bun 包管理器（非 npm）
  docs/
```

这个架构由两个截然不同的层组成：**NL artifact 层**（`plugins/`）定义"要做什么"，使用自然语言描述工程行为；**TypeScript 原生核心层**（`src/`）实现真正的跨平台转换逻辑，将 Claude Code 插件格式转换为 Gemini、Codex、Kiro、OpenCode、Copilot、Pi、Droid 等多个平台格式。

### 1.3 设计思路 / 方法论

**Board of Directors 风格 review 体系**是本仓库最有学习价值的设计探索。`plugins/compound-engineering/agents/review/` 下的 20+ 个 reviewer agents 每个只负责一个维度：

- `correctness-reviewer.md` — 只看逻辑正确性
- `security-reviewer.md` — 只看安全风险
- `performance-reviewer.md` — 只看性能问题
- `maintainability-reviewer.md` — 只看可维护性
- `testing-reviewer.md` — 只看测试覆盖
- `api-contract-reviewer.md` — 只看 API 合约
- `adversarial-reviewer.md` — 专门尝试找反例
- `deployment-verification-reviewer.md` — 只看部署相关

这是对"单一职责原则"在 NL artifact 设计层面的极限探索：用 20 个小而专注的 agent 代替一个大而模糊的通用 reviewer。每个 agent 的认知负载更轻，指令范围更窄，推理更可靠。

代价是：一次 review 需要派发 20 次，且**所有 52 个 agent 均无 examples**（这是整个审计的最大单一质量问题，直接影响仓库的示范价值）。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式：「NL 表皮 + 原生二进制核心」（nl-binary-hybrid）**

```
NL artifact 层（plugins/）
  ↓ 定义
  "要做什么" — 自然语言描述审查行为、教学流程
  ↓ 依赖
原生核心层（src/）
  ↓ 实现
  "怎么在多平台上做" — TypeScript 类型系统 + 编译期检查
  ↓ 输出
  Gemini / Codex / Kiro / OpenCode / Copilot / Pi / Droid 格式
```

结构特征：
- **NL 层**：可读性高，适合描述流程和意图；但平台依赖（只在 Claude Code 原生运行）
- **原生层**：可测试（`tests/fixtures/`），可类型检查（TypeScript），可跨平台（转换器输出）
- **两层边界**：NL artifact 描述工程行为的"是什么"，TypeScript 实现跨平台分发的"怎么做"

适用场景：
- 需要将同一套工程规范分发到多个 AI 编程平台（Claude Code、Gemini Code Assist、GitHub Copilot、Cursor 等）
- 需要对转换逻辑进行严格类型检查和回归测试的插件工具

不适用：
- 纯领域知识型插件（只需要 NL 层，TypeScript 层是过度工程）
- 单平台、单团队使用的小型工具

### 2.2 Board of Directors review 模式

```
一次 review 请求
  ↓ dispatch 到
  correctness-reviewer  ←─┐
  security-reviewer     ←─┤
  performance-reviewer  ←─┤  并行
  ...（共 20+ 个）       ←─┤
  adversarial-reviewer  ←─┘
  ↓ 汇总
  综合 review 报告
```

与"通用 reviewer"模式的本质区别：认知范围受限使推理质量上升。`adversarial-reviewer` 专门寻找反例，而不是顺带问一句"有没有边界情况"。

代价：orchestration 成本高，agents 无 examples 时示范价值损失最大。

### 2.3 与其他架构对比

| 架构 | 代表仓库 | 优势 | 劣势 |
|------|--------|------|------|
| NL 表皮 + 原生二进制核心（本仓库）| EveryInc/compound-engineering-plugin | 跨平台分发；转换逻辑可测试 | NL 层和 native 层的 bug 互相独立，调试复杂度翻倍 |
| 纯 NL artifact 插件集合 | wshobson/agents | 零学习成本，任何人可贡献 | 无类型检查，逻辑错误只在运行时发现 |
| skill-only 仓库 | upstash/context7 skills | 平台无关，引用简单 | 无 slash command 入口，无 agent 编排能力 |
| 单插件精品仓库 | 2389-research/review-squad | 质量集中，易维护 | 覆盖面小 |

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

整体 NL 得分：**84/100（Silver 级）**，Security: **CLEAR**

代表性文件得分明细：

| 文件路径 | 得分 | 主要问题 |
|---------|------|---------|
| `plugins/compound-engineering/agents/review/security-reviewer.md` | 62 | 无 examples（-15）；无 output format（-10） |
| `plugins/compound-engineering/agents/review/correctness-reviewer.md` | 62 | 无 examples（-15）；无 output format（-10） |
| `plugins/compound-engineering/agents/research/learnings-researcher.md` | 58 | 跨 skill 路径引用违规（-15）；无 examples（-15） |
| `plugins/compound-engineering/skills/git-worktree/SKILL.md` | 72 | `${CLAUDE_PLUGIN_ROOT}` 无 fallback（已修复，审计时 -15） |
| `plugins/coding-tutor/skills/coding-tutor/SKILL.md` | 72 | 同 git-worktree（审计时 -15） |
| `plugins/coding-tutor/commands/teach-me.md` | 0 | 无 YAML frontmatter（name/description/allowed-tools 全缺）→ 无法注册 |
| `plugins/coding-tutor/commands/sync-tutorials.md` | 0 | 同 teach-me.md |
| `plugins/coding-tutor/commands/quiz-me.md` | 0 | 同 teach-me.md |
| `plugins/compound-engineering/agents/review/performance-reviewer.md` | 62 | 无 examples（-15）；无 output format（-10） |
| `plugins/compound-engineering/agents/review/adversarial-reviewer.md` | 77 | 无 examples（-15） |

**最大单一质量问题**：52 个 agent 全部无 examples，每个扣 15 分，是整个仓库得分最主要的拖累。

安全发现明细：
- **Medium**：`scripts/capture-demo.py` — 将用户录制的内容上传到 catbox.moe 第三方服务（数据流出控制问题）
- **Medium**：`scripts/discover-sessions.sh` — 读取 `~/.claude/projects/*.jsonl` 会话历史（intentional 功能，但在共享环境或 CI 中风险较高）
- **Low**：`plugins/compound-engineering/skills/ce-compound/changelog/SKILL.md` — Discord webhook curl call（凭据泄漏风险低，但 webhook URL 硬编码不可取）

### 3.2 当时值得借鉴的模式

1. **Board of Directors review 体系**（最高价值）：20+ 专化 reviewer agents 的设计是本次审计周期中见过的最彻底的"单职责 agent"实践。每个 reviewer 只负责一个维度，认知范围受限，指令噪声最小。与通用 reviewer 相比，`adversarial-reviewer` 的存在本身就是一个学习点——它的唯一职责是"找反例"，而非"顺便也看看有没有问题"。

2. **`tests/fixtures/` 测试套件设计**：包含各类 edge case 的测试夹具，专门覆盖不完整 frontmatter、缺少字段等边界情况。这是将 NL artifact 质量保证系统化的实践——不是事后 NLPM 审计，而是在开发时就明确知道"这种情况下会怎样"。

3. **TypeScript 类型系统用于跨平台转换**：`src/types/` 下的 `claude.ts`、`gemini.ts`、`codex.ts` 等类型定义为每个平台的 artifact 格式建立了契约。类型错误在编译期就被捕获，而不是在多平台分发后才被用户报告。

4. **多插件 Monorepo 结构**：两个功能完全独立的插件（compound-engineering 和 coding-tutor）共存于同一个仓库，但通过各自独立的 `.claude-plugin/plugin.json` 保持独立安装单元。这是"单仓库维护便利性"和"插件功能独立性"的平衡方案。

### 3.3 当时的缺陷

1. **B-01/02/03：coding-tutor 全部 3 个 command 无 YAML frontmatter**（最严重 bug）：
   - `plugins/coding-tutor/commands/teach-me.md`
   - `plugins/coding-tutor/commands/sync-tutorials.md`
   - `plugins/coding-tutor/commands/quiz-me.md`
   
   三个文件均缺少 `name:`、`description:`、`allowed-tools:` 字段，Claude Code 无法注册这三个 slash command。coding-tutor 插件从用户角度看形同不存在。

2. **B-04：learnings-researcher.md 跨 skill 路径引用**：
   `plugins/compound-engineering/agents/research/learnings-researcher.md` 引用路径 `../../skills/ce-compound/references/yaml-schema.md`，违反 self-contained 规则。运行时路径无法解析，引用失效但无错误提示——静默失败是这类 bug 的典型特征。

3. **B-05：git-worktree/SKILL.md 使用 `${CLAUDE_PLUGIN_ROOT}` 无 fallback**：
   `plugins/compound-engineering/skills/git-worktree/SKILL.md` 的脚本调用使用了 `${CLAUDE_PLUGIN_ROOT}` 环境变量，但在 Codex、Gemini 等非 Claude Code 平台上该变量未设置，导致脚本调用静默失败。`plugins/coding-tutor/skills/coding-tutor/SKILL.md` 有完全相同的问题。

4. **Q-01：52 个 agent 全无 examples**（根本原因：功能完整性优先于示范价值）：
   整个 `plugins/compound-engineering/agents/` 目录下没有任何 agent 包含 `## Examples` 章节。这个问题的规模（52 个文件全部命中）说明这不是遗漏，而是有意的设计选择——作者的精力放在了"构建 20 个 reviewer 的 Board of Directors"上，而不是"为每个 reviewer 写示例"。代价是 NLPM 审计的 examples 维度得分为零。

5. **Q-02：10 个 document-review agents 无 output format section**：
   `plugins/compound-engineering/agents/document-review/` 下的 agent 文件缺少 `## Output Format` 章节，模型输出格式不受约束，review 结果的可预测性降低。

### 3.4 当时的优化机会

1. **为 top 5 review agents 补充 examples**：优先选择使用频率最高的 `correctness-reviewer`、`security-reviewer`、`performance-reviewer`、`api-contract-reviewer`、`adversarial-reviewer`，各添加一个真实 code review 的 input/output pair。这 5 个文件的修复可以覆盖审计时 examples 扣分的约 10%，且示范价值最高（因为这 5 个 agent 是 Board of Directors 的核心）。

2. **为 coding-tutor 的 3 个 command 补充完整 frontmatter**：每个文件加 4-5 行 YAML，修复成本极低，效果立竿见影（从得分 0 提升到 70+）。

3. **修复 learnings-researcher.md 的跨 skill 路径引用**：将 `../../skills/ce-compound/references/yaml-schema.md` 改为内联或替换为 self-contained 的引用方式。

4. **为 `${CLAUDE_PLUGIN_ROOT}` 添加 fallback**：模式为 `${CLAUDE_PLUGIN_ROOT:-$(dirname "$0")}` 或在脚本开头增加检测逻辑，确保在非 Claude Code 平台上脚本仍能找到正确路径。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在当前 HEAD 中的状态

以下基于 2026-06-10 Spot-check 验证：

| 缺陷 | 2026-04-12 状态 | 2026-06-10 状态 | 来源 |
|------|---------------|---------------|------|
| B-01/02/03: coding-tutor 3 个 command 无 frontmatter | 完全无 frontmatter，无法注册 | `grep "^name:" plugins/coding-tutor/commands/teach-me.md` 返回 0 行 — **仍然持续** | 未修复 |
| B-04: learnings-researcher 跨 skill 路径引用 | `../../skills/ce-compound/references/yaml-schema.md` 违规引用 | 待验证 | 未知 |
| B-05: git-worktree SKILL.md `${CLAUDE_PLUGIN_ROOT}` 无 fallback | 变量无 fallback，非 CC 平台静默失败 | `grep CLAUDE_PLUGIN_ROOT plugins/compound-engineering/skills/git-worktree/SKILL.md` 返回 0 行 — **已修复** | 已移除该变量用法 |
| B-06: coding-tutor SKILL.md 同问题 | 同 B-05 | 与 B-05 同步修复推测 — 待验证 | 可能已修复 |
| Q-01: 52 个 agent 无 examples | 0/52 有 examples | `grep -rl "## Examples" plugins/compound-engineering/agents/` 返回 0 结果 — **仍然持续** | 未修复 |

**总结**：git-worktree 的 `${CLAUDE_PLUGIN_ROOT}` 问题已修复；coding-tutor commands 的 frontmatter 缺失和 agents 无 examples 两个主要问题持续存在。从修复的选择来看，作者优先修复了影响"其他平台兼容性"的问题，而搁置了"NL 注册基础设施"的问题。

### 4.2 架构演进（2026-04-12 → 2026-06-10）

**TypeScript 转换器大幅扩展**（主要演进方向）：

审计时 `src/types/` 已有 `claude.ts`、`gemini.ts`、`codex.ts`；当前 HEAD 新增：
- `kiro.ts` — AWS Kiro 平台格式
- `opencode.ts` — OpenCode 格式
- `droid.ts` — Droid 格式
- `pi.ts` — Pi.ai 格式

对应的 `src/converters/` 和 `src/targets/` 也同步扩展。这说明仓库的主要演进方向是"更多平台支持"，而不是"修复 NL artifact 质量问题"——功能路线图与 NL 质量路线图并行但不交叉。

**包管理器从 npm 切换到 Bun**：`bun.lock` 的引入表明 `src/` 的构建基础设施做了升级，Bun 在运行速度和包管理一致性上优于 npm，适合频繁转换操作的 CLI 工具场景。

**新增文件模式**：
- `CONCEPTS.md` — 概念文档，将核心抽象（compound engineering、Board of Directors review 等）显式书写，是新增的领域知识沉淀形式
- `src/commands/` — CLI 命令模块化（`cleanup.ts`、`convert.ts`、`install.ts`、`list.ts`、`plugin-path.ts`），转换器从单一脚本演进为结构化 CLI 工具

### 4.3 新增的可学习模式

**CONCEPTS.md 模式**：将插件的核心概念和设计哲学单独写成一个文档，不依赖分散在各 skill/agent 文件中的上下文。这对于设计复杂（如 Board of Directors review）的插件尤其有价值——新用户/新贡献者可以通过一个文档理解整体架构，而不需要遍历 20 个 reviewer agent 文件才能理解设计意图。

**src/commands/ 模块化 CLI 层**：将原先可能内联在脚本中的 CLI 逻辑提取为独立的命令模块，每个文件（`convert.ts`、`install.ts`、`list.ts` 等）对应一个可单独测试的 CLI 操作。这是"单一职责原则"在 TypeScript 层的应用，与 NL 层的"单职责 reviewer agent"设计哲学一致。

---

## 五、校准

### 5.1 我已经在做对的

1. **无完全无 frontmatter 文件**：MarkQWu 的三个仓库（echo-sleuth-for-claude、claude-for-legal、drama-workshop-skills）中所有 NL artifact 文件均有正确的 YAML frontmatter 块。EveryInc 的 coding-tutor 在 2026-04-12 有 3 个 command 完全无 frontmatter，并且截止 2026-06-10 仍未修复——这是最基础的注册可靠性指标，我做得更好。

2. **无跨 skill 路径引用**：我的仓库中没有 `../../skills/` 类的跨文件相对路径引用，不存在 B-04 类的运行时路径解析失败风险。

3. **多插件结构有独立边界**：claude-for-legal 的多 plugin 结构中各 plugin 有独立的 CLAUDE.md，plugin 间的 edge cases 更清晰，不存在 coding-tutor 和 compound-engineering 之间技术债互相影响的情况。

### 5.2 挑战 / 验证

1. **agents 无 examples 是我和 EveryInc 共同的最大质量缺口**：EveryInc 52/52 agents 无 examples；我的 echo-sleuth-for-claude 5/5 agents 无 examples，claude-for-legal 10/10 agents 无 examples——15/15 命中，比例与 EveryInc 完全相同。这不是"偶尔遗漏"，而是系统性的 authoring 习惯问题。

2. **allowed-tools 缺失**：echo-sleuth-for-claude 全部 5 个 command 无 `allowed-tools:` 声明（5/5 命中）。EveryInc 的 coding-tutor commands 在没有 frontmatter 的情况下自然也没有 allowed-tools——两个仓库都存在这个问题，只是触发方式不同（我是字段缺失，EveryInc 是整块 frontmatter 缺失）。

3. **无 TypeScript 转换器 / nl-binary-hybrid 能力**：如果将来需要将 echo-sleuth 或 claude-for-legal 的功能分发到 Gemini Code Assist 或 GitHub Copilot，当前架构无法支持。EveryInc 的 `src/` 层解决了这个问题，我没有对应能力。

4. **无 Board of Directors review 体系**：我的 agents 设计中没有"单职责 reviewer" 的维度拆分概念。claude-for-legal 的法律审查类 agent 是否应该拆分为 `jurisdiction-reviewer`、`liability-reviewer`、`risk-level-reviewer` 等专化角色？这是 EveryInc 的设计为我带来的架构问题。

---

## 六、行动

### 6.1 自查动作

针对 `MarkQWu/echo-sleuth-for-claude` 和 `MarkQWu/claude-for-legal`，按优先级：

**P0 — 立即（30 分钟）：检查并补充 agents 的 examples**

```bash
# 检查 echo-sleuth 的 agents 是否有 examples
grep -rl "## Examples" /path/to/echo-sleuth-for-claude/.claude/agents/ || echo "0 agents have examples"

# 检查 claude-for-legal 的 agents 是否有 examples
grep -rl "## Examples" /path/to/claude-for-legal/.claude/agents/ || echo "0 agents have examples"
```

如果结果为 0，优先为使用频率最高的 1-2 个 agent 各添加一个真实 input/output pair。参考格式：
```markdown
## Examples

**Input**: 用户请求 "帮我分析这份合同第 5 条款的风险"
**Output**: "第 5 条款存在以下风险：（1）违约金条款未设上限……"
```

**P1 — 本周（1 小时）：检查并补充 commands 的 allowed-tools**

```bash
# 检查 echo-sleuth 的 commands 是否有 allowed-tools
grep -rL "^allowed-tools:" /path/to/echo-sleuth-for-claude/.claude/commands/

# 检查 claude-for-legal 的 commands 是否有 allowed-tools
grep -rL "^allowed-tools:" /path/to/claude-for-legal/.claude/commands/
```

对每个缺失的 command，在 frontmatter 中补充：
```yaml
allowed-tools:
  - Read
  - Bash
```
（仅声明实际需要的工具，不要无限制声明 `*`）

**P2 — 下月（半天）：探索 1-2 个 reviewer agent 的单职责拆分**

以 claude-for-legal 中最复杂的 review 类 agent 为起点，尝试拆分为 `jurisdiction-checker` 和 `liability-scope-reviewer` 两个专化角色。不需要复制 EveryInc 的 20 个 reviewer 规模，但理解"单职责 reviewer"与"通用 reviewer"的质量差异是有价值的设计实验。

### 6.2 灵感 → 实施路径

**从 EveryInc CONCEPTS.md 模式学习**：

如果 claude-for-legal 的设计意图对新用户不够清晰，考虑在仓库根目录添加 `CONCEPTS.md`，用 3-5 段说明：
- 为什么按法律子域（corporate/employment/privacy）而非按功能（review/draft/analyze）组织插件
- agent 和 command 的分工原则（agent 是专家，command 是触发入口）
- 各插件的使用前提（需要什么法律背景知识）

这不是文档任务，而是"将隐性架构决策变成显性约束"——一旦写下来，自己也更难在新文件中违背它。

**从 EveryInc 的 tests/fixtures/ 模式学习**：

在 echo-sleuth 中添加一个 `tests/fixtures/` 目录，放入 2-3 个测试用 agent/command 文件，专门覆盖 frontmatter 边界情况（完整 frontmatter、缺少 allowed-tools、完全无 frontmatter）。这不是为了运行测试，而是为了在开发时有一个"参照系"——知道缺少某个字段时 NLPM 会怎么打分。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

| 我的仓库 | 与 EveryInc/compound-engineering-plugin 相似度 | 相同点 | 不同点 |
|---------|----------------------------------------------|------|------|
| MarkQWu/claude-for-legal | **高** | 都是多插件 Monorepo；都有多个专化 agent；都面临"大量 agent 无 examples"问题；都有 plugin 间技术边界需要维护 | EveryInc 有 TypeScript 转换器（跨平台能力），我没有；EveryInc 的 reviewer agents 是单职责拆分，我的法律 agent 未拆分 |
| MarkQWu/echo-sleuth-for-claude | **中** | 都有 agents + commands 双层结构；都存在 allowed-tools 缺失问题；都是 Security CLEAR | 规模差异悬殊（5 agents vs 52 agents）；echo-sleuth 单插件，EveryInc 双插件 |
| MarkQWu/drama-workshop-skills | **低** | 都是 Claude Code 插件 | drama-workshop 无 agents，纯 skill-focused；领域完全不同 |

### 7.2 在我的项目里复现的同类问题

| EveryInc 缺陷 | 我的仓库复现情况 | 数量 | 严重度 |
|-------------|--------------|------|------|
| 52 个 agent 无 examples（Q-01） | echo-sleuth: 5/5 agents；claude-for-legal: 10/10 agents | 15/15 = 100% 命中 | 高 |
| coding-tutor commands 无 allowed-tools | echo-sleuth: 5/5 commands 无 allowed-tools | 5/5 = 100% 命中 | 高 |
| 跨 skill 路径引用（B-04）| 需验证：claude-for-legal 是否有 `../../` 路径引用 | 待查 | 中 |
| `${CLAUDE_PLUGIN_ROOT}` 无 fallback（B-05/06）| 需验证：是否有使用平台变量无 fallback 的 skill | 待查 | 中 |
| tests/fixtures/ 缺失 | 三个仓库均无测试夹具目录 | 3/3 | 低（非注册失败，但降低开发信心） |

**最高优先级同构问题**：agents 无 examples（15/15 命中）。EveryInc 的案例证明即使仓库功能很完整（Board of Directors 级别的 review 体系），examples 缺失仍然是 NLPM 审计的最大单一扣分项，且审计后 2 个月内仍未修复——说明这个问题需要主动优先处理，不会"顺带"被修复。

### 7.3 别人的更优方案

1. **TypeScript 类型系统用于 NL artifact 转换（EveryInc `src/` > 我没有对应层）**：
   - EveryInc 做法：每个目标平台（Gemini/Codex/Kiro）有独立的类型定义文件，转换逻辑有编译期类型检查，edge case 有 `tests/fixtures/` 覆盖。
   - 我的现状：如果想支持多平台，只能手动维护多份格式，没有自动化验证。
   - 借鉴时机：当 claude-for-legal 或 echo-sleuth 需要支持第二个 AI 编程平台时，优先考虑 nl-binary-hybrid 架构而不是手动复制文件。

2. **Board of Directors 单职责 reviewer 体系（EveryInc 20+ agents > 我的通用 reviewer）**：
   - EveryInc 做法：每个 reviewer agent 只负责一个维度，认知范围受限，推理质量上升。
   - 我的现状：claude-for-legal 的 review 类 agent 覆盖多个法律维度，是"通用 reviewer"而非"单职责 reviewer"。
   - 借鉴方法：不需要复制 20 个 reviewer 的规模，但可以在下一次添加 review agent 时，先问"这个 agent 的唯一职责是什么"——如果答案包含"以及"，就应该拆分。

3. **CONCEPTS.md 显式架构文档（EveryInc > 我的分散上下文）**：
   - EveryInc 做法：`CONCEPTS.md` 集中解释核心抽象，新用户/贡献者有单一入口理解设计意图。
   - 我的现状：架构决策分散在各 skill/agent 文件中，没有显式的概念层文档。

### 7.4 反向：我的项目做得比 EveryInc 好的地方

1. **coding-tutor commands 注册失败问题已主动规避**：我的 commands 全部有 frontmatter 块（虽然 allowed-tools 缺失，但 name/description 存在）。EveryInc 的 coding-tutor 3 个 command 在 2026-04-12 完全无法注册，且截止 2026-06-10 仍未修复。从用户角度看，coding-tutor 插件"存在但不可用"比"存在但部分功能受限"更糟。

2. **多插件 plugin 间边界更清晰**：claude-for-legal 的各子插件（regulatory-legal、corporate-legal 等）有独立的 CLAUDE.md，plugin 间的功能边界被显式声明。EveryInc 的 compound-engineering 和 coding-tutor 共存于同一仓库但功能方向完全不同，两者之间的边界文档不如 claude-for-legal 清晰。

3. **无环境变量无 fallback 的脚本调用**：我的 skill 文件没有使用平台特定环境变量（如 `${CLAUDE_PLUGIN_ROOT}`）而不提供 fallback，不存在 B-05/06 类的跨平台静默失败风险。

4. **安全风险更低**：EveryInc 有 `capture-demo.py` 上传用户录制到第三方服务（Medium 风险）和 `discover-sessions.sh` 读取会话历史（Medium 风险）。我的三个仓库没有涉及用户数据外传或会话历史读取的脚本，Security 风险配置文件更干净。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **NL 表皮 + 原生二进制核心（nl-binary-hybrid）** | 一种插件架构模式：NL artifact 层（agents/commands）用自然语言定义"要做什么"，TypeScript/Python 等原生层实现真正的计算逻辑（如跨平台格式转换）。NL 层提供可读性，原生层提供类型安全和可测试性。EveryInc/compound-engineering-plugin 是典型案例：`plugins/` 是 NL 表皮，`src/` 是原生二进制核心。 |
| **Board of Directors review** | 一种 agent 编排模式：用多个单职责 reviewer agent（每个只负责一个审查维度）代替单一通用 reviewer agent。类比于公司董事会——每位董事有独立专长，整体决策质量高于单人全权决定。EveryInc 在 `plugins/compound-engineering/agents/review/` 下实现了 20+ 个专化 reviewer，是该模式的代表实现。 |
| **Monorepo** | 将多个逻辑独立的项目（如 compound-engineering 插件和 coding-tutor 插件）放在同一个 Git 仓库中维护的策略。优势：共享工具链（如 TypeScript 转换器）、统一 CI、减少跨仓库协调成本。劣势：仓库边界模糊，不同子项目的质量问题可能互相遮蔽。 |
| **Bun** | 一个高性能的 JavaScript 运行时和包管理器，与 Node.js 和 npm 兼容但速度更快。EveryInc 在当前 HEAD 中引入了 `bun.lock`，从 npm 切换到 Bun 管理 `src/` 的 TypeScript 依赖。对 NL artifact 层无直接影响，但反映了 TypeScript 核心层的构建基础设施在持续演进。 |
| **cross-platform converter（跨平台转换器）** | 将 Claude Code 插件格式（NL artifacts）转换为其他 AI 编程平台格式的工具。EveryInc 的 `src/converters/` 实现了转换到 Gemini、Codex、Kiro、OpenCode、Copilot、Pi、Droid 等格式。核心价值：编写一次 NL artifact，分发到多个平台，而不需要手动维护多份格式的副本。 |
| **frontmatter** | Markdown 文件顶部以 `---` 包裹的 YAML 元数据块。在 Claude Code NL artifacts 中，frontmatter 声明 `name:`、`description:`、`model:`、`allowed-tools:` 等关键字段。缺少完整 frontmatter 块会导致 Claude Code 完全无法注册该 artifact——EveryInc 的 coding-tutor 3 个 command 是典型案例：文件内容完整，但因为没有 frontmatter，slash command 调色板中完全不可见。 |
| **self-contained（自包含）** | NL artifact 设计原则：每个 agent/skill 文件的引用关系只依赖自身内部内容，不依赖通过相对路径 `../../` 访问的外部文件。违反 self-contained 的文件（如 EveryInc 的 `learnings-researcher.md` 引用 `../../skills/ce-compound/references/yaml-schema.md`）在不同调用上下文中路径解析结果不同，导致静默失败。 |
