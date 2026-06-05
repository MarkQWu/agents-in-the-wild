# jarrodwatts/claude-hud — 学习案例

**仓库**：https://github.com/jarrodwatts/claude-hud
**Stars**：21,673 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-05（基于当前仓库 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `template-design`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-hud 是一个 Claude Code 插件（21,673 stars），在终端状态栏（statusline）实时显示 Claude Code 的运行状态：context 用量、活跃工具、agent 追踪、todo 进度。它是「给开发者的 Claude Code 仪表盘」，让用户在与 Claude 协作时有更直观的上下文感知。

关键事实：
- 由 [Jarrod Watts](https://github.com/jarrodwatts) 独立开发，是一个结构极简的薄插件
- **技术核心是 TypeScript**：`src/` 目录有 21 个 `.ts` 文件 + `dist/` 编译输出，真正的工作在 Node.js 进程里完成
- NL 制品极少：只有 2 个命令文件（`commands/setup.md`、`commands/configure.md`）和 1 个 `plugin.json`
- 每隔 ~300ms 被 Claude Code 唤起一次，通过 stdin 接收 JSON 数据，渲染后写回 stdout

### 1.2 架构剖析

```
claude-hud/
├── plugin.json                    ← Manifest（满分 100/100）
├── CLAUDE.md                      ← 项目说明（88/100）
├── commands/
│   ├── setup.md                   ← 安装命令（BUG：缺 Write 工具声明）
│   └── configure.md               ← 配置命令（90/100）
├── src/                           ← TypeScript 源码（21 个文件）
│   ├── index.ts                   ← 主入口：stdin JSON → render → stdout
│   ├── transcript.ts              ← JSONL 解析：tools/agents/todos
│   ├── render/                    ← 渲染层：context-line / tools-line / session-line
│   ├── config.ts                  ← 配置读取
│   ├── i18n/                      ← 国际化：en.ts + zh-Hans.ts
│   └── extra-cmd.ts               ← 自定义额外命令（SEC-001：exec 而非 execFile）
└── dist/                          ← 编译产物（63 个 JS 文件）
```

- **文件类型分布**：2 个 command、1 个 project manifest、63 个 JS/TS 代码文件
- **编排关系**：两个命令文件（`setup.md` 和 `configure.md`）职责完全分离，setup 只做首次安装，configure 只做后续配置调整，互不干涉
- **跨件契约**：`plugin.json` 声明了两个命令路径（`./commands/setup.md`、`./commands/configure.md`），两者都存在，[manifest](#manifest) 没有悬空引用

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「NL 表皮 + 原生二进制核心」——NL 命令文件只做安装/配置引导，真正的工作全部在 TypeScript 代码里完成
- **解决什么问题**：用户在 Claude 会话中无法直觉性地感知「还剩多少 context」「现在在用哪个工具」——claude-hud 把这些信息实时显示在终端状态栏
- **做了什么 trade-off**：实现简单（`dist/index.js` 直接读 stdin、渲染、输出）vs 功能扩展性（没有 plugin 架构，要加新指标就得改 TS 代码）
- **反映什么认知模型**：作者理解「NL 制品 = 配置入口，不是逻辑层」——这个仓库是少数真正做到 NL 和代码职责分离的例子

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「NL 表皮 + 原生二进制核心」（[nl-binary-hybrid](#nl-binary-hybrid)）

特征清单：
- **NL 制品只做 2 件事**：安装引导（setup.md）和配置调整（configure.md）
- **核心功能全在 TypeScript**：性能敏感的状态栏每 300ms 刷新一次，必须由编译型代码承担
- **plugin.json 满分**：所有必填字段完整（name、description、version、author、commands、homepage、repository、license、keywords）
- **数据流单向且简单**：Claude Code → stdin JSON → parse → render → stdout，没有副作用

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要高频更新的 UI 组件 | ✅ 高度适用 | 300ms 刷新频率必须用编译型代码，NL 命令撑不住 |
| 需要跨平台（macOS/Linux/Windows）的工具 | ✅ 适用 | TS 编译后可跨平台，setup.md 有平台检测逻辑 |
| 纯领域知识类 skill | ❌ 过度设计 | 法律分析、代码审查等不需要编译层，纯 NL 就够 |
| 快速验证 AI workflow 想法 | ❌ 不适用 | 本模式上手成本高，需要 TypeScript 开发能力 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 二进制核心（本仓库） | jarrodwatts/claude-hud | 性能最优、功能无上限 | 需要编译语言开发能力，维护成本高 |
| 纯 NL 制品 | expo/skills、AgriciDaniel/claude-ads | 零代码、AI 原生、任何人可写 | 功能受限于 AI 能力，无法做高频 UI |
| NL + Python 脚本 | hesreallyhim/awesome-claude-code | 维护脚本功能强，Python 易上手 | 依赖 Python 环境，性能不如编译型 |

### 2.4 改进空间

1. **当前问题**：`setup.md` 的 `allowed-tools` 缺少 `Write`，但 Step 4 需要创建 `config.json`  
   **改进做法**：在 frontmatter 里添加 `Write` 工具：`allowed-tools: Bash, Read, Edit, Write, AskUserQuestion`  
   **预期收益**：Step 4 创建新文件时不会因权限缺失而失败

2. **当前问题**：`extra-cmd.ts` 使用 `exec()`（shell=true），而非 `execFile()` + 参数数组  
   **改进做法**：把 `execAsync(cmd)` 改为 `execFileAsync(binary, args[])`，消除 shell 注入层  
   **预期收益**：即使 `--extra-cmd` 参数包含 shell 特殊字符也不会被解析为命令

3. **当前问题**：`configure.md` 的 description 过于简短（「Configure claude-hud as your statusline」），没有反映命令实际完成的跨平台检测、ghost 安装清理等复杂工作  
   **改进做法**：扩写 description 为「Detect ghost installations, configure statusline command for macOS/Linux/Windows, and write config.json」  
   **预期收益**：用户在命令选择器里看到 description 就能知道这个命令会做什么

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

当时评分 **92/100**，Security: CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `CLAUDE.md` | 88 | 开发者文档，无 NL 制品结构 |
| `commands/configure.md` | 90 | 无渲染后的预览示例 |
| `commands/setup.md` | 91 | BUG：`Write` 不在 `allowed-tools` |
| `.claude-plugin/plugin.json` | 100 | 所有字段完整，无问题 |

**安全状态**：CLEAR——无 Critical/High 安全发现。Medium 1 个（`exec()` vs `execFile()`），Low 4 个（`gh repo star` 触发 GitHub API mutation、3 个未固定 semver 范围）。

### 3.2 当时值得借鉴的模式

1. **plugin.json 满分设计**：所有必填字段（name、description、version、author、commands、homepage、repository、license、keywords）一个不少。这是 [manifest](#manifest) 设计的教科书级别——清单即契约
2. **两个命令的职责分离**：`setup.md` 做一次性安装，`configure.md` 做后续调整——职责完全分离，不混合，不互相依赖。用户第一次用 setup，之后改配置用 configure，直觉上完全正确
3. **CLAUDE.md 的架构描述**：清晰地用 ASCII 流程图展示数据流（`Claude Code → stdin JSON → parse → render lines → stdout`），这种内联架构图对后续维护者极有价值

### 3.3 当时的缺陷

1. **问题**：`commands/setup.md` 的 `allowed-tools: Bash, Read, Edit, AskUserQuestion`——Step 4 需要创建新文件 `config.json`，但缺少 `Write` 工具，`Edit` 无法创建新文件  
   **根本原因**：作者写命令时只想到了「修改现有文件」（Edit），没有意识到创建新文件需要单独的 `Write` 工具。这是一个「测试用例不完整」导致的 bug——只测试了已有 config.json 的情况，没测试首次安装时 config.json 不存在的情况  
   **自查**：我的 echo-sleuth commands 里有没有类似的「声明了 Edit 但实际需要 Write」的情况？

2. **问题**：`setup.md` 第 233 行「within a few seconds」——没有说超过多久应该认为失败  
   **根本原因**：作者从用户体验角度描述了正常情况，但忘记了 AI 需要「异常情况下怎么办」的指导。对人类用户说「几秒内」是正常描述，但对 AI 来说，必须有「超过 X 秒后执行 Y」的明确行为规范  
   **自查**：我的 SKILL.md 里有没有「等待一会儿」「稍等片刻」这类对 AI 没有意义的时间描述？

3. **问题**：`setup.md` 第 316 行「most common cause on macOS」——暗示了一种排序但没有给出依据  
   **根本原因**：作者基于自己的经验给出了一个「常见原因」的排序，但这个排序对 AI 而言是无效的——AI 无法用「常见」来权重化排查步骤，它需要的是「如果 A 不满足则查 B；如果 B 也不满足则查 C」这样的条件树  
   **自查**：我的故障排查文档是「描述性排序」还是「条件性决策树」？

### 3.4 当时的优化机会

1. 在 `setup.md` frontmatter 加 `Write` 工具
2. 把「within a few seconds」改为「within 5 seconds; if it hasn't produced output after 10 seconds, cancel and proceed to the Debug section」
3. 把 `exec(cmd)` 改为 `execFile(binary, args)` 消除 shell 层

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `setup.md` 缺 `Write` 工具 | `grep "allowed-tools" commands/setup.md` → `Bash, Read, Edit, AskUserQuestion`（无 Write） | **PERSISTS** | 首次安装时如果 config.json 不存在，Step 4 仍然会失败 |
| 「within a few seconds」 | `grep "few seconds" commands/setup.md` → 第 286、289 行命中 | **PERSISTS** | 时间描述对 AI 仍无实际约束意义 |
| 「most common cause on macOS」 | `grep "most common cause" commands/setup.md` → 第 577 行命中 | **PERSISTS** | 故障排查指导仍是描述性的，不是决策树 |

### 4.2 架构演进

对比 2026-04-06 到现在，claude-hud 最显著的变化是增加了 **国际化支持**（`src/i18n/en.ts` + `src/i18n/zh-Hans.ts`）——这说明有中文用户群体需求，作者做了响应。NL 制品层（2 个命令文件 + plugin.json）结构没有变化，3 个已知 bug 全部未修复。

Plugin.json 也有小更新：keywords 数组新增了 `todos`，说明 todo tracking 功能是近期新增的。

总体来看：TypeScript 代码层在持续迭代（新功能），NL 制品层静止（没有变化，包括 bug 未修复）。这与 iOfficeAI/AionUi 的模式类似——开发者把精力放在功能层，NL 层被当成「写完不用管」的配置文件。

### 4.3 新增的可学习模式

当前 HEAD 新增了 `i18n/zh-Hans.ts`，说明插件的 UI 文案支持中英双语。这对于面向中文社区的插件有参考价值——把所有用户可见字符串提取到 i18n 文件，而不是硬编码在渲染代码里。

---

## 五、校准

### 5.1 我已经在做对的

1. **plugin.json 字段完整**：echo-sleuth 的 plugin.json（如有）应检查是否所有字段都填写。claude-hud 的 100/100 manifest 是标准参考
2. **命令职责分离**：我的 echo-sleuth `commands/` 目录里每个命令都做单一任务（recall、audit、extract 等）
3. **NL 和代码职责分离的意识**：我理解「复杂逻辑应该在代码里，NL 文件做引导和配置」

### 5.2 挑战 / 验证

本案例挑战了我的一个假设：「92 分的高质量插件应该有完整的 allowed-tools 声明」。

结果发现：92 分的插件在最关键的那个命令（setup.md，用户第一次运行的命令）上有一个会导致失败的 bug。说明**整体高分不等于关键路径无 bug**。

对我的 echo-sleuth 的启示：应该重点检查用户**第一次接触**的那个命令是否完整可用，而不是只看平均质量。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查创建新文件的命令步骤是否声明了 Write 工具
grep -A3 "allowed-tools" ~/.claude/commands/*.md | grep -v "Write" | head -10
# 然后手动检查这些命令里是否有「创建文件」步骤
```
命中后怎么办：找到命令里「创建 X 文件」的步骤，在 frontmatter 的 `allowed-tools` 里加上 `Write`。

```bash
# 检查我的 skill/command 里是否有对 AI 无意义的时间描述
grep -rn "a few seconds\|稍等\|等一下\|a moment\|shortly" ~/.claude/ --include="*.md" | head -10
```
命中后怎么办：替换为具体的时间数字和超时后的行为：「等待最多 10 秒；如果超过 10 秒没有输出，取消并跳转到 Debug 步骤」。

```bash
# 检查故障排查文档是否是条件决策树而不是模糊描述
grep -rn "most common\|通常是\|一般来说" ~/.claude/ --include="*.md" | head -10
```
命中后怎么办：把「最常见的原因是 X」改写为「如果看到错误 A，检查 X；如果检查通过但仍有问题，检查 Y」。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 建一个类似 claude-hud 的「会话健康 HUD」——在终端显示当前挖掘进度：已扫描文件数 / 待处理文件数 / 发现的经验模式数
   - **为何可行**：echo-sleuth 的 skills 每次运行时生成结构化输出（JSONL），可以被一个简单的 Node.js 脚本读取并渲染为状态行
   - **第一步**：参照 claude-hud 的 `dist/index.js` 结构，写一个 `echo-sleuth-status.js`，读取 echo-sleuth 的输出文件，生成状态摘要，约 60 分钟

2. **想法**：用 claude-hud 的「NL 表皮分离」模式重构 echo-sleuth 的 commands
   - **为何可行**：目前 echo-sleuth commands 里有些包含了较多的逻辑判断，这些应该下沉到 skills 或 agents 里，commands 应该只做触发和引导
   - **第一步**：审查 `commands/recall.md`，把「如何判断记忆是否相关」的逻辑从 command 提取到 `skills/memory-management/SKILL.md`，约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 jarrodwatts/claude-hud 的核心目的**：Claude Code 会话状态实时可视化 HUD 插件

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code 可安装插件结构（有 plugin.json 意图） | echo-sleuth 是历史挖掘工具，claude-hud 是实时监控工具 | 高 |
| MarkQWu/claude-for-legal | 低 | 同为 Claude Code 制品 | 领域和交互模式完全不同 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 制品 | 领域完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `Write` 工具但有创建文件步骤 | `grep -rn "Write\|创建.*文件" my-repos/MarkQWu-echo-sleuth-for-claude/commands/` | 需手动核对各命令步骤 | 中 |
| 命令缺 `allowed-tools` 声明 | `grep -L "allowed-tools" my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **命中 4/7**：recap.md、timeline.md、recall.md、lessons.md | 高 |
| 模糊时间描述（「稍等」「等一下」） | `grep -rn "稍等\|等一下\|a moment\|shortly" my-repos/MarkQWu-echo-sleuth-for-claude/` | **需要检查**（快速 grep 未命中，但需人工审核） | 中 |

**命中后的具体行动建议**：
- `echo-sleuth/commands/recall.md` → 加 `allowed-tools: Bash, Read` → 2 分钟
- `echo-sleuth/commands/timeline.md` → 同上 → 2 分钟
- `echo-sleuth/commands/lessons.md` → 加 `allowed-tools: Bash, Read, Write`（如果有写入操作）→ 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：plugin.json 完整性
   - **本案例做法**：`plugin.json` 包含所有字段：`name`、`description`、`version`、`author`（含 url）、`commands`、`homepage`、`repository`、`license`、`keywords`——100/100
   - **我的项目现状**：需要检查 echo-sleuth 的 plugin.json（如有）是否有缺失字段
   - **如何借鉴**：以 jarrodwatts/claude-hud 的 `plugin.json` 为模板，对照检查并补齐 echo-sleuth 的 manifest

2. **领域**：命令文件的架构内联文档
   - **本案例做法**：`CLAUDE.md` 里有明确的 ASCII 数据流图（`Claude Code → stdin JSON → parse → render → stdout`），任何新开发者读完 CLAUDE.md 就能理解整体架构
   - **我的项目现状**：echo-sleuth 的 CLAUDE.md 描述了技术栈但没有数据流图
   - **如何借鉴**：在 echo-sleuth `CLAUDE.md` 里加一个「数据流」节，用 ASCII 图展示「transcript JSONL → git-mining → experience-synthesis → memory-management → recall」的处理链

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：深度领域知识覆盖
- **我的做法**：claude-for-legal 的每个 skill 都包含具体的法律概念、流程步骤和判断标准（如 COPPA 合规、GDPR 要求、诉讼时效等），这是 AI 真正能用来提升工作质量的知识
- **本案例做法（弱在哪）**：claude-hud 的 2 个命令文件都是安装/配置向导，没有领域知识内容。这不是缺点（因为它的核心功能在 TypeScript 代码里），但对于「NL 制品质量」的维度，内容深度明显不如领域知识类仓库
- **意义**：「NL 制品的价值」有两种路径：要么像 claude-hud 一样用代码承载真正的价值，NL 只做接口；要么像 claude-for-legal 一样让 NL 本身承载价值。两条路都是正确的，但不能混淆——不能期待 2 个命令文件承载和 150 个 SKILL.md 一样多的价值

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 Claude Code 插件架构：少量 NL 命令文件（.md）作为用户交互入口（「表皮」），实际工作由编译型代码（TypeScript、Go、Rust 等）完成（「二进制核心」）。适合性能敏感、功能复杂或需要系统级操作的场景。claude-hud 就是典型——状态栏每 300ms 刷新，必须用 Node.js 进程，不能靠 AI 推理。

### <a name="manifest"></a>manifest
> 插件的「清单文件」（`plugin.json`），告诉 Claude Code 这个插件叫什么、有哪些命令、版本号是多少。如果 manifest 里漏写了命令路径，那个命令即使在硬盘上也不会被加载。claude-hud 的 manifest 满分 100/100，因为所有字段都填了，是 manifest 设计的参考标准。

### <a name="allowed-tools"></a>allowed-tools
> 命令文件 frontmatter 里声明该命令可以调用哪些工具的字段。`Edit` 可以修改已有文件，`Write` 才能创建新文件——这是两个不同的工具，不能混用。claude-hud 的 setup.md 遗漏了 `Write`，导致首次安装时创建配置文件这一步会失败。

### <a name="statusline"></a>statusline
> 终端（Terminal）底部或顶部的一行状态信息，持续显示当前程序状态。典型例子：Vim 的状态栏会显示文件名和光标位置。claude-hud 把这个概念用在 Claude Code 上——让用户在与 AI 对话时，实时看到 context 用量、当前工具、进行中的 agent 等信息，不用猜 AI 在干什么。

### <a name="exec-vs-execfile"></a>exec 与 execFile 的区别
> Node.js 执行系统命令的两种方式。`exec(cmd)` 把整个字符串交给 shell 解析，shell 会识别管道（`|`）、重定向（`>`）等特殊字符；`execFile(binary, args[])` 直接调用可执行文件并传参数数组，不经过 shell，特殊字符不会被解析。如果参数来自用户输入，用 `exec` 有 shell 注入风险；`execFile` 则安全——但不支持 shell 管道语法。
