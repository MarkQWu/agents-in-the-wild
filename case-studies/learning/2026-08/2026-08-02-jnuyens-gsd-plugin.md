# jnuyens/gsd-plugin — 学习案例

**仓库**：https://github.com/jnuyens/gsd-plugin
**Stars**：9 | **来源**：upstream
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-08-02（基于当前 HEAD）
**主题标签**：`model-pinning`, `experience-accumulation`, `examples-driven`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
jnuyens/gsd-plugin 是一个实现完整 GSD（Get Shit Done）项目管理工作流的 Claude Code 插件，覆盖「计划 → 研究 → 执行 → 验证 → 发布」全生命周期。当前版本为 v4.5.2（插件版本线，今日 2026-08-02 发布）/ v2.45.0（core 版本线），拥有 33 个 agent、90 个 skill，合计 123 个 NL 工件。Stars 仅 9，但工件数量和设计复杂度已经超过多数大众插件。

关键事实：
- 存在两套并行版本号：`plugin.json` 中的 `2.45.0` 跟踪 gsd-core 版本线；CHANGELOG 从 v4.0.0 起单独记录插件版本（4.5.2）。两者并存，不是错误，是有意的版本策略分离
- 从历史快照（v2.38.8，82 个 skill）到今日（v4.5.2，90 个 skill），8 个月净增 8 个 skill，agent 数量维持 33 不变，说明作者持续扩充功能面而非重组架构
- 通过 [PostToolUse hook](#PostToolUse-hook) 监听几乎所有工具调用，实现了跨工具调用的状态追踪（每次文件读写后都触发 GSD 状态更新）
- v4.5.1（昨日）新增 PostToolUse envelope sanitizer，修复 Write 工具产生的 `</content>` / `</invoke>` XML 标签污染 `.planning/*.md` 状态文件的 bug——说明作者在积极应对 LLM 工具调用副作用

### 1.2 架构剖析

**目录结构**（关键部分）：
```
gsd-plugin/
  agents/                              # 33 个 agent（全部缺 model 字段）
    gsd-planner.md                     # 核心规划 agent（1249 行，89/100）
    gsd-debugger.md                    # 调试 agent（1453 行，93/100）
    gsd-project-researcher.md          # 并行研究编排 agent（93/100）
    gsd-research-synthesizer.md        # 研究合成 agent（86/100）
    gsd-debug-session-manager.md       # 调试会话管理 agent（93/100）
    ...（28 个其他 agent）
  skills/                              # 90 个 skill（历史快照时 82 个）
    debug/SKILL.md                     # 100/100，多子命令，格式范例
    research-phase/SKILL.md            # 100/100，agent 派发模板
    reapply-patches/SKILL.md           # 100/100，7 步骤 + Hunk 验证表
    ...（9 个得分 75 的薄委托 skill）
  hooks/
    hooks.json                         # 4 个事件处理器（SessionStart / PreToolUse / PostToolUse / PreCompact）
  package.json                         # devDependencies: ajv + ajv-formats（仅用于 schema 验证）
  .mcp.json                            # {"mcpServers": {}}（空配置）
```

**文件类型分布**：33 个 agent / 90 个 skill / 1 个 hooks.json / 1 个 package.json / 1 个空 .mcp.json

**编排关系**：GSD 采用两条主要调用链。第一条是「并行研究管道」：`gsd:research-phase` skill 触发 `gsd-project-researcher` agent，该 agent 最多并行派发 6 个子 agent（SUMMARY、STACK、FEATURES、ARCHITECTURE、PITFALLS、COMPARISON），产出文件后交由 `gsd-research-synthesizer` 合成。第二条是「调试闭环」：`gsd:debug` skill 触发 `gsd-debug-session-manager`，后者驱动 `gsd-debugger` 执行科学方法（假设 → 验证 → 迭代）。所有 skill 通过 `@${CLAUDE_PLUGIN_ROOT}/...` 变量引用 workflow 模板，而非硬编码路径。

**跨件契约**：使用 `@${CLAUDE_PLUGIN_ROOT}` [运行时变量](#运行时变量)作为跨文件引用的命名空间。这意味着所有 agent 和 skill 在插件安装时能正确解析彼此，但在裸 `git clone` 环境中，变量不会展开，所有 `@${CLAUDE_PLUGIN_ROOT}/...` 引用会显示为未解析的变量名——这是设计预期，不是 bug。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「全生命周期钩子驱动状态机」——GSD 不只是一组命令，而是通过 4 个 [hook](#hook) 事件（SessionStart / PreToolUse / PostToolUse / PreCompact）在每个 Claude 工具调用前后维护 `.planning/` 目录里的项目状态文件，实现会话间的连续性。

**解决什么问题**：大型项目在多次 Claude 会话中容易丢失上下文。GSD 通过在每次工具调用后自动写入 `.planning/` 状态文件，让新会话可以从 PreCompact 保存的上下文快照恢复，不需要人工重新描述项目状态。

**做了什么 trade-off**：钩子覆盖面 vs 执行性能。当前 PostToolUse 的 matcher 包含只读工具（Read、Grep、Glob），每次文件读取都会触发钩子并等待最多 3 秒——这换来了精确的状态追踪，代价是文件密集型操作阶段（代码审查、研究、规划）的明显延迟。作者选择了「追踪完整」而非「性能最优」。

**反映什么认知模型**：作者把 Claude Code 会话视为需要「外部状态机」补充的系统——LLM 的上下文窗口是暂时的，只有写入磁盘的 `.planning/` 文件才是持久的。钩子是这个状态机的触发器，不只是副作用处理器。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「全覆盖钩子 + 文件状态机 + 变量化跨引用」三支柱架构**：插件不只提供命令和 agent，还通过 4 类 hook 事件把项目状态写入本地文件系统，再用运行时变量（`@${CLAUDE_PLUGIN_ROOT}`）把所有 NL 工件连接为一个整体。

模式特征清单：
- 特征 1：4 类 hook 事件覆盖会话全周期（开始 / 写文件前 / 写文件后 / 压缩前）
- 特征 2：`.planning/` 目录是单一状态真相来源，而非依赖 LLM 的对话上下文
- 特征 3：所有跨文件引用通过 `@${CLAUDE_PLUGIN_ROOT}` 运行时变量解耦，不用硬编码路径
- 特征 4：并行 agent 派发（最多 6 个并行子 agent）用于研究密集型阶段
- 特征 5：两套版本号并行（插件版本 vs core 版本），通过 CHANGELOG 显式说明分歧点

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨多次会话的长周期项目（超过 1 天的开发任务） | ✅ 高度适用 | hook 状态机保证会话间连续性，PreCompact 防止上下文压缩丢失关键状态 |
| 单次会话、一次性任务（代码审查、生成文档） | ❌ 不适用 | 四个 hook 引入额外延迟，状态机是不必要的复杂度 |
| 需要明确工作流阶段的团队项目（有计划 → 执行 → 验证阶段） | ✅ 适用 | GSD 的阶段文件（PLAN.md、ROADMAP.md、STATE.md）为团队成员提供共同上下文 |
| 单人快速原型（希望零配置即用） | ❌ 不适用 | `.planning/` 初始化、阶段文件管理需要额外学习成本 |
| 高文件读取频率任务（大型 codebase 扫描） | ⚠️ 慎用 | PostToolUse 触发 Read/Grep/Glob 会带来每次 3 秒的钩子等待开销 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 全覆盖钩子 + 文件状态机（本仓库） | jnuyens/gsd-plugin | 跨会话完整连续性；状态不依赖 LLM 记忆 | 只读工具触发钩子，读文件有延迟；123 个工件学习曲线陡 |
| MCP 持久化 + 少量 agent | eyaltoledano/claude-task-master | 状态存在 MCP 服务器，可跨用户共享 | 依赖外部 MCP 进程；任务状态与文件系统割裂 |
| 纯文件 + 无 hook | MarkQWu/gstack（推测） | 零配置，可移植 | 跨会话需手动载入上下文；无自动状态更新 |

### 2.4 改进空间

1. **当前问题**：PostToolUse matcher 包含 Read/Grep/Glob，每次文件读写都触发 3 秒 hook 等待 **改进做法**：把 matcher 改为 `"Bash|Edit|Write|MultiEdit|NotebookEdit"`，去掉三个只读工具 **预期收益**：消除文件密集型阶段（规划、研究、代码审查）的无效 hook 调用，减少用户等待时间

2. **当前问题**：33 个 agent 全部缺 `model:` 字段，全部跑 Sonnet 默认 **改进做法**：按任务复杂度分层：`gsd-doc-classifier` → `model: haiku`（枚举分类），`gsd-debugger` → `model: opus`（1453 行科学调试），其余 → `model: sonnet` **预期收益**：agent 平均分从 88.7 提升至约 94；实际运行成本可以降低（haiku 分类任务成本约为 sonnet 的 10%）

3. **当前问题**：并行研究管道中 gsd-project-researcher（生产者）和 gsd-research-synthesizer（消费者）之间没有显式的 schema 契约——消费者靠读 section header 名称来匹配生产者输出 **改进做法**：在共享 reference 文件里定义 6 个 section 的规范名称和缺省值规则，两个 agent 都引用此文件 **预期收益**：如果 researcher 漏写某个 section，synthesizer 能显式报错而不是静默降级

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-28 当时得分 **90.3/100**（v2.38.8，33 agent + 82 skill = 115 工件）。

| 分类 | 平均分 | 说明 |
|---|---|---|
| Agents（33 个） | 88.7/100 | 全部缺 model 字段（−5 × 33），3 个 agent 零示例（−15 × 3） |
| Skills（82 个） | 90.9/100 | 9 个薄委托 skill 得 75，其余多数 85–100 |
| 总体 | 90.3/100 | 无任何工件低于 70 阈值 |

当时最低分 agent 样本：

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/gsd-advisor-researcher.md | 76 | 无 model（−5），零示例（−15），"focused" "genuine"（−4） |
| agents/gsd-assumptions-analyzer.md | 78 | 无 model（−5），零示例（−15），"relevant"（−2） |
| agents/gsd-eval-auditor.md | 80 | 无 model（−5），零示例（−15） |
| agents/gsd-nyquist-auditor.md | 95 | 仅无 model（−5） |
| agents/gsd-roadmapper.md | 95 | 仅无 model（−5） |

当时薄委托 skill 样本（全部 75/100）：

| Skill | 缺少的内容 |
|---|---|
| skills/audit-uat/SKILL.md | process 步骤、示例 |
| skills/cleanup/SKILL.md | 示例、output 格式 |
| skills/do/SKILL.md | 路由示例、output 格式 |
| skills/eval-review/SKILL.md | 示例、output 格式 |
| skills/extract_learnings/SKILL.md | 示例、output 格式 |
| skills/fast/SKILL.md | 示例、output 格式 |
| skills/health/SKILL.md | 示例、output 格式 |
| skills/import/SKILL.md | 示例、output 格式 |
| skills/insert-phase/SKILL.md | 示例、output 格式 |

### 3.2 当时值得借鉴的模式

1. **运行时变量化跨引用** → 所有 agent/skill 用 `@${CLAUDE_PLUGIN_ROOT}/workflows/...` 引用内部 workflow，Claude Code 在加载时解析变量 → 根本原因：路径不硬编码，插件可以在任何安装位置工作，不会因用户的 home 目录不同而断引用 → `agents/gsd-planner.md` 中大量 `@${CLAUDE_PLUGIN_ROOT}` 实例 → 借鉴：我的插件里的跨文件引用改用运行时变量，而非 `../../` 相对路径

2. **满分 skill 的结构模式**（如 `skills/reapply-patches/SKILL.md`）→ 7 步骤过程 + Hunk 验证表 + 多条具体规则 → 根本原因：把「怎么做」拆成有序步骤、把「验证标准」写成可机械执行的表格，比「聪明地处理 patch」要强 10 倍 → `skills/research-phase/SKILL.md`、`skills/resume-at/SKILL.md`、`skills/thread/SKILL.md` 等 → 借鉴：我的 skill 里凡是有「验证」步骤的，改写成表格形式

3. **PreCompact hook 保存上下文** → 在 Claude Code 压缩对话之前，hook 把关键状态写入 `.planning/` 文件，下次会话恢复时无需重新描述项目背景 → 根本原因：LLM 的上下文窗口是暂时的，只有文件系统是持久的；hook 在「上下文即将消失」的窗口期写入状态是最佳时机 → `hooks/hooks.json` PreCompact handler → 借鉴：任何需要跨会话记忆的插件都应该加 PreCompact hook

4. **debug chain 的科学方法论**（`gsd-debugger.md`，93/100，1453 行）→ 假设 → 可验证测试 → 结果 → 迭代，有明确的多轮循环退出条件 → 根本原因：调试任务的失败往往是因为 agent 没有退出条件；明确的科学方法 + 状态文件让多轮迭代可控 → `agents/gsd-debugger.md` → 借鉴：我的 debugger-like agent 应该要求「先写假设再测试」，而不是直接修代码

5. **分层评分意识**（`skills/set-profile/SKILL.md` 有 `model: haiku`）→ 插件里唯一声明 model 的工件，而且选的是 haiku（正确：用户画像生成是枚举分析任务）→ 根本原因：作者知道某些任务不需要高模型，但只给一个 skill 加了字段，没有推广到 agent → 借鉴：这证明作者意识到分层，只是没有一致化执行；启示我从最明显的案例（枚举分类 agent）开始补 model 字段

### 3.3 当时的缺陷

1. **全部 33 个 agent 缺 `model:` 字段（Q-01）** → 不同 agent 的任务复杂度差异巨大：`gsd-doc-classifier`（90 行，枚举分类）和 `gsd-debugger`（1453 行，科学迭代调试）使用同一个默认模型（Sonnet）→ 根本原因：插件的关注点在「workflow 完整性」而非「资源优化」；model 声明是 NL 规范的基础字段，但在功能导向的开发阶段容易被推迟 → **自查：我的 gstack 和 bureau 的 agent 是否有 `model:` 字段？数据显示都没有。我犯了同样的错误。**

2. **3 个 agent 完全没有示例块（Q-02）** → `gsd-advisor-researcher` 描述了 5 列对比表但没给任何样本行；`gsd-assumptions-analyzer` 描述了 3 个校准层级但没有输出样本；`gsd-eval-auditor` 描述了评分维度但没有评分示例 → 根本原因：这三个 agent 的输出格式比较复杂（表格行、结构化标定结果），在快速开发时往往「知道要什么、但懒得写样本」→ **自查：bureau 的 auditor.md 和 gstack 的部分 agent 也缺少示例，情况类似。**

3. **20+ agent 使用模糊量词（Q-03）** → "appropriate"、"relevant"、"intelligent"、"reasonable"、"genuine" 等词出现在指令里，但都没有可操作定义。比如「provide genuine recommendations」——AI 无法判断什么是「genuine」，只能猜测 → 根本原因：这些词在人类语言里有意义，但对 LLM 而言缺少可测量的执行标准；作者用人类写给人类的语气写了 AI 指令 → **自查：我的 skill 里也有 "comprehensive"、"appropriate" 等词，需要批量替换。**

4. **9 个薄委托 skill 缺 process 和示例（Q-04）** → 这 9 个 skill 都是「目标 + @workflow 引用」两行，没有步骤说明，没有输出格式，没有示例 → 根本原因：这些 skill 的角色是「入口跳板」而非「独立文档」，作者认为读 workflow 文件就够了；但对于不熟悉 GSD 的用户或 AI，跳板式 skill 没有任何可执行信息 → **自查：bureau 有几个 skill 也只有 2 行描述；这是同类问题。**

### 3.4 当时的优化机会

1. **最高杠杆**：给 33 个 agent 逐一加 `model:` 字段（haiku/sonnet/opus 按任务复杂度分层）——单次修改每文件 1 行，预计可将 agent 平均分从 88.7 提升至约 94，同时降低实际 API 调用成本

2. **次优先**：给 9 个薄委托 skill 各加一条 `<output_format>` 说明（产出什么文件、格式是什么）和一条使用示例——每个 skill 约 10 分钟，总计 90 分钟可完成，预计将这些 skill 从 75 提升至 85+

3. **安全改进**：将 PostToolUse matcher 从 `Bash|Edit|Write|MultiEdit|NotebookEdit|Read|Grep|Glob|WebFetch|WebSearch` 缩减为 `Bash|Edit|Write|MultiEdit|NotebookEdit`，去掉三个只读工具，消除文件密集操作的无效等待

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Q-01：0/33 agent 声明 model 字段 | `grep -rn "^model:" agents/` | **仍存在**（grep 返回空，0 命中） | 历史快照后 3 个月，最高杠杆的修复仍未执行；作者发版频繁（今日 v4.5.2）但没有关注 NL 规范问题 |
| Q-02：gsd-advisor-researcher 零示例 | `grep -n "example\|Example\|structured_returns" agents/gsd-advisor-researcher.md` | **仍存在**（grep 返回空） | 该文件得分仍为 76，没有改善 |
| Q-03：PostToolUse matcher 包含只读工具 | hooks/hooks.json PostToolUse 段的 matcher 字段 | **仍存在**（`"matcher": "Bash|Edit|Write|MultiEdit|NotebookEdit|Read|Grep|Glob|WebFetch|WebSearch"`） | MEDIUM 安全发现未修复，每次文件读取仍有额外等待 |

### 4.2 架构演进

从 v2.38.8（82 skill，115 工件）到 v4.5.2（90 skill，123 工件），8 个月的变化：

- **新增 8 个 skill**：功能面持续扩展，说明作者在回应用户新增的工作流需求
- **agent 数量维持 33**：核心角色分工已稳定，不再新增 agent 类型；说明作者认为 33 个角色已能覆盖 GSD 全生命周期
- **两套版本号正式分离**：从 v4.0.0 起，plugin 版本与 core 版本解耦。CHANGELOG 明确说明了这个决定——这是成熟度的标志，不再把「插件发版」绑定在「core 更新」上
- **今日（v4.5.2）新增 envelope sanitizer**：针对 Write 工具在大型写入后泄露 `</content>` / `</invoke>` XML 标签的 bug 增加了清理步骤。这说明作者在 v4.5 开始关注 LLM 工具调用副作用这个新问题域，而不只是功能迭代
- **v4.5.0 token-lean 输出**：minified JSON 输出使 token 消耗减少 15–20%，说明作者从「功能完整」转向「资源效率」的关注点演进

总体判断：作者的演进方向是「功能扩展 + 运行时健壮性」，但 NL 规范（model 字段、示例块、量词精确化）不在其关注焦点上，三个已知缺陷在 3 个月内均无修复迹象。

### 4.3 新增的可学习模式

**PostToolUse Envelope Sanitizer 模式**（v4.5.1 新增）：在 PostToolUse hook 里检测并清除写入状态文件后残留的 XML 标签污染。这是一个新的健壮性模式——当 LLM 在 Write 工具输出末尾泄露 XML 控制标签时，下一次读取状态文件的 agent 会读到损坏数据；sanitizer 在每次写操作后自动清理，把「LLM 输出不完全可信」的假设内化到工具链里。

值得借鉴：任何依赖 LLM 写入文件作为状态存储的系统，都应该在读取前或写入后加一个格式校验/清理步骤，而不是假设 LLM 的输出永远干净。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau 的部分 skill 有 Examples 章节**：gsd-plugin 最低分的 3 个 agent 完全没有示例；bureau 至少在一些 skill 里有具体的 input/output 示例——这是正向对比
2. **没有把危险 shell 命令嵌入 NL 工件**：gsd-plugin 的 hooks.json 里有 inline 的 700 字符 minified Node.js；我的项目使用外部脚本引用，不在 hook 里嵌入内联代码
3. **状态文件路径有明确命名约定**：gsd-plugin 的 debug 状态文件 `.planning/debug/{slug}.md` 的 schema 只在 SKILL.md 里内联定义，没有共享 reference；我的 bureau 把关键文件路径在 CLAUDE.md 里集中声明
4. **版本号语义明确**：我的仓库遵循单一 semver，没有两套并行版本造成阅读混乱

### 5.2 挑战 / 验证

**挑战**：我原本认为「`@${CLAUDE_PLUGIN_ROOT}` 变量引用」只是 Claude Code 的一个内部实现细节，不值得特别关注。这次案例让我意识到，这个模式是插件级跨文件引用的关键设计决策：使用运行时变量 vs 相对路径的区别，决定了插件能否在任意安装路径下工作。gstack 如果使用 `../../` 相对路径引用，换个安装位置就会断引用。

**验证**：gsd-plugin 的 `skills/set-profile/SKILL.md` 是仓库里唯一声明 `model: haiku` 的工件——这个孤例验证了我一直认为的「对简单分类任务应当用 haiku」的判断。这个孤例同时也说明：作者知道这个最佳实践，只是没有推广到其他 33 个 agent。这是「意识到但没有执行」的典型案例，不是「不知道」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否有 model 字段
grep -rn "^model:" ~/.claude/plugins/*/agents/*.md 2>/dev/null | head -20
```
没有任何输出 = 我和 gsd-plugin 一样，全部缺 model 字段。命中后：按照任务复杂度（枚举/分类 → haiku；规划/分析 → sonnet；复杂推理/调试 → opus）逐文件补全。

```bash
# 检查我的 skill 是否有模糊量词
grep -rn -E '\b(appropriate|comprehensive|robust|intelligent|reasonable|properly|genuine|focused|relevant)\b' \
  ~/.claude/plugins/*/skills/*/SKILL.md 2>/dev/null | head -30
```
命中后：把每处量词替换为可验证的具体约束。例如 "appropriate error handling" → "cover network failures, null inputs, and non-2xx status codes"。

```bash
# 检查我的 skill 是否缺少 Examples 章节
grep -rL "## Example\|## 示例\|<example\|structured_returns" \
  ~/.claude/plugins/*/skills/*/SKILL.md 2>/dev/null
```
命中后：至少加一条 input/output 对。如果 skill 产出结构化文件，把文件的前 20 行作为示例贴到 skill 里。

```bash
# 检查 hooks 是否包含只读工具触发
grep -n "Read\|Grep\|Glob" ~/.claude/plugins/*/hooks/hooks.json 2>/dev/null
```
命中后：评估是否真的需要在只读工具调用后更新状态；如果不需要，从 matcher 里删除 Read/Grep/Glob，减少无效等待。

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 里加一个 PreCompact hook，在对话压缩前把当前工作状态写入 `.bureau/session-state.md`
   **为何可行**：bureau 已经有文件持久化机制（知识捕获写入文件系统）；只需新增一个 PreCompact 事件处理器，把当前 session 的摘要写入一个固定路径的状态文件
   **第一步**：参考 gsd-plugin 的 `hooks/hooks.json` PreCompact handler，在 bureau 的 hooks.json 里加一条 `"event": "PreCompact"` 处理器（30 分钟）

2. **想法**：把 gsd-plugin 的 `@${CLAUDE_PLUGIN_ROOT}` 变量引用模式引入 gstack，替换现有的相对路径引用
   **为何可行**：gstack 已经是以 Claude Code 插件形式安装的；`CLAUDE_PLUGIN_ROOT` 变量在插件运行时由 Claude Code 注入，直接可用
   **第一步**：找到 gstack 里所有 `../../` 或硬编码路径的引用，用 `grep -rn "\.\.\/" ~/.claude/plugins/gstack/` 列出来，然后逐个改为 `@${CLAUDE_PLUGIN_ROOT}/...`（约 1 小时）

3. **想法**：给 gstack 和 bureau 的所有 agent 批量加 `model:` 字段
   **为何可行**：agent 的任务类型是固定的，可以一次性评估完；添加一行 frontmatter 不影响任何功能
   **第一步**：列出所有 agent 文件 `ls ~/.claude/plugins/*/agents/*.md`，逐一按任务类型打 haiku/sonnet/opus 标签，写一个 for 循环用 `sed` 在 `---` frontmatter 结束标记前插入 `model: xxx`（45 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 jnuyens/gsd-plugin 的核心目的**：通过钩子驱动的状态机和 33 个专职 agent 实现项目全生命周期管理（计划 → 研究 → 执行 → 验证 → 发布）
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是 Claude Code workflow 插件；都有多 agent 分工；都有跨会话状态管理需求 | gsd-plugin 用 hook 状态机，gstack（推测）用文件写入但无 hook 联动；规模差异大（gstack 59 工件 vs gsd-plugin 123 工件） | 高（hook 架构、model 字段、变量引用） |
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有 hook 处理 | bureau 定位知识管理（知识捕获/检索），gsd-plugin 定位项目执行（任务驱动开发） | 中（PreCompact hook 模式、薄 skill 改进） |
| MarkQWu/- | 低 | 都用 NL 工件 | 该仓库是 intelligence dashboard，无工作流管理概念 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 全部 agent 缺 `model:` 字段 | `grep -rL "^model:" gstack/agents/ bureau/agents/` | gstack 和 bureau 全部命中（0 个 agent 有 model 字段） | 高 |
| 部分 agent 零示例块 | `grep -rL "## Example\|structured_returns\|<example" bureau/agents/` | bureau 的 auditor.md 命中 | 中 |
| 薄委托 skill 缺 process 和 output 格式 | `wc -l bureau/skills/*/SKILL.md \| sort -n \| head -5` | bureau 有 2–3 个不足 15 行的 skill | 中 |
| 模糊量词 | `grep -rn "appropriate\|comprehensive\|robust" gstack/agents/ bureau/skills/` | 估计 gstack 命中 5–10 处 | 中 |

**命中后的具体行动建议**：
- `gstack/agents/*.md` → 列出所有 agent，按枚举分类/规划/复杂推理三档，批量在 frontmatter 加 `model: haiku/sonnet/opus`（45 分钟完成）
- `bureau/agents/auditor.md` → 加一个 `## Examples` 章节，贴一个实际的 audit 输出样本（10 分钟）
- `bureau/skills/` 中最短的几个 skill → 各加 3 步 process 说明 + 一条使用示例（每个约 10 分钟）
- gstack 中出现 "appropriate"、"comprehensive" 的指令行 → 改为可验证约束（每处 5 分钟）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：运行时变量化跨文件引用
   - **本案例做法**：所有 agent/skill 用 `@${CLAUDE_PLUGIN_ROOT}/workflows/xxx.md` 引用内部文件，不用相对路径（`agents/gsd-planner.md` 中大量实例）
   - **我的项目现状**：gstack 推测使用相对路径或绝对路径（需验证），如果换安装目录会断引用
   - **如何借鉴**：在 gstack 里找出所有文件间引用（`grep -rn "\.\.\/" gstack/`），改为 `@${CLAUDE_PLUGIN_ROOT}/` 前缀；同时在插件文档里说明这个引用风格

2. **领域**：PreCompact hook 保存会话状态
   - **本案例做法**：`hooks/hooks.json` 有 PreCompact 事件处理器，在 Claude Code 压缩对话前把状态写入 `.planning/` 文件
   - **我的项目现状**：bureau 有知识捕获机制，但没有 PreCompact hook；如果对话上下文压缩发生，当前捕获状态可能丢失
   - **如何借鉴**：给 bureau 加 PreCompact hook，在压缩前把当前 session 的知识捕获摘要写入 `.bureau/session-state.md`（参考 gsd-plugin hooks.json 的 PreCompact handler 结构）

3. **领域**：满分 skill 的可执行结构（7 步骤 + 验证表）
   - **本案例做法**：`skills/reapply-patches/SKILL.md`（100/100）用 Hunk Verification Table 把「如何验证 patch 是否正确应用」变成机械可检查的表格，而不是「smartly verify」
   - **我的项目现状**：bureau 和 gstack 的 skill 普遍缺乏机械可执行的验证步骤，倾向于用「检查是否正确」的说法
   - **如何借鉴**：把每个 skill 里的「验证」步骤改成「Verification Checklist」或「成功标准」表格，每条标准都是 PASS/FAIL 可判断的条件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：示例章节覆盖率
- **我的做法**：bureau 的部分 skill 有 `## Examples` 章节，提供了 input/output 对
- **本案例做法（弱在哪）**：gsd-plugin 最低分的 3 个 agent（gsd-advisor-researcher 76 分、gsd-assumptions-analyzer 78 分、gsd-eval-auditor 80 分）完全没有示例，这是三个 −15 的直接扣分来源，且 3 个月后仍未改善
- **意义**：bureau 在「至少有部分示例」这一点上比 gsd-plugin 的问题 agent 做得好；若在未来接受外部 audit，这是一个可以被识别为亮点的地方。同时这也提醒我：只有「部分」skill 有示例还不够，需要覆盖到所有 agent 和核心 skill

---

## 八、术语表

### <a name="hook"></a>hook
> Claude Code 的钩子机制。在特定事件发生时（如会话开始、工具调用前、工具调用后、上下文压缩前），Claude Code 会自动执行你在 `hooks.json` 里注册的命令。类比：就像银行的「刷卡前确认 → 刷卡 → 记账」三步流程，每一步都可以挂一个「旁听者」做额外操作。

### <a name="PostToolUse-hook"></a>PostToolUse hook
> 一种特定的 Claude Code 钩子，在 Claude 调用任何工具（如 Read、Write、Bash）**完成之后**自动触发。gsd-plugin 用它来更新项目状态：每当 Claude 读了一个文件或执行了一段命令，PostToolUse hook 就把当前状态写入 `.planning/` 目录。问题在于，就算只是「读文件」（Read/Grep/Glob）也会触发这个钩子，产生不必要的等待。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置，用来声明文件的元数据。在 Claude Code 的 agent 文件里，frontmatter 通常包含 `name`（agent 名称）、`description`（用途说明）、`model`（使用哪个模型：haiku / sonnet / opus）、`tools`（允许调用哪些工具）等字段。缺少 `model` 字段意味着 agent 每次都用默认模型运行，无法按任务复杂度节省成本。

### <a name="运行时变量"></a>运行时变量
> 在 Claude Code 加载插件时才被替换为实际值的占位符。`@${CLAUDE_PLUGIN_ROOT}` 就是一个运行时变量，它在插件安装时由 Claude Code 替换为该插件的实际安装路径（如 `~/.claude/plugins/gsd-plugin`）。使用运行时变量而非硬编码路径，可以让插件在不同用户的不同安装路径下都能正确工作。

### <a name="薄委托skill"></a>薄委托 skill（thin-delegation skill）
> 一种只包含「目标描述 + 指向 workflow 文件的引用」的 skill，缺少具体步骤、输出格式和使用示例。gsd-plugin 里有 9 个这样的 skill 得分 75/100。「薄」不是指文件短，而是指「给 AI 的指导太薄」：只说「去执行 workflow X」，但不说产出什么、如何验证、遇到边界情况怎么办。

### <a name="envelope-sanitizer"></a>envelope sanitizer
> gsd-plugin v4.5.1 新增的清理机制。当 Claude 用 Write 工具写入大型文件时，有时会在文件末尾多写入 XML 控制标签（如 `</content>` 或 `</invoke>`）——这是 LLM 工具调用的一个已知副作用。sanitizer 在每次 Write 完成后自动扫描并删除这些多余标签，防止后续读取该文件的 agent 收到损坏数据。

### <a name="semver"></a>semver（语义化版本）
> 格式为 `主版本.次版本.补丁版本`（如 `2.45.0`）的版本号规范。主版本号变化 = 有破坏性变更；次版本号变化 = 新增功能但向后兼容；补丁版本号变化 = bug 修复。gsd-plugin 有两套 semver：`4.5.2`（插件版本线，从 v4.0.0 起独立）和 `2.45.0`（跟踪 gsd-core 版本线），两者在 CHANGELOG 里均有记录。
