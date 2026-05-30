# caliber-ai-org/ai-setup — 学习案例

**仓库**：https://github.com/caliber-ai-org/ai-setup
**Stars**：698 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-30（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `template-design`, `cross-reference`, `monorepo-vs-split`, `single-purpose`

**xiaolai 案例**：[../2026-04-25-caliber-ai-org-ai-setup.md](../2026-04-25-caliber-ai-org-ai-setup.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
caliber-ai-org/ai-setup 的核心价值主张一句话：`caliber sync` 一条命令，把 skill 文件同步到 Claude Code、Cursor、Codex（OpenAI Codex）三个平台——同一套知识，三种工具各拿各的格式。698 颗星表明"多平台 AI skill 同步"这个场景存在真实需求。

关键事实：
- 同一套 8 种 skill，分别部署到 `.claude/`、`.agents/`、`.cursor/` 三个目录，共 25 个 NL 工件
- [telemetry](#telemetry) 依赖（`posthog-node`）在 audit 后已加入 README 披露和 opt-out 机制
- 全部 25 个 NL 工件在 Audit 时加权平均得分 100/100——然后因为 4 个跨平台签名分歧，仓库被标记了 4 个 Bugs
- CLAUDE.md 100 分，8 类 skill 中 4 类在 audit 时存在平台间 TypeScript 接口分歧

### 1.2 架构剖析
```
caliber-ai-org/ai-setup/
├── src/                          # TypeScript 核心代码
│   ├── llm/                      # LLM provider 抽象层
│   │   ├── types.ts              # LLMProvider 接口（单一真相源）
│   │   └── index.ts              # provider 工厂
│   ├── writers/                  # 多平台 skill 文件生成器
│   │   └── index.ts              # writeSetup() 编排函数
│   ├── scoring/                  # 评分模块
│   │   └── index.ts              # check 函数注册点
│   └── cli.ts                    # CLI 命令注册（named import）
├── .claude/skills/               # Claude Code 适配（8种skill）
│   ├── llm-provider/SKILL.md     # 最详细版（7步骤，完整代码块）
│   ├── writers-pattern/SKILL.md  # 同步接口（最权威）
│   ├── scoring-checks/SKILL.md   # dir:string 签名
│   ├── adding-a-command/SKILL.md # named export
│   ├── save-learning/SKILL.md    # 4个平台一致，高质量
│   ├── find-skills/SKILL.md
│   ├── setup-caliber/SKILL.md
│   └── caliber-testing/SKILL.md
├── .agents/skills/               # Codex/Agents 适配（8种skill，已修复）
├── .cursor/skills/               # Cursor 适配（8种skill，已修复）
├── .claude/rules/                # 新增：确定性校验规则
│   └── scoring-patterns.md
├── .context/                     # 轻量 context 文件
│   ├── notes.md
│   └── todos.md
├── scripts/                      # 安装/演示脚本
├── CLAUDE.md                     # 100分，项目总上下文
├── AGENTS.md                     # 面向 agent 的元指令（audit后新增）
└── package.json                  # posthog-node（现已有 opt-out）
```

- **文件类型分布**：25 个 NL 工件（24 SKILL.md + 1 CLAUDE.md）/ 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：`.claude/` 版本作为权威源，`.agents/` 和 `.cursor/` 是生成/同步的副本
- **跨件契约**：每个 skill 的 `paths:` frontmatter 字段指向 `src/` 中对应的 TypeScript 文件

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一次定义，多平台分发」—— 不是为 Claude/Cursor/Codex 各写一套，而是从 caliber 的 TypeScript 源码中提炼 skill，然后分发到三个平台
- **解决什么问题**：开发者在不同时间用不同 AI 工具工作，skill 应该跟着人走而不是绑定工具
- **Trade-off**：多平台副本带来同步成本；audit 发现 4 个 skill 因各平台版本独立演化而出现接口分歧
- **认知模型**：把 skill 看作"TypeScript 接口的自然语言文档"——skill 里的代码示例必须和 `src/` 中的实际接口完全一致

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单一真相源 + 多平台派生」**

特征清单：
- 特征 1：`.claude/` 版本是权威源，其他平台从此派生（或应当派生）
- 特征 2：每个 skill 的 `paths:` frontmatter 字段指定了该 skill 覆盖的代码路径，建立 skill ↔ 代码的精确映射
- 特征 3：4 个"稳定 skill"（save-learning/find-skills/setup-caliber/caliber-testing）三平台完全一致；4 个"活跃接口 skill"（llm-provider 等）曾经漂移
- 特征 4：CLAUDE.md + 24 个 SKILL.md 共 25 件 NL 工件，NL 质量投入极高（平均 100 分）
- 特征 5：`paths:` 约束帮助 agent 理解"何时应该加载此 skill"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨 Claude/Cursor/Codex 多工具团队 | ✅ 高度适用 | 这正是 caliber 的设计靶点 |
| 接口频繁变化的活跃开发期 | ⚠️ 高风险 | 接口变动后必须同时更新三平台 skill，否则漂移 |
| 个人单工具项目 | ❌ 过度设计 | 三套副本成本远超收益 |
| 需要接口精确性的 SDK 文档 | ✅ 适用 | `paths:` 约束 + 接口示例的组合是良好文档范例 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单一真相源+多平台派生（本仓库）| caliber-ai-org/ai-setup | 统一用户体验，可自动同步 | 维护三套，漂移风险高 |
| 手动多副本（无生成器）| breaking-brake/cc-wf-studio | 无需构建工具 | 漂移已在 audit 中被发现（.github vs .roo） |
| 文档驱动生成 | antfu/skills | 大规模自动化 | 缺少对接口准确性的校验 |

### 2.4 改进空间
1. **当前问题**：`.agents/` 和 `.cursor/` 的 skill 文件目前靠人工维护同步，没有 CI 校验。**改进做法**：加一个 `pnpm run verify-skill-sync` 脚本，对比三平台的 skill 是否与 `src/llm/types.ts` 等 TypeScript 接口一致。**预期收益**：下次接口变更时不会被 audit 发现而是被 CI 拦截。
2. **当前问题**：4 个"稳定 skill"和 4 个"活跃接口 skill"没有分类标注，维护者难以判断哪些需要三平台同步。**改进做法**：在 skill frontmatter 加 `stability: stable | interface-bound`。**预期收益**：维护者更新接口时知道该同步哪些 skill。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-20 得分 **100/100**（加权平均 99.68 向上取整），安全状态 **CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/skills/scoring-checks/SKILL.md | 98 | 模糊量词 "appropriate category section" |
| .claude/skills/save-learning/SKILL.md | 98 | 模糊量词 "appropriate type prefix" |
| .agents/skills/save-learning/SKILL.md | 98 | 同上 |
| .cursor/skills/save-learning/SKILL.md | 98 | 同上 |
| 其余 21 个工件 | 100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **`paths:` frontmatter 约束** → 每个 skill 明确写出对应的 TypeScript 文件路径（如 `paths: - src/llm/**/*.ts`），让 agent 知道该 skill 适用于哪些文件操作。借鉴：我的 skill 可以加 `paths:` 字段明确管辖范围，避免越权触发。

2. **7步骤完整实施指南（llm-provider/SKILL.md）** → 不是"你应该实现 X 接口"，而是"Step 1 做什么 → Step 2 做什么 → Step N 做什么"，每步有代码块示例。借鉴：技术性 skill 的指令应该是 step-by-step 而非描述式。

3. **Critical 段分离** → skill 主体先写 `## Critical`（必须遵守的约束），再写 `## Instructions`（如何做的步骤）。两段职责清晰。借鉴：把红线约束和操作指南分层，agent 能快速找到禁止条款。

4. **稳定 skill 完全跨平台一致** → save-learning/find-skills/setup-caliber/caliber-testing 四个 skill 三平台完全一致，说明这类"行为约定类 skill"（而非接口类 skill）天然稳定。借鉴：行为约定类 skill 可以三平台共享一份。

5. **CLAUDE.md 100 分**：整个项目的总上下文文档质量极高，为所有 skill 提供了强有力的基础。

### 3.3 当时的缺陷
1. **4个 skill 跨平台接口分歧（核心 Bug）** → 同一个 `llm-provider` skill，`.claude/` 描述回调式 `stream(options, callbacks)`，`.agents/` 描述 `AsyncGenerator<string>`，`.cursor/` 描述 `AsyncIterable<StreamChunk>`。三种签名只有一种正确（`.claude/` 版本与 `src/llm/types.ts` 吻合）。根本原因：三平台 skill 分别独立演化，没有"谁是权威源"的机制约束。自查：**如果我将来要适配多平台，必须先定义权威源，再生成其他副本，而不是并行手写。**

2. **模糊量词"appropriate"（4处）** → `save-learning/SKILL.md` 里"appropriate type prefix"没有给出选择规则。根本原因：作者熟悉领域知识，省略了"何时选哪种前缀"的决策树。自查：我的 `claude-for-legal` 同样有"appropriate"，应补充决策条件。

3. **telemetry 未披露** → `posthog-node` 在 npm install 时默默收集使用数据，无 README 说明，无 opt-out 机制。根本原因：作者认为是合理的产品遥测，忽略了用户知情权。自查：我的工具类项目如果将来引入遥测，必须先在 README 明确说明。

### 3.4 当时的优化机会
1. 修复 4 个跨平台接口分歧（最高优先级，影响构建和运行时正确性）
2. 为 telemetry 添加 README 披露和 opt-out env var
3. 把"appropriate"替换为明确的类型前缀选择规则

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| llm-provider 跨平台接口分歧 | 对比 `.claude/`、`.agents/`、`.cursor/` 的 stream() 签名 | **已修复**：三平台均显示 `stream(options: LLMStreamOptions, callbacks: LLMStreamCallbacks): Promise<void>` | NLPM 提交的 PR #178 已合并，校准成功 |
| writers-pattern sync/async 分歧 | 对比三平台 writeSetup 相关调用 | **已修复**：`.agents/` 和 `.claude/` 均显示统一的 Integration point 描述 | PR #180 已合并 |
| telemetry 未披露 | 查 README + package.json | **已修复**：README 加入了 telemetry 披露和 `CALIBER_TELEMETRY_DISABLED=1` opt-out | PR #182 已合并 |

全部关键 Bug **均已修复**，这是本批次 4 个案例中唯一一个"所有 Bug 已修复"的案例，验证了 NLPM → PR → 合并的完整闭环。

### 4.2 架构演进
Audit 后的新增变化（对比 Audit 快照）：
- 新增 `AGENTS.md`：面向 agent 的元指令（与 antfu/skills 相同的模式）
- 新增 `.claude/rules/scoring-patterns.md`：把 scoring 相关的规则从 skill 中提取为独立 rules 文件
- 新增 `.context/notes.md` 和 `.context/todos.md`：轻量级 context 管理文件
- 新增 `CONTRIBUTING.md`、`CODE_OF_CONDUCT.md`：说明对外接受贡献，开放了接收 PR 的信号

这说明作者接受了 NLPM 的贡献，并在此基础上继续扩展元文档体系。**AGENTS.md 的新增可能是受 NLPM 提交 PRs 这一事件本身的启发**。

### 4.3 新增的可学习模式
- **`.claude/rules/` 目录**：把"校验模式"类规则（不是操作指南，而是约束规则）单独放进 `.claude/rules/`，与 SKILL.md 分层。这是比 CLAUDE.md 更细粒度的约束层。
- **`.context/` 目录**：轻量级跨会话 context 存储（notes/todos），不是 SKILL.md，而是 agent 工作时可以读写的"便笺"。

---

## 五、校准

### 5.1 我已经在做对的
1. **Critical 段分离**：我的 `claude-for-legal` 的 skill 主体也有"Critical"/"不得"类约束段
2. **稳定 skill 不需要多平台适配**：我没有多平台需求，省去了同步成本
3. **telemetry 零引入**：我的三个仓库均无 posthog 或类似遥测，隐私风险为零

### 5.2 挑战 / 验证
这次案例验证了一个反直觉的结论：**NL 质量 100 分不代表代码会正确运行**。

caliber 的 25 个 NL 工件个个质量极高，但 4 个 skill 描述了错误的 TypeScript 接口，agent 照着做会产生在编译时就失败的代码。NL 评分和技术准确性是两个完全独立的维度。

对我自己的提示：**技术性 skill（描述接口、函数签名、协议格式）需要定期对照源码校验，而不仅仅是 NL 审查**。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的 skill 是否有 "paths:" frontmatter（学习 caliber 的范围约束）
grep -rn "^paths:" /tmp/my-repos/MarkQWu-* --include="SKILL.md" | head -10 || echo "无 paths: 约束"
```
命中后怎么办：继续保持。未命中：对技术性 skill 考虑加入 `paths:` 字段约束适用范围。

```bash
# 检查是否有描述代码接口的 skill（需要定期校验接口准确性）
grep -rn "interface\|function\|class\|export" /tmp/my-repos/MarkQWu-* --include="SKILL.md" | head -10
```
命中后怎么办：把 skill 里描述的接口和实际代码对照，确保签名一致。

### 6.2 灵感 → 实施路径
1. **想法**：为 echo-sleuth 的 `skills/` 目录添加 `paths:` frontmatter 字段
   - **为何可行**：echo-sleuth 的每个 skill 明确对应特定的 JSONL 文件操作，加 `paths:` 可以缩小 agent 的适用范围
   - **第一步**：读 `jsonl-core/SKILL.md` 的 frontmatter，加 `paths: - ~/.claude/conversations/*.jsonl`，约 5 分钟

2. **想法**：在 drama-workshop-skills 里加 `.claude/rules/` 目录存放行为红线规则
   - **为何可行**：短剧创作有一些绝对禁区（如"不写真实人名""不写犯罪情节"），放进 rules/ 比分散在各 skill 里更权威
   - **第一步**：建 `.claude/rules/content-safety.md`，列 5 条红线规则，约 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **caliber-ai-org/ai-setup 的核心目的**：一条命令把 AI skill 同步到 Claude/Cursor/Codex 三平台
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是工具类 Claude Code 插件 | echo-sleuth 是单平台，caliber 是多平台同步工具 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都有 SKILL.md | 领域不同（多平台同步 vs 短剧创作） | 低 |
| MarkQWu/claude-for-legal | 低 | 都有多 skill 集合 | 工具定位不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| R01：模糊量词 "appropriate" | `grep -rn "appropriate" claude-for-legal --include="SKILL.md"` | **claude-for-legal 命中**（matter-workspace 3处） | 低 |
| 技术性 skill 接口准确性 | 手动对照 skill 里的函数签名和实际代码 | **echo-sleuth 需要校验**：`jsonl-core/SKILL.md` 描述的 grep 模式和实际代码有无出入？ | 中（待验证）|

**命中后的具体行动建议**：
- 打开 `echo-sleuth/skills/jsonl-core/SKILL.md`，对照 `references/extraction-patterns.md` 里的 grep 命令，用 `bash -n` 验证语法，约 10 分钟

### 8.3 别人的更优方案

1. **领域**：Critical 段 + Instructions 段分层
   - **caliber 做法**：每个 SKILL.md 先写 `## Critical`（不可违反的约束），再写 `## Instructions`（操作步骤）；agent 读完 Critical 段就知道红线在哪
   - **我的现状**：`claude-for-legal` 的 skill 没有统一的 Critical 段，约束散布在正文各处
   - **如何借鉴**：把每个 legal skill 里的"MUST NOT"/"WARNING"类句子汇总到 `## Critical` 段顶部

2. **领域**：`paths:` frontmatter 精确映射
   - **caliber 做法**：`paths: - src/llm/**/*.ts` 告诉 agent 这个 skill 管辖哪些文件
   - **我的现状**：所有 skill 均无 `paths:` 字段，agent 需要靠名称推断适用范围
   - **如何借鉴**：为 `echo-sleuth/skills/jsonl-core/SKILL.md` 加 `paths: - ~/.claude/projects/**/*.jsonl`

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：单平台聚焦
- **我的做法**：只支持 Claude Code，不需要多平台同步，零漂移风险
- **caliber 做法**（弱在哪）：4 个 skill 因跨平台漂移产生接口分歧，需要外部审计才能发现
- **意义**：多平台支持是有价值的功能，但它带来了"跨平台接口漂移"这个持续的维护成本——我目前的单平台策略免疫了这类问题

---

## 八、术语表

### <a name="telemetry"></a>telemetry（遥测）
> 软件在用户不知情或未主动许可的情况下，自动收集使用数据（命令调用频率、功能使用情况等）并发送到开发者服务器的机制。caliber 使用 `posthog-node` 收集 CLI 使用数据，Audit 前未在 README 中说明，也没有 opt-out 机制。

### <a name="paths-frontmatter"></a>paths: frontmatter
> SKILL.md 文件顶部 `---` 区域（frontmatter）中的 `paths:` 字段，用于告诉 Claude Code"这个 skill 适用于哪些文件路径"。当 agent 打开对应路径的文件时，Claude Code 会自动加载声明了该路径的 skill。

### <a name="posthog"></a>PostHog
> 一款开源的产品分析平台，提供事件跟踪、用户行为分析等功能。`posthog-node` 是其 Node.js SDK，引入后每次 CLI 命令执行都会发送一条事件到 PostHog 服务器。

### <a name="named-export"></a>named export（命名导出）vs default export（默认导出）
> TypeScript/JavaScript 中两种模块导出方式。`export function foo()` 是命名导出，调用方用 `import { foo } from '...'`；`export default function foo()` 是默认导出，调用方用 `import foo from '...'`。caliber 的 Bug #3 就是 `.cursor/` skill 描述了默认导出，但实际代码用命名导出，导致 agent 生成的代码无法被正确 import。
