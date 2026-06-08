# 2389-research/review-squad — 学习案例

**仓库**：https://github.com/2389-research/review-squad
**Stars**：1 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-08（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `template-design`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Review Squad 是一个 Claude Code 插件，把代码审查"分派成专家小组"——用户说一句"review this"，就能让并行的 SEO 专家、无障碍审计师、安全分析师、性能工程师等各自独立分析，最后汇总成一份按严重度排序的报告。作者 2389-research 同时维护另一个插件 simmer（本批次第 3 个案例），两者在 2389.ai 旗下。仓库极简：6 个文件，零可执行面，安全评级 CLEAR。

关键事实：
- 4 个 skill：experts（并行专家审计）、normies（普通用户体验测试）、regulars（任务型用户）、well-actually（挑剔型评审者）
- 0 个命令，0 个 hook，0 个脚本，0 个 MCP 配置
- NL 得分 96/100，是本批次得分最高的仓库
- 安装方式：`/plugin install review-squad@2389-research`

### 1.2 架构剖析

**目录结构**：
```
skills/
  experts/SKILL.md      ← 并行专家审计（SEO/无障碍/安全/性能…）
  normies/SKILL.md      ← 非技术用户体验模拟
  regulars/SKILL.md     ← 任务导向型用户测试
  well-actually/SKILL.md← 挑剔型评审（找文案、命名、逻辑错误）
CLAUDE.md               ← 插件概览（无 frontmatter，低分原因）
.claude-plugin/plugin.json  ← manifest（满分 100）
```

**文件类型分布**：4 个 SKILL.md / 0 个 command / 0 个 agent / 0 个脚本

**编排关系**：完全平铺。没有 router，没有层次。4 个 skill 各自独立，CLAUDE.md 是人类阅读的总览文档。触发方式：用户输入关键词（如"review"、"audit"、"launch review"），Claude 选择对应 skill 调用。

**跨件契约**：
- CLAUDE.md 记录了并行/串行的分派模式（experts 并行，normies/regulars/well-actually 串行）
- CLAUDE.md 中的"No-Code Guards"说明了哪些 skill 需要 browser MCP，哪些不需要（experts 只读代码，其余三个需要 browser）
- plugin.json `commands: []` 是故意留空的——这个插件设计为只暴露 skill，不暴露 slash command

### 1.3 设计思路 / 方法论

**核心设计哲学**：「角色清晰 > 规模大」。4 个 skill 代表 4 种视角，每种视角有独立的认知模型和评审策略，拒绝合并成一个"万能 review skill"。

**解决什么问题**：单一 reviewer 的盲区问题——一个 SEO 专家不会在意 TypeScript 类型安全，一个无障碍审计师不会关注数据库查询性能。Review Squad 通过角色分工强制覆盖多个盲区。

**做了什么 trade-off**：
- 4 个独立 skill vs 1 个带参数的 review skill：可维护性和清晰度优先，每个 skill 的触发条件和 agent prompt 完全独立，不共享代码
- 无 slash command vs 有命令入口：选择纯 skill，让用户通过自然语言触发，降低使用门槛（不需要记命令名）

**反映什么认知模型**：作者把"代码审查"看作一个多维度、不可压缩的任务——每一个视角都需要独立的专注力，不能由同一个 agent 在同一个上下文中同时扮演所有角色。这是对 AI 单上下文注意力有限这一特性的直接回应。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：平铺多角色 Skill 架构（Parallel Reviewer Dispatch）

最简单的多-skill 设计：每个 skill 代表一种独立角色，无 router，无层级，直接靠触发关键词区分调用场景。

模式特征清单：
- **零可执行面**：纯 markdown，没有脚本、hook 或 MCP 配置，安全风险极低
- **角色和触发词绑定**：每个 skill 有专属触发 keyword 列表，减少歧义
- **No-Code Guard 内置**：需要 browser MCP 的 skill 在 prompt 模板中内置"无代码时如何处理"的分支
- **并行/串行混用**：experts 并行发散，其他串行聚焦，分工明确

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多视角分析的单一对象（代码/文档/设计稿） | ✅ 高度适用 | 每个角色专注自己的维度，不互相干扰 |
| skill 数 ≤ 5 的插件 | ✅ 高度适用 | 平铺清晰，无需引入 router 层 |
| 有复杂状态需要跨 skill 传递的工作流 | ❌ 不适用 | 平铺架构无法优雅地在 skill 间传递上下文 |
| 需要动态选渠道的搜索/数据聚合工具 | ❌ 不适用 | 应使用 Router + Channels 架构（见 Autosearch 案例） |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 平铺多角色（本仓库） | review-squad | 极简，安全，易维护 | skill 多后触发词容易冲突 |
| 带命令的多 skill | Autosearch | 入口明确，参数可传递 | 用户需要记命令名 |
| 单体大 skill + 参数 | 很多简单插件 | 开发成本最低 | 角色混淆，注意力分散 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 无 frontmatter，NLPM 扫描器无法解析，扣了 -10 分。**改进做法**：在 CLAUDE.md 顶部加 `---\nname: review-squad\ndescription: ...\n---`。**预期收益**：NLPM 得分从 96 → 98+，同时让机器能正确索引这个插件。
2. **当前问题**：experts/SKILL.md L95 的"suggest what's relevant"没有具体标准，什么算 relevant？**改进做法**：改为"suggest reviewers that directly address gaps in the detected tech stack (e.g., suggest an i18n reviewer for multi-locale sites)"。**预期收益**：AI 选择 reviewer 时有可验证的标准，而不是依赖主观判断。
3. **当前问题**：CLAUDE.md 中没有 worked invocation example，读者看完不知道一次完整调用长什么样。**改进做法**：加一个 Example 块，展示用户输入"pre-launch review"后每个 reviewer 输出什么类型的反馈。**预期收益**：用户上手时有参照，减少试错成本。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **96/100**，安全评定：CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 80 | 无 frontmatter，无 invocation 示例 |
| skills/experts/SKILL.md | 98 | "relevant" 一处模糊量词（L95） |
| skills/normies/SKILL.md | 99 | Clean |
| skills/regulars/SKILL.md | 99 | Clean |
| skills/well-actually/SKILL.md | 99 | Clean |
| .claude-plugin/plugin.json | 100 | Clean |

### 3.2 当时值得借鉴的模式

1. **No-Code Guards** → normies 和 regulars 的 SKILL.md 内置了"如果没有代码可以审查时的处理分支"，防止 agent 在空输入时瞎跑。根本原因：明确的 fallback 路径比让 AI 自行决定更可预测。借鉴：任何依赖外部资源（代码、文件、URL）的 skill 都应该有 "如果资源不存在时" 的显式分支。
2. **plugin.json commands: []** → 空命令数组是有意为之，不是遗漏。说明作者主动选择了"纯 skill 无命令"的设计。根本原因：如果 skill 的触发条件已经足够明确（review/audit/launch check），强制要求用户记命令名是额外摩擦。借鉴：不是所有插件都需要 slash command。
3. **并行 experts + 串行 normies/regulars** → CLAUDE.md 明确文档化了哪些 skill 适合并行运行（不需要 browser MCP 的 experts），哪些需要串行（browser-using agents）。根本原因：browser MCP 资源竞争，并行运行会导致冲突。借鉴：在文档中明确标注并发约束，避免用户误操作。
4. **触发关键词列表** → 每个 skill 有多个触发关键词（review/audit/launch review/health check/pre-launch/post-refactor），覆盖用户可能用的多种表达方式。根本原因：用户不会总是用"标准"词，需要宽泛匹配。借鉴：设计 skill 触发条件时，想 3-5 个同义词，不要只写一种。
5. **Relationship block** → CLAUDE.md 中列出了互补插件（fresh-eyes-review、scenario-testing），帮助用户了解这个插件的边界在哪里。根本原因：用户经常在"用这个还是那个"上困惑，主动建立生态关系减少搜索成本。借鉴：在 CLAUDE.md 中加 "See also" 或 "Complementary plugins" 章节。

### 3.3 当时的缺陷

1. **CLAUDE.md 无 YAML frontmatter** → 根本原因：作者把 CLAUDE.md 当成纯人类阅读文档，忽略了机器可读性。NLPM 的自动发现和评分依赖 frontmatter 中的 `name` 和 `description` 字段。自查：我的 echo-sleuth/CLAUDE.md 也是从 `# Echo Sleuth — Plugin Development Guide` 直接开始，没有 frontmatter——同类问题！
2. **experts/SKILL.md 中 "always suggest what's relevant" 无标准** → 根本原因："relevant"是高度主观的词，没有给出"什么情况下应该建议额外 reviewer"的判断标准，AI 执行时结果不可预测。自查：我的 echo-sleuth 和 claude-for-legal skills 中同样存在 `relevant` 用作模糊量词的情况。
3. **CLAUDE.md 无 worked invocation example** → 根本原因：写文档时习惯于描述"是什么"，而不是"长什么样"。一个具体的输入→输出例子比三段描述更有信息量。自查：我的 drama-workshop-skills 的 SKILL.md 有完整的中文示例，这方面做得更好。

### 3.4 当时的优化机会

1. **为 CLAUDE.md 添加 frontmatter** → 3 行代码，NLPM 得分从 96 → 98+。
2. **experts/SKILL.md L95 用具体标准替换 "relevant"** → 改成"suggest reviewers that directly address gaps in the detected tech stack"。
3. **CLAUDE.md 补充 worked example** → 展示一个完整的"用户输入 → 各 reviewer 输出"的映射。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 无 frontmatter | `head -5 CLAUDE.md` | **仍存在**：首行为 `# Review Squad Plugin`，无 `---` frontmatter 块 | 机器可读性问题未修复，NLPM 扫描器仍无法索引 |
| experts/SKILL.md L95 "relevant" | `grep -n "relevant" skills/experts/SKILL.md` | **仍存在**：L95 `always suggest what's relevant` 未改动 | 模糊量词持续存在，AI 选择 reviewer 时仍缺乏确定性 |

### 4.2 架构演进

从 audit 快照到当前 HEAD，文件结构无变化（仍是 6 个文件）。没有新增 skill 或命令。说明作者选择了「稳定小插件」策略，不做功能扩展，保持当前定义明确的 4 种审查视角。

### 4.3 新增的可学习模式

暂无——当前 HEAD 和 audit 快照结构一致，无新模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **SKILL.md 有完整 frontmatter**：drama-workshop-skills 和 claude-for-legal 的所有 SKILL.md 都有 `name` 和 `description` 字段，这方面比 review-squad 的 CLAUDE.md 做得好。
2. **单职责 skill**：echo-sleuth 的每个 skill（experience-synthesis、git-mining）都聚焦单一功能，和 review-squad 的 4 个角色分明的设计理念一致。
3. **触发条件描述**：claude-for-legal 的 skill `description` 字段列出了触发短语（"当用户说X时使用"），和 review-squad 的触发关键词设计类似。

### 5.2 挑战 / 验证

这次案例**挑战**了我对"CLAUDE.md 是给人看的，不需要 frontmatter"的假设。review-squad 的 CLAUDE.md 得了 80 分，主要原因就是缺 frontmatter——这证明 CLAUDE.md 同时要服务人类读者和机器扫描器，两者不是非此即彼的。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 CLAUDE.md 是否缺少 frontmatter（对应本案例核心缺陷）
head -3 /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/CLAUDE.md
```
命中（首行是 `#` 而不是 `---`）后怎么办：在 CLAUDE.md 顶部加：
```yaml
---
name: echo-sleuth
description: Mine past Claude Code conversations for decisions, mistakes, patterns, and wisdom.
---
```

```bash
# 检查我的 skills 中的模糊量词（对应 experts/SKILL.md 的 "relevant" 问题）
grep -rn -E '\b(relevant|appropriate|robust)\b' \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md \
  /tmp/my-repos/MarkQWu-claude-for-legal/*/skills/*/SKILL.md 2>/dev/null | head -20
```
命中后怎么办：把每个 `relevant` 替换成具体条件（如"relevant → 含目标关键词或在指定时间范围内的"）。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 CLAUDE.md 加 frontmatter，同时加一个 worked example 章节。
   - **为何可行**：review-squad 的案例证明这两个改动合计只需 10 行，但能把 NLPM 分从 ~80 提升到 95+。
   - **第一步**：打开 `echo-sleuth/CLAUDE.md`，在最顶部插入 3 行 frontmatter。10 分钟。

2. **想法**：为 claude-for-legal 补充 "See also" 或 "Complementary plugins" 章节。
   - **为何可行**：review-squad 的 Relationship block 设计很实用，用户经常问"这个插件和那个插件有什么关系"。
   - **第一步**：在 claude-for-legal/README.md 底部加 "Related tools" 章节，列出互补工具。30 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 2389-research/review-squad 的核心目的**：代码/项目多视角并行审查，通过角色分工覆盖单一评审者的盲区。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都有多个专业视角的独立 skill（legal 有 regulatory/product/employment 等域）；都有触发条件描述 | review-squad 面向代码审查，claude-for-legal 面向法律工作流 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都是纯 markdown 无可执行面 | 目的不同（历史会话挖掘 vs 代码审查） | 中 |
| MarkQWu/drama-workshop-skills | 无 | — | 目的完全不同 | 低 |

若全部「无」：我的仓库中 claude-for-legal 在架构上最接近 review-squad 的"多专业角色平铺"模式。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 无 frontmatter | `head -3 CLAUDE.md` | echo-sleuth/CLAUDE.md 确认：首行是 `# Echo Sleuth — Plugin Development Guide`，**无 frontmatter** | 高 |
| "relevant" 等模糊量词 | `grep -rn "relevant" skills/*/SKILL.md` | echo-sleuth 中 2 处（experience-synthesis:118, git-mining:93）；claude-for-legal 中多处 | 中 |
| 无 worked invocation example | CLAUDE.md 内搜索示例块 | echo-sleuth 的 CLAUDE.md 有架构说明但无完整 input→output 示例 | 中 |

**命中后的具体行动建议**：
- `echo-sleuth/CLAUDE.md` → 加 frontmatter（3 行）+ 加 1 个 Example 章节（展示用户输入"recall 上周 auth 决策" → echo-sleuth 输出什么）→ 15 分钟
- `echo-sleuth/skills/experience-synthesis/SKILL.md:118` → `relevant sessions` → `sessions containing this keyword or tagged with this project` → 5 分钟

### 8.3 别人的更优方案

1. **领域**：No-Code Guard 内置在 skill body 中
   - **本案例做法**：normies 和 regulars 的 SKILL.md 在 agent prompt 模板中有"如果没有可测试的界面时"的明确分支，不让 AI 在空输入时自行发挥。
   - **我的项目现状**：claude-for-legal 的某些 skill（如 launch-review）在没有 PRD 时没有显式 fallback 处理。
   - **如何借鉴**：在 claude-for-legal 的每个 skill body 中加"如果提供的文件/URL 不存在时，先问用户 X"的明确分支。按 skill 逐一补充，每个 10-15 分钟。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：SKILL.md 正文有完整 worked examples
  - **我的做法**：drama-workshop-skills 的 SKILL.md 包含多个完整的中文示例对话，包括用户输入和期望输出的完整映射。
  - **本案例做法**（弱在哪）：CLAUDE.md 无 worked example，skill 文件有 agent prompt template 但没有"真实调用长什么样"的示范。
  - **意义**：用户上手成本更低，也是 NLPM 评分中"Examples-driven"一条的加分项。

---

## 八、术语表

### <a name="No-Code-Guard"></a>No-Code Guard
> review-squad 的术语，指在需要有代码/界面才能运行的 skill 中内置的"如果没有代码时的处理分支"。防止 AI 在输入不完整时仍然执行整个 skill 流程（比如没有 URL 却尝试用 browser 访问网站）。通用含义：对外部依赖做防御性检查，输入不满足条件时先问用户而不是猜。

### <a name="Dispatch"></a>Dispatch（并行分派）
> 同时启动多个独立 agent/skill，各自分析同一个对象，最后合并结果。review-squad 的 experts skill 并行 dispatch 4-6 个专家 agent，每人只关注自己的专业维度。对比：串行运行（一个做完再下一个）更慢但资源占用更低；并行分派更快但如果涉及 browser MCP 等共享资源则可能冲突。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。对于 CLAUDE.md 来说，有 frontmatter = 机器可读 = NLPM 可以正确索引和评分；没有 frontmatter = 只有人类能读，机器当成普通文档处理。
