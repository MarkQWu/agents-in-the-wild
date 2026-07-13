# dontbesilent2025/dbskill — 学习案例

**仓库**：https://github.com/dontbesilent2025/dbskill
**Stars**：N/A | **来源**：upstream audit（exemplar_published=true）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-13（基于当前 HEAD）
**主题标签**：`router-channels`, `cross-reference`, `single-purpose`, `security-gate`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`dontbesilent2025/dbskill`（"dontbesilent 商业技能包"，下称 dbskill）是一套面向中文创业者的 Claude Code 商业诊断工具集。作者 dontbesilent2025 是内容创作者兼商业顾问，把自己的方法论（阿德勒心理学、奥地利经济学等）结构化为可调用的 NL skill。

截至 2026-07-13 当前 HEAD：
- **27 个 skill**（审计时 12 个；新增 15 个：dbs-bridge、dbs-agent-migration、dbs-chatroom、dbs-content-system、dbs-decision、dbs-goal、dbs-good-question、dbs-learning、dbs-report、dbs-resonate、dbs-restore、dbs-save、dbs-script-flow、dbs-spread、dbs-update、dbs-wechat-html）
- **dbskill-upgrade skill 已完全移除**（审计时存在高危安全风险）
- **dbs-content-system**：含完整内容管理 scaffold（rules/、templates/、docs/）的复合 skill
- **dbs-bridge**：新增 scripts/bridge-skill.sh 执行表面
- 版本通过 VERSION 文件管理；.claude-plugin/marketplace.json 注册
- GitHub Actions release.yml 自动化发布流程

### 1.2 架构剖析
- **目录结构**：
  ```
  dbskill/
  ├── skills/
  │   ├── dbs/SKILL.md              # 主路由器（11→27 个目标）
  │   ├── dbs-action/               # 执行力诊断（阿德勒框架）
  │   ├── dbs-ai-check/             # AI 内容质量检测
  │   ├── dbs-benchmark/            # 竞品对标
  │   ├── dbs-bridge/               # 新：NL bridge 脚本
  │   │   ├── SKILL.md
  │   │   └── scripts/bridge-skill.sh
  │   ├── dbs-content-system/       # 新：内容管理系统脚手架
  │   │   ├── SKILL.md
  │   │   ├── docs/
  │   │   ├── scaffold/root/         # AGENTS.md、CLAUDE.md 等模板
  │   │   └── scaffold/rules/        # 内容单元规则（5 个规则文件）
  │   │   └── templates/             # 内容单元模板（5 类）
  │   ├── dbs-chatroom/             # 新：通用聊天室
  │   ├── dbs-chatroom-austrian/    # 奥派经济学聊天室
  │   ├── dbs-deconstruct/
  │   ├── dbs-diagnosis/
  │   ├── dbs-hook/
  │   ├── dbs-slowisfast/
  │   ├── ... （共 27 个）
  ├── tools/                        # 辅助工具
  ├── 知识库/Skill知识包/           # 每个 skill 的知识包文件
  ├── docs/
  └── scripts/record-demo.sh
  ```
- **文件类型分布**：27 个 SKILL.md / 0 个 agent / 0 个 command / 1 个执行脚本（bridge-skill.sh）/ 丰富的知识库文件
- **编排关系**：`dbs/SKILL.md` 是主路由器，用三列表格（信号→目标→一句描述）路由到所有叶节点 skill。部分 skill（如 dbs-diagnosis）会根据诊断结果推荐下一步 skill，形成动态路径。`chatroom-austrian` 走独立入口（`/奥派`），不在主路由器中列出。
- **跨件契约**：每个深层 skill 的"下一步建议"引用其他 skill；`知识库/Skill知识包/` 提供 skill 的扩展知识文件；`dbs-content-system` 内嵌完整 scaffold，相当于一个"项目初始化器"。

### 1.3 设计思路 / 方法论
- **核心设计哲学**："心理框架为诊断引擎，可观察行为替代抽象理论"——不是讲"阿德勒认为 X"，而是把阿德勒的洞见转化为"当用户出现 Y 行为时，信号是 Z，路由到 dbs-action"的可执行判断规则。
- **解决什么问题**：创业者的痛点（执行力缺失、商业模式不清、内容质量差）通常被描述为"我就是做不到"的模糊感受。dbskill 把这些模糊感受结构化为诊断信号，让 Claude 能给出基于框架的、可验证的建议，而不是泛泛鼓励。
- **做了什么 trade-off**：选择"路由器 + 叶节点"而非"单一大 skill"——好处是每个 skill 专注单一诊断域，Claude 载入上下文少；代价是路由器需要随 skill 增加不断更新（27 个 skill 的路由表已相当大）。
- **反映什么认知模型**：作者把 Claude 视为"方法论载体"——方法论（阿德勒、奥派经济学、慢即是快）不应该让用户自己搜索和学习，而是应该被内嵌进工具，在需要时自动激活。这和"工具即顾问"的认知一致。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「路由器 + 独立诊断域叶节点 + 知识包侧车」**

一个薄路由器用信号-目标表格做粗粒度分发，每个诊断域的叶节点 skill 用心理学框架驱动深层推理，知识库文件作为侧车提供扩展数据。

模式特征清单：
- **特征 1**：路由器极薄（85 行），不含业务逻辑，只负责信号识别和路由
- **特征 2**：双语触发（中英文 trigger phrases 并列），覆盖中英混用的真实用户
- **特征 3**：每个叶节点 skill 内嵌具体心理框架名称和可观察行为模板
- **特征 4**：知识包侧车（`知识库/Skill知识包/`）和 skill 解耦，便于独立更新
- **特征 5**：dbs-content-system 引入了 scaffold 模式，skill 变成"项目脚手架生成器"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 垂直领域的深度诊断（心理/法律/医疗建议） | ✅ 高度适用 | 路由器+叶节点可以把专业框架模块化，每个框架独立迭代 |
| 多语言用户的技能包（中英混用） | ✅ 适用 | 双语 trigger phrases 直接解决语言匹配问题 |
| 高安全要求的工具（需要安装/升级用户文件） | ❌ 不适用 | dbskill 曾有 dbskill-upgrade 的反面教材：升级逻辑不能放在 NL skill 里 |
| 功能边界不清晰的通用工具 | ⚠️ 有限适用 | 路由器会变得臃肿（27 个 skill 已有此倾向） |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 路由器 + 叶节点（本仓库） | dontbesilent2025/dbskill | 路由器薄；每个 skill 可独立测试和更新 | 路由器随 skill 增加膨胀；27 个路由项已接近上限 |
| 单一 monolith skill | 早期小型技能包 | 安装简单，无路由维护负担 | 超 500 行后 Claude 检索困难 |
| 分层路由（meta router + sub-router） | Q00/ouroboros（参考） | 可扩展至 100+ skill，路由层不膨胀 | 结构复杂，用户心智模型成本高 |

### 2.4 改进空间
1. **当前问题**：路由器 `dbs/SKILL.md` 已有 27 个路由项，随着 skill 继续增加，路由器会超过 500 行。**改进做法**：拆分成"主路由器"（dbs）+ "内容路由器"（dbs-content，管理 content-system/save/restore/spread 等）两层，每层各管 15 个以内的 skill。**预期收益**：避免路由器变成 monolith。
2. **当前问题**：`dbs-bridge/scripts/bridge-skill.sh` 引入了新的执行表面，但审计未覆盖。**改进做法**：定期用 `/nlpm:security-scan` 扫描 bridge-skill.sh，建立 shell 脚本白名单。**预期收益**：避免执行表面悄悄扩大。
3. **当前问题**：router 引入了 27 个 skill 的一对一表格，但新 skill 被加入时未必每次都更新 router。**改进做法**：在 CI 中加一个检查：`ls skills/ | wc -l` 是否等于 router 的路由条目数。**预期收益**：防止新增 skill 未注册到 router 导致用户找不到。

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
90/100（12 个文件，安全评级 REVIEW）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| dbskill-upgrade/SKILL.md | 78 | 内嵌 rm -rf + git clone；非标准 frontmatter；纯中文 description |
| dbs-ai-check/SKILL.md | 86 | 引用不存在的 /文风分析 skill |
| dbs-benchmark/SKILL.md | 90 | "合理时间内" 模糊量词 |
| dbs-diagnosis/SKILL.md | 90 | "合适的时机" 模糊量词 |
| dbs-xhs-title/SKILL.md | 90 | 无问题（75 个内联公式，质量高） |
| dbs/SKILL.md | 91 | chatroom-austrian 未在编号菜单中 |
| chatroom-austrian/SKILL.md | 93 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **双语 trigger phrase 并列** → 每个 skill description 同时包含中文（"知道该做什么但就是不做"）和英文（"I know what to do but can't do it"）触发短语。**为何好**：技术小白中英混用时，无论用哪种语言描述问题，都能命中 skill。**原文路径**：`skills/dbs-action/SKILL.md:3-7`。**如何借鉴**：中英混合受众的 skill 中，description 字段中英并列写，不要只写一种语言。
2. **心理框架名称内嵌触发器** → "dontbesilent 执行力诊断。用**阿德勒心理学框架**诊断..."——不只说"诊断执行力"，而是说用什么框架，定位更精准，用户信任度更高。**如何借鉴**：skill description 中明确框架/方法论名称，不只描述功能。
3. **路由器的信号-目标-描述三列表** → `dbs/SKILL.md` 用三列表（用户信号 → 路由目标 → 一句描述）而不是文字段落做路由，Claude 可以快速扫描匹配。**如何借鉴**：路由器 skill 用表格而不是列表或段落。

### 3.3 当时的缺陷
1. **dbskill-upgrade 中的 rm -rf + git clone（HIGH 安全）** → `skills/dbskill-upgrade/SKILL.md` 嵌入了 bash 脚本，执行 `rm -rf ~/.claude/skills/dbs*` 后从 GitHub 克隆覆盖——相当于一个未经完整性校验的自动更新器。根本原因：作者想提供便捷升级路径，但没有意识到 NL skill 执行 shell 命令是高安全风险。自查：我的 skill 是否有引导 Claude 执行破坏性 shell 命令的内容？
2. **引用不存在的 /文风分析 skill（Bug）** → `dbs-ai-check` "下一步建议"中引用了 `/文风分析`，但该 skill 不存在。根本原因：skill 演进过程中删除或重命名了某个 skill，但未更新所有引用它的地方。自查：我的 skill 中的"下一步推荐"是否都对应实际存在的 skill？
3. **`scripts/record-demo.sh` sed 注入风险（MEDIUM）** → `MARKETPLACE_REPO` 变量来自 git remote URL，未经净化就代入 sed 表达式。根本原因：在 shell 脚本中信任了 git remote 的返回值。自查：我的脚本是否有将 git remote URL 直接代入 shell 命令而未净化的情况？

### 3.4 当时的优化机会（仅供学习）
1. `dbskill-upgrade` 中的安全高危脚本 → 移除或用安全更新机制（如官方 marketplace 升级）替代
2. `/文风分析` 死引用 → 移除或替换为已存在的 skill
3. `record-demo.sh:63` sed 注入 → 对 MARKETPLACE_REPO 变量过滤特殊字符

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| dbskill-upgrade HIGH 安全风险 | `ls skills/ | grep upgrade` → 无输出 | **已移除**：整个 upgrade skill 被彻底删除，不是打补丁而是移除整个概念 | 作者选择了"删除危险入口"而不是"修复危险逻辑"——更彻底的安全决策 |
| /文风分析 死引用 | `grep -rn "文风分析" skills/` → 无命中 | **已修复**：死引用被完全移除 | NL bug 修复 |
| record-demo.sh sed 注入 | `grep -A2 "MARKETPLACE_REPO" scripts/record-demo.sh` → 第 62 行有 `tr -dc 'A-Za-z0-9/_.-'` 净化 | **已修复**：加入字符白名单过滤 | Medium 安全修复 |

### 4.2 架构演进
从 12 skills（纯诊断工具）→ 27 skills（诊断 + 内容管理 + 桥接 + 聊天室）：
- **最大变化 1**：移除 dbskill-upgrade。这说明作者认识到"自升级"的安全风险，选择通过正常 marketplace 机制更新，而不是内嵌更新逻辑。
- **最大变化 2**：新增 `dbs-content-system`——含 scaffold 模板的复合 skill。这是从"诊断建议"到"脚手架生成"的功能升级，说明用户需要从 Claude 获得可直接使用的结构，而不只是建议。
- **最大变化 3**：新增 `dbs-bridge`——含 shell 脚本的执行表面。这是一个悖论：刚移除了 dbskill-upgrade 的执行风险，又新增了 bridge-skill.sh。需要关注此脚本是否存在类似风险。
- **命名演进**：`chatroom-austrian` 重命名为 `dbs-chatroom-austrian`（加了 dbs 前缀），统一了命名前缀规范。

### 4.3 新增的可学习模式
1. **Scaffold-as-a-skill 模式**：`dbs-content-system` 不只是"提建议的 skill"，而是包含完整 CLAUDE.md 模板、规则文件、内容单元模板的脚手架生成器。当 Claude 被调用时，它可以直接输出或生成这些文件结构，让用户的项目直接从零到有框架。这是"skill 作为项目初始化器"的新用法。
2. **移除有风险的 skill 而非修复**：面对高安全风险，选择"移除整个功能"而不是"在危险代码上打补丁"是更彻底的安全决策。这对应"减少执行表面"原则。

---

## 五、校准

### 5.1 我已经在做对的
1. **路由器 + 叶节点分离**：echo-sleuth-for-claude 的 `commands/` 是路由层，`agents/` 和 `skills/` 是执行层，逻辑分离和本仓库一致。
2. **双语触发（潜在）**：我的 gstack skill 的 description 主要是英文，但应用场景覆盖中英混用用户时，可能需要加中文触发短语。
3. **知识包侧车**：echo-sleuth 的 `skills/` 提供解析规则和合成分类法，和 dbskill 的 `知识库/Skill知识包/` 思路一致——知识和技能分层存放。

### 5.2 挑战 / 验证
本案例挑战了我的假设：**"skill 越多越好，功能越全越有价值"**。dbskill 的演进（12→27 skills）伴随着路由器膨胀风险和新执行表面引入的安全风险。在 gstack 或 echo-sleuth 中增加 skill 前，应该先问：**是新建 skill，还是扩展现有 skill 的 trigger phrases 覆盖边界情况？**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中的"下一步建议"或"推荐命令"是否都对应存在的 skill 文件
grep -rn "/[a-zA-Z一-鿿]" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null | head -10
# 命中后：验证每个 /command 引用是否对应 commands/ 中的实际文件
```

```bash
# 检查我的 shell 脚本是否有将 git remote URL 直接代入 shell 命令
grep -rn "git remote\|origin" \
  /tmp/my-repos/MarkQWu-*/scripts/ 2>/dev/null | grep -v ".git" | head -10
# 命中后：确认变量是否经过字符净化再代入 sed/eval 等
```

```bash
# 检查我的 skill 是否有引导 Claude 执行 rm -rf 或 git clone 到 ~/.claude 的内容
grep -rn "rm -rf\|git clone" \
  /tmp/my-repos/MarkQWu-*/skills/ \
  /tmp/my-repos/MarkQWu-*/commands/ 2>/dev/null | head -10
# 命中后：评估是否应该改为引导用户手动操作，而非 Claude 直接执行
```

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 加"scaffold-as-a-skill"功能——当用户问"帮我新建一个 echo-sleuth 会话目录"时，生成标准的 `~/.claude/projects/xxx/` 目录结构和初始配置。**为何可行**：本案例证明 skill 可以携带 scaffold 模板；echo-sleuth 已有文件结构知识。**第一步**：在 `skills/` 下新建 `session-init/SKILL.md`，描述 trigger phrases 和输出格式；将标准目录模板写在 skill body 中。30 分钟可完成原型。
2. **想法**：给 gstack 的 skill description 加中文触发短语，覆盖中文用户场景（如"帮我做 iOS 设计评审" → ios-design-review）。**为何可行**：dbskill 证明双语触发有效，且改动只需编辑 SKILL.md 的 description 字段。**第一步**：选 gstack 中最常用的 3 个 skill，在 description 后追加 `也可用于：<中文触发场景1>、<中文触发场景2>`。改动 3 个文件各 1 行。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 dontbesilent2025/dbskill 的核心目的**：通过 27 个诊断+内容管理 skill，让 Claude 为中文创业者提供基于心理学和经济学框架的个性化商业诊断。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是"给特定用户群的多 skill 诊断工具集"；都有 router→skill 分发 | echo-sleuth 针对技术用户分析 session；dbskill 针对创业者做商业诊断 | 中 |
| MarkQWu/bureau | 低 | 都有结构化知识管理思路 | bureau 是知识库系统；dbskill 是实时诊断工具 | 低 |
| MarkQWu/shiji-kb | 低 | 都有中文内容处理场景 | shiji-kb 是诗词知识库；dbskill 是商业工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 引用不存在的 slash command | `grep -rn "^\- /\|推荐.*/" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*.md 2>/dev/null` | echo-sleuth skills 中无对其他 skill 的引用；命中 0 条 | 无 |
| shell 脚本中 git remote URL 未净化 | `grep -rn "git remote" /tmp/my-repos/MarkQWu-*/scripts/ 2>/dev/null` | 未发现脚本直接将 git remote 代入 sed/eval | 无 |
| skill 内嵌破坏性 bash（rm -rf / git clone 到用户目录） | `grep -rn "rm -rf\|git clone.*~/" /tmp/my-repos/MarkQWu-*/skills/ 2>/dev/null` | 0 命中 | 无 |

**结论**：本案例的三类问题在我的项目中均未复现。但考虑到 dbskill 在添加 dbs-bridge（有脚本）时引入了新的执行表面，我应在 bureau 或 echo-sleuth 新增任何带 `scripts/` 的 skill 时主动用 `/nlpm:security-scan` 扫描。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：双语触发短语设计
   - **本案例做法**：`skills/dbs-action/SKILL.md` description 字段中文触发短语（"知道该做什么但就是不做"）和英文触发短语（"why do I procrastinate"）并列，且包含用户会原话说出的情绪化表达。
   - **我的项目现状**：MarkQWu/echo-sleuth-for-claude 的 skill description 全为英文，缺少中文触发；gstack skill 也主要是英文。
   - **如何借鉴**：给 echo-sleuth 的 `/recall` skill 的 description 加中文触发："帮我找上次做了什么"、"上次 session 我处理的是什么"。改动 1 行，5 分钟。

2. **领域**：路由器用信号-目标-描述三列表格
   - **本案例做法**：`dbs/SKILL.md` 的路由条目是紧凑的表格（| 信号 | /dbs-xxx | 一句描述 |），而不是段落文字，Claude 扫描速度快。
   - **我的项目现状**：echo-sleuth 的 `commands/recall.md` 内有 agent 路由逻辑，但是以段落描述为主，没有使用表格。
   - **如何借鉴**：把 recall command 中的"根据不同场景路由到不同 agent"部分改为三列表格（用户场景 | 路由到 agent | 一句说明）。改动 1 个文件，10 分钟。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全执行边界的一致性
- **我的做法**：MarkQWu/bureau 和 MarkQWu/echo-sleuth-for-claude 的 skill 文件均不包含直接的 shell 命令执行逻辑——shell 操作被隔离在 `scripts/` 目录中，skill 只描述"调用脚本时的参数和格式"，不内嵌 bash 代码块。
- **本案例做法**（弱在哪）：dbskill 曾将 `rm -rf` 和 `git clone` 直接嵌在 SKILL.md 的 bash 代码块中，相当于让 Claude 直接执行破坏性命令。虽然 dbskill-upgrade 已删除，但 dbs-bridge 新增了 `scripts/bridge-skill.sh`，有重蹈覆辙的风险。
- **意义**：我的"skill 只描述协议，scripts/ 才是执行层"的分离是正确的安全架构。应该在 CLAUDE.md 中明确写出这个设计原则，防止未来无意中把 shell 逻辑移回 skill 文件。

---

## 八、术语表

### <a name="slash-command"></a>slash command
> 用户在聊天框中输入 `/something` 触发的命令，如 `/dbs-action`。Claude Code 会识别斜杠加名称的格式，并查找对应的 skill 或 command 来处理请求。

### <a name="scaffold"></a>脚手架（scaffold）
> 预先定义好的项目目录和文件模板集合，帮助用户快速创建有规范结构的新项目。`dbs-content-system` 的 scaffold 目录包含了 CLAUDE.md 模板、规则文件、内容单元模板——当用户调用这个 skill 时，Claude 可以用这些模板帮用户建立一套完整的内容管理系统目录。

### <a name="sed-injection"></a>sed 注入
> 在 bash 中，如果用变量直接代入 `sed "s|A|$B|g"` 命令，而 `$B` 包含特殊字符（如 `|`），会破坏 sed 的语法结构，可能导致执行意外命令。解决方法是先过滤 `$B` 的特殊字符（如 `tr -dc 'A-Za-z0-9/_.-'`），再代入命令。

### <a name="trigger-phrase"></a>触发短语（trigger phrase）
> skill description 中写给 Claude 看的"匹配关键词"列表。Claude 在理解用户意图时，会把用户的输入和 skill description 做语义匹配；trigger phrase 越精准、越贴近用户原话，skill 被正确激活的概率越高。双语 trigger phrase 是指同时提供中文和英文版本。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（如 `name`、`description`）。

### <a name="manifest"></a>manifest（.claude-plugin/plugin.json）
> 插件的"清单文件"，告诉 Claude Code 这个插件包含哪些组件（skill 路径、版本、作者等）。marketplace.json 是发布到市场的版本。
