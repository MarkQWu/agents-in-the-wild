# xiaolai/codex-toolkit-for-claude — 学习案例

**仓库**：https://github.com/xiaolai/codex-toolkit-for-claude
**Stars**：registry 未记录（xiaolai 为 NLPM 作者，本仓库是其生产级自用工具链）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-10（基于当前 HEAD）
**主题标签**：`cross-reference`, `template-design`, `security-gate`, `single-purpose`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[xiaolai/codex-toolkit-for-claude](https://github.com/xiaolai/codex-toolkit-for-claude) 是 NLPM 作者 xiaolai 为桥接 Claude Code 和 OpenAI Codex 而设计的工具包，功能与 `openai/codex-plugin-cc` 有重叠，但架构截然不同：30 个 NL 工件（20 个 command + 5 个 shared partial + 1 个 agent + 1 个 skill + 3 个 hook）构成一套完整的 Codex 编排系统，并内置了一套自审查工具（`/audit-skill`、`/audit-plugin`、`/audit-agent` 等）。

关键事实：
- 作者身份：xiaolai 同时是 NLPM 规则体系的制定者，这让这个仓库成为「规则作者」实际使用自己规则的活标本
- 规模：30 个 NL 工件，在单插件范围内属于大型项目；OpenAI 官方版本只有 13 个
- 自审查能力：内置的 `/audit-*` 命令系列可对自身进行 NL 质量检查，是「工具自检」的少见案例
- MCP 集成：通过 `.mcp.json` 注册 `codex mcp-server`，在 Claude Code 工具体系里又加了一层

### 1.2 架构剖析

```
xiaolai/codex-toolkit-for-claude/
├── .claude-plugin/plugin.json    ← manifest（100/100 满分）
├── .mcp.json                     ← MCP server 注册（codex mcp-server）
├── CLAUDE.md                     ← 项目文档（标注了约定外的例外）
├── commands/
│   ├── audit-skill.md            ← 审查 skill
│   ├── audit-plugin.md           ← 审查 plugin（例外：直接分析，不走 Codex）
│   ├── audit-agent.md            ← 审查 agent
│   ├── audit-command.md          ← 审查 command
│   ├── audit-rules.md            ← 审查规则
│   ├── audit-nlp.md              ← NLP 质量审查
│   ├── audit-fix.md              ← 审查后修复
│   ├── audit.md                  ← 审查入口（路由）
│   ├── implement.md              ← 实现功能
│   ├── verify.md                 ← 验证实现
│   ├── review-plan.md            ← 检视计划
│   ├── bug-analyze.md            ← 分析 bug
│   ├── status.md                 ← 查询任务状态
│   ├── cancel.md                 ← 取消任务
│   ├── result.md                 ← 查询结果
│   ├── setup.md                  ← 安装 Codex 运行时
│   ├── init.md                   ← 初始化项目
│   ├── preflight.md              ← 预检
│   ├── continue.md               ← 继续任务
│   ├── refresh-knowledge.md      ← 刷新 Codex 知识
│   └── shared/                   ← 共享 partial（5 个）
│       ├── codex-call.md         ← Codex 调用协议
│       ├── fallback.md           ← 降级路径
│       ├── model-selection.md    ← 模型选择逻辑
│       ├── plugin-discover.md    ← 插件发现扫描
│       └── scope-parse.md        ← 作用域解析
├── agents/cross-validator.md     ← 交叉验证 agent（CONFIRMED/DISPUTED/UNCERTAIN）
├── hooks/hooks.json              ← SessionStart + SessionEnd + Stop hooks
├── skills/codex-toolkit/
│   └── claude-code-conventions/SKILL.md  ← 约定知识库
├── schemas/audit-output.schema.json       ← 结构化输出约定
└── scripts/                      ← 脚本层（codex-runner.mjs 等）
```

- **文件类型分布**：20 个 command / 5 个 shared partial / 1 个 agent / 1 个 skill / 1 个 hook config / N 个脚本
- **编排关系**：分层明确——所有 command 注入 `shared/codex-call.md` partial，用 `commands/shared/model-selection.md` 选模型，用 `commands/shared/fallback.md` 处理降级。`audit-plugin.md` 是唯一例外（直接分析，不走 Codex）——而且在 CLAUDE.md 里有明确标注
- **跨件契约**：shared partial 被多个 command 引用（`codex-call.md` 被 19 个 command 使用），是整个系统的单一权威来源。`cross-validator.md` agent 引用 `codex-toolkit:claude-code-conventions` skill

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「所有 Codex 调用走统一入口 partial」——20 个 command 都通过 `shared/codex-call.md` 发 Codex 请求，这意味着 Codex 调用协议的变更只需改一个文件
- **解决什么问题**：如何让 20 个 command 共享相同的调用规范和降级逻辑，同时各自保持独立的业务语义
- **做了什么 trade-off**：
  - 不声明 `allowed-tools`（让 command 动态使用任意工具）vs. 最小权限——选择了前者，因为工作流复杂到无法静态枚举需要的工具
  - `audit-plugin` 绕过 Codex，直接用 Claude 分析——记录了这个例外而不是强制统一
  - 系统性没有 `allowed-tools`，但 audit 建议简单 command（cancel/result/status）可以加
- **反映什么认知模型**：作者把这套工具当作「持续维护的生产工具」，而不是一次性脚本。shared partial 的设计说明作者在写第一个 command 时就预见了第 20 个 command 的扩展需求

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「共享 Partial 中心化 + 单一例外明文标注」**：所有同类操作通过共享 partial 保证一致性；唯一的例外（`audit-plugin` 直接分析）在项目文档里明确标注，让「约定中的例外」对读者可见而不是隐藏的。

模式特征清单：
- 特征 1：5 个 shared partial 是整个系统的「协议层」——command 文件只写业务逻辑，不写调用细节
- 特征 2：CLAUDE.md 里的「Adding new commands」有 7 步流程，包括「Update README.md commands table」——这把维护成本写进了文档约定
- 特征 3：`audit-plugin.md` 的例外在 CLAUDE.md 里有标注，形成「例外清单」模式
- 特征 4：`cross-validator.md` agent 使用 CONFIRMED/DISPUTED/UNCERTAIN 三态而不是 true/false，体现「不确定性也是一等公民」的认知
- 特征 5：`schemas/audit-output.schema.json` 是结构化输出的 JSON Schema 约束，确保 agent 输出可机器解析

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多个 command 共享同一外部服务调用协议 | ✅ 高度适用 | shared partial 天然解决「协议统一」问题 |
| 工具自检 / 自审查场景 | ✅ 适用 | 内置 audit 命令系列可直接复用 |
| 小型单一功能插件（1-3 个 command） | ❌ 不适用 | shared partial 的成本在 command 少时是过度工程 |
| 需要最小 `allowed-tools` 约束的高安全环境 | ⚠️ 谨慎 | 系统性没有 allowed-tools 是已知缺陷 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 共享 Partial 中心化（本仓库） | xiaolai/codex-toolkit | 协议统一，变更只改一处 | 5 个 partial 增加了认知负担 |
| 每 command 自包含 | openai/codex-plugin-cc | 简单，每文件独立理解 | 3 个 command 有同一 bug（各自复制导致一致性失守） |
| 纯 Skill 套件 | slavingia/skills | 最简单，无 command 层 | 无法编排多步任务 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 的「Adding new commands」第 7 步要求更新 README.md 命令表——这创建了一个「隐式外部依赖」，plugin 工件集不跟踪 README 的状态。**改进做法**：把命令表移入 `plugin.json` 的 `commands` 字段（命令描述），由 manifest 作为权威而非 README。**预期收益**：README 可从 manifest 自动生成，不再需要人工同步
2. **当前问题**：`cross-validator.md` 里「If uncertain, say so」没有定义 UNCERTAIN 的边界——什么情况下选 UNCERTAIN 而不是 DISPUTED？**改进做法**：添加「UNCERTAIN 标准：无法在仓库里找到对应文件验证，且问题不可通过现有工具确认」这样的具体判据。**预期收益**：输出三态的分布更可预测，机器处理时误分类减少

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **96/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/refresh-knowledge.md` | 93 | 无 allowed-tools；「relevant」模糊量词 |
| 20 个 command（除 refresh-knowledge） | 95 | 无 allowed-tools（每个 -5） |
| `hooks/hooks.json` | 95 | 顶层 `description` 字段不在 schema 里 |
| `CLAUDE.md` | 95 | 结构性小问题 |
| 5 个 shared partial | 97-98 | 轻微主观表述 |
| `agents/cross-validator.md` | 97 | UNCERTAIN 无边界定义 |
| `.claude-plugin/plugin.json` | 100 | 完美 |

### 3.2 当时值得借鉴的模式

1. **所有 20 个 command 引用同一个 `codex-call.md` partial** → 协议变更只改一处。这是「DRY 原则在 NL 工件里的最纯粹实现」。**借鉴**：如果有 3 个以上 command 共享同一调用方式，提取 shared partial

2. **`model-selection.md` partial 把模型选择逻辑外化** → command 文件不写「用哪个模型」，全部委托给 partial。**借鉴**：模型选择逻辑写一次，其他 command 注入

3. **`cross-validator.md` 的三态输出（CONFIRMED/DISPUTED/UNCERTAIN）** → 对比「是/否」的二值输出，三态更接近真实认知——「不确定」本身也是有价值的信号。**借鉴**：agent 的验证输出考虑加 UNCERTAIN 或 MAYBE 状态

4. **`audit-plugin.md` 的例外在 CLAUDE.md 有标注** → 例外不可避免，但必须写进文档才是受控的例外。**借鉴**：对于「有意识绕过架构规范」的文件，在 CLAUDE.md 里加注解

5. **`schemas/audit-output.schema.json` 强制输出结构** → 结合 agent 的 `schema:` 字段，输出是机器可验证的 JSON，不是自由文本。**借鉴**：需要下游解析的 agent 输出，配套 JSON Schema

### 3.3 当时的缺陷

1. **系统性缺少 `allowed-tools` 声明** → **根本原因**：20 个 command 覆盖的工具集太动态（视任务类型在 AskUserQuestion、Read、Bash、MCP、Task 之间变化），无法静态枚举。作者选择了「不限制」。**根本问题**：这让最小权限原则失效——`cancel` 这样的简单命令理论上可以调用任意工具。**自查**：我的 bureau `commands/` 同样没有 allowed-tools

2. **`commands/status.md` 的 `$ARGUMENTS` 未引号** → **根本原因**：grep 输出通过 `echo $ARGUMENTS | grep -o '--all'` 限制了最终值，作者认为是安全的。但原始的 `echo $ARGUMENTS` 仍然在进行 word splitting 和 glob 展开，是已知的 shell 行为。**自查**：我的 command 文件里有 `!` 命令行模式吗？→ 需要检查

3. **`refresh-knowledge.md` 的「relevant」模糊量词** → **根本原因**：`relevant command files` 没有定义「相关性」的判据。Claude 只能靠推断，每次执行行为可能不同。**自查**：我的 skill 里有没有「relevant」未加定义？→ 高频出现的问题

### 3.4 当时的优化机会（学习材料，不用于 PR）

1. `status.md` 的 `echo $ARGUMENTS` 改为 `echo "$ARGUMENTS"`（安全修复）——已修复（见 §4.1）
2. 给 `cancel`/`result`/`status`/`preflight` 这四个工具集稳定的简单 command 加 `allowed-tools`
3. 把 skill description 里「canonical reference for...」改为「Use when auditing Claude Code plugins」

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `status.md` 中未引号 `$ARGUMENTS` | `grep -n "ARGUMENTS" commands/status.md` | **已修复**（`echo "$ARGUMENTS"`） | xiaolai 自己接受了这个安全修复，时间线不明 |
| 系统性无 `allowed-tools` | `grep -rL "allowed-tools" commands/*.md` | **仍存在**（所有 command 无声明） | 这是架构决策，20 个 command 全都没有 |
| `refresh-knowledge.md` 的「relevant」 | `grep -n "relevant" commands/refresh-knowledge.md` | 仍存在（line 136） | 未被视为优先修复 |

### 4.2 架构演进

当前 HEAD 与 audit 时的差异：

1. `commands/status.md` 第 24 行由 `echo $ARGUMENTS` 变为 `echo "$ARGUMENTS"`——这是安全修复，且是在 NLPM audit 后发生的，可能是 audit 触发的改动
2. 整体架构无重大变化，30 个工件数量未变，shared partial 体系保持完整
3. 新增了 `nlpm-badge.json`（NLPM 徽章文件，记录 audit 分数，是 NLPM v0.9+ 功能）

### 4.3 新增的可学习模式

**`nlpm-badge.json`**——NLPM 的新功能，让仓库自己宣告「已通过 NLPM audit，分数 96/100」。这是「审查即标准」的具体化：外部工具的审查结果可以成为仓库的元数据，供安装者判断质量。

---

## 五、校准

### 5.1 我已经在做对的

1. **Plugin.json 完整性**：我的 bureau 和 echo-sleuth 的 plugin.json 都有完整的注册，与 xiaolai 的 100/100 manifest 实践一致
2. **Shared 知识外化**：我的 echo-sleuth 用 `references/` 子目录存放不变的知识（record-types、extraction-patterns），与 xiaolai 的 shared partial 哲学一致
3. **例外要在文档里标注**：我的 bureau CLAUDE.md 里有「Known limitations」section，标注了有意的设计决策

### 5.2 挑战 / 验证

这次案例挑战了我对「allowed-tools 是必须的」的绝对化认知：xiaolai（NLPM 规则的制定者！）在自己的生产工具里也没有给 20 个 command 加 allowed-tools，因为工具集太动态。这说明「最小权限」不是一刀切的规则——**工具集静态可枚举时用 allowed-tools，动态时可以不用，但要意识到这是一个已知的 trade-off，而不是遗漏**。

同时验证了「shared partial 的价值在大型 command 集」——我的 bureau 只有 8 个 command，引入 shared partial 可能过度。但如果增长到 15+，就该考虑了。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查是否有类似 "relevant" 这样的模糊引用词
grep -rn '\brelevant\b' commands/*.md skills/*.md 2>/dev/null
# 命中后怎么办：把「relevant」替换为「所有名字以 audit-*.md 结尾的 command 文件」这样的具体描述

# 检查 command 里是否有 "relevant" 以外的横向引用词
grep -rn '\b(related|applicable|appropriate|corresponding)\b' commands/*.md 2>/dev/null
# 命中后怎么办：在每个词旁边加具体的枚举或查找条件
```

```bash
# 检查 agent 输出是否有配套的 schema
find agents/ -name "*.md" | while read f; do
  echo -n "$f: "
  grep "schema\|Schema" "$f" | head -1 || echo "no schema reference"
done
# 命中后怎么办：为需要下游解析的 agent 添加 JSON Schema 约束
```

### 6.2 灵感 → 实施路径

1. **想法**：把 bureau 的 `commands/` 里重复的「invoke via Bash」调用模式提取为 `commands/shared/bureau-run.md` partial
   - **为何可行**：bureau 有 3 个 command 都通过 bash 调用同一个脚本，这是可以提取的重复
   - **第一步**：找出 3 个共享调用的 command，提取公共部分到 `commands/shared/bureau-run.md`，更新引用，30 分钟

2. **想法**：给 echo-sleuth 的 `cross-validator`（如果有的话）或新增 agent 添加三态输出（CONFIRMED/PLAUSIBLE/UNCERTAIN）
   - **为何可行**：xiaolai 的 CONFIRMED/DISPUTED/UNCERTAIN 三态证明这不是「设计过度」，而是合理的不确定性建模
   - **第一步**：在 `echo-sleuth/agents/memory-auditor.md` 的 Output Format 里把 true/false 改为三态，配套更新下游处理逻辑，15 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 xiaolai/codex-toolkit-for-claude 的核心目的**：以 shared partial 为中心，为 20 个 Codex 编排 command 提供统一的调用协议和降级路径

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是多 command 套件；都有 skill 作为知识层 | bureau 8 个 command vs. 20 个；bureau 无 shared partial 层 | 高 |
| MarkQWu/graphify | 中 | 都有复杂工作流编排 | graphify 是代码知识图谱，不是任务编排 | 中 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 agent 层；都使用 schema 约束输出 | echo-sleuth 是分析类而非执行类 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 无 allowed-tools（系统性） | `grep -rL "allowed-tools" commands/*.md` | **bureau 全部 8 个 command 命中** | 中（接受同样的 trade-off：工具集动态） |
| 「relevant」等模糊横向引用 | `grep -rn "relevant\|related" commands/*.md skills/*.md` | **bureau/skills/recall/SKILL.md** line 34: "recall relevant past decisions" | 低 |
| agent 无三态输出 | `grep -rn "CONFIRMED\|DISPUTED\|UNCERTAIN" agents/*.md` | **echo-sleuth/agents/** 全无三态，只有 "found/not found" | 低（影响下游处理精度） |

**命中后的具体行动建议**：
- `MarkQWu/bureau/skills/recall/SKILL.md` line 34 → 「relevant past decisions」改为「decisions made in the current project (grep DECISION keyword in BUREAU.md)」→ 5 分钟
- `MarkQWu/echo-sleuth-for-claude/agents/memory-auditor.md` → Output Format 加三态定义 → 10 分钟

### 7.3 别人的更优方案

1. **领域**：Shared partial 消除 command 间的协议重复
   - **本案例做法**：`commands/shared/codex-call.md` 被 19 个 command 注入，任何协议变更只改一个文件
   - **我的项目现状**：`bureau/commands/` 里的 3 个 command 都有「invoke the bureau script」这样的描述，各自写了一遍
   - **如何借鉴**：提取 `bureau/commands/shared/bureau-run.md`，把「调用 bureau CLI 的协议」集中管理

2. **领域**：结构化输出 + JSON Schema 约束
   - **本案例做法**：`schemas/audit-output.schema.json` 是 agent 输出的机器可验证约束；`cross-validator` 的 `schema:` 字段引用这个文件
   - **我的项目现状**：`echo-sleuth/agents/` 的输出是自由文本，没有 schema 约束
   - **如何借鉴**：为 `memory-auditor` 创建 `schemas/memory-audit-output.schema.json`，定义三个字段（findings/confidence/next_steps）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：allowed-tools 的存在（对于简单 command）
- **我的做法**：虽然我也没有普遍声明 allowed-tools，但 `echo-sleuth` 的 `agents/file-historian.md` 只用 Read 和 Glob，如果加了 `allowed-tools: [Read, Glob]` 会更精确
- **本案例做法**：系统性没有声明，包括 `cancel`、`result` 这些只需要 Bash 的简单 command
- **意义**：我意识到了这个 trade-off，可以有选择地给简单 command 加限制，而不是一刀切全不加或全加。这比 xiaolai 的「全不加」更精细

---

## 八、术语表

### <a name="shared-partial"></a>共享 partial
> 在 NL 工件系统里，「partial」指被多个文件引用的「片段文件」——不是独立可执行的 skill 或 command，而是被其他文件注入的公共片段。`commands/shared/codex-call.md` 就是一个 partial：它描述「如何调用 Codex」，被 19 个 command 注入，确保每个 command 用相同的协议调用 Codex。

### <a name="DRY-原则"></a>DRY 原则
> "Don't Repeat Yourself"（不要重复你自己）——软件工程的基本原则：同一段逻辑只写一次，通过引用复用，而不是复制粘贴多份。NL 工件里，shared partial 就是 DRY 原则的实现：协议写一次，20 个 command 引用，变更只改一处。

### <a name="MCP-server"></a>MCP server
> Model Context Protocol server——Claude Code 的扩展机制，允许插件向 Claude 暴露自定义工具（不是 NL 工件，而是真实的可执行工具）。`xiaolai/codex-toolkit` 通过 `.mcp.json` 注册了 `codex mcp-server`，让 Claude 能用 MCP 工具直接调用 Codex 二进制，而不只是通过 bash 脚本。

### <a name="word-splitting"></a>word splitting
> Shell 的一种默认行为：对于未加引号的变量 `$VAR`，Shell 会按空格/tab/换行分割成多个参数。例如 `$ARGUMENTS` 等于 `task-123; malicious-cmd` 时，未加引号会导致 Shell 把 `;` 解析为命令分隔符，执行 `malicious-cmd`。加引号 `"$ARGUMENTS"` 后整个字符串作为一个参数传递，Shell 不做分割。
