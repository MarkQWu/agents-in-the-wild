# aaronmaturen/claude-plugin — 学习案例

**仓库**：https://github.com/aaronmaturen/claude-plugin
**Stars**：1 | **来源**：本地 audit
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-28（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `template-design`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
aaronmaturen/claude-plugin（插件名 "atm"）是一个面向全栈 Web 开发工作流的 Claude Code 插件，提供 26 个命令，覆盖代码审查、无障碍审计、Angular/Django 框架操作、Git 历史管理、PR 反馈处理、演示文稿生成等领域。仓库还集成了 Context7 MCP 服务（通过 npx 动态拉取），使命令能访问实时的第三方文档。插件配置文件位于 `.claude-plugin/plugin.json`，有别于更常见的根目录 `plugin.json` 放置方式。当前仓库处于**静止状态**——自 2026-04-29 audit 以来，26 个命令文件全部未经修改，两个已确认缺陷均未修复。

### 1.2 架构剖析
- **目录结构**：
  ```
  claude-plugin/
  ├── .claude-plugin/
  │   └── plugin.json        # 插件清单（含 MCP 配置）
  ├── .claude/
  │   └── skills/
  │       ├── ai/SKILL.md    # AI 协作技能（84/100）
  │       └── bash/SKILL.md  # Bash 脚本技能（88/100）
  ├── commands/              # 26 个命令文件（均无 frontmatter）
  ├── CLAUDE.md              # 项目入口，重定向至 AGENTS.md
  ├── AGENTS.md              # 实际项目文档
  ├── README.md
  └── PLUGIN.md
  ```
- **命令分布**（26 个）：技术深度命令（spike-investigation、feature-investigation、bug-investigation）、框架专属命令（angular-component、angular-service、django-model、django-view 等）、工程质量命令（a11y-audit、a11y-expert、ai-agent-audit、pr-review、self-review）、Git 操作命令（git-revise-history、commit-msg、summarize-branch）、输出生成命令（generate-slidedeck、ticket-explainer、scaffold、simplify）。
- **MCP 集成**：`plugin.json` 的 `mcpServers` 配置通过 `npx -y @upstash/context7-mcp` 启动 Context7，直接影响 4+ 个命令的文档检索能力。
- **CLAUDE.md 设计**：CLAUDE.md 仅有一行重定向，将读者引导至 AGENTS.md——这是一个有意的间接层，使项目文档与 Claude Code 配置入口解耦。

### 1.3 设计思路 / 方法论
- **核心理念**：将全栈工程师的日常高频操作打包为标准化命令，减少重复输入上下文的成本。26 个命令覆盖"调研 → 实现 → 审查 → 发布"完整链路。
- **MCP 作为知识扩展层**：Context7 集成体现了作者的判断——框架文档（Angular、Django 等）变更频繁，不适合静态写入 skill，而是在运行时从权威来源动态获取，保持命令长期有效。
- **CLAUDE.md 间接层**：将真实文档放在 AGENTS.md，CLAUDE.md 只做重定向，使 AI Agent 生态（如 GitHub Copilot、Cursor 等）和 Claude Code 都能找到文档入口，且未来改文档路径时只需改一个重定向。
- **Trade-off**：广度优先（26 个命令）换来了工具箱的覆盖面，但也带来了维护成本——frontmatter 缺失横跨所有文件，说明没有任何自动化质量门禁在工作，单个作者难以手动维护 26 个文件的规范性。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「动态文档注入型插件」（MCP-backed Command Plugin）**

命令不把框架文档内联进 skill，而是依赖 MCP 服务在运行时查询权威文档，命令本身只描述操作步骤和判断逻辑。这与"自包含 skill"（把知识写死在 SKILL.md 里）形成对比。

模式特征：
- **MCP 作为知识后端**：命令/skill 只写"如何做"，"知道什么"来自外部服务
- **零文档维护成本**：框架文档更新时命令不需要修改
- **26 命令扁平布局**：无分层、无 skill 引用，每个命令独立可用
- **CLAUDE.md 重定向模式**：配置入口与文档内容物理分离

### 2.2 适用场景

| 场景 | 适用度 | 原因 |
|---|---|---|
| 全栈开发者，同时维护 Angular 前端 + Django 后端 | ✅ 高度适用 | 命令按框架分类，覆盖二者的核心操作 |
| 需要 Context7 实时框架文档的项目 | ✅ 高度适用 | MCP 集成是核心设计意图 |
| 企业内网环境，无法访问 npx/npm | ❌ 不适用 | MCP 启动依赖 `npx -y @upstash/context7-mcp`，断网场景失败 |
| 需要严格安全审计的项目（如金融、医疗） | ⚠️ 需评估 | 未固定版本的 npm 包在每次启动时可能拉取不同代码 |
| 插件共享给团队或发布到 marketplace | ❌ 当前不适合 | 26 个命令均无 frontmatter，Claude Code 无法正确注册 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 动态文档注入（本仓库） | aaronmaturen/claude-plugin | 知识永远最新，命令无需随文档迭代 | MCP 依赖外部网络；版本未固定存在安全风险 |
| 自包含 skill（静态知识） | JuliusBrussee/cavekit | 离线可用；无网络依赖；完全可审计 | 文档过时需手动更新 skill |
| 混合型（skill + 按需 MCP） | 高成熟度插件 | 离线降级 + 实时增强 | 设计复杂度更高 |

### 2.4 改进空间
1. **当前问题**：所有 26 个命令缺少 YAML frontmatter（name、description、allowed-tools）。**改进做法**：每个命令文件顶部加最小 frontmatter，例如 `pr-review.md` 改为声明 `name: atm:pr-review`、`description: 审查 PR 代码质量`、`allowed-tools: Read Bash`。**预期收益**：Claude Code 能正确解析并注册命令，分数上限从 45 提升至 100。
2. **当前问题**：`@upstash/context7-mcp` 未固定版本，每次 `npx -y` 拉取最新版，存在供应链风险。**改进做法**：改为 `@upstash/context7-mcp@1.x.x`（固定 minor 版本）或使用 `--prefer-offline` 加本地缓存。**预期收益**：消除 MEDIUM 安全告警，防止上游包被篡改后影响本地环境。
3. **当前问题**：`commit-msg.md` 使用 `pbcopy` 复制到剪贴板，macOS 独有命令在 Linux/Windows 上报错。**改进做法**：加平台检测：`command -v pbcopy &>/dev/null && pbcopy || xclip -selection clipboard || echo "$MSG"`。**预期收益**：命令在跨平台环境（如 WSL、Linux 开发机）可正常使用。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

整体得分 **36/100**（分数下限强制：26 个命令全部缺 frontmatter，每个命令分数上限被强制压至 45）。

| 文件类型 | 当时分数区间 | 主要问题 |
|---|---|---|
| 26 个 commands/*.md | 强制上限 45 | 无 frontmatter（name/description/allowed-tools 全缺）|
| .claude/skills/ai/SKILL.md | 84 | 含模糊量词 |
| .claude/skills/bash/SKILL.md | 88 | 含模糊量词 |
| CLAUDE.md | 95 | 无明显问题（重定向模式设计正确）|
| plugin.json | 96 | 已知 MCP 版本未固定问题 |

### 3.2 当时值得借鉴的模式
1. **CLAUDE.md 重定向至 AGENTS.md** → 根本原因：AI 工具生态多元化，不同工具读取不同入口文件（Claude Code 读 CLAUDE.md，GitHub Copilot/Cursor 读 AGENTS.md），重定向使同一份文档被多个工具发现。→ 借鉴：在需要兼容多套 AI 工具的项目中，保持 CLAUDE.md 轻量，实质文档写在 AGENTS.md。
2. **plugin.json 96 分的高质量清单** → frontmatter 完整，命令清单结构清晰，MCP 集成字段格式正确。说明作者对 `plugin.json` 规范了解深入，问题出在忽视了 command 文件层面的 frontmatter 要求。
3. **Context7 MCP 集成的设计意识** → 主动引入实时文档服务，说明作者意识到静态知识的局限性，是面向未来维护成本的设计决策。
4. **$REPORT_BASE 约定的跨文件一致性** → `simplify.md` 和 `spike-investigation.md` 都遵循相同的报告基础约定，说明作者在高频输出命令之间维护了横向一致性。

### 3.3 当时的缺陷
1. **BUG-missing-frontmatter（HIGH 严重度）**：26 个命令文件全部无 YAML frontmatter。→ 为什么失败：没有自动化检查工具，作者在写命令时未参照 Claude Code 的 command 规范。每个命令分数强制上限 45，是本次低分的根本原因。→ 自查：我的命令文件是否每个都有 `name`、`description`、`allowed-tools` 三个字段？
2. **CC-terminology-drift（LOW 严重度）**：插件名为 "atm"，但命令调用方式写成 `/atm-scaffold` 风格（连字符），应为 `/atm:scaffold`（冒号）。→ 为什么失败：命令文件内部说明和 plugin.json 的命名约定不一致，可能导致用户混淆。
3. **SEC-unpinned-npm-install（MEDIUM 安全风险）**：`plugin.json` 中 `"args": ["-y", "@upstash/context7-mcp"]` 无版本固定，每次 `npx -y` 运行都可能拉取不同版本。→ 为什么失败：动态拉取是 npx 的便捷用法，但在安全敏感场景下等同于每次执行时信任未知代码。→ 自查：我的项目有没有类似的 `npx -y <package>` 或 `curl | sh` 模式？
4. **空 $ARGUMENTS 无守卫**：6 个命令在 $ARGUMENTS 为空时无处理逻辑，会导致命令静默执行或产生无意义输出。
5. **pbcopy macOS 绑定**：`commit-msg.md` 直接调用 `pbcopy`，Linux/Windows 用户会报错。

### 3.4 当时的优化机会
1. 批量为 26 个命令添加最小 frontmatter（可用脚本批量生成模板，人工补充 description）
2. 将 `@upstash/context7-mcp` 固定到具体版本号
3. 为 6 个无守卫命令添加 `$ARGUMENTS` 空值检查
4. 修复 `commit-msg.md` 的 macOS-only `pbcopy` 调用
5. 统一命令调用符号（连字符 → 冒号）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 26 个命令缺 frontmatter | `grep -rL "^---" commands/` | **仍存在**（全部 26 个文件确认无 frontmatter）| 3 个月内零修复，作者可能不知道 frontmatter 对 Claude Code 的意义 |
| MCP 版本未固定 | `grep "@upstash/context7-mcp" .claude-plugin/plugin.json` | **仍存在**（`"-y", "@upstash/context7-mcp"` 无版本号）| 安全风险持续存在 |
| CC-terminology-drift | `grep -r "/atm-" commands/` | **仍存在**（命令说明中仍使用连字符风格）| 低优先级问题，未引起关注 |

### 4.2 架构演进
对比 2026-04-29 快照与当前 HEAD：plugin.json 版本号、文件结构、命令数量（26）均无变化。仓库处于维护静止状态，没有新增命令，也没有删除任何文件。这一"冻结"状态有两种可能解读：一是作者认为工具集已满足个人需求，无需迭代；二是维护热情消退，仓库进入无人维护状态。

### 4.3 新增的可学习模式
当前 HEAD 与 audit 时状态完全一致，无新增模式。但这本身也是一个教训：**没有自动化质量反馈（如 pre-commit hook 或 CI 检查）的项目，缺陷在发现后往往也不会被修复**，因为作者感知不到压力。

---

## 五、校准

### 5.1 我已经在做对的
1. **MarkQWu/bureau 的命令 description 字段**：bureau 的所有命令都有 `description` 和 `argument-hint`，比 aaronmaturen/claude-plugin 的零 frontmatter 前进了一步，说明对 frontmatter 规范有基本认知。
2. **MarkQWu/gstack 的 SKILL.md 完整 frontmatter**：gstack 的 SKILL.md 文件有正确的 frontmatter，说明 skill 层的规范意识是存在的。
3. **本案例 CLAUDE.md 重定向模式**：若我的项目也面向多 AI 工具生态，保持 CLAUDE.md 轻量、内容写在 AGENTS.md 是值得复用的设计。

### 5.2 挑战 / 验证
本案例挑战了一个假设："知道插件规范的开发者会把 frontmatter 写完整"。实际情况是：**plugin.json 几乎满分（96）的作者，commands 文件却全部缺 frontmatter**——说明对清单文件的规范认知和对命令文件的规范认知是两个独立的知识点，一个掌握了并不代表另一个也掌握了。对我的启示：检查工具必须覆盖所有文件类型，不能因为某类文件质量高就认为全局都没问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 command 文件是否包含完整的三字段 frontmatter
for f in $(find /home/user -name "*.md" -path "*/commands/*" 2>/dev/null); do
  missing=""
  grep -q "^name:" "$f" || missing="$missing name"
  grep -q "^description:" "$f" || missing="$missing description"
  grep -q "^allowed-tools:" "$f" || missing="$missing allowed-tools"
  [ -n "$missing" ] && echo "MISSING [$missing ]: $f"
done
# 命中后：根据命令实际使用的工具逐一补全 frontmatter
```

```bash
# 检查项目中是否有未固定版本的 npx 调用（安全风险）
grep -rn '"npx"' /home/user --include="*.json" 2>/dev/null | grep '"-y"' | grep -v '@[0-9]'
# 命中后：将 @package 改为 @package@x.y.z 固定版本
```

```bash
# 检查 skill 文件中的模糊量词
grep -rn -E '\b(sometimes|usually|often|typically|generally|appropriate|relevant)\b' \
  /home/user --include="SKILL.md" 2>/dev/null
# 命中后：判断每处是否能改为"if X then Y"的条件句
```

```bash
# 检查 command 文件内部的调用格式是否使用冒号而非连字符
grep -rn '/[a-z]\+-[a-z]' /home/user --include="*.md" -path "*/commands/*" 2>/dev/null
# 命中后：将 /plugin-command 改为 /plugin:command
```

```bash
# 检查使用 pbcopy 等平台专属命令的文件
grep -rn 'pbcopy\|pbpaste\|clip\.exe' /home/user --include="*.md" 2>/dev/null
# 命中后：添加跨平台检测逻辑
```

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/bureau 的 11 个命令补全 `name` 和 `allowed-tools` 字段
   - **为何可行**：bureau 命令已有 `description` 和 `argument-hint`，只缺 `name` 和 `allowed-tools`，补全成本低，收益高（命令可被正确注册）
   - **第一步**：`grep -rL "^name:" /path/to/bureau/commands/` 列出所有缺少 name 的文件，逐一加上 `name: bureau:<command-stem>`——每个文件约 5 分钟，全部完成约 1 小时

2. **想法**：为任何使用 `npx -y <package>` 的 MCP 配置固定版本
   - **为何可行**：Context7 有语义化版本，固定 minor 版本（如 `@1`）可在获得补丁更新的同时防止破坏性变更或供应链攻击
   - **第一步**：`grep -rn 'npx.*-y' ~/.claude --include="*.json"` 检查本地所有 Claude 配置中的 MCP 声明，对每个无版本号的包，查阅其 npm 主页获取当前稳定版本号后固定——约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：对 MarkQWu/bureau 的直接检查结果

### 8.1 目的对齐度

- **本案例 aaronmaturen/claude-plugin 的核心目的**：全栈开发工作流工具集，26 个命令覆盖"调研→实现→审查→发布"链路，搭配 Context7 MCP 实时文档
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同为 Claude Code 插件；同有 commands + skills 分层；同为个人工作流工具 | bureau 专注知识管理，aaronmaturen 专注全栈开发；bureau 无 MCP 集成 | 高 |
| MarkQWu/gstack | 中 | 同为 Claude Code 工具集；同有多命令设计 | gstack 基于 Garry Tan 风格的管理层工具，非开发工具 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | bureau 命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 `name` 字段 | `grep -rL "^name:" bureau/commands/` | **全部 11 个命令缺 name 字段**（description✅、argument-hint✅，但 name❌）| 中（命令无法被正确识别） |
| Command 缺 `allowed-tools` | `grep -rL "allowed-tools" bureau/commands/` | **全部 11 个命令缺 allowed-tools**（compile.md、crew.md 等全部未声明）| 中（工具权限不透明）|
| 模糊量词 | `grep -rn "relevant\|sometimes\|usually" bureau/` | 待验证，bureau skills 0 个模糊量词（已知）| 低 |

**命中后的具体行动建议**：
- `bureau/commands/compile.md` → 加 `name: bureau:compile`、`allowed-tools: Read Write Bash` → 5 分钟
- `bureau/commands/note.md` → 加 `name: bureau:note`、`allowed-tools: Read Write` → 5 分钟
- 其余 9 个命令类似处理，总计约 1 小时

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：MCP 动态文档注入
   - **本案例做法**：`plugin.json` 的 `mcpServers` 字段声明 Context7，命令运行时自动获取框架最新文档，无需手动维护 skill 内容
   - **我的项目现状**：bureau 和 gstack 均无 MCP 集成，知识全靠静态 SKILL.md 维护，文档过时需手动更新
   - **如何借鉴**：对于 bureau 中需要引用外部 API 文档的查询命令，评估是否值得引入 Context7 MCP，使 AI 能实时查阅被查询工具的最新 API

2. **领域**：CLAUDE.md 重定向至 AGENTS.md 的间接层设计
   - **本案例做法**：CLAUDE.md 极简（一行重定向），实质文档在 AGENTS.md，兼容多 AI 工具生态
   - **我的项目现状**：bureau 的 CLAUDE.md 可能直接包含大量内容
   - **如何借鉴**：若 bureau 面向多 AI 工具，将实质文档迁移到 AGENTS.md，CLAUDE.md 改为重定向，维护单一文档源头

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：frontmatter 部分完整性（description 和 argument-hint）
- **bureau 的做法**：所有 11 个命令文件都有 `description` 和 `argument-hint` 字段，说明作者了解 Claude Code 的命令元数据规范
- **本案例做法（弱在哪）**：aaronmaturen/claude-plugin 的 26 个命令**完全没有任何 frontmatter**，连 description 也没有——这是更彻底的规范缺失
- **意义**：bureau 的 frontmatter 缺陷是"最后一公里"问题（缺 name 和 allowed-tools），而本案例是"起点"问题（零 frontmatter）。bureau 的部分合规反映了作者对规范有初步认知，只是未完全跟进最新要求。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，用于声明文件的元数据。Claude Code 读取 command 文件时先解析 frontmatter 中的 `name`、`description`、`allowed-tools` 字段来注册命令。缺少 frontmatter 会导致 NLPM 强制将该命令的分数上限压至 45。

### <a name="allowed-tools"></a>allowed-tools
> Command frontmatter 中的字段，声明该命令需要调用哪些工具（如 `Read`、`Write`、`Bash`）。未声明的工具在受限环境中不会被自动授权，用户在安装时也无法预先批准对应权限。

### <a name="unpinned-npm"></a>未固定版本的 npx（unpinned npx）
> 使用 `npx -y @package` 而非 `npx -y @package@x.y.z` 的调用方式。每次运行时都会拉取 npm registry 上的最新版本，存在供应链攻击风险——若上游包被恶意发布者控制，可在用户无感知的情况下执行任意代码。NLPM 将此标记为 MEDIUM 安全风险。

### <a name="context7-mcp"></a>Context7 MCP
> Upstash 提供的 MCP 服务，用于实时查询第三方框架（如 Angular、Django、React 等）的官方文档。通过 `npx @upstash/context7-mcp` 启动，Claude Code 命令可在对话中直接调用，获取版本精确的 API 参考。

### <a name="score-ceiling"></a>分数上限（score ceiling）
> NLPM 评分体系中的强制罚分机制：当某类必填字段完全缺失时（如 frontmatter），不仅扣减固定分数，还将该文件的最终分数强制压至某个上限（如 45/100），无论内容质量多高。用于区分"格式不规范"和"内容不高质量"两类问题。

### <a name="pbcopy"></a>pbcopy（macOS 专属剪贴板命令）
> macOS 系统内置的命令行工具，将标准输入内容复制到系统剪贴板。在 Linux 上无此命令（对应工具为 `xclip` 或 `xsel`），在 Windows 上为 `clip.exe`。跨平台 shell 脚本中直接调用 `pbcopy` 会导致非 macOS 用户报 "command not found" 错误。
