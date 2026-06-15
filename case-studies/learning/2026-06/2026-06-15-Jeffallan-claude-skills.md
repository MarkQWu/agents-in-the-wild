# Jeffallan/claude-skills — 学习案例

**仓库**：https://github.com/Jeffallan/claude-skills
**Stars**：⭐未收录 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-15（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `manifest-discipline`, `template-design`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`fullstack-dev-skills` 是一个面向全栈开发者的 Claude Code [skill](#skill) 合集插件，当前版本 0.4.15，包含 66 个专项 skill（12 种编程语言专家 + 10 种后端框架 + 6 种前端/移动端 + 基础设施/DevOps/安全/测试），并配有一套"渐进式披露"(progressive disclosure)架构的 [command](#command) 工作流，覆盖 Jira/Confluence 驱动的项目发现→规划→执行全链路。作者 Jeffallan 是一位全栈开发者，仓库通过 Claude Code plugin marketplace 分发，以 `claude plugin install fullstack-dev-skills@Jeffallan` 方式安装。

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── .claude-plugin/
  │   └── plugin.json          # manifest
  ├── skills/                   # 66 个 SKILL.md（每个一个子目录）
  │   ├── api-designer/
  │   │   ├── SKILL.md
  │   │   └── references/      # 详细引用文档（REST、versioning、pagination等）
  │   ├── react-expert/
  │   ├── python-pro/
  │   └── ... (共 66 个)
  ├── commands/                 # 项目管理命令链
  │   ├── common-ground/
  │   │   ├── COMMAND.md       # 通用基础命令
  │   │   └── references/      # 上下文引用文档（无 frontmatter，仅供内部引用）
  │   └── project/
  │       ├── discovery/       # create-epic-discovery → synthesize → approve
  │       ├── planning/        # create-epic-plan → create-implementation-plan
  │       ├── execution/       # execute-ticket → complete-ticket → complete-epic
  │       └── retrospectives/  # complete-sprint
  └── scripts/                 # 5 个 Python/shell 工具（迁移、验证、文档更新）
  ```
- **文件类型分布**：66 个 SKILL.md / 10 个 command / 3 个内部引用文档（无 frontmatter）/ 1 个 manifest
- **编排关系**：skill 是平铺的、独立调用的；command 构成线性工作流链（create-epic-discovery → synthesize-discovery → approve-synthesis → create-epic-plan → create-implementation-plan → execute-ticket → complete-ticket → complete-epic → complete-sprint）
- **跨件契约**：command 通过 `atlassian-mcp` skill 访问 Jira/Confluence，但依赖关系仅在 command 正文中描述，并未通过 `allowed-tools` 声明在 [frontmatter](#frontmatter) 里；skills 通过 `related-skills` 字段互相引用，引用的 slug 都能在 `skills/` 目录下找到对应目录

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单职责 skill + 渐进式引用」—— 每个 skill 只做一件事（api-designer 只管 API 设计），但通过 `references/` 子目录把详细知识按需加载，保持主 SKILL.md 精炼
- **解决什么问题**：让开发者不用在 Claude Code 里从零提 prompt，直接激活领域专家角色（`use api-designer skill`）就能得到专业水准输出
- **做了什么 trade-off**：66 个独立 skill 平铺意味着用户必须主动知道要激活哪个；选择了「菜单宽、单品深」而不是「有 router 帮你自动分发」—— 适合专家用户，对新手略有认知负担
- **反映什么认知模型**：作者把 Claude Code skill 视为「角色卡」，每个 SKILL.md 就是一个有具体 triggers、scope、output-format 的 AI 人格，而不是通用的 system prompt

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单仓多 skill 平铺」**：所有 skill 独立并列于 `skills/` 目录，无父级路由器，用户直接调用目标 skill slug。

模式特征清单：
- 特征 1：每个 skill 配一个独立子目录，支持 `references/` 子文档按需加载
- 特征 2：[manifest](#manifest) (`plugin.json`) 声明 `"skills": "./skills/"` 统一扫描，无需逐一登记
- 特征 3：skill 之间通过 `related-skills` 提示用户组合，但不强制编排
- 特征 4：command 链（工作流）与 skill（角色）分开管理，关注点分离
- 特征 5：所有 SKILL.md 遵循相同 frontmatter schema（name, description, triggers, role, scope, output-format, related-skills）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 开发者个人/团队 toolbelt | ✅ 高度适用 | 用户明确知道需要什么领域，直接激活对应 skill |
| 企业内部知识管理 | ✅ 适用 | 可以增量添加 skill，互不干扰 |
| 面向业务人员的自动化流程 | ❌ 不适用 | 业务人员不知道该激活哪个 skill，需要 router |
| 跨 skill 复杂推理任务 | ❌ 较难 | 无自动编排，多 skill 组合依赖用户手动串联 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单仓多 skill 平铺（本仓库） | Jeffallan/claude-skills | 结构清晰、易扩展、单 skill 可单独更新 | 用户发现成本高，skill 间协作无自动编排 |
| Router + Channels 分层 | 777genius/claude-notifications-go | 用户只需一个入口，路由自动派发 | Router 复杂度高，skill 之间高耦合 |
| 单一超级 skill | addyosmani/agent-skills | 上手简单，一个 skill 覆盖宽 | 臃肿，对话历史消耗大，修改一处影响全局 |

### 2.4 改进空间
1. **当前问题**：commands 没有声明 `allowed-tools`，AI 不知道哪些工具是允许的。**改进做法**：在每个 command 的 frontmatter 里加 `allowed-tools: [atlassian_mcp, mcp__atlassian__create_issue, ...]`。**预期收益**：静态分析可验证 command 的工具依赖，减少权限过宽问题。
2. **当前问题**：api-designer 等 skill 有 `comprehensive`/`appropriate` 等模糊量词。**改进做法**：把「Write comprehensive OpenAPI spec」改成「Write OpenAPI 3.1 spec covering all endpoints, including error responses per RFC 7807 and pagination parameters」。**预期收益**：AI 行为可测试，减少不确定输出。
3. **当前问题**：skill 没有 `## Examples` 块，用户不知道输入什么、期待什么输出。**改进做法**：为每个 skill 添加 1-2 个 input→output 示例。**预期收益**：新用户上手效率提升，skill 行为预期明确。

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-12 当时得分 **92/100**（整体得分高，主要拖分在 commands 层）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 40/100 | 缺少 frontmatter（name/description/output-format）—— 但这是 project config，非真实 artifact 缺陷 |
| commands/project/retrospectives/complete-sprint.md | 50/100 | 缺 `name` + 10+ 模糊量词（comprehensive、appropriate…） |
| skills/api-designer/SKILL.md | 82/100 | 9 个模糊量词匹配（comprehensive×2, proper×2） |
| 51 个 skills 中的大多数 | 96–100/100 | 极少量模糊量词或零问题 |
| .claude-plugin/plugin.json | 100/100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **Manifest 精准声明** → plugin.json 中 `"skills": "./skills/"` 目录级声明让所有 skill 被自动扫描，无需手动登记每个文件 → 可借鉴到 echo-sleuth 的 plugin.json 升级
2. **分层 reference 文档** → api-designer/references/ 包含 REST 规范、versioning 等 5 个引用文档，主 SKILL.md 保持精炼 → echo-sleuth 的 memory-management skill 也可拆出子引用
3. **跨 command 工作流链条清晰** → 每个 command 都有「Workflow Chain」段标注前置和后置步骤 → 我的 claude-for-legal agents 缺这个明确声明
4. **`related-skills` 字段** → 让用户知道哪些 skill 可以组合 → echo-sleuth 各 skill 之间有明显关联但没有声明
5. **脚本都写干净无注入** → 5 个 Python/shell 脚本无 eval/exec，无外网调用，全部 CLEAR → 安全习惯值得学习

### 3.3 当时的缺陷
1. **问题**：10 个 commands 均缺少 `name` 字段 → **根本原因**：command 如果没有 `name`，Claude Code 无法按名字注册和展示它，用户在命令面板里找不到 → **自查**：我的 echo-sleuth 的 recall.md agent 有没有 `name` 字段？
2. **问题**：command frontmatter 全部缺少 `allowed-tools` → **根本原因**：没有机器可读的工具边界声明，静态分析工具无法验证权限范围，运行时权限可能比预期宽 → **自查**：我的所有 skill/agent 都缺这个字段
3. **问题**：skill 存在模糊量词（comprehensive、appropriate、proper）→ **根本原因**：这类词对 AI 来说是不可测试的形容词，执行结果因上下文而异，导致 skill 行为不稳定 → **自查**：echo-sleuth 的 memory-management skill 里有 20 个类似用词

### 3.4 当时的优化机会
1. 为所有 command 添加 `allowed-tools` 声明，列出 atlassian-mcp 等 MCP 工具
2. 为 api-designer、python-pro、angular-architect、pandas-pro、spring-boot-engineer 等 skill 替换模糊量词
3. 确认 commands/common-ground/references/*.md 是否应该有 frontmatter（如不需要，加注释说明是内部引用文档）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 10 个 commands 缺 `name` 字段 | `grep -l "^name:" commands/**/*.md` | **全部修复** ✅（10/10 commands 现在都有 `name`）| 维护者很快响应了这个最高优先级 bug |
| 3 个 reference 文件无 frontmatter | `grep -L "^name:" commands/common-ground/references/` | **保留原状**（仍无 frontmatter）| 维护者确认这是有意设计——它们是内部引用文档，不是用户可调用的 artifact |
| api-designer 有 9 个模糊量词 | `grep -c "comprehensive\|appropriate\|proper" skills/api-designer/SKILL.md` | **还有 8 个**（基本未改）| 模糊量词问题对使用影响较小，维护者优先级较低 |

### 4.2 架构演进
从 audit 时的 plugin.json v0.4.11 升级到当前 v0.4.15：skill 数量从 audit 文件列举的约 65 个增长到现在确认的 66 个（新增了一个 skill）。整体目录结构稳定，未发生重组。command 层加了 `name` 字段后工作流链条现在完整可注册。

### 4.3 新增的可学习模式
命令修复后，工作流链完整了：所有 9 步（create-epic-discovery → complete-sprint）现在都能被 Claude Code 正确识别和调用。这个「完整工作流链 + 每步 workflow-chain 注释」的设计在修复后变得可用，是一个值得学习的 command 编排模式。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责原则**：echo-sleuth 的每个 skill（git-mining、memory-management 等）也各管一事，这与 Jeffallan 的设计哲学一致
2. **独立目录**：echo-sleuth 的 `skills/memory-management/SKILL.md` 结构和 Jeffallan 的 `skills/api-designer/SKILL.md` 结构相同，子目录隔离
3. **Manifest 统一扫描**：claude-for-legal 也用 plugin.json 声明 `"skills":` 目录而不是逐个文件
4. **Triggers 字段**：echo-sleuth 的 SKILL.md 有 triggers 字段帮助 AI 知道何时激活

### 5.2 挑战 / 验证
这次案例验证了「command 必须有 `name` 字段才能被注册」这个我之前不确定的细节——过去我以为 `description` 就够了，现在确认 `name` 是注册的必要条件。同时，模糊量词问题是普遍现象（连 92 分的优秀仓库也有），但高质量仓库的修复策略是「技术名称里的词不算模糊量词」（如 `comprehensive OpenAPI spec` 中的 comprehensive 是行业术语），这个区分很有价值。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 agent/skill 是否有 name 字段
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "*.md" | xargs grep -L "^name:" 2>/dev/null

# 检查我的 skill 是否有模糊量词
grep -rn -E '\b(comprehensive|appropriate|robust|relevant|carefully|thorough|significant)\b' /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null

# 检查我的 commands 是否有 allowed-tools
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null | grep "\.md$"
```
命中后怎么办：
- 缺 `name`：在 frontmatter 第一行加 `name: <slug-without-md-extension>`
- 模糊量词：用具体数量/步骤替换（「comprehensive」→「covering all X scenarios listed below」）
- 缺 `allowed-tools`：根据 skill 实际使用的工具在 frontmatter 添加

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 的 skill 添加 `references/` 子目录，把长段落知识移出主 SKILL.md
   - **为何可行**：memory-management skill 目前有很长的「Staleness Scoring」数学公式，适合拆出
   - **第一步**：在 `skills/memory-management/` 下新建 `references/staleness-formula.md`，主 SKILL.md 里改成 Reference Guide 表格，约 20 分钟
2. **想法**：给 claude-for-legal 的 agents 添加工作流链注释
   - **为何可行**：legal 插件有 reg-change-monitor → launch-watcher 等隐式流程，明确标注有助于复用
   - **第一步**：在每个 agent.md 添加「## Workflow Position」段，约 15 分钟/个

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 Jeffallan/claude-skills 的核心目的**：为全栈开发者提供专业领域 skill 合集，激活后即获得该领域专家能力
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同为 Claude Code plugin，有多个 skill，通过 plugin.json 分发 | Jeffallan 是技术领域 skill（开发工具），echo-sleuth 是元功能（挖掘对话记忆）| **高** |
| MarkQWu/claude-for-legal | 中 | 都有 agents + commands 层 | claude-for-legal 有 agents 概念，Jeffallan 无 agents 只有 skills | **中** |
| MarkQWu/drama-workshop-skills | 低 | 都是专域技能合集 | drama-workshop 是中文内容创作，不是 Claude Code plugin 标准结构 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 缺 `## Examples` 块 | `grep -rL "## Example" skills/*/SKILL.md` | echo-sleuth：4/4 个 skill 全部命中 | **高** |
| 模糊量词泛滥 | `grep -c "comprehensive\|appropriate\|relevant\|careful" skills/*/SKILL.md` | echo-sleuth：20 处匹配（memory-management、experience-synthesis 最多）| **中** |
| 缺少 `allowed-tools` | `grep -rL "allowed-tools"` | echo-sleuth：4/4 skills + 5/5 agents 全部命中 | **中** |

**命中后的具体行动建议**：
- `echo-sleuth/skills/memory-management/SKILL.md` → 添加 `## Examples` 块：`input: "audit memory files for staleness" → output: "Found 3 stale files (score>80): [list with reasons]"` → 约 10 分钟
- `echo-sleuth/skills/experience-synthesis/SKILL.md` → 把「carefully synthesize」→「extract and list each distinct decision with its rationale」→ 约 5 分钟

### 7.3 别人的更优方案

1. **领域**：`references/` 子目录按需加载知识
   - **本案例做法**：`skills/api-designer/references/` 包含 5 个专题文档（rest-patterns.md、versioning.md 等），主 SKILL.md 通过 Reference Guide 表格按需引用
   - **我的项目现状**：`echo-sleuth/skills/memory-management/SKILL.md` 直接把所有内容（格式规范、数学公式、路由规则）塞在一个文件里，700+ 行
   - **如何借鉴**：把 staleness scoring 公式拆到 `references/staleness-formula.md`，把 claim extraction 规则拆到 `references/extraction-rules.md`；在主文件加 Reference Guide 表格

2. **领域**：工作流 command 链清晰标注
   - **本案例做法**：每个 command 有「Workflow Chain」注释，标注「前一步是哪个 command → 本步 → 后一步是哪个 command」
   - **我的项目现状**：`claude-for-legal` 的 agents 之间有隐式依赖（reg-change-monitor 的输出被 launch-watcher 使用），但没有任何文档化的链条声明
   - **如何借鉴**：在每个 agent 的 frontmatter 或正文加「Workflow: [prev] → [this] → [next]」

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：多语言内容 + 版本化 frontmatter
- **我的做法**：echo-sleuth 的 skill 有 `version: 0.2.0` 字段，方便追踪变更
- **本案例做法**：Jeffallan 的 SKILL.md 有版本（如 api-designer 是 1.1.0），但 command 文件没有版本
- **意义**：版本字段有助于用户知道 skill 是否是最新的，也方便调试

---

## 八、术语表

### <a name="skill"></a>skill
> Claude Code 里的「专家角色卡」，写成一个 SKILL.md 文件。你在对话里说「use api-designer skill」，Claude 就会加载这个文件、扮演 API 设计专家角色。

### <a name="command"></a>command
> Claude Code 的斜杠命令文件（如 `/complete-sprint`），写成一个 .md 文件，用户在对话框输入 `/` 就能调出列表。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（name、description、triggers 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才知道这个 skill 叫什么、什么时候触发。

### <a name="manifest"></a>manifest
> 项目的「清单文件」（这里是 `plugin.json`），告诉 Claude Code 这个插件有哪些 commands、skills 的目录路径。如果 manifest 写错，文件在硬盘上存在也不会被加载。

### <a name="allowed-tools"></a>allowed-tools
> frontmatter 里的一个字段，声明这个 skill/command 允许调用哪些工具（如 `bash`、`mcp__atlassian__create_issue`）。没有声明就相当于「不限制」，权限边界不明确。
