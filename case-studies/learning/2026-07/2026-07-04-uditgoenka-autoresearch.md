# uditgoenka/autoresearch — 学习案例

**仓库**：https://github.com/uditgoenka/autoresearch
**Stars**：3612 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-04（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `single-purpose`, `vague-quantifier`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
uditgoenka 构建的「自主迭代研究工具」——给 Claude Code、Codex CLI、OpenCode 等多个 AI 编程工具使用的统一 skill/command 套件，核心是 `/autoresearch` 命令：给出一个目标和验证指标，让 AI 反复「改→验→保/丢弃」直到收敛。

关键事实：
- 36 个 NL 工件（4 个平台 × 约 9 个 command + 1 skill + 1 manifest）
- 多平台同步分发：`.claude/`（Claude Code）、`.opencode/`（OpenCode）、`.agents/`（agents CLI）、`claude-plugin/`（marketplace 发布用）
- 平台差异由 `scripts/sync-codex.sh` 和 `scripts/sync-opencode.sh` 脚本自动同步
- NL 得分 88/100；安全状态 CLEAR（medium/low 级别安全问题，无 critical/high）

### 1.2 架构剖析
```
autoresearch/
├── .claude/                    ← Claude Code 原生版本
│   ├── skills/autoresearch/SKILL.md
│   └── commands/
│       ├── autoresearch.md     ← 主命令（dispatch 入口）
│       └── autoresearch/       ← 子命令（12 个：plan/debug/fix/ship/...）
├── .opencode/                  ← OpenCode 平台版本
│   ├── agents/docs-manager.md
│   ├── skills/autoresearch/SKILL.md
│   └── commands/               ← 10 个命令（命名规则：autoresearch_xxx）
├── .agents/                    ← agents CLI 版本
│   └── skills/autoresearch/SKILL.md
├── claude-plugin/              ← marketplace 发布版本
│   ├── .claude-plugin/plugin.json
│   ├── skills/autoresearch/SKILL.md
│   └── commands/
├── scripts/                    ← 同步脚本
│   ├── sync-codex.sh           ← 从 .claude/ 同步到 claude-plugin/
│   └── sync-opencode.sh        ← 从 .claude/ 同步到 .opencode/
└── guide/                      ← 用户文档（不是 NL 工件）
```

- **文件类型分布**：4 个平台 × 各 1 个 SKILL.md（共 4 个）+ 约 30 个 command + 1 个 agent + 1 个 plugin.json
- **编排关系**：主命令 `autoresearch.md` 作为 dispatch 路由，根据参数（有无 Metric/Verify 字段、有无子命令标志）分发到 12 个子命令；`autoresearch_learn.md` 内部调用 `docs-manager` agent
- **跨件契约**：`.claude/` 版本是规范源（canonical source），其他平台版本由 sync 脚本生成，适配各平台的工具名和语法差异

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单规范源 + 多平台分发」——在 `.claude/` 维护一份权威版本，sync 脚本负责适配差异，保证跨平台行为一致
- **解决什么问题**：同一个 skill 要在 4 个 AI 工具里工作，各工具对 frontmatter 字段、工具名称、命令调用语法有差异。手动维护 4 套版本极易出现分叉
- **做了什么 trade-off**：同步脚本带来维护成本（sync 脚本本身有 security 问题），换来跨平台功能一致性
- **反映什么认知模型**：把 AI 工具当「目标平台」来对待，就像 web 开发的 polyfill 思路——有一个理想版本，再为各平台降级适配

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单规范源 + 多平台分发（Canonical Source + Multi-Platform Distribution）」**

一份规范 NL 工件，脚本自动同步到多个目标平台，各平台版本为衍生产物。

模式特征清单：
- 特征 1：`.claude/` 目录是唯一写入点，其他目录只读（只由脚本写入）
- 特征 2：同步脚本处理平台差异（工具名映射、frontmatter 字段增删）
- 特征 3：各平台目录结构相似但命名规则不同（`.claude` 用 `autoresearch/plan.md`，`.opencode` 用 `autoresearch_plan.md`）
- 特征 4：`plugin.json` manifest 只存在于 `claude-plugin/`，不在 `.claude/` 原生目录
- 特征 5：SKILL.md 有 4 份副本，功能等价但适配了各平台语法

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 面向多 AI 工具平台分发的 skill 套件 | ✅ 高度适用 | 规范源确保一致性，sync 处理差异 |
| 个人项目 + 单一工具使用 | ❌ 过度工程 | 4 个目录 + sync 脚本的维护成本不值得 |
| 团队协作 skill 库 | ✅ 适用 | 规范源 + PR review 流程配合良好 |
| 频繁改动的实验性 skill | ⚠️ 慎用 | 每次改动都要 sync，忘记 sync 会导致平台行为不一致 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单规范源分发（本仓库） | uditgoenka/autoresearch | 跨平台一致性有保障 | sync 脚本本身是维护负担和安全面 |
| 单平台单 skill | twostraws/SwiftUI-Agent-Skill | 简单，零维护成本 | 不跨平台 |
| monorepo 多 skill 平铺 | addyosmani/agent-skills | 统一版本管理 | 各 skill 不能独立发布 |

### 2.4 改进空间
1. **当前问题**：sync 脚本用 `python3 -c "... '$DST' ..."` 把 bash 变量插入 Python 字符串，路径含特殊字符会破坏脚本。**改进做法**：sync 脚本用 `sys.argv[1]` 传路径参数，或把 Python 代码写入临时文件再执行。**预期收益**：消除路径注入风险。
2. **当前问题**：20 个 `.claude/commands/` 文件和 `claude-plugin/commands/` 文件全部缺少 `allowed-tools` 声明，导致在权限限制环境下工具访问不可预期。**改进做法**：在 sync 脚本中加一步：为 Claude 版本的命令文件自动注入 `allowed-tools: [Bash, Read, Write, AskUserQuestion]`。**预期收益**：在权限沙箱中稳定运行。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **88/100**（36 个文件加权平均）。

| 文件组 | 当时分数 | 主要问题 |
|---|---|---|
| `.opencode/agents/docs-manager.md` | 55/100 | 缺 `name` 字段、无 model 声明、零示例 |
| `.opencode/commands/autoresearch*.md`（10 个） | 75/100 | 缺 `name` 字段 |
| 4 个平台的 SKILL.md | 88/100 | 嵌入社交星标提示（`gh api starred`） |
| `.claude/commands/autoresearch*.md`（20 个） | 95/100 | 缺 `allowed-tools` 声明 |
| `claude-plugin/.claude-plugin/plugin.json` | 100/100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **子命令路由表** → skill body 开头的 dispatch 表清晰列出所有模式和触发条件，用户一眼看清能力边界 → `.claude/skills/autoresearch/SKILL.md` 第 17 行的 `## Dispatch` 表 → 复杂 skill 都应该有这个路由总览
2. **普适标志位（Universal Flags）** → 用 `--evals`、`--chain`、`--dry-run` 等标志控制跨子命令行为，API 设计一致 → SKILL.md 第 45 行的 `## Universal Flags` 表 → 多子命令的工具应统一 flag 命名
3. **Orchestrator vs Classic 二模式** → 根据输入特征自动选模式（有 Metric 就走 Classic，自然语言目标走 Orchestrator），降低用户心智负担 → SKILL.md 中的 dispatch 逻辑 → 复杂工具值得做自动模式识别
4. **handoff.json 链式交接** → 子命令结果写入 `handoff.json`，下个子命令读取，形成有状态的调用链 → SKILL.md `## Safety Invariants` 第 4 条 → 长链任务应有状态持久化机制
5. **常量集中声明** → `REVIEWER_MODEL`、`MAX_ROUNDS`、`SCORE_THRESHOLD` 等常量在 SKILL.md 顶部统一声明，支持 override → 降低参数散落各处的维护痛苦

### 3.3 当时的缺陷
1. **星标社交提示嵌入 skill**：SKILL.md 里有 `gh api -X PUT /user/starred/uditgoenka/autoresearch` 指令，让 Claude 在每次完成研究后自动给仓库点 Star → 根本原因：作者把增长黑客手段植入工具内部，对用户形成无感知的 GitHub 操作副作用 → 这个设计把 AI 工具变成了社交营销机器，违反了最小副作用原则 → 自查：我的 skill 是否有类似的「顺手」外部操作？
2. **缺少 `allowed-tools`**：20 个 `.claude/commands/` 文件无 `allowed-tools` → 根本原因：作者可能不了解 Claude Code 在受限模式下会对每个工具调用弹出确认对话框 → 自查：我的 bureau skill 全部缺少这个字段
3. **opencode agent name 字段缺失**：`docs-manager.md` 无 `name` 字段，注册依赖文件名推断 → 根本原因：OpenCode 和 Claude Code 的 frontmatter 规范不同，作者复制时遗漏

### 3.4 当时的优化机会
1. 删除或改造 star-repo 社交提示（移出 SKILL.md，改为 command 层的可选确认步骤）
2. 为 20 个 `.claude/commands/` 文件加 `allowed-tools` 声明
3. 为 `.opencode/agents/docs-manager.md` 加 `name` 字段、model 声明、至少 1 个示例

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 星标社交提示 | `grep "starred" .claude/skills/autoresearch/SKILL.md` | ✅ **已删除** | 作者接受了这个反馈，这是最高优先级修复之一 |
| opencode `docs-manager.md` 缺 `name` | `grep "^name:" .opencode/agents/docs-manager.md` | ✅ **已修复**（`name: docs-manager`） | Bug #1 已修复 |
| opencode commands 缺 `name` | `head -3 .opencode/commands/autoresearch_plan.md` | ✅ **已修复**（`name: autoresearch_plan`） | Bug #2 已修复 |
| `.claude/commands/` 缺 `allowed-tools` | `grep "allowed-tools" .claude/commands/autoresearch.md` | ❌ **仍未修复**（20 个文件全部缺失） | Bug #3 持续存在 |

### 4.2 架构演进
与 audit 时相比，新增了 `guide/` 目录（用户文档，非 NL 工件）和几个新子命令（`autoresearch_probe.md`、`autoresearch_improve.md`、`autoresearch_evals.md` 等）。基础架构未变，command 数量从 9 个增加到约 12-13 个，说明作者在持续扩展子命令能力。

### 4.3 新增的可学习模式
`autoresearch_probe.md`（8 personas interrogate requirements until saturation）——「多人格穷举需求」模式：用 8 个不同人格对同一需求进行反复质问直到饱和（没有新问题出现为止），是一种比单次追问更彻底的需求澄清方法。这个模式在 audit 时不存在。

---

## 五、校准

### 5.1 我已经在做对的
1. **子命令路由结构**：gstack 的 SKILL.md 顶部有触发条件和 when-to-invoke 说明，功能类似 autoresearch 的 dispatch 表
2. **常量声明**：bureau 的命令文件把关键参数（如 trust tier 阈值）集中在 frontmatter 或顶部说明中
3. **无社交操作副作用**：我的 skill 不包含任何隐式外部操作指令
4. **状态文件持久化**：bureau 的 `capture/SKILL.md` 有明确的状态写入路径

### 5.2 挑战 / 验证
**挑战**：我之前觉得 `allowed-tools` 是「可写可不写」的装饰性字段——反正 Claude 会用它能用的工具。但 autoresearch 案例提醒：在受限执行环境（CI、GitHub Actions、企业沙箱）中，缺少 `allowed-tools` 会让 Claude Code 对每个工具调用弹出交互式确认，直接破坏自动化场景。这个字段是**功能性必填**，不是装饰。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 SKILL.md 是否缺少 allowed-tools 字段
find /tmp/my-repos/MarkQWu-bureau /tmp/my-repos/MarkQWu-gstack \
  -name "SKILL.md" 2>/dev/null | xargs -I{} sh -c \
  'echo "--- {} ---"; grep -m1 "^allowed-tools" {} || echo "❌ MISSING allowed-tools"'
```
命中后：为每个 skill 添加最小权限的 `allowed-tools` 声明，参考该 skill 实际调用的工具。bureau 的 7 个 skill 全部需要补充。

```bash
# 检查我的 skill 是否有隐式外部操作指令（gh api、curl 外部服务等）
grep -rn -E "gh api|curl.*http|wget" /tmp/my-repos/MarkQWu-bureau \
  /tmp/my-repos/MarkQWu-gstack --include="SKILL.md" 2>/dev/null
```
命中后：评估该操作是否应该移到 command 层并加 user confirmation gate。

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau 的 7 个 SKILL.md 补充 `allowed-tools` 声明
   - **为何可行**：bureau 是 Claude Code plugin，在受限环境部署时 `allowed-tools` 是必填的
   - **第一步**：打开 `bureau/skills/recall/SKILL.md`，在 frontmatter 中加 `allowed-tools: [Read, Glob, Grep]`（recall skill 只需要读操作）→ 同理逐个补充其他 6 个 skill → 预计 15 分钟

2. **想法**：为 gstack 的主 command 文件加 dispatch 路由总览表（仿 autoresearch 的 `## Dispatch` 表）
   - **为何可行**：gstack 有 20+ 个 skill，新用户不知道从哪里入手；一个路由总览表能大幅提升可发现性
   - **第一步**：在 `gstack/SKILL.md`（顶层）增加 `## Skill 索引` 表格，列出每个 skill 的调用触发词和一句话用途 → 预计 30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 uditgoenka/autoresearch 的核心目的**：自主目标迭代循环——修改→验证→保留/丢弃，直到达成可测量目标

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是 Claude Code 多命令工具套件；都有子命令路由 | gstack 是工程工作流工具，autoresearch 是研究迭代工具 | 中 |
| MarkQWu/bureau | 低 | 同为 Claude Code plugin | bureau 是知识管理，目标完全不同 | 低 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都是对话内容分析工具 | echo-sleuth 挖掘历史对话，autoresearch 做前向迭代研究 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `allowed-tools` 完全缺失 | `grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | **bureau 7/7 SKILL.md 全部缺失** | 高 |
| `allowed-tools` 字段存在但为空 | `grep "allowed-tools:$" gstack/*/SKILL.md` | gstack 多个 skill 有 `allowed-tools:` 字段但值为空（如 `setup-deploy`） | 中 |

**命中后的具体行动建议**：
- `bureau/skills/recall/SKILL.md` → 添加 `allowed-tools: [Read, Glob, Grep]` → 2 分钟
- `bureau/skills/capture/SKILL.md` → 添加 `allowed-tools: [Read, Write, Edit, Bash]` → 2 分钟
- `bureau/skills/compile/SKILL.md` → 添加 `allowed-tools: [Read, Write, Edit, Glob, Grep]` → 2 分钟
- `gstack/setup-deploy/SKILL.md` → 检查实际调用了哪些工具，填充 `allowed-tools:` 空值 → 5 分钟

### 8.3 别人的更优方案

1. **领域**：子命令分发路由表
   - **本案例做法**：SKILL.md 顶部的 `## Dispatch` 表列出所有模式和触发条件，用户一眼看清（`.claude/skills/autoresearch/SKILL.md` 第 17 行）
   - **我的项目现状**：gstack 无统一的 skill 索引，用户需要逐一打开文件才知道每个 skill 做什么
   - **如何借鉴**：在 gstack 根目录 `SKILL.md` 中加 `## Skill 索引` 表格

2. **领域**：状态检查点（checkpoint recovery）
   - **本案例做法**：`REFINE_STATE.json` 在每个 phase 边界持久化状态，session 中断后可恢复 → `research-refine/SKILL.md` 第 70 行
   - **我的项目现状**：echo-sleuth 的分析流程无状态持久化，中断后需从头开始
   - **如何借鉴**：在 echo-sleuth 的 command 里加 `state.json` 写入步骤，记录当前分析阶段

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：sync 脚本的安全写法
- **我的做法**：bureau 的 `scripts/` 目录下的脚本用参数传递路径，不把 bash 变量插入 Python 字符串
- **本案例做法**：`scripts/sync-codex.sh` 用 `python3 -c "... '$DST' ..."` 把 bash 变量嵌入 Python 字符串，存在路径注入风险（audit finding #1 MEDIUM）
- **意义**：这是一个我已经无意识做对的安全实践，值得显式化为「脚本路径参数传递规范」写入 CONTRIBUTING.md

---

## 八、术语表

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md / command 文件 frontmatter 中的字段，声明该 skill 运行时被允许调用的 Claude Code 工具列表（如 `[Bash, Read, Write]`）。在权限限制环境（CI、企业沙箱）中，缺少此字段会导致每个工具调用触发交互式确认，破坏自动化流程。

### <a name="canonical-source"></a>规范源（canonical source）
> 多个副本中，被指定为「唯一权威版本」的那个文件。其他副本是衍生产物，只由脚本生成，不手动修改。修改规范源 → 运行 sync 脚本 → 衍生版本自动更新。

### <a name="dispatch"></a>dispatch 路由
> 主命令根据输入特征（是否包含 `Metric:` 字段、参数格式等）自动决定调用哪个子命令或子模块的逻辑。类似后端的路由层——输入进来，先判断走哪条路，再执行具体逻辑。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块。Claude Code 先解析 frontmatter 才知道如何注册和调用该 skill/command。
