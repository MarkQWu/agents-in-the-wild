# alirezarezvani/claude-code-skill-factory — 学习案例

**仓库**：https://github.com/alirezarezvani/claude-code-skill-factory
**Stars**：721 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-29（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `template-design`, `cross-reference`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-code-skill-factory 是一个"元插件"——它的核心功能不是提供直接的业务 skill，而是帮助用户**创造自己的 skills、hooks 和 agents**。作者 Alireza Rezvani 把仓库定位为参考库和工厂模板：clone 下来，用里面的 5 个 guide agents 指导你生成适合自己项目的 NL 工件。

关键事实：
1. **5 个专用 guide agents**：skills-guide、hooks-guide、factory-guide、prompts-guide、agents-guide，每个都是一位"NL 工程顾问"
2. **14 个已生成的 skills**（`generated-skills/`），同时作为使用示例
3. **模块化 CLAUDE.md**：根据当前工作目录自动加载不同的上下文文件（`.github/CLAUDE.md`、`claude-skills-examples/CLAUDE.md` 等），是少见的"上下文敏感 CLAUDE.md"设计
4. **状态：contributed**（registry 显示 contributed），说明 NLPM 已对该仓库发起 PR

### 1.2 架构剖析

```
claude-code-skill-factory/
├── CLAUDE.md                        # 根 CLAUDE（导航总图，指向模块）
├── .github/
│   ├── CLAUDE.md                    # GitHub workflow 上下文
│   └── EMERGENCY_CLEANUP.sh         # 紧急清理脚本（低危安全问题）
├── .claude/
│   ├── agents/                      # 5 个 guide agents
│   │   ├── factory-guide.md
│   │   ├── skills-guide.md
│   │   ├── hooks-guide.md           # ⚠️ 含硬编码开发者路径
│   │   ├── prompts-guide.md
│   │   └── agents-guide.md
│   └── commands/                    # 11 个已注册 + 9 个未注册命令
│       ├── ci-guard.md              # 有 frontmatter（description + argument-hint）
│       ├── review.md, security-scan.md, run-release.md ... 
│       └── test-factory.md, install-skill.md ... (9 个无 frontmatter)
├── generated-skills/                # 14 个生产就绪 skills
│   ├── hook-factory/, prompt-factory/
│   ├── agent-factory/, slash-command-factory/
│   └── [10 个领域 skills]
├── claude-skills-examples/          # 参考实现示例
│   └── CLAUDE.md
└── documentation/
    ├── CLAUDE.md
    └── references/, templates/      # 官方文档 + 模板
```

- **文件类型分布**：14 个 SKILL.md / 11 个有注册 command / 9 个无注册 command / 5 个 agent
- **编排关系**：guide agents 指导用户使用 templates/ 创建工件，generated-skills/ 展示输出结果
- **跨件契约**：`generated-skills/CLAUDE.md` 作为生成 skill 的目录，但 Audit 时只收录 9 / 14 个 skills，5 个 skill 没有被文档化

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**"授人以鱼不如授人以渔"**——不是直接给用户一套固定的工具，而是教用户如何制造工具。这是少见的 meta-level 工具设计
- **解决什么问题**：大多数 Claude Code 用户不知道如何从零写出高质量的 skill/agent/hook，需要有人（或有 AI）手把手指导
- **Trade-off**：factory 模式让通用性和灵活性最高，但代价是仓库本身的"即用性"低——用户不能直接 `claude plugin install` 就得到有用的功能，必须先学习再生成
- **认知模型**：作者把 NL 编程视为"需要学习的工艺"，而非"直接使用的工具"，因此设计了培训路径而非直接工具包

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「教学型 Factory 仓库」**：核心资产不是可直接使用的工件，而是指导创建工件的 agents 和模板。仓库本身既是老师，也是参考书，也是工厂流水线。

模式特征清单：
- **特征 1**：guide agents 把隐性知识（如何写一个好 hook）显式化为可执行的顾问对话
- **特征 2**：模块化 CLAUDE.md 让不同目录有不同的上下文，减少信息噪音
- **特征 3**：generated-skills/ 既是输出示例也是可直接使用的 skill
- **特征 4**：documentation/references/ 包含官方文档抄本，确保 factory 顾问的知识是权威的
- **特征 5**：CHANGELOG.md 记录了所有版本演进，是工厂自身的学习记录

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| NL 编程新手学习项目 | ✅ 高度适用 | guide agents 降低了入门门槛 |
| 团队统一 skill 写法标准 | ✅ 适用 | 模板 + 顾问 agent 确保一致性 |
| 需要快速使用 skill 的用户 | ❌ 不适用 | 需要先学习 factory 再生成，成本高 |
| 单一领域深度工具 | ❌ 不适用 | factory 太宽泛，专用工具更合适 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 教学型 Factory（本案例）| claude-code-skill-factory | 通用性最高、授人以渔 | 即用性低、学习成本高 |
| 开箱即用 plugin | alexgreensh/token-optimizer | 立即可用 | 解决特定问题，不可扩展 |
| 超级框架 | SuperClaude_Framework | 功能全面 | 学习曲线陡峭 |

### 2.4 改进空间

1. **当前问题**：9 个 commands 缺 frontmatter，无法被 Claude Code 注册（截止当前 HEAD 仍存在）**改进做法**：批量加最小 frontmatter，factory 本身的命令不可用是最严重的矛盾 **预期收益**：修复后用户可以用 `/test-factory` 等命令验证他们创建的 skill

2. **当前问题**：hooks-guide.md 硬编码 `/Users/rezarezvani/projects/` 路径（截止当前 HEAD 仍存在）**改进做法**：改用 `$(pwd)` 或 `$CLAUDE_PROJECT_DIR` 动态替换 **预期收益**：任何用户安装后 hooks-guide 的步骤 5-6 可以正常执行

3. **当前问题**：`generated-skills/CLAUDE.md` 只记录了 9/14 个 skills **改进做法**：在生成每个新 skill 时自动更新 CLAUDE.md 目录 **预期收益**：避免用户误以为"看到的就是全部"

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **80/100**，安全等级 REVIEW（非 CLEAR，因为有 Medium + Low 安全发现）。

| 文件类型 | 平均分 | 主要问题 |
|---|---|---|
| agents (5) | 85/100 | 模糊量词（comprehensive×3、seamless），hooks-guide 硬编码路径 |
| commands (11 有注册) | 89/100 | 缺 allowed-tools |
| commands (9 无注册) | 66/100 | 无 frontmatter（最严重） |
| skills (14) | 85/100 | 模糊量词高密度（comprehensive×4+），tech-stack-evaluator 达 8 处 |

### 3.2 当时值得借鉴的模式

1. **模块化 CLAUDE.md** → 根 CLAUDE.md 作为导航总图，各子目录有自己的 CLAUDE.md，Claude 根据工作目录自动加载相关上下文 → `CLAUDE.md` 中的 Navigation Map 表格 → 适合体量大的仓库，避免 CLAUDE.md 臃肿
2. **factory agents 的顾问模式** → hooks-guide、skills-guide 等把"如何创建"的知识固化成 agent，用户通过对话完成学习+创建 → `.claude/agents/` 目录 → 用 agent 来传授知识而不是靠 README
3. **prompt-factory 的 69 个预设** → 明确列出 69 个分领域 prompt 预设，用分类法（domain taxonomy）组织，便于检索 → `generated-skills/prompt-factory/SKILL.md` → 具体枚举比"丰富的预设"这种描述有效 100 倍
4. **documentation/references/ 官方文档存档** → 把 Claude Code 官方 Agents、Skills、Hooks 文档存入 repo，确保 guide agents 引用的是权威来源 → `documentation/references/` → 在有外部知识依赖的 skill 里，把关键参考文档本地化

### 3.3 当时的缺陷

1. **9 个 commands 无 frontmatter** → 这些命令就像"店门上没有招牌"，描述再好也无法通过 `/` 触发 → 问题根因：命令文件是渐进式添加的，没有统一的 checklist 确保 frontmatter 完整 → 自查：我的 echo-sleuth 所有 commands 已验证有 frontmatter

2. **hooks-guide.md 硬编码开发者路径** → Steps 5-6 的 `cd /Users/rezarezvani/projects/claude-code-skills-factory` 让任何非作者的用户在执行时直接报"目录不存在"→ 问题根因：在自己机器上测试过的代码直接提交，没有意识到路径是个人相关的 → 自查：我的任何 agent 或 command 中有没有硬编码的绝对路径？

3. **模糊量词高密度** → "comprehensive"、"intelligent"、"seamless" 这三个词在所有生成的 skills 中都大量出现，像是 factory 模板本身就在复制这些词 → 问题根因：用于生成 skills 的 prompt 模板没有"禁止模糊量词"的约束，导致系统性扩散 → 自查：如果我用 factory 或模板批量生成工件，要检查模板本身是否有这类词

### 3.4 当时的优化机会

1. 为 9 个无 frontmatter 的 commands 批量添加最小 frontmatter（最高优先级）
2. hooks-guide.md 中用 `$(pwd)` 替换硬编码路径
3. 更新 `generated-skills/CLAUDE.md` 目录以覆盖全部 14 个 skills

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| test-factory.md 无 frontmatter | `head -5 .claude/commands/test-factory.md` | **仍存在**（以 `# /test-factory` 直接开头） | 高价值 command 仍无法被注册 |
| hooks-guide.md 硬编码路径 | `grep -n "rezarezvani" .claude/agents/hooks-guide.md` | **仍存在**（lines 253, 259 仍含完整路径） | 任何非作者的用户执行 Steps 5-6 会报路径错误 |
| ci-guard.md 缺 allowed-tools | `head -8 .claude/commands/ci-guard.md` | **部分修复**（有 description + argument-hint，但仍无 allowed-tools） | 轻微改善但未达标 |

### 4.2 架构演进

当前 HEAD 相比 audit 时 ci-guard.md 等命令已加了 `description` 和 `argument-hint`，说明作者开始补充 frontmatter，但优先补充了有 frontmatter 的命令而不是那 9 个完全没有 frontmatter 的命令。`.archive/` 目录的出现说明作者在清理旧的 session 文件，项目管理更规范化。

### 4.3 新增的可学习模式

**`.archive/` 目录管理过期工件**：把旧的 test 文件和 session 摘要归档到 `.archive/`，而不是直接删除。这是长期项目的"数字考古"做法——保留历史但不污染主工作区。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 完整 frontmatter**：echo-sleuth 的所有 command 均有 frontmatter，比 skill-factory 还在"部分修复"的状态强
2. **无硬编码路径**：我的 agents 和 commands 使用 `$CLAUDE_PLUGIN_ROOT` 等环境变量，不硬编码绝对路径
3. **模块化 SKILL 设计**：echo-sleuth 每个 skill 专注一件事（experience-synthesis 就是经验综合，不做其他）

### 5.2 挑战 / 验证

**验证了模式**：模块化 CLAUDE.md 是个好主意。skill-factory 用 Navigation Map 表格清楚地说明每个 CLAUDE.md 在什么情境下被加载，这和我的 echo-sleuth 的 CLAUDE.md 只有一层相比更有扩展性。当仓库体量增长时，要考虑拆分 CLAUDE.md。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 中是否有硬编码绝对路径
grep -rn "/Users/\|/home/\|/root/" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/ \
  /tmp/my-repos/MarkQWu-drama-workshop-skills/ \
  2>/dev/null | grep -v ".git"
```
命中后怎么办：将硬编码路径替换为 `$CLAUDE_PLUGIN_ROOT`、`$(pwd)` 或 `$HOME` 等环境变量。

```bash
# 检查我的仓库是否有模糊量词
grep -rn "comprehensive\|seamless\|intelligent\|robust" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ --include="*.md" \
  2>/dev/null | grep -v "README\|CHANGELOG\|.git"
```
命中后怎么办：将"comprehensive analysis"改为具体描述，例如"analysis covering values, decisions, mistakes, patterns, and questions — one item per line"。

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 添加类似 factory 的模块化 CLAUDE.md
   - **为何可行**：drama-workshop-skills 有 short-drama 和 short-drama-remake 两个平行模块，当前只有一个根 README，没有上下文分割
   - **第一步**：在 `short-drama/` 目录创建 `CLAUDE.md`，专注描述国内短剧工作流；在 `short-drama-remake/` 创建单独的 `CLAUDE.md`，约 20 分钟

2. **想法**：在 echo-sleuth 中加一个 "guide agent"（类似 factory-guide）来帮助用户理解如何配置和使用 echo-sleuth
   - **为何可行**：echo-sleuth 的 agents 目前都是"执行型"，没有"教学型"的 agent 帮助新用户入门
   - **第一步**：创建 `agents/setup-guide.md`，描述如何第一次配置 echo-sleuth 的完整流程，约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 alirezarezvani/claude-code-skill-factory 的核心目的**：提供 NL 工件（skill/hook/agent）的生成工厂，帮助用户学习并创建自己的 Claude Code 工具

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code skill 集合 | skill-factory 是元工具，drama-workshop 是直接业务工具 | 低 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都是 Claude Code 插件，都有多个 agent | 领域完全不同 | 低 |
| MarkQWu/claude-for-legal | 中 | 都是"一套 skills 集合"的组织方式 | claude-for-legal 直接提供法律 skills，而非教如何创建 | 中 |

我的仓库中无目的相近的项目，本节主要做技术模式对照。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 frontmatter | 已检查，全部有 | 未命中 | N/A |
| 硬编码绝对路径 | `grep -rn "/Users/\|/home/" /tmp/my-repos/MarkQWu-*/` | **未命中** | N/A |
| 生成 skill 的模板有模糊量词 | `grep -rn "comprehensive" /tmp/my-repos/MarkQWu-claude-for-legal/ --include="*.md" \| head -5` | claude-for-legal 中无命中（法律术语更精确）| N/A |

### 8.3 别人的更优方案

1. **领域**：模块化 CLAUDE.md（Navigation Map）
   - **本案例做法**：根 CLAUDE.md 有一张 Navigation Map 表格，明确列出子目录 CLAUDE.md 的用途和加载时机
   - **我的项目现状**：claude-for-legal 每个子目录（ai-governance-legal、regulatory-legal 等）有各自的 CLAUDE.md，但根目录的 CLAUDE.md 没有说明这种关系
   - **如何借鉴**：在 claude-for-legal 的根 CLAUDE.md 中加一个"子目录 CLAUDE.md 导航"表格，约 10 分钟

2. **领域**：documentation/references/ 官方文档本地化
   - **本案例做法**：把 Claude Code 官方 Skills/Agents/Hooks 文档存入 `documentation/references/`，agents 引用时读本地文件而非访问网络
   - **我的项目现状**：echo-sleuth 中没有本地化官方文档，新功能开发时需要实时查官网
   - **如何借鉴**：在 echo-sleuth 中创建 `docs/references/claude-code-hooks-spec.md`，把相关官方规范本地化，约 15 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：commands 完整注册可用性
- **我的做法**：MarkQWu/echo-sleuth-for-claude 所有 8 个 command 文件均有完整 frontmatter（name、description、allowed-tools），可以直接使用
- **本案例做法**：9 个 commands 无 frontmatter，截至当前 HEAD 仍未修复，用户无法通过 `/test-factory` 等命令使用工厂核心功能
- **意义**：这是 factory 仓库最大的反讽——教别人如何做好 NL artifact，自己却有 9 个无法注册的命令。echo-sleuth 的完整注册率（100%）是更好的实践范例。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明 `name`、`description`、`allowed-tools` 等元数据。没有 frontmatter 的 command 文件无法被 Claude Code 识别和注册为 `/command-name` 斜杠命令，即使内容再好也无法通过 `/` 前缀调用。

### <a name="argument-hint"></a>argument-hint
> command frontmatter 中的可选字段，向用户提示该命令接受哪些参数格式（如 `[ref]`、`[scope]`）。不影响注册，只影响 Claude Code UI 中的提示文字。
