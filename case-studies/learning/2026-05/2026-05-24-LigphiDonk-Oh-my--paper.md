# LigphiDonk/Oh-my--paper — 学习案例

**仓库**：https://github.com/LigphiDonk/Oh-my--paper
**Stars**：506 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-30（历史快照）| **生成日期**：2026-05-24（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `security-gate`, `single-purpose`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Oh My Paper（缩写 OMP）是一套**学术论文全流程研究框架**，由 LigphiDonk（大概率中国开发者，界面/注释全为中文）构建。核心定位：让 Claude Code 充当研究团队——从 idea 生成、文献调研，到实验设计、论文撰写，一套命令链走完。仓库有两个形态共存：

1. **Tauri 桌面应用**：`src-tauri/` 目录下有完整 Rust/JavaScript 桌面 App，35 个 SKILL.md 嵌入其中作为 `resources`。
2. **Claude Code 插件**：`plugins/oh-my-paper/` 下有 10 个 agent + 9 个 command，作为可安装的 `/omp:*` 命令集。

另有 `templates/harness/` 提供可独立使用的无前缀 templates 版本。仓库同时维护这两个版本，跨平台（桌面+Claude Code）覆盖，但也因此带来了严重的维护分裂问题。

### 1.2 架构剖析
- **目录结构**：
  ```
  Oh-my--paper/
  ├── plugins/oh-my-paper/
  │   ├── agents/          # 5 个 agent（conductor, reviewer, literature-scout, experiment-driver, paper-writer）
  │   ├── commands/        # 9 个命令（/omp:setup, /omp:plan, /omp:ideate, /omp:survey, /omp:experiment, /omp:write, /omp:review, /omp:delegate, /omp:sync）
  │   ├── hooks/           # hooks.json（SessionStart + Stop + PostToolUse:Write）
  │   └── plugin.json
  ├── templates/harness/
  │   ├── agents/          # 5 个 agent（同上内容的无前缀版本）
  │   ├── commands/        # 7 个命令
  │   └── templates/research/CLAUDE.md
  ├── src-tauri/resources/skills/  # 35 个 SKILL.md（Tauri 桌面端资源）
  └── skills/                      # 34 个 SKILL.md（镜像副本，已开始漂移）
  ```
- **文件类型分布**：10 个 agent / 16 个 command（含模板和插件两个版本）/ 35 个 skill / 3 个 hook 脚本（未公开）/ 1 个 [manifest](#manifest)
- **编排关系**：[conductor](#conductor) agent 是中枢，负责询问用户选择工作模式，然后分发给 4 个专职 agent。command 层是用户入口，调用 `/codex:rescue` 来分发任务（依赖一个外部未声明的 codex 插件）。
- **跨件契约**：所有命令都依赖 `@references/trace_schema.md`、`@templates/session_summary.md` 等在 skill 安装后才存在的路径。commands 和 hooks.json 重复注册同一个 SessionStart hook，造成双写风险。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「全自动研究工厂」——把博士阶段的研究流程（idea→文献→实验→写作→审稿）完整编排给 AI 多 agent 执行，每步有检查点（AskUserQuestion gate）但不需要人工深度干预。
- **解决什么问题**：独立研究者（可能是 PhD 学生）在独立探索科研课题时缺乏有经验的同行审阅人和实验辅助。用 AI 多 agent 模拟同行环境。
- **做了什么 trade-off**：选择同时维护 Tauri 桌面版和 Claude Code 插件版，追求两端覆盖，代价是 skills 目录出现两份副本且已开始漂移（已 diverge：`skills/literature-pdf-ocr-library/` 在 Tauri 版本中不存在，2 个 JSON 文件内容已不同）。
- **反映什么认知模型**：作者把 Claude 当作「会议室里的多个专家」——不同角色（conductor/reviewer/literature-scout）有各自专长，通过标准化的 pipeline（`.pipeline/` 目录）传递状态。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[多 agent 专家委员会](#多agent专家委员会)」（Multi-Agent Expert Panel 架构）**

一个 conductor agent 作为主持人，负责工作流路由；多个专职 agent 各管一个领域（文献/实验/写作/审稿）；通过 `.pipeline/tasks.json` 等状态文件跨 agent 传递进度。

模式特征清单：
- **角色分工明确**：每个 agent 只做一件事，conductor 不做具体工作
- **状态持久化**：`.pipeline/` 目录是跨 session 的工作状态载体
- **命令入口分离**：用户用 `/omp:*` 命令触发工作流，不直接调用 agent
- **跨会话 resume**：`/omp:sync` 和 conductor 的会话启动逻辑设计了断点续传
- **平台双维护**：同一套 skills 维护了桌面端（Tauri resources）和插件端（skills/）两份副本

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 独立 PhD / 研究员的日常科研流程 | ✅ 高度适用 | 正是设计对象，覆盖 idea 到成稿 |
| 需要同行评审反馈的论文写作 | ✅ 适用 | reviewer agent 专门处理这个需求 |
| 团队协作论文（多人同时改一稿） | ❌ 不适用 | `.pipeline/tasks.json` 不支持并发写入 |
| 快速 one-shot 内容生成 | ❌ 不适用 | 框架有 9 个步骤，overhead 大 |
| 工业界项目（非学术） | ⚠️ 部分适用 | skills 全为学术场景设计，迁移成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多 agent 专家委员会（本仓库） | Oh My Paper | 专职分工，复杂任务可并行，resume 机制 | 安装复杂，依赖外部 codex 插件，10 个 agent 全不可注册 |
| 单 skill 大型 orchestrator | KhazP/vibe-coding | 安装简单，一个 SKILL.md 驱动全流程 | 单文件超 500 行，难以单独优化某个阶段 |
| Router + Channels 分层 | 0xmariowu/Autosearch | 按意图路由，channels 可独立扩展 | 初始配置复杂 |

### 2.4 改进空间
1. **当前问题**：技能库同时维护两份副本（`src-tauri/resources/skills/` + `skills/`），已发现漂移（2 个 JSON 文件不同，`skills/` 多出一个 `literature-pdf-ocr-library` 目录）。**改进做法**：在 `src-tauri/` 构建脚本里加入符号链接或 CI diff 检查，确保两份相同；或者干脆指定 `skills/` 为唯一真相，`src-tauri` 构建时从那里复制。**预期收益**：消除双写风险，避免桌面版和插件版行为不同。
2. **当前问题**：所有命令依赖 `/codex:rescue` 但未在 [manifest](#manifest) 中声明依赖。**改进做法**：在 `plugin.json` 加 `"dependencies": ["codex"]`，并在 `/omp:setup` 里检查 codex 是否已安装，若未安装则提示。**预期收益**：用户安装 OMP 后直接报错"找不到 codex 命令"，原因不明；声明依赖后体验更清晰。
3. **当前问题**：`plugins/oh-my-paper/commands/setup.md` 里内嵌 `node -e` 脚本修改 `.claude/settings.json`，同时 `hooks/hooks.json` 里也注册了同一个 SessionStart hook。**改进做法**：删除 setup.md 里的 node 脚本，让 hooks.json 作为唯一的 hook 注册点。**预期收益**：消除重复注册风险和 shell injection 向量（`${CLAUDE_PLUGIN_ROOT}` 路径注入）。

---

## 三、过去审查发现（2026-04-30 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-30 审计时**综合得分 65/100**，有大量关键性缺陷拉低了分数。

| 文件类型 | 当时均分 | 主要问题 |
|---|---|---|
| Agent 文件（10 个） | 30/100 | 全部缺少 YAML [frontmatter](#frontmatter)（name/description/model/examples 一项都没有） |
| Command 文件（16 个） | 60/100 | 缺少 `name` 字段 |
| Skill 文件（src-tauri, 35 个） | 77/100 | 整体质量尚可；部分文件缺 description、含个人内容 |
| CLAUDE.md | 65/100 | 角色不清晰（是 agent 文件还是 project rules？） |

### 3.2 当时值得借鉴的模式

1. **`AskUserQuestion` gate 贯穿命令链** → 每个命令的关键决策点都用 `AskUserQuestion` 工具确认，而不是让 AI 自行猜测用户意图。根本原因好：多步骤任务里 AI 的假设可能在第 3 步才暴露错误，早确认省大成本。原文示例路径：`plugins/oh-my-paper/commands/ideate.md` step 1-2。如何借鉴：我的多步骤命令在第一步就应该问清楚 scope，而不是在最后才问"你满意吗？"。

2. **inno 系列 skill 的例子质量极高** → `inno-idea-generation/SKILL.md` 有 2 个完整 SCAMPER/SWOT/Mind Map 例子，`inno-grant-proposal/SKILL.md` 覆盖 NSF/NIH/NSFC 三种格式。对应 R08 规则"examples 是一等公民"。如何借鉴：我的 skill 经常只写"Input: X, Output: Y"这种伪例子，应该像 OMP inno 系列一样给完整的 input→output 配对。

3. **conductor agent 的角色定位清晰** → conductor 专注做决策路由（询问工作模式→分发给对应角色），不参与具体写作。体现了单职责原则：orchestrator 不应该干 worker 的活。如何借鉴：我的一些命令文件同时做"理解用户意图"和"执行任务"两件事，应该分离。

4. **`.pipeline/` 状态目录设计** → 跨 session 状态用文件系统持久化（`tasks.json`, `research_brief.json`），而不是依赖 AI 的内存。保证断点续传的可靠性。如何借鉴：echo-sleuth 如果要支持长任务，应考虑类似的状态文件设计。

### 3.3 当时的缺陷

1. **[BUG-01] 全部 10 个 agent 文件缺少 YAML frontmatter** → 根本原因：作者把 agent 文件当成了"给人看的文档"而不是"给 Claude Code 注册的机器可读文件"。没有 `name:` 字段，Claude Code 就无法在 plugin registry 里找到这个 agent，`/omp:setup` 安装后调用任何 agent 都会静默失败。自查：我的 echo-sleuth 的 agent 文件**有** frontmatter（recall.md 开头就是 `---\nname: recall\ndescription: ...`）——这一点我做对了。

2. **[BUG-02] 全部 16 个 command 文件缺少 `name` frontmatter 字段** → 根本原因：可能是从某个 Markdown 文档模板迁移过来，原始文档不需要 `name` 字段，迁移时忘了补。结果是 `/omp:plan` 这类命令的斜杠路径无法注册。自查：我的 echo-sleuth 命令**有** `name:` 字段，但有 4 个命令缺 `allowed-tools`（lessons.md, recall.md, recap.md, timeline.md）——我有类似但轻量版的问题。

3. **[SEC-H1/H2] `--dangerously-skip-permissions` 和 `--approval-mode full-auto`** → 根本原因：追求"全自动"效果，直接把最宽松的权限模式写进了 skill 指令里。任何读取这个 skill 的 agent 都会绕过 Claude Code 的工具权限检查。危险：如果用户的工作目录有敏感文件，agent 可以在用户不知情的情况下读写。自查：我没有在 skill 或 command 里使用这两个 flag，这是正确的。

### 3.4 当时的优化机会
1. **给 `bioinformatics-init-analysis/SKILL.md` 补充真正的 description**（当前只有 Markdown heading 作为 description）
2. **将 `inno-rclone-to-overleaf/SKILL.md` 里的个人内容（"Eason's Workflow Requirements"）参数化**，替换为 `<your-rclone-remote>` 占位符
3. **`skills/SKILL.md` 删掉结尾商业广告**（"K-Dense Web" 推广内容不应出现在技术 skill 定义里）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-01: agents 缺 frontmatter | `head -5 plugins/oh-my-paper/agents/conductor.md` | **仍然存在** — conductor.md 开头直接是 `# Oh My Paper Conductor（统筹者）`，无 frontmatter | 插件安装后所有 agent 依然无法注册，这是 p0 级别的问题，维护者近一个月未修复 |
| BUG-02: commands 缺 `name` 字段 | `grep -l "^name:" plugins/oh-my-paper/commands/*.md` | **仍然存在** — 9 个 command 文件中有 name 的为 0 个 | 命令注册失败，`/omp:plan` 等命令安装后不可用 |
| SEC-H3: setup.md inline node -e | `grep -n "node -e" plugins/oh-my-paper/commands/setup.md` | **仍然存在** — 第 58 行仍有 `node -e "..."` 脚本修改 settings.json | 安全风险未消除；如果 `CLAUDE_PLUGIN_ROOT` 路径含特殊字符将造成命令注入 |
| CC-01: skills 目录重复 | `diff -rq src-tauri/resources/skills/ skills/` | **已漂移** — `skills/literature-pdf-ocr-library/` 在 src-tauri 版本中不存在；2 个 JSON 文件内容已不同 | 预测命中：维护者在某次更新中只改了一侧，导致两端行为将持续分裂 |

### 4.2 架构演进
从 audit 到现在，主要变化是 `skills/` 目录新增了 `literature-pdf-ocr-library` 子目录（src-tauri 版本未同步），说明维护者在继续新增功能但没有解决底层的双副本问题。核心 bug（agent/command frontmatter 缺失）约一个月内未动，反映维护者可能不知道 Claude Code plugin 注册机制的要求，或认为"内容质量"比"格式规范"更优先。

### 4.3 新增的可学习模式
从当前 HEAD 新增的 `skills/literature-pdf-ocr-library/` 可以看出维护者在扩展 PDF 处理能力（与 `src-tauri/resources/skills/inno-paper-writing/` 的 PDF 依赖呼应），这是 research workflow 的自然延伸。该新 skill 的质量未经 NLPM 审计，但从文件名判断是独立功能模块，符合单职责原则。

---

## 五、校准

### 5.1 我已经在做对的
1. **Agent frontmatter 完整**：echo-sleuth 的所有 agent 文件都有 `name:` + `description:` 字段，与 OMP 的 30/100 agent 相比，这是我们的明显优势。
2. **Command frontmatter 完整**：echo-sleuth 命令文件有 `name:` 和 `description:`，不像 OMP 的 commands 缺 `name` 导致命令无法注册。
3. **没有 `--dangerously-skip-permissions`**：我的所有 skill/command 都没有使用权限绕过 flag，安全基线保持了。
4. **没有内联 node 脚本修改配置文件**：echo-sleuth 的 hook 注册通过 hooks.json 完成，而不是 command 里的副作用。
5. **用了 AskUserQuestion**：echo-sleuth 的 recall/audit 命令在关键决策点使用 AskUserQuestion gate，与 OMP 的好实践一致。

### 5.2 挑战 / 验证
- **挑战了的假设**："506 star 的仓库应该没有 p0 级别的功能性 bug"——OMP 否证了这个假设。一个项目可以有非常丰富的内容（35 个 skill，完整学术流程覆盖）同时有全部 agent 无法注册的根本性问题。内容质量和工程规范是两个正交维度。
- **验证了的做法**："维护两份相同内容副本一定会漂移"——CC-01 的预测一个月内就命中了，`skills/` 已经跑偏。单一真相原则（single source of truth）的重要性被验证。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 echo-sleuth agent 是否都有 model 字段
grep -L "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md
# 命中则补充 model 字段（召回类用 claude-haiku-4-5-20251001，分析类用 claude-sonnet-4-6）

# 检查 echo-sleuth 命令是否都有 allowed-tools
grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
# 命中则按命令实际工具使用情况添加 allowed-tools frontmatter

# 检查我的项目是否有双副本问题（多余的 skills/ 拷贝）
find ~/claude-projects -name "SKILL.md" -newer ~/claude-projects/*/skills/ 2>/dev/null | head -5
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 所有 agent 添加 `model:` 字段
   - **为何可行**：echo-sleuth 5 个 agent 用途明确；schema-scout 是 Haiku 级别机械任务，recall/analyze 需要 Sonnet
   - **第一步**：打开 `/tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/`，给 schema-scout 加 `model: claude-haiku-4-5-20251001`，给 recall/analyze/memory-auditor/file-historian 加 `model: claude-sonnet-4-6`（约 10 分钟）

2. **想法**：给 echo-sleuth 4 个缺 `allowed-tools` 的命令补字段
   - **为何可行**：每个命令用的工具可以从 body 里分析出来（recall 用 Bash 调脚本，lessons 用 Read+Bash）
   - **第一步**：在 `commands/recall.md`、`lessons.md`、`recap.md`、`timeline.md` 的 frontmatter 里逐一添加 `allowed-tools`，参考 audit.md 和 dashboard.md 已有的写法（约 15 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 LigphiDonk/Oh-my--paper 的核心目的**：多 agent 协作驱动学术研究全流程（idea → literature → experiment → writing → review）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都使用多 agent 分工（coordinator/worker 模式），都有持久化状态的设计意图 | echo-sleuth 是"回顾分析"型而非"生产输出"型；OMP 的任务更重量级 | 高（架构模式直接可借鉴） |
| MarkQWu/claude-for-legal | 低 | 都有多专业子领域 skill 分工 | claude-for-legal 是单人工具，无 agent 编排层 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都是创作工作流 | 短剧是单一 skill 架构，无 multi-agent | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 缺 `model:` 字段 | `grep -L "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md` | **全部命中**（5/5 agent 缺 model）：analyze.md, file-historian.md, memory-auditor.md, recall.md, schema-scout.md | 中（功能不会失效，但 model 选择不确定） |
| Command 缺 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **4 个命中**：lessons.md, recall.md, recap.md, timeline.md | 中（某些 Claude Code 配置下工具调用可能需要额外确认） |
| Skills 目录双副本漂移 | `diff -rq my-skills-dir-1/ my-skills-dir-2/` | **不适用**：我的项目没有双副本设计 | 暂无 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/agents/schema-scout.md` → frontmatter 加 `model: claude-haiku-4-5-20251001` → 5 分钟
- `echo-sleuth-for-claude/agents/recall.md` + `analyze.md` + `memory-auditor.md` + `file-historian.md` → 各加 `model: claude-sonnet-4-6` → 10 分钟
- `echo-sleuth-for-claude/commands/recall.md` → 加 `allowed-tools: Bash, Read` → 3 分钟
- `echo-sleuth-for-claude/commands/lessons.md` → 加 `allowed-tools: Bash, Read` → 3 分钟
- `echo-sleuth-for-claude/commands/recap.md` + `timeline.md` → 各加 `allowed-tools: Bash, Read` → 5 分钟

### 7.3 别人的更优方案

1. **领域**：多 agent 专职分工 + conductor 路由模式
   - **本案例做法**：conductor.md 专门做"询问工作模式→路由到对应 agent"，不做任何具体研究工作；每个专职 agent（reviewer/literature-scout/experiment-driver/paper-writer）职责单一。
   - **我的项目现状**：echo-sleuth 的 `recall.md` agent 同时负责"搜索会话"和"分析决策考古"和"定位错误"三种任务（虽然有细分条件），比 OMP 的粒度粗。
   - **如何借鉴**：考虑把 recall agent 拆分为 `search-agent.md`（机械搜索，Haiku）+ `analyze-agent.md`（深度分析，Sonnet），recall command 做路由层，按用户意图分发。

2. **领域**：`.pipeline/` 状态文件实现跨 session 断点续传
   - **本案例做法**：`tasks.json`、`research_brief.json` 在 `.pipeline/` 目录里持久化，session 结束后下次可以 `/omp:sync` 恢复。
   - **我的项目现状**：echo-sleuth 没有任务状态持久化，每次 `/recall` 都是无状态的新搜索。
   - **如何借鉴**：如果 echo-sleuth 要支持长期知识积累任务，可以在 `.echo-sleuth/state.json` 里记录上次提取的教训 + 时间戳，实现增量提取。

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Agent frontmatter 完整性
- **我的做法**：echo-sleuth-for-claude 的所有 agent 都有 `name:` + `description:` + 完整 examples block（recall.md 的 description 有 5 行和 3 个 examples）
- **本案例做法**（弱在哪）：10 个 agent 全部缺少 frontmatter，等于 plugin 安装后 agent 完全不可用
- **意义**：echo-sleuth 在 agent 可注册性上保持了 p0 基线；未来做 audit 时这是一个加分项

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 SKILL.md / agent.md 时先解析 frontmatter 才能知道这个组件怎么注册和调用。没有 frontmatter 的 agent 就像没有名字牌的员工——系统找不到他。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉 Claude Code 插件系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="conductor"></a>conductor
> 在多 agent 架构中担任"指挥家"或"主持人"角色的 agent。conductor 负责理解用户意图、决定把任务交给哪个专职 agent，自己不做具体工作。Oh My Paper 的 conductor agent 会在每次会话开始时询问用户"是继续已有项目还是新建？"，然后切换到对应角色。

### <a name="多agent专家委员会"></a>多 agent 专家委员会
> 一种 NL 编排架构：有一个专门做路由的 orchestrator agent，加上若干专职 worker agent，通过持久化状态文件传递任务上下文。类比：项目经理（conductor）+ 各专业工程师（worker agents），项目文档（.pipeline/）是他们之间的共享信息载体。
