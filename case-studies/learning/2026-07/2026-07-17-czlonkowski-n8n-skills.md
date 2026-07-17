# czlonkowski/n8n-skills — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-skills
**Stars**：N/A（exemplar_published=true）| **来源**：upstream（exemplar）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-17（基于当前 HEAD v1.25.0）
**主题标签**：cross-reference, single-purpose, examples-driven, template-design, vague-quantifier

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

czlonkowski/n8n-skills 是一个针对 [n8n](#n8n) 工作流自动化平台的 Claude Code [插件](#manifest) 技能集，核心定位是「把 AI 助手变成 n8n 专家」。关键事实：

- 作者为波兰 AI 顾问 Romuald Członkowski，旗下网站 aiadvisors.pl
- 通过 Claude Code 技能机制加载，依赖 [n8n-mcp](#n8n-mcp) MCP 服务器提供 800+ 节点数据
- 版本从 Audit 时的 v1.4.0 发展到当前 v1.25.0，3 个月内增加了 6 个新技能（9→15）
- 技能分层清晰：每个 SKILL.md 配套多个知识参考文档（DATA_ACCESS.md、ERROR_PATTERNS.md 等）

### 1.2 架构剖析

**目录结构**：

```
n8n-skills/
├── .claude-plugin/plugin.json    # 插件注册清单
├── CLAUDE.md                     # 项目上下文
├── build.sh                      # zip 打包脚本
├── docs/                         # 补充文档
├── evaluations/                  # 评估用例
├── hooks/                        # Claude Code 钩子
└── skills/                       # 15 个技能目录
    ├── n8n-mcp-tools-expert/
    │   ├── SKILL.md              # 触发入口
    │   ├── OPERATIONS_GUIDE.md   # 操作参考
    │   └── SEARCH_GUIDE.md
    ├── n8n-code-javascript/
    │   ├── SKILL.md
    │   ├── DATA_ACCESS.md
    │   ├── ERROR_PATTERNS.md
    │   ├── COMMON_PATTERNS.md
    │   └── BUILTIN_FUNCTIONS.md
    ├── n8n-agents/               # 新增（Audit 后）
    │   ├── SKILL.md
    │   ├── HUMAN_REVIEW.md
    │   ├── MEMORY.md
    │   ├── SUBWORKFLOW_AS_TOOL.md
    │   ├── RAG.md
    │   ├── TOOLS.md
    │   └── EXAMPLES.md
    └── …（12 个其他技能目录）
```

**文件类型分布**：15 个 SKILL.md + 约 60 个参考文档 + 1 个 CLAUDE.md + 1 个 plugin.json + 1 个 build.sh + hooks 目录

**编排关系**：[n8n-mcp-tools-expert](#n8n-mcp) 是首要入口，通过 IMPORTANT 哨兵语句要求 Claude 在调用任何 MCP 工具前先加载此技能。其余技能平行部署，分别覆盖：JavaScript 代码、Python 代码、表达式语法、工作流模式、节点配置、验证专家、agents、self-hosting 等领域。无 router 分发层，依赖 [frontmatter](#frontmatter) description 触发。

**跨件契约**：每个 SKILL.md 末尾均有 `## Related Skills` 交叉引用表，显式列出何时切换到哪个技能。参考文档用 Markdown 链接引用，路径全部已验证存在。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「MCP 数据层 + NL 编排层双层解耦」——MCP 服务器提供 800+ 节点的结构化数据，技能提供「如何使用这些数据」的专家判断，两者严格分工
- **解决什么问题**：n8n MCP 工具有 40+ 个函数，直接使用时 Claude 经常选错工具或传错参数格式；技能层通过「先加载再操作」纠正这个问题
- **做了什么 trade-off**：技能文件较大（每个 SKILL.md + 多个参考文档），换取极高的回答精度；没有 router，换取更低的调用路径复杂度
- **反映什么认知模型**：作者把 Claude 视为需要「领域知识灌输」的新员工，技能是系统性培训材料而非临时提示词

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「MCP 数据层 + NL 编排层双层解耦」

这是 AI 工具使用类技能的典型形态：底层工具（MCP/API）负责数据，上层 NL 技能负责使用判断。

模式特征清单：
- **特征 1**：技能不复制 MCP 文档，只写「什么时候选哪个工具、为什么」
- **特征 2**：每个技能专注单一 n8n 子领域（代码/表达式/节点/验证……）
- **特征 3**：大型参考资料拆出为独立文档，SKILL.md 保持简洁可读
- **特征 4**：跨技能引用有明确的判断条件（「当用户需要 X 时切换到 Y」），不是泛化的「另见」

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 复杂工具套件（MCP/API 有 20+ 函数） | ✅ 高度适用 | 技能层能补充工具文档无法表达的「使用判断」 |
| 需要频繁迭代工具知识的项目 | ✅ 适用 | 参考文档独立维护，不影响 SKILL.md 触发逻辑 |
| 简单单一工具包装 | ❌ 过度设计 | 单个 SKILL.md 就够，不需要分层 |
| 无 MCP 支撑的纯提示词增强 | ❌ 不适用 | 这个模式的价值来自工具数据与 NL 判断的组合 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 数据层 + NL 编排层（本仓库） | czlonkowski/n8n-skills | 精度高，工具判断准确 | 需要配套 MCP 服务器 |
| 纯 NL 平铺 | addyosmani/agent-skills | 零依赖，即装即用 | 工具版本漂移时技能知识会过时 |
| 路由器 + 专科平铺 | dontbesilent2025/dbskill | 用户入口统一 | 多一跳，路由本身需要维护 |

### 2.4 改进空间

1. **当前问题**：plugin.json 缺少 `entries` 或 `skills` 数组，自动化工具无法枚举技能列表  
   **改进做法**：添加 `"skills": ["skills/n8n-mcp-tools-expert/SKILL.md", ...]` 字段  
   **预期收益**：支持 NLPM 等工具自动发现，减少 CI 中的手工维护

2. **当前问题**：CLAUDE.md 第 211 行残留陈旧的 `npm install` 安装方式，实际主路径是 `claude plugin install`  
   **改进做法**：删除 npm 安装引用或标注「实验性」  
   **预期收益**：减少用户困惑，降低 README 与实际安装路径的分歧

3. **当前问题**：6 个新增技能（n8n-agents、n8n-self-hosting 等）采用相同模板但质量参差——n8n-agents 的参考文档比早期技能丰富许多，但 CLAUDE.md 中的技能清单尚未更新  
   **改进做法**：每次新增技能后跑 NLPM cross-check，确保 CLAUDE.md 技能清单同步  
   **预期收益**：用户阅读 CLAUDE.md 时能看到准确的技能全貌

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 **93/100**（共 9 个 artifact）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 90 | "effectively"、"expert guidance" 等模糊描述词 |
| skills/n8n-code-javascript/SKILL.md | 92 | "comprehensive"（×2）、"many" |
| skills/n8n-code-python/SKILL.md | 92 | "comprehensive"/"complete"（×3） |
| skills/n8n-mcp-tools-expert/SKILL.md | 92 | description 字段中 "effectively" |
| skills/n8n-node-configuration/SKILL.md | 94 | "comprehensive" |
| skills/n8n-validation-expert/SKILL.md | 94 | "comprehensive" |
| skills/n8n-expression-syntax/SKILL.md | 95 | 无重大问题 |
| skills/n8n-workflow-patterns/SKILL.md | 95 | 无重大问题 |
| .claude-plugin/plugin.json | 95 | 无 entries/skills 数组 |

### 3.2 当时值得借鉴的模式

1. **触发升级哨兵（R04）**：n8n-mcp-tools-expert 的 description 用全大写 IMPORTANT 声明「在调用任何 MCP 工具之前先加载此技能」，将触发词从被动匹配升级为主动强制加载 → 原文：`skills/n8n-mcp-tools-expert/SKILL.md:3`

2. **catch-all 触发句（R04）**：n8n-workflow-patterns 在 9 个触发短语后加了一句「even if they don't explicitly mention 'patterns'」，捕捉用户不用该词但含义相符的查询 → 如何借鉴：在我的技能 description 末尾加一句「even if [核心词] is not mentioned」

3. **✅/❌ 代码对比（R06）**：n8n-expression-syntax 对每个错误语法都给出错/对代码对，无需散文解释 → 让读者可以直接复制「对」的版本

4. **跨件 dispatch map（R07）**：每个深度技能末尾都有「当用户需要 X 时切换到 Y 技能」的条件-目标表，而不是泛泛的「另见」列表

5. **参考文档分离（R08）**：SKILL.md 保持简洁触发逻辑，深度参考内容（ERROR_PATTERNS.md 等）拆出独立，按需引用

### 3.3 当时的缺陷

1. **问题**：多个 description 字段含 "effectively"、"comprehensive"（合计约 12 处）  
   **根本原因**：这类词无法量化，Claude 无法从中获取操作指令，变成纯装饰语；audit 时已存在但未修复  
   **自查**：我的技能中有没有同类修饰词？可用 `grep -rn -E "effectively|comprehensive|robust" ~/.claude/skills/*/SKILL.md` 检查

2. **问题**：plugin.json 缺少 entries/skills 数组  
   **根本原因**：Claude Code 插件加载器可通过文件系统约定自动发现技能，所以作者没感受到问题；但自动化工具（如 NLPM）无法枚举  
   **自查**：我的 plugin.json 里有没有 skills/entries 字段？

3. **问题**：CLAUDE.md 第 211 行的 npm install 引用是陈旧的  
   **根本原因**：快速迭代中文档更新优先级低；文档与实际安装路径脱节  
   **自查**：我的 CLAUDE.md 里有没有过时的安装命令？

### 3.4 当时的优化机会

1. 给 plugin.json 加 `skills` 数组（5 分钟可完成）
2. 删除 CLAUDE.md 里的陈旧 npm 安装行（2 分钟）
3. 将 "comprehensive"/"effectively" 替换为具体动词短语（如「列出所有 n8n 内置函数的签名」）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| description 中 "effectively"/"comprehensive" | `grep -r "effectively\|comprehensive" skills/*/SKILL.md` | **持续存在**（多处命中） | 作者未将此视为 actionable 问题 |
| plugin.json 无 entries 数组 | `cat .claude-plugin/plugin.json \| jq '.entries'` | **持续存在**（null） | 低优先级，不影响功能 |
| CLAUDE.md npm 安装引用陈旧 | `grep "npm install" CLAUDE.md` | **持续存在**（line 211） | 文档维护欠佳 |

### 4.2 架构演进

从 Audit 时（9 技能，v1.4.0）到现在（15 技能，v1.25.0）：

- **新增 6 个技能**：n8n-agents、n8n-binary-and-data、n8n-code-tool、n8n-error-handling、n8n-multi-instance、n8n-self-hosting、n8n-subworkflows、using-n8n-mcp-skills（实际 8 个，与 CLAUDE.md 描述一致）
- **版本号从 1.4.0 跳至 1.25.0**，说明作者保持高频发布节奏
- **参考文档数量大幅增加**：n8n-agents 技能下有 HUMAN_REVIEW.md、MEMORY.md、RAG.md 等多个深度参考

这说明作者的工作重心在**水平扩展覆盖面**（更多 n8n 子领域），而非纵向修复已知的 NL 质量问题。这是一种合理的优先级选择：新用户受益于更多覆盖，NL 分数细节对日常使用影响小。

### 4.3 新增的可学习模式

1. **agent 编排技能化**：n8n-agents/SKILL.md 专门处理 AI agent 工作流这一新兴场景，配套 HUMAN_REVIEW.md（何时插入人工审核）和 MEMORY.md（如何管理长期记忆）——这是将复杂子领域专项化的典型示范

2. **using-n8n-mcp-skills 元技能**：新增的 `using-n8n-mcp-skills/SKILL.md` 是一个元技能，告诉用户何时触发哪个专科技能——将「技能发现」问题转变为「技能推荐」问题，降低用户认知负担

---

## 五、校准

### 5.1 我已经在做对的

1. **技能单职责**：我的 gstack 技能（setup-deploy、document-release 等）同样遵循单一职责，每个技能只做一件事
2. **参考文档分离**：我在 gstack 的 plan-devex-review/sections/ 下也有分离的参考 section 文件，模式相近
3. **触发短语写在 description**：我的技能 frontmatter 中明确有 `triggers:` 字段列出触发短语
4. **禁止模糊词**：我的技能在正文中显式禁止 "comprehensive"、"robust" 等词——比 czlonkowski 更严格

### 5.2 挑战 / 验证

这次案例**验证**了我之前对「description 字段应写触发短语而非内容摘要」的判断——czlonkowski 的 exemplar 案例正是因为这点入选 R04 正面案例。

同时挑战了「NL 分数问题一定会被作者修复」的假设——3 个月、版本号从 1.4.0 跳到 1.25.0、从 9 个技能扩展到 15 个，vague quantifier 问题一个都没修。这说明：**对很多活跃项目来说，功能扩展的优先级远高于 NL 质量细节**。学习视角要与贡献视角分开。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能中是否含有模糊量词
grep -rn -E '\b(effectively|comprehensive|appropriate|robust|complete)\b' \
  ~/.claude/skills/*/SKILL.md \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | grep -v "No em dashes"
```
命中后怎么办：将 "comprehensive guide" 改为「列出 X 的所有 Y 字段及其类型」等具体描述。

```bash
# 检查我的 plugin.json 是否有 skills/entries 数组
find ~/.claude -name "plugin.json" -exec python3 -c \
  "import json,sys; d=json.load(open(sys.argv[1])); print(sys.argv[1], 'skills:', d.get('skills','MISSING'))" {} \;
```
命中后怎么办：如果没有 skills 数组，按 NLPM 规范补充（参考 CLAUDE.md 中的 manifest 规范）。

### 6.2 灵感 → 实施路径

1. **想法**：为我的 bureau 或 gstack 添加「元技能」入口，告诉用户有哪些技能以及何时触发  
   **为何可行**：czlonkowski 的 using-n8n-mcp-skills 证明这个模式有效，且实现成本很低（一个额外 SKILL.md）  
   **第一步**：在 MarkQWu/gstack 下创建 `gstack-skills-guide/SKILL.md`，列出所有技能触发条件对照表，30 分钟可完成

2. **想法**：在关键技能的 description 末尾加 "catch-all" 句子  
   **为何可行**：n8n-workflow-patterns 案例证明「even if [关键词] not mentioned」能显著减少触发漏匹配  
   **第一步**：修改 `MarkQWu/gstack/spec/SKILL.md` 的 description，在 trigger 列表末尾加 "even if the user doesn't say 'spec'"，10 分钟可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 czlonkowski/n8n-skills 的核心目的**：通过专业技能集教会 Claude 使用 n8n MCP 工具，覆盖工作流构建的每个子领域

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多技能集、同样覆盖多个操作子领域 | gstack 是 CEO/设计/工程角色扮演；n8n-skills 是工具使用专家 | 高 |
| MarkQWu/bureau | 中 | 同为 Claude Code plugin，有多个 SKILL.md | bureau 聚焦知识管理；n8n-skills 聚焦工具操作 | 中 |
| MarkQWu/graphify | 低 | 同为 AI 辅助工具 | graphify 是知识图谱查询；n8n-skills 是工作流编排 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| description 中模糊量词（在 forbidden list 以外的散文中） | `grep -rn "effectively\|comprehensive" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md \| grep -v "No em dashes"` | 命中 1 处：`land-and-deploy/SKILL.md:1883` 中 "reasonable timeouts" | 低 |
| plugin.json 缺少 skills/entries 数组 | `python3 -c "import json; d=json.load(open('/tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json')); print(d.get('skills', 'MISSING'))"` | **bureau 命中**：plugin.json 无 entries/skills 字段 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的 `.claude-plugin/plugin.json` → 添加 `"skills": ["skills/capture/SKILL.md", "skills/compile/SKILL.md", ...]` 字段 → 10 分钟可完成

### 8.3 别人的更优方案

1. **领域**：参考文档分离模式
   - **本案例做法**：每个技能目录下有 DATA_ACCESS.md、ERROR_PATTERNS.md、COMMON_PATTERNS.md 等独立参考文档，SKILL.md 用 Markdown 链接引用，而非把所有内容塞进 SKILL.md
   - **我的项目现状**：MarkQWu/gstack 的 `plan-devex-review/sections/` 有类似模式，但其他技能（如 `setup-deploy/SKILL.md`）所有内容都在单个大文件里（584+ 行）
   - **如何借鉴**：将 gstack/setup-deploy/SKILL.md 中「部署平台配置细节」部分拆出为 `PLATFORM_CONFIG.md`，SKILL.md 保留触发逻辑和主要步骤，通过链接引用细节

2. **领域**：IMPORTANT 哨兵加载顺序控制
   - **本案例做法**：n8n-mcp-tools-expert 的 description 用全大写 IMPORTANT 声明「在调用任何 MCP 工具之前必须加载」
   - **我的项目现状**：我的技能 description 是普通触发短语列表，没有加载顺序约束
   - **如何借鉴**：在 MarkQWu/bureau 的 `scribe/SKILL.md` description 末尾加「IMPORTANT — Load this skill before any write or append operation」

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：模糊词禁止声明
  - **我的做法**：MarkQWu/gstack 每个 SKILL.md 的底部有「No AI vocabulary: delve, crucial, robust, comprehensive...」明确禁止列表（约 584 行处）
  - **本案例做法（弱在哪）**：czlonkowski/n8n-skills 虽然有 exemplar 评分，description 中仍有 "effectively"、"comprehensive" 未修复
  - **意义**：gstack 把「禁止模糊词」作为技能输出规范，而非 NL 质量事后检查，这是更强的机制

---

## 八、术语表

### <a name="n8n"></a>n8n
> 一个开源的工作流自动化平台，类似 Zapier 或 Make（原 Integromat），可以把各种 API 和服务用可视化节点连接起来。用户拖拽节点就能搭建自动化流程，比如「收到邮件 → 提取数据 → 写入数据库 → 发 Slack 通知」。

### <a name="n8n-mcp"></a>n8n-mcp
> n8n 的 MCP（Model Context Protocol）服务器。它是一个独立运行的服务，向 Claude 提供 n8n 的工具函数（如节点查询、工作流验证、模板获取）。Claude 通过 MCP 协议调用这些函数，而不是靠记忆。

### <a name="manifest"></a>manifest
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skills 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。
