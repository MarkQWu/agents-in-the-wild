# breaking-brake/cc-wf-studio — 学习案例

**仓库**：https://github.com/breaking-brake/cc-wf-studio
**Stars**：4893 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-30（基于当前 HEAD）
**主题标签**：`security-gate`, `examples-driven`, `vague-quantifier`, `curl-pipe-bash-risk`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
cc-wf-studio 是一个 VSCode 扩展 + [MCP](#mcp) 服务的组合工具，让开发者在视觉化的工作流画布上编排 Claude Code 任务——把 Claude 的 AI 编辑能力嵌入 VS Code 的 React Flow 画布。4893 颗星说明"可视化编排 Claude Code 工作流"是被广泛认可的需求。

关键事实：
- pnpm monorepo：4 个 package（core/mcp/cli/vscode），生产级工程结构
- NL 工件（skills）嵌在 `.claude/skills/`、`.github/skills/`、`.roo/skills/` 三个路径下，为不同 AI 工具提供适配
- CLAUDE.md 高达 98 分——项目文档质量高
- 审计状态：**BLOCKED**（安全封锁），因为 `.specify/scripts/bash/` 存在 3 处 [eval 命令注入](#eval-injection)漏洞

### 1.2 架构剖析
```
cc-wf-studio/
├── packages/               # pnpm monorepo
│   ├── core/               # 共享类型、Mermaid 生成器、Schema
│   ├── mcp/                # MCP 服务端（stdio 模式）
│   ├── cli/                # ccwf CLI 命令（render/validate/export）
│   └── vscode/             # VSCode 扩展 + React 画布 UI
│       └── src/webview/    # React Flow 可视化画布
├── .claude/skills/         # 6 个 Claude Code skill（主要 NL 工件）
│   ├── pr-to-production/SKILL.md    # 95分，最佳范例
│   ├── pr-review-analysis/SKILL.md  # 93分
│   ├── pr-to-main/SKILL.md          # 80分，缺示例
│   ├── pr-to-main-cleanup/SKILL.md  # 75分，最低分
│   ├── jira-driven-planning/SKILL.md
│   └── workflow-schema-tuning/SKILL.md
├── .github/skills/         # GitHub Copilot 适配
│   └── cc-workflow-ai-editor/SKILL.md   # 88分（与 .roo/ 完全一致）
├── .roo/skills/            # Roo Code 适配（与 .github/ 内容相同）
│   └── cc-workflow-ai-editor/SKILL.md
├── .specify/scripts/bash/  # ⚠️ 高危：含 eval 注入漏洞的 shell 脚本
├── .mcp.json               # MCP 配置（npx -y mcp-remote，未锁版本）
└── CLAUDE.md               # 98分，项目主上下文文档
```

- **文件类型分布**：8 个 NL 工件（6 SKILL.md + 1 CLAUDE.md + 1 跨工具 skill）
- **编排关系**：skill 之间平列；`cc-workflow-ai-editor` 在 `.github/` 和 `.roo/` 两处完全复制
- **跨件契约**：skill 通过 `assets/*.md` 引用 PR 模板；`references/planning-template.md` 被 jira 规划 skill 引用

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「把 AI 编辑能力嵌进视觉化工作流」—— 不是让 agent 理解工作流，而是让工作流画布本身理解 AI
- **解决什么问题**：Claude Code 的任务是线性对话，缺乏对"多步、分支、并行"工作流的视觉化表达；cc-wf-studio 用画布补这个缺口
- **Trade-off**：同一 skill（`cc-workflow-ai-editor`）在 `.github/` 和 `.roo/` 各放一份——增加了多 AI 工具覆盖，但引入了同步维护的成本
- **认知模型**：把 Claude Code 的 AI 能力看作可以被外部工具调度的"执行引擎"，而非对话伙伴

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「多工具 NL 适配层 + 核心产品分离」**

特征清单：
- 特征 1：产品代码（packages/）和 NL 工件（.claude/ .github/ .roo/）物理分离，各自独立
- 特征 2：同一 skill 在多个 AI 工具路径下保持**完全一致**的副本（设计意图）
- 特征 3：CLAUDE.md 是项目级上下文，skill 是工作流级协议，两层各司其职
- 特征 4：skill 依赖 assets/ 模板文件，而非把模板内联在 SKILL.md 里
- 特征 5：安全执行面（shell 脚本）与 NL 工件完全分离，但两者都在同一仓库

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要同时支持 Claude/GitHub Copilot/Roo 的项目 | ✅ 适用 | 多路径适配设计完全吻合 |
| PR 工作流自动化 | ✅ 适用 | pr-to-production/SKILL.md 是高质量范例 |
| 包含可执行 shell 脚本的仓库 | ⚠️ 高风险 | 安全审核必须先于 NL 质量优化 |
| 仅 Claude Code 用户的简单项目 | ❌ 过度设计 | 三套 skill 路径的维护成本得不偿失 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多工具适配层（本仓库）| cc-wf-studio | 覆盖多 AI 工具 | 同一 skill 多副本，同步成本高 |
| 单一 .claude/ 路径 | axtonliu/skills | 维护简单，零同步成本 | 只支持 Claude Code |
| 统一 skill 生成分发 | caliber-ai-org/ai-setup | 从单一源生成多平台 | 实现复杂，见 caliber 案例 |

### 2.4 改进空间
1. **当前问题**：`cc-workflow-ai-editor` 在 `.github/` 和 `.roo/` 各保留一份完全相同的文件，任何改动都需要手动同步两处。**改进做法**：设立规范源（canonical source）在 `.claude/skills/`，用构建脚本复制到 `.github/` 和 `.roo/`，或改用 symlink。**预期收益**：消除同步漂移风险（参见 caliber 案例的教训）。
2. **当前问题**：`pr-to-main-cleanup/SKILL.md`（75 分）缺示例且无输出格式规范。**改进做法**：添加一个"成功完成 cleanup 后应输出什么"的格式示例，即使是伪数据也行。**预期收益**：R06 扣分从 -25 降为 0，agent 执行后知道任务完成的样子。
3. **当前问题**：`.mcp.json` 用 `npx -y mcp-remote`（无版本锁定）。**改进做法**：指定具体版本如 `mcp-remote@0.x.y`。**预期收益**：消除供应链漂移风险，安全评分从 Medium 降为 Clear。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 得分 **87/100**，安全状态 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/skills/pr-to-main-cleanup/SKILL.md | 75 | 无示例；无输出格式规范 |
| .claude/skills/jira-driven-planning/SKILL.md | 80 | 无示例；Output Requirements 只描述内容不描述格式 |
| .claude/skills/pr-to-main/SKILL.md | 80 | 无示例；PR body 格式未规定 |
| .github/skills/cc-workflow-ai-editor/SKILL.md | 88 | 只有 1 个领域示例，缺用户可见输出格式 |
| .claude/skills/pr-review-analysis/SKILL.md | 93 | 示例用占位符而非真实数据 |
| .claude/skills/pr-to-production/SKILL.md | 95 | 近乎完美 |
| CLAUDE.md | 98 | "Follow standard conventions"略模糊 |

### 3.2 当时值得借鉴的模式
1. **pr-to-production/SKILL.md（95分）** → 仓库最佳范例。其写法特点：步骤明确（numbered steps）、条件分支清晰（"if staging exists → X; else → Y"）、资产文件引用格式化（`assets/pr-template.md`）。借鉴：PR 相关 skill 应当把触发条件、执行步骤、输出格式三件事各自独立成段。

2. **assets/ 模板解耦** → PR body 格式存在 `assets/pr-template.md`，不塞进 SKILL.md 主体。借鉴：模板类内容适合独立存档，skill 主体保持精简。

3. **CLAUDE.md 作为项目总上下文（98分）** → 高质量的 CLAUDE.md 弥补了个别 skill 的不足，让 agent 理解整体项目约束。借鉴：CLAUDE.md 质量应与 skill 质量并重，甚至更重要。

4. **多工具路径有意覆盖** → 明确在架构层面支持 Claude/GitHub Copilot/Roo，在 README 中标注，不是意外出现的重复文件。借鉴：有意为之的重复需要文档说明，否则日后维护者不敢修改。

5. **安全执行面审计先于 NL 质量** → BLOCKED 状态让 NLPM 在所有 NL 质量修复之前优先处理安全问题。借鉴：自己的仓库如果有 shell 脚本，安全自审应在 NL 质量优化之前进行。

### 3.3 当时的缺陷
1. **R06：三个 skill 完全无示例**（pr-to-main-cleanup/jira-driven-planning/pr-to-main）→ 根本原因：这些 skill 描述的是"agent 应该做什么"，但没有"做完后输出什么样"的示例——agent 执行完不知道任务是否成功。自查：我的 `echo-sleuth/skills/memory-management/SKILL.md` 也是 0 个代码块，同类问题。

2. **High 安全：三处 `eval $(get_feature_paths)`** → 根本原因：`SPECIFY_FEATURE` 环境变量来自外部，如果在 CI/CD 中被攻击者控制，可以注入任意 shell 命令。`eval $( )` 是 shell 脚本中最危险的模式之一：输入先被解释为命令，再被执行。自查：我的仓库目前无 shell 脚本，风险为零——但这是将来添加脚本时的红线。

3. **同一 skill 两处完全相同的副本**（`.github/` 和 `.roo/` 的 `cc-workflow-ai-editor`）→ 根本原因：为了快速支持多工具，直接复制文件，没有建立同步机制。caliber 案例已经证明跨平台副本必然漂移——这里只是还没漂移，时间问题。自查：我无此问题（没有多平台适配需求），但将来如果要适配 Cursor 或 Codex，必须考虑同步机制。

### 3.4 当时的优化机会
1. 修复 3 处 eval 注入：用 `read` 解析 `get_feature_paths` 输出，或用 `declare` 赋值，完全消除 eval（安全优先）
2. 为 pr-to-main-cleanup、jira-driven-planning、pr-to-main 各添加一个 3-5 行的示例输出片段
3. 给 `.mcp.json` 的 `mcp-remote` 锁定具体版本

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| High：`eval $(get_feature_paths)` 注入 | `grep -n "eval" .specify/scripts/bash/check-prerequisites.sh` | **持续**（第 82 行原样保留）| 高危漏洞未修复，audit BLOCKED 状态仍然有效 |
| R06：pr-to-main-cleanup 无示例 | `grep -c '^\`\`\`' .claude/skills/pr-to-main-cleanup/SKILL.md` | **持续**（当前 HEAD 目录未改变）| NL 质量问题未处理 |
| `.mcp.json` mcp-remote 无版本锁 | `cat .mcp.json` 中 `npx -y mcp-remote` 无版本号 | **持续**（已确认存在）| 供应链风险持续 |

**结论**：所有高优先级缺陷均持续存在，无修复迹象。BLOCKED 状态在本次生成日期（2026-05-30）仍然有效——这意味着 NLPM 不应向该仓库发起 PR，直到 eval 注入被私下披露并修复。

### 4.2 架构演进
当前 HEAD 增加了几个值得关注的新文件：
- 新增 `SECURITY.md`：说明安全披露流程（讽刺的是，项目虽有 SECURITY.md 但安全漏洞未修复）
- 新增 `.specify/memory/constitution.md`：agent 行为约束文件，类似 AGENTS.md 的作用
- 新增 `.claude/skills/workflow-schema-tuning/SKILL.md`：Audit 后新增的 skill，覆盖工作流 schema 调整

说明作者在 Audit 后继续扩充 NL 工件，但未回应安全问题。

### 4.3 新增的可学习模式
- **constitution.md 模式**：`.specify/memory/constitution.md` 是一个专门用于约束 agent 行为的"宪法文件"，比 CLAUDE.md 更专注于 agent 自我约束（如"不得修改……""必须先……"）。这是比普通 CLAUDE.md 更强的行为约束层。

---

## 五、校准

### 5.1 我已经在做对的
1. **零执行面**：我的仓库没有 shell 脚本，eval 注入风险为零
2. **pr-to-production 式的步骤清晰度**：我的 `claude-for-legal/product-legal/skills/launch-review/SKILL.md` 同样用 numbered steps 结构
3. **CLAUDE.md 重视程度**：我理解 CLAUDE.md 作为总上下文的重要性，质量投入不低于 skill

### 5.2 挑战 / 验证
这次案例给出了一个清晰的教训：**高分 CLAUDE.md + 中低分 skill 的组合，是"文档优先、行为约束后置"的典型症状**。

CLAUDE.md 98 分，但三个 skill 得了 75/80/80。说明作者在写项目总结时很认真，但在写"让 agent 可执行的操作规程"时不够精细。

对我自己的提示：**检查 CLAUDE.md 和 skill 之间的分数差**。如果 CLAUDE.md 远高于 skill，说明我在"描述"而非"规范"。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的仓库是否有 shell 脚本，以及是否使用 eval
find /tmp/my-repos/MarkQWu-* -name "*.sh" | xargs grep -l "eval" 2>/dev/null || echo "无 eval 风险"
```
命中后怎么办：立即用 `read` 或 `declare` 替换 eval，或完全改写为不依赖 shell 展开的逻辑。

```bash
# 检查我的 .mcp.json 是否有版本锁定
find /tmp/my-repos/MarkQWu-* -name ".mcp.json" | xargs grep "npx -y" 2>/dev/null
```
命中后怎么办：给 `npx -y` 后面的包名加上具体版本号。

```bash
# 检查我的 skill 中是否有零示例的 SKILL.md（同类 R06 问题）
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" | while read f; do
  blocks=$(grep -c "^\`\`\`" "$f" 2>/dev/null || echo 0)
  [ "$blocks" -eq 0 ] && echo "ZERO EXAMPLES: $f"
done
```
命中后怎么办：添加 2-3 行 "## Examples" 段，哪怕是伪代码。

### 6.2 灵感 → 实施路径
1. **想法**：为 echo-sleuth 添加 `memory-management/SKILL.md` 的 Examples 段
   - **为何可行**：该 skill 0 个代码块，功能操作明确（保存/检索记忆），有固定调用格式
   - **第一步**：写 2 个 snippet：一个"如何触发保存记忆"，一个"触发后预期输出格式"，约 10 分钟

2. **想法**：为 drama-workshop-skills 添加 `.specify/memory/constitution.md` 风格的约束文件
   - **为何可行**：短剧创作中有一些"绝对不做的事"（如不写现实人物、不改变核心人设），用 constitution 而非分散在各 skill 中约束更集中
   - **第一步**：建 `.claude/constitution.md`，列出 5 条 agent 行为红线，约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **cc-wf-studio 的核心目的**：可视化编排 Claude Code 工作流，适配多 AI 工具，自动化 PR 管理流程
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 都是 Claude Code 相关工具 | echo-sleuth 是 mining 工具，cc-wf-studio 是工作流编排 | 低 |
| MarkQWu/drama-workshop-skills | 无 | — | 领域完全不同 | 低 |
| MarkQWu/claude-for-legal | 低 | 都有工作流自动化场景（PR vs 法律 matter） | 目标用户不同 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| R06：SKILL.md 无示例 | `find . -name "SKILL.md" -exec grep -L '^\`\`\`' {} \;` | **echo-sleuth `memory-management/SKILL.md` 命中**：0 个代码块 | 中 |
| shell 脚本 eval 注入 | `find . -name "*.sh" -exec grep -l "eval" {} \;` | **我的仓库无命中**（无 shell 脚本） | 无 |

### 8.3 别人的更优方案

1. **领域**：assets/ 模板文件独立存放（解耦输出格式与步骤描述）
   - **cc-wf-studio 做法**：PR body 模板存在 `.claude/skills/pr-to-main/assets/pr-template.md`，skill 主体通过引用路径指向它
   - **我的现状**：`claude-for-legal/litigation-legal/skills/demand-draft/SKILL.md` 把格式模板内联在 SKILL.md 里（约 30 行的格式规范嵌在正文）
   - **如何借鉴**：把模板段提取到 `assets/demand-template.md`，skill 主体保留一行引用

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全表面控制
- **我的做法**：`echo-sleuth` 和 `drama-workshop-skills` 均无 shell 脚本、无 eval、无 npm 依赖，安全表面为零
- **cc-wf-studio 做法**（弱在哪）：3 处 eval 注入高危漏洞持续存在，audit BLOCKED 状态持续
- **意义**："零执行面"不是懒惰，而是一种主动的安全设计决策——能用 NL 工件解决的问题就不引入可执行代码

---

## 八、术语表

### <a name="mcp"></a>MCP（Model Context Protocol）
> Anthropic 定义的开放协议，让 AI 模型通过标准化接口调用外部工具（文件系统、数据库、API 等）。cc-wf-studio 的 `.mcp.json` 配置了一个 MCP 服务，让 Claude Code 能调用工作流画布上的工具节点。

### <a name="eval-injection"></a>eval 命令注入
> 一种 shell 脚本安全漏洞。`eval $( )` 会先把括号内容当成命令执行，再把输出当成 shell 命令再次执行。如果括号内的内容来自不可信的环境变量（如用户输入、CI/CD 参数），攻击者可以注入任意系统命令，包括删文件、窃取密钥、建立反弹 shell。

### <a name="pnpm-monorepo"></a>pnpm monorepo
> 用 pnpm（Node.js 包管理器）管理的"单仓多包"结构。cc-wf-studio 把 core/mcp/cli/vscode 四个模块放在同一个 git 仓库里，通过 workspace 机制共享依赖，统一构建和发版。

### <a name="roo-code"></a>Roo Code
> 一款基于 Claude Code 的 AI 编程助手扩展，支持自定义模式和 skill 文件，和 Claude Code 使用类似的 SKILL.md 格式。cc-wf-studio 在 `.roo/skills/` 下放置适配文件，使其 AI 编辑功能也能被 Roo Code 使用。

### <a name="react-flow"></a>React Flow
> 用于构建节点连线图（流程图/画布）的 React 组件库。cc-wf-studio 用它在 VSCode 侧边面板中渲染工作流画布，每个节点代表一个工作流步骤。
