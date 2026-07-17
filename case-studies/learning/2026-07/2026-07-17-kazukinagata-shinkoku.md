# kazukinagata/shinkoku — 学习案例

**仓库**：https://github.com/kazukinagata/shinkoku
**Stars**：335 | **来源**：upstream（exemplar，stars < 500）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-17（基于当前 HEAD v0.6.5）
**主题标签**：nl-binary-hybrid, template-design, vague-quantifier, examples-driven, manifest-discipline

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

kazukinagata/shinkoku（「申告」，日语「报税」之意）是一个日本确定申告（[確定申告](#kakutei-shinkoku)）全流程自动化的 Claude Code 插件，同时包含 NL 技能层和 Python 工具链。关键事实：

- 作者 Kazuki Nagata，聚焦日本个人所得税、消费税、青色申告、ふるさと纳税等细分场景
- Audit 时（2026-04-06）有 26 个 artifact，当前 HEAD 有 **24 个技能**（微缩减，结构优化）
- 版本自 Audit 以来保持 **v0.6.5 未变**，无新版发布
- 独特之处：同时包含 NL 技能（SKILL.md）和 Python 工具（pyproject.toml + uv 管理），形成 NL 表皮 + Python 工具层的混合架构
- 被选为 exemplar，正面体现 R01（无模糊量词）、R04（触发词设计）、R06（可运行示例）、R07（跨件引用）、R08（体量控制）

### 1.2 架构剖析

**目录结构**：

```
shinkoku/
├── .claude-plugin/plugin.json    # 插件注册
├── CLAUDE.md                     # 税务精算规则速查表（内含精确公式）
├── README.md / CHANGELOG.md / SECURITY.md
├── pyproject.toml                # Python 工具链声明
├── uv.lock                       # 依赖锁文件
├── shinkoku.config.example.yaml  # 用户配置示例
├── src/                          # Python 源码（PDF 解析、计算）
├── tests/                        # 测试套件
└── skills/                       # 24 个技能目录
    ├── assess/SKILL.md           # 场景评估
    ├── gather/SKILL.md           # 材料收集清单
    ├── income-tax/SKILL.md       # 所得税计算（873 行）
    ├── e-tax/SKILL.md            # 电子申报（含 scripts/ 目录）
    │   └── scripts/etax-stealth.js  # 浏览器兼容补丁
    ├── tax-advisor/SKILL.md      # 综合税务顾问
    │   └── reference/            # 参考文档目录（注：reference 而非 references）
    ├── settlement/SKILL.md       # 结账核验
    └── …（18 个其他技能目录）
```

**文件类型分布**：24 个 SKILL.md + 1 个 CLAUDE.md（含税务公式表）+ 多个 references/ 参考文档 + Python 源码 + uv 锁文件

**编排关系**：无路由器，依赖每个技能 frontmatter description 的双语触发词覆盖。CLAUDE.md 作为全局精算规则参考，被所有计算类技能隐式依赖。技能之间有「先后顺序」的隐性编排（gather → assess → income-tax/consumption-tax → settlement → e-tax → submit）。

**跨件契约**：每个技能的参考文档（`references/` 目录）存放领域深度资料，SKILL.md 用 `(reference/xxx.md)` 链接引用，保持技能本体简洁。tax-advisor 使用 `reference/`（单数），其他技能使用 `references/`（复数），存在命名不一致。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「法定条文可执行化」——将日本税法条文转化为精确的整数运算公式（含条文号），让 Claude 和 Python 工具都能按法定规则操作，而非估算
- **解决什么问题**：日本确定申告流程繁琐、条文密集，个人纳税人容易遗漏扣除项或用错舍入方式，导致多缴税或申报错误
- **做了什么 trade-off**：每个技能深度纵向（873 行的 income-tax），换取覆盖深度；技能数量较多（24 个），换取细粒度控制；Python 工具链增加了部署复杂度，换取计算准确性
- **反映什么认知模型**：作者把 Claude 视为「税务师助手」，不能靠记忆，必须有法律依据；Python 层做精确计算，NL 层做流程引导

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「领域深度纵向 skill 包 + Python 工具链混合架构（[NL-binary-hybrid](#nl-binary-hybrid)）」

NL 层（SKILL.md）提供流程引导和上下文解释，Python 层（pyproject.toml + src/）提供精确数值计算，两者互为补充。

模式特征清单：
- **特征 1**：CLAUDE.md 内嵌精确的整数运算公式和条文引用，作为全局精算规则速查表
- **特征 2**：每个技能垂直深入单一税种/流程（所得税 / 消费税 / 青色申告 / 结账）
- **特征 3**：Python 工具处理 PDF 解析和精确计算，NL 技能处理用户对话和决策引导
- **特征 4**：references/ 子目录存放领域深度参考（医疗费计算指南、住宅贷款扣除指南等）
- **特征 5**：双语触发词（日英）保证从不同语言场景触发

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有严格法律条文要求的计算（税务/会计/合规） | ✅ 高度适用 | Python 层保证数值精度，NL 层引导流程 |
| 需要处理结构化文档（PDF 单据） | ✅ 适用 | Python 工具链天然适合文档解析 |
| 纯对话类助手（无精确计算需求） | ❌ 过度设计 | Python 层增加了安装复杂度，纯 NL 就够 |
| 需要快速分发（零安装）的轻量插件 | ❌ 不适用 | pyproject.toml 依赖安装增加摩擦 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL + Python 混合（本仓库） | kazukinagata/shinkoku | 计算准确，可处理 PDF | 安装复杂，依赖管理有风险 |
| 纯 NL 平铺 | addyosmani/agent-skills | 零安装，即装即用 | 数值计算靠 Claude 估算，精度不保证 |
| NL + 原生二进制 | 777genius/claude-notifications-go | 性能高，真正离线 | 开发门槛高，需编译 |

### 2.4 改进空间

1. **当前问题**：`skills/setup/SKILL.md:21` 中的安装命令 `uv tool install git+https://github.com/kazukinagata/shinkoku` 未固定版本，任意时刻安装可能得到不同版本  
   **改进做法**：改为 `uv tool install git+https://github.com/kazukinagata/shinkoku@v0.6.5`，并在每次发版时更新  
   **预期收益**：安装行为可重现，消除 Medium 安全风险

2. **当前问题**：`tax-advisor/` 使用 `reference/`（单数），其他 23 个技能使用 `references/`（复数），命名不一致  
   **改进做法**：将 `tax-advisor/reference/` 重命名为 `tax-advisor/references/`，同步更新 SKILL.md 内链接  
   **预期收益**：贡献者不需要记「哪个技能用哪个命名」，认知一致

3. **当前问题**：pyproject.toml 依赖使用 `>=` 下限约束（pydantic>=2.0），可能在未来某个小版本 breaking change 时无声损坏税务计算  
   **改进做法**：明确文档化 `uv.lock` 是权威安装方式；或切换到上限约束（`>=2.0,<3.0`）  
   **预期收益**：减少「装上去但计算错误」的静默错误

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **94/100**（共 26 个 artifact）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 85 | 无 NL frontmatter（设计符合 CLAUDE.md 规范，informational） |
| skills/e-tax/SKILL.md | 88 | 文件超过 25k token 读取上限，评估受限 |
| skills/income-tax/SKILL.md | 92 | 全仓库最长技能（873 行）；部分复杂度导致估算扣分 |
| skills/settlement/SKILL.md | 96 | "妥当か"（是否妥当）×2 次模糊量词 |
| skills/tax-advisor/SKILL.md | 96 | "網羅的に"（综合地）模糊量词；reference/ 命名不一致 |
| skills/furusato/SKILL.md | 97 | "適切に設定されているか"（是否适当设定）模糊量词 |
| 其余 20 个技能 | 95-96 | 整体干净 |

### 3.2 当时值得借鉴的模式

1. **税法公式精确化（R01）**：CLAUDE.md 内的舍入规则表达为整数运算表达式（如 `tax * 21 // 1000`）并注明条文号，消除了所有「合理舍入」类模糊词 → 原文：CLAUDE.md 税额计算表格

2. **日英双语触发（R04）**：每个技能 description 既有日文场景触发词（"必要書類"、"書類を集める"）也有英文触发句（"what documents to collect"），8 个 JP 触发短语 + 1 个 EN 场景句 → 原文：`skills/gather/SKILL.md:3-9`

3. **CLI 可运行示例（R06）**：consumption-tax 技能包含完整的 CLI 调用示例，含具体 JSON 输入和逐字段输出描述，Claude 无需推断参数格式 → 原文：`skills/consumption-tax/SKILL.md:102-132`

4. **跨件决策树（R07）**：每个技能有基于可观测条件的转跳表（"当用户拥有副业时 → `/income-tax`"），而非通用「另见」

5. **参考文档分层（R08）**：SKILL.md 保持 <150 行，深度资料在 references/ 子目录（如 medical-expenses.md、housing-loan.md 等）

### 3.3 当时的缺陷

1. **问题**：`skills/setup/SKILL.md` 的安装命令 `uv tool install git+https://github.com/kazukinagata/shinkoku` 无版本号  
   **根本原因**：git URL 安装绕过了 PyPI 的版本管理，没有固定版本 = 每次安装可能得到不同代码；更严重的是如果 GitHub repo 被攻击，安装行为会变成供应链攻击  
   **自查**：我的技能中有没有 `git clone` 或 `pip install git+`？

2. **问题**：三处日语模糊量词未替换（"妥当か"、"適切に設定されているか"、"網羅的に"）  
   **根本原因**：日语的「妥当/適切」是法律文书常用词，作者未意识到 AI 无法从中提取操作指令  
   **自查**：我用日文/中文写技能时有没有类似「合理/适当」类词？

3. **问题**：e-tax/SKILL.md 文件超过 25k token 读取上限，审计工具无法完整读取  
   **根本原因**：日本 e-Tax 系统的操作流程非常复杂，作者把所有步骤放入一个 SKILL.md  
   **自查**：我的技能中有没有单文件超过 500 行（NLPM 软限制）或接近工具读取上限的？

### 3.4 当时的优化机会

1. 在 setup/SKILL.md 中把安装命令固定到版本 tag（`@v0.6.5`）
2. 把三处日语模糊量词替换为具体标准（如"残高がマイナスでないか（前年比±50%超の異常値）"）
3. 把 e-tax/SKILL.md 拆分为「主 SKILL.md（触发 + 总流程）」+「分节技能（各表单填写细节）」

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| setup/SKILL.md 安装无版本号 | `grep "uv tool install" skills/setup/SKILL.md` | **持续存在**（无 @tag） | 作者未修复 Medium 安全风险 |
| "妥当か" 模糊量词（settlement，×2） | `grep "妥当か" skills/settlement/SKILL.md` | **持续存在**（line 98, 279） | 未视为优先问题 |
| "適切に設定されているか"（furusato） | `grep "適切に" skills/furusato/SKILL.md` | **持续存在**（line 252） | 同上 |
| "網羅的に"（tax-advisor） | `grep "網羅的" skills/tax-advisor/SKILL.md` | **持续存在**（line 148） | 同上 |
| reference/ 命名不一致（tax-advisor） | `ls skills/tax-advisor/` | **持续存在** | 同上 |
| pyproject.toml 依赖无上限约束 | `grep "pydantic\|pdfplumber" pyproject.toml` | **持续存在**（仍为 `>=` 格式） | 作者新增了 types-pyyaml 依赖，但未改约束格式 |

### 4.2 架构演进

从 Audit 时（26 artifact，v0.6.5）到现在（24 技能，**仍为 v0.6.5**）：

- **版本号完全未变**：这是本批次 4 个案例中唯一一个 Audit 后没有发版的仓库，说明作者可能进入维护期或暂停开发
- **技能数微缩（26→24）**：可能有 2 个 artifact 被合并或移除，整体结构未大变
- **新增依赖 types-pyyaml**：pyproject.toml 增加了 `types-pyyaml>=6.0.12.20250915`，显示有小型增量维护
- **e-tax/scripts/etax-stealth.js 仍然存在**：浏览器 UA 伪装脚本未被移除，仍是 LOW 安全发现

整体判断：shinkoku 目前处于低频维护状态，大改可能性小。这对学习来说是好事——架构在 Audit 时的状态基本就是「成熟定型」状态，可以放心从稳定版学习。

### 4.3 新增的可学习模式

暂无——版本从 Audit 时以来未升级，未观察到新增可学习模式。etax-stealth.js 中的 Low 安全发现（浏览器 UA 伪装）仍存在，含有更好的注释说明用途（现符合 Audit 建议），但不构成新模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **参考文档分离**：我的 gstack/plan-devex-review/sections/ 目录与 shinkoku 的 references/ 目录模式相近，均为共享参考层
2. **可运行代码示例**：gstack 的多个技能包含具体 bash 命令和 git 命令示例，而非描述性文字
3. **精确数值优先**：我的 land-and-deploy/SKILL.md 中的超时设置有具体秒数（"30-second intervals"），而非"reasonable timeouts"
4. **尝试双语/多语支持**：bureau 的技能 description 同时包含中英触发意图（虽未完全做到 shinkoku 级别的 8 个 JP 触发短语）

### 5.2 挑战 / 验证

本案例**验证**了「公式 + 条文引用」比「合理计算」更能消除模糊量词的直觉——shinkoku 被选为 R01（无模糊量词）exemplar 的核心正是 CLAUDE.md 里的整数运算表格。这比单纯「删掉 appropriate」更进一层：直接给出机器可执行的表达式。

本案例**挑战**了「exemplar 仓库的维护者会积极修复已知问题」的假设——shinkoku 有 5 个持续存在的已知缺陷，包括 Medium 安全风险，3 个月以来零修复。exemplar 资格基于某个时间点的最佳实践，不代表维护者的响应速度。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能中是否有未固定版本的安装命令
grep -rn "git+https://\|pip install\|uv tool install" \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：如果有 `git+https://` 安装命令，添加 `@tag` 固定版本；如果是 pypi 包，改为精确版本号。

```bash
# 检查我的技能中是否有单文件超过 500 行
wc -l /tmp/my-repos/MarkQWu-gstack/*/SKILL.md | sort -rn | head -10
```
命中后怎么办：超过 500 行的技能考虑拆分——主 SKILL.md 保留触发逻辑和总步骤，细节移入 references/ 子目录。

```bash
# 检查参考文档命名一致性（references vs reference）
find /tmp/my-repos/MarkQWu-gstack -type d -name "reference*" 2>/dev/null
find /tmp/my-repos/MarkQWu-bureau -type d -name "reference*" 2>/dev/null
```
命中后怎么办：统一为 `references/`（复数），与 NLPM 约定一致。

### 6.2 灵感 → 实施路径

1. **想法**：在 MarkQWu/bureau 的 CLAUDE.md 中加入类似 shinkoku 的「公式精算规则表」——把 bureau 的信任等级判断（tier 0/1/2）转化为可检查的布尔条件  
   **为何可行**：shinkoku CLAUDE.md 的税务公式表证明了「把人工判断转化为可执行表达式」可以显著提升 AI 行为一致性  
   **第一步**：在 bureau/CLAUDE.md 中新增「信任度评估表」一节，每个 tier 用「会话数 >= X AND 人工确认 = true」等可检查条件表述，30 分钟可完成

2. **想法**：仿照 shinkoku 的 references/ 模式，把 gstack/land-and-deploy/SKILL.md 的平台配置细节拆出  
   **为何可行**：shinkoku income-tax 873 行的经验说明「大型技能应该拆分」，land-and-deploy 目前超过 1800 行  
   **第一步**：新建 `gstack/land-and-deploy/references/PLATFORM_CONFIG.md`，把 Vercel/Cloudflare/Heroku 各平台配置步骤移入，主 SKILL.md 保留 <500 行，1-2 小时可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 kazukinagata/shinkoku 的核心目的**：自动化日本个人报税全流程，NL 技能引导 + Python 工具精确计算

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 中 | 同为「AI 辅助复杂领域知识处理」；都有 NL 技能 + 代码工具 | graphify 是代码知识图谱；shinkoku 是税务申报 | 中（NL+工具混合架构可借鉴） |
| MarkQWu/bureau | 中 | 同为有 Python 工具链的 Claude Code 插件 | bureau 是知识管理；shinkoku 是领域专家 | 中（references/ 模式可借鉴） |
| MarkQWu/gstack | 低 | 同为多技能套件 | gstack 聚焦技术操作；shinkoku 聚焦领域合规 | 低 |
| MarkQWu/shiji-kb | 低 | 同有中日文内容 | shiji-kb 是古诗检索；shinkoku 是税务向导 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 无版本号的 git 安装命令 | `grep -rn "git+https" /tmp/my-repos/MarkQWu-*/*/SKILL.md` | 未命中 | 无 |
| 单技能文件超大（>500 行） | `wc -l /tmp/my-repos/MarkQWu-gstack/land-and-deploy/SKILL.md` | **命中：1883 行**（land-and-deploy） | 高 |
| 参考文档命名不一致 | `find /tmp/my-repos/MarkQWu-gstack -type d -name "reference*"` | 未命中（gstack 无 references/ 目录） | 无（但缺少分离本身是问题） |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的 `land-and-deploy/SKILL.md`（1883 行）→ 创建 `land-and-deploy/references/` 子目录 → 把「各平台详细配置」和「错误处理细节」移入 → 主文件缩减到 500 行以内 → 1-2 小时可完成

### 8.3 别人的更优方案

1. **领域**：法定条文 + 精确公式写入 CLAUDE.md
   - **本案例做法**：CLAUDE.md 直接用 Python 整数运算表达式（`(amount // 1_000) * 1_000`）+条文号（「国税通則法118条」）描述每条舍入规则，Claude 无需「估算」
   - **我的项目现状**：MarkQWu/bureau 的 CLAUDE.md 有信任等级说明，但描述为「高信任度的 session」这类模糊措辞，没有可检查的条件
   - **如何借鉴**：bureau/CLAUDE.md 中把 tier 判断改为「tier2 = 人工 review = confirmed AND session_count >= 3」等精确条件

2. **领域**：8 个日文触发短语并排在 description
   - **本案例做法**：每个技能 description 枚举 6-8 个具体日文用词（"必要書類"、"書類を集める"、"何を準備すればいい"），覆盖用户不同措辞
   - **我的项目现状**：MarkQWu/gstack 的 triggers 字段有 2-4 个英文短语，覆盖不够全面
   - **如何借鉴**：把 gstack/spec/SKILL.md 的触发短语从 2 个扩展到 8-10 个，覆盖「写规格」「需求文档」「PRD」「功能说明」等不同措辞（10 分钟）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：技能文件体量控制
  - **我的做法**：MarkQWu/bureau 的 skills/capture/SKILL.md 保持在 200 行以内，触发逻辑和核心步骤清晰分离
  - **本案例做法（弱在哪）**：skills/income-tax/SKILL.md 达到 873 行，skills/e-tax/SKILL.md 超过 25k token 读取上限——审计工具无法完整读取意味着 CI 检查失效
  - **意义**：bureau 的技能文件体量控制更好，任何工具都可以完整读取；shinkoku 的「超大单文件」模式是一个技术债

---

## 八、术语表

### <a name="kakutei-shinkoku"></a>確定申告（かくていしんこく）
> 日本的个人所得税年度申报制度，相当于中国的个税汇算清缴。每年 2 月-3 月，有副业、自营、资本利得、医疗费扣除等情况的纳税人需要向税务署提交申报表，计算当年实际应缴税额。比中国的流程更复杂，有青色申告（bookkeeping-based，可获更多扣除）和白色申告之分。

### <a name="nl-binary-hybrid"></a>NL-binary-hybrid（NL 表皮 + 原生工具层）
> 一种 Claude Code 插件架构：顶层是 NL 技能（SKILL.md）提供对话界面和流程指导，底层是本地代码工具（Python/Go/Rust 等）提供精确计算或系统操作。NL 层负责「做什么」和「结果解释」，代码层负责「怎么算」和「精确执行」。shinkoku 用 Python 解析 PDF 税务单据，就是这种模式的典型。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道如何注册和触发这个技能。

### <a name="uv"></a>uv
> 一个由 Astral 开发的极速 Python 包管理和虚拟环境工具，功能类似 pip + virtualenv 的组合，但速度快 10-100 倍。shinkoku 使用 uv 管理 Python 依赖，`uv.lock` 是其锁文件（记录精确的依赖版本树）。

### <a name="supply-chain-attack"></a>供应链攻击
> 攻击者控制了一个被目标信任的上游资源（如 GitHub repo、npm 包），通过这个中间环节向受害者传递恶意代码。shinkoku 的 `git+https://github.com/...` 安装命令如果不固定版本，攻击者劫持 GitHub 账号后就可以通过这个安装命令向所有用户推送恶意代码。
