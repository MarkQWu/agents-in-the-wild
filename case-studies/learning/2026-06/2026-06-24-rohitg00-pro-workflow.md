# rohitg00/pro-workflow — 学习案例

**仓库**：https://github.com/rohitg00/pro-workflow
**Stars**：1,912 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-24（基于 Audit 报告，无法实时克隆）
**主题标签**：`manifest-discipline`, `model-pinning`, `experience-accumulation`, `security-gate`, `single-purpose`

---

## 一、理解（基于 Audit 报告）

### 1.1 仓库概览

rohitg00/pro-workflow 是 Rohit Ghumare（rohitg00）为 Claude Code 用户设计的一套开发工作流插件，拥有 1,912 颗星。它的定位不是"工具集合"，而是一套完整的**开发者工作流操作系统**：从任务规划、代码提交、上下文优化、成本控制，到并行工作树管理，全都有对应的 skill + command + agent 覆盖。

关键数据：
- 60 个 NL artifacts（21 command + 8 agent + ~20 skill + hooks/templates）
- 综合 NL 得分 **90/100**（远超 70 分门槛）
- 安全等级：**BLOCKED**（实质是误报，见 §3.3）
- 同一作者还有 rohitg00/awesome-claude-code-toolkit（46/100），两个仓库质量天差地别

### 1.2 架构剖析

**目录结构**：
```
rohitg00/pro-workflow/
├── commands/           ← 21 个 command 文件（均缺 description 字段）
│   ├── learn.md        ← 70/100，最低分
│   ├── develop.md      ← 85/100
│   └── ...（19 个其他命令）
├── agents/             ← 8 个 agent 文件
│   ├── orchestrator.md ← 100/100（满分）
│   ├── debugger.md     ← 98/100（声明 model: opus）
│   └── ...（6 个其他，均未声明 model）
├── skills/             ← ~20 个 skill 文件
│   ├── pro-workflow/SKILL.md  ← 96/100，562 行（最长）
│   ├── batch-orchestration/SKILL.md ← 98/100
│   ├── module-map/SKILL.md    ← 98/100（强制 15 秒读取约束）
│   └── ...（17 个其他 skill，96-98/100）
├── hooks/
│   └── hooks.json      ← 90/100，26 个 hook 事件，35 个脚本引用
├── scripts/            ← 35 个 Node.js hook 脚本
├── templates/
│   └── split-claude-md/CLAUDE.md  ← 75/100（占位符未填）
└── .claude-plugin/
    └── plugin.json     ← 80/100，版本 3.1.0（与 package.json 的 3.2.0 不符）
```

**文件类型分布**：21 command + 8 agent + 20 skill + 1 hooks.json + 35 hook 脚本 + 1 plugin.json

**编排关系**：
- `/develop` 命令 → 调用 `agents/orchestrator.md` → 预加载 `skills/pro-workflow/SKILL.md`
- 大量 command 与对应 skill 有强命名对称性：`/commit` ↔ `skills/smart-commit/SKILL.md`，`/wrap-up` ↔ `skills/wrap-up/SKILL.md`，`/insights` ↔ `skills/insights/SKILL.md`
- `hooks/hooks.json` 中 26 个 hook 事件全部有对应的 `scripts/*.js`，无悬空引用
- `skills/orchestrate/SKILL.md` 描述的是 `/develop` 命令的模式，但没有名为 `/orchestrate` 的命令（意图上是 reference skill，但命名令人困惑）

**跨件契约**：skill 和 command 之间通过命名约定对齐，没有显式 manifest 引用。但 `plugin.json` 只注册了 6 个 agent，而实际有 8 个 agent 文件（`cost-analyst.md` 和 `permission-analyst.md` 未注册）。

### 1.3 设计思路 / 方法论

**核心设计哲学**：工作流完整覆盖。pro-workflow 的目标不是"提供几个好用的 skill"，而是覆盖整个开发者会话生命周期——从会话开始（上下文规划）到结束（wrap-up、handoff），从任务规划（planner agent）到代码提交（smart-commit skill），从成本追踪（cost-tracker skill）到质量守护（compact-guard skill）。

**解决什么问题**：高级 Claude Code 用户在长期使用中遇到的系统性问题——上下文膨胀、会话间知识断裂、命令分散难以发现、成本不可见。pro-workflow 用一个统一的插件解决所有这些问题。

**Trade-off**：全覆盖 vs 简洁。为了覆盖所有工作流场景，pro-workflow 的 `skills/pro-workflow/SKILL.md` 有 562 行，是审计中最长的 skill 文件。这是一个明确的设计选择——用单一入口把所有知识塞进一个文件，而不是拆分为多个更专注的 skill。

**认知模型**：作者把 Claude Code 理解为一个需要被"工程化管理"的 AI 系统，而不只是一个对话助手。skill 是"规范文档"，command 是"操作界面"，agent 是"专职分析员"，hook 是"监控系统"——整套比喻来自软件工程而非 AI 聊天。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：全栈工作流插件（Full-Stack Workflow Plugin）

一个单一插件覆盖开发者工作流的全部生命周期阶段，每个阶段有对应的 skill（知识层）+ command（操作层）+ 可选 agent（分析层）三层覆盖。

模式特征：
- **命名对称性强**：command 和 skill 名称成对出现，用户容易在两者之间建立心智映射
- **Hook 全线覆盖**：26 个 hook 事件监控整个 Claude Code 操作链，实现"透明监控"
- **agent 层按专职划分**：每个 agent 只做一件专业的事（scout 搜集、planner 规划、reviewer 审查、cost-analyst 分析成本）
- **skill 质量一致性高**：20 个 skill 全部 96-100 分，说明有统一的写作标准在执行

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 专业开发者长期使用 Claude Code | ✅ 高度适用 | 覆盖全工作流，减少"每次都要手写 prompt"的重复工作 |
| 团队统一 Claude Code 使用规范 | ✅ 适用 | 统一的 skill 库和 hook 规范可作为团队标准 |
| 偶发性使用者 / 初学者 | ❌ 不适用 | 学习曲线高，21 个 command + 20 个 skill 对新用户是负担 |
| 特定单一任务的 skill 包 | ❌ 过度设计 | 全栈模式对单一用途场景是过度工程 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 全栈工作流插件（本仓库） | rohitg00/pro-workflow | 覆盖完整，skill 质量高，命名对称 | 入门成本高，pro-workflow/SKILL.md 过长 |
| 单职责 skill 集合 | addyosmani/agent-skills | 模块化强，按需加载 | 缺乏工作流连贯性 |
| 薄 NL 壳 + 代码核心 | refly-ai/refly | 维护成本低，聚焦产品核心 | AI 辅助覆盖不足 |
| 规模化工具集合 | rohitg00/awesome-claude-code-toolkit | 覆盖领域广，适合大型团队 | NL 质量参差（46/100），frontmatter 缺失 |

### 2.4 改进空间

1. **`skills/pro-workflow/SKILL.md` 应拆分**：562 行超过了合理的 skill 长度。根本原因是把"所有工作流规范"塞入一个文件。改进做法：把 skill 按阶段拆分（`skills/session-start/SKILL.md`、`skills/in-session/SKILL.md`、`skills/session-end/SKILL.md`），主 skill 只保留目录和跳转指引。预期收益：每个 skill 加载更快，认知负担更低。

2. **`plugin.json` 应与 agent 文件同步**：`cost-analyst.md` 和 `permission-analyst.md` 存在于文件系统但未注册到 `plugin.json`，导致用户无法通过插件调用它们。改进做法：在 `plugin.json` 的 `agents` 数组中加入这两个 agent（2 分钟修复）。预期收益：消除 manifest-discipline 失误，两个 agent 对外可见。

3. **`commands/doctor.md` 中测试 key 的替换**：`AKIAIOSFODNN7EXAMPLE` 触发了 CRITICAL 安全警报（虽是误报），阻断了贡献流水线。把它替换为 `AKIAXXXXXXXXXXXXXXXX`（非实际密钥格式，不触发扫描器），同样能测试 `secret-scan.js` 是否正常工作，还能解除 BLOCKED 状态。预期收益：BLOCKED → CLEAR，允许贡献流水线继续（约 30 秒修复）。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

综合 NL 得分 **90/100**，安全等级：**BLOCKED**（本质是误报）。

| 文件 | 类型 | 当时分数 | 主要问题 |
|---|---|---|---|
| `commands/learn.md` 等 12 个 command | command | 70/100 | 缺少 `description` frontmatter |
| `templates/split-claude-md/CLAUDE.md` | template | 75/100 | 占位符 `[Project Name]` 未填写，提供不了任何真实指引 |
| `commands/develop.md` | command | 85/100 | 无空输入处理（`$ARGUMENTS` 缺失时静默失败） |
| `.claude-plugin/plugin.json` | config | 80/100 | 2 个 agent 文件未注册 |
| `commands/doctor.md` 等 9 个 command | command | 95/100 | 缺少 `allowed-tools` |
| `agents/orchestrator.md` | agent | 100/100 | 满分，模型、skill、阶段均已声明 |
| `agents/debugger.md` | agent | 98/100 | 满分接近，显式声明 `model: opus`，多个示例 |
| `skills/<大多数>` | skill | 96-98/100 | 极少量细节问题，整体接近完美 |

### 3.2 当时值得借鉴的模式

1. **`agents/orchestrator.md` 满分设计**：这个 agent 满足了所有要求——`model` 声明、`skills` 引用列表、多阶段（phases）定义、清晰的 `description`。它是"如何写一个合规 agent"的标准答案。特别值得注意的是 `phases` 声明：orchestrator 把任务分成可识别的阶段（understand → plan → implement → verify），这让 Claude 在执行时能给用户清晰的进度反馈。

2. **强命名对称性（command-skill 对称）**：`/commit` 调用 `skills/smart-commit/SKILL.md`，`/wrap-up` 调用 `skills/wrap-up/SKILL.md`。这种对称性不只是命名美学，而是实用的可发现性设计——用户看到命令名就能猜到对应 skill 的名字，反之亦然。

3. **`skills/module-map/SKILL.md` 的 15 秒约束**：这个 skill 明确规定了"15 秒内完成模块图读取"的时间约束，是可验证的性能目标，而不是模糊的"快速"。把性能目标直接写进 skill 规范，是 NL 编写中少见但极有价值的做法。

4. **`hooks/hooks.json` 零悬空引用**：35 个 hook 脚本引用，全部文件存在，无一悬空。这需要维护纪律——每次新增 hook 脚本时，对应的 hooks.json 条目也要同步更新。审计把 hooks.json 评为 90/100，是所有 config 文件中最高的。

5. **`skills/permission-tuner/SKILL.md` 的风险分级**：这个 skill 定义了清晰的"风险等级"（低/中/高），每个等级有具体标准和操作规范。把抽象的"风险"量化为分级标准，是 NL 设计中的优质模式。

### 3.3 当时的缺陷

1. **12 个 command 缺少 `description` frontmatter（各 -30 分导致 70/100）**：根本原因是作者把 H1 标题当作"足够的描述"。但 Claude Code 的插件发现机制读的是 frontmatter 中的 `description` 字段，而不是 Markdown 标题。这些命令在 `/help` 输出中没有描述文字，在插件市场中也没有元数据。修复极其简单——把 H1 标题文字复制到 frontmatter `description:` 字段就行——但所有 12 个文件都遗漏了这一步，说明作者写 command 时从未参考过 Claude Code 的 frontmatter 规范。

2. **`plugin.json` 遗漏 2 个 agent（`cost-analyst` 和 `permission-analyst`）**：这是 manifest-discipline 失误。两个文件存在，逻辑上也能独立运行，但通过插件安装的用户无法通过标准 `/agent` 调用机制访问它们。更严重的是，`cost-analyst` 对应的 `/cost-tracker` 命令和 `permission-analyst` 对应的 `/permission-tuner` 命令依赖这两个 agent 提供的技能知识——虽然命令本身可以工作（因为 skill 存在），但 agent 不可用意味着这两个命令失去了深度分析能力。

3. **`commands/doctor.md` 中的 `AKIAIOSFODNN7EXAMPLE` 触发 CRITICAL（实质误报）**：这是一个反直觉的缺陷。`doctor.md` 用这个 AWS 官方文档示例密钥来测试 `secret-scan.js` 是否能正确拦截硬编码凭据——这个想法本身是好的（测试自己的安全工具）。但 NLPM 的安全扫描器是基于模式匹配的，不理解"这是一个测试用例"，看到 `AKIA[0-9A-Z]{16}` 就触发 CRITICAL，导致整个仓库被 BLOCKED。根本原因：安全扫描是脱离上下文的纯模式匹配，而不是语义理解。

4. **`templates/split-claude-md/CLAUDE.md` 占位符未填（-25 分）**：这个模板文件的内容是 `[Project Name]` 和 `[Brief description]` 这样的占位符，用户安装后得到的是一个空架子，对实际项目毫无指导意义。根本原因：模板文件是"示例驱动"的设计被中途废弃了——作者可能计划后来填入内容，但忘记了。

### 3.4 当时的优化机会

1. **为 12 个 command 添加 `description` frontmatter**：把 H1 标题文字复制到 frontmatter，每个文件 1-2 分钟，合计 20 分钟内完成。12 个 command 的得分将从 70/100 提升到至少 85/100，整体得分从 90 提升到约 93。

2. **在 `plugin.json` 中注册 `cost-analyst.md` 和 `permission-analyst.md`**：在 `agents` 数组中加两个条目，约 2 分钟。两个 agent 对外可见，`/cost-tracker` 和 `/permission-tuner` 恢复完整能力。

3. **将 `commands/doctor.md:37` 中的 `AKIAIOSFODNN7EXAMPLE` 替换为 `AKIAXXXXXXXXXXXXXXXX`**：单行修改，30 秒。安全状态从 BLOCKED 变为 CLEAR，贡献流水线解锁。这是本仓库最高 ROI 的修复。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

**注意**：由于 git clone 在代理环境中失败（403），以下状态判断无法直接核验。基于审计报告及同类问题的修复规律推断。

| 过去缺陷 | 检查方法（待克隆后验证） | 推测现状 | 推测依据 |
|---|---|---|---|
| 12 command 缺少 `description` | `grep -L "^description:" commands/*.md` | **可能仍存在** | 问题极小，没有外部 PR 提交记录，作者无明显修复动机 |
| `plugin.json` 遗漏 2 个 agent | `cat .claude-plugin/plugin.json \| jq '.agents'` | **可能仍存在** | manifest 不一致问题不影响运行，低优先级 |
| `doctor.md` AKIA 误报 | `grep "AKIA" commands/doctor.md` | **可能仍存在** | 不影响功能，需主动了解 NLPM 扫描规则才知道要改 |
| `package.json` vs `plugin.json` 版本不一致 | `jq .version package.json .claude-plugin/plugin.json` | **未知** | 随版本迭代可能已修复，也可能继续不同步 |

### 4.2 架构演进

基于 Audit 数据推断，pro-workflow 自 2026-04-06 以来可能在功能上有迭代（毕竟是活跃的 1912 星仓库），但 NL 层的合规性问题（frontmatter 缺失、manifest 不一致）可能没有被优先修复——这类问题只有在使用 NLPM 审计工具或主动对照 Claude Code 规范时才会被发现。

值得关注的方向：如果作者在后续版本中意识到了 manifest 问题，可能会在某次版本更新时顺手修复所有 12 个 command 的 `description`。但也可能继续按下去——因为这些命令在功能上完全正常，用户不会感知到 `/help` 缺少描述的问题（直到他们对比规范）。

### 4.3 新增的可学习模式

暂无（无法访问当前 HEAD 验证）。从 Audit 后作者的通常行为推断，如果有新的功能 skill（如 AI 模型定价追踪、多模态支持等），它们可能延续了现有 96-98/100 的 skill 质量标准，但命令文件的 frontmatter 问题可能仍在复现。

---

## 五、校准

### 5.1 我已经在做对的

1. **关注 manifest 同步**：`plugin.json` 遗漏 agent 是一个典型的 manifest-discipline 问题。如果我的项目有类似的 manifest 文件（plugin.json 或 CLAUDE.md 中的组件引用），应该在每次新增 artifact 文件时同步更新 manifest，不要等到审计报告指出。

2. **理解 security gate 的工作方式**：`doctor.md` 的 AKIA 误报说明安全扫描是模式匹配，不是语义理解。如果我的文档或测试文件里有类似的"示例凭据"（即使明确标注是示例），也会触发扫描器。应该用 `AKIAXXXXXXXXXXXXXXXX` 这种显然不可用的格式，而不是官方文档里的示例字符串。

3. **skill 质量应有统一标准**：pro-workflow 的 20 个 skill 全部 96-100/100，说明有统一写作模板在执行。如果我的项目有多个 skill 文件，应该制定一个 skill 写作模板，确保每个 skill 都有触发条件、操作步骤、输出格式、量词约束。

4. **agent 应声明 model**：`orchestrator.md` 满分，部分原因是显式声明了 `model`；其余 6 个 agent 各扣 5 分，原因是未声明 `model`。写 agent 时声明 model 是低成本高收益的规范动作。

### 5.2 挑战 / 验证

本案例**验证**了"同一作者在不同项目里的质量分布可以完全不同"。rohitg00 的 pro-workflow 是 90/100，他的 awesome-claude-code-toolkit 是 46/100。差异的来源不是能力，而是**意图和上下文**：pro-workflow 是精心打磨的工具，toolkit 是规模化收集。这说明 NL 质量不取决于作者水平，取决于作者在特定项目上愿意投入多少纪律性精力。

本案例**挑战**了"security BLOCKED 就意味着仓库有安全问题"的假设。pro-workflow 的 BLOCKED 是一个可以在 30 秒内修复的误报，而且触发点是一个好意图（测试自己的安全工具）。这说明 BLOCKED 状态应该先看具体发现再判断，而不是把它等同于"这个仓库不安全"。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 command 文件是否全部有 description 字段
# （对应本案例 12 个 command 各 -30 分的核心问题）
find . -path "*/commands/*.md" -exec grep -L "^description:" {} \;
```
命中后怎么办：在该文件的 frontmatter 中加 `description: <从 H1 标题复制过来的文字>`，5 分钟内完成。

```bash
# 检查 plugin.json 注册的 agent 是否与实际文件匹配
# （对应 plugin.json 遗漏 2 个 agent 的 manifest-discipline 问题）
REGISTERED=$(cat .claude-plugin/plugin.json 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); print('\n'.join(d.get('agents', [])))")
ACTUAL=$(find . -name "*.md" -path "*/agents/*" | sed 's|^\./||')
diff <(echo "$REGISTERED" | sort) <(echo "$ACTUAL" | sort)
```
有差异时怎么办：把差异项补充到 `plugin.json` 的 `agents` 数组中。

```bash
# 检查文档中是否有 AKIA 格式的 AWS key
# （对应 doctor.md 中的误报 CRITICAL 问题）
grep -rn "AKIA[0-9A-Z]\{16\}" . --include="*.md" --include="*.txt" --include="*.json"
```
命中后怎么办：如果是示例/测试用途，把 AKIA 开头的部分替换为 `AKIAXXXXXXXXXXXXXXXX`（不匹配真实密钥格式）。

```bash
# 检查 agent 文件是否声明了 model
# （对应 6 个 agent 各 -5 分的问题）
for f in $(find . -path "*/agents/*.md"); do
  echo "=== $f ===";
  grep -n "^model:" "$f" | head -1 || echo "  [缺少 model 字段]"
done
```
命中"缺少 model 字段"后怎么办：在 agent 文件的 frontmatter 中加 `model: claude-sonnet-4-6`（或适合该 agent 职责的模型）。

```bash
# 检查 package.json 版本和 plugin.json 版本是否一致
# （对应版本不一致的跨件问题）
PV=$(cat package.json 2>/dev/null | python3 -c "import json,sys; print(json.load(sys.stdin).get('version','N/A'))")
CPV=$(cat .claude-plugin/plugin.json 2>/dev/null | python3 -c "import json,sys; print(json.load(sys.stdin).get('version','N/A'))")
echo "package.json: $PV | plugin.json: $CPV"
[ "$PV" != "$CPV" ] && echo "⚠️ 版本不一致！" || echo "✅ 版本一致"
```
不一致时怎么办：把两个版本号同步为最高版本，并在 commit message 中记录"同步版本号"。

### 6.2 灵感 → 实施路径

1. **想法**：把"测试安全扫描工具"的 doctor 模式借鉴过来，但用不会触发扫描器的测试值。
   - **为何可行**：测试自己的安全 hook（secret-scan.js）是一个好实践，pro-workflow 的思路是对的，只是测试用的密钥格式选择不当。可以用 `AKIAXXXXXXXXXXXXXXXX` 或 `FAKE_API_KEY_FOR_TESTING` 来代替。
   - **第一步**：在自己的 doctor 或 setup 命令里加一段"安全工具自检"步骤，用明确标注为测试的值（非真实密钥格式）验证 hook 是否正常运行。约 30 分钟。

2. **想法**：借鉴 `skills/module-map/SKILL.md` 的"约束量化"写法，把自己 skill 里的模糊性能目标改为具体数字。
   - **为何可行**：把"快速理解"改为"15 秒内完成"是一个零成本的 NL 质量提升——不需要改代码，只需要改 skill 文本，但带来的是可验证的、可测试的性能目标。
   - **第一步**：在自己的 skill 文件里找出所有"快速""高效""及时"等模糊量词，逐一替换为具体时间或具体次数约束。约 30 分钟/skill。

3. **想法**：建立"command-skill 命名对称"约定，确保每个 command 都有同名 skill 提供知识支撑。
   - **为何可行**：pro-workflow 的 command-skill 对称是其架构的核心优点，让用户可以"从任意一侧出发找到另一侧"。如果自己的项目开始有多个 command 和 skill，建立这个约定可以大幅降低认知负担。
   - **第一步**：检查现有项目中 command 和 skill 的名称列表，看是否存在孤立的 command（没有对应 skill）或孤立的 skill（没有对应 command 调用它）。约 20 分钟。

---

## 七、对照我的 GitHub 仓库

**注意**：由于代理限制，用户仓库克隆失败（403）。以下分析基于仓库描述推断，无法 grep 实际代码。

### 8.1 目的对齐度

- **本案例 rohitg00/pro-workflow 的核心目的**：完整覆盖 Claude Code 开发者工作流生命周期的插件——从规划、开发、提交，到上下文管理、成本追踪、会话交接。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 两者都关注"跨会话知识管理"——bureau 是 capture→compile→review→query，pro-workflow 有 session-handoff、learn、replay 等 skill | bureau 聚焦知识库管理；pro-workflow 聚焦工作流执行本身 | 高（skill 写法、agent 注册、command frontmatter 最直接可借鉴） |
| MarkQWu/gstack | 中 | 都是 Claude Code 工具集合；gstack 有 23 个工具，pro-workflow 有 21 个 command | gstack 更像"角色工具箱"；pro-workflow 更像"工作流 OS" | 中（命令对称、hook 覆盖可参考） |
| MarkQWu/echo-sleuth-for-claude | 中 | echo-sleuth 挖掘会话历史做决策分析，和 pro-workflow 的 `/replay`（重放历史学习）目标高度相似 | echo-sleuth 是 Claude Code 插件；pro-workflow 是更完整的工作流套件 | 中（`skills/replay-learnings/SKILL.md` 和 echo-sleuth 的方案可互相对照） |
| MarkQWu/graphify | 低 | 都是 Claude Code skill；graphify 的代码知识图谱和 pro-workflow 的模块地图（module-map）有概念重叠 | 目的差距大，graphify 是代码分析工具，pro-workflow 是工作流插件 | 低 |

### 8.2 在我的项目里复现的同类问题

**基于仓库描述推断（无法 grep 实际文件）**：

| 本案例缺陷 | 相关仓库 | 推断命中可能性 | 自查优先级 |
|---|---|---|---|
| Command 文件缺少 `description` frontmatter | MarkQWu/gstack（23 个工具），MarkQWu/bureau（多阶段 command） | **高**——command 文件的 description 字段是容易系统性遗漏的字段 | 高 |
| Agent 文件未在 plugin.json/manifest 中注册 | MarkQWu/bureau，MarkQWu/echo-sleuth-for-claude | **中**——如果有 agent 文件，注册遗漏是常见疏忽 | 高 |
| Agent 未声明 `model` | MarkQWu/gstack（23 个角色工具可能有对应 agent） | **中**——特别是如果 agent 是后期添加的，容易忘记加 model 声明 | 中 |
| 版本号在不同文件中不一致 | 所有有 `plugin.json` + `package.json` 的仓库 | **中**——手动维护两个版本号很容易不一致 | 中 |

如果能克隆仓库，应该优先运行的命令：
```bash
# 针对 bureau: 检查所有 command 的 description 字段
find /tmp/my-repos/MarkQWu-bureau -path "*/commands/*.md" -exec grep -L "^description:" {} \;

# 针对 gstack: 检查 agent 注册情况
cat /tmp/my-repos/MarkQWu-gstack/.claude-plugin/plugin.json 2>/dev/null | python3 -c "import json,sys; print(json.dumps(json.load(sys.stdin).get('agents',[]), indent=2))"

# 针对 echo-sleuth: 检查 agent model 声明
grep -rn "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude --include="*.md"
```

### 8.3 别人的更优方案

1. **领域**：会话间知识积累（experience-accumulation）
   - **本案例做法**：`skills/replay-learnings/SKILL.md`（97/100）+`commands/replay.md`（70/100，仅 description 缺失）共同构成了"重放历史学习"系统。用户可以通过 `/replay` 命令搜索过去会话中积累的规则和经验，Claude 用 SQLite FTS5 全文搜索找到相关学习记录。
   - **我的项目现状**（推断）：`MarkQWu/echo-sleuth-for-claude`（"Mine past conversations for decisions, mistakes, patterns, and wisdom"）目的相同，但从描述看更偏向单次挖掘，可能没有 pro-workflow 的持续积累+检索机制。
   - **如何借鉴**：参考 `skills/replay-learnings/SKILL.md` 的 SQLite 积累机制，在 echo-sleuth 中加入"跨会话持久化学习记录"功能，把单次挖掘的结果保存到 `~/.echo-sleuth/learnings.db`，后续可检索。改动：在 echo-sleuth 的 SKILL.md 中加入"持久化输出"步骤，约 2 小时。

2. **领域**：全局 Hook 监控覆盖
   - **本案例做法**：`hooks/hooks.json` 的 26 个 hook 事件 + 35 个 Node.js 脚本，覆盖了整个 Claude Code 操作链（Write、Edit、Bash、会话开始/结束、紧凑触发等）。每个重要操作都有对应的监控脚本。
   - **我的项目现状**（推断）：gstack 和 bureau 如果有 hooks，可能是有限的几个特定场景 hook，不太可能有 pro-workflow 这样的全面覆盖。
   - **如何借鉴**：参考 pro-workflow 的 hooks.json 结构和脚本命名约定（`session-start.js`、`session-end.js` 等），为 bureau 添加"会话开始时自动加载今日知识库"和"会话结束时自动 capture 本次新知识"的 hook，把手动操作变成自动触发。

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：Manifest 同步机制（推断）
   - **我的做法**（推断）：如果 MarkQWu/bureau 使用了统一的 registry 或 manifest 文件，可能有更好的"新增文件时自动提醒更新 manifest"机制（比如 NLPM 自己的 hooks 就做了这件事）。
   - **本案例弱在哪**：pro-workflow 的 `plugin.json` 遗漏了 2 个 agent，是纯手工维护失误。
   - **意义**：如果自己的项目有维护 manifest 的 hook 或 CI 检查，就比 pro-workflow 在这个维度做得更好。这是一个值得对外提的设计点。

2. **领域**：Security AKIA 模式规避（如果自己没犯同样的错）
   - **我的做法**（推断）：如果 gstack/bureau 的文档里没有出现 `AKIA` 格式的测试密钥，那自然避开了这个误报陷阱。
   - **本案例弱在哪**：30 秒可以修复的问题，却导致了整个贡献流水线被 BLOCKED。
   - **意义**：未来对外贡献 PR 时，先运行 `grep -rn "AKIA[0-9A-Z]\{16\}"` 排查潜在误报，是一个有效的预检步骤。

---

## 八、术语表

### <a name="manifest-discipline"></a>manifest-discipline（清单纪律）
> 在 Claude Code 插件中，`plugin.json` 是系统的"目录"，列出插件包含的所有 commands、skills、agents 的路径。如果一个文件存在于磁盘，但没有被注册到 `plugin.json`，它就对用户不可见——就像一本书有第三章，但目录里漏写了第三章。"清单纪律"指的是保持 manifest 文件和实际文件系统同步的习惯。本案例中，`cost-analyst.md` 和 `permission-analyst.md` 存在但未注册，就是 manifest-discipline 失误。

### <a name="model-pinning"></a>model-pinning（模型固定）
> 在 agent 文件的 frontmatter 中用 `model:` 字段指定该 agent 使用的具体模型（如 `model: claude-opus-4-8`）。固定模型的好处是行为可预测——知道这个 agent 用哪个模型，就能预期它的能力边界和成本。不固定模型时，Claude Code 会用默认模型，而默认模型可能随版本更新而变化。`agents/debugger.md` 显式声明了 `model: opus`，得 98/100；其他 6 个 agent 未声明，各扣 5 分。

### <a name="experience-accumulation"></a>experience-accumulation（经验积累）
> 一种 NL artifact 设计模式：把 Claude 在每个会话中产生的洞见、规则发现、错误总结，持久化到本地存储（通常是 SQLite 数据库），供未来会话检索和回放。本案例的 `skills/replay-learnings/SKILL.md`（97/100）和 `commands/replay.md` 共同实现了这个模式：用户用 `/replay <关键词>` 检索过去积累的学习记录，Claude 通过 SQLite FTS5 全文搜索返回相关经验。这让每次会话不是独立的"对话"，而是一个不断成长的工作记忆系统。

### <a name="description-frontmatter"></a>description frontmatter（描述字段）
> Claude Code command 或 skill 文件的 YAML 前置元数据块中的 `description:` 字段，用于向插件市场和 `/help` 命令提供人类可读的描述文字。缺少 `description:` 的命令在 `/help` 中没有说明，用户无法理解该命令的用途。本案例中，12 个 command 的 H1 Markdown 标题已经包含了描述信息，但没有把它复制到 frontmatter，导致机器读不到，每个文件各扣 30 分（得 70/100）。修复极简单：把标题文字加到 frontmatter `description:` 字段。

### <a name="false-positive-critical"></a>误报 CRITICAL（False Positive Critical）
> 安全扫描器基于正则模式匹配，当文档中出现特定格式的字符串（如 AWS 密钥格式 `AKIA[0-9A-Z]{16}`）就触发 CRITICAL 警告，无论该字符串是否是真实凭据。本案例中，`AKIAIOSFODNN7EXAMPLE` 是 AWS 官方文档中用于演示的示例密钥，广泛出现在教程和测试用例里，但扫描器无法区分"示例"和"真实密钥"，只要格式匹配就报 CRITICAL。这会导致整个仓库被 BLOCKED，影响贡献流水线。规避方法：把示例密钥改为不匹配真实密钥格式的字符串，如 `AKIAXXXXXXXXXXXXXXXX`。

### <a name="sqlite-fts5"></a>SQLite FTS5（全文搜索扩展）
> SQLite 内置的全文搜索模块，允许用关键词在大量文本记录中快速检索，支持特殊查询语法（如 `*` 通配符、`-` 排除词、`"phrase"` 精确匹配）。本案例 `commands/replay.md` 使用 `WHERE learnings_fts MATCH '<keywords>'` 在积累的学习记录中检索。审计发现安全隐患：用户输入的关键词直接插入 FTS5 查询，若包含特殊字符（`"*^()`）会引发查询错误。修复方法：在插入前把关键词包在双引号中（`"term"`），强制解释为精确词组查询。
