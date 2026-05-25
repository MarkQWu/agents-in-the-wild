# PeonPing/peon-ping — 学习案例

**仓库**：https://github.com/PeonPing/peon-ping
**Stars**：4,595 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-05-25（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `experience-accumulation`, `vague-quantifier`, `manifest-discipline`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
PeonPing 是一个跨平台的桌面通知和声音提醒系统，专为 Claude Code 会话集成设计——当 Claude 完成任务、需要输入或遇到错误时，通过声音/桌面通知告知用户。它不只是一个简单的 bash 脚本，而是一个完整的「声音包」生态系统（含注册表、manifest、下载验证），并在内部使用了一套叫 **[.gitban](#gitban)** 的 AI 辅助代码审查管道来开发自身。

关键事实：
- 4,595 stars，在「Claude Code 配套工具」细分市场里是代表作
- 支持 macOS / Linux / Windows / WSL2，用 bash + PowerShell + Swift + Python 实现多平台 parity
- 有 `GEMINI.md` 和 `CLAUDE.md` 两个 AI 协作文档——同时为 Claude Code 和 Gemini CLI 用户服务
- 内部开发使用 `.gitban/`——一套基于 AI agent 的代码审查管道（dispatcher → planner → executor → reviewer），是仓库最独特的「元编程」特性

### 1.2 架构剖析
- **目录结构**：
  ```
  peon.sh / peon.ps1    ← 主运行时（bash/PowerShell，内嵌 Python hook）
  scripts/              ← 30+ 脚本：hook、通知、TTS、声音下载、安装工具
  skills/               ← 5 个 SKILL.md（config/log/rename/toggle/use）
  install.sh/install.ps1 ← 安装器
  .gitban/              ← AI 辅助开发管道（见下）
    agents/             ← dispatcher/planner/executor/reviewer 四个 agent 的 inbox
    cards/              ← 任务卡片（类似 Kanban）
    hooks/              ← gitban 专属 hook
    templates/          ← 消息模板
  adapters/             ← 多 AI 平台适配
  ```
- **文件类型分布**：5 个 SKILL.md + 98 个 NL artifact（含大量 .gitban inbox 消息）+ 30+ bash/PowerShell/Swift/Python 脚本
- **编排关系**：用户安装 peon.sh → Claude Code hook 在 SessionStart/Stop 时调用 peon.sh → peon.sh 根据事件播放声音/发通知 → 用户可用 SKILL.md 里的指令通过 Claude Code 管理配置。5 个 skill 覆盖 peon-ping 的生命周期（init/config/use/toggle/log/rename）
- **跨件契约**：skills 里的所有脚本路径引用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`，但 peon-ping-log 例外地硬编码了 `~/.claude`（已知 BUG）。.gitban 消息通过命名约定（`HOOKLOG-xxx-planner-1.md`）形成可追踪的开发线程

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「声音作为 AI 交互的反馈频道」——当用户在等待 Claude 时，视觉通知效率低（需要看屏幕），声音通知更自然。这是一个「注意力管理」工具
- **解决什么问题**：Claude Code 任务完成时用户可能在做其他事，没有通知机制导致浪费等待时间
- **做了什么 trade-off**：
  - 选择 [NL表皮](#nl-表皮) + [原生二进制核心](#原生二进制核心)：5 个 SKILL.md 作为 AI 接口，实际逻辑全在 bash/PowerShell 脚本里
  - 选择声音包注册表（远程 JSON 中心化）而非捆绑声音文件——减少仓库体积，但引入了注册表信任问题
  - 选择 .gitban 管道做内部开发——把 AI 辅助代码审查流程化、可追溯，代价是 `.gitban/agents/inbox/` 有大量中间状态文件进入 git 历史
- **反映什么认知模型**：作者把 AI agent 开发理解为一个「软件工程团队协作流程」——dispatcher 是项目经理，planner 是架构师，executor 是开发者，reviewer 是 code reviewer。这套认知被具象化为 .gitban 管道

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「元编程式 AI 辅助开发管道（.gitban）」**

.gitban 是 peon-ping 最独特的贡献——它不只是一个工具，而是一套用 AI agent 开发 AI 工具的方法论。每个功能变更通过 hook 触发，生成标准化的 inbox 消息，按 planner → executor → reviewer 流水线处理，全程留有可审计的文本记录。

模式特征清单：
- **特征 1**：dispatcher/planner/executor/reviewer 四个角色，每个角色有自己的 inbox 目录
- **特征 2**：消息命名约定（`HOOKLOG-xxx-planner-1.md`）包含任务 ID + 角色 + 序列号，可追溯完整开发轨迹
- **特征 3**：所有 .gitban 消息都是 NL 工件，受 NLPM 质量评分（也被审计为 77/100 整体分数的组成部分）
- **特征 4**：hook 触发新任务卡片，planner 写实施方案，executor 执行，reviewer 验收——和传统 PR review 流程同构，但全由 AI 完成

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要严格代码审查流程的 AI 工具开发 | ✅ 高度适用 | .gitban 确保每次变更都有规划和验收 |
| 单人快速原型 | ❌ 不适用 | .gitban 流程开销太大，收益不足 |
| 团队协作但团队本身不用 AI | ❌ 不适用 | 这个模式需要 AI agent 作为「团队成员」 |
| 需要可审计开发历史的工具 | ✅ 适用 | inbox 消息构成天然的开发日志 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| .gitban AI 开发管道（本仓库） | peon-ping | 开发过程可追溯；强制代码审查；AI 流程化 | inbox 文件污染 git 历史；流程开销大 |
| 简单 CLAUDE.md 指引 | 大多数项目 | 零开销；灵活 | 无法追溯 AI 决策；无强制审查 |
| GitHub PR 流程 | 传统工程 | 成熟工具链；团队协作 | 无法原生集成 AI 流水线 |

### 2.4 改进空间
1. **当前问题**：`user_invocable: false` 与 CLAUDE.md 里「用户可通过 `/peon-ping-config` 调用」相矛盾。**改进做法**：把 SKILL.md 里的 `user_invocable` 改为 `true`，或在 CLAUDE.md 里说明这是「通过其他 skill 间接调用」。**预期收益**：消除歧义，用户实际可用该 skill
2. **当前问题**：`peon-ping-log/SKILL.md` 硬编码 `~/.claude` 而非 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`。**改进做法**：把所有 `~/.claude` 引用替换为 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`（Cursor 等非标准安装用户需要这个）。**预期收益**：修复 Cursor 用户的 log skill 静默失败
3. **当前问题**：`.gitban/agents/dispatcher/inbox/kr62ia-dispatch-log.md` 第 9 行有未展开的 shell 字面量 `$(date -u +"%Y-%m-%dT%H:%M:%SZ")`。**改进做法**：在模板里加注释提醒替换，或用 pre-commit hook 检测未展开的 `$(...)` 字面量。**预期收益**：dispatch log 显示正确的时间戳，不再误导维护者

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 当时得分 **77/100**，Security: **REVIEW**（无 CRITICAL，6 个 MEDIUM）。

| 文件类型 | 当时平均分 | 主要问题 |
|---|---|---|
| planner inbox 消息 | 74-80 | "consider" 用作指令动词（-2/次，共 15+ 次） |
| executor inbox 消息 | 80-86 | 整体质量较高，关闭式消息清晰 |
| reviewer inbox 消息 | 82-85 | "appropriate" 重复（-2/次，共 8 次） |
| skills/peon-ping-* | 84-90 | 缺 model 声明；缺输出格式；路径不一致 |
| CLAUDE.md | 82 | "proactively suggest" 轻微模糊 |

### 3.2 当时值得借鉴的模式
1. **peon-ping-use SKILL.md 的完整性** → 得分 90（该仓库最高），有完整的手动回退步骤（Manual Fallback）、错误处理协议、Cursor 兼容性说明 → 借鉴：凡是有 CLI 依赖的 skill，都应该写「如果 CLI 不可用时怎么办」的降级路径

2. **多平台 parity 的严格执行** → Unix 路径用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`，Windows 用 `$Env:APPDATA`，代码对每种平台都有专门分支 → 借鉴：如果工具要跨平台，路径相关代码必须全部参数化，不能有 hardcode

3. **结构化的 inbox 消息格式** → planner 消息格式：L1/L2/L3/L4 分级描述任务（L1=必须做，L2=要做，L3=可以做，L4=如果时间允许）→ 这是一种「优先级明确的任务卡片」格式 → 借鉴：用 AI 分配任务时，明确标注优先级级别而非用「应该做」这种模糊语言

4. **使用 `mapfile` + 参数传递而非字符串拼接** → peon.sh 内嵌 Python 时用 `sys.argv` 传参而非把路径插入到字符串里 → 这直接防止了路径注入（如路径里含空格/特殊字符）→ 借鉴：任何需要在 bash 里调用 Python 时，一律用参数传递而非字符串内插

### 3.3 当时的缺陷
1. **`user_invocable: false` 与 CLAUDE.md 矛盾** → 为什么失败：可能是开发者把 config skill 设计为「只能被其他 skill 调用」，但 CLAUDE.md 里的写法让用户以为可以直接调用 → 根本原因：缺少「skill 的调用层级」设计文档 → 自查：我的 skill 的 `user_invocable` 字段是否和实际预期一致？

2. **planner 消息大量使用 "consider" 作为指令动词** → 正确的指令动词应该是「implement」「add」「refactor」，而不是「consider implementing」——「consider」暗示可选而非必须 → 为什么重要：AI agent 执行任务时，「consider」会让 executor 觉得这是建议不是要求，可能跳过 → 自查：我的 AI 指令文件里有没有用 "consider" 代替具体动词的情况？

3. **peon-ping-log 硬编码路径** → 其他 4 个 skill 都用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`，唯独 log skill 用了 `~/.claude` → 为什么：这个 skill 可能是后来单独添加的，没有遵循已有约定 → 自查：我的脚本里有没有硬编码的路径，而不是用环境变量参数化？

### 3.4 当时的优化机会
1. 把 `user_invocable: false` 改为 `true`（或更新 CLAUDE.md 澄清调用方式）
2. 把 `peon-ping-log/SKILL.md` 里的 `~/.claude` 全部替换为 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`
3. 在 planner inbox 模板里，把 "consider" 替换为具体动词，提升指令清晰度

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `user_invocable: false` 矛盾 | `grep "user_invocable" skills/peon-ping-config/SKILL.md` | **仍然存在**：line 4 仍是 `user_invocable: false` | 约 4 周未修复，可能维护者认为这是「预期行为」 |
| 硬编码 `~/.claude` in peon-ping-log | `grep "~/.claude" skills/peon-ping-log/SKILL.md` | **仍然存在**：共 4 处 `~/.claude/hooks/peon-ping/peon.sh` | Cursor 用户的 log 功能仍然失效 |
| 未展开的 `$(date ...)` 字面量 | `grep '$(date' .gitban/agents/dispatcher/inbox/kr62ia-dispatch-log.md` | **仍然存在**：line 9 仍有 `$(date -u +"%Y-%m-%dT%H:%M:%SZ")` 字面量 | dispatch log 的时间戳记录错误持续存在 |

### 4.2 架构演进
从 audit（2026-04-26）到当前 HEAD：
- 仓库增加了 `GEMINI.md`——明确支持 Gemini CLI，跨 LLM 策略更明确
- `adapters/` 目录新增，说明在做多平台/多 LLM 适配的系统化重构
- `plans/` 目录出现，里面有 AI 协作的规划文档，说明 .gitban 的使用在深化
- `trainer/` 目录新增——可能是个人 AI 训练助手功能的扩展

**作者后来意识到什么**：peon-ping 正在从「Claude Code 专属工具」扩展为「多 LLM AI 工作流通知中心」，这是一个更大的平台化愿景。

### 4.3 新增的可学习模式
- **GEMINI.md + CLAUDE.md 双文档策略**：给不同 LLM 平台的用户分别提供定制化的协作指南，而不是一份通用 README。这是跨 LLM 设计的最佳实践之一
- **`adapters/` 目录**：把不同平台的适配层统一管理，而不是散落在各个脚本里——这和 MemPalace 的双插件策略（.claude-plugin + .codex-plugin）是类似的设计思路

---

## 五、校准

### 5.1 我已经在做对的
1. **路径参数化**：echo-sleuth 暂无 hook 脚本，规避了硬编码路径问题
2. **user_invocable 一致性**：echo-sleuth 的 SKILL.md 里没有 user_invocable 字段（默认 true），与 plugin.json 的注册意图一致，无矛盾
3. **指令动词清晰**：echo-sleuth 的 SKILL.md 用「Extract」「Store」「Search」等明确动词，没有「consider」类模糊指令

### 5.2 挑战 / 验证
- **被挑战的假设**：「.gitban 这种 AI 辅助开发管道是过度设计」。peon-ping 的代码质量（严格多平台测试、结构化 PR 流程）比大多数同类工具高得多，.gitban 功不可没。对于要长期维护的工具，这种结构化流程是有价值的投资
- **被验证的做法**：「CLAUDE.md 和 GEMINI.md 分开写」——我的 drama-workshop-skills 只有一个 README，如果未来要支持 Gemini CLI，应该参考 peon-ping 的做法

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 里有没有 "consider" 作为指令动词
grep -rn "\bconsider\b" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude \
  /tmp/my-repos/MarkQWu-claude-for-legal --include="SKILL.md" --include="*.md" 2>/dev/null | head -20
```
命中后怎么办：把「consider implementing X」改为「implement X」；「consider adding X」改为「add X」。

```bash
# 检查我的脚本里有没有硬编码 ~/.claude 路径
grep -rn "~/\.claude\b" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude \
  /tmp/my-repos/MarkQWu-claude-for-legal --include="*.sh" --include="*.py" 2>/dev/null | head -10
```
命中后怎么办：替换为 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`，确保 Cursor 等用户也能正常使用。

```bash
# 检查 echo-sleuth 的 skill 是否有手动回退路径说明
grep -rL "fallback\|Fallback\|手动\|manual" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：为每个依赖 CLI 或外部工具的 skill 加「如果工具不可用怎么办」的说明（参考 peon-ping-use 的 Manual Fallback 节）。

### 6.2 灵感 → 实施路径
1. **想法**：在 echo-sleuth 里引入简化版的 AI 开发管道（参考 .gitban 的 planner/executor 消息格式）
   - **为何可行**：echo-sleuth 是 AI 辅助工具，用 AI agent 来开发 AI 工具本身是「吃自己的狗粮」
   - **第一步**：在 echo-sleuth 里加 `.ai-dev/` 目录，放一个 `dev-planner.md`，用 L1/L2/L3 格式追踪下一步要做的功能（约 10 分钟搭架子）

2. **想法**：给 drama-workshop-skills 加 `GEMINI.md`（如果要支持多 LLM）
   - **为何可行**：drama-workshop-skills 的核心价值是「戏剧创作工作流」，这个价值不依赖 Claude 特定 API，理论上 Gemini 也可以运行
   - **第一步**：复制 drama-workshop-skills 的 CLAUDE.md，修改 Claude Code 特有的指令为 Gemini CLI 等价指令，保存为 `GEMINI.md`（约 20 分钟）

3. **想法**：借鉴 peon-ping-use 的「Manual Fallback」模式，给 echo-sleuth 的 git-mining skill 加降级路径
   - **为何可行**：git-mining skill 依赖 git 命令和 jsonl 解析，如果环境有问题需要手动路径
   - **第一步**：在 `git-mining/SKILL.md` 末尾加 `## Manual Fallback` 节，列出「如果 git 命令失败，改用这些步骤」（约 15 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 PeonPing/peon-ping 的核心目的**：给 AI 工作会话提供声音/通知反馈，让用户在等待时可以离开屏幕

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是 Claude Code 配套工具；都用 NL+脚本混合架构 | echo-sleuth 挖经验；peon-ping 发通知 | 高（架构模式借鉴） |
| MarkQWu/drama-workshop-skills | 低 | 都有 SKILL.md | 创意写作 vs 系统工具，完全不同 | 低 |
| MarkQWu/claude-for-legal | 低 | 都是多功能插件集 | 法律内容 vs 系统工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 缺 model 声明 | `grep -rn "^model:" skills/*/SKILL.md` | echo-sleuth 4/4 SKILL.md 无 model 字段 | 中 |
| skill 缺输出格式说明 | `grep -rL "## Output" skills/*/SKILL.md` | echo-sleuth 3/4 无 ## Output 节 | 高 |
| "consider" 作为指令动词 | `grep -rn "\bconsider\b" --include="SKILL.md"` | 待跑命令验证（echo-sleuth 可能命中） | 中 |

**命中后的具体行动建议**：
- `echo-sleuth/skills/experience-synthesis/SKILL.md` → 加 `## Output` 节：「输出是一个 markdown 文件 `/path/to/INSIGHTS.md`，包含 N 条经验条目，每条有标题/背景/结论/可执行建议」（约 10 分钟）
- `echo-sleuth/skills/git-mining/SKILL.md` → 加 `## Output` 节（类似）（约 10 分钟）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：Manual Fallback（降级路径）
   - **本案例做法**：`skills/peon-ping-use/SKILL.md` 有完整的 Manual Fallback 节，列出「如果 peon-ping 没有安装时，如何手动设置 hook」的步骤
   - **我的项目现状**：`echo-sleuth/skills/git-mining/SKILL.md` 依赖 git 和 jsonl 工具，但没有「工具不可用时怎么办」的说明
   - **如何借鉴**：给所有依赖外部工具的 skill 加 `## Manual Fallback` 节（5-15分钟/skill）

2. **领域**：L1/L2/L3/L4 分级任务指令
   - **本案例做法**：`.gitban/agents/planner/inbox/` 里的消息用 L1（必须）/L2（应该）/L3（可以）/L4（如果时间允许）明确分级，executor 知道优先级
   - **我的项目现状**：当我用 Claude Code 处理 claude-for-legal 的 TODO 时，描述是「有时间的话做 X，也可以做 Y」——这种描述方式会让 AI 无法判断优先级
   - **如何借鉴**：在给 AI 分配任务时，用「P0/P1/P2」或「L1/L2/L3」明确标注，而不是用「可以」「应该」这种模糊程度词

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：无硬编码路径（echo-sleuth 目前无 hook 脚本）
- **我的做法**：echo-sleuth 暂无 bash hook 脚本，因此不存在路径硬编码问题
- **本案例做法**（弱在哪）：peon-ping-log 硬编码 `~/.claude` 而非 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}`，导致 Cursor 用户的 log 静默失败
- **意义**：当 echo-sleuth 未来需要添加 hook 脚本时，应从一开始就使用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}` 模式，不要等到出问题才修复

---

## 八、术语表

### <a name="gitban"></a>.gitban
> 「Git + Kanban」的组合词。PeonPing 自创的 AI 辅助开发管道——把传统 Kanban 看板（任务管理）和 AI agent 角色（planner/executor/reviewer）结合，通过 git 仓库里的 inbox 消息文件实现协作。每次 hook 触发一个新任务，消息按「dispatcher → planner → executor → reviewer」流转，全程文本化、可 git 追踪。

### <a name="nl-表皮"></a>NL 表皮
> 见 MemPalace 案例的同名词条。peon-ping 的 NL 表皮是 5 个 SKILL.md，原生二进制核心是 peon.sh / peon.ps1 / Swift 脚本。

### <a name="原生二进制核心"></a>原生二进制核心
> 见 MemPalace 案例的同名词条。peon-ping 的「原生二进制」是指 bash/PowerShell/Swift/Python 脚本——不依赖 Claude Code 就能独立运行的实际逻辑。

### <a name="fallback-chain"></a>fallback chain（降级链）
> 当主路径失败时，系统按预定顺序尝试备选方案的机制。peon-ping-use SKILL.md 的 Manual Fallback 节就是一个降级链：`peon.sh 正常 → 如果找不到 peon.sh → 手动设置 Claude Code hook → 如果手动设置失败 → 用最简化的 echo 命令作为兜底`。

### <a name="CLAUDE_CONFIG_DIR"></a>CLAUDE_CONFIG_DIR
> Claude Code 的配置目录环境变量。默认值是 `~/.claude`（即 `$HOME/.claude`），但用户可以设置为其他路径（如 Cursor 用户可能用不同的配置目录）。最佳实践是始终用 `${CLAUDE_CONFIG_DIR:-$HOME/.claude}` 而不是硬编码 `~/.claude`，以兼容非标准安装。

### <a name="user_invocable"></a>user_invocable
> SKILL.md frontmatter 里的布尔字段，控制 Claude Code 是否在 slash command 自动补全里显示这个 skill。`true` = 用户可以直接输入 `/skill-name` 调用；`false` = 设计为只被其他 skill/command 调用，不在用户界面显示。peon-ping-config 把这个字段设为 false，但 CLAUDE.md 描述用户可以直接调用，造成矛盾。
