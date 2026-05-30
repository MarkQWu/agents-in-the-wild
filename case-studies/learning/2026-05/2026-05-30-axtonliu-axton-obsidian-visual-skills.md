# axtonliu/axton-obsidian-visual-skills — 学习案例

**仓库**：https://github.com/axtonliu/axton-obsidian-visual-skills
**Stars**：2669 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-30（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `examples-driven`, `manifest-discipline`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
一个专注于 [Obsidian](#obsidian) 可视化工具的 skill 三件套：Excalidraw 手绘图、Mermaid 流程图、Obsidian Canvas 画布——用户一条安装命令把三种视觉化能力接入 Claude Code。Axton Liu（hey@axtonliu.com）维护，提供中英双语 README，面向需要在 Obsidian 笔记中生成可视化图表的知识工作者。

关键事实：
- 2669 stars，说明"Obsidian + AI 可视化"是一个存在真实需求的细分市场
- 仓库结构极简：3 个 skill 目录 + 各自的 references/ + 一个 [marketplace.json](#manifest)
- 注册机制：用 `.claude-plugin/marketplace.json` 而非 `plugin.json`（与标准略有不同）
- 中英双语 README 说明作者同时面向中文和英文用户群体

### 1.2 架构剖析
```
axton-obsidian-visual-skills/
├── excalidraw-diagram/
│   ├── SKILL.md              # 94分，三种输出模式明确，无重大问题
│   └── references/
│       └── excalidraw-schema.md
├── mermaid-visualizer/
│   ├── SKILL.md              # 88分，6处模糊量词
│   └── references/
│       └── syntax-rules.md
├── obsidian-canvas-creator/
│   ├── SKILL.md              # 89分，6处模糊量词，缺JSON输出示例
│   ├── assets/
│   └── references/
│       ├── canvas-spec.md
│       └── layout-algorithms.md
├── assets/
├── .claude-plugin/
│   └── marketplace.json      # 注册文件：列出3个skill路径
├── README.md
└── README_CN.md
```

- **文件类型分布**：3 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：3 个 skill 完全平列，无互相调用；每个 skill 独立解决一种可视化需求
- **跨件契约**：3 个 skill 均引用各自 `references/` 子文档；[marketplace.json](#manifest) 是唯一注册点

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单一功能，深度覆盖」—— 每个 skill 只做一种可视化，但把那种可视化讲透（输出格式、设计规则、错误处理）
- **解决什么问题**：Obsidian 用户习惯在笔记里用各种可视化工具，但让 AI 直接输出正确格式的图表代码门槛高；这套 skill 封装了格式规范，降低出错率
- **Trade-off**：3 个独立 skill 而非 1 个"通用可视化 skill"——增加了安装粒度控制（用户可按需选装），牺牲了跨工具的统一上下文
- **认知模型**：把 skill 看作"工具适配器"——把某个具体工具（Excalidraw）的规范翻译成 agent 可以遵循的操作协议

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单职责工具适配器三件套」**

特征清单：
- 特征 1：每个 skill 对应一种具体工具，不混搭
- 特征 2：每个 skill 都有独立 references/ 放置格式规范（schema、语法规则、布局算法）
- 特征 3：skill 主体描述"如何判断该用这个工具"，references/ 描述"格式细节"
- 特征 4：三个 skill 可独立安装，互不依赖
- 特征 5：中英双语注册，目标用户群明确

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有 3 种以上同类工具需要覆盖 | ✅ 适用 | 独立 skill 让用户按需选择 |
| 工具输出格式非常具体（JSON/DSL）| ✅ 适用 | references/ 可以放完整的格式 spec |
| 跨工具需要统一决策逻辑 | ❌ 不适用 | 没有 router skill，无法自动选工具 |
| 需要在 skill 间共享上下文 | ❌ 不适用 | 三个 skill 完全解耦 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单职责三件套（本仓库）| axtonliu/axton-obsidian-visual-skills | 粒度精细，可独立安装 | 无自动选工具的 router |
| 统一可视化 skill | 假设存在 | 一次安装搞定 | skill 过大，选择逻辑混入 |
| Router + 子 skill | obra/superpowers | 自动路由，用户无需关心工具选择 | 实现复杂度高 |

### 2.4 改进空间
1. **当前问题**：`obsidian-canvas-creator/SKILL.md` 示例块只展示操作步骤，不展示实际 JSON Canvas 输出。**改进做法**：在 Examples 段增加一个 20 行的最小 JSON Canvas 输出片段。**预期收益**：agent 在执行时有格式锚点，不需要每次去读 canvas-spec.md 才能推断格式。
2. **当前问题**：3 个 skill 之间没有"选哪个"的决策提示。**改进做法**：在 README 或 SKILL.md description 中加一行"当用户提到 Canvas/Excalidraw/Mermaid 时分别触发对应 skill"的决策规则。**预期收益**：agent 调用时不会混淆工具。
3. **当前问题**：模糊量词"appropriate/balanced/sensible/subtle/strategic"共 6 处，分布在 2 个 skill 里。**改进做法**：逐一替换为可量化标准（如"balanced"→"节点数 5-15 个时使用左右布局，15 个以上改用层次布局"）。**预期收益**：agent 执行图表布局时有明确依据。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| mermaid-visualizer/SKILL.md | 88 | 模糊量词 "appropriate/balanced/sensible" (-6) |
| obsidian-canvas-creator/SKILL.md | 89 | 模糊量词 "appropriate/subtle/Strategic" (-6)；示例缺 JSON 输出格式（informational）|
| excalidraw-diagram/SKILL.md | 94 | 中文双语段存在轻微模糊措辞 |

### 3.2 当时值得借鉴的模式
1. **三模式 Excalidraw** → `excalidraw-diagram/SKILL.md` 明确定义了三种输出模式（快速草图/详细图/流程图），每种模式的适用场景用 if-then 格式描述。借鉴：复杂 skill 的输出变体应当穷举列出，而不是让 agent 自由发挥。

2. **references/ 完整性** → 所有 3 个 references/ 文件（excalidraw-schema.md、syntax-rules.md、canvas-spec.md + layout-algorithms.md）均存在且被正确引用，无孤儿文件。借鉴：references/ 文件要和 SKILL.md 中的引用保持 1:1 对应。

3. **marketplace.json 注册** → 用 `.claude-plugin/marketplace.json` 列出 3 个 skill 路径，注册完整、无遗漏。借鉴：多 skill 仓库必须有集中的注册文件，且每次加 skill 必须同步更新。

4. **中英双语 README** → 面向不同用户群提供母语文档，降低上手门槛。借鉴：面向多语种用户的 skill 仓库值得提供 README_CN.md。

5. **无 hook/script/MCP 执行面** → 安全评分 CLEAR，原因就是没有任何可执行的表面。借鉴：纯 skill 仓库保持零执行面是最简单的安全策略。

### 3.3 当时的缺陷
1. **R01 模糊量词**：6 处"appropriate/balanced/sensible/subtle/Strategic"让 agent 无法把规则转化为可执行判断。根本原因：作者用自然语言描述设计直觉，而设计直觉对人是隐性的，对 agent 是无效指令。自查：我的 `claude-for-legal` 里 3 处"appropriate"是同类问题。

2. **R06 缺 JSON 示例**：`obsidian-canvas-creator` 的示例块只展示操作步骤（"Step 1: 分析用户需求"），不展示实际 JSON 格式。根本原因：作者把格式规范全部放进 references/canvas-spec.md，主文件认为"有引用就够了"——但 agent 执行时需要一个格式锚点来验证自己的输出。自查：我的某些 skill 也犯了"全靠 references/ 撑场面"的毛病。

3. **无 scope note**（R07，未在此仓库被扣分，但值得学）：三个 skill 没有明确说明"何时**不**用这个 skill"，也没有说明彼此的分工边界。如果用户同时安装了三个 skill，agent 在面对"画个图"的请求时可能无从选择。

### 3.4 当时的优化机会
1. 用测量标准替换 6 处模糊量词（最高价值，每处改动约 2 分钟）
2. `obsidian-canvas-creator` 添加最小 JSON 输出示例（约 10 分钟）
3. 添加 scope note 说明 3 个 skill 的选择规则

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| mermaid "appropriate/balanced/sensible" | `grep -n "appropriate\|balanced\|sensible" mermaid-visualizer/SKILL.md` | **持续**（第 17、24、197 行） | 作者未收到或未接受 PR，3 处模糊量词原样保留 |
| obsidian "appropriate/subtle/Strategic" | `grep -n "appropriate\|subtle\|Strategic" obsidian-canvas-creator/SKILL.md` | **持续**（第 62、68、74、200 行） | 同上，4 处全部保留 |
| obsidian 缺 JSON 示例 | `grep -c '^\`\`\`json' obsidian-canvas-creator/SKILL.md` | **持续**（0 个 JSON 代码块） | 示例段仍为步骤描述，未补充格式示例 |

全部 3 个过去缺陷**均持续存在**，无任何修复。这是有意保留还是维护者未意识到是问题，无从判断——但对学习者而言，这验证了"R01 模糊量词是工程师最常忽视的低优先级问题"。

### 4.2 架构演进
对比 Audit 当时记录与当前 HEAD：目录结构完全未变。没有新增 skill，没有重组目录，没有新增 AGENTS.md 或任何元文档。仓库处于"发布即稳定"状态——3 个 skill 满足了作者和用户的核心需求，无主动迭代动力。

这提示：对于"单点工具 skill"类仓库，初始质量决定长期质量。发布前应确保 R01/R06 已解决。

### 4.3 新增的可学习模式
暂无——当前 HEAD 与 Audit 快照相比无结构性变化。

---

## 五、校准

### 5.1 我已经在做对的
1. **references/ 完整性**：我的 `echo-sleuth/skills/jsonl-core/references/extraction-patterns.md` 存在且被正确引用，无孤儿
2. **单职责 skill**：我的 `claude-for-legal` 的每个 skill 只覆盖一种法律工作流，和 axtonliu 的三件套一样保持单一职责
3. **零执行面**：我的 `echo-sleuth` 和 `drama-workshop-skills` 均无 hook/script，安全评分可预期 CLEAR

### 5.2 挑战 / 验证
这次案例验证了一个我之前犹豫的做法：**把格式规范放进 references/ 而非主 SKILL.md**。

`excalidraw-diagram/SKILL.md` 得了 94 分且无示例——因为它有清晰的 references/excalidraw-schema.md 处理格式细节。但 `obsidian-canvas-creator/SKILL.md` 得了 89 分，同样有 references/canvas-spec.md，却因为**主文件的示例不展示格式**而被扣分。

结论：**references/ 放格式规范是对的，但主文件仍然需要至少一个格式示例**来让 agent 验证自己的输出方向。这是一个"主文件给方向，references/ 给细节"的分工。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的 skill 中的模糊量词（axtonliu 典型问题）
grep -rn -E '\b(appropriate|balanced|sensible|subtle|strategic|strategic)\b' \
  /tmp/my-repos/MarkQWu-claude-for-legal/ --include="SKILL.md"
```
命中后怎么办：对每一处用具体场景描述替换，例如"appropriate"→"当…时（给出条件）"。

```bash
# 检查我的 skill 是否有格式类输出示例（尤其是JSON输出的skill）
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" | xargs grep -L "^\`\`\`json" 2>/dev/null
```
命中后怎么办：若该 skill 需要输出结构化格式（JSON/YAML/特定DSL），加一个最小化示例。

### 6.2 灵感 → 实施路径
1. **想法**：为 `claude-for-legal` 的 `matter-workspace/SKILL.md` 增加 "Do NOT use" scope note
   - **为何可行**：legal skill 的边界不清楚时，agent 容易越权（例如把诉讼事项塞进合规事项 skill）
   - **第一步**：在 description frontmatter 里加"Do NOT use for criminal litigation matters"，约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **axtonliu/axton-obsidian-visual-skills 的核心目的**：为 Obsidian 用户提供 3 种可视化工具的 AI skill 适配
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 低 | 都是特定领域的多 skill 集合 | 领域不同（可视化 vs 短剧创作） | 低 |
| MarkQWu/claude-for-legal | 低 | 都有多个独立 skill，各自单职责 | 领域不同（可视化 vs 法律工作流） | 中 |
| MarkQWu/echo-sleuth-for-claude | 无 | — | 工具型 vs 知识型 skill | 低 |

我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| R01：模糊量词 "appropriate" | `grep -rn "appropriate" claude-for-legal/ --include="SKILL.md"` | **claude-for-legal 命中 3 处**（matter-workspace 3×，deposition-prep 1处，privilege-log 1处） | 低 |
| R06：格式类 skill 无 JSON 示例 | `grep -L '^\`\`\`json' claude-for-legal/**/*.md` | **claude-for-legal 命中**：大部分 skill 无 JSON 示例（但该领域主要输出文本，影响较低）| 低 |

**命中后的具体行动建议**：
- `claude-for-legal/*/skills/matter-workspace/SKILL.md` 的"appropriate"→改为"当事项同时涉及合同、诉讼、合规三类子议题时"，3 处约 10 分钟

### 8.3 别人的更优方案

1. **领域**：三种工具输出模式在 SKILL.md 中明确穷举
   - **axtonliu 做法**：`excalidraw-diagram/SKILL.md` 明确列出三种模式（Quick Sketch / Detailed / Flowchart），每种有触发条件
   - **我的现状**：`drama-workshop-skills/short-drama-remake/SKILL.md` 没有穷举"短剧的几种类型对应几种输出格式"
   - **如何借鉴**：在 short-drama-remake 里加一段"输出模式"章节，列出快速提纲/详细大纲/完整剧本三种模式及触发条件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：实际 input→output 示例的覆盖度
- **我的做法**：`drama-workshop-skills/short-drama/SKILL.md` 有 24 个代码块，每个关键操作都有 before/after 示例
- **axtonliu 做法**（弱在哪）：`obsidian-canvas-creator/SKILL.md` 示例只有步骤描述，无实际 JSON 输出
- **意义**：我的示例驱动风格在"格式精确度要求高"的 skill 领域是明显优势

---

## 八、术语表

### <a name="obsidian"></a>Obsidian
> 一款基于本地 Markdown 文件的知识管理笔记软件，支持插件扩展和双向链接。用户群体以知识工作者和研究者为主。Excalidraw、Mermaid、Canvas 都是 Obsidian 生态中的可视化工具插件。

### <a name="manifest"></a>marketplace.json / manifest
> 插件的"清单文件"，告诉 Claude Code 这个仓库包含哪些 skill，以及它们的路径。`.claude-plugin/marketplace.json` 是 axtonliu 选择的注册格式（与标准 `plugin.json` 类似但路径不同）。如果 manifest 里漏写了某个 skill，那个 skill 即使文件存在也不会被加载。

### <a name="excalidraw"></a>Excalidraw
> 一款开源的手绘风格白板工具，输出格式是 JSON。其 JSON schema 定义了节点、连线、样式等元素结构。

### <a name="mermaid"></a>Mermaid
> 一种文本描述图表的 DSL（领域特定语言），支持流程图、时序图、甘特图等。用 `graph LR` 这样的语法描述，由渲染器转为图形。
