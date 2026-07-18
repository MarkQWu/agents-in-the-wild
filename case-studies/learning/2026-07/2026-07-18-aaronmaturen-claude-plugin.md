# aaronmaturen/claude-plugin — 学习案例

**仓库**：https://github.com/aaronmaturen/claude-plugin
**Stars**：1 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-18（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `template-design`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
ATM（aaronmaturen/claude-plugin）是一个个人 Claude Code 插件集合，内含 26 个面向全栈开发者日常工作的命令——覆盖 Angular、Django、a11y 审计、PR review、提交信息生成、项目脚手架等。2 个通用技能（bash、ai）配合这些命令使用。作者是一名使用 Angular + Django 技术栈的全栈工程师，仓库以 Claude Code 插件形式打包分发，依赖 Context7 MCP 服务器做文档查询。目前仅 1 star，规模小但覆盖面广。

### 1.2 架构剖析
- **目录结构**：
  ```
  aaronmaturen/claude-plugin/
  ├── .claude-plugin/
  │   ├── plugin.json          # 插件 manifest
  │   └── marketplace.json
  ├── .claude/
  │   └── skills/
  │       ├── ai/SKILL.md      # AI 辅助技能
  │       └── bash/SKILL.md    # Bash 脚本技能
  ├── commands/                # 26 个命令文件
  │   ├── scaffold.md
  │   ├── pr-review.md
  │   ├── a11y-audit.md
  │   ├── angular-*.md (4个)
  │   ├── django-*.md (5个)
  │   └── ... (16个其他)
  ├── CLAUDE.md → AGENTS.md   # 重定向文件
  └── AGENTS.md               # 项目说明
  ```
- **文件类型分布**：26 个 command / 2 个 SKILL.md / 1 个 CLAUDE.md / 1 个 [plugin.json](#plugin-json)
- **编排关系**：命令平列，互不依赖；commands 通过 `skills:` 隐式使用 bash 和 ai 技能，但没有在 [frontmatter](#frontmatter) 里声明
- **跨件契约**：`$REPORT_BASE` 变量在 bash skill 和部分命令里共享（scaffold、spike-investigation），其余命令输出路径不一致；Context7 MCP 至少被 4 个命令隐式依赖，但仅在 plugin.json 里声明一次

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「把我每天重复的工程任务变成一个命令」——作者将真实工作流（排查 bug、写 PR review、做可访问性审计）直接翻译成命令，强实用性
- **解决什么问题**：减少工程师在上下文切换和样板操作上的重复劳动，用命令封装领域知识（Angular best practice、Django security checklist 等）
- **做了什么 trade-off**：选择命令数量（26个）而非质量深度——每个命令都覆盖一个场景，但没有一个做到极致精炼；注重快速上线而非 manifest 合规
- **反映什么认知模型**：作者把 Claude Code 命令当「智能 shell alias」用，而非「可注册、可发现、有协议契约的 NL 程序」——这解释了为什么 [frontmatter](#frontmatter) 被忽略：在本地 shell alias 思维下，文件名就是命令名，不需要额外元数据

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「裸命令堆叠」（Bare-Command Stack）**：所有命令都是直接的 Markdown 指令文档，没有 [frontmatter](#frontmatter) 协议声明，没有分层路由，平铺在 commands/ 目录下。

模式特征清单：
- 所有文件直接以 `#` 标题开头（无 YAML 头），靠文件名做身份标识
- 技能存在但与命令的关联不显式声明
- 命令之间无交叉引用、无公共 router
- 单一 MCP 依赖（Context7）在 plugin.json 里声明，命令不知道
- CLAUDE.md 是单行重定向，配置集中在 AGENTS.md

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人本地使用、不发布 | ✅ 勉强可用 | 文件名作为命令名在自用时能找到，不需要 NL 发现 |
| 作为 Claude Code 插件发布 | ❌ 不适用 | NLPM 扫描器无法注册无 frontmatter 的命令，用户安装后命令不可见 |
| 多人协作、代码审查 | ❌ 不适用 | 没有 name/description，代码库里找不到命令用途 |
| 需要工具权限声明 | ❌ 不适用 | 缺 allowed-tools，Claude Code 无法预授权工具 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 裸命令堆叠（本仓库） | aaronmaturen/claude-plugin | 上手快，0 配置成本 | 无法作为插件被注册和发现，NLPM score 极低 |
| frontmatter 协议化命令 | agent-sh/enhance | 可注册、可发现、工具权限清晰 | 需要维护 frontmatter，有学习成本 |
| NL 表皮 + 原生工具核心 | agent-sh/agnix | 性能 + 精确验证 | 架构复杂，Rust 技术门槛高 |

### 2.4 改进空间
1. **当前问题**：26 个命令全无 frontmatter，不可注册 **改进做法**：用脚本批量生成 frontmatter 骨架（`name: atm:<slug>`，`description: <h1 text 改写为 trigger phrase>`），然后逐文件精细化 **预期收益**：NLPM score 从 36 提升到约 75+
2. **当前问题**：Context7 MCP 使用 `npx -y @upstash/context7-mcp`（不固定版本）**改进做法**：改为 `@upstash/context7-mcp@X.Y.Z` **预期收益**：消除供应链风险，每次启动版本固定
3. **当前问题**：专家型命令（a11y-expert、angular-expert）只有人设声明，无流程步骤 **改进做法**：加入 3-5 步骤的结构化检查清单 **预期收益**：可重复性提升，减少模型随机输出

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 **36/100**（均值）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/spike-investigation.md | 15 | 无 frontmatter（-55 基础扣分）+ 空参数检查缺失（-10）+ 模糊量词 ≥10 个（-20 封顶）|
| commands/feature-investigation.md | 19 | 无 frontmatter + 空参数检查缺失 + 8 个模糊量词 |
| commands/*.md (大多数) | 25-35 | 无 frontmatter 导致 -55 基础扣分 |
| .claude/skills/ai/SKILL.md | 84 | 8 个模糊量词 |
| .claude/skills/bash/SKILL.md | 88 | 6 个模糊量词（Always×3 无条件使用等）|
| .claude-plugin/plugin.json | 96 | 描述中有 2 个模糊词（Professional、comprehensive）|

### 3.2 当时值得借鉴的模式
1. **CLAUDE.md → AGENTS.md 重定向** → 统一多工具入口，bash skill 文档里明确说明这是设计意图，不是 bug → `.claude/skills/ai/SKILL.md §Claude Code Configuration` → 借鉴：在有多工具支持场景时，这种重定向是正确模式
2. **`$REPORT_BASE` 约定** → 在 bash skill 文档中定义报告输出路径变量，scaffold 和 spike-investigation 遵守 → 借鉴：建立跨命令的输出路径约定，避免文件散落各处
3. **JIRA CLI 一致性使用** → bug-investigation、feature-investigation、ticket-explainer 三个命令用相同的 `jira issue view` 模式 → 借鉴：同类命令共享工具调用模式

### 3.3 当时的缺陷
1. **全部命令缺 frontmatter** → 为什么会失败：Claude Code 插件扫描器根据 frontmatter 中的 `name` 字段注册命令；没有 frontmatter 等于「这个命令对插件系统不存在」。作者以「本地使用」心智写代码，忽略了插件的注册协议。自查：我的 gstack 每个 SKILL.md 都有 frontmatter（name、description、allowed-tools 齐备），这点我做对了。
2. **空参数无防护** → 为什么失败：`$ARGUMENTS` 未检查直接使用，用户调用 `/atm:pr-review` 不带参数时，模型会困惑或产生无意义输出。自查：我的命令是否都有空参检查？（待自检）
3. **MCP 版本未固定** → 为什么失败：`npx -y @upstash/context7-mcp` 每次启动都安装最新版，供应链攻击或 breaking change 版本会静默影响所有用户。自查：我的 bureau 和 graphify 里有 MCP 配置需要检查版本固定情况。

### 3.4 当时的优化机会
1. 批量添加 frontmatter（一次 PR 可将 mean score 从 36 提升到约 75）
2. 为专家型命令（a11y-expert、angular-expert、django-expert）补充结构化步骤
3. 统一空参数检查模式并提取为 bash skill 里的可复用 snippet

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 26/26 命令无 frontmatter | `head -3 commands/*.md \| grep -c "^---"` → 结果 0 | **仍存在**：2026-07-18 HEAD，26/26 命令仍无 frontmatter | PR 未被合并；作者可能不认为这是优先问题 |
| Context7 MCP 未固定版本 | `cat .claude-plugin/plugin.json \| grep context7` → `npx -y @upstash/context7-mcp` | **仍存在**：仍无版本号 | 供应链风险持续暴露 |
| 空参数无防护 | `grep -L "ARGUMENTS.*-z\|test.*ARGUMENTS" commands/*.md` | **仍存在**：6 个命令无空参检查 | 用户体验问题 |

### 4.2 架构演进
2026-04-29 Audit 时 30 个 artifacts，当前 HEAD 仍是相同结构。没有发现文件增减或目录重组迹象。结论：该仓库自 audit 后基本没有更新活动，是一个「功能完整但未维护」的个人工具库。

### 4.3 新增的可学习模式
暂无——当前 HEAD 与 Audit 时状态几乎相同，未发现新增设计模式。

---

## 五、校准

### 5.1 我已经在做对的
1. **frontmatter 协议化**：gstack 每个 SKILL.md 都有 name、description、allowed-tools、triggers 字段，注册协议完整
2. **$REPORT_BASE 类约定**：我的 bureau 有类似的 `$BUREAU_DIR` 约定统一文件输出路径
3. **CLAUDE.md 单职责**：不在 CLAUDE.md 里堆砌所有文档

### 5.2 挑战 / 验证
- **挑战**：这个案例加深了「frontmatter 是命令的注册证，不是可选装饰」的认知。仓库功能上完整，代码逻辑也对，但因缺少这层元数据协议，对外完全不可发现——这是 NL programming 里「写了代码但没有 API 声明」的典型错误
- **验证**：我之前犹豫「是不是每个 skill 都需要 triggers 字段？」——本案例验证：至少 name 和 description 是硬性要求，缺一个命令就不可注册

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 frontmatter
find ~/.claude -name "*.md" -path "*/commands/*" | while read f; do
  if ! head -3 "$f" | grep -q "^---"; then
    echo "缺 frontmatter: $f"
  fi
done
# 命中后：补 frontmatter，至少加 name 和 description

# 检查我的 MCP 配置是否有版本固定
grep -rn "npx.*-y\|npx.*@[^0-9]" ~/.claude ~/.claude-plugin 2>/dev/null
# 命中后：把 @package 改为 @package@X.Y.Z
```

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau 的 MCP 依赖添加版本固定
   - **为何可行**：bureau 可能有 MCP 配置，与本案例同类风险
   - **第一步**：检查 bureau/.claude-plugin/plugin.json，找到 mcpServers，确认是否有 npx -y，5 分钟完成

2. **想法**：给 gstack 写一个「frontmatter 合规性检查脚本」
   - **为何可行**：gstack 有 20+ skills，随着新增容易忘加 frontmatter
   - **第一步**：在 gstack/scripts/ 或 gstack/health/ 新建 check-frontmatter.sh，遍历 */SKILL.md 检查 name/description 是否存在，10 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 aaronmaturen/claude-plugin 的核心目的**：把工程师日常重复任务（PR review、bug 排查、可访问性审计）封装成 Claude Code 命令

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是命令/skill 集合，都面向工程师日常任务 | gstack 更系统化（有 frontmatter 协议、版本管理），ATM 更随意 | 高 |
| MarkQWu/bureau | 低 | 无 | bureau 是知识库工具，ATM 是工程辅助工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令文件无 frontmatter | `find /tmp/my-repos/MarkQWu-gstack -name "*.md" -path "*/commands/*"` | 未命中：gstack 使用 SKILL.md 而非 commands 目录，且所有 SKILL.md 有 frontmatter | 低 |
| MCP 未固定版本 | `grep -rn "npx.*-y" /tmp/my-repos/MarkQWu-*` | 未在 my-repos 中发现 npx -y 模式 | 低 |
| 空参数无防护 | `grep -L "ARGUMENTS" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | 部分 SKILL.md 接收参数，需手动确认是否有防护 | 中 |

**命中后的具体行动建议**：暂无高优先级命中，但建议在 gstack 新增 skill 时检查 argument-hint 字段是否配套。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：`$REPORT_BASE` 输出路径约定
   - **本案例做法**：bash skill 定义 `$REPORT_BASE` 变量，scaffold 和 spike-investigation 遵循
   - **我的项目现状**：gstack 各 skill 输出路径不统一（部分写相对路径，部分写 `$HOME/.claude/...`）
   - **如何借鉴**：在 gstack/CLAUDE.md 或 gstack 顶层 SKILL.md 里声明 `$GSTACK_REPORT_DIR`，所有 skill 写报告时引用此变量

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：frontmatter 协议化执行
- **我的做法**（gstack）：每个 SKILL.md 有完整的 name、description、triggers、allowed-tools、version
- **本案例做法**（弱在哪）：0 frontmatter，命令不可注册
- **意义**：如果此仓库的作者来看我的代码，frontmatter 规范是可以作为学习材料的亮点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读取命令或 SKILL 文件时先解析 frontmatter 才能知道这个文件怎么注册和调用。没有 frontmatter 的命令文件，Claude Code 无法将其注册为可用命令。

### <a name="plugin-json"></a>plugin.json
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些命令、技能、代理的路径，以及依赖哪些 MCP 服务器。如果清单里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="manifest"></a>manifest
> 同 plugin.json——项目的「目录总表」，列出所有组件的位置和元数据。

### <a name="MCP"></a>MCP 服务器
> Model Context Protocol Server，是 Claude Code 可以调用的外部服务，为模型提供额外工具或文档查询能力。Context7 MCP 服务器提供库文档查询功能。
