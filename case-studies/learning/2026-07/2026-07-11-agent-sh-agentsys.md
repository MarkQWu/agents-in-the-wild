# agent-sh/agentsys — 学习案例

**仓库**：https://github.com/agent-sh/agentsys
**Stars**：755（来源：xiaolai 2026-04 文章）| **来源**：本地 audit（case_study_candidate=True）
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-07-11（基于当前 HEAD）
**主题标签**：`examples-driven`, `security-gate`, `single-purpose`, `cross-reference`

**xiaolai 案例**：[../../2026-04-24-avifenesh-agentsys.md](../../2026-04-24-avifenesh-agentsys.md)（同一仓库；另见早期学习案例：[../2026-05/2026-05-31-avifenesh-agentsys.md](../2026-05/2026-05-31-avifenesh-agentsys.md)）

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

[agent-sh/agentsys](https://github.com/agent-sh/agentsys)（原 avifenesh/agentsys）是一个面向 AI 辅助编码工作流的「技能集合型」插件，定位于「AI 写代码，agentsys 负责其他所有事情」。755 颗星印证了这一定位对社区的吸引力——它不是一个「帮你写代码」的工具，而是补全了 AI 编码生命周期中被忽视的周边环节：性能调优、文档同步、仓库情报、代码评审编排、提示词优化等。

关键事实：

- **规模**：项目自报 19 个插件、47 个 agent、40 个 skill；audit 时直接计入 NLPM 扫描的工件为 32 个（30 个 skill、1 个 CLAUDE.md、1 个 plugin.json）
- **双平台布局**：同一套技能同时服务 Claude Code（`.claude-plugin/`）和 Kiro AI（`.kiro/`），以及 OpenCode 运行时
- **版本一致性**：`package.json` 与 `.claude-plugin/plugin.json` 均为 v5.8.3，版本号对齐
- **自 audit 以来的演进**：仓库规模明显扩大，新增了 `hooks/` 目录，工件数量超过 audit 时的 32 个

### 1.2 架构剖析

```
agent-sh/agentsys/
├── .claude-plugin/plugin.json      ← Claude Code manifest（v5.8.3）
├── .kiro/
│   └── skills/                     ← 主技能目录（Claude Code + Kiro 共用）
│       ├── web-auth/SKILL.md       ← 浏览器认证自动化（已修复硬编码路径）
│       ├── web-browse/SKILL.md     ← 浏览器操控（已修复 40+ 处硬编码路径）
│       ├── enhance-prompts/SKILL.md
│       ├── enhance-skills/SKILL.md
│       ├── enhance-agents/SKILL.md
│       ├── enhance-orchestrator/SKILL.md  ← 含区分表：区别于 enhance-agent-prompts
│       ├── perf-code-paths/SKILL.md   ┐
│       ├── perf-benchmarker/SKILL.md  │
│       ├── perf-baseline-manager/SKILL.md │  8 个协同技能
│       ├── perf-analyzer/SKILL.md     │  全部引用 docs/perf-requirements.md
│       ├── perf-theory-gatherer/SKILL.md │  作为规范契约
│       ├── perf-theory-tester/SKILL.md   │
│       ├── perf-profiler/SKILL.md    │
│       ├── perf-investigation-logger/SKILL.md ┘
│       ├── consult/SKILL.md
│       ├── learn/SKILL.md
│       ├── drift-analysis/SKILL.md
│       ├── orchestrate-review/SKILL.md  ← audit 最高分（98/100）
│       ├── deslop/SKILL.md
│       ├── debate/SKILL.md
│       ├── discover-tasks/SKILL.md
│       └── meta/skills/maintain-cross-platform/SKILL.md  ← 1024 行元技能
├── docs/perf-requirements.md       ← 性能调优流水线的规范契约
├── CLAUDE.md                       ← 项目说明（audit 88/100）
├── hooks/                          ← 【新增】钩子目录（audit 后引入）
└── package.json                    ← 含 prepare 脚本（已修复为 setup-hooks）
```

**关键架构关系**：

- `maintain-cross-platform/SKILL.md` 是整个仓库的「元层」：1024 行文档记录了跨平台约定、内部引用路径规范、命名规则，所有其他技能的跨平台行为都以此为依据
- 8 个 `perf-*` 技能形成一条流水线，但彼此并不互相引用——它们共同引用 `docs/perf-requirements.md` 这一外部契约，实现了「松耦合 + 强一致」
- `enhance-orchestrator/SKILL.md` 内嵌区分表，明确说明自身与 `enhance-agent-prompts` 和 `enhance-prompts` 的边界，防止使用者混淆

### 1.3 设计思路 / 方法论

- **「AI 负责写，技能负责其余」**：agentsys 不与 AI 编码核心功能竞争，而是补全 AI 编码之外的工程环节——性能、文档、评审、提示词质量。定位清晰是高分（91/100）的基础之一
- **单一职责到技能粒度**：`enhance-prompts`、`enhance-skills`、`enhance-agents` 分别对应不同的提示词优化对象，而非合并为一个万能的「enhance」技能。粒度细意味着使用者无需在技能内部用参数区分场景
- **共享规范契约模式**：8 个 perf-* 技能不靠互相引用保持一致，而是全部指向同一份 `docs/perf-requirements.md`。增删任一技能不影响其他技能；修改契约则立即传播到全部 8 个
- **多平台设计不是事后兼容**：`maintain-cross-platform/SKILL.md` 的存在说明跨平台支持是设计时的一等公民，不是「再适配一个运行时」的补丁

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名称 | 在 agentsys 中的体现 | 对应 NLPM 规则 |
|----------|---------------------|--------------|
| 单一职责技能 | enhance-prompts / enhance-skills / enhance-agents 三拆一 | R05（单一目的） |
| 共享规范契约 | 8 个 perf-* 技能共引 docs/perf-requirements.md | R18（规范引用） |
| 元层技能 | maintain-cross-platform 记录跨平台约定 | R22（设计文档化） |
| 明示区分表 | enhance-orchestrator 内嵌与相邻技能的对比表 | R11（边界清晰） |
| 流水线松耦合 | perf-* 技能彼此独立，通过契约文件对齐 | R18（规范引用） |

### 2.2 适用场景

- **共享规范契约**：适合一组技能共同服务同一领域（如性能、安全、文档），且需要保持语义一致，但又不应彼此硬依赖的情况
- **单一职责拆分**：适合一个主题下存在 2–4 个自然子类的情况；若子类超过 5 个，考虑引入分层（父技能 + 子技能引用）
- **明示区分表**：适合技能名称相似、用途容易混淆的情况；比在描述里用「本技能不包括……」更易扫描

### 2.3 与其他架构对比

| 对比项 | agentsys（技能集合型） | grill-for-claude（命令编排型） | bureau（命令+agent 型） |
|--------|----------------------|------------------------------|------------------------|
| 入口 | 无统一入口，按需加载单个技能 | 单一 `/grill:roast` 命令 | 多个 commands/ 入口 |
| 协同机制 | 共享外部契约文件 | 共享 grill-core skill | agent model 字段统一 |
| 跨平台 | 明确支持（.kiro/ + .claude-plugin/） | 后期扩展 Codex | 单平台 |
| 适合规模 | 大型（30+ 技能） | 中型（6 agent + 1 skill） | 中型 |

### 2.4 改进空间

- **示例块覆盖率低**：audit 时 12 个 perf-* 及周边技能均无示例块，每个扣 15 分；这是最可量化的集体缺陷
- `maintain-cross-platform/SKILL.md` 超出 500 行限制（1024 行），是技术债——元层文档过长容易变成「谁都知道但谁都不维护」的只写文档
- CLAUDE.md 规则缺少 WHY 解释，且存在负面措辞（「不要……」而非「请……」）；高分（88）掩盖了可读性问题

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

**整体得分：91 / 100**，安全状态：CLEAR

| 文件 | 分数 | 主要扣分点 |
|------|------|-----------|
| orchestrate-review/SKILL.md | 98 | 近乎完美，仅有极小措辞瑕疵 |
| maintain-cross-platform/SKILL.md | 95 | 超出 500 行限制（1024 行）|
| web-auth/SKILL.md | 90 | 硬编码 `/Users/avifen/` 绝对路径 |
| web-browse/SKILL.md | 90 | 同上，且出现 40+ 次 |
| CLAUDE.md | 88 | 规则缺 WHY 解释；含负面措辞 |
| perf-code-paths 等 11 个 perf-* 技能 | 85 | 缺示例块（各扣 -15） |
| repo-intel/SKILL.md | 85 | 缺示例块 |
| sync-docs/SKILL.md | 83 | 缺示例块 + 模糊量词 |

### 3.2 当时值得借鉴的模式

- `orchestrate-review/SKILL.md`：完整的 ## Output Format、清晰的步骤编号、明确的错误处理——这三点组合使其接近满分，是全仓库的最佳范本
- `maintain-cross-platform/SKILL.md`：尽管超长，但跨平台约定写成技能而非散落在 README 中，这个组织决策本身正确
- 8 个 perf-* 技能全部引用同一份 `docs/perf-requirements.md`：即使技能本身示例不足，契约引用模式仍值得复制

### 3.3 当时的缺陷

**安全类（Medium）**：

- `package.json` 中 `prepare` 生命周期脚本在每次 `npm install` 时自动安装 git hooks，属于非预期的副作用行为（用户未必知情）
- `web-auth/SKILL.md` 硬编码 `/Users/avifen/.agentsys/...`：对所有非 avifen 用户失效，且暴露了开发者本地路径
- `web-browse/SKILL.md` 同上问题，且出现超过 40 次，影响范围更大

**安全类（Low）**：

- `js-yaml` 依赖版本范围为 `^4.1.1`（允许任意小版本跳升），过于宽松
- `agentsys` 包存在自引用依赖

**质量类**：

- 12 个技能缺示例块（-15/个）
- CLAUDE.md 规则无 WHY 说明，含否定式措辞
- `sync-docs/SKILL.md` 含模糊量词（「尽快」「尽量」类表述）

### 3.4 当时的优化机会

- 将 `maintain-cross-platform/SKILL.md` 拆分为「约定概览技能（≤500 行）」+ 「平台细节参考文档」两个文件，各司其职
- 为 perf-* 技能统一补充 ## Examples 块，可使用模板化格式（输入场景 → 预期输出）批量生成
- 将 CLAUDE.md 中「不要做 X」改为「请做 Y（因为 Z）」，增加可遵循性

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | audit 状态 | 2026-07-11 状态 | PR |
|------|-----------|----------------|-----|
| web-auth/web-browse 硬编码 `/Users/avifen/` | 存在 | **已修复**（grep 无结果） | #333 |
| prepare 脚本自动安装 git hooks | 存在 | **已修复**（改为 `setup-hooks` 手动命令） | #334 |
| js-yaml `^4.1.1` 版本范围过宽 | 存在 | **部分修复**（`~4.2.0`，波浪线限制补丁版本） | #335 |
| 12 个技能缺示例块 | 存在 | 未验证（需逐一 grep） | — |
| CLAUDE.md 规则缺 WHY | 存在 | 未验证 | — |

**补充说明**：
- #333、#334、#335 三个 PR 均于 2026-04-23 合并，响应速度（约 11 天）反映了维护者的活跃程度
- `js-yaml` 改为 `~4.2.0` 是「部分修复」：波浪线语法限制补丁版本跳升（允许 4.2.x，不允许 4.3.x），比 `^` 严格但仍非精确锁定（`=4.2.0` 才是）；对安全而言已足够，但对可复现构建不够
- agentsys 自引用依赖：未修复，可能是有意为之（monorepo 内部引用场景）

### 4.2 架构演进

- 新增 `hooks/` 目录：说明 agentsys 从纯技能集合扩展到了钩子层，覆盖更多工作流触发点
- 工件数量从 32 增加（具体数量未完整统计）：仓库在持续生长
- 版本从早期升至 v5.8.3：agentsys 是成熟迭代的项目，不是一次性发布

### 4.3 新增的可学习模式

- `hooks/` 目录的引入提供了一个额外维度：技能被动响应调用，钩子主动在工具事件时触发——二者组合能覆盖更完整的工作流生命周期
- 版本号在 `package.json` 与 `plugin.json` 双重保持一致（v5.8.3），说明维护者在发布时有意识地执行版本对齐检查；这是可以自动化的基础设施动作

---

## 五、校准

### 5.1 我已经在做对的

- **单一职责切分**：`gstack` 的 `openclaw/skills/` 按功能域分目录，与 agentsys 的细粒度拆分思路一致
- **有 frontmatter**：`gstack` 技能使用 `name:` 和 `preamble-tier:` 字段，具备基本的元数据结构
- **避免模糊量词**：`gstack` 技能语言相对精确，低模糊量词密度与 agentsys 最高分技能（orchestrate-review）的写法相近
- **多技能项目有目录组织**：`drama-workshop-skills` 作为社区技能集，与 agentsys 的定位最接近，均是「策划供他人使用的技能库」

### 5.2 挑战 / 验证

- **待验证**：`gstack` 和 `bureau` 技能是否有示例块？agentsys 最大的集体失分就是缺示例（-15/个），我的技能是否有同样问题？
- **待验证**：我的 CLAUDE.md 或 AGENTS.md 规则是否有 WHY 说明？还是只有裸规则？
- **待验证**：`drama-workshop-skills` 与 `gstack` 技能的 `model:` 字段：audit 对 agentsys 未特别标记 model 字段缺失问题，说明技能本身不强制要求 model 字段（model 字段更多是 agent 层面的约束）——我的技能可能也不需要；但若 `bureau` 的 agent 文件缺 model 字段，则是实际缺陷
- **挑战**：我尚未建立类似 agentsys `docs/perf-requirements.md` 的「跨技能规范契约」文件——当我有多个技能服务同一领域时（如 `gstack` 的多个 openclaw 技能），它们是否通过某份文档保持语义对齐？

---

## 六、行动

### 6.1 自查动作

**检查示例块覆盖率（gstack + bureau）**：

```bash
# 扫描所有 SKILL.md 文件，找出缺少 ## Examples 块的
grep -rL "## Example" /path/to/gstack/openclaw/skills/ --include="SKILL.md"
grep -rL "## Example" /path/to/bureau/skills/ --include="SKILL.md"
```

**检查硬编码本地路径**：

```bash
# 替换为你的实际用户名
grep -r "/Users/markwu\|/home/markwu\|MarkQWu" \
  /path/to/gstack/openclaw/skills/ \
  /path/to/bureau/skills/ \
  --include="*.md"
```

**检查 CLAUDE.md / AGENTS.md 规则是否有 WHY 说明**：

```bash
# 找出「不要」「禁止」「不得」开头的规则行（负面措辞）
grep -rn "不要\|禁止\|不得\|Never\|Don't\|Do not" \
  /path/to/bureau/CLAUDE.md \
  /path/to/gstack/CLAUDE.md \
  --include="*.md"
```

**检查 drama-workshop-skills 与 agentsys 是否有共同领域技能（需规范契约）**：

```bash
# 列出你的技能目录名，手动判断是否有 2+ 个技能服务同一领域
ls /path/to/drama-workshop-skills/
ls /path/to/gstack/openclaw/skills/
```

**检查 bureau agent 文件是否有 model 字段**：

```bash
grep -rL "^model:" /path/to/bureau/agents/ --include="*.md"
```

**检查 package.json 中的 prepare 脚本（如果有 Node 项目）**：

```bash
# 找出所有包含 prepare 脚本的 package.json
grep -rn '"prepare"' /path/to/your-repos/ --include="package.json"
```

### 6.2 灵感 → 实施路径

**灵感 1：为 drama-workshop-skills 中服务同一领域的技能建立规范契约文件**

- 参照 `docs/perf-requirements.md` 模式
- 实施路径：
  1. 识别 `drama-workshop-skills` 中有 2 个或以上技能共享某个核心假设或术语的领域
  2. 在该目录下新建 `docs/<domain>-requirements.md`，列出核心定义和约束
  3. 在相关技能的 `## Context` 或 `## Inputs` 节中添加一行引用：`参见 docs/<domain>-requirements.md`
  4. 验证：运行 `/nlpm:check` 查看跨件引用一致性

**灵感 2：为 gstack 和 bureau 技能补充示例块**

- 参照 `orchestrate-review/SKILL.md`（agentsys 最高分）的格式：输入场景描述 → 预期动作 → 预期输出样例
- 实施路径：
  1. 从 `6.1` 的 grep 结果中拿到缺失列表
  2. 优先处理「最常被调用」的技能（一般是工具列表靠前的几个）
  3. 每个示例块保持 10–20 行，不求完整展示所有分支

**灵感 3：将 CLAUDE.md 负面措辞改写为正向+理由格式**

- 模板：「~~不要做 X~~」→「请做 Y（因为 Z）」
- 实施路径：对 `6.1` grep 到的每条负面规则，逐一改写；改写后跑 `/nlpm:score` 确认 CLAUDE.md 分数提升

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | agentsys 对应功能域 | 对齐度 |
|---------|-------------------|--------|
| `drama-workshop-skills` | 社区技能策划集合（最近似） | 高——同为「供他人使用的策划技能库」 |
| `gstack` | openclaw 工具集，类似 agentsys 的功能性技能 | 中——gstack 更聚焦单一工具栈 |
| `bureau` | commands + agents + skills 的组合型工作流 | 中——bureau 有命令层，agentsys 无 |
| `echo-sleuth-for-claude` | 会话存档类，agentsys 无直接对应 | 低 |
| `graphify` | 知识图谱单技能，agentsys 无直接对应 | 低 |

### 8.2 在我的项目里复现的同类问题

**高概率存在（需用 6.1 命令验证）**：

- **`gstack` + `bureau` 技能缺示例块**：agentsys 12 个技能因此各扣 15 分；若我的技能也缺，预计 NLPM 评分会有类似折损。`gstack` 的 `openclaw/skills/` 和 `bureau/skills/` 是首要排查对象
- **CLAUDE.md 规则无 WHY 说明**：agentsys 的 CLAUDE.md 因此得 88 分而非 92+；我的 `bureau` 或 `gstack` CLAUDE.md 大概率有同类问题，因为写规则时很容易忘记解释动机
- **`bureau` commands 缺 `allowed-tools` 字段**：audit 提到 agentsys 某些 command 字段不完整；`bureau` 的 commands/ 有 `description:` 但可能缺 `allowed-tools`——这是同类问题

**低概率但需检查**：

- 硬编码本地路径：若 `gstack` 的技能曾在本地开发时写入过绝对路径，需清理（agentsys 的 web-browse 有 40+ 处，说明这种问题在快速开发时很容易蔓延）

### 8.3 别人的更优方案

- **跨技能规范契约（`docs/perf-requirements.md` 模式）**：agentsys 的 8 个 perf-* 技能通过这份文件保持语义一致，且彼此解耦。我的 `drama-workshop-skills` 若有多个相关技能，目前没有等价的契约文件——每个技能各自定义术语，迟早出现漂移
- **明示区分表（`enhance-orchestrator` 模式）**：agentsys 在技能内嵌对比表，区分相邻技能。我在 `gstack` 或 `drama-workshop-skills` 中若有功能相近的技能，尚未采用这种写法；一旦技能超过 5 个，使用者分辨成本上升，区分表的价值就会显现
- **元层技能（`maintain-cross-platform` 模式）**：即使不需要跨平台，把「技能库的设计约定」写成一个独立的技能文件（而非散在 README 里），对后续维护者是更好的导引

### 8.4 反向：我的项目做得比他们好的地方

- **`bureau` 的 agent model 字段**：`bureau` 的 agent 文件声明了 `model: sonnet`，这使得模型选择显式可见；agentsys 的 audit 未明确提及 model 字段，说明其技能层可能没有模型约束声明——在未来模型版本迁移时，`bureau` 的显式声明更安全
- **`bureau` 命令有 `description:` 字段**：agentsys 作为技能集合不使用命令层，但 `bureau` 为每个命令提供了描述，使命令在 `/` 触发列表中语义清晰——这是 agentsys 架构上没有的维度
- **较小的技能库规模**：我的技能库规模较小，意味着 `maintain-cross-platform` 那种因体量膨胀导致的超长文件问题（1024 行）短期内不会出现；在规模还小时保持每个技能 ≤500 行比等到超长后再拆分容易得多

---

## 八、术语表

| 术语 | 含义 |
|------|------|
| **性能调优流水线**（perf pipeline） | agentsys 中 8 个 `perf-*` 技能构成的协同链路：从路径识别（perf-code-paths）、基准采集（perf-benchmarker/perf-baseline-manager），到理论分析（perf-theory-gatherer/perf-theory-tester）、剖析（perf-profiler）、记录（perf-investigation-logger），全链路覆盖一次性能调优闭环 |
| **frontmatter** | Markdown 文件顶部以 `---` 包裹的 YAML 元数据块，在 NLPM 技能文件中用于声明 `name:`、`description:`、`model:`、`skills:` 等字段 |
| **manifest**（清单文件） | 插件的配置声明文件，如 `.claude-plugin/plugin.json`；告知运行时该插件包含哪些命令、技能和 agent，是插件被加载的入口 |
| **postinstall hook**（安装后钩子） | `package.json` 中 `scripts.prepare` 或 `scripts.postinstall` 字段指定的脚本；在 `npm install` 完成后自动执行。agentsys 的问题在于用此机制静默安装 git hooks，产生了用户未授权的副作用 |
| **规范契约**（canonical contract） | 多个技能或 agent 共同引用的单一权威文档，用于统一术语定义、接口格式或行为约束。agentsys 的 `docs/perf-requirements.md` 是典型实例：8 个 perf-* 技能各自独立，但对「性能要求」的定义均以此文件为准 |
| **元层技能**（meta skill） | 描述「如何维护其他技能」的技能，不直接执行业务任务，而是为整个技能库的维护者提供约定参考。agentsys 的 `maintain-cross-platform/SKILL.md` 是典型实例 |
| **波浪线版本范围**（tilde range） | npm 依赖版本语法 `~x.y.z`：允许补丁版本升级（`x.y.*`），不允许次版本升级。比 `^x.y.z`（允许次版本升级）更严格，但仍非精确锁定（`=x.y.z`） |
