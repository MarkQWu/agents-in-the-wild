# forrestchang/andrej-karpathy-skills — 学习案例

**仓库**：https://github.com/forrestchang/andrej-karpathy-skills
**Stars**：125,813 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-01（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `vague-quantifier`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是 GitHub 上关注度最高的 Claude Code [skill](#skill) 仓库之一，125,813 颗星。作者 Forrest Chang（GitHub：forrestchang）以 Andrej Karpathy 于 https://x.com/karpathy/status/2015883857489522876 发布的推文为蓝本，将其中关于 LLM 编码陷阱的具体观察转化为一套可机器加载的行为指南。

整个仓库只有三个核心 NL 工件：`CLAUDE.md`、`.claude-plugin/plugin.json`、`skills/karpathy-guidelines/SKILL.md`。此外还附带 `CURSOR.md`（Cursor IDE 镜像）、`.cursor/rules/karpathy-guidelines.mdc`（Cursor 规则文件）、`EXAMPLES.md`（示例说明）、`README.md` 和 `README.zh.md`（中文 README）。

关键事实：
1. CLAUDE.md 和 SKILL.md 内容几乎完全相同——这是刻意的[双呈现模式](#dual-presence)设计：CLAUDE.md 用于项目级自动加载，SKILL.md 用于 [agent](#agent) 显式按需加载
2. 零执行面：无 hooks，无 scripts，无 MCP 配置——安全评级 CLEAR，无需安全门控
3. [plugin.json](#plugin-json) 干净，`skills` 数组正确引用 `"./skills/karpathy-guidelines"`，路径解析到 SKILL.md
4. SKILL.md [frontmatter](#frontmatter) 完整：`name: karpathy-guidelines`、`description: ...`、`license: MIT`
5. 仓库本身就是 Karpathy「简洁优先」原则的活体示范——只有必要的文件，没有任何推测性内容

### 1.2 架构剖析

目录结构：
```
andrej-karpathy-skills/
  .claude-plugin/
    plugin.json              # manifest，注册唯一 skill
  skills/
    karpathy-guidelines/
      SKILL.md               # agent 可加载版本的行为指南
  CLAUDE.md                  # 项目级行为指南（内容与 SKILL.md 近同）
  CURSOR.md                  # Cursor 镜像（内容与 CLAUDE.md 近同）
  .cursor/rules/
    karpathy-guidelines.mdc  # Cursor 规则文件
  EXAMPLES.md
  README.md
  README.zh.md
```

- **文件类型分布**：1 个 skill，0 个 agent，0 个 command，0 个 hook，1 个 manifest（plugin.json）
- **编排关系**：无编排层。CLAUDE.md 在 Claude Code 打开项目时自动激活；SKILL.md 可被其他 agent 或用户显式加载。两者独立工作，互不依赖，也不需要 router 或调度器
- **跨件契约**：plugin.json 声明 skill 路径，SKILL.md 的 frontmatter name 须与调用名一致。除此之外无跨件约定

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「最小可行指令集」——只包含 Karpathy 原推文中有足够实践信号的规则，不扩展、不推测
- **解决什么问题**：LLM 写代码时容易犯的系统性错误（提前过度设计、改动扩散、目标不清就开工），这些问题 prompt 级提示容易被遗忘，而 CLAUDE.md 会在每次 session 开始时自动加载，相当于把检查清单内置到工具里
- **做了什么 trade-off**：刻意保持内容精短、无示例（SKILL.md 本身就是行为约束而非任务执行器，输出格式不是关键），代价是失去 `<example>` 块带来的 NLPM 评分加成；但作者明显选择了「可读性 + 低维护」而非「满分」
- **反映什么认知模型**：作者把 Claude Code [skill](#skill) 视为「行为约束层」而非「功能执行器」——skill 的价值在于限制和引导 Claude 的默认行为，而非主动完成任务。这与大多数功能型 skill 仓库的定位截然不同

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单 Skill 极简原则」（Single-Skill Minimalist）**

这个模式的核心特征：整个仓库只有一个 [skill](#skill)，内容是行为约束而非任务执行器；CLAUDE.md 与 SKILL.md [双呈现](#dual-presence)同一套内容；零执行面。

模式特征清单（4 条）：
- 特征 1：唯一 skill，内容是 LLM 行为约束（do/don't 列表 + 判断规则），而非「执行某任务」的操作指南
- 特征 2：[双呈现模式](#dual-presence)——同一内容以 CLAUDE.md（项目级自动加载）和 SKILL.md（agent 显式加载）两种形式存在，覆盖所有使用场景
- 特征 3：零执行面——无 hooks、无 scripts、无 MCP 配置，安全风险为零，用户无需审查执行代码
- 特征 4：跨编辑器镜像——CURSOR.md 和 `.cursor/rules/` 让同一套规则在 Cursor 生态同样生效，一次维护，多处覆盖

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人或团队统一 LLM 编码行为规范 | ✅ 高度适用 | CLAUDE.md 自动加载，无需手动调用；skill 可按需加载到特定 agent |
| 作为其他插件的「基础行为层」叠加 | ✅ 适用 | 零执行面，不会与其他插件的 hooks/scripts 冲突 |
| 需要跨 IDE 同步规范（Claude Code + Cursor）| ✅ 适用 | 双呈现 + 镜像文件机制天然支持 |
| 需要复杂多步任务执行的功能型插件 | ❌ 不适用 | 无 agent、无 command，无任务调度能力 |
| 需要自动化脚本或系统集成 | ❌ 不适用 | 故意不提供执行面，这不是此模式的定位 |
| 需要丰富示例驱动型指南 | ⚠️ 有限适用 | SKILL.md 无 `<example>` 块，EXAMPLES.md 以散文形式提供说明但不是机器可解析的结构化示例 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单 Skill 极简原则（本仓库） | forrestchang/andrej-karpathy-skills | 零安全风险，维护成本极低，随处可安装 | 无任务执行能力，无示例，NLPM 评分受限 |
| 单仓多 Agent 平铺 | 0xfurai/claude-code-subagents | 覆盖面广，按领域按需使用 | 无 manifest，无示例，质量参差 |
| 完整插件（commands + agents + skills） | MarkQWu/echo-sleuth-for-claude | 功能完整，可在 marketplace 发布 | 结构复杂，维护负担重，执行面大需安全审查 |
| 规则集（.claude/rules/）| 各单 CLAUDE.md 仓库 | 无需插件安装，直接使用 | 无法通过 marketplace 分发，跨项目复用困难 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 第 3 行含[模糊量词](#vague-quantifier)「as needed」。**改进做法**：把「Merge with project-specific instructions as needed」改为「Merge with project-specific instructions by appending your project rules after this file's content」，给出具体合并操作的参考方式。**预期收益**：NLPM 评分从 99 升至 100，消除唯一扣分项。
2. **当前问题**：SKILL.md 无 `<example>` 块，用户无法快速验证加载后的行为变化。**改进做法**：在 SKILL.md 末尾加 `## Examples` 节，展示一个「未加载 skill 时 Claude 的行为」vs「加载后 Claude 的行为」对比，帮助用户建立预期。**预期收益**：提升用户信任度，降低「安装了有没有用」的疑虑。
3. **当前问题**：README 未说明 CLAUDE.md 与 SKILL.md 的关系（双呈现设计意图不透明）。**改进做法**：在 README 中增加一段「Architecture」说明，解释两个文件的分工。**预期收益**：降低贡献者和高级用户的理解成本。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **99/100**，接近满分。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 98 | 第 3 行「as needed」[模糊量词](#vague-quantifier)，-2 分 |
| SKILL.md | 100 | 无问题 |
| .claude-plugin/plugin.json | 100 | 无问题，路径引用正确 |

整体无 bug，无安全发现，所有交叉引用均验证通过。唯一扣分点是 CLAUDE.md 第 3 行的一个模糊量词。

### 3.2 当时值得借鉴的模式

1. **[双呈现模式](#dual-presence)** → 为什么好：CLAUDE.md 保证行为指南在项目级始终生效（每次 session 自动加载），SKILL.md 允许其他 agent 按需显式加载，覆盖了「被动激活」和「主动调用」两种使用场景，互不干扰 → 原文路径：`CLAUDE.md` 和 `skills/karpathy-guidelines/SKILL.md` 内容对比 → 如何借鉴：针对行为约束类内容，同时维护一份 CLAUDE.md（项目级）和一份 SKILL.md（agent 可加载），内容保持同步。
2. **零执行面设计** → 为什么好：无 hooks、无 scripts 意味着用户安装这个插件时无需审查任何可执行代码，安全门控直接通过，降低了用户的信任成本 → 原文路径：仓库目录结构，无 `hooks/`、`scripts/` 目录 → 如何借鉴：如果 skill 的核心价值是「约束行为」而非「执行操作」，主动保持零执行面，把安全审查负担降到零。
3. **[manifest](#plugin-json) 干净精准** → 为什么好：plugin.json 只声明一个 skill，路径引用正确，没有多余字段，没有引用不存在的文件 → 原文路径：`.claude-plugin/plugin.json` 的 `skills` 数组 → 如何借鉴：plugin.json 里的每条路径都应该对应一个实际存在且有效的文件，定期用 `nlpm:check` 验证交叉引用。

### 3.3 当时的缺陷

1. **CLAUDE.md 第 3 行「as needed」模糊量词**：「Merge with project-specific instructions as needed」中的「as needed」没有给出判断标准——什么情况下需要合并，什么情况下不需要？用户遇到这句话时必须自己判断，而不同用户的判断会产生不一致的使用方式。为什么这会失败：[模糊量词](#vague-quantifier)让指令失去可测量性，Claude 在解释「as needed」时会基于上下文自行发挥，导致行为不可预期。**自查**：我的 CLAUDE.md 或 SKILL.md 里有没有「as needed」「when appropriate」「if necessary」这类词？→ 检查命令：`grep -rn "as needed\|when appropriate\|if necessary" ~/.claude/ 2>/dev/null`。

### 3.4 当时的优化机会

1. 将 CLAUDE.md 第 3 行「as needed」替换为具体的合并说明，一行改动消除唯一扣分项，评分从 99 升至 100。
2. 在 SKILL.md 增加 `## Examples` 节，哪怕只有一个最小对比示例，提升用户对「安装有什么效果」的感知。
3. 在 README 中增加「双呈现设计」的说明，让高级用户理解 CLAUDE.md 和 SKILL.md 分工的架构意图。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md:3「as needed」模糊量词 | `head -5 CLAUDE.md` 查看第 3 行 | **持续存在**：当前 HEAD 第 3 行仍为「Merge with project-specific instructions as needed.」 | 维护者在 Audit 后约 2 个月内未修复；这是刻意的 trade-off 接受，而非遗漏 |

### 4.2 架构演进

从 audit 时（2026-04-06）到当前 HEAD（2026-06-01），核心架构没有变化。三个 NL 工件（CLAUDE.md、plugin.json、SKILL.md）结构与内容保持稳定，无新增 agent、command 或 hook。CURSOR.md 和 `.cursor/rules/` 属于跨 IDE 镜像，与 NL 工件设计无关。

**作者后来意识到的**：唯一缺陷（「as needed」模糊量词）的扣分值仅 -2，且在语义上并未造成实质性歧义——用户看到「按需合并」时通常能理解其意图。维护者选择接受这个小瑕疵，而非为追求满分进行微小改动。这是一个理性的「好到足够」决策，而非懒惰。

### 4.3 新增的可学习模式

暂无新增架构模式。2 个月内仓库保持稳定，无结构变化。稳定性本身是一个信号：对于行为约束类 skill，核心规则确定后无需频繁迭代——这与功能型插件（需要持续扩充场景覆盖）的维护节奏截然不同。

---

## 五、校准

### 5.1 我已经在做对的

1. **使用完整 plugin.json**：我的 echo-sleuth-for-claude 和 claude-for-legal 都有完整的 plugin.json，所有工件均注册，可通过 `claude plugin install` 一键安装——与本仓库 manifest 精准的做法一致。
2. **SKILL.md frontmatter 完整**：我的 skill 文件有完整的 `name`、`description` 等 frontmatter 字段，与本仓库 SKILL.md 格式一致。
3. **避免在行为约束型内容里引入不必要的执行面**：我的 drama-workshop-skills 是单 skill 仓库，无 hook，无 script，与本仓库的零执行面理念一致。

### 5.2 挑战 / 验证

- **这次案例验证了「双呈现模式」的价值**：我之前只维护 SKILL.md，没有配套的 CLAUDE.md。看到本仓库通过 CLAUDE.md 实现「始终激活」的效果，我意识到对于核心行为约束类内容，缺少 CLAUDE.md 意味着用户必须显式加载才能生效——这增加了使用摩擦。对于 drama-workshop-skills 这类专注单一场景的 skill，补充一个 CLAUDE.md 是有价值的改进。
- **认知修正**：我以为「99 分仓库一定做了很多事」，但这个仓库证明极度克制本身就是一种做对的事。125,813 颗星来自 3 个文件，核心洞察是：规则的质量比规则的数量重要得多。
- **「as needed」的处理方式值得思考**：维护者 2 个月不修复这个-2 分的模糊量词，说明在现实中「满分」不是目标，「有效」才是目标。这对我的修改优先级判断有参考价值——不必为微小的 NLPM 扣分项耗费精力，先修有实质影响的问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 CLAUDE.md 和 SKILL.md 是否含模糊量词
grep -rn "as needed\|when appropriate\|if necessary\|as required\|where applicable" \
  ~/.claude/ ~/path/to/my-repos/ 2>/dev/null
# 命中后：替换为具体条件，例如「as needed」→「when your project has domain-specific terminology not covered here」

# 验证自己的 plugin.json 路径引用是否都能解析到实际文件
for f in $(python3 -c "
import json, sys
d = json.load(open('plugin.json'))
for s in d.get('skills', []): print(s)
for a in d.get('agents', []): print(a)
for c in d.get('commands', []): print(c)
"); do
  [ -e "$f" ] || echo "MISSING: $f"
done
# 命中后：补充缺失文件或更新 plugin.json 路径

# 检查自己是否有「双呈现」缺口——有 SKILL.md 但缺对应 CLAUDE.md
for skill_dir in skills/*/; do
  skill_name=$(basename "$skill_dir")
  [ -f "CLAUDE.md" ] || echo "MISSING CLAUDE.md for skill: $skill_name"
done
# 命中后：创建 CLAUDE.md，内容与核心行为约束的 SKILL.md 同步

# 检查 SKILL.md frontmatter 完整性
for f in skills/*/SKILL.md; do
  python3 -c "
import sys
content = open('$f').read()
if not content.startswith('---'):
    print('MISSING FRONTMATTER: $f')
elif 'name:' not in content.split('---')[1]:
    print('MISSING name: $f')
elif 'description:' not in content.split('---')[1]:
    print('MISSING description: $f')
"
done
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 补充 CLAUDE.md（双呈现模式）
   - **为何可行**：drama-workshop-skills 的 SKILL.md 包含核心行为约束（戏剧创作规范），但需要用户显式加载 skill 才能生效。补充一个 CLAUDE.md 可以让这些规范在项目打开时自动激活，对于把整个仓库 `git clone` 到本地作为项目工作区的用户特别有价值
   - **第一步**：把 `skills/short-drama/SKILL.md` 的核心约束段落提炼到 `CLAUDE.md`，去掉纯 agent 元数据（frontmatter 字段），保留行为规则正文；约 15 分钟

2. **想法**：把「双呈现模式」作为我所有行为约束类 skill 的标准设计模式
   - **为何可行**：本仓库证明了这个模式在最高流量 skill 仓库中的有效性。对于我的每个「设定行为约束」而非「执行任务」的 skill，双呈现是一次投入、永久受益的设计选择
   - **第一步**：在自己的 skill 仓库 README 中标注哪些 skill 是「行为约束型」，哪些是「任务执行型」；行为约束型的均补充对应 CLAUDE.md；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：用户仓库 `MarkQWu/echo-sleuth-for-claude`、`MarkQWu/drama-workshop-skills`、`MarkQWu/claude-for-legal`

### 8.1 目的对齐度

- **本案例 forrestchang/andrej-karpathy-skills 的核心目的**：将 Karpathy 对 LLM 编码陷阱的观察固化为可机器加载的行为约束，通过双呈现模式在 Claude Code 生态中无缝分发

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为单 skill 导向，聚焦单一领域；同样极简结构；同为行为/风格约束而非功能执行 | drama-workshop-skills 缺少 CLAUDE.md，无双呈现；针对创作场景而非工程场景 | 极高——最应该学习双呈现模式 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件，有 plugin.json | echo-sleuth 是功能型插件（commands + agents + skills 完整架构），而非单 skill 行为约束层；复杂度不在同一量级 | 中——学习 manifest 精准度和零执行面意识 |
| MarkQWu/claude-for-legal | 低 | 同为 skill 为主的设计；同样有 README | 多领域垂直插件，skill 是任务执行器而非行为约束器；执行面更大 | 低——架构差异过大，难以直接借鉴 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| [模糊量词](#vague-quantifier)「as needed」 | `grep -rn "as needed\|when appropriate" skills/ CLAUDE.md 2>/dev/null` | echo-sleuth experience-synthesis/SKILL.md:118 含「relevant」；git-mining/SKILL.md:93 含「relevant」 | 中（-2 分/处） |
| 缺少 CLAUDE.md（双呈现缺口） | `ls CLAUDE.md 2>/dev/null \|\| echo MISSING` | drama-workshop-skills 完全没有 CLAUDE.md | 高——用户必须显式加载才能激活行为约束 |
| SKILL.md 缺 `## Output Format` 节 | `grep -l "## Output Format" skills/*/SKILL.md` | drama-workshop-skills/short-drama/SKILL.md 无 Output Format 节（注：本案例也无，因内容是行为约束而非任务执行器，两者相同理由） | 低（同本案例，行为约束型 skill 此节非必须） |

**命中后的具体行动建议**：
- drama-workshop-skills → 补充 `CLAUDE.md`，将 short-drama SKILL.md 的核心行为约束提炼进去；约 15 分钟
- echo-sleuth experience-synthesis/SKILL.md → 将「relevant」替换为具体限定词，例如「relevant（与当前问题直接相关的，而非泛泛相关的）」；约 5 分钟/处

### 8.3 别人的更优方案

1. **领域**：双呈现模式（CLAUDE.md + SKILL.md）
   - **本案例做法**：CLAUDE.md 在项目打开时自动激活行为约束，SKILL.md 允许 agent 按需显式加载同一内容（`skills/karpathy-guidelines/SKILL.md`，100/100）
   - **我的项目现状**：drama-workshop-skills 只有 SKILL.md，用户必须通过 `claude --skill karpathy-guidelines` 或在 agent frontmatter 中声明才能使用；无法在项目级自动生效
   - **如何借鉴**：创建 `CLAUDE.md`，将 SKILL.md 正文中的行为约束部分（去掉 frontmatter 元数据）复制进去；维护时两个文件同步更新；改动约 15 分钟，一次投入，永久受益

2. **领域**：零执行面设计意识
   - **本案例做法**：刻意不引入任何 hooks 或 scripts，安全评级 CLEAR，用户无需审查执行代码，降低安装信任门槛
   - **我的项目现状**：echo-sleuth-for-claude 有执行面（commands 触发 agent，agent 调用 Bash 工具），这是功能需要，但缺少安全说明；用户无法快速判断执行面的风险等级
   - **如何借鉴**：在 README 中增加「Security」节，说明插件的执行面范围（哪些工具会被调用、不会访问网络/凭证等），降低用户的安全顾虑

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：结构化示例（`<example>` 块）
  - **我的做法**：echo-sleuth-for-claude 的 experience-synthesis/SKILL.md 包含显式输出格式说明，用户可以预期 skill 执行后的输出结构
  - **本案例做法**（弱在哪）：forrestchang/andrej-karpathy-skills 的 SKILL.md 无 `<example>` 块，EXAMPLES.md 是散文说明，不是机器可解析的结构化示例；用户只能靠 README 文字描述推断效果
  - **意义**：对于任务执行型 skill，结构化示例是我已经在做且做得比本仓库好的地方，应该继续保持

- **领域**：README 语言覆盖
  - **我的做法**：项目 README 提供完整中英双语说明
  - **本案例做法**（弱在哪）：本仓库有 README.md 和 README.zh.md 两个文件，维护两份文档存在同步成本；内容是否保持一致需要人工保证
  - **意义**：单文件双语 README（用 `## English` / `## 中文` 节分隔）比双文件方案维护成本更低，不过这是风格选择，无优劣之分

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`license` 等）。Claude Code 读取 skill / agent / command 文件时，先解析 frontmatter 才能知道这个工件如何注册和调用。`---` 必须严格从第 1 列（行首）开始，有任何前导空格都会导致解析失败。本仓库 SKILL.md 的 frontmatter 包含 `name: karpathy-guidelines`、`description: ...`、`license: MIT` 三个字段，是标准写法的范例。

### <a name="skill"></a>skill
> Claude Code 插件体系中的「可复用知识单元」。与 agent（自主行动者）和 command（用户触发的流程）不同，skill 的定位是「加载进上下文的参考知识或行为约束」，由其他 agent 在 frontmatter 中通过 `skills:` 字段声明按需加载，或由用户显式引用。本仓库的 `skills/karpathy-guidelines/SKILL.md` 是一个行为约束型 skill，加载后会限制 Claude 的默认编码行为。

### <a name="agent"></a>agent
> Claude Code 插件体系中的「自主行动者」，有独立的目标、上下文和工具权限声明（frontmatter 中的 `allowed-tools`）。与 skill 的区别：skill 是知识/约束，agent 是执行主体。本仓库没有定义任何 agent——这是零执行面设计的体现之一。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> 指令文本中缺乏可测量标准的副词或形容词，例如「as needed」「when appropriate」「comprehensive」「robust」「relevant」等。这类词让 AI 在执行时必须自行判断标准，不同调用之间结果可能不一致。NLPM 评分体系对每个模糊量词扣 2 分（上限 -10 分/文件）。本仓库唯一的扣分项就是 CLAUDE.md 第 3 行的「as needed」，导致总分从 100 降至 99。

### <a name="dual-presence"></a>双呈现模式（dual-presence）
> 将同一套内容以两种形式同时维护：CLAUDE.md（项目级，Claude Code 打开项目时自动加载，无需用户干预）和 SKILL.md（agent 可加载版本，供其他 agent 在 frontmatter 中显式声明 `skills:` 引用）。这种设计覆盖了「被动激活」（用户克隆仓库后自动生效）和「主动调用」（agent 按需加载）两种使用场景，代价是需要同步维护两份内容。本仓库是这一模式的典型实现，也是其在大型 skill 仓库中有效性的实证。

### <a name="plugin-json"></a>plugin.json
> Claude Code 插件的「清单文件」（manifest）。位于 `.claude-plugin/plugin.json`，里面列出插件包含的所有 commands、skills、agents 的路径，以及插件的名称、版本、作者等信息。如果 plugin.json 里漏写了某个文件，那个文件即使在磁盘上存在也不会被 Claude Code 加载。本仓库的 plugin.json `skills` 数组正确引用 `"./skills/karpathy-guidelines"`，是 manifest 精准度的范例。
