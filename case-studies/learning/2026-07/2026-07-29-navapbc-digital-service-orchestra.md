# navapbc/digital-service-orchestra — 学习案例

**仓库**：https://github.com/navapbc/digital-service-orchestra
**Stars**：3 | **来源**：upstream audit
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-07-29（基于当前 HEAD）
**主题标签**：`router-channels`, `cross-reference`, `examples-driven`, `manifest-discipline`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Digital Service Orchestra（DSO）是 **Nava PBC**（美国政府数字服务咨询公司）为内部工程流程开发的 Claude Code 工作流插件，当前版本 v1.16.11。它把整个软件开发生命周期（需求分析 → 架构 → 实现 → 代码审查 → 提交 → 会话结束）封装成一套由 63 个 [agent](#agent) 驱动的协调式工作流。

关键事实：
- 作者背景：Nava PBC 是专为美国联邦政府提供数字化转型服务的咨询公司，代码质量标准极高
- 安装：通过 `marketplace.json` 发布两个渠道——`dso`（稳定版，指向 v1.16.11 tag）和 `dso-dev`（开发版，指向 main）
- 规模：63 个 agent + 39 个 SKILL.md + 4 个 plugin 命令 + 完整的 `tests/` 套件（含 fixture 和 drift injection 测试）
- GitHub Instructions：有 `.github/instructions/` 目录支持 GitHub Copilot，双轨兼容

### 1.2 架构剖析
```
digital-service-orchestra/
├── CLAUDE.md              ← 项目使用手册（含完整命令速查表）
├── plugins/dso/
│   ├── .claude-plugin/plugin.json  ← 插件注册表
│   ├── agents/            ← 63 个专用 agent（代码审查/规划/验证/诊断等）
│   ├── skills/            ← 39 个 SKILL.md
│   ├── commands/          ← 4 个核心命令（commit、review、end、fp-recovery）
│   ├── docs/workflows/    ← 工作流编排文档（phase 文件）
│   └── scripts/dso        ← ticket CLI 工具（./scripts/dso ticket list 等）
├── .claude/commands/      ← 23 个 dispatch shim（自动生成）
├── tests/
│   ├── drift_injection/   ← 漂移注入测试
│   └── fixtures/          ← 大量 JSON/MD fixture（coverage-review、classifier 等）
└── .github/instructions/  ← GitHub Copilot 兼容指令（8 个文件）
```

- **文件类型分布**：63 个 agent / 39 个 SKILL.md / 4 个 plugin 命令 / 23 个 [dispatch shim](#dispatch-shim)
- **编排关系**：多层架构——dispatch shim 路由 → plugin 命令 → agent 执行；agent 之间通过结构化 JSON 传递状态（RESULT schema）
- **跨件契约**：agent 之间通过严格的 JSON schema 通信（如 `RESULT`、`finding`、`verdict` 对象）；`scripts/dso` CLI 是 ticket 系统和 workflow 的唯一接触点

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「协议驱动的多 agent 编排」——每个 agent 只做一件事，通过标准化 JSON schema 和下一个 agent 通信，没有散文式的"把结果告诉我"
- **解决什么问题**：政府数字项目的代码审查需要多视角（安全/正确性/性能/架构）；单一 agent 容易有死角；DSO 把审查拆分成 11 个专用 reviewer agent
- **Trade-off**：精确的 agent 分工换来了高维护成本——63 个 agent 的 schema 要保持同步
- **认知模型**：把 AI 工作流当成「管道」（pipeline）而不是「对话」——每个 agent 是一个处理节点，输入和输出都是机器可读的 JSON

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「Router + 多维度专用 agent 矩阵」**：dispatch shim 作为路由器，把用户请求路由到对应的 plugin 命令；plugin 命令再协调多个专用 agent 并行工作；每个 agent 通过 JSON schema 输出标准化结果。

模式特征（5 条）：
- **特征 1**：审查任务被拆分为 11 个专用 reviewer（架构/正确性/安全红队/安全蓝队/性能/测试等），并行执行
- **特征 2**：JSON schema 驱动——agent 的 RESULT 是 JSON，不是自然语言叙述，机器可处理
- **特征 3**：Dispatch shim 与 plugin 命令分离——shim 是安装侧产物（低 NL 分），plugin 命令是 NL artifacts（高 NL 分）
- **特征 4**：`tests/fixtures/` 丰富——有 convergence fixtures、drift injection、LLM review replay，说明在 NL artifact 层面做了测试
- **特征 5**：双渠道发布（稳定版 + 开发版），通过 git tag 而不是 npm 版本控制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多视角代码审查的工程团队 | ✅ 极度适用 | 11 个 reviewer 覆盖了几乎所有代码质量维度 |
| 个人开发者 | ❌ 过度工程 | 63 个 agent 的维护成本超过个人价值 |
| 需要合规审计的政府/企业项目 | ✅ 高度适用 | JSON schema 输出使审计日志结构化 |
| 需要快速 MVP 的初创公司 | ❌ 不适用 | 流程过重，与快速迭代的需要冲突 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Router + agent 矩阵（本仓库） | navapbc/dso | 多视角并行；JSON 可机器处理 | 体积大；维护成本高 |
| 大型单一命令集 | c0x12c/ai-toolkit | 覆盖广；易发现 | 无并行；agent 专业化不足 |
| 极简 skill 集 | vladikk/modularity | 零维护；高质量 | 无编排；无多视角 |

### 2.4 改进空间
1. **当前问题**：dispatch shim 命令（23 个）无 frontmatter，NLPM 得分 15/100，拉低全局均分 **改进做法**：为 shim 模板添加最小 3 行 frontmatter（`name`、`description: Dispatch shim`、`---`） **预期收益**：全局均分从 66.8 提升至约 78
2. **当前问题**：32 个 SKILL.md 缺 `model:` 声明（已有 rule override 提议，但未落地） **改进做法**：在 `.claude/nlpm.local.md` 里添加 `NL-MISSING-MODEL` 的 scope 排除，承认这是有意为之 **预期收益**：消除 -160 分的聚合质量影响，不误导 NLPM 审计报告
3. **当前问题**：`bot-psychologist.md` 的 description 说"17-point taxonomy"但 body 说"16 failure modes" **改进做法**：统一为"16-point"或者真正把 taxonomy 扩展到 17 条，并更新 schema 注释 **预期收益**：消除 schema 消费者的混淆

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-27 当时得分 **84.8/100**（plugin 文件单独评分）；全部 artifact 含 shim 时为 **66.8/100**。

| 类别 | 当时分数 | 主要问题 |
|---|---|---|
| Agents 均值 | 90.8 | 11 个 reviewer 各少 1 个示例（-5 各）；3 个 huge-diff agent 自相矛盾的 REPO_ROOT 指导（-3 各） |
| Dispatch shims | 15.0 | 无 frontmatter（设计如此，但拉低了总分） |
| Plugin commands | 约 20 | commit/review/end 缺 frontmatter |
| Skills | N/A | 缺 `model:` 声明（有意为之） |

### 3.2 当时值得借鉴的模式
1. **JSON schema 通信协议** → agent 之间用 `RESULT` JSON schema 传递结果，而不是散文 → 根本原因：JSON 是机器可读的，可以做验证、过滤、聚合，散文不行 → 借鉴：当我的项目有多 agent 协作时，定义一个 JSON schema，不要让 agent 直接输出自然语言给下一个 agent
2. **11 维度并行审查** → code-reviewer-deep-arch、code-reviewer-deep-correctness、code-reviewer-security-red-team 等 11 个专用 reviewer 并行执行 → 根本原因：不同维度的问题需要不同的思考框架；并行比串行快，且不会互相干扰 → 借鉴：审查类任务考虑拆成「安全维度」「架构维度」「性能维度」3 个并行 agent
3. **Dispatch shim 分离** → shim 是安装侧的路由代码，不是 NL artifact；把它们明确区分，避免混入 NL 质量评估 → 根本原因：基础设施代码和业务逻辑应该用不同标准评估 → 借鉴：如果我的项目有自动生成的文件，在 NLPM scope 里排除它们
4. **测试 fixtures** → `tests/fixtures/` 有真实的输入/输出对（classifier、convergence、drift），说明作者在测试 NL artifact 的行为 → 借鉴：学习 NL-TDD 模式，为重要的 skill/agent 写 fixture 测试

### 3.3 当时的缺陷
1. **11 个 reviewer agent 各只有 1 个示例（-5/agent）** → 根本原因：作者统一生成这些 agent（可能用脚本），只加了最小示例；没有为"2 个形成对比示例"的质量要求做批量处理 → 自查：如果我用脚本批量生成 skill/agent，是否也会因为"最小满足"而错过对比示例？
2. **`bot-psychologist` taxonomy 计数不一致** → 历史上是 16，但 description 里写成了其他数字（audit 时 description 正确但 body 有误；现在 description 说 17 但 body 说 16） → 根本原因：schema 注释和描述文字是分开维护的，改了一处没有联动更新 → 自查：我的 agent/skill 里如果有"N 个规则"这样的数字声明，是否和实际内容数量对齐？
3. **Plugin commands 缺 frontmatter**（commit.md、review.md、end.md）→ 根本原因：这些命令有功能性的步骤描述，但作者没有意识到 NLPM 要求 frontmatter → 这让本来是 80/100 的命令降到了 20/100

### 3.4 当时的优化机会
1. 为 11 个 reviewer agent 批量添加第二个对比示例（可以用脚本）
2. 统一 `bot-psychologist` 的 taxonomy 数字
3. 为 `commit.md`、`review.md`、`end.md` 添加 frontmatter

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| bot-psychologist taxonomy 不一致 | `grep "taxonomy\|failure" plugins/dso/agents/bot-psychologist.md` | **持续且演变**：description 现在说 "17-point" 但 body 说 "16 failure modes" | 不一致从「内部矛盾」变成了「版本迁移未完成」 |
| 11 reviewer 各只有 1 个示例 | 查看 agent body | **持续**（未加第二个示例） | -5 × 11 = -55 分仍未修 |
| Plugin commands 缺 frontmatter | `grep "^name:" plugins/dso/commands/*.md` | 需进一步验证 | 待查 |

### 4.2 架构演进
**最重要的变化**：agent 数量从 31 → 63（翻倍！），skill 从 ~25 → 39。这说明：
- 作者大幅扩张了 agent 矩阵，新增了更多专用审查 agent
- plugin 版本从 audit 时的早期版本迭代到 v1.16.11
- `.github/instructions/` 新增，开始双轨支持 Claude Code 和 GitHub Copilot
- `tests/fixtures/` 大幅扩充（新增 sprint-trailer-enforcement、818-corpus 等）
- `scripts/dso` 从简单脚本演进为完整 ticket CLI

作者后来意识到了什么：单纯的代码审查 agent 不够，需要扩展到规划、验证、范围管理等更多维度。

### 4.3 新增的可学习模式
- **`tests/drift_injection/`**：故意注入漂移测试——向系统注入一个错误，然后验证 agent 能否检测到。这是 NL artifact 测试的高级模式，比一般的 fixture 测试更主动
- **`.github/instructions/` 双轨**：同一套知识同时为 Claude Code 和 GitHub Copilot 提供服务，通过不同格式（SKILL.md vs `.instructions.md`）分发
- **`rulesets/` 目录**：存储 GitHub 仓库规则集的快照（before 状态），便于回滚和审计——这是版本控制扩展到配置层面的例子

---

## 五、校准

### 5.1 我已经在做对的
1. 我的项目规模适当——没有尝试过早扩张到 63 个 agent，保持了适合当前阶段的规模
2. 我注意到了 agent 的 `name:` 字段的重要性（从 kangraemin 案例学到的），DSO 的所有 agent 都有正确的 name 声明
3. 我认可「专用 agent 优于万能 agent」的原则，DSO 验证了这一点

### 5.2 挑战 / 验证
- **验证**：「dispatch shim 不是 NL artifact」这个区分很重要——DSO 的做法证明了「不是所有 .claude/ 里的文件都应该被当成 NL artifact 打分」。如果我的仓库有自动生成的命令文件，应该在 NLPM config 里排除它们。
- **挑战**：从 31 → 63 个 agent 的扩张说明「架构可以持续演进」，但每次扩张时都需要同步更新所有 schema 和 cross-reference。本案例的 bot-psychologist taxonomy 不一致就是扩张时没有做好联动更新的代价。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库里有没有"N 个规则"这样的数字与实际内容不一致的情况
grep -rn "[0-9]\+-point\|[0-9]\+ rules\|[0-9]\+ steps" /tmp/my-repos/MarkQWu-*/ --include="*.md" 2>/dev/null | head -10

# 检查我的项目有没有自动生成的 NL 文件（需要排除出 NLPM 评分）
find /tmp/my-repos/MarkQWu-*/ -name "*.md" | xargs grep -l "Auto-generated\|auto-generated\|GENERATED" 2>/dev/null

# 检查 bureau 的命令是否有 dispatch shim 类的文件
ls /tmp/my-repos/MarkQWu-bureau/commands/ 2>/dev/null
head -5 /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null | head -30
```

命中后怎么办：
- 发现数字声明与实际不一致 → 立即修复，并考虑用脚本检查（`grep "N 个" *.md | wc -l` 对比实际数量）
- 发现自动生成文件 → 在 `.claude/nlpm.local.md` 里添加 scope 排除规则

### 6.2 灵感 → 实施路径
1. **想法**：参考 DSO 的 JSON schema 通信模式，为 bureau 的 capture→compile→review pipeline 定义 JSON 结构化输出
   - **为何可行**：DSO 验证了 JSON schema 可以让多 agent 系统更可靠；bureau 的 pipeline 也有类似需求
   - **第一步**：为 bureau 的 compile 阶段定义一个 `entry` JSON schema（5 个字段），并在 capture agent 的 output format 里引用它（30 分钟）

2. **想法**：学习 DSO 的「drift injection 测试」，为 MarkQWu/- 的 SKILL.md 写一个「注入错误情境 → 验证 skill 能正确识别」的测试
   - **为何可行**：drift injection 是比 fixture 更强的测试——它主动创造边界条件
   - **第一步**：为 `track-unrest-events/SKILL.md` 写一个「当 API 返回 404 时，skill 应该怎么处理」的测试 fixture（20 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例的核心目的**：为工程团队提供 Claude Code 驱动的完整 SDLC 工作流（从需求到部署）
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是 Claude Code 插件，用于管理工程流程；都有 capture→review 类工作流 | bureau 专注知识管理，DSO 专注代码审查 | 高（JSON schema 通信、dispatch shim 分离模式直接可借鉴） |
| MarkQWu/gstack | 中 | 都是多 skill 的工程工具集 | gstack 是单人工具集；DSO 是团队协作框架 | 中（并行 agent 模式可借鉴） |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Taxonomy 数字不一致 | `grep -rn "[0-9]\+-point\|[0-9]\+ rules" MarkQWu-*/ --include="*.md"` | **未发现**（我的项目无此类声明） | 无 |
| Dispatch shim 拉低总分 | 检查是否有自动生成命令 | **未发现**（bureau 无自动生成 shim） | 无 |
| Agent 示例只有 1 个 | 查看 agent body | **未命中**（我无 agent 文件） | 无 |

**命中后的具体行动建议**：暂无命中。本案例对我的主要学习价值在于架构模式（JSON schema、并行 agent）而不是缺陷修复。

### 8.3 别人的更优方案

1. **领域**：多 agent 并行审查 + JSON schema 通信
   - **本案例做法**：11 个 reviewer agent 并行运行，每个输出 `{"findings": [...], "verdict": "triggered/not-triggered"}` JSON
   - **我的项目现状**：bureau 的 review 阶段是单一 agent 串行执行，输出自然语言
   - **如何借鉴**：为 bureau 的 review 阶段拆出 2-3 个专用 reviewer（信息准确性、引用完整性、版本冲突），各自输出 JSON → merge 后决策（3-5 小时）

2. **领域**：`tests/drift_injection/` 漂移注入测试
   - **本案例做法**：故意向系统注入错误，验证 agent 能否检测到——比被动 fixture 测试更主动
   - **我的项目现状**：无任何 NL artifact 测试
   - **如何借鉴**：为 MarkQWu/- 的关键 SKILL.md 各写一个「错误输入 → 预期检测」的测试用例，存为 `tests/fixtures/` （30-60 分钟起步）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：dispatch shim 与 plugin 命令的分离意识
- **我的做法**：bureau 的命令都是手写的 NL artifacts，没有自动生成的 shim 文件
- **本案例弱点**：23 个 dispatch shim 因为 15/100 的评分拉低了整体 NL 质量指标，混淆了「真实 NL 质量」和「基础设施代码」的界限
- **意义**：如果未来 bureau 引入自动生成文件，要立即在 NLPM config 里做 scope 排除，保持 NL 质量指标的纯粹性

---

## 八、术语表

### <a name="agent"></a>agent
> Claude Code 里的「专用子代理」。在 `.claude/agents/` 目录下用 Markdown 写成，有固定的专长领域。Claude 可以把子任务派给 agent 执行——就像把一个具体问题交给一个专业顾问。`name:` 字段是必填的注册 key。

### <a name="dispatch-shim"></a>dispatch shim
> 「路由转发文件」。一段极短的 Markdown 代码（通常 5-7 行），内容是检测插件是否安装，然后把请求转发到真正的 skill 或命令文件。它本身没有业务逻辑，只是「中间人」。因为没有 frontmatter，被 NLPM 打低分，但这是设计选择，不是质量问题。

### <a name="json-schema"></a>JSON Schema 通信
> agent 之间约定用 JSON 格式传递结果，而不是用自然语言。例如审查 agent 输出 `{"findings": [{"file": "main.py", "line": 42, "severity": "high"}]}`，下游 agent 直接解析这个 JSON 做决策。比散文更精确，更容易验证，也更容易被脚本处理。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（`name`、`description`、`model`、`allowed-tools` 等）。Claude Code 先读这里才知道这个 skill/agent/command 是谁、能干什么。

### <a name="sdlc"></a>SDLC
> Software Development Life Cycle，「软件开发生命周期」。从需求分析 → 架构设计 → 编码实现 → 测试 → 部署 → 维护的完整循环。DSO 的目标是把这个完整循环都纳入 Claude Code 的工作流。
