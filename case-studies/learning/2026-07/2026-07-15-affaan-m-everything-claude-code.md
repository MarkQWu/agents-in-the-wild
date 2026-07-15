# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：N/A（registry 未记录）| **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-15（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `security-gate`, `examples-driven`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`everything-claude-code` 是 affaan-m 为多语言、多框架工程场景构建的超大型 Claude Code 配置仓库。核心卖点是「一个仓库涵盖所有场景」——从 Angular、Django 到 React、Kotlin，从可访问性审计到 SEO，每个场景都有对应的 slash command。

关键事实：
- 在审计时（2026-04-06）有 934 个 NL 工件；到 2026-07-15 HEAD，Markdown 文件已增至 **2495 个**（增长约 167%）
- 提供 7+ 种语言的本地化版本（zh-CN、zh-TW、ko-KR、pt-BR、tr、ja-JP）
- `SOUL.md` 是独特设计：定义了 Claude 的「性格和价值观」（协作、诚实、谦虚），而非仅技术规范
- 安装通过 Claude Code plugin 机制，`plugin.json` 是入口

### 1.2 架构剖析

**目录结构**（2026-07-15 HEAD）：
```
everything-claude-code/
├── CLAUDE.md                  # 主配置
├── SOUL.md                    # Claude「性格」声明
├── RULES.md                   # 代码规约
├── AGENTS.md                  # agent 清单
├── CHANGELOG.md               # 变更记录（版本追踪）
├── commands/                  # 核心：框架特定的 slash commands
│   ├── spike-investigation.md
│   ├── angular-architecture-audit.md
│   ├── a11y-audit.md
│   └── ...（50+ 个）
├── skills/
│   ├── ai/SKILL.md
│   ├── bash/SKILL.md
│   └── customs-trade-compliance/SKILL.md
├── docs/                      # 多语言本地化版本
│   ├── zh-CN/commands/
│   ├── zh-TW/commands/
│   ├── ko-KR/commands/
│   ├── pt-BR/commands/
│   ├── tr/commands/
│   └── ja-JP/commands/
└── COMMANDS-QUICK-REF.md      # 所有命令的速查索引
```

**文件类型分布**（当前 HEAD）：
- 50+ slash commands（.md，在 commands/ 下）
- 6+ 语言 × ~10 commands = 60+ 本地化副本（在 docs/ 下）
- 3 个 SKILL.md
- 大量文档和配置文件（SOUL.md、RULES.md、CHANGELOG.md 等）
- 总计 2495 个 Markdown 文件

**编排关系**：
- commands 是平列的（无层级，无 router），用户直接通过 `/spike-investigation` 等调用
- 3 个 skills 是辅助知识库（ai、bash、customs-trade-compliance）
- 本地化版本是 commands/ 的平行镜像，手工同步（漂移风险）
- SOUL.md 是全局性格约束，作用于所有命令

**跨件契约**：
- commands/ 和 docs/*/commands/ 应保持内容同步（手工，无自动化）
- COMMANDS-QUICK-REF.md 是命令目录，需随 commands/ 更新

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「全覆盖」——用一个仓库包含所有可能的工程场景命令，让用户安装后立刻拥有完整工具箱
- **解决什么问题**：开发者每次开新项目都要重新配置 Claude Code，这个仓库提供了「一键装好一切」的解决方案
- **做了什么 trade-off**：
  - 覆盖广 vs 质量深：2495 个文件中许多是本地化副本和模板，真正高质量的核心命令只有 50+
  - 单仓大包 vs 按需安装：用户得到一切，但也背负了无关命令的「认知噪音」
  - 手工本地化 vs 自动翻译：7 种语言本地化是手工维护的，版本之间可能出现内容漂移
- **SOUL.md 的独特认知**：作者把 Claude 视为有「性格」的协作者，而非工具。`SOUL.md` 声明的「诚实、谦虚、协作」是跨所有命令生效的行为准则——这是一种把「价值观」作为 NL 编程参数的方式

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「全景库 + 多语言镜像」

一个仓库覆盖所有框架/场景，commands 平铺，外加多语言本地化镜像层。

模式特征清单：
- **横向全覆盖**：50+ commands 覆盖多语言（Angular/Django/React/Kotlin）、多职能（a11y/SEO/PR review）
- **多语言镜像层**：原始英文 + 6 种语言本地化，结构完全镜像
- **性格声明文件**：`SOUL.md` 定义跨命令的「行为价值观」
- **速查索引**：`COMMANDS-QUICK-REF.md` 帮助用户快速定位命令
- **版本追踪**：`CHANGELOG.md` 记录版本演进

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多框架、多语言的全栈团队 | ✅ 适用 | 覆盖面广，不同角色能找到所需命令 |
| 单一技术栈的专注团队 | ❌ 不适用 | 无关命令造成噪音；按需安装单一 skill 更轻量 |
| 需要本地化内容的国际团队 | ✅ 适用 | 7 种语言本地化已就绪 |
| 安全敏感的企业环境 | ❌ 不适用 | Security BLOCKED（7 项安全发现，含 Critical）未解决 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 全景库（本仓库） | affaan-m/everything-claude-code | 一次安装获得一切；覆盖面极广 | 质量参差；无关命令噪音；本地化维护成本高 |
| 单 skill 专注 | 2389-research/simmer | 质量极高；学习成本低 | 只覆盖一个场景 |
| 按团队分 agent | LerianStudio/ring | 职能清晰；模型最优 | 重量级；非企业场景不适用 |
| 社区整合型 | awesome-claude-code-plugins | 收录第三方 | 质量不一；无统一风格 |

### 2.4 改进空间

1. **当前问题**：6 种语言本地化与英文 commands/ 手工同步，漂移不可避免（比如 zh-CN/commands/eval.md 的功能落后于 eval.md）。**改进做法**：用 `i18n.json` 对照表 + CI 检查来验证本地化版本与英文版的段落覆盖率。**预期收益**：消除"本地化版本看起来在但其实过时"的问题。

2. **当前问题**：Security BLOCKED 状态下仍在持续增加内容。**改进做法**：在 README 顶部明确标注"Security BLOCKED — do not use in production"；修复 7 项安全发现后解除 BLOCKED 状态。**预期收益**：对用户诚实，避免意外引入安全风险。

3. **当前问题**：2495 个 .md 文件中大量是内容相似的本地化副本，用 NLPM 扫描成本高。**改进做法**：把本地化移到单独的仓库（如 `everything-claude-code-i18n`），主仓只保留英文核心。**预期收益**：主仓质量更聚焦；国际化版本由各语区社区维护。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **84/100**。

| 文件/群组 | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 23 | 缺 name+description frontmatter |
| commands/eval.md | 50 | 缺 name；无 allowed-tools；无输出格式 |
| skills/customs-trade-compliance/SKILL.md | 55 | vague 量词上限 + 缺 output format |
| skills/ai/SKILL.md | 84 | 8 个 vague 量词（effective/specific/appropriate/focused...） |
| plugin.json | 96 | 2 个 vague 词（Professional/comprehensive） |

**说明**：84/100 的高分主要来自少数高质量文件（plugin.json 96分、CLAUDE.md 95分）拉高均值；实际上 **20+ commands 得分在 23–33 分**，属于严重低质量。

### 3.2 当时值得借鉴的模式

1. **SOUL.md 性格声明** → 在技术配置之外，为 Claude 定义价值观和协作风格，让 Claude 不仅"能做事"，还有一致的行为风格。→ 借鉴：在重要项目的 CLAUDE.md 里加 `## 协作价值观` 章节。

2. **CHANGELOG.md 版本追踪** → 记录每次大更新做了什么，让用户知道仓库在演进。→ 借鉴：plugin 仓库应维护 CHANGELOG.md，尤其是有 breaking changes 时。

3. **COMMANDS-QUICK-REF.md 速查表** → 50+ commands 必须有一个索引文件，否则用户无法发现所有命令。→ 借鉴：命令数超过 10 个的仓库应有速查索引。

4. **RULES.md 代码规约独立文件** → 不把代码规约塞进 CLAUDE.md，而是独立成 RULES.md，让 CLAUDE.md 保持简洁。→ 借鉴：配置与规约分离。

### 3.3 当时的缺陷

1. **大量 commands 缺 name/description [frontmatter](#frontmatter)** → 根本原因：仓库是快速积累的"命令集合"，早期添加的命令在 frontmatter 规范建立之前就写好了，后来没有系统性回填。这是"先 ship 后规范"模式的典型后果。自查：我的 commands 仓库是否有类似的早期无 frontmatter 文件？

2. **commands 缺 allowed-tools 声明** → 根本原因：作者对 `allowed-tools` 字段的重要性认识不足，认为 Claude 会"自动选择工具"。但实际上未声明 tools 会导致 runtime 行为不可预测。自查：我的 SKILL.md 是否都有 allowed-tools？drama-workshop-skills 的 short-drama/SKILL.md 是否声明了工具？

3. **Security BLOCKED（7 项安全发现）** → 根本原因：命令里有 `curl | bash`、`subprocess` 等高危执行模式，这些是用户在各种技术栈场景下"自然写出来"的辅助命令，但 NLPM 安全扫描会标记它们。自查：我的命令文件里有没有 `curl`、`wget`、`subprocess` 等执行外部命令的指令？

4. **多语言本地化版本质量低于英文原版** → 根本原因：本地化是手工完成的，人力不足以持续同步。`zh-CN/commands/eval.md` 得分只有 55 而英文原版更高，说明翻译时只翻译了内容，没有同步质量改进。自查：我是否在维护多语言文档？如果是，如何保证同步？

### 3.4 当时的优化机会

1. 批量为所有缺 frontmatter 的 commands 添加 `name:` 和 `description:` 字段（脚本可在 15 分钟内完成）
2. 为 2-3 个高频命令（eval.md、orchestrate.md）添加完整的 `## Examples` 章节（input→output 对）
3. 把 7 项安全发现中 Medium/Low 的先行修复，申请从 BLOCKED 降级到 REVIEW

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| commands 缺 description frontmatter | `find commands/ -name "*.md" \| xargs grep -L "^description:"` | **94/94 commands 现在都有 description** | **已修复**：frontmatter 已批量回填 |
| commands 缺 name frontmatter | `grep "^name:" commands/test-coverage.md` | test-coverage.md 有 description 但无独立 `name:` 字段 | **部分改进**：description 已有，name 字段仍需验证 |
| Security BLOCKED（7 项发现） | 审计文件无更新记录 | registry 中状态仍为 `audited`（未 contributed，未修复记录） | **持续存在**：安全状态未改变 |

**重要发现**：从 934 个工件增至 2495 个文件（+167%），而 commands 下的 frontmatter 已批量修复——这说明作者在大幅扩展的同时也修复了最明显的质量问题。这是积极的信号。

### 4.2 架构演进

- **最显著变化**：仓库规模从 934 个 NL 工件扩展到 2495 个 Markdown 文件（2026-04-06 → 2026-07-15，3 个月增长 167%）
- **frontmatter 回填**：94/94 commands 的 description 字段已填写，这是对当时最严重的质量问题的系统性修复
- **结构保持稳定**：核心目录结构（commands/、docs/、skills/）没有根本性重组
- **新文件类型增加**：SOUL.md、CONTRIBUTING.md、CODE_OF_CONDUCT.md 等「社区文件」新增，说明仓库从个人项目向开源社区项目演进

这说明作者后来意识到：**frontmatter 质量是外部可见的门面**，修复它能让仓库看起来更专业、更可信。

### 4.3 新增的可学习模式

SOUL.md 是 audit 时就有的，但 CONTRIBUTING.md、CODE_OF_CONDUCT.md 是新增的——这反映了「NL 编程仓库社区化」的趋势：不只是工具代码，还需要贡献者指南和行为准则。这对开源 Claude Code 插件有参考意义。

---

## 五、校准

### 5.1 我已经在做对的

1. **CLAUDE.md 驱动配置**：和本案例一样，我的仓库也以 CLAUDE.md 为核心配置源。
2. **CHANGELOG.md 版本追踪**：gstack 和 bureau 有 CHANGELOG，方向正确。
3. **专注单一能力**：drama-workshop-skills 专注于「微短剧剧本创作」，比本案例「什么都包」的全景库设计更聚焦。

### 5.2 挑战 / 验证

本案例验证了一个重要认知：**「先有仓库，再补文档」的模式是可行的**。affaan-m 最初的 commands 没有 frontmatter，但后来系统性地修复了——这说明 debt 不是不能还的。关键是要**有意识地去还**，而不是任由 technical debt 积累。

同时挑战了我的假设：**「大仓库质量一定差」**。虽然 2495 个文件里确实有质量不均的本地化副本，但核心命令（94 个 commands）的 frontmatter 修复率已达 100%。数量和质量不是对立的。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库中命令文件是否都有 frontmatter
find . -name "*.md" -path "*/commands/*" | xargs grep -L "^description:\|^name:" 2>/dev/null
# 命中后怎么办：用以下模板批量添加 frontmatter
# sed -i '1s/^/---\ndescription: <一句话说明>\n---\n\n/' 文件名

# 检查 skills 里有无 vague 量词
grep -rn -iE '\b(effective|appropriate|comprehensive|robust|efficient|relevant|proper|good|clear)\b' \
  ~/.claude/skills/*/SKILL.md 2>/dev/null | head -20
# 命中后：替换为具体指标，如 "comprehensive" → "covers all exported functions"

# 检查是否有 curl/wget/subprocess 高危调用
grep -rn "curl\|wget\|subprocess\|exec(" . --include="*.md" | grep -v ".git" | head -10
# 命中后：评估是否必要；必要的话加安全说明注释
```

### 6.2 灵感 → 实施路径

1. **想法**：为我的仓库添加 SOUL.md「AI 性格声明」
   - **为何可行**：本案例证明 SOUL.md 是被 Claude 主动参考的跨命令约束，能让 AI 行为更一致
   - **第一步**：参考 affaan-m/SOUL.md 的格式，在 drama-workshop-skills 里写一份「剧本创作 AI 的价值观」声明（诚实评估市场、不写低俗内容等）；20 分钟可完成

2. **想法**：给 bureau 仓库添加 COMMANDS-QUICK-REF.md 速查索引
   - **为何可行**：当命令超过 5 个时，速查表让新用户 onboarding 快 3-5×
   - **第一步**：列出 bureau 的所有 commands，写一个两列 Markdown 表格（命令名 + 一句话说明）；15 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 affaan-m/everything-claude-code 的核心目的**：「全覆盖」命令集，一个仓库满足多框架、多语言开发者的所有 Claude Code 配置需求

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是「一站式」Claude Code 配置仓库；多种 skill 覆盖多场景 | gstack 聚焦于角色扮演（CEO/Designer），而非框架场景 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都有 SKILL.md，都在 Claude Code 生态 | drama-workshop-skills 单一场景聚焦；affaan-m 全覆盖 | 低 |
| MarkQWu/graphify | 低 | 都有 NL 工件 | graphify 是知识图谱工具，完全不同域 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 description frontmatter | `find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" \| xargs grep -L "^description:"` | gstack：9/10 被查 SKILL.md 缺 description，仅有 name 字段 | 高 |
| vague 量词（effective/comprehensive） | `grep -r "appropriate\|comprehensive" /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md` | drama-workshop-skills 的 short-drama/SKILL.md 含"complete"等 vague 词汇 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` → 检查 `pair-agent/SKILL.md`：当前无 `description:` 字段，添加一句话说明；5 分钟
- `MarkQWu/drama-workshop-skills/short-drama/SKILL.md` → 搜索并替换 "complete" 等 vague 词为具体标准；10 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：批量 frontmatter 回填（从 0 到 94/94 commands 修复）
   - **本案例做法**：系统性地对所有 commands 添加 `description:` frontmatter，实现 100% 覆盖率
   - **我的项目现状**：gstack 约有 30-40% 的 SKILL.md 缺少 description 字段（仅有 name）
   - **如何借鉴**：写一个 bash 脚本检查所有 SKILL.md 的 frontmatter 完整性，然后批量添加（或用 nlpm:fix 自动修复）

2. **领域**：SOUL.md 跨命令价值观声明
   - **本案例做法**：单独一个文件声明 AI 协作者的价值观（诚实、谦虚、协作），作用于所有 commands
   - **我的项目现状**：没有等价文件；价值观约束散在各个 SKILL.md 的描述里
   - **如何借鉴**：在 gstack 或 bureau 创建 `SOUL.md`，声明「这个仓库里的 AI 应该…」；30 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：场景聚焦度（drama-workshop-skills）
- **我的做法**：`MarkQWu/drama-workshop-skills` 只聚焦微短剧创作，skill 设计深度远超一个泛化的命令
- **本案例做法（弱在哪）**：`everything-claude-code` 的 `commands/generate-slidedeck.md` 这类命令得分只有 35，明显是泛泛而写、不够深入
- **意义**：「宽而浅」vs「窄而深」是设计选择；drama-workshop-skills 在单场景深度上胜出，若被审计这是亮点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 读 SKILL.md 或 command .md 时先解析 frontmatter，才知道这个文件叫什么、怎么注册。没有 frontmatter 的命令文件像一个"没有名字的员工"——系统找不到它。

### <a name="SOUL.md"></a>SOUL.md
> affaan-m 发明的一种特殊配置文件，用来定义 Claude 在这个项目里的「性格和价值观」。比如"诚实对待不确定性"、"主动提出改进建议"。这些不是功能指令，而是行为约束——Claude 在整个项目里都会参考这些价值观行事。类比：技术手册 vs 企业文化手册，CLAUDE.md 是技术手册，SOUL.md 是文化手册。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code [frontmatter](#frontmatter) 里的一个字段，声明这个 skill 或 command 被允许使用哪些工具（如 `Read`、`Write`、`Bash`）。不声明的话，Claude 可能因为权限不足而静默失败，或者被提示批准每一步操作，打断工作流。
