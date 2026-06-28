# qwibitai/nanoclaw — 学习案例

**仓库**：https://github.com/qwibitai/nanoclaw
**Stars**：27917 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-06-28（基于 Audit 数据）
**主题标签**：`router-channels`, `curl-pipe-bash-risk`, `security-gate`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于 Audit 快照）

### 1.1 仓库概览

qwibitai/nanoclaw 是一个面向多渠道通讯整合的 Claude Code 插件集合，Stars 高达 27917，是本轮审计批次中规模最大的仓库之一。项目的核心价值主张：**以 Claude Code skill 作为渠道适配器**，将 Discord、Slack、Telegram、WhatsApp、WeChat、Signal、Teams、Matrix、iMessage、Webex、Email（Resend）等十几个即时通讯平台以统一的 skill 格式接入。

从 Audit 快照（2026-04-25）看，仓库包含 56 个 NL 制品，其中约 44 个 SKILL.md 文件、3 个 CLAUDE.md（分布于容器、主分组、全局分组），另有若干 hook 与配置文件。NL 质量评分为 **81/100**，处于中等偏上水准，但存在少数严重拖分文件。安全评级为 **CRITICAL**——因 `init-onecli/SKILL.md` 中的 curl-pipe-to-shell 模式触发最高风险等级，导致仓库整体被安全门阻断，无法进入自动贡献流程。

由于克隆目标时代理受阻（网络封锁），本案例完全基于 Audit 快照数据，无法访问当前 HEAD。

### 1.2 架构剖析

nanoclaw 采用**渠道即 skill（channel-as-skill）**的扁平分发模式。每个通讯渠道对应 `.claude/skills/add-<channel>/` 目录下的一个 SKILL.md，渠道之间结构高度一致，形成可批量安装的 skill 矩阵：

```
.claude/skills/
├── add-discord/SKILL.md
├── add-slack/SKILL.md
├── add-telegram/SKILL.md
├── add-whatsapp/SKILL.md
├── add-whatsapp-cloud/SKILL.md
├── add-opencode/SKILL.md
├── add-wechat/SKILL.md
├── add-signal/SKILL.md
├── add-teams/SKILL.md
├── add-matrix/SKILL.md
├── add-linear/SKILL.md
├── add-github/SKILL.md
├── add-gchat/SKILL.md
├── add-resend/SKILL.md
├── add-imessage/SKILL.md
├── add-webex/SKILL.md
├── add-codex/SKILL.md
├── add-emacs/SKILL.md
├── add-vercel/SKILL.md
└── add-parallel/SKILL.md   ← 最低分（50），完全缺失 frontmatter

container/skills/
├── self-customize/SKILL.md
├── agent-browser/SKILL.md
├── slack-formatting/SKILL.md
└── welcome/SKILL.md

groups/
├── main/CLAUDE.md           ← 分组主记忆（65分）
└── global/CLAUDE.md

container/CLAUDE.md          ← 容器级项目记忆
```

除渠道 skill 外，项目还维护了三个 CLAUDE.md 文件，分别管理容器级、主分组级和全局分组级的项目记忆，构成**三层上下文架构**：容器（container）→ 主分组（main group）→ 全局分组（global group）。

最高分制品：`CLAUDE.md`（容器级，92 分）和 `.claude/skills/get-qodo-rules/SKILL.md`（92 分）。
最低分制品：`add-parallel/SKILL.md`（50 分，完全无 [frontmatter](#术语表)）和 `customize/SKILL.md`（62 分，引用已失效的 v1 路径）。

### 1.3 设计思路 / 方法论

nanoclaw 的核心设计命题是**"一套 Claude Code skill 驱动全渠道消息路由"**。这一设计有三个值得注意的选择：

**选择一：渠道即 skill（channel-as-skill）**——将每个通讯渠道的接入逻辑封装为独立 SKILL.md，而非编写一个统一的"消息路由"入口。好处是每个渠道可以独立安装、独立更新、独立测试；代价是 20+ 个渠道 skill 形成了大量结构相似的制品，维护一致性（尤其是 [frontmatter](#术语表) 字段完整性）的压力倍增。

**选择二：三层 CLAUDE.md 上下文分层**——container / main-group / global-group 的分层意味着记忆的作用域被精细切割。全局级上下文对所有实例可见，主分组上下文仅对该分组内的 agent 可见，容器上下文最为本地化。这是一种成熟的上下文隔离思路，但当前 `groups/main/CLAUDE.md` 的 v1/v2 混合问题（65 分）说明分层的价值尚未被充分利用。

**选择三：最高分 skill 的范式（get-qodo-rules/SKILL.md，92 分）**——该 skill 具备完整的版本字段、`allowed-tools` 声明和明确的触发条件，是项目内部质量最高的范本。它证明作者知道"好 skill 应该长什么样"，但这一标准在 20+ 渠道 skill 中没有被统一执行。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：渠道路由矩阵（Channel Router Matrix）**

核心特征：
- **同构 skill 批量生成**：每个渠道对应一个结构相同的 SKILL.md，使得用户可以 `claude skill install nanoclaw add-slack` 这样的命令单独安装某渠道支持，无需引入整个项目
- **分层上下文记忆**：三个 CLAUDE.md 文件在不同作用域维护上下文，而非将所有记忆塞入单一文件
- **容器级 skill 补充**：`container/skills/` 下的 skill（self-customize、agent-browser、slack-formatting、welcome）处理跨渠道的公共逻辑，避免在每个渠道 skill 中重复

### 2.2 适用场景

- 需要同时支持多个平台/渠道，且各平台的集成逻辑差异较小（可以用统一 skill 模板覆盖）
- 用户群体希望按需选择渠道，不愿安装整个 monorepo
- 项目有明确的"核心功能 + 扩展渠道"分层，容器 skill 承担核心，渠道 skill 承担扩展

### 2.3 与其他架构对比

| 维度 | nanoclaw（渠道矩阵） | 单一路由 skill | drama-workshop-skills（非结构化集合） |
|------|---------------------|---------------|--------------------------------------|
| skill 数量 | 20+ 渠道 skill | 1 | 不定，按主题散落 |
| 安装粒度 | 单渠道可独立安装 | 整体安装 | 整体安装 |
| 维护一致性压力 | 高（20+ 文件需同步更新） | 低 | 中（无强制结构） |
| [frontmatter](#术语表) 一致性风险 | 高（实测有文件完全缺失） | 低 | 中 |
| 上下文分层 | 三层 CLAUDE.md | 无 | 无 |
| 批量分发适合度 | 优秀 | 差 | 一般 |

### 2.4 改进空间

1. **frontmatter 一致性缺口**：20+ 渠道 skill 中，约 20 个缺少 `output-format` 字段，`add-parallel/SKILL.md` 更是完全无 frontmatter。说明项目缺少 skill 模板或 lint 检查，导致部分渠道 skill 未按最高标准完成。建议引入 `.claude/templates/channel-skill-template.md` 作为新渠道 skill 的强制起点。

2. **v1/v2 版本混用未清理**：`customize/SKILL.md` 和 `groups/main/CLAUDE.md` 仍引用 v1 路径，而 v2 已将这些路径删除或移位。批量渠道 skill 存在被 v1/v2 迁移污染的系统性风险，需要一次全库的 [cross-reference](#术语表) 扫描。

3. **安全设计缺陷未隔离**：`init-onecli/SKILL.md` 的 curl-pipe-to-shell 属于 onboarding 流程的安全设计问题，而非个别 bug。应将 onboarding 类 skill 的安全标准与渠道 skill 分开审视，并为 onboarding skill 增加人工确认步骤。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：81 / 100**

| 文件 | 分数 | 主要问题 |
|------|------|---------|
| add-parallel/SKILL.md | 50 | 完全缺失 frontmatter（name、description 均无），skill 无法被安装或按名引用 |
| customize/SKILL.md | 62 | 引用不存在的 v1 文件路径（src/ipc.ts、src/router.ts、src/db.ts 等） |
| groups/main/CLAUDE.md | 65 | v1/v2 混合内容，引用 /workspace/ipc/、registered_groups.json（v1 路径）、register_group MCP 工具（v2 已删除） |
| x-integration/SKILL.md | 72 | 版本标签过时（标注 "Compatibility: NanoClaw v1.0.0"） |
| CLAUDE.md（容器级） | 92 | 优秀——结构清晰，供应链规则完整 |
| get-qodo-rules/SKILL.md | 92 | 最高分 skill——有版本、allowed-tools、明确触发条件 |

整体 81 分的拖分来源高度集中：去掉最低分的三个文件（50、62、65），剩余制品的平均分应在 85 分以上。这说明项目存在明显的**质量双峰**：头部制品（CLAUDE.md、get-qodo-rules）接近满分，但尾部制品（add-parallel、customize、groups/main）显著拉低了整体评分。

### 3.2 当时值得借鉴的模式

**1. get-qodo-rules/SKILL.md 的三要素完整性**

该 skill 是本项目内部质量最高的范本，具备：
- **版本字段**：明确声明 skill 的版本，便于用户判断兼容性
- **allowed-tools 声明**：精确列出 skill 执行时允许调用的工具，限制了 skill 的权限边界
- **触发条件（triggers）**：明确说明"什么情况下该 skill 应当被激活"，减少 AI agent 的歧义判断

这三个要素使得 get-qodo-rules 在 56 个制品中脱颖而出，应成为所有渠道 skill 的强制规范。

**2. 容器级 CLAUDE.md（92 分）的供应链规则设计**

项目主 CLAUDE.md（容器级）得分 92，其核心优势是**供应链安全规则的完整性**——明确声明了依赖安装的安全边界，这在存在 curl-pipe-to-shell 风险的项目中尤为有价值（尽管执行层面安全漏洞依然存在）。CLAUDE.md 作为容器最高层级的上下文，应当承担"全局安全政策声明"的职责，nanoclaw 在这一点上做法正确。

**3. 渠道 skill 的模块化分发结构**

20+ 渠道 skill 每个独立于 `.claude/skills/add-<channel>/` 子目录，使得用户可以选择性安装特定渠道。这一模式相比"将所有渠道逻辑塞入单一 SKILL.md"优越——后者在用户不需要某些渠道时也必须加载全部内容，前者则实现了按需安装。

### 3.3 当时的缺陷

**缺陷 BUG-01：add-parallel/SKILL.md 完全缺失 frontmatter**

`add-parallel/SKILL.md` 既无 `name` 字段也无 `description` 字段——即完全没有 YAML [frontmatter](#术语表)。这使得：
- skill 无法被 Claude Code 按名称引用（`claude skill install nanoclaw add-parallel` 行为未定义）
- `add-parallel` 是项目中与安全问题叠加的文件（SEC-HIGH-02：sed 替换将用户输入插入正则），无 frontmatter 意味着 skill 的权限边界完全未声明，安全风险进一步放大

**缺陷 BUG-02：customize/SKILL.md 引用 v1 文件路径**

`customize/SKILL.md` 引用了多个在 v2 中已被删除或移位的路径：`src/ipc.ts`、`src/router.ts`、`src/db.ts` 等。这些引用是 [cross-reference](#术语表) 死链——AI agent 在执行该 skill 时会尝试访问不存在的文件，导致工具调用失败或产生误导性错误。

**缺陷 BUG-03：groups/main/CLAUDE.md 的 v1/v2 混合内容**

`groups/main/CLAUDE.md` 是一个 v1 时代文件，在 v2 迁移后没有被同步更新，导致以下问题并存：
- 引用 `/workspace/ipc/`（v1 路径，v2 已不存在）
- 引用 `registered_groups.json`（v1 数据文件，v2 已删除）
- 引用 `register_group` MCP 工具（v2 中该工具已不存在）

agent 在读取该 CLAUDE.md 后获得的上下文是错误的，可能导致工具调用失败或对项目结构产生错误假设。

### 3.4 当时的优化机会

1. **QUAL-01：约 20 个渠道 skill 缺失 output-format frontmatter 字段**——添加 `output-format` 字段是低成本、高价值的改进，可以统一 AI agent 的输出结构，减少渠道间的行为不一致性

2. **QUAL-02：模糊量词（vague-quantifier）分散于多个文件**——"appropriate"、"relevant"、"some"、"various"、"sufficient" 等词语在批量渠道 skill 中反复出现，说明写作习惯层面缺乏对 [模糊量词](#术语表) 的主动规避意识

3. **QUAL-03：x-integration/SKILL.md 的版本标签过时**——标注 "Compatibility: NanoClaw v1.0.0" 的 skill 在 v2 环境中运行时，用户无法判断该 skill 是否仍然有效，应更新为当前版本或移除兼容性标注

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

由于克隆目标时网络代理受阻（HTTP 403），无法访问当前 HEAD。以下状态均为**无法验证（克隆受阻）**。

| 缺陷 | Audit 日期（2026-04-25） | 当前状态 | 说明 |
|------|--------------------------|----------|------|
| BUG-01：add-parallel/SKILL.md 完全缺失 frontmatter | 存在 | 无法验证（克隆受阻） | 无法访问当前仓库，无法确认是否已补全 frontmatter |
| BUG-02：customize/SKILL.md 引用 v1 文件路径 | 存在 | 无法验证（克隆受阻） | 无法确认 v1 路径是否已在 v2 中修正或文件是否已删除 |
| BUG-03：groups/main/CLAUDE.md v1/v2 混合内容 | 存在 | 无法验证（克隆受阻） | 无法确认混合内容是否已清理，v1 引用是否已替换为 v2 路径 |
| SEC-CRITICAL：init-onecli/SKILL.md curl-pipe-to-shell | 存在 | 无法验证（克隆受阻） | curl-pipe-to-shell 模式是 onboarding 设计决策，修复需要架构调整，短期可能仍然存在 |

### 4.2 架构演进

暂无——无法访问当前 HEAD，无法判断架构是否有重大变化。基于 Audit 快照（2026-04-25），渠道矩阵架构和三层 CLAUDE.md 分层已经建立。若未来能访问仓库，重点关注：

- 是否引入了统一的渠道 skill 模板，解决 20+ 文件的 frontmatter 一致性问题
- groups/main/CLAUDE.md 的 v1/v2 混合内容是否已清理
- SEC-CRITICAL 的 onboarding 安全问题是否通过架构调整（人工确认步骤）得到缓解

### 4.3 新增的可学习模式

暂无——无法访问当前 HEAD。若未来克隆成功，重点关注以下潜在演进：若 BUG-01（无 frontmatter）已被修复，修复方案本身（是补全模板还是删除该 skill）具有学习价值。

---

## 五、校准

### 5.1 我已经在做对的

基于 nanoclaw 的高分实践和低分教训，以下做法与其最高分制品的范式一致：

1. **frontmatter 字段完整性意识**：get-qodo-rules/SKILL.md（92 分）证明版本 + allowed-tools + triggers 三要素是高质量 skill 的必要条件。在写新 skill 时先写 frontmatter、后写正文，可以强制要求自己在一开始就完成元数据声明。

2. **版本迁移时同步更新 CLAUDE.md**：BUG-03 的根本原因是 v1→v2 迁移后 `groups/main/CLAUDE.md` 没有被同步更新。在做版本迁移时，将 CLAUDE.md 文件列入强制检查清单，可以避免同类问题。

3. **容器级 CLAUDE.md 作为全局安全政策声明**：nanoclaw 的容器级 CLAUDE.md（92 分）包含供应链规则声明，这一做法值得在自己的项目中复现——把安全边界写进最高层级的 CLAUDE.md，而非分散在各 skill 文件中。

### 5.2 挑战 / 验证

1. **挑战：批量同构 skill 的一致性维护**：nanoclaw 有 20+ 渠道 skill，其中约 20 个缺少 `output-format`，说明"批量生成容易、批量维护难"。若我的项目（如 drama-workshop-skills）也有多个结构相似的 skill，应当建立一个 skill lint 脚本，在每次新增 skill 后自动检查必要 frontmatter 字段是否完整。

2. **验证点：shell injection 风险的自查**：nanoclaw 的 SEC-HIGH-02（add-parallel/SKILL.md 中 sed 替换将用户输入插入正则）提示：在 skill 正文中如果包含 shell 命令模板，应当明确检查是否存在用户输入被直接拼接进命令的情况。在我的项目中包含 shell 命令的 skill 文件中运行 grep 检查是必要的自查步骤。

---

## 六、行动

### 6.1 自查动作

以下命令可在任意 Claude Code 项目根目录执行，检测与 nanoclaw 类似的常见问题：

**命令 1：检测缺失 frontmatter 的 SKILL.md（复现 BUG-01）**
```bash
# 找出没有以 --- 开头的 SKILL.md 文件（即完全缺失 frontmatter）
for f in $(find . -name "SKILL.md" 2>/dev/null); do
  head -1 "$f" | grep -q "^---" || echo "NO FRONTMATTER: $f"
done
```

**命令 2：检测 SKILL.md 中的死链引用（复现 BUG-02、BUG-03）**
```bash
# 提取 SKILL.md 和 CLAUDE.md 中本地文件引用，检查目标是否存在
grep -rn --include="*.md" -oP '(?<=\()\.{0,2}/[^\)]+\.(ts|js|json|md|sh)(?=\))' \
  .claude/ skills/ container/ groups/ CLAUDE.md 2>/dev/null \
  | while IFS=: read file lineno ref; do
    basedir=$(dirname "$file")
    target="$basedir/$ref"
    [ -f "$target" ] || echo "DEAD LINK: $file:$lineno -> $ref"
  done
```

**命令 3：检测模糊量词（复现 QUAL-02）**
```bash
grep -rn --include="*.md" \
  -e "\bappropriate\b" \
  -e "\brelevant\b" \
  -e "\bsome\b" \
  -e "\bvarious\b" \
  -e "\bsufficient\b" \
  -e "\bwhen possible\b" \
  -e "\bwhen applicable\b" \
  .claude/ skills/ container/ groups/ CLAUDE.md 2>/dev/null \
  | grep -v "^Binary"
```

**命令 4：检测 curl-pipe-to-shell 安全风险（复现 SEC-CRITICAL）**
```bash
grep -rn --include="*.md" \
  -e "curl.*|.*sh" \
  -e "curl.*|.*bash" \
  -e "wget.*|.*sh" \
  -e "curl.*pipe" \
  . 2>/dev/null
```

### 6.2 灵感 → 实施路径

**灵感 1：为渠道矩阵类项目建立统一 skill 模板**

若我的项目（如 drama-workshop-skills）需要新增多个同构 skill，可以参照 nanoclaw 的渠道 skill 结构建立模板：

1. 创建 `.claude/templates/skill-template.md`，包含所有必要 frontmatter 字段（name、description、version、allowed-tools、triggers、output-format）
2. 每次新增 skill 时从模板 copy，而非从空文件开始
3. 在 pre-commit hook 中运行命令 1，强制检查 frontmatter 完整性

**灵感 2：将 onboarding 脚本的安全风险替换为审计后的安装方式**

nanoclaw 的 SEC-CRITICAL 提示：如果项目有 onboarding 流程且涉及 curl 安装，应改为：
1. 使用官方包管理器（brew、apt、pip）替代 curl-pipe-to-sh
2. 若必须用 curl，先下载到临时文件，让用户审查内容后再手动执行
3. 在 skill 正文中明确标注"该命令从互联网下载并执行代码，请在受信任网络环境下运行"

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 用户仓库 | 与 nanoclaw 的相似度 | 对齐维度 |
|----------|---------------------|----------|
| MarkQWu/drama-workshop-skills | 高 | 都是 Claude Code skill 集合，都有多个主题相似的 SKILL.md 文件；nanoclaw 的渠道矩阵模式与 drama-workshop-skills 的 skill 集合在"批量分发 skill"这一目标上高度对齐 |
| MarkQWu/gstack | 中 | 都是多 skill 平铺结构，gstack 的 23 个工具 skill 与 nanoclaw 的 20+ 渠道 skill 在"同构 skill 批量维护"问题上具有相同挑战 |
| MarkQWu/graphify | 低 | nanoclaw 无知识图谱功能，但 skill 架构模式（frontmatter 一致性）同样适用 |
| MarkQWu/bureau | 低-中 | bureau 的 knowledge capture 理念与 nanoclaw 的三层 CLAUDE.md 上下文分层在"分级记忆管理"这一点有交叉 |
| MarkQWu/echo-sleuth-for-claude | 低-中 | nanoclaw 的 groups/CLAUDE.md 机制（对话分组记忆）与 echo-sleuth 挖掘过去对话的目标有一定共鸣：两者都在处理"历史上下文如何被后续 agent 利用"的问题 |

最直接相似：`drama-workshop-skills`（同为 skill 集合，同有批量分发需求）和 `gstack`（同构 skill 平铺维护挑战相似）。

### 8.2 在我的项目里复现同类问题

由于目标克隆和用户仓库克隆均不可用，无法直接执行检查。以下是可在本地仓库目录中运行的具体 grep 命令：

**针对 MarkQWu/drama-workshop-skills（最相关）**：
```bash
# 假设路径为 ~/drama-workshop-skills，检测缺失 frontmatter 的 SKILL.md
cd ~/drama-workshop-skills
for f in $(find . -name "SKILL.md" 2>/dev/null); do
  head -1 "$f" | grep -q "^---" || echo "MISSING FRONTMATTER: $f"
done

# 检测缺失 output-format 字段的 SKILL.md（复现 QUAL-01）
grep -rL "output-format" $(find . -name "SKILL.md" 2>/dev/null)

# 检测模糊量词密度
grep -rn --include="SKILL.md" \
  -e "\bappropriate\b" -e "\brelevant\b" -e "\bvarious\b" \
  -e "\bsufficient\b" -e "\bsome\b" \
  . 2>/dev/null | wc -l
```

**针对 MarkQWu/gstack（次相关）**：
```bash
# 假设路径为 ~/gstack，检测 skill 间的死链引用（复现 BUG-02）
cd ~/gstack
grep -rn --include="*.md" -oP '(?<=\()\.{0,2}/[^\)]+\.(md|ts|js|json)(?=\))' \
  . 2>/dev/null | while IFS=: read file lineno ref; do
    basedir=$(dirname "$file")
    [ -f "$basedir/$ref" ] || echo "DEAD LINK: $file:$lineno -> $ref"
  done
```

建议将上述检查加入 pre-commit hook，以防 nanoclaw 类型的 frontmatter 缺失和死链引用积累。

### 8.3 别人的更优方案（值得借鉴的）

**优点 1：get-qodo-rules/SKILL.md（92 分）的三要素规范——版本 + allowed-tools + triggers**

nanoclaw 项目内部质量最高的 skill 具备完整的版本声明、工具权限声明和触发条件声明。这三个元素使 skill 的行为边界完全可预测。若 drama-workshop-skills 的 skill 缺少 `allowed-tools` 声明，skill 执行时拥有的工具权限是模糊的——Claude Code 可能允许 skill 调用任意工具，存在不必要的权限扩张风险。

**建议**：以 get-qodo-rules/SKILL.md 的 frontmatter 结构为模板，逐一为 drama-workshop-skills 中的 skill 补充三要素。

**优点 2：nanoclaw 的 20+ 渠道 skill 形成统一分发矩阵，优于 drama-workshop-skills 的非结构化平铺**

nanoclaw 将所有渠道 skill 放在 `.claude/skills/add-<channel>/` 的统一命名空间下，用户一眼就能理解安装路径（`add-discord`、`add-slack` 等），且渠道 skill 彼此独立，互不干扰。若 drama-workshop-skills 的 skill 平铺在根目录而无命名约定，用户难以通过目录结构理解 skill 的用途分类。

**建议**：参照 nanoclaw 的渠道前缀命名（`add-<channel>`），为 drama-workshop-skills 引入明确的功能前缀命名约定（如 `write-<genre>`、`edit-<mode>`），使 skill 的用途在文件名层即可见。

### 8.4 反向：我的项目做得比他们好的地方

**我可能做得更好的地方 1：output-format 字段的覆盖率**

nanoclaw 约 20 个渠道 skill 缺少 `output-format` frontmatter 字段，这是 QUAL-01 质量问题的根源。若 drama-workshop-skills 的 skill 在创建时就统一包含了 `output-format` 字段（声明输出是 Markdown、纯文本还是结构化 JSON），则在这一维度上做得比 nanoclaw 更好。

**我可能做得更好的地方 2：安全设计的保守性**

drama-workshop-skills 作为创意写作辅助 skill 集合，不涉及 curl 安装、sudo 命令或 shell 替换，因此不存在 nanoclaw 的 CRITICAL 安全风险。从安全设计角度，"不引入不必要的可执行表面"本身就是一种优势——简单的 skill 集合（只做 LLM 提示，不做系统操作）的安全审计天然比 nanoclaw 这类涉及系统操作的项目容易通过。

**我需要继续关注的差距**：drama-workshop-skills 目前的 README/文档覆盖很可能比 nanoclaw 更好（27917 Stars 的项目有强大的社区贡献动力），但 skill 内部的 frontmatter 完整性是否达到 nanoclaw 的最高分水准（get-qodo-rules/SKILL.md 的三要素），需要实际检查后方可判断。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **artifacts（制品）** | 在 NLPM 语境下，指用自然语言写成的、供 AI agent 或人类开发者使用的结构化文本文件，如 SKILL.md、CLAUDE.md、hook 配置文件等 |
| **skill** | 在 Claude Code 体系中，技能定义文件（通常为 SKILL.md），描述 AI agent 应如何完成某类任务的声明式指令文档 |
| **frontmatter** | Markdown 文件开头以 `---` 包裹的 YAML 元数据块，用于声明文件类型、版本、allowed-tools、output-format 等机器可读属性；缺少 frontmatter 会导致 skill 无法被 Claude Code 按名称引用 |
| **allowed-tools** | frontmatter 中声明 skill 执行时允许调用的工具列表；未声明则 skill 可调用任意工具，存在不必要的权限扩张风险 |
| **triggers** | frontmatter 中声明 skill 应在什么条件下被激活的字段；有明确 triggers 的 skill 可以减少 AI agent 的歧义判断 |
| **output-format** | frontmatter 中声明 skill 输出格式的字段（如 markdown、plaintext、json）；缺失时 AI agent 输出格式不受约束，渠道间行为可能不一致 |
| **cross-reference（交叉引用）** | skill 或文档文件中指向其他文件的引用；NLPM Audit 会验证所有引用目标是否真实存在；死链引用会导致 AI agent 工具调用失败 |
| **vague-quantifier（模糊量词）** | 不提供可操作定义的形容词或副词，如 "appropriate"、"relevant"、"various"、"sufficient"；NLPM 将其视为质量缺陷，因为 AI agent 无法对其作出一致判断 |
| **curl-pipe-to-shell** | 形如 `curl -fsSL <url> | sh` 的 shell 命令模式；从互联网下载脚本并直接以 shell 执行，属于 NLPM 安全扫描的最高风险（Critical）模式，因为下载内容在执行前无法被人工审查 |
| **security-gate（安全门）** | NLPM 审查流程中在 NL 质量评分之前执行的安全扫描步骤；若发现 Critical 风险则标记 `security-blocked`，阻断后续贡献流程，需要人工介入清除 |
| **router-channels（渠道路由矩阵）** | 将每个通讯渠道的接入逻辑封装为独立同构 skill、统一命名（add-<channel>）的架构模式；支持按渠道独立安装，是 nanoclaw 的核心架构模式 |
| **shell injection** | 当用户输入被直接拼接进 shell 命令（如 sed 替换的正则参数），攻击者可通过构造特殊输入执行任意命令；add-parallel/SKILL.md 的 SEC-HIGH-02 即属此类 |
| **v1/v2 混合（version bleed）** | 版本迁移后，旧版本的路径、工具名、数据文件引用残留在新版本的文档中，导致 AI agent 获得错误上下文；nanoclaw 的 BUG-02、BUG-03 均属此类 |
| **onboarding skill** | 负责项目初始化、环境安装配置的 skill；此类 skill 天然涉及系统级操作（curl、sudo），安全标准应高于普通功能 skill，必须包含人工确认步骤 |
| **无法验证（克隆受阻）** | 当网络代理阻断导致无法克隆目标仓库时，对缺陷当前状态的标注；意味着既不能确认已修复，也不能确认仍然存在 |
| **NLPM** | Natural-Language Programming Manager，本项目的名称；提供 NL 制品的质量评分、一致性检查、安全扫描、自动修复等工具链 |
| **CLAUDE.md** | Claude Code 识别的项目级配置文件，用于向 AI agent 传递项目上下文、规则约束和工作流指引；nanoclaw 在三个作用域（container、main group、global group）各维护一个 CLAUDE.md |
| **spec-first** | 先写规格文档、后实现代码的开发方式；customize/SKILL.md 的 v1 路径死链很可能源于 spec-first 写法中预先引用了规划中的路径 |
