# czlonkowski/n8n-skills — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-skills
**Stars**：N/A | **来源**：本地 audit（exemplar_published=True）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-11（基于当前 HEAD）
**主题标签**：`examples-driven`, `manifest-discipline`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

[czlonkowski/n8n-skills](https://github.com/czlonkowski/n8n-skills) 是一套专为 [n8n](#n8n) 工作流自动化平台设计的 Claude Code skill 套件。它的核心使命是教会 Claude 如何正确编写 n8n 工作流——包括 JavaScript/Python 节点代码、[MCP 工具](#mcp-tools)调用、节点配置、数据验证和表达式语法。

关键事实：
- **获取方式**：通过 Claude Code 插件市场安装（`claude plugin install n8n-skills@czlonkowski`）
- **Exemplar 状态**：NLPM 评分 ≥90 + 安全状态 CLEAR，被 NLPM 收录为正向教学典范（exemplar_published=True）
- **版本追踪**：audit 时 plugin.json 与 build.sh 版本均为 v1.4.0，实现了「版本对齐」——manifest 声明的版本和打包脚本使用的版本完全一致
- **生态定位**：「领域工具 skill 化」的样板案例——把 n8n 平台的专有知识（API、表达式语法、节点行为）打包为 Claude 可调用的结构化指令，填补了 Claude 训练数据对 n8n 领域覆盖不足的空缺
- **重要说明**：与同作者的 `czlonkowski/n8n-mcp`（见 `learning/2026-06/2026-06-02-czlonkowski-n8n-mcp.md`）是两个不同仓库——n8n-skills 是 Claude Code skill 套件，n8n-mcp 是 MCP 服务器

### 1.2 架构剖析

```
n8n-skills/
├── .claude-plugin/
│   ├── plugin.json            ← manifest，v1.4.0，版本与 build.sh 对齐
│   └── marketplace.json
├── skills/
│   ├── n8n-code-javascript/
│   │   ├── SKILL.md           ← 「协调者」：调用场景 + 跨 skill 引用
│   │   ├── BUILTIN_FUNCTIONS.md   ← 伴随参考文件：内置函数速查
│   │   └── STANDARD_LIBRARY.md   ← 伴随参考文件：标准库参考
│   ├── n8n-code-python/
│   │   ├── SKILL.md
│   │   └── STANDARD_LIBRARY.md
│   ├── n8n-mcp-tools-expert/
│   │   ├── SKILL.md
│   │   └── SEARCH_GUIDE.md    ← 伴随参考文件：MCP 工具搜索策略
│   ├── n8n-node-configuration/
│   │   ├── SKILL.md
│   │   └── DATA_ACCESS.md     ← 伴随参考文件：节点数据访问模式
│   ├── n8n-validation-expert/
│   │   ├── SKILL.md
│   │   └── VALIDATION_GUIDE.md
│   ├── n8n-expression-syntax/
│   │   └── SKILL.md
│   ├── n8n-workflow-patterns/
│   │   ├── SKILL.md
│   │   └── WORKFLOW_GUIDE.md
│   │
│   ├── n8n-agents/            ← 新增（HEAD）
│   ├── n8n-binary-and-data/   ← 新增（HEAD）
│   ├── n8n-code-tool/         ← 新增（HEAD）
│   ├── n8n-error-handling/
│   │   ├── SKILL.md           ← 新增（HEAD）
│   │   └── ERROR_PATTERNS.md  ← 伴随参考文件：错误模式库
│   ├── n8n-multi-instance/    ← 新增（HEAD）
│   ├── n8n-self-hosting/
│   │   ├── SKILL.md           ← 新增（HEAD）
│   │   └── DEPENDENCIES.md    ← 伴随参考文件：依赖关系
│   ├── n8n-subworkflows/      ← 新增（HEAD）
│   └── using-n8n-mcp-skills/  ← 新增（HEAD）：集成指南
│
├── hooks/                     ← 新增（HEAD）
│   ├── pre-tool-use.sh        ← PreToolUse：n8n MCP 工具调用前拦截
│   └── post-tool-use.sh       ← PostToolUse：n8n MCP 工具调用后审计
├── evaluations/               ← 新增（HEAD）：skill 效果评估
├── build.sh                   ← 自包含打包脚本，无网络调用，安全
└── CLAUDE.md
```

**文件类型分布**（当前 HEAD）：15 个 SKILL.md + 多个伴随参考文件 + 2 个 hook 脚本 + 0 个 agent command

**编排关系**：15 个 skill 平铺，每个 SKILL.md 包含「跨 skill 集成」章节，显式列出并描述所有兄弟 skill——这是水平协议（每个 skill 知道整个生态系统），而非垂直编排（没有中央 router）。

### 1.3 设计思路 / 方法论

**核心设计哲学：「协调者 + 知识库分离」**

这是本仓库最值得深入理解的设计决策。每个 skill 目录分为两类文件：

- **SKILL.md（协调者）**：定义调用场景、说明核心原则、列出跨 skill 引用，以及指向伴随参考文件。SKILL.md 是「入口」，而非「全量知识」
- **伴随参考文件（知识库）**：`DATA_ACCESS.md`、`ERROR_PATTERNS.md`、`BUILTIN_FUNCTIONS.md` 等——深度领域知识外化为独立文件，按主题分层

**为什么比「把所有知识塞进一个 SKILL.md」更好**：

1. **上下文窗口预算**：Claude 每次对话的上下文窗口有限。把所有知识堆在一个 SKILL.md 里，加载时会占用大量窗口，导致 Claude 在「读文档」上消耗太多注意力。伴随文件让 SKILL.md 保持短小精悍，Claude 只在需要时才引用对应文件
2. **关注点分离**：SKILL.md 回答「何时用、怎么用」；参考文件回答「具体是什么」。一个处理调用逻辑，另一个处理领域知识——职责清晰，各自独立演进
3. **选择性加载**：n8n 的 JavaScript 节点不需要 Python 标准库；n8n 的 MCP 工具调用不需要内置函数表。伴随文件按需引用，避免「一次性加载全部知识」的噪声
4. **可独立维护**：n8n 升级后，只需更新对应的 `BUILTIN_FUNCTIONS.md`，SKILL.md 的调用逻辑不受影响。单文件设计里，每次知识更新都要重写整个 SKILL.md

**其他设计决策**：
- **零安全面积（audit 时）**：无 hooks、无网络调用脚本、无 eval——这是故意的，工具内容通过参考文件传递而非脚本执行
- **版本对齐纪律**：plugin.json 和 build.sh 的版本号同步，防止「manifest 显示旧版本」的用户困惑
- **build.sh 自包含**：打包脚本不依赖任何外部包管理器或网络资源，可在离线环境运行

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「协调者 + 知识库分离」模式（Coordinator + Knowledge-Base Separation）**

模式特征清单：
- 特征 1：每个 skill 目录有且仅有一个 SKILL.md（入口 / 协调者），配以若干主题化参考文件（知识库分片）
- 特征 2：SKILL.md 的长度被刻意控制——不超过 200 行；深度知识下沉到参考文件
- 特征 3：所有 SKILL.md 都有「跨 skill 集成」章节，形成显式的水平感知图谱
- 特征 4：参考文件按知识主题命名（`DATA_ACCESS`、`ERROR_PATTERNS`、`BUILTIN_FUNCTIONS`），不按使用顺序命名
- 特征 5：零内部断链——所有 SKILL.md 中引用的参考文件均存在于磁盘

### 2.2 适用场景

| 场景 | 适用度 | 原因 |
|---|---|---|
| 深度领域知识 skill 化（n8n、数据库、框架文档等） | 高 | 领域知识量大，必须分层管理；参考文件天然契合「随查随用」的使用模式 |
| 平台 API / 语法速查类 skill | 高 | API 细节频繁更新，独立参考文件可单独维护 |
| 跨多个工具/命令的 skill 生态系统 | 高 | 水平感知图谱（跨 skill 集成章节）让 Claude 知道「这个问题该转哪个 skill」 |
| 单一职责的简单 skill（≤50 行知识点） | 低 | 知识量少时，分文件增加目录复杂度而无显著收益 |
| 需要实时状态感知的 skill（运行时调用 API） | 低 | 参考文件是静态知识，无法反映运行时状态 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 协调者 + 知识库分离 | czlonkowski/n8n-skills | 上下文窗口高效；知识可独立维护；SKILL.md 简洁 | 目录结构复杂；新手难以理解哪个文件看什么 |
| 全量自包含 SKILL.md | slavingia/skills | 单文件即完整，无需跨文件理解结构 | SKILL.md 过长时，Claude 加载效率下降 |
| 命令 + skill 分层 | openai/codex-plugin-cc | 一条命令触发完整流程，用户体验好 | 需要维护两层文件，复杂度高 |
| 带 hooks 的 skill 套件 | czlonkowski/n8n-skills（HEAD） | hooks 实现工具调用前后的守卫逻辑 | hooks 引入安全审查面积，需额外审计 |

### 2.4 改进空间

1. **当前问题**：15 个 skill 没有「索引入口」，用户在开始使用 n8n 时不知道从哪个 skill 入手。**改进做法**：添加 `using-n8n-mcp-skills` 作为入口 skill（HEAD 已存在类似尝试），或者添加一个 `/n8n:start` command 询问用户的任务类型。**预期收益**：首次用户从「我应该调用哪个 skill？」变为「告诉我你想做什么，我来导航」
2. **当前问题**：`comprehensive`、`effectively` 等[模糊量词](#vague-quantifier)仍存在于多个 SKILL.md（见 §4.1）。**改进做法**：将 `comprehensive` 替换为具体的数量或范围（「覆盖 n8n v1.70+ 的所有 Code 节点内置对象」），将 `effectively` 替换为具体行为动词（「避免在 Code 节点中使用 `require()` 导入外部模块」）。**预期收益**：Claude 的输出从「效果上有效」变为「按照具体规则执行」
3. **当前问题**：新增的 hooks/ 目录在 CLAUDE.md 中缺乏使用指引（根据数据描述推断）。**改进做法**：在 CLAUDE.md 的「hooks 使用」章节补充：「PreToolUse hook 会在每次 n8n MCP 工具调用前检查参数合规性；PostToolUse hook 会在调用后记录结果」。**预期收益**：用户安装后能快速理解 hook 的守卫边界，而不是通过试错发现

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 **93/100**，共 9 个 artifact。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 90 | "effectively"、"expert guidance" 模糊量词各 -2 |
| n8n-code-javascript/SKILL.md | 92 | "comprehensive" ×2、"many" ×1，共 -8 |
| n8n-code-python/SKILL.md | 92 | "comprehensive"/"complete" ×3，共 -8 |
| n8n-mcp-tools-expert/SKILL.md | 92 | "effectively" 在 description 字段，-8 |
| n8n-node-configuration/SKILL.md | 94 | "comprehensive" 出现在 guides 引用中，-6 |
| n8n-validation-expert/SKILL.md | 94 | "comprehensive" 出现在 guides 引用中，-6 |
| n8n-expression-syntax/SKILL.md | 95 | 无重大问题 |
| n8n-workflow-patterns/SKILL.md | 95 | 无重大问题 |
| .claude-plugin/plugin.json | 95 | 无重大问题 |

**无 bug，无安全发现**——所有扣分均为质量类（模糊量词），与功能正确性无关。

**跨组件差距（信息性，非 bug）**：
- `n8n-mcp-tools-expert` 引用了 `n8n_health_check()` 函数，但 CLAUDE.md 的「Key MCP Tools」章节未列出该函数
- CLAUDE.md 提到 `npm install @anthropic/claude-code-plugin-n8n-skills` 安装方式，但仓库中不存在 `package.json`

因为没有 bug 或安全问题，**未提交任何 PR**——这也是 exemplar 的特点之一：质量高到贡献者找不到值得修复的内容。

### 3.2 当时值得借鉴的模式

1. **全量零断链的内部引用**：所有 9 个 SKILL.md 中引用的伴随参考文件在磁盘上均实际存在，内部链接零断链。这说明作者在新建参考文件时同步更新了 SKILL.md 的引用，而不是「先写引用，后补文件」。**借鉴**：每次在 SKILL.md 中新增对参考文件的引用时，立即同步创建该文件（哪怕是空文件占位）；提交前运行 `grep -r "\.md" skills/*/SKILL.md | xargs -I{} test -f {}` 验证零断链

2. **跨 skill 集成章节的标准化**：每个 SKILL.md 都有「与其他 skill 的集成」章节，且每条引用都包含 skill 名称和一句话职责描述——不只是列出 skill 名称，而是解释「为什么要交给它」。**借鉴**：跨 skill 引用格式固定为「`skill-name`：[一句话描述该 skill 负责什么]」，不能只写名称

3. **版本对齐纪律（plugin.json ↔ build.sh）**：audit 时两处版本号均为 v1.4.0，严格同步。**借鉴**：将版本号声明在单一真相源（如 `VERSION` 文件），plugin.json 和 build.sh 均从该文件读取，消除「修改一处忘记另一处」的风险

4. **build.sh 的自包含设计**：打包脚本不调用 `npm install`、`pip install` 或任何外部网络资源。在 CI 环境、离线开发机和受限企业网络下均可运行。**借鉴**：凡是需要打包 / 发布插件的 build 脚本，优先使用语言内置工具（`zip`、`cp`、`tar`），避免依赖包管理器

5. **伴随参考文件按主题命名**：`DATA_ACCESS.md`、`ERROR_PATTERNS.md`、`BUILTIN_FUNCTIONS.md`——命名直接反映「这个文件回答什么问题」，而非「这个文件在流程的哪一步」。**借鉴**：参考文件命名用名词短语（描述知识类型），不用动词或序号（`step1.md`、`do-this.md`）

### 3.3 当时的缺陷

1. **高频模糊量词「comprehensive」**：在 4 个 SKILL.md 中共出现 6+ 次。**根本原因**：作者在描述参考文件的覆盖范围时，倾向于用「comprehensive guide」表达「这个文件很全面」，但 Claude 无法从「comprehensive」推断出具体覆盖了哪些 n8n 版本的哪些 API。**自查**：检查我的所有 SKILL.md 中是否有「comprehensive coverage」「comprehensive guide」类表达（见 §6.1）

2. **description 字段的「effectively」**：`n8n-mcp-tools-expert/SKILL.md` 的 frontmatter description 字段写了「effectively use n8n MCP tools」。Description 字段是 Claude Code 用来匹配 skill 触发条件的关键字段——「effectively」在这里没有区分信息，让 Claude 无法判断何时该触发这个 skill 而非其他 skill。**自查**：检查我的 skill description 字段，确保是可触发的情境短语（「Use when...」）而非品质形容词

3. **CLAUDE.md 中的安装命令与仓库结构不符**：`npm install @anthropic/claude-code-plugin-n8n-skills` 命令暗示存在 npm 包，但仓库根目录无 `package.json`。**根本原因**：可能是计划中的发布方式写进了文档但未落地，或者 npm 包由另一个仓库管理。**自查**：我的 CLAUDE.md 中的所有安装命令，是否都能在仓库当前状态下实际执行成功

### 3.4 当时的优化机会（学习材料，不用于 PR）

1. 将 `comprehensive guide` 替换为「覆盖 n8n v1.x 所有 Code 节点内置对象的参考手册」（高优先级，影响 Claude 输出精度）
2. 将 `n8n-mcp-tools-expert` 的 description 改为「Use when working with n8n's built-in MCP node or calling external MCP tool endpoints from a workflow」（中优先级，影响 skill 触发精度）
3. CLAUDE.md 中安装命令与实际打包产物对齐（低优先级，文档问题）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 验证命令 | 现状 | 含义 |
|---|---|---|---|
| `comprehensive` 模糊量词 | `grep -rn "comprehensive" skills/*/SKILL.md` | **仍存在**（n8n-code-python/SKILL.md 及多个文件命中） | 3 个月未修复——作者把它视为风格而非 bug，或未关注 NLPM audit |
| `effectively` 在 description | `grep -n "effectively" skills/n8n-mcp-tools-expert/SKILL.md` | **需验证**（数据未更新） | 如仍存在，说明 frontmatter 的 description 字段被视为文案而非触发条件 |
| 安装命令不对齐 | `test -f package.json && echo EXISTS || echo MISSING` | **需验证** | build.sh 自包含设计与 npm 安装方式存在概念冲突 |

### 4.2 架构演进

从 audit（2026-04-13）到 HEAD（2026-07-11），该仓库经历了显著扩张：

- **skill 数量**：7 个 → 15 个（+8 个新 skill）
- **新 skill 主题**：n8n-agents、n8n-binary-and-data、n8n-code-tool、n8n-error-handling、n8n-multi-instance、n8n-self-hosting、n8n-subworkflows、using-n8n-mcp-skills
- **新增 hooks/ 目录**：从「零安全面积」变为「有两个 hook 脚本」——这是一次方向性转变，引入了工具调用前后的拦截逻辑
- **新增 evaluations/ 目录**：加入了 skill 效果评估基础设施，这在 audit 时完全不存在

**核心洞察**：

> 这不是「维护型演进」（修复 bug、更新版本）而是「扩张型演进」（覆盖 n8n 更多使用场景）。hooks/ 的加入标志着从「纯知识型 skill」向「带守卫逻辑的 skill 生态」升级——这是架构意图的转变，而非自然生长。

**版本对齐状态**（推断）：需验证 plugin.json 版本是否仍与 build.sh 的 `VERSION=` 保持同步。扩张后的维护纪律比初版更难保持。

### 4.3 新增的可学习模式

1. **Hooks-as-Guardrails 模式**：`pre-tool-use.sh` + `post-tool-use.sh` 专门针对 n8n MCP 工具调用的前后拦截。这是「skill 知道如何使用工具」之外更进一步的设计——「系统知道 skill 何时在调用工具，并可以介入」。适用场景：当 skill 调用的外部工具可能产生副作用（写数据库、触发 webhook），需要前置检查或后置审计
2. **using-n8n-mcp-skills 元 skill**：15 个领域 skill + 1 个「如何使用这套 skill 生态」的集成指南 skill。这是对「入口缺失问题」（见 §2.4）的一种回应——不用 command 做路由，而是用一个元 skill 做导航

---

## 五、校准

### 5.1 我已经在做对的

1. **模糊量词密度低于 czlonkowski**：根据数据，`MarkQWu/gstack` 的 `openclaw/skills/` 仅有 1-2 个模糊量词命中，而 czlonkowski 在 audit 时有 8+ 个命中。在这个维度上，我的仓库做得比这个 93 分的 exemplar 更好
2. **有跨 skill 引用意识**：`MarkQWu/bureau` 的 skills/ 已有描述兄弟 skill 的内容，虽然系统性不如 czlonkowski 的「跨 skill 集成」标准章节，但方向正确
3. **单职责 skill 设计**：我的各 skill 职责边界清晰，没有「万能 skill」——这与 czlonkowski 的设计哲学一致

### 5.2 挑战 / 验证

这个案例验证了「伴随参考文件模式」的价值，但也让我意识到一个盲区：

**我以前认为**「SKILL.md 越详细越好，把所有知识写进去，Claude 不用跳转就能获取全量信息」。

**czlonkowski 证伪了这个假设**：93 分的 exemplar 恰恰是「SKILL.md 简洁，知识外化到参考文件」的架构。关键不是「Claude 能看到多少知识」，而是「Claude 在需要时能找到对应的知识，同时保持主 skill 文件的清晰可读」。

**需要验证的问题**：
- 我的哪个 skill 超过了 200 行？超出部分是否适合外化为参考文件？
- `MarkQWu/bureau` 和 `MarkQWu/graphify` 的 skills/ 目录中，有没有「知识密集型 skill」（大量 API 名称、规则表、代码示例）应该拆分为参考文件？

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查我的所有仓库中的模糊量词（comprehensive / effectively / efficiently / robust / powerful）
# 对每个仓库分别运行：

# gstack
grep -rn -iE '\b(comprehensive|effectively|efficiently|robust|powerful|sophisticated|meaningful|appropriate)\b' \
  ~/MarkQWu/gstack/openclaw/skills/ 2>/dev/null
# 命中后怎么办：将模糊量词替换为具体数量、版本范围、或行为动词

# bureau
grep -rn -iE '\b(comprehensive|effectively|efficiently|robust|powerful|sophisticated|meaningful|appropriate)\b' \
  ~/MarkQWu/bureau/skills/ 2>/dev/null

# graphify
grep -rn -iE '\b(comprehensive|effectively|efficiently|robust|powerful|sophisticated|meaningful|appropriate)\b' \
  ~/MarkQWu/graphify/graphify/skills/ 2>/dev/null

# echo-sleuth
grep -rn -iE '\b(comprehensive|effectively|efficiently|robust|powerful|sophisticated|meaningful|appropriate)\b' \
  ~/MarkQWu/echo-sleuth-for-claude/skills/ 2>/dev/null
```

```bash
# 2. 检查哪些 SKILL.md 过长（超过 150 行）——这些是伴随参考文件改造的候选
for f in $(find ~/MarkQWu -name "SKILL.md" 2>/dev/null); do
  lines=$(wc -l < "$f")
  if [ "$lines" -gt 150 ]; then
    echo "$lines 行  $f"
  fi
done
# 命中后怎么办：识别文件中的「知识密集型章节」（API 列表、规则表），
# 提取到同级目录的参考文件（如 DATA_ACCESS.md），SKILL.md 改为引用
```

```bash
# 3. 检查 SKILL.md 的 description 字段是否是品质形容词而非情境触发词
grep -rn -A1 "^description:" ~/MarkQWu/*/skills/*/SKILL.md 2>/dev/null | \
  grep -iE '(comprehensive|powerful|robust|efficiently|effectively)' 
# 命中后怎么办：将 description 改为「Use when <具体情境>」格式
```

```bash
# 4. 检查是否有跨 skill 引用了不存在的文件（内部断链）
# 对 bureau 仓库为例：
cd ~/MarkQWu/bureau
for skill_md in skills/*/SKILL.md; do
  dir=$(dirname "$skill_md")
  # 提取所有 .md 引用
  grep -oE '\[.*?\]\(([^)]+\.md)\)' "$skill_md" | grep -oE '\(.*\)' | tr -d '()' | while read ref; do
    full="$dir/$ref"
    if [ ! -f "$full" ]; then
      echo "断链：$skill_md -> $ref"
    fi
  done
done
# 命中后怎么办：创建缺失文件，或移除失效引用
```

```bash
# 5. 检查 plugin.json 版本与 build.sh 版本是否对齐（如有 build.sh）
for repo in ~/MarkQWu/gstack ~/MarkQWu/bureau ~/MarkQWu/graphify ~/MarkQWu/echo-sleuth-for-claude; do
  pj="$repo/.claude-plugin/plugin.json"
  bs="$repo/build.sh"
  if [ -f "$pj" ] && [ -f "$bs" ]; then
    pj_ver=$(python3 -c "import json; print(json.load(open('$pj')).get('version','MISSING'))")
    bs_ver=$(grep -oE 'VERSION="[^"]+"' "$bs" | head -1 | tr -d 'VERSION="')
    echo "$repo: plugin.json=$pj_ver, build.sh=$bs_ver"
  fi
done
# 命中后怎么办：统一版本到单一真相源（VERSION 文件），两处均引用
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `MarkQWu/bureau` 的最长 skill 提取伴随参考文件
   - **为何可行**：bureau 的 skills/scribe 和 skills/recall 可能包含大量规则描述，适合外化
   - **第一步**：运行上方命令 2，找出 >150 行的 SKILL.md；在其同级目录新建 `RULES.md` 或 `REFERENCE.md`，将规则表和细节迁移进去；SKILL.md 改为「详细规则见 [RULES.md](./RULES.md)」的引用形式
   - **时间估算**：30-45 分钟可完成一个 skill 的改造

2. **想法**：给 `MarkQWu/graphify` 的 skills/ 添加跨 skill 集成章节
   - **为何可行**：czlonkowski 的最高价值模式之一是「每个 skill 知道整个生态」，graphify 的多个 skill 之间应该有协作关系
   - **第一步**：读 graphify/skills/ 目录，列出所有 skill 名称和一句话职责；在每个 SKILL.md 末尾添加「## 与其他 skill 的集成」章节，格式参考 czlonkowski 的标准
   - **时间估算**：每个 skill 5-10 分钟，整套 graphify skills 约 30-60 分钟

3. **想法**：为 `MarkQWu/echo-sleuth-for-claude` 的工具型 agents 考虑 hooks-as-guardrails 模式
   - **为何可行**：echo-sleuth 涉及 git mining 等有副作用的工具调用（读仓库历史、可能触发 API）；PreToolUse hook 可以做参数校验，PostToolUse hook 可以做结果审计
   - **第一步**：先分析 echo-sleuth 的 agents/ 中哪些工具调用有副作用；如果有，参考 czlonkowski 的 hook 结构设计最小化 pre-tool-use 检查脚本
   - **注意事项**：hooks 引入安全审查面积——只在有真实守卫需求时添加，不要为了「看起来专业」而添加

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

**本案例 czlonkowski/n8n-skills 的核心目的**：将 n8n 平台专有领域知识打包为深度 skill 套件，用「协调者 + 知识库分离」架构管理大量领域细节，让 Claude 成为 n8n 专家

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为「专有工具 skill 化」；都是多 skill 平铺；都面向工具使用场景 | gstack 的 openclaw/skills/ 是 curated 工具集，没有配套参考文件 | 高 |
| MarkQWu/graphify | 高 | 同为「领域知识 skill 化」；graphify skills 涉及知识图谱专有概念 | graphify 的知识量可能需要伴随参考文件 | 高 |
| MarkQWu/bureau | 中 | 同为多 skill 套件，有跨 skill 协作设计 | bureau 有 command + agent 层；czlonkowski 是纯 skill 平铺 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 agents/ 和 skills/ | echo-sleuth 是代码挖掘工具，目的与 n8n 工作流设计差异大 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 自查命令 | 我的项目预期命中 | 严重度 |
|---|---|---|---|
| `comprehensive` 模糊量词 | `grep -rn "comprehensive" ~/MarkQWu/gstack/openclaw/skills/` | gstack 已知 1-2 处（数据来源）——需确认具体位置 | 低（已知少量，可定点修复） |
| skill description 字段含品质形容词 | `grep -A1 "^description:" ~/MarkQWu/*/skills/*/SKILL.md \| grep -iE "powerful|robust|comprehensive"` | 需运行确认 | 中（影响 skill 触发精确度） |
| 无伴随参考文件（所有知识塞在 SKILL.md） | `find ~/MarkQWu -path "*/skills/*" -name "*.md" ! -name "SKILL.md"` | 预期：gstack/graphify/bureau 中伴随文件极少或为零 | 中（对知识密集型 skill 影响大） |
| 跨 skill 集成章节缺失或不完整 | `grep -rL "集成\|Integration\|其他.*skill" ~/MarkQWu/*/skills/*/SKILL.md` | 预期：大多数 SKILL.md 无此章节 | 中（影响 Claude 在多 skill 生态中的导航能力） |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 中的 `comprehensive` → 替换为「覆盖 openclaw v{版本} 所有 {具体 API 类型} 的参考」→ 5 分钟可完成
- `MarkQWu/graphify` 中最长的 SKILL.md → 提取「概念定义」章节到同级 `CONCEPTS.md` → 30 分钟可完成

### 8.3 别人的更优方案

1. **领域**：处理领域知识量大的 skill 设计
   - **czlonkowski 做法**：将 skill 拆分为「协调者 SKILL.md（≤200 行）+ 按主题分片的参考文件（DATA_ACCESS.md、ERROR_PATTERNS.md 等）」
   - **我的项目现状**：`bureau/skills/scribe/SKILL.md` 或 `graphify` skills 可能是自包含的单文件，知识量超出时会越来越难维护
   - **如何借鉴**：识别知识密集型 skill（>150 行），提取规则表、API 列表、错误模式到主题化参考文件；SKILL.md 只保留调用逻辑和对参考文件的引用

2. **领域**：跨 skill 集成的水平感知图谱
   - **czlonkowski 做法**：每个 SKILL.md 包含标准化「跨 skill 集成」章节，列出所有兄弟 skill 的名称和一句话职责
   - **我的项目现状**：`MarkQWu/bureau` 的多个 skill 之间存在协作关系，但可能没有系统性地在每个 skill 里声明
   - **如何借鉴**：为 bureau 的每个 SKILL.md 添加「## 与 bureau 其他 skill 的协作」章节，格式：「`bureau:skill-name`：[一句话职责]」

3. **领域**：版本对齐纪律（manifest ↔ build 脚本）
   - **czlonkowski 做法**：plugin.json version 与 build.sh `VERSION=` 严格同步
   - **我的项目现状**：如果我的仓库有 build.sh，需要验证两者是否同步（见 §6.1 命令 5）
   - **如何借鉴**：创建单一 `VERSION` 文件，plugin.json 的版本从中读取，build.sh 的 `VERSION=` 也从中引用

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：模糊量词控制
   - **我的做法**：`MarkQWu/gstack` 的 skills 模糊量词密度为 1-2 处，是已知的少量命中
   - **czlonkowski 做法**：audit 时 8+ 处「comprehensive」/「effectively」，且 3 个月后仍未修复
   - **意义**：在这个维度上，我的仓库比这个 93 分的 exemplar 做得更好——说明我在写 skill 时对精确语言有更强的约束意识。这是可以继续保持的优势；在改造其他仓库时，把模糊量词清查作为第一步

2. **领域**：文档与实际结构的一致性
   - **我的做法**（假设）：我的 CLAUDE.md 中的安装指令与仓库实际结构对齐
   - **czlonkowski 的问题**：CLAUDE.md 提到 `npm install` 但无 `package.json`——这是文档与实现不一致
   - **意义**：CLAUDE.md 是用户的第一接触点；安装命令失败会立即损害信任。维护文档与实现的一致性是基本质量门控，不应该出现在 93 分的 exemplar 里

---

## 八、术语表

### <a name="n8n"></a>n8n

> n8n 是一个开源的工作流自动化平台（类似 Zapier 或 Make），允许用户通过可视化界面将多个应用程序和服务连接起来，自动化重复性任务。它支持自托管（self-hosting），可以在自己的服务器上运行，数据不经过第三方服务器。n8n 有两类核心「节点」（node）：功能节点（HTTP Request、Database、Send Email 等）和代码节点（允许用 JavaScript 或 Python 编写自定义逻辑）。czlonkowski/n8n-skills 的核心价值就是教会 Claude 如何正确编写这些代码节点。

### <a name="mcp-tools"></a>MCP tools

> MCP（Model Context Protocol）是 Anthropic 发布的开放协议，允许 AI 模型调用外部工具、访问外部数据源。「MCP tools」指通过 MCP 协议暴露的工具集——例如，n8n 可以通过 MCP node 调用外部 MCP 服务器提供的工具。在 czlonkowski/n8n-skills 的语境里，`n8n-mcp-tools-expert` skill 教导 Claude 如何在 n8n 工作流中正确配置和调用 MCP 工具节点；HEAD 版本新增的 hooks 脚本专门拦截 n8n MCP 工具的调用前后逻辑。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）

> 「模糊量词」是 NLPM 中对「无法被 Claude 操作化的模糊程度副词或形容词」的统称。典型例子：`comprehensive`（全面的）、`effectively`（有效地）、`many`（许多）、`significant`（显著的）。这类词的问题是：它们传达的是作者的主观评价，而非 Claude 可以执行的具体行为。当 Claude 读到「provide comprehensive coverage」时，它无从判断「comprehensive」是指「所有已知 API」还是「常见用法」，因此只能依赖训练数据的通用直觉，而不是 skill 中的专业知识。NLPM 的评分规则会对每个模糊量词实例扣分（每处 -2 分），鼓励作者将模糊量词替换为可验证的具体描述。

### <a name="exemplar"></a>exemplar（正向教学典范）

> 在 NLPM 的 auditor 流水线中，当一个仓库的 NLPM 评分 ≥90 且安全状态为 CLEAR 时，auditor-exemplar 工作流会将其标记为 `exemplar_published=True`，并在 `auditor/exemplars/` 目录生成一个教学文件。这个文件被 `skills/nlpm/rules/` 中的规则文件引用为「现实世界中的正向参考案例」。czlonkowski/n8n-skills 是少数达到 exemplar 标准的仓库之一——93 分 + CLEAR 安全——说明其设计决策可以被当作 NL 编程的最佳实践参考。

### <a name="协调者"></a>协调者（coordinator）

> 在「协调者 + 知识库分离」模式中，「协调者」是指 SKILL.md 文件本身：它定义调用场景（何时触发）、列出核心原则（怎么使用）、引用伴随参考文件（知识在哪里）。协调者是「轻量级入口」，不承担存储所有领域知识的职责。与之对应的是「知识库分片」——按主题分割的参考文件（`DATA_ACCESS.md`、`ERROR_PATTERNS.md` 等），负责深度知识的存储和维护。
