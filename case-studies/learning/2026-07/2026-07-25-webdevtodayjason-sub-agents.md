# webdevtodayjason/sub-agents — 学习案例

**仓库**：https://github.com/webdevtodayjason/sub-agents
**Stars**：189 | **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-25（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `manifest-discipline`, `experience-accumulation`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`sub-agents` 是一个以「岗位专家」（role-based specialist）为核心概念的 Claude Code 多 agent 插件，提供 15 个专门化 agent（DevOps 工程师、前端开发者、API 文档工程师等）+ 20 个路由命令（dispatch commands），加上一套 JavaScript CLI 框架（含 context-forge 系统和内存 API）。

关键事实：
- 作者 Jason（webdevtodayjason），YouTuber / Claude Code 内容创作者
- 当前版本 1.5.5，有详细的 CHANGELOG 和多份 RELEASE NOTES
- 当前 189 颗星，较活跃
- 核心卖点：「对话启动专家」——每个 /command 启动对应岗位的 agent

### 1.2 架构剖析
**目录结构**：
```
sub-agents/
├── agents/                 # 15 个专家 agent
│   ├── api-developer/agent.md
│   ├── code-reviewer/agent.md
│   ├── debugger/agent.md
│   ├── devops-engineer/agent.md
│   ├── frontend-developer/agent.md
│   ├── shadcn-ui-builder/agent.md
│   ├── work-completion-summary/agent.md
│   └── ...（共 15 个）
├── commands/               # 20 个路由命令
│   ├── devops.md           # → devops-engineer agent
│   ├── debug.md            # → debugger agent
│   └── ...（含 context-forge/ 子目录）
├── src/                    # JavaScript CLI 框架
│   ├── commands/           # CLI 子命令实现（js）
│   ├── memory/index.js     # 内存 API（进程内，非跨 agent 共享）
│   └── utils/
├── bin/
├── dashboard/              # Next.js 仪表盘
└── docs/
```

**文件类型分布**：15 个 agent / 20 个 command / 1 个 plugin.json（推测）

**编排关系**：用户运行 `/devops` → `commands/devops.md` 路由到 `devops-engineer` agent → agent 接管完整的 DevOps 任务。commands 是无业务逻辑的纯路由层。

**跨件契约**：`context-forge` 子系统中的命令（continue-implementation.md 等）依赖 JavaScript 内存 API（memory.get/set）。但这个 API 是 CLI 框架内部实现的，Claude Code 内 agent 无法访问——这是一个跨件契约断裂的典型案例。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「每个任务匹配最适合的专家」——不让通用 Claude 处理所有事情，而是按岗位分配专家
- **解决什么问题**：通用提示词做所有事情导致质量参差不齐；专家 agent 可以持有更深的领域知识
- **Trade-off**：15 个专家 agent 需要分别维护（model、描述、工具列表），维护成本高；但每个 agent 的质量可以单独优化
- **新特性（v1.5+）**：引入了 ElevenLabs MCP 语音播报——agent 完成任务后会通过 MCP 工具朗读完成报告，非常有创意

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「命令路由层 + 岗位专家 Agent 分层」

核心特征：
- 特征 1：command 文件是纯路由，不含业务逻辑（仅 1-2 行正文）
- 特征 2：agent 文件承载完整的岗位职责、工具列表、输出格式
- 特征 3：用户通过岗位名称（`/devops`、`/tdd`）而非技术描述来触发
- 特征 4：跨 agent 的「共享记忆」通过外部 CLI 框架（非原生 Claude Code 能力）模拟

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多种工作类型的日常开发 | ✅ 高度适用 | 每类任务都有专门 agent，省去每次手动描述上下文 |
| 单一专域的深度任务 | ⚠️ 可用 | 单个 agent 足够，不需要 15 个 |
| 需要跨 agent 共享状态 | ❌ 不适用 | 声称的「memory API」在 Claude Code 内部不可用，只是 CLI 内模拟 |
| 语音交互场景 | ✅ 特色功能 | ElevenLabs MCP 集成让 agent 完成后能语音播报 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 岗位专家分层（本仓库） | sub-agents | 心智模型清晰，用岗位名触发 | 维护 15 agent 成本高 |
| 单 agent 通用 | addyosmani/agent-skills | 维护简单 | 上下文不够深 |
| 路由 + Channels 分层 | shinpr/claude-code-workflows | 动态路由，高度灵活 | 复杂度高，新手难上手 |

### 2.4 改进空间
1. **当前问题**：context-forge 命令依赖 CLI 内存 API（Claude Code 内不可用）**改进做法**：在命令文档中明确标注「此功能仅在 CLI 模式下可用，在 Claude Code 内部仅用于描述上下文格式」**预期收益**：消除用户在 Claude Code 内运行 context-forge 命令时的困惑
2. **当前问题**：20 个命令有两个重复路由对（devops/deploy → devops-engineer，marketing/content → marketing-writer）**改进做法**：删除重复命令，或在描述中明确区分使用场景 **预期收益**：减少用户困惑，保持命令集简洁

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时得分 **78/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 48 | 无 frontmatter（name、description 缺） |
| agents/devops-engineer/agent.md | 64 | 无使用示例、无 Output Format |
| commands/debug.md | 65 | 缺 name、无空输入处理 |
| commands/devops.md | 70 | 缺 name、缺 allowed-tools |
| agents/shadcn-ui-builder/agent.md | 79 | 无 Bash 工具、失效 MCP 引用 |

### 3.2 当时值得借鉴的模式
1. **commands 作为零逻辑路由层** → 每个 command 只有 2-3 行，纯粹调用 agent → 命令层和执行层完全解耦 → 可借鉴「dispatcher + executor」的分层模式
2. **context-forge 系统的需求追踪概念**（即使实现有问题）→ PRP（Product Requirements Pass）文件用于追踪实现进度 → 概念上类似 issue 系统，但内置于 Claude Code
3. **岗位名称即命令名** → 用 `/tdd`、`/security-scan` 这样的直觉性名称，而非 `/run-tdd-specialist` → 降低认知摩擦

### 3.3 当时的缺陷
1. **所有 20 个命令缺少 `name:` 字段**：根本原因：可能以为文件名就是命令名，忽略了 [frontmatter](#frontmatter) 中的 `name:` 字段。 **这是批量性问题，说明作者的写法习惯没有被工具纠正。**自查：我的命令文件都有 `name:` 吗？
2. **shadcn-ui-builder 调用 Bash 但未声明该工具**：根本原因：agent 文件从模板复制后，修改了正文但没有同步更新工具列表。 **工具列表和正文的分离写法天然容易造成不一致。**
3. **跨件契约断裂：声称有共享 memory API，实际上无法在 Claude Code 内实现**：根本原因：作者混淆了「CLI 框架内的内存」和「Claude Code 的 agent 间共享状态」，这是两个完全不同的运行环境。自查：我的 skills 是否有类似的「声称能做但实际做不到」的功能？

### 3.4 当时的优化机会
1. 批量为 20 个命令添加 `name:` 字段和 `allowed-tools: Task`（高性价比批量修复）
2. 为 devops-engineer 添加示例对话（最低分 agent）
3. 修复 shadcn-ui-builder 的工具列表

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：命令缺 name 字段 | head -8 commands/devops.md | `name: devops` 已存在 → **已修复** | 20 个命令均已添加 name 字段 |
| Bug #3：shadcn-ui-builder 无 Bash 工具 | grep "tools:" agents/shadcn-ui-builder/agent.md | `tools: Glob, Grep, LS, ExitPlanMode, Read, NotebookRead, WebFetch, TodoWrite, Task, Bash` → **已修复** | Bash 已声明 |
| Bug #4：MCP 引用失效 | grep "MCP\|mcp" agents/shadcn-ui-builder/agent.md | 旧 MCP 引用已删，新增 ElevenLabs MCP 用于语音 → **已变更（方向转变）** | 不再声称需要 ShadCN MCP，改用声音播报 |
| 所有 agent 缺 model 字段 | grep "^model:" agents/*/agent.md | 全部已有 `model: sonnet` → **已修复** | 批量修复 |
| Security：shell:true | grep "shell:true" src/commands/dashboard.js | 无命中 → **已修复** | 1 行 diff 消除了注入风险 |

**全部已知缺陷均已修复**，且新增了 ElevenLabs MCP 语音特性——这是一个大版本迭代。

### 4.2 架构演进
- 所有 agent 从「无 model」到「model: sonnet」——这是单次批量提交能完成的事情
- shadcn-ui-builder 的 MCP 引用从「失效的 ShadCN MCP」变为「ElevenLabs 语音 MCP」——功能方向完全转变
- context-forge 子命令保留但 `allowed-tools` 仍未声明（4 个命令）——这个问题持续存在

这说明作者对「代码类缺陷」（工具列表、模型声明）的修复意愿强，对「架构类问题」（context-forge API 与 Claude Code 不兼容的文档说明）关注较少。

### 4.3 新增的可学习模式
- **ElevenLabs MCP 语音播报**：agent 完成任务后调用 `mcp__ElevenLabs__text_to_speech()` 朗读完成报告。这是一个有创意的「任务完成通知」设计，让 agent 工作变得更具临场感
- **`work-completion-summary` 专用 agent**：专门负责汇报任务完成情况，将「汇报逻辑」从业务逻辑中解耦出来

---

## 五、校准

### 5.1 我已经在做对的
1. **Skills 之间无共享状态依赖**：bureau 的每个 skill 都是独立的，不依赖跨 agent 的内存 API
2. **命令路由逻辑简洁**：gstack 的路由命令也是「薄路由层」风格
3. **vague quantifier 零容忍**：bureau skills 中 "comprehensive"、"appropriate" 出现次数 = 0

### 5.2 挑战 / 验证
这个案例展示了**修复速度可以很快**：从审计到全部缺陷修复，中间经历了约 3 个月（4月 → 7月），但这期间 repo 还完成了大版本功能迭代。这挑战了我「审计 bug 修复需要很长时间」的假设——只要作者重视，批量修复可以在一两次提交内完成。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否混淆了运行环境（CLI vs Claude Code 内部）
grep -rn "memory\.\|localStorage\|sessionStorage\|process\.env" \
  /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null
```

命中后：检查该能力是否真的在 Claude Code 内可用；若仅在 CLI 可用，在文档中明确标注。

```bash
# 检查我的 skills 是否有声称的功能但实际上无法实现
grep -rn "ALWAYS use\|MUST use\|requires.*MCP\|requires.*API" \
  /tmp/my-repos/MarkQWu-bureau/skills/ 2>/dev/null
```

### 6.2 灵感 → 实施路径
1. **想法**：仿照 `work-completion-summary` 为 bureau 加一个任务完成汇报 skill **为何可行**：每次 scribe/capture 完成后有一个简洁的汇报，提升工作流可见性 **第一步**：新建 `bureau/skills/summarize/SKILL.md`，描述「当用户要求汇报已完成工作时，输出一段结构化的工作摘要」
2. **想法**：给 gstack 的 agent 命令加 `allowed-tools: Task` **为何可行**：参考 sub-agents 的修复路径，这是一行修复 **第一步**：检查 gstack 所有 `*.md` 命令文件，加上 `allowed-tools: Task`

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 webdevtodayjason/sub-agents 的核心目的**：提供岗位专家化 agent，通过命令快速启动特定领域的专家对话

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是面向开发者的 Claude Code 工具集，都有命令路由 | gstack 是 tool-oriented，sub-agents 是 role-oriented | 高 |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件 | bureau 专注知识管理 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agents 缺 model 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | **全部 7 个命中** | 中 |
| 声称的 API 实际不可用 | 手动核查 | 无此问题，bureau 不依赖外部 API | 无 |
| 命令缺 allowed-tools | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null \| wc -l` | gstack skills 均有 allowed-tools 声明，**命中 0** | 无 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 所有 7 个 skills → 在 frontmatter 加 `model: claude-haiku-4-5-20251001`（任务较轻量适合 Haiku）→ 35 分钟完成

### 8.3 别人的更优方案

1. **领域**：批量修复 model 声明
   - **本案例做法**：一次提交为 15 个 agent 全部加上 `model: sonnet`，整齐划一
   - **我的项目现状**：bureau 7 个 skills 均无 model 声明，等待修复
   - **如何借鉴**：写一个 sed 脚本或 Python 脚本批量在 frontmatter 的 `name:` 行后面插入 `model: xxx`，一次完成

2. **领域**：任务完成语音通知
   - **本案例做法**：`work-completion-summary/agent.md` 调用 ElevenLabs MCP 语音播报完成情况
   - **我的项目现状**：bureau 任务完成后无通知机制
   - **如何借鉴**：如已集成 ElevenLabs MCP，可以在 bureau/skills/scribe/SKILL.md 末尾加一行语音汇报指令

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：消除跨件契约断裂
- **我的做法**：bureau 的 skills 不依赖任何 CLI 内存 API，所有功能在 Claude Code 内部即可完全实现
- **本案例做法**（弱在哪）：context-forge 系统依赖 `memory.get()`/`memory.set()` 等 CLI 框架内部 API，在 Claude Code 内部运行这些命令是无效的，但文档没有明确说明这个限制
- **意义**：「所见即所得」—— skill 文档里写的功能必须在 Claude Code 内部可以实现，这是我一直坚持的设计原则

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。Claude Code 依赖 `name:` 字段来注册命令，如果没有 `name:` 字段，系统只能靠文件名来猜命令名，在某些场景下会失败或产生歧义。

### <a name="context-forge"></a>context-forge
> sub-agents 中的一个子系统，用来追踪「产品需求 Pass（PRP）」的实现进度。设计上希望不同 agent 之间共享实现状态，但在 Claude Code 内部（而非 CLI 模式），这种跨 agent 共享状态实际上不可用。

### <a name="MCP"></a>MCP
> Model Context Protocol，Anthropic 定义的一套开放协议，让 Claude 可以调用外部工具服务器（如数据库、文件系统、第三方 API）。`mcp__ElevenLabs__text_to_speech` 就是一个 ElevenLabs 语音服务的 MCP 工具。
