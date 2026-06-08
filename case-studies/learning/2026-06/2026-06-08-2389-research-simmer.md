# 2389-research/simmer — 学习案例

**仓库**：https://github.com/2389-research/simmer
**Stars**：4 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-08（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `single-purpose`, `cross-reference`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Simmer 是 2389-research 的迭代精炼插件：给它任何工件（prompt、代码、配置文件、pipeline），配上你的评判标准，它就反复改进，直到你满意为止。每一轮循环有标准化的 Setup → Generate → Judge → Reflect 四个步骤。与其姐妹插件 review-squad（一次性多视角审查）不同，Simmer 聚焦"同一对象的多轮演进"。安全评级：CLEAR（零可执行面，纯 markdown）。

关键事实：
- 6 个 SKILL.md（1 个 orchestrator + 5 个 subskill）
- 0 个命令，0 个 hook，0 个脚本
- NL 得分 93/100（Audit 时），2026.md 兼容
- 安装：`claude plugin install simmer@2389-research`（但 audit 时 CLAUDE.md 没写这行）
- 版本 3.0.0（plugin.json），有设计文档 `docs/specs/2026-03-16-simmer-v2-design.md`

### 1.2 架构剖析

**目录结构**：
```
skills/
  simmer/SKILL.md               ← Orchestrator（550 行，超长）
  simmer-setup/SKILL.md         ← 识别工件 + 确定标准 + 确定评估方法
  simmer-generator/SKILL.md     ← 生成改进候选（基于当前版本 + ASI + 背景）
  simmer-judge/SKILL.md         ← 按标准评分（1-10），产出 ASI（改进建议）
  simmer-judge-board/SKILL.md   ← 多评委面板（JUDGE_MODE=board 时替换 judge）
  simmer-reflect/SKILL.md       ← 记录轨迹，追踪最佳，传递 ASI
.claude-plugin/plugin.json      ← manifest（满分 100）
CLAUDE.md                       ← 概览（无 install 命令，扣分点）
tests/integration/simmer-scenario.md  ← 集成测试场景
docs/specs/                     ← 设计文档
```

**文件类型分布**：6 个 SKILL.md / 0 个 command / 0 个 agent / 0 个脚本

**编排关系**：Orchestrator + subskill 链式调用。
1. 用户说"simmer this"触发 `simmer/SKILL.md`（orchestrator）
2. Orchestrator 串行调用：setup → [generator → judge（or judge-board）→ reflect] × N 轮
3. `JUDGE_MODE=board` 时，judge 被 judge-board 的多评委模式替换（drop-in replacement）

**跨件契约**：
- Orchestrator 通过 skill 名（`simmer:simmer-setup` 等）引用 subskill
- 所有 5 个 subskill 名都在 plugin.json 中声明，审计时 100% 一致，无孤立引用
- CLAUDE.md Subskills 表在 audit 时漏掉 simmer-judge-board，但当前 HEAD 已修复（L88/L90）
- judge-board 是 judge 的"可插拔替换"，通过 `JUDGE_MODE` 变量切换，两者接口契约相同

### 1.3 设计思路 / 方法论

**核心设计哲学**：「可替换评判层 + 确定性迭代结构」。每次迭代的步骤固定（S-G-J-R），但评判层可以是单评委（judge）或多评委面板（judge-board），靠 JUDGE_MODE 切换。设计目标是让"改进"这件事像流水线一样可重复、可观测、可中断。

**解决什么问题**：作者 2389.ai 的博文《the story behind Simmer》隐含了一个洞察：人类写 prompt 或代码时"凭感觉改"，不知道改了哪里，为什么，是否真的好了。Simmer 强制把每次迭代的评分、改进建议（[ASI](#ASI)）、轨迹都记录下来，让"改进"从黑盒变成可审计的过程。

**做了什么 trade-off**：
- 独立 simmer-judge vs simmer-judge-board（而非用参数区分）：可读性 > 复用性。两个文件内容不一样但入口相同，通过环境变量切换，而不是在同一个 skill 里写 if-else。
- Orchestrator 550 行（超 500 行上限）：作者选择了把所有模式（单 agent / 多 agent / runnable evaluator）都放进同一个 orchestrator，而不是拆分为多个。这方便用户理解全貌，但超出了 NLPM 的文件长度建议。

**反映什么认知模型**：作者把"精炼工件"看作一个科学实验——有假设（标准）、有对照（先前版本）、有实验（generator）、有评估（judge）、有记录（reflect）。这种"实验室"心智模型让整个流程高度结构化和可复现。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：Orchestrator + 链式 Subskill 架构（迭代精炼型）

一个 orchestrator 编排多个 subskill 按固定顺序链式执行，每轮产出传递给下一轮，支持 N 次迭代直到满足终止条件。

模式特征清单：
- **固定结构，可变内容**：每轮迭代结构固定（S-G-J-R），但内容（工件本身）在变
- **Drop-in Replacement**：judge 和 judge-board 有相同接口，靠变量切换，对 orchestrator 透明
- **ASI 作为上下文桥梁**：每轮 judge 产出的 ASI（改进建议索引）作为下一轮 generator 的输入，保证方向连续性
- **Reflect 记录轨迹**：专门的 reflect subskill 记录最佳版本和演进历史，防止迭代中"越改越烂"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| Prompt 工程 / 文本工件优化 | ✅ 高度适用 | 有明确标准时迭代精炼效果最好 |
| 代码质量改进（有测试套件） | ✅ 高度适用 | Runnable evaluator 模式可直接跑测试套件 |
| 创意生成（无固定标准） | ⚠️ 慎用 | 标准模糊时 judge 的评分随机性高，迭代效果不稳定 |
| 需要并行探索的任务 | ❌ 不适用 | Simmer 是串行深化，并行探索应用 simmer 的姐妹工具 cookoff/omakase-off |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Orchestrator + 链式 subskill（本仓库） | simmer | 迭代方向连续；有轨迹记录 | Orchestrator 文件容易超长 |
| 平铺多 skill（单次） | review-squad | 简单，一次完成 | 无法多轮精炼 |
| 单体 skill + 内置循环 | 很多简单插件 | 开发成本最低 | 复杂性集中，难以替换某个步骤 |

### 2.4 改进空间

1. **当前问题**：skills/SKILL.md 550 行（超 500 行上限），single-agent mode 章节（约 35 行）可以抽出。**改进做法**：把 single-agent mode 提取到 `references/single-agent-mode.md` 并在 orchestrator 中 `loads:` 引用。**预期收益**：orchestrator 降到 515 行内，满足 R05 要求，同时 single-agent 文档独立可读。
2. **当前问题**：simmer-judge-board 中 5 处 `relevant` 无具体标准，评委拿到提示后不知道"哪些工件和评分标准相关"。**改进做法**：把通用的 `relevant` 替换成"候选文件、评估脚本、配置文件和先前候选文件"（具体文件类型列表）。**预期收益**：评委行为更可预测，打分一致性提高。
3. **当前问题**：CLAUDE.md 无 `claude plugin install` 命令，读者不知道如何安装。**改进做法**：在 CLAUDE.md 加 Usage 章节（一行安装命令 + 触发短语）和 Prerequisites 章节（"None — pure markdown"）。**预期收益**：NLPM 从 93 → 97，用户上手时间从"找 README" 降到"看 CLAUDE.md"。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **93/100**，安全评定：CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 75 | 无 build/run/test 命令；无 prerequisites |
| skills/SKILL.md | 88 | 550 行超限；"appropriate" 模糊量词 |
| skills/simmer-judge-board/SKILL.md | 90 | "relevant" × 5 处模糊量词 |
| skills/simmer-judge/SKILL.md | 96 | "relevant" × 2 |
| skills/simmer-setup/SKILL.md | 98 | "some" 一处 |
| skills/simmer-generator/SKILL.md | 100 | Clean |
| skills/simmer-reflect/SKILL.md | 100 | Clean |
| .claude-plugin/plugin.json | 100 | Clean |

### 3.2 当时值得借鉴的模式

1. **Drop-in Replacement 设计** → judge 和 judge-board 有完全相同的输入/输出接口，orchestrator 通过 `JUDGE_MODE` 变量切换，自身无需修改。根本原因：解耦评判策略和编排逻辑，评判策略可独立演进。借鉴：任何有"策略可替换"需求的工作流都可以用这种 drop-in 模式。
2. **ASI（改进建议索引）作为跨轮通信协议** → judge 输出 ASI 时不只是评分，而是给 generator 的"具体改哪里"的索引，generator 下一轮基于 ASI 有方向地改进而不是随机探索。根本原因：跨 skill 传递结构化上下文比传递自由文本更可控。借鉴：两个 skill 之间如果需要传递"下一步怎么做"的信息，定义一个结构化的"工作令"格式（类似 ASI），而不是让下游 skill 自己理解上游输出。
3. **simmer-generator 和 simmer-reflect 得满分** → 这两个 skill 文件完全没有模糊量词，有完整的 input/output 描述，是教科书级别的 SKILL.md 写法。根本原因：职责极其明确（generator 只做"生成候选"，reflect 只做"记录轨迹"），职责窄才能写得精确。借鉴：单职责是写出满分 SKILL.md 的前提条件。
4. **plugin.json manifest 完整性** → manifest 中的所有 5 个 subskill 引用都经过验证，无孤立引用，无缺少文件。根本原因：作者在开发时保持了 manifest 和文件系统的同步更新。借鉴：每次新建 SKILL.md 时同步更新 plugin.json，别等到批量审计时才发现缺项。
5. **集成测试场景文件** → `tests/integration/simmer-scenario.md` 定义了可执行的集成测试场景，说明作者为这个插件设计了 NL-TDD 工作流。根本原因：插件开发也需要回归测试防止迭代中的退化。借鉴：为每个核心 skill 写至少一个 NL-TDD 场景文件。

### 3.3 当时的缺陷

1. **CLAUDE.md 无安装命令、无测试命令、无 prerequisites** → 根本原因：把 CLAUDE.md 写成了纯概念文档（概览 + 表格 + 流程图），忘记了它同时是"新用户第一次看到的使用指南"。结果：新用户不知道怎么安装，不知道怎么运行测试，不知道有没有前置要求。自查：我的 echo-sleuth/CLAUDE.md 也缺少 `claude plugin install` 命令，但有 architecture 说明。
2. **skills/SKILL.md 550 行超限** → 根本原因：作者把三种运行模式（multi-agent / single-agent / hybrid evaluator）都塞进同一个 orchestrator，保持了概念完整性，但违反了 NLPM R05 规则（500 行上限）。自查：我的 echo-sleuth skills 文件目前都在 200 行以内，未来扩展时需要注意。
3. **simmer-judge-board 中 5 处 "relevant" 模糊** → 根本原因：写"评委应该做什么"时用了主观词，没有列举具体文件类型或条件。这会导致不同评委实例对"哪些工件 relevant"理解不一致。自查：我的 claude-for-legal skills 中有多处 `relevant` 用法，需要逐一检查是否给出了具体标准。

### 3.4 当时的优化机会

1. **CLAUDE.md 补充 3 个章节**（安装、测试、prerequisites）→ 预计 10 行，NL 得分从 75 → 90+。
2. **skills/SKILL.md 提取 single-agent mode 到独立引用文件** → 文件降到 515 行以内，满足 R05。
3. **simmer-judge-board 中 5 处 `relevant` 替换为文件类型列表** → 评委行为更一致，judge-board 得分从 90 → 97+。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 无安装命令 | `grep -n "install\|claude plugin" CLAUDE.md` | **仍存在**：CLAUDE.md 第 70-73 行提到 simmer 是"test-kitchen 生态的一部分，可独立安装"，但没有给出具体的 `claude plugin install` 命令行 | 新用户上手体验未改善 |
| "relevant" × 5 在 judge-board | `grep -n "relevant" skills/simmer-judge-board/SKILL.md` | **仍存在**：L53/L125/L267/L271/L275 的 5 处 `relevant` 全部保留原文 | 评委行为仍缺乏确定性标准 |
| CLAUDE.md 目录树漏 simmer-judge-board | `grep -n "simmer-judge" CLAUDE.md` | **已修复**：L88 `simmer-judge/`，L90 `simmer-judge-board/` 均出现在目录树中 | 作者主动维护了文档准确性 |

### 4.2 架构演进

当前 HEAD 和 audit 快照结构基本一致（6 个 SKILL.md，无新增 subskill）。主要变化是 CLAUDE.md 的目录树修正（加入了 simmer-judge-board/）。说明作者在维护文档准确性上有意识，但还没有系统性地修复 NLPM 扣分项（安装命令、模糊量词）。

### 4.3 新增的可学习模式

暂无实质性新模式。CLAUDE.md 目录树修正是维护类改动，不产生新的可学习模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **Plugin.json manifest 同步**：我的 echo-sleuth 没有 manifest 不一致的问题（项目规模较小）。
2. **Skill 文件不超长**：echo-sleuth 的所有 SKILL.md 都在 200 行以内，远未触及 500 行上限。
3. **Integration test 文件**：echo-sleuth 有 `tests/` 目录，和 simmer 的 `tests/integration/` 同样重视可测试性。

### 5.2 挑战 / 验证

这次案例**验证**了我的一个认知：CLAUDE.md 里写安装命令、测试命令和 prerequisites 是机械性工作，但对用户（和 NLPM 评分）影响很大。simmer 因为这 3 件事扣了 -25 分（-10 -5 -5 -5），这些都是几分钟能修复的问题，但文件里就是没有。教训：完成一个插件的"最后一公里"（让新用户能在 5 分钟内用起来），是单独的工作，不会自然完成。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 CLAUDE.md 是否有安装命令（对应本案例 CLAUDE.md -25 分问题）
grep -n "plugin install\|npm install\|pip install" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/CLAUDE.md 2>/dev/null
```
没有命中（应该有）后怎么办：在 CLAUDE.md 中加 "## 安装" 章节，写 `claude plugin install echo-sleuth@MarkQWu` 或等效命令。

```bash
# 检查我的 orchestrator skill（如果有）是否超过 500 行
find /tmp/my-repos -name "SKILL.md" -exec wc -l {} \; | sort -rn | head -10
```
命中（>500 行）后怎么办：拆分超长章节到 references/ 目录，在主文件中用 `loads:` 引用。

```bash
# 检查 simmer-judge-board 类似的 "relevant" 积累
grep -rn "relevant" /tmp/my-repos/MarkQWu-claude-for-legal/*/skills/*/SKILL.md 2>/dev/null | wc -l
```
命中多于 5 处后怎么办：逐文件排查，替换为"具体条件"描述。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 引入类似 simmer 的 "ASI 格式"——在 extract 和 lessons skill 之间定义结构化的"知识令"（包含：发现了什么、来源是哪个 session、置信度），而不是传递自由文本。
   - **为何可行**：echo-sleuth 的 extract → lessons 工作流也是链式的，目前 extract 输出自由文本，lessons 再归纳。如果定义结构化中间格式，lessons 的归纳质量会更稳定。
   - **第一步**：在 `skills/experience-synthesis/SKILL.md` 中定义一个"知识令"JSON schema（`{type, content, source_session, confidence}`），让 extract 输出时遵循这个格式。1 小时内可完成初版。

2. **想法**：为 claude-for-legal 的某个 skill 写 NL-TDD 测试场景文件，参考 simmer 的 `tests/integration/simmer-scenario.md`。
   - **为何可行**：claude-for-legal 目前无集成测试场景，每次改动都要手动测试。参考 simmer 的格式可以快速写出第一个测试。
   - **第一步**：创建 `tests/regulatory-scenario.md`，定义一个"给定一段监管文本，gap-surfacer 应该输出什么格式"的测试场景。30-45 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 2389-research/simmer 的核心目的**：多轮迭代精炼任意工件（prompt/代码/配置），每轮标准化评判 + 方向性改进建议。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 都是面向 AI 开发者的工具；都有 orchestrator + subskill 结构思想 | 目的不同（历史挖掘 vs 工件精炼） | 中（架构模式可借鉴） |
| MarkQWu/drama-workshop-skills | 低 | drama-workshop 本身就是一个"反复精炼剧本"的工具，和 simmer 的精炼意图类似 | simmer 通用；drama 专域 | 中 |
| MarkQWu/claude-for-legal | 无 | — | 目的完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 无安装/测试/prerequisites 章节 | `grep -cn "install\|test" CLAUDE.md` | echo-sleuth 有架构说明，但无安装命令（确认） | 高 |
| "relevant" 模糊量词积累 | `grep -rn "relevant" skills/*/SKILL.md` | echo-sleuth 2 处；claude-for-legal 多处（已见上节）| 中 |
| Orchestrator skill 潜在超长风险 | `wc -l skills/*/SKILL.md \| sort -rn` | echo-sleuth 当前最大 skill < 200 行（未触发）；drama-workshop-skills/short-drama/SKILL.md 需检查 | 低（当前） |

**命中后的具体行动建议**：
- `echo-sleuth/CLAUDE.md` → 加 "## 安装" 一节（`claude plugin install echo-sleuth@MarkQWu`）+ "## 测试" 一节 + "## Prerequisites" 一节（无前置依赖的话写"None"）→ 10 分钟
- `claude-for-legal/*/skills/*/SKILL.md` 中的 `relevant` → 按文件逐一替换，每个 5 分钟

### 8.3 别人的更优方案

1. **领域**：Drop-in Replacement 的 skill 设计
   - **本案例做法**：judge 和 judge-board 接口相同，由 JUDGE_MODE 变量切换，orchestrator 代码无需修改。
   - **我的项目现状**：echo-sleuth 的 recall 和 extract 功能目前是独立命令，没有"可互换策略"的设计。
   - **如何借鉴**：如果未来 echo-sleuth 要支持"深度分析"和"快速概览"两种模式，可以设计两个接口相同的 skill（`recall-deep` / `recall-lite`），靠参数切换，上层 orchestrator 不用改。

2. **领域**：Reflect skill 专门记录迭代轨迹
   - **本案例做法**：`simmer-reflect/SKILL.md` 负责记录"当前最佳版本"和"改进历史"，防止迭代中越改越差且无法回溯。
   - **我的项目现状**：echo-sleuth 的 extract 命令把知识写到 memory 文件，但没有"版本轨迹"概念；drama-workshop-skills 的分镜迭代也没有轨迹记录。
   - **如何借鉴**：在 drama-workshop-skills 的关键工件（如 creative-plan.md）生成时加一个"版本快照 + 本轮改了什么"的记录步骤，类似 reflect skill 的职责。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安装脚本的用户体验
  - **我的做法**：drama-workshop-skills 有 `install.sh`（macOS/Linux）和 `install.ps1`（Windows），支持跨平台安装
  - **本案例做法**（弱在哪）：simmer CLAUDE.md 没有安装命令，用户只能翻 README 才能找到 `claude plugin install`
  - **意义**：安装入口更低摩擦，用户体验优于 simmer。

---

## 八、术语表

### <a name="ASI"></a>ASI（改进建议索引，Actionable Synthesis Index）
> Simmer 自定义的跨轮通信协议。每轮结束时，judge skill 产出一个 ASI：不只是评分，而是"下一轮 generator 应该重点改进哪几个方面"的结构化索引。ASI 保证了迭代方向的连续性——generator 不会随机探索，而是基于 ASI 有针对性地改进。类比：代码 review 的"action items"，而不是泛泛的"代码可以更好"。

### <a name="Drop-in-Replacement"></a>Drop-in Replacement（可插拔替换）
> 设计两个 skill，让它们有完全相同的输入/输出接口，靠一个配置变量（如 `JUDGE_MODE`）切换，上层调用者（orchestrator）无需修改。类比：换一个 USB 设备——接口一样（USB-A），里面实现不同（U 盘 vs 键盘），主机不用关心里面是什么。优势：评判策略可以单独演进，单独测试，而不影响整个 orchestrator 的工作流。

### <a name="Orchestrator"></a>Orchestrator（编排器）
> 在多 skill 架构中，负责"按顺序调用子 skill 并传递上下文"的主 skill。Simmer 的 orchestrator 就是 `skills/simmer/SKILL.md`——它不做实质的生成或评判工作，只负责"先调 setup，再循环调 generate-judge-reflect"这件编排工作。类比：项目经理，不写代码，但负责告诉每个人该做什么、什么时候做。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部 `---` 包住的 YAML 块。CLAUDE.md 如果没有 frontmatter，NLPM 会把它当成普通文档而非 NL artifact，无法评分、无法发现其中的 `name` 和 `description` 字段。
