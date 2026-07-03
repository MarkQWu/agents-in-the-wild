# wanshuiyin/Auto-claude-code-research-in-sleep — 学习案例

**仓库**：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
**Stars**：⭐6387 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-07-03（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `cross-reference`, `manifest-discipline`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

**一句话**：这是一套为 AI 学术研究全流程设计的 Claude Code 技能库，覆盖从「想法发现」到「论文写作/投稿/返修」再到「专利申请」的完整研究工作流，并附带多个 MCP 服务器（飞书消息、Claude Review、Gemini Review、LLM Chat 等）。

关键事实：
- ⭐6387，是四个案例中星数最高的，反映了 AI 辅助学术研究的强烈需求
- 100 个 SKILL.md 文件（audit 时），覆盖 80+ 个独立技能，且持续增加（当前 HEAD 发现新技能如 `citation-audit`、`kill-argument`、`jurisdiction-format`）
- 多平台分发：`skills/`（主版本）、`skills/skills-codex/`（Codex 版）、`skills/skills-codex-claude-review/`、`skills/skills-codex-gemini-review/`
- 附带真实的 Python 工具集（`tools/`）和 MCP 服务器（`mcp-servers/`），是全栈学术研究工具包

### 1.2 架构剖析

```
Auto-claude-code-research-in-sleep/
├── skills/
│   ├── research-refine/SKILL.md     # 核心：研究精炼循环
│   ├── auto-review-loop/SKILL.md    # 自动审稿循环
│   ├── paper-{plan,write,slides,...}/SKILL.md  # 论文生命周期
│   ├── idea-{creator,discovery,...}/SKILL.md   # 想法探索
│   ├── experiment-{plan,run,audit}/SKILL.md    # 实验管理
│   ├── patent-{review,pipeline}/SKILL.md       # 专利流程
│   ├── skills-codex/                # ⚠️ Codex 版本副本（40+ 文件）
│   ├── skills-codex-claude-review/  # Claude Review 版本（8 个文件）
│   └── skills-codex-gemini-review/  # Gemini Review 版本（3 个文件）
├── tools/
│   ├── save_trace.sh               # ⚠️ CRITICAL：shell 变量注入 Python
│   ├── experiment_queue/queue_manager.py  # ⚠️ HIGH：shell=True 命令执行
│   ├── arxiv_fetch.py / exa_search.py / semantic_scholar_fetch.py
│   └── smart_update.sh / meta_opt/ watchdog.py
├── mcp-servers/
│   ├── feishu-bridge/server.py     # 飞书消息推送
│   ├── claude-review/server.py     # Claude 代码审查 MCP
│   ├── gemini-review/server.py     # Gemini 审查 MCP
│   └── llm-chat/ minimax-chat/     # 多模型对话 MCP
├── templates/                      # 工作流模板
├── aris-monitor/                   # 论文监控工具
└── assets/ docs/ tests/
```

- **文件类型分布**：100+ 个 SKILL.md + 4 套发行版 + 5 个 MCP 服务器 + 15+ Python 工具脚本
- **编排关系**：技能之间有组合关系（`research-refine-pipeline` 引用 `research-refine` + `research-review`），但通过名字引用而非 plugin.json 声明；MCP 服务器为技能提供外部 API 能力
- **跨件契约**：`tools/experiment_queue/queue_manager.py` 的 JSON manifest 驱动实验调度，技能通过 Bash 工具调用这些 Python 脚本

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「全流程垂直覆盖 + 工具链配套」——不只提供 AI 提示，还包含实现这些流程的 Python 工具；技能和工具形成闭环（skill 告诉 AI 怎么做，tool 执行具体操作）
- **解决什么问题**：学术研究是典型的「长周期、多轮迭代、跨工具」任务，单个 skill 只能解决局部问题；这套系统试图覆盖整个研究周期，让「在睡觉时 AI 自主运行研究」成为可能
- **做了什么 trade-off**：覆盖广度（100 个 skill）换来了质量分散（平均 80 分，不如专精单一领域的高分 skill）；也换来了维护复杂度（4 套发行版 × 100 个文件 = 400 个文件需要同步）
- **反映什么认知模型**：作者把 AI 看作「虚拟研究助理」，每个 skill 对应一个研究环节的专业助理（paper-plan 是计划助理，auto-review-loop 是审稿助理），整体构成「AI 研究团队」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「垂直领域全覆盖 + 工具链配套（Domain Full-Stack）」

技能库覆盖某一专业领域的完整工作流（本案例是学术研究），同时附带实现这些工作流所需的真实工具（Python 脚本、MCP 服务器）。技能是「控制层」，工具是「执行层」。

模式特征清单：
- 特征 1：**广度优先**：80+ 技能覆盖域内所有环节，没有刻意精简
- 特征 2：**技能 + 工具闭环**：每个关键技能有对应的 Python tool 或 MCP server 支撑（不只是提示）
- 特征 3：**多平台分发**：针对 Claude/Codex/Claude-Review/Gemini-Review 四平台适配
- 特征 4：**MCP 服务器生态**：5 个 MCP 服务器提供外部 API（飞书通知、多模型审查、LLM 聊天）
- 特征 5：**自监控能力**：`aris-monitor`、`tools/watchdog.py` 实现实验进度监控，支持「睡觉时 AI 自主运行」的场景

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 深度垂直领域（学术研究 / 法律 / 医疗）全流程 AI 辅助 | ✅ 高度适用 | 广覆盖是这套架构的核心价值 |
| 单次使用的简单任务 | ❌ 不适用 | 100 个技能 + 5 个 MCP 服务器安装成本过高 |
| 需要精确单个技能质量的场景 | ⚠️ 谨慎 | 广度换质量，单个技能平均 80 分 |
| 需要严格安全审查的企业环境 | ❌ 暂不适用 | 2 处 CRITICAL 安全问题未解决（v2026-07 仍存在）|
| 长周期自主 agent 场景（隔夜跑实验） | ✅ 适用 | watchdog + feishu 通知 + 自动迭代专为此设计 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 垂直领域全覆盖（本仓库） | wanshuiyin | 用户一站式，无需拼凑技能 | 维护成本极高，质量参差 |
| 单技能精深 | swiftui-pro（9 个 reference） | 质量高，context 可控 | 只解决一个问题 |
| 多命令编排引擎 | autoresearch（14 命令）| 单入口多出口，有护栏 | 不覆盖领域知识 |
| 社区聚合 | awesome-claude-code | 汇聚最佳单技能 | 无统一编排，质量不一 |

### 2.4 改进空间

1. **当前问题**：`tools/save_trace.sh` 内有 4 处 `python3 -c "...${CLI_ARG}..."` 的 shell 变量内联到 Python 字符串，CLI 参数可以注入任意 Python 代码（CRITICAL，当前 HEAD 确认仍在）。**改进做法**：把 Python 脚本写到临时文件，用 `sys.argv[1]` 传参；或改写为纯 Python 脚本、取消 Bash 调用。**预期收益**：消除 CRITICAL 安全漏洞，恢复 NLPM 安全审查通过资格。

2. **当前问题**：`skills/skills-codex/` 下约 40 个技能文件缺 `allowed-tools` frontmatter 字段。**改进做法**：通过 `tools/smart_update.sh`（仓库已有）或 sed 批量追加 `allowed-tools` 字段。**预期收益**：技能在 Codex 环境中不会因工具权限不明确而静默失败。

3. **当前问题**：核心组合技能（`research-refine-pipeline`、`auto-paper-improvement-loop`）通过名字引用子技能，但没有版本锁定。**改进做法**：在引用处加版本约束注释（`# 依赖 research-refine v2.0+`），或改用绝对路径引用。**预期收益**：子技能接口变化时，引用者收到明确信号。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **81/100**（100 个文件的加权平均）。

| 文件组 | 当时分数范围 | 主要问题 |
|---|---|---|
| `skills/skills-codex/*/SKILL.md`（40 个） | 74-85 | 缺 `allowed-tools`（-5）+ 模糊量词（-2 per 词） |
| `skills/*/SKILL.md`（主技能，60 个） | 75-85 | 模糊量词密度高（"appropriate", "ensure" 等）|
| `skills/system-profile/SKILL.md` | 75 | 主技能中唯一缺 `allowed-tools` 的 |
| Security | **BLOCKED** | 1 处 CRITICAL（save_trace.sh Python 注入）+ 1 处 HIGH（queue_manager.py shell=True）|

### 3.2 当时值得借鉴的模式

1. **飞书通知 MCP（Feishu Bridge）**：`mcp-servers/feishu-bridge/server.py` 在实验完成/异常时通过飞书消息推送通知，实现「隔夜跑实验 → 早上看手机」的使用场景。这是「异步 agent + 即时通知」的完整闭环，适合长时间无监督的 agent 任务。

2. **`research-refine/SKILL.md` 的高质量流程设计**：当时得分 83 分（主技能组中较高）——有明确的迭代阶段（Phase 1/2/3）、退出条件（quality threshold）、输出格式。这是「循环迭代 skill」的教科书设计。

3. **实验队列管理**：`tools/experiment_queue/queue_manager.py` 实现了完整的实验调度（JSON manifest → 按 GPU 资源调度 → 结果收集），配合对应的 `experiment-queue/SKILL.md`，形成「AI 写 manifest → 工具执行 → AI 分析结果」的闭环。

4. **多模型审查对比**：同一篇论文同时用 Claude Review + Gemini Review 两套 MCP 服务器审查，对比结果用于裁决改进建议。这种「多裁判制」减少了单一模型偏见。

5. **广泛的 Arxiv/SemanticScholar 工具集成**：`tools/arxiv_fetch.py`、`tools/semantic_scholar_fetch.py`、`tools/exa_search.py` 等，让 AI 能在技能执行过程中实时检索学术数据库。这是「技能 + 外部数据源」整合的实际案例。

### 3.3 当时的缺陷

1. **CRITICAL：`tools/save_trace.sh` Python 代码注入**：`python3 -c "... 'purpose': '${PURPOSE}' ..."` 这类写法让 CLI 参数（`--purpose`、`--model`、`--skill`）直接插值到 Python 字符串字面量内部。攻击者（或使用了特殊字符的合法用户）传入 `', ); __import__('os').system('id'); x=('` 这样的值，就能在 Python 进程内执行任意代码。根本原因：习惯了 bash 变量用法，没意识到嵌入 Python 源代码和嵌入文件路径是本质不同的风险。自查：如果我的工具脚本里有 `python3 -c` + bash 变量，都要检查。

2. **`skills/skills-codex/` 51 个文件缺 `allowed-tools`**：Codex 发行版的文件没有工具声明。根本原因：sync 脚本在适配 Codex schema 时，没有把 `allowed-tools` 字段纳入同步逻辑（因为 Codex 的 schema 不一定有此字段）。自查：我的所有 skill 都有 `allowed-tools`（gstack 已确认），但 bureau 的 commands 文件需要检查。

3. **模糊量词密度高（全面感染）**：主技能组 80+ 个文件中，几乎全部有 "appropriate"（-2）、"ensure"（-2）、"relevant"（-2）、"suitable"（-2）等。根本原因：100 个 skill 是逐步积累的，没有写完就过 linter 的习惯，模糊量词在积累过程中扩散。自查：我的 gstack 同样是逐步积累，同样全面感染。

### 3.4 当时的优化机会

1. **私下报告安全漏洞**：save_trace.sh CRITICAL + queue_manager.py HIGH，需要先私下报告
2. **批量修复 `allowed-tools`**：51 个 skills-codex 文件，用脚本批量追加 `allowed-tools` frontmatter
3. **减少高频模糊量词**：`appropriate`、`ensure`、`relevant` 是最高频命中词，每个出现多次，优先处理这三个

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `save_trace.sh` Python 注入 | `grep -n "python3 -c" tools/save_trace.sh` | **仍存在**：4 处（行 113/130/146/162），与 audit 时相同 | CRITICAL 问题未修复；仓库持续活跃（有新 skill）说明维护者在更新功能，未处理安全 |
| `skills-codex/` 缺 `allowed-tools` | `grep -r "allowed-tools" skills/skills-codex/ \| wc -l` | **结构已变化**：`skills-codex/` 仍作为 `skills/` 的子目录存在（当前 HEAD ls 可见），但需要抽样确认 allowed-tools 是否补齐 | 待进一步确认；从整体结构看新增了更多 skills 而非简化目录 |
| 新增技能不断增加 | `ls skills/ \| wc -l` | **增加至 80+ 个顶层技能目录**（vs audit 时 100 个总文件）；新增 `citation-audit`、`kill-argument`、`jurisdiction-format`、`vast-gpu`、`overleaf-sync` 等 | 仓库仍在活跃扩展；安全问题被「功能迭代」持续覆盖 |

### 4.2 架构演进

显著变化：
1. **新增专利领域技能**：`jurisdiction-format`、`patent-novelty-check`、`claims-drafting`（专利权利要求撰写）——从「纯学术论文」扩展到「学术 + 专利」双轨
2. **新增 `aris-monitor/` 子项目**：一个完整的论文监控系统，追踪 arXiv 上的最新相关论文
3. **新增 `vast-gpu/` skill**：针对 vast.ai GPU 云的实验调度，说明用户反馈了异构 GPU 云环境需求
4. **技能组织仍保持多层副本结构**：`skills-codex/`、`skills-codex-claude-review/`、`skills-codex-gemini-review/` 依然并存

这反映了「功能驱动 > 质量驱动」的开发模式——每次更新都是加新 skill，而不是改善现有 skill 的质量或修复安全问题。

### 4.3 新增的可学习模式

**实验监控 + 专利流程完整覆盖**（audit 时部分未覆盖）：

`aris-monitor/` 实现了论文自动跟踪——定期扫描 arXiv，筛选相关论文，生成每日摘要推送到飞书。配合 `patent-pipeline/SKILL.md` 的全流程专利撰写，这套系统从「发现论文→复现实验→创新→写作→专利」形成完整的研究价值链。

---

## 五、校准

### 5.1 我已经在做对的

1. **无 `python3 -c` + bash 变量注入**：我的工具脚本中没有使用这种模式（gstack/bureau 主要是纯 Bash 或纯 Python，没有混合）
2. **`allowed-tools` 声明**：gstack 全部 SKILL.md 均有 `allowed-tools` 字段
3. **不追求广度**：gstack 保持单一职责的技能设计，不堆积大量功能重叠的 skill

### 5.2 挑战 / 验证

这次案例最大的认知冲击：**⭐6387 的仓库，CRITICAL 安全漏洞可以存在超过 3 个月未被修复，同时功能还在持续增加**。

这说明：
1. Star 数和安全质量之间没有正相关
2. 功能迭代的速度经常大于安全问题的修复速度
3. 对于自主 agent 工具（尤其是「在睡觉时运行」的场景），用户往往不审查工具脚本

对我的影响：在使用任何外部工具脚本之前，检查 `grep -rn "python3 -c\|eval\|shell=True" tools/` 应该成为习惯。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库是否有 python3 -c + bash 变量注入模式
find /tmp/my-repos/MarkQWu-gstack \
     /tmp/my-repos/MarkQWu-bureau \
     /tmp/my-repos/MarkQWu-echo-sleuth-for-claude \
     -name "*.sh" -o -name "*.py" | \
     xargs grep -l "python3 -c" 2>/dev/null | \
     xargs grep -n 'python3 -c.*\$' 2>/dev/null
# 命中后：确认变量是否来自外部输入；若是，改用 sys.argv 传参或 tempfile

# 检查 subprocess.run 是否有 shell=True
grep -rn "shell=True" /tmp/my-repos/MarkQWu-bureau/ \
                      /tmp/my-repos/MarkQWu-graphify/ 2>/dev/null
# 命中后：确认 cmd 是否包含外部输入；若是，改用列表参数（shell=False）

# 检查我的 bureau commands 是否有 allowed-tools
grep -L "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
# 命中后：逐一添加 allowed-tools: [列出实际使用的工具]
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 添加「隔夜运行 + 飞书通知」支持，参考 wanshuiyin 的 feishu-bridge 模式
   - **为何可行**：bureau 的 recall + compile 流程可以在后台运行（积累知识），完成后推送到飞书比定时看 terminal 效率高
   - **第一步**：了解飞书机器人 API（webhook 模式），在 `bureau/hooks/on-stop.cjs` 中发送完成通知；参考 feishu-bridge 的 server.py 格式（30-60 分钟）

2. **想法**：为 gstack 中的论文/研究场景 skill 引入多裁判对比模式（类似 wanshuiyin 的 claude-review + gemini-review）
   - **为何可行**：gstack 的 `review/` skill 目前只用一个模型；用 echo-sleuth 储存两种模型的审查结果，对比找共识
   - **第一步**：在 `gstack/review/SKILL.md` 添加「对照审查」模式：先用 Claude 审查，再用另一模型审查，列出两者不同意见；20 分钟可完成 skill 修改

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：为学术研究全流程提供 AI 辅助，从想法到论文/专利，配合 MCP 服务器实现自主运行

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是知识处理类工具，都有 capture→compile→query 流程 | bureau 聚焦知识管理，不是学术研究流程 | 中（飞书通知可借鉴） |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有跨会话记忆/知识提炼能力 | echo-sleuth 做过去会话挖掘，不做新研究 | 低 |
| MarkQWu/shiji-kb | 低 | 都与知识库/研究有关 | shiji-kb 是史记历史语料库，非 AI 工具 | 无 |

若全部「无」：我的仓库中无直接目的相近的项目，本节主要做技术模式对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词密度高 | `grep -rn "appropriate\|ensure\|relevant\|suitable" gstack/*/SKILL.md \| wc -l` | **gstack 命中 400+ 次**（52 个文件，平均每文件 8 处） | 高 |
| `allowed-tools` 缺失 | `grep -L "allowed-tools" bureau/commands/*.md` | **bureau 4 个 command 文件无此字段** | 中 |
| Python 注入风险 | `grep -rn "python3 -c" gstack/ bureau/ echo-sleuth/` | **未命中**：我的仓库无此模式 | 无 |

**命中后的具体行动建议**：
- `MarkQWu-gstack/design-review/SKILL.md`（27 处命中，最严重）→ 优先重写，目标减少到 ≤3 处（每处精确化），预计 30 分钟
- `MarkQWu-bureau/commands/*.md`（4 个文件）→ 追加 `allowed-tools` frontmatter，每个 2 分钟

### 8.3 别人的更优方案

1. **领域**：飞书通知 MCP（异步 agent 通知）
   - **本案例做法**：`mcp-servers/feishu-bridge/server.py` 在实验/任务完成时自动推送飞书消息；配合 `skills/feishu-notify/SKILL.md`，agent 可以在任何步骤调用通知
   - **我的项目现状**：bureau 的长时间 recall/compile 没有完成通知机制；用户必须主动查看 terminal
   - **如何借鉴**：参考 feishu-bridge 的 MCP 格式，在 bureau 中实现 PostToolUse hook，当 `compile` 命令完成时通过 Claude Code 的系统通知（或 PushNotification 工具）推送摘要

2. **领域**：多平台分发的自动同步脚本
   - **本案例做法**：`tools/smart_update.sh` + `tools/generate_codex_claude_review_overrides.py` 实现 claude 版 → codex 版的技能同步，包含 schema 差异处理
   - **我的项目现状**：graphify 有 `skill.md` 和 `skill-opencode.md` 两个版本，完全手工维护，无同步机制
   - **如何借鉴**：写一个 `scripts/sync-opencode.sh`，从 `graphify/skill.md` 自动生成 `graphify/skill-opencode.md`（主要是替换工具名称），预计 30 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全性（无 shell 注入、无 curl|bash）
- **我的做法**：bureau 和 gstack 的工具脚本（如果有）均无 `python3 -c` + bash 变量注入模式；无 `subprocess.run(shell=True)` 与外部输入组合的风险
- **本案例做法（弱在哪）**：`tools/save_trace.sh` 有 4 处 CRITICAL 级 Python 注入，且在 3+ 个月后仍未修复；任何使用这个工具的人都面临攻击面
- **意义**：安全是我目前的相对优势，需要继续保持；在考虑新增工具脚本时，这次案例提供了一个具体的反面模板

---

## 八、术语表

### <a name="shell-injection"></a>Shell 注入 / Python 注入
> 把外部输入（用户参数、文件内容）直接拼接进 Shell 脚本或 Python 源代码字符串，让攻击者可以控制代码逻辑。`python3 -c "... '${PURPOSE}' ..."` 中，如果 `PURPOSE` 是 `'; import os; os.system("rm -rf /")'`，Python 就会执行删除命令。防御方法：把代码写成独立文件，用 `sys.argv[1]` 接收参数，永远不要把外部输入拼接进代码字符串。

### <a name="mcp-server"></a>MCP 服务器（Model Context Protocol Server）
> Claude Code 的能力扩展机制。MCP 服务器是一个独立运行的进程，通过标准接口向 Claude 提供额外工具（如「发送飞书消息」、「调用 Gemini API 审查代码」）。Claude 可以在技能执行过程中调用 MCP 工具，就像调用内置工具（Bash、Read）一样。

### <a name="queue-manager"></a>实验队列管理（Experiment Queue）
> 一种调度模式：把「要跑的实验」写成 JSON manifest，由 queue_manager 按照 GPU 资源可用性依次执行。类似任务队列（如 Celery、RabbitMQ），但专为 AI 实验调度设计。wanshuiyin 的实现中，Claude 通过 `experiment-queue` skill 操作 manifest，queue_manager.py 执行实际调度。

### <a name="multi-judge"></a>多裁判制（Multi-Judge Pattern）
> 对同一份内容用多个不同模型（Claude、Gemini、MinMax 等）分别评审，然后对比结果、找共识或争议点。wanshuiyin 的做法是：同一篇论文同时通过 claude-review MCP 和 gemini-review MCP 审查，差异点作为最高优先级修改点。这种做法能减少单一模型的系统性偏见。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md 或 command.md [frontmatter](#frontmatter) 中声明「此技能允许使用的工具列表」的字段。`skills-codex/` 下缺少此字段，意味着在 Codex 环境（工具权限受限）下，技能可能尝试调用未授权工具并静默失败，或调用超出预期的工具。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 元数据区块，声明 `name`、`description`、`allowed-tools`、`version`、`tags` 等。Claude Code 或 Codex 先解析 frontmatter，才知道如何注册和约束这个技能的行为。
