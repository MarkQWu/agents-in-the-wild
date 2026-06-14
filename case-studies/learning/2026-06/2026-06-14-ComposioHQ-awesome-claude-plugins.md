# ComposioHQ/awesome-claude-plugins — 学习案例

**仓库**：https://github.com/ComposioHQ/awesome-claude-plugins
**Stars**：未收录（registry 未更新）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-14（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `single-purpose`, `vague-quantifier`, `cross-reference`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`awesome-claude-plugins` 是 [Composio](https://composio.dev) 维护的一个**多作者插件集合仓库**，将来自不同贡献者的 25 个独立 Claude Code 插件聚合在同一 GitHub 仓库中。用户可以按需安装其中任一插件。

关键事实：
1. Composio 是一家提供 AI agent 工具集成平台的公司，该仓库是其社区生态的一部分
2. 仓库结构是「插件 monorepo」：每个子目录都是一个完整插件，含自己的 `plugin.json`、commands/、skills/、agents/
3. 2026-04-06 审计时共 88 个 NL 工件，涵盖 12 个 agent、26 个 command、23 个 skill
4. 质量分化极大：`perf/` 和 `skill-bus/` 系列接近完美（97-100 分），而 `commit/`、`create-pr/`、`bug-fix/` 等实用命令几乎是空壳
5. 存在一个重要的隐藏运行时依赖：`ship.md` 和 `perf.md` 均通过 `require()` 调用 `@awesome-slash/lib` 等 Node.js 库模块，但这些依赖未在任何插件的 manifest 中声明

### 1.2 架构剖析

**目录结构**（关键层级）：
```
awesome-claude-plugins/
├── perf/                    # 性能分析 — 8 个技能 + 5 个 agent + 1 个命令 (全部 85-100 分)
│   ├── .claude-plugin/plugin.json
│   ├── commands/perf.md
│   ├── agents/              # perf-orchestrator, theory-gatherer, profiler...
│   └── skills/              # analyzer, benchmark, baseline, code-paths...
├── ship/                    # CI/CD 发布 — 1 个主命令 + 3 个引用文档
│   ├── commands/ship.md
│   └── commands/ship-deployment.md    # ← 无 frontmatter（BUG）
├── skill-bus/               # 技能订阅系统 — 9 个命令 + 4 个技能 + 3 个 hooks
│   ├── hooks/dispatch.sh
│   └── ...
├── commit/                  # Git 提交助手 — 仅 1 个命令
├── create-pr/               # PR 创建 — 仅 1 个命令
├── bug-fix/                 # Bug 修复 — 仅 1 个命令
├── audit-project/           # 项目审计 — 1 个命令 + 2 个引用文档
├── debugger/                # 调试器 — 1 个 agent
├── code-review/             # 代码审查 — 1 个命令
├── connect-apps/            # Composio 应用连接 — 1 个命令
└── ...（共 25 个插件目录）
```

- **文件类型分布**：23 个 skill（avg 97/100）、12 个 agent（avg 80/100）、26 个命令（avg 84/100）、4 个 hooks（2 Python + 4 Bash）、25 个 plugin.json
- **编排关系**：各插件完全独立，没有跨插件的 router 或 meta-skill。`perf` 内部有明确的分层：`perf.md` 命令 → `perf-orchestrator` agent → 分发给 7 个专业 agent → 7 个对应 skill
- **跨件契约**：`ship.md` 引用 `ship-deployment.md` / `ship-ci-review-loop.md` 作为「相位参考文档」，通过文件路径隐式依赖；`skill-bus` 命令与技能形成镜像配对（add-sub 命令 ↔ add-sub 技能）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：社区贡献优先、质量参差兼容。Composio 的定位不是发布一个质量标准统一的插件，而是搭建一个贡献者生态 —— 这解释了为什么同一个仓库里既有接近 100 分的 `perf` 套件，也有 65 分的空壳命令
- **解决什么问题**：原始问题是「如何让工程师快速获取可用的 Claude Code 增强工具」，monorepo 模式让用户可以一眼看到所有可选插件
- **Trade-off**：单仓多插件 vs 每个插件独立仓库 —— 前者降低了贡献者门槛（fork 一个仓库即可），但牺牲了版本独立性和质量一致性。25 个 plugin.json 中有 9 个缺少 `version` 字段就是这种 trade-off 的直接体现
- **认知模型**：把 Claude Code 插件视为「可安装的工作流程模块」，每个插件解决工程师日常的一个具体痛点（提交、发 PR、性能分析、调试）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「开放社区插件集合（Curated Plugin Monorepo）」**

关键特征：
- 单 repo 托管多个独立插件，每个插件有自己的 `plugin.json` + 完整目录
- 贡献者各自维护自己的插件目录，质量标准差异是预期而非意外
- 顶层 README 作为插件目录和质量指南入口
- 没有跨插件的共享 skill 或 agent（完全解耦）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 公司对外展示 AI 工具生态 | ✅ 高度适用 | 一个仓库即可展示多个插件，贡献者门槛低 |
| 内部工程团队统一工具链 | ⚠️ 有条件适用 | 需要额外设置质量门控（CI 检查、PR 模板） |
| 个人开发者快速探索 Claude Code 能力 | ✅ 适用 | 一次 clone 即可看到 25 种不同用法 |
| 需要严格质量保证的生产插件 | ❌ 不适用 | 多作者风格混杂，无法保证一致的质量水平 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：社区集合 monorepo | ComposioHQ/awesome-claude-plugins | 生态可见性高、贡献门槛低 | 质量分化大、版本管理复杂 |
| 备选 A：单一高质量插件 | kepano/obsidian-skills | 质量可控、迭代专注 | 功能单一、贡献者少 |
| 备选 B：按域拆分独立仓库 | Anthropic 官方模式 | 每个插件独立发版、质量独立管控 | 用户发现成本高、维护碎片化 |

### 2.4 改进空间

1. **当前问题**：9 个 plugin.json 缺少 `version` 字段，无法做版本兼容管理 **改进做法**：在 CI 中加一条 `jq` 检查，如果 plugin.json 缺少 `version` 字段就阻止合并 **预期收益**：未来不会再出现同样遗漏
2. **当前问题**：`commit/`、`create-pr/` 等命令缺少 `allowed-tools`，实际上无法执行任何工具调用 **改进做法**：在 PR 模板中加「allowed-tools checklist」，要求贡献者声明命令需要哪些工具 **预期收益**：消除「看起来能用但实际跑不了」的空壳命令
3. **当前问题**：`ship.md` 和 `perf.md` 依赖 `@awesome-slash/lib` 等运行时 Node.js 库但未在任何 manifest 中声明 **改进做法**：在各插件的 plugin.json 中增加 `dependencies` 字段，或在 README 中明确写出安装前提 **预期收益**：用户不会在缺少依赖时遇到神秘的运行时失败

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **88/100**（加权平均，61 个 NL 文件）。

| 文件类型 | 平均分 | 主要问题 |
|---|---|---|
| Skills (23 个) | 97/100 | 极少数有模糊量词，整体接近满分 |
| Commands (26 个) | 84/100 | 6 个命令缺 allowed-tools；5 个引用文档缺 frontmatter |
| Agents (12 个) | 80/100 | 大多数缺示例（-15）；部分缺 model 声明（-5）；2 个完全没有 tools 字段 |

### 3.2 当时值得借鉴的模式

1. **`perf` 套件的层次化分工** → 每个 agent 只做一件事（收集理论 / 执行代码路径 / 分析 / 记录日志），各有专属 skill，顶层 `perf-orchestrator` 统一调度 → 路径 `perf/agents/` + `perf/skills/` → 借鉴：多步骤工作流用「指挥 + 专家」模式，避免单个 agent 承担全部工作
2. **`skill-bus` 的命令与技能镜像配对** → `add-sub.md` 命令 ↔ `add-sub/SKILL.md`，功能对齐、命名一致 → 路径 `skill-bus/commands/` + `skill-bus/skills/` → 借鉴：命令是用户入口，技能是可复用知识，两者应当有明确的分工而不是功能重叠
3. **`changelog-generator/SKILL.md` 达到 100 分** → frontmatter 完整、有具体输出格式、无模糊量词 → 路径 `changelog-generator/skills/changelog-generator/SKILL.md` → 借鉴：一个好的 SKILL.md 就是「名称 + 触发条件 + 分步说明 + 具体输出格式」，缺一不可
4. **`security_reminder_hook.py` 的安全提醒模式** → 每次写入敏感文件前触发提醒，属于「写时检查」而非「用时报错」→ 路径 `security-guidance/hooks/security_reminder_hook.py` → 借鉴：安全检查放在工具调用的 hook 层，比放在命令里更可靠
5. **`connect-apps/commands/setup.md` 满分** → 清晰的步骤式 setup 命令，每步都有可验证的输出 → 借鉴：配置类命令不要写「根据需要配置」，而要写「第 1 步做什么、预期输出是什么」

### 3.3 当时的缺陷

1. **5 个「引用文档」没有 [frontmatter](#frontmatter)** (`ship-deployment.md`、`ship-ci-review-loop.md`、`ship-error-handling.md`、`audit-project-agents.md`、`audit-project-github.md`) → 根本原因：作者把这些文件当成「ship.md 的附件」而不是独立注册的命令，但 Claude Code 按路径枚举时会把它们识别为命令却无法注册 → 自查：我的仓库有没有 `commands/` 目录下的非命令文件？
2. **6 个命令完全缺 `allowed-tools`**（commit、create-pr、documentation-generator、bug-fix、pr-review、agent-sdk-dev/new-sdk-app） → 根本原因：命令体内描述了 git / gh 操作，但作者没有在 frontmatter 中声明工具权限，导致命令「描述能做 A，但实际没有权限做 A」→ 自查：我写命令时有没有先想清楚「这个命令需要哪些工具」
3. **`test-writer-fixer` agent 完全没有 `tools` 字段** → 根本原因：frontmatter 写了 `color: cyan` 却漏了 `tools`，说明作者当时不清楚 agent 和 command 在 frontmatter 格式上的差异 → 自查：agent 的 frontmatter 字段：`name` / `description` / `model` / `tools`，缺少 `tools` 意味着 agent 的工具预算未定义

### 3.4 当时的优化机会（仅学习用途）

1. 给 `debugger`、`frontend-developer`、`backend-architect`、`test-writer-fixer` 这 4 个 agent 补充 `model` 声明（当时全缺）
2. 给 `perf` 系列的 5 个 agent 各补 1 个示例 block —— 即使是「输入：运行 `npm test`，输出：列出 3 个瓶颈」这样的简例
3. 给 `ship.md` 的 `--strategy` 参数加 allowlist 校验（`squash|merge|rebase`），防止参数注入

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `ship-deployment.md` 无 frontmatter | `head -5 ship/commands/ship-deployment.md` | **仍存在**：文件以 `# Phases 7-10:` 开头，无 frontmatter | 作者把它定位为 `ship.md` 的附件，选择不修复 |
| `commit/commands/commit.md` 缺 `allowed-tools` | `grep allowed-tools commit/commands/commit.md` | **仍存在**：无 `allowed-tools` 字段 | commit 命令实际仍无法进行 git 操作 |
| `debugger` agent 无 `model` 声明 | `head -10 debugger/agents/debugger.md` | **已修复**：现 frontmatter 含 `name/description/tools`；但仍无 `model` 字段 → 依赖平台默认值 | 部分修复：tools 补全了，model 仍未声明 |

### 4.2 架构演进

从 2026-04-06 到当前 HEAD，主要变化：
- `debugger/agents/debugger.md` 获得了完整的 frontmatter（name、description、tools），但 model 仍缺失
- 整体目录结构稳定，未见重大重组
- `ship-deployment.md` 等引用文档作者有意保持无 frontmatter，说明这是设计选择而非遗漏

### 4.3 新增的可学习模式

当前 HEAD 未见 audit 日期后的重大新增模式。值得注意的是 `debugger` 的修复路径：先补 tools（权限修复）再考虑 model（质量优化），这个优先顺序是对的。

---

## 五、校准

### 5.1 我已经在做对的

1. **`allowed-tools` 声明**：echo-sleuth 的 4 个命令均有明确的 `allowed-tools` 字段，避免了「写了做不到」的问题
2. **技能专注单一职责**：echo-sleuth 的 git-mining/memory-management/experience-synthesis 各自聚焦一个能力，与 `perf` 套件的单职责哲学一致
3. **避免模糊量词**：echo-sleuth 命令的模糊量词计数较低（grep 结果 18 处，多在注释类内容中）

### 5.2 挑战 / 验证

- **验证**：本案例验证了「描述能力 ≠ 声明权限」这个原则。`commit.md` 写了很多 git 操作描述，但没有 `allowed-tools: Bash(git:*)`，命令实际上无法执行。这提醒我：写完命令后要做一个「工具权限核查」步骤

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令是否有 allowed-tools 声明
find ~/.claude/commands /tmp/my-repos -name "*.md" -path "*/commands/*" | \
  xargs grep -L "allowed-tools" 2>/dev/null

# 命中后：为每个命中文件的 frontmatter 补上 allowed-tools 字段（哪怕是 allowed-tools: []）
```

```bash
# 检查我的 plugin.json 是否都有 version 字段
find /tmp/my-repos -name "plugin.json" | xargs grep -L '"version"' 2>/dev/null

# 命中后：在 plugin.json 中加 "version": "0.1.0"
```

```bash
# 检查我的 agent 是否都声明了 model
find /tmp/my-repos -name "*.md" -path "*/agents/*" | xargs grep -L "^model:" 2>/dev/null

# 命中后：在 frontmatter 中加 model: sonnet（或根据任务复杂度选 haiku）
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth-for-claude 的每个 agent 补 1 个示例 block  
   **为何可行**：当前 5 个 agent 都缺示例，正是 ComposioHQ `perf` 系列 agent 的同类问题  
   **第一步**：打开 `agents/memory-auditor.md`，在文件末尾加 `## Example` 段，写一个「输入：什么触发了你 → 输出：你返回什么格式」的 3 行示例 → 预计 10 分钟

2. **想法**：在 drama-workshop-skills 的 release-manifest.json 旁加一个 CI 脚本，检查所有 NL 工件的 frontmatter 完整性  
   **为何可行**：ComposioHQ 的 9 个 plugin.json 缺 version 的问题就是靠人工审查发现的，CI 自动化可以防止这类遗漏  
   **第一步**：在 `release-gate/` 目录下写一个 `check-frontmatter.sh`，用 grep 检查每个 SKILL.md 是否有 `name:` / `description:` → 预计 20 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 ComposioHQ/awesome-claude-plugins 的核心目的**：聚合多作者贡献的 Claude Code 插件，形成可供社区安装的工具集合

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为 Claude Code skills 集合 | drama-workshop 是单作者、聚焦单一领域（短剧）；ComposioHQ 是多作者、全功能覆盖 | 中 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code 插件（含 commands + agents + skills） | echo-sleuth 是单一插件，ComposioHQ 是 25 个插件的集合 | 高 |
| MarkQWu/claude-for-legal | 低 | 同为多子模块结构 | claude-for-legal 是按法律领域分区，而非按插件功能分区 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 无 `model` 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/` | **命中 5/5**：echo-sleuth 的 5 个 agent 均无 model 字段 | 高 |
| agent 无示例 block | `find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents -name "*.md" -exec grep -L "## Example" {} \;` | **命中 5/5**：全部 5 个 agent 无示例 | 高 |
| 命令无 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | 未命中：echo-sleuth 4 个命令均有 allowed-tools | 无 |

**具体行动建议**：
- echo-sleuth 的 `agents/memory-auditor.md` → 在 frontmatter 中加 `model: sonnet`，并在文件末加 `## Example` 段 → 15 分钟可完成
- echo-sleuth 的其余 4 个 agent 同理 → 每个约 10 分钟

### 8.3 别人的更优方案

1. **领域**：Skill 与 Command 的明确分工  
   - **本案例做法**：`skill-bus` 插件每个用户功能对应一对「命令（用户入口）+ 技能（可复用知识）」，`add-sub.md` 命令调用 `add-sub/SKILL.md` 技能，职责清晰  
   - **我的项目现状**：echo-sleuth 的 `commands/extract.md` 同时包含触发逻辑和实现细节，未抽取为独立 SKILL.md  
   - **如何借鉴**：把 `commands/extract.md` 的核心逻辑抽取为 `skills/extract-conversations/SKILL.md`，命令保留触发条件和输出格式

2. **领域**：perf 系列的「指挥 + 专家」agent 分层  
   - **本案例做法**：`perf-orchestrator` 只负责任务分配，每个 `perf-theory-gatherer`、`perf-analyzer` 等只做一件专业任务  
   - **我的项目现状**：echo-sleuth `agents/memory-auditor.md` 兼顾了发现、分析、写入三个职责  
   - **如何借鉴**：拆分 `memory-auditor` 为 `memory-scanner`（发现）和 `memory-writer`（写入），让每个 agent 更聚焦

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 声明完整度  
  - **我的做法**：echo-sleuth 的所有 4 个命令均有 `allowed-tools` 声明，且与命令体中实际用到的工具一致  
  - **本案例做法（弱在哪）**：6 个命令完全缺 `allowed-tools`，时至今日 commit 和 create-pr 仍未修复  
  - **意义**：我在这一点上避免了「写了描述但实际跑不了」的坑，这是被审计时的亮点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读取命令文件时先解析 frontmatter 才能知道这个命令怎么注册和调用。如果缺少 frontmatter，文件即使存在也无法被正确识别和注册。

### <a name="plugin.json"></a>plugin.json（manifest）
> 插件的「目录清单」文件，告诉 Claude Code 这个插件包含哪些组件（命令、技能、agent）以及各自的文件路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。`version` 字段是 manifest 的必填项，用于版本兼容管理。

### <a name="allowed-tools"></a>allowed-tools
> command 文件 frontmatter 中的一个字段，声明该命令在执行时有权调用哪些工具（如 `Bash`、`Read`、`Edit`、`Glob`）。如果命令体内的步骤需要运行 git 命令，但 `allowed-tools` 里没有 `Bash(git:*)`，Claude 执行时会因为权限不足而失败。这是「声明式权限」而非「按需申请权限」的设计。

### <a name="monorepo"></a>monorepo
> 把多个独立项目（本案例中是 25 个插件）放在同一个 git 仓库中管理的方式。优点：易于发现和贡献；缺点：各项目的版本、质量、发布节奏难以独立管理。
