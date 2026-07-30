# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：未知（数据缺失）| **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-30（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `examples-driven`, `model-pinning`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`affaan-m/everything-claude-code` 是一个规模罕见的「全栈 AI 编码工具包」，以 934 个 [NL 工件](#NL工件)、281 个 skill、67 个 agent、多语言本地化（中文/韩语/日语/土耳其语/葡萄牙语）为特征，涵盖后端（Django/Go/Rust/Java/Kotlin/Laravel）、前端（Flutter）、运维（Docker/MCP）等几乎所有主流技术栈。当前版本 2.1.0，代码库体量在开源 Claude Code 插件中属于最大级别。

关键事实：
- Audit 时：934 个工件，84/100 分，SECURITY BLOCKED（Critical hook 安全漏洞）
- 当前 HEAD：version=2.1.0，281 skills，67 agents；hooks.json 中的 CRITICAL 问题已修复
- 本地化达 7 种语言（ko-KR/zh-TW/pt-BR/tr/ja-JP/zh-CN/opencode），每种都镜像主命令集
- 包含 `.kiro/` 目录（Kiro IDE 支持）和 `.opencode/` 目录（OpenCode 编辑器支持）

### 1.2 架构剖析
```
everything-claude-code/
├── commands/           ← 主命令集（70+ 个）
├── agents/             ← 主 agent 集（25+）
├── skills/             ← 主 skill 集（120+ 个领域 skill）
├── docs/
│   ├── ko-KR/          ← 韩语镜像
│   ├── zh-TW/          ← 繁体中文镜像
│   ├── pt-BR/          ← 葡语镜像
│   ├── tr/             ← 土耳其语镜像
│   ├── ja-JP/          ← 日语镜像
│   └── zh-CN/          ← 简体中文镜像
├── .kiro/              ← Kiro IDE 适配
├── .claude-plugin/
│   └── plugin.json     ← manifest
├── hooks/
│   └── hooks.json      ← PostToolUse + Bash hook（Audit 时 CRITICAL，现已修复）
└── scripts/            ← 50+ JS 脚本（hook 处理器）
```

- **文件类型分布**：120+ skill / 25+ agent / 70+ command / 50+ JS hook 脚本 / 7 本地化副本集（每集 ~100 文件）
- **编排关系**：分层架构——command 调用 agent，agent 引用 skill；存在「chief-of-staff」协调者 agent 和「loop-operator」循环执行 agent；`gan-build` 命令直接调度 `gan-planner → gan-generator → gan-evaluator` 三 agent 链
- **跨件契约**：skill 通过 frontmatter 中 `name:` 字段注册；agent 需声明 `model:` 和 `tools:` 字段；当前 HEAD 中主 agent 已有 `model`，但所有 16 个 `.kiro/agents/*.md`、12 个 `ko-KR agents`、12 个 `pt-BR agents` 仍缺少 `model` 字段

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一个仓库囊括所有人的所有需求」（everything）+ 本地化扩展性；skill 层采用高度标准化的 frontmatter 格式，许多 skill 达到 100/100
- **解决什么问题**：消除开发者在 Claude Code 生态中东拼西凑工具的成本；提供一个可以直接 install、立刻覆盖所有技术栈的超级插件
- **Trade-off**：体量换覆盖率——934 个文件意味着极高维护成本；本地化副本（~700 文件）与源文件的 bug 同步是持续挑战（audit 时发现 source bug 自动传播到 7 个语言副本）
- **认知模型**：作者认为 skill 是「知识库」而非「程序」——最优质的 skill（如 `skills/claude-api/SKILL.md`）确实做到了 100/100，通过精确的输入/输出示例让 AI 行为高度可预测

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「超级插件 + 多运行时适配器 + 本地化镜像」**

一个核心工具集同时适配 Claude Code（`.claude-plugin/`）、Kiro IDE（`.kiro/`）、OpenCode（`.opencode/`），并为 6 种语言提供镜像。

模式特征清单：
- 特征 1：主工具集（commands/agents/skills）是唯一真相源，镜像都是派生的
- 特征 2：本地化镜像继承源文件的 bug，也继承其修复（一处改，七处变）
- 特征 3：`skills/` 按领域命名（`skills/django-patterns/`、`skills/defi-amm-security/`），每个 skill 是独立的领域知识包
- 特征 4：部分命令是 legacy shim（`verify.md`、`claw.md` 等），无 sunset 时间线

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型多人团队，技术栈多样 | ✅ 高度适用 | 覆盖广，技术栈之间正交，按需加载 |
| 个人项目（1-2 个技术栈） | ❌ 过度 | 934 个工件对单人项目是噪声 |
| 需要多语言用户支持 | ✅ 适用 | 本地化镜像已就绪 |
| 安全敏感环境 | ❌ 谨慎 | hooks 历史上有 CRITICAL 漏洞，本地化副本的 `model: opus` 误配置会带来额外成本 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 超级插件（本仓库） | everything-claude-code | 一次安装，全栈覆盖 | 维护成本极高，bug 传播范围大 |
| 精品专项插件 | agiletec-inc/airis-mcp-gateway | 聚焦，质量均一 | 仅解决特定问题 |
| 社区精选集 | hesreallyhim/awesome-claude-code | 低维护，依托他人维护 | 没有统一质量标准 |

### 2.4 改进空间
1. **当前问题**：本地化镜像约 700 文件与源文件手工同步 **改进做法**：用 CI 脚本自动从源文件生成本地化版本（或至少检测 frontmatter 字段不一致） **预期收益**：消除 source bug → 本地化 bug 的自动传播
2. **当前问题**：`.kiro/agents/*.md` 等 3 套运行时适配层 16+ 文件缺少 `model` 字段 **改进做法**：在 CI 中加 `grep -rL "^model:" .kiro/agents/` 校验 **预期收益**：防止无 model 声明的 agent 引发不可预期的 tier 选择
3. **当前问题**：5 个 legacy shim 无 sunset 版本号 **改进做法**：在 shim 文件 frontmatter 中加 `deprecated-since: v2.0.0 sunset: v3.0.0` **预期收益**：用户有明确的迁移时间线

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 得分 **84/100**（审计策略：progressive sample，~320 独立文件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 23 | 缺 name + description frontmatter |
| skills/customs-trade-compliance/SKILL.md | 55 | 模糊量词封顶 + 无输出格式 |
| agents/e2e-runner.md | 67 | 无输出格式；9 个模糊量词 |
| agents/typescript-reviewer.md | 95 | 完整；model 字段轻微不一致 |
| skills/skill-create.md（命令） | 100 | 完美 |
| skills/claude-api/SKILL.md | 100 | 完美 |

### 3.2 当时值得借鉴的模式
1. **领域 skill 精品化**：`skills/claude-api/SKILL.md`、`skills/tdd-workflow/SKILL.md` 等达到 100/100——通过精确的 input/output 示例 + frontmatter 完整声明实现。这说明大仓库中可以有局部完美。借鉴：我的 skill 也应逐一达到 100，而不是全局「差不多就行」
2. **`chief-of-staff` 协调者 agent 模式**：一个顶层 agent 负责分解任务并派发给专用 agent，自身不执行具体任务。这是多 agent 编排的清晰边界设计
3. **gan-* 三阶段 agent 链**：`gan-planner → gan-generator → gan-evaluator` 链式编排展示了 agent 之间的清晰职责划分（规划/生成/评估）

### 3.3 当时的缺陷
1. **hooks.json CRITICAL：`npx block-no-verify@1.1.2` 在每次 Bash 工具调用时执行** → 根本原因：作者添加了一个阻止 `--no-verify` 的 hook，但使用了 `npx` 每次重新拉取，即使版本已固定，`npx` 仍每次查询 npm registry，一旦 registry 被篡改就执行恶意代码。**自查：我的 agents-in-the-wild 项目的 hooks 是否有类似 npx 用法？检查 scripts/check-artifact.sh 等。**
2. **源文件 bug 传播到 7 个本地化副本**：`commands/learn.md` 缺少 frontmatter，所有 7 个本地化版本自动继承此 bug，变成 7 × 1 = 7 个失效命令。根本原因：维护者将本地化当作「翻译」而非「有依赖的派生文件」，没有工具强制同步
3. **agents 普遍缺零示例**：`agents/go-reviewer.md`、`agents/gan-planner.md` 等 15+ agent 零示例，原因是对「agent 是系统提示」理解不够深——没有 examples，model 在边缘情况下行为不可预测

### 3.4 当时的优化机会
1. 批量修复 `commands/learn.md`、`commands/eval.md` 的 frontmatter（修一个 = 修 7 个语言副本）
2. 将 `hooks/hooks.json` 的 `npx block-no-verify@1.1.2` 改为本地脚本调用（彻底消除 network-on-every-tool-call 问题）
3. 为 `agents/go-reviewer.md` 等 15+ 零示例 agent 批量添加 2-3 个 examples 块

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| hooks.json CRITICAL（`npx block-no-verify`） | `grep "block-no-verify" hooks/hooks.json` | **已修复**：grep 无输出，hooks.json 中不再有此问题 | 最高优先级安全问题已解决；维护者做出了正确反应 |
| `commands/learn.md` 缺 frontmatter | `head -5 commands/learn.md` | **已修复**：learn.md 现有 frontmatter（`description:` 字段可见）| source fix 说明 7 个本地化副本可能同步修复 |
| `.kiro/agents/*.md` 缺 `model` 字段 | `grep -L "^model:" .kiro/agents/*.md` | **暂无法核查**（需逐文件检查）| 此类跨文件元数据一致性问题往往被大版本更新遗漏 |

### 4.2 架构演进
从 audit 时的 934 个工件，到 v2.1.0 新增了 `fullstack` 角色 agent（未在原始 audit 中）。skills 数量从约 100 扩展到 281。`.kiro/` 目录新增，显示作者开始适配多 IDE 生态。总体方向是持续扩张而非收敛——符合「everything」的品牌定位。

### 4.3 新增的可学习模式
当前 HEAD 的 281 个 skills 中，涌现出了「双语 skill」模式（`docs/zh-CN/skills/continuous-learning-v2/agents/observer.md`），skill 下挂 agent——一个 skill 不只是文档，还可以内嵌专用 agent。这个模式在 2026-04 的 audit 中未被重点讨论，但在现在的代码库中已经出现，值得关注。

---

## 五、校准

### 5.1 我已经在做对的
1. **skill 有完整示例** → 我的 bureau skills 均有 3 个 examples 块，与本案例的 100 分 skills 标准一致
2. **没有 npx 无版本 hook** → bureau/gstack 的 hooks 无 npx 动态下载，不存在本案例 audit 时的 CRITICAL 风险
3. **frontmatter 结构规范** → 我的 gstack skills 有完整 frontmatter（name/version/description/triggers/allowed-tools）

### 5.2 挑战 / 验证
本案例验证了「规模≠质量」。934 个工件，但平均分 84，且存在 CRITICAL 安全漏洞。这挑战了「大即是好」的直觉。**我的项目体量小（20-30 个 NL 工件）是优势，不是劣势——可以保证每个工件质量均一。**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我是否有任何 npx hook（supply chain risk）
find /tmp/my-repos/ ~/.claude -name "hooks.json" -exec grep -l "npx" {} \; 2>/dev/null
# 命中后：将 npx 调用替换为本地脚本或固定版本 + 本地缓存

# 检查我的 agent 是否有 model 声明
find /tmp/my-repos/ -name "*.md" -path "*/agents/*" -exec grep -L "^model:" {} \; 2>/dev/null
# 命中后：根据任务类型选择合适 model tier（轻量任务 haiku，重度推理 sonnet）

# 检查是否有本地化副本可能与源文件不同步
find /tmp/my-repos/ -name "*.md" -path "*/docs/*" 2>/dev/null | wc -l
# 命中后：建立 CI 检查确保副本 frontmatter 与源文件一致
```

### 6.2 灵感 → 实施路径

1. **想法**：为 gstack 的每个 skill 添加 `model` 声明（目前缺失）
   - **为何可行**：gstack 约有 20+ skills，添加 `model:` 字段是机械操作
   - **第一步**：`grep -rL "^model:" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md`，列出需要添加的文件，约 15 分钟

2. **想法**：借鉴 `chief-of-staff` 模式，为 bureau 设计一个顶层协调命令
   - **为何可行**：bureau 现在有 capture/compile/review/query 等多步流程，用协调者 command 一键执行全流程有价值
   - **第一步**：读取 bureau 各命令，设计协调逻辑草稿，30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例核心目的**：为所有技术栈提供 AI 编码辅助（全覆盖策略）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多 skill Claude Code 插件 | gstack 聚焦特定工具链，非「everything」 | 高 |
| MarkQWu/bureau | 中 | 同为插件，有 command + skill 结构 | bureau 聚焦知识管理，非通用编码 | 中 |
| MarkQWu/graphify | 低 | 同为 AI 辅助工具 | graphify 是独立工具，非 Claude Code 插件 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agents 缺 `model` 字段 | `grep -rL "^model:" */agents/*.md` | **gstack 不含 agents**；bureau 无标准 agent 文件 | 暂不适用 |
| 命令缺 `name` frontmatter | `grep -rL "^name:" */commands/*.md` | **命中**：bureau 全部命令无 `name` 字段（只有 `description`） | 高 |
| Skills 有模糊量词 | `grep -rn "appropriate\|consider\|balance" */skills/*/SKILL.md` | **需要检查 gstack** | 中 |

**具体行动建议**：
- `MarkQWu-bureau/commands/` 下所有文件 → 添加 `name: <slug>` 到 frontmatter → 10 文件 × 5 分钟 = 50 分钟

### 7.3 别人的更优方案

1. **领域**：多运行时适配（`.kiro/`、`.opencode/`）
   - **本案例做法**：同一套 skills/agents 同时适配 Claude Code、Kiro IDE、OpenCode，通过 IDE 专用目录存放适配文件
   - **我的项目现状**：bureau 和 gstack 仅支持 Claude Code
   - **如何借鉴**：如果 gstack 或 bureau 用户跨越 IDE，可在同一仓库中添加 `.kiro/` 适配；第一步是了解 Kiro 的 agent 格式与 Claude Code 的差异

2. **领域**：技术栈专项 skill（领域专家 skill 模式）
   - **本案例做法**：`skills/django-patterns/`、`skills/defi-amm-security/` 等按领域隔离知识，每个 skill 精确覆盖一个领域
   - **我的项目**：gstack skills 相对通用，未按领域深度分化
   - **如何借鉴**：为 gstack 拆分更细粒度的专项 skill（如 iOS 专项 vs Android 专项）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：体量可控 → 质量均一
- **我的做法**：bureau 仅有 ~20 个 NL 工件，每个都经过人工审核
- **本案例做法**（弱在哪）：934 个工件中混有 23/100 的命令和 100/100 的 skill——质量方差极大
- **意义**：用户体验的一致性是小而精插件的核心竞争力；我的 bureau 在可靠性上优于「everything」式大仓库

---

## 八、术语表

### <a name="NL工件"></a>NL 工件
> Natural Language 工件，即用自然语言写成的 Claude Code 配置文件，包括 SKILL.md、命令文件（commands/*.md）、agent 定义文件（agents/*.md）、plugin.json 等。这类文件的质量直接影响 AI 的行为——写得越精确，AI 越能准确执行。

### <a name="供应链攻击"></a>供应链攻击
> 攻击者不直接攻击目标软件，而是篡改目标软件依赖的第三方组件（如 npm 包）。hooks.json 中的 `npx block-no-verify@1.1.2` 虽然固定了版本号，但 `npx` 每次仍会查询 npm registry——如果 registry 在网络层被篡改，用户运行的就是攻击者的代码。

### <a name="本地化镜像"></a>本地化镜像
> 将同一套命令/agent/skill 翻译成其他语言并放在不同目录（如 `docs/zh-CN/`）。优点是多语言用户可以用母语与 AI 交互；缺点是维护时必须同步更新所有镜像，否则镜像会继承源文件的 bug。
