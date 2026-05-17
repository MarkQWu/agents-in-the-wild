# AgriciDaniel/claude-blog — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-blog
**Stars**：562 | **来源**：xiaolai 上游审查
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD）
**主题标签**：model-pinning, vague-quantifier, examples-driven, manifest-discipline, template-design

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-blog 是一个以 Claude Code 插件形式发布的博客写作助手，当前 HEAD（901e9217b0f542e10a0b0c0f8692a95d1f42f43c）已从审查时的 28 个自然语言制品扩展至 35 个以上技能文件。插件面向内容创作者，提供从策略规划、写作、SEO 优化到多语言本地化的全流程自动化支持。仓库主体语言为 Markdown（技能与代理定义），辅以 Python 脚本（NotebookLM 集成、流程同步）。

### 1.2 架构剖析

插件采用三层架构：

1. **编排层**：`blog/SKILL.md` 作为顶层技能调度者，统一入口。
2. **功能技能层**：22 个专项技能覆盖写作、改写、分析、摘要、大纲、日历、策略、SEO 检查、Schema、图表、再利用、地理定向、审计、图像、关键词蚕食、事实核查、角色、分类法、NotebookLM、音频等领域；审查后新增多语言集群（blog-translator、blog-locale-audit、blog-localize、blog-multilingual、blog-translate、blog-flow、blog-cluster）。
3. **参考层**：`references/` 和 `templates/` 目录存放被技能引用的静态内容规范（如 `cta-placement.md`）。

代理层由 4 个 Sonnet 级别代理组成（blog-researcher、blog-writer、blog-reviewer、blog-seo），分别负责研究、写作、审阅和搜索引擎优化。代理均未在前置元信息中声明 `model:`，所有代理均缺少 `<example>` 块。

配套工程文件包括 `pyproject.toml`、`requirements.lock`、`scripts/sync_flow.py`，以及 `docs/` 目录下 6 份文档，体现出项目工程化程度持续提升。

### 1.3 设计思路 / 方法论

- **技能即模块**：每个写作环节独立为一个技能文件，可被 `blog/SKILL.md` 单独或组合调用，符合"单一职责"原则。
- **参考资料分离**：将规范性内容（CTA 放置指南、模板）抽离到 `references/` 目录，避免在技能文件中重复定义，但也带来了引用一致性维护的挑战。
- **多语言扩展**：审查后新增的多语言集群反映出作者对国际化场景的持续投入，体现清晰的演进方向。
- **工程配套**：`requirements.lock` 和 `pyproject.toml` 的引入说明项目已从纯 Markdown 插件向可重现工程环境演进。

---

## 二、过去审查发现（2026-04-06 历史快照）

### 2.1 当时质量评分（NLPM）

**综合评分：92 / 100**（安全状态：BLOCKED）

| 文件 | 得分 | 主要扣分项 |
|------|------|-----------|
| agents/blog-researcher.md | 72 | 无 model 声明（-5）、无示例块（扣分上限）、模糊词"suitable"×2、"appropriate"×2、"relevant"×1（-8） |
| agents/blog-writer.md | 76 | 无 model 声明（-5）、无示例块、模糊词"natural"、"relevant" |
| agents/blog-reviewer.md | 77 | 无 model 声明（-5）、无示例块、声明 Bash 工具但从未使用（-3） |
| agents/blog-seo.md | 80 | 无 model 声明（-5）、无示例块 |
| skills/blog-write/SKILL.md | 88 | 模糊词"appropriate"×2、"relevant"×3、"suitable"×1（-12） |
| skills/blog-strategy/SKILL.md | 92 | 模糊词"appropriate"、"relevant"×2、"authentic"（-8） |
| CLAUDE.md | 88 | 含 curl-pipe-sh 安装命令（安全风险） |

### 2.2 当时值得借鉴的模式

- **三层技能架构**清晰，编排技能统一入口，子技能职责单一，参考资料独立存放，分层思路成熟。
- **技能覆盖度广**：在 28 个制品中涵盖写作全流程，体现出作者对博客工作流的深度理解。
- **plugin.json 与 CLAUDE.md 规范**：插件元信息结构完整，安装流程有文档说明，具备可发布条件。
- **高整体评分（92/100）**：在大量技能文件的情况下保持较高平均质量，说明作者具备 NL 编程基础规范意识。

### 2.3 当时的缺陷

1. **所有 4 个代理缺少 `model:` 声明**：运行时依赖 claude-code-action 默认值，模型升级时行为不可预测。
2. **所有 4 个代理缺少 `<example>` 块**：代理无法通过示例约束输出格式，降低行为可重现性。
3. **`blog-reviewer.md` 声明 Bash 工具但未使用**：引入不必要的权限请求。
4. **`skills/blog-write/SKILL.md` 引用不存在文件**：第 161、165 行引用 `references/cta-placement.md`，该文件在审查时不存在，导致运行时引用断裂。
5. **技能文件中大量模糊量词**：`appropriate`、`suitable`、`relevant`、`natural` 等词在多个技能文件中反复出现，削弱指令的可执行性。

### 2.4 当时的优化机会

- 在所有代理前置元信息中添加 `model:` 字段，明确固定模型版本。
- 为每个代理添加至少一个真实的 `<example>` 输入/输出对，展示期望行为。
- 将模糊量词替换为可测量的具体标准（如"3-5 个段落"替代"appropriate length"）。
- 补充缺失的 `references/cta-placement.md` 文件。
- 修复 CLAUDE.md 中的 curl-pipe-sh 安装命令，提供带完整性校验的安装方式。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 缺陷 | 审查时状态 | 当前 HEAD 状态 |
|------|-----------|--------------|
| 代理 `model:` 声明缺失 | 4 个代理全部缺失 | **仍未修复**：`grep -rn "^model:" agents/` 返回空 |
| 代理 `<example>` 块缺失 | 4 个代理全部缺失 | **仍未修复**：`grep -rl "<example>" agents/` 返回空 |
| `references/cta-placement.md` 不存在 | 缺失 | **已修复**：文件现存于 `skills/blog/references/cta-placement.md` |
| 技能文件模糊量词 | 若干次出现 | **未改善，反而恶化**：当前 `grep -rn "appropriate|suitable|relevant" skills/` 达 129 处，远超审查时数量 |
| `blog-reviewer.md` 声明但未使用 Bash | 存在 | 未见修复记录 |
| CLAUDE.md curl-pipe-sh 命令 | 第 119 行 | 安全审计标记为 CRITICAL，需人工确认是否仍存在 |

### 3.2 架构演进

审查后仓库经历了显著扩展：

- **新增多语言集群**（7 个新技能）：blog-translator、blog-locale-audit、blog-localize、blog-multilingual、blog-translate、blog-flow、blog-cluster，将插件从单一语言写作工具扩展为国际化内容平台。
- **工程化提升**：引入 `pyproject.toml`、`requirements.lock`、`scripts/sync_flow.py`，Python 脚本的依赖管理从无到有，可重现性大幅提升。
- **文档体系建立**：`docs/` 目录下新增 6 份文档，配合 `CITATION.cff`、`PRIVACY.md`、`SUPPORT.md`，项目开源规范性显著提升。
- **MCP 配置安全化**：将 `.mcp.json` 移至 `.mcp.example.json`，降低用户直接执行自动安装 MCP 包的风险（尽管 HIGH 安全风险的根本原因——`npx -y` 自动安装——仍需在示例文件中提示）。
- **参考文件补全**：`cta-placement.md` 已归入 `skills/blog/references/`，修复了审查时发现的引用断裂问题。

### 3.3 新增的可学习模式

- **多语言集群设计**：将翻译、本地化审计、多语言协调拆分为独立技能，而非塞入单一"翻译"技能，体现细粒度职责划分的工程判断。
- **`requirements.lock` 固定依赖**：Python 脚本配合锁文件，避免上游包版本漂移破坏行为，是 NL 制品配套脚本的最佳实践。
- **`CITATION.cff` 学术引用支持**：对希望插件被学术或专业工具正式引用的项目具有参考价值。

---

## 四、校准

### 4.1 我已经在做对的

- **三层技能架构**（编排 → 功能 → 参考）是可复用的组织模式，适用于任何复杂度超过 5 个技能的插件。
- **参考资料独立存放**（`references/` 目录）减少了技能文件的臃肿，提升了内容规范的可维护性。
- **锁文件管理 Python 依赖**是配套脚本的标准做法，值得在所有含 Python 自动化的插件中推广。

### 4.2 挑战 / 验证

claude-blog 案例提出了两个值得深入思考的问题：

**挑战一：架构扩展与质量退化的矛盾**
仓库从 28 个制品扩展至 35 个以上，但模糊量词数量从审查时的个位数增长到 129 处。这说明在快速添加新技能时，质量约束机制（如 NLPM 评分门槛）若未纳入 CI 流程，规模扩展本身会带来质量稀释。**验证点**：在自己的项目中，新增技能时是否同步运行 `/nlpm:score` 检查？是否有最低分数门槛阻止低质量技能合并？

**挑战二：代理前置元信息的长期治理**
4 个代理在超过一年的开发周期内始终缺少 `model:` 声明和 `<example>` 块，说明这类"结构性缺失"在无自动检查时极易被忽视——即使维护者对其他部分保持高质量。**验证点**：当前项目中的所有代理是否都声明了 `model:`？是否都有至少一个 `<example>` 块？

---

## 五、行动

### 5.1 自查动作

以下命令可在任何含 NL 制品的项目中直接运行：

```bash
# 检查所有代理是否声明 model:
grep -rn "^model:" agents/ || echo "⚠️  存在代理未声明 model:"

# 检查所有代理是否包含示例块
grep -rl "<example>" agents/ || echo "⚠️  存在代理缺少 <example> 块"

# 统计技能文件中的模糊量词
grep -rn "appropriate\|suitable\|relevant\|natural\|reasonable" skills/ | wc -l

# 检查 curl-pipe-sh 安装命令
grep -rn "curl.*|.*bash\|curl.*|.*sh" . --include="*.md" --include="*.sh"

# 检查 npx -y 自动安装
grep -rn "npx -y" . --include="*.json" --include="*.md"

# 验证技能文件中的引用目标是否存在
grep -rn "references/" skills/ | awk -F'references/' '{print $2}' | cut -d'"' -f1 | sort -u
```

### 5.2 灵感 → 实施路径

**灵感一：三层架构 → 立即可用**

在当前项目中检查技能组织方式是否已分离编排层与功能层。若所有技能平铺在同一目录，可参考 claude-blog 的模式：

1. 创建顶层 `SKILL.md` 作为统一调度入口。
2. 将功能技能移入子目录（按职责域命名，如 `blog-write/`、`blog-seo/`）。
3. 将规范性内容（模板、引用标准）移入 `references/` 子目录，并在技能文件中通过路径引用。

**灵感二：模型固定 → 今日可改**

为所有代理前置元信息补充 `model:` 字段，选择当前最适合任务复杂度的模型（研究类任务用 Sonnet，简单分类用 Haiku）。格式参考：

```yaml
---
name: blog-researcher
model: claude-sonnet-4-6
tools: WebSearch, Read
---
```

**灵感三：`<example>` 块 → 本周可完成**

为每个代理添加一个真实的输入/输出示例。示例无需完美，但应覆盖最常见的调用场景。优先为输出格式最不稳定的代理补充示例（通常是研究类和写作类代理）。

**灵感四：模糊量词治理 → 纳入 CI**

将 `grep -rn "appropriate\|suitable\|relevant" skills/ | wc -l` 的结果与基准值对比，超过阈值时阻断合并。claude-blog 的案例表明，若无自动门槛，规模扩展必然导致模糊词积累。可在 `.github/workflows/` 中添加一步：

```yaml
- name: 检查模糊量词数量
  run: |
    COUNT=$(grep -rn "appropriate\|suitable\|relevant" skills/ | wc -l)
    echo "模糊量词数量：$COUNT"
    [ "$COUNT" -le 30 ] || (echo "超过阈值 30，请替换为具体标准" && exit 1)
```

**灵感五：安全配置 → 参考 .mcp.example.json 模式**

将 `.mcp.json` 更名为 `.mcp.example.json`，在 README 或 CLAUDE.md 中说明用户需自行复制并填写真实配置。避免将含 `npx -y` 自动安装的 MCP 配置直接提交为默认配置文件，减少用户在不知情的情况下触发网络安装行为。
