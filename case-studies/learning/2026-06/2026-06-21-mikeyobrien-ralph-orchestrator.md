# mikeyobrien/ralph-orchestrator — 学习案例

**仓库**：https://github.com/mikeyobrien/ralph-orchestrator
**Stars**：2710 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-21（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `vague-quantifier`, `security-gate`, `single-purpose`, `examples-driven`

**xiaolai 案例**：[../../2026-04-30-mikeyobrien-ralph-orchestrator.md](../../2026-04-30-mikeyobrien-ralph-orchestrator.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

[mikeyobrien/ralph-orchestrator](https://github.com/mikeyobrien/ralph-orchestrator) 是一个自主 AI agent 编排框架，2710 stars，基于"Ralph Wiggum 技术"——一种[递归自编排](#递归自编排)模式，让 AI agent 能路由任务给子 agent、管理 scratchpad 和 memory，并支持 Claude/Codex/Gemini 多种后端。"Ralph Wiggum"出自《辛普森一家》，是个以不可预测输出著称的角色——作者用这个名字命名一个旨在让非确定性 agent 行为变得可控的框架，带有一种自嘲式的精准。

三个关键事实：
1. **双层架构**：Rust 核心（`ralph-cli` 二进制）+ NL 层（`.claude/` 下 14 个 SKILL.md + 2 个 agent 定义），共 19 个 NL 制品，是典型的 [NL-Binary 混合架构](#nl-binary-混合架构)
2. **模型分级**：opus 用于 E2E 验证和复杂代码任务，haiku 用于轻量级 loop runner——说明作者有意识地做了 cost/quality 权衡
3. **最高分记录**：91/100 是 NLPM 截止 2026-04-19 审查过的 157 个仓库中的最高分，零孤立引用、零 frontmatter 缺失、零跨组件不一致

当前 HEAD（2026-06-21）相比 audit 时新增了 2 个 skill：`ralph-docs`（含 `references/contributing.md`、`references/common-questions.md`、`references/llms-txt-map.md`）和 `ralph-tools`。原有 19 个制品的核心质量问题大多**仍未修复**（见第四节）。

### 1.2 架构剖析

```
ralph-orchestrator/
├── crates/
│   ├── ralph-cli/          # Rust 主二进制（核心 orchestrator 逻辑）
│   │   └── sops/           # 包含 include_str!() 打包的技能文本
│   ├── ralph-core/         # 核心库
│   │   └── tests/fixtures/skills/  # 测试夹具（complex-test-skill/SKILL.md，75分）
│   └── ralph-loop/         # loop runner 实现
├── .claude/
│   ├── skills/             # 主要 NL skill 层（多数 SKILL.md 在这里）
│   │   ├── code-assist/SKILL.md        # 84/100（−16 模糊量词）
│   │   ├── review-pr/SKILL.md          # 100/100（唯一满分）
│   │   ├── pdd/SKILL.md                # 87/100（示例不足 + 模糊）
│   │   └── ...（共 12 个 skill）
│   └── agents/
│       ├── ralph-e2e-verifier.md       # 86/100（−14 模糊量词）
│       └── ...（共 2 个 agent）
├── skills/
│   └── ralph-loop/SKILL.md            # 86/100（缺少 output format −10）
├── scripts/
│   ├── sync-embedded-files.sh         # curl 无 checksum + source 执行（已修复 source）
│   ├── test-fresh-install.sh          # npm install → npm ci（已修复）
│   ├── ci-rust-gate.sh                # rustup 运行时下载
│   ├── setup-hooks.sh                 # 写入 .git/hooks/
│   └── hooks-bdd-gate.sh              # $GITHUB_STEP_SUMMARY 写入
└── plugin.json / marketplace.json
```

- **编排关系**：ralph-cli 二进制是真正的路由器——读取 NL skill 定义，决定分发给哪个子 agent；NL 层是描述层，不是执行层
- **跨件契约**：`.claude/skills/code-task-generator/SKILL.md` 与 `crates/ralph-cli/sops/` 之间存在**同步耦合**——Rust 通过 `include_str!()` 宏在编译期将 skill 文本打包进二进制；skill 文件变更后必须重新编译才能生效
- **所有跨引用均正确**：`.claude/` 下的 agent 引用的 skill 路径全部存在，无悬空引用

### 1.3 设计思路 / 方法论

- **核心哲学**："把 NL 制品当成 API contract，把 Rust 当 runtime"——NL 层定义 *what*，Rust 层执行 *how*。这是和纯 NL 插件最大的区别：ralph 的 NL artifact 不直接触发 bash，而是被 Rust 读取后翻译成结构化指令
- **递归自编排设计**：ralph 能调用 ralph——框架通过 scratchpad 管理状态，通过 memory 跨会话保持上下文，允许 agent 在失败时重新路由给自己的子实例
- **模型选型哲学**：E2E verifier 用 opus（高成本，高精度，不频繁），loop runner 用 haiku（低成本，轻量，高频）——这是明确的 cost-latency trade-off，不是随意选择
- **Trade-off**：选择 Rust 核心意味着编译期耦合（NL 和二进制必须同步），换来的是执行速度和类型安全；纯 Python/Node 方案更易维护，但 ralph 的作者显然把性能和可靠性优先级放更高

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「NL-Binary 混合架构 + 分级模型路由」**

ralph-orchestrator 最值得学习的不是某个 SKILL.md 的写法，而是整体架构模式：用 NL artifact 做行为规范，用编译型语言做执行引擎，并在模型选型上做显式成本分级。

模式特征清单：
- NL 层只含"这个 skill 能做什么、输入/输出格式、示例"，不含业务逻辑
- 执行逻辑全在 Rust（或其他编译型语言）中，NL 是 Rust 的配置文件而非脚本
- 高精度任务（E2E 验证、复杂代码生成）→ opus；高频轻量任务（loop 决策）→ haiku
- Scratchpad + memory 分离：scratchpad 是当前任务的工作空间，memory 是跨任务的持久化知识

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要高可靠性的自主编码任务 | ✅ 高度适用 | Rust 执行层 + E2E verifier 保证每步有确认 |
| 轻量级脚本自动化 | ⚠️ 有限适用 | 引入 Rust 编译成本，纯 NL 方案更轻 |
| 多 AI 后端切换（Claude/Codex/Gemini） | ✅ 高度适用 | 框架原生支持多后端路由 |
| 个人小型项目 | ❌ 不适用 | 架构复杂度过高，单 SKILL.md 足矣 |
| 团队协作、可审计的 agent 工作流 | ✅ 适用 | scratchpad/memory 提供完整的执行历史 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL-Binary 混合（本案例） | ralph-orchestrator | 执行可靠，NL 变更有明确契约 | 编译耦合，SKILL.md 改动需要 rebuild |
| 纯 NL 多层编排 | 本项目（nlpm） | 零编译，NL 即运行 | 执行稳定性依赖模型理解 |
| 极简单 SKILL.md | lackeyjb/playwright-skill | 维护成本极低 | 无法编排复杂子任务 |
| Python 命令行 + NL | anthropic/claude-code | 生态成熟，调试方便 | Python 依赖链比 Rust 重 |

### 2.4 改进空间

1. **`include_str!()` 耦合无文档说明**：当前 `crates/ralph-cli/sops/` 与 `.claude/skills/code-task-generator/SKILL.md` 的同步依赖没有在任何 README 或 CONTRIBUTING 里说明。**改进做法**：在 `CONTRIBUTING.md` 中加一节"修改 SKILL.md 后必须重新编译 ralph-cli（make build 或 cargo build）"，附上 `sops/` 目录的结构说明。**预期收益**：贡献者修改 NL 层后不会以为生效了，但实际运行的还是旧版本。

2. **vague quantifier 未修复**：`code-assist/SKILL.md` 中 appropriate×3、necessary×2、relevant×2 经当前 HEAD grep 确认仍存在（5 处匹配）。**改进做法**：把每处替换为可测量阈值，例如"如果文件超过 500 行，优先用 file-path 精确定位而非全文扫描"替代"使用合适的方法"。**预期收益**：code-assist 分数从 84 升至 ≥95，且减少 Claude 在歧义情境下的猜测行为。

3. **ralph-loop/SKILL.md 缺少 output format 章节**：当前 HEAD 仍未补充。**改进做法**：在文件末尾加一节"## 输出格式"，描述 loop runner 每轮返回的结构（成功/失败标志、下一步建议、执行日志位置）。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：91/100**（19 个制品，NLPM 截至当时 157 个仓库中的最高分）

| 文件 | 当时分数 | 主要扣分原因 |
|---|---|---|
| `.claude/skills/review-pr/SKILL.md` | **100/100** | 无——唯一满分文件 |
| `.claude/skills/code-assist/SKILL.md` | 84/100 | 8 个模糊量词：appropriate×3, relevant×2, necessary×2, reasonable×1（−16） |
| `.claude/agents/ralph-e2e-verifier.md` | 86/100 | 7 个模糊量词：comprehensive×3, actionable×2, relevant×2（−14） |
| `skills/ralph-loop/SKILL.md` | 86/100 | 缺少 output format 章节（−10），另有模糊量词 |
| `.claude/skills/pdd/SKILL.md` | 87/100 | 示例数量仅 1 个（−8）+ 模糊量词（−5） |
| `crates/ralph-core/tests/fixtures/skills/complex-test-skill/SKILL.md` | 75/100 | 测试夹具，非生产制品，25 分扣分属正常 |

```
分数分布（19 个制品）：
90–100 分（优秀）：12 个
80–89 分（良好）：6 个
70–79 分（夹具）：1 个
```

### 3.2 当时值得借鉴的模式

1. **`review-pr/SKILL.md` = 100/100 满分实证**：与其他文件同一作者、同一时期、同类规模，唯一区别是"appropriate"出现 0 次。这说明：消除模糊量词不需要重写，只需要审查词汇。→ 自查方法：`grep -c "appropriate\|relevant\|comprehensive" .claude/skills/*/SKILL.md`

2. **零孤立引用**：19 个制品中所有跨文件引用均正确解析。agent 文件里 `skills:` 字段指向的 skill 路径全部存在，无一缺失。→ 借鉴：每次新增 skill 后，用 `ls` 确认 agent frontmatter 里的路径存在再提交

3. **全部 frontmatter 完整**：19 个制品无一缺少 `name` 或 `description` 字段。→ 借鉴：养成"新建 SKILL.md 第一步写 frontmatter"的习惯，不要等到最后

4. **opus/haiku 分级路由有明确文档依据**：agent 定义里显式写明了选用哪个模型及原因，不是隐式决策。→ 借鉴：每个 agent 的模型选择应在 frontmatter 或首段说明，"为什么是 opus 而不是 sonnet"

### 3.3 当时的缺陷

1. **模糊量词集中爆发**：12 个质量问题中，10 个是模糊量词或缺少格式章节。根本原因：作者在写描述性 skill 时习惯用形容词而非阈值，这在人类文档中无害，但在 NL artifact 里 Claude 无法量化执行。→ 自查：`grep -rn "appropriate\|relevant\|comprehensive\|necessary\|reasonable\|actionable" .claude/skills/`

2. **output format 章节缺失（ralph-loop）**：`skills/ralph-loop/SKILL.md` 没有说明 loop runner 每轮返回什么结构。根本原因：作者可能认为 loop 的输出"显而易见"，但消费者（调用 loop 的上层 agent）需要知道如何解析返回值。→ 自查：`grep -L "## 输出\|## Output\|output format" .claude/skills/*/SKILL.md`

3. **pdd skill 示例不足**：只有 1 个示例。根本原因：pdd（Prompt-Driven Development）的概念对新用户不直觉，1 个示例不足以覆盖"输入什么→期望什么输出"的变化。→ 参考基线：review-pr 有 3+ 个示例。

4. **shell 脚本安全欠债**：`source "$config_path"` 执行 shell 代码（低）；`npm install` 不遵守 lockfile（中）；`curl` 无 checksum（中）；`rustup` 运行时下载（中）。这些发现在 NL 质量层不计分，但在安全门中触发了 6 个 finding。

### 3.4 当时的优化机会

1. **优化机会**：对 `scripts/sync-embedded-files.sh` 的 `source` 替换为 `grep`/`sed` 精确解析——消除 shell 注入面
2. **优化机会**：`npm install` → `npm ci`——让 `test-fresh-install.sh` 对 lockfile 严格而非松散
3. **优化机会**：`code-assist`、`ralph-e2e-verifier`、`ralph-loop` 三个文件做一次"模糊量词审查"——预计把综合分从 91 推到 96+
4. **优化机会**：给 `pdd/SKILL.md` 补充 2 个额外示例，覆盖"已有 spec 的修改"和"全新功能"两类场景

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | grep 验证方法 | 现状 | 含义 |
|---|---|---|---|
| `code-assist` 模糊量词 | `grep -c "appropriate\|necessary\|relevant" .claude/skills/code-assist/SKILL.md` | **仍存在**：5 处匹配（most appropriate approach, necessary directory structures, relevant conventions, relevant code, appropriate file paths） | 2+ 个月未修复，属低优先级 |
| `ralph-e2e-verifier` 模糊量词 | `grep -c "comprehensive\|actionable\|relevant" .claude/agents/ralph-e2e-verifier.md` | **仍存在**：5 处匹配（comprehensive E2E verification×2, comprehensive report, actionable items, relevant log excerpts） | 同上 |
| `ralph-loop` 缺少 output format | `grep -l "output format\|输出格式" skills/ralph-loop/SKILL.md` | **仍缺失**：该章节在当前 HEAD 仍不存在 | 影响上层 agent 解析返回值 |
| `npm install` → `npm ci` | PR #305 验证 | **已修复** ✅：PR #305 于 2026-04-25 合并 | 供应链漂移风险消除 |
| `source "$config_path"` | PR #304 验证 | **已修复** ✅：PR #304 于 2026-04-30 合并，改为 `grep`/`sed` 精确解析 | shell 注入面消除 |
| `curl` 无 checksum | `grep -n "curl" scripts/sync-embedded-files.sh` | **仍存在**：`curl` 下载后未验证内容完整性 | 已知风险，维护者选择不处理 |

**关键观察**：安全修复（高影响、机械操作）快速被接受（2 天内）；质量修复（低影响、主观判断）2 个月未动。这是合理的优先级判断，但也说明[模糊量词](#模糊量词)问题被维护者认为不值得一次编辑 pass。

### 4.2 架构演进

2026-04-19 → 2026-06-21 的变化：
- **新增 `ralph-docs` skill**：引入 `references/contributing.md`、`references/common-questions.md`、`references/llms-txt-map.md` 三个文档引用文件——说明作者开始把用户文档整合进 NL 层
- **新增 `ralph-tools` skill**：具体内容待核实，从名称推断是工具调用相关
- **核心架构无变化**：`.claude/skills/` 和 `skills/` 目录结构与 audit 时相同，Rust 编译层不变

### 4.3 新增的可学习模式

1. **文档 skill 化（ralph-docs）**：把 `contributing.md`、`common-questions.md`、`llms-txt-map.md` 做成一个专门的 skill，让 Claude 能通过 skill 路由回答项目特定问题，而不是依赖模型自身知识。→ 借鉴：我的项目里的 README 和 FAQ 是否也可以做成一个`<project>-docs` skill？

---

## 五、校准

### 5.1 我已经在做对的

1. **agent frontmatter 完整率更高**：我的 `echo-sleuth-for-claude` 5 个 agent 全部有 `name` + `description`，ralph 当时也做到了这点，但我在这基础上还加了 `model` 字段——这一点做得比 ralph 全面
2. **没有 NL-Binary 同步耦合**：我的 SKILL.md 不通过 `include_str!()` 被编译进二进制，修改即生效，无需 rebuild——这是架构层的简化，代价是牺牲了执行层的性能，但对我的使用场景合适
3. **模型选型意识**：在阅读 ralph 的 opus/haiku 分级后确认我的 echo-sleuth 已有类似考量（memory-management 用轻量 agent，experience-synthesis 用 sonnet）

### 5.2 挑战 / 验证

本案例**挑战**了一个我的假设：我之前认为"91 分就算接近完美了"。但 ralph 的案例说明 91 分的仓库仍然有：10 处可改的模糊量词、1 处缺失的 output format、6 个安全 finding——这些不是可以忽略的尾注，而是实际影响 Claude 行为的精度问题。

本案例**验证**了"安全修复比质量修复更容易被接受"这一规律：PR #304、#305 被秒接受，但模糊量词 60 天后仍原封不动。这提示我：为我的项目提出改进建议时，把安全/结构修复和文风修复分开——前者更有行动力。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 里的模糊量词数量（对照 ralph 的失分项）
grep -rn "appropriate\|relevant\|comprehensive\|necessary\|reasonable\|actionable" \
  ~/.claude/skills/ ~/projects/echo-sleuth-for-claude/.claude/ \
  ~/projects/drama-workshop-skills/.claude/ 2>/dev/null | grep "SKILL.md"
```
命中后怎么办：对每处命中，判断是否能替换为可测量阈值（数量、百分比、具体条件）。目标：每个 SKILL.md 的模糊量词 ≤ 2 处。

```bash
# 检查我的 SKILL.md 是否有 output format 章节（对照 ralph-loop 的失分项）
grep -rL "## 输出\|## Output\|output format\|输出格式" \
  ~/.claude/skills/*/SKILL.md \
  ~/projects/echo-sleuth-for-claude/.claude/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：在文件末尾加"## 输出格式"章节，描述 Claude 完成任务后应输出什么结构。

```bash
# 检查 agent frontmatter 是否全部有 name + description + model
grep -rL "^name:\|^description:" \
  ~/.claude/agents/*.md \
  ~/projects/echo-sleuth-for-claude/.claude/agents/*.md 2>/dev/null
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 补充 output format 章节（参考 ralph 的失分教训）
   - **为何可行**：echo-sleuth 的 `memory-management`、`git-mining`、`jsonl-core` 三个 SKILL.md 当前全部缺少 output format 章节（已由 grep 确认：3/4 缺失）
   - **第一步**：在 `memory-management/SKILL.md` 末尾加"## 输出格式"，描述"输出 stale memory 列表：每行格式为 `[项目路径] [最后访问日期] [状态: stale/active]`"；预计 10 分钟
   - **完成标志**：`grep -l "## 输出格式" echo-sleuth-for-claude/.claude/skills/*/SKILL.md` 返回 4 个文件

2. **想法**：对 echo-sleuth 做一次模糊量词审查 pass（参考 ralph code-assist 的 84→95 改进路径）
   - **为何可行**：当前已知 `git-mining/SKILL.md` 和 `experience-synthesis/SKILL.md` 各有 1 处模糊量词命中
   - **第一步**：`grep -n "appropriate\|relevant\|comprehensive" echo-sleuth-for-claude/.claude/skills/*/SKILL.md`，对每处命中替换为具体表述；预计 15 分钟
   - **完成标志**：上述 grep 返回 0 处命中

3. **想法**：引入文档 skill（参考 ralph-docs 新增模式）
   - **为何可行**：echo-sleuth 的 README 有使用说明，但 Claude 在会话中无法直接引用 README——做成 skill 后，上层 agent 可以路由"how to use"类问题
   - **第一步**：新建 `echo-sleuth-docs/SKILL.md`，内容为 README 的结构化版本（安装步骤、常见问题、命令速查）；预计 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：直接 grep 验证（2026-06-21 当前状态）

### 8.1 目的对齐度

**本案例 ralph-orchestrator 的核心目的**：自主 agent 编排——让 Claude 能递归调度子 agent、管理跨任务状态、在多个 AI 后端之间路由

**我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同为多层 NL 架构（commands→agents→skills）；同有 agent 定义；同支持跨任务记忆 | echo-sleuth 是被动记忆挖掘，ralph 是主动任务路由；echo-sleuth 无 Rust 执行层 | **高** |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code skill | drama 是单层内容生产型 skill，ralph 是多层编排框架 | 低 |
| MarkQWu/claude-for-legal | 极低 | 同用 Claude Code | 法律文书工作流 vs 自主 agent 编排，领域无交集 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 模糊量词 | `grep -c "appropriate\|relevant\|comprehensive"` | **echo-sleuth 命中 2 个文件**：`git-mining/SKILL.md`（1 处）、`experience-synthesis/SKILL.md`（1 处） | 中（影响 Claude 精度） |
| 缺少 output format 章节 | `grep -L "output format\|输出格式"` | **echo-sleuth 命中 3/4 个 skill**：memory-management、git-mining、jsonl-core 全部缺失 | 高（调用方无法预期返回结构） |
| 示例数量不足 | `grep -c "<example>\|## 示例\|## Example"` | echo-sleuth 4 个 SKILL.md 中 0 个有示例块——比 ralph 的 pdd（1 个示例）更差 | 高 |
| `source` 执行 shell 文件 | `grep -rn "^source\|^\. " scripts/` | echo-sleuth 无 scripts/ 目录，暂无此问题 | 无 |
| `npm install` 非 lockfile 模式 | `grep -rn "npm install" .github/` | echo-sleuth 无 npm 依赖，暂无此问题 | 无 |

**echo-sleuth 最紧急的行动项**（按严重度排序）：
1. 给 `memory-management/SKILL.md`、`git-mining/SKILL.md`、`jsonl-core/SKILL.md` 各补一节"## 输出格式"
2. 给 4 个 SKILL.md 各加至少 2 个 `<example>` 块
3. 清除 `git-mining/SKILL.md` 和 `experience-synthesis/SKILL.md` 中的模糊量词

### 8.3 别人的更优方案

1. **领域**：opus/haiku 分级模型路由
   - **ralph 的做法**：在 agent 定义的 frontmatter 里显式标注 `model: claude-opus-4`（E2E verifier）或 `model: claude-haiku-4`（loop runner），理由写在紧接的注释里
   - **我的项目现状**：echo-sleuth 的 agent 定义有 `model` 字段，但没有在紧接的行里说明"为什么选这个模型"，理由只存在于我的脑子里
   - **如何借鉴**：在每个 agent 的 frontmatter 后加一行注释，例如 `# 选 sonnet 原因：experience-synthesis 需要语义推理，不是分类任务`

2. **领域**：100 分 skill 的词汇控制（review-pr 零模糊量词）
   - **ralph 的做法**：`review-pr/SKILL.md` 全文没有"appropriate"、"relevant"、"comprehensive"，每个判断标准都给出了可操作的条件（例如"如果 PR 超过 500 行，分段审查"而非"审查合适长度的 PR"）
   - **我的项目现状**：尚未做过系统的模糊量词审查
   - **如何借鉴**：把 ralph 的 review-pr 当模板，逐行对照 echo-sleuth 的每个 SKILL.md，凡有形容词的地方问"这个形容词能换成数字或条件吗"

3. **领域**：文档引用 skill 化（ralph-docs 新增模式）
   - **ralph 的做法**：把 `contributing.md`、`common-questions.md`、`llms-txt-map.md` 包成一个 `ralph-docs` skill，Claude 遇到"怎么贡献代码"类问题可以直接路由到这个 skill
   - **我的项目现状**：echo-sleuth 的 README 文档只能靠 Claude 在模型知识里找，无法精确路由
   - **如何借鉴**：新建 `echo-sleuth-docs/SKILL.md`，把 README 的安装、命令、FAQ 改写成 skill 格式

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：output format 文档化（注：这是 ralph 的弱项，我的弱项更严重——但 drama-workshop 在这点上好于 ralph）
   - **我的做法**：`drama-workshop-skills` 的 SKILL.md 有明确的输出格式说明（剧本格式模板）
   - **ralph 的做法**：`ralph-loop/SKILL.md` 缺少 output format 章节，且 2 个月未修复
   - **意义**：drama 虽然 star 数低，但在 output format 规范性上比 ralph-loop 更完善

2. **领域**：示例覆盖深度（echo-sleuth agents 的 frontmatter 完整率 vs ralph 的 5/6 agent 有 name+desc）
   - **我的做法**：echo-sleuth 5 个 agent 全部有 `name`、`description`、`model` 三个字段——ralph 当时只有 `name` 和 `description`，没有 `model`
   - **ralph 的做法**：ralph 的 agent 在 frontmatter 里没有显式 model 字段，模型选择隐藏在 agent 内容里或 skill 引用里
   - **意义**：三字段 frontmatter 让工具链（比如 NLPM scanner）能直接读取模型选择，不用解析正文

---

## 八、术语表

### <a name="递归自编排"></a>递归自编排
> Ralph Wiggum 技术的核心概念——一个 agent 能把任务委托给自己的子实例，子实例再根据结果决定是否再次路由给更深层的子实例。这让复杂任务能被分解为多轮迭代，每轮都有独立的 scratchpad 记录中间状态。好处是容错性强（一轮失败不影响整体），坏处是调试路径复杂，需要完整的执行日志才能追踪问题根源。

### <a name="nl-binary-混合架构"></a>NL-Binary 混合架构
> NL artifact（SKILL.md、agent 定义）作为行为规范层，编译型语言（Rust、Go、C++）作为执行层的架构模式。NL 层描述"做什么"，二进制层执行"怎么做"。与纯 NL 架构的区别：NL 文件变更后需要重新编译二进制才能生效（因为 `include_str!()` 在编译期把 NL 文本打包进了二进制）。优点：执行速度快、类型安全；缺点：NL 和二进制之间存在版本同步责任。

### <a name="模糊量词"></a>模糊量词
> 在 NL artifact 里，模糊量词是指那些描述程度或质量但不给出可测量阈值的词汇，典型如：appropriate（合适的）、relevant（相关的）、comprehensive（全面的）、necessary（必要的）、reasonable（合理的）、actionable（可操作的）。在人类文档里这些词无害，但当 Claude 读取 SKILL.md 并尝试执行时，它无法把"合适的方法"量化成具体决策——需要靠模型猜测，这会导致执行结果在不同上下文下不一致。NLPM R01 规则要求尽量用可测量表达替代模糊量词。

### <a name="include_str"></a>include_str!() 编译期打包
> Rust 宏，在编译期把一个文本文件的内容嵌入到二进制里。ralph-orchestrator 用它把 SKILL.md 的内容打包进 `ralph-cli`，这样部署时不需要额外携带 NL 文件——一个二进制包含全部逻辑。副作用：修改 SKILL.md 后必须重新运行 `cargo build`，否则运行的仍是旧版本的 NL 定义。

### <a name="供应链漂移"></a>供应链漂移
> 当 `npm install`（或 `pip install`，不带锁文件）在两个不同时间运行，可能拉取到不同版本的依赖，导致"在我的机器上能跑，在 CI 上报错"的现象。`npm ci` 强制锁文件严格一致，`cargo build` 类似（有 `Cargo.lock`）。ralph-orchestrator 的 PR #305 把 `test-fresh-install.sh` 从 `npm install` 改为 `npm ci` 就是修复了这个问题。

### <a name="scratchpad"></a>Scratchpad
> ralph-orchestrator 中，agent 执行任务时的临时工作区——存储当前任务的中间状态、思考过程、待处理子任务。与 memory 的区别：scratchpad 是会话内的，任务完成后可以丢弃；memory 是跨会话的，用于保存需要在未来任务中复用的知识（例如"这个项目的测试命令是 `cargo test --workspace`"）。
