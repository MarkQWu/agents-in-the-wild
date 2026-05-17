# taishi-i/awesome-japanese-nlp-resources — 学习案例

**仓库**：https://github.com/taishi-i/awesome-japanese-nlp-resources
**Stars**：数千级别（注册表中无精确记录）| **来源**：本地 audit
**Audit 日期**：2026-05-17（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD）
**主题标签**：single-purpose, cross-reference, template-design, manifest-discipline

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是一个知名的日语 NLP 资源 awesome 列表，核心内容是 README 中对各类日语自然语言处理工具、数据集、预训练模型的系统性汇总。插件作为附加层叠加于内容仓库之上，让 Claude 能够以结构化方式检索和分析同一份数据。

插件位于 `plugins/awesome-japanese-nlp-resources/` 目录，包含以下文件：

- `plugin.json`：插件元数据清单
- `data/resources.json`：结构化机器可读资源数据库，backing 所有技能
- `skills/search/SKILL.md`：关键词检索资源
- `skills/find-new-resources/SKILL.md`：发现未收录的新资源
- `skills/research-issues/SKILL.md`：研究资源相关 issues
- `skills/research-trends/SKILL.md`：分析日语 NLP 技术趋势

当前 HEAD：`c3029cbaf68245df4db1251706fda9becd3fddf9`（与 audit 时 SHA 完全一致，仓库在 audit 后未发生任何变动）。

### 1.2 架构剖析

该插件体现了一种特殊的设计模式：**内容仓库上叠加插件**（plugin-on-top-of-content-repo）。插件不是独立的工具集，而是读取仓库自身的结构化数据（`data/resources.json`）来提供能力。工作流如下：

```
search（关键词检索）
    ↓
find-new-resources（发现新资源，引用 search）
    ↓
research-issues / research-trends（深度分析，相互引用）
```

四个技能文件均采用双语（日语 + 英语）输出模板，每个文件长度在 257–279 行之间，配有编号步骤和 Python 代码示例块。`find-new-resources`、`research-issues`、`research-trends` 三个技能在输出模板中互相交叉引用，形成有机的工作流网络。

### 1.3 设计思路 / 方法论

**单一数据源原则**：`data/resources.json` 作为规范数据源，README 和插件技能均从此派生，避免了人工维护两套列表的一致性问题。

**渐进式工作流**：技能之间形成明确的调用顺序——先检索已有资源，再发现未收录资源，最后做深度研究。这种设计降低了每个技能的职责范围，也使技能间引用关系自然形成。

**双语输出策略**：输出模板同时包含日语和英语，与仓库定位（日语 NLP）高度契合，降低了国际协作者的使用门槛。

**防空输入设计**：所有 4 个技能均设有 Step 0，专门处理参数为空的情况，将边界案例纳入技能主体而非依赖调用方处理。

---

## 二、过去审查发现（2026-05-17 历史快照）

### 2.1 当时质量评分（NLPM）

**整体评分：79/100**，安全状态：CLEAR（0 Critical，0 High，1 Medium，1 Low）

| 文件 | 分数 | 主要扣分原因 |
|---|---|---|
| `plugin.json` | 100/100 | 无问题 |
| `skills/find-new-resources/SKILL.md` | 75/100 | frontmatter 缺少 `name` 字段（-25） |
| `skills/research-issues/SKILL.md` | 75/100 | frontmatter 缺少 `name` 字段（-25） |
| `skills/research-trends/SKILL.md` | 75/100 | frontmatter 缺少 `name` 字段（-25） |
| `skills/search/SKILL.md` | 72/100 | frontmatter 缺少 `name` 字段（-25），无指向兄弟技能的交叉引用（-3） |

注：NLPM 流水线将本仓库标记为 `case_study_skipped`，原因为"thin narrative: score=79, security=CLEAR, merged=0, applied_separately=0, rejected=0, rule_adopted=False"。本案例为自学目的手动生成。

### 2.2 当时值得借鉴的模式

1. **输出格式：双语编号模板** — 所有 4 个技能均提供日英双语的分步骤输出格式，结构化程度高，不依赖模糊描述。

2. **空输入处理（Step 0）** — 每个技能的第一步专门处理参数为空的场景，这是防御性技能设计的典型实践，完全符合 R07 的要求。

3. **技能间交叉引用** — `find-new-resources`、`research-issues`、`research-trends` 三个技能均在输出模板中引用兄弟技能，形成工作流闭环，用户无需手动串联。

4. **Python 代码示例块** — 技能正文中嵌入可执行的 Python 示例，提供具体的实现参考，而非仅描述应做什么。

5. **体量克制** — 每个技能文件 257–279 行，远低于 400 行警戒线，信息密度适中。

6. **零模糊量词** — 任何技能文件中均未发现 R01 所列举的模糊量词（appropriate、comprehensive、efficient 等），措辞准确具体。

7. **plugin.json 满分** — 清单文件完整规范，元数据无遗漏，体现了对配置层的认真对待。

### 2.3 当时的缺陷

1. **全部 4 个 SKILL.md 缺少 `name` 字段**（每个 -25 分）：frontmatter 中缺少 `name:` 键，这是 NLPM 最重量级的单项扣分规则。四个文件统一犯同一错误，说明作者在初始创建时未参考 SKILL.md schema，或使用了不包含 `name` 字段的模板。

2. **`search/SKILL.md` 无对外交叉引用**（-3 分）：其他三个技能都在输出模板中引用了 `search`，但 `search` 本身没有任何指向兄弟技能的引用。作为工作流入口，`search` 应在适当位置告知用户下一步可使用哪些技能。

### 2.4 当时的优化机会

1. **统一修复 `name` 字段**：四个文件使用相同的单行修复——在 frontmatter 中各添加一行 `name: <skill-name>`。修复成本极低，收益（+25 分/文件）极高。

2. **为 `search` 添加对外引用**：在 `search/SKILL.md` 的输出模板末尾或 "See also" 区块中，添加指向 `find-new-resources`、`research-issues`、`research-trends` 的引用，使工作流引导完整。

3. **消除 `/tmp` 硬编码路径**：将 `find-new-resources/SKILL.md` 第 124 行的 `/tmp/awesome_ja_nlp_existing_urls.txt` 替换为基于项目根目录或 `$TMPDIR` 的相对路径，消除世界可读的安全风险。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

**HEAD SHA 与 audit SHA 完全相同（`c3029cbaf68245df4db1251706fda9becd3fddf9`）**，意味着仓库在 audit 完成后没有任何提交。所有缺陷均原样保留：

| 缺陷 | audit 时状态 | 当前 HEAD 状态 |
|---|---|---|
| 4 个 SKILL.md 缺少 `name` 字段 | 存在 | **仍未修复** |
| `search/SKILL.md` 无对外交叉引用 | 存在 | **仍未修复** |
| `/tmp` 硬编码路径（Medium 安全） | 存在 | **仍未修复** |
| `find ${HOME}/.claude/plugins` 模式（Low） | 存在 | **仍未修复** |

实地验证：
- `grep "^name:" skills/*/SKILL.md` → 全部无匹配
- `grep -n "find-new-resources\|research-issues\|research-trends\|See also" search/SKILL.md` → 无匹配
- `find-new-resources/SKILL.md` 第 124 行：`/tmp/awesome_ja_nlp_existing_urls.txt` 仍存在

### 3.2 架构演进

无架构演进。仓库内容在 audit 日期与 HEAD 之间未发生变化，这是一个静态快照，架构与 audit 时完全一致。

这一现象本身值得关注：即使是知名的 awesome 列表仓库，Claude Code 插件部分也可能长期保持静态，维护重心更多集中在 README 资源列表本身，而非插件质量。

### 3.3 新增的可学习模式

由于 HEAD 未变动，无新增模式可供分析。但已有的"内容仓库上叠加插件"这一架构模式本身值得深入借鉴：

**"插件即数据接口"模式** — `data/resources.json` 充当技能的数据后端，插件技能本质上是结构化查询和分析接口，而非硬编码知识。当仓库数据更新（新增资源条目）时，技能能力自动提升，无需修改技能文件本身。这是一种高度可维护的插件设计哲学。

---

## 四、校准

### 4.1 我已经在做对的

通过分析该仓库，以下实践已经正确，值得坚持：

1. **frontmatter `name` 字段从不遗漏** — 该仓库的核心教训是四个文件统一犯同一低级错误。创建 SKILL.md 时应始终从包含 `name` 字段的完整模板出发，而不是从空文件写起。

2. **为每个技能提供空输入处理** — Step 0 模式是防御性设计的基本功，避免了"用户忘记传参数时技能无声失败"的问题。

3. **技能间交叉引用** — 在多技能插件中，技能之间的工作流引导应是双向的或至少单向完整的，不应出现"只被引用、从不引用他人"的孤立技能。

4. **代码示例优于文字描述** — 内嵌 Python 代码块将抽象的"应该做什么"转换为"具体怎么做"，显著提升技能的可用性。

### 4.2 挑战 / 验证

1. **`name` 字段遗漏的系统性根因** — 四个文件同时缺少同一字段，高度暗示作者使用了一个不完整的内部模板。挑战：如何在团队/个人工作流中引入模板校验步骤？验证方式：在创建技能文件后立即运行 `grep "^name:" SKILL.md`，或使用 NLPM 的 pre-commit 检查。

2. **插件维护动力不足问题** — awesome 列表的核心价值在于资源汇总，插件是锦上添花。当核心内容活跃更新时，插件质量往往被忽视。挑战：如何设计激励机制让作者持续维护插件质量？验证方式：在 CI 中加入 `/nlpm:score` 或 `nlpm-check` 步骤，让低分成为可见的阻断信号。

3. **`/tmp` 路径安全性** — 硬编码 `/tmp` 看似无害（只是缓存 URL 列表），但在多用户环境下是信息泄露风险。挑战：Python 技能文件中的临时文件路径应如何规范化？验证方式：改用 `tempfile.mkstemp()` 并在使用后显式删除，或将缓存文件放在项目本地目录（如 `.claude/cache/`）。

---

## 五、行动

### 5.1 自查动作

在自己的插件项目中执行以下检查：

1. **frontmatter 完整性扫描**
   ```bash
   # 检查所有 SKILL.md 是否含 name 字段
   grep -rL "^name:" skills/*/SKILL.md
   # 无输出 = 全部合格；有输出 = 需要补充
   ```

2. **交叉引用对称性检查**
   ```bash
   # 列出所有技能名称，逐一检查是否被其他技能引用
   for skill in skills/*/; do
     name=$(basename "$skill")
     echo "=== $name 被以下技能引用 ==="
     grep -rl "$name" skills/ | grep -v "$skill"
   done
   ```

3. **硬编码路径扫描**
   ```bash
   # 查找 /tmp 或 /var/tmp 的硬编码使用
   grep -rn "/tmp\|/var/tmp" skills/
   ```

4. **模糊量词扫描**
   ```bash
   # 检查 R01 所列模糊词
   grep -rni "appropriate\|comprehensive\|efficient\|optimal\|various\|several" skills/
   ```

5. **体量控制**
   ```bash
   # 检查是否有超过 400 行的技能文件
   wc -l skills/*/SKILL.md | awk '$1 > 400 {print $0}'
   ```

### 5.2 灵感 → 实施路径

**灵感 1：内容仓库上叠加插件（plugin-on-top-of-content-repo）**

该模式适用于任何以结构化数据为核心的仓库（awesome 列表、资源目录、数据集索引）。

实施路径：
1. 将 README 中的列表数据抽取为 `data/resources.json`（或同类结构化文件），使数据机器可读
2. 创建 `skills/search/SKILL.md`，让 Claude 通过关键词检索 JSON
3. 按工作流层级依次创建分析型技能（发现 → 研究 → 趋势），并在技能间建立交叉引用
4. 在 `plugin.json` 中声明所有技能，确保 `name` 字段与目录名一致
5. 当 JSON 数据更新时，技能能力自动扩展，无需修改技能文件

**灵感 2：防御性边界处理（Step 0 模式）**

将空输入处理写入技能正文的第一步（Step 0），而非依赖调用方处理。

实施路径：
1. 在每个 SKILL.md 的 `## Steps` 区块第一个步骤标注 `Step 0: 参数验证`
2. 说明当参数为空时的默认行为（如：使用最近一次搜索结果、列出所有可用类别、提示用户补充输入）
3. 此步骤应在任何实质性操作之前执行，作为技能的前置保护

**灵感 3：双语输出模板设计**

当插件服务于特定语言社区（如日语 NLP 社区）时，双语输出同时覆盖本地用户和国际协作者。

实施路径：
1. 在 `## Output Format` 区块中分别定义日语段落和英语段落的格式
2. 对技术术语保留英语原文，对说明性文字提供目标语言翻译
3. 在 Step 描述中同样采用双语标注，降低初次使用门槛
