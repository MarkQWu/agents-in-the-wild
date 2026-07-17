# dontbesilent2025/dbskill — 学习案例

**仓库**：https://github.com/dontbesilent2025/dbskill
**Stars**：N/A（exemplar_published=true）| **来源**：upstream（exemplar）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-17（基于当前 HEAD）
**主题标签**：router-channels, single-purpose, vague-quantifier, security-gate, experience-accumulation

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

dontbesilent2025/dbskill 是一个面向中文创业者的商业诊断技能套件，通过「一个路由器 + 多个专科技能」的结构，让 Claude 扮演企业诊断顾问。关键事实：

- 作者定位为「不要沉默」哲学的实践者，关注内容创作者的商业决策困境
- 审计时（2026-04-13）有 12 个技能，当前 HEAD 有 **29 个技能**（增幅 2.4 倍）
- 原 `dbskill-upgrade` 技能（含 HIGH 安全风险）已**完全移除**
- 新增技能涵盖内容系统（dbs-content-system）、目标管理（dbs-goal）、学习管理（dbs-learning）等
- 全站中英双语触发词，支持中文母语用户和英文工具场景

### 1.2 架构剖析

**目录结构**：

```
dbskill/
├── LICENSE
├── README.md / README.en.md / README.zh-TW.md / README.ko.md / README.ja.md
├── VERSION
├── scripts/record-demo.sh         # 演示录制脚本（开发工具）
├── 知识库/Skill知识包/             # 中文知识库参考目录
├── docs/                          # 补充文档
├── tools/                         # 辅助工具
└── skills/                        # 29 个技能目录
    ├── dbs/SKILL.md               # 主路由器
    ├── dbs-action/SKILL.md        # 执行力诊断
    ├── dbs-benchmark/SKILL.md     # 基准测试
    ├── dbs-diagnosis/SKILL.md     # 综合诊断
    ├── dbs-content-system/        # 新增：内容系统（含模板文件）
    │   ├── SKILL.md
    │   ├── templates/             # 内容单元模板
    │   └── scaffold/              # 初始化脚手架
    ├── dbs-goal/SKILL.md          # 新增：目标管理
    ├── dbs-learning/SKILL.md      # 新增：学习管理
    └── …（22 个其他技能目录）
```

**文件类型分布**：29 个 SKILL.md + 1 个路由 SKILL.md（dbs）+ 多个模板文件 + 知识库参考文档

**编排关系**：`dbs/SKILL.md`（[路由器](#router)）是唯一入口，解析用户意图后派发到对应专科技能。专科技能之间也有少量横向引用（如 dbs-diagnosis 在检测到心理问题时引用 dbs-action）。这是标准的 [Router + Channels](#router) 分层模式。

**跨件契约**：`知识库/Skill知识包/` 目录作为共享知识源，被多个专科技能引用。所有内部引用路径已验证存在。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「路由优先」——用户无需知道具体技能名，说出问题，路由器自动派发
- **解决什么问题**：创业者面临多维度困境（执行力缺失 / 商业模式不清 / 内容增长停滞），需要不同「专科医生」诊断，但用户不想手动挑选
- **做了什么 trade-off**：集中入口换取更高的触发一致性；代价是路由器本身需要维护，新增技能要同步更新路由表
- **反映什么认知模型**：作者把 Claude 视为「商业顾问团队」，每个技能是一位专科顾问，路由器是前台接待

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「路由器 + 专科诊断平铺扩展式」

路由器层极轻（85 行），专科层各自独立、可无限追加。新增技能只需：①创建 `skills/dbs-xxx/SKILL.md`；②在路由器表格里加一行。

模式特征清单：
- **特征 1**：单一入口（`/dbs`），用户零学习成本
- **特征 2**：路由表为三列格式（触发信号 → 目标技能 → 一句话说明），维护成本低
- **特征 3**：专科技能末尾有「条件→跳转」表，形成反向引用网
- **特征 4**：知识库独立于技能，作为共享引用层
- **特征 5**：中英双语触发词并排，一份技能覆盖两个语言环境

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 用户群不熟悉技能名、习惯描述问题 | ✅ 高度适用 | 路由器把自然语言问题映射到技能，降低用户门槛 |
| 多维诊断类工具（医疗/商业/法律） | ✅ 适用 | 子领域边界清晰，路由判断容易 |
| 技能数量少（<5 个）的简单套件 | ❌ 过度设计 | 直接用 description 触发就够，无需路由层 |
| 子领域高度重叠、难以路由的场景 | ❌ 不适用 | 路由错误会让用户困惑，比不路由更糟 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 路由器 + 专科平铺（本仓库） | dontbesilent2025/dbskill | 用户体验统一，无需学习技能名 | 路由器维护负担，每个新技能要更新两处 |
| 纯 description 触发平铺 | czlonkowski/n8n-skills | 零路由层，加载最快 | 用户需要知道技能名或精确描述 |
| 层级嵌套调用 | mikeyobrien/ralph-orchestrator | 可组合复杂工作流 | 编排复杂，上下文消耗大 |

### 2.4 改进空间

1. **当前问题**：路由表随技能增加（12→29）变大，路由器本身开始超过 85 行  
   **改进做法**：把路由表拆为「核心路由（常用 10 个）」和「扩展路由（低频技能）」两段，或用动态路由  
   **预期收益**：路由器 token 占用减少，核心场景响应更快

2. **当前问题**：dbs-benchmark 和 dbs-diagnosis 中的模糊量词（"合理时间内"、"合适的时机"）持续未修复  
   **改进做法**：将 "合理时间内" 替换为「3-6 个月内」；"合适的时机" 替换为「当对方主动提到情绪困扰时」  
   **预期收益**：Claude 行为更可预测，避免因不同语境对「合理」有不同解读

3. **当前问题**：dbs-content-system 新增了 scaffold/ 目录（含 CLAUDE.md、AGENTS.md 等），但这些 scaffold 文件是用来初始化新项目的，与技能运行逻辑耦合不清晰  
   **改进做法**：在 dbs-content-system/SKILL.md 开头明确说明 scaffold 的用途和何时使用  
   **预期收益**：减少用户对「这些文件是干什么的」的困惑

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 **90/100**（共 12 个 artifact）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/dbskill-upgrade/SKILL.md | 78 | 嵌入危险 shell 脚本；非标 frontmatter；仅中文 description |
| skills/dbs-ai-check/SKILL.md | 86 | `/文风分析` 死链引用 |
| skills/dbs-benchmark/SKILL.md | 90 | "合理时间内" 模糊量词 |
| skills/dbs-diagnosis/SKILL.md | 90 | "合适的时机" 模糊量词 |
| skills/dbs/SKILL.md | 91 | 路由器整体干净，chatroom-austrian 未入主菜单（设计意图） |
| skills/dbs-action/SKILL.md 等 | 91-93 | 无重大问题 |

### 3.2 当时值得借鉴的模式

1. **双语触发词嵌入（R04）**：每个技能的 description 同时包含中文和英文触发短语（如「/dbs-action」「我知道该怎么做但就是不做」「why do I procrastinate」），一份技能覆盖两种查询语言 → 原文：`skills/dbs-action/SKILL.md:3-7`

2. **路由器三列表格（R05）**：dbs/SKILL.md 在 85 行内用「信号 → 目标 → 说明」三列表格完成 11 个技能的路由，零散文解释 → 如何借鉴：设计路由层时用表格而非散文，节省 token

3. **心理框架可操作化（R06）**：dbs-action 不说「用阿德勒框架」，而是把阿德勒心理学转化为具体的「4 种目的」检查清单，Claude 可以逐条执行 → 抽象理论 + 具体步骤化才能让 AI 可用

4. **反向引用网（R07）**：每个专科技能末尾有「当观察到 X 信号时，切换到 Y 技能」的条件表，与路由器形成双向网络

5. **知识库独立层（R08）**：`知识库/Skill知识包/` 作为共享参考，各技能引用而不复制，保持单一数据源

### 3.3 当时的缺陷

1. **问题**：`dbskill-upgrade/SKILL.md` 嵌入 `rm -rf ~/.claude/skills/dbs*` + `git clone` + `cp` 脚本，构成供应链攻击面  
   **根本原因**：作者把「一键自动升级」设计为技能内的 shell 脚本，但这意味着 Claude 会在没有明确安全门禁的情况下删除用户文件并安装远程代码  
   **自查**：我的技能中有没有嵌入 `rm -rf`、`git clone`、`curl | bash` 类命令？

2. **问题**：`dbs-ai-check/SKILL.md:255` 引用了 `/文风分析` 技能，但该技能不存在  
   **根本原因**：技能迭代时删除/重命名了某个技能，但引用方未同步清理  
   **自查**：我的技能内有没有引用不存在的 `/技能名`？

3. **问题**：两处模糊量词（"合理时间内"、"合适的时机"）——前者是资源评估标准，后者是介入时机判断  
   **根本原因**：中文技能写作时「合理/合适」比英文「appropriate」更常见，但 AI 同样无法从中得出可执行操作  
   **自查**：我的中文技能中有没有「合理/合适/恰当/适当」类量词？

### 3.4 当时的优化机会

1. 完全移除 dbskill-upgrade 或替换为纯提示词版（不执行 shell）
2. 修复 `/文风分析` 死链（改为已存在的技能名或删除引用）
3. 将 "合理时间内" 替换为具体时间范围（如「6 个月内」）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| dbskill-upgrade HIGH 安全风险（rm -rf + git clone） | `ls skills/ \| grep upgrade` | **已修复**：dbskill-upgrade 目录完全移除 | 作者选择「删除」而非「修补」，彻底消除攻击面 |
| /文风分析 死链引用 | `grep "文风分析" skills/dbs-ai-check/SKILL.md` | **已修复**：grep 无匹配 | 清理时一并删除了该引用 |
| "合理时间内" 模糊量词 | `grep "合理时间" skills/dbs-benchmark/SKILL.md` | **持续存在**（line 71） | 作者未视为优先问题 |
| "合适的时机" 模糊量词 | `grep "合适的时机" skills/dbs-diagnosis/SKILL.md` | **持续存在**（line 410） | 同上 |

### 4.2 架构演进

从 Audit 时（12 技能）到现在（29 技能）的重大变化：

- **安全加固**：删除 dbskill-upgrade 整个技能目录——这是「以删代修」的典型策略，当一个组件风险高于价值时，移除比修复更安全
- **内容系统化**：新增 dbs-content-system，包含 templates/ 和 scaffold/，把「知识内容组织」从简单的技能升级为带有模板和脚手架的微型系统
- **生命周期管理技能**：新增 dbs-goal（目标）、dbs-learning（学习）、dbs-update（更新记录）——从「诊断」扩展到「持续追踪」，覆盖创业者全生命周期
- **国际化**：增加 README.ko.md（韩文）和 README.zh-TW.md，显示作者扩展到更多亚洲语言用户群

这说明作者的方向是：安全问题靠「删除」解决，功能演进靠「追加」推进，NL 质量问题（vague quantifier）保持低优先级。

### 4.3 新增的可学习模式

1. **脚手架嵌入技能目录**：dbs-content-system 在 `scaffold/` 子目录里包含完整的项目初始化文件（CLAUDE.md、AGENTS.md、规则集），技能不仅提供操作指导，还提供可直接复制的初始文件——这是「技能即脚手架」模式，大幅降低用户从零开始的门槛

2. **多语言 README 并行**：5 个语言版本的 README（中/英/繁中/韩/日），说明作者认为「文档国际化」和「技能国际化」是分开维护的——技能本身用中英双语，README 则逐语言维护

---

## 五、校准

### 5.1 我已经在做对的

1. **技能单职责**：我的 gstack 每个技能（setup-deploy、ios-design-review 等）同样专注单一操作域
2. **路由思维存在**：bureau 的 skills 目录下各技能各司其职（capture/compile/review/recall），虽无显式路由器，但职责清晰
3. **知识库参考分离**：gstack 的 plan-devex-review/sections/ 目录类似 dbskill 的 `知识库/` 层，均为共享参考
4. **无危险嵌入脚本**：我没有在技能中嵌入 `rm -rf` 或 `git clone` 类操作

### 5.2 挑战 / 验证

本案例**挑战**了「安全问题修复需要巧妙方案」的假设——dontbesilent 的方案是直接删除整个 dbskill-upgrade 目录，3 个 HIGH 安全问题瞬间归零。有时候「移除」比「修复」更正确。这让我思考：当我的某个技能风险高于价值时，应该直接删除而不是继续修补。

本案例还**验证**了「中英双语触发词」的实际价值——dbskill 被选为 exemplar，R04（触发词设计）是核心理由之一，双语并排正是其亮点。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能中是否嵌入危险 shell 命令
grep -rn "rm -rf\|git clone\|curl.*bash\|wget.*sh" \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：评估该命令是否可改为引导用户手动执行（安全门禁）；若命令是核心功能，则保留但加确认提示。

```bash
# 检查我的中文技能中是否有模糊量词
grep -rn "合理\|合适\|恰当\|适当\|必要时" \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：将「合理时间」改为具体数字（如「5 分钟内」），将「合适的时机」改为可观测条件（如「当用户明确表达沮丧时」）。

```bash
# 检查我的技能中是否有死链（引用了不存在的技能名）
grep -rn "/[a-z-]*" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null \
  | grep -E "/(capture|compile|review|recall|lint|scribe|guide)" | head -10
```
命中后怎么办：逐一确认引用的技能名在 plugin.json 里已注册。

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/bureau 添加「脚手架嵌入」——在 skills/capture/scaffold/ 下放一份标准的 CLAUDE.md 和日志格式模板  
   **为何可行**：dontbesilent 的 dbs-content-system 证明了「技能即脚手架」模式，用户可以直接复制文件启动新项目  
   **第一步**：在 `skills/capture/scaffold/` 下创建 `session-log-template.md`，30 分钟可完成

2. **想法**：为 MarkQWu/gstack 添加一个极轻路由器技能 `gstack-meta/SKILL.md`，作为主入口  
   **为何可行**：gstack 目前有 20+ 个技能，用户需要知道具体技能名；路由器可把「我需要部署」映射到 `land-and-deploy`  
   **第一步**：创建 `gstack-meta/SKILL.md`，路由表只列最常用的 8 个技能，1 小时可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 dontbesilent2025/dbskill 的核心目的**：为中文创业者提供多维商业诊断，通过路由器统一入口派发到专科技能

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多技能套件，服务一个有角色感的用户群（gstack 是 CEO 工具）| gstack 聚焦技术操作；dbskill 聚焦商业诊断 | 高（路由器模式可借鉴） |
| MarkQWu/bureau | 中 | 同为内容型工具，有知识管理维度 | bureau 是知识库系统；dbskill 是诊断顾问 | 中（脚手架模式可借鉴） |
| MarkQWu/shiji-kb | 低 | 同有中文内容处理 | shiji-kb 是古诗知识库，纯检索；dbskill 是交互诊断 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词（"合理"/"合适"） | `grep -rn "合理\|合适" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | 未命中（bureau 技能为英文） | 无 |
| 技能间引用死链 | `grep -rn "/(capture\|compile\|review)" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | 未检测到跨技能引用 → 无死链风险 | 无 |
| gstack 中嵌入 shell 脚本的风险 | `grep -n "rm -rf\|git clone" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | 未命中 | 无 |

**结论**：我的项目在本案例覆盖的问题维度上无命中，风险较低。

### 8.3 别人的更优方案

1. **领域**：中英双语触发词并排  
   - **本案例做法**：每个 SKILL.md 的 description 同时包含中文自然语言触发（「我知道该做什么但就是不做」）和英文触发（「I know what to do but can't do it」），一份技能覆盖两种查询语言  
   - **我的项目现状**：MarkQWu/gstack 所有触发词（`triggers:` 字段）均为英文  
   - **如何借鉴**：在 gstack 的关键技能（如 `spec/SKILL.md`）description 中加中英双语触发，针对中文用户查询场景（5-10 分钟改动）

2. **领域**：路由器模式统一入口  
   - **本案例做法**：`dbs/SKILL.md` 只做路由，用户输入 `/dbs` 就能到达任何子技能  
   - **我的项目现状**：gstack 没有主路由器，用户需要知道确切的技能名（如 `land-and-deploy`）才能触发  
   - **如何借鉴**：创建 `gstack-meta/SKILL.md` 作为路由器，内容只有一个「信号→技能」三列表格

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：禁止词声明机制  
  - **我的做法**：MarkQWu/gstack 每个 SKILL.md 都有约 580 行处的明确 "No AI vocabulary" 禁止列表，运行时强制约束  
  - **本案例做法（弱在哪）**：dbskill 没有对输出词汇做约束，中文 AI 痕迹词（「综合来看」「不难发现」等）未做显式禁止  
  - **意义**：若 dbskill 被 NLPM 检测，这类输出风格问题不会扣分，但用户体验上可能有 AI 味；我的禁止词机制是更主动的质量控制

---

## 八、术语表

### <a name="router"></a>路由器（Router 技能）
> 在 Claude Code 插件中，一个专门负责「接收用户输入并决定派发到哪个技能」的 SKILL.md。类似网站的首页导航——用户不需要记住每个页面的 URL，只需告诉首页要做什么。dbskill 的 `dbs/SKILL.md` 就是路由器：用户说「帮我看看」，路由器判断是执行力问题还是商业模式问题，然后跳转到对应的专科技能。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道如何注册和触发这个技能。

### <a name="curl-pipe-bash-risk"></a>curl-pipe-bash 风险
> 一种高风险安装模式：`curl https://... | bash`——直接把从网络下载的内容传给 bash 执行。如果下载源被攻击者控制，用户机器会执行恶意代码。dbskill-upgrade 的 git clone + cp 模式与此类似——从 GitHub 拉代码然后复制到用户的 Claude 配置目录，如果该 GitHub repo 被劫持，用户所有后续 Claude 会话都会加载恶意技能。
