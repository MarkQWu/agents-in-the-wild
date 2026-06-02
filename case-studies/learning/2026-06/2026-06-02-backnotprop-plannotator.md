# backnotprop/plannotator — 学习案例

**仓库**：https://github.com/backnotprop/plannotator
**Stars**：4,561 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-02（基于当前 HEAD）
**主题标签**：`cross-reference`, `template-design`, `manifest-discipline`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

plannotator 是 4,561 颗星的 Claude Code 生态插件，核心功能是为 AI 编程会话中的 Plan Mode 提供「注释与审查」协作体验。当 Claude 进入 Plan Mode（生成执行计划但尚未动手之前），plannotator 会将计划存入结构化存档，让用户和 Claude 可以对每个计划决策进行注释、审查、追溯，最终形成可查询的决策历史。

截至 2026-06-02 当前 HEAD，仓库已覆盖以下 AI 编程平台：

| 应用目录 | 平台 |
|---|---|
| apps/hook | Claude Code 原生（hooks 机制） |
| apps/copilot | GitHub Copilot |
| apps/opencode-plugin | OpenCode |
| apps/gemini | Gemini CLI |
| apps/codex | OpenAI Codex |
| apps/amp-plugin | Amp |
| apps/droid-plugin | Factory.ai Droid |
| apps/vscode-extension | VS Code 扩展 |
| apps/pi-extension | Pi 扩展 |
| apps/review | 代码审查专项 |
| apps/portal | 用户门户 |
| apps/paste-service | 粘贴服务 |
| apps/waitlist-service | 等待列表服务 |
| apps/marketing | 营销网站（Astro） |
| apps/skills | 可复用 skill 合集 |

关键设计事实：
1. 同一套 plan 注释功能以各平台的本地惯例分别实现，而非使用单一抽象层——这是有意为之的[多平台同构设计](#multi-harness)
2. 每个平台的 commands 文件夹里的 `.md` 工件内容高度相似但因各平台工具名不同而略有差异（如 `Bash` vs `shell`）
3. 版本已从 audit 时的 0.19.1 迭代至 0.19.26，说明仓库维护非常活跃
4. `CLAUDE.md` 现在是指向 `AGENTS.md` 的符号链接——对所有支持读取 `CLAUDE.md` 的 AI 工具同时适配 `AGENTS.md` 格式
5. 新增了三个 skill：`plannotator-visual-explainer`、`plannotator-setup-goal`、`plannotator-last`

### 1.2 架构剖析

目录结构（简化）：

```
plannotator/
  CLAUDE.md → AGENTS.md          # 符号链接，跨工具通用入口
  AGENTS.md                      # 通用 agent 上下文说明
  apps/
    hook/                        # Claude Code 原生平台
      .claude-plugin/
        plugin.json              # manifest（version: 0.19.26）
      commands/                  # 5 个 command（全部 100 分）
        plannotator-annotate.md
        plannotator-archive.md
        plannotator-last.md
        plannotator-review.md
        plannotator-...
      hooks/
        hooks.json               # PreToolUse/EnterPlanMode + PermissionRequest/ExitPlanMode
    copilot/
      commands/                  # 3 个 command（全部 100 分）
    opencode-plugin/
      commands/                  # 4 个 command（2 个有问题）
    gemini/
      hooks/
        settings-snippet.json
    codex/ amp-plugin/ droid-plugin/ ...
    skills/
      plannotator-compound/
        SKILL.md                 # 100 分
      plannotator-visual-explainer/
        SKILL.md                 # 新增
      plannotator-setup-goal/
        SKILL.md                 # 新增
      plannotator-last/
        SKILL.md                 # 新增
  .agents/
    skills/                      # 4 个内部 skill（全部 100 分）
      pierre-guard/ release/ review-renovate/ update-deps/
  scripts/
    install.sh                   # 主安装脚本（同时复制到 apps/marketing/public/）
```

**编排关系**：每个平台的 commands 是无状态的入口点，调用底层 `plannotator` 二进制（通过 `Bash(plannotator:*)` 或 `shell(plannotator:*)` 工具权限）。二进制承担实际的存档读写逻辑。NL 工件的职责是「告诉 AI 何时调用什么参数」，不直接操作文件系统。

**跨件契约**：
- hook 平台：`allowed-tools: Bash(plannotator:*)` → Claude Code 的 `Bash` 工具
- copilot 平台：`allowed-tools: shell(plannotator:*)` → Copilot 的 `shell` 工具
- opencode 平台：不使用 `allowed-tools` 字段（OpenCode 运行时不处理该字段）
- 工具名不同是平台差异的合法体现，但移植 command 时必须调整（Bug#1 的根因正是遗漏了这一步）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「平台原生表达，而非最小公分母」——plannotator 不尝试写一套在所有平台都能运行的抽象 command，而是为每个平台写一份符合该平台惯例的实现。代价是维护多份近似内容，收益是每个平台用户都获得原生体验。
- **解决什么问题**：Plan Mode 是 AI 编程中高价值但高风险的决策点——AI 在这里提议要做什么，但用户往往来不及仔细审查。plannotator 把这个决策点持久化并可注释，把「AI 自言自语的计划」变成「可协作的结构化记录」。
- **做了什么 trade-off**：为了覆盖更多平台，接受了 command 内容重复和跨平台同步维护的成本；选择薄 NL 层 + 厚二进制层的架构，NL 层只做调度，业务逻辑全部在二进制里——这使 NL 工件保持简洁，也意味着 NL 工件的质量上限主要体现在状态处理的完备性上。
- **CLAUDE.md 符号链接技巧**：将 CLAUDE.md 做成指向 AGENTS.md 的符号链接，是在「Claude Code 读 CLAUDE.md」和「其他兼容工具读 AGENTS.md」之间取得一致性的简洁方案，避免维护两份内容。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多平台同构分发」（Multi-Harness Isomorphic Distribution）**

核心特征：同一套功能为多个 AI 编程平台分别实现，每份实现遵循该平台的本地惯例（工具名、frontmatter 字段、hook 事件名等），共享底层二进制逻辑但不共享 NL 层。

模式特征清单（5 条）：
- 特征 1：每个平台有独立的 `apps/<platform>/` 目录，包含该平台所需的全部 NL 工件
- 特征 2：同一 command 在不同平台使用不同的工具名（`Bash` vs `shell` vs 无 `allowed-tools`），这是正确的而非错误
- 特征 3：底层执行逻辑集中在二进制（`plannotator` CLI），NL 工件只做「何时/如何调用」的编排，不做业务逻辑
- 特征 4：平台间 command 内容高度相似但非完全一致——状态处理（empty/approve/dismiss 分支）的丰富程度因平台成熟度不同而有差异
- 特征 5：一个共享安装脚本（`install.sh`）处理多平台的安装选择，用户选择平台后脚本把对应的 NL 工件写入正确位置

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 功能需要在多个 AI 平台上都能使用 | ✅ 高度适用 | 多平台同构设计的核心目标 |
| 底层业务逻辑已有成熟的 CLI 工具 | ✅ 适用 | 薄 NL 层 + 厚二进制层的前提是有二进制可调 |
| 功能本身与平台 Plan Mode 或会话生命周期强关联 | ✅ 适用 | hook 事件（EnterPlanMode/ExitPlanMode）是这一场景的天然触发点 |
| 只需要支持单一平台的简单插件 | ❌ 不适用 | 多平台同构带来的维护成本对单平台场景是纯负担 |
| 没有独立二进制、业务逻辑全在 NL 层 | ⚠️ 有限适用 | 需要在每个平台维护完整业务逻辑，重复度极高，容易出现跨平台不一致 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多平台同构分发（本仓库） | backnotprop/plannotator | 每个平台原生体验；底层逻辑集中 | 跨平台 NL 工件同步维护成本高；移植时易遗漏平台差异（Bug#1 根因） |
| 单平台深度（hook-only）| 大多数 Claude Code 插件 | 维护简单；可专注单平台最佳实践 | 无法服务其他平台用户 |
| 单一抽象层（通过 MCP）| MCP server 型插件 | 一份实现，所有支持 MCP 的客户端可用 | MCP 尚未覆盖所有平台；协议版本兼容性问题 |
| 完整插件无二进制（纯 NL）| 多数社区插件 | 无安装依赖；零执行面 | 复杂逻辑难以在纯 NL 中可靠实现 |

### 2.4 改进空间

1. **当前问题**：`apps/opencode-plugin/commands/plannotator-last.md` 至今仍无 body，agent 调用该 command 时没有任何指导。**改进做法**：参照 `apps/hook/commands/plannotator-last.md` 或 `apps/copilot/commands/plannotator-last.md`，将对应的指令正文适配到 OpenCode 平台（去掉 `!`bang 语法，确认无 `allowed-tools`），添加为 body。**预期收益**：Bug#2 修复，该文件从 65 分升至 90+ 分，OpenCode 用户获得实际可用的 last 命令。

2. **当前问题**：`apps/opencode-plugin/commands/plannotator-annotate.md` 和 `plannotator-review.md` 缺少 empty/approve/dismiss 状态分支，而 hook 和 copilot 变体有完整的三路分支处理。**改进做法**：在这两个文件中补充「如果没有任何变更请求，确认并继续」「如果用户批准，归档并退出」「如果用户驳回，列出理由并重试」三个显式分支。**预期收益**：两个文件从 90 分升至 100 分，消除 OpenCode 平台的状态处理盲区。

3. **当前问题**：`apps/gemini/hooks/settings-snippet.json` 是片段文件，不包含 `experimental.plan: true` 配置块；手动应用该 snippet 的用户会错过 Gemini plan-mode 启用。**改进做法**：在 snippet 文件顶部增加注释，说明该 snippet 需要配合 install.sh 生成的完整配置使用，或直接将 `experimental.plan: true` 包含进 snippet 并附注若已存在则合并。**预期收益**：消除手动安装场景下的 Gemini plan-mode 静默失效。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

**加权平均分：93/100**（23 个工件，批量评分策略）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| apps/opencode-plugin/commands/plannotator-archive.md | 65 | 包含 Claude Code `!`bang 语法和 `Bash(plannotator:*)` — OpenCode 中均无效，archive 静默失效 |
| apps/opencode-plugin/commands/plannotator-last.md | 65 | body 为空，agent 无任何指令 |
| apps/marketing/src/content/docs/commands/*.md（4 个）| 90 | 非可执行文档工件，无 agent 指令语义（Astro 内容集合）|
| apps/hook/hooks/hooks.json | 90 | EnterPlanMode hook 存在但 install.sh 未写入 |
| apps/gemini/hooks/settings-snippet.json | 90 | 片段式配置，缺失 Gemini plan-mode 启用块 |
| apps/opencode-plugin/commands/plannotator-annotate.md | 90 | 缺少 empty/approve/dismiss 状态分支 |
| apps/opencode-plugin/commands/plannotator-review.md | 90 | 同上，缺少状态分支 |
| apps/hook/.claude-plugin/plugin.json | 95 | manifest 完整有效 |
| copilot commands（3 个）| 100 | — |
| hook commands（5 个）| 100 | — |
| skills（5 个）| 100 | — |

**安全评级**：REVIEW（3 个 Critical 均为假阳性：`curl … | bash` 出现在 usage() heredoc 文本中从不执行；1 个 HIGH：`node -e` 块中 `$GEMINI_SETTINGS` 未引用，实际攻击面可忽略但架构上不安全）

### 3.2 当时值得借鉴的模式

1. **hook command 状态处理三路分支** → 为什么好：hook 平台的 5 个 command 均显式处理了 empty output、approve、dismiss 三种状态，agent 在任何响应情况下都有明确行为路径，不会陷入「用户什么也没说该怎么办」的不确定态 → 原文路径：`apps/hook/commands/plannotator-annotate.md` 和 `plannotator-review.md` → 如何借鉴：在自己的 command 中，不论何种场景，都应显式列出「如果输出为空/用户批准/用户拒绝」三路分支，消除隐含前提。

2. **skill 内部化（内部 agent skill 分离）** → 为什么好：`.agents/skills/` 下有 4 个专为仓库内部 agent 使用的 skill（pierre-guard、release、review-renovate、update-deps），与面向用户的 `apps/skills/` 明确分层——内部操作知识和外部发布知识各有归属，互不干扰 → 原文路径：`.agents/skills/` vs `apps/skills/` 目录对比 → 如何借鉴：维护多 agent 的仓库中，将「仓库自身维护流程相关的 skill」和「面向用户功能的 skill」分开放置。

3. **manifest 字段精准（plugin.json 版本跟踪）** → 为什么好：`apps/hook/.claude-plugin/plugin.json` 声明 version `0.19.1`，release skill 要求 7 个文件原子性同步更新版本号——这形成了强一致性约束，防止版本漂移 → 原文路径：`apps/hook/.claude-plugin/plugin.json` + `.agents/skills/release/SKILL.md` → 如何借鉴：如果插件涉及多文件版本号，写一个 release skill 明确列出所有需要同步更新的文件路径，作为发版 checklist。

### 3.3 当时的缺陷

1. **Bug#1 — archive.md bang 语法移植错误**：`apps/opencode-plugin/commands/plannotator-archive.md` 是 hook 变体的逐字拷贝，保留了 Claude Code 特有的 `!``plannotator archive`` ` bang 命令语法和 `allowed-tools: Bash(plannotator:*)` frontmatter。OpenCode 运行时不处理这两个字段，archive 子命令在 OpenCode 中永远不会被调用，功能静默失效。为什么这会发生：跨平台移植时遗漏了「清理源平台特有语法」这一步骤，没有「移植清单」机制来强制执行差异点检查。

2. **Bug#2 — last.md body 为空**：`apps/opencode-plugin/commands/plannotator-last.md` 的 frontmatter 后正文为空（整个文件只有 3 行）。agent 调用该 command 时接收不到任何指令，行为完全不确定。为什么这会发生：该文件很可能是用来「占位」——先建立文件结构，正文留待填写，但后来被遗忘。

3. **EnterPlanMode hook install 缺口**：`apps/hook/hooks/hooks.json` 注册了 `PreToolUse/EnterPlanMode` hook，但 `scripts/install.sh` 只写入 `PermissionRequest/ExitPlanMode` hook。通过 marketplace 安装的用户能得到两个 hook，通过 install.sh 安装的用户只得到一个。这造成了安装路径不同导致行为不同的静默差异。

### 3.4 当时的优化机会

1. 为 OpenCode 的 annotate.md 和 review.md 补充 empty/approve/dismiss 三路分支，两个文件从 90 升至 100。
2. 修复 Bug#1：重写 archive.md，去掉 bang 语法和 Claude Code 专有 `allowed-tools`，参照其他 OpenCode commands 的写法。
3. 修复 Bug#2：为 last.md 补充 body，哪怕只有 1-2 行简短指令也比空文件强。
4. 在 install.sh 中补充 EnterPlanMode hook 写入逻辑，消除安装路径不一致。
5. 修复 `node -e` 块中 `$GEMINI_SETTINGS` 的未引用扩展问题，防御路径包含 shell 元字符的场景。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查结果 | 现状 | 含义 |
|---|---|---|---|
| Bug#1：archive.md bang 语法 | 当前 `plannotator-archive.md` frontmatter 只有 `description: Browse saved plan decisions in the archive`，无 bang 语法，无 `allowed-tools` | **已修复** | 维护者在 audit 后的约 1 个月内修复了此 bug；跨平台移植检查点已被重视 |
| Bug#2：last.md body 为空 | 当前文件仍为 3 行（frontmatter only，无 body） | **持续存在** | 修复 archive.md 时未同步修复 last.md；或维护者认为 last command 对 OpenCode 用户的优先级较低 |
| EnterPlanMode hook install 缺口 | `install.sh` 第 521-523 行现包含 PreToolUse/EnterPlanMode 写入逻辑 | **已修复** | 安装路径差异已消除，所有安装方式现在都能得到完整 hook 配置 |
| node -e 未引用变量 | 未单独核查，架构模式未变化 | **状态未知** | 属于安全加固项而非功能 bug，维护者可能已单独处理或接受了风险 |

**总结**：2 个 PR 级 bug 中修复了 1 个；1 个跨组件 install 缺口已修复；1 个 bug 持续存在。版本从 0.19.1 → 0.19.26，说明主动维护节奏良好。

### 4.2 架构演进

从 audit（2026-04-26）到当前 HEAD（2026-06-02），约 5 周内发生了以下显著变化：

1. **新增 5 个平台支持**：`droid-plugin`（Factory.ai）、`amp-plugin`（Amp）、`portal`、`paste-service`、`waitlist-service` 这 5 个 `apps/` 子目录在 audit 时不存在。这说明多平台同构扩张仍在高速进行中。

2. **新增 3 个 skill**：`plannotator-visual-explainer`、`plannotator-setup-goal`、`plannotator-last` 进入 `apps/skills/`。skill 层的扩充意味着核心功能在向「可组合知识单元」方向演进，而不只是平台特化命令集。

3. **CLAUDE.md 符号链接化**：`CLAUDE.md` 现在是指向 `AGENTS.md` 的符号链接。这是一个架构简化决策——消除了 CLAUDE.md 和 AGENTS.md 内容潜在不同步的问题，同时保持了对两类工具的兼容性。

4. **版本快速迭代**：0.19.1 → 0.19.26，5 周 25 个版本号间隔，说明活跃的功能迭代。

### 4.3 新增的可学习模式

1. **CLAUDE.md 符号链接模式**：将 CLAUDE.md 做成指向 AGENTS.md 的符号链接，是同时兼容「Claude Code 读 CLAUDE.md」和「其他工具读 AGENTS.md」的零维护成本方案。适用于多工具场景下的通用上下文声明文件。

2. **平台攀登策略（Platform Ladder）**：audit 时已有 9 个 `apps/` 目录，5 周后增至 15 个。这说明「先在核心平台（hook/copilot）建立完善实现，再逐步移植到新平台」是一种可持续的扩张节奏——不必等待所有平台同时完善再发布，允许新平台工件处于「功能基本可用但状态处理不完整」的中间状态。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 均有完整 frontmatter 和 body**：我的 echo-sleuth-for-claude 的 8 个 command 都有 frontmatter 且有正文，不存在 Bug#2 同类问题（空 body 命令）。这是基础质量的体现，应持续保持。

2. **agents 有明确的工具权限声明**：echo-sleuth 的 agent 文件中有 `allowed-tools` 声明，与 plannotator hook 平台 commands 的做法一致。plannotator 的 Bug#1 恰恰是在跨平台移植时没有调整工具名，说明正确声明工具名本身是重要的。

3. **claude-for-legal 的 agents 全部有 model 字段**：所有 agent 声明了使用的模型，这是我已经做对的地方，plannotator audit 的评分细则也肯定了这一实践。

### 5.2 挑战 / 验证

- **plannotator 证明了「移植清单」的必要性**：Bug#1 的根因是跨平台移植时没有检查清单来提醒开发者「清理源平台专有语法」。我的 echo-sleuth 目前是单平台（Claude Code），但如果未来考虑移植到其他平台，必须事先写一份「移植差异点清单」，否则会重蹈 Bug#1。

- **「占位文件」的风险**：Bug#2（空 body last.md）很可能源于「先建文件结构，正文待填」的开发习惯。我需要检查自己的仓库是否存在类似的「frontmatter 完整但 body 为空或仅有 placeholder 文字」的工件。

- **多平台 skill 的新增模式值得跟踪**：plannotator 新增的 3 个 skill（visual-explainer、setup-goal、last）说明维护者开始把可复用逻辑从 command 抽离到 skill 层。这种「从命令化到知识化」的演进路径，对于我的 echo-sleuth 也有参考价值——5 个 agent 里是否有重复的知识块可以抽离成 skill？

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的仓库中是否存在 body 为空（或极短）的 command 文件
# 逐个检查 commands/ 目录下的 .md 文件，打印非 frontmatter 行数
for f in commands/*.md; do
  body_lines=$(awk '/^---/{n++} n>=2{print}' "$f" | grep -cv '^[[:space:]]*$')
  [ "$body_lines" -lt 2 ] && echo "SHORT BODY ($body_lines lines): $f"
done

# 检查 opencode/copilot 类平台的 command 文件是否遗留了 Claude Code 特有语法
# bang 命令语法：以 ! 开头的行
grep -rn "^\!" apps/*/commands/*.md 2>/dev/null && echo "FOUND bang syntax — check if this is an OpenCode file"

# 检查 allowed-tools 字段在各平台文件中是否使用了正确的工具名
# Claude Code 平台：应为 Bash(...)，Copilot 平台：应为 shell(...)
grep -rn "allowed-tools:" apps/*/commands/*.md 2>/dev/null | \
  grep "opencode" | grep -v "shell\|Bash" && echo "UNEXPECTED tool name in opencode commands"

# 检查自己的 plugin.json 中声明的所有文件路径是否实际存在
python3 -c "
import json, os, sys
data = json.load(open('plugin.json'))
missing = []
for key in ('commands', 'skills', 'agents'):
    for path in data.get(key, []):
        full = os.path.join(os.path.dirname('plugin.json'), path)
        if not os.path.exists(full):
            missing.append(f'{key}: {path}')
if missing:
    print('MISSING FILES IN plugin.json:')
    for m in missing: print(' ', m)
else:
    print('All plugin.json paths exist.')
"

# 检查仓库内是否有「frontmatter 存在但 body 完全为空」的工件
find . -name "*.md" -path "*/commands/*" | while read f; do
  # 计算 frontmatter 外的非空行数
  lines=$(python3 -c "
import sys
content = open('$f').read()
parts = content.split('---')
body = '---'.join(parts[2:]) if len(parts) >= 3 else ''
print(len([l for l in body.splitlines() if l.strip()]))
")
  [ "$lines" -eq 0 ] && echo "EMPTY BODY: $f"
done

# 检查 agent 文件是否都有 model 字段
for f in agents/*.md; do
  grep -q "^model:" "$f" || echo "MISSING model field: $f"
done

# 检查是否存在模糊量词（vague quantifier）
grep -rn \
  "as needed\|when appropriate\|if necessary\|as required\|where applicable\|relevant\|comprehensive\|robust" \
  agents/ commands/ skills/ 2>/dev/null | grep -v "^Binary"
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 建立「平台移植清单」skill
   - **为何可行**：echo-sleuth 目前只在 Claude Code 上运行，但如果未来要移植到 Copilot 或 OpenCode，需要一份平台差异备忘录（工具名、frontmatter 字段、hook 事件名差异）。plannotator Bug#1 证明这类清单在有人维护的活跃仓库中也会被遗漏。
   - **第一步**：在 `.agents/skills/` 下创建 `platform-porting/SKILL.md`，记录 Claude Code / Copilot / OpenCode / Gemini 的工具名差异和 frontmatter 差异表；约 20 分钟。

2. **想法**：对 echo-sleuth 的重复性 agent 知识块做 skill 抽离
   - **为何可行**：plannotator 从命令层向 skill 层演进的趋势（新增 3 个 skill），提示了一种优化路径：如果多个 agent 共用同一套「如何解读会话数据」的判断规则，把它抽成共享 skill 后每个 agent 只需 `skills:` 引用，维护更集中。
   - **第一步**：审查 echo-sleuth 5 个 agent 的正文，找出重复出现 2 次以上的段落，提炼为候选 skill；约 30 分钟。

3. **想法**：将 CLAUDE.md 做成符号链接（如果同时维护 AGENTS.md）
   - **为何可行**：plannotator 的 `CLAUDE.md → AGENTS.md` 符号链接技巧对于任何同时支持 Claude Code 和其他 AI 工具（如 Copilot、Cursor）的仓库都是零成本的兼容性改进。
   - **第一步**：检查自己的仓库是否有 AGENTS.md；如果有，用 `ln -sf AGENTS.md CLAUDE.md` 替换当前的 CLAUDE.md（前提是两份内容相同或 AGENTS.md 是权威版本）；约 5 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`MarkQWu/echo-sleuth-for-claude`、`MarkQWu/drama-workshop-skills`、`MarkQWu/claude-for-legal`

### 8.1 目的对齐度

plannotator 的核心目的是「为 AI 编程 session 的 Plan Mode 提供协作式注释与审查体验」，属于增强 AI session 质量的工具。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中高 | 同为增强 AI session 体验的工具（echo-sleuth 分析 session 历史，plannotator 注释 session 计划）；同有 commands + agents + skills 完整架构；都调用 Bash 工具操作本地数据 | echo-sleuth 是事后分析工具（挖掘历史），plannotator 是实时协作工具（拦截当前计划）；echo-sleuth 无多平台支持 | 高——架构复杂度相近，问题集也高度重叠 |
| MarkQWu/claude-for-legal | 低 | 同为多 agent 结构的专业工作流插件 | 法律工作流与 Plan Mode 注释目的完全不同；claude-for-legal 的执行面（agent 工具权限）比 plannotator 更谨慎 | 低——目的差异大，仅 manifest 精准度可参考 |
| MarkQWu/drama-workshop-skills | 极低 | 同为 Claude Code 生态插件 | 创作工坊 vs 会话协作工具，完全不同域；drama-workshop 是单 skill 极简架构，plannotator 是多平台复杂架构 | 低——架构模式不同，借鉴价值有限 |

### 8.2 在我的项目里复现的同类问题

| plannotator 缺陷 | 检查方式 | echo-sleuth 命中情况 | 严重度 |
|---|---|---|---|
| Bug#2：command body 为空 | `find commands/ -name "*.md" \| xargs awk '/^---/{n++} n>=2{found=1} END{if(!found)print FILENAME}'` | 8 个 command 均有 body（无命中） | — |
| 跨平台工具名不一致 | 单平台，不适用 | 不适用 | — |
| agent 缺少 model 字段 | `grep -L "^model:" agents/*.md` | echo-sleuth 5 个 agent 全部缺少 model 字段（0/5 有 model 声明） | 高——NLPM 每个 agent 扣 5 分 |
| 模糊量词 | `grep -rn "as needed\|when appropriate\|relevant" agents/` | echo-sleuth agents 中有 4 处模糊量词 | 中（每处 -2 分） |
| 平台差异移植遗漏 | 单平台，不适用 | 不适用 | — |

**最高优先级行动**：echo-sleuth 的 5 个 agent 全部缺少 `model:` 字段，这是和 plannotator 高分（93/100）相比最大的单项扣分差距。为每个 agent frontmatter 添加 `model: claude-sonnet-4-5` 或项目实际使用的模型，预计每个 agent 提升 5 分。

### 8.3 别人的更优方案

1. **领域**：hook 平台 commands 的三路状态处理（empty/approve/dismiss）
   - **plannotator 做法**：hook 平台的 annotate 和 review commands 明确列出三种用户响应情况的处理逻辑，agent 在任何响应情况下都有确定行为（`apps/hook/commands/plannotator-annotate.md`，100 分）
   - **我的项目现状**：echo-sleuth commands 的 agent 指令中通常只描述「成功路径」，缺少对「用户无响应」「用户拒绝」等边缘状态的显式处理
   - **如何借鉴**：在每个需要用户交互的 command 中，在指令末尾增加一节「边缘状态处理」，明确列出空响应、拒绝、批准三种情况下的下一步行为

2. **领域**：内部 skill 与外部 skill 分层放置
   - **plannotator 做法**：`.agents/skills/`（内部维护流程）和 `apps/skills/`（面向用户功能）两个目录明确区分了 skill 的目标受众
   - **我的项目现状**：echo-sleuth 所有 skill 放在同一个 `skills/` 目录，没有区分「供用户加载」和「供内部 agent 使用」
   - **如何借鉴**：将 echo-sleuth 中仅供内部 agent 使用的 skill（如与 session 数据格式解析相关的技术 skill）移至 `.agents/skills/`，面向用户的功能型 skill 保留在 `skills/`

3. **领域**：CLAUDE.md 符号链接化
   - **plannotator 做法**：`CLAUDE.md → AGENTS.md`，零维护成本的跨工具兼容性
   - **我的项目现状**：echo-sleuth 只有 CLAUDE.md，没有 AGENTS.md；如果将来需要支持其他 AI 工具，维护两份文件的成本将会出现
   - **如何借鉴**：创建 AGENTS.md 作为权威上下文声明文件，将 CLAUDE.md 做成符号链接

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：commands 空 body 问题
   - **我的做法**：echo-sleuth 的 8 个 command 全部有完整的指令正文，没有空文件存在
   - **plannotator 的弱点**：audit 时 plannotator-last.md（OpenCode 变体）body 为空，截至 2026-06-02 仍未修复
   - **意义**：这是基础质量把关的体现，说明我在工件完整性上比这个 4,561 星仓库做得更严格

2. **领域**：agent model 字段（claude-for-legal）
   - **我的做法**：claude-for-legal 的所有 agent 都声明了 `model:` 字段，符合 NLPM 规范
   - **plannotator 的做法**：plannotator 的 NL 层 commands 不包含 agent 文件，因此 model 字段不适用；但对比同类含 agent 的仓库，claude-for-legal 的做法更规范
   - **意义**：如果 echo-sleuth 的 agent 补上 model 字段，将在「agent 声明完整性」上超越大多数社区仓库

---

## 八、术语表

### <a name="multi-harness"></a>多平台同构设计（multi-harness isomorphic distribution）
> 为多个 AI 编程平台（harness）分别实现同一套功能，每份实现遵循该平台的本地惯例，而非使用单一抽象层或最小公分母接口。「同构」指各平台实现的功能语义相同，但语法形式（工具名、frontmatter 字段、hook 事件名）按平台规范分别调整。本仓库是多平台同构设计的典型案例，支持 15 个 `apps/` 子目录对应不同平台。

### <a name="bang-syntax"></a>bang 语法（bang command syntax）
> Claude Code 特有的命令调用语法，格式为 `` !`command arg` ``，用于在 command 的 body 中触发外部 CLI 命令执行。这是 Claude Code 运行时特有的语法，其他平台（如 OpenCode、Copilot）不解析此语法。将含 bang 语法的 Claude Code command 直接复制到其他平台的 commands 目录是 Bug#1 的根因。

### <a name="harness"></a>运行时平台（harness）
> 加载和执行 NL 工件（commands、skills、agents）的宿主程序。不同 harness 对同一概念使用不同的名称（如 Claude Code 的 `Bash` 工具 vs Copilot 的 `shell` 工具）和不同的配置字段（如 `allowed-tools` 在 OpenCode 中被忽略）。编写跨 harness 工件时必须了解目标 harness 的规范，否则会出现静默失效。

### <a name="plan-mode"></a>Plan Mode
> Claude Code 的一种操作模式，在该模式下 AI 先生成执行计划（plan）并等待用户确认，而非直接执行操作。plannotator 的核心功能就是在 Plan Mode 的进入（EnterPlanMode）和退出（ExitPlanMode）两个 hook 事件点介入，对计划内容进行存档和注释。

### <a name="hook-event"></a>hook 事件（hook event）
> Claude Code hooks 机制支持在特定生命周期事件发生时自动触发外部命令。与 plannotator 相关的两个主要事件：`PreToolUse/EnterPlanMode`（即将进入 Plan Mode 时触发，用于写入上下文）和 `PermissionRequest/ExitPlanMode`（用户批准计划、即将退出 Plan Mode 时触发，用于存档计划决策）。

### <a name="fallback-chain"></a>fallback chain（回退链）
> 当某个处理路径不可用时，按优先级依次尝试备用路径的机制。在 plannotator 的 command 设计中，hook 平台的 commands 对用户响应有明确的三路分支（empty/approve/dismiss），当某一分支条件不满足时会回退到默认行为——这是隐式的 fallback chain。相比之下，OpenCode 变体缺少这些分支，导致意外输入时没有定义好的回退行为。

### <a name="dead-file"></a>死文件（dead file）
> 存在于仓库中但实际上不会被任何运行时调用或执行的文件。audit 时 `apps/opencode-plugin/commands/plannotator-archive.md`（包含无效 bang 语法）和 `apps/opencode-plugin/commands/plannotator-last.md`（空 body）都属于死文件——前者的指令永不执行，后者 agent 收到的是空指令。死文件比完全不存在更危险：它们制造了「功能已覆盖」的假象，让用户以为可以使用实际上无效的命令。

### <a name="state-handling"></a>状态处理（state handling）
> command 指令中对不同用户输入状态的显式分支处理。plannotator hook 平台 commands 的三路状态：（1）empty — 用户无响应或无变更请求，确认并继续；（2）approve — 用户批准计划，存档并退出 Plan Mode；（3）dismiss — 用户拒绝计划，列出理由并重新规划。缺少状态处理会导致 agent 在边缘情况下行为不确定。

### <a name="skill-layer"></a>skill 层（skill layer）
> 插件架构中存放可复用知识单元（SKILL.md 文件）的层次。与 command 层（用户触发的执行流程）和 agent 层（自主执行主体）不同，skill 层的内容是「知识和约束」，由 agent 在 frontmatter 中通过 `skills:` 字段按需加载。plannotator 将 `apps/skills/` 用于面向用户的功能 skill，将 `.agents/skills/` 用于仓库内部维护流程 skill，形成两层分离的 skill 架构。

### <a name="symlink-compat"></a>符号链接兼容技巧（symlink compatibility）
> 通过将 CLAUDE.md 做成指向 AGENTS.md 的符号链接，同时满足「Claude Code 读 CLAUDE.md」和「其他兼容工具读 AGENTS.md」的需求，且只需维护一份内容。该技巧的前提是操作系统文件系统支持符号链接（Linux/macOS 原生支持；Windows 需要开发者模式）。plannotator 的 `CLAUDE.md → AGENTS.md` 是该模式在活跃仓库中的实际应用案例。
