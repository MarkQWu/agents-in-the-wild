# KhazP/vibe-coding-prompt-template — 学习案例

**仓库**：https://github.com/KhazP/vibe-coding-prompt-template
**Stars**：2278 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-19（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `examples-driven`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

KhazP/vibe-coding-prompt-template 是一套面向"vibe coding"（快速原型 + 感觉驱动开发）场景的 Claude Code [skill](#skill) 工作流模板，核心价值是把"把想法变成 MVP"这条路径拆成 5 步可执行的技能链，每步有时间估算和明确的交接物。作者身份是社区贡献者（非 Anthropic 员工），通过 GitHub 分发，直接将技能文件放在 `.claude/skills/` 目录下供用户复制使用。2278 Stars 说明它命中了"AI 辅助快速建站"这个高需求场景。

### 1.2 架构剖析

**目录结构**：
```
vibe-coding-prompt-template/
├── .claude/
│   ├── skills/
│   │   ├── vibe-workflow/SKILL.md   ← 主编排者（master orchestrator）
│   │   ├── vibe-research/SKILL.md
│   │   ├── vibe-prd/SKILL.md
│   │   ├── vibe-techdesign/SKILL.md
│   │   ├── vibe-agents/SKILL.md
│   │   └── vibe-build/SKILL.md
│   └── hooks/hooks.json             ← 自动化钩子（格式化 + 安全守护 + 通知）
├── part1-deepresearch.md
├── part3-tech-design-mvp.md
└── README.md
```

**文件类型分布**：6 个 SKILL.md，0 个独立 agent 定义，1 个 hook 配置，0 个 [manifest](#manifest)（无 plugin.json）。

**编排关系**：分层结构，`vibe-workflow` 是路由层，显式调用其余 5 个 sub-skill。工作流链路：`vibe-research` → `vibe-prd` → `vibe-techdesign` → `vibe-agents` → `vibe-build`。`vibe-workflow` 的第一步是读取项目文件状态来判断从哪个阶段恢复，这是一个"断点续传"设计。

**跨件契约**：5 个子技能通过 markdown 文件系统耦合（`docs/research-*.md`、`docs/PRD-*.md`、`docs/TechDesign-*.md`、`AGENTS.md`），orchestrator 在 Step 1 读取这些文件来推断当前进度。跨件之间没有显式版本号或接口声明。

### 1.3 设计思路 / 方法论

**核心设计哲学**：时间盒化的线性流水线。每个阶段都有时间估算（Research 20 分钟、PRD 15 分钟等），把 AI 辅助开发从"随机漫步"变成"有节拍的流程"。

**解决什么问题**：用户启动 AI 辅助开发项目时缺乏结构感——不知道从哪开始、做到什么程度才算完成一个阶段、如何在多次对话之间保持连贯性。

**做了什么 trade-off**：选择了"单仓库 + 手动复制安装"而非 plugin 注册机制，降低了分发复杂度但牺牲了版本管理能力。技能链是线性的而非图状，简单直观但缺少分支和错误路径。

**反映什么认知模型**：作者把 AI 技能设计成类似"SOP（标准操作程序）"而非"智能代理"——技能提供框架，AI 在框架内执行，用户控制节奏。这与"让 AI 自主决策"的方向有意地保持距离。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「时间盒化线性流水线」**

这个仓库的架构模式是：将一个开放式任务（"从想法到 MVP"）切割成有时间预算的线性阶段，每个阶段是一个独立技能，由一个 orchestrator 技能统一调度和状态感知。

模式特征清单：
- **有时间预算**：每个阶段明确标注预计耗时，让用户知道投入产出比
- **文件系统作为状态机**：通过检查 `docs/` 目录文件是否存在来判断阶段完成度
- **单一入口**：`vibe-workflow` 是用户的唯一接触点，其余技能可以单独用也可以通过 orchestrator 链式调用
- **交接物明确**：每阶段输出特定的文件（research 报告、PRD 文档等），这些文件既是当前阶段的产出，也是下一阶段的输入
- **断点续传**：`vibe-workflow` Step 1 检查现有文件，允许用户中断后从任意阶段恢复

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 快速验证产品想法（周末 hackathon） | ✅ 高度适用 | 时间盒子 + 线性步骤正好匹配有限时间窗口 |
| 长期产品开发（数月周期） | ❌ 不适用 | 缺少迭代循环和版本管理，线性流程与敏捷迭代冲突 |
| 团队协作项目 | ⚠️ 有限适用 | 没有多人并发保护，文件状态机在多人操作下会混乱 |
| 教学场景（AI 开发入门） | ✅ 适用 | 步骤清晰、有时间预算，适合新手建立 AI 辅助开发的心智模型 |
| 纯 API 集成项目（无前端） | ❌ 不适用 | 部分技能假设有 UI 设计需求，对纯后端项目有空跑阶段 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 时间盒化线性流水线（本仓库） | KhazP/vibe-coding-prompt-template | 上手简单、节奏感强、状态可视化 | 无分支、无错误路径、无迭代机制 |
| Router + 并行 Channels | eyaltoledano-claude-task-master | 任务可并发执行，适合复杂项目管理 | 设置复杂度高，学习曲线陡 |
| 单一重型 SKILL | mattpocock-skills | 单文件部署最简单 | 难以模块化，上下文消耗大 |

### 2.4 改进空间

1. **当前问题**：技能链只支持顺序推进，无法回退或跳步。**改进做法**：在 `vibe-workflow` 中增加"重做上一步"和"跳到指定阶段"的命令。**预期收益**：减少用户因思路变化而重启对话的情况。

2. **当前问题**：5 个子技能没有 `model` 字段声明，Claude Code 加载时使用默认模型，可能超出性能/成本预期。**改进做法**：为 Research/PRD 声明 `model: claude-sonnet`，为 Build 声明 `model: claude-opus`（计算量最大的阶段）。**预期收益**：成本可控，且为高计算阶段提供更强的推理能力。

3. **当前问题**：状态检测（Step 1）依赖文件路径硬编码（`docs/research-*.md`），用户一旦重命名文件就会失效。**改进做法**：改为检测文件内容特征（如特定标题或 frontmatter 标记）而非文件名 glob。**预期收益**：对用户自定义文件名更宽容。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 **80/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| vibe-workflow/SKILL.md | 75 | 缺 `allowed-tools`；无示例块 |
| vibe-build/SKILL.md | 78 | 缺 `allowed-tools`；模糊量词；无足够示例 |
| vibe-agents/SKILL.md | 80 | 无 `model` 字段；无示例块 |
| vibe-prd/SKILL.md | 80 | 无 `model` 字段；无示例块 |
| vibe-research/SKILL.md | 80 | 无 `model` 字段；无示例块 |
| vibe-techdesign/SKILL.md | 80 | 无 `model` 字段；无示例块 |
| hooks/hooks.json | 90 | 安全缺陷（命令注入） |

### 3.2 当时值得借鉴的模式

1. **「可恢复的 orchestrator」模式** → 设计良好：Step 1 先读取现有文件状态再决定从哪步开始，避免用户重复劳动。借鉴点：任何跨会话 workflow 技能都应有状态感知恢复逻辑。路径：`vibe-workflow/SKILL.md` §Step 1。

2. **「时间盒子标注」** → 在技能开头给出时间预估（20 min, 15 min...），直接影响用户对投入的心理预期。借鉴点：为复杂技能添加"预计执行时间"提示，让用户知道什么时候可以去倒杯水。

3. **「跨件一致性」** → audit 发现 5 个子技能的交接文件引用全部对齐，无孤儿组件，Cross-Component 检查通过。借鉴点：多技能系统中，建立显式的「产出文件清单」以便自动验证。

### 3.3 当时的缺陷

1. **所有 5 个子技能缺少 `model` 字段** → 为什么会失败：Claude Code 加载技能时若无 model 声明，使用 session 默认模型，可能在高负载时降级到更弱模型，导致输出质量不稳定。自查：我的 drama-workshop-skills 的 2 个 SKILL.md 同样无 model 字段——**命中**。

2. **`vibe-workflow` 和 `vibe-build` 缺少 `allowed-tools`** → 为什么会失败：技能体内明确调用了 `Read`/`Glob`（读取 docs 文件）和 `Bash`/`Write`（npm 命令和文件写入），但 frontmatter 未声明这些工具，运行时 Claude Code 可能静默阻止工具调用，导致 orchestrator 在 Step 1 就失败。

3. **全部技能无示例块（<example>）** → 为什么会失败：用户不知道如何触发技能、预期什么格式的输出。`vibe-workflow` 作为入口技能，缺少 input→output 示例意味着用户需要反复试错。自查：echo-sleuth-for-claude 的 4 个 SKILL.md 均无示例块——**命中**。

4. **Notification hook 存在 shell injection** → 将 `d.message` 直接拼接到 `osascript` 命令字符串中，攻击者可通过构造含 shell 元字符的通知消息执行任意命令。这是真实的可利用漏洞。

### 3.4 当时的优化机会

1. `vibe-build/SKILL.md` 中的 "relevant"（line 29）、"brief"（line 41）、"minimal"（line 63）替换为可测量标准，如 "the agent_docs/ file matching the current sprint phase"。
2. 为 `vibe-workflow` 增加"当前状态报告"示例，让用户知道 orchestrator 检测到项目处于哪个阶段时的输出格式。
3. 在 `allowed-tools` 里声明最小权限集（`vibe-workflow` 只需 `Read`, `Glob`；`vibe-build` 需要 `Bash`, `Write`, `Read`）。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 5个技能缺 `model` 字段 | `grep -r "^model:" .claude/skills/` | **仍存在**：grep 返回空，5 个技能均无 model 声明 | 6 个月过去，这个问题未被修复，可能作者认为 model 声明是可选的 |
| `vibe-workflow` 缺 `allowed-tools` | `grep "allowed-tools" .claude/skills/vibe-workflow/SKILL.md` | **仍存在**：frontmatter 无该字段 | 工具调用静默失败的风险依然存在 |
| Notification hook shell injection | 检查 hooks.json Notification 段 | **仍存在**：`msg` 仍直接拼接到 osascript/notify-send/PowerShell 命令中（未使用 execFile 或参数数组） | 中危安全漏洞持续 6 个月未修复；HIGH 级别的 PostToolUse 注入已通过 `JSON.stringify(p)` 改写得到部分缓解 |
| 无示例块 | `grep -r "<example>" .claude/skills/` | **仍存在**：全部 6 个技能无 `<example>` 标签 | 用户体验痛点，新用户必须阅读 README 才能知道怎么触发 |

### 4.2 架构演进

对比 audit 时的文件清单和当前 HEAD：目录结构完全一致，无新增技能，无删除技能，无重组迹象。`part1-deepresearch.md` 和 `part3-tech-design-mvp.md` 是"前插件时代"的 markdown 版本，至今保留作为替代方案。这说明作者将这个仓库视为"稳定模板"而非"持续演进项目"。

值得注意的是：hooks.json 的 PreToolUse 段（保护 .env 和 package-lock.json 不被直接修改）是正确且值得学习的设计，且 audit 期间就存在，当前 HEAD 保留了这个设计。

### 4.3 新增的可学习模式

暂无。当前 HEAD 与 audit 快照结构相同，6 个月内无明显更新。唯一值得记录的是：`PostToolUse` hook 对 `file_path` 的处理已从直接字符串拼接改为用 `JSON.stringify` 包裹（消除了 HIGH 级别注入），但 Notification hook 的 MEDIUM 级别问题仍未修复。说明作者对安全问题有部分意识但处理不完整。

---

## 五、校准

### 5.1 我已经在做对的

1. **明确的触发条件描述**：`description` 字段用"Use when..."句式——我的 drama-workshop-skills 也采用了类似约定（`description: "当用户说..."` 触发格式）。✓
2. **单职责技能**：每个 vibe-* 技能只做一件事（Research / PRD / Tech Design...），我的 short-drama 系列也保持了 short-drama 和 short-drama-remake 的职责分离。✓
3. **交接物明确**：每个技能产出的文件路径在上下技能中保持一致，我的 drama 系列也有对应的输出路径约定。✓

### 5.2 挑战 / 验证

**挑战了什么假设**：我原来认为"有时间标注的 workflow 是过度设计"，但这个仓库的 2278 stars 证明用户明确需要时间感知——知道"Research 需要 20 分钟"会帮助用户规划注意力，而不是一直盯着屏幕等待。

**验证了什么**：skills 之间通过文件系统（而非通过内存或变量）传递状态是切实可行的——这个架构经过了真实用户的检验。我在 drama-workshop-skills 中也依赖文件输出（`.org` 文件）作为阶段产物，方向正确。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否都声明了 model 字段
find ~/.claude/skills -name "SKILL.md" | xargs grep -L "^model:"
# 命中后：在 frontmatter 中添加 `model: claude-sonnet-4-6`（具体根据技能复杂度选择）

# 检查我的 SKILL.md 是否都有 allowed-tools 声明
find ~/.claude/skills -name "SKILL.md" | xargs grep -L "allowed-tools"
# 命中后：对照技能体内实际用到的工具，在 frontmatter 中补全最小权限声明

# 检查我的技能是否有示例块
find ~/.claude/skills -name "SKILL.md" | xargs grep -L "<example>"
# 命中后：为每个无示例块的技能添加 1-2 个 User: / Assistant: 格式示例
```

### 6.2 灵感 → 实施路径

1. **想法**：给 drama-workshop-skills 的主技能加时间盒子标注（类似"分集纲要约需 30 分钟"）。
   - **为何可行**：不需要改逻辑，只需在 SKILL.md 开头的概述段加一行时间预估。
   - **第一步**：编辑 `short-drama/SKILL.md`，在"工作流概览"部分每个阶段后加 `（约 N 分钟）`。约 10 分钟完成。

2. **想法**：为 drama-workshop-skills 添加状态感知恢复逻辑（类似 vibe-workflow Step 1）。
   - **为何可行**：drama-workshop 的阶段产物（`.org` 文件、导出文件）已有固定命名模式，可以用 Glob 检查。
   - **第一步**：在主技能的"初始化"阶段加一个判断块：如果发现 `~/Documents/notes/分集纲要*.org` 存在，跳过策划和分集步骤。约 30 分钟完成。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 KhazP/vibe-coding-prompt-template 的核心目的**：把"AI 辅助从想法到 MVP"这条路径结构化为可执行的 5 步技能链，让没有项目管理经验的开发者也能有节奏地推进。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同样是"把一个复杂创作流程拆成多步技能"；同样有 orchestrator 主入口 + 子技能链 | drama-workshop 是内容创作领域；vibe-template 是技术产品领域 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同样是多技能协作系统，有 agents + skills 两层 | echo-sleuth 的技能是并列的而非线性链；目标是知识管理而非工作流执行 | 中 |
| MarkQWu/claude-for-legal | 低 | 都有 skill 体系 | claude-for-legal 是领域知识型，不是 workflow 型 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 `model` 字段 | `grep -L "^model:" .../skills/*/SKILL.md` | **drama-workshop-skills 命中 2/2**（short-drama、short-drama-remake 均无 model 声明） | 中 |
| 缺 `<example>` 示例块 | `grep -L "<example>" .../skills/*/SKILL.md` | **drama-workshop-skills 命中 2/2**；**echo-sleuth 命中 4/4** | 高 |
| 缺 `allowed-tools` 声明 | `grep -L "allowed-tools" .../skills/*/SKILL.md` | **drama-workshop-skills 命中 2/2**（技能体内用到 Bash、Write、Read，但无声明） | 中 |

**命中后的具体行动建议**：
- `drama-workshop-skills/short-drama/SKILL.md` → 在 frontmatter 末尾添加 `model: claude-sonnet-4-6` 和 `allowed-tools: Bash, Read, Write, Glob` → 约 5 分钟
- `drama-workshop-skills/short-drama/SKILL.md` + `short-drama-remake/SKILL.md` → 各添加 1 个完整 `<example>` 块（User 触发语 + Assistant 预期输出摘要） → 约 20 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：workflow 断点续传
   - **本案例做法**：`vibe-workflow/SKILL.md` Step 1 用表格列出所有阶段对应的检查文件，Claude 读取文件系统后自动定位当前阶段（路径：`.claude/skills/vibe-workflow/SKILL.md` §Step 1 Assess Current State）
   - **我的项目现状**：drama-workshop-skills 没有状态感知逻辑，用户每次需要手动告知"我已经完成了策划，现在要写分集"
   - **如何借鉴**：在 short-drama SKILL.md 的开头增加一个状态检测块，通过 Glob 查找 `~/Documents/notes/策划-*.org` 等文件来判断当前进度

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：多语言和市场分层支持
  - **我的做法**：drama-workshop-skills 明确区分国内模式和出海模式（`genre-guide.md`、`ai-live-rules.md`），输出时自动切换内容规范
  - **本案例做法**：vibe-coding-template 完全没有国际化或市场分层的概念，假设所有用户都在同一市场
  - **意义**：若未来给上游 PR，可以提议增加"regional template"分支

---

## 八、术语表

### <a name="skill"></a>Skill（技能）
> Claude Code 中一种 Markdown 文件格式（SKILL.md），放在 `.claude/skills/` 目录下，用来告诉 Claude "当用户说出特定词语时，按照这个文件里描述的步骤执行"。类比：餐厅里的标准作业卡，服务员按卡片上的步骤为顾客服务，不用每次从头思考。

### <a name="manifest"></a>manifest（清单文件）
> 项目的"目录"，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest。如果某个 SKILL.md 没有被 manifest 引用，它就不会被自动加载，相当于菜单上没有列出的菜——厨房有，但顾客点不到。本仓库没有 plugin.json，靠用户手动复制 `.claude/skills/` 目录来安装。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md [frontmatter](#frontmatter) 中的一个字段，声明这个技能需要使用哪些 Claude Code 内置工具（如 `Read`、`Bash`、`Write`）。若不声明，Claude Code 在执行时可能拒绝工具调用，导致技能静默失败。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道技能的名称、描述、允许的工具等。
