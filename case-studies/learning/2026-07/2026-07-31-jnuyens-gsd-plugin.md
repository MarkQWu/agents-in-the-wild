# jnuyens/gsd-plugin — 学习案例

**仓库**：https://github.com/jnuyens/gsd-plugin
**Stars**：9 | **来源**：upstream audit
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-07-31（基于当前 HEAD）
**主题标签**：`model-pinning`, `vague-quantifier`, `examples-driven`, `single-purpose`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
jnuyens/gsd-plugin 是一个功能密度极高的 Claude Code 插件：v2.38.8，包含 **33 个 agents + 82 个 skills**，实现完整的项目管理工作流（GSD = Get Stuff Done）。从规划到执行、从代码审查到文档生成、从调试到安全审计，涵盖了开发工作的全生命周期。9 Stars 远低于其实际质量（90.3/100），属于「被低估的高质量插件」。

关键事实：
- 由 jnuyens 个人维护，面向个人开发者的 Claude Code 生产力提升
- 使用 hooks 系统对工作流状态进行持久化追踪（sessions、phases、milestones 的状态机）
- CHANGELOG 极其详细，v2.38.8 说明这是高度迭代演进的产品
- Security: CLEAR（无高危发现，仅有 3 个低/中等 hook 相关问题）

### 1.2 架构剖析

```
jnuyens/gsd-plugin/
├── agents/                      # 33 个专职 agents（每个聚焦单一工作流节点）
│   ├── gsd-planner.md           # 计划制定（1249 行，最复杂的 agent）
│   ├── gsd-debugger.md          # 科学调试（丰富示例，93/100）
│   ├── gsd-debug-session-manager.md  # 调试状态管理（读.planning/debug/*.md）
│   └── ... （共 33 个）
├── skills/                      # 82 个操作技能（覆盖所有 GSD 工作流命令）
│   ├── debug/SKILL.md           # 100/100，多子命令 + 安全注意事项
│   ├── reapply-patches/SKILL.md # 100/100，7步骤 + Hunk Verification Table
│   ├── thread/SKILL.md          # 100/100，五种模式 + 安全说明
│   └── ... （共 82 个）
├── hooks/
│   └── hooks.json               # 4 类 hooks（SessionStart/PreToolUse/PostToolUse/PreCompact）
├── bin/
│   └── gsd-tools.cjs            # Hook 运行时核心（CJS 模块）
└── CLAUDE.md                    # 项目上下文
```

- **文件类型分布**：33 个 agent + 82 个 skill + 0 个 command（skills 兼任命令角色）
- **编排关系**：skills 调度 agents；agents 之间有隐式链式关系（如 debug 链：skill → session-manager → debugger）；hooks 追踪所有工具调用
- **跨件契约**：agents 通过文件路径（`.planning/debug/*.md`）共享状态，状态模式内联在 skills 中而非独立 schema 文件

### 1.3 设计思路 / 方法论

- **核心哲学**：「有状态的工作流」——大多数 GSD 操作会读写 `.planning/` 下的 YAML/Markdown 状态文件，跨 session 持久化
- **解决的问题**：Claude 会话无状态的根本局限——每次新建会话都忘记上次到哪了、哪些任务完成了、什么是下一步
- **Trade-off**：状态持久化带来了复杂性（状态机、debug session 链、phase 状态等），但换来了跨 session 的记忆能力
- **认知模型**：作者把 Claude 视为「需要外部记忆的执行引擎」，而 GSD 插件提供了那个外部记忆的操作界面

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「有状态工作流 + 全覆盖 hook 监控」**模式：每个工作流操作都会更新外部状态文件；hooks 监控每一次工具调用，维护工作流状态机的一致性。

模式特征：
- 有一个持久化状态目录（此处 `.planning/`）存储所有会话状态
- Hooks 对所有状态变更工具调用进行监控
- Agents 之间通过文件路径（而不是内存变量）传递状态
- Skills 是入口，agents 是执行单元

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨多会话的长期项目管理 | ✅ 高度适用 | 状态持久化正是为此设计 |
| 单会话完成的一次性任务 | ❌ 不适用 | 状态机开销超过收益 |
| 需要人类审查每步操作的工作流 | ✅ 适用 | hooks 提供了细粒度拦截点 |
| 无状态的知识问答 | ❌ 不适用 | 完全不需要状态持久化 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 有状态工作流（本案） | jnuyens/gsd-plugin | 跨 session 记忆；工作流一致性 | 复杂度高；状态 schema 需同步 |
| 无状态技能库 | addyosmani-agent-skills | 简单轻量；无副作用 | 无记忆；每次重新开始 |
| 侧车记忆文件（简化版） | BayramAnnakov/claude-reflect | 中等复杂度 | 状态结构不如 GSD 系统化 |

### 2.4 改进空间

1. **当前问题**：33 个 agents 全部无 `model:` 声明，默认使用 Sonnet。但 `gsd-doc-classifier`（90 行，简单分类）应该用 Haiku，`gsd-debugger`（科学调试）应该保持 Sonnet。**改进做法**：按复杂度分三档：Haiku（分类/验证类）、Sonnet（计划/调试类）、`model` 不变（默认即 Sonnet 的复杂任务）。**预期收益**：Haiku agents 调用成本降低约 10 倍。

2. **当前问题**：debug 链的状态 schema（`.planning/debug/{slug}.md`）内联在 `debug/SKILL.md` 中，不在共享引用文件。**改进做法**：创建 `references/debug-state-schema.md`，让 session-manager 和 debugger 两个 agents 都指向它。**预期收益**：schema 更新时只需改一个文件，消除双向漂移风险。

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-28 当时得分 **90.3/100**，SECURITY **CLEAR**。

| 类别 | 当时平均 | 主要问题 |
|---|---|---|
| Agents（33个）| 88.7 | 0/33 声明 model；3 个零示例（-15×3）；20+ 个模糊量词 |
| Skills（82个）| 90.9 | 9 个瘦委托 skills 75/100；20+ 个模糊量词 |

### 3.2 当时值得借鉴的模式

1. **多子命令 skill（debug/thread/intel/quick）** → 每个 skill 在同一个 SKILL.md 中提供多个子命令（`gsd debug fix`、`gsd debug session-list` 等），避免了大量零散的单功能 skills。代码示例、安全说明、模式文档一体化。

2. **reapply-patches/SKILL.md 的「Hunk Verification Table」** → 100/100 的示例——提供了一个具体的表格格式来追踪每个 patch 的应用状态，比「执行成功/失败」更细粒度。

3. **gsd-debug-session-manager 的调度表** → 用一个明确的 dispatch table（枚举所有状态转换）来管理调试 session 的状态机，而不是用 if/else prose。代码清晰度极高。

4. **PRIVACY.md + SECURITY.md + RELEASING.md** → 完整的维护者文档三件套，体现了工程成熟度。

5. **set-profile/SKILL.md 的 `model: haiku`** → 这是整个 82 个 skills 中唯一声明了 model 的文件，提示分类任务用 Haiku 是作者知道的正确做法，但没有系统性应用。

### 3.3 当时的缺陷

1. **0/33 agents 无 model 声明** → 为什么这么设计会失败：所有 agents 都用 Sonnet，但 `gsd-doc-classifier`（分类任务）、`gsd-roadmapper`（生成路线图，也许不需要 Opus）等理应用更便宜的模型。成本预测不可能，用量大时费用累积不可控。**自查**：我的 agents 都合理声明了 model 吗？

2. **gsd-advisor-researcher 76/100（最低分）** → 三重扣分：无 model(-5) + 零示例(-15) + 模糊量词 "focused" "genuine"(-4)。"genuine recommendations" 无法被 AI 操作化——什么叫「真诚的建议」是主观的，AI 无法验证。根本原因：作者希望 agent 有高质量输出，但用了不可验证的量词表达期望。

3. **9 个瘦委托 skills 75/100** → `cleanup`、`do`、`fast` 等 skills 只有「目标」描述和 `@workflow` 引用，没有示例和输出格式。这类 skill 存在是合理的（轻量入口），但缺乏示例使得用户无法快速判断调用效果。

### 3.4 当时的优化机会

1. 给全部 33 agents 加 `model:` 声明（Haiku/Sonnet 分级）
2. 给 9 个瘦委托 skills 加 output format 描述和最小示例
3. 替换 gsd-advisor-researcher 中的 "genuine" 为可测量约束

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 0/33 agents 无 model 声明 | `grep -l "^model:" agents/*.md` | **持续**：0 个 agents 声明 model | 3 个月完全未修 |
| gsd-advisor-researcher 零示例 | `grep -c "Example\|Returns:" agents/gsd-advisor-researcher.md` | **持续**：无示例 markers | 最低分 agent 未改进 |
| PostToolUse 匹配过宽（含 Read/Grep/Glob）| `grep "matcher" hooks/hooks.json` | **已修复**：PostToolUse 现在仅匹配 `Edit|Write`（另有独立 Bash hook）| 性能优化已落地 |

**关键发现**：hooks.json 已经发生了重大重构——从「单层 hook + 宽匹配」演进为「分层嵌套 hook + 精确匹配」。现在的 hooks.json 有多个 SessionStart hook（分别处理 session 状态、bash hook、shadowing detector 等），架构比审计时更为复杂和健壮。MEDIUM 安全问题（PostToolUse 宽匹配）被修复，但 model 声明的 LOW-HIGH aggregate 问题完全未动。

### 4.2 架构演进

hooks.json 的重构是最大变化：从 4 个简单 hook 扩展为「多层嵌套 hooks + run-bash-hook.cjs 独立进程 + 读检测器（gsd-shadowing-sdk-detector.js、gsd-staleness-reminder.js 等）并行运行」。这些只读检测器（判断是否有 shadowing、是否有过期提示）被设计为幂等的，即使在多版本并存时也不会双触发。这是一个架构升级：从「单一 hook 入口」到「分层 hook 系统」。

### 4.3 新增的可学习模式

**Hook 分层 + 幂等探测器**：新增的多个只读探测器 hook（`gsd-shadowing-sdk-detector.js`、`gsd-staleness-reminder.js`、`gsd-prompt-guard.js`、`gsd-read-injection-scanner.js` 等）展示了一个新模式：hook 可以分为「状态变更型」（需要选举机制避免双触发）和「只读检测型」（幂等，双触发无害）。这两类 hook 的处理策略完全不同，分开是正确的架构决策。

---

## 五、校准

### 5.1 我已经在做对的

1. **有状态工作流的思路**：我的 bureau 项目也在探索跨 session 持久化，与 GSD 思路一致
2. **skills 单职责**：我的 gstack skills 每个聚焦单一操作
3. **多子命令 skill 结构**：gstack 的 openclaw skills 也采用了类似的子命令描述方式

### 5.2 挑战 / 验证

**挑战了我的假设**：我之前认为「hooks 只适合做简单的触发点」。但 gsd-plugin 的 hooks 实现了完整的工作流状态机，用 hooks 追踪每次工具调用的「前因后果」。这说明 hooks 可以是一个严肃的状态管理机制，而不只是「运行完命令后发个通知」。值得深入研究。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 有没有声明 model
for f in $(find . -path "*agents/*.md" 2>/dev/null | grep -v node_modules); do
  has_model=$(grep -c "^model:" "$f" 2>/dev/null)
  if [ "$has_model" -eq 0 ]; then
    echo "NO model declared: $f"
  fi
done
```
命中后怎么办：根据 agent 复杂度分级——分类/验证任务 → `model: haiku`；规划/调试/研究 → `model: sonnet`；无需 Opus 级别的任务不要声明 opus。

```bash
# 检查 agents 中的模糊量词
grep -rn -E '\b(appropriate|comprehensive|relevant|reasonable|proper|genuine|focused)\b' \
  .claude/agents/*.md 2>/dev/null | head -20
```
命中后怎么办：用具体的、可测量的描述替换。例如，「appropriate error handling」→「对网络错误、空输入、4xx/5xx 状态码分别处理」。

### 6.2 灵感 → 实施路径

1. **想法**：为 hooks 引入「只读检测器」模式
   - **为何可行**：gsd-plugin 证明了 hooks 可以安全地进行幂等读取检查（staleness、shadowing），不会影响主流程
   - **第一步**：为 bureau 或 WorldMonitor 写一个 `PostToolUse` 只读检测 hook，在每次写文件后检查 schema 合规性（30-60 分钟）

2. **想法**：给 agents 的 model 声明按「任务复杂度矩阵」系统化处理
   - **为何可行**：gsd-plugin 中 set-profile 用 Haiku 是正确的，但只是孤立实例；矩阵化决策让未来新增 agent 时有明确规则
   - **第一步**：建立一个简单的决策矩阵：分类/枚举 → Haiku；规划/调试/研究 → Sonnet；无 opus 级需求。为现有所有 agents 逐一判定（30 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：Claude Code 全生命周期项目管理工作流（计划→执行→调试→发布）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 均探索跨 session 状态持久化；均有结构化工作流 | bureau 专注知识管理/复盘，GSD 专注任务执行 | 高 |
| MarkQWu/gstack | 中 | 均为 Claude Code 工具集；均有 skills + agents | gstack 无状态（无持久化），GSD 有完整状态机 | 中 |
| MarkQWu/- | 低 | 均为生产级工具 | WorldMonitor 是数据仪表盘，非工作流框架 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agents 无 model 声明 | `grep -L "^model:" /tmp/my-repos/MarkQWu-gstack/openclaw/agents-gstack-section.md` | gstack 的 agents 文档是 AGENTS.md 汇总格式，无单独 agent 文件，无此问题 | 无 |
| 模糊量词 | `grep -rn "appropriate\|relevant\|reasonable" /tmp/my-repos/MarkQWu-gstack/openclaw/skills/*/SKILL.md` | 需人工验证 | 中 |
| 无示例 agents | 暂无独立 agent 文件可检查 | 暂无 | 无 |

**命中后的具体行动建议**：
- 若未来在 bureau/gstack 中新建 agent，默认模板应包含 `model: haiku` 或 `model: sonnet` 字段

### 8.3 别人的更优方案

1. **领域**：Hook 系统的架构设计
   - **本案做法**：hooks 分层（状态变更 vs 只读检测），PostToolUse 精确匹配（仅 Edit|Write），避免在 Read/Grep/Glob 时触发不必要的状态更新
   - **我的项目现状**：WorldMonitor 和 bureau 目前无 hooks 配置
   - **如何借鉴**：为 bureau 的核心工作流添加 PostToolUse hook，在写入 capture/ 后触发 schema 验证；参考 gsd-plugin 的「Edit|Write 匹配」而非宽匹配

2. **领域**：debug/SKILL.md 的安全说明内嵌
   - **本案做法**：`skills/debug/SKILL.md`（100 分）在 Examples 之后有专门的「Security Notes」小节，列出调试时需注意的边界（如不执行 untrusted code、sandbox before running）
   - **我的项目现状**：gstack/graphify 的 skills 无安全说明
   - **如何借鉴**：在有权限操作（写文件、执行脚本）的 skills 中加 `## Security Notes` 小节

### 8.4 反向：我的项目做得比他们好的地方

MarkQWu/- 的所有 25 个 skills 都有完整的 `name:` 字段和版本号（`version: 1`），且 skill 文件置于规范的 `.well-known/agent-skills/` 路径下，发现性（discoverability）优于 gsd-plugin 的 agents 完全无 model 声明的状态。世界监控项目的 skills 也均包含认证说明（Authentication 小节），这是 gsd-plugin 未涉及的额外质量维度。

---

## 八、术语表

### <a name="状态机"></a>状态机（State Machine）
> 一种根据「当前状态 + 触发事件」决定下一步行为的编程模式。gsd-plugin 用 `.planning/` 目录下的文件记录当前处于哪个 phase、哪个 milestone，每次 Claude 执行操作时 hook 更新状态文件——这就是状态机。与「无状态」相比，状态机能让 AI 「记住」上次做到哪里了。

### <a name="幂等"></a>幂等（Idempotent）
> 执行一次和执行多次效果相同的操作。gsd-plugin 的只读检测器 hooks（如 `gsd-staleness-reminder.js`）被设计为幂等的：即使被多次触发（比如两个版本的插件同时加载），结果也不会叠加。与之对比，状态写入操作不是幂等的——如果两次都写入「phase 完成」，可能导致状态文件冲突。

### <a name="dispatch-table"></a>Dispatch Table（调度表）
> 用一张表（通常是 JSON 或 Markdown 表格）明确列出「这个输入 → 调用那个函数/agent」的映射关系。gsd-debug-session-manager 的 debug 状态调度表是一个范例——避免了一大段 if/else 描述逻辑，让调用关系一目了然。
