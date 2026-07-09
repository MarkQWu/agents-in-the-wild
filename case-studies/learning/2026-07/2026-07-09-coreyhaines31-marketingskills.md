# coreyhaines31/marketingskills — 学习案例

**仓库**：https://github.com/coreyhaines31/marketingskills
**Stars**：0 | **来源**：upstream（exemplar_published，SECURITY CLEAR，status=tracked）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-09（基于当前 HEAD）
**主题标签**：`template-design`, `single-purpose`, `manifest-discipline`, `examples-driven`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
marketingskills 是一个面向 SaaS 产品营销的 Claude Code 技能集合，覆盖 SEO、广告投放、内容策略、转化优化（CRO）、定价策略等 35 个营销专项技能。每个技能是独立的营销领域专家，共同实现的效果是给任何 SaaS 产品配备一支完整的营销专家团队。

关键事实：
- 35 个技能，是本次 4 案例中 **数量最多**的技能集
- 所有 35 个技能都通过 `marketplace.json` 完整声明（100% 覆盖）
- 有一个统一模式：每个技能都先检查 `product-marketing-context`（产品营销上下文）——强制一致性
- 有 `tools/clis/` 目录包含 61 个 Node.js CLI 工具，是本次 4 案例中唯一有 Node.js 执行面的插件
- 当前 HEAD 技能数量增至 47 个（audit 时 35 个）

### 1.2 架构剖析
```
marketingskills/
├── marketplace.json            ← 技能清单（47 个技能，vs audit 时 35 个）
├── skills/                     ← 47 个技能目录
│   ├── ab-testing/SKILL.md     ← 示例：A/B 测试（无 Output Format）
│   ├── ai-seo/SKILL.md         ← AI SEO 优化（无 Output Format）
│   ├── ad-creative/SKILL.md    ← 广告创意（有 Output Format）
│   ├── analytics-tracking/     ← 分析追踪（有 Output Format）
│   └── ... (47 total)
├── tools/clis/                 ← 61 个 Node.js CLI 工具（营销 API 封装）
│   ├── zapier.js               ← 低安全风险：任意 webhook URL
│   └── ... (60 more)
└── validate-skills.sh          ← 本地技能验证脚本
```

- **文件类型分布**：47 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：完全平铺，47 个技能无调用关系；所有技能有相同的前置步骤（检查 product-marketing-context）
- **跨件契约**：`product-marketing-context/SKILL.md` 是所有技能的隐式依赖——每个技能开头都先调用它，确保 LLM 了解产品背景才开始工作

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「产品上下文优先」——任何营销技能执行前必须先了解产品是什么，避免给不相关的产品生成营销内容
- **解决什么问题**：通用营销 AI 不了解具体产品，生成的内容泛泛而谈——marketingskills 通过 product-marketing-context 前置注入产品知识，让所有营销建议都是产品专属的
- **Trade-off**：35+ 个独立平铺技能 vs. 统一入口路由 → 选择了平铺，因为营销任务之间没有数据流依赖，但用户需要记住技能名称
- **反映什么认知模型**：营销是一系列专项工作（不是流水线），每个专项工作由不同的"专家"负责，LLM 扮演的是配备了产品知识的营销专家角色

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「产品上下文注入 + 大规模技能平铺」模式**：有一个"上下文技能"（product-marketing-context）被所有其他技能在执行前隐式调用，确保所有输出都以产品知识为基础。

模式特征清单：
- 特征 1：1 个上下文技能 + N 个领域技能，N 个领域技能共享同一个上下文前置步骤
- 特征 2：技能数量可以无限扩展，不影响整体架构（47 个 vs 原来 35 个，结构相同）
- 特征 3：所有技能有一致的"相关技能"（Related Skills）交叉链接，形成知识网络
- 特征 4：version 字段在技能间一致（除版本更新批次不同的例外）
- 特征 5：有 `validate-skills.sh` 本地验证脚本，保证发布前的自动化质量门

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域专项知识工具集（营销/法律/医疗） | ✅ 高度适用 | 产品上下文模式在任何领域都适用 |
| 需要跨技能数据流的工作流 | ❌ 不适用 | 技能间无数据流，无法自动串联 |
| 技能数量快速增长（从 35 → 47） | ✅ 适用 | 平铺架构对添加新技能无感，只需新建目录 |
| 希望用户一键完成全营销流程 | ❌ 不适用 | 用户需要手动选择各营销技能 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 上下文注入 + 平铺（本仓库） | marketingskills | 扩展性好，每个技能独立可测试 | 技能多后难以发现，无法组合调用 |
| Orchestrator 路由 | simmer（Case 2） | 可组合，流程自动化 | 复杂度高，orchestrator 易膨胀 |
| 纯平铺（无上下文技能） | review-squad（Case 1） | 最简单 | 技能间无法共享上下文 |

### 2.4 改进空间
1. **当前问题**：23/47 技能缺 `## Output Format` 段落 **改进做法**：为每个营销技能加输出格式声明（如 "交付：Markdown 表格，含行动项和优先级列"） **预期收益**：技能行为可预测，用户知道期待什么格式的输出
2. **当前问题**：1.0.0 版技能未随主版本一起更新（customer-research、community-marketing、lead-magnets）**改进做法**：在每次批量更新时加 `--bump-all` 步骤 **预期收益**：版本号一致，易于追踪变更
3. **当前问题**：缺 Router 技能，用户面对 47 个技能难以选择 **改进做法**：加一个 `marketing-navigator/SKILL.md` 技能，先问用户当前营销挑战，再推荐 3 个最相关的技能 **预期收益**：降低技能发现成本

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-13 得分 96/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| 13 个技能 | 90 | 缺 Output Format 段落（-10） |
| skills/programmatic-seo/SKILL.md | 98 | "appropriate" 模糊量词（-2） |
| 21 个技能 | 100 | 无 |

### 3.2 当时值得借鉴的模式
1. **product-marketing-context 前置注入** → 所有 35 个技能统一前置产品上下文检查 → 上下文一致性保障 → 借鉴：我的技能集如果有全局前提，应提取为独立技能并让其他技能前置调用
2. **marketplace.json 100% 覆盖** → 35 个技能在 marketplace.json 中全部声明，无遗漏 → manifest 完整性 → 借鉴：每次新增技能后立即更新 manifest，不要事后补
3. **Related Skills 全网络** → 所有技能都有 `## Related Skills` 段落，形成可浏览的知识网络 → 技能可发现性 → 借鉴：大技能集合应建立交叉链接，让用户能"顺藤摸瓜"发现相关技能
4. **CLI 工具环境变量化** → 61 个 Node.js CLI 工具全部从 `process.env` 读取 API 密钥，无硬编码 → 安全实践 → 借鉴：凡是 API 密钥，一律用环境变量，绝不硬编码
5. **版本化技能（大多数在 1.1.0）** → 技能文件有版本号，可追踪变更 → 借鉴：为 SKILL.md 的 frontmatter 加版本字段

### 3.3 当时的缺陷
1. **13/35 技能缺 Output Format（质量问题）** → 为什么失败：作者优先写了技能内容（框架、流程、检查清单），但没有声明"产出物是什么格式"；有内容没有声明输出结构，LLM 会自由发挥格式 → 自查：我的 bureau 6/7 技能缺 Output Format；gstack 46/54 技能缺 Output Format
2. **"appropriate" 模糊量词（-2 分，轻微）** → "CTAs appropriate to intent" 没有说明什么是"appropriate"——对电商适合折扣 CTA，对 SaaS 适合 demo CTA，需要显式说明 → 自查：我的技能里有多少处"appropriate"
3. **3 个技能版本不一致（1.0.0 vs 1.1.0）** → 为什么失败：后加进去的技能忘记在首次合入时检查版本一致性 → 自查：我的 skills 版本是否一致

### 3.4 当时的优化机会
1. 为 13 个技能批量加 `## Output Format` 段落（各技能内容不同，需要分别定义）
2. programmatic-seo/SKILL.md L137 "CTAs appropriate to intent" → "CTAs matched to page conversion goal"
3. 3 个 1.0.0 版技能更新到 1.1.0

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 13/35 技能缺 Output Format | `grep -rL "## Output Format" skills/*/SKILL.md \| wc -l` → 23 个 | **恶化**：从 13 → 23 个（+10 个新增技能也没有 Output Format） | 新增 12 个技能中大部分仍沿用旧模式，缺陷持续扩大 |
| "appropriate" 模糊量词 | `grep -rn "appropriate" skills/programmatic-seo/SKILL.md` | 需现场核查（clone 已删除） | 未验证 |

### 4.2 架构演进
技能数量从 35 → 47，新增了 co-marketing、aso（应用商店优化）、ads 等 12 个技能。架构没有变化，但问题在于：**新加的 12 个技能也没有 Output Format 声明**，导致缺陷从 13 个扩散到 23 个。这说明作者没有把"加 Output Format"加入新技能的工作流检查清单。

### 4.3 新增的可学习模式
`validate-skills.sh` 脚本是 audit 时提到但未深入的部分。这个脚本用于本地验证技能格式，相当于 NL 技能的 lint 工具。如果把"缺 Output Format"的检查加入这个脚本，就能防止新技能上线时遗漏。

---

## 五、校准

### 5.1 我已经在做对的
1. **CLI 工具用环境变量读 API 密钥**：我的所有工具脚本都没有硬编码密钥
2. **技能 Related Skills 链接**：bureau 的技能有互相引用的依赖说明
3. **manifest 完整性**：我的 bureau 和 gstack 的 manifest 覆盖了所有技能
4. **产品上下文意识**：gstack 的 CLAUDE.md 有项目背景说明，类似 product-marketing-context 的功能

### 5.2 挑战 / 验证
这个案例**验证了**"加 Output Format 是高价值改进"——marketingskills 有 13 个技能因此扣分，而这 13 个技能内容上都很完整（有检查清单、有框架）。这告诉我：技能内容写得多不代表高分，**输出格式声明是独立的评分维度**，不能被内容的丰富性代替。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 SKILL.md 是否有 Output Format 声明
grep -rL "## Output Format\|## Output\|output format" \
  ~/projects/MarkQWu-bureau/skills/*/SKILL.md \
  ~/projects/MarkQWu-gstack/*/SKILL.md \
  ~/projects/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
```
命中后：为每个命中文件加 `## Output Format` 段落，说明这个技能返回什么格式（Markdown 列表？表格？JSON？几行短文？）

```bash
# 检查我的 skills 中有多少处 "appropriate" 模糊量词
grep -rn "\bappropriate\b" \
  ~/projects/MarkQWu-*/skills/*/SKILL.md \
  ~/projects/MarkQWu-*/*/SKILL.md
```
命中后：把 "appropriate" 替换为具体标准（如 "matched to the detected framework's convention"）

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 创建一个 `context/SKILL.md`，供所有其他技能前置调用，加载 bureau 的 canon 知识库
   - **为何可行**：marketingskills 的 product-marketing-context 模式证明了"上下文技能"可以让所有领域技能更精准；bureau 的 canon/ 目录里有积累的领域知识，技能应该先读这些
   - **第一步**：在 `bureau/skills/` 下新建 `context/SKILL.md`，写一段"读取 `canon/` 目录内容并注入上下文"的提示词，10 分钟
2. **想法**：给 gstack 加一个 `validate-skills.sh` 脚本，检查所有技能的 Output Format 声明
   - **为何可行**：marketingskills 有 `validate-skills.sh` 作为质量门；gstack 有 54 个技能，手动检查不现实
   - **第一步**：写一个简单脚本 `grep -rL "## Output Format" */SKILL.md && echo "所有技能均有输出格式声明" || echo "以上技能缺输出格式声明"`，5 分钟
3. **想法**：批量为 bureau 的 6 个技能加 Output Format
   - **为何可行**：bureau 的 6 个缺失技能（capture/compile/guide/recall/review/scribe）内容丰富，但缺输出格式声明，与 marketingskills 的同类问题完全一样
   - **第一步**：逐一打开每个 SKILL.md，在末尾加 `## Output Format` 段落（各 5 分钟，共 30 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：为 SaaS 产品提供涵盖全营销领域的专业技能集合

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 都是某个领域（营销 vs 戏剧）的专项技能集合，技能平铺 | drama-workshop 面向创意写作；marketingskills 面向商业营销 | 高 |
| MarkQWu/gstack | 中 | 都是大规模技能集合（47 vs 54 个） | gstack 是通用工作流工具；marketingskills 是垂直领域专家 | 高 |
| MarkQWu/bureau | 低 | 都有多个技能文件 | bureau 是知识管理流水线，不是垂直领域专家集合 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 技能缺 Output Format | `grep -rL "## Output Format" gstack/*/SKILL.md \| wc -l` | **gstack 46/54 命中；bureau 6/7 命中；echo-sleuth 3/4 命中** | 高 |
| 新增技能不加 Output Format | 新加技能的 PR 流程有无检查清单？ | **gstack 和 bureau 均无新技能检查清单** | 中 |
| 版本不一致 | `grep "^version:" */SKILL.md \| sort -u` | 需本地验证 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/skills/capture/SKILL.md` → 加 `## Output Format`：「返回确认消息，格式：`✓ 已捕获 [标题]（[时间戳]）。条目存入 [路径]。`」→ 10 分钟
- `MarkQWu/bureau/skills/recall/SKILL.md` → 加 `## Output Format`：「返回 Markdown 列表，每项格式：`- [标题]（[日期]）：[摘要一句话]`，最多 10 条」→ 10 分钟

### 8.3 别人的更优方案

1. **领域**：上下文技能前置模式
   - **本案例做法**：`product-marketing-context/SKILL.md` 被所有 47 个技能在执行前检查——确保每个技能输出都基于产品真实情况
   - **我的项目现状**：gstack 的各技能各自读 `CLAUDE.md`，但没有一个专门的"上下文加载技能"
   - **如何借鉴**：在 `gstack/` 下新建 `context/SKILL.md`，声明"读取 CLAUDE.md + ARCHITECTURE.md + CHANGELOG.md 并提供结构化项目上下文"；在 `gstack/CLAUDE.md` 里说明所有技能在执行前应先调用 `gstack:context`

2. **领域**：Related Skills 全网络链接
   - **本案例做法**：所有 47 个技能都有 `## Related Skills` 段落，指向 2-5 个相关技能，形成可导航的知识网络
   - **我的项目现状**：gstack 的技能之间无 Related Skills 链接，用户不知道从 `design-review` 之后应该调用 `ios-design-review` 还是 `qa`
   - **如何借鉴**：为 gstack 最常用的 10 个技能加 `## Related Skills` 段落，30 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：CLAUDE.md 有 build/run/test 命令说明
- **我的做法**：`gstack/CLAUDE.md` 顶部有完整的 `bun install` / `bun test` 等命令，bureau 有 `node press/index.js` 说明
- **本案例做法（弱在哪）**：marketingskills 的 README.md 和 marketplace.json 没有明确的安装命令；用户需要自己推断 `claude plugin install marketingskills@coreyhaines31`
- **意义**：我在 CLAUDE.md 中维护 build 命令是一个好习惯；未来若贡献 PR，给 marketingskills 加安装说明是低成本高价值的改进

---

## 八、术语表

### <a name="product-marketing-context"></a>product-marketing-context
> marketingskills 插件中的一个核心技能，在任何营销任务执行前被隐式调用。它的职责是让 AI 先了解"你的产品是什么、目标用户是谁、当前增长阶段在哪"，然后再做营销建议。没有这个上下文，AI 会给一个企业级 B2B 工具推荐消费级 TikTok 营销策略。

### <a name="CRO"></a>CRO（Conversion Rate Optimization，转化率优化）
> 提高网站/产品页面中用户完成目标动作（注册、购买、升级）比例的一系列方法。marketingskills 有 page-cro、form-cro、onboarding-cro、popup-cro、paywall-upgrade-cro 等多个 CRO 专项技能，覆盖用户旅程的不同阶段。

### <a name="marketplace.json"></a>marketplace.json
> Claude Code 插件的技能清单文件，类似 App Store 的应用信息页。它列出了插件包含的所有技能、每个技能的 name、description、trigger phrases（触发短语）等元数据。NLPM 会检查 marketplace.json 中声明的技能与磁盘上的文件是否完全匹配——缺少任何一个都是 BUG。

### <a name="Output-Format"></a>Output Format（输出格式声明）
> 在 SKILL.md 中 `## Output Format` 段落里声明的"这个技能会产出什么格式的内容"。例如：「返回 Markdown 表格，第一列行动项，第二列优先级（高/中/低），第三列预期完成时间」。没有这个声明，AI 每次自由决定格式，导致用户无法预期输出结构，也无法在下游技能中消费这个输出。

### <a name="模糊量词"></a>模糊量词（vague quantifier）
> 无法量化的描述词。marketingskills 中典型例子："CTAs appropriate to intent"——什么叫 appropriate？对不同产品、不同页面类型、不同用户阶段的"appropriate"完全不同，LLM 会自行推断。修复：改为"matched to the page's primary conversion goal（e.g., trial signup for SaaS free tiers, direct purchase for one-time tools）"。
