# ooiyeefei/ccc — 学习案例

**仓库**：https://github.com/ooiyeefei/ccc
**Stars**：405（exemplar_published 覆盖星数门槛）| **来源**：upstream（exemplar_published，SECURITY CLEAR）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-23（基于当前 HEAD）
**主题标签**：`examples-driven`, `cross-reference`, `template-design`, `manifest-discipline`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

ooiyeefei/ccc（Community Claude Code）是一个多插件 skills 集合，面向产品管理、MVP 孵化、UAT 测试、Excalidraw 图表、Pitch Deck 制作、Google Analytics 配置等多个垂直场景。作者 ooiyeefei 是新加坡开发者，曾在多个 AI 产品公司工作。仓库是个人实践集合，而非公司产品。

关键事实：
- 当前 HEAD 包含 **8 个插件** + **18 个独立 skill**（审计时仅有 4 个独立 skill + 3 个插件）
- Excalidraw skill 和 product-management 插件的 gap-analyst agent 各得 100/100
- NLPM exemplar 例示 R04/R07/R08/R09/R12/R17 六条规则
- 安全状态：CLEAR（仅 medium/low 级别问题，无 Critical/High）

### 1.2 架构剖析

```
ooiyeefei/ccc/
├── commands/                    # 顶层独立命令（streak 系列 6 个）
│   ├── streak.md
│   ├── streak-new.md
│   ├── streak-switch.md
│   ├── streak-list.md
│   ├── streak-stats.md
│   └── streak-insights.md
├── plugins/                     # 8 个插件（每个自包含）
│   ├── agentic-toolkit/         # NEW（审计后新增）
│   ├── bash-safety-guard/       # NEW
│   ├── daily-chief/             # NEW（带 commands/ skills/ bin/ scripts/）
│   ├── deckling/                # 原有（PPTX 生成器）
│   │   ├── commands/deckling.md
│   │   ├── scripts/deckling_worker.py
│   │   └── skills/deckling-pptx.md  ← 已修复（Bug #1）
│   ├── mvp-launch/              # 原有（MVP 可行性检查）
│   ├── product-management/      # 原有（PM 工作流：landscape/gaps/prd/file/sync）
│   │   ├── agents/              # gap-analyst、prd-generator、research-agent
│   │   ├── commands/            # analyze、gaps、landscape、prd、file、sync
│   │   ├── hooks/hooks.json
│   │   └── skills/
│   ├── rethink-surveys/         # NEW
│   └── ...
└── skills/                      # 18 个独立 skill（审计时 4 个）
    ├── excalidraw/              # 原有（100/100）
    ├── streak/                  # 原有（Telegram bot + 打卡系统）
    ├── uat-testing/             # 原有
    ├── landing-page-gtm/        # 原有
    ├── google-analytics-setup/  # NEW（含 references/ 子目录）
    ├── htmldrop/                # NEW（含 references/ 子目录）
    ├── pitch-deck/              # NEW（含 references/ 子目录）
    ├── pitch-package/           # NEW
    ├── project-showcase/        # NEW（含 references/ 子目录）
    ├── secure-by-design/        # NEW（含 references/ 子目录）
    └── ... （共 18 个）
```

- **文件类型分布**：27 个原始 artifact（NLPM 审计范围），目前已扩展到约 60+ 个文件；6 个 agent 文件、8+ 个命令文件、18+ 个 SKILL.md
- **编排关系**：product-management 插件中有三层编排（command → agent → skill），其余插件多为平列
- **跨件契约**：产品管理工作流（`/pm:analyze` → `/pm:landscape` → `/pm:gaps` → `/pm:prd` → `/pm:file`）形成完整流水线，通过 `.pm/` 目录下的持久化 JSON 文件传递状态

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「描述即触发图 + 查找表替代散文 + 显式错误路径」——12 个触发短语在 description 中构建触发地图；所有评分、格式、错误处理都用表格/公式，不用散文
- **解决什么问题**：把产品经理的高频重复工作（竞品分析、功能 gap 识别、PRD 生成）变成可一键触发的 AI 工作流，且结果格式统一
- **Trade-off**：`references/` 子目录让主 SKILL.md 保持简短，但引入了「skill 目录复杂化」的维护成本
- **认知模型**：作者把 Claude 当做「需要明确地图的执行者」——不期望 Claude 理解语境，期望 Claude 按图索骥

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「触发地图 + 深度子目录分压」**

每个 skill 的 `description` 字段是一张触发地图（8-12 个具体用户短语），body 中复杂内容通过 `references/` 子目录拆分，主 SKILL.md 保持在 200-300 行以内，细节按需通过跨文件引用加载。

模式特征清单：
- 特征 1：description 字段列出 6-12 个具体用户短语，每个代表不同使用场景
- 特征 2：`references/` 子目录存放查找表、模板、案例，主 SKILL.md 只保留核心流程
- 特征 3：agent 文件中有 `<example>` 块，每个 example 覆盖不同触发场景（含非显眼短语）
- 特征 4：所有 output 用明确模板定义（含表头、示例行、占位符语法）
- 特征 5：edge cases section 覆盖「没有数据时怎么做」的正向和负向两类情况

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 工作流复杂、需要覆盖多种用户说法 | ✅ 高度适用 | 触发地图让 skill 在任何措辞下都能被调用 |
| 输出格式需要高度一致（如报告、PRD） | ✅ 适用 | 模板固定化输出格式 |
| 简单一次性 task（如「读一个文件」） | ❌ 过度工程 | 不需要 6-12 个触发短语和 references/ |
| 深度领域知识（如法律、医学） | ✅ 适用 | references/ 可以按主题拆分知识，主文件保持可读 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：触发地图 + 子目录分压 | ooiyeefei/ccc | 触发召回率高；主文件简洁；深度参考按需加载 | 目录结构复杂；references/ 维护成本高 |
| 备选 A：单一长文件 SKILL.md | 很多入门仓库 | 简单，一个文件搞定 | 易超过 500 行（R05），可读性差 |
| 备选 B：纯 agent 编排（无 skill） | 部分复杂工作流 | 每步灵活控制 | 触发机制完全依赖用户描述，召回不稳定 |

### 2.4 改进空间

1. **当前问题**：`plugins/product-management/.claude-plugin/plugin.json` 版本 `0.1.0` 但 SKILL.md 声明 `0.2.0`（Bug #2，持续）。**改进做法**：每次更新 SKILL.md version 时同步更新 plugin.json。**预期收益**：用户安装时 manifest 显示正确版本，避免误解。

2. **当前问题**：`/pm:review` 在 SKILL.md 文档中列为标准流程步骤（行 131），但 `commands/review.md` 不存在（Bug #3，持续）。**改进做法**：创建 `review.md` 实现 gap review 流程，或从文档中删除对 `/pm:review` 的引用。**预期收益**：消除文档-实现不一致，用户按文档操作不会触发「命令不存在」错误。

3. **当前问题**：顶层 6 个 `commands/streak-*.md` 全部缺少 `allowed-tools` 声明（-30 分累计）。**改进做法**：批量添加，如 `allowed-tools: [Read, Write]`。**预期收益**：每次运行不再需要用户手动批准工具，用户体验大幅提升。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 96/100，审计了 27 个文件。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| plugins/deckling/commands/deckling.md | 65 | 缺 allowed-tools；无步骤编号；无输出格式；引用缺失 skill |
| commands/streak-switch.md | 85 | 缺 allowed-tools；无空输入处理 |
| plugins/product-management/commands/prd.md | 90 | 无空输入处理 |
| commands/streak-list/stats/insights/new.md | 95 | 缺 allowed-tools |
| plugins/mvp-launch/skills/mvp-launch/SKILL.md | 96 | "appropriate" 模糊 |
| plugins/product-management/agents/research-agent.md | 98 | "relevant" 模糊 |
| 10+ 个文件 | 100 | — |

### 3.2 当时值得借鉴的模式

**1. R04：description 即触发地图**（`product-management/SKILL.md:2-3`）：12 个引号括住的具体用户短语，确保「分析产品」「研究竞品」「找功能 gap」「创建 PRD」「排序 backlog」等任意说法都能触发。

**2. R07：集成说明书**（`product-management/SKILL.md:163-174`）：PM 插件和 spec-kit 的边界用「WHAT + WHY vs HOW」+「流程图（GitHub Issue 作为 handoff）」完整描述，一段代替数段解释。

**3. R08：箭头定位查找表**（`excalidraw/SKILL.md:79-87`）：4 行表格替代「箭头怎么定位」的所有散文解释，Claude 查表、对号入座、完成——全程无需理解几何概念。

**4. R09：`<example>` 块覆盖非显眼触发**（`gap-analyst.md:4-13`）：第三个 example 用「Now that we know what Linear does, what are we missing?」这种追问式短语，与 agent 名称无关，只有 example 才能让这个短语触发正确 agent。

**5. R12+R17：output 模板 + edge cases 双保险**（`gap-analyst.md:120-203`）：output section 给出含示例行的完整表格模板；edge cases 覆盖 4 种情况，最后一条「所有 gap 都是已存在的」还告诉 Claude「要庆祝好的覆盖度」——连「没有问题」也是一个 case。

### 3.3 当时的缺陷

**1. deckling.md 引用不存在的 skill（Bug #1）**
- **根本原因**：`commands/deckling.md` 委托了一个「中间层」skill（`deckling-pptx.md`），但作者只实现了底层 Python 脚本（`deckling_worker.py`），忘了创建 NL 层的 skill 文件
- **自查**：我的仓库有没有命令文件引用了不存在的 skill？检查方式：`grep "skills/" commands/*.md | grep -v "^#"`

**2. product-management WINNING 公式与实现不一致（Cross-Component）**
- **根本原因**：SKILL.md 顶部 "WINNING = Pain × Timing × Execution"（3 因子乘法）是个「口号式公式」，但下面的实现（gap-analyst、gaps.md）是 6 个因子相加最高 60 分。两个版本同时存在，且矛盾
- **自查**：我的文档中有没有「顶部写了一个简洁版本，但实际实现更复杂」的情况？这是典型的「摘要漂移」

**3. streak 命令全部缺 allowed-tools（-30 分）**
- **根本原因**：作者把 streak 命令当做内部脚本写，没有按 Claude Code 的 skill 规范写 frontmatter
- **自查**：我的命令文件有没有缺 `allowed-tools` 的？缺失会导致每次执行都弹出工具批准提示

### 3.4 当时的优化机会

1. 创建 `plugins/deckling/skills/deckling-pptx.md`（修复 Bug #1）→ **已修复** ✓
2. 创建 `plugins/product-management/commands/review.md`（修复 Bug #3）→ **持续未修复**
3. 统一 WINNING 公式（SKILL.md 顶部 vs 实现）→ **未验证**
4. 批量给 streak 命令加 `allowed-tools` → **持续未修复**

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：deckling-pptx.md 不存在 | `ls plugins/deckling/skills/` | **已修复** ✓（`deckling-pptx.md` 已存在） | NLPM PR 或作者自行修复，deckling 命令现在可以正常运行 |
| Bug #2：plugin.json 版本 0.1.0 vs SKILL.md 0.2.0 | `grep '"version"' .claude-plugin/plugin.json` | **持续**（仍为 0.1.0） | 版本元数据漂移，misleading |
| Bug #3：/pm:review 命令不存在 | `ls plugins/product-management/commands/` | **持续**（无 review.md，6 个命令：analyze/gaps/landscape/prd/file/sync） | 文档描述的完整工作流仍有断点 |
| Quality：streak 命令缺 allowed-tools | `grep "allowed-tools" commands/streak*.md` | **持续**（6 个文件均无声明） | 用户每次运行 streak 命令都要手动批准工具 |

**一个修复，三个持续**——Bug #1 是最严重的功能性 bug（命令完全不可用），被修复；其余的均为次要或质量问题，持续存在。

### 4.2 架构演进

从审计时到当前 HEAD，仓库经历了显著扩展：

| 维度 | 审计时（2026-04-06） | 当前 HEAD |
|---|---|---|
| 独立 skills | 4 个（excalidraw、streak、uat-testing、landing-page-gtm） | 18 个（+14 个） |
| 插件 | 3 个（deckling、mvp-launch、product-management） | 8 个（+5 个：agentic-toolkit、bash-safety-guard、daily-chief、rethink-surveys 等） |

扩展方向分析：
1. **深度分压**：新 skill（google-analytics-setup、secure-by-design 等）都带有 `references/` 子目录，说明作者把 exemplar 中推荐的「深度子目录」模式彻底内化了
2. **垂直场景拓展**：从产品管理核心，扩展到 pitch deck、project showcase、安全设计等周边场景
3. **原有缺陷未修复**：扩展的同时，旧缺陷（version mismatch、缺失 review.md、streak allowed-tools）仍未清理，说明作者的精力在「建新的」而非「修旧的」

### 4.3 新增的可学习模式

**`references/` 子目录全面落地**：当前 HEAD 中所有新增 skill 都带有 `references/` 子目录（含 4-6 个 Markdown 参考文件）。这是本次 exemplar 报告推荐的 Worth Adopting 模式，作者在审计后完整落地。说明 NLPM exemplar 对这个仓库的作者有实际影响。

**bash-safety-guard 插件**：新增插件专注于「防止 Claude Code 运行危险 Bash 命令」，用 Hook 拦截所有 Bash 调用并比对白名单。这是安全增强的专用插件，设计思路值得学习。

---

## 五、校准

### 5.1 我已经在做对的

1. **references/ 子目录**：我的 `MarkQWu/graphify` 广泛使用 `references/` 子目录（`graphify/skills/kilo/references/`、`graphify/skills/codex/references/` 等），和 ooiyeefei/ccc 的模式完全一致
2. **触发短语列表**：我的 `gstack/spec/SKILL.md` 有 `triggers:` 字段（6 个具体短语），与 R04 方向一致
3. **agent 带 example 块**：我的仓库中有 agent 文件带 `<example>` 块

### 5.2 挑战 / 验证

**这次案例挑战了我的一个假设**：我以为「WINNING 公式 = Pain × Timing × Execution（3 因子）」这种口号式公式是「好的简化」。但 ooiyeefei/ccc 的缺陷恰恰说明：当文档中的简化版本和实际实现不一致时，用户（和 Claude）会被搞混。口号式公式不是简化，是漂移的种子——**要么用实际实现的公式，要么明确标注「概念性简化，非实现公式」**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令/skill 文件是否引用了不存在的其他文件
for f in /tmp/my-repos/MarkQWu-bureau/commands/*.md; do
  refs=$(grep -o 'skills/[a-zA-Z0-9_/-]*\.md' "$f" 2>/dev/null)
  for ref in $refs; do
    base=$(dirname "$f")
    if [ ! -f "$base/$ref" ] && [ ! -f "/tmp/my-repos/MarkQWu-bureau/$ref" ]; then
      echo "BROKEN REF in $f: $ref"
    fi
  done
done
# 命中后怎么办：立即创建缺失文件或修正引用路径
```

```bash
# 检查我的命令文件是否缺 allowed-tools
for f in /tmp/my-repos/MarkQWu-bureau/commands/*.md; do
  if ! grep -q "^allowed-tools:" "$f"; then
    echo "MISSING allowed-tools: $f"
  fi
done
# 命中后怎么办：根据 body 中实际使用的工具补充声明
```

```bash
# 检查文档中的公式/规则是否与实现一致（手动检查的入口）
grep -rn "=.*×\|formula\|WINNING\|formula" /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null | head -20
# 命中后怎么办：逐条确认「文档公式」是否与实际执行步骤一致，不一致则标注或修改
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `MarkQWu/graphify` 增加 edge cases section（仿 gap-analyst.md 的 R17 模式）
   - **为何可行**：graphify 的 skill 有多个「数据不存在」的边缘情况（图不存在、节点找不到、权限不足），目前处理方式不统一
   - **第一步**：在 `graphify/skills/claude/SKILL.md` 底部加一个 `## Edge Cases` section，列出 3-4 种异常情况和对应处理方式

2. **想法**：给 `MarkQWu/bureau` 添加「bash-safety-guard 类似机制」——PostToolUse hook 监控 Write 操作是否写入 `bureau.json`（核心配置），防止误修改
   - **为何可行**：bureau 的核心配置 `bureau.json` 一旦被错误写入，会影响整个会话的信任层判断
   - **第一步**：参考 ooiyeefei/plugins/bash-safety-guard/hooks/ 的结构，写一个检查 `bureau.json` 写入的 PostToolUse hook

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 ooiyeefei/ccc 的核心目的**：将产品管理、设计、测试等多个垂直场景的重复工作流打包成可一键触发的 Claude Code skill 集合

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | 多 skill 集合，有 references/ 子目录，面向专业工作流 | graphify 专注知识图谱领域，ccc 是通用 PM 工具集 | 高 |
| MarkQWu/gstack | 中 | 同样是多 skill 工具集，有触发短语设计 | gstack 更全面，ccc 更垂直 | 中 |
| MarkQWu/bureau | 低 | 都有多个协作 skill | bureau 专注会话记忆，ccc 专注工作流执行 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| streak 命令缺 allowed-tools | `grep -L "allowed-tools" commands/*.md` | `bureau/commands/` 11 个命令，**命中 0 个**（都有 allowed-tools） | 无需处理 |
| 文档公式与实现不一致 | 手动核对 | 未系统检查；graphify 的 schema 文档可能有此问题 | 中 |
| plugin.json version 与 SKILL.md 不同步 | `grep '"version"' plugin.json` vs `grep '^version:' SKILL.md` | 未检查 graphify 和 gstack 的版本同步 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/graphify` 中如有 `plugin.json`：对比版本号，不同步则更新
- 手动对照 graphify/skills/*/SKILL.md 中的描述性公式与实际步骤，消除「口号式规则」

### 7.3 别人的更优方案

1. **领域：`<example>` 块覆盖非显眼触发短语（R09）**
   - **本案例做法**：`gap-analyst.md` 的 example 3 用「Now that we know what Linear does, what are we missing?」——这个短语与 agent 名称无关，只有 example 才能让它触发正确 agent
   - **我的项目现状**：`MarkQWu/gstack` 的 agent 文件缺少 `<example>` 块，`MarkQWu/bureau` 的 agent 文件也缺少
   - **如何借鉴**：为 bureau 的 `auditor/agent.md` 加 2-3 个 `<example>` 块，覆盖用户不按「官方名称」问时的触发场景

2. **领域：references/ 子目录全面使用（Exemplar Worth Adopting）**
   - **本案例做法**：所有新 skill 都有 `references/` 子目录，每个子文件专注一个主题（verify-checklist、recommended-events、install-by-platform 等）
   - **我的项目现状**：`MarkQWu/graphify` 已用（每个 skill 有 8 个 references 文件），但 `MarkQWu/gstack` 的 skill 几乎无 references 子目录
   - **如何借鉴**：找 gstack 中最长的 SKILL.md，把超过 200 行的部分拆到 `references/` 子文件，主文件保持在 150 行内

### 7.4 反向：我的项目做得比他们好的地方

- **领域：allowed-tools 声明完整**：我的 `bureau/commands/` 11 个命令 **100%** 声明了 `allowed-tools`，而 ooiyeefei 的 6 个 streak 命令全部缺失。这说明我有系统化的 NL 规范执行习惯
- **意义**：用户运行 bureau 命令时不会遇到工具批准提示的干扰，体验更流畅；若被人审计，这是一个明确的优势点

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（如 `name`、`description`、`allowed-tools`）。Claude Code 读 SKILL.md 或命令文件时先解析 frontmatter，再读 body。

### <a name="allowed-tools"></a>allowed-tools
> frontmatter 中的白名单字段，声明该命令/skill 可以使用的 Claude 工具。未列出的工具被调用时会弹出用户批准提示。缺失此字段意味着所有工具调用都需要手动批准。

### <a name="edge-case"></a>edge case（边缘情况）
> 程序或流程中「正常输入」以外的特殊情况。在 NL 编程中，edge case section 告诉 Claude「当条件 X 不满足时该做什么」——比如「没有竞品数据」「用户没有提供必要参数」「所有 gap 都已存在」。不写 edge case 意味着 Claude 遇到这些情况时会自行发挥，输出不可预测。

### <a name="manifest"></a>manifest
> 项目的清单文件（如 `plugin.json`），声明插件包含哪些命令、skills、agents。manifest 中的版本号应与 SKILL.md 中的 `version:` 保持一致，否则安装时元数据会误导用户。
