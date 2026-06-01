# nextlevelbuilder/ui-ux-pro-max-skill — 学习案例

| 字段 | 值 |
|------|-----|
| 仓库 | [nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) |
| Stars | 70,122 |
| 来源 | xiaolai upstream |
| 审查日期 | 2026-04-26 |
| 生成日期 | 2026-06-01 |
| 主题标签 | `manifest-discipline` · `vague-quantifier` · `security-gate` · `cross-reference` · `template-design` |

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`nextlevelbuilder/ui-ux-pro-max-skill` 是一个面向 Claude Code 的 UI/UX 设计领域技能插件，拥有 70,122 颗 Star。该仓库将 UI/UX 设计知识系统化地组织为一组相互协作的 [SKILL.md](#skill) 文件，覆盖品牌识别、设计系统、UI 样式、幻灯片、Banner 设计等多个细分域。

仓库当前版本（2026-06-01 HEAD）提供了 7 个 SKILL.md 文件，并附带一个 TypeScript CLI 工具（`cli/`）用于安装和初始化。其知识数据库以设计资产枚举方式组织：67 种样式、161 套调色板、57 组字体搭配、25 类图表，以及支持 15 个技术栈的 UI 组件建议（React、Next.js、Vue、Svelte、Astro、SwiftUI、React Native、Flutter、Tailwind、shadcn/ui、Nuxt、Jetpack Compose 等）。

整体定位是一个"为 Claude 赋予专业 UI/UX 设计决策能力"的知识库，目标用户是需要在开发过程中获得设计指导的工程师。

### 1.2 架构剖析

当前目录结构：

```
nextlevelbuilder/ui-ux-pro-max-skill/
├── .claude/
│   └── skills/
│       ├── banner-design/SKILL.md
│       ├── brand/SKILL.md
│       ├── design/SKILL.md
│       ├── design-system/SKILL.md
│       ├── slides/SKILL.md
│       ├── ui-styling/SKILL.md
│       └── ui-ux-pro-max/SKILL.md     ← 主路由技能
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── cli/                                ← TypeScript CLI 安装工具
├── src/
│   └── ui-ux-pro-max/                  ← 源数据目录
├── CLAUDE.md
└── skill.json
```

架构的核心机制是**"路由技能 + 领域子技能"分层**：顶层 `ui-ux-pro-max/SKILL.md` 作为调度路由（router），负责将用户请求分发到各领域专属技能（`brand`、`design`、`design-system`、`ui-styling`、`slides`、`banner-design`）。这是一种典型的"NL 路由层 + 原生脚本核心"混合模式。

CLI 工具（`cli/`）提供了独立于 Claude Code 插件机制的安装自动化路径，是该仓库在同类插件中较为少见的基础设施投入。

### 1.3 设计思路 / 方法论

该插件的方法论建立在三个核心原则之上：

1. **数据库驱动的设计知识**：将 UI/UX 决策知识编码为可枚举的资产库（67 种样式、161 套调色板等），而非模糊的设计原则描述。这使得 Claude 可以给出具体的、可追溯的设计建议，而不是泛泛而谈。

2. **领域路由分层**：顶层路由技能承担意图识别和分发职责，领域技能承担具体执行职责。路由层保持轻量，领域层保持专注。这种分层有效限制了单个 SKILL.md 文件的体积，避免"知识大杂烩"反模式。

3. **TypeScript CLI 安装自动化**：通过 `cli/` 提供程序化的安装和初始化流程，降低新用户的上手门槛，同时与 Python 脚本（`src/ui-ux-pro-max/`）协作完成图片获取、设计 Token 同步等数据维护任务。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名称 | 描述 |
|---------|------|
| **路由技能模式（Router Skill）** | 顶层技能只做意图分发，不包含领域知识本身；领域知识下沉到子技能文件 |
| **数据库枚举模式（Database Enumeration）** | 将设计资产显式枚举为可引用的集合（67 种样式、161 套调色板），取代模糊描述，使建议可验证 |
| **CLI 辅助安装模式（CLI-assisted Setup）** | TypeScript CLI 工具管理安装、初始化和数据同步，将插件交付与维护从手动脚本升级为程序化工具 |
| **多域并行技能模式（Multi-domain Parallel Skills）** | 多个领域（品牌、系统、样式、幻灯片等）各有独立 SKILL.md，彼此平行，通过路由技能聚合 |
| **外部脚本依赖模式（External Script Dependency）** | 技能引用 Python/Node.js 外部脚本完成数据密集型操作（图片获取、Token 同步），技能文件本身保持精简 |

### 2.2 适用场景

| 模式 | 适用场景 | 不适用场景 |
|------|---------|-----------|
| **路由技能模式** | 覆盖多个相关子域、用户请求意图多样、希望保持单一入口点的技能库 | 单一职责、无需分发的聚焦型技能（强行加路由只增加复杂度） |
| **数据库枚举模式** | 决策依据可枚举（设计风格、调色板、字体）、需要给出可验证建议的领域 | 知识本质上是程序性的（如"如何做"的步骤），枚举无法穷举的场景 |
| **CLI 辅助安装模式** | 插件有复杂初始化（数据下载、Token 生成）、目标用户具备 Node.js 环境 | 轻量插件（一两个 SKILL.md）或非技术受众（CLI 反而提高门槛） |
| **多域并行技能模式** | 需要对多个相关但独立的领域提供深度覆盖（如 UI/UX 的品牌、样式、系统各域） | 领域间高度耦合、无法独立加载的场景（拆分后使用者需同时加载全部技能） |
| **外部脚本依赖模式** | 数据量大、需要程序化生成或同步（设计 Token、远程资产）的场景 | 离线或沙箱环境、无法运行外部脚本的用户（依赖变成阻断点而非便利） |

### 2.3 与其他架构对比

| 维度 | nextlevelbuilder/ui-ux-pro-max-skill（路由 + 数据库） | obra/superpowers（纯技能网格） | 单文件全功能技能 |
|------|---------------------------------------------------|------------------------------|----------------|
| 知识组织方式 | 数据库枚举（可验证）+ 路由分发 | 方法论叙述 + 交叉引用 | 线性叙述，无结构 |
| 可扩展性 | **高**：新增领域只需新增 SKILL.md + 在路由中注册 | 高：新增技能无需改动现有技能 | 低：文件体积随内容增长失控 |
| 外部依赖风险 | **高**：多处依赖外部脚本和非捆绑技能 | 低：无外部脚本依赖 | 低：自给自足 |
| 建议可验证性 | **高**：数字枚举（67 种/161 套）可直接核查 | 中：方法论质量难以量化 | 低：全凭描述，无锚点 |
| 安装复杂度 | 高：需要 CLI 工具和 Python 脚本 | 低：纯 Markdown，无构建 | 低：单文件，直接加载 |
| [孤立引用](#orphan-reference)风险 | **高**：多处引用未捆绑的外部技能 | 低：引用技能均在同仓库内 | 无：无跨文件引用 |

### 2.4 改进空间

1. **消除孤立引用**：`design/SKILL.md` 引用 6 个外部技能（`frontend-design`、`ai-artist`、`ai-multimodal` 等），`banner-design/SKILL.md` 引用 4 个外部技能，这些技能均未捆绑在插件内。用户安装此插件后，这些引用在实际使用中会静默失效。修复路径：要么将被引用技能纳入插件（捆绑），要么明确标注"需另行安装"，要么移除引用。

2. **消除重复工作流**：`design/SKILL.md` 和 `banner-design/SKILL.md` 之间存在 Banner 设计工作流的重叠。两个技能各自描述 Banner 创建流程，导致维护者更新时需同步两处，也让用户困惑应使用哪个入口。应以 `banner-design/SKILL.md` 为唯一权威，`design/SKILL.md` 通过引用指向它。

3. **补全路由技能的工作流指导**：`brand/SKILL.md` 是纯路由，缺乏工作流指导和输出格式说明。路由技能至少应声明"当用户请求被路由到此技能时，Claude 应按照 X 步骤处理，最终以 Y 格式输出"，否则路由成功后的执行行为完全不确定。

4. **修复[清单](#manifest)数量不一致**：当前 `plugin.json` 与 `ui-ux-pro-max/SKILL.md` 之间在技术栈数量描述上存在历史性出入（本次 HEAD 已将描述扩展为"15 stacks"含完整列表），但仍应通过自动化测试持续验证清单与技能内容的一致性。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：85/100**
**安全等级：CLEAR**（无 Critical / High 安全发现，PR 贡献通道畅通）

按文件分解：

| 文件 | 得分 | 主要扣分原因 |
|------|------|-------------|
| `slides/SKILL.md` | 78 | 第 14 行残留模板占位符 `<args>$ARGUMENTS</args>`（[stale template artifact](#template-artifact)）；内容单薄 |
| `design/SKILL.md` | 82 | 引用 6 个未捆绑的外部技能（[孤立引用](#orphan-reference)） |
| `brand/SKILL.md` | 83 | 纯路由无工作流指导，缺少输出格式说明 |
| `ui-ux-pro-max/SKILL.md` | 86 | 缺少 `ckm:` [命名空间前缀](#namespace-prefix)；含过时声明"React Native only" |
| `design-system/SKILL.md` | 87 | 轻微的外部脚本依赖假设 |
| `ui-styling/SKILL.md` | 87 | 含[模糊量词](#vague-quantifier)"museum-quality" |
| `banner-design/SKILL.md` | 88 | 引用 4 个未捆绑的外部技能（孤立引用） |
| `CLAUDE.md` | 88 | 路径假设特定安装目录结构 |
| `plugin.json` | 90 | 数量声明与 SKILL.md 内容不一致（"67 styles, 15 stacks" vs "50+ styles, 10 stacks"） |

安全发现（均为 Medium / Low，未达到阻断阈值）：

| 严重级别 | 位置 | 描述 |
|---------|------|------|
| Medium | `sync-brand-to-tokens.cjs` 第 253 行 | `execSync` 拼接模板字符串，有命令注入隐患 |
| Medium | `shadcn_add.py` 第 101 行 | `npx shadcn@latest` 版本未固定，存在供应链投毒风险 |
| Medium | `cli/src/utils/github.ts` 第 72 行 | 下载 ZIP 文件时无校验和验证 |
| Low | `cli/package.json` | semver 依赖版本未固定 |
| Low | `requirements.txt` | 含无上界的 `>=` 版本约束 |

### 3.2 当时值得借鉴的模式

1. **数字化资产枚举**：以"67 种样式、161 套调色板、57 组字体搭配"等具体数字描述知识库规模，而非使用"丰富的"、"全面的"等[模糊量词](#vague-quantifier)。数字是可验证的锚点，也是向用户传达价值的有效方式。

2. **路由 + 子域分层**：顶层路由技能控制入口，领域子技能各司其职。这种架构在审查时即已成熟，7 个 SKILL.md 文件覆盖了 UI/UX 领域的主要决策维度，既不遗漏也不重叠（除 Banner 工作流外）。

3. **CLI 工具投入**：`cli/` 目录提供了 TypeScript 编写的安装工具，这在同类 Claude Code 插件中相当罕见。这表明作者将用户上手体验视为一等公民，而不仅仅是发布 Markdown 文件了事。

4. **安全 CLEAR，PR 通道畅通**：尽管存在若干 Medium / Low 级安全发现，但均未触发 Critical / High 阻断阈值，表明作者在外部脚本设计上保持了基本的安全意识边界。

### 3.3 当时的缺陷

**Bug（两处明确可重现）：**

1. `slides/SKILL.md` 第 14 行：`<args>$ARGUMENTS</args>` 是从模板复制后未替换的占位符。这段文本会被 Claude 原样读取，导致幻灯片生成技能在遇到此处时行为不确定——Claude 可能将其解读为一个变量引用、一个 XML 标签或纯文本噪声。

2. `plugin.json` 数量声明错误："67 styles, 15 stacks" vs SKILL.md 中的"50+ styles, 10 stacks"。这不仅是数字不一致，还揭示了[清单](#manifest)与实际内容之间的同步机制缺失——任何一方更新都需要手动记住同步另一方。

**质量问题（系统性）：**

- `design/SKILL.md` 和 `banner-design/SKILL.md` 引用的外部技能均未捆绑在插件内，构成系统性[孤立引用](#orphan-reference)问题。共 10 处外部引用（6+4），占全部跨技能引用的大多数。
- `brand/SKILL.md` 是路由存根（routing stub）而非完整技能：只声明"将请求路由到 X"，不描述路由后的工作流，也不定义输出格式。
- `ui-ux-pro-max/SKILL.md` 缺少 `ckm:` [命名空间前缀](#namespace-prefix)，与同插件其他 6 个技能的命名惯例不一致，破坏了插件的内部一致性。
- `ui-styling/SKILL.md` 中"museum-quality"属于无客观标准的[模糊量词](#vague-quantifier)，Claude 无法将其转化为可操作的设计判断。

### 3.4 当时的优化机会

1. **一行修复**：删除 `slides/SKILL.md` 第 14 行的 `<args>$ARGUMENTS</args>`，或替换为有意义的内容，消除最低分技能（78 分）的 Bug 扣分。

2. **清单自动验证**：在 CLI 工具或 CI 中添加一个断言：`plugin.json` 中的数量声明必须与各 SKILL.md 中的实际枚举数一致，使 `plugin.json` 数量不一致问题在发布前自动被捕获。

3. **孤立引用解决策略**：三选一 —— 将 `frontend-design`、`ai-artist`、`ai-multimodal` 等技能捆绑进插件；在引用处添加"需另行安装"注释；或移除引用，改为内联相关知识摘要。

4. **为 `brand/SKILL.md` 补充工作流**：即使只是两三句话描述"品牌风格被应用到的输出格式是什么"，也能将 83 分的薄弱技能拉升到与其他技能持平的水平。

5. **替换"museum-quality"为具体标准**：例如"配色方案须通过 WCAG AA 对比度标准，字体大小不小于 16px，留白比例不低于 40%"，这类标准 Claude 可以实际执行检查。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-26 快照 | 2026-06-01 当前 HEAD | 状态 |
|------|----------------|---------------------|------|
| `slides/SKILL.md` 第 14 行 `<args>$ARGUMENTS</args>` 残留占位符 | 存在 | **仍然存在** | 持续存在（未修复） |
| `ui-ux-pro-max/SKILL.md` 缺少 `ckm:` 命名空间前缀 | 存在 | **仍然存在**（`name: ui-ux-pro-max`，无前缀） | 持续存在（未修复） |
| `plugin.json` 数量声明与 SKILL.md 不一致 | 存在（"50+ styles, 10 stacks" vs "67, 15"） | **部分改善**：`plugin.json` 描述已扩展为"67 styles, 161 palettes, 57 font pairings, 25 charts, 15 stacks"并附完整列表；但"15 stacks"的准确性仍依赖手动维护 | 部分改善，根因未解决 |
| `design/SKILL.md` 孤立引用（6 处） | 存在 | 未直接验证（脚本未检查） | 待验证 |
| `banner-design/SKILL.md` 孤立引用（4 处） | 存在 | 未直接验证 | 待验证 |
| `brand/SKILL.md` 纯路由无工作流指导 | 存在 | 未直接验证 | 待验证 |
| Medium 安全：`execSync` 模板字符串 | 存在 | 未直接验证 | 待验证 |
| Medium 安全：`npx shadcn@latest` 未固定 | 存在 | 未直接验证 | 待验证 |
| Medium 安全：ZIP 下载无校验和 | 存在 | 未直接验证 | 待验证 |

**结论**：两处被明确验证的问题（`slides/SKILL.md` 占位符 Bug 和 `ckm:` 前缀缺失）均持续存在超过 36 天，表明这两处缺陷未进入作者的修复优先级。`plugin.json` 描述的扩展（新增调色板、字体配对、图表数量）表明作者在积极扩充知识库内容，但[清单](#manifest)同步仍是手动过程。

### 4.2 架构演进

从审查日期到当前 HEAD，可观察到的演进方向：

- **知识库内容扩充**：从"67 styles, 15 stacks"扩展为"67 styles, 161 palettes, 57 font pairings, 25 charts, 15 stacks"，新增了调色板、字体配对、图表三类资产枚举。这与"数据库驱动"的核心设计哲学一致——作者在持续向数据库添加记录。

- **核心架构未变**：7 个 SKILL.md 的路由分层结构没有变化，CLI 工具框架没有变化，外部脚本依赖模式没有变化。

- **两处已知 Bug 未修复**：`slides/SKILL.md` 占位符和 `ckm:` 前缀缺失是纯机械性修复（各需 1 行代码），但 36+ 天未被处理，暗示作者的维护注意力集中在内容扩充而非质量修复。

### 4.3 新增的可学习模式

- **渐进式枚举扩展**：`plugin.json` 从简单的"67 styles, 15 stacks"演进为包含四类资产的详细枚举，每次扩展都增加了可验证的具体性。这是一种可借鉴的知识库成熟化路径：从宏观分类开始，逐步细化到可枚举的具体维度。

- **清单作为展示面而非仅作为技术元数据**：当前 `plugin.json` 的描述兼顾了技术准确性（具体列出 12 个框架名称）和市场传播价值（具体数字让插件功能一目了然）。清单文件可以同时服务于安装工具的程序化解析和用户的决策参考。

---

## 五、校准

### 5.1 我已经在做对的

- **无孤立引用问题**：我的 `claude-for-legal` 和 `echo-sleuth-for-claude` 中所有跨技能引用均指向同一仓库内的捆绑文件，不存在"引用了未安装技能"的问题。这是 `ui-ux-pro-max-skill` 最严重的系统性质量缺陷，而我的项目天然回避了它。

- **无模板占位符残留**：我的技能文件中没有类似 `<args>$ARGUMENTS</args>` 的未替换模板占位符。这类 Bug 通常来自"复制模板后忘记修改"，但我的技能是从头编写的，因此不存在此风险。

- **无命名空间不一致问题**：我的仓库中没有采用 `ckm:` 式的命名空间前缀方案，因此也不存在"7 个技能中有 1 个忘记加前缀"的一致性漏洞。

### 5.2 挑战 / 验证

**反直觉发现：85 分的仓库中，最严重的 Bug（`<args>$ARGUMENTS</args>`）是一行删除就能修复的纯机械问题，但它从 2026-04-26 持续到 2026-06-01 未被处理。**

这个案例揭示了一个重要的维护动力学规律：**内容增长的激励远强于质量修复的激励**。作者在同期将知识库从"67 styles, 15 stacks"扩展到包含 161 套调色板的详细枚举（这需要大量内容工作），却没有投入 30 秒删除一行占位符文本。

校准意义：NLPM 质量评分中的高扣分项（如 slides/SKILL.md 的 78 分）不总是意味着作者不知道问题存在。有时它意味着"作者认为内容价值比规范性修复更值得投资"。在评估一个仓库时，应区分"不知道的问题"和"知道但推迟的问题"——后者在高活跃度仓库中更常见，风险通常也更低（因为一旦有人提 PR，作者大概率会接受）。

---

## 六、行动

### 6.1 自查动作

对自己的仓库执行以下检查，确认无类似问题：

```bash
# 检查是否有未替换的模板占位符（<args>$ARGUMENTS</args> 或类似形式）
for repo in ~/echo-sleuth-for-claude ~/drama-workshop-skills ~/claude-for-legal; do
  echo "=== $repo : stale template artifacts ==="
  grep -rn '\$ARGUMENTS\|<args>\|{{.*}}\|__PLACEHOLDER__' "$repo" \
    --include="*.md" --include="*.json" 2>/dev/null || echo "  (none found)"
done

# 检查所有 SKILL.md 是否缺少 ## Output Format 章节
for repo in ~/echo-sleuth-for-claude ~/drama-workshop-skills ~/claude-for-legal; do
  echo "=== $repo : skills missing Output Format ==="
  find "$repo" -name "SKILL.md" | while read f; do
    grep -q "## Output Format\|## 输出格式" "$f" \
      || echo "  MISSING: $f"
  done
done

# 检查技能文件中的模糊量词（vague quantifiers）
for repo in ~/echo-sleuth-for-claude ~/drama-workshop-skills ~/claude-for-legal; do
  echo "=== $repo : vague quantifiers in SKILL.md files ==="
  grep -rn "\bproper\b\|\bappropriate\b\|\brelevant\b\|\bsuitable\b\|\bmuseum-quality\b\|\bhigh.quality\b" \
    "$repo" --include="SKILL.md" 2>/dev/null || echo "  (none found)"
done

# 检查 plugin.json 中的数量声明是否与 SKILL.md 技能数量一致
for repo in ~/echo-sleuth-for-claude ~/drama-workshop-skills ~/claude-for-legal; do
  echo "=== $repo : skill count in plugin.json vs actual SKILL.md files ==="
  actual=$(find "$repo" -name "SKILL.md" | wc -l)
  echo "  Actual SKILL.md count: $actual"
  grep -o '"skills":\s*\[[^]]*\]' "$repo/.claude-plugin/plugin.json" 2>/dev/null \
    | tr ',' '\n' | grep -c '"' \
    && echo "  (compare with above)" || echo "  (plugin.json not found or no skills key)"
done

# 检查跨技能引用是否均指向仓库内已存在的文件（孤立引用检查）
for repo in ~/echo-sleuth-for-claude ~/drama-workshop-skills ~/claude-for-legal; do
  echo "=== $repo : checking for orphan cross-references ==="
  find "$repo" -name "SKILL.md" | while read f; do
    grep -oP '(?<=skills/)[^\s"]+/SKILL\.md' "$f" 2>/dev/null | while read ref; do
      target="$repo/.claude/skills/$ref"
      [ -f "$target" ] || echo "  ORPHAN in $f: references $ref (not found at $target)"
    done
  done
done
```

### 6.2 灵感 → 实施路径

**灵感 1：数据库枚举模式应用到 `claude-for-legal`**

`ui-ux-pro-max-skill` 用"67 种样式、161 套调色板"取代了模糊的设计描述，这个思路可以直接迁移到法律工作流场景。

实施路径：
1. 梳理 `claude-for-legal` 中各领域技能，识别可枚举的知识维度（如"覆盖的合同类型：NDA、SaaS 服务协议、劳动合同、股权协议……"）
2. 在每个领域 SKILL.md 的开头添加"本技能覆盖范围"枚举表，替换当前的模糊描述
3. 同步更新 `plugin.json` 的 `description` 字段，用具体数字反映覆盖范围

**灵感 2：为 `echo-sleuth-for-claude` 补全 `## Output Format` 章节**

审查发现我的 `echo-sleuth-for-claude` 中有三个技能缺少输出格式章节，这与 `ui-ux-pro-max-skill` 中 `brand/SKILL.md` 缺少输出格式的问题性质相同。

实施路径：
1. 对 `jsonl-core/SKILL.md`、`memory-management/SKILL.md`、`git-mining/SKILL.md` 各添加 `## Output Format` 章节
2. 每个章节至少包含：输出的结构（列表/表格/JSON？）、关键字段、成功示例（3-5 行）
3. 在 `drama-workshop-skills/short-drama/SKILL.md` 执行同样的补全

**灵感 3：考虑为 `cli/` 式安装工具建立基础设施**

`ui-ux-pro-max-skill` 的 TypeScript CLI 工具处理了安装、初始化和数据同步，比 `install.sh` 脚本更结构化。当前我的仓库使用 shell 脚本安装，维护体验较差。

实施路径（适合规模增长后执行）：
1. 评估 `echo-sleuth-for-claude` 的安装复杂度是否已超过 shell 脚本的维护舒适度
2. 若超过，参考 `ui-ux-pro-max-skill` 的 `cli/` 目录结构，用 TypeScript + Commander.js 重写安装逻辑
3. 将数据同步（如 JSONL 数据库更新）也纳入 CLI，减少手动维护操作

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | ui-ux-pro-max-skill 核心目的 | 对齐度 |
|---------|----------------------------|-------|
| MarkQWu/claude-for-legal | 为 Claude 提供多领域（合同、诉讼、尽调等）法律工作流技能 | **高**：同样是多域并行技能 + 路由分层架构，目标用户同样是有特定专业领域需求的工程师/从业者 |
| MarkQWu/echo-sleuth-for-claude | 为 Claude 代理提供对话历史挖掘和经验提炼的方法论技能 | **中**：同样是多技能协作（路由思路可借鉴），但 echo-sleuth 是垂直纵深而非横向多域 |
| MarkQWu/drama-workshop-skills | 为 Claude 提供短剧制作领域的创作技能 | **低**：单一领域聚焦，无需路由层；但数据库枚举模式（如枚举剧情类型、冲突结构）可借鉴 |

### 8.2 在我的项目里复现的同类问题

| 问题类型 | ui-ux-pro-max-skill 中的表现 | 我的项目中的表现 | 文件 |
|---------|---------------------------|----------------|------|
| 缺少 `## Output Format` | `brand/SKILL.md` 无输出格式 | 同样缺失 | `echo-sleuth-for-claude`: `jsonl-core/SKILL.md`、`memory-management/SKILL.md`、`git-mining/SKILL.md`；`drama-workshop-skills`: `short-drama/SKILL.md` |
| 模糊量词 | `ui-styling/SKILL.md` 中"museum-quality" | `relevant` 在指令性文本中无客观标准 | `echo-sleuth-for-claude`: `experience-synthesis/SKILL.md` 和 `git-mining/SKILL.md` |
| 清单数量不一致风险 | `plugin.json` 描述 vs SKILL.md 内容历史性不一致 | 未专项检查，存在潜在漂移风险 | 各仓库 `plugin.json` |

### 8.3 别人的更优方案

**数字化知识规模声明（ui-ux-pro-max-skill 优于我）**：

`ui-ux-pro-max-skill` 用"67 种样式、161 套调色板、57 组字体搭配"等具体数字让用户立即理解插件的覆盖广度。我的 `claude-for-legal` 和 `echo-sleuth-for-claude` 的 `plugin.json` 描述使用的是功能性短语（"分析合同条款"、"挖掘对话历史"），缺乏类似的量化锚点。数字化声明不仅提升用户信任，也对内容维护形成可测试的约束。

**TypeScript CLI 安装工具（ui-ux-pro-max-skill 优于我）**：

`ui-ux-pro-max-skill` 的 `cli/` 工具将安装和初始化流程程序化，提供了比 `install.sh` 更好的错误处理、版本管理和用户反馈体验。我的三个仓库使用 shell 脚本安装，随着仓库内容增长，维护复杂度会线性提升。CLI 工具的抽象层使得添加新安装步骤不需要修改脚本逻辑结构。

### 8.4 反向：我的项目做得比他们好的地方

1. **零孤立引用**：`ui-ux-pro-max-skill` 有 10 处指向未捆绑外部技能的[孤立引用](#orphan-reference)，这些引用在用户只安装该插件时会静默失效。我的所有仓库中，跨技能引用均指向同仓库内的捆绑文件，不存在此类"看起来能用、实际不能用"的陷阱。这一差异对用户的实际体验影响远大于评分差异所显示的。

2. **无模板残留 Bug**：`slides/SKILL.md` 第 14 行的 `<args>$ARGUMENTS</args>` 是一个持续存在 36+ 天的可重现 Bug，会直接影响幻灯片生成技能的行为。我的技能文件从头编写，不存在"从模板复制后忘记替换占位符"的风险。

3. **命名一致性**：`ui-ux-pro-max-skill` 在 7 个技能中有 1 个（主路由技能）缺少其他 6 个都有的 `ckm:` [命名空间前缀](#namespace-prefix)，破坏了插件内部的命名一致性。我的仓库没有采用此类前缀方案，因此也不存在局部遗漏的一致性问题。

---

## 八、术语表

| 术语 | 说明 |
|------|------|
| <a name="frontmatter">frontmatter</a> | Markdown 文件开头由 `---` 包围的 YAML 元数据块。Claude Code 通过它解析技能的 `name`、`description`、`skills`（依赖的技能列表）等属性。[frontmatter](#frontmatter) 中的字段缺失或拼写错误通常会导致该文件对 Claude Code 不可见或行为异常。 |
| <a name="skill">skill（技能）</a> | 以 `SKILL.md` 文件形式存在的一段 Claude 行为规程，描述 Claude 在特定场景下应遵循的方法论、步骤和输出格式。技能不直接执行，而是被代理或命令通过 `skills:` frontmatter 字段加载后，作为上下文知识供 Claude 参考。 |
| <a name="manifest">manifest（清单文件）</a> | 插件根目录下的 `plugin.json`（或 `marketplace.json`），声明插件的 ID、版本、名称、包含的技能/命令/代理列表及其描述。Claude Code 通过清单文件发现和安装插件内容。清单中的数量声明、路径引用若与实际文件不一致，会导致用户获得错误的功能预期。 |
| <a name="namespace-prefix">namespace prefix（命名空间前缀）</a> | 在技能、命令或代理的 `name` 字段中加入的固定前缀，用于区分来自不同插件的同名资产。例如 `ckm:ui-styling` 中的 `ckm:` 是命名空间前缀。若同一插件的多个技能中只有部分使用了前缀，会破坏内部一致性，并可能导致按名称查找技能时出现歧义。 |
| <a name="orphan-reference">orphan reference（孤立引用）</a> | 技能、命令或代理文件中引用了未随该插件一起安装的外部资产（如另一个插件的技能）。孤立引用在技术上不会报错，但在用户只安装了当前插件的情况下，被引用的资产不存在，引用会静默失效——Claude 会忽略该引用或产生不可预期的行为。修复方式：捆绑被引用资产，或在文档中明确说明依赖关系。 |
| <a name="vague-quantifier">vague quantifier（模糊量词）</a> | 在指令性文本中使用的、缺乏客观判断标准的形容词或副词，例如 `proper`（恰当的）、`appropriate`（适当的）、`relevant`（相关的）、`museum-quality`（博物馆级品质）。这类词语要求 Claude 自行界定标准，导致不同上下文下输出不一致。修复方式：替换为可量化的具体标准（如将"museum-quality color palette"替换为"通过 WCAG AA 对比度测试、包含 5-7 个颜色角色的调色板"）。 |
| <a name="template-artifact">template artifact（模板占位符残留）</a> | 从模板文件复制后未替换的占位符文本，如 `<args>$ARGUMENTS</args>`、`{{YOUR_CONTENT_HERE}}`、`__PLACEHOLDER__`。这类残留在技能文件中通常不会触发解析错误，但会作为噪声文本传入 Claude 的上下文，导致行为不确定（Claude 可能将其解读为变量名、XML 标签或指令的一部分）。这类 Bug 往往是纯机械性的一行删除修复，但因其不影响大多数使用路径，容易在代码审查中被忽略。 |
