# vectorize-io/hindsight — 学习案例

**仓库**：https://github.com/vectorize-io/hindsight
**Stars**：10609 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-27（基于当前 HEAD）
**主题标签**：curl-pipe-bash-risk, security-gate, vague-quantifier, cross-reference, experience-accumulation

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Hindsight 是一个专为 Claude Code 设计的**会话记忆与知识保留服务**，核心功能是在每次 Claude Code 会话结束时自动捕获并存储会话 [transcript](#transcript)（完整对话记录），以便后续会话能够"记住"过去的决策、错误与解决方案。凭借 10609 颗 Stars，它是 Claude Code 插件生态中最受关注的记忆增强插件之一，验证了"跨会话知识积累"这一核心需求的真实性。

该项目支持三种部署模式，满足不同用户的隐私与基础设施需求：
- **Cloud 模式**：由 Vectorize 托管的云端服务，开箱即用，数据发送至 Hindsight 官方端点
- **Local 模式**：在用户本地机器上运行，数据不离开本地
- **Self-hosted 模式**：用户自行部署后端，完全控制数据流向

这三种模式对应三个独立的 [SKILL.md](#skill-md) 文件，分别为 `skills/hindsight-cloud/SKILL.md`、`skills/hindsight-local/SKILL.md`、`skills/hindsight-self-hosted/SKILL.md`。

### 1.2 架构剖析

Hindsight 的架构可以分为三层：

**第一层：技能层（Skills Layer）**
- 四个核心 SKILL.md 文件：`hindsight-architect`（编排总控）、`hindsight-docs`（文档引用）、加上上述三种部署模式的 skill
- `hindsight-docs/SKILL.md` 维护一个 `references/` 子目录，包含 `best-practices`、`developer/api/`、`sdks/`、`cookbook/` 等结构化参考资料
- `.claude/skills/code-review/SKILL.md` 提供代码审查能力，被 `CLAUDE.md` 第174行显式引用

**第二层：集成层（Integration Layer）**
- **claude-code 集成**：`hindsight-integrations/claude-code/`，使用 `${CLAUDE_PLUGIN_ROOT}` 运行时变量，包含干净的 `hooks/hooks.json`（评分95分）和 Python [daemon](#daemon) 脚本
- **codex 集成**：`hindsight-integrations/codex/`，使用 `__SCRIPTS_DIR__` 安装时模板占位符，两套集成之间存在路径约定不一致的问题

**第三层：[Hook](#hook) 自动化层**
- 通过 Claude Code 的 Stop [hook](#hook) 事件，在每次会话结束时自动触发 `retain.py` 脚本，将完整会话 [transcript](#transcript) POST 到配置的 Hindsight API 端点

两套集成（claude-code + codex）并行存在，体现了对不同 AI 工具链的支持意图，但也引入了维护上的分叉复杂性。

### 1.3 设计思路 / 方法论

Hindsight 的核心设计哲学是**"会话记忆外包给基础设施"**：与其期望 Claude 在当前上下文窗口内保留历史，不如在每次会话结束时将知识固化到持久存储，在下次会话开始时通过 skill 检索并注入。

这个设计的核心价值在于：
1. **突破上下文窗口限制**：长期积累的决策历史不受单次会话长度约束
2. **跨项目知识迁移**：同一个用户在不同项目中积累的模式可以被复用
3. **可检索的错误记录**：过去犯过的错误以结构化形式存储，下次遇到类似问题时可被检索

**隐私 Tradeoff**：Cloud 模式下，完整会话 [transcript](#transcript)（包含可能的敏感代码、凭证片段、商业逻辑）会被 POST 到第三方服务器。`hindsight-integrations/claude-code/scripts/retain.py` 第147行明确将完整 transcript 发送到配置的 API 端点。这是**设计意图而非漏洞**，但安全审计将其标记为 MEDIUM 风险——因为用户可能未充分了解传输内容的范围。Self-hosted 和 Local 模式正是为解决这一隐私顾虑而存在的。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**会话记忆侧车模式（Session Memory Sidecar Pattern）**

Hindsight 实现了一种"记忆侧车"（[sidecar](#sidecar)）架构：主业务是 Claude Code 会话，记忆捕获是附加在主业务旁边的独立进程，通过 [hook](#hook) 机制在生命周期事件（SessionStop）触发时激活，不干扰主会话流程。

这一模式的关键特征：
- **松耦合**：通过 [hook](#hook) 事件解耦，移除 Hindsight 不影响 Claude Code 核心功能
- **透明捕获**：用户无需手动操作，会话结束时自动记录
- **多后端适配**：同一 hook 触发机制，通过配置切换 cloud/local/self-hosted 三种存储后端

**多运行时技能三分法（Multi-Runtime Skill Tripartition）**

针对同一功能（hindsight 交互），按部署模式拆分为三个独立 SKILL.md，而非在单一 skill 中用条件分支区分。这种拆分让每个 skill 保持单一职责，避免了"if cloud ... elif local ... elif self-hosted ..."的条件逻辑污染。

### 2.2 适用场景

- 任何需要**跨会话知识积累**的 AI 辅助工具
- 需要支持多种部署形态（SaaS + 私有化）的 Claude Code 插件
- 希望通过 Stop [hook](#hook) 在会话结束时触发后台任务（报告生成、数据同步、清理）的场景

### 2.3 与其他架构对比

| 维度 | Hindsight（侧车模式） | 内置记忆（如 mem0 集成） | 纯文件记忆 |
|------|----------------------|------------------------|-----------|
| 耦合度 | 低（hook 事件驱动） | 高（深度集成 LLM 调用） | 低 |
| 延迟影响 | 几乎零（异步 POST） | 高（LLM 调用） | 零 |
| 隐私风险 | Cloud 模式中等 | 依赖第三方 | 无（本地文件） |
| 检索质量 | 向量检索，高 | 依赖实现 | 关键词检索，低 |
| 维护成本 | 中（两套集成） | 高 | 低 |

### 2.4 改进空间

1. **统一路径约定**：claude-code 和 codex 两套集成使用不同的路径变量机制，应抽象出统一的安装时/运行时解析层
2. **渐进式同意（Progressive Consent）**：Cloud 模式安装时应向用户明确展示"以下内容将被上传"的具体说明，而非仅在文档深处提及
3. **完整性验证**：`curl | bash` 安装方式可引入基于校验和的完整性验证（如 `curl ... | sha256sum -c` 先验证再执行）作为缓解手段

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**综合评分：94/100**（Security：**BLOCKED**）

| 文件 | 类型 | 分数 | 主要问题 |
|------|------|------|---------|
| hindsight-integrations/codex/hooks/hooks.json | Hook Config | 88 | `__SCRIPTS_DIR__` 模板占位符未替换；缺 SessionEnd hook；Stop 缺 async:true |
| .claude/skills/code-review/SKILL.md | Skill | 90 | 5个模糊量词：non-trivial×3, significantly changed, meaningful |
| CLAUDE.md | Project Context | 90 | 结构良好，少量模糊措辞 |
| skills/hindsight-architect/SKILL.md | Skill | 94 | "appropriate"×3 |
| skills/hindsight-docs/SKILL.md | Skill | 95 | 引用 references/ 目录；干净 |
| hindsight-integrations/claude-code/.claude-plugin/plugin.json | Plugin Manifest | 95 | 干净 |
| hindsight-integrations/claude-code/hooks/hooks.json | Hook Config | 95 | 干净 |
| skills/hindsight-cloud/SKILL.md | Skill | 98 | non-trivial×1 |
| skills/hindsight-local/SKILL.md | Skill | 98 | non-trivial×1 |
| skills/hindsight-self-hosted/SKILL.md | Skill | 98 | non-trivial×1 |

Security BLOCKED 的存在使得"NL 质量94分"这一高分失去了贡献资格——任何 Critical 安全发现都会触发硬性拦截，不论其他质量指标多高。

### 3.2 当时值得借鉴的模式

**亮点一：两套集成的良好抽象对比**

Hindsight 同时维护 claude-code 和 codex 两套集成，这种并行集成本身体现了对不同工具链用户群的覆盖意识。更值得关注的是，claude-code 集成的 `hooks.json`（评分95）和 codex 集成的 `hooks.json`（评分88）之间的质量差异，恰好揭示了"安装时模板变量"vs"运行时环境变量"两种路径解析策略的优劣——前者（`${CLAUDE_PLUGIN_ROOT}`）在 hook 实际执行时已由运行时解析完毕，后者（`__SCRIPTS_DIR__`）依赖安装步骤的字符串替换，一旦安装步骤跳过或失败，路径静默失效。

**亮点二：hindsight-docs/SKILL.md 的 references/ 目录完整性管理**

`skills/hindsight-docs/SKILL.md` 中引用的 `references/` 子目录（含 `best-practices`、`developer/api/`、`sdks/`、`cookbook/`）**全部实际存在于仓库中**。这是一个高质量的交叉引用（[cross-reference](#cross-reference)）实践：SKILL.md 不仅声明了参考资料的存在，实际文件系统也与之保持同步。对比常见的"引用一个不存在的文件路径"反模式，这种一致性管理显著提升了 skill 的可靠性。

**亮点三：CLAUDE.md 显式引用 code-review skill 的做法**

`CLAUDE.md` 第174行显式引用了 `.claude/skills/code-review/SKILL.md`，形成了"项目配置 → 技能文件"的明确指针。这种做法让读者（人类或 Claude）能够快速定位具体能力定义的位置，而不是依赖约定俗成的文件路径猜测。

### 3.3 当时的缺陷

**缺陷一：[curl-pipe-bash](#curl-pipe-bash) Critical——最严重的 NL 编程反模式**

`skills/hindsight-cloud/SKILL.md` 第24行和 `skills/hindsight-self-hosted/SKILL.md` 第24行均包含：

```bash
curl -fsSL https://hindsight.vectorize.io/get-cli | bash
```

这是 NL 编程中最严重的安全反模式之一，其危险性在 Claude Code 上下文中被**倍数放大**：

- **普通 curl|bash 场景**：人类用户主动选择执行，有意识地承担风险
- **SKILL.md 中的 curl|bash 场景**：SKILL.md 是对 Claude 的指令，当 Claude 遵循 skill 建议时，它会**在用户机器上替用户执行**这条命令，而用户可能未注意到这条指令的存在

这意味着：任何能够污染 `https://hindsight.vectorize.io/get-cli` 端点的攻击者（DNS 劫持、BGP 路由攻击、Hindsight 服务器被入侵），都能在用户不知情的情况下，通过 Claude 在用户机器上执行任意代码，且无任何完整性验证（无校验和、无签名验证）。

NLPM 安全规则 SEC-curl-pipe-sh 将此标记为 **Critical**，而非 High——正是因为 SKILL.md 的执行者是 Claude，而 Claude 不会像人类用户那样质疑"这条命令是否安全"。

**缺陷二：`__SCRIPTS_DIR__` 模板占位符——两套集成的维护噩梦**

`hindsight-integrations/codex/hooks/hooks.json` 使用 `__SCRIPTS_DIR__` 作为路径前缀（双下划线包围，典型的安装时模板占位符格式），而 `hindsight-integrations/claude-code/hooks/hooks.json` 使用 `${CLAUDE_PLUGIN_ROOT}`（运行时环境变量）。

两种机制的根本差异：
- `${CLAUDE_PLUGIN_ROOT}`：运行时由 shell 解析，路径错误会立刻产生明确报错
- `__SCRIPTS_DIR__`：依赖安装脚本执行 `sed` 替换，若替换未发生，[hook](#hook) 文件包含字面量 `__SCRIPTS_DIR__`，Claude Code 尝试执行时路径不存在，**静默失效**，无任何错误提示

这构成一个直接 Bug：codex hooks 在未经正确安装的情况下指向不存在的路径，所有 codex [hook](#hook) 均不会触发，且不产生可见错误。更深层的问题是：同一个项目维护两套使用不同约定的集成，意味着每次路径逻辑变更时需要同步修改两处，维护成本随时间线性增长。

**缺陷三：[模糊量词](#vague-quantifier)"non-trivial"×5——可操作性的隐性损失**

三个 skill 文件（`hindsight-cloud`、`hindsight-local`、`hindsight-self-hosted`）各出现1次 `non-trivial`，`code-review/SKILL.md` 出现3次。"non-trivial"是一个典型的主观量词——它告诉 Claude"某件事是重要的"，但没有告诉 Claude"重要到什么程度才触发相应行为"。

具体替代示例（以 code-review/SKILL.md 为例）：

| 原文 | 建议替换 |
|------|---------|
| `non-trivial changes` | `changes touching ≥3 files or modifying public API surface` |
| `significantly changed` | `changed by more than 20 lines or renamed` |
| `meaningful refactor` | `refactor that moves logic across module boundaries` |

具体的可量化描述让 Claude 能够一致地判断边界条件，而"non-trivial"在不同的上下文中可能被理解为截然不同的阈值。

### 3.4 当时的优化机会

1. **curl|bash 替代方案**：提供预编译二进制下载 + 校验和验证（如 `curl ... -o hindsight && echo "sha256sum" | sha256sum -c && chmod +x hindsight`），或提供包管理器安装路径（`brew install hindsight`、`pip install hindsight-cli`）
2. **统一路径约定**：将两套集成的路径解析统一到运行时环境变量机制
3. **模糊量词替换**：系统性替换所有 skill 文件中的 `non-trivial`、`appropriate`、`meaningful`、`significantly`
4. **补全 codex hooks**：添加 `SessionEnd` [hook](#hook)，在 Stop hook 中添加 `async: true`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

由于 git clone 和 GitHub MCP API 均因代理限制返回 403，无法直接访问当前仓库 HEAD 状态。基于分析推断：

**curl|bash Critical（probable-persists——极可能持续）**

`curl -fsSL https://hindsight.vectorize.io/get-cli | bash` 是 Hindsight CLI 的**核心安装机制**，而非边缘案例。要移除这一模式，需要：
1. 建立二进制分发渠道（GitHub Releases + 校验和、Homebrew tap、PyPI 等）
2. 同步更新三个 skill 文件的安装指令
3. 维护多个包管理器渠道的持续发布流程

这是一项工程量显著的迁移工作。在没有外部压力（如重大安全事件、主要平台拒绝列出）的情况下，维护者大概率会维持现状。BLOCKED 状态导致 NLPM 未能贡献任何修复建议，该 Critical 问题也因此未经我们的反馈渠道触达维护者。

**`__SCRIPTS_DIR__` Bug（cannot-verify——无法验证）**

无法访问当前仓库代码，无法确认此 Bug 是否已修复。

**shell=True HIGH（cannot-verify——无法验证）**

`hindsight-integrations/claude-code/scripts/lib/daemon.py` 第297行和 `hindsight-integrations/codex/scripts/lib/daemon.py` 第269行的 [SubprocessPopen](#subprocess-popen)（`subprocess.Popen(..., shell=True)`）状态无法远程验证。

### 4.2 架构演进

暂无——因无法访问当前仓库 HEAD，无法评估架构层面的变化。10609 Stars 的高热度表明该项目仍处于活跃维护状态，但具体演进方向不可知。

### 4.3 新增的可学习模式

暂无——同上，无法访问当前代码。基于高 Star 数量推断，项目可能已扩展更多集成（如 Cursor、Windsurf 等 AI 编辑器），但无法确认。

---

## 五、校准

### 5.1 我已经在做对的

- **避免 curl|bash 模式**：在自己的 skill 和 hook 中，如果需要调用安装脚本，应优先使用本地相对路径或运行时环境变量，而非远程 curl 下载
- **使用运行时变量而非安装时占位符**：`${VARIABLE}` 形式的路径在运行时解析，出错时有明确报错，优于 `__PLACEHOLDER__` 形式的安装时字符串替换
- **交叉引用的文件完整性**：在 SKILL.md 或 CLAUDE.md 中引用的文件路径，确保对应文件实际存在

### 5.2 挑战 / 验证

**这个案例挑战了一个常见假设：高 NL 质量分 = 可贡献、可信赖**

Hindsight 的 NL 质量评分为 **94/100**，在所有被审计的仓库中属于高分区间。但这一高分并未阻止它被 Security **BLOCKED**。这意味着：

- NL 质量评分（可读性、具体性、交叉引用一致性）和安全性（执行面风险）是**两个正交的维度**，高分 NL 质量完全可以与 Critical 安全风险共存
- 从作者的角度：一个写得很好、结构清晰的 SKILL.md，如果其中包含 curl|bash 指令，其"写得好"反而使这条危险指令更容易被 Claude 遵循执行
- 从审计的角度：安全扫描必须独立于 NL 质量评分运行，不能用高质量分来推定安全性

**需要验证的假设**：在 `__SCRIPTS_DIR__` 模板占位符场景下，Claude Code 是否会在 hook 执行失败时产生任何可见的用户提示？还是真的完全静默失效？这影响到对该 Bug 严重程度的判断。

---

## 六、行动

### 6.1 自查动作

**自查一：检查自己的 skill 和 hook 文件中有无 curl|bash 模式**
```bash
grep -rn 'curl.*|.*bash\|curl.*|.*sh' ~/.claude/
```
这条命令扫描 Claude Code 全局配置目录下所有文件，查找 `curl ... | bash` 或 `curl ... | sh` 形式的管道执行模式。任何匹配都应评估是否可替换为更安全的安装方式。

**自查二：检查 Python hook 脚本中有无 shell=True**
```bash
grep -rn 'shell=True' .
```
在项目目录下运行，查找所有 Python [SubprocessPopen](#subprocess-popen) 调用中使用 `shell=True` 的位置。`shell=True` 将命令字符串传递给 shell 解析，若命令包含用户输入，存在命令注入风险；即使有 `shlex.join()` 缓解，仍被安全扫描标记为 HIGH。

**自查三：检查 Markdown 文件中的模糊量词**
```bash
grep -rn 'non-trivial\|appropriate\|meaningful\|significantly' . --include="*.md"
```
在项目目录下扫描所有 Markdown 文件中的模糊量词。每个匹配项都应检查是否可以替换为可量化的具体描述。

### 6.2 灵感 → 实施路径

**灵感：从 Hindsight 的三种部署模式学习"同一 skill 支持多运行时的设计"**

Hindsight 将 cloud/local/self-hosted 三种部署形态拆分为三个独立 SKILL.md，而非在单一 skill 中用条件分支处理。这一设计思路可迁移到任何需要支持多种运行环境的 Claude Code 插件中。

实施路径：
1. **识别运行时维度**：列出 skill 所服务的不同运行环境（云端 vs 本地、Docker vs 裸金属、PostgreSQL vs SQLite 等）
2. **评估拆分收益**：若各环境的操作指令差异超过50%，则拆分为独立 SKILL.md；若差异在20%以内，则用配置变量区分
3. **建立命名约定**：`skill-<功能>-<运行时>.md`（如 `hindsight-cloud`、`hindsight-local`）使文件名自文档化
4. **维护引用一致性**：在 CLAUDE.md 或 `hindsight-architect` SKILL.md 中明确列出所有变体及其适用场景，保持 [cross-reference](#cross-reference) 完整性

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

基于用户仓库元数据分析，以下两个项目与 Hindsight 具有**高度相似的目标定位**：

**MarkQWu/echo-sleuth-for-claude（高相似度）**
- 相似点：同为"从 Claude 会话中提取并保留知识"的记忆增强插件，目标用户群体完全重叠
- 差异点：echo-sleuth 侧重从**历史会话**中挖掘决策/错误/模式/智慧（后处理型），而 Hindsight 侧重**实时捕获**每次会话结束时的完整 transcript（实时型）
- 竞争关系：两者解决同一问题的不同子路径，可以互补（echo-sleuth 处理 Hindsight 捕获的历史数据）

**MarkQWu/bureau（高相似度）**
- 相似点：AI 会话 → 知识库的完整 capture→compile→review→query 流程，与 Hindsight 的 capture→store→retrieve 流程高度对应
- 差异点：bureau 更系统化，建立了完整的知识管理工作流；Hindsight 更轻量，专注于捕获和检索
- 学习关系：bureau 的 compile 和 review 步骤是 Hindsight 所缺乏的"知识提炼"环节，两者的组合可能产生更高质量的知识积累

**MarkQWu/gstack 和 MarkQWu/graphify（低相似度）**
- gstack 是 Claude Code 工具栈管理，graphify 是代码知识图谱，与 Hindsight 的会话记忆定位差异明显

### 8.2 在我的项目里复现的同类问题

由于用户仓库 git clone 全部失败（403，代理限制），无法直接检查源代码。建议在本地运行以下自查命令：

```bash
# 检查自己项目中有无 curl|bash 模式（在各项目根目录下运行）
grep -rn 'curl.*|.*bash\|curl.*|.*sh' ~/path/to/echo-sleuth-for-claude/ --include="*.md"
grep -rn 'curl.*|.*bash\|curl.*|.*sh' ~/path/to/bureau/ --include="*.md"

# 检查 SKILL.md 和 CLAUDE.md 中的模糊量词
grep -rn 'non-trivial\|appropriate\|meaningful\|significantly' ~/path/to/echo-sleuth-for-claude/ --include="*.md"
grep -rn 'non-trivial\|appropriate\|meaningful\|significantly' ~/path/to/bureau/ --include="*.md"
```

需要特别关注的风险点：
- 如果 echo-sleuth 或 bureau 的 SKILL.md 中包含安装命令（如安装某个 Python 包的脚本），是否存在 curl|bash 模式？
- 如果有 Python hook 脚本处理会话数据，是否使用了 `shell=True`？

### 8.3 别人的更优方案（值得借鉴的）

**Hindsight 的 references/ 目录自动同步机制优于常见做法**

`skills/hindsight-docs/SKILL.md` 引用的 `references/` 子目录与实际文件系统完全一致，所有被引用的路径均存在。这种**引用-实现一致性**在许多项目中往往被忽视（README 里引用的文档路径已过时、SKILL.md 引用的脚本已被移除等）。

对于 echo-sleuth 和 bureau 这类包含 SKILL.md 的项目，值得借鉴的做法是：
1. 在 CI（如 GitHub Actions）中添加"引用完整性检查"步骤，验证 SKILL.md 和 CLAUDE.md 中引用的所有文件路径实际存在
2. 建立 `references/` 目录统一管理外部文档引用，而非在各 skill 文件中分散引用

**Hindsight 的 claude-code 集成双层验证机制（plugin.json + hooks.json 分离）**

将插件声明（`plugin.json`，评分95分）和 hook 配置（`hooks.json`，评分95分）分离到两个独立文件，职责清晰。若 echo-sleuth 或 bureau 将插件元数据和 hook 配置混合在单一文件中，可以参考这种分离模式。

### 8.4 反向：我的项目做得比他们好的地方

暂无——由于无法访问用户仓库源代码（代理限制导致 git clone 失败），无法进行直接的代码层面对比。一旦代理限制解除，建议执行以下对比：
- 检查 echo-sleuth 和 bureau 的 hook 配置是否比 Hindsight codex 集成更干净（无 `__PLACEHOLDER__` 模式）
- 检查这两个项目的 SKILL.md 是否避免了 curl|bash 模式

如有发现，可在此处补充。

---

## 八、术语表

<a name="transcript"></a>
**transcript**：Claude Code 会话的完整文字记录，包含用户输入、Claude 输出、工具调用及结果。Hindsight 的核心捕获对象。Cloud 模式下 transcript 会被 POST 到第三方服务器，存在隐私风险。

<a name="skill-md"></a>
**SKILL.md**：Claude Code 技能文件，以 Markdown 格式编写的对 Claude 的指令集合。Claude 在执行相关任务时会读取并遵循其中的指导，包括工作流、命令示例和注意事项。技能文件中的命令示例会被 Claude 在用户机器上执行。

<a name="hook"></a>
**hook（钩子）**：Claude Code 的生命周期回调机制，在特定事件（SessionStart、SessionEnd、Stop、PostToolUse 等）发生时自动触发预配置的命令或脚本。Hindsight 通过 Stop hook 实现会话结束时的自动记录。

<a name="sidecar"></a>
**sidecar（侧车模式）**：软件架构模式，将辅助功能（日志、监控、记忆捕获）作为独立进程附加在主进程旁边运行，通过事件或接口交互，不修改主进程代码。Hindsight 是 Claude Code 的记忆侧车。

<a name="daemon"></a>
**daemon（守护进程）**：在后台持续运行的进程，无需用户交互。Hindsight 的 `daemon.py` 在后台监听并处理会话数据的上传任务。

<a name="curl-pipe-bash"></a>
**curl-pipe-bash（curl管道bash反模式）**：`curl <URL> | bash` 或 `curl <URL> | sh` 形式的命令，从远程 URL 下载脚本并立即执行，无任何内容验证（无校验和、无签名）。危险性在于：任何能够控制该 URL 响应内容的攻击者（DNS 劫持、服务器入侵、中间人攻击）都能执行任意代码。在 SKILL.md 中出现时危险性倍增——Claude 会以"遵循 skill 指令"的方式在用户机器上自动执行，绕过了用户的手动确认。NLPM 将其标记为 Critical 安全风险（SEC-curl-pipe-sh）。

<a name="cross-reference"></a>
**cross-reference（交叉引用）**：在一个文件中引用另一个文件的路径或内容，建立文档间的关联。NLPM 检查交叉引用的完整性——被引用的文件必须实际存在，被引用的 skill 名称必须与实际文件名一致。

<a name="vague-quantifier"></a>
**vague-quantifier（模糊量词）**：在 SKILL.md 或其他 NL 编程文件中使用的主观性、不可量化的修饰词，如 `non-trivial`、`appropriate`、`meaningful`、`significantly`、`reasonable` 等。模糊量词导致 Claude 无法一致地判断边界条件，可能在不同上下文中产生不同行为。NLPM 的 NL 质量评分会为每个模糊量词扣分。

<a name="subprocess-popen"></a>
**SubprocessPopen（子进程调用）**：Python 标准库 `subprocess.Popen()` 函数，用于在 Python 脚本中启动外部进程。当 `shell=True` 参数被设置时，命令字符串通过系统 shell（如 `/bin/sh`）解析执行，若命令包含来自外部的输入，存在命令注入（Shell Injection）风险。NLPM 安全扫描将 `shell=True` 标记为 HIGH 风险，即使有 `shlex.join()` 缓解。

<a name="postinstall"></a>
**postinstall（安装后脚本）**：在包管理器（如 npm）执行 `install` 命令后自动运行的脚本，在 `package.json` 的 `scripts.prepare` 或 `scripts.postinstall` 字段配置。Hindsight 的 `package.json` 通过 `prepare` 字段在 `npm install` 时自动执行 `./scripts/setup-hooks.sh`，被安全扫描标记为 MEDIUM 风险（用户可能未意识到 install 触发了额外的系统操作）。

<a name="session-memory"></a>
**session memory（会话记忆）**：跨越单次 Claude Code 会话边界的持久化知识存储机制。由于 Claude Code 每次会话的上下文窗口相互独立，会话记忆插件（如 Hindsight、echo-sleuth、bureau）通过外部存储将知识从一次会话传递到另一次会话，模拟"长期记忆"效果。

<a name="retain"></a>
**retain（保留/捕获）**：Hindsight 中将会话 [transcript](#transcript) 持久化到后端存储的操作。对应 `retain.py` 脚本和 `hindsight retain` CLI 命令。Stop [hook](#hook) 触发后，retain 操作将当前会话数据 POST 到配置的 API 端点。
