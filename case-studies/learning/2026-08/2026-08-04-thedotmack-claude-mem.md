# thedotmack/claude-mem — 学习案例

**仓库**：https://github.com/thedotmack/claude-mem
**Stars**：N/A | **来源**：upstream audit
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-08-04（基于当前 HEAD v13.13.1）
**主题标签**：`security-gate`, `manifest-discipline`, `curl-pipe-bash-risk`, `nl-binary-hybrid`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

thedotmack/claude-mem 是一个为 Claude Code 提供**跨会话持久记忆**的插件系统。核心机制是把用户与 AI 的交互内容持久化到本地 SQLite 数据库，并通过 system prompt 注入的方式把历史记忆带入新会话。从 v11.0.1（audit 时）演进到 v13.13.1（当前），plugin 从 6 个 skill 扩展到 19 个 skill，新增了 cloud-sync、standup、weekly-digests 等企业级功能。

关键事实：
- 架构是典型的「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」：SKILL.md 是 NL 层，`src/` 中有 TypeScript/Bun 实现的 SQLite 服务、WebSocket worker、Electron 式查看器
- NLPM 历史得分 97/100，但安全扫描 BLOCKED（8 个 CRITICAL），是本期案例中「高 NL 质量 × 高安全风险」的典型反差样本
- `openclaw/` 子目录为 OpenClaw 网关集成提供安装指引，是安全风险的主要来源
- 设有完整的 CLAUDE.md 分层体系（每个子目录都有自己的 CLAUDE.md 约束 AI 行为）

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
thedotmack/claude-mem/
├── plugin/
│   ├── skills/              (19 skills, 扩展自 audit 时的 6 个)
│   ├── hooks/               (hooks.json + hooks.md)
│   ├── modes/               (角色模式配置)
│   ├── .claude-plugin/plugin.json
│   └── scripts/
├── openclaw/                (OpenClaw 网关安装向导 — 安全风险集中区)
│   ├── SKILL.md             (❌ 仍缺 frontmatter)
│   └── install.sh           (curl|bash 安装脚本)
├── src/                     (TypeScript 核心：SQLite 服务、CLI、Worker)
├── .claude/
│   └── commands/anti-pattern-czar.md  (❌ 仍缺 frontmatter)
├── install/public/install.sh (主安装脚本)
├── CLAUDE.md                (根级上下文约束)
└── package.json             (v13.13.1)
```

**文件类型分布**：19 个 skill / 1 个 command / N 个 hook（JS）/ 20+ 个 CLAUDE.md 上下文文件

**编排关系**：
- skill 是最小执行单元，互相独立；`make-plan` skill 输出结构化计划供其他 skill 消费
- hooks（PostToolUse、PreToolUse）自动触发记忆存储/注入，无需用户显式调用
- `knowledge-agent` 和 `pathfinder` skill 作为「记忆检索层」，为其他 skill 提供上下文
- `cloud-sync` skill 在本地 SQLite 和远端服务之间做双向同步

**跨件契约**：
- plugin.json → plugin/skills/ 路径映射（版本在 package.json 和两个 plugin.json 间必须保持同步）
- hooks.json → scripts/ 中的 JS/CJS 钩子脚本
- `CLAUDE.md` 分层：根级 → src/CLAUDE.md → src/services/CLAUDE.md → src/services/domain/CLAUDE.md，每层仅约束自己目录的行为

### 1.3 设计思路 / 方法论

**核心设计哲学**：记忆是「AI 第二大脑」——不靠用户手动整理，靠 hooks 自动捕获 + 自动注入。这是「透明持久化」模式：用户感知不到记忆系统的存在，但 AI 在每次会话里已经「记得」历史。

**解决什么问题**：Claude Code 是无状态的——每个新会话从零开始，不记得上次讨论过什么、用户偏好是什么、项目决策是什么。claude-mem 通过本地 SQLite + hooks 把这些知识沉淀下来。

**Trade-off**：
- 本地 SQLite vs 云端存储：选了本地优先，隐私安全；cloud-sync 作为可选扩展
- Hook 自动触发 vs 手动调用 skill：选了 Hook 自动，降低用户摩擦，代价是安装时的「curl|bash」高风险安装流程
- 丰富 skill 库（19 个）vs 精简核心：选了扩展，带来了维护成本和质量不一致的问题（anti-pattern-czar.md 缺 frontmatter）

**认知模型**：作者把 AI 记忆类比于「人类海马体」——需要持续强化（PostToolUse hook 记录）+ 定期巩固（weekly-digests skill）+ 选择性召回（mem-search skill）。整个系统是一个神经科学类比的实现。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：CLAUDE.md 分层上下文树（Hierarchical CLAUDE.md Context Tree）**

claude-mem 在每一个有特定行为约束的目录下都放了 CLAUDE.md，形成「树状上下文继承」结构。根级 CLAUDE.md 给出全局约束，子目录 CLAUDE.md 给出局部精化。Claude Code 读取文件时，自动把路径上的所有 CLAUDE.md 叠加为上下文。

模式特征清单：
- **粒度最小化原则**：每个 CLAUDE.md 只约束自己目录的文件，不越权描述其他目录
- **可单独替换**：修改 `src/services/domain/CLAUDE.md` 不影响其他 CLAUDE.md
- **约束即文档**：CLAUDE.md 同时是 AI 的指令和人类的架构文档
- **hook 约束分离**：hooks 的行为约束单独放在 `plugin/hooks/CLAUDE.md`，不污染根级
- **测试有自己的上下文**：`tests/CLAUDE.md` 专门约束测试目录的 AI 行为

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型多模块项目（src/ 有多个子系统） | ✅ 高度适用 | 每个子系统有独立的 AI 行为约束，防止越界 |
| 有 hook 需要精细控制的项目 | ✅ 适用 | hooks/ 单独有 CLAUDE.md 隔离控制 |
| 小型单文件脚本项目 | ❌ 不适用 | 一个根级 CLAUDE.md 就够，分层是过度工程 |
| 频繁重构目录结构的项目 | ❌ 不适用 | 目录 CLAUDE.md 会随重构而失效，维护成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| CLAUDE.md 分层树（本仓库） | claude-mem | 精细控制每个子系统；天然文档 | CLAUDE.md 数量多，新贡献者学习成本高 |
| 单根级 CLAUDE.md | MarkQWu/bureau | 简单，一处修改全局生效 | 大型项目里上下文混杂，AI 容易犯跨模块错误 |
| skills-as-context（用 skill 代替 CLAUDE.md） | vladikk/modularity | NLPM 可扫描；可被 /use-skill 调用 | 需要显式调用，不自动继承 |

### 2.4 改进空间

1. **当前问题**：`openclaw/SKILL.md` 充当安装文档而非真正的 SKILL（无 frontmatter）。**改进做法**：把安装说明移入 `docs/openclaw-setup.md`；新建一个真正的 `openclaw/SKILL.md` 只做 Claude Code 可调用的技能定义，不包含 curl|bash 安装指令。**预期收益**：消除 4 个 CRITICAL 安全发现；skill 变成真正可注册的 NL 工件。

2. **当前问题**：`install/public/install.sh` 和 `openclaw/install.sh` 仍使用 `curl|bash` 模式。**改进做法**：改用验证安装方式：curl 下载 → 验证 SHA256 → 执行，或者提供 Homebrew/apt 包作为替代。**预期收益**：消除最高危安全风险类别，解除 NLPM 的 SECURITY BLOCKED 状态。

3. **当前问题**：skill 从 6 个扩展到 19 个，但测试覆盖未同步增长（tests/ 里的 CLAUDE.md 从 audit 时就存在，但 skill 级测试未见增加）。**改进做法**：为每个新 skill 在 `.nlpm-test/` 中加一个 NL-TDD 规格文件，借鉴 agents-in-the-wild 的 dogfood 做法。**预期收益**：skill 行为有验证基线；/nlpm:test 可发现退化。

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）

thedotmack/claude-mem 在 2026-04-07 得分 **97/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude/commands/anti-pattern-czar.md` | 35 | 缺 name+description(-50)、缺 allowed-tools(-5)、缺空参数处理(-10) |
| `openclaw/SKILL.md` | 50 | 缺 name+description(-50)（整个 frontmatter 不存在） |
| `CLAUDE.md`（根级）| 98 | 模糊量词 "relevant"(-2) |
| `plugin/skills/*`（6 个）| 100 | 无问题 |
| `plugin/hooks/hooks.json` | 100 | 无问题 |
| 20+ 个 CLAUDE.md | 100 各 | 无问题 |

97 分的原因是两个质量差文件（anti-pattern-czar 和 openclaw/SKILL.md）的权重被大量 100 分文件稀释。

### 3.2 当时值得借鉴的模式

1. **CLAUDE.md 分层体系** → `plugin/CLAUDE.md`、`src/services/CLAUDE.md`、`tests/CLAUDE.md` 等分层。为什么好：AI 只在自己「看得见」的上下文范围内操作，防止跨模块越界。→ 在自己的多模块项目里，每个子系统加一个 CLAUDE.md 约束 AI 边界。

2. **version-bump skill** → `plugin/skills/version-bump/SKILL.md` 把版本同步步骤（哪些文件要改、按什么顺序）固化为 skill。为什么好：版本同步容易漏步骤（audit 时确实漏了一个 plugin.json），skill 化后 AI 可以自动执行完整流程。→ 把「容易忘的多步骤流程」封装成 skill。

3. **hook 配置文件规范** → `plugin/hooks/hooks.json` 得了满分，hooks 有独立的 CLAUDE.md 隔离上下文。为什么好：hook 是高权限执行表面，独立隔离约束可以防止 AI 误读 hook 指令。→ 在自己项目的 hooks/ 目录单独放一个 CLAUDE.md，说明「hook 只能做什么，不能做什么」。

4. **记忆注入架构** → `mem-search` + `make-plan` + PostToolUse hook 形成「记录→检索→注入」闭环。为什么好：这是真正的跨会话学习系统，不靠用户手动维护。→ 类比：把「会话日志 → NL 摘要 → 注入下次 session」的模式用在自己的 bureau 项目里。

### 3.3 当时的缺陷

1. **BUG-1/2/3/4：anti-pattern-czar.md 和 openclaw/SKILL.md 缺整个 frontmatter YAML 块**
   - **根本原因**：这两个文件是从普通 Markdown 文档「升级」来的，作者没有意识到 SKILL.md 文件需要 YAML frontmatter 才能被 Claude Code 注册。这是「文档格式 vs 工件格式」混淆的典型模式——文件内容很好，但格式不对所以无法被系统识别。
   - **自查**：我的 bureau/skills/lint/SKILL.md 有正确的 frontmatter 吗？需要检查。

2. **SECURITY CRITICAL：smart-install.js 中的 `curl|bash` 自动安装**
   - **根本原因**：作者为了降低用户安装摩擦（Bun 和 uv 不在用户机器上时自动安装），选择了最省事的 `curl|bash` 模式。这个决策把安全风险从「可选」变成了「不可避免」——只要用户安装插件，这些脚本就会执行。
   - **为什么会失败**：任何一个 CDN 或域名被劫持（bun.sh、astral.sh、install.cmem.ai），用户机器上就会执行攻击者的代码。这不是假设风险，2024 年已有多起 CDN 劫持事件。
   - **自查**：我的项目脚本里有没有 `curl|bash` 模式？搜索 `grep -rn "curl.*bash\|curl.*sh" ~/.claude/` 确认。

3. **BUG-5：版本漂移（root plugin.json 落后 1 个大版本）**
   - **根本原因**：多个地方维护版本号（package.json + 两个 plugin.json）。`version-bump` skill 把应该更新的文件列出来了，但人工执行时漏了一个。这是「手工同步多处版本」的经典失败模式。
   - **自查**：我的插件有没有多处版本号？如果有，是否有测试验证它们一致？

### 3.4 当时的优化机会

1. **为 openclaw/SKILL.md 加 YAML frontmatter**：5 分钟，立即让 skill 变可注册。
2. **将 anti-pattern-czar.md 加 frontmatter + allowed-tools + 空参数保护**：10 分钟修复，从 35 分升至 95 分。
3. **把 curl|bash 替换为带校验和的安装方式**：这是解除 SECURITY BLOCKED 的前提条件。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-1/2：anti-pattern-czar.md 缺 frontmatter | `head -3 .claude/commands/anti-pattern-czar.md` | **持续** — 文件仍以 `# Anti-Pattern Czar` 开头，无 YAML 块 | 4 个月过去仍未修，说明该文件是「内部工具」而非对外插件功能的核心路径 |
| BUG-3/4：openclaw/SKILL.md 缺 frontmatter | `head -3 openclaw/SKILL.md` | **持续** — 文件仍以 `# Claude-Mem OpenClaw Plugin — Setup Guide` 开头 | 与 BUG-1 同理，openclaw 面向特定用户群，修复优先级低 |
| BUG-5：版本漂移 | 对比 `package.json` 和两个 `plugin.json` 的 version | **已修复** — 三处版本均为 13.13.1 | `version-bump` skill 发挥了作用；或直接更改了工作流 |
| SECURITY：smart-install.js curl|bash | `ls scripts/smart-install.js` | **部分修复** — `scripts/smart-install.js` 已被删除；但 `install/public/install.sh` 和 `openclaw/install.sh` 仍有 `curl|bash` | 最高风险的自动触发场景（Setup hook）已消除；手动安装路径风险仍存 |
| SECURITY：openclaw/SKILL.md curl|bash 指引 | `grep "curl.*bash" openclaw/SKILL.md` | **持续** — 4 处 curl|bash 仍在文档中 | 文档层风险继续，但已不是自动触发 |

### 4.2 架构演进

- **Skills 从 6 → 19（增长 213%）**：新增 cloud-sync、standup、weekly-digests、knowledge-agent、pathfinder、what-the 等，从「单机记忆」演进为「团队记忆 + 工作报告」场景。
- **smart-install.js 删除**：移除了最危险的自动依赖安装逻辑，把安装依赖推到用户侧（需要手动安装 Bun/uv），是重大安全改进，同时也提高了安装门槛。
- **版本号从 v11 → v13**：两个大版本跨越，说明 3 个月内有重大架构变更（但因 `--depth 1` clone 无法追溯细节）。
- **workers/ 目录出现**：新增 workers/ 说明引入了后台任务机制（可能是 cloud-sync 的 worker）。

### 4.3 新增的可学习模式

`plugin/modes/` 目录：新增了「角色模式」机制，允许用户切换 claude-mem 的行为模式（类似 Claude Code 的 [模型上下文协议](#mcp) 的 profile 概念）。这在 audit 时不存在，是 v11→v13 期间引入的新模式。这种「配置化角色切换」减少了用户手动提 prompt 的负担。

---

## 五、校准

### 5.1 我已经在做对的

1. **CLAUDE.md 有约束 AI 行为的文字**：bureau/CLAUDE.md 和 gstack/CLAUDE.md 都存在，方向正确。
2. **plugin.json 版本管理在意**：bureau 有 version 字段，方向正确。
3. **skill 有完整 frontmatter**：gstack/SKILL.md 的 name、description、allowed-tools 均有，对比 openclaw/SKILL.md 是正确的。
4. **不用 curl|bash 安装依赖**：我的项目脚本里没有这种危险模式，是正确的安全意识。

### 5.2 挑战 / 验证

本案例挑战了「高 NLPM 分 = 高安全质量」的假设。claude-mem 的 97/100 完全来自 NL 工件质量，而 SECURITY BLOCKED 来自完全不同的执行表面（安装脚本）。这提醒我：

**NL 质量分和安全质量分是正交的**。我自己的项目也需要单独审查「可执行表面」（hooks、scripts），不能用 NLPM 分数代替安全审查。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的项目是否有 curl|bash 危险安装模式
grep -rn "curl.*bash\|curl.*|bash\|curl.*|sh\|iex\|IEX" \
  /tmp/my-repos/MarkQWu-*/ 2>/dev/null | grep -v ".git"
# 命中后：替换为带 SHA256 校验的下载方式，或改用系统包管理器
```

```bash
# 检查我的 SKILL.md 是否有完整 frontmatter
find /tmp/my-repos/MarkQWu-bureau -name "SKILL.md" | while read f; do
  if ! head -1 "$f" | grep -q "^---"; then
    echo "MISSING FRONTMATTER: $f"
  fi
done
# 命中后：在文件最顶部添加 --- name: ... description: ... allowed-tools: [...] ---
```

```bash
# 检查我的项目是否有 CLAUDE.md 分层（子目录是否有独立约束）
find /tmp/my-repos/MarkQWu-bureau -name "CLAUDE.md" | sort
# 如果只有根级 CLAUDE.md，考虑为 commands/、skills/、hooks/ 各加一个
```

### 6.2 灵感 → 实施路径

1. **想法**：仿照 version-bump skill，为 bureau 做一个「发布检查 skill」
   - **为何可行**：bureau 每次更新 BUREAU.md 或 skills 时，需要同步更新 plugin.json 版本号，这是一个容易忘的多步骤流程
   - **第一步**：新建 `skills/release-check/SKILL.md`，列出「发布前必须验证的 5 个文件」清单；加入 allowed-tools: [Read, Glob]；约 20 分钟

2. **想法**：借鉴 CLAUDE.md 分层树，为 bureau 的 hooks/ 目录加单独约束
   - **为何可行**：bureau 有 hooks/ 目录（PostToolUse），目前无独立 CLAUDE.md；AI 在 hooks 上下文里可能做出越权操作
   - **第一步**：在 `hooks/CLAUDE.md` 中写「hook 只能记录，不能修改 canon/ 目录；hook 执行的任何写操作必须先检查路径」；约 15 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 thedotmack/claude-mem 的核心目的**：为 Claude Code 提供跨会话持久记忆，自动捕获 + 自动注入。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同样是「AI 会话知识持久化」；都有 capture → compile → recall 管线 | bureau 是显式的「审阅 → 规范 → 查询」流程；claude-mem 是透明的「hooks 自动触发」流程 | 高 |
| MarkQWu/gstack | 低 | 都是 Claude Code 插件 | gstack 是工作流工具，无记忆管理功能 | 低 |

若做类比：claude-mem 是「海马体式」记忆（自动触发、透明）；bureau 是「日记式」记忆（显式记录、需人工审阅）。两种模式不互斥，可以互补。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| BUG-1/2：SKILL.md 缺完整 frontmatter | `find MarkQWu-bureau/skills -name "SKILL.md" \| head -3 \| xargs head -1` | **未命中** — bureau skills 均有 `---` 开头的 frontmatter | 无 |
| SECURITY：curl|bash 安装模式 | `grep -rn "curl.*bash" MarkQWu-*` | **未命中** — 我的项目无此模式 | 无 |
| BUG-5：多处版本号漂移 | `grep -r '"version"' MarkQWu-bureau/ \| grep -v node_modules` | bureau 的 plugin.json 和 package.json 需要对照检查 | 中（需确认） |

**命中后的具体行动建议**：
- 若 bureau 多处版本号不一致：新建 `skills/version-sync/SKILL.md`，把「需要同步的 3 个版本文件路径」写进去；下次更新时调用 `/use-skill version-sync` 执行

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：CLAUDE.md 分层深度
   - **本案例做法**：20+ 个 CLAUDE.md 覆盖每个子目录（`src/services/domain/CLAUDE.md`、`tests/utils/CLAUDE.md` 等），精细到「方法论层」
   - **我的项目现状**：`MarkQWu-bureau` 有根级 `CLAUDE.md` 和 `BUREAU.md`，但子目录（skills/、commands/、hooks/）均无专用 CLAUDE.md
   - **如何借鉴**：为 bureau 的 `hooks/` 目录加 `CLAUDE.md`（约束 hook 只能做只读操作，不能直接写 canon/），10 分钟可完成

2. **领域**：PostToolUse hook 做自动化
   - **本案例做法**：`hooks/hooks.json` 的 PostToolUse 事件自动捕获 AI 产出，无需用户手动调用 capture 命令
   - **我的项目现状**：bureau 的 capture 需要用户手动运行 `bureau:capture`，摩擦高
   - **如何借鉴**：为 bureau 加一个 PostToolUse hook，当 Claude 完成 Write/Edit 操作时自动创建 `status: logbook` 分钟记录草稿；用户只需在 review 时批准；约 1 小时实施

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安全约束层
- **我的做法**：`MarkQWu-bureau` 的 `bureau:serve` 命令有「reviewer token 打印到终端；propose 开放；dispose 是人的行为」的明确权限模型，AI 无法绕过人工批准直接写 canonical
- **本案例做法（弱在哪）**：claude-mem 的 hook 自动触发写入，安装脚本有 `curl|bash`，缺乏「人工确认层」——透明性和安全性之间选择了透明性
- **意义**：bureau 的「人工门控」设计在安全审计时是亮点，未来可以在 NLPM audit 的「安全设计」维度作为正面参考

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 Claude Code 插件架构：对外暴露的是 Markdown SKILL.md 文件（NL 层），但真正的执行逻辑在 TypeScript/Go/Python 等编译型语言写的程序里（核心层）。SKILL.md 指挥 AI 去调用这些程序，AI 本身不执行计算密集或系统级操作。claude-mem 的「核心」是 `src/` 里的 SQLite 服务，「表皮」是 `plugin/skills/` 里的 SKILL.md。

### <a name="curl-pipe-bash"></a>curl|bash
> 一种常见但有风险的软件安装命令：`curl -fsSL https://some-url/install.sh | bash`。意思是「从网络下载一个脚本然后立刻执行」。风险在于：如果下载 URL 被劫持，用户会在不知情的情况下执行攻击者的代码。安全替代方案是：先下载到本地，验证 SHA256 校验和，再执行。

### <a name="mcp"></a>模型上下文协议（MCP）
> Model Context Protocol 的缩写，Anthropic 发布的标准接口规范，允许外部工具（数据库、搜索引擎、文件系统）以统一方式向 Claude 提供上下文。claude-mem 的 `plugin/.mcp.json` 把自己的记忆服务注册为 MCP 服务，让 Claude 可以通过标准接口查询历史记忆。

### <a name="sqlite"></a>SQLite
> 一种把整个数据库存在单个文件（`.db` 文件）里的数据库引擎。不需要服务器，直接读写本地文件。claude-mem 用它存储记忆条目，优点是零配置、跨平台、隐私友好。

### <a name="hookconfig"></a>hooks.json
> Claude Code 的 hook 配置文件。声明哪些事件（PostToolUse、PreToolUse 等）触发哪些脚本执行。claude-mem 用它在 AI 完成写操作后自动把内容存入记忆数据库。
