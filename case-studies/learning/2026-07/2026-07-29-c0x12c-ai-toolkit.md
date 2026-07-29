# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-07-29（基于当前 HEAD）
**主题标签**：`template-design`, `single-purpose`, `manifest-discipline`, `vague-quantifier`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`@c0x12c/ai-toolkit` 是一个**全栈开发工程纪律插件**，从 v1.22.1 成长到了 v1.27.0（当前版本）。它围绕"Spartan / GSD（Get Stuff Done）"工作流框架，为 Micronaut 后端 + NextJS 前端 + 基础设施的全栈开发场景打包了 **9 个 agent + 69 个命令 + 29 个 skill + 21 条规则**（当前）。作者 Khoa Tran 是越南 c0x12c 公司的工程师。

关键事实：
- 插件规模：plugin.json 描述为"12 packs organized"——这是同类插件中规模最大的之一
- 有完善的 `templates/` 目录（12 个 Markdown 模板，如 feature-spec、prd-template、epic）
- 有 `claude-md/` 目录（组合式 CLAUDE.md 构建器，可按需组合不同技术栈模块）
- 有 `bridges/` 目录（Telegram bridge + 核心 bridge），显示作者在扩展 Claude Code 的通知渠道

### 1.2 架构剖析
```
ai-toolkit/
├── toolkit/
│   ├── .claude-plugin/plugin.json  ← 插件注册表（v1.27.0）
│   ├── agents/           ← 9 个专用 agent（CTO、SRE、Micronaut、设计师等）
│   ├── commands/
│   │   ├── spartan.md    ← 命令路由器（新增）
│   │   └── spartan/      ← ~25+ 个 spartan:* 命令
│   ├── skills/           ← 29 个 SKILL.md（深度研究、基础设施、Terraform 等）
│   ├── templates/        ← 12 个 Markdown 文档模板
│   ├── claude-md/        ← 分模块的 CLAUDE.md 片段（可按技术栈组合）
│   ├── frameworks/       ← lean-canvas、design-sprint 框架参考
│   └── ETHOS.md          ← Spartan GSD 哲学文档
├── bridges/              ← Telegram + 核心通知桥接
└── .claude-plugin/
    └── marketplace.json  ← 根级别 marketplace 条目
```

- **文件类型分布**：9 个 agent / 69 个 command（含 spartan.md 路由器）/ 29 个 SKILL.md / 21 条 rules / 12 个模板
- **编排关系**：`spartan.md` 作为命令路由器，统一入口；各 agent 独立运行；skills 被 commands 按需加载
- **跨件契约**：`claude-md/` 的片段按技术栈分层（00-header、01-core、05-database 等），可以自由组合拼出完整 CLAUDE.md

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「工程纪律优于工具数量」——ETHOS.md 里的 Spartan GSD 框架强调「先做、做完、做好」，而不是用 AI 生成更多文档
- **解决什么问题**：全栈团队（Micronaut + NextJS + Infra）在使用 Claude Code 时缺乏统一的流程约束；这个插件注入了完整的 SDLC 规范
- **Trade-off**：插件规模极大（69 命令），上手门槛高，但覆盖了全栈开发的每个角落；用「深度」换了「广度」
- **认知模型**：把 Claude Code 当「工程师团队」——每个 agent 是一个专业角色（CTO、SRE、设计师），每个 command 是一个标准化工作流

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「大型编排插件 + 路由器模式」**：以 `spartan.md` 为单一入口命令路由器，把 69 个 spartan:* 命令统一在一个命名空间下；agent 层负责角色扮演，skill 层提供领域知识，template 层提供输出模板。

模式特征（5 条）：
- **特征 1**：单一命名空间（`spartan:*`）——所有命令共享同一前缀，用户发现成本低
- **特征 2**：`templates/` 目录独立于 skills——templates 是**输出格式**，skills 是**操作知识**，两者分离
- **特征 3**：`claude-md/` 分片组合——CLAUDE.md 不是单一文件，而是可按技术栈拼装的片段库
- **特征 4**：每个 agent 高度专业化（micronaut-backend-expert、sre-architect 等），避免"万能 agent"
- **特征 5**：`ETHOS.md` 记录工作哲学，不只是技术文档

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要统一全栈团队 AI 用法的公司 | ✅ 高度适用 | 插件覆盖完整 SDLC，有哲学文档 |
| 个人小项目 | ❌ 不适用 | 69 命令对小项目是过度工程 |
| Micronaut + NextJS 技术栈 | ✅ 极度适用 | 专为这个组合设计 |
| 需要快速上手的初学者 | ❌ 不适用 | 体积大，学习曲线陡 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 大型路由器插件（本仓库） | c0x12c/ai-toolkit | 覆盖完整；统一入口 | 允许工具缺失；维护成本高 |
| 精简 skill 集 | trailofbits/skills | 专注领域；高质量 | 覆盖面窄 |
| 模块化组合 | navapbc/digital-service-orchestra | 可扩展；工作流驱动 | 架构复杂 |

### 2.4 改进空间
1. **当前问题**：所有 spartan:* 命令缺少 `allowed-tools` 声明（-5 分/命令，-345 分的聚合影响） **改进做法**：为每个命令在 frontmatter 里加入实际需要的工具声明，至少写最常用的（Read, Grep, Bash） **预期收益**：NL 质量分从 94.8 提升至 99+，且安全性提升（防止 Claude 使用未声明的工具）
2. **当前问题**：`phase-reviewer` agent 在 body 里用 `cat` / `ls` 等 shell 命令，但 `allowed-tools` 未声明 Bash **改进做法**：在 phase-reviewer 的 frontmatter 里加 `tools: ["Read", "Bash", "Glob"]` **预期收益**：消除"引用了未声明工具"的 Bug #1
3. **当前问题**：`spartan:guard` 命令里 `{{ args[0] }}` 没有空值 fallback **改进做法**：改为 `{{ args[0] | default("当前目录") }}` 或用 `$ARGUMENTS` 配合 empty-input guard **预期收益**：消除「无参调用时命令崩溃」的问题

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-25 当时得分 **96/100**。

| 类别 | 当时得分 | 主要问题 |
|---|---|---|
| agents 均值 | 94.3 | research-planner/idea-killer/ai-designer 零示例（-15 各） |
| commands 均值 | 94.8 | 所有命令缺 `allowed-tools`（-5 各）；guard 缺空值处理（额外 -5） |
| skills 均值 | 97.5 | 个别技能有模糊量词 |

### 3.2 当时值得借鉴的模式
1. **`templates/` 与 `skills/` 分离** → 输出格式（templates）和操作知识（skills）各守其职 → 根本原因：分离让两类内容可以独立更新，feature-spec.md 变了不影响 deep-research/SKILL.md → 借鉴：在我的项目里也用 `templates/` 存放输出模板，skill 只讲操作方法
2. **`claude-md/` 片段化 CLAUDE.md** → 按技术栈分层（数据库层、基础设施层、前端层），可自由组合 → 根本原因：单一大文件难以维护；片段化后不同团队可按需拼装 → 借鉴：如果 bureau 的 CLAUDE.md 越来越长，考虑拆分成片段
3. **角色化 agent 命名**（micronaut-backend-expert、sre-architect、solution-architect-cto）→ 名字本身就是职责说明，无需读 description 就知道该用哪个 → 根本原因：好的命名是最便宜的文档 → 借鉴：未来的 agent 用「角色-领域」格式命名
4. **ETHOS.md 工作哲学** → 告诉 Claude（和用户）这个工具的设计价值观，避免"功能用对了但哲学用错了" → 借鉴：重要的插件可以写一份 ETHOS.md 或 PHILOSOPHY.md

### 3.3 当时的缺陷
1. **25 个命令全部缺 `allowed-tools`** → 根本原因：作者可能认为 Claude 会自己判断要用哪些工具，忽视了显式声明对安全边界的作用 → 自查：MarkQWu/gstack 的命令是否也缺 allowed-tools？（未发现，但需验证）
2. **`phase-reviewer` 引用 `cat`/`ls` 但未声明 Bash** → 根本原因：agent 在写 body 时自然写了 shell 命令，没有回头检查 frontmatter 是否对齐 → 这是一个「body 与 frontmatter 不同步」的经典问题 → 自查：写了 Bash 相关命令的 skill/agent，是否有对应 `allowed-tools: Bash` 声明？
3. **3 个 agent 零示例**（research-planner、idea-killer、ai-designer）→ 根本原因：这三个 agent 做的是「思维型任务」（规划、批判），作者可能认为无法举例；但示例可以是"给定问题 X → agent 输出结构 Y" → 自查：我的 SKILL.md 里有没有类似的「觉得示例说不清就不写了」的漏洞？

### 3.4 当时的优化机会
1. 批量为 25 个 spartan:* 命令添加 `allowed-tools: ["Read", "Bash", "Grep"]`
2. `phase-reviewer` frontmatter 加 `tools: ["Read", "Bash", "Glob"]`
3. `spartan:guard` 加空值 fallback

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 25 命令缺 `allowed-tools` | `find toolkit/commands/ -name "*.md" -exec grep -L "allowed-tools" {} \;` | **持续扩散**（新增的 spartan.md、fundraise.md、ux.md、gate-review.md、figma-to-code.md 也缺） | 从 v1.22.1 到 v1.27.0 版本迭代但未修复 |
| `phase-reviewer` 用 `cat`/`ls` 未声明 Bash | `grep "cat\|ls " toolkit/agents/phase-reviewer.md` | **持续**（body 仍在第 88-97 行用 cat/ls） | Bug #1 未修 |
| `spartan:guard` 无空值处理 | `grep "args\[0\]" toolkit/commands/spartan/guard.md` | **持续**（第 15-16 行仍用 `{{ args[0] }}`） | 空调用仍崩溃 |

**注意**：`phase-reviewer` 已加入 2 个 Example 块（`<example>` tag），这是改进——examples 缺失问题已修复。`research-planner` 也加了 `tools: ["Read", "Grep", "Glob", "WebSearch"]` 声明。

### 4.2 架构演进
- 版本从 v1.22.1 → v1.27.0（5 个版本迭代，说明活跃维护）
- 新增命令：`spartan.md`（路由器）、`fundraise.md`、`ux.md`、`gate-review.md`、`figma-to-code.md`
- plugin.json 描述升级为"69 commands, 21 rules, 29 skills, 9 agents in 12 packs"（更完整的描述）
- `bridges/telegram/` 和 `bridges/core/` 新增，说明作者开始扩展 Claude Code 的边界——这是值得关注的新方向

### 4.3 新增的可学习模式
- **`bridges/` 目录**：通知桥接层（Telegram bridge）——让 Claude Code 完成任务后可以发 Telegram 消息。这是「Claude Code + 外部通知渠道」的集成模式，在其他仓库少见
- **`spartan.md` 路由器命令**：作为整个 spartan 命名空间的目录页，让用户不用记所有子命令，只需 `/spartan` 就能看到完整列表

---

## 五、校准

### 5.1 我已经在做对的
1. MarkQWu/gstack 的 SKILL.md 有完整 frontmatter，包含 `model:` 和 `description:`——与本仓库 95+ 分的 skill 一致
2. 我没有「body 里用工具但 frontmatter 未声明」的问题（目前无 agent）
3. 我的项目规模小，没有走「69 命令」的大型插件路线——这是合适的：不同规模的项目应该有不同规模的 NL 层

### 5.2 挑战 / 验证
- **验证**：`templates/` 与 `skills/` 分离的模式在本仓库得到了清晰验证——feature-spec.md（模板）和 deep-research/SKILL.md（操作知识）确实是两个完全不同的维度，分开管理更合理。
- **挑战**：我原以为「大而全」的插件是好事。这个仓库提醒我：69 个命令但每个都缺 `allowed-tools`，整体安全性反而不如 10 个命令但声明精准的小插件。规模不能替代精准度。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令是否有 allowed-tools 声明
find /tmp/my-repos/MarkQWu-*/ -name "*.md" -path "*commands*" | xargs grep -L "allowed-tools" 2>/dev/null

# 检查我的 skill/agent body 是否引用了 Bash 命令但未声明
grep -rn "^.*\`\`\`bash\|^- Run \`\|Bash(" /tmp/my-repos/MarkQWu-*/ --include="SKILL.md" 2>/dev/null | head -10

# 检查 gstack 的 skill frontmatter 完整性
for f in /tmp/my-repos/MarkQWu-gstack/**/SKILL.md; do
  echo "=== $f ==="
  grep -E "^name:|^description:|^model:|^allowed-tools:" "$f" 2>/dev/null | head -5
done
```

命中后怎么办：
- 命令缺 `allowed-tools` → 审查命令 body 实际用了哪些工具，在 frontmatter 里精确声明（不要写 `*` 或省略）
- Body 引用 Bash 但未声明 → 在 frontmatter `tools:` 里加 `"Bash"`

### 6.2 灵感 → 实施路径
1. **想法**：为 MarkQWu/bureau 的命令写一个「命令检查清单」——确保每个命令都有 `name`、`allowed-tools`、非空 `$ARGUMENTS` 处理
   - **为何可行**：本仓库验证了「缺 allowed-tools」在 25 个命令上的规模化损失；清单可以在写新命令时预防
   - **第一步**：在 bureau 的 CLAUDE.md 里加一节"新增命令前的 5 项检查"（10 分钟）

2. **想法**：参考 `claude-md/` 分片模式，把 MarkQWu/- 的 CLAUDE.md 拆成「core.md + geopolitical.md + api-integration.md」
   - **为何可行**：MarkQWu/- 的 CLAUDE.md 目前不存在，但随着项目复杂度增加，分片模式可以避免 CLAUDE.md 变成一个大杂烩
   - **第一步**：先写 core.md（项目目的 + 基本约束），API 细节放 api-integration.md（30 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例的核心目的**：为全栈开发团队（Micronaut + NextJS）提供统一的 Claude Code 工程纪律插件
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有 commands + skills 组合 | bureau 专注知识管理，不是工程纪律 | 中（命令结构和 allowed-tools 模式有参照价值） |
| MarkQWu/gstack | 高 | 都是多 skill 的 Claude Code 工具集；都追求工程流程规范 | gstack 是 Garry Tan 的工具复现；c0x12c 是自创工程框架 | 高（skill 质量标准直接参考） |
| MarkQWu/graphify | 低 | 都是 Claude Code 插件 | graphify 是知识图谱查询，无工程流程 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `allowed-tools` | `grep -rL "allowed-tools" MarkQWu-*/.claude/commands/` | **未发现**（我的命令较少，但需确认） | 待验证 |
| Body 引用 shell 命令但 frontmatter 未声明 Bash | `grep -rn "cat \|ls \|grep " MarkQWu-gstack/*/SKILL.md` | **未检查**（gstack SKILL.md 数量多，需要专项验证） | 中 |
| template 与 skill 未分离 | 检查 gstack 目录结构 | **命中**（gstack 没有专门的 templates/ 目录） | 低（个人项目规模小） |

**命中后的具体行动建议**：
- MarkQWu/gstack → 检查每个 SKILL.md 的 body 里有无 shell 命令；若有，在 frontmatter 加 `model: sonnet`（如缺）和 `tools: ["Bash"]`（约 20 分钟）

### 8.3 别人的更优方案

1. **领域**：`templates/` 与 `skills/` 分离
   - **本案例做法**：`toolkit/templates/` 存放输出模板（feature-spec.md、prd-template.md 等）；`toolkit/skills/` 存放操作知识；两者在 plugin.json 里分别注册
   - **我的项目现状**：MarkQWu/gstack 把模板内容混在 SKILL.md body 里，没有独立的模板层
   - **如何借鉴**：在 gstack 里创建 `templates/` 目录，把 SKILL.md 里"最终输出应该长什么样"的部分提取出来作为模板文件（1 小时）

2. **领域**：`ETHOS.md` 工作哲学文档
   - **本案例做法**：Spartan GSD 框架的核心原则写在 ETHOS.md，让使用者理解「工具背后的思考方式」
   - **我的项目现状**：bureau 的 BUREAU.md 有部分设计哲学，但不够系统
   - **如何借鉴**：为 bureau 写一个 PHILOSOPHY.md，解释「为什么是这个 capture-compile-review-query 流程，而不是其他方式」（30 分钟）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：命令规模克制
- **我的做法**：bureau 的命令数量 < 10，每个命令职责单一
- **本案例弱点**：69 个命令但全部缺 `allowed-tools`，体量大但规范度低
- **意义**：我的小而精的命令设计是正确的。规模放大之前，先确保每个命令的 allowed-tools 声明精准——这是本案例最重要的反向教训

---

## 八、术语表

### <a name="gsd"></a>GSD（Get Stuff Done）
> 「把事情做完」的工作哲学。强调执行优先于完美规划——先把最重要的事情做出来，而不是一直在讨论怎么做最好。Spartan 框架的核心价值观。

### <a name="spartan"></a>Spartan 命令前缀
> `spartan:` 是这个插件所有命令的命名空间前缀，就像一个品牌标记。用 `/spartan:debug` 而不是 `/debug`，是为了在多插件环境里避免名字冲突。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code [frontmatter](#frontmatter) 里的一个字段，声明这个命令/skill/agent 被允许调用哪些工具（如 `Read`、`Bash`、`Grep` 等）。没有声明的工具，Claude Code 在执行时会询问用户权限或被拒绝。精准声明是安全边界的关键。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（`name`、`description`、`model`、`allowed-tools` 等）。Claude Code 先读这里才知道这个 skill/agent/command 是谁、能干什么。
