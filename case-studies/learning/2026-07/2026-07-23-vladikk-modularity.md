# vladikk/modularity — 学习案例

**仓库**：https://github.com/vladikk/modularity
**Stars**：337（exemplar_published 覆盖星数门槛）| **来源**：upstream（exemplar_published，SECURITY CLEAR）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-23（基于当前 HEAD）
**主题标签**：`single-purpose`, `cross-reference`, `template-design`, `model-pinning`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

vladikk/modularity 是一个四技能 Claude Code 插件，将 Vlad Khononov 的 [Balanced Coupling](#balanced-coupling)（平衡耦合）领域模型打包为 AI 可直接调用的知识库和工作流。作者是《Balancing Coupling in Software Design》的作者，是 DDD 社区知名实践者。仓库建立目的单一：让 Claude 用这套模型做软件模块化分析、架构设计和文档生成。

关键事实：
- 4 个 SKILL.md，总计约 821 行，平均每个 205 行
- 发布在 Claude Code [marketplace](#manifest)，stars 较低但质量极高（NLPM 98/100）
- 无脚本、无 Hook、无 MCP、无 Package manifest——零可执行面
- 被 NLPM 评选为 exemplar，例示 R04/R05/R07/R08 四条规则的最佳实践

### 1.2 架构剖析

```
vladikk/modularity/
├── .claude-plugin/
│   └── plugin.json              # manifest，注册所有 4 个 skill
├── skills/
│   ├── balanced-coupling/
│   │   └── SKILL.md             # 参考 skill（知识注入用，user-invocable: false）
│   ├── design/
│   │   └── SKILL.md             # 主动 skill：高层架构设计
│   ├── document/
│   │   └── SKILL.md             # 主动 skill（依赖）：文档生成 + 链接表
│   │   └── assets/template.html # 模板文件
│   └── review/
│       └── SKILL.md             # 主动 skill：模块化审查
└── README.md
```

- **文件类型分布**：4 个 SKILL.md（1 参考 + 3 主动），1 个 plugin.json，1 个 HTML 模板
- **编排关系**：两层——`review` 和 `design` 作为用户入口，二者都在 [frontmatter](#frontmatter) 中声明 `skills: [balanced-coupling, document]`，将参考知识注入上下文；`balanced-coupling` 和 `document` 标记为 `user-invocable: false`
- **跨件契约**：`review` 和 `design` 在 body 中写明「(preloaded from the balanced-coupling skill)」，告知 Claude 该知识已在上下文，防止重复加载或找不到

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「参考 skill 作注入上下文，不作可调用命令」——一个大型领域模型只打包一次，通过 `skills:` [frontmatter](#frontmatter) 注入到所有需要它的主动 skill 中
- **解决什么问题**：把一本专业书的核心模型（Balanced Coupling 三维框架）变成 Claude 可以机械执行的查找表，而不是让作者每次都在 prompt 里重复解释
- **Trade-off**：深度知识集中在一个参考 skill 中 vs 分散在每个主动 skill 的 body 里。集中的代价：如果 `balanced-coupling` 变，所有依赖它的 skill 都受益；分散的代价：重复维护、容易漂移
- **认知模型**：作者把 Claude 当做需要「领域注入」的专家系统。他不期望 Claude 自行理解耦合理论——他预加载整个理论，让 Claude 只做机械匹配

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「参考知识注入 + 主动技能调用」（Reference Injection Architecture）**

领域知识（Reference Skill）被标记为不可用户调用，仅作为上下文注入；可调用的主动技能（Active Skill）声明依赖后即可使用该知识，且在 body 中用「(preloaded from X skill)」注释告知 Claude。

模式特征清单：
- 特征 1：知识层与动作层分离——参考 skill 只存知识，主动 skill 只描述工作流
- 特征 2：`user-invocable: false` 防止参考 skill 出现在用户的 skill 列表中造成困惑
- 特征 3：frontmatter `skills:` 声明依赖，body 注释声明已加载（双重声明）
- 特征 4：查找表替代理论解释——将抽象规则转为 Claude 可机械执行的表格/公式
- 特征 5：所有 skill body 控制在 100-330 行以内

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有一个大型领域模型（如 DDD、SOLID、Clean Architecture）需要在多个工作流中共用 | ✅ 高度适用 | 一次打包，到处注入，维护单一来源 |
| 技能非常简单，没有共享知识 | ❌ 不适用 | 过度工程；直接用一个 SKILL.md 足够 |
| 需要用户显式调用「加载知识库」 | ❌ 不适用 | 违背此模式的隐式注入设计意图 |
| 多个团队成员各自写主动 skill，共享同一参考 | ✅ 适用 | 参考 skill 成为团队契约 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：参考注入架构 | vladikk/modularity | 知识单一来源；主动 skill 简洁 | 需要所有依赖 skill 都在同一插件中发布 |
| 备选 A：CLAUDE.md 全局注入 | 大多数项目 | 所有会话都能访问 | 重量级，不适合按需加载 |
| 备选 B：每个 skill 内联知识 | 很多平铺 skill 集 | 自包含，无依赖 | 维护重复，版本漂移风险 |

### 2.4 改进空间

1. **当前问题**：`skills/design/SKILL.md` 无 `model:` 声明，但它驱动多步交互式工作流（Read/Write/AskUserQuestion/TaskCreate），实际上在充当 agent。**改进做法**：加 `model: claude-sonnet-5`（或 Opus 4.8 如需要高质量架构推理）。**预期收益**：运行时模型选择可控，审计日志可查。

2. **当前问题**：`design/SKILL.md` 中 "good Balanced Coupling decisions"（第 43 行），"good" 没有可验证标准。**改进做法**：替换为「明确列出三维（integration strength、distance、volatility）各自的二元判断标准」。**预期收益**：Claude 不再凭感觉判断"好"，而是查表得出结论。

3. **当前问题**：`review/SKILL.md` 中 "particularly dangerous"（第 62 行）无阈值定义。**改进做法**：改为「隐式耦合导致 X 种失败模式（列举），而显式耦合只导致 Y 种」，让危险程度可量化。**预期收益**：Claude 在报告中能给出可行动的分级，而不是泛化警告。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 98/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/design/SKILL.md | 93 | 无 model: 声明；"good" 模糊量词 |
| skills/review/SKILL.md | 98 | "particularly" 模糊量词 |
| .claude-plugin/plugin.json | 100 | — |
| skills/balanced-coupling/SKILL.md | 100 | — |
| skills/document/SKILL.md | 100 | — |

### 3.2 当时值得借鉴的模式

**1. R04：描述即触发**（`balanced-coupling/SKILL.md:3-11`）：10 个具体触发短语，每个匹配不同用户查询类型——「deciding whether to split or merge」和「applying DDD strategic patterns」触发条件完全不同，宽度是真实的而非填充。

**2. R05：body 长度克制**：最大的 `balanced-coupling` 的 330 行中，用布尔表达式 `MODULARITY = STRENGTH XOR DISTANCE`、`COMPLEXITY = STRENGTH AND DISTANCE` 替代数段解释性散文，压缩比极高。

**3. R07：依赖声明 + 正文注释双保险**：frontmatter `skills:` 是机器可读声明，body `(preloaded from the balanced-coupling skill)` 是 Claude 可读注释——两者都有，Claude 既知道要加载什么，也知道加载后在哪使用。

**4. R08：查找表替代理论**（`document/SKILL.md:96-119`）：每个耦合概念（「Intrusive coupling」、「Functional coupling」等）直接映射到 coupling.dev 的具体 URL，Claude 无需「理解」，直接查表即可。

**5. user-invocable: false 正确用法**：参考 skill 不出现在用户列表，只通过依赖注入；文档中从不告诉用户「去调用 balanced-coupling」。

### 3.3 当时的缺陷

**1. design/SKILL.md 无 model: 声明（-5分）**
- **根本原因**：作者可能把 skill 和 agent 看成同一类文件，没有意识到「多步+工具调用+交互」组合触发了 agent-level 行为，需要显式声明模型
- **自查**：我的 68 个 SKILL.md 中 **无一** 有 `model:` 声明，问题比这里严重得多

**2. design/SKILL.md "good" 模糊量词（-2分）**
- **根本原因**：作者在讲一个高度具体的模型，但在描述「如何做好决策」时退回了日常语言。说明即使领域专家也会在指令写作时滑入模糊表达
- **自查**：我的 SKILL.md 描述中有类似问题，`gstack` 里多个 skill 用了 "good"、"appropriate" 等

**3. review/SKILL.md "particularly dangerous" 无阈值（-2分）**
- **根本原因**：「危险」是相对判断，没有基准线就无法量化。作者知道它危险，但 Claude 需要知道「比什么更危险，危险到什么程度才要干预」
- **自查**：类似问题在我的 `bureau/skills/review/SKILL.md` 中很可能存在

### 3.4 当时的优化机会

1. 给 `design/SKILL.md` 加 `model: claude-sonnet-5`（或当时的 Sonnet 版本）
2. 将 "good Balanced Coupling decisions" 替换为三维检查清单
3. 将 "particularly dangerous" 替换为可数的失败模式列举

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| design/SKILL.md 无 model: 声明 | `grep "^model:" skills/design/SKILL.md` | **持续**（零结果） | PR 可能未被采纳，或作者未意识到此问题 |
| design/SKILL.md "good" 模糊 | `grep '\bgood\b' skills/design/SKILL.md` | **持续**（第 43 行仍有） | 三项质量问题全部未修复 |
| review/SKILL.md "particularly" 模糊 | `grep '\bparticularly\b' skills/review/SKILL.md` | **持续**（第 62 行仍有） | 同上 |

三项质量问题全部持续，说明 NLPM 的 PR 如果有提交，未被接受；或者这些是作者的审美选择。

### 4.2 架构演进

当前 HEAD（commit `bcdca9a`）与审计时相比，目录结构**无变化**：仍是 4 个 SKILL.md + 1 个 plugin.json + 1 个 assets/template.html。没有新增 skill、没有拆分目录、没有新增 Hook。

这意味着：作者认为这个四技能设计已经足够，仓库进入了维护而非扩展阶段。稳定本身也是一个信号——高质量设计不需要频繁重构。

### 4.3 新增的可学习模式

暂无——当前 HEAD 与审计时状态一致，未发现新增设计。

---

## 五、校准

### 5.1 我已经在做对的

1. **references/ 子目录**：我的 `MarkQWu/graphify` 和 `MarkQWu/drama-workshop-skills` 都用了 `references/` 子目录存放深度参考内容，与 ooiyeefei/ccc（同批次案例）中的 exemplar 模式一致
2. **触发短语列表**：我的 `gstack/spec/SKILL.md` 用了 `triggers:` 列表（6 个具体短语），与 R04 方向一致，虽然不是 `description:` 字段直接写入
3. **body 长度克制**：`bureau` 的 skill 文件普遍较短

### 5.2 挑战 / 验证

**这次案例验证了一个我之前模糊感受到的事情**：`user-invocable: false` 不只是个配置项，它是「知识库 vs 工具库」的架构分界线。我的 `bureau` 里的 `capture`、`lint` 等都应该被审视：哪些是用户会直接叫的，哪些只应该被其他 skill 调用？目前我的项目里**没有任何** skill 标记为 `user-invocable: false`，说明我还没有意识到这个分层设计。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中 model: 声明缺失情况
find /tmp/my-repos -name "SKILL.md" | xargs grep -L "^model:" 2>/dev/null | wc -l
# 命中后怎么办：对有多步交互、AskUserQuestion、TaskCreate 的 skill 加 model: 声明
```

```bash
# 检查我的 skill 中模糊量词
grep -rn -E '\b(good|appropriate|relevant|particularly|comprehensive)\b' /tmp/my-repos/*/*/SKILL.md 2>/dev/null | grep -v "^Binary"
# 命中后怎么办：逐条替换为可验证的具体标准
```

```bash
# 检查我的 skill 是否有 user-invocable: false 的候选
find /tmp/my-repos -name "SKILL.md" | xargs grep -l "skills:" 2>/dev/null
# 命中后怎么办：这些 skill 如果被其他 skill 依赖，考虑标记为 user-invocable: false
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `MarkQWu/bureau` 的 `balanced-coupling` 类似场景——把 `bureau` 的核心信任模型（trust tier 定义）独立成一个参考 skill，注入到 `capture`、`review`、`compile` 中
   - **为何可行**：`bureau` 里多个 skill 都引用 trust tier 概念，目前每个 skill 都重复写，维护成本高
   - **第一步**：创建 `bureau/skills/trust-model/SKILL.md`，标记 `user-invocable: false`，将 canonical/unverified/logbook 三级定义移入；在 `capture`、`review` frontmatter 中加 `skills: [trust-model]`

2. **想法**：在 `MarkQWu/gstack` 的 design-相关 skill 上加 `model:` 声明
   - **为何可行**：`design-review`、`design-shotgun`、`design-html` 都是多步交互工作流，不加 model 会用默认模型执行
   - **第一步**：`grep -l "AskUserQuestion" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` 找出所有需要模型声明的候选文件，批量加 `model: claude-sonnet-5`

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 vladikk/modularity 的核心目的**：把专业领域模型（Balanced Coupling）打包成可注入的 AI 参考知识，通过多 skill 协作实现设计→审查→文档的完整工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 多 skill 协作，有参考知识（trust tier model） | bureau 面向会话记忆，modularity 面向代码设计 | 高 |
| MarkQWu/gstack | 低 | 有设计相关 skill（design-review, design-html） | gstack 是全套工具集，不是单领域模型注入 | 中 |
| MarkQWu/graphify | 无 | — | graphify 是代码知识图谱，与模块化设计不直接相关 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 无 model: 声明 | `find /tmp/my-repos -name "SKILL.md" \| xargs grep -L "^model:"` | **命中 68/68**（全部）gstack 59 个，bureau 7 个，drama 2 个 | 高 |
| 模糊量词 "good/appropriate/relevant" | `grep -rn -E '\b(good\|appropriate\|relevant)\b' /tmp/my-repos` | 多处命中（bureau/skills/capture 等） | 中 |
| 无 user-invocable: false 分层 | `grep -rl "user-invocable: false" /tmp/my-repos` | 零命中——我完全没用过此模式 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack/design-review/SKILL.md` → 加一行 `model: claude-sonnet-5` → 1 分钟
- `MarkQWu/bureau/skills/review/SKILL.md` → 同上 → 1 分钟
- 批量处理：`find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" | xargs grep -l "AskUserQuestion"` → 这些都需要 model: 声明

### 7.3 别人的更优方案

1. **领域：参考 skill 注入架构**
   - **本案例做法**：`skills/balanced-coupling/SKILL.md` 标记 `user-invocable: false`，被 `skills/review/SKILL.md` 和 `skills/design/SKILL.md` 通过 `skills:` frontmatter 引用；body 中用 `(preloaded from the balanced-coupling skill)` 注释
   - **我的项目现状**：`MarkQWu/bureau` 里 trust model 分散在各 skill 的 body 中重复定义，无独立参考 skill
   - **如何借鉴**：创建 `bureau/skills/trust-model/SKILL.md`（`user-invocable: false`），将 trust tier 定义集中，然后在 capture/review/compile 的 frontmatter 加 `skills: [trust-model]`

2. **领域：布尔公式压缩领域规则**
   - **本案例做法**：`BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY`（2 行替代数段散文）
   - **我的项目现状**：`bureau` 的 review/SKILL.md 用散文描述 trust tier 判断规则，读起来费力
   - **如何借鉴**：把 trust tier 晋升逻辑也写成类似的逻辑表达式（`CANONICAL = HUMAN_APPROVED AND NOT CONTRADICTED`）

### 7.4 反向：我的项目做得比他们好的地方

- **领域：触发短语写法**：我的 `gstack/spec/SKILL.md` 用了专门的 `triggers:` frontmatter 字段（非 description 的附属），比 vladikk 把触发短语嵌入 `description:` 文本更机器可读
- **意义**：若 Claude Code 未来支持独立的 `triggers:` 字段，我的格式更面向未来；若被人审计时，这是一个可以讨论的亮点

---

## 八、术语表

### <a name="balanced-coupling"></a>Balanced Coupling（平衡耦合）
> Vlad Khononov 提出的软件设计框架，认为模块间的耦合不是越少越好，而是应该在三个维度（集成强度 Integration Strength、模块距离 Distance、波动性 Volatility）之间保持平衡。关键公式：`BALANCE = (STRENGTH XOR DISTANCE) OR NOT VOLATILITY`。简单说：要么强耦合但距离近，要么弱耦合但距离远——两个高的情况才是危险的。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（如 `name`、`description`、`model`、`skills:`）。Claude Code 读 SKILL.md 时先解析 frontmatter，再读 body。

### <a name="manifest"></a>manifest
> 项目的清单文件，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——列出所有 skills 的路径。manifest 里漏写某个文件，该文件即使在硬盘上也不会被加载。
