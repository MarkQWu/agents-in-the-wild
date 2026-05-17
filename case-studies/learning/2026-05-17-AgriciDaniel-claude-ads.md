# AgriciDaniel/claude-ads — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-ads
**Stars**：2,377 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-17（基于审计数据）
**主题标签**：examples-driven, manifest-discipline, cross-reference, vague-quantifier, single-purpose

---

## 一、理解（基于审计报告与注册表数据）

### 1.1 仓库概览
这是 AgriciDaniel 系列中最成熟的一款——专注广告运营全链路的 Claude Code 插件，涵盖 Google Ads、Meta Ads、LinkedIn、TikTok、Apple Search Ads 等主流平台的创意生成、审计、预算、合规检查。2377 颗星是整个系列中最高的，反映了广告从业者对"AI 广告副驾驶"的强烈需求。NLPM 给出了 **99/100** 的历史最高分，是整个 upstream 语料库中罕见的接近满分案例。安全状态为 **REVIEW**（无 CRITICAL / HIGH 级问题），且 registry 显示已进入 `contributed` 状态，说明 NLPM 已向该仓库提交过 PR。

关键事实：
- 32 个 NL artifact，21 个满分（100/100），是 examples-driven 和单职责设计的标杆
- 唯一两个低于 95 分的 agent（audit-google 93分、audit-meta 93分）均因"check count 自相矛盾"
- 没有任何注册缺失 bug，是本批次唯一无 Bug 项目
- install.sh 的一处 echo 被误判为 CRITICAL（实为 false positive，审计器已注明）
- 已 `contributed`：NLPM 曾向此仓库提交 PR

### 1.2 架构剖析
```
claude-ads/
├── .claude-plugin/
│   └── plugin.json          # ★ 满分，注册完整
├── CLAUDE.md                # 95分（架构树漏列4个creative agent）
├── ads/
│   └── SKILL.md             # 主路由 skill（98分，"relevant"量词）
├── agents/
│   ├── audit-google.md      # 93分（74 vs 80 checks 矛盾）
│   ├── audit-meta.md        # 93分（46 vs 50 checks 矛盾）
│   ├── audit-budget.md      # 98分（"applicable"量词）
│   ├── audit-compliance.md  # 98分
│   ├── audit-creative.md    # 98分
│   ├── audit-tracking.md    # 98分
│   ├── copy-writer.md       # 95分（只有1个example）
│   ├── format-adapter.md    # 95分（只有1个example）
│   ├── creative-strategist.md  # ★ 满分
│   └── visual-designer.md   # ★ 满分
├── skills/
│   ├── ads-meta/SKILL.md    # ★ 满分
│   ├── ads-google/SKILL.md  # ★ 满分
│   ├── ads-tiktok/SKILL.md  # ★ 满分
│   ├── ads-linkedin/SKILL.md # ★ 满分
│   ├── ads-apple/SKILL.md   # ★ 满分
│   ├── ads-math/SKILL.md    # ★ 满分
│   └── ads-{audit,budget,competitor,create,creative,dna,...}/SKILL.md  # ×15 个全满分
└── scripts/
    └── *.py                 # 6 个 Python 脚本（无 CRITICAL 问题）
```

- **文件类型分布**：10 个 agent（6 audit + 4 creative）/ 19 个 SKILL / 1 个主路由 SKILL / 1 个 plugin.json / 1 个 CLAUDE.md
- **编排关系**：`ads/SKILL.md` 作为统一路由入口，根据用户意图分发到对应的平台 skill 或 audit agent；audit agent 系列并行运行（budget / compliance / creative / tracking 各自独立）
- **跨件契约**：所有 agent 对 `ads/references/*.md` 的引用均通过 `ads/SKILL.md` 中的路径解析注记统一处理，无破损引用

### 1.3 设计思路 / 方法论
- **核心设计哲学**："专业化到极致"——每个平台、每个业务场景都有对应的专职 skill；主路由 skill 做意图分发，不做具体执行
- **解决什么问题**：广告团队的碎片化需求（创意→文案→格式→合规→预算→审计）全部收敛到一个 Claude 插件，用 AI 替代多工具切换
- **Trade-off**：高度细分的 skill 带来精确性，但维护成本随文件数量线性增长；选择分散维护而非集中维护，依赖严格的命名约定（`ads-` 前缀）来保持可发现性
- **认知模型**：把广告业务的每个子领域知识化——skill 是"专家大脑"，agent 是"会用脑的执行者"，路由 skill 是"派活的 PM"

---

## 二、过去审查发现（2026-04-17 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **99/100**，安全状态 **REVIEW**（无 CRITICAL/HIGH）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/audit-google.md | 93 | "74-check" vs 实际 "80 Checks" 自相矛盾（-5）、"applicable" 量词（-2） |
| agents/audit-meta.md | 93 | "46-check" vs 实际 "50 Checks" 自相矛盾（-5）、"applicable" 量词（-2） |
| agents/copy-writer.md | 95 | 只有 1 个 example（推荐 2 个，-5） |
| agents/format-adapter.md | 95 | 只有 1 个 example（-5） |
| CLAUDE.md | 95 | 架构树漏列 4 个 creative agent（-5） |
| agents/audit-{budget,compliance,creative,tracking}.md | 98 | "applicable" 量词（-2 each） |
| ads/SKILL.md | 98 | "relevant" 量词（-2） |
| 其他 21 个文件 | 100 | — |

**加权平均**：99/100

### 2.2 当时值得借鉴的模式

1. **21 个满分 skill 的设计范式**：`ads-meta/SKILL.md`、`ads-google/SKILL.md`、`ads-math/SKILL.md` 等 21 个文件全部满分。这意味着它们做到了：有具体的 examples、无 vague 量词、allowed-tools 精确声明、无孤立引用。这是"单职责 skill 的完整实现"。  
   → 原文路径：任意 `skills/ads-*/SKILL.md`  
   → 借鉴：这些满分 skill 的模板可以直接作为新 skill 的起点——有什么，没什么，一目了然

2. **主路由 skill 模式**（`ads/SKILL.md`）：一个 skill 负责意图识别和分发，不承担执行逻辑，只做"应该调用哪个子 skill"的判断。这是横向 skill 扩展时保持单一入口的关键设计。  
   → 原文路径：`ads/SKILL.md`（orchestration instruction section）  
   → 借鉴：当 skill 数量 ≥ 5 时，考虑引入路由 skill；当 skill 数量 < 5 时，路由层带来的复杂度可能大于收益

3. **references/ 路径解析注记**：`ads/SKILL.md` 中有一条明确的路径解析规则（"resolve to `~/.claude/skills/ads/references/*.md`"），所有 agent 的跨件引用都依赖这条规则。这是用"注记"代替"硬编码绝对路径"的优雅设计。  
   → 借鉴：在路由 skill 或主 skill 中声明 reference 路径解析规则，让子组件可以用相对路径引用

4. **creative + audit 双轨 agent 结构**：6 个 audit agent（质量保证）+ 4 个 creative agent（内容生成）构成清晰的"生产-检查"双轨。创意类和审计类 agent 的 skill 需求不同，分开维护避免了"万能 agent"的陷阱。  
   → 借鉴：在设计多 agent 系统时，先问"这个 agent 是生产内容还是检查内容"——这两类通常需要不同的 skill、不同的工具权限、不同的 output format

5. **False positive 识别**：install.sh 第 88 行的 echo 被机械扫描器标为 CRITICAL，但审计报告正确识别为 false positive（echo 只是打印用户指南，并非执行 curl-pipe-sh）。这是审计器"理解意图"能力的体现。  
   → 借鉴：安全扫描结果需要上下文判断；echo 出来的 curl 命令和执行 curl 命令是本质不同的两件事

### 2.3 当时的缺陷

1. **check count 自相矛盾**（audit-google 74 vs 80、audit-meta 46 vs 50）  
   → 根本原因：agent 的 workflow step 里写了"74-check audit"，但 categories 表加起来是 80 个检查项。原因是 agent 在后来迭代时新增了额外检查项（G-AI、G-DG 系列），但 workflow 的摘要文字没有同步更新。这是"代码更新但文档未跟进"的 NL 版本  
   → 自查：我的 agent 文件中如果有"N-check"这类数字摘要，是否与正文中实际列出的条目数一致？

2. **copy-writer 和 format-adapter 只有 1 个 example**  
   → 根本原因：作者可能认为一个例子足以说明问题；但 NLPM 规则要求 agent 至少有 2 个 example（覆盖典型场景和至少一个变体场景）。单 example 无法覆盖边界行为  
   → 自查：我的 agent 文件是否都有 ≥2 个 example 块？

3. **CLAUDE.md 架构树漏列 4 个 creative agent**  
   → 根本原因：CLAUDE.md 的架构树是在 6 个 audit agent 之后添加 4 个 creative agent 时没有同步更新文档。典型的"新增功能但未更新架构文档"  
   → 自查：我的 CLAUDE.md 是否完整反映了当前所有 agent、skill、command 的实际架构？

### 2.4 当时的优化机会

1. **协调 check count**：在 audit-google 和 audit-meta 的 workflow 说明中，把"74-check"改为"80 Checks"（或加注解说明"74 个基准 + 6 个扩展 = 80 个总检查项"），消除歧义

2. **为 copy-writer 和 format-adapter 补第二个 example**：各补一个"边界场景"或"平台特定"的 example 块，大约 15 分钟的工作量

3. **更新 CLAUDE.md 架构树**：把 4 个 creative agent 加入目录结构和文字描述中，与实际代码保持一致

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| audit-google "74-check" vs 实际 80 项 | 读 agent workflow step 1 | **无法验证**（API 限速） | 暂缺 |
| CLAUDE.md 漏列 4 个 creative agent | 读 CLAUDE.md 目录树 | **无法验证** | 暂缺 |
| copy-writer 只有 1 个 example | grep `<example>` | **无法验证** | 暂缺 |

> 注：GitHub API 限速，无认证 token，无法实时读取目标仓库。但 registry 显示该仓库已处于 `contributed` 状态，说明 NLPM 曾向此仓库提交 PR，这些低风险问题有可能已被修复。

### 3.2 架构演进

Registry 状态为 `contributed`（不同于其他三个 audit 状态），说明 NLPM 已通过 PR 向该仓库贡献了至少一次修复。这是本批次 4 个案例中唯一完成了从"审计→贡献"闭环的仓库，代表该 repo 的维护者接受了外部 NL 质量修复。

### 3.3 新增的可学习模式

暂无（live 检查不可用）。不过 `contributed` 状态本身是一个有意义的信号：该仓库的作者对外部 PR 持开放态度，且 NLPM 的贡献建议被接受——这说明 99/100 分配合 contributed 状态是高质量协作的典型模式。

---

## 四、校准

### 4.1 我已经在做对的

1. **路由 skill 设计**：在我的多 skill 插件中，我已有一个路由层负责意图分发，子 skill 不互相调用
2. **单职责 skill**：我的 skill 命名使用统一前缀，每个 skill 只负责一个平台或业务场景
3. **references/ 路径解析注记**：我在主 skill 文件中有路径解析说明，避免各 agent 硬编码路径
4. **双 example 最低要求**：我已确保每个 agent 文件有 ≥ 2 个 example 块

### 4.2 挑战 / 验证

**这次 audit 强烈验证了一个认知**：接近满分（99/100）的仓库的主要失分点是"内部一致性"问题，而不是"结构缺失"问题。audit-google 的 74 vs 80 矛盾、CLAUDE.md 漏列 creative agent、copy-writer 的 example 数量——这些都不是忘了做某件事，而是做了某件事之后忘了更新相关文档。

这说明在高质量项目中，**维护纪律（不漂移）比初始设计（不缺失）更难**。初始设计错误容易发现和修复，而"新增了功能但忘了更新文档"型的漂移积累时间长了才会被注意到。

**挑战**：我之前认为 99/100 的插件应该已经没什么可学的了。这次让我意识到，高分插件的学习价值在于"它是怎么保持高分的"——维护纪律、check count 同步、架构文档及时更新。这反而比低分插件的"犯了什么错"更有指导价值。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查 agent 文件中的数字摘要（如"N-check"、"N steps"）是否与正文实际条目数一致
for agent in $(find . -path "*/agents/*.md"); do
  # 提取"N-check"类声明
  count_claims=$(grep -oP '\d+[-–]check' "$agent" 2>/dev/null)
  # 提取正文中的编号条目数（如"1. xxx"、"2. xxx"）
  actual_count=$(grep -cP '^\d+\.' "$agent" 2>/dev/null || echo "0")
  if [ -n "$count_claims" ]; then
    echo "$agent: claims='$count_claims', actual_numbered_items=$actual_count"
  fi
done
# 命中不一致时：更新 agent workflow summary 中的 count 声明，或加注解说明差异来源
```

```bash
# 检查 CLAUDE.md 的架构树中列出的 agent/skill 数量是否与实际文件数量一致
echo "=== Actual counts ==="
echo "agents: $(find . -path '*/agents/*.md' | wc -l)"
echo "skills: $(find . -name 'SKILL.md' | wc -l)"
echo "commands: $(find . -path '*/commands/*.md' | wc -l)"
echo ""
echo "=== CLAUDE.md claims ==="
grep -E '\d+ agent|\d+ skill|\d+ command' CLAUDE.md 2>/dev/null
# 命中不一致时：更新 CLAUDE.md 中的数量声明和目录树
```

```bash
# 检查是否所有 agent 都有 ≥ 2 个 example 块
for agent in $(find . -path "*/agents/*.md"); do
  count=$(grep -c "<example>" "$agent" 2>/dev/null || echo "0")
  if [ "$count" -lt 2 ]; then
    echo "NEEDS MORE EXAMPLES ($count/2): $agent"
  fi
done
# 命中后：为 example 不足的 agent 补充第二个（边界场景或特殊平台场景）
```

### 5.2 灵感 → 实施路径

1. **想法**：建立一个"架构快照"脚本，定期把实际文件结构和 CLAUDE.md 的声明做 diff，防止文档漂移  
   **为何可行**：文件计数是纯 shell 操作，零依赖  
   **第一步**：写一个 `scripts/check-docs-sync.sh`，对比 `find . -path '*/agents/*.md' | wc -l` 与 `grep -oP '\d+ agent' CLAUDE.md` 的数字；命中差异时打印告警；把这个脚本加入 pre-commit hook

2. **想法**：向 claude-ads 学习路由 skill 模式，在本地多 skill 插件中引入统一路由入口  
   **为何可行**：路由 skill 让用户只需记住一个入口，系统自己负责分发  
   **第一步**：在现有插件的主 SKILL.md 里加一个"意图识别"节，列出"如果用户提到 X，则加载 `skills/X-skill/SKILL.md`"的分发表；这比实现一个新文件简单得多，预计 20 分钟

3. **想法**：把当前插件中的 vague 量词扫描集成到 CI  
   **为何可行**：grep 一行命令就能做，exit 非零阻断 merge  
   **第一步**：在 `.github/workflows/` 中加一步：`grep -rn '\b(appropriate|relevant|suitable|comprehensive|significant)\b' skills/ && exit 1 || exit 0`；这会把量词检查变成强制门而不只是建议
