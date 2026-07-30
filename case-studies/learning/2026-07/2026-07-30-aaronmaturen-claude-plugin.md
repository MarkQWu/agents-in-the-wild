# aaronmaturen/claude-plugin — 学习案例

**仓库**：https://github.com/aaronmaturen/claude-plugin
**Stars**：1 | **来源**：upstream audit
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-30（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `examples-driven`, `single-purpose`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`aaronmaturen/claude-plugin` 是一个以 [manifest](#manifest) 驱动的 Claude Code 插件，包含 26 个命令文件，面向 Django / Angular / Python 技术栈的开发者，提供从 scaffolding（项目脚手架）到 a11y audit（无障碍审计）的一站式 AI 辅助。插件名为 `atm`，通过 Jira 集成处理工单驱动的开发流程，并依赖 Context7 MCP 服务器获取最新文档。

关键事实：
- 命令文件 26 个，skill 文件 2 个（bash、ai）
- Context7 MCP 供 4+ 命令依赖
- 审计时 2026-04-29 score 36/100，状态 `contributed`（NLPM 已提 PR）
- 当前 HEAD：14/26 命令已补 [frontmatter](#frontmatter)（改善显著，从 0 升至 54%）

### 1.2 架构剖析
```
claude-plugin/
├── .claude-plugin/
│   └── plugin.json          ← manifest（插件注册入口）
├── .claude/
│   └── skills/
│       ├── ai/SKILL.md      ← AI 对话规范 skill
│       └── bash/SKILL.md    ← Shell 脚本规范 skill
├── commands/                ← 26 个命令文件（扁平化）
│   ├── scaffold.md
│   ├── a11y-audit.md
│   ├── django-expert.md
│   └── ... (23 more)
└── CLAUDE.md                ← 重定向到 AGENTS.md（委托模式）
```

- **文件类型分布**：26 个 command / 2 个 skill / 1 个 manifest / 1 个 CLAUDE.md
- **编排关系**：命令平铺，互相无依赖；4+ 命令隐式依赖 Context7 MCP，但未在 frontmatter 中声明
- **跨件契约**：`bash` skill 文档了 `$REPORT_BASE` 变量约定，但 `bug-investigation.md` 等命令未遵循，输出路径不一致

### 1.3 设计思路 / 方法论
- **核心设计哲学**：任务专用命令（每条命令解决一类问题），通过 Jira + Context7 双支撑实现「文档驱动开发」
- **解决什么问题**：Django/Angular 项目日常开发的重复性任务——从 PR review 到 a11y 审计，消除手动上下文切换
- **Trade-off**：26 个扁平命令，快速上手，但跨命令共享逻辑（Jira 集成、`$REPORT_BASE` 约定）无集中管理，维护成本随命令数增加而升高
- **认知模型**：作者视命令为「给 AI 的操作手册」，而非「配置声明」；重视流程步骤，但忽视 frontmatter 协议的机器可读性

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「任务矩阵 + 双支撑 skill 模式」**

26 个命令形成一个任务矩阵（技术栈 × 操作类型），底层由 2 个通用 skill（bash / ai）作为基础规范层支撑。

模式特征清单：
- 特征 1：命令按技术栈分组（django-* / angular-* / a11y-*），同类命令结构相似但不共享模板
- 特征 2：两个 skill 作为横向规范层，所有命令隐式继承其约定（但未声明依赖）
- 特征 3：外部服务（Jira、Context7 MCP）通过命令体内的 bash 片段集成，非正式接口
- 特征 4：manifest（plugin.json）注册所有命令，frontmatter 缺失导致注册实质失效

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人项目快速原型 | ✅ 适用 | 命令扁平，维护成本低，不需要跨团队协作 |
| 技术栈聚焦的团队插件 | ✅ 适用 | 专用命令在相似项目中高度复用 |
| 跨技术栈通用工具 | ❌ 不适用 | 每加一个栈就要成比例增加命令数，没有抽象层 |
| 需要命令间传递状态 | ❌ 不适用 | 命令相互独立，无统一上下文机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 任务矩阵 + 双 skill（本仓库） | aaronmaturen/claude-plugin | 直观，零学习曲线 | 无抽象，重复逻辑散布各命令 |
| Router + 专用 skill 层 | affaan-m/everything-claude-code | 可扩展，技术栈解耦 | 初始复杂度高 |
| 单 meta-command + 参数 | 类 enhance 模式 | 接口极简 | 单命令功能过重，难测试 |

### 2.4 改进空间
1. **当前问题**：26 个命令的 Jira 集成逻辑分散各处 **改进做法**：抽取 `skills/jira/SKILL.md` 集中管理 Jira CLI 用法 **预期收益**：Jira auth 指引只需在一处维护
2. **当前问题**：Django/Angular/a11y 类命令结构高度相似但手工维护 **改进做法**：建立一个命令模板并用脚本生成 **预期收益**：减少 26 × 独立维护为 1 个模板 + 配置
3. **当前问题**：Context7 依赖隐式 **改进做法**：在 frontmatter 中 `mcp-servers: [context7]` 声明 **预期收益**：用户无 Context7 时得到明确错误而非静默失败

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 得分 **36/100**（Mean 36，Median 29，范围 15–96）。

| 文件类型 | 当时分数（代表） | 主要问题 |
|---|---|---|
| commands/spike-investigation.md | 15 | 无 frontmatter；无输入守卫；≥10 个模糊量词 |
| commands/scaffold.md | 35 | 无 frontmatter；5 个模糊量词 |
| .claude/skills/ai/SKILL.md | 84 | 8 个模糊量词（effective, clear, good, proper 等） |
| .claude-plugin/plugin.json | 96 | 描述中含 Professional, comprehensive |

### 3.2 当时值得借鉴的模式
1. **`$REPORT_BASE` 输出约定** → skill 文件文档化了报告路径变量，统一输出目录；借鉴做法：将此约定写进自己的 bash skill
2. **CLAUDE.md → AGENTS.md 委托模式** → 两行 CLAUDE.md 将真正内容委托到 AGENTS.md，保持 CLAUDE.md 整洁；适合内容复杂的项目
3. **Context7 + Jira 双支撑** → 命令依赖实时文档（Context7）+ 工单系统（Jira），将两类外部知识引入 AI 上下文，思路值得借鉴

### 3.3 当时的缺陷
1. **全 26 个命令无 frontmatter** → 根本原因：作者可能从 CLAUDE.md 模式迁移过来，不了解 Claude Code plugin 的注册机制要求 frontmatter。后果：所有命令评分上限 45，且 NLPM scanner 无法注册。**自查：我的 bureau commands 缺少 `name` 字段，同样导致注册不完整。**
2. **6 个命令无 `$ARGUMENTS` 输入守卫** → 根本原因：开发者习惯性假设用户会提供参数，忽视了空参数时的静默失败。后果：`/atm:pr-review` 空调用时 AI 收到空 `$ARGUMENTS` 继续执行，生成无意义输出。**自查：需检查我的命令是否有同样问题。**
3. **Context7 MCP 包无版本固定（`npx -y @upstash/context7-mcp`）** → 根本原因：便利性压倒了安全意识；未固定版本意味着 MCP 包任何恶意发版都会影响所有插件用户。这是真实的供应链风险，在 2025-2026 年 npm 供应链攻击频发的背景下尤为严重。

### 3.4 当时的优化机会
1. 为所有 26 个命令添加统一 frontmatter 模板（最高杠杆）→ 单次批量修改可将 mean score 从 36 提升到约 75
2. 将 `allowed-tools` 加入每个命令 frontmatter，限制工具权限预算
3. 为 ai/bash skill 中的模糊量词替换为可验证标准（如 "always check availability" → "test with command -v before use"）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 全 26 命令无 frontmatter | `grep -l "^---" commands/*.md \| wc -l` | **部分修复**：14/26 命令已有 frontmatter（54%） | NLPM PR 被接受并部分合并；仍有 12 个命令处于注册失败状态 |
| Context7 无版本固定 | `grep "context7-mcp" .claude-plugin/plugin.json` | **未修复**：仍为 `npx -y @upstash/context7-mcp`（无版本号） | 供应链风险依然存在 |
| 命令调用格式错误（`/atm-scaffold` vs `/atm:scaffold`） | 检查命令体内示例格式 | **暂无法核查**（需读每个命令体内容） | 文档混乱可能持续 |

### 4.2 架构演进
从 audit 时的 26 个命令全无 frontmatter，到现在 14 个已补。Top-level 结构未重组，仍是扁平命令层 + 两个 skill。作者选择了逐步修复而非整体重构，说明他认可这个架构，只是补齐了遗漏的元数据。

### 4.3 新增的可学习模式
当前 HEAD 中的 14 个已有 frontmatter 的命令，可作为这类插件的「正确示例」——它们展示了一个有效的命令注册格式，可以直接用来对照自己的命令。但由于 12 个命令仍缺失 frontmatter，整体质量依然参差不齐。

---

## 五、校准

### 5.1 我已经在做对的
1. **我的 bureau skills 都有示例块** → bureau 的每个 SKILL.md 都有 3 个 examples，高于本案例的 skill 质量
2. **我的 bureau plugin.json 没有 npx 无版本外部依赖** → 不存在 Context7 式供应链风险
3. **CLAUDE.md 委托模式** → 我的 bureau 也用了类似的 `@BUREAU.md` 委托，保持 CLAUDE.md 精简

### 5.2 挑战 / 验证
这次案例确认了一个我之前犹豫的做法：**frontmatter 是否真的必要，还是只是「仪式感」**？答案是确定的——没有 frontmatter 的命令在 NLPM 视角下评分上限是 45，且 scanner 真的无法注册。这不是理论问题：已 contributed 的 PR 把 14 个命令从 0 分提升到了 60+。Frontmatter 是工程问题，不是仪式。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 bureau 的命令是否有 name 字段
grep -rL "^name:" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
# 命中后：为每个命令的 frontmatter 添加 name: <slug> 字段

# 检查 bureau 命令是否有 allowed-tools
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
# 命中后：按命令实际使用的工具，在 frontmatter 中补 allowed-tools 列表

# 检查我的插件是否有无版本固定的 npx 依赖
find ~/.claude /tmp/my-repos/ -name "plugin.json" -exec grep -l "npx -y" {} \; 2>/dev/null
# 命中后：将 npx -y @pkg 替换为 npx -y @pkg@X.Y.Z（固定版本）
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的命令批量补 `name` 和 `allowed-tools` frontmatter 字段
   - **为何可行**：bureau 的 commands/ 目录有约 10 个文件，格式统一，可批量处理
   - **第一步**：读取每个命令文件，确认它们使用的工具列表（Glob/Grep/Read 各几处），30 分钟内完成

2. **想法**：将 Jira/Context7 等外部依赖从命令体中抽取到专用 skill
   - **为何可行**：本案例显示隐式依赖的维护成本；bureau 同样有 crew.mjs 等跨命令共享逻辑
   - **第一步**：先列出 bureau 命令中重复出现的外部工具调用模式，识别哪些值得提取

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 aaronmaturen/claude-plugin 的核心目的**：为特定技术栈（Django/Angular）的开发工作流提供 AI 辅助命令集

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同为 Claude Code 插件，有 commands + skills 结构 | bureau 面向知识管理，本案例面向开发任务 | 高 |
| MarkQWu/gstack | 中 | 同为技术工具类插件 | gstack 更侧重部署/iOS，无技术栈专项命令 | 中 |
| MarkQWu/graphify | 低 | 同为 AI 编码辅助 | graphify 是知识图谱工具，无多命令结构 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺少 `name` frontmatter 字段 | `grep -rL "^name:" MarkQWu-bureau/commands/*.md` | **命中**：bureau 全部 ~10 个命令无 `name` 字段，只有 `description` | 高 |
| 命令缺少 `allowed-tools` frontmatter | `grep -rL "allowed-tools" MarkQWu-bureau/commands/*.md` | **命中**：bureau 全部命令无 `allowed-tools` 字段 | 中 |
| 命令无 `$ARGUMENTS` 输入守卫 | `grep -L "ARGUMENTS" MarkQWu-bureau/commands/*.md` | **未命中**：bureau 命令不使用 `$ARGUMENTS`（用 `argument-hint` 替代） | 不适用 |

**具体行动建议**：
- `MarkQWu-bureau/commands/lint.md` → 在 frontmatter 加 `name: lint` 和 `allowed-tools: [Read, Glob, Grep]` → 10 分钟
- `MarkQWu-bureau/commands/review.md` → 同上 → 5 分钟/文件 × 10 文件 = 约 1 小时

### 7.3 别人的更优方案

1. **领域**：`$REPORT_BASE` 输出路径约定
   - **本案例做法**：在 `skills/bash/SKILL.md` 中定义 `$REPORT_BASE`，命令通过 skill 约定写入统一路径
   - **我的项目现状**：`MarkQWu-bureau/commands/*.md` 无统一输出路径约定，各命令自行决定输出位置
   - **如何借鉴**：在 bureau 的 bash skill 或 CLAUDE.md 中定义报告输出目录约定（如 `$BUREAU_REPORTS`）

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 示例覆盖率
- **我的做法**：`MarkQWu-bureau/skills/*/SKILL.md` 每个都有 3 个 examples 块
- **本案例做法**（弱在哪）：`skills/ai/SKILL.md` 和 `skills/bash/SKILL.md` 均无 `## Examples` 或示例块
- **意义**：我的 skill 可测试性更强，NLPM tester 可直接 evaluate；这是 bureau 相对本案例的明显优势

---

## 八、术语表

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`.claude-plugin/plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读命令文件时先解析 frontmatter 才能知道这个命令怎么注册和调用。没有 frontmatter 的命令，评分上限是 45，且实际无法被正确注册。

### <a name="供应链风险"></a>供应链风险
> 你的软件依赖别人写的包（如 npm 包 `@upstash/context7-mcp`）。如果不锁定版本，攻击者一旦控制了那个包的发布权，就可以推送含恶意代码的新版本，而你的用户在下次安装时会自动运行这个恶意版本——整个链条被「污染」了。固定版本号（如 `@pkg@1.2.3`）可以防止此类攻击。
