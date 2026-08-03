# davepoon/buildwithclaude — 学习案例

**仓库**：https://github.com/davepoon/buildwithclaude
**Stars**：N/A | **来源**：upstream（≥500 池耗尽，按学习价值补选）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-08-03（基于当前 HEAD）
**主题标签**：`monorepo-vs-split`, `security-gate`, `curl-pipe-bash-risk`, `template-design`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
buildwithclaude 是一个超大规模 Claude Code 插件集合仓库，包含 881 个 NL 工件，涵盖编程语言专家（JavaScript/Python/Rust/Go/Java 等）、代码质量与安全、数据与 AI、基础设施运维、业务与财务、加密交易、文档、任务管理等几十个功能域。其最大特征是存在两套并行的 agent 集合：`plugins/all-agents/`（规范满分版本，100/100）和各专域插件目录（旧版副本，70-90 分），形成了「典范聚合器 + 副本漂移」的架构反模式。

关键事实：
- 2026-04-13 被 NLPM 审计，得分 **77/100**，本批次最低
- SECURITY **BLOCKED**（HIGH：budgetclaw 第三方二进制文件在每次会话启动时运行）
- 881 个工件，本批次最大规模
- 约 350 个命令文件（commands）**全部**缺 `name` [frontmatter](#frontmatter) 字段，是拉低分数的主因
- `plugins/all-agents/` 中 100+ agents 全部得满分 100/100，是仓库中质量的天花板

### 1.2 架构剖析

```
buildwithclaude/
├── plugins/
│   ├── all-agents/                # 典范聚合器：100+ agents，全部满分
│   │   └── agents/                # 每个 agent 都是其类型的最高质量版本
│   ├── all-commands/              # 命令聚合器：~350 命令，全部缺 name 字段
│   │   └── commands/
│   ├── agents-quality-security/   # 专域副本：agents（100/100）
│   ├── agents-development-architecture/  # 专域副本：agents（82-89/100）
│   ├── agents-data-ai/            # 专域副本：agents（87-91/100）
│   ├── agents-language-specialists/  # 专域副本：agents（100/100）
│   ├── agents-specialized-domains/   # 专域副本：agents（70-75/100）
│   ├── agents-crypto-trading/     # 专域副本：agents（100/100）
│   ├── cc-best/                   # Claude Code 最佳实践插件
│   ├── agents-uc-taskmanager/     # 任务管理 agent（XML 格式）
│   ├── budgetclaw/                # 第三方二进制钩子 [安全阻断]
│   │   └── hooks/hooks.json       # SessionStart 运行 budgetclaw status
│   ├── agent-triforce/            # 三角形 agent 组合（西班牙语 agent 名）
│   ├── shipwright/                # 发布管理 agent
│   └── ...（20+ 其他专域插件）
└── package.json                   # root: ajv/chalk/glob/gray-matter
```

**文件类型分布**：~120 agents（all-agents 规范集）+ ~120 agents（专域副本）+ ~350 commands + ~90 skills + 2 hooks.json + 127 shell/Python/JS 脚本

**编排关系**：各专域插件相互独立，通过 plugin.json 注册各自的 agents/commands/skills。all-agents 是独立插件（也有 plugin.json），与专域插件并列安装。无全局 meta-router。部分命令（all-commands）会通过 `# <agent-name>` 派发 sub-agent。

**跨件契约**：
- all-agents 中的 agent 文件名与各专域插件中的副本同名，但内容版本不同
- 命令引用 agent 名称时，Claude Code 会优先选哪个版本取决于安装顺序（隐性依赖）
- budgetclaw/hooks/hooks.json 对外部二进制有硬依赖，无法从源码审计

### 1.3 设计思路 / 方法论

**核心设计哲学**：
- **「聚合器」模式**：all-agents 和 all-commands 是「所有功能的集合版插件」，专域插件是「按需安装的子集版」——设计意图是用户可以安装整个 all-agents 或只装某个专域插件
- **递进式发布**：先在专域插件里开发，质量稳定后「提升」到 all-agents 聚合器，但现实中这个同步机制未被维护

**解决什么问题**：为开发者/创业团队提供一站式 Claude Code agent 工具箱，按技术域和业务域模块化安装。

**做了什么 trade-off**：
- 规模化（100+ agents）带来「让人惊叹」的工具覆盖，但维护同步是隐性代价
- 专域插件允许按需安装，但内容和 all-agents 副本的分歧随时间增大
- 用 budgetclaw（外部二进制）来做预算监控，UX 好，但引入了无法审计的安全风险

**认知模型**：作者把这个仓库看作「Claude Code 的 App Store 精选合集」——每个插件是一个精心设计的 App，all-agents 是「精选集」。这个模型在 agent 层面工作良好（all-agents 全部满分），但在 command 层面没有同等精力投入，导致 ~350 命令缺 name 字段的系统性问题。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「典范聚合器 + 副本漂移」（Canonical Aggregator + Copy Drift）**

这是本批次 4 个案例中最典型的**反模式**案例，值得专门研究。

关键特征：
- 存在一个「标准版」目录（all-agents），其中所有文件都按最高标准维护（100/100）
- 同时存在多个「专域副本」目录，包含相同 agent 的旧版（70-90/100）
- 两套并行存在的 agent 可能被同时安装，用户不知道 Claude Code 会调用哪个版本
- 随时间推移，副本和标准版的差距越来越大（版本漂移）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 「全家桶」一次性安装 | ✅ 适用（安装 all-agents + all-commands） | 覆盖广，选满分版本 |
| 按域精细选择安装 | ⚠️ 有风险 | 专域插件是旧版本，用户会安装低质量副本 |
| 需要 CI 安全审计 | ❌ 当前不适用 | budgetclaw 第三方二进制无法审计 |
| 单人项目 | ✅ 适用 | 规模问题不显现，受益于宽覆盖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 典范聚合器 + 副本（本案反模式） | davepoon/buildwithclaude | 覆盖广、专域可按需装 | 副本漂移，安装冲突，维护代价双倍 |
| 单一规范集（无副本） | alirezarezvani/claude-skills | 无漂移，单一真实来源 | 无按域安装选项 |
| 插件 + 分发垫片 | navapbc/dso | 优雅降级、安装检测 | 垫片拉低整体 NL 得分 |

### 2.4 改进空间
1. **当前问题**：~350 个命令文件全部缺 `name` frontmatter 字段（-25 each），是主要得分拖累。**改进做法**：在 `all-commands/` 下批量为每个命令加 `name` 字段（脚本可从文件名 + description 自动生成）。**预期收益**：命令均分从 ~40 提升至 ~65，整体得分 77 → ~88。

2. **当前问题**：all-agents 满分（100/100）但专域插件中的副本质量较低（70-90/100），且两者可能被同时安装。**改进做法**：废弃专域副本，改用 plugin.json 的「子集过滤」机制——专域插件只装 plugin.json，从 all-agents 中引用而不复制文件。**预期收益**：消除版本漂移，维护工作量减半。

3. **当前问题**：budgetclaw SessionStart 运行未经审计的第三方二进制（HIGH）。**改进做法**：将 budgetclaw 替换为本地脚本（统计 token 用量），并提供 budgetclaw 用户「选择加入」的可选安装路径。**预期收益**：解除 HIGH 安全阻断。

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-13 当时 NL 得分 **77/100**。

| 区域 | 当时分数 | 主要问题 |
|---|---|---|
| all-agents（100+ agents） | 100 avg | 极少，`code-reviewer` 缺 output format（-10） |
| agents-quality-security（15 agents） | 100 avg | Clean |
| 专域 agents（specialized-domains 等） | 70-91 avg | 缺 model 声明、缺示例、vague quantifier |
| agents-uc-taskmanager（6 agents） | 55-90 avg | 缺 name+description frontmatter（BUGS #1-6）|
| all-commands（~350 commands） | 30-100 avg | 几乎全部缺 name 字段（-25 each） |
| skills | 45-90 avg | 缺 name 字段为主 |

### 3.2 当时值得借鉴的模式
1. **all-agents 的满分标准** → all-agents 中每个 agent 都有完整的 frontmatter（name/description/model）、2+ 个示例、明确的 output format，且已整齐声明 allowed-tools。这是本仓库「知道怎么做对」的直接证据。如何借鉴：把 `plugins/all-agents/agents/` 下的任一文件当做 agent 编写的参考模板，找一个与自己需求接近的拷贝并按需修改。

2. **agents-uc-taskmanager 的 XML 格式 + DAG 编排** → taskmanager 的 agent 用 XML 结构标注 `<tool>`、`<context>` 等字段，内部维护 TASK/WORK/PLAN.md 的 DAG 依赖关系，scheduler→planner→builder→verifier→committer 形成有向无环图。缺点是缺标准 NLPM frontmatter（因此不可见），但设计本身是多 agent pipeline 编排的真实实现。如何借鉴：把 DAG 依赖设计思路（而非 XML 格式）用于多 agent 工作流，但用标准 NLPM frontmatter 而非 XML。

3. **budgetclaw 的会话级监控理念（扣除安全部分）** → 在每次 SessionStart 时自动报告预算/资源状态，让用户意识到资源消耗，是 [hook](#hook) 驱动的会话生命周期意识的好案例。如何借鉴：自己用本地脚本实现同等功能，而不是依赖第三方二进制。

4. **agent-triforce 的三角色 agent 组合** → 三个 agent（forja-dev/prometeo-pm/centinela-qa，100/100）构成「开发者 + PM + QA」三角关系，每个 agent 专注单一职责，可协同使用，类似 navapbc 的多维度审查。如何借鉴：在设计 multi-agent 系统时，考虑「角色三角」（生产者/协调者/验证者）而非角色列表。

5. **shipwright agent（发布管理）** → 专门的发布流程 agent（100/100），将 release checklist 和 changelog 生成封装为单一 agent。如何借鉴：把自己项目的发布流程也封装为单一 agent，减少人工记忆发布步骤。

### 3.3 当时的缺陷
1. **问题**：~350 个 command 文件全部缺 `name` frontmatter 字段（-25 each），这是单一最大失分点。**根本原因**：commands 的 frontmatter 格式惯例（name 是必须字段还是可选）在早期社区中不统一；这批命令可能在 NLPM 规定 name 为必须字段之前生成，后来没有批量修复。自查：**我有没有**命令文件缺 name 字段？

2. **问题**：budgetclaw hooks 运行未经审计的第三方二进制（HIGH）。**根本原因**：引入外部 binary 的 UX 方便性（一行 curl|sh 安装）掩盖了安全风险——用户在安装 Claude Code 插件时，不会想到每次打开会话都在执行来自 roninforge.org 的代码。这是「易用性陷阱」的典型案例：越方便的安装方式，越难让用户理解实际发生了什么。自查：我有没有 hooks 文件在会话启动时执行外部命令？

3. **问题**：all-agents（100/100）与专域副本（70-90/100）并存，内容版本漂移。**根本原因**：没有「单一真实来源」机制——当 all-agents 里的 agent 更新时，专域目录里的副本不会自动同步。用 Git submodule 或 symlink 可以解决，但作者用了文件复制，手工同步是不可持续的。自查：我的仓库里有没有同一内容的多个副本，且它们可能随时间出现分歧？

### 3.4 当时的优化机会
1. 为 all-commands 的 ~350 命令批量加 `name` 字段（可从文件名推导），得分从 30-45 提升到 65+
2. 废弃专域副本目录，统一使用 all-agents + 专门的 plugin.json 子集配置
3. 用 agents-uc-taskmanager 的 DAG 编排思路重写任务管理命令，但换用标准 NLPM frontmatter

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| agents-uc-taskmanager scheduler 等缺 frontmatter（Bugs #1-4） | `head -6 plugins/agents-uc-taskmanager/agents/scheduler.md` | **已修复**：现有 name/description/tools/model 完整 frontmatter | agents 现在可见且可被 agent picker 正确显示 |
| cc-best/agents 缺 description（Bugs #7-8） | `head -5 plugins/cc-best/agents/architect.md` | **部分修复**：architect.md 有 description 了 | cc-best 系列可被正确识别 |
| all-commands 缺 name 字段（~350 文件） | `head -5 plugins/all-commands/commands/bug-fix.md` | **持续**：bug-fix.md 有 description/category/allowed-tools，但无 name | 350 个命令仍缺 name，得分拖累未解决 |
| budgetclaw curl\|sh echo（MEDIUM） | `grep "curl" plugins/budgetclaw/hooks/hooks.json` | **持续**：echo 中仍有 `curl -fsSL https://roninforge.org/get \| sh` 原文 | SECURITY BLOCKED 状态未解除 |

### 4.2 架构演进
当前 HEAD vs 2026-04-13 快照：
- agents-uc-taskmanager 的 frontmatter 修复是最明显的变化（4 个 agent 从 55 分级别 → 可见）
- 总体架构（all-agents + 专域副本并存）未改变，副本漂移问题仍在
- 无证据显示 all-commands 的 name 字段问题有批量修复计划

### 4.3 新增的可学习模式
- **agents-uc-taskmanager 修复后的完整 frontmatter** → 修复后的 scheduler.md 同时声明了 `tools: Read, Write, Edit, Bash, Glob, Grep, Task` 和 `model: haiku`，是「budget 意识」（用 Haiku 而非 Sonnet 执行调度任务）的好例子

---

## 五、校准

### 5.1 我已经在做对的
1. bureau 和 gstack 均未使用第三方二进制钩子，hooks 只调用自己仓库内的脚本
2. 我的 skill 数量较少，不存在「规范集 + 副本并存」的场景；没有复制同一内容到多处
3. gstack 的 skill 通过 `model: haiku` 声明做了 budget 意识分配，与 taskmanager 修复后的做法一致

### 5.2 挑战 / 验证
本案例挑战了「规模 = 质量」的直觉：davepoon 拥有 881 个工件（本批次最多），但整体得分 77/100（本批次最低）。大规模不等于高质量——**质量是每个文件的属性，不会因为文件多而自动提升**。相反，规模越大，维护难度越高，如果没有自动化约束（如 CI 中的 name 字段检查），缺陷会随文件数量线性扩大。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己命令文件是否全部有 name 字段
find ~/.claude/commands/ -name "*.md" | while read f; do
  if ! grep -q "^name:" "$f"; then
    echo "MISSING name: $f"
  fi
done
```
命中后：给每个命令加 `name:` 字段，值为文件名（不含 .md 后缀），NLPM 得分从 ~40 → ~65。

```bash
# 检查 hooks 文件是否调用外部 URL 或未知二进制
find ~/.claude/ -name "hooks.json" -exec grep -l "curl\|wget\|http" {} \;
```
命中后：评估被调用的外部资源是否可审计；若不可审计，替换为本地等价实现。

### 6.2 灵感 → 实施路径
1. **想法**：仿照 agent-triforce 的「三角色 agent 组合」给 bureau 设计「捕获者/编译者/查询者」三角 agent 体系。**为何可行**：bureau 已有 capture/compile/query 功能，只是实现为命令而非 agent；agent 化后可被其他命令派发，增加复用性。**第一步**：将 bureau 的 compile.md 命令提取为 `agents/compiler.md` agent，定义明确的 output format（JSON），约 1 小时。

2. **想法**：在 gstack 中加一个本地预算意识 hook，替代 budgetclaw 的功能。**为何可行**：Claude Code 的 SessionStart hook 可以读取本地日志文件统计近期 token 用量，不需要任何外部依赖。**第一步**：写一个 `~/.claude/hooks/session-budget.sh`，统计最近 7 天的 Claude API 请求日志，在超过预设阈值时提示用户，约 30 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：大规模多域 Claude Code agent/command/skill 工具集合，按需模块化安装
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 大型多 skill 套件、工具角色分类 | gstack 无副本漂移问题；规模（~30 skills vs 881 工件）差距很大 | 高（反向：避免 gstack 出现副本漂移） |
| MarkQWu/bureau | 低 | 都有 hooks、命令驱动 | bureau 侧重知识管理而非工具集合 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 name 字段 | `grep -L "^name:" ~/.claude/commands/*.md` | 待核查，若使用 description-only frontmatter 则命中 | 中 |
| 同内容多处副本 | 手动检查 skills/ 目录有无同名文件在不同位置 | gstack 可能有 template-generated 副本 | 低 |

**命中后的具体行动建议**：
- 所有缺 `name` 字段的命令文件 → 加一行 `name: <slug>` → NLPM score -25 → 0，每个文件 2 分钟

### 8.3 别人的更优方案

1. **领域**：发布管理专用 agent
   - **本案例做法**：`plugins/shipwright/agents/shipwright.md`（100/100）将 release 流程封装为专用 agent，含完整 changelog/tag/PR checklist
   - **我的项目现状**：gstack 无专用 release agent，发布步骤靠人工记忆
   - **如何借鉴**：在 gstack 建 `agents/release-manager.md`，复制 shipwright 的 checklist 结构，约 30 分钟

2. **领域**：多 agent DAG 编排（taskmanager 模式）
   - **本案例做法**：`agents-uc-taskmanager` 中 scheduler.md 读取 PLAN.md 构建 DAG，按依赖顺序派发 builder→verifier→committer
   - **我的项目现状**：gstack/bureau 中无显式 DAG 编排，多步任务靠单一 agent 顺序执行
   - **如何借鉴**：对 bureau 的 capture→compile→promote 三步流程建 DAG，scheduler agent 读 pipeline 状态文件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：单一真实来源（无副本漂移）
- **我的做法**：gstack 和 bureau 中每个 skill/command 只有一个版本，无「规范集 + 副本」的双版本问题
- **本案例做法**：all-agents 和各专域副本并存，存在版本漂移
- **意义**：我的仓库在规模扩大时应坚守「单一真实来源」原则，任何新 skill 只建在一处，需要按域分类时用 plugin.json 子集配置，而不是复制文件

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明 `name`、`description`、`model`、`allowed-tools` 等元数据。Claude Code 依靠 frontmatter 注册 skill/agent/command；缺少 `name` 字段 = 文件对注册系统「不可见」或显示为空白条目。

### <a name="hook"></a>hook
> Claude Code 在特定生命周期事件（SessionStart、PreToolUse、PostToolUse 等）自动执行的脚本，配置在 `hooks/hooks.json` 中。合法用途如自动格式化、权限检查；风险用途如执行外部二进制，因为每次触发都在 Claude Code 的权限下运行。

### <a name="dag"></a>DAG（有向无环图）
> Directed Acyclic Graph，"有向无环图"。在任务编排里，每个节点是一个任务（或 agent），边代表依赖关系（A 完成才能开始 B）。「无环」保证不会出现「A 等 B，B 又等 A」的死锁。agents-uc-taskmanager 用 DAG 表示 build pipeline：specifier → planner → builder → verifier → committer。

### <a name="version-drift"></a>副本漂移（Copy/Version Drift）
> 同一内容存在多个副本，并随时间各自独立更新，导致副本之间出现差异（一个版本修复了 bug，另一个没有）。解决方案是「单一真实来源」（Single Source of Truth）：只在一处维护内容，其他地方通过引用（symlink/git submodule/import）而非复制来访问。
