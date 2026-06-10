# AgriciDaniel/claude-seo — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-seo
**Stars**：未记录（Upstream Pool）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-10（基于当前 HEAD）
**主题标签**：`examples-driven`, `security-gate`, `manifest-discipline`, `single-purpose`, `cross-reference`

---

## 一、理解

### 1.1 仓库概览

AgriciDaniel/claude-seo 是一个垂直领域深度 Claude Code 插件，专注于 SEO（搜索引擎优化）全流程自动化。与覆盖多领域的插件集合不同，该仓库将所有精力集中在 SEO 这一条赛道上，以"深度"换"广度"。

规模数据（2026-04-12 审计快照，HEAD v1.9.8 状态）：

- **Skills**：25+ 个子 skill 目录，覆盖 seo（核心路由）、seo-cluster（语义聚类）、seo-sxo（搜索体验优化）、seo-drift（SEO 漂移监控）、seo-ecommerce（电商 SEO）、seo-hreflang（国际化）等专化场景
- **Agents**：18 个（15 个核心 + 3 个 extension agents），涵盖 seo-technical、seo-content、seo-schema、seo-geo、seo-local、seo-maps 等角色
- **Extensions**：8 个可选扩展（banana、dataforseo、firecrawl、ahrefs、bing-webmaster、seranking、unlighthouse、profound），各带独立 `install.sh`
- **整体 NLPM 得分**：**94/100（Gold 级）**，Security: **REVIEW**（3 个 HIGH findings）
- **Exemplar 状态**：被 NLPM 选为高分标本，exemplifies R04、R05、R07、R08、R12、R13、R30

关键背景：仓库有社区贡献者参与维护——CHANGELOG.md 记录了 Lutfiya Miller 添加 seo-cluster、Florian Schmitz 添加 seo-sxo、Dan Colta 添加 seo-drift、Matej Marjanovic 添加 seo-ecommerce，形成了真实的开源协作模型。当前最新版本为 v1.9.8，相比审计时的 v1.6 增加了 4 个新 extension 和多个专化 sub-skill。

### 1.2 架构剖析

**目录结构（HEAD v1.9.8）：**

```
claude-seo/
  CLAUDE.md                    ← 详细架构文档
  AGENTS.md                    ← agent 总览
  CHANGELOG.md                 ← 社区贡献记录
  CITATION.cff                 ← 学术引用元数据
  .claude-plugin/plugin.json   ← v1.9.8
  marketplace.json
  agents/                      ← 18 个 agent 文件
    seo-technical.md
    seo-content.md
    seo-schema.md
    seo-geo.md
    seo-local.md
    seo-maps.md
    ...（共 18 个）
  skills/                      ← 25+ 个子 skill 目录
    seo/SKILL.md               ← 核心 orchestrator skill（路由表）
    seo-cluster/SKILL.md       ← 语义聚类
    seo-sxo/SKILL.md           ← 搜索体验优化
    seo-drift/SKILL.md         ← SEO 漂移监控
    seo-ecommerce/SKILL.md     ← 电商 SEO
    seo-hreflang/SKILL.md      ← 国际化
    ...（共 25+）
  extensions/                  ← 8 个可选 addon
    dataforseo/install.sh      ← 带 credential 注入逻辑
    banana/install.sh
    firecrawl/install.sh
    ahrefs/                    ← 新增
    bing-webmaster/            ← 新增
    seranking/                 ← 新增
    unlighthouse/              ← 新增
  scripts/                     ← 21 个 Python 辅助脚本
  hooks/
    hooks.json                 ← 限制为 HTML 文件触发
    validate-schema.py
  schema/templates.json
```

**三层架构实现**：

```
directive 层    → CLAUDE.md 定义整体行为规范
orchestration 层 → skills/seo/SKILL.md 做请求路由（显示路由表）
execution 层   → 各专化 sub-skill（seo-cluster, seo-drift 等）执行具体任务
```

这是一个典型的"单 skill 入口 + 多 sub-skill 展开"设计：用户触发 `seo` skill，orchestrator 根据用户意图路由到最合适的专化 skill。

### 1.3 设计思路 / 方法论

**约定优于配置**：25 个 sub-skill 全部遵循相同的文件结构——YAML frontmatter（name、description、version、triggers on 触发列表）、能力说明段落、限制声明。没有任何一个例外，这是集体纪律，而非个人风格。

**单职责原则**：每个 sub-skill 只做一件事。`seo-cluster` 只处理语义聚类，`seo-drift` 只监控 SEO 漂移，`seo-hreflang` 只处理国际化标签。职责边界在 description 中明确声明，包括"不做什么"。

**Orchestrator Skill 路由表**：`skills/seo/SKILL.md` 的 description 不仅描述功能，还明确展示一张路由分发表——哪类请求转到哪个 sub-skill。这让 Claude 在选择 skill 时有确定性依据，而不是依赖模型推理。

**双重触发策略**（R04 核心实现）：每个 skill 在 description 中同时包含任务描述和 `Triggers on:` 关键词列表，例如：

```
description: "Full website SEO audit...Use when user says audit, full SEO
check, analyze my site, or website health check."
```

这一设计让 skill 既能被语义理解触发，也能被精确关键词匹配触发，覆盖了用户表达的最大宽度。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

本仓库体现了一个值得独立命名的架构模式：

**模式：「领域专精大型 Plugin + 可选扩展层」**

结构定义：

```
一个 Git 仓库
  ↓ 核心层
  orchestrator skill（seo/SKILL.md）— 路由入口
  N 个专化 sub-skills（各司其职）
  M 个专化 agents（执行具体操作）
  ↓ 扩展层（可选）
  K 个 extension（通过独立 install.sh 按需安装）
  每个 extension 包含额外的 skill + agent 文件
  ↓ 共同构成
  深度垂直领域的完整 Claude Code 工具链
```

与"垂直插件市集"（如 wshobson/agents）的核心区别：本模式所有组件服务于同一个领域，彼此协作；市集模式的组件相互独立，彼此无关。

### 2.2 适用场景

**适用于**：

- 需要深度领域知识的垂直应用（SEO、法律文书、医学诊断、财务分析）
- 领域内有明确的子任务分类，且每个子任务差异足够大到需要专化 skill
- 有可选的第三方 API 集成，用户可能只需要其中几个
- 社区贡献者可能添加新子领域，需要清晰的扩展接口

**不适用于**：

- 通用工具（应该是多个独立 skill，而非单一深度垂直 plugin）
- 子任务之间有强顺序依赖、必须全部安装才能使用（此时应用 pipeline 而非 skill-router 模式）
- 领域知识变化极快，难以维护深度内容（会导致 skill 内容快速过时）

### 2.3 与其他架构对比

| 架构 | 代表仓库 | 优势 | 劣势 |
|------|---------|------|------|
| 领域专精 + 扩展层（本仓库）| AgriciDaniel/claude-seo | 深度高，子任务划分清晰，扩展性强 | 不同子任务质量难以统一，extensions 引入安全风险 |
| 垂直插件市集 | wshobson/agents | 覆盖广，用户按需选择 | 插件间质量参差，frontmatter 纪律难统一 |
| 单精品插件 | 2389-research/review-squad | 质量高（96/100），维护专注 | 覆盖面极小，扩展成本高 |
| skill-only 仓库 | upstash/context7 | 平台无关，可跨插件引用 | 无 slash command 入口，靠自然语言触发 |

### 2.4 可迁移的设计决策

1. **Orchestrator Skill 的路由表写法**：在 description 中内嵌简明路由表，列出"用户意图 → 转发到哪个 sub-skill"。这一写法成本极低（5-10 行），但对 Claude 的工具选择决策提升显著。

2. **Extension 的独立 install.sh 模式**：每个 extension 有自己的安装脚本，核心 plugin 不依赖任何 extension。用户可以只安装核心，或按需叠加 extension。这比"一次性安装所有依赖"的方式更灵活，也让安全审计范围更可控。

3. **版本号在 SKILL.md 中声明**：每个 skill 文件头部有 `version:` 字段（审计时 v1.6.1，core v1.8.2），可以通过工具自动检测 stale extension skill 问题——这是机械可检测的 bug 类型。

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

整体 NL 得分：**94/100（Gold 级）**

代表性文件得分明细：

| 文件路径 | 得分 | 主要扣分原因 |
|---------|------|------------|
| skills/seo/SKILL.md | 100/100 | — |
| skills/seo-cluster/SKILL.md | 100/100 | — |
| skills/seo-drift/SKILL.md | 100/100 | — |
| skills/seo-ecommerce/SKILL.md | 100/100 | — |
| agents/seo-technical.md | 85/100 | 无 examples（-15） |
| agents/seo-content.md | 85/100 | 无 examples（-15） |
| agents/seo-schema.md | 85/100 | 无 examples（-15） |
| extensions/dataforseo/agents/seo-dataforseo.md | 80/100 | 无 examples（-15），无 model 声明（-5） |

**Skill 均分：100/100 | Agent 均分：82-85/100 | 整体：94/100**

Skills 全部 100 分是极少见的成就——意味着 25 个 skill 文件全部通过了 NLPM 的所有规则检查，无任何可机械检测的问题。

### 3.2 核心亮点（Exemplar 认证依据）

**R04：触发器设计（双重触发策略）**

`skills/seo/SKILL.md` 的 description 实现了教科书级的双重触发：

```markdown
description: "Full website SEO audit...Use when user says audit,
full SEO check, analyze my site, or website health check."
```

同时，description 中的 `Triggers on: SEO, audit, schema...` 列出 13 个关键词，确保各种用户表达都能命中。这是 R04 的标准实现，也是整个 skills 层 100 分的核心原因。

**R07：Hooks 精准触发**

`hooks/hooks.json` 配置限制 `validate-schema.py` 只对 HTML 文件触发，而非所有文件写入：

```json
{
  "matcher": "*.html",
  "hooks": [...]
}
```

这避免了"schema 验证器响应非 HTML 写操作"的无意义触发，也降低了 hook 执行的安全面。

**R12：SSRF 保护一致实现**

21 个 Python 辅助脚本（`scripts/` 目录）全部包含 `validate_url()` 函数，在发出任何 HTTP 请求前做 SSRF 防护检查。这是一致性 > 完美性的体现——没有任何一个 network-facing 脚本跳过了这个检查。

**R13：3 层架构明确分离**

CLAUDE.md 明确标注了 directive/orchestration/execution 三层边界，并说明了每层的职责范围。这不是注释，而是架构契约。

### 3.3 当时缺陷

**缺陷一：Agent Examples 全部缺失（-15 × 15 = 最大单一扣分来源）**

15 个核心 agent 全部没有 `## Examples` 章节。这是 94/100 而非 100/100 的最大原因。

根本原因分析：Skills 的内容质量（description、triggers、constraints）作者显然投入了大量精力，但 agent 文件的 examples 章节可能被认为是"可选补充"而非"必填项"。这种认知偏差在 skill-heavy 的深度插件中很常见——作者把精力花在 skill 内容深度上，却忽视了 agent 的交互质量。

代价：每个 agent 少 15 分意味着 Claude 在使用该 agent 时缺少了"什么是好的输入/输出"的锚点，可能导致输出格式不稳定。

**缺陷二：Extension Agent 缺少 Model 声明（-5 × 2）**

`extensions/dataforseo/agents/seo-dataforseo.md` 和 `extensions/banana/agents/seo-image-gen.md` 均无 `model:` 字段声明。Extension agent 执行的任务（API 调用、图像生成）对模型能力要求不同，没有声明 model 意味着使用默认模型，可能导致性能或成本问题。

**缺陷三：Python Code Injection（Security HIGH × 3）**

这是最严重的缺陷，见第三章 3.4 节详述。

### 3.4 安全发现（Security: REVIEW，HIGH × 3）

3 个 HIGH findings 全部属于同一种攻击模式——**Triple-Quote Injection（三重引号注入）**：

| Extension | 文件 | 代码行 | 注入变量 |
|-----------|------|--------|---------|
| dataforseo | extensions/dataforseo/install.sh line 103-106 | `username = '''${DFSE_USERNAME}'''` | DFSE_USERNAME, DFSE_PASSWORD |
| banana | extensions/banana/install.sh line 115 | `GOOGLE_AI_API_KEY = '''${GOOGLE_AI_API_KEY}'''` | GOOGLE_AI_API_KEY |
| firecrawl | extensions/firecrawl/install.sh line 88 | `FIRECRAWL_API_KEY = '''${FIRECRAWL_API_KEY}'''` | FIRECRAWL_API_KEY |

**攻击原理**：install.sh 用 shell 变量替换直接将用户提供的 API key/credential 插入 Python 源码字符串。如果攻击者控制了这些环境变量的值（例如通过 `.env` 污染、supply-chain attack 或社会工程），可以用 `'''` 关闭三重引号后注入任意 Python 代码，在安装时以执行 install.sh 的用户权限执行——这是 RCE（远程代码执行）风险。

**正确做法**：credential 应该在运行时通过 `os.environ.get()` 从环境变量读取，而非在安装时硬编码进源码文件。

### 3.5 假设修复路径

若给 top 5 agent 各添加 2-3 个 examples，预计总分从 94 → 98+：

```
当前：94/100（-6 来自 agent examples 缺失的平均扣分）
修复后：98-99/100（保留 -1~-2 的其他微小扣分）
```

安全修复不影响 NLPM 质量得分，但 Security: REVIEW → Security: CLEAR 对社区信任度有实质影响。

---

## 四、现在 vs 过去对比（2026-04-12 → 2026-06-10）

### 4.1 未修复问题（PERSISTS）

**安全 Bug 持续存在**：

对 HEAD 的 spot-check 确认，`extensions/dataforseo/install.sh` line 116-117 仍然使用 triple-quote injection 模式：

```bash
username = '''${DFSE_USERNAME}'''
password = '''${DFSE_PASSWORD}'''
```

这意味着从 2026-04-12 审计至 2026-06-10，将近两个月，3 个 HIGH 安全 finding 均未被修复。可能原因：作者未关注 NLPM 审计报告；或认为 extension install.sh 属于"用户自己运行的脚本"，风险可接受。无论如何，这是一个持续的可利用漏洞。

**Agent Examples 仍为零**：

```bash
grep -l "## Examples" agents/*.md
# 输出：0 个匹配
```

18 个 agent 文件（从审计时的 15 个增加到 18 个，新增了 3 个）依然全部没有 `## Examples` 章节。v1.6 到 v1.9.8 的 8 个版本迭代中，从未有任何一个 agent 被补充 examples。这已经不是遗漏，而是作者的显式选择——或者说，是对 agent examples 价值的系统性低估。

### 4.2 新增进展（v1.6 → v1.9.8）

**功能扩展显著**：

| 维度 | 审计时（v1.6）| HEAD（v1.9.8）| 变化 |
|------|------------|--------------|------|
| 核心 agents | 15 | 18 | +3 |
| Extensions | 3 | 8 | +5 |
| Sub-skills | ~20 | 25+ | +5+ |
| Contributors | 1 | 5+ | 真实社区 |

新增 sub-skills（seo-cluster、seo-sxo、seo-drift、seo-ecommerce）均由社区贡献者添加，并通过了 CLAUDE.md 的 skill 规范要求——这说明单职责 + orchestrator 路由的架构设计成功降低了贡献门槛。

**Extensions 生态成熟**：从 3 个扩展（dataforseo、banana、firecrawl）增加到 8 个（新增 ahrefs、bing-webmaster、seranking、unlighthouse、profound），覆盖了 SEO 领域几乎所有主流第三方工具。这种扩展速度在 Claude Code 插件生态中属于快速成长型。

### 4.3 技术实时性维护

`agents/seo-technical.md` 中包含一处值得单独记录的更新：

```markdown
INP replaced FID — "FID was fully removed from all Chrome tools
on September 9, 2024"
```

Google 在 2024 年 9 月正式从 Core Web Vitals 移除 FID（First Input Delay），以 INP（Interaction to Next Paint）替代。claude-seo 的 agent 文件已反映这一变化。这对 SEO 工具来说是正确性维护的基准要求——一个没有跟上 Google 算法变化的 SEO plugin 是不可信赖的。这个更新说明作者（或社区）在主动追踪领域动态。

---

## 五、校准

本案例对以下评分规则的理解有校准意义：

**R04（触发器设计）的实践确认**：`Triggers on:` 关键词列表 + description 中内嵌"Use when user says..."的组合是经过实战验证的最高分写法。单纯的功能描述得 80 分；加上触发关键词枚举得 95 分；双重触发策略得 100 分。

**Agent Examples 的权重校准**：-15 分/agent 的扣分力度导致 15 个 agent 全无 examples 的仓库整体仍达 94 分，而非更低。这说明 NLPM 的 skill quality 权重 > agent examples 权重——一个 skill 全 100 分的仓库，即使 agent examples 全缺，仍然可以是 Gold 级别。但在实际使用体验中，agent examples 的缺失会造成输出稳定性下降，这是分数没有完全反映的质量损失。

**Security: REVIEW 与 Quality: Gold 可以共存**：94/100 的质量得分和 Security: REVIEW 状态同时存在，说明 NLPM 的质量分和安全评级是正交维度。一个文件写得极好的 plugin，install.sh 里照样可以有 RCE 漏洞。安全评级不影响质量分，但会影响 auditor-contribute 的 `security-blocked` 门控——这个仓库在当前状态下不会被 NLPM 贡献工作流自动贡献。

**Community Contribution Model 的扩展性**：4 个社区贡献者添加的 4 个 sub-skill 全部通过了质量标准，说明单职责 + 清晰 YAML frontmatter 规范足够低门槛，外部贡献者可以直接遵守，不需要深度理解整个 plugin 架构。这是 convention-over-configuration 原则的实证。

---

## 六、行动

基于本案例，以下是可直接执行的自查命令：

**检查自己的 agents 是否有 examples：**

```bash
# 检查某个仓库下所有 agent 文件是否包含 Examples 章节
grep -rL "## Examples" /path/to/your-repo/agents/
# 输出：所有缺少 examples 的 agent 文件列表
# 目标：输出为空
```

**检查自己的 install.sh 是否有 triple-quote injection 模式：**

```bash
# 检测 shell 脚本中将环境变量注入 Python 三重引号字符串的危险模式
grep -rn "'''\${" /path/to/your-repo/extensions/
grep -rn '"""\${' /path/to/your-repo/extensions/
# 输出：所有匹配行（有输出 = 有风险）
# 目标：输出为空
```

**更安全的 credential 处理模式（替代 inject-to-source）：**

```python
# 危险：安装时将 credential 写入源码
api_key = '''${API_KEY}'''  # install.sh 里的 shell 替换

# 安全：运行时从环境变量读取
import os
api_key = os.environ.get("API_KEY")
if not api_key:
    raise ValueError("API_KEY environment variable not set")
```

**检查 Skill 触发器是否使用了双重触发策略：**

```bash
# 检查 skills 的 description 是否包含 "Triggers on:" 或 "Use when"
grep -rL "Triggers on:\|Use when" /path/to/your-repo/skills/
# 输出：缺少触发关键词的 skill 文件
```

**检查 Extension Agent 是否声明了 model：**

```bash
# 检查 agent 文件是否有 model: 字段（在 YAML frontmatter 中）
grep -rL "^model:" /path/to/your-repo/extensions/*/agents/
# 输出：缺少 model 声明的 extension agent 文件
```

---

## 七、对照我的 GitHub 仓库

### 7.1 最相似仓库：MarkQWu/claude-for-legal

`claude-for-legal` 和 `claude-seo` 是目前最具可比性的两个仓库：

| 维度 | AgriciDaniel/claude-seo | MarkQWu/claude-for-legal |
|------|------------------------|-------------------------|
| 领域 | SEO（搜索引擎优化）| 法律工作流 |
| 架构 | 多 plugin + 垂直领域 | 多 plugin + 垂直领域 |
| Plugin 数量 | 8 个 extensions | 8 个（regulatory, corporate, IP, employment, privacy 等）|
| Agents | 18 个 | 10 个 |
| Install 脚本 | 独立 install.sh | 无独立 install.sh |
| 社区贡献 | 有（4 个外部贡献者）| 无 |

两者都是"垂直领域 + 多 plugin"的架构，但 claude-seo 有独立的 extension install.sh 机制，而 claude-for-legal 的多 plugin 之间关系更清晰（因为都是法律的不同分支，天然有逻辑关联）。

### 7.2 最直接的可改进点：Agent Examples

这是我的仓库与 claude-seo 共有的最大问题：

```bash
# 验证结果
grep -l "## Examples" claude-for-legal/agents/*.md   # 0/10
grep -l "## Examples" echo-sleuth-for-claude/agents/*.md  # 0/5
```

15 个 agent 全部命中（10 个 claude-for-legal + 5 个 echo-sleuth），与 claude-seo 的 18 个 agent 全无 examples 完全一致。

**优先处理建议**：不需要为所有 agent 同时补充 examples，选择使用频率最高的 3 个 agent（claude-for-legal 里可能是 regulatory-reviewer、contract-analyzer、privacy-auditor），各补 2-3 个 examples，可以明显改善输出稳定性。

一个合格的 examples 章节格式：

```markdown
## Examples

### 审查劳动合同
**输入**：请审查这份劳动合同，关注不公平条款
**输出**：找到 3 处需要关注的条款：(1) 竞业限制范围过宽...

### 合规检查
**输入**：这份隐私政策是否符合 GDPR？
**输出**：基本合规，但以下 2 处需要修改...
```

### 7.3 最值得借鉴的技术决策

**Orchestrator Skill 的路由表写法**：claude-for-legal 目前每个 plugin 相对独立，没有一个统一的"法律工作流路由入口"。参考 `skills/seo/SKILL.md` 的设计，可以创建一个 `skills/legal/SKILL.md` 作为入口，内嵌路由表：

```markdown
description: "Legal workflow router. Routes to:
- regulatory-legal: compliance, regulatory, GDPR, data protection
- corporate-legal: contracts, M&A, corporate governance
- IP: patents, trademarks, copyright
- employment: labor, HR, workplace
Use when user says: legal review, contract, compliance, legal help"
```

**Triggers on: 语法**：我的项目目前没有使用这一触发器设计。claude-seo 的 25 个 skill 全部使用了 `Triggers on:` + description 双重触发，这是 skills 层拿满分的核心原因。在 drama-workshop-skills 的 skill 文件中已有 `version:` 字段的良好习惯，补充 `Triggers on:` 是自然的下一步。

### 7.4 我的项目相对优势

**Plugin 间逻辑关系更清晰**：claude-for-legal 的 8 个 plugin 都是法律的不同专业分支，彼此在领域上有自然的边界和关联。相比 claude-seo 的 extension 有时边界模糊（`banana` 这个名字就让人困惑它是什么功能），我的 plugin 命名更直白（regulatory-legal、corporate-legal、IP 等），用户直接能理解覆盖范围。

**没有 install.sh 安全风险**：claude-for-legal 没有独立的 extension install.sh 脚本，也就没有 credential injection 的安全面。这是架构选择带来的天然优势，但同时意味着无法支持第三方 API 的可选集成——当需要引入 court-API 或 legal-database 集成时，需要仔细设计安全的 credential 处理方式。

---

## 八、术语表

**Python Code Injection（Python 代码注入）**：通过将不可信输入（如用户提供的环境变量）直接插入 Python 源码字符串，使攻击者能够终止字符串并插入任意代码。一旦该 Python 文件被执行，注入的代码将以执行者的权限运行。

**Triple-Quote Injection（三重引号注入）**：Python Code Injection 的一种变体，利用 Python 的三重引号字符串（`'''...'''` 或 `"""..."""`）。攻击者在变量值中包含 `'''`，即可提前关闭字符串字面量，后续内容将被 Python 解释器当作代码执行。具体形式为：`username = '''${DFSE_USERNAME}'''` 在 shell 中展开后，若 `DFSE_USERNAME` 值为 `x'''; import os; os.system('rm -rf /')#`，则展开结果将执行任意命令。

**SSRF（Server-Side Request Forgery，服务端请求伪造）**：攻击者诱使服务器发出攻击者指定的 HTTP 请求，通常用于访问内网资源、云元数据服务（如 `169.254.169.254`）或进行端口扫描。claude-seo 的 `validate_url()` 函数通过检查目标 IP 是否属于私有地址段来防范此类攻击。

**3 层架构（Three-Layer Architecture）**：在 Claude Code plugin 中，将功能划分为三层：directive 层（行为规范，通常是 CLAUDE.md）、orchestration 层（请求路由和协调，通常是 orchestrator skill 或 command）、execution 层（具体执行，通常是专化 sub-skill 或 agent）。与 MVC 等 Web 架构类似，但适用于 LLM 工具链。

**Orchestrator Skill（编排 Skill）**：一个专门负责请求路由和子任务分发的 skill，本身不执行具体业务逻辑，而是根据用户意图将请求转发给最合适的专化 sub-skill。`skills/seo/SKILL.md` 是典型实现，其 description 中内嵌路由表，明确说明"何种意图 → 转发到哪个 skill"。

**垂直 Plugin（Vertical Plugin）**：专注于单一领域的深度 Claude Code 插件，与通用工具或多领域插件集合相对。垂直 plugin 的特征是：所有 skill、agent、extension 服务于同一个业务领域，相互协作，而非独立并存。claude-seo（SEO 领域）和 claude-for-legal（法律领域）都是典型的垂直 plugin。
