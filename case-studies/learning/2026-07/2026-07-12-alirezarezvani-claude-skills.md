# alirezarezvani/claude-skills — 学习案例

**仓库**：https://github.com/alirezarezvani/claude-skills
**Stars**：N/A | **来源**：upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-12（基于当前 HEAD）
**主题标签**：`security-gate`, `curl-pipe-bash-risk`, `vague-quantifier`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-skills 是一套按业务领域深度组织的 Claude Code 技能库：431 个 NL 工件，涵盖工程（engineering）、营销（marketing-skill）、产品（product-team）、C 级顾问（c-level-advisor）、法规质量（ra-qm-team）、商业增长（business-growth）、财务（finance）等七大领域。作者 Alireza Rezvani 以"覆盖一家中型 SaaS 公司所有岗位"为目标构建，在生态中的位置：最接近企业内训知识库的 Claude Code 插件集。

关键事实：
- 431 个 NL 工件，横跨 7 个业务领域，每个领域有独立目录
- `ra-qm-team/` 子系统 6/14 个 skill 满分 100，代表最高质量基准
- `engineering/autoresearch-agent` 和 `engineering/agenthub` 子系统均 100 分
- 存在 CRITICAL 安全漏洞（`curl -sL ... | bash`），本次 audit BLOCKED

### 1.2 架构剖析

```
claude-skills/
├── engineering/
│   ├── autoresearch-agent/        # 100分，含 sub-skills (setup/status/run/resume/loop)
│   │   └── evaluators/            # Python 评估脚本（HIGH安全风险）
│   ├── agenthub/                  # 100分，含 sub-skills (status/run/eval/board/merge/spawn/init)
│   ├── llm-wiki/                  # 100分
│   └── ... (30+ engineering skills)
├── engineering-team/
│   ├── playwright-pro/            # 含 7 个 sub-skill 缺 frontmatter（已部分修复）
│   │   ├── skills/ (testrail/coverage/generate/report/migrate/review/browserstack)
│   │   └── hooks/hooks.json       # 100分
│   └── self-improving-agent/
├── ra-qm-team/                    # 6/14满分，最高质量子系统
│   ├── gdpr-dsgvo-expert/         # 100分
│   ├── risk-management-specialist/ # 100分
│   └── ...
├── c-level-advisor/               # 15个 skill，executive 工具
├── marketing-skill/               # 20+ skill，多个 95分
├── product-team/                  # 10+ skill，含 spec-to-repo
├── agents/                        # cs-前缀 agent，分 engineering/product/c-level 等
│   └── engineering/
│       ├── cs-wiki-linter.md      # 工具权限 bug：Write/Edit 声明在只读 agent
│       └── cs-wiki-librarian.md
├── commands/                      # 根级命令，多个 100分（saas-health/okr/retro 等）
├── docs/plugins/index.md          # CRITICAL: curl-pipe-bash 安全漏洞
└── CLAUDE.md                      # 100分
```

- **文件类型分布**：约 50 个 agent / 30 个 command / 250+ 个 SKILL.md / 2 套 hooks
- **编排关系**：agents/ 下的 cs-* agent 通过 `skills:` 字段引用 engineering/、product-team/ 等领域技能；autoresearch-agent 和 agenthub 有完整的 sub-skill 树形编排
- **跨件契约**：`engineering/llm-wiki/` 定义底层知识库 skill，`agents/engineering/cs-wiki-*.md` 引用它；`ra-qm-team/` 各 skill 独立，无相互依赖

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「领域专精，岗位覆盖」——每个子目录模拟一个专业岗位或业务领域，skill 为该岗位赋能
- **解决什么问题**：让一个人用 Claude Code 调用"整个团队"的领域知识——从 CTO 顾问到 GDPR 合规专家
- **trade-off**：深度 vs 宽度。ra-qm-team 走极深路线（14 个精细 skill），c-level-advisor 走极宽路线（15 个高层顾问）；engineering/ 则中度深（每个 skill 独立，但 autoresearch 和 agenthub 有完整 sub-skill 体系）
- **认知模型**：作者把 Claude Code 看成"企业 AI 操作系统"，skill 是岗位插件，agents 是岗位协作编排

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：领域深度树（Domain Skill Forest）**

以业务领域为根节点，每个领域内再按功能细分 sub-skill，形成"领域/子功能/SKILL.md"的三层树形结构。不同领域之间通过 agent 的 `skills:` 引用字段跨树调用。

模式特征清单：
- **特征 1**：领域即目录——`engineering/`、`ra-qm-team/` 等顶级目录对应业务领域
- **特征 2**：sub-skill 树内自封装——autoresearch-agent 的 setup/status/run 各自是独立 SKILL.md，共享父级上下文
- **特征 3**：[frontmatter](#frontmatter) 协议一致——高质量子系统（ra-qm-team、engineering/agenthub）所有 skill 均有完整 name/description/allowed-tools
- **特征 4**：横向孤立，纵向联动——领域间互不依赖，领域内 sub-skill 紧密配合
- **特征 5**：安全成熟度两极分化——NL 工件质量高（92分），但文档层（docs/）存在 CRITICAL 安全漏洞

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业多岗位知识覆盖 | ✅ 高度适用 | 领域树可映射组织结构，岗位切换简单 |
| 深度专精单一技术领域 | ✅ 适用 | 可只维护一棵树（如 engineering/） |
| 快速试验单一技能 | ❌ 效率低 | 理解完整目录结构有成本，简单场景用单仓更快 |
| 需要跨领域实时协调 | ⚠️ 有限支持 | 需要 agent 用 `skills:` 显式引用，无动态路由 |
| 公开安装（供外人使用）| ❌ 目前 BLOCKED | docs/ 中的 curl-pipe-bash 漏洞须先修复 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域深度树（本仓库）| alirezarezvani/claude-skills | 覆盖面广，领域内质量高 | 目录复杂，安全漏洞难审查 |
| 单仓单领域 | lackeyjb/playwright-skill | 专注，安全面小 | 功能范围受限 |
| 平列式集市 | ananddtyagi/cc-marketplace | 贡献门槛低，发现性强 | 质量不一，无门禁 |
| NL + 二进制混合 | agent-sh/agentsys | 性能高，跨平台 | 复杂性高，维护成本大 |

### 2.4 改进空间
1. **当前问题**：docs/ 中的安装示例使用 [curl-pipe-bash](#curl-pipe-bash) 反模式，任何使用该文档安装的用户都在运行未验证的远程脚本。**改进做法**：将安装示例改为 `curl -sL url -o install.sh && sha256sum install.sh && bash install.sh`，或者提供 `pip install` / 发布到 npm 的固定版本安装路径。**预期收益**：消除 CRITICAL 安全评级，解除 BLOCKED 状态。
2. **当前问题**：cs-wiki-linter/librarian/ingestor 三个只读 agent 声明了 Write/Edit 工具。**改进做法**：将这三个 agent 的 `tools` 改为 `[Read, Grep, Glob, Bash]`，去掉 Write/Edit。**预期收益**：消除潜在的意外文件修改风险，符合最小权限原则。
3. **当前问题**：`engineering/autoresearch-agent/evaluators/` 的 Python 脚本使用 `subprocess(shell=True)` 并插值动态变量（HIGH 安全风险）。**改进做法**：改用 `subprocess(shell=False, args=[cmd_list])` 方式调用，消除 shell 注入向量。**预期收益**：5 个 HIGH 安全发现降为 LOW。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**，**安全状态：BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/product/cs-product-analyst.md | 70 | 缺输出格式，示例不足 |
| commands/competitive-matrix.md | 65 | 无 allowed-tools，无步骤编号，无示例 |
| engineering-team/playwright-pro/skills/testrail/SKILL.md | 85 | 缺 name+description frontmatter（BUG）|
| agents/engineering/cs-wiki-linter.md | 78 | Write/Edit 声明在只读 agent（BUG）|
| ra-qm-team/gdpr-dsgvo-expert/SKILL.md | 100 | 满分，最佳参考 |
| engineering/agenthub/skills/*/SKILL.md | 100×7 | 满分，sub-skill 模式典范 |

### 3.2 当时值得借鉴的模式

1. **ra-qm-team 满分子系统** → 证明高规格写作标准可持续  
   原文路径：`ra-qm-team/gdpr-dsgvo-expert/SKILL.md`、`ra-qm-team/risk-management-specialist/SKILL.md`（共 6 个 100 分）  
   借鉴方式：把 ra-qm-team 作为模板标杆，复制其 frontmatter 结构和示例格式到其他领域

2. **agenthub sub-skill 树** → 单一功能拆分为可独立使用的子 skill  
   原文路径：`engineering/agenthub/SKILL.md` + `skills/status/`、`run/`、`eval/`、`board/`、`merge/`、`spawn/`、`init/`（7 个 100 分子 skill）  
   借鉴方式：当一个 skill 功能复杂时，按操作类型拆分子 skill，每个子 skill 有独立的 frontmatter 和示例

3. **clean commands 满分集合** → 简洁命令做到 100 分的极简范本  
   原文路径：`commands/saas-health.md`、`commands/okr.md`、`commands/retro.md`（等 13 个 100 分命令）  
   借鉴方式：命令不需要复杂，做到有 `name`、`description`、`allowed-tools`、清晰步骤即可满分

4. **hooks 100 分设计** → 完整 hook 文件的正确写法  
   原文路径：`engineering-team/playwright-pro/hooks/hooks.json` 和 `self-improving-agent/hooks/hooks.json`（均 100 分）  
   借鉴方式：hooks 触发条件明确，不使用通配符，无节流问题

### 3.3 当时的缺陷

1. **docs/ 安装示例使用 curl-pipe-bash**  
   根本原因：作者把文档安装步骤当成"开发者懂得如何保护自己"的受众，没意识到这是 NLPM 安全扫描会标记为 CRITICAL 的反模式。`curl -sL url | bash` 的本质是"从互联网下载任意脚本并以当前用户权限执行"，如果 URL 被劫持，后果是任意代码执行。  
   自查：我的任何 docs/ 或 README 里有类似安装指令吗？

2. **playwright-pro 子 skill 集体缺 frontmatter**  
   根本原因：作者在父级 `playwright-pro/SKILL.md` 中写了完整 frontmatter（93 分），但在子目录新建 skill 时没有从父级复制模板，导致 7 个子 skill 都没有 `name` 和 `description`，注册失败。这是"一次性建好顶层再补子层"工作模式的典型副作用。  
   自查：我的 echo-sleuth/skills/ 下每个子 skill 都有 frontmatter 吗？

3. **wiki agent 工具权限越界**  
   根本原因：作者在声明 agent `tools` 时列了 `[Read, Write, Edit, Bash, Grep, Glob]`，可能是从其他 agent 复制的通用模板，没有根据"只读审计"的职责收敛权限。Write/Edit 赋予了读审计 agent 改文件的能力，违反最小权限原则。  
   自查：我的 agent 的 `tools` 列表是否严格对应该 agent 实际需要的权限？

### 3.4 当时的优化机会
1. **批量修复 playwright-pro 子 skill frontmatter**：7 个文件各自添加 `name` 和 `description`，是简单的单 PR 修复
2. **去掉 wiki agents 的 Write/Edit**：3 个文件各删一行，立竿见影
3. **修复 competitive-matrix.md**：补充 `allowed-tools`、输出格式说明、步骤编号

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| docs/plugins/index.md curl-pipe-bash | `grep -n "curl" docs/plugins/index.md` → 第 84 行 | **持续** — `curl -sL ... \| bash` 仍在第 84 行 | CRITICAL 安全问题 3 个月未修，仓库仍处于 BLOCKED 状态 |
| playwright-pro/skills/testrail/SKILL.md 缺 frontmatter | `head -3 engineering-team/playwright-pro/skills/testrail/SKILL.md` | **已修复** — 现在有 `name: "testrail"` 和 description | 说明 PR 贡献路径有效；bug 被接受并合并 |
| cs-wiki-linter.md Write/Edit 工具声明 | `grep "Write\|Edit" agents/engineering/cs-wiki-linter.md` | **持续** — tools: [Read, Write, Edit, Bash, Grep, Glob] 仍在 | 权限越界问题未修 |

### 4.2 架构演进
playwright-pro 子 skill frontmatter 的修复说明：贡献渠道已打通，社区或作者本人接受了这类修改。而 CRITICAL 安全问题（curl-pipe-bash）和工具权限越界长期未修，原因可能是：安全问题需要更主动的行动（作者没收到私下安全报告），wiki agents 工具越界影响不明显（功能可用但权限过大）。

### 4.3 新增的可学习模式
playwright-pro 修复后，`engineering-team/playwright-pro/` 子系统的 frontmatter 完整性显著提升，可作为"先修 bug 再提 PR"工作流程的正向案例。

---

## 五、校准

### 5.1 我已经在做对的
1. **无 curl-pipe-bash**：我的仓库 docs/ 和 README 中没有使用 `curl | bash` 安装模式
2. **echo-sleuth skill frontmatter 相对完整**：skills/ 下的文件大多有 name 和 description
3. **认识到工具权限最小化**：bureau 的 agent 不随意声明 Write/Edit
4. **commands/ 目录 CLAUDE.md 有完整规范**：类似 alirezarezvani 的 agents/CLAUDE.md（95 分）做法

### 5.2 挑战 / 验证
本案例挑战了"高质量 NL 工件 = 安全"的假设。alirezarezvani/claude-skills 的 NL 工件得了 92 分，是本批次最高分之一，但安全状态是 BLOCKED。安全漏洞藏在 docs/（文档）和 evaluators/（Python 脚本）里，不在 NL 工件本身。这说明**安全审查和 NL 质量审查是独立维度**——NL 满分仓库也可能有 CRITICAL 安全问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库 README 或 docs 里是否有 curl-pipe-bash 模式
grep -rn "curl.*|.*bash\|curl.*|.*sh" ~/my-repos/ 2>/dev/null
```
命中后：替换为更安全的安装方式（下载后手动验证再执行，或提供 pip/npm 包）。

```bash
# 检查 agent 的 tools 字段是否包含 Write 或 Edit
grep -rn "Write\|Edit" ~/.claude/agents/*.md 2>/dev/null | grep "tools:"
```
命中后：逐一确认该 agent 是否真的需要写文件；如果是只读/分析类 agent，去掉 Write/Edit。

```bash
# 检查 subprocess 调用是否用了 shell=True
grep -rn "shell=True" ~/my-repos/**/*.py 2>/dev/null
```
命中后：将 `subprocess.run("cmd string", shell=True)` 改为 `subprocess.run(["cmd", "arg1", "arg2"], shell=False)`。

```bash
# 检查我的 skill sub-tree 每个子 skill 是否有 frontmatter
for f in ~/.claude/skills/**/**/SKILL.md; do
  head -1 "$f" | grep -q "^---" || echo "MISSING frontmatter: $f"
done
```
命中后：在文件顶部加 `---\nname: <slug>\ndescription: <一句话>\n---` 块。

### 6.2 灵感 → 实施路径

1. **想法**：把 ra-qm-team 的 SKILL.md 结构作为写作模板，用于提升 echo-sleuth/skills/ 质量
   - **为何可行**：ra-qm-team 6 个 100 分 skill 有明显共性：精确的 description、清晰的 allowed-tools、具体的 examples；这是模板可以捕获的结构
   - **第一步**：选一个 ra-qm-team 的 100 分 SKILL.md，复制其骨架（name/description/allowed-tools/## Examples 结构），替换内容；约 15 分钟/skill

2. **想法**：将 agenthub sub-skill 模式用于 echo-sleuth 的 memory-management skill 重构
   - **为何可行**：memory-management 当前是单一大 SKILL.md，可以按操作类型（remember/recall/forget/summarize）拆成 4 个子 skill，每个独立可调用
   - **第一步**：在 `echo-sleuth/skills/memory-management/skills/` 下新建 `remember.md`，从现有 SKILL.md 提取对应段落，加 frontmatter；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新）

### 8.1 目的对齐度

- **本案例 alirezarezvani/claude-skills 的核心目的**：为 SaaS 企业各业务领域（工程/营销/产品/C-suite）提供深度技能覆盖
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都有多个领域 skill（经验/记忆/归档），有 agents 编排 | echo-sleuth 是单应用（对话挖掘），alirezarezvani 覆盖企业全栈 | 高 |
| MarkQWu/bureau | 中 | 都有 skill + command + agent 完整三层 | bureau 是具体工具（知识库），alirezarezvani 是通用能力集 | 中 |
| MarkQWu/graphify | 低 | 都有多个 SKILL.md | graphify 专注图谱构建，领域非常窄 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| wiki agent 工具权限越界（Write/Edit on 只读角色）| `grep "tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md` | **部分命中**：echo-sleuth 的 recall.md agent 声明了 Write 工具但职责是查询/召回，可能越权 | 中 |
| sub-skill 缺 frontmatter | `head -1 /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | **轻微命中**：skills/ 下部分 SKILL.md frontmatter 完整，但缺 examples | 低 |
| commands 缺 allowed-tools | `grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/` | **命中**：echo-sleuth commands/ 下的命令缺 allowed-tools 声明 | 高 |

**命中后的具体行动建议**：
- `echo-sleuth/agents/recall.md` → 检查职责定义，如果是只查询（不写文件），从 tools 里去掉 Write；5 分钟
- `echo-sleuth/commands/*.md` → 逐一添加 `allowed-tools: [Read, Bash, Grep]`（根据命令实际需要）；20 分钟

### 8.3 别人的更优方案

1. **领域**：sub-skill 树的系统化设计
   - **本案例做法**：`engineering/agenthub/` 把功能拆成 7 个独立子 skill（status/run/eval/board/merge/spawn/init），每个都是独立可调用的 100 分工件，粒度极细
   - **我的项目现状**：echo-sleuth 的 memory-management 是单一大 SKILL.md，包含多种操作，没有拆分
   - **如何借鉴**：把 memory-management 按操作类型拆分为 `remember/SKILL.md`、`recall/SKILL.md`、`synthesize/SKILL.md`，复用父级 context，约 1 小时

2. **领域**：领域顶层 SKILL.md 作为导航层
   - **本案例做法**：`ra-qm-team/SKILL.md`（95 分）作为领域入口，描述整个 ra-qm-team 子系统的职责边界和调用方式；用户先读这个再选子 skill
   - **我的项目现状**：echo-sleuth/skills/ 没有顶层 SKILL.md 描述整体功能边界
   - **如何借鉴**：新建 `echo-sleuth/skills/SKILL.md`，描述"这套 skill 系统做什么、包含哪些子 skill、典型调用场景"；约 15 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全实践
- **我的做法**：echo-sleuth 和 bureau 的安装文档不使用 `curl | bash`，安装路径是 `claude plugin install`
- **本案例做法（弱在哪）**：docs/plugins/index.md 第 84 行至今仍有 `curl -sL ... | bash` 安装指令，是 CRITICAL 安全缺陷，BLOCKED 状态 3 个月未解除
- **意义**：我的仓库在安全实践上处于正确轨道；这是一个不能忽视的基线，也是我在给别人推荐安装方式时应坚持的标准

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（name、description、model、tools 等）。Claude Code 加载 SKILL.md 或 agent 时先解析 frontmatter，没有 frontmatter 的文件无法被注册，即使内容再好也不会被调用。

### <a name="curl-pipe-bash"></a>curl-pipe-bash
> 一种危险的命令行安装模式：`curl -sL <url> | bash`。含义是"从互联网下载某个脚本，立刻以当前用户权限执行"，完全不验证脚本内容。如果 URL 被劫持（DNS 污染、CDN 缓存投毒），执行的就是攻击者的恶意代码。安全替代方案：下载后校验哈希值，或使用包管理器（npm/pip）的固定版本安装。

### <a name="subprocess-shell-true"></a>subprocess shell=True
> Python 里 `subprocess.run("cmd", shell=True)` 的含义是"把整个字符串交给系统 shell 解释执行"。如果 `cmd` 字符串里包含用户输入或外部配置，攻击者可以注入 `;rm -rf /` 这样的命令（shell 注入）。安全写法：`subprocess.run(["cmd", "arg1", "arg2"], shell=False)`——用列表传参，shell 不介入，注入向量消失。

### <a name="最小权限原则"></a>最小权限原则
> 系统安全的基本原则：一个组件只应拥有完成其职责所需的最小权限，不多一点。对 Claude Code agent 而言，一个"只读审计"角色只需要 Read/Grep/Glob，声明 Write/Edit 就是违反最小权限原则——不是说一定会被滥用，而是增加了意外写入的风险面。

### <a name="BLOCKED状态"></a>BLOCKED 状态
> NLPM 安全扫描的最高风险结论。含义：该仓库存在 Critical 或 High 级别的安全发现，在安全问题被私下披露并修复之前，不应向该仓库提交任何公开 PR（因为 PR 本身可能暴露更多信息）。BLOCKED 不等于"代码不好"，也不代表 NL 质量低——alirezarezvani/claude-skills NL 质量 92 分，但安全是 BLOCKED。
