# code-yeongyu/oh-my-openagent — 学习案例

**仓库**：https://github.com/code-yeongyu/oh-my-openagent
**Stars**：51912 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-31（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `examples-driven`, `experience-accumulation`, `security-gate`

**xiaolai 案例**：[../../../case-studies/2026-05-07-code-yeongyu-oh-my-openagent.md](../../../case-studies/2026-05-07-code-yeongyu-oh-my-openagent.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

oh-my-openagent（原名 oh-my-opencode，已更名）是目前 Claude Code 生态中**Star 数最高的 agent 配置框架之一**，以 51,912 stars、4,577 forks 的体量远超其他同类项目。项目由 YeonGyu-Kim 维护，定位是"the best agent harness"，提供浏览器控制、Git 工作流、GitHub 分类、前端 UI/UX 和代码安全研究等内置 skill，以及一套自动生成的 `AGENTS.md` 文档系统覆盖仓库所有 agent 目录。

关键事实：
- 51,912 stars（截至审查日，生态中罕见的超大型项目）
- 技术栈：TypeScript + OpenAgent runtime（取代 OpenCode）
- 采用 npm 分发，含 postinstall 脚本（安全门有记录）
- 自动生成的 AGENTS.md 遍布仓库各目录层级（20+ 个）
- 包含 posthog-node 遥测依赖（有隐私含义）

### 1.2 架构剖析

```
oh-my-openagent/
├── src/
│   ├── agents/                         ← 11 个 agent 定义（含子 agent）
│   │   ├── AGENTS.md                   ← 自动生成总索引（已修复 frontmatter）
│   │   ├── hephaestus/AGENTS.md        ← 已修复
│   │   ├── prometheus/AGENTS.md        ← 已修复
│   │   └── sisyphus/AGENTS.md          ← 已修复
│   ├── features/
│   │   └── builtin-skills/             ← 工具型 skill（browser、git、frontend、security）
│   │       ├── agent-browser/SKILL.md
│   │       ├── dev-browser/SKILL.md    ← broken references 已修复（references/ 目录现在存在）
│   │       ├── frontend-ui-ux/SKILL.md
│   │       ├── git-master/SKILL.md
│   │       └── security-research/SKILL.md ← 审查后新增
│   └── hooks/atlas/                    ← TypeScript hook 源码（非 NL 工件）
├── .opencode/                          ← 原 opencode 配置
│   └── skills/                         ← 编排型 skill
│       ├── github-triage/SKILL.md
│       ├── work-with-pr/SKILL.md
│       └── pre-publish-review/SKILL.md
├── .agents/                            ← 新增，agent 配置目录
├── postinstall.mjs                     ← npm 安装脚本（execSync 安全门）
└── package.json                        ← 含 posthog-node 遥测、trustedDependencies
```

**文件类型分布**：20+ 个 AGENTS.md（自动生成）/ 5个 builtin-skill SKILL.md / 3个 opencode SKILL.md / 1个 package.json

**编排关系**：`builtin-skills/` 提供工具层（单一功能、陈述性）；`.opencode/skills/` 提供编排层（多步骤流程，调用工具层 skill）。两层分工明确，互补而不重叠。

**跨件契约**：AGENTS.md 由工具链自动生成（带 `**Generated:**` 时间戳），frontmatter 在 PR #3578 后统一修复；`dev-browser/references/` 已创建并包含 installation.md 和 scraping.md。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**分层架构 + 自动文档化**。工具层 skill 描述"能力"，编排层 skill 描述"流程"，两层用目录物理分离。AGENTS.md 的自动生成保证文档始终和代码同步。
- **解决什么问题**：大型 agent 仓库的文档腐烂问题——手写 AGENTS.md 很快过时。自动生成解决了这个问题。
- **做了什么 trade-off**：自动生成的 AGENTS.md 缺少语义化 frontmatter，导致 NL 工具无法索引——这是本次审查的核心 bug 来源。通过 PR #3578 在生成模板中加入 frontmatter，一次性修复所有文件。
- **反映什么认知模型**：作者把 agent 定义视为**源码**（自动生成、版本控制、可 diff），而非手工文档。这和"配置即代码"（CaC）理念一致。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「工具层 + 编排层双轨 skill 架构」

核心思路：把 skill 分为两类：`builtin-skills/`（工具型，描述单一能力，如"如何操控浏览器"）和 `.opencode/skills/`（编排型，描述多步骤业务流程，调用工具型 skill）。两类 skill 物理分离，职责清晰。

特征清单：
- 特征 1：工具型 skill 无状态、可复用、无流程依赖
- 特征 2：编排型 skill 有明确阶段划分和退出条件
- 特征 3：AGENTS.md 自动生成，前端固定模板 + frontmatter 注入
- 特征 4：postinstall 脚本只做版本探测（无网络下载），安全面最小化
- 特征 5：安全依赖（trustedDependencies）列表显式声明，不用全局信任

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型多 agent 框架（20+ agent/skill） | ✅ 高度适用 | 双轨分离是唯一可扩展的结构 |
| 需要自动文档化的团队项目 | ✅ 适用 | 自动生成 AGENTS.md 消除文档腐烂 |
| 个人单一功能工具 | ❌ 过度设计 | 双轨架构对小项目是过度复杂 |
| 需要严格输出格式的工具（如 CI 校验） | ⚠️ 谨慎 | 审查时 builtin-skills 缺 output 格式声明 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 双轨 skill（本仓库） | oh-my-openagent | 工具层可复用，编排层聚焦流程 | 两个目录，初学者不知道写哪里 |
| 备选 A：平铺单层 skill | 多数 skill 仓库 | 简单直接，全在一处找 | 能力和流程混杂，难以复用 |
| 备选 B：技能注册中心 | 无成熟案例 | 动态组合，最灵活 | 需要 runtime 支持，复杂度高 |

### 2.4 改进空间

1. **当前问题**：`builtin-skills/` 中多个 SKILL.md 缺 `output:` 格式声明，AI 调用后输出格式不确定。**改进做法**：在每个 builtin-skill 的 SKILL.md 加 `## Output Format` 节，描述调用成功后应产生的输出结构（如截图路径 / JSON 摘要 / 行数报告）。**预期收益**：编排层 skill 可以确定性地解析工具层 skill 的输出。

2. **当前问题**：遥测依赖 `posthog-node` 缺少 opt-in 文档，用户不知道启用了哪些数据收集。**改进做法**：在 README 加"Data Collection"节，说明收集什么、如何禁用；提供 `OHMYOA_TELEMETRY=0` 等环境变量。**预期收益**：减少隐私疑虑，增加用户信任。

3. **当前问题**：`pre-publish-review/SKILL.md` 中"significant findings"和"minor issues"是模糊量词，AI 无法一致执行。**改进做法**：定义为"≥2个 HIGH 级别 finding 为 significant，≤1个为 minor"这类可枚举的条件。**预期收益**：AI 行为可预测、可测试。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **74/100**（勉强通过 70 分阈值）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `src/agents/AGENTS.md`（×4） | 50 | 缺 YAML frontmatter（name/description） |
| `src/hooks/atlas/tsconfig.json` | 50 | 非 NL 工件被错误纳入扫描 |
| `src/features/builtin-skills/dev-browser/SKILL.md` | 80 | 两个 broken 引用（references/ 子目录不存在）+ 缺 output 格式 |
| `.opencode/skills/github-triage/SKILL.md` | 90 | 重复 `---` 分隔符 |
| `.opencode/skills/pre-publish-review/SKILL.md` | 94 | "significant" ×2、"minor" ×1 模糊量词 |
| `src/features/builtin-skills/git-master/SKILL.md` | 98 | "suitable" ×1 |

### 3.2 当时值得借鉴的模式

1. **双轨 skill 架构**（工具层 + 编排层）：物理分离两类 skill，职责清晰，可复用性高。→ 如何借鉴：`claude-for-legal` 可以把"能力型"skill（如提取条款）和"流程型"skill（如起草合同的完整流程）分目录存放。

2. **自动生成 AGENTS.md**：文档从代码生成，始终和 agent 定义同步，无手工维护负担。→ 如何借鉴：大型 skill 套件可以写生成脚本，从 frontmatter 提取 name+description 自动生成索引。

3. **trustedDependencies 显式列表**：`@code-yeongyu/comment-checker` 等显式列入 `trustedDependencies`，没有使用"全部信任"的宽泛策略。→ 如何借鉴：凡是有 npm 依赖的项目，都应明确而不是隐式信任安装脚本。

4. **postinstall 最小权限**：`postinstall.mjs` 只做 `execSync("opencode --version")` 版本探测，不做网络下载或文件系统写入。→ 如何借鉴：在自己的安装脚本中，只做"验证存在性"，不做"获取资源"。

### 3.3 当时的缺陷

1. **AGENTS.md 缺 frontmatter**：4个自动生成的文件都没有 `name` 和 `description`，对 NL 工具完全不可见。→ 根本原因：自动生成脚本的模板没有包含 frontmatter 模块，是生成器设计遗漏而非手工漏写。→ 自查：如果有自动生成的 NL 工件，检查生成器模板是否包含 frontmatter 块。

2. **dev-browser broken 引用**：`SKILL.md` 引用 `references/installation.md` 和 `references/scraping.md`，但文件不存在。→ 根本原因：作者在写 SKILL.md 时预先规划了参考文件，但忘记创建，或创建后被误删。→ 自查：`grep -rn "references/" ~/.claude/skills/*/SKILL.md | xargs -I{} check-file`。

3. **模糊量词未清理**："significant findings"、"minor issues"、"suitable target commit"都是 AI 无法一致执行的标准。→ 根本原因：作者写 skill 时使用了人类能理解但机器无法量化的语言。→ 自查：`grep -rn "significant\|suitable\|reasonable\|appropriate" ~/.claude/skills/`。

### 3.4 当时的优化机会

1. 为所有 AGENTS.md 生成模板加 frontmatter（修复后影响所有未来生成的文件）
2. 创建 `dev-browser/references/installation.md` 和 `references/scraping.md`
3. 将 `pre-publish-review/SKILL.md` 的模糊阈值替换为可枚举的判断条件

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| AGENTS.md 缺 frontmatter（×4） | `head -5 src/agents/AGENTS.md` | **已修复**：现在有 `name: agents-directory`、`description:...`（PR #3578 合并） | PR 已被接受并合并，新增生成的文件也会带 frontmatter |
| dev-browser broken 引用 | `ls src/features/builtin-skills/dev-browser/references/` | **已修复**：references/ 目录现存在，含 installation.md 和 scraping.md | 文档补全，broken reference 消除 |
| pre-publish-review 模糊量词 | `grep "significant\|minor" .opencode/skills/pre-publish-review/SKILL.md` | 暂无法验证（需深入读文件） | 需 follow-up 检查 |

### 4.2 架构演进

- **新增 security-research skill**：`src/features/builtin-skills/security-research/SKILL.md` 是审查后新增的，说明作者在扩展工具层覆盖面
- **新增 `.agents/` 目录**：顶层增加了专门的 agent 配置目录，与 `.opencode/` 并列，说明项目开始支持更多 agent runtime
- **AGENTS.md 密度增加**：自动生成的 AGENTS.md 从 4 个子目录扩展到 20+ 个（src/agents/sisyphus-junior/ 等），说明 agent 体系在快速扩张
- **名称变更**：oh-my-opencode → oh-my-openagent，表明项目从 OpenCode 绑定变为 agent runtime 无关

### 4.3 新增的可学习模式

**新增 security-research skill**：在 builtin-skills 中加入安全研究能力，说明项目的工具层在从"开发工作流"向"安全工作流"扩展。这是**能力层横向扩展**的典型做法：不改变架构，只在工具层加新 skill。整个系统的边界得到扩展，而无需修改编排层的结构。

---

## 五、校准

### 5.1 我已经在做对的

1. **`echo-sleuth-for-claude` agents 有 frontmatter**：recall.md、analyze.md 等 agent 文件已有 name/description，和 PR #3578 修复后的 oh-my-openagent 水准一致。
2. **`claude-for-legal` 的 skill 有 references/ 结构**：ip-legal 等区域 skill 有实际存在的 references/ 文件，无 broken 引用。
3. **`drama-workshop-skills` 的 SKILL.md 无模糊量词**：script-quality 等节的标准是可量化的（"每集 3-5 个爽点"而非"适当数量的爽点"）。

### 5.2 挑战 / 验证

**挑战**：这个案例挑战了"只要文件存在就够了"的假设。oh-my-openagent 的 AGENTS.md 文件物理存在、内容正确，但缺少 frontmatter 导致对机器完全不可见——**文件对人可见不等于对工具可见**。这个区分在我自己的项目中可能也存在：有没有文件我认为"写了"但实际上 frontmatter 不完整？

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我所有 agent/AGENTS.md 类文件是否有 frontmatter
find /tmp/my-repos/MarkQWu-* -name "AGENTS.md" -o -name "*.md" -path "*/agents/*" 2>/dev/null \
  | xargs grep -L "^---" 2>/dev/null

# 检查所有 SKILL.md 中是否有 broken 引用（references/ 文件不存在）
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" | while read f; do
  dir=$(dirname "$f")
  grep -oP 'references/\K[^\)]+\.md' "$f" 2>/dev/null | while read ref; do
    [[ ! -f "$dir/references/$ref" ]] && echo "BROKEN: $f -> references/$ref"
  done
done

# 检查模糊量词
grep -rn "significant\|minor issue\|suitable\|reasonable" \
  /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null
```

命中 broken 引用后：创建对应的 references/ 文件，即使只有一句"TODO: 待补充"也比 broken 好。
命中模糊量词后：将每个量词替换为可枚举的判断条件。

### 6.2 灵感 → 实施路径

1. **想法**：为 `claude-for-legal` 引入双轨 skill 架构——把"能力型"（提取条款、识别风险）和"流程型"（起草合同完整流程）分目录。
   - **为何可行**：claude-for-legal 当前 skill 已有隐式两类，但混在一起；分开后可独立复用能力型 skill。
   - **第一步**：在 commercial-legal/ 中创建 `tools/` 子目录，把单一能力型 skill 迁移进去。预计 30 分钟。

2. **想法**：为 `echo-sleuth-for-claude` 的 agent 索引添加自动生成脚本，生成类似 oh-my-openagent 的 AGENTS.md。
   - **为何可行**：echo-sleuth 有 5 个 agent，手写索引已经落后于实际状态。
   - **第一步**：写 `scripts/gen-agents-index.sh`，遍历 agents/*.md 提取 name+description 生成 agents/AGENTS.md。预计 20 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 code-yeongyu/oh-my-openagent 的核心目的**：提供跨 AI runtime 的通用 agent 配置框架，内置浏览器、git、PR 等工具型 skill

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是 Claude Code plugin，有 agent + skill 结构 | 功能聚焦（会话挖掘 vs 通用开发工具） | 高 |
| MarkQWu/claude-for-legal | 低 | 都有多 skill 套件 | 垂直领域（法律 vs 通用开发）；法律不需要浏览器控制 | 低 |
| MarkQWu/drama-workshop-skills | 无 | 都是 skill 仓库 | 创意内容 vs 开发工具；目的无交集 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| AGENTS.md / agent 文件缺 frontmatter | `grep -L "^---" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md` | **未命中**：recall.md 等均有 frontmatter | 无 |
| Broken 引用（references/ 不存在） | `find /tmp/my-repos/MarkQWu-* -name "SKILL.md" -exec grep "references/" {} \;` | claude-for-legal 的部分 SKILL.md 引用 references/ 文件，需人工核查 | 中 |
| 模糊量词 | `grep -rn "significant\|suitable" /tmp/my-repos/MarkQWu-*/*/SKILL.md` | 未系统检查，建议运行上述命令核查 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/claude-for-legal` 中 references/ 引用 → 运行 broken-ref 检查脚本（见 §6.1），对命中的文件创建最小化占位文件，5-10 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：自动生成 agent 索引文档
   - **本案例做法**：AGENTS.md 由工具链自动生成，带 `**Generated:**` 时间戳，和代码始终同步（`code-yeongyu-oh-my-openagent/src/agents/AGENTS.md`）
   - **我的项目现状**：`echo-sleuth-for-claude` 无 AGENTS.md 类索引；用户需要手动查看 agents/ 目录
   - **如何借鉴**：写 `scripts/gen-agents-index.sh`，从 agents/*.md frontmatter 提取 name+description 生成索引。改动 1 个文件，20 分钟可完成

2. **领域**：工具层 skill 的 output 格式声明
   - **本案例做法**：security-research/SKILL.md（新增）有明确的 output 格式声明（虽然 builtin-skills 老文件部分缺失）
   - **我的项目现状**：`drama-workshop-skills` 的 SKILL.md 有输出格式规范（"分镜格式：每集 N 场，每场含...）
   - **如何借鉴**：echo-sleuth 的 agents 缺少明确的 output 格式声明，可参照 drama-workshop 补充

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：量化标准代替模糊量词
- **我的做法**：`drama-workshop-skills/short-drama/SKILL.md` 的质量标准使用可量化的数字（"每集 3-5 个爽点"、"付费卡点每 10 集不少于 2 个"）
- **本案例做法**（弱在哪）：`pre-publish-review/SKILL.md` 用"significant findings"这类模糊量词，AI 无法一致执行
- **意义**：量化标准是 drama-workshop-skills 的显著优势，也是它经过多次迭代磨合出来的。这是值得对外宣传的亮点。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（`name`、`description`、`model` 等）。NL 工具扫描 SKILL.md 或 AGENTS.md 时首先解析 frontmatter；缺少 frontmatter 的文件对工具不可见，即使内容再好也无法被自动索引和发现。

### <a name="trustedDependencies"></a>trustedDependencies
> npm 配置字段，列出允许自动执行 postinstall 脚本的包名。未列出的包的安装脚本会被暂停，等待用户手动批准。oh-my-openagent 显式列出 `@code-yeongyu/comment-checker`，而非默认全部信任。

### <a name="postinstall"></a>postinstall（安装后脚本）
> npm 包在 `npm install` 完成后自动执行的脚本（配置在 package.json 的 `"scripts"."postinstall"` 字段）。用于下载二进制、生成配置等初始化操作。oh-my-openagent 的 `postinstall.mjs` 只做版本探测（`execSync("opencode --version")`），是安全面最小化的示范。

### <a name="broken-reference"></a>broken reference（断裂引用）
> 一个文件引用了另一个文件的路径，但被引用的文件不存在。在 SKILL.md 中，`references/installation.md` 如果不存在，用户点击链接或 AI 尝试读取时会遇到 404 / 文件不存在错误。

### <a name="vague-quantifier"></a>vague quantifier（模糊量词）
> 如"significant"、"minor"、"suitable"、"appropriate"这类词，在不同人/不同 AI 执行时可能产生不同结果，因为它们没有明确的数量或条件边界。NL 编程规范（Rule 4）要求用可验证的具体标准代替模糊量词。
