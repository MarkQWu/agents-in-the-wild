# uppinote20/claude-dashboard — 学习案例

**仓库**：https://github.com/uppinote20/claude-dashboard
**Stars**：404 | **来源**：upstream audit（exemplar_published=true，覆盖 <500 星门槛）
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-07-22（基于当前 HEAD）
**主题标签**：`examples-driven`, `template-design`, `manifest-discipline`, `nl-binary-hybrid`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`uppinote20/claude-dashboard` 是一个 Claude Code [插件](#plugin)，在终端 statusline 上显示实时的 AI 使用数据：context 消耗、API rate limit、成本、多 CLI 对比（Claude / Codex / Gemini / z.ai）。代码核心是 TypeScript，NL 工件只有 4 个 command 文件——典型的「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」架构。

关键事实：
- 审计时 6 个 artifacts，现在仍是 6 个（未扩张）—— 功能深度而非广度
- `plugin.json` 版本从 `0.x` 升至 **1.30.0**，说明在过去三个月有 30+ 次版本迭代
- 审计时定位「Claude 状态栏」，现在扩展为「多 CLI 监控」（Claude + Codex + Gemini + z.ai 统一仪表盘）
- 在 NLPM exemplar 库以 **R14/R15/R16/R18/R35/R38** 六条规则正向示范入选
- 审计时 NL 得分 **99/100**——全库仅 1 个 bug，3 个质量小问题

### 1.2 架构剖析

**目录结构**（当前 HEAD）：
```
claude-dashboard/
├── .claude-plugin/
│   ├── plugin.json          # 插件 manifest
│   └── marketplace.json     # 市场元数据
├── commands/                # 4 个 NL command 文件
│   ├── setup.md             # 交互式配置向导
│   ├── setup-alias.md       # Shell alias 安装器
│   ├── check-usage.md       # 使用量查看（调用 TS 脚本）
│   └── update.md            # 插件版本更新
├── scripts/                 # TypeScript 核心实现
│   ├── statusline.ts        # 主入口（statusline）
│   ├── check-usage.ts       # 使用量仪表盘入口
│   ├── widgets/             # Widget 系统（模型/context/成本/rate limit 等）
│   └── types.ts             # 类型定义
├── dist/                    # 编译后的 JS
├── locales/                 # i18n（英文/韩文）
└── .github/workflows/       # CI 和自动发版
```

**文件类型分布**：4 个 command（无 agent，无 skill）+ 1 个 plugin.json + TypeScript 核心
**编排关系**：NL layer 极薄——4 个 command 各自独立，无相互调用。`check-usage.md` 直接调用编译后的 JS 文件；`setup.md` 写入 `~/.claude/settings.json`。NL 文件的职责是「引导配置」，实际数据采集全在 TypeScript 里。
**跨件契约**：命令之间无耦合，都是直接操作文件系统或调用固定路径的 JS 文件。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：NL 只管「配置 / 入口」，TypeScript 管「数据和逻辑」。command 文件里不会有复杂判断逻辑，只有交互步骤和 shell 命令调用
- **解决什么问题**：Claude Code 使用者不知道自己消耗了多少 token / 花了多少钱 / 还剩多少 rate limit——claude-dashboard 填补了这个可见性空白
- **核心 trade-off**：把核心逻辑放在 TypeScript 而非 NL 里，牺牲了「纯 NL 方案的零依赖」，换来了精确的数据计算和可测试性。4 个 command 的精简 NL 层让 99/100 的 NL 质量成为可能
- **反映的认知模型**：作者把 Claude Code 的 NL 能力定位为「配置 wizard」—— Claude 擅长理解用户意图、询问配置参数；TypeScript 擅长精确计算和 API 调用。两者各司其职

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「NL 配置表皮 + 原生二进制核心」的极简版

特征清单：
- 特征 1：NL layer 只有 command 文件（无 skill/agent），厚度极薄
- 特征 2：核心逻辑在编译型代码里（TypeScript），claude-dashboard 的 JS 文件就是「原生核心」
- 特征 3：command 文件的职责是「引导用户完成配置」，数学计算/API 请求交给脚本
- 特征 4：numbered steps + empty-input 处理（R14/R15）是 command 文件质量的决定因素
- 特征 5：`dist/` 预编译，用户安装时无需 `npm install`，零运行时依赖

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要精确数据计算的工具（使用量、成本、指标） | ✅ 高度适用 | JS/TS 精确，NL 不适合做浮点数计算 |
| 需要操作系统文件/配置的工具 | ✅ 高度适用 | NL command 可以调用 Bash 修改 ~/.claude/settings.json |
| 纯文本分析/理解类任务 | ❌ 不适用 | 不需要外部脚本，纯 NL 更简洁 |
| 团队需要频繁修改业务逻辑的场景 | ❌ 不适用 | 修改 TS 需要构建步骤，不如纯 NL 灵活 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 配置表皮 + TS 核心（本案例） | uppinote20/claude-dashboard | 精确、可测试、NL 层极简 | 需要构建步骤，维护 dist/ |
| 纯 NL 表皮（所有逻辑在 command 文件） | vladikk/modularity | 零构建，直接可读 | 复杂计算困难，不适合有状态操作 |
| NL 表皮 + Python 核心 | ooiyeefei/ccc (streak 部分) | Python 生态丰富 | 依赖 Python 环境 |

### 2.4 改进空间

1. **当前问题**：`CLAUDE.md` 的 `commands/` 目录列表只有 2 个命令，实际有 4 个 **改进做法**：更新 CLAUDE.md 的结构树；或者增加 CI 检查：`commands/` 目录文件数与 CLAUDE.md 中列举的数量一致 **预期收益**：贡献者不会因看不到 setup-alias 和 update 命令而困惑

2. **当前问题**：`check-usage.md` 第 39 行 `$ARGUMENTS` 未加引号，存在 shell 注入风险 **改进做法**：把 `$ARGUMENTS` 改为 `"$ARGUMENTS"` **预期收益**：防止用户输入特殊字符（空格、`$()`、`;`）触发非预期命令执行

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）

得分 **99/100**（6 个 artifacts），SECURITY: **REVIEW**（1 个 High + 3 个 Medium + 1 个 Low）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/setup.md | 97 | `Bash(cat:*)` 声明了但步骤中不使用 |
| CLAUDE.md | 98 | "comprehensive" 模糊量词；结构树缺 2 个命令 |
| commands/setup-alias.md | 98 | 步骤标题里有 "appropriate" |
| 其余 3 个文件 | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **Numbered Steps（R14）** → `update.md` 的 3 个步骤带编号、每步有 copy-paste shell block，无 "then" 连接词 → 三步更新命令，阅读者一眼知道边界 → 把我的命令文件里的「bullet 列表」改成「序号步骤」

2. **Empty-Input 分支（R15）** → `setup.md` 在顶部明确区分 `$ARGUMENTS` 有无两条路径，空参数路径用 `AskUserQuestion` 引导 → 用户输错不会静默失败 → 我的所有接受参数的命令都应该有「若 $ARGUMENTS 为空则…」分支

3. **多语言 locales/（R18 原则）** → `locales/en.json` + `locales/ko.json`，UI 文本从代码分离 → 这个模式对 NL 工件来说不直接适用，但体现了「配置与逻辑分离」的思维

4. **CLAUDE.md 作为贡献者向导** → CLAUDE.md 的结构面向「想来改代码的人」而非「想用这个插件的人」，Tech Stack / Project Structure / 开发工作流都写清楚 → 我的 CLAUDE.md 有没有清楚告诉贡献者怎么跑起来？

### 3.3 当时的缺陷

1. **`CLAUDE.md` 结构树遗漏两个命令** → 根本原因：CLAUDE.md 在项目早期（只有 2 个命令时）写好，后来加命令没同步文档。「文档与代码分离」的经典死穴。自查：我的 README / CLAUDE.md 有没有过期的目录结构描述？→ **贡献者向导不准确，造成认知负担**

2. **High 安全漏洞：`$ARGUMENTS` 未加引号** → `node "$(ls ...)" $ARGUMENTS` 里的 `$ARGUMENTS` 没有用双引号包裹。如果用户输入 `; rm -rf ~` 这样的内容会被 shell 执行。根本原因：写命令文件时没有经过安全思维检查。自查：我的命令文件里有没有把用户输入直接拼进 shell 命令？

3. **`Bash(cat:*)` 在 `allowed-tools` 里声明但未使用** → 多声明工具不是安全问题，但是 anti-pattern——让 Claude 以为 `cat` 是这个任务的标准操作，可能影响执行计划。根本原因：重构时删了用到 cat 的步骤，忘记更新 frontmatter。自查：我有没有在 allowed-tools 里声明了但在正文步骤中从不使用的工具？

### 3.4 当时的优化机会

1. 更新 CLAUDE.md 结构树，加入 setup-alias.md 和 update.md
2. `check-usage.md` 第 39 行 `$ARGUMENTS` 加引号（高优先级安全修复）
3. `setup.md` 删除 `Bash(cat:*)` 或者真正在步骤里使用 cat

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 结构树缺 `setup-alias.md` / `update.md` | `grep "setup-alias\|update.md" CLAUDE.md` | **持续** ❌ — CLAUDE.md 第 21 行起仍只列两个命令 | 这个 bug 已存在 3+ 个月，作者优先级不高 |
| `$ARGUMENTS` 未加引号（High 安全） | `grep ARGUMENTS commands/check-usage.md` | **持续** ❌ — 第 39 行仍为 `$ARGUMENTS` 不带引号 | 需要私下披露给维护者，不适合直接 PR |

### 4.2 架构演进

命令文件数量从 4 个→保持 4 个（无增减）。但 TypeScript 核心大幅扩张：
- 从单一 `statusline.ts` → Widget 系统（`widgets/` 目录 10+ 个 widget 文件）
- 新增 `locales/` 国际化支持（英文 + 韩文）
- `plugin.json` 版本从审计时的某个版本升至 **1.30.0**，多 CLI 支持（Codex/Gemini/z.ai）
- `.github/workflows/` 中有自动发版工作流

说明作者意识到：NL 工件层应该保持稳定，快速迭代放在 TypeScript core 里。

### 4.3 新增的可学习模式

**Widget 系统架构**：TypeScript 核心从单文件 → 多 Widget 模块化，每个 widget 独立可测试。这个模式跟 NL 工件的「单职责 skill」思路完全一致——同样的模块化哲学在不同层次的体现。

---

## 五、校准

### 5.1 我已经在做对的

1. **numbered steps 在 commands 里**：我的 commands 文件基本都是步骤化的，符合 R14 要求
2. **功能分离**：我的 bureau 也区分 skill（领域知识）和 command（执行流程），类似 claude-dashboard 的 NL / TS 分层
3. **CLAUDE.md 贡献者向导**：我的仓库有 CLAUDE.md 面向贡献者，思路与 uppinote20 一致
4. **plugin.json 版本管理**：我的 bureau 有明确版本号并随发版更新

### 5.2 挑战 / 验证

这次案例验证了我之前对「命令参数安全」的担忧是正确的：`$ARGUMENTS` 直接拼进 shell 命令是真实风险，即使是 99/100 高分仓库也踩了这个坑。后续检查我的命令文件时要把这个列为优先项。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令里有没有未加引号的 $ARGUMENTS 传入 Bash
grep -rn 'ARGUMENTS' /tmp/my-repos/MarkQWu-bureau/commands/ /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null | \
  grep -v '"$ARGUMENTS"' | grep 'ARGUMENTS'
```
命中后：把 `$ARGUMENTS` 改为 `"$ARGUMENTS"`（加双引号），30 秒可完成每处修复。

```bash
# 检查 CLAUDE.md 结构树与实际文件数是否一致
comm -23 \
  <(ls /tmp/my-repos/MarkQWu-bureau/commands/ | sort) \
  <(grep -o '[a-z-]*\.md' /tmp/my-repos/MarkQWu-bureau/CLAUDE.md | sort | uniq)
```
命中后（有文件在目录但不在文档）：更新 CLAUDE.md 的结构树，把漏写的命令补进去。

```bash
# 检查 allowed-tools 有没有声明了但正文不用的工具
# 对每个 command: 取 allowed-tools 里的工具列表，检查正文是否真的引用了它
grep -A1 "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/lint.md
```
命中后：删除没有在正文中实际使用的工具声明。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的每个 command 增加 empty-input 分支处理
   - **为何可行**：bureau 的命令（note、query、capture）都可能被空参调用
   - **第一步**：打开 `commands/note.md`，在 Task 开头加两段：「If $ARGUMENTS is empty: use AskUserQuestion to ask for content. If provided: …」—— 15 分钟一个命令

2. **想法**：增加 CI 检查，防止 CLAUDE.md 结构树与实际文件脱节
   - **为何可行**：这是纯文本对比，shell 脚本 10 行可实现
   - **第一步**：在 `.github/workflows/` 增加一个 `check-docs-sync.yml`，检查 `commands/*.md` 的文件数与 CLAUDE.md 里提到的文件数是否一致

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 uppinote20/claude-dashboard 的核心目的**：用 NL 工件配合 TypeScript 核心，给 Claude Code 用户提供 AI 使用量可见性

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是 Claude Code plugin，都有 commands + plugin.json | bureau 聚焦知识管理，claude-dashboard 聚焦监控 | 高（NL command 写法可直接参考） |
| MarkQWu/gstack | 中 | 都有多工具覆盖日常工作流 | gstack 无 TS 核心，是纯 NL | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `$ARGUMENTS` 未加引号 | `grep -n 'ARGUMENTS' commands/*.md \| grep -v '"'` | **待检查**（高风险，需要手动验证 bureau 每个命令） | 高 |
| CLAUDE.md 结构树过期 | 比较目录文件与 CLAUDE.md 描述 | **MarkQWu/bureau 可能命中** — bureau 结构在演进中 | 中 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/commands/*.md` → 逐行检查所有 `$ARGUMENTS` 出现位置是否加了双引号 → 高优先级，30 秒一处

### 7.3 别人的更优方案

1. **领域**：NL command 的 empty-input 显式分支（R15）
   - **本案例做法**：`setup.md` 在 Task 开头分两个明确路径："If no arguments provided (interactive mode): use AskUserQuestion…"
   - **我的项目现状**：bureau 的 commands 大概率没有明确的空参数分支（`grep -c "empty\|no argument\|If.*ARGUMENTS" commands/*.md` 需要验证）
   - **如何借鉴**：在 `commands/note.md` 第一段加空参数处理，5 行可搞定

2. **领域**：TypeScript Widget 系统的模块化（对 NL 工件的设计启示）
   - **本案例做法**：每个数据维度（context、cost、rate-limit）是独立 Widget，各自封装
   - **我的项目现状**：bureau 的 skills 可能也有机会按数据来源拆分为更小的单元
   - **如何借鉴**：检查 bureau 的 `compile` skill 是否可以拆为「capture compile」和「review compile」两个独立步骤

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 的精确性
  - **我的项目现状**：（bureau 命令当前实际上缺 allowed-tools，这是一个未修复的 bug —— 所以我在这里不具优势）
  - 暂无明确的优势维度可以相对 uppinote20/claude-dashboard 陈述

本案例未发现我的项目更优的维度（在 NL 工件质量维度，99/100 的仓库没有明显弱点）。

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 Claude Code 插件架构模式：用 Markdown（NL 工件）做用户界面和配置向导，用编译型语言（TypeScript/Go/Rust）做实际的数据处理和计算。"表皮" 指 command 文件，"核心" 指编译后的 JS/二进制文件。优点是精确可测试，缺点是需要构建步骤。

### <a name="plugin"></a>Claude Code 插件（Plugin）
> 一组打包在一起的 commands、skills、agents，通过 `plugin.json`（manifest 文件）向 Claude Code 声明。用户可以一次安装整个插件，而不是逐个安装文件。与「独立 skill」的区别在于有 manifest 文件。

### <a name="statusline"></a>statusline
> Claude Code 窗口底部的状态栏，可以显示自定义信息（如 context 使用量、成本、rate limit 剩余）。claude-dashboard 的核心就是通过 TypeScript 脚本填充这个区域。

### <a name="r14-r15"></a>R14 / R15 规则
> NLPM 的两条重要规则：R14 要求 command 里的多步骤任务必须用数字编号（而非项目符号或散文），R15 要求当 command 接受 `$ARGUMENTS` 时必须显式处理「用户未提供参数」的情形（要么给默认值，要么用 AskUserQuestion 引导）。
