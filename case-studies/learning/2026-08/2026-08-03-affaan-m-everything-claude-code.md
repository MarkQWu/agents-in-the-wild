# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：N/A | **来源**：upstream（≥500 池耗尽，按学习价值补选）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-08-03（基于当前 HEAD）
**主题标签**：`examples-driven`, `vague-quantifier`, `security-gate`, `template-design`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
everything-claude-code（ECC）是本批次 4 个案例中规模最大的仓库（934 个 NL 工件），以覆盖广度为核心竞争力：支持 12+ 自然语言（Korean/Chinese/Portuguese/Turkish/Japanese/German/Spanish/Russian/Thai/Urdu/Vietnamese 等）、兼容 Claude Code 和 Kiro IDE 两个平台、技术栈覆盖从 Flutter/Kotlin/Rust/Go 到 Django/Laravel/Spring Boot/Next.js。其最显著的创新是**「GAN 风格生成-评估迭代循环」**（gan-build 命令族）和分层 [hook](#hook) 系统。

关键事实：
- 2026-04-06 被 NLPM 审计，得分 **84/100**，安全 BLOCKED（7 个发现含 1 CRITICAL）
- 934 个工件，本批次最大规模
- 当前 HEAD 已将 CRITICAL 安全问题（npx block-no-verify 在每次 Bash 调用时外链拉取）替换为本地 Node.js bootstrap 脚本
- skills/ 树质量极高，100+ skill 文件得满分 100/100
- commands/ 树质量偏低，大多缺 name 和 allowed-tools 字段
- 存在多 IDE 支持：`.claude/`（Claude Code）+ `.kiro/`（Kiro IDE）

### 1.2 架构剖析

```
everything-claude-code/
├── agents/                    # 主 agents（40+，多数 85-95/100）
├── commands/                  # 主 commands（72+，多数缺 name 字段）
├── skills/                    # 主 skills（100+，大多数 100/100）
├── hooks/
│   └── hooks.json             # 复杂 PreToolUse/PostEdit/Write hook
├── scripts/hooks/             # 50+ JS 脚本（post-edit-format.js/mcp-health-check.js 等）
├── .claude/                   # Claude Code 专用配置和命令
├── .kiro/                     # Kiro IDE 专用 agents/skills/hooks
│   ├── agents/                # 16 个 Kiro 专用 agents（88-93/100，缺 model 声明）
│   ├── skills/                # Kiro 专用 skills
│   └── hooks/                 # Kiro 专用 hook 配置
├── docs/                      # 超过 12 个语言的本地化副本
│   ├── ko-KR/                 # 韩语
│   ├── zh-CN/                 # 简体中文
│   ├── zh-TW/                 # 繁体中文
│   ├── ja-JP/                 # 日语
│   ├── pt-BR/                 # 葡萄牙语（巴西）
│   ├── tr/                    # 土耳其语
│   ├── de-DE/                 # 德语
│   ├── es/                    # 西班牙语
│   ├── ru/                    # 俄语
│   ├── th/                    # 泰语
│   ├── ur/                    # 乌尔都语
│   └── vi-VN/                 # 越南语
└── examples/                  # 使用样例
```

**文件类型分布**：~40 agents + ~72 commands + ~100+ skills（主） + ~12×40 本地化副本 + 16 .kiro agents + 50+ 脚本

**编排关系**：
- commands/ 是入口；部分命令（gan-build/orchestrate）显式派发 agents（gan-planner/gan-generator/gan-evaluator）
- agents 可调用 skills（通过 `skills:` 字段引用）
- hooks 系统通过 PreToolUse/PostEdit 拦截工具调用，自动格式化、检查质量

**跨件契约**：
- 英文源文件（`agents/*.md`、`commands/*.md`）是本地化副本的「上游」；源文件 bug 自动传播到所有 12 个语言版本
- `.kiro/agents/` 使用 `allowedTools` 字段而非 `tools`，与主 Claude Code 规范不兼容（Kiro vs Claude Code schema 差异）
- GAN 三件套（gan-planner/gan-generator/gan-evaluator）需同时安装才能执行 gan-build 命令

### 1.3 设计思路 / 方法论

**核心设计哲学**：
- **「面向所有人」的覆盖优先**：12 语言 × 2 IDE × 60+ 技术栈 = 最大用户群覆盖
- **[GAN 循环](#gan-loop) 理念**：用「生成→评估→迭代」替代一次性输出，引入有界循环（bounded iterations）防止无限递归
- **「Prompt Defense Baseline」内嵌**：gan-generator.md 在 agent 正文开头嵌入防注入声明，覆盖 Unicode trick、zero-width 字符、context overflow 等攻击向量

**解决什么问题**：让不同语言背景、不同 IDE 的开发者都能获得标准化的 Claude Code 工作流，同时提供超出「对话式 AI」能力的迭代式生成循环。

**做了什么 trade-off**：
- 12 语言本地化导致 bug 传播倍增（源文件 1 个 bug = 12 个副本 bug）
- 多 IDE 支持（Claude Code + Kiro）引入 schema 差异（`tools` vs `allowedTools`），维护双套规范
- skill 和 command 之间的质量差距（skill 100/100 vs command 常见 50-80/100）说明「高质量模板」资源只流向了更稳定的部分

**认知模型**：作者把整个仓库看作「AI 工作流的操作系统」——技术栈是应用（skill），命令是系统调用（command），GAN 循环是进程调度（反复执行直至满足退出条件），hook 系统是内核拦截器。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「多平台多语言 NL 工具集 + GAN 迭代生成」（Multi-Platform Multi-Lingual Toolkit with GAN Loop）**

关键特征：
- 主 artifact 与本地化副本分开目录（docs/），主目录是单一真实来源
- 单一 hooks.json 通过分层 Node.js 脚本处理多种事件类型，而非多个 hooks 文件
- agent 分「主版本」（agents/）和「IDE 专用版本」（.kiro/agents/），结构相同但 schema 字段不同
- GAN 循环：gan-planner 生成规格，gan-generator 实现，gan-evaluator 评分，命令循环直到分数达标或达最大迭代次数

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多语言国际化开发团队 | ✅ 高度适用 | 12 语言本地化覆盖，团队成员可用母语阅读 NL 工件 |
| 同时使用 Claude Code + Kiro 的项目 | ✅ 适用 | 两套配置并存，覆盖两个 IDE |
| 需要迭代型生成（而非一次性输出） | ✅ 独特适用 | GAN 循环是生态中罕见的显式有界迭代设计 |
| 小型单人项目 | ❌ 过度设计 | 934 工件 + 多语言对单人来说维护成本过高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多平台多语言（本案） | affaan-m/ecc | 覆盖最广；GAN 迭代独特 | 规模维护成本高；命令质量偏低 |
| 单语言大型套件 | alirezarezvani/claude-skills | 质量（92/100）更均匀 | 无多语言，无 IDE 双支持 |
| 精巧单插件 | navapbc/dso | 质量最高（84.8/100 plugin score）| 覆盖窄（仅工程团队） |

### 2.4 改进空间
1. **当前问题**：source 文件（commands/test-coverage.md/eval.md 等）缺 name + description，导致 12 语言副本全部继承此 bug（Bug #1-14）。**改进做法**：修源文件 6 个命令，12 语言 × 6 = 72 个文件的 bug 自动消失。**预期收益**：commands 得分平均提升 25 分。

2. **当前问题**：`.kiro/agents/` 使用 `allowedTools` 字段而所有其他 agent 用 `tools`（Bug #18）。**改进做法**：在 `.kiro/` 目录根加 README 说明「Kiro IDE 使用 allowedTools 字段，这是有意为之，不是 bug」，加 REVIEW-DEFENSE 注释。**预期收益**：消除未来维护者的困惑。

3. **当前问题**：zh-TW/ja-JP 本地化版本的 agents 模型声明为 `opus`，而英文源版本是 `sonnet`，导致运行成本 5× 差异。**改进做法**：本地化脚本在生成时统一继承源文件的 `model` 字段而非硬编码 `opus`。**预期收益**：zh-TW/ja-JP 运行成本从 5× 降到 1×。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时 NL 得分 **84/100**。

| 区域 | 得分范围 | 主要问题 |
|---|---|---|
| skills/ 主目录 | 85-100 avg | 多数满分；flutter/dart 类 skill vague quantifier 超限 |
| agents/ 主目录 | 67-98 avg | 部分 agent 零示例；vague quantifier |
| commands/ 主目录 | 23-100 avg | 大量缺 name/description（最低 23/100） |
| .kiro/agents/ | 85-93 avg | 全部缺 model；1 个用错字段名 |
| 本地化副本（docs/） | 50-90 avg | 继承源文件 bug；部分 opus model 成本问题 |

### 3.2 当时值得借鉴的模式
1. **GAN 迭代循环** → `commands/gan-build.md` 实现「生成→评估→迭代」有界循环：gan-generator 实现功能，gan-evaluator 评分，循环直到 score≥threshold 或达 max-iterations。关键：有**退出条件**（分数达标）和**越界保护**（最大迭代次数），防止无限递归。如何借鉴：任何需要「生成→验证→修改」的工作流都可用此模式，关键是定义可量化的退出条件。

2. **`Prompt Defense Baseline` 内嵌安全声明** → `agents/gan-generator.md` 在正文开头嵌入反注入声明，覆盖 Unicode 技巧、context overflow、外部数据 trust 等向量。相比事后审计安全，这种「在 agent 定义时就写入防御规则」的方式更主动。如何借鉴：在所有接受外部输入的 agent 里加「Prompt Defense Baseline」节。

3. **skills/ 的 100/100 标准** → `skills/tdd-workflow/SKILL.md`、`skills/rust-patterns/SKILL.md` 等大量 skill 满分，说明技术约束明确的 skill 质量自然趋向完美。如何借鉴：写 skill 时先确定「这个 skill 的可验证约束是什么」（而非「应该做什么」）。

4. **本地化副本结构** → `docs/<lang>/agents/` 和 `docs/<lang>/commands/` 与主目录结构完全对应。本地化是 diff-apply 模式（在源文件上翻译），而非重写。如何借鉴：如果未来要国际化，建 `docs/<lang>/` 镜像目录，用自动翻译 + 人工校验，而不是手工维护多份。

5. **IDE 多支持（.kiro/ 并列）** → `.kiro/` 目录完全复制了 Claude Code 的 agents/skills/hooks 结构，但适配 Kiro IDE schema。这是「单仓多平台」的工程实现。如何借鉴：若将来同时支持多 AI 编码助手，参考 ECC 的「相同内容，不同格式目录」策略（而非 alirezarezvani 的 GEMINI.md 单文件策略）。

### 3.3 当时的缺陷
1. **问题**：`hooks/hooks.json` 使用 `npx block-no-verify@1.1.2` 在**每次 Bash 工具调用时**从 npm registry 下载并执行外部包（CRITICAL）。**根本原因**：开发者想用 `block-no-verify` 实现 `--no-verify` 拦截功能，但 `npx` 每次调用都可能重新拉取包；即使版本固定为 `@1.1.2`，若 npm registry 被攻击（typosquatting/供应链攻击），每个安装 ECC 的用户的每个 Bash 操作都会受影响。自查：我的 hooks 文件是否在每次工具调用时执行外部脚本？

2. **问题**：英文源命令文件（eval.md/learn.md/test-coverage.md 等）缺 name+description，导致 12 语言 × 6 文件 = 72 个副本全部继承 bug。**根本原因**：本地化流水线复制 source file → 翻译内容，但不修复源文件 bug。本地化副本的质量上限是源文件。自查：如果我的仓库要做本地化，有没有「先修源文件再翻译」的流程？

3. **问题**：zh-TW 和 ja-JP 本地化版 agent 模型硬编码为 `opus`，而英文源是 `sonnet`，造成约 5× 费用差异（Bug 不在 bug 列表，在 cross-component 质量问题）。**根本原因**：翻译 agent 时手工改了 model 字段（或模板默认 opus），没有与源文件同步。自查：我的 skill 的 model 声明是否被其他地方的副本静默更改为更贵的选项？

### 3.4 当时的优化机会
1. 修 6 个源命令文件的 name+description，消灭 72 个本地化副本的同类 bug
2. 将 npx 钩子替换为本地 Node.js 脚本（实际上这在当前 HEAD 已经做到了）
3. 为 agents/gan-evaluator.md 的 `tools` 列表加 `Playwright`（Bug #17），防止运行时工具调用失败

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| hooks/hooks.json CRITICAL npx 外链（Finding #1） | `grep "npx\|block-no-verify" hooks/hooks.json` | **已修复**：当前 hooks.json 使用本地 `node -e` + `scripts/hooks/plugin-hook-bootstrap.js` 替代 npx | CRITICAL 安全问题已解除，仓库安全状态从 BLOCKED 改善 |
| commands/test-coverage.md 缺 name+description（Bug #6） | `head -5 commands/test-coverage.md` | **持续**：当前只有 `description` 字段，无 `name` | 命令仍无法被 Claude Code 正确注册为具名命令 |
| commands/gan-build.md 缺 name+description（Bug #7） | `head -5 commands/gan-build.md` | **持续**：当前只有 `description` 字段，无 `name` | GAN 循环命令仍不可见于命令列表 |

### 4.2 架构演进
当前 HEAD vs 2026-04-06 快照：
- **最大变化**：安全修复。hooks.json 从 npx 外链改为本地 Node.js bootstrap，这是对 CRITICAL 安全 finding 的直接响应
- **新增语言**：docs/ 目录中新增了 de-DE/es/ru/th/ur/vi-VN 等语言（audit 时仅 6 语言，现在 12+），本地化覆盖继续扩大
- **架构稳定**：GAN loop、.kiro/ 并列结构、skills/ 高质量层 均未改变

### 4.3 新增的可学习模式
- **本地 Node.js hook bootstrap 模式**：当前 hooks.json 用内联 `node -e` 动态解析 `CLAUDE_PLUGIN_ROOT`，找到插件根目录后加载本地脚本，而不是依赖外部包。这是「钩子自举」（hook self-bootstrap）的最佳实践——hook 知道自己在哪里，不依赖外部状态。

---

## 五、校准

### 5.1 我已经在做对的
1. bureau 和 gstack 的 hooks 只调用仓库内本地脚本，无外链依赖——与 ECC 修复后的做法一致
2. 我的 skill 没有做本地化，不存在「源文件 bug 传播 12 份」的问题；当前规模不需要本地化是合理决策
3. gstack 明确声明 `model: haiku` 或 `model: sonnet`，不存在 ECC zh-TW 版本的 `opus` 意外升级问题

### 5.2 挑战 / 验证
本案例验证了：**「先修源文件，后做本地化」是本地化 bug 管理的铁律**。ECC 的 6 个命令 bug 因为未在源文件修复，在 12 种语言里各自存在，成为 72 个 bug 实例。如果我将来要国际化任何项目，必须先有「源文件 bug 归零」的 CI 门槛，再开放本地化流水线。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 hooks 文件有没有调用外部 npx 或 curl 的命令
find ~/.claude/ -name "hooks.json" -exec grep -l "npx\|curl.*http\|wget" {} \;
```
命中后：评估被执行的外部资源；用本地脚本替代，或至少确保版本固定且有 checksum 验证。

```bash
# 如果有本地化副本，检查副本的 model 字段是否与源文件一致
# 此处以假设的 docs/zh-CN/ 目录为例
diff <(grep "^model:" agents/*.md 2>/dev/null) \
     <(grep "^model:" docs/zh-CN/agents/*.md 2>/dev/null) 2>/dev/null
```
命中差异：副本的 model 字段被意外覆盖，找到源头并统一。

### 6.2 灵感 → 实施路径
1. **想法**：在 bureau 中实现简化版 GAN 循环——capture-agent 生成知识草稿，eval-agent 评分，循环直到满足「可引用标准」。**为何可行**：bureau 的 capture 和 compile 已有前期工作，加一个评分 agent 和循环指令即可构成 GAN 结构。**第一步**：参考 gan-evaluator.md 的评分标准（可量化的质量维度），在 bureau 的 CLAUDE.md 里定义知识条目的「合格标准」（3 条），约 30 分钟。

2. **想法**：为 gstack 的高风险 agent 加「Prompt Defense Baseline」节，防止提示注入。**为何可行**：gstack 有接受用户输入的 agent，这类 agent 面临提示注入风险；内嵌声明是零成本的防御措施。**第一步**：找 gstack 中最常接受外部输入的 2 个 agent，在正文开头加 5 行防注入声明（模仿 gan-generator.md），约 15 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：面向多语言、多 IDE 的大型 Claude Code NL 工具集，含迭代式生成循环
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 大型 skill/agent 套件，覆盖多技术栈 | gstack 无多语言，规模约 1/30；无 GAN 循环 | 中 |
| MarkQWu/bureau | 低 | 有 hooks，有命令驱动工作流 | bureau 侧重知识管理，无多语言 | 低（安全 hook 模式参考） |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| hooks 外链 npm 包 | `grep "npx" ~/.claude/*/hooks/hooks.json` | 待核查，bureau/gstack 预计无命中（无 npx 使用） | 低（预计已 OK） |
| commands 缺 name 字段 | `grep -L "^name:" ~/.claude/commands/*.md` | 待核查 | 中 |

**命中后的具体行动建议**：
- 若 commands 缺 name → 加一行 `name: <slug>` → NLPM score +25

### 8.3 别人的更优方案

1. **领域**：GAN 迭代循环
   - **本案例做法**：`commands/gan-build.md` + 三件套 agents，实现「生成→评估→迭代→退出」有界循环，退出条件可量化（score threshold）
   - **我的项目现状**：gstack/bureau 的多步任务靠单 agent 线性执行，无迭代验证机制
   - **如何借鉴**：在 bureau 的 compile 命令加评分步骤，若评分不达标则重新 compile，最多 3 次

2. **领域**：Prompt Defense Baseline
   - **本案例做法**：`agents/gan-generator.md` 正文开头的安全声明块，声明拒绝哪些类型的提示注入
   - **我的项目现状**：gstack/bureau 的 agent 无显式安全声明
   - **如何借鉴**：从 gan-generator.md 复制 Prompt Defense Baseline 的结构，加入自己的高风险 agent

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：命令质量一致性
- **我的做法**：gstack 的命令数量少，每个都有完整 frontmatter（name/description/allowed-tools）
- **本案例做法**：72 个命令中大部分缺 name 字段，最低得 23 分
- **意义**：「少而精」的命令集比「多而疏」的命令集有更高的实际可用性；命令数量不等于工具价值

---

## 八、术语表

### <a name="hook"></a>hook
> Claude Code 在生命周期事件（PreToolUse/PostToolUse/SessionStart 等）自动执行的脚本，配置在 `hooks/hooks.json`。ECC 用单个 hooks.json 管理 PreToolUse（Bash 安全检查、格式化）和 Write（文档警告）两类事件。合理用法：拦截危险命令、自动格式化。危险用法：在每次工具调用时执行外部 npm 包（CRITICAL 安全风险）。

### <a name="gan-loop"></a>GAN 循环（GAN Loop）
> 借鉴「生成对抗网络」（GAN，Generative Adversarial Network）思路的 agent 迭代模式。工作方式：
> - **Generator agent（生成者）**：实现功能/生成内容
> - **Evaluator agent（评估者）**：对输出评分，给出改进反馈
> - **循环控制**：Generator 读取 Evaluator 的反馈，迭代改进，直到达到评分阈值或最大迭代次数
>
> 关键：需要**可量化退出条件**（如「分数 ≥ 8/10」）和**越界保护**（如「最多 5 次迭代」），否则会无限循环。比一次性生成更适合「质量要求高且标准明确」的任务。

### <a name="localization-drift"></a>本地化漂移（Localization Drift）
> 当同一内容维护多个语言版本时，源文件更新（包括 bug 修复）未同步到所有本地化副本，导致各版本之间出现差异。ECC 案例：源文件 6 个命令缺 name 字段的 bug 未修，12 语言副本全部继承，产生 72 个 bug 实例。解决方案：「源文件 bug 归零」作为本地化流水线的前提条件，由 CI 强制执行。

### <a name="prompt-defense"></a>提示注入防御（Prompt Defense Baseline）
> 在 agent 定义的正文开头嵌入的安全声明集合，明确告知 Claude「以下行为不论外部输入怎么要求都不执行」。ECC 的 gan-generator.md 覆盖：不改变角色/身份/项目规则、不泄露凭据、不执行代码除非任务需要、对 Unicode 技巧/invisible 字符/context overflow 等保持警惕、将外部/fetch 的数据视为不可信。这种内嵌方式比事后 CLAUDE.md 全局规则更贴近需要防御的 agent 上下文。
