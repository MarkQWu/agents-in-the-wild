# jnuyens/gsd-plugin — 学习案例

**仓库**：https://github.com/jnuyens/gsd-plugin
**Stars**：9 | **来源**：upstream audit
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-08-04（基于当前 HEAD v2.45.0）
**主题标签**：`model-pinning`, `vague-quantifier`, `examples-driven`, `single-purpose`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

jnuyens/gsd-plugin（GSD = Get Stuff Done）是一个围绕「项目里程碑生命周期管理」构建的 Claude Code 插件，v2.38.8（audit 时）→ v2.45.0（当前），包含 33 个 agent + 90 个 skill（audit 时为 82），是本批次案例中**规模最大的生产级 NL 工件库**。插件核心是一套「规划 → 研究 → 执行 → 验证 → 复盘」的工作流引擎，所有组件通过 `@${CLAUDE_PLUGIN_ROOT}` 变量做跨件引用。

关键事实：
- 插件是个人项目，无公司背书，但规模和成熟度超过多数团队维护的插件
- NLPM 总分 90.3/100，SECURITY CLEAR，是罕见的「大体量 + 无安全问题」案例
- 新增 `mcp/` 目录（独立 MCP 服务器）、`sdk/` 目录（外部 SDK 接口）、`contexts/` 目录（上下文预置）
- hooks 从 4 个简单事件处理器演进为「共存竞选（coexistence election）」机制，支持同时安装多个版本

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
jnuyens/gsd-plugin/
├── agents/             (33 agents — 完整研发生命周期覆盖)
├── skills/             (90 skills，audit 时 82 个)
├── workflows/          (跨 agent 工作流编排)
├── templates/          (输出模板，agents 读取)
├── references/         (共享参考文档)
├── contexts/           (上下文预置，NEW)
├── hooks/              (hooks.json + 8个 detector JS 文件 + run-bash-hook.cjs)
├── mcp/                (独立 MCP 服务器，NEW)
├── sdk/                (外部 SDK 接口，NEW)
├── bin/                (gsd-tools.cjs 二进制 — hook 调度核心)
├── migrations/         (跨版本迁移脚本)
├── dist/               (构建产物)
└── package.json        (v2.45.0)
```

**文件类型分布**：33 个 agent / 90 个 skill / 8 个 hook 检测器 / 1 个 hook dispatcher（gsd-tools.cjs）

**编排关系**：
- `gsd-planner` 是「中枢 agent」，读取 `.planning/` 状态机文件，协调其他 agent 执行
- 「研究管线」：`gsd:research-phase` skill → `gsd-project-researcher`（生产 6 个并行子文件）→ `gsd-research-synthesizer`（消费并合并）
- 「调试链」：`gsd:debug` skill → `gsd-debug-session-manager`（管理状态文件）→ `gsd-debugger`（执行）
- hooks.json 的 `gsd-tools.cjs` 是运行时调度器，实现「版本共存竞选」——两个版本的插件同时安装时，只有一个版本的 hook 实际执行

**跨件契约**：
- `@${CLAUDE_PLUGIN_ROOT}/workflows/*.md` 是跨 skill/agent 的共享工作流协议，运行时由 Claude Code 解析
- `.planning/*.md` 是 hook 和 agent 之间共享的状态文件（约定大于配置）
- `gsd-research-synthesizer` 通过读取 section header（而非类型定义）消费 `gsd-project-researcher` 的产出 → 隐式契约，存在漂移风险

### 1.3 设计思路 / 方法论

**核心设计哲学**：「GSD（Get Stuff Done）」= 每件事都有明确的完成标准。每个 agent 的职责边界是「一个工程岗位的一个职责」，每个 skill 的完成标准是「成功条件（success criteria）可验证」。

**解决什么问题**：大型项目里，工程师和 AI 在不同会话间「忘记」进展，导致重复工作或方向偏移。GSD 通过 `.planning/` 状态机文件、里程碑追踪、阶段验证来维持跨会话一致性。

**Trade-off**：
- 大而全（33 agent + 90 skill）vs 精而深：选了大而全，换来覆盖整个研发周期；代价是模型选择（`model:` 字段）全部为默认值，无法针对任务复杂度优化成本
- 隐式跨件契约（section header）vs 显式类型定义：选了隐式，减少了 schema 维护成本；代价是「synthesizer 依赖 researcher 输出格式」的断裂不可静态检测
- hooks.json 内联 minified JS vs 外部脚本：当前版本演进到「大量独立 detector 文件 + 单一 dispatcher」，接近了审计报告的建议

**认知模型**：作者把整个研发周期看作「一系列可验证的阶段门（phase gates）」。每个 skill 不只是一个操作，而是一个「带成功标准的合同」：执行前有前提条件，执行后有验证步骤。这与敏捷里程碑的「DoD（Definition of Done）」理念一致。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：状态文件驱动的跨 agent 工作流（State-File Driven Agent Orchestration）**

GSD 的不同 agent 不通过直接调用互相通信，而是通过读写 `.planning/*.md` 状态文件来传递上下文。`gsd-debug-session-manager` 写入 `debug/{slug}.md`，`gsd-debugger` 读取它继续调试。这是一种「黑板架构（Blackboard Pattern）」的 NL 实现。

模式特征清单：
- **异步解耦**：生产者 agent 和消费者 agent 不需要同时运行
- **可审计**：状态文件是人类可读的 Markdown，整个工作流历史可在 `.planning/` 目录查看
- **跨会话持久**：Claude Code 会话结束后状态保留，下次会话可从断点继续
- **零 API 通信**：agent 间靠文件系统传递信息，无需 WebSocket 或消息队列
- **脆弱性：隐式 schema**：状态文件格式在 skill 文档里描述，但无类型系统强制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多天、多会话的大型项目开发 | ✅ 高度适用 | 状态持久化正是为了解决跨会话连续性问题 |
| 需要多 agent 协作的复杂工作流 | ✅ 适用 | 状态文件作为协作介质比直接调用更灵活 |
| 简单的单会话一次性任务 | ❌ 不适用 | 引入状态文件是过度工程；直接 skill 调用更简洁 |
| 需要强类型 agent 接口的企业场景 | ❌ 不适用 | 隐式 schema 在大型团队中容易漂移，需要类型系统 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 状态文件驱动（本仓库） | jnuyens/gsd-plugin | 跨会话持久；人类可审计；无中间件依赖 | 隐式 schema；文件冲突风险；无事务保证 |
| 直接 agent 调用 | webdevtodayjason/sub-agents | 简单直接；调用链清晰 | 跨会话丢失状态；agent 间耦合高 |
| MCP 服务器传递状态 | thedotmack/claude-mem | 结构化查询；强类型接口 | 需要服务器进程；复杂度高 |

### 2.4 改进空间

1. **当前问题**：`gsd-research-synthesizer` 通过读取 section header 消费 `gsd-project-researcher` 的输出，隐式契约脆弱。**改进做法**：在 `references/` 目录新建 `research-output-schema.md`，明确 6 个并行子文件的 section 名称和格式；两个 agent 都引用这个 schema。**预期收益**：一处修改输出格式，两个 agent 同步更新；不再有「synthesizer 读到空 section」的静默失败。

2. **当前问题**：33 个 agent 全部缺 `model:` 字段，统一用默认 Sonnet，无法按任务复杂度分配模型。**改进做法**：把 33 个 agent 按「分类器（Haiku）/ 规划器（Sonnet）/ 调试器（Opus）」分三组，各自加 1 行 `model: haiku/sonnet/opus`。**预期收益**：`gsd-doc-classifier` 用 Haiku 降低 80% 成本；`gsd-debugger` 用更高能力模型提升精度；平均成本显著下降。

3. **当前问题**：skills/ 里 9 个「薄委托」skill（audit-uat、cleanup 等）得分 75，缺示例和输出格式。**改进做法**：为每个薄 skill 加一个 `## Output Format` section 和一个 `<usage>` 示例，仿照 `skills/complete-milestone/SKILL.md`（100 分参考实现）。**预期收益**：9 个 skill 从 75 分升至约 90 分，整体平均分从 90.3 升至约 93。

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）

jnuyens/gsd-plugin 在 2026-04-28 得分 **90.3/100**（agents 88.7 / skills 90.9）。

| 类别 | 得分 | 主要问题 |
|---|---|---|
| Agents（33个）| 88.7/100 | 0/33 有 model 字段（-5 × 33 = -165 pts）；3 个 agent 零示例 |
| Skills（82个）| 90.9/100 | 9 个薄委托 skill 得 75（缺示例+缺输出格式） |
| **总体** | **90.3/100** | — |

### 3.2 当时值得借鉴的模式

1. **100 分 skill 参考实现** → `skills/complete-milestone/SKILL.md`（8 步、多示例、输出格式）、`skills/reapply-patches/SKILL.md`（7 步、三路合并逻辑、Hunk Verification Table）。根本原因：步骤清晰 + 验证标准 + 输出格式三者齐全，AI 执行时不会「猜」。→ 把这两个 skill 作为自己写 SKILL.md 的 checklist。

2. **并行研究 agent 模式** → `gsd-project-researcher` 并行生产 6 个专题子文件（SUMMARY、STACK、ARCHITECTURE 等）→ `gsd-research-synthesizer` 合并。根本原因：并行执行降低总时间，专题化分工降低每个 agent 的认知负荷。→ 自己有「大型调研任务」时，可以仿照这个「扇出写文件 + 扇入合并」模式。

3. **hooks.json 里的防御性 SDK shadowing 检测** → `gsd-shadowing-sdk-detector.js`：在 SessionStart 检测用户项目是否有 `@anthropic-ai/sdk` 被本地版本覆盖（shadowed），如果是则发出警告。根本原因：这种 SDK shadowing 问题很隐蔽，claude-code-action 的 CI 场景里曾多次出现。把安全检测写成 hook 而非 skill，是因为它是「系统级约束」而非「用户主动操作」。→ 把「必须强制执行的约束」写成 hook，而非 skill。

4. **`skills/intel/SKILL.md` 的反模式文档** → intel skill 在描述「如何更新竞品情报」时，专门有一个 `## Anti-patterns` section 列出「不能做的事」（不能删除情报不确认已过时、不能用主观评价代替事实引用等）。根本原因：正面示例告诉 AI 做什么，反模式告诉 AI 不做什么，两者结合才能精确约束边界。→ 为自己的 skill 同时写「正面示例」和「反模式」。

### 3.3 当时的缺陷

1. **Q-01：33 个 agent 全部缺 model 字段**
   - **根本原因**：作者在设计 agent 时，没有把「选择模型」作为 agent 设计决策之一。默认模型（Sonnet）对所有任务都能运行，所以缺失 `model:` 字段不会立刻暴露问题，但造成了「分类器用 Sonnet 的高成本浪费」和「调试器用 Sonnet 而非 Opus 的精度损失」两个看不见的问题。
   - **自查**：我的 bureau agents 有没有 model 字段？如果没有，哪些 agent 应该用 Haiku，哪些应该用 Sonnet？

2. **Q-04：薄委托 skill 缺输出格式和示例**
   - **根本原因**：薄委托 skill 的职责是「调度 workflow」，作者认为「有 workflow 链接就够了」，忽略了 AI 在执行这个 skill 时需要知道「最终产出是什么格式」。这是「协议 vs 说明书」的混淆：workflow 是协议，但 skill 需要说明书。
   - **自查**：我的 bureau skills 有没有类似的「只有 workflow 链接，没有输出格式」的情况？

3. **Cross-component：研究 synthesizer 的隐式 schema**
   - **根本原因**：`gsd-project-researcher` 和 `gsd-research-synthesizer` 之间的接口靠「section header 名称约定」维持，没有任何静态检查。两个 agent 各自修改时可能不同步，导致 synthesizer 读到空 section 时静默降级。
   - **自查**：bureau 的 `capture → compile` 管线是否有类似的隐式 schema？如果 capture 输出格式变化，compile 能否感知？

### 3.4 当时的优化机会

1. **batch PR：为所有 33 个 agent 加 model 字段**（分 Haiku/Sonnet/Opus 三组）：一次性修复，对成本影响最大。
2. **为 9 个薄 skill 加输出格式 + 使用示例**：每个 skill 修改约 10 分钟，共约 1.5 小时。
3. **PostToolUse hook 缩小 matcher 范围**：移除 Read/Grep/Glob，减少不必要的 hook 调用延迟。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Q-01：33 agent 缺 model | `grep -l "^model:" agents/*.md \| wc -l` | **持续** — 仍为 0/33，`gsd-doc-classifier` 依然没有 `model: haiku` | 4 个月内未修，说明 model pinning 没有进入作者的维护 checklist |
| Q-02：3 agent 零示例 | `grep -c "<example>" agents/gsd-advisor-researcher.md` | **持续** — gsd-advisor-researcher / gsd-assumptions-analyzer / gsd-eval-auditor 仍为 0 | 同上 |
| Q-04：9 薄 skill 缺示例 | `grep -c "## Example" skills/cleanup/SKILL.md` | **持续** — cleanup/SKILL.md 仍无示例无输出格式，得分仍为 75 | 批量修复 9 个 skill 属于「打包工作」，没有被排上议程 |
| Security：PostToolUse 对 Read 过度触发 | 检查 hooks.json matcher | **架构演进** — hooks 系统大幅重构为「dispatcher + 多 detector」，旧的 4 个简单 hook 已替换；新 hook 架构更复杂 | 旧的单点 PostToolUse 问题已不再适用，但新 hooks 系统本身更复杂，有新的可审计性挑战 |

### 4.2 架构演进

- **hooks 从「4 个简单事件处理器」→「共存竞选（Coexistence Election）架构」**：当用户同时安装了两个版本的 gsd-plugin，新机制确保每个 hook 只执行一次（无论哪个版本触发）。这是一个非常具体的工程问题：版本升级时用户可能同时有旧版和新版，双触发会导致 `.planning/` 状态文件损坏。
- **新增 8 个专用 detector hook**：`gsd-shadowing-sdk-detector.js`、`gsd-staleness-reminder.js`、`gsd-prompt-guard.js`、`gsd-workflow-guard.js`、`gsd-read-guard.js`、`gsd-auth-detector.js`、`gsd-read-injection-scanner.js`、`gsd-context-monitor.js`。每个 detector 专注一个安全/质量场景，比之前的单体 hook 更模块化、更可审计。
- **新增 mcp/ 目录**：提供独立 MCP 服务器接口，允许外部工具（非 Claude Code 的 AI 代理）访问 GSD 状态。
- **新增 sdk/ 目录**：提供 SDK 接口，支持 GSD 状态的编程访问，说明作者开始把 GSD 从「个人工具」定位为「可集成的平台」。

### 4.3 新增的可学习模式

**Coexistence Election 模式**：同一 plugin 的两个版本共存时，通过「版本竞选」机制确保每个 hook 只执行一次。实现方式：在 `gsd-tools.cjs` 的 `case 'hook'` 里加共享竞选点，被竞选中的版本执行，另一个版本 yield（放弃执行）。这个模式解决了「Claude Code 插件升级的零停机问题」，在其他仓库里从未见过。

---

## 五、校准

### 5.1 我已经在做对的

1. **skills 有具体的输出格式描述**：bureau/skills/lint/SKILL.md 有「输出：findings.md 到 canon/health/YYYY-MM-DD-lint.md」这样的具体路径，比 GSD 的薄委托 skill 强。
2. **commands 的参数有 default 处理**：bureau 的 command 用 `{{ args[0] | default('') }}` 处理空参数，比 c0x12c 和 gsd-plugin 都更防御性。
3. **安全设计意识**：bureau 的权限模型有「人工门控」层，比 gsd-plugin 更保守。

### 5.2 挑战 / 验证

本案例验证了「大规模 NL 工件库的维护惰性」：90 分仓库的 4 个主要缺陷在 4 个月后全部持续存在。这不是作者能力问题，而是说明：**NL 工件的维护需要工具化支持（如 NLPM hook）**，仅靠人工 PR 没有规律性触发机制，质量会停滞。

对我的启发：bureau 和 gstack 也需要一个「定期质量检查」的机制（如每次 push 时跑 nlpm-check），否则跟 gsd-plugin 面临同样的命运——3个月后这些问题仍然存在。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否有 model 字段
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau \
  -name "*.md" -path "*/agents/*" | xargs grep -L "^model:" 2>/dev/null
# 命中后：评估任务复杂度，分配 haiku/sonnet

find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau \
  -name "SKILL.md" | xargs grep -L "<example>\|## Example" 2>/dev/null
# 命中后：为每个缺示例的 SKILL.md 加一个 ## Example section
```

```bash
# 检查 bureau 的 capture→compile 管线是否有隐式 schema
grep -n "section\|format\|schema\|output" /tmp/my-repos/MarkQWu-bureau/skills/capture/SKILL.md 2>/dev/null
grep -n "section\|format\|schema\|input" /tmp/my-repos/MarkQWu-bureau/skills/compile/SKILL.md 2>/dev/null
# 如果两个文件的「接口契约」不一致，说明有隐式 schema 漂移风险
```

### 6.2 灵感 → 实施路径

1. **想法**：仿照 GSD 的「Coexistence Election」模式，为 bureau 的 hook 加版本检查
   - **为何可行**：bureau 也有 hooks，如果同时安装了两个版本，可能导致 double-trigger
   - **第一步**：在 `hooks/hooks.json` 的 PostToolUse handler 里加一行：检查 `CLAUDE_PLUGIN_ROOT` 是否与当前运行版本匹配，如果不匹配则 exit 0（让另一个版本执行）；约 30 分钟

2. **想法**：仿照 GSD 的「状态文件驱动」，把 bureau 的 capture → review → compile 管线用 `.bureau-state/` 状态文件串联
   - **为何可行**：bureau 目前靠 human 手动触发每个步骤，丢失了「当前有多少 logbook minutes 待 compile」的状态感知
   - **第一步**：在 `bureau:capture` 命令末尾加一步：写入 `.bureau-state/pending.json`（记录未处理的 minutes 数量）；`bureau:status` 读取这个文件展示；约 45 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 jnuyens/gsd-plugin 的核心目的**：为多天、多会话的大型项目提供结构化里程碑管理和跨 agent 工作流编排。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 同样是「AI 辅助知识管理 + 跨会话持久化」 | bureau 管理的是「记忆/事实」；GSD 管理的是「项目里程碑/任务」 | 中 |
| MarkQWu/gstack | 低 | 都是「工程工作流」插件 | gstack 是角色模拟（CEO/CTO）；GSD 是阶段门管理 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Q-01：agent 缺 model 字段 | `grep -rL "^model:" */agents/*.md` | gstack 仅 1 个 agent（openai.yaml），命中但影响小 | 低 |
| Q-04：skill 缺示例和输出格式 | 检查 bureau/skills/*/SKILL.md | **需进一步核查** — bureau skills 可能有「只有 workflow 引用，没有示例」的情况 | 中 |
| 隐式跨件 schema | capture→compile 管线契约检查 | **潜在风险** — bureau 的 capture 和 compile 之间的格式约定未成文 | 中 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/skills/capture/SKILL.md` → 在末尾加 `## Output Format` section，明确写出「输出文件的路径、YAML 字段、status 枚举」 → 20 分钟
- `MarkQWu-bureau/skills/compile/SKILL.md` → 在开头加 `## Input Contract` section，列出期望从 logbook 读到的字段 → 10 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：成功标准（success criteria）内置于 skill
   - **本案例做法**：`skills/complete-milestone/SKILL.md` 在 8 步流程结束时有明确的「验证步骤」和 `## Success Criteria`，Claude 执行后自知是否完成
   - **我的项目现状**：`MarkQWu-bureau/skills/review/SKILL.md` — 需确认是否有成功标准描述
   - **如何借鉴**：为每个 bureau skill 末尾加 `## Success Criteria` section，1-3 条可验证标准

2. **领域**：专用 detector hook 的模块化
   - **本案例做法**：8 个独立的 JS detector，每个专注一个安全/质量场景（shadowing、staleness、prompt injection 等）
   - **我的项目现状**：bureau 的 hooks 目前是单体，如果要加 3+ 个检测逻辑，会变成难以维护的单体 hook
   - **如何借鉴**：把 bureau hook 的「reviewer token 检查」和「canon 路径保护」逻辑拆分为两个独立的 hook 文件；通过 hooks.json 分别注册

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：skill 示例质量
- **我的做法**：`MarkQWu-bureau` 的 command（如 `lint.md`、`review.md`）有完整的 step-by-step 流程，且每步都说明「为什么」
- **本案例做法（弱在哪）**：9 个薄委托 skill 只有 `@workflow` 引用，没有示例，Claude 执行时靠猜
- **意义**：bureau 的 command 详细度是标准，在 NLPM review 中可以展示这一点

---

## 八、术语表

### <a name="blackboard-pattern"></a>黑板架构（Blackboard Pattern）
> 一种多 agent 协作设计模式：所有 agent 通过读写一个共享「黑板」（这里是 `.planning/*.md` 文件）来协调工作，而非直接调用对方。优点是解耦；缺点是「黑板」内容格式如果没有契约，容易出现不同 agent 写的格式不一致。

### <a name="coexistence-election"></a>共存竞选（Coexistence Election）
> GSD 插件独创的机制。当用户同时安装了两个版本的插件（升级时常见），hooks.json 会被两个版本各自注册，导致同一事件触发两次 hook。竞选机制让两个版本的 hook 「比赛」，只有一个赢得执行权（通过检查共享锁文件或版本号），另一个主动退出（exit 0）。确保状态文件不被双重写入。

### <a name="thin-delegation"></a>薄委托（Thin Delegation）
> 指一个 skill 的主体只有「调用另一个 workflow/skill」的引用，没有自己的执行逻辑、示例或输出格式。NLPM 把这类 skill 视为质量缺陷，因为 Claude 收到这种 skill 时，不知道任务的输出格式是什么，无法验证是否完成。

### <a name="success-criteria"></a>成功标准（Success Criteria）
> 在 skill/command 末尾明确列出的「这个任务算完成的可验证条件」。如「测试通过率 > 95%」、「输出文件存在且不为空」。有成功标准的 skill，Claude 可以自我检查并报告「已完成 / 未完成」。
