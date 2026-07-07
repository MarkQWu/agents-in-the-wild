# trailofbits/skills — 学习案例

**仓库**：https://github.com/trailofbits/skills
**Stars**：未收录（upstream 池，已通过 500+ stars 发现筛选）| **来源**：upstream（SECURITY BLOCKED，高危未修）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-07（基于当前 HEAD）
**主题标签**：`security-gate`, `router-channels`, `examples-driven`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[Trail of Bits](https://www.trailofbits.com/) 是顶级安全研究公司（专注于区块链安全审计、逆向工程、漏洞研究），其 `skills` 仓库是迄今为止审计过的规模最大、最专业的 Claude Code skill 集合之一——**151 个 NL [工件](#工件)（artifacts）、40 个独立 plugin**。覆盖从 fuzzing（模糊测试）、YARA 规则编写到 Semgrep 规则创建，再到区块链智能合约漏洞扫描（Cairo、Solana、Substrate 等），是安全领域 AI 辅助工具集的标杆。

关键事实：
- 作者：Trail of Bits 工程团队（安全公司）
- 规模：40 个 plugin，151 个 artifact（skills + agents + commands + hooks）
- 领域：安全审计、fuzzing、代码分析、区块链智能合约、devcontainer 配置
- 安全状态：**BLOCKED**——4 个 HIGH 安全发现，需私下披露才能提 PR

### 1.2 架构剖析
- **目录结构**：
  ```
  plugins/
    zeroize-audit/          ← 8 个 agent + 1 个 skill + 工具脚本
    fp-check/               ← 5 个 agent + 1 个 skill + hook
    dimensional-analysis/   ← 5 个 agent + 1 个 skill
    building-secure-contracts/ ← 8 个 skill（多链）
    testing-handbook-skills/ ← 15 个 skill（fuzzing 全家桶）
    trailmark/              ← 6 个 skill（协议建模）
    gh-cli/                 ← hook + 5 个脚本
    skill-improver/         ← command + agent + skill
    ...（共 40 个 plugin）
  .codex/                   ← Codex 平台专用 skill
  ```
- **文件类型分布**：80 个 skill + 23 个 agent + 8 个 command + 4 个 hook + 27 个工具脚本（zeroize-audit）+ 5 个 hook 脚本（gh-cli）
- **编排关系**：最复杂的是 `zeroize-audit` plugin——8 个 agent 形成有序流水线（0-preflight → 1-mcp-resolver → 2-source-analyzer → 2b-rust-source-analyzer → 3-tu-compiler-analyzer → ... → 6-test-generator），数字前缀显式编码了执行顺序。`fp-check` 也是 5 agent 流水线。
- **跨件契约**：`skill-improver` 命令分发给 `skill-improver` agent，该 agent 又加载 `skill-improver` skill 作为参考材料。`dimensional-analysis` 的 5 个 agent 共享同一 skill 文件定义的数学规则。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「安全工具就是工作流的可执行化」——每个 plugin 对应一个安全分析工作流，skill 是知识库，agent 是流水线节点，command 是人机接口
- **解决什么问题**：将 Trail of Bits 团队积累多年的安全分析方法论（如 zeroize audit、fp-check、dimensional analysis）编码为 AI 可执行的流程，降低安全审计的人力成本
- **做了什么 trade-off**：功能强大（40 个 plugin、151 个 artifact）vs 维护复杂度和安全风险面广（4 个 HIGH 安全发现）；为了集成 GitHub、Devcontainer 等外部工具，引入了 PATH 注入、sudo 操作等高风险面
- **反映什么认知模型**：Trail of Bits 把 Claude Code 视为「安全审计的 AI 副手」——不是简单的问答工具，而是能执行工作流步骤（调用 Semgrep、Burp Suite、Rust 编译器、YARA 等专业工具）的 AI agent

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「有序编号 Agent 流水线」模式**：用 `0-preflight → 1-resolver → 2-analyzer → ... → 6-test-generator` 的命名约定显式编码 agent 的执行顺序，在文件系统层面就能看到流水线结构。

模式特征清单：
- **命名即文档**：`0-`, `1-`, `2-` 前缀让任何人看文件名就知道执行顺序，无需单独维护流程图
- **专职 agent**：每个 agent 只做一步（mcp-resolver 只负责解析 MCP 依赖，source-analyzer 只负责分析源码），符合单一职责原则
- **共享 skill 作为知识库**：多个 agent 共享同一个 skill（`zeroize-audit/SKILL.md`）获取领域知识，skill 是唯一的 SSOT（单一可信源）
- **工具脚本分离**：shell 脚本和 Python 脚本放在 `tools/` 目录，agent 通过 `Bash` 调用，而不是把 shell 逻辑内联在 Markdown 里

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多步骤安全审计工作流 | ✅ 高度适用 | 流水线模式天然适合有明确阶段顺序的审计流程 |
| 需要专业工具集成的任务 | ✅ 适用 | Agent 可以调用 Semgrep/YARA/Burp 等专业工具 |
| 简单问答或知识检索 | ❌ 过度工程 | 单个 skill 足够，不需要 8 个 agent 的流水线 |
| 对安全要求极高的生产环境 | ⚠ 谨慎 | HIGH 安全发现（sudo + bypassPermissions）需先修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 有序 Agent 流水线（本仓库） | trailofbits/skills（zeroize-audit） | 步骤清晰、可调试、可单独运行某步 | 复杂度高，需要维护 8 个 agent 文件 |
| 单 Agent + 工具列表 | 大多数简单插件 | 简单，易维护 | 对于长流程，context 会被撑爆 |
| Skill-only（无 Agent） | mattpocock/skills | 零复杂度，纯知识封装 | 无法执行多步工作流 |

### 2.4 改进空间
1. **当前问题**：`dimensional-analysis` 的 5 个 agent 都没有声明 `model` 字段，会默认使用用户设置的模型，行为不可预测。**改进做法**：加 `model: inherit` 或指定具体模型（如 `model: claude-sonnet-5`）。**预期收益**：审计行为可复现，不受用户全局 model 设置影响。
2. **当前问题**：`workflow-skill-reviewer` agent（唯一目的是审查 workflow skill 质量）本身没有 examples，且没有 model 声明——是其自身领域的最差反例。**改进做法**：给这个 agent 加 2 个 input→output 示例，声明 model。**预期收益**：修复「灯塔不照己之处」的尴尬，提升 agent 质量。
3. **当前问题**：`devcontainer-setup` 的 `post_install.py` 执行 `sudo chown` 时路径参数来自函数参数，存在路径注入风险（HIGH finding）。**改进做法**：先向维护者私下披露，再提供 patch（路径白名单验证或沙盒化）。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **95/100**，但因高危安全发现被标记为 SECURITY BLOCKED。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `spec-compliance-checker.md`（agent） | 60 | 无 examples、无 output format、Write/Bash 工具过度声明 |
| `skill-improver.md`（command） | 63 | 缺少必需 `name` 字段（BUG） |
| `cancel-skill-improver.md`（command） | 65 | 缺少必需 `name` 字段（BUG） |
| `function-analyzer.md`（agent） | 70 | 无 examples、无 output format、无 model |
| 多个 `zeroize-audit` agent | 80 | 无 examples |
| 大多数 skill | 94-100 | 少量 vague quantifier（appropriate/various） |

### 3.2 当时值得借鉴的模式
1. **有序编号 Agent 流水线**：`zeroize-audit` 的 8 个 agent 用 0-6 的数字前缀编码顺序。为什么好：文件名即文档，不需要另写流程说明。原文示例：`plugins/zeroize-audit/agents/`。如何借鉴：多步骤工作流改用编号命名，而不是自然语言名（`analyze`）。
2. **100 分的安全专项 skill**：`constant-time-analysis`、`firebase-apk-scanner`、`seatbelt-sandboxer` 等完全无量词，每个 skill 聚焦一个具体的安全技术。为什么好：安全技术细节本身就精确，不需要也不应该模糊。如何借鉴：技术工具类 skill 应避免「适当使用」这类描述，改为明确的工具调用语法。
3. **Hook + Shim 模式**：`gh-cli` plugin 用 `setup-shims.sh` hook 在 session 启动时注入 gh CLI shim，让 Claude 可以「无缝」调用 GitHub CLI。为什么好：用户不需要手动配置 PATH，plugin 自动处理环境。如何借鉴：需要特定工具的 plugin 可以用 SessionStart hook 自动准备工具环境（但需注意 PATH 注入的安全风险）。

### 3.3 当时的缺陷
1. **「Write/Bash 工具过度声明」**（spec-compliance-checker）：合规检查 agent 声明了 `Write` 和 `Bash` 工具，但其职责是「只读分析」。根本原因：工具列表没有经过最小权限原则审查，开发者「以防万用」地加了额外工具。自查：我的 agent 是否也有「以防万用」的工具声明？→ 需要检查。
2. **「名字缺失 bug」**（skill-improver command）：command 文件缺少 `name` 字段，plugin registry 无法通过名字找到它。根本原因：frontmatter 的必填字段没有 linter 验证，作者写完 body 后可能忘记了 `name`。自查：我的 command 文件是否有 `name` 字段？→ 需要检查。
3. **「灯塔不照己之处」**（workflow-skill-reviewer）：审查 workflow skill 质量的 agent 本身没有 examples 和 model，违反了它声称要推广的最佳实践。根本原因：质量标准没有在自身上自动执行——「说一套做一套」在 NL 代码里和在普通代码里同样会发生。

### 3.4 当时的优化机会
1. 给 `skill-improver` 和 `cancel-skill-improver` 加 `name` 字段（5 分钟 bug fix）
2. 从 `spec-compliance-checker` 移除 `Write` 和 `Bash` 工具声明（10 分钟 bug fix）
3. 给缺少 examples 的 6 个 agent 各补 2 个 input→output 示例（各约 30 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `skill-improver.md` 缺少 `name` 字段 | `head -10 plugins/skill-improver/commands/skill-improver.md` | **PERSISTS**：frontmatter 无 `name` 字段（有 description、argument-hint、allowed-tools） | `/skill-improver` 命令仍无法通过名字注册到 plugin registry |
| `cancel-skill-improver.md` 缺少 `name` | 同上 | **PERSISTS**：同样缺失 `name` | 同上 |
| `spec-compliance-checker` 有 Write/Bash | `grep "^tools:" plugins/spec-to-code-compliance/agents/spec-compliance-checker.md` | **PERSISTS**：仍然有 `tools: Read, Grep, Glob, Write, Bash` | 合规检查 agent 仍有写入权限，存在 prompt injection 风险 |
| HIGH security（post_install.py sudo/bypassPermissions） | `ls plugins/devcontainer-setup/.../resources/post_install.py` | **PERSISTS**：文件存在，HIGH 风险未修 | 需要私下披露；贡献者 PR 仍被 SECURITY BLOCKED |
| PATH 注入（setup-shims.sh） | `grep "CLAUDE_ENV_FILE" plugins/gh-cli/hooks/setup-shims.sh` | **PARTIAL FIX**：已添加 `CLAUDE_ENV_FILE` 为空时的防护 guard（`if [[ -z "${CLAUDE_ENV_FILE:-}" ]]; then exit`），但 PATH prepend 本身仍存在 | Medium 风险改善，High 本质未变 |

### 4.2 架构演进
与 audit 时相比，新增了 `c-review` 和一个 `let-fate-decide`（随机决策工具），plugin 总数从约 35 增至 40。`gh-cli` 的 `setup-shims.sh` 加了 `CLAUDE_ENV_FILE` 空值 guard，说明团队看到了 Medium 安全反馈并有所响应，但 HIGH 级别的问题仍然待处理。

### 4.3 新增的可学习模式
当前 HEAD 中 `c-review` plugin（C 语言代码审查）是新增的，延续了「专项安全工具 plugin」的模式。无重大架构变化，说明 40 个 plugin 的结构已趋于稳定。

---

## 五、校准

### 5.1 我已经在做对的
1. **工具最小权限**：echo-sleuth 和 bureau 的 agent 只声明了确实需要的工具（Read、Grep、Glob 等），没有「以防万用」的 Write/Bash。
2. **Agent examples**：bureau 的 agent 文件有清晰的 input→output 示例，没有出现 trailofbits 「0 examples」的情况。
3. **没有 bypassPermissions**：我的任何 plugin 都没有设置 `bypassPermissions: true`，不会修改全局 Claude 权限配置。

### 5.2 挑战 / 验证
这个案例验证了我之前的判断：**「SECURITY BLOCKED 状态对贡献者是实质性障碍」**。trailofbits 的 HIGH 安全发现（`bypassPermissions: true`、sudo chown 路径注入）已经 3 个月没有修复，说明团队知道问题但内部优先级排序使其被拖延。对于维护者来说，安全问题的修复成本（需要内部评审、测试）远高于 bug 修复。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否有 name 字段
find /tmp/my-repos/MarkQWu-* -path "*/commands/*.md" -exec grep -L "^name:" {} \;
```
命中后怎么办：在 frontmatter 里加 `name: <命令名>` 字段。

```bash
# 检查我的 agent 是否声明了不必要的 Write/Bash 工具
find /tmp/my-repos/MarkQWu-* -path "*/agents/*.md" -exec grep -l "Write\|Bash" {} \; | while read f; do
  # 验证这个 agent 是否真的需要写权限
  echo "CHECK: $f"
  grep "^tools:" "$f"
done
```
命中后怎么办：如果 agent 职责是「只读分析」，移除 Write/Bash；如果确实需要写，加注释说明原因。

```bash
# 检查我的 agent 是否有 model 声明
find /tmp/my-repos/MarkQWu-* -path "*/agents/*.md" -exec grep -L "^model:" {} \;
```
命中后怎么办：加 `model: claude-sonnet-5` 或 `model: inherit` 明确声明。

### 6.2 灵感 → 实施路径

1. **想法**：将 bureau 的多步骤知识捕获流程改为有序编号 agent 流水线（`0-capture → 1-compile → 2-review`）。
   - **为何可行**：bureau 的 capture/compile/review 本来就是有序的三步骤，当前是单一 command 驱动，改为 3 个 agent 后可以单独调试每一步。
   - **第一步**：把 `bureau/commands/capture.md` 的步骤拆分为 `agents/0-ingest.md`、`agents/1-classify.md`，修改 command 为 dispatcher。约 2 小时。

2. **想法**：参考 `skill-improver` plugin 的设计，给 echo-sleuth 加一个「skill 自改进」机制。
   - **为何可行**：echo-sleuth 已有从历史 session 中挖掘信息的能力，加一个 skill 让它定期分析 Claude 的错误并改进相关 skill，形成闭环。
   - **第一步**：读 trailofbits 的 `skill-improver/SKILL.md` 作为模板，在 echo-sleuth 里创建 `skills/skill-evolution/SKILL.md`。约 3 小时。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 trailofbits/skills 的核心目的**：将安全研究公司的专业审计方法论（fuzzing、合约审计、YARA、Semgrep）编码为 Claude Code 可执行的 skill/agent 工作流，降低安全审计的人力成本。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 skill+agent 组合 | echo-sleuth 聚焦记忆挖掘，不是安全审计 | 低（主要学架构模式） |
| MarkQWu/bureau | 低 | 都有多步骤 agent 流水线结构 | bureau 是知识管理，不是安全工具 | 中（流水线模式可借鉴） |

「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| command 缺少 `name` 字段 | `grep -L "^name:" commands/*.md` | echo-sleuth commands 目录有 4 个命令，均需核查 | 高（如命中则 command 无法注册） |
| agent 工具过度声明（Write/Bash） | `grep "Write\|Bash" agents/*.md` | bureau agents 中有 `scribe` 和 `capture`，scribe 需要写权限是合理的，capture 也需要写 | 低（有合理需要） |
| agent 无 `model` 声明 | `grep -L "^model:" agents/*.md` | echo-sleuth 的 agents/ 目录无文件，bureau 的 agents 可能缺少 model 声明 | 中 |

**具体命中行动**：
```bash
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "*.md" -path "*/commands/*" | \
  xargs grep -L "^name:" 2>/dev/null
```
如命中：在 frontmatter 加 `name:` 字段（每个约 2 分钟）。

### 7.3 别人的更优方案

1. **领域**：有序编号 Agent 流水线（命名即文档）
   - **本案例做法**：`0-preflight.md`, `1-mcp-resolver.md`, `2-source-analyzer.md` ... `6-test-generator.md`——数字前缀让文件系统本身就是执行顺序文档
   - **我的项目现状**：bureau 的 agent 命名为 `scribe.md`、`review.md`，无顺序标识；如果将来加更多 agent，顺序关系会不清晰
   - **如何借鉴**：将多步骤 agent 改为 `0-`, `1-`, `2-` 前缀命名；单步骤 agent 保持现状

2. **领域**：Plugin 内 skill 作为领域知识 SSOT
   - **本案例做法**：`zeroize-audit` 的 8 个 agent 都通过 Read 加载同一个 `zeroize-audit/SKILL.md` 获取领域知识，修改知识只需改一个文件
   - **我的项目现状**：bureau 的 agents 各自内联领域描述，如果需要更新规则，需要逐个文件修改
   - **如何借鉴**：把 bureau 的 agents 共用的知识（如 trust tier 规则、memory 格式规范）提取到一个共享 skill 文件，各 agent 通过 Read 加载

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：security 设计（无 bypassPermissions）
- **我的做法**：echo-sleuth 和 bureau 的所有 agent/skill 没有设置 `bypassPermissions: true`，不会修改 Claude 的全局权限模型
- **本案例做法（弱在哪）**：`devcontainer-setup` 的 `post_install.py` 写入 `~/.claude/settings.json` 并设置 `bypassPermissions: true`，这是一个明确的 HIGH 级安全问题——修改了用户 Claude Code 的全局安全设置，可能在所有后续会话中削弱权限保护
- **意义**：这一设计决策（不修改全局权限）是正确的安全实践，在被审查时是亮点。

---

## 八、术语表

### <a name="工件"></a>工件（artifact）
> 在 NLPM 体系中，「工件」泛指所有用自然语言编写的 Claude Code 配置文件，包括 SKILL.md、agent 定义、command、hook 配置、plugin manifest 等。「151 个工件」= 这些文件的总数。

### <a name="bypassPermissions"></a>bypassPermissions
> Claude Code 的一个全局设置项，位于 `~/.claude/settings.json`。设置为 `true` 后，Claude 在执行文件写入、运行命令等操作时不再弹出权限确认弹窗——所有操作直接执行。这在开发环境中可以提升效率，但也意味着 Claude 可以在没有用户确认的情况下修改任何文件、执行任何命令。安全风险：如果 plugin 被攻击者控制，`bypassPermissions: true` 会让后续所有操作绕过安全防护。

### <a name="YARA"></a>YARA 规则
> 一种用于描述恶意软件模式的领域专用语言，主要用于安全分析和威胁情报。Trail of Bits 的 `yara-authoring` skill 专门帮助编写 YARA 规则，用于检测恶意软件样本中的已知模式。

### <a name="fuzzing"></a>Fuzzing（模糊测试）
> 一种自动化测试技术，通过向程序输入大量随机或半随机数据，寻找会导致崩溃或安全漏洞的边界情况。Trail of Bits 的 `testing-handbook-skills` 包含 15 个 fuzzing 相关 skill（cargo-fuzz、AFL++、libFuzzer 等），覆盖 Rust/C 等主流语言的 fuzzing 工作流。

### <a name="Semgrep"></a>Semgrep
> 一款开源的静态代码分析工具，支持自定义规则以检测安全漏洞和代码模式。Trail of Bits 的 `semgrep-rule-creator` 和 `variant-analysis` 等 plugin 将 Semgrep 的专业用法编码为 Claude Code skill。

### <a name="shim"></a>Shim（垫片）
> 一个薄的中间层程序，拦截对某个工具的调用，在调用前后做额外处理。Trail of Bits 的 `gh-cli` plugin 用 `setup-shims.sh` 创建一个 `gh` shim，让 Claude Code 调用 `gh` 命令时经过 Trail of Bits 的拦截脚本处理（如记录 session ID、清理临时克隆等）。
