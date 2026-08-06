# aaronmaturen/claude-plugin — 学习案例

**仓库**：https://github.com/aaronmaturen/claude-plugin
**Stars**：1 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-08-06（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `template-design`, `examples-driven`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

一个面向 Angular/Django 全栈工程师的 Claude Code 插件，名为 `atm`，提供约 26 个专业命令（a11y 审计、Angular 架构分析、Django 安全审查、PR 审查等）和 2 个 skill 文件（bash、ai）。插件结构极简：命令平铺在 `commands/`，skill 放在 `.claude/skills/`，插件清单在 `.claude-plugin/plugin.json`。仓库目前 1 颗星，属于个人开发者私用工具对外开放。

### 1.2 架构剖析

- **目录结构**：
  ```
  .
  ├── .claude-plugin/
  │   └── plugin.json          # 插件清单
  ├── .claude/
  │   ├── skills/
  │   │   ├── ai/SKILL.md
  │   │   └── bash/SKILL.md
  │   └── ... (非 NL 文件)
  ├── commands/
  │   ├── a11y-audit.md
  │   ├── angular-architecture-audit.md
  │   └── ... (共 26 个命令)
  └── CLAUDE.md                # 仅两行 redirect 到 AGENTS.md
  ```
- **文件类型分布**：0 个独立 agent / 2 个 SKILL.md / 26 个 command / 0 个 hook
- **编排关系**：纯平铺命令集，无 router、无 meta skill、无 agent 编排。每个命令独立运行。
- **跨件契约**：`commands/` 中存在两个隐式契约：① `simplify.md` 和 `spike-investigation.md` 共用 `$REPORT_BASE` 变量约定（bash skill 有记录）；② `scaffold.md`、`spike-investigation.md` 等 4 个命令依赖 Context7 MCP 服务（在 plugin.json 中声明，但版本未固定）。但这些契约均为隐式，从未在命令文件的 frontmatter 中声明 `skills:` 字段。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：命令直接描述专业角色 + 任务，不抽象中间层。如 `angular-expert.md` = "你是 Angular 高级工程师，开始工作"。
- **解决什么问题**：复用作者自己在项目中积累的 Angular/Django 专业提示词，通过插件机制分发。
- **做了什么 trade-off**：选择了极简结构（无 agent 层、无 hook），代价是每个命令都是孤岛，缺乏系统性的 frontmatter 规范。
- **反映什么认知模型**：作者把 Claude Code 命令视为"高级提示词文件"而非"[NL 工件](#nl工件)"——他没有采用 YAML [frontmatter](#frontmatter) 声明接口，说明对 NLPM 注册机制不了解或未将插件注册当作必要条件。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：专业命令平铺集（Expert Command Flat Pool）**

这是最简单的 Claude Code 插件模式：把多个独立命令文件放进 `commands/` 目录，配一个 `plugin.json` 清单。没有代理、没有编排，每个命令独立调用。

模式特征清单：
- 每个命令对应一个专业领域或任务（审计、专家角色、生成等）
- 命令间无显式依赖，可单独使用任意一条
- 结构极轻量：只需两层目录 + 一个清单文件
- 适合个人工具箱快速发布，不适合需要跨命令协作的工作流

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人工具集，命令彼此无依赖 | ✅ 高度适用 | 平铺结构最轻量，0 学习曲线 |
| 需要多命令协作的复杂工作流 | ❌ 不适用 | 无 router 和 agent 层，命令间无法传递上下文 |
| 快速原型/私用发布 | ✅ 适用 | 没有注册规范也能在本机跑，方便迭代 |
| 面向他人分发的正式插件 | ⚠️ 慎用 | 缺 frontmatter 导致 NLPM 无法注册，外部用户找不到命令 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 专业命令平铺集（本仓库） | aaronmaturen/claude-plugin | 极简，快速上手 | 无法注册，命令孤立 |
| Skill 层 + 命令分发 | gstack, bureau | skill 可复用，命令薄且聚焦 | 需维护 skill-command 契约 |
| Agent 网格 + 命令路由 | alirezarezvani/claude-skills | 覆盖面广，角色清晰 | 工程量大，维护成本高 |

### 2.4 改进空间

1. **当前问题**：26 个命令全部缺少 YAML frontmatter（`name`、`description`、`allowed-tools`）。**改进做法**：用脚本批量生成 frontmatter 框架，每个命令文件顶部加 3 行 YAML。**预期收益**：NLPM 分数从 36 提升到约 75，命令可被 NLPM scanner 正确注册。
2. **当前问题**：命令调用示例写成 `/atm-scaffold` 而实际注册路径是 `atm:scaffold`（冒号分隔）。**改进做法**：在 bash skill 里加一个"正确调用格式"速查表。**预期收益**：消除 BUG CC-terminology-drift，用户不会找不到命令。
3. **当前问题**：Context7 MCP 依赖版本未固定（`npx -y @upstash/context7-mcp`）。**改进做法**：在 `plugin.json` 里固定版本号 `@upstash/context7-mcp@X.Y.Z`。**预期收益**：消除供应链风险。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **36/100**（行业最低四分位）。均值 36，中值 29，范围 15–96。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/spike-investigation.md | 15 | 无 frontmatter + 无 empty input guard + 模糊量词上限 |
| commands/a11y-audit.md | 25 | 无 frontmatter + 模糊量词上限 |
| .claude/skills/ai/SKILL.md | 84 | 8 个模糊量词 |
| .claude/skills/bash/SKILL.md | 88 | 6 个模糊量词 |
| .claude-plugin/plugin.json | 96 | 清单描述含 2 个模糊词 |

### 3.2 当时值得借鉴的模式

1. **`$REPORT_BASE` 约定** → `commands/simplify.md` 和 `spike-investigation.md` 共用统一的报告输出路径变量。好处：命令输出物位置可预测，方便 CI 集成。示例：`simplify.md` 写 `$REPORT_BASE/simplify-report.md`。如何借鉴：自己的命令集里定一个统一输出目录约定。
2. **`CLAUDE.md` 到 `AGENTS.md` 的 redirect 模式** → CLAUDE.md 只有两行，转发给 AGENTS.md，避免两个地方维护配置。这是 ai/SKILL.md 里明确记录的模式，有意为之，不是 bug。如何借鉴：项目 CLAUDE.md 只做入口索引，真正的规范放 AGENTS.md 或 skill 文件。
3. **领域专精命令** → 26 个命令覆盖 a11y、Angular、Django、PR 审查等具体领域，每条命令职责清晰。如何借鉴：专题命令比通用命令好用——`angular-architecture-audit.md` 比 `audit.md` 更有用。

### 3.3 当时的缺陷

1. **所有 26 个命令缺 YAML frontmatter** → 为什么这么设计会失败：Claude Code 的 NLPM scanner 通过 frontmatter 的 `name` 和 `description` 字段识别并注册命令，缺失后命令在插件列表中不可见，用户安装插件后实际上什么都找不到。根本原因是作者把命令文件当普通 Markdown 提示词文件处理，没有意识到注册机制的存在。自查：我的 `bureau/commands/` 目录的命令文件有 frontmatter 吗？
2. **expert persona 命令无流程步骤** → `a11y-expert.md` 和 `angular-expert.md` 只设定了角色（"你是高级工程师"），没有编号步骤，模型收到的指令是"成为专家"，而不是"按 1→2→3 步做 X"。为什么这么设计会失败：AI 模型需要可重复的执行路径，persona 声明本身不够，缺乏步骤导致每次运行结果不一致。自查：我的命令文件有没有只有 persona 没有流程步骤的？
3. **模糊量词密度极高** → 87 处模糊量词在 30 个文件中，密度为每文件平均 2.9 处。"comprehensive"、"thorough"、"appropriate" 等词无法给模型提供可验证的完成标准，模型不知道什么程度算"够了"。自查：我的 skill 文件里有没有大量"ensure"、"appropriate"？

### 3.4 当时的优化机会

1. **批量生成 frontmatter**：用 Python/bash 脚本读每个命令文件的 `#` 标题和首段，自动生成 `name` 和 `description`，将分数从 36 拉到约 75。
2. **引入 `$REPORT_BASE` 约定给所有输出型命令**：现有只有 2 个命令使用，其余命令输出路径随机。统一后可建自动存档流程。
3. **为 expert persona 命令补充结构化步骤清单**：按角色对应的 checklist 模板加 3–5 个编号步骤。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 26 个命令全缺 frontmatter | `grep -rl "^---" commands/` → 14 有 frontmatter，12 无 | **部分修复**：12 个命令仍缺 frontmatter，14 个已补 | 作者开始修复但未完成；仍有约 46% 命令无法被 NLPM 注册 |
| Context7 MCP 版本未固定 | `grep "context7" .claude-plugin/plugin.json` | **持续**：`npx -y @upstash/context7-mcp` 仍无版本号 | 供应链风险未解除 |
| 命令调用示例写成 `/atm-scaffold` | `grep "/atm[-:]" commands/scaffold.md` | **持续**：`/atm-scaffold`（应为 `atm:scaffold`） | 文档与实际调用路径不符 |

### 4.2 架构演进

从 2026-04-29 的快照到现在：14/26 命令补充了 frontmatter，是最重要的进展。整体目录结构没有变化：仍是平铺命令集，无 agent 层引入，无 router。说明作者的修复是机械性的（补 frontmatter），而非架构性的（引入分层）——这没问题，因为架构本身对单人工具集是合适的。

### 4.3 新增的可学习模式

暂无新增值得记录的架构模式，14/26 frontmatter 修复是增量改进，非新模式引入。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau/commands/ 文件有 frontmatter**：检查发现 `lint.md` 和 `status.md` 都有 `description` 和 `argument-hint` 字段，是正确做法，避免了 aaronmaturen 最大的陷阱。
2. **命令有编号步骤或明确流程**：bureau 的命令体是有结构的，不是纯 persona 声明。
3. **gstack 的 skill 文件有具体 allowed-tools 声明**：`diagram/SKILL.md` 有完整的 `allowed-tools` 列表，比 aaronmaturen 的命令缺 allowed-tools 规范很多。

### 5.2 挑战 / 验证

这次案例**验证了**一个我一直半信半疑的假设：frontmatter 不是"nice to have"而是"must have"。看到 36/100 的实际分数——完全是因为 -55（name + description + allowed-tools 各缺）重复乘以 26 个文件——这个教训比任何理论都直观。「不写 frontmatter = 插件形同未安装」是本案例的核心结论。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 frontmatter
find ~/.claude/commands/ ~/my-projects -name "*.md" -path "*/commands/*" | \
  xargs grep -L "^---" 2>/dev/null
# 命中后怎么办：命中的文件是没有 frontmatter 的命令，手动补 name/description/allowed-tools
```

```bash
# 检查我的 plugin.json 里 MCP server 是否有版本固定
find . -name "plugin.json" | xargs grep -n '"args"' 2>/dev/null | grep "\-y "
# 命中后怎么办：把 npx -y @package 改成 npx -y @package@X.Y.Z
```

```bash
# 检查命令里的调用示例是否使用正确的 namespace:command 格式
grep -rn "^Run.*\/[a-z]" --include="*.md" .
# 命中后怎么办：把 /plugin-command 改成 plugin:command（冒号格式）
```

### 6.2 灵感 → 实施路径

1. **想法**：给我的命令集写一个 `$REPORT_BASE` 约定，所有输出文件写到统一目录。
   - **为何可行**：aaronmaturen 的 bash skill 里已有这个约定，说明是可落地的 pattern。
   - **第一步**：在 gstack 或 bureau 的 CLAUDE.md 中定义 `REPORT_BASE=.claude/reports`，然后修改 2–3 个输出型命令使用该变量。30 分钟内可完成。

2. **想法**：为所有 expert persona 类命令补充 3–5 步执行清单。
   - **为何可行**：persona + checklist 的组合在 alirezarezvani/claude-skills 中大量成功案例为证。
   - **第一步**：找到我的项目中有 "You are a..." 开头但缺步骤的命令，加一个 `## Steps` section。20 分钟可完成一个。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 aaronmaturen/claude-plugin 的核心目的**：个人工程技术工具集，覆盖 Angular/Django 全栈场景，以命令形式暴露专业经验。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 个人工具集，大量命令/skill，面向工程场景 | gstack 有 binary 核心和测试框架，aaronmaturen 纯 NL 文件 | 高 |
| MarkQWu/bureau | 中 | 有 commands/ 目录，命令覆盖知识管理场景 | bureau 有 hook 和 TypeScript 后端，更完整 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都是 skill 集合 | drama-workshop-skills 是社区策划集，非个人工具箱 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 frontmatter | `grep -L "^---" commands/*.md`（bureau 和 gstack） | bureau 命中 0（`lint.md`/`status.md` 都有 frontmatter）；gstack commands 均有 frontmatter | 无 |
| 模糊量词高密度 | `grep -c "ensure\|appropriate\|comprehensive\|robust\|effective" MarkQWu-gstack/**/*.md` | gstack SKILL.md 文件命中 98 处（54 个文件）；平均 1.8 处/文件 | 中 |
| MCP 版本未固定 | `grep -rn "npx -y @" .claude-plugin/plugin.json` | gstack 中未发现此模式；bureau 无 MCP server 配置 | 无 |

**命中后的具体行动建议**：
- gstack 98 处模糊量词：优先处理 `diagram/SKILL.md`、`sync-gbrain/SKILL.md` 中 "ensure" 用法，把"确保质量"改成"检查 X 字段非空且 Y 格式正确"。每个文件 10–15 分钟可完成。

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：`$REPORT_BASE` 统一输出约定
   - **本案例做法**：`bash/SKILL.md` 定义 `$REPORT_BASE` 变量，2 个命令遵守此约定（`simplify.md`、`spike-investigation.md`）
   - **我的项目现状**：gstack 和 bureau 命令各自输出到不同路径，无统一约定
   - **如何借鉴**：在 gstack 的 `diagram/SKILL.md` 或 bash-wrapper skill 里定义 `GSTACK_OUTPUT_DIR=.gstack/outputs`；改 3 个输出型命令引用该变量

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：frontmatter 完整性
- **我的做法**：bureau/commands/ 的命令文件全部有 `description` + `argument-hint` frontmatter（lint.md、status.md 验证）
- **本案例做法（弱在哪）**：26/26 命令缺 frontmatter，NLPM 分数被该单一问题拉到 36/100
- **意义**：bureau 在 frontmatter 规范上已达到及格线，这是竞争性优势；未来若有人对 bureau 做 NLPM audit，不会在这个最基础的维度上失分。

---

## 八、术语表

### <a name="nl工件"></a>NL 工件
> Natural Language artifact 的简称。指 Claude Code 中用自然语言写成、能被 NLPM（NL Programming Manager）扫描和注册的文件，包括 SKILL.md、agent 定义文件、slash command 文件（`commands/*.md`）和 hook 配置。NLPM 通过这些文件里的 YAML frontmatter 来"认识"这个工件是什么、能做什么。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读命令文件时先解析 frontmatter 才能知道这个命令怎么注册和调用。缺少 frontmatter = 命令对外不可见。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。
