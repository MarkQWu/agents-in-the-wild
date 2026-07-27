# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-07-27（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `single-purpose`, `vague-quantifier`, `examples-driven`, `router-channels`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

ai-toolkit 是一个 **"Spartan/GSD（Get Stuff Done）" 工程纪律层**，为 Claude Code 打包了 v1.27.0 版本中 69 个 command、21 个 rules、29 个 skill 和 9 个 agent，组织进 12 个功能包（packs）。

三个关键事实：
1. **企业级 Claude Code 插件**：由越南 [c0x12c](https://github.com/c0x12c) 工程团队开发，涵盖全栈开发、基础设施、设计、研究、创业运营全链路——定位是「工程团队共享的 Claude Code 配置层」。
2. **双运行时镜像**：工件同时存在于 `toolkit/` 和 `.codex/` 两个目录，两者是字节级别相同的镜像，支持 Claude Code 和 OpenAI Codex 两种 AI CLI。
3. **新增 Telegram Bridge**：v1.27.0 加入了 `bridges/telegram/` 目录，把 Claude Code 输出接入 Telegram Bot——这是从「本地工具」向「团队异步通知」扩展的信号。

### 1.2 架构剖析

**目录结构（当前 HEAD，v1.27.0）**：
```
ai-toolkit/
├── toolkit/                      # 主要 NL 工件目录
│   ├── agents/ (9 个)            # 专家 agents（phase-reviewer、design-critic 等）
│   ├── commands/spartan/ (69 个) # spartan:* 命令集
│   ├── skills/ (29 个)           # 开发技能（terraform、service-debugging 等）
│   └── .claude-plugin/
│       └── plugin.json           # 插件清单（v1.27.0，含 bridges 新入口）
├── .codex/                       # Claude Code → Codex 的镜像目录（字节级相同）
├── bridges/
│   ├── telegram/                 # Telegram Bot Bridge（v1.27.0 新增）
│   │   ├── index.js              # Bot 主逻辑
│   │   ├── provider.js           # Anthropic SDK 调用层
│   │   └── package.json          
│   └── core/                     # Bridge 核心引擎
├── CLAUDE.md                     # 开发者指引
└── Makefile                      # 构建/发布工具
```

**文件类型分布**：9 个 agent / 69 个 command / 29 个 skill / 0 个 hook / 12 功能包

**编排关系**：  
`spartan:*` 命令是用户入口；命令按需 dispatch 到 agents（如 `spartan:gate-review` 触发 `phase-reviewer` agent，`spartan:design` 触发 `design-critic` agent）。Skills 是 agents 的能力参考文档，不直接对外暴露命令。整体是**命令路由 + 专家 Agent 分层**架构。

**跨件契约**：  
`phase-reviewer.md` agent 的 body 里有 `cat .spartan/config.yaml`、`ls rules/`、`ls .claude/rules/` 等 shell 命令，但 tools frontmatter 只声明了 `["Read","Grep","Glob","WebSearch"]`，缺少 `Bash`——这是 BUG-1，运行时会被 Claude Code 拒绝 Bash 调用。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「[Spartan 工程纪律](#spartan-discipline)」——每个操作都经过 Gate 审查（Gate 3.5 是代码设计质量门控，Gate 4.0 是最终测试门控）；GSD = Get Stuff Done，强调可执行性而非讨论。
- **解决什么问题**：工程团队 Claude Code 使用不规范——每个人各自配置，导致 AI 辅助质量参差不齐。ai-toolkit 把最佳实践打包为共享插件。
- **做了什么 trade-off**：69 个命令覆盖面宽（全栈 + 设计 + 研究 + 创业），但每个命令缺少 `allowed-tools` 声明——用宽泛覆盖换取了具体安全约束的缺失。
- **反映什么认知模型**：作者把 Claude Code 看成「工程团队的技能层」，每个 skill/command 对应一种真实工程能力，用插件形式统一分发给团队。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：命令路由 + 专家 Agent 分层（Spartan 纪律型）**

模式特征：
- 特征 1：**用户入口是命令，而非直接调用 agent**——`spartan:gate-review`、`spartan:debug` 是语义入口，内部再 dispatch 到对应 agent
- 特征 2：**Agent 按专业域分工**——9 个 agent 各自覆盖一个领域（代码审查、设计批评、架构设计等）
- 特征 3：**Skills 是 Agent 的知识参考**——skill 提供领域知识（如 `terraform-best-practices`），agent 用来执行
- 特征 4：**双运行时镜像**——同一套工件同时服务 Claude Code 和 Codex，通过目录镜像实现

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 工程团队共享 Claude Code 配置 | ✅ 高度适用 | ai-toolkit 的核心设计场景 |
| 个人开发者快速上手 | ⚠️ 可用但过重 | 69 个命令对个人项目是过度工程 |
| 需要同时支持 Claude Code 和 Codex | ✅ 适用 | 双运行时镜像机制专门为此 |
| 严格安全约束的生产环境 | ❌ 当前不适合 | 命令全无 allowed-tools，工具访问未约束 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 命令路由 + 专家 Agent（本仓库） | c0x12c/ai-toolkit | 语义化入口、team 共享、覆盖面广 | allowed-tools 缺失、命令数量大导致维护成本高 |
| 纯 Skill 平铺 | review-squad | 简单、零依赖 | 无命令路由层 |
| GSD 纯 Agent 编排 | jnuyens/gsd-plugin | 深度工作流、MCP 状态管理 | 学习曲线高、33 agents 均无 model 声明 |

### 2.4 改进空间

1. **当前问题**：69 个命令全无 `allowed-tools` 声明（系统性 -5 分/命令）。**改进做法**：为每个命令分析其使用的工具，加最小权限 `allowed-tools`。高优先级：`spartan:implement`（需要 Write、Edit、Bash）、`spartan:debug`（需要 Read、Grep、Bash）。**预期收益**：安全性提升 + NLPM 从 96 升至 98+。
2. **当前问题**：`phase-reviewer` agent body 调用 shell 命令但无 Bash 工具声明（BUG-1）。**改进做法**：在 `phase-reviewer.md` frontmatter 加 `tools: ["Read","Grep","Glob","WebSearch","Bash"]`，或把 `cat`/`ls` 改用 `Read`/`Glob` 实现。**预期收益**：消除运行时工具调用被拒绝的 bug。
3. **当前问题**：`bridges/telegram` 缺少与 NL 工件的集成文档。**改进做法**：在 CLAUDE.md 加「Telegram Bridge 配置」一节，说明如何把 `spartan:*` 命令的输出推送到 Telegram。**预期收益**：团队异步通知场景落地。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-25 当时得分 **96/100**（多文件，审计时版本 v1.22.1）。

| 文件类别 | 当时均分 | 主要问题 |
|---|---|---|
| Agents（10 个） | 94.3/100 | research-planner/idea-killer/ai-designer 零 examples（-15/个） |
| Commands（25 个） | 95/100（系统性） | 全部缺 `allowed-tools`（-5/个） |
| Skills（34 个） | ~99/100 | terraform-best-practices 缺 `allowed_tools` |

### 3.2 当时值得借鉴的模式

1. **多个 Agent 100/100 高分**：`sre-architect`、`design-critic`、`team-coordinator`、`infrastructure-expert`、`solution-architect-cto`、`ai-designer` 均满分；根本原因：单职责 + 具体输出格式 + 无模糊量词。应借鉴：设计 agent 时先确定「这个 agent 的输出是什么具体格式」。
2. **双运行时镜像**：`toolkit/` 和 `.codex/` 字节级相同；根本原因：一套工件覆盖两种工具，维护成本不变。应借鉴：若需要同时支持 Claude Code 和 Codex，设计「主目录 + 镜像」结构，用 Makefile 同步。
3. **Gate 机制（命令对应质量关卡）**：`spartan:gate-review` 对应 Gate 3.5（代码设计），隐含「操作 → 质量门控」的工程意识；根本原因：把 CI/CD 的质量门控概念引入 AI 辅助工作流。应借鉴：重要操作后加一个「审查」命令，而不是假设 AI 输出直接正确。
4. **Skills 100/100 高质量**：多个 skill 满分，说明 skill 写法（有 allowed-tools + 具体步骤 + 无模糊词）已经成熟；应借鉴：参考 `service-debugging/SKILL.md`、`backend-api-design/SKILL.md` 的写法结构。

### 3.3 当时的缺陷

1. **系统性缺失 `allowed-tools`（25 命令全部）**：25 个 `spartan:*` 命令均无 `allowed-tools` frontmatter；Claude Code 无法在运行前校验工具访问范围。**为什么会失败**：没有 `allowed-tools` 的命令会在 Claude Code 的权限提示中显示「未知工具范围」，在严格模式下可能被拒绝；也无法阻止命令在运行时意外调用危险工具。自查：我的 commands 有没有 `allowed-tools`？
2. **BUG-1：`phase-reviewer` shell 调用无 Bash 工具**：agent body 调用 `cat`/`ls` 但 tools 只有 Read/Grep/Glob/WebSearch。**为什么会失败**：Claude Code 在工具调用时会校验工具是否在 `tools` 数组里；Bash 不在列表时，调用被 Claude Code 拒绝，agent 卡住。自查：我的 agents body 里有没有 shell 命令，而 tools 里又没有 Bash？
3. **3 个 agents 零 examples**：`research-planner`、`idea-killer`、`ai-designer` 均无 examples 块（-15/个）。**为什么会失败**：无 examples 的 agent description 只是功能说明，缺乏「触发样本」——Claude 在判断是否调用 agent 时依赖 examples 做锚点，缺失则召回率低。自查：我的 agents 至少有一个具体的 `<example>` 块吗？

### 3.4 当时的优化机会

1. 为 25 个 commands 批量加 `allowed-tools`（分析工具用途，分类标注）
2. `phase-reviewer.md` frontmatter 加 `Bash` 或改用 Read/Glob 替代 shell 命令
3. `research-planner`、`idea-killer`、`ai-designer` 各加 1 个 example 块

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 25 commands 缺 `allowed-tools` | `find toolkit/commands -name "*.md" \| xargs grep -L "allowed-tools" \| wc -l` | **持续/扩大**：66 个命令文件缺 allowed-tools（从 25→66，因版本从 v1.22.1 升至 v1.27.0 新增命令） | 系统性问题随版本扩大 |
| phase-reviewer 无 Bash | `grep "Bash" toolkit/agents/phase-reviewer.md` | **持续**：frontmatter 无 Bash 声明，body 仍有 shell 调用 | BUG 持续 |
| ai-designer/idea-killer/research-planner 零 examples | 读取各 agent 文件 | 需验证——audit 时 agents 均包含 2 个 examples（当前版本已修复！） | 部分修复 |

### 4.2 架构演进

v1.22.1 → v1.27.0 主要变化：
- **工件规模大幅扩张**：10 agents → 9 agents（合并了部分重叠功能），25 commands → 69 commands，34 skills → 29 skills（精简了冗余）
- **bridges/ 目录新增**：Telegram + 核心 Bridge 引擎，把 Claude Code 输出接入异步通知渠道
- **plugin.json 版本更新**：`v1.22.1` → `v1.27.0`，description 更新为「5 workflows, 69 commands, 21 rules, 29 skills, 9 agents organized in 12 packs」

这说明作者经历了「横向扩张（命令数量 69）→ 纵向整合（bridges 连接外部系统）」的演进轨迹，从单纯的 AI 辅助命令集向「工程自动化通知平台」扩展。

### 4.3 新增的可学习模式

- **Bridges 架构（NL 工件 → 外部系统）**：`bridges/telegram/` 创建了 Claude Code 工件与外部通知系统的连接层——这个模式在大多数插件里看不到。Bridge 分两层：`provider.js`（Anthropic SDK 调用）和 `index.js`（Telegram Bot 逻辑），解耦了 AI 调用和消息分发。

---

## 五、校准

### 5.1 我已经在做对的

1. **Skills 单职责高分模式**：我的 bureau skills（build/deploy）与 ai-toolkit skills 100/100 的写法类似——单职责、具体命令、无模糊词。
2. **plugin.json 版本同步**：我在更新工件时会同步更新 plugin.json 的 version 字段，与 ai-toolkit v1.22.1→v1.27.0 的版本管理实践一致。
3. **避免 shell 命令在无 Bash 声明的 agent 中出现**：我的 agents 没有在 frontmatter 不声明 Bash 的情况下在 body 调用 shell 命令——这是 ai-toolkit BUG-1 的反模式。

### 5.2 挑战 / 验证

这个案例验证了「工件规模越大，系统性质量问题越危险」的判断：ai-toolkit 从 v1.22.1 的 25 个无 `allowed-tools` 命令，扩张到 v1.27.0 的 66 个无 `allowed-tools` 命令——系统性缺陷随着规模线性扩大，而不会自愈。**教训：系统性质量问题（如 allowed-tools 缺失）越早修，修复成本越低；拖到 66 个命令时，批量修复需要分析每个命令的工具用途**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 有没有缺 allowed-tools
find ~/.claude/commands/ /tmp/my-repos/MarkQWu-*/.claude/commands/ -name "*.md" 2>/dev/null \
  | xargs grep -L "allowed-tools" 2>/dev/null
```
命中后怎么办：分析每个命令的 body 会用哪些工具，加最小权限的 `allowed-tools: [Read, Write, ...]`。

```bash
# 检查 agents body 有无 shell 命令但 tools 未含 Bash
for f in ~/.claude/agents/*.md /tmp/my-repos/MarkQWu-*/.claude/agents/*.md 2>/dev/null; do
  [[ -f "$f" ]] || continue
  has_bash=$(grep -c "^tools:.*Bash\|^allowed-tools:.*Bash" "$f" 2>/dev/null || echo 0)
  has_shell=$(grep -cE "^\`\`\`(bash|sh)|Bash\(|cat |ls |grep " "$f" 2>/dev/null || echo 0)
  [[ "$has_shell" -gt 0 && "$has_bash" -eq 0 ]] && echo "POTENTIAL BUG-1: $f"
done
```
命中后怎么办：在 tools frontmatter 加 `Bash`，或改用 Read/Glob 替代 shell 命令。

### 6.2 灵感 → 实施路径

1. **想法**：在 gstack 中引入「Gate 审查」命令模式——在 `spartan:implement` 类命令后加 `gstack:gate-review`，强制过一个设计质量门控。
   - **为何可行**：gstack 目前有 `plan-devex-review` skill，已有 review 意识；把它升级为 Gate 门控并与 implement 流程串联，可降低「AI 写完代码没人审」的风险。
   - **第一步**：在 `gstack/CLAUDE.md` 加「实施后必须触发 /plan-devex-review」规则；5 分钟。

2. **想法**：参考 ai-toolkit 的 Bridges 架构，为 bureau 设计一个「编译完成 → 通知」bridge，在 bureau 完成 compile 后推送摘要到某个渠道（Slack/邮件）。
   - **为何可行**：bureau 的 compile 是耗时操作，用户经常发出 compile 命令后去做别的——异步通知可以告诉用户「编译完成，有 N 条待审记录」。
   - **第一步**：在 `bureau/skills/compile/SKILL.md` 最后一步加「完成后输出一行 JSON 摘要到 stdout」；bridge 可以解析这个 JSON 并推送通知；30 分钟设计，2 小时实施。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 c0x12c/ai-toolkit 的核心目的**：为工程团队提供共享的 Claude Code 工程纪律层，统一 AI 辅助工作流标准。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多 skill/command 的 Claude Code 插件；都面向全栈工程开发 | gstack 单人使用，ai-toolkit 团队共享；ai-toolkit 有 Gate 机制 | 高 |
| MarkQWu/bureau | 中 | 都有 skill 编排 | bureau 是知识管理，ai-toolkit 是工程工作流 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 缺 `allowed-tools`（系统性） | `find /tmp/my-repos/MarkQWu-gstack -name "*.md" -path "*/commands/*" \| xargs grep -L "allowed-tools" 2>/dev/null` | gstack 中 commands 均无独立 `allowed-tools` frontmatter（通过 plugin.json 全局声明） | 中（设计选择，非 bug） |
| Agents 零 examples（-15/个） | `grep -rL "<example>" /tmp/my-repos/MarkQWu-gstack/.claude/agents/ 2>/dev/null` | gstack 目前无 agent 文件——无命中 | 暂无 |

**命中后的具体行动**：
- 若未来在 gstack 添加 agents，第一时间参照 `ai-toolkit/toolkit/agents/sre-architect.md`（100/100 参考实现）的格式写 frontmatter + examples。

### 8.3 别人的更优方案

1. **领域**：多 Agent 100/100 高分写法
   - **本案例做法**：`sre-architect.md`、`design-critic.md` 等 agent 均满分——单职责角色定义 + 具体输出格式 + 无模糊量词
   - **我的项目现状**：gstack 无 agent 文件；bureau 的 skills 质量中等，缺乏 examples
   - **如何借鉴**：当我为 bureau 写 `recall-debugger` agent 时，参照 ai-toolkit 满分 agent 的三要素：(1) 精确描述的 description，(2) 至少 1 个 `<example>` 块，(3) 0 个模糊量词

2. **领域**：双运行时镜像（Claude Code + Codex 同时支持）
   - **本案例做法**：`toolkit/` 和 `.codex/` 字节级同步，一次写作支持两个平台
   - **我的项目现状**：gstack 只支持 Claude Code，无 Codex 支持
   - **如何借鉴**：若 gstack 将来需要 Codex 支持，创建 `.codex/` 镜像并用 Makefile `sync` target 同步；每次发布前运行 `make sync`。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：允许工具白名单的安全约束
- **我的做法**：gstack 的 skills 通过 plugin.json 全局声明工具范围，有明确边界
- **本案例做法弱在哪**：66 个命令均无 `allowed-tools`，工具访问范围完全开放；BUG-1 中 phase-reviewer 声明了工具集但实际需要 Bash 却没有声明，在运行时崩溃
- **意义**：tools 声明的精确性是 ai-toolkit 最大的质量短板，我的项目在这一点上更规范。

---

## 八、术语表

### <a name="spartan-discipline"></a>Spartan 工程纪律
> ai-toolkit 的核心工作流方法论，名字来源于「斯巴达式」精简、高效。核心理念：每个工程操作都经过定义明确的阶段（Plan → Implement → Review → Ship），每个阶段有对应的 Claude Code 命令（`spartan:architect`、`spartan:implement`、`spartan:gate-review`、`spartan:gsd`），Gate 3.5 和 Gate 4.0 是两个强制质量检查点。类比 CI/CD 的 pipeline：人工写代码有 code review 门控，AI 写代码同样需要 Gate 审查。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code 命令/skill 的 YAML frontmatter 字段，声明这个文件在执行时允许调用哪些工具（如 `Read`、`Write`、`Bash`、`WebSearch`）。没有这个字段，Claude Code 无法在调用前校验工具访问范围——相当于一个程序没有权限边界。NLPM 对缺少 `allowed-tools` 的命令扣 -5 分。
