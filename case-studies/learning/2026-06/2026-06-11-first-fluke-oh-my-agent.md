# first-fluke/oh-my-agent — 学习案例

**仓库**：https://github.com/first-fluke/oh-my-agent
**Stars**：663 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-06-11（基于当前 HEAD）
**主题标签**：`router-channels`, `security-gate`, `curl-pipe-bash-risk`, `manifest-discipline`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`oh-my-agent`（简称 `oma`）是一个**厂商无关的多智能体编排框架**，v8.52.5。它将整个项目的「AI 工作范式」打包成可安装的 Claude Code 插件：一套 skill 库、一套 workflow 库、一套 agent 团队、以及多厂商 hook 变体（Claude Code / Gemini CLI / Codex CLI / Qwen）。用户通过 `curl | bash` 安装，框架自动在 `.agents/` 目录落地全部组件。核心亮点：发一句自然语言就能触发「orchestrate / ultrawork / ralph」等复杂多步工作流。

关键事实：
1. 作者 First Fluke（韩国），2025 年初发布，GitHub 663 stars，npm 月下载量活跃
2. 支持 Claude Code、Gemini CLI、Codex CLI、Qwen 4 个 AI 运行时，通过 hook 变体配置切换
3. 当前 33 个 skill，10 个 agent，20+ workflow，相比 2026-04-20 审计时大幅扩充
4. 用 `serena MCP`（符号感知搜索）替代原生 grep/find，是相对少见的工具优先策略

### 1.2 架构剖析

```
.agents/
├── agents/          # 10个专业 agent（architecture-reviewer, backend-engineer 等）
├── skills/          # 33个 skill（oma-backend, oma-translator 等），每个子目录含 SKILL.md + resources/
├── workflows/       # 20+个工作流 markdown（orchestrate.md, ultrawork.md 等）
├── hooks/           # core/（TypeScript 脚本）+ variants/（claude/gemini/codex/qwen JSON）
├── rules/           # 规则文件（.agents/rules/*.md）
└── oma-config.yaml  # 运行时配置：语言、目标厂商、agent 映射
cli/                 # bun+TypeScript 的 CLI 工具（install.sh 入口）
hooks/
└── hooks.json       # 插件安装后的 Claude Code hook 配置（UserPromptSubmit + Stop）
.claude-plugin/
└── plugin.json      # 插件清单（paths 指向 .claude/ 安装目标，不是源 .agents/）
CLAUDE.md            # 自动维护的 OMA:START 块，包含 workflow 路由表
```

- **文件类型分布**：33 个 SKILL.md，10 个 agent，20+ workflow，4 个 hook 变体 JSON，1 个 plugin.json
- **编排关系**：CLAUDE.md 的关键词自动检测层 → 路由到 `.agents/workflows/*.md` → workflow 按步调用 agent（`oma agent:spawn` 或运行时原生 subagent） → agent 加载对应 skill → skill 引用 `resources/*.md` 辅助文件
- **跨件契约**：所有 skill 均引用 `resources/execution-protocol.md`（厂商注入协议），workflow 通过 session-id 传递上下文，`oma-config.yaml` 是全局单一配置源

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「安装即可用，约定优于配置」——`oma install` 把所有组件落地到 `.claude/` 目录，无需用户配置路径
- **解决什么问题**：跨厂商 AI agent 编排的摩擦——换用 Gemini CLI 时所有 workflow 和 skill 还能用
- **Trade-off**：skill 内容依赖外部 `resources/` 文件，导致单文件可读性弱、审计不透明；换取了内容复用和文件精简
- **认知模型**：把 AI agent 看作「可插拔的领域专家团队」——每个 agent 有专业定位，workflow 是工作指令，hook 是感知触发器

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「NL 触发层 + 多厂商适配层 + 资源解耦 skill」**

关键特征：
- 自然语言关键词通过 UserPromptSubmit [hook](#hook) 实时检测，自动路由 workflow
- hook 变体以 JSON schema 形式存储，同一 skill 逻辑在多个 AI 运行时复用
- skill 内容本身轻量（SKILL.md 简洁），重量信息外置到 `resources/*.md`
- [manifest](#manifest) 中 `agents/skills` 路径指向安装目标（`.claude/`），不是源目录（`.agents/`）
- 版本已到 8.52.5，说明团队在持续高频迭代

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型工程项目，需多 agent 协作（前端+后端+QA） | ✅ 高度适用 | 10 个专业 agent + 20+ workflow，开箱即用 |
| 跨 AI 运行时团队（有人用 Claude，有人用 Gemini） | ✅ 适用 | hook 变体机制天然支持 |
| 个人小工具项目 | ❌ 过重 | 33 个 skill + CLI 安装流程，杀鸡用牛刀 |
| 需要完全离线的环境 | ❌ 不适用 | install.sh 依赖网络拉取 bun/uv 安装器 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 触发 + 多厂商适配 | oh-my-agent | 跨平台，关键词自动触发，skill 复用 | install.sh 安全风险，skill 内容外置不透明 |
| 单仓多 skill 平铺 | wshobson/agents | 简单直观，无安装步骤 | 无工作流编排，agent 间无协作协议 |
| 单职责插件（每个功能一个 plugin） | google-labs-code/stitch-skills | 边界清晰，可单独安装 | 无跨插件编排，workflow 缺失 |

### 2.4 改进空间

1. **当前问题**：`plugin.json` 中 `skills/agents` 路径指向安装目标 `.claude/`，在源码开发时迷惑性强。**改进做法**：增加 `src` vs `dist` 说明注释，或用构建脚本生成最终 plugin.json。**预期收益**：降低首次贡献者迷惑。
2. **当前问题**：所有 skill 依赖 `resources/execution-protocol.md`，但该文件未进入审计范围，skill 质量部分不可验证。**改进做法**：在每个 SKILL.md 头部加 `## Execution Protocol (summary)` 行内摘要，资源文件作为详述补充。**预期收益**：单文件可读性提升，AI 降级时也能正确执行基础流程。
3. **当前问题**：10 个 agent 均无 `model` 字段，Claude Code 无法路由到正确的模型层级。**改进做法**：根据 agent 职责分级声明（如 `model: haiku` 用于快速分类，`model: sonnet` 用于深度分析）。**预期收益**：降低 token 开销，加快低复杂度任务响应。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-20 得分 **86/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| oma-translator/SKILL.md | 95 | 最优秀的工件，7 阶段工作流 |
| oma-orchestrator/SKILL.md | 90 | 「oh-my-ag agent:spawn」typo |
| oma-db/SKILL.md | 92 | 「oh-my-agent agent:spawn」CLI 不一致 |
| 所有 9 个 agent 文件 | 80 | 无 model 字段，无示例 |
| oma-coordination/SKILL.md | 80 | 无 output format |

### 3.2 当时值得借鉴的模式

1. **oma-translator 的 7 阶段工作流** → 为什么好：每个阶段有具体输入输出约定，AI 无需猜测下一步。路径：`.agents/skills/oma-translator/SKILL.md`。如何借鉴：为自己每个 skill 定义 3-5 个显式阶段，而不是一大段自然语言指令。
2. **hook 变体 JSON Schema** → 为什么好：用 schema 约束厂商特定的 hook 配置，防止格式漂移。路径：`.agents/hooks/variants/hook-variant.schema.json`。如何借鉴：配置文件用 JSON Schema 验证。
3. **qa-reviewer 的 Output Format section** → 为什么好：明确声明产出到 `.agents/results/result-qa.md`，agent 不会自由发挥输出位置。路径：`.agents/agents/qa-reviewer.md`。如何借鉴：每个 agent/skill 都加 `## Output Format`，明确文件路径和内容结构。

### 3.3 当时的缺陷

1. **问题**：9 个 agent 全部缺少 `model` 字段。**根本原因**：作者没有意识到 Claude Code 用 `model` 字段路由模型层级，默认行为会使所有 agent 用同一模型，无法实现轻量任务用便宜模型的优化。**自查**：我的 echo-sleuth agent 有没有声明 model？→ 没有声明。
2. **问题**：oma-orchestrator 有「oh-my-ag」typo（截断命令名）。**根本原因**：CLI 别名 `oma` 和全名 `oh-my-agent` 并存，作者在编辑时混用导致截断未被发现。**自查**：我的项目 CLI 调用是否有类似的名称不一致？→ echo-sleuth 中命令名与调用名需要对照检查。
3. **问题**：`cli/install.sh` 使用 `curl|bash` 安装 bun 和 uv，无校验和。**根本原因**：优先考虑安装便利性，没有把供应链安全纳入设计约束。**自查**：我的项目 install.sh 有类似模式吗？→ drama-workshop-skills 有 install.sh，需要检查。

### 3.4 当时的优化机会

1. 为所有 9 个 agent 添加 `model` 字段（每个 agent 1 行改动，高性价比）
2. oma-coordination skill 加 `## Output Format` section
3. gemini.json hook 变体把超时单位（5000ms vs 其他变体的 5s）对齐并文档化

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| oma-orchestrator typo「oh-my-ag agent:spawn」| `grep -n "oh-my-ag"` in oma-orchestrator/SKILL.md | **已修复** ✓ — 当前文件中只有 `oma agent:spawn` | Bug PR 修复有效（或作者自行修复） |
| oma-db CLI 不一致「oh-my-agent agent:spawn」| `grep -n "oh-my-agent agent:spawn"` in oma-db/SKILL.md | **部分残留** — 第 179 行仍为 `oh-my-agent agent:spawn` | 一致性问题尚未彻底解决 |
| 所有 agent 无 model 字段 | `grep -rn "^model:" .agents/agents/` | **仍未修复** — qa-reviewer 等文件无 model 声明 | 模型路由问题持续 |

### 4.2 架构演进

从审计时的 ~20 个 skill → 现在 33 个 skill。新增：oma-academic-writer, oma-deepsec, oma-image, oma-market, oma-observability, oma-scholar, oma-skill-creator, oma-slide, oma-video, oma-voice 等。这说明作者的策略是**向垂直领域快速扩张**（学术写作、安全、媒体生成、幻灯片），而非深化现有 skill 的质量。agents 从 10 保持不变，但 workflows 文件有更新。

### 4.3 新增的可学习模式

- **serena MCP 优先策略**：CLAUDE.md 中明确列出「Prefer serena MCP tools over native find/grep」，并附带具体操作映射表（find_symbol → grep, list_dir → ls 等）。这是把第三方工具融入工作流的优雅方案——不是替换，而是明确的优先级声明 + 降级路径。
- **`oma-config.yaml` 作为单一运行时配置**：通过一个 YAML 文件控制语言、目标厂商、agent 映射，而不是散落在各 skill 里的 if/else 条件。

---

## 五、校准

### 5.1 我已经在做对的

1. **CLAUDE.md 作为架构文档**：echo-sleuth 的 CLAUDE.md 也采用了类似的「架构概览 + 文件路径说明」结构，这是对的。
2. **Commands 声明 allowed-tools**：echo-sleuth 的 commands 都有 `allowed-tools`，与 qa-reviewer 的好模式一致。
3. **agent 专业化分工**：echo-sleuth 把 recall/file-historian 等功能拆分成独立 agent，与 oh-my-agent 的架构思路一致。

### 5.2 挑战 / 验证

本案例验证了一个我之前犹豫过的做法：**工作流文件（workflow）值得从 skill 中分离出来**。oh-my-agent 单独维护 `workflows/` 目录，skill 负责「知道什么」，workflow 负责「做什么步骤」，两者职责清晰。我的 echo-sleuth 命令文件混合了流程和知识，应该拆分。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否声明了 model 字段
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/ -name "*.md" -exec grep -L "^model:" {} \;
# 命中后：为每个 agent 根据复杂度选择合适的 model（haiku/sonnet/opus）
```

```bash
# 检查 install.sh 是否有 curl|bash 模式
grep -n "curl.*|.*bash\|curl.*|.*sh" /tmp/my-repos/MarkQWu-drama-workshop-skills/install.sh 2>/dev/null
# 命中后：改为 curl -o script.sh + 人工确认后执行，或加 checksum 校验
```

```bash
# 检查 CLI 调用名是否在所有 skill 中一致
grep -rn "agent:spawn\|skill:invoke" ~/.claude/plugins/ 2>/dev/null | sort | uniq -c | sort -rn | head -20
# 命中多种格式后：统一为一种规范调用格式
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 agents 添加 model 字段分级
   - **为何可行**：agents/ 目录已有 recall, file-historian 等，复杂度差异明显
   - **第一步**：打开每个 agent 文件，根据任务复杂度添加 1 行 `model: haiku/sonnet`，5 分钟可完成

2. **想法**：从 echo-sleuth 命令文件中提取「工作流」到单独目录
   - **为何可行**：/extract 等命令既有知识（skill）也有流程（workflow），混在一起
   - **第一步**：创建 `workflows/` 目录，把 extract.md 中的「Step 1/2/3」部分移至 workflows/extract.md，commands/extract.md 只保留触发逻辑

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 first-fluke/oh-my-agent 的核心目的**：为工程开发提供完整的多 agent 工作流编排框架，支持多厂商 AI 运行时
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都有 agents/、skills/、commands/ 三层架构；都用 Claude Code 插件形式分发 | oma 有多 workflow + 多厂商支持；echo-sleuth 单一工具目的（挖掘对话历史） | 高 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件 | oma 是通用工程框架；drama 是垂直领域创作工具 | 低 |
| MarkQWu/claude-for-legal | 低 | 都有领域专业 skill | oma 的 skill 有明确 agent 调用协议；claude-for-legal 是工具集不是编排框架 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 缺少 model 字段 | `find agents/ -name "*.md" -exec grep -L "^model:" {} \;` | echo-sleuth: recall.md, file-historian.md 均无 model 字段（2/2 命中） | 中 |
| skill 无 allowed-tools | `grep -rL "allowed-tools" skills/*/SKILL.md` | echo-sleuth: 4 个 SKILL.md 均无 allowed-tools；claude-for-legal: regulatory SKILL.md 需检查 | 中 |

**命中后的具体行动建议**：
- echo-sleuth/agents/recall.md → 第 2 行添加 `model: sonnet`（复杂推理），5 分钟
- echo-sleuth/skills/memory-management/SKILL.md → 第 5 行添加 `allowed-tools: Read Write Edit Bash`，5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：工作流与知识分离
   - **本案例做法**：`.agents/workflows/` 独立目录，每个工作流 markdown 只描述步骤；skill 只描述领域知识（路径：`.agents/workflows/orchestrate.md` vs `.agents/skills/oma-orchestrator/SKILL.md`）
   - **我的项目现状**：echo-sleuth/commands/extract.md 同时包含步骤和知识，职责混合
   - **如何借鉴**：创建 echo-sleuth/workflows/ 目录，抽取命令文件中的多步骤流程

2. **领域**：model 分级声明
   - **本案例做法**：虽然当前仍有缺失，但设计上 agent 应声明 `model: haiku/sonnet/opus`，匹配任务复杂度
   - **我的项目现状**：echo-sleuth/agents/ 无任何 model 字段
   - **如何借鉴**：为 recall agent（重分析，sonnet）和 file-historian（轻查找，haiku）分别声明

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：commands 的 allowed-tools 覆盖率
- **我的做法**：echo-sleuth/commands/ 下所有命令文件均声明了 allowed-tools（dashboard: Bash, audit: Bash+Task 等）
- **本案例做法**（弱在哪）：oh-my-agent 的 command 层通过 CLAUDE.md 关键词触发，绕过了 command frontmatter，allowed-tools 约束实际上不生效
- **意义**：echo-sleuth 在工具权限约束上比 oh-my-agent 更规范。

---

## 八、术语表

### <a name="hook"></a>hook
> Claude Code 的「事件监听器」。你可以告诉 Claude Code：「每次用户发消息之前，先运行这个脚本」。oh-my-agent 用它来检测用户输入的关键词（如「orchestrate」「ultrawork」），自动把对话路由到对应的工作流文件，不需要用户手动输入命令。类比：餐厅里的接待员——顾客一开口说「我要点火锅」，接待员自动把你带到火锅区，不需要你自己找桌子。

### <a name="manifest"></a>manifest
> 插件的「目录清单」。plugin.json 告诉 Claude Code：这个插件有哪些 skill、哪些 agent、用哪个 hook 配置。如果清单里没写，那个文件即使在硬盘上也不会被加载。就像图书馆的书目系统——书架上有书，但没录入系统，读者就查不到。

### <a name="curl-pipe-bash-risk"></a>curl|bash 安全风险
> `curl url | bash` 是一种「下载后立即执行」的命令，中间没有任何检查步骤。攻击者如果能控制那个 URL（DNS 劫持、中间人攻击），就能在你的机器上执行任意代码。安全做法：先 `curl -o install.sh url`，人工看一眼，再 `bash install.sh`。oh-my-agent 的 README 就用了这个风险模式来吸引新用户快速安装。
