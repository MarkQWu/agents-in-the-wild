# zebbern/claude-code-guide — 学习案例

**仓库**：https://github.com/zebbern/claude-code-guide
**Stars**：3921 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-07-06（基于当前 HEAD）
**主题标签**：`model-pinning`, `vague-quantifier`, `single-purpose`, `manifest-discipline`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[zebbern/claude-code-guide](https://github.com/zebbern/claude-code-guide) 是一个大型 Claude Code [subagent](#subagent) 集合仓库，目标是「为每个技术领域各配一个专家级 AI 助手」。截至当前 HEAD，仓库包含约 110 个专域 agent 文件（按 `snake_case.agent.md` 规范命名）和 76 个安全方向 skill 文件，涵盖从 DevOps、全栈开发、机器学习到渗透测试的全部主流技术栈。仓库主要作为「Claude Code 参考配置库」使用，用户 clone 后将 agents/ 目录内容复制到自己的 `.claude/agents/` 使用。

关键事实：
- **创建背景**：个人维护的公开配置仓库，无 package 发布，直接 copy-and-use
- **规模**：110 个 agent（audit 时 105 个，当前已扩充），76 个安全 skill
- **特殊性**：安全领域 skill（渗透测试、红队工具、Active Directory 攻击）与开发领域 agent 并列，在公开仓库中属罕见组合
- **生态位置**：与 addyosmani/agent-skills、obra/superpowers 类似，属于「个人经验集」型仓库

### 1.2 架构剖析

```
zebbern/claude-code-guide/
├── agents/              # 110 个专域 agent（snake_case.agent.md）
│   ├── README.md        # 人类阅读索引，非 agent 本体
│   ├── react_specialist.agent.md
│   ├── rust_engineer.agent.md
│   └── ... (107 more)
├── skills/              # 76 个安全方向 SKILL.md
│   ├── active-directory-attacks/SKILL.md
│   ├── pentest-checklist/SKILL.md
│   ├── top-web-vulnerabilities/SKILL.md
│   └── ... (73 more)
├── guides/              # 2 个 CLAUDE.md 示例（zebbern 和 Sabrina）
└── archive/             # 旧版本归档
```

- **文件类型分布**：110 个 agent / 76 个 skill / 2 个 guide / 1 个 README 索引
- **编排关系**：agent 之间是平列关系（无 router）；部分 agent 在 prompt 中互相引用（如 `microservices-architect` 引用 7 个其他 agent），但引用是文本描述而非 [frontmatter](#frontmatter) 声明
- **跨件契约**：agents 和 skills 之间无显式注册——两层相互独立，任何 agent 都可加载任何 skill，但没有任何声明约束这种关系

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「专家角色扁平化」——把每个专业方向独立为一个 agent，用户按需选择，无需理解系统整体
- **解决什么问题**：Claude Code 的默认状态是「全才」，但对专域任务（如 `.net core` 开发、Kubernetes 管理）缺乏深度。该仓库通过为每个域提供专门的 system prompt 来弥补这个缺口
- **Trade-off**：110 个独立 agent vs. 一个带路由的通用 agent。前者上手简单（select agent，开始干活），但维护成本高（110 个文件里的相同字段要同步维护），且缺乏跨 agent 上下文共享
- **认知模型**：作者把 Claude Code 中的 agent 视为「可换的角色帽子」——每顶帽子一个专家，戴上即用，不假设用户对 NL 编程有任何深度理解

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「专家角色扁平阵列（Flat Expert Role Array）」

把每个专业领域封装为独立 agent 文件，无层次，无 router，无共享状态。文件数量代替系统复杂度——覆盖的领域越多，文件越多。

模式特征清单：
- 特征 1：**无 router / 无 meta agent**——选哪个 agent 完全由用户决定，系统本身不做推断
- 特征 2：**agent 之间无运行时依赖**——每个 agent 自成一体，可独立复制使用
- 特征 3：**文件数 = 覆盖域数**——扩展能力 = 添加文件，删减能力 = 删除文件，极度线性
- 特征 4：**skill 层与 agent 层解耦**——skills 可自由加载但无声明约束
- 特征 5：**用文件系统命名约定（snake_case.agent.md）做类型标记**

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人开发者的多域工具箱 | ✅ 高度适用 | 按需用、按需加，无需学习路由逻辑 |
| 团队共享、需要保持一致性 | ⚠️ 谨慎 | 110 个文件的同步更新成本极高 |
| 需要 agent 协作完成复杂任务 | ❌ 不适用 | 架构无法支持跨 agent 上下文共享 |
| 快速 fork 然后裁剪出子集 | ✅ 适用 | 每个文件独立，删除任意子集不破坏其余 |
| 希望 Claude 自动选择合适专家 | ❌ 不适用 | 无 router，选择权在用户，不在系统 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：扁平阵列 | zebbern/claude-code-guide | 极低学习门槛，任何文件独立可用 | 无法自动路由，维护 110 个文件代价高 |
| Router + Channels | mikeyobrien/ralph-orchestrator | 自动分配专家，跨 agent 协作 | 配置复杂，新手上手成本高 |
| 单仓多 skill 平铺 | obra/superpowers | 轻量，skill 可组合 | 专域深度有限，无 agent 角色切换 |

### 2.4 改进空间

1. **当前问题**：所有 agent 缺少 `model:` 声明，运行时使用项目默认模型。**改进做法**：按任务复杂度分层声明——简单工具类用 `haiku`，架构审查类用 `sonnet`，安全审计类用 `opus`。**预期收益**：成本降低，同时高复杂任务的输出质量可控。

2. **当前问题**：`agents/README.md` 位于被扫描目录内，虽有文字注释说明「不是 agent」，但无 [frontmatter](#frontmatter) 保护，任何基于 NLPM 的扫描都会误判为 agent 定义文件。**改进做法**：添加 `name: specialized-domains-index` / `description: Index file, not an agent` 的 frontmatter，或移出 `agents/` 目录。**预期收益**：消除扫描误判，NLPM score +5。

3. **当前问题**：agent 间引用（如 microservices-architect 引用 7 个其他 agent）是文本描述，无法被 Claude Code 的 agent delegation 机制识别。**改进做法**：在 agent 描述中使用 `use the @<agent-name>` 语法触发真实的 subagent 调用。**预期收益**：实现真正的多 agent 协作。

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-16 当时得分 **86/100**（137 个工件）。

| 文件范围 | 当时分数 | 主要问题 |
|---|---|---|
| agents/README.md | 23/100 | 无任何 frontmatter（名称、描述、示例均缺失） |
| 所有 agent 文件（105 个） | 75–95/100 | 全部缺少 `model:` 字段（-5 each）+ 模糊量词 |
| skills/top-web-vulnerabilities/SKILL.md | 80/100 | 17 行模糊量词（命中 -20 上限） |
| skills/burp-suite-testing/SKILL.md | 88/100 | 6 行模糊量词 |
| 其余安全 skill（24 个） | 88–98/100 | 少量模糊量词 |

### 3.2 当时值得借鉴的模式

1. **广覆盖 + 深细分**：把 DevOps 拆成 `devops-engineer`、`kubernetes-specialist`、`deployment-engineer`、`performance-monitor` 四个不同角色——每个比「通用 DevOps」更精准。借鉴方式：在自己的仓库中，把笼统的「Backend Agent」拆成 API 设计、数据库、部署三个专项角色。

2. **安全技能文档化**：将渗透测试知识（Active Directory 攻击、SQL Injection 测试方法）封装为 SKILL.md，与开发 agent 共存。这把「查文档」的行为变成「调 skill」。借鉴方式：将自己常查的技术规范（如 OpenAPI 设计规范、SQL 最佳实践）写成 SKILL.md。

3. **交叉引用声明**：`microservices-architect` 在 prompt 正文中明确列出应与哪些其他 agent 配合——这是轻量级的意图声明，虽未被系统识别，但对人类读者有价值。

### 3.3 当时的缺陷

1. **`agents/README.md` 无 frontmatter 导致扫描器误判**。根本原因：README 文件放在 `agents/` 目录内，而所有 `.md` 文件都被扫描器视为候选 agent 定义。这暴露了「目录即类型边界」的隐式约定——一旦打破（放进一个人类文档），整个约定失效。**自查**：我的仓库中有没有 README.md 放在被 NLPM 扫描的目录里？

2. **所有 agent 缺少 `model:` 声明**。根本原因：作者批量创建 agent 时复用了同一个模板，该模板没有 `model:` 字段。批量创建的「效率收益」直接变成「批量技术债」。**自查**：我的 bureau/gstack 里的 agent 有没有全部声明 model？

3. **高密度模糊量词（top-web-vulnerabilities: 17 行，命中上限）**。根本原因：安全文档领域的行文风格习惯用「appropriate」「comprehensive」「robust」这类形容词，而这些词在 NLPM rubric 里一律被计为负分。模糊量词不只是写作风格问题——它意味着缺少可验证的标准。**自查**：我的 SKILL.md 中有没有「确保系统安全」「提供全面分析」这类表述？

### 3.4 当时的优化机会

1. 为所有 agent 批量添加 `model: claude-sonnet-4-5`（当时最新 stable），可用一行 `sed` 命令完成
2. 将 `agents/README.md` 移至根目录或添加最小 frontmatter
3. 对命中 -20 上限的 7 个文件（search-specialist、rails-expert 等）做模糊量词替换，每文件约 10 处改动

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| agents/README.md 无 frontmatter | `head -10 agents/README.md` | ⚠️ **部分处理**：README 正文加了「README.md is documentation only and is not an agent」的文字注释，但仍无 YAML frontmatter | 文字注释对人有效，但对 NLPM 扫描器无效；bug 技术上仍存在 |
| 所有 agent 缺 model 字段 | `grep -l "^model:" agents/*.md \| wc -l` | ❌ **仍存在**：110 个 agent 中 0 个有 `model:` 字段 | 最高优先级质量债未偿还，且随文件增加（105→110）债务扩大 |
| top-web-vulnerabilities 17 行模糊量词 | `grep -c vague_words skills/top-web-vulnerabilities/SKILL.md` | ✅ **已改善**：当前 5 行（从 17 行降至 5 行），降幅 70% | 作者有意识优化了模糊语言，但未完全清零 |

### 4.2 架构演进

从 audit 时（105 个 agent，`kebab-case.md`）到当前 HEAD（110 个 agent，`snake_case.agent.md`）：

1. **命名约定重构**：从 `react-specialist.md` 改为 `react_specialist.agent.md`。`.agent.md` 后缀是一个显式类型标记——文件名本身宣告了它是什么类型的工件，不再依赖目录位置隐式推断
2. **规模扩充**：新增约 5 个 agent（包括 `auth_specialist`、`code_archaeologist` 等新角色）
3. **skill 层扩充**：从 29 个安全 skill 增至 76 个，新增了 `academic-paper-reviewer`、`api-shape-explorer`、`audit-flow` 等非安全方向 skill
4. **README 文字警告**：在 `agents/README.md` 顶部加入「README.md is documentation only」声明

作者后来意识到了：命名约定是「目录即类型」隐式约定的替代方案——用后缀明示比用目录位置隐示更健壮。

### 4.3 新增的可学习模式

- **后缀类型标记**（`.agent.md`）：这是一个在其他仓库中很少见的约定。它使得文件类型可从文件名单独判断，无需上下文。这对需要跨目录引用的系统尤其有价值。

---

## 五、校准

### 5.1 我已经在做对的

1. bureau 仓库的 agent 有 `model: sonnet` 声明，不会陷入零模型声明的问题
2. bureau 的 SKILL.md 文件有 `<example>` 块（如 `recall/SKILL.md`、`review/SKILL.md`），而 zebbern 的 agent 几乎全部缺少 example
3. gstack 的 SKILL.md 有 `version`、`triggers` 等丰富 frontmatter，结构化程度高于 zebbern 的 agent

### 5.2 挑战 / 验证

**挑战**：这个案例让我意识到「文件数量」不等于「系统质量」。110 个 agent 的仓库 NLPM 只得 86 分，而 bureau 的 7 个 skill + 1 个 agent 的小型仓库结构上更严谨。规模扩张带来的技术债是非线性的——每添加一个 agent 都在放大「缺少 model 声明」这个缺陷的影响面。

**验证**：后缀类型标记（`.agent.md`）的做法被这个案例验证为值得学习的模式——它比目录位置更健壮。我目前的 bureau 用目录区分（`skills/`、`crew/`），但如果文件迁移到其他目录，类型边界会丢失。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否全部有 model 声明
find ~/.claude/agents/ /tmp/my-repos/MarkQWu-bureau/crew/ -name "*.md" 2>/dev/null \
  | xargs grep -L "^model:" 2>/dev/null
# 命中后怎么办：为每个缺失的 agent 添加 model: claude-sonnet-4-6（或对应合适 tier）

# 检查是否有 README.md 误入 agent 扫描目录
find ~/.claude/agents/ -name "README.md" -o -name "readme.md" 2>/dev/null
# 命中后怎么办：移出目录或添加最小 frontmatter（name: xxx-index）

# 检查模糊量词密度
grep -rn -c -E '\b(appropriate|comprehensive|robust|ensure|various|suitable|necessary|proper|effective|efficient)\b' \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | grep -v ":0"
# 命中后怎么办：用具体可测量的标准替换模糊形容词
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 agent 补充 `tools:` 声明，参照 zebbern 每个 agent 的工具列表
   - **为何可行**：bureau 当前的 `auditor/agent.md` 有 `tools: Read, Grep, Glob` 但未覆盖 Write
   - **第一步**：`cat /tmp/my-repos/MarkQWu-bureau/.claude/agents/auditor.md`，核查工具声明是否完整，约 5 分钟

2. **想法**：在 gstack 的 SKILL.md 中引入 `.skill.md` 后缀约定（仿照 zebbern 的 `.agent.md`）
   - **为何可行**：gstack 的 SKILL.md 分散在各子目录，文件类型只能靠目录名（`skills/`）推断
   - **第一步**：评估 gstack 的构建脚本（`bun run gen:skill-docs`）是否会因后缀变化而需要同步修改，约 15 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 zebbern/claude-code-guide 的核心目的**：为各技术域提供可直接复用的专家 agent 集合
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是技术专域工具集合 | gstack 是 skill-first（无 agent），zebbern 是 agent-first（无 orchestration） | 高 |
| MarkQWu/bureau | 低 | 都有 NL 工件 | bureau 是知识管理专用，zebbern 是通用开发辅助 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 全部缺 model 声明 | `grep -L "^model:" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | gstack 无 agent，skill 无需 model 声明；bureau agent 有 `model: sonnet` ✅ | 低（不适用 gstack，bureau 已处理） |
| README.md 误入扫描目录 | `find /tmp/my-repos/MarkQWu-bureau/.claude/ -name "README*"` | 未命中 ✅ | N/A |
| SKILL.md 高密度模糊量词 | `grep -c -E '\b(appropriate\|comprehensive\|robust\|ensure\|proper)\b' /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | gstack SKILL.md 多处命中（CHANGELOG.md 13 处，部分 SKILL.md 有 2 处） | 中 |

**命中后具体行动**：
- `gstack/CHANGELOG.md` 中的模糊量词多为叙述性文字，影响较低，暂不修改
- 如果 `gstack/spec/SKILL.md` 或 `retro/SKILL.md` 有模糊量词，在下次修改该 skill 时顺手替换，约 5 分钟

### 7.3 别人的更优方案

1. **领域**：Agent 后缀类型标记（`.agent.md`）
   - **本案例做法**：所有 agent 文件用 `snake_case.agent.md` 命名，文件名即类型声明（`ai-bridge/services/claude/...` 中的 JS 文件同理）
   - **我的项目现状**：bureau 用目录区分（`crew/auditor/agent.md`），gstack 用目录名（`skills/<name>/SKILL.md`）——类型边界依赖目录位置
   - **如何借鉴**：在新建 agent 文件时使用 `<name>.agent.md` 后缀约定；现有文件迁移成本较高，作为长期目标

2. **领域**：安全知识 skill 化
   - **本案例做法**：29-76 个安全方向 SKILL.md，把「参考资料」变成「可调用的上下文」
   - **我的项目现状**：bureau/gstack 均无安全相关 skill
   - **如何借鉴**：如果有常查的安全清单（OWASP Top 10、SQL 注入防御模式），可以写成一个 SKILL.md 放在 bureau 的 `skills/` 下

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill frontmatter 完整性（gstack）
- **我的做法**：gstack 的 SKILL.md 包含 `version`、`triggers`、`allowed-tools` 字段，结构化信息远比 zebbern 的 agent 丰富
- **本案例做法**：zebbern 的 agent 只有 `name`、`description`，无版本、无触发词
- **意义**：`triggers` 字段允许 Claude Code 自动识别调用时机（「spec this out」→ 触发 spec skill），zebbern 的 agent 完全依赖用户手动选择

---

## 八、术语表

### <a name="subagent"></a>subagent
> Claude Code 中可被另一个 agent 或主对话「调用」的子级 AI 助手。每个 subagent 有自己的 system prompt（存在 `.md` 文件里），在被调用时独立执行任务并返回结果。可以理解为「可插拔的专家角色」——主任务把某部分工作外包给专门的 subagent 处理。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`tools` 等）。Claude Code 读 agent/skill 文件时先解析 frontmatter 才能知道这个工件怎么注册和调用。缺少关键字段（如 `name`）会导致注册失败。

### <a name="router"></a>router
> 多 agent 系统中负责「任务分发」的中间层。接收用户请求后，根据请求内容决定调用哪个专门的 agent 处理。没有 router 的系统（如 zebbern/claude-code-guide）需要用户自己选择 agent；有 router 的系统（如 ralph-orchestrator）由 AI 自动推断。
