# centminmod/my-claude-code-setup — 学习案例

**仓库**：https://github.com/centminmod/my-claude-code-setup
**Stars**：2210 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-31（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `model-pinning`, `experience-accumulation`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

centminmod/my-claude-code-setup 是一个**个人 Claude Code 设置共享仓库**，作者是知名 CentOS 系统管理员和 nginx 性能优化专家（centminmod.com 维护者）。2210 stars 表明个人 Claude Code 配置共享是有市场的。仓库包含命令、agent、skill、钩子脚本、MCP 配置和多套 CLAUDE.md 模板，全部面向个人开发者直接 clone 使用。

关键事实：
- 个人配置共享仓库，无 npm 分发机制（直接 clone 或 copy）
- 6 个 agent、7 个 skill、多个 command（按子目录分类）
- 包含两个 HIGH 风险的 shell 注入 agent（codex-cli/zai-cli）
- 新增 rules/、hooks/、mcp/ 目录（审查后显著扩展）
- 提供 8 套 CLAUDE.md 模板（通用版、Cloudflare 版、Convex 版等）

### 1.2 架构剖析

```
my-claude-code-setup/
├── CLAUDE.md                           ← 主配置（无 frontmatter，审查时评 40 分）
├── CLAUDE-template-*.md                ← 多套 CLAUDE.md 模板（云平台专版）
├── CLAUDE-cloudflare.md
├── CLAUDE-convex.md
├── .claude/
│   ├── settings.json                   ← Claude Code 全局设置（新增）
│   ├── settings.local.json             ← 本地设置（新增）
│   ├── agents/                         ← 6 个 agent
│   │   ├── codex-cli.md                ← HIGH: shell 注入风险（仍存在）
│   │   ├── zai-cli.md                  ← HIGH: shell 注入风险（仍存在）
│   │   ├── memory-bank-synchronizer.md ← 缺 model 声明（仍存在）
│   │   ├── ux-design-expert.md         ← 缺 model 声明（仍存在）
│   │   ├── code-searcher.md            ← model: sonnet（已声明）
│   │   └── get-current-datetime.md     ← 缺 model 声明（仍存在）
│   ├── commands/                       ← 12+ 个 command（分子目录，全部无 frontmatter）
│   │   ├── anthropic/
│   │   ├── cleanup/
│   │   ├── documentation/
│   │   ├── promptengineering/
│   │   ├── refactor/
│   │   ├── security/                   ← 含 prompt injection 测试固件
│   │   └── ccusage/
│   ├── skills/                         ← 7 个 skill（session-metrics、ai-image-creator 等）
│   ├── hooks/                          ← unified_notifier.py（新增）
│   ├── rules/                          ← core-rules.md（新增）
│   └── mcp/                            ← MCP 配置（新增）
```

**文件类型分布**：6 个 agent / 7 个 SKILL.md / 12+ 个 command / 8 套 CLAUDE.md 模板 / 1 个 hook 脚本

**编排关系**：`consult-codex/SKILL.md` 调用 `codex-cli` agent；`consult-zai/SKILL.md` 调用 `zai-cli` agent；两对形成 skill → agent 的委托模式。其他 skill 和 command 独立平列。

**跨件契约**：`consult-codex` → `codex-cli` 和 `consult-zai` → `zai-cli` 引用完整；CLAUDE.md 引用 `claude-docs-consultant` skill 正确。settings.json 管理全局权限，但内容对外共享（潜在风险）。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**实用主义**。仓库是个人工作环境的完整快照，不追求"正确的 NL 工程"，追求"对我的工作有用"。每个 agent 和 command 解决了作者实际遇到的问题（如需要用 codex-cli 代理 AI 请求、需要会话统计）。
- **解决什么问题**：CI/CD 工程师（centminmod 主业）需要在 Claude Code 中调用 codex-cli、zai-cli 等外部 AI 工具，并追踪会话使用量。
- **做了什么 trade-off**：为了方便直接调用 shell 命令，选择了将用户 prompt 直接代入 shell 命令字符串——换来了调用简单，但引入了 shell 注入风险。
- **反映什么认知模型**：作者把 Claude Code 视为**编排层**，各个外部 CLI 工具（codex、zai）是真正的执行层。这和微服务架构中"网关代理后端服务"的思想一致，只是安全边界处理不足。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「CLI 代理 agent + skill 委托」

核心思路：为外部 CLI 工具（codex-cli、zai-cli）各写一个 agent 作为代理层，再用对应 skill 作为"召唤接口"。用户调用 skill → skill 委托 agent → agent 执行 CLI 命令。

特征清单：
- 特征 1：agent 是 CLI 工具的 NL 代理，把自然语言 prompt 转为 CLI 调用
- 特征 2：skill 作为门面，隐藏 agent 的 CLI 细节
- 特征 3：多套 CLAUDE.md 模板覆盖不同云平台场景（Cloudflare/Convex/通用）
- 特征 4：session-metrics skill 实现会话使用量追踪（经验积累模式）
- 特征 5：security/ 命令目录含 prompt injection 测试固件，体现安全意识

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人开发环境快速配置 | ✅ 高度适用 | clone 即用，无需构建 |
| 需要在 Claude Code 中调用外部 AI CLI 的场景 | ✅ 适用（修复注入后） | CLI 代理 agent 是合理抽象 |
| 团队共享配置（多人使用） | ❌ 不适用 | shell 注入风险在多用户场景危害更大 |
| 生产环境 AI 工作流 | ❌ 不适用 | 命令缺 frontmatter、agent 缺 model 声明、安全未修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| CLI 代理 agent（本仓库） | centminmod/my-claude-code-setup | 零配置集成外部 CLI；直接调用，无网络请求 | shell 注入风险；依赖本地安装的 CLI |
| 备选 A：MCP 服务器集成 | czlonkowski/n8n-mcp | 安全隔离；双向通信；无注入风险 | 需要搭建 MCP 服务器 |
| 备选 B：HTTP API 调用 | 多数 AI 服务集成 | 标准接口；安全可控 | 需要网络；有延迟 |

### 2.4 改进空间

1. **当前问题**：codex-cli/zai-cli agent 直接将用户 prompt 代入 shell 命令，存在注入风险。**改进做法**：使用 `shlex.quote()` (Python) 或等效 shell 转义包裹用户输入，或改为 MCP 集成。**预期收益**：消除 HIGH 级安全漏洞。

2. **当前问题**：所有 command 文件缺 frontmatter，不能被 Claude Code 注册为合法 slash command。**改进做法**：为每个 command 加 4 行 frontmatter（`---\nname: ...\ndescription: ...\n---`）。**预期收益**：command 可被发现、可注册。

3. **当前问题**：memory-bank-synchronizer 和 ux-design-expert agent 无 model 声明，会继承调用者模型。**改进做法**：`memory-bank-synchronizer` 声明 `model: sonnet`（需要上下文记忆），`ux-design-expert` 声明 `model: sonnet`（需要创意能力），`get-current-datetime` 声明 `model: haiku`（单一机械任务）。**预期收益**：成本和能力精确匹配，避免用昂贵模型做简单任务。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 当时得分 **49/100**（未通过 70 分阈值），安全状态 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/anthropic/update-memory-bank.md` | 15 | 单行文件，无 frontmatter，无结构 |
| `commands/refactor/refactor-code.md` | 20 | 无 frontmatter，10+ 个模糊量词 |
| 20 个 command 文件 | 15-39 | 全部无 frontmatter |
| `agents/codex-cli.md` | 73 | 无示例，无输出格式 |
| `agents/zai-cli.md` | 73 | 同上 |
| `agents/get-current-datetime.md` | 74 | 无 model，两个未使用工具 |
| `skills/session-metrics/SKILL.md` | 96 | 同仓库最优质文件 |

### 3.2 当时值得借鉴的模式

1. **session-metrics skill（96分）**：仓库中质量最高的文件，有清晰的 frontmatter、触发词、输出格式和示例。是"精品孤岛"模式——仓库整体质量低，但某个 skill 质量远超平均。→ 如何借鉴：即使在快速迭代的个人仓库中，至少把核心 skill 写到高标准。

2. **consult-codex → codex-cli 委托链**：skill 作为门面调用 agent，用户只看到 skill 不看到 agent 实现。→ 如何借鉴：`echo-sleuth-for-claude` 的 recall agent 可以加一层 skill 门面，让用户不直接调用 agent。

3. **多套 CLAUDE.md 模板**：为不同云平台（Cloudflare/Convex/通用）提供定制版 CLAUDE.md。→ 如何借鉴：`claude-for-legal` 已有类似设计（各法律领域有独立 CLAUDE.md），这是正确方向。

4. **security/ 命令中的 prompt injection 测试固件**：测试用的 base64 payload、不可见字符、CSS 隐藏等注入样本作为测试数据存在。→ 价值：表明作者有安全意识，但测试固件放在 commands/ 路径会被意外注册为 slash command。

### 3.3 当时的缺陷

1. **20个 command 全部无 frontmatter**：Claude Code 的 slash command 系统要求 `name` 和 `description`，无此字段则命令无法注册。→ 根本原因：作者把 command 文件当作"笔记"而非"注册配置"来写，忽略了 frontmatter 是机器协议而非可选注释。→ 自查：`find ~/.claude/commands -name "*.md" | xargs grep -L "^---"`。

2. **shell 注入：codex-cli 和 zai-cli agent**：`bash -i -c "codex -p readonly exec 'USER_PROMPT' --json"` 中，用户 prompt 包含 `'` 时可以注入任意 shell 命令。→ 根本原因：CLI 代理模式的固有风险——要把自然语言传给 CLI，必须做 shell 序列化，但如果序列化不安全，就是注入漏洞。→ 自查：检查自己的 agent/command 中是否有把 `$ARGUMENTS` 代入 bash/sh -c 的模式。

3. **`refactor-code.md` 密集模糊量词**：10+ 个"appropriate"/"comprehensive"/"significant"，NLPM 评 20 分。→ 根本原因：作者用了 AI 生成的 command 模板（AI 生成内容倾向使用模糊量词），未做人工清洗。→ 自查：`grep -c "appropriate\|comprehensive\|robust" ~/.claude/commands/*.md`，超过 3 个即应重写。

### 3.4 当时的优化机会

1. 为所有 command 加最小化 frontmatter（name + description，共 40 个文件，工具可批量完成）
2. 私下报告 codex-cli/zai-cli 注入漏洞（BLOCKED 状态，不开 public PR）
3. 将 security/ 测试固件迁移至 `.claude/skills/security/test-fixtures/` 以避免被注册为 slash command

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 20个 command 无 frontmatter | `grep -l "^---" .claude/commands/**/*.md \| wc -l` | **仍存在**：0 个文件有 frontmatter | 最高优先级缺陷完全未修复 |
| shell 注入（codex-cli）| `grep "bash -i\|zsh.*USER" .claude/agents/codex-cli.md` | **仍存在**：`bash -i -c "codex … 'USER_PROMPT' …"` 原样保留（第 31、37 行） | HIGH 安全漏洞未修复 |
| agents 缺 model 声明 | `grep "^model:" .claude/agents/*.md` | **部分修复**：codex-cli 和 zai-cli 现有 `model: haiku`；memory-bank-synchronizer、ux-design-expert、get-current-datetime 仍无 model | 两个 CLI 代理已声明，另三个未处理 |

### 4.2 架构演进

- **显著扩展**：新增 `.claude/rules/core-rules.md`（集中规则管理）、`.claude/hooks/unified_notifier.py.md`（统一通知 hook）、`.claude/mcp/`（MCP 配置）
- **Settings 文件化**：`settings.json` 和 `settings.local.json` 现在版本控制，说明作者开始把配置视为代码
- **新增 SKILL**：`task-breakdown/SKILL.md` 和 `audit-session-metrics/SKILL.md` 新增，session-metrics 延伸为更精细的审计能力
- **整体方向**：从"简单个人配置"向"更系统化的工程化配置"演进，但核心 bug（frontmatter 缺失、shell 注入）仍未修复

### 4.3 新增的可学习模式

**unified_notifier hook**：一个 Python hook 文件统一处理所有通知逻辑（命名为 `unified_notifier.py`），而非为每种事件写单独脚本。这是 **hook 集中化** 模式——单一入口，内部分发，便于日志和调试。比分散的多个 hook 文件更易维护。

---

## 五、校准

### 5.1 我已经在做对的

1. **`echo-sleuth-for-claude` 的 agent 有 model 声明**：recall.md 等 agent 文件有 model 字段，和 codex-cli/zai-cli 修复后的状态一致。
2. **`claude-for-legal` 的 command 在部分文件中有 frontmatter**：legal-builder-hub/ 下 skill 有 allowed-tools，说明对 frontmatter 有意识。
3. **我的 agent 中没有 shell 注入模式**：未发现把 `$ARGUMENTS` 直接代入 `bash -c` 的模式。

### 5.2 挑战 / 验证

**挑战**：这个案例挑战了"star 数反映代码质量"的直觉。centminmod/my-claude-code-setup 有 2210 stars，但 NLPM 评分仅 49/100，包含两个 HIGH 安全漏洞，且审查后超过一个月仍未修复。Star 数反映的是"有用"或"有名"，不是"符合规范"——这在开源世界是普遍现象，但在 NL 编程领域尤其明显，因为"能用"和"正确用"之间有很大的鸿沟。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 agent 是否有 shell 注入风险（最高优先级）
grep -rn "bash -c\|bash -i\|sh -c\|shell.*ARGUMENTS\|\$ARGUMENTS.*bash" \
  /tmp/my-repos/MarkQWu-*/agents/*.md 2>/dev/null

# 检查 agent 是否都有 model 声明
find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/agents/*" | while read f; do
  echo -n "$(basename $f): "
  grep "^model:" "$f" 2>/dev/null || echo "⚠️ NO model"
done

# 检查 command 的 frontmatter 覆盖率
find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/commands/*" \
  | xargs grep -L "^---" 2>/dev/null | wc -l
echo "上述数字应为 0（全部有 frontmatter）"
```

命中 shell 注入风险后：立即修复，使用 `shlex.quote()` 或改为参数化调用，不要直接字符串插值。
命中 model 缺失后：分析 agent 任务复杂度，简单机械任务→haiku，需要推理→sonnet，复杂分析→opus。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 `unified_notifier` hook 设计，为 `echo-sleuth-for-claude` 添加单一 PostToolUse hook，记录每次工具调用日志。
   - **为何可行**：echo-sleuth 核心功能是挖掘历史，有自己的 hook 记录工具使用会提供更丰富的历史数据。
   - **第一步**：创建 `.claude/hooks/echo-logger.py`，监听 PostToolUse，将工具名+时间戳写入本地日志。预计 30 分钟。

2. **想法**：为 `claude-for-legal` 的 CLAUDE.md 创建多套模板（类似 centminmod 的多套 CLAUDE.md），针对不同法律实践领域（诉讼律师 vs 公司律师 vs 合规专员）。
   - **为何可行**：claude-for-legal 已有多个领域 CLAUDE.md；统一入口模板可以让用户选择"快速配置"。
   - **第一步**：创建 `CLAUDE-litigation.md`、`CLAUDE-corporate.md` 两个专用模板，各引用对应领域 skill。预计 45 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 centminmod/my-claude-code-setup 的核心目的**：个人 Claude Code 环境配置共享，含 CLI 工具代理 agent、会话统计 skill 和多套 CLAUDE.md 模板

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是个人 Claude Code 插件；都有 agent + skill 组合；都关注会话历史 | echo-sleuth 功能更聚焦（挖掘历史），centminmod 是综合配置 | 高 |
| MarkQWu/claude-for-legal | 低 | 都有多个 CLAUDE.md 模板 | 垂直领域不同；claude-for-legal 不是"个人配置"而是"通用产品" | 低 |
| MarkQWu/drama-workshop-skills | 无 | 都是 skill 仓库 | 功能和目标完全不同 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 缺 model 声明 | `grep -L "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md` | 需人工检查（前述 agent 文件有 frontmatter，但 model 是否声明未确认） | 中 |
| Command 缺 frontmatter | `grep -L "^---" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | recap.md、timeline.md 有无 frontmatter 需确认 | 中 |
| Shell 注入（$ARGUMENTS 代入 bash） | `grep -rn "bash.*ARGUMENTS\|sh.*ARGUMENTS" /tmp/my-repos/MarkQWu-*/agents/` | 未发现命中（0处） | 无 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/agents/*.md` → 对每个 agent 加 `model:` 声明：analyze.md → `model: sonnet`（分析复杂度高）；recall.md → `model: haiku`（检索任务）。5 分钟可完成。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：多 CLAUDE.md 模板按云平台场景区分
   - **本案例做法**：`CLAUDE-cloudflare.md`、`CLAUDE-convex.md`、`CLAUDE-template-*.md` 覆盖不同部署场景，用户按需 copy 到自己的项目
   - **我的项目现状**：`claude-for-legal` 各领域有独立 CLAUDE.md 但没有"快速选择"的入口模板
   - **如何借鉴**：在 `claude-for-legal` 根目录创建 `QUICKSTART-CLAUDE.md`，引导用户选择对应领域 CLAUDE.md。改动 1 个文件，15 分钟。

2. **领域**：unified hook 集中化
   - **本案例做法**：`.claude/hooks/unified_notifier.py` 单一入口处理所有 hook 事件
   - **我的项目现状**：`echo-sleuth-for-claude` 无 hook，NLPM 本身的 hooks/hooks.json 是单 hook
   - **如何借鉴**：若 echo-sleuth 需要多个 hook（记录工具使用 + 记录对话摘要），从一开始就用 unified notifier 模式，避免多 hook 管理复杂

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：NL 工件质量标准
- **我的做法**：`MarkQWu/echo-sleuth-for-claude` 有 NLPM badge（`[![Validated by NLPM]...]`），说明仓库经过 NLPM 校验，质量有背书
- **本案例做法**（弱在哪）：centminmod/my-claude-code-setup 的 NLPM 评分仅 49/100，BLOCKED 状态，无质量背书
- **意义**：echo-sleuth 的 NLPM badge 是对外的质量信号，值得在 claude-for-legal 和 drama-workshop-skills 中也引入（前提是评分达到阈值）

---

## 八、术语表

### <a name="shell-injection"></a>shell 注入（shell injection）
> 一种安全漏洞：程序把用户输入的字符串直接拼接到 shell 命令中执行，导致恶意输入可以注入并执行任意命令。例如，把用户输入 `'; rm -rf ~/; echo '` 代入 `bash -c "codex '用户输入' --json"` 后，会变成 `bash -c "codex ''; rm -rf ~/; echo '' --json"`，删除主目录。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置，声明 `name`、`description` 等元数据。Claude Code 注册 slash command 必须读取 frontmatter 中的 `name` 字段；没有 frontmatter 的 command 文件对 Claude Code 不可见，无法被 `/` 触发。

### <a name="model-pinning"></a>model 声明（model pinning）
> 在 agent 定义文件 frontmatter 中用 `model: haiku`、`model: sonnet`、`model: opus` 声明该 agent 使用的 Claude 模型版本。未声明时，agent 继承调用者的当前模型——可能用昂贵的 Opus 做"获取时间"这类简单任务，浪费成本；也可能用 Haiku 做需要复杂推理的任务，降低质量。

### <a name="cli-proxy-agent"></a>CLI 代理 agent（CLI proxy agent）
> 一种 agent 设计模式：agent 的主要工作是把自然语言 prompt 转换为某个外部命令行工具（CLI）的调用，把 CLI 输出返回给用户。codex-cli 和 zai-cli 是典型示例。这种模式的核心风险是"用户输入 → shell 命令字符串"的序列化过程中的注入漏洞。

### <a name="trustedDependencies"></a>settings.json（Claude Code 全局设置）
> `.claude/settings.json` 是 Claude Code 的权限和行为配置文件，控制 allowed tools、hook 路径、permission 策略等。将 settings.json 加入版本控制（如 centminmod 所做）意味着权限配置对所有 clone 用户可见，这是透明度和风险并存的选择。
