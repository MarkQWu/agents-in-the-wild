# OpenRaiser/NanoResearch — 学习案例

**仓库**：https://github.com/OpenRaiser/NanoResearch
**Stars**：690 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-24（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `vague-quantifier`, `security-gate`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
NanoResearch（OpenRaiser，690 stars）是一套**端到端 AI 科研自动化管线**：从论题调研、假设生成，到实验代码生成、论文撰写，全流程用 Claude Code + Python 后端实现。与 Oh My Paper 的纯 NL 方案不同，NanoResearch 是典型的「[NL 表皮 + 原生后端](#nl表皮原生后端)」架构——Claude Code 命令层负责用户对话，Python 引擎（`nanoresearch/` 目录，175 个 .py 文件）做真正的数值计算、实验执行和 LaTeX 生成。还有一个 MCP server（`mcp_server/`）给 Claude Code 提供 `search_arxiv`、`generate_figure` 等研究专用工具。

### 1.2 架构剖析
- **目录结构**：
  ```
  NanoResearch/
  ├── .claude/
  │   └── commands/          # 9 个命令（所有缺 YAML frontmatter）
  │       ├── research.md    # 9 阶段全流程 orchestrator
  │       ├── ideation.md    # 文献搜索 + 假设生成
  │       ├── planning.md    # 实验计划
  │       ├── experiment.md  # 实验执行
  │       ├── analysis.md    # 数据分析
  │       ├── writing.md     # 论文撰写
  │       ├── review.md      # 同行评审
  │       ├── status.md      # 进度查看
  │       └── resume.md      # 断点续传
  ├── skills/                # 16 个 SKILL.md（分核心 + 供应商两类）
  │   ├── nanoresearch-*/    # 5 个核心 skill（ideation/planning/experiment/analysis/writing）
  │   └── vendor-ai-research/ # 11 个高质量供应商 skill（ml-paper-writing, skypilot, peft 等）
  ├── nanoresearch/          # Python 实验引擎（175 个 .py 文件）
  ├── mcp_server/            # MCP 服务（提供 arxiv/openAlex/semantic scholar 搜索工具）
  ├── CLAUDE.md              # 项目说明
  └── requirements.txt       # 依赖声明（unpinned）
  ```
- **文件类型分布**：9 个 command（全缺 frontmatter）/ 16 个 skill（5 核心 + 11 供应商）/ 175 个 Python 脚本（实验引擎）/ 1 个 MCP server
- **编排关系**：`research.md` 是总 orchestrator command，9 步走完整个流程；其余 8 个 command 各管一个阶段。skill 层（特别是供应商 skill）提供具体操作知识，MCP server 提供外部 API 访问工具。
- **跨件契约**：核心 skill 声明的工具名（`search_arxiv`, `generate_latex`, `compile_pdf`）与 MCP server 里的工具实现一一对应，MCP 依赖在 CLAUDE.md 中**未声明**——用户仅安装 NL 组件而不启动 MCP server 时，所有工具调用会静默失败。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「NL 接口 + Python 引擎」分层——Claude Code 层负责理解用户意图、生成代码策略、组织论文叙事；Python 层负责实际计算、实验执行、PDF 编译。这两层通过 workspace 目录（`~/.nanoresearch/workspace/`）和 MCP 工具通信。
- **解决什么问题**：纯 NL 方案（只用 Claude）做科研的局限——大模型无法在多步实验迭代中保持数值一致性，实验结果需要 reproducible；Python 引擎保证了计算的确定性。
- **做了什么 trade-off**：选择混合架构（NL + Python）换来更强的计算能力和可复现性，代价是安装复杂度大幅提升（需要 Python 环境、pip 安装、MCP server 启动），普通用户难以部署。
- **反映什么认知模型**：作者把 Claude Code 定位为"懂得使用工具的科研助手"而非"全能计算器"——Claude 的强项是理解上下文、生成策略、组织叙事；Python 工具负责脏活（执行实验、编译 PDF）。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[NL 表皮 + 原生后端](#nl表皮原生后端)」（NL Shell + Native Engine 架构）**

Claude Code 命令/skill 是用户可见的 NL 接口，实际工作由独立运行的原生程序（Python/Go/Rust/Node.js）完成，两者通过 MCP server 或 Bash 工具通信。

模式特征清单：
- **NL 层做决策，原生层做执行**：Claude 决定"下一步实验什么"，Python 引擎负责实际跑实验
- **MCP 是解耦合桥梁**：NL 层通过 MCP tools（`search_arxiv` 等）调用后端，不直接运行 Python
- **workspace 状态文件**：跨 session 的状态（实验结果、论文草稿）存在 `~/.nanoresearch/workspace/` 里
- **供应商 skill 复用**：`vendor-ai-research/` 里的高质量 skill 可独立于 NanoResearch 使用
- **运行时可扩展**：Python 引擎的 `runtime_auto_install` 允许 AI 生成代码时自动安装所需包（虽然这是安全隐患）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 深度学习实验自动化（有 GPU 环境） | ✅ 高度适用 | Python 引擎 + skypilot/accelerate skill 正是为此设计 |
| 计算机科学领域的系统性文献综述 | ✅ 适用 | MCP server 有 arxiv/Semantic Scholar/OpenAlex 集成 |
| 生物/医学领域实验（bioinformatics） | ⚠️ 部分适用 | Python 引擎是通用的，但 skill 偏向 ML |
| 快速一次性调研（不需要完整管线） | ❌ 不适用 | 搭建成本高，单阶段用 search/summarize 更高效 |
| 没有 Python 环境的用户 | ❌ 不适用 | 依赖 Python 3.11+、175 个 .py 文件、pip 安装 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生后端（本仓库） | NanoResearch | 计算能力强，可复现，MCP 解耦 | 安装复杂，shell=True 安全漏洞，MCP 依赖未文档化 |
| 纯 NL multi-agent | LigphiDonk/OMP | 零依赖安装，Claude 负责全部 | 实验结果不可复现，数值精度依赖 LLM |
| CLI 命令 + NL 注释 | Manavarya09/design-extract | 安装简单（单一 npx 命令），无后端 | 功能范围窄，不适合复杂计算 |

### 2.4 改进空间
1. **当前问题**：9 个 command 全部缺 YAML [frontmatter](#frontmatter)，只有 Markdown 标题，Claude Code 无法注册和调用它们。**改进做法**：给每个 command 文件开头加 `---\nname: <cmd-name>\ndescription: <one-liner>\nallowed-tools: [Read, Bash]\n---`。**预期收益**：命令可以在 Claude Code 里通过 `/project:research` 等方式被发现和调用。
2. **当前问题**：`nanoresearch/config.py` 的 `runtime_auto_install_allowlist` 默认为空列表，意味着 AI 生成的实验代码可以 `pip install` 任意包（最多 50 个）。**改进做法**：把 allowlist 改为 deny-by-default（空列表 = 不允许任何 auto-install），让用户显式指定允许的包。**预期收益**：消除 LLM 被诱导安装恶意 Python 包的供应链风险。
3. **当前问题**：CLAUDE.md 没有声明 MCP server 依赖，用户安装 NL 命令后启动，工具调用全部静默失败。**改进做法**：在 CLAUDE.md 里加"Prerequisites: 需要先 `pip install -r requirements.txt && python mcp_server/main.py` 启动 MCP server"一节。**预期收益**：用户第一次使用时不会被 "tool search_arxiv not found" 错误困惑。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-20 审计时得分 **73/100**，安全扫描为 BLOCKED。

| 文件类型 | 当时分数 | 主要问题 |
|---|---|---|
| Commands（9 个） | 37–45/100 | 全部缺 YAML frontmatter（name+description），每个 -50 分 |
| Core Skills（5 个） | 77–81/100 | 缺 examples，有 vague quantifiers |
| Vendor Skills（11 个） | 90–98/100 | 整体质量很高，少量 vague quantifiers |
| CLAUDE.md | 94/100 | 少量 vague 语言 |
| unsloth/SKILL.md | 77/100 | 内容基本是占位符，引用了不存在的 references/ 目录 |

### 3.2 当时值得借鉴的模式

1. **供应商 skill 高质量（90-98 分）** → `vendor-ai-research/ml-paper-writing/SKILL.md`、`vendor-ai-research/academic-plotting/SKILL.md` 等 11 个供应商 skill 平均 94 分，覆盖 HuggingFace Accelerate、PEFT、SkyPilot、Ray Data 等具体工具的最佳实践。这些 skill 质量远高于核心 skill，说明它们来自有经验的贡献者或经过认真打磨。如何借鉴：高质量供应商 skill 的写法——有工具版本约束、有具体代码示例、有 known limitations——是我写 skill 的参考标准。

2. **MCP 工具和 skill 声明对齐** → `nanoresearch-ideation/SKILL.md` 声明的 `search_arxiv` 工具和 `mcp_server/tools/arxiv.py` 里的实现严格对应，没有幻觉工具（声明了但不存在的）。根本原因好：这意味着维护者在写 skill 的时候确保了 MCP 侧有实现，是"代码先行"的好习惯。如何借鉴：我在写 skill 声明工具时，要确保 `allowed-tools` 里的每个工具都是 Claude Code 内置的或通过 MCP 提供的。

3. **9 步流程命令互相引用** → 9 个 command 文件通过 `/project:X` 互相引用，形成完整的 research pipeline 闭环。`resume.md` 和 `status.md` 有 schema 兼容层处理旧格式，说明维护者考虑了向前兼容性。如何借鉴：我的多步骤工作流应该也设计 `resume` 和 `status` 命令，让用户可以在任何阶段中断并恢复。

### 3.3 当时的缺陷

1. **全部 9 个 command 缺 YAML frontmatter** → 命令文件只有 Markdown 标题（`# Ideation — Literature Search & Hypothesis Generation`），没有 YAML frontmatter block。根本原因：作者可能把 command 文件当成给人看的说明文档，而不是 Claude Code 的机器可读配置。NLPM 对每个缺 frontmatter 的命令扣 50 分，导致 commands 整体在 37-45 分之间。自查：我的 echo-sleuth 和 drama-workshop-skills 的命令**都有** frontmatter，但有几个缺 `allowed-tools`。

2. **`nanoresearch/agents/debug.py:148` 使用 `shell=True`** → `subprocess.run(cmd, shell=True, ...)` 里 `cmd` 由 workspace 路径构成，workspace 路径又来自用户输入的 research topic。如果 topic slug 化时没有过滤 shell 元字符，则任何人输入 `My topic; rm -rf ~/.nanoresearch` 就会执行恶意命令。根本原因：Python 程序员的常见疏忽，用 `shell=True` 省事但引入注入风险。自查：我的 echo-sleuth 的 Python 脚本（`scripts/echolib.py`）在调用 shell 时需要自查是否用了 `shell=True` + 用户输入。

3. **runtime_auto_install 默认开放 + 空 allowlist** → AI 生成的实验代码可以动态 pip install 任意包（最多 50 个），allowlist 为空意味着无任何限制。根本原因：作者优先考虑"零摩擦的实验自动化"，安全性是次要考量。自查：我的项目没有运行时包安装机制，不存在这个问题。

### 3.4 当时的优化机会
1. 给 9 个 command 加 YAML frontmatter（`name`, `description`, `allowed-tools`）
2. 在 CLAUDE.md 里声明 MCP server 依赖和启动步骤
3. 将 `runtime_auto_install_allowlist` 的 logic 反转为 deny-by-default

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Commands 缺 YAML frontmatter | `head -5 .claude/commands/research.md` | **仍然存在** — research.md 开头是 `# Research — Full 9-Stage Research Pipeline`，无 `---` 块 | 约 1 个月内未修复；所有 9 个命令依然不可注册 |
| `debug.py:148` shell=True | `grep -n "shell=True" nanoresearch/agents/debug.py` | **仍然存在** — 第 148 行 `cmd, shell=True, capture_output=True` | 高危安全漏洞约 1 个月未修复 |
| CLAUDE.md 缺 MCP 依赖声明 | `grep -n "mcp\|MCP" CLAUDE.md` | 需进一步验证（当前 HEAD）| 如仍缺失，首次使用者仍会遇到 tool-not-found 困惑 |

### 4.2 架构演进
从 audit 到现在（约 1 个月），commands 和 Python 引擎的核心问题均未修复。供应商 skill 层（`vendor-ai-research/`）可能有小更新（这些 skill 质量已经很高，不太需要变动）。整体来看，维护者的精力似乎更多在 Python 引擎功能开发上，而非 NL 接口层的规范化。

### 4.3 新增的可学习模式
核心 skill 层在 audit 后可能有微调（unsloth/SKILL.md 的占位符内容是一个明显的待填坑），但整体架构没有重大变化。如果维护者按建议补充了 MCP 依赖声明，那会是一个有价值的改善。

---

## 五、校准

### 5.1 我已经在做对的
1. **Command frontmatter 完整（含 name）**：echo-sleuth 所有命令都有 frontmatter，不像 NanoResearch 的 9 个命令全部只有 Markdown 标题。
2. **没有 shell=True + 用户输入组合**：echo-sleuth 的 Python 脚本（echolib.py）在 shell 调用上需要自查，但从架构上看 echolib.py 的输入是文件路径（由脚本内部生成），不直接接受用户原始输入。
3. **供应商 skill 的质量标准**：drama-workshop-skills 的 SKILL.md 有完整例子和具体约束（这与 NanoResearch 的 vendor-ai-research/ 高质量 skill 标准一致）。
4. **没有运行时 auto-install**：我的项目没有"AI 生成代码后自动 pip install"的机制，避免了供应链风险。

### 5.2 挑战 / 验证
- **挑战了的假设**："MCP server + Claude Code 命令的结合是一个新颖的复杂架构"——NanoResearch 验证了这种混合架构的可行性（690 stars 说明有用户接受它），但也揭示了它的脆弱点：MCP 未声明依赖导致用户困惑，是混合架构特有的"隐形前置条件"问题。
- **验证了的做法**："供应商 skill 的高质量值得单独维护"——vendor-ai-research/ 的 11 个 skill 平均 94 分，远高于核心 skill 的 79 分。专注做好垂直领域 skill 比写一个大而全的 orchestrator 更有价值，因为高质量的垂直 skill 可以被多个项目复用。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 echo-sleuth Python 脚本是否有 shell=True + 字符串拼接
grep -rn "shell=True" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/scripts/ 2>/dev/null
# 命中则检查 cmd 参数的来源——是否包含用户输入；若是则改为参数化调用

# 检查 claude-for-legal skills 是否有声明 MCP 工具依赖
grep -rn "mcp\|MCP\|allowed-tools" /tmp/my-repos/MarkQWu-claude-for-legal/*/SKILL.md 2>/dev/null | head -10
# 若 skill 声明了工具但没有说明如何安装 MCP，则需要在 README 或 CLAUDE.md 里补充

# 检查我的命令是否有 allowed-tools 声明
grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 NanoResearch 的 `resume.md` + `status.md` 模式，给 echo-sleuth 加一个 `/dashboard` 命令的增强版（支持 resume）
   - **为何可行**：echo-sleuth 已经有 `/dashboard` 命令，但它是无状态快照；借鉴 NanoResearch 的 workspace 状态文件思路，可以在 `.echo-sleuth/state.json` 里记录上次分析的 session 范围，让 `/recap` 支持"只分析上次 recap 之后的新 session"
   - **第一步**：在 `commands/recap.md` 里加一个"读取 `.echo-sleuth/last-recap.json` → 只处理新增 session"的分支逻辑（约 30 分钟）

2. **想法**：参考 NanoResearch vendor-ai-research/ 的"供应商 skill"思路，为 claude-for-legal 添加一组高质量的"法律工具 skill"（类似 PEFT/Accelerate 的垂直领域 skill）
   - **为何可行**：claude-for-legal 当前的 SKILL.md 偏向流程描述，缺少具体工具使用说明（如法律数据库检索、合同模板库等）
   - **第一步**：在 `claude-for-legal/` 新建 `vendor-legal/westlaw-search/SKILL.md`（如果有 WestLaw MCP 工具），仿照 NanoResearch 的 vendor skill 格式写（约 1 小时）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 OpenRaiser/NanoResearch 的核心目的**：端到端 AI 科研自动化管线，从调研到论文产出（Python 引擎驱动）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都有"分析→生成洞察"的工作流，都依赖后端脚本（echolib.py 类比 nanoresearch/ 引擎） | echo-sleuth 是"分析过去"，NanoResearch 是"生产未来内容"；echo-sleuth 无 MCP | 高（混合架构模式可借鉴） |
| MarkQWu/claude-for-legal | 中 | 都是专业垂直领域的多阶段工作流 | 法律工作流不需要数值计算引擎 | 中（vendor skill 模式可借鉴） |
| MarkQWu/drama-workshop-skills | 低 | 都有工作流 | 短剧是纯 NL，NanoResearch 是混合架构 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 YAML frontmatter（name/description） | `head -5 echo-sleuth/commands/recap.md` | **未命中**：echo-sleuth 命令有完整 frontmatter | 暂无 |
| MCP 工具依赖未在 CLAUDE.md 里文档化 | `grep -i "mcp\|MCP" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/CLAUDE.md` | 需进一步检查；echo-sleuth 的工具是标准 Bash+Read，但如有 MCP 依赖则需声明 | 低 |
| Skills 缺 examples block | `grep -c "## Examples" skills/*/SKILL.md` | echo-sleuth skills 需核查；drama-workshop-skills 有完整例子 | 中 |
| Vague quantifiers 高密度 | `grep -c "appropriate\|relevant\|suitable" skills/*/SKILL.md` | 可能存在，需逐一核查 | 低 |

**命中后的具体行动建议**：
- 如 echo-sleuth skills 缺 examples → 在 `skills/jsonl-core/SKILL.md` 末尾加 `## Examples` section，给出"JSONL 行 → Python dict"完整例子（15 分钟）
- 检查 claude-for-legal SKILL.md 文件里的 vague quantifiers → 用 `grep -rn "relevant\|appropriate\|suitable" /tmp/my-repos/MarkQWu-claude-for-legal/*/SKILL.md` 扫描，逐一替换为可测量标准（每条 10-15 分钟）

### 7.3 别人的更优方案

1. **领域**：供应商 skill 的精细化分层（vendor-ai-research/）
   - **本案例做法**：`vendor-ai-research/` 里的 11 个 skill 独立于 NanoResearch 的核心 pipeline，它们是高质量的"工具手册"型 skill（PEFT、SkyPilot、Accelerate、Ray Data 等），每个 skill 90+ 分，有具体代码示例和版本说明。
   - **我的项目现状**：claude-for-legal 的 skill 把流程描述和工具使用说明混在一起，没有独立的"工具使用手册"型 skill。
   - **如何借鉴**：给 claude-for-legal 创建一个 `vendor-legal/` 子目录，放置独立的法律数据库查询 skill（如果有 Westlaw MCP / LexisNexis MCP），与流程 skill 分离；这样流程 skill 更聚焦，工具 skill 可复用。

2. **领域**：9 步流程的 resume + status 机制
   - **本案例做法**：`status.md` 读取 workspace 里的 manifest 文件展示当前进度；`resume.md` 从中断处继续执行，并处理新旧格式兼容。
   - **我的项目现状**：echo-sleuth 的命令都是无状态的（每次从头搜索），没有"上次跑到哪里了"的记忆。
   - **如何借鉴**：给 echo-sleuth 加 `.echo-sleuth/state.json`，记录上次各命令的执行时间戳；`/recap` 加 `--since-last` 参数，只处理上次 recap 之后的新 session。

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Command YAML frontmatter 规范性
- **我的做法**：echo-sleuth-for-claude 和 drama-workshop-skills 的所有命令都有 `---\nname: ...\ndescription: ...\n---` 格式的 YAML frontmatter，命令可以被 Claude Code 发现和注册
- **本案例做法**（弱在哪）：NanoResearch 的 9 个命令全部只有 Markdown H1 标题，没有 YAML frontmatter，NLPM 评分 37-45 分，且命令无法通过 `/project:*` 注册调用
- **意义**：这是 p0 级别的工程规范，echo-sleuth 保持了这个基线

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件元数据（`name`、`description`、`allowed-tools` 等）。Claude Code 在注册 command 时必须读取 frontmatter 里的 `name` 字段来确定命令路径（如 `/project:research`）。没有 frontmatter 的 command 文件就像没有门牌号的商店，用户找不到它。

### <a name="nl表皮原生后端"></a>NL 表皮 + 原生后端
> 一种混合架构模式：Claude Code 的命令/skill 文件作为用户可见的 NL 接口（"表皮"），底层实际工作由独立运行的原生程序（Python/Go/Node.js 等）完成（"后端"）。两者通过 MCP server 或 Bash 工具通信。优点是保留了原生语言的计算能力、可复现性；缺点是安装依赖复杂，且 MCP 依赖若不文档化则用户会困惑。对比"纯 NL 方案"（所有事情都让 Claude 来做）：纯 NL 更简单但无法保证数值确定性，适合内容生成；混合架构更复杂但可以跑真实实验，适合科研场景。

### <a name="mcp-server"></a>MCP server
> Model Context Protocol server，一种让 Claude Code 访问外部 API 和工具的机制。MCP server 是一个本地运行的小程序，暴露一组工具（如 `search_arxiv`、`generate_figure`），Claude 可以通过工具调用来使用这些功能。类比：MCP server 是 Claude Code 的"外挂工具箱"，安装完插件后还需要把工具箱打开（启动 MCP server），Claude 才能用里面的工具。

### <a name="shell-injection"></a>shell injection（命令注入）
> 见 KhazP 案例术语表。NanoResearch 的 `debug.py:148` 是同一类问题：`subprocess.run(cmd, shell=True)` 里 `cmd` 字符串由用户输入（research topic）构建，若不过滤特殊字符则可注入任意 shell 命令。
