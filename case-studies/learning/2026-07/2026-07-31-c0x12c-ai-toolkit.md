# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-07-31（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `template-design`, `single-purpose`, `cross-reference`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
c0x12c/ai-toolkit 是「Spartan / GSD（Get Stuff Done）」工作流框架——面向全栈开发团队的 Claude Code 插件，覆盖后端 API 设计、基础设施、设计、研究、创业运营等场景。版本 1.27.0（2026-06-18），已从最初审计的 69 个工件（10 agents + 25 commands + 34 skills）增长到 80+ 工件，尤其是 commands 从 25 增至 61。这是本批审计中工件数量增长最快的仓库。

关键事实：
- 由越南软件公司 c0x12c 维护，面向其内部全栈团队（Kotlin/Terraform/Spartan 工作流）
- 核心架构：`toolkit/` 是原始来源，`.codex/` 是自动生成的镜像（由 `compile-packs.js` 在 `prepublishOnly` 时同步）
- 61 Stars，v1.27.0，活跃开发中（CHANGELOG 显示 2026 年内有多次大版本）
- 2026-04-25 审计时发现 2 个 bug，当前均已修复（有趣的是修复方式与预期不同）

### 1.2 架构剖析

```
c0x12c/ai-toolkit/
├── toolkit/                     # 原始来源（单一真实来源）
│   ├── agents/                  # 9 个 agents（audit 时 10 个，team-coordinator 已删）
│   ├── commands/spartan/        # 61 个命令（audit 时 25 个，大幅扩展）
│   ├── skills/                  # 34 个 skills（未变）
│   └── rules/                   # 设计/架构/提交规则文档（未注册为 skills）
├── .codex/                      # 自动生成镜像，供 Codex CLI 使用
│   ├── commands/spartan/        # 镜像 toolkit/commands
│   ├── agents/                  # 部分镜像（5 of 9，镜像缺口问题）
│   └── skills/                  # 部分镜像（26 of 34，8 个 skills 只在 toolkit/）
├── bridges/                     # Telegram 等 AI bridge（包含 bypassPermissions 配置）
├── CLAUDE.md                    # 项目上下文（由多个 claude-md/*.md 片段组成）
├── AGENTS.md                    # Agent 一览表
└── CHANGELOG.md                 # 语义化版本记录
```

- **文件类型分布**：9 个 agent + 61 个 command + 34 个 skill
- **编排关系**：`toolkit/` → `compile-packs.js` → `.codex/`（自动镜像）；commands 调用 agents 和 skills
- **跨件契约**：`.codex/` 的镜像完整性由 `prepublishOnly` 脚本保证，但镜像缺口（8 skills + 5 agents 只在 toolkit/）是持续的 cross-component 问题

### 1.3 设计思路 / 方法论

- **核心哲学**：「GSD (Get Shit Done)」——所有工件命名为 `spartan:*`，强调落地执行而非理论探讨
- **双运行时设计**：同一个 toolkit 同时服务 Claude Code（via `toolkit/`）和 Codex CLI（via `.codex/`），用自动镜像而非人工同步
- **Trade-off**：双运行时提高了覆盖面，但引入了镜像完整性风险——8 个 skills 在 `.codex/` 缺席，意味着 Codex 用户调用这些 skills 会静默失败
- **进化策略**：v1.24.0 彻底删除 GSD pack（包括 `get-shit-done-cc@latest` npm 依赖）——解决了安全审计发现的 M-3 问题（unpinned latest npm execution）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单一来源 + 自动多目标镜像」**模式：`toolkit/` 是真相的唯一来源，`compile-packs.js` 在发布时自动生成 `.codex/` 镜像，确保两个运行时同步。

模式特征：
- 有一个「权威目录」（此处 `toolkit/`），所有人类编辑只在这里进行
- 有一个「镜像目录」（此处 `.codex/`），自动生成，不人工编辑
- 发布流程集成了镜像同步步骤
- 镜像可以是部分镜像（此处 `.codex/` 只包含部分 agents/skills）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 支持多个 AI 运行时的工具集 | ✅ 高度适用 | 避免了 N 份人工维护的拷贝 |
| 单 AI 运行时插件 | ❌ 不适用 | 镜像机制增加发布复杂度 |
| 需要细粒度版本控制每个 skill | ❌ 不适用 | 镜像自动同步与独立版本化冲突 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单来源多目标镜像（本案） | c0x12c/ai-toolkit | 减少重复，自动同步 | 镜像缺口风险；发布流程更复杂 |
| 单仓单运行时 | addyosmani-agent-skills | 简单直接 | 无法服务多运行时 |
| 多仓各自维护 | 多数团队 | 独立演进 | 容易漂移，难以共享基础 rules |

### 2.4 改进空间

1. **当前问题**：`.codex/agents/` 只有 5 个，`toolkit/agents/` 有 9 个，4 个 agents 缺席。**改进做法**：在 `compile-packs.js` 中加入断言：镜像后的 agent 数量必须等于源目录数量，否则 build 失败。**预期收益**：未来新增 agent 时，不再可能因为疏漏而导致 Codex 用户看不到。

2. **当前问题**：`toolkit/rules/` 是高质量规则文档（DESIGN_PROCESS.md、GIT_COMMIT.md 等），但未注册为 skills，无法被 `/use-skill` 调用。**改进做法**：为每个规则文档创建一个轻量 SKILL.md wrapper（只有 frontmatter 和 `@rules/<file>` 引用）。**预期收益**：NLPM 可以发现并评分，用户可以通过 skill 调用。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-25 当时得分 **96/100**，SECURITY **UNKNOWN**（3 Medium + 3 Low）。

| 类别 | 当时平均 | 主要问题 |
|---|---|---|
| Agents（10个）| 94.3 | 3 agents 零示例（-15×3）；1 bug（phase-reviewer 缺 Bash）|
| Commands（25个）| 94.8 | 全部 25 个命令缺 `allowed-tools` 字段（-5×25）|
| Skills（34个）| 98.6 | 5 个 skills 缺 Gotchas 小节（-3×5）；1 bug（terraform-best-practices 缺 `allowed_tools`）|

### 3.2 当时值得借鉴的模式

1. **Skills 平均 98.6 分** → 原因：大多数 skills 有完整的示例、输出格式、Gotchas 小节。`terraform-review`、`deep-research`、`testing-strategies` 等全部 100 分，是教科书级示例。

2. **Mirror 架构** → `toolkit/` → `.codex/` 自动同步的设计思路值得借鉴，尤其是「单一真实来源」原则。

3. **多 agent 协作有明确分工** → 9 个 agents 各有清晰的角色边界（phase-reviewer 专注代码评审、idea-killer 专注想法质疑），没有功能重叠。

4. **CHANGELOG 语义化** → 每个版本都有清晰的 Added/Changed/Fixed/Removed 分类，便于快速了解演进历史。

### 3.3 当时的缺陷

1. **phase-reviewer.md 引用 cat/ls 但 tools 里无 Bash** → 为什么会失败：声明了 `tools: [Read, Grep, Glob, WebSearch]` 但指令里要 Claude 执行 `cat`/`ls`——Bash 未授权，这些 shell 命令会在运行时被拒绝。**自查**：我的 agents 有没有在指令中用到未声明工具的命令？

2. **25 个命令全部缺 `allowed-tools`** → 为什么会失败：command [frontmatter](#frontmatter) 没有 `allowed-tools` 字段，Claude Code 无法在执行前校验工具授权。这是一个系统性问题，说明作者不知道命令也需要声明 allowed-tools（或知道但认为不重要）。

3. **terraform-best-practices/SKILL.md 缺 `allowed_tools`** → 孤立的 convention 违反，暗示该文件是后来添加的，没有经过同等审查。

### 3.4 当时的优化机会

1. 给 research-planner、idea-killer、ai-designer 添加示例 block
2. 给全部 25 个命令批量添加 `allowed-tools`
3. 给 terraform-best-practices/SKILL.md 添加 `allowed_tools`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| phase-reviewer 缺 Bash tool | `grep "tools:" toolkit/agents/phase-reviewer.md` | **已修复**：`tools:` 字段被移除，使用默认工具集（含 Bash）；同时新增了 `model: sonnet` 和 2 个 `<example>` | 修复方式是移除限制而非显式授权 |
| terraform-best-practices 缺 `allowed_tools` | `grep "allowed_tools" toolkit/skills/terraform-best-practices/SKILL.md` | **持续**：无 `allowed_tools` 字段 | 孤立缺陷持续 |
| 25 个命令缺 `allowed-tools` | `grep "allowed-tools" .codex/commands/spartan/ask.md` | **持续** + **扩大**：命令数从 25 增至 61，全部缺字段 | 系统性缺口随规模扩大 |
| M-3: npx get-shit-done-cc@latest | `grep "get-shit-done" toolkit/scripts/setup.sh` | **已修复**：v1.24.0 删除了整个 GSD pack | 删除依赖比固定版本更彻底 |

**关键观察**：phase-reviewer 的修复方式很有启发性——作者选择「移除限制」（删除 `tools:` 字段）而非「显式授权 Bash」。这是一个务实但略显随意的修复：工具默认集包含 Bash，所以功能恢复了，但失去了明确声明的安全边界。

### 4.2 架构演进

v1.24.0（2026-05-02）是重大架构转折：**完全删除 GSD pack**（`project-mgmt` pack、team-coordinator agent、全部 GSD 命令）。这意味着 ai-toolkit 从「Spartan + GSD 双引擎」演进为「纯 Spartan 框架」。同时 commands 从 25 增至 61，新增了大量 `/spartan:*` 命令（如 `codex`、`commit-message-with-codex`、`ship-pr-codex`），引入 Codex CLI 集成。

这说明作者的优先级明确：**向前扩张（更多命令）比向后修补（安全/quality issues）更优先**。61 个命令全部缺 `allowed-tools` 正是这种取向的副产品。

### 4.3 新增的可学习模式

**Codex CLI 双运行时扩展**（v1.25.0-v1.27.0）：新增了一套 `cdx-*` shell helpers（`cdx-review`, `cdx-pr`, `cdx-ship`），让 Codex CLI 作为 Claude Code 的「第二视角审查员」。`spartan:ship-pr-codex` 实现多轮「请求 Copilot review → 等待 → 修复 → reply → 继续」循环。这是一个「多 AI 协作 + 自动修复循环」的值得关注的新模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **skills 的高质量标准**：参考此案 skills 98.6 分的实现，我的 skills 也应保持示例 + 输出格式 + Gotchas 三件套
2. **CHANGELOG 语义化**：按照 Added/Changed/Fixed/Removed 分类记录变更
3. **命令专注单一职责**：spartan 命令各自命名清晰，功能不重叠

### 5.2 挑战 / 验证

**挑战了我的假设**：我之前认为「提交 PR 后维护者会优先修复 bug」。但 c0x12c 的行为模式是「删除整个依赖（v1.24.0 删 GSD pack）解决安全问题，但不修复单个文件的 allowed-tools 缺失」——说明开发者更倾向于通过「架构调整」解决安全问题，而不是「逐文件补字段」。这是务实决策，不一定是对的，但是真实的。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 allowed-tools
for f in $(find . -path "*/.claude/commands/*.md" 2>/dev/null); do
  has_allowed=$(grep -c "allowed-tools\|allowed_tools" "$f" 2>/dev/null)
  if [ "$has_allowed" -eq 0 ]; then
    echo "MISSING allowed-tools: $f"
  fi
done
```
命中后怎么办：根据命令体中实际使用的工具，添加 `allowed-tools: [Tool1, Tool2, ...]` 到 frontmatter。

```bash
# 检查 agents 的 tools: 字段（显式声明 vs 默认）
for f in $(find . -path "*/.claude/agents/*.md" 2>/dev/null); do
  has_tools=$(grep -c "^tools:" "$f" 2>/dev/null)
  echo "$(basename $f): tools_declared=$has_tools"
done
```
命中后怎么办：如果 agent 使用了特定工具（尤其是 Bash），显式声明比依赖默认更安全。

### 6.2 灵感 → 实施路径

1. **想法**：为 commands 批量添加 `allowed-tools` 用脚本自动生成初稿
   - **为何可行**：61 个命令手动一个个加效率太低，但可以写脚本扫描 command body 中出现的工具名，自动生成 `allowed-tools:` 候选列表
   - **第一步**：写 30 行 Python，扫描 `.claude/commands/*.md` 中的工具关键词（Read/Write/Edit/Bash/Grep/Glob），生成 `allowed-tools` 初稿附在文件末尾，人工确认后插入 frontmatter（2-3 小时）

2. **想法**：为 `toolkit/rules/` 目录下的规则文档创建 SKILL.md wrapper
   - **为何可行**：规则文档质量高，做成 skill 可以被 Claude 按需加载，不需要 CLAUDE.md 全量注入
   - **第一步**：找出仓库中有「我不希望 Claude 每次都读，但偶尔会用到」的文档，为每个写一个 5 行 SKILL.md wrapper（10-15 分钟/文件）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：全栈开发团队的 Claude Code 工作流框架（Spartan GSD 方法论）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 均为 Claude Code 工作流工具集，均有多个 skills + 命令 | gstack 是 Garry Tan 的个人工具集，ai-toolkit 是公司工作流框架 | 高 |
| MarkQWu/graphify | 低 | 均为开发者工具 | graphify 专注知识图谱，不是工作流框架 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 缺 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-gstack/openclaw/skills/*/SKILL.md` | gstack 的 openclaw skills 4/6 缺少 `allowed_tools` | 中 |
| Skills 缺 Gotchas 小节 | `grep -L "Gotcha\|gotcha\|Common\|Avoid" /tmp/my-repos/MarkQWu-gstack/openclaw/skills/*/SKILL.md` | 需人工验证 | 低-中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack/openclaw/skills/gstack-openclaw-ceo-review/SKILL.md` → 加 `allowed_tools: [Read, Glob, Grep]`
- 同样适用于 `gstack-openclaw-investigate`、`gstack-openclaw-office-hours`、`gstack-openclaw-retro`

### 8.3 别人的更优方案

1. **领域**：Skills 的 Gotchas 小节
   - **本案做法**：`terraform-review/SKILL.md`、`testing-strategies/SKILL.md` 等均有 `## Gotchas` 小节，记录反模式和常见错误
   - **我的项目现状**：gstack/graphify 的 SKILL.md 有「Rules」小节但缺专门的 Gotchas 列举
   - **如何借鉴**：在我的 SKILL.md 中加一个 `## Gotchas` 小节，列出 2-3 条「不要这样用」的反例

2. **领域**：CHANGELOG 详细到精确的 API 行为变化
   - **本案做法**：v1.25.0 CHANGELOG 描述了「`created_at >= REQUESTED_AT` + `commit_id == HEAD_SHA` 双重保证」的具体实现逻辑
   - **我的项目现状**：CHANGELOG 条目偏简短
   - **如何借鉴**：对涉及 race condition 或边界条件的修改，在 CHANGELOG 中描述「旧行为 → 新行为」的精确差异

### 8.4 反向：我的项目做得比他们好的地方

MarkQWu/gstack 的 `pair-agent/SKILL.md` 和 `skillify/SKILL.md` 都有 `allowed_tools` 字段。相比 ai-toolkit 中 61 个命令全部缺 `allowed-tools`，我在 gstack 的核心 skills 上维护了这个字段，这是规范性上的优势。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。command 的 frontmatter 包含 `description`、`allowed-tools` 等字段；skill 的 frontmatter 包含 `name`、`description`、`allowed_tools`、`model` 等。没有 `allowed-tools`，Claude Code 无法在执行命令前校验工具授权。

### <a name="GSD"></a>GSD（Get Shit Done）
> 一个强调「执行力优先于完美主义」的工程方法论，由 `get-shit-done-cc` npm 包实现。ai-toolkit 的 v1.24.0 删除了对这个包的依赖，把 GSD 思想内化到 Spartan 命令体系中，而不再依赖外部 npm 包。

### <a name="bypassPermissions"></a>bypassPermissions
> Claude Code 的一个配置项，当设为 `true` 时，所有工具调用跳过用户确认提示。在 ai-toolkit 的 `bridges/core/engine.js` 中，这是默认值（`permInteractive=false` 时）——用户采用 bridge 时会在不知情的情况下获得最大权限级别的 agent。
