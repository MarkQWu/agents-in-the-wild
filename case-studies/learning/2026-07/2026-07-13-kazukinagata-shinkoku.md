# kazukinagata/shinkoku — 学习案例

**仓库**：https://github.com/kazukinagata/shinkoku
**Stars**：335 | **来源**：upstream audit（exemplar_published=true）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-13（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `template-design`, `examples-driven`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`kazukinagata/shinkoku`（「申告」= 日文"确定申告/tax filing"）是一个专门帮助日本个人纳税者（公司员工 + 副业/事业所得）完成青色申告全流程的 Claude Code 插件。作者 kazukinagata 构建了端到端的税务申报辅助系统，覆盖从"收集资料"到"电子申报"的完整流程。

截至 2026-07-13 当前 HEAD：
- **版本 0.6.5**（与 audit 时一致，未做版本更新）
- **26 个 skill**（与 audit 时一致）+ 大量新增文档
- **新增**：CHANGELOG.md、docs/（含 system-overview.md 和各类研究文档）、CONTRIBUTING.md、CODE_OF_CONDUCT.md、SECURITY.md、Makefile、.pre-commit-config.yaml、`.github/workflows/`（3 个 workflow）
- **e-tax/research/**：新增 14 个逐屏操作记录文件（从 CC-AA-010 到各 kessan 截面）
- MIT 协议，由 uv 工具链管理（pyproject.toml + uv.lock）

### 1.2 架构剖析
- **目录结构**：
  ```
  shinkoku/
  ├── skills/                        # 26 个 SKILL.md
  │   ├── capabilities/              # 能力清单 + 范围边界
  │   ├── gather/                    # 资料收集
  │   ├── assess/                    # 税额估算
  │   ├── income-tax/                # 所得税（873 行，最长）
  │   ├── consumption-tax/           # 消费税
  │   │   └── references/tax-classification.md
  │   ├── settlement/                # 决算
  │   ├── e-bookkeeping-compliance/  # 电子账簿保全
  │   ├── e-tax/                     # 电子申报（>25k token）
  │   │   ├── SKILL.md
  │   │   ├── scripts/etax-stealth.js  # 浏览器 UA 伪装脚本
  │   │   └── research/              # 新增：14 个逐屏记录
  │   ├── setup/                     # 安装配置
  │   ├── furusato/                  # 故乡纳税
  │   ├── tax-advisor/               # 税务总顾问
  │   │   └── reference/             # 注意：singular（其他 skill 用 references/）
  │   ├── reading-*/                 # 6 个证明书读取 skill
  │   ├── tax-*-context/             # 4 个上下文提供 skill
  │   └── ... （共 26 个）
  ├── docs/                          # 新增：设计方案、系统概览、WSL 说明
  ├── CHANGELOG.md                   # 新增
  ├── CONTRIBUTING.md / CODE_OF_CONDUCT.md / SECURITY.md  # 新增
  ├── Makefile                       # 新增
  ├── .pre-commit-config.yaml        # 新增
  ├── pyproject.toml                 # Python 依赖
  └── .claude-plugin/plugin.json
  ```
- **文件类型分布**：26 个 SKILL.md / 0 个 agent / 0 个 command / 1 个 JS 脚本（etax-stealth.js）/ 丰富的 references/ 文件
- **编排关系**：平铺式，没有显式路由器。用户通过名称调用各 skill，skill 间通过 `description` 和 `references` 互相引用（如 `income-tax` 引用 `reading-withholding`）。所有 skill 互相调用链已在 audit 验证无循环引用。
- **跨件契约**：CLAUDE.md 的技能表和 plugin.json 严格对应；`pyproject.toml` 版本和 `plugin.json` 版本同步（均为 0.6.5）；每个 skill 的 `references/` 子目录提供分类知识文件。

### 1.3 设计思路 / 方法论
- **核心设计哲学**："税务精度即法律精度"——所有计算不用模糊描述（"合适的四舍五入"），必须给出整数运算公式和对应法律条文（国税通则法第 X 条）。skill 是纳税指南，不是聊天机器人。
- **解决什么问题**：日本个人税务申报流程复杂（所得税 + 消费税 + 青色申告特别控除 + 电子账簿 + e-Tax 电子提交），涉及大量专业规则和政府网站操作。Claude 没有领域知识时容易犯错；skill 把税务专家经验结构化为可复用的 NL 知识。
- **做了什么 trade-off**：选择"26 个单职责平铺 skill"而非"分层路由"——好处是每个 skill 专注一个税务环节，边界清晰，专家可以独立完善某一 skill；代价是没有 always-on 路由器，用户需要了解 skill 名称才能正确调用。
- **反映什么认知模型**：作者把 Claude 定位为"税务流程执行者"，而非"税务咨询师"——skill 不只给建议，而是告诉 Claude 每一步的具体操作（运行哪个命令，验证哪个字段，捕获哪类错误），实现流程自动化而不只是知识传递。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「法规精度驱动的平铺 Skill 集」**

每个 skill 聚焦单一税务环节，用整数运算公式替代模糊描述，用法律条文替代"合理判断"，用触发短语覆盖日英双语场景。

模式特征清单：
- **特征 1**：每个计算规则附带整数算术公式（`(amount // 1_000) * 1_000`），无需解释如何四舍五入
- **特征 2**：法律条文引用（"国税通则法 118 条"）作为规则来源锚点，可验证可追溯
- **特征 3**：日英双语触发短语覆盖（不只是日文关键词）
- **特征 4**：references/ 目录（或 reference/ 单数，见讨论）提供分类知识文件，技能正文精简
- **特征 5**：CLAUDE.md 中的 tax rounding table 作为全局精度规范，所有 skill 遵守

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 法规驱动的专业领域（税务/法律/医疗） | ✅ 高度适用 | 精度即合规；整数公式+法规引用消除 Claude 的"创意计算"风险 |
| 多步骤流程自动化（有明确前置步骤） | ✅ 适用 | 每个 skill 代表流程中的一步，用户按顺序调用 |
| 需要快速发现和选择的开放场景 | ❌ 有限适用 | 没有路由器，用户需要知道 skill 名称 |
| 频繁更新的规则（如每年修订的税法） | ⚠️ 需谨慎 | skill 内嵌的法律条文需要随法规更新，否则会给出过期建议 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 法规精度平铺 Skill（本仓库） | kazukinagata/shinkoku | 精度高；每 skill 可独立验证；边界清晰 | 没有路由器，用户需要知道 skill 名称 |
| 路由器 + 叶节点 | dontbesilent2025/dbskill | 用户只需描述症状，路由器自动分发 | 路由器膨胀；专业精度可能不如平铺 |
| 单一 monolith skill | 早期小型技能包 | 安装简单，无路由维护 | 规则量大时超过 500 行，Claude 难以检索 |

### 2.4 改进空间
1. **当前问题**：`skills/setup/SKILL.md` 的安装命令 `uv tool install git+https://github.com/kazukinagata/shinkoku` 无版本标签，每次安装都会拉取最新 commit。**改进做法**：改为 `uv tool install git+https://github.com/kazukinagata/shinkoku@v0.6.5`，并在 release CI 中自动更新这个版本标签。**预期收益**：消除 Medium 安全风险，安装结果可复现。
2. **当前问题**：`tax-advisor/SKILL.md` 使用 `reference/`（单数），其他 26 个 skill 都用 `references/`（复数），命名不一致。**改进做法**：把 `skills/tax-advisor/reference/` 重命名为 `skills/tax-advisor/references/`，并更新 SKILL.md 内的所有路径引用。**预期收益**：消除目录命名不一致，贡献者不会困惑。
3. **当前问题**：`skills/settlement/SKILL.md` 中"残高が妥当か"（line 98）和"経費科目が妥当か"（line 279）两处用了模糊量词"妥当"，NLPM -4 分。**改进做法**：替换为"残高がマイナスでないか、または前年比±50%超の異常値がないか"（line 98）和"異常に大きい科目（前年比200%超）または小さい科目（前年0円）がないか"（line 279）。**预期收益**：NLPM 得分 +4，且 Claude 不再依赖主观判断做检查。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
94/100（26 个文件，批次策略审计）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 85 | 无 NL frontmatter（CLAUDE.md 类型豁免） |
| skills/e-tax/SKILL.md | 88 | 文件超 25k token，审计只覆盖前 150 行 |
| skills/income-tax/SKILL.md | 92 | 873 行，长度导致质量代价 |
| skills/settlement/SKILL.md | 96 | "妥当か" ×2（-4 分） |
| skills/tax-advisor/SKILL.md | 96 | "網羅的に" ×1（-2 分） |
| skills/furusato/SKILL.md | 97 | "適切に設定されているか" ×1（-2 分） |
| 其余 20 个 skill | 95 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **税额计算用整数运算公式** → CLAUDE.md rounding table 中：`(amount // 1_000) * 1_000`（千円未満切捨）、`tax * 21 // 1000`（復興税）等，每行附法律条文编号。**为何好**：Claude 看到整数公式后直接计算，不会"合理估算"；法律编号可溯源核查。**如何借鉴**：任何涉及精确计算的 skill，用算术表达式代替文字描述（如"四舍五入到百元"→ `(amount // 100) * 100`）。
2. **双语触发 + 英文 "use when" 前导句** → `skills/gather/SKILL.md` 以"This skill should be used when..."开头，然后跟"Trigger phrases include: 必要書類、書類を集める..."，日英结构化并列。**如何借鉴**：description 先写英文 use-when 句，再附双语触发短语列表，两部分互补而不重复。
3. **per-skill references/ 子目录** → 每个 skill 有自己的 `references/` 目录存放分类知识文件（如 `tax-classification.md`），skill 正文保持精简，引用时指向知识文件路径。**如何借鉴**：超过 200 行的 skill，把专业知识表格、分类表、案例库拆到 references/ 子目录，正文只保留决策逻辑和步骤。

### 3.3 当时的缺陷
1. **setup.md git URL 未加版本标签（Medium 安全）** → `uv tool install git+https://github.com/kazukinagata/shinkoku` 不带 `@vX.Y.Z`，每次安装拉取最新 commit，供应链完整性无保障。根本原因：作者认为自己控制 repo，不需要 pin——但 git clone 绕过了 package registry 的完整性机制（如 pypi 的 hash 校验）。自查：我的 skill 或 CLAUDE.md 是否有类似"克隆 git repo 然后本地安装"的指引？
2. **"妥当か" / "適切に設定されているか" 模糊量词** → 日文中"妥当"（appropriate）、"適切"（suitable）和英文的 appropriate/comprehensive 一样是 NLPM 的模糊量词。根本原因：在日文语境中这些词是日常书面语，作者没有意识到它们在 NL programming 视角下是"不可验证的形容词"。自查：我的中文 skill 中是否有类似"合理的"、"适当的"、"适时"等没有定义阈值的词？
3. **e-tax/SKILL.md 超 25k token，审计覆盖不完整** → 审计工具只能读前 150 行，真实质量无法保证。根本原因：e-Tax 申报流程步骤多，作者把所有步骤放在一个文件里。自查：我的 skill 是否有单文件超过 500 行（NLPM 阈值）或预计超过 25k token 的情况？

### 3.4 当时的优化机会（仅供学习）
1. `skills/setup/SKILL.md:21` → 加版本标签：`@v0.6.5`
2. `skills/settlement/SKILL.md:99,279` → 用具体数值标准替换"妥当か"
3. `skills/e-tax/SKILL.md` → 拆分为"e-tax 主协调 skill + 各申告书子 skill"，每个子 skill < 25k token

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| setup.md git URL 无版本标签 | `grep "uv tool install" /tmp/target-kazukinagata-shinkoku/skills/setup/SKILL.md` → `uv tool install git+https://github.com/kazukinagata/shinkoku`（无 @vX） | **仍然存在** | Medium 安全风险 3 个月未修复 |
| settlement.md 中"妥当か" | `grep "妥当" /tmp/target-kazukinagata-shinkoku/skills/settlement/SKILL.md` → line 98, 279 | **仍然存在** | NLPM -4 分问题未改 |
| tax-advisor/reference/ 目录命名 | `ls /tmp/target-kazukinagata-shinkoku/skills/tax-advisor/` → reference（单数） | **仍然存在** | 命名不一致，贡献者困惑 |

### 4.2 架构演进
从 2026-04-06 audit 到 2026-07-13：
- **最大变化 1**：`skills/e-tax/research/` 新增 14 个逐屏操作记录文件（CC-AA-010、CC-AE-090 等）。这说明作者以"研究文档 + 现行 SKILL.md"的组合方式应对 e-Tax 大文件问题，而不是拆分 SKILL.md——knowledge 研究文档独立维护，SKILL.md 引用。
- **最大变化 2**：新增社区标准文件（CONTRIBUTING.md、CODE_OF_CONDUCT.md、SECURITY.md）和 CI workflows（test.yml、claude-code-review.yml）。说明项目开始以"开源协作项目"而非"个人工具"运营。
- **最大变化 3**：`.pre-commit-config.yaml` 和 `Makefile`——引入本地质量门控，说明作者开始把工程化规范引入 NL 插件开发流程。

### 4.3 新增的可学习模式
1. **Research 文档侧车**：`skills/e-tax/research/` 存放逐屏 UI 操作研究记录，SKILL.md 可以引用这些文档而不是把所有细节塞进 SKILL.md。这比拆分 SKILL.md 本身更灵活——研究文档可以随政府网站变更独立更新。
2. **CI 代码审查 workflow**：`.github/workflows/claude-code-review.yml` 用 Claude 自动审查 PR，形成"Claude 帮助维护 Claude 插件"的自洽闭环。这是把 AI 工具引入 NL 插件维护流程的典型案例。
3. **Makefile 标准化构建命令**：Makefile 把常用操作（测试、lint、发布）标准化，降低贡献门槛。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责 skill**：我的 bureau 和 echo-sleuth 都有职责清晰的 skill 边界，与本仓库理念一致。
2. **引用外部知识文件**：echo-sleuth 的 skills/ 目录有专用知识文件（parsing-rules、synthesis-taxonomy），与本仓库的 references/ 模式一致。
3. **版本同步**：我的 bureau 和 echo-sleuth 没有多处版本号（因此无同步问题），是更简单的状态。

### 5.2 挑战 / 验证
本案例验证了我的判断：**"模糊量词在翻译后仍然是模糊量词"**。"合理"（合理时間内）、"妥当"（妥当か）、"适切"（適切に）——无论日文还是中文，NLPM 都识别为 vague quantifier，因为它们都没有给出 Claude 可以程序化判断的阈值。我的中文 skill 中出现"合理"/"适当"/"及时"时，应该同样替换为数值标准。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中是否有"合理的"/"适当的"/"及时"/"合适"等中文模糊量词
grep -rn -E '合理|适当|适时|及时|合适|适度' \
  /tmp/my-repos/MarkQWu-bureau/ \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ \
  --include="*.md" 2>/dev/null | grep -v ".git" | head -10
# 命中后：替换为具体阈值，如"合理时间内"→"60 秒内"；"适时"→"每次 commit 前"
```

```bash
# 检查我的 skill 是否有超过 500 行的单文件
wc -l /tmp/my-repos/MarkQWu-*/skills/**/*.md 2>/dev/null | \
  awk '$1 > 500 {print $0}' | head -10
# 命中后：考虑拆分为主 skill + 子 skill / references 文件
```

```bash
# 检查我的 CLAUDE.md 或 skill 是否有未经版本标签的 git install 指引
grep -rn "git+https://\|git clone\|pip install git" \
  /tmp/my-repos/MarkQWu-*/CLAUDE.md \
  /tmp/my-repos/MarkQWu-*/skills/**/*.md 2>/dev/null | head -5
# 命中后：验证是否有 @vX.Y.Z 标签；无标签则补充或改用 package registry
```

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 的 `skills/` 中专业术语密集的 skill（如 canon 规则 skill）加 `references/` 子目录，把术语表、规则分类表移入，保持 SKILL.md 正文 < 300 行。**为何可行**：本仓库证明 references/ 模式对专业知识密集型 skill 有效。**第一步**：识别 bureau 中最长的一个 skill 文件，把非决策逻辑的内容（术语、案例、示例表格）提取到 `references/glossary.md`，20 分钟可完成。
2. **想法**：在 echo-sleuth 的 CLAUDE.md 中加一个"精度表"，列出所有需要精确格式的字段（如 session timestamp 格式、file path 格式）及对应的正则表达式，消除 Claude 自由发挥的空间。**为何可行**：本仓库的 rounding table 证明"公式表格 > 文字描述"的精度。**第一步**：在 `echo-sleuth/CLAUDE.md` 中新增 `## Output Format Precision` 章节，列出 5 个关键字段的格式规范，15 分钟可完成。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 kazukinagata/shinkoku 的核心目的**：端到端税务申报流程自动化，通过 26 个专域 skill 覆盖从资料收集到 e-Tax 提交的完整流程，精度由法律条文和整数运算公式保障。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是"多步骤流程管理 + 结构化知识"；都有严格的规则系统 | bureau 是知识管理；shinkoku 是税务申报；bureau 有 hooks；shinkoku 无 | 高 |
| MarkQWu/shiji-kb | 低 | 都有大量专业数据（诗词 vs 税法）；都要求精度 | shiji-kb 是知识库；shinkoku 是交互式流程工具 | 低 |
| MarkQWu/graphify | 低 | 都是把专业领域知识结构化为 Claude 可用的 skill | graphify 是代码知识图；shinkoku 是税务流程 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 中文/日文模糊量词（"合理"/"妥当"） | `grep -rn "合理\|适当\|适时" /tmp/my-repos/MarkQWu-bureau/ --include="*.md" 2>/dev/null` | bureau/CLAUDE.md 中未发现；**0 命中** | 无 |
| 单 skill 文件超 500 行 | `wc -l /tmp/my-repos/MarkQWu-bureau/skills/**/*.md 2>/dev/null` | 无 skill 文件；bureau 的 skill 知识由 crew 系统管理 | 不适用 |
| git install 无版本标签 | `grep "git+https://" /tmp/my-repos/MarkQWu-*/CLAUDE.md 2>/dev/null` | 0 命中 | 无 |

**命中后的具体行动建议**：本次扫描均未命中。下一步建议在 bureau 或 echo-sleuth 新增 skill 时，将"无模糊量词"作为 PR checklist 的一条显式规则。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：计算公式的精确表达（整数算术 + 法律引用）
   - **本案例做法**：CLAUDE.md 中有税额计算对照表，每行是 `(amount // 1_000) * 1_000 | 国税通則法118条`，让 Claude 精确复现计算规则。
   - **我的项目现状**：bureau 的规则系统（`canon/` 目录）目前用自然语言描述规则（如"每个条目必须有时间戳"），没有用可执行公式表达精度要求。
   - **如何借鉴**：给 bureau 的 canon 验证规则加"机器可检查格式"——对于有精度要求的规则，写成正则表达式或 Python 条件表达式（如 `len(timestamp) == 25`），而不只是"格式正确的时间戳"。

2. **领域**：逐屏操作研究文档（e-tax/research/）
   - **本案例做法**：为 e-Tax 政府网站的每个操作屏幕建立独立研究文档（CC-AA-010、CC-AE-090 等），SKILL.md 引用这些文档，而不是把所有操作细节塞进单一 SKILL.md。
   - **我的项目现状**：echo-sleuth 的 `skills/` 文件把解析规则和操作步骤混合在一起，没有独立的"操作研究文档"。
   - **如何借鉴**：给 echo-sleuth 中最复杂的 skill（如 JSONL 解析规则）加 `research/` 子目录，把各种边界案例（malformed JSON、empty session、nested tool calls）的研究记录独立存放，SKILL.md 只引用，不内嵌。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：hooks 执法层
- **我的做法**：MarkQWu/bureau 有 `hooks.json` 和 `compile.md` 等强制前置步骤，通过 hooks 确保 canon 条目在提交前经过校验（lint.md）。
- **本案例做法**（弱在哪）：shinkoku 没有任何 hook，完全依赖用户手动调用正确的 skill 序列（gather → assess → income-tax → settlement → e-tax），如果用户跳过步骤，没有任何机制提示。
- **意义**：对于多步骤流程（bureau 的 note → compile → review → canon；shinkoku 的 gather → assess → file），hooks 是避免用户跳步的关键机制。shinkoku 如果加一个 SessionStart hook 注入"当前推荐下一步"的上下文，用户体验会大幅提升。

---

## 八、术语表

### <a name="青色申告"></a>青色申告（あおいろしんこく）
> 日本税务制度中，满足条件的个人事业主可以选择的申告方式。与简单的"白色申告"相比，青色申告可以享受"青色申告特别控除"（最高 65 万円）、损失结转等优惠，但要求更严格的账簿记录（一般需双式记账）和电子账簿保全。

### <a name="e-Tax"></a>e-Tax
> 国税庁（日本国税局）提供的网上税务申报系统，可以在线提交所得税、消费税等申告书，无需纸质提交。由于该网站只官方支持 Windows 系统，`skills/e-tax/scripts/etax-stealth.js` 通过伪装浏览器 User-Agent 让 Linux 用户也能访问。

### <a name="uv"></a>uv
> 一个极快的 Python 包管理器（类似 pip，但以 Rust 编写，速度更快）。`uv.lock` 是锁文件，记录了所有依赖的精确版本；`uv tool install` 安装命令行工具。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> NLPM 评分系统中的术语，指没有给出可验证数值标准的形容词，如"合理的"、"适当的"、"appropriate"、"comprehensive"。这些词让 Claude 需要自行判断"什么算合理"，可能导致每次执行结果不一致。NLPM 的 R01 规则要求用精确数值或条件替代这类词。

### <a name="references"></a>references/ 目录
> skill 文件夹内的知识侧车目录，存放分类知识文件（如税率表、分类表、错误目录）。SKILL.md 正文通过路径引用这些文件，保持正文精简。shinkoku 中大部分 skill 用 `references/`（复数），只有 tax-advisor 用 `reference/`（单数），这是一个待修复的命名不一致。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的 `name`、`description` 等元数据。Claude Code 读 SKILL.md 时先解析 frontmatter 才能注册和路由这个 skill。CLAUDE.md 类型的文件不需要 frontmatter（豁免）。
