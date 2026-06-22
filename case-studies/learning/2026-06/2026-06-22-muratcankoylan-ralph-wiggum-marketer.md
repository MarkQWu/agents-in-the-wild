# muratcankoylan/ralph-wiggum-marketer — 学习案例

**仓库**：https://github.com/muratcankoylan/ralph-wiggum-marketer
**Stars**：720 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-06-22（基于当前 HEAD）
**主题标签**：`fallback-chain`, `experience-accumulation`, `single-purpose`, `vague-quantifier`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

[muratcankoylan/ralph-wiggum-marketer](https://github.com/muratcankoylan/ralph-wiggum-marketer) 是一个自主文案写作循环插件，720 stars。其核心设计理念是：用 Claude Code 的 Stop hook 实现"持续文案生产机"——Claude 完成一轮写作后不退出，而是由 Stop hook 拦截退出信号，检查[状态文件](#状态文件)，若存在则强制开启下一轮。内容数据通过 [SQLite](#sqlite) 持久化，形成一个自增长的文案资产库。

三个关键事实：
1. **Stop hook 循环架构**：整个插件的灵魂在于 `hooks/stop-hook.sh`——它检查 `.claude/ralph-marketer-loop.local.md` 是否存在来决定是否延续循环，这是一种把 Claude Code 钩子事件转化为状态机驱动器的经典用法
2. **SQLite 作为内容数据库**：`scripts/src/` 目录下的 Node.js 脚本把每一轮生成的文案写入 SQLite，实现跨会话的内容积累与去重——不依赖对话历史，而是依赖外部持久状态
3. **极小制品集**：7 个 NL 制品（4 命令 + 1 技能 + 1 钩子配置 + 1 清单），职责分明，无冗余——ralph-wiggum-marketer 是"单一用途插件"的典型代表

### 1.2 架构剖析

```
ralph-wiggum-marketer/
├── commands/
│   ├── ralph-marketer.md       # 主命令：启动自主文案写作循环（88/100）
│   ├── ralph-init.md           # 初始化：安装依赖、建立 SQLite 数据库（81/100）
│   ├── ralph-status.md         # 查询：展示内容数据库统计与循环状态（77/100）
│   └── ralph-cancel.md         # 取消：删除状态文件以终止循环（87/100）
├── skills/
│   └── copywriter/
│       └── SKILL.md            # 文案写作技能定义（98/100）
├── hooks/
│   ├── hooks.json              # Stop hook 注册配置（100/100）
│   └── stop-hook.sh            # 循环控制逻辑（含安全漏洞 M1/M2）
├── scripts/
│   └── src/                    # Node.js SQLite 操作脚本
├── templates/                  # 内容模板
├── .claude-plugin/             # 插件元数据
└── plugin.json                 # 插件清单（100/100）
```

**组件关系**：

```
ralph-init.md
    └──→ 建立 SQLite 数据库 + 安装 better-sqlite3

ralph-marketer.md
    └──→ 调用 skills/copywriter/SKILL.md
    └──→ 应当创建 .claude/ralph-marketer-loop.local.md  ← ⚠️ BUG B1：此步骤缺失

hooks/stop-hook.sh
    └──→ 检查 .claude/ralph-marketer-loop.local.md 是否存在
    └──→ 存在 → 注入新一轮写作提示，阻止 Claude 退出
    └──→ 不存在 → 允许正常退出

ralph-cancel.md
    └──→ 删除 .claude/ralph-marketer-loop.local.md → 终止循环
```

三个组件构成完整的状态机：创建状态文件（`ralph-marketer.md`）→ 维持循环（`stop-hook.sh`）→ 销毁状态文件（`ralph-cancel.md`）。但创建侧的步骤在 `ralph-marketer.md` 中自始至终缺席，导致整个循环机制永远无法激活。

### 1.3 设计思路 / 方法论

- **核心哲学**："外部状态驱动自主循环"——不依赖 Claude 的对话上下文维持连续性，而是在文件系统上留下一个明确的[状态文件](#状态文件)作为循环信号。这种设计的优点是状态透明、可被脚本检查，缺点是引入了"谁来创建状态文件"的协调责任
- **SQLite 作为经验积累层**：每一轮生成的文案不会丢失，而是按主题、日期、评分写入数据库，支持后续的内容去重和质量过滤——这是典型的[经验积累模式](#经验积累模式)
- **单一职责边界**：每个命令只做一件事——init 不管写作，status 不管取消，cancel 只删文件。这种边界清晰度是插件保持低复杂度的关键
- **Trade-off**：选择 Shell + Node.js 混合（stop-hook.sh + SQLite 脚本）而非纯 Python，换来的是 SQLite 生态成熟度（`better-sqlite3`）和 hooks 脚本的轻量性；但也引入了变量转义等 Shell 安全坑

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「Stop Hook 状态机 + 外部持久化循环」**

ralph-wiggum-marketer 展示了一种用 Claude Code 钩子事件模拟"无限循环 agent"的方法：不修改 Claude 核心，只在 Stop 事件上挂载一个 Shell 脚本，通过检查外部状态来决定是否重新注入任务。

模式特征清单：
- 状态文件（`.local.md` 后缀）作为布尔开关：存在 = 循环继续，不存在 = 循环停止
- Stop hook 读取状态 → 重新生成提示 → 阻止退出（返回非零退出码或写入 `STOP_HOOK_ACTIVE`）
- 独立的 cancel 命令删除状态文件，提供优雅退出路径
- SQLite 数据库跨循环积累结构化输出，不依赖对话历史

### 2.2 适用场景

- 需要持续运行直到外部干预的内容生成任务（批量文案、报告、代码生成）
- 输出需要持久化并支持后续查询的场景（不能只靠对话历史）
- 希望让 Claude 在无人值守时自主迭代的工作流

### 2.3 与其他架构对比

| 维度 | ralph-wiggum-marketer（Stop Hook 循环） | 传统 loop 命令（如 /loop） | 多 agent 编排（如 ralph-orchestrator） |
|------|----------------------------------------|--------------------------|---------------------------------------|
| 循环触发 | Claude 退出事件 → Stop hook 拦截 | 用户或脚本反复调用命令 | Orchestrator agent 分派子 agent |
| 状态管理 | 文件系统状态文件 | 无（或 scratchpad） | Scratchpad + memory 双层 |
| 持久化 | SQLite 结构化存储 | 无 | 取决于实现 |
| 复杂度 | 低（7 个制品） | 极低 | 高（19+ 制品） |
| 适合任务 | 单一重复型任务 | 手动迭代 | 多任务路由 |

### 2.4 改进空间

1. **BUG B1 修复（最高优先级）**：在 `commands/ralph-marketer.md` 中补充"创建 `.claude/ralph-marketer-loop.local.md`"步骤，否则 Stop hook 永远不会被触发
2. **输出格式缺失**：`ralph-status.md`、`ralph-init.md`、`ralph-cancel.md` 均无 `## Output Format` 章节，Claude 每次运行结果格式不可预测
3. **安全修复**：`stop-hook.sh` 第 96-101 行的 `$SYSTEM_MSG` 应用 `jq --arg` 而非直接插入 JSON heredoc；第 47 行的 `$COMPLETION_PROMISE` 应对 grep 模式转义
4. **依赖锁定**：`package.json` 中 `better-sqlite3: "^11.0.0"` 改为精确版本，避免次版本引入破坏性变更

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）

| 制品 | 路径 | 分数 |
|------|------|------|
| ralph-marketer.md | commands/ralph-marketer.md | 88 |
| ralph-cancel.md | commands/ralph-cancel.md | 87 |
| ralph-init.md | commands/ralph-init.md | 81 |
| ralph-status.md | commands/ralph-status.md | 77 |
| SKILL.md | skills/copywriter/SKILL.md | 98 |
| hooks.json | hooks/hooks.json | 100 |
| plugin.json | plugin.json | 100 |

**整体 NL 评分：90/100**（安全状态：CLEAR，含 2 个 Medium、1 个 Low 安全发现）

### 3.2 当时值得借鉴的模式

- **`skills/copywriter/SKILL.md`（98分）**：frontmatter 完整、职责边界清晰、示例丰富，是高质量 SKILL.md 的标杆——唯一的扣分点是"genuinely good"这一处[模糊量词](#模糊量词)
- **`hooks/hooks.json`（100分）**：Stop hook 注册配置完整规范，钩子事件类型、脚本路径、超时设置均正确——是 hooks.json 写法的参考范例
- **`plugin.json`（100分）**：清单文件信息完整，插件元数据规范
- **单一用途设计**：4 个命令各司其职，没有一个命令承担两种责任，这是命令设计的最佳实践

### 3.3 当时的缺陷

**BUG B1（致命）**：`commands/ralph-marketer.md` 缺少创建 `.claude/ralph-marketer-loop.local.md` 的步骤。`stop-hook.sh` 通过检查此文件来决定是否激活循环——文件不存在，循环永远不会触发。`ralph-cancel.md` 有删除此文件的步骤，但创建者不存在，三角关系中最关键的一角缺失。

**安全发现 M1（Medium）**：`stop-hook.sh` 第 96-101 行将 `$SYSTEM_MSG` 直接插入 JSON heredoc，如果变量内容含双引号或反斜杠，会破坏 JSON 结构，存在注入风险。

**安全发现 M2（Medium）**：`stop-hook.sh` 第 47 行将 `$COMPLETION_PROMISE` 直接作为 `grep` 的模式参数，未对正则元字符转义，若变量含 `.*[]` 等字符会产生非预期匹配。

**安全发现 L1（Low）**：`package.json` 中 `better-sqlite3: "^11.0.0"` 使用宽松版本范围，次版本升级可能引入破坏性变更。

### 3.4 当时的优化机会

- `ralph-status.md`（77分）：缺少 `## Output Format`（−10），步骤未编号（−8），frontmatter 缺少 `allowed-tools`（−5）——三项叠加是四个命令中最低分的原因
- `ralph-init.md`（81分）：缺少 `## Output Format`（−10），frontmatter 声明了 `Write/Read/Glob` 工具但步骤中从未使用
- `ralph-cancel.md`（87分）：缺少 `## Output Format`（−10），声明了 `Write` 工具但仅用 `Bash` 删除文件
- `ralph-marketer.md`（88分）："great content"是典型[模糊量词](#模糊量词)，未定义"great"的可操作标准；循环状态文件创建步骤缺失（见 B1）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

**BUG B1 仍然存在（已确认）**：截至 2026-06-22，`commands/ralph-marketer.md`（85 行）中仍无任何创建 `.claude/ralph-marketer-loop.local.md` 的步骤。对该文件执行全文检索，关键词 `ralph-marketer-loop`、`loop.local`、`状态文件` 均无匹配。循环机制自插件发布以来从未能正常工作。

**安全问题 M1/M2 状态未知**：`stop-hook.sh` 在 2026-06-22 克隆的版本中仍存在，但未收到作者针对安全问题的 commit 记录，推测 M1/M2 仍未修复。

**依赖问题 L1**：`package.json` 中 `better-sqlite3` 版本固定策略未见更新。

### 4.2 架构演进

当前 HEAD 与 2026-04-16 审查时相比，目录结构基本一致：
- `commands/`（4 个文件）、`hooks/`（2 个文件）、`skills/copywriter/`、`scripts/src/`、`templates/`、`.claude-plugin/` 结构保持不变
- `ralph-marketer.md` frontmatter 中当前包含 `allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, WebFetch]` 和 `model: sonnet`——这说明作者在审查后确实更新了 frontmatter，补充了工具声明和模型选择
- 但核心循环逻辑步骤（状态文件创建）仍未补充

**结论**：作者修复了表层的 frontmatter 规范问题，但忽略了深层的设计完整性缺陷（B1）。

### 4.3 新增的可学习模式

- **模型显式声明**：当前版本在 `ralph-marketer.md` frontmatter 中明确 `model: sonnet`，这是一个好习惯——让插件用户知道预期的模型消耗，而不是依赖默认值
- **工具列表完整化**：`allowed-tools` 涵盖 `WebFetch`，表明作者预期文案写作会需要网络检索——这是对命令能力边界的诚实声明

---

## 五、校准

### 5.1 我已经在做对的

- **`echo-sleuth-for-claude` 的工具声明**：在 `dashboard.md`、`audit.md`、`prune.md`、`extract.md` 中均正确声明了 `allowed-tools`，避免了 ralph-wiggum-marketer `ralph-status.md` 的同类错误
- **命令职责单一**：echo-sleuth 同样遵循"每个命令只做一件事"的原则，与 ralph-wiggum-marketer 的最佳实践方向一致
- **理解状态文件的重要性**：看到 B1 之后，我意识到自己在涉及跨组件状态协调时，需要显式地画出"谁创建、谁检查、谁销毁"的三角关系图

### 5.2 挑战 / 验证

- **Stop hook 循环模式在我的项目中是否适用**：`drama-workshop-skills` 有质量控制 Stop hook，但目前只做"拦截并报告"而非"拦截并重新注入任务"。ralph-wiggum-marketer 的模式可以扩展到"批量生成 N 集剧本"的场景，但需要先解决 B1 类问题——即状态文件由谁在何时创建
- **SQLite 持久化的适用门槛**：ralph-wiggum-marketer 的 SQLite 方案引入了 `better-sqlite3` 依赖和 Node.js 运行时，对于轻量级插件来说成本不低。需要验证：我的项目是否真的需要结构化查询，还是 JSONL 追加写入已经够用

---

## 六、行动

### 6.1 自查动作

1. **状态文件三角检查**：对任何涉及 Stop hook 的命令，画出"创建 → 检查 → 销毁"流程图，确认三个角色都有对应的命令/脚本步骤负责
2. **输出格式扫描**：检查我所有命令文件是否包含 `## Output Format` 章节；ralph-status.md 因缺少此章节被扣 10 分，这是高性价比的修复点
3. **Shell 变量注入检查**：审查所有 hook 脚本中将变量插入 JSON 字符串的位置，确认使用 `jq --arg` 而非字符串拼接
4. **依赖版本锁定**：检查 `package.json` 中是否有 `^` 或 `~` 前缀的关键依赖，考虑是否需要锁定到精确版本

### 6.2 灵感 → 实施路径

**灵感：将 Stop hook 用于批量生成**

`drama-workshop-skills` 当前场景：用户一次只生成一集剧本，完成后退出。

实施路径：
1. 在主写作命令（如 `write-episode.md`）末尾增加步骤：若用户传入 `--batch N` 参数，创建 `.claude/drama-batch-loop.local.md`，写入剩余集数
2. 在 Stop hook 中检查此文件：存在则读取剩余数量，减一后写回，重新注入"继续写下一集"的提示
3. 增加 `cancel-batch.md` 命令：删除状态文件，立即终止批量写作
4. 注意：状态文件内容格式需要明确——是纯数字还是 JSON，需在写作命令和 Stop hook 脚本中保持一致

**灵感：SQLite 经验积累**

ralph-wiggum-marketer 用 SQLite 积累每轮生成内容。可以考虑在 `echo-sleuth-for-claude` 中将 session 挖掘结果写入 SQLite，支持按标签、时间范围查询历史挖掘记录——当前的 JSONL 方案已经够用，但如果需要复杂查询则 SQLite 是合理的升级路径。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 目的 | 与 ralph-wiggum-marketer 的关联度 |
|---------|------|----------------------------------|
| `MarkQWu/echo-sleuth-for-claude` | 挖掘历史 Claude 会话中的模式和知识 | **高**：同为循环驱动的状态管理命令插件，均有 commands/ 结构，均依赖跨会话持久状态 |
| `MarkQWu/drama-workshop-skills` | 中文微短剧写作技能库 | **中**：有 Stop hook 质量控制，可借用批量生成循环模式 |
| `MarkQWu/claude-for-legal` | 法律工作流技能与 agent | **低**：技能定义为主，无循环控制需求 |

### 8.2 在我的项目里复现的同类问题

- **`drama-workshop-skills` 的输出格式**：需要确认所有写作类命令是否包含 `## Output Format` 章节。ralph-status.md 因此扣 10 分，这类问题在 NLPM 评分中权重高且修复成本低
- **Stop hook 状态文件协调**：`drama-workshop-skills` 的 Stop hook 目前只做质量拦截，还没有涉及"状态文件创建者"的问题。但一旦扩展到批量生成模式，B1 类问题就会出现。提前设计"三角关系"比事后修复省力
- **工具声明完整性**：echo-sleuth 已经做对了（每个命令都有 `allowed-tools`），但需要定期验证，防止新增命令时遗漏

### 8.3 别人的更优方案

- **ralph-wiggum-marketer 的 `skills/copywriter/SKILL.md`（98分）**比我的技能文件写得更规范：示例充分、输入/输出格式明确、边界清晰。我的 `drama-workshop-skills` 中的 SKILL.md 文件需要对照此标准进行审查——特别是"示例是否足够具体"和"边界条件是否显式说明"
- **Stop hook 作为循环驱动器**：这个模式在 `drama-workshop-skills` 中尚未被利用。ralph-wiggum-marketer 证明了仅用 Shell 脚本 + 状态文件就能实现可靠的 agent 循环，不需要引入复杂的编排框架

### 8.4 反向：我的项目做得比他们好的地方

- **`echo-sleuth-for-claude` 的输出格式规范**：ralph-wiggum-marketer 的 3 个命令缺少 `## Output Format`，导致评分普遍偏低。echo-sleuth 的每个命令都有明确的输出格式定义，这在 NLPM 评分中是直接的加分项
- **循环完整性**：echo-sleuth 不依赖 Stop hook 进行核心循环（而是通过命令内部步骤迭代），因此不存在 B1 类的"状态文件创建者缺失"问题——简单架构反而规避了复杂状态机的协调难题
- **依赖管理**：如果 echo-sleuth 有外部依赖，需要验证是否锁定了精确版本——如果是，则在此维度优于 ralph-wiggum-marketer 的 `^11.0.0` 宽松约束

---

## 八、术语表

**Stop hook**
Claude Code 的钩子事件类型之一，在 Claude 准备退出（结束当前会话或任务）时触发。通过在 `hooks/hooks.json` 中注册对应脚本，可以在 Claude 退出前执行任意 Shell 逻辑——包括检查状态文件、重新注入提示，从而实现"阻止退出并启动下一轮"的循环效果。ralph-wiggum-marketer 的整个循环机制建立在此事件上。

**SQLite**
轻量级嵌入式关系数据库，以单文件形式存储，无需独立服务进程。ralph-wiggum-marketer 通过 `better-sqlite3`（Node.js 绑定）将每轮生成的文案写入 SQLite，实现跨会话的内容积累和结构化查询。相比 JSONL 追加写入，SQLite 支持 SQL 查询（按主题过滤、去重、排序），代价是引入了编译型原生依赖。

**状态文件**
本文中特指 `.claude/ralph-marketer-loop.local.md`——一个仅通过"存在/不存在"来传递布尔状态的文件。`.local.md` 后缀表示它是本地运行时文件，不应被提交到版本控制。Stop hook 通过检查此文件是否存在来决定是否继续循环；`ralph-cancel.md` 通过删除此文件来终止循环。BUG B1 的根本原因是"创建此文件"的责任没有在任何命令中被分配。

**经验积累模式**
一种 agent 设计模式：agent 每次执行后将结构化输出写入持久存储（文件、数据库），后续轮次可以读取历史记录以避免重复、参考先例、或统计质量趋势。ralph-wiggum-marketer 的 SQLite 内容库是此模式的典型实现。与"无状态执行"（每次从零开始）相对，经验积累模式适合需要长期迭代改进的内容生成任务。

**模糊量词**
在 NL 制品（命令、技能、agent 定义）中使用的、缺乏可操作定义的形容词或副词，如"great content"、"genuinely good"、"high quality"。NLPM 评分规则将模糊量词视为扣分项，因为它们无法被 Claude 转化为具体的验收标准——Claude 会自行解释，导致输出不可预测。修复方法：将模糊量词替换为可测量的标准（如"每篇文案包含一个具体数据点、一个行动号召、字数在 150-300 之间"）。

**NL-Binary 混合架构**
一种插件架构模式，其中 NL 制品（SKILL.md、命令文件）作为行为规范层，编译型语言（Rust、Go、C++）或脚本语言（Node.js、Python）作为执行层。ralph-wiggum-marketer 是轻量级混合（Shell + Node.js），ralph-orchestrator 是重度混合（Rust 核心）。与纯 NL 插件相比，混合架构能处理需要精确控制的逻辑（JSON 解析、数据库写入、进程管理），但也引入了语言边界的维护成本。
