# nyldn/claude-octopus — 学习案例

**仓库**：https://github.com/nyldn/claude-octopus
**Stars**：2575 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-06-23（基于当前 HEAD）
**主题标签**：multi-provider, persona-system, security-hooks, agent-routing, large-scale

**xiaolai 案例**：[../2026-04-24-nyldn-claude-octopus.md](../2026-04-24-nyldn-claude-octopus.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-octopus 是 Chris S（nyldn）构建的 Claude Code 多模型并行路由插件。其核心思想是"八爪鱼式并行"：把一个编程任务同时分发给 Claude、Gemini、Codex、Ollama 等模型，汇总各方结果，让盲点在合并前就被发现。2575 星，266 forks，活跃发布节奏（截至 audit 后 48 小时已出 4 个版本）。

关键事实：
- 仓库包含 157 个 NL artifact，分为 9 类（GitHub Actions subagents、personas、droids、skill-router agents、principles、default subagents、commands、skills、config）
- 有 4 个 shell hook（codex-exec-guard、security-gate、quality-gate、post-compact）和一个 UserPromptSubmit hook（user-prompt-submit.sh + python3）
- 有一个自研 MCP server（`mcp-server/` 目录）
- OpenClaw 集成（`openclaw/` 目录）是主要安全隐患来源
- 工作流基于 Double Diamond 方法论，在 `config/workflows/CLAUDE.md` 中有完整文档

### 1.2 架构剖析

```
claude-octopus/
├── agents/
│   ├── personas/    # 32 个富角色 agent（85/100 avg）
│   ├── droids/      # 10 个任务型 agent（⚠️ 无 tools 声明）
│   ├── skills/      # 3 个 skill-router agent
│   └── principles/  # 4 个批判规则 agent
├── .claude/
│   ├── agents/      # 10 个默认 subagent（84/100 avg）
│   └── commands/    # 49 个 slash command（⚠️ 44/49 缺 allowed-tools）
├── skills/          # 29 个 SKILL.md（85/100 avg）
├── hooks/           # 5 个 hook 脚本
├── .github/agents/  # 10 个 CI subagent（63/100 avg，最弱）
├── config/
│   ├── providers/   # 每个 AI 提供商的配置文档
│   └── workflows/   # Double Diamond 工作流文档
├── openclaw/        # ⚠️ OpenClaw 集成（HIGH 安全发现来源）
└── mcp-server/      # 自研 MCP server
```

- **文件类型分布**：32 personas / 10 droids / 49 commands / 29 skills / 10 CI agents / 5 hooks / 4 principle agents
- **编排关系**：用户通过 slash command 触发，command 调用 `orchestrate.sh` 脚本分发给多模型，结果汇总返回。Hook 作为横切面，在工具调用前后拦截验证。
- **跨件契约**：flow skills（flow-discover/define/develop/deliver）实现了严格的 MANDATORY 执行合约，numbered steps + blocking gate + 验证要求。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：模型多样性对冲单模型盲点——"没有一个 AI 是全知的，但八个 AI 很难都犯同一个错"
- **解决什么问题**：单一 AI 在复杂代码审查中会产生系统性遗漏；并行多模型路由可以覆盖盲区
- **trade-off**：157 个 artifact 覆盖极广，但规模带来了跨文件一致性的维护难题（40% personas 缺 tools 声明，65% commands 缺 allowed-tools）
- **认知模型**：作者把 Claude Code 视为"总指挥"，把各 AI 模型视为"专业顾问"。skills 定义工作规约，droids 是执行单元，personas 是专业顾问的角色卡片。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「多维 Agent 分层 + 多提供商横向路由」

核心特征：
- 特征 1：Persona（角色）和 Droid（任务执行器）分离——角色定义"是谁"，任务定义"做什么"
- 特征 2：Security + Quality hook 在工具调用层拦截，不依赖 skill 内部实现
- 特征 3：Provider 配置文档化（`config/providers/`），每个模型的特性、接口、限制有单独文档
- 特征 4：UserPromptSubmit hook 做意图识别，自动把用户输入路由到正确的 skill
- 特征 5：Principle agents 作为独立的批判层——不执行任务，只做质量门控

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多模型交叉验证代码质量 | ✅ 核心用例 | 路由架构专为此设计 |
| 个人小项目快速迭代 | ❌ 不适用 | 157 个 artifact 维护成本高，overkill |
| 有 DevOps / Infra 专项需求 | ✅ 适用 | cloud-architect、devops-troubleshooter 等 persona 覆盖 |
| 需要精确的工具权限控制 | ⚠️ 谨慎 | 65% commands 缺 allowed-tools，权限边界模糊 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多维 Agent 分层（claude-octopus） | nyldn/claude-octopus | 覆盖面广，角色专业化；hooks 作为横切面 | 维护面大，一致性难保证 |
| 单层 skill 集合 | numman-ali/n-skills | 简单，每个 skill 独立，维护成本低 | 无角色系统，无横向路由能力 |
| 任务驱动 command 集合 | peterkrueck/Claude-Code-Development-Kit | 功能明确，每个 command 一件事 | 规模小，无多模型路由 |

### 2.4 改进空间

1. **当前问题**：65% commands 缺 `allowed-tools`，Claude Code 无法约束命令的工具访问面。**改进做法**：对每个 command 分析其 body 中实际使用的工具，在 frontmatter 加 `allowed-tools:`。**预期收益**：防止命令意外调用超出预期的工具，缩小攻击面。
2. **当前问题**：13 个 personas 缺 `tools:` 声明，作为 subagent 调用时静默降级为纯文本生成。**改进做法**：为 devops-troubleshooter、incident-responder 等明显需要 shell 的 persona 加 `tools: [Bash, Read, Grep]`。**预期收益**：消除"调用了但不能用工具"的隐性故障。
3. **当前问题**：`.github/agents/` 10 个 CI subagent 平均 63 分，是全仓最弱的一类。**改进做法**：每个 CI agent 加一个 3-5 行的 Examples section，说明在什么 CI 事件下如何使用。**预期收益**：CI/CD AI 工作流的输出质量从随机提升到可预期。

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-16 当时得分 **79/100**（Security: REVIEW，Bugs: 6，Quality Issues: 14，Security Findings: 6）。

| 类别 | 文件数 | 平均分 | 主要问题 |
|---|---|---|---|
| `.github/agents/` | 10 | 63 | 最小化模板，readonly 与工具需求冲突 |
| `agents/droids/` | 10 | 78 | 所有 droid 缺 tools 声明 |
| `.claude/commands/` | 49 | 74 | ~65% 缺 allowed-tools |
| `agents/personas/` | 32 | 85 | 40% 缺 tools 声明 |
| `skills/` | 29 | 85 | 强执行合约，示例充分 |

### 3.2 当时值得借鉴的模式

1. **MANDATORY 执行合约 flow skills**：`flow-discover/define/develop/deliver` 每个 skill 都有 numbered steps + "MANDATORY: STOP and verify before proceeding" 阻断点。把质量门控嵌入 skill 协议，而不是依赖 AI 自觉。
2. **Security + Quality 防御性 hook**：`codex-exec-guard.sh` 拦截裸 codex 调用，`security-gate.sh` 要求 OWASP 类别覆盖，`quality-gate.sh` 检查 reference 完整性——横切面安全设计的教科书案例。
3. **Principle agents 作为独立批判层**：`agents/principles/security.md`、`general.md` 等是纯批判角色，不执行任务。分离"做"和"评"。
4. **Provider 完整文档化**：`config/providers/` 每个模型有独立配置文档，覆盖 Codex / Gemini / Copilot / Ollama / OpenCode 的接口差异。
5. **Double Diamond 工作流**：在 `config/workflows/CLAUDE.md` 中完整记录方法论，让 Claude 和开发者共享相同的问题解决框架。

### 3.3 当时的缺陷

1. **40% personas 无 tools 声明**：`business-analyst`、`devops-troubleshooter` 等 persona 描述了需要 Bash 和 Read 的复杂工作流，但 frontmatter 没有 tools 字段。作为 subagent 调用时，Claude Code 继承父上下文所有工具（过度授权）或无工具（功能缺失），取决于调用方式。根本原因：32 个 persona 中只有约一半遵循"谁用工具、谁声明"的规范，另一半是遗漏。**我有没有犯？** bureau 的 auditor.md 有明确的 tools 声明，这个错我没犯。
2. **HIGH: curl-pipe-sh 双发现**：`openclaw-admin.md` 和 `skill-claw/SKILL.md` 各有一处 `curl ... | bash`。`openclaw-admin` 用的是错误域名 `gogcli.sh`（应为 `openclaw.ai`），更可疑。根本原因：安装便利性 vs 安全性的取舍中，这里完全没有验证步骤。**自查**：我的任何 skill 或脚本里有没有 curl-pipe-sh？检查结果是没有。
3. **65% commands 缺 allowed-tools**：`embrace`、`factory`、`pipeline` 等复杂的多提供商编排命令没有声明工具，Claude Code 无法限制其访问面。根本原因：规模膨胀时，新增 command 没有强制 frontmatter 检查的 CI 门控。**自查**：gstack 有 129 处 allowed-tools，但不是所有 skill 都有，这是潜在风险。

### 3.4 当时的优化机会

1. 给 droids 全部加 `tools:` 声明（BUG-002，10 个文件，每个只需加 2-4 行）
2. 修复 `openclaw-admin.md` 的域名 `gogcli.sh` → `openclaw.ai`（BUG-006 + FINDING-01）
3. 给 `hooks/user-prompt-submit.sh` 加 python3 不可用时的 fallback（BUG-005）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| openclaw-admin.md 错误域名 curl-pipe-sh | `grep -n "curl" agents/personas/openclaw-admin.md` | **部分修复**（域名从 `gogcli.sh` → `openclaw.ai`，但 curl-pipe-sh 模式仍在 line 112） | 域名 BUG 修复，安全模式未根治 |
| droids 全部缺 tools 声明 | `head -10 agents/droids/octo-backend-architect.md` | **仍存在**（frontmatter 仅 name/description/model，无 tools） | BUG-002 未修复，droids 仍是哑角色 |
| commands 缺 allowed-tools | `grep -rn "allowed-tools" .claude/commands/ \| wc -l` | **小幅改善**（5/49 有 allowed-tools，较 audit 时的 17/49 实际下降）→ 注意 audit 报告为"约 17 正确"，实测现在只有 5 | 整体覆盖率仍低，49 个命令中 44 个缺少声明 |

**xiaolai 案例衔接**：xiaolai 于 2026-04-24 记录了 v9.26.0 的修复（BUG-002 droid tools 修复、skill-extract beta 声明等），但当前 HEAD 的 droids 仍缺 tools 字段——可能是后续版本重构时的回退，也可能是 xiaolai 案例记录的是 v9.26.0 的短暂状态，之后 droid 目录经历了重写。这说明**快速迭代仓库的 re-audit 结果有时效性**。

### 4.2 架构演进

从 audit 时的 157 个 artifact，到当前 HEAD 仍有大量文件，但目录结构有演进：
- 新增 `mcp-server/` 目录（自研 MCP server，audit 时为空配置）
- 新增 `openclaw/` 目录（OpenClaw 集成）
- `managed-settings.d/` 目录出现（settings 分片管理）

这说明作者在向更深的平台集成发展——不仅是 skill 和 agent，还在构建自己的 MCP server。

### 4.3 新增的可学习模式

1. **managed-settings.d/ 分片 settings**：通过多个 settings 文件片段按需合并，而不是一个大 settings.json。适合多环境、多模型配置场景。
2. **自研 MCP server**：从依赖第三方 MCP → 自己实现，掌控更深的上下文传递能力。

---

## 五、校准

### 5.1 我已经在做对的

1. bureau 的 agent 严格声明 `tools:` 字段（Read, Grep, Glob），不过度授权
2. gstack 的 allowed-tools 覆盖面高（129 处），command 工具访问面基本受控
3. 我没有任何 curl-pipe-sh 模式

### 5.2 挑战 / 验证

这个案例颠覆了我一个假设：**规模越大的仓库质量管理越困难**。claude-octopus 的 157 个 artifact 里，越老越核心的 skills（29 个）质量最高（85/100），越新越边缘的 CI agents（10 个）质量最低（63/100）。这说明质量衰减有方向性——新扩展的边缘区域最先崩溃。我的项目如果扩展也会面临同样的模式。解法是：**在 CI 里加 NLPM threshold 检查，让新 artifact 不符合标准时阻断 merge**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 里有没有 tools 声明缺失
for f in $(find ~/.claude/agents -name "*.md" 2>/dev/null); do
  if ! grep -q "^tools:" "$f"; then
    echo "MISSING tools: $f"
  fi
done
```
命中后怎么办：查看 agent body 里实际用到了哪些工具，加对应的 `tools:` 声明。

```bash
# 检查我的 slash command 里有没有 allowed-tools 缺失
for f in $(find ~/.claude/commands -name "*.md" 2>/dev/null); do
  if ! grep -q "^allowed-tools:" "$f"; then
    echo "MISSING allowed-tools: $f"
  fi
done
```
命中后怎么办：分析命令需要的工具，加 `allowed-tools: Bash, Read, Grep` 等声明。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 claude-octopus 的 security-gate hook 模式，给 bureau 加一个 PostToolUse hook，在 Write 工具调用后验证写入文件是否在 canon/ 目录的权限范围内
   - **为何可行**：bureau 有严格的 canon 目录权限分层，hook 拦截是最合适的门控位置
   - **第一步**：在 bureau/hooks/ 新建 `canon-write-guard.sh`，检查写入路径是否在 `canon/` 下，是则允许，否则 block。约 20 分钟可完成。

2. **想法**：参考 claude-octopus 的 `agents/principles/` 批判层，给 gstack 加一个专门的 code-reviewer principle agent，不执行 review 只评判别人 review 的质量
   - **为何可行**：gstack 已有 review skill，缺的是一个独立于执行的批判层
   - **第一步**：在 gstack/agents/ 下新建 `principles/review-critic.md`，frontmatter 声明 `tools: Read`，body 说明如何批判 code review 报告。约 25 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 nyldn/claude-octopus 的核心目的**：多模型并行路由，用 agent 分层和 hook 防御网保证代码质量

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多 agent 工程工具套件，都有 hook | gstack 单模型中心化，claude-octopus 多模型分布 | 高 |
| MarkQWu/bureau | 中 | 都有 agent + hook 组合 | bureau 专注知识库，claude-octopus 专注代码 | 中 |
| MarkQWu/graphify | 低 | 都有 skill 体系 | graphify 是知识图谱工具，目的不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| personas 缺 tools 声明 | `grep -L "^tools:" /tmp/my-repos/MarkQWu-bureau/.claude/agents/*.md` | bureau/auditor.md 有 tools 声明 → 未命中 | N/A |
| commands 缺 allowed-tools | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md` | bureau/commands/ 下所有 md 需逐一检查 | 中 |
| CI agents 质量最低 | 检查 gstack 是否有 `.github/agents/` | gstack 无专属 CI agents 目录 → 未命中 | N/A |

**命中后的具体行动建议**：
- bureau/commands/ 里的 md 文件 → 逐一加 `allowed-tools:` frontmatter → 约 30 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：Principle agents 作为独立批判层
   - **本案例做法**：`agents/principles/security.md` 等是只"评"不"做"的 agent，让批判和执行完全解耦（`agents/principles/` 目录，4 个文件）
   - **我的项目现状**：gstack 的 review skill 和 code-reviewer 角色混在一起，既执行 review 又是 review 本身
   - **如何借鉴**：gstack 里把"review 执行"和"review 批判"拆成两个文件，一个做，一个评，让每次 review 都有内部质量检查

2. **领域**：PostToolUse security hook
   - **本案例做法**：`hooks/security-gate.sh` 在每次工具调用后检查 OWASP 类别覆盖，不在 skill 层做判断
   - **我的项目现状**：gstack 有类似的 hook 框架，但 bureau 没有任何 hook
   - **如何借鉴**：bureau 加一个 PostToolUse 的 canon 边界守卫 hook，参考上面 6.2 第 1 条

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：tools 声明的一致性
- **我的做法**：bureau/.claude/agents/auditor.md 有明确 `tools: Read, Grep, Glob` + `model: sonnet` 声明，比 claude-octopus 13 个缺 tools 的 persona 更规范
- **本案例做法**：40% personas 无 tools 声明，存在隐性授权问题
- **意义**：bureau 的 agent 声明规范性是可以作为 PR 示例提供给 claude-octopus 的参考模式

---

## 八、术语表

- **[Persona agent](#persona-agent)**：代表某种专业角色的 agent（如 backend-architect、security-auditor），描述"是谁"、"有什么专长"，通常需要 `tools:` 声明才能作为 subagent 正确工作。
- **[Droid agent](#droid-agent)**：claude-octopus 对任务执行型 agent 的称呼，代表"做什么"，对应特定工作流步骤。缺 tools 声明时只能生成文本。
- **[allowed-tools](#allowed-tools)**：Claude Code slash command 的 frontmatter 字段，限制命令可以调用的工具列表，是工具访问的权限边界。
- **[UserPromptSubmit hook](#userpromptsubmit-hook)**：Claude Code hook 事件类型，在用户提交 prompt 时触发，可在 prompt 到达 AI 之前拦截和注入内容。claude-octopus 用它做意图识别和自动 skill 路由。
- **[Double Diamond](#double-diamond)**：一种设计方法论，把问题分为"发现问题"和"解决问题"两个阶段，每个阶段各有"发散"（探索）和"收敛"（决策）步骤，形成两个钻石形状。
- **[MCP server](#mcp-server)**：Model Context Protocol server，Claude Code 可以通过 MCP 协议连接的外部能力提供方，扩展 AI 可用的工具集。
- **[OWASP](#owasp)**：开放 Web 应用安全项目，发布了常见的 Web 安全漏洞分类（OWASP Top 10），`security-gate.sh` 要求 code review 覆盖这些类别。
- **[curl-pipe-sh](#curl-pipe-sh)**：`curl URL | bash` 或 `curl URL | sh` 模式，下载远程脚本并直接执行，没有内容验证步骤。是 NL 安全审查的高风险模式（对应 SEC-curl-pipe-sh 规则）。
- **[frontmatter](#frontmatter)**：Markdown 文件顶部 `---` 包围的 YAML 块，Claude Code 用它识别 artifact 类型和声明权限（tools、allowed-tools、model 等）。
