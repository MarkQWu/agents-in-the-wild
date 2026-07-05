# wasp-lang/open-saas — 学习案例

**仓库**：https://github.com/wasp-lang/open-saas
**Stars**：14349 | **来源**：upstream
**Audit 日期**：2026-05-05（历史快照）| **生成日期**：2026-07-05（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `examples-driven`, `offline-capable`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
wasp-lang/open-saas 是一个基于 [Wasp](#wasp) 框架的全栈 SaaS 模板，集成了 Auth、Stripe、PostHog 等开箱即用的 SaaS 基础设施。Claude Code skill 是该模板的「锦上添花」组件——3 个 skill 内嵌在 `template/app/.agents/skills/` 下，让 AI 助手能够即时获取最新文档并引导用户上手。

关键事实：
- 14349 颗星，是最受欢迎的 Claude Code 植入式 SaaS 模板之一
- skills 不是独立插件，而是嵌在 SaaS 模板内部，随整个项目结构一起分发
- 所有 skill 均使用「运行时拉取实时文档」策略（[llms.txt](#llms-txt) 索引），不把文档固化在 skill body 里
- CLAUDE.md 已声明「Always fetch before answering」原则，和 skill 策略完全一致

### 1.2 架构剖析
```
open-saas/
└── template/app/
    ├── CLAUDE.md                          ← 项目总指令（要求先 fetch 文档）
    └── .agents/
        └── skills/
            ├── guided-tour/SKILL.md       ← 引导用户逐节浏览项目结构
            ├── getting-started/SKILL.md   ← 引导用户从零运行项目
            └── add-wasp-skills/SKILL.md   ← 引导安装 Wasp 插件生态
```

- **文件类型分布**：3 个 SKILL.md，0 个 agent，0 个 command，1 个 CLAUDE.md，无 plugin.json（非独立插件，不注册到 marketplace）
- **编排关系**：3 个 skill 平列，互不依赖；CLAUDE.md 是「主控层」，声明全局文档获取规则，skill 是「执行层」，每个 skill 负责一种引导场景
- **跨件契约**：skill 和 CLAUDE.md 共享同一套 llms.txt 端点（`docs.opensaas.sh/llms.txt`）；`add-wasp-skills` 指向独立的 `wasp-agent-plugins` 仓库，清晰分离了 Open SaaS 知识和 Wasp 框架知识

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「文档不内联，运行时拉取」——skill body 只写流程步骤，不把 500 行文档塞进去；每次执行 skill 时实时 fetch 最新文档，始终与上游保持同步
- **解决什么问题**：SaaS 模板文档会随框架版本不断更新，如果固化在 skill 里，每次框架升级都要同步改 skill；live-fetch 策略彻底消除了「skill 内容落后于文档」的维护痛点
- **做了什么 trade-off**：实时性 vs. 离线可用性。如果 `docs.opensaas.sh` 挂掉，3 个 skill 全部降级到空响应。当前 skill 均无 fallback，这是一个隐患
- **反映什么认知模型**：作者把 skill 理解为「动态文档代理」而非「知识存储」，Claude 是中介，不是知识库

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「实时文档代理型 Skill（Live-Doc-Proxy）」**

skill body 只定义步骤流程，知识内容通过运行时 fetch 动态注入。skill 扮演「导游」，文档扮演「景区地图」，每次游览都拿最新地图。

模式特征清单：
- 特征 1：skill 体积极小（< 30 行），步骤简洁，知识全在外部 URL
- 特征 2：每个 skill 第一步固定是「获取文档索引（llms.txt）」
- 特征 3：skill 和 CLAUDE.md 共用同一套文档端点，策略一致
- 特征 4：无 plugin.json，直接随主项目分发，不走 marketplace
- 特征 5：三个 skill 分别对应三种典型新用户场景（入门、浏览、扩展），职责明确

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 文档迭代频繁的框架/SDK（Wasp、React、Stripe） | ✅ 高度适用 | skill 无需随文档更新，live-fetch 保持同步 |
| 嵌入特定项目的引导 skill（非通用 plugin） | ✅ 适用 | 不需要 marketplace 注册，随项目分发即可 |
| 离线环境或网络受限场景 | ❌ 不适用 | 全部步骤依赖网络 fetch，无 fallback |
| 需要快速响应、不能承受 fetch 延迟 | ❌ 不适用 | 每次执行都有网络往返时间 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 实时文档代理（本仓库） | wasp-lang/open-saas | 始终与上游文档同步，skill 几乎不需维护 | 无网络则失效，无 fallback |
| 参考文件驱动（本地文件） | twostraws/SwiftUI-Agent-Skill | 可离线，知识明确可审计 | 需手动同步更新 reference 文件 |
| 知识内联型 | 大多数简单 skill | 完全自包含，零依赖 | 文档过期后需改 skill，维护成本高 |

### 2.4 改进空间
1. **当前问题**：3 个 skill 均无 fallback——fetch 失败时 Claude 拿不到任何指导。**改进做法**：在 Step 1 后增加「如果 fetch 失败，告知用户访问 `https://docs.opensaas.sh` 并手动查找相关章节」的降级指令。**预期收益**：网络异常时用户仍有可操作路径。
2. **当前问题**：3 个 skill 均无 Output Format 声明，模型不知道用什么格式呈现引导结果。**改进做法**：在每个 skill 末尾加「以编号步骤列表呈现，每步完成后询问用户是否继续」。**预期收益**：减少 NLPM −10 penalty，提升用户体验一致性。
3. **当前问题**：`guided-tour` 中 `Keep the pace comfortable` 是不可测量指令。**改进做法**：改为「每完成一个 section 后，显式询问：'准备好继续下一个章节了吗？'」。**预期收益**：消除模糊量词，行为变得可预期。

---

## 三、过去审查发现（2026-05-05 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-05 当时得分 **87/100**（3 个 SKILL.md 加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `guided-tour/SKILL.md` | 84/100 | 缺少 output format；「clearly and concisely」「comfortable」模糊量词（−6） |
| `add-wasp-skills/SKILL.md` | 88/100 | 缺少 output format；「Walk the user through」指令模糊（−2） |
| `getting-started/SKILL.md` | 88/100 | 缺少 output format；「walk them through」指令模糊（−2） |

### 3.2 当时值得借鉴的模式

1. **Live-Doc-Proxy 模式** → 为什么好：文档和 skill 解耦，维护成本接近零 → 参考 `guided-tour/SKILL.md` Step 1-2 → 借鉴：凡是文档会持续演化的场景，skill body 只写步骤，内容用 fetch 注入
2. **frontmatter 三件套完整** → 为什么好：`name`、`description`、`user_invocable: true` 均在位，NLPM 可正常注册 → 3 个 skill 全部满分 → 借鉴：这是每个 skill 的最低基线，不可少
3. **llms.txt 统一索引** → 为什么好：skill 不硬编码具体文档 URL，而是先拿 index，再按需取章节，天然适应文档重组 → `CLAUDE.md` + 所有 skill 共用同一个 index 端点 → 借鉴：有多个文档页的项目，先建一个 llms.txt 索引文件，让 AI 通过 index 导航

### 3.3 当时的缺陷

1. **缺少 Output Format 声明**：3 个 skill 均不声明输出格式，模型每次可能以不同格式响应（有时用列表，有时用散文），用户体验不一致。根本原因：作者专注于步骤逻辑，忽视了「如何呈现」和「如何完结」。自查：我的 bureau skills 中 `review`、`scribe`、`capture` 均无 output mentions，同样存在此问题。
2. **模糊量词未清除**：「clearly and concisely」「comfortable pace」等形容词给 Claude 留下了主观判断空间，不同会话行为不可预期。根本原因：作者把给人类读者写说明文的习惯带进了给 AI 写指令——人类读完懂意图，AI 需要可测量标准。自查：gstack 59 个 skill 中大量命中（最高单文件 18 处），是我最大的质量债务。
3. **无 fallback 策略**：live-fetch 失败时无任何兜底，skill 在断网或文档迁移时完全失效。根本原因：设计时假设网络总是可用，未考虑降级路径。自查：gstack 中的 `browse`、`scrape` 等 skill 同样无 fetch 失败处理。

### 3.4 当时的优化机会

1. 为 `guided-tour/SKILL.md` 添加 Output Format 段：「用 H2 标题分隔每个 section，每节末尾用'准备好了吗？'询问确认，最后输出一条总结清单」（+10 分）
2. 替换 `clearly and concisely` → `每个 section 用 3-5 条要点呈现`；替换 `comfortable` → `每节后主动询问用户`（+6 分）
3. 在 3 个 skill 末尾添加 fetch 失败降级说明（+稳健性，无 NLPM 分值）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个 skill 缺少 output format | `grep -n "output\|Output" .../guided-tour/SKILL.md` → 无命中 | **仍然存在** | 2 个月内未修，低优先级；live-fetch 功能正常，作者不觉得格式问题影响使用 |
| `clearly and concisely`、`comfortable` | `grep -n "clearly\|comfortable" .../guided-tour/SKILL.md` → L23、L28 命中 | **仍然存在** | 模糊量词未清除，NLPM 仍扣分 |
| `Walk the user through`（add-wasp-skills） | `grep -n "Walk" .../add-wasp-skills/SKILL.md` → L19 命中 | **仍然存在** | 3 个缺陷全部原封不动 |

### 4.2 架构演进

与 audit 时相比，skill 目录结构无变化——仍是 3 个 skill 原位。但 CLAUDE.md（`template/app/CLAUDE.md`）有实质改善：增加了明确的「先 fetch 文档再作答」的优先级声明（`Always fetch and verify your knowledge against the Open SaaS documentation`），比 audit 时更清晰地告知 Claude 文档权威性。这说明作者在「AI 使用文档的方式」上投入了思考，但没有回头优化 skill 的格式质量。

### 4.3 新增的可学习模式

当前 HEAD 中的 `add-wasp-skills/SKILL.md` 指向独立的 `wasp-agent-plugins` 生态仓库，这个「skill 内的 skill 引导」模式（一个 skill 帮用户安装更多 skill）在 audit 时未被重点分析，但实际上是一种有趣的「skill 市场入口」设计。

---

## 五、校准

### 5.1 我已经在做对的

1. **所有 skill 均有 `name` frontmatter**：gstack 59 个 skill 全部命中，无一遗漏——wasp-lang 也是如此，双方对齐
2. **output mentions 覆盖率高**：gstack 大多数 skill 有大量 output 相关内容（8-49 处），比 wasp-lang 的 0 处好得多
3. **项目级别 CLAUDE.md 策略清晰**：gstack 应该也有文档引用策略；wasp-lang 的 CLAUDE.md 可作为参考，确认我的 CLAUDE.md 是否声明了文档优先原则

### 5.2 挑战 / 验证

这次案例挑战了我的一个假设：「低 NLPM 分数的仓库才值得深入学习」。wasp-lang/open-saas 得了 87 分，缺陷全是「未加 output format」和「用了几个模糊形容词」，但整体设计（live-doc-proxy、llms.txt、CLAUDE.md 与 skill 策略对齐）非常成熟。**有时候缺陷不重要，模式才重要。** 我以后会在寻找「优秀模式」时给高分仓库更多时间，而不只盯着低分的问题仓库。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 bureau skills 是否有 output format 声明
grep -rL "output\|Output" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md
# 命中后怎么办：对每个命中文件，在末尾加「## Output Format」段，描述输出结构和完结信号

# 检查我的 skill 里有多少 clearly / concisely / comfortable 等模糊量词
grep -rn -E "\bclearly\b|\bconcisely\b|\bcomfortably\b|\bappropriate pace\b" ~/.claude/skills/*/SKILL.md /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null
# 命中后怎么办：替换为可测量的完成信号（句子数量、字数限制、明确的确认步骤）
```

### 6.2 灵感 → 实施路径

1. **想法**：在 gstack 的 `browse` 和 `scrape` skill 中加 fetch 失败降级说明
   - **为何可行**：只需在 Step 1 后加 3 行 fallback 文字，5 分钟可完成
   - **第一步**：打开 `gstack/browse/SKILL.md`，在 fetch 步骤后增加「如果 fetch 失败，告知用户并列出可替代的手动获取路径」

2. **想法**：给 bureau 的 5 个无 output-mentions 的 skill 补 Output Format 段
   - **为何可行**：bureau 有 7 个 skill，其中 `review`、`scribe`、`capture`、`guide`、`compile` 均无 output 声明；每个 skill 只需加 1 个 section
   - **第一步**：打开 `MarkQWu-bureau/skills/review/SKILL.md`，在文件末尾追加「## Output Format：以 Markdown 表格呈现审查发现；每条发现包括：位置、问题、建议；最后一行输出总结句」

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 wasp-lang/open-saas 的核心目的**：为 SaaS 模板提供 AI 引导技能，帮助开发者快速上手
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是 Claude Code skill 套件，都面向开发者 | gstack 是通用生产力工具，open-saas skill 是嵌入式引导 | 高 |
| MarkQWu/bureau | 低 | 都使用 SKILL.md 格式 | bureau 专注知识管理，open-saas 专注框架引导 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Output Format 声明缺失 | `grep -cL "output" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | bureau 命中 5/7（review、scribe、capture、guide、compile） | 高 |
| 模糊量词（clearly、appropriate 等） | `grep -rn "appropriate\|comprehensive\|relevant" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | gstack 命中 59/59，最高 office-hours 12 处，review 13 处 | 高 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/skills/review/SKILL.md` → 在文末追加 Output Format 段 → 5 分钟可完成
- `MarkQWu-gstack/review/SKILL.md`（13 处命中）→ 优先处理这个高频 skill，把「comprehensive review」改为「涵盖以下 5 个维度的检查：…」→ 30 分钟可完成

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：Live-Doc-Proxy 策略
   - **本案例做法**：skill body 只写步骤，第一步固定是 fetch llms.txt 索引，然后按需拉具体章节（`guided-tour/SKILL.md`）
   - **我的项目现状**：gstack 的 `browse/SKILL.md` 和 `scrape/SKILL.md` 有 fetch 逻辑，但没有统一索引策略，每个 skill 硬编码各自的 URL
   - **如何借鉴**：在 gstack 主 `SKILL.md` 的 frontmatter 中声明 `docs_index` 字段，所有需要查文档的子 skill 引用这个统一入口

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Output Format 声明覆盖率
- **我的做法**：gstack 59 个 skill 全部有 `output_mentions`（最低 4 处），说明对输出结构有明确思考
- **本案例做法（弱在哪）**：wasp-lang 3 个 skill 全部 0 output mentions，模型自由发挥
- **意义**：gstack 在输出规范化上比 wasp-lang 做得扎实；bureau 虽然有问题，但 gstack 可以作为正面参照

---

## 八、术语表

### <a name="wasp"></a>Wasp
> 一个用于快速构建全栈 Web 应用的框架，底层是 React + Node.js + Prisma。开发者用类似配置文件的语法描述应用结构，Wasp 自动生成路由、认证等样板代码。Open SaaS 是基于 Wasp 构建的开箱即用 SaaS 模板。

### <a name="llms-txt"></a>llms.txt
> 一种约定格式的文本文件（类似 robots.txt），放在文档网站根目录，列出所有文档页的原始 URL。AI 工具可以通过读取 `llms.txt` 快速发现并按需获取具体文档，而不需要爬取整个网站。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`user_invocable` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 如何注册和调用。

### <a name="fallback"></a>fallback（降级路径）
> 当主要执行路径失败时（如网络不通、文件不存在），切换到备用的简化路径，保证用户仍然得到有用的响应而不是空白错误。
