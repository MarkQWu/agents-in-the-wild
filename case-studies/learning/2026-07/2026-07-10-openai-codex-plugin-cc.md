# openai/codex-plugin-cc — 学习案例

**仓库**：https://github.com/openai/codex-plugin-cc
**Stars**：registry 未记录（OpenAI 官方仓库，生产级使用）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-10（基于当前 HEAD）
**主题标签**：`security-gate`, `model-pinning`, `vague-quantifier`, `cross-reference`, `fallback-chain`

**xiaolai 案例**：[../2026-04-07-openai-codex-plugin-cc.md](../2026-04-07-openai-codex-plugin-cc.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc) 是 OpenAI 官方发布的 Claude Code 插件，将 OpenAI Codex（独立的 agentic 编程助手）集成进 Claude Code 工作流。用户安装后可通过 `/codex:review`、`/codex:adversarial-review`、`/codex:rescue` 等命令把 Claude Code 中的任务委托给 Codex 执行，或在 Codex 卡住时发起援救。

关键事实：
- 维护者：Dominik Kundel（OpenAI devrel）及至少一位 OpenAI 工程师 reviewers
- 架构特点：Claude Code 与 OpenAI Codex 的跨系统桥接——Claude 是用户入口，Codex 是执行后端
- 自 audit 以来新增：`transfer.md` command（将 Claude Code session 转为可恢复的 Codex thread）
- 经 xiaolai 手动案例记录，两个 PR 被 OpenAI 工程师 39 小时内双人 review 合并

### 1.2 架构剖析

```
openai/codex-plugin-cc/
├── plugins/codex/
│   ├── .claude-plugin/plugin.json    ← manifest（100/100，全满分）
│   ├── agents/codex-rescue.md        ← 援救代理（当 Codex 卡住时）
│   ├── commands/
│   │   ├── adversarial-review.md     ← 对抗性代码审查
│   │   ├── cancel.md                 ← 取消 Codex 任务
│   │   ├── rescue.md                 ← 发起援救路由
│   │   ├── result.md                 ← 查询任务结果
│   │   ├── review.md                 ← 标准代码审查
│   │   ├── setup.md                  ← 安装 Codex 运行时
│   │   ├── status.md                 ← 查询任务状态
│   │   └── transfer.md               ← 【新增】转移 session 到 Codex thread
│   ├── hooks/hooks.json              ← SessionStart/Stop hook 配置
│   ├── prompts/                      ← 外部 prompt 模板（不是 NL 工件）
│   ├── schemas/review-output.schema.json ← 结构化输出 schema
│   └── skills/
│       ├── codex-cli-runtime/SKILL.md
│       ├── codex-result-handling/SKILL.md
│       └── gpt-5-4-prompting/SKILL.md
│           └── references/           ← 3 个参考文档
├── package.json                      ← Node.js 运行时依赖
└── .claude-plugin/marketplace.json
```

- **文件类型分布**：1 个 agent / 8 个 command（+1 新增 transfer）/ 3 个 skill / 1 个 hook config / 若干脚本
- **编排关系**：commands 是用户入口 → `rescue.md` 路由到 `codex:codex-rescue` agent → agent 加载 `codex-cli-runtime` + `gpt-5-4-prompting` skills。3 层调用链：command → agent → skills
- **跨件契约**：`gpt-5-4-prompting/SKILL.md` 引用 3 个 `references/` 子文档；`codex-rescue` agent 声明 `skills: [codex-cli-runtime, gpt-5-4-prompting]`；`hooks.json` 引用 `scripts/session-lifecycle-hook.mjs` 和 `scripts/stop-review-gate-hook.mjs`——所有引用在 audit 时和现在均可解析

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「Claude 是前端，Codex 是后端」——Claude Code 提供用户接口和编排逻辑，OpenAI Codex 提供 agentic 编程执行能力，两个 AI 系统通过 Node.js 脚本层桥接
- **解决什么问题**：Claude Code 和 Codex 各有擅长，但用户需要在两者之间手动切换。此插件让用户不切换界面，在 Claude Code 中直接发起 Codex 任务、监控结果、接管失败任务
- **做了什么 trade-off**：
  - 引入 Node.js 运行时依赖，安装复杂度上升（`package.json`）
  - 用 [OIDC token](#OIDC-token) / 环境变量管理两套 AI 服务的认证，安全面扩大
  - `codex-rescue` agent 同时记录运行时知识（与 `codex-cli-runtime` skill 存在内容重叠），变更时需双处维护
- **反映什么认知模型**：OpenAI 将 Codex 定位为「可被外部系统控制的 agentic 工作单元」，插件就是这套控制协议的 Claude Code 实现

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「跨 AI 系统桥接 + Command-Agent-Skill 三层」**：Command 是用户界面层（描述意图），Agent 是路由执行层（处理异常、分发任务），Skill 是知识编码层（runtime 约定、prompt 工程），最底层是 Node.js 脚本（实际进程控制）。

模式特征清单：
- 特征 1：Command 文件只描述「用户能做什么」，不包含实现细节——实现委托给 agent 或直接调用脚本
- 特征 2：Agent 文件是「异常处理器」——`codex-rescue` 专门处理 Codex 卡住的情况，不处理正常路径
- 特征 3：Skills 是「知识 API」——`gpt-5-4-prompting` skill 把 GPT-5/4 的 prompt 工程技巧外化为可注入的知识
- 特征 4：`rescue.md` command 和 `codex-rescue` agent 的关系：command 描述「何时触发」，agent 描述「如何执行」——职责不重叠
- 特征 5：外部 prompt 模板（`prompts/adversarial-review.md`）通过 `loadPromptTemplate` 注入，把长 prompt 移出 NL 工件到版本化文本文件

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要调用外部 AI 服务 / 工具的集成插件 | ✅ 高度适用 | 三层架构天然适合多系统编排 |
| 异常恢复、超时处理等边界情况丰富的场景 | ✅ 高度适用 | Agent 层专门处理异常路径，与正常路径隔离 |
| 纯文本知识分享类 skill 套件 | ❌ 不适用 | 引入 Node.js 运行时是过度工程 |
| 需要最小安全攻击面的环境 | ⚠️ 谨慎 | hooks 脚本 + shell 命令 + CLAUDE_ENV_FILE 写入增加安全面 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Command+Agent+Skill+Script 四层 | openai/codex-plugin-cc | 职责清晰，异常路径独立处理 | 维护面宽，skill/agent 知识重叠风险 |
| Command+Skill 两层 | slavingia/skills | 简单，零运行时依赖 | 无 agent，无法处理复杂编排 |
| Command+多 Agent+CoreSkill | xiaolai/grill-for-claude | 并行分析，适合审查 | 仅适合批评类任务，不适合执行类 |

### 2.4 改进空间

1. **当前问题**：`codex-rescue.md` agent 和 `codex-cli-runtime/SKILL.md` 存在知识重叠（runtime flags、`--resume`/`--fresh` 语义）。**改进做法**：agent 的「技术知识段落」完全移除，仅保留路由逻辑，知识靠 skill 注入。**预期收益**：两处维护变一处
2. **当前问题**：`cancel.md`、`result.md` 无空输入处理（`argument-hint` 标注 `[job-id]` 可选但无回退行为描述）。**改进做法**：添加「若 `$ARGUMENTS` 为空，列出最近 5 个可操作的任务」的 fallback 分支。**预期收益**：用户不带参数执行命令时不会陷入无响应

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **93/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `agents/codex-rescue.md` | 80 | 无 model 声明；零示例 blocks |
| 6 个 command | 90 | 多步流程用无序列表；cancel/result 无空输入处理 |
| 3 个 skill | 98 | 单个模糊量词（「better」「actionable」） |
| `plugin.json` + `status.md` | 100 | 完美 |

### 3.2 当时值得借鉴的模式

1. **`plugin.json` 100/100 满分** → 完整的 manifest，所有 skill/command/agent 路径显式注册，无遗漏。**借鉴**：写完插件后，把每个 NL 工件路径都加进 plugin.json 的对应字段

2. **`gpt-5-4-prompting` skill 的三层参考文档** → 主 SKILL.md 是「调用方法」，`references/prompt-blocks.md` / `codex-prompt-recipes.md` / `codex-prompt-antipatterns.md` 是「细节知识库」。主文件轻，细节外化。**借鉴**：超过 200 行的 skill，把「食谱类内容」拆入 references/ 子目录

3. **安全一致性检查** → NLPM 发现 `review.md` 和 `adversarial-review.md` 正确引用 `"$ARGUMENTS"`，而 `cancel.md`/`result.md`/`status.md` 没有。这种「内部不一致」比「普遍缺失」更容易被察觉（有了对比基准）。**借鉴**：写 command 时找同类命令对照，而不是从零开始

4. **hook 配置引用真实存在的脚本** → `hooks.json` 的每个 hook 引用在仓库里都可解析。**借鉴**：写 hooks.json 后立即做 `cat scripts/*.mjs` 验证引用存在

### 3.3 当时的缺陷

1. **`cancel.md`/`result.md`/`status.md` 中的未引号 `$ARGUMENTS`** → **根本原因**：三个短 command 是后来添加的「快捷方式」，复制了长 command 的结构但漏掉了引号。内部不一致最难发现——开发者写新代码时参考了「错的文件」。**自查**：我的 bureau `commands/lint.md` 里有没有类似的 `$ARGUMENTS` 未引号？→ 需要检查

2. **`codex-rescue.md` 无 `model` 声明** → **根本原因**：救援 agent 是后期添加的，开发者可能以为「不声明就用默认」。但「救援」场景正是最需要稳定模型层的——成本和能力都不应该是未知数。**自查**：我的 echo-sleuth `agents/memory-auditor.md` 是否有 model 声明？→ 需要检查

3. **多步流程用无序列表而不是编号列表** → **根本原因**：Markdown 写 `-` 更快，但无序列表让「第三步必须在第二步之后执行」这个约束无法从格式层面体现。**自查**：我的 command 文件里的步骤是不是也用了 `-`？→ 常见问题

### 3.4 当时的优化机会（学习材料，不用于 PR）

1. `codex-rescue.md` 加 `model: sonnet`（当时 audit 建议，已被 OpenAI 接受并 merge）
2. 三个 `!` 命令的 `$ARGUMENTS` 加引号（已 merge）
3. 四个 command 的多步流程改为编号列表（作者未接受，归类为风格问题）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| 未引号 `$ARGUMENTS` | `grep -n "ARGUMENTS" commands/cancel.md` | **已修复**（`"$ARGUMENTS"` 有引号） | OpenAI 工程师 39h 内合并了 PR #168，三个文件同时修复 |
| `codex-rescue.md` 无 model | `grep "model:" agents/codex-rescue.md` | **已修复**（`model: sonnet`） | PR #169 同批次合并 |
| 多步流程用无序列表 | `head -30 commands/adversarial-review.md` | **仍存在** | 作者认为这是风格选择而非 bug，外部 PR 未被接受 |

### 4.2 架构演进

自 audit 至今新增了 `commands/transfer.md`。分析：

```
transfer.md 特征：
- disable-model-invocation: true   ← 不启动 Claude 模型，纯脚本执行
- allowed-tools: Bash(node:*)      ← 最小权限声明
- 单行实现：!`node ".../codex-companion.mjs" transfer "$ARGUMENTS"`
```

这是 audit 后对话模式演进的证据：作者发现用户需要在两个 AI 系统之间「带着上下文转移」，而不只是「在 Claude 里触发 Codex」。`disable-model-invocation: true` 是新模式——命令完全绕过 Claude 的生成，直接执行脚本，这在 audit 时的其他 command 里没有出现。

新的可学习模式：**`disable-model-invocation: true` + `allowed-tools` = 纯脚本执行命令**。适合「不需要 Claude 推理、只需要执行预定操作」的 command。

### 4.3 新增的可学习模式

`transfer.md` 引入了「最小权限 + 禁用模型调用」的 command 模式：
```yaml
---
disable-model-invocation: true
allowed-tools: Bash(node:*)
---
```
这在 audit 时不存在，是架构在实际使用中自然演进出来的实用模式。**结论**：不是所有 command 都需要 Claude 的语言理解能力；确定性的任务用 `disable-model-invocation: true` 更快、更可预测。

---

## 五、校准

### 5.1 我已经在做对的

1. **`$ARGUMENTS` 引号**：我的 bureau `commands/` 不使用 `!` 命令行模式，所以没有这个风险
2. **跨件引用可解析**：我的 echo-sleuth `agents/` 声明的 skills 都存在于仓库中
3. **Plugin.json 完整性**：bureau 和 echo-sleuth 的 plugin.json 都列出了实际存在的文件

### 5.2 挑战 / 验证

这次案例强化了一个我之前模糊的认知：**「agent 应该专门处理异常路径，正常路径由 command 直接处理」**。我之前的 echo-sleuth 里 `memory-auditor` agent 同时处理正常分析和异常情况，职责混杂。分离「正常路径 command」和「异常恢复 agent」是更清晰的分工。

也验证了**「内部不一致是最危险的缺陷」**：OpenAI 工程师自己没有发现三个 command 缺少引号，但 NLPM 通过「对比同类文件」发现了。这是 NL linting 比人工 review 更可靠的典型场景。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 command 里的 $ARGUMENTS 是否有引号
grep -rn '\$ARGUMENTS[^"]' commands/*.md
# 也检查单个双引号的情况
grep -rn 'ARGUMENTS[^"]' commands/*.md | grep -v '"$ARGUMENTS"'
# 命中后怎么办：将 $ARGUMENTS 改为 "$ARGUMENTS"（加双引号）
```

```bash
# 检查 agent 是否有 model 声明
grep -rL "^model:" agents/*.md 2>/dev/null
# 命中后怎么办：在 frontmatter 里加 "model: sonnet"（或 haiku/opus 视任务复杂度）
```

```bash
# 检查多步流程是否用了编号列表
# 找出「步骤」类关键词后紧接 - 而不是 1. 的 command
grep -l "Step\|step\|步骤\|阶段" commands/*.md | xargs grep -l "^- " | head -5
# 命中后怎么办：把连续的 - 改为 1. 2. 3.，特别是有依赖顺序的步骤
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 `commands/lint.md` 加 `disable-model-invocation: true` + `allowed-tools` 限制（如果 lint 是纯脚本执行的话）
   - **为何可行**：`transfer.md` 证明「纯脚本 command」是 OpenAI 生产级实践；减少不必要的模型调用可降成本
   - **第一步**：检查 `bureau/commands/lint.md` 是否有非脚本步骤，若无则加上 `disable-model-invocation: true`，10 分钟

2. **想法**：给 echo-sleuth 的 `agents/memory-auditor.md` 加 `model: haiku` 声明
   - **为何可行**：audit 报告证明缺少 model 声明是「成本和能力未知数」；memory 审计是相对简单的分类任务，haiku 足够
   - **第一步**：在 `agents/memory-auditor.md` frontmatter 加 `model: haiku`，2 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 openai/codex-plugin-cc 的核心目的**：在 Claude Code 中桥接外部 AI 服务（Codex），并提供完整的任务生命周期管理（启动、监控、取消、救援）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都有 command + skill 层；都需要管理「任务状态」 | bureau 是知识管理，不涉及外部 AI 调用 | 中 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 agent 层；都处理复杂工作流 | echo-sleuth 是单系统分析，无跨 AI 桥接 | 中 |
| MarkQWu/graphify | 低 | 都是工具型插件 | graphify 是代码知识图谱，无 agent 编排 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 缺少 model 声明 | `grep -rL "^model:" agents/*.md` | **echo-sleuth/agents/memory-auditor.md** 和 **file-historian.md** 均无 model 声明 | 高（不知道用什么模型跑任务） |
| 多步流程用无序列表 | `grep -rn "^- " commands/*.md` in bureau | **bureau/commands/status.md**：4 个步骤全用 `- ` 无序列表 | 低（影响可读性，不影响功能） |
| 未声明 allowed-tools | `grep -rL "allowed-tools" commands/*.md` in bureau | bureau 的 7 个 command 文件全部没有 allowed-tools 声明 | 中（遵循 openai 的「动态 command 可以不限」原则则可接受） |

**命中后的具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude/agents/memory-auditor.md` → frontmatter 加 `model: haiku` → 2 分钟
- `MarkQWu/echo-sleuth-for-claude/agents/file-historian.md` → frontmatter 加 `model: haiku` → 2 分钟
- `MarkQWu/bureau/commands/status.md` → 4 个 `- ` 步骤改为 `1.`/`2.`/`3.`/`4.` → 3 分钟

### 7.3 别人的更优方案

1. **领域**：`references/` 子目录用于 skill 知识分层
   - **本案例做法**：`gpt-5-4-prompting/SKILL.md` 主文件描述用法，`references/prompt-blocks.md`、`references/codex-prompt-recipes.md`、`references/codex-prompt-antipatterns.md` 存放细节食谱——主文件精简，细节外化，可独立维护
   - **我的项目现状**：`graphify/graphify/skill.md` 是全部内容写在主文件里，文件超长（结构较臃肿）
   - **如何借鉴**：把 graphify skill 里的「extraction patterns」部分拆出为 `references/extraction-patterns.md`，主文件只保留调用方法和示例

2. **领域**：`disable-model-invocation: true` 用于确定性脚本命令
   - **本案例做法**：`transfer.md` 完全绕过 Claude 模型，节省推理成本，提高确定性
   - **我的项目现状**：bureau 的 `serve.md` 命令只需启动本地服务，没有必要经过模型推理
   - **如何借鉴**：给 `bureau/commands/serve.md` 加 `disable-model-invocation: true`

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：`## Examples` section 覆盖率
- **我的做法**：`echo-sleuth/skills/git-mining/SKILL.md` 包含具体的 git log 输出示例和解析说明，让 Claude 知道期望的输入格式和输出结构
- **本案例做法**：`codex-cli-runtime/SKILL.md` 和 `codex-result-handling/SKILL.md` 在 audit 时无示例（codex-rescue agent 因此被扣 15 分）；现在仍未确认是否添加
- **意义**：examples 是 skill 精确性的保险——能防止「看起来对但行为错」的情况。我坚持写示例是正确的

---

## 八、术语表

### <a name="OIDC-token"></a>OIDC token
> 一种短期身份证。GitHub Actions 在运行 workflow 时可以用 OIDC token 向 Anthropic / OpenAI 等服务证明「我是一个合法的 GitHub Actions 任务」，避免直接把长期 API key 放在仓库里。OIDC 的全称是 OpenID Connect，是业界标准的身份验证协议。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools`、`disable-model-invocation` 等）。Claude Code 读这个配置来决定如何注册和执行这个文件。

### <a name="原生二进制核心"></a>原生二进制核心
> 在此插件的语境里，指的是 OpenAI Codex 本身——一个独立安装的命令行工具（非 Web API），通过 `codex` 命令在本地执行。Claude Code 插件通过 Node.js 脚本（`codex-companion.mjs`）调用这个本地二进制，而不是直接发 HTTP 请求。

### <a name="agentic"></a>agentic
> 指 AI 系统能够「自主规划和执行多步任务」的能力，而不是只回答问题。Codex 是 agentic 的——它能读代码、写代码、运行测试、自动修复，而不需要用户逐步指导。Claude Code 也是 agentic 的。这个插件的核心是把两个 agentic 系统串联起来。
