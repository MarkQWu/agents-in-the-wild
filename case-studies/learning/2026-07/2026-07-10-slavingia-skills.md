# slavingia/skills — 学习案例

**仓库**：https://github.com/slavingia/skills
**Stars**：registry 未记录（作者 Sahil Lavingia 为 Gumroad 创始人，预计可观）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-07-10（基于当前 HEAD）
**主题标签**：`single-purpose`, `manifest-discipline`, `vague-quantifier`, `template-design`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[slavingia/skills](https://github.com/slavingia/skills) 是 Sahil Lavingia（Gumroad 创始人、《The Minimalist Entrepreneur》作者）将其书中核心商业框架封装进 Claude Code 的 skill 套件。10 个 skill 文件 + 1 个 [manifest](#manifest) 构成一套从「找到社群」到「可持续增长」的完整创业指导工作流，用户安装后可直接在 Claude Code 对话中调用这些商业咨询框架。

关键事实：
- 作者背景：Sahil Lavingia 是知名连续创业者，《极简创业者》一书在创业社区广受传播
- 获取方式：通过 Claude Code 插件市场安装（`claude plugin install minimalist-entrepreneur@slavingia`）
- 生态定位：「专业知识 skill 化」的典型代表——把领域专家的方法论转化为 AI 可调用的结构化指令
- 当前版本：1.0.0（自 audit 以来版本未变）

### 1.2 架构剖析

```
slavingia/skills/
├── .claude-plugin/
│   ├── plugin.json          ← 插件 manifest（无 skills 注册字段！）
│   └── marketplace.json
├── skills/
│   ├── find-community/SKILL.md    ← 第 1 步：找到所在社群
│   ├── validate-idea/SKILL.md     ← 第 2 步：验证想法
│   ├── processize/SKILL.md        ← 第 3 步：流程化
│   ├── mvp/SKILL.md               ← 第 4 步：构建 MVP
│   ├── first-customers/SKILL.md   ← 第 5 步：获取首批客户
│   ├── pricing/SKILL.md           ← 第 6 步：定价
│   ├── marketing-plan/SKILL.md    ← 第 7 步：营销计划
│   ├── grow-sustainably/SKILL.md  ← 第 8 步：可持续增长
│   ├── company-values/SKILL.md    ← 第 9 步：公司价值观
│   └── minimalist-review/SKILL.md ← 横切关注点：全程复查
└── README.md
```

- **文件类型分布**：10 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：10 个 skill 平铺，没有 router 或 meta skill。`find-community` 结尾有 `Go back to /find-community` 反向引用，形成循环锚点，但没有中央调度层
- **跨件契约**：各 SKILL.md 均有 `name` + `description` frontmatter 且包含 `## Output` section，一致性极高；`processize` 在步骤说明中引用 `/find-community`，是唯一的跨 skill 显式引用

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「书即 skill」——将一本已经过读者验证的商业方法论直接转录为 AI 可执行的 skill，无需重新发明框架
- **解决什么问题**：创业者与 Claude 对话时缺少领域结构——对话变成「通用建议」而非「基于具体方法论的系统指导」。skill 套件强制 Claude 在这套框架的语境内工作
- **做了什么 trade-off**：
  - 无 commands / agents，上手极简，但失去「一键触发完整流程」的能力
  - 每个 skill 独立，用户需手动从「第一步」导航到「第二步」
  - `minimalist-review` 设计为横切关注点而非线性步骤，在不改变工作流的前提下添加了品质门控
- **反映什么认知模型**：作者把 skill 当作「知识的 API」，不是「任务的自动化」。用户是主动的决策者，Claude 是框架的执行者，而不是自主代理

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「领域书籍 → 线性 skill 链」**：单一主题的专业知识按逻辑顺序拆解为多个独立 skill，通过 skill description 中的情境触发词（「Use when someone is ready to build their first product」）引导用户在正确时机调用正确 skill。

模式特征清单：
- 特征 1：所有 skill 共享同一哲学前提（「Be the Minimalist Entrepreneur」），一致性靠描述而非代码约束
- 特征 2：无 command / agent 层，用户交互完全通过自然语言触发 skill
- 特征 3：skill description 是触发入口，`Use when...` 短语精确描述调用场景
- 特征 4：`minimalist-review` 作为「无状态的横切 skill」，任何阶段都可调用，不与线性流绑定
- 特征 5：无共享 partial / 模板，每个 skill 自包含，修改互不干扰

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 专业领域知识 skill 化（创业、法律、设计等） | ✅ 高度适用 | 知识本身有内在顺序，用户需要结构化引导 |
| 书 / 课程的数字化延伸 | ✅ 高度适用 | 作者天然是内容权威，用户信任度高 |
| 需要自动编排多步工作流的场景 | ❌ 不适用 | 没有 command 层，无法「一键启动全流程」 |
| 需要系统级工具调用（文件读写、API 请求）的场景 | ❌ 不适用 | 纯 NL skill，没有 allowed-tools 声明 |
| 多用户协作或有状态跟踪需求的场景 | ❌ 不适用 | skill 无状态，每次对话独立 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：线性 skill 链 | slavingia/skills | 极低认知负担，知识点清晰 | 无自动编排，用户需手动导航 |
| Command + Skill 分层 | openai/codex-plugin-cc | 一条命令触发完整流程 | 需要维护两层文件 |
| Router + Multi-Agent | xiaolai/grill-for-claude | 并行分析，结果结构化 | 复杂度高，适合审查类任务 |

### 2.4 改进空间

1. **当前问题**：10 个 skill 没有「入口导引」command，用户不知道从哪里开始。**改进做法**：添加一个 `/minimalist-start` command，询问用户的创业阶段，路由到对应 skill。**预期收益**：首次用户体验从「搜索哪个 skill？」变为「告诉我你在哪个阶段」
2. **当前问题**：plugin.json 没有 `skills` 注册字段（见 §3.3），依赖 Claude Code 约定扫描。**改进做法**：添加显式 `skills` 数组列出所有 10 个 skill 路径。**预期收益**：消除因加载器差异导致的 skill 丢失风险
3. **当前问题**：`pricing/SKILL.md` 中「20-50% is typical」和「feels natural」是未校准的模糊量词。**改进做法**：替换为「Gumroad 数据：SaaS 工具 pricing 集中在 $9–$49/月，高杠杆知识产品常见 $500–$2000/lifetime」。**预期收益**：Claude 给出的建议从直觉性变为数据锚定

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-07 当时得分 **97/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude-plugin/plugin.json` | 88 | 缺少 `skills` 枚举字段 (-12) |
| `skills/marketing-plan/SKILL.md` | 96 | 模糊指令「Be authentic」 (-2×2) |
| `skills/pricing/SKILL.md` | 96 | 模糊量词「typical」「feels natural」(-2×2) |
| `skills/company-values/SKILL.md` | 98 | 「regularly」无节奏定义 (-2) |
| 其余 7 个 SKILL.md | 98 | 整洁 |

### 3.2 当时值得借鉴的模式

1. **完整的 frontmatter 一致性** → 所有 10 个 SKILL.md 都有 `name` + `description`，且都包含 `## Output` section。没有例外，没有「快速凑数」的文件 → 这说明作者在动笔前就设定了统一模板，而不是每个 skill 独立发挥。**借鉴**：写 skill 前先定模板，写完后全量检查 `## Output` 存在性

2. **Skill description 用「Use when...」精确定义调用场景** → `mvp/SKILL.md`: "Use when someone is ready to build their first product or struggling with scope." 这是可机器验证的触发条件，而不是「帮助用户构建 MVP」这种模糊描述。**借鉴**：每个 skill description 必须包含具体情境短语

3. **`minimalist-review` 的横切设计** → 单独一个 skill 负责「任何阶段都可调用的品质检查」，避免在其他 9 个 skill 里重复复查逻辑。**借鉴**：识别横切关注点（日志、复查、校验），用独立 skill 封装而非散布在各 skill 里

4. **线性工作流通过 `find-community` 的反向引用形成闭环** → `processize/SKILL.md` 结尾有 `Go back to /find-community`。这是 skill 间协议化引用的最小实现，不需要 router。**借鉴**：在 skill 末尾添加「下一步」或「返回」指针，帮助用户导航

### 3.3 当时的缺陷

1. **`plugin.json` 缺少 `skills` 注册字段** → **根本原因**：作者依赖 Claude Code「约定扫描所有 SKILL.md」的行为，没有显式声明。但 manifest 的语义是「权威清单」——如果加载器在某个版本变更了约定，所有 10 个 skill 将静默消失而没有任何报错。**自查**：我的仓库（bureau、echo-sleuth）plugin.json 也没有 `skills` 字段——**共同踩坑**

2. **`pricing/SKILL.md` 中的模糊量词「typical」「feels natural」** → **根本原因**：定价本来就主观，作者可能觉得给具体数字会「限制场景」。但「感觉自然」让 Claude 的输出完全依赖训练数据而非 skill 中的业务知识，降低了 skill 的信息量。**自查**：`bureau/skills/scribe` 里有「appropriate length」，同类问题

3. **`marketing-plan/SKILL.md` 的「Be authentic」** → **根本原因**：这是书中语气的延续——书面语鼓励真诚，但 skill 里的「authentic」无法被 Claude 操作化。正确做法是「Share real CAC/conversion metrics, be explicit about past failures」。**自查**：我的 drama-workshop-skills 里有「create compelling content」，同类问题

### 3.4 当时的优化机会（学习材料，不用于 PR）

1. `plugin.json` 加 `skills` 字段，显式枚举 10 个 skill 路径（高优先级，影响可靠性）
2. `pricing` skill 里的「typical」替换为 Gumroad / Indie Hackers 的真实区间数据
3. `marketing-plan` 里的「authentic」替换为具体行动项（分享真实数据、承认失败案例）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 缺少 `skills` 字段 | `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); print('skills' in d)"` | **仍存在**（输出 False） | 时隔 3 个月，这个 Bug 未被修复——说明约定扫描目前工作良好，作者没有感受到痛点 |
| pricing「typical」「feels natural」 | `grep -n "typical\|feels natural" skills/pricing/SKILL.md` | **仍存在** | 作者认为这是书的语气而非 bug，或未关注 NLPM audit |
| marketing-plan「Be authentic」 | `grep -n "authentic" skills/marketing-plan/SKILL.md` | **仍存在** | 同上 |

### 4.2 架构演进

Audit（2026-04-07）至今（2026-07-10）：目录结构无任何变化，版本停留在 1.0.0。没有新增 skill，没有重组，没有 command 层。这说明：

> 作者把这套 skill 套件当作「书的静态延伸」，而非持续演进的软件产品。这是一个合理的选择——《极简创业者》的方法论本身不需要频繁迭代。

### 4.3 新增的可学习模式

暂无——当前 HEAD 与 audit 状态完全一致。

---

## 五、校准

### 5.1 我已经在做对的

1. **SKILL.md 都有 `## Output` section**：我的 `bureau/skills/recall/SKILL.md`、`echo-sleuth` 的所有 skill 都包含输出格式声明，这与 slavingia 的最佳实践一致
2. **skill description 写法**：我的 `bureau` skills 用「Use when the user wants to recall past decisions」这类情境短语，与 slavingia 的 `Use when...` 模式对齐
3. **单职责 skill 设计**：我的各 skill 职责清晰不重叠，没有「万能 skill」

### 5.2 挑战 / 验证

这次案例验证了一个我之前犹豫的做法：**「10 个 skill 可以没有 command 入口」**——前提是 skill description 写得足够精确，让 Claude 能在对话中自动识别触发时机。slavingia 用 97/100 的分数证明「纯 skill + 精确 description」是完整可行的架构，不一定非要加 command 层。

但同时挑战了另一个假设：**「manifest 不写 skills 字段无所谓」**——看来在当前生态下约定扫描确实管用，3 个月都没出问题。这让我对强制显式注册的必要性降低了一些，但我仍然会维持显式注册，因为这是防御性设计。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 plugin.json 是否有显式 skills 注册
for pj in $(find . -name "plugin.json"); do
  has=$(python3 -c "import json; d=json.load(open('$pj')); print('YES' if 'skills' in d else 'NO')")
  echo "$pj -> skills field: $has"
done
# 命中后怎么办：在 plugin.json 里加 "skills": ["skills/xxx/SKILL.md", ...]
```

```bash
# 检查 skill description 是否有「Use when...」情境触发词
grep -rL "Use when\|When the user\|when someone" skills/*/SKILL.md
# 命中后怎么办：在该 skill 的 description 字段末尾补充 "Use when <具体情境>"
```

```bash
# 检查模糊量词
grep -rn -E '\b(authentic|typical|feels natural|meaningful|appropriate|regularly)\b' skills/*/SKILL.md
# 命中后怎么办：替换为可操作的具体描述（数据区间、频率定义、行动动词）
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `bureau` 套件添加入口 command `/bureau:start`，询问用户当前任务类型（recall/capture/review 等），路由到对应 skill
   - **为何可行**：bureau 的 7 个 skill 现在需要用户自己记住名字；加入口 command 后用户说「帮我记录今天的决策」就能自动触发
   - **第一步**：在 `bureau/commands/` 新建 `start.md`，用 `AskUserQuestion` 做 5 选 1 路由，参考 slavingia `minimalist-review` 的横切设计，20 分钟可完成

2. **想法**：给 `echo-sleuth` 的技术型 skill 添加 `## Examples` section
   - **为何可行**：slavingia 的成功告诉我「input→output 示例是高分 skill 的标配」；echo-sleuth 的 skill 描述了流程但没有示例
   - **第一步**：读 `echo-sleuth/skills/git-mining/SKILL.md`，在末尾添加 2 个具体的 git log 分析示例，30 分钟可完成

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 slavingia/skills 的核心目的**：将专业领域方法论（创业框架）打包为 skill 套件，让 Claude 成为这套框架的执行者

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为「领域知识 skill 化」；都是多个 skill 平铺；都面向非技术创作者 | drama 有 references/ 子目录结构；slavingia 是纯 SKILL.md | 高 |
| MarkQWu/bureau | 中 | 都是 knowledge-management 类 skill 套件 | bureau 有 command 层和 agent 层；功能型而非知识型 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为工具型 skill | echo-sleuth 是数据挖掘工具，slavingia 是咨询框架 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺少 skills 字段 | `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); print('skills' in d)"` | **bureau 命中、echo-sleuth 命中**（均输出 False） | 中（当前约定扫描可工作，但脆弱） |
| skill description 缺少「Use when」情境词 | `grep -L "Use when\|When the user" skills/*/SKILL.md` | **drama-workshop-skills**：`short-drama/SKILL.md` description 用「A comprehensive tool」开头，没有 Use when 短语 | 低 |
| 模糊量词 | `grep -n "appropriate\|comprehensive\|meaningful" skills/*/SKILL.md` | **bureau/skills** 0 命中（已处理干净）；**gstack/** 299 命中但多为注释 | 低（gstack 需单独清理） |

**命中后的具体行动建议**：
- `MarkQWu/bureau/.claude-plugin/plugin.json` → 添加 `"skills": ["skills/recall/SKILL.md", "skills/review/SKILL.md", ...]` → 5 分钟可完成
- `MarkQWu/drama-workshop-skills/short-drama/SKILL.md` → description 改为「Use when creating a short drama script, storyboarding, or developing character arcs」→ 2 分钟可完成

### 7.3 别人的更优方案

1. **领域**：skill 描述精确度（`Use when...` 情境触发）
   - **本案例做法**：每个 skill 的 description 包含精确的 `Use when <具体场景>` 短语（例：`skills/validate-idea/SKILL.md`: "Use when someone has a business idea and wants to test it before building"）
   - **我的项目现状**：`drama-workshop-skills/short-drama/SKILL.md` description 是「A comprehensive tool for...」，没有情境触发词
   - **如何借鉴**：将 short-drama skill description 改为「Use when creating a short drama script, developing a story arc for a 3–5 minute video, or storyboarding a drama episode」

2. **领域**：横切关注点用独立 skill 封装
   - **本案例做法**：`minimalist-review` 作为独立 skill，任何阶段都可调用，不与线性流绑定
   - **我的项目现状**：`bureau/skills/lint` 和 `bureau/skills/review` 已经是独立的横切 skill，但没有明确标注「可在任何阶段调用」
   - **如何借鉴**：在 bureau 的 lint / review skill description 里加「Use at any stage of the session to...」

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：引用子目录结构（references/）
- **我的做法**：`drama-workshop-skills/short-drama/references/` 包含 15 个子引用文档（剧本规则、故事板、角色发展等），让 skill 主体文件保持简洁，细节外化
- **本案例做法**：每个 SKILL.md 是自包含的——`mvp/SKILL.md` 把所有框架内容（三个阶段、四个问题、检查清单）都写在一个文件里，文件较长
- **意义**：我的 references/ 模式在内容多的 skill 里更易维护；slavingia 的自包含模式在内容少时更简洁。对于未来有大量细节的 skill，坚持用 references/ 是正确选择

---

## 八、术语表

### <a name="manifest"></a>manifest
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出插件名称、版本、描述，以及（可选的）skills/commands/agents 的路径清单。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也可能不会被加载。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 如何注册和调用。

### <a name="横切关注点"></a>横切关注点
> 软件工程术语，指「跨越多个模块都需要的功能」，如日志、安全检查、数据验证等。放在 NL skill 语境里，就是「任何阶段都需要的 skill」——slavingia 的 `minimalist-review` 就是一个横切关注点 skill：无论你在创业流程的哪个阶段，都可以调用它来复查当前决策是否符合极简主义原则。
