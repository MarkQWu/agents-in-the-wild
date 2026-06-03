# gmickel/flow-next — 学习案例

**仓库**：https://github.com/gmickel/flow-next
**Stars**：568 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-06-03（基于当前 HEAD）
**主题标签**：`cross-reference`, `router-channels`, `examples-driven`, `security-gate`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`flow-next` 是一个面向软件开发工作流的多智能体编排插件，核心思路是"用侦察兵（Scout）并行采集上下文，用 Worker 执行代码修改"。关键事实：
- 单仓库（monorepo）同时包含 `plugins/flow-next/`（主流）和独立的 `flow-next-tui/`（终端 UI 子项目）
- **架构演进重要发现**：`plugins/flow/`（v1 版本）在 audit 后已被完全删除，当前 HEAD 只剩 `plugins/flow-next/`
- NL 工件总量 87 个（audit 时），当前版本新增了更多技能（`flow-next-map`、`flow-next-capture`、`flow-next-strategy` 等）
- 独立 CLI 工具 `flowctl`（Python），通过 scripts 管理工作树和 PR
- NLPM 93/100，安全 BLOCKED（Critical：测试脚本中的 eval 命令注入）

### 1.2 架构剖析
```
gmickel/flow-next/
├── plugins/flow-next/
│   ├── commands/flow-next/    # 11 个命令（work, plan, plan-review...）
│   ├── agents/                # 20 个侦察 Agent（repo-scout, context-scout...）
│   ├── skills/                # 20+ 个技能（flow-next-plan, flow-next-work...）
│   ├── codex/skills/          # ← flow-next/skills/ 的 Codex 适配副本
│   ├── scripts/               # flowctl + 测试脚本 + Hook 脚本
│   │   ├── hooks/ralph-guard.py  # 关键 Hook：防止 Ralph 角色扮演越界
│   │   └── ralph_e2e_rp_test.sh  # 含 eval 的 E2E 测试脚本
│   ├── hooks/hooks.json       # PostToolUse Hook 配置
│   └── .claude-plugin/plugin.json
├── flow-next-tui/             # 终端 UI（独立子项目）
└── scripts/                   # 仓库级脚本（sync-codex.sh, bump.sh）
```
- **文件类型分布**：20+ 个 SKILL.md，20 个 agent，11 个 command，1 个 hook 配置，多个 Shell/Python 脚本
- **编排关系**：分层设计——命令文件（用户入口）→ 技能文件（工作流蓝图）→ 侦察 Agent（上下文采集）→ Worker Agent（执行）
- **跨件契约**：`flow-next/skills/` 是权威源，`codex/skills/` 由 `sync-codex.sh` 自动生成，所有跨文件引用（`[phases.md]`, `[workflow.md]` 等）在 audit 时全部验证通过

### 1.3 设计思路 / 方法论
- **核心设计哲学**："侦察兵-执行者"分离（Scout/Worker 模式）——先让多个专业 Agent 并行采集上下文（仓库结构、依赖、测试、安全、文档），再基于这些上下文让 Worker 执行修改
- **解决什么问题**：大型代码库中，Claude 在没有足够上下文时往往做出错误决策；本插件通过结构化的上下文采集确保 Worker 有足够的信息再行动
- **做了什么 trade-off**：每次任务都运行多个侦察 Agent，代价是 token 消耗较高；换来的是决策质量的显著提升和"有迹可查的思考过程"
- **反映什么认知模型**：作者把软件开发工作流视为一个"情报收集 + 计划 + 执行 + 反思"的循环，而不是"输入问题 → 输出代码"的单次交互

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：侦察兵-执行者分层（Scout/Worker）**

用多个专业化的 Agent（每个只负责一个维度的上下文采集）并行工作，把结果汇总后再交给 Worker 执行，而不是让 Worker 在执行时边查边做。

模式特征清单：
- 特征 1：侦察 Agent 只读（不修改文件），Worker 只写（不做大量分析）
- 特征 2：侦察 Agent 按领域分工（repo-scout、security-scout、testing-scout、docs-scout 等），每个高度专业化
- 特征 3：侦察 Agent 输出结构化报告（带 `[VERIFIED]`/`[INFERRED]` 置信度标签），不是自由文本
- 特征 4：`context-scout` 作为元侦察兵，决定哪些专项侦察兵需要被派遣

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型代码库的功能开发 | ✅ 高度适用 | 侦察兵采集的上下文对大仓库影响最大 |
| 单文件的快速修改 | ❌ 过度设计 | 派侦察兵的 token 成本高于直接修改 |
| 需要可审计工作流的团队 | ✅ 适用 | 每一步都有结构化报告，便于 review |
| 学习 Claude Code 多智能体模式 | ✅ 极好的教材 | 代码组织清晰，注释详细，适合参考 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 侦察兵-执行者分层（本仓库） | flow-next | 决策质量高，过程可审计 | token 成本高，延迟长 |
| 单 Agent 全能（无分层） | 普通 command | 简单，延迟低 | 上下文不足时容易出错 |
| Router + 专家 Agent | mikeyobrien/ralph-orchestrator | 动态路由，灵活 | 路由逻辑复杂，调试难 |

### 2.4 改进空间
1. **当前问题**：所有 command 文件缺 `allowed-tools`（audit 时 11 个全缺）。**改进做法**：根据每个 command 实际调用的工具，添加 `allowed-tools: Task, Read, Bash` 等声明。**预期收益**：在严格权限模式下 command 不会因权限阻断而失败。
2. **当前问题**：测试脚本里的 `eval "$(retry_cmd ... "$REVIEW_SUMMARY"...)"` 存在命令注入风险，且 2 个月后仍未修复。**改进做法**：用 `W=$(flowctl rp setup-review ...)` 直接赋值替代 eval，消除 shell 解释。

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-16 当时得分 **93/100**，安全扫描结论 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `plugins/flow-next/agents/epic-scout.md` | 90 | 规则说"用 haiku"但 frontmatter 声明 `model: claude-sonnet-4-6`，矛盾 |
| `plugins/flow/agents/practice-scout.md` | 95 | "Current year is 2025"（已过时） |
| `plugins/flow-next/commands/*/` (11 个) | 95 | 全部缺 `allowed-tools` |
| `plugins/flow-next/codex/skills/flow-next/SKILL.md` | 100 | 完美 |
| 13 个 flow-next agents | 95 | 单输出示例（缺真实 input 配对） |

### 3.2 当时值得借鉴的模式
1. **置信度标签（[VERIFIED]/[INFERRED]）**：`context-scout.md` 和 `memory-scout.md` 要求 Agent 在输出中标注信息来源的确定性，这是"防幻觉"的实用技巧。借鉴：我的 `echo-sleuth/agents/recall.md` 在返回会话记录时，可以加 `[EXACT]`（完整引用）和 `[PARAPHRASE]`（概括）标签。
2. **sync-codex.sh 双副本自动同步**：`skills/` 是权威源，`codex/skills/` 由脚本自动生成，不手工维护副本。借鉴：如果我的 `claude-for-legal` 未来需要为不同 AI 平台维护不同格式，应建立类似的单源自动生成机制。
3. **Worker Agent 的阶段化工作流**：`worker.md` 把执行分为明确的阶段（计划 → 实现 → 验证），每个阶段有完整的示例。这是多步骤 agent 文档的最佳实践。

### 3.3 当时的缺陷
1. **`epic-scout.md` 规则和 frontmatter 模型矛盾**：规则里说"用 haiku"，但声明的模型是 sonnet。根本原因：作者重构时修改了 frontmatter 但忘记同步更新规则注释，或者规则是从旧版本复制粘贴来的。自查：我的 agent 文件里有没有在正文里提到具体模型名的地方？这类"规则固化了实现细节"的写法很脆弱。
2. **`practice-scout.md` 硬编码年份 "2025"**：根本原因是把动态信息（当前年份）写进了静态文档。正确做法是用占位符 `{{CURRENT_YEAR}}` 或指向 CLAUDE.md 里的 `currentDate` 变量。自查：我的技能文档里有没有硬编码的年份？
3. **13 个 agents 只有输出模板，没有配对示例**：Agent 文档只给了"这是输出格式"，没有给"这是输入 + 对应输出"的完整示例。根本原因：作者把输出格式当成了"足够的约束"，但缺少真实示例时模型很难准确对齐。

### 3.4 当时的优化机会
1. 删除或修复 `epic-scout.md` 的规则/frontmatter 矛盾（已完成：epic-scout.md 已被删除）
2. 更新 `practice-scout.md` 的年份（已完成：已无 "2025" 字样）
3. 给 13 个 flow-next agents 各加一个配对示例（高 ROI，改善对齐质量）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `epic-scout.md` 规则/模型矛盾 | `ls plugins/flow-next/agents/epic-scout.md` | **已修复：文件已删除**（不在当前 agents 目录中）| 作者选择通过删除文件而非修复来解决矛盾 |
| `practice-scout.md` 年份 "2025" | `grep "2025" plugins/flow/agents/practice-scout.md` | **已修复**（grep 无输出，且 `plugins/flow/` 整个目录已删除）| 随着 flow v1 删除，问题一并消失 |
| eval 命令注入（Critical）in test scripts | `grep "eval" plugins/flow-next/scripts/ralph_e2e_rp_test.sh` | **仍然存在**（line 323: `eval "$(retry_cmd ...)"`)| 安全 CRITICAL 2 个月后仍未修复 |

### 4.2 架构演进
**重大变化：`plugins/flow/`（v1 版本）整个目录已被删除。**

审计时报告包含两个并行的 plugin：
- `plugins/flow/`：v1，有 agents/ commands/ skills/ 各一套
- `plugins/flow-next/`：v2，功能超集，有更多 scouts 和 codex 适配层

当前 HEAD 只剩 `plugins/flow-next/`，flow v1 已完成历史使命。同时，`flow-next/skills/` 新增了多个技能（`flow-next-map`、`flow-next-capture`、`flow-next-strategy`、`flow-next-prospect`、`flow-next-audit`、`flow-next-memory-migrate`、`flow-next-make-pr`、`flow-next-resolve-pr`、`flow-next-spec-completion-review`），audit 时的 14 个技能已增长到 20+。

**这个变化说明**：作者在 audit 后不是去修小问题，而是先完成了更大的架构收敛（删除 v1），然后继续扩展新功能。对 NL 质量问题的修复被推迟——与其说作者不重视，不如说 Roadmap 优先级不同。

### 4.3 新增的可学习模式
`flow-next/agents/` 增加了多个新侦察兵：
- `spec-scout.md`：负责检查规格文档（spec）的完整性
- `pr-comment-resolver.md`：专门处理 PR review 评论，说明作者把 PR 协作流程也纳入了多智能体编排
- `docs-gap-scout.md`：检查文档缺口——文档完整性审计作为一个独立侦察兵

这些新增侦察兵说明 Scout/Worker 架构的扩展性极好：新增一个关注维度只需新增一个 scout.md 文件，不需要改动核心工作流。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的 agent 文档没有在正文里写死模型名（避免了 epic-scout 那种矛盾）
2. 我的技能文档里没有硬编码年份（但需要 grep 验证）
3. 我的脚本里没有 `eval $VAR` 这种模式（grep 验证：无命中）

### 5.2 挑战 / 验证
本案例**挑战**了我的一个想法：**"高分（93/100）且结构精良的项目应该修复已发现的 bug"**。flow-next 的 Security CRITICAL（eval 命令注入）2 个月后仍未修复，说明即使是组织良好的项目，NL bug 修复优先级也可能被功能开发压制。这意味着**不能期望上游自我纠正，要在使用前自己评估**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件里是否有硬编码年份
find . -name "*.md" -path "*/agents/*" | xargs grep -n "202[0-9]" 2>/dev/null
# 命中后：替换为"当前年份"引用，如"refer to today's date from CLAUDE.md"

# 检查我的 agent 文件里是否在规则里写死了模型名
find . -name "*.md" \( -path "*/agents/*" -o -path "*/skills/*" \) | \
  xargs grep -n "claude-\|haiku\|sonnet\|opus" 2>/dev/null
# 命中后：把规则里的模型名改为描述性语言（"如果速度优先选择轻量模型"）

# 检查我的 agent 文件是否有配对的 input→output 示例
for f in $(find . -name "*.md" -path "*/agents/*"); do
  if ! grep -qi "## example\|## 示例\|input.*output" "$f" 2>/dev/null; then
    echo "无配对示例: $f"
  fi
done
# 命中后：至少加一个真实的输入→输出示例
```

### 6.2 灵感 → 实施路径

1. **想法**：在 `echo-sleuth` 中引入置信度标签（参考 flow-next 的 `[VERIFIED]`/`[INFERRED]`）
   - **为何可行**：echo-sleuth 的 `recall.md` 有时返回的是确切引用，有时是概括，用户无法区分
   - **第一步**：打开 `echo-sleuth/agents/recall.md`，在 Output 格式里加两个标签定义：`[EXACT]`=完整引用，`[SUMMARY]`=概括，在示例中展示用法，30 分钟

2. **想法**：参考 Scout/Worker 模式，给 `echo-sleuth` 的 `audit.md` command 拆分侦察步骤
   - **为何可行**：目前 audit 是一个大命令，里面有"找文件"+"分析模式"+"生成报告"混在一起；按 Scout/Worker 拆分后每步更专注
   - **第一步**：把 audit 的"找文件"部分独立成一个隐式 agent（`schema-scout` 已存在），让 audit command 调用 schema-scout 作为侦察步骤，20 分钟修改 `audit.md`

3. **想法**：建立版本演进追踪——flow-next 删除 v1 的经验说明 NL 工件有生命周期
   - **为何可行**：我的 `claude-for-legal` 有多个互相取代的 cold-start-interview 版本分散在不同子目录
   - **第一步**：在 `claude-for-legal` 的 README 里标注哪些技能是"当前推荐"，哪些是"遗留保留"，10 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **gmickel/flow-next 的核心目的**：通过多智能体侦察-执行工作流，提高 Claude Code 在大型代码库中完成软件开发任务的质量和可靠性
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同样是多 Agent 编排，同样有专职 scouts（schema-scout, file-historian 等） | echo-sleuth 针对历史分析，flow-next 针对代码开发任务 | 高 |
| MarkQWu/claude-for-legal | 低 | 都有 agent 文件 | 领域完全不同（法律 vs. 软件开发） | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 规则里写死模型名 | `grep -rn "claude-\|haiku\|sonnet" agents/` | 未检查（需手工核对），已知 echo-sleuth 没有这个问题 | 低 |
| Agent 只有输出格式，无配对示例 | `find agents/ -name "*.md" \| xargs grep -L "example\|示例"` | **命中全部 5 个 echo-sleuth agents + 10 个 claude-for-legal agents** | 高 |
| Command 缺 `allowed-tools` | 已知 | **echo-sleuth 4 个 commands 命中** | 中 |

**命中后的具体行动建议**：
- `echo-sleuth/agents/schema-scout.md` → 加示例：输入"分析 sessions/ 目录结构"，输出"发现 3 种文件格式..."，20 分钟
- `echo-sleuth/agents/analyze.md` → 加示例：输入"找我 2026-05 在 API 设计上的决策"，输出对应的分析报告格式，30 分钟
- `claude-for-legal/*/agents/*.md` (10 个) → 逐个添加领域相关的 input→output 示例，每个 20-30 分钟

### 8.3 别人的更优方案

1. **领域**：Scout/Worker 模式的职责分离
   - **本案例做法**：20 个专职侦察 Agent，每个只读不写，输出带置信度标签的结构化报告；Worker 收到汇总上下文后才执行修改
   - **我的项目现状**：`echo-sleuth` 的 `analyze.md` agent 既做"读取历史"又做"分析模式"，职责未分离；`claude-for-legal` 的 watcher agents 也混合了"监控触发"和"报告生成"
   - **如何借鉴**：将 `echo-sleuth/agents/analyze.md` 拆分为 `session-scout.md`（读取，输出原始数据）和 `pattern-analyzer.md`（分析，输出洞察）

2. **领域**：codex/ 双版本自动同步（sync-codex.sh）
   - **本案例做法**：`scripts/sync-codex.sh` 从权威源（`skills/`）自动生成 Codex 适配版本（`codex/skills/`），用路径替换脚本处理两个平台的路径差异
   - **我的项目现状**：没有多平台需求，暂不适用；但未来如果 `claude-for-legal` 需要在 Claude Code 和其他 AI 平台（如 Codex、Gemini）同时运行，这个模式值得参考
   - **如何借鉴**：现在无需实施，记录这个模式备用

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Command 文件 `allowed-tools` 声明
- **我的做法**：`echo-sleuth` 的 `audit.md`、`dashboard.md`、`extract.md`、`prune.md` 均有 `allowed-tools` 声明
- **本案例做法（弱在哪）**：`flow-next` 11 个 command 文件全部缺 `allowed-tools` 声明（audit 时，当前未验证是否已修复，但 Security CRITICAL 未修复说明 NL 质量问题可能也未处理）
- **意义**：在基础合规点上，我比这个 568 星的项目做得更严谨，说明我对 Claude Code command 规范的掌握比平均水平高

---

## 八、术语表

### <a name="Scout-Worker模式"></a>Scout/Worker 模式
> 多智能体设计模式。"侦察兵（Scout）"是只读的专职 Agent，每个负责收集一个维度的信息（如代码结构、安全漏洞、测试覆盖、文档状态）；"执行者（Worker）"是只写的 Agent，在侦察兵完成报告后才根据汇总的信息执行修改。类比军事：侦察部队先摸清地形，主力部队再推进——避免在信息不足的情况下盲目行动。

### <a name="monorepo"></a>monorepo（单体仓库）
> 把多个相关项目放在同一个 git 仓库里管理的策略。`flow-next` 仓库同时包含插件代码（`plugins/`）、终端 UI（`flow-next-tui/`）和 CLI 工具（`scripts/flowctl`）。优点：共享依赖和工具链；缺点：不同子项目的维护周期不同，容易相互干扰。

### <a name="命令注入"></a>命令注入（Command Injection）
> 安全漏洞。攻击者通过在输入中嵌入特殊字符（如 `;`、`$(...)`、`` ` `` ），让程序把用户数据当作命令来执行。`eval "$(retry_cmd ... "$REVIEW_SUMMARY"...)"` 中，如果 `$REVIEW_SUMMARY` 包含 `$(rm -rf /)` 这样的字符串，Shell 会在 eval 时真的执行这个命令。

### <a name="置信度标签"></a>置信度标签
> Agent 在输出中标注信息确定性程度的标记系统。flow-next 使用 `[VERIFIED]`（直接从代码/文件确认的事实）和 `[INFERRED]`（从已知信息推断的结论）。这种做法让下游 Agent 或用户能区分"确定的事实"和"合理的猜测"，减少基于推断做出错误决策的风险。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`model`、`allowed-tools`）。Claude Code 读 agent 或 command 文件时先解析 frontmatter 获取配置。如果 frontmatter 里的 `model:` 与正文规则描述的"应该用 haiku"矛盾，使用者会看到不一致的信息。
