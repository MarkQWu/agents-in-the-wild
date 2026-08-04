# gadievron/raptor — 学习案例

**仓库**：https://github.com/gadievron/raptor
**Stars**：N/A | **来源**：upstream audit
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-08-04（基于当前 HEAD）
**主题标签**：`security-gate`, `cross-reference`, `examples-driven`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

gadievron/raptor（RAPTOR = Recursive Automated Penetration Testing and Offensive Research）是一个围绕**二进制安全研究**构建的 Claude Code 插件，覆盖漏洞分析、崩溃调查、开源软件取证、可利用性验证等专业安全工作流。Audit 时共 50 个[NL 工件](#NL工件)（16 agents + 21 commands + 13 skills/config），当前 HEAD 已扩展至约 65 个（17 agents + 35 commands + 13 SKILL.md），是本批次案例中唯一以**进攻性安全**为核心领域的插件。

关键事实：
- 面向漏洞研究人员和渗透测试工程师，具备 CodeQL、崩溃分析、fuzzing、可利用性验证等深度工作流
- NLPM 得分 93/100，**SECURITY BLOCKED**（1 CRITICAL — Dockerfile [curl|bash](#curl-pipe-bash)）
- 开发期间活跃：commands 从 21 增长到 35，新增 frida、exploit-dev、audit、exploitation 等技术领域
- 最新 commit 为「Delete commands directory」后重建，说明作者经历了一次命令体系的结构性重组
- 这是一个专家工具，而非通用插件——设计假设用户已具备安全领域知识

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
gadievron/raptor/
├── .claude/
│   ├── agents/             (17 个专域 agent)
│   ├── commands/           (35 条工作流入口)
│   ├── skills/             (13 个 SKILL.md + 2 个 Markdown 参考)
│   │   ├── crash-analysis/ (4 个子技能)
│   │   ├── oss-forensics/  (5 个子技能)
│   │   ├── SecOpsAgentKit/ (空目录，占位用)
│   │   ├── exploit-dev/    (新增)
│   │   ├── exploitation/   (新增)
│   │   ├── frida/          (新增 SKILL.md)
│   │   ├── audit/          (新增 SKILL.md)
│   │   ├── exploitability-validation/
│   │   └── code-understanding/
│   └── settings.json
├── plugins/
│   └── coverage/hooks/hooks.json   (PostToolUse hook)
├── .devcontainer/
│   └── Dockerfile           (含安全争议的 nodesource 安装脚本)
├── packages/                (RAPTOR Python 包)
├── libexec/                 (编译型辅助二进制)
└── requirements.txt
```

**文件类型分布**：
- 17 个 agent（専域——崩溃分析、oss 取证、可利用性验证、进攻专家）
- 35 条 command（工作流入口，audit 时 21 条）
- 13 个 SKILL.md（技能参考库）
- 1 个 hook 配置（Coverage 数据自动触发）

**编排关系**：commands 是主要入口（用户调用 `/crash-analysis`、`/scan`、`/raptor`），commands 分发给 agents，agents 载入对应的 skills。例如 `/crash-analysis` command → `crash-analysis-agent` 编排 → 子 agents（`coverage-analysis-generator-agent`、`function-trace-generator-agent`、`crash-analyzer-agent`、`crash-analyzer-checker-agent`）并行执行 → 汇总结果。

**跨件契约**：agent 的 `tools:` [frontmatter](#frontmatter) 声明其可用工具，skill 路径通过硬编码相对路径引用（`.claude/skills/{domain}/{skill}/SKILL.md`）。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「深度专域 + 可复用技能树」——每个 agent 只覆盖一个安全子域，通用方法论沉淀为 SKILL.md，agent 载入 skill 后具备「懂方法、有工具、会操作」的完整能力。
- **解决什么问题**：安全研究中的工作流太碎——CodeQL 分析、崩溃重现、可利用性判断、取证报告各是独立步骤，缺乏协调。RAPTOR 将这些步骤串联为可重复运行的 Claude Code 工作流。
- **做了什么 trade-off**：用「多 agent 分工」换取「精深领域知识」——每个 agent prompt 只聚焦一件事，但对用户来说学习成本高，35 条 commands 需要知道用哪一条。
- **反映什么认知模型**：作者视 AI agent 为「配备正确工具和参考手册的安全分析师」——skills 就是分析师的工具手册，tools frontmatter 声明「能用什么设备」，agent body 描述「怎么用这些设备完成任务」。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「安全领域专家网格（Domain-Expert Agent Mesh）」**

核心特征：将一个大领域（安全研究）拆解为若干专业子域，每个子域由一个专域 agent 负责，agent 通过声明式 skill 路径获取该子域的方法论知识，commands 作为「调度层」汇聚多个专域 agent。

模式特征清单：
- **特征 1**：专域 agent 粒度对齐「单一技术能力单元」（coverage 生成 / 函数追踪 / 崩溃分析 / 取证报告各一个 agent）
- **特征 2**：技能树分层——通用 skill（`code-understanding/`）+ 专域 skill（`crash-analysis/gcov-coverage/`）独立维护
- **特征 3**：可利用性验证流水线是「有门控的串行执行」（gated execution）：skill 显式规定必须先完成前置步骤才能输出结论
- **特征 4**：commands 做薄包装，不含业务逻辑，只负责路由到正确的 agent
- **特征 5**：hooks 做被动观测（PostToolUse 触发 coverage 数据聚合），不干预主流程

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 深度技术领域工具（安全/医疗/法律分析） | ✅ 高度适用 | 专域知识密度高，单一大 agent 扛不住，需要细粒度分工 |
| 多步骤技术工作流（崩溃分析 → 可利用性 → 报告） | ✅ 适用 | 串行分工减少单个 agent 的上下文长度，每个子步骤都有可验证的输出 |
| 面向普通用户的生产力工具 | ❌ 不适用 | 35 条 commands 对非专家来说噪音过大，缺少统一入口导引 |
| 纯数据处理脚本（无需技术决策） | ❌ 不适用 | 用脚本就够，agent 网格是过度工程 |
| 团队共用知识库 | ⚠️ 慎用 | 技能树好维护，但 agent 粒度需要团队对子域划分达成共识 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 专域 agent 网格（本案例） | gadievron/raptor | 每个 agent 专注一件事；skill 可跨 agent 复用 | 命令入口多；agent 命名需要严格维护以防 CLAUDE.md 失同步 |
| 单 agent 大 prompt | 大多数简单工具 | 用户只需学一个入口；上手快 | prompt 过长、领域知识稀释；无法组合子任务 |
| 路由 agent + 专域 agent（Router + Channels） | jnuyens/gsd-plugin | 用户只需记一个路由命令；路由 agent 做意图分发 | 多一层间接引用；路由 agent 本身需要维护所有子域的入口元数据 |

### 2.4 改进空间

1. **当前问题**：35 条 commands 没有统一的「总目录」agent，用户需要知道调用哪个命令。**改进做法**：增加一个 `raptor-help` agent，读取所有 command 的 `description` 字段，按安全工作流阶段（侦察 → 分析 → 验证 → 报告）归类呈现。**预期收益**：新用户 onboarding 成本降低 50%，命令可发现性大幅提升。

2. **当前问题**：`SecOpsAgentKit/` 目录是空占位符，`offsec-specialist.md` 声称能从其中读取 skills，实际运行时静默失败。**改进做法**：要么填充 `SecOpsAgentKit/skills/offsec/` 下的实际 SKILL.md，要么删除空目录 + 更新 agent 引用到已有 skills 路径（`exploit-dev/`、`exploitation/`）。**预期收益**：消除 BUG-1，进攻测试工作流恢复可用。

3. **当前问题**：9 个 OSS 取证子 agents 没有调用示例，编排 agent 如何拼装它们的输出对协作者不透明。**改进做法**：在 `oss-forensics/orchestration/SKILL.md` 中补充完整的「主编排 agent 提示词 → 子 agent 调用序列 → 最终报告结构」示例。**预期收益**：对照 exploitability validation pipeline 的标杆质量，提分可达 93→97。

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 **93/100**（agents 平均 89.6 / commands 平均 91.2 / skills+config 平均 100.0）。

| 组件类型 | 代表文件 | 当时分数 | 主要问题 |
|---|---|---|---|
| Agent（零示例） | oss-hypothesis-checker-agent.md | 83 | 无调用示例（-15）、模糊量词「appropriate」（-2） |
| Agent（零示例） | oss-report-generator-agent.md 等 8 个 | 85 | 无调用示例（-15） |
| Agent（单示例） | coverage-analysis-generator-agent.md | 95 | 仅 1 个示例（-5）、缺 tools 字段 |
| Command（含空输入） | codeql.md、diagram.md 等 8 个 | 85 | 缺 allowed-tools（-5）、无空输入处理（-10） |
| Command（基础） | 其余 13 条 commands | 95 | 仅缺 allowed-tools（-5） |
| Skill / Config | 所有 SKILL.md + hooks.json | 100 | 无问题 |
| Agent（模范） | exploitability-validator-agent.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **可利用性验证门控流水线**：`exploitability-validation/SKILL.md` 规定「必须先完成 crash reproduction，才能进行可利用性判断」——这是「[门控执行](#门控执行)」模式的典型实现。原文路径：`.claude/skills/exploitability-validation/SKILL.md`。借鉴：任何需要前置条件的 AI 任务都应在 skill 中显式声明门控，而非靠 agent 自律。

2. **崩溃分析四层技能树**：`crash-analysis/` 下有 `function-tracing/`、`gcov-coverage/`、`line-execution-checker/`、`rr-debugger/` 四个平行子技能，粒度与安全工具的功能单元对齐。借鉴：技能树的划分粒度应与「一个人类专家做的一个操作」对齐，而非与「一个功能模块」对齐。

3. **exploitability-validator-agent 的双示例标准**：该 agent 有 2 个完整工作示例（包含调用 agent 的上下文、中间输出、最终结论），是 NLPM 满分 100 的原因。借鉴：示例至少 2 个，分别覆盖「成功路径」和「边界/失败路径」。

4. **OSS 取证 orchestration SKILL**：`oss-forensics/orchestration/SKILL.md` 独立维护编排规则，子 agents 各自实现，解耦编排逻辑与执行逻辑。借鉴：当工作流涉及多个 agent 协作时，把「谁先谁后、如何传递结果」写进专门的 orchestration skill，而不是分散在各子 agent 的 body 里。

5. **hooks 的被动观测设计**：`plugins/coverage/hooks/hooks.json` 在每次 Read 工具调用后异步触发 coverage 数据聚合，不干预主流程。借鉴：hooks 适合「数据收集 / 副作用」，不适合做「前置验证」（会阻塞 agent）。

### 3.3 当时的缺陷

1. **BUG-1：offsec-specialist 引用不存在的技能路径**：`offsec-specialist.md` Phase 1 明确让 agent 去 `.claude/skills/SecOpsAgentKit/skills/offsec/` 加载技能，但该路径在仓库中完全不存在。根本原因：agent 是从旧版本架构拷贝而来的模板，技能迁移到新目录结构后没有同步更新 agent 中的路径引用。教训：跨件路径引用必须纳入 CI 或 `/nlpm:check` 验证，不能靠人工同步。**我有没有犯同样的错？** 有风险——bureau 的 agent 引用了 `skills/` 目录下的技能，一旦重命名目录就会失同步。

2. **BUG-2/3：CLAUDE.md 文档与实际 agent 文件名不一致**：CLAUDE.md 中写的 `oss-forensics-agent`、`oss-investigator-gh-api-agent`、`oss-investigator-gh-recovery-agent`，对应的实际文件分别是「不存在 / `oss-investigator-github-agent.md` / `oss-investigator-wayback-agent.md`」。根本原因：agent 被重命名时，CLAUDE.md 没有同步更新。新贡献者按文档找不到文件，任何基于名称索引 agents 的工具都会出错。**我有没有犯？** 这是「文档漂移」经典场景——bureau 的 CLAUDE.md 里如果有 agent 列表，同样面临此风险。

3. **BUG-4：coverage/trace agents 缺 `tools:` 声明**：`coverage-analysis-generator-agent.md` 和 `function-trace-generator-agent.md` frontmatter 中都没有 `tools:` 字段，但两者需要用 Bash 执行 gcc 编译、gcov、gdb 等命令。根本原因：tools 字段被视为「可选」而省略，但在某些 Claude Code 上下文中 Bash 不是默认工具。**我有没有犯？** bureau 的 auditor agent 正确声明了 `tools: Read, Grep, Glob`——这个案例验证了做对了。

4. **Q 质量：所有 21 条 commands 缺 `allowed-tools` 字段**：每条 -5 分，合计 -105 分（但 scoring 有 floor，实际体现约 -7 分到最终 93 分）。根本原因：commands 的 frontmatter 约定比 agents 更容易被忽略——很多开发者以为 commands 只是「描述步骤的文档」，而非「需要声明工具的工件」。**我有没有犯？** bureau 的 11 条 commands 全部缺 `allowed-tools`——完全复现了这个问题。

### 3.4 当时的优化机会

1. **为 9 个 OSS 取证子 agents 补充调用示例**（当时 -15 分/个）：示例应从主编排 agent 视角展示如何调用这些子 agents，并给出典型返回格式。
2. **修复 8 条 commands 的空输入处理**（当时 -10 分/条）：`codeql.md` 在没有 `--repo` 参数时应提示用户，而非让 Python 命令报错。
3. **offsec-specialist 的模糊量词清理**（当时 -4 分）：「appropriate authorization」→ 「prior written authorization from the system owner」；「relevant skills」→ 「skills listed under `.claude/skills/SecOpsAgentKit/skills/offsec/`」。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-1：`offsec-specialist` 引用 `SecOpsAgentKit/skills/offsec/`（路径不存在） | `ls .claude/skills/SecOpsAgentKit/`；`grep -n SecOpsAgentKit .claude/agents/offsec-specialist.md` | **仍存在（变体）**：`SecOpsAgentKit/` 目录已创建但为**空占位符**（无任何文件）；agent 仍引用 `SecOpsAgentKit/skills/offsec/` 这个不存在的子路径 | 作者意识到问题并开始填充，但目录内容尚未提交。进攻测试工作流 Phase 1 仍静默失败。 |
| BUG-2/3：CLAUDE.md 内 `oss-forensics-agent`、`oss-investigator-gh-api-agent` 等名称错误 | `grep -n "oss-forensics-agent\|oss-investigator-gh-api" CLAUDE.md` | **仍存在**：第 192 行仍为 `oss-forensics-agent - Main orchestrator`；`oss-investigator-gh-api-agent` 等错误名称未修复 | 文档漂移问题持续。贡献者查 CLAUDE.md 导航仍会找不到文件。 |
| BUG-4：`coverage-analysis-generator-agent.md` 和 `function-trace-generator-agent.md` 缺 `tools:` 字段 | `head -8 .claude/agents/coverage-analysis-generator-agent.md`；frontmatter 只有 `name` / `description` / `model` | **仍存在**：两个 agent 的 frontmatter 均无 `tools:` 行 | 在严格工具声明的上下文中，这两个 agent 仍可能无法使用 Bash 执行编译命令。 |
| SECURITY CRITICAL：Dockerfile 中 `curl -fsSL ... \| bash -` | `grep -n "curl.*bash\|nodesource" .devcontainer/Dockerfile` | **部分修复**：从 `curl ... \| bash -` 改为 `curl ... -o /tmp/nodesource_setup.sh && bash /tmp/nodesource_setup.sh`（捕获 + 再执行）；同时升级 Node.js 版本从 20.x → 24.x | curl 退出码现在可被检测（若 curl 失败，`&&` 阻止 bash 执行），但下载脚本本身**仍无 checksum 或签名验证**，供应链风险依然存在。 |

### 4.2 架构演进

从 audit（50 工件）到当前 HEAD（~65 工件）的变化：

| 维度 | Audit 时（2026-04-13） | 当前 HEAD | 含义 |
|---|---|---|---|
| Commands | 21 条 | 35 条（+14） | 大规模扩展：新增 `frida`、`binary`、`cve-diff`、`threat-model`、`scorecard`、`sage`、`audit`、`review` 等，覆盖更多安全工作流场景 |
| Agents | 16 个 | 17 个（+1） | 增量新增，结构基本稳定 |
| Skills（SKILL.md） | 10 个 | 13 个（+3）+ 2 个 Markdown 参考 | 新增 `frida/SKILL.md`、`audit/SKILL.md`、`exploit-dev/`、`exploitation/`、`SecOpsAgentKit/`（空） |
| Dockerfile Node.js 版本 | setup_20.x | setup_24.x | 追踪 Node.js LTS |
| SecOpsAgentKit | 不存在 | 空目录 | 作者开始构建进攻技能树，尚未填充 |

最重要的演进：commands 的爆炸式增长（21 → 35）说明作者正在快速扩展支持的安全工作流类型，工具已经从「崩溃分析 + 取证」拓展到「威胁建模 + SCA + Frida 动态分析 + CVE 差异分析」。

### 4.3 新增的可学习模式

1. **`frida/SKILL.md`**：为 Frida 动态插桩工具建立独立技能，统一了 `function-trace-generator-agent` 在运行时动态追踪场景下的操作规范。这是「工具绑定 skill」模式的延伸——每个外部工具（gcov / Frida / rr）对应一个 SKILL.md。

2. **`exploit-dev/` + `exploitation/` 目录**：区分「开发阶段」（写 exploit 原型）和「执行阶段」（验证 exploit 有效性），与 exploitability validation 流水线的分阶段思路一致，颗粒度比 audit 时更细。

---

## 五、校准

### 5.1 我已经在做对的

1. **agent 声明 `tools:` 字段**：bureau 的 `auditor` agent 正确声明了 `tools: Read, Grep, Glob`，避免了 BUG-4 的问题。本案例验证这是必要且有价值的做法。
2. **知识与执行分离**：bureau 的 skills 目录存储知识和规范，agents 负责执行——和 raptor 的 `skills/` + `agents/` 分层一致。
3. **单职责 agent 设计**：bureau 的 `auditor` agent 只做「知识库一致性审查」，不兼职执行其他任务，与 raptor 专域 agent 网格的理念相同。

### 5.2 挑战 / 验证

**挑战**：本案例的 BUG-2/3 让我意识到「CLAUDE.md 中的 agent 列表」是一个高频漂移点——作者在重命名 agent 文件时，几乎必然忘记同步 CLAUDE.md。我之前认为 CLAUDE.md 只是「给人读的文档」，不那么重要；但这个案例表明它会影响贡献者和工具的 agent 发现，应当和代码一样严格维护。

**验证**：本案例中 `exploitability-validator-agent.md` 因为有 2 个完整示例而得到 100 分，而 9 个没有示例的 agents 只有 85 分——差距 15 分。这验证了「示例数量直接影响 NL 工件质量」这一规律，和 jnuyens/gsd-plugin 案例结论一致。示例不是装饰，是分数关键项。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否有 tools: 字段
grep -rL "^tools:" /tmp/my-repos/MarkQWu-bureau/crew/*/agent.md 2>/dev/null

# 检查我的 commands 是否有 allowed-tools（预期会命中大量）
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null
```
命中后怎么办：在 frontmatter 里加 `allowed-tools: Bash, Read, Write`（按实际需要的工具列举），每条 command 和每个 agent 都要有。

```bash
# 检查我的 CLAUDE.md 里的 agent 列表是否和实际文件名一致
diff <(grep -oP 'oss-\w+' /tmp/my-repos/MarkQWu-bureau/CLAUDE.md | sort) \
     <(ls /tmp/my-repos/MarkQWu-bureau/crew/*/agent.md | xargs -I{} basename {} .md | sort)
```
命中后怎么办：手动更新 CLAUDE.md 中的 agent 名称列表，或将列表改为「参见 crew/ 目录」的动态描述，避免硬编码名称。

```bash
# 检查我的 skills 路径引用是否在文件系统中真实存在
grep -rn "skills/" /tmp/my-repos/MarkQWu-bureau/crew/*/agent.md 2>/dev/null | \
  awk -F': ' '{print $2}' | grep -v "^$"
# 手动核验输出的路径是否都存在
```
命中后怎么办：对每个路径引用执行 `ls` 验证，若路径不存在则修复 agent 引用或补充缺失文件。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 raptor 的「技能树 + 门控执行」，为 bureau 的 `review` 工作流增加「必须先 lint 才能 review」的门控规则。
   - **为何可行**：bureau 的 `review` 和 `lint` 已经是两条独立 commands，在 `skills/review/SKILL.md` 里加一行「先决条件：`bureau:lint` 通过」即可，无需改代码。
   - **第一步**：编辑 `/tmp/my-repos/MarkQWu-bureau/skills/review/SKILL.md`，在最顶部加「## Prerequisites」段，10 分钟可完成。

2. **想法**：借鉴 raptor 的 exploitability validation pipeline（2 个完整示例），为 bureau 的 `auditor` agent 补充第 2 个示例（当前只有 1 个）。
   - **为何可行**：auditor agent 已有 1 个示例，再加一个「canon 有矛盾时的详细报告示例」能直接提分，且本案例数据证明 2 个示例 = 满分。
   - **第一步**：打开 `/tmp/my-repos/MarkQWu-bureau/crew/auditor/agent.md`，在 `<example>` 后追加第二个 `<example>` 块，20 分钟可完成。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 gadievron/raptor 的核心目的**：将安全研究工作流（崩溃分析、取证、漏洞验证）包装为可重复执行的 Claude Code 工作流。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是「多 skills + 少数 agents」的垂直领域工具 | raptor 面向安全研究（外部目标），bureau 面向知识管理（内部数据）；raptor 有大量命令入口，bureau 更集中 | 高（架构模式直接可借鉴） |
| MarkQWu/gstack | 低 | 都用 Claude Code 插件形式 | gstack 无复杂 agent 编排 | 低 |
| MarkQWu/graphify | 无 | — | 数据可视化方向，与安全工具无重叠 | 低 |

### 7.2 在我的项目里复现的同类问题

对 §3.3 缺陷逐条核查 bureau：

| 本案例缺陷 | 检查方法 | 我的 bureau 命中情况 | 严重度 |
|---|---|---|---|
| BUG-4：agents 缺 `tools:` 字段 | `grep -L "^tools:" crew/*/agent.md` | ✅ **未命中**：bureau 的 `auditor` agent 有 `tools: Read, Grep, Glob` | 无 |
| Q：所有 commands 缺 `allowed-tools` | `grep -rL "allowed-tools" commands/*.md` | ❌ **命中：11/11 条 commands 全部缺失** | 高 |
| BUG-2/3：CLAUDE.md agent 名称漂移 | `grep "agent" CLAUDE.md` 对比 `ls crew/*/agent.md` | ⚠️ **部分风险**：CLAUDE.md 有 agent 引用，需人工比对 | 中 |
| BUG-1：skills 路径引用失效 | `grep -n "skills/" crew/auditor/agent.md` | ✅ **未命中**：bureau auditor 不引用外部 skill 文件路径 | 无 |

**命中后的具体行动建议**：
- `bureau/commands/lint.md` → 在 frontmatter 加 `allowed-tools: Bash, Read, Write`，5 分钟可完成
- 对所有 11 条 commands 批量检查后统一补充 `allowed-tools`，合计 30-45 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：可利用性验证的门控执行规范
   - **本案例做法**：`exploitability-validation/SKILL.md` 明确写「必须先完成崩溃重现（crash reproduction），才能继续判断可利用性」，并列出每个前置步骤的验证标准（`.claude/skills/exploitability-validation/SKILL.md`）
   - **我的 bureau 现状**：`skills/review/SKILL.md` 存在，但没有「前置条件」声明，review 可以在 lint 未通过时直接执行
   - **如何借鉴**：在 `skills/review/SKILL.md` 的 `## Prerequisites` 段声明「bureau:lint 得分 ≥ 80 是 bureau:review 的前置条件」；若无该段，则新增一段

2. **领域**：技能树子目录按工具/阶段组织
   - **本案例做法**：`crash-analysis/` 下有 4 个子 SKILL.md，每个对应一个具体工具（gcov、rr、function-tracing、line-execution-checker）
   - **我的 bureau 现状**：`skills/` 下有 7 个 SKILL.md 平铺，没有子目录组织
   - **如何借鉴**：当 skills 超过 10 个时，按功能域分子目录（如 `skills/ingestion/`、`skills/recall/`、`skills/output/`）；bureau 目前 7 个还不急，但随增长应规划好命名空间

### 7.4 反向：我的项目做得比他们好的地方

1. **领域**：agent 的 `tools:` 字段声明
   - **我的 bureau 做法**：`crew/auditor/agent.md` 的 frontmatter 中有 `tools: Read, Grep, Glob`，精确声明了只读工具集（`crew/auditor/agent.md`）
   - **本案例弱在哪**：`coverage-analysis-generator-agent.md` 和 `function-trace-generator-agent.md` 没有 `tools:` 字段，BUG-4 持续存在
   - **意义**：这是一个小但重要的做法——声明工具集不仅影响运行时行为，也是向未来维护者传递「这个 agent 的权限边界」的方式；bureau 在这里做对了

---

## 八、术语表

### <a name="NL工件"></a>NL 工件
> Natural Language Artifact（自然语言工件）的简称。在 Claude Code 生态里，指用 Markdown 写成的、Claude 可直接加载和执行的配置文件，包括 SKILL.md（技能定义）、agent.md（智能体定义）、command.md（命令定义）。与传统的「代码文件」（`.py`、`.ts`）相对，这些文件用自然语言而非编程语言描述行为逻辑。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`tools` 等）。Claude Code 读 SKILL.md 或 agent.md 时先解析 frontmatter 才能知道这个工件的名称、权限范围、使用哪个模型。缺少 frontmatter 的工件可能无法被正确注册。

### <a name="curl-pipe-bash"></a>curl|bash（curl pipe bash）
> 一种常见但有安全风险的安装方式：`curl -fsSL https://example.com/install.sh | bash`，即「下载远程脚本后直接用 bash 执行」。风险在于：如果远程服务器被攻击者控制，或 TLS 证书被伪造，执行的就是攻击者代码；且 curl 失败时 bash 可能会执行一个空命令，不会报错。更安全的方式是先下载（`-o file.sh`），人工检查或验证 checksum 后再执行。本案例中 raptor 的 Dockerfile 从 `curl ... | bash` 改为 `curl -o /tmp/setup.sh && bash /tmp/setup.sh` 是一个改进（curl 失败可被检测），但仍然缺少 checksum 验证。

### <a name="门控执行"></a>门控执行（Gated Execution）
> 在 AI agent 工作流中，「门控执行」是指「步骤 B 只有在步骤 A 成功完成且输出满足特定条件后才能执行」的设计模式。与简单的顺序执行不同，门控执行需要在 SKILL.md 或 agent body 中**显式声明前置条件**，防止 agent 在前置步骤未完成的情况下跳过到后续步骤并给出没有依据的结论。例如 raptor 的可利用性验证要求「必须先有崩溃重现的截图才能判断是否可利用」。

### <a name="供应链风险"></a>供应链风险（Supply Chain Risk）
> 软件依赖链条上游被污染的安全风险。在本案例的 Dockerfile 场景中，`deb.nodesource.com` 是一个第三方服务器；如果 NodeSource 的 CDN 被黑、域名被劫持、或 TLS 证书被伪造，每一个运行 `docker build` 的开发者都会执行攻击者代码。「无 checksum 验证」使得这种篡改无法被检测。这类风险在 2020 年代的开源软件供应链攻击事件（如 SolarWinds、xz-utils）中被大量利用。

### <a name="静默失败"></a>静默失败（Silent Failure）
> 程序或 agent 在出现错误时**不抛出错误、不输出警告，直接以「无结果」或「降级行为」继续运行**的现象。本案例中 `offsec-specialist.md` Phase 1 尝试列出一个不存在的目录，agent 会得到「目录不存在」的错误，但如果 agent 被训练为「遇到错误继续下一步」，它可能会跳过 skill 加载步骤，直接用通用 LLM 知识进行安全分析——输出结果看起来合理，实际上缺少结构化 skill 的指导。
