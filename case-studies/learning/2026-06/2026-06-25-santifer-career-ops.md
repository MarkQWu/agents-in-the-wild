# santifer/career-ops — 学习案例

**仓库**：https://github.com/santifer/career-ops
**Stars**：31,983 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-25（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `security-gate`, `fallback-chain`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

santifer/career-ops 是一个**求职自动化 Claude Code 插件**，将简历投递、职位评估、跟进管理等求职流程封装成 slash command，让 AI agent 全程协助找工作。关键事实：

- 拥有 31,983 颗星，是「个人效率 + AI agent」赛道中关注度最高的代表
- 同时支持两个 AI Coding CLI：Claude Code（`/career-ops-*` slash commands）和 OpenCode（`.opencode/commands/`）
- 包含 15 个 NL 工件：13 个 OpenCode commands、1 个 SKILL.md、1 个 CLAUDE.md
- 安全状态：**严重阻断**（1 High：`execSync('npm install')` 在远程代码拉取后执行，供应链风险；2 Medium 含 `--dangerously-skip-permissions`）
- NL 评分 **91/100**——在高规模仓库中极少见的高质量 NL 层，缺陷集中在「所有 OpenCode command 缺失 output format」一个维度

### 1.2 架构剖析

- **目录结构**：
  ```
  career-ops/
  ├── .claude/
  │   └── skills/career-ops/SKILL.md   # 核心 skill，路由到各 mode
  ├── .opencode/
  │   └── commands/                    # 13 个 OpenCode thin shim commands
  │       ├── career-ops.md            # 主命令
  │       ├── career-ops-apply.md      # 投递
  │       ├── career-ops-evaluate.md   # 职位评分
  │       └── ...（10 个其他 mode）
  ├── modes/                           # 各 mode 的详细逻辑（.md 文件）
  ├── batch/
  │   └── batch-runner.sh              # 批量处理脚本（含安全问题）
  ├── CLAUDE.md                        # 项目配置文档（96/100）
  └── update-system.mjs                # 自动更新脚本（含 High 安全问题）
  ```

- **文件类型分布**：1 个 SKILL.md，13 个 OpenCode command，1 个 CLAUDE.md，0 个 agent，0 个 hook

- **编排关系**：**单 SKILL 路由层 + 多 mode 扩展**。`SKILL.md` 包含一个路由表（61-62 行），根据用户输入分发到 13 个 mode（apply、evaluate、scan、pdf 等）。OpenCode commands 是 Claude Code 命令的精确映射——`/career-ops-apply` 调用 `skill({ name: "career-ops" })` 并传入 mode 参数。

- **跨件契约**：CLAUDE.md 中声明的 13 个命令 ↔ `.opencode/commands/*.md` 文件一一对应，审计时 0 个断链。`modes/_shared.md` 提供跨 mode 共享逻辑，被 SKILL.md 和各 mode 引用。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「单 skill + 多 mode 路由」——一个 SKILL.md 通过 mode 参数覆盖所有求职场景，避免碎片化。这是「中央路由」模式的典型实现
- **解决什么问题**：求职是一个有明确阶段（搜索→评估→投递→跟进→复盘）的结构化任务，非常适合被 AI agent 接管——每个阶段都有清晰的 input、process、output
- **做了什么 trade-off**：选择支持双平台（Claude Code + OpenCode），代价是维护两套 command 定义；`update-system.mjs` 的自动更新机制牺牲了供应链安全性换取用户体验（无需手动拉取更新）
- **反映什么认知模型**：作者把 AI agent 视为「个人求职 Chief of Staff」——不只是写简历，而是全程状态管理、进度追踪、决策辅助。这比「单次任务」的 AI 使用模式要深一个层次

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「中央路由 SKILL + 薄 Command 层」

单个高质量 SKILL.md 包含完整业务逻辑，多个 command 文件仅作为精薄 [shim](#shim)，负责将不同入口（OpenCode 命令名）映射到同一 skill 的不同 mode。

模式特征清单：
- 特征 1：SKILL.md 的路由表集中管理所有 mode 和触发条件，新增 mode 只需修改一处
- 特征 2：13 个 OpenCode command 全部使用相同模板（`skill({ name: "career-ops" })`），无重复业务逻辑
- 特征 3：`modes/_shared.md` 提供跨 mode 共享配置，避免每个 mode 重复定义通用逻辑
- 特征 4：CLAUDE.md 包含完整的用户引导（欢迎对话、配置步骤、命令参考表），作为项目的「入职文档」
- 特征 5：`batch/batch-runner.sh` 为批量场景提供非交互式路径，无需用户逐个手动触发

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 单一主题的多 mode 工作流 | ✅ 高度适用 | 求职、项目管理等有明确阶段的任务完美匹配这个模式 |
| 需要同时支持多个 AI CLI 平台 | ✅ 适用 | thin shim 模式让新增平台支持成本极低（只需复制 shim 模板） |
| 高度随机化、无结构化工作流的任务 | ❌ 不适用 | 「中央路由」依赖于任务可以被分解为明确的 mode，随机任务无法提前定义 mode |
| 需要零维护的只读工具 | ⚠️ 谨慎 | `update-system.mjs` 引入的供应链风险会随更新频率上升 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：「中央路由 SKILL + 薄 shim」 | santifer/career-ops | 业务逻辑集中、可测试，新平台接入成本低 | SKILL.md 随 mode 增加而膨胀；路由表需要人工维护 |
| 备选 A：「每个 mode 独立 SKILL.md」 | 多数早期插件 | 每个 SKILL 可独立优化和测试 | 共享逻辑重复，mode 间一致性难维护 |
| 备选 B：「单一全能 SKILL 无路由表」 | 传统大 prompt | 配置最简单 | AI 自行判断时机容易误触发或漏触发 |

### 2.4 改进空间

1. **当前问题**：所有 13 个 OpenCode command 缺失 output format 描述（当时扣分维度） **改进做法**：在每个 command 底部加一行 `## Output`，描述该 mode 产出什么（如：`Produces an A–F evaluation report saved to reports/<job-slug>.md`） **预期收益**：用户在触发命令前知道预期产出；NLPM 分数从 90 升至 100
2. **当前问题**：`update-system.mjs` 在 `git fetch` 后执行 `execSync('npm install --silent')`（High 安全） **改进做法**：将 npm install 拆分为独立的 `apply-update` 命令，要求用户确认依赖变更后再执行 **预期收益**：更新流程透明化，供应链攻击窗口缩小
3. **当前问题**：`batch/batch-runner.sh` 使用 `--dangerously-skip-permissions`（Medium） **改进做法**：改为 `--allowedTools` 白名单模式，明确列出批量处理允许的工具 **预期收益**：提示注入攻击无法通过 tool 权限越权

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **91/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .opencode/commands/career-ops-apply.md | 90 | 缺 output format（−10） |
| .opencode/commands/career-ops-evaluate.md | 90 | 缺 output format（−10） |
| [其余 11 个 OpenCode commands] | 90（均） | 全部缺 output format（−10） |
| .claude/skills/career-ops/SKILL.md | 95 | 缺 mode 输出示例（−5） |
| CLAUDE.md | 96 | 模糊量词「a few details」出现 2 次（−4） |

### 3.2 当时值得借鉴的模式

1. **CLAUDE.md 作为「入职文档」** → CLAUDE.md 包含欢迎对话模板、onboarding 步骤、完整命令参考表、技巧与注意事项。使用者第一次打开项目时能通过 CLAUDE.md 快速上手，而无需阅读源码。借鉴：CLAUDE.md 应该是「新用户引导文档」而不只是「项目配置说明」。

2. **路由表设计** → SKILL.md 第 61-62 行的路由表清晰映射了所有 mode 和触发词，审计无断链。借鉴：有多个 mode 的 skill 应该在 SKILL.md 顶部或末尾加一个路由表，用「触发词 → mode → 产出物」格式映射。

3. **双平台支持的 thin shim 模式** → 13 个 OpenCode command 全部是相同模板的实例化，业务逻辑保持在 SKILL.md，新增平台只需复制 shim 模板。借鉴：当一个 skill 要支持多个工具时，用 thin shim 而不是重复 skill body。

4. **批量处理路径** → `batch/batch-runner.sh` 为批量处理 100+ 职位提供了非交互路径，让 agent 可以自动化大量重复操作。借鉴：涉及重复操作的 skill 应该设计一个批量入口（CLI script 或 `batch` mode），不要强迫用户逐个手动触发。

5. **阶段化工作流覆盖** → 从「搜索职位」到「发送跟进邮件」再到「复盘反思」，整个求职生命周期都被 mode 覆盖。借鉴：为领域设计 skill 时，先画出完整的用户工作流地图，确保每个阶段都有对应的 mode 或 command。

### 3.3 当时的缺陷

1. **13 个 OpenCode command 全部缺 output format（−10 × 13）** → 根本原因：作者认为 OpenCode command 是「入口触发器」而非「完整说明文档」，输出由被调用的 SKILL.md 负责描述——这在逻辑上说得通，但 NLPM 的计分规则要求每个 NL 工件自身必须包含 output format。自查：我的 command 文件有没有描述「调用这个命令会产出什么」？

2. **High 安全：`execSync('npm install')` 在 git fetch 后执行** → 根本原因：`update-system.mjs` 的设计目标是「一键更新」，但实现方式是先拉取上游代码，再执行 `npm install --silent`——如果上游 `package.json` 被投毒，所有通过自动更新的用户都会在下次更新时执行恶意 postinstall 脚本。这是供应链安全的经典攻击面。自查：我的项目有没有类似的「从远端拉取后立即执行」的脚本？

3. **Medium：`--dangerously-skip-permissions` 在批量模式** → 根本原因：批量处理时 agent 需要无提示执行，作者选择了最简单的实现方式（跳过所有权限）。风险在于：当 job posting 网页包含提示注入内容时，agent 会无限制地执行恶意指令。自查：我有没有使用 `--dangerously-skip-permissions` 的地方？是否可以改成 `--allowedTools` 白名单？

### 3.4 当时的优化机会

1. 为所有 13 个 OpenCode command 添加一行 output format 描述（批量修改，30 分钟）
2. `batch-runner.sh` 第 338-339 行的 URL 转义补充 `&` 字符的处理（1 行代码，5 分钟）
3. `package.json` 中 js-yaml 和 playwright 的版本号从 `^` 改为精确版本（2 处修改，5 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 13 个 OpenCode command 缺 output format | 搜索 `output_format repo:santifer/career-ops path:.opencode/commands` | **返回 0 结果**：说明 output_format 字段/内容仍不存在 | 该缺陷**未修复**，13 个 command 仍无 output format |
| `execSync('npm install')` 供应链风险 | 间接验证（CHANGELOG 无修复记录） | **可能仍存在**：无法直接确认 | 供应链风险仍有效 |
| `batch-runner.sh` sed `&` 转义缺失 | Bug #3 是确定性一行修复 | **状态不明**：未见 PR 或 commit 记录提及 | 仍需确认 |

### 4.2 架构演进

由于 career-ops 的安全状态为 BLOCKED，NLPM 未对该仓库提交任何 PR——所以当前状态与审计时基本一致，没有 NLPM 引起的外部修改。**NL 层（91 分）依然保持高水平，安全层缺陷依然存在**。这是一个「NL 质量高但安全门阻拦贡献」的典型案例。

### 4.3 新增的可学习模式

通过当前 HEAD 观察：`modes/` 目录的存在（`modes/_shared.md`、`modes/oferta.md` 等）说明作者将「mode 逻辑」从「command 定义」和「skill 路由」中分离出来，形成三层：

```
command（入口触发）→ SKILL（路由）→ modes（业务逻辑）
```

这比「SKILL 包含所有逻辑」更清晰——modes 可以独立维护，skill 只做路由，command 只做映射。**这是「中央路由」模式的进化版本**，值得在自己的多 mode 项目里借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **为 command 文件写 output format**：我的 command 文件描述了产出物格式，不会让用户调用后不知道会得到什么
2. **避免无条件 `--dangerously-skip-permissions`**：我的批量场景如有涉及，会优先使用工具白名单
3. **CLAUDE.md 作为项目说明入口**：我的项目 CLAUDE.md 包含配置说明，类似 career-ops 的做法

### 5.2 挑战 / 验证

这次案例**验证了「路由表」这个犹豫中的设计决策**。我曾不确定是否值得在 SKILL.md 里维护一个显式路由表（感觉像过度设计），career-ops 的案例证明：当 mode 超过 3 个时，显式路由表是必须的——它让维护者（和 AI）在不阅读所有 mode 文档的情况下就能理解系统结构。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 OpenCode command 文件是否有 output format 描述
grep -rL '## Output\|output_format\|输出\|produces\|Produces' .opencode/commands/*.md 2>/dev/null
```
命中后：在每个命中文件末尾添加「## Output」section，描述「调用这个命令会产出什么文件/内容」（5 分钟/文件）。

```bash
# 检查项目中是否有远程拉取后立即执行安装的脚本（供应链风险）
grep -rn 'git.*fetch\|git.*pull\|npm install\|pip install' *.mjs *.sh *.js 2>/dev/null | \
  grep -i 'update\|sync\|refresh'
```
命中后：检查是否有用户确认步骤隔离「拉取」和「安装」操作。

```bash
# 检查是否有 --dangerously-skip-permissions 的使用
grep -rn 'dangerously-skip-permissions' . 2>/dev/null
```
命中后：评估是否可改为 `--allowedTools`，或者是否有 `--print (-p)` 模式可替代。

### 6.2 灵感 → 实施路径

1. **想法**：为自己的多 mode SKILL 添加路由表
   - **为何可行**：career-ops 的路由表（SKILL.md 第 61-62 行）是一个 Markdown 表格，无需任何代码，维护成本极低
   - **第一步**：在现有 SKILL.md 的顶部「## Overview」节后添加一个 3 列表格（触发词 / Mode / 产出物），花 15 分钟把现有 mode 整理进去

2. **想法**：为批量操作场景设计一个「沙盒化」脚本而不是 `--dangerously-skip-permissions`
   - **为何可行**：career-ops 的经验表明，批量模式 + 危险权限是一个明确的安全反模式；`--allowedTools Read,Write,Bash(grep:*),Bash(curl:*)` 等细粒度控制是更安全的替代
   - **第一步**：在现有批量脚本中，把 `--dangerously-skip-permissions` 替换为 `--allowedTools "Read,Write,Bash(grep:*)"` 并测试（30 分钟）

3. **想法**：仿照 career-ops 的「三层架构」（command → skill → modes）重组自己的多 mode 项目
   - **为何可行**：当前很多项目把路由逻辑和业务逻辑混在同一个 SKILL.md，分层后每层职责清晰，可以独立测试
   - **第一步**：识别现有 skill 中最复杂的一个，把业务逻辑提取到 `modes/` 目录下，SKILL.md 只保留路由（2 小时）

---

## 七、对照我的 GitHub 仓库

> 注：本次运行中 my-repos 克隆因网络策略限制失败。以下分析基于 my-repos.json 描述。

### 7.1 目的对齐度

- **本案例 santifer/career-ops 的核心目的**：将求职全生命周期流程化，AI agent 协助执行每个阶段的任务

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为「AI 辅助个人工作流」插件（echo-sleuth 挖掘对话记录，career-ops 管理求职流程） | echo-sleuth 是被动挖掘，career-ops 是主动执行 | 高（架构模式层面） |
| MarkQWu/bureau | 中 | 同为结构化工作流管理（bureau 管理 AI 会话知识，career-ops 管理求职进度） | bureau 侧重知识沉淀，career-ops 侧重行动执行 | 高（CLAUDE.md 作为入职文档） |
| MarkQWu/gstack | 低 | 同为 Claude Code 插件 | gstack 是角色定义而非工作流管理 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 output format | `grep -L '## Output' .opencode/commands/*.md` | echo-sleuth 如有 OpenCode commands，大概率有类似缺失 | 中 |
| SKILL.md 缺 mode 输出示例 | `grep -c 'Example\|示例' skills/*/SKILL.md` | bureau 的多 mode skill 可能缺具体 mode 输出示例 | 中 |
| CLAUDE.md 模糊量词 | `grep -n 'a few\|some\|various\|appropriate' CLAUDE.md` | 所有仓库都可能有这个问题 | 低 |

**命中后的具体行动建议**：
- bureau 的 SKILL.md → 检查各 mode 是否有输出示例 → 补充「## Example Output」节（20 分钟/mode）
- echo-sleuth 的 CLAUDE.md → 搜索模糊量词 → 替换为具体描述（10 分钟）

### 7.3 别人的更优方案

1. **领域**：「CLAUDE.md 作为入职引导文档」
   - **本案例做法**：`CLAUDE.md` 包含欢迎消息模板、分步 onboarding 流程（Step 1: 运行 doctor → Step 2: 配置 profile → Step 3: 运行 scan）、完整命令参考表
   - **我的项目现状**：bureau 和 echo-sleuth 的 CLAUDE.md 根据仓库描述可能更偏技术说明，缺乏 onboarding 引导
   - **如何借鉴**：在 bureau 的 CLAUDE.md 里添加一个「Getting Started」section，包含 3 个具体的「第一次使用」步骤（15 分钟）

2. **领域**：「三层架构：command → skill → modes」
   - **本案例做法**：13 个 thin shim command → 1 个路由 SKILL.md → N 个 modes/ 业务逻辑文件
   - **我的项目现状**：echo-sleuth 可能把所有逻辑集中在单个 SKILL.md 中
   - **如何借鉴**：如果 echo-sleuth 有多个 mode（如 search、analyze、report），提取 modes/ 目录，SKILL.md 只保留路由表

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：「安全设计」
- **我的做法**：echo-sleuth 和 bureau 的设计目标是读取和分析，不涉及自动拉取远程代码后执行安装——没有 career-ops 的供应链攻击面
- **本案例做法**（弱在哪）：`update-system.mjs` 的「git fetch → checkout → npm install」三连操作是一个完整的供应链攻击链；`batch-runner.sh` 的 `--dangerously-skip-permissions` 对提示注入完全开放
- **意义**：设计「个人工具」时不能因为「只有我一个人用」而放松供应链安全；供应链攻击不需要攻击每个用户，攻击上游仓库一次即可批量影响所有使用者

---

## 八、术语表

### <a name="shim"></a>shim（垫片）
> 在软件里，shim 是一个极薄的中间层，把来自一个接口的调用转换为另一个接口的格式，自身不含业务逻辑。career-ops 的 13 个 OpenCode command 就是 shim——每个文件只有几行，把 `/career-ops-apply` 这样的命令转换为 `skill({ name: "career-ops", mode: "apply" })` 调用。

### <a name="中央路由"></a>中央路由
> 一个 skill 接收所有入口请求，根据 mode 参数（或触发词）分发到具体处理器的设计模式。类比：前台（skill）接到来电后，根据「您拨打的是哪个部门」转接给对应工作人员（mode）。优点是业务逻辑集中，缺点是单个 SKILL.md 随 mode 增加而变大。

### <a name="供应链攻击"></a>供应链攻击
> 不直接攻击目标用户，而是攻击目标用户依赖的上游库或服务。career-ops 的风险在于：如果 santifer 的 GitHub 账户被攻击，攻击者可以修改上游 package.json 加入恶意 postinstall，所有通过 `update-system.mjs` 自动更新的用户在下次更新时都会运行恶意代码。

### <a name="thin-shim-模式"></a>thin shim 模式
> 为多平台支持设计的最小代价方案：每个平台的命令文件（shim）只包含「调用共享 skill」的一行代码，业务逻辑统一保留在 skill 中。新增平台支持时只需复制 shim 模板，修改命令名即可，不需要重复维护业务逻辑。

### <a name="postinstall"></a>postinstall
> `npm install` 执行完成后自动触发的脚本（定义在 `package.json` 的 `scripts.postinstall`）。可以执行任意 Node.js 代码，包括下载文件、修改系统路径、运行安装程序。是 npm 供应链攻击最常利用的入口点之一。
