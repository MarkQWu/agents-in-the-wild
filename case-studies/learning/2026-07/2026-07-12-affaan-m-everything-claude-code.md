# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：N/A（注册时未记录）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-12（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `model-pinning`, `curl-pipe-bash-risk`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`everything-claude-code` 是一个面向 Claude Code 生态的"全域 NL 工具箱"——单仓收录 80+ 领域 [SKILL.md](#SKILL.md)、20+ 专用 agent、94 个命令，并提供完整的六语言（ko-KR / zh-TW / pt-BR / tr / ja-JP / zh-CN）本地化副本和 Kiro IDE（`.kiro/`）适配层。作者 Affaan Mustafa 将其定位为"Claude Code 的一站式超能力套装"，包含从 DeFi 安全到 Perl 测试、从视频剪辑到 Django TDD 的几乎所有垂类场景。

关键事实：
- 创建时间：2025 年下半年（GitHub 推断）
- 总 [artifact](#artifact) 数量：934（审计时进行渐进采样 ~320）
- 分发方式：GitHub star + ECC 生态自传播，配合 install.sh / install.ps1 一键安装
- 在生态中的位置：ECC（everything-claude-code）是 Claude Code 非官方社区最活跃的技能仓库之一，被多个下游仓库直接引用

### 1.2 架构剖析

**目录结构（关键层级）**：
```
everything-claude-code/
├── skills/            # 80+ 领域技能，每个子目录含 SKILL.md
├── agents/            # 20+ 专用 agent（reviewer / orchestrator / builder 等）
├── commands/          # 94 个命令（当前 HEAD）
├── hooks/             # hooks.json + 7 个 .sh / 50+ .js 脚本
├── docs/              # 6 语言本地化副本（ko-KR/zh-TW/pt-BR/tr/ja-JP/zh-CN）
├── .kiro/             # Kiro IDE 专用 skills + agents
├── scripts/           # 格式化、类型检查、MCP 健康检测等钩子脚本
├── manifests/         # 多 plugin.json
├── examples/          # CLAUDE.md 示例
└── ecc2/              # v2 版本（开发中）
```

**文件类型分布**（当前 HEAD 实测）：
- SKILL.md：80+ 个（多数得分 100/100）
- agent：20+ 个
- command：94 个
- hook：hooks.json × 1 + JS hook 脚本 50+
- 本地化副本：约 7× 倍增

**编排关系**：
- Skills 作为「参考知识」被 agent 和 command 加载（通过 [frontmatter](#frontmatter) 中 `skills:` 声明）
- Commands 可以通过 `AgentTask` 调度特定 agent（如 `commands/gan-build.md` → `agents/gan-planner.md`）
- `.kiro/` 是平行层，镜像部分核心技能，面向 Kiro IDE 用户
- `docs/` 本地化目录完全镜像英文主目录，**不通过 `include` 机制引用**，而是手动维护副本

**跨件契约**：
- 主要通过文件路径约定传递上下文，无显式的 router 或 meta skill
- `plugin.json`（[manifest](#manifest)）在 `manifests/` 目录下集中管理

### 1.3 设计思路 / 方法论

- **核心设计哲学**："广度优先 + 垂类深度"——先建立覆盖面极广的技能库，再在高频垂类（如 Kotlin、Go、Rust）中做到深度
- **解决什么问题**：开发者安装 Claude Code 后缺乏"领域专家技能"，不知道怎么让模型在特定技术栈中真正有用
- **Trade-off**：
  - 牺牲单点精深（如 Playwright 技能仅 88 分）换取覆盖面（80+ 垂类）
  - 手动维护 6 语言副本（高成本）换取国际化用户覆盖
  - 本地化机制引入了"源文件 bug 指数放大"的脆弱性
- **认知模型**：作者将 Claude Code 技能库视为"可安装的领域专家"——装了就有，不装则无，独立于主代码库

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「全域 NL 工具箱」(Universal NL Toolkit)

特征清单：
- 特征 1：**单仓多垂类**——80+ 独立 SKILL.md，按领域名命名子目录，可单独安装
- 特征 2：**CI 驱动质量**——eslint.config.js + pyproject.toml + jest 测试套件，质量门控内嵌
- 特征 3：**手动维护国际化**——6 个 `docs/` 子目录是英文源的翻译副本
- 特征 4：**双 IDE 适配**——`.kiro/` 目录为 Kiro IDE 提供独立的 skill/agent 集合
- 特征 5：**命令丰度超过深度**——94 个命令覆盖几乎所有场景，但平均完整度偏低（大量缺 `allowed-tools`）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|------|---------|------|
| 技术栈多样、需要跨领域技能共享 | ✅ 高度适用 | 单仓安装，一次获得所有技能 |
| 国际化产品，需要多语言技能 | ⚠️ 谨慎适用 | 本地化副本维护成本高，bug 传播风险大 |
| 需要精深某一垂类（如 Playwright）| ❌ 不适用 | 广度优先导致垂类深度不够 |
| 单人小项目快速上手 | ✅ 适用 | install.sh 一键安装，立刻可用 |
| 企业内部标准化技能库 | ❌ 不适用 | 无权限控制，无版本锁定机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|------|---------|------|------|
| 全域 NL 工具箱（本仓库）| everything-claude-code | 覆盖面极广，一键安装 | 质量不均，本地化维护成本高 |
| 垂类深耕仓库 | alirezarezvani/claude-skills | 垂类质量极高（92分），ra-qm 多数 100 分 | 覆盖面受限，需多仓协作 |
| 精品 Marketplace | ananddtyagi/cc-marketplace | 社区共同贡献，多样性高 | 质量参差，无统一质量门控 |

### 2.4 改进空间

1. **当前问题**：94/94 命令缺 `allowed-tools` 声明，Claude Code 无法知道哪些工具被允许 **改进做法**：以 `commands/skill-create.md`（唯一 100 分命令）为模板，批量添加 `allowed-tools: [Read, Grep, Glob]`（大多数命令只需读权限）**预期收益**：权限透明化，减少用户意外授权
2. **当前问题**：本地化副本手动维护，源文件 bug 指数放大（1 个 bug → 14 个副本） **改进做法**：改用单一英文源 + 运行时 i18n 参数，或在 CI 中加入"本地化副本同步检查" **预期收益**：消除 localization drift
3. **当前问题**：zh-TW / ja-JP agents 使用 `model: opus`，英文版用 `model: sonnet`，成本 5× **改进做法**：CI 中加入跨本地化 model-tier 一致性检查 **预期收益**：避免意外高成本

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 84/100（进行性扫描 ~320 个 artifact）

| 文件 | 当时分数 | 主要问题 |
|------|---------|---------|
| commands/test-coverage.md | 23 | 缺 name + description frontmatter |
| commands/gan-build.md | 25 | 缺 frontmatter；引用未声明 agent 依赖 |
| skills/customs-trade-compliance/SKILL.md | 55 | 模糊量词过多 + 缺输出格式 |
| agents/e2e-runner.md | 67 | 缺输出格式；模糊量词 9× |
| commands/skill-create.md | 100 | 完美 |
| skills/dart-flutter-patterns/SKILL.md | 84 | "appropriate" 出现 8 次 |
| 80+ 其他 skills | 100 | 完美 |

### 3.2 当时值得借鉴的模式

1. **精品 Skill 100 分矩阵** → 80+ 个 domain skill 达到满分，说明作者对"好的 SKILL.md 应该是什么样"非常清楚。根本原因：有明确示例驱动（`<examples>` block + input→output）。原文示例路径：`skills/dart-flutter-patterns/SKILL.md`。借鉴：每个 skill 都要有 2+ concrete 示例

2. **`commands/skill-create.md` = 事实标准** → 唯一得满分的 command，格式规范到可以当模板。根本原因：有 `name`, `description`, `allowed-tools`, 完整 numbered steps, 明确 output format。借鉴：所有新 command 应 diff `skill-create.md` 格式

3. **hooks.json 版本固定反面示例** → `npx block-no-verify@1.1.2` 虽然 pin 了版本，但每次调用都重新从 npm 拉包，在 critical path 上形成供应链攻击面。根本原因：`npx` 的语义是"每次都执行"，非真正的锁定。借鉴：hooks 应该 bundled 进本地 `node_modules`，而非每次 npx

4. **跨语言本地化矩阵** → 6 语言扩展触达国际用户群。根本原因：社区驱动翻译。如何借鉴：先做英文高质量版，再机器翻译+人工校对

5. **greptile.json + CI 工具链** → 内嵌代码索引和持续集成，体现"质量文化"。借鉴：为自己的 NL 工具库建立质量门控

### 3.3 当时的缺陷

1. **commands/learn.md 缺 `name` + `description`（传播到 14 个本地化副本）** → 为什么失败：frontmatter 中没有 `name` 字段，Claude Code 无法以命名命令的方式注册它；14 个本地化副本继承此 bug，修一处才能修全部。自查：`bureau` 和 `echo-sleuth` 的命令是否有同样问题？（答案：是的）

2. **zh-TW / ja-JP agents 使用 `model: opus`，英文原版用 `model: sonnet`** → 为什么会失败：多语言用户运行 zh-TW agent fleet 时成本是英文用户的 5 倍，而能力无差异。根本原因：翻译时没有保留 `model` 字段原值。自查：我的任何多语言副本是否有此问题？

3. **94 个命令全部缺 `allowed-tools`** → 为什么失败：Claude Code 不知道命令需要哪些工具，用户每次都得手动授权，或者过度信任"*"。根本原因：没有把 `allowed-tools` 当成必填字段对待。自查：`bureau` 的 5 个命令全部缺 `allowed-tools`

4. **agents/gan-evaluator.md 用 Playwright 但未在 `tools` 列表声明** → 为什么失败：runtime tool call 失败，agent 崩溃。根本原因：描述了工具行为但忘记在 frontmatter 中声明。

5. **依赖传递 bug：commands/gan-build.md 调度 gan-planner/gan-generator/gan-evaluator 但未声明为依赖** → 为什么失败：若 agent 插件未安装，dispatch 静默失败，用户得不到任何错误提示。根本原因：NL 层缺乏依赖注入机制，只靠名字引用，无编译时检查。

### 3.4 当时的优化机会

1. **以 `commands/skill-create.md` 为模板批量补全 59 个缺 `allowed-tools` 的命令**（当时数量；现已增至 94）
2. **在 CI 中加入跨本地化 model-tier 一致性检查**，阻止 zh-TW/ja-JP 用错误 model tier
3. **将 hooks.json 的 npx 调用替换为本地捆绑脚本**（安全问题，需私下披露）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---------|---------|------|------|
| hooks/hooks.json 中的 `npx block-no-verify@1.1.2` 供应链风险 | `grep -n 'block-no-verify' hooks/hooks.json` | ✅ **已修复**：结果为空 | 安全问题得到解决 |
| commands/learn.md 缺 `name` 字段 | `head -10 commands/learn.md` | ⚠️ **部分修复**：有了 `description` 但仍无 `name` | 命令仍无法正确注册为命名命令 |
| zh-TW agents 用 `model: opus` | `grep -rn 'model: opus' docs/zh-TW/agents/` | ❌ **仍存在**：确认 5 个文件（refactor-cleaner / architect / go-reviewer / planner / e2e-runner）仍用 opus | 高成本风险持续 |
| 94 个命令缺 `allowed-tools` | `find commands/ -name '*.md' \| xargs grep -L allowed-tools` | ❌ **更严重**：94/94 全部缺（当时 59/72，现更差）| 仓库继续添加命令但未补全该字段 |
| skills/ralphinho-rfc-pipeline/SKILL.md 缺 frontmatter | `head -5 skills/ralphinho-rfc-pipeline/SKILL.md` | ✅ **已修复**：现在有正确的 `name` 和 `description` | 该 skill 可以正常注册 |

### 4.2 架构演进

从 audit 到现在的主要变化（推断）：
- `commands/eval.md` 已被**删除**（而非修复缺失 frontmatter），说明作者选择移除而非维护这个命令
- `hooks/hooks.json` 完成了安全加固，供应链攻击面消除
- 命令数量从 72（audit 时样本中可见）增长到 94，但 `allowed-tools` 补全工作未随之进行
- `skills/ralphinho-rfc-pipeline/SKILL.md` frontmatter 被修复，说明有人在看 NLPM 的 bug 报告

### 4.3 新增的可学习模式

- **`ecc2/` 目录**：v2 版本目录出现，说明作者在酝酿重大架构重构（暂无详情）
- **`.opencode` 本地化**：出现了第 7 种本地化目录（opencode 平台），说明仓库正在往多 IDE 方向扩张
- **删除而非修复**：`commands/eval.md` 被删除的做法揭示了一种"断舍离"维护哲学——质量差的就删，不修补

---

## 五、校准

### 5.1 我已经在做对的

1. **有独立的 skill 子目录**：echo-sleuth-for-claude 已有 skills/experience-synthesis 等目录，与此仓库单目录单 SKILL.md 的组织方式一致
2. **有 agents + commands 分离**：echo-sleuth 已将 agent 和 command 分开管理，与本仓库同构
3. **有测试目录**：echo-sleuth 有 tests/ 目录，体现质量文化
4. **不做手动本地化**：我的仓库未维护多语言副本，避免了 localization-drift 风险

### 5.2 挑战 / 验证

本案例**验证**了一个我之前犹豫的做法：**`allowed-tools` 对所有命令都是必填字段，不是可选的**。此仓库 94 个命令一个都没有，导致权限混乱——这是活生生的反面案例。此后我的所有命令都必须声明 `allowed-tools`。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令是否缺 name 字段
grep -rL '^name:' ~/my-repos/*/commands/*.md 2>/dev/null

# 检查我的命令是否缺 allowed-tools
grep -rL 'allowed-tools' ~/my-repos/*/commands/*.md 2>/dev/null

# 检查我的 agent 是否缺 model 字段
grep -rL '^model:' ~/my-repos/*/agents/*.md 2>/dev/null

# 检查我的 skill 是否有 examples 块
grep -rL '<examples>\|## Examples\|example' ~/my-repos/*/skills/*/SKILL.md 2>/dev/null
```

命中后怎么办：
- 缺 `name`：在 frontmatter 第一行加 `name: <命令 slug>`，如 `name: audit`
- 缺 `allowed-tools`：根据命令体分析所用工具，加 `allowed-tools: [Read, Grep, Glob]`（最小权限原则）
- 缺 `model`：加 `model: claude-sonnet-5` 或 `model: claude-haiku-4-5-20251001`（haiku 用于机械任务）
- 缺 examples：至少加 2 个 `<example>` 块，含 input → output 对

### 6.2 灵感 → 实施路径

1. **想法**：以 `commands/skill-create.md`（100 分）为模板，创建一个命令模板文件
   - **为何可行**：该命令已经证明格式最优，照抄即可
   - **第一步**：复制 `commands/skill-create.md` 的 frontmatter 部分到 `.claude/command-template.md`，去掉 body，留出 placeholder——15 分钟

2. **想法**：为 bureau 仓库的所有命令补全 `name` + `allowed-tools`
   - **为何可行**：命令数量少（5 个），工作量小
   - **第一步**：打开 `MarkQWu/bureau/commands/compile.md`，加 `name: compile` 和 `allowed-tools: [Read, Write, Bash]`——10 分钟 × 5 个文件

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 affaan-m/everything-claude-code 的核心目的**：为 Claude Code 用户提供一站式、多垂类的 NL 技能库

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|-------|-------|-------|----------|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code 插件；有 skills + agents + commands | 功能单一（对话历史挖掘），非全域工具箱 | 高 |
| MarkQWu/bureau | 中 | 同为 Claude Code 插件；有 commands + hooks | 知识库管理，非技能库 | 高 |
| MarkQWu/gstack | 低 | 也有 SKILL.md | NL 表皮 + 原生 CLI 二进制，完全不同架构 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---------|---------|--------------|------|
| commands 缺 `name:` 字段 | `grep -rL '^name:' .../commands/*.md` | **echo-sleuth 命中 4/8**（lessons/recall/recap/timeline）；**bureau 全中 5/5** | 高 |
| commands 缺 `allowed-tools` | `grep -rL 'allowed-tools' .../commands/*.md` | **bureau 全中 5/5** | 高 |
| agents 缺 `model:` 字段 | `grep -rL '^model:' .../agents/*.md` | **echo-sleuth 全中 5/5**（analyze/file-historian/memory-auditor/recall/schema-scout）| 高 |
| skills 缺 `<examples>` 块 | `grep -rL 'example' .../skills/*/SKILL.md` | **echo-sleuth 全中 4/4** | 高 |

**具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude/commands/lessons.md` → 加 `name: lessons`，`allowed-tools: [Read, Bash]` → 10 分钟
- `MarkQWu/bureau/commands/compile.md` 等 5 个文件 → 各加 `name:` 和 `allowed-tools:` → 30 分钟总计
- `MarkQWu/echo-sleuth-for-claude/agents/*.md` × 5 → 各加 `model: claude-haiku-4-5-20251001` → 15 分钟
- `MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md` 等 4 个 → 各加 2 个 `<example>` 块 → 1 小时总计

### 8.3 别人的更优方案

1. **领域**：Skill 100 分的写法
   - **本案例做法**：`skills/dart-flutter-patterns/SKILL.md` 有完整的 frontmatter + 详细示例 + 具体输出格式
   - **我的项目现状**：`MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md` 缺乏具体 `<example>` 块
   - **如何借鉴**：打开 `skills/dart-flutter-patterns/SKILL.md` 的 examples 部分，照抄格式 → 在 echo-sleuth 的每个 skill 中加入 2 个 `<example>` 块

2. **领域**：质量门控内嵌 CI
   - **本案例做法**：eslint.config.js + pyproject.toml + jest → 推送前自动检查
   - **我的项目现状**：echo-sleuth 有 tests/ 但无 CI 集成
   - **如何借鉴**：在 echo-sleuth 的 `.github/workflows/` 中加一个简单的 `nlpm-check.yml`（参考 `templates/workflows/nlpm-check.yml`）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：不维护手动本地化副本
- **我的做法**：echo-sleuth 只有英文版，无本地化目录
- **本案例做法（弱在哪）**：6 个本地化目录，bug 传播率 14×；zh-TW/ja-JP 还有 model-tier 不一致的额外成本风险
- **意义**：我"不做本地化"的决策是对的——如果有国际化需求，应该用运行时 i18n 参数而非手动维护副本

---

## 八、术语表

### <a name="SKILL.md"></a>SKILL.md
> Claude Code 技能的定义文件。放在某个子目录里，包含 frontmatter（描述技能的元数据）和技能内容（模型应该怎么做、有哪些示例）。Claude Code 读取它来"安装"一个技能到当前会话中。

### <a name="artifact"></a>artifact
> NLPM（本工具）扫描后识别出的 NL 制品，包括 SKILL.md、agent 文件、command 文件、hooks.json 等。每个 artifact 都会被打分和检查。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`tools` 等）。Claude Code 读取 frontmatter 才能知道如何注册和调用这个文件。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。
