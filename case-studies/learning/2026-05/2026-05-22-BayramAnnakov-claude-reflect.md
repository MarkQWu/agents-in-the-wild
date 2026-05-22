# BayramAnnakov/claude-reflect — 学习案例

**仓库**：https://github.com/BayramAnnakov/claude-reflect
**Stars**：1000 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `manifest-discipline`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Code 的自学习系统插件。通过钩子自动捕获用户对 Claude 的纠错行为（如"不，要用 X"、"其实应该…"），将学习内容排队，用户运行 `/reflect` 命令后手动审查并写入 CLAUDE.md 或规则文件，实现跨会话经验积累。版本 3.1.0，作者 Bayram Annakov，通过 `claude plugin install` 安装。

关键事实：
1. 两阶段架构：Stage 1（自动捕获）+ Stage 2（手动消化），分工明确
2. 学习目标有 5 个层次：全局 `~/.claude/CLAUDE.md`、项目 `./CLAUDE.md`、个人私有 `./CLAUDE.local.md`、模块化规则 `./.claude/rules/*.md`、自动记忆 `~/.claude/projects/<project>/memory/*.md`
3. 纯 Python 标准库实现（无 pip 依赖），全平台可用
4. 有完整的 4 事件钩子系统（SessionStart、UserPromptSubmit、PreCompact、PostToolUse）
5. 版本 3.0.x → 3.1.0 期间新增了 tests/ 目录和 5 个测试文件，说明作者在主动提升代码质量

### 1.2 架构剖析
- **目录结构**：
```
claude-reflect/
  commands/
    reflect.md          # 主命令：审查队列、写入学习
    skip-reflect.md     # 跳过当前队列项
    reflect-skills.md   # 从已有 SKILL.md 生成新技能
    view-queue.md       # 查看待审队列
  hooks/
    hooks.json          # 4 个 hook 事件注册
  scripts/
    capture_learning.py       # UserPromptSubmit 捕获纠错
    check_learnings.py        # PreCompact 汇总
    post_commit_reminder.py   # PostToolUse 提醒
    session_start_reminder.py # SessionStart 欢迎
    extract_session_learnings.py
    extract_tool_errors.py    # v3.1.0 新增
    extract_tool_rejections.py
    compare_detection.py
    read_queue.py
    lib/
      reflect_utils.py
      semantic_detector.py    # 语义分析，调用 Claude CLI
      __init__.py
    legacy/                   # 已废弃 bash 脚本
  tests/
    test_integration.py
    test_reflect_utils.py
    test_semantic_detector.py
    test_memory_hierarchy.py  # v3.1.0 新增
    test_tool_errors.py       # v3.1.0 新增
  SKILL.md, CLAUDE.md
  CHANGELOG.md, DISTRIBUTION.md, RELEASING.md
  .claude-plugin/
    plugin.json         # 插件 manifest（v3.1.0）
    marketplace.json
```
- **文件类型分布**：4 个 command，1 个 SKILL.md，1 个 hooks config，12 个 Python 脚本（active），5 个 legacy bash 脚本，5 个测试文件，0 个 agent
- **编排关系**：hooks → Python 脚本（后台自动捕获） + commands → Python 脚本（用户主动消化）。两个通道独立运作，通过 `~/.claude/learnings-queue.json` 共享状态。不存在 router 层，每个命令直接调用对应脚本
- **跨件契约**：`hooks/hooks.json` 里的 4 个脚本路径全部有效且存在。CLAUDE.md 文档说 [manifest](#manifest) 「points to hooks」但 `plugin.json` 里实际上没有 `hooks` 字段——这是 Bug #1，见 §3.3

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「两阶段捕获-消化循环」——机器自动观察（Stage 1），人主动决策（Stage 2）。Claude 的每次纠错都是潜在的学习机会，但把学习写哪里、写什么，由人决定而非自动化
- **解决什么问题**：Claude Code 没有跨会话的「自我改进」机制。每次新建会话，Claude 都从同一个起点出发，用户对它的纠正反复在重复。`claude-reflect` 把纠错捕获下来，积累成可复用的行为规则
- **做了什么 trade-off**：自动捕获降低了用户的记录负担，但「需要用户运行 /reflect 来消化」意味着如果用户不定期执行，队列会积压。更彻底的自动化（直接写入 CLAUDE.md）风险是误学——作者有意保留人工确认这道闸
- **反映什么认知模型**：作者把 AI 学习视为「有监督的渐进积累」，不是「大模型一次性学会」。每条学习是一个有 context 的决策，需要人来判断是通用规则还是特例。这是一种务实的、经验主义的 AI 协同观

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「两阶段捕获-消化循环」（Two-Stage Capture-Digest Loop）**

这个模式的关键特征是：系统分为「自动观察层」（hooks 驱动）和「人工决策层」（commands 驱动），两层共享一个持久化队列（`~/.claude/learnings-queue.json`），人工层消费自动层的产出。

模式特征清单（4 条）：
- 特征 1：后台自动捕获不打扰用户，只把候选学习项写入队列
- 特征 2：前台命令让用户以「审稿人」身份逐项 confirm / skip / edit
- 特征 3：学习目标分层（全局 / 项目 / 私有 / 模块化规则），不同知识去不同容器
- 特征 4：语义分析（`semantic_detector.py`）用来过滤噪声，只把真正的纠错行为写入队列，而不是所有 UserPromptSubmit 事件

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 长期使用 Claude Code 的个人用户，想积累工作流偏好 | ✅ 高度适用 | 核心价值场景，积累越多越有用 |
| 团队共享项目，需要统一 AI 行为规范 | ✅ 适用 | 学习可写入 ./CLAUDE.md（项目层）共享 |
| 一次性脚本任务 | ❌ 不适用 | 积累的经验没有下次调用的机会 |
| 需要完全自动化（无人值守）的 CI 环境 | ❌ 不适用 | Stage 2 的人工确认步骤需要交互式终端 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 两阶段捕获-消化循环（本仓库） | claude-reflect | 自动捕获 + 人工确认，平衡自动化与可控性 | 需要用户定期执行 /reflect，队列可能积压 |
| 纯手动记忆挖掘 | MarkQWu/echo-sleuth-for-claude | 无需后台钩子，按需查阅历史 JSONL | 完全依赖用户主动，忘了就丢失了 |
| 全自动写入（无人工确认） | 假想设计 | 零摩擦，最及时 | 误学风险高，垃圾内容直接污染规则文件 |

### 2.4 改进空间
1. **当前问题**：`plugin.json` 没有 `hooks` 字段，全部 4 个钩子可能在全新安装时静默失效（见 Bug #1）。**改进做法**：在 `.claude-plugin/plugin.json` 里加 `"hooks": "../hooks/hooks.json"` 字段（或框架约定的等效写法）。**预期收益**：消除 Stage 1 自动捕获在新安装时失效的风险，这是整个插件核心能力的保证。
2. **当前问题**：`commands/reflect.md` 的上下文注入用 `$0` 解析插件根路径，而其他命令用 `${CLAUDE_PLUGIN_ROOT}`，两种模式并存。**改进做法**：统一改为 `${CLAUDE_PLUGIN_ROOT}`，与 `skip-reflect.md` 和 `reflect-skills.md` 保持一致。**预期收益**：消除路径解析失败时的静默降级（当前是 `|| echo "[]"` 兜底，用户看不到错误）。
3. **当前问题**：`commands/reflect.md` 中「appropriate heading」「relevant section header」「makes semantic sense」等表述没有给出判断标准，Claude 每次执行时可能做出不一致的选择。**改进做法**：在 reflect.md 中加一张「学习类型 → 目标 heading」的映射表，把「relevant」变成「查表得出的具体 heading」。**预期收益**：消除 3 处 vague quantifier 扣分，且实际行为更可预测。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **99/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/reflect.md | 94 | 模糊量词：「appropriate heading」「relevant section header」「makes semantic sense」（-6），$0 路径解析脆弱（informational） |
| commands/view-queue.md | 97 | `Read` 声明在 allowed-tools 但从未使用（-3） |
| commands/reflect-skills.md | 98 | 模糊量词：「Relevant tools based on workflow」（-2） |
| commands/skip-reflect.md | 100 | 无 |
| SKILL.md | 100 | 无 |
| CLAUDE.md | 100 | 无 |
| hooks/hooks.json | 100 | 无 |
| .claude-plugin/plugin.json | 100（跨组件问题另列） | 无直接扣分，但有跨组件 Bug（缺 hooks 字段） |

### 3.2 当时值得借鉴的模式
1. **单职责命令分工**（single-purpose）→ 为什么好：4 个命令各管一件事（审查 / 跳过 / 生成技能 / 查看队列），没有一个「大而全」的主命令，每个文件的意图一眼可读 → 原文路径：`commands/` 目录整体结构 → 如何借鉴：自己的插件如果命令逻辑超过 3 步，考虑拆分为独立命令
2. **多层学习目标**（experience-accumulation）→ 为什么好：知识按「全局 / 项目 / 私有 / 规则模块」分层存储，全局记忆不被项目特定知识污染，私有偏好不提交到 git → 原文路径：`SKILL.md` 的学习目标说明 → 如何借鉴：设计记忆系统时先定「容器层级」，而不是把所有内容堆进一个文件
3. **语义过滤减少噪声**（single-purpose）→ 为什么好：`semantic_detector.py` 在写入队列前先判断是否真的是「纠错信号」，而不是把所有 UserPromptSubmit 都存下来，避免了队列快速膨胀 → 原文路径：`scripts/lib/semantic_detector.py`，`scripts/capture_learning.py` → 如何借鉴：后台捕获型钩子要有过滤层，不要无脑写入
4. **legacy/ 隔离废弃代码**（single-purpose）→ 为什么好：旧 bash 脚本没有删除而是移到 `scripts/legacy/` 并有明确的废弃说明，不在生产路径上，但保留了历史参考 → 原文路径：`scripts/legacy/` → 如何借鉴：废弃代码不要就地注释掉，移到专门目录并加 DEPRECATED 标注
5. **hooks.json 与脚本路径完全对齐**（manifest-discipline）→ 为什么好：audit 核查了 hooks.json 里的 4 个脚本路径，全部真实存在且名称匹配，无悬空引用 → 原文路径：`hooks/hooks.json` → 如何借鉴：每次新增或重命名脚本，立即同步检查 hooks.json

### 3.3 当时的缺陷
1. **Bug #1（高影响）：`plugin.json` 缺 `hooks` 字段**：`.claude-plugin/plugin.json` 里没有指向 `hooks/hooks.json` 的字段，而 CLAUDE.md 声称 manifest 「points to hooks」。为什么会失败：如果 Claude Code 插件框架不做自动钩子发现（这点文档未明确），全部 4 个钩子（SessionStart、UserPromptSubmit、PreCompact、PostToolUse）在全新安装时将静默不注册。Stage 1 自动捕获完全失效，用户不会看到任何错误提示，只是默默没有学习效果。**自查**：我的 `plugin.json` 里有没有 `hooks` 字段？
2. **`reflect.md` 使用 `$0` 解析插件根路径**：上下文注入行 `!python3 "$(dirname "$(dirname "$(readlink -f "$0")")")/scripts/read_queue.py"` 依赖 `$0`，但在 Claude Code 的 `!` 上下文里 `$0` 是 shell 解释器路径，不是 skill 文件路径。失败时用 `|| echo "[]"` 静默降级。其他命令（skip-reflect.md、reflect-skills.md）正确使用了 `${CLAUDE_PLUGIN_ROOT}`。为什么会失败：路径解析错误导致读不到队列，用户运行 /reflect 时看到空队列，误以为「没有待审学习」。**自查**：我的命令文件里有没有 `$0` 路径依赖？
3. **`view-queue.md` 声明了从未使用的 `Read` 工具**：`allowed-tools: [Read, Bash]` 但命令体全程只调 `Bash`（`python3 scripts/read_queue.py`）。为什么会失败：NLPM 对「声明但未使用的工具」扣 -3 分（R22 规则），更重要的是会误导 Claude Code 运行时误以为该命令会执行文件读取，影响权限决策。**自查**：我的命令文件 `allowed-tools` 里有没有多余声明？

### 3.4 当时的优化机会
1. **给 `plugin.json` 加 `hooks` 字段**：一行改动，消除 Stage 1 静默失效风险，影响极高，代价极低
2. **统一 `reflect.md` 到 `${CLAUDE_PLUGIN_ROOT}`**：把 3 层嵌套的 `$(dirname ...)` 替换为 `${CLAUDE_PLUGIN_ROOT}/scripts/read_queue.py`，与其他命令保持一致
3. **把 `reflect.md` 里的 vague quantifier 替换成查表逻辑**：在命令体里加一个「学习类型 → 目标 heading」映射表，消除「appropriate」「relevant」「semantic sense」三处模糊描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| `plugin.json` 缺 `hooks` 字段 | `grep -i "hooks" .claude-plugin/plugin.json` → 结果：MISSING，plugin.json 内容仅含 name/version/description/author/repository/license | **持续存在** | Bug #1 完全未修复，全新安装时 Stage 1 自动捕获仍可能静默失效 |
| `reflect.md` 使用 `$0` 路径解析 | `grep -n '\$0' commands/reflect.md` → 第 541、1312 行仍有 `$0`；`grep -n 'CLAUDE_PLUGIN_ROOT' commands/skip-reflect.md` → 存在 | **持续存在** | 路径解析与其他命令不一致的问题仍在，静默降级风险未消除 |
| `reflect.md` 的 vague quantifier | `grep -n 'appropriate\|relevant\|semantic sense' commands/reflect.md` → 第 541、1295、1312、1342 行命中 | **持续存在** | 3 处模糊量词全部保留，评分仍为 94/100 |

### 4.2 架构演进
从审查时的文件清单到当前 HEAD，有以下变化：

- **新增**：`tests/test_memory_hierarchy.py`、`tests/test_tool_errors.py`（v3.1.0 的测试扩展）；`scripts/extract_tool_errors.py`（新的工具错误追踪脚本）；`DISTRIBUTION.md`、`RELEASING.md`（发布文档）
- **版本**：3.0.x → 3.1.0（minor bump）
- **未变**：全部 4 个 commands 文件结构未变；`plugin.json` 内容未变（Bug #1 仍在）；`hooks/hooks.json` 未变

这说明作者在 v3.1.0 的关注点是：扩展追踪能力（工具错误 = 新的学习信号来源）、提升测试覆盖（memory_hierarchy 和 tool_errors 的专项测试），以及完善发布流程文档。所有审查发现的 NL 层 bug 均未触及，作者的优先级在功能扩展而非 NL 规范修复。

### 4.3 新增的可学习模式
**工具错误作为学习信号**：v3.1.0 新增的 `scripts/extract_tool_errors.py` 和 `tests/test_tool_errors.py` 说明作者把「工具调用失败」也纳入了自学习的候选信号来源。这是对「纠错行为」定义的扩展——不只是用户显式说「不对，要用 X」，工具报错也是潜在的行为模式异常信号。这个扩展思路值得借鉴：自学习系统的信号源可以是多元的（用户纠错、工具失败、会话紧凑前的总结等）。

---

## 五、校准

### 5.1 我已经在做对的
1. **commands→agents/scripts 分层**：echo-sleuth-for-claude 的架构是「commands dispatch to agents, agents use skills for domain knowledge and call scripts (via Bash tool) for data extraction」，与 claude-reflect 的「command → Python script → 学习目标」有相同的分层思路——NL 层指挥，执行层（脚本/代理）干活
2. **manifest 同步纪律**：我的 plugin.json 没有出现「文件存在但未注册」的情况，每次新增命令/技能时都同步更新 manifest，与本仓库的 hooks.json ↔ scripts 对齐做法相同
3. **单职责命令分工**：echo-sleuth 的 8 个命令（recall/recap/timeline/lessons/dashboard/audit/extract/prune）各管一件事，与 claude-reflect 的 4 命令分工思路一致
4. **allowed-tools 对应工具**：我的所有命令文件有 `name:` 字段，且 allowed-tools 声明基本与实际调用对应（无冗余声明）

### 5.2 挑战 / 验证
- **这次案例验证了一个我之前的做法是对的**：把「信息收集」和「用户决策」拆成两个阶段。echo-sleuth 的 `extract` 命令负责挖掘，`recall`/`lessons` 等命令负责展示决策，与 claude-reflect 的 Stage 1（自动捕获）+ Stage 2（手动消化）高度同构。两个项目不约而同地选择了「先收集、后决策」模式，说明这是处理「历史信息 → 行为规则」转化的普遍有效模式。
- **这次案例挑战了我的一个假设**：「99/100 高分项目的 bug 应该已经被修复了」。实际上 Bug #1（plugin.json 缺 hooks 字段）和所有 vague quantifier 在 v3.1.0 中依然存在。高 NL 分数和 bug 修复率是独立的：高分说明绝大多数文件写得好，但不保证作者会回头修已知问题。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查 plugin.json 是否有 hooks 字段（类比 Bug #1）
grep -i '"hooks"' .claude-plugin/plugin.json && echo "OK: hooks field present" || echo "MISSING: no hooks field in plugin.json"
# 命中（MISSING）后：参考 claude-reflect Bug #1，检查框架文档确认是否需要显式声明，若需要则补加字段

# 检查所有命令文件是否使用了 $0 做路径解析
grep -rn '\$0' commands/*.md 2>/dev/null && echo "FOUND: \$0 usage in commands" || echo "OK: no \$0 usage"
# 命中后：替换为 \${CLAUDE_PLUGIN_ROOT}，与其他命令保持一致

# 检查 allowed-tools 声明了但实际从未在文件体中调用的工具
for f in commands/*.md; do
  declared=$(grep -A2 'allowed-tools:' "$f" | grep -oE 'Read|Write|Bash|Edit|MultiEdit|WebFetch|TodoWrite' | sort -u)
  for tool in $declared; do
    grep -q "$tool" "$f" || echo "POSSIBLY UNUSED $tool in $f"
  done
done
# 命中后：审查命令体，确认是否真的调用了该工具，未调用则从 allowed-tools 移除

# 检查 skill/command 文件中的模糊量词（类比 R14 规则）
grep -rn -iE '\b(appropriate|relevant|suitable|reasonable|as needed|as necessary|where it makes sense|semantic sense)\b' commands/*.md skills/*/SKILL.md 2>/dev/null
# 命中后：把每个命中的描述替换为「具体的判断标准」或「查表逻辑」，不允许 Claude 自行发挥
```

### 6.2 灵感 → 实施路径
1. **想法**：为 echo-sleuth-for-claude 的 `extract` 命令增加一个「工具调用失败」信号采集，类比 claude-reflect v3.1.0 的 `extract_tool_errors.py`
   - **为何可行**：echo-sleuth 已经有 `extract` 命令从 JSONL 挖掘数据，工具错误信号同样记录在 Claude Code 的 JSONL 里，只需要新增一个过滤规则
   - **第一步**：在 `agents/extract.md` 里新增一个「工具错误模式」识别步骤，从 JSONL 里提取 `tool_result` 类型且包含错误内容的记录；对应新建 `scripts/extract_tool_errors.py`，预计 30-45 分钟
2. **想法**：给 echo-sleuth 的 4 个缺少 `allowed-tools` 的命令文件补齐声明（`commands/lessons.md`、`commands/timeline.md`、`commands/recap.md`、`commands/recall.md`）
   - **为何可行**：类比 claude-reflect 的 `view-queue.md` 问题，缺少 `allowed-tools` 声明会触发 NLPM 扣分，也可能影响运行时权限判断
   - **第一步**：对 4 个文件逐一检查命令体实际调用了哪些工具（`Bash` 或 `Read`），然后在各自 [frontmatter](#frontmatter) 里加上 `allowed-tools:` 声明，每个文件约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 BayramAnnakov/claude-reflect 的核心目的**：帮助 Claude Code 用户跨会话积累经验，将纠错信号转化为持久化的行为规则
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同为「meta-tool」，帮助 Claude 从历史信息中学习；都有「收集 → 呈现/消化」的两阶段结构；都是 Claude Code 插件生态中的记忆扩展工具 | claude-reflect 通过 hooks 自动捕获纠错信号，echo-sleuth 需要用户主动 `/extract` 挖掘；claude-reflect 写入规则文件，echo-sleuth 侧重展示和回顾；claude-reflect 有 5 个测试文件，echo-sleuth 无测试 | 高 |
| MarkQWu/drama-workshop-skills | 无 | — | 短剧编剧工具，与记忆/学习主题无关 | 低 |
| MarkQWu/claude-for-legal | 无 | — | 法律插件，与记忆/学习主题无关 | 低 |

### 8.2 在我的项目里复现的同类问题

对 §3.3「当时的缺陷」和 §4.1「现在仍存在的缺陷」逐条核查 `MarkQWu/echo-sleuth-for-claude`：

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺 hooks 字段（Bug #1） | `grep '"hooks"' .claude-plugin/plugin.json` | echo-sleuth 无 hooks 系统，不适用；但 plugin.json 是否正确列出所有 commands/agents 需检查 | 中（若有 hooks 则为高） |
| allowed-tools 声明冗余/缺失 | `grep -rn 'allowed-tools' commands/*.md` | echo-sleuth 的 4 个命令文件（lessons/timeline/recap/recall）**缺少** allowed-tools 声明 | 高 |
| vague quantifier（模糊量词） | `grep -rn -iE '\b(relevant|appropriate|suitable)\b' skills/*/SKILL.md` | `skills/git-mining/SKILL.md:93` 命中「relevant git commits」；`skills/experience-synthesis/SKILL.md:118` 命中「relevant sessions」 | 中 |
| $0 路径解析 | `grep -rn '\$0' commands/*.md agents/*.md` | echo-sleuth 未使用 $0 路径（使用 CLAUDE_PLUGIN_ROOT） | 无 |

**命中后的具体行动建议**：
- `echo-sleuth/commands/lessons.md` → 检查命令体调用了哪些工具（`Bash`）→ 在 frontmatter 加 `allowed-tools: [Bash]` → 5 分钟可完成
- `echo-sleuth/commands/timeline.md`、`commands/recap.md`、`commands/recall.md` → 同上处理 → 合计约 15-20 分钟
- `echo-sleuth/skills/git-mining/SKILL.md:93` → 把「relevant git commits」改为具体标准（如「距当前分支最近 50 条 commit，或 HEAD 到 base branch 的所有 commit」）→ 5 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：自动化信号捕获（hooks 驱动）
   - **本案例做法**：`hooks/hooks.json` 注册 4 个 hook 事件，后台自动捕获纠错行为，用户不需要记住「要去记录这件事」。`capture_learning.py` + `semantic_detector.py` 形成「捕获 + 过滤」流水线（`hooks/hooks.json`，`scripts/capture_learning.py`，`scripts/lib/semantic_detector.py`）
   - **我的项目现状**：`MarkQWu/echo-sleuth-for-claude` 完全没有 hooks 系统，用户必须主动运行 `/extract` 才能从 JSONL 里挖掘信息。如果用户几天没有运行，这段时间的学习机会就流失了
   - **如何借鉴**：为 echo-sleuth 新增一个 `UserPromptSubmit` hook，在每次会话中自动记录「新的值得回顾的 session 标记」到轻量队列；新增 `hooks/hooks.json` 和对应 `scripts/auto_tag_session.py`；在 `plugin.json` 里加 `hooks` 字段。改动范围：3 个新文件 + 1 行 plugin.json 修改

2. **领域**：测试覆盖（Python 脚本的可验证性）
   - **本案例做法**：`tests/` 目录下有 5 个测试文件（`test_integration.py`、`test_reflect_utils.py`、`test_semantic_detector.py`、`test_memory_hierarchy.py`、`test_tool_errors.py`），覆盖核心 lib 和集成行为，v3.1.0 还在扩展
   - **我的项目现状**：`MarkQWu/echo-sleuth-for-claude` 无任何测试文件（`tests/` 目录不存在）。所有 Python 脚本（`scripts/` 下的 data extraction 脚本）只能通过实际运行来验证
   - **如何借鉴**：从最核心的脚本开始，给 `scripts/extract.py` 或 `scripts/lib/` 下的 helper 函数写 `unittest`，用真实 JSONL 样本做 fixture；新建 `tests/test_extract.py`，预计 45-60 分钟

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：agents 分层设计
   - **我的做法**：`MarkQWu/echo-sleuth-for-claude` 有 5 个 agent（`analyze`、`file-historian`、`memory-auditor`、`recall`、`schema-scout`），形成「commands → agents → skills → scripts」4 层架构，每层职责清晰
   - **本案例做法（弱在哪）**：claude-reflect 没有 agent 层，commands 直接调用 Python 脚本，中间没有「可推理的中间层」。对于复杂的学习场景（如需要跨多个 CLAUDE.md 做冲突检测），这种扁平结构会让 command 文件越来越臃肿
   - **意义**：echo-sleuth 的 4 层架构是 NLPM 最推荐的「分层编排」模式，在未来审查中这是架构亮点。若考虑给 claude-reflect 提 PR，可以建议把复杂的「学习目标路由逻辑」提取为独立 agent

---

## 八、术语表

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`.claude-plugin/plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills 的路径。如果 manifest 里漏写了某个组件（比如 hooks），那个组件即使在硬盘上存在也不会被框架加载或注册。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools`、`model` 等）。Claude Code 读 command/skill 文件时先解析 frontmatter 才能知道如何注册和调用。`---` 必须严格从行首（第 1 列）开始，否则不被识别。

### <a name="hook"></a>hook（钩子）
> Claude Code 在特定事件发生时自动触发的外部脚本。例如 `SessionStart`（新会话开始）、`UserPromptSubmit`（用户提交消息）、`PreCompact`（压缩上下文前）、`PostToolUse`（工具调用后）。`hooks/hooks.json` 定义了每个事件触发哪个脚本。claude-reflect 用这 4 个事件实现 Stage 1 的自动学习捕获。

### <a name="vague-quantifier"></a>vague quantifier（模糊量词）
> 指自然语言指令中「无法被验证或精确执行」的描述词，如「appropriate」（适当的）、「relevant」（相关的）、「as needed」（按需）、「where it makes sense」（有意义的地方）。NLPM 的 R14 规则对每处模糊量词扣 -2 分，因为 Claude 看到模糊指令时会「自行发挥」，导致每次执行结果不一致。修复方法：用具体判断标准或查表逻辑替代。

### <a name="stage1-stage2"></a>两阶段捕获-消化循环
> claude-reflect 的核心架构模式。Stage 1（自动）：由 hooks 后台静默监测 Claude Code 会话，用语义分析识别「用户正在纠错 Claude」的信号，把候选学习项写入 `~/.claude/learnings-queue.json` 队列。Stage 2（手动）：用户运行 `/reflect` 命令，逐项审查队列，决定把每条学习写入哪个目标文件，或跳过。两个阶段解耦，Stage 1 负责「无遗漏地捕获」，Stage 2 负责「有质量地消化」。
