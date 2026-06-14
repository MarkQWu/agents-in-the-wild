# Donchitos/Claude-Code-Game-Studios — 学习案例

**仓库**：https://github.com/Donchitos/Claude-Code-Game-Studios
**Stars**：未收录（registry 未更新）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-14（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `model-pinning`, `single-purpose`, `template-design`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`Claude-Code-Game-Studios`（简称 CCGS）是一个**游戏开发专项 AI agent 工作室**，将整个游戏工作室的职能（技术总监、创意总监、游戏设计师、剧情总监、音效设计师等）映射成 Claude Code agent，用 AI 扮演游戏工作室的每一个专业角色。

关键事实：
1. 作者 Donchitos 是独立游戏开发者，把 Claude Code 作为「全能游戏工作室」来使用
2. 仓库体量极大：49 个 agent + 72 个 skill + 多个 CLAUDE.md + 一个自检测框架（CCGS Skill Testing Framework）
3. 技能质量普遍较高（avg 94/100），但 agent 几乎全部缺示例（-15 per agent），导致整体得分拉低
4. 核心设计特色：**门控表决系统（Gate Verdict Pattern）** —— 每个「Director」agent 会对工作产物发出 `[GATE-ID]: APPROVE / CONCERNS / REJECT`，下游 skill 读取 gate 结果决定是否继续
5. 内置 CCGS 测试框架（`CCGS Skill Testing Framework/`），对自身的 agent 和 skill 做行为规格测试（dogfooding）

### 1.2 架构剖析

**目录结构**（关键层级）：
```
Claude-Code-Game-Studios/
├── .claude/
│   ├── agents/                # 49 个 agent（技术总监、创意总监、游戏设计师等所有角色）
│   │   ├── technical-director.md   # Gate 表决者：APPROVE/CONCERNS/REJECT
│   │   ├── creative-director.md    # 同上
│   │   ├── game-designer.md        # 专业设计师角色
│   │   ├── devops-engineer.md      # 模型：haiku（存争议）
│   │   └── ... (共 49 个)
│   ├── skills/                # 72 个 skill（start、sprint-plan、hotfix、gate-check 等）
│   │   ├── gate-check/SKILL.md     # 调用 director gate 并等待表决
│   │   ├── story-done/SKILL.md     # 完成故事任务，写入 session-state
│   │   └── ... (共 72 个)
│   └── docs/                  # 协调规则、gate ID 目录、模板
├── CCGS Skill Testing Framework/  # 自检测框架
│   ├── agents/                # 49 个测试规格文件（无 frontmatter，非注册 agent）
│   └── skills/                # 测试辅助技能
├── design/
│   └── CLAUDE.md              # 设计目录上下文
├── docs/
│   └── CLAUDE.md              # 文档目录上下文
└── src/
    └── CLAUDE.md              # 源码目录上下文
```

- **文件类型分布**：49 agent、72 skill、4 CLAUDE.md（各子目录）、49 测试规格文件（伪装成 agent 的测试 spec，无 frontmatter）
- **编排关系**：skill → 调用 agent（通过 Task dispatch）→ agent 输出 gate 表决 → skill 读取表决结果决定是否进入下一阶段。这是典型的「委托-评审」双层结构
- **跨件契约**：`story-done/SKILL.md` 读取 `.claude/docs/director-gates.md` 中的 gate ID 定义；多个 skill 读取 `production/review-mode.txt` 决定是否进入精简模式；`team-*` 系列 skill 读取 `technical-preferences.md` 动态选择引擎专家 agent

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「游戏工作室即 agent 网络」—— 每个传统游戏公司的职能角色都对应一个 agent，工作流程映射成 skill 调用序列
- **解决什么问题**：独立游戏开发者通常一人承担所有职能，用 AI agent 网络「模拟」工作室的多专业协作，为作者提供缺失的专业视角
- **Trade-off**：49 个 agent 提供了极高的角色粒度，但同时带来了「全员零示例」的问题 —— 每个 agent 都在描述角色身份，却没有展示「我怎么被调用」和「我输出什么格式」
- **认知模型**：作者把 Claude Code agent 视为「可插拔的虚拟员工」，gate 表决系统是这批虚拟员工相互制约的「内部 QA 协议」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多角色 Agent 网络 + 门控表决（Director Gate Protocol）」**

关键特征：
- 职能角色 1:1 映射为 agent（Director 级 + Lead 级 + Specialist 级三层）
- 每个 Director agent 输出结构化表决：`[GATE-ID]: APPROVE / CONCERNS / REJECT`
- Gate ID 统一定义在 `.claude/docs/director-gates.md`，形成全局字典
- skill 作为工作流驱动器，召集相关 agent，等待 gate 表决，再决定后续步骤

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要多专业视角评审的创意项目（游戏/影视/设计） | ✅ 高度适用 | director gate 系统天然支持多视角表决 |
| 个人开发者需要「假想队友」的项目 | ✅ 适用 | 每个 agent 都是一个可以给出专业意见的虚拟角色 |
| 纯代码工程项目（后端 API 等） | ⚠️ 过度设计 | 49 个 agent + 72 个 skill 对于常规工程任务偏重 |
| 需要快速迭代的原型开发 | ❌ 不适用 | 多 agent 协商流程慢，适合深度评审而非快速试验 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：多角色 agent + gate 系统 | Donchitos/CCGS | 深度专业角色、结构化评审协议 | 体量大、维护成本高 |
| 备选 A：单 orchestrator + 专业 agent | Ibrahim-3d/orchestrator-supaconductor | 灵活、模块化 | 无固定评审协议，由调用者自定义 |
| 备选 B：单 agent 多技能 | 大多数个人插件 | 简单易维护 | 无法提供多角色交叉验证 |

### 2.4 改进空间

1. **当前问题**：49 个 agent 零示例 **改进做法**：给每类 agent 写一个「角色调用模板」，内含 1 个最典型调用场景（触发词 + 预期输出片段），统一放在 `.claude/docs/agent-invocation-examples.md`，各 agent 文件通过 `@import` 引用 **预期收益**：解决最大的单项扣分（-15 × 49 = -735 raw penalty）
2. **当前问题**：`devops-engineer` 和 `community-manager` 使用 `model: haiku`，但任务复杂度需要 sonnet **改进做法**：修改这两个 agent 的 frontmatter `model: sonnet` **预期收益**：长文撰写和实施类任务质量提升，与 `.claude/docs/coordination-rules.md` 的模型使用策略保持一致
3. **当前问题**：`team-combat/SKILL.md` 缺少引擎未配置时的 fallback，但 `team-audio/SKILL.md` 有 **改进做法**：在 `team-combat` Phase 2 中加一行 `If technical-preferences.md is missing or engine is [TO BE CONFIGURED], skip specialist spawn and use generic code-review skill` **预期收益**：新项目首次运行不会因为未配置引擎而崩溃

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-12 当时得分 **82/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude/agents/live-ops-designer.md` | 59 | 零示例 + 无输出格式 + 8 个模糊量词 |
| `.claude/agents/game-designer.md` | 65 | 零示例 + 11 个模糊量词（上限） |
| `.claude/skills/architecture-decision/SKILL.md` | 72 | BUG：allowed-tools 缺 Edit |
| `.claude/skills/story-done/SKILL.md` | 74 | BUG：allowed-tools 缺 Write |
| `.claude/skills/gate-check/SKILL.md` | 94 | 较优秀，少量模糊量词 |

### 3.2 当时值得借鉴的模式

1. **门控表决协议（Gate Verdict Pattern）** → `[QL-TEST-COVERAGE]: ADEQUATE / GAPS / INADEQUATE` 这样的结构化输出让下游 skill 可以机械判断 → 路径：`.claude/agents/technical-director.md`、`qa-tester.md` → 借鉴：多 agent 协作时，让每个评审 agent 输出有限枚举值而非自由文本，下游才能可靠判断
2. **CCGS 测试框架（自检测）** → `CCGS Skill Testing Framework/` 里的 49 个测试规格文件对应 49 个 agent 的行为期望 → 借鉴：给自己的 agent 写测试规格（类似 NL-TDD），强制澄清「这个 agent 应该输出什么」
3. **`hotfix/SKILL.md` 的紧急模式** → skill 体中明确区分「正常路径」vs「紧急路径」，紧急时跳过 director gate 直接行动 → 借鉴：关键 skill 要有 fallback 路径，不能所有情况都必须等 gate
4. **多 CLAUDE.md 分层上下文** → `design/CLAUDE.md`、`docs/CLAUDE.md`、`src/CLAUDE.md` 各自提供子目录级别的上下文，而不是把所有信息挤进根 `CLAUDE.md` → 借鉴：大型仓库的 CLAUDE.md 应该分层，每个子目录的 CLAUDE.md 只讲该目录的规则
5. **`skill-test/SKILL.md` 的技能验证模式** → 可以用 skill 来验证其他 skill 的输出是否符合规格，形成闭环 → 借鉴：NL 工件可以用来测试其他 NL 工件

### 3.3 当时的缺陷

1. **全部 49 个 agent 零示例** → 根本原因：作者专注于描述每个角色的「知识和职责」，忽视了「如何被调用」这个维度；用户无法从 agent 文件中了解典型的调用触发词和预期输出格式 → 自查：我的 agent 有没有 `## Example` 节？
2. **`architecture-decision/SKILL.md` 的 `Edit` 工具缺失** → 根本原因：作者在 `allowed-tools` 里写了 `Read, Glob, Grep, Write`，但忘了 `Edit`；而 skill 体内第 6 步明确写「使用 Edit 工具更新架构注册表」→ 这是「声明」与「实现」不一致的典型 bug → 自查：我的 skill 中有没有「allowed-tools 声明了但体内用到了没声明的工具」？
3. **`devops-engineer` 和 `community-manager` 使用 `model: haiku` 做复杂任务** → 根本原因：作者可能为了降低 token 成本统一选了 haiku，但 `.claude/docs/coordination-rules.md` 明确规定 haiku 仅用于「只读状态检查、格式化、简单查询」—— 写 CI/CD 配置和撰写危机公关文案明显超出这个范围，形成了代码与文档的矛盾 → 自查：我的 agent model 分配是否与我自己的分层策略一致？

### 3.4 当时的优化机会

1. 给 49 个 agent 各补 1 个示例 block（每个约 5 分钟 × 49 = ~4 小时，但分批可行）
2. `qa-lead` agent 缺少 gate verdict 格式文档，虽然多个 skill 会读取其 `QL-*` gate 结果
3. 约 6 个非编码角色 agent（community-manager、game-designer 等）中有逐字复制的程序员专用「实现工作流」样板（「Should this be a static utility class or a scene node?」），与角色不符

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `architecture-decision/SKILL.md` 缺 `Edit` | `grep "allowed-tools" .claude/skills/architecture-decision/SKILL.md` | **已修复**：`allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion` ✓ | 作者接受了修复；Phase 0 和 Phase 6 现在可以正常调用 Edit |
| `story-done/SKILL.md` 缺 `Write` | `grep "allowed-tools" .claude/skills/story-done/SKILL.md` | **已修复**：`allowed-tools: Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, Task` ✓ | session-state/active.md 初始化不再失败 |
| `devops-engineer` 使用 `model: haiku` | `grep "model:" .claude/agents/devops-engineer.md` | **仍存在**：`model: haiku` 未改 | 作者可能认为 haiku 的成本优势优先于质量 |
| 49 个 agent 零示例 | `grep -l "## Example" .claude/agents/*.md \| wc -l` | **基本未改**：仅 1/49 个 agent 有示例节 | 示例添加工作量大，作者尚未系统性解决 |

### 4.2 架构演进

当前 HEAD 文件数（md 文件）为 392，审计时约 175 个，增长超过一倍。主要新增：
- `CCGS Skill Testing Framework/skills/` 新增了多个测试辅助技能子目录（readiness、gate、team、review、pipeline 等）
- `CCGS Skill Testing Framework/agents/` 的测试规格文件基本保持不变
- 核心 agent 目录未见重组

这说明作者把主要精力放在**扩充测试框架**，而非修复 agent 示例缺失，优先级判断是：「测试能力 > 文档完善性」

### 4.3 新增的可学习模式

- **更细粒度的测试框架分类**：`CCGS Skill Testing Framework/skills/` 下新增了 readiness、gate、review、pipeline 等专项测试类别，把测试技能本身也按「测什么」做了分层 —— 这是对「测试即代码」思想的深度实践

---

## 五、校准

### 5.1 我已经在做对的

1. **skill 的 allowed-tools 与 body 一致**：echo-sleuth 目前没有「声明了但体内未用」或「体内用了但未声明」的工具矛盾
2. **避免 model 分配矛盾**：echo-sleuth 的 agent 虽然未声明 model，但也没有为复杂任务分配 haiku 的问题
3. **多层 CLAUDE.md**：claude-for-legal 已经在每个法律子目录下放置专属 CLAUDE.md，与 CCGS 的分层上下文策略一致

### 5.2 挑战 / 验证

- **挑战**：本案例挑战了「agent 描述身份就够了」的假设。49 个高质量角色定义，因为没有示例，用户根本不知道怎么触发这些 agent。这告诉我：**agent 最重要的不是「我是谁」，而是「你什么时候叫我、我返回什么格式」**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 的 allowed-tools 是否与 body 中的工具调用一致
# 在 body 里找 "Edit"、"Write"、"Bash" 等关键词，对照 frontmatter
for f in /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md; do
  echo "=== $f ==="
  echo "frontmatter tools:"
  grep -A3 "^tools:" "$f" | head -4
  echo "body mentions:"
  grep -o "\b\(Edit\|Write\|Bash\|Read\|Glob\|Grep\)\b" "$f" | sort -u
done

# 命中「body 里有但 frontmatter 没有」的工具→ 补到 frontmatter 的 tools 字段里
```

```bash
# 检查 model: haiku 的 agent 是否承担了超出「只读、格式化、简单查询」的工作
grep -rn "model: haiku" /tmp/my-repos/ 2>/dev/null
# 命中后：读 agent body，判断它是否做了「写入」「长文生成」「实施判断」等 haiku 力不从心的工作
```

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 的 `agents/memory-auditor.md` 引入类似 CCGS 的结构化输出格式（枚举值而非自由文本）  
   **为何可行**：memory-auditor 目前返回自由文本，下游 `commands/prune.md` 要解析它；如果改成 `VERDICT: KEEP / PRUNE / ARCHIVE` 这样的结构化输出，prune 命令逻辑会更稳定  
   **第一步**：在 `agents/memory-auditor.md` 的末尾加 `## Output Format` 段，写出 `VERDICT: <KEEP|PRUNE|ARCHIVE>` 的格式约定 → 10 分钟

2. **想法**：给 claude-for-legal 的每个法律领域 agent 写一个测试规格文件（类 CCGS 自检测思路）  
   **为何可行**：法律 agent 的错误代价高（误判条款），CCGS 的 dogfooding 测试框架能帮助在修改 agent 后快速验证行为是否如预期  
   **第一步**：在 `litigation-legal/` 下新建 `tests/` 目录，写 `docket-watcher.spec.md`，描述「给定这样的触发条件，agent 应该输出这样的结果」→ 30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 Donchitos/CCGS 的核心目的**：用 AI agent 网络模拟游戏工作室的多专业协作，为独立开发者提供虚拟「队友」

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 同为「领域专家 agent 网络」（法律角色 vs 游戏角色） | claude-for-legal 是领域知识库，CCGS 是工作流驱动器；两者的 agent 职责定位不同 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件 | 目的完全不同（挖掘对话 vs 游戏开发） | 低 |
| MarkQWu/drama-workshop-skills | 中 | 同为创意内容领域 | drama-workshop 是 skills 集合而非 agent 网络 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 零示例 | `find /tmp/my-repos -name "*.md" -path "*/agents/*" -exec grep -L "## Example" {} \;` | **claude-for-legal：5/5 agents 无示例** | 高 |
| skill body 与 allowed-tools 不一致 | 逐文件手动检查 | echo-sleuth 未发现不一致 | 无 |
| 模糊量词（appropriate/relevant/suitable） | `grep -rn "appropriate\|relevant\|suitable" /tmp/my-repos/MarkQWu-claude-for-legal/ \| wc -l` | **命中 236 处**（以 claude-for-legal 最集中） | 高 |

**具体行动建议**：
- `claude-for-legal/regulatory-legal/agents/reg-change-monitor.md` → 在末尾加 `## Example` 节，写出「哪种监管公告触发我 → 我输出什么格式的变更摘要」→ 15 分钟
- `claude-for-legal` 中的模糊量词问题：优先处理频率最高的几个文件，用 `grep -n "appropriate"` 定位后逐条替换为具体标准

### 8.3 别人的更优方案

1. **领域**：门控表决协议（Gate Verdict Pattern）  
   - **本案例做法**：director agent 输出 `[GATE-ID]: APPROVE / CONCERNS / REJECT` 结构化表决，skill 机械判断结果  
   - **我的项目现状**：claude-for-legal 的 agent 输出自由文本，没有结构化的「批准/拒绝」信号，下游命令无法机械判断  
   - **如何借鉴**：在 `corporate-legal/agents/dataroom-watcher.md` 的 Output Format 段中引入 `RISK_VERDICT: LOW / MEDIUM / HIGH / CRITICAL`，让调用方可以直接根据枚举值决定后续动作

2. **领域**：多 CLAUDE.md 分层上下文  
   - **本案例做法**：`design/CLAUDE.md`、`docs/CLAUDE.md`、`src/CLAUDE.md` 各自聚焦子目录  
   - **我的项目现状**：claude-for-legal 有 `regulatory-legal/CLAUDE.md` 等，但覆盖不完整，`commercial-legal/` 无 CLAUDE.md  
   - **如何借鉴**：为缺失 CLAUDE.md 的子目录各补一个，内容只需说明该目录的职责和关键文件路径

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skill 的 `allowed-tools` 与 body 一致性  
  - **我的做法**：echo-sleuth 和 claude-for-legal 的 skill 均未发现「声明了没用」或「用了没声明」的工具矛盾  
  - **本案例做法（弱在哪）**：`architecture-decision` 和 `story-done` 都出现了严重的 allowed-tools 与 body 不一致（Edit/Write 缺失），导致实际运行失败  
  - **意义**：工具权限管理的一致性是我的项目相对较优的维度，维持这个好习惯

---

## 八、术语表

### <a name="gate-verdict"></a>门控表决（Gate Verdict Pattern）
> 一种多 agent 协作中的评审协议：某个「评审 agent」（Director）对工作产物给出结构化的有限枚举值表决，如 `[GATE-ID]: APPROVE / CONCERNS / REJECT`。下游技能根据这个枚举值机械决定是否继续流程，而不是解析自由文本。这样做的好处是：无论评审 agent 返回多少解释性文字，下游只看那几个固定的结果词，不会因为措辞差异而误判。

### <a name="dogfooding"></a>自检测（Dogfooding）
> 用自己的产品测试自己的产品。在本案例中，CCGS 项目写了 49 个「agent 行为规格文件」来描述 49 个 agent 应该做什么，然后用这些规格来验证 agent 是否按预期运行 —— 相当于给 NL agent 写了单元测试。「Dogfooding」原义是「吃自己的狗粮」，软件行业用来指「内部使用自己的产品」。

### <a name="model-haiku"></a>Haiku 模型（model: haiku）
> Claude 系列中成本最低、速度最快的模型，适合简单、高频、低复杂度的任务（只读查询、格式化、状态检查）。对于需要长文写作、创意决策、代码实现等高复杂度任务，Haiku 的输出质量低于 Sonnet。在 agent frontmatter 中指定 `model: haiku` 意味着该 agent 会始终使用 Haiku，无论任务多复杂。

### <a name="allowed-tools"></a>allowed-tools
> skill 或 command 的 frontmatter 字段，声明该工件在执行时有权调用哪些工具。如果 skill 体内写了「用 Edit 工具修改文件」，但 `allowed-tools` 里没有 `Edit`，Claude 执行到那一步时会报权限错误。这是「声明式权限」设计 —— 必须显式列出才有权限。
