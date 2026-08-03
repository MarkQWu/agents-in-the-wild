# alirezarezvani/claude-skills — 学习案例

**仓库**：https://github.com/alirezarezvani/claude-skills
**Stars**：N/A | **来源**：upstream（≥500 池耗尽，按学习价值补选）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-08-03（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `template-design`, `security-gate`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
alirezarezvani/claude-skills 是一个大型企业级 Claude Code [skill](#skill) 套件，按业务域组织，覆盖 C 级顾问（CEO/CTO/CFO/CMO/COO）、工程团队、市场营销、产品团队、财务、合规/监管（RA/QM）、业务增长等七个主要领域。每个域有独立目录，内有该域的 SKILL.md 入口和若干子 skill/agent。整个仓库包含约 431 个 NL 工件，是本批次 4 个案例中工件数量第二多（仅次于 affaan-m）的仓库。

关键事实：
- 2026-04-06 被 NLPM 审计，得分 **92/100**，本批次最高
- SECURITY **BLOCKED**（11 个安全发现，其中 1 个 CRITICAL、5 个 HIGH）
- 作者 Alireza Rezvani 是独立开发者，针对 B2B SaaS 初创团队设计
- 包含 [SKILL-AUTHORING-STANDARD.md](#skill-authoring-standard)，这是本仓库最大的亮点——元级别的 skill 编写规范文档
- engineering/ 子树质量极高，数十个 skill 得满分 100/100

### 1.2 架构剖析

```
claude-skills/
├── SKILL-AUTHORING-STANDARD.md   # 元级别：所有 skill 的编写模板与规范
├── CONVENTIONS.md                # 代码规范
├── SKILL_PIPELINE.md             # skill 迭代流水线
├── .claude/
│   └── commands/                # 全局命令（review/security-scan/git/* 等）
├── commands/                    # 业务命令（sprint-plan/wiki-query/competitive-matrix 等）
├── agents/                      # 跨域 agent（c-level/engineering/personas 等）
├── c-level-advisor/             # 域：C 级顾问（15+ skills，ceo/cfo/cto/coo/cmo）
├── engineering/                 # 域：基础工程（满分 skill 集中地）
├── engineering-team/            # 域：工程团队协作（playwright-pro/self-improving-agent）
├── marketing-skill/             # 域：营销（25+ skills）
├── product-team/                # 域：产品（8+ skills）
├── finance/                     # 域：财务
├── ra-qm-team/                  # 域：监管/质量管理（ISO 13485/SOC2/GDPR 等）
└── business-growth/             # 域：业务增长
```

**文件类型分布**：约 431 个工件，其中 ~300 个 SKILL.md、~50 个 agent、~40 个 command、2 个 hooks.json、多个 Python/Shell 脚本

**编排关系**：以域级 SKILL.md（如 `engineering-team/SKILL.md`）作为入口，域内 sub-skill 可被用户直接调用或通过顶层命令（commands/）派发。部分域（engineering-team）有显式的 agent 子目录，包含 playwright-pro、self-improving-agent 等多 agent 子系统。无全局 router，各域独立。

**跨件契约**：
- SKILL-AUTHORING-STANDARD.md 定义统一的 [frontmatter](#frontmatter) 格式（含 `license: MIT`、`metadata.version`、`metadata.updated` 等标准字段）
- 域内 SKILL.md 通过 skills 字段或文字引用关联子 skill
- 钩子（hooks.json）在 playwright-pro 和 self-improving-agent 中独立使用，互不干扰

### 1.3 设计思路 / 方法论

**核心设计哲学**：
- **域隔离 + 统一规范**：业务域隔离减少耦合，SKILL-AUTHORING-STANDARD.md 保证全仓库质量下限
- **「三种模式」skill 设计**：SKILL-AUTHORING-STANDARD.md 要求每个 skill 支持「从零构建 / 优化已有 / 情境专用」三种操作模式，而非一刀切
- **上下文优先**：每个 skill 的第一步是「如果 `[domain]-context.md` 存在，先读它」，减少重复提问

**解决什么问题**：初创公司（SaaS）各职能团队用 Claude Code 处理日常业务任务，希望有一套标准化、可安装、跨域复用的 AI 工作流工具箱。

**做了什么 trade-off**：
- 横向覆盖 7 个业务域换来「通盘可用」，但纵向每个域的深度有限
- SKILL-AUTHORING-STANDARD.md 标准化了 frontmatter，但 vague quantifier 问题说明内容层面仍缺乏自动约束
- 大量业务导向 skill 用商业管理词汇（「strategic」「comprehensive」「optimize」）写的，这些词本身就是 vague quantifier

**认知模型**：作者把 AI skill 看作「可切换的专家顾问」——每个 skill 是一个领域专家，用户「切换」到它来处理特定业务问题。Skill 是服务，不是功能模块；用户体验第一，技术约束靠规范文档保证。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「域隔离多 plugin 套件 + 元规范文档」（Domain-Bucketed Multi-Skill Suite with Meta-Standard）**

关键特征：
- 顶层按业务域（而非技术关注点）切分目录
- 每个域有自己的 SKILL.md「入口」充当路由
- 独立的元规范文档（SKILL-AUTHORING-STANDARD.md）定义全仓库编写约定，而非内嵌在 CLAUDE.md 里
- Skill 数量规模化（300+）时，没有 meta-router，依赖用户直接调用域入口
- 安全性风险集中在安装脚本和工程工具脚本，而非 NL 工件本身

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| B2B SaaS 初创全职能覆盖 | ✅ 高度适用 | 域划分和 SaaS 职能自然对齐 |
| 个人开发者（单职能） | ❌ 过度设计 | 用 1-2 个域就够，全套 431 工件太重 |
| 企业内部多团队共享 skill 库 | ✅ 适用 | 元规范文档能保证跨团队质量一致性 |
| 需要精确安全审计通过 | ⚠️ 当前不适用 | CRITICAL curl\|bash 安装指令尚未修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 域隔离多 plugin 套件（本案） | alirezarezvani/claude-skills | 覆盖广、域内聚合好、元规范保质量 | 域间无自动路由、跨域任务需用户手动选 |
| 单一扁平 skill 清单 | 小型个人仓库 | 简单直接 | 300+ skill 后难以管理，无自文档化 |
| 插件 + 分发垫片（navapbc 模式） | navapbc/dso | 优雅降级、支持安装检测 | 结构复杂、垫片拉低 NLPM 整体得分 |

### 2.4 改进空间
1. **当前问题**：安装文档的 `curl | bash` 指令无校验和（CRITICAL 安全）。**改进做法**：改为 `curl -sL <url> -o install.sh; sha256sum install.sh; bash install.sh`，并在 README 中列出预期 hash。**预期收益**：解除 SECURITY BLOCKED 状态。

2. **当前问题**：全仓库系统性 vague quantifier（"appropriate" "comprehensive" "ensure" "optimize" 等），87 个质量问题中大多数来自此。**改进做法**：在 SKILL-AUTHORING-STANDARD.md 中加一节「禁止词汇表」（参考 bureau/CLAUDE.md 的做法），同时在 CI 中加 grep 检查。**预期收益**：vague quantifier 问题密度从当前水平（每个 skill 约 2-4 个）降至 0-1。

3. **当前问题**：playwright-pro 子 skill 在 audit 时 7 个文件缺 frontmatter，部分已修复。**改进做法**：在 SKILL-AUTHORING-STANDARD.md 模板里的 frontmatter 加一行注释「勿省略 name + description，否则 skill 不会注册」。**预期收益**：防止类似问题在新 skill 中复现。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时 NL 得分 **92/100**，是本批次最高分。

| 区域 | 得分范围 | 主要问题 |
|---|---|---|
| engineering/ 树 | 95-100 avg | 极少问题，多数 Clean |
| ra-qm-team/ | 90-100 avg | 合规专用词汇导致轻微 vague quantifier |
| c-level-advisor/ | 75-95 avg | vague quantifier 超限（coo/cmo 各 -20 分） |
| marketing-skill/ | 88-95 avg | 少量 vague quantifier |
| 问题最多的个别文件 | 65-78 | competitive-matrix.md（65），business-investment-advisor（78） |

### 3.2 当时值得借鉴的模式
1. **SKILL-AUTHORING-STANDARD.md 元规范** → 在仓库根目录放一个专门的 skill 编写规范文档，包含 frontmatter 模板、三种操作模式、上下文优先原则。相比 navapbc 把约定分散在各 agent 里，这种「单一真实来源」的方式让新 skill 作者只需读一个文件。如何借鉴：在 gstack 或 bureau 的根目录建一个类似文档，把现有的 frontmatter 约定集中到那里。

2. **域级 SKILL.md 入口** → 每个域（engineering-team/、c-level-advisor/ 等）在根目录有一个汇总的 SKILL.md 文件（得分 95-100），用「何时调用本域 skill」的描述串联所有子 skill。如何借鉴：在 gstack 的顶层建一个 INDEX.SKILL.md，按 category 归类现有 skill，帮助用户选择。

3. **engineering/ 树的 100/100 标准** → `engineering/terraform-patterns/SKILL.md`、`engineering/database-schema-designer/SKILL.md` 等工程类 skill 全部满分，说明技术约束越明确（Terraform HCL、数据库 schema），vague quantifier 越少。如何借鉴：为技术 skill 写约束先于写内容，从「如何做」而非「应该做什么」开始描述。

4. **自提升 agent（self-improving-agent）** → 一个能「记忆→提取→推广」自身学习成果的 agent 子系统：memory-analyst.md 分析会话，skill-extractor.md 提取可复用 skill，promote 技能将成果推回仓库。这是 skill 作为「经验积累」（experience-accumulation）的直接体现。如何借鉴：bureau 的 capture→compile 流程与此理念一致，可以借鉴 promote 技能的「提取 → 写回」机制。

5. **ra-qm-team 合规专域的高质量** → ISO 13485、GDPR、SOC2 等合规 skill 得分 100，说明高度专业化的领域（合规条款引用）反而产出更精确的描述，因为领域词汇本身是精确的。如何借鉴：在自己的专业领域 skill 里，用领域标准名称（RFC 编号、法规条款号）代替模糊描述。

### 3.3 当时的缺陷
1. **问题**：系统性 vague quantifier（87 个质量问题，大多数文件受影响）。**根本原因**：商业管理类 skill（C 级顾问、市场营销）本身的词汇就是模糊词——「制定综合性战略」是行业习语，但 NLPM 无法区分领域词汇和真正的模糊。没有领域词汇白名单机制，也没有 vague quantifier 的 CI 检查。自查：**我有没有**在业务向 skill 里过度使用 "ensure/appropriate/comprehensive"？

2. **问题**：wiki agents（cs-wiki-linter、cs-wiki-librarian、cs-wiki-ingestor）被审计为「只读操作声明了 Write/Edit 工具」（Bugs #13-15）。**根本原因**：审计时把「描述上说不主动修改」误判为「工具声明应为只读」——但 linter 需要 Write 来生成报告文件，librarian 显式说会把答案「写回 wiki」。**这是一个审计误报**，工具声明和实际行为是自洽的。自查：NLPM 分错时，我会怎么处理？这是建立 `REVIEW-DEFENSE` 类文档的理由。

3. **问题**：docs/plugins/index.md 中的 `curl | bash` 安装指令（CRITICAL）。**根本原因**：安装脚本的 UX 方便性（一行命令）和安全性（无校验和）直接对立，作者选择了 UX。这在开发者工具中极为常见，但一旦 `raw.githubusercontent.com` 上的文件被篡改，所有安装用户会被攻击。自查：我的仓库有没有 one-liner 安装命令？

### 3.4 当时的优化机会
1. 在 SKILL-AUTHORING-STANDARD.md 中加「禁止词汇」一节，把 appropriate/comprehensive/robust/ensure 等词列为禁止词，从源头减少 vague quantifier
2. 为 docs/plugins/index.md 的安装命令加校验和步骤，解除 CRITICAL 安全阻断
3. 为 competitive-matrix.md（65/100）加 output format 和编号步骤，估计能到 90+

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| playwright-pro sub-skills 缺 frontmatter（Bugs #1-7） | `head -3 engineering-team/playwright-pro/skills/review/SKILL.md` | **部分修复**：review/SKILL.md 和 testrail/SKILL.md 已加 frontmatter | 最明显的 2 个修复了，其余 5 个（coverage/generate/report/migrate/browserstack）状态待查 |
| curl\|bash 安装指令（CRITICAL） | `grep -n "curl" docs/plugins/index.md` | **持续**：第 84 行仍为 `curl -sL … \| bash` | 3 个月内 CRITICAL 安全问题未修复，仓库仍处于 SECURITY BLOCKED |
| wiki agents Write/Edit 工具声明（Bugs #13-15） | `head -10 agents/engineering/cs-wiki-linter.md` | **持续，但属审计误报**：linter 的描述明确说「产出报告文件」，Write 工具是合理的 | 这是 NLPM 审计的误判，后续可作为 suppress 案例 |

### 4.2 架构演进
当前 HEAD vs 2026-04-06 快照：
- 根目录新增多个规范文档（CONVENTIONS.md、SKILL_PIPELINE.md、STORE.md、GEMINI.md），说明作者在向「协议化仓库」方向演进
- 新增了 `compliance-os/`、`commercial/`、`research/`、`research-ops/`、`productivity/` 等新域，工件数量继续增加
- SKILL-AUTHORING-STANDARD.md 是 audit 后新增的，说明作者看到了质量问题后针对性建立了规范

### 4.3 新增的可学习模式
- **GEMINI.md** 并行配置文件：说明仓库在向「多 AI 引擎兼容」演进，同一套 skill 同时支持 Claude Code 和 Gemini CLI
- **STORE.md / SKILL_PIPELINE.md**：skill 的发布市场和迭代流水线文档化，让 skill 有了「产品」的生命周期管理

---

## 五、校准

### 5.1 我已经在做对的
1. bureau 的 CLAUDE.md 有明确的禁止词汇表（delve/crucial/robust/comprehensive 等），这比 alirezarezvani 当前状态更严格——是正向领先
2. gstack 的 `version: 2.0.0` 字段比本仓库的 `metadata.version` 更轻量，但同样实现了版本追踪
3. 我的 skill 数量较少，没有出现「域级 SKILL.md 入口」的需求，现有扁平结构是合理的

### 5.2 挑战 / 验证
本案例验证了：**规模化（300+ skill）后，必须有元级别规范文档来保证质量下限**。过去我认为「CLAUDE.md 里写规范就够了」，这个案例提醒：CLAUDE.md 是给 Claude 读的配置，SKILL-AUTHORING-STANDARD.md 是给新 skill 作者读的写作指南，二者服务不同受众，不能合并。当仓库里 skill 超过 20 个时，单独的 SKILL-AUTHORING-STANDARD.md 文档值得建立。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 skill 里有多少 vague quantifier
grep -rn -E '\b(appropriate|comprehensive|robust|ensure|optimize|effective|proper|relevant|suitable|strategic)\b' \
  ~/.claude/skills/*/SKILL.md 2>/dev/null | wc -l
```
命中 5 个以上：找最常出现的，逐一替换为可验证的具体描述（如「至少 3 个」「字段非空」）。

```bash
# 检查我的仓库里有没有 curl|bash 安装命令
grep -rn "curl.*|.*bash\|curl.*|.*sh" ~/projects/ 2>/dev/null | grep -v ".git"
```
命中后：改为分步：先 curl 下载，输出校验和，再由用户决定是否执行。

### 6.2 灵感 → 实施路径
1. **想法**：为 gstack/bureau 建立类似 SKILL-AUTHORING-STANDARD.md 的元规范文档。**为何可行**：两个仓库的 skill 数量已达到需要「写作指南」的量级；bureau 的 CLAUDE.md 已有部分规范，只需提取并专门化。**第一步**：在 bureau 根目录建 SKILL-AUTHORING-STANDARD.md，把 CLAUDE.md 里涉及 skill 写作的章节迁移过去，约 30 分钟。

2. **想法**：仿照 self-improving-agent 的「提取 → 写回」机制在 bureau 实现 skill 自演进。**为何可行**：bureau 已有 capture 流程收集会话，缺的是「从会话中提炼 skill 并写到 skills/ 目录」的 promote 阶段。**第一步**：读 engineering-team/self-improving-agent/skills/promote/ 的 SKILL.md，仿照其逻辑在 bureau/skills/ 加一个 `skill-promoter` skill，约 1 小时。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：B2B SaaS 初创团队全职能 AI skill 套件，覆盖 C 级顾问到工程研发
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 大型多 skill 套件、域分类、文档规范 | gstack 用工具角色（CEO/设计师）分类而非业务域；面向个人 | 高 |
| MarkQWu/bureau | 中 | 有 CLAUDE.md 规范、知识管理导向 | bureau 侧重 capture→compile，而非职能覆盖 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 系统性 vague quantifier | `grep -c "appropriate\|comprehensive\|ensure" gstack/skills/*/SKILL.md` | gstack 待核查，预计有命中（模板生成的 skill 易带商业词汇） | 中 |
| 缺 SKILL-AUTHORING-STANDARD 文档 | `ls gstack/ \| grep -i "standard\|authoring"` | gstack 和 bureau 均无专用 skill 编写规范文档 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` → 建 SKILL-AUTHORING-STANDARD.md，把 `preamble-tier`/`interactive`/`version` 等非标准 frontmatter 字段的含义解释清楚（目前只有 gstack 内部知道这些字段的意义）

### 8.3 别人的更优方案

1. **领域**：元规范文档
   - **本案例做法**：根目录独立的 SKILL-AUTHORING-STANDARD.md，含 frontmatter 模板、三种操作模式、上下文优先模式；与 CLAUDE.md 分开
   - **我的项目现状**：gstack 的 skill 编写约定分散在各 skill 文件中，bureau 的约定内嵌在 CLAUDE.md 里
   - **如何借鉴**：从 bureau/CLAUDE.md 里提取「skill 编写规范」段落到独立文件，30 分钟

2. **领域**：域级 SKILL.md 入口
   - **本案例做法**：每个业务域（engineering-team/SKILL.md、marketing-skill/SKILL.md）有一个汇总 skill，描述何时调用该域、与其他域的区别
   - **我的项目现状**：gstack 的 skills/ 是扁平目录，无域级入口
   - **如何借鉴**：若 gstack skills 超过 30 个，按工具角色建子目录并加入口 SKILL.md

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：禁止词汇的执行力度
- **我的做法**：`MarkQWu/bureau` 的 CLAUDE.md 有明确禁止词汇清单，且措辞是「No AI vocabulary」而非建议
- **本案例做法**：SKILL-AUTHORING-STANDARD.md 未包含禁止词汇表，导致系统性 vague quantifier（87 个质量问题）
- **意义**：在 skill 编写规范里加禁止词汇清单是性价比极高的质量门，值得向社区推广

---

## 八、术语表

### <a name="skill"></a>skill
> Claude Code 中的「技能」模块，对应 `SKILL.md` 文件。用 `---` 包裹的 [frontmatter](#frontmatter) 声明名称、描述、允许工具等；正文描述 Claude 在该技能下的行为。用户通过 `/skill-name` 或 `#skill-name` 调用。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明 `name`、`description`、`model`、`allowed-tools` 等元数据。Claude Code 依靠 frontmatter 注册 skill/agent/command；缺少 frontmatter = 文件对注册系统不可见。

### <a name="skill-authoring-standard"></a>SKILL-AUTHORING-STANDARD.md
> 一种元规范文档，专门给「写 skill 的作者」看，与给 Claude 看的 CLAUDE.md 不同。包含：frontmatter 模板、允许/禁止的写法、三种操作模式（从零构建/优化已有/情境专用）。当仓库 skill 超过 20 个时，这类文档能保证不同作者产出的 skill 质量一致。

### <a name="vague-quantifier"></a>模糊量词（Vague Quantifier）
> NL 工件中出现的模糊程度词，如 "appropriate"（合适的）、"comprehensive"（全面的）、"ensure"（确保）、"robust"（健壮的）。NLPM 规则认为这类词不可验证——「确保代码健壮」无法被 AI 判断是否达成，而「不少于 3 个单元测试覆盖率 >80%」可以。模糊量词过多会导致 AI 无法明确执行，也会让人工审查者无法核实完成度。
