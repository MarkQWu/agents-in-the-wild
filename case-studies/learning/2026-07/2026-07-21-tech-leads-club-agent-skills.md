# tech-leads-club/agent-skills — 学习案例

**仓库**：https://github.com/tech-leads-club/agent-skills
**Stars**：N/A（exemplar_published=true 覆盖星数门槛）| **来源**：upstream audit
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-07-21（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `cross-reference`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`tech-leads-club/agent-skills` 是 Tech Leads Club 社区维护的大型 Claude Code skill 仓库，覆盖 14 个工程领域（architecture、cloud、creation、design、development、gtm、learning、monitoring、performance、quality、security、tooling、web-automation、decision-making），截至当前 HEAD 共有 **84 个 SKILL.md 文件**（审计时 78 个，净增 6 个）。

关键事实：
- GTM 相关 skills（16 个）均标注 `original_author: Chad Boyda / agent-gtm-skills`，是从 [agent-gtm-skills](#agent-gtm-skills) 仓库 fork 过来的，并有 `modified_by` 叠加标记
- 仓库内含一个独立的 `packages/mcp/` 子包，提供 MCP server（可通过 MCP 协议远程调用 skills），这是除 Claude Code 插件之外的第二种分发机制
- `packages/mcp/.claude-plugin/plugin.json` **仍然缺少 `version` 字段**（audit 时已发现，当前 HEAD 持续）
- 架构 skills（10 个）构成一条分析流水线（`domain-analysis → domain-identification-grouping → component-identification-sizing → coupling-analysis → decomposition-planning-roadmap`），skill 之间通过 prose 描述引用，无 `references/` 链接

### 1.2 架构剖析
```
agent-skills/
├── skills/                         # 84 个 SKILL.md，按领域目录分组
│   ├── (architecture)/             # 架构分析流水线（10 个 skills）
│   │   ├── domain-analysis/
│   │   ├── domain-identification-grouping/
│   │   ├── component-identification-sizing/
│   │   ├── coupling-analysis/
│   │   └── decomposition-planning-roadmap/
│   ├── (cloud)/                    # 云部署（Vercel/Netlify/Cloudflare/AWS/Render）
│   ├── (creation)/                 # 文档/规格/RFC/ADR 创建
│   ├── (design)/                   # Figma 设计实现
│   ├── (development)/              # 开发辅助（Jira/Confluence/React Native/NestJS）
│   ├── (gtm)/                      # GTM 营销（fork from Chad Boyda）
│   ├── (learning)/                 # 学习机会识别
│   ├── (monitoring)/               # Sentry 监控
│   ├── (performance)/              # Web 性能优化
│   ├── (quality)/                  # 代码质量/SEO/a11y
│   ├── (security)/                 # 安全审查
│   ├── (tooling)/                  # NX/gh/chrome-devtools/mermaid-studio
│   ├── (web-automation)/           # Playwright 自动化
│   └── (decision-making)/          # the-fool（逆向决策辅助）
└── packages/
    └── mcp/                        # MCP server（skills 的第二种分发机制）
        ├── src/                    # TypeScript MCP 实现
        │   ├── tools/              # list/search/fetcher 工具
        │   └── registry.ts         # skill 注册表
        └── .claude-plugin/
            └── plugin.json         # 缺 version 字段（bug）
```

- **文件类型分布**：84 个 SKILL.md（无 agents，无 commands）+ 1 个 MCP 包（TypeScript）
- **编排关系**：所有 skills 并列，无 router，无 agent 入口层。用户直接通过 Claude Code skill 名称调用，或通过 MCP tools（`list_skills`/`search_skills`/`fetch_skill`）按需加载
- **跨件契约**：架构类 skills 之间存在 prose-level 引用（「在 domain-analysis 之后运行」），但无机器可读的 `references/` 链接，重构时不会触发「断裂引用」提示

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「按域聚合的 skill 目录 + MCP 远程分发」——社区贡献者可以按领域分工贡献，MCP server 提供「按需拉取」而非「全量加载」的访问模式
- **解决什么问题**：工程师日常有大量「固化流程」（部署、代码审查、性能分析、SEO）可以 skill 化，但单个人很难覆盖所有域；Tech Leads Club 用社区协作来填充这个 skill 库
- **做了什么 trade-off**：技能越多越有价值，但「大而全」带来了质量参差不齐的问题——84 个 skills 中，最低分 70、最高分 100，方差显著
- **反映什么认知模型**：作者把 Claude Code 技能库视为「工程师助手工具箱」，每个 skill 是一个可复用的 SOP（标准作业程序），MCP server 是让 Claude「自助查阅工具手册」的检索系统

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「域分桶大型 Skill 目录 + MCP 分发层」

这个模式的核心是：不靠单一 agent 或 router 来调度，而是让目录结构本身承担「组织职责」，用 MCP server 承担「按需检索/加载」职责。

模式特征清单：
- **特征 1**：技能按领域目录分组，目录名即检索单元（`(architecture)/`）
- **特征 2**：MCP server 提供 `search_skills` + `fetch_skill` 两阶段访问——先搜再读，避免上下文爆炸
- **特征 3**：GTM skills 明确标注 `original_author` + `modified_by`，追溯贡献来源
- **特征 4**：架构类 skills 构成的流水线是隐式的（prose 描述），不是硬编码的编排链
- **特征 5**：无 agent 入口层，每个 skill 都可以被直接调用，也可以由用户按序手动调用

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖多个领域的工具集 | ✅ 高度适用 | 目录分桶让各领域独立演进 |
| 社区协作共建 | ✅ 适用 | 按域分工，PR 冲突少 |
| 需要「按需检索」而非「全量加载」skills | ✅ 适用 | MCP 两阶段访问解决上下文限制 |
| 强调编排顺序的流水线任务 | ❌ 不适用 | 没有 agent/router 层，顺序靠人工记忆 |
| 需要 skill 间通信/传参 | ❌ 不适用 | 所有 skills 独立，无共享状态机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 域分桶大目录 + MCP（本仓库） | tech-leads-club/agent-skills | 可扩展至数百 skills，社区协作友好 | 质量参差，无强制编排 |
| 小型聚焦工具集（单域） | pe-menezes/fin-claude-plugin | 质量统一，领域一致性强 | 覆盖范围有限 |
| Agent 编排 + skills | shinpr/claude-code-workflows | 流水线显式，可测试 | 扩展新领域需改编排层 |

### 2.4 改进空间

1. **当前问题**：`security-best-practices/SKILL.md` 等高频使用 skill 的模糊量词密度极高（10 个命中，-20 分封顶），会直接影响 Claude 决策质量。**改进做法**：把「ensure」「appropriate」等词替换为动词 + 具体标准（如「run `npm audit` with exit code 0 required」）。**预期收益**：security skill 从 70 分提升至 90+，Claude 输出的安全建议更可操作。

2. **当前问题**：26 个 skills 缺少 `## Output` 格式声明。**改进做法**：每个缺少的 skill 加一个 3-5 行的 Output 小节，描述交付物的格式（报告结构/严重等级标签/代码变更格式）。**预期收益**：Claude 执行后知道输出什么，一致性提升。

3. **当前问题**：`packages/mcp/.claude-plugin/plugin.json` 缺少 `version` 字段，影响 MCP 插件注册和去重。**改进做法**：加一行 `"version": "1.0.0"`，是一行修复。**预期收益**：MCP 插件可正确注册、升级、去重。

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-07 当时整体得分 **93/100**，78 个工件。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| (security)/security-best-practices/SKILL.md | 70 | 模糊词 ×10 (-20 封顶) + 无输出格式(-10) |
| (development)/codenavi/SKILL.md | 80 | 模糊词 ×11 (-20 封顶) |
| (creation)/cursor-subagent-creator/SKILL.md | 80 | 模糊词 ×11 (-20 封顶) |
| (tooling)/nx-run-tasks/SKILL.md | 81 | 模糊词 ×7 (-14) + 无输出格式(-5) |
| (cloud)/render-deploy/SKILL.md | 82 | 模糊词 ×4 + 无输出格式(-18 合计) |
| packages/mcp/.claude-plugin/plugin.json | 90 | 缺 version 字段 (bug -10) |
| (monitoring)/sentry/SKILL.md | 100 | 满分 |
| (gtm)/ai-pricing/SKILL.md | 100 | 满分 |
| (architecture)/domain-analysis/SKILL.md | 100 | 满分 |

### 3.2 当时值得借鉴的模式

1. **满分 Sentry 监控 skill 设计**：`(monitoring)/sentry/SKILL.md` 100 分——无模糊词，有具体输出格式，工具声明完整。为什么好：一个单一职责、边界清晰的 skill 完全可以做到满分。如何借鉴：设计新 skill 时，把 sentry/SKILL.md 当参考模板。

2. **MCP 两阶段访问设计**（search → fetch）：先 `search_skills` 返回清单，用户选择后 `fetch_skill` 拉全文。为什么好：避免把 84 个 SKILL.md 全加载进上下文。如何借鉴：大型 skill 集合应该提供「检索入口」而非要求一次性全量加载。

3. **架构分析流水线隐式设计**：5 个架构 skills 构成可单独使用、也可流水线串联的分析链。为什么好：用户可以只用其中一个 skill 单独运行，也可以按顺序全部跑一遍。如何借鉴：设计相关 skills 时保持可独立调用，同时在 SKILL.md 描述里说明「前置/后置」关系。

4. **GTM fork 的原作者归属**：所有 fork 自 Chad Boyda 的 skills 都有 `original_author` 字段。为什么好：追溯来源、避免版权/贡献纠纷、上游更新时知道哪些 skill 是 fork 的。如何借鉴：在自己的 skill 集里，凡是参考了其他仓库的 skill，在 frontmatter 里加 `source:` 字段。

### 3.3 当时的缺陷

1. **高频模糊量词（security-best-practices 命中 10 个）**：根本原因：安全领域 skill 往往是从人类安全手册翻译过来的，原文就充满「适当的」「确保」「综合性的」——这在人类沟通里是可接受的，但 AI 需要可验证标准，「确保使用强密码」对 Claude 来说和「密码最低 12 位，包含大写、数字、特殊字符」不是同一回事。自查：我的 skills 里有没有从通用安全文档直接翻译过来的内容？

2. **26 个 skills 缺输出格式**：根本原因：作者可能更关注「描述步骤」，而不关注「描述产出」；但对 Claude 来说，不知道该输出什么格式，它会自由发挥导致一致性差。自查：我的 bureau skills 有没有缺 `## Output` 声明的？

3. **plugin.json 缺 version 字段**（MCP 包 bug）：根本原因：作者同时维护 Claude Code 插件和 MCP server 两套分发机制，两套配置文件格式不同，version 字段被遗漏在 MCP 侧。自查：我的 plugin.json 都有 version 字段吗？

### 3.4 当时的优化机会

1. **首要 Bug 修复**：`packages/mcp/.claude-plugin/plugin.json` 加 `"version": "1.0.0"` 一行
2. **最高 ROI 质量修复**：security-best-practices、codenavi、cursor-subagent-creator 三个文件替换模糊词，能一次性提升整体 corpus 平均分约 1 分
3. **批量补输出格式**：26 个缺 `## Output` 的 skills 各加 3-5 行

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 缺 version（MCP bug） | `grep "version" packages/mcp/.claude-plugin/plugin.json` | **持续**：当前 plugin.json 仍无 version 字段（已验证）| 这个单行修复 4+ 个月仍未合入，说明 MCP 分发机制不是维护优先级 |
| security-best-practices 模糊量词 | `grep -c "appropriate\|ensure" skills/(security)/security-best-practices/SKILL.md` | 未能直接验证（路径含括号，grep 可能需转义）| 需实际检查文件内容 |
| 26 个 skills 缺输出格式 | `find skills -name "SKILL.md" \| xargs grep -L "## Output" \| wc -l` | 84 个 skills 中仍大量缺失（估计持续）| 批量修复需要大量 PR，维护力量不足 |

### 4.2 架构演进

**从 78 → 84 个 skills**（+6 个）：说明社区仍在活跃贡献新 skills。新增 skills 覆盖了新领域（具体文件未确认），整体规模扩张。

MCP server（`packages/mcp/`）依然在维护（TypeScript 源码完整），说明双分发策略（Claude Code 插件 + MCP）是长期路线，不是临时设计。

### 4.3 新增的可学习模式

当前 HEAD 中架构 skills 的「流水线 README」模式值得注意：部分 skills 在 frontmatter 里有 `sequence:` 或类似字段提示「在哪个分析步骤之后使用」——虽然不是正式的跨件引用，但比纯 prose 引用更可发现。

---

## 五、校准

### 5.1 我已经在做对的

1. **领域聚焦**：我的 bureau 和 graphify 都是领域专注的小型 skill 集，质量比大而全的集合更均匀
2. **单职责 skill**：bureau 的 recall/capture/compile/scribe 每个只做一件事，与 sentry/SKILL.md 100 分的设计一致
3. **MCP 后端隔离**：graphify 的 MCP 层与 NL 层分离，与 tech-leads-club 的 MCP 分发思路相通
4. **具体输出格式**：bureau 的部分 skills 有 `## Output` 格式声明（虽不完整）

### 5.2 挑战 / 验证

本案例挑战了我「大而全 skill 库总比小集合好」的假设。tech-leads-club 的 84 个 skills 中最低 70 分，说明规模和质量之间有实际张力——在资源有限的情况下，维护 84 个 skills 的质量比维护 20 个 skills 的质量更难。这验证了我坚持「小而精」策略的合理性。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 有没有 ## Output 格式声明
find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" | \
  xargs grep -L "## Output\|## 输出\|output format" 2>/dev/null
# 命中后：每个缺少的 skill 加 3-5 行输出格式描述，20-30 分钟可完成

# 检查 bureau skills 里的模糊量词密度
grep -rn "ensure\|appropriate\|comprehensive\|robust\|relevant\|leverage\|optimal" \
  /tmp/my-repos/MarkQWu-bureau/skills/ --include="*.md" 2>/dev/null
# 命中后：把每个模糊词替换为可验证标准（「ensure X」→「verify X outputs Y」）

# 检查我的 plugin.json 是否包含 version 字段
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau \
  -name "plugin.json" -exec grep -l "." {} \; | \
  xargs -I{} bash -c 'echo "FILE: {}"; grep version {} || echo "MISSING"' 2>/dev/null
# 命中（缺 version）后：加 "version": "x.y.z" 行
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 skills 增加 MCP 分发层（类似 tech-leads-club 的 packages/mcp/）
   - **为何可行**：bureau 有 7 个 skills，在 Claude Code 对话里全量加载上下文开销较大；MCP 按需加载能减少 token 浪费
   - **第一步**：参考 tech-leads-club 的 `packages/mcp/src/registry.ts`，写一个最小化版本（指向 bureau/skills/ 目录），用 `npx` 启动（4-6 小时）

2. **想法**：为 gstack 的架构类 skills 添加「前置 skill」声明
   - **为何可行**：gstack 有 `spec/SKILL.md`→`plan-design-review/SKILL.md`→`review/SKILL.md` 的隐式顺序
   - **第一步**：在这 3 个 skills 的 frontmatter 里加 `prerequisites:` 字段，注明「建议先运行 X」（30 分钟）

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **tech-leads-club/agent-skills 核心目的**：覆盖 14 个工程领域的大型 skill 目录，社区共建，通过 MCP 按需分发

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是工程师工具集；都有多个工程领域覆盖 | gstack 更个人化（Garry Tan 风格），agent-skills 更社区化 | **高** |
| MarkQWu/drama-workshop-skills | 中 | 都是「curated skill 集合」，有外部来源 | drama-workshop-skills 聚焦特定垂直领域 | 中 |
| MarkQWu/bureau | 低 | 都有 skill 层 | bureau 是知识管理系统，非工程工具集 | 低 |
| MarkQWu/graphify | 低 | 都有 MCP 分发 | graphify 是代码图谱工具，非 skill 目录 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skills 缺 `## Output` 格式 | `find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" \| xargs grep -L "## Output"` | **命中 10+ 个**（setup-deploy, diagram, gstack-upgrade 等）| **高** |
| 模糊量词高密度 | `grep -c "appropriate\|ensure\|relevant" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | gstack 总计 354 个命中（部分为 README 内容）| 中 |
| plugin.json 缺 version | `grep "version" /tmp/my-repos/MarkQWu-gstack/plugin.json 2>/dev/null` | 需检查（gstack 的 plugin.json 路径待确认）| 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的 `diagram/SKILL.md` → 在文件末尾加 `## Output Format` 节，说明「生成的图表类型（mermaid/plantuml）和内容结构」；5 分钟
- `MarkQWu/gstack` 的 `spec/SKILL.md` → 检查有无输出格式声明，若无则加「PRD/spec 的结构：概述 + 用户故事 + 验收标准」；10 分钟
- 对 gstack 所有使用「ensure」的 skills，逐一替换为 `verify ... results in ...` 的可验证形式；2-3 小时批量完成

### 7.3 别人的更优方案

1. **领域**：MCP 两阶段访问（search → fetch）减少上下文膨胀
   - **本案例做法**：`packages/mcp/src/tools/search-tool.ts` 先返回 skill 清单，`fetcher-tool.ts` 按需加载单个 SKILL.md 全文
   - **我的项目现状**：`MarkQWu/graphify` 的 skill 文件在调用时全量加载，无检索层
   - **如何借鉴**：在 graphify 的 skill 目录下加一个 `index.yaml`（列出所有 skills + 一行描述），主 skill 先读 index 再按需读具体文件

2. **领域**：GTM fork 的 `original_author` 归属追踪
   - **本案例做法**：所有 fork skills 的 frontmatter 有 `metadata.original_author` 和 `metadata.modified_by` 字段
   - **我的项目现状**：`MarkQWu/drama-workshop-skills` 有外部来源的 skills，但无来源追踪字段
   - **如何借鉴**：在 drama-workshop-skills 的 SKILL.md frontmatter 里加 `source:` 字段，记录原始参考来源

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：聚焦深度 vs 覆盖广度
- **我的做法**：`MarkQWu/bureau` 7 个 skills 每个都深度覆盖自己的领域（recall 是完整的记忆检索协议，compile 是完整的知识编译流程）
- **本案例做法**：84 个 skills 中最低分 70 分，`security-best-practices` 等高频 skill 质量不达标，说明规模扩张牺牲了质量
- **意义**：维护小而精的 skill 集比维护大而全的 skill 目录更可持续，这是我的项目策略在数据上得到验证的地方

---

## 八、术语表

### <a name="agent-gtm-skills"></a>agent-gtm-skills
> Chad Boyda 开源的 GTM（Go-To-Market，市场进入策略）Claude Code skills 集合。「GTM」在创业/营销语境里指一个产品从开发完成到触达目标用户的完整策略，包括定价、渠道、冷启动、内容营销、销售话术等。tech-leads-club/agent-skills 的 GTM 类 skills（`ai-seo/`、`ai-cold-outreach/`、`positioning-icp/` 等 16 个）是从这里 fork 过来的。

### <a name="MCP-server"></a>MCP server
> Model Context Protocol 服务端程序。Claude 通过 `.mcp.json` 连接 MCP server，然后可以调用 server 提供的工具（tools）。tech-leads-club 的 MCP server 提供 `search_skills`、`fetch_skill` 两个工具，让 Claude 在对话里按需检索和加载 skill 内容，不需要把 84 个 SKILL.md 全部加载进上下文。

### <a name="plugin.json"></a>plugin.json
> Claude Code 插件的注册文件（[manifest](#manifest)）。必须包含 `name`（插件标识符）和 `version`（版本号）字段，Claude Code 插件注册、去重和升级都依赖 version 字段。tech-leads-club 的 MCP 包的 `plugin.json` 缺少 `version` 字段，导致插件无法正确注册，是一个需要单行修复的 bug。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目有哪些组件以及如何注册它们。`plugin.json` 是 Claude Code 插件的 manifest。若 manifest 里的字段不完整（如缺少 `version`），系统无法正确识别和管理这个插件。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包围的 YAML 配置块。对 SKILL.md 来说，frontmatter 声明了 skill 的 `name`（调用名）、`description`（触发条件描述）、`allowed-tools`（可用工具列表）等关键元数据。Claude Code 读 skill 时先解析 frontmatter，再读正文。

### <a name="vague-quantifier"></a>模糊量词
> 在 NL programming 语境下，「模糊量词」指缺乏可验证标准的形容词或副词，如「appropriate（适当的）」「ensure（确保）」「comprehensive（全面的）」「robust（健壮的）」。这些词在人类写给人类看的文档里是可接受的，但在 AI 执行的 skill 里，它们给了 Claude 过多的自由裁量空间，导致每次执行结果不一致。NLPM 规则把每个模糊量词扣 2 分，上限 -20 分（一个 skill 里超过 10 个就封顶不再追加扣分）。
