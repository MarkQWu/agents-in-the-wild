# mbruhler/claude-orchestration — 学习案例

**仓库**：https://github.com/mbruhler/claude-orchestration
**Stars**：215 | **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-25（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `manifest-discipline`, `single-purpose`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-orchestration` 是一个专门为 Claude Code 设计的工作流编排插件，提供一套自定义的 DSL（[领域专用语言](#领域专用语言)）语法和多个 skill/agent/command，帮助用户用自然语言描述、创建和执行多步 AI 工作流。

关键事实：
- 作者 mbruhler（全名 mbroler），个人项目
- 通过 Claude Code 插件系统分发，安装后提供 `/orchestration:*` 系列命令
- 当前 215 颗星，属于中等规模个人插件
- 核心卖点：用 YAML-like 自定义语法编写工作流 `.flow` 文件

### 1.2 架构剖析
**目录结构**：
```
claude-orchestration/
├── commands/          # 10 个斜线命令（pull/init/orchestrate/run/help/menu/explain/template/examples/create）
├── agents/            # 3 个 agent（workflow-socratic-designer, workflow-syntax-designer, security-auditor）
├── skills/            # 9 个 skill 目录（各含 SKILL.md + 子文档）
│   ├── creating-workflows/
│   ├── executing-workflows/
│   ├── managing-agents/
│   ├── managing-temp-scripts/
│   ├── designing-syntax/
│   ├── debugging-workflows/
│   ├── using-orchestration/
│   ├── using-templates/
│   └── creating-workflows-from-description/
└── docs/              # 6 个子目录（core/features/plans/reference/testing/topics）
```

**文件类型分布**：10 个 command / 3 个 agent / 9 个 skill / 1 个 plugin.json

**编排关系**：用户调用 `/orchestration:orchestrate` → 路由到四个子命令之一 → 触发相应 skill → 可选调用 agent（Socratic 对话收集需求 → Syntax designer 生成 .flow 语法）。

**跨件契约**：`creating-workflows/SKILL.md` 与 `workflow-socratic-designer` agent 共享对 `docs/TEMP-SCRIPTS-DETECTION-GUIDE.md` 的引用，但该文件当前缺失。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「提问优于假设」——通过 Socratic 对话 agent 收集用户需求，不直接生成，而是先问清楚再生成
- **解决什么问题**：Claude Code 原生缺乏跨会话工作流状态管理机制；该插件用一套 `.flow` 语法填补这个空白
- **Trade-off**：选择了一套私有的 DSL 语法，上手成本高但表达力强；命令依赖 `$ARGUMENTS` 模板，未强制要求 `name:` [frontmatter](#frontmatter) 字段，牺牲 AI 发现性换取更简洁的文件结构
- **认知模型**：把 AI agent 当作可编程节点，工作流是节点间的有向图；这与传统编程的 pipeline 思维一致

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「Socratic Agent + 自定义 DSL 双层架构」

核心特征：
- 特征 1：命令层（commands）做路由与入口，不含业务逻辑
- 特征 2：Agent 层用 Socratic 问答收集结构化需求，而非直接执行
- 特征 3：Skill 层承载可复用的执行知识（DSL 语法、模板库）
- 特征 4：`.flow` 文件作为跨会话持久化载体

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要反复执行的多步 AI 工作流 | ✅ 高度适用 | DSL 文件可复用，避免每次重复描述流程 |
| 单次、一次性任务 | ❌ 不适用 | 引入 DSL 学习成本过高，直接对话更快 |
| 团队共享工作流 | ⚠️ 部分适用 | 社区 registry 存在但生态小，文档不足 |
| 企业安全审计场景 | ✅ 可用 | 新增的 security-auditor agent 有明确的 OWASP 框架指引 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Socratic + DSL（本仓库） | claude-orchestration | 需求准确、流程可复用 | DSL 学习门槛高，维护成本高 |
| 直接 task agent 编排 | agentsys, shinpr-claude-code-workflows | 上手即用，无需学 DSL | 工作流不可持久化，复用性差 |
| 模板 + 参数注入 | gstack, mattpocock-skills | 零学习成本，文件直观 | 工作流逻辑受限于模板复杂度 |

### 2.4 改进空间
1. **当前问题**：10 个 command 文件均缺少 `name:` 字段 **改进做法**：为每个 command 添加 `name: orchestration:pull` 等字段 **预期收益**：Claude Code 注册时不再依赖文件名推断，AI 可精确发现命令
2. **当前问题**：`examples/` 子目录在 `executing-workflows/SKILL.md` 中被引用但不存在 **改进做法**：创建 5 个 stub 文件（sequential.md / parallel.md / conditional.md / error-handling.md / checkpoints.md） **预期收益**：消除 5 个死链，用户调用时不再遇到找不到示例的错误
3. **当前问题**：namespace 不统一（`orchestration:creating-workflows` vs `managing-agents`）**改进做法**：统一全部 skill 使用 `orchestration:` 前缀 **预期收益**：消除 AI 配置混淆，便于按命名空间过滤

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时得分 **82/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/pull.md | 58 | 缺 name (-25)、无 allowed-tools (-5)、无空输入处理 (-10) |
| commands/template.md | 61 | 缺 name (-25)、无 allowed-tools (-5)、模糊词 (-4) |
| commands/menu.md | 64 | 缺 name (-25)、无 allowed-tools (-5) |
| agents/workflow-socratic-designer.md | 73 | Write 工具未声明、无 model、无 Output 节 |
| skills/executing-workflows/SKILL.md | 96 | 引用 examples/ 子目录死链 (-4) |

### 3.2 当时值得借鉴的模式
1. **Socratic 问答驱动设计** → 先问再做，避免需求偏差 → `workflow-socratic-designer.md` → 可借鉴到任何需求收集类 skill
2. **skill 内嵌子文档（参考注入）** → `creating-workflows/` 下有 `patterns.md`、`socratic-method.md` 等子文档，SKILL.md 引用它们扩展上下文 → 可借鉴「模块化参考资料」的 skill 组织方式
3. **docs/ 深度知识库** → 大量参考文档按 core/features/plans/reference 分类 → 对复杂插件，这种分层文档组织很值得学习

### 3.3 当时的缺陷
1. **Write 工具在 tools 列表中缺失**：`workflow-socratic-designer` 声明 `tools: [Read, Grep, Task]` 但在第 294 行调用 `Write` 工具。根本原因：手动维护工具列表易遗漏，没有自动校验机制。**自查**：我的 agents 是否也有类似的工具声明遗漏？
2. **10 个命令全部缺少 `name:` 字段**：根本原因：作者可能认为文件名即命令名，省略了冗余声明，但这违背了 [frontmatter](#frontmatter) 契约。自查：我的命令文件有完整的 frontmatter 吗？
3. **硬编码作者绝对路径**：`/Users/mbroler/...` 出现在 agent 指令中。根本原因：在作者自己机器开发时直接 copy-paste，忘记替换为仓库相对路径。**这是每个人开发本地插件时都可能犯的错误。**

### 3.4 当时的优化机会
1. 为所有 command 添加 `name:` 和 `allowed-tools:` 字段（批量修复，高性价比）
2. 为 agents 添加 `model:` 声明（两个 agent 都缺）
3. 创建 `docs/TEMP-SCRIPTS-DETECTION-GUIDE.md` 消除死引用

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：Write 工具未声明 | grep "tools:" agents/workflow-socratic-designer.md | `tools: [Read, Grep, Task]`，Write 仍缺失 → **持续** | 高优先级未修复，agent 运行时会失败 |
| Bug #3：硬编码绝对路径 /Users/mbroler/... | grep "mbroler\|/Users/" | 无命中 → **已修复** | 可移植性问题已解决 |
| Bug #4：TEMP-SCRIPTS-DETECTION-GUIDE.md 缺失 | grep + find | creating-workflows/SKILL.md 仍引用该文件，docs/ 下仍不存在 → **持续** | 死链依然有效 |
| Bug #5：examples/ 子目录缺失 | ls skills/executing-workflows/examples/ | 目录不存在 → **持续** | 5 个死链未修复 |

### 4.2 架构演进
**过去**：代码库结构相对扁平，agents 无 `name:` 字段，无 `namespace:` 字段。
**现在**：
- agents 新增 `namespace:` 字段（如 `namespace: orchestration:workflow-socratic-designer`）
- agents 现在有 `name:` 和 `usage:` 字段
- `skills/executing-workflows/` 下直接增加了 `parallel.md`、`checkpoints.md` 等子文档（但 `examples/` 子目录依然缺失）
- 新增了第三个 agent `security-auditor.md`，且这个新 agent 正确声明了 `tools: [Read, Grep, Task, Bash]`

这说明作者已意识到 agent 的 `name:` 和 `namespace:` 重要性，新 agent 写得更规范；但老 agent 的工具声明 bug 仍未回填修复。

### 4.3 新增的可学习模式
- **`namespace:` 字段作为 agent 定位器**：新 agent 用 `namespace: orchestration:security-auditor` 配合 `usage: "Use via Task tool with subagent_type: 'orchestration:security-auditor'"` 形成了完整的调用说明
- **安全审计 agent 专项化**：将安全审计从通用 workflow 中分离，作为独立 agent，遵循单职责原则

---

## 五、校准

### 5.1 我已经在做对的
1. **skill 子文档结构**：我的 bureau/skills/recall/ 也使用 SKILL.md + 子文档的组织方式
2. **frontmatter 完整性**：我的 gstack 所有 SKILL.md 都有 `name:` 字段
3. **命令式 description**：我的 skills 使用「Use when...」格式，符合触发词规范

### 5.2 挑战 / 验证
这个案例验证了一个我之前犹豫的做法：**agent 的 `tools:` 列表必须完整列出所有使用的工具**。作者显然知道自己会用 Write，但没有加进去，结果 bug 持续了至少 3+ 个月到今天。这强化了我的认知：工具声明不是可选项，是合约（contract）。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否有 tools 声明
grep -rn "^tools:" /tmp/my-repos/MarkQWu-bureau/skills/ /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null | head -20

# 检查我的 commands 是否有 name 字段
find /tmp/my-repos/MarkQWu-gstack -name "*.md" | xargs grep -L "^name:" 2>/dev/null | head -10
```

命中后：为缺少 `name:` 的命令文件补充 frontmatter 中的 `name:` 字段。

```bash
# 检查我的 skills 是否有死链引用（引用了不存在的子文档）
grep -rn "See.*\.md\|read.*\.md\|→.*\.md" /tmp/my-repos/MarkQWu-bureau/skills/ 2>/dev/null | while IFS=: read f l content; do
  ref=$(echo "$content" | grep -oP '[a-z/-]+\.md')
  [ -n "$ref" ] && [ ! -f "$(dirname $f)/$ref" ] && echo "DEAD: $f:$l → $ref"
done
```

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 的 skills 加上 `namespace:` 字段 **为何可行**：规范化调用路径，避免同名冲突 **第一步**：修改 bureau/skills/capture/SKILL.md，在 frontmatter 加 `namespace: bureau:capture`，30 分钟可完成
2. **想法**：实现 Socratic 问答模式的 skill **为何可行**：需求收集是高频需求 **第一步**：在 gstack 新建 `require-context/SKILL.md`，内容参考 `workflow-socratic-designer.md` 的结构，去除 DSL 部分，保留 Socratic 对话流程

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 mbruhler/claude-orchestration 的核心目的**：为 Claude Code 提供可持久化的多步 AI 工作流编排能力

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件，都有 skill 组织 | bureau 专注知识库，orchestration 专注流程编排 | 中 |
| MarkQWu/gstack | 低 | 都是 Claude Code skills | gstack 是工具集合，无编排 DSL | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agents 缺 `model:` 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | **全部 7 个 bureau skills 命中** | 中 |
| 命令缺 `name:` | `find /tmp/my-repos/MarkQWu-gstack -name "*.md" \| xargs grep -L "^name:"` | gstack skills 均有 name 字段，**命中 0** | 无 |
| skills 死链引用 | 手动核查 | bureau/skills 引用均为已知存在文件，**命中 0** | 无 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/skills/capture/SKILL.md` → 在 frontmatter 加 `model: claude-sonnet-4-5` → 10 分钟 × 7 个文件 = 70 分钟可完成

### 8.3 别人的更优方案

1. **领域**：参考注入式 skill 组织（子文档模式）
   - **本案例做法**：`skills/creating-workflows/` 下有 `patterns.md`、`socratic-method.md`、`examples.md`、`temp-agents.md`、`custom-syntax.md` 五个子文档，SKILL.md 引用它们
   - **我的项目现状**：bureau/skills/recall/SKILL.md 是单文件，所有知识压缩在一个文件里
   - **如何借鉴**：把 recall 拆分为 `SKILL.md`（主触发文件）+ `patterns.md`（查询模式）+ `trust-tiers.md`（信任级别规则）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：避免模糊量词
- **我的做法**：bureau/skills/capture/SKILL.md 中无 "comprehensive"、"appropriate" 等模糊词（`grep -c "comprehensive\|appropriate"` = 0）
- **本案例做法**：多个 skills 有模糊量词扣分（-4 to -8），如 `managing-agents/SKILL.md` 的 "Make prompts comprehensive and specific"
- **意义**：写精确指令是我的优势，若有机会给上游 PR，可以专门针对这类模糊词提 fix

---

## 八、术语表

### <a name="领域专用语言"></a>领域专用语言
> 英文 DSL（Domain-Specific Language）。为某一特定任务设计的小型编程语言，只能做那一件事，但做得很好。比如 SQL 是专门用来查数据库的语言。claude-orchestration 的 `.flow` 文件格式就是一种 DSL——只能用于描述 Claude Code 工作流，不能拿来写通用程序。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。缺少 `name:` 字段时，Claude Code 只能靠文件名来猜命令名，在某些场景下会失败。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。
