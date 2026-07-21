# pe-menezes/fin-claude-plugin — 学习案例

**仓库**：https://github.com/pe-menezes/fin-claude-plugin
**Stars**：0（exemplar_published=true 覆盖星数门槛）| **来源**：upstream audit
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-21（基于当前 HEAD v0.5.0）
**主题标签**：`single-purpose`, `cross-reference`, `vague-quantifier`, `experience-accumulation`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
[fin-claude-plugin](#fin-claude-plugin) 是一个完整的葡萄牙语个人财务管理 Claude Code 插件，通过一个 AI 代理（financeiro）和六个专用 skill 组合，调用 FIN App MCP 后端处理所有账目操作。作者 Pedro Menezes 设计目标是「与记账应用/存储方式无关」——无论用户使用 Obsidian、本地文件夹或自定义路径，插件都能适配。

关键事实：
- 2026 年 4 月 audit 时为 v0.4.x，当前已演进到 **v0.5.0**（新增 marketplace.json）
- 整个插件只有 **8 个 NL 工件**（1 agent + 6 skills + 1 plugin manifest），但功能完整
- 全程葡萄牙语（巴西口音），是非英语插件的典型代表
- 通过 `${user_config.fin_api_key}` 用 Claude Code 内置 userConfig 系统传递敏感凭据，而非写死

### 1.2 架构剖析
```
fin-claude-plugin/
├── agents/
│   └── financeiro.md       # 唯一 agent，调度所有 6 个 skill
├── skills/
│   ├── onboarding/         # 首次配置向导
│   ├── lancar/             # 单条交易录入
│   ├── extrato/            # 批量流水处理
│   ├── fatura/             # 信用卡账单处理
│   ├── conciliar/          # 账户对账
│   └── instalar-fin-mcp/   # MCP 安装向导
├── .claude-plugin/
│   ├── plugin.json         # Claude Code 插件注册
│   └── marketplace.json    # 市场元数据（v0.5.0 新增）
├── .mcp.json               # FIN App MCP 服务器配置
└── README.md
```

- **文件类型分布**：1 agent + 6 skills + 1 plugin manifest + 1 MCP config
- **编排关系**：agent 是唯一入口，根据用户意图分发至 6 个对应的 `/financeiro:*` slash command。Skills 之间无互调，全部单向由 agent 触发
- **跨件契约**：6 个 skills 都统一引用相同的 4 个[记忆文件](#记忆文件)（`Preferências.md`、`Contas e Cartões.md`、`Estabelecimentos.md`、`Status Conciliação.md`），命名和路径完全一致——这是本仓库最值得借鉴的设计

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「存储无关 + 记忆文件显式持久化」——Claude 的跨会话记忆不依赖任何特定工具，只通过读写用户本地 `.md` 文件实现
- **解决什么问题**：个人财务记录繁琐、各软件互不兼容；作者希望用 Claude 作为统一的智能记账中间层，后端可插拔（当前是 FIN App）
- **做了什么 trade-off**：6 个 skills 保持高度独立（每个只做一件事），代价是重复性较高——每个 skill 都要读那 4 个记忆文件，没有共享 preload 机制
- **反映什么认知模型**：作者相信「会话内状态靠 agent，跨会话状态靠文件」，这和 memory-bank 类工具的思路相同，但更简洁（4 个文件 vs 通用记忆系统）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「单 Agent 六 Skill 财务编排」

这个模式的核心特征是：一个 agent 负责理解用户意图并分发，所有实质操作在专用 skill 里执行，agent 本身只做路由，不做计算。

模式特征清单：
- **特征 1**：Agent 是纯调度器，不包含业务规则，只声明「何时用哪个 skill」
- **特征 2**：Skill 是独立执行单元，每个 skill 自包含（读哪些文件、调哪些 MCP tool）
- **特征 3**：共享记忆文件集（4 个文件），由所有 skill 协议性引用，而非传递参数
- **特征 4**：MCP backend 隔离——FIN API 只有 MCP server 看到，NL 层只调用语义化 tool 名（`fin_criar_despesa`）
- **特征 5**：用户 config 注入（`${user_config.fin_api_key}`）而非硬编码凭据

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 用户有明确的「任务类型」可以分类的工具 | ✅ 高度适用 | 财务、医疗、日历等有清晰子任务类别 |
| 工具有持久化状态需要跨会话共享 | ✅ 适用 | 记忆文件模式天然支持 |
| 工具需要 MCP 后端 + NL 前端分层 | ✅ 适用 | MCP 层隔离 API 细节 |
| 单次对话内的即兴任务 | ❌ 不适用 | 6 个 skill 的 setup cost 对一次性任务太重 |
| 需要 skill 之间互相调用的复杂流水线 | ❌ 不适用 | 这个架构没有 skill-to-skill 通道 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：1 agent + N skills（单入口分发）| fin-claude-plugin | 结构清晰，每个 skill 独立，用户意图识别集中 | Agent 没有 model 声明，examples 缺失 |
| 多 agent 编排（每个 agent 是领域专家）| shinpr/claude-code-workflows | 并发处理能力强，角色边界清晰 | 配置更复杂，entry point 多 |
| 纯 skills 平铺（无 agent）| gstack / tech-leads-club | 用户直接调用 skill，轻量 | 需要用户自己判断调用哪个 skill |

### 2.4 改进空间
1. **当前问题**：`agents/financeiro.md` 零示例，新用户无法通过阅读 agent 理解「什么时候会触发哪个 skill」。**改进做法**：在 frontmatter 或 body 加 2-3 个 input/output 示例（如「用户说'lancei R$50 de almoço'→触发 `/financeiro:lancar`」）。**预期收益**：新贡献者 onboarding 时间减半，agent 可读性提升。

2. **当前问题**：`skills/onboarding/SKILL.md` 的 `allowed-tools` 声明了 `Bash` 但没有使用——Bash 实际用途在 `instalar-fin-mcp` skill 里。**改进做法**：移除 onboarding 的 Bash 声明，避免运行时加载无用 tool 权限。**预期收益**：每次 onboarding 调用减少一个工具权限请求，行为更可预测。

3. **当前问题**：`.mcp.json` 中 `fin-app-mcp` 没有版本固定，每次 `npx -y` 自动获取最新版可能带来兼容性破坏。**改进做法**：改为 `fin-app-mcp@X.Y.Z`，配合 CHANGELOG 手动更新。**预期收益**：供应链稳定性提升，避免上游发布破坏性更新后插件自动中断。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-29 当时整体得分 **94/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/financeiro.md | 80 | 零示例（-15），无 model 声明（-5） |
| skills/onboarding/SKILL.md | 92 | Bash 声明但未用（-3），2 个模糊量词（-4） |
| skills/fatura/SKILL.md | 95 | "tipicamente"无阈值（-2） |
| skills/conciliar/SKILL.md | 96 | "alguns reais"语义模糊（-2） |
| skills/extrato/SKILL.md | 96 | "confiança baixa"无量化标准（-2） |
| skills/instalar-fin-mcp/SKILL.md | 97 | 几乎无问题 |
| skills/lancar/SKILL.md | 97 | "caso raro"无决策标准（-2） |
| .claude-plugin/plugin.json | 100 | 满分 |

### 3.2 当时值得借鉴的模式

1. **跨件记忆文件契约**：所有 6 个 skills 用相同路径引用相同 4 个 `.md` 文件，形成隐式契约。为什么好：不需要 skill-to-skill 传参，记忆自动共享。示例：所有 skills 头部都有「读 Preferências.md」步骤。如何借鉴：在自己的多 skill 项目里定义一套「共享记忆清单」文件，并在所有相关 skills 的 Opening Steps 里统一引用。

2. **MCP tool 语义命名**：`fin_criar_despesa`、`fin_buscar_transacoes`、`fin_criar_estorno` 等命名动词+对象，不暴露 REST 路径。为什么好：NL 层读到工具名就知道语义，不需要理解 API 细节。如何借鉴：设计 MCP tool 时用动词+领域名词，而非 HTTP 动词（`create_expense` 好过 `post_v1_transactions`）。

3. **敏感凭据 userConfig 注入**：用 `${user_config.fin_api_key}` 替代硬编码。为什么好：key 存储在系统 keychain，不出现在 git 历史或对话上下文。如何借鉴：任何需要 API key 的 MCP 插件都应用这个模式。

4. **Agent 作为纯调度器**：financeiro.md agent 只描述「何时触发哪个 slash command」，业务规则都在 skills 里。为什么好：agent 可测试、可替换，不混杂实现细节。如何借鉴：设计 agent 时写一个明确的「dispatch table」——输入→触发 skill 的映射表。

### 3.3 当时的缺陷

1. **Agent 零示例**（-15 分）：`agents/financeiro.md` 没有任何 input/output 示例。根本原因：作者可能以为 description 足够了，但 description 描述「能做什么」，示例才能说明「怎么触发」，两者不可替代。自查：我的 agent 们有多少个有具体的 input/output 示例？

2. **无 model 声明**（-5 分）：Agent frontmatter 没有 `model:` 字段。根本原因：不声明 model 会随 Claude Code 版本静默升级，对财务类敏感场景来说这意味着行为可能变化。自查：我的 gstack 中有哪些 agent 没有 model 声明？

3. **模糊量词**（多项 -2 分）：「tipicamente」「alguns reais」「caso raro」「muito próximas」。根本原因：这些词源于人类沟通的模糊性，在 NL programming 里每个不确定语义都是 AI 自由发挥的机会，导致不可预测行为。自查：我的 skills 里有多少类似的「通常」「大约」「少量」？

### 3.4 当时的优化机会

1. **首要**：在 `agents/financeiro.md` 加 2-3 个 input/output 示例 + `model:` 声明，可从 80 分提升到 ~95
2. 将 `onboarding/SKILL.md` 里多余的 `Bash` 从 allowed-tools 移除
3. 在 `.mcp.json` 中固定 `fin-app-mcp` 版本号

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Agent 零示例（-15） | `grep -n "example\|input\|output" agents/financeiro.md` | **持续**：grep 返回空，仍无任何示例 | 3 个月后仍未修，作者可能不清楚这对可读性的影响 |
| 无 model 声明（-5） | `grep "^model:" agents/financeiro.md` | **持续**：frontmatter 仍无 model 字段 | 运行时版本随 Claude Code 漂移 |
| `.mcp.json` 未固定版本 | `cat .mcp.json \| grep fin-app-mcp` | **持续**：仍为 `"fin-app-mcp"`，无版本号 | 供应链风险未修，每次重启可能拉新版 |

### 4.2 架构演进

**从 v0.4 到 v0.5.0**：
- 新增 `.claude-plugin/marketplace.json` ——说明作者开始为上架 Claude Code 市场做准备
- 核心 NL 工件（agents/ + skills/）**无变化**——功能稳定，问题也原封不动保留
- 这说明：marketplace 集成优先于质量修复，是合理的阶段性决策

### 4.3 新增的可学习模式

当前 HEAD 新增了 `marketplace.json`，其结构值得参考：将面向用户的元数据（搜索关键词、许可证、作者信息）从 `plugin.json`（技术注册）中分离，保持两者职责清晰。

---

## 五、校准

### 5.1 我已经在做对的

1. **MCP 后端隔离**：我的 graphify 和 bureau 同样把 backend 操作封装在 MCP 或 scripts/ 层，NL 层只调用语义化工具名
2. **敏感信息不写死**：bureau 使用 Claude Code 内置 CLAUDE_CODE_OAUTH_TOKEN，graphify 依赖环境变量
3. **Agent 模型声明**：bureau 的 agent.md 都有 `model: sonnet` 声明（bureau: 全部有，gstack: 1/N 缺失）
4. **单职责 skill**：我的 bureau skills（recall/capture/compile/scribe 等）各自职责清晰，无交叉依赖

### 5.2 挑战 / 验证

本案例验证了一个我一直犹豫的做法：**4 个共享记忆文件作为跨 skill 的隐式协议**。我之前倾向于把共享状态做成数据库或 JSON，但这个案例说明简单的 `.md` 文件 + 统一路径约定完全够用，且对 AI 来说读写 `.md` 比解析 JSON 更自然。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 gstack 中缺少 model 声明的 agent
find /tmp/my-repos/MarkQWu-gstack -name "*.md" | xargs grep -l "^description:" | \
  xargs grep -L "^model:" 2>/dev/null
# 命中后：为每个 agent 按照实际需要声明 claude-haiku 或 claude-sonnet 级别

# 检查我的所有 skills 里有没有「通常/适当/大约/少量」类模糊量词
grep -rn "通常\|适当\|大约\|少量\|一般来说\|合适的" \
  /tmp/my-repos/MarkQWu-gstack/ /tmp/my-repos/MarkQWu-bureau/ --include="*.md" 2>/dev/null
# 命中后：替换为具体阈值或可验证标准

# 检查 bureau skills 有没有 ## Output 格式声明
find /tmp/my-repos/MarkQWu-bureau -name "SKILL.md" | \
  xargs grep -L "## Output\|## 输出" 2>/dev/null
# 命中后：每个缺少的 skill 加 3-5 行输出格式描述
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 skills 定义一套「共享上下文文件清单」（类似 fin 的 4 个记忆文件）
   - **为何可行**：bureau 的多个 skills（recall/compile/scribe）都需要读 workspace 状态，当前各自独立查找
   - **第一步**：在 `bureau/BUREAU.md` 里加一节「共享状态文件」，列出所有 skills 都应在开始时读的文件路径，然后给每个 SKILL.md 的 Opening Steps 里加统一的「先读共享上下文」步骤（30 分钟）

2. **想法**：为 gstack 所有缺少 `## Output Format` 的 skills 批量补充输出格式声明
   - **为何可行**：50 个 gstack skills 有 10+ 缺少输出格式，影响预测性
   - **第一步**：`find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" | xargs grep -L "## Output"` 找到清单，逐一加 3 行最小化输出格式描述（2-3 小时）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **pe-menezes/fin-claude-plugin 核心目的**：葡萄牙语个人财务记账 + AI 智能分类，通过记忆文件跨会话持久化，MCP 后端处理真实 API 调用

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都有跨会话记忆机制；都是「单仓多 skill + 1 入口」架构；都用结构化 .md 文件持久化状态 | bureau 更通用（知识管理 vs 财务），bureau 有 crew/agent 层而非纯 dispatch | **高** |
| MarkQWu/gstack | 中 | 都是 multi-skill 工具集；都有 agent 作为入口 | gstack 更像工具箱（每个 skill 独立），fin 更像聚焦应用（所有 skills 协同） | 中 |
| MarkQWu/graphify | 低 | 都用 MCP 作为后端 | graphify 是代码知识图谱，与财务无关 | 低 |
| MarkQWu/drama-workshop-skills | 无 | - | 戏剧工坊 skills，领域完全不同 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 零 input/output 示例 | `find /tmp/my-repos/MarkQWu-bureau -name "agent.md" \| xargs grep -L "example\|input.*output"` | bureau 的 `crew/auditor/agent.md` 有示例；但 gstack 无专用 agent 文件 | 低（gstack 无 agent 层） |
| Skills 缺失 `## Output` 格式声明 | `find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" \| xargs grep -L "## Output"` | **命中 10+ 个** `setup-deploy`, `document-release`, `diagram` 等 | **高** |
| 模糊量词（typicamente / alguns） | `grep -rn "通常\|适当\|大约" /tmp/my-repos/MarkQWu-gstack/ --include="*.md"` | 命中数量未统计（gstack 有 354 个宽泛量词命中，部分是 README） | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的 `diagram/SKILL.md` → 加 `## Output Format` 声明图表类型和内容；5 分钟
- `MarkQWu/gstack` 的 `setup-deploy/SKILL.md` → 加部署成功/失败的输出格式样例；10 分钟
- `MarkQWu/bureau` 的所有 `skills/*.md` → 加「每次调用前先读共享上下文文件」步骤；30 分钟

### 7.3 别人的更优方案

1. **领域**：跨 Skill 共享记忆文件的显式协议
   - **本案例做法**：在 6 个 skills 的「每次运行前必读」步骤里统一声明相同 4 个文件路径（`Preferências.md`, `Contas e Cartões.md`, `Estabelecimentos.md`, `Status Conciliação.md`），skill 之间无需传参
   - **我的项目现状**：`MarkQWu/bureau` 的 skills 各自独立查找状态，没有标准化的「session 开始先读 N 个文件」契约
   - **如何借鉴**：在 `bureau/BUREAU.md` 定义「共享上下文协议」，每个 SKILL.md 开头统一读 `bureau/state/*.md` 里的核心状态文件

2. **领域**：MCP userConfig 注入替代环境变量硬编码
   - **本案例做法**：`plugin.json` 声明 `userConfig.fin_api_key`，`.mcp.json` 用 `${user_config.fin_api_key}` 注入
   - **我的项目现状**：`MarkQWu/graphify` 使用环境变量传递，未利用 Claude Code 的 keychain 机制
   - **如何借鉴**：graphify 的 API key 配置改为 `plugin.json` 的 `userConfig` 声明，利用系统 keychain 更安全

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Agent 模型显式声明
- **我的做法**：`MarkQWu/bureau/crew/auditor/agent.md` 声明了 `model: sonnet`，明确绑定模型级别
- **本案例做法**：`agents/financeiro.md` 完全无 `model:` 声明，依赖 Claude Code 默认值
- **意义**：bureau 的 agent 在 Claude Code 版本升级时行为更稳定；若上游收到我的 PR，这是可提的改进点

---

## 八、术语表

### <a name="fin-claude-plugin"></a>fin-claude-plugin
> Pedro Menezes 开发的个人财务管理 Claude Code 插件。「fin」是「financeiro（财务）」的缩写，「plugin」是 Claude Code 的插件包格式。它通过一个 AI 代理（financeiro agent）和六个专用技能（skills），结合 FIN App 的 MCP 后端，在 Claude 对话界面里完成账单录入、对账、账单处理等操作。

### <a name="记忆文件"></a>记忆文件
> 插件在用户本地存储的 `.md` 格式文本文件，用于保存跨 AI 会话的持久状态。比如「我最近加的这个 Nubank 卡片，下次 Claude 还记得吗？」——答案是靠把卡片信息写进 `Contas e Cartões.md`。这类文件是低技术门槛的「数据库替代品」，AI 读写 `.md` 就像读写笔记，比解析 SQL 自然很多。

### <a name="MCP"></a>MCP
> Model Context Protocol。Anthropic 发布的标准化协议，让 AI 模型（如 Claude）能够安全地调用外部服务或工具。`.mcp.json` 配置文件告诉 Claude「有哪些 MCP 服务器可用，如何启动它们」。fin-claude-plugin 用 MCP 服务器封装 FIN App 的 REST API，这样 NL 层不需要直接处理 HTTP 请求。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包围的 YAML 配置块。Claude Code 读 agent 或 SKILL.md 时，先解析 frontmatter 获取元数据（如 `name`、`model`、`tools`、`allowed-tools`），再读正文内容。`model:` 没写在 frontmatter 里 = Claude Code 不知道该用哪个 tier 的模型，会用默认值。

### <a name="userConfig"></a>userConfig
> Claude Code 插件系统的用户配置注入机制。在 `plugin.json` 里声明 `userConfig` 字段后，Claude Code 会在安装插件时提示用户填写敏感配置（如 API key），并加密存储在系统 keychain 里。插件里用 `${user_config.xxx}` 引用，不会出现在 git 历史或对话日志中。比环境变量更安全，因为不需要用户手动设置。
