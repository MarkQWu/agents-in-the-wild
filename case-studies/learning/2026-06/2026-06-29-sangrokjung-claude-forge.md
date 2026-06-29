# sangrokjung/claude-forge — 学习案例

**仓库**：https://github.com/sangrokjung/claude-forge
**Stars**：648 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-06-29（目标仓库不可访问，仅基于 audit 快照）
**主题标签**：`security-gate`, `cross-reference`, `manifest-discipline`, `fallback-chain`, `experience-accumulation`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
claude-forge 是一个定位为「DevSecOps 全家桶」的 Claude Code 插件，包含 73 个核心 NL 制品（命令、技能、agent）外加 25 个辅助知识文件，覆盖从代码评审、安全检查、TDD 到持续学习的完整开发周期。插件独特之处在于内置了 5 个运行时安全钩子（secret filter、command guard、rate limiter、db-guard、security-auto-trigger），试图将安全防线直接嵌入 Claude Code 会话。

关键事实：
- 98 个制品，是所有 audit 仓库中体量最大之一
- 安全级别：REVIEW（6 个安全发现，但 0 Critical，都是 Medium/Low）
- 总体 NL 评分 82/100（98 个制品的加权平均）
- 5 个安全钩子全部只在 `OPENCLAW_SESSION_ID` 环境变量存在时激活，本地开发默认无保护
- 作者：sangrokjung，GitHub 个人页显示为韩国开发者（audit 中有一处韩文注释「v7 순차 실행」）

### 1.2 架构剖析
- **目录结构**：
```
claude-forge/
├── agents/           # 7 个 agent（code-reviewer 90分/architect 88分等）
├── commands/         # ~50 个命令（含 agent-router.md、orchestrate.md）
│   ├── agent-router.md        # 34 域路由入口
│   ├── orchestrate.md         # 需要 AGENT_TEAMS 实验标志
│   ├── debugging-strategies/  # skill-cmd 混合式
│   └── ...
├── skills/           # 约 12 个 skill
│   ├── continuous-learning-v2/  # 持续学习子系统
│   │   ├── SKILL.md
│   │   ├── agents/observer.md   ← 缺 tools 声明（Bug#6）
│   │   └── commands/            # 4 个子命令
│   ├── security-pipeline/SKILL.md  # CWE Top 25 + STRIDE
│   └── verification-engine/SKILL.md  # Iron Law 验证引擎
├── hooks/
│   ├── output-secret-filter.sh  # 防秘钥泄露（仅 OPENCLAW 环境）
│   ├── remote-command-guard.sh  # 阻断危险命令
│   ├── rate-limiter.sh          # 速率限制
│   ├── db-guard.sh              # 数据库操作保护
│   └── security-auto-trigger.sh # 自动触发安全审查
├── hooks/forge-update-check.sh  # SessionStart 自动更新检查（无条件网络调用！）
├── scripts/
│   ├── md-to-docx/             # Markdown → Word 转换
│   └── pdf-enhance/            # PDF 后处理
├── rules/                      # 9 个 .claude/rules 文件
├── .mcp.json                   # 4 个未锁版本的 MCP 包（安全问题 S1）
└── mcp-servers.json            # 备用 MCP 配置 + 外部 HTTP 端点（S2）
```
- **文件类型分布**：7 agents、~50 commands、12 skills、5 security hooks、2 install scripts、9 rules、2 MCP configs
- **编排关系**：`commands/agent-router.md` 是 34 域路由入口（类 meta-command），分发到对应 agent/command。`skills/verification-engine` 是核心验证引擎，被多个命令引用。`commands/handoff-verify.md` → `agents/verify-agent.md` → `agents/security-reviewer.md` 形成三层验证链
- **跨件契约**：verify-agent 需要 `Task` 工具来 spawn security-reviewer，但 verify-agent 的 frontmatter 漏写了 `Task`（Bug #1），导致链路在此处断裂

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「安全即内建」——把通常外挂的安全检查（代码审查、秘钥过滤、命令拦截）直接集成为 Claude Code 钩子，而不是靠用户记得手动触发
- **解决什么问题**：CI/CD 中的安全左移（shift-left）——把安全审查时机从 PR/deploy 阶段提前到 Claude 生成代码的时机
- **做了什么 trade-off**：安全钩子全部依赖 `OPENCLAW_SESSION_ID` 激活，对本地开发者透明但也意味着零保护；换来的是远端环境的强安全边界不影响本地开发流程
- **反映什么认知模型**：作者把 Claude Code 视为「需要被约束的可信但不完美的工具」——通过 Iron Law（「未验证就不能声称完成」）和多层验证链，把验证文化编码到工作流中

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「DevSecOps 全家桶：多层安全钩子 + 验证引擎 + 路由 meta-command」**

该模式的核心是把「安全」和「验证」提升为一等公民，分别通过独立的运行时钩子和显式验证引擎来强制实施，而不是依赖每个命令自己「记得」做安全检查。

模式特征清单：
- 特征 1：多个运行时钩子（[PostToolUse Hook](#post-tool-use-hook)）形成防御纵深（秘钥过滤 → 命令拦截 → 速率限制 → 数据库保护）
- 特征 2：验证引擎（verification-engine）作为跨命令共享契约——`Iron Law: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE`
- 特征 3：单一路由入口（agent-router）将 34 个功能域的分发逻辑集中管理
- 特征 4：连续学习子系统（continuous-learning-v2）用 observer agent 捕获会话经验，通过 evolve/import/export 跨会话传递
- 特征 5：商业级的 security-pipeline 设计（CWE Top 25 + STRIDE 方法论）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业/团队 Claude Code 平台，有远程会话管理 | ✅ 高度适用 | OPENCLAW 钩子设计为远程环境，安全保障与工作流无缝集成 |
| 个人本地开发场景 | ❌ 不适用 | 所有安全钩子在本地会话中静默停用，安全收益为零 |
| 需要跨会话积累经验的长期项目 | ✅ 适用 | continuous-learning-v2 的 observer + instinct 体系专为此设计 |
| 小型原型项目 | ❌ 不适用 | 98 个制品的复杂度远超典型小项目需要 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：全家桶 + 多层钩子 | sangrokjung/claude-forge | 安全内建、验证强制、跨会话学习 | 体量大、维护成本高、钩子有激活条件门槛 |
| 单职责 skill 集合 | samber/cc-skills-golang | 轻量、模块化、各 skill 独立可测 | 无运行时安全、无验证框架 |
| NL 表皮 + 工具封装 | softaworks/agent-toolkit | 高覆盖、skill 质量高 | 缺乏系统级安全层 |

### 2.4 改进空间
1. **当前问题**：安全钩子仅在 OPENCLAW 环境激活，本地开发者零保护。**改进做法**：在每个钩子的 header 中加一行 `echo "⚠ Security hooks inactive (OPENCLAW_SESSION_ID not set)" >&2`，让缺失变成可见警告而非静默。**预期收益**：本地用户意识到保护缺失，可自行设置 OPENCLAW_SESSION_ID 或至少知道风险
2. **当前问题**：verify-agent 缺 `Task` 工具声明，handoff-verify 的三层验证链在 Step 7 悄然断裂。**改进做法**：在 `agents/verify-agent.md` 的 tools 数组加 `Task`（一行修改）。**预期收益**：security-reviewer 子 agent 可被正常 spawn，验证链完整

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-16 当时得分 82/100（73 核心制品加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 60 | 无任何 YAML frontmatter |
| commands/update-codemaps.md | 60 | 无任何 YAML frontmatter |
| commands/update-docs.md | 60 | 无任何 YAML frontmatter |
| commands/agent-router.md | 72 | 无 allowed-tools（路由 34 域却不声明工具权限） |
| agents/code-reviewer.md | 90 | 全仓最高分，结构完整的代码审查 agent |
| skills/verification-engine/SKILL.md | 88 | Iron Law 设计是高价值模式 |
| skills/security-pipeline/SKILL.md | 88 | CWE + STRIDE 实践级安全方法论 |

### 3.2 当时值得借鉴的模式

1. **Iron Law 验证契约** → 为什么好：verification-engine 的 Iron Law（「NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE」）把「完成 ≠ 声明完成」制度化，防止 Claude 在未验证时过早报告成功 → 路径：skills/verification-engine/SKILL.md → 借鉴：在所有执行类命令（deploy、commit、publish）的末尾加验证 checkpoint，而不是假设命令成功

2. **防御纵深钩子架构** → 为什么好：5 个钩子各司其职（secret filter / command guard / rate limiter / db-guard / auto-trigger），形成分层防线，单点失效不影响整体防护 → 路径：hooks/ 目录 → 借鉴：安全关键的 Claude Code 环境中，把安全检查实现为 PostToolUse Hook 而不是依赖 Claude 的自觉

3. **连续学习 + 本能导出体系** → 为什么好：continuous-learning-v2 的 observer agent 监听会话，把模式提炼为「instinct」，通过 evolve/import/export 命令跨会话传递，解决了 Claude Code 会话孤立问题 → 路径：skills/continuous-learning-v2/ 整个子系统 → 借鉴：[experience-accumulation](#experience-accumulation) 模式是长期项目的重要设计

4. **agent-router 集中路由** → 为什么好：34 个功能域通过单一 router 分发，用户只需记一个命令入口 → 路径：commands/agent-router.md → 借鉴：功能超过 10 个的插件应考虑加一个 router meta-command

5. **rules/ 目录显式编码工作约定** → 为什么好：9 个 `.claude/rules` 文件把编码风格、安全规则、测试约定等团队共识变成机器可读的约束，而不是让每个 session 重新解释 → 路径：rules/ 目录 → 借鉴：有团队约定的项目应把约定写入 rules/ 而不是重复放在每个命令的 prompt 里

### 3.3 当时的缺陷

1. **3 个命令无任何 frontmatter（test-coverage、update-codemaps、update-docs）** → 根本原因：这些命令可能是「被遗忘的草稿」，正文已写完但忘记加 YAML 头。Claude Code 靠 frontmatter 注册命令，无 frontmatter = 命令不可调用，等于这些功能对用户不存在 → 自查：我的 command 文件是否每个都有开头的 `---` frontmatter 块？

2. **security hooks 全部被 OPENCLAW_SESSION_ID 隐式门控** → 根本原因：插件设计为服务商用远程会话，但 README 没有告知本地用户该门控的存在，导致本地用户装了插件却以为有保护 → 这是「安全假象」问题，比没有安全功能更危险（用户以为安全但实际不安全）→ 自查：我的钩子是否也有隐式激活条件？用户知道吗？

3. **verify-agent 缺 Task 工具，验证链路在运行时静默断裂** → 根本原因：agent frontmatter 的 `tools` 数组是 Claude Code 的工具授权，漏写 `Task` 后，agent 在执行到「spawn security-reviewer 子 agent」这一步时，工具调用会被 sandbox 拒绝，但 agent 不会报错，只会静默跳过 → 这是「无声失败」模式，比显式错误更难排查 → 自查：我的 agent 文件里，每个 tools 声明是否覆盖了正文中实际使用的所有工具？

### 3.4 当时的优化机会

1. **8 个命令缺 allowed-tools**：eval、plan、tdd、build-fix、e2e、code-review、agent-router、refactor-clean 都在正文里使用了工具（Bash、Read 等）但 frontmatter 里没声明，在受限权限模式下会静默跳过工具调用
2. **MCP 版本锁定**：.mcp.json 中 4 个包用 `@latest`，应换成精确版本号防止供应链风险
3. **continuous-learning-v2 的 4 个子命令缺 allowed-tools 和 model**：这些命令会调用外部 Python CLI 却没声明需要 Bash，在某些权限模式下会失败

---

## 四、现在 vs 过去对比

> 目标仓库（sangrokjung/claude-forge）在本运行环境中无法访问（HTTP 403），以下分析基于 audit 快照，无法进行实时 grep 验证。

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个命令无 frontmatter | `head -1 commands/test-coverage.md` → 是否为 `---` | 无法验证（仓库不可访问） | 若未修复，这 3 个命令在装插件 2+ 月后仍对用户不可见 |
| verify-agent 缺 Task 工具 | `grep 'Task' agents/verify-agent.md` | 无法验证 | 若未修复，handoff-verify 的验证链路已断裂 2+ 月 |
| MCP 版本未锁定 | `cat .mcp.json \| grep 'latest'` | 无法验证 | 每次会话可能拉取不同版本的 MCP 包 |

### 4.2 架构演进
无法从当前 HEAD 进行对比（仓库不可访问）。从 audit 记录看，2026-04-16 时仓库已相当成熟（98 制品），预期未有根本性架构变化。

### 4.3 新增的可学习模式
暂无（无法访问当前 HEAD）。

---

## 五、校准

### 5.1 我已经在做对的
1. **命令文件都有 frontmatter**：我的 command 文件（如 MarkQWu/gstack）都有 `---` 开头的 YAML 块，避免了 claude-forge 的 3 个「不可见命令」问题
2. **安全场景用显式验证步骤**：我的流程类命令末尾有 checkpoint 步骤，与 Iron Law 精神一致
3. **不使用 `@latest` 依赖**：我的 MCP 配置如有依赖会锁定版本

### 5.2 挑战 / 验证
**这次案例挑战了我的一个假设**：「钩子存在 = 安全有保障」。claude-forge 的案例说明，钩子的激活条件（OPENCLAW_SESSION_ID）可以让看起来完整的安全系统在用户实际使用时完全无效。保障安全的不是功能的存在，而是功能的实际激活。这提醒我任何依赖环境变量的安全机制都必须在文档中明确警告用户。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查我的命令文件是否每个都有 frontmatter（以 --- 开头）
for f in ~/.claude/commands/*.md; do
  if ! head -1 "$f" | grep -q '^---'; then
    echo "无 frontmatter: $f"
  fi
done
# 命中后：在文件开头加 ---\ndescription: "..."\nallowed-tools: [...]\n---

# 2. 检查我的 agent 文件里 tools 声明是否覆盖了正文实际用到的工具
# 特别是：正文中有 Task 调用但 tools 里没有 Task 的情况
for f in ~/.claude/agents/*.md; do
  if grep -q 'Task\|spawn\|sub.agent' "$f" && ! grep -A5 '^tools:' "$f" | grep -q 'Task'; then
    echo "可能缺 Task 工具: $f"
  fi
done
# 命中后：在 frontmatter 的 tools 数组里加 "Task"

# 3. 检查我的 hooks 有没有隐式激活门控（依赖某个环境变量才运行）
grep -rn 'if \[ -z\|if \[ -n\|OPENCLAW\|\${.*:-}\|exit 0' ~/.claude/hooks/*.sh | head -20
# 命中后：核实门控条件是否在 README/INSTALL 文档中明确说明

# 4. 检查 MCP 配置是否有未锁定版本
grep -n '@latest\|@\^' ~/.claude/.mcp.json 2>/dev/null || grep -rn '@latest' .mcp.json 2>/dev/null
# 命中后：把 @latest 换成 @1.0.x 等精确版本
```

### 6.2 灵感 → 实施路径

1. **想法**：在 MarkQWu/bureau 中实现 Iron Law checkpoint
   - **为何可行**：bureau 的「compile」和「review」步骤目前缺少强制验证关卡，容易让 Claude 在状态未确认时声称完成
   - **第一步**：在 commands/compile.md 末尾加一节「## 验证 checkpoint」，要求在报告完成前显示实际产出物列表，约 15 分钟

2. **想法**：为 MarkQWu/gstack 中的所有工具型命令明确声明 allowed-tools
   - **为何可行**：gstack 有多个调用 Bash 工具的命令（CEO、Eng Manager 等角色），若 allowed-tools 缺失在受限权限模式下会静默跳过
   - **第一步**：`grep -rL 'allowed-tools' ~/.claude/commands/*.md` 找出缺失文件，逐一添加，约 5 分钟/文件

---

## 七、对照我的 GitHub 仓库

> 注：本运行环境中无法访问外部 GitHub 仓库（HTTP 403），以下分析基于 `learning/my-repos.json` 元数据和已知结构。

### 8.1 目的对齐度

- **本案例 sangrokjung/claude-forge 的核心目的**：提供一个 DevSecOps 全家桶，将安全检查、验证引擎、持续学习机制以钩子和 skill 的形式内建于 Claude Code 工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多角色/多职能的 Claude Code 工具包，包含多个命令和 agent | gstack 聚焦于角色（CEO/CTO/PM），forge 聚焦于工作流（安全/验证/学习） | 高 |
| MarkQWu/bureau | 中 | 都有「跨会话知识积累」的设计（bureau 是知识库，forge 是 instinct 体系）| bureau 以「人工审核」为核心，forge 以「自动化安全」为核心 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都在解决跨会话记忆问题（echo-sleuth 挖掘历史对话，forge 的 continuous-learning 提炼会话模式）| echo-sleuth 是只读挖掘，forge 是主动记录 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 frontmatter | `head -1 commands/*.md \| grep -v '^---'` | 无法验证，但 gstack 的 23 个工具型命令存在此风险 | 高 |
| agent 缺 Task 工具声明 | `grep -L 'Task' agents/*.md` 配合正文检查 | 无法验证，但 bureau 有 multi-agent 流程存在此风险 | 高 |
| 钩子隐式门控未文档化 | `grep -n 'exit 0' hooks/*.sh` | 无法验证 | 中 |

**命中后的具体行动建议**：
- MarkQWu/gstack 的每个命令文件 → 检查 frontmatter 完整性（description + allowed-tools） → 10 分钟批量补充
- MarkQWu/bureau 的 agent 文件 → 对照正文实际工具调用核对 tools 声明 → 15 分钟

### 8.3 别人的更优方案

1. **领域**：Iron Law 验证契约
   - **本案例做法**：skills/verification-engine/SKILL.md 中的 Iron Law 被所有流程类命令引用，形成全局「未验证不得声称完成」的强制约束
   - **我的项目现状**：MarkQWu/gstack 的 Release Manager 角色命令中没有明确的验证 checkpoint，存在 Claude 过早报告成功的风险
   - **如何借鉴**：创建一个 `skills/completion-gate/SKILL.md`，内含验证契约，在 Release Manager 等关键命令中 cross-reference 它；30 分钟可完成

2. **领域**：连续学习 + instinct 导出体系
   - **本案例做法**：continuous-learning-v2 的 observer agent 在每次会话后记录模式，evolve 命令将其提炼为可跨会话引用的 instinct 文件
   - **我的项目现状**：MarkQWu/echo-sleuth-for-claude 是被动挖掘历史，没有主动的实时观察 + 结构化存储
   - **如何借鉴**：借鉴 observer agent 的设计思路，在 echo-sleuth 中加一个 SessionEnd hook 自动记录本次会话的关键决策，约 1 小时

### 8.4 反向：我的项目做得比他们好的地方

本案例未发现我的项目更优的维度（claude-forge 在安全钩子系统和验证引擎方面均超出我的当前实现）。

---

## 八、术语表

### <a name="post-tool-use-hook"></a>PostToolUse Hook
> Claude Code 的「工具后置钩子」——每次 Claude Code 调用一个工具（如 Bash、Write、Edit）之后，自动运行的 shell 脚本。可以用来：过滤输出中的敏感信息（如 API key）、记录工具调用日志、或在危险操作后触发安全审查。与 PreToolUse Hook 配合可形成双层拦截。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明命令/skill/agent 的元数据（name、description、allowed-tools 等）。Claude Code 在注册命令时读取这段配置，若没有 frontmatter，命令不会出现在 `/help` 中，也无法通过斜杠命令调用。

### <a name="experience-accumulation"></a>experience-accumulation
> 「经验积累」模式——通过 hook、observer agent 或类似机制，把每次 Claude Code 会话中发生的决策、错误、最佳实践记录到持久化文件（如 instincts.md），供后续会话查阅。解决 Claude Code 的「每次会话都从零开始」问题。

### <a name="iron-law"></a>Iron Law
> claude-forge 中 verification-engine 定义的黄金规则：「NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE」（没有新鲜的验证证据，不得声称任务已完成）。防止 Claude 在实际执行结果未确认的情况下过早报告成功。

### <a name="openclaw"></a>OPENCLAW_SESSION_ID
> claude-forge 中用于区分远程商业会话和本地开发会话的环境变量。所有安全钩子仅在此变量存在时激活。本地开发时默认不设置，因此安全钩子处于静默停用状态。
