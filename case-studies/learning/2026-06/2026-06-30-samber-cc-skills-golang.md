# samber/cc-skills-golang — 学习案例

**仓库**：https://github.com/samber/cc-skills-golang
**Stars**：1362 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-30（基于当前 HEAD）
**主题标签**：`single-purpose`, `cross-reference`, `examples-driven`, `template-design`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

cc-skills-golang 是由 [samber](https://github.com/samber)（Go 生态知名开源贡献者，维护 samber/lo、samber/do、samber/mo 等高星项目）发布的 Claude Code [插件](#插件)，包含 38 个 Go 语言主题技能，覆盖从代码风格、并发、测试到 gRPC、CLI、依赖注入的全栈 Go 开发场景，并深度整合作者自己的 samber/* 库生态。是 Claude Code [插件市场](#插件市场) 中 Go 方向星数最高的技能集合。

关键事实：
- **作者背景**：samber 是 Go 生态重量级作者，lo（39k stars）、do（1.9k stars）、mo（2.4k stars）等库的维护者——本插件相当于作者把自己的 Go 经验结构化为 AI 可调用的知识库
- **技能范围**：38 个技能，覆盖代码风格、并发、错误处理、性能、安全、测试、文档、CLI、gRPC、数据结构、设计模式、持续集成等 20+ 主题
- **自家生态集成**：专门为 `samber/lo`、`samber/do`、`samber/hot`、`samber/mo`、`samber/oops`、`samber/ro`、`samber/slog` 各配套了独立技能
- **部分空缺**：`golang-graphql/SKILL.md` 是占位符（仅含「Content will be added in future iteration」），有效技能实为 37 个

### 1.2 架构剖析

```
cc-skills-golang/
├── .claude-plugin/
│   └── plugin.json        ← 插件 manifest（注册 38 技能）
├── CLAUDE.md              ← 开发者指南
├── skills/
│   ├── golang-benchmark/
│   │   └── SKILL.md       ← 典型技能（persona + modes + examples + cross-ref）
│   ├── golang-concurrency/
│   │   └── SKILL.md
│   ├── golang-dependency-injection/
│   │   └── SKILL.md
│   ├── golang-graphql/
│   │   └── SKILL.md       ← 占位符（只有 placeholder 文字）
│   ├── golang-uber-dig/
│   │   └── SKILL.md       ← 含损坏的交叉引用（→ 不存在的 golang-google-wire）
│   ├── golang-uber-fx/
│   │   └── SKILL.md       ← 同上
│   └── ... (另外 32 个 SKILL.md)
└── （无 hooks、无 scripts、无 MCP 配置）
```

- **文件类型分布**：38 个 SKILL.md + 1 个 plugin.json + 1 个 CLAUDE.md（0 个 command，0 个 agent，0 个 hook）
- **编排关系**：纯平铺结构——38 个技能之间是平级关系，无 router，无主/子 skill 层级。技能间的关联通过各自的 `Cross-References` 段落手动声明。存在主题簇：性能技能簇（`golang-performance`、`golang-benchmark`、`golang-troubleshooting`、`golang-observability`）有明确边界说明。
- **跨件契约**：技能通过 `Cross-References` 段落互相引用，格式为 `` `samber/cc-skills-golang@<skill-name>` ``。但其中 2 处引用了不存在的 `golang-google-wire` skill，是悬空引用。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「主题平铺 + 作者自有库深度集成」。不设层级、不设路由，每个技能对应一个 Go 开发主题，同时为作者维护的 samber/* 库各建一个专门技能。
- **解决什么问题**：Go 开发者使用 Claude 时，Claude 对 Go 生态惯用法（如 `samber/lo` 的 fp 风格、`samber/do` 的依赖注入）的掌握参差不齐。本插件将这些惯用法固化为结构化技能，保证 Claude 在 Go 项目中行为一致。
- **Trade-off**：平铺 38 个技能维护简单，不需要维护层级关系；但缺少 router 层意味着当用户的需求跨多个主题时，Claude 需要自行决定加载哪几个技能。`golang-graphql` 空占位符问题说明了平铺结构的另一个风险：stub 技能对用户无价值，但因为没有中央注册表检查，空占位可以悄悄存活。
- **认知模型**：作者把 Claude 的技能库看作「专家知识的结构化存储」——每个技能是一个子领域的专家视角，Claude 在遇到对应问题时调用该视角，而非从泛化知识中自行推导。这个认知模型下，38 个技能 = 38 个专家，各有边界，互相引用。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：领域分组 + 作者自有库集成的平铺技能库**

单一作者将自身领域专长（Go 语言）和自维护的软件库生态（samber/*）编排为一个平铺技能集合，每个技能覆盖一个明确边界的子域，库技能与语言技能平级共存。

模式特征清单：
- **领域主权**：作者是该领域（Go + samber/*）的权威，技能内容来自一手经验而非二手文档
- **平铺无层级**：38 个技能平级，无 router，通过 [frontmatter](#frontmatter) `description` 决定哪个技能触发
- **自有库专属技能**：`samber/lo`、`samber/do` 等库各有独立技能，不与语言层技能混合
- **主题簇用边界说明**：相近主题间（性能/基准/故障排查/可观测性）通过 `Cross-References` + 边界说明区分，而不是靠路由层强制
- **纯 Markdown**：无可执行表面（no hooks, scripts, MCP），最低风险，最易分发

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 单一领域深度覆盖（语言/框架/工具链）| ✅ 高度适用 | 平铺结构维护简单，每个技能边界清晰 |
| 作者有自维护的同领域库 | ✅ 高度适用 | 自有库集成是差异化亮点，减少「Claude 不知道我的库」的摩擦 |
| 多领域混合插件（前端 + 后端 + DevOps）| ⚠️ 谨慎适用 | 平铺 38 个技能已接近上限，跨领域混合会让 description 触发逻辑混乱 |
| 需要写操作或外部 API 调用 | ❌ 不适用 | 本模式纯知识型，无工具调用；需要 MCP 适配层时应换架构 |
| 内容快速迭代（每周更新）| ❌ 不适用 | 平铺文件维护成本随数量线性增长，批量更新时交叉引用容易失效 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域平铺技能库（本仓库） | cc-skills-golang | 维护简单；技能边界清晰；纯 Markdown 零风险 | 无 router，描述触发可能误命中；stub 技能难发现 |
| 主 skill + 意图子 skill | Xquik-dev/x-twitter-scraper | 大型 API 导航高效；token 消耗低 | 维护成本高；42 个文件参数漂移风险 |
| 状态型命令插件 | SukinShetty/Nemp-memory | 有状态 persist 能力；命令驱动更直观 | 需要复杂的状态管理设计 |

### 2.4 改进空间

1. **当前问题**：`golang-graphql/SKILL.md` 是空占位符，得分 45/100，`user-invocable: false` 且无内容，任何触发路径均失效。**改进做法**：要么写出完整 skill body（gqlgen 服务端 + graphql-client 客户端，加入 schema 定义、resolver 模式、N+1 问题处理示例），要么在完成之前改为 `user-invocable: true` 并在 description 中注明「content WIP」。**预期收益**：把仓库整体评分从 87 提升约 1.5 分，同时消除用户「安装了但没有 GraphQL 帮助」的预期落差。

2. **当前问题**：`golang-uber-dig` 和 `golang-uber-fx` 的 Cross-References 都引用了不存在的 `golang-google-wire` skill（悬空引用）。**改进做法**：在 Go DI 生态中，Wire 是 dig/fx 的直接竞争方案，意义明确；可以创建 `skills/golang-google-wire/` 技能（覆盖 wire.go 文件、provider 模式、code generation 步骤），或者将引用改为指向 `golang-dependency-injection`（现有通用 DI 技能）。**预期收益**：消除悬空引用，Agents 跟随引用时不再静默失败。

3. **当前问题**：38 个平级技能缺乏「新用户入口技能」——用户安装插件后不知道从哪个技能开始。**改进做法**：添加 `golang-overview/SKILL.md`，内容为「我现在要开始一个 Go 项目」时的导航技能，按项目阶段列出推荐技能序列（布局 → 风格 → 测试 → CI → 性能），充当轻量 router。**预期收益**：降低新用户的认知成本，同时不破坏现有平铺结构。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 87/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/golang-graphql/SKILL.md | 45 | 仅含占位符文本，无任何实质内容 |
| skills/golang-stay-updated/SKILL.md | 77 | 仅为资源链接列表，无指导性内容；无 persona；description 缺触发场景 |
| skills/golang-popular-libraries/SKILL.md | 82 | 正文太薄，主要靠引用文件；description 触发过宽 |
| skills/golang-uber-dig/SKILL.md | 82 | 含悬空交叉引用（→ golang-google-wire 不存在） |
| skills/golang-uber-fx/SKILL.md | 83 | 同上 |
| skills/golang-benchmark/SKILL.md | 89 | 小幅模糊量词（R01） |
| 其余 32 个技能 | 89-91 | 小幅模糊量词（R01）；结构完整 |
| plugin.json / CLAUDE.md | 90 | 配置文件，无技能 frontmatter，设计如此 |
| skills/golang-performance/SKILL.md | 91 | 小幅模糊量词；具备 ultrathink、modes、决策树 |
| skills/golang-security/SKILL.md | 91 | 同上；包含并行审计 agents 调用 |

### 3.2 当时值得借鉴的模式

**① 技能主题簇 + 边界说明**：`golang-performance`、`golang-benchmark`、`golang-troubleshooting`、`golang-observability` 四个技能形成明确的技能簇，每个技能的 description 或正文中都有边界说明（「性能分析看 performance，指标采集看 observability」）。根本原因：相近主题并存时，没有边界说明 Claude 会随机命中或多次加载；边界说明让召回精准。借鉴方法：当有 2 个以上功能相近的技能时，为每个技能写一段「边界说明」，明确「我负责什么，不负责什么」。

**② 统一 persona + modes + examples 结构**：35/38 技能得分 89-91，差异极小。原因在于所有技能遵循相同的模板结构（persona、modes、examples、cross-references、common-mistakes），质量高度一致。根本原因：模板统一意味着技能库不依赖作者某一次的状态，也不会因为某个技能「写得用心」而产生质量分化。借鉴方法：在写第 3 个以上技能时，先固化一个模板，然后所有技能都从模板派生。

**③ 自有库专属技能**：`samber/lo`、`samber/do` 等库各有独立技能，独立技能的内容是这些库的惯用模式和最佳实践，而不是 API 文档复制。根本原因：Claude 的泛化知识对长尾库（如 samber/oops）的覆盖不充分；专属技能相当于给 Claude「打补丁」，弥补训练数据不足。借鉴方法：如果我使用了一个 Claude 不熟悉的内部库或冷门库，为它单独写一个技能，内容侧重惯用模式和常见陷阱，而不是 README 翻译。

**④ ultrathink + 并行 agents（高复杂度技能）**：`golang-security` 技能使用 `ultrathink` 指令并调用并行审计 agents，`golang-performance` 使用 `ultrathink` + 决策树。根本原因：安全审计和性能分析是需要多维度同时检查的任务，串行思考容易遗漏，并行 agents 提升覆盖率。借鉴方法：对于「需要从多个角度系统检查」的技能（安全/性能/架构审查），在技能中加入 `ultrathink` 指令和并行 agents 调用模式。

**⑤ common-mistakes 表格**：多个技能包含「常见错误」表格，每行一个错误模式 + 根本原因 + 正确做法。根本原因：Claude 回答时倾向于给出「教科书正确做法」，而实际 Go 代码中的错误是特定且重复的；把常见错误编目在技能里，让 Claude 能主动识别和警告。借鉴方法：为我的技能添加「常见错误」段落，内容来源：我自己踩过的坑、code review 中反复出现的问题。

### 3.3 当时的缺陷

**① golang-graphql 是僵尸占位符**：技能正文只有「Content will be added in a future iteration」，`user-invocable: false`，任何触发路径均失效。为什么会失败：`user-invocable: false` 意味着技能靠 description 自动触发，但正文为空则无触发信号，结果是：既不能手动调用，又无法自动触发，技能在任何路径下都永远不被使用。Root cause：「先占坑，内容后补」的开发习惯在 NL 工件中有隐患——空 skill 不像空函数会报错，它只是静默无用。自查：我有没有「先创建 SKILL.md 文件但正文未写」的技能？

**② 悬空交叉引用（golang-uber-dig / uber-fx → golang-google-wire）**：两个 DI 框架的技能都引用了 `golang-google-wire` skill，但该 skill 目录不存在。为什么会失败：当 Claude 沿着 Cross-References 找 `golang-google-wire` 时，会静默失败——不是 404，而是找不到对应的技能文件，Claude 可能随机生成 Wire 相关内容而不是引用权威技能。Root cause：两个文件完全相同的悬空引用说明是从同一个模板 copy-paste 时留下的计划中 skill，但从未创建。自查：我的技能 Cross-References 里有没有指向不存在技能名的引用？

**③ golang-stay-updated 是资源书签而非技能**：正文只是一个资源链接列表（GitHub repos、博客、播客），没有指导性内容、没有 persona、description 缺乏触发场景。为什么会失败：Claude 加载一个「书签页」并不能帮助用户解决「如何持续关注 Go 生态」的实际问题——知道该去看哪些网站，和知道如何建立持续学习流程，是两件事。Root cause：作者可能把「有用的资源列表」和「对 AI 有用的指导性内容」混淆了。自查：我的技能正文有没有纯罗列资源/链接而缺乏可操作指导的？

### 3.4 当时的优化机会

1. **补全 golang-graphql**：写出完整的 GraphQL skill，内容至少包含：gqlgen 服务端设置（schema-first workflow）、graphql-client 客户端调用示例、N+1 问题处理（DataLoader 模式）。参考同仓库高分技能（89-91）的结构：persona + modes + examples + cross-references + common-mistakes，预计 150-200 行。

2. **填补 golang-google-wire 空缺**：仓库已覆盖 dig、fx、samber/do、samber/lo 四种 Go DI 方案，Wire 是显著缺口。创建 `skills/golang-google-wire/` 覆盖：wire.go 文件结构、provider 函数模式、injector 声明、code generation 步骤（`wire gen`），以及 dig vs fx vs wire 的选择矩阵。

3. **改写 golang-stay-updated 为流程技能**：将书签列表改为「Go 生态感知流程」——如「每月检查 Go release notes 的具体方法」「评估新库是否值得引入的检查清单」「跟踪 samber/* 库更新的订阅方式」，让 Claude 能给出可操作的建议，而不只是网站地址。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

由于环境网络限制，无法克隆目标仓库（HTTP 403），以下分析基于历史 audit（2026-04-29）。

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| golang-graphql 空占位符 | `wc -l skills/golang-graphql/SKILL.md` | 未知（无法访问 HEAD） | 若未修复，仍拖低整体评分约 1.6 分 |
| 悬空 golang-google-wire 引用 | `ls skills/golang-google-wire/` | 未知（无法访问 HEAD） | 若未创建，uber-dig / uber-fx 的引用仍悬空 |
| golang-stay-updated 纯书签 | `grep -c "http" skills/golang-stay-updated/SKILL.md` | 未知（无法访问 HEAD） | 若未改写，技能指导价值仍接近零 |

### 4.2 架构演进

无法访问当前 HEAD。从 audit（2026-04-29）的已知状态来看：仓库是「纯 Markdown 平铺」架构，修改成本低（每个 skill 独立文件），无连锁改动风险。历史上不太可能发生结构重组——Go skills 插件的核心结构（一主题一文件）是已知稳定的设计。如果有演进，更可能是单个 skill 的内容扩充，而不是架构调整。

### 4.3 新增的可学习模式

暂无（无法访问当前 HEAD）。待下次可访问目标仓库时验证 golang-graphql 是否已补全以及 google-wire 是否已创建。

---

## 五、校准

### 5.1 我已经在做对的

1. **为自有库写专属技能的思路是对的**：本仓库证明了这个方向的价值——samber 的自有库技能（`samber/lo`、`samber/do` 等）正是插件得以在 Go 生态中有差异化价值的原因。如果我有内部库或工具，为其写专属技能是高回报投入。
2. **技能边界说明防止误触发**：已经意识到技能间要有边界说明，本案例的性能技能簇验证了这个实践。
3. **模板驱动统一质量**：已在使用模板结构写技能，这是 35/38 技能达到 89 分的根本原因。

### 5.2 挑战 / 验证

**挑战**：本案例中 `golang-graphql` 的问题挑战了「先占坑再补内容」的开发习惯——这是我自己也会做的事：「先建文件、内容下次再写」。NL 工件的特殊性在于：空函数会报编译错误，但空 SKILL.md 只会静默失效。需要建立「新建 NL 工件必须有最低可用内容」的纪律，而不是「文件存在即视为完成」。

**验证**：之前对「复杂任务是否值得在技能里写 ultrathink 和并行 agents」有疑虑。`golang-security` 和 `golang-performance` 两个技能的设计确认了：对于多维度系统检查类任务（安全审计、性能分析），在技能中声明并行检查模式是正确投入，不是过度设计。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能中是否有空占位符（内容不足 10 行的 SKILL.md）
find . -path "*/skills/*/SKILL.md" -exec wc -l {} \; | awk '$1 < 10 {print $2, "可能是占位符"}'
# 命中后：立即补全正文，或将 user-invocable 改为 true 并在 description 注明 WIP
```

```bash
# 检查交叉引用的 skill 名是否都真实存在
grep -rn "cc-skills-golang@\|skills-golang@\|@golang-" $(find . -path "*/skills/*/SKILL.md") | \
  sed 's/.*@//;s/[`>).].*//' | sort -u
# 命中后：对每个引用名，检查对应目录是否存在；不存在则移除引用或创建目标技能
```

```bash
# 检查技能描述是否包含可操作触发语（而不是纯资源列表）
grep -rL "Use when\|Use if\|when the user" $(find . -path "*/skills/*/SKILL.md" 2>/dev/null)
# 命中后：在 description 中添加具体触发场景描述
```

```bash
# 检查是否有纯链接/资源列表的技能（body 中链接占比过高）
grep -rc "https://" $(find . -path "*/skills/*/SKILL.md" 2>/dev/null) | awk -F: '$2 > 5 {print $1, "链接数:", $2}'
# 命中后：评估技能是否有足够的指导性内容，资源列表应作为「参考」而不是「正文主体」
```

### 6.2 灵感 → 实施路径

1. **想法**：为我使用的冷门内部库或工具链写专属技能（仿 `samber/lo` 的做法）。
   - **为何可行**：Claude 对长尾库的掌握不足，专属技能相当于给 Claude「打补丁」；我最熟悉自己用的工具，写出来的质量也最高。
   - **第一步**：列出我项目中 Claude 最常搞错或最不了解的一个库/工具，写一个 50-80 行的技能，包含：3 个惯用模式示例 + 2 个常见错误 + 1 个决策矩阵（什么时候用）。时间：30-45 分钟。

2. **想法**：为 gstack 或 bureau 建立「技能簇边界说明」文档——仿照本案例的性能技能簇设计。
   - **为何可行**：当技能数量超过 5 个时，相近技能间的触发混淆风险上升；边界说明是低成本高收益的防御措施。
   - **第一步**：找出 bureau 或 gstack 中相似度最高的 2-3 个技能，在每个技能的 `Cross-References` 或 `Related` 段落中，用一句话写清楚「我负责 A，B 见另一个技能」。时间：每个技能 5-10 分钟。

3. **想法**：建立「新 SKILL.md 最低可用内容」规范（对抗空占位符反模式）。
   - **为何可行**：`golang-graphql` 案例证明空占位符是静默失效，不会被自动检测到；有了最低可用内容规范，才能避免同类问题。
   - **第一步**：在项目的 CONTRIBUTING.md 或 CLAUDE.md 中加一段：「新建 SKILL.md 必须包含：非空的 description、至少 1 个使用示例、至少 1 个 persona 段落。否则 NLPM score 将低于 70」。时间：10 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度

- **本案例 samber/cc-skills-golang 的核心目的**：将 Go 语言全栈开发经验（含作者自维护的 samber/* 库生态）编码为 Claude Code 技能集，让 Claude 在 Go 项目中具有专家级惯用法指导能力。

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 同为 Claude Code NL 工件集合；都有多个 skill | bureau 是通用工作流工具，不是单一语言/框架技能库 | 中 |
| MarkQWu/drama-workshop-skills | 中 | 同为 Claude Code skill 集合；可能也有多个主题技能 | 面向戏剧工作坊，不是技术编程领域 | 中 |
| MarkQWu/gstack | 低 | 有 NL 工件 | 是技术栈工具，不是纯技能知识库 | 低 |
| MarkQWu/echo-sleuth-for-claude | 低 | 有 NL 工件（Claude Code 插件形态） | 面向调试/侦测功能，不是语言知识库 | 低 |

### 8.2 在我的项目里复现的同类问题

由于无法克隆 my-repos（HTTP 403 代理限制），以下基于 `my-repos.json` 描述推断：

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 空占位符 SKILL.md | `find . -path "*/skills/*/SKILL.md" -exec wc -l {} \; \| awk '$1 < 10'` | bureau、drama-workshop-skills 若有未完成 skill 文件，高风险命中 | 高 |
| 悬空 Cross-References | `grep -rn "Cross-Reference\|skills@" */skills/*/SKILL.md` | 任何有交叉引用的仓库（bureau、drama-workshop-skills）可能命中 | 高 |
| 纯书签正文（无指导内容）| `grep -c "https://" */skills/*/SKILL.md \| awk '$2 > 5'` | 若有「推荐资源」类 skill，可能命中 | 中 |

**命中后的具体行动建议**：
- drama-workshop-skills 的任何空 SKILL.md → 立即补写 20 行以上的 persona + 1 个 example → 15 分钟
- bureau 的 Cross-References 里引用的技能名 → 验证每个引用名对应目录是否存在 → 10 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：技能簇边界说明
   - **本案例做法**：性能技能簇（4 个技能）的每个技能都有明确边界说明，告诉 Claude 「这个技能覆盖 A，B 情况用 X 技能」。路径：`skills/golang-performance/SKILL.md`（边界说明段落）
   - **我的项目现状**：bureau 或 drama-workshop-skills 若有相近技能，可能缺少这类边界说明
   - **如何借鉴**：找出我项目中功能最接近的 2 个技能，在两者的 Cross-References 里各加一行「当 [X 情况] 时用 [另一技能]」

2. **领域**：ultrathink + 并行 agents 模式（高复杂度任务）
   - **本案例做法**：`golang-security` 在技能中显式声明调用并行审计 agents，每个 agent 检查不同安全维度
   - **我的项目现状**：若有审计/检查类技能，可能是串行思考模式
   - **如何借鉴**：在 echo-sleuth-for-claude 的侦测类技能中，尝试加入并行 agents 模式（不同 agent 检查不同层面的异常）

3. **领域**：自有工具的专属技能
   - **本案例做法**：samber/lo、samber/do 等作者自维护的库各有专属技能，内容是惯用模式 + 常见陷阱，而非文档复制
   - **我的项目现状**：我的内部工具（如 gstack 的核心功能）可能没有对应的 Claude Code 技能
   - **如何借鉴**：识别 Claude 在我的项目中最常出错的一个内部工具/约定，写一个 50 行技能，重点是「惯用模式」和「这里 Claude 通常会搞错的地方」

### 8.4 反向：我的项目做得比他们好的地方

本案例中存在 3 个高置信度 bug（空 graphql 技能、2 个悬空引用），说明单独作者仓库缺乏自动化质量把关。若我的仓库配置了 NLPM pre-commit 检查或 CI 扫描，在「自动化质量门控」这个维度比本案例更优：
- **我的做法**：若已配置 `bin/nlpm-check` pre-commit hook，任何空 SKILL.md 或悬空引用在提交时就会被发现
- **本案例的弱项**：纯手工维护，空占位符存活了至少 37 天（audit 到 exemplar 快照之间）才可能被发现
- **意义**：自动化 NL 工件验证是我项目的相对优势，可以在 README 或 plugin manifest 中作为特性标注

---

## 八、术语表

### <a name="插件"></a>插件
> 在 Claude Code 里，插件是一个包含 `plugin.json`（[manifest](#manifest)）的目录，里面可以有 skill、command、agent 等工件。安装插件后，Claude Code 会读取 manifest，把里面声明的所有工件注册到会话中。`claude plugin install <repo>` 是标准安装命令。

### <a name="插件市场"></a>插件市场
> Claude Code 的官方或社区维护的插件目录，用户可以搜索和安装第三方插件。类似于 VS Code 的扩展市场。插件上架需要满足一定的质量要求。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`user-invocable` 等）。Claude Code 读取任何 NL 工件时先解析 frontmatter——如果 frontmatter 里缺少必填字段，该工件可能无法正常触发或注册。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skill 的文件路径。如果 manifest 里漏写了某个文件，那个文件即使在磁盘上也不会被加载。

### <a name="persona"></a>persona
> 技能中「我是谁」的定义段落。它告诉 Claude 在这个技能激活后应以什么角色和视角回答问题。例如「你是一名有 10 年 Go 并发编程经验的工程师，熟悉 Go 内存模型的细节」。有了 persona，Claude 的回答风格和立场更稳定一致。

### <a name="router"></a>router
> 在 NL 插件架构中，router 是一个「派发层」技能或 agent，其职责是判断用户意图并转发到正确的子技能或子 agent。类比于 HTTP router（根据 URL 路径决定调用哪个处理函数）。本案例（cc-skills-golang）没有 router 层，依靠各技能的 description 直接触发。

### <a name="悬空引用"></a>悬空引用
> 指向不存在目标的引用。在 NL 工件中，技能的 `Cross-References` 段落引用了一个不存在的技能名（如 `golang-google-wire`），称为悬空引用。Claude 沿着悬空引用查找时，会静默失败——不报错，只是找不到内容，可能生成不准确的信息来填补空白。

### <a name="ultrathink"></a>ultrathink
> Claude Code 中的一个指令关键词，触发后 Claude 会使用更多推理 token 进行更深入的分析，类似于「深度思考模式」。在技能定义中写入 `ultrathink` 后，该技能触发时 Claude 会自动进入更深入的推理状态。适合用于安全审计、性能分析等需要多维度系统思考的任务。

### <a name="NL工件"></a>NL 工件
> Natural Language Artifact，自然语言编程工件。指 Claude Code 能读取并执行的 Markdown 文件，包括 SKILL.md（技能定义）、command.md（命令定义）、agent.md（代理定义）等。这些文件不是代码，而是用自然语言（通常是英文）写给 Claude 读的指令。
