# kepano/obsidian-skills — 学习案例

**仓库**：https://github.com/kepano/obsidian-skills
**Stars**：26,283 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-07（基于当前 HEAD）
**主题标签**：examples-driven, manifest-discipline, single-purpose, template-design, zero-execution-surface

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

kepano/obsidian-skills 是 Obsidian 官方团队为 Claude Code 发布的技能插件。作者 Steph Ango 是 Obsidian 的创始人——拥有 5M+ 用户的个人知识管理（PKM）工具。这是典型的「自己吃自己的狗粮」（dogfooding）：工具的创造者亲自为该工具提供 Claude Code 原生集成。

仓库结构极度克制：6 个 SKILL.md 文件，1 个 plugin.json，1 个 marketplace.json，1 个 README.md。没有钩子（hooks），没有脚本，没有 MCP 配置，没有依赖清单。覆盖范围是 Obsidian 生态中最核心的 5 个领域：

| 技能 | 用途 |
|------|------|
| `skills/obsidian-cli/SKILL.md` | 通过 Obsidian CLI 与运行中的 Obsidian 实例交互 |
| `skills/obsidian-bases/SKILL.md` | Obsidian Bases 数据库视图功能 |
| `skills/json-canvas/SKILL.md` | JSON Canvas 开放格式白板规范 |
| `skills/obsidian-markdown/SKILL.md` | Obsidian 专有 Markdown 扩展（嵌入、标注、属性） |
| `skills/defuddle/SKILL.md` | defuddle CLI 工具（网页转 Markdown） |

每个技能按需附带 `references/` 子目录，存放精选参考文档：

```
skills/obsidian-bases/references/FUNCTIONS_REFERENCE.md
skills/json-canvas/references/EXAMPLES.md
skills/obsidian-markdown/references/EMBEDS.md
skills/obsidian-markdown/references/CALLOUTS.md
skills/obsidian-markdown/references/PROPERTIES.md
```

NLPM Audit（2026-04-26）打出 **100/100**，Security: **CLEAR**，零缺陷，零安全风险。这是所有已审查仓库中极少见的满分案例。

### 1.2 架构剖析

**零执行面（Zero Execution Surface）**

整个插件不包含任何可执行制品：无 hooks/、无 scripts/、无 MCP 配置文件、无 package.json。这意味着安全扫描器没有任何可分析的攻击面。这是一个深思熟虑的架构选择：以牺牲程序化能力为代价，换来完全的安全透明度。

**约定优于配置（Convention over Configuration）**

`plugin.json` 中没有显式声明技能路径：

```json
{
  "name": "obsidian",
  "version": "1.0.1",
  "description": "Create and edit Obsidian vault files...",
  "author": {"name": "Steph Ango", "url": "https://stephango.com/"},
  "repository": "github.com/kepano/obsidian-skills",
  "license": "MIT",
  "keywords": ["obsidian", "markdown", "bases", "canvas", "pkm", "notes"]
}
```

Claude Code 按目录约定自动发现 `skills/*/SKILL.md`。这使 manifest 保持最小化，添加新技能无需修改 manifest。

**技能描述的精确触发条件**

`obsidian-cli/SKILL.md` 的 frontmatter description 字段示范了理想写法：

```
Interact with Obsidian vaults using the Obsidian CLI to read, create, search,
and manage notes, tasks, properties, and more. Also supports plugin and theme
development with commands to reload plugins, run JavaScript, capture errors,
take screenshots, and inspect the DOM. Use when the user asks to interact with
their Obsidian vault, manage notes, search vault content, perform vault
operations from the command line, or develop and debug Obsidian plugins and
themes.
```

这段描述做到了三件事：（1）说明工具能做什么；（2）列举具体使用场景；（3）用「Use when...」明确触发条件。Claude 读到这段描述就知道何时加载这个技能。

**分层引用文档（Tiered Reference Docs）**

复杂技能（obsidian-markdown、obsidian-bases、json-canvas）配有 `references/` 子目录，存放该技能领域的权威参考。这些文件不是 SKILL.md 的简单重复，而是原始规范的提炼或直接引用——例如 `json-canvas/references/EXAMPLES.md` 中的 JSON Canvas 格式示例，`obsidian-markdown/references/CALLOUTS.md` 中 Obsidian 标注块的完整语法。

简单技能（obsidian-cli、defuddle）不带 references/，因为其内容本身已足够简洁，或外部文档（`obsidian help`）可实时获取。

### 1.3 设计思路 / 方法论

**最小插件，最大语义**

Steph Ango 的选择是：不试图把 Obsidian 的所有功能塞进一个 SKILL.md，而是按功能域垂直切分，每个技能只做一件事，做到极致清晰。

**官方即权威**

作为 Obsidian 的作者，他拥有最高信息权威。`references/` 中的文档不是第三方解读，而是第一手规范。这种「官方技能包」模式是其他社区维护技能包无法复制的优势。

**实用主义胜于完美主义**

defuddle 技能非常简短，仅告知安装命令，不包含故障排查。这是刻意的取舍：defuddle 是简单的 npm 全局工具，大多数场景下一条命令解决问题。过度文档化反而增加噪音。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式 | 描述 |
|------|------|
| **零执行面插件** | 仅由 SKILL.md 组成，无任何可执行制品，安全风险为零 |
| **功能域垂直切分** | 每个 SKILL.md 对应一个内聚的功能域，而非一个大而全的技能 |
| **分层引用文档** | 复杂技能配 references/ 子目录，简单技能不配，按需分层 |
| **精确触发描述** | frontmatter description 明确写出「Use when...」触发条件 |
| **约定驱动 manifest** | plugin.json 不声明技能路径，依赖目录约定自动发现 |

### 2.2 适用场景

**零执行面模式**适合：
- 工具 / 库的官方 Claude Code 集成
- 任何不需要自动化脚本的纯知识型插件
- 面向最终用户（而非开发者）的技能包

**功能域垂直切分**适合：
- 产品功能面广的工具（Obsidian 有 CLI、数据库、白板、Markdown 扩展等）
- 需要按需加载不同技能的场景（避免上下文膨胀）

**分层引用文档**适合：
- 有复杂语法规范的领域（如 JSON Canvas 格式、Markdown 扩展）
- 技能内容本身无法内联所有参考信息时

### 2.3 与其他架构对比

| 维度 | kepano/obsidian-skills | 典型社区插件 | 重型工具插件（如 claude-task-master） |
|------|----------------------|------------|--------------------------------------|
| 执行面 | 零 | 低（偶有脚本） | 高（hooks + scripts + MCP） |
| 安全风险 | 零 | 低-中 | 中-高（需逐项审查） |
| 技能粒度 | 细（按功能域） | 不均匀 | 粗（按工作流） |
| Manifest 复杂度 | 极简 | 中等 | 复杂 |
| 参考文档 | 按需分层 | 少见 | 内联于 SKILL.md |

### 2.4 改进空间

**defuddle 技能的故障排查缺失**

这是 2026-04-26 Audit 发现的唯一信息性问题，当前 HEAD 仍未修复：`skills/defuddle/SKILL.md` 告知了安装命令（`npm install -g defuddle`），但未处理静默失败场景（npm 未安装、权限错误、网络超时）。

建议补充：

```markdown
## 故障排查

**命令未找到**：运行 `npm install -g defuddle`。若 `npm` 本身未安装，请先安装 Node.js（https://nodejs.org）。
**权限错误**：使用 `sudo npm install -g defuddle` 或配置 npm 全局目录到用户空间。
```

**缺少 CHANGELOG / 版本说明**

plugin.json 版本为 1.0.1，但仓库无 CHANGELOG。版本号变更的含义对使用者不透明。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

| 制品 | 分数 | 主要发现 |
|------|------|---------|
| `.claude-plugin/plugin.json` | 100 | 无显著问题 |
| `skills/defuddle/SKILL.md` | 100 | 内容精简（信息性） |
| `skills/json-canvas/SKILL.md` | 100 | 无 |
| `skills/obsidian-bases/SKILL.md` | 100 | 无 |
| `skills/obsidian-cli/SKILL.md` | 100 | 无 |
| `skills/obsidian-markdown/SKILL.md` | 100 | 无 |
| **总分** | **100/100** | Security: CLEAR |

**跨组件引用检查**：全部通过。
- `json-canvas/SKILL.md` → `references/EXAMPLES.md` ✓
- `obsidian-bases/SKILL.md` → `references/FUNCTIONS_REFERENCE.md` ✓
- `obsidian-markdown/SKILL.md` → `references/EMBEDS.md`, `CALLOUTS.md`, `PROPERTIES.md` ✓

### 3.2 当时值得借鉴的模式

1. **极简 manifest**：plugin.json 仅含必填字段（name、version、description、author、repository、license、keywords），valid semver，无冗余字段。
2. **references/ 分层架构**：复杂技能的引用文档独立存放，而非内联膨胀 SKILL.md。
3. **精准触发描述**：每个技能的 description 都明确了使用场景和触发条件。
4. **零执行面**：无 hooks、无脚本，安全扫描通过率 100%。
5. **单一职责**：每个 SKILL.md 只覆盖一个清晰的功能边界。

### 3.3 当时的缺陷

审查发现**零缺陷**（0 bugs，0 安全问题）。仅有 1 个信息性质量提示（0 扣分）：

`skills/defuddle/SKILL.md` 内容极简，缺少 CLI 依赖的故障排查章节。具体场景：若用户环境中 `npm` 未安装，`npm install -g defuddle` 会静默失败或输出难以理解的错误，SKILL.md 对此无任何引导。

这不是严重缺陷，而是一个「可以更好」的机会。对于一个满分仓库，这已是最边缘的改进空间。

### 3.4 当时的优化机会

- 为 defuddle 技能补充故障排查章节（低优先级）
- 添加 CHANGELOG 以追踪版本演进语义（可选）
- 考虑 plugin.json 中是否需要添加 `minClaudeCodeVersion` 字段（前瞻性兼容）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 发现 | 2026-04-26 状态 | 2026-06-07 状态 | 变化 |
|------|----------------|----------------|------|
| defuddle 缺少故障排查 | 信息性提示（0 扣分） | **仍存在**，未修复 | 无变化 |

由于原始 Audit 零缺陷，无任何「已修复」项可追踪。所有 6 个 SKILL.md 文件和 plugin.json 的结构自 Audit 以来保持不变。

### 4.2 架构演进

当前 HEAD 与 2026-04-26 快照在目录结构、manifest 格式、技能数量上完全一致。这是一个刻意保持稳定的插件——功能已完整，无需频繁迭代。

版本号仍为 1.0.1。

### 4.3 新增的可学习模式

本次审查（HEAD 对比）未发现新增模式。仓库维持了原有的高质量基线，没有引入退化，也没有显著的新增内容。

这本身就是一个可学习的模式：**成熟的插件不需要频繁变动**。一旦覆盖了目标功能域，保持稳定比持续扩展更有价值。

---

## 五、校准

### 5.1 我已经在做对的

**references/ 目录结构**：`MarkQWu/drama-workshop-skills` 已采用 `references/` 子目录存放扩展参考（如 `overseas/`、`three-layer-control.md` 等），与 obsidian-skills 的 references/ 模式高度一致。这是一个已验证的正确选择，继续保持。

**精准触发描述（中文版）**：drama-workshop 中的中文 description 写法（「当用户说'写短剧'、'短剧剧本'...」）与 kepano 的「Use when the user asks to...」在语义结构上等价——都明确列出了触发条件而非泛泛描述功能。这是正确的做法。

**单一职责切分**：echo-sleuth-for-claude 的 4 个 SKILL.md 按功能切分，每个只覆盖一个明确的职责范围，与 obsidian-skills 的垂直切分策略一致。

### 5.2 挑战 / 验证

**零执行面的取舍**：obsidian-skills 的零执行面是因为它不需要脚本。我的 claude-for-legal 有 connectors 和 managed-agent-cookbooks，echo-sleuth 的会话挖掘需要一定的程序化能力。不能生搬零执行面原则，但可以向它学习：**每一个执行面都应该有充分的必要性理由**。

**defuddle 教训**：即使是极简技能，也应思考「如果依赖未安装会发生什么」。这对 echo-sleuth（会话挖掘依赖特定 Claude Code 功能）和 claude-for-legal（可能依赖外部 API 或工具）同样适用。

**manifest 极简化**：检查我的 plugin.json 是否包含不必要字段，是否遵循 valid semver，keywords 是否精准。

---

## 六、行动

### 6.1 自查动作

**检查所有技能的 description 触发条件是否明确**：

```bash
# 提取所有 SKILL.md 的 description 字段，检查是否包含触发条件关键词
grep -r "^description:" \
  ~/repos/drama-workshop-skills/skills/ \
  ~/repos/claude-for-legal/skills/ \
  ~/repos/echo-sleuth-for-claude/skills/ \
  2>/dev/null
```

预期：每个 description 都应包含「当用户...」或「Use when...」这样的触发条件说明。

**检查哪些技能缺少故障排查章节**：

```bash
# 找出缺少故障排查章节的 SKILL.md 文件
for f in $(find ~/repos/echo-sleuth-for-claude/skills/ ~/repos/drama-workshop-skills/skills/ -name "SKILL.md"); do
  if ! grep -qiE "## (Troubleshoot|故障排查|常见问题|错误处理)" "$f"; then
    echo "缺少故障排查: $f"
  fi
done
```

**验证 plugin.json 的 semver 合规性**：

```bash
# 检查版本号格式
for f in $(find ~/repos -name "plugin.json" -path "*/.claude-plugin/*"); do
  echo -n "$f: "
  python3 -c "
import json, re, sys
d = json.load(open('$f'))
v = d.get('version','')
ok = bool(re.match(r'^\d+\.\d+\.\d+', v))
print(v, '✓' if ok else '✗ 非 semver')
"
done
```

**检查跨文件引用是否都能解析**：

```bash
# 找出 SKILL.md 中引用了 references/ 文件但该文件不存在的情况
for skill_dir in $(find ~/repos -name "SKILL.md" | xargs -I{} dirname {}); do
  refs=$(grep -oP '(?<=references/)[^\s\)]+' "$skill_dir/SKILL.md" 2>/dev/null)
  for ref in $refs; do
    if [ ! -f "$skill_dir/references/$ref" ]; then
      echo "断链: $skill_dir/SKILL.md → references/$ref"
    fi
  done
done
```

### 6.2 灵感 → 实施路径

**灵感 1：为 echo-sleuth 的每个技能添加 references/ 目录**

echo-sleuth-for-claude 的 4 个 SKILL.md 当前没有 references/ 子目录。对于会话挖掘领域，可以添加：
- `skills/session-mining/references/QUERY_PATTERNS.md`（常用查询模式）
- `skills/analysis/references/METRICS.md`（可分析的指标定义）

实施路径：
1. 识别每个技能中内联的「参考型」内容（语法、示例、枚举值）
2. 将这些内容提取到 `references/` 子目录
3. 在 SKILL.md 中用「See `references/XXX.md`」替换内联内容
4. 验证引用路径可解析

**灵感 2：为有外部依赖的技能补充故障排查**

对 claude-for-legal 中依赖外部 API 或工具的技能，补充标准故障排查章节：

```markdown
## 故障排查

**API 连接失败**：检查环境变量 `XXX_API_KEY` 是否已设置。
**超时**：默认超时 30s，可通过 `--timeout` 调整。
**权限错误**：确认 API Key 有对应资源的读写权限。
```

**灵感 3：obsidian-cli 的 description 写法作为模板**

将 obsidian-cli 的 description 结构作为模板，改写现有技能中泛泛而谈的描述：

原结构：`<工具名> + <能做什么> + Use when <触发条件列表>`

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

| 我的仓库 | 主要目的 | 与 obsidian-skills 目的相似度 | 可借鉴的架构模式 |
|---------|---------|------------------------------|----------------|
| MarkQWu/drama-workshop-skills | 中文戏剧剧本创作辅助 | 低（领域无关） | references/ 分层、垂直切分、触发描述 |
| MarkQWu/claude-for-legal | 法律工作流辅助 | 低（领域无关） | 极简 manifest、单一职责切分 |
| MarkQWu/echo-sleuth-for-claude | Claude Code 会话挖掘 | 低（领域无关） | 零执行面思路（尽量减少脚本依赖） |

三个仓库均与 Obsidian PKM 领域无交集，目的对齐度整体偏低。但**架构模式**的可借鉴性不依赖于领域相似度。

### 8.2 在我的项目里复现的同类问题

**echo-sleuth-for-claude：缺少故障排查章节**

4 个 SKILL.md 文件均未包含故障排查章节（grep 确认）。会话挖掘依赖 Claude Code 内部功能，若环境不满足（版本过低、功能未启用）会静默失败，与 defuddle 问题高度类似。

**drama-workshop-skills 和 echo-sleuth：缺少 references/ 结构**

echo-sleuth 的技能内容中可能内联了应当提取为 references/ 的信息（如查询语法、数据格式）。drama-workshop 已有 references/ 但尚未系统化。

**所有三个仓库：description 触发条件不统一**

部分技能的 description 描述了「能做什么」，但未明确「什么时候用」。kepano 的「Use when...」写法使 Claude 能更精确地决定何时激活技能，减少误激活。

### 8.3 别人的更优方案

**obsidian-cli/SKILL.md 的 description 写法优于我的多数技能**

kepano 的 description 同时覆盖了功能描述和触发条件，一段话解决两个问题。我的部分技能 description 只有功能描述，缺少触发条件，导致 Claude 需要读完整个 SKILL.md 才能判断是否适用。

**跨文件引用的清晰性**

obsidian-bases、obsidian-markdown 的 references/ 结构让 SKILL.md 保持简洁，细节下沉到引用文件。drama-workshop 的 references/ 结构相似，但 SKILL.md 本身是否充分利用了这些引用文件（通过明确的「See references/...」链接）值得检查。

**plugin.json 的关键词选取**

kepano 的 keywords：`["obsidian", "markdown", "bases", "canvas", "pkm", "notes"]`——精准覆盖搜索场景，无冗余。我的仓库 plugin.json 的 keywords 需要用同样的标准审视。

### 8.4 反向：我的项目做得比他们好的地方

**drama-workshop 的触发条件写法（中文版）质量相当**

`drama-workshop-skills` 中文 description「当用户说'写短剧'、'短剧剧本'...」的写法与 kepano 的英文版本在精确度上不相上下——都明确了触发关键词。中文场景下这种写法更自然，是值得保留的优势。

**claude-for-legal 的领域深度**

claude-for-legal 覆盖 10 个法律子领域，有 connectors 和 managed-agent-cookbooks，在**领域深度**和**集成能力**上超越了 obsidian-skills 的纯技能定位。obsidian-skills 的克制是刻意选择，而 claude-for-legal 的丰富性是法律工作流复杂度的必要响应——两者各有其合理性，不能简单比较优劣。

**echo-sleuth 的技能边界清晰**

echo-sleuth-for-claude 的 4 个技能边界划分清晰，职责不重叠，这一点与 obsidian-skills 的单一职责原则一致，是已经做对的地方。

---

## 八、术语表

| 术语 | 释义 |
|------|------|
| **frontmatter** | Markdown 文件开头由 `---` 包裹的 YAML 元数据块。SKILL.md 中的 `name` 和 `description` 字段写在此处，Claude Code 据此识别和激活技能。 |
| **plugin manifest** | 插件的元数据声明文件，通常为 `plugin.json`，包含名称、版本、作者、仓库等必填字段。Claude Code 插件生态用它来标识和管理插件。 |
| **semver** | 语义版本号（Semantic Versioning），格式为 `MAJOR.MINOR.PATCH`（如 `1.0.1`）。MAJOR 表示不兼容变更，MINOR 表示向后兼容的新功能，PATCH 表示向后兼容的缺陷修复。 |
| **PKM** | 个人知识管理（Personal Knowledge Management）。Obsidian 是 PKM 工具的代表，帮助用户组织、连接和检索个人知识库。 |
| **wikilink** | 双括号链接语法 `[[Note Name]]`，Obsidian 的核心链接机制。通过文件名（而非路径）引用笔记，支持模糊解析。`obsidian-cli` 的 `file=<name>` 参数遵循相同的 wikilink 解析规则。 |
| **defuddle** | 一个将网页内容转换为干净 Markdown 的 CLI 工具，需通过 `npm install -g defuddle` 全局安装。kepano 为其提供了专用的 SKILL.md，但未包含安装失败的故障排查。 |
| **零执行面（Zero Execution Surface）** | 插件中不包含任何可执行制品（hooks、scripts、MCP 配置、package.json）的状态。安全扫描器没有可分析的代码路径，安全风险从根本上为零。这是一种以「放弃程序化能力」换取「绝对安全透明度」的架构策略。 |
| **约定优于配置（Convention over Configuration）** | 软件设计原则，指通过合理的默认约定减少显式配置。obsidian-skills 的 plugin.json 不声明技能路径，Claude Code 按 `skills/*/SKILL.md` 目录约定自动发现。 |
| **dogfooding** | 自己使用自己产品（eating your own dog food 的缩写俚语）。kepano 作为 Obsidian 创始人为 Obsidian 写 Claude Code 技能包，是 dogfooding 的典型案例。 |
| **跨组件引用（Cross-component Reference）** | SKILL.md 中通过相对路径引用 `references/` 子目录文件的做法。NLPM 审查会验证这些引用路径是否可解析，断链会导致扣分。 |
