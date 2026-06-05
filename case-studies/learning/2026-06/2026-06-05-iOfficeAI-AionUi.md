# iOfficeAI/AionUi — 学习案例

**仓库**：https://github.com/iOfficeAI/AionUi
**Stars**：22,516 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-05（基于当前仓库 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `curl-pipe-bash-risk`, `template-design`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

AionUi 是 iOfficeAI 出品的 **AI 桌面/Web 应用框架**（22,516 stars），让用户可以在本地运行 AI agents 和 skills。与大多数纯 NL 制品仓库不同，AionUi 是一个完整的软件产品——包含 Electron 桌面应用、Web UI、mobile 客户端，以及一个 Claude Code `.claude/skills/` 自动化套件供开发者使用。

关键事实：
- 当前 HEAD（2026-06-05）结构与 audit 时（2026-04-06）有**重大差异**：`src/process/resources/skills/`（包含 officecli-* 技能家族）整个目录已被移除，重构为 `packages/desktop`、`packages/web-cli`、`packages/web-host` 等模块化结构
- 现存 NL 制品：26 个（10 个 `.claude/skills/`、10 个 `examples/` 下的 skills/agents/assistants、6 个其他 .md）
- `.claude/skills/` 目录是一个高质量的 Claude Code **自动化 PR 工作流套件**，覆盖 review → fix → ship → verify 全链路

### 1.2 架构剖析

```
AionUi/                               ← 桌面/Web AI 应用框架
├── CLAUDE.md                         ← 单行委派 @AGENTS.md
├── AGENTS.md                         ← 实际项目上下文（90/100）
├── .claude/
│   ├── skills/                       ← 高质量 PR 自动化套件（10 个 skills）
│   │   ├── pr-automation/SKILL.md    ← 总编排：标签状态机驱动
│   │   ├── pr-review/SKILL.md        ← 代码审查
│   │   ├── pr-fix/SKILL.md           ← 修复建议
│   │   ├── pr-ship/SKILL.md          ← 合并发布
│   │   ├── pr-verify/SKILL.md        ← 验证
│   │   ├── testing/SKILL.md          ← 测试（90/100）
│   │   └── ...
│   └── commands/
│       └── package-assistant.md      ← BUG：硬编码 /Users/veryliu/ 路径
├── packages/
│   ├── desktop/                      ← Electron 桌面应用
│   ├── web-cli/                      ← Web CLI
│   └── web-host/                     ← Web 服务器
└── examples/
    ├── hello-world-extension/        ← BUG：两个 agent 无 frontmatter
    └── e2e-full-extension/           ← 完整集成示例
```

- **文件类型分布**：10 个 skill、6 个 agent/assistant、1 个 command、1 个 project（CLAUDE.md）
- **编排关系**：`.claude/skills/pr-automation/SKILL.md` 是整个 PR 工作流的总编排，通过标签状态机（`pr-review-requested → pr-reviewed → pr-fix-requested → pr-ship`）协调其他 skills
- **跨件契约**：所有 PR 相关 skills 通过统一的 HTML 注释标记（`<!-- pr-automation-bot -->`、`<!-- pr-review-bot -->`）相互识别，并约定工作目录路径（`/tmp/aionui-pr-*`）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「协议化跨件通信」——所有 PR skills 通过 HTML 注释标记互相识别和传递状态，而不是依赖文件名或隐式约定
- **解决什么问题**：在 Claude Code 中实现全自动化的 PR 生命周期管理——从代码审查到修复到发布，无需人工干预每个步骤
- **做了什么 trade-off**：`.claude/skills/` 的高质量 vs `examples/` 的低质量——开发者自己用的工具做得很好，但示例代码写得很粗糙（两个 demo agent 完全没有 frontmatter）
- **反映什么认知模型**：作者非常清楚「状态机 + 标签驱动」是 AI 工作流的可靠模式，但在安全意识上存在明显盲点（curl|bash 的危险没有被识别为风险）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「标签状态机 + 跨件协议标记」

特征清单：
- **标签驱动工作流**：GitHub PR 标签变化触发不同的 skill 执行，不需要手动指定「下一步做什么」
- **HTML 注释作为通信协议**：skills 通过往 PR 评论里写入特定 HTML 注释来传递状态，其他 skill 通过 grep 这些注释来感知上下文
- **工作目录约定**：所有 PR 相关操作使用固定路径模式（`/tmp/aionui-pr-<issue-number>`），避免路径冲突
- **单职责 skill**：每个 skill 只做一件事（只审查、或只修复、或只发布），职责边界清晰

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 团队 PR 流程自动化 | ✅ 高度适用 | 标签状态机 + skills 的组合完全覆盖 PR 生命周期 |
| 个人项目快速迭代 | ✅ 适用 | `.claude/skills/pr-review` 单独使用就很有价值 |
| 需要 curl\|bash 安装依赖 | ❌ 不适用 | 这是高风险反模式，即使方便也不应该这样做 |
| 跨团队共享工具库 | ❌ 需谨慎 | `package-assistant.md` 的硬编码路径问题说明团队内部工具没有做可移植设计 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 标签状态机（本仓库） | iOfficeAI/AionUi .claude/skills | 工作流完全自动化、状态透明可追踪 | 实现复杂、依赖 GitHub PR 生态 |
| 平铺 skills | addyosmani/agent-skills | 简单直接、每个 skill 独立可用 | 多步骤工作流需要手动串联 |
| Pipeline（外部触发） | github/awesome-copilot react19-upgrade | 严格门控、失败快速 | 配置复杂，对用户要求高 |

### 2.4 改进空间

1. **当前问题**：`package-assistant.md` 硬编码了 `/Users/veryliu/Documents/GitHub/officecli/`  
   **改进做法**：替换为环境变量 `$OFFICECLI_REPO` 或相对路径 `./`，并在命令顶部说明如何配置  
   **预期收益**：命令对任何贡献者可用，而不只是原作者的机器

2. **当前问题**：`examples/hello-world-extension/agents/` 里两个 demo agent 完全无 frontmatter  
   **改进做法**：为两个文件添加最小 frontmatter（`name`、`description`、`model`）  
   **预期收益**：examples 目录才能真正作为「新手参考实现」使用——连 frontmatter 都没有的示例会教坏新手

3. **当前问题**：`.claude/skills/pr-automation/SKILL.md` 的标签状态机文档仅在 skill 内部描述，没有 `README.md` 说明整个套件的工作方式  
   **改进做法**：新增 `.claude/skills/README.md`，用流程图说明标签驱动的工作流  
   **预期收益**：新团队成员可以快速理解整套 PR 自动化如何运作

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

当时评分 **75/100**，**Security: BLOCKED**（发现 CRITICAL 安全风险）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `examples/hello-world-extension/agents/hello-researcher-context.md` | 30 | 无任何 frontmatter（-50 分） |
| `examples/hello-world-extension/agents/hello-coder-context.md` | 30 | 无任何 frontmatter（-50 分） |
| `.claude/commands/package-assistant.md` | 45 | 无 frontmatter；硬编码 `/Users/veryliu/` |
| `src/process/resources/skills/officecli-financial-model/SKILL.md` | 50 | YAML 注释破坏 parser（现已重构移除） |
| `.claude/skills/testing/SKILL.md` | 90 | 高质量（当前仍存在） |

**安全状态**：BLOCKED——`office-cli/SKILL.md` 包含 `curl -fsSL … | bash` 直接下载执行脚本，NLPM 将此标为 CRITICAL，阻止贡献 PR。

### 3.2 当时值得借鉴的模式

1. **PR 自动化套件的跨件协议设计**：5 个 PR skills 通过 HTML 注释标记 + 工作目录约定实现了无缝协作。这个「隐形协议」设计极其优雅——每个 skill 可以独立运行，合在一起又形成完整工作流
2. **`pr-automation` 的标签状态机**：用 GitHub PR 标签（而不是复杂的 webhook 或外部触发器）驱动 AI 工作流——这是成本最低的「AI 工作流编排」之一，只需要会打标签就能驱动整个流程
3. **`testing/SKILL.md` 的「行为优先测试指导」**：测试 skill 明确规定「先写行为描述，再写断言」，有具体的 edge-case 检查清单。这是测试思维具体化的好案例

### 3.3 当时的缺陷

1. **问题**：`office-cli/SKILL.md` 包含 `curl -fsSL https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.sh | bash`  
   **根本原因**：开发者把「安装文档」原封不动写进了 AI 要执行的指令里，没有意识到 AI 会真的运行这段代码。`curl | bash` 是安全反模式：下载+执行一步完成，无法验证内容，一旦 CDN 被攻击或 URL 被劫持，执行的就是恶意代码  
   **自查**：我的仓库没有类似的「让 AI 执行 curl|bash」指令，这条底线必须坚守

2. **问题**：7 个 `officecli-*` SKILL.md 文件的 YAML frontmatter 里有 `# officecli: v1.0.24` 这种注释行，导致 AionUi 自己的 parser 崩溃，这 7 个 skills 在 Skills Center 里完全不可见  
   **根本原因**：YAML [frontmatter](#frontmatter) 不支持注释（`#` 在 YAML 中是注释符，但在 `---` 块里的行为取决于 parser 实现）。开发者在 YAML 块内用 `#` 加版本注释，恰好触发了 AionUi parser 的 bug——最讽刺的是，`package-assistant.md` 的文档里**已经记录了这个 bug**，说明团队知道问题存在，但没有修复  
   **自查**：我的 SKILL.md 里从来没有在 frontmatter 块内加注释，这个好习惯需要保持

3. **问题**：`package-assistant.md` 硬编码了开发者本机路径 `/Users/veryliu/Documents/GitHub/officecli/`  
   **根本原因**：开发者在自己机器上测试可用，就直接提交了——没有「这段代码在别人机器上能不能运行」的意识。这是团队协作中「只在我机器上测试」反模式的 NL 版本  
   **自查**：我的命令文件是否有类似的环境假设？需要检查

### 3.4 当时的优化机会

1. 将 `curl | bash` 替换为版本固定+校验和验证的下载方式，或发布到包管理器
2. 从所有 officecli-* SKILL.md 的 `---` frontmatter 块内移除 `# officecli: vX.X.X` 注释行
3. 将 `package-assistant.md` 里的绝对路径替换为环境变量 `$OFFICECLI_REPO`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| YAML 注释破坏 officecli-* SKILL.md parser | `grep "^# officecli:" src/process/resources/skills/officecli-*/SKILL.md` | **FIXED**（目录已不存在） | 整个 `officecli-*` 技能家族已从仓库移除，随着架构重构一起消失 |
| `curl \| bash` 安全风险 | `grep "curl.*\|.*bash" src/process/resources/skills/_builtin/office-cli/SKILL.md` | **FIXED**（目录已不存在） | 同上，随整个 skills 目录重构一起消失 |
| 硬编码 `/Users/veryliu/` 路径 | `grep "veryliu" .claude/commands/package-assistant.md` — 第 11、49、138 行命中 | **PERSISTS** | 2 个月后，这个命令仍然只能在 veryliu 的机器上正常运行 |

### 4.2 架构演进

这是本批案例中**架构变化最剧烈**的一个。从 audit 时的 `src/process/resources/skills/`（27 个 officecli skill + AionUi platform skills）到现在的 `packages/`（desktop/web-cli/web-host 三包结构），AionUi 几乎完全重写了 NL 制品层。

这次重构的含义：
- 七个「YAML 注释破坏 parser」的 bug 通过删除整个目录来「修复」——不是真正修复，而是迁移。那 7 个 officecli skills 的功能现在放在哪里？不知道，也许被移到了单独的仓库
- 保留下来的 `.claude/skills/` 套件（10 个 skills）没有被触动，质量仍然是 87-90/100
- `examples/` 目录仍然存在，且两个无 frontmatter 的 hello-world agent 也还在

结论：开发团队的重心从「提供 AI skills 给用户」转向「建设 AionUi 桌面应用平台本身」。NL 制品层对他们而言是工具，不是产品。

### 4.3 新增的可学习模式

当前 HEAD 中的 `.claude/skills/` 套件核心设计未变，但 `pr-automation` skill 已更新，包含了对 `ScheduleWakeup` 集成的引用——这说明团队在探索「让 Claude Code 可以定时唤醒执行 PR 自动化」的能力。这个模式值得关注。

---

## 五、校准

### 5.1 我已经在做对的

1. **SKILL.md frontmatter 内不加注释**：我的所有 SKILL.md 的 `---` 块内是干净的 YAML，没有 `#` 注释
2. **没有让 AI 执行 `curl | bash`**：我的任何 skill 都没有这种不受控的远程代码执行指令
3. **命令文件不硬编码个人路径**：我的命令文件没有 `/Users/xxx/` 这种只在特定机器有效的路径

### 5.2 挑战 / 验证

本案例验证了一个我之前已有但没有足够案例支持的判断：**「已知问题 + 没有修复」是正常状态，不是例外**。

`package-assistant.md` 的文档里明确写着「这个命令需要 `/Users/veryliu/…` 路径存在」——团队知道这个问题，但两个月过去了，仍然没有修复。**知道 ≠ 修复**。这对我自己的「自查动作」有重要启示：自查发现问题后，要立即修复，不能记录后「以后再说」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有硬编码的绝对路径
grep -rn "/Users/\|/home/[a-z]" ~/.claude/commands/ --include="*.md" | head -10
```
命中后怎么办：把绝对路径替换为环境变量（`$HOME`、`$PROJECT_ROOT`）或相对路径。

```bash
# 检查我的 SKILL.md frontmatter 内是否有注释行
find ~/.claude/skills/ -name "SKILL.md" | while read f; do
  awk '/^---$/{if(in_fm){exit} else {in_fm=1; next}} in_fm && /^#/{print FILENAME": 注释行: "$0}' "$f"
done
```
命中后怎么办：将 frontmatter 内的 `# 注释` 移到 `---` 块之后，改为 HTML 注释 `<!-- 注释 -->` 或直接删除。

```bash
# 检查我的 skill/agent 文件是否有让 AI 执行 curl|bash 的指令
grep -rn "curl.*|.*bash\|wget.*|.*sh" ~/.claude/ --include="*.md" | grep -v "# 注释\|<!--" | head -10
```
命中后怎么办：立即替换为安全的安装方式（包管理器、checksummed 下载、Docker image 等）。

### 6.2 灵感 → 实施路径

1. **想法**：仿照 AionUi 的 PR 自动化套件，为 echo-sleuth 建立一个小型「experience-mining 标签驱动工作流」
   - **为何可行**：echo-sleuth 的多个 skills（git-mining → experience-synthesis → memory-management）天然适合串联，标签状态机可以驱动这个流程
   - **第一步**：在 echo-sleuth `.claude/skills/` 下新建 `mining-pipeline/SKILL.md`，定义 3 个标签状态（`mining-requested → synthesized → archived`），调用对应 skill，约 30 分钟

2. **想法**：为 claude-for-legal 建立类似的「matter workflow」标签状态机（matter-opened → under-review → advice-drafted → closed）
   - **为何可行**：claude-for-legal 的 matter-workspace skill 已有雏形，补充标签驱动逻辑可让工作流更自动化
   - **第一步**：读 AionUi `pr-automation/SKILL.md` 的状态机定义格式，然后仿写 `matter-automation/SKILL.md`，约 45 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 iOfficeAI/AionUi 的核心目的**：AI 桌面应用框架 + Claude Code 自动化开发工作流工具套件

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 高 | 同为多 skill 套件，有编排逻辑，覆盖一个完整领域的工作流 | claude-for-legal 是领域知识工具，AionUi 是开发工具；AionUi 有标签状态机 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同为多 skill + agent 结构，有工作流编排意图 | echo-sleuth 较小（4 skills），AionUi .claude/skills 是完整的 10-skill 套件 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 skill 结构 | 领域完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令文件硬编码绝对路径 | `grep -rn "/Users/\|/home/" my-repos/MarkQWu-*/commands/ --include="*.md"` | **未命中**：我的命令文件无硬编码路径 | 无 |
| Example agent 无 frontmatter | `find my-repos/ -path "*/examples/*" -name "*.md"` → 检查 | **未检查到 examples 目录**：我的仓库没有 examples/ | 无 |
| SKILL.md frontmatter 内有注释 | `grep "^#"` within `---` blocks in my SKILL.md files | **未命中**：我的 SKILL.md frontmatter 干净 | 无 |

本案例主要缺陷在我的项目里没有复现，这是一个正向确认。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：PR 自动化套件的跨 skill 协议设计
   - **本案例做法**：`.claude/skills/pr-automation/SKILL.md` 定义了 HTML 注释标记协议（`<!-- pr-automation-bot -->`），所有 PR skills 通过识别这些标记来感知状态，不需要共享任何外部状态文件
   - **我的项目现状**：claude-for-legal 的 skills 之间通过文件（matter-workspace 的 `MATTER.md`）传递状态，没有这种轻量的注释标记协议
   - **如何借鉴**：对于 echo-sleuth 的多 skill 工作流，引入 HTML 注释标记作为状态传递协议——在 JSONL 文件的摘要评论里写入 `<!-- echo-sleuth-stage: mining-complete -->`，让后续 skill 可以识别

2. **领域**：单职责 + 高内聚的 skill 套件设计
   - **本案例做法**：`pr-review`、`pr-fix`、`pr-ship`、`pr-verify` 各做一件事，每个 skill 内部完整，但它们通过明确的接口（HTML 标记 + 工作目录约定）组合成完整工作流
   - **我的项目现状**：echo-sleuth 的 `experience-synthesis/SKILL.md` 试图同时做「提取 + 分析 + 写入」三件事
   - **如何借鉴**：把 experience-synthesis 拆分为「提取」和「综合」两个 skills，让每个 skill 的职责更单一

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：示例代码的可移植性
- **我的做法**：claude-for-legal 的所有 skill 不假设任何特定的本地路径，所有文件引用都是相对路径或通过 frontmatter 声明的参数
- **本案例做法（弱在哪）**：`package-assistant.md` 里 3 处 `/Users/veryliu/…` 绝对路径，导致命令只能在作者自己机器上使用
- **意义**：可移植性是协作工具的基本素质。我的仓库在这一点上做得更好

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 和 AionUi 都先解析 frontmatter 才知道这个 skill/agent 怎么注册。如果 frontmatter 被破坏（如 YAML 注释导致 parser 报错），整个文件会被忽略——就像一本书的封面撕掉了，图书馆目录里找不到它。

### <a name="curl-pipe-bash-risk"></a>curl-pipe-bash 风险
> 一种高危的软件安装模式：`curl URL | bash`——从网络下载一个脚本然后立即执行。危险在于：你无法在执行前验证脚本内容；如果 CDN 被攻击、域名被劫持或 URL 重定向，执行的可能是恶意代码。在 AI skill 文件里写这个命令更危险，因为 AI 会无条件执行它。安全替代方案：固定版本的包管理器安装（`brew install xxx@1.2.3`），或先下载、验证 SHA256、再执行。

### <a name="label-state-machine"></a>标签状态机
> 用 GitHub PR/Issue 的标签（Label）来驱动工作流状态转换的设计模式。例如：给 PR 打上 `review-requested` 标签 → 触发 AI 代码审查 → AI 完成后打上 `reviewed` → 触发修复建议 → 修复完成后打上 `ship-ready` → 触发合并。这个模式的优雅之处在于：状态完全可见（标签一眼可看），触发无需复杂 webhook，人工干预也很自然（手动加标签）。

### <a name="html-comment-protocol"></a>HTML 注释协议
> 在 PR 评论或 Markdown 文件里使用 `<!-- key: value -->` 格式的 HTML 注释来传递机器可读的状态，而对人类读者完全不可见。AionUi 的 PR skills 用这个技巧让多个 AI skills 在同一个 PR 的评论串里「互相认识对方写的内容」，而不干扰人类读者的阅读体验。
