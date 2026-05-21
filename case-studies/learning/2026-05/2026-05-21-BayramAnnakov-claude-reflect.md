# BayramAnnakov/claude-reflect — 学习案例

**仓库**：https://github.com/BayramAnnakov/claude-reflect
**Stars**：1000 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-21（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `manifest-discipline`, `single-purpose`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-reflect 是一个 Claude Code 插件，核心命令 `/reflect` 在会话结束时触发，自动从对话历史中提取用户对 Claude 的纠正行为，并将这些"教训"写回 `CLAUDE.md`。版本 3.1.0，1000 Stars，是今日批次中 NL 质量得分最高的仓库（99/100）。

仓库目录布局（HEAD，2026-05-21）：

```
.claude-plugin/marketplace.json, plugin.json
commands/reflect.md, reflect-skills.md, skip-reflect.md, view-queue.md
hooks/hooks.json
scripts/capture_learning.py, semantic_detector.py, read_queue.py ...
scripts/lib/reflect_utils.py, semantic_detector.py
scripts/legacy/（shell 历史版本，保留为参考）
tests/test_reflect_utils.py, test_semantic_detector.py,
      test_memory_hierarchy.py, test_tool_errors.py, test_integration.py
.github/workflows/test.yml
SKILL.md, CLAUDE.md, CHANGELOG.md, DISTRIBUTION.md, RELEASING.md
```

### 1.2 架构剖析

插件实现了一个**四层记忆层次**（memory hierarchy）：

| 层次 | 存储位置 | 生命周期 |
|------|----------|----------|
| 即时上下文 | 当前对话 | 会话结束即消失 |
| 会话记忆 | 学习队列（JSONL） | 跨会话，待确认 |
| 项目记忆 | `CLAUDE.md` | 项目级持久 |
| 跨项目记忆 | `SKILL.md` | 全局持久 |

四个 hooks 构成捕捉管道：

- **SessionStart**（`session_start_reminder.py`）：启动时提示 Claude 本会话是否有待处理学习
- **UserPromptSubmit**（`capture_learning.py`）：每次用户输入时检测纠正行为
- **PreCompact**（`check_learnings.py`）：压缩前检查学习队列，防止上下文截断导致丢失
- **PostToolUse/Bash**（`post_commit_reminder.py`）：commit 后提醒运行 `/reflect`

`semantic_detector.py`（核心）：调用 `subprocess.run(["claude", "-p", ...])` 对用户消息分类，判断是否是"值得捕捉的纠正"。这是递归 AI 调用——用 Claude CLI 分析 Claude 的行为。

### 1.3 设计思路 / 方法论

插件的根本洞见：**用户纠正是隐性知识（tacit knowledge）**。在没有系统化手段的情况下，每次对话中 Claude 犯错、用户纠正、Claude 改正这一过程产生的知识，在对话结束时完全蒸发。`/reflect` 是"关闭知识循环"的机制。

设计哲学体现在四个约束上：
1. 单一命令（`reflect.md`）负责闭环，其他命令只是辅助
2. Python stdlib only，无 pip 安装依赖
3. legacy/ 目录保留 shell 旧实现作为参考，但 Python 版本是规范版本
4. 正式软件包管理：`CHANGELOG.md` + `DISTRIBUTION.md` + `RELEASING.md`

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**经验积累型（Experience Accumulation）**：与"任务执行型"或"工具扩展型"插件不同，claude-reflect 属于 AI 内省（introspection）类插件。它不执行外部任务，而是观察 AI 自身的行为并写回自身的行为约束。

### 2.2 适用场景

- 团队长期使用 Claude Code 于同一代码库，希望减少重复纠错
- 有明确偏好风格的个人开发者（缩进、命名约定、注释风格）
- 需要跨会话"学习期望"的场景（例如：同一仓库的 code review 准则）

### 2.3 与其他架构对比

| 维度 | claude-reflect | 普通 CLAUDE.md 手写 | 外部记忆插件 |
|------|---------------|---------------------|-------------|
| 知识来源 | 自动从纠正中提取 | 手动编写 | 向量数据库 |
| 维护成本 | 低（自动） | 高（手动） | 高（需基础设施） |
| 准确性 | 依赖语义检测 | 人工精确 | 依赖检索质量 |
| 可移植性 | 跟仓库走 | 跟仓库走 | 需要外部服务 |

### 2.4 改进空间

1. `plugin.json` 至今（v3.1.0）缺失 `hooks` 字段，若 Claude Code 未来改变自动发现行为，4 个 hooks 将静默失败
2. `reflect.md` 的 `$0` 路径解析与 `skip-reflect.md` 的 `${CLAUDE_PLUGIN_ROOT}` 不一致，是维护债务
3. 语义检测器的递归调用引入了提示注入攻击面（Medium 级安全风险）

---

## 三、过去审查发现（历史快照）

### 3.1 当时质量评分（NLPM）

| 文件 | 分数 |
|------|------|
| commands/reflect.md | 94/100 |
| commands/view-queue.md | 97/100 |
| commands/reflect-skills.md | 98/100 |
| commands/skip-reflect.md | 100/100 |
| SKILL.md | 100/100 |
| CLAUDE.md | 100/100 |
| hooks/hooks.json | 100/100 |
| plugin.json（manifest） | 100/100（但有跨组件问题） |

**综合 NL 质量：99/100**，今日批次最高分。

### 3.2 当时值得借鉴的模式

- **SKILL.md + CLAUDE.md 双 100**：说明项目级约定和技能描述写得极其清晰、无模糊词
- **hooks.json 100**：4 个 hook 全部触发条件明确、动作精确
- **skip-reflect.md 100**：单一职责命令，使用 `${CLAUDE_PLUGIN_ROOT}` 正确解析路径
- **正式发布管理**：CHANGELOG + DISTRIBUTION + RELEASING——业界少有的把 Claude Code 插件当软件产品管理的做法

### 3.3 当时的缺陷

**Bug #1（高严重度）**：`plugin.json` 缺少 `hooks` 字段。`CLAUDE.md` 中写道 manifest "points to hooks"，但实际 `grep "hooks" .claude-plugin/plugin.json` 返回空。若 Claude Code 不自动发现 `hooks/hooks.json`，则 4 个 hooks 在 fresh install 时静默失败。

**Bug #2（路径解析不稳）**：`reflect.md` 第 20 行使用 `$0` 解析路径：
```bash
!python3 "$(dirname "$(dirname "$(readlink -f "$0")")")/scripts/read_queue.py" 2>/dev/null || echo "[]"
```
在 Claude Code 的 `!` 上下文中，`$0` 是 shell 解释器路径，而非技能文件路径，因此会 fallback 到 `|| echo "[]"`，静默返回空数组。

**质量问题**：
- `reflect.md`：3 处模糊限定词——"appropriate heading"、"relevant section header"、"makes semantic sense"
- `view-queue.md`：`allowed-tools` 声明 `Read`，但所有步骤均通过 `Bash`（`python3 read_queue.py`）实现，Read 从未实际使用
- `reflect-skills.md`：`"Relevant tools based on workflow"` 指导 Claude 模糊选取工具

### 3.4 当时的优化机会

- 将 `reflect.md` 中 `$0` 改为 `${CLAUDE_PLUGIN_ROOT}`（参照 `skip-reflect.md` 的正确做法）
- 在 `plugin.json` 补充 `hooks` 字段：`"hooks": "hooks/hooks.json"`
- 将 3 处模糊限定词替换为具体指示（例如：将 "appropriate heading" 改为 "以 `## ` 开头的二级标题"）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-06 | 2026-05-21（HEAD v3.1.0） | 状态 |
|------|-----------|--------------------------|------|
| plugin.json 缺 hooks 字段 | 存在 | `grep "hooks" .claude-plugin/plugin.json` → 空 | **未修复** |
| reflect.md `$0` 路径 | 存在 | 第 20 行仍为 `readlink -f "$0"` | **未修复** |
| view-queue.md 冗余 Read | 存在 | 现声明 `allowed-tools: Bash, Read` | **部分改善**（Bash 补充声明，但 Read 是否实际使用未知） |
| 3x 模糊限定词 | 存在 | 未跟踪 | **未知** |

关键观察：两个最严重的 bug（hooks 字段缺失、`$0` 路径）在两个月内未修复，且版本号从旧版升到 3.1.0。这表明**作者实际运行时未观察到 impact**，可能的解释：Claude Code 确实会自动发现插件根目录下的 `hooks/hooks.json`，以及 `$0` 的 fallback `|| echo "[]"` 对实际使用影响可以接受（队列为空的场景会正常退化）。

### 4.2 架构演进

从 April 到 May，仓库有两个明显的架构扩展：

1. **测试覆盖扩展**：新增 `tests/test_memory_hierarchy.py` 和 `tests/test_tool_errors.py`，以及 `.github/workflows/test.yml`（自动化 CI）。这表明作者在固化"四层记忆层次"模型——`test_memory_hierarchy.py` 的存在意味着这个层次模型已上升为正式的可测试接口，而非仅是设计文档中的描述。

2. **社区化**：`.github/FUNDING.yml` 出现，说明插件已开始寻求可持续化支持。

### 4.3 新增的可学习模式

**CI 测试流水线（test.yml）**：Claude Code 插件社区中极为罕见的自动化测试。通常插件作者只有手动验证，甚至连 `tests/` 目录都没有。test.yml 的出现意味着：每次 push 都会运行 pytest，防止回归。

**内存层次的形式化**：`test_memory_hierarchy.py` 将"即时→会话→项目→跨项目"的四层模型编码为可断言的测试案例，是将设计哲学转化为可验证约束的典范。

---

## 五、校准

### 5.1 我已经在做对的

- **使用 CLAUDE.md 存储项目上下文**：这是插件的核心假设，我的工作习惯与之对齐
- **Python stdlib 优先**：避免额外 pip 依赖，降低安装摩擦——这与 `scripts/` 目录的实现策略完全一致
- **单一职责命令**：每个 `.md` 只做一件事（`skip-reflect.md` 仅跳过，`view-queue.md` 仅查看）

### 5.2 挑战 / 验证

**挑战一：递归 AI 调用的设计范式**。`semantic_detector.py` 用 `subprocess.run(["claude", "-p", ...])` 分析用户消息的语义，这是"用 Claude 分析 Claude 行为"的最简递归形式。我从未在自己的工作流中考虑过这种设计。这个模式的核心价值：用 AI 做分类，避免正则或规则系统的脆弱性。代价：引入提示注入攻击面（Medium 级）。

**挑战二：未修复的 bug 不代表 bug 无影响**。`plugin.json` 缺失 `hooks` 字段、`$0` 路径问题在 v3.1.0 仍未修复，但 Star 数达到 1000。可能解释：Claude Code 平台层面的行为比规范文档更宽容，自动发现机制弥补了 manifest 的不完整。这提示我：在生产插件中，实测比规范更重要。

**验证**：单文件 SKILL.md + 4 commands + hooks.json 就能达到 99/100。质量来自设计清晰度，不来自文件数量。我的插件如果在每个文件中消除模糊限定词，分数可能比现在更高。

---

## 六、行动

### 6.1 自查动作

- [ ] 检查自己的插件 `plugin.json` 是否声明了 `hooks` 字段，对照 `hooks/hooks.json` 是否存在
- [ ] 全局搜索自己的 commands 中是否有 `$0`、`dirname "$0"` 等路径解析，改为 `${CLAUDE_PLUGIN_ROOT}`
- [ ] 在 `allowed-tools` 声明的工具是否在步骤中真正用到——没用到的要么删除声明，要么在步骤中实际调用
- [ ] 在命令文本中搜索 "appropriate"、"relevant"、"reasonable"、"makes sense"——每一处都是待替换的模糊限定词
- [ ] 如果有 Python 脚本，确认是否仅用 stdlib，否则补充安装说明

### 6.2 灵感 → 实施路径

**灵感：在自己的工作流中实现"经验闭环"**

1. 仿照 `UserPromptSubmit` hook，在每次用户消息后记录纠正事件到 `learning_queue.jsonl`
2. 实现一个简化版 `semantic_detector.py`：只做二元分类（"是纠正" / "不是纠正"），用 `claude -p` 完成
3. 在 `reflect.md` 中调用队列读取，汇总后通过 Edit 工具写回 `CLAUDE.md` 的 `## Lessons Learned` 节
4. 验证：使用 `test_memory_hierarchy.py` 的测试思路，为每一层记忆写一个 pytest fixture

**灵感：把插件当软件产品管理**

即使是个人工具，也维护 `CHANGELOG.md` 和版本号。版本号让用户（包括未来的自己）能够理解"这是 breaking change 还是 patch fix"。3.1.0 的版本号意味着作者对向后兼容性有承诺。

---

## 七、术语表

| 术语 | 含义 |
|------|------|
| **PreCompact hook** | Claude Code 在压缩对话上下文（compaction）之前触发的 hook 事件。claude-reflect 用它在上下文截断前检查未处理的学习队列，防止知识丢失 |
| **语义检测器（semantic detector）** | `scripts/lib/semantic_detector.py`。用于判断用户消息是否是"值得捕捉的纠正行为"的分类器，底层通过调用 Claude CLI 实现，是插件的智能核心 |
| **递归 AI 调用** | 用 AI 系统（Claude CLI）来分析或分类另一个 AI 系统（Claude Code 会话）的行为。claude-reflect 中体现为 `subprocess.run(["claude", "-p", user_message])` 在 Claude Code 内部被执行 |
| **manifest hooks field** | `plugin.json` 中声明 hooks 配置文件路径的字段。claude-reflect 的 `CLAUDE.md` 声称 manifest 指向 hooks，但实际 `plugin.json` 中此字段缺失，属于跨组件不一致 bug |
| **隐性知识（tacit knowledge）** | 难以显式编码、存在于实践中的知识。会话中用户纠正 Claude 的行为就是典型隐性知识——不写下来就消失，而 `/reflect` 的价值正是将其显式化 |
| **内省（introspection）** | AI 系统观察并分析自身行为的能力。claude-reflect 通过语义检测器和学习队列实现了一种轻量级 AI 自省：Claude 的会话行为被记录、分类、并作为约束写回 Claude 的上下文 |
| **vague quantifier（模糊限定词）** | 自然语言规范中缺乏精确语义的修饰词，如 "appropriate"、"relevant"、"reasonable"。NLPM 规则体系中记为 R19 类问题。claude-reflect 的 `reflect.md` 因此类问题扣分 6 分（94/100） |
| **四层记忆层次（memory hierarchy）** | claude-reflect 定义的记忆模型：即时上下文 → 会话学习队列 → 项目 CLAUDE.md → 跨项目 SKILL.md。`test_memory_hierarchy.py` 将此模型编码为可断言的测试约束 |
