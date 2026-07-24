# josstei/maestro-orchestrate — 学习案例

**仓库**：https://github.com/josstei/maestro-orchestrate
**Stars**：370 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-30（历史快照）| **生成日期**：2026-07-24（基于当前 HEAD）
**主题标签**：`router-channels`, `model-pinning`, `vague-quantifier`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

maestro-orchestrate 是一个**跨 AI 运行时的多角色 agent 编排平台**，允许同一套角色定义（分析师、架构师、前端工程师等）在 Gemini CLI、Claude Code、Codex、Qwen Code 四个 AI 系统上并发运行。核心理念：把「角色知识」从「运行时实现」中解耦出来，一次定义、多处复用。

关键事实：
- 创建于 2025-2026 年，作者 josstei 维护，以 npm 包形式分发（`npx maestro-orchestrate init`）
- 支持 4 个 AI 运行时：Gemini CLI、Claude Code（claude 目录）、Codex、Qwen Code
- 通过 [hooks](#hooks) 系统实现运行时适配（PostToolUse 拦截器将请求路由到对应 adapter）
- 发布时同时包含 `plugin.json`（Claude Code 插件）和 `gemini-extension.json`（Gemini 扩展）

### 1.2 架构剖析

**目录结构**（当前 HEAD，主要层级）：
```
maestro-orchestrate/
├── src/agents/          # 原始 agent 定义（39 个）—— 多运行时通用版
├── claude/
│   ├── agents/          # MCP 委托存根（39 个，score 75）
│   └── src/agents/      # Claude 增强版（39 个，score 85-95）
├── plugins/maestro/
│   └── src/agents/      # Maestro 插件专用版（39 个，score 89-95）
├── hooks/               # 运行时适配 hook（hooks.json + adapter.js × 2）
├── scripts/             # 安装脚本（generate.js、install-codex-plugin.js 等）
├── mcp/                 # MCP 服务器（maestro-server.js）
└── package.json
```

**文件类型分布**（当前 HEAD）：
- Agent 定义：~156 个 .md 文件（4 层 × 39 个）
- Hooks：1 个 hooks.json + 3 个 adapter.js
- MCP 配置：claude/.mcp.json
- 清单：plugin.json（Claude Code）+ gemini-extension.json

**编排关系**：
```
用户请求
  │
  ├─ Claude Code ──→ claude/agents/ 存根 ──→ MCP get_agent ──→ claude/src/agents/ 完整定义
  ├─ Gemini CLI ──→ src/agents/ 通用版
  ├─ Codex ──→ scripts/install-codex-plugin.js 安装 → plugins/maestro/src/agents/
  └─ Qwen Code ──→ hooks/adapters/qwen-adapter.js
```

**跨件契约**：`claude/agents/` 存根的 body 只写一句话 `"Agent methodology loaded via MCP tool get_agent"`，所有实际逻辑在运行时通过 [MCP](#MCP) 工具注入。这使得 claude/agents/ 存根能保持极简，但产生了「MCP 宕机时 25 个 agent 全部变空壳」的单点故障。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：运行时无关性（runtime-agnostic）。角色定义一次，MCP 注入给 Claude，hooks 适配给其他运行时。
- **解决什么问题**：AI 工具生态碎片化。一家公司可能同时用 Claude + Codex + Gemini，不想为每个平台重写 25 个角色的 system prompt。
- **做了什么 trade-off**：
  - ✅ 用存根换来了 Claude Code 集成的干净注册（仅 frontmatter，无重复正文）
  - ❌ 存根层因此在 NLPM 评分中得分 75（-15 无示例，-10 无输出格式），与完整版差了 14-20 分
  - 存根设计是「有意识的质量牺牲」，但没有任何文档说明这是刻意选择
- **反映什么认知模型**：作者将 agent 视为「协议适配器」而非「自包含的 AI 人格」——每个角色是一个接口规范，底层实现由运行时决定。这是企业级设计思维（接口 vs 实现分离），在个人工具场景下显得工程过度。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：多运行时适配器 + 分层质量梯度（Multi-Runtime Adapter + Tiered Quality Gradient）**

这个模式的核心是：同一份角色知识用 N 个「质量层级」存在于仓库中，每层针对不同的集成方式做了不同程度的实现。

模式特征清单：
- **特征 1**：存在 3-4 个并行实现层（src/ → claude/src/ → claude/ 存根 → plugins/），每层分别优化不同目标（通用性 / Claude 增强 / 零内容注册 / 插件发布）
- **特征 2**：最低质量层（存根，75 分）有意牺牲 NL 质量换取运行时灵活性
- **特征 3**：MCP 服务器作为「知识注入总线」在运行时弥补存根的内容空缺
- **特征 4**：[Output Contract](#output-contract) 共享模板在所有 agent 文件中统一定义输出规范，但同时把模糊量词传播到整个仓库
- **特征 5**：hooks 系统作为运行时路由层，在 PostToolUse 时选择正确的适配器

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业内多 AI 工具并存（Claude + Codex + Gemini） | ✅ 高度适用 | 核心设计目标就是解决这个问题 |
| 个人开发者只用一个 AI 工具 | ❌ 过度设计 | 4 层结构的维护成本远超收益 |
| 需要统一角色库供团队共享 | ✅ 适用 | 单仓管理 + npm 分发是正确选择 |
| 角色需要频繁个性化定制 | ❌ 不适用 | 多层同步会让定制扩散成维护噩梦 |
| 离线或内网环境 | ❌ 不适用 | 存根层依赖 MCP 服务器，无网络降级为空壳 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多运行时适配器（本仓库） | josstei/maestro-orchestrate | 一份定义多处复用；MCP 集中管理 | 存根层 NL 质量低；MCP 单点故障无降级 |
| 平铺 skill 单仓（Flat Skill Repo） | MarkQWu/gstack | 简单直接；每个 SKILL.md 自包含 | 多平台兼容需手动复制；无统一注入机制 |
| 分仓 + 独立 plugin（Split Repo） | levnikolaevich 演进后结构 | 各平台独立迭代；互不干扰 | 跨仓同步困难；一致性维护成本高 |

### 2.4 改进空间

1. **当前问题**：存根层（claude/agents/）得分 75，比完整版低 14-20 分，但没有任何说明这是有意为之。**改进做法**：在 claude/agents/ 目录添加 `README.md`，明确写「这些文件是 MCP 委托存根，不包含实体内容，请看 claude/src/agents/ 获取完整实现」，并在存根的 agent body 里加一行 `## 降级说明\n如果 MCP 不可用，本 agent 将回退到 claude/src/agents/ 中同名文件的静态定义`。**预期收益**：让 NLPM 评分者（人或工具）理解存根是刻意设计，而不是内容缺失。

2. **当前问题**：Output Contract 共享模板中的 `"additional necessary work discovered"` 把 `necessary` 这个模糊量词传播到所有使用该模板的 agent（约 75 个，每个 -2 分）。**改进做法**：将 `necessary` 替换为 `unexpected` 或具体描述（`"additional work outside original scope discovered"`）。**预期收益**：一处修改消除约 75 个文件的 -2 扣分。

3. **当前问题**：75 个 agent 文件未声明 `model:` 字段，运行时使用继承值，行为不可预期。**改进做法**：在 generate.js 的模板生成逻辑中加入 `model: claude-sonnet-5` 作为默认值。**预期收益**：每个 agent -5 扣分消除；更重要的是跨运行时行为可预期。

---

## 三、过去审查发现（2026-04-30 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-30 当时得分 87/100。代表性文件评分：

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| claude/agents/（25 个存根） | 75 | 无 body 示例（-15），无输出格式（-10） |
| src/agents/design-system-engineer.md | 79 | `appropriate` ×4 + `necessary` ×1（-10 模糊词） |
| claude/src/agents/tester.md | 83 | 无输出格式（-10），`comprehensive` ×1（-2） |
| claude/src/agents/coder.md | 85 | `appropriate` ×2, `proper` ×2, `necessary`（-10） |
| claude/src/agents/accessibility-specialist.md | 95 | 未声明 model（-5） |

### 3.2 当时值得借鉴的模式

1. **Output Contract 统一输出契约** → 好处：所有 agent 用同一模板定义「交付物格式」，阅读者看一个就懂全部。根本原因：标准化减少了认知负担。借鉴路径：在我的 gstack 的每个 SKILL.md 里加一个 `## 输出格式` 节，内容参照统一模板。

2. **MCP 运行时知识注入** → 好处：存根 vs 实现分离，Claude Code 注册的是轻量文件，实际知识在运行时按需注入。根本原因：分离关注点，注册层和内容层独立演化。借鉴路径：如果我的 graphify 技能库将来支持多个 AI 运行时，可以用类似的 MCP 委托架构。

3. **Zero NL Bugs 零缺陷** → 好处：100 个 agent 全部有合法的 `name` 和 `description` frontmatter，无断裂交叉引用。根本原因：使用 generate.js 代码生成确保 frontmatter 格式一致。借鉴路径：对于批量 agent 的场景，用脚本生成比手写更可靠。

4. **多文档体系** → 好处：ARCHITECTURE.md、EXAMPLES.md、USAGE.md、OVERVIEW.md 分层覆盖不同读者。根本原因：不同深度的读者有不同信息需求。借鉴路径：我的仓库目前只有 README.md，可以拆分出 EXAMPLES.md 和 ARCHITECTURE.md。

### 3.3 当时的缺陷

1. **存根层无降级文档**（claude/agents/ 25 个文件）：每个存根的 body 只有 `"Agent methodology loaded via MCP tool get_agent"`，没有任何说明 MCP 不可用时的行为。根本原因：作者假设 MCP 永远可用，没有考虑离线场景或 MCP 启动失败。自查：我的仓库中是否有类似的隐性前置依赖而没有降级路径？→ 是的，bureau 的 scribe 技能依赖 `git` 可用，没有 fallback 文档。

2. **Output Contract 模糊词扩散**：共享 Output Contract 模板中的 `necessary` 一词导致使用该模板的所有 agent 都被扣 -2。根本原因：共享模板是双刃剑——既传播标准化，也传播缺陷。作者修模板的收益是 N 倍放大，但修错了也是 N 倍放大的代价。自查：我有没有在 gstack 的 SKILL.md 中用了类似「robust」、「comprehensive」这样的套话？→ **有**，gstack 的 132 个模糊词命中中 comprehensive 出现 75 次。

3. **75 个 agent 无 model 声明**：src/、claude/src/、plugins/ 三层的 agent 文件全部没有 `model:` 字段。根本原因：generate.js 的模板里没有 model 字段，所有生成的 agent 都继承了这个遗漏。自查：我的 gstack 的 59 个 SKILL.md 全部缺少 model 声明——同样的问题，同样来自无 model 的默认模板。

### 3.4 当时的优化机会

1. 将 Output Contract 模板中的 `necessary` 替换为 `unexpected`，一次修改消除 ~75 个文件的 -2 扣分（影响：约 +1.5 分整体提升）。

2. 在 generate.js 模板里加 `model: claude-sonnet-5` 默认值，一次生成修复 75 个 `未声明 model` 问题（影响：每文件 +5 分）。

3. 为 claude/agents/ 存根层补充 `## 降级说明` 节和更丰富的 body 示例（哪怕是指向 claude/src/agents/ 的交叉引用），消除 -15（无示例）+ -10（无输出格式）扣分，将存根得分从 75 提升到至少 90。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| prepare 自动运行 install-git-hooks.js（HIGH 安全） | `grep "prepare" package.json` | **已修复**：package.json scripts 中已无 prepare 字段 | 作者有安全意识，在收到私下报告或自查后主动移除了自动触发 |
| 75 个 agent 无 model 声明 | `grep -rL "model:" claude/src/agents/` | **仍存在**：claude/src/agents/ 39 个文件均无 model 字段 | 这是遗留问题，未随安全修复一起处理 |
| Output Contract 模糊词扩散 | `grep -rn "necessary" src/agents/ --include="*.md"` | **仍存在**：Output Contract 模板未修改 | 仅安全问题被优先处理，质量问题被搁置 |

### 4.2 架构演进

从 audit 快照到当前 HEAD，主要变化：
- **prepare 脚本移除**：`"prepare": "node scripts/install-git-hooks.js"` 已从 package.json 中删除，改为用户显式调用 `npm run install-hooks`。这是安全门的关键修复。
- **agent 数量扩张**：每层从 audit 时约 25 个存根 + 75 个完整版，到现在每层各 39 个，四层 × 39 = 约 156 个 agent。新增了 integration-engineer、compliance-reviewer、copywriter 等更多角色。
- **文档体系增强**：新增 ARCHITECTURE.md、QWEN.md、GEMINI.md 等文档，覆盖各运行时的配置说明。

这说明作者在安全问题暴露后做了响应式修复，同时持续在横向扩张角色库，但没有做纵向的质量提升（模糊词、model 声明、存根补全）。

### 4.3 新增的可学习模式

**多运行时独立文档**：当前 HEAD 新增了 QWEN.md 和 GEMINI.md，每个文件专门描述对应运行时的配置和使用方法。这比把所有运行时信息混在 README.md 里要清晰得多——不同用户只需阅读自己用的那个运行时的文档。这个「分运行时文档」模式值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责 SKILL.md**：我的 gstack/bureau/graphify 中的每个 SKILL.md 都聚焦在一个功能上（setup-deploy、recall、scribe 等），没有试图把多个功能塞进一个文件。
2. **无 MCP 单点故障**：我的仓库目前没有「agent body 全靠 MCP 注入」的设计，每个 SKILL.md 都是自包含的——降级路径天然存在。
3. **有效 frontmatter**：我的 SKILL.md 文件的 name 和 description 字段都有值，不会因缺少 frontmatter 而无法注册。

### 5.2 挑战 / 验证

**挑战**：这次案例打破了我对「共享模板」的正面假设。我之前觉得「统一 Output Contract 模板 = 好的标准化」，但 josstei 的案例说明：共享模板是模糊词的传播放大器。模板里一个 `necessary` 传染 75 个文件，每文件 -2，总损失 150 分（分摊到每个文件是 -2，但对整个仓库是 -1.5 的总体分拉低）。

**验证**：gstack 的 132 个模糊词命中（comprehensive × 75，robust × 59）证实了这个风险在我自己的仓库里已经存在。不是假设，是现实。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 gstack 的模糊量词（重点：comprehensive、robust、appropriate）
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|necessary|proper)\b' \
  /tmp/my-repos/MarkQWu-gstack/ --include="*.md" | grep -v "README\|CHANGELOG" | sort | head -20
```
命中后怎么办：找出出现最多的词（目前是 `comprehensive` ×75），从各 SKILL.md 中逐一替换为具体标准（如「覆盖 80% 的边界情况」「每一步有确认步骤」）。

```bash
# 检查 gstack 有多少 SKILL.md 缺少 model 声明
grep -rL "^model:" /tmp/my-repos/MarkQWu-gstack/ --include="SKILL.md"
```
命中后怎么办：在每个缺失 `model:` 的 SKILL.md 的 frontmatter 中加上 `model: claude-sonnet-5`。

```bash
# 检查 bureau/graphify 是否有类似的模糊词扩散
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|necessary|proper)\b' \
  /tmp/my-repos/MarkQWu-bureau/ /tmp/my-repos/MarkQWu-graphify/ --include="*.md"
```
命中后怎么办：对 bureau 的 7 个 SKILL.md 逐一审查，替换模糊词为可验证标准。

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 的 59 个 SKILL.md 批量加上 `model: claude-sonnet-5` 声明
   - **为何可行**：gstack 已有固定的 SKILL.md 文件列表，可以用一行 sed 命令批量处理 frontmatter
   - **第一步**：`find /path/to/gstack -name "SKILL.md" | xargs -I{} sed -i '/^---$/a model: claude-sonnet-5' {}` — 预计 10 分钟完成，验证后提交

2. **想法**：参照 josstei 的分运行时文档模式，给 graphify 加一个 `CLAUDE.md` 和 `CODEX.md`，分别描述在两个运行时下的最优配置
   - **为何可行**：graphify 已支持多运行时（Claude Code, Codex, OpenCode, Cursor, Gemini CLI），但配置说明全混在 README.md 里
   - **第一步**：从 README.md 中提取 Claude Code 专属配置段，写成 `CLAUDE.md`（约 30 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（最近刷新：2026-07-20）

### 8.1 目的对齐度

- **josstei/maestro-orchestrate 的核心目的**：跨 AI 运行时统一 agent 角色库，一份定义在 Gemini / Claude / Codex / Qwen 上复用

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为多角色 AI 工具套件；都用 SKILL.md 定义各专项能力；都面向工程师工作流 | gstack 是单运行时（Claude Code 为主）；josstei 是 4 运行时并行；规模 gstack 59 vs josstei 156 | 高 |
| MarkQWu/graphify | 中 | 同样声明支持多 AI 运行时（Claude Code, Codex, Gemini CLI 等） | graphify 是单一技能（知识图谱），josstei 是多角色团队；graphify 无适配器架构 | 中 |
| MarkQWu/bureau | 低 | 同为 Claude Code 插件 | bureau 是知识管理工具，josstei 是角色编排工具；目标不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Output Contract 模糊词扩散（appropriate, necessary） | `grep -rn "comprehensive\|robust\|appropriate" /tmp/my-repos/MarkQWu-gstack/ --include="*.md"` | **gstack 命中 132 次**（comprehensive ×75, robust ×59, appropriate ×33） | 高 |
| agent/skill 缺少 model 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-gstack/ --include="SKILL.md"` | **gstack 59/59 SKILL.md 全部缺少 model 声明** | 中 |
| 存根层无降级路径文档 | N/A（gstack 无存根架构） | 不适用 | N/A |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的全部 SKILL.md → 在 frontmatter 加 `model: claude-sonnet-5`（每文件 30 秒，59 个文件 ~30 分钟或写脚本 5 分钟）
- `MarkQWu/gstack` 的 `comprehensive` ×75 → 优先检查 `cso/SKILL.md`（已知包含 843 行，comprehensive 密度最高），将「全面」替换为具体的覆盖标准

### 8.3 别人的更优方案

1. **领域**：生成式工具链（generate.js 代码生成 agent 文件）
   - **josstei 做法**：用 `scripts/generate.js` 从模板批量生成所有 agent 的 MD 文件，确保 frontmatter 格式 100% 一致（原文件路径：`scripts/generate.js`）
   - **我的项目现状**：gstack 的 59 个 SKILL.md 全部手写，格式有微小差异（有的用 `---` 有的没有结尾分隔符）
   - **如何借鉴**：写一个 `scripts/generate-skill.js` 脚本，接受 name/description/model 参数生成标准 SKILL.md 骨架；将现有文件用脚本重新格式化

2. **领域**：分运行时文档（QWEN.md、GEMINI.md、CLAUDE.md）
   - **josstei 做法**：每个支持的 AI 运行时有独立的配置文档（当前 HEAD 新增 QWEN.md, GEMINI.md）
   - **我的项目现状**：graphify 声称支持 6 个 AI 运行时，但配置说明全部混在 README.md 里
   - **如何借鉴**：从 graphify/README.md 中提取 Claude Code 专属部分写成 `CLAUDE.md`，Codex 部分写成 `CODEX.md`（各约 30 分钟）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skill 自包含性（无 MCP 单点故障）
  - **我的做法**：bureau/skills/ 的每个 SKILL.md 都包含完整的方法论正文，不依赖 MCP 注入；skill 可以在 MCP 不可用时独立运行
  - **josstei 的做法**（弱在哪）：claude/agents/ 的 25 个存根在 MCP 不可用时完全降级为空壳，无任何内容
  - **意义**：如果有人审查我的 bureau，「自包含 skill」会是一个明确的亮点；若考虑给 josstei 提 PR，可以建议在存根 body 里加 `## 静态降级内容` 节

---

## 八、术语表

### <a name="hooks"></a>hooks
> Claude Code 的 hook 系统允许开发者在特定事件发生时（如工具调用前后、session 开始/结束）自动运行一段脚本。josstei 用 PostToolUse hook 来拦截 agent 调用，根据当前运行时（Claude / Gemini / Qwen）选择对应的适配器逻辑。
> **类比**：就像浏览器的 JavaScript 事件监听器——某件事发生时，自动触发你注册的代码。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 定义的标准协议，允许外部程序（MCP 服务器）把工具和数据暴露给 Claude。josstei 的 `maestro-server.js` 实现了一个 MCP 服务器，暴露 `get_agent` 工具——Claude 调用这个工具就能在运行时拿到 agent 的完整方法论，而不需要把内容写死在 SKILL.md 里。
> **对比**：传统方案是把所有内容写进文件；MCP 方案是文件只放引用，内容按需从服务器拉取。

### <a name="output-contract"></a>Output Contract（输出契约）
> josstei 在所有 agent 文件末尾放置的统一模板节，定义每个 agent 交付时必须包含的内容格式（如「执行摘要」「实现细节」「后续步骤」）。好处是标准化，坏处是模板里的措辞（如 `necessary`）会被复制到所有使用者。
> **类比**：就像公司的 Word 文档模板——所有人都用同一个底稿，如果底稿里有错字，所有人的文档都有那个错字。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（`name`、`description`、`model` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才知道如何注册和调用这个 skill。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 是 Claude Code 插件的 manifest，`gemini-extension.json` 是 Gemini CLI 插件的 manifest。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。
