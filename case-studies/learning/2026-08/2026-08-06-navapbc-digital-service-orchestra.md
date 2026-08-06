# navapbc/digital-service-orchestra — 学习案例

**仓库**：https://github.com/navapbc/digital-service-orchestra
**Stars**：3 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-08-06（基于当前 HEAD）
**主题标签**：`router-channels`, `examples-driven`, `template-design`, `cross-reference`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`digital-service-orchestra`（DSO）是由 navapbc（美国数字服务技术公司）出品的重型 Claude Code 插件，为工程团队提供完整的软件交付工作流：sprint 规划、bug 修复、代码审查（含 red team 和 blue team）、实现计划、接口合同等。当前版本 1.17.149，有 53 个 agent、38 个 skill、4 个 plugin 命令、完整的 ticket 系统、pre-commit hooks、CI review 流水线。Stars 只有 3，但这是企业内部工具对外开放的典型——代码质量远超星数。

### 1.2 架构剖析

- **目录结构**：
  ```
  .
  ├── plugins/dso/
  │   ├── .claude-plugin/plugin.json   # 插件清单（version 1.17.149）
  │   ├── agents/                      # 53 个 agent（代码审查专家团）
  │   │   ├── code-reviewer-*.md       # 分维度代码审查 agent（arch/correctness/security/perf等）
  │   │   ├── bloat-*.md               # 代码膨胀检测 agent
  │   │   ├── completion-verifier.md   # 完成状态验证
  │   │   └── complexity-evaluator.md  # 复杂度评估
  │   ├── commands/                    # 4 个命令（commit/end/review/dryrun）
  │   └── skills/                      # 38 个 skill
  ├── .claude/
  │   ├── hooks/pre-commit/             # pre-commit 检查 shell scripts
  │   ├── skills/skill-refactor/SKILL.md
  │   └── scripts/dso                  # DSO 控制脚本
  ├── scripts/dso_ci_review/           # CI LLM 代码审查 Python 包
  ├── prompt-registry/                 # 提示词注册表
  └── tests/BASELINE.md
  ```
- **文件类型分布**：53 个 agent / 38 个 SKILL.md / 4 个 plugin 命令 / 4 个 pre-commit hook
- **编排关系**：多层分工。`sprint/SKILL.md` 是 orchestrator，它 spawn 53 个专业 agent（`code-reviewer-deep-arch.md`、`code-reviewer-security-red-team.md` 等）并聚合结果。每个 agent 对应一个审查维度，互相独立运行，最终由 `code-reviewer-arbiter.md` 仲裁冲突意见。
- **跨件契约**：`commit.md` 命令通过 `!`{动态引用} 读取 `${CLAUDE_PLUGIN_ROOT}/docs/workflows/COMMIT-WORKFLOW.md`，真正的执行步骤在外部文档里——命令文件本身只有 10 行。这是一种「瘦命令+厚文档」的分离模式。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「评审委员会模型」——不用一个全能 agent，而是把代码审查分解为独立维度（架构/正确性/安全/性能/卫生），每个维度由一个专业 agent 独立评估，最终仲裁。这反映了「多独立视角 > 单一全知视角」的认知模型。
- **解决什么问题**：企业级软件交付流程中，代码审查是质量瓶颈——人工审查慢且依赖个人经验，不同审查者关注点不同导致标准不一致。DSO 用 agent 网格提供标准化、可重复的多维度审查。
- **做了什么 trade-off**：53 个 agent 意味着每次 sprint 运行时并发 spawn 大量子任务，token 消耗大，但覆盖面广。他们选择覆盖面而非成本，反映企业工具的优先级排序。
- **反映什么认知模型**：作者把 AI agent 视为可以扮演具体专业角色的「数字员工」，而不是通用助手。`red-team-reviewer.md` 和 `blue-team-filter.md` 的命名本身就说明作者在"雇佣"一个红队和蓝队。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：专家 Agent 网格 + 仲裁层（Expert Agent Grid + Arbiter）**

把一个复杂任务（代码审查）分解为 N 个独立维度，每个维度由一个专业 agent 独立评估，最终结果由一个仲裁 agent 聚合和协调。任何单个 agent 不知道其他 agent 的存在——它们是平行独立的工人，仲裁者是它们的上层管理者。

模式特征清单（3-5 条）：
- N 个 agent 各司一职，维度之间无依赖
- 每个 agent 输出结构化 JSON（便于仲裁者聚合）
- 专门的仲裁 agent 负责解决冲突意见
- orchestrator skill（sprint）负责启动 N 个 agent 并收集结果
- 命名遵循 `{维度}-{角色}.md` 约定（如 `code-reviewer-deep-arch.md`）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多角度独立评估的复杂任务 | ✅ 高度适用 | 多 agent 独立性保证了覆盖面，仲裁层解决冲突 |
| 需要快速、轻量的单次操作 | ❌ 不适用 | 53 agent 并发消耗大，kill fly with cannon |
| 企业/团队内部工具（成本不敏感） | ✅ 高度适用 | token 成本可接受，一致性和覆盖面是更高优先级 |
| 个人开发工具 | ❌ 不适用 | 对个人开发者来说 token 消耗太大 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 专家 Agent 网格+仲裁层（本仓库） | navapbc/DSO | 覆盖全面，标准一致，可扩展 | token 消耗大，维护 53 个 agent 文件工作量大 |
| 单 orchestrator skill | gstack, bureau | 轻量，token 友好 | 覆盖面受 orchestrator 能力限制 |
| 顺序 pipeline | agent-sh/enhance | 确定性强，便于调试 | 无法并行，速度慢 |

### 2.4 改进空间

1. **当前问题**：所有 32 个 skill 文件缺少 `model:` 声明（每个扣 -5）。**改进做法**：在 sprint、fix-bug 等重型 orchestrator skill 里加 `model: claude-sonnet-5`；轻量 skill 如 quick-ref 加 `model: claude-haiku-4-5`。**预期收益**：分数从 85.8 提升到 90.8，同时按任务复杂度优化成本。
2. **当前问题**：`commit.md`、`review.md` 等 3 个 plugin 命令缺 YAML frontmatter（name/description/model 全无）。**改进做法**：补充完整 frontmatter。**预期收益**：分数从 18–20 提升到 90+，这 3 个命令目前实际上无法被 NLPM 注册。
3. **当前问题**：`plan-review/SKILL.md` 零示例（-15）。**改进做法**：加一个 "PR diff → review output JSON" 示例。**预期收益**：分数从 85 提升到 100。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-27 当时得分 **84.8/100（plugin 工件）/ 66.8/100（含 dispatch shim）**。

| 类别 | 当时分数 | 主要问题 |
|---|---|---|
| 31 个 agent（均值） | 90.8/100 | 大多数 1 个示例（应 2+）|
| 23 个 dispatch shim（均值） | 15/100 | 设计上无 frontmatter，属基础设施文件 |
| 3 个 plugin 命令（均值） | 18/100 | 无 YAML name/description/model/examples |
| 32 个 skill（均值） | 85.8/100 | 全部缺 model 声明（-5 each）|

### 3.2 当时值得借鉴的模式

1. **`complexity-evaluator.md` 满分 97/100** → 6 个"before/after"对比示例，评分标准具体到数字（如"圈复杂度 > 15 = high"），输出为结构化 JSON。如何借鉴：所有评估类 agent 都应该有 B/F 示例 + 数值化标准。
2. **`completion-verifier.md` 96/100** → 有 sentinel-state 状态表 + 3 个 gate 通过/不通过示例。如何借鉴：验证类 agent 应该有明确的"通过/不通过"状态机和案例。
3. **`code-reviewer-arbiter.md`** → 专门解决多 agent 意见冲突的仲裁层设计，在多 agent 体系中非常必要。如何借鉴：任何需要合并多个来源输出的场景都值得考虑仲裁层。
4. **`red-test-evaluator.md` 95/100，5 个 verdict 示例** → 红队测试评估 agent，有非常清晰的 pass/fail 判定标准。如何借鉴：任何有确定性输出的 agent 都应该用 verdict 模式（而不是模糊的"给出建议"）。
5. **pre-commit hook 质量守门** → `.claude/hooks/pre-commit/` 有 5 个 shell 脚本检查插件边界、SKILL.md 自引用等——把质量检查内置到 git workflow 中。如何借鉴：把 NL 工件的关键约束写成 pre-commit hook。

### 3.3 当时的缺陷

1. **所有 skill 缺 model 声明** → 为什么这么设计会失败：Claude Code 的 skill 继承调用者的模型，缺少 `model:` 声明不代表不工作，但意味着系统无法按任务复杂度分配最优模型（轻量任务用 Haiku 节省成本）。这是"能用但不够好"的设计缺陷。自查：我的 skill 有没有缺 model 声明的？
2. **`commit.md` / `review.md` 零 frontmatter** → 这两个命令是用户最常用的入口，却得分最低（15–20/100）。根本原因：作者把核心逻辑放在外部文档（`COMMIT-WORKFLOW.md`）里，命令本身只做"读外部文档然后执行"的转发，因此忽略了命令文件自身需要完整 frontmatter。这不是懒，而是架构决策的副作用——瘦命令模式牺牲了 NL 注册质量。自查：我的命令文件有没有靠引用外部文档的？
3. **dispatch shim 的系统性低分** → 23 个 shim 平均 15/100，全部是因为「设计上无 frontmatter」。审计者明确指出这是有意为之的基础设施文件，不是语义质量问题。但这导致「plugin 工件 87/100」和「全部工件 66.7/100」产生巨大差距。自查：我的项目有没有生成或存在「基础设施 Markdown」文件被 NLPM 错误计分的情况？

### 3.4 当时的优化机会

1. **为 plugin commands 补全 frontmatter**：3 个命令（commit/end/review）补充完整 YAML frontmatter，分数从 18 到 90+。
2. **为 plan-review/SKILL.md 补示例**：1 个 PR diff → review JSON 示例，从 85 到 100。
3. **为 skill 加 model 声明**：建议轻量 skill（quick-ref/status）声明 `model: haiku`，重型 skill（sprint/debug-everything）声明 `model: sonnet`。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 32 个 skill 缺 model 声明 | `grep -l "^model:" plugins/dso/skills/*/SKILL.md` → 2 个有 | **持续（几乎全部）**：38 个 skill 中仅 2 个有 model 声明，36 个仍缺 | 版本从 1→1.17，技术债未解决 |
| commit.md 零 frontmatter | `head -3 plugins/dso/commands/commit.md` | **持续**：仍无 YAML frontmatter，第一行是 `# Commit Command` | 最高优先级命令依旧无法被 NLPM 注册 |
| dispatch shim 消失 | `ls .claude/commands/` | **已修复**：`.claude/commands/` 目录现已清空（0 文件），dispatch shim 被完全移除 | 重大架构变化：从「shim 路由」改为「直接调用 plugin commands」，66.7 的总分问题因此消失 |

### 4.2 架构演进

从 2026-04-27 到现在，最重大的变化是：
1. **dispatch shim 被全部删除**：原来 `.claude/commands/` 里有 23 个 shim 文件（15/100 各），现在目录为空。这意味着整体 NL 分数的分母缩小了 23，总体分数会大幅提升。
2. **agent 数从 31 增到 53**：新增 22 个 agent，覆盖 `approach-proposer`、`architectural-probe`、`bug-classifier-haiku`（专门用 Haiku 做分类）等更细化的任务。
3. **skills 从 32 增到 38**：新增 6 个 skill。
4. **底层代码完全重写**：CI LLM review runner 从 bash 迁移到 Python 包（`dso_ci_review`）。

说明作者后来意识到 shim 层不必要（增加维护负担且 NLPM 分数差），直接删除比修复更简洁——这是「架构决策优于打补丁」的好例子。

### 4.3 新增的可学习模式

1. **`bug-classifier-haiku.md`** — 专门声明 `model: claude-haiku` 的轻量分类 agent。用 Haiku 做枚举分类而不是高成本模型，是成本优化的具体实践。
2. **Python CI review runner**：`scripts/dso_ci_review/` 包含 `classifier.py`、`findings.py`、`dispatch.py`、`runner.py`——把 CI 环境的 LLM 调用从 bash 脚本迁移到类型化的 Python 包，可测试性和可维护性大幅提升。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau 的命令文件有 frontmatter**：bureau/commands/ 的 lint.md 和 status.md 都有完整 frontmatter，避免了 commit.md 的陷阱。
2. **gstack 的 agent 文件有明确 model 声明**：gstack 的 SKILL.md 虽然缺 model 字段，但在 plugin.json 层面有版本控制，结构比 DSO 的 skill 层更规范。
3. **结构化示例在部分 skill 中已存在**：bureau 的 7/7 skill 文件有 `## Examples`（虽然示例不够丰富）。

### 5.2 挑战 / 验证

**认知挑战**：dispatch shim 从 23 个缩减到 0 个，是这次案例最出乎意料的发现。我之前以为 shim 是一种「必要的间接层」，而 DSO 的演进告诉我：如果 plugin 系统支持直接调用，shim 是额外的维护负担，删掉比维护更好。这挑战了我「多一层间接总是更灵活」的默认假设。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 是否有 model 声明
find . -name "SKILL.md" | xargs grep -L "^model:" 2>/dev/null | head -20
# 命中后怎么办：按任务复杂度加 model 声明（轻量=haiku，重型=sonnet）
```

```bash
# 检查命令是否有完整 frontmatter（name + description + allowed-tools）
find . -name "*.md" -path "*/commands/*" | while read f; do
  has_name=$(grep -c "^name:" "$f" 2>/dev/null || echo 0)
  has_desc=$(grep -c "^description:" "$f" 2>/dev/null || echo 0)
  [ "$has_name" -eq 0 ] || [ "$has_desc" -eq 0 ] && echo "MISSING: $f"
done
# 命中后怎么办：补全 name/description/allowed-tools
```

```bash
# 检查 skill 示例数量（应 ≥2）
find . -name "SKILL.md" | while read f; do
  count=$(grep -c "## Example" "$f" 2>/dev/null || echo 0)
  [ "$count" -lt 2 ] && echo "$f: $count examples"
done
# 命中后怎么办：命中文件的示例数不足 2，至少加 1 个 before/after 或 input→output 示例
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 DSO 的 `complexity-evaluator.md` 模式，为 bureau 的 `review/SKILL.md` 加数值化评分标准。
   - **为何可行**：`complexity-evaluator.md` 用圈复杂度数字触发行动（>15=high），避免了"过于复杂"这样的模糊判断。bureau 的 review skill 目前用"高/中/低"语义，不够确定。
   - **第一步**：修改 bureau 的 `review/SKILL.md`，把"发现重大问题"改成"发现 ≥3 个跨文件矛盾"，把"次要问题"改成"1–2 个单文件不一致"。20 分钟可完成。

2. **想法**：把 DSO 的 pre-commit hook 质量检查引入我的项目。
   - **为何可行**：DSO 的 `check-plugin-boundary.sh` 等 hook 在 commit 时检查插件边界，把 NL 工件的约束内置到 git workflow 里。我的项目目前没有 NL 工件专属的 pre-commit 检查。
   - **第一步**：参考 NLPM 的 `templates/pre-commit-nlpm.sh`，在 bureau 项目根加一个 pre-commit hook，检查 skill 文件是否有 frontmatter。10 分钟可完成。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 navapbc/digital-service-orchestra 的核心目的**：企业级软件交付工作流 Claude Code 插件，核心是多维度代码审查 + sprint 管理 + 完成验证。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是「AI 辅助知识/质量管理」工具；都有 hooks；都有多个 skill 协同 | bureau 管知识库，DSO 管软件交付；规模差异巨大（7 skill vs 53 agent） | 高（agent 网格模式可参考）|
| MarkQWu/gstack | 低 | 都是个人/团队工具套件 | gstack 是轻量工具集，DSO 是重型企业平台 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 缺 model 声明 | `grep -L "^model:" bureau/skills/*/SKILL.md gstack/*/SKILL.md` | bureau 7/7 skill 缺；gstack 54/54 SKILL.md 缺 | 中（影响成本优化，不影响功能）|
| Plugin 命令缺 frontmatter | `grep -L "^name:" bureau/commands/*.md` | bureau/commands/lint.md 有 description 无 name；status.md 同 | 中（name 缺失影响 NLPM 注册）|
| 单示例不足 | `grep -c "## Example" bureau/skills/*/SKILL.md` | bureau recall/SKILL.md 有 2 个示例，其余 6 个各 1 个 | 中（≥2 示例是最佳实践）|

**命中后的具体行动建议**：
- bureau/commands/lint.md 缺 `name:` 字段 → 加 `name: lint` 到 frontmatter 顶部。5 分钟可完成。
- gstack 54 个 SKILL.md 全缺 model → 重型 skill（diagram、land-and-deploy）加 `model: claude-sonnet-5`，其余轻量 skill 加 `model: claude-haiku-4-5-20251001`。可用脚本批量处理，30 分钟完成。

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：多维度独立评估 + 仲裁层
   - **本案例做法**：53 个 agent 分维度独立跑，`code-reviewer-arbiter.md` 聚合冲突意见。每个 agent 输出结构化 JSON，仲裁者可以精确比较
   - **我的项目现状**：bureau 的 `review/SKILL.md` 是单一 skill，一次扫描所有问题，没有维度分离也没有仲裁层
   - **如何借鉴**：为 bureau 的 review 功能创建两个轻量 agent：`consistency-checker.md`（检查逻辑矛盾）和 `freshness-checker.md`（检查内容时效性），结果由主 skill 聚合

2. **领域**：`completion-verifier.md` 的 sentinel-state 状态机
   - **本案例做法**：验证 agent 维护一个状态表（PENDING/PARTIAL/COMPLETE/BLOCKED），输出包含当前状态和阻塞原因
   - **我的项目现状**：bureau 的 `review/SKILL.md` 输出是叙述性文本，没有状态机
   - **如何借鉴**：在 bureau review 的输出规范里加一个 `status: CLEAN|ISSUES_FOUND|BLOCKED` 字段

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 文件 `## Examples` 存在率
- **我的做法**：bureau 7/7 skill 文件都有至少 1 个 `## Examples` section（gstack 也普遍有示例）
- **本案例做法（弱在哪）**：audit 时 `plan-review/SKILL.md` 有 0 个示例（扣 -15），32 个 skill 中大多数只有 1 个示例（每个扣 -5）
- **意义**：bureau 在示例覆盖率上已有基础优势，需要的是从「有示例」提升到「≥2 个高质量示例」。

---

## 八、术语表

### <a name="agent网格"></a>Agent 网格
> 多个专业 AI Agent 并行独立完成各自的评估任务，最终由一个协调层汇总结果的架构。类比一个审查委员会：每个委员（agent）只评审自己负责的维度（架构/安全/性能），委员长（arbiter）综合所有意见。好处是覆盖面广、各维度不互相影响；坏处是成本高。

### <a name="dispatch-shim"></a>Dispatch Shim
> 一个只有 5–10 行的「转发文件」，功能是把用户的调用路由到真正的 skill 文件。例如：`.claude/commands/sprint.md` 只有一行"调用 `dso:sprint` skill"，真正的逻辑在 `plugins/dso/skills/sprint/SKILL.md`。Shim 的好处是允许用户用项目本地命令调用 plugin skill；坏处是需要维护两份文件，且 NLPM 会对 shim 单独计分（通常很低）。

### <a name="sentinel-state"></a>Sentinel State
> 一种状态机模式：用明确的枚举值（如 PENDING/COMPLETE/BLOCKED）表示工作项的当前状态，而不是用模糊描述。当 Agent 输出包含 sentinel state 时，下游 Agent 或自动化系统可以精确判断"是否完成"而不需要解析自然语言。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读命令或 skill 文件时先解析 frontmatter，才能知道这个文件是什么、能做什么。
