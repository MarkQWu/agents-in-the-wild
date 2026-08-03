# navapbc/digital-service-orchestra — 学习案例

**仓库**：https://github.com/navapbc/digital-service-orchestra
**Stars**：3 | **来源**：upstream（≥500 池耗尽，按学习价值补选）
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-08-03（基于当前 HEAD）
**主题标签**：`router-channels`, `single-purpose`, `examples-driven`, `cross-reference`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
DSO（Digital Service Orchestra）是 navapbc（美国政府数字服务机构）开发的 Claude Code 插件，专为政府数字项目工程团队设计，提供标准化代码审查、计划审查、调试、记录和冲刺管理工作流。它通过"[分发垫片](#dispatch-shim)"（`.claude/commands/` 中的 7 行基础设施文件）路由到 `plugins/dso/` 中的真实 NL 指令，实现安装检测与优雅降级。

关键事实：
- 2026-04-27 被 NLPM 审计
- navapbc 是真实政府机构（前身 USDS/18F 生态），内部工程质量标准高
- 89 个工件（31 agents + 26 commands + 32 skills），但 23 个命令是垫片
- Plugin 工件 NL 得分 **84.8/100**，整体含垫片 **66.8/100**
- 安全审计 PASS（无 Critical/High 发现）

### 1.2 架构剖析

```
digital-service-orchestra/
├── .claude/
│   └── commands/          # 23 个分发垫片（基础设施，非 NL 工件）
│       ├── architect-foundation.md  # 7 行 boilerplate
│       ├── debug-everything.md
│       └── ...
├── plugins/
│   └── dso/
│       ├── .claude-plugin/plugin.json
│       ├── agents/        # 31 个高质量 agent（平均 90.8/100）
│       │   ├── code-reviewer-deep-correctness.md
│       │   ├── bot-psychologist.md       # 17 点故障分类法
│       │   ├── completion-verifier.md
│       │   └── ...
│       ├── commands/      # 3 个真实 plugin commands（非垫片）
│       │   ├── commit.md
│       │   ├── review.md
│       │   └── end.md
│       ├── skills/        # 32 个 skills（平均 85.8/100）
│       │   ├── brainstorm/SKILL.md
│       │   ├── sprint/SKILL.md
│       │   └── ...
│       ├── hooks/         # auto-format.sh, config-paths.sh
│       └── scripts/       # onboarding, verify-review-diff.sh
├── docs/workflows/        # 外部 phase 文件，SKILL.md 引用
└── prompt-registry/       # 可能用于 plan-review-dispatch.md
```

**文件类型分布**：31 agents / 23 dispatch shims / 3 真实 plugin commands / 32 skills / hooks / scripts

**编排关系**：
1. 用户输入 → `.claude/commands/debug-everything.md`（分发垫片）
2. 垫片检测插件是否安装 → 路由到 `Skill tool` 或 `plugins/dso/skills/debug-everything/SKILL.md`
3. SKILL.md 可能派发 agents（如 code-reviewer-deep-correctness）

**跨件契约**：
- plugin.json 注册所有 31 个 agents，与文件系统完全同步（CC002 已验证无孤岛、无遗漏）
- agents 通过 `REPO_ROOT` 环境变量与 `scripts/dso` shim 通信
- 生成型 agents（code-reviewer-*）共享同一 JSON findings schema

### 1.3 设计思路 / 方法论

**核心设计哲学**：
- **协议化 JSON schema**：11 个 code-reviewer agents 全部输出相同的 findings JSON，下游可机器读取
- **分层质量**：垫片（基础设施）vs 真实 NL 工件（plugin commands/agents/skills）分开评分
- **`REVIEW-DEFENSE` 模式**：在 agent 正文中用 200 字的注释块解释「为什么此处不遵循常规约定」，防止未来维护者「修复」掉有意为之的设计

**解决什么问题**：政府项目跨团队 code review 标准化 —— 不同严格程度（light / standard / deep）的审查者 + 专域（correctness / security / performance / test-quality）分工，产出可追溯的结构化 findings。

**做了什么 trade-off**：
- 分发垫片导致整体 NL 得分从 84.8 降到 66.8（23 个垫片每个 15 分），用户看到的得分不真实
- Skills 无 model 声明（继承自调度 orchestrator），学习成本换灵活性
- 外部 phase 文件（`docs/workflows/prompts/`）使 skills 更薄，但阅读时需跳转

**认知模型**：作者把 AI agents 看作「可编程的专家审查者」—— 每个 agent 承担单一审查维度，通过 JSON 契约与上游 orchestrator 通信，而非自由文字输出。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「分发垫片 + 插件指令分层」（Dispatch-Shim Layer）**

关键特征：
- 基础设施文件（垫片）不携带 NL 语义，只做路由
- 真实指令全在 plugin 目录，可独立评分和维护
- 垫片模板化生成（install 时自动写入 `.claude/commands/`）
- NLPM 需要将垫片排除在聚合得分之外，否则失真

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要「安装/未安装」优雅降级 | ✅ 高度适用 | 垫片可检测插件并路由 |
| 小型个人 skill 套件 | ❌ 过度设计 | 无需垫片层，直接在 `.claude/commands/` 写真实命令即可 |
| 多团队共享插件仓库 | ✅ 适用 | 垫片标准化安装接口 |
| 要求 NLPM 得分真实 | ⚠️ 需排除垫片 | 否则 66.8 vs 84.8 的差距误导评估 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 分发垫片 + 插件分层（本案） | navapbc/dso | 优雅降级、真实 NL 工件可独立评分 | 垫片拉低整体得分，需 NLPM 排除规则 |
| 直接命令（无垫片） | `addyosmani/agent-skills` | 简单，得分直观 | 无安装检测，插件不存在时静默失败 |
| Meta-skill Router | `musistudio/claude-code-router` | 单点入口，动态路由 | 增加路由复杂度 |

### 2.4 改进空间
1. **当前问题**：`commit.md`/`review.md`/`end.md` 三个真实 plugin commands 缺少 YAML [frontmatter](#frontmatter)（score 20/100）。**改进做法**：加三行 frontmatter（name + description + allowed-tools）。**预期收益**：score 提升至 ~80/100，Claude Code agent picker 可正确显示。

2. **当前问题**：`huge-diff-reviewer-*` 三个 agent 的 REPO_ROOT 警告和代码块相互矛盾（BUG-001）。**改进做法**：给 Step 1/3 代码块加注释「此处 git rev-parse 仅用于定位 shim 脚本，勿用于源文件查找」，警告改为精确描述使用场景。**预期收益**：agent 在 worktree 会话中不再产生误报/漏报。

3. **当前问题**：32 个 skills 全部缺 `model:` 声明，每个 -5 分。**改进做法**：在 `.claude/nlpm.local.md` 加规则覆盖，suppres `NL-MISSING-MODEL` 作用于 `plugins/dso/skills/**`。**预期收益**：plugin-only score 从 84.8 提升至 ~87+。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-27 当时 plugin 工件得分 **84.8/100**，整体 **66.8/100**。

| 区域 | 当时分数 | 主要问题 |
|---|---|---|
| Agents（31 个） | 90.8 avg | 多数 agent 只有 1 个 JSON 示例（−5 each） |
| 分发垫片（23 个） | 15.0 avg | 无 frontmatter by design |
| Plugin commands（3 个） | 18.0 avg | 全部缺 YAML name+description |
| Skills（32 个） | 85.8 avg | 全部缺 model 声明（−5 each），plan-review 零示例（−15） |

### 3.2 当时值得借鉴的模式
1. **`REVIEW-DEFENSE` 注释块** → 在 `scope-drift-reviewer.md` 中用专门块解释「name 为什么不加前缀」，防止未来维护者「修正」这个有意为之的设计。这是「让 AI 不要修复的注释」的最佳实践。如何借鉴：在自己的 NL 工件中遇到违反惯例的地方，用 `REVIEW-DEFENSE:` 格式显式说明原因。

2. **结构化 JSON findings 契约** → 所有 11 个 code-reviewer agents 输出完全相同的 findings JSON schema，可被机器解析和聚合。如何借鉴：设计多 agent pipeline 时，先定义 schema，再写 agent，而非让每个 agent 自由输出。

3. **CC002 manifes t完整性验证** → plugin.json 与文件系统 100% 同步，NLPM 审计专门验证了这一点。如何借鉴：在 CI 中加 `jq` 验证脚本，对比 plugin.json 中 agents 列表和 agents/ 目录文件数是否匹配。

4. **`completion-verifier.md` 哨兵状态表** → 用 sentinel-state 表格而非文字描述工作完成标准（95/100）。如何借鉴：验证型 agent 的输出格式优先用状态表而非段落。

5. **多维度专家系统** → 11 个单维度 reviewer（correctness/performance/security-red/security-blue/test-quality 等）而非一个「全面的」reviewer。如何借鉴：宁可拆分为多个窄职责 agent，而非在一个 agent 里塞太多。

### 3.3 当时的缺陷
1. **问题**：`huge-diff-reviewer-*` 中 Step 1/3 代码块与 Step 2 警告直接矛盾。**根本原因**：文档逐步更新时，警告描述比代码块更新（或反之），没有单一真实来源指导什么时候可以用 `git rev-parse`。自查：**我有没有**这种「文档说不要做 X，代码做了 X」的矛盾？在任何 CLAUDE.md 或 SKILL.md 中搜索「do NOT」附近是否有对应代码块。

2. **问题**：3 个 plugin commands（commit/review/end）缺少全部 YAML frontmatter。**根本原因**：这三个文件可能是早期手写的，没有走 plugin template 流程。自查：检查自己的 `.claude/commands/` 下是否有无 frontmatter 的命令文件。

3. **问题**：`plan-review.md` agent 零示例，仅 5 行，指向外部模板。**根本原因**：把复杂度推给外部 prompt 文件后，agent 本身变成「空壳」，使用者要读两个文件才能理解行为。自查：我的 skill/agent 是否有「只引用外部文件」但自身零示例的情况？

### 3.4 当时的优化机会
1. 为 23 个分发垫片 template 加 3 行最小 frontmatter（name/description），从 15 → ~60 分
2. 为 `plan-review.md` 加 2 个示例（通过 / 失败），从 85 → 95 分
3. 为 plugin commands 加 YAML frontmatter，从 18 → ~80 分

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-001 REPO_ROOT 矛盾 | `grep -n "REPO_ROOT\|Do NOT re-derive" huge-diff-reviewer-standard.md` | **部分修复**：警告措辞更精确（"for all grep/Read/Glob calls"），但 Step 1/3 代码块仍用 `git rev-parse` | 矛盾仍在，但副作用风险降低 |
| BUG-002 15 点→16 点分类法不一致 | `grep "taxonomy" bot-psychologist.md` | **变更**：现在 description 和代码均说 **17 点分类法**，分类项已从 16 增至 17 | 项目持续扩展分类法，文档自我同步 |
| Q004 commit.md 无 frontmatter | `head -5 plugins/dso/commands/commit.md` | **持续**：仍无 YAML frontmatter | 3 个月内未修复 |

### 4.2 架构演进
审计快照 vs 当前 HEAD：
- `bot-psychologist` 分类法从 15 点 → 16 点 → 17 点演进，说明作者在持续精炼故障分类体系
- 总体文件结构未变，说明核心架构稳定
- 安全中等风险（SEC-001 eval, SEC-002 sudo, SEC-003 curl 无 checksum）在当前 HEAD 中状态未验证

### 4.3 新增的可学习模式
- 分类法驱动的 `taxonomy_item` schema 字段设计：版本号从分类法数量而来（17 点 → 可能未来 18 点），说明 schema 是动态文档

---

## 五、校准

### 5.1 我已经在做对的
1. `allowed-tools:` 字段在 gstack SKILL.md 中已完整声明（plan-devex-review/SKILL.md 有 7 种工具明确列出）
2. `version:` 字段管理 skill 版本，比 DSO 的 skills 更有生命周期意识
3. bureau 的 CLAUDE.md 明确禁止 AI 词汇（"No AI vocabulary: robust, comprehensive..."），比 DSO 在 vague quantifier 控制上更主动

### 5.2 挑战 / 验证
本案例验证了：**dispatch shim 架构是真实需求，不是过度设计** —— 有真实用户场景（安装检测 + 优雅降级）。过去我认为「在 `.claude/commands/` 直接写就好」，这个案例提醒：如果你在做分发给他人安装的插件（而非个人配置），shim 层是合理的工程选择。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的命令文件是否有 YAML frontmatter
find ~/.claude/commands/ -name "*.md" -exec sh -c '
  if ! head -1 "$1" | grep -q "^---"; then
    echo "NO FRONTMATTER: $1"
  fi
' _ {} \;
```
命中后：给命令文件加 `name`/`description`/`allowed-tools`，得分从 ~40 → ~80。

```bash
# 检查自己的 agent/skill 中是否有"do NOT"与代码块矛盾
grep -rn "do NOT\|Do NOT" ~/.claude/agents/ ~/.claude/skills/ 2>/dev/null | \
  grep -v "^Binary"
```
命中后：手动检查附近代码块是否与警告一致。

### 6.2 灵感 → 实施路径
1. **想法**：在 bureau 或 gstack 中引入 `REVIEW-DEFENSE:` 注释约定。**为何可行**：bureau 的 CLAUDE.md 已有代码规范意识，加一个「为什么违反规范」的显式格式是自然扩展。**第一步**：在 bureau/CLAUDE.md 中加一节「REVIEW-DEFENSE 格式约定」，约定任何违反 NLPM 规范的有意设计必须用此格式标注，15 分钟可完成。

2. **想法**：在 gstack 的 skill pipeline 中引入 JSON schema 输出契约。**为何可行**：gstack 的 plan-devex-review 已有结构化输出，显式化 schema 可让下游 skills 可靠消费。**第一步**：选一个最常用的 gstack skill，在 `## Output Format` 节加 JSON schema block，30 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：政府工程团队的标准化代码审查 + 工程流程编排插件
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 工程流程插件、commands 驱动、hooks 存在 | bureau 侧重知识管理而非代码审查 | 高 |
| MarkQWu/gstack | 中 | 大型 skill 套件、allowed-tools 明确 | gstack 无 dispatch shim，面向个人 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Plugin commands 缺 frontmatter（Q004） | `head -1 bureau/commands/*.md` | bureau/commands/ 下 commit.md 等命中（head 显示无 `---`）| 中 |
| Agent/skill 零示例（Q003 pattern） | `grep -L "## Example" bureau/skills/*/SKILL.md` | 待核查，预计有命中 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的 `commands/*.md` → 逐一加 YAML frontmatter（每个文件 5 分钟）

### 8.3 别人的更优方案

1. **领域**：多维度专家 agent 拆分
   - **本案例做法**：11 个 code-reviewer agent，每个专注单一维度（correctness/security-red/security-blue/test-quality 等）；`plugins/dso/agents/code-reviewer-deep-correctness.md`
   - **我的项目现状**：gstack 没有审查类 agents，bureau 的 `review.md` 命令是单一全能型
   - **如何借鉴**：bureau review 命令拆分为「功能正确性审查」「安全风险审查」两个独立流程

2. **领域**：`REVIEW-DEFENSE` 文档化有意设计
   - **本案例做法**：`scope-drift-reviewer.md` 包含 200 字说明块，解释 name 为何不加前缀
   - **我的项目现状**：bureau/gstack 中违反惯例的设计没有文字解释
   - **如何借鉴**：bureau/CLAUDE.md 加「REVIEW-DEFENSE 约定」节，在所有违反 NLPM 规范的地方加标注

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 生命周期管理
- **我的做法**：`MarkQWu/gstack` 的 SKILL.md 有 `version: 2.0.0` 字段，支持版本追踪
- **本案例做法**：DSO 的 32 个 skills 无版本字段，无法追踪变更历史
- **意义**：版本字段是 NL 工件 API 契约意识的体现，值得在 PR 中主动展示

---

## 八、术语表

### <a name="dispatch-shim"></a>分发垫片（Dispatch Shim）
> Claude Code 插件安装时自动写入宿主项目 `.claude/commands/` 目录的 7 行 Markdown 文件。这些文件不包含真正的 AI 指令，只做一件事：检测插件是否安装，然后把调用路由到插件内的真正 SKILL.md 文件。如果插件没安装，就用 `Skill tool` 降级。类比：门店门口的「这个口味今日无货，请到服务台」指示牌——不是真正的商品，只是路由标识。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块。Claude Code 读取 SKILL.md 或 agent 文件时，先解析这里声明的 `name`、`description`、`model`、`allowed-tools` 等字段。缺少 frontmatter = 这个文件对 Claude Code 的自动注册系统「不可见」。

### <a name="sentinel-state"></a>哨兵状态表
> Agent 输出中用表格列出的「已完成/未完成」布尔检查项。比文字描述更机器可读，也更容易被另一个 agent 解析。navapbc 的 `completion-verifier.md` 用这种格式验证每个工作步骤是否真正完成，防止模型过早宣布任务完成。
