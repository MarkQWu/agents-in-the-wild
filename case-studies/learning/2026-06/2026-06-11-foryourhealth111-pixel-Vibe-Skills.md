# foryourhealth111-pixel/Vibe-Skills — 学习案例

**仓库**：https://github.com/foryourhealth111-pixel/Vibe-Skills
**Stars**：1,570 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-11（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `security-gate`, `single-purpose`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`Vibe-Skills` 是一个**超大型 Claude Code skill 集合包**，v2.x（安装包名 `vibe`）。核心是「Vibe Governed Runtime」（VCO）——一套管控用户意图的执行入口，在运行 AI 编程任务前强制锁定需求、规划阶段和验证步骤，防止 AI 随意发散。同时附带 258 个领域 skill，涵盖生物医学数据库（GWAS、ClinVar、PDB）、科学计算（dask、polars、PyTorch Lightning）、文档处理（LaTeX、PPTX）、Windows 工具等。

关键事实：
1. 作者面向科研 + 数据科学社区，大量生物信息学 / 化学数据库 skill（alphafold、clinicaltrials 等）
2. 当前 258 个 bundled skills（审计时 58 个，4.4 倍增长），说明在高速扩张阶段
3. 存在多语言混合：部分 skill 主体为中文（report-generator、video-studio），面向双语用户
4. 包含 Windows 专用 skill（deepagent-memory-fold 硬编码 `C:\Users\羽裳\...` 路径）

### 1.2 架构剖析

```
bundled/skills/         # 258 个 skill（核心资产）
  ├── timesfm-forecasting/SKILL.md   # 高质量示例
  ├── fluidsim/SKILL.md              # 含 bug 的示例
  ├── autonomous-builder/assets/auto-continue.sh  # 安全风险点
  └── <250+ others>/SKILL.md
agents/
  └── templates/         # 4 个 agent 模板（无 frontmatter，不可用）
commands/
  ├── vibe.md            # 主入口
  ├── vibe-implement.md
  ├── vibe-review.md
  ├── vibe-do-it.md      # 新增
  ├── vibe-how-do-we-do.md  # 新增
  └── vibe-what-do-i-want.md
config/opencode/         # OpenCode 格式的对应 agents + commands
  ├── agents/            # 3 个 opencode agent（无 name 字段）
  └── commands/          # 3 个 opencode command
SKILL.md                 # 顶层 vibe 入口 skill（frontmatter 完整）
install.sh / install.ps1 # 安装脚本
scripts/setup/           # Windows credential 持久化脚本（安全风险点）
```

- **文件类型分布**：258 SKILL.md，4 agent 模板（不可用），7 command，3 opencode agent/command
- **编排关系**：`SKILL.md`（vibe 入口）→ 协议文件（`protocols/runtime.md`、`protocols/do.md`）→ 具体 skill。各 skill 完全独立，无跨 skill 引用关系（平铺模式）
- **跨件契约**：主入口 SKILL.md frontmatter 完整（name、description），但 commands 全部缺少 name 字段，不可被 Claude Code 索引

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「Governed Runtime」——不允许 AI 在未锁定需求前开始执行，通过强制需求冻结 + 计划阶段 + 验证关卡来约束模型行为
- **解决什么问题**：「Vibe Coding」导致的 AI 任意发散、不按规矩做事的问题
- **Trade-off**：科学领域 skill 质量极高（90+ 分），但通用工程层（commands、agent templates）结构严重缺失，形成「厚底薄顶」的倒三角
- **认知模型**：作者把 AI agent 看作「可绑定到特定科学数据库的专业助手」，每个 skill 就是一本「怎么用这个工具」的操作手册

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「厚 skill 层 + 薄编排层」（Skill-Heavy, Orchestration-Light）**

关键特征：
- skill 数量极多（258 个），每个 skill 聚焦单一工具或数据库
- commands/agents 层极薄，甚至有结构性缺失（无 name 字段）
- 顶层 vibe 入口 skill 是有效的管控层，但中间层（commands）跳过了 [frontmatter](#frontmatter)
- 科学领域 skill（timesfm、speckit-*）质量是整个项目的天花板

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 科研人员需要对接生物医学数据库 | ✅ 高度适用 | 专业 skill 质量高，覆盖 gwas/clinvar/pubchem 等 |
| 数据科学 workflow（polars/dask/PyTorch）| ✅ 适用 | ML 相关 skill 完整 |
| 需要 agent 团队协作的工程开发 | ❌ 不适用 | agent 层结构缺失，无编排协议 |
| 非 Windows 用户需要 Windows 专用 skill | ❌ 不适用 | deepagent-memory-fold 硬编码 Windows 路径 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 厚 skill + 薄编排 | Vibe-Skills | skill 数量多，领域覆盖广 | 顶层结构缺失，commands 不可索引 |
| 编排+skill 平衡 | first-fluke/oh-my-agent | workflow+skill 职责清晰，agent 有协作协议 | skill 数量少（33 vs 258） |
| 纯高质量精选 | google-gemini/gemini-skills | 4 个 skill 全部 95+，质量极高 | 覆盖极窄 |

### 2.4 改进空间

1. **当前问题**：所有 command 缺少 `name` 字段，Claude Code 无法索引。**改进做法**：为 vibe.md 等 7 个命令文件各添加 1 行 `name: vibe`（文件名即命令名）。**预期收益**：约 +10 分/文件，整体分从 ~79 升至 ~86。
2. **当前问题**：fluidsim/SKILL.md 的 `uv uv pip install` 双重命令 bug 持续。**改进做法**：改为 `uv pip install fluidsim`（删除重复的 `uv`），1 行修改。**预期收益**：用户按文档操作不会报错。
3. **当前问题**：agent templates 无 [frontmatter](#frontmatter)，放在 `agents/` 目录会被误以为是可用 agent。**改进做法**：移至 `docs/examples/` 目录，或加 `---\nname: debugger-template\n---` frontmatter。**预期收益**：避免用户困惑。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

当时整体加权平均约 **79/100**（4 类：agent templates ~30，opencode agents ~55，commands ~58，skills ~85）。

| 类别 | 当时分数 | 主要问题 |
|---|---|---|
| 4 个 agent templates | 30 | 无 frontmatter，-25 名称，-15 无示例 |
| 7 个 commands | 58-60 | -25 名称，-10 无空输入处理 |
| 58 个 skills | ~85 | 主体质量高，allowed-tools 覆盖不足 |
| timesfm-forecasting | 92 | 最高分，质量检查清单完整 |

### 3.2 当时值得借鉴的模式

1. **speckit 套件的 4 阶段串联** → speckit-plan→speckit-specify→speckit-analyze→speckit-implement 形成完整工作链，每个 skill 明确交接什么输出给下一个。路径：`bundled/skills/speckit-*/SKILL.md`。如何借鉴：多 skill 项目用显式「handoff」声明串联。
2. **科学数据库 skill 的代码示例结构** → 每个数据库 skill（clinvar、pubchem 等）都包含「调用示例 → 返回格式 → 错误处理」三段式，满足审计的「examples-driven」要求。路径：`bundled/skills/clinvar-database/SKILL.md`。
3. **timesfm-forecasting 的质量检查清单** → skill 末尾有自检清单，让 AI 在执行后验证输出是否符合预期。路径：`bundled/skills/timesfm-forecasting/SKILL.md`。

### 3.3 当时的缺陷

1. **问题**：所有 46 个 skill 缺少 `allowed-tools` 声明。**根本原因**：作者批量生成 skill 时，没有为每个 skill 思考「它实际会用到哪些工具」，只写了功能描述。**自查**：我的 echo-sleuth SKILL.md 文件是否声明了 allowed-tools？→ 均未声明。
2. **问题**：`autonomous-builder/auto-continue.sh` 默认启用 `--dangerously-skip-permissions`。**根本原因**：作者把「自动化便利性」放在「安全约束」之上，在无监督脚本里不加任何门控就使用最高权限标志。**自查**：我的项目有没有类似无门控的高权限操作？→ 需要检查 drama-workshop 的 install.sh。
3. **问题**：`deepagent-memory-fold` 硬编码 Windows 绝对路径 `C:\Users\羽裳\...`。**根本原因**：作者在自己的机器上开发，把个人路径直接写进公开发布的 skill，没有抽象为可配置的变量。**自查**：我的 skill 有没有硬编码个人路径？→ 可用 `grep -rn "/home/$USER\|~/" skills/` 检查。

### 3.4 当时的优化机会

1. 为 7 个 command 文件各加 1 行 `name:` 字段（最小改动，最大收益）
2. 修复 fluidsim 的 `uv uv pip install` typo（1 行改动，防止用户操作报错）
3. 为 error-resolver 和 excel-analysis 的 name 字段改用 kebab-case（`error-resolver` 而非 `Error Resolver`）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| fluidsim `uv uv pip install` typo | `grep -n "uv uv" bundled/skills/fluidsim/SKILL.md` | **仍存在** — 第 33、36、39 行均有双重 `uv` | 小型可用性 bug 长期未修 |
| agent templates 无 frontmatter | `head -5 agents/templates/debugger.md` | **仍存在** — 文件开头直接是 `# Debugger Agent Template`，无 YAML | 结构问题未修 |
| commands 缺 name 字段 | `grep -L "^name:" commands/*.md` | **仍存在** — 但新增了 vibe-do-it.md 和 vibe-how-do-we-do.md，同样无 name | 甚至在扩张 |

### 4.2 架构演进

从审计时 58 个 skill → 当前 258 个（4.4 倍）。新增大量命令（vibe-do-it.md, vibe-how-do-we-do.md），说明作者在拓展用户操作入口。但结构性问题（agent templates 无 frontmatter、commands 无 name）保持不变。可以判断：**作者的精力主要在扩充 skill 数量，而非修复已知质量问题**。这是一种「广度优先」增长策略，在快速构建用户基础时有效，但技术债在积累。

### 4.3 新增的可学习模式

- **vibe-do-it 和 vibe-how-do-we-do 的存在**揭示了作者对「governed runtime」入口的持续细化——不只是「实现」和「审查」，还有「怎么做」和「直接执行」两个新姿势，说明用户反馈推动了命令层扩展。
- 258 个 skill 中出现了更多「工具 + 代码示例 + API 参考」三合一的 skill 模式，质量上限在提高。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 有 allowed-tools**：echo-sleuth 的命令文件都声明了 allowed-tools，比 Vibe-Skills 的 commands 层更规范。
2. **避免硬编码路径**：echo-sleuth 的 skill 没有个人绝对路径，用相对路径或 `~/.claude/` 这类标准路径。
3. **frontmatter 完整**：echo-sleuth/skills/memory-management/SKILL.md 的 name/description/version 都存在。

### 5.2 挑战 / 验证

本案例挑战了「skill 越多越好」的直觉。Vibe-Skills 以 4.4 倍速度扩张 skill 数量，但核心结构缺陷（command 无 name、agent templates 无 frontmatter）一直未修。**正确的增长顺序是**：先把编排层结构打扎实（10 分钟的修复），再扩 skill 数量。否则底层的可用性问题会随数量增长被放大。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否缺少 allowed-tools
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ -name "SKILL.md" -exec grep -L "allowed-tools" {} \;
# 命中后：为每个 skill 根据它实际调用的工具添加 allowed-tools 字段
```

```bash
# 检查我的 skill 有没有硬编码个人路径
grep -rn "/home/\|C:\\\\Users\\\\" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null
grep -rn "/home/\|C:\\\\Users\\\\" /tmp/my-repos/MarkQWu-claude-for-legal/ 2>/dev/null
# 命中后：替换为 ~/.claude/ 或 $PROJECT_DIR 等可移植路径
```

```bash
# 检查 commands 是否有 name 字段
grep -rL "^name:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/ 2>/dev/null
# 命中后：添加 name: <command-name> 到 frontmatter
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 speckit 套件，为 echo-sleuth 的 extract→synthesize 流程建立显式 handoff 协议
   - **为何可行**：/extract 和 /recap 之间有隐式的数据流，当前没有文档化
   - **第一步**：在 echo-sleuth/CLAUDE.md 中添加「数据流」部分，说明 /extract 输出 JSONL → /recap 读取它

2. **想法**：参考 timesfm 的质量检查清单，为 claude-for-legal 的关键 skill 添加自检清单
   - **为何可行**：法律文件的准确性要求高，自检清单能防止 AI 漏检关键点
   - **第一步**：在 regulatory-legal/skills/reg-feed-watcher/SKILL.md 末尾加 `## Quality Checklist` 5 条项目

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：为科研/数据科学用户提供大规模领域专属 skill 库，并通过 VCO 管控 AI 执行纪律
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是「大量领域专用 skill + 入口管控」模式；都有多个垂直子域（法律 vs 生物医学） | Vibe-Skills 有 command 层（即使不完整）；claude-for-legal 是纯 skill 集合 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 SKILL.md 格式的知识组件 | echo-sleuth 是单一目的工具；Vibe-Skills 是通用框架 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件 | 领域完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 allowed-tools | `grep -rL "allowed-tools" skills/*/SKILL.md` | echo-sleuth: 全部 4 个 SKILL.md 命中（memory-management, git-mining, experience-synthesis, jsonl-core） | 中 |
| agent 无 frontmatter/name | `head -5 agents/templates/*.md` | echo-sleuth: agents 目录有 frontmatter，问题不存在 | N/A |
| commands 缺 name 字段 | `grep -rL "^name:" commands/` | echo-sleuth: commands 有 `allowed-tools` 但需确认是否有 `name` 字段 | 中 |

**命中后的具体行动建议**：
- echo-sleuth/skills/memory-management/SKILL.md：第 5 行后添加 `allowed-tools:\n  - Read\n  - Write\n  - Edit`，10 分钟
- echo-sleuth/skills/git-mining/SKILL.md：添加 `allowed-tools:\n  - Bash`，5 分钟

### 8.3 别人的更优方案

1. **领域**：自检清单（Quality Checklist）
   - **本案例做法**：timesfm-forecasting/SKILL.md 末尾有明确的执行后自检清单，强制 AI 验证输出（路径：`bundled/skills/timesfm-forecasting/SKILL.md`）
   - **我的项目现状**：claude-for-legal/regulatory-legal/skills/ 下的 skill 只描述工作流程，没有验证步骤
   - **如何借鉴**：为 claude-for-legal 的 matter-workspace/SKILL.md 末尾加 5 条自检 checklist

2. **领域**：数据库集成 skill 的三段式结构（调用示例→返回格式→错误处理）
   - **本案例做法**：clinvar-database, pubchem-database 等 skill 都有完整三段式
   - **我的项目现状**：echo-sleuth/skills/jsonl-core/SKILL.md 只有解析规则，没有错误处理示例
   - **如何借鉴**：在 jsonl-core/SKILL.md 添加「错误处理」小节，包含文件不存在、JSON 格式错误的处理示例

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：结构完整性（commands 有 name + allowed-tools）
- **我的做法**：echo-sleuth/commands/extract.md 有 `allowed-tools: Bash, Read, Edit, Write, AskUserQuestion`
- **本案例做法**（弱在哪）：Vibe-Skills 的全部 7 个 command 都缺少 `name` 字段，导致无法被 Claude Code 正确索引
- **意义**：echo-sleuth 的 command 层规范度高于 Vibe-Skills，这是可以在代码审查中作为正向案例引用的做法

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，例如：
> ```yaml
> ---
> name: vibe
> description: Vibe Governed Runtime Entry
> ---
> ```
> Claude Code 读这段配置来决定「这个 skill/agent/command 叫什么名字」「有什么描述」。如果 frontmatter 不存在，Claude Code 就不知道这个文件代表什么，无法注册和调用它。agent templates 没有 frontmatter，就像一件衣服没有标签——放在衣橱里但不知道尺码，穿不了。

### <a name="vague-quantifier"></a>模糊量词
> 像「适当的（appropriate）」「全面的（comprehensive）」「深度的（deep）」这类词，听起来有意义，但 AI 看到后不知道具体要做什么。NLPM 审计会扣分，因为这些词让 AI 自由发挥，每次执行结果可能不同。好的替代方案：「最多 3 个要点」「检查所有超过 50 行的函数」「输出 JSON 格式，包含 type 和 description 两个字段」。
