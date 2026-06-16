# itsmostafa/aws-agent-skills — 学习案例

**仓库**：https://github.com/itsmostafa/aws-agent-skills
**Stars**：1,076 | **来源**：auditor pipeline
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-16（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `template-design`, `manifest-discipline`

**xiaolai 案例**：无

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

itsmostafa/aws-agent-skills 是一个专注于 AWS 云服务的 Claude Code 技能库，收录了 18 个覆盖主流 AWS 服务的 SKILL.md 文件：api-gateway、bedrock、cloudformation、cloudwatch、cognito、dynamodb、ec2、ecs、eks、eventbridge、iam、lambda、rds、s3、secrets-manager、sns、sqs、step-functions。每个文件对应一个 AWS 服务，聚焦单一职责。截至审计日期，项目拥有 **1,076 stars**。

关键数字（2026-04-06 审计快照）：

- NLPM 综合评分：**99/100**（18 个工件）
- 安全评级：**CLEAR**（2 Medium + 4 Low，均限于自动化脚本，无 Critical）
- 完美评分（100/100）文件数：**6 个**（cloudformation、dynamodb、eks、iam、s3、secrets-manager、sns）
- 评分低于 100 的文件：**11 个**（96–98/100，唯一问题是 vague quantifier）
- 运行时 bug：**0 个**
- 质量问题总计：**13 个**（全部为模糊数量词）

### 1.2 架构剖析

**目录结构**：

```
skills/
  api-gateway/SKILL.md
  bedrock/SKILL.md
  cloudformation/SKILL.md
  cloudwatch/SKILL.md
  cognito/SKILL.md
  dynamodb/SKILL.md
  ec2/SKILL.md
  ecs/SKILL.md
  eks/SKILL.md
  eventbridge/SKILL.md
  iam/SKILL.md
  lambda/SKILL.md
  rds/SKILL.md
  s3/SKILL.md
  secrets-manager/SKILL.md
  sns/SKILL.md
  sqs/SKILL.md
  step-functions/SKILL.md

scripts/
  check-aws-updates.py        # RSS feed 检查脚本，轮询 AWS 更新公告
  generate-update-issues.py   # 从 RSS 内容生成 GitHub Issues

requirements.txt              # Python 依赖：feedparser>=6.0.0, requests>=2.28.0, PyYAML>=6.0
```

**18 个技能文件的统一结构**（批量更新于同一日期 `last_updated: "2026-01-07"`）：

```
frontmatter（name, description, last_updated）
└── 目录（ToC）
    ├── Core Concepts       # 服务核心概念
    ├── Common Patterns     # 常见使用模式
    ├── CLI Reference       # AWS CLI 命令参考（含具体命令示例）
    ├── Best Practices      # 最佳实践
    ├── Troubleshooting     # 故障排查
    └── References          # 参考链接
```

**两个自动化脚本的职责**：

- `check-aws-updates.py`：订阅 AWS 官方 RSS 源，定期检测服务文档更新
- `generate-update-issues.py`：将检测到的更新转化为 GitHub Issues，驱动人工或自动技能维护

### 1.3 设计思路 / 方法论

**核心设计哲学：「单一职责 skill 平铺 + 批量维护」**

18 个技能文件遵循完全相同的 6 节结构，在同一日期批量更新。这不是偶然——而是刻意的「模板化维护」策略：当 AWS 发布新服务文档时，更新脚本检测变化，触发 Issue，维护者对照固定模板修改对应服务的 SKILL.md，只改内容不改结构。

**解决什么问题**：AWS 有数百个服务，单个 SKILL.md 无法覆盖全部内容，但每个服务的技能文件需要保持结构一致，方便 Claude 跨服务时以相同的方式定位信息（「第 3 节一定是 CLI Reference」）。平铺结构避免了分层带来的引用复杂度——18 个 AWS 服务彼此独立，没有「lambda 调用 iam 的 helper-skill」这样的层级依赖，平铺是正确选择。

**做了什么 trade-off**：

- 优先可维护性而非分层：没有 recipe 层或 workflow 层，每个技能文件只描述单个服务能做什么，不描述如何跨服务组合。用户自行组合服务完成任务。
- 优先结构一致性而非内容个性：所有文件相同的 6 节结构，即使某服务（如 SQS）的 Best Practices 天然有「应根据场景选择合适的可见性超时」这样需要上下文的建议，也不会为其专门增加一个「配置决策指南」节。
- 自动化监测 vs 固定版本：脚本订阅 RSS 来检测更新，但 requirements.txt 使用宽松的语义版本约束（`>=`），引入了供应链风险。

**「满分技能」与「96/98 分技能」的分野**：

6 个满分技能（cloudformation、dynamodb、eks、iam、s3、secrets-manager、sns）的共同点是：这些服务的最佳实践可以用具体规则表述——「IAM 策略应遵循最小权限原则」、「S3 存储桶应启用版本控制」——不需要「视具体情况而定」。11 个 96/98 分技能在 Best Practices 节出现模糊数量词，原因是这些服务的配置确实有上下文依赖（ECS 的 health check 间隔、SQS 的可见性超时），作者试图给出建议但回避了具体数值。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「单一职责 skill 平铺 + 批量维护」多服务技能库模式**：在一个覆盖多个独立服务或域的技能库中，为每个服务建立一个单独的 SKILL.md 文件，所有文件共享完全相同的节结构，通过自动化脚本监控上游变化并批量触发维护动作。

模式特征清单：
- 特征 1：一个服务 / 一个 SKILL.md，职责边界清晰，文件间无依赖
- 特征 2：所有文件采用相同节结构（ToC → Core Concepts → Common Patterns → CLI Reference → Best Practices → Troubleshooting → References）
- 特征 3：所有文件在同一批次更新时同步修改 frontmatter `last_updated` 字段，维护日期可追溯
- 特征 4：自动化脚本（RSS 监控 + Issue 生成）将「内容过期检测」与「技能文件维护」解耦——脚本发现过期，人工（或 Claude）更新内容
- 特征 5：vague quantifier 问题集中于 Best Practices 节，CLI Reference 节几乎无模糊描述

### 2.2 适用场景

| 场景 | 是否适用 | 原因 |
|---|---|---|
| 多个彼此独立的 API/服务参考库 | ✅ 高度适用 | 本案例即为此场景，18 个 AWS 服务互相独立，平铺是最优选择 |
| 法律领域多域技能库 | ✅ 适用 | 各法律领域（合同法、劳动法、合规）彼此独立，可借鉴平铺结构 |
| 需要跨服务组合的工作流 | ⚠️ 不足 | 平铺结构无 recipe/workflow 层，跨服务组合需用户自行完成，可能缺少引导 |
| 内容频繁变化的技术文档 | ✅ 适用 | RSS 监控 + Issue 生成的自动化维护链路，专为频繁变化设计 |
| 少于 5 个服务的小型技能库 | ⚠️ 过度设计 | 自动化维护脚本的价值只在服务数量较多时才显现 |

### 2.3 与其他架构对比

| 架构类型 | 代表性仓库 | 核心差异 | 优缺点 |
|---|---|---|---|
| 单一职责平铺（本案例） | itsmostafa/aws-agent-skills | 18 文件同级，无层级，共享节结构 | 维护成本低、批量更新友好；缺乏跨服务工作流引导 |
| 显式抽象梯度 | googleworkspace/cli | api → helper → recipe → workflow 四层 | 可维护性极高，支持跨服务组合；层级复杂度高，recipe 易出现命令错误 |
| 场景专用化 | feiskyer/claude-code-settings | 每场景一工件，无分层 | 简单直观；规模化后各工件间关系模糊 |
| 单领域深度 | 多数专项技能库 | 一个领域的全栈深度 | 深度强；跨域需额外配置 |

### 2.4 改进空间

**改进点 1：vague quantifier 替换为带范围的具体建议**

- 当前问题：`ecs/SKILL.md` 的 Best Practices 节写「Configure health checks properly」和「Set appropriate deregistration delay」；`sqs/SKILL.md` 出现「appropriate」2 次（含「usually 3-5 guidance」被「appropriate」弱化）。
- 改进方式：将模糊描述替换为「Configure health check interval: 30s for production, 10s for dev-test」和「Set deregistration delay to 30s (low-traffic) or 300s (high-traffic with connection draining)」，提供具体数值范围而非「适当的」。
- 预期收益：从 96/100 升至 100/100，消除用户「那到底设多少」的困惑。

**改进点 2：scripts/requirements.txt 固定精确版本**

- 当前问题：`feedparser>=6.0.0`、`requests>=2.28.0`、`PyYAML>=6.0` 使用宽松的 `>=` 约束，允许主版本升级，引入供应链风险。
- 改进方式：固定到精确版本（如 `feedparser==6.0.11`、`requests==2.31.0`、`PyYAML==6.0.2`），并在 CI 中使用 `pip install -r requirements.txt --require-hashes`。
- 预期收益：消除 4 个 Low 安全发现（L1–L3），使安全等级从 CLEAR 保持并提升确定性。

**改进点 3：generate-update-issues.py 的 RSS 内容注入防护**

- 当前问题：脚本将 RSS 源的 `title` 和 `body` 字段直接传递给 `gh issue create`，无截断或转义，存在命令注入风险（安全发现 L4）。
- 改进方式：对 `title` 和 `body` 进行长度截断（title ≤ 200 字符，body ≤ 65535 字符），并对 shell 特殊字符进行转义或使用 `subprocess` 的列表参数形式避免 shell 解析。
- 预期收益：消除 L4 安全发现，将两个 Medium 降级至 Low 甚至清除。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**综合评分：99/100**（18 个工件，6 个满分）

| 文件 | 得分 | 主要扣分原因 |
|---|---|---|
| `skills/ecs/SKILL.md` | **96** | Best Practices 节：「Configure health checks **properly**」+ 「Set **appropriate** deregistration delay」，共 2 个 vague quantifier |
| `skills/sqs/SKILL.md` | **96** | Best Practices 节：「appropriate」出现 2 次，其中含「usually 3-5」建议被「appropriate」弱化，共 2 个 vague quantifier |
| 另外 9 个技能文件 | **96–98** | 各有 1–2 个 vague quantifier，分布于 Best Practices 或 Troubleshooting 节 |
| `cloudformation/SKILL.md` | **100** | 无扣分 |
| `dynamodb/SKILL.md` | **100** | 无扣分 |
| `eks/SKILL.md` | **100** | 无扣分 |
| `iam/SKILL.md` | **100** | 无扣分 |
| `s3/SKILL.md` | **100** | 无扣分 |
| `secrets-manager/SKILL.md` | **100** | 无扣分 |
| `sns/SKILL.md` | **100** | 无扣分 |

注意：99/100 的高分仅有 13 个模糊数量词导致，零运行时 bug，零结构性缺陷。这是一个高质量参考实现。

### 3.2 当时值得借鉴的模式

**模式 1：统一 6 节结构 + 批量 `last_updated` 管理**

- 为何有效：18 个文件都有相同的 6 节目录，Claude 在处理跨服务任务时不需要适应每个文件不同的信息组织方式——「CLI Reference 永远在第 3 节」。`last_updated` 字段批量更新于同一日期（2026-01-07），说明这批文件经过了统一的维护周期，不是零散更新。
- 如何借鉴：在建立多服务技能库时，首先确定「所有文件必须共享的节结构」，并写入 CONTRIBUTING.md 或模板文件，避免后续维护者自由发挥导致结构漂移。

**模式 2：满分技能的共同特征——可验证的具体规则**

- 为何有效：cloudformation、iam、s3 等 6 个满分技能的 Best Practices 节使用的是可验证的具体规则（「启用 S3 版本控制」、「IAM 策略附加到角色，不附加到用户」），而非「视情况而定」的建议。具体规则直接可执行，Claude 不需要额外推断。
- 如何借鉴：在写 Best Practices 节时，对每条建议自问「这条建议是否给出了 Claude 可以直接执行的判断标准？」如果答案是「要看具体情况」，那就进一步给出「情况 A → 做 X，情况 B → 做 Y」的分支表，而非写「适当的 X」。

**模式 3：CLI Reference 节零模糊描述**

- 为何有效：审计发现 13 个模糊数量词全部集中于 Best Practices 和 Troubleshooting 节，CLI Reference 节无一例外全部给出了具体的 AWS CLI 命令示例（含参数和示例值）。这表明作者对「命令参考」和「最佳实践建议」有隐性的认知区分——命令不容模糊，建议可以有弹性。
- 如何借鉴：在结构化技能文件时，将「可执行命令」和「判断建议」分属不同的节，CLI Reference 节强制要求每条命令有完整的参数示例，Best Practices 节在写有弹性的建议时至少给出取值范围。

**模式 4：自动化维护链路（RSS 监控 → Issue 生成）**

- 为何有效：技能文件的最大维护成本是「发现过期内容」。check-aws-updates.py 订阅 AWS 更新 RSS，将「内容是否过期」的检测自动化；generate-update-issues.py 自动生成待处理 Issue，驱动维护动作。维护者只需处理 Issue，不需要自己订阅 RSS 或人工扫描 18 个文件。
- 如何借鉴：对于引用外部 API 文档的技能库，可为每个关键 API 建立一个监控机制（RSS、changelog 订阅、定期 scraping），将「内容过期检测」与「内容更新」解耦，降低维护者的认知负担。

### 3.3 当时的安全发现

**Medium M1：feedparser.parse() 无超时**

- 问题描述：`scripts/check-aws-updates.py` 调用 `feedparser.parse(url)` 时没有设置超时，若 RSS 端点挂起，脚本会无限等待，耗尽 CI 资源。
- 根因：feedparser 默认无超时保护，需要调用者通过 `socket.setdefaulttimeout()` 或 `requests` 的 `timeout` 参数手动设置。
- 影响：CI 作业挂起，手动取消前不释放 runner。

**Medium M2：环境变量 GITHUB_OUTPUT 直接读取**

- 问题描述：脚本读取 `os.environ["GITHUB_OUTPUT"]` 写入 GitHub Actions 输出，如果在非 GitHub Actions 环境中执行，会引发 KeyError 或指向意外路径。
- 影响：在本地开发环境运行脚本时行为不可预测。

**Low L1–L3：unpinned semver 依赖**

- 问题描述：`requirements.txt` 中 `feedparser>=6.0.0`、`requests>=2.28.0`、`PyYAML>=6.0` 均使用 `>=` 约束，允许主版本升级。
- 影响：依赖包发布破坏性更新或恶意版本时，`pip install` 自动安装新版本，影响脚本行为。

**Low L4：RSS 衍生内容传递给 `gh issue create` 无注入防护**

- 问题描述：`generate-update-issues.py` 将 RSS 源的 `title` 和 `body` 字段直接通过 `subprocess` 或 shell 字符串拼接传递给 `gh issue create`，未截断或转义。
- 影响：恶意 RSS 源可通过构造特殊标题注入 shell 命令或创建异常长度的 Issue。

### 3.4 当时的优化机会

1. **最高 ROI（消除 vague quantifier，每处修改 1–2 行）**：
   - `ecs/SKILL.md` Best Practices：将「Configure health checks properly」改为「Set health check interval to 30s (production) / 10s (dev)，unhealthy threshold to 3」；「Set appropriate deregistration delay」改为「Set deregistration delay: 30s for stateless services, 300s for persistent-connection services」
   - `sqs/SKILL.md` Best Practices：将「appropriate visibility timeout」改为「Set visibility timeout to 6× the average message processing time (minimum 30s)」

2. **中等 ROI（消除 L4 安全发现）**：
   - `generate-update-issues.py`：对 RSS `title` 截断至 200 字符，`body` 截断至 65535 字符，使用 `subprocess` 列表参数形式避免 shell 解析

3. **中等 ROI（固定依赖版本）**：
   - `requirements.txt` 改为 `feedparser==6.0.11`、`requests==2.31.0`、`PyYAML==6.0.2`（以当时最新稳定版为准）

4. **低 ROI（添加超时保护）**：
   - `check-aws-updates.py`：在 `feedparser.parse()` 调用前添加 `socket.setdefaulttimeout(30)`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 验证方法（可在克隆后执行） | 当前状态 | 原因 |
|---|---|---|---|
| `ecs/SKILL.md` Best Practices「properly」 | `grep -n "properly" skills/ecs/SKILL.md` | ❌ **仍存在** | 当前 HEAD 未修复 |
| `ecs/SKILL.md` Best Practices「appropriate」 | `grep -n "appropriate" skills/ecs/SKILL.md` | ❌ **仍存在** | 当前 HEAD 未修复 |
| `sqs/SKILL.md` Best Practices「appropriate」×2 | `grep -n "appropriate" skills/sqs/SKILL.md` | ❌ **仍存在**（含「usually 3-5」被弱化） | 当前 HEAD 未修复 |
| `requirements.txt` unpinned semver | `grep ">=" scripts/requirements.txt` | ❌ **仍存在**（feedparser>=6.0.0 等 3 项） | 当前 HEAD 未修复 |
| `generate-update-issues.py` 无注入防护 | `grep -n "sanitiz\|truncat\|safe" scripts/generate-update-issues.py` | ❌ **仍存在**（grep 无匹配，确认无防护代码） | 当前 HEAD 未修复 |

仓库自审计后基本冻结在审计状态，无新提交可见。所有发现均持续存在。

### 4.2 架构演进

基于当前 HEAD 的可验证信息：

- 18 个技能文件的结构在审计后未发生变化——6 节统一结构保持稳定
- `last_updated: "2026-01-07"` 批量元数据保持不变，无新一轮批量更新
- 自动化脚本（check-aws-updates.py、generate-update-issues.py）无修改，安全发现持续存在
- 仓库处于「高质量静态参考」状态，不是活跃迭代的项目

### 4.3 新增的可学习模式

**「vague quantifier 聚集区」的结构性规律**：

审计数据揭示了一个可预测的规律：vague quantifier 几乎不出现在 CLI Reference 节，而集中出现在 Best Practices 节。这不是偶然——CLI 命令必须是具体的（否则无法执行），而 Best Practices 建议天然有「视具体情况」的倾向。结构本身决定了哪个节更容易引入模糊语言。

这个规律的意义在于：可以在写技能文件时**针对性地**对 Best Practices 节进行 vague quantifier 扫描，而不需要对整个文件进行全量检查。CLI Reference 节几乎可以跳过。

**「批量 last_updated」作为维护健康度指标**：

18 个文件在同一日期（2026-01-07）批量更新 `last_updated`，说明维护者执行了一次有组织的全库复查。如果 6 个月后某些文件的 `last_updated` 没有更新，说明该文件已经落后于当前维护周期——这是一个可以用脚本检测的「维护债务」信号。

---

## 五、校准

### 5.1 我已经在做对的

1. **多文件技能库的单一职责分割**：我的 MarkQWu/echo-sleuth-for-claude 将 5 个 agent 和 4 个 skill 分别维护，每个文件聚焦单一职责，与 aws-agent-skills 的「一服务一文件」原则一致。

2. **CLI Reference 节使用具体命令**：在 claude-for-legal 的技能文件中，我使用具体的查询语法示例而非「查询相关数据」类描述，这与 aws-agent-skills 的 CLI Reference 节零模糊描述的做法一致，也是满分技能的核心特征。

3. **frontmatter 完整性**：我的 3 个仓库中的技能文件均有必要的 frontmatter 字段，与 aws-agent-skills 的 18 个文件全部有 frontmatter 的「完整 frontmatter 纪律」一致。

### 5.2 挑战 / 验证

**挑战 1：我的 Best Practices 节是否存在同类 vague quantifier？**

aws-agent-skills 的 13 个 vague quantifier 全部集中于 Best Practices 节。我的 claude-for-legal 的多域技能文件中，Best Practices 节是否存在「appropriate」「relevant」「comprehensive」「proper」「meaningful」等模糊数量词？需要实际检查，而非假设没有。

**挑战 2：我的技能文件是否有 last_updated 字段？**

aws-agent-skills 对 `last_updated` 的批量维护使维护时间可追溯。如果我的 claude-for-legal 技能文件没有 `last_updated` 字段，就无法判断哪些文件可能已经过时。当库规模扩大时，缺失这个字段会让维护工作更难有序推进。

**挑战 3：满分技能 vs 96 分技能的分野——我是否能说出具体规则？**

本案例的关键发现：满分技能的 Best Practices 可以用「可验证的具体规则」表述，而非「视情况而定」。对我的 claude-for-legal，「合同条款应当完整」这类表述是 vague quantifier 风险点；「合同必须包含：当事人全称、合同标的、价款、履行期限、违约责任」才是具体可验证的规则。

---

## 六、行动

### 6.1 自查动作

**检查 Best Practices 节中的 vague quantifier（最高优先级）**：

```bash
# 在自己的技能库根目录执行
# 扫描 Best Practices 节附近的模糊数量词
grep -rn "appropriate\|relevant\|comprehensive\|proper\|meaningful\|sufficient\|adequate" \
  /path/to/claude-for-legal/**/*.md 2>/dev/null

# 仅扫描 Best Practices 节（节标题到下一节标题之间的内容）
# 先定位 Best Practices 节的行号范围
grep -n "## Best Practices\|## 最佳实践" /path/to/claude-for-legal/skills/*/SKILL.md

# 针对 aws-agent-skills 的已知问题词汇，在 ecs/sqs 等服务技能中验证
grep -n "appropriate\|properly" \
  /path/to/aws-agent-skills/skills/ecs/SKILL.md \
  /path/to/aws-agent-skills/skills/sqs/SKILL.md
```

**检查 frontmatter 中是否有 last_updated 字段**：

```bash
# 在自己的技能库中检查所有 SKILL.md 是否有 last_updated 字段
grep -rL "last_updated" /path/to/my-skill-repo/skills/ 2>/dev/null
# 无输出 = 所有文件均有该字段；有输出 = 列出缺失的文件

# 检查 last_updated 的分布（是否批量更新过）
grep -rh "last_updated" /path/to/my-skill-repo/skills/ | sort | uniq -c
# 如果所有日期都不同，说明没有执行过统一维护周期
```

**检查 requirements.txt 依赖版本固定情况**：

```bash
# 查找使用宽松约束的依赖（>= 或 ~= 或 ^）
grep -E ">=|~=|\^" /path/to/my-project/requirements.txt 2>/dev/null

# 与 aws-agent-skills 已知问题对比
cat /path/to/aws-agent-skills/scripts/requirements.txt
# 期望看到固定版本如 feedparser==6.0.11，实际可能仍是 feedparser>=6.0.0
```

**检查脚本是否对外部内容注入有防护**：

```bash
# 在 generate-update-issues.py 中检查是否有截断或转义逻辑
grep -n "sanitiz\|truncat\|safe\|escape\|shlex" \
  /path/to/aws-agent-skills/scripts/generate-update-issues.py

# 检查 gh issue create 的调用方式（列表形式更安全）
grep -n "gh issue create\|subprocess" \
  /path/to/aws-agent-skills/scripts/generate-update-issues.py
```

**统计满分技能与 96/98 分技能的比例（自我基线）**：

```bash
# 对自己的技能库运行 /nlpm:score 后，统计得分分布
# 目标：类比 aws-agent-skills 的 6/18 满分比例（33%），自己的满分比例是多少？
# 如果满分比例低于 30%，重点检查 Best Practices 节的 vague quantifier
```

### 6.2 灵感 → 实施路径

**灵感 1：为 claude-for-legal 引入统一 6 节模板并批量验证（高价值）**

- 背景：aws-agent-skills 的核心竞争力是 18 个文件「同构可预测」——Claude 和用户都知道 CLI Reference 永远在第 3 节。claude-for-legal 的多域技能文件是否也有统一节结构？如果不同文件节结构不同，Claude 在跨域任务中定位信息的一致性会降低。
- 第一步：列出 claude-for-legal 所有技能文件的节标题，统计节结构的一致性：`grep -rn "^## " /path/to/claude-for-legal/skills/ | awk -F: '{print $NF}' | sort | uniq -c | sort -rn`
- 实施路径：如果节结构不统一，参照 aws-agent-skills 的 6 节模板制定 claude-for-legal 的标准模板（Core Concepts → Common Patterns → CLI/API Reference → Best Practices → Troubleshooting → References），批量更新现有文件并写入 CONTRIBUTING.md。

**灵感 2：为 Best Practices 节的 vague quantifier 制定「具体数值化」改写规范（中价值）**

- 背景：aws-agent-skills 的 96/98 分技能告诉我们，Best Practices 节是 vague quantifier 的主要聚集区。改进方式不是删除建议，而是将「视情况而定」改为「情况 A → 数值 X，情况 B → 数值 Y」的决策表。
- 第一步：对 echo-sleuth-for-claude 的 `experience-synthesis/SKILL.md` 第 118 行的「relevant」进行改写试验：找出「relevant」所在的具体建议句，将其改为「当会话长度 > 5 轮时选择 X；当会话包含明确的领域关键词时选择 Y」。
- 实施路径：改写试验完成后，总结改写模式，作为 claude-for-legal 技能文件编写规范的附录。

**灵感 3：为 claude-for-legal 添加 RSS/changelog 监控脚本（低优先级，高长期价值）**

- 背景：claude-for-legal 引用的法律数据库（如合同数据库、案例库）的 API 会定期更新接口版本。aws-agent-skills 的 check-aws-updates.py + generate-update-issues.py 提供了一个轻量级的「内容过期检测」基础设施。
- 注意事项：直接借鉴时需避免 M1（添加 30 秒超时）和 L4（使用 `subprocess` 列表参数，对 RSS 内容截断至 200/65535 字符）的问题。requirements.txt 使用精确版本而非 `>=` 约束。
- 实施路径：先从最关键的 2–3 个法律 API 的 changelog 页面入手，建立一个简单的 `check-updates.py`，而非一次性覆盖所有数据源。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

itsmostafa/aws-agent-skills 的核心目的：为使用 AWS 服务的开发者提供一套结构统一、单一职责、可批量维护的 Claude Code 技能参考库，覆盖 18 个主流 AWS 服务的 CLI 操作和最佳实践。

| 我的仓库 | 相似度 | 原因 |
|---|---|---|
| **MarkQWu/claude-for-legal** | **高** | 两者都是「多域服务技能库」——claude-for-legal 的法律子域（commercial-legal、employment-legal 等）对应 aws-agent-skills 的 AWS 服务（ec2、s3、iam 等），都是多个独立领域的技能平铺集合。最主要区别是 claude-for-legal 规模更大、域间关联更强（多个法律域会交叉引用） |
| **MarkQWu/echo-sleuth-for-claude** | **中** | echo-sleuth 有 4 个 SKILL.md + 5 个 agent，规模较小；aws-agent-skills 18 个文件的「批量维护」经验在 echo-sleuth 规模下价值有限，但 vague quantifier 自查方法（扫描 experience-synthesis/SKILL.md 第 118 行的「relevant」）直接适用 |
| **MarkQWu/drama-workshop-skills** | **低** | drama-workshop-skills 是单一创作领域（戏剧工作坊），无多域技能平铺需求，无 API/CLI 参考节，结构参照价值有限 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷类型 | 在我的仓库的对应风险 | 验证方法 | 预估严重度 |
|---|---|---|---|
| Best Practices 节的 vague quantifier | echo-sleuth `experience-synthesis/SKILL.md` 第 118 行已知有「relevant」；claude-for-legal 多域技能文件 Best Practices 节未深查 | `grep -rn "appropriate\|relevant\|comprehensive\|proper\|meaningful" /path/to/claude-for-legal/**/*.md` | **中**（影响评分，用户无法获取可操作建议） |
| frontmatter 缺少 `last_updated` | 我的 3 个仓库技能文件是否有 `last_updated` 字段未确认 | `grep -rL "last_updated" /path/to/echo-sleuth/skills/` | **低**（影响维护可追溯性，不影响功能） |
| Python 依赖使用宽松版本约束 | claude-for-legal 和 drama-workshop-skills 的 `scripts/` 或 `requirements.txt` 是否有类似 `>=` 约束 | `grep -rn ">=" /path/to/claude-for-legal/scripts/ 2>/dev/null` | **中**（供应链风险，影响可重现性） |
| 外部内容传递给 shell 命令无防护 | claude-for-legal 的自动化脚本是否将外部 API 响应直接传递给 shell 命令 | `grep -rn "subprocess\|os.system" /path/to/claude-for-legal/scripts/` | **高**（如果存在，运行时可能被注入） |

**重要认知**：aws-agent-skills 的 99/100 分掩盖了 13 个 vague quantifier 的存在。我在评估自己的 claude-for-legal 时，不应该因为 NLPM 综合评分高就跳过 Best Practices 节的逐行检查——vague quantifier 对聚合分数影响小（每个仅扣 2–4 分），但对用户实际使用体验影响大（用户不知道「appropriate」具体是多少）。

### 7.3 别人的更优方案

**方案 1：统一 6 节结构 + 共享模板文件**

- 文件：所有 18 个 `skills/*/SKILL.md`
- 比我的方案更优之处：18 个文件的节结构完全相同，这不依赖维护者的自律，而是通过「模板文件」（一份标准 SKILL.md 模板，所有新技能文件从此模板复制）在创建阶段就强制统一结构。claude-for-legal 的多域技能文件是否有类似的模板文件驱动结构一致性？如果没有，随着域数量增加，节结构的漂移风险越来越高。
- 可以直接借鉴：在 claude-for-legal 根目录创建 `templates/SKILL.md`（包含标准 6 节框架和每节的填写说明），并在 CONTRIBUTING.md 中规定「所有新技能文件必须从 templates/SKILL.md 复制」。

**方案 2：批量 `last_updated` 维护纪律**

- 文件：所有 18 个技能文件的 frontmatter `last_updated: "2026-01-07"`
- 比我的方案更优之处：18 个文件在同一日期批量更新 `last_updated` 字段，说明维护者执行了一次有组织的全库复查，而非零散更新。这种「批量复查周期」的纪律让库的维护状态对外可见，也让内部维护者知道「上次全库复查是何时」。我的 claude-for-legal 可能缺乏类似的维护周期概念。
- 可以直接借鉴：为 claude-for-legal 建立「季度全库复查」的维护日历，每次复查后批量更新所有技能文件的 `last_updated`，并在 commit message 中注明「Q2 2026 全库复查」。

**方案 3：自动化维护链路（RSS 监控 + Issue 生成）**

- 文件：`scripts/check-aws-updates.py`、`scripts/generate-update-issues.py`
- 比我的方案更优之处：aws-agent-skills 将「内容过期检测」完全自动化——维护者不需要主动订阅 AWS 更新，脚本自动发现变化并生成 Issue。claude-for-legal 和 echo-sleuth 的技能文件引用了外部数据源，但是否有等效的「内容过期检测」机制值得检查。
- 注意：直接借鉴时需修复 M1（添加超时）和 L4（截断 + 使用列表参数），以及将 `requirements.txt` 改为固定版本。

### 7.4 反向：我的项目做得比他们好的地方

**1. MarkQWu/claude-for-legal 有显式的跨域集成文档**

aws-agent-skills 的 18 个服务技能彼此独立，没有一个文档说明「如何在一个任务中组合使用 Lambda + DynamoDB + IAM」——用户需要自行从 3 个独立文件中提取信息，自行处理跨服务的 IAM 权限配置、触发器设置等集成细节。

claude-for-legal 的 `scripts/` 和 `references/` 目录（如果有集成说明文档）提供了「跨域组合使用」的指导。法律工作中多域交叉（如劳动争议涉及 employment-legal + commercial-legal）是常见场景，集成文档的价值比 AWS 服务集成更高。

**2. MarkQWu/echo-sleuth-for-claude 有明确的 `tools:` 声明（per-agent 工具约束）**

aws-agent-skills 是纯技能库（无 agent 定义），不涉及工具声明。echo-sleuth 的 5 个 agent 每个都有显式的 `tools:` 字段，约束每个 agent 可以使用的工具范围。这是 agent 设计层面的最佳实践（最小权限原则在 agent 工具层面的应用），aws-agent-skills 作为技能库不需要也无法比较。

**3. MarkQWu/drama-workshop-skills 有 release-gate + release-manifest.json（发布门控机制）**

aws-agent-skills 没有发布门控——技能文件直接 push 到主分支即为发布状态，没有版本号管理或发布确认步骤。drama-workshop-skills 的 release-gate 和 release-manifest.json 提供了「版本发布前的最终确认」机制，在技能文件有依赖关系时（如 install scripts 依赖特定版本的 SKILL.md）能防止半发布状态。

---

## 八、术语表

| 术语 | 解释 |
|---|---|
| **frontmatter** | Markdown 文件顶部用 `---` 包裹的 YAML 元数据区域。在 Claude Code 工件中，用于声明 `name`（工件名称）、`description`（触发条件）、`last_updated`（最后更新日期）等关键属性。缺失 frontmatter 会导致 Claude Code 无法正确识别和调用工件 |
| **vague quantifier（模糊数量词）** | 技能文件中使用的无法量化的描述性词汇，如「appropriate（适当的）」「properly（正确地）」「relevant（相关的）」「comprehensive（全面的）」。这类词汇无法给 Claude 提供可执行的判断标准，降低技能文件的可操作性。NLPM 将其列为质量扣分项 |
| **single-purpose skill（单一职责技能）** | 每个 SKILL.md 文件只覆盖一个服务、领域或操作的设计原则。本案例中，每个 AWS 服务对应一个独立的 SKILL.md，服务间无引用依赖。单一职责使每个文件可以独立维护和加载，降低上下文消耗 |
| **semver（Semantic Versioning）** | 语义化版本号规范，格式为 MAJOR.MINOR.PATCH。`>=` 前缀（如 `feedparser>=6.0.0`）表示接受该版本及以上的任意版本，包括主版本升级。安全最佳实践是固定到精确版本（如 `feedparser==6.0.11`），避免依赖包自动升级引入破坏性变更或安全漏洞 |
| **manifest discipline（清单纪律）** | 在多文件技能库中，对 frontmatter 关键字段（如 `last_updated`、`name`、`description`）保持一致性和完整性的维护习惯。本案例中，18 个文件全部有完整 frontmatter 且批量同步 `last_updated`，体现了严格的清单纪律 |
| **template design（模板设计）** | 为同类工件（如多个服务的 SKILL.md）定义并强制执行统一节结构的设计方法。本案例的「6 节统一结构」（Core Concepts → Common Patterns → CLI Reference → Best Practices → Troubleshooting → References）是模板设计的典型实现 |
| **RSS feed 监控** | 订阅目标服务的 RSS 更新源，自动检测文档变化。本案例的 check-aws-updates.py 订阅 AWS 官方 RSS 源，将「内容是否过期」的检测自动化，维护者只需响应生成的 Issue 而无需主动巡查 |
| **command injection（命令注入）** | 攻击者通过在输入数据中嵌入 shell 命令，使脚本在执行外部命令时意外执行恶意代码。本案例的 L4 安全发现即为此类风险：RSS 源的 `title` 和 `body` 字段如果包含 shell 特殊字符，可能在 `gh issue create` 调用时被解析为命令 |
| **供应链风险（supply chain risk）** | 因依赖包（第三方库）的版本不固定，依赖包发布恶意版本或破坏性更新时，项目在下次 `pip install` 时自动受影响的风险。使用 `>=` 版本约束（而非精确版本）是常见的供应链风险来源 |
| **NLPM 99/100 掩盖效应** | 高聚合评分不等于零质量问题。本案例的 99/100 分来自 18 个文件的平均，13 个 vague quantifier 对聚合分数影响极小（每个约扣 0.1–0.2 分），但对用户实际使用体验的影响较大。不应以综合评分代替对单个问题的逐项核查 |
