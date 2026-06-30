# SukinShetty/Nemp-memory — 学习案例

**仓库**：https://github.com/SukinShetty/Nemp-memory
**Stars**：⭐95 | **来源**：xiaolai upstream（NLPM 官方 Exemplar，exemplifies R04/R08/R14/R15/R16/R17/R18/R30）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-30（目标仓库 clone 受代理限制，有 exemplar 文件）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `manifest-discipline`, `fallback-chain`

无 xiaolai 案例交叉引用。NLPM 审计系统已将此仓库发布为教学 Exemplar（exemplar_path: `auditor/exemplars/SukinShetty-Nemp-memory.md`）。

---

## 一、理解（基于审计快照 + Exemplar 文件）

### 1.1 仓库概览
Nemp-memory 是一个 **本地持久化记忆插件**，让 Claude Code 在会话之间记住工作上下文。定位："零云端、纯 JSON、跨会话保留记忆"。用户通过 24 个 `/nemp:*` 命令管理记忆（保存、召回、导出、衰减、健康检查等），数据存储在项目根目录的 `.nemp/` 目录中。

关键事实：
- 24 个 command（`.md` 文件）+ 1 个主技能 SKILL.md + 1 个 hook（PostToolUse）
- 无云端依赖，数据完全本地化（普通 JSON 文件）
- 含 Pro 版扩展（`/nemp-pro:export`），支持导出到 Codex/Cursor/Windsurf
- 被 NLPM 评选为 Exemplar，在 R04、R08、R14、R15 等规则上树立了标杆
- 审计评分：92/100（良好），主要扣分来自命令缺少 `allowed-tools` 字段

### 1.2 架构剖析

**目录结构**：
```
Nemp-memory/
├── .claude-plugin/
│   ├── plugin.json          ← manifest（version: 0.3.0）
│   ├── marketplace.json     ← 市场展示（version: 0.1.0 ← BUG：版本不同步）
│   └── hooks/
│       ├── hooks.json       ← PostToolUse hook，匹配 Edit|Write|Bash
│       └── post-tool.md     ← hook 行为定义（Markdown 形式）
├── commands/                ← 24 个命令文件
│   ├── init.md              ← 初始化
│   ├── save.md              ← 保存记忆
│   ├── recall.md            ← 召回记忆
│   ├── health.md            ← 健康检查（10 步）
│   ├── decay.md             ← 记忆衰减
│   ├── suggest.md           ← AI 建议（模糊词最密集 -20 扣）
│   ├── activate.md          ← Pro 版激活
│   ├── nemp-pro-export.md   ← Pro 版导出
│   └── ...（其余 16 个命令）
├── skills/
│   └── nemp-memory/SKILL.md ← 主技能（含关键词展开表）
├── SKILL.md                 ← 根 SKILL（frontmatter 有效，body 空 = BUG）
└── CLAUDE.md                ← 仓库说明
```

**文件类型分布**：2 个 SKILL.md / 0 个 agent / 24 个 command / 1 个 hook-config + 1 个 hook-body。

**编排关系**：Command 驱动，用户通过 `/nemp:*` 主动触发。Hook 在后台自动监听写操作，自动同步记忆。SKILL.md 提供上下文，在模型遇到记忆相关请求时自动加载。

**跨件契约**：`commands/activate.md` 在激活成功后输出格式化的确认消息，其中包含后续命令列表——但这里有 BUG：确认消息引用了错误的命令命名空间（见 §3.3）。Plugin.json 与 marketplace.json 版本字段不同步（0.3.0 vs 0.1.0）。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**"零云端、完全可控"** — 记忆数据不离开本地，不依赖外部 API，用户完全掌控数据。这反映了对开发者隐私意识的精准把握。
- **解决什么问题**：Claude Code 会话之间上下文不持续——每次新会话需要重新解释项目背景、架构决策、约定等。Nemp 通过结构化 JSON 存储这些信息，在会话开始时加载。
- **Trade-off**：本地存储 vs 云端同步 — 本地意味着无跨设备，但获得了零延迟读写和隐私保障。这对开发者比对团队更适合。
- **认知模型**：把"AI 记忆"视为**数据库条目**而非"自然语言历史"，通过关键词展开（keyword expansion）解决语义匹配问题（见 §3.2 示例）。这是工程思维对自然语言问题的巧妙解答。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：状态型命令插件（Stateful Command Plugin）**

关键特征：
- **命令是状态机的操作**：save/recall/decay/export 对应状态机的 CRUD 操作
- **Hook 作为副作用触发器**：监听工具调用 → 自动更新状态
- **分离"存储"与"展示"**：`.nemp/memories.json` 是持久化，commands 是交互层
- **Pro 版通过命名空间分离**：`/nemp:*` 是免费版，`/nemp-pro:*` 是 Pro 版

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨会话状态保留（记忆、日志、配置） | ✅ 高度适用 | 这是核心设计目标 |
| 实时操作（文件编辑、代码生成） | ❌ 不适用 | Command 插件适合元层面操作，不是主任务流 |
| 单用户单机开发环境 | ✅ 高度适用 | 本地 JSON 的简洁性在单机场景下是优势 |
| 团队协作（多设备同步） | ❌ 不适用 | 本地存储无法跨设备同步 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 状态型命令插件（本仓库） | Nemp-memory | 无外部依赖，完全可控，隐私安全 | 无跨设备，命令集大（24 个）学习成本 |
| 云端记忆服务 | mem0ai/mem0 | 跨设备，语义搜索，自动组织 | 依赖网络，有隐私顾虑，API 调用有成本 |
| 会话内上下文（无持久化） | 大多数 skills | 零摩擦，无需安装 | 会话结束即清空，不能跨会话 |

### 2.4 改进空间
1. **当前问题**：24 个命令全部缺少 `allowed-tools` 字段，Bash 和 Write 工具调用无声明。**改进做法**：给每个 command 的 frontmatter 加 `allowed-tools: [Bash, Read, Write]`（按实际需要）。**预期收益**：NLPM 分数从 92 上升约 5 点，安全姿态改善。
2. **当前问题**：marketplace.json 版本与 plugin.json 不同步（0.1.0 vs 0.3.0）。**改进做法**：在 `git commit` 或 CI 中加版本一致性检查。**预期收益**：消除市场展示版本混乱，安装时行为一致。
3. **当前问题**：`commands/suggest.md` 模糊词密度最高（>10 个），导致 -20 惩罚。**改进做法**：把"intelligent suggestions"改为"根据 `.nemp/memories.json` 中的最新 5 条记录生成具体建议"。**预期收益**：分数从 75 → 约 90。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 得分 **92/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/suggest.md` | 75 | 模糊词 >10（-20 上限）+ 无 allowed-tools（-5） |
| `commands/context.md` | 81 | 模糊词 7 个（-14）+ 无 allowed-tools（-5） |
| CLAUDE.md | 85 | 文件数 stale（20 vs 实际 24） |
| `SKILL.md`（根） | 90 | body 为空（-10） |
| 大多数 commands | 95 | 仅 -5（无 allowed-tools） |

### 3.2 当时值得借鉴的模式

1. **描述即触发器（R04 Exemplar）** → `skills/nemp-memory/SKILL.md` 的 description 字段包含 5 个独立触发场景，每个都对应用户的真实行为动作。为什么好：模型遇到"新会话开始""用户说记住这个""做架构决策"都会自动加载这个 skill。原文：`"Use when starting a new session, when the user mentions remembering something, when you need project context, when making architecture decisions, or when working with other agents on the same project."`。**借鉴**：description 不是功能描述，是触发器列表，列出 3-5 个具体用户行为。

2. **关键词展开表（R08 Exemplar）** → skill body 中有一张精确的同义词扩展表，把"auth"扩展为 authentication/login/session/jwt/token/oauth 等。这让模糊的用户查询命中精确的记忆条目。**原文示例**：`auth -> authentication, login, session, jwt, token, oauth`。**借鉴**：在知识检索型 skill 中，把领域同义词制成展开表，而非指望模型自己发挥。

3. **10 步结构化命令（R14 Exemplar）** → `commands/health.md` 有 10 个编号步骤，步骤内有字母子步骤（3a-3f），Step 8 是强制中止门（如果 Step 1 失败则停止）。这让健康检查的执行顺序完全可预测。**借鉴**：多步操作一律编号，中止条件明确标注"Stop here"。

4. **空输入处理（R15 Exemplar）** → `commands/activate.md` 在 Step 1 检查参数是否为空，立即返回精确的用法说明（含格式示例 `NEMP-PRO-XXXX-XXXX`）并写"Stop."哨兵字符。**借鉴**：每个需要参数的 command，第一步就检查空输入。

5. **输出格式模板（R17/R18 Exemplar）** → 每个 command 的输出格式用 emoji + Markdown 表格精确定义，包括成功、失败、部分失败三种情况。**借鉴**：command 不只写"怎么做"，还要写"输出长什么样"。

### 3.3 当时的缺陷

1. **Pro 命令命名空间混乱（B1+B2）** → `commands/export.md` 和 `commands/activate.md` 都在正文中引用 `/nemp:export --codex`（错误），实际应该是 `/nemp-pro:export --codex`。**根本原因**：Pro 功能是后来加的，在两个核心命令中更新了内容但没有系统性搜索替换所有引用。这让刚激活 Pro 的用户立刻看到错误示例，打击信心。**自查**：我的插件有没有命令在文档里引用了已经改名的命令？

2. **根 SKILL.md body 为空** → 根目录下的 `SKILL.md` 有有效 frontmatter，但 body 为空（5 行）。**根本原因**：有"存在即注册"的误解——以为 frontmatter 足够。实际上空 body 让这个 skill 加载后什么都提供不了。**自查**：我有没有 `find . -name "SKILL.md" -exec sh -c '[ $(wc -l < "$1") -lt 10 ] && echo "EMPTY: $1"' _ {} \;`？

3. **marketplace.json 版本落后两个小版本（B3）** → plugin.json v0.3.0 vs marketplace.json v0.1.0。**根本原因**：发布流程中更新 plugin.json 但忘记同步 marketplace.json 是经典"双写"失效。**自查**：我的仓库有没有多个版本声明文件？如果有，有没有检查它们是否一致？

### 3.4 当时的优化机会

1. **系统性添加 `allowed-tools`**——24 个 command 每个都需要，用脚本批量注入更高效
2. **修复 Pro 命名空间**——两处 `export.md` + `activate.md` 替换，10 分钟内可完成
3. **填写根 SKILL.md body**——即使只写 3-4 行摘要也比空体好

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

目标仓库 clone 受代理限制。以下基于 exemplar 文件（commit SHA: `5f3e48af`，2026-05-13 快照）和 findings.jsonl 推断：

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Pro 命名空间混乱（B1+B2） | grep "nemp:export --codex" commands/*.md | PR 未提交（findings 状态 false_positive=false） | **可能仍存在** |
| marketplace.json 版本落后（B3） | diff plugin.json marketplace.json 的 version 字段 | 未知 | 待验证 |
| 根 SKILL.md body 为空 | wc -l SKILL.md | 未知 | 待验证 |

### 4.2 架构演进

无法访问当前 HEAD。Exemplar 文件的 commit SHA 为 `5f3e48af`（2026-05-13），距审计（2026-04-06）约 5 周。由此推断仓库在审计后有持续更新，但核心架构（24 命令 + hook 设计）保持稳定。

### 4.3 新增的可学习模式

Exemplar 文件揭示 `commands/health.md` 的 10 步设计是当时审计快照中已存在的高质量模式，被特别选为 R14（Steps must be numbered）的正向示例。说明这个仓库在步骤化命令设计上从一开始就做对了。

---

## 五、校准

### 5.1 我已经在做对的
1. **本地存储优先**：bureau 项目也是"本地 JSON 存储 AI 会话数据"的思路，与 Nemp 的哲学一致。
2. **命令分离关注点**：save/recall/export 各自独立，单职责原则。
3. **提供明确的初始化命令**（/nemp:init）：我的 bureau 也有类似的引导流程。

### 5.2 挑战 / 验证
- **挑战**：Nemp 的 description 字段有 5 个精确触发短语，这让我重新思考我的 skill 描述是否太像"功能介绍"而不是"触发器列表"。
- **验证**：关键词展开表（keyword expansion map）的做法被验证为 R08 的最佳实践。我在 bureau 的 recall 功能中可以加类似的同义词展开。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 allowed-tools 声明
for f in commands/*.md; do
  if ! grep -q "allowed-tools" "$f" 2>/dev/null; then
    echo "缺 allowed-tools: $f"
  fi
done
# 命中后：查看该命令实际调用哪些工具，然后加到 frontmatter 的 allowed-tools 列表
```

```bash
# 检查版本文件一致性
PLUGIN_VER=$(jq -r '.version' .claude-plugin/plugin.json 2>/dev/null)
MARKET_VER=$(jq -r '.plugins[0].version' .claude-plugin/marketplace.json 2>/dev/null)
if [ "$PLUGIN_VER" != "$MARKET_VER" ]; then
  echo "版本不一致: plugin.json=$PLUGIN_VER, marketplace.json=$MARKET_VER"
fi
# 命中：把 marketplace.json 的版本改为与 plugin.json 一致
```

```bash
# 检查我的 commands 中有没有引用过时的命名空间
grep -rn ":\(export\|import\|activate\)" commands/ | grep -v "nemp-pro:"
# 命中后：确认这些是 Free 版命令还是 Pro 版，按正确命名空间修改
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 recall 命令加关键词展开表（同义词映射）
   - **为何可行**：bureau 存储的是 AI 会话决策，用户可能用不同词描述同一概念（"auth"、"登录"、"鉴权"）
   - **第一步**：在 `commands/recall.md` 中加 `## Keyword Expansion` 章节，列出 5-8 个常见同义词组（30 分钟）

2. **想法**：把 bureau 的所有 command 的 description 改为"触发器句式"
   - **为何可行**：bureau 命令目前可能用的是功能描述句式，改为"Use when..."句式可以提升自动加载准确率
   - **第一步**：逐一打开 `commands/*.md`，把 `description: "将会话导出..."` 改为 `description: "Use when the user asks to export session, review past decisions, or check session history"` （每个 5 分钟，共约 40 分钟）

---

## 七、对照我的 GitHub 仓库

> 注：用户仓库因代理限制无法 clone，以下基于 `learning/my-repos.json` 描述。

### 8.1 目的对齐度

- **本案例 SukinShetty/Nemp-memory 的核心目的**：给 Claude Code 提供跨会话本地记忆，通过结构化 JSON 存储开发上下文

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同是"把 AI 会话信息本地持久化 + 后续查询"，都提供命令行接口 | bureau 增加了"编译/审核/信任层"和 dashboard，Nemp 更轻量 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同是从历史会话中挖掘信息（决策/错误/模式） | echo-sleuth 是分析型（挖掘洞察），Nemp 是存储型（记住上下文） | 中 |
| MarkQWu/gstack | 低 | 同是 Claude Code 插件 | gstack 是角色型，Nemp 是记忆型 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 可能命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `allowed-tools` | `grep -L "allowed-tools" bureau/commands/*.md` | 高概率命中（常见遗漏） | 中 |
| 版本文件不同步 | 对比 bureau 中所有含 version 字段的 JSON 文件 | 可能存在（若有 marketplace.json） | 中 |
| 模糊量词（intelligent/smart/comprehensive） | `grep -rn "intelligent\|smart\|comprehensive" bureau/commands/` | 极有可能命中 | 中 |

**命中后的具体行动建议**：
- bureau/commands/*.md 缺 allowed-tools → 查看每个命令调用了哪些工具，加入 frontmatter，每个 5 分钟
- 模糊词 → 用具体数量或条件替换："intelligent suggestions" → "根据最近 5 次会话的决策记录生成 3 条建议"

### 8.3 别人的更优方案

1. **领域**：触发器句式 description（R04 Exemplar）
   - **本案例做法**：`description: "Use when starting a new session, when the user mentions remembering something, ..."`（5 个具体场景）
   - **我的项目现状**：bureau 的 skill description 可能更像功能介绍（"管理 AI 会话知识库"）
   - **如何借鉴**：打开 bureau 的主 SKILL.md，把 description 改为 3-5 个"Use when..."触发场景

2. **领域**：强制中止哨兵（R15 Exemplar）
   - **本案例做法**：`commands/activate.md` 在参数为空时返回用法 + "Stop." 明确终止执行
   - **我的项目现状**：bureau 命令可能在参数错误时继续执行或给出模糊错误
   - **如何借鉴**：在所有需要参数的 command 第一步加参数检查 + Stop 哨兵

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：信任层和多阶段审核
- **我的做法**（bureau）：capture → compile → review → query，引入信任层管理 AI 生成内容的可靠性
- **本案例做法**（弱在哪）：Nemp 直接存储 AI 的输出，没有人工审核层，记忆内容的质量完全依赖 AI 自我评估
- **意义**：bureau 的分层信任模型是 Nemp 没有的差异化价值，这在面向认知密集型工作时是优势

---

## 八、术语表

### <a name="allowed-tools"></a>allowed-tools
> Claude Code command 文件的 [frontmatter](#frontmatter) 字段，明确列出这个命令允许调用哪些工具（如 `Bash`、`Read`、`Write`）。不声明的工具调用会触发权限提示或被拒绝。系统性缺失 allowed-tools 是本案例最大的质量扣分项。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（name、description、allowed-tools 等）。Claude Code 读 command 文件时先解析 frontmatter 才知道如何注册和执行这个命令。

### <a name="哨兵字符"></a>哨兵字符（Stop sentinel）
> 在 AI 指令中用来明确终止执行流程的特殊词或短语。本案例中用 "Stop." 作为哨兵——模型看到这个词就知道不应该继续执行后续步骤。与普通错误信息不同，哨兵是明确的控制流指令而非提示性文字。
