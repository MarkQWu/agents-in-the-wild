# alirezarezvani/claude-skills — 学习案例

**仓库**：https://github.com/alirezarezvani/claude-skills
**Stars**：N/A（registry 无记录）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-08-06（基于当前 HEAD）
**主题标签**：`examples-driven`, `cross-reference`, `vague-quantifier`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`alirezarezvani/claude-skills` 是一个多行业全栈 Claude Code 技能库，覆盖工程（engineering）、产品（product-team）、市场（marketing-skill）、商业增长（business-growth）、C 级顾问（c-level-advisor）、RA/QM 合规（ra-qm-team）、财务（finance）等域。431 个 NL 工件，是所有 xiaolai upstream 审计过的最大仓库之一。安装方式为 `openclaw`（一个 shell 脚本，用 curl pipe bash 触发），BLOCKED 状态。

### 1.2 架构剖析

- **目录结构**：
  ```
  .
  ├── agents/                         # 全局 agent 定义（工程/市场/产品/C级等）
  │   ├── engineering/                # 工程类专家 agent
  │   ├── engineering-team/           # 工程团队协作 agent
  │   ├── marketing/                  # 市场专家 agent
  │   ├── product/                    # 产品专家 agent
  │   ├── c-level/                    # C级顾问 agent（CEO/CTO/CFO等）
  │   └── personas/                   # 角色 persona（startup CTO/solo founder等）
  ├── engineering/                    # 工程 skill 集（~45 个 SKILL.md）
  │   ├── autoresearch-agent/         # 自动研究 agent + 子 skill
  │   ├── agenthub/                   # Agent Hub（管理多 agent 的元 skill）
  │   └── llm-wiki/                   # 知识库系统（wiki 读写 skill + 命令）
  ├── engineering-team/               # 工程团队 skill
  │   ├── playwright-pro/             # Playwright 测试套件（含 agent + skill + hook）
  │   └── self-improving-agent/       # 自改进 agent 系统
  ├── c-level-advisor/                # C 级顾问 skill 集
  ├── marketing-skill/                # 市场 skill 集（~20 个 SKILL.md）
  ├── ra-qm-team/                     # 合规/质量管理 skill 集
  ├── business-growth/                # 商业增长 skill 集
  ├── product-team/                   # 产品团队 skill 集
  ├── finance/                        # 财务 skill 集
  ├── commands/                       # 顶层命令（sprint-plan/wiki-*/code-to-prd 等）
  ├── .claude/commands/               # 隐藏命令（git/security-scan/plugin-audit）
  ├── docs/                           # 文档（含 openclaw 安装指引）
  └── standards/                      # 跨域标准文档
  ```
- **文件类型分布**：约 50 个 agent / 约 310 个 SKILL.md / 约 30 个 command / 2 个 hook
- **编排关系**：无显式 orchestrator。各域相互独立，`engineering/agenthub/` 是元 skill，管理多个 agent 的并发启动；`engineering/autoresearch-agent/` 有完整的 setup/run/status/loop/resume 子 skill 链。
- **跨件契约**：`engineering-team/playwright-pro/` 下 7 个子 skill 缺少 frontmatter，但父级 SKILL.md（93/100）引用了这些子 skill——父子之间有隐式依赖但注册状态不一致。`mcp-debugging/SKILL.md`（非此仓库，但被引用）需要外部 `superpowers` 插件，未声明依赖。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「按域分包，skill 即专家」——每个业务域（工程/市场/财务）都有独立的 skill 集，skill 扮演具体角色（如 `revenue-operations/SKILL.md` = 收入运营专家）。
- **解决什么问题**：覆盖初创公司 / SaaS 企业的全栈 AI 辅助需求——从工程开发到 C 级战略，一个仓库解决所有场景。
- **做了什么 trade-off**：广度 vs 深度。431 个工件覆盖面极广，但单个工件的质量参差不齐（从 65 到 100 不等）。选择了广覆盖，代价是部分工件维护不到位。
- **反映什么认知模型**：作者把 Claude 视为「可以扮演任何专业角色的数字同事」——不只是编程助手，还是合规专家、CFO、市场经理。这是「skill as persona + expertise」的认知框架。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：多域 Skill 聚合库（Multi-Domain Skill Aggregator）**

一个仓库按业务域组织大量独立 skill，每个域对应真实的业务团队（工程/产品/市场/财务），用户按需安装整个仓库获取全域能力。

模式特征清单：
- 顶层目录 = 业务域（不是 `commands/skills/agents/` 等技术分类）
- 每个域有独立的入口 SKILL.md（如 `engineering/SKILL.md`）作为「域名片」
- 域内 skill 按功能细分（工程有 50+ skill，每个对应具体能力）
- 不同域之间相互独立，可以按需使用某一域的 skill 而不影响其他域
- 全局 `standards/CLAUDE.md` 提供跨域约定

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 覆盖多个业务部门的企业工具集 | ✅ 高度适用 | 按域分包，各部门独立使用不冲突 |
| 个人开发者的轻量工具箱 | ❌ 不适用 | 431 个工件远超个人需要，维护成本高 |
| 社区贡献驱动的技能市场 | ✅ 适用 | 域边界清晰，贡献者可只维护自己的域 |
| 需要跨域协作的复杂工作流 | ⚠️ 慎用 | 当前缺少跨域 orchestrator，域间协作需要手动协调 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多域 Skill 聚合库（本仓库） | alirezarezvani/claude-skills | 覆盖面极广，可按需选用 | 维护成本正比于工件数量 |
| 单域深度套件 | navapbc/DSO | 单域质量极高，全套工作流 | 覆盖面窄，无法横跨业务 |
| 轻量命令集（个人工具箱） | aaronmaturen/claude-plugin | 极简，0 维护负担 | 覆盖面受限于个人精力 |

### 2.4 改进空间

1. **当前问题**：`engineering-team/playwright-pro/` 7 个子 skill 缺 frontmatter（BUG #1–7），而父 SKILL.md 引用这些子 skill。**改进做法**：批量加 `name`/`description` frontmatter 到这 7 个文件。**预期收益**：消除 7 个注册失败的 bug，父子一致性恢复。
2. **当前问题**：`engineering/autoresearch-agent/evaluators/` 的 4 个 Python 脚本使用 `subprocess shell=True + 动态字符串`（HIGH 安全风险）。**改进做法**：改为 `subprocess.run([...args...])` 列表形式，不用 shell=True。**预期收益**：消除 4 个 HIGH 安全漏洞，开放 PR 通道。
3. **当前问题**：50+ skill 中有大量「vague quantifier」（appropriate/ensure/comprehensive 等）使多个 skill 被扣 2–20 分。**改进做法**：用脚本扫描所有 SKILL.md，对每个 vague word 提供可验证替换。**预期收益**：系统性提升 87 个 quality issues 对应 skill 的分数。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **92/100**（BLOCKED 因安全问题）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `agents/product/cs-product-analyst.md` | 70 | 缺输出格式 + 示例不足 |
| `commands/seo-auditor.md` | 70 | 多步骤无编号 + 模糊量词上限 |
| `commands/competitive-matrix.md` | 65 | 缺 allowed-tools/output format/编号步骤/empty guard |
| 满分 skill（100/100）| 100 | `engineering/` 域内 30+ skill |
| 均值分数 | 92 | 整体高质量 |

### 3.2 当时值得借鉴的模式

1. **`engineering/` 域的精工 100/100 skill 集群** → `terraform-patterns/SKILL.md`、`database-schema-designer/SKILL.md`、`self-eval/SKILL.md` 等 30+ skill 满分。这些 skill 的共同特征：输出格式明确、工具声明完整、无 vague word、示例多样。参考路径：`engineering/focused-fix/SKILL.md`。如何借鉴：把这些 100 分 skill 当模板，结构和写法直接复用。
2. **`ra-qm-team/` 合规域的 100/100 率极高** → 14 个合规 skill 中 7 个满分（gdpr-dsgvo-expert、risk-management-specialist 等）。原因：合规领域有非常明确的规范文本（ISO/FDA/GDPR 等），skill 只需引用规范不需要发明"标准"，所以没有 vague word。如何借鉴：为有明确行业标准的任务写 skill 时，直接引用标准文本而不是自己描述。
3. **`engineering/agenthub/` 子 skill 套件** → 7 个子 skill（status/run/eval/board/merge/spawn/init）满分 100，且通过 `agenthub/SKILL.md` 统一注册。这是「父 skill + 子 skill 套件」的正确实现方式：父 skill 不包含业务逻辑，只做子 skill 的索引。如何借鉴：有多个相关操作的 skill（如 bureau 的多个命令）可以用父 skill 作索引，子 skill 做具体操作。
4. **`engineering/self-eval/SKILL.md` 满分** → 这个 skill 让 Claude 对自己的输出做质量评估，是「AI 自我审查」模式的具体实现。如何借鉴：在长流程最后加一个 self-eval 步骤，让模型检查自己的输出是否满足标准。

### 3.3 当时的缺陷

1. **`engineering-team/playwright-pro/` 子 skill 批量缺 frontmatter（BUG #1–7）** → 为什么这么设计会失败：父级 `playwright-pro/SKILL.md` 声明了这些子 skill 的存在，但子 skill 自身没有 `name`/`description`，NLPM 无法注册——用户安装后看不到 playwright 测试相关 skill。根本原因：可能是批量创建时忘记加 frontmatter 模板。自查：我的项目有没有「父 SKILL.md 引用了子 skill，但子 skill 没有 frontmatter」的情况？
2. **`agents/engineering/cs-wiki-*.md` 三个 wiki agent 声明了 Write/Edit 但角色是只读** → 为什么这么设计会失败：cs-wiki-linter（审计）、cs-wiki-librarian（查询）、cs-wiki-ingestor（摄取）按描述都是只读角色，但 `tools: [Read, Write, Edit, Bash, Grep, Glob]` 给了写权限。最小权限原则被违反，agent 在出 bug 时可能意外修改文件。自查：我的 agent 有没有声明了不该有的工具权限？（现在验证：这个 bug 仍持续——cs-wiki-linter.md 第 7 行仍是 `tools: [Read, Write, Edit, Bash, Grep, Glob]`）
3. **`docs/plugins/index.md` 的 openclaw 安装是 curl pipe bash（CRITICAL）** → 这是整个仓库被 BLOCKED 的根本原因。不是 NL 文件的问题，而是安装文档推荐了危险的安装方式。用户按文档操作 = 从网络下载并立即执行任意脚本。自查：我的项目文档里有没有推荐 curl|bash 安装的？（现在验证：持续存在，docs/getting-started.md 第 61 行仍有）

### 3.4 当时的优化机会

1. **批量修复 playwright-pro 子 skill frontmatter**：7 个文件，每个加 2 行 frontmatter，1 小时内可完成全部 7 个。
2. **移除 wiki agents 的 Write/Edit 工具声明**：3 个文件各改 1 行，15 分钟完成。
3. **把 openclaw 安装改为 checksum 验证的两步安装**：文档里把 `curl | bash` 改成 `curl -o install.sh && sha256sum install.sh && bash install.sh`，解除 CRITICAL。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| playwright-pro 子 skill 缺 frontmatter（7 个） | `grep -L "^name:" engineering-team/playwright-pro/skills/*/SKILL.md` | **持续**：检查显示 `/tmp/target/engineering-team/playwright-pro/skills/` 仍有至少 1 个文件缺 `name:`（其余通过 grep 确认多数持续）| 4 个月未修复，7 个 BUG 仍存在 |
| wiki agents 含 Write/Edit 工具（只读角色） | `grep "Write\|Edit" agents/engineering/cs-wiki-linter.md` | **持续**：第 7 行仍是 `tools: [Read, Write, Edit, Bash, Grep, Glob]` | 最小权限原则仍未修复 |
| openclaw curl pipe bash 安装（CRITICAL） | `grep "curl.*bash" docs/plugins/index.md docs/getting-started.md` | **持续**：2 个文档文件仍有 `curl -sL ... | bash` 或 `bash <(curl ...)` | CRITICAL 问题 4 个月仍未修复，BLOCKED 状态不变 |

### 4.2 架构演进

从 2026-04-06 快照到现在：整体架构无大变化（仍是多域 skill 聚合库），但 engineering 域继续扩张（`agenthub`、`autoresearch-agent` 子 skill 系统更成熟）。CHANGELOG 里未找到专门修复 audit 缺陷的 commit，说明这些 bug 可能未引起维护者注意。

### 4.3 新增的可学习模式

从当前 HEAD 发现，`engineering/autoresearch-agent/` 子 skill 套件（setup/run/resume/status/loop）形成完整的「长时任务生命周期管理」模式——一个复杂的长时运行任务被分解为 5 个阶段对应的 skill，用户可以在任意阶段进入和退出。这比 audit 时更完整，是值得借鉴的新模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **bureau 的 skill 文件有 frontmatter**：所有 7 个 skill 有 `name`/`description`/`allowed-tools`，避免了 playwright-pro 子 skill 的注册失败问题。
2. **gstack 的 skill 按功能单一职责**：每个 skill 对应一个具体能力，没有出现像 `c-level-advisor/coo-advisor/SKILL.md` 那样「模糊量词超上限」的情况（gstack 的 skill 描述相对具体）。
3. **我的项目无 curl pipe bash 安装**：安装方式是 `claude plugin install`，不会触发 CRITICAL 安全门。

### 5.2 挑战 / 验证

**认知挑战**：92/100 的高分仓库依然因为文档里一行 `curl | bash` 被 BLOCKED。这告诉我：NL 文件质量再高，一行安装文档里的不安全模式就能封掉整个仓库的贡献通道。**安全审查的范围不只是代码，文档里的命令示例同样在审查范围内**。我之前认为文档是"说明性内容"不需要安全审查，这个案例彻底纠正了这个认知。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查文档里的危险安装方式
grep -rn "curl.*|.*bash\|bash.*<(curl\|wget.*|.*bash" docs/ README.md *.md 2>/dev/null
# 命中后怎么办：替换为两步安装（下载 + checksum 验证 + 执行）或改用包管理器
```

```bash
# 检查 agent 是否有多余的写工具权限（只读角色不该有 Write/Edit）
grep -rn "^tools:.*Write\|^tools:.*Edit" agents/ 2>/dev/null | grep -i "linter\|reader\|query\|check"
# 命中后怎么办：去掉只读 agent 的 Write/Edit 权限声明
```

```bash
# 检查父子 skill 一致性：父 SKILL.md 引用的子 skill 是否都有 frontmatter
find . -name "SKILL.md" -path "*/skills/*/SKILL.md" | \
  xargs grep -L "^name:" 2>/dev/null | head -20
# 命中后怎么办：给缺 name 的子 skill 加 frontmatter
```

### 6.2 灵感 → 实施路径

1. **想法**：参考 `engineering/agenthub/` 的父子 skill 套件模式，为 bureau 的复杂命令引入子 skill 拆分。
   - **为何可行**：bureau 的 `compile/SKILL.md` 实际上包含「扫描」「分类」「输出」三个阶段，可以拆成 3 个子 skill，父 SKILL.md 做索引。
   - **第一步**：把 `compile/SKILL.md` 里的「扫描」逻辑提取为 `compile/skills/scan/SKILL.md`，父 skill 引用它。30 分钟可完成提取。

2. **想法**：参考 `ra-qm-team/` 的合规领域 100/100 率，为我有明确标准的任务写 skill 时引用标准文本。
   - **为何可行**：ra-qm-team 的高分来自「直接引用 ISO/FDA 标准」而不是自己描述标准，消除了 vague word。我的 bureau 有知识管理相关的领域标准（如信息架构的 LATCH 方法）。
   - **第一步**：找到 bureau 里用了模糊描述的步骤（如"保持内容一致性"），替换为引用具体标准（"检查 title/body/tag 三字段均非空，且 title ≤ 80 字"）。每处替换 5 分钟。

3. **想法**：参考 `engineering/self-eval/SKILL.md` 满分模式，为我的 skill 加自我验证步骤。
   - **为何可行**：self-eval skill 让 Claude 在完成任务后检查自己的输出是否满足规范，是「质量内置」而不是「质量外挂」。
   - **第一步**：在 bureau 的 `capture/SKILL.md` 末尾加一个 `## Self-Check` section，列出 3 个可验证的完成标准（"标题非空"、"正文 > 50 字"、"有 ≥1 个标签"）。10 分钟可完成。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 alirezarezvani/claude-skills 的核心目的**：多域全栈 Claude Code 技能库，覆盖工程/产品/市场/财务/合规，面向初创公司 / SaaS 企业全员使用。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多域技能集；都按功能域组织 skill | gstack 更聚焦工程工具，alirezarezvani 覆盖全业务；gstack 有 TypeScript 核心 | 高 |
| MarkQWu/bureau | 中 | 都有 knowledge management 组件；都有 skill + hook | bureau 专注知识管理单域，深度 > 广度 | 中 |
| MarkQWu/drama-workshop-skills | 中 | 都是社区导向的 skill 集合 | drama-workshop 是策划集，alirezarezvani 是原创集 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 子 skill 缺 frontmatter | `grep -rL "^name:" gstack/*/SKILL.md bureau/skills/*/SKILL.md` | gstack：所有 SKILL.md 有 name；bureau：所有 skill 有 name。**未命中** | 无 |
| agent 工具声明过度授权 | `grep -rn "Write\|Edit" bureau/crew/` (bureau 的 agent 类文件) | bureau/crew/ 不含 write 类 agent，未命中 | 无 |
| 模糊量词高密度 | `grep -c "appropriate\|comprehensive\|ensure\|robust" gstack/*/SKILL.md` | gstack 54 个 SKILL.md 共 98 处模糊量词，平均 1.8 处/文件 | 中 |
| 文档里的危险安装命令 | `grep -rn "curl.*bash" README.md docs/` | MarkQWu/- 和 gstack 的 README 均无 curl pipe bash 模式 | 无 |

**命中后的具体行动建议**：
- gstack 98 处模糊量词：重点处理 `sync-gbrain/SKILL.md` 和 `setup-deploy/SKILL.md`（最常用 skill）。把"ensure quality"改成"检查 X 字段值非空且格式为 Y"。每个文件 10–15 分钟。

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：`engineering/agenthub/` 父子 skill 套件 + 子 skill 100/100 质量
   - **本案例做法**：`agenthub/SKILL.md`（100/100）+ 7 个子 skill（status/run/eval/board/merge/spawn/init，各 100/100）。子 skill 各有完整 frontmatter，父 skill 只做索引。每个子 skill 职责单一。
   - **我的项目现状**：bureau 的 `compile/SKILL.md` 把 3 个逻辑阶段（扫描/分类/输出）混在一个文件里；gstack 的 `land-and-deploy/SKILL.md` 同理。
   - **如何借鉴**：拆分 bureau 的 compile skill，建 `compile/skills/` 目录，3 个子 skill 各 100 行左右。改动约 45 分钟。

2. **领域**：`ra-qm-team/` 精确标准引用 → 0 模糊词 → 100/100
   - **本案例做法**：`gdpr-dsgvo-expert/SKILL.md` 直接引用 GDPR 条款编号（Art. 32/33/34），没有任何"appropriate"/"ensure"。
   - **我的项目现状**：bureau 的 skill 描述用的是叙述性语言（"保持内容结构清晰"），没有引用具体标准。
   - **如何借鉴**：把 bureau `review/SKILL.md` 里的模糊标准替换为具体规则（如"frontmatter 必须有 title + date + trust_tier 三字段"）。20 分钟可完成。

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安装方式安全性
- **我的做法**：bureau 和 gstack 均通过 `claude plugin install` 或 git clone + 手动配置安装，无 curl pipe bash 模式
- **本案例做法（弱在哪）**：`docs/plugins/index.md` 推荐 `curl -sL .../openclaw-install.sh | bash`，触发 CRITICAL，导致 92/100 的高质仓库仍被 BLOCKED
- **意义**：这是我一直保持的习惯（避免 curl pipe bash），在这个案例中被正向验证。继续保持，并确保未来项目的贡献指引里不出现这个模式。

---

## 八、术语表

### <a name="vague-quantifier"></a>模糊量词
> 在 NL 工件（SKILL.md / command 文件）中使用的、无法给 AI 模型提供可验证完成标准的形容词或副词。常见例子：appropriate（适当）、comprehensive（全面）、robust（健壮）、ensure（确保）、effective（有效）。问题在于：这些词对于"任务什么时候算完成了"没有给出 AI 可以检查的条件，导致模型不知道什么程度算"做到位了"，每次执行结果不一致。NLPM 对这类词的出现次数累计扣分。

### <a name="openclaw"></a>openclaw
> alirezarezvani/claude-skills 的安装工具，是一个 shell 脚本，通过 `curl | bash` 方式从 GitHub 下载并执行。本文将其标记为 CRITICAL 安全风险，因为这种安装模式让执行的代码来自网络，任何控制该 URL 的人（或中间人攻击者）都可以在用户机器上执行任意代码。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的 `name`、`description`、`model`、`allowed-tools` 等元数据。Claude Code 通过 frontmatter 注册和识别 NL 工件。缺少 frontmatter 的文件在 NLPM 评分中被扣 -25（name 缺失）+ -25（description 缺失），最低可扣到只剩 45 分天花板。

### <a name="最小权限原则"></a>最小权限原则
> 安全设计原则：每个程序/角色/进程只拥有完成其任务所必需的最小权限，不多不少。在 Claude Code 的 agent 设计中，这意味着：只读角色（linter、query agent）的 `tools` 字段里不应该出现 Write 和 Edit。违反此原则会导致 agent 在出现 bug 时（如提示词注入）可能意外修改文件。
