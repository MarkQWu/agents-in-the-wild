# dontbesilent2025/dbskill — 学习案例

**仓库**：https://github.com/dontbesilent2025/dbskill
**Stars**：N/A | **来源**：本地 audit（exemplar_published=True, case_study_candidate=True）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-11（基于当前 HEAD）
**主题标签**：`router-channels`, `security-gate`, `curl-pipe-bash-risk`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

[dontbesilent2025/dbskill](https://github.com/dontbesilent2025/dbskill) 是一个以中文为第一语言的 Claude Code skill 插件集，服务于「Don't Be Silent」（DBS）这一内容创作方法论。仓库现有 26 个 skill，覆盖从诊断、基准测试、学习、决策、内容制作到奥派经济学辩论室的完整使用场景。

关键事实：
- **主要用户群**：中文创作者，所有 skill 的描述、示例、内容均以中文为主
- **唯一入口**：`/dbs` 是路由器 skill，用户只需记住这一个命令，由它负责分发到具体的诊断场景
- **架构演进**：从 audit 时的 12 个 skill 增长到当前的 26 个 skill，翻了一倍以上
- **最重要的变化**：`dbskill-upgrade` skill 目录已被完整删除——这正是 audit 中安全评级被标为 REVIEW 的根源所在
- **特色功能**：`dbs-chatroom-austrian` 实现了哈耶克 × 米塞斯 × Claude 三方辩论室，是整个仓库最具创意的单一设计

当前 skills/ 目录清单（26 个）：

```
skills/
├── dbs/SKILL.md                    ← 路由器（主入口）
├── dbs-action/SKILL.md
├── dbs-agent-migration/SKILL.md    ← 【新增】
├── dbs-ai-check/SKILL.md
├── dbs-benchmark/SKILL.md
├── dbs-bridge/SKILL.md             ← 【新增】含 scripts/bridge-skill.sh
├── dbs-chatroom/SKILL.md
├── dbs-chatroom-austrian/SKILL.md
├── dbs-content/SKILL.md
├── dbs-content-system/SKILL.md     ← 【新增】
├── dbs-decision/SKILL.md
├── dbs-deconstruct/SKILL.md
├── dbs-diagnosis/SKILL.md
├── dbs-goal/SKILL.md
├── dbs-good-question/SKILL.md
├── dbs-hook/SKILL.md
├── dbs-learning/SKILL.md
├── dbs-report/SKILL.md
├── dbs-resonate/SKILL.md
├── dbs-restore/SKILL.md
├── dbs-save/SKILL.md
├── dbs-script-flow/SKILL.md
├── dbs-slowisfast/SKILL.md
├── dbs-spread/SKILL.md
├── dbs-wechat-html/SKILL.md
└── dbs-xhs-title/SKILL.md
```

值得注意：`知识库/Skill知识包/` 目录存放原子化 JSON 知识文件，是 skill 内容的结构化底座；`scripts/record-demo.sh` 是开发者工具，不属于面向用户的 skill 表面。

### 1.2 架构剖析

```
用户
  │
  ▼
/dbs  ← dbs/SKILL.md（路由器）
  │
  ├──→ /dbs-diagnosis    诊断
  ├──→ /dbs-benchmark    基准测试
  ├──→ /dbs-decision     决策
  ├──→ /dbs-learning     学习
  ├──→ /dbs-goal         目标
  ├──→ /dbs-hook         挂钩写作
  ├──→ /dbs-content      内容制作
  ├──→ /dbs-spread       传播
  ├──→ /dbs-report       报告
  ├──→ /dbs-ai-check     AI 质量核查
  ├──→ /dbs-save         保存
  ├──→ /dbs-restore      恢复
  ├──→ ... （其余专科 skill）
  │
  └──→ /奥派  ← 直接访问 dbs-chatroom-austrian（绕过路由器）

知识底座
  知识库/Skill知识包/*.json  ← 原子化结构化知识

脚本层（开发者工具，非用户表面）
  scripts/record-demo.sh
  dbs-bridge/scripts/bridge-skill.sh
```

架构要点：
- **单一职责**：每个 skill 对应一个业务诊断场景，不重叠
- **路由器 + 专科**：`dbs` 作为调度中心，专科 skill 聚焦自身逻辑
- **例外**：`dbs-chatroom-austrian` 可通过 `/奥派` 直接触发，不必经过路由器——这是对高频特色功能的快捷通道设计
- **知识与逻辑分离**：JSON 原子文件承载知识内容，SKILL.md 承载执行逻辑

### 1.3 设计思路 / 方法论

**核心设计哲学**：「用户只需记住一个入口，系统承担分发的认知负担。」

在 Claude Code skill 生态里，用户记忆成本是真实存在的摩擦点。如果每个场景都需要单独记一个命令名，用户会遗忘、猜错，或干脆不用。DBS 的设计选择是：把认知复杂度从用户侧转移到 skill 系统侧——路由器 `dbs` 扮演「门诊导诊台」的角色，用户只说「我要用 DBS 的功能」，由导诊台决定送去哪个专科。

**解决什么问题**：
1. 多场景 skill 集合的可发现性问题——用户不需要知道 `dbs-benchmark` 这个名字
2. 中文用户的使用习惯——中文创作者更倾向于自然语言描述需求，而不是记忆英文命令符号
3. 场景边界的主动界定——路由器的菜单本身就是对「DBS 方法论能做什么」的声明式文档

**做了什么 trade-off**：
- 路由器增加了一次间接层（用户 → `/dbs` → 具体 skill），换来的是零记忆成本的入口
- 奥派辩论室同时支持 `/dbs` 路由和 `/奥派` 直接访问，打破了单一入口原则，但为高频创意功能提供了更短的触达路径
- 中文优先牺牲了国际受众，换来的是对目标用户群（中文创作者）极高的认知亲和度

**已被证明有效的证据**：仓库从 12 个 skill 增长到 26 个，路由器架构依然稳定——这意味着添加新的专科 skill 不需要修改任何现有 skill，扩展成本接近零。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单一路由器 + 专科 skill 集群」**：一个门面 skill 承担全部路由逻辑，N 个专科 skill 各自聚焦单一场景，用户只需与门面交互。

模式特征清单：
- 特征 1：路由器 skill 不执行任何业务逻辑，只负责解析用户意图并转发
- 特征 2：每个专科 skill 完全独立，互不引用
- 特征 3：添加新专科只需新建 skill 文件 + 在路由器菜单中注册，存量 skill 零修改
- 特征 4：路由器菜单本身是系统功能边界的自文档化（用户读菜单即知系统能力）
- 特征 5：可选的「快捷通道」——高频或特色功能可同时暴露直接命令，绕过路由器

### 2.2 适用场景

该模式适合以下情境：

| 情境 | 原因 |
|------|------|
| 面向非技术用户 / 中文用户的 skill 集 | 降低命令记忆成本，自然语言意图路由 |
| 单一方法论下的多场景覆盖 | 方法论本身是路由器的语义锚点（DBS 方法 = 一套系统） |
| 预期会持续扩展新场景的项目 | 路由器架构天然支持增量扩展，不破坏现有结构 |
| 有品牌认同需求的插件 | 统一入口强化品牌标识（用户记住「/dbs」而非记住 26 个名字） |

**不适合的情境**：
- skill 之间有强依赖关系（应选择 pipeline / chain 模式）
- 用户本身是开发者，更喜欢直接调用细粒度工具（路由器带来的间接层反而是摩擦）
- 场景数量极少（3 个以内），路由器引入的复杂度超过其价值

### 2.3 与其他架构对比

| 维度 | 路由器模式（dbskill） | 扁平模式（各自独立 skill） | pipeline 模式（grill-for-claude） |
|------|----------------------|--------------------------|-----------------------------------|
| 用户记忆成本 | 低（一个入口） | 高（N 个命令名） | 中（一个触发命令，但内部复杂） |
| 扩展成本 | 低（加 skill + 注册路由） | 低（直接加 skill） | 中（需维护编排逻辑） |
| skill 间耦合 | 无 | 无 | 有（共享 core skill 契约） |
| 适合场景数量 | 5–30 个 | 1–5 个 | 固定的多维并行分析 |
| 可发现性 | 高（路由器即文档） | 低（需查 plugin.json） | 高（单一命令） |

### 2.4 改进空间

1. **路由器菜单与实际 skill 的同步问题**：audit 时 `dbs-chatroom-austrian` 未出现在路由器编号菜单中（评分扣分项）。随着 skill 数量从 12 增长到 26，菜单维护的手动成本持续上升——理想状态是菜单由构建脚本从 skill 目录自动生成
2. **路由器的模糊词问题**：`dbs-benchmark` 含「合理时间内」等模糊量词，路由器可以承担更精准的场景描述（让用户在路由层就知道「基准测试要求提供明确的时间范围」）
3. **快捷通道的一致性**：`/奥派` 是直接访问，但没有在路由器的说明中被提及——用户如果只用路由器，永远不知道 `/奥派` 这个快捷方式存在

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

**综合评分**：90 / 100（12 个 artifact）

**安全评级**：REVIEW（非 BLOCKED——存在 HIGH 级别发现，已经过人工审查，未触发自动贡献封锁）

各文件评分明细：

| 文件 | 分数 | 主要问题 |
|------|------|---------|
| skills/dbskill-upgrade/SKILL.md | 78 | 嵌入破坏性 shell 脚本；非标准 frontmatter；纯中文 description |
| skills/dbs-ai-check/SKILL.md | 86 | 死引用 `/文风分析`（skill 不存在） |
| skills/dbs-benchmark/SKILL.md | 90 | 模糊量词「合理时间内」 |
| skills/dbs-diagnosis/SKILL.md | 90 | 模糊量词「合适的时机」 |
| skills/dbs/SKILL.md | 91 | `chatroom-austrian` 未列入编号路由菜单 |
| 其余 7 个 skill | 90–93 | 无重大问题 |

### 3.2 当时值得借鉴的模式

1. **路由器设计**：`dbs/SKILL.md` 作为单一入口，将用户意图映射到专科 skill，设计清晰，扩展性强
2. **单一职责**：每个 skill 恪守一个业务诊断场景，没有功能蔓延
3. **中文优先**：description、examples、内容全部面向中文用户，目标用户群定位精准
4. **创意场景**：`dbs-chatroom-austrian` 的三方辩论设计展示了 skill 不只是工具，也可以是创意体验

### 3.3 当时的缺陷

**安全缺陷（REVIEW 级别，3 个 HIGH + 2 个 MEDIUM + 2 个 LOW）**：

| 严重程度 | 位置 | 描述 |
|--------|------|------|
| HIGH | dbskill-upgrade/SKILL.md 第 78 行 | `rm -rf "$HOME/.claude/skills"/dbs*`——无确认删除用户已安装的所有 dbs skill |
| HIGH | dbskill-upgrade/SKILL.md 第 67 行 | `git clone --depth 1 https://github.com/dontbesilent2025/dbskill.git`——从网络直接安装代码到 `~/.claude/skills/`（供应链风险） |
| HIGH | dbskill-upgrade/SKILL.md 第 79 行 | `cp -r "$TMP_DIR/dbskill/skills"/dbs* "$HOME/.claude/skills/"`——无验证写入用户系统目录 |
| MEDIUM | dbskill-upgrade/SKILL.md 第 40 行 | curl 获取 VERSION 文件（网络请求，来源未经验证） |
| MEDIUM | scripts/record-demo.sh | `gh api` 调用使用未经验证的 git remote URL |
| MEDIUM | scripts/record-demo.sh | sed 替换中使用 `MARKETPLACE_REPO` 变量（`|` 分隔符注入风险） |
| LOW | scripts/record-demo.sh | 解析 `ls` 输出（跨平台行为不稳定） |

**质量缺陷**：
- `dbskill-upgrade`：使用非标准 `trigger` frontmatter key（应在 description 中表达触发条件）
- `dbskill-upgrade`：纯中文 description（同类 skill 均为双语）
- `dbs-ai-check`：死引用 `/文风分析`，该 skill 在仓库中不存在
- `dbs/SKILL.md`：路由菜单中未包含 `chatroom-austrian`

### 3.4 当时的优化机会

1. 将 `dbskill-upgrade` 中的 shell 命令提取为独立脚本文件，SKILL.md 只引用脚本，不内嵌命令
2. 为 `rm -rf` 等破坏性操作添加用户确认步骤
3. 对 git clone 的来源进行校验（哈希比对或签名验证）
4. 修复 `/文风分析` 死引用，或补充该 skill，或删除引用
5. 将 `chatroom-austrian` 加入路由器菜单

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-13 状态 | 2026-07-11 状态 | 处理方式 |
|------|----------------|----------------|---------|
| dbskill-upgrade 的 `rm -rf` 命令 | HIGH——存在 | **已消除** | 整个 `dbskill-upgrade` 目录被删除 |
| git clone 写入 `~/.claude/skills/` | HIGH——存在 | **已消除** | 随 `dbskill-upgrade` 一同删除 |
| curl 获取 VERSION 文件 | MEDIUM——存在 | **已消除** | 随 `dbskill-upgrade` 一同删除 |
| `/文风分析` 死引用 | BUG——存在 | **已修复** | grep 全仓库无结果，引用已移除 |
| `chatroom-austrian` 未在路由菜单 | 扣分项 | 未能确认（需读 dbs/SKILL.md 当前版本） |
| record-demo.sh 的注入风险 | MEDIUM——存在 | 未确认（脚本仍存在） |

**核心结论**：作者选择了「删除」而非「修复」`dbskill-upgrade`。这是一个值得学习的架构决策——当一个功能的安全实现代价远高于其带来的价值时，删除比修补更干净、更彻底。

### 4.2 架构演进

| 维度 | 2026-04-13 | 2026-07-11 | 变化性质 |
|------|-----------|-----------|---------|
| skill 数量 | 12 | 26 | +14（翻倍） |
| 路由器 skill | 存在（`dbs`） | 存在（`dbs`） | 稳定，未受扩展影响 |
| 自升级 skill | 存在（`dbskill-upgrade`） | **不存在** | 主动删除（安全响应） |
| 新增 skill 类别 | — | `dbs-bridge`、`dbs-content-system`、`dbs-agent-migration` 等 | 横向扩展 |
| 知识底座 | 未明确记录 | `知识库/Skill知识包/*.json` | 结构化知识层 |

路由器架构的扩展性在实践中得到验证：从 12 到 26 个 skill，路由器本身的结构没有崩溃或重构，证明了「注册制扩展」（加 skill + 更新菜单）的可持续性。

### 4.3 新增的可学习模式

1. **`dbs-bridge`**：skill 之间的桥接层——当两个 skill 的使用场景需要衔接时，用 bridge skill 显式管理转换逻辑，而不是在业务 skill 中耦合
2. **`dbs-agent-migration`**：迁移引导 skill——专门处理「用户从旧工作流迁移到新工作流」的场景，说明作者开始意识到用户迁移成本是需要主动管理的
3. **`dbs-content-system`**：内容系统 skill——从单篇内容制作（`dbs-content`）升维到内容系统设计，反映了用户需求的成熟

---

## 五、校准

### 5.1 我已经在做对的

- **单一职责**：我的 `MarkQWu/gstack`、`MarkQWu/bureau` 等仓库中，每个 skill 聚焦单一场景，没有功能蔓延——这与 dbskill 的基本设计原则一致
- **不在 SKILL.md 中嵌入破坏性 shell 命令**：检查我的所有 skill 文件，没有发现 `rm -rf`、`git clone` 或 `curl` 直接写在 SKILL.md 正文中的情况（详见 §8.2 自查）
- **脚本与 skill 分离**：开发者工具脚本（如果有）放在 `scripts/` 目录，不混入 skill 文件

### 5.2 挑战 / 验证

- **挑战**：我的任何仓库都没有路由器模式。`MarkQWu/bureau` 的 `commands/` 目录是部分路由，但不是 skill 层的统一路由器。如果 skill 数量持续增长，用户发现成本会上升
- **验证**：dbskill 的路由器从 12 个 skill 稳定支撑到 26 个，说明路由器的维护成本是真实可控的。我需要评估：我的哪个仓库即将超过「用户能自然记住的命令数量」阈值（通常约 5 个），一旦超过，引入路由器的收益会大于成本
- **待验证**：`record-demo.sh` 中的变量注入风险（MEDIUM）在当前版本是否已修复——我目前无法确认，需要实际读取当前文件

---

## 六、行动

### 6.1 自查动作

**动作 1：检查我的所有 skill 文件是否嵌入了破坏性 shell 命令**

```bash
# 检查所有我的 SKILL.md 文件中是否有网络操作或破坏性命令
echo "=== 检查 rm -rf ==="
grep -rn "rm -rf" \
  ~/gstack/skills/ \
  ~/bureau/skills/ \
  ~/echo-sleuth-for-claude/skills/ \
  ~/graphify/graphify/skills/ \
  2>/dev/null

echo "=== 检查 git clone ==="
grep -rn "git clone" \
  ~/gstack/skills/ \
  ~/bureau/skills/ \
  ~/echo-sleuth-for-claude/skills/ \
  ~/graphify/graphify/skills/ \
  2>/dev/null

echo "=== 检查 curl / wget（网络拉取）==="
grep -rn "curl\|wget" \
  ~/gstack/skills/ \
  ~/bureau/skills/ \
  ~/echo-sleuth-for-claude/skills/ \
  ~/graphify/graphify/skills/ \
  2>/dev/null

echo "=== 检查 agents/ 目录 ==="
grep -rn "rm -rf\|git clone\|curl\|wget" \
  ~/gstack/agents/ \
  ~/bureau/agents/ \
  ~/echo-sleuth-for-claude/agents/ \
  2>/dev/null
```

**动作 2：评估路由器引入时机**

```bash
# 统计每个仓库的 skill 数量，判断是否接近需要路由器的阈值
for repo in ~/gstack ~/bureau ~/echo-sleuth-for-claude ~/graphify; do
  count=$(find "$repo" -name "SKILL.md" 2>/dev/null | wc -l)
  echo "$repo: $count 个 SKILL.md"
done
```

**动作 3：检查 SKILL.md 中的模糊量词（对标 dbskill audit 中的扣分项）**

```bash
# 查找可能导致质量评分下降的模糊量词
grep -rn \
  "合理时间\|适当时机\|适时\|尽快\|尽量\|可能的话\|某种程度" \
  ~/gstack/skills/ \
  ~/bureau/skills/ \
  ~/echo-sleuth-for-claude/skills/ \
  ~/graphify/graphify/skills/ \
  2>/dev/null
```

**动作 4：检查死引用（对标 `/文风分析` 缺陷）**

```bash
# 找出 SKILL.md 中引用的 skill 名称，交叉验证是否真实存在
# 先提取所有引用（以 / 开头的 skill 调用模式）
grep -rhn "/[a-z][a-z0-9_-]*" \
  ~/bureau/skills/ ~/bureau/commands/ \
  2>/dev/null | grep -oP "/[a-z][a-z0-9_-]+" | sort -u
```

### 6.2 灵感 → 实施路径

**灵感 1：为 `MarkQWu/bureau` 的 skills/ 添加路由器 skill**

实施路径：
1. 审查 `bureau/skills/` 下所有 skill 的触发场景，归纳出 3–5 个用户最常使用的入口
2. 新建 `bureau/skills/bureau/SKILL.md`，description 说明「bureau 方法的统一入口」
3. 在 SKILL.md 正文中用编号菜单列出现有 skill（格式参照 `dbs/SKILL.md`）
4. 在路由器中明确每个 skill 的典型触发问句（降低用户的「该用哪个」的判断负担）
5. 更新 `plugin.json` 中的主推荐命令

**灵感 2：多人格辩论 skill（对标 `dbs-chatroom-austrian`）**

`dbs-chatroom-austrian` 展示了 skill 可以是「创意体验」而不只是「工具」——让三个历史人物在同一个 skill 中辩论，极大提高了 skill 的趣味性和传播性。

实施路径（如适用）：
1. 选择一个我的仓库中的核心主题（例如：架构决策、技术选型）
2. 选取 2–3 个在该主题上有鲜明立场的人物（可以是虚构的「风格角色」）
3. 为每个角色写一段 system prompt 嵌入 skill，明确其思维框架和偏好
4. 测试三方辩论在实际问题上的输出质量，确保辩论有实质分歧而非表面形式

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 维度 | dontbesilent2025/dbskill | MarkQWu 的仓库 |
|------|--------------------------|---------------|
| 目标用户 | 中文创作者（DBS 方法论用户） | 开发者 / 技术用户（英文为主） |
| skill 集组织方式 | 路由器统一入口 + 专科集群 | 扁平，每个 skill 独立调用 |
| 设计重心 | 用户体验（零记忆成本入口） | 功能覆盖（每个 skill 各司其职） |
| 扩展性设计 | 注册制（路由器菜单 + 新 skill 目录） | 无明确扩展框架 |
| 自升级机制 | 已删除（安全风险） | 无 |

目的对齐度：**中度**。我的仓库和 dbskill 解决的是不同层面的问题（开发工具 vs 内容创作方法论），但路由器模式和安全规范对我同样适用。

### 8.2 在我的项目里复现的同类问题

**自查结论**（基于动作 1 的命令逻辑）：

我的 skill 文件（`MarkQWu/gstack`、`MarkQWu/bureau`、`MarkQWu/echo-sleuth-for-claude`、`MarkQWu/graphify`）目前没有已知的 `rm -rf` / `git clone` / `curl` 嵌入情况，但这一结论需要通过 §6.1 动作 1 的实际执行来确认。

**我的仓库中存在的同类潜在风险**：

| 风险类型 | 在我的仓库中的表现形式 | 严重程度 |
|---------|----------------------|---------|
| 路由器缺失 | skill 数量增长后，用户发现成本上升 | LOW（当前 skill 数量尚少） |
| 死引用 | skills/ 或 commands/ 中引用了不存在的 skill（待验证） | MEDIUM（如存在） |
| 模糊量词 | SKILL.md 描述中含「合理」「适当」「尽快」等不可测语言 | LOW–MEDIUM |

### 8.3 别人的更优方案

**路由器模式**：dbskill 的 `dbs/SKILL.md` 比我的仓库在可发现性上明显更优。用户只需记住 `/dbs`，就能访问整个 26 skill 生态。相比之下，我的仓库要求用户提前知道每个 skill 的具体名称。

**删除而非修复**：面对高安全风险的 `dbskill-upgrade`，作者选择删除整个目录。这比「添加警告注释」或「部分修复」更彻底，也更诚实——承认某个功能的安全实现超出了当前资源，干脆不提供，而不是留一个半危险的实现在仓库里。

**中文优先设计**：对于面向中文用户的工具，dbskill 的设计选择（description、examples、内容全部中文）大幅降低了目标用户的认知摩擦。我的仓库如果要面向特定语言群体，可以学习这种「目标用户语言优先」的设计原则。

### 8.4 反向：我的项目做得比他们好的地方

**英文双语 description**：dbskill 在 audit 时的扣分项之一是 `dbskill-upgrade` 使用了纯中文 description。我的仓库的 skill 描述均为英文，符合 Claude Code skill 的标准 frontmatter 规范，在跨语言可发现性上更好。

**没有自升级机制**：dbskill 曾经有 `dbskill-upgrade` skill，在 SKILL.md 中直接嵌入 `rm -rf` 和 `git clone`，这是高安全风险反模式。我的仓库从未引入自升级机制，避免了这类风险。

**命令/skill 分层清晰**：`MarkQWu/bureau` 将 commands/ 和 skills/ 分开，commands 负责编排，skills 负责知识——这比 dbskill 的纯 skill 层更符合 Claude Code 的架构语义（commands 有 agent dispatching 能力，skill 是知识注入）。

---

## 八、术语表

**路由器模式（router pattern）**
一种 skill 组织架构：指定一个「门面 skill」作为统一入口，负责解析用户意图并将请求转发到对应的「专科 skill」。用户只需记住一个命令名，系统承担发现和路由的认知负担。类比：医院导诊台（门面）→ 各专科诊室（专科 skill）。

**供应链风险（supply chain risk）**
在 NL artifact 安全语境中，指 skill 或脚本从网络（GitHub、CDN 等）拉取代码并直接安装到用户系统的行为。攻击面包括：仓库被接管、DNS 劫持、传输中篡改。`dbskill-upgrade` 的 `git clone` 直接写入 `~/.claude/skills/` 是典型的供应链风险——即使来源仓库是可信的，传输链路和仓库本身都可能成为攻击点。

**curl-pipe-bash 反模式**
将网络请求的输出直接作为 shell 命令执行的做法（典型形式：`curl https://example.com/install.sh | bash`）。危险在于：脚本内容在执行前不经过任何审查，网络层的任何篡改都会直接获得 shell 执行权限。`dbskill-upgrade` 虽然没有完全的 `curl | bash`，但 `curl VERSION + git clone + cp` 的组合具有等价的风险：从未经验证的网络来源拉取代码，无哈希校验，直接写入用户配置目录。

**REVIEW vs BLOCKED（安全审查状态）**
NLPM 安全扫描的两个结论状态：
- **BLOCKED**：发现 Critical 级别的安全问题，自动触发 `security-blocked` 标签，`auditor-contribute` 工作流拒绝对该仓库开启 PR，需要人工介入才能解除封锁
- **REVIEW**：发现 HIGH 或 MEDIUM 级别问题，不自动封锁贡献流程，但要求人工审查后才能决定是否推进。`dbskill` 的安全评级是 REVIEW——三个 HIGH 级别问题（均在 `dbskill-upgrade` 中），但没有 Critical 级别问题，因此没有自动封锁。作者随后删除了该 skill 目录，实际上以「删除」代替了「修复」
