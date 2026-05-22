# taishi-i/awesome-japanese-nlp-resources — 学习案例

**仓库**：https://github.com/taishi-i/awesome-japanese-nlp-resources
**Stars**：962 | **来源**：本地 audit（NOT in upstream manifest）
**Audit 日期**：2026-05-17（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `cross-reference`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
这是一个双层仓库：**外层**是精心维护的"awesome list"——以 [JSON](#json) 格式收录 1200+ 个日本语 NLP（自然语言处理）相关资源（库、预训练模型、数据集等）；**内层**是一个 Claude Code 插件，将这份 JSON 数据集包装成 4 个技能，让用户可以用自然语言检索和研究这些资源。作者 taishi-i 是日本 NLP 领域的资源整理者，仓库有 962 颗星，说明该 awesome list 在日本 NLP 社区有相当影响力。

关键事实：
1. 核心价值在数据：`data/resources.json` 收录 1200+ 资源，是整个插件的"大脑"
2. 插件安装方式：`claude plugin install awesome-japanese-nlp-resources@taishi-i --scope project`
3. 4 个技能形成工作流链：搜索 → 发现新资源 → 研究问题 → 研究趋势
4. [plugin.json](#plugin-json) 满分（100/100），5 个文件整体得分 79/100
5. Security 状态：CLEAR（无 Critical/High 风险）

### 1.2 架构剖析
- **目录结构**：
```
plugins/awesome-japanese-nlp-resources/
  .claude-plugin/
    plugin.json               # 插件 manifest（满分 100/100）
  skills/
    search/
      SKILL.md                # 入口技能：关键词/自然语言检索资源
    find-new-resources/
      SKILL.md                # 从网络发现新资源
    research-issues/
      SKILL.md                # 深入研究某资源的问题
    research-trends/
      SKILL.md                # 追踪 NLP 领域趋势
  data/
    resources.json            # 1200+ 日本 NLP 资源数据库（核心）
  README.md
```
- **文件类型分布**：4 个 SKILL.md，1 个 plugin.json，1 个核心数据文件（resources.json）。0 个 command，0 个 agent，0 个 hook
- **编排关系**：4 个技能平列部署，逻辑上形成单向工作流。`search` 是逻辑入口（检索已有资源），`find-new-resources` 是发现层（搜索网络找新资源），`research-issues` 和 `research-trends` 是深入分析层。技能之间通过输出模板里的文字引用传递用户，而非代码级调用
- **跨件契约**：`find-new-resources`、`research-issues`、`research-trends` 三个技能在各自的输出模板里用文字提示用户「接下来可以运行 `/awesome-japanese-nlp-resources:search`」。反向不成立：`search` 技能的输出模板里没有指向其他三个技能的引用。`data/resources.json` 是所有技能的共享数据源，通过 `find "${HOME}/.claude/plugins"` 在运行时定位

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「把已有的结构化数据包装成 NL 检索层」。这不是让 Claude 从零生成知识，而是给一份已经精心整理好的 JSON 数据集加一层自然语言接口——Claude 的作用是解析用户的自然语言查询意图，然后在 resources.json 里找匹配
- **解决什么问题**：awesome list 的传统浏览方式（翻 README、Ctrl+F）在资源超过 1000 条后变得低效。用户想问「有没有适合零资源学习场景的语音识别数据集？」这类问题，搜索引擎和 GitHub 搜索都很难回答，但 Claude + JSON 结构化数据可以
- **做了什么 trade-off**：选择了 4 个独立技能而非 1 个全能技能。单一"大技能"更简单，但会让提示体积膨胀到 400 行以上（NLPM 的惩罚线），而且会让每次调用都加载所有指令——拆分后每个技能聚焦单一任务，符合单职责原则
- **反映什么认知模型**：作者把 Claude 视为「数据库的自然语言查询层」，而不是「知识生成者」。这是一个清醒的认知：把 Claude 用在它擅长的地方（理解自然语言、结构化输出），把数据准确性交给人工维护的 JSON 文件

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「Awesome 列表搜索层模式」（Curated-List NL Search Wrapper）**

核心特征：把一份已有的精心策划型 JSON 数据集包装成多个 Claude 技能，让用户可以用自然语言对这份数据进行检索、发现和分析，而不需要学习 JSON 格式或写代码查询。

模式特征清单（4 条）：
- 特征 1：**数据先于逻辑**——`data/resources.json` 是核心资产，技能文件是它的查询界面
- 特征 2：**分工式工作流**——4 个技能覆盖「已有资源检索 → 新资源发现 → 问题研究 → 趋势分析」不同阶段
- 特征 3：**双语输出模板**——所有技能的输出格式同时提供日文和英文版本，降低语言切换成本
- 特征 4：**Step 0 空输入守卫**——每个技能都有显式的空参数处理逻辑，避免 Claude 在无输入时乱猜

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| Awesome list / 策划型资源清单维护者 | ✅ 高度适用 | 已有结构化数据 + 需要 NL 检索，完美匹配 |
| 公司内部技术选型数据库 | ✅ 适用 | 把内部 JSON 数据库包装成技能，无需开发后端 |
| 单纯的 FAQ 或文档检索 | ✅ 适用 | 只需 1 个 search 技能，可以更精简 |
| 需要实时抓取外部网站的场景 | ⚠️ 有限适用 | find-new-resources 用 WebSearch 工具扩展，但 search 主要依赖本地 JSON |
| 数据频繁变动（每天更新）的场景 | ❌ 不适用 | JSON 文件更新需要手动维护，data/resources.json 是静态文件 |
| 需要写入/修改数据的双向操作 | ❌ 不适用 | 这个模式是只读查询层，写操作需要完全不同的架构 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Awesome 列表搜索层（本仓库） | taishi-i/awesome-japanese-nlp-resources | 零后端、数据完全可控、Claude 负责语义理解 | 数据量有上限（JSON 全量加载到上下文有成本） |
| 纯 MCP 服务器搜索 | 自建 MCP + 向量数据库 | 支持语义向量检索，适合数万条数据 | 需要服务器部署、运维成本高 |
| 单一全能技能 | 多数简单插件 | 实现简单，用户记一个命令 | 提示体积大，职责不清，超过 400 行后质量下降 |

### 2.4 改进空间
1. **当前问题**：4 个 SKILL.md 的 [frontmatter](#frontmatter) 里全部缺少 `name:` 字段，文件不自描述。**改进做法**：在每个 SKILL.md 的 frontmatter 里加一行 `name: <技能名>`（如 `name: search`）。**预期收益**：NLPM 质量分从 79/100 升到 89/100（4 × -25 惩罚各减少为 0）；文件看一眼 frontmatter 就知道叫什么，不需要看目录名
2. **当前问题**：`search` 技能是逻辑入口，但它的输出模板里没有指向其他 3 个技能的引用，用户找到资源后不知道下一步可以做什么。**改进做法**：在 search/SKILL.md 的 description 下方加一个「相关技能」scope note，例如「找到资源后：用 `research-issues` 深入调研，用 `find-new-resources` 发现更多，用 `research-trends` 追踪趋势」。**预期收益**：修复 R07 惩罚（-3），更重要的是改善用户工作流的发现性
3. **当前问题**：`find-new-resources` 的内嵌 Python 代码把去重 URL 写入 `/tmp/awesome_ja_nlp_existing_urls.txt`（硬编码路径）。**改进做法**：改用 Python 的 `tempfile.NamedTemporaryFile(delete=False)` 动态生成临时文件路径，并在脚本结束后删除。**预期收益**：消除 Medium 安全风险，在多用户共享系统上不会出现 URL 数据泄漏

---

## 三、过去审查发现（2026-05-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-17 当时得分 **79/100**，Security 状态：**CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/search/SKILL.md | 72 | 缺 name（-25）；无指向兄弟技能的跨引用（R07，-3） |
| skills/find-new-resources/SKILL.md | 75 | 缺 name（-25） |
| skills/research-issues/SKILL.md | 75 | 缺 name（-25） |
| skills/research-trends/SKILL.md | 75 | 缺 name（-25） |
| .claude-plugin/plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **双语输出模板**（template-design）→ 为什么好：日本语 NLP 的使用者横跨日文和英文社区，每次输出同时给出日文和英文版本，用户不需要切换工具处理语言。这是「知道自己的用户是谁」的表现 → 原文路径：所有 4 个 SKILL.md 的输出格式 section → 如何借鉴：面向双语用户群的技能，在输出模板里加「日文版 / 英文版」两个 block，而不是让 Claude 即兴翻译

2. **Step 0 空输入守卫**（single-purpose）→ 为什么好：每个技能在最开头都明确写「如果 `$ARGUMENTS` 为空，输出使用提示并停止」。这防止了 Claude 在没有输入时胡乱猜测用户意图。传统技能往往忽略空输入，导致 Claude 「自由发挥」 → 原文路径：4 个 SKILL.md 各自的 Step 0 → 如何借鉴：所有接受参数的技能，第一步都加空输入检测：若参数为空则输出规范化的使用提示（展示合法例子）并提前返回

3. **4 技能工作流链设计**（cross-reference）→ 为什么好：4 个技能不是孤立的，它们覆盖了「检索现有 → 发现新增 → 深入问题 → 追踪趋势」完整的研究工作流，每个环节完成后有下一步指引。这比一个什么都能干的大技能更易维护，比一堆孤立技能更有用 → 原文路径：find-new-resources、research-issues、research-trends 的 Step 6 输出模板 → 如何借鉴：设计技能组合时先画工作流图，确认每个技能的输入来自哪个上游、输出指向哪个下游，然后在输出模板里写明「下一步可以…」

4. **内嵌可执行代码示例**（examples-driven）→ 为什么好：`find-new-resources` 里直接内嵌了 Python 代码块，Claude 可以在运行时执行这段代码来做去重检查。这把「操作说明」和「实现代码」合并在一个文件里，用户和 Claude 都能理解意图 → 原文路径：find-new-resources/SKILL.md 第 124 行附近的 Python 代码块 → 如何借鉴：对需要具体操作的步骤，用 ```python 或 ```bash 代码块内嵌，而不是用自然语言描述「执行类似这样的操作」

### 3.3 当时的缺陷

1. **4 个 SKILL.md 全部缺少 `name:` frontmatter 字段**：所有技能文件的 [frontmatter](#frontmatter) 里只有 `description:`，没有 `name:`。为什么这样设计会留下隐患：Claude Code 在没有 `name:` 时用目录名作为技能名（`search`、`find-new-resources` 等），所以功能上不会出错，但文件不再自描述——把一个 SKILL.md 复制到另一个位置，它的"身份"就变了，因为名字是从目录路径推断的，而不是文件本身声明的。这违反了 NL 编程的「自文档化」原则。**自查**：我的 SKILL.md 文件 frontmatter 里有没有 `name:` 字段？

2. **search 技能是信息孤岛**：`search` 作为逻辑入口技能，只接收来自其他 3 个兄弟技能的引用，自己却不指向任何人。为什么会失败：用户第一次接触这个插件，大概率先用 `search`，找到资源后不知道还有 `research-issues` 可以深入研究。这个「发现断层」让整个 4 技能工作流的价值损失了一半——只有从非 search 入口进来的用户才会看到完整链条。**自查**：我的「入口技能」有没有在输出里告诉用户工作流的下一步是什么？

3. **`/tmp` 路径硬编码**：`find-new-resources/SKILL.md` 第 124 行的内嵌 Python 把临时数据写入 `/tmp/awesome_ja_nlp_existing_urls.txt`（固定文件名）。为什么这样设计有问题：固定文件名意味着多个会话并发运行时会互相覆盖数据；在多用户 Linux 系统上，`/tmp` 是全局可读的，URL 列表（即使看起来无害）会暴露给系统上的其他用户；而且这个文件不会自动清理，会在系统上永久残留。**自查**：我的技能里有没有向 `/tmp` 写固定文件名的代码？

### 3.4 当时的优化机会
1. **给 4 个 SKILL.md 各加一行 `name:`**（质量修复，4 文件单行改动，合计 5 分钟）——把 NLPM 分数从 79 推到 89
2. **在 search/SKILL.md 加 scope note 引用三个兄弟技能**（R07 修复，1 处单行修改）——关闭工作流发现断层
3. **把 `/tmp/awesome_ja_nlp_existing_urls.txt` 改为 `tempfile.NamedTemporaryFile()`**（Medium 安全修复）——消除跨会话数据残留风险

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（grep） | 现状 | 含义 |
|---|---|---|---|
| 4 个 SKILL.md 缺 `name:` | `grep -r "^name:" plugins/awesome-japanese-nlp-resources/skills/` 结果为空 | **持续存在**：4 个技能文件均无 `name:` 字段 | audit 后 5 天内作者未修复；这属于「知道但没做」类型的遗留问题 |
| search 无跨引用到兄弟技能 | `grep -n "find-new-resources\|research-issues\|research-trends" plugins/awesome-japanese-nlp-resources/skills/search/SKILL.md` 无命中 | **持续存在**：search/SKILL.md 仍无任何对兄弟技能的引用 | 工作流发现断层未修复 |
| `/tmp` 硬编码路径 | `grep -n "/tmp/" plugins/awesome-japanese-nlp-resources/skills/find-new-resources/SKILL.md` 第 124 行命中 | **持续存在**：仍使用 `/tmp/awesome_ja_nlp_existing_urls.txt` | 安全修复未跟进 |

### 4.2 架构演进
Audit 日期是 2026-05-17，当前生成日期是 2026-05-22，间隔仅 5 天。根据目录结构检查，没有发现新增或删除的文件。`data/resources.json` 依然是核心资产，4 个技能文件结构未变。这个时间窗口太短，预期没有结构性变化——5 天对于 NL 层的迭代来说几乎是瞬间。

**作者当下关注点的信号**：仓库的 962 颗星说明维护者主要关注「awesome list 数据质量」（资源是否最新、分类是否准确），Claude Code 插件是附加功能，NL 合规性改进（加 name: 字段、加 scope note）不是当前优先级。这是一个常见的「主业功能完善、NL 接口层欠打磨」模式。

### 4.3 新增的可学习模式
暂无——本次生成日期距 audit 仅 5 天，未观察到架构变化。如需追踪，建议在 2026-06 中旬重新检查（届时 `find-new-resources` 的 `/tmp` 修复若落地，将是一个「安全反馈推动代码改进」的正面案例）。

---

## 五、校准

### 5.1 我已经在做对的

1. **跨引用链条完整**：echo-sleuth-for-claude 里 commands 指向 agents，agents 指向 skills，skills 相互可通过 description 发现——工作流发现性比本案例的 search 孤岛做得好。本案例提醒我这条链要双向
2. **双语支持意识**：drama-workshop-skills 已经在技能输出格式里考虑了多语言情境，思路上与本案例的双语模板对齐；区别是本案例的双语更结构化（明确分日文/英文两个 block），我可以往这个方向靠拢
3. **单职责拆分意识**：我的技能组合也有类似的「一技能一任务」意识，避免把所有功能堆在一个 SKILL.md 里
4. **plugin.json 同步维护**：本案例的 plugin.json 满分 100，说明 manifest 维护做得好；我在新增技能时也会同步更新 plugin.json
5. **避免了 Critical 安全问题**：我的技能里没有 curl-bash 执行链，安全门槛比本案例的 Medium 风险还低

### 5.2 挑战 / 验证

**这次案例验证了一个我曾经犹豫的认知**：「Claude Code 依赖目录名推断 skill name，所以不写 `name:` 也能用」——这个描述技术上是正确的，但本案例说明它还是应该写。原因不是注册失败，而是「自文档化原则」：当你把一个 SKILL.md 文件复制到另一个位置（迁移、重组目录结构时），文件的「身份」会因目录名变化而变化，但如果 frontmatter 里有 `name:` 字段，身份就是固定的，跟目录在哪里无关。**结论**：`name:` 字段不是功能前提，而是文件自描述的合规要求，不能因为「不写也能跑」就省略。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 SKILL.md 是否有 name: 字段
find ~/.claude/plugins -name "SKILL.md" | while read f; do
  grep -q "^name:" "$f" || echo "MISSING name: $f"
done
# 命中后：在对应 SKILL.md 的 frontmatter 里加 name: <技能目录名>

# 检查我的「入口技能」是否指向下游工作流技能（出站跨引用）
# 以 echo-sleuth 为例（调整路径为你的实际插件路径）
find ~/.claude/plugins -name "SKILL.md" | while read f; do
  count=$(grep -c "SKILL\|skill\|:search\|:recall\|:mine" "$f" 2>/dev/null || echo 0)
  if [ "$count" -eq 0 ]; then
    echo "POSSIBLY ISOLATED (no cross-refs): $f"
  fi
done
# 命中后：确认这个技能是否是入口/中间/出口节点，入口和中间节点必须有 See also / 下一步指引

# 检查我的技能里是否有硬编码 /tmp 路径
grep -rn "/tmp/" ~/.claude/plugins/*/skills/*/SKILL.md 2>/dev/null
# 命中后：把 /tmp/固定文件名 改为 Python tempfile.NamedTemporaryFile() 或 Shell 的 mktemp
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth-for-claude 的 recall.md 技能加 Step 0 空输入守卫
   - **为何可行**：本案例展示了 Step 0 空输入检测的标准写法；echo-sleuth 的 recall 技能如果用户不给关键词，当前行为是让 Claude 自由发挥，容易产生不一致输出
   - **第一步**：在 recall/SKILL.md 开头加「Step 0：若 `$ARGUMENTS` 为空，输出使用示例（e.g. `/echo-sleuth:recall 技术决策`）并停止」，预计 10 分钟

2. **想法**：把 drama-workshop-skills 的输出模板改为更结构化的「中文版 / 英文版」双 block 格式
   - **为何可行**：drama-workshop 的用户可能跨中英文社区，本案例的双语模板是经过验证的成熟做法（已有 962 星的实际用户验证）
   - **第一步**：找到 drama-workshop-skills 里最常用的 1 个技能，把输出格式 section 改为「**中文剧本版**：…」和「**English Script Version**：…」两个子 block；测试效果后推广到其他技能，预计 20 分钟

3. **想法**：为 echo-sleuth-for-claude 绘制工作流图并检查技能间引用完整性
   - **为何可行**：本案例证明了「搜索入口技能不指向下游」是一个具体的用户体验损失（用户不知道后续工作流）；echo-sleuth 的 mine → recall → 分析 链条是否每个节点都有双向引用？
   - **第一步**：用 `grep -n "echo-sleuth\|:mine\|:recall" ~/.claude/plugins/echo-sleuth-for-claude/skills/*/SKILL.md` 检查当前引用密度，找出孤岛节点，补上 scope note，预计 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 taishi-i/awesome-japanese-nlp-resources 的核心目的**：把一份精心策划的领域资源 JSON 数据库包装成 Claude Code 多技能插件，让用户可以用自然语言检索和研究

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 都是「领域专家插件」，聚焦单一垂直领域，多个技能组合 | drama-workshop 有状态管理（剧本上下文）；本案例是无状态的纯查询层 | 高（双语模板、工作流链设计可直接借鉴） |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有多技能协同的工作流设计 | echo-sleuth 是「挖掘对话历史」（记忆型），本案例是「查询静态数据库」（检索型），数据流向相反 | 中（cross-reference 链条设计可借鉴） |
| MarkQWu/claude-for-legal | 低 | 都有多个领域技能 | claude-for-legal 是法律工作流辅助，没有静态 JSON 数据库作为核心资产 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 所有 SKILL.md 缺 `name:` 字段 | `grep -L "^name:" ~/.claude/plugins/*/skills/*/SKILL.md` | 需执行检查（echo-sleuth 预期有 name:；drama-workshop 预期有 name:）——若无命中则已规避 | 高（会降低 NLPM 分数 25 分/文件） |
| 入口技能无出站跨引用（search 孤岛） | `grep -c "skill\|:recall\|:mine" ~/.claude/plugins/echo-sleuth-for-claude/skills/recall/SKILL.md` | echo-sleuth 的 recall.md 引用了 lessons 相关描述，部分覆盖；需确认是否有标准 scope note 格式 | 中（影响用户工作流发现性） |
| 内嵌代码向 `/tmp` 写固定文件名 | `grep -rn "/tmp/" ~/.claude/plugins/*/skills/*/SKILL.md` | echo-sleuth 脚本可能有 /tmp 使用——需执行检查 | 中（多用户系统上 Medium 安全风险） |

**命中后的具体行动建议**：
- 若 drama-workshop-skills 命中「缺 name:」：在每个 SKILL.md 的 frontmatter 第一行加 `name: <技能目录名>`，共 N 个文件，每个 30 秒，合计 5 分钟
- 若 echo-sleuth 命中「/tmp 固定文件名」：找到具体行，改为 `import tempfile; tmp = tempfile.NamedTemporaryFile(delete=False, suffix='.txt')`，并在技能末尾加清理步骤 `os.unlink(tmp.name)`，预计 15 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：双语结构化输出模板
   - **本案例做法**：每个技能的输出格式 section 都明确分「日文输出块」和「英文输出块」，不是让 Claude 即兴翻译，而是预置两套格式化模板（原文件路径：4 个 SKILL.md 的「Output Format」section）
   - **我的项目现状**：drama-workshop-skills 有一定双语意识，但输出模板没有预置双语块——中英切换依赖 Claude 的即兴判断
   - **如何借鉴**：在 drama-workshop-skills 最核心的 1-2 个技能里，把输出格式 section 改为「**中文剧本**：[结构化模板]」+「**English Script**：[结构化模板]」双 block，git diff 大概 +20/-5 行

2. **领域**：Step 0 空输入守卫的标准化写法
   - **本案例做法**：每个技能第一步都是明确的「若 $ARGUMENTS 为空则输出使用示例并停止」，措辞统一，4 个技能如出一辙（原文件路径：4 个 SKILL.md 的 Step 0）
   - **我的项目现状**：echo-sleuth 的技能有空输入处理，但写法不统一——有的是 Claude 推断的隐式处理，不是明确的 Step 0 声明
   - **如何借鉴**：在 echo-sleuth 所有接受 $ARGUMENTS 的技能里，加标准 Step 0：「若 `$ARGUMENTS` 为空，输出「用法：`/echo-sleuth:<技能名> <关键词>`，示例：…」并停止」

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：跨引用链条的双向性
   - **我的做法**：echo-sleuth-for-claude 的技能链——mine（挖掘）→ recall（回顾）——两个方向都有引用（mine 提到 recall，recall 提到 mine），形成双向发现
   - **本案例做法（弱在哪）**：search 技能完全没有出站引用，其他 3 个技能只有单向引用（→ search），整个引用图是「三条单向箭头指向 search」，search 是悬挂节点
   - **意义**：若未来有人 audit echo-sleuth，双向跨引用是 R07 的正面证据；若考虑给 taishi-i/awesome-japanese-nlp-resources 提 PR，修复 search 孤岛是最容易被接受的单行改动（scope note 加一行，影响面极小）

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 叫什么、如何注册和调用。`---` 必须严格从行首（第 1 列）开始，中间不能有多余空格。
> **对比**：frontmatter 是「身份证」——告诉系统这个文件是谁；正文是「操作说明」——告诉 Claude 执行什么。

### <a name="plugin-json"></a>plugin.json
> Claude Code 插件的「清单文件」（[manifest](#manifest)），告诉系统这个插件包含哪些 commands、skills 和 agents。如果 plugin.json 里漏写了某个文件，那个文件即使在磁盘上也不会被加载——用户完全看不到它。

### <a name="manifest"></a>manifest
> 项目的「成员名单」。对于 Claude Code 插件来说，`plugin.json` 就是 manifest——里面列出所有组件的路径。只有在 manifest 里登记过的文件，才会被系统识别和加载。

### <a name="json"></a>JSON
> 一种结构化文本格式，用花括号 `{}` 和方括号 `[]` 组织数据，非常适合存储和交换列表型数据。`data/resources.json` 就是一个 JSON 文件，里面用统一格式存着 1200+ 个日本 NLP 资源的名称、链接、分类等信息。Claude 可以读取并理解 JSON 内容，这是本案例「数据驱动搜索层」模式的技术基础。

### <a name="awesome-list"></a>awesome list
> GitHub 上一类约定俗成的「精选资源清单」仓库，通常以 `awesome-` 开头命名。维护者人工筛选某个领域的高质量工具、库、教程等资源，以 Markdown 格式列出。因质量高、更新及时，这类仓库往往收获大量 Stars。`taishi-i/awesome-japanese-nlp-resources` 就是日本 NLP 领域的 awesome list，962 颗星说明它已是该领域的事实标准参考。

### <a name="scope-note"></a>scope note
> SKILL.md 文件里的一小段说明性文字，通常放在 description 下方，告诉用户「这个技能的边界在哪里」以及「相关技能有哪些」。NLPM 的 R07 规则要求有出站引用的技能要有 scope note，目的是让用户在用完这个技能后知道下一步可以去哪里。
> **对比**：description 说「这个技能做什么」，scope note 说「这个技能做到哪里停、接下来去哪里」。

### <a name="single-source-of-truth"></a>单一真相源（Single Source of Truth）
> 一个系统里某类信息只存在一处，其他所有地方都从这里读取，而不是各自维护一份拷贝。本案例中，`data/resources.json` 是所有资源数据的单一真相源——4 个技能都通过文件路径访问它，而不是在各自的 SKILL.md 里复制一份数据。好处：更新一处，全部生效；坏处：这个文件损坏，所有技能都失效。
