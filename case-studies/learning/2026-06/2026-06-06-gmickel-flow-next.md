# gmickel/flow-next — 学习案例

**仓库**：https://github.com/gmickel/flow-next
**Stars**：568 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-06-06（基于当前 HEAD）
**主题标签**：`security-gate`, `model-pinning`, `cross-reference`, `single-purpose`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

flow-next 是一个针对结构化软件开发工作流的 Claude Code 插件（568 stars），提供完整的开发生命周期流水线：**访谈 → 规划 → 执行 → 评审 → 史诗管理**。这个仓库同时包含两个插件版本（v1 的 `plugins/flow` 和 v2 的 `plugins/flow-next`）、一个 Codex 兼容副本（`codex/`）、一套终端 UI（`flow-next-tui/`）以及一个编排整个系统的 CLI 工具 `flowctl`。

关键事实：
- **双版本共存**：v1（flow）和 v2（flow-next）同时存在于同一仓库，支持两组用户并行迁移
- **规模庞大**：87 个 NL 制品（11 个命令 + 15 个 agent + 28 个 skills 双路径），加上 TUI 和 CLI
- **Ralph 模式**：自主 agent 循环功能，通过 `ralph-guard.py` 守卫，实现持续迭代开发
- **Codex 同步**：`sync-codex.sh` 自动生成 Codex 兼容的 skills 副本，model 映射由 CLAUDE.md 记录

### 1.2 架构剖析

```
plugins/
  flow/               ← v1（5 命令 + 6 agent + 5 skills）
    agents/           ← context-scout, repo-scout, docs-scout, quality-auditor,
    |                    flow-gap-analyst, practice-scout
    commands/flow/    ← work.md, plan.md, plan-review.md, impl-review.md, interview.md
    skills/           ← flow-plan, flow-work, flow-impl-review, flow-interview,
                         rp-explorer, worktree-kit
  flow-next/          ← v2（11 命令 + 15 agent + 14+14 skills 双路径）
    agents/           ← 15 个 scout + plan-sync, memory-scout, docs-gap-scout,
    |                    github-scout, worker, context-scout（6 个满分 100/100）
    commands/flow-next/ ← work, plan, plan-review, epic-review, uninstall, setup,
                          prime, impl-review, interview, ralph-init, sync
    skills/flow-next-*/ ← 14 个 skills（正式版）
    codex/skills/     ← 14 个 skills（Codex 自适应版，sync-codex.sh 生成）
    hooks/hooks.json  ← 条件化依赖 ralph-guard.py 的存在
    scripts/          ← flowctl.py, ralph-guard.py, smoke tests, e2e tests
flow-next-tui/        ← 终端 UI
scripts/              ← bump.sh, sync-codex.sh, install-codex.sh
```

- **编排层级**：命令 → agent（并行 scout）→ skills（知识库），三层完全分离
- **双路径设计**：`skills/flow-next-*/`（Claude 版）与 `codex/skills/`（Codex 版）通过脚本保持同步
- **跨件契约**：所有 agent 引用的 skills 均已验证存在，无悬空引用

### 1.3 设计思路 / 方法论

- **核心哲学**：「同仓库双版本」——v1 和 v2 并存，新用户用 v2，老用户可以继续用 v1 直到稳定再迁移，不强制升级
- **解决什么问题**：大型软件项目的 AI 辅助开发缺乏结构化——没有统一的访谈模板、规划格式、评审标准。flow-next 用 11 个命令把整个开发周期 AI 化
- **做了什么 trade-off**：功能完整性（11 个命令、87 个制品）vs 安全复杂性（`flowctl rp setup-review` 在多个地方被 `eval` 调用，引入了注入面）
- **Ralph 模式的认知模型**：把 Claude Code 当成「可以无限循环运行的 agent」——`ralph-guard.py` 守门，`bypassPermissions: true` 在 hooks 里授权，让 AI 在沙盒中自主迭代

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「双版本迭代 + Codex 同步枢纽」

特征清单：
- **同仓库 v1/v2 并存**：旧用户不被强制迁移，新用户直接用最新版
- **脚本生成副本**：`sync-codex.sh` 从 `flow-next/skills/` 生成 `codex/skills/`，有唯一真相来源
- **15 个 scout agents 并行**：所有 scout 在 plan 阶段并行运行，减少等待时间
- **flowctl CLI 统一入口**：Python 脚本统一封装所有子命令，供 skills 和测试脚本调用

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要支持多个 AI 平台（Claude + Codex 等） | ✅ 高度适用 | 双路径 skills 设计正是为此而生 |
| 大型项目有复杂开发生命周期 | ✅ 适用 | 11 个命令覆盖从访谈到史诗管理 |
| 个人小工具或单一功能插件 | ❌ 过度设计 | 87 个制品对小项目来说维护成本极高 |
| 需要自主 agent 循环（无人值守开发） | ✅ 适用（但需谨慎） | Ralph 模式适合受控环境，安全风险需评估 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 双版本迭代 + Codex 同步（本仓库） | gmickel/flow-next | 不强制迁移，多平台支持，维护有唯一来源 | 仓库体积大，安全面广 |
| 单版本 NL 插件 | expo/skills | 轻量，维护简单 | 不支持平台多样化，迁移代价高 |
| 命令 → agent 分发（无 skills） | wshobson/agents | 层级清晰 | 缺少可复用的知识组件 |
| NL 表皮 + 二进制核心 | jarrodwatts/claude-hud | 性能优，功能无上限 | 需要编译语言开发能力 |

### 2.4 改进空间

1. **当前问题**：`eval "$(...)"` 在多个 skills 的工作流文档里被当作 shell 使用示例写出，`$REVIEW_SUMMARY` 来自用户输入  
   **改进做法**：把 `eval "$($FLOWCTL rp setup-review ... --summary "$REVIEW_SUMMARY")"` 改写为直接调用 `flowctl` 的子命令，完全去掉 eval 层  
   **预期收益**：消除 shell 注入面，解除 security-gate BLOCKED 状态，恢复贡献通道

2. **当前问题**：`epic-scout.md` 中 rules 说「使用 haiku」但 frontmatter 声明 `model: claude-sonnet-4-6`，自相矛盾  
   **改进做法**：要么删除 frontmatter 里的 model 字段（让调用方决定），要么统一把 rules 里的描述改为匹配的 sonnet  
   **预期收益**：消除 model 声明歧义，避免运行时行为不可预测

3. **当前问题**：13 个 flow-next agents 只有单一输出格式示例，缺乏「好/坏对比」  
   **改进做法**：参照 6 个满分 agents 的做法，每个 agent 加一个 before/after 风格的双示例  
   **预期收益**：agent 输出质量更稳定，模型有更强的锚点

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）

当时加权平均评分 **93/100**，Security: **BLOCKED**（Critical×2, High×0, Medium×4, Low×2）。

| 文件 / 类别 | 当时分数 | 主要问题 |
|---|---|---|
| `plugins/flow-next/agents/epic-scout.md` | 90 | model 字段与 rules 矛盾 |
| `plugins/flow/agents/practice-scout.md` | 88 | 年份写死为「2025」 |
| 13 个 flow-next scouts（非满分组） | 95 | 单一输出格式示例 |
| 26+ skills | 100 | 全部满分，无问题 |
| 6 个明星 agents（context-scout 等） | 100 | 参见 §3.2 |

**安全状态**：BLOCKED——Critical 2 个（eval 命令替换注入），Medium 4 个（eval + 用户受控变量），Low 2 个（调试日志泄漏完整 payload）。

### 3.2 当时值得借鉴的模式

1. **满分 agent 的双输出示例**：`context-scout.md`、`worker.md` 等 6 个满分 agents 都有两个输出格式示例（一个精简示例 + 一个完整示例），而不仅仅是单一模板。这种「before/after」或「minimal/full」对比让 AI 有更强的锚点，输出质量更稳定。这是 **agent 设计的最高水准**，87 个制品的加权高分主要来自这 26+ 个 100 分 skills 和 6 个 100 分 agents 的拉动

2. **置信度标签 `[VERIFIED]` / `[INFERRED]`**：flow-next agents 在输出格式里明确区分「已验证的事实」和「推断的内容」。例如 agent 的输出字段里，从 git history 直接读到的数据标 `[VERIFIED]`，从代码结构推断的信息标 `[INFERRED]`。这让下游 agent 或用户能立刻判断哪些信息需要额外核实，哪些可以直接信赖。这是一个**极少见的认知透明度模式**

3. **CLAUDE.md 的 model 映射表**：明确记录了 Claude model 到 Codex model 的对应关系（如 sonnet → gpt-5.4-mini），任何新增 agent 的开发者都能查到应该在 Codex 版里用什么模型。这是**跨平台一致性的文档契约**

4. **`sync-codex.sh` 单一真相来源**：skills 只维护一份，Codex 副本由脚本自动生成，绝不手工维护两份。这杜绝了「两份内容慢慢漂移、最终不一致」的问题

### 3.3 当时的缺陷

1. **问题**：`ralph_e2e_rp_test.sh` 第 323 行和 `ralph_smoke_rp.sh` 第 289 行使用 `eval "$(...)"`，直接 eval 命令替换的输出  
   **根本原因**：`flowctl rp setup-review` 命令的输出被设计为「可以作为 shell 命令执行」，这在开发环境下方便，但在安全角度是高危模式——如果 flowctl 的输出包含恶意内容，eval 会直接执行  
   **自查**：我的 hooks、scripts 里有没有 `eval "$(...)"`  这样的写法？

2. **问题**：skills 的工作流文档（`workflow.md`）里的示例代码包含 `eval "$($FLOWCTL rp setup-review ... --summary "$REVIEW_SUMMARY" --create)"`，`$REVIEW_SUMMARY` 来自用户提供的评审摘要  
   **根本原因**：作者把「flowctl 使用示例」直接写进 skills 文档，而这些示例本身就包含危险用法。用户复制文档里的示例到自己的脚本，就会引入注入漏洞  
   **自查**：我的 skills 文档里有没有「可直接复制但不安全」的示例代码？

3. **问题**：`flow/agents/practice-scout.md` 写死了「Current year is 2025」  
   **根本原因**：年份硬编码，而不是引导 AI 动态判断或从上下文获取。过了 2025 年后这条信息立刻变成错误的  
   **自查**：我的 agents 和 skills 里有没有硬编码年份？

### 3.4 当时的优化机会

1. 在 `ralph_e2e_rp_test.sh` 和 `ralph_smoke_rp.sh` 里，把 `eval "$(...)"` 替换为直接解析 flowctl 输出（JSON 解析或逐行读取），消除 eval 层
2. 在 skills 工作流文档里，把危险的 eval 示例改为安全的替代写法，并加上注释说明为什么不用 eval
3. 把 `practice-scout.md` 的年份硬编码改为「the current year」或通过 `date` 命令动态获取

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷状态（2026-04-16 → 2026-06-06）

| 缺陷 | 位置 | 现状 | 含义 |
|---|---|---|---|
| Critical eval（测试脚本） | `ralph_e2e_rp_test.sh` 第 323 行 | **PERSISTS** | 2 个月未修复 |
| Critical eval（smoke 脚本） | `ralph_smoke_rp.sh` 第 289 行 | **PERSISTS** | 同上 |
| Medium eval（impl-review skill） | `flow-next-impl-review/workflow-rp.md` 第 42 行 | **PERSISTS** | 出现了新路径 workflow-rp.md（原为 workflow.md 第 126 行）|
| Medium eval（plan-review skill） | `flow-next-plan-review/workflow.md` 第 193 行 | **PERSISTS** | 行号变化说明有代码添加 |
| Medium eval（NEW: spec-completion-review） | `flow-next-spec-completion-review/workflow-rp.md` 第 40 行 | **NEW — 新增回归** | 原审查时不存在，现在新增了相同危险模式 |
| Medium eval（NEW: export-context） | `flow-next-export-context/SKILL.md` 第 64 行 | **NEW — 新增回归** | 同上 |
| 年份硬编码 `practice-scout.md` | flow-next 版 | **FIXED** | flow-next 版本已更新为「2026」|
| epic-scout model 矛盾 | `plugins/flow-next/agents/epic-scout.md` | **CANNOT VERIFY** | 当前 HEAD 未找到预期路径 |

**关键观察**：安全状态不仅没有改善，反而发生了**回归**——两个新增的 skills（spec-completion-review、export-context）延续了相同的 eval 危险模式。这说明 eval 用法已经成为这个仓库的「风格惯例」，在新功能里被不断复制。

### 4.2 架构演进

对比 2026-04-16 到现在，flow-next 最显著的变化是：
- **新增 skills**：`flow-next-spec-completion-review` 和 `flow-next-export-context` 是新添加的功能模块
- **`workflow.md` → `workflow-rp.md`**：impl-review skill 分裂出了 RP（Review Prompt）专用路径，说明 Ralph 模式的使用场景在细化
- **年份修复**：flow-next 版本的 practice-scout 年份已更新

NL 制品层（agents、commands）结构基本稳定，核心 bug（eval 模式）在老文件里全部未修复，在新文件里继续复制。

### 4.3 新增模式

当前 HEAD 的 `workflow-rp.md` 路径（相对于原来的 `workflow.md`）反映了一个**分支工作流模式**：同一 skill 有「标准路径」（`workflow.md`）和「Ralph/RP 专用路径」（`workflow-rp.md`）。这个设计允许同一 skill 在自主 agent 循环中表现与交互模式下不同，是个有趣的「同 skill 双行为」模式——但也意味着维护两份几乎相同的工作流文档。

---

## 五、校准

### 5.1 我已经在做对的

1. **没有在 hooks 或 scripts 里使用 eval**：这个案例给了我一个明确的参照——eval + 命令替换是高危模式，不论代码多简洁
2. **命令职责分离**：我的项目里每个命令做单一任务，这与 flow-next 满分 agents 的设计哲学一致
3. **避免年份硬编码**：我已经意识到 NL 制品里不应该写死时间相关的断言

### 5.2 挑战 / 验证

本案例挑战了我的一个假设：「93 分的高质量仓库在安全上应该也是安全的」。

结果发现：93 分的 NL 质量分数与安全状态完全正交——这个仓库质量分高，安全状态是 BLOCKED。**NL 质量高不等于安全**。

更重要的发现：安全问题在两个月内**扩展而非收缩**——新增代码延续了危险模式。这说明如果一个危险编码模式不被显式识别和阻止，它会随着项目增长而蔓延。对我的项目的启示：需要在 CONTRIBUTING 或 CLAUDE.md 里明确写出「禁止 eval + 用户输入」的规范，而不是期待维护者自然意识到。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有仓库里是否有 eval + 命令替换的危险组合
grep -rn 'eval "\$(' ~/.claude/ 2>/dev/null | head -20
grep -rn "eval '\\\$(" ~/.claude/ 2>/dev/null | head -20

# 更广泛地检查任何 eval 用法（不限于命令替换）
grep -rn '\beval\b' ~/.claude/ --include="*.sh" --include="*.py" --include="*.md" | head -20
```

命中后怎么办：找到 eval 的目的——如果是「执行动态生成的 shell 命令」，问自己能否用参数数组直接调用；如果是「解析变量赋值输出」（如 `eval "$(some-tool --export-vars)"`），确认 some-tool 的输出是否完全在你的控制之内，如果有任何用户输入流入输出就必须重写。

```bash
# 检查我的 skills 文档里有没有「可直接复制但含 eval」的示例代码
grep -rn 'eval' ~/.claude/ --include="*.md" | head -20
```

命中后怎么办：如果示例代码里有 eval，要么改写为安全替代方案，要么加上醒目注释「警告：此示例仅供理解流程，不要在生产中使用 eval」。

```bash
# 检查我的 agents 里是否有硬编码年份
grep -rn '202[0-9]' ~/.claude/ --include="*.md" | grep -v "version\|date\|changelog\|commit" | head -20
```

命中后怎么办：把硬编码年份替换为「the current year」或通过动态方式（如 `$(date +%Y)`）获取。

### 6.2 灵感 → 实施路径

1. **想法**：为我的 agents 加置信度标签 `[VERIFIED]` / `[INFERRED]`  
   - **为何可行**：我的 echo-sleuth 的 agents 在挖掘 git history 时，直接从 commit 读到的是 VERIFIED，从代码结构推断的是 INFERRED——这个区别本就存在，只是没有显式标注  
   - **第一步**：在 echo-sleuth 最核心的 agent（比如 memory-scout）的输出格式里，把每个字段加上 `[VERIFIED]` 或 `[INFERRED]` 标记，约 15 分钟

2. **想法**：为 echo-sleuth agents 加双输出示例（minimal + full）  
   - **为何可行**：flow-next 6 个满分 agents 用这个方法把质量推到 100/100，我的 echo-sleuth agents 目前完全没有输出示例  
   - **第一步**：选一个最常用的 agent，手工写一个「最小输出」和一个「完整输出」的示例，插入到 agent 文档的 `## Output Format` 节里，约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 gmickel/flow-next 的核心目的**：结构化 AI 辅助软件开发工作流插件，命令 → agents 并行分发 → skills 知识库

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中高 | 同为命令 → agents → skills 三层架构，同为 Claude Code 插件 | echo-sleuth 是会话分析工具，flow-next 是开发流程编排工具 | **高** |
| MarkQWu/claude-for-legal | 中低 | 同为 Claude Code 制品，同有领域专用 skills | 法律领域，无 agent 编排，无 CLI | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 制品 | 创作领域，结构完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 自查方法 | 预计命中情况 | 严重度 |
|---|---|---|---|
| agents 只有单一输出示例（或完全没有） | `grep -L "Output Format" echo-sleuth/agents/*.md` | **很可能命中**：echo-sleuth agents 目前无输出示例 | 高（影响评分） |
| agents 缺 model 声明 | `grep -L "model:" echo-sleuth/agents/*.md` | **可能命中**：v1 flow agents 有同样缺失 | 中 |
| skills 文档含不安全的示例代码 | `grep -rn 'eval' echo-sleuth/skills/` | 需手动检查 | 高（安全） |

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：置信度标签 `[VERIFIED]` / `[INFERRED]`

   **flow-next 的做法**：agent 输出格式里，每个字段都明确标注信息来源的确定程度。例如：
   ```
   - dependency_count: 42 [VERIFIED — from package.json]
   - test_coverage: ~60% [INFERRED — from test file count vs source file count]
   ```
   从代码直接读到的数据标 `[VERIFIED]`，从结构、比例或模式推断的数据标 `[INFERRED]`。**这让下游 agent 和用户可以基于置信度做决策**——`[VERIFIED]` 的数据可以直接引用，`[INFERRED]` 的数据需要额外核实后再做决策。

   **我的项目现状**：echo-sleuth 的 agents 挖掘 git history，输出里混合了「从 commit message 直接读到的」和「从代码变更量推断的」，完全没有区分标注

   **如何借鉴**：在 echo-sleuth 的 agent 输出格式里加 `[VERIFIED]` / `[INFERRED]` 规范，并在 SKILL.md 里说明两者的定义和使用规则

2. **领域**：双输出示例（minimal + full）

   **flow-next 的做法**：6 个满分 agents 在 `## Output Format` 节里提供两个示例——一个精简示例展示最小可接受输出，一个完整示例展示理想输出。AI 模型有两个锚点，比单一示例更能生成风格稳定的输出

   **我的项目现状**：echo-sleuth agents 完全没有输出示例

   **如何借鉴**：为每个 agent 写 minimal（3-5 行）和 full（20-30 行）两个示例，直接在文档末尾加 `## Examples` 节

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全编码意识（至少在 NL 制品层）
- **我的做法**：echo-sleuth 和 claude-for-legal 的 skills 文档里没有包含 `eval` 命令的示例代码，不会通过文档把危险模式传播给用户
- **flow-next 弱在哪**：skills 的工作流文档被用来教用户「如何配合 flowctl 使用」，而这些教学示例本身包含 `eval` 用法——文档即传播媒介，危险模式随文档传播给每一个复制示例的用户
- **意义**：编写 skills 文档时，**示例代码需要像生产代码一样接受安全审查**。「这只是文档」不是豁免安全标准的理由

---

## 八、术语表

### <a name="eval-injection"></a>eval 注入
> 一种 shell 安全漏洞。`eval "$(some_command)"` 的意思是：先运行 `some_command`，把它的输出当作 shell 代码执行。如果 `some_command` 的输出里有来自用户输入的内容（比如评审摘要文本），攻击者只要在文本里插入 `; rm -rf ~` 或类似内容，eval 就会把它当命令执行。**简单记忆：eval 是「把字符串当命令」，只要字符串来源不是百分之百可信的代码，就不要用 eval。** 安全替代方案：解析结构化输出（JSON），或把命令拆成二进制 + 参数数组直接调用。

### <a name="flowctl"></a>flowctl
> flow-next 仓库的 CLI 编排工具（`scripts/flowctl.py`）。它封装了所有子命令（`plan`、`work`、`rp setup-review` 等），供 skills 文档里的工作流示例和测试脚本调用。设计目标是统一的命令入口，避免每个脚本直接调用底层 API。当前版本的 `rp setup-review` 子命令被设计为「输出可 eval 的 shell 代码」，这是 Critical 安全问题的根源。

### <a name="codex"></a>Codex
> OpenAI 的 Codex 模型/平台，与 Claude Code 是竞品。flow-next 支持 Codex 的方式是：在 `codex/skills/` 目录下维护一套 Codex 适配版 skills，由 `sync-codex.sh` 从 Claude 版自动生成，model 映射关系（如 sonnet → gpt-5.4-mini）记录在 CLAUDE.md 里。这是「一套内容，多平台发布」的典型工程实践。

### <a name="ralph"></a>Ralph 模式
> flow-next 的自主 agent 循环功能。启用后，Claude Code 可以在没有人工介入的情况下循环执行「规划 → 执行 → 评审」的开发周期。`ralph-guard.py` 作为安全守卫，通过 hooks 机制在关键点拦截。hooks.json 里的 `bypassPermissions: true` 允许 Ralph 模式绕过某些权限确认，这是 Low 级安全风险（完整 payload 被写入 `/tmp/ralph-guard-debug.log`）。

### <a name="worktree"></a>worktree
> Git 的多工作树功能（`git worktree`），允许同一个 Git 仓库在多个目录下同时 check out 不同分支。flow-next 用它支持「在同一机器上并行开发多个功能分支」，`worktree-kit` skill 封装了相关操作指南。

### <a name="tui"></a>TUI（Terminal UI）
> 终端用户界面（Terminal User Interface），在命令行里用文字绘制出类似 GUI 的交互界面（带边框、颜色、菜单等）。`flow-next-tui/` 目录包含 flow-next 的终端 UI 组件，让用户可以在终端里以可视化方式管理开发流程，而不是只通过命令行参数交互。

### <a name="bypassPermissions"></a>bypassPermissions
> Claude Code hooks 配置里的一个字段，设为 `true` 时允许该 hook 触发的操作跳过正常的用户权限确认弹窗。flow-next 在 `hooks/hooks.json` 里用这个字段支持 Ralph 模式的无人值守运行。**风险**：如果 hook 的触发条件写错，或者守卫脚本（ralph-guard.py）被绕过，bypassPermissions 会允许 Claude 执行通常需要用户确认的高危操作。
