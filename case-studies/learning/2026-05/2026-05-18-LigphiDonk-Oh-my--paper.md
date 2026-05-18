# LigphiDonk/Oh-my--paper — 学习案例

**仓库**：https://github.com/LigphiDonk/Oh-my--paper
**Stars**：506 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-30（历史快照）| **生成日期**：2026-05-18（基于当前 HEAD）
**主题标签**：security-gate, manifest-discipline, cross-reference, model-pinning, single-purpose

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Oh-my-paper（Oh My Paper）是一个面向科研工作者的 Claude Code 插件，506 颗星表明它触达了一个真实但细分的用户群体：需要在 Claude Code 中完成论文撰写、实验编排和文献追踪的研究者。仓库架构较为复杂：既有 Claude Code 插件（`plugins/oh-my-paper/`），又有哈内斯模板（`templates/harness/`），还内嵌了一个完整的 Tauri 桌面应用（`src-tauri/`）。技能库的上游来自 `dr-claw` 项目，包含 35+ 个经过充分元数据标注的研究技能。

### 1.2 架构剖析

```
Oh-my--paper/
├── plugins/oh-my-paper/        # Claude Code 插件主体
│   ├── agents/                 # 5 个 agent（conductor, reviewer, writer等）
│   ├── commands/               # 9 个 command（plan, survey, experiment等）
│   └── hooks/
│       └── hooks.json          # 3 个 hook（SessionStart, Stop, PostToolUse:Write）
├── templates/harness/          # 可复用的通用研究哈内斯模板
│   ├── agents/                 # 5 个模板 agent（与插件版本近乎相同）
│   └── commands/               # 7 个模板 command
├── src-tauri/resources/skills/ # 35 个从 dr-claw 上游引入的技能（带完整元数据）
├── skills/                     # 同上 35 个技能的完全镜像（！无差异）
└── scripts/                    # sync-research-skills.mjs 等工具脚本
```

- **文件类型分布**：10 个 agent（5 模板 + 5 插件）/ 16 个 command（7 模板 + 9 插件）/ 71 个 SKILL.md / 3 个 hook
- **编排关系**：conductor agent 是核心调度者，通过 `/codex:rescue` 外部命令分发子任务（这是未声明的外部依赖）；commands 是用户入口，agent 是任务执行层，skills 是知识层
- **跨件契约**：命令层依赖外部 `codex` 插件的 `/codex:rescue` 命令，但没有任何地方声明此依赖；`plugins/oh-my-paper/hooks/hooks.json` 和 `commands/setup.md` 双重注册同一个 SessionStart hook（冲突）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「研究工作流的完整生命周期编排」——从创意生成（/omp:ideate）到实验执行（/omp:experiment）到论文撰写（/omp:write）的端到端流程，体现了对科研工作流的深度领域理解
- **解决什么问题**：科研工作中大量重复性认知劳动（文献检索、格式要求、实验记录）占据研究者时间；插件试图将这些环节自动化
- **Trade-off**：功能完整性 vs 安装可靠性——`--dangerously-skip-permissions`、inline Node.js 设置写入、未发布的 hook 脚本三个安全问题叠加，使得「安装完整研究套件」的路径充满风险
- **认知模型**：作者（来自中国高校研究背景，SKILL.md 中有中文描述）将 Claude Code 视为可以编排完整研究流程的 AI 协调器，而非简单的代码助手；这种雄心带来了架构复杂度，也带来了相应的质量债务

---

## 二、过去审查发现（2026-04-30 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-30 当时得分 **65/100**，Security: **REVIEW**（3 个 HIGH + 1 个 MEDIUM）。

| 文件类别 | 当时平均分 | 主要问题 |
|---|---|---|
| Agents（10 个） | 30/100 | 全部无 frontmatter |
| Commands（16 个） | 60/100 | 全部无 `name:` 字段 |
| Skills（src-tauri，35 个） | 77/100 | 部分缺 description，个人化内容 |
| CLAUDE.md（模板） | 65/100 | 角色定位模糊 |

### 2.2 当时值得借鉴的模式

1. **富元数据 skill 设计（dr-claw 上游风格）** → `inno-idea-generation/SKILL.md` 的 frontmatter 包含 `id`、`version`、`stages`、`primaryIntent`、`intents`、`capabilities`、`domains`、`keywords`、`upstream.repo/path/revision` 等字段。这是远超 NLPM 最低要求的元数据设计，为 skill 路由、版本追踪和来源审计提供完整信息 → 借鉴：为我的高复杂度 skill 增加 `version`、`stages` 和 `upstream` 字段
2. **资源标志（resourceFlags）** → frontmatter 中 `resourceFlags.hasScripts`、`hasTemplates`、`scriptCount`、`templateCount` 等字段，让使用者在加载 skill 前就知道它有哪些支撑资源 → 借鉴：为带有外部资源的 skill 添加资源清单
3. **AskUserQuestion 门控** → 多个 command（idea-forge、ideate）在关键决策点使用 `AskUserQuestion`，确保用户在生成前确认方向，而非 agent 自行假设 → 长流程 command 应该设置人工检查点
4. **明确的模板/脚本分离** → skills 下的 `templates/` 和 `scripts/` 子目录区分了「输出物模板」和「辅助脚本」，提高了 skill 目录的可读性
5. **多格式学术标准覆盖** → `ml-paper-writing/SKILL.md` 同时覆盖 NeurIPS/ICML/ICLR 格式要求，配合 50 个模板片段。这种「一次 skill 覆盖多场景」的设计减少了用户在不同会议间切换 skill 的摩擦

### 2.3 当时的缺陷

1. **10 个 agent 全无 frontmatter**：`name`、`description`、`model`、`examples` 全部缺失，agent 无法被 Claude Code 注册。根本原因：作者可能将 agent 文件视为内容文档而非注册配置，没有意识到 frontmatter 是 Claude Code 发现机制的先决条件。最严重的是 conductor agent——它是整个系统的调度核心，但它本身无法被 Claude Code 识别。**自查：我的 agent 文件是否全部有 `name:` 和 `description:` 字段？**
2. **`--dangerously-skip-permissions` 写入 skill 默认值**：`claude-code-dispatch/SKILL.md` 将 `--dangerously-skip-permissions` 作为标准调用方式写入 skill 正文。根本原因：作者为了让 conductor agent 能够不受阻碍地执行子任务而采用了最激进的权限绕过方式，但把这个标志硬写进 skill 意味着所有读取该 skill 的 agent 都会用最高权限模式运行。**自查：我的 skill 中有没有写入任何 `--dangerously-skip-permissions` 或等效权限绕过指令？**
3. **35 个 skill 的双目录镜像无差异**：`src-tauri/resources/skills/` 和 `skills/` 完全相同，是冗余维护负担的来源。根本原因：src-tauri 桌面应用构建需要在 resources/ 下有 skill 文件，而 Claude Code 插件从 skills/ 读取，作者选择了简单复制而非符号链接或构建步骤。**自查：我的项目中有没有同步维护的重复文件？**

### 2.4 当时的优化机会

1. 为所有 10 个 agent 添加最小化 frontmatter（`name`、`description`、`model`），尤其是 conductor 和 reviewer（对系统功能影响最大）
2. 将 `--dangerously-skip-permissions` 从 skill 正文中移除，替换为注释说明「如需全自动模式，用户需手动添加此标志」
3. 在 `plugin.json` 中添加 `codex` 插件依赖声明，或在 `setup.md` 中加入前提检查：「如果 /codex:rescue 不可用，提示用户先安装 codex 插件」

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 10 个 agent 全无 frontmatter | `grep -rL "^name:" plugins/oh-my-paper/agents/ templates/harness/agents/` | **仍缺失**：10/10 agent 仍无 `name:` 字段 | 最高优先级 bug 持续 3+ 周未修复；插件在技术上仍不可安装使用 |
| 16 个 command 无 `name:` | `grep -rL "^name:" plugins/oh-my-paper/commands/ templates/harness/commands/` | **仍缺失**：16/16 command 仍无 `name:` | 同上，说明作者可能没有看到或没有认同 audit 结论 |
| `--dangerously-skip-permissions` | `grep -rn "dangerously-skip-permissions" skills/ src-tauri/` | **仍存在**：两个目录下都有（skills/claude-code-dispatch/SKILL.md 和 src-tauri 镜像），共 4 处出现 | 安全问题未修复；双目录镜像意味着即使修复一处也需要同步修复另一处——这正是 CC-01 双目录问题的恶果 |

### 3.2 架构演进

当前 HEAD 相比 audit 时的主要变化：`skills/` 根目录中新增了 `paper-finder/` 子目录（audit 时未出现），表明 skill 库仍在扩充。src-tauri 桌面应用部分有 `src/editor/lezer-latex/` 新增（LaTeX 编辑器集成），表明 Tauri 桌面端在向更完整的学术编辑器演进。audit 中关注的核心 Claude Code 插件部分（agents/commands/hooks）无架构变化，3 个关键 bug 均未处理。

### 3.3 新增的可学习模式

`skills/paper-finder/` 是 audit 后出现的新 skill，专门用于从 arXiv/semantic scholar 等来源搜索相关论文。从 `resourceFlags.hasScripts: true` 可以看出它同样延续了 dr-claw 风格的富元数据 frontmatter——说明新增 skill 遵循了同一元数据规范，具有架构一致性。这是一个积极信号：即使核心 bug 未修复，新增内容的质量标准有所提升。

---

## 四、校准

### 4.1 我已经在做对的
- Agent frontmatter 完整性：我的 agent 文件全部有 `name`、`description`、`model`；这个案例展示了缺失它们的代价（整个系统不可用）
- 不在 skill 正文中写入权限绕过指令：`--dangerously-skip-permissions` 这类标志不应出现在 skill 默认值中
- 依赖声明：外部插件依赖应在 plugin.json 或 setup command 中明确声明
- 避免重复目录：保持单一事实来源，用符号链接或构建步骤处理多路径需求

### 4.2 挑战 / 验证

这次 audit 挑战了「功能完整性 ≠ 可安装性」的直觉。Oh-my-paper 拥有 35 个高质量 skill、5 个 agent、16 个 command 和完整的研究工作流，但 10 个 agent 无 frontmatter 意味着安装后什么都运行不了。这揭示了一个反直觉的规律：**NL artifact 的注册机制是系统的最脆弱点**——不管内容多好，frontmatter 缺失就是 0 分的功能。一个全面但无法注册的系统比一个简单但可运行的系统更让用户沮丧。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查当前项目中所有 agent 是否有完整 frontmatter（name + description + model）
find .claude/agents -name "*.md" 2>/dev/null | while read f; do
  missing=""
  grep -q "^name:" "$f" || missing="${missing}name "
  grep -q "^description:" "$f" || missing="${missing}description "
  grep -q "^model:" "$f" || missing="${missing}model "
  [ -n "$missing" ] && echo "MISSING [${missing}] in $f"
done
# 命中后：补充对应 frontmatter 字段
```

```bash
# 检查 skill 正文中是否有权限绕过指令
grep -rn -E "dangerously-skip-permissions|approval-mode full-auto|--no-verify" \
  ~/.claude/skills/ .claude/skills/ 2>/dev/null
# 命中后：移除或改为注释（说明用户需手动添加的可选标志）
```

```bash
# 检测重复的 skill/agent 目录
find . -name "SKILL.md" 2>/dev/null | xargs -I{} dirname {} | \
  xargs -I{} basename {} | sort | uniq -d
# 命中后：确认是否确实有重复目录，考虑符号链接替代
```

### 5.2 灵感 → 实施路径

1. **想法**：借鉴 dr-claw 的富元数据 frontmatter，为我最复杂的 skill 添加 `version`、`stages` 和 `upstream` 字段
   - **为何可行**：这些字段对 skill 的可发现性和版本管理有实质帮助，而且是纯增量改动，不破坏现有功能
   - **第一步**：选择我最常用的 skill，在 frontmatter 中添加：`version: 1.0.0`、`stages: [analysis]`（或实际阶段名）、`upstream: {repo: 自己的 repo, path: 相对路径}`；5 分钟可完成

2. **想法**：在长流程 command 中增加 AskUserQuestion 决策门控点
   - **为何可行**：Oh-my-paper 的 ideate、survey 等命令得分高于其他命令，部分原因是设置了用户参与的检查点；这减少了 agent 在关键路口「自行决定」的频率
   - **第一步**：找出我最复杂的一个 command，在它的「规划完成 / 执行前」这个时间点插入 `AskUserQuestion`：「我准备执行以下操作：[列出计划步骤]，是否继续？」；20 分钟可完成

3. **想法**：为我的插件添加外部依赖声明
   - **为何可行**：Oh-my-paper 的 `/codex:rescue` 依赖是一个隐患；用户安装后调用命令，却发现依赖缺失，体验极差
   - **第一步**：在 `plugin.json` 中添加 `"peerDependencies": {"other-plugin": "^1.0.0"}` 字段；在 setup command 中添加「检查所有依赖是否已安装」的第一步；30 分钟可完成
