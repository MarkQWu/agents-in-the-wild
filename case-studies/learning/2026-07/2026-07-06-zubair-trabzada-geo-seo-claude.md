# zubair-trabzada/geo-seo-claude — 学习案例

**仓库**：https://github.com/zubair-trabzada/geo-seo-claude
**Stars**：5411 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-06（基于当前 HEAD）
**主题标签**：`security-gate`, `cross-reference`, `vague-quantifier`, `model-pinning`, `examples-driven`

**xiaolai 案例**：[../2026-04-29-zubair-trabzada-geo-seo-claude.md](../2026-04-29-zubair-trabzada-geo-seo-claude.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[zubair-trabzada/geo-seo-claude](https://github.com/zubair-trabzada/geo-seo-claude) 是一个面向 GEO（Generative Engine Optimization，即 AI 搜索引擎优化）的 Claude Code 插件，帮助 SEO 从业者优化内容以在 ChatGPT、Perplexity 等 AI 搜索引擎中获得更高引用率。插件组织了 5 个专职 [subagent](#subagent)（geo-ai-visibility、geo-content、geo-platform-analysis、geo-schema、geo-technical）和 15 个 skill 文件，并附带 7 个 Python 脚本（品牌扫描、可引用性评分、CRM 看板、PDF 报告生成等）。这个仓库的特殊性在于：**它在 audit 后 23 天内完成了 NLPM 推荐的全部 5 个修复**，是本 batch 4 个案例中唯一完成修复循环的仓库。

关键事实：
- **创建背景**：作者 Zubair Trabzada 的 GEO 咨询工作流工具化，把「GEO 咨询服务」打包成 Claude Code 插件
- **规模**：5 个 agent / 15 个 skill / 1 个顶级 skill（geo/SKILL.md）/ 7 个 Python 脚本
- **特殊性**：业务工作流（CRM、提案生成、月度对比报告）与技术能力（Schema 生成、爬虫分析）并列，定位「全栈 GEO 咨询助理」
- **修复状态**：audit 后 7 天内（2026-04-29），5 个修复 PR (#50-#54) 被合并——作者读了 audit 报告，启动了本地 Claude Code session，在 16 秒内提交了 5 个 PR

### 1.2 架构剖析

```
zubair-trabzada/geo-seo-claude/
├── geo/
│   └── SKILL.md          # 顶级编排 skill（子 skill 注册表）
├── agents/
│   ├── geo-ai-visibility.md
│   ├── geo-content.md
│   ├── geo-platform-analysis.md
│   ├── geo-schema.md
│   └── geo-technical.md
├── skills/
│   ├── geo-audit/
│   ├── geo-brand-mentions/
│   ├── geo-citability/
│   ├── geo-compare/
│   ├── geo-content/
│   ├── geo-crawlers/
│   ├── geo-llmstxt/       # 注意：audit 后改名（geo-llms-txt → geo-llmstxt）
│   ├── geo-platform-optimizer/
│   ├── geo-proposal/
│   ├── geo-prospect/
│   ├── geo-report/
│   ├── geo-report-pdf/
│   ├── geo-schema/
│   └── geo-technical/
├── scripts/              # 7 个 Python 脚本
│   ├── brand_scanner.py
│   ├── citability_scorer.py
│   ├── crm_dashboard.py
│   ├── fetch_page.py     # 已修复 SSRF
│   ├── generate_pdf_report.py
│   ├── llmstxt_generator.py # 已修复 SSRF
│   └── webapp/
│       └── app.py        # 已修复 Flask debug=True
├── schema/               # 6 个 JSON schema 文件
└── requirements.txt
```

- **文件类型分布**：5 个 agent / 15 个 skill / 7 个 Python 脚本 / 6 个 JSON schema / 1 个 requirements.txt
- **编排关系**：`geo/SKILL.md` 是顶级 skill，登记了 5 个 agent 和 15 个 sub-skill，以及它们之间的调用关系；agent 并行执行，结果汇总到 `geo-report/SKILL.md`
- **跨件契约**：`geo/SKILL.md` 维护了一张「子 skill 表」，是跨件引用的唯一来源；audit 发现的 `geo-llms-txt` 拼写错误就出现在这张表的引用中

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「全栈 GEO 咨询自动化」——从技术审计（爬虫分析、Schema 生成）到业务交付（PDF 报告、CRM 更新、客户提案）一站式覆盖
- **解决什么问题**：GEO 咨询是一个新兴领域（2024-2025 年才大规模出现），缺乏标准化工具链；作者把自己的咨询方法论编码进插件
- **Trade-off**：全栈覆盖 vs. 安全表面积——Python 脚本层引入了 SSRF、Flask debug 等安全问题，而纯 NL 设计不会有这些问题；作者选择了「可用性第一」，安全问题在 audit 后修复
- **认知模型**：作者把 Claude Code 视为「AI 驱动的咨询服务平台」——不只是代码工具，而是业务交付的全链路助理。这是本批次 4 个案例中最「业务化」的设计视角

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「业务工作流 skill 化（Business Workflow Skill-ification）」

把一套完整的专业咨询工作流（技术审计 → 客户报告 → CRM 跟进 → 月度对比）每个阶段封装为一个独立 skill，通过顶级编排 skill 串联，Python 脚本处理无法在 NL 层完成的数据处理任务。

模式特征清单：
- 特征 1：**工作流 = skill 集合**——每个业务阶段有对应 skill，不同 skill 覆盖不同阶段
- 特征 2：**顶级编排 skill 作为注册表**——`geo/SKILL.md` 是「接线板」，登记所有子 skill 及其关系
- 特征 3：**Python 脚本处理数据密集任务**——NL 层负责理解和判断，脚本层负责数据处理和格式转换
- 特征 4：**业务工作流可见**——从 skill 命名（geo-prospect、geo-proposal、geo-compare）直接读出商业咨询流程
- 特征 5：**外部 schema 文件作为知识库**——`schema/` 目录中的 JSON schema 是结构化知识，与 NL skill 并存

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 专业咨询工作流自动化 | ✅ 高度适用 | 每个咨询步骤 = 一个 skill，业务边界清晰 |
| 数据量大、需要脚本处理 | ✅ 适用 | NL + Python 混合可以处理报告生成、爬虫等任务 |
| 严格安全环境（企业内网） | ⚠️ 谨慎 | Python 脚本引入了 SSRF 和 debug 模式等安全问题 |
| 通用开发辅助 | ❌ 不适用 | 高度领域专属（GEO/SEO），不适合迁移 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：业务工作流 skill 化 | geo-seo-claude | 业务覆盖全面，脚本处理复杂数据 | Python 脚本引入安全面，非 GEO 领域难以复用 |
| 纯 NL skill 集合 | obra/superpowers | 零安全风险（无执行层），轻量 | 无法处理 PDF 生成等数据密集任务 |
| Multi-agent 流水线 | zhsama/claude-sub-agent | 阶段门控，质量可测量 | 配置复杂，不适合业务咨询场景 |

### 2.4 改进空间

1. **当前问题**：5 个 agent 均无 `model:` 声明，`geo/SKILL.md` 说这些 agent 并行执行，但没有指定模型层级。**改进做法**：轻量分析 agent（geo-content）用 haiku，复杂技术 agent（geo-technical、geo-ai-visibility）用 sonnet，区分成本和能力。**预期收益**：并行 5 个 agent 的成本可降低 30-50%。

2. **当前问题**：`scripts/fetch_page.py` 的 URL 来源验证（scheme 检查）已修复，但未添加超时配置——在爬虫场景下，慢响应的目标 URL 会阻塞整个流程。**改进做法**：`requests.get(url, timeout=10)` 添加超时参数。**预期收益**：防止慢 URL 阻塞 GEO 审计流程。

3. **当前问题**：`skills/geo-brand-mentions/SKILL.md` 模糊量词密度最高（6+ 处「relevant」「appropriate」），这是影响该 skill 评分的主因。**改进做法**：将「分析 relevant mentions」改为「分析过去 30 天内包含品牌名称的公开 AI 回应」——量词 → 可测量标准。**预期收益**：geo-brand-mentions score 从 88 提升至约 96。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **88/100**（20 个工件）；安全评级：CLEAR（5 个发现，均为 Medium/Low）。

| 文件范围 | 当时分数 | 主要问题 |
|---|---|---|
| 5 个 agent 文件 | 74–76/100 | 无 model 声明（-5）+ 零示例块（-15） |
| skills/geo-brand-mentions | 88/100 | 6+ 处模糊量词（-12） |
| skills/geo-report | 90/100 | 破损 skill 引用（geo-llms-txt）+ 模糊量词 |
| skills/geo-content, geo-schema | 90/100 | 5 处模糊量词（-10） |
| skills/geo-prospect | 98/100 | 几乎完美 |

### 3.2 当时值得借鉴的模式

1. **顶级 skill 作为编排注册表**：`geo/SKILL.md` 不是一个功能 skill，而是一张「接线板」——它列出所有 5 个 agent 和 15 个 sub-skill，以及谁调用谁。这让系统的架构从文件列表变成了「可读的流程图」。路径：`geo/SKILL.md`。借鉴：在 bureau 中，用 `BUREAU.md` 或一个顶级 skill 记录各 skill 的调用关系。

2. **业务流程命名（geo-prospect → geo-proposal → geo-compare）**：skill 命名直接对应客户咨询生命周期（获客 → 提案 → 月度对比），业务逻辑从命名中直接可读。借鉴：gstack 的 skill 也可以按项目生命周期命名（issue → spec → ship → retro），让 `ls skills/` 变成一张流程图。

3. **schema 作为结构化知识**：`schema/` 目录中的 6 个 JSON schema 文件（organization.json、product-ecommerce.json 等）是供 agent 参考的结构化知识库。这把「参考文档」变成了「可解析的配置」。借鉴：在 bureau 中，将常用的数据结构（如 session 记录的字段规范）写成 JSON schema 而非只在 SKILL.md 中描述。

4. **geo-prospect/SKILL.md 的高质量**（98/100）：该 skill 语言精准、流程可验证、无模糊量词，是本仓库的标杆。可作为写新 skill 时的参照。

### 3.3 当时的缺陷

1. **`skills/geo-report/SKILL.md` 引用 `geo-llms-txt`（带连字符），实际 skill 名为 `geo-llmstxt`（无内部连字符）**。根本原因：skill 引用的名称和实际安装的 skill 目录名不一致，差一个连字符。NLPM 无法发现这类错误，因为只是字符串不匹配而非语法错误。这种 typo 的危害是隐性的——`geo-report` skill 会静默地跳过 llms.txt 评估，产出的报告看起来完整但实际上缺少了一个关键维度。**自查**：我的 SKILL.md 中引用其他 skill 时，skill 名是否与实际目录名完全一致（包括连字符、下划线的细节）？

2. **Flask `app.run(debug=True)` 在生产入口文件中**。根本原因：开发阶段为了方便调试，hardcode 了 `debug=True`；发布时忘记改回来。这是一个「开发习惯带入生产」的经典问题——Werkzeug interactive debugger 在开放端口时允许浏览器执行任意 Python 代码。**自查**：我的 Python 项目（如果有）有没有写死 `debug=True`？

3. **`scripts/fetch_page.py` 和 `scripts/llmstxt_generator.py` 无来源验证**。根本原因：脚本为开发者本地使用设计，假设输入是可信的；但在自动化 pipeline 中，URL 来源是用户提供的，不可信。SSRF（服务器端请求伪造）的核心风险：攻击者提供 `http://169.254.169.254/latest/meta-data/`（AWS 元数据服务 URL），脚本会帮他访问内网资源。**自查**：我的脚本中有没有 `requests.get(user_provided_url)` 这样的模式，而没有先验证 URL scheme 和 host？

### 3.4 当时的优化机会

1. **Bug fix（最高优先级）**：`skills/geo-report/SKILL.md:23` 将 `geo-llms-txt` 改为 `geo-llmstxt`——一字符改动，修复静默数据丢失
2. **Security fix（Medium）**：`scripts/webapp/app.py:214` 将 `debug=True` 改为读环境变量——标准做法，3 行代码
3. **Security fix（Medium）**：`fetch_page.py` 添加 URL scheme 校验 + `llmstxt_generator.py` 添加域名来源校验——各约 3-5 行代码

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| geo-report/SKILL.md 引用 `geo-llms-txt`（typo） | `grep "geo-llms" skills/geo-report/SKILL.md` | ✅ **已修复**（PR #50，2026-04-29）：现在正确引用 `geo-llmstxt` | Bug 在 audit 后 23 天内修复，数据完整性恢复 |
| Flask `debug=True` hardcode | `grep -n "debug" scripts/webapp/app.py` | ✅ **已修复**（PR #51）：改为 `debug = os.environ.get("FLASK_DEBUG", "false").lower() == "true"` | 正确的环境变量模式，默认安全 |
| 5 个 agent 无 `model:` 声明 | `grep -l "^model:" agents/*.md \| wc -l` | ❌ **仍存在**：0/5 agent 有 model 声明 | 最大的质量改进点未触及；audit 推荐的「批量 PR 添加 model 声明」未被执行 |

### 4.2 架构演进

从 audit（2026-04-06）到当前 HEAD：

1. **`geo-llmstxt` skill 目录**：确认仍存在，naming 现在与 geo-report 中的引用完全一致
2. **`skills/geo-update/`**：新增了一个 `geo-update` skill（audit 时不存在），说明插件在持续扩充功能
3. **`schema/` 目录**：6 个 JSON schema 文件完整保留，结构化知识库稳定

作者后来意识到：**可执行的修复清单比模糊的改进建议更有价值**。NLPM 的 audit 报告给出了精确到行号的修复建议（`skills/geo-report/SKILL.md:23`），作者直接用它做 checklist，逐条执行。这是「机器可读 audit 格式」的价值体现。

xiaolai 案例记录了一个值得注意的细节：5 个 PR 在 16 秒内合并，说明这是单次 Claude Code session 完成的批量操作，而非 5 次手动 review。维护者把 audit 报告作为 Claude Code 的输入，让 AI 完成了修复——这个「audit → AI 执行 → merge」的模式本身就是 NL 编程的一个示范。

### 4.3 新增的可学习模式

- **`skills/geo-update/`**：审计后新增的 skill，暗示作者在 audit 期间发现了「月度对比」场景还缺乏覆盖，在修复 bug 的同时补充了新功能。这说明 audit 过程本身可以作为「功能发现」的触发器，不只是「问题修复」。

---

## 五、校准

### 5.1 我已经在做对的

1. bureau 和 gstack 的 Python 脚本（如果有）没有 `debug=True` 的 hardcode——这一点我没有犯同样的错误
2. bureau 的 skill 文件有 `<example>` 块——geo-seo-claude 的 5 个 agent 全部缺少示例
3. gstack 的 skill 命名（spec、retro、ship）已经反映了开发生命周期，与 geo-seo-claude 的「业务流程命名」模式一致

### 5.2 挑战 / 验证

**挑战**：xiaolai 案例（来源：`case-studies/2026-04-29-zubair-trabzada-geo-seo-claude.md`）记录了维护者在 audit 后「7 天 5 PR」的快速响应。这让我重新思考「audit 报告的格式」：如果我的自查报告也精确到文件名和行号，是不是更容易让未来的自己（或 Claude）直接执行修复，而不只是「知道有问题」？精确度是行动力的前提。

**验证**：「连字符 vs 无连字符」这一字之差（`geo-llms-txt` vs `geo-llmstxt`）验证了「skill 名称必须严格匹配」的 NLPM 规则。这种错误无法被语法检查发现，只能通过运行时行为（skill 被静默忽略）或专门的 linting 工具（如 `bin/nlpm-check`）检测。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 bureau 的 skill 交叉引用名称是否与实际目录名完全匹配
# 1. 提取所有 SKILL.md 中引用其他 skill 的名称
grep -rn "skill.*geo-\|load.*skill\|use.*skill" /tmp/my-repos/MarkQWu-bureau/skills/ 2>/dev/null | head -10
# 2. 对比实际 skill 目录名
ls /tmp/my-repos/MarkQWu-bureau/skills/
# 命中后怎么办：若引用名与目录名不一致，统一为目录名

# 检查我的 Python 脚本中有无 debug=True 硬编码
grep -rn "debug.*True\|debug=True" /tmp/my-repos/MarkQWu-*/  2>/dev/null | grep "\.py" | grep -v ".git"
# 命中后怎么办：改为读环境变量，默认 False

# 检查我的脚本中对用户来源 URL 的 requests.get 调用
grep -rn "requests\.get\|requests\.post" /tmp/my-repos/MarkQWu-*/ 2>/dev/null | grep -v ".git" | head -10
# 命中后怎么办：确认 URL 来源；若来自外部输入，添加 scheme 白名单和超时
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 中创建「顶级编排 skill」——类似 `geo/SKILL.md` 的接线板，列出所有 bureau skill 及其调用关系
   - **为何可行**：bureau 已有 7 个 skill（capture、compile、review、recall、lint、scribe、guide），但没有一个文件描述它们的整体流程关系
   - **第一步**：在 bureau 根目录创建 `BUREAU.md`（已存在，看内容是否已覆盖），若未覆盖则在现有 `BUREAU.md` 中添加「skill 编排关系图」章节，约 20 分钟

2. **想法**：给 gstack 的 5 个核心 skill 添加 `geo-report`-style 的顶层 summary skill，让用户可以用一个命令获取所有工具的状态
   - **为何可行**：gstack 已有 `skill:check` 命令（来自 CLAUDE.md），但无 NL 层的汇总 skill
   - **第一步**：查看 gstack 现有 `skill:check` 脚本的输出格式，评估是否可以包装成 SKILL.md，约 15 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 zubair-trabzada/geo-seo-claude 的核心目的**：将 GEO-SEO 咨询工作流（技术审计 → 报告生成 → 客户提案 → 月度跟踪）自动化
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是「工作流自动化」插件，都有多个 skill 覆盖流程不同阶段 | bureau 是知识管理工作流，geo-seo 是咨询工作流；bureau 无 Python 脚本层 | 高 |
| MarkQWu/gstack | 中 | 都有多个专域 skill，都有按流程阶段命名的 skill | gstack 是开发工具集，geo-seo 是业务工作流 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 交叉引用名称 typo（连字符差异） | `grep -rn "skill" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | bureau 的 skill 无交叉引用（各自独立），不适用 | N/A |
| Agent 缺 model 声明 | `grep -L "^model:" /tmp/my-repos/MarkQWu-bureau/crew/*/agent.md` | bureau auditor agent 有 `model: sonnet` ✅ | N/A |
| Python 脚本 debug=True 硬编码 | `grep -rn "debug=True" /tmp/my-repos/MarkQWu-*/*.py` | 未命中（bureau/gstack 无 Python 脚本）✅ | N/A |
| Agent 无示例块 | `grep -L "<example>" /tmp/my-repos/MarkQWu-bureau/crew/*/agent.md` | bureau 的 auditor agent 需检查是否有示例块 | 高 |

**命中后的具体行动建议**：
- 检查 `bureau/.claude/agents/auditor.md` 是否有 `<example>` 块；若无，在 agent 描述中添加一个「输入：session 路径，输出：audit 报告摘要」的示例，约 10 分钟

### 7.3 别人的更优方案

1. **领域**：顶级编排 skill 作为注册表（`geo/SKILL.md`）
   - **本案例做法**：`geo/SKILL.md` 是整个系统的「接线板」，任何想了解系统架构的人读这一个文件就够了
   - **我的项目现状**：bureau 的 `BUREAU.md` 有系统概述，但各 skill 之间的调用关系没有在一个地方明确声明
   - **如何借鉴**：在 `BUREAU.md` 中添加「skill 调用关系」章节，用简单的列表或 Mermaid 图描述流程（capture → compile → review → recall），约 15 分钟

2. **领域**：精确到行号的自查 checklist（NLPM audit 格式）
   - **本案例做法**：NLPM audit 报告精确到 `skills/geo-report/SKILL.md:23` 和 `scripts/webapp/app.py:214`，维护者可以直接拿报告做 checklist
   - **我的项目现状**：我的自查动作大多是「检查是否有问题」，而非「如果命中，改第 X 行的第 Y 处」
   - **如何借鉴**：在未来的自查 checklist 中，明确给出「命中后改哪个文件的哪行」——让 Claude 可以直接执行，而非仅报告问题

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 文件有 `<example>` 块
- **我的做法**：bureau 的 `recall/SKILL.md`、`review/SKILL.md`、`scribe/SKILL.md` 均有 `<example>` 块，展示典型调用输入和预期输出
- **本案例做法**：geo-seo-claude 的 5 个 agent 均无示例块（-15 each），是得分最低的原因之一
- **意义**：`<example>` 块不只是 NLPM 评分项——它对用户的实际意义是「我知道调用这个 skill/agent 应该输入什么、期望得到什么」。bureau 在这里比 geo-seo-claude 做得更好。

---

## 八、术语表

### <a name="subagent"></a>subagent
> Claude Code 中可被主对话或 command 调用的专职 AI 助手，有独立的 system prompt。geo-seo-claude 的 5 个 geo-* agent 都是 subagent——geo/SKILL.md 协调它们并行执行，每个负责一个审计维度。

### <a name="ssrf"></a>SSRF（服务器端请求伪造）
> 攻击者提供一个指向内网服务的 URL（如云服务商的元数据端点 `169.254.169.254`），让服务器代为发送请求并返回结果，从而绕过防火墙访问内网资源。`fetch_page.py` 在没有 URL 来源验证的情况下，直接 `requests.get(user_url)`，就是典型的 SSRF 风险场景。修复方法：只允许 `http://` 和 `https://` scheme，并可选地验证 host 不在内网 CIDR 范围内。

### <a name="geo"></a>GEO（Generative Engine Optimization）
> 生成式引擎优化，即针对 ChatGPT、Perplexity、Claude 等 AI 搜索引擎的内容优化策略，目标是让内容被 AI 引用。类似传统 SEO（搜索引擎优化），但面向「AI 会不会引用你的内容」而非「你的网页会不会排在第一页」。这是一个 2024-2025 年才大规模出现的新兴领域。

### <a name="llmstxt"></a>llms.txt
> 一种约定俗成的文本文件格式（类似 `robots.txt`），网站放在根目录告知 AI 爬虫「哪些内容适合被 AI 引用、引用时应注意什么」。`geo/skills/geo-llmstxt/SKILL.md` 帮助用户分析和生成自己网站的 `llms.txt` 文件。注意：`llms.txt` 中无内部连字符，而 audit 前的 geo-seo-claude 错误地用了 `geo-llms-txt`（带连字符），导致 skill 引用失败。
