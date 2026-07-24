# memvid/claude-brain — 学习案例

**仓库**：https://github.com/memvid/claude-brain
**Stars**：465 | **来源**：upstream（近 500 星，星数门槛临界，优先处理）
**Audit 日期**：2026-04-30（历史快照）| **生成日期**：2026-07-24（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `security-gate`, `manifest-discipline`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
memvid/claude-brain 是一个给 Claude Code 提供**持久化记忆能力**的插件，核心卖点是「把所有记忆压缩进一个 `.claude/mind.mv2` 二进制文件，可以 `git commit`、`scp`、共享」。底层是 [Rust 写的 memvid SDK](https://github.com/memvid/memvid)，NL 层（`.md` 命令 + hooks）只是入口表皮。

关键事实：
- 2026 年 4 月审计，465 星，独立作者维护
- 安装方式：`claude plugin install memvid:claude-brain`
- 核心接口：`/mind:ask`、`/mind:search`、`/mind:recent`、`/mind:stats`
- 运行时架构：SessionStart 钩子自动加载记忆 → PostToolUse 实时捕获 → Stop 钩子持久化

### 1.2 架构剖析

```
claude-brain/
├── commands/          # NL 表皮：ask.md / search.md / recent.md / stats.md
├── hooks/
│   └── hooks.json     # 声明 SessionStart / PostToolUse / Stop 钩子
├── skills/
│   ├── memory/SKILL.md    # ⚠ 与 mind/SKILL.md 内容几乎相同（bug）
│   └── mind/SKILL.md
├── src/               # TypeScript 源码
│   └── hooks/         # smart-install.ts / session-start.ts / post-tool-use.ts / stop.ts
├── dist/              # 编译产物（JS）—— 实际运行的 hooks
├── .claude-plugin/
│   └── plugin.json    # manifest，100/100
└── package.json       # @memvid/sdk 依赖（caret 未固定）
```

- **文件类型分布**：4 个 command（平均 93 分）、2 个 SKILL.md（93 分，互为重复）、3 个 hooks.json、4 个 TypeScript hook 源文件
- **编排关系**：4 个命令各自独立调用 `find.js`/`ask.js`/`stats.js`/`timeline.js`，钩子层自动捕获，命令层被动查询。无 router，平列式。
- **跨件契约**：命令调用 `node "${CLAUDE_PLUGIN_ROOT}/dist/scripts/*.js"` 硬编码绝对路径，依赖 `CLAUDE_PLUGIN_ROOT` 环境变量正确注入。

### 1.3 设计思路 / 方法论
- **核心哲学**：「一个文件打包所有记忆」——离线、可移植、可版本化。拒绝数据库依赖（no SQLite, no ChromaDB）。
- **解决问题**：AI 会话跨天断记忆，团队成员无法共享 AI 上下文。
- **Trade-off**：选择 Rust 原生二进制（性能 + 跨平台）而非 Python/JS，代价是 TypeScript 薄薄的 NL 层要做显式的 `npm install` 运行时检查，带来安全隐患。
- **认知模型**：AI agent 的记忆应该像 git 仓库一样——小、可追踪、可合并。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「NL 表皮 + 原生二进制核心」**

Markdown 命令/钩子 = 入口协议；真正的计算由 TypeScript/Rust 层完成，Claude 只是转发参数和展示结果。

特征清单：
- TypeScript 作为 NL 层与 Rust SDK 之间的胶水层
- 命令文件极简（最多 20 行），逻辑全在 `dist/scripts/*.js`
- `plugin.json` 声明所有命令路径，manifest 满分（100）
- hooks.json 驱动全生命周期自动化（安装、捕获、持久化）
- 不依赖 Claude 做计算，Claude 只做展示

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要高性能向量搜索的记忆系统 | ✅ 高度适用 | Rust 核心性能优越 |
| 纯 NL 提示词工具（无需持久化） | ❌ 过度工程 | 用 SKILL.md 即可 |
| 多人共享 AI 上下文 | ✅ 适用 | `.mv2` 文件可 git push |
| 企业网络隔离环境 | ⚠️ 谨慎 | runtime npm install 有网络依赖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制核心（本案例） | memvid/claude-brain | 性能极高，可移植，离线 | 二进制依赖复杂，安全面大 |
| 纯 NL SKILL.md 链 | BayramAnnakov/claude-reflect | 零依赖，上手快 | 跨会话记忆靠 Claude 自身，不可靠 |
| MCP Server + NL 命令 | czlonkowski/n8n-skills | 功能丰富，协议标准 | MCP 安装依赖外部进程 |

### 2.4 改进空间
1. **当前问题**：`skills/memory/` 和 `skills/mind/` 内容几乎相同 **改进做法**：删除一个，只保留 `skills/mind/SKILL.md`，`plugin.json` 中相应删除重复注册 **预期收益**：消除双维护负担，得分 +0（已是 93 分，但可防未来分叉）
2. **当前问题**：`search.md` 的 `[limit]` 参数硬编码为 10 **改进做法**：改为 `node "…/find.js" "$1" "$2"` 分别传 query 和 limit **预期收益**：API 契约与文档一致
3. **当前问题**：`npm install @memvid/sdk` 在每次脚本调用时再次检查 **改进做法**：仅在 `smart-install.ts`（SessionStart）执行一次，`ask.ts`/`find.ts` 中删除重复检查 **预期收益**：消除 MEDIUM 安全发现 #2/#3

---

## 三、过去审查发现（2026-04-30 历史快照）

### 3.1 当时质量评分（NLPM）
96/100。扣分主要来自命令层（ask/search 各 -10 无空输入处理，recent/stats 各 -2 模糊量词）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/search.md | 90 | 无空输入处理；limit 参数解析 bug |
| commands/ask.md | 88 | 无空输入处理；"when applicable" |
| skills/memory/SKILL.md | 93 | `.mind.mv2` 路径错误；与 mind SKILL 重复 |
| hooks/hooks.json | 95 | Stop timeout 10s vs src 30s 不一致 |
| .claude-plugin/plugin.json | 100 | — |

### 3.2 当时值得借鉴的模式
1. **plugin.json 满分** → 所有命令路径、名称、描述均注册，manifest 零遗漏。原文：`.claude-plugin/plugin.json` 100/100。借鉴：写完新命令立刻同步 plugin.json。
2. **钩子全生命周期** → SessionStart（安装+加载）、PostToolUse（实时捕获）、Stop（持久化）三钩连贯。借鉴：设计钩子时画出「会话生命周期图」，而不是只写 PostToolUse。
3. **极简命令文件** → 命令 `.md` 只有 20 行左右，重逻辑在 dist/scripts/ 里。NL 层职责纯粹。

### 3.3 当时的缺陷
1. **`search.md` limit 参数不可用** → 文档说 `[limit]` 可选，但代码硬编码 10。用户发现参数无效但不知道原因，产生认知错误。**我有没有？** 我的 bureau 项目里 `.claude/commands/query.md` 也可能有类似隐含默认值问题，需要检查。
2. **两个 SKILL.md 内容重复** → `memory/SKILL.md` 和 `mind/SKILL.md` 只有 `name:` 字段不同。维护时改一个忘改另一个，迟早分叉。**我有没有？** bureau 和 graphify 里若有类似语义重叠的 skill，值得合并。
3. **HIGH：unquoted `${ARGUMENTS:-20}`** → `recent.md` 第 15 行未加引号，`/mind:recent $(id)` 能注入。**我有没有？** 应检查 gstack 里所有使用 `$ARGUMENTS` 的命令。

### 3.4 当时的优化机会
1. **加空输入处理**：`if [[ -z "$ARGUMENTS" ]]; then echo "Usage: /mind:ask <question>"; exit 1; fi`
2. **修复 search.md 的 limit 解析**：拆分 `$ARGUMENTS` 为 `$1` 和 `${2:-10}`
3. **统一 Stop 超时**：所有 hooks.json 的 Stop timeout 改为一致的 30s

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| search.md limit 参数硬编码为 10 | `grep -n "ARGUMENTS.*10" commands/search.md` | **持续** — 第 16 行仍是 `"$ARGUMENTS" 10` | 维护者未修复，用户期望与行为仍不匹配 |
| `.mind.mv2` 路径错误（skill 第 71 行） | `grep "\.mind\.mv2" skills/memory/SKILL.md` | **持续** — SKILL.md 第 71 行仍写 `.mind.mv2`（正确应为 `.claude/mind.mv2`） | 文档误导用户共享错误文件名 |
| memory/mind 两个 SKILL 重复 | `diff skills/memory/SKILL.md skills/mind/SKILL.md` | **持续** — 仅 `name:` 字段不同 | 双重维护负担保留 |

### 4.2 架构演进

当时审计 → 当前 HEAD 无明显目录重组。命令、钩子、skills 结构保持稳定。主要变化在 `dist/` 编译产物（TypeScript 版本迭代）。说明：**作者专注底层 SDK 演进，NL 层当作稳定接口，未因 NLPM 问题迭代**。

### 4.3 新增的可学习模式

当前 `hooks/hooks.json` 中 Stop hook timeout 从 10s 更新为 10s（未变），但 `dist/hooks/hooks.json` 保持 30s。说明 drift 问题仍存在，作者显然以 dist 为准而非 hooks/ 根目录。**可学习**：若项目有生成产物，明确标注 `hooks/` 是否为「真实运行配置」，不要同时维护两份。

---

## 五、校准

### 5.1 我已经在做对的
1. bureau 项目 plugin.json 包含所有命令声明（与本案例 manifest 满分做法一致）
2. graphify 项目命令文件也是薄壳模式，逻辑下沉到 Python
3. drama-workshop-skills 钩子配置文件单一来源（不存在 src/dist 双副本问题）

### 5.2 挑战 / 验证
本案例验证了：**命令文件极简（<25 行）是可行的架构**，不需要把所有逻辑塞进 Markdown。我之前担心「命令太简单会不会让 Claude 不知道怎么做」——事实证明只要 dist/scripts 层逻辑清晰，Claude 正确转发参数就够了。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令里是否有未引号的 $ARGUMENTS
grep -rn 'ARGUMENTS' ~/.claude/commands/*.md /tmp/my-repos/MarkQWu-*/.*claude/commands/*.md 2>/dev/null | grep -v '"$ARGUMENTS"' | grep '\$ARGUMENTS'
# 命中后：把裸的 $ARGUMENTS 改成 "$ARGUMENTS"（加双引号）
```

```bash
# 检查重复 SKILL.md（内容相同只有 name 不同）
for dir in /tmp/my-repos/MarkQWu-*/; do
  md5sum "$dir"/**/*SKILL.md 2>/dev/null | sort | awk 'seen[$1]++{print $1, $2}'
done
# 命中后：保留一个，另一个改为单行引用（`MANDATORY READ: <canonical SKILL>`）
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 `/query` 命令加空输入守护
   - **为何可行**：当前 `query.md` 直接使用 `$ARGUMENTS` 无判断，空调用会产生空查询
   - **第一步**：在 `query.md` 第一个 Bash 块前加 `if [[ -z "$ARGUMENTS" ]]; then echo "Usage: /bureau:query <question>"; exit 0; fi`（5 分钟）

2. **想法**：将 bureau 命令从参数注入改为显式 `$1`/`$2` 分割
   - **为何可行**：当前 `$ARGUMENTS` 传整串给后端脚本，多个参数解析靠后端自己 split
   - **第一步**：在 `serve.md` 中测试 `"${ARGUMENTS%% *}"` 取首参数，验证行为后推广

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 memvid/claude-brain 的核心目的**：给 Claude Code 会话提供可移植持久化记忆

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同样是「AI 会话知识持久化」，同样有 Claude Code 钩子 | bureau 是「多会话知识库」面向团队；brain 是「个人记忆文件」轻量简单 | 高 |
| MarkQWu/graphify | 低 | 同样依赖 NL 命令调用底层脚本 | graphify 专注知识图谱，无跨会话记忆设计 | 低 |
| MarkQWu/gstack | 低 | 都是 Claude Code 插件集合 | gstack 是工具集聚合，无记忆功能 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺空输入守护 | `grep -L 'if.*ARGUMENTS.*-z' .claude/commands/*.md` | bureau 命中 11/11 个命令（lint/status/serve/inspect/review/note/init/query/file-session/crew/compile 全部无守护） | 中 |
| `$ARGUMENTS` 未加引号 | `grep -n '\$ARGUMENTS[^"=]' .claude/commands/*.md` | bureau 和 gstack 部分命令有裸变量（需逐一验证） | 高 |
| SKILL.md 路径说明错误 | `grep -rn '\.mv2\|\.brain\|\.bureau' ~/.claude/skills/` | 暂未命中，bureau 的路径说明在 CLAUDE.md 里 | 低 |

**命中后的具体行动建议**：
- bureau 的 `note.md` → 第 3 行前加空输入判断 → 10 分钟可完成
- bureau 的 `query.md` → 审查 `$ARGUMENTS` 是否有引号 → 5 分钟

### 7.3 别人的更优方案

1. **领域**：hooks 全生命周期覆盖
   - **本案例做法**：SessionStart（安装+加载）、PostToolUse（实时捕获）、Stop（持久化）三钩连贯，在 `hooks/hooks.json` 里统一声明
   - **我的项目现状**：bureau 只有 PostToolUse 钩子用于捕获 Write 事件，SessionStart 和 Stop 未使用，会话开始时不恢复上下文
   - **如何借鉴**：在 bureau 的 hooks.json 里加 SessionStart 钩子，调用 `bureau init --quiet` 加载最近的 SESSION.md 快照；加 Stop 钩子调用 `bureau compile --auto` 自动归档

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：命令数量 vs 功能覆盖平衡
- **我的做法（bureau）**：11 个命令各有专门用途，比 brain 的 4 个命令覆盖了更完整的工作流（init/note/compile/review/query 等）
- **本案例做法（弱在哪）**：4 个命令偏少，没有「手动添加记忆」的 `/mind:remember` 命令，用户无法主动控制记忆内容
- **意义**：bureau 主动管理 + 被动捕获双轨制，比 brain 纯被动捕获更灵活

---

## 八、术语表

### <a name="NL表皮"></a>NL 表皮
> Markdown 格式的「壳」——用 `.md` 文件告诉 Claude 要做什么、调什么工具、返回什么格式。本身不包含真正的计算逻辑，计算由背后的 JS/Python/Rust 脚本完成。类比：NL 表皮是餐厅菜单，真正做菜的是厨房（原生代码）。

### <a name="原生二进制核心"></a>原生二进制核心
> 用编译型语言（Rust/Go/C）写出的可执行程序，不需要解释器直接运行。本案例中的 `@memvid/sdk` 底层是 Rust，性能比纯 JS 高约 10 倍，且内存占用更低。

### <a name="hooks.json"></a>hooks.json
> Claude Code 的钩子配置文件，声明在特定事件（会话开始、工具使用后、会话结束）时要运行的命令。类比：CI/CD pipeline 里的「触发器」。

### <a name="PostToolUse"></a>PostToolUse
> Claude Code 钩子事件：每次 Claude 调用工具（Write/Bash/Edit 等）之后触发。memvid 用它来自动记录 Claude 做了什么。

### <a name="manifest"></a>manifest
> 插件的「清单文件」，告诉 Claude Code 这个插件有哪些命令、技能、agents。在本案例中即 `.claude-plugin/plugin.json`。
