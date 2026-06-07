# luongnv89/claude-howto — 学习案例

**仓库**：https://github.com/luongnv89/claude-howto
**Stars**：27,150 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-07（基于当前 HEAD）
**主题标签**：`model-pinning`, `vague-quantifier`, `cross-reference`, `template-design`, `security-gate`

**xiaolai 案例**：[../2026-04-25-luongnv89-claude-howto.md](../2026-04-25-luongnv89-claude-howto.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-howto 是目前 Claude Code 生态中 star 数最高的"How-To"指南仓库之一，审计时 28,667 颗星（现为 27,150，略有下降）。它以**多语言、多主题**的教学材料为核心，覆盖 vi/zh/uk/en 四个语言版本，将插件开发、PR 评审、DevOps 等进阶主题包装成可直接使用的 Claude Code 制品（命令 + agent + skill）。

关键事实：
- **规模**：117 个 NL 制品，分布于 commands/、agents/、skills/ 三层
- **多语言结构**：每一份英文模板在 vi（越南语）、zh（中文）、uk（乌克兰语）三个 locale 下各有镜像副本
- **渐进式策略**：仓库采用"progressive"学习策略，材料按难度递进排列（01-基础 → 07-插件）
- **教学定位**：明确面向初学者，大量制品是说明性存根（illustrative stub），而非生产级制品

xiaolai 将这个仓库的评分总结为「77 分——不是成绩，而是起跑线」，并将其文章命名为《The Textbook With Broken Pages: Auditing Claude Code's Most-Starred How-To Guide》。这个比喻精准：教科书的价值在于内容，而不是封底分数；但教科书里的印刷错误依然需要被发现和修正。

### 1.2 架构剖析

```
claude-howto/
├── vi/                         ← 越南语镜像
│   ├── 01-getting-started/
│   ├── ...
│   └── 07-plugins/
│       └── pr-review/
│           ├── commands/
│           │   └── review-pr.md        ← 定义5步但不派发
│           └── agents/
│               ├── security-reviewer.md   ← 曾含非法工具名 "diff"
│               ├── test-checker.md
│               ├── performance-analyzer.md
│               └── ...
├── zh/                         ← 中文镜像（同结构）
├── uk/                         ← 乌克兰语镜像（同结构）
├── en/                         ← 英文版（07-plugins/pr-review/）
├── scripts/
│   ├── build_epub.py           ← subprocess 含可控二进制路径（MEDIUM 安全风险）
│   ├── check_links.py          ← urlopen 接受 markdown 来源 URL（MEDIUM SSRF 风险）
│   ├── check_mermaid.py        ← 环境变量控制 --no-sandbox（MEDIUM + LOW）
│   ├── check_cross_references.py ← 路径遍历（已修复，PR #91）
│   ├── requirements.txt        ← 原全未钉版本（已修复，PR #90）
│   └── requirements-dev.txt    ← 仍为 floor-only 约束
└── CLAUDE.md
```

关键架构观察：

**多语言放大器效应**（xiaolai 首次命名）：仓库以单一模板为核心，向四个 locale 镜像。这一设计带来两个方向的杠杆：
- **效率杠杆**：一次写好 = 四倍内容，教学覆盖面自动放大
- **缺陷放大器**：一个模板错误 = 四个 locale 各有一个错误。`security-reviewer.md` 中的 `tools: diff` 非法标识符在 vi/zh/uk/en 四处同时出现，形成"1 bug × 4 locales = 4 bugs"的乘法效应

**跨组件断连**（cross-component disconnection）：`07-plugins/pr-review/commands/review-pr.md` 在文本中描述了5个步骤，并对应了5个子 agent（security-reviewer、test-checker、performance-analyzer 等），但命令本身从不派发（dispatch）到这些 agent。组件之间的关系存在于文档描述中，而不存在于可执行的编排逻辑中。

### 1.3 设计思路 / 方法论

**意图性质量梯度**（intentional quality gradient）是理解这个仓库的关键视角。

xiaolai 的审计揭示了一个重要的解读问题：如果把教学存根（tutorial stub）当成生产制品来评分，结果会系统性偏低。luongnv89 的 agent 缺少 model 声明、没有 examples、没有 output format——但这些都是教科书惯例的省略，而非作者的疏漏。一本教编程的书里的代码示例，不需要包含完整的错误处理和生产级优化。

然而，有些问题超越了"教学存根"的免责范围：
1. **工具标识符错误**（`diff` 而非 `bash`）是功能性 bug，教学文档同样不该出错
2. **依赖未钉版本**是可重复性风险，与内容定位无关
3. **路径遍历**是安全缺陷，不因教学目的而豁免

xiaolai 的`diff`缺陷洞察："这正是系统性扫描能发现而人工审阅容易遗漏的那类问题"——因为人工审阅会聚焦语义，而标识符合法性是机械的、不依赖上下文的检查。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式一：多语言模板镜像（Locale-Mirror Design）**

将单一英文模板通过目录结构镜像到多个语言版本。这是一个"内容放大器"架构，适合教学类仓库。

```
repo/
├── en/topic/artifact.md    ← 主模板
├── vi/topic/artifact.md    ← 越南语镜像
├── zh/topic/artifact.md    ← 中文镜像
└── uk/topic/artifact.md    ← 乌克兰语镜像
```

核心权衡：维护成本 vs. 覆盖范围。四份镜像意味着任何修复都需要在四处同步执行，否则镜像之间产生漂移（locale drift）。PR #89 在4分钟内同时修复四个 locale，说明维护者已建立了同步意识。

**模式二：渐进式难度分层（Progressive Depth Ladder）**

```
01-getting-started/    ← 入门
02-basic-commands/
03-workflows/
04-advanced-patterns/
...
07-plugins/            ← 最复杂（PR review + DevOps agent 系统）
```

用户可以从任意层级进入，命令自成体系，无强依赖。这是教学仓库的标准层次设计。

**模式三：跨组件描述-执行断连（Description-Execution Gap）**

这是一个反模式，但值得命名，因为它在教学类仓库中极为常见：命令文本描述了一个多步骤工作流（"步骤1、步骤2……步骤5"），并列举了参与的 agent，但命令本身的执行逻辑中没有 `dispatch` 或 `invoke` 指令。读者看到的是一幅架构图，而不是一套可运行的编排。

### 2.2 适用场景

- **Locale-Mirror**：适合受众明确分布在多个语言社区、且内容相对稳定的教学类仓库。不适合频繁迭代的生产插件（同步开销太高）。
- **Progressive Depth Ladder**：适合任何以学习曲线为设计目标的内容仓库。
- **Description-Execution Gap（反模式）**：在生产插件中应完全避免。在教学仓库中，如果存档作者明确标注"这是概念示例"，可以接受——但应该在文档中显式声明，否则读者会误以为可以直接执行。

### 2.3 与其他架构对比

| 维度 | luongnv89/claude-howto | gmickel/flow-next |
|------|------------------------|-------------------|
| 定位 | 教学指南（How-To） | 生产工作流插件 |
| 多语言 | 四语言镜像（vi/zh/uk/en） | 单语言（en），Codex 双路径 |
| 跨组件 | 描述-执行断连（反模式） | 三层完整编排（命令→agent→skill） |
| model 声明 | 全部缺失（教学存根） | 全部声明 |
| 依赖管理 | 修复后完整钉版（PR #90） | 未见 requirements.txt 安全问题 |
| 质量梯度 | 意图性存根 vs. 功能性 bug 混合 | 均匀高质量（87 制品均 ≥95）|

flow-next 证明了：即使在规模更大（87 vs. 117 制品）的情况下，完整的 frontmatter（含 model）和跨组件绑定是可以实现的。luongnv89 选择的是"教学优先"路径，两者定位不同，不能简单地用分数高低评判。

### 2.4 改进空间

1. **Locale 同步自动化**：当前 PR #89 的4分钟四 locale 修复依赖维护者的人工意识。可以引入 CI 检查，确保任何模板修改都同步到所有 locale（类似 flow-next 的 `sync-codex.sh`）。

2. **存根标注**：在教学性 agent/命令的 frontmatter 中添加 `stub: true` 字段或在文件头部的注释中显式声明"此为概念示例，不可直接执行"，避免读者产生错误期望。

3. **依赖安全**：`requirements-dev.txt` 仍为 floor-only 约束（如 `>=3.0`），建议统一为精确钉版，与已修复的 `requirements.txt` 保持一致。

4. **SSRF 与沙箱绕过**：`check_links.py` 的 urlopen 调用和 `check_mermaid.py` 的 `--no-sandbox` 环境变量控制属于结构性设计问题，修复需要重构相关函数。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

- **综合得分**：77/100 | **安全状态**：REVIEW（未达到 BLOCKED，但有7项发现）
- **制品总数**：117
- **Bug（PR 级）**：4
- **质量问题**：275
- **安全发现**：7

最差评分分布（按制品类型）：

| 制品类型 | 分布 | 得分 | 核心扣分项 |
|----------|------|------|------------|
| Documentation agents | 8 个 × 4 locale = 32 | 68/100 | 无 model，无 examples（-15），无 output format（-10），含模糊词 |
| PR-review + DevOps agents | 28 个 × 4 locale | 70/100 | 无 model，无 examples，无 output format |
| Commands | 44 个 × 4 locale | 73-75/100 | 无 allowed-tools（-5），无空输入处理（-10），无 output format（-10） |
| Brand-voice skills | 3 个 | 85/100 | 无 model，无 output format |
| 其余 skills | 20 个 | 95/100 | 仅缺 model 声明（-5） |

### 3.2 当时值得借鉴的模式

- **渐进式目录结构**：01→07 的难度阶梯清晰，是组织教学内容的优秀实践
- **多语言覆盖**：四个 locale 的系统性并行，在 Claude Code 生态中极为罕见，体现了对国际化社区的关注
- **Skill 质量**：20 个非品牌 skill 平均 95 分，frontmatter 结构完整，是仓库中的质量高地

### 3.3 当时的缺陷

**功能性 Bug（四处同步）**：

`vi/07-plugins/pr-review/agents/security-reviewer.md`（以及 zh/uk/en 三处镜像）在 frontmatter 中声明：

```yaml
tools: read, grep, diff
```

`diff` 不是合法的 Claude Code 工具标识符。合法的 Bash 工具名是 `bash`。这导致 agent 无法正确注册工具权限。由于 locale-mirror 设计，单一错误在四处同步复现。

**安全发现（7项）**：

- MEDIUM × 3：subprocess 含用户控制二进制路径（`build_epub.py` L291-309）；urlopen 接受 markdown 来源 URL（`check_links.py` L70，SSRF 风险）；环境变量控制 `--no-sandbox`（`check_mermaid.py` L56）
- LOW × 4：env 变量禁用 Chrome 沙箱（`check_mermaid.py` L30）；`requirements.txt` 6个包全未钉版；`requirements-dev.txt` floor-only 约束；`check_cross_references.py` L91 路径遍历（resolve() 无根约束）

**质量问题（275项）**：

- 36 个 agent：无 model（-5 each）、无 examples（-15 each）、无 output format（-10 each）
- 44 个命令：无 allowed-tools（-5 each）、无空输入处理（-10 each）、无 output format（-10 each）
- 跨组件：`review-pr.md` 描述5步骤对应5个子 agent，但从不派发
- 过度授权：`deployment-specialist` 和 `incident-commander` 声明了 `write` 和 `bash`，但它们是协调类 agent，不需要直接写文件或执行命令

### 3.4 当时的优化机会

1. 统一为所有 agent 和命令添加 `model` 字段，哪怕是教学存根也可以示范 frontmatter 的完整形态
2. 为命令添加 `allowed_tools` 和空输入处理（`if empty: echo "请提供..."` 模式）
3. 将 `diff` 修正为 `bash`（四处同步）
4. 钉定 requirements.txt 中的所有依赖版本
5. 约束 `check_cross_references.py` 的路径遍历到仓库根目录

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

**diff → bash 修复（PR #89，已验证）**

xiaolai 文章发布后4分钟内，维护者在同一天（2026-04-24）提交了 PR #89，将四个 locale 中的 `diff` 全部修正为 `bash`。当前 HEAD 中 `grep` 结果确认：

```
07-plugins/pr-review/agents/security-reviewer.md:
tools: read, grep, bash
```

四个 locale（vi/zh/uk/en）均已修复。这是 xiaolai 文章的最直接成果：一篇文章，4分钟，4处修复。

**requirements.txt 依赖钉版（PR #90，已验证）**

当前 `requirements.txt` 内容已从全部未指定版本改为精确钉版：

```
ebooklib==0.18
markdown==3.7
beautifulsoup4==4.12.3
httpx==0.28.1
pillow==11.1.0
tenacity==9.0.0
jinja2==3.1.4   ← 新增依赖，同时钉版
```

**路径遍历修复（PR #91，已验证）**

`check_cross_references.py` 的 `resolve()` 调用已添加根目录约束，路径遍历风险消除。

**仍然存在的问题**：

- **跨组件断连**：`review-pr.md` 依然不派发到子 agent。维护者未修复，可能是有意为之（教学设计决策）。
- **model 声明**：所有 agent/命令的 frontmatter 仍无 `model` 字段。这与教学仓库的定位一致——维护者优先修复功能性 bug，而非补全结构性 frontmatter。
- **SSRF 与沙箱风险**：`check_links.py` 和 `check_mermaid.py` 的结构性安全问题仍然存在。
- **requirements-dev.txt**：仍为 floor-only 约束。

**Issue #92**：xiaolai 文章撰写时，针对剩余问题开了 Issue #92，但审计时未见维护者回应。

### 4.2 架构演进

整体架构保持稳定。三项 PR 修复的都是局部问题（工具标识符、依赖钉版、路径遍历），未触及多语言镜像结构或跨组件设计。这说明维护者认为：

1. 当前架构（locale-mirror + progressive depth）是正确的长期选择
2. 功能性 bug 和安全问题优先于结构性质量（model 声明、examples、output format）
3. 跨组件断连是教学设计的一部分，不需要修复

star 数从 28,667 降至 27,150（-1,517）是一个值得注意的信号。"最多星"仓库的 star 下降通常与负面报道或生态内新竞争者出现有关，但幅度约 5% 属于正常波动范围。

### 4.3 新增的可学习模式

**快速响应循环**：xiaolai 发表文章 → 维护者4分钟内提交 PR #89/90/91 → 三项 PR 合并。这证明了高质量审计文章作为"外部 CI"的实际效果：发现问题 → 公开表达 → 维护者修复。对于 star 数高的仓库，这个循环可以在极短时间内完成。

**修复的选择性**：维护者修复了功能性 bug（diff/路径遍历）和可重复性风险（依赖钉版），但保留了结构性质量问题（model 声明、examples）和设计缺陷（跨组件断连）。这种选择性体现了一个务实的优先级判断：先保证"能运行"，再考虑"运行得好"。

---

## 五、校准

### 5.1 我已经在做对的

这一节用于校准：luongnv89 的案例哪些地方我的仓库已经做对了？

- **单语言（无 locale-mirror 复杂度）**：我的仓库（drama-workshop-skills、claude-for-legal、echo-sleuth-for-claude）都是单语言，避免了 locale-mirror 带来的同步维护负担。这是更简单、更可维护的选择，对于生产级插件是正确的。
- **生产定位而非教学定位**：echo-sleuth-for-claude 是生产插件，不是教学存根，这意味着"model 声明缺失"对我的仓库是真实缺陷，而非可接受的教学省略。

### 5.2 挑战 / 验证

- **model 字段缺失**：echo-sleuth-for-claude 的4个 SKILL.md 文件均无 model 字段（grep 已确认）。同样问题在 drama-workshop-skills（2个 SKILL.md）和 claude-for-legal（多个文件）中也存在。这与 luongnv89 的 skill 得分 -5/制品 是完全相同的扣分模式。
- **依赖钉版**：claude-for-legal 有 `scripts/` 目录，其 requirements 文件是否已钉版未知。luongnv89 的 PR #90 修复提醒我检查这一点。
- **跨组件断连**：我的 echo-sleuth-for-claude 是否存在"命令描述了 agent 参与但不派发"的情况？需要专门检查。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查所有 SKILL.md 是否有 model 字段
grep -r "^model:" \
  /path/to/echo-sleuth-for-claude \
  /path/to/drama-workshop-skills \
  /path/to/claude-for-legal \
  --include="*.md" -l

# 如果无输出，说明所有 SKILL.md 均缺 model 字段（与 luongnv89 skill -5 问题相同）

# 检查命令文件是否有 allowed_tools 字段
grep -r "^allowed_tools:" \
  /path/to/echo-sleuth-for-claude/.claude/commands \
  --include="*.md" -l

# 检查是否有空输入处理
grep -r "if.*empty\|if.*\$1.*==" \
  /path/to/echo-sleuth-for-claude/.claude/commands \
  --include="*.md"

# 检查 scripts/ 目录的依赖文件是否钉版
cat /path/to/claude-for-legal/scripts/requirements.txt 2>/dev/null
# 检查是否含 "==" 精确版本，或仅有 ">=" floor-only 约束

# 检查 echo-sleuth 命令是否存在跨组件断连
grep -r "dispatch\|invoke\|agent" \
  /path/to/echo-sleuth-for-claude/.claude/commands \
  --include="*.md" -A2
```

### 6.2 灵感 → 实施路径

**行动1：为所有 SKILL.md 补充 model 字段**

优先级：高。echo-sleuth-for-claude 的4个 SKILL.md 每个缺 model 字段会扣 5 分（共 -20）。修复方式：在每个 SKILL.md frontmatter 中添加：

```yaml
model: claude-sonnet-4-5   # 或当前推荐 Sonnet 版本
```

**行动2：钉定所有 scripts/ 依赖版本**

参考 luongnv89 PR #90 的修复模式。检查 claude-for-legal/scripts/ 下的 requirements 文件，将所有 `>=x.y` 约束改为 `==x.y.z` 精确钉版。

**行动3：建立 locale-drift 防护意识**

如果未来在任何仓库中引入多语言内容，建立 CI 检查脚本，确保所有 locale 中的同名文件保持同步。可以参考 flow-next 的 `sync-codex.sh` 思路，将"模板 → 多目标"的同步自动化。

**行动4：在命令中显式绑定 agent**

对于 echo-sleuth-for-claude 中存在"描述了 agent 参与"的命令，检查是否包含实际的 dispatch 指令。如果是概念示例，添加注释：`<!-- 注：此为架构示意，实际执行时需取消下方 dispatch 注释 -->`。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

| 我的仓库 | 定位 | 与 luongnv89 对齐度 |
|---------|------|---------------------|
| MarkQWu/drama-workshop-skills | 生产插件（创作工具） | 低——luongnv89 是教学指南，drama-workshop 是生产工具；定位不同 |
| MarkQWu/claude-for-legal | 生产插件（法律工作流） | 低——同上，生产 vs. 教学 |
| MarkQWu/echo-sleuth-for-claude | 生产插件（会话挖掘） | 低——生产定位，但架构规模（4 SKILL.md + agents + commands）与 luongnv89 的 skill 子集相近 |

目的对齐度低意味着：不应照搬 luongnv89 的"教学存根"设计决策，但技术缺陷（model 缺失、依赖未钉版）是跨定位的共同问题，值得借鉴修复方式。

### 8.2 在我的项目里复现的同类问题

**问题1：model 字段缺失（与 luongnv89 skills -5/个 完全相同）**

- echo-sleuth-for-claude：4个 SKILL.md，无一含 `model:` 字段（grep 已确认）
- drama-workshop-skills：2个 SKILL.md（short-drama.md, short-drama-remake.md），无 model 字段
- claude-for-legal：多个 SKILL.md，model 字段状态待确认，但模式与前两者一致

luongnv89 的每个 skill 因此扣 5 分，最终 skill 平均 95 分。我的仓库中，每个缺失 model 字段的 SKILL.md 都面临相同的 -5 扣分。

**问题2：依赖管理未钉版（与 luongnv89 PR #90 前的状态相同）**

claude-for-legal 有 `scripts/` 目录，其依赖文件的钉版状态未知。luongnv89 的案例说明，即使是脚本工具类依赖，全未钉版也是可被发现和修复的安全/可重复性问题。

**问题3：跨组件引用的完整性**

echo-sleuth-for-claude 含 agents 和 commands。命令是否真正 dispatch 到 agent，还是仅在文本中描述了 agent 的参与？这与 luongnv89 的 review-pr.md 断连问题同类。

### 8.3 别人的更优方案

**luongnv89 PR #90 的修复模式**（依赖精确钉版）是一个可以直接复制到我的仓库的实践：

```
# luongnv89 修复后的 requirements.txt 风格：
ebooklib==0.18
markdown==3.7
beautifulsoup4==4.12.3
httpx==0.28.1
pillow==11.1.0
tenacity==9.0.0
jinja2==3.1.4
```

对比"floor-only"风格（`httpx>=0.20`），精确钉版确保了跨环境可重复安装，避免上游包更新引入的破坏性变更。

**locale-mirror 的结构化思路**虽然维护成本高，但对国际化受众的覆盖思路是值得学习的。即使我的仓库不采用四语言镜像，在文档层面提供英文 README 也是类似的扩展思路。

### 8.4 反向：我的项目做得比他们好的地方

- **无 locale-drift 风险**：我的仓库是单语言，不存在跨 locale 的同步维护负担，也不会出现"1 bug × 4 = 4 bugs"的乘法效应。
- **生产定位的完整性期望**：我的仓库不以"教学存根"为借口省略 frontmatter 字段。一旦修复 model 字段缺失，SKILL.md 的完整性将超过 luongnv89 的 agent 制品（68-70 分区间）。
- **规模更小、更聚焦**：echo-sleuth-for-claude 的 4 个 SKILL.md 远比 luongnv89 的 117 个制品容易维护，质量一致性更容易保证。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **frontmatter** | Markdown 文件头部的 YAML 元数据块，以 `---` 包围。Claude Code 的 agent/命令/skill 使用 frontmatter 声明 model、tools、description 等结构化属性。 |
| **model declaration** | frontmatter 中的 `model:` 字段，显式指定该制品应使用的语言模型（如 `claude-sonnet-4-5`）。缺失时 Claude Code 使用默认模型，NLPM 扣 5 分/制品。 |
| **allowed-tools** | 命令 frontmatter 中的 `allowed_tools:` 字段，声明命令可以使用的工具列表（如 `bash, read, write`）。缺失时 NLPM 扣 5 分/命令。 |
| **tool identifier** | Claude Code 工具系统中的合法工具名，如 `bash`、`read`、`write`、`grep`。`diff` 不是合法工具名，会导致工具注册失败。luongnv89 的 `security-reviewer.md` 最初错误使用了 `diff`。 |
| **SSRF** | Server-Side Request Forgery（服务端请求伪造）。攻击者通过控制服务器发起的 HTTP 请求，访问内网资源或执行侦察。`check_links.py` 中 urlopen 接受 markdown 来源 URL 是此类风险的来源。 |
| **path traversal** | 路径遍历攻击。通过构造包含 `../` 的路径，使程序访问预期目录之外的文件。`check_cross_references.py` 的 `resolve()` 调用在修复前缺少根目录约束，存在此风险。 |
| **locale-mirror** | 本案例的核心架构模式。将同一套 NL 制品（commands/agents/skills）以目录结构复制到多个语言版本（locale）下，以覆盖不同语言社区。 |
| **locale drift** | locale-mirror 架构中，当一个 locale 的内容被修改但其他 locale 未同步时，产生的内容不一致状态。 |
| **progressive depth ladder** | 渐进式难度阶梯。通过编号目录（01→07）将内容按学习难度组织，用户可从任意层级进入。 |
| **description-execution gap** | 跨组件断连的具体表现：命令文本描述了多步骤工作流和参与的 agent，但命令执行逻辑中没有实际的 dispatch 指令。 |
| **stub / tutorial stub** | 教学存根。仅用于说明概念而非实际执行的 NL 制品，通常省略生产级所需的 frontmatter 字段（model、examples、output format）。 |
| **floor-only constraint** | 依赖版本约束中只指定下限（如 `>=3.0`）而不指定精确版本，导致不同时间安装可能获得不同版本，破坏可重复性。 |
| **locale-amplifier effect** | xiaolai 命名的效应：locale-mirror 设计中，单一模板缺陷会在所有 locale 中同步复现，形成"1 bug × N locales = N bugs"的乘法效应。 |
