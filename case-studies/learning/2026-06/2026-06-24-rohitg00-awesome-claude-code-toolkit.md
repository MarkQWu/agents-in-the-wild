# rohitg00/awesome-claude-code-toolkit — 学习案例

**仓库**：https://github.com/rohitg00/awesome-claude-code-toolkit
**Stars**：1,214 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-06-24（基于 Audit 报告 + xiaolai 案例，无法实时克隆）
**主题标签**：`manifest-discipline`, `template-design`, `examples-driven`, `monorepo-vs-split`, `curl-pipe-bash-risk`

**xiaolai 案例**：[../../case-studies/2026-05-11-rohitg00-awesome-claude-code-toolkit.md](../../case-studies/2026-05-11-rohitg00-awesome-claude-code-toolkit.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

rohitg00/awesome-claude-code-toolkit 是 GitHub 上 **内容最齐全的第三方 Claude Code 组件合集**，由 [Rohit Ghumare](https://github.com/rohitg00) 维护。它不是一个单一功能插件，更像一座可安装的组件城市：135 个 agent、42 个 command、176+ 个插件、20 个 hook、15 条 rule、7 个模板、14 个 MCP 配置、26 个伴生应用、52 个生态条目——一次安装，数十个领域的 Claude Code 增强能力同步到位。

关键事实：
- xiaolai 审计时（2026-04-17）该仓库有 **1,617 stars 和 496 forks**（registry 当前显示 1,214，可能因注册时间差所致）
- 以插件形式组织：顶层 `plugins/` 下按领域分 sub-plugin，`agents/` 下有专域 agent，`hooks/` 下有 25 个 hook 条目
- 覆盖至少 40 个技术领域：DevOps、移动、安全、AI/ML、可访问性、观测性、区块链、游戏开发、SEO 等
- 558 个 NL artifact 总量；审计抽样 100 个

> **注意**：由于代理环境限制，本案例无法实时 clone 目标仓库，所有状态描述均基于 2026-04-17 审计快照及 2026-05-11 xiaolai 案例报告。当前仓库实际状态以实时 clone 为准。

### 1.2 架构剖析

xiaolai 给这份案例起名叫 **「Scale Without Schema」（无 Schema 的规模化）**。这个标题精准：仓库在 _内容规模_ 上做到了极致，却在 _结构规模_ 上几乎一致性地缺席。

```
awesome-claude-code-toolkit/
├── plugins/
│   ├── devops-toolkit/commands/          # ≈10 个 command，全部无 frontmatter
│   ├── mobile-dev/commands/              # ≈8 个 command，全部无 frontmatter
│   ├── security-suite/commands/          # ≈9 个 command，全部无 frontmatter
│   ├── accessibility-checker/commands/   # 含 aria-fix.md（无 frontmatter）
│   ├── screen-reader-tester/commands/    # 含 fix-aria.md（无 frontmatter）
│   ├── code-guardian/
│   │   ├── agents/reviewer.md            # ✅ 有结构
│   │   └── commands/security-scan.md    # ⚠️ 无 frontmatter
│   ├── api-architect/
│   │   └── agents/api-expert.md          # ✅ 有结构
│   └── ...（共 176+ 个插件，82 个 command 文件全部无 frontmatter）
├── agents/
│   ├── specialized-domains/
│   │   ├── embedded-systems.md          # ✅ 10 步流程 + 技术标准 + 验证 (~85/100)
│   │   ├── geospatial.md                # ✅ 同上
│   │   ├── blockchain.md                # ✅ 同上
│   │   ├── game-dev.md                  # ✅ 同上
│   │   ├── seo-specialist.md            # ✅ 同上
│   │   └── ...（共 15 个专域 agent）
│   └── developer-experience/
│       └── api-documentation.md         # ✅ 结构完整
├── hooks/
│   ├── hooks.json                       # 25 个 hook 条目（每次 Write/Edit/Bash 触发）
│   └── scripts/
│       ├── auto-test.js                 # ⚠️ file_path 传给 execFileSync，无工作目录边界检查
│       ├── post-edit-check.js           # ⚠️ 同上
│       ├── lint-fix.js                  # ⚠️ 同上
│       ├── block-dev-server.js          # ⚠️ 基于 TMUX 环境变量做安全决策（可伪造）
│       ├── secret-scanner.js            # 🔗 与 code-guardian 命令互补但无交叉引用
│       └── ...（共 19 个 JS 脚本 + 1 个 Python 脚本）
└── templates/                           # 7 个模板文件
```

- **文件类型分布**：18 个 agent（15 专域 + 1 开发者体验 + 2 plugin-embedded）、82 个 command、558 个 NL artifact 总量
- **编排关系**：插件级平铺，各 plugin sub-directory 独立，不共享 agent 或 skill。hook 作为横切面覆盖全局所有写操作
- **跨件契约**：`code-guardian/commands/security-scan.md` 与 `hooks/scripts/secret-scanner.js` 功能互补（命令层 + hook 层的双重安全检查），但两者互不引用，用户无法从任一侧发现另一侧的存在

### 1.3 设计思路 / 方法论

「无 Schema 的规模化」这个标题揭示了一个架构矛盾：

- **Scale（规模）**：本仓库在内容广度上可能是整个 Claude Code 生态独一无二的——40 个技术领域、558 个 NL artifact、一次安装解决多团队需求
- **Without Schema（无 Schema）**：82 个 command 文件全部缺失 frontmatter，这不是疏漏，而是从未有过模板约束的系统性结果

这暗示了一个开发流程假设：**agent 是精心设计的，command 是批量生成的**。18 个 agent 有完整的 10 步流程、技术标准、验证节点——显然有人认真设计过 agent 模板并严格执行；82 个 command 的步骤体仅有分类标签（"Deploy:"、"Validate:"）而无动作内容——显然是从薄模板批量产出，且没有过 frontmatter 门控的 review 步骤。

两类制品在同一仓库里相距 48 分（agent avg 85 vs command avg 37），像邻居却被分发了不同的入住须知。

核心 trade-off：**选择了覆盖广度，代价是结构一致性**。一个 558 artifact 的仓库，如果每个 command 都精心维护，维护成本接近 40 个独立插件的总和。批量生产节省了时间，但使 82 个 command 在插件注册表里没有规范身份。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「领域聚合插件集（Domain-Aggregation Plugin Monorepo）」**

核心特征：
- **多域 plugin sub-directory**：每个技术领域是一个独立的 plugin 子目录，有自己的 commands/ 和可选的 agents/
- **全局 agent 层分离**：跨域通用的高质量 agent 集中在顶层 `agents/`，与各域插件解耦
- **全局 hook 横切面**：`hooks/hooks.json` 覆盖所有写操作，作为工具调用的横切面拦截层
- **量大而约束薄**：依赖人工维护而非模板门控来保证质量一致性

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多团队"一次安装，全域覆盖"的工具链需求 | ✅ 高度适用 | 558 artifact 的广度正是为此设计，一次 `plugin install` 即可 |
| 团队快速探索 Claude Code 不同领域能力 | ✅ 适用 | 15 个专域 agent 各有完整结构，可直接参考 |
| 依赖插件注册表做命令发现（如 `/help`）| ❌ 当前不适用 | 82 个 command 无 frontmatter，注册表退回到文件名推断 |
| 作为 NL 工程质量标杆参考 | ⚠️ 有选择地适用 | Agent 层可作正面标杆；command 层需要回避 |
| 个人单一领域插件开发 | ❌ 不适用 | 合集架构，不适合单一场景的聚焦插件 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域聚合 monorepo（本仓库） | rohitg00/awesome-claude-code-toolkit | 覆盖广，一次安装，跨域探索成本低 | 质量一致性难控，注册表可见性依赖模板纪律 |
| 单一高质量插件 | kepano/obsidian-skills | 聚焦深度，质量完全可控，schema 一致 | 功能范围窄，跨域需多次安装 |
| 开放社区 monorepo | ComposioHQ/awesome-claude-plugins | 贡献门槛低，生态活跃 | 质量方差大，多作者风格冲突 |
| 精选资源汇编 | hesreallyhim/awesome-claude-code | 发现成本极低，Star 最多 | 第一方 NL artifact 极少，质量不可控 |

### 2.4 改进空间

1. **当前问题**：82 个 command 无 frontmatter，注册表无法识别。**改进做法**：建立一个 `templates/command-template.md`（带完整 frontmatter），并在 PR 模板中加"是否已复制自 template"的 checklist。也可用脚本一次性批量注入 frontmatter 骨架（name 从文件名推断，description 从第一个 `#` 标题提取）。**预期收益**：82 个命令对注册表可见，`/help` 输出完整，score 从 46 升至约 62（仅 frontmatter 修复即 +16 分）。

2. **当前问题**：18 个 agent 全部缺失 `## Example` 块（-15 each）。**改进做法**：每个 agent 文件末尾补一个最小示例（触发输入 → 期望输出格式，3-5 行即可）。专域 agent（blockchain、geospatial 等）加示例后，score 可再 +15。**预期收益**：agent 均分从 85 升至 ~97，整体 score 达到约 72（超过默认阈值 70）。

3. **当前问题**：`code-guardian/commands/security-scan.md` 与 `hooks/scripts/secret-scanner.js` 互补但互不引用，用户无法感知双层保护的存在。**改进做法**：在 `security-scan.md` 的步骤体加一行「注意：本命令与 hooks/scripts/secret-scanner.js 形成双层检测，hook 在每次写操作时实时触发」；在 `secret-scanner.js` 顶部注释加「另见 /security-scan 命令用于手动全量检测」。**预期收益**：跨件契约显式化，降低新用户认知成本。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-17 当时得分 **46/100**（100 个 artifact 抽样，加权均值）。

| 文件类型 | 数量（抽样）| 平均分 | 主要问题 |
|---|---|---|---|
| Agents（专域 + plugin-embedded） | 18 | ~85/100 | 全部缺 `## Example` 块（-15 each） |
| Commands（高质量层）| 21 | ~45/100 | 无 frontmatter（-82），有步骤内容但骨架薄 |
| Commands（核心层） | 60 | ~35/100 | 无 frontmatter，步骤体仅有分类标签 |
| Commands（最低层）| 1 | ~25/100 | 无 frontmatter + 无输出格式 + 无空输入保护 |

**Bug 清单（164 个）**：

| 问题 | 数量 | 影响 |
|---|---|---|
| 缺 `name:` frontmatter | 82 | 注册表无法规范识别 command |
| 缺 `description:` frontmatter | 82 | `/help` 无帮助文本，marketplace 不可见 |
| 缺 `allowed-tools:` frontmatter | 82 | 工具权限未声明 |
| 缺 `## Example` 示例块 | 18 (agents) | 预期行为未规定 |
| 缺 `## Format` 输出格式节 | 24 | 输出结构无规范 |
| 无空输入保护 | 32 | 参数省略时行为未定义 |
| 步骤体骨架化 | 24 | 步骤仅为分类标签，无动作内容 |

**安全发现（CLEAR，无高危/致命）**：

| 级别 | 文件 | 问题 |
|---|---|---|
| MED | auto-test.js、post-edit-check.js、lint-fix.js | `file_path` 传给 `execFileSync` 无工作目录边界检查（使用数组参数，实际风险低） |
| MED | 6 个 JS 脚本 | 写会话状态到 `~/.claude/`（项目目录外，设计意图如此） |
| MED | command 文件 | 指导 Claude 运行 `gh pr view <number>`，未提供用户参数消毒指引 |
| LOW | block-dev-server.js | 基于 `process.env.TMUX` 做安全判断（环境变量可伪造） |
| LOW | hooks.json | 25 个 hook 条目全部覆盖 Write/Edit/Bash（攻击面宽，不影响日常使用）|

### 3.2 当时值得借鉴的模式

1. **专域 agent 的 10 步流程模板**：`agents/specialized-domains/` 下的每个 agent 都遵循相同的骨架——「角色定位 → 核心能力 → 10 步工作流程 → 技术标准 → 验证节点」。这不是偶然，而是有人认真设计过 agent 写作模板并严格执行了它。路径：`agents/specialized-domains/*.md`。借鉴：在我的仓库里，如果有多个同类 agent，应当先建立一个 agent-template.md，把骨架固化，然后再批量产出。

2. **领域覆盖策略**：embedded-systems、geospatial、blockchain、game-dev、SEO——这 15 个专域 agent 的选题本身就是一个可复用的「Claude Code 高价值应用领域清单」。借鉴：评估自己的 agent 集是否覆盖了目标用户最高频的使用场景，而不是从技术栈出发，从用户场景出发。

3. **双层安全检测架构**（设计意图层面）：`code-guardian` 命令提供手动触发的全量安全审查，`secret-scanner.js` hook 提供自动触发的实时检测——两者形成互补。即使这个架构没有被显式文档化，设计意图本身是正确的：静态命令 + 动态 hook 的双重覆盖。借鉴：为我的安全相关制品规划「命令层（手动深度）+ hook 层（自动实时）」的双重防线。

4. **`aria-fix` 与 `fix-aria` 的命名冲突风险提示**：`accessibility-checker/commands/aria-fix.md` 和 `screen-reader-tester/commands/fix-aria.md` 在不同目录下，文件名不同因此不碰撞，但功能描述重叠。这说明：在 monorepo 里，不同插件之间缺乏跨域命名协调机制时，功能漂移是默认结果，不是意外。借鉴：如果我维护多个 plugin，应当维护一个「命令功能清单」，防止不同插件的命令在功能上悄悄重叠。

### 3.3 当时的缺陷

1. **82 个 command 全部缺 frontmatter**：根本原因不是遗漏，而是从未有过——command 的创建流程里不存在「必须填 frontmatter」的步骤，也没有 lint/CI 门控。自查：我的 commands/ 目录下，是否每个文件都有 `name:`、`description:` 和 `allowed-tools:` 三行？

2. **18 个 agent 全部缺示例块**：agent 模板设计得很好（10 步结构 + 技术标准），但缺少了「## Example」节。根本原因：模板作者可能认为专域 agent 的「示例」场景太具体，没有放进通用模板。自查：我的每个 agent 文件末尾，有没有至少一个「触发输入 → 期望输出」的具体示例？

3. **32 个 command 缺空输入保护**：这些命令依赖用户提供参数（如 PR 号、文件路径），但没有声明当参数缺失时的行为。根本原因：批量生产的命令体通常描述「正常流程」，防御性分支（空输入、错误输入）需要额外思考和篇幅。自查：我的每个需要用户参数的命令，有没有一个「如果用户没有提供 X，则…」的分支？

4. **步骤体骨架化（24 个 command）**：步骤写的是「Deploy:」「Validate:」这样的分类标签，不是动作指令。根本原因：LLM 批量生产的命令结构合理，但内容填充不足，没有经过人工 review 补充动作语义。自查：我的命令步骤，是否每步都有一个具体的动词 + 对象（如「运行 `npm test`，验证输出无 FAIL」），而不只是个名词？

### 3.4 当时的优化机会

1. 用脚本一次性给 82 个 command 注入 frontmatter 骨架（name 从文件名推断，description 从 `#` 标题提取，allowed-tools 置为空列表待填写），而后人工逐一补充 `allowed-tools`
2. 给 18 个 agent 各补充 1 个 `## Example` 节（哪怕只是一个「输入：……，期望输出：……」的 3 行示例）
3. 在 `security-scan.md` 和 `secret-scanner.js` 中各加一行交叉引用注释，显式化双层检测架构
4. 修复 `block-dev-server.js` 的 TMUX 检查——改为检查 TMUX socket 是否实际存在（`test -S $TMUX`）或使用更可靠的进程检查

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

xiaolai 案例记录，两个 issue 于 **2026-05-11 18:55:58 和 18:56:01 UTC 关闭**（相差 3 秒）。这个 3 秒间隔不像是逐一阅读，更像是一次批量关闭操作。仓库历史中没有找到提及 NLPM 的 commit，没有提交记录说明 frontmatter 被批量添加，也没有 PR 被打开（`contribute-approved` 标签从未被加到任何一个 issue 上）。

| 过去缺陷 | 检查方法 | 现状推断 | 含义 |
|---|---|---|---|
| 82 个 command 缺 `name:` frontmatter | `head -5 plugins/*/commands/*.md` | **无法直接验证（克隆受限）**；未见对应修复 commit | 若未修复：82 个命令仍对注册表不可见 |
| 82 个 command 缺 `description:` frontmatter | 同上 | **无法直接验证（克隆受限）**；未见对应修复 commit | 若未修复：`/help` 仍无帮助文本 |
| 18 个 agent 缺 `## Example` 块 | `grep -L "## Example" agents/**/*.md` | **无法直接验证（克隆受限）** | 若未修复：agent 期望行为仍未规范 |
| block-dev-server.js TMUX 检查 | `grep TMUX hooks/scripts/block-dev-server.js` | **无法直接验证（克隆受限）** | 安全决策基于可伪造环境变量的风险保留 |

**结论**：现有证据无法确认 frontmatter 修复是否已实施。3 秒关闭的间隔、无 NLPM commit 的历史、无 PR 记录——三者合并指向「issue 被关闭为信息性，而非修复性」。但这是推断，不是事实。**无法直接验证当前状态（克隆受限）**。

### 4.2 架构演进

基于审计快照，无法观察到 2026-04-17 后的具体演进。可以合理推断：

- **不变的可能性高的部分**：插件目录结构（558 artifact 的重组成本极高）、专域 agent 的 10 步骨架（已经是亮点）
- **可能有变化的部分**：star 数量（从 1617 降至 1214，可能是 registry 记录时间差，也可能反映实际关注度变化）；个别 hook script 的安全改进（MED 级别的发现有可能触发维护者自主修复）

### 4.3 新增的可学习模式

无法从克隆受限的状态中观察到 audit 后新增模式。如果修复已发生，批量脚本注入 frontmatter 的具体方案本身会是一个值得记录的「机械修复」模式案例。

---

## 五、校准

### 5.1 我已经在做对的

1. **Frontmatter 完整性**：如果我的 commands/ 下每个文件都有 `name:`、`description:` 和 `allowed-tools:`，我就已经避免了本案例 82 个 command 的核心缺陷。本案例是最有力的反面证明：558 artifact 规模，46/100 的根因只是三行 YAML 从未被要求存在。

2. **Agent 与 Command 使用不同模板**：本案例揭示了一个教训——同一仓库里，agent 和 command 如果来自不同质量标准的创作过程，会产生显著的分数分化（85 vs 37）。如果我在开发不同类型制品时都使用了对应的质量模板，这个分化就不会发生在我的仓库里。

3. **跨件引用的显式化**：如果我的 hook 脚本和对应的命令文件有交叉引用注释（各自提到对方的存在），我就已经比本案例的 `secret-scanner.js` + `security-scan.md` 做得更好——后两者功能互补却互不知晓。

### 5.2 挑战 / 验证

本案例**挑战了一个关于规模的直觉**：「仓库越大、star 越多，质量维护压力越高，出问题是正常的」。但仔细分析会发现，本案例的问题不是「规模导致的维护失控」，而是「模板阶段的遗漏被规模放大了」。

- 如果 command 模板从第一天就包含 frontmatter，82 个命令就都有了——批量生产反而是优势
- 如果 agent 模板从第一天就包含 `## Example` 节，18 个 agent 就都有示例了——同一个模板执行了多次

这验证了一个原则：**模板是 NL 工程里最高杠杆的投资点**。一行模板改动 = N 个文件的质量改变。本案例的教训不是「不要做大仓库」，而是「在第一个文件里就做对」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否全部有 frontmatter（name + description + allowed-tools）
find ~/.claude/commands/ ~/my-repos/ -name "*.md" -path "*/commands/*" 2>/dev/null | \
  while read f; do
    missing=""
    grep -q "^name:" "$f" || missing="${missing} name"
    grep -q "^description:" "$f" || missing="${missing} description"
    grep -q "^allowed-tools:" "$f" || missing="${missing} allowed-tools"
    [ -n "$missing" ] && echo "MISSING[$missing]: $f"
  done
# 命中后：在文件顶部 --- ... --- 块中补入缺失字段
```

```bash
# 检查我的 plugin 下是否有功能重叠的命令（简单版：同关键词出现在不同 command 文件里）
find ~/my-repos/ -name "*.md" -path "*/commands/*" 2>/dev/null | \
  xargs grep -l "aria\|accessibility\|a11y" 2>/dev/null
# 命中多个文件时：对比功能描述，确认是否真正互补还是重复覆盖
```

```bash
# 检查我的 agent 文件是否全部有示例块
find ~/my-repos/ -name "*.md" -path "*/agents/*" 2>/dev/null | \
  xargs grep -rL "## Example\|## 示例\|## 用法示例" 2>/dev/null
# 命中后：在文件末尾加 ## Example 节，至少一个「输入 → 输出」的 3 行示例
```

```bash
# 检查我的 hook script 有没有对 file_path 做工作目录边界检查
grep -rn "execFileSync\|exec\|spawn\|child_process" ~/my-repos/ --include="*.js" --include="*.py" 2>/dev/null | \
  grep -v "test\|spec\|mock"
# 对每处命中：确认传给子进程的路径参数来源（是 hook 输入？是用户输入？）并加 cwd 或路径断言
```

### 6.2 灵感 → 实施路径

1. **想法**：建立一个 `templates/command-template.md`，作为 gstack 等仓库的 command 创建起点
   - **为何可行**：本案例最有力的教训——批量生产命令不是错，但必须从有 frontmatter 的模板批量。我的 gstack 有 23 个工具，若没有模板约束，同样的风险就在那里
   - **第一步**：参考 NLPM conventions 的 command frontmatter 规范，写一个包含 `name`、`description`、`allowed-tools`、`## 步骤`、`## 输出格式`、`## 空输入处理` 的模板文件，放到 `templates/command-template.md`，在 CONTRIBUTING.md 里要求新命令从模板复制创建 → 预计 20 分钟

2. **想法**：给 gstack 或 graphify 的每个 agent 补充 `## Example` 节
   - **为何可行**：本案例 18 个 agent 因为缺示例集体 -15 分，而 agent 的 10 步结构已经很好——只差最后那一步。我的 agent 如果也缺示例，相同的问题就在复现
   - **第一步**：列出我的仓库里所有 agent 文件（`find . -path "*/agents/*.md"`），逐一确认是否有示例块，对每个缺失的补一个 3-5 行的「输入场景 → 期望行为描述」示例 → 每个约 10 分钟

3. **想法**：检查 bureau（知识库管理）仓库中任何 hook script 是否对外部输入路径做了边界检查
   - **为何可行**：bureau 的核心功能是捕获和编译 AI 会话输出，如果用到 hook script 读写文件，file_path 来自 hook 输入——这与本案例 auto-test.js 的 MED 级发现完全同构
   - **第一步**：打开 bureau 的 hook script 文件，找到任何接受 file_path 参数的地方，加 `assert path.is_relative_to(project_root)` 或等价检查 → 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：用户仓库信息（`MarkQWu/gstack`、`MarkQWu/bureau`、`MarkQWu/graphify`、`MarkQWu/drama-workshop-skills`）

### 7.1 目的对齐度

- **本案例 rohitg00/awesome-claude-code-toolkit 的核心目的**：聚合多个技术领域的 Claude Code 组件（agent、command、hook、plugin），以单一仓库提供「全域覆盖」的 AI 编程辅助工具集

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多个 Claude Code 工具的集合（23 个工具），覆盖不同工程角色 | gstack 聚焦「工程角色视角」（CEO、设计师、工程经理），本案例聚焦「技术领域视角」（DevOps、移动、安全）；规模差异大（23 vs 558） | 高 |
| MarkQWu/bureau | 中 | 同为有 NL artifact 的 AI 工具 | bureau 是单一用途工具（会话 → 知识库），本案例是多域工具集 | 中 |
| MarkQWu/graphify | 中 | 同为 Claude Code skill，有 NL artifact | graphify 是单一 skill（代码 → 知识图谱），本案例是大型聚合 | 中 |
| MarkQWu/drama-workshop-skills | 高 | **最相似**：同为 Claude Code skills 集合，面向社区，多个工具组合在一起 | drama-workshop 聚焦单一领域（gobuildit 社区剧本创作），本案例跨 40 个领域；drama-workshop 无 NL artifact（has_nl_artifacts: false），本案例有 558 个 | 高 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令（示意）| 我的项目推断命中情况 | 严重度 |
|---|---|---|---|
| command 缺 `name:` frontmatter | `grep -rL "^name:" gstack/commands/` | **待核实**：gstack 有 23 个工具，若无统一模板约束，存在风险 | 高（若命中） |
| command 缺 `description:` frontmatter | `grep -rL "^description:" gstack/commands/` | 同上 | 高（若命中） |
| command 缺 `allowed-tools:` | `grep -rL "^allowed-tools:" gstack/commands/` | 同上 | 高（若命中） |
| agent 缺 `## Example` 块 | `grep -rL "## Example" gstack/agents/` | **待核实**：取决于 agent 模板 | 中 |
| 跨 command 功能重叠（无命名协调） | 人工 review 命令功能描述 | **drama-workshop-skills 有此风险**：社区 skill 集合，多个贡献来源可能有功能重叠 | 中 |

> **注**：无法实时 clone 用户仓库，上述均为基于仓库描述的推断，需实际 grep 验证。

### 7.3 别人的更优方案

1. **领域**：专域 agent 的结构化模板
   - **本案例做法**：`agents/specialized-domains/` 的 15 个 agent 全部遵循「角色 → 10 步流程 → 技术标准 → 验证节点」骨架，显然有统一模板在背后支撑
   - **我的项目（gstack）推断现状**：gstack 有 23 个工具覆盖不同角色（CEO、设计师、工程经理、发布经理等），若每个工具的 agent/command 结构不一致，用户切换工具时需要重新学习
   - **如何借鉴**：在 gstack 里建立一个 `templates/role-agent-template.md`，定义统一的「角色入口 → 核心能力 → 工作流程 → 验证步骤」骨架，让 CEO 工具和工程经理工具的结构保持一致

2. **领域**：双层安全检测（命令层 + hook 层）
   - **本案例做法**：`code-guardian` 命令 + `secret-scanner.js` hook 形成手动深度 + 自动实时的双重覆盖（尽管未被显式交叉引用）
   - **我的项目（bureau）推断现状**：bureau 的核心功能是捕获 AI 会话并编译为知识库，若处理外部内容（如从 git log 抓取），内容安全检查可能只在命令层，缺少 hook 层的实时拦截
   - **如何借鉴**：在 bureau 的 `capture` 流程中，除命令层的内容过滤外，加一个 PostToolUse hook 在写入知识库文件时检查是否含有敏感内容（API key 模式、密码模式）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：规模与质量的平衡策略
- **我的做法（推断）**：gstack 的 23 个工具规模合理，如果有统一的 frontmatter 模板，维护成本可控；drama-workshop-skills 作为社区工具集，聚焦单一领域，质量一致性相对容易保证
- **本案例弱在哪**：558 artifact、82 个 command 无一有 frontmatter——规模本身不是问题，但规模掩盖了模板缺失的代价。当一个缺陷被复制 82 次时，修复成本就变成了机械修复的技术债
- **意义**：保持「小而精」或者「大而有模板门控」，两者都比「大而无约束」更可持续。我的仓库若能在第一个文件里就做对 frontmatter，之后每增加一个工具都是对的

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据。Claude Code 的 command 文件必须有 frontmatter 才能被插件注册表识别。`name:` 是命令的规范名称（用于 `/name` 调用）；`description:` 是 `/help` 里显示的简介；`allowed-tools:` 声明命令有权调用哪些工具。三行缺一，命令就从「已安装」变成「不可见」。类比：书在书架上，但没有书脊标签——你没法通过书名找到它。

### <a name="command"></a>command（斜杠命令）
> Claude Code 的用户触发入口，用户输入 `/command-name` 即可执行。存储为 `commands/*.md` 文件，格式是「frontmatter + 自然语言步骤描述」。与 skill 的区别：command 是用户显式调用的，skill 是 Claude 在需要时参考的背景知识。command 需要 frontmatter 才能注册；skill 通常不需要（但有 `name:` 会更好）。

### <a name="agent"></a>agent
> 一种特殊的 Claude Code 制品，封装需要多步骤、多工具协作的复杂任务。有自己的 `model`、`tools`、`description` 声明，可以被其他 command 或 skill 调度，也可以直接在 Claude Code 里作为 subagent 调用。与 command 的区别：agent 有自己的执行上下文，可以跨多个工具调用保持状态；command 是单次执行的指令集。

### <a name="plugin-registry"></a>plugin registry（插件注册表）
> Claude Code 在安装插件后维护的组件索引，记录了所有已知 command、agent、skill 的规范名称和描述。注册表依赖 frontmatter 的 `name:` 字段来建立索引；如果 `name:` 缺失，注册表只能从文件名推断（`aria-fix.md` → `aria-fix`），没有显示名称，也没有描述。`/help` 的输出和 marketplace 的展示都来自这个索引。

### <a name="hook"></a>hook（钩子）
> Claude Code 在特定工具调用事件（Write、Edit、Bash 等）前后自动执行的脚本。本案例有 25 个 hook 条目，覆盖每次 Write/Edit/Bash——这意味着 hook 是一个「全局拦截层」，任何写操作都会触发。hook 的攻击面随触发频率增大：越是全局覆盖，越需要关注 hook script 本身的安全性（如 `file_path` 的边界检查）。

### <a name="allowed-tools"></a>allowed-tools
> command 文件 frontmatter 中声明工具权限的字段。如果 `allowed-tools: Bash, Read` 则该命令只能调用 Bash 和 Read 工具，不能调用 Edit 或 Write——Claude Code 会在工具调用前做权限检查。缺少 `allowed-tools` 时，工具权限处于未声明状态，行为取决于全局默认策略。本案例 82 个 command 全部缺此字段，意味着工具权限边界在命令层完全未定义。

### <a name="manifest-discipline"></a>manifest discipline（manifest 纪律）
> 维护者确保 manifest（plugin.json 或 frontmatter）字段始终完整、准确的工程习惯。本案例的「manifest 纪律」在 agent 层得到了执行（所有 agent 有完整结构），在 command 层从未存在（82 个 command 无一有 frontmatter）。「manifest 纪律」是 NL 工程里最容易通过模板 + CI 强制的质量维度，却也是最容易在批量生产时被忽略的一个。
