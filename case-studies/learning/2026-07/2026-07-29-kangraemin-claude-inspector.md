# kangraemin/claude-inspector — 学习案例

**仓库**：https://github.com/kangraemin/claude-inspector
**Stars**：115 | **来源**：upstream audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-29（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `cross-reference`, `examples-driven`, `template-design`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Inspector 是一个 **Electron 桌面应用**，用于调试 HTTP 代理流量——尤其用来检测 Claude Code 实际发出了哪些 Claude API 调用（命令、skill 还是 agent？用了哪些 tool？），帮助开发者逆向工程自己的 Claude Code 设置。作者是韩国开发者 kangraemin，README 和 CLAUDE.md 均用韩文撰写。

关键事实：
- 主语言：Electron (Node.js) + HTML/CSS/JS，带 notarize 脚本（macOS 公证）
- 有 Homebrew tap（`homebrew-tap/README.md`），说明有正式分发路径
- NL 层极简：3 个 [agent](#agent)（reviewer、ui-debugger、proxy-analyzer）+ 3 个 skill（e2e、build、deploy）
- CLAUDE.md 内容非常专业——直接描述 proxyDetailView 的 CSS 结构，是真正在用 Claude Code 开发这个 Electron 应用

### 1.2 架构剖析
```
claude-inspector/
├── CLAUDE.md              ← 韩文，描述 UI 调试原则 + CSS 结构约定
├── .claude/
│   ├── agents/
│   │   ├── proxy-analyzer.md   ← 代理流量分析 agent（核心）
│   │   ├── reviewer.md         ← 代码审查 agent
│   │   └── ui-debugger.md      ← UI bug 诊断 agent
│   ├── skills/
│   │   ├── build/SKILL.md      ← 100/100（完美）
│   │   ├── deploy/SKILL.md     ← 100/100（完美）
│   │   └── e2e/SKILL.md        ← 100/100（完美）
│   └── settings.json
├── public/tips.json       ← 应用内 tips 数据
├── scripts/notarize.js    ← macOS 公证脚本（含安全缺陷）
└── package.json           ← Electron 应用配置
```

- **文件类型分布**：3 个 agent / 3 个 SKILL.md / 0 个 command
- **编排关系**：3 个 skill 是**独立、平行**的操作集（build/deploy/e2e），没有 router；3 个 agent 各司其职，proxy-analyzer 是核心（分析 Claude Code 的实际行为）
- **跨件契约**：`reviewer.md` 引用了 `~/.claude/rules/review-rules.md`（用户本地文件，不在仓库内）——这是核心缺陷之一

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「用自己的工具开发自己」——用 Claude Code + 这3个 agent 来开发 Claude Inspector 本身，是典型的自引用工具
- **解决什么问题**：Claude Code 的行为对用户是黑盒；这个工具的代理层让每次 API 调用都可见，可以区分"这是 CLAUDE.md 触发的还是 slash command 触发的"
- **Trade-off**：选择了「agent 数量极少 + skill 质量高」的模式，而不是覆盖一切操作
- **认知模型**：把 Claude Code agents 当作「专用调试工具」——每个 agent 对应一个调试视角（流量分析 / UI bug / 代码审查）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「高质量工具层 + 专用调试 agent」**：skill 层（build/deploy/e2e）打满分，而 agent 层（3个）作为领域专用的调试工具，skills 负责工程操作，agents 负责诊断。

模式特征（4 条）：
- **特征 1**：skill 和 agent 职责严格分开——skills 是可重复的操作（build、test），agents 是需要推理的诊断
- **特征 2**：skill 质量极高（3 个全部 100/100），但 agent 质量极差（3 个平均 47/100）——这个反差说明作者重视使用体验但忽视了注册规范
- **特征 3**：CLAUDE.md 面向具体技术细节（CSS 结构）而不是项目概述——只写了 Claude 实际需要知道的信息
- **特征 4**：reviewer agent 依赖外部用户文件（`review-rules.md`）——依赖了仓库外的文件系统

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 开发工具类 Electron 应用 | ✅ 高度适用 | CLAUDE.md 专注技术细节；skills 匹配开发操作 |
| 需要团队共享 NL 配置的项目 | ❌ 不适用 | reviewer 依赖 `~/.claude/rules/review-rules.md`，换台机器就失效 |
| 个人开发工具（单台机） | ✅ 适用 | 外部文件引用在个人机器上有效 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 最小 agent 集 + 完美 skill（本仓库） | claude-inspector | 每个 skill 打磨精良；维护负担低 | agent 完整性低（缺 name 字段） |
| 全面 agent 集（大而全） | navapbc/digital-service-orchestra | 覆盖完整 SDLC 流程 | 体积大；维护成本高 |
| 纯 skill 无 agent | vladikk/modularity | 结构最简单 | 无法处理复杂推理任务 |

### 2.4 改进空间
1. **当前问题**：3 个 agent 的 [frontmatter](#frontmatter) 全部缺 `name:` 字段，导致 Claude Code 无法注册 **改进做法**：每个 agent 的 `---` 块里加一行 `name: proxy-analyzer`（类推其余 2 个） **预期收益**：agents 从「无法注册」变为「可用」；得分从 43-55 提升至 75+
2. **当前问题**：`reviewer.md` 引用 `~/.claude/rules/review-rules.md`（外部文件） **改进做法**：把 `review-rules.md` 提交到仓库内，用相对路径引用，或在 README 里写明需要手动创建这个文件 **预期收益**：消除「freshclone 时 reviewer agent 悄悄失效」的问题
3. **当前问题**：3 个 agent 全部无示例（zero examples） **改进做法**：为 proxy-analyzer 添加 1-2 个「输入：某段 API 流量 → 输出：机制分类（CLAUDE.md / skill / sub-agent）」的示例 **预期收益**：agent 的 NLPM 得分从 43 提升至 75+

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-19 当时得分 **76/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| proxy-analyzer.md | 43 | 缺 `name` frontmatter + 零示例 + 无 output format |
| ui-debugger.md | 45 | 缺 `name` frontmatter + 零示例 + 无 output format |
| reviewer.md | 55 | 缺 `name` frontmatter + 零示例 |
| CLAUDE.md | 88 | 内容清晰具体，仅少量小问题 |
| e2e/SKILL.md | 100 | 完美 |
| build/SKILL.md | 100 | 完美 |
| deploy/SKILL.md | 100 | 完美 |

### 3.2 当时值得借鉴的模式
1. **Skill 100 分实现** → build/deploy/e2e 三个 skill 全部 100 分 → 根本原因：这三个 skill 有完整的 frontmatter、清晰的 output format、具体的命令描述、而且工具声明精准 → 借鉴：把这三个作为「skill 标准模板」学习，尤其是 e2e/SKILL.md
2. **CLAUDE.md 专注性** → 只写 CSS 结构和调试原则，没有废话 → 根本原因：作者知道 Claude 只需要「不能从代码里推断出来的信息」 → 借鉴：检查我的 CLAUDE.md，删掉能从代码读到的冗余信息
3. **Agent 职责分离** → proxy-analyzer / ui-debugger / reviewer 各负责不同诊断维度 → 根本原因：单一职责让每个 agent 的上下文保持聚焦，不互相干扰

### 3.3 当时的缺陷
1. **3 个 agent 全部缺 `name:` 字段** → 根本原因：作者把 agent 文件当"给人看的文档"写，忘记了 Claude Code 需要通过 `name:` 字段来注册和调用 agent → 自查：我的 agent 文件是否全部有 `name:` 字段？（我目前无 agents，暂无风险）
2. **reviewer.md 引用外部文件** → `~/.claude/rules/review-rules.md` 在 freshclone 时不存在 → 根本原因：作者在本机开发时那个文件存在，没有意识到仓库里的文件不能依赖机器上的个人文件 → 自查：我的仓库里有没有引用 `~/.claude/` 路径？（发现 drama-workshop-skills 有类似引用）
3. **零示例 + 无 output format（agents）** → 根本原因：可能作者把 `description:` frontmatter 当成了"已经够了"，没有意识到 NLPM 还要求 body 里有 Examples 和 Output Format → 自查：如果我未来写 agents，需要记住这两个必须项

### 3.4 当时的优化机会
1. 为所有 3 个 agent 添加 `name:` 字段（每个 30 秒，零风险）
2. 把 `review-rules.md` 提交到仓库，或在 README 里写明依赖
3. 为 proxy-analyzer 添加 2 个 input→output 示例（最有价值的 agent）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个 agent 缺 `name:` 字段 | `grep "^name:" .claude/agents/*.md` | **全部持续**（3 个文件均只有 `description:`，无 `name:`） | 3 个 agent 至今无法被 Claude Code 注册 |
| reviewer.md 引用外部 `review-rules.md` | `grep "review-rules" .claude/agents/reviewer.md` | **持续**（第 17 行：`~/.claude/rules/review-rules.md`） | freshclone 后 reviewer agent 依然静默失效 |
| 3 个 agent 零示例 | 查看 agent 文件 body | **持续**（无 Examples section） | 得分未改善 |

### 4.2 架构演进
- 自 audit 以来，仓库**几乎没有变化**：3 个 skill（build/deploy/e2e）、3 个 agent，结构完全相同
- `scripts/notarize.js` 存在但未修改安全缺陷（凭据在进程列表可见）
- 没有新增命令或 skill
- 说明：作者可能已停止活跃开发，或以 agent 形式使用它而对注册缺陷无感（本机上如果手动配置 name 就能用）

### 4.3 新增的可学习模式
当前 HEAD 与 audit 时几乎无变化。暂无新增可学习模式。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的 skill 文件（MarkQWu/gstack 里的 SKILL.md）都有 frontmatter 声明——与本仓库的 skill 100 分实践一致
2. 我的仓库内没有 agent，避开了「缺 name 字段」这个陷阱
3. 我的 CLAUDE.md（bureau 项目）专注于约束而非泛泛介绍——方向与本仓库一致

### 5.2 挑战 / 验证
- **验证**：`name:` 字段缺失让 agent 「静默失效」——这比报错更危险，因为用户以为 agent 在用，但实际上 Claude Code 根本没有加载它。这验证了一个原则：**NL artifact 的功能性错误比质量问题危险得多**，因为质量问题让效果变差，而注册错误让工具直接不可用。
- **挑战**：我原以为只要 `description:` 写清楚，agent 就能被发现。实际上 `name:` 字段是必填的注册 key——没有它，description 再好也没用。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库里有没有 agent 文件缺少 name 字段
find /tmp/my-repos/MarkQWu-*/ -name "*.md" -path "*/agents/*" | xargs grep -L "^name:" 2>/dev/null

# 检查有没有外部路径引用（~/.claude/）
grep -rn "~/.claude/" /tmp/my-repos/MarkQWu-*/ --include="*.md" 2>/dev/null | grep -v README | grep -v ".gitignore"

# 检查 drama-workshop-skills 的外部引用
grep -rn "~/.claude/\|~/.codex/" /tmp/my-repos/MarkQWu-drama-workshop-skills/ --include="*.md" | head -10
```

命中后怎么办：
- agent 缺 `name:` → 立即添加（30 秒/文件），这是功能性 bug 不是质量问题
- 外部 `~/.claude/` 引用 → 评估：是安装路径（合理）还是功能依赖（危险）？依赖场景要么提交到仓库要么写进 README

### 6.2 灵感 → 实施路径
1. **想法**：参考 kangraemin 的 build/deploy/e2e skill 结构，为 MarkQWu/- 的 SKILL.md 建立一个「完美 skill 模板」
   - **为何可行**：这三个 skill 是少见的 100/100 实现，结构值得直接模仿
   - **第一步**：Read `kangraemin/claude-inspector/.claude/skills/e2e/SKILL.md`，提取模板结构，应用到 MarkQWu/- 的第一个 SKILL.md（15 分钟）

2. **想法**：如果未来在任何项目里加 agent，先建立「agent 最小可用 checklist」
   - **为何可行**：本案例证明缺 `name:` 就注册失败，最小 checklist 能防止这类静默 bug
   - **第一步**：写一个 5 项 checklist（`name:` / `description:` / `model:` / body 有示例 / output format），放进 CLAUDE.md 或 notes（10 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例的核心目的**：调试 Claude Code 的 API 调用行为；用 Electron 桌面应用可视化代理流量
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/- | 低 | 都是工具型项目；都有 NL agents 的潜在需求 | 我的是情报仪表板，不是调试工具 | 低（架构差异大） |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件；有 CLAUDE.md | bureau 不调试 API 流量 | 低 |

我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 缺 `name:` 字段 | `find MarkQWu-*/ -path "*/agents/*.md" -exec grep -L "^name:" {} \;` | **未命中**（我无 agent 文件） | 无 |
| 外部文件引用（~/.claude/） | `grep -rn "~/.claude/" MarkQWu-drama-workshop-skills/` | **命中**（drama-workshop-skills 多处引用 ~/.claude/skills/） | 中（安装路径引用，合理，但需确认是安装后的路径而非开发依赖） |
| agent 零示例 | 无 agent → N/A | **未命中** | 无 |

**命中后的具体行动建议**：
- `MarkQWu/drama-workshop-skills` → 检查 `~/.claude/skills/` 引用是安装目的地（合理）还是运行时依赖（危险）；在 README 里明确标注哪些路径需要用户手动创建

### 8.3 别人的更优方案

1. **领域**：skill 的 100/100 完整实现
   - **本案例做法**：build/deploy/e2e 的 SKILL.md 全部包含完整 frontmatter（`name`、`description`、`model`、`allowed-tools`）+ 清晰的步骤描述 + 精准的工具声明
   - **我的项目现状**：MarkQWu/gstack 的 SKILL.md 有 frontmatter，但 allowed-tools 未经验证是否精准
   - **如何借鉴**：逐一对照 gstack 的每个 SKILL.md，验证 `allowed-tools` 里的工具是否真的在 body 里被调用（30 分钟）

2. **领域**：CLAUDE.md 技术精准性
   - **本案例做法**：CLAUDE.md 直接写 CSS 选择器（`#proxyDetailView`）、具体 JS 方法（`container.style.cssText`）
   - **我的项目现状**：bureau 的 CLAUDE.md 描述概念，较少具体技术细节
   - **如何借鉴**：在 bureau 的 CLAUDE.md 里加入「关键数据结构」和「已知技术约束」小节，给 Claude 提供无法从代码推断的关键信息

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent 注册完整性
- **我的做法**：我暂无 agent 文件，因此不存在缺 `name:` 字段的问题
- **本案例弱点**：3 个 agent 全部因缺 `name:` 字段而无法注册，这是功能级别的 bug
- **意义**：我在引入第一个 agent 时要把 `name:` 作为第一项检查；这也是未来开 PR 给别人时一个高置信度的 bug fix 切入点

---

## 八、术语表

### <a name="agent"></a>agent
> Claude Code 里的「专用子代理」。在 `.claude/agents/` 目录下用 Markdown 写成，每个 agent 有自己的 `name`、`description`、`model` 和工具声明。Claude 在需要时可以把一个子任务派给某个 agent 执行，agent 就像一个有固定专长的「工作组成员」。`name:` 是必填字段，缺了它 Claude Code 找不到这个 agent。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（`name`、`description`、`model`、`allowed-tools` 等）。Claude Code 先读这里才知道这个 skill/agent/command 是谁、能干什么。缺少必填字段（如 `name`）会导致组件无法被注册，但通常不会有报错提示——是「静默失效」。
