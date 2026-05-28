# addyosmani/web-quality-skills — 学习案例

**仓库**：https://github.com/addyosmani/web-quality-skills
**Stars**：1804 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-28（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `template-design`, `examples-driven`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是 Addy Osmani（Google Chrome 团队工程师，《Learning JavaScript Design Patterns》作者）发布的 Claude Code 插件，专注于 Web 质量评审。当前共 6 个技能文件，覆盖 Core Web Vitals、性能、无障碍访问、最佳实践、SEO 和综合审查，外加 CLAUDE.md 主入口和 plugin.json 清单文件。

关键事实：
1. NLPM 评分 **97/100**——在所有已审查的公开仓库中排名前列，接近满分
2. 所有 6 个技能文件均以具体数值阈值定义标准（LCP ≤ 2.5s、INP ≤ 200ms、CLS ≤ 0.1、Total < 1.5 MB）
3. 0 个 bug，0 个安全问题，9 个低严重性质量问题（均为模糊量词）
4. 当前 HEAD 已新增 `codex/.codex-plugin/plugin.json`（Codex 扩展）和 `gemini-extension.json`（Gemini 扩展）——多运行时扩张
5. 历史遗留问题：`skills/core-web-vitals/references/LCP.md` 孤文件至今未被 SKILL.md 引用

### 1.2 架构剖析

```
web-quality-skills/
  .claude-plugin/
    marketplace.json           # 市场发布元数据
    plugin.json                # 主清单文件
  skills/
    accessibility/SKILL.md     # 98 分，1 个模糊词
    best-practices/SKILL.md    # 96 分，2 个模糊词
    core-web-vitals/
      SKILL.md                 # 100 分，零模糊词
      references/
        LCP.md                 # ⚠️ 孤文件：存在但未被引用
    performance/SKILL.md       # 100 分，零模糊词
    seo/SKILL.md               # 92 分，4 个模糊词（最弱）
    web-quality-audit/SKILL.md # 96 分，2 个模糊词
  CLAUDE.md                    # 100 分，主入口
  scripts/analyze.sh           # 只读分析脚本，无安全风险
  codex/.codex-plugin/plugin.json  # NEW：Codex 运行时扩展
  gemini-extension.json            # NEW：Gemini 运行时扩展
  README.md
```

- **文件类型分布**：6 个 skill，0 个 agent，0 个 command，1 个 hook 脚本（只读），1 个主 plugin.json
- **编排关系**：6 个技能通过 CLAUDE.md 整合为一个统一的质量评审工作流，各技能相互引用解析均正确
- **跨件契约**：CLAUDE.md 技能表与磁盘文件一致；plugin.json 路径全部解析——唯一的例外是 references/LCP.md 孤文件

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「数值即标准，Lighthouse 即尺子」——所有质量门控都有具体数值，直接对应 Google Lighthouse 评分体系，无需用户猜测什么叫「好的性能」
- **解决什么问题**：Web 开发者缺乏一个系统的、Claude-native 的质量检查框架——Lighthouse 告诉你分数，但不告诉你 Claude 应该怎么审查和改进代码
- **做了什么 trade-off**：框架无关（agnostic）——所有 6 个技能不假设 React/Vue/Angular，代价是缺乏框架特定的示例；数值门控使标准客观，代价是 SEO 等领域有些内容本质上难以量化（因此出现了"合理数量的链接"这类模糊表达）
- **反映什么认知模型**：作者把「质量」视为可测量的数值集合，而非主观评价——这与 Lighthouse 的哲学完全一致，也是 97/100 高分的根本原因

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「数值门控技能集」（Numeric-Threshold Skill Set）**

这个模式的核心特征：所有技能文件的判断标准以具体数值阈值表达，而非形容词描述。评审时 Claude 只需检查「是否 ≤ 2.5s」，而不是「是否足够快」。

模式特征清单（4 条）：
- 特征 1：CLAUDE.md 在顶部集中声明所有数值阈值（LCP、INP、CLS、Total 大小），作为全局参照基线
- 特征 2：每个技能文件对应一个单一维度（性能 / CWV / 无障碍 / SEO / 最佳实践 / 综合），职责不交叉
- 特征 3：跨技能引用均使用相对路径，路径全部可解析——形成可验证的知识网络
- 特征 4：plugin.json + marketplace.json 严格同步，清单与磁盘文件无漂移

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要客观标准的技术审查领域（性能、安全、可访问性） | ✅ 高度适用 | 该领域本身存在业界公认的数值标准，可直接编码 |
| 框架无关的通用最佳实践 | ✅ 适用 | 无框架假设，适用面广 |
| 需要主观判断的创意类工作 | ❌ 不适用 | 数值门控无法描述「好看的设计」 |
| 需要实时数据的动态检查 | ❌ 需配合脚本 | 纯 skill 文件不能执行，需 scripts/ 辅助 |
| 需要多步骤 agent 协作的复杂 workflow | ⚠️ 部分适用 | 当前无 agent 层，只靠技能驱动 Claude 的原生推理 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 数值门控技能集（本仓库） | addyosmani/web-quality-skills | 标准客观可测，近满分质量，Lighthouse 生态直接对接 | 无 agent 层，对复杂 workflow 支持有限 |
| 主观最佳实践合集 | shanraisshan/claude-code-best-practice | 覆盖面广，适用于无明确数值标准的领域 | 模糊量词密度高，评分偏低，标准难以验证 |
| 多维度代理 + 技能混合 | echo-sleuth-for-claude | 代理和技能分工明确，支持复杂 workflow | 维护成本更高，跨件依赖需要持续同步 |

### 2.4 改进空间

1. **当前问题**：`skills/core-web-vitals/references/LCP.md` 存在于磁盘但未被任何文件引用（孤文件）。**改进做法**：在 `skills/core-web-vitals/SKILL.md` 的 LCP 小节中加一行 `参见：[LCP 详解](references/LCP.md)`。**预期收益**：消除孤文件警告，让这份参考材料真正可被发现；NLPM 交叉件分数从警告变为通过。
2. **当前问题**：`skills/seo/SKILL.md` 有 4 个模糊量词（"relevant"、"reasonable"、"important"、"naturally"），是 6 个技能中最弱的（92/100）。**改进做法**：把"合理数量的链接"改为"每页内部链接 ≤ 100 个（Google 爬虫推荐上限）"；把"重要页面"改为"canonical 页面和 sitemap.xml 中列出的页面"。**预期收益**：SEO 技能从 92 提升到 98+，整体仓库评分达到 99/100。
3. **当前问题**：`skills/web-quality-audit/SKILL.md` 第 45 行"Every img has meaningful alt text"——"meaningful"没有定义。**改进做法**：补充判断标准，例如"alt 文本不为空字符串，不以 'image of' 或 'photo of' 开头，不等于文件名"。**预期收益**：将定性检查变为可执行的验证规则。
4. **当前问题**：Gemini 扩展和 Codex 扩展已添加，但 README.md 和 CLAUDE.md 尚未更新多运行时安装说明。**改进做法**：在 CLAUDE.md 顶部新增「多运行时支持」节，说明三种安装路径（Claude Code / Codex / Gemini）。**预期收益**：降低新用户的认知负担，清单文件与文档保持同步。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **97/100**（7 个 NL 制品，0 个 bug，9 个质量问题，全为低严重性）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 100 | 无问题，满分 |
| skills/core-web-vitals/SKILL.md | 100 | 无问题，满分 |
| skills/performance/SKILL.md | 100 | 无问题，满分 |
| skills/accessibility/SKILL.md | 98 | L10："comprehensive"——大纲声明缺乏枚举 |
| skills/best-practices/SKILL.md | 96 | L10："modern"（无年份），L567："appropriate"（无阈值） |
| skills/web-quality-audit/SKILL.md | 96 | L10："comprehensive"，L45："meaningful"（标准未定义） |
| skills/seo/SKILL.md | 92 | L120："naturally"，L237："relevant"，L239："reasonable"，L470："important" |

**跨件警告**：`skills/core-web-vitals/references/LCP.md` 存在但未被 SKILL.md 引用（孤文件）。  
**安全**：CLEAR——`scripts/analyze.sh` 为只读脚本，0 个安全发现。

### 3.2 当时值得借鉴的模式

1. **数值阈值直接编码（numeric-threshold encoding）** → 为什么好：CLAUDE.md 在顶部声明 `LCP ≤ 2.5s`、`INP ≤ 200ms`、`CLS ≤ 0.1`、`Total < 1.5 MB`，Claude 审查时有明确的判断门控，不依赖模型的主观感受 → 原文路径：`CLAUDE.md` 第 1-20 行 → 如何借鉴：在自己的 CLAUDE.md 里为每个质量维度都写一行「阈值声明」，哪怕只有 3-5 个，也比全靠形容词可靠。

2. **单维度技能分离（single-dimension skill isolation）** → 为什么好：性能、SEO、无障碍各成一个文件，Claude 在处理特定问题时只加载相关技能，不被其他维度干扰 → 原文路径：`skills/` 目录下 6 个技能文件各对应一个维度 → 如何借鉴：不要把所有检查规则堆在一个 SKILL.md 里，按维度分文件，每个文件只聚焦一个关注点。

3. **清单严格同步（manifest discipline）** → 为什么好：CLAUDE.md 技能表、plugin.json 路径、磁盘文件三者完全一致，NLPM 交叉件检查零漂移 → 原文路径：`.claude-plugin/plugin.json`、`CLAUDE.md` 技能列表 → 如何借鉴：每次新增或删除技能文件，同步更新 plugin.json 和 CLAUDE.md，把这两步写进 PR checklist。

4. **参考材料与 SKILL.md 分层存放（reference layering）** → 为什么好：把 LCP 的详细规范放在 `references/LCP.md`，SKILL.md 只保留核心规则，使 SKILL.md 精简可读 → 原文路径：`skills/core-web-vitals/references/` 目录 → 如何借鉴：当某个规则的背景解释超过 5 行时，把它移到 `references/` 子目录，SKILL.md 只留主干。（注意：本仓库这个模式有一个瑕疵——LCP.md 放进去了但忘记在 SKILL.md 里加引用链接，成为孤文件。）

### 3.3 当时的缺陷

1. **孤文件 `references/LCP.md`**：文件存在但未被任何 SKILL.md 引用。为什么这是问题：孤文件意味着这份参考材料对使用者不可见——Claude 加载技能时不会主动发现子目录里的未引用文件；用户也无法通过 SKILL.md 导航到它。**自查**：在自己的技能仓库里运行 `find skills/ -name "*.md" | while read f; do grep -rq "$f" . || echo "ORPHAN: $f"; done`，检查是否有孤立参考文件。

2. **`skills/seo/SKILL.md` 的 4 个模糊量词**：L237 "relevant internal pages"（无筛选标准）、L239 "reasonable number"（无上限数字）、L470 "important pages"（重要性未定义）、L120 "naturally"（无可测量指标）。为什么这会失败：模糊量词在 SEO 这个领域尤其危险，因为 SEO 有很多业界公认的具体数值——当这些数值存在时，用模糊词代替是主动放弃了精确性。**自查**：`grep -n "relevant\|reasonable\|important\|naturally" skills/seo/SKILL.md`。

3. **`SKILL.md` 大纲声明缺乏枚举**：多个技能文件的首行描述用 "comprehensive" 概括范围（如"Comprehensive accessibility guidelines"），但未列出"comprehensive 意味着覆盖哪些具体维度"。为什么会失败：声明范围不代表定义范围——用户无法通过这句话判断该技能是否覆盖了他关心的某个具体场景。**自查**：检查自己的技能 description 字段，把"comprehensive X"替换成"X 覆盖：[维度 1]、[维度 2]、[维度 3]"。

### 3.4 当时的优化机会

1. **修复孤文件（3 分钟）**：在 `skills/core-web-vitals/SKILL.md` 的 LCP 小节末尾加一行 `> 参考：[LCP 详细规范](references/LCP.md)`，立即消除孤文件警告。
2. **修复 SEO 模糊量词（30 分钟）**：逐一为 4 个模糊行补充数值，SEO 技能从 92 升至 98+，整体仓库从 97 升至 99+。
3. **统一 description 字段规范（10 分钟）**：把所有技能的 description 字段从"Comprehensive X"改为"X audit covering [A, B, C, D]"，消除剩余的范围声明问题。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 孤文件 `references/LCP.md` | 在 `skills/core-web-vitals/SKILL.md` 中搜索 "LCP.md" 引用 | **持续存在**：SKILL.md 仍未引用 LCP.md | 孤文件跨越 4 周仍未修复，说明作者未将其纳入维护优先级 |
| SEO 模糊量词 | `grep -n "relevant\|reasonable\|important\|naturally" skills/seo/SKILL.md` | **部分存在**：仍有 2 次 "comprehensive\|appropriate" 命中 | 从 4 个减少，但未完全清除 |
| description 大纲声明 | `grep -n "^Comprehensive" skills/*/SKILL.md` | **持续存在**：多个技能文件 description 字段保持原样 | 低优先级问题未被触及 |

### 4.2 架构演进

**新增运行时扩展**是当前 HEAD 与 audit 时最大的变化。作者新增了 `codex/.codex-plugin/plugin.json` 和 `gemini-extension.json`，表明仓库正在向「多运行时通用质量框架」演进——同一套技能，在 Claude Code、Codex 和 Gemini 三个环境中分别提供入口。

这是一个战略性扩张，而非质量修复。6 个核心技能文件本身没有变动，原有的 9 个质量问题（含孤文件）依然存在。

**作者的优先级判断**：横向扩张（覆盖更多运行时）> 纵向打磨（修复剩余的模糊词和孤文件）。这在商业意义上是合理的——97/100 的高分意味着「足够好」，修复到 99/100 的边际收益远小于接入新运行时生态的增量用户。

### 4.3 新增的可学习模式

**「多运行时单一技能源」（Multi-Runtime Single Source of Skills）**：同一套技能文件（`skills/`）通过不同的清单文件分别适配 Claude Code（`.claude-plugin/plugin.json`）、Codex（`codex/.codex-plugin/plugin.json`）和 Gemini（`gemini-extension.json`）。这意味着维护者只需维护一份技能内容，多个生态的用户都能使用。

这个模式的前提是：技能内容与运行时无关——只要技能描述的是领域知识（Web 质量标准），而非平台特定操作（如 Claude Code 的钩子机制），它就可以跨运行时复用。

---

## 五、校准

### 5.1 我已经在做对的

1. **在顶层文件声明全局阈值**：我的 CLAUDE.md 里为关键质量指标提供了数值门控，而不是全靠形容词，和本仓库的 100/100 CLAUDE.md 策略一致。
2. **按维度拆分技能文件**：我没有把所有检查逻辑塞进一个文件，而是按关注点分文件，和本仓库的 6 个独立技能文件策略一致。
3. **清单与磁盘同步**：我在新增技能文件时同步更新 plugin.json，没有出现路径漂移问题。
4. **避免 "comprehensive" 作为 description 的主词**：本仓库因为这个词在多个 description 字段拿到了 -2 惩罚，我的技能文件避免了这种开头。

### 5.2 挑战 / 验证

- **这次案例验证了「97/100 不等于没有问题」**：本仓库的高分容易让人误以为它无懈可击，但孤文件和 SEO 模糊词是真实存在的问题——只是严重性低、影响范围窄。高分不代表完美，只代表大多数判断标准都通过了。
- **认知挑战**：我之前认为「references/ 目录是好的设计」，但本仓库提醒我：放进去还不够，还需要从 SKILL.md 里显式引用，否则参考材料是隐形的。「存在即可发现」是一个危险的假设。
- **新认知**：「最后 3 分是最贵的」——从 97 到 100 需要修复 SEO 域内的模糊词（这些词在 SEO 领域本就难以精确化）和孤文件（需要找对应的 SKILL.md 段落加引用）。这些操作本身不难，但需要主动意识到它们的存在。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查 references/ 下是否有孤立文件（未被任何 .md 引用）
find . -path "*/references/*.md" | while read f; do
  basename_f=$(basename "$f")
  grep -rq "$basename_f" --include="*.md" . && echo "OK: $f" || echo "ORPHAN: $f"
done
# 命中后：找到对应 SKILL.md 的适当位置，加一行 > 参考：[标题](相对路径)

# 2. 检查 SKILL.md description 字段是否以 "comprehensive" 开头
grep -rn "^[Cc]omprehensive" skills/*/SKILL.md 2>/dev/null
# 命中后：改写为枚举形式 "X covering: [A], [B], [C]"

# 3. 检查 SEO 类技能中的常见模糊量词
grep -n "relevant\|reasonable\|important\|naturally\|meaningful\|appropriate\|modern" skills/seo/SKILL.md 2>/dev/null
# 命中后：逐行检查是否存在业界公认数值可替代模糊词

# 4. 检查 plugin.json 和 CLAUDE.md 技能表是否同步
python3 -c "
import json, re, sys
with open('.claude-plugin/plugin.json') as f:
    manifest_skills = {s['name'] for s in json.load(f).get('skills', [])}
with open('CLAUDE.md') as f:
    content = f.read()
    doc_skills = set(re.findall(r'skills/(\w[\w-]*)/SKILL\.md', content))
diff = manifest_skills.symmetric_difference(doc_skills)
print('MISMATCH:', diff) if diff else print('OK: manifest and CLAUDE.md in sync')
"
```

### 6.2 灵感 → 实施路径

1. **想法**：为自己的技能仓库建立「数值阈值注册表」
   - **为何可行**：本仓库 CLAUDE.md 的 100/100 核心就是把阈值集中声明，这是一个可以直接复制的模式
   - **第一步**：在 CLAUDE.md 顶部新增「质量门控阈值」表格，列出每个技能维度的可测量标准，例如「无障碍：WCAG AA（对比度 ≥ 4.5:1）」「性能：FCP ≤ 1.8s」——大约 20 分钟

2. **想法**：为每个 references/ 文件加「入口守卫」测试
   - **为何可行**：本仓库的孤文件问题是因为 LCP.md 写好了但没人在 SKILL.md 里加链接。一个自动检查脚本可以彻底消除这类遗漏
   - **第一步**：在项目根目录新建 `scripts/check-references.sh`，遍历 `references/` 目录下所有 `.md` 文件，检查是否被同级 `SKILL.md` 引用；把这个脚本加入 pre-commit 钩子——大约 15 分钟

3. **想法**：把「数值门控」扩展到创意类技能的可量化维度
   - **为何可行**：数值阈值在 Web 性能领域效果极好，但在微短剧（drama-workshop-skills）这样的创意领域，也有可量化的维度——例如「每集字数 500-800 字」「每条 hook 不超过 20 字」
   - **第一步**：审查 drama-workshop-skills 里的每个技能，找出 3 个可以从形容词改为数值的地方；估计 30 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

| 我的仓库 | 对应的本仓库模式 | 对齐度 |
|---|---|---|
| MarkQWu/drama-workshop-skills（微短剧质量框架） | 多维度技能集 + 数值门控 | 高对齐——同样是「质量框架」类插件，都有多维度技能分文件，但我的还未达到本仓库的数值化程度 |
| MarkQWu/claude-for-legal（法律工作流） | 领域专用技能集（单一领域深度） | 中对齐——法律领域有明确的条文规定，理论上比 Web 领域更适合数值化，但目前实现还依赖大量形容词 |
| MarkQWu/echo-sleuth-for-claude（会话历史分析） | 技能 + 代理混合 | 低对齐——echo-sleuth 有 agent 层，本仓库没有；但两者在「低模糊量词密度」上策略相似（echo-sleuth 只有 2 次命中，本仓库技能平均约 1.5 次） |

### 7.2 在我的项目里复现的同类问题

1. **echo-sleuth-for-claude 的孤文件风险**：该仓库有 `references/` 类目录，需要检查是否有类似 LCP.md 这样的未引用文件。本仓库的案例提醒我：写完参考文件后必须在 SKILL.md 里显式加链接，否则「存在即隐形」。

2. **drama-workshop-skills 的描述词问题**：微短剧质量维度里有「戏剧性冲突要足够强」这类表达，和本仓库 `skills/seo/SKILL.md` 的"reasonable number of links"是同一类问题——这类判断在业务上有意义，但 AI 执行时缺乏可测量的判断依据。

3. **claude-for-legal 的 "comprehensive" 风险**：法律工作流技能如果用"全面审查合同条款"这类描述，会触发和本仓库 L10 相同的惩罚。更好的写法：「审查合同的以下 8 个维度：当事人、标的物、违约条款……」

### 7.3 别人的更优方案

1. **数值阈值的 CLAUDE.md 集中声明**：本仓库 CLAUDE.md 拿到 100/100 的关键是把所有数值阈值（LCP/INP/CLS/Total）写在顶部，让整个插件有一个单一真相来源。我的 CLAUDE.md 目前分散在各技能文件里各自声明，不如集中在顶部一目了然。

2. **`manifest discipline`（清单纪律）**：本仓库 plugin.json 和 CLAUDE.md 技能表零漂移，这背后是作者每次修改都同步更新两个地方的习惯。我需要把这个同步检查加入自己的 PR checklist。

3. **单维度技能严格分离**：本仓库的 SEO 技能只讲 SEO，不掺杂性能建议；性能技能只讲性能，不重复 SEO 规则。这种维度纯粹性让每个技能文件在被单独引用时也能独立工作，而我的技能文件之间有一定的内容重叠。

### 7.4 反向：我的项目做得比他们好的地方

1. **echo-sleuth-for-claude 有 agent 层**：本仓库只有技能（6 个 SKILL.md），没有任何 agent 定义。echo-sleuth 有 4 个技能 + 5 个代理，支持更复杂的多步骤 workflow。在需要 agent 协作的场景下，echo-sleuth 的架构表达力比本仓库强。

2. **references/ 文件的引用完整性（echo-sleuth）**：echo-sleuth 的参考材料都在 SKILL.md 里有对应引用，没有孤文件。这正是本仓库唯一的跨件警告所在——我的仓库在这个细节上比本仓库做得更好。

3. **模板多样性（drama-workshop-skills）**：本仓库 6 个技能使用相似的平铺结构，适合专家查阅但对新手不友好。drama-workshop-skills 有分层模板（新手路径 vs 专家路径），降低了不同受众的认知负担——这是本仓库未提供的。

---

## 八、术语表

### <a name="manifest"></a>manifest（清单文件）
> 项目的「目录」——告诉运行时这个插件包含哪些组件及其路径。`plugin.json` 就是 Claude Code 插件的 manifest。本仓库的 manifest 纪律（manifest discipline）体现在 `plugin.json`、`CLAUDE.md` 技能表、磁盘文件三者完全一致，NLPM 交叉件检查零漂移。如果清单里漏写了某个技能路径，那个技能即使在磁盘上也不会被加载。

### <a name="orphan-file"></a>孤文件（orphan file）
> 存在于文件系统中但未被任何其他文件引用或链接的文件。本仓库的 `skills/core-web-vitals/references/LCP.md` 是典型孤文件——它包含有价值的 LCP 规范内容，但 `skills/core-web-vitals/SKILL.md` 没有引用它，导致用户和 Claude 都无法通过技能导航发现这份文档。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> 在指令文件中使用的、无法被机械验证的形容词或副词，如 "comprehensive"、"reasonable"、"important"、"naturally"。模糊量词是 NLPM 评分中最常见的扣分项（-2 分/处）。本仓库 97/100 的原因正是这 9 处低严重性的模糊量词——其中集中在 SEO 技能（4 处），造成该文件 92/100 的最低单分。

### <a name="numeric-threshold"></a>数值阈值（numeric threshold）
> 用具体数字代替形容词来定义质量标准，例如 `LCP ≤ 2.5s` 而非「加载速度足够快」。本仓库的核心质量优势来自于在 CLAUDE.md 顶部集中声明数值阈值，使 Claude 在审查时有客观的判断门控。NLPM 评分中，数值阈值可以直接避免模糊量词扣分，是从 90 分迈向满分的关键手段之一。

### <a name="multi-runtime"></a>多运行时（multi-runtime）
> 同一套技能内容通过不同的清单文件适配多个 AI 开发工具（Claude Code、Codex、Gemini 等）。本仓库在当前 HEAD 新增了 `codex/.codex-plugin/plugin.json` 和 `gemini-extension.json`，实现同一份 `skills/` 目录在三个运行时环境中分别可用。前提是技能内容与特定运行时无关——描述的是领域知识，而非平台操作。
