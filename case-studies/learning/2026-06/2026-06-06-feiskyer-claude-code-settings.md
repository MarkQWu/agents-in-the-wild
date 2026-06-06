# feiskyer/claude-code-settings — 学习案例

**仓库**：https://github.com/feiskyer/claude-code-settings
**Stars**：1434 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-06-06（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `model-pinning`, `security-gate`, `single-purpose`, `experience-accumulation`

**xiaolai 案例**：[../../2026-04-25-feiskyer-claude-code-settings.md](../../2026-04-25-feiskyer-claude-code-settings.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

feiskyer/claude-code-settings 是一个「全能技能集合库」，由 Kubernetes/云原生领域知名开发者 feiskyer 维护，在 Claude Code 生态中以 1434 stars 高居前列。仓库将作者日常开发所需的 AI 辅助工具全部集中管理：15 个 skill、10 个 agent（7 根级 + 3 个子 agent）、多个 hook 和 manifest，并有完整的 plugin 嵌套结构。

关键数字：
- 45 个可识别工件，35 个 NL 内容文件，NLPM 综合评分 **85/100**
- 安全评级：**CLEAR**（Medium×2，Low×2，无 Critical/High）
- 技能涵盖：代码审查、技能创建、自主循环执行、规格驱动开发、深度研究、图像生成、翻译等
- 生态定位：介于「个人工具箱」和「社区参考实现」之间，因高 star 数事实上承担了后者角色

### 1.2 架构剖析

**目录结构**：

```
skills/                          # 15 个 skill
  skill-creator/                 # 多 agent 流水线
    SKILL.md
    agents/analyzer.md, grader.md, comparator.md
  autonomous-skill/              # 循环执行器
    SKILL.md
    scripts/run-session.sh
    hooks/stop-hook.sh
  spec-kit-skill/
    SKILL.md
    scripts/detect-phase.sh
    helpers/detection-logic.md   # 审计时缺失，当前 HEAD 已补齐
  deep-research/, codex-skill/, github-fix-issue/, github-review-pr/
  reflection/, command-creator/, kiro-skill/, eureka/
  nanobanana-skill/ (+ nanobanana.py)
  youtube-transcribe-skill/, translate/, gpt-image-skill/ (+ gpt_image.py)

agents/                          # 7 个根级 agent
  instruction-reflector.md, pr-reviewer.md, command-creator.md
  github-issue-fixer.md, insight-documenter.md, ui-engineer.md, deep-reflector.md

hooks/hooks.json                 # Stop hook（在 3 处重复）
.claude-plugin/plugin.json + marketplace.json
plugins/                         # 6 个嵌套插件（含 kiro 命令×5）
scripts/update-cc-plugins.sh
settings/ (deepseek, minimax, vertex JSON)
.mcp.json                        # chrome-devtools-mcp
guidances/ (github-copilot.md, llm-gateway-litellm.md)
```

**文件类型分布**：15 个 SKILL.md，7+3 个 agent，5 个 kiro 命令，hook×3 处，manifest×7，Python×11

**编排关系**：skill-creator 是仓库内最复杂的编排——skill（入口）→ comparator（盲测比较）→ analyzer（结果分析）→ grader（打分）形成三阶段流水线。其余 skill 与根级 agent 相互独立，没有跨 skill 的依赖关系。

**跨件契约**：kiro-skill 引用 `helpers/kiro-identity.md` 和 `helpers/workflow-diagrams.md`（仍未找到）；spec-kit-skill 引用的 `helpers/detection-logic.md` 在审计时缺失，但当前 HEAD 已补齐。plugins/ 目录镜像了 6 个 skill（双部署模式）。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「一仓多能，按场景触发」——不同于「单仓多插件」的横向扩展，feiskyer 采用纵向深度：每个 skill 有明确的单一用途（translate 只做翻译，deep-research 只做研究），但整体仓库覆盖从 UI 开发到云原生运维的完整开发链路。

**解决什么问题**：熟练开发者在不同工作场景下需要不同认知模式的 AI 辅助——审查 PR 需要批判思维，实现 UI 需要创造思维，调试需要诊断思维。这个仓库为每个认知场景定制了一个专用 agent/skill，避免通用 prompt 的「认知漂移」。

**做了什么 trade-off**：专用优于通用（每个工件单一职责，但数量多维护成本高）；快速迭代优于质量对齐（持续新增 skill，但 model 字段、examples 字段等系统性补全工作被推后）；功能优先于安全细节（autonomous-skill 为了功能完整性使用了 `bypassPermissions`，作者认为这是合理取舍）。

**双部署模式的认知**：skills/ 和 plugins/ 各维护一份 kiro-skill/autonomous-skill/codex-skill/nanobanana-skill/spec-kit-skill/youtube-transcribe-skill，这是刻意的——前者是「本地优先」快速访问，后者是「插件市场」标准包装。这个模式增加了维护成本，但满足了不同部署场景的需求。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「认知场景专用化」单仓多技能模式**：每个 skill/agent 对应一个具体的开发认知场景，整体仓库覆盖完整的开发工作流。

模式特征清单：
- 特征 1：每个工件有极其明确的单一目的（translate = 翻译，github-fix-issue = 修 issue），无功能重叠
- 特征 2：最复杂的场景（skill-creator）用多 agent 流水线处理，其余场景用单一 skill/agent
- 特征 3：skills/ 和 plugins/ 双轨部署，分别服务本地使用和市场分发
- 特征 4：Python 脚本（nanobanana.py、gpt_image.py）嵌入 skill 目录，技能可以调用外部工具
- 特征 5：hooks.json 作为全局拦截器，stop-hook.sh 与 autonomous-skill 协同实现循环控制

### 2.2 适用场景

| 场景 | 是否适用 | 原因 |
|---|---|---|
| 个人全栈开发者，覆盖多种开发任务 | ✅ 高度适用 | 专用化设计减少 prompt 切换认知负担 |
| 团队统一 AI 工具配置 | ✅ 适用 | 可选择性安装特定 skill/plugin |
| 需要复杂多步骤任务自动化 | ✅ 适用 | skill-creator 流水线模式可复制 |
| 单一技术栈专注团队 | ⚠️ 过重 | 15 个 skill 多数无关，维护成本高于收益 |
| 对安全有严格要求的企业环境 | ⚠️ 谨慎 | bypassPermissions 使用需要明确风险评估 |

### 2.3 与其他架构对比

| 架构类型 | 代表性仓库 | 核心差异 | 适用场景 |
|---|---|---|---|
| 认知场景专用化（本案例） | feiskyer/claude-code-settings | 纵深覆盖，每场景一工件 | 全栈开发者个人工具集 |
| 技术栈横向覆盖 | fcakyon/claude-codex-settings | 按技术栈分插件，25+ domain | 多技术栈团队共享 |
| 单一领域深耕 | 领域专用 plugin | 一个 plugin，多个配合工件 | 特定场景深度集成 |

### 2.4 改进空间

**改进点 1：系统性补全 model 字段**
- 当前问题：7 个根级 agent 均无 `model:` 字段，导致 Claude Code 使用默认模型，无法利用 Haiku（成本优化）或特定版本
- 改进方式：在每个 agent frontmatter 中根据任务复杂度选择模型（简单任务用 haiku，复杂用 sonnet）
- 预期收益：约 -5 分罚分消除，每 agent +5 分；对用户而言，明确模型选择减少意外成本

**改进点 2：消除三处重复 hooks.json**
- 当前问题：hooks.json 在仓库根、hooks/ 目录、plugins/ 下各有一份（内容相同），维护时需同步更新三处
- 改进方式：统一到 hooks/hooks.json，其余用软链接或统一安装脚本引用
- 预期收益：减少「改了一处忘了另外两处」的维护风险

**改进点 3：补齐 kiro-skill 的缺失 helper 引用**
- 当前问题：kiro-skill/SKILL.md 引用 `helpers/kiro-identity.md` 和 `helpers/workflow-diagrams.md` 但文件不存在
- 改进方式：要么创建缺失文件，要么从 SKILL.md 中删除引用
- 预期收益：消除跨组件一致性问题，NLPM check 不再报 broken ref

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

| 文件 | 得分 | 主要扣分原因 |
|---|---|---|
| skill-creator/agents/analyzer.md | 45/100 | 缺少 name + description frontmatter（-55 分） |
| skill-creator/agents/grader.md | 45/100 | 同上 |
| skill-creator/agents/comparator.md | 45/100 | 同上 |
| ui-engineer.md | 70/100 | 无 model(-5)，无 examples(-15)，vague×5(-10) |
| 其余 6 个根级 agent | 80/100 各 | 无 model(-5)，无 examples(-15) |
| skill-creator/SKILL.md | 80/100 | vague×10+，上限罚 -20 |
| kiro 命令×5 | 85/100 | 无 allowed-tools(-5)，无空输入处理(-10) |
| command-creator/SKILL.md | 88/100 | vague×6(-12) |
| 多数其他 skill | 92-98/100 | vague×2-4(-4 至 -8) |
| translate/SKILL.md | 100/100 | 无扣分 |
| hooks/manifests | 100/100 | 无扣分 |

**综合评分：85/100**（45 工件，35 NL 内容文件）

### 3.2 当时值得借鉴的模式

**模式 1：三阶段盲测流水线（skill-creator）**
- 为何有效：comparator 执行盲测（不知道哪个是 A 哪个是 B），analyzer 事后解盲分析，grader 给出量化评分。三步分离消除了评判偏见，是少见的严谨多 agent 设计
- 原始路径：`skills/skill-creator/SKILL.md`，`agents/analyzer.md`，`agents/comparator.md`，`agents/grader.md`
- 如何借鉴：凡是需要「对比两个方案」的场景（代码审查、文档版本比较），都可以用这个模式

**模式 2：translate/SKILL.md 的极简完美**
- 为何有效：description 触发词完整（中英日三语触发条件），正文结构清晰，无 vague 量词，得分 100/100
- 原始路径：`skills/translate/SKILL.md`
- 如何借鉴：单一明确功能的 skill 应该能写出 100 分——description 覆盖所有触发场景，正文避免「尽量」「适当」等模糊词

**模式 3：autonomous-skill 的自主循环设计**
- 为何有效：run-session.sh 启动子进程，stop-hook.sh 作为终止条件，两者配合实现「Claude 自主运行直到任务完成」的循环控制
- 原始路径：`skills/autonomous-skill/scripts/run-session.sh`，`hooks/stop-hook.sh`
- 如何借鉴：需要长时间自主任务时，hook + script 的组合比单纯提示词更可靠

**模式 4：双部署镜像（skills/ + plugins/）**
- 为何有效：开发者本地测试用 skills/ 目录（快速修改），发布到市场用 plugins/ 目录（有标准 manifest）
- 如何借鉴：如果同时维护「本地用」和「发布用」两个版本，可以建立同步脚本（scripts/update-cc-plugins.sh）

### 3.3 当时的缺陷

**缺陷 1：子 agent frontmatter 完全缺失**
- 问题描述：skill-creator 下的三个子 agent 没有 `name` 和 `description` 字段，NLPM 罚分 -55，评分仅 45/100
- 根因：子 agent 被当作「内部实现细节」而非独立工件来对待，忽略了 Claude Code 需要 frontmatter 来正确调用子 agent
- 自查方法：`grep -L "^name:" skills/*/agents/*.md agents/*.md`

**缺陷 2：7 个根级 agent 无 model 字段**
- 问题描述：所有根级 agent 的 frontmatter 都缺少 `model:` 字段，无法控制模型选择，每个 agent 扣 5 分
- 根因：作者习惯「让 Claude Code 自动选择」，没有意识到显式指定是最佳实践
- 自查方法：`grep -L "model:" agents/*.md`

**缺陷 3：skill-creator/SKILL.md 模糊量词过多**
- 问题描述：用了 10+ 个「尽量」「适当」「合适」等不可验证的描述词，vague 罚分上限 -20
- 根因：skill 描述复杂工作流时倾向于用「灵活」表达，但对 LLM 来说这些词缺少执行标准
- 自查方法：`grep -c "尽量\|适当\|合理\|必要时\|可能" skills/skill-creator/SKILL.md`

### 3.4 当时的优化机会

1. **最高 ROI**：给三个子 agent 补 frontmatter（各仅需 2 行，但修复 -55 分罚分，立即提升整体评分）
2. **系统性清理**：给 7 个根级 agent 统一加 `model:` 字段（7 个文件，每个 +5 分）
3. **kiro 命令补全**：给 5 个 kiro 命令加 `allowed-tools` 和空输入处理（中等工作量，消除 -15 分罚分）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 验证方法 | 当前状态 | 意义 |
|---|---|---|---|
| analyzer.md 无 frontmatter | `head -5 skills/skill-creator/agents/analyzer.md` | ✅ **已修复**，name + description 已补齐 | 作者响应了最高优先级修复 |
| grader.md 无 frontmatter | `head -5 skills/skill-creator/agents/grader.md` | ✅ **已修复**，name + description 已补齐 | 同上 |
| comparator.md 无 frontmatter | `head -5 skills/skill-creator/agents/comparator.md` | 需验证（与上两个同批修复） | 可能已修复 |
| 7 个根级 agent 无 model 字段 | `grep -L "model:" agents/*.md` | ❌ **仍存在**，全部 7 个 agent 均无 model 字段 | 系统性问题，作者未优先处理 |
| spec-kit-skill helpers/detection-logic.md 缺失 | `ls skills/spec-kit-skill/helpers/` | ✅ **已修复**，文件现已存在 | 跨组件一致性改善 |
| kiro 命令 allowed-tools 缺失 | 需深入 plugins/ 目录检查 | 待验证（数据不足） | 不影响本案例结论 |

### 4.2 架构演进

**发生了什么变化**：
- 最高优先级的 bug（子 agent frontmatter 缺失，-55 分）已在审计后修复
- 新增了 `helpers/detection-logic.md`，补全了 spec-kit-skill 的跨组件引用
- 根级 agent 的 model 字段问题未处理（7 个 agent 均无 model）

**作者的学习轨迹**：
- 作者明显优先处理了影响功能正确性的问题（frontmatter 缺失会导致子 agent 无法被正确调用）
- 「非功能性」质量问题（model 字段、examples）被推后，印证了「配置即代码」心态——够用就好
- helpers 文件的补齐说明作者在持续完善跨组件引用的完整性

### 4.3 新增的可学习模式

当前 HEAD 中，`helpers/detection-logic.md` 的存在提供了一个值得学习的模式：**将复杂检测逻辑从主 SKILL.md 中分离到 helpers/ 子目录**。这样 SKILL.md 保持简洁（只描述意图），具体的检测规则放在 helper 文件中（详细、可维护）。这是「关注点分离」在 NL 工件设计中的体现。

---

## 五、校准

### 5.1 我已经在做对的

1. **工件单一职责**：echo-sleuth-for-claude 中每个 agent 有明确职责（analyze.md 专注分析，recall.md 专注记忆查询），与 feiskyer 的专用化设计一致
2. **manifest 完整性**：我的 plugin.json 包含完整的 name/description/version，与 feiskyer 的 hooks/manifests 得分 100/100 做法一致
3. **helpers 目录分离**：已有将辅助内容从主文件分离的意识（参考 feiskyer spec-kit-skill 的 helpers/detection-logic.md 模式）
4. **commands 中使用 argument-hint**：echo-sleuth 的命令已有参数提示，比 feiskyer 的根级 agent 在用户引导上做得更好

### 5.2 挑战 / 验证

**挑战**：本案例挑战了「model 字段是可选的」这一假设。feiskyer 7 个 agent 均无 model 字段且在生产中正常运行，说明 Claude Code 确实有合理的默认行为。但从可控性和成本优化角度，显式指定 model 仍是最佳实践——这是「可以省略」vs「应该不省略」的区别。

**验证**：三阶段盲测流水线（skill-creator 模式）验证了「多 agent 分工的正确方式是按认知阶段而非按功能模块分工」——comparator 做盲测，analyzer 做分析，grader 做量化，三者的分工依据是「认知操作类型」，不是「工具调用类型」。这个验证对我设计 echo-sleuth 的 memory-auditor + schema-scout 分工很有参考价值。

---

## 六、行动

### 6.1 自查动作

**检查所有 agent 是否有 model 字段**：
```bash
grep -rL "model:" agents/*.md skills/*/agents/*.md 2>/dev/null
# 若有输出 → 为每个缺失的文件添加 model: claude-sonnet-4-5 (复杂任务) 或 model: claude-haiku-4-5 (简单分类)
```

**检查所有 NL 工件是否有 vague 量词**：
```bash
grep -rn "尽量\|适当\|合理\|必要时\|可能\|适量\|相关" skills/ agents/ commands/ 2>/dev/null
# 每处匹配 → 替换为具体的可验证描述（如「尽量简短」→「不超过 3 句话」）
```

**检查跨组件引用完整性**：
```bash
grep -rn "helpers/" skills/ | grep -v "^Binary" | while read line; do
  file=$(echo "$line" | cut -d: -f2 | tr -d '"' | xargs)
  [ -f "$file" ] || echo "BROKEN REF: $file in $line"
done
# 若有 BROKEN REF 输出 → 创建缺失文件或删除 SKILL.md 中的引用
```

**检查子 agent frontmatter 完整性**：
```bash
find skills/ agents/ -name "*.md" | xargs grep -L "^name:" 2>/dev/null
# 若有输出 → 添加 name: 和 description: frontmatter
```

### 6.2 灵感 → 实施路径

**灵感 1：在 echo-sleuth 引入三阶段盲测模式（高价值）**
- 背景：echo-sleuth 目前的分析是单 agent 串行，容易有确认偏见
- 第一步：在 echo-sleuth/agents/ 下创建 `evaluator.md`，负责盲测两个「回声分析」结果，不看来源直接选优
- 实施路径：参照 `skills/skill-creator/agents/comparator.md` 的盲测设计，适配到回声腔检测场景

**灵感 2：为所有 echo-sleuth agent 补全 model 字段（低成本，高合规性）**
- 背景：当前 echo-sleuth 的 5 个 agent 均无 model 字段，与 feiskyer 同类问题
- 第一步：`grep -L "model:" agents/*.md` 确认范围，然后批量编辑
- 实施路径：analyze.md / memory-auditor.md / schema-scout.md 用 `model: claude-sonnet-4-5`；recall.md / file-historian.md 用 `model: claude-haiku-4-5`（查询类任务，Haiku 够用）

**灵感 3：为 4 个缺少 allowed-tools 的命令补全字段（合规性）**
- 背景：echo-sleuth 的 recall.md/recap.md/timeline.md/lessons.md 命令缺少 `allowed-tools`
- 第一步：确认每个命令实际需要调用哪些工具，精确列出
- 实施路径：参照 feiskyer ui-engineer.md 的 `tools: Read, Write, Edit, MultiEdit, LS, Glob, Grep, Bash, WebFetch` 格式

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

feiskyer/claude-code-settings 的核心目的：为全栈开发者提供覆盖完整开发工作流的专用化 AI 技能集合。

| 我的仓库 | 相似度 | 原因 |
|---|---|---|
| echo-sleuth-for-claude | **高** | 同为多 agent + skill + command 的 Claude plugin，有相似的编排复杂度和相同的 model 缺失问题 |
| drama-workshop-skills | 中 | 同为 skill-based，但专注单一领域（中文短剧），无多 agent 流水线 |
| claude-for-legal | 低 | 法律工作流专用，单一领域，无多 agent 编排 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 验证方法 | echo-sleuth 命中情况 | 严重度 |
|---|---|---|---|
| agent 无 model 字段 | `grep -L "model:" agents/*.md` | ✅ 全部 5 个 agent 均无 model 字段 | 中（-5/agent） |
| commands 缺少 allowed-tools | `grep -L "allowed-tools" commands/*.md` | ✅ 4/8 命令缺失（recall, recap, timeline, lessons） | 中（-5/command） |
| vague 量词 | `grep -c "尽量\|适当\|合理" skills/*/SKILL.md` | 需实测，预估 2-4 处 | 低（-4 至 -8） |
| 跨组件引用断裂 | `grep -rn "helpers/"` | 暂无 helpers 引用 | 无 |

**具体行动建议**：
1. 优先修复：echo-sleuth 所有 agent 加 `model:` 字段（共 5 处，预期 +25 分）
2. 次优修复：4 个缺 allowed-tools 命令补全（共 4 处，预期 +20 分）
3. 长期优化：系统清查 vague 量词并替换为可验证描述

### 8.3 别人的更优方案

**方案 1：skill-creator 的三阶段盲测流水线**
- 文件路径：`skills/skill-creator/SKILL.md` + `agents/comparator.md` + `agents/analyzer.md` + `agents/grader.md`
- 比我的方案更优之处：echo-sleuth 当前对「分析结果」的评估是主观的单 agent 判断；feiskyer 的盲测分离方案消除了确认偏见，analyzer 在不知道来源的情况下独立评判，结论更可信
- 可以直接借鉴：在 echo-sleuth 中用类似模式对比两次「回声腔分析」的结果

**方案 2：translate/SKILL.md 的多语言触发词设计**
- 文件路径：`skills/translate/SKILL.md`
- 比我的方案更优之处：description 不仅用英文描述触发条件，还加入了「翻译」「translate to Chinese」等多语言关键词，确保不同语言习惯的用户都能触发 skill。我的 skill 触发词通常只有英文
- 可以直接借鉴：在 echo-sleuth 的 recall.md 等命令描述中加入中文触发词

### 8.4 反向：我的项目做得比他们好的地方

echo-sleuth-for-claude 在 commands 中使用了 argument-hint（参数提示），引导用户提供正确格式的输入。feiskyer 的 7 个根级 agent 和多数 skill 没有这样的用户引导设计——用户触发 agent 后需要自行猜测如何提供输入。这一点 echo-sleuth 做得更好，降低了用户的使用门槛。

---

## 八、术语表

| 术语 | 解释 |
|---|---|
| **frontmatter** | Markdown 文件顶部用 `---` 包裹的 YAML 元数据区域。在 Claude Code 工件中，用来声明 `name`（名称）、`description`（触发描述）、`model`（使用哪个 AI 模型）等关键属性。缺失 frontmatter 会导致 Claude Code 无法正确识别和调用工件 |
| **plugin** | Claude Code 的可安装扩展包，包含 skills、commands、agents、hooks 等工件，有 `plugin.json` 清单文件。可以通过 `claude plugin install` 命令安装 |
| **skill** | 以 `SKILL.md` 为主文件的知识型工件，定义 Claude 在特定领域的专业行为。Skill 是「参考知识」，通常由 slash command 或 agent 按需加载 |
| **agent** | 以 `.md` 文件定义的自主执行工件，有自己的系统提示和工具权限。Agent 可以调用其他 agent（形成多 agent 流水线） |
| **YAML** | 一种以缩进表示层级的配置文件格式，Claude Code frontmatter 和 hooks.json 中使用。语法特点：用 `key: value` 表示键值对，用 `-` 表示列表项 |
| **bypassPermissions** | Claude Code 的一个权限模式，允许子进程跳过正常的用户确认步骤自动执行操作。属于 Medium 安全风险，在受信任的环境中可用，但在公共/共享环境中需谨慎 |
| **MCP** | Model Context Protocol，AI 模型与外部工具通信的标准协议。`.mcp.json` 定义了 Claude Code 可以调用的外部服务（如本案例中的 chrome-devtools-mcp） |
| **allowed-tools** | Agent/command frontmatter 中的字段，精确列出该工件被允许调用哪些工具（如 Read、Write、Bash）。缺失此字段意味着工件可以调用所有工具，可能带来安全风险 |
| **vague quantifier（模糊量词）** | 无法被机器或用户客观验证的描述词，如「尽量」「适当」「合理」「必要时」。NLPM 会统计并扣分，因为这类词给 LLM 的指令不够精确，执行结果难以预测 |
| **hooks.json** | Claude Code 的钩子配置文件，定义在特定工具调用前后执行的 shell 命令。本案例中用于实现自主循环的 stop-hook（终止条件检查） |
