# mksglu/context-mode — 学习案例

**仓库**：https://github.com/mksglu/context-mode
**Stars**：9822 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-21（基于当前 HEAD）
**主题标签**：security-gate, curl-pipe-bash-risk, template-design, cross-reference, offline-capable

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

context-mode 是一个 Claude Code 插件，核心功能是**跨会话上下文追踪**：在本地 SQLite 数据库中存储每次工具调用的内容，供后续会话检索复用。9822 stars，是上下文管理类插件中星数最高的之一。

关键事实：
1. **多平台支持**：不止 Claude Code，还覆盖 Cursor、Codex、Gemini CLI、VSCode Copilot、JetBrains Copilot、Kiro 等 6 个 AI 编程工具
2. **当前技能列表（共 8 个）**：context-mode、ctx-doctor、ctx-index、ctx-search、ctx-insight、ctx-purge、ctx-stats、ctx-upgrade
3. **Audit 时（2026-04-06）技能列表（共 9 个）**：上述 8 个中原有 context-mode-ops，无 ctx-index、ctx-search——架构经历了一次重组
4. **原生模块依赖**：使用 `better-sqlite3`（需要编译 native addon），通过 `ensure-deps.mjs` 自动安装
5. **Hooks 驱动**：pretooluse、posttooluse、sessionstart、precompact、userpromptsubmit 全覆盖，是 hooks 使用最密集的插件之一

```
context-mode/
├── .claude-plugin/
│   └── plugin.json              # 插件清单（90/100）
├── hooks/
│   └── hooks.json               # PostToolUse 全工具捕获（85/100）
├── scripts/
│   ├── ensure-deps.mjs          # ← HIGH 安全风险：execSync + 字符串插值
│   ├── pretooluse.mjs           # ← MEDIUM：静默写入 ~/.claude/settings.json
│   ├── posttooluse.mjs          # ← MEDIUM：全工具调用内容存入 SQLite，无隐私披露
│   └── ctx-debug.sh             # ← MEDIUM+LOW：shell 变量插入 node -e JS 字符串
├── CLAUDE.md                    # 根目录主 CLAUDE.md（88/100）
├── configs/
│   └── claude-code/
│       └── CLAUDE.md            # 平台特化配置（90/100）⚠ 与根 CLAUDE.md 内容重复
├── skills/
│   ├── context-mode/
│   │   └── SKILL.md             # 88/100，引用 ./references/ 下 4 个文件
│   ├── context-mode-ops/        # ⚠ 当前 HEAD 已不存在（Audit 时 73/100）
│   ├── ctx-doctor/SKILL.md      # 90/100
│   ├── ctx-index/SKILL.md       # 新增（audit 后）
│   ├── ctx-insight/SKILL.md     # 88/100
│   ├── ctx-purge/SKILL.md       # 90/100
│   ├── ctx-search/SKILL.md      # 新增（audit 后）
│   ├── ctx-stats/SKILL.md       # 90/100
│   └── ctx-upgrade/SKILL.md     # 88/100
├── package.json                 # postinstall 自动执行 scripts/postinstall.mjs
└── .mcp.json                    # MCP 配置
```

### 1.2 架构剖析

**上下文追踪机制**

posttooluse.mjs 作为 PostToolUse hook 响应每一次工具调用，将工具名称、参数、结果写入本地 SQLite 数据库。SessionStart hook 在新会话开始时加载历史上下文，让 Claude 在不依赖 `/compact` 或外部存储的情况下感知跨会话状态。这是一种**纯本地、离线可用**的上下文持久化方案。

**依赖管理策略**

`better-sqlite3` 是 C++ 原生模块，每次 npm 安装环境不同都需要重新编译。插件通过以下路径自动处理：
1. `package.json` 的 `postinstall` 字段触发 `scripts/postinstall.mjs`
2. 运行时 hook 首次导入时，`ensure-deps.mjs` 检测依赖是否存在，若无则调用 `execSync('npm install ...')`

这个设计解决了原生模块跨平台编译的痛点，但也带来了严重安全隐患（见第三节）。

**多平台适配**

`configs/` 目录下按平台分别维护 CLAUDE.md，核心原则不变但工具调用语法各异。根目录 CLAUDE.md 作为"Think in Code"通用规范，各平台 CLAUDE.md 继承并扩展。

### 1.3 设计思路 / 方法论

1. **"Think in Code"原则**：CLAUDE.md 和 configs/claude-code/CLAUDE.md 共同强调"用代码思考，而非自然语言"——在响应前先写伪代码或结构图，减少幻觉
2. **Tool Selection 列表**：两份 CLAUDE.md 均含有相同的工具选择优先级列表（Read before Edit, Grep before Read 等），是实践中沉淀的 AI 操作规范
3. **技能细分**：每个技能只做一件事（ctx-purge 清理、ctx-stats 统计、ctx-upgrade 迁移），符合单一职责原则
4. **诊断友好**：ctx-doctor 和 ctx-debug.sh 专门用于调试上下文数据库状态，说明作者重视可观测性

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名 | 描述 | context-mode 的实现 |
|--------|------|---------------------|
| **跨会话上下文持久化** | 将工具调用记录存储在本地，供后续会话检索 | SQLite + posttooluse hook |
| **运行时依赖自举** | 插件首次运行时自动安装所需依赖 | ensure-deps.mjs + postinstall |
| **多平台 CLAUDE.md** | 针对不同 AI 编程工具维护特化配置 | configs/ 目录层级 |
| **细粒度技能切分** | 每个操作（清理/统计/升级）对应独立 SKILL.md | ctx-purge / ctx-stats / ctx-upgrade |
| **全钩子覆盖** | 5 种 hook 事件全部利用，形成完整生命周期感知 | hooks.json |

### 2.2 适用场景

- **长期项目维护**：需要跨会话记忆决策背景、已试方案、放弃原因
- **团队共享 AI 上下文**：多人协作时同步 AI 操作历史（需共享 SQLite 文件）
- **调试复杂问题**：ctx-search 检索历史工具调用，快速定位之前的操作路径
- **插件开发参考**：全钩子覆盖 + 多平台适配是写"重型插件"的模板

### 2.3 与其他架构对比

**对比 manaflow-ai/cmux**（同为高星插件）：cmux 是 Swift 原生应用，NL 工件只是开发辅助；context-mode 反过来，NL 工件（hooks + skills）是核心产品本身，二进制层极薄（仅 sqlite）。

**对比 echo-sleuth-for-claude**（MarkQWu 的会话挖掘插件）：两者都做会话数据分析，但 echo-sleuth 是**事后分析**（读取已有 JSONL 日志），context-mode 是**实时捕获**（PostToolUse 写入 SQLite）。echo-sleuth 无钩子，无安全风险；context-mode 全钩子，安全面更大。

**对比 drama-workshop-skills**：drama-workshop-skills 是纯技能集合（无钩子），context-mode 是重度钩子驱动——两者代表了 Claude Code 插件设计的两个极端。

### 2.4 改进空间

1. **隐私透明度**：PostToolUse 捕获全部工具调用内容（含文件内容、命令输出），用户无感知，应在首次运行时提示并获取同意
2. **配置去重**：根 CLAUDE.md 与 configs/claude-code/CLAUDE.md 的"Think in Code"块和 Tool Selection 列表完全相同，应提取为共享模板或通过 `@include` 引用
3. **依赖锁定**：`package.json` 中 `better-sqlite3` 使用 `^semver`（如 `^9.4.3`），patch/minor 更新可能破坏原生编译；应锁定到精确版本
4. **外部 npm 请求**：`ctx-debug.sh` 中有向 npm registry 发送 HTTPS 请求的逻辑，无用户确认

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**整体评分：87/100**（安全状态：BLOCKED，无法进入贡献流程）

| 文件 | 得分 | 主要扣分原因 |
|------|------|-------------|
| skills/context-mode-ops/SKILL.md | 73 | 无示例（−15），8 个同级文件引用均无效 |
| hooks/hooks.json | 85 | PostToolUse 广度未文档化（隐私风险） |
| CLAUDE.md | 88 | — |
| skills/context-mode/SKILL.md | 88 | ./references/ 下 4 个引用文件不在 audit 范围 |
| skills/ctx-insight/SKILL.md | 88 | — |
| skills/ctx-upgrade/SKILL.md | 88 | — |
| .claude-plugin/plugin.json | 90 | — |
| configs/claude-code/CLAUDE.md | 90 | — |
| skills/ctx-doctor/SKILL.md | 90 | — |
| skills/ctx-purge/SKILL.md | 90 | — |
| skills/ctx-stats/SKILL.md | 90 | — |

**安全发现（共 11 项）**：

- **HIGH（2项）**
  - `scripts/ensure-deps.mjs`：`execSync` 使用 `shell: true` 且字符串插值（包名当前硬编码，但为潜在注入面）
  - `package.json` postinstall：每次 `npm install` 自动执行 `scripts/postinstall.mjs`（供应链攻击面）

- **MEDIUM（5项）**
  - 运行时 hook 首次导入触发 `npm install`，用户无感知
  - `pretooluse.mjs` 静默写入 `~/.claude/settings.json` 和 `~/.claude/plugins/installed_plugins.json`
  - PostToolUse 捕获全工具调用内容存入 SQLite，无隐私披露
  - `ctx-debug.sh`：shell 变量插值进入 `node -e` JS 字符串
  - `ctx-debug.sh`：向 npm registry 发送 HTTPS 请求，无用户确认

- **LOW（4项）**
  - `ctx-debug.sh` 永久导出 `NODE_PATH`
  - `eval` 用于间接展开
  - codesign 路径在 shell 字符串中插值
  - `better-sqlite3` 等依赖使用非固定 `^semver`

### 3.2 当时值得借鉴的模式

1. **细粒度技能命名**：ctx-purge、ctx-stats、ctx-upgrade 每个只做一件事，命名即职责
2. **诊断技能配套**：ctx-doctor 专门诊断插件自身状态，可观测性意识强
3. **多平台支持结构**：`configs/` 目录层级清晰，每平台独立 CLAUDE.md
4. **高分技能的写法**：ctx-doctor（90）、ctx-purge（90）等技能格式规范，无遗漏字段

### 3.3 当时的缺陷

**Bug 1（无效跨引用）**：`skills/context-mode-ops/SKILL.md` 引用 8 个同级文件：
- tdd.md、validation.md、agent-teams.md、communication.md
- marketing.md、review-pr.md、triage-issue.md、release.md

这 8 个文件在 audit 时全部不存在，导致该技能得 73/100（同时叠加"无示例 −15"惩罚）。

**Bug 2（引用目录缺失）**：`skills/context-mode/SKILL.md` 引用 `./references/` 目录下：
- `patterns-javascript.md`、`patterns-python.md`、`patterns-shell.md`、`anti-patterns.md`

Audit 时这 4 个文件均不在 audit 范围内，无法验证引用有效性。

**质量问题 1**：`CLAUDE.md`（根目录）与 `configs/claude-code/CLAUDE.md` 存在内容重复——"Think in Code"模块和 Tool Selection 列表完全相同，两个文件均为 91 行，是机械复制而非引用。

**质量问题 2**：`hooks.json` 的 PostToolUse matcher 匹配全部工具调用，但未在任何文档中说明数据采集范围，构成隐私风险。

### 3.4 当时的优化机会

1. 补全 `context-mode-ops` 的 8 个伴随文件，或改为内联内容
2. 确认并提交 `skills/context-mode/references/` 目录下的 4 个文件
3. 将两份 CLAUDE.md 的公共块提取为共享模板
4. 在 hooks.json 的 matcher 说明或 CLAUDE.md 中披露数据采集范围
5. 将 `ensure-deps.mjs` 中的 `execSync` 替换为不使用 `shell: true` 的形式

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | Audit（2026-04-06）| 当前 HEAD（2026-06-21）| 状态 |
|------|-------------------|----------------------|------|
| context-mode-ops 引用 8 个不存在文件 | 存在，得 73/100 | **整个 context-mode-ops 目录已删除** | 以删代修 |
| context-mode/references/ 目录缺失 | 4 个文件不在 audit 范围 | **references/ 目录现已存在，4 个文件全部就位** | 已修复 |
| CLAUDE.md 与 configs/claude-code/CLAUDE.md 内容重复 | 重复（两文件均 91 行） | **依然重复，两文件仍为 91 行** | 未修复 |
| HIGH 安全：ensure-deps.mjs execSync + shell=true | 存在 | 未验证是否修复（安全 gate 阻断贡献流程） | 未知 |

### 4.2 架构演进

**技能集重组**：`context-mode-ops`（综合操作技能，得分最低）被删除，同时新增了功能更聚焦的 `ctx-index` 和 `ctx-search`。这是一次**"以拆代补"**的重构策略——与其修复引用无效、无示例的综合技能，不如删除并用专项技能替代。

| 时间点 | 技能数量 | 变化 |
|--------|---------|------|
| Audit（2026-04-06） | 9 个 | 含 context-mode-ops |
| 当前 HEAD（2026-06-21） | 8 个 | 删除 context-mode-ops，新增 ctx-index、ctx-search |

**引用补全**：`context-mode/SKILL.md` 引用的 `./references/` 目录从 audit 时的"推测存在"变为真实存在，且 4 个文件（`anti-patterns.md`、`patterns-javascript.md`、`patterns-python.md`、`patterns-shell.md`）全部齐备。这是一次**主动补齐**，而非等待外部反馈。

### 4.3 新增的可学习模式

**ctx-index + ctx-search 组合**：这两个新技能共同构成上下文**索引与检索**层，是原有"记录 → 读取"二阶模型升级为"记录 → 索引 → 检索"三阶模型的体现。对于数据量大的长期项目，全量扫描 SQLite 不如索引检索高效，这次演进展示了对性能瓶颈的前瞻性设计。

---

## 五、校准

### 5.1 我已经在做对的

1. **无钩子设计**：echo-sleuth 和 drama-workshop-skills 均无 hooks 目录，完全规避了 context-mode 中 HIGH/MEDIUM 安全风险的根源。对于纯技能型插件，这是正确选择。
2. **单一职责技能**：echo-sleuth 的 4 个 SKILL.md（memory-management、git-mining、jsonl-core、experience-synthesis）与 context-mode 的细粒度技能哲学一致，每个文件只聚焦一个领域。
3. **agents 使用规范 frontmatter**：echo-sleuth 的 5 个 agent 文件均有正确的 YAML frontmatter，这是 context-mode 的 agent 部分（如果有的话）需要做到的。

### 5.2 挑战 / 验证

1. **跨引用有效性**：echo-sleuth 的部分 SKILL.md 是否引用了不存在的文件？类似 context-mode-ops 的 Bug 1 在我的仓库中是否也存在？需要 grep 检查所有 `@`引用和相对路径。
2. **输出格式缺失**：echo-sleuth 的 `memory-management` 和 `jsonl-core` 两个 SKILL.md 无 output format 字段——context-mode 中 73 分的 context-mode-ops 同样缺失示例（−15）。两者都有待补全。
3. **`vague:` 字段的准确性**：`git-mining` 标注为 `appropriate`，`experience-synthesis` 标注为 `relevant`——这些判断是否经过校准？context-mode 的 audit 发现模糊词处理较好，值得对比分析。

---

## 六、行动

### 6.1 自查动作

针对 echo-sleuth 和 drama-workshop-skills 的具体自查清单：

1. **检查所有 SKILL.md 中的引用链接**
   ```bash
   grep -r "\./references/" /path/to/echo-sleuth/skills/
   grep -r "\./references/" /path/to/drama-workshop-skills/skills/
   ```
   验证被引用文件是否实际存在，防止出现 context-mode 的 Bug 1 和 Bug 2。

2. **检查输出格式字段**
   grep 确认 echo-sleuth 的 `memory-management` 和 `jsonl-core` 是否缺少 `output_format:` 或"## Output"章节。

3. **CLAUDE.md 去重检查**
   如有多份 CLAUDE.md（根目录 vs 平台特化），用 `diff` 比较公共部分，提取为单一来源。

4. **package.json 安全审查**
   如未来 echo-sleuth 或其他项目添加 npm 依赖，检查是否有 `postinstall` 脚本，避免引入 context-mode 式的供应链风险。

### 6.2 灵感 → 实施路径

**灵感 A：诊断技能**

context-mode 的 ctx-doctor 是一个值得移植的模式：专门检查插件自身状态（数据库是否损坏、依赖是否完整）。echo-sleuth 当前无此类技能。

实施路径：
- 新建 `skills/echo-health/SKILL.md`
- 内容：检查 JSONL 日志文件是否存在、格式是否正确、大小是否超限
- 触发方式：用户手动调用，或在 SessionStart 时作为静默检查

**灵感 B：references/ 目录模式**

context-mode 的 `skills/context-mode/references/` 存放语言特化模式文件（patterns-javascript.md 等），避免主 SKILL.md 过长。

实施路径：
- echo-sleuth 的 `git-mining` 技能若要扩展多语言仓库支持，可将"Python 仓库 git 挖掘模式"、"Monorepo git 挖掘模式"分别放入 `skills/git-mining/references/` 子目录
- 主 SKILL.md 中通过 `See references/patterns-monorepo.md for monorepo-specific patterns` 引用

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 核心用途 | 与 context-mode 的关系 |
|----------|---------|----------------------|
| MarkQWu/echo-sleuth-for-claude | 会话事后挖掘分析 | 互补：context-mode 实时捕获，echo-sleuth 事后分析 |
| MarkQWu/drama-workshop-skills | 中文微短剧创作辅助 | 不同领域；架构对比有参考价值 |
| MarkQWu/claude-for-legal | 法律工作流辅助 | 不同领域；多技能管理策略可借鉴 |

echo-sleuth 与 context-mode 功能上最接近——都处理 Claude 会话数据——但设计哲学相反：context-mode 侵入式（hooks 实时写入），echo-sleuth 非侵入式（读取已有日志文件）。这个对比为 echo-sleuth 的定位提供了清晰的差异化叙述。

### 8.2 在我的项目里复现的同类问题

1. **输出格式缺失**（echo-sleuth）：`memory-management` 和 `jsonl-core` 两个 SKILL.md 无输出格式定义。这与 context-mode-ops（73/100，无示例）的主要扣分原因性质相同——技能告诉 AI"做什么"但不说明"输出什么格式"，导致调用结果不稳定。

2. **无诊断技能**：echo-sleuth 的 4 个技能均是功能性技能（挖掘、分析、合成），没有类似 ctx-doctor 的自检技能。当 JSONL 日志损坏或路径变更时，用户需要手动排查。

3. **无 hooks**：echo-sleuth 目前无 hooks 目录——这既是安全上的优势，也意味着无法实现实时捕获类功能。对于当前定位（事后分析），这是正确选择，但需要明确记录在 CLAUDE.md 中，防止未来开发者随意添加 hooks。

### 8.3 别人的更优方案

1. **细粒度技能命名**：context-mode 将"清理"、"统计"、"升级"拆为独立技能（ctx-purge、ctx-stats、ctx-upgrade），每个 SKILL.md 专注一个操作。echo-sleuth 的 `experience-synthesis` 承担了"分析 + 合成 + 输出报告"多个职责，可以进一步拆分。

2. **references/ 语言特化模式**：context-mode 将 JavaScript、Python、Shell 的上下文模式分别存放在 `references/` 子目录，主 SKILL.md 保持简洁。echo-sleuth 的 `git-mining` 技能若要支持多语言仓库，这是值得采用的组织方式。

3. **多平台 CLAUDE.md 层级**：context-mode 的 `configs/` 目录为多平台支持提供了清晰的结构。虽然当前 echo-sleuth 只需支持 Claude Code，但这个模式在未来扩展时可直接复用。

### 8.4 反向：我的项目做得比他们好的地方

1. **无高安全风险**：echo-sleuth 和 drama-workshop-skills 均无 hooks 目录，完全没有 context-mode 的 HIGH 安全风险（execSync + shell=true、postinstall 自动执行）。对于插件发布而言，这是重大优势。

2. **无内容重复**：echo-sleuth 和 drama-workshop-skills 均只有单一 CLAUDE.md，不存在 context-mode 的"根目录与平台特化目录内容机械复制"问题（两份 91 行文件重复至今未修复）。

3. **agent frontmatter 规范**：echo-sleuth 的 5 个 agent 文件均有完整 YAML frontmatter，符合 Claude Code 规范。context-mode 审计时未特别提及 agents，说明其 agent 管理不是设计重点；echo-sleuth 的 agent-first 架构在可维护性上更清晰。

4. **无原生依赖**：echo-sleuth 无需编译原生模块（无 `better-sqlite3` 类依赖），安装即用，不存在跨平台编译失败的风险。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **PostToolUse hook** | Claude Code 在每次工具调用完成后执行的钩子；context-mode 用它实时捕获所有工具调用内容 |
| **execSync + shell=true** | Node.js 中以 shell 模式同步执行命令；`shell: true` 使字符串插值成为命令注入向量 |
| **supply-chain surface（供应链攻击面）** | `package.json` 的 `postinstall` 自动执行脚本，若 npm 包被劫持，攻击者代码会在用户安装时自动运行 |
| **better-sqlite3** | Node.js 的 SQLite 绑定库，含 C++ 原生模块，需在目标机器上编译；跨平台兼容性依赖 node-gyp |
| **以拆代修** | 本文用于描述 context-mode 的技能重构策略：删除得分低、引用无效的综合技能，新增功能更专一的替代技能 |
| **三阶模型** | 记录 → 索引 → 检索，相对于原有的记录 → 读取（二阶）；ctx-index + ctx-search 的加入代表这一升级 |
| **非侵入式会话分析** | 不通过 hooks 实时拦截，而是事后读取已有日志文件（如 Claude Code 的 JSONL 日志）进行分析；echo-sleuth 采用此方案 |
| **fingerprint（指纹）** | NLPM 审计系统中用于跨 audit 追踪同一 finding 的唯一标识符，由文件路径 + 规则编号 + 内容哈希计算得出 |
| **finding_verified** | NLPM 事件类型：在 PR 合并后对目标仓库进行 re-audit，确认原 finding 对应的问题已从代码中消失 |
| **BLOCKED 状态** | 安全扫描发现 Critical 或（本例中）High 风险时设置，阻止 NLPM 自动贡献流程进入 fork-and-PR 阶段 |
