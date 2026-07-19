# krodak/clickup-cli — 学习案例

**仓库**：https://github.com/krodak/clickup-cli
**Stars**：57 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-19（基于当前 HEAD）
**主题标签**：`manifest-discipline, examples-driven, nl-binary-hybrid, template-design, single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`@krodak/clickup-cli`（命令行工具名 `cup`）是一个为 AI agent 和人类用户同时设计的 ClickUp 项目管理 CLI，用 TypeScript 实现，发布到 npm 并支持 Homebrew 安装。核心特点：双输出模式（TTY 交互式 / 管道 Markdown 输出）、严格 TypeScript 配置（strict + verbatimModuleSyntax）、通过 `cup skill` 命令自助刷新 AI 辅助技能文件。当前版本 1.39.0（Audit 时为 1.25.2，约 14 个大版本迭代）。

关键事实：
- 发布渠道：npm（OIDC 信任发布，无需写死 API key）+ Homebrew tap
- 用户获取方式：`npm install -g @krodak/clickup-cli` 或 `brew install clickup-cli`
- 双模式输出：终端直接运行 → @inquirer/prompts 交互式 UI（vim 风格 j/k 导航）；管道/AI 上下文 → Markdown 格式化输出
- 生态位置：Claude Code 插件体系中的"CLI 封装型 skill"范式代表，既服务人类操作员又服务 AI agent

### 1.2 架构剖析

**目录结构**：

```
krodak/clickup-cli/
├── src/                          # TypeScript 源码
│   ├── index.ts                  # CLI 入口（Commander 注册）
│   ├── api.ts                    # ClickUp API 客户端（v2 + v3）
│   ├── config.ts                 # 配置加载（~/.config/cup/config.json）
│   ├── output.ts                 # TTY 检测 + 输出模式决策
│   ├── interactive.ts            # 交互 TUI（任务选择器）
│   ├── markdown.ts               # Markdown 详情视图
│   └── commands/                 # 每个命令一个文件
├── tests/
│   ├── unit/                     # 镜像 src/ 结构
│   └── e2e/                      # 集成测试（需要 .env.test）
├── skills/
│   └── clickup-cli/
│       └── SKILL.md              # 外部公开 skill（随 npm 包发布）
├── .agents/skills/
│   ├── using-clickup-cli/
│   │   └── SKILL.md → ../../../skills/clickup-cli/SKILL.md  # 符号链接！
│   ├── releasing-clickup-cli/
│   │   └── SKILL.md              # 内部 skill（internal: true）
│   └── testing-clickup-cli/
│       └── SKILL.md              # 内部 skill（internal: true）
├── .claude-plugin/
│   └── plugin.json               # Claude Code 插件 manifest
├── AGENTS.md                     # 项目级 agent 指导文档
└── docs/
    ├── commands.md               # 完整命令参考（由脚本同步）
    └── api-coverage.md           # API 覆盖矩阵
```

**文件类型分布**：
- 4 个 SKILL.md（实际上是 3 个独立文件 + 1 个符号链接）
- 0 个 agent 定义文件（agent 通过 AGENTS.md 文档化，非结构化注册）
- 0 个 hook
- 1 个 plugin.json（manifest）
- 1 个 AGENTS.md（项目说明文档）

**编排关系**：
外部使用者通过 `skills/clickup-cli/SKILL.md` 了解 `cup` 命令集。内部开发者通过 `.agents/skills/releasing-clickup-cli/` 和 `.agents/skills/testing-clickup-cli/` 获得发布/测试的逐步指引。`using-clickup-cli/SKILL.md` 是符号链接，作为 `npx skills add` 的规范安装路径，指向同一份公开 skill——两处路径指向一份内容，消除同步负担。AGENTS.md 作为总入口，描述何时调用哪个 skill，并声明两个外部 skill 依赖（`typescript-pro`、`cli-developer`）。

**跨件契约**：
- `scripts/sync-command-docs.ts` 负责同步 `docs/commands.md` 快速引用表、`skills/clickup-cli/SKILL.md` 版本头、`.claude-plugin/plugin.json` 版本——一次执行保持三处一致
- CI 中的 version synchronization 测试捕获版本漂移
- `releasing-clickup-cli/SKILL.md` 显式引用 `.claude-plugin/plugin.json`、`skills/clickup-cli/SKILL.md` 等路径，形成文档级约束

### 1.3 设计思路 / 方法论

**核心设计哲学**：「机器可消费优先（Machine-Consumable First）」——所有文档（SKILL.md、AGENTS.md、docs/）都以 AI agent 能直接利用为首要标准，人类可读性作为次要目标。

**解决什么问题**：ClickUp REST API 繁杂、每次集成都要写重复的 HTTP 调用和错误处理；直接让 AI agent 调用 API 又绕过了人机共享的工作流。作者用 `cup` 把 API 封装成 CLI，通过输出模式分叉（TTY/管道）让同一个命令既能给人交互、也能给 agent 输出结构化文本。

**做了什么 trade-off**：
- 外部 skill vs 内部 skill：公开 skill 随 npm 包发布，任何用户都能 `cup skill` 刷新；内部 skill（releasing/testing）加 `metadata.internal: true`，防止误暴露给普通用户
- 符号链接 vs 两份文件：`using-clickup-cli` 不复制内容而是符号链接，保持单一真实来源，代价是 Git 树中需要正确处理符号链接
- TypeScript 严格模式：显著增加开发摩擦，换来的是 CI 类型门控，减少运行时错误进入用户手中

**反映什么认知模型**：作者认为 AI agent 是一等用户，不是附加需求。SKILL.md 里的 Output Modes 章节、DELETE SAFETY 块、每个命令的 Markdown 输出格式说明——这些都是站在「agent 眼中的工具」视角写的。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)，内外 skill 分层」

这个模式的关键特征：
- **特征 1**：有真实的可执行程序（`cup` 命令），NL skill 描述的是这个程序的使用方式，而非代替程序本身
- **特征 2**：skill 分两层——公开层（随包分发）+ 内部层（仅开发者可见），通过 `metadata.internal: true` 区分
- **特征 3**：符号链接实现单一真实来源——`using-clickup-cli/SKILL.md` 和 `skills/clickup-cli/SKILL.md` 是同一个文件
- **特征 4**：脚本驱动的版本同步（sync-command-docs.ts），防止 skill 文本和实际命令行为漂移
- **特征 5**：双输出模式（TTY / 管道）让同一个工具同时服务人类和 agent

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 封装第三方 SaaS API 的 CLI 工具 | ✅ 高度适用 | CLI 吸收 API 复杂度，skill 描述 CLI 接口，对 agent 友好 |
| 需要交互式操作 + 批量脚本双模式的工具 | ✅ 高度适用 | output.ts 的 TTY 检测模式是成熟方案 |
| 纯数据处理脚本（无 API 交互） | ❌ 不适用 | 程序逻辑太简单，NL 封装层是过度工程 |
| 只面向 agent 不面向人类的自动化工具 | ⚠️ 部分适用 | 不需要 TTY 分支，但 skill + manifest 架构仍然有价值 |
| 涉及破坏性操作（删除、支付）的工具 | ✅ 高度适用 | DELETE SAFETY 块模式是良好实践 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：CLI + 内外 skill 分层 | krodak/clickup-cli | 可执行程序保证计算准确性；skill 内外分层防止误暴露 | 维护 CLI 本身需要工程资源；符号链接在 Windows 上易出问题 |
| 备选 A：纯 NL skill（无 CLI） | czlonkowski/n8n-skills | 零工程投入，快速迭代 | 复杂操作（多步、分页、错误处理）完全靠 agent 自己做，不稳定 |
| 备选 B：MCP server | 各种 MCP 实现 | 标准化接口，平台无关 | 部署复杂度高；调试困难；对 ClickUp 这类有丰富 CLI 工具的 SaaS 是重复造轮子 |

### 2.4 改进空间

1. **当前问题**：[plugin.json](#manifest) 缺少 `entries`/`skills` 数组，自动化工具无法枚举插件暴露的面。**改进做法**：在 `plugin.json` 加 `"skills": ["skills/clickup-cli/SKILL.md"]`，并在 `sync-command-docs.ts` 中校验 skills 数组与实际文件的一致性。**预期收益**：NLPM 分数从 90 提升到满分；`nlpm:ls` 等工具可正确发现 skill。

2. **当前问题**：生产依赖使用 `^` 版本范围，`npm install` 时可能拉取非预期的 minor 版本。**改进做法**：在发布流程（`releasing-clickup-cli/SKILL.md` step 3 之后）加一步 `npm shrinkwrap` 生成锁文件，或将版本范围改为精确版本。**预期收益**：消除供应链不确定性，与"版本同步测试"精神一致。

3. **当前问题**：`using-clickup-cli` 使用符号链接实现单一真实来源，在某些 Windows 环境和 Git 浅克隆场景下符号链接行为不一致。**改进做法**：在 CI 中加一步 `git ls-files --error-unmatch .agents/skills/using-clickup-cli/SKILL.md` 验证符号链接被正确追踪，或在 README 中说明 Windows 支持状态。**预期收益**：减少跨平台贡献者的困惑。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 95/100（四文件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude-plugin/plugin.json` | 90/100 | 缺少 entries/skills 数组；自动化无法枚举插件面 |
| `.agents/skills/testing-clickup-cli/SKILL.md` | 95/100 | step 3 "edge cases" 措辞略显模糊 |
| `.agents/skills/releasing-clickup-cli/SKILL.md` | 97/100 | 无显著问题 |
| `skills/clickup-cli/SKILL.md` | 97/100 | 无显著问题 |

### 3.2 当时值得借鉴的模式

1. **Output Modes 明确文档化** → 公开 skill 有专门章节列出 TTY 模式与管道模式的行为差异，agent 无需猜测输出格式。根本原因：双模式不是 magic，文档化让 agent 可预测地使用。原文：`skills/clickup-cli/SKILL.md` 的 `## Output Modes` 章节。如何借鉴：为任何有双输出模式的工具加专门章节，明确"在何种条件下输出什么格式"。

2. **DELETE SAFETY 块** → skill 文末用独立区块标注不可逆操作，并给出明确的"先 list 再 delete"模式。根本原因：破坏性操作失误代价高，预防性文档直接降低风险。原文：`skills/clickup-cli/SKILL.md` 末尾 `## DELETE SAFETY` 区块。如何借鉴：所有涉及删除/修改的工具 skill 加同等块。

3. **内部 skill 用 metadata.internal 标记** → 开发者专用 skill（releasing/testing）明确标注 `metadata.internal: true`，与公开 skill 区分。根本原因：防止用户看到内部流程细节，降低认知负担。原文：`.agents/skills/releasing-clickup-cli/SKILL.md` frontmatter。如何借鉴：当项目同时有"公开接口文档"和"内部开发流程"时，用这个字段分层。

4. **版本同步测试** → CI 中有专门测试验证 `package.json`、`plugin.json`、`SKILL.md` 版本字符串完全一致。根本原因：多处硬编码版本号不可避免地会漂移，机器检查比人工检查可靠。原文：`tests/unit/` 中的 version synchronization 测试。如何借鉴：任何有多处硬编码版本号的项目都应加版本一致性测试。

5. **Agent Workflow Examples 分组** → 公开 skill 按任务类型分组展示多命令组合示例（不是单命令罗列），agent 能直接套用工作流模板。根本原因：agent 的实际工作是完成任务，不是记住单个命令——任务级示例比命令级示例更实用。原文：`skills/clickup-cli/SKILL.md` 的 `## Agent Workflow Examples` 章节。如何借鉴：为 CLI 工具写 skill 时，优先补充"组合用法"而非"命令列表"。

### 3.3 当时的缺陷

1. **plugin.json 缺 entries 声明**：[manifest](#manifest) 文件没有 `entries`、`commands` 或 `skills` 数组。为什么会失败：NLPM 等自动化工具无法通过 manifest 枚举插件面，必须做文件系统扫描，破坏了"manifest 是单一权威来源"的约定。自查：我的 bureau 仓库的 `.claude-plugin/plugin.json` 是否同样缺少这个字段？

2. **Skill name 与 package name 不一致**：`skills/clickup-cli/SKILL.md` 的 [frontmatter](#frontmatter) 中 `name: clickup` 而非 `clickup-cli`，与包名、目录名不一致。为什么会失败：用户用 `name` 字段加载 skill，不一致的名字导致误解（用户可能尝试 `clickup-cli` 而非 `clickup`）；无文档说明这是故意的短名别名。自查：我的 skill 的 `name` 字段是否与 skill 所属的工具/项目名称一致？

3. **生产依赖使用 `^` 版本范围**：`@inquirer/prompts`、`chalk`、`commander` 均用 caret 范围，每次 `npm install` 可能拉取不同的 minor 版本。为什么会失败：caret 范围在供应链攻击场景下是风险面——恶意包发布者只需发布符合 semver 约束的版本即可被自动拉取。自查：我的项目（bureau、graphify 等）的 `package.json` 是否也使用 caret 范围？

### 3.4 当时的优化机会

1. 在 `plugin.json` 中加 `"skills": ["skills/clickup-cli/SKILL.md"]`，让 NLPM 和 Claude Code 可以通过 manifest 正确发现 skill，无需文件系统扫描。

2. 在 `testing-clickup-cli/SKILL.md` step 3 的"edge cases"后面追加括号说明，例如：「边界场景（如：缺失可选标志、无效 task ID、API 错误响应）」，把抽象指令变成可执行检查清单。

3. `skills/clickup-cli/SKILL.md` frontmatter 加注释行 `# note: intentionally using short alias 'clickup' instead of 'clickup-cli' for ergonomics`，消除名称不一致的歧义。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 缺 entries 数组 | `cat .claude-plugin/plugin.json`（无 entries/skills/commands 键） | **仍存在** | 一年 14 个版本迭代，这个问题始终没修；维护者认为文件系统发现已足够 |
| 生产依赖 `^` caret 范围 | `cat package.json`：commander 从 `^14.0.3` 升到 `^15.0.0`，其余两个不变 | **部分改变**：commander 升了大版本但依然用 caret；其他两个 caret 范围不变 | commander 大版本升级说明作者不回避 breaking 更新，但锁定策略没变 |
| Skill name 与 package name 不一致 | `head -5 skills/clickup-cli/SKILL.md`（仍为 `name: clickup`） | **仍存在** | 作者确认这是故意的短名别名，非缺陷——但仍无注释说明 |

### 4.2 架构演进

Audit 时（2026-04-06）：3 个独立 SKILL.md（releasing/testing/clickup-cli），无 using-clickup-cli。
现在（2026-07-19 HEAD，v1.39.0）：增加了 `.agents/skills/using-clickup-cli/` 目录，其中 `SKILL.md` 是指向公开 skill 的符号链接。

这一变化说明作者意识到：
- `npx skills add` 等工具期望 skill 在 `.agents/skills/<name>/SKILL.md` 路径下——即规范化安装路径
- 把 skill 同时放在 `skills/` 和 `.agents/skills/` 两处会导致维护重复
- 符号链接是优雅解法：两个路径都有效，单一文件是真实来源

此外，版本从 1.25.2 升到 1.39.0，14 个 minor 版本，说明项目活跃。AGENTS.md 也更完善，加入了详细的"Adding a New Command"和"Modifying Commands"流程说明。

### 4.3 新增的可学习模式

**符号链接统一内外 skill 路径**：`using-clickup-cli/SKILL.md → ../../../skills/clickup-cli/SKILL.md` 这个设计允许：
- npm 包用户通过 `cup skill` 获取最新版（从 `skills/clickup-cli/` 读取）
- AI agent 工具链通过 `.agents/skills/using-clickup-cli/` 找到 skill（规范路径）
- 维护者只需维护一份文件

这在 Audit 时不存在，是架构演进的新增模式，可以直接借鉴到任何需要"同一 skill 在多个路径下可见"的项目。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责 skill**：我的 bureau 和 graphify 等仓库中的 skill 各司其职，与 krodak/clickup-cli 的 releasing/testing/using 三层分离策略一致。
2. **版本在 frontmatter 中声明**：我的 skill 文件在 frontmatter 里声明版本，与这个仓库的版本同步测试精神一致。
3. **AGENTS.md 作为总入口**：我的仓库同样用 AGENTS.md（或 CLAUDE.md）指引 agent 何时用哪个 skill，结构类似。
4. **示例驱动**：我的 skill 中有具体的 input→output 示例，与 krodak 的 Agent Workflow Examples 模式类似。

### 5.2 挑战 / 验证

**挑战**：我之前认为 plugin.json 只要"字段齐全"（name/description/version）就足够了。这个案例说明，即使 plugin.json 格式正确、版本同步做得很好，**缺少 entries 数组这一条就能让自动化工具看不到 skill**——plugin.json 是自动化的接口，不只是人类读的文档。

**验证**：符号链接作为"单一真实来源"的实现方式，我之前对它在 Git 中的兼容性有顾虑。这个案例验证了这个方案在实际项目中是可行的（项目活跃 14 个版本，没有在 Git 符号链接上踩坑的记录）。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否缺少 entries/skills/commands 字段
for f in $(find /tmp/my-repos -name "plugin.json" 2>/dev/null); do
  echo "=== $f ===" && python3 -c "
import json, sys
d = json.load(open('$f'))
missing = [k for k in ['entries','skills','commands'] if k not in d]
print('缺少:', missing if missing else '无（OK）')
"
done
```

命中后怎么办：按缺少的字段补全，例如 `"skills": ["path/to/SKILL.md"]`。

```bash
# 检查我的 package.json 是否使用 caret 范围的生产依赖
for f in $(find /tmp/my-repos -name "package.json" -not -path "*/node_modules/*" 2>/dev/null); do
  echo "=== $f ===" && python3 -c "
import json, sys
d = json.load(open('$f'))
deps = d.get('dependencies', {})
caret = {k: v for k, v in deps.items() if v.startswith('^')}
if caret: print('caret 范围生产依赖:', list(caret.keys()))
else: print('无 caret 范围（OK）')
"
done
```

命中后怎么办：评估是否改为精确版本，或加 `npm shrinkwrap` 保证可重现安装。

```bash
# 检查我的 skill 的 name 字段是否和目录名或包名一致
grep -rn "^name:" /tmp/my-repos/*/skills/ /tmp/my-repos/*/.agents/skills/ 2>/dev/null | head -20
```

命中后怎么办：不一致时，要么对齐命名，要么在 frontmatter 中加注释说明是故意的别名。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 plugin.json 补全 entries/skills 数组，让 NLPM 和 `nlpm:ls` 能正确发现 skill。
   - **为何可行**：bureau 已有结构化 SKILL.md，只需在 plugin.json 里列出路径；参考 krodak 的 patch 思路（在 plugin.json 加 `"skills": [...]`）。
   - **第一步**：打开 bureau 的 `.claude-plugin/plugin.json`，加 `"skills"` 数组，列出所有 skill 路径。估时 10 分钟。

2. **想法**：在 graphify 的 skill 里仿照 DELETE SAFETY 模式，加"不可逆操作"警告块。
   - **为何可行**：graphify 有涉及图数据库节点删除的操作，明确标注这些操作可降低 agent 误操作风险。
   - **第一步**：找到 graphify skill 中描述删除/覆盖操作的章节，在节末加 `> ⚠️ DELETE SAFETY：此操作不可撤销。执行前先用 list 命令确认目标。`。估时 15 分钟。

3. **想法**：如果 bureau 有多个 skill 文件，用符号链接统一规范路径（`.agents/skills/` 指向 `skills/`），而不是维护两份文件。
   - **为何可行**：这个案例验证了符号链接方案在真实项目中稳定可行。
   - **第一步**：检查 bureau 当前是否有重复的 skill 文件，如有，用 `ln -s` 替换副本，并在 CI 中加符号链接有效性验证。估时 20 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 krodak/clickup-cli 的核心目的**：把 SaaS 工具（ClickUp）的 REST API 封装成 AI agent 友好的 CLI，配套 NL skill 文档，同时服务人类和 AI。

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是 Claude Code 插件，都有 plugin.json，都有 skill 文件，plugin.json 都缺 entries 字段 | bureau 没有独立 CLI 二进制；bureau 定位是知识库管理，不是 SaaS 封装 | 高 |
| MarkQWu/graphify | 中 | 都是 AI 工具辅助型插件，都有 NL skill；graphify 也有 Python 脚本作为"可执行核心" | graphify 的核心是图数据库操作，不是 CLI 发布 | 中 |
| MarkQWu/gstack | 中 | 都是多 skill 架构；gstack 有类似的"工具描述 + 使用指南"结构 | gstack 没有 plugin.json；gstack 没有独立 CLI | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都有多个 skill | drama-workshop-skills 聚焦戏剧创作场景，无 CLI 封装 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺 entries/skills 数组 | `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); print([k for k in ['entries','skills','commands'] if k not in d])"` | **bureau 命中**（`.claude-plugin/plugin.json` 同样缺少 entries 字段，已在 2026-07-19-dontbesilent2025-dbskill 案例中确认） | 高 |
| Skill name 与目录/包名不一致 | `grep "^name:" */skills/*/SKILL.md` | 暂无已知命中，待核查 | 低 |

**命中后的具体行动建议**：
- MarkQWu/bureau 的 `.claude-plugin/plugin.json` → 加 `"skills": ["path/to/SKILL.md"]` 数组 → 参考 krodak 的 patch 思路，5-10 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：版本同步的机器保障
   - **本案例做法**：`scripts/sync-command-docs.ts` + CI 中的 version synchronization 单元测试，同时更新三个文件（plugin.json / SKILL.md / docs/commands.md）并在 CI 验证。原文件：`tests/unit/`（version 测试）+ `scripts/sync-command-docs.ts`
   - **我的项目现状**：bureau 和 graphify 的版本更新是手动的，没有机器保障；存在版本漂移风险
   - **如何借鉴**：写一个简单的 Python 脚本检查 plugin.json 的 `version` 和 SKILL.md 第一行的版本字符串是否一致，加入 pre-commit hook 或 CI 步骤

2. **领域**：DELETE SAFETY 块的使用
   - **本案例做法**：`skills/clickup-cli/SKILL.md` 末尾有独立的 DELETE SAFETY 章节，明确列出不可逆操作、建议先 list 再 delete
   - **我的项目现状**：graphify 的 skill 描述图节点操作，但没有专门的不可逆操作警告区块
   - **如何借鉴**：在 graphify skill 末尾加 `## 不可逆操作警告` 章节，列出节点删除、关系清除等操作，给出"先查再删"的模式

3. **领域**：内外 skill 的 metadata.internal 分层
   - **本案例做法**：开发者专用 skill（releasing/testing）用 `metadata.internal: true` 标记，不会混入用户可见的 skill 列表
   - **我的项目现状**：bureau/gstack 的 skill 没有内外区分，开发流程相关的 skill 和用户面向的 skill 平铺在一起
   - **如何借鉴**：将 bureau 中仅用于内部开发的 skill（如"如何发布新版本"）加 `metadata.internal: true`，与用户面向的 skill 区分

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：CLAUDE.md 的完整性
  - **我的做法**：MarkQWu/agents-in-the-wild 的 CLAUDE.md 涵盖架构、命令、代理、技能、钩子、构建步骤、开发规范等，信息量极大
  - **本案例做法**：krodak/clickup-cli 用 AGENTS.md 代替 CLAUDE.md，但覆盖范围主要是 Claude Code 相关配置，没有声明式的 commands/agents/hooks 索引
  - **意义**：结构化的 CLAUDE.md（有 Architecture/Commands/Agents/Skills 分节）比纯 AGENTS.md 更利于 NLPM 类工具发现和扫描

- **领域**：audit 回路（learning case study 机制）
  - **我的做法**：agents-in-the-wild 有自动化的 auditor 流水线，发现缺陷 → 贡献 PR → 追踪结果 → 案例学习形成闭环
  - **本案例做法**：krodak/clickup-cli 没有对外部 PR 的追踪机制，依赖维护者自主发现问题
  - **意义**：外部 audit + 学习案例机制是 agents-in-the-wild 的独特优势，未来若考虑向上游提 PR 关于 entries 字段，这是切入点

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 AI 工具架构模式：有一个真实的可执行程序（如 `cup` 命令）负责真正的计算或 API 调用，同时配套一个 NL（自然语言）技能文件（SKILL.md）告诉 AI agent 如何正确调用这个程序。"NL 表皮"是 agent 读的文档，"原生二进制核心"是实际执行任务的程序。对比：纯 NL skill = 只有文档，没有可执行程序，agent 要自己实现逻辑；混合模式 = 文档 + 可执行程序，agent 只负责调度，程序负责执行。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被自动化工具发现（虽然 Claude Code 的文件系统发现机制可能仍然加载它）。

### <a name="TTY"></a>TTY
> "TeleTYpewriter"的缩写。在现代语境中，"是否在 TTY 中运行"意思是"程序的标准输出是否连接到一个真正的终端（供人看的屏幕）"。如果是 TTY，程序可以显示交互式菜单、颜色等；如果不是（比如输出被管道 `|` 传给另一个程序），程序应该输出干净的文本，不含颜色代码或交互式控件。

### <a name="OIDC-token"></a>OIDC token
> 一种短期身份证。GitHub Actions 在运行 workflow 时可以用 OIDC token 向 npm 等服务证明"我是一个合法的 GitHub Actions 任务"，避免直接把长期 API key 放在仓库里。krodak/clickup-cli 的 release workflow 使用 OIDC trusted publishers 发布到 npm，是安全最佳实践。

### <a name="semver"></a>semver
> "Semantic Versioning"的缩写，版本号格式 MAJOR.MINOR.PATCH（如 `8.3.0`）。`^8.3.0` 中的 `^`（caret）表示"接受 8.x.x 的任何版本，但不接受 9.0.0 以上"，意味着每次 `npm install` 可能拉到不同的具体版本，带来可重现性风险。精确版本 `8.3.0` 表示"只接受这一个版本"。

### <a name="符号链接"></a>符号链接
> 文件系统中的一种特殊文件，它不存储真实内容，只存储指向另一个文件的路径。就像 Windows 的快捷方式。访问符号链接时，操作系统自动转到目标文件。`using-clickup-cli/SKILL.md` 是符号链接，访问它和访问 `skills/clickup-cli/SKILL.md` 读到的内容完全相同。
