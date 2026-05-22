# Dammyjay93/interface-design — 学习案例

**仓库**：https://github.com/Dammyjay93/interface-design
**Stars**：4630 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `vague-quantifier`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
面向 Claude Code 的界面设计技能插件，核心主张是「阻止 AI 复现训练数据里的泛化模式，强迫每一个设计决策都有意图」。4630 Stars，是本学习系列中 Stars 最高的单技能插件之一，说明 UI/UX 工程师群体对「把设计哲学注入 AI 提示词工作流」这件事有强烈需求。版本号 `2026.2.9.1212`，采用 [CalVer](#CalVer) 而非 [semver](#semver)，体现作者把「时间序」而非「功能里程碑」视为版本语义。

关键事实：
1. 整个仓库是纯 Markdown——零脚本、零 hook、零 MCP 配置、零依赖，Security 状态 **CLEAR**
2. 唯一的 SKILL.md 拥有异常密集的哲学性散文，而非常见的任务清单格式——这是本案例最值得研究的设计选择
3. 5 个命令（critique, audit, extract, init, status）覆盖「评审 → 诊断 → 提取系统 → 初始化 → 状态查询」完整工作流
4. SKILL.md 中 `Deep Dives` 一节引用了三个不存在的参考文件（`references/principles.md`, `references/validation.md`, `references/critique.md`），这是当前仓库尚未修复的已知缺陷
5. 没有 xiaolai 为此仓库写过的案例，本文是第一篇

### 1.2 架构剖析
- **目录结构**：
```
interface-design/
  .claude-plugin/
    plugin.json               # 插件 manifest（CalVer 2026.2.9.1212）
  .claude/
    commands/
      critique.md             # 设计评审
      audit.md                # 深度诊断
      extract.md              # 提取设计系统
      init.md                 # 初始化项目（SKILL.md Commands 未列出）
      status.md               # 状态查询
    skills/
      interface-design/
        SKILL.md              # 核心：设计哲学注入
```
- **文件类型分布**：1 个 SKILL.md，5 个 command，1 个 [manifest](#manifest)，无 agent，无脚本，无 hook
- **编排关系**：所有 5 个命令是平列关系，均独立可调用。没有路由层，没有入口技能，没有代理池。用户直接调用 `/critique`、`/audit` 等命令，每条命令依赖 SKILL.md 提供的哲学框架来约束模型行为
- **跨件契约**：SKILL.md 是唯一的共享知识基础。5 个命令读取同一份设计哲学，理论上应与 SKILL.md 保持语义一致。SKILL.md 的 `Scope` 节提到 `/frontend-design` 作为前置插件，但该插件未在本仓库定义——是一个悬空的外部引用

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「意图注入」——SKILL.md 的开篇即点明核心问题：「You will generate generic output. Your training has seen thousands of dashboards. The patterns are strong.」作者的答案不是给 AI 一份清单，而是通过哲学性散文改变模型的「默认倾向」。这是一种反向工程：不提供模板，而是拆解 AI 生成泛化输出的根本原因，然后在认知层面干预它
- **解决什么问题**：所有 AI 设计工具面临同一个问题：生成的 UI 看起来像每一个训练样本的平均值。SKILL.md 把这个问题明确命名为「Sameness Is Failure」，并通过四个章节（`Where Defaults Hide`、`Intent First`、`Every Choice Must Be A Choice`、`Sameness Is Failure`）系统性拆解失败路径
- **做了什么 trade-off**：用散文而非结构化规则来约束 AI——可读性极高，能传递「为什么」而不只是「做什么」；但代价是可验证性低（无法用 grep 检查「设计意图是否正确表达」），也更难维护（一旦需要更新，要重写整段散文而非改一行规则）
- **反映什么认知模型**：作者把 Claude 视为一个有「训练数据偏见」需要被矫正的实体，而不是一个中立的执行工具。SKILL.md 的整个结构是「先描述 AI 的失败模式，再给出矫正方向」——这是一种「元认知设计」，作者在设计 AI 的认知框架，而不只是写任务指令

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「纯 Markdown 精品意图注入」（Pure-Markdown Intent Injection）**

这个模式的关键特征：不通过工具调用、脚本、hook 来影响 AI 输出，而是通过精心构建的哲学性散文在「认知层」上重塑 AI 的默认行为。技能文件不是「做什么的清单」，而是「为什么这么做的论证」——用叙事性语言把设计判断力内化到模型的工作状态里。

模式特征清单（5 条）：
- 特征 1：SKILL.md 以「指出 AI 会犯的错」开篇，而非以「介绍我是什么技能」开篇——先建立问题意识，再给答案
- 特征 2：正文是散文而非 bullet point 清单——传递「为什么」的论证链，而非只传递「做什么」
- 特征 3：零执行表面（no scripts, no hooks, no MCP）——技能完全依赖语言影响力，没有任何代码杠杆
- 特征 4：关键概念用金句固化（「typography isn't holding your design — it IS your design」），便于模型在生成时触发和引用
- 特征 5：命令层（5 个 commands）是这个哲学框架的「入口」，不重复定义逻辑，只是触发点

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域专业知识密集、需要改变模型「默认风格」的技能 | ✅ 高度适用 | 意图注入的核心价值就是克服训练偏见 |
| UI/UX、写作、架构设计等主观判断类任务 | ✅ 适用 | 「好坏」无法用代码检查，必须靠语言传递判断标准 |
| 需要频繁与外部 API 交互的工作流 | ❌ 不适用 | 纯 Markdown 没有执行能力，必须引入脚本或 MCP |
| 需要跨会话记忆的复杂状态管理 | ❌ 不适用 | 无持久化机制，每次会话都是新鲜启动 |
| 有严格可验证输出格式要求的场景 | ❌ 不适用 | 散文式约束难以机械验证，应改用结构化规则 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 纯 Markdown 意图注入（本仓库） | interface-design | 零依赖，可读性极高，传递设计哲学而非只传任务 | 可验证性低，无法用工具检查「意图是否落地」 |
| 结构化规则 + 命令平列 | AgriciDaniel/claude-ads | 规则清晰，易维护，适合标准化输出 | 容易退化为格式清单，失去「为什么」的论证 |
| 路由层 + 代理池 + 脚本层 | AgriciDaniel/claude-blog | 功能最完整，支持复杂工作流 | 维护成本高，依赖多，适合团队级工具 |

### 2.4 改进空间
1. **当前问题**：SKILL.md 的 `Deep Dives` 节引用了三个 `references/` 文件，但这些文件根本不存在。**改进做法**：要么创建这三个文件，要么把引用改为内联内容（将原本想放在 references/ 里的内容直接嵌入 SKILL.md 对应小节），要么删去这三行引用。**预期收益**：消除三个悬空引用 Bug，用户调用 Deep Dives 时不再静默失效。

2. **当前问题**：所有 5 个命令文件缺少 `allowed-tools` 声明。对于 critique.md 这样需要读取当前文件或 UI 截图的命令，工具权限不明确会导致 Claude 要么过度扩展权限、要么保守拒绝读取必要文件。**改进做法**：在每个命令的 [frontmatter](#frontmatter) 里加 `allowed-tools: [Read, Bash]`（或仅 Read），明确声明需要的工具。**预期收益**：NLPM 各命令从 95 升至 100，减少运行时权限歧义。

3. **当前问题**：critique.md 没有空输入处理——如果用户在没有任何设计文件上下文的情况下调用 `/critique`，命令会无声地等待或产生无意义输出。**改进做法**：在 critique.md 开头加一段「当没有检测到设计文件或截图时，先引导用户提供上下文」的回退逻辑。**预期收益**：改善空调用体验，避免哲学框架在无上下文时「悬空运行」。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-27 当时得分 **94/100**，Security 状态：**CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/skills/interface-design/SKILL.md | 98 | 模糊量词「some」(-2)：「some decisions are creative and others are structural」 |
| .claude/commands/critique.md | 85 | 无空输入处理 (-10)，缺少 allowed-tools 声明 (-5) |
| .claude-plugin/plugin.json | 95 | CalVer 版本（信息性警告，不扣分） |
| .claude/commands/audit.md | 95 | 缺少 allowed-tools 声明 (-5) |
| .claude/commands/extract.md | 95 | 缺少 allowed-tools 声明 (-5) |
| .claude/commands/init.md | 95 | 缺少 allowed-tools 声明 (-5) |
| .claude/commands/status.md | 95 | 缺少 allowed-tools 声明 (-5) |

### 3.2 当时值得借鉴的模式

1. **「意图优先」启动协议**（Intent First）→ 为什么好：SKILL.md 的 `Intent First` 章节要求在任何设计动作之前先回答三个问题——WHO（用户是谁）、WHAT（他们在完成什么）、HOW（这影响设计语言）。这把主观判断转化为可验证的前置条件，而不是让 AI 凭感觉「开始设计」 → 原文路径：`.claude/skills/interface-design/SKILL.md` → 如何借鉴：在任何创意类 SKILL 里加一个「前置问题清单」章节，要求模型先完成回答再进入执行阶段，避免跳过意图直接生成

2. **「同质即失败」反模式命名**（Sameness Is Failure）→ 为什么好：把 AI 最常见的失败模式明确命名为「Sameness」，而不是模糊地说「要有创意」。命名反模式比命名目标更有约束力——告诉 AI「这个结果是失败的」比告诉 AI「要达到这个目标」更能改变生成行为，因为模型对「这是错的」信号比「这是对的」信号更敏感 → 原文路径：SKILL.md `Sameness Is Failure` 章节 → 如何借鉴：给自己的 SKILL 写「失败模式词典」——把 3-5 个常见的 AI 失败输出命名并描述，而非只写正面目标

3. **关键概念用等式金句固化**（examples-driven）→ 为什么好：「typography isn't holding your design — it IS your design」这类等式句式把两个概念的关系从「修饰性」升级为「本质性」，在生成时更容易被模型触发引用，也更难被稀释 → 原文路径：SKILL.md 排版章节 → 如何借鉴：把技能中最关键的认知转变用「X isn't Y — X IS Y」或「X 不是 Y 的辅助工具，X 就是 Y 本身」句式固化，一句话传递设计观

4. **零执行表面的安全架构**（security-gate）→ 为什么好：没有任何脚本、hook、MCP 配置，Security 状态 CLEAR。这不是无心之举——纯 Markdown 插件天然没有远程代码执行面，维护者不需要考虑供应链攻击、权限泄露等问题 → 如何借鉴：对于纯「知识注入」类技能，默认拒绝添加脚本，只有在有明确需求时才引入执行表面

5. **CalVer 版本语义选择**（manifest-discipline）→ 为什么好：`2026.2.9.1212` 用发布日期作为版本号，传递「这是一个持续演进的知识库」而非「这是一个有稳定 API 的库」的信号。对于设计哲学类技能，CalVer 比 semver 更诚实——没有 breaking change 的语义，只有「上次更新时间」 → 如何借鉴：对知识类/哲学类技能考虑 CalVer；对有 CLI 接口或代码调用的插件保持 semver

### 3.3 当时的缺陷

1. **SKILL.md 三个 references/ 文件悬空引用（Bugs 1-3）**：`references/principles.md`、`references/validation.md`、`references/critique.md` 全部不存在。为什么会失败：Claude 读到引用路径时会尝试访问对应文件，文件不存在时不会报错——而是用训练数据「自动补全」那部分内容。这意味着「Deep Dives」章节声称提供的深度参考，实际上是模型幻想出来的，不可预测也不可复现。**自查**：我的 SKILL.md 里有没有 `references/xxx.md` 这样的路径引用？全部验证是否实际存在于磁盘上？

2. **所有 5 个命令缺少 `allowed-tools` 声明**：命令文件没有声明自己需要哪些工具（如 Read、Bash、WebFetch）。为什么会失败：缺少声明时，Claude Code 要么需要在运行时申请权限（打断用户流），要么使用过于宽松的默认权限（超出命令实际需要范围）。对于 critique.md 这种「需要读取设计文件和截图」的命令，工具权限歧义直接影响是否能正常运行。**自查**：我的所有命令文件 frontmatter 是否都有 `allowed-tools:` 字段？

3. **critique.md 无空输入处理**：当用户在没有任何设计上下文（无文件、无截图、无代码）的情况下调用 `/critique`，命令没有任何回退逻辑。为什么会失败：一个哲学框架再强大，也需要具体的输入才能运作；无上下文时模型会对「假想中的 UI」进行评审，生成对用户毫无价值的通用建议，正好制造了这个插件最想避免的「泛化输出」。**自查**：我的命令文件里，有没有依赖用户提供上下文但又没有空输入回退的情况？

### 3.4 当时的优化机会
1. 给所有 5 个命令 frontmatter 加 `allowed-tools:` 字段，至少声明 `Read`（审查类命令必需）
2. 创建 `references/` 目录并补充三个引用文件，或将其内容内联到 SKILL.md 对应章节
3. 在 critique.md 开头加「检测上下文是否存在」的前置检查：如无设计文件或截图，先要求用户提供

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| references/ 三个文件悬空 | `ls .claude/ 中是否有 references/ 目录` | **持续存在**：仓库无 references/ 目录，三个引用文件均不存在 | Deep Dives 章节依赖的知识文件从未被创建；这是一个已知的设计负债 |
| 所有命令缺 allowed-tools | `grep -L allowed-tools .claude/commands/*.md` | **持续存在**：5 个命令文件全部无 allowed-tools 字段 | NLPM 评分各命令 -5 分的扣项至今未修复；权限模糊问题仍存在 |
| critique.md 无空输入处理 | `grep "context\|empty\|fallback\|no.*input" .claude/commands/critique.md` | **持续存在**：critique.md 无任何空上下文回退逻辑 | 最影响用户体验的缺陷未被修复；零上下文调用仍会产生泛化输出 |

### 4.2 架构演进
对比 2026-04-27 audit 时的文件清单与当前 HEAD，目录结构无变化——仍是同一个扁平结构（5 commands + 1 SKILL.md + 1 plugin.json）。没有新增文件，没有删除文件，没有重组目录。

这说明：作者在 audit 之后没有进行架构迭代。4630 Stars 表明这个插件在社区中有强大的吸引力，但高 Stars 并不等同于持续维护——这是「标杆仓库停滞」现象的典型案例。作者可能已经满足于当前状态，或者认为这些缺陷不影响核心价值（哲学框架完整，命令功能可用）。

### 4.3 新增的可学习模式
**暂无**：当前 HEAD 与 audit 快照完全一致，无新增设计模式或文件。

---

## 五、校准

### 5.1 我已经在做对的
1. **命令有使用场景说明**：我的命令文件有描述「什么时候用这个命令」，而不是只描述「这个命令做什么」——和 interface-design 命令层的触发语义类似
2. **技能文件包含「为什么」**：我的 SKILL.md 不只写步骤，也写设计原因，符合本仓库「传递意图而非只传规则」的精神
3. **审查 references 引用的有效性**：我习惯在写完 SKILL.md 后检查所有 `references/` 路径是否实际存在——本案例的 Bugs 1-3 提示我这个习惯是必要的
4. **Security CLEAR**：我的纯知识类技能也是零执行表面，和本仓库的安全架构策略一致
5. **命令工作流覆盖**：和本仓库 5 个命令覆盖完整工作流类似，我尽量让命令组覆盖「启动 → 执行 → 检查」的完整链条，而不是单点工具

### 5.2 挑战 / 验证
- **这次案例挑战了一个假设**：「哲学性散文不如结构化规则有效，SKILL.md 应该用 bullet point」。本仓库用 4630 Stars 证明了相反的结论——哲学性散文可以比规则清单更有力地改变 AI 行为，前提是散文要针对 AI 的具体失败机制而不是泛泛而谈。SKILL.md 的核心句「You will generate generic output. Your training has seen thousands of dashboards. The patterns are strong.」是在对 AI 直接说话，而不是对人类用户说话——这个叙事角度我之前没有用过。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能文件中是否有悬空的 references/ 引用
grep -rn "references/" ~/.claude/skills/*/SKILL.md 2>/dev/null | while IFS=: read file line content; do
  ref=$(echo "$content" | grep -oP 'references/[^\s\)\"]+')
  dir=$(dirname "$file")
  [ -n "$ref" ] && [ ! -f "$dir/$ref" ] && echo "BROKEN: $file line $line → $ref"
done
# 命中后：要么创建对应的 references/*.md 文件，要么将内容内联到 SKILL.md

# 检查所有命令文件是否有 allowed-tools 声明
grep -rL "allowed-tools" ~/.claude/commands/*.md 2>/dev/null
# 命中后：在对应命令 frontmatter 中加 allowed-tools: [Read] 或所需工具列表

# 检查命令文件是否有空输入回退逻辑
for f in ~/.claude/commands/*.md; do
  grep -q "context\|fallback\|empty\|no.*input\|如果没有\|当.*未提供" "$f" || echo "MISSING fallback: $f"
done
# 命中后：在命令文件开头加「当未检测到上下文时，先要求用户提供 X」的条件分支

# 检查 SKILL.md 中是否有模糊量词
grep -rn -E '\bsome\b|\bappropriate\b|\bcomprehensive\b|\brobust\b|\befficient\b' ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：将 "some decisions" 改为具体描述——「typography decisions, color decisions」替代「some decisions」
```

### 6.2 灵感 → 实施路径

1. **想法**：在 drama-workshop-skills/short-drama/SKILL.md 的开篇加一段「AI 在中文短剧创作中的典型失败模式」——类似 interface-design SKILL.md 的「You will generate generic output」段落
   - **为何可行**：interface-design 证明了「先命名失败模式」比「列举成功目标」更能改变模型行为；短剧领域同样有训练数据偏见问题（AI 生成的剧情遵循固定套路）
   - **第一步**：打开 `short-drama/SKILL.md`，在文件最顶部（frontmatter 之后）加一个 `## 你的默认失败` 章节，描述 AI 在中文短剧领域最常犯的 3 个具体错误（如「主角总是工作繁忙的都市精英」「反转总在第 3 集」「结局总是原谅与和解」）；预计 30 分钟

2. **想法**：给所有我的命令文件补全 `allowed-tools` 字段，顺便修复 echo-sleuth-for-claude 里的 4 个缺失命令
   - **为何可行**：本案例确认了「缺少 allowed-tools 是高频问题」（interface-design 5/5 缺失，echo-sleuth 4/N 缺失），统一修复有固定操作步骤
   - **第一步**：`grep -rL "allowed-tools" ~/projects/echo-sleuth-for-claude/commands/` 列出缺失文件，对每个文件在 frontmatter 最后一行加 `allowed-tools: [Read]`（lessons.md、timeline.md、recall.md 均为读取类命令）；预计 15 分钟

3. **想法**：把「反模式命名」技术应用到 drama-workshop-skills 的命令设计说明里——创建一个 `references/failure-patterns.md` 文件，列出中文短剧 AI 写作的 5 个命名失败模式，并在 SKILL.md 中通过有效路径引用
   - **为何可行**：interface-design 的悬空引用缺陷（Bugs 1-3）提示了一个反向机会——如果我先建好 references/ 文件再在 SKILL.md 引用，就能既借鉴「反模式命名」的力量，又避免悬空引用的 Bug；两个教训同时落地
   - **第一步**：在 `drama-workshop-skills/short-drama/references/failure-patterns.md` 创建文件，写 5 个短剧 AI 失败模式（每条格式：「名称 → 表现 → 为什么 AI 倾向于这样」），然后在 SKILL.md `Deep Dives` 节加引用；约 45 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 Dammyjay93/interface-design 的核心目的**：通过哲学性散文改变 Claude 的领域默认行为，注入「界面设计专业判断力」，阻止泛化输出

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为「单技能深度专业插件」，SKILL.md 均采用哲学性叙事章节而非纯清单格式，均缺少 allowed-tools 声明 | interface-design 的哲学批判指向 AI 本身（「你的训练数据是问题」），drama-workshop 更偏程序性；drama-workshop 有中文双语支持，interface-design 全英文 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同样存在 allowed-tools 缺失问题（4 个命令），命令结构平列 | 目的不同（记忆挖掘 vs 界面设计），echo-sleuth 更注重跨会话状态，interface-design 无状态 | 中（仅 allowed-tools 缺失问题） |
| MarkQWu/claude-for-legal | 低 | 同为专业领域插件 | 法律插件更偏事务性（查询/起草），结构更接近 claude-ads 的清单模式，而非 interface-design 的哲学框架 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 所有命令缺 allowed-tools | `grep -rL "allowed-tools" my-repos/*/commands/*.md` | drama-workshop-skills：short-drama/SKILL.md 无 allowed-tools 字段（SKILL 文件无此字段为正常，但命令文件待查）；echo-sleuth-for-claude：lessons.md、timeline.md、recap.md、recall.md 确认缺失（4/N 命中） | 高 |
| 哲学性 SKILL.md 未命名 AI 失败模式 | 主观阅读，无法 grep | drama-workshop-skills/short-drama/SKILL.md：叙事详细但未明确命名「AI 的默认失败模式」，缺少「你会犯这个错」的对 AI 直接定向批评 | 中 |
| references/ 悬空引用 | `grep -rn "references/" my-repos/*/skills/*/SKILL.md` | 目前我的技能文件未使用 references/ 引用模式——这是 **规避了该 Bug**，但也意味着没有用到「按需深度参考」的架构优势 | 低（当前未命中，但将来计划加 references/ 时需注意） |

**命中后的具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude` 的 `commands/lessons.md` → 在 frontmatter 加一行 `allowed-tools: [Read]` → 5 分钟可完成
- `MarkQWu/echo-sleuth-for-claude` 的 `commands/timeline.md`、`commands/recap.md` → 同上，各 5 分钟
- `MarkQWu/drama-workshop-skills/short-drama/SKILL.md` → 在开篇加一段「AI 在短剧领域的默认失败」，具体写「你会生成套路剧情，因为训练数据里有数万个相似的短视频剧本，这些模式很强」→ 20-30 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：直接对 AI 讲话的元认知 SKILL 架构
   - **本案例做法**：SKILL.md 开篇直接用第二人称对 Claude 说：「You will generate generic output. Your training has seen thousands of dashboards. The patterns are strong.」——主语是「你（Claude）」，把 AI 的训练数据偏见作为需要被主动对抗的已知问题（`.claude/skills/interface-design/SKILL.md` 前 20 行）
   - **我的项目现状**：`drama-workshop-skills/short-drama/SKILL.md` 的叙事是对人类用户写的（「如何使用这个技能」），不是对 AI 写的（「你会犯这个错，这是为什么，这是矫正方向」）。这个叙事角度的差异决定了是否能在「认知层」而非「任务层」影响 AI
   - **如何借鉴**：在 short-drama/SKILL.md 增加一个 `## 你的训练偏见` 章节，第二人称书写，具体列出中文短剧领域的 3 个 AI 默认失败点，并给出「为什么训练数据会让你倾向于这样」的根因解释；修改 `short-drama/SKILL.md` 第 1-30 行，约 25 分钟

2. **领域**：命令平列 + 哲学共享层的轻量编排
   - **本案例做法**：5 个命令共用同一个 SKILL.md 哲学框架，每个命令只是「入口」，不重复定义设计原则，保持 SKILL.md 为单一信息源
   - **我的项目现状**：drama-workshop-skills 和 echo-sleuth-for-claude 的命令文件各自包含了部分原则性说明，导致跨命令出现轻微的「原则漂移」风险——critique 说的和 audit 说的可能不完全一致
   - **如何借鉴**：将各命令文件里的「原则性段落」移到 SKILL.md 统一管理，命令文件只保留触发语义和工具声明；git diff 思路：在命令文件里删除重复原则段，在 SKILL.md 末尾增加对应章节，并用 `See also: skills/xxx/SKILL.md §节名` 替代

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：中文语言支持与双语状态管理
   - **我的做法**：`MarkQWu/drama-workshop-skills/short-drama/SKILL.md` 原生支持中文创作指令，部分关键段落提供中英双语描述，命令触发词也有中文版
   - **本案例做法（弱在哪）**：interface-design 全英文，没有任何本地化支持；对于非英语母语的设计师（包括大量中国 UI 工程师），在工具调用和错误描述时会有语言摩擦
   - **意义**：如果考虑给 interface-design 开 PR，可以贡献一个「中文版设计哲学导读」或中文命令触发别名，这是现有仓库的空白，也是 drama-workshop-skills 做法证明可行的方向

2. **领域**：域级状态管理与项目上下文持久化
   - **我的做法**：echo-sleuth-for-claude 的 recall.md 和 recap.md 命令涉及跨会话记忆的读取与整理，有明确的状态管理语义
   - **本案例做法（弱在哪）**：interface-design 完全无状态——每次调用 `/critique` 都是新鲜启动，没有「这个项目上次评审的结论」这类持久化机制；status.md 命令描述中的「状态」指的是当前会话的设计状态，而非跨会话记录
   - **意义**：对于设计系统类插件，「上次评审的决定」是重要上下文，echo-sleuth 的跨会话记忆模式是 interface-design 尚未解决的演进方向

---

## 八、术语表

### <a name="CalVer"></a>CalVer（日历版本）
> 用发布日期（年.月.日.时间戳）代替功能里程碑（1.0.0、2.3.1）来标记版本的方式。例如 `2026.2.9.1212` 表示 2026 年 2 月 9 日 12:12 发布。适合知识库、文档、技能文件等「没有 API 稳定性承诺」的内容——版本号只说明「这是什么时候的快照」，不说明「相比上版本有没有破坏性变更」。

### <a name="semver"></a>semver（语义化版本）
> 格式为 `主版本.次版本.修订号`（如 `1.4.2`）的版本规范。主版本号变化表示不兼容的 API 变更，次版本号变化表示新增功能但向下兼容，修订号变化表示 bug 修复。适合有 API 接口的库和工具。与 CalVer 相比，semver 对「变更性质」有语义承诺，CalVer 只有「时间序」承诺。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读命令文件时先解析 frontmatter 才能知道这个命令需要什么工具权限、如何注册命令名。

### <a name="manifest"></a>manifest（清单文件）
> 项目的「目录」，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands 和 skills 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载；如果 manifest 里列了一个不存在的路径，插件安装会报错。

### <a name="allowed-tools"></a>allowed-tools（工具白名单）
> 命令 frontmatter 里的一个字段，声明该命令在运行时被允许使用哪些 Claude Code 内置工具（如 `Read`、`Bash`、`WebFetch`、`Write` 等）。未声明时 Claude Code 使用宽松默认权限，可能导致「工具调用权限被拒」或「超出实际需要的权限范围」两种相反的问题。

### <a name="悬空引用"></a>悬空引用（dangling reference）
> 文件 A 引用了文件 B 的路径，但文件 B 在磁盘上不存在。与编程语言的「空指针」类似，但在 Claude Code SKILL.md 里不会抛出异常——而是静默失效：AI 读不到被引用的内容，自动用训练数据「脑补」填补，产生不可预测的输出。本案例的 references/ 三个文件就是这种情况。

### <a name="元认知设计"></a>元认知设计（meta-cognitive design）
> 在设计 AI 的工作说明时，不只描述「要做什么任务」，而是描述「AI 的认知过程中哪里容易出错，以及为什么」——即对 AI 的认知行为建模并加以干预。interface-design SKILL.md 是典型的元认知设计：它不是在教 AI 如何做界面设计，而是在矫正 AI「看到 UI 任务就自动调用训练数据里最频繁的模式」这一行为倾向。
