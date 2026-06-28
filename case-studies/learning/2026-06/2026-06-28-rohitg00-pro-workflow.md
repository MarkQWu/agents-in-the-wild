# rohitg00/pro-workflow — 学习案例

**仓库**：https://github.com/rohitg00/pro-workflow
**Stars**：1912 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-28（基于 audit 数据）
**主题标签**：`manifest-discipline`, `single-purpose`, `cross-reference`, `vague-quantifier`, `examples-driven`

**xiaolai 案例**：无（本案例为首份）

---

## 一、理解（基于 2026-04-06 audit 快照）

### 1.1 仓库概览

[rohitg00/pro-workflow](https://github.com/rohitg00/pro-workflow) 是 Rohit Ghumare 发布的一个全方位 AI 开发工作流插件，1912 stars，定位是"从会话启动到代码提交、从成本分析到安全审计"的一站式工作流工具集。60 个 artifact 覆盖 21 个命令、14+ 个技能、8 个 agent 文件、35 个 Node.js hook 脚本，是一个典型的[分层协作型工作流架构](#分层协作型工作流架构)。

NLPM 评分 **90/100**，属于高质量区间——这是 audit 时 157 个仓库中靠前的分数。但这个 90 分并不均匀：orchestrator.md 满分 100，而 12 个命令因为缺少 `description` frontmatter 全部得分 70，拉低了整体均值。安全门被触发（BLOCKED），但主要 Critical finding 是测试用例中的 AWS 文档密钥（误报），实际安全状况比标签显示的更乐观。

三个关键事实：
1. **orchestrator.md 是最佳 agent 范本**：model(opus)、skills 列表、memory 声明、phases 全部到位，score=100，是 NLPM 审计过的 agent 中少数做到满分的案例
2. **12 个命令缺少 description frontmatter**：导致这些命令无法在 `/help` 中显示，marketplace 信息不完整，是最大的机械性 bug 集群
3. **hooks.json 连线完整**：26 个 hook 事件、35 个脚本引用，无断链，自动化守护层设计扎实

### 1.2 架构剖析

```
pro-workflow/
├── commands/               -- 21 个 command md 文件
│   ├── learn.md            -- 学习记录（有 SQL 注入风险）
│   ├── commit.md           -- 智能提交
│   ├── wrap-up.md          -- 会话收尾
│   ├── handoff.md          -- 跨会话交接
│   ├── develop.md          -- 开发流（委托 orchestrator）
│   ├── doctor.md           -- 环境诊断（含测试密钥）
│   ├── replay.md           -- 会话重放（有 FTS5 注入风险）
│   └── ...（另 14 个）
├── skills/                 -- 14+ 个 skill md 文件
│   ├── pro-workflow/SKILL.md        -- 核心工作流（562 行，过长）
│   ├── smart-commit/SKILL.md        -- 提交策略（score=98）
│   ├── session-handoff/SKILL.md     -- 跨会话交接（score=98）
│   ├── deslop/SKILL.md              -- 输出质量控制
│   ├── bug-capture/SKILL.md         -- Bug 捕获（score=98）
│   └── batch-orchestration/SKILL.md -- 批量编排
├── agents/                 -- 8 个 agent md 文件
│   ├── orchestrator.md     -- 核心编排器（score=100）
│   ├── debugger.md         -- 调试专家（score=98）
│   ├── scout.md            -- 侦察 agent（未声明 model）
│   ├── cost-analyst.md     -- 成本分析（未注册到 plugin.json）
│   └── permission-analyst.md -- 权限分析（未注册到 plugin.json）
├── scripts/                -- 35 个 Node.js hook 脚本
├── hooks/
│   └── hooks.json          -- 26 个 hook 事件，35 个脚本引用
├── .claude-plugin/
│   └── plugin.json         -- 插件清单（score=80，2 个 agent 未注册）
└── templates/
    └── split-claude-md/CLAUDE.md   -- 模板（score=75，纯占位符）
```

**制品质量分布**：

| 制品类型 | 数量 | 平均分估计 | 主要缺陷 |
|----------|------|------------|---------|
| agent | 8 | ~90 | 6 个未声明 model（各 -5） |
| command | 21 | ~72 | 12 个缺 description（各 -10） |
| skill | 14+ | ~95 | pro-workflow/SKILL.md 过长 |
| hooks.json | 1 | ~90 | 完整，26 事件无断链 |
| plugin.json | 1 | 80 | 2 个 agent 未注册 |
| CLAUDE.md 模板 | 1 | 75 | 纯占位符内容 |

**加权平均**：90/100

**跨件关系**（强项）：大多数 command 与对应 skill 的输出格式匹配——`/commit` 调用 `skills/smart-commit`，`/wrap-up` 调用 `skills/session-handoff`，`/develop` 委托 `agents/orchestrator.md`。hooks.json 的 35 个脚本引用全部有效，无悬空引用。

### 1.3 设计思路：command-skill-agent 三层分工，hooks 后台守护

pro-workflow 的核心设计哲学是**职责分离**：

- **command 层**：用户入口，单一动词（learn / commit / wrap-up），触发即执行
- **skill 层**：参考知识库，command 和 agent 共享读取，不直接执行
- **agent 层**：执行者，由 command 委托，orchestrator 负责调度其他 agent
- **hooks 层**：后台守护，用 35 个 Node.js 脚本自动在工具调用后触发检查

这个分层让"命令轻、技能重、执行者专"——command 文件保持简短，业务逻辑放在 skill 里，复杂编排交给 orchestrator。这是与 `rohitg00/awesome-claude-code-toolkit`（该仓库 command 和 agent 之间无显式编排引用）相比最大的结构进步：**pro-workflow 有明确的编排中心**。

Trade-off：hooks 层使用 35 个 Node.js 脚本带来了 npm 依赖链，增加了供应链攻击面（package.json 中 better-sqlite3 使用 `^` 范围，版本漂移风险存在）。纯 bash hooks 会更安全但表达能力弱。

---

## 二、架构模式（横向）

### 2.1 模式归类

**模式名：「分层协作型工作流架构」**

pro-workflow 最值得学习的是它如何把 60 个 artifact 组织成一个有内聚性的工作流——而不是 60 个互不相关的文件堆。核心模式特征：

- command 是前门：用户说 `/develop`，command 文件接收 `$ARGUMENTS`，委托给 agent
- skill 是共享知识：多个 command 和 agent 都可以声明 `skills: [smart-commit]`，避免重复定义
- agent 是专家：orchestrator 不自己写代码，而是把子任务路由给 debugger、scout 等专用 agent
- hooks 是哨兵：每次工具调用后自动触发，不需要用户主动调用

这个模式的关键洞察：**单一 orchestrator 作为路由中心，其他 agent 各司其职**。orchestrator.md 得分 100 正是因为它完整声明了调度意图（phases）、引用了哪些 skill、使用什么模型（opus）、如何管理 memory。

### 2.2 适用场景

| 场景 | 适用度 | 原因 |
|------|--------|------|
| 团队多人协作工作流标准化 | ✅ 高度适用 | 统一的 command 入口 + skill 共享层，团队成员无需了解底层 agent 实现 |
| 个人 AI 开发效率工具集 | ✅ 适用 | 60 个 artifact 覆盖全流程，安装即用 |
| 极简单一任务自动化 | ⚠️ 过度设计 | 单个 SKILL.md + 一个 command 足矣，三层架构是额外成本 |
| 需要精细成本控制的团队 | ⚠️ 有限适用 | cost-analyst.md 未注册到 plugin.json，功能入口缺失 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|------|----------|------|------|
| 分层协作型（本案例） | pro-workflow | 职责清晰，单一 orchestrator 路由，hooks 自动守护 | hooks 依赖 Node.js，描述型 command 缺 description |
| 内容驱动扁平型 | rohitg00/awesome-claude-code-toolkit | 覆盖广度大，生态曝光强 | command 与 agent 无编排关系，质量 46/100 |
| NL-Binary 混合型 | mikeyobrien/ralph-orchestrator | 执行可靠，NL 是 Rust 的配置文件 | 编译耦合，SKILL.md 改动需 rebuild |
| 极简单文件型 | lackeyjb/playwright-skill | 维护成本极低 | 无法编排复杂子任务 |

### 2.4 改进空间

1. **12 个命令缺 description frontmatter**：`commands/learn.md`、`commands/handoff.md` 等 12 个文件的 YAML frontmatter 缺少 `description:` 字段，导致 `/help` 命令无法显示这些命令的用途，marketplace 信息不完整。**改进做法**：在每个命令文件的 frontmatter 顶部加一行 `description: <一句话说明>`。**预期收益**：命令在 `/help` 中可见，用户无需猜测功能。

2. **plugin.json 未注册 2 个 agent**：`cost-analyst.md` 和 `permission-analyst.md` 存在于 `agents/` 目录但未出现在 `.claude-plugin/plugin.json` 的 `agents` 数组中。用户安装插件后无法通过正常入口调用这两个专用 agent。**改进做法**：在 plugin.json 的 `agents` 数组中追加这两个 agent 的路径。**预期收益**：plugin.json score 从 80 升至 ≥95。

3. **skills/pro-workflow/SKILL.md 过长（562 行）**：单个 skill 文件超过合理长度（建议 ≤200 行），Claude 在读取时可能截断或偏重前半部分内容。**改进做法**：按职责拆分为 3–4 个子 skill（如 `pro-workflow-setup`、`pro-workflow-session`、`pro-workflow-review`），orchestrator.md 按需引用。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：90/100**（60 个 artifact，加权均值）

| 文件 | 当时分数 | 主要扣分原因 |
|------|----------|------------|
| `agents/orchestrator.md` | **100/100** | 无——model(opus)、skills、memory、phases 全声明 |
| `agents/debugger.md` | **98/100** | 多个 examples，model=opus，接近完美 |
| `skills/bug-capture/SKILL.md` | **98/100** | domain-language 焦点明确 |
| `skills/smart-commit/SKILL.md` | **98/100** | review suppressions list 模式优秀 |
| `skills/session-handoff/SKILL.md` | **98/100** | resume command 模式，跨会话价值高 |
| `hooks/hooks.json` | **90/100** | 26 事件连线完整 |
| `agents/scout.md`（及另 5 个 agent）| ~95 | 未声明 model（各 -5） |
| `.claude-plugin/plugin.json` | 80 | 2 个 agent 未注册 |
| `commands/develop.md` | 85 | 无 `$ARGUMENTS` 空输入检测 |
| `commands/learn.md`（及另 11 个命令）| 70 | 缺 `description` frontmatter（各 -10） |
| `templates/split-claude-md/CLAUDE.md` | 75 | 纯占位符内容（[Project Name] 等） |

```
分数分布（60 个 artifact）：
95–100 分（优秀）：~10 个（orchestrator, debugger, 3 个 skill）
85–94 分（良好）：~25 个
70–84 分（待改进）：~25 个（12 个命令 + plugin.json + 模板）
```

### 3.2 当时值得借鉴的模式

1. **orchestrator.md = 100 分满分的 agent 范本**：完整的 `model: claude-opus-4`、`skills:` 列表、`memory:` 声明、`phases:` 分阶段执行计划——这四个要素同时到位是 NLPM 审计中少数案例。自查方法：`grep -L "^model:\|^skills:\|^memory:\|^phases:" agents/*.md`

2. **session-handoff 的 resume command 模式**：`skills/session-handoff/SKILL.md` 定义了一个跨会话状态传递协议——会话结束时写入 handoff 文件，新会话开始时读取并恢复上下文。这是处理 Claude 上下文窗口限制的实用模式。→ 借鉴：我的工作流是否有会话间状态传递的机制？

3. **smart-commit 的 review suppressions list**：在 commit 之前，skill 会扫描一个 suppressions 清单，避免把临时调试代码、TODO 注释、本地路径硬编码提交进去。这是"commit 质量门"的 NL 实现。

4. **hooks.json 的完整性**：35 个脚本引用，26 个 hook 事件，全部有效无断链。做到这一点需要在每次新增脚本时同步更新 hooks.json——是严格的[manifest discipline](#manifest-discipline)体现。

### 3.3 当时的缺陷

1. **12 个命令缺 description frontmatter（最大 bug 集群）**：`learn.md`、`handoff.md`、`wrap-up.md`、`commit.md` 等 12 个命令文件的 YAML frontmatter 没有 `description:` 字段。根本原因：命令文件是批量编写的，作者专注于内容而忽略了 frontmatter 规范。影响：`/help` 无法显示这些命令，marketplace 安装后用户需要逐一阅读源码才能了解功能。

2. **6 个 agent 未声明 model**：`scout.md`、`cost-analyst.md`、`permission-analyst.md` 等 6 个 agent 文件缺少 `model:` frontmatter 字段，但 `orchestrator.md` 和 `debugger.md` 有。根本原因：高优先级 agent 优先完善，其余 agent 未及同步。影响：Claude Code 无法在工具链层面读取模型选择，只能靠内容推断。

3. **plugin.json 未注册 2 个 agent**：`cost-analyst.md` 和 `permission-analyst.md` 在文件系统存在但在 plugin.json 的 `agents` 数组中缺失。这是[manifest discipline](#manifest-discipline)失效——文件存在但清单不知道。

4. **commands/learn.md SQL 注入风险（MEDIUM）**：第 207 行将用户输入直接插入 `sqlite3 INSERT` shell 命令，无参数化处理。攻击者构造含特殊字符的学习记录内容可导致 shell 命令注入。→ 修复：改用参数化查询或先转义特殊字符。

5. **commands/replay.md FTS5 注入风险（MEDIUM）**：第 27 行将关键词直接插入 FTS5 MATCH 子句，特殊字符可导致查询语法错误或意外行为。

### 3.4 当时的优化机会

1. **给 12 个命令补 description frontmatter**：机械操作，每个文件 1 行，批量完成
2. **给 6 个 agent 加 model 声明**：参考 orchestrator.md 的写法，判断每个 agent 的任务复杂度选 opus/sonnet/haiku
3. **在 plugin.json 中注册 cost-analyst 和 permission-analyst**：plugin.json 的 `agents` 数组追加两个路径
4. **修复 commands/learn.md 的 SQL 注入**：参数化处理用户输入
5. **拆分 skills/pro-workflow/SKILL.md**：562 行 → 3 个子 skill，各自聚焦一个职责

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷的当前状态

> 注：目标仓库克隆不可用（代理阻断）。以下分析基于 2026-04-06 audit 快照数据，无法通过 grep 验证当前 HEAD 状态。建议读者在本地克隆后执行验证命令。

| 过去缺陷 | 建议验证方法 | audit 时状态 | 备注 |
|----------|------------|------------|------|
| 12 个命令缺 description | `grep -rL "^description:" commands/*.md` | 确认存在，12 个文件 | 机械修复，预计优先级低 |
| 6 个 agent 未声明 model | `grep -rL "^model:" agents/*.md` | 6 个文件缺失 | 包括 scout, cost-analyst 等 |
| plugin.json 未注册 2 agent | `jq '.agents' .claude-plugin/plugin.json` | cost-analyst, permission-analyst 缺失 | 影响用户调用 |
| learn.md SQL 注入 | `grep -n "sqlite3\|INSERT" commands/learn.md` | 第 207 行，用户输入未转义 | MEDIUM 风险 |
| replay.md FTS5 注入 | `grep -n "MATCH\|FTS5" commands/replay.md` | 第 27 行，关键词直接插入 | MEDIUM 风险 |
| version 不同步 | `grep -h '"version"' package.json .claude-plugin/plugin.json` | package.json=3.2.0, plugin.json=3.1.0 | 版本管理混乱 |

### 4.2 架构稳定性评估

基于 audit 数据的架构判断：

- **强稳定区**：orchestrator.md + hooks.json + 三个高分 skill（smart-commit、session-handoff、bug-capture）构成了核心价值，这部分架构成熟，不太可能有破坏性变化
- **弱稳定区**：12 个缺 description 的命令和 2 个未注册的 agent，属于遗漏而非设计决策，随时可能被补充修复
- **安全门状态**：CRITICAL finding（doctor.md 中的 `AKIAIOSFODNN7EXAMPLE`）是 AWS 官方文档测试密钥，属于误报，不构成实际风险；Medium finding 需要关注

### 4.3 可能的演进方向

根据 pro-workflow 的设计意图推断，未来版本可能：
1. 补全命令 description（机械修复，社区 PR 极可能被接受）
2. 注册 cost-analyst 和 permission-analyst 到 plugin.json（功能完整性修复）
3. 拆分 pro-workflow/SKILL.md 大文件（重构，作者主动发起概率中等）

---

## 五、校准

### 5.1 我已经在做对的

1. **理解了 agent 满分的必要条件**：audit 揭示了 orchestrator.md 得 100 分的原因——不是写得多，而是四个关键字段（model/skills/memory/phases）全部到位。这个标准比我之前认为的"有 description 就够了"严格得多。如果我的 agent 缺少 phases 声明，即使内容再丰富也会失分。

2. **hooks 连线纪律**：hooks.json 35 个引用全部有效这件事让我意识到，hooks.json 是"清单"，而非"文档"——每次新增脚本文件，必须同步更新 hooks.json，否则脚本存在但不被触发。这是典型的[manifest discipline](#manifest-discipline)要求。

3. **session-handoff 模式的价值**：`skills/session-handoff/SKILL.md` 得分 98 并不只是因为写得好，而是因为它解决了一个真实痛点——Claude 的上下文窗口限制。跨会话的状态传递是长期项目使用 AI 的必要设施。

### 5.2 挑战 / 验证

本案例**挑战**了我的一个假设：我之前认为"90 分的仓库已经足够好了"。但 pro-workflow 的 90 分中藏着 13 个 bug，其中 12 个是命令缺 description——这不是质量问题，而是可见性问题。用户安装了插件后，`/help` 显示出来的只是 9 个有 description 的命令，另外 12 个完全不可见，等于功能损失了一半。**90 分不等于 90% 的功能可用**。

本案例**验证**了"orchestrator 作为路由中心"的架构优越性：与 `rohitg00/awesome-claude-code-toolkit`（同作者，46 分）相比，pro-workflow 多了一个 orchestrator.md 作为编排中心，整体分数高出 44 分，跨件引用质量也显著更好。同一个作者，有没有编排中心，差距是 44 分。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 是否缺 description frontmatter（对照 pro-workflow 的最大 bug）
grep -rL "^description:" \
  ~/.claude/commands/*.md \
  ~/projects/gstack/.claude/commands/*.md 2>/dev/null

# 命中说明该命令在 /help 中不可见——需要加 description: 行
```

```bash
# 检查我的 agents 是否缺 model 声明（对照 pro-workflow 6 个 agent 失分）
grep -rL "^model:" \
  ~/.claude/agents/*.md \
  ~/projects/gstack/.claude/agents/*.md 2>/dev/null

# 命中后：参考 orchestrator.md，根据 agent 任务复杂度选 opus/sonnet/haiku
```

```bash
# 检查 plugin.json 注册的 agents 是否与实际文件一致（对照 2 个未注册 agent 问题）
# 先列出文件系统的 agent 文件
ls agents/*.md 2>/dev/null

# 再检查 plugin.json 中注册的 agents 列表
cat .claude-plugin/plugin.json | grep -A 20 '"agents"'

# 对比两个列表，确认无遗漏
```

```bash
# 检查 hooks.json 中的脚本引用是否都存在（对照 hooks 连线纪律）
# 提取 hooks.json 中的所有脚本路径，验证文件存在
python3 -c "
import json, os, sys
with open('hooks/hooks.json') as f:
    data = json.load(f)
missing = []
for hook in data.get('hooks', []):
    script = hook.get('script', '')
    if script and not os.path.exists(script):
        missing.append(script)
if missing:
    print('断链脚本:', missing)
    sys.exit(1)
else:
    print('所有脚本引用有效')
"
```

```bash
# 检查我的 agent 是否有 phases 声明（orchestrator.md 满分的关键）
grep -rL "^phases:" \
  ~/.claude/agents/*.md \
  ~/projects/gstack/.claude/agents/*.md 2>/dev/null

# 命中后不一定需要加 phases——但对于 orchestrator 型 agent，phases 是必要的
```

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 的所有命令补 description frontmatter
   - **为何可行**：pro-workflow 的最大 bug 就是这个，批量修复每个文件只需 1 分钟
   - **第一步**：运行上面的 grep，确认哪些命令缺失；逐个加 `description: <一句话>` 到 frontmatter
   - **完成标志**：`grep -rL "^description:" ~/.claude/commands/*.md` 返回空

2. **想法**：给 orchestrator 型 agent 加 phases 声明（参考 orchestrator.md 100 分范本）
   - **为何可行**：pro-workflow 的 orchestrator.md 满分来源之一是 phases 声明，让 Claude 知道"先做什么，后做什么"
   - **第一步**：找到 gstack 中最复杂的 agent（如果有编排器角色），在 frontmatter 加 `phases:` 章节，列出 3–5 个执行阶段
   - **完成标志**：`grep -l "^phases:" ~/projects/gstack/.claude/agents/*.md` 至少命中 1 个

3. **想法**：引入 session-handoff 模式（参考 skills/session-handoff/SKILL.md，score=98）
   - **为何可行**：gstack 如果用于长期项目，上下文窗口限制是真实问题；session-handoff 协议可以让每次新会话快速恢复上下文
   - **第一步**：参考 `skills/session-handoff/SKILL.md` 的设计，在 gstack 中创建 `skills/session-state/SKILL.md`，定义会话结束写入 / 会话开始读取的格式
   - **完成标志**：新建的 SKILL.md 有 `## 输出格式` 章节，明确说明 handoff 文件的字段结构

---

## 七、对照我的 GitHub 仓库

> 数据源：audit 快照分析 + 对 gstack 目标相似度的类比推断。gstack 代码库未能直接 grep（建议读者在本地执行验证命令）。

### 8.1 目的对齐度

**本案例 pro-workflow 的核心目的**：全方位 AI 开发工作流工具集——从会话启动（learn）到代码提交（commit）到会话收尾（wrap-up），覆盖 AI 辅助开发的完整生命周期

**我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|----------|--------|--------|--------|------------|
| MarkQWu/gstack | **高** | 同为多命令开发工作流工具集；同面向 AI 开发效率；同有 command + skill 分层思路 | gstack 定位 CEO/Designer/Eng Manager 视角（23 工具），pro-workflow 定位开发执行层（learn/commit/debug）；gstack 工具数量更少但视角更广 | **高** |
| MarkQWu/bureau | 中 | bureau 的 knowledge capture pipeline 与 session-handoff 概念高度相似 | bureau 是知识管理，pro-workflow 是开发工作流 | 中（session-handoff 模式可直接借鉴） |
| MarkQWu/graphify | 低 | 同有 orchestrator 模式参考 | graphify 是知识图谱工具，编排目的不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| pro-workflow 缺陷 | 检查命令 | gstack 预期风险 | 严重度 |
|------------------|---------|----------------|--------|
| 命令缺 description frontmatter | `grep -rL "^description:" ~/projects/gstack/.claude/commands/*.md` | **高风险**：gstack 有 23 个工具，如果批量编写，很可能有部分缺 description | 高（影响 `/help` 可见性） |
| agent 未注册到 plugin.json | 对比 `ls agents/` 与 `jq '.agents'` plugin.json | **中风险**：gstack 若有 agent，需确认全部注册 | 高（功能不可用） |
| agent 未声明 model | `grep -rL "^model:" ~/projects/gstack/.claude/agents/*.md` | **中风险**：model 声明常被遗漏 | 中（工具链读取失败） |
| version 不同步 | `grep '"version"' package.json plugin.json` | **低风险**：若有两个版本来源，需同步 | 低 |

**gstack 最优先的自查项**（按影响排序）：
1. 确认 23 个工具的命令文件全部有 `description:` frontmatter
2. 确认 plugin.json 注册了所有 agent 文件
3. 确认所有 agent 有 `model:` 声明

### 8.3 别人的更优方案

1. **领域**：orchestrator agent 的 phases 声明
   - **pro-workflow 的做法**：orchestrator.md 在 frontmatter 中显式声明 `phases:`，列出"分析 → 拆分子任务 → 分配 agent → 汇总结果"的执行阶段，Claude 在读取 agent 定义时就知道任务的宏观结构
   - **gstack 的可能现状**：gstack 的 CEO/Designer/Eng Manager 视角工具可能有类似的"思考框架"，但如果没有 phases 声明，只在内容里描述，Claude 读取效率较低
   - **如何借鉴**：在 gstack 的最高层协调 agent 中加 `phases:` frontmatter，例如 `phases: [需求分析, 角色分配, 并行执行, 整合审查]`

2. **领域**：session-handoff 的跨会话状态恢复
   - **pro-workflow 的做法**：`skills/session-handoff/SKILL.md`（score=98）定义了一个完整的 handoff 协议——会话结束时 `/wrap-up` 命令触发 skill 写入状态文件，下次会话 `/handoff` 命令读取并恢复
   - **bureau 的现状**：bureau 的 knowledge capture pipeline 也处理会话间知识传递，但可能没有标准化的 handoff 文件格式
   - **如何借鉴**：参考 pro-workflow 的 handoff 格式，在 bureau 中定义统一的 `session-state.json` schema，确保每次会话结束都有可恢复的状态

3. **领域**：smart-commit 的 suppressions list
   - **pro-workflow 的做法**：`skills/smart-commit/SKILL.md` 维护一个"不应提交内容"清单（调试代码标志、TODO 标记模式、本地路径特征），在 commit 前扫描
   - **gstack 的可能现状**：gstack 的工具集中如果有 commit 相关工具，可能没有这个质量门
   - **如何借鉴**：在 gstack 的 commit 相关工具中引用 smart-commit 的 suppressions 逻辑，或直接在 skill 中内嵌一个简化版清单

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：工具集的视角完整性
   - **gstack 的做法**："23 opinionated tools that serve as CEO, Designer, Eng Manager..."——覆盖了决策层到执行层的完整视角，不只是开发者工具
   - **pro-workflow 的做法**：60 个 artifact 集中在开发执行层（learn/commit/debug/doctor），缺少面向产品/设计视角的工具
   - **意义**：gstack 的横向覆盖度比 pro-workflow 更广；如果两个插件同时安装，gstack 补充了 pro-workflow 没有的决策层视角

2. **领域**：plugin.json 的注册完整性（如果 gstack 已做到）
   - **gstack 的潜在优势**：如果 gstack 的 plugin.json 把所有 agent 都注册了（包括类似 cost-analyst 这样的分析型 agent），这是 pro-workflow 没做到的
   - **pro-workflow 的遗漏**：cost-analyst.md 和 permission-analyst.md 未注册，用户安装后根本不知道这两个 agent 存在
   - **意义**：manifest 完整性直接影响用户的功能发现路径；plugin.json 是用户与功能之间的目录，目录不全等于功能不存在

3. **领域**：命令 description 的完整性（如果 gstack 已做到）
   - **gstack 的潜在优势**：如果 gstack 的 23 个工具命令全部有 `description:` frontmatter，`/help` 命令可以完整显示工具列表，用户体验显著优于 pro-workflow（12 个命令不可见）
   - **意义**：description frontmatter 是最基础的可见性要求；做到了这一点，即使其他方面不如 pro-workflow，用户上手路径也更顺畅

---

## 八、术语表

### <a name="分层协作型工作流架构"></a>分层协作型工作流架构
> pro-workflow 采用的架构模式。三层结构：command 层（用户入口，接收 `$ARGUMENTS`，委托执行）→ skill 层（共享参考知识，多个 command/agent 共同读取）→ agent 层（专用执行者，由 orchestrator 路由调度）。第四层是 hooks 层（后台守护，在工具调用后自动触发，用户无感知）。与扁平架构的区别：command 不直接实现业务逻辑，而是通过 skill 引用 + agent 委托实现，这让每一层都可以独立更新而不影响其他层。

### <a name="manifest-discipline"></a>Manifest Discipline（清单纪律）
> 在 Claude Code 插件开发中，"清单纪律"是指每次新增 artifact 文件时，必须同步更新对应的注册清单（plugin.json 的 agents 数组、hooks.json 的脚本引用、命令的 description frontmatter）。违反清单纪律的典型症状：文件存在但工具链不知道（如 pro-workflow 的 2 个未注册 agent），或脚本存在但从不被触发（hooks.json 断链）。清单纪律是 NL artifact 的"编译"等价物——没有这一步，文件存在等于功能不存在。

### <a name="phases-声明"></a>Phases 声明
> agent frontmatter 中的 `phases:` 字段，描述 agent 执行复杂任务时的阶段划分。orchestrator.md（score=100）的 phases 声明让 Claude 在任务开始前就知道宏观执行步骤，避免在长任务中迷失方向。典型 phases 格式：`phases: [分析输入, 拆分子任务, 路由给专用 agent, 汇总结果, 质量验证]`。没有 phases 声明时，Claude 需要在内容里推断执行顺序，增加歧义。

### <a name="orchestrator-路由模式"></a>Orchestrator 路由模式
> 在多 agent 系统中，由一个中心 orchestrator agent 负责解析用户意图并路由给专用子 agent 的模式。pro-workflow 的 orchestrator.md 是范本：它不自己写代码，而是分析任务类型，决定委托给 debugger（调试问题）、scout（信息收集）还是 cost-analyst（成本分析）。与"平等多 agent"模式的区别：平等模式中每个 agent 都可以被用户直接调用；路由模式中用户只和 orchestrator 交互，子 agent 由 orchestrator 调度，用户不需要知道哪个 agent 处理了哪一步。

### <a name="session-handoff"></a>Session Handoff（会话交接）
> 在 Claude 上下文窗口耗尽时保留任务状态、在新会话中恢复的协议。pro-workflow 的 `skills/session-handoff/SKILL.md`（score=98）实现了这个协议：会话结束时 `/wrap-up` 命令触发 skill 将当前任务状态（未完成项、关键决策、下一步计划）写入 handoff 文件；新会话开始时 `/handoff` 命令读取文件并向 Claude 注入上下文。这是解决长期项目使用 AI 工具"断档"问题的实用模式。

### <a name="安全误报"></a>安全误报（Security False Positive）
> 安全扫描器检测到的问题，经人工审核后认定不构成实际风险的 finding。pro-workflow 的 CRITICAL finding——`commands/doctor.md:37` 的 `AKIAIOSFODNN7EXAMPLE`——是 AWS 官方文档中专门用于示例的测试密钥（不可用于实际 API 调用），被 secret-scan.js 检测为泄漏的 AWS 密钥，属于误报。对于安全门状态为 BLOCKED 的仓库，不能只看标签，需要人工检查每个 finding 的实际上下文。

### <a name="sql-注入风险"></a>SQL 注入风险（SQL Injection Risk）
> 当用户输入被直接拼接进 SQL 语句（而非通过参数化查询传递）时存在的安全漏洞。pro-workflow 的 `commands/learn.md` 第 207 行将用户输入直接插入 `sqlite3 INSERT` shell 命令，攻击者可构造含 `; DROP TABLE` 或 `'` 的输入破坏数据库或执行任意命令。在 Claude Code 的 NL artifact 中，凡是 command 接收用户输入（`$ARGUMENTS`）并将其传递给 shell 命令的地方，都需要考虑注入风险。修复方式：改用 here-string 参数化，或在传递前对特殊字符进行转义。
