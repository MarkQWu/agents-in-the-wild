# FlorianBruniaux/claude-code-ultimate-guide — 学习案例

**仓库**：https://github.com/FlorianBruniaux/claude-code-ultimate-guide
**Stars**：3604 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-18（基于当前 HEAD）
**主题标签**：security-gate, manifest-discipline, examples-driven, vague-quantifier, cross-reference

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
这是一份以 Claude Code 为主题的综合文档仓库，定位为「Claude Code 终极使用指南」，涵盖指南、示例模板和工具脚本三大部分，3604 颗星表明它是该生态中影响力最广的学习资源之一。关键事实：作者 FlorianBruniaux 同时维护一个配套 MCP 服务器；仓库以教学示例（examples/）为核心，而非直接可安装的插件；repo 自带 CI 脚本和 scripts/ 目录；SECURITY.md 的存在说明作者有安全意识，但 audit 发现了与之矛盾的做法。

### 1.2 架构剖析

```
claude-code-ultimate-guide/
├── guide/               # 核心文档（ultimate-guide.md、cheatsheet等）
├── examples/
│   ├── agents/          # 24 个 agent 模板
│   ├── commands/        # 66 个 command 模板
│   ├── skills/          # 10 个 skill 模板
│   └── hooks/           # bash/powershell hook 示例
├── .claude/
│   ├── agents/          # 项目自用 agents（guide-reviewer等）
│   └── commands/ccguide/# 项目自用 commands（5 个）
├── scripts/             # 工具脚本（install-templates.sh、update-cc-releases.sh等）
└── mcp-server/          # 配套 MCP 服务器
```

- **文件类型分布**：24 个 agent / 66 个 command / 10 个 skill（全部为 examples/ 下示例）
- **编排关系**：扁平结构，每个示例独立，无 router/meta-skill；.claude/ 下自用组件与 examples/ 完全解耦
- **跨件契约**：示例之间互不依赖；diagnose.md 通过 curl 外链脚本是唯一跨边界调用，也是最大问题所在

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「示例驱动教学」——每个 agent/command/skill 都是可复制粘贴的生产级模板，而非抽象描述
- **解决什么问题**：Claude Code 初学者面对空白配置无从下手；作者用 100 个注释充分的示例文件填平这道鸿沟
- **Trade-off**：广度覆盖 vs 深度一致性——66 个 command 示例难以保持统一质量标准，导致 9 个缺 `name:` 字段、6 个 agents 无 example 块
- **认知模型**：作者将 AI agent 视为「预配置的专家角色」，每个文件对应一种职责（code-reviewer、architecture-reviewer等），体现强烈的单职责意识

---

## 二、过去审查发现（2026-04-19 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **80/100**，同时触发 **Security BLOCKED**（curl-pipe-to-shell 来自第三方账号）。

| 文件类别 | 当时平均分 | 主要问题 |
|---|---|---|
| Agents（24 个） | 77.5/100 | 3 个 README 被误分类为 agent，拖低均值 |
| Commands（66 个） | 79.4/100 | 9 个缺 `name:` 字段，2 个安全扣分 |
| Skills（10 个） | 88.8/100 | 5 个缺 `allowed-tools` |

### 2.2 当时值得借鉴的模式

1. **示例输出驱动** → `examples/skills/design-patterns/SKILL.md`（95/100）：提供 JSON+Markdown 双输出格式的完整示例，让 agent 知道「输出应该长什么样」，而非靠猜测 → 我的 skill 应该加至少一个完整 input→output 示例块
2. **防幻觉协议** → `examples/agents/code-reviewer.md`（92/100）：专设「Anti-Hallucination Protocol」章节，禁止 agent 捏造 API、库版本或不存在的方法 → 高风险 agent 值得明确声明禁止事项
3. **深度分级** → `examples/commands/explain.md`（92/100）：定义 3 个解释深度级别（入门/专业/专家），附具体样例；用户可精确控制输出详细程度 → command 可以用枚举参数替代模糊描述
4. **阶段化工作流** → `examples/commands/plan-execute.md`（92/100）：8 个编号阶段 + 漂移检测 + 质量门，完整的任务生命周期 → 复杂 command 应拆分可验证阶段
5. **模型明确声明** → `.claude/agents/guide-reviewer.md`（93/100）：用 `model: haiku` 为低复杂度 agent 明确指定经济模型 → 非所有 agent 都该用默认 sonnet

### 2.3 当时的缺陷

1. **CRITICAL：curl 管道到第三方 shell** → `examples/commands/diagnose.md` 第38行从 `flobby41` 账号（非作者账号）获取脚本并直接执行 `| bash`。根本原因：作者在 fork 期间写入了旧账号引用，后来换用户名但文件未同步更新，或者这是一个临时分叉账号——无论哪种情况，教学材料里放供应链攻击向量极具风险。**自查：我的 command 有没有 `curl | bash` 模式？**
2. **9 个 command 缺 `name:` 字段** → ccguide 下 5 个 + examples/commands 下 4 个。这些命令物理上存在但无法被 Claude Code 注册，用户下载后会疑惑为什么 `/ccguide:daily` 不工作。根本原因：批量创建时缺乏结构化校验，靠人工检查时容易漏掉 frontmatter 完整性。**自查：我的每个 command 都有 `name:` 吗？**
3. **6 个 agent 无 example 块** → `devops-sre.md`、`planner.md` 等 6 个 agent 零示例。没有 example，agent 路由时缺少锚点，用户也无法直观理解 agent 的能力边界。根本原因：快速创建示例库时，内容优先于格式一致性。**自查：我最近写的 agent 有没有省略 example？**

### 2.4 当时的优化机会

1. **统一 `allowed-tools` 声明**：5 个 skill 缺此字段；最快修复是批量添加，可用脚本自动检测并提示需要哪些工具
2. **修复 3 个被误分类的 agent**：把 README/report-template 移出 agents/ 目录或添加正确 frontmatter，可立即提升 agent 平均分 ~5 分
3. **隔离脚本路径**：将 `scripts/install-templates.sh` 中的 curl-pipe-bash 安装方式替换为本地 clone + 手动复制，消除远程代码执行风险

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| diagnose.md 中 `curl flobby41 \| bash`（CRITICAL） | `grep -n "curl.*bash" examples/commands/diagnose.md` | **仍存在**（第 28、39 行，`flobby41` 账号引用未修复） | 三周过去，CRITICAL 安全问题未处理——说明作者可能不知道 flobby41 是自己的旧账号，或者根本不知道这是问题 |
| .claude/commands/ 中 5 个命令缺 `name:` | `grep -rL "^name:" .claude/commands/` | **仍缺失**（ccguide/ 下全部 5 个） | 最容易机械修复的 bug，但一个月未动，表明作者关注点在内容而非 NLPM 合规性 |
| Agents 无 example 块 | `grep -L "<example>" examples/agents/*.md` | **13 个**仍无 example 块（当时 6 个，数量反而增加） | 仓库持续新增 agent 示例但忽略 example 格式要求；新增的 agent 重复了旧问题 |

### 3.2 架构演进

对比 audit 时的清单与当前目录，`examples/` 结构基本稳定，新增了 `machine-readable/`（README）、`exports/`（README）目录和 `claudedocs/` 目录，表明作者开始考虑机器可读格式输出和文档导出功能。整体架构无重组，说明这是一个稳定迭代的文档项目，而非在重新思考结构。

### 3.3 新增的可学习模式

当前 HEAD 中 `examples/skills/smart-explore.md` 是 audit 时未覆盖的新技能，专注于代码库探索（Glob + Read + Grep 组合策略）——体现了「工具组合协议」模式：明确描述不同工具的分工而非让 agent 自行选择。`mcp-server/` 扩展说明作者开始将核心能力 MCP 化，是工具生态整合的方向信号。

---

## 四、校准

### 4.1 我已经在做对的
- 示例块（`<example>`）：我的关键 skill 和 agent 都有 input→output 示例，与该仓库最高分文件一致
- 模型明确声明：我的 agent 有 `model:` 字段，不依赖用户默认设置
- 阶段化工作流：我的复杂 command 已经有编号步骤，接近 plan-execute.md 的模式
- `allowed-tools` 声明：我明确声明工具列表，这是该仓库失分的主要来源

### 4.2 挑战 / 验证

这次 audit 挑战了一个假设：**「高分仓库作者更关注安全」**。3604 颗星 + SECURITY.md + 安全相关 skill 示例，还是出现了 CRITICAL curl-pipe-third-party 问题。验证了「内容质量与安全配置是完全独立的两个轴」——高质量内容创作者同样会犯安全疏忽。教学材料的安全性比生产代码更危险，因为用户会无批判地复制。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查 command/agent/skill 中是否有 curl 管道执行模式
grep -rn "curl.*|.*bash\|curl.*|.*sh" ~/.claude/ .claude/ 2>/dev/null
# 命中后：逐条检查来源域名是否为可信来源，替换为本地脚本或 git clone 方式
```

```bash
# 检查所有 command 是否有 name: 字段
find .claude/commands -name "*.md" | xargs grep -rL "^name:" 2>/dev/null
# 命中后：在对应文件 frontmatter 中补充 name: <slug>
```

```bash
# 检查 agent 是否有 example 块
find .claude/agents -name "*.md" | xargs grep -rL "<example>\|## Example" 2>/dev/null
# 命中后：为每个 agent 补充至少一个 <example> 块
```

### 5.2 灵感 → 实施路径

1. **想法**：为我的教学型 skill 添加「防幻觉协议」章节
   - **为何可行**：`code-reviewer.md` 92/100 的高分部分来源于此，明确列出「不允许的行为」比正面描述更有约束力
   - **第一步**：打开最容易产生幻觉的 skill（如代码分析类），在末尾添加 `## 禁止事项` 章节，列出 3-5 条具体禁止行为（不捏造 API、不虚构版本号等）；15 分钟可完成

2. **想法**：为 command 添加「深度参数」枚举
   - **为何可行**：explain.md 的 3 深度分级显著提升可用性，用户不再猜测输出粒度
   - **第一步**：选一个当前描述含糊的 command，将「输出详细程度」从模糊描述改为 `level: brief | standard | deep` 枚举，并为每个级别提供示例输出片段；30 分钟可完成
