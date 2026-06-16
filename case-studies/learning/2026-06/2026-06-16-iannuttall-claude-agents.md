# iannuttall/claude-agents — 学习案例

**仓库**：https://github.com/iannuttall/claude-agents
**Stars**：⭐2046 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-16（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `examples-driven`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-agents 是一个面向日常开发和内容生产的 Claude Code sub-agent 集合，当前包含 7 个专职 agent：code-refactorer、content-writer、frontend-designer、prd-writer、project-task-planner、security-auditor、vibe-coding-coach。设计理念是「每个 agent 只干一件事」——重构代码的不写文档、写 PRD 的不碰代码，职责边界清晰。

仓库以 2046 颗星进入 xiaolai 审计视野，是中等热度的实用型工具集。没有 CLI 工具或 hook 脚本，纯 agent 定义文件，安装即用（Claude Code 原生加载）。

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── README.md                      # 项目说明，列出全部 7 个 agent 的名称和用途
  ├── agents/
  │   ├── code-refactorer.md         # 代码重构 agent
  │   ├── content-writer.md          # 内容写作 agent（含 outline / write 两种模式）
  │   ├── frontend-designer.md       # 前端设计规范生成 agent
  │   ├── prd-writer.md              # 产品需求文档 agent
  │   ├── project-task-planner.md    # 项目任务规划 agent
  │   ├── security-auditor.md        # 安全审计 agent
  │   └── vibe-coding-coach.md       # Vibe coding 教练 agent
  ```
- **文件类型分布**：7 个 agent 定义 / 1 个 README（无 skill、无 hook、无 command）
- **编排关系**：无 orchestrator，各 agent 独立被用户按需调用，互不依赖
- **跨件契约**：README.md 精确列出全部 7 个 agent 的名称和简介，与 `agents/` 目录完全对齐——这是「manifest 纪律」的体现

### 1.3 设计思路 / 方法论
- **核心设计哲学**：单职责原则（Single Responsibility）——每个 agent 专注一个领域，不互相越界，让用户按需选用，不引入不必要的依赖
- **解决什么问题**：开发者经常要在代码重构、写 PRD、设计前端规范、安全审计之间切换，每次切换都要重新给 Claude 设定上下文；专职 agent 把上下文配置固化，切换成本接近于零
- **做了什么 trade-off**：专职换来了精准，但也意味着每个 agent 只能在自己的技能边界内行动；当任务跨越多个领域时，用户需要手动协调多个 agent，没有 orchestrator 帮忙串联
- **反映什么认知模型**：作者把 Claude 视为「可插拔的专家模块」，用户是调度者，根据任务类型拨号到对应专家，而不是依赖一个「全能助手」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单职责 agent 集合 + manifest 纪律」**：每个 agent 文件只做一件事，README 作为 manifest 精确索引所有 agent，用户通过 manifest 发现能力，通过 agent 文件配置行为。

模式特征清单：
- 特征 1：每个 agent 有明确单一职责，描述范围不重叠（code-refactorer 和 vibe-coding-coach 都涉及代码，但切入角度完全不同）
- 特征 2：README.md 作为「agent 目录」，完整列出所有 agent 名称和一句话说明，无遗漏
- 特征 3：当前版本 agent description 字段包含内联 `<example>` 块，用具体任务示例帮助用户理解适用场景
- 特征 4：无 hook / 无 orchestrator——低依赖、低学习成本，适合「想要哪个就用哪个」的轻量需求

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 日常单领域任务（重构、写文档、安全审计）| ✅ 高度适用 | 专职 agent 配置完备，直接调用即可 |
| 跨领域编排（先 PRD 再设计稿再代码）| ⚠️ 需要用户手动串联 | 无 orchestrator，用户需要自己协调顺序 |
| 团队共享 agent 库 | ✅ 适用 | 单文件 agent 易于 fork 和定制 |
| 需要写文件输出的任务 | ❌ 部分失效 | content-writer / frontend-designer / vibe-coding-coach 无 `tools:` 声明，实际无法写文件 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单职责 agent 集合（本仓库）| claude-agents | 职责清晰、易于查找和定制 | 跨领域任务需手动协调、tools 声明遗漏会导致静默失效 |
| 全能 orchestrator + subagent | eyaltoledano/claude-task-master | 自动任务分解和串联 | 配置复杂、debug 困难 |
| 纯 skill 参考文档 | kepano/obsidian-skills | 最轻量、跨工具兼容 | 不能主动执行任务，只是行为约束 |

### 2.4 改进空间
1. **当前问题**：3 个 agent（content-writer、frontend-designer、vibe-coding-coach）body 描述了写文件的能力，但 frontmatter 缺少 `tools:` 声明，Claude Code 不会授权写文件工具。**改进做法**：在这三个 agent 的 frontmatter 加 `tools: Write, Read, Bash`（至少 Write）。**预期收益**：消除「承诺了但做不到」的静默失效，NLPM 分数从 88 升至约 93
2. **当前问题**：prd-writer 声明了 `tools: Task` 但 body 明确说「not responsible or allowed to create tasks」——声明与行为约束互相矛盾。**改进做法**：从 `tools:` 中移除 `Task`，或删除 body 中的禁止语句（二选一，取决于作者的真实意图）。**预期收益**：消除混淆，让工具声明真实反映 agent 能力
3. **当前问题**：全部 7 个 agent 缺少 `model:` 字段，使用平台默认模型。**改进做法**：根据任务复杂度选择：重逻辑的 agent（security-auditor、code-refactorer）用 Sonnet；内容生产类（content-writer、prd-writer）可以用 Haiku 降低成本；加 `model:` 字段明确锁定。**预期收益**：成本可控、行为可预期

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时整体得分 **88/100**

| agent | 当时分数 | 主要问题 |
|---|---|---|
| code-refactorer.md | 95/100 | 缺 `model:` 字段 |
| content-writer.md | 90/100 | 缺 `model:`，缺 `tools:`（body 承诺写文件但无法执行）|
| frontend-designer.md | 90/100 | 缺 `model:`，缺 `tools:`（body 生成 frontend-design-spec.md 但无法写）|
| prd-writer.md | 90/100 | 缺 `model:`，声明 `tools: Task` 但 body 禁止创建任务（矛盾）|
| project-task-planner.md | 90/100 | 缺 `model:`，声明 NotebookEdit 但 agent 只写 Markdown（多余声明）|
| security-auditor.md | 82/100 | 缺 `model:`，3 个未用工具（Edit、MultiEdit、NotebookEdit）+ 模糊词 "relevant"、"appropriate" |
| vibe-coding-coach.md | 80/100 | 缺 `model:`，缺 `tools:`（body 描述构建完整应用但无法创建/修改文件），无输出格式定义 |

**安全状态**：CLEAR——无 hook、无脚本、无 MCP 配置，没有可执行攻击面。

### 3.2 当时值得借鉴的模式
1. **README = agent manifest** → README.md 精确列出全部 7 个 agent 名称，与 `agents/` 目录一一对应，零遗漏 → 这是「manifest 纪律」：不把发现能力的任务甩给用户自己去 `ls` 目录
2. **vibe-coding-coach 的 UX 定位** → 不是「帮你写代码」而是「教你如何 vibe code」——这个定位差异让它不与 code-refactorer 重叠，单职责切分非常干净
3. **security-auditor 的工具声明审慎** → 只声明了必要工具（Bash 运行命令），没有声明可改代码的 Edit/Write——这反映了「审计不修改」的原则（尽管还带了 3 个多余工具）

### 3.3 当时的缺陷
1. **缺陷（关键 Bug）**：content-writer、frontend-designer、vibe-coding-coach 的 body 描述了输出文件的能力，但 frontmatter 无 `tools:` 声明，Claude Code 不会授权任何写文件工具 → **根本原因**：作者写 body 时想的是「这个 agent 会做什么」，但忘了 frontmatter 需要声明实现这些行为所需的工具权限——这是「正文契约工具」（body contracts tools）反模式
2. **缺陷（关键 Bug）**：prd-writer 声明 `tools: Task` 但 body 说「not responsible or allowed to create tasks」→ **根本原因**：工具声明和行为约束是两处维护的，作者修改了 body 的权限限制但没有同步更新 frontmatter
3. **缺陷（质量问题）**：全部 7 个 agent 无 `model:` 字段 → **根本原因**：Claude Code 有合理的平台默认值，初写 agent 时容易忘记锁定，直到出现模型升级导致行为变化时才注意到

### 3.4 当时的优化机会
1. 给 content-writer、frontend-designer、vibe-coding-coach 加 `tools: Write, Read`（各 30 秒，机械修复）
2. 解决 prd-writer 的 tools/body 矛盾（移除 Task 或修改 body，二选一）
3. 给全部 7 个 agent 加 `model:` 字段（建议按任务复杂度分两档：Sonnet / Haiku）
4. 清理 security-auditor 的 3 个未用工具声明（机械删除）
5. 给 project-task-planner 移除多余的 NotebookEdit 声明（该 agent 不处理 Jupyter 文件）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| content-writer 无 `tools:` | `grep "tools:" agents/content-writer.md` | **仍未修复** — frontmatter 无 `tools:` 行 | Bug 持续 |
| frontend-designer 无 `tools:` | `grep "tools:" agents/frontend-designer.md` | **仍未修复** — frontmatter 无 `tools:` 行 | Bug 持续 |
| vibe-coding-coach 无 `tools:` | `grep "tools:" agents/vibe-coding-coach.md` | **仍未修复** — frontmatter 无 `tools:` 行 | Bug 持续 |
| prd-writer tools/body 矛盾 | `grep "tools:" agents/prd-writer.md` | **仍未修复** — 仍声明 `tools: Task, Bash, Grep, LS, Read, Write, WebSearch, Glob`，body 仍禁止创建 Task | Bug 持续 |
| project-task-planner 多余 NotebookEdit | `grep "NotebookEdit" agents/project-task-planner.md` | **仍未修复** — NotebookEdit 仍在声明中 | 质量问题持续 |
| security-auditor 3 个未用工具 | `grep "tools:" agents/security-auditor.md` | **仍未修复** — Edit、MultiEdit、NotebookEdit 仍在 | 质量问题持续 |
| 全部 agent 无 `model:` | `grep -L "^model:" agents/*.md` | **仍未修复** — 7 个文件均无 model 行 | 质量问题持续 |

### 4.2 架构演进
从审计快照到当前 HEAD，代码逻辑层面的 bug 无一修复——但 agent **描述质量有实质性改善**：

- **content-writer.md**：description 字段现在明确区分两种工作模式（outline 模式 / write 模式），并包含具体用法示例，可发现性大幅提升
- **vibe-coding-coach.md**：description 字段新增内联 `<example>` 块，带 `color: pink` 视觉标识，包含把用户构想转化为可运行应用的具体示例
- **其他 agent**：description 字段也有不同程度的充实，行文更清晰

这个现象值得注意：**维护者在没有 PR 提示的情况下，自发改善了「可发现性」维度，但忽略了「工具声明完整性」维度**。两个维度在用户体验上的差异是：description 的改善用户立刻能感知（选 agent 时看得到），tools 声明的缺失用户在调用失败前感知不到。这是「可见质量 vs 隐形质量」的典型分化。

### 4.3 新增的可学习模式
**内联 `<example>` 块在 description 字段中的使用**是这批更新最值得借鉴的写法：

```markdown
description: |
  A content writer that helps you create high-quality written content.
  
  <example>
  用户说：帮我写一篇关于 TypeScript 类型体操的博客
  → agent 先进入 outline 模式，输出文章骨架请用户确认
  → 用户确认后，进入 write 模式，逐节展开内容
  </example>
```

这个写法把「agent 能做什么」从抽象描述变成了可观察的行为序列，让用户在选 agent 时就能判断是否匹配自己的需求，而不是调用后才发现不符合预期。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责切分**：echo-sleuth-for-claude 的 5 个 agent（analyze、file-historian、memory-auditor、recall、schema-scout）职责边界清晰，互不越界——与 iannuttall 的设计思路一致
2. **工具声明完整**：echo-sleuth 的所有 agent 都有 `tools:` 声明（Read、Bash、Grep、Glob），不存在「body 承诺工具但 frontmatter 缺声明」的问题——这点比 iannuttall 做得好

### 5.2 挑战 / 验证
这次案例对我最大的认知更新是**「正文契约工具」反模式的无声失效特性**：当 agent body 描述了写文件的能力，但 frontmatter 没有声明 Write 工具时，agent 会正常响应用户的请求，生成完整的文件内容——然后什么都不写。用户看到的是一段「我已经为你生成了 frontend-design-spec.md，内容如下……」的回复，但磁盘上没有文件。这种失效不报错、不警告，用户只能自己去检查文件是否真的创建了。

NLPM 的机械检查能直接发现这类问题（`tools:` 字段是否存在），这是 NLPM 对用户有实际价值的体现。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 有没有「body 提到写文件但 frontmatter 无 Write 工具」
# 先找出所有声明或隐含了「写文件」语义的 agent
grep -rl "write\|save\|creat.*file\|output.*file\|生成文件" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/ 2>/dev/null

# 再检查这些 agent 的 frontmatter 是否包含 Write 工具
grep -L "Write" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null

# 核心自查：我的所有 agent 有没有 model: 字段（iannuttall 的共性缺陷）
grep -rL "^model:" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null

# 检查有没有「tools 声明但 body 明确禁止使用」的矛盾（prd-writer 反模式）
# 先找有 tools 声明的 agent，再人工对照 body 看有没有禁止语句
grep -rn "^tools:" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/ 2>/dev/null

# 检查有没有声明了但 body 完全没有用到的工具（security-auditor 反模式）
# 示例：查找声明了 NotebookEdit 但 body 没有提到 Jupyter/notebook 的 agent
grep -l "NotebookEdit" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md 2>/dev/null \
  | xargs grep -L "notebook\|jupyter\|ipynb" 2>/dev/null
```

命中后怎么办：
- **body 写文件但无 Write 声明**：在 frontmatter 加 `tools: Write, Read`（30 秒机械修复）
- **无 `model:` 字段**：按任务复杂度加 `model: claude-sonnet-4-5` 或 `model: claude-haiku-4-5`
- **tools/body 矛盾**：二选一——要么从 `tools:` 删除矛盾工具，要么修改 body 去掉禁止语句，取决于真实设计意图
- **声明了但未用的工具**：直接从 `tools:` 删除，不影响 agent 功能

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 的 agent description 字段加内联 `<example>` 块（借鉴 iannuttall 当前版本的改进）
   - **为何可行**：echo-sleuth 的 5 个 agent 当前描述都很简短，用户不容易判断「这个 agent 适合什么场景」；加 example 可以把使用门槛从「先试错」降到「先看示例」
   - **第一步**：从 recall.md 开始，在 description 末尾加一个 `<example>` 块，描述用户问「我上周修改了哪些配置文件」→ recall 怎么查找和回答；约 15 分钟
2. **想法**：给 echo-sleuth 的所有 agent 加 `model:` 字段
   - **为何可行**：5 个 agent 全部缺 model（同 iannuttall 的共性缺陷），当平台默认模型升级时，agent 行为可能无声变化
   - **第一步**：在每个 agent 的 frontmatter 加 `model: claude-haiku-4-5`（echo-sleuth 是分析类任务，不需要最强模型，Haiku 足够且成本低）；5 个文件各 30 秒，总计约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 iannuttall/claude-agents 的核心目的**：为常见开发和内容生产任务提供开箱即用的专职 agent 集合，单职责、低依赖、按需组合
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | **高** | 都是「小型专职 agent 集合」（7 vs 5 个 agent），都面向日常生产力提升 | echo-sleuth 聚焦 session 挖掘分析，iannuttall 聚焦开发/内容生产 | **高** |
| MarkQWu/claude-for-legal | 中 | 都有多个独立 agent，各自覆盖不同领域 | claude-for-legal 的 agent 有领域深度，iannuttall 是通用生产力工具 | **中** |
| MarkQWu/drama-workshop-skills | 低 | — | 完全不同领域，且 drama-workshop 是 skill 而非 agent | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 所有 agent 缺 `model:` 字段 | `grep -L "^model:" agents/*.md` | **命中**：echo-sleuth 全部 5 个 agent（analyze、file-historian、memory-auditor、recall、schema-scout）均无 `model:` 字段 | **中** |
| body 承诺工具但 frontmatter 无声明 | `grep -rL "Write" agents/*.md` | **未命中**：echo-sleuth 所有 agent 都有明确 `tools:` 声明（Read, Bash, Grep, Glob），无隐藏能力承诺 | N/A |
| tools/body 矛盾声明 | 人工对照 | **未命中**：echo-sleuth 的 tools 声明和 body 描述一致 | N/A |
| 声明了未使用的工具 | 对照 body 检查 | **潜在命中**：echo-sleuth 所有 agent 声明了相同的 4 个工具（Read, Bash, Grep, Glob），但 Write 缺失——如果某个 agent 需要输出文件（如 schema-scout 发现模式后需要写报告），当前声明不够 | **中** |

**命中后的具体行动建议**：

1. **`model:` 缺失（5 个 agent 全命中）**：在每个 agent frontmatter 加一行。推荐分两档：
   - recall.md、schema-scout.md（查询类）→ `model: claude-haiku-4-5`
   - analyze.md、memory-auditor.md、file-historian.md（分析类）→ `model: claude-sonnet-4-5`

2. **schema-scout 可能需要 Write**：schema-scout 的任务是发现会话模式，如果它应该生成报告文件，当前缺少 Write 工具。检查 body 描述，如有输出文件的意图，补加 `Write`

### 7.3 别人的更优方案

1. **领域**：agent description 中的内联示例（iannuttall 当前版本新增）
   - **本案例做法**：在 description 字段写具体的用户 → agent 交互流程，用 `<example>` 块包裹，让用户在 Claude Code 的 agent 选择界面就能判断是否适用
   - **我的项目现状**：echo-sleuth 的 agent description 都是一句话功能说明，没有示例，用户需要实际调用后才知道是否符合需求
   - **如何借鉴**：参照 iannuttall 的 content-writer 写法，给 echo-sleuth 每个 agent 加 1 个 `<example>` 块；recall.md 最有代表性（用户最容易误解「召回」是什么意思），优先改这个

2. **领域**：README 作为 agent 发现 manifest（不是文档，是索引）
   - **本案例做法**：README.md 就是一张表格：agent 名称 + 一句话说明；维护者保持了 README 和 `agents/` 目录的完全对齐，这是一个需要纪律的约定
   - **我的项目现状**：echo-sleuth 的 README 描述整体功能，但没有逐 agent 列出名称和简介；用户需要进 `agents/` 目录才能知道有哪些 agent
   - **如何借鉴**：在 echo-sleuth README 的「Agents」段落加一张表格，列出 5 个 agent 名称 + 一句话说明；约 10 分钟

### 7.4 反向：我的项目做得比他们好的地方

1. **领域**：tools 声明完整性（无「正文契约工具」反模式）
   - **我的做法**：echo-sleuth 的 5 个 agent 都有完整的 `tools:` 声明，body 没有承诺 frontmatter 未声明的工具能力
   - **本案例做法**：3 个 agent 存在「body 承诺写文件但 frontmatter 无 Write 声明」的静默失效 bug，且从 2026-04-06 至今（2026-06-16）超过 70 天未修复
   - **意义**：这个对比说明「正文契约工具」反模式是可以系统性避免的——在写 agent body 时，每当写到「保存」「生成文件」「输出」等词，就立刻检查 frontmatter 的 tools 是否覆盖了所需工具。这个检查习惯零成本，收益是避免 70 天无声失效

---

## 八、术语表

### <a name="正文契约工具"></a>正文契约工具（body contracts tools）反模式
> 指 agent 的 body（描述文本）声称自己具备某种能力（如「保存 Markdown 文件」「生成设计规范文档」），但 frontmatter 的 `tools:` 字段没有声明实现该能力所需的工具（如 `Write`）。结果是：agent 会按 body 的描述给出完整的文字输出，但 Claude Code 不会执行任何文件写入操作——静默失效，不报错，用户只能在检查磁盘后才发现文件没有创建。NLPM 的 `tools:` 完整性检查可以机械发现此类问题。

### <a name="manifest-纪律"></a>manifest 纪律
> 在有多个 agent / skill / command 的仓库中，保持「入口索引文件（README / plugin.json 等）与实际文件目录完全对齐」的工程习惯。具体要求：每新增一个 artifact，就同步更新 manifest；删除 artifact 时，同步从 manifest 移除。违反 manifest 纪律的症状：manifest 列出了实际不存在的 artifact，或目录里有 artifact 但 manifest 未提及。iannuttall/claude-agents 的 README 是 manifest 纪律的正面案例。

### <a name="单职责-agent"></a>单职责 agent（single-purpose agent）
> 每个 agent 只承担一个明确定义的功能领域，功能边界不与其他 agent 重叠。典型标志：agent 的名称直接说明其职责（code-refactorer、content-writer），且 body 描述不延伸到其他领域。与「全能助手」（general assistant）相对，单职责 agent 的优势是行为可预测、工具声明最小化、用户能精确选择所需能力。

### <a name="vague-quantifier"></a>模糊量化词（vague quantifier）
> agent body 或 skill 描述中使用的模糊限定词，如 "relevant"（相关的）、"appropriate"（合适的）、"some"（一些）、"various"（各种）。这类词的问题是：它们把判断标准留给 Claude 自行决定，导致不同调用间行为不一致。NLPM 会标记这类词并建议替换为具体标准（如 "within the last 7 days"、"up to 3 items"）。

### <a name="model-字段"></a>model 字段
> agent frontmatter 中的 `model:` 字段，指定该 agent 使用的具体模型 ID（如 `claude-sonnet-4-5`、`claude-haiku-4-5`）。缺少 model 字段时，Claude Code 使用平台默认模型。问题在于：当 Anthropic 升级平台默认模型时，agent 行为可能无声发生变化（新模型的风格、工具使用习惯可能不同）。建议：生产用 agent 锁定 model 字段，在明确测试并决定升级时才手动更新。

### <a name="可见质量-vs-隐形质量"></a>可见质量 vs 隐形质量
> agent 工程中的两类质量维度：**可见质量**指用户在选择和使用 agent 时能直接感知的部分（description 清晰度、示例丰富度、命名直觉性）；**隐形质量**指只有在执行失败时才能发现的部分（tools 声明完整性、model 锁定、工具与 body 的一致性）。iannuttall 案例展示了典型的分化：维护者自发改善了可见质量（description 加 example），但隐形质量缺陷（tools 声明缺失）在 70 天内无人感知、无人修复。
