# czlonkowski/n8n-skills — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-skills
**Stars**：N/A（exemplar_published）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-16（基于当前 HEAD）
**主题标签**：`examples-driven`, `vague-quantifier`, `template-design`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
n8n-skills 是波兰 AI 顾问 Romuald Członkowski 为 [n8n](#n8n) 工作流自动化平台打造的 Claude Code 专家技能集，含 15 个深度技术 skill，覆盖 n8n 的 JavaScript/Python 代码节点、表达式语法、验证、[MCP](#MCP) 工具、工作流模式、错误处理、自托管、[子工作流](#子工作流)、binary 数据等域。2026 年活跃迭代，当前版本 1.24.1（审计时 1.4.0，20 个次版本之跨），在波兰 AI 顾问社区有较强影响力。

- **创建时间**：2025 年底（基于版本历史推断）
- **作者背景**：aiadvisors.pl 创始人，专职 n8n/AI 咨询
- **获取方式**：`claude plugin install` + `.claude-plugin/plugin.json`
- **生态位置**：目前 n8n 生态中最系统化的 Claude Code skill 集

### 1.2 架构剖析

```
czlonkowski/n8n-skills
├── skills/
│   ├── n8n-code-javascript/    ← JS 代码节点 skill + 6 个参考文件
│   │   ├── SKILL.md
│   │   ├── DATA_ACCESS.md
│   │   ├── COMMON_PATTERNS.md
│   │   ├── ERROR_PATTERNS.md
│   │   └── ...
│   ├── n8n-code-python/        ← Python 代码节点 skill + 5 个参考文件
│   ├── n8n-mcp-tools-expert/   ← MCP 工具 + SEARCH_GUIDE.md + OPERATIONS_GUIDE.md
│   ├── n8n-node-configuration/ ← 节点配置 + OPERATION_PATTERNS.md + DEPENDENCIES.md
│   ├── n8n-validation-expert/  ← 验证 + ERROR_CATALOG.md + FALSE_POSITIVES.md
│   ├── n8n-expression-syntax/  ← 表达式语法
│   ├── n8n-workflow-patterns/  ← 工作流模式
│   ├── n8n-agents/             ← Agent 构建 + RAG.md + TOOLS.md + EXAMPLES.md（新增）
│   ├── n8n-self-hosting/       ← 自托管 + SECURITY.md + QUEUE_MODE.md + DAY2.md（新增）
│   ├── n8n-subworkflows/       ← 子工作流（新增）
│   ├── n8n-binary-and-data/    ← 二进制数据（新增）
│   ├── n8n-code-tool/          ← Code Tool 节点（新增）
│   ├── n8n-error-handling/     ← 错误处理（新增）
│   ├── n8n-multi-instance/     ← 多实例（新增）
│   └── using-n8n-mcp-skills/   ← 入门引导 skill（新增）
├── .claude-plugin/
│   └── plugin.json             ← 含 hooks enforcement layer
├── build.sh                    ← 打包脚本
└── CLAUDE.md
```

- **文件类型分布**：15 个 SKILL.md / 每个 skill 目录配 2-8 个参考 MD 文件 / 1 个 [plugin.json](#manifest)（无 skills 枚举数组）
- **编排关系**：15 个 skill 平级并存，无路由层。入门 skill `using-n8n-mcp-skills` 负责引导用户选择合适的 skill。各 skill 通过 `See: [DATA_ACCESS.md]` 引用同目录内的参考文件，skill 之间没有强依赖。
- **跨件契约**：每个 skill 自包含（SKILL.md + 配套参考文件），几乎无跨 skill 引用。版本通过 `build.sh`（`VERSION="1.24.1"`）与 `plugin.json` 对齐。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「知识库外置 + SKILL 精炼」——SKILL.md 本身只写核心规则和速查表，大量参考知识放在同目录的 `DATA_ACCESS.md`、`ERROR_PATTERNS.md`、`EXAMPLES.md` 等文件里，通过 `See:` 引用。这使 SKILL.md 保持可读，同时不丢失深度。
- **解决什么问题**：n8n 的 API 文档分散，JS 代码节点有 `$input.all()` 等独特 API，不同于标准 Node.js，初学者和 AI 都容易写错。作者将这些坑系统化为 skill。
- **Trade-off**：15 个 skill 平铺后，用户需要知道"该用哪个"。无路由层时，依赖 `using-n8n-mcp-skills` 这个入口 skill 兜底，但 AI 不一定会自动加载它。
- **认知模型**：把 n8n 视为一个有领域语言的系统，每个领域（代码/表达式/验证/MCP）需要独立的专家 skill，而不是一个大而全的通用 skill。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「领域知识库卫星式」**——每个 SKILL.md 是核心节点，围绕它的是多个「卫星知识文件」（`DATA_ACCESS.md`、`ERROR_PATTERNS.md`、`COMMON_PATTERNS.md` 等）。SKILL.md 只写规则和决策路径，卫星文件存放具体案例和参考数据。

模式特征清单：
- 特征 1：每个 skill 目录包含 2-8 个配套参考 MD 文件
- 特征 2：SKILL.md 通过 `See: [filename.md]` 链接到卫星文件，不直接内联大量内容
- 特征 3：按领域单一职责划分 skill，不做大而全的通用 skill
- 特征 4：入口引导 skill (`using-n8n-mcp-skills`) 作为「元导航」帮助用户选择
- 特征 5：版本号在 `build.sh`、`plugin.json` 双处对齐

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 深度技术平台（有独特 API/语法）的 skill 库 | ✅ 高度适用 | 领域知识量大，卫星文件分层防 token 爆炸 |
| 多平台 skill 共享 | ⚠️ 部分适用 | 目录内的参考文件无法在多平台 skill 间共享 |
| 需要路由/派发的场景 | ❌ 不适用 | 无路由层，AI 需要自行判断调哪个 skill |
| 快速原型 / 小型项目 | ❌ 不适用 | 每个 skill 要维护多个参考文件，维护成本高 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域知识库卫星式（本仓库） | czlonkowski/n8n-skills | 知识深度高，SKILL.md 精炼 | 无路由，skill 间孤立 |
| 路由 + 渠道分层 | dontbesilent2025/dbskill | 统一入口，AI 无需选择 | router skill 是单点故障 |
| 单 skill 极简 | slavingia/skills | 上手极快 | 深度有限 |

### 2.4 改进空间
1. **当前问题**：plugin.json 无 `skills` 枚举数组，自动化工具无法发现 skill 列表 **改进做法**：添加 `"skills": ["skills/n8n-code-javascript", ...]` 字段，对齐 Claude Code 官方约定 **预期收益**：工具链可枚举，安装时验证文件完整性
2. **当前问题**：15 个 skill 没有路由入口，用户不知道该选哪个 **改进做法**：在 `using-n8n-mcp-skills/SKILL.md` 中加一张技能决策树（场景 → 推荐 skill），并在 CLAUDE.md 中声明为首选入口 **预期收益**：AI 初始化时自动加载入口 skill，减少"不知道该用哪个"的场景
3. **当前问题**：CLAUDE.md 中 `npm install @anthropic/claude-code-plugin-n8n-skills` 是 aspirational 指令（实际无 package.json）**改进做法**：删除该行或改为 `claude plugin install` 命令 **预期收益**：减少用户混淆

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-13 当时得分 **93/100**（7 个 skill，v1.4.0）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 90 | "effectively"、"expert guidance" 模糊量词 |
| n8n-code-javascript/SKILL.md | 92 | "comprehensive"×2，"many"×1 |
| n8n-code-python/SKILL.md | 92 | "comprehensive"/"complete"×3 |
| n8n-mcp-tools-expert/SKILL.md | 92 | "effectively" 在 description 字段 |
| n8n-node-configuration/SKILL.md | 94 | "comprehensive" 在参考节 |
| n8n-validation-expert/SKILL.md | 94 | 同上 |
| n8n-expression-syntax/SKILL.md | 95 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **卫星知识文件体系** → 为什么好：SKILL.md 保持简洁（快速加载），知识深度靠 `DATA_ACCESS.md`、`ERROR_PATTERNS.md` 等提供，AI 按需引用。原文：`skills/n8n-code-javascript/DATA_ACCESS.md` → 借鉴：在 skill 目录里添加 `EXAMPLES.md`、`ERROR_PATTERNS.md` 作为可选参考文件，主 SKILL.md 轻量化
2. **版本双处对齐** → 为什么好：`build.sh` 和 `plugin.json` 共享同一 `VERSION` 变量，无法不同步。原文：`build.sh:VERSION="1.4.0"` 对应 `plugin.json:version:"1.4.0"` → 借鉴：引入打包脚本来强制版本对齐
3. **无 Hook/无脚本的干净安全面** → 为什么好：纯 skill 库无需执行表面，安全风险最低。完全免于 SECURITY REVIEW。原文：`Security Scan: 0 findings` → 借鉴：能用纯 skill 解决的需求，不引入 hook 或安装脚本

### 3.3 当时的缺陷
1. **问题**：所有参考性文字链接描述用"comprehensive"（"the comprehensive guide"）**根本原因**：这类形容词在英文写作中很自然，但对 AI 来说是无信息的修饰词，AI 无法从"comprehensive"判断文档的实际内容类别 **自查**：我的 skill 的 `See:` 链接描述里有无类似模糊词？
2. **问题**：CLAUDE.md 中有虚构的 `npm install` 安装路径 **根本原因**：作者对 npm 发布有计划但还未实施，aspirational 内容混入了安装文档 **自查**：我的 CLAUDE.md 中有无未落地的"计划中"功能描述？
3. **问题**：`n8n_health_check()` 出现在 SKILL.md 中但 CLAUDE.md 的工具清单里没有 **根本原因**：工具清单未与实际 skill 引用同步更新 **自查**：我的工具文档与实际 skill 引用是否一致？

### 3.4 当时的优化机会
1. 在 `plugin.json` 添加 `skills` 枚举数组（`-10 分` 质量扣分点）
2. 修复 CLAUDE.md 中的虚构 npm 安装路径
3. 将所有 `See: [xxx.md] for the comprehensive guide` 的 "comprehensive" 替换为具体描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| "comprehensive"/"effectively" 模糊量词 | `grep -rn "comprehensive" skills/` → 15 处命中 | **持续存在** | 15 个 skill 中仍有此模式，扩展时被新 skill 继承 |
| npm install 虚构路径 | `grep "npm install" CLAUDE.md` → 第 211 行命中 | **持续存在** | 未随版本增长清理 |
| plugin.json 无 skills 数组 | `cat .claude-plugin/plugin.json` | **持续存在** | 从 v1.4.0 到 v1.24.1 全程未添加 |

### 4.2 架构演进
从 7 个 skill（v1.4.0）到 15 个 skill（v1.24.1），新增 8 个 skill：

| 新增 skill | 覆盖领域 | 说明 |
|---|---|---|
| n8n-agents | AI Agent 构建 | 新增 EXAMPLES.md、RAG.md、TOOLS.md 等 7 个卫星文件 |
| n8n-self-hosting | 自托管运维 | 含 SECURITY.md + QUEUE_MODE.md + SINGLE_MODE.md |
| n8n-subworkflows | 子工作流 | 模块化 workflow 拆分模式 |
| n8n-binary-and-data | 文件/二进制处理 | 填补数据流 skill 的空白 |
| n8n-code-tool | Code Tool 节点 | 与 code-javascript 互补 |
| n8n-error-handling | 错误处理 | 从 validation 中独立 |
| n8n-multi-instance | 多实例部署 | 生产运维场景 |
| using-n8n-mcp-skills | 入门引导 | 意识到需要元导航 |

这说明作者意识到：n8n 的场景比预期更广（运维、Agent、子工作流），原来的 7 skill 只覆盖了开发场景。`using-n8n-mcp-skills` 的加入说明作者意识到没有导航 skill 时用户不知道该从哪里开始。

### 4.3 新增的可学习模式
- **运维 skill 的卫星文件体系**：`n8n-self-hosting/SECURITY.md`、`n8n-self-hosting/QUEUE_MODE.md`、`n8n-self-hosting/DAY2.md` 分别对应「安全加固」「队列模式配置」「第 2 天运维」三个相互独立但关联的主题，用文件名即可理解各自职责。这是知识库卫星式模式在运维场景的具体应用。

---

## 五、校准

### 5.1 我已经在做对的
1. **bureau 的 7 个 skill 都有明确的单一职责**（capture/compile/recall/review/lint/scribe/guide），与 n8n-skills 的领域单一职责一致
2. **bureau 的 plugin.json 版本与实际 skill 版本对应**，与 n8n-skills 的版本对齐设计一致
3. **gstack 的 SKILL.md 明确把"comprehensive"列为禁用词**（`No AI vocabulary: delve, crucial, robust, comprehensive...`），我的实现比 n8n-skills 更严格——用负面规则彻底封堵，而不是靠自觉

### 5.2 挑战 / 验证
- **挑战的假设**：我原来认为「参考文件应该内联在 SKILL.md 中，方便 AI 一次加载」。但 n8n-skills 的实践证明，SKILL.md 轻量 + 按需引用卫星文件的方式同样有效，而且可以随领域深化不断扩展参考文件，而不让 SKILL.md 本身膨胀。
- **本案例未产生认知冲突**（仅验证了已有认知的不同实现方式）

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中是否有 See/Reference 链接用了模糊描述词
grep -rn -E 'See.*\b(comprehensive|complete|full|detailed|thorough)\b' \
  /tmp/my-repos/MarkQWu-bureau/skills/ \
  /tmp/my-repos/MarkQWu-gstack/ \
  --include="SKILL.md"
```
命中后：将 "comprehensive guide" 改为具体内容描述，如 "六种 `_input.all()` 用法示例"。

```bash
# 检查我的 plugin.json 是否声明了 skills 数组
cat /tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json | python3 -c \
  "import json,sys; d=json.load(sys.stdin); print('缺 skills/entries:', 'skills' not in d and 'entries' not in d)"
```
命中后：在 plugin.json 中添加 `"skills": ["skills/capture", "skills/compile", ...]` 枚举。

```bash
# 检查 CLAUDE.md 中是否有未落地的 aspirational 安装路径
grep -n "npm install\|pip install\|brew install" /tmp/my-repos/MarkQWu-bureau/CLAUDE.md
grep -n "npm install\|pip install\|brew install" /tmp/my-repos/MarkQWu-gstack/*.md 2>/dev/null
```
命中后：确认该安装路径是否真实可用；若是计划中功能，加 `(TBD)` 标注或删除。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 skills 目录添加卫星知识文件
   - **为何可行**：bureau 的 compile/recall/review 三个 skill 都有较多操作步骤，一些具体的 case pattern 可以外置为 `EXAMPLES.md`
   - **第一步**：在 `skills/recall/` 下创建 `QUERY_PATTERNS.md`，把 recall 的 3 种查询模式迁移进去，SKILL.md 改为 `See: [QUERY_PATTERNS.md]`；约 20 分钟

2. **想法**：为 gstack 添加 `using-gstack/SKILL.md` 入门引导 skill
   - **为何可行**：gstack 有 20+ skill，用户首次接触时同样面临"该用哪个"的困惑
   - **第一步**：在 gstack/ 根目录创建 `using-gstack/SKILL.md`，列出 5 个最常见场景及对应 skill 的推荐表；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 czlonkowski/n8n-skills 的核心目的**：为 n8n 平台提供深度技术 skill 集，降低开发 n8n 工作流时的 API 记忆负担

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是功能性 Claude Code skill 集，按领域单一职责组织 | bureau 是知识库管理工具，n8n-skills 是平台技术参考 | 高 |
| MarkQWu/gstack | 中 | 都是 20+ skill 的大型 skill 集 | gstack 是工作流编排，n8n-skills 是技术知识库 | 中 |
| 其余仓库 | 低/无 | — | 领域完全不同 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺 skills 枚举数组 | `cat bureau/.claude-plugin/plugin.json \| python3 -c "import json,sys; d=json.load(sys.stdin); print('skills' in d)"` | **bureau 命中**：`False`，无 skills/entries 数组 | 中 |
| vague quantifier 在参考链接中 | `grep -rn "comprehensive" gstack/ --include="SKILL.md"` | **gstack 0 实际命中**：出现的 90 处均在 "No AI vocabulary:" 禁用词列表中（非实际使用）| 无 |
| aspirational 安装路径 | `grep "npm install" bureau/CLAUDE.md` | **bureau 0 命中** | 无 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/.claude-plugin/plugin.json` → 添加 `"skills": ["skills/capture", "skills/compile", "skills/recall", "skills/review", "skills/scribe", "skills/lint", "skills/guide"]` → 5 分钟可完成

### 7.3 别人的更优方案

1. **领域**：卫星知识文件体系
   - **本案例做法**：每个 skill 目录有独立的 `DATA_ACCESS.md`、`ERROR_PATTERNS.md`、`EXAMPLES.md` 等，SKILL.md 通过 `See:` 引用，保持精炼（路径：`skills/n8n-code-javascript/DATA_ACCESS.md`）
   - **我的项目现状**：bureau 的 `skills/recall/SKILL.md` 把所有查询模式都内联在 SKILL.md 中，导致文件较长
   - **如何借鉴**：将查询模式、示例案例迁移到 `QUERY_PATTERNS.md`，SKILL.md 改为引用式，diff 思路：删除 SKILL.md 第 50-120 行（具体案例），新建 `skills/recall/QUERY_PATTERNS.md`

2. **领域**：入门引导 skill
   - **本案例做法**：`using-n8n-mcp-skills/SKILL.md` 作为 15 个 skill 的导航入口（版本 1.24.1 新增）
   - **我的项目现状**：gstack 有 20+ skill 但无导航 skill，用户和 AI 都需要自行探索
   - **如何借鉴**：创建 `gstack/using-gstack/SKILL.md`，列出场景 → skill 映射表

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：模糊量词管理
- **我的做法**（gstack）：在每个 SKILL.md 末尾加 "No AI vocabulary:" 负面词列表（`gstack/setup-deploy/SKILL.md:584` 等），通过规则而非自觉避免模糊词
- **本案例做法（弱在哪）**：SKILL.md 中 15 处 "comprehensive" 仍存在，历史存量未清理，新 skill 沿袭了旧模式
- **意义**：gstack 的"负面词列表"机制更可靠——规则即约束，不依赖作者记忆。若被人审到，这是亮点。

---

## 八、术语表

### <a name="n8n"></a>n8n
> 一个低代码工作流自动化工具（类似 Zapier 但可自托管）。通过图形界面把不同服务连接成"工作流"，也支持 JavaScript/Python 代码节点做复杂处理。n8n-skills 里的 skill 专门教 AI 怎么写 n8n 的代码节点。

### <a name="MCP"></a>MCP
> Model Context Protocol，Claude Code 的工具调用协议。n8n 有专门的 n8n-mcp 服务器，让 Claude 可以通过 MCP 工具直接操作 n8n 工作流，不只是写代码。

### <a name="子工作流"></a>子工作流
> 在 n8n 中，一个工作流可以调用另一个工作流（子工作流），类似函数调用。这让大型工作流可以模块化拆分，提升复用性。

### <a name="manifest"></a>plugin.json（manifest）
> Claude Code 插件的"清单文件"，声明插件的名称、版本、作者、包含哪些 skill/command 等信息。Claude Code 读 plugin.json 才知道这个插件里有什么。
