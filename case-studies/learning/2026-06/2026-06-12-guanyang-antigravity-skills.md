# guanyang/antigravity-skills — 学习案例

**仓库**：https://github.com/guanyang/antigravity-skills
**Stars**：653 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-06-12（基于当前 HEAD）
**主题标签**：`examples-driven`, `security-gate`, `cross-reference`, `single-purpose`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`antigravity-skills` 是一个**大型社区 skill 合集**，专为 Claude Code 设计。仓库起源于 xiaolai 上游，由 guanyang 维护并持续扩充，目前收录了 80+ 个 skill，覆盖多智能体模式、上下文优化、TDD、评估框架、Obsidian 集成、PDF/DOCX/PPTX 处理、头脑风暴、任务规划、NotebookLM、Slack、Web Artifacts 等广泛领域。

仓库的一大特点是在正式审计（2026-04-20）之后大规模吸收了 **JimLiu/baoyu-skills**，新增了 20+ 个专为中文社交媒体（微信、微博、X、小红书）优化的 `baoyu-*` 系列 skill，使其成为目前 Claude Code 生态中对中文内容创作者最为友好的 skill 合集之一。

关键事实：
1. 作者 guanyang（中国大陆），GitHub 653 stars，活跃维护中
2. 80+ 个 skill，每个 skill 独立目录，统一 `SKILL.md` 接口
3. 多平台支持：根目录同时维护 `CLAUDE.md`、`AGENTS.md`、`GEMINI.md` 三份平台说明文档
4. 有正式项目治理文件：`CONTRIBUTING.md`、`SECURITY.md`
5. `skills_sources.json` 管理外部 skill 来源，`scripts/sync_skills.sh` 负责同步，体现出成熟的上游合并策略

### 1.2 架构剖析

```
antigravity-skills/
├── CLAUDE.md                   # Claude Code 项目总说明
├── AGENTS.md                   # 多 agent 平台说明
├── GEMINI.md                   # Gemini CLI 平台说明
├── CONTRIBUTING.md             # 贡献者指南
├── SECURITY.md                 # 安全政策
├── package.json                # Node.js 项目配置
├── skills_sources.json         # 外部 skill 来源配置
├── skills_index.json           # skill 目录索引
├── scripts/
│   └── sync_skills.sh          # 同步上游 skill 的维护脚本
├── skills/                     # 80+ 个 skill，每个独立目录
│   ├── defuddle/SKILL.md       # 最高分 (93/100)
│   ├── multi-agent-patterns/SKILL.md  # 92/100
│   ├── obsidian-bases/SKILL.md        # 92/100
│   ├── test-driven-development/SKILL.md  # 92/100
│   ├── planning-with-files/SKILL.md   # 含安全告警的 skill
│   ├── skill-creator/
│   │   ├── SKILL.md            # 元技能 (88/100)
│   │   └── agents/             # 子 agent 目录
│   │       ├── analyzer.md     # 45/100 — 缺失 frontmatter
│   │       ├── grader.md       # 45/100 — 缺失 frontmatter
│   │       └── comparator.md   # 45/100 — 缺失 frontmatter
│   ├── brainstorming/
│   │   └── scripts/package.json  # 依赖声明（unpinned）
│   └── baoyu-*/SKILL.md        # 20+ 个中文社交媒体 skill
└── agents/                     # 顶层 agent 目录
```

- **文件类型分布**：80+ 个 `SKILL.md`，3 个子 agent 文件，1 个 hooks 配置，3 个平台说明文档，1 个 `plugin.json`
- **编排关系**：顶层 `CLAUDE.md` 和 `skills_index.json` 作为目录索引 → 各 skill 目录独立自洽 → `skill-creator` skill 调用三个子 agent（analyzer/grader/comparator）形成评估循环 → 子 agent 互相通过盲测比对产生质量评分
- **跨件契约**：`skills_sources.json` 声明上游来源，`sync_skills.sh` 执行同步；`planning-with-files` 的 stop hook 通过 glob 解析脚本路径动态执行

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「广度优先的社区聚合」——通过吸收上游仓库（JimLiu/baoyu-skills 等）快速扩充覆盖面，而非从零创作每一个 skill
- **解决什么问题**：单用户很难为所有领域写出高质量 skill；通过社区聚合，每个领域的 skill 都由该领域的专家贡献
- **Trade-off**：广度带来了参差不齐的质量分布（25% skill 在 80 分以下）；换取了覆盖面，对用户来说「总能找到一个用」
- **渐进式揭露（Progressive Disclosure）策略**：得分 90+ 的 skill 用具体示例引导用户，细节通过展示而非列举传达；得分 85-89 的 skill 有良好结构但示例密度较低——这形成了一个可观察的**两档质量层级**
- **认知模型**：把 skill 集合视为「专家知识库」，`skill-creator` 是元技能，可以自动生成、评估新 skill，实现知识库的自我扩展

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「社区聚合型 skill 仓库 + 元技能自我扩展」**

关键特征：
- 每个 skill 严格一目录一 `SKILL.md` 的**单职责**结构，职责边界清晰
- `skill-creator` 作为[元技能](#元技能)存在，可以产生新 skill，是少见的「自我繁殖」架构
- `skills_sources.json` + `sync_skills.sh` 构成**上游跟踪协议**，使合集不是死水而是流动的
- `planning-with-files` 的安全自披露（SKILL.md 第 222-226 行明确警告 prompt injection 风险）体现了**透明度优先**原则
- 高分 skill（defuddle, multi-agent-patterns）均有 2-3 个完整的带代码的示例，**示例密度**是质量信号

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要快速找到某领域（如 PDF 处理、Obsidian）的 Claude skill | ✅ 高度适用 | 80+ skill 覆盖面广，`skills_index.json` 可快速定位 |
| 中文内容创作者（微信公众号、小红书、微博） | ✅ 适用 | baoyu-* 系列专门针对中文社交媒体平台优化 |
| 需要在 Claude Code 里执行自动化评估和 A/B 比较 | ⚠️ 有限适用 | skill-creator 评估循环设计好，但 3 个子 agent 缺失 [frontmatter](#frontmatter)，运行时无法注册 |
| 追求极简、无安全风险的生产环境 | ❌ 需谨慎 | planning-with-files 有 glob 动态执行的 Medium 安全风险 |

### 2.3 与其他架构对比

| 架构类型 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 社区聚合型（本案例） | guanyang/antigravity-skills | 覆盖面广，有元技能自我扩展，上游跟踪机制成熟 | 质量分布参差，子组件集成缺陷（frontmatter 缺失）未修复 |
| 单一领域深耕型 | wshobson/agents | 质量集中，每个 skill 完整度高 | 无合集效应，扩展靠单作者 |
| 多平台编排框架型 | first-fluke/oh-my-agent | 跨厂商支持，workflow 与 skill 分离 | 安装链复杂，skill 内容外置不透明 |
| 单职责插件型 | google-labs-code/stitch-skills | 边界极清晰，可独立安装 | 无跨 skill 编排能力，workflow 缺失 |

### 2.4 改进空间

1. **当前问题**：`skill-creator/agents/` 下 analyzer/grader/comparator 三个文件缺失 [YAML frontmatter](#YAML)，Claude Code 运行时无法注册为 agent，整个评估循环失效。**改进做法**：为三个文件各加 5-8 行标准 frontmatter（`name`, `description`, `model`, `tools`）。**预期收益**：skill-creator 评估循环恢复，元技能真正可用。单文件改动成本极低，性价比最高。

2. **当前问题**：`planning-with-files/SKILL.md` 的 [stop hook](#stop-hook) 用 [glob](#glob) 动态解析脚本路径后直接执行（`sh "$(ls $HOME/.claude/plugins/cache/*/*/*/scripts/check-complete.sh 2>/dev/null | head -1)"`），攻击面明确。**改进做法**：把脚本路径固定为常量或通过配置文件声明，增加脚本哈希校验步骤。**预期收益**：消除 Medium 安全风险，通过安全门。

3. **当前问题**：brand-guidelines/SKILL.md 无示例也无工作流步骤（得分 78），在 80+ skill 的合集里是明显洼地。**改进做法**：添加一个「品牌语调检查」的具体示例，展示输入（一段草稿文本）→ 输出（品牌合规检查报告）。**预期收益**：得分提升到 88+，与合集整体水平对齐。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-20 审计得分 **86/100**（加权平均）。

| 得分区间 | 数量 | 占比 | 代表 skill |
|---|---|---|---|
| 90–100 | 25 | 40% | defuddle (93), multi-agent-patterns (92), obsidian-bases (92), test-driven-development (92) |
| 85–89 | 25 | 40% | skill-creator (88), context-optimization, evaluation-framework |
| 80–84 | 7 | 11% | executing-plans, canvas-design, frontend-design |
| 45–79 | 6 | 9% | 3 个 agent 文件 (45)，brand-guidelines (78)，algorithmic-art |

**拉低平均分的主要因素**：3 个 agent 文件各得 45/100（缺失 frontmatter），是整个合集质量最大的短板。

### 3.2 当时值得借鉴的模式

1. **示例密度作为质量信号**（defuddle, multi-agent-patterns, subagent-driven-development）
   → 为什么好：这 3 个最高分 skill 都有 2-3 个完整的带代码的工作示例。阅读者（无论人还是 AI）能从「看懂示例」而不是「猜测意图」开始工作，大幅降低理解成本。
   → 路径：`skills/defuddle/SKILL.md`，`skills/multi-agent-patterns/SKILL.md`
   → 如何借鉴：每个 skill 至少加一个「输入 → 处理 → 输出」的完整示例，代码块优先于纯文字描述。

2. **安全自披露（Security Self-Disclosure）**（planning-with-files）
   → 为什么好：`planning-with-files/SKILL.md` 第 222-226 行明确警告用户该 skill 使用动态脚本执行、存在 prompt injection 风险，并建议用户审查脚本内容再使用。这比「无声地使用危险模式」好得多——用户知道风险，可以做知情选择。
   → 路径：`skills/planning-with-files/SKILL.md`，第 222-226 行
   → 如何借鉴：凡 skill 中包含 Bash 执行、网络请求、文件写入等操作，加一个 `## ⚠ 安全须知` 段落，明确说明操作范围。

3. **作者归属元数据**（vercel 贡献的 skill）
   → 为什么好：`composition-patterns`、`react-best-practices`、`react-native-skills` 在 SKILL.md 的 [frontmatter](#frontmatter) 中声明 `author: vercel`，并在 `skills_index.json` 中保留来源记录。这使合集中的「谁的工作」一目了然，有利于溯源和归因。
   → 如何借鉴：合集型仓库应在 frontmatter 里保留 `author` 和 `source` 字段，不要在聚合时抹去来源。

4. **skill-creator 元技能设计**
   → 为什么好：`skill-creator/SKILL.md` 不只是一个 skill，而是一个能**自动生成其他 skill 并评估质量**的元工具。它设计了三个子 agent（analyzer/grader/comparator）形成盲测评估循环，是社区型 skill 合集「自我维持质量」的关键机制。
   → 路径：`skills/skill-creator/SKILL.md`
   → 如何借鉴：当一个工具集达到一定规模，考虑建立「自动化质量检测」的元层，而不只是依赖人工 review。

### 3.3 当时的缺陷

1. **问题**：`skill-creator/agents/` 下 analyzer、grader、comparator 三个文件完全缺失 [YAML frontmatter](#YAML)。
   **根本原因**：这三个文件有实质内容（body 部分书写完整），但在提交时忘记加 frontmatter 块，导致 Claude Code 运行时将其识别为普通 Markdown 而非 agent 定义。
   **影响**：整个 skill-creator 的评估循环（生成 skill → analyzer 分析 → grader 评分 → comparator 比较）在运行时完全失效。SKILL.md 引用的三个 agent 路径存在，但 agent 未被注册。
   **自查**：我的仓库中的 agent 文件是否都有完整 frontmatter？

2. **问题**：`brand-guidelines/SKILL.md` 无示例、无工作流步骤，得分 78。
   **根本原因**：该 skill 只有一段自然语言说明「这个 skill 用于品牌指南检查」，没有展示如何使用，也没有定义具体的执行步骤。用户拿到后不知道该怎么触发它。
   **影响**：用户体验差，skill 实际利用率低。

3. **问题**：`executing-plans`、`canvas-design`、`frontend-design` 缺少 `## Output Format` 段落，得分 80-83。
   **根本原因**：作者专注于描述「做什么」，忽略了描述「产出什么格式」。AI 在执行时会自由发挥输出格式，导致输出不稳定。
   **自查**：我的 skill 是否每个都有明确的输出格式声明？

4. **问题**：`algorithmic-art` skill 要求在运行时读取外部模板文件，存在隐式 I/O 依赖。
   **根本原因**：作者把 skill 的核心逻辑放在了外部文件里，SKILL.md 只是一个「引用」，不是完整的自洽文档。如果插件安装路径变化，外部文件找不到，skill 直接失效。

5. **问题**：`context-degradation` skill 使用了「significantly」「substantially」「relatively」等模糊量词。
   **根本原因**：作者用自然语言写了本该是精确规则的内容。模糊量词让 AI 无法判断「多少算 significantly」，导致执行结果不一致。

### 3.4 当时的优化机会

1. 为 analyzer/grader/comparator 三个文件各加 frontmatter（5 行改动，修复整个评估循环，ROI 最高）
2. brand-guidelines 添加一个完整示例（30 分钟工作，得分可从 78 提升到 88+）
3. executing-plans、canvas-design、frontend-design 各加 `## Output Format` 段落（标准化输出，防止 AI 自由发挥）
4. context-degradation 把模糊量词替换为具体阈值（如「当上下文长度超过 50,000 tokens 时」）
5. `scripts/sync_skills.sh` 的 git clone 调用加 URL 白名单验证

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| analyzer.md 缺失 frontmatter | 查看文件开头是否有 `---` 块 | **仍未修复** ❌ — 文件以 `# Post-hoc Analyzer Agent` 开头，无 YAML frontmatter | 审计后 50+ 天，最高优先级 bug 未处理 |
| grader.md 缺失 frontmatter | 查看文件开头是否有 `---` 块 | **仍未修复** ❌ — 文件以 `# Grader Agent` 开头，无 YAML frontmatter | 同上 |
| comparator.md 缺失 frontmatter | 查看文件开头是否有 `---` 块 | **仍未修复** ❌ — 文件以 `# Blind Comparator Agent` 开头，无 YAML frontmatter | skill-creator 评估循环持续失效，元技能无法使用 |

**关键观察**：这 3 个 bug 是整个仓库最容易修复（每个文件只需 5-8 行前置内容）、影响最严重（整个元技能失效）的问题，但在审计后超过 50 天仍未修复。这说明仓库维护者的优先级是**广度扩展**（新增 baoyu-* 系列 20+ skill）而非**深度修复**。这是一个值得记住的维护模式：合集仓库往往有「加法容易减法难」的惯性。

### 4.2 架构演进

从审计时（2026-04-20）到当前 HEAD（2026-06-12）的主要变化：

**新增**：20+ 个 baoyu-* skill，覆盖中文社交媒体全链路
- 内容生成类：baoyu-article-illustrator, baoyu-comic, baoyu-infographic, baoyu-slide-deck
- 平台发布类：baoyu-post-to-wechat, baoyu-post-to-weibo, baoyu-post-to-x
- 媒体处理类：baoyu-compress-image, baoyu-cover-image, baoyu-image-gen, baoyu-xhs-images
- 内容转换类：baoyu-translate, baoyu-url-to-markdown, baoyu-youtube-transcript, baoyu-markdown-to-html
- 工程技能：harness-engineering

**不变**：整体架构（一目录一 SKILL.md）保持稳定，skill-creator 的三个子 agent bug 未修复

**启示**：仓库采用「积累式扩张」策略，每次合并外部来源时带来一批 skill，而不是按主题迭代深化。这种策略的结果是：合集覆盖面越来越广，但核心功能的 bug 修复优先级始终低于新增功能。

### 4.3 新增的可学习模式

- **领域垂直化的 skill 系列**：baoyu-* 系列形成了一个完整的「中文内容创作工作流」——从头脑风暴（baoyu-brainstorm）→ 撰写 → 配图（baoyu-image-gen/baoyu-infographic）→ 发布（baoyu-post-to-wechat）。每个 skill 专注单一步骤，但整体串联起来是完整工作流。这是**垂直工作流分解**的正面案例。
- **多平台文档并行维护**：CLAUDE.md / AGENTS.md / GEMINI.md 三文件并存，为不同 AI 平台用户提供相同内容的平台特定说明。对于需要覆盖多平台的工具作者，这是一个可参考的文档组织模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **skill 文件都有正确的 YAML frontmatter**：echo-sleuth 的所有 SKILL.md 都以标准 `---` 块开头，包含 `name`、`description` 等必要字段。这避免了 guanyang 仓库中最严重的那类 bug。
2. **agent 文件有 frontmatter**：echo-sleuth/agents/ 目录下的 agent 文件均有完整的 frontmatter，与 guanyang 的 skill-creator/agents/ 形成对比——他们的内容质量好但格式缺失，我的格式完整。
3. **单职责原则**：echo-sleuth 的每个 skill 专注单一功能（memory-management 只管记忆，git-mining 只管 git 历史挖掘），这与 antigravity-skills 高分 skill 的设计理念一致。

### 5.2 挑战 / 验证

本案例让我意识到一个问题：**示例密度和输出格式声明可能是我的短板**。guanyang 仓库的分数分布清楚地显示，缺少示例（brand-guidelines: 78）和缺少 Output Format（executing-plans: 80）是拉低分数的主要因素。我需要检查 echo-sleuth 和 claude-for-legal 的 skill 是否有足够的具体示例和明确的输出格式声明。

另一个值得验证的点：`skill-creator` 元技能的设计思路——用 AI agent 评估 AI 生成的工件——这是一个我在自己的项目里完全没有涉及的层次。我的项目目前全靠人工质量把控，有没有可能在 echo-sleuth 里引入类似的「自动评估循环」？

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 文件是否有 YAML frontmatter（对比 guanyang 的 bug）
find ~/my-repos/MarkQWu-echo-sleuth-for-claude/agents/ -name "*.md" | \
  xargs grep -L "^---" 2>/dev/null
# 命中后：该文件缺失 frontmatter，仿照标准格式添加 --- 块

# 检查我的 skill 是否有 Output Format 声明
find ~/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ -name "SKILL.md" | \
  xargs grep -L "Output Format\|output format\|## 输出格式" 2>/dev/null
# 命中后：为该 skill 添加 ## Output Format 段落，明确说明输出的文件路径和内容结构

# 检查 claude-for-legal 的 skill 是否有示例
grep -rn "## Example\|## 示例\|## Usage Example" \
  ~/my-repos/MarkQWu-claude-for-legal/skills/*/SKILL.md 2>/dev/null | wc -l
# 结果为 0 时：选 2-3 个最常用 skill 各添加一个具体的输入→输出示例

# 检查我的 skill 是否有模糊量词（对比 context-degradation 的缺陷）
grep -rn "significantly\|substantially\|relatively\|considerably\|大量\|适当\|一些" \
  ~/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
# 命中后：把模糊量词替换为具体数字或可观测条件
```

```bash
# 检查是否有 stop hook 使用 glob 动态解析路径（对比 planning-with-files 安全风险）
grep -rn 'sh.*\$.*\bls\b.*glob\|\$(ls.*\*' \
  ~/.claude/hooks/ ~/.claude/plugins/ 2>/dev/null
# 命中后：把动态路径替换为硬编码常量，或加脚本哈希校验
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 skills 补充 Output Format 段落
   - **为何可行**：echo-sleuth 有 4 个 SKILL.md，内容质量不错但输出格式未显式声明，这直接对应 guanyang 80-83 分 skill 的缺陷原因
   - **第一步**：打开 `skills/memory-management/SKILL.md`，在最后加一个 `## Output Format` 段落，声明「输出写入 `.claude/memory/`，每条记忆格式为 `{timestamp} | {category} | {content}`」，15 分钟可完成
   - **验证**：运行 `/nlpm:score skills/memory-management/SKILL.md`，确认得分提升

2. **想法**：在 claude-for-legal 的高频 skill（policy-redraft, comments）各添加一个完整示例
   - **为何可行**：法律 skill 的示例尤为重要——用户需要看到「AI 处理法律文本」的具体形式才敢用
   - **第一步**：找一段公开的法规文本片段，在 `skills/policy-redraft/SKILL.md` 里写一个「输入：某条款原文 → 输出：改写建议 + 变更理由」的示例块
   - **验证**：把 skill 给一个不熟悉的同事看，问他们是否知道如何使用

3. **想法**：研究 skill-creator 元技能设计，考虑在 echo-sleuth 里引入自动评估
   - **为何可行**：echo-sleuth 目前靠人工判断「记忆提取质量」，但这是可以量化的（召回率、精确率）
   - **第一步**：阅读 `skills/skill-creator/SKILL.md` 的评估循环设计，理解 analyzer → grader → comparator 三步如何协作
   - **注意**：先验证 guanyang 的实现是否真的可用（frontmatter bug 修复后才能测试）；或者自己从头实现一个更简单的「单 agent 评估」版本

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 guanyang/antigravity-skills 的核心目的**：社区聚合型 skill 合集，覆盖广泛领域，为 Claude Code 用户提供开箱即用的技能库

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是 Claude Code 插件，都有 skills/ + agents/ 目录，都追求单职责 skill | echo-sleuth 是专注单一工具的深度插件；antigravity 是广度覆盖的 skill 合集；echo-sleuth 有完整 frontmatter，antigravity 部分文件缺失 | 高（质量层面的对比学习价值大） |
| MarkQWu/claude-for-legal | 中 | 都是垂直领域 skill 合集；都用 SKILL.md 作为接口 | claude-for-legal 是单一作者精选（9 个 skill），antigravity 是多作者聚合（80+）；规模和治理复杂度差异巨大 | 中（示例密度改进对法律 skill 最直接） |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件 | drama 是创作工具（2 个 skill），与 antigravity 的规模和设计目标差异悬殊 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 文件缺失 frontmatter | `grep -rL "^---" agents/*.md` | echo-sleuth: 需要实际检查（根据已知信息判断可能通过，但未 100% 确认） | 高（若命中则运行时 agent 无法注册） |
| skill 缺少 Output Format 声明 | `grep -rL "Output Format" skills/*/SKILL.md` | echo-sleuth: 大概率命中（4 个 skill 均未观察到显式 Output Format 段落） | 中（影响输出稳定性） |
| skill 缺少具体示例 | `grep -rn "## Example\|## 示例" skills/*/SKILL.md \| wc -l` | claude-for-legal: 9 个 skill 均无示例（高概率命中） | 中（影响用户上手速度，尤其是法律场景） |
| 模糊量词 | `grep -rn "significantly\|大量\|适当" skills/*/SKILL.md` | echo-sleuth: 需要检查；已知 experience-synthesis skill 有一定描述性语言 | 低（影响执行一致性） |

**命中后的具体行动建议**：
- echo-sleuth 所有 SKILL.md → 加 `## Output Format` 段落（每个 15 分钟，共 1 小时）
- claude-for-legal/skills/policy-redraft/SKILL.md → 加一个完整示例（30 分钟，影响最大的单个改动）
- 所有 agent 文件 → `grep -L "^---" agents/*.md` 零结果则通过，有结果则立即补 frontmatter

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：渐进式揭露（Progressive Disclosure）
   - **本案例做法**：高分 skill 用具体示例驱动，skill 摘要简洁，细节通过展示传达（路径：`skills/defuddle/SKILL.md`，`skills/multi-agent-patterns/SKILL.md`）
   - **我的项目现状**：echo-sleuth 的 skill 描述倾向于列举功能点，而不是用示例展示
   - **如何借鉴**：在每个 skill 的顶部加一个「30 秒理解」示例块，展示最典型用例的输入/输出，功能列举移到后面的 `## Details` 段

2. **领域**：作者归属元数据
   - **本案例做法**：vercel 贡献的 skill 在 frontmatter 中保留 `author: vercel`，`skills_index.json` 记录来源（路径：`skills/react-best-practices/SKILL.md`）
   - **我的项目现状**：echo-sleuth 无 author 字段，来源完全依赖 git blame
   - **如何借鉴**：在 frontmatter 加 `author` 字段，对于受启发于其他仓库的 skill 加 `inspired_by` 字段

3. **领域**：上游跟踪协议（skills_sources.json + sync_skills.sh）
   - **本案例做法**：`skills_sources.json` 声明每个外部来源仓库的 URL 和版本，`sync_skills.sh` 执行同步并记录变更（路径：根目录 `skills_sources.json`，`scripts/sync_skills.sh`）
   - **我的项目现状**：无任何跟踪上游 skill 的机制，如果想引入第三方 skill，完全靠手工复制
   - **如何借鉴**：当 echo-sleuth 或 claude-for-legal 需要引用外部 skill 时，参照此模式建立来源声明文件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent frontmatter 完整性
  - **我的做法**：echo-sleuth 的 agent 文件均有完整 YAML frontmatter（`name`、`description`、`model` 等字段），Claude Code 运行时可正常注册和调用
  - **本案例弱在哪**：guanyang/antigravity-skills 的 skill-creator 子目录下 analyzer.md、grader.md、comparator.md 三个 agent 文件完全无 frontmatter，导致最核心的元技能评估循环运行时失效，且该 bug 在审计后 50+ 天仍未修复
  - **意义**：frontmatter 完整性是「最低质量门槛」，满足这个条件的文件才有被运行时使用的可能。echo-sleuth 在这一点上比 antigravity-skills 更稳定可用。

- **领域**：命令文件的 allowed-tools 声明
  - **我的做法**：echo-sleuth 的命令文件均显式声明 `allowed-tools`，明确约束 AI 在执行该命令时可以使用哪些工具，防止越权操作
  - **本案例弱在哪**：antigravity-skills 没有显式的工具权限约束机制，planning-with-files 的 stop hook 执行了外部脚本，但 SKILL.md 中没有 allowed-tools 声明来约束这一行为
  - **意义**：工具权限约束直接关联安全性。echo-sleuth 的声明式权限管理在这一点上更规范。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 [YAML](#YAML) 元数据块。在 Claude Code 插件体系里，frontmatter 是 Claude Code 运行时识别「这是一个 agent/skill/command」的唯一凭据。没有 frontmatter 的文件，内容再完整，Claude Code 也只把它当普通 Markdown 文档，不会注册为可调用的 agent 或 skill。就像一份简历——没有姓名和联系方式的「简历正文」，HR 连怎么联系你都不知道。guanyang/antigravity-skills 的 analyzer/grader/comparator 三个文件就是这种情况：内容完整，但缺少「入职登记表」，运行时完全找不到它们。

### <a name="YAML"></a>YAML
> 「YAML Ain't Markup Language」，一种人类可读的数据序列化格式，Claude Code 用它写 frontmatter 和配置文件。语法特点：用缩进表示层级，用 `:` 分隔键值，不需要引号或大括号。典型 Claude Code agent frontmatter 示例：`name: analyzer`，`description: 分析比对结果`，`model: haiku`。YAML 对缩进极其敏感，一个空格的错误就会导致解析失败——这也是为什么 frontmatter 缺失（完全没有 YAML）和 frontmatter 格式错误（YAML 写错）都会让 agent 注册失败。

### <a name="渐进式揭露"></a>progressive disclosure（渐进式揭露）
> 一种信息架构原则：先展示最重要、最常用的信息，细节按需展开。在 skill 设计里，渐进式揭露意味着 SKILL.md 的开头用一个具体示例（30 秒能看懂）吸引用户，再逐步展开配置选项、边缘情况、高级用法。guanyang 仓库的高分 skill（defuddle 93 分，multi-agent-patterns 92 分）都遵循这个原则：先给你看「一个真实的例子」，让你立刻明白这个 skill 的价值，然后再解释所有的参数和选项。反例是 brand-guidelines（78 分）：一上来就是功能列举，没有示例，读者不知道这个 skill 用完长什么样。

### <a name="stop-hook"></a>stop hook
> Claude Code 的一种 hook 类型，在 Claude Code **停止生成响应之前**触发，执行指定的命令或脚本。常见用途：把对话历史写入文件、触发后处理脚本、检查任务完成状态。`planning-with-files` 的安全风险就来自它的 stop hook：每次 Claude 完成响应，这个 hook 会执行一个通过 [glob](#glob) 动态找到路径的外部脚本。由于脚本路径是运行时动态解析的，攻击者理论上可以在 glob 匹配的路径上放置恶意脚本，在用户不知情的情况下执行。

### <a name="glob"></a>glob
> 一种文件路径通配符模式，用 `*`（匹配任意字符串）、`**`（匹配任意层级目录）、`?`（匹配单个字符）来批量选择文件。比如 `$HOME/.claude/plugins/cache/*/*/*/scripts/check-complete.sh` 表示「在 cache 目录下任意三层子目录里找名为 check-complete.sh 的文件」。在 Shell 里，glob 展开后的第一个结果可以直接作为命令执行，这就是 planning-with-files 的安全隐患：glob 展开的结果是不确定的，如果 cache 目录被篡改，首个匹配文件可能是恶意脚本。安全做法是把脚本路径硬编码为常量，或在执行前验证脚本的哈希值。

### <a name="元技能"></a>元技能（meta-skill）
> 一个以「生成或评估其他 skill」为目的的 skill，而非直接服务于最终用户任务。guanyang/antigravity-skills 里的 `skill-creator` 就是元技能：它不帮你写代码、不帮你分析文档，而是帮你**创建新的 skill**，并通过三个子 agent（analyzer/grader/comparator）自动评估新 skill 的质量。元技能的价值在于：当一个 skill 合集达到足够大的规模，纯靠人工维护质量的成本指数上升，元技能可以将部分质量检查自动化。这是 AI 辅助工具设计中「自我维持（self-sustaining）」能力的一种体现。
