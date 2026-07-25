# LerianStudio/ring — 学习案例

**仓库**：https://github.com/LerianStudio/ring
**Stars**：174 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-25（基于当前 HEAD）
**主题标签**：security-gate, template-design, manifest-discipline, experience-accumulation, cross-reference

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Ring 是 LerianStudio（巴西金融科技公司 Lerian）为自身开发流程定制的 [Claude Code](#claude-code) 插件，把 AI agent 工程化为"多团队评审委员会"——backend、frontend、QA、文档等专职 [agent](#agent) 分团队协作，通过门控（Gate）流程保证代码质量。

关键事实：
1. **作者背景**：LerianStudio 是 Lerian 的 GitHub 组织，以 Go/TypeScript 金融软件为主营；ring 是内部效率工具，专为 Lerian 的代码审查和开发流程而建
2. **获取方式**：`bash ring-install.sh --claude` 通过脚本安装，或从 GitHub 直接 clone
3. **生态位置**：Claude Code 多 agent 协作的企业级示例；以 `ring:` 统一命名空间封装所有组件
4. **规模演进**：审计时（2026-04-19）有 41 个活跃 agent + 17 个存档 agent；当前 HEAD 有 42 个 agent + 137 个 skill，finops-team 和 pmo-team 已删除，仅保留 4 个团队

### 1.2 架构剖析

**目录结构**（当前 HEAD）：

```
ring/
├── CLAUDE.md          ← 系统"法典"，含 7 条强制规则（AGENTS.md 是其 symlink）
├── AGENTS.md          ← 软链接，指向 CLAUDE.md
├── ARCHITECTURE.md    ← 架构文档
├── MANUAL.md          ← 使用手册
├── ring-install.sh    ← 安装脚本（Windows: .ps1）
├── default/           ← 默认团队（code-reviewer, session hooks, 通用 skills）
│   ├── agents/        ← 14 个 agent
│   ├── hooks/         ← session-start.sh（含 PyYAML 自动安装逻辑）
│   ├── skills/        ← 通用 skill 库
│   └── commands/      ← 命令层
├── dev-team/          ← 开发团队（backend-go, frontend, bff, 多个 reviewer）
│   ├── agents/        ← 20 个 agent（含并行评审 Pool）
│   ├── hooks/         ← validate-gate-progression.sh（门控验证）
│   └── skills/
├── pm-team/           ← 产品/研究团队（analyst, designer, researcher）
│   └── agents/ + skills/
├── tw-team/           ← 技术写作团队（api-writer, docs-reviewer, functional-writer）
│   └── agents/ + skills/
└── shared/            ← 跨团队共享库（lib/, scripts/）
```

**文件类型分布**：42 个 agent / 137 个 [SKILL.md](#skill-md) / 10 个 hook / 1 个 CLAUDE.md

**编排关系**：
- `default/agents/code-reviewer.md` 是 Gate 8 的入口，与 dev-team 的 10 个并行 reviewer（commons-reviewer, security-reviewer, nil-reviewer 等）组成"评审池"
- `dev-team/hooks/validate-gate-progression.sh` 是门控守卫，验证 reviewer 池的完整性
- `default/hooks/session-start.sh` 在会话启动时自动安装依赖

**跨件契约**：
- 所有组件强制使用 `ring:` 前缀（`name: ring:code-reviewer`）
- CLAUDE.md 的"SEVEN-FILE UPDATE RULE"规定：每次修改 reviewer 池，必须同一个 commit 内更新 7 个文件
- AGENTS.md 是 CLAUDE.md 的 [symlink](#symlink)，保证两份文件内容永远同步

### 1.3 设计思路 / 方法论

**核心设计哲学**："[法典先行](#法典先行)（Constitution-First）"——CLAUDE.md 不只是说明文档，而是一套强约束的操作规程，包含"MUST NOT"级别的禁止项、多文件同步规则和反理性化检查表（[Anti-Rationalization Table](#anti-rationalization-table)）。

**解决什么问题**：金融软件团队的代码审查质量不稳定——人容易在压力下跳过门控、接受理由充分的例外。Ring 通过 AI 强制执行门控流程，消除"这次先跳过"的借口。

**核心 Trade-off**：
- **严格性 vs 灵活性**：CLAUDE.md 的规则极度刚性（MUST NOT, FORBIDDEN），牺牲了快速开发的灵活性，换取了流程的可审计性
- **Rolling Standards vs 版本稳定**：所有 agent 在运行时从 GitHub `main` 分支实时拉取标准（WebFetch），保证了标准总是最新的，但引入了供应链依赖
- **多团队分工 vs 协调成本**：4 个专职团队确保了单一职责，但 SEVEN-FILE UPDATE RULE 也说明跨文件协调成本不低

**认知模型**：作者把 AI agent 视为"可执行的纪律系统"——不是智能助手，而是不允许被说服的流程执行者（CLAUDE.md Rule 2："Agents are EXECUTORS, Not DECISION-MAKERS"）。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「[法典驱动的多团队评审委员会](#法典驱动的多团队评审委员会)」

CLAUDE.md 作为不可绕过的"法典"，多个专职 agent 分团队各司其职，通过 hook 强制执行门控，并行评审池减少序列化瓶颈。

模式特征清单：
- **强制性 CLAUDE.md**：不只是说明，而是含 `⛔ MUST NOT` 的操作规程，agent 不得违反
- **统一命名空间**：全部组件用 `ring:` 前缀，避免与用户已有 skill 命名冲突
- **门控 + 并行评审**：Gate N 机制 + 10 个 reviewer 并行运行，不允许跳过
- **反理性化表**：每个 reviewer agent 都内嵌"压力测试"——列出常见被接受的例外理由并明确拒绝它们
- **法典自同步**：AGENTS.md symlink 到 CLAUDE.md，消除"文档漂移"问题

### 2.2 适用场景

这个架构最适合什么样的项目？什么时候**不要**用？

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 金融/医疗等强合规要求的团队 | ✅ 高度适用 | 门控+反理性化机制正是为此设计 |
| 10+ 人工程团队，有成熟代码评审流程 | ✅ 适用 | 替代 / 增强人工评审，一致性更高 |
| 个人项目或早期 MVP | ❌ 过度工程 | 42 个 agent + 137 个 skill 的维护成本远超收益 |
| 技术栈多变的跨平台项目 | ❌ 不适用 | Ring 深度绑定 Go/TypeScript/Helm，迁移成本高 |
| 离线或内网受限环境 | ⚠️ 有风险 | Rolling Standards 依赖实时访问 GitHub main 分支 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 法典驱动多团队评审委员会（本仓库） | LerianStudio/ring | 流程强执行、反理性化、团队专职化 | 维护成本高、Rolling Standards 供应链风险、agent 全部缺 model |
| 单人工作流套件（命令路由层） | webdevtodayjason/sub-agents | 简单易上手，一人可维护 | 无门控机制，无法防止跳步 |
| NL 表皮 + 原生二进制核心 | agent-sh/agnix | 性能高、CLI 原生体验 | 需要编译能力，非 AI 原生 |
| 单仓平铺 skill 库 | MarkQWu/bureau | 轻量、易于迭代 | 无 agent 协调机制 |

### 2.4 改进空间

1. **当前问题**：所有 agent 缺 `model:` 字段，Claude Code 无法感知预期模型，高频的 reviewer agent 和低频的 analyst agent 使用相同（默认）模型，成本/速度无法优化  
   **改进做法**：reviewer 类（并行运行、高频次）→ `model: claude-haiku-4-5`；specialist/planner 类 → `model: claude-sonnet-4-6`  
   **预期收益**：reviewer 并行 10 个时成本降低约 80%，且评审速度更快

2. **当前问题**：PyYAML 自动安装版本范围 `>=6.0,<7.0`，默认 `RING_AUTO_INSTALL_DEPS=true`，每次 session 启动都可能安装不同 patch 版本  
   **改进做法**：改为 `'PyYAML==6.0.2'`（精确版本），同时将默认值改为 `false`（opt-in）  
   **预期收益**：消除 HIGH 级安全风险，企业环境不会在用户不知情下发起网络调用

3. **当前问题**：Rolling Standards（agent 每次运行时从 `main` 分支拉标准）无版本锁定，标准变更立即影响所有正在进行的开发周期  
   **改进做法**：增加 `RING_STANDARDS_REF` 环境变量，支持指定 commit/tag，只在 CI/CD 时使用 `main`  
   **预期收益**：生产部署可锁定到已验证的标准版本，消除 MEDIUM 安全风险

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 当时得分 **87/100**。Security 状态：**APPROVE WITH REQUIRED FIXES**（含 2 个 HIGH 安全风险）。

| 组件 | 当时分数 | 主要问题 |
|---|---|---|
| 活跃 agent（41 个平均） | 88/100 | 全部缺 `model:`（-5），仅 1 个示例（-5），少量模糊量词 |
| 存档 agent（17 个平均） | 78/100 | 同上 + 双 `---` bug（-5）+ 缺 `ring:` 前缀 |
| skills（94 个估计） | ~88/100 | 同活跃 agent 模式，结构较好 |
| CLAUDE.md | 93/100 | 近乎完美（不适用 `model:` 字段） |
| 整体加权 | **87/100** | 存档拉低了总分 |

### 3.2 当时值得借鉴的模式

1. **反理性化检查表（Anti-Rationalization Table）** → 每个 reviewer agent 都列出"这些理由无论多充分，都不能绕过门控"，从根本上封堵了"这次先跳过"的逃逸路径 → 路径：`dev-team/agents/code-reviewer.md` → 借鉴：在自己的 reviewer skill 里加一个"不得绕过"小节

2. **SEVEN-FILE UPDATE RULE** → 每次修改 reviewer 池，CLAUDE.md 明确列出必须同步更新的 7 个文件，把"跨文件同步"这个隐式约定变成显式规程 → 路径：`CLAUDE.md` §7 → 借鉴：任何涉及多文件联动的设计都应在 CLAUDE.md 里写出同步清单

3. **AGENTS.md 作为 CLAUDE.md 的 symlink** → 一份文档，两个入口，彻底消除文档漂移 → 路径：`AGENTS.md`（symlink）→ 借鉴：若项目同时需要 CLAUDE.md 和 AGENTS.md，优先用 symlink

4. **统一命名空间 `ring:`** → 所有组件 `name: ring:xxx`，一条 grep 可以找到全部组件，且不会和用户环境的其他 skill 产生名字冲突 → 路径：所有 agent [frontmatter](#frontmatter) → 借鉴：自己的多 skill 项目应统一用 `<project>:xxx` 格式

5. **Gate 机制 + 并行评审池** → 10 个 reviewer 并行 + hook 验证完整性，串行评审可能因为某个 reviewer 被说服而失效，并行则增加了"共谋"的难度 → 路径：`dev-team/agents/*.md` → 借鉴：高可信度的自动评审应设计为并行多 reviewer

### 3.3 当时的缺陷

1. **B1 + B2：存档 agent 双 `---` bug + 缺 `ring:` 前缀**  
   为什么失败：历史 agent 在迁移入 `.archive/` 时，可能是直接移动文件而没有同步更新 [frontmatter](#frontmatter)，缺少迁移检查脚本；`ring:` 前缀规则是后来制定的，旧文件没有回填  
   自查：我有没有犯同样的错？ → 我的 bureau 技能文件在重构时曾经挪动过路径，但因为 bureau 没有命名空间规则，所以没有暴露这个问题

2. **Q1：所有 agent 缺 `model:` 字段（100% 覆盖率）**  
   为什么失败：ring 可能认为 `model:` 是可选的，让 Claude Code 自动选择；但这意味着成本和行为都不可控，特别是 10 个并行 reviewer 同时用默认模型时费用会翻倍  
   自查：我的 gstack 有 54 个 skill，bureau 有 7 个 skill，两者均 100% 缺 `model:`——和 ring 完全相同的系统性问题

3. **H1：PyYAML 自动安装版本不固定 + 默认开启**  
   为什么失败：开发者习惯于用版本范围（`>=6.0,<7.0`）确保兼容性，但在自动安装场景下，minor/patch 版本的差异可能引入供应链风险；默认开启则意味着所有安装用户都会在 session 启动时触发网络请求  
   自查：我的项目没有 session-start hook，这个具体风险不适用，但我需要警惕任何"静默安装依赖"的模式

4. **CC4：Rolling Standards 没有文档化的风险说明**  
   为什么失败："Rolling standards"是一个刻意的设计选择，但 CLAUDE.md 和 README 没有明确写出"这会创建对 main 分支的供应链依赖"，用户可能在不知情的情况下接受了这个风险  
   自查：我的任何 skill 如果用 WebFetch 拉外部数据，都应该在 skill 说明里注明这是实时拉取还是版本锁定

### 3.4 当时的优化机会（学习用）

1. **给 `session-start.sh` 加 SHA256 校验**：安装脚本已经有 `MARKETPLACE_JSON_SHA256` 的变量占位（line 98），但没有真正实现校验逻辑，补上即可消除 H2

2. **`tw-team` 的"Standards Compliance Report"节改为有内容或删除**：3 个 TW agent 都有 `## Standards Compliance Report: **N/A**` 的占位节，视觉噪音；改为检查语气/声音一致性的自检清单更有价值

3. **为 `frontend-bff-engineer-typescript.md` 去除重复节**：B3 bug，"Post-Implementation Validation"出现了两次；删除第二个即可

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| B1 存档 agent 双 `---` bug（17 个文件） | `ls /tmp/target-LerianStudio-ring/.archive/` | `.archive/` 目录**已不存在**，整个存档被删除 | Bug 消失——不是修复而是删除了有问题的存档；存档 agent 不再维护 |
| B2 存档 agent 缺 `ring:` 前缀 | 同上 | 同上，存档已删除 | 同上 |
| Q1 所有 agent 缺 `model:` | `grep -r "^model:" dev-team/agents/` → 0 命中 | **仍然存在**，100% 的 agent 缺 `model:` | 这是 ring 最久未修的系统性问题；174 天（2026-04-19 → 2026-07-25）过去了，仍未修复 |
| H1 PyYAML 版本不固定 | `grep "PyYAML" default/hooks/session-start.sh` | **仍然存在**：`'PyYAML>=6.0,<7.0'`，且 `RING_AUTO_INSTALL_DEPS:-true` | HIGH 安全风险持续；说明作者认为精确固定版本的代价（每次要更新版本号）高于风险 |

### 4.2 架构演进

**删除了什么**：
- `.archive/` 整个目录（含 17 个存档 agent）——说明作者决定不再维护历史版本，干净利落地清理技术债
- `finops-team/`（FinOps 团队）和 `pmo-team/`（项目管理团队）——团队精简，聚焦核心开发/测试/文档流程

**增加了什么**：
- `ring-install.ps1`（Windows 支持）
- `pm-team/` 保留（产品/研究），但结构重组
- 137 个 skill（从审计时约 94 个增加到 137 个）——skill 持续增长说明知识沉淀在加速

**说明作者意识到了什么**：
- 存档 agent 的维护成本 > 价值，不如直接删除
- finops / PMO 流程不是 ring 的核心使用场景，聚焦比覆盖更重要
- skill 数量快速增长说明 ring 被积极使用，团队持续向系统沉淀流程知识

### 4.3 新增的可学习模式

**Skill 数量的快速增长（94 → 137，+46%）**：审计 3 个月后 skill 增加了 43 个，说明在真实使用中"写 skill"成为了团队的常规习惯，而不是偶发行为。这是 [experience-accumulation](#experience-accumulation) 模式的最好佐证——系统设计要让"沉淀"比"临时解决"更省力。

**Windows 安装支持（ring-install.ps1）**：跨平台安装脚本，说明 ring 的实际用户扩展到了 Windows 环境，这对企业内部工具来说是成熟度的标志。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau 的 skill 结构有 `name:` 字段**：ring 的 `ring:` 前缀规则和我 bureau 的 skill 命名是同一个方向，我已经在为 skill 取有意义的名字，只是没有像 ring 那样加项目前缀

2. **bureau 有 `hooks/hooks.json`**：我的 bureau 已经配置了 hook，这和 ring 的 session-start.sh 是同样的思路——利用 Claude Code 生命周期事件做自动化

3. **bureau 的模糊量词控制**：上一个案例（2026-07-25 agent-sh-agnix）记录到 bureau 的 vague quantifier 计数为 0，这是比 ring 的 `frontend-bff-engineer-typescript.md`（5+ 个模糊量词）更好的实践

4. **agents-in-the-wild 有正式的文档结构**：类似 ring 的 CLAUDE.md，agents-in-the-wild 的 CLAUDE.md 也规定了整个 pipeline 的行为规则，只是约束性没有 ring 那么强

### 5.2 挑战 / 验证

**被验证的假设**：我一直认为"CLAUDE.md 的内容越具体越好"，ring 的案例证实了这一点——ring 的 CLAUDE.md 得了 93/100，其高分来自于极度具体的 MUST NOT 列表和 SEVEN-FILE UPDATE RULE，而不是模糊的"建议"。

**被挑战的假设**：我本来认为"Rolling Standards（永远用最新版本）比版本锁定更好维护"，但 ring 的案例表明这个设计选择本身就是一个 MEDIUM 级安全风险——在安全敏感环境中，"永远最新"等于"永远不可预期"。正确做法是：提供版本锁定选项（`RING_STANDARDS_REF`），让用户自己选择稳定性 vs 新鲜度。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 gstack 的 skill 是否全部缺 model:（ring 的 Q1 问题）
grep -rL "^model:" /tmp/my-repos/MarkQWu-gstack/ --include="*.md" 2>/dev/null | grep -i "skill\|SKILL" | wc -l
# 命中后：逐个 skill 加 model: 字段，reviewer 类用 haiku，complex 类用 sonnet

# 检查 bureau 的 skill 是否全部缺 model:
grep -rL "^model:" /tmp/my-repos/MarkQWu-bureau/skills/ 2>/dev/null | wc -l
# 命中后：同上，7 个 skill 一次修完，优先级最高

# 检查我是否有任何 skill/agent 用了 WebFetch 但没有说明是否 rolling（供应链风险）
grep -rn "WebFetch\|fetch.*http" /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null
# 命中后：在该 skill 的说明里注明"实时拉取 / 版本锁定"及其安全含义

# 检查我的 bureau/gstack 是否有 hooks 自动安装依赖
grep -rn "pip install\|npm install\|apt-get" /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null
# 命中后：确认版本是否固定（必须用精确版本号，不用范围）
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 7 个 skill 都加 `model:` 字段  
   **为何可行**：7 个文件，每个只改 1 行，30 分钟可全部完成；消除和 ring Q1 一样的系统性问题  
   **第一步**：打开 `bureau/skills/capture/SKILL.md`，在 [frontmatter](#frontmatter) 第一个 `---` 块内加 `model: claude-haiku-4-5`（capture/recall/lint 等轻量 skill）或 `model: claude-sonnet-4-6`（compile/review 等复杂 skill）

2. **想法**：在 bureau 的 CLAUDE.md 里加一条"MUST NOT"规则：任何 skill 不得有 `model:` 字段缺失  
   **为何可行**：ring 证明了把约定写进 CLAUDE.md 比依赖个人记忆更可靠；这是从 ring 的 SEVEN-FILE UPDATE RULE 精神中学到的  
   **第一步**：在 `bureau/CLAUDE.md` 的 Critical Rules 节加 `⛔ All skills MUST declare model: in frontmatter`，30 分钟

3. **想法**：学习 ring 的 AGENTS.md symlink 模式  
   **为何可行**：bureau 同时有 CLAUDE.md，如果将来需要 AGENTS.md，symlink 是零维护成本的方案  
   **第一步**：`ln -sf CLAUDE.md AGENTS.md` 在 bureau 根目录执行，1 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（上次刷新：2026-07-20）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 LerianStudio/ring 的核心目的**：企业级 Claude Code 插件，多团队 agent 协作 + 门控评审流程

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是 Claude Code 插件，有 plugin.json + hooks + 多个 skill | bureau 是个人轻量工具，无门控/评审机制；ring 是企业多团队系统 | 高 |
| MarkQWu/gstack | 中 | 都有大量 skill 文件（gstack 54 个，ring 137 个） | gstack 以 stack 为组织单位而非团队；没有 agent 协调 | 中 |
| MarkQWu/agents-in-the-wild | 低 | 都有 CLAUDE.md 作为规程文档 | agents-in-the-wild 是审计/学习系统，不是开发辅助工具 | 低 |

### 8.2 在我的项目里复现的同类问题

对 §3.3「当时的缺陷」和 §4.1「现在仍存在的缺陷」逐条核查我的项目：

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Q1 所有 agent/skill 缺 `model:` | `grep -rL "^model:" /tmp/my-repos/MarkQWu-bureau/skills/` | bureau 7/7 命中（100%）；gstack 54/54 命中（100%） | 高 |
| 存档文件迁移时 frontmatter 未同步 | `find /tmp/my-repos/ -name "*.archive" -o -name "*/.archive/"` | 无存档目录，N/A | 低 |
| Rolling Standards 无文档化风险说明 | `grep -rn "WebFetch\|http" /tmp/my-repos/MarkQWu-bureau/` | 未发现 WebFetch 调用，N/A | 低 |

**命中后的具体行动建议**：

- `MarkQWu/bureau` 的 `skills/capture/SKILL.md` 等 7 个文件 → 在 [frontmatter](#frontmatter) 加 `model: claude-haiku-4-5`（或 sonnet 视 skill 复杂度）→ 每个文件约 2 分钟，共 14 分钟
- `MarkQWu/gstack` 的 54 个 SKILL.md → 批量加 `model:` → 先用 `grep -rL "^model:"` 列出缺失文件，再批量 sed 插入，30 分钟可搞定

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：CLAUDE.md 作为强约束规程（法典）  
   - **本案例做法**：CLAUDE.md 含 `⛔ CRITICAL RULES`（MUST NOT 级别）、SEVEN-FILE UPDATE RULE、反理性化表、symlink 规则；得分 93/100  
   - **我的项目现状**：bureau 的 CLAUDE.md 较短，主要是说明而非强制规程；缺少"MUST NOT"级别的约束  
   - **如何借鉴**：在 bureau 的 CLAUDE.md 里增加一个 `⛔ CRITICAL RULES` 节，至少包含：(1) 所有 skill 必须有 `model:`；(2) 修改 hook 时必须同步更新 plugin.json 的 `hooks` 路径

2. **领域**：统一命名空间前缀  
   - **本案例做法**：`name: ring:xxx` 全部组件统一前缀，一条 grep 找到所有 ring 组件  
   - **我的项目现状**：bureau 的 skill 名字没有项目前缀（如 `name: capture` 而不是 `name: bureau:capture`）  
   - **如何借鉴**：在 bureau 的所有 skill 里加 `bureau:` 前缀；这是一个 replace_all 操作，可以用 sed 批量完成

3. **领域**：reviewer agent 的反理性化检查表  
   - **本案例做法**：每个 reviewer agent 都有"即使对方理由再充分，这些门控也不能跳过"的清单  
   - **我的项目现状**：agents-in-the-wild 的 auditor agents 没有类似的"禁止被说服"条款  
   - **如何借鉴**：在 agents-in-the-wild 的 `auditor/` 相关 agent 里加 `⛔ MUST NOT accept these bypass reasons` 节

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：模糊量词控制  
   - **我的做法**：bureau 的 skill 文件 vague quantifier 计数为 0（前案例 2026-07-25 中已验证）  
   - **本案例做法**（弱在哪）：ring 的 `frontend-bff-engineer-typescript.md` 含 5+ 个模糊量词（"appropriate", "relevant", "comprehensive"），Q3 缺陷  
   - **意义**：如果将来 ring 团队发现了这个问题并想改，bureau 的"零模糊量词"实践可以作为参考；对自己来说，这是一个已建立的好习惯，要保持

---

## 八、术语表

### <a name="claude-code"></a>Claude Code
> Anthropic 提供的 AI 编程助手，可以通过命令行或 IDE 插件调用。支持通过 `plugin.json` 安装第三方插件，扩展 Claude 的 skill 和 agent 能力。

### <a name="agent"></a>agent
> Ring 中的"专职执行者"——一个包含特定角色说明的 Markdown 文件，Claude Code 加载后该角色就能执行特定任务（如代码审查、文档撰写）。与"人工智能助手"不同，agent 被设计为"不做决策只执行流程"的自动化组件。

### <a name="skill-md"></a>SKILL.md
> Claude Code 插件中的知识单元文件。与 agent.md 不同，SKILL.md 是参考资料（给 agent 看），不是执行者。Ring 里的 SKILL.md 定义了各类编程规范、流程步骤、质量标准等。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读取 agent.md 或 SKILL.md 时，先解析 frontmatter 才能知道如何注册和调用这个组件。

### <a name="symlink"></a>symlink
> 符号链接，一种"快捷方式文件"。`ln -sf CLAUDE.md AGENTS.md` 命令让 AGENTS.md 变成指向 CLAUDE.md 的符号链接——读 AGENTS.md 实际上读的是 CLAUDE.md 的内容。修改 CLAUDE.md 时，AGENTS.md 也自动更新，消除了两份文档各自维护的漂移问题。

### <a name="法典先行"></a>法典先行（Constitution-First）
> 一种文档设计哲学：把操作规程（而不只是说明文档）写进 CLAUDE.md，包含 `⛔ MUST NOT`、`⚠️ MUST`、多文件同步规则等强约束条款。目的是让 AI agent 在执行任务时不能"被说服"绕过规则——规则是机器可读的约束，而不只是人类建议。

### <a name="anti-rationalization-table"></a>Anti-Rationalization Table（反理性化检查表）
> Ring 独创的 agent 内置机制：每个 reviewer agent 里都有一个清单，列出"这些理由无论多充分，都不能作为绕过门控的理由"。例如："时间紧" / "这是小改动" / "老板同意了" 都不是跳过安全审查的理由。这个机制从根本上防止 AI agent 被用户的"合理化"说辞所说服。

### <a name="experience-accumulation"></a>经验沉淀（Experience Accumulation）
> 一种系统设计模式：每次解决问题后，把解决方案沉淀为 skill 文件，而不是每次从头来过。Ring 的 skill 数量从审计时约 94 个增长到 137 个，说明团队在积极把实际工作中的流程知识沉淀进系统。这个模式让系统越用越聪明，而不是每次重置。

### <a name="法典驱动的多团队评审委员会"></a>法典驱动的多团队评审委员会
> 本案例的架构模式名。CLAUDE.md 作为强约束"法典"，多个专职 agent 分团队各司其职（开发、产品、技术写作），通过 hook 强制执行门控（Gate），并行评审池替代单个 reviewer，系统性防止流程被绕过。

### <a name="rolling-standards"></a>Rolling Standards（滚动标准）
> ring 的一个刻意设计选择：reviewer agent 在每次运行时通过 WebFetch 从 GitHub `main` 分支实时拉取最新的编码标准文档，而不是使用安装时的固定版本。优点是标准永远最新；缺点是创建了对 GitHub main 分支的供应链依赖——如果 ring 仓库被攻陷，所有正在运行的 agent 都会立即受到影响。
