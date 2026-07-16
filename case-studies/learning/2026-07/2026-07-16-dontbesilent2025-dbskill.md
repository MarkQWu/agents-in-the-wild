# dontbesilent2025/dbskill — 学习案例

**仓库**：https://github.com/dontbesilent2025/dbskill
**Stars**：N/A（exemplar_published）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-16（基于当前 HEAD）
**主题标签**：`security-gate`, `router-channels`, `experience-accumulation`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
dbskill 是中国创业者内容创作者 dontbesilent（@dontbesilent2025）构建的「商业诊断工具箱」Claude Code 插件，把作者多年来在社交媒体上累积的**4176 个知识原子**、**奥派经济学**视角和商业诊断经验系统化为 28 个专业技能，覆盖商业诊断、内容创作（小红书标题、微信 HTML、脚本流程）、行动设计、决策框架、目标追踪等。当前版本 2.17.10，从审计时的 12 skill 成长到今天的 28 skill，知识库仍在快速扩充。

- **创建时间**：2025 年
- **作者背景**：活跃的中文创作者/商业咨询师，推文内容积累成知识原子
- **获取方式**：通过 `marketplace.json`（已从 plugin.json 迁移）
- **生态位置**：中文内容创作 + 奥派商业诊断，垂直领域罕见

### 1.2 架构剖析

```
dontbesilent2025/dbskill
├── skills/
│   ├── dbs/SKILL.md             ← 主路由 skill（派发入口）
│   ├── dbs-diagnosis/SKILL.md   ← 商业诊断
│   ├── dbs-benchmark/SKILL.md   ← 基准对比
│   ├── dbs-deconstruct/SKILL.md ← 拆解分析
│   ├── dbs-action/SKILL.md      ← 行动设计
│   ├── dbs-content/SKILL.md     ← 内容创作
│   ├── dbs-content-system/SKILL.md ← 内容系统（新增）
│   ├── dbs-hook/SKILL.md        ← 钩子/引流
│   ├── dbs-xhs-title/SKILL.md   ← 小红书标题
│   ├── dbs-wechat-html/SKILL.md ← 微信 HTML（新增）
│   ├── dbs-script-flow/SKILL.md ← 脚本流程（新增）
│   ├── dbs-chatroom/SKILL.md    ← 聊天室（新增）
│   ├── dbs-chatroom-austrian/SKILL.md ← 奥派聊天室
│   ├── dbs-slowisfast/SKILL.md  ← 慢即是快
│   ├── dbs-ai-check/SKILL.md    ← AI 自检
│   ├── dbs-bridge/SKILL.md      ← 过渡/桥接（新增）
│   ├── dbs-decision/SKILL.md    ← 决策（新增）
│   ├── dbs-goal/SKILL.md        ← 目标追踪（新增）
│   ├── dbs-good-question/SKILL.md ← 好问题（新增）
│   ├── dbs-learning/SKILL.md    ← 学习（新增）
│   ├── dbs-report/SKILL.md      ← 报告（新增）
│   ├── dbs-resonate/SKILL.md    ← 共鸣（新增）
│   ├── dbs-restore/SKILL.md     ← 恢复（新增）
│   ├── dbs-save/SKILL.md        ← 保存（新增）
│   ├── dbs-skill-cleaner/SKILL.md ← Skill 清理（新增）
│   ├── dbs-spread/SKILL.md      ← 传播（新增）
│   ├── dbs-agent-migration/SKILL.md ← Agent 迁移（新增）
│   └── dbs-update/SKILL.md      ← 更新
├── .claude-plugin/
│   └── marketplace.json         ← 取代了原 plugin.json
├── .github/workflows/           ← CI/CD（release, ci, deploy-pages）
└── README.md + README.en.md
```

- **文件类型分布**：28 个 SKILL.md / 1 个 marketplace.json / 3 个 GitHub Actions workflow / 知识库原子（在各 skill 的知识包引用中）
- **编排关系**：`dbs/SKILL.md` 是路由 skill，用户发起请求后由 dbs 判断转发到哪个子 skill。`dbs-chatroom-austrian` 可从 `dbs-diagnosis` 和 `dbs-deconstruct` 条件性跳转，不在主菜单中。
- **跨件契约**：各 skill 通过 `知识库/Skill知识包/` 路径引用知识原子文件，形成「skill → 知识原子」的二层结构。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「[经验积累侧车](#经验积累侧车)」——把作者个人的推文/文章沉淀为结构化的「知识原子」，再用 skill 来激活这些原子。Skill 本身是检索/激活层，知识密度在原子文件里。
- **解决什么问题**：高质量内容创作和商业决策需要大量领域经验，而这些经验散布在作者的历史推文中，难以系统调用。
- **Trade-off**：skill 数量快速扩张（12→28），但每个 skill 的功能相对单薄；路由 skill 是单点——如果 `dbs/SKILL.md` 解析错误，整个工具箱都会失灵。
- **认知模型**：把 AI skill 视为「个人知识外脑的激活接口」，每个 skill 对应作者在某个主题上的系统化认知，而非通用功能。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「单路由 + 知识原子二层结构」**——顶层是一个路由 skill（`dbs`），承接所有用户请求并派发到 27 个专业 skill；每个专业 skill 内部引用「知识原子」文件作为知识来源。

模式特征清单：
- 特征 1：统一入口（`/dbs`）+ 按意图派发，用户无需记住 28 个 skill 名
- 特征 2：所有 skill 都以 `dbs-` 前缀命名，视觉上自成体系
- 特征 3：路由 skill 维护菜单表，新增 skill 需同步更新路由表
- 特征 4：知识原子是独立文件，skill 是激活层（分层解耦）
- 特征 5：敏感/专业 skill（如 dbs-chatroom-austrian）通过间接跳转而非主菜单暴露

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人知识库 + 内容创作系列 | ✅ 高度适用 | 路由入口统一，用户学习成本低 |
| 需要快速扩展 skill 数量的项目 | ✅ 适用 | `dbs-` 命名体系天然支持批量增加 |
| 多人协作维护 | ⚠️ 风险 | 路由 skill 是单点，多人修改易产生冲突 |
| 没有明确知识积累的通用工具 | ❌ 不适用 | 没有知识原子，路由层价值减半 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单路由+知识原子（本仓库） | dontbesilent/dbskill | 统一入口，个人品牌强 | 路由是单点，skill 同质化风险 |
| 无路由平铺 | czlonkowski/n8n-skills | 每个 skill 独立，无单点 | 用户需自行选择 skill |
| 多 plugin 聚合 | ccplugins/awesome-claude-code-plugins | 功能最丰富 | 无统一约定，质量参差 |

### 2.4 改进空间
1. **当前问题**：路由 skill 是单点故障 **改进做法**：为 `dbs/SKILL.md` 加一个 fallback 机制：当无法匹配意图时，展示完整菜单 + 引导用户描述具体需求 **预期收益**：减少用户卡住的概率
2. **当前问题**：marketplace.json 取代了 plugin.json，但不兼容标准 Claude Code plugin loader **改进做法**：保留 `marketplace.json` 的同时，添加标准 `plugin.json` 包装层 **预期收益**：兼容标准 `claude plugin install` 流程
3. **当前问题**：28 个 skill 的 `dbs-` 前缀虽统一，但功能边界模糊（`dbs-content` vs `dbs-content-system`）**改进做法**：在路由 skill 的菜单表中加二级分类标签（如「内容类」「诊断类」「工具类」）**预期收益**：路由决策更精确

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-13 当时得分 **90/100**（12 skill，SECURITY REVIEW）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/dbskill-upgrade/SKILL.md | 78 | 内嵌破坏性脚本、非标准 frontmatter、仅中文描述 |
| skills/dbs-ai-check/SKILL.md | 86 | 失效的 `/文风分析` skill 引用 |
| skills/dbs-benchmark/SKILL.md | 90 | "合理时间内" 模糊量词 |
| skills/dbs-diagnosis/SKILL.md | 90 | "合适的时机" 模糊量词 |
| skills/dbs-xhs-title/SKILL.md | 90 | 无显著问题 |
| skills/dbs/SKILL.md | 91 | chatroom-austrian 未在编号菜单中 |
| skills/chatroom-austrian/SKILL.md | 93 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **路由菜单设计** → 为什么好：`dbs/SKILL.md` 通过编号菜单收纳所有诊断入口，用户只需记 `/dbs`，不需要记 11 个子 skill 名。原文：`skills/dbs/SKILL.md` 路由表 → 借鉴：当 skill 数量 > 5 时，考虑用路由 skill 聚合
2. **知识原子与 skill 解耦** → 为什么好：知识内容（4176 个知识原子）独立于 skill 逻辑，skill 更新时知识原子不需要改，知识原子更新时 skill 也不需要改。原文：`知识库/Skill知识包/` 目录结构 → 借鉴：把可复用的知识内容从 SKILL.md 里提取到独立文件
3. **`dbs-` 命名前缀体系** → 为什么好：所有 skill 统一前缀，视觉识别度高，且 bash 通配符可批量操作。原文：所有 skill 目录名均以 `dbs-` 或 `dbs` 开头 → 借鉴：同一 plugin 的所有 skill 用统一前缀命名

### 3.3 当时的缺陷
1. **问题**：`skills/dbskill-upgrade/SKILL.md` 中嵌入了 `rm -rf ~/.claude/skills/dbs*` + `git clone` + 文件写入三件套（3 个 HIGH 安全发现）**根本原因**：作者为了让 Claude 自动帮用户更新 skill，把自动更新器的完整逻辑写进了 skill。这是一个「功能上追求方便，安全上完全失控」的设计——如果仓库被攻陷，任何人运行 `/dbskill-upgrade` 都会自动拉取恶意代码并写入用户的 `~/.claude/skills/` **自查**：我有没有在 skill 里嵌入 `git clone`、`curl` + 写文件的组合？
2. **问题**：`dbs-ai-check/SKILL.md` 引用了不存在的 `/文风分析` skill **根本原因**：skill 在迭代中被删除或重命名，但引用它的 skill 没有同步更新 **自查**：我的 skill 中的跨 skill 引用是否有定期校验？
3. **问题**：`dbskill-upgrade` 的 description 字段只有中文，其他 skill 都是双语 **根本原因**：这个 skill 是后期快速添加的，没有遵循已有的双语约定 **自查**：我新增 skill 时是否检查已有 skill 的约定？

### 3.4 当时的优化机会
1. 修复 `dbs-ai-check` 中的死引用 `/文风分析`（最低风险 PR，仅删除一行）
2. 修复 `scripts/record-demo.sh` 中的 sed 注入风险
3. 将 `dbskill-upgrade` 中的自动更新逻辑改为「显示 diff + 等待用户确认」而非直接执行

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| dbskill-upgrade HIGH 安全风险（rm -rf + git clone + 写文件） | `ls skills/dbskill-upgrade/` | **已修复（整体删除）** — 目录不存在 | 作者选择最彻底的修法：删除整个危险 skill |
| `/文风分析` 死引用 | `grep -rn "文风分析" skills/` | **已修复** — grep 0 命中 | 引用被清理，或相关 skill 已重命名 |
| 模糊量词（合理时间内/合适的时机） | `grep -n "合理\|合适" skills/dbs-benchmark/SKILL.md skills/dbs-diagnosis/SKILL.md` | **需进一步验证**（现有文件内容未完整读取） | 历史快照中存在，当前状态待确认 |

### 4.2 架构演进
从 12 skill（v当时）到 28 skill（v2.17.10），增加了 16 个 skill，涵盖领域：

- **工具类**（维护类）：`dbs-save`、`dbs-restore`、`dbs-skill-cleaner`（从危险的 dbskill-upgrade 拆分为更安全的维护工具）
- **内容深化类**：`dbs-script-flow`、`dbs-wechat-html`、`dbs-content-system`、`dbs-spread`、`dbs-resonate`
- **决策/目标类**：`dbs-decision`、`dbs-goal`、`dbs-good-question`、`dbs-learning`
- **系统工具类**：`dbs-agent-migration`、`dbs-bridge`、`dbs-report`
- **Manifest 迁移**：`plugin.json` → `marketplace.json`（自定义格式）

这说明作者在删除危险的 `dbskill-upgrade` 后，把维护/更新的需求拆分成了更小粒度的安全工具（save、restore、skill-cleaner），这是一个成熟的安全架构演进。

### 4.3 新增的可学习模式
- **`dbs-skill-cleaner` 模式**：专门用一个 skill 处理「清理 skill 目录」的需求，而不是在升级 skill 里内联 `rm -rf`。这是对审计反馈的直接响应——把危险的批量删除操作拆出来成为一个有名字、有说明的独立 skill，用户调用时知道自己在做什么，而不是由升级脚本暗中执行。

---

## 五、校准

### 5.1 我已经在做对的
1. **gstack/gstack-upgrade/SKILL.md** 包含 `git clone + rm -rf` 组合（类似 dbskill-upgrade），但我应当意识到这是同类风险
2. **bureau 没有任何自动安装/升级 skill**，从一开始就回避了这个安全反模式
3. **gstack 的所有 skill 都有双语约定意识**（不存在仅中文 description 的情况）

### 5.2 挑战 / 验证
- **这次案例挑战了我的假设**："把自动升级逻辑写进 skill 是合理的，因为方便用户。" — dbskill-upgrade 的 HIGH 安全评级清楚地证明：在 skill 中内嵌 `git clone + rm -rf + cp 到系统目录` 是供应链攻击的直接向量，方便性不能成为安全豁免的理由。
- **验证的认知**：修复安全问题的最佳方法有时是删除功能，而不是修复它。dbskill-upgrade 被整体删除、拆分为更小的安全工具，是更好的架构选择。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中是否有 git clone + 写文件 的危险组合
grep -rn "git clone\|rm -rf\|cp -r.*\.claude" \
  /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/ \
  --include="SKILL.md"
```
**命中后**：检查该 skill 是否在用户明确确认前就执行了 `git clone + rm -rf` 的组合。如果是，考虑拆分为 "检查更新" + "用户确认" + "执行" 三步，或者删除自动安装逻辑，改为显示手动操作指令。

```bash
# 检查跨 skill 引用是否都有对应的实际 skill 文件
grep -rn "^- /\|^\`/\|called via /\|invoke /\|run /" \
  /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/skills/ \
  --include="SKILL.md" | grep -oP '(?<=/)[a-z][a-z-]+' | sort -u
```
**命中后**：对每个提取出的 skill 名，检查对应的 `skills/<name>/SKILL.md` 是否存在。

### 6.2 灵感 → 实施路径

1. **想法**：把 `gstack-upgrade/SKILL.md` 中的 `git clone + rm -rf` 改为安全的三步流程
   - **为何可行**：gstack 的升级 skill 与 dbskill-upgrade 有完全相同的安全反模式（确认：`gstack/gstack-upgrade/SKILL.md:142`）
   - **第一步**：在 `gstack-upgrade/SKILL.md` 中添加「Step 0: 显示当前版本和将要安装的版本，等待用户确认」，把 `rm -rf` 操作后移到 `Step 3` 且必须用户二次确认；约 20 分钟

2. **想法**：给 bureau 和 gstack 添加 cross-skill 引用校验检查
   - **为何可行**：dbskill 的死引用 bug 根本原因是没有自动校验机制
   - **第一步**：写一个 10 行 Python 脚本，提取所有 SKILL.md 中的 `/skill-name` 引用，对比 `skills/` 目录下的实际 skill 名，输出不匹配项；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 dontbesilent2025/dbskill 的核心目的**：把个人积累的商业诊断+内容创作经验系统化为 Claude Code skill 集，提供统一入口

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是以个人/团队工作流为中心的 skill 集，都有升级维护类 skill | gstack 更关注 iOS 开发流程，dbskill 更关注商业诊断和内容创作 | 高 |
| MarkQWu/bureau | 中 | 都有知识积累的概念 | bureau 是结构化知识库系统，dbskill 是非结构化知识原子激活 | 中 |
| 其余仓库 | 低/无 | — | 领域差异大 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 中嵌入 git clone + rm -rf + 写系统目录 | `grep -rn "git clone" gstack/gstack-upgrade/SKILL.md` | **gstack 命中**：第 142 行有 `git clone --depth 1 https://github.com/garrytan/gstack.git`，第 146/176/184/186/192 行有多处 `rm -rf` | 高 |
| 跨 skill 引用死链 | `grep -rn "/[a-z]" gstack/ --include="SKILL.md"` | 需要人工抽查，未发现明显死链 | 待验证 |
| 新 skill 违反已有约定 | 检查新加 skill 的 frontmatter | 未发现违规 | 无 |

**命中后的具体行动建议**：
- `gstack/gstack-upgrade/SKILL.md` 的 git clone + rm -rf 组合 → 在第 140 行前插入一个明确的「显示 diff + 等待用户确认」步骤 → 15-30 分钟可完成

### 7.3 别人的更优方案

1. **领域**：危险操作的安全重构
   - **本案例做法**：删除 dbskill-upgrade（含危险 rm -rf + git clone），新增 dbs-save、dbs-restore、dbs-skill-cleaner 三个更小粒度的安全工具（路径：`skills/dbs-skill-cleaner/SKILL.md`）
   - **我的项目现状**：gstack 的 `gstack-upgrade/SKILL.md` 仍有 `git clone + rm -rf` 的直接组合，未拆分
   - **如何借鉴**：参照 dbs 的拆分思路，把 gstack-upgrade 拆分为 `gstack-save-backup`（备份当前）、`gstack-pull`（拉取新版）、`gstack-restore`（失败回滚）

2. **领域**：`dbs-` 统一命名前缀
   - **本案例做法**：所有 skill 统一 `dbs-` 前缀，shell 可用 `ls skills/dbs-*` 批量操作（路径：`skills/dbs-*/`）
   - **我的项目现状**：gstack 的 skill 目录名没有统一前缀（`setup-deploy`、`ios-sync`、`diagram` 等混合风格）
   - **如何借鉴**：如果重新设计 gstack，用 `gs-` 前缀统一所有 skill 名；现有项目不强制重命名，但新增 skill 遵循统一前缀约定

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：跨 skill 引用校验
- **我的做法**（gstack）：每个 SKILL.md 末尾有统一的 "Writing Guidelines" 章节，包含对 `See Also:` 引用格式的约定，减少了死引用风险
- **本案例做法（弱在哪）**：`dbs-ai-check` 在 2026-04-13 时有一个死引用 `/文风分析`（现已修复），说明当时没有跨 skill 引用校验机制
- **意义**：gstack 的统一 Writing Guidelines 让未来的引用格式一致，降低了死链风险。

---

## 八、术语表

### <a name="经验积累侧车"></a>经验积累侧车
> 把创作者/作者多年积累的经验（如推文、文章、讲义）整理成结构化文件（"知识原子"），让 AI skill 可以引用这些文件来激活和应用这些经验，而不是把经验直接写死在 skill 里。"侧车"（sidecar）比喻：就像摩托车的侧车，知识原子紧挨着 skill 但独立存在，可以单独更新。

### <a name="知识原子"></a>知识原子
> 最小粒度的、独立可引用的知识单元。dbskill 中的知识原子来自 dontbesilent 的推文，被整理为 4176 个独立文件，每个文件代表一个具体的观点、方法或案例。AI skill 引用这些文件来回答特定领域的问题。

### <a name="奥派经济学"></a>奥派经济学
> 奥地利学派经济学（Austrian School of Economics）的简称，强调个人主义、自由市场和主观价值论。`dbs-chatroom-austrian` 是专门从奥派视角分析商业问题的 skill，只在特定条件下被触发（而非主菜单入口）。
