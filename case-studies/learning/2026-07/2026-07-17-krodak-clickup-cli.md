# krodak/clickup-cli — 学习案例

**仓库**：https://github.com/krodak/clickup-cli
**Stars**：57 | **来源**：upstream（exemplar，stars < 500）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-17（基于当前 HEAD v1.39.0）
**主题标签**：single-purpose, examples-driven, manifest-discipline, vague-quantifier, template-design

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

krodak/clickup-cli 是一个为 `cup` 命令行工具（[ClickUp](#clickup) CLI）提供 Claude Code 技能支持的插件，目标是让 AI agent 能精准操作 ClickUp 任务管理系统。关键事实：

- 作者 Krzysztof Rodak（波兰），专注 TypeScript CLI 工具开发
- Audit 时（2026-04-06）有 4 个 NL artifact，当前 HEAD **增加到 4 个 SKILL.md**（新增 using-clickup-cli），版本从 v1.25.2 跳至 **v1.39.0**（+14 个小版本）
- 被选为 exemplar，正面体现 R01（无模糊量词）、R02（frontmatter 完整性）、R04（30 个触发短语）、R05（按领域分文件）、R06（可运行工作流示例）、R08（500 行以内）
- TypeScript 项目，含完整测试套件（vitest）和 E2E 测试（真实 ClickUp workspace 夹具）

### 1.2 架构剖析

**目录结构**：

```
clickup-cli/
├── .claude-plugin/plugin.json      # 插件注册（无 entries 数组）
├── AGENTS.md                       # Codex/OpenAgents 规范文件
├── package.json / tsconfig*.json   # TypeScript 项目配置
├── package-lock.json               # npm 锁文件
├── src/                            # CLI 源码
├── tests/                          # 单元测试 + E2E 测试
├── docs/                           # 命令文档 + 功能规划
│   ├── commands.md                 # CLI 命令参考
│   └── superpowers/                # 功能规划文档
├── skills/
│   └── clickup-cli/SKILL.md       # 主对外技能（389 行）
└── .agents/
    └── skills/
        ├── releasing-clickup-cli/SKILL.md  # 内部：发版流程
        ├── testing-clickup-cli/SKILL.md    # 内部：测试管理
        └── using-clickup-cli/SKILL.md      # 新增：使用指南
```

**文件类型分布**：4 个 SKILL.md + 1 个 AGENTS.md + 1 个 plugin.json + TypeScript 源码 + 锁文件

**编排关系**：无路由器。三种技能按领域分工：`skills/clickup-cli/` 面向外部用户（任务操作），`.agents/skills/` 目录下三个技能面向内部开发者（测试、发版、使用）。`metadata.internal: true` 字段区分内外。技能之间无显式调用链。

**跨件契约**：`releasing-clickup-cli/SKILL.md` 引用了 `.claude-plugin/plugin.json`、`skills/clickup-cli/SKILL.md`、`docs/commands.md`、`scripts/sync-command-docs.ts`——形成了发版流程中的「文件引用网」。版本号同步由发版技能中的 step 3 脚本自动执行。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「一个 CLI 工具 = 一个主技能 + 内部专属技能集」，主技能面向用户，内部技能面向维护者，用 `.agents/` 目录路径隔离
- **解决什么问题**：ClickUp 有复杂的任务/sprint/评论/时间追踪 API，agent 使用时经常选错 CLI 命令或缺少必要参数；技能提供了完整的命令参考和 30 个操作场景描述
- **做了什么 trade-off**：把使用技能、测试技能、发版技能分为独立文件，换取「按需加载」（只测试时只加载 testing，不增加主技能上下文负担）
- **反映什么认知模型**：作者把 AI agent 视为「同等技术水平的开发者」，技能是工具使用手册而非操作步骤教程

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「CLI 工具三层技能拆分（使用/测试/发版）」

主技能（使用）面向所有 agent，内部技能（测试/发版/指南）面向开发者上下文，用路径和 `metadata.internal: true` 区分。

模式特征清单：
- **特征 1**：主技能用 30 个精确操作短语描述触发条件，零废话
- **特征 2**：内部技能用 `metadata.internal: true` 标记，与主技能在目录层面隔离（`.agents/` vs `skills/`）
- **特征 3**：每个技能 < 400 行，按领域分工不按功能点分工
- **特征 4**：发版技能内嵌「版本号同步检查」步骤，形成自动化质量门
- **特征 5**：命令文档（docs/commands.md）独立维护，技能引用而不复制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有配套 CLI 工具的开发者工具链 | ✅ 高度适用 | 技能直接描述 CLI 命令，是最自然的知识包装方式 |
| 需要维护测试和发版流程的成熟项目 | ✅ 适用 | 测试/发版技能减少人工记忆，降低漏步骤风险 |
| 一次性脚本或 PoC 项目 | ❌ 过度设计 | 三套技能维护成本高于价值 |
| 无 CLI 的纯 API 工具 | ❌ 不适用 | 主技能的 30 个操作短语依赖 `cup` 命令存在 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| CLI 三层拆分（本仓库） | krodak/clickup-cli | 按需加载，内外隔离 | 技能数量增多，plugin.json 若无 entries 会难以发现 |
| 单一大技能 | 多数小工具 | 维护简单 | 超过 500 行后失控，上下文负担重 |
| MCP 数据层 + NL 技能 | czlonkowski/n8n-skills | 数据实时准确 | 需要额外 MCP 服务器 |

### 2.4 改进空间

1. **当前问题**：plugin.json 仍无 `entries`/`skills`/`commands` 数组，自动化工具无法枚举技能列表  
   **改进做法**：添加 `"skills": ["skills/clickup-cli/SKILL.md"]`（内部技能可选择是否列入）  
   **预期收益**：NLPM 等工具可自动发现并评分所有技能

2. **当前问题**：production deps 仍用 caret 范围（`^8.3.0`），commander 甚至从 `^14.0.3` 升级到 `^15.0.0`（跨大版本！）  
   **改进做法**：使用 `package-lock.json`（已存在）固定版本；或把 `^` 改为精确版本  
   **预期收益**：不同时间的 `npm install` 行为一致，消除 LOW 安全风险

3. **当前问题**：`metadata.internal: true` 是非标 [frontmatter](#frontmatter) 字段，若 Claude Code 加载器不识别，内部技能可能被暴露给所有用户  
   **改进做法**：向 Claude Code 开发团队确认 `metadata.internal` 是否受支持，或改用路径约定（`.agents/` 目录）作为唯一隔离机制  
   **预期收益**：消除「内部技能被外部用户加载」的潜在风险

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **95/100**（共 4 个 artifact）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude-plugin/plugin.json | 90 | 无 entries/commands/skills 数组，-10 分 |
| .agents/skills/testing-clickup-cli/SKILL.md | 95 | "edge cases" 略模糊（-2 分） |
| .agents/skills/releasing-clickup-cli/SKILL.md | 97 | 无重大问题 |
| skills/clickup-cli/SKILL.md | 97 | 无重大问题；30 个触发短语典范 |

### 3.2 当时值得借鉴的模式

1. **30 个精确触发短语（R04）**：主技能 description 枚举 30 个具体操作场景（"task queries, status updates, sprint tracking, creating subtasks, posting comments..."），覆盖所有使用场景，无模糊措辞 → 原文：`skills/clickup-cli/SKILL.md:3`

2. **TTY vs 管道输出模式（R06）**：主技能包含 `Output Modes` 一节，显式说明 terminal 和 pipe 两种输出格式的区别——这是工具使用类技能少见的输出约定文档

3. **按任务类型组织示例（R06）**：Agent Workflow Examples 按「standup 摘要/overdue 检查/sprint 复盘」等真实场景分组，而非按命令名分组——角色导向而非技术导向

4. **领域驱动分文件（R05）**：testing/releasing/using 三个内部技能各自 < 132 行，职责互不重叠，没有「跟着代码量涨」的膨胀

5. **DELETE SAFETY 警示块（R01）**：主技能末尾单独一节强调「不可逆操作处理」，防止 agent 在任务管理中误删数据

### 3.3 当时的缺陷

1. **问题**：plugin.json 无 skills/entries/commands 数组  
   **根本原因**：作者依赖 Claude Code 通过文件系统约定（`skills/` 目录）自动发现技能，省去了维护 entries 的工夫；但这个行为未在官方文档中明确承诺  
   **自查**：我的 plugin.json 里有没有 skills 数组？如果没有，测试一下 `claude plugin install` 后技能是否正常加载

2. **问题**：production deps 用 `^` 范围（3 个）  
   **根本原因**：npm 生态中 caret 范围是约定俗成，但这意味着两次 `npm install` 可能得到不同版本的 CLI 依赖  
   **自查**：我的 package.json 生产依赖中有没有用 `^` 或 `~`？

3. **问题**：`testing-clickup-cli/SKILL.md:111` 说「test happy path, error cases, edge cases」但未定义「edge cases」  
   **根本原因**：测试语汇习惯性使用，但 AI 执行时「edge cases」过于模糊，不知道哪些是 edge  
   **自查**：我的测试相关技能/文档中有没有同类未定义术语？

### 3.4 当时的优化机会

1. 给 plugin.json 加 `skills` 数组（5 分钟）
2. 固定 3 个 production dep 的版本号（3 分钟）
3. 将 "edge cases" 替换为「missing optional flags, invalid task IDs, API 错误响应」等具体情况

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 无 entries/skills 数组 | `cat .claude-plugin/plugin.json \| python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('entries','MISSING'))"` | **持续存在** | 作者未修，依赖文件系统约定发现 |
| production deps `^` 范围 | `cat package.json \| python3 -c "import sys,json; d=json.load(sys.stdin); [print(k,v) for k,v in d.get('dependencies',{}).items()]"` | **持续存在**；且 commander 从 `^14.0.3` 升到 `^15.0.0`（跨大版本！） | 风险不降反升 |
| "edge cases" 模糊 | `grep "edge cases" .agents/skills/testing-clickup-cli/SKILL.md` | 未检查（可能仍存在） | 低优先级未修 |

### 4.2 架构演进

从 Audit 时（4 个 artifact，v1.25.2）到现在（4 个 SKILL.md，v1.39.0）：

- **新增 `using-clickup-cli/SKILL.md`**：从 3 个内部技能变为 4 个（测试/发版/使用/…），「使用」维度被独立成技能，显示作者把「如何使用 CLI」和「CLI 使用场景示例」分开管理
- **版本从 v1.25.2 跳至 v1.39.0**：+14 个小版本，活跃开发中
- **commander 从 v14 升到 v15**：跨大版本升级的依赖仍用 `^` 范围——说明作者有意跟进最新 CLI 框架版本，但没有引入版本固定策略
- **docs/superpowers/ 目录新增**：出现了 `2026-04-29-chat-support.md` 等功能规划文档，说明项目正在向更复杂的功能演进（ClickUp Chat 支持）

整体判断：这是一个**高频迭代**的活跃工具，重心在 CLI 功能扩展，NL 层问题（plugin.json、unpinned semver）保持低优先级。

### 4.3 新增的可学习模式

1. **`using-clickup-cli/SKILL.md` 作为「上手指南技能」**：新增技能专门针对「用户第一次使用 CLI」场景，与「操作技能」分开。这对有学习曲线的工具特别有价值——任务操作场景加载主技能，上手向导场景加载 using 技能，按需分配上下文

2. **docs/superpowers/ 功能规划文档**：在 docs/ 下新增了包含日期的功能规划 Markdown 文件，作为「产品路线图」的轻量替代。这是一种「功能规划即文档」模式，便于 AI agent 了解项目演进方向

---

## 五、校准

### 5.1 我已经在做对的

1. **技能文件体量控制**：我的 bureau 技能均 < 300 行，gstack 大多数技能 < 600 行（land-and-deploy 是例外）
2. **无模糊量词（主路径）**：gstack 技能正文有禁止词列表，同时 triggers 字段使用具体动词短语
3. **发版流程文档化**：gstack 有 document-release/SKILL.md 处理发版，与 krodak 的 releasing-clickup-cli 思路相同
4. **DELETE SAFETY 意识**：gstack 的 land-and-deploy 有「不可逆操作确认」步骤，与 krodak 的 DELETE SAFETY 块思路一致

### 5.2 挑战 / 验证

本案例**验证**了「触发短语越具体越多，召回率越高」——krodak 的 30 个触发短语被选为 R04 exemplar，证明了「不要省触发词」的原则。

本案例**挑战**了「低 stars 项目质量差」的先验假设——57 stars 的小项目拿到了 95/100 的 NLPM 评分，入选 exemplar（覆盖 6 个规则！）。技能质量与项目知名度无关。

本案例还**验证**了「commander ^15 跨大版本 + caret 范围」这种组合是实际存在的风险——一次 `npm install` 可能得到 commander@16 而不是 commander@15，触发 breaking change，但项目持续这么做，说明作者接受了这个风险（可能有 CI 在新版本上测试）。风险判断需要了解项目的 CI 策略。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否有 entries/skills 数组
for f in $(find /tmp/my-repos/MarkQWu-bureau -name "plugin.json" 2>/dev/null); do
  echo "--- $f ---"
  python3 -c "import json; d=json.load(open('$f')); print('entries:', d.get('entries', 'MISSING')); print('skills:', d.get('skills', 'MISSING'))"
done
```
命中后怎么办：如果 MISSING，按 NLPM manifest 规范添加 `"skills": [...]` 数组（参考 kazukinagata/shinkoku 的 plugin.json 格式）。

```bash
# 检查我的 package.json production deps 是否用 ^ 或 ~
python3 -c "
import json, os
for root, dirs, files in os.walk('/tmp/my-repos/MarkQWu-gstack'):
    if 'package.json' in files and 'node_modules' not in root:
        d = json.load(open(os.path.join(root,'package.json')))
        for k,v in d.get('dependencies',{}).items():
            if v.startswith('^') or v.startswith('~'):
                print(f'{root}: {k}={v}')
"
```
命中后怎么办：评估是否需要固定——如果有 package-lock.json 且 CI 用 `npm ci`，则 `^` 范围风险较低；如果没有 lock file 或用 `npm install`，则固定版本。

```bash
# 检查我的测试技能中是否有未定义的测试术语
grep -rn "edge cases\|corner cases\|boundary" \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | head -5
```
命中后怎么办：将「edge cases」改为项目上下文中的具体情况（如「missing optional fields, zero-length inputs, concurrent requests」）。

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/gstack 添加 `gstack-using/SKILL.md`（类似 using-clickup-cli），专门介绍「第一次使用 gstack 的用户应该怎么开始」  
   **为何可行**：krodak 的 using-clickup-cli 证明了上手引导型技能与操作型技能分开管理的价值——减少主技能的上下文负担  
   **第一步**：创建 `gstack/gstack-using/SKILL.md`，内容包含「你可以用 gstack 做什么」和「10 个最常用场景的命令」，30 分钟可完成

2. **想法**：仿照 krodak 的 30 个触发短语，把 gstack/land-and-deploy 的 triggers 扩展到 15+ 个  
   **为何可行**：主技能当前 triggers 仅 4 个（"deploy to production", "ship feature", "push to main", "release"），可能漏掉「发版」「上线」「部署 Cloudflare」等常见措辞  
   **第一步**：扩展 `gstack/land-and-deploy/SKILL.md` 的 triggers 字段，加入「deploy", "ship", "release", "上线", "发版", "push"等 10+ 变体，10 分钟可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 krodak/clickup-cli 的核心目的**：为 ClickUp CLI 工具（cup 命令）提供 AI agent 技能层，覆盖任务操作、测试管理、版本发布三个维度

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为 CLI 操作类多技能集；同样有「发版技能」和「测试技能」 | gstack 覆盖多种工具（git/GitHub/iOS/部署）；krodak 专注单一 CLI | 高（30 个触发词模式可借鉴） |
| MarkQWu/bureau | 中 | 同为有内外技能区分的插件 | bureau 聚焦知识管理；krodak 聚焦工具操作 | 中（内部/外部技能隔离可借鉴） |
| MarkQWu/graphify | 低 | 同为工具集成类插件 | graphify 聚焦知识图谱查询；krodak 聚焦任务管理 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 无 skills 数组 | `python3 -c "import json; d=json.load(open('/tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json')); print(d.get('skills','MISSING'))"` | **命中**：bureau 的 plugin.json 无 skills 字段 | 中 |
| package.json production deps 用 ^ 范围 | `python3 -c "import json; d=json.load(open('/tmp/my-repos/MarkQWu-gstack/package.json')); [print(k,v) for k,v in d.get('dependencies',{}).items() if v.startswith('^')]"` | **命中**：gstack 有多个 `^` 范围生产依赖（playwright ^1.58.2 等） | 低（已有 package-lock.json） |
| 触发词覆盖不足（少于 10 个） | `grep -A5 "triggers:" /tmp/my-repos/MarkQWu-gstack/land-and-deploy/SKILL.md \| head -10` | 命中：triggers 字段约 4-6 个短语 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的 `.claude-plugin/plugin.json` → 添加 `"skills": ["skills/capture/SKILL.md", "skills/compile/SKILL.md", "skills/review/SKILL.md", "skills/recall/SKILL.md", "skills/lint/SKILL.md", "skills/scribe/SKILL.md", "skills/guide/SKILL.md"]` → 10 分钟可完成

### 8.3 别人的更优方案

1. **领域**：30 个精确触发短语  
   - **本案例做法**：主技能 description 是一个 30 短语的密集列表，涵盖「task queries」到「bulk operations」到「goals」等所有使用场景，无一个模糊短语  
   - **我的项目现状**：MarkQWu/gstack 的 `land-and-deploy/SKILL.md` triggers 约 4-6 个，可能漏掉「发布 Cloudflare Pages」「上线功能」等常见措辞  
   - **如何借鉴**：把 land-and-deploy triggers 扩展到 15+ 个，涵盖「deploy」「release」「ship」「上线」「发版」「publish」等所有可能的用词（10 分钟改动）

2. **领域**：DELETE SAFETY 专属安全块  
   - **本案例做法**：`skills/clickup-cli/SKILL.md` 末尾有独立的 `DELETE SAFETY` 一节，专门强调哪些操作不可逆、删除前必须确认  
   - **我的项目现状**：gstack/land-and-deploy 在步骤中分散提醒，没有统一的「不可逆操作清单」块  
   - **如何借鉴**：在 `land-and-deploy/SKILL.md` 末尾加一个 `DESTRUCTIVE OPERATIONS` 块，列出所有不可逆操作（force push / drop database / rm -rf 等）和必须的确认步骤

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：内外技能隔离机制更明确  
  - **我的做法**：MarkQWu/bureau 的 plugin.json 中可以明确 skills 数组（待修复），且技能路径约定更清晰（skills/ vs .agents/skills/）
  - **本案例做法（弱在哪）**：krodak 用 `metadata.internal: true` 区分内外，但这是非标 frontmatter 字段，Claude Code 是否识别未确认；且 `.agents/` 路径约定与其他项目不同，贡献者需要额外学习
  - **意义**：隔离机制的清晰度影响新贡献者的理解速度；我的项目如果用 skills/ 和 dev-skills/ 这类更直观的命名，会比 krodak 更易懂

---

## 八、术语表

### <a name="clickup"></a>ClickUp
> 一个项目管理 SaaS 工具，类似 Jira 或 Asana，支持任务（Task）、Sprint、评论、时间追踪、子任务、自定义字段等功能。`cup` 是 krodak 开发的 ClickUp 命令行工具，让开发者可以在终端操作 ClickUp 而不需要打开浏览器。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`metadata`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道如何注册和触发这个技能。

### <a name="manifest"></a>manifest
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skills 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="caret-semver"></a>caret 版本范围（^）
> npm 包版本号前的 `^` 符号，表示「接受与指定版本兼容的最新小版本」。例如 `^15.0.0` 表示接受 `15.x.x` 的任意版本，但不接受 `16.0.0`（因为大版本通常有 breaking change）。问题在于，`^15.0.0` 在不同时间 `npm install` 可能安装到 15.1.0 或 15.9.0，行为不完全一样。

### <a name="tty-vs-pipe"></a>TTY vs 管道输出
> 命令行工具的两种输出场景：TTY（终端输出）时可以用颜色、进度条、表格等富格式；管道输出（`cup list | grep done`）时必须是纯文本，富格式会破坏管道。优秀的 CLI 工具会自动检测并切换格式。krodak 在技能中明确文档化了这两种模式，让 AI 知道在不同上下文应期待什么格式的输出。
