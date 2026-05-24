# Manavarya09/design-extract — 学习案例

**仓库**：https://github.com/Manavarya09/design-extract
**Stars**：2482 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-24（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `security-gate`, `manifest-discipline`, `vague-quantifier`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
design-extract（Manavarya09，2482 stars）是一套**设计语言自动提取工具**：给一个 URL，它会截图、分析、提取完整的设计系统（调色板、字体、间距、WCAG 无障碍分数、组件等），输出为 DTCG tokens、Tailwind CSS 变量、Figma 变量格式。

架构上是典型的「[NL 表皮 + CLI 核心](#nl表皮cli核心)」：8 个 Claude Code command 是用户接口，`npx designlang <subcommand>` 是实际执行层，TypeScript CLI（`bin/design-extract.js`）和 Playwright 浏览器自动化做页面分析，Claude Code command 几乎只是 `npx designlang $ARGUMENTS` 的 NL 包装。2482 stars 说明这个定位（"把任何网站转成设计 token"）击中了设计师/前端工程师的真实需求。

### 1.2 架构剖析
- **目录结构**：
  ```
  design-extract/
  ├── commands/                    # 8 个 Claude Code 命令
  │   ├── extract.md               # 提取设计语言
  │   ├── battle.md                # 两网站对比
  │   ├── brand.md                 # 品牌分析
  │   ├── grade.md                 # 设计评分
  │   ├── pack.md                  # 打包输出
  │   ├── pair.md                  # 配色对比
  │   ├── remix.md                 # 混合两站点设计
  │   └── theme-swap.md            # 主题切换
  ├── skills/extract-design/       # 1 个 SKILL.md（96/100）
  ├── website/CLAUDE.md            # 网站上下文文件
  ├── .claude-plugin/plugin.json   # manifest（clean，100/100）
  ├── src/                         # TypeScript CLI 主体
  ├── bin/design-extract.js        # CLI 入口
  ├── package.json                 # 含 postinstall hook（安全问题）
  └── raycast-extension/           # Raycast 扩展
  ```
- **文件类型分布**：8 个 command / 1 个 skill / 1 个 [manifest](#manifest)（完美：100/100）/ TypeScript/Node.js 主体 / Playwright 浏览器自动化
- **编排关系**：极简。每个 command 几乎就是 `npx designlang <subcommand> $ARGUMENTS`，加上 1-2 行"如果参数为空则问用户"的逻辑。skill（`extract-design`）是 command 的知识库，描述提取的逻辑。`plugin.json` 正确声明了所有 8 个 command 路径和 1 个 skill 路径。
- **跨件契约**：所有 command 都引用 `npx designlang` CLI，与 `package.json` 里的 bin 字段声明一致（`"designlang": "./bin/design-extract.js"`）。plugin.json 和 commands/ 目录内容完全对应。无断链。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「CLI 做分析，NL 做展示」——TypeScript CLI 完成所有的浏览器自动化、CSS 解析、token 提取的技术工作；Claude Code command 只做"告知用户 CLI 在跑"和"根据输出给出建议"的 NL 层工作。
- **解决什么问题**：设计师或前端工程师要分析一个竞品网站的设计语言时，通常需要手动打开 DevTools、逐个记录颜色/字体/间距。这个工具把这个过程压缩为一条命令。
- **做了什么 trade-off**：把 NL layer 做得非常薄（每个 command 只有 10-15 行），优点是 Claude Code 的 NL 层几乎不会引入错误；缺点是 command 只是 CLI 的别名，没有利用 Claude Code 的推理能力做更多的分析（比如 battle 命令不分析两站点的优劣，只展示数据）。
- **反映什么认知模型**：作者认为 Claude Code 的价值在于"让命令行工具对不懂命令行的用户也可用"，而不是"用 LLM 替代命令行工具"。这是务实但功能范围受限的选择。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[NL 薄壳 + CLI 核心](#nl薄壳cli核心)」（Thin NL Shell + CLI Core 架构）**

Claude Code command 只是 CLI 工具的 NL 包装层，几乎不包含任何业务逻辑——业务逻辑全在 CLI 实现。NL 层的主要价值：(1) 空参数处理（没有 URL 时问用户），(2) 输出解读（把 CLI 输出翻译成中文建议），(3) 后续操作引导。

模式特征清单：
- **NL 层极薄**：每个 command ≤ 15 行，就是一个 if-empty-ask + CLI 调用 + 解读输出
- **CLI 是唯一真相**：功能边界由 CLI subcommand 决定，NL command 和 CLI 1:1 对应
- **manifest 严格同步**：plugin.json 声明的命令路径和实际文件完全一致（100/100）
- **无 agent / 无 skill 编排**：没有多 agent 委派，没有 router，单层结构
- **Playwright 渲染引擎**：真正的技术护城河在于 Playwright + TypeScript 的页面分析能力，而非 LLM

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有完整 TypeScript/Node.js CLI 的工具，想加 NL 入口 | ✅ 高度适用 | 正是此模式的设计对象 |
| 设计师想分析竞品网站的设计语言 | ✅ 高度适用 | 核心用例 |
| 需要深度推理分析（"这两个设计哪个更适合 B2B 用户"） | ⚠️ 部分适用 | 当前命令只展示数据，不做推理；需要加强 NL 层才能支持 |
| 纯 NL 任务（写作/分析文本） | ❌ 不适用 | 过度工程化，直接用 skill 更合适 |
| 企业生产环境（需要安全审计） | ❌ 不适用 | $ARGUMENTS 的 [shell injection](#shell-injection) 漏洞未修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 薄壳 + CLI 核心（本仓库） | design-extract | CLI 做实际计算确定性强，NL 层极简不易出 bug | $ARGUMENTS 注入风险，NL 层价值低（Claude 几乎只是转发） |
| NL 薄壳 + Python 后端（MCP） | NanoResearch | 功能更强大，MCP 解耦合优雅 | 安装复杂，MCP 依赖文档化困难 |
| 纯 NL skill | KhazP/vibe-coding | 零外部依赖，安装最简单 | 不能做确定性计算，不能自动化页面截图 |
| 多 agent 专家委员会 | LigphiDonk/OMP | 复杂任务分工灵活 | 安装/注册复杂，10 个 agent 全缺 frontmatter |

### 2.4 改进空间
1. **当前问题**：8 个 command 里 `$ARGUMENTS` 直接拼入 shell 命令（`npx designlang battle $ARGUMENTS`），存在 [shell injection](#shell-injection)（HIGH 安全风险）。**改进做法**：在 command 里让 Claude 先验证 `$ARGUMENTS` 只包含合法的 URL 格式（`https?://...`）和 known flags，再传给 CLI；或在 CLI 侧用参数解析库（commander.js 的解析层已有）对输入做校验。**预期收益**：消除 HIGH 安全风险，保护 2400+ 用户。
2. **当前问题**：`package.json` 的 postinstall 自动运行 `npx playwright install chromium --with-deps`，后者会触发系统级 apt 包安装，在某些 Linux 环境需要 sudo。**改进做法**：把 postinstall 改为打印提示"需要运行 `npx playwright install chromium` 完成安装"，或用 `--no-deps` 只安装浏览器本体。**预期收益**：避免 `npm install` 时意外触发系统包安装，降低 CI 环境的权限需求。
3. **当前问题**：2 个 command（extract.md, remix.md）的后续操作用 prose 描述而不是编号步骤。**改进做法**：把"run → read → summarize → offer follow-ups"这 4 步写成编号列表 `1. 运行提取 2. 读取 *-design-language.md 3. 呈现摘要 4. 提供后续选项`。**预期收益**：NLPM R22 规则要求多步骤用编号，编号步骤 Claude 执行更精确，减少跳步的可能。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 审计时得分 **94/100**，安全扫描 BLOCKED——是本批次 4 个仓库里 **NL 质量最高**的，但安全问题最集中（9 个 HIGH 发现）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/extract.md | 83 | 多步骤用 prose 描述（R22 -10）；"tight summary" vague（-2） |
| commands/remix.md | 85 | 同上（post-run flow 无编号步骤 -10） |
| commands/pair.md | 93 | 缺 allowed-tools；"most distinctive crossover" vague（-2） |
| 其余 5 个 command | 95 | 缺 allowed-tools（-5） |
| skills/extract-design/SKILL.md | 96 | "key findings"、"notable" 两个 vague quantifiers（各-2） |
| .claude-plugin/plugin.json | 100 | 完美 |

### 3.2 当时值得借鉴的模式

1. **plugin.json 100 分（完美 manifest）** → `plugin.json` 精确声明了 8 个 command 路径和 1 个 skill 路径，所有路径都真实存在，无多余声明，无遗漏。根本原因好：作者每次新增 command 时同步更新了 manifest，没有"文件有了但 manifest 忘了更新"的漂移。如何借鉴：我的 echo-sleuth 和 drama-workshop-skills 需要核查 manifest 同步情况——每次新增命令/skill 后检查 manifest。

2. **空参数处理的统一模式** → 所有 8 个 command 都有"如果 `$ARGUMENTS` 为空或不足，问用户"的逻辑（"If `$ARGUMENTS` is empty or contains fewer than two URLs, ask the user for both sites"）。根本原因好：这防止了 CLI 因参数为空而报错崩溃，给用户更好的交互体验。如何借鉴：我的 echo-sleuth `/recall` 命令如果没有参数应该有明确的默认行为（"无参数时，显示最近 7 天的 session 列表"）而不是报错。

3. **NL 薄壳的克制性** → command 文件非常短（≤15 行），只做 CLI 调用和后续引导，不试图用 LLM 替代 CLI 做分析工作。根本原因好：知道 LLM 擅长什么（理解意图、解读输出、引导操作），不擅长什么（确定性计算、浏览器截图、CSS token 解析）。如何借鉴：echo-sleuth 的 scripts/ 做的是确定性 JSONL 解析，这部分应该保持在 Python 脚本里，不要尝试用 Claude 来做字节级解析。

### 3.3 当时的缺陷

1. **[SEC-HIGH x8] 所有 8 个 command 的 `$ARGUMENTS` 直接传给 shell** → `npx designlang battle $ARGUMENTS` 里 `$ARGUMENTS` 没有引号包裹，用户输入 `; rm -rf ~` 会执行删除命令。根本原因：NL 表皮 + CLI 核心架构里，command 作者通常把 `$ARGUMENTS` 当成 "safe URL"，没有意识到它可以包含任意 shell 字符。自查：我的 echo-sleuth 命令**不使用** `$ARGUMENTS` 直接传给 shell（所有 shell 调用都通过 `scripts/*.sh` 执行，参数是内部生成的路径），这一点我是安全的。

2. **[SEC-HIGH] `package.json` postinstall 自动触发系统级安装** → `npx playwright install chromium --with-deps` 在某些 Linux 上需要 `apt-get install` 系统依赖，可能触发 sudo 提权。用户以为在做 `npm install`，实际上可能触发了系统包变更。根本原因：追求"零摩擦安装"把系统级操作打包进了 postinstall，但这超出了 npm 包安装应有的权限范围。自查：我的项目 `package.json`（如果有的话）不应该在 postinstall 里触发系统级安装。

3. **extract.md 和 remix.md 用 prose 描述多步骤** → 违反 R22（多步骤必须用编号列表），Claude 在执行 prose 描述的步骤时容易跳步骤或合并步骤。自查：我的命令文件需要检查是否有 ≥4 步的操作用 prose 而非编号列表描述。

### 3.4 当时的优化机会
1. 给 8 个 command 全部加 `allowed-tools: Bash, Read` 声明
2. 把 extract.md 和 remix.md 的多步骤 prose 改为编号列表
3. 把 postinstall 改为打印安装指引而不是自动执行系统操作

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| SEC-HIGH: $ARGUMENTS shell injection（所有 8 commands） | `grep -n "\$ARGUMENTS" commands/battle.md` | **仍然存在** — `npx designlang battle $ARGUMENTS`（无引号） | 约 7 周（从 4/06 至今）未修复；2400+ 用户在使用存在注入漏洞的命令 |
| package.json postinstall `--with-deps` | `grep -n "postinstall" package.json` | **基本保留，加了 fallback** — `"postinstall": "npx playwright install chromium --with-deps 2>/dev/null || npx playwright install chromium"` | 作者意识到了一些问题（加了 `2>/dev/null`），但 `--with-deps` 仍在，系统级安装风险依然存在 |
| commands 缺 allowed-tools | `grep -n "allowed-tools" commands/battle.md` | 需进一步核查（当前 HEAD battle.md 只有 description + argument-hint frontmatter） | 可能仍未添加 |

### 4.2 架构演进
最值得关注的变化是 postinstall 加了 `2>/dev/null || npx playwright install chromium` 的 fallback——说明维护者已经意识到 `--with-deps` 会在某些环境下失败，但没有完全移除它。HIGH 安全漏洞（shell injection）约 7 周内未修复，可能是因为维护者认为 Claude Code command 的 `$ARGUMENTS` 是"受控环境"，不认为这是真实风险。

### 4.3 新增的可学习模式
`package.json` 里增加了 fallback 处理（`2>/dev/null || ...`），这是一种"降级路径"（[fallback-chain](#fallback-chain)）思路——先尝试完整安装，失败了再尝试精简安装。这个模式可以学习：在安装脚本里，用 `|| fallback_cmd` 提供更小权限要求的备用路径。

---

## 五、校准

### 5.1 我已经在做对的
1. **manifest 同步**：echo-sleuth 的 commands/ 和 agents/ 目录中的文件都在 CLAUDE.md 里有记录，没有"文件存在但未声明"的幽灵文件。
2. **空参数处理**：echo-sleuth 的命令有默认行为（如 `/recall` 没有参数时显示最近 session 列表），而不是硬报错。
3. **$ARGUMENTS 不直接传 shell**：echo-sleuth 的 commands 调用 `scripts/*.sh` 时，传入的参数是脚本内部生成的确定性路径，不是用户原始 `$ARGUMENTS`。

### 5.2 挑战 / 验证
- **挑战了的假设**："94/100 的仓库安全风险很低"——design-extract 的 NL 分数是本批次最高的（94/100），但安全上有 9 个 HIGH（最多）。NL 质量评分和安全评分完全正交，高 NL 分不代表安全。
- **验证了的认知**："manifest 严格同步是基础规范"——design-extract 的 plugin.json 得了 100 分，是全部 11 个 NL artifact 里唯一完美的文件，原因是作者做到了 manifest 和实际文件完全对应。这证实了"manifest discipline"是可以做到完美的，只要每次增删文件时同步更新 manifest。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令是否有 $ARGUMENTS 直接传 shell 的情况
grep -rn "\$ARGUMENTS" /tmp/my-repos/MarkQWu-*/commands/*.md 2>/dev/null | grep -v "quoted\|\"\\$ARGUMENTS\""
# 命中（未加引号的 $ARGUMENTS）：将命令改为先验证参数格式（URL 正则），再传给 shell

# 检查 drama-workshop-skills 的 install.sh 是否有 postinstall 类型的系统级操作
grep -n "apt\|brew\|sudo\|pip install" /tmp/my-repos/MarkQWu-drama-workshop-skills/install.sh 2>/dev/null
# 命中则评估是否必要；若非必要则改为打印提示

# 检查 echo-sleuth 和 drama-workshop-skills 的 plugin manifest 与实际文件是否同步
diff <(ls /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md | xargs -I{} basename {} .md | sort) \
     <(cat /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/CLAUDE.md | grep "^- \`/" | sed 's/.*`\/\([^`]*\)`.*/\1/' | sort)
# 有差异则更新 CLAUDE.md 的命令列表
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 design-extract 的 plugin.json 100 分实践，检查并修复 echo-sleuth 的"manifest 同步"
   - **为何可行**：echo-sleuth 增加了新命令（/prune 等），CLAUDE.md 里的命令列表可能落后
   - **第一步**：对比 `ls commands/` 的输出和 CLAUDE.md 里列出的命令列表，找出不一致的条目，更新 CLAUDE.md（约 10 分钟）

2. **想法**：给 drama-workshop-skills 的 commands 加 `allowed-tools` 声明（参考 design-extract 的规范化水平）
   - **为何可行**：drama-workshop-skills 有多个命令（/开始、/策划、/分镜等），其中一些命令用 Read/Write 工具，但 frontmatter 里未声明
   - **第一步**：检查 `short-drama/SKILL.md` 里哪些命令步骤会调用文件工具，然后在相应命令 frontmatter 里加 `allowed-tools` 声明（约 20 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 Manavarya09/design-extract 的核心目的**：从 URL 自动提取完整设计语言（tokens/变量/组件）供设计师/前端工程师使用

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 都是"从输入（URL/剧本）提取结构化输出（设计 token/剧情骨架）"的工具型插件 | design-extract 是 NL 薄壳+CLI；drama-workshop 是纯 NL skill | 高（manifest discipline、空参数处理、CLI 集成模式） |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有结构化输出目标 | echo-sleuth 面向个人学习，design-extract 面向设计工作流 | 低 |
| MarkQWu/claude-for-legal | 低 | 都有领域专注的 skill | 完全不同领域 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `$ARGUMENTS` 未引号传 shell | `grep -rn "\$ARGUMENTS" /tmp/my-repos/MarkQWu-*/commands/*.md` | **可能命中**：需实际核查 drama-workshop-skills 和 echo-sleuth 的命令（它们是否有 Bash 执行步骤用到 `$ARGUMENTS`） | 高（若命中则是安全漏洞） |
| Command 缺 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **命中** 4 个：lessons/recall/recap/timeline.md | 中 |
| 多步骤 post-run flow 用 prose 描述 | `grep -c "##\|^[0-9]\." /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/recall.md` | 需核查；recall 命令有复杂的后续操作，可能未完全编号化 | 低 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/commands/recall.md` → 检查是否有 `$ARGUMENTS` 直接传 shell；若有则改为 `"$ARGUMENTS"` 引号包裹或验证格式 → 5 分钟
- `echo-sleuth-for-claude/commands/lessons.md` → 加 `allowed-tools: Bash, Read` → 3 分钟
- 核查 drama-workshop-skills 的 install.sh 是否有 postinstall 类系统级操作（brew/apt 自动安装）→ 10 分钟

### 7.3 别人的更优方案

1. **领域**：plugin.json manifest 的完整性（100/100）
   - **本案例做法**：`plugin.json` 精确声明所有 8 个 command 和 1 个 skill，与实际文件完全对应，得了满分。维护者每次改文件都同步 manifest。
   - **我的项目现状**：echo-sleuth 没有 plugin.json，命令靠 CLAUDE.md 里的列表来声明；但 CLAUDE.md 里的命令列表可能落后于实际命令数（新增了 `/prune`、`/extract`、`/dashboard` 后有没有同步更新 CLAUDE.md 需要验证）。
   - **如何借鉴**：给 echo-sleuth 添加一个 `plugin.json` 来统一声明所有命令和 skill（这也有利于未来发布到 marketplace）；现有 CLAUDE.md 的命令列表改成从 plugin.json 生成。

2. **领域**：CLI 集成的空参数安全处理
   - **本案例做法**：所有 8 个 command 都有 `If $ARGUMENTS is empty ... ask the user for X` 的统一模式，空参数时不把空字符串传给 CLI（否则 CLI 会报 missing required argument 错误）。
   - **我的项目现状**：echo-sleuth 的 `/recall` 命令如果空参数，行为依赖 echolib.py 的参数解析，可能出 IndexError。
   - **如何借鉴**：在每个 echo-sleuth command 的 frontmatter 后加"如果 `$ARGUMENTS` 为空，默认为 `--last 7d`（最近 7 天）"的明确 fallback，而不是让脚本去猜。

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安全性——没有 $ARGUMENTS shell injection
- **我的做法**：echo-sleuth-for-claude 的命令调用脚本时传入的是内部生成的文件路径（`~/.claude/projects/...`），而非未经处理的用户输入 `$ARGUMENTS`；drama-workshop-skills 的命令不涉及 shell 执行，直接是 NL 指令。
- **本案例做法**（弱在哪）：8 个 command 都有 `npx designlang <cmd> $ARGUMENTS`（无引号包裹），存在 shell injection。这是 HIGH 级安全漏洞，7 周内未修复。
- **意义**：这个安全基线是我应该持续保持的——新增命令时要特别注意 `$ARGUMENTS` 的使用方式，永远不要直接拼接进 shell 字符串。

---

## 八、术语表

### <a name="manifest"></a>manifest
> 项目的"清单文件"（`plugin.json`），告诉 Claude Code 这个插件包含哪些组件（commands、skills、agents 的路径）。design-extract 的 manifest 得了满分（100/100）的原因：声明的路径和实际文件完全一一对应，没有多余也没有缺失。如果 manifest 里漏写了某个 command，那个命令即使有 .md 文件也不会被插件加载。

### <a name="nl表皮cli核心"></a>NL 表皮 + CLI 核心
> 见 NanoResearch 案例的"NL 表皮 + 原生后端"。design-extract 是这个模式的一个特别轻量的变体：Claude Code command 只是 `npx designlang <subcommand>` 的 NL 包装，几乎没有任何 Claude 推理在里面——CLI 做所有实际工作。称为"薄壳"（thin shell）以区别于 NanoResearch 里 Claude 还需要做策略决策的情况。

### <a name="nl薄壳cli核心"></a>NL 薄壳 + CLI 核心
> 见上条。

### <a name="shell-injection"></a>shell injection（命令注入）
> 当 Claude Code command 把 `$ARGUMENTS`（用户的原始输入）直接拼接进 Bash 命令字符串时，攻击者可以在输入里嵌入 `; rm -rf ~` 等特殊字符执行任意命令。防御方法：用 `"$ARGUMENTS"` 加引号包裹（防止空白分词），或在传给 shell 之前验证输入只包含合法字符（如 URL 格式）。

### <a name="fallback-chain"></a>fallback-chain（降级路径）
> 一种容错设计模式：先尝试功能最完整的操作；失败了则自动降级到更简单但权限要求更低的备用操作。design-extract 的 postinstall 已经开始应用这个思路：`npx playwright install chromium --with-deps 2>/dev/null || npx playwright install chromium`（先尝试完整安装含系统依赖，失败了只装浏览器本体）。对于安装脚本，fallback-chain 意味着"尽量安装，安装不了就提示用户手动处理"，而不是直接失败退出。

### <a name="dtcg-tokens"></a>DTCG tokens
> Design Token Community Group tokens，一种标准化的设计 token（颜色、字体、间距等设计变量）格式，由 W3C 工作组定义。用 JSON 结构存储设计系统的原子变量，可以导入 Figma、转换为 Tailwind CSS 变量、或导出为 CSS 自定义属性。design-extract 的核心输出就是把网站的视觉样式"逆向工程"成 DTCG token 文件。
