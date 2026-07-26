# ananddtyagi/cc-marketplace — 学习案例

**仓库**：https://github.com/ananddtyagi/cc-marketplace
**Stars**：N/A（注册表未记录）| **来源**：upstream audit（contributed 状态，5 个 PR 仍在开放中）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-26（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `monorepo-vs-split`, `examples-driven`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
一个「Claude Code 插件市场」——119 个独立插件聚合在同一个 [monorepo](#monorepo) 下，每个插件都有自己的 `plugin.json`、命令、代理和技能。不同贡献者提交自己写的插件，维护者 Anand Tyagi 做汇聚和管理。审计时（2026-04-06）共 272 个 NL 工件，当前 HEAD 共 177 个 .md 文件 + 119 个插件目录。

关键事实：
- 多贡献者集合体（`plugins/experienced-engineer` 有 10 个代理均由同一人贡献，`plugins/sugar` 是另一套独立系统）
- 提供 `PLUGIN_SCHEMA.md` 作为贡献者规范——但质量差异显示规范并未被严格执行
- 有高质量的「样板」插件（`api-contract-sync-manager`、`math`、`sugar`），也有大量质量极低的插件（无 frontmatter、无示例、占位符描述）
- NLPM 已提交 5 个 bug-fix PR（#44-#48），截至当前 HEAD 全部仍处于 open 状态，未被合并

### 1.2 架构剖析

**目录结构：**
```
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json              ← 插件 manifest（必须）
    commands/*.md              ← 命令文件（可选）
    agents/*.md                ← 代理文件（可选）
    skills/<name>/SKILL.md     ← 技能文件（可选）
    hooks/hooks.json           ← 钩子配置（可选）
    scripts/                   ← 脚本工具（可选）
PLUGIN_SCHEMA.md               ← 贡献规范
scripts/                       ← 仓库级脚本
```

**文件类型分布（当前 HEAD）：** 91 代理文件 + 45 命令文件 + 14 SKILL.md + 119 个 plugin.json（总 177 .md 文件）

**编排关系：** 每个插件是**完全独立的单元**——没有跨插件引用，没有公共技能池，没有路由层。「市场」是物理上的文件聚合，而非架构层面的组合系统。

**跨件契约：** 各插件内部契约自成体系，质量参差不齐。有些插件（sugar、claude-dev-infrastructure）有完整的内部 hook 系统，而大多数插件是「单文件」简单命令或代理。

### 1.3 设计思路 / 方法论

**核心设计哲学：** 「低门槛聚合」——降低贡献门槛，快速扩大数量，让市场本身对优质插件进行自然筛选（用户安装投票）。

**解决什么问题：** Claude Code 插件生态初期碎片化，用户难以发现好用的插件。集中化市场降低了发现成本。

**Trade-off：**
- **广度 vs. 深度**：选择聚合数量（119 个插件）而非保证质量（5 个高质量插件）
- **开放贡献 vs. 质量把关**：贡献门槛极低（有 PLUGIN_SCHEMA.md 但无强制检查），导致大量低质量工件进入
- **单仓 vs. 独立仓**：monorepo 方便集中管理 README/更新记录，但插件版本无法独立迭代

**认知模型：** 维护者把这个仓库视为「目录」而非「标准库」——用户自行评估质量，维护者只做汇聚。这解释了为什么 5 个 bug-fix PR 长期未被合并：维护者对质量不负连带责任。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「低门槛贡献市场 + 质量二八分层」**

关键特征：
- 贡献者单独维护自己的插件子目录，主仓库只做目录化
- 无统一的 CI/lint 检查强制执行质量标准
- 质量呈现严重的二八分布：约 20% 插件（sugar、api-contract-sync、math）展示了最佳实践；约 80% 插件有系统性 NLPM 缺陷
- 「示范插件」（best-in-class）在仓库中被 audit 明确指出，但没有把这些示范提升为规范的机制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 快速构建插件生态系统 | ✅ 初期适用 | 低门槛带来快速增长 |
| 企业内部工具标准化 | ❌ 不适用 | 无质量把关，标准无法保证 |
| 技术社区展示/发现 | ✅ 适用 | 相当于「AI 工具 awesome 列表」 |
| 作为学习参考 | ⚠️ 慎用 | 大多数插件质量不足以作为模板 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 低门槛市场 | 本仓库 | 增长快、生态广 | 质量参差，无法作为规范 |
| 策划性精选库 | hesreallyhim/awesome-claude-code | 有筛选，质量有保证 | 增长慢，维护成本高 |
| 单作者高质量库 | deanpeters/Product-Manager-Skills | 质量一致，有认知深度 | 无法快速扩展 |

### 2.4 改进空间

1. **当前问题**：无 CI 检查，贡献者提交的 frontmatter 缺失、malformed 等问题无法被自动拦截。**改进做法**：引入 `bin/nlpm-check` 作为 pre-merge CI check（GitHub Actions），设置最低分数阈值 60 分（当前许多插件低于此分）。**预期收益**：新 PR 提交时自动检查，杜绝 bug 级别的问题进入主分支。

2. **当前问题**：`PLUGIN_SCHEMA.md` 是文本规范，但实际上有大量违反规范的文件已存在。**改进做法**：把规范里的要求转化为 `bin/nlpm-check` 的 config 文件（`.claude/nlpm.config.md`），把「文字约定」变成「可执行的约束」。

3. **当前问题**：best-in-class 插件（`api-contract-sync-manager`、`sugar`）没有被显著标识为「参考模板」。**改进做法**：在 README 中增加「模板插件」章节，把这几个插件高亮为新贡献者的模板。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **73/100**，272 个工件，9 个 bug，200 个质量问题，13 个安全发现。

| 问题类型 | 受影响数量 | 最低分典型 |
|---|---|---|
| 完全无 YAML frontmatter 的代理文件 | 5 个 | 35 分（growth-hacker 等） |
| 命令缺 `name` 字段 | ~38 个 | 60 分 |
| 命令缺 `allowed-tools` | ~43 个 | 60-75 分 |
| 代理缺 `model` 声明 | ~70 个 | 65 分 |
| 代理无 `<example>` 块 | ~25 个 | 65 分 |
| 模糊量词 | ~60 个代理 | -4 到 -6 分/每文件 |
| Malformed frontmatter（全在一行） | 1 个 | 72 分 |
| 占位符描述（plugin.json） | 1 个 | -25 分 |

### 3.2 当时值得借鉴的模式

1. **`api-contract-sync-manager` 技能** → `plugins/api-contract-sync-manager/skills/api-contract-sync/SKILL.md`：有 `allowed-tools: Read, Grep, Glob, RunTerminalCmd`、清晰的 description、结构化的工作流——是整个库里的最高分工件。如何借鉴：这个文件就是「命令技能 SKILL.md 应该怎么写」的具体范例。

2. **`sugar` 插件的 hook 系统** → `plugins/sugar/hooks/hooks.json`：11 个 hook，有节流保护（throttle guards）。如何借鉴：hook 不是越多越好，需要避免级联触发——sugar 的做法是加 throttle 条件，防止 hook A 触发 B 触发 C 的链式 blast。

3. **`math` 插件的代码示例** → `plugins/math/skills/SKILL.md`：用实际 Python 代码（SymPy）展示了工具行为，而非文字描述。如何借鉴：在技能里描述数学/计算类任务时，直接放可运行的代码示例比文字描述更可靠。

4. **`PLUGIN_SCHEMA.md` 的存在意义** → 即使规范未被严格执行，它的存在让新贡献者有一个「期望格式」的参考。如何借鉴：为多贡献者项目提前写贡献规范，即使无法强制执行，也能降低质量下限。

### 3.3 当时的缺陷

1. **5 个代理完全无 YAML frontmatter**（growth-hacker、reddit-community-builder、twitter-engager、content-creator、changelog-generator）→ 根本原因：这些文件直接以 Markdown 正文开头，Claude Code 无法解析为代理——实质是「装修好的普通文本文件，未被 AI 运行时识别」。这是最严重的 bug 类型，因为无论内容多好，运行时都忽略它。自查：我有没有写了很好的指令但忘记加 frontmatter 的文件？

2. **`desktop-app-dev` frontmatter 全在一行（malformed YAML）** → 根本原因：YAML 要求每个字段占一行，压缩成一行会导致解析失败。这种 bug 很隐蔽——文件有 frontmatter 但无法被正确解析，维护者可能以为文件正常。自查：我的 frontmatter 有没有被意外压缩到一行的情况？

3. **`b2b-project-shipper` name 不匹配（`name: project-shipper` vs 目录 `b2b-project-shipper`）** → 根本原因：agent `name` 字段用于 dispatch 引用，如果名字不匹配，任何 `subagent_type: "b2b-project-shipper"` 调用都会失败。自查：我的 agent `name` 字段与插件目录名、调用方使用的名字是否三方一致？

### 3.4 当时的优化机会

1. 引入 CI/CD 自动检查：在 GitHub Actions 里跑 NLPM 检查，阻止低分工件合并
2. 把 best-in-class 插件（sugar、api-contract-sync）提升为「模板插件」，并在 `PLUGIN_SCHEMA.md` 中明确指向
3. 要求新插件提交时：frontmatter 字段 `name`、`description`、`model`（代理）、`allowed-tools`（命令）为必填项

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| growth-hacker 无 frontmatter | `head -3 plugins/growth-hacker/agents/growth-hacker.md` | ❌ **仍存在**：仍以 `# Growth Hacker` 开头，无 YAML | NLPM PR #47（加 frontmatter）未被合并 |
| changelog-generator 无 frontmatter | `head -3 plugins/changelog-generator/agents/changelog-generator.md` | ❌ **仍存在**：仍以 "You are an expert..." 开头 | NLPM PR #48 未被合并 |
| b2b-project-shipper name mismatch | `grep "^name:" plugins/b2b-project-shipper/agents/b2b-project-shipper.md` | ❌ **仍存在**：`name: project-shipper` | NLPM PR #44 未被合并 |
| desktop-app-dev malformed frontmatter | `head -5 plugins/desktop-app-dev/agents/desktop-app-dev.md` | ❌ **仍存在**：所有 frontmatter 字段仍在一行 | NLPM PR #46 未被合并 |
| accessibility-expert 占位符描述 | `cat plugins/accessibility-expert/.claude-plugin/plugin.json` | ❌ **仍存在**：`"description": "Examples:"` | NLPM PR #45 未被合并 |

### 4.2 架构演进

Audit 时：272 工件，91 个代理 + 45 个命令 + 14 个技能（当前 HEAD 数据）对比难以精确，但插件总数从 ~80 增长到了 119（当前），说明新插件在持续加入，但旧的 bug 未修复。这是「增量不优化」的典型增长模式：新内容不断进入，问题存量不断积累。

### 4.3 新增的可学习模式

5 个 NLPM PR 全部仍 OPEN，未合并。这是整个 audit 案例里**最重要的学习信号**，不是技术的，而是生态的：

- **维护者沉默 ≠ 拒绝**：PR #44-#48 已经 4 个月了，没有被关闭，没有评论，只是「搁置」。在开源社区，这是常见模式——维护者对外部 PR 的优先级很低，即使修改正确也如此。
- **「正确修复」不等于「被接受的修复」**：NLPM 的 audit 流程把 bug 级别的问题视为 PR 值得提交的依据，但实际上维护者没有回应。这说明 PR 成功率不只取决于修复质量，还取决于「维护者的活跃度」和「他是否有意主动管理外来贡献」。
- **反向信号**：对于类似的「目录型」仓库（低活跃维护者，以聚合为主），即使 bug 很明显，外来 PR 的转化率也可能很低。这影响 audit 流程的 PR 策略。

---

## 五、校准

### 5.1 我已经在做对的

1. `MarkQWu/bureau` 和 `MarkQWu/gstack` 的技能/命令文件全部有正确的多行 YAML frontmatter，不存在 malformed 或缺失的情况
2. `MarkQWu/gstack` 的代理文件有 `model:` 声明（在 gstack 的 pair-agent/SKILL.md 里明确了模型策略）
3. 我的仓库都是单一维护者，不存在「多贡献者质量失控」的问题
4. bureau 的 `name` 字段与插件实际引用名称一致——无命名漂移

### 5.2 挑战 / 验证

**挑战**：这个案例挑战了我「bug-fix PR 一定有价值」的假设。NLPM 提交了 5 个 bug-fix PR，4 个月后全部悬空。如果我的目标是「通过 PR 改善生态」，那么「目录型」仓库（以汇聚为主、维护者不活跃）可能根本不是合适的目标——即使 bug 很明显，PR 也可能永远搁置。

更有效的路径可能是：与维护者先建立联系（issue 讨论），或者 fork 出高质量版本，或者专注于真正活跃的仓库。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 NL 工件 frontmatter 是否正确（非 malformed）
for f in $(find /tmp/my-repos/ -name "*.md" -path "*/commands/*" -o -name "*.md" -path "*/agents/*" -o -name "SKILL.md"); do
  # 检查文件是否以 --- 开头（有 frontmatter）
  head -1 "$f" | grep -q "^---$" || echo "MISSING FRONTMATTER: $f"
done 2>/dev/null | head -20
```
命中后：为缺少 frontmatter 的文件加上 `---\n...\n---` 块。

```bash
# 检查 agent name 字段是否与目录名一致
find /tmp/my-repos/ -name "*.md" -path "*/agents/*" 2>/dev/null | while read f; do
  dir=$(basename $(dirname "$f"))
  name=$(grep "^name:" "$f" 2>/dev/null | head -1 | sed 's/name: //')
  [ -n "$name" ] && [ "$name" != "$dir" ] && echo "NAME MISMATCH: $f (name=$name, dir=$dir)"
done | head -10
```
命中后：使 `name:` 字段与目录名和所有调用方引用的名字完全一致。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 引入 `CONTRIBUTING.md` + 基本 CI 检查（即使是单维护者仓库）
   - **为何可行**：bureau 现在有足够的 NL 工件，未来若开放贡献或接受 PR，有基础规范可以省去手动审查
   - **第一步**：在 `bureau/` 根目录写 10 行 `CONTRIBUTING.md`，列出「每个命令必须有 description、argument-hint；每个技能必须有 name、description、type」等要求

2. **想法**：参考 PLUGIN_SCHEMA.md 的思路，为 gstack 写一个 `SKILL_SCHEMA.md`，定义 SKILL.md 的最小结构
   - **为何可行**：gstack 有 `bun run skill:check` 命令，但检查规则是内化在代码里的，明文文档更易沟通
   - **第一步**：查看 `gstack/bin/` 里的 skill 检查逻辑，提取规则，写成 Markdown 文档 `SKILL_SCHEMA.md`

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（MarkQWu/gstack, MarkQWu/bureau, MarkQWu/graphify）

### 7.1 目的对齐度

- **本案例核心目的**：聚合多贡献者 Claude Code 插件，构建可发现的插件市场

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 低 | 同样有命令/技能 NL 工件 | bureau 是功能工具，不是市场 | 低（架构目的差异大） |
| MarkQWu/gstack | 低 | 同样有多个 SKILL.md | gstack 是单一项目内工具集，不是聚合市场 | 低 |
| MarkQWu/graphify | 无 | 无 | 代码分析工具与插件市场无交叉 | 无 |

若全部「无」：我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 代理/命令无 YAML frontmatter | `head -1 */agents/*.md` | **bureau**：commands 文件均以 `---` 开头（正常），未命中 | 无 |
| Frontmatter malformed（单行） | `grep -c "^---" *.md` < 2 | 未命中（所有 frontmatter 正确格式化） | 无 |
| `name` 字段与调用名不匹配 | `grep "^name:" agents/*.md` vs 目录名 | 未命中（bureau 无 agents 目录，gstack 的 pair-agent name 与 SKILL.md 文件名一致） | 无 |
| 占位符描述（plugin.json / SKILL.md） | `grep -rn "Examples:\|TODO\|placeholder" *.json` | **gstack 潜在**：SKILL.md.tmpl 中有模板占位符 `[VERB + TARGET]`（合理，是模板文件本身）| 无（合理）|

**总体评估**：本案例的典型缺陷（无 frontmatter、malformed YAML、name 不匹配）在我的仓库中均未复现。我的仓库规模小、单维护者，这类「贡献者纪律」问题不会出现。

### 7.3 别人的更优方案

1. **领域**：`api-contract-sync-manager` 技能的 `allowed-tools` 精细声明
   - **本案例做法**：`allowed-tools: Read, Grep, Glob, RunTerminalCmd`——精确到工具级别，而非粗粒度的 `Bash`
   - **我的项目现状**：`MarkQWu/bureau` 的命令文件无 `allowed-tools`（前述缺陷）；`MarkQWu/gstack` 的部分 SKILL.md 有 `allowed-tools`，但多数写的是较宽泛的 `Bash`
   - **如何借鉴**：把 gstack 中 `allowed-tools: Bash` 替换为实际需要的工具组合——如果只需要读文件，写 `allowed-tools: Read, Glob` 而非允许整个 `Bash`

2. **领域**：`sugar` 插件的 hook 节流机制
   - **本案例做法**：`hooks/hooks.json` 中有 throttle 条件，防止 hook 触发级联
   - **我的项目现状**：`MarkQWu/bureau` 有 `hooks/` 目录（推测），但无节流机制文档
   - **如何借鉴**：在 bureau 的 hook 配置里加 `"stopSequence": true` 或时间窗口条件，避免一次操作触发多个 hook 级联

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：自动化质量检查机制
- **我的做法**：`MarkQWu/gstack` 有 `bun run skill:check` + `bun test` 提供技能健康仪表盘，并通过 `bun run slop:diff` 检测 slop 词汇
- **本案例做法**：零自动化检查，质量完全依赖贡献者自律 + 维护者人工审查（但维护者明显无精力审查大量 PR）
- **意义**：如果 cc-marketplace 引入了哪怕是最简单的 CI 检查（`bin/nlpm-check` 最低分 > 60），5 个被 NLPM 发现的 bug 都不会进入主分支。自动化门槛是质量的最后保障，不是可选项。

---

## 八、术语表

### <a name="monorepo"></a>monorepo
> 把多个独立项目（这里是多个 Claude Code 插件）放在同一个 Git 仓库中管理的做法。好处是集中管理、统一可见；坏处是项目之间互相影响、单个项目无法独立发版、规模大后 CI 和权限管理变复杂。对比：每个插件独立仓库（micro-repo）。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块。Claude Code 加载代理/命令/技能文件时，首先解析 frontmatter 获取 `name`、`description`、`model`、`tools` 等关键元数据。如果文件没有 frontmatter，Claude Code 会忽略这个文件（它对 AI 运行时是不可见的）。

### <a name="malformed-YAML"></a>malformed YAML（畸形 YAML）
> frontmatter 存在但格式错误，最常见的是把所有字段压缩在一行（如 `name: foo description: bar`）。YAML 解析器无法正确解析单行格式的多字段，导致 frontmatter 部分或全部失效。检测方法：用 `head -5` 查看文件开头，YAML frontmatter 应该每字段一行。

### <a name="hook节流"></a>hook 节流（throttle guard）
> 在 Claude Code hooks.json 中设置的防护机制，限制 hook 在短时间内的触发频率（如「同一个 hook 60 秒内只触发一次」）。防止 hook A 的触发导致 B 触发，B 的触发又导致 A 触发，形成无限级联调用。`sugar` 插件在 11 个 hook 的系统中使用了此机制。
