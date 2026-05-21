# AgriciDaniel/claude-obsidian — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-obsidian
**Stars**：976 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-21（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `security-gate`, `vague-quantifier`, `template-design`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-obsidian 是一个将 Claude Code 深度嵌入 Obsidian 知识库的 AI 编程插件，灵感来自 Andrej Karpathy 提出的"LLM Wiki"模式。当前版本 1.6.0，拥有 976 Stars，是 GitHub 上最受关注的 Claude Code 插件之一。

仓库的核心主张只有一句话：**wiki IS the project，Claude 是园丁而非作者**。这意味着 Obsidian vault 的真实内容（Markdown 笔记、模板、链接网络）与 Claude 插件定义（commands、skills、agents）共存于同一个 Git 仓库。Claude 每次会话结束后，会把有价值的回答归档回 wiki，实现知识的复利累积。

当前仓库目录结构：

```
.claude-plugin/     .cursor/rules/      .windsurf/rules/
.obsidian/          .raw/               agents/
bin/                commands/           skills/
_templates/         CLAUDE.md           GEMINI.md
AGENTS.md           Makefile
```

### 1.2 架构剖析

**命令层（4 个）**：autoresearch.md、canvas.md、save.md、wiki.md，覆盖了"研究→生成→保存→查询"的完整知识流转链路。

**技能层（11 个）**：wiki-query、wiki-lint、wiki-ingest、autoresearch、save、canvas、obsidian-bases、obsidian-markdown、defuddle 等，每个技能负责一个明确的知识操作语义。

**代理层（2 个）**：wiki-ingest（批量摄取外部内容到 wiki）、wiki-lint（检查内部链接一致性）。两个代理的职责正交，互不干涉。

**核心机制**：`wiki/hot.md` — 一个语义缓存文件，记录最近若干会话的活动摘要。`hooks/hooks.json` 里的 SessionStart hook 在每次会话开始时把 hot.md 注入 Claude 的上下文，实现跨会话的"记忆热启动"。

### 1.3 设计思路 / 方法论

该仓库体现了三层递进的设计哲学：

1. **持久化知识积累**：每次会话产出的好答案不丢失，通过 save 命令归档为 wiki 页面。
2. **语义缓存热启动**：hot.md 让 Claude 在新会话伊始就能感知上下文，而无需用户反复交代背景。
3. **DragonScale Memory**：层级化日志折叠 + 确定性页面地址 + boundary-first 自动研究策略，实现结构化的长期记忆管理。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：Wiki-as-Truth 知识复利架构

核心特征：NL 插件与 Obsidian vault 共享同一个 Git 仓库，Claude 的每次有价值输出都通过 save 命令归档到 wiki，形成可被下次会话读取的知识沉淀。

### 2.2 适用场景

- 长期研究型项目（文献管理、技术调研、竞品分析）
- 需要跨会话记忆积累的个人知识管理
- 任何希望"AI 越用越聪明"的场景（因为 wiki 内容会反哺 Claude 的上下文）

### 2.3 与其他架构对比

| 维度 | claude-obsidian | 普通 Claude Code 插件 |
|------|-----------------|----------------------|
| 知识持久性 | wiki 归档，Git 历史可追溯 | 无，会话结束即消散 |
| 跨会话记忆 | hot.md 注入上下文 | 靠用户手动复述 |
| AI 工具兼容性 | CLAUDE.md + GEMINI.md + AGENTS.md | 仅 CLAUDE.md |
| IDE 兼容性 | .cursor/rules/ + .windsurf/rules/ | 仅 Claude Code |

跨 IDE 规则镜像（cross-IDE rule mirroring）是 1.6.0 引入的架构亮点：同一套 NL 规范被同步写入 .cursor/rules/、.windsurf/rules/、CLAUDE.md、GEMINI.md、AGENTS.md，实现了"IDE-agnostic NL 规范"。这是目前主流插件中少见的设计。

### 2.4 改进空间

- **安全隔离**：hot.md 缓存机制设计精巧，但与 Claude 的提示上下文直连，是典型的间接提示注入（indirect prompt injection）入口（详见第三节）。
- **工具声明完整性**：allowed-tools 字段有多处遗漏，导致技能声明的行为无法实际执行。
- **命令注册**：4 个命令全部缺少 name 字段，在 Claude Code 中无法正常注册为斜杠命令。

---

## 三、过去审查发现（历史快照）

### 3.1 当时质量评分（NLPM）

**总体 NL 分：91/100，安全状态：BLOCKED（不得贡献 PR）**

| 文件 | 得分 | 主要扣分原因 |
|------|------|------------|
| commands/autoresearch.md | 60 | 缺 name 字段 -25，无 allowed-tools -5，步骤未编号 -10 |
| commands/save.md | 60 | 同上 |
| commands/canvas.md | 70 | 缺 name 字段 -25，无 allowed-tools -5 |
| commands/wiki.md | 70 | 同上 |
| skills/wiki-query/SKILL.md | 85 | Write 未列入 allowed-tools |
| agents/wiki-lint.md | 97 | 声明了 Bash 但实际未使用 |
| hooks/hooks.json | 90 | .raw/ 自动提交，Stop hook 通过 echo 注入指令 |

安全发现共 8 条：2 Critical、1 High、4 Medium、1 Low。

### 3.2 当时值得借鉴的模式

- **单一职责技能**：11 个技能每个只做一件事（查询、摄取、归档、格式化），边界清晰。
- **双代理正交设计**：wiki-ingest 和 wiki-lint 职责完全不重叠，可独立演化。
- **plugin.json 满分**：元数据完整，版本语义清晰。
- **CLAUDE.md 满分**：项目背景、使用说明、边界定义都到位。

### 3.3 当时的缺陷

**Bug 级（功能性失效）：**

1. `commands/autoresearch.md`：frontmatter 无 `name:` 字段，无法在 Claude Code 中注册为 `/autoresearch`。
2. `commands/canvas.md`：同上。
3. `commands/save.md`：同上。
4. `commands/wiki.md`：同上。
5. `skills/wiki-query/SKILL.md`：技能说明要求"把答案写回 wiki 页面"，但 `allowed-tools: Read Glob Grep` 里没有 `Write`，写入操作会被权限拦截，静默失败。

**安全漏洞（Critical 级）：**

**Critical #1**：`hooks/hooks.json` 第 46 行，Stop hook 通过 `echo` 注入多句自然语言指令：
```
echo 'WIKI_CHANGED: Wiki pages were modified this session. Please update wiki/hot.md with a brief summary...'
```
这条 echo 输出会被当作 tool output 注入 Claude 的当前上下文，相当于攻击者可以通过影响 hook 脚本来给 Claude 发指令。这是经典的间接提示注入向量。

**Critical #2**：`hooks/hooks.json` 第 9 行，SessionStart hook：
```json
"command": "[ -f wiki/hot.md ] && cat wiki/hot.md || true"
```
把 `wiki/hot.md` 的原始内容注入 Claude 的上下文。如果 hot.md 曾经被恶意内容写入（例如通过 Critical #1 的提示注入），下一次会话开始时这些内容会被悄无声息地重新注入，形成**持久化提示注入**闭环。

**High 级**：`skills/canvas/SKILL.md` 第 79 行，`python3 -c "...Image.open('[path]')..."` 中的 `[path]` 来自用户输入，若输入值为 `'); import os; os.system('...')` 则会执行任意代码。

### 3.4 当时的优化机会

- 5 个技能文件含有模糊量词（vague quantifier）："major contradictions"、"significant cross-references"、"most valuable content"、"significant query exchange"——这类词让 Claude 的行为不可预测，应替换为可量化的标准（如"超过 3 处矛盾"）。
- `skills/wiki/SKILL.md` 在强制输出中嵌入了宣传性页脚（"Join the AI Marketing Hub"），与知识管理工具的定位不符。
- commands 里引用了 `skills/wiki/references/plugins.md` 等不存在的文件，形成悬空引用。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

截至 2026-05-21 HEAD（v1.6.0），对照 2026-04-20 审查结论：

| 缺陷编号 | 描述 | 当前状态 |
|---------|------|---------|
| Bug 1-4 | 4 个命令缺 name 字段 | **未修复** — 所有 4 个命令 frontmatter 仍无 `name:` |
| Bug 5 | wiki-query Write 缺失 | **未修复** — `allowed-tools: Read Glob Grep`，Write 仍缺席 |
| Critical #1 | Stop hook echo 注入 | **未修复** — hooks/hooks.json 第 46 行原样保留 |
| Critical #2 | SessionStart cat 注入 | **未修复** — hooks/hooks.json 第 9 行原样保留 |

**结论**：4 个月过去，所有 Bug 和两个 Critical 安全漏洞均未修复。仓库的迭代重心在于功能扩展（跨 IDE 支持、新技能），而非已知问题的收敛。

### 4.2 架构演进

v1.6.0 相较审查时期的主要变化：

- **多 AI 支持**：新增 GEMINI.md、AGENTS.md，与 CLAUDE.md 并列，面向 Gemini 和通用 agentic 框架。
- **跨 IDE 规则镜像**：新增 .cursor/rules/、.windsurf/rules/ 目录，将 NL 规范推广到 Cursor 和 Windsurf IDE。
- **bin/ 目录**：新增 `setup-dragonscale.sh`、`setup-multi-agent.sh`，为复杂设置提供脚本化入口。
- **Makefile**：作为新用户的 onboarding 入口，这在 NL 插件生态中较为罕见。
- **技能扩充**：从 7 个增至 11 个（新增 obsidian-bases、obsidian-markdown、defuddle、canvas 等）。

### 4.3 新增的可学习模式

**模式 A：跨 IDE NL 规范镜像**
同一套约束同时维护在 .cursor/rules/、.windsurf/rules/、CLAUDE.md、GEMINI.md，实现"换工具不换规范"。缺点是多份文件需手动同步，存在规范漂移风险。

**模式 B：Makefile 作为 onboarding 入口**
对于需要外部依赖（Obsidian 插件、Python 库）的 NL 插件，Makefile 提供了比 README 更可执行的引导路径。

**模式 C：DragonScale Memory 层级折叠**
层级化日志 + 确定性页面地址 + boundary-first 自动研究，为长期知识积累提供了可操作的结构规范。这是 hot.md 语义缓存机制的高阶演化版本。

---

## 五、校准

### 5.1 我已经在做对的

- **单一职责技能**：我的技能文件也遵循"每个技能只做一件事"的原则，这一点与 claude-obsidian 的优秀实践一致。
- **示例驱动的命令**：用示例锚定命令行为，而非靠自然语言描述产生歧义。
- **CLAUDE.md 项目上下文**：在项目根目录维护完整的背景说明，让 Claude 每次都有稳定的起点。

### 5.2 挑战 / 验证

**挑战一："wiki IS the repo"这种设计模式我从未见过。**

通常 NL 插件和知识库是分开的——插件定义放在 .claude/ 或插件目录，内容放在另一个仓库。claude-obsidian 把两者合并为一个 Git 仓库，优势是知识与工具定义共同演化、版本历史一致；劣势是 Git diff 里会混入大量 Obsidian vault 的内容变更，噪声较大。

**验证动作**：下次设计个人知识库项目时，尝试把 CLAUDE.md + commands/ + skills/ 与实际知识内容放在同一个仓库，观察使用体验的差异。

**挑战二：hot.md 语义缓存是个好主意，但它也是最大的安全漏洞。**

hot.md 注入机制的安全模型假设：写入 hot.md 的内容是可信的。但 Critical #1（Stop hook echo 注入）已经破坏了这个假设——恶意 hook 输出可以写入 hot.md，而 Critical #2 确保这些内容会在下次会话时被重新注入，形成持久化攻击链。

**验证动作**：如果我要实现类似的"会话记忆热启动"机制，应当：(a) 不用 `cat` 直接注入文件，改用结构化的 JSON sidecar + 固定格式解析；(b) Stop hook 只允许写入明确格式的数据，不允许注入自然语言指令。

---

## 六、行动

### 6.1 自查动作

1. **检查所有命令文件**：`grep -r "^name:" commands/` — 确保每个命令都有 `name:` frontmatter 字段。
2. **核对 allowed-tools 完整性**：逐一比对技能正文中提及的工具操作与 `allowed-tools:` 声明，Write/Edit/Bash 缺一不可。
3. **审查 hooks.json 的输出内容**：所有 hook 的 `echo`/`printf` 输出是否包含自然语言指令？若有，改为结构化 key=value 格式。
4. **扫描模糊量词**：`grep -r "major\|significant\|most valuable\|substantial" skills/` — 用可量化标准替换每一个模糊词。
5. **验证技能引用路径**：commands 里的 `See also:` 链接是否都指向实际存在的文件？

### 6.2 灵感 → 实施路径

**灵感**：hot.md 语义缓存机制（跨会话记忆热启动）

**安全实施路径**：
1. 在项目根目录创建 `session-cache.json`（结构化，不是自然语言 Markdown）。
2. Stop hook 只做一件事：把本次会话产生的关键事实写入 `session-cache.json`，格式固定（如 `{"last_file": "...", "last_topic": "..."}`）。
3. SessionStart hook 读取 `session-cache.json`，用固定模板生成一行上下文注入（如 `[CACHE] Last session: edited X, worked on Y`），而非直接 `cat` 整个文件。
4. 设置 `session-cache.json` 的 schema 验证，拒绝任何包含 CLAUDE.md 关键词（`SYSTEM:`、`Please`、`You are` 等）的值。

**灵感**：跨 IDE NL 规范镜像

**实施路径**：用一个单一的 `rules/base.md` 作为 source of truth，通过 Makefile target 或 pre-commit hook 自动同步到 CLAUDE.md、.cursor/rules/、.windsurf/rules/，避免多处手动维护的规范漂移。

---

## 七、术语表

| 术语 | 解释 |
|------|------|
| **frontmatter** | Markdown 文件顶部以 `---` 包围的 YAML 元数据块，在 Claude Code 插件中用于声明命令/技能的 `name`、`allowed-tools`、`description` 等注册信息 |
| **allowed-tools** | 技能或命令 frontmatter 中声明 Claude 可使用的工具白名单（如 `Read Glob Grep Write Bash`）。未列入的工具调用会被权限系统拦截，可能导致功能静默失败 |
| **SessionStart hook** | `hooks/hooks.json` 中配置的会话开始时自动执行的 shell 命令。claude-obsidian 用它来 `cat wiki/hot.md`，把热缓存文件内容注入 Claude 的初始上下文 |
| **间接提示注入（indirect prompt injection）** | 攻击者通过影响 Claude 会读取的外部数据（文件、网页、hook 输出）来注入恶意指令，而非直接对话。与直接提示注入不同，攻击路径经过中间介质，更难被用户察觉 |
| **hot.md 语义缓存** | claude-obsidian 独创的跨会话记忆机制：Stop hook 在每次会话结束时更新 `wiki/hot.md`，写入本次会话的活动摘要；SessionStart hook 在下次会话开始时注入该文件，让 Claude 感知历史上下文 |
| **DragonScale Memory** | claude-obsidian 1.6.0 引入的层级化记忆管理策略：通过日志折叠、确定性页面地址和 boundary-first 自动研究，为长期知识积累提供结构化框架 |
| **vague quantifier（模糊量词）** | 技能或命令中使用的无法量化的形容词或副词，如"major"、"significant"、"most valuable"。NLPM R 规则要求替换为可测量的标准，因为模糊量词会导致 Claude 行为不确定，违背 NL 规范的可重复性原则 |
| **wiki IS the repo** | claude-obsidian 的核心设计哲学：Obsidian vault 的真实内容与 Claude 插件定义共存于同一个 Git 仓库，Claude 的角色是"知识园丁"，把每次会话的有价值产出归档回 wiki，形成知识复利 |
