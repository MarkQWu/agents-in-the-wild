# kazukinagata/shinkoku — 学习案例

**仓库**：https://github.com/kazukinagata/shinkoku
**Stars**：335（exemplar_published）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-16（基于当前 HEAD）
**主题标签**：`cross-reference`, `template-design`, `vague-quantifier`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
shinkoku（申告）是日本工程师 kazukinagata 为个人税务申报打造的 Claude Code 插件，聚焦**会社员 + 副业（事业所得/[青色申报](#青色申报)）**的完整确定申告流程。插件以 20+ 个高度专业化的 skill 覆盖全流程：从读取发票/收据/源泉徴収票，到汇总收入/扣除，再到电子申报（e-Tax）提交，以及消费税计算、副业收入申报等。当前版本 0.6.5，335 颗星，是罕见的日语专业领域 Claude Code 插件。

- **创建时间**：2025 年
- **作者背景**：日本工程师，有副业需要自行确定申告的实际用户
- **获取方式**：`claude plugin install` + `uv tool install git+https://github.com/kazukinagata/shinkoku`（[uv 工具](#uv)）
- **生态位置**：日语/日本税务法领域的事实标准 Claude Code 插件，无直接竞品

### 1.2 架构剖析

```
kazukinagata/shinkoku
├── skills/
│   ├── tax-advisor/SKILL.md     ← 税务顾问（AI 查阅引导）
│   │   └── reference/           ← 注意：单数，其他 skill 用 references/（不一致）
│   ├── assess/SKILL.md          ← 评估当年申告状态
│   ├── gather/SKILL.md          ← 收集所需材料清单
│   ├── setup/SKILL.md           ← 环境安装设置
│   ├── income-tax/SKILL.md      ← 所得税计算
│   ├── consumption-tax/SKILL.md ← 消费税计算
│   ├── e-tax/SKILL.md           ← 电子申报（1481 行，超大文件）
│   ├── submit/SKILL.md          ← 提交流程
│   ├── settlement/SKILL.md      ← 账目结算
│   ├── journal/SKILL.md         ← 分录记账
│   ├── furusato/SKILL.md        ← 故乡纳税
│   ├── incorporation/SKILL.md   ← 法人化
│   ├── invoice-system/SKILL.md  ← 发票系统（インボイス制度）
│   ├── e-bookkeeping-compliance/SKILL.md ← 电子账簿保存法
│   ├── reading-invoice/SKILL.md ← 解读发票
│   ├── reading-receipt/SKILL.md ← 解读收据
│   ├── reading-withholding/SKILL.md    ← 解读源泉徴収票
│   ├── reading-deduction-cert/SKILL.md ← 解读扣除证明书
│   ├── reading-payment-statement/SKILL.md ← 解读支払調書
│   ├── capabilities/SKILL.md    ← 能力概览（导航）
│   ├── tax-housing-loan-context/SKILL.md ← 住宅贷款相关
│   ├── tax-legal-context/SKILL.md       ← 税法条文引用
│   ├── tax-ebookkeeping-context/SKILL.md← 电子账簿法规
│   └── tax-invoice-credit-context/SKILL.md ← 发票抵扣法规
├── .claude-plugin/plugin.json
├── pyproject.toml + uv.lock    ← Python 工具链
├── skills/e-tax/scripts/etax-stealth.js ← 浏览器 UA 伪装脚本
├── SECURITY.md
└── .pre-commit-config.yaml
```

- **文件类型分布**：24 个 SKILL.md / 1 个 [plugin.json](#manifest) / 1 个 JS 脚本（etax-stealth.js）/ Python 工具链（pyproject.toml + uv.lock）
- **编排关系**：三层设计：①**流程 skill**（assess → gather → income-tax/consumption-tax → submit → e-tax）负责申告主线；②**阅读 skill**（reading-invoice/receipt/withholding 等）负责解读书面材料；③**上下文 skill**（tax-legal-context、tax-invoice-credit-context 等）提供法规参考供其他 skill 调用。`capabilities/SKILL.md` 是导航入口。
- **跨件契约**：所有跨 skill 调用路径（`/reading-receipt`、`/income-tax` 等）均经过验证，无死链。`plugin.json version: 0.6.5` 与 `pyproject.toml version = "0.6.5"` 对齐。**唯一不一致**：`skills/tax-advisor/` 使用 `reference/` 目录（单数），其他 skill 使用 `references/`（复数）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「流程即 skill」——把确定申告的全流程分解为若干可独立调用的 skill，用户可以跟着 skill 一步步完成申告，也可以只调用某一步（如只解读一张源泉徴収票）。
- **解决什么问题**：日本确定申告对有副业的会社员极为复杂（青色申报、各种扣除、e-Tax 系统的 Linux 兼容性问题），对普通人门槛极高。
- **Trade-off**：流程粒度细化有助于准确性，但学习曲线较高——用户需要理解整个申告流程才能知道该调哪个 skill。`capabilities/SKILL.md` 试图解决这个问题。
- **认知模型**：把税务申报视为一个有明确步骤的流程，AI 是「懂税法的助理」，skill 是助理的专业手册。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「流程阶段分解 + 上下文 skill 注入」**——把一个复杂的端到端流程分解为若干阶段 skill（assess/gather/income-tax/submit/e-tax），每个阶段 skill 可调用「上下文 skill」（tax-legal-context 等）来注入领域法规知识。

模式特征清单：
- 特征 1：有明确的流程顺序（assess → gather → calculate → submit）
- 特征 2：「阅读 skill」系列（reading-*）作为共享子任务，多个阶段 skill 可复用
- 特征 3：「上下文 skill」（*-context）专门提供静态法规知识，不执行操作
- 特征 4：导航 skill（capabilities）帮助用户理解整体结构
- 特征 5：版本号在 plugin.json 和 pyproject.toml 双处严格对齐

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有明确流程的领域（医疗、法律、税务、审计） | ✅ 高度适用 | 流程分解自然，skill 职责清晰 |
| 包含大量法规/标准的领域 | ✅ 适用 | 上下文 skill 可以封装法规细节，主流程 skill 保持简洁 |
| 需要频繁外部文档解读的场景 | ✅ 适用 | 阅读 skill 系列解耦了材料解读与流程执行 |
| 动态变化、无固定流程的场景 | ❌ 不适用 | 流程 skill 假设步骤相对固定 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 流程阶段分解（本仓库） | kazukinagata/shinkoku | 职责清晰，复用性强 | 用户需理解流程才能正确调用 |
| 单路由派发 | dontbesilent/dbskill | 用户只需记一个入口 | 路由 skill 是单点，细粒度差 |
| 领域知识库卫星式 | czlonkowski/n8n-skills | 知识深度高 | 无流程引导，离散 |

### 2.4 改进空间
1. **当前问题**：`skills/e-tax/SKILL.md` 长达 1481 行，超出 AI 工具单次读取上限 **改进做法**：按 e-Tax 表单章节拆分为 `e-tax-income.md`、`e-tax-deductions.md`、`e-tax-submit.md`，主 SKILL.md 作为编排层引用这些子文件 **预期收益**：AI 可完整读取每个子文件，审计覆盖率提升，分数从当前 88 提升至 95+
2. **当前问题**：`skills/setup/SKILL.md` 中的 `uv tool install git+https://github.com/kazukinagata/shinkoku` 无版本 pin **改进做法**：改为 `uv tool install git+https://github.com/kazukinagata/shinkoku@v0.6.5`，并在每次版本更新时同步修改 **预期收益**：消除供应链风险，用户安装可复现
3. **当前问题**：`reference/` vs `references/` 目录名不一致 **改进做法**：将 `skills/tax-advisor/reference/` 重命名为 `references/`，更新所有引用 **预期收益**：贡献者不再需要记忆特例

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **94/100**（26 skill，SECURITY CLEAR + 3 安全发现）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 85 | 无 NL [frontmatter](#frontmatter)（CLAUDE.md 类型豁免，info 级别）|
| skills/e-tax/SKILL.md | 88 | 文件超过 25k token 读取上限，审计覆盖受限 |
| skills/income-tax/SKILL.md | 92 | 873 行，复杂度高 |
| skills/settlement/SKILL.md | 96 | "妥当か" 模糊量词 ×2 |
| skills/tax-advisor/SKILL.md | 96 | "網羅的に" 模糊量词 ×1 |
| skills/furusato/SKILL.md | 97 | "適切に設定されているか" 模糊量词 ×1 |
| .claude-plugin/plugin.json | 100 | 满分 |

### 3.2 当时值得借鉴的模式
1. **所有 skill 调用路径零死链** → 为什么好：26 个 skill 相互引用没有一个死链，说明作者每次新增 skill 都同步更新所有引用。原文：`Cross-Component: All inter-skill invocations verified ✓` → 借鉴：新增或重命名 skill 后，搜索全仓库的 skill 引用，确认全部更新
2. **版本号双处严格对齐** → 为什么好：`plugin.json version: 0.6.5` 与 `pyproject.toml version = "0.6.5"` 完全一致，不依赖人工记忆，说明作者有发布前版本对齐检查流程。原文：`Version sync verified ✓` → 借鉴：引入 pre-commit hook 或打包脚本检查两处版本号是否一致
3. **上下文 skill 的知识封装** → 为什么好：`tax-legal-context/SKILL.md`、`tax-invoice-credit-context/SKILL.md` 等专门封装法规条文，其他 skill 调用它们而不是重复内联同样的法规内容。原文：`skills/tax-legal-context/` 目录结构 → 借鉴：把跨多个 skill 重复的领域知识提取为独立的 context skill

### 3.3 当时的缺陷
1. **问题**：`skills/setup/SKILL.md` 安装命令 `uv tool install git+https://github.com/kazukinagata/shinkoku` 无版本 pin **根本原因**：git URL 安装不经过 npm/PyPI 的版本控制机制，任何 push 都立即生效，安全性依赖仓库不被攻陷 **自查**：我的安装指令是否都指向固定版本？
2. **问题**：`skills/e-tax/SKILL.md` 的大小超过 25k token 读取上限 **根本原因**：e-Tax 有大量繁琐的表单说明，作者将其写入单一文件，导致 AI 无法完整读取进行审计或使用 **自查**：我的 SKILL.md 有没有超过 1000 行？超过时该如何拆分？
3. **问题**：`tax-advisor` 使用 `reference/`（单数），其他 skill 用 `references/`（复数）**根本原因**：`tax-advisor` 可能是早期 skill，当时命名约定尚未确立，后来其他 skill 统一用了 `references/`，但 `tax-advisor` 未回填 **自查**：我的 skill 目录结构中有类似的历史遗留不一致吗？

### 3.4 当时的优化机会
1. 在 `skills/setup/SKILL.md` 安装命令中加版本 tag（`@v0.6.5`）
2. 将 `skills/e-tax/SKILL.md` 按表单章节拆分为 3-4 个子文件
3. 将 `skills/tax-advisor/reference/` 重命名为 `references/` 对齐约定

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| git install 无版本 pin | `grep "git+" skills/setup/SKILL.md` → 第 21 行命中 | **持续存在** | 4 个月过去，v0.6.5 未加版本 tag |
| e-tax SKILL.md 超长 | `wc -l skills/e-tax/SKILL.md` → 1481 行 | **持续存在**（从 25k+ token 到 1481 行，仍超出 1000 行最佳实践） | 未拆分 |
| 模糊量词（妥当か/適切に/網羅的に） | `grep "妥当か\|適切に設定\|網羅的に" skills/settlement/SKILL.md skills/furusato/SKILL.md skills/tax-advisor/SKILL.md` → 命中 4 处 | **持续存在** | 全部 4 处模糊量词均未替换为具体标准 |
| reference/ vs references/ 不一致 | `ls skills/tax-advisor/` → `reference/` 仍存在 | **持续存在** | 未对齐 |

### 4.2 架构演进
从审计时的 26 个 skill 到现在，文件结构基本稳定（未明显扩张），新增了：
- `SECURITY.md`（明确文档化安全政策）
- `.pre-commit-config.yaml`（引入 pre-commit 规范化代码质量门）

这说明作者在功能上已经稳定，专注于质量/安全维护，而非继续扩充 skill 数量。`SECURITY.md` 的出现说明作者意识到 `etax-stealth.js` 需要明确的安全说明。

### 4.3 新增的可学习模式
- **`SECURITY.md` + `.pre-commit-config.yaml` 组合**：在插件层面主动声明安全政策（`SECURITY.md`）并用 pre-commit hook 自动执行代码规范，是成熟 skill 库的质量信号。这比只靠人工 code review 更可靠。

---

## 五、校准

### 5.1 我已经在做对的
1. **bureau 的 plugin.json 版本与 package.json 版本一致**，与 shinkoku 的双处版本对齐设计一致
2. **bureau 的 skill 相互引用经过人工验证**，没有死链（与 shinkoku 零死链一致）
3. **gstack 的 SKILL.md 没有超过 1000 行的文件**（分 skill 粒度控制了文件大小）

### 5.2 挑战 / 验证
- **这次案例验证了**：即使是高质量插件（94/100），仍然有 3 处 vague quantifier 在 4 个月后未修复。这说明 vague quantifier 在实际写作中难以自觉发现，需要工具检测（NLPM 这类自动扫描器）而非依赖人工。
- **认知更新**：上下文 skill 模式（`tax-legal-context` 等纯法规文档 skill）是一个我此前未意识到的 skill 设计维度——不执行操作，只提供领域知识注入。这种「被动参考 skill」可以大幅减少其他 skill 的冗余内容。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 文件大小，找出超 800 行的文件
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau \
  -name "SKILL.md" -exec wc -l {} \; | sort -rn | head -10
```
命中后：对超过 800 行的文件，考虑拆分出「EXAMPLES.md」或「PATTERNS.md」卫星文件。

```bash
# 检查日语/中文模糊量词（适当/合理/妥当等）
grep -rn -E '(適切|合理|妥当|適用|十分|十分な|適当)' \
  /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/skills/ \
  --include="SKILL.md"
```
命中后：将模糊量词替换为可验证的具体标准，如"適当な間隔" → "30秒間隔"。

```bash
# 检查我的版本号在多处声明的地方是否对齐
cat /tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json | python3 -c \
  "import json,sys; print('version:', json.load(sys.stdin).get('version'))"
cat /tmp/my-repos/MarkQWu-bureau/package.json | python3 -c \
  "import json,sys; print('version:', json.load(sys.stdin).get('version'))"
```
命中后：若两处版本不一致，更新较旧的那个，考虑引入版本同步脚本。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 创建「上下文 skill」（知识封装层）
   - **为何可行**：bureau 的 compile 和 recall 两个 skill 都需要了解「知识库结构」和「trust tier 规则」，这部分知识目前在两个 skill 里重复定义
   - **第一步**：创建 `skills/bureau-context/SKILL.md`，把知识库结构定义和 trust tier 枚举迁移进去，其他 skill 改为 `See: /bureau-context for structure definitions`；约 20 分钟

2. **想法**：给 gstack 添加 SECURITY.md
   - **为何可行**：gstack 有 `gstack-upgrade` 这类有安全敏感操作的 skill，应该有明确的安全政策文档
   - **第一步**：参考 shinkoku 的 SECURITY.md 格式，创建 `gstack/SECURITY.md`，声明：①哪些 skill 会执行 shell 命令；②哪些操作需要用户二次确认；③如何报告安全问题；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 kazukinagata/shinkoku 的核心目的**：把日本税务申报的复杂流程分解为可逐步执行的 Claude Code skill 集，降低副业者自行申告的门槛

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是有明确流程步骤的工具集，都有知识封装的概念 | bureau 是知识库管理，shinkoku 是税务流程执行 | 高（上下文 skill 模式） |
| MarkQWu/gstack | 低 | 都是技术工具集 | 领域差异大（iOS 开发 vs 税务申报） | 低 |
| 其余仓库 | 无 | — | 领域完全不同 | 无 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 安装命令无版本 pin | `grep "git+\|@latest" bureau/CLAUDE.md gstack/CLAUDE.md` | **0 命中** — bureau 和 gstack 没有 git URL 安装方式 | 无 |
| SKILL.md 超 800 行 | `find . -name "SKILL.md" -exec wc -l {} \; \| sort -rn \| head -5` | **gstack 多个 SKILL.md 超过 800 行**（如 `land-and-deploy`、`ios-design-review`） | 中 |
| 命名约定不一致（目录名） | 检查 skills/ 目录结构命名规律 | **gstack：无前缀统一约定**（`setup-deploy`、`ios-sync`、`diagram` 混用风格）| 低（影响可维护性） |

**命中后的具体行动建议**：
- `gstack/land-and-deploy/SKILL.md` → 把「CI/CD 操作」一节迁移到 `CI_PATTERNS.md`，主 SKILL.md 通过 `See:` 引用 → 15 分钟可完成

### 7.3 别人的更优方案

1. **领域**：上下文 skill 模式
   - **本案例做法**：`tax-legal-context/SKILL.md`、`tax-invoice-credit-context/SKILL.md` 等纯知识 skill 只存储法规内容，不执行操作，其他 skill 按需引用（路径：`skills/tax-legal-context/SKILL.md`）
   - **我的项目现状**：bureau 的 `skills/compile/SKILL.md` 和 `skills/recall/SKILL.md` 都内联了「知识库结构定义」部分，重复了约 200 行
   - **如何借鉴**：创建 `skills/bureau-context/SKILL.md` 提取共享知识，diff 思路：删除 compile 和 recall 中重复的结构定义，改为 `See: /bureau-context`

2. **领域**：pre-commit 代码质量门
   - **本案例做法**：`.pre-commit-config.yaml` 在 commit 前自动运行代码规范检查（路径：`shinkoku/.pre-commit-config.yaml`）
   - **我的项目现状**：gstack 和 bureau 均无 pre-commit hook，代码质量检查依赖手动运行
   - **如何借鉴**：在 gstack 和 bureau 根目录添加 `.pre-commit-config.yaml`，至少配置 `check-yaml`、`end-of-file-fixer`、`trailing-whitespace` 三个基础 hook；约 15 分钟

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：模糊量词自动阻断
- **我的做法**（gstack）：在 SKILL.md 末尾明确列出「禁用的 AI 词汇」列表（`robust`、`comprehensive`、`appropriate` 等），通过规则阻断而非靠自觉
- **本案例做法（弱在哪）**：4 处日语模糊量词（妥当か、適切に設定されているか、網羅的に）在 2026-04-06 的审计后仍未修复（4 个月未动），说明作者在主观写作时无法自觉发现这类问题
- **意义**：gstack 的「禁用词列表 = 规则约束」比「靠作者自觉 = 依赖人为」更可靠。这是 gstack 的一个可展示的设计优势。

---

## 八、术语表

### <a name="青色申报"></a>青色申报（青色申告）
> 日本税务制度中，个体户/副业者可以选择的高级申报方式。相比「白色申报」，青色申报允许扣除最高 65 万日元的「青色申报特别控除」，但要求记录双式簿记（[分录记账](#分录记账)）并保存账簿。shinkoku 主要服务于选择青色申报的会社员 + 副业者。

### <a name="分录记账"></a>分录记账（仕訳）
> 会计双式簿记中，将每笔交易分别记录为「借方」和「贷方」的操作。例如：购买办公用品 1000 円 → 借：消耗品費 1000 / 贷：現金 1000。shinkoku 的 `journal/SKILL.md` 负责帮助用户完成这个步骤。

### <a name="uv"></a>uv
> Astral 公司开发的 Python 包管理工具，类似 pip 但速度快 10-100 倍，支持虚拟环境和工具安装（`uv tool install`）。shinkoku 用 uv 安装命令行工具（`shinkoku` binary）。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，用于声明文件元数据（如 `name`、`description`、`model`）。CLAUDE.md 是项目指令文件，按约定不需要 frontmatter；但 SKILL.md 必须有 frontmatter，否则 Claude Code 无法正确加载该 skill。

### <a name="manifest"></a>plugin.json（manifest）
> 插件清单文件，告诉 Claude Code 插件的名称、版本、作者和包含的组件。shinkoku 的 plugin.json 是满分 100 的参考实现：版本号与 pyproject.toml 严格对齐，所有标准字段都存在。
