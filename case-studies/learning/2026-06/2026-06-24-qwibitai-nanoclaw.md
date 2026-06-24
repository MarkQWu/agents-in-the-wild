# qwibitai/nanoclaw — 学习案例

**仓库**：https://github.com/qwibitai/nanoclaw
**Stars**：27,917 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-06-24（基于 Audit 报告，无法实时克隆）
**主题标签**：router-channels, curl-pipe-bash-risk, vague-quantifier, template-design, model-pinning

---

## 一、理解（基于 Audit 报告）

### 1.1 仓库概览

NanoClaw 是 qwibitai 构建的 AI 消息 Agent 平台，核心能力是让同一套 AI agent 逻辑跨 10+ 消息渠道（WhatsApp、Discord、Slack、Telegram、Signal、Teams、Matrix、WeChat、iMessage 等）并行运行。27,917 星说明这是同类工具中影响力最高的之一。

关键事实：
- 仓库包含 56 个 NL artifact，以 `.claude/skills/add-<channel>/SKILL.md` 形式组织，每个渠道一个技能文件
- 根目录 `CLAUDE.md` 得分 92/100，是全仓最高分文件，内容全面，包含明确的供应链规则
- 仓库历经 v1（文件 IPC + 每渠道独立模块）到 v2（SQLite 数据库 + 从分支安装模型）的完整架构重写，残留大量 v1 引用
- 有一个 CRITICAL 级别安全发现（curl-pipe-to-shell）和两个 HIGH 级别发现，导致 Security 结论为 CRITICAL
- NL 质量评分 81/100，三个离群值（add-parallel: 50、customize: 62、groups/main: 65）将加权均值从 84 拉低到 81

### 1.2 架构剖析

```
nanoclaw/
├── CLAUDE.md                        # 根配置，92/100，含供应链规则
├── package.json                     # prepare: husky（中风险 postinstall hook）
├── scripts/
│   └── run-migrations.ts            # line 86: execFileSync argv 路径注入
├── .claude/
│   └── skills/
│       ├── add-whatsapp/SKILL.md    # 渠道安装技能（~40 个，同类结构）
│       ├── add-discord/SKILL.md
│       ├── add-slack/SKILL.md
│       ├── add-telegram/SKILL.md
│       ├── add-signal/SKILL.md
│       ├── add-parallel/SKILL.md    # ⚠️ BUG-01：无 YAML frontmatter，得分 50
│       ├── customize/SKILL.md       # ⚠️ BUG-02：v1 路径引用，得分 62
│       ├── init-onecli/SKILL.md     # ⚠️ CRITICAL：curl-pipe-sh
│       ├── convert-to-apple-container/SKILL.md  # ⚠️ HIGH：无确认的 sudo pfctl
│       ├── x-integration/SKILL.md   # 版本号过期
│       ├── x-integration/scripts/setup.ts       # Chrome session 持久化到磁盘
│       ├── debug/SKILL.md           # ⚠️ QUAL-02：含 appropriate/relevant 等模糊词
│       ├── manage-channels/SKILL.md # ⚠️ QUAL-02：模糊量词
│       ├── setup/SKILL.md           # ⚠️ QUAL-02：模糊量词
│       ├── get-qodo-rules/SKILL.md  # 最佳实践：92/100，有 version、allowed-tools、triggers
│       └── [其余 ~25 个渠道 skill]  # ⚠️ QUAL-01：缺 output-format 字段
├── container/
│   └── skills/
│       └── agent-browser/SKILL.md  # 92/100，结构清晰
└── groups/
    └── main/
        └── CLAUDE.md               # ⚠️ BUG-03：v1/v2 混合内容，得分 65
```

- **文件类型分布**：40+ 渠道安装 SKILL.md / 3 个容器技能 / 多个 groups CLAUDE.md / 根 CLAUDE.md / TypeScript 脚本 / package.json
- **编排关系**：用户通过 `/nanoclaw:add-<channel>` 触发对应 SKILL.md；根 CLAUDE.md 设置全局规则；`groups/*/CLAUDE.md` 为渠道群组提供局部上下文；container skills 由容器环境调用
- **跨件契约**：渠道 skill 之间理论上通过共同的 SQLite 数据库和统一的 install-from-branch 模型协作，但 v1/v2 混合残留使得 groups CLAUDE.md 中的引用路径与实际 v2 代码脱节

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「写一套 agent，跑遍所有渠道」——通过将渠道差异封装进独立 SKILL.md，让同一个 AI 推理核心无需感知底层通信协议
- **解决什么问题**：企业和开发者不想为每个消息平台维护独立的 AI bot 代码库；NanoClaw 把渠道适配层声明化（每个 skill 告诉 Claude 如何安装该渠道），推理层复用
- **做了什么 trade-off**：用「模板化 skill 批量复制」换取快速覆盖渠道广度，代价是所有 skill 的质量天花板被复制的模板锁死——模板少了 `output-format`，40 个 skill 全部缺失；v1 migration 时只更新了部分文件，另一部分文件维护了 v1 引用
- **反映什么认知模型**：作者把 Claude Code skills 视为「平台安装说明书」而非「AI 推理规约」——skills 更像 README 安装指南，而不是明确约束 AI 行为边界的规范文件。这解释了为什么 `output-format` 和 `allowed-tools` 的覆盖率低——作者没有把这些字段纳入「安装说明书」的必要组成部分

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「渠道路由器 + 模板复制型 Skill 集合」

核心特征：
- 特征 1：每个集成渠道有且仅有一个 SKILL.md，文件命名遵循 `add-<channel>` 统一前缀
- 特征 2：根 CLAUDE.md 设全局规则，`groups/*/CLAUDE.md` 设局部渠道群组规则，形成两级配置继承
- 特征 3：容器 skills（`container/skills/`）与主 skills 物理分离，区分部署环境
- 特征 4：技能粒度对齐「安装动作」而非「推理任务」——每个 skill 的主要内容是安装步骤，而不是行为约束
- 特征 5：有最优实践参照（`get-qodo-rules/SKILL.md`，92/100），但未将其变成全仓模板

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要快速覆盖 10+ 渠道的消息 agent | ✅ 核心用例 | 模板复制策略是最快覆盖渠道广度的方式 |
| 渠道配置需要频繁变更和独立维护 | ✅ 适用 | 每渠道独立 SKILL.md，改一个不影响其他 |
| 需要精细控制 AI 在每个渠道的输出格式 | ⚠️ 谨慎 | ~40 个 skill 缺 `output-format`，AI 行为边界模糊 |
| 历经架构大版本迁移（v1→v2）的项目 | ❌ 当前失效 | BUG-02/BUG-03 说明模板复制策略在 v1→v2 迁移中容易产生批量残留引用 |
| 有安全审查要求的企业部署 | ❌ 需先修复 | CRITICAL curl-pipe-sh + 两个 HIGH 发现，需先解决安全问题 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 渠道路由器 + 模板 skill（nanoclaw） | qwibitai/nanoclaw | 覆盖渠道广度快，每渠道独立，改动不串扰 | 模板质量上限直接决定全仓质量上限；v1→v2 迁移产生批量残留 |
| 多维 Agent 分层 + 多提供商路由 | nyldn/claude-octopus | 有独立批判层，hook 横切面安全 | 维护面（157 个 artifact）更大，一致性更难保证 |
| 任务驱动型单层 skill 集合 | numman-ali/n-skills | 简单，每个 skill 独立自包含 | 无渠道路由概念，不适合多平台集成场景 |
| 企业级配置标准化 | feiskyer/claude-code-settings | 配置标准化，可复用 | 无 skill/agent 编排，覆盖的问题域不同 |

### 2.4 改进空间

1. **当前问题**：约 40 个渠道 skill 缺 `output-format` 字段（QUAL-01）。**改进做法**：将 `get-qodo-rules/SKILL.md` 的 frontmatter 结构升级为全仓模板，批量给渠道 skill 补充 `output-format: markdown`（或对应格式）。具体步骤：先确定各渠道 skill 的输出期望格式，然后用脚本批量注入 frontmatter 字段，再逐一人工确认。**预期收益**：消除约 40 个 skill 的输出格式模糊问题，AI 行为可预期性从「随机」提升到「受约束」。

2. **当前问题**：BUG-01 中 `add-parallel/SKILL.md` 完全没有 YAML frontmatter，导致该 skill 无法被 Claude Code 按名称安装（SKILL.md 必须有 frontmatter 才能被识别为有效 artifact）。**改进做法**：参照 `get-qodo-rules/SKILL.md` 的 frontmatter 结构，给 `add-parallel/SKILL.md` 补充完整的 `name`、`description`、`version`、`allowed-tools` 字段，同时修复 HIGH 级别注入风险（`sed -i.bak` 模式中的用户输入未转义）。**预期收益**：该 skill 从「不可安装」变为「可正常使用」，同时消除注入风险。

3. **当前问题**：`init-onecli/SKILL.md` 的 CRITICAL curl-pipe-to-shell 模式（`curl -fsSL onecli.sh/install | sh`）和 `convert-to-apple-container/SKILL.md` 的无确认 `sudo pfctl` 修改系统包过滤。**改进做法**：curl-pipe-sh 改为「下载 → 校验 SHA256 → 用户确认 → 执行」四步流程；sudo 命令前加「向用户展示将执行的命令并等待确认」步骤。**预期收益**：CRITICAL 安全发现降级为 LOW，HIGH 发现降级为 INFO。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-25 当时得分 **81/100**（Security: CRITICAL，Artifacts: 56，三个离群值拖低均值）。

| 文件 / 类别 | 得分 | 主要扣分原因 |
|---|---|---|
| `CLAUDE.md`（根） | 92 | 全仓最高；内容完整，供应链规则清晰 |
| `get-qodo-rules/SKILL.md` | 92 | 最佳 skill；有 version、allowed-tools、triggers |
| `container/skills/agent-browser/SKILL.md` | 92 | 结构清晰 |
| ~40 个渠道 add-* skill | ~84（均值） | 缺 output-format；三个离群值未计入 |
| `customize/SKILL.md` | 62 | BUG-02：v1 路径引用（-15 penalty） |
| `groups/main/CLAUDE.md` | 65 | BUG-03：v1/v2 混合内容（-20 penalty） |
| `add-parallel/SKILL.md` | 50 | BUG-01：无 frontmatter（-50 penalty） |
| **全仓加权均值** | **81** | 三个离群值将均值从 84 拉低到 81 |

### 3.2 当时值得借鉴的模式

1. **根 CLAUDE.md 的供应链规则设计**：根目录 `CLAUDE.md`（92/100）明确规定了供应链安全策略，包含对第三方依赖的约束。这是少数把供应链规则写进 CLAUDE.md 的仓库之一，值得直接借鉴——在自己的 `CLAUDE.md` 里加一节「第三方依赖规则」。

2. **`get-qodo-rules/SKILL.md` 的 frontmatter 完整性**：这个 skill 有 `version` 字段（说明 skill 的版本）、`allowed-tools` 字段（限制工具访问面）、`triggers` 字段（声明何时自动触发）。这三个字段组合构成一个「可观测、可控、可维护」的 skill 最低标准。

3. **`container/skills/` 物理分离容器技能**：把容器环境下的 skills 单独放在 `container/skills/` 目录，而不是混在主 `.claude/skills/` 里。这种环境隔离做法让部署边界清晰，容器外不会意外触发容器内的 skill。

4. **`add-<channel>` 命名前缀一致性**：40+ 个渠道 skill 统一使用 `add-<channel>` 前缀，形成可预测的 skill 命名空间。用户不需要记忆每个渠道的 skill 名，只需要知道前缀规则。这是一种「约定优于配置」的命名设计。

5. **两级 CLAUDE.md 配置继承**：根 `CLAUDE.md` 设全局规则（适用所有渠道），`groups/*/CLAUDE.md` 设局部规则（仅适用于该渠道群组）。这种两级继承结构比单一全局配置更灵活，比每个文件都写完整规则更易维护。

### 3.3 当时的缺陷

1. **BUG-01（-50 分）：`add-parallel/SKILL.md` 无 YAML frontmatter**

   **表象**：文件直接以 `# Add Parallel AI Integration` 标题开头，没有 `---` 包围的 YAML 块。这个 skill 无法被 Claude Code 按名称安装，实际处于「存在于仓库但永远不能使用」的状态。

   **根本原因**：这个文件极可能是从一个普通 Markdown 文档（README 风格）手动转化为 SKILL.md 时，作者忘记添加 frontmatter 的结果。SKILL.md 和普通 Markdown 文档的最大区别就是 frontmatter——没有 frontmatter 的 SKILL.md 对 Claude Code 来说不是一个「技能」，只是一个普通的文本文件。更深的原因：仓库没有 CI 门控检查「所有 .claude/skills/ 下的 .md 文件必须有 frontmatter」，使得这个 bug 在没有自动检测的情况下长期存在。

2. **BUG-02（-15 分）：`customize/SKILL.md` 引用 v1 路径**

   **表象**：文件引用了 `src/ipc.ts`、`src/channels/whatsapp.ts`、`src/db.ts`、`src/whatsapp-auth.ts`、`groups/CLAUDE.md` 等 v1 文件路径，而这些路径在 v2 代码库中全部不存在。

   **根本原因**：v1→v2 的架构迁移改变了整个代码库的文件布局（从文件 IPC 改为 SQLite DB，从每渠道独立模块改为 install-from-branch 模型），但 skill 文件的迁移更新没有与代码库迁移同步进行。模板复制策略（批量创建渠道 skill）带来的另一面是「批量迁移也需要同步批量更新」——但人工更新容易遗漏，而没有自动化路径验证的 CI 使得遗漏无法被发现。这是「skill 作为代码文档，而文档没有和代码一起接受版本控制」的典型失效模式。

3. **BUG-03（-20 分）：`groups/main/CLAUDE.md` v1/v2 混合内容**

   **表象**：同一个文件中既有 v1 产物（`registered_groups.json`、WhatsApp JIDs、`/workspace/ipc/` 路径、`~/.config/nanoclaw/sender-allowlist.json`、`register_group` MCP tool），又有 v2 概念，形成内部矛盾的状态机描述。Claude 读到这个文件时，会在 v1 和 v2 两套相互矛盾的认知模型之间摇摆，无法做出确定性决策。

   **根本原因**：`groups/` 目录下的 CLAUDE.md 属于「局部上下文覆盖」文件，它们的内容在 v1→v2 迁移中可能被部分更新（有人更新了 v2 内容），但没有完全清理 v1 残留（v1 引用没有被删除）。根本设计缺陷是：v1 和 v2 共存的迁移期内，没有用「已废弃」注释或者版本门控把 v1 内容与 v2 内容明确区分。

4. **CRITICAL：`init-onecli/SKILL.md` 的 curl-pipe-to-shell**

   **表象**：skill 包含 `curl -fsSL onecli.sh/install | sh`，下载远程脚本并直接执行，没有任何校验步骤。`onecli.sh` 是短域名，TLD 域名劫持风险高于 `onecli.dev` 或 `onecli.io` 等常见 TLD。

   **根本原因**：安装便利性（一行命令）与安全性（可验证的安装链）的权衡，在这里完全倒向了便利性，没有提供任何安全备选路径。NL artifact 里的安装命令和普通文档里的安装命令不同——Claude Code 会实际执行它们，因此 NL artifact 里的安全标准应该高于普通 README。

### 3.4 当时的优化机会

1. **最高优先级：修复 CRITICAL 安全发现**（`init-onecli/SKILL.md`）。将 curl-pipe-sh 替换为「下载脚本到临时文件 → 打印脚本内容供用户确认 → 用户输入 yes 后再执行」的三步流程。这是 0 星期的工作量（30 分钟内可完成），但影响最大。

2. **次优先级：给 `add-parallel/SKILL.md` 补充 frontmatter**（BUG-01）。参照 `get-qodo-rules/SKILL.md` 的结构，添加 `name`、`description`、`version`、`allowed-tools` 字段，同时修复 sed 注入风险（将 `${API_KEY_FROM_USER}` 在传入 sed 之前做 shell 转义处理）。

3. **中期优化：将 `customize/SKILL.md` 和 `groups/main/CLAUDE.md` 的 v1 内容完整清理**（BUG-02、BUG-03）。建立一个「v1 path checklist」，通过 `grep -rn "src/ipc\|registered_groups\|/workspace/ipc\|sender-allowlist\|register_group"` 扫描全仓残留 v1 引用，逐一更新或删除。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

当前仓库状态无法直接核验（克隆受限）。以下状态评估基于 2026-04-25 Audit 报告，无法确认截至 2026-06-24 的实际修复情况。

| 过去缺陷 | 可验证命令（如有仓库访问） | 当前推测状态 | 依据 |
|---|---|---|---|
| BUG-01：`add-parallel/SKILL.md` 无 frontmatter（-50） | `head -5 .claude/skills/add-parallel/SKILL.md` | **未知**（克隆受限，无法验证） | 此类 bug 通常需要刻意修复；无修复记录则保持 bug 状态 |
| BUG-02：`customize/SKILL.md` v1 路径引用（-15） | `grep -n "src/ipc\|src/channels\|src/db\|whatsapp-auth" .claude/skills/customize/SKILL.md` | **未知**（克隆受限，无法验证） | v1 路径清理需要逐文件人工处理，遗漏率高 |
| BUG-03：`groups/main/CLAUDE.md` v1/v2 混合（-20） | `grep -n "registered_groups\|/workspace/ipc\|register_group" groups/main/CLAUDE.md` | **未知**（克隆受限，无法验证） | 同 BUG-02，需人工清理 |
| CRITICAL：`init-onecli/SKILL.md` curl-pipe-sh | `grep -n "curl.*onecli.sh.*\| sh" .claude/skills/init-onecli/SKILL.md` | **未知**（克隆受限，无法验证） | CRITICAL 发现在公开 audit 报告曝光后有较高修复概率 |
| QUAL-01：~40 skill 缺 `output-format` | `grep -rL "output-format" .claude/skills/` | **未知**（克隆受限，无法验证） | 批量缺失字段通常需要脚本补充，手动修复率低 |

### 4.2 架构演进

基于 Audit 报告可观察到的信息，nanoclaw 经历了一次完整的 v1→v2 架构重写：

- **v1 架构**：文件 IPC（`src/ipc.ts`）、每渠道独立 TypeScript 模块（`src/channels/whatsapp.ts` 等）、基于 JSON 的群组注册（`registered_groups.json`）、本地配置文件（`~/.config/nanoclaw/sender-allowlist.json`）
- **v2 架构**：SQLite 数据库取代文件 IPC、install-from-branch 模型（渠道通过安装分支的方式添加，而不是修改主仓库代码）、统一的 MCP tool 接口（`register_group` 等 v1 MCP tool 被废弃）

这次架构重写是「核心逻辑现代化」，但 NL artifact 的维护没有跟上代码库的迁移节奏。这是「代码层面已经 v2，文档/skill 层面还有 v1 残留」的典型模式——NL artifact 对开发者来说不是「可执行代码」，容易被遗漏在迁移 checklist 之外。

### 4.3 新增的可学习模式

基于 Audit 报告，在 v2 架构下出现了一个值得关注的新模式：

1. **install-from-branch 渠道模型**：v2 的渠道不是通过「修改主仓库 TypeScript 代码」来添加，而是通过「安装一个 branch 上的渠道包」来添加。这与 NL artifact 的 skill 体系天然契合——每个渠道的 SKILL.md 描述「如何安装该渠道」，本身就是这个安装流程的 AI 接口层。这种设计把渠道的安装复杂度封装进 skill，让最终用户只需要告诉 Claude「帮我安装 Telegram 渠道」，而不需要知道底层的 git branch 操作。这是一个「skill 作为复杂操作入口」的优秀设计范例，值得在需要向用户隐藏技术复杂度的场景中借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **不在 skill 里使用 curl-pipe-sh 模式**：curl-pipe-to-shell 是 NL 安全审查中最高风险的模式之一。自己的 skills 如果需要安装第三方工具，应该使用包管理器（`npm install`、`pip install`、`brew install`）或者「下载→校验→确认→执行」的四步流程。

2. **在架构迁移时同步更新 NL artifact**：BUG-02 和 BUG-03 的根本原因是「代码库迁移了，skill 没有跟着迁移」。正确做法是把 NL artifact 的更新纳入迁移 checklist——每次改变代码路径，同时 grep 所有 NL artifact 里的路径引用，确保 skill 和代码保持同步。

3. **认识到 frontmatter 是 SKILL.md 的必要条件，而不是可选字段**：BUG-01 中 `add-parallel/SKILL.md` 因为缺 frontmatter 而完全失效。任何 SKILL.md 如果没有 YAML frontmatter，对 Claude Code 来说都不是一个技能，只是一个文档。`name` 和 `description` 字段是 skill 可被识别的最低要求。

4. **关注 `output-format` 字段**：nanoclaw 约 40 个 skill 缺 `output-format`（QUAL-01）。这个字段定义了 skill 执行后 AI 的输出格式期望，缺失意味着输出格式随 AI 自由发挥。在输出格式对下游有依赖的场景（如 webhook 接收、日志格式分析），缺 `output-format` 会导致不可预期的格式漂移。

5. **理解模糊量词的危害**：`debug/SKILL.md`、`manage-channels/SKILL.md`、`setup/SKILL.md` 中的 `appropriate`、`relevant`、`some`、`various` 等词（QUAL-02）是 NL artifact 中的高风险模式——它们把「什么情况下做什么」的决策权完全留给 AI，无法产生一致可预期的行为。具体词替代模糊词是 skill 质量提升最直接的动作。

### 5.2 挑战 / 验证

这个案例让我重新审视了「模板复制」策略的双面性。nanoclaw 用模板复制快速构建了 40+ 渠道 skill，但同时把模板的缺陷（缺 `output-format`）也复制到了每一个渠道 skill 里。这挑战了我一个隐性假设：「批量生成可以节省质量保障工作」。

实际上，批量生成只是把质量工作从「写 N 个 skill」变成了「写 1 个高质量模板 + 验证 N 个文件的模板合规性」。如果模板本身有缺陷（如缺 `output-format`），批量生成反而会把缺陷的修复成本乘以 N。

验证：在自己的项目里，如果使用脚本批量生成 NL artifact，应该先手动写一个符合所有 NLPM 规范的「黄金模板」，通过 `/nlpm:score` 验证模板得分达到 90+ 后，再用模板批量生成。这样质量门控在生成前而不是在生成后。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 .claude/skills/ 目录下有没有缺 frontmatter 的 SKILL.md
for f in $(find ~/.claude/skills -name "SKILL.md" 2>/dev/null); do
  # 检查文件前两行是否以 --- 开头（YAML frontmatter 标志）
  first_line=$(head -1 "$f")
  if [ "$first_line" != "---" ]; then
    echo "MISSING frontmatter: $f"
  fi
done
```
命中后怎么办：打开该文件，参照 `get-qodo-rules/SKILL.md` 的结构，在文件顶部添加包含 `name`、`description`、`version`、`allowed-tools` 字段的 YAML frontmatter 块。

```bash
# 检查 NL artifact 里有没有 curl-pipe-sh 模式
grep -rn "curl.*| sh\|curl.*| bash\|curl.*| zsh" \
  ~/.claude/skills/ ~/.claude/agents/ ~/.claude/commands/ 2>/dev/null
```
命中后怎么办：将命中的行替换为「下载到临时文件 → SHA256 校验 → 用户确认 → 执行」四步流程，或改用包管理器安装命令。

```bash
# 检查 NL artifact 里有没有模糊量词
grep -rn "\bappropriate\b\|\brelevant\b\|\bsome\b\|\bvarious\b\|\bsuitable\b\|\breasonable\b" \
  ~/.claude/skills/ ~/.claude/agents/ ~/.claude/commands/ 2>/dev/null
```
命中后怎么办：逐一替换为具体的限定词。例如「appropriate actions」→「以下三种操作之一：…」；「some files」→「以下目录下的 .md 文件：…」。

```bash
# 检查 .claude/skills/ 里有没有缺 output-format 的 SKILL.md
grep -rL "output-format" ~/.claude/skills/ 2>/dev/null | grep "SKILL.md"
```
命中后怎么办：确定该 skill 的预期输出格式（markdown 表格 / JSON / 纯文本 / 代码块），在 frontmatter 里加 `output-format: <格式>`。

```bash
# 如果使用 sed 处理用户输入，检查有没有未转义的用户输入注入风险
grep -rn 'sed.*\$\|sed.*USER\|sed.*INPUT\|sed.*KEY' \
  ~/.claude/skills/ ~/.claude/agents/ 2>/dev/null
```
命中后怎么办：在将用户输入传入 sed 之前，使用 `printf '%s\n' "$var" | sed 's/[[\.*^$()+?{|]/\\&/g'` 对特殊字符转义，或改用 Python 脚本处理字符串替换（`python3 -c "import sys; print(open('f').read().replace(old, new))"` 更安全）。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 nanoclaw 的「两级 CLAUDE.md 继承」模式，给有多个子系统的项目建立「根 CLAUDE.md 设全局规则 + 子目录 CLAUDE.md 设局部规则」的两级结构
   - **为何可行**：Claude Code 支持目录级 CLAUDE.md 覆盖，局部规则会在对应目录上下文中自动生效
   - **第一步**：在根 `CLAUDE.md` 里明确写「全局规则适用于所有子系统」，在每个子系统目录下建 `CLAUDE.md` 写「本目录覆盖规则：…」；在局部 CLAUDE.md 里引用根规则时说明「继承自根 CLAUDE.md §X」。约 30 分钟可完成基础结构。

2. **想法**：建立「黄金模板 + 批量验证」的 skill 创建流程，避免 nanoclaw 模板复制缺陷被批量放大的问题
   - **为何可行**：`/nlpm:score` 可以对单个文件评分；一旦模板得分稳定在 90+，用模板生成的 skill 质量下限就有了保障
   - **第一步**：在 `templates/` 目录下建一个 `SKILL.template.md`，包含所有必要 frontmatter 字段（`name`、`description`、`version`、`allowed-tools`、`output-format`、`triggers`），body 里有 numbered steps 和 MANDATORY 门控点的占位结构，然后运行 `/nlpm:score templates/SKILL.template.md`，确保模板本身得分 90+。约 45 分钟可完成。

3. **想法**：建立「架构迁移时的 NL artifact 同步更新 checklist」，避免 BUG-02/BUG-03 中 v1 路径残留的问题
   - **为何可行**：这是一个纯流程问题，不需要工具支持，只需要在 migration PR 的 checklist 里加一条
   - **第一步**：在项目的 `CONTRIBUTING.md` 或 `.github/PULL_REQUEST_TEMPLATE.md` 里，在「架构变更 PR」的 checklist 里加一条「`grep -rn '<旧路径>' .claude/ agents/ *.md` 搜索所有 NL artifact 里的旧路径引用，确认全部更新」。约 10 分钟可完成。

4. **想法**：参考 `get-qodo-rules/SKILL.md` 的 `triggers` 字段设计，给自己的核心 skill 加自动触发条件
   - **为何可行**：`triggers` 字段让 Claude Code 可以自动识别何时调用该 skill，不需要用户手动触发
   - **第一步**：分析已有的 skill，找出有明确触发场景的（如「当用户说『部署』时」），在 frontmatter 里加 `triggers: ["当用户提到部署", "当代码包含 deploy 命令"]`，然后用 `/nlpm:score` 验证语法正确。约 20 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

**本案例 qwibitai/nanoclaw 的核心目的**：用 skill 封装渠道安装复杂度，让一套 AI agent 逻辑跨 10+ 消息渠道运行

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是多 skill 套件；都有根 CLAUDE.md 全局规则 | gstack 是工程工具套件，nanoclaw 是消息渠道路由；gstack 无渠道概念 | 中（模板设计和 CI 门控值得借鉴） |
| MarkQWu/bureau | 低-中 | 都有 NL artifact 体系 | bureau 专注 AI 会话知识库，与消息渠道无关；bureau 有严格 tools 声明 | 低（主要借鉴安全发现方面的负面案例） |
| MarkQWu/graphify | 低 | 都有 skill 体系 | graphify 是代码知识图谱工具，目的域完全不同 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code skill 集合 | drama-workshop 无 NL artifact（`has_nl_artifacts: false`） | 低（可从本案例的模板设计获得初始化灵感） |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 NL artifact | echo-sleuth 专注挖掘历史会话，与渠道路由无关 | 低 |

### 8.2 在我的项目里复现的同类问题

因克隆环境受限，无法执行 grep 验证。以下分析基于仓库描述推断（「基于仓库描述推断」，并非对实际文件内容的断言）。

| 本案例缺陷 | 推断方法 | 推断结论 | 建议验证命令（有访问时） |
|---|---|---|---|
| SKILL.md 缺 frontmatter（BUG-01） | gstack 是「23 个工具套件」，工具数量多，快速迭代时遗漏 frontmatter 的概率较高；基于仓库描述推断，因克隆环境受限，无法执行 grep 验证 | **中风险**：多工具仓库批量添加时容易遗漏 frontmatter | `for f in $(find . -name "SKILL.md" -o -name "*.md" -path "*/.claude/*"); do head -1 "$f"; done` |
| v1 路径引用残留（BUG-02） | bureau 仓库描述强调「维护的知识库」，如有架构演进则有路径过期风险；gstack 活跃更新（last push 2026-06-21），有架构演进的可能；基于仓库描述推断，因克隆环境受限，无法执行 grep 验证 | **低-中风险**：取决于这些仓库是否经历过架构迁移 | `grep -rn "TODO\|FIXME\|deprecated\|v1\|legacy" .claude/ agents/ 2>/dev/null` |
| 模糊量词（QUAL-02） | graphify「Turn folder of code into queryable knowledge graph」描述了复杂操作，此类功能描述容易出现 appropriate/relevant 等词；基于仓库描述推断，因克隆环境受限，无法执行 grep 验证 | **中风险**：功能描述复杂的 skill 更容易出现模糊量词 | `grep -rn "\bappropriate\b\|\brelevant\b\|\bsome\b\|\bvarious\b" .claude/ agents/ 2>/dev/null` |
| 缺 `output-format` 字段（QUAL-01） | 所有仓库的 NL artifact 都可能缺 output-format；gstack「CEO、Designer、Eng Manager 等角色」的输出格式差异大，更需要明确声明；基于仓库描述推断，因克隆环境受限，无法执行 grep 验证 | **中-高风险**：gstack 各角色输出格式差异大，缺 output-format 导致格式漂移风险较高 | `grep -rL "output-format" .claude/skills/ agents/ 2>/dev/null \| grep "\.md$"` |

**高优先级行动建议**：
- MarkQWu/gstack：检查 23 个工具 skill 是否全部有 frontmatter + `output-format`（因「CEO、Designer、Eng Manager」等角色输出格式差异大，这个字段价值最高）
- MarkQWu/graphify：检查 skill 里有无模糊量词（图谱查询的结果描述容易出现「返回相关节点」等模糊表述）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：供应链安全规则写进根 CLAUDE.md
   - **本案例做法**：nanoclaw 的根 `CLAUDE.md`（92/100）明确包含供应链规则，约束第三方依赖的引入方式。这让 Claude 在处理「安装新依赖」的请求时有明确的决策依据。
   - **我的项目现状**：基于仓库描述推断，gstack（「23 个工具」）和 bureau（「维护知识库」）的根 CLAUDE.md 可能有安全说明，但是否有明确的「供应链规则」章节尚不确定；因克隆环境受限，无法执行 grep 验证。
   - **如何借鉴**：在 gstack 和 bureau 的根 `CLAUDE.md` 里加一节「依赖引入规则」，内容包括：禁止 curl-pipe-sh、所有新依赖必须有版本锁定、npm 依赖变更需要人工确认。具体格式参考 nanoclaw 的 `CLAUDE.md` 供应链规则章节结构。

2. **领域**：`container/skills/` 环境隔离设计
   - **本案例做法**：容器环境的 skills 放在 `container/skills/` 目录，与主 `.claude/skills/` 物理分离。容器外不会意外触发容器内 skill，部署边界清晰。
   - **我的项目现状**：基于仓库描述推断，graphify（「AI coding assistant skill」）和 gstack 可能有多环境部署需求；如果有 Docker 容器化，则容器 skill 隔离设计有直接适用价值；因克隆环境受限，无法执行 grep 验证。
   - **如何借鉴**：如果 gstack 或 graphify 有容器化部署场景，建立 `container/skills/` 目录存放容器环境特定 skill，并在根 CLAUDE.md 里说明「`container/` 目录下的 skill 仅在容器环境下有效」。

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：tools 声明一致性（对比 BUG-01 导致的 skill 完全失效风险）
   - **本案例问题**：`add-parallel/SKILL.md` 无 frontmatter（得分 50，skill 无法安装）；约 20 个 skill 缺 `output-format`
   - **我的项目推断优势**：bureau（「Turn AI sessions into a maintained, human-reviewed knowledge base」）和 echo-sleuth（「Mine past conversations」）的描述表明这两个项目有明确的功能边界，基于仓库描述推断其 NL artifact 的 frontmatter 完整性可能高于 nanoclaw 的渠道 skill；因克隆环境受限，无法执行 grep 验证，这一推断待验证。

2. **领域**：避免 curl-pipe-sh（对比 CRITICAL 安全发现）
   - **本案例问题**：`init-onecli/SKILL.md` 的 `curl -fsSL onecli.sh/install | sh` 是 CRITICAL 安全发现，直接导致 Security 结论为 CRITICAL，阻断所有贡献流程
   - **我的项目推断优势**：从仓库描述来看，我的五个仓库都不是安装工具类项目，curl-pipe-sh 的使用场景较少；基于仓库描述推断，gstack（「Garry Tan 的 Claude Code setup」）、bureau（「知识库」）、graphify（「知识图谱」）、echo-sleuth（「挖掘历史会话」）都以分析和管理为核心，不太可能需要 curl-pipe-to-shell 安装模式；因克隆环境受限，无法执行 grep 验证。

---

## 八、术语表

- **[SKILL.md](#skillmd)**：Claude Code 技能文件，必须以 YAML frontmatter 开头。frontmatter 中的 `name` 字段是技能的唯一标识符，Claude Code 通过此字段按名称安装和调用技能。没有 frontmatter 的 .md 文件不会被识别为技能。

- **[frontmatter](#frontmatter)**：Markdown 文件顶部 `---` 包围的 YAML 块，Claude Code 用它识别 artifact 类型和声明元数据（`name`、`description`、`version`、`allowed-tools`、`output-format`、`triggers` 等）。SKILL.md 的 frontmatter 是技能可被识别的必要条件，不是可选字段。

- **[curl-pipe-sh](#curl-pipe-sh)**：`curl URL | sh` 或 `curl URL | bash` 模式，下载远程脚本并直接执行，没有任何内容验证步骤。在 NL 安全审查中属于 CRITICAL 风险模式（对应 SEC-curl-pipe-sh 规则）。NL artifact 里的此类命令会被 Claude Code 实际执行，风险高于普通文档中的说明性命令。

- **[allowed-tools](#allowed-tools)**：Claude Code skill / command 的 frontmatter 字段，声明该 artifact 允许调用的工具列表（如 `Bash`、`Read`、`Grep`）。未声明的工具在 skill 执行时不可用或被过度授权，取决于调用上下文。

- **[output-format](#output-format)**：SKILL.md frontmatter 字段，声明 skill 执行后 AI 的预期输出格式（如 `markdown`、`json`、`plain-text`）。缺少此字段时，AI 自由选择输出格式，在输出格式有下游依赖的场景（如 webhook 接收、日志解析）会导致不可预期的格式漂移。

- **[triggers](#triggers)**：SKILL.md frontmatter 字段，声明 Claude Code 应当自动调用该 skill 的条件。有此字段的 skill 在满足条件时由 Claude Code 自动触发，无需用户手动指定，提升工作流的自动化程度。

- **[vague quantifier（模糊量词）](#vague-quantifier)**：NL artifact 中的模糊限定词，包括 `appropriate`、`relevant`、`some`、`various`、`suitable`、`reasonable` 等。这类词把「在什么情况下做什么」的决策权留给 AI，无法产生一致可预期的行为。NLPM 规则要求用具体限定词替代模糊量词（对应 R07 规则）。

- **[v1/v2 混合内容（路径残留）](#v1v2-path-residue)**：在架构大版本迁移后，部分 NL artifact 仍引用旧版本的文件路径、工具名称、配置文件路径等。这类残留使 Claude 在读取 artifact 时得到与实际代码库不符的上下文，导致生成的指令指向不存在的文件。预防方法：在迁移 PR checklist 里加「grep 扫描所有 NL artifact 的旧路径引用」步骤。

- **[install-from-branch 模型](#install-from-branch)**：nanoclaw v2 的渠道添加机制，通过安装 git 分支上的渠道包来添加渠道支持，而不是修改主仓库 TypeScript 代码。渠道的 SKILL.md 描述「如何执行这个安装过程」，是安装流程的 AI 接口层。

- **[两级 CLAUDE.md 继承](#two-level-claude-md)**：在根目录放全局 `CLAUDE.md`（适用所有上下文），在子目录放局部 `CLAUDE.md`（仅适用于该目录上下文）。Claude Code 在处理某个目录下的文件时，会同时加载根 CLAUDE.md 和最近父目录的 CLAUDE.md，局部规则可覆盖全局规则。

- **[供应链规则（Supply Chain Rules）](#supply-chain-rules)**：在 `CLAUDE.md` 中明确约束第三方依赖引入方式的规则集，典型内容包括：禁止 curl-pipe-sh 安装、所有依赖必须版本锁定、新依赖引入需要人工确认。nanoclaw 的根 CLAUDE.md（92/100）是包含供应链规则的良好示例。

- **[sed 注入风险](#sed-injection)**：在 shell 脚本中使用 `sed -i "s/pattern/${USER_INPUT}/"` 时，如果 `USER_INPUT` 包含 sed 的特殊字符（`/`、`&`、`\` 等），会导致 sed 命令语义改变，形成注入漏洞。nanoclaw 的 BUG-01（HIGH 级别）在 `add-parallel/SKILL.md` 中有此模式（`sed -i.bak "s/^PARALLEL_API_KEY=.*/PARALLEL_API_KEY=${API_KEY_FROM_USER}/"`）。

- **[NLPM（Natural-Language Programming Manager）](#nlpm)**：本报告使用的 NL artifact 质量评分系统，100 分制，从 100 分开始按规则扣分，底线 0 分，上限 100 分。默认及格线 70 分（可通过 `.claude/nlpm.local.md` 配置）。本案例全仓得分 81/100。
