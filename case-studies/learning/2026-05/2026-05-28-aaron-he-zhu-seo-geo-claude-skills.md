# aaron-he-zhu/seo-geo-claude-skills — 学习案例

**仓库**：https://github.com/aaron-he-zhu/seo-geo-claude-skills
**Stars**：1050 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-28（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `cross-reference`, `security-gate`, `single-purpose`

**xiaolai 案例**：[../../2026-04-24-aaron-he-zhu-seo-geo-claude-skills.md](../../2026-04-24-aaron-he-zhu-seo-geo-claude-skills.md)

---

## 一、理解

### 1.1 仓库概览

**seo-geo-claude-skills** 是一个面向 SEO（搜索引擎优化）和 GEO（生成式引擎优化）的 Claude Code [插件](#插件)库，作者 Aaron Zhu 在上面构建了两套专有质量框架——[CORE-EEAT](#core-eeat) 和 [CITE](#cite)——并为 7 种语言提供多语言触发器，支持 Claude Code、Cursor、Codex 等 35+ 个 AI 智能体运行时。

关键事实：
1. 版本从 audit 时的 9.0.1 演进到当前 9.9.9，经历了一次 7 阶段「瘦身/v10」大压缩，从 37,129 行压缩到 24,587 行（减少 34%）
2. Audit 时有 38 个 NL 产物（15 个 command，20 个 SKILL.md，钩子、[manifest](#manifest) 等）；当前命令层已重组为 17 个命令
3. 因 `commands/contract-lint.md` 缺少 `allowed-tools` 字段，该库的 SHA-256 完整性校验机制长期处于「纸上谈兵」状态——这是本案例核心讽刺：**守门人自己没有入门的钥匙**
4. PR #10（xiaolai 提交 `allowed-tools` 修复）在 issue 提交后 6 小时内被合并，维护者响应极为积极
5. 当前版本新增 `gemini-extension.json`、`qwen-extension.json`（多运行时支持）及 `references/decisions/` ADR 目录，架构显著成熟

### 1.2 架构剖析

```
seo-geo-claude-skills/
  commands/               # 17 个命令（当前）
    /aaron:auto           # 全自动 SEO 流水线
    /aaron:max            # 最大化模式
    /aaron:discover       # 关键词发现
    /aaron:compete        # 竞争对手分析
    /aaron:audit          # 域名审查
    /aaron:write          # 内容生成
    ... (共 17 个)
  skills/                 # 20 个 SKILL.md（跨 SEO/GEO 领域）
    research/             # 关键词研究类
    build/                # 内容生成类
    optimize/             # 技术 SEO 类
    monitor/              # 监控与报告类
    cross-cutting/        # 跨域（记忆管理、质量审查）
  references/             # 内部参考文档
    decisions/            # ADR（架构决策记录）
    auditor-runbook.md
    core-eeat-benchmark.md
    skill-contract.md
    evolution-protocol.md
  hooks/hooks.json        # PostToolUse + Stop 钩子
  .claude-plugin/
    plugin.json           # manifest（当前版本 9.9.9）
  gemini-extension.json   # Gemini 运行时扩展
  qwen-extension.json     # Qwen 运行时扩展
  scripts/
    validate-skill.sh     # 技能校验脚本
  .mcp.json               # 14 个外部 API 的 MCP 服务器配置
```

**文件类型分布**：20 个 SKILL.md，17 个 command，1 个 hooks.json，1 个 plugin.json manifest，2 个运行时扩展文件，多个参考文档。

**编排关系**：命令层是用户入口（`/aaron:*`），调用 skill 层执行具体工作；`cross-cutting/` 下的 `memory-management` 和 `content-quality-auditor` 是跨领域技能，被多条链路复用。`contract-lint` 命令专门验证 auditor 类 skill 的 SHA-256 完整性——这个校验链路在 audit 时因缺少 `allowed-tools` 而断裂。

**跨件契约**：`skill-contract.md` 作为显式契约文档定义了 skill 间的接口规范；`auditor-runbook.md` 定义了 auditor 类 skill 的 SHA 标记协议；版本号通过 `validate-library` 命令在 `plugin.json` 与所有 SKILL.md 之间同步校验。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「SEO/GEO 领域知识的系统化提炼」——CORE-EEAT（内容可信度框架）和 CITE（引用触发框架）不是通用模式，而是作者在 SEO/GEO 领域多年积累的专有判断标准，被系统化为 AI 可执行的操作规范
- **解决什么问题**：Claude 做内容质量判断时缺乏可量化标准；GEO 是新兴领域，没有既有规范可循。该库的根本目标是把领域专家的隐性知识转化为 AI 可复现的显性规则
- **Trade-off**：多语言触发器（7 种语言）极大提升了全球可用性，但也带来了触发歧义风险（audit 发现 4 个 skill 在发现查询上存在触发重叠）；`skill-contract.md` + SHA 校验机制提升了完整性保障，但增加了维护摩擦——`contract-lint` 本身就是因为维护摩擦（缺 `allowed-tools`）而失效的
- **认知模型**：作者把 AI skill 视为「可执行的领域知识单元」而非「通用助手提示词」，`evolution-protocol.md` 说明其有意识地把 skill 版本化为可演进的协议，而不是静态配置

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「领域契约化 + 自校验质量门控」**

这个架构的核心不只是「有很多 skill」，而是在 skill 层之上建立了一套**自我约束机制**：`skill-contract.md` 定义接口规范，`contract-lint` 命令验证规范被执行，`validate-library` 验证版本一致性。问题在于，这套自校验机制本身也是需要被校验的——而 audit 恰好发现它没有通过自己的标准。

模式特征清单（4 条）：
- 特征 1：存在**专有质量框架**（CORE-EEAT、CITE），不依赖通用标准
- 特征 2：**契约文档显式化**——`skill-contract.md` 是 skill 间接口的唯一权威来源
- 特征 3：**SHA 完整性校验**——auditor 类 skill 通过哈希标记验证内容未被篡改
- 特征 4：**多运行时支持**——同一套 skill 通过不同 extension 文件适配多个 AI 平台

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 垂直领域专家知识系统化 | ✅ 高度适用 | CORE-EEAT/CITE 框架就是领域专家知识的 AI 可执行版本 |
| 需要跨平台部署的 skill 库 | ✅ 适用 | 多运行时扩展文件设计支持 Claude/Gemini/Qwen 等 |
| 小型单用途工具 | ❌ 不适用 | 契约化 + SHA 校验的维护成本过高，对小项目是过度工程 |
| 纯实验性/快速迭代项目 | ❌ 不适用 | 版本协议和契约文档需要稳定的命名和接口，频繁重构成本高 |
| 需要强一致性保障的生产环境 | ✅ 适用 | SHA 校验 + manifest 同步机制正是为此设计 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| **领域契约化 + 自校验**（本仓库） | seo-geo-claude-skills | 完整性有保障；专有框架深度定制 | 自校验机制本身可能失效（本案例核心教训）；维护负担高 |
| **单职责 skill 平铺**（无契约） | The-Vibe-Company/companion | 简单，每个 skill 独立可测试 | 无跨 skill 一致性保障；规模扩大后容易碎片化 |
| **Router + 分层编排** | SuperClaude-Org/SuperClaude | 有中心路由，工作流可编排 | 路由逻辑复杂；单 skill 替换影响整条链路 |

### 2.4 改进空间

1. **当前问题**：`contract-lint` 是自校验守门人，但其 `allowed-tools` 声明是人工维护的——未来压缩重构可能再次遗漏。**改进做法**：在 CI 中加入一个静态检查步骤，验证所有引用了 `shasum`/`awk`/`Grep` 等工具的命令文件都包含对应的 `allowed-tools` 声明。**预期收益**：把「守门人失效」从运行时错误提前为 CI 错误，避免下一次「瘦身」压缩时再次丢失。

2. **当前问题**：`skill-contract.md` 是单点权威，但没有机制验证 skill 是否真的遵循了契约中定义的接口。**改进做法**：在 `validate-library` 命令中增加契约合规性检查（如验证每个 SKILL.md 是否包含契约要求的必填节）。**预期收益**：从「声明式契约」升级为「可执行契约」。

3. **当前问题**：多语言触发器（7 种语言）分散在各 SKILL.md 中，trigger 重叠问题难以全局发现。**改进做法**：在 `validate-library` 中加入跨 skill trigger 去重检查，检测语义相近的触发词。**预期收益**：消除 audit 发现的 4 个 skill trigger 重叠问题。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：91/100**（weighted：命令层 88.0 / skill 层 93.6 / 基础设施 88.3）。Security：CLEAR（2 个 Low 级发现）。Bugs：3 个，质量问题：21 个。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/contract-lint.md` | 80 | BUG：缺 `allowed-tools`，SHA-256 校验功能完全失效 |
| `commands/wiki-lint.md` | 82 | 缺 `allowed-tools`，7 步检查表无编号顺序步骤 |
| `commands/optimize-meta.md` | 83 | 缺 `allowed-tools`，缺 `argument-hint` |
| `commands/keyword-research.md` 等 4 个 | 85 | 缺 `allowed-tools`，有模糊量词 |
| `commands/audit-domain.md` | 88 | 缺 `allowed-tools` |
| `commands/p2-review.md` | 88 | 缺 `allowed-tools`，硬编码墓碑日期 |
| `commands/geo-drift-check.md` | 91 | 缺数值阈值（「notable drift」无量化定义） |
| `commands/validate-library.md` | 96 | 接近满分，最优命令 |
| 20 个 SKILL.md | 92–95 | 模糊量词（「comprehensive」「appropriate」等） |
| `hooks/hooks.json` | 87 | Stop 钩子自动追加否决项，缺会话上下文检查 |
| `.claude-plugin/plugin.json` | 90 | BUG：版本 9.0.1 vs SKILL.md 版本 9.0.0，版本漂移 |

### 3.2 当时值得借鉴的模式

1. **专有框架系统化**（R05）→ CORE-EEAT 和 CITE 不是通用描述，而是可量化评分的领域框架，每个维度都有可测量指标 → 路径：各 SKILL.md 的 evaluation 节 → 借鉴方式：垂直领域工具不要借用通用质量标准，要把领域专家判断显式编码为评分维度

2. **多语言触发器覆盖**（R04）→ 同一 skill 提供英/中/日/韩/西/法/德 7 种触发短语，触发成功率远高于单语言 → 路径：所有 SKILL.md 的 `triggers:` [frontmatter](#frontmatter) 字段 → 借鉴方式：面向国际用户的工具需在 frontmatter 声明多语言触发变体

3. **SHA 完整性守护**（R08）→ auditor 类 skill 通过 SHA-256 标记机制防止无意篡改，`contract-lint` 作为守门命令验证这些标记 → 路径：`cross-cutting/content-quality-auditor/SKILL.md`，`commands/contract-lint.md` → 借鉴方式：高价值 skill 可以引入内容哈希作为变更检测机制，配合 CI 验证

4. **版本同步验证命令**（R12）→ `validate-library` 命令专门检查 `plugin.json` 版本与所有 SKILL.md 版本是否一致，把手动对齐变成自动验证 → 路径：`commands/validate-library.md` → 借鉴方式：多文件插件必须有版本同步验证步骤，不能依赖人工检查

5. **`references/decisions/` ADR 模式**（R07）→ 用架构决策记录（ADR）记录为什么做某个设计选择，防止后续维护者撤回已验证的决策 → 路径：`references/decisions/` 目录 → 借鉴方式：在 NL 插件中引入 `decisions/` 目录，记录重要的设计选择和被否定的替代方案

### 3.3 当时的缺陷

1. **质量守门人自己过不了关**：`contract-lint` 缺少 `allowed-tools: ["Read", "Grep", "Bash"]`，导致其声明的 SHA-256 校验逻辑在默认权限配置下完全无法执行。根本原因：`allowed-tools` 字段在作者阶段没有静态验证，缺失是沉默的——命令文件语法正确，库自己的 linter 也无法发现这个缺陷，因为 linter 本身也有同样的问题。自查：我的命令文件有没有声明了工具调用却没有 `allowed-tools`？

2. **15 个命令中 7 个缺 `allowed-tools`**：不只是 `contract-lint`，包括 `keyword-research`、`report`、`setup-alert`、`write-content`、`wiki-lint`、`optimize-meta`、`audit-domain`，合计 7 个命令声明了工具使用却没有授权声明。根本原因：命令文件的工具声明和 `allowed-tools` 字段没有强制配对，一个写了，另一个可以忘记。自查：我的 `echo-sleuth-for-claude` 里的 agent 文件有没有声明了工具却没有 `allowed-tools`？

3. **18/20 个 SKILL.md 含模糊量词**：「comprehensive」「appropriate」「thorough」「significant」「relevant」高频出现。根本原因：这些词是领域语言中的习惯用法，写的时候感觉很自然，但对 AI 执行者来说它们没有操作边界。部分 skill 通过并列的数值标准（如 8 阶段工作流、禁词列表）缓解了这个问题，但不是所有 skill 都做到了这一点。自查：我的 skill 里有多少「高质量」「完整」「适当」这类词？

4. **硬编码业务日期**：`p2-review.md` 里有一个具体的墓碑审查日期（tombstone review date），时间过了就变成僵尸指令。根本原因：把时态敏感的业务规则硬编码进 NL 文件，而不是用相对时间或配置变量。自查：我的 NL 文件里有没有硬编码的绝对日期或版本号？

5. **两个 auditor 类 skill 哈希标记相同**：`content-quality-auditor` 和 `domain-authority-auditor` 的 SHA 标记相同，说明它们被创建时从同一个模板复制，但 `contract-lint` 的校验无法区分这两个来源。根本原因：模板复用时 SHA 标记没有被重新计算。自查：我如果使用了哈希标记作为完整性证明，有没有确保每个文件的哈希是唯一计算的？

### 3.4 当时的优化机会

1. **最高杠杆**：一次性修复 7 个命令文件的 `allowed-tools` 缺失——每个文件加 1 行，合计恢复 7 个命令的完整功能。耗时估算：30 分钟内完成全部 7 个。

2. **最大分数提升**：为模糊量词添加数值锚点——不是删除「comprehensive」，而是紧跟一个具体的量化标准（如「comprehensive（覆盖 8 个维度，见附表）」）。最容易改的是那些已经有评分框架但没有数值的 skill。

3. **最精确的单点修复**：`geo-drift-check.md` 中的「notable drift」需要一个数值阈值（如「>5% 的排名变化」），否则「notable」完全是模型自由裁量。这一个词决定了整个漂移检测命令的行为。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `contract-lint.md` 缺 `allowed-tools` | 检查文件是否存在及内容 | `commands/contract-lint.md` 路径已不存在（命令层重组）；`plugin.json` 版本 9.9.9，SKILL.md 版本 9.9.9 **一致** | Bug #3 版本漂移已修复；命令层在瘦身重组中被重新命名，原路径失效 |
| `wiki-lint.md` 无编号步骤 | 文件路径检查 | 原路径 `commands/wiki-lint.md` 不存在（命令重组） | Bug #2 随命令重组消失或被重新实现 |
| `plugin.json` 版本漂移 | 比对 manifest 与 SKILL.md 版本字段 | 两者均为 9.9.9，**已对齐** | Bug #3 明确修复 |
| 17 个命令（当前）vs 15 个命令（过去） | CLAUDE.md 命令清单 | 命令数量从 15 增加到 17 | 功能有扩展；`/aaron:auto`、`/aaron:max` 等新命令反映了「全自动流水线」新设计方向 |

### 4.2 架构演进

从 audit 时（v9.0.1，38 个产物）到现在（v9.9.9），架构经历了两个独立演进：

**演进 1：瘦身/v10 压缩（2026-04-23，7 阶段）**
- 命令层：15 个命令压缩 31%，同步修复了 `allowed-tools` 缺失问题（35 个发现中 25 个由维护者独立修复）
- skill 层：20 个 SKILL.md 压缩约 20%，模糊量词问题被清理
- 参考文件层：86 个参考文件压缩约 69%（去除冗余的通用内容）
- 这个演进是在 xiaolai PR 提交后数小时内独立启动的，不是响应 PR 的结果，而是维护者自己的重构计划

**演进 2：v9.9.9 架构扩展（压缩后）**
- 命令从 15 个增加到 17 个，新增 `/aaron:auto`（全自动流水线）和 `/aaron:max`（最大化模式），反映了向「全托管 SEO 工作流」的设计转型
- 新增 `gemini-extension.json` 和 `qwen-extension.json`，正式支持多运行时部署
- 新增 `references/decisions/` ADR 目录，架构决策记录化

**这说明作者意识到了什么**：原有命令层的「冗长 + 缺陷」是可以同时解决的——压缩掉的不只是行数，还有通过强制重写而修复的 `allowed-tools` 遗漏。

### 4.3 新增的可学习模式

1. **ADR 模式（`references/decisions/`）**：明确记录架构选择的理由和被否定的方案，防止后继维护者因不了解上下文而撤回已验证的决策。这是从隐性架构知识到显性协议知识的演进。

2. **多运行时扩展文件（`gemini-extension.json`、`qwen-extension.json`）**：同一套 skill 内容通过独立的运行时适配文件支持不同平台，而不是在 skill 文件内用条件分支处理平台差异。这是「关注点分离」在 NL 插件架构中的体现。

3. **`/aaron:auto` 全自动流水线命令**：从离散的单步骤命令（`/seo:keyword-research`、`/seo:write-content`）升级为端到端编排命令（`/aaron:auto` 一次执行完整 SEO 流程）。这个演进说明维护者已经有了「用户不应该手动编排多个 skill」的认知。

---

## 五、校准

### 5.1 我已经在做对的

1. **版本号单一权威来源**：我的 `echo-sleuth-for-claude` 只有一处版本声明，不存在 manifest 和 SKILL.md 版本不同步的问题。本案例的 Bug #3 验证了「版本号必须单点维护或自动同步」的原则。

2. **避免硬编码业务日期**：我的 NL 文件里没有绝对日期的业务规则。本案例的「tombstone date」问题提醒了这个反模式的危险性。

3. **小型工具不过度工程化**：我的三个项目都没有试图建立 SHA 校验机制——本案例证明即使建了这个机制，如果 `allowed-tools` 缺失，整个机制也是空的。在没有 CI 保障的情况下，手动维护校验链路的成本很高。

4. **单 skill 单职责**：我的 `echo-sleuth-for-claude` 的 4 个 skill 各有清晰边界（mine-decisions、mine-mistakes、mine-patterns、mine-wisdom），对应本案例的「单职责纵向切片」模式，这是对的方向。

### 5.2 挑战 / 验证

**挑战 1**：本案例颠覆了「高分仓库的质量门控是可信的」这一默认假设。91/100 的库的核心完整性机制在默认权限配置下是完全失效的。这说明 NL 产物的分数高不等于所有功能路径都通——功能可用性检查和质量评分是两个正交的维度。

**挑战 2**：「压缩重构可能引入新缺陷」被 re-audit 的 32 个新发现量化证实。这提醒我：任何大规模重写（即使目标是「删除冗余」）都应该有回归测试覆盖关键功能路径。对 NL 文件来说，「回归测试」就是 NLPM `/nlpm:test` 和 `/nlpm:check`。

**验证**：「维护者对外部 audit PR 的响应速度是库活跃度的信号」这个判断被本案例强化——同日合并 PR + 独立启动 7 阶段重构，远超「被动修复」的模式。选择学习对象时，响应速度和独立行动意愿是重要筛选维度。

---

## 六、行动

### 6.1 自查动作

**检查 1：我的命令/agent 文件有没有声明了工具但没有 `allowed-tools`？**

```bash
# 找出所有引用了工具名称（如 Bash、Grep、Read）却没有 allowed-tools 字段的文件
grep -rln "Bash\|Grep\|Write\|Edit" \
  ~/echo-sleuth-for-claude/ \
  ~/drama-workshop-skills/ \
  --include="*.md" 2>/dev/null \
| xargs grep -L "allowed-tools" 2>/dev/null
```

命中后怎么办：逐一审查命中文件，确认哪些工具调用是在 `allowed-tools` 字段里声明的，哪些只是正文描述。缺少声明的文件需要在 [frontmatter](#frontmatter) 或命令 body 的工具声明节补充 `allowed-tools`。

**检查 2：我的 NL 文件里有多少模糊量词？**

```bash
# 统计模糊量词出现次数（按文件排序）
grep -rn -iE '\b(comprehensive|appropriate|thorough|significant|relevant|robust|efficient|high.quality)\b' \
  ~/echo-sleuth-for-claude/ \
  ~/drama-workshop-skills/ \
  --include="*.md" 2>/dev/null \
| sed 's/:.*//' | sort | uniq -c | sort -rn
```

命中后怎么办：命中次数 >3 次的文件是重点整改对象。对每个词，检查其前后是否有量化标准或枚举列表作为补充；如果没有，用具体标准替换或在旁边加括号注释。

**检查 3：我的 plugin.json 版本和 SKILL.md 版本是否一致？**

```bash
# 提取 plugin.json 版本
grep -r '"version"' ~/echo-sleuth-for-claude/**/*.json 2>/dev/null | grep -v node_modules

# 提取所有 SKILL.md 的版本声明
grep -rn "^version:" ~/echo-sleuth-for-claude/ --include="SKILL.md" 2>/dev/null
```

命中后怎么办：如果版本不一致，确认是「发布前的版本预跑」还是「漏更新」，用 `validate-library` 类的命令检查所有文件是否同步。

**检查 4：我的 NL 文件里有没有硬编码的绝对日期？**

```bash
# 查找形如 2026-xx-xx 或 20xx 的绝对年份
grep -rn -E '\b20[0-9]{2}[-/][0-9]{2}[-/][0-9]{2}\b|\b20[0-9]{2}\b' \
  ~/echo-sleuth-for-claude/ \
  ~/drama-workshop-skills/ \
  --include="*.md" 2>/dev/null \
| grep -v "changelog\|history\|created\|#"
```

命中后怎么办：评估是业务规则还是历史记录。业务规则型日期（如「到 2025-12 前必须完成」）需要改为相对时间或配置变量；历史记录型日期（changelog）不需要改。

### 6.2 灵感 → 实施路径

1. **为 `echo-sleuth-for-claude` 的 agent 文件补充 `allowed-tools`**
   - **想法**：根据 task 描述中提到的「No allowed-tools in agents」，这个问题和 `contract-lint` 的 Bug #1 同构
   - **为何可行**：agent 文件声明了工具使用意图但没有 `allowed-tools`，在默认权限配置下功能路径断裂；补充这个字段是 1 行改动
   - **第一步**：用检查 1 的命令找出所有命中文件，逐一确认使用了哪些工具，补充 `allowed-tools` 字段；预计 15–30 分钟

2. **参考 ADR 模式，在 `drama-workshop-skills` 里引入 `decisions/` 目录**
   - **想法**：我的 `drama-workshop-skills` 已经有 `references/` 目录，离 ADR 模式只差一步
   - **为何可行**：当前「为什么选择 7 维度质量框架而不是 5 维度」这类决策只存在于 commit message 里，6 个月后自己也会忘记
   - **第一步**：在 `references/` 下新建 `decisions/` 子目录，写第一个 ADR 记录最重要的一个设计选择（如「为什么不用 frontmatter 而是 body 来声明质量维度」）；预计 20 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 aaron-he-zhu/seo-geo-claude-skills 的核心目的**：把 SEO/GEO 领域专家知识系统化为 AI 可执行的专有框架，支持多平台部署

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为领域专用 skill 库（创作 vs SEO）；有 references/ 目录；有质量框架（7+8 维度）| 无多运行时支持；无 SHA 校验；无版本同步命令 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code 插件，有 skill + agent 架构 | 功能完全不同（历史挖掘 vs SEO）；agent 缺 `allowed-tools`（同构问题）| 高（结构问题） |
| MarkQWu/claude-for-legal | 低 | 同为领域专用插件 | 法律 vs SEO；架构信息不足 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令/agent 缺 `allowed-tools` | `grep -L "allowed-tools" echo-sleuth-for-claude/**/*.md` | `echo-sleuth-for-claude` 的 5 个 agent 文件均无 `allowed-tools`（已知，任务描述中提及） | 高 |
| SKILL.md 含模糊量词 | `grep -cE "comprehensive\|appropriate\|thorough" drama-workshop-skills/skills/*/SKILL.md` | `drama-workshop-skills` 命中约 2 处（任务描述：远少于本案例的 18/20）| 低 |
| 版本号多文件漂移 | `grep "version" plugin.json echo-sleuth-for-claude/SKILL.md` | 需实际检查；本案例提醒这个问题高度隐蔽 | 中 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude` 的所有 agent 文件 → 参照本案例 Bug #1 的修复方式，在每个 agent 文件的 frontmatter 中添加 `allowed-tools` 字段，列出该 agent 实际调用的工具；5–15 分钟/文件，共 5 个文件，预计 1 小时内完成

### 8.3 别人的更优方案

1. **领域：版本同步验证命令**
   - **本案例做法**：`commands/validate-library.md` 是一个专用命令，驱动 Claude 检查所有 SKILL.md 版本与 `plugin.json` 版本是否一致，`validate-library.md` 得分 96/100，是命令层最高分
   - **我的项目现状**：`drama-workshop-skills` 和 `echo-sleuth-for-claude` 没有类似的版本同步检查命令，版本一致性完全依赖人工
   - **如何借鉴**：在 `echo-sleuth-for-claude` 的 commands/ 下新建 `validate-plugin.md`，参照 `validate-library.md` 的结构，定义检查 manifest 版本与所有 SKILL.md 版本一致性的工作流；完成后在每次发布前运行 `/validate-plugin`

2. **领域：SHA 完整性标记（保守版本）**
   - **本案例做法**：auditor 类 skill 内嵌 SHA-256 标记，`contract-lint` 命令定期验证标记是否匹配源内容
   - **我的项目现状**：没有任何完整性校验机制
   - **如何借鉴**：对 `drama-workshop-skills` 里最核心的 1–2 个 skill（质量评估框架 skill），可以在文件末尾加一条 `<!-- sha256: <hash> -->`，并在 CLAUDE.md 里写明「修改此文件后需更新 SHA 注释」。不需要完整的 `contract-lint`，但建立「核心文件有完整性标记」的习惯

3. **领域：ADR（架构决策记录）**
   - **本案例做法**：`references/decisions/` 目录记录架构选择的理由，防止维护者自我撤回
   - **我的项目现状**：`drama-workshop-skills` 有 `references/` 但没有 `decisions/` 子目录，设计决策散落在 commit message 里
   - **如何借鉴**：按 6.2 第 2 条的路径，为 `drama-workshop-skills` 新建 `references/decisions/`，写第一篇 ADR

### 8.4 反向：我的项目做得比他们好的地方

1. **领域：模糊量词密度**
   - **我的做法**：`echo-sleuth-for-claude` 的 skill 文件只有约 2 处模糊量词（任务描述提及「only 2 hits」）
   - **本案例做法**：18/20 个 SKILL.md 含模糊量词，是 audit 最高频的问题类别之一
   - **意义**：这说明我在 skill 写作时对「可执行性 > 可读性」的直觉已经比较准——「只有 2 处」相比「18/20 文件」是量级差距。这是维持的优点，不是需要改进的地方

---

## 八、术语表

### <a name="插件"></a>插件
> Claude Code 中，「插件」是一组打包在一起的 NL 产物（skill、command、agent、hook），通过 `plugin.json` [manifest](#manifest) 声明所有组件路径。用户用 `claude plugin install` 安装，装完后就能直接用插件里定义的命令和技能。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。本案例中 manifest 版本和 SKILL.md 版本不一致（Bug #3）就是 manifest 维护不当的典型表现。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`triggers`、`allowed-tools` 等）。Claude Code 读 SKILL.md 或 command.md 时先解析 frontmatter 才知道这个组件怎么注册和调用。`allowed-tools` 字段就在 frontmatter 里——缺了它，命令文件里声明的工具调用权限就不存在。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code [frontmatter](#frontmatter) 里的一个字段，声明该命令/agent 被允许调用的工具列表（如 `["Read", "Bash", "Grep"]`）。在默认权限配置下，没有声明在 `allowed-tools` 里的工具调用会被拒绝——不是报错，而是静默拒绝。本案例核心 Bug #1 的根源就在于：`contract-lint` 命令需要 Bash 和 Grep 执行 SHA-256 校验，但没有在 `allowed-tools` 里声明，导致校验逻辑完全无法执行。

### <a name="CORE-EEAT"></a>CORE-EEAT
> 本仓库自定义的内容可信度评估框架，全称 Core Experience, Expertise, Authoritativeness, Trustworthiness（核心经验、专业性、权威性、可信度）。它是 Google EEAT 框架的 SEO/GEO 领域扩展版本，把 AI 内容的评估维度系统化为可量化的评分标准。在 skill 文件里，它作为 Claude 执行内容评估时的判断依据。

### <a name="CITE"></a>CITE
> 本仓库自定义的 GEO（生成式引擎优化）框架，全称 Citation Influence and Trigger Effectiveness（引用影响力与触发效果）。专门针对 AI 搜索引擎（如 Perplexity、ChatGPT Search）引用内容的机制设计，指导如何写内容才能让 AI 搜索引擎更频繁地引用。

### <a name="SHA-256"></a>SHA-256
> 一种哈希算法，把任意内容转换为固定长度（64 个十六进制字符）的「指纹」。同样的内容永远产生同样的指纹；内容哪怕改了一个字符，指纹就完全不同。本仓库用 SHA-256 给 auditor 类 skill 的内容生成指纹，`contract-lint` 命令定期重新计算并比对，确保这些核心 skill 没有被无意修改。

### <a name="GEO"></a>GEO（生成式引擎优化）
> Generative Engine Optimization 的缩写。随着 ChatGPT、Perplexity 等 AI 搜索引擎的兴起，用户越来越多地从这些平台获取信息，「让内容被 AI 搜索引擎引用」成为新的 SEO 任务。GEO 就是针对这个场景的内容优化方法论——和传统 SEO（针对 Google 蜘蛛）不同，GEO 更关注内容的权威性、引用价值和结构化表达。

### <a name="ADR"></a>ADR（架构决策记录）
> Architecture Decision Record 的缩写。一种文档格式，用来记录「为什么做了这个架构选择」而不只是「做了什么」。通常包含：背景、被考虑的选项、最终决定、决定的理由、被否定的选项。本仓库的 `references/decisions/` 目录就是 ADR 存放位置。好的 ADR 防止维护者因「不知道为什么这么设计」而撤回已验证的决策。

### <a name="瘦身/v10"></a>瘦身/v10 压缩
> 本案例中，Aaron Zhu 在 2026-04-23 发起的一次 7 阶段库级别重写，把仓库总行数从 37,129 压缩到 24,587（减少 34%）。各阶段分别针对：删除废弃提案和 changelog（Phase 1）、压缩 20 个 SKILL.md（Phase 2）、压缩 86 个参考文件（Phase 3，通用内容删减约 69%）、压缩 15 个命令（Phase 4，减少 31%）、重构根文档（Phase 5）、精简触发器和压缩钩子（Phases 6–7）。这次压缩由 16 个内部 agent 验证，但也引入了 32 个新的 NL 质量发现（re-audit 结果）。

### <a name="MCP"></a>MCP 服务器
> Model Context Protocol 的缩写。Claude Code 支持通过 `.mcp.json` 配置外部工具服务器，让 Claude 能调用外部 API（如 Ahrefs、SEMrush、HubSpot 等）。本仓库配置了 14 个 HTTP MCP 服务器，对应 14 个外部 SEO/营销 API。MCP 服务器的安全风险在于：如果 MCP 服务器地址被劫持或服务器本身恶意，Claude 的所有调用都会通过它——本案例的 2 个 Low 级安全发现之一就是这 14 个 HTTP MCP 服务器（无 API key 泄露，但存在外部依赖风险）。

### <a name="触发器重叠"></a>触发器重叠
> 当多个 skill 的 `triggers` 字段包含语义相近的关键词时，用户的一次输入可能同时触发多个 skill，导致 Claude 不知道该执行哪一个。本仓库 4 个 skill 在「发现查询」类触发词上存在重叠——这是多语言触发器（7 种语言 × 20 个 skill = 大量触发词）带来的副作用。
