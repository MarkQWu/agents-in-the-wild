# Imbad0202/academic-research-skills — 学习案例

**仓库**：https://github.com/Imbad0202/academic-research-skills
**Stars**：未收录（registry 未更新）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-14（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `template-design`, `experience-accumulation`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`academic-research-skills` 是一个**学术研究自动化管道**，为学术用户提供从「选定研究主题」到「生成完整论文」的端到端 AI 工作流，包含深度研究（deep-research）、论文撰写（academic-paper）、论文同行评审（academic-paper-reviewer）和研究完整性验证（academic-pipeline）四大模块。

关键事实：
1. 作者 Imbad0202 是研究用途的 Claude Code 深度用户，仓库服务于真实的学术写作场景
2. 该仓库是本批次中**演进最剧烈**的案例：2026-04-12 审计得分仅 20/100（全批次最低），但当前 HEAD 已有根本性改进
3. 核心设计：4 个独立工作流（SKILL.md 为入口），每个 SKILL.md 下有专属的多个 agent 分工协作
4. 最重要的演进：**全部 35 个 agent 都补充了 [frontmatter](#frontmatter)**，从完全无法注册到可以正常工作
5. 新增了 `shared/` 目录（跨模块共享资源：模板、规格合约、参考文献）和独立 `skills/` 目录

### 1.2 架构剖析

**当前 HEAD 目录结构**：
```
academic-research-skills/
├── deep-research/                  # 深度研究工作流
│   ├── SKILL.md                   # 入口：触发整个深度研究流程
│   ├── agents/                    # 12 个专业 agent（研究问题 / 来源验证 / 综合 / 元分析等）
│   │   ├── research_question_agent.md
│   │   ├── source_verification_agent.md
│   │   └── ...
│   ├── references/
│   └── examples/
├── academic-paper/                 # 论文撰写工作流
│   ├── SKILL.md
│   ├── agents/                    # 11 个 agent（结构架构师 / 草稿写手 / 引用合规 / 格式化等）
│   └── templates/
├── academic-paper-reviewer/        # 论文评审工作流
│   ├── SKILL.md                   # 版本号：1.10.0（frontmatter）
│   ├── agents/                    # 8 个 agent（领域评审 / 方法论评审 / EIC 等）
│   └── references/
├── academic-pipeline/              # 完整研究流水线（整合以上 3 个工作流）
│   ├── SKILL.md
│   └── agents/                    # 4 个 agent（状态跟踪 / 完整性验证等）
├── shared/                         # ← 新增：跨模块共享资源
│   ├── agents/                    # 共享 agent（如双语摘要生成）
│   ├── templates/                 # 格式模板
│   ├── references/                # 参考资料
│   └── contracts/                 # ← 关键：接口规格合约（evaluator/audit/reviewer/writer 等）
│       ├── evaluator/
│       ├── audit/
│       ├── reviewer/
│       └── writer/
└── skills/                         # ← 新增：独立技能目录
```

- **文件类型分布**（当前 HEAD）：4 个 SKILL.md（入口）、~35 个 agent（全部已补 frontmatter）、约 8 个 contract 规格文件、共享模板和参考文献
- **编排关系**：用户调用对应模块的 SKILL.md → SKILL.md 依序调用该模块的专业 agent → 共享 `shared/agents/` 中的通用 agent（如双语摘要）按需引入 → `shared/contracts/` 定义 agent 间的接口规格
- **跨件契约**：`shared/contracts/` 是本仓库当前 HEAD 最重要的新增 —— 每种 agent 角色（evaluator/writer/reviewer/auditor 等）都有对应的接口规格文件，定义「这类 agent 应该接收什么输入、输出什么格式」

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「领域特化 + 角色专一」—— 每个 agent 只负责学术写作流程中的一个细粒度角色（提问专家、来源验证专家、综合专家、元分析专家），通过 SKILL.md 串联
- **解决什么问题**：学术写作的各阶段（选题→研究→写作→评审→修订）对专业度要求不同，泛用 AI 一次性完成质量不稳定；专化 agent 网络让每个阶段的 AI 行为更可预测
- **Trade-off**：35 个 agent 的粒度极细，维护成本高；但作者用 `shared/contracts/` 解决了「接口不一致」问题，是一个正确的演进方向
- **认知模型**：把学术研究流程视为「多步骤、多专家、可验证」的质量保证流水线，每个 agent 是流水线上的一道工序，合约是工序之间的质量标准

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「领域管道（Domain Pipeline）+ 接口合约（Interface Contract）」**

关键特征：
- 业务流程映射为 SKILL 级别的工作流（每个 SKILL 是一个完整业务流）
- SKILL 内部由细粒度 agent 协作完成（每个 agent 是一个专业工序）
- 跨模块共享的 agent 和资源统一放入 `shared/` 目录
- `contracts/` 定义 agent 角色的接口规格（输入/输出约定），确保跨模块 agent 之间的互操作性

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域知识密集型工作流（学术、法律、医疗） | ✅ 高度适用 | 每个工序都有专业 agent，质量可控 |
| 输出质量有可验证标准的场景 | ✅ 适用 | `contracts/` 定义了可验证的接口规格 |
| 快速一次性的轻量任务 | ❌ 不适用 | 35 个 agent 的引擎过重 |
| 需要实时交互的场景 | ❌ 不适用 | 多 agent 串行流程延迟过高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：领域管道 + 接口合约 | Imbad0202/academic-research-skills | 领域专化、接口规范、跨模块复用 | 维护 35 个 agent + 接口合约成本高 |
| 备选 A：单一全能 SKILL | 大多数个人 skill | 简单、易维护 | 质量不稳定，无法针对性优化 |
| 备选 B：外部 API 驱动 | 调用学术数据库 API | 能获取实时文献 | 依赖网络访问权限，NL 描述性弱 |

### 2.4 改进空间

1. **当前问题**：4 个 SKILL.md 均缺 `model` 声明和 `allowed-tools` 声明 **改进做法**：入口 SKILL 至少声明 `model: sonnet`（研究类任务需要高质量输出）和 `allowed-tools: Task` **预期收益**：SKILL 的执行行为更可预测
2. **当前问题**：`shared/contracts/` 是手动维护的规格文档，与 agent 定义是分离的 **改进做法**：在每个 agent 的 frontmatter 中加 `contract: shared/contracts/<role>/spec.md` 字段，或在 SKILL.md 中注明「本模块使用的合约版本」 **预期收益**：接口合约和 agent 定义之间形成可追踪的显式关联
3. **当前问题**：版本管理混乱（`academic-paper-reviewer/SKILL.md` frontmatter 写 1.10.0，但 body 的 changelog 引用与之不一致） **改进做法**：单一数据源：版本号只写在 frontmatter 里，body 中的版本信息通过 `See references/changelog.md for version history` 跳转 **预期收益**：消除未来的版本号不一致

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-12 当时得分 **20/100**（本批次最低）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `integrity_verification_agent.md` | 10 | 无 frontmatter + 零示例 + 无 model + ≥10 模糊量词 |
| `pipeline_orchestrator_agent.md` | 10 | 同上 |
| `academic-paper-reviewer/SKILL.md` | 65 | 版本号不一致（frontmatter 1.8 vs body 1.7）+ 无 model + 无 allowed-tools |
| `deep-research/SKILL.md` | 70 | 无 model + 无 allowed-tools + ≥10 模糊量词 |

**得 20 分的根本原因**：全部 35 个 agent 都没有 YAML frontmatter —— 这意味着这些 agent 在 Claude Code 中根本无法被注册，相当于 NL 架构的「零注册」状态。

### 3.2 当时值得借鉴的模式

1. **细粒度角色专化** → 35 个 agent 各司一职（提问专家、来源验证、偏见风险评估、元分析、双语摘要等），比「研究助手」这样的泛化 agent 更可靠 → 借鉴：领域专项 skill 要大胆拆细 agent，一个 agent 一个工序
2. **4 大工作流 + 共享资源的双层架构** → deep-research/academic-paper/reviewer/pipeline 是业务层，shared/ 是技术层 → 借鉴：当多个工作流有共同的 agent 或模板时，不要在每个工作流里复制，而是提取到 shared/ 目录
3. **`academic-pipeline/SKILL.md` 的端到端串联** → 把 3 个子工作流串联为完整的研究流水线，提供了「傻瓜化」的全流程入口 → 借鉴：复杂系统需要一个「全流程入口命令」，让用户不需要知道内部拆分就能完成完整任务
4. **结构分离（SKILL.md 与 agents/ 目录）** → 入口 SKILL.md 只描述流程逻辑和触发条件，具体执行由 agents/ 下的专家 agent 完成，职责分离清晰 → 借鉴：SKILL.md 是「总指挥」，不是「全能工」

### 3.3 当时的缺陷

1. **全部 35 个 agent 缺少 YAML [frontmatter](#frontmatter)**（包括 `name`、`description` 字段） → 根本原因：作者在设计 agent 时将其视为「纯功能描述文档」而非「可注册的 Claude Code 组件」，忽视了 frontmatter 是 Claude Code 注册机制的基础 → 自查：我的所有 agent 文件开头是不是 `---`？
2. **4 个 SKILL.md 均缺少 `model` 和 `allowed-tools`** → 根本原因：作者意识到了 SKILL 作为入口需要清晰的描述，但没有进一步到 frontmatter 字段的层次 → 自查：我的 SKILL.md frontmatter 里有没有 `model` 和 `allowed-tools`？
3. **`academic-paper-reviewer/SKILL.md` 版本号不一致（1.8 vs 1.7）** → 根本原因：版本号在两个地方维护（frontmatter 的 `version` 字段 + body 的 Version Info 表格），两处没有同步更新 → 自查：我有没有在多处维护同一份「元数据」（如版本号、作者名）？

### 3.4 当时的优化机会

1. 为所有 35 个 agent 批量生成最小 frontmatter（name 取文件名的下划线格式，description 取 body 第一句话）
2. 给 4 个 SKILL.md 补充 `model: sonnet` 和 `allowed-tools: Task`
3. 统一版本号：只在 frontmatter 中维护，body 中用「See changelog for history」代替内嵌版本表

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 全部 35 agent 无 frontmatter | `head -1 deep-research/agents/research_question_agent.md` | **已大量修复**：抽查多个 agent 均已有完整 frontmatter（name + description） | 核心修复；agent 现在可以被 Claude Code 正常注册 |
| `academic-paper-reviewer/SKILL.md` 版本号不一致 | `grep "version" academic-paper-reviewer/SKILL.md` | **部分演进**：frontmatter 现为 `1.10.0`，但 body 中仍有多处版本引用 | 版本号继续演进，但「两处维护」的根本问题未解决 |
| 4 个 SKILL.md 缺 `model` 和 `allowed-tools` | `grep "model:\|allowed-tools" deep-research/SKILL.md` | **仍存在**：4 个 SKILL.md 均未见 model/allowed-tools 声明 | 入口 SKILL 仍缺关键 frontmatter 字段 |

### 4.2 架构演进

从 20/100（2026-04-12）到当前 HEAD，这是本批次演进最大的案例：

**重大变化**：
1. 全部 35 个 agent 补充了 frontmatter（从零注册到完全可注册）
2. 新增 `shared/` 目录：包含跨模块共享 agent、模板、参考文献，以及最关键的 `contracts/` 规格目录
3. 新增独立 `skills/` 目录（具体内容待深挖）
4. `academic-paper/` 下新增 `examples/` 目录（具体示例文件）

**作者的演进逻辑**：先解决「注册」问题（补 frontmatter），再解决「复用」问题（建 shared/），再解决「规格」问题（建 contracts/）。这是正确的优先级顺序：**能跑 → 能复用 → 规格清晰**。

### 4.3 新增的可学习模式

**`shared/contracts/` 接口合约目录**是最重要的新增模式。它解决了多 agent 协作中的「隐性接口」问题：

- 当 `academic-paper-reviewer/` 中的 `evaluator` agent 和 `academic-pipeline/` 中的 `integrity_verification` agent 需要对同类输出做判断时，如果它们各自定义自己的「评分格式」，上游就无法统一处理
- `shared/contracts/evaluator/` 定义了所有「评估类」agent 应该遵守的输入/输出格式
- 这让跨模块的 agent 可以互操作，而不需要每次都适配

---

## 五、校准

### 5.1 我已经在做对的

1. **agent frontmatter 完整**：echo-sleuth 的 5 个 agent 均有 frontmatter（name + description），避免了该案例的核心问题
2. **不在多处维护同一份元数据**：echo-sleuth 的版本信息只在 `nlpm-badge.json` 里维护，不在每个 SKILL.md 中重复
3. **skill 与 agent 分层**：echo-sleuth 已经有 `skills/` 目录和 `agents/` 目录的分离，与本案例的良好实践一致

### 5.2 挑战 / 验证

- **验证**：本案例验证了「先解决注册，再解决质量」的演进优先级。当 NL 工件连最基本的 frontmatter 都没有时，优化示例和输出格式是没有意义的 —— Claude Code 甚至找不到这些工件。**注册能力 > 质量优化，这是 NL 工程的生死线**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查所有 agent 文件开头是否有 frontmatter
find /tmp/my-repos -name "*.md" -path "*/agents/*" | while read f; do
  first=$(head -1 "$f")
  if [ "$first" != "---" ]; then
    echo "NO FRONTMATTER: $f"
  fi
done

# 命中后：在文件开头加最小 frontmatter：
# ---
# name: <filename-without-extension>
# description: "<first sentence of body as description>"
# ---
```

```bash
# 检查我的 SKILL.md 是否有 model 和 allowed-tools 声明
find /tmp/my-repos -name "SKILL.md" | while read f; do
  has_model=$(grep -c "^model:" "$f" 2>/dev/null)
  has_tools=$(grep -c "^allowed-tools:" "$f" 2>/dev/null)
  [ "$has_model" -eq 0 ] && echo "NO MODEL: $f"
  [ "$has_tools" -eq 0 ] && echo "NO ALLOWED-TOOLS: $f"
done

# 命中后：在 frontmatter 加 model: sonnet 和 allowed-tools: Task（或具体工具列表）
```

```bash
# 检查我有没有在多处维护版本号
grep -rn "version" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null | grep -v ".git"

# 命中多处后：确认版本号的唯一数据源是哪里，其他地方改为「参见 X」
```

### 6.2 灵感 → 实施路径

1. **想法**：为 claude-for-legal 建立 `shared/contracts/` 目录，定义法律 agent 的接口规格  
   **为何可行**：claude-for-legal 有 5 个法律领域，每个领域的 `watcher` agent 都需要返回「风险摘要」；如果定义一个统一的 `contracts/watcher/spec.md`，所有 watcher agent 的输出格式就能统一，上游汇总命令就不用「适配 5 种格式」  
   **第一步**：在 `claude-for-legal/shared/contracts/watcher/` 下新建 `spec.md`，定义 watcher 类 agent 的输出格式（如：`RISK_LEVEL`, `SUMMARY`, `SOURCE`, `DEADLINE`） → 20 分钟

2. **想法**：给 echo-sleuth 的 shared agent（未来可能有的跨命令共用逻辑）提前规划 `shared/agents/` 目录  
   **为何可行**：echo-sleuth 的 `memory-auditor` 当前同时被 `audit` 和 `prune` 命令调用，将来如果 `extract` 命令也需要质量评估，这个 agent 就是「共享 agent」的天然候选；提前规划目录结构避免将来重组  
   **第一步**：把 `agents/memory-auditor.md` 移到 `shared/agents/memory-auditor.md`，更新命令文件中的引用路径 → 10 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 Imbad0202/academic-research-skills 的核心目的**：为学术研究流程提供从选题到论文的端到端 AI 工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 高 | 同为「领域知识密集型多工作流插件」，都有多个子模块、多个 agent | claude-for-legal 聚焦法律领域，academic-research-skills 聚焦学术写作；两者架构模式极为相似 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同为多 skill + 多 agent 的 Claude Code 插件 | echo-sleuth 是单一工具，无多业务流水线 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 同为创意领域工具 | drama-workshop 是 skills 集合，academic-research 是工作流管道 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 无 frontmatter | `find /tmp/my-repos -path "*/agents/*.md" -exec sh -c 'head -1 "$1" \| grep -v "^---" && echo "$1"' _ {} \;` | **未命中**：claude-for-legal 的 agent 均有 frontmatter | 无 |
| SKILL.md 缺 `model` | `grep -rL "^model:" /tmp/my-repos/MarkQWu-claude-for-legal/ 2>/dev/null \| grep "SKILL.md"` | **命中**：claude-for-legal 多个 SKILL.md 无 model 声明 | 中 |
| 版本号多处维护 | 手动检查 | claude-for-legal 未发现版本号多处维护问题 | 无 |
| 4 个 SKILL.md 缺 `allowed-tools` | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-claude-for-legal/*/SKILL.md 2>/dev/null` | **命中**：多个 SKILL.md 无 `allowed-tools` 声明 | 中 |

**具体行动建议**：
- `claude-for-legal/regulatory-legal/skills/reg-feed-watcher/SKILL.md` → 在 frontmatter 补 `model: sonnet` + `allowed-tools: Bash, Read` → 5 分钟
- 批量处理：`find /tmp/my-repos/MarkQWu-claude-for-legal -name "SKILL.md" -exec grep -L "^model:" {} \;` 的所有命中文件，逐一补 frontmatter → 每个约 3 分钟

### 8.3 别人的更优方案

1. **领域**：接口合约目录（shared/contracts/）  
   - **本案例做法**：`shared/contracts/evaluator/`、`shared/contracts/writer/` 等为每类 agent 角色定义标准化的输入/输出接口，所有该类 agent 都遵守合约  
   - **我的项目现状**：claude-for-legal 的 `watcher` 类 agent 各自有不同的输出格式（有的是 markdown 表格，有的是纯文本，有的是 JSON），上游汇总时需要解析不同格式  
   - **如何借鉴**：建立 `claude-for-legal/shared/contracts/watcher-output.md`，定义统一的输出结构；各 `watcher` agent 的 Output Format 节引用此合约

2. **领域**：「从 20 分到可用」的演进路径  
   - **本案例做法**：先补 frontmatter（让工件可注册）→ 建 shared/（让资源可复用）→ 建 contracts/（让接口规范化）。三步有先后，每步解决一个层次的问题  
   - **我的项目现状**：claude-for-legal 的演进路径不够系统，有时候在质量已不完善的基础上继续添加新功能  
   - **如何借鉴**：在修改 claude-for-legal 之前，先做一次「注册能力全检」（所有 frontmatter 是否完整），再做「复用能力全检」（有哪些 agent/模板可以提取到 shared/），最后才考虑新增功能

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：SKILL.md 的覆盖深度  
  - **我的做法**：echo-sleuth 的 4 个 SKILL.md 均有 `allowed-tools` 声明（Bash、Task 等），声明了执行所需的工具权限  
  - **本案例做法（弱在哪）**：即使到当前 HEAD，4 个 SKILL.md 仍然缺少 `allowed-tools` 字段，意味着这些入口 skill 的工具权限处于「隐性」状态  
  - **意义**：我在「SKILL.md frontmatter 完整性」这个维度上做得比 Imbad0202 好，但这也提醒我继续检查 `model` 字段是否也声明了

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。Claude Code 读取 agent/skill 文件时，必须先通过 frontmatter 中的 `name` 和 `description` 字段来注册组件。如果文件没有 frontmatter，Claude Code 无法识别这个文件是「可注册的 agent」还是「普通文档」，该 agent 会被跳过，永远不会被调用。这就是 Imbad0202 仓库在审计时得 20 分的根本原因。

### <a name="contracts"></a>接口合约（Interface Contracts）
> 一套描述「某类 agent 应当遵守什么输入/输出格式」的规格文档。在 Imbad0202 的 `shared/contracts/` 目录中，每种 agent 角色（evaluator、writer、reviewer、auditor）都有对应的合约文件。当多个 agent 属于同一「角色类型」时，合约确保它们的输出格式一致，上游 agent 可以统一处理，而不需要为每个 agent 单独写解析逻辑。

### <a name="single-source-of-truth"></a>单一数据源（Single Source of Truth）
> 某一份信息（如版本号、作者名、接口规格）只在一个地方定义，其他地方通过引用指向这个唯一来源。当这份信息需要更新时，只改一个地方即可。Imbad0202 的版本号在 frontmatter 和 body 两处维护，是「单一数据源」原则被违反的典型例子——任何一处更新、另一处没同步就会产生不一致。

### <a name="端到端管道"></a>端到端管道（End-to-End Pipeline）
> 将一个完整业务流程（从输入到最终输出）的所有步骤串联为一个自动化序列。本案例的 `academic-pipeline/SKILL.md` 就是一个端到端管道：用户只需触发这一个 skill，它会按顺序完成研究问题定义 → 深度研究 → 论文撰写 → 评审 → 完整性验证，用户不需要知道内部有 35 个 agent 在协作。
