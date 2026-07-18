# agent-sh/enhance — 学习案例

**仓库**：https://github.com/agent-sh/enhance
**Stars**：3 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-07-18（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `single-purpose`, `nl-binary-hybrid`, `template-design`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
enhance 是 agent-sh 团队的「插件结构与工具使用分析器」——一个 Claude Code 插件，通过 8 个专门的增强代理（enhancer agents）并行分析现有 NL 工件（SKILL.md、agents、hooks、文档、CLAUDE.md、prompts），给出改进建议并可选自动应用。与 agnix（验证合规性）不同，enhance 侧重「质量提升」而非「合规检查」。后端依赖 `agent-sh/agent-analyzer` 原生二进制，通过运行时下载分发，3 stars，仍处于早期用户阶段。

### 1.2 架构剖析
- **目录结构**：
  ```
  agent-sh/enhance/
  ├── .claude-plugin/plugin.json   # 插件 manifest（版本 5.1.0）
  ├── commands/enhance.md          # 主入口命令
  ├── agents/                      # 8 个专门化增强代理
  │   ├── agent-enhancer.md
  │   ├── claudemd-enhancer.md
  │   ├── cross-file-enhancer.md
  │   ├── docs-enhancer.md
  │   ├── hooks-enhancer.md
  │   ├── plugin-enhancer.md
  │   ├── prompt-enhancer.md
  │   └── skills-enhancer.md
  ├── skills/                      # 8 个匹配技能（每个代理一个）
  │   ├── enhance-agent-prompts/SKILL.md
  │   ├── enhance-claude-memory/SKILL.md
  │   ├── enhance-cross-file/SKILL.md
  │   ├── enhance-docs/SKILL.md
  │   ├── enhance-hooks/SKILL.md
  │   ├── enhance-orchestrator/SKILL.md
  │   ├── enhance-plugins/SKILL.md
  │   ├── enhance-prompts/SKILL.md
  │   └── enhance-skills/SKILL.md
  ├── lib/                         # Node.js 封装层
  │   ├── binary/index.js          # 运行时二进制下载 + 执行
  │   ├── collectors/              # 工件收集器
  │   ├── enhance/                 # 增强逻辑
  │   └── ...
  ├── agent-knowledge/             # 学习数据持久化
  ├── scripts/setup-hooks.sh
  └── package.json                 # npm 包（版本 1.0.0）
  ```
- **文件类型分布**：1 个 command / 8 个 agent / 9 个 SKILL.md / 1 个 plugin.json / ~92 个 Node.js lib 文件
- **编排关系**：`commands/enhance.md` 是 router，解析用户参数（`--focus=TYPE`、`--apply`）后并行分发到对应的 agent；每个 agent 加载对应 skill 作为知识来源；agents 通过 Skill 工具调用 skill 内容
- **跨件契约**：所有 8 个 agent 声明同样的工具集（Skill、Read、Glob、Grep、Bash(git:*)）；enhance-orchestrator SKILL.md 协调整体流程；`agent-knowledge/` 存储跨会话学习数据（--no-learn 可禁用）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「专门化代理并行分析」——不用一个全能代理处理所有增强任务，而是针对每种工件类型设一个专业代理（hooks-enhancer 只懂 hooks，docs-enhancer 只懂文档）
- **解决什么问题**：Claude Code 用户的 NL 工件存在已知但难以系统化发现的质量问题（vague prompts、missing examples、broken references），enhance 把这个过程自动化
- **做了什么 trade-off**：选择「8 个专门代理」而非「1 个通用代理 + 丰富 skill」——分工明确但意味着 8 倍的维护表面；选择运行时下载二进制而非包内打包（减小包体积但引入供应链风险）
- **反映什么认知模型**：作者相信「每种工件类型有独特的质量维度，专家比通才准确」——这是一个微服务思维在 NL 编程层的投影

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「1 命令 + N 专门代理 + N 技能」（Command-Router + Specialist-Agent + Skill-Knowledge）**：单一命令入口做路由，按工件类型分派到专门代理，每个代理携带对应技能知识。

模式特征清单：
- 命令层极薄：只做参数解析和路由分发
- 每个代理有明确的单一职责（只处理一类工件）
- 代理-技能 1:1 配对（每个代理有自己的知识技能）
- 并行执行是默认行为（`--focus` 才是顺序执行单个代理）
- 学习数据持久化到 `agent-knowledge/`（跨会话积累）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多种工件类型需要并行分析 | ✅ 高度适用 | 专门代理可独立迭代，质量互不干扰 |
| 需要「自动修复」功能（--apply）| ✅ 适用（有条件）| 每个代理了解具体工件结构，可精准写回 |
| 只有 1-2 种工件类型 | ❌ 不适用 | 8 个代理的维护成本超过收益，单代理 + 单 skill 够用 |
| 完全不需要学习/记忆能力 | ❌ 不适用 | agent-knowledge/ 的价值在积累之后，新项目收益低 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 1 命令 + N 专门代理（本仓库）| agent-sh/enhance | 关注点分离，易独立迭代 | 8× 维护表面，代理间一致性容易漂移 |
| 1 命令 + 1 代理 + 多 skill | 大多数小型插件 | 简单，维护成本低 | 代理职责过重，质量上限受 context 大小限制 |
| 纯命令驱动（无代理层）| aaronmaturen/claude-plugin | 极简，无编排复杂度 | 无法并行，无法积累跨会话学习 |

### 2.4 改进空间
1. **当前问题**：8 个 agent 都声明 `--fix` / apply 行为，但工具列表里无 Write/Edit **改进做法**：在每个 agent frontmatter 的 tools 里加 `Write`、`Edit`，或在 skill 体里明确说明「通过 Skill 工具的 apply 路径写文件」的机制 **预期收益**：自动修复功能语义清晰，不再依赖运行时猜测
2. **当前问题**：plugin.json 5.1.0 vs package.json 1.0.0（版本漂移）**改进做法**：参考 agnix 的 sync-versions.sh，在发布流程里同步两个文件 **预期收益**：消除版本混乱，enhance-plugins skill 对这个 bug 的自动检测能真正报告并修复自身

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-27 当时得分 **86/100**（均值）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/claudemd-enhancer.md | 73 | 无 examples(-15)、无 output format(-10)、vague description(-2) |
| agents/cross-file-enhancer.md | 73 | 无 examples(-15)、无 output format(-10)、"complex"模糊(-2) |
| agents/\*.md (其余 6 个) | 73-75 | 无 examples(-15)、无 output format(-10) |
| CLAUDE.md | 90 | 8 条关键规则无 WHY 解释 |
| skills/enhance-hooks/SKILL.md | 92 | 4 个模糊词（complex、simple、unreasonably、cautious）|
| commands/enhance.md | 93 | 缺 allowed-tools(-5)、"relevant"模糊(-2) |
| .claude-plugin/plugin.json | 95 | 版本 5.1.0 与 package.json 1.0.0 不同步 |

### 3.2 当时值得借鉴的模式
1. **专门代理 1:1 配对技能** → 每个 enhancer agent 只加载与自己职责匹配的 skill → `agents/hooks-enhancer.md` + `skills/enhance-hooks/SKILL.md` → 借鉴：避免单一巨型 skill 服务所有代理
2. **argument-hint 完整声明** → commands/enhance.md 的 argument-hint 列出所有可用标志（`--apply`、`--focus=TYPE`、`--verbose` 等），用户调用时有提示 → 借鉴：命令参数要在 frontmatter 里声明完整用法
3. **agent-knowledge/ 跨会话学习** → 通过持久化文件记录历史增强模式，后续调用可参考 → 借鉴：[经验侧车文件](#experience-sidecar) 模式的具体实现

### 3.3 当时的缺陷
1. **所有 8 个 agent 无 examples block** → 为什么失败：agent 没有 examples，模型不知道「增强后的输出应该是什么样的」，会产生格式漂移。自查：**我的 gstack 某些 skill 也缺 examples**，需要检查
2. **allowed-tools 在 commands/enhance.md 里缺失** → 为什么失败：Claude Code 在分派命令前需要知道该命令会用哪些工具做权限预授权；缺失意味着权限模型断裂，Claude 需要运行时猜测工具权限。自查：**值得检查我的命令文件**
3. **plugin.json 与 package.json 版本不同步（5.1.0 vs 1.0.0）** → 为什么失败：两个文件由不同开发者/流程维护，没有单一来源约束。enhance-plugins skill 自动检测到了这个 bug 并报告，但没有自动修复，说明 enhance 的 --apply 对自身版本文件不生效。自查：**bureau 同样有此问题**

### 3.4 当时的优化机会
1. 为 8 个 agent 统一添加 examples block（至少 2 个 input→output 对）
2. 在 commands/enhance.md frontmatter 里加 allowed-tools
3. 同步 plugin.json 和 package.json 的版本号

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 8 个 agent 无 examples block | `grep -L "examples\|## Example" agents/*.md` | **仍存在**：无任何 agent 有 examples block | 作者优先级排序中，examples 不是紧迫项 |
| commands/enhance.md 缺 allowed-tools | `head -12 commands/enhance.md` | **仍存在**：frontmatter 仍无 allowed-tools 字段 | PR 可能被拒绝或作者认为不需要 |
| plugin.json 5.1.0 vs package.json 1.0.0 | `cat .claude-plugin/plugin.json \| python3 -c "..."` 和 `cat package.json` | **仍存在**：5.1.0 vs 1.0.0，差距从未缩小 | 发布流程未绑定两者；这个 bug 是 enhance 自身 `enhance-plugins` skill 检测的 HIGH-certainty 问题，但 enhance 没有修复自身——体现「鞋匠没穿鞋」效应 |

### 4.2 架构演进
2026-04-27 Audit（19 个 artifacts）到当前 HEAD 基本一致——lib/ 目录有新增子模块（drift-detect、perf、repo-intel、repo-map 等），说明作者在增强分析能力（检测代码漂移、性能分析、仓库拓扑理解），但 NL 表皮（commands/、agents/、skills/）基本未变。重心在 lib/ 而非 NL 层质量改善。

### 4.3 新增的可学习模式
- **lib/drift-detect/**：专门的「漂移检测」模块，追踪代码库随时间的结构变化。这是「代码健康监控」模式的具体实现，与 agnix 的静态验证形成互补（一个是时点快照，一个是时序追踪）。

---

## 五、校准

### 5.1 我已经在做对的
1. **命令-代理-技能分层思维**：我的 bureau 也有类似的「命令 → 代理 → skill」三层，区别在 bureau 更专注知识库场景
2. **argument-hint 声明**：我的 gstack SKILL.md 有 triggers 和 argument-hint（虽然名称不同），用户知道如何触发
3. **单职责 skill**：gstack 每个目录是一个专门 skill，而非一个巨型 skill 处理所有场景

### 5.2 挑战 / 验证
- **挑战**：「鞋匠不穿鞋」现象——enhance 的 enhance-plugins skill 能检测到自身的版本漂移 bug，但这个 bug 在 HEAD 仍未被修复。这说明「工具能检测」和「维护者会修复」是两回事。对我的启示：自动检测只是手段，建立修复反馈闭环（如 pre-commit 钩子阻止版本不同步的提交）才是根治方案
- **验证**：「8 个专门代理比 1 个通用代理质量更高」这个假设——在 enhance 案例中，8 个代理的 NL 质量都是 73-75 分，说明专门化并不自动带来 NL 质量提升；质量取决于每个代理的维护投入，而非数量

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否有 examples block
find ~/.claude -name "*.md" \( -path "*/agents/*" -o -name "*.agent.md" \) | while read f; do
  if ! grep -q "## Example\|<examples>" "$f"; then
    echo "缺 examples: $f"
  fi
done
# 命中后：添加至少 2 个 input→output 示例

# 检查版本一致性（bureau 和其他有 plugin.json 的仓库）
python3 -c "
import json, glob
for pf in glob.glob(os.path.expanduser('~/.claude-plugin/plugin.json')) + glob.glob(os.path.expanduser('~/*/plugin.json')):
    d = json.load(open(pf))
    print(f'plugin {pf}: {d.get(\"version\", \"N/A\")}')
"
# 命中（版本不一致）后：写 sync-versions.sh 统一版本
```

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau 添加 pre-commit 钩子检查版本一致性（仿 enhance 的坑但不犯同样错误）
   - **为何可行**：pre-commit 阻止版本不同步进入代码库，比事后修复成本低
   - **第一步**：在 bureau/.git/hooks/pre-commit 里加 2 行：读取 plugin.json 和 package.json 的 version，不同则 echo error 并 exit 1，10 分钟完成

2. **想法**：参考 enhance 的 1:1 代理-技能配对模式，检查我的 gstack 代理（如有）是否有配套专属技能
   - **为何可行**：gstack 有复杂的多 skill 协作，检查是否每个「场景类型」都有对应的专门 skill
   - **第一步**：列出 gstack 所有 skill 目录，检查是否有「通用 skill 承担多种场景」的情况，15 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 agent-sh/enhance 的核心目的**：分析 Claude Code 插件/工件结构，给出质量改进建议（可选自动应用）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是 Claude Code skill 集合，都面向 NL 工件质量 | gstack 是使用侧，enhance 是分析侧 | 中 |
| MarkQWu/agents-in-the-wild | 高 | 都做 NL 工件分析/评分 | agents-in-the-wild 做 audit，enhance 做 enhancement；规模不同 | 高 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 与 package.json 版本不同步 | 对比两文件 version 字段 | **命中**：bureau plugin.json=0.5.2，package.json=0.0.1 | 中 |
| agent 无 examples block | `grep -L "Example\|examples"` agents | gstack 无 agents 目录，bureau 中未找到 agent 文件 | 低 |
| CLAUDE.md 规则无 WHY 解释 | 检查 CLAUDE.md 规则条目 | **可能命中**：gstack CLAUDE.md 的部分规则是裸命令格式，无 WHY | 低 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/.claude-plugin/plugin.json` 和 `MarkQWu-bureau/package.json` → 统一到 0.5.2 → 5 分钟
- `MarkQWu-gstack/CLAUDE.md` → 为关键规则加 1 句 WHY 解释 → 30 分钟

### 8.3 别人的更优方案

1. **领域**：1:1 代理-技能配对的模块化设计
   - **本案例做法**：`agents/hooks-enhancer.md` 独占 `skills/enhance-hooks/SKILL.md`，职责边界清晰
   - **我的项目现状**：gstack 的某些 skill 承担多种场景的引导（如 plan-devex-review），可考虑拆分
   - **如何借鉴**：识别 gstack 中「职责过重」的 skill，将不同场景拆分为独立 skill 文件

2. **领域**：跨会话学习（agent-knowledge/）
   - **本案例做法**：每次运行 enhance 的结果存入 agent-knowledge/，后续运行可参考历史模式
   - **我的项目现状**：gstack 无跨会话学习机制，每次运行都从零开始
   - **如何借鉴**：在 gstack 的 context-save/SKILL.md 模式下，扩展一个「skill 改进记录」持久化文件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：自检闭环——工具能发现并修复自身问题
- **我的做法**（agents-in-the-wild）：NLPM 对自身仓库每次 push 跑 `/nlpm:score` 检查
- **本案例做法**（enhance）：enhance-plugins skill 检测到自身 version drift 但未自动修复，「鞋匠不穿鞋」
- **意义**：自动检测 + 自动修复的闭环比「检测但不修复」更有价值；这是 agents-in-the-wild 的 hooks 机制的核心优势

---

## 八、术语表

### <a name="experience-sidecar"></a>经验侧车文件
> 一种跨会话学习模式：在每次 AI 会话结束时，把有价值的发现（发现的模式、常见错误、改进效果）以结构化形式写入一个独立的 JSON/Markdown 文件（「侧车文件」）。下次会话开始时读取这个文件，从历史经验出发，而不是从零开始。enhance 的 `agent-knowledge/` 目录就是这种模式的实现。

### <a name="write-edit-tool"></a>Write/Edit 工具声明
> Claude Code agent 在 frontmatter 的 `tools:` 字段里必须显式声明它有权使用哪些工具。声明 `Write` 表示代理可以创建/覆盖文件；声明 `Edit` 表示可以做局部编辑。如果代理的行为（如 --apply 自动修复）需要写文件，但 tools 里没有声明 Write/Edit，则实际运行时可能静默失败或触发权限提示。
