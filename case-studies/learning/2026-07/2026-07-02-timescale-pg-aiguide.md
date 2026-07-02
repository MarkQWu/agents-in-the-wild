# timescale/pg-aiguide — 学习案例

**仓库**：https://github.com/timescale/pg-aiguide
**Stars**：1680 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-16（历史快照）| **生成日期**：2026-07-02（基于当前 HEAD）
**主题标签**：`cross-reference`, `vague-quantifier`, `security-gate`, `examples-driven`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
**timescale/pg-aiguide** 是由 Timescale 公司维护的 AI 指导工具，专门解决 AI 生成 PostgreSQL 代码质量低下的问题（过时 API、缺少约束和索引、不了解现代 PG 特性、缺乏真实业务经验）。

关键事实：
- 创建者 Timescale 是 TimescaleDB 的商业公司，对 PostgreSQL 有深度专业背景
- 三重集成模式：Agent Skills（via `npx skills`）+ 公共 MCP 服务器（`https://mcp.tigerdata.com/docs`）+ Claude Code 插件
- 9 个 skill 文件覆盖 PostgreSQL 的完整工作流：设计 → 优化 → 扩展（TimescaleDB/PostGIS）→ 搜索
- Apache 2.0 协议，属于 Timescale 的生态拓展产品

### 1.2 架构剖析
- **目录结构**：
  ```
  pg-aiguide/
  ├── skills/
  │   ├── postgres/SKILL.md              # 总调度 skill（orchestration）
  │   ├── design-postgres-tables/        # 表设计规范
  │   ├── design-postgis-tables/         # PostGIS 地理数据设计
  │   ├── pgvector-semantic-search/      # pgvector 向量检索
  │   ├── postgres-hybrid-text-search/   # 混合文本搜索
  │   ├── setup-timescaledb-hypertables/ # 时序表设置
  │   ├── migrate-postgres-tables-to-hypertables/ # 时序表迁移
  │   └── find-hypertable-candidates/    # 时序表候选识别
  ├── src/                               # TypeScript MCP 服务器实现
  ├── CLAUDE.md                          # 项目整体说明
  └── skills/postgres/references/        # ← 该目录【实际不存在】
  ```
- **文件类型分布**：8 个 SKILL.md + 1 个 CLAUDE.md = 9 个 NL 工件；另有 TypeScript MCP 服务端（约 15 个文件）
- **编排关系**：`skills/postgres/SKILL.md` 是唯一的 [orchestration skill](#orchestration-skill)，作用是路由器——根据用户任务加载对应的子 skill（如设计表 → design-postgres-tables）。子 skill 之间有明确的「参见」交叉引用（如 pgvector 建议先看 postgres-hybrid-text-search）
- **跨件契约**：TimescaleDB trilogy（find-hypertable-candidates → migrate → setup）通过文本描述正确交叉引用；BUT：postgres/SKILL.md 的 7 个 `references/` 链接指向不存在的目录，是最关键的跨件断裂

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「专业知识的协议化编码」—— 把 Timescale 工程师多年的 PostgreSQL 最佳实践用 skill 文件固化，让任何 AI coding assistant 都能调用
- **解决什么问题**：AI 工具在没有领域知识时生成 `VARCHAR(255)`、缺 INDEX、不知道 TIMESTAMPTZ vs TIMESTAMP 区别等 anti-pattern；pg-aiguide 通过 skill 注入专家知识
- **做了什么 trade-off**：选择了「MCP 服务器 + skill 文件」双轨并行，MCP 提供实时文档检索，skill 文件提供固化的最佳实践；代价是两者需要独立维护，可能产生不一致
- **反映什么认知模型**：作者将 skill 看作「专家知识的编译单元」—— 内容以具体技术规范为主（如「用 `GENERATED ALWAYS AS IDENTITY` 而非 `SERIAL`」），而非「告诉模型怎么思考」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「Orchestration Skill + 知识 Skill 分层架构」**

顶层 `postgres/SKILL.md` 充当路由层：识别用户意图，将任务分发到对应的专业 skill。各专业 skill 只负责一个垂直领域，互不侵犯。

模式特征清单：
- 特征 1：专门设置一个「分发者」skill（postgres/SKILL.md），它本身不执行任务，只做路由
- 特征 2：各领域 skill 之间存在显式的「参见」交叉引用（如「完成后请阅读 pgvector skill」）
- 特征 3：skill 内容以专业技术规范为主，不含模糊指导语（如「使用合适的方式」）
- 特征 4：API 废弃表（旧函数 → 新函数）内嵌在相关 skill 中，防止模型生成过时代码
- 特征 5：与 MCP 服务器互补——skill 是静态最佳实践，MCP 是动态文档检索

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 某个技术领域有深度专业知识可编码 | ✅ 高度适用 | 分层架构能组织大量专业内容 |
| 垂直领域内子任务之间有自然工作流顺序 | ✅ 适用 | orchestration skill 能建模 TimescaleDB trilogy 这样的串行流程 |
| 需要跨工具（Claude、Cursor、Copilot）复用知识 | ✅ 适用 | `npx skills` 安装一次，多工具可用 |
| 知识更新频率高（如 API 频繁变化） | ❌ 谨慎 | skill 文件是静态的，需要手动维护，MCP 服务器更适合高更新频率 |
| 通用工作流提升（非专业领域） | ❌ 不适用 | 分层 skill 对通用任务过于厚重 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Orchestration + 知识分层（本仓库） | timescale/pg-aiguide | 结构清晰、专业深度高 | orchestration skill 的 references/ bug 会让整个路由层失效 |
| 单文件全包含 | antfu/skills | 安装简单、无跨件依赖 | 单文件过大时不可维护 |
| MCP 服务器唯一入口 | upstash/context7 | 实时更新、动态检索 | 需要服务器运行，离线不可用 |

### 2.4 改进空间
1. **当前问题**：`skills/postgres/SKILL.md` 中 7 个 `references/` 链接指向不存在的目录，orchestration skill 完全失效。**改进做法**：将链接从 `references/design-postgres-tables.md` 改为 `../design-postgres-tables/SKILL.md`（相对路径指向实际文件）。**预期收益**：orchestration skill 从实际无效（Bug #1）恢复正常功能，score 从 85 升至 95+。
2. **当前问题**：CLAUDE.md 中出现「required for gpt-5 compatibility」（GPT-5 不存在）。**改进做法**：将错误模型名改为 GPT-4 或删除该说明。**预期收益**：减少贡献者困惑，提高文档可信度。
3. **当前问题**：`src/migrate.ts` 和 `src/searchDocs.ts` 中的 `${schema}` 变量未经转义直接插入 SQL。**改进做法**：提取 `safeIdentifier()` helper 函数（使用 `pgPool.escapeIdentifier()` 或正则白名单 `/^[a-z_][a-z0-9_]*$/`），在两个文件的所有 SQL 模板处统一调用。**预期收益**：消除 SQL 注入向量，即使攻击者控制了 DB_SCHEMA 环境变量也无法注入。

---

## 三、过去审查发现（2026-04-16 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-16 当时得分 **94/100**（9 个工件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/postgres/SKILL.md | 85 | 7 个 references/ 链接全部失效（Bug #1） |
| CLAUDE.md | 88 | 「gpt-5 compatibility」错误模型名 |
| skills/design-postgis-tables/SKILL.md | 94 | 「appropriate」出现 3 次（各 -2 分） |
| skills/pgvector-semantic-search/SKILL.md | 96 | 「significantly」「roughly」各 -2 分 |
| skills/design-postgres-tables/SKILL.md | 98 | 「usually optimal」-2 分 |
| skills/find-hypertable-candidates/SKILL.md | 98 | 「mostly」-2 分 |

### 3.2 当时值得借鉴的模式

1. **API 废弃表内嵌 skill** → `setup-timescaledb-hypertables/SKILL.md` 包含旧→新函数名对照表。根本原因：防止模型生成废弃 API（如 `add_dimension` → `add_dimension_config`）。如何借鉴：自己的 skill 如果涉及有历史包袱的 API，内嵌废弃对照表是最可靠的防止「历史代码幻觉」的方法。

2. **TimescaleDB trilogy 正确交叉引用** → 3 个时序相关 skill 之间通过「参见」形成正确工作流（find → migrate → setup）。根本原因：作者在设计时想清楚了任务的自然顺序。如何借鉴：一组相关 skill 的最后一步应包含「下一步建议」，引导用户/模型进入后续 skill。

3. **版本感知的文档搜索** → MCP 服务器对 PostgreSQL 文档支持版本感知查询。根本原因：不同版本的 PG 特性差异大，版本无关的文档会导致错误建议。如何借鉴：自己的领域知识 skill 如果涉及有版本差异的 API，在 skill 顶部明确支持的版本范围。

4. **「Not for」排除子句** → 每个 skill 的 description 明确说明「不适用于哪些场景」。根本原因：防止模型将 skill 误用到不适合的场景。如何借鉴：在 `description` 字段后加 `Not for: <反例场景>` 是避免调用歧义的高效手段。

5. **完整覆盖技术生命周期** → 8 个 skill 从表设计（design）→ 优化（index、search）→ 时序扩展（hypertable）→ 地理扩展（postgis）形成完整 PostgreSQL 技能图谱。根本原因：Timescale 作为专业公司，有能力覆盖完整知识图谱。如何借鉴：设计垂直领域插件时，先画出领域的完整「知识地图」，再按知识节点设计 skill。

### 3.3 当时的缺陷

1. **Bug #1 — 7 个 references/ 链接全部断裂**（`skills/postgres/SKILL.md`）→ orchestration skill 让 Claude「加载引用文件」，但 references/ 目录从未存在，Claude 无法加载任何子 skill 内容。**根本原因**：作者设计时规划了 references/ 目录来存储 skill 的「精简摘要版」，但实现时只创建了完整的 SKILL.md 文件，从未创建 references/ 目录。这是典型的「设计没有完整落地」。**自查**：我的 orchestration skill（如果有的话）中的所有内部链接是否都指向实际存在的文件？

2. **中级安全 — SQL 注入向量**（`src/migrate.ts`、`src/searchDocs.ts`）→ `${schema}` 环境变量未转义直接插入 SQL 标识符位置。**根本原因**：TypeScript 模板字符串的视觉简洁性让开发者忘记 SQL 标识符也需要转义（与 SQL 值转义一样重要）。**自查**：我的工具代码中是否有任何 SQL 语句拼接了来自环境变量或用户输入的字符串？

3. **质量 — 模糊量词扩散**（design-postgis、pgvector、hybrid-search 等 skill）→ 「appropriate」、「significantly」、「roughly」等在多个 skill 中反复出现。**根本原因**：专业领域知识越深，越倾向于用「适当」来表达「根据具体情况判断」，但这对 AI 来说等于没有约束。**自查**：我的 skill 文件中是否有 appropriate、comprehensive、significant 等模糊量词？

### 3.4 当时的优化机会

1. **修复 orchestration skill 的 references/ 链接**：将 7 个 `references/X.md` 改为 `../X/SKILL.md`，成本最低但收益最大（orchestration layer 恢复功能）。
2. **统一 SQL 标识符转义**：提取 `safeIdentifier()` 函数，用于 migrate.ts 和 searchDocs.ts 的所有 SQL 模板字符串，同时也是一个安全修复。
3. **将模糊量词替换为可验证阈值**：例如 `appropriate local projections` → `使用当地标准坐标系（如中国 CGCS2000，欧洲 ETRS89）`；`significantly increase latency` → `通常增加 20-50% 查询延迟`。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：skills/postgres/SKILL.md 的 references/ 链接断裂（7 个） | WebFetch skills/postgres/SKILL.md，检查链接格式 | **依然存在且恶化** — 当前共有 9 个 references/ 链接（审计时 7 个，现在 9 个），目录仍不存在 | 超过 3 个月，orchestration skill 仍完全无效；甚至新增了 2 个同样断裂的链接，说明维护者在继续向 references/ 模式添加内容而不知道目录不存在 |
| CLAUDE.md 中 gpt-5 错误 | WebFetch CLAUDE.md | 无法直接验证 | 低优先级问题可能长期未修 |
| SQL 注入向量 | 需读 src/migrate.ts | 无法验证（WebFetch 未覆盖） | 安全问题通常需要私下报告后才有人处理 |

### 4.2 架构演进
audit 后超过 3 个月，orchestration skill 的 references/ 链接不仅没修，还增加了 2 个新的同样断裂的链接（从 7 个增至 9 个）。这表明维护者对 Bug #1 完全不知情——他们在往一个错误方向持续投资，创建新的 references/ 链接而这个目录从未被创建。

### 4.3 新增的可学习模式
暂无新增可学习模式（无法访问完整 repo 历史）。`references/` 链接数量从 7→9 反而是一个新增的反面教材：**当核心设计假设错误时，继续在上面叠加功能只会放大问题**。

---

## 五、校准

### 5.1 我已经在做对的
1. **output format 明确** — bureau/query.md 有详细的 Output Format 节（含 inline citation 格式、Sources 列表规范），比 pg-aiguide 的某些 skill 更规范
2. **避免 orchestration 依赖不存在的资源** — 我的 skill 文件如有交叉引用，应验证目标路径存在
3. **技术关键词的精确化** — bureau 的 tier 系统（proposed/verified/canonical）是精确定义，不用模糊词
4. **单一职责** — bureau 各命令职责单一（note/query/compile/lint 各干各的），与 pg-aiguide 的分层模式一致

### 5.2 挑战 / 验证
**验证了一个已有判断**：「orchestration skill 必须确保所引用的所有文件实际存在」。pg-aiguide 的 orchestration skill 是整个仓库评分最低的文件（85 分），原因正是引用了不存在的资源。这个反面案例强化了：在设计 orchestration layer 时，应该先创建被引用文件，再写 orchestration；而不是先写 orchestration，再「以后再创建」那些文件。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 skill 文件中所有内部链接是否实际存在
find ~/.claude/ -name "SKILL.md" | while read skill; do
  dir=$(dirname "$skill")
  grep -oE '\[.*\]\(([^)]+)\)' "$skill" | \
    grep -oE '\(([^)]+)\)' | tr -d '()' | \
    while read link; do
      [[ "$link" == http* ]] && continue
      fullpath="$dir/$link"
      [ ! -f "$fullpath" ] && echo "BROKEN LINK in $skill: $link"
    done
done
```
命中后怎么办：将断裂链接改为实际存在的文件路径，或创建缺失文件。

```bash
# 检查 TypeScript/Python 代码中是否有未转义的 SQL 字符串拼接
grep -rn '\$\{[a-zA-Z]*\}' src/ 2>/dev/null | grep -E '(CREATE|SELECT|INSERT|ALTER|DROP|TABLE|SCHEMA)'
```
命中后怎么办：提取 `safeIdentifier()` 函数，使用 `escapeIdentifier()` 或正则白名单替换所有此类拼接。

```bash
# 检查 skill 中的模糊量词
grep -rn -iE '\b(appropriate|significant|roughly|mostly|usually optimal|similar)\b' ~/.claude/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：每处替换为具体数字或明确条件，例如「significant」→「超过 50% 的情况」。

### 6.2 灵感 → 实施路径

1. **想法**：为 MarkQWu/graphify 设计一个 orchestration skill，路由到不同的 graph 操作（build/query/visualize/export）
   - **为何可行**：graphify 目前各命令平行，没有统一入口；添加 orchestration skill 可以让用户只说「帮我分析这个代码仓库」就自动被路由到正确的子操作
   - **第一步**：在 graphify/skills/ 下创建 `index/SKILL.md`，先列出所有子 skill 的名字和触发条件，验证所有引用路径存在后再发布，30 分钟

2. **想法**：给 bureau 的 skill 加 API 废弃对照表（bureau 内部操作的 breaking change 记录）
   - **为何可行**：bureau 有 logbook/cabinet 等核心概念，如果将来重命名，需要 AI 知道新旧名称对应关系
   - **第一步**：在 BUREAU.md 中加 `## 历史变更` 节，记录概念重命名时间线，10 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 timescale/pg-aiguide 的核心目的**：将数据库领域的专家知识编码为 AI 可调用的 skill，专治 AI 生成低质量 PostgreSQL 代码

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | 都是「把技术领域知识打包成 AI 可调用工具」；都针对特定技术栈（PG vs 知识图谱）提供深度能力 | pg-aiguide 是静态 skill，graphify 兼有代码执行；pg-aiguide 有 MCP 服务器，graphify 也有 | 高 |
| MarkQWu/shiji-kb | 中 | 都是垂直领域专业知识的 AI 辅助系统 | shiji-kb 是人文领域（诗歌），pg-aiguide 是技术领域 | 低 |

若无直接对应：graphify 是最接近的项目，二者都是「专业知识 → AI 工具」的模式。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| orchestration skill 引用不存在路径 | `grep -E '\[.*\]\(references/' graphify/skills/*/SKILL.md` | graphify 目前无法验证（本次无法读取仓库），但若有 orchestration layer 则需自查 | 高 |
| SQL 模板字符串注入 | `grep -rn '\${' graphify/src/` | graphify 描述为「知识图谱 AI 助手」，可能有 SQL/Cypher 查询，需验证 | 中 |
| 模糊量词 | `grep -rn 'appropriate\|significant' graphify/skills/*/SKILL.md` | 无法验证，但 graphify 涉及技术细节，建议自查 | 低 |

**命中后的具体行动建议**：
- graphify 如有 TypeScript SQL 代码 → 检查所有模板字符串插值，确认已使用参数化查询或 `escapeIdentifier`，30 分钟
- graphify 的 skill 文件 → 运行模糊量词检查 grep，每处修改约 5 分钟

### 7.3 别人的更优方案

1. **领域**：领域知识的完整性覆盖（「知识地图先行」设计）
   - **本案例做法**：8 个 skill 覆盖 PostgreSQL 完整生命周期（design → index → search → timeseries → geospatial），彼此正确交叉引用
   - **我的项目现状**：graphify 的 skills 覆盖程度无法直接验证，但从描述看以「build graph」和「query」为主，可能缺少「graph 维护」「错误恢复」等 skill
   - **如何借鉴**：为 graphify 画出「图谱操作生命周期图」（build → query → update → visualize → export → diagnose），确认每个节点都有对应 skill

2. **领域**：API 废弃表内嵌防止「历史代码幻觉」
   - **本案例做法**：`setup-timescaledb-hypertables/SKILL.md` 有旧→新 API 对照表，防止 AI 生成废弃函数
   - **我的项目现状**：graphify 涉及的图数据库 API（如 Cypher 语法）可能也有版本差异，但 skill 文件中可能未有废弃表
   - **如何借鉴**：在 graphify skill 中加「已废弃 vs 推荐用法」对照节，10 分钟

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：orchestration 依赖验证（不引用不存在的文件）
- **我的做法**：bureau 的命令文件（query.md 等）不依赖外部 references/ 目录，所有内容自包含
- **本案例做法（弱在哪）**：orchestration skill 引用了 9 个不存在的 references/ 文件，且 3 个月后还在往里加新的断裂链接，说明维护者完全不知道这个目录不存在
- **意义**：「自包含 skill」（不依赖外部文件）比「引用外部 references/」更健壮；如果确实需要分模块，应该先创建目标文件再写引用

---

## 八、术语表

### <a name="orchestration-skill"></a>orchestration skill（调度 skill）
> 不执行具体任务，只负责「判断用户意图并分发给合适的子 skill」的特殊 skill。相当于一个路由器或总指挥。pg-aiguide 的 `postgres/SKILL.md` 就是一个 orchestration skill——它告诉 Claude「如果用户需要设计表，去调用 design-postgres-tables；如果需要时序，去调用 setup-timescaledb」。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 制定的开放协议，让 AI 助手可以通过标准接口调用外部工具和服务。pg-aiguide 的 MCP 服务器就是一个通过 MCP 暴露 PostgreSQL 文档搜索能力的外部服务，Claude 可以通过 `mcp__tigerdata__*` 工具调用它。

### <a name="SQL注入"></a>SQL 注入（SQL injection）
> 攻击者通过在用户输入中嵌入 SQL 语法，改变原有 SQL 查询语义的攻击手段。pg-aiguide 的漏洞属于「标识符注入」变种——不是注入 SQL 值，而是注入表名/列名/schema 名，可以导致访问不该访问的数据或破坏数据库结构。
