# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream
**Audit 日期**：2026-04-25（历史快照，v1.22.1）| **生成日期**：2026-08-02（基于当前 HEAD，v1.27.0）
**主题标签**：`manifest-discipline`, `template-design`, `single-purpose`, `model-pinning`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

c0x12c/ai-toolkit 是一个生产级 Claude Code 插件，实现了「Spartan / GSD（Get Stuff Done）」工程框架。它将 10 个 agent、61+ 个命令、35 个 skill 打包成一个插件，覆盖全栈开发、Terraform 基础设施、UI 设计、市场研究和创业运营等场景。运营方是软件公司 c0x12c，而非个人作者。

关键事实：
1. **公司级出品**：作者不是个人黑客，而是一家软件公司。NL 工件质量的高度一致性（96/100）与此直接相关
2. **双入口架构**：`toolkit/` 是人工维护的源文件，`.codex/` 是通过 `compile-packs.js` 自动生成的发行版镜像，两者在 `prepublishOnly` 钩子时同步
3. **用户获取方式**：`claude plugin install @c0x12c/ai-toolkit` 或通过 `npx get-shit-done-cc@latest` setup 脚本
4. **生态位置**：与其他「工作流框架」插件（如 MarkQWu/gstack）同属「把 Claude Code 变成团队助手」这一赛道，是目前综合覆盖面最广的公开插件之一
5. **版本跨度**：本案例横跨 v1.22.1（Audit 快照）到 v1.27.0（当前 HEAD），共经历约 5 个版本迭代

### 1.2 架构剖析

- **目录结构**：

```
c0x12c/ai-toolkit/
  toolkit/                    # 人工维护的源文件（唯一事实来源）
    agents/                   # 9 个 agent（v1.27.0；v1.22.1 时 10 个）
    commands/spartan/         # 61 个 command（v1.22.1 时 25 个）
    skills/                   # 35 个 skill（v1.22.1 时 34 个）
    rules/                    # 设计规范文档（非 skill，不可被 Claude Code 加载）
    hooks/                    # JS hook 脚本（spartan-check-update.js 等）
    scripts/                  # Shell/Python 工具脚本
  .codex/                     # compile-packs.js 自动生成的 Codex CLI 发行版
    agents/                   # 8 个（v1.22.1 时 5 个）
    commands/spartan/         # 与 toolkit/ 同步
    skills/                   # 26 个（v1.22.1 至今未变，toolkit/ 已有 35 个）
  bridges/                    # Telegram、Web 等集成桥接层（有独立 package.json）
  compile-packs.js            # 源 → 发行版编译脚本
  plugin.json / package.json  # 元数据与发布配置
```

- **文件类型分布（v1.27.0 实际数）**：
  - Skill：35 个（toolkit/）；26 个（.codex/）— 存在 9 个未同步
  - Agent：9 个（toolkit/）；8 个（.codex/）— 存在 1 个未同步
  - Command：61 个（toolkit/commands/spartan/）
  - Hook：1 个 hooks.json + 若干 JS/Shell 脚本

- **编排关系**：Command 触发 Agent（如 `spartan:gsd` 调度 `sre-architect`）；Agent 通过 `skills:` 声明自动加载 Skill；Skill 提供领域知识参考。三层分工：命令→代理→技能。没有单一 Router；每个命令独立路由到对应 Agent 或直接执行。

- **跨件契约**：`compile-packs.js` 是唯一的跨件契约守护者——在 npm `prepublishOnly` 时把 `toolkit/` 的内容复制到 `.codex/`。人工写 `toolkit/`，发布时自动生成 `.codex/`。**但**：这个同步是手动触发（仅在发布时），不是每次提交都运行，所以存在镜像滞后窗口。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「[Spartan 框架](#spartan-框架)」= 去掉废话，只做能推进项目的事。每个命令都是一个具体的工程动作（debug、pr、implement、review），而非抽象的「帮我做 X」
- **解决什么问题**：公司内部有一套成熟的工程约定（Git commit 格式、Terraform 模块规范、设计评审流程），但每次换新 Claude 会话就要重新交代。ai-toolkit 把这些约定打包成可安装的 skill，做到「装好即用，不用重复解释」
- **做了什么 trade-off**：选择了「单仓双入口」而非「拆分两个仓库」——维护成本集中在一处，但同步靠脚本，出错时两个入口会发散（已被镜像缺口证实）。选择了 61 个命令的细粒度覆盖，而非几个泛用命令——上手更直观，但命令数量多意味着 [frontmatter](#frontmatter) 维护量翻倍
- **反映什么认知模型**：作者把 Claude Code 插件视为「公司工程规范的可执行化载体」——不只是 prompting，而是把工程纪律编码进工具。这种视角比「帮我写代码」高一个抽象层次

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单仓双入口编译分发」（Single-Source Dual-Entry Compiled Distribution）**

一份源文件，通过编译步骤生成两个入口：一个面向开发者（source），一个面向用户（distribution）。这和前端工程里 `src/` → `dist/` 的思路完全一致。

模式特征清单（5 条）：
- 特征 1：`toolkit/` 是唯一的人工编辑入口，`.codex/` 禁止直接修改
- 特征 2：`compile-packs.js` 是幂等的转换器——多次运行结果相同，失败时不留半成品
- 特征 3：同步时机绑定在 npm 发布流程（`prepublishOnly`），不依赖 CI/CD 也能保证发布时两者一致
- 特征 4：两个入口面向不同的 CLI（Claude Code vs Codex CLI），共享同一套 NL 工件内容
- 特征 5：镜像缺口（toolkit 有但 .codex 没有的文件）是「作者还没决定是否暴露给 Codex 用户」的隐式信号，而非 bug

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 同一套 NL 工件需要服务多个 CLI 工具 | ✅ 高度适用 | 一次维护，多处发布；compile 脚本保证一致性 |
| 内容需要在发布前做预处理（变量替换、验证） | ✅ 适用 | compile 步骤天然是预处理插入点 |
| 快速迭代期，文件结构频繁变动 | ⚠️ 谨慎 | 每次加新文件都要更新 compile 脚本，容易漏同步 |
| 只有一个 CLI 入口的个人项目 | ❌ 不适用 | 维护两套目录增加复杂性，收益为零 |
| 由多人提交 PR 的开源项目 | ⚠️ 谨慎 | 贡献者容易直接改 .codex/ 而不改 toolkit/，需要 CI 守护 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：单仓双入口编译分发 | c0x12c/ai-toolkit | 一套源码，两个发行渠道；一致性靠脚本保证 | 镜像同步窗口，leaky abstraction（8 个 skill 只有 1 个入口能看到） |
| 单仓单入口平铺 | MarkQWu/gstack | 简单直接，没有同步负担 | 无法同时服务多个 CLI；没有发布前预处理能力 |
| 多仓拆分 | 独立的 `skills-repo` + `commands-repo` | 每个仓库职责单一，可独立版本化 | 跨仓引用复杂，发布协调成本高；用户装多个插件 |

### 2.4 改进空间

1. **当前问题**：镜像同步只在 `prepublishOnly` 触发，提交阶段没有守护。**改进做法**：在 CI（GitHub Actions）中运行 `node compile-packs.js && git diff --exit-code .codex/` 作为 PR 门控。**预期收益**：杜绝「本地发布时忘记 compile」导致的版本漂移，8 个镜像缺口问题不会再复现。

2. **当前问题**：61 个命令全部缺少 `allowed-tools` [frontmatter](#frontmatter) 字段。**改进做法**：在 `compile-packs.js` 的编译步骤中加入 frontmatter 验证——如果 toolkit/ 里的 command 没有 `allowed-tools`，编译失败并打印缺失文件列表。**预期收益**：把质量门控内嵌进发布流程，而不依赖人工记忆。

3. **当前问题**：`toolkit/rules/` 下的规范文档（DESIGN_PROCESS.md、ARCHITECTURE.md 等）是高质量领域知识，但没有包装成 [skill](#skill)，Claude Code 无法通过 `/use-skill` 加载。**改进做法**：给每个 rule 文档加一层 SKILL.md 包装（frontmatter + `## Reference` 引用原文内容）。**预期收益**：这些文档从「人读的参考资料」升级为「Claude 可检索的工具知识」，大幅提升 agent 引用规范的精准度。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-25 当时得分 **96/100**（v1.22.1）。

| 类别 | 文件数 | 平均分 | 权重 | 主要问题 |
|---|---|---|---|---|
| Agents | 10 | 94.3 | 14.5% | 3 个零示例（-15 各）；BUG-1 缺 Bash 工具 |
| Commands | 25 | 94.8 | 36.2% | 全部缺 `allowed-tools`（-5 各，系统性） |
| Skills | 34 | 98.6 | 49.3% | BUG-2 缺 `allowed_tools`；7 个缺 Gotchas 区块 |
| **总分** | **69** | **96.0** | — | — |

### 3.2 当时值得借鉴的模式

**模式 1：大多数 skill 100 分** → 根本原因是 c0x12c 有内部 SKILL_AUTHORING.md 规范，每个 skill 都按同一模板写，格式高度一致。→ 原文路径：`toolkit/skills/deep-research/SKILL.md`（满分范例）→ 借鉴：建立一份自己的 SKILL 写作规范，而不是每个文件凭感觉写。

**模式 2：Jinja 模板语法处理命令参数** → 用 `{{ args[0] | default: "" }}` 而非裸露的 `$ARGUMENTS`，当用户不传参数时 Jinja 引擎能优雅降级而非报错。→ 原文路径：`toolkit/commands/spartan/spartan:guard`（同时也是一个缺陷案例，因为该命令本身没加 default，但其他命令有正确用法）→ 借鉴：凡是 command 需要接受可选参数，都用 `{{ args[0] | default('') }}` 包裹。

**模式 3：领域 skill 的精准分层** → terraform 相关 skill 拆成了四个独立文件：`terraform-review`、`terraform-best-practices`、`terraform-module-creator`、`terraform-security-audit`，每个 skill 职责单一而非塞进一个大 skill。→ 原文路径：`toolkit/skills/terraform-*/SKILL.md`→ 借鉴：当一个领域有多个不同操作类型时，拆 skill 比合并更容易复用。

**模式 4：agent 声明明确的 `model:` 字段** → 大部分 agent 头部有 `model: sonnet` 声明，明确锁定使用的模型版本，避免插件行为随默认模型升级而意外漂移。→ 借鉴：给每个 agent 加 `model:` 声明，把「使用什么 AI」作为工件的一部分而非外部配置。

**模式 5：命令命名空间统一** → 所有命令都在 `spartan:` 命名空间下，避免和用户自己的命令冲突。→ 借鉴：插件的命令应有唯一前缀，和系统命令、其他插件区分。

### 3.3 当时的缺陷

**缺陷 1：BUG-1 — `phase-reviewer.md` 声明了错误的工具集**
- 问题：agent body 里调用了 `cat`、`ls`（Bash 命令），但 [frontmatter](#frontmatter) 的 `tools` 数组里只有 `[Read, Grep, Glob, WebSearch]`，没有 `Bash`。
- 根本原因：**工具声明和 body 描述是两个分离的维度，人工维护时极容易不同步。** agent 作者写 body 的时候想的是「要看文件，用 cat」，但填 frontmatter 的时候想的是「Read 工具就够了」——两个认知没有对齐。
- 自查：我的 agent 有没有类似问题？凡是 body 里提到「运行」「执行」「查看目录」的地方，Bash 是否在 tools 里？

**缺陷 2：BUG-2 — `terraform-best-practices/SKILL.md` 缺少 `allowed_tools` [frontmatter](#frontmatter)**
- 问题：该 skill 的 frontmatter 只有 `name:` 和 `description:`，缺少必填的 `allowed_tools` 字段。
- 根本原因：**这是「漏网之鱼」问题**——34 个 skill 里只有这 1 个漏了。根本原因是没有自动化验证：缺字段只在有人运行 NLPM 审查时才被发现，日常开发中不报错，不阻断流程。
- 自查：我的 skill 有没有漏写 `allowed_tools`？

**缺陷 3：Q-1 — 全部 25 个命令缺少 `allowed-tools` [frontmatter](#frontmatter)（系统性）**
- 问题：所有命令缺少工具白名单声明，Claude Code 无法在执行前验证工具访问范围。
- 根本原因：**这是一个「没人告诉过我需要加」的知识盲区**，而非疏忽。命令在没有 `allowed-tools` 的情况下仍然正常运行，所以作者不会因为功能不 work 而被迫填写这个字段。缺少强制验证门控，知识盲区就永久留下来。
- 自查：我的命令有没有 `allowed-tools`？

**缺陷 4：Q-2 — 3 个 agent 零示例**
- 问题：`research-planner`、`idea-killer`、`ai-designer` 三个 agent 完全没有 input → output 示例。
- 根本原因：**示例是最费时间写的部分，在 agent 「能用」之后容易被推迟**。但示例是 Claude Code 校准 agent 行为最重要的机制，没有示例就是在用模糊的期望替代具体的行为规格。
- 自查：我的 agent 有没有至少一个完整的 input → output 示例？

### 3.4 当时的优化机会

1. **批量补全命令 `allowed-tools`**：最高杠杆操作。25 个命令 × -5 = -125 惩罚分全部来自这里。一个批量 PR 可以把命令平均分从 94.8 提到约 99.8。
2. **补充 3 个 agent 的示例**：`research-planner`、`idea-killer`、`ai-designer` 各加一个示例，3 × +15 = 回收 45 惩罚分。
3. **将 `toolkit/rules/` 文档封装为 skill**：现有的 DESIGN_PROCESS.md、GIT_COMMIT.md 等都是高质量领域知识，加 SKILL.md 包装后可被 Claude Code 作为 skill 加载，大幅提升这些规范的可检索性。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状（v1.27.0） | 含义 |
|---|---|---|---|
| BUG-1：phase-reviewer 缺 Bash 工具 | 读取 `toolkit/agents/phase-reviewer.md` frontmatter 的 `tools` 字段 | **已修复**：现版本 frontmatter 有 `model: sonnet`，有 2 个示例，body 不再直接引用 `cat`/`ls` | 高严重度 bug 在 5 个版本内修复，说明团队有 bug 跟进机制 |
| BUG-2：terraform-best-practices 缺 `allowed_tools` | 读取 `toolkit/skills/terraform-best-practices/SKILL.md` frontmatter | **仍存在**：frontmatter 只有 `name:` 和 `description:`，无 `allowed_tools` | 低严重度 bug 被忽略；缺乏自动化验证 |
| Q-1：全部命令缺 `allowed-tools` | grep `-r 'allowed-tools' toolkit/commands/` | **仍存在**：命令从 25 个增长到 61 个，但 `allowed-tools` 依然全军覆没 | 问题随规模扩大；系统性缺陷不会自愈 |

### 4.2 架构演进

从 v1.22.1 到 v1.27.0 的变化：
- **命令数量**：25 → 61（+144%），说明团队在持续扩展覆盖场景
- **agent 数量**：toolkit/ 从 10 → 9（`team-coordinator` 移除）；.codex/ 从 5 → 8（`team-coordinator` 和 `ai-designer` 加入 .codex/）
- **skill 数量**：toolkit/ 从 34 → 35（+1）；.codex/ 从 26 → 26（未变）
- **镜像缺口**：.codex/skills 少 8 个，.codex/agents 少 1 个——缺口没有缩小，甚至在 agent 层面出现了 `team-coordinator` 在 toolkit/ 删除但在 .codex/ 存在的反向不一致

**这说明作者意识到了什么**：命令是用户入口，快速扩展是正确的；但镜像同步是基础设施负担，在没有 CI 守护的情况下难以可靠维护。两件事的维护优先级相差悬殊。

### 4.3 新增的可学习模式

**命令参数的 `argument-hint:` 字段**：v1.27.0 版本的命令 frontmatter 新增了 `name:` 和 `argument-hint:` 字段（v1.22.1 审查时未发现），例如：

```yaml
name: spartan:implement
argument-hint: "<feature description>"
```

`argument-hint` 让 Claude Code 在命令提示界面显示参数说明，用户不需要读文档就知道该传什么参数。这是一个小改动，但对用户体验影响很大——相当于把命令的 usage string 内嵌进了 frontmatter 协议。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责命令**：gstack 的每个命令也是一个具体工程动作（CEO 视角、设计师视角、发布管理等），和 ai-toolkit 的 `spartan:pr`、`spartan:debug` 理念一致——不做泛用命令
2. **skill 的领域分层**：gstack 的 skill 也按领域分文件，没有把所有知识堆在一个大 SKILL.md 里
3. **命名空间隔离**：gstack 命令有自己的前缀，和系统命令不冲突
4. **声明 `allowed-tools`（在 skill 层面）**：根据已知信息，gstack 的 skill 有工具白名单声明，这是 ai-toolkit 在 skill 层面做到的（平均 98.6）但在 command 层面完全缺失的
5. **不过度依赖脚本同步**：gstack 没有「双入口」需求，所以不存在镜像缺口风险

### 5.2 挑战 / 验证

**挑战了的假设**：「命令数量多 = 维护复杂度高 = 质量会下降」。c0x12c 在 61 个命令的规模下仍保持 94.8/100 的命令均分——关键是他们用统一的模板和公司内部规范约束了每个文件的写法。**质量不由数量决定，由写作规范决定。**

**验证了的做法**：「给 agent 写具体示例」的重要性。三个零示例 agent 各被扣 15 分，是本仓库单个缺陷里惩罚最重的。这验证了「示例是校准 agent 行为的第一工具」这个判断。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否有 Bash 命令调用但 tools 中没有 Bash
# (在 gstack 仓库根目录运行)
grep -rn -E '\b(cat |ls |find |mkdir |rm |cd |echo |chmod )\b' ~/.claude/agents/*.md 2>/dev/null || \
grep -rn -E '\b(cat |ls |find |mkdir |rm |cd |echo |chmod )\b' .claude/agents/*.md 2>/dev/null
```
命中后怎么办：打开命中的 agent 文件，检查 frontmatter 的 `tools` 数组是否包含 `Bash`。如果 body 提到了 Bash 操作但 tools 没有声明，补上 `Bash`。

```bash
# 检查我的 skill 是否缺 allowed_tools 字段
grep -rL 'allowed.tools' ~/.claude/skills/*/SKILL.md 2>/dev/null || \
find . -path '*/skills/*/SKILL.md' | xargs grep -rL 'allowed.tools'
```
命中后怎么办：在缺失文件的 frontmatter 末尾加 `allowed_tools: [Read, Glob, Grep]`（根据 skill 实际需要调整）。

```bash
# 检查我的 command 是否缺 allowed-tools 字段
find . -path '*/commands/*.md' | xargs grep -L 'allowed.tools' 2>/dev/null
```
命中后怎么办：批量为命令 frontmatter 添加 `allowed-tools` 字段，参考该命令 body 里提到的操作类型决定白名单内容。

```bash
# 检查我的 agent 是否有零示例（没有 ## Example 区块）
grep -rL '## Example' ~/.claude/agents/*.md 2>/dev/null || \
find . -path '*.claude/agents/*.md' | xargs grep -L '## Example'
```
命中后怎么办：为每个命中的 agent 添加至少一个具体 input → output 示例，说明：「用户说了什么」→「agent 做了什么、输出了什么」。

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 的命令补全 `allowed-tools` 声明
   - **为何可行**：gstack 命令数量（约 23 个）比 ai-toolkit 还少，批量操作 30 分钟可完成
   - **第一步**：`find . -path '*/commands/*.md' -print0 | xargs -0 grep -L 'allowed-tools'` 列出所有缺失文件，然后读每个命令 body，根据实际调用的工具类型填写 `allowed-tools` 数组

2. **想法**：为 gstack 的每个 agent 补充至少一个 input → output 示例
   - **为何可行**：示例格式固定（`## Example`、`**Input:**`、`**Output:**`），每个约 10-20 行，每个 agent 大约 15 分钟
   - **第一步**：打开最常用的 agent（如 CEO 角色 agent），想「上次我用这个 agent 做了什么」，把那次对话的输入/输出提炼成示例写进去

3. **想法**：参考 `compile-packs.js` 的模式，为 gstack 写一个 frontmatter 验证脚本
   - **为何可行**：验证逻辑简单——解析 YAML frontmatter，检查必填字段是否存在，不需要 AI 参与
   - **第一步**：写一个 Python 脚本（约 50 行），输入一个目录，输出所有 `.md` 文件中缺失必填 frontmatter 字段的报告；绑定到 git pre-commit hook

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（2026-07-27 刷新）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 c0x12c/ai-toolkit 的核心目的**：把公司工程规范打包成 Claude Code 插件，为全团队提供统一的 AI 辅助开发框架（多 agent + 多 skill + 多 command，覆盖全栈开发到创业运营）

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 也是「workflow framework for Claude Code」；多 skill + 多 agent；基于 Garry Tan 的 GSD 思想（来源一致！） | gstack 更个人化，没有公司规范；命令数量少（~23 vs 61）；没有双入口架构 | 高 |
| MarkQWu/- | 低 | 有 NL 工件 | 目的完全不同（全球情报仪表板 vs 工程框架） | 低 |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件 | bureau 是知识库管理，ai-toolkit 是工程框架 | 低 |

gstack 是本案例最直接的对照——两者都在实现「Garry Tan 的 Get Stuff Done 工程哲学」，只是 gstack 是个人实现，ai-toolkit 是公司级实现。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `allowed-tools`（Q-1，系统性） | `find . -path '*/commands/*.md' \| xargs grep -L 'allowed-tools'` | gstack 命令大概率命中（同类工程框架，此字段常被忽略） | 高 |
| skill 缺 `allowed_tools`（BUG-2 类型） | `find . -path '*/skills/*/SKILL.md' \| xargs grep -L 'allowed_tools'` | bureau 的 skill 需验证 | 中 |
| agent 零示例（Q-2 类型） | `find . -path '*agents/*.md' \| xargs grep -L '## Example'` | gstack 和 bureau 的 agent 均需验证 | 中 |
| agent body 与 tools 声明不一致（BUG-1 类型） | `grep -rn 'cat \|ls \|find \|mkdir ' .claude/agents/*.md` + 检查 tools 声明 | 概率低但高危，需主动排查 | 高 |

**命中后的具体行动建议**：
- gstack 的所有 `.claude/commands/*.md` → 在每个 frontmatter 末尾加 `allowed-tools: [Read, Write, Edit, Bash, Glob, Grep]`（根据实际调整）→ 约 30 分钟可完成批量操作
- bureau 的 `skills/*/SKILL.md` → 检查并补全 `allowed_tools` frontmatter → 每个文件约 2 分钟
- gstack 最常用的 agent（CEO、Designer、EM）→ 各加一个 `## Example` 区块 → 每个约 15 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：「`argument-hint:` 参数提示字段」
   - **本案例做法**：v1.27.0 命令 frontmatter 加了 `argument-hint: "<feature description>"`，让用户在输入命令时看到参数提示（文件路径：任意 `toolkit/commands/spartan/*.md` v1.27.0 版本）
   - **我的项目现状**：gstack 命令没有 `argument-hint` 字段，用户需要读文档才知道该传什么
   - **如何借鉴**：为 gstack 里需要接受参数的命令（如 `/gstack:review`、`/gstack:debug`）加 `argument-hint: "<what to pass>"` 到 frontmatter；约 5 分钟完成所有命令

2. **领域**：「编译脚本守护双入口一致性」
   - **本案例做法**：`compile-packs.js` 在 npm `prepublishOnly` 时自动从 toolkit/ 生成 .codex/，确保发布时两者同步（文件路径：`compile-packs.js`）
   - **我的项目现状**：gstack 是单入口，没有双入口需求。但 gstack 目前没有任何发布前验证脚本——如果将来支持多 CLI 入口，没有这个机制会立刻产生同步问题
   - **如何借鉴**：即使现在不需要双入口，可以借鉴「把 frontmatter 验证绑到 prepublishOnly」的思路，写一个 validate-nl-artifacts.js 在发布前检查所有 NL 工件的必填字段

3. **领域**：「Jinja 模板语法处理命令参数」
   - **本案例做法**：命令用 `{{ args[0] | default('') }}` 代替裸露的 `$ARGUMENTS`，有参数时展开，无参数时降级为空字符串，不报错（文件路径：`toolkit/commands/spartan/` 中多个命令）
   - **我的项目现状**：gstack 命令用 `$ARGUMENTS` 直接展开，如果用户漏传参数，行为未定义
   - **如何借鉴**：把 gstack 里「需要参数才能工作」的命令中的 `$ARGUMENTS` 替换为 `{{ args[0] | default('') }}`，并在命令 body 开头加空参数检查（如果参数为空就输出使用说明）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：「Skill 层的 `allowed_tools` 声明完整度」
  - **我的做法**：gstack 的 skill 有 `allowed-tools` 声明（根据已知信息）
  - **本案例做法**（弱在哪）：ai-toolkit 命令层完全缺失 `allowed-tools`（61 个命令全部缺失）；skill 层绝大多数正确，但仍有 1 个（terraform-best-practices）在两次审查间均缺失 `allowed_tools`
  - **意义**：gstack 在命令和 skill 的工具白名单声明上比 ai-toolkit 更完整，这是一个可以在 PR 中指出的改进点；同时也验证了「给 skill 加 allowed-tools 是高优先级」这一判断

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读取命令、agent、skill 文件时，首先解析 frontmatter，才能知道这个文件怎么注册和调用。**缺少必填 frontmatter 字段，文件要么无法加载，要么以未定义行为运行。**

### <a name="allowed-tools"></a>allowed-tools / allowed_tools
> Command frontmatter 里的 `allowed-tools` 字段（Skill frontmatter 里叫 `allowed_tools`）。声明这个 NL 工件在执行时允许 Claude 使用哪些工具（如 `Read`、`Write`、`Bash`、`Glob` 等）。**不声明不等于没有限制——它等于「Claude Code 不知道该用什么」**，可能导致权限不足或权限过宽。
> 注意命令和 skill 的字段名有细微差异：命令用连字符 `allowed-tools`，skill 用下划线 `allowed_tools`。

### <a name="skill"></a>skill
> Claude Code 中的「可加载知识单元」。一个 SKILL.md 文件就是一个 skill——它包含 frontmatter（描述、工具白名单）和正文（领域知识、约定、示例）。Agent 在 frontmatter 的 `skills:` 字段声明需要哪些 skill，Claude Code 会在 agent 运行时自动注入这些知识。**Skill 和工具（Tool）不同**：skill 是知识，tool 是能力。

### <a name="spartan-框架"></a>Spartan 框架
> c0x12c 基于「Get Stuff Done」理念设计的工程方法论，核心是「去掉废话，只做推进项目的事」。在 ai-toolkit 里，Spartan 框架通过 `spartan:` 命名空间的命令实现——每个命令对应一个具体工程动作（写 commit、做 PR、debug、review 等），避免了泛用式 AI 对话。

### <a name="compile-packs"></a>compile-packs.js / 编译步骤
> ai-toolkit 仓库中的一个 Node.js 脚本，负责将 `toolkit/`（源文件）的内容同步到 `.codex/`（发行版目录）。绑定在 npm 的 `prepublishOnly` 生命周期钩子上——只有在执行 `npm publish` 前才自动运行。**「编译」在这里不是指把代码变成机器码，而是「从源格式生成分发格式」的处理步骤。**

### <a name="prepublishOnly"></a>prepublishOnly
> npm 的生命周期钩子（lifecycle hook）之一，在执行 `npm publish` 之前自动运行声明的脚本。ai-toolkit 用它来运行 `compile-packs.js`，确保每次发布时 `.codex/` 都是 `toolkit/` 的最新镜像。等价于「发布前的自动检查 / 生成步骤」。

### <a name="Jinja-模板"></a>Jinja 模板语法
> 一种用于文本模板的语法规范（来源于 Python 社区），在 Claude Code 命令文件里用来处理用户传入的参数。语法形如 `{{ args[0] }}` 表示「第一个参数」，`{{ args[0] | default('') }}` 表示「第一个参数，如果没传则用空字符串代替」。相比直接用 `$ARGUMENTS`，Jinja 语法支持默认值，避免了「用户忘记传参数时命令报错」的问题。

### <a name="镜像缺口"></a>镜像缺口
> 在「单仓双入口」架构中，`toolkit/`（源文件）有某些文件，但 `.codex/`（发行版）里没有对应镜像的情况。镜像缺口意味着：通过 Codex CLI 安装插件的用户无法访问这些工件，而通过 Claude Code 直接安装的用户可以访问——两种用户获得的功能集不一致。ai-toolkit 在 v1.27.0 仍有 9 个 skill 和 1 个 agent 存在镜像缺口。
