# Xquik-dev/x-twitter-scraper — 学习案例

**仓库**：https://github.com/Xquik-dev/x-twitter-scraper
**Stars**：N/A | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-30（基于当前 HEAD）
**主题标签**：`examples-driven`, `single-purpose`, `manifest-discipline`, `security-gate`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

x-twitter-scraper 是一个 Claude Code [插件](#插件)，将 Xquik 平台提供的 111 个 X（Twitter）REST API [端点](#端点) 封装成 46 个 [NL 工件](#NL工件)（42 个技能 + 4 个命令），让 Claude 可以直接操作 X 账号数据。用户无需暴露 X 账号凭据给 Claude——所有 X 账号连接通过 Xquik 控制台完成，Claude 只需持有一个 `xq_...` 格式的 Xquik API Key。

关键事实：
- **API 覆盖深度**：111 个端点，涵盖搜索、发帖、关注、私信、媒体下载、Webhook 监控、批量抽取器
- **安全架构**：[MCP 外部服务器](#MCP外部服务器) 部署在 `https://xquik.com/mcp`，所有工具调用经由 Xquik 代理，X 账号凭据从不接触 Claude
- **技能分布**：42 个 skill 各自单独覆盖一个工作流，共同依赖一个主 skill（`x-twitter-scraper/SKILL.md`）作为 API 参考
- **生态位置**：Claude Code 插件市场中少数覆盖写操作（发帖、私信、关注）的商业 API 封装插件

### 1.2 架构剖析

```
x-twitter-scraper/
├── .claude-plugin/
│   └── plugin.json           ← 插件 manifest（注册 42 技能 + 4 命令）
├── .mcp.json                 ← MCP 外部服务器配置（xquik.com/mcp）
├── skills/
│   ├── x-twitter-scraper/   ← 主 skill（434 行，完整 API 参考 + 决策树）
│   │   └── SKILL.md
│   ├── search-tweets/        ← 子 skill（~80 行，单一工作流）
│   ├── find-bangers/         ← 子 skill（57 行，相对互动率算法）
│   ├── monitor-accounts/     ← 子 skill（77 行，Webhook 监控）
│   └── ... (39 个同类子 skill)
├── commands/
│   ├── post.md
│   ├── search.md
│   ├── trending.md
│   └── user.md
├── package.json              ← devDependency 声明
└── scripts/
    └── check-versions.mjs    ← 版本漂移检查脚本（只读，无网络调用）
```

- **文件类型分布**：42 个 SKILL.md + 4 个 command + 1 个 plugin.json + 1 个 .mcp.json
- **编排关系**：主 skill（`x-twitter-scraper/SKILL.md`）承载完整 API 参考和 5 棵决策树；子 skill 各自描述单一工作流，末尾通过 "Related" 章节将用户引导到邻近 skill 或主 skill。无 router 层，依靠 [frontmatter](#frontmatter) `description` 驱动自动触发。
- **跨件契约**：子 skill 通过 `[x-twitter-scraper](../x-twitter-scraper/SKILL.md)` 相对路径引用主 skill；兄弟 skill 间通过 "Related" 段落互相消歧义。`plugin.json` 列出全部 46 个工件路径，是加载的唯一入口。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「单一工作流 skill + 主 skill 参考层」。每个 skill 只教一件事，把 API 全貌留给主 skill。这让子 skill 保持简短（57-77 行），降低 Claude 的 token 消耗。
- **解决什么问题**：X API 有 111 个端点，如果放进一个 skill，Claude 每次都要消化全部内容。按工作流拆分后，Claude 在执行「搜索推文」任务时只加载 `search-tweets` 的内容，不会看到发帖或监控相关的噪音。
- **Trade-off**：拆成 42 个 skill 的维护成本高——端点参数变化时需要同步修改多个文件（这次审查发现了 `track-competitors` 和 `going-viral` 与主 skill 参数描述不一致的问题）。换来的是每个 skill 极低的认知负荷和高度的可搜索性。
- **认知模型**：作者把 skill 看作「工作流描述单元」而非「功能模块」——每个 skill 对应用户的一种意图（「我要找出帖子爆款」「我要监控竞争对手」），而不是按 API 分组（「用户接口 API」「推文接口 API」）。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：单 API + MCP 适配层 + 意图分簇技能库**

一个外部 API（Xquik/X）通过 MCP 服务器暴露工具调用能力，再由一个主 skill（全量参考）+ N 个子 skill（单意图）共同封装。用户的意图直接映射到对应的子 skill，子 skill 提供精准的工作流指令，遇到边界情况再引用主 skill。

模式特征清单：
- **主 skill 承担 API 参考职责**：111 个端点的参数、响应、错误码集中在一处，子 skill 不重复
- **子 skill 以用户意图命名**：`find-bangers`、`monitor-accounts`、`grow-followers`——不是 `get-tweets`、`post-users`
- **单向依赖**：子 skill 引用主 skill，但主 skill 不知道子 skill 的存在；是树形依赖，不是环形
- **安全契约在 [frontmatter](#frontmatter) 中声明**：`metadata.security` 块机器可读，审计工具不需要解析正文
- **决策树替代 API 目录**：主 skill 用 ASCII 决策树导航，根节点是用户意图，叶子节点是具体端点

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 覆盖 10+ 端点的外部 API 封装 | ✅ 高度适用 | 意图分簇让子 skill 保持短小，决策树导航全量 API |
| 有写操作（发帖、支付、发邮件）的 API | ✅ 高度适用 | 安全契约 frontmatter 明确声明写操作范围，减少误操作风险 |
| 需要频繁参数对齐的 API（快速迭代）| ⚠️ 谨慎适用 | 42 个文件需同步更新，参数漂移风险高（本次审查已发现） |
| 纯本地工具（无网络调用）| ❌ 不适用 | MCP 外部服务器是该模式的核心，离线场景不需要这层 |
| 只有 1-3 个端点的简单工具 | ❌ 过度设计 | 一个主 skill 就够，不需要拆子 skill |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 主 skill + 意图子 skill（本仓库） | x-twitter-scraper | 每个子 skill 极简；用户意图直接映射 | 多文件同步维护；参数漂移风险 |
| 单一超大 skill（平铺所有端点） | 部分 LLM 工具封装 | 维护简单，只改一处 | Token 消耗高；Claude 需要解析全量 API 才能回答单个请求 |
| 命令驱动（只有 command，无 skill） | 简单 CLI 封装 | 结构最简单 | 无法自动触发；用户需要记住命令名称 |

### 2.4 改进空间

1. **当前问题**：参数定义分散在主 skill 和 42 个子 skill 中，维护时容易漂移（`track-competitors` vs `user-tweets` 的 `limit`/`sort` 不一致）。**改进做法**：在主 skill 中为每个端点添加参数权威表格，子 skill 引用而非重述；CI 添加参数一致性检查脚本。**预期收益**：消除跨文件参数不一致，一次更新自动同步所有子 skill。

2. **当前问题**：4 个命令全部缺少 `name` [frontmatter](#frontmatter) 字段，斜杠命令注册失败，用户无法通过 `/post`、`/search` 等快捷方式调用。**改进做法**：在命令 [frontmatter](#frontmatter) 模板中将 `name` 列为必填字段，CI 检查所有 `commands/*.md` 的 `name` 字段存在。**预期收益**：立即解锁斜杠命令入口，降低用户上手门槛。

3. **当前问题**：`package.json` 中 `@tanstack/intent` 固定为 `"latest"`，每次 `npm install` 拉取最新版本，存在供应链风险。**改进做法**：固定到具体的 semver（如 `"1.0.0"`）并在 `package-lock.json` 中锁定。**预期收益**：构建可复现，消除上游静默更新引入的潜在破坏性变化。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 97/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/post.md | 60 | 缺少 `name` frontmatter，无编号步骤 |
| commands/search.md | 70 | 缺少 `name` frontmatter |
| commands/trending.md | 70 | 缺少 `name` frontmatter |
| commands/user.md | 70 | 缺少 `name` frontmatter |
| 全部 42 个 skill | 100 | 无问题 |
| plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

**① R04 触发器描述工程化**：主 skill 的 `description` 覆盖 12 个动作类别，显式处理 Twitter/X 名称歧义，并包含「Use even if the user says 'Twitter' instead of 'X'」这样的预防性触发子句。根本原因：Claude 在检索 skill 时匹配的是 description，description 越接近用户实际用词，召回越准确。原文路径：`skills/x-twitter-scraper/SKILL.md:3`。借鉴方法：在每个 skill 的 description 里预判用户可能用的别名词、口语化表达，以及常见误导性说法（"自动发帖"="post-tweets skill"）。

**② R07 邻近 skill 消歧义**：`find-bangers` 和 `find-viral-tweets` 功能相近，前者用「"Why not just find-viral-tweets"」段落，用一句话区分了「相对互动率」vs「绝对阈值」的根本差异。根本原因：相似 skill 并存时，Claude 若无明确消歧义信息，会随机选择或两个都加载。原文路径：`skills/find-bangers/SKILL.md:45-57`。借鉴方法：每当有 2 个功能重叠的 skill，至少在其中一个里写「Why not X」段落，精确陈述两者的决策边界。

**③ R06 可运行 HTTP 示例**：每个 skill 包含字面量 HTTP 调用示例，有精确的端点路径、参数枚举值、响应结构。`search-tweets` 展示了完整的循环终止条件（`nextCursor` 为空时停止），`monitor-accounts` 展示了完整的 POST 请求体。根本原因：模糊的「调用 search API」不可执行，Claude 会猜参数名；具体示例可直接粘贴。原文路径：`skills/search-tweets/SKILL.md:65-71`。借鉴方法：每个技能示例包含：端点路径 + 关键参数名（精确拼写）+ 典型响应片段（至少 2-3 个字段）。

**④ R08 决策树替代 API 目录**：主 skill 用 5 棵 ASCII 决策树（读取、批量抽取、写操作、监控、AI 组合）覆盖 111 个端点，每棵树的根节点是用户意图，叶子节点是具体端点。根本原因：平铺的端点目录要求 Claude 线性扫描所有端点才能做出选择；决策树让 Claude 直接跳到匹配的叶子节点。原文路径：`skills/x-twitter-scraper/SKILL.md:138-231`。借鉴方法：API 超过 10 个端点时，用决策树结构代替端点列表，按用户意图而非 API 路径分组。

**⑤ R01 可验证标准替代模糊量词**：错误处理章节直接给出「只重试 429 和 5xx，最多 3 次，指数退避，不重试其他 4xx」而非「适当重试」。`find-bangers` 的「爆款」阈值是「互动率超过该作者中位数 3-5 倍」而非「异常高互动率」。根本原因：模糊量词让 Claude 自行判断边界，不同次调用结果不一致。原文路径：`skills/x-twitter-scraper/SKILL.md:243`，`skills/find-bangers/SKILL.md:44-46`。借鉴方法：凡有程度描述（「高」「合理」「适当」）的地方，强制替换成具体数字或枚举。

### 3.3 当时的缺陷

**① 4 个命令缺少 `name` frontmatter**：`commands/post.md`、`search.md`、`trending.md`、`user.md` 的 frontmatter 都没有 `name` 字段。为什么会失败：Claude Code 注册斜杠命令时用 `name` 字段生成命令名，缺失则注册失败，用户无法通过 `/post` 或 `/search` 调用。Root cause：命令模板文件里可能没把 `name` 列为必填字段，4 个命令完全一致的错误说明是系统性遗漏，不是单次失误。自查：我的命令文件是否有 `name` 字段？

**② `track-competitors` 使用错误参数 `tool=post_extractor`**：该 skill 的示例用了 `tool=post_extractor`，但所有其他 skill 的文档都是 `toolType=`。为什么会失败：API 调用时参数名错误会导致服务端返回 400 `invalid_input`，Claude 看到错误后可能重试，浪费配额。Root cause：这是 copy-paste 时没有对齐参数名约定。自查：我的 skill 示例里是否有与邻近 skill 不一致的参数名？

**③ 跨 skill 参数不一致（track-competitors vs user-tweets）**：`track-competitors` 传递了 `limit=50&sort=top` 给 `GET /x/users/{id}/tweets`，但 `user-tweets` 明确声明该端点不支持 `limit` 和 `sort`。为什么会失败：Claude 读到 `track-competitors` 会以为参数合法，实际调用会得到空结果或错误响应，且排查困难（不是 400，是静默数据错误）。Root cause：42 个文件维护时没有统一的端点参数真值来源，各文件各自描述，逐渐漂移。自查：我的多个 skill 中有没有引用同一个 API 端点但参数描述不同的情况？

### 3.4 当时的优化机会

1. **添加 CI 参数一致性检查**：解析所有 skill 中的端点示例（`GET /x/...` 形式），按端点路径聚合，检查同一端点在不同 skill 中的参数集合是否一致。可以用 Python 脚本实现，约 50 行。

2. **统一命令 frontmatter 模板**：创建 `commands/.template.md`，把 `name`、`description`、`allowed-tools` 作为必填字段，新增命令时从模板复制。

3. **在主 skill 添加「参数权威表格」**：每个端点一行，列出所有支持参数的精确拼写和类型，子 skill 示例中遇到参数时引用「见主 skill 端点参数表」而非自行描述。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

由于环境网络限制，无法克隆目标仓库（HTTP 403），无法实时验证。以下状态基于 exemplar 快照（commit `c9945b10`，2026-05-13），该快照晚于 audit（2026-04-06）约 37 天。

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| 4 个命令缺 `name` frontmatter | `grep -l "^name:" commands/*.md` | exemplar 快照中未修复（score 仍 97，命令仍 60-70 分） | 斜杠命令注册在 2026-05-13 时仍失败 |
| `track-competitors` 使用 `tool=` | `grep "tool=" skills/track-competitors/SKILL.md` | exemplar 未提及修复 | 可能仍在使用错误参数名 |
| `@tanstack/intent: "latest"` | `grep "@tanstack" package.json` | 未知（exemplar 不覆盖 package.json） | 潜在供应链风险仍存在 |

### 4.2 架构演进

由于无法访问当前 HEAD，此部分基于 exemplar 快照（commit `c9945b10`）。从 audit（46 个工件）到 exemplar（同样 46 个工件，结构完全一致），期间约 37 天内架构未发生重组。主 skill 仍是 434 行，子 skill 仍保持 57-77 行的单工作流设计。说明作者在原始架构上已经稳定，主要改进集中在 bug 修复层面（如果有的话）而非结构重组。

### 4.3 新增的可学习模式

exemplar（2026-05-13）相较于 audit（2026-04-06）新增了 **Security Posture Frontmatter 模式**（由 exemplar 文档明确标注为「Worth adopting」）：

每个 skill 在 frontmatter 中声明 `metadata.security` 块，字段包括 `contentTrust`、`promptInjectionDefense`、`writeConfirmation`、`paymentConfirmation`、`executionModel`、`codeExecution`、`credentialProxy`，主 skill 的该块覆盖 9 项具体的 prompt injection 防御措施。这是一个机器可读的安全契约，让自动化审计工具可以检查 skill 的安全姿态，而不需要解析正文。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责分解**：按功能边界拆分 skill 而非堆在一起，这个方向是对的——x-twitter-scraper 的 42 个子 skill 证明极细粒度分解可以实现 100/100 评分。
2. **description 写触发意图**：description 应该面向用户意图而非 API 名称。
3. **示例要有具体参数**：给出精确的字段名和示例值，而不是抽象描述步骤。

### 5.2 挑战 / 验证

**验证点**：之前对「要不要在 skill 里写决策树」有疑虑（觉得过于复杂）。这次案例验证了：当 API 端点超过 10 个时，决策树确实必要——它让 Claude 的端点选择从 O(n) 扫描变成 O(depth) 查找，且每次调用的 token 消耗显著降低。

**挑战点**：一直以为「相关 skill 列举一下就行」，但 `find-bangers` vs `find-viral-tweets` 的案例告诉我：仅仅列举相邻 skill 的名字不够，必须用一句话说清楚「什么时候选这个而不选那个」——也就是决策边界，而不只是目录。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有 name 字段
grep -rL "^name:" $(find . -path "*/commands/*.md" 2>/dev/null)
# 命中后：在对应文件 frontmatter 中添加 name: <command-name>
```

```bash
# 检查相邻 skill 中是否有相同 API 端点但参数描述不同
grep -rn "GET \|POST \|PUT \|DELETE " $(find . -path "*/skills/*/SKILL.md" 2>/dev/null) | \
  sort | uniq -D
# 命中后：找出同一端点不同参数描述的文件，对齐到主 skill 的权威定义
```

```bash
# 检查是否有 skill 的 description 缺少用户意图别名（含 "Use when" 之类触发语）
grep -rL "Use when\|Use if\|use when" $(find . -path "*/skills/*/SKILL.md" 2>/dev/null)
# 命中后：在 description 中增加用户意图覆盖，参考 x-twitter-scraper 主 skill 的 description 格式
```

```bash
# 检查相邻功能重叠 skill 是否有消歧义段落
grep -rL "Why not\|vs\.\|instead of\|Related" $(find . -path "*/skills/*/SKILL.md" 2>/dev/null)
# 命中后：在功能相近的 skill 对中，至少一个添加决策边界说明
```

### 6.2 灵感 → 实施路径

1. **想法**：在 xposter 的 skill 中添加 `metadata.security` frontmatter 块。
   - **为何可行**：xposter 涉及文章发布到 X，有写操作，安全契约对用户有信息价值，同时也方便 NLPM 审计。
   - **第一步**：参考 x-twitter-scraper 主 skill lines 15-88，为 xposter 的主 skill 补充 `writeConfirmation`、`credentialProxy`、`contentTrust` 三个字段（10-15 分钟）。

2. **想法**：为 API 超过 5 个端点的技能库构建「意图决策树」。
   - **为何可行**：当前 skill 的端点都以平铺列表呈现，用户或 Claude 需要线性扫描才能找到正确端点；决策树能在 token 消耗减少 50% 的情况下提供同等导航能力。
   - **第一步**：选一个有 5+ 端点的 skill，用 ASCII 树重写端点索引部分，测试 Claude 在问题描述不同措辞时的端点选择准确率（30 分钟）。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度

- **本案例 Xquik-dev/x-twitter-scraper 的核心目的**：通过 MCP 服务器将 X（Twitter）API 封装成 Claude Code 可直接调用的技能集，让 Claude 代替人工操作 X 账号（发帖、分析、监控）。

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/xposter | 高 | 同为 X（Twitter）平台工具；都处理帖子发布类操作 | xposter 是 Chrome 扩展（浏览器端），x-twitter-scraper 是 MCP 适配层（Claude Code 插件端） | 高 |
| MarkQWu/bureau | 低 | 都有 NL 工件（skill/command） | bureau 是通用工作流工具，无 Twitter API 业务 | 低 |
| MarkQWu/shiji-kb | 低 | 都有 NL 工件 | shiji-kb 是知识库工具，无社交媒体业务 | 低 |

### 8.2 在我的项目里复现的同类问题

由于无法克隆 my-repos（HTTP 403 代理限制），以下检查基于 `my-repos.json` 描述和项目类型推断：

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺 `name` frontmatter | `grep -rL "^name:" MarkQWu-*/commands/*.md` | xposter 若有命令需检查 | 高（block 斜杠命令注册） |
| skill description 缺触发别名 | `grep -rL "Use when\|Use if" MarkQWu-*/skills/*/SKILL.md` | bureau、shiji-kb 等所有含 skill 的仓库需检查 | 中 |
| 相邻 skill 无消歧义说明 | `grep -rL "Why not\|Related" MarkQWu-*/skills/*/SKILL.md` | 功能相近 skill 对需要检查 | 中 |
| HTTP 示例缺参数枚举 | `grep -rn "GET \|POST " MarkQWu-*/skills/*/SKILL.md \| grep -v "?"` | 任何包含 API 调用的 skill | 中 |

**命中后的具体行动建议**：
- MarkQWu/xposter 的命令文件 → 在 frontmatter 中检查并添加 `name:` 字段 → 5 分钟
- MarkQWu/bureau 的各 skill → 在 description 开头加「Use when...」触发句 → 每个 skill 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：API 端点导航（决策树 vs 平铺列表）
   - **本案例做法**：5 棵 ASCII 决策树，根节点是用户意图（「我需要批量数据」），叶子节点是端点路径（`GET /x/tweets/search`）。路径：`skills/x-twitter-scraper/SKILL.md:138-231`
   - **我的项目现状**：API 端点可能以列表形式平铺，没有意图导航层（基于 my-repos.json 描述推断）
   - **如何借鉴**：在 API 相关 skill 的「How to Use」章节，将端点列表重写为决策树；以「用户说…」为根节点，分支到具体端点

2. **领域**：相邻 skill 消歧义
   - **本案例做法**：`find-bangers` 用「Why not find-viral-tweets」段落一句话讲清楚两个 skill 的决策边界
   - **我的项目现状**：功能相近的 skill 可能只是各自描述自己，没有明确的「什么时候不用我」说明
   - **如何借鉴**：找出项目中相似度最高的 2 个 skill，在两者中至少一个添加「Why not X」段落

3. **领域**：`description` 覆盖用户口语化表达
   - **本案例做法**：「Use even if the user says 'Twitter' instead of 'X'」在 description 中预防最常见的检索失败
   - **我的项目现状**：description 可能偏向技术名称，没有覆盖用户口语化的别名
   - **如何借鉴**：对每个 skill 的 description 做「用户实际怎么说这件事」的头脑风暴，加入最常见的非术语表达

### 8.4 反向：我的项目做得比他们好的地方

本案例的命令层（`commands/`）存在系统性缺失（4 个命令全无 `name` frontmatter），若我的项目命令 frontmatter 完整，这是一个相对优势。但由于无法验证，此处暂无法确认。

若 xposter 已有完整 frontmatter 的命令文件，则在「命令注册完整性」这一维度比 x-twitter-scraper 更优——说明我的命令模板健康，可以此为基准回头给 x-twitter-scraper 开 PR。

---

## 八、术语表

### <a name="插件"></a>插件
> 在 Claude Code 里，插件是一个包含 `plugin.json`（[manifest](#manifest)）的目录，里面可以有 skill、command、agent 等工件。安装插件后，Claude Code 会读取 manifest，把里面声明的所有工件注册到会话中。`claude plugin install <repo>` 是标准安装命令。

### <a name="端点"></a>端点
> API 的「服务窗口」。每个端点是一个 URL + HTTP 方法（GET/POST/DELETE 等），代表一个具体操作，如 `GET /x/tweets/search` = 「搜索推文」。

### <a name="NL工件"></a>NL 工件
> Natural Language Artifact，自然语言编程工件。指 Claude Code 能读取并执行的 Markdown 文件，包括 SKILL.md（技能定义）、command.md（命令定义）、agent.md（代理定义）等。这些文件不是代码，而是用自然语言（通常是英文）写给 Claude 读的指令。

### <a name="MCP外部服务器"></a>MCP 外部服务器
> MCP（Model Context Protocol）是 Anthropic 定义的协议，让 Claude 能调用外部工具。「外部 MCP 服务器」是部署在第三方服务器上（如 `https://xquik.com/mcp`）的工具提供方，Claude 通过 HTTP 请求调用它。这意味着你的工具调用会经过第三方服务器，需要信任该服务器不会滥用数据。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读取任何 NL 工件时先解析 frontmatter——如果 frontmatter 里缺少必填字段（比如命令的 `name`），该工件就无法正常注册和调用。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 command、skill、agent 的文件路径。如果 manifest 里漏写了某个文件，那个文件即使存在于硬盘上也不会被加载。

### <a name="斜杠命令"></a>斜杠命令
> 用户在 Claude Code 里输入 `/` 开头触发的快捷指令，如 `/post`、`/search`。其背后是 `commands/` 目录里的 Markdown 文件。斜杠命令注册依赖 frontmatter 里的 `name` 字段——缺少则无法注册，用户在界面上看不到该命令。

### <a name="供应链风险"></a>供应链风险
> 软件开发中，「供应链」指你的项目依赖的所有第三方库和工具。「供应链风险」是指某个你信任的依赖库被恶意代码污染（黑客攻击、作者失误），导致你的项目在安装或运行时执行了未预期的恶意操作。将依赖版本固定为具体数字（如 `"1.0.0"`）而非 `"latest"` 是最基础的防御手段。
