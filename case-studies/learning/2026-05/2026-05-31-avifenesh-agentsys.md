# avifenesh/agentsys — 学习案例

**仓库**：https://github.com/avifenesh/agentsys
**Stars**：741 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-31（基于当前 HEAD）
**主题标签**：`cross-reference`, `single-purpose`, `model-pinning`, `manifest-discipline`, `nl-binary-hybrid`

**xiaolai 案例**：[../../../case-studies/2026-04-24-avifenesh-agentsys.md](../../../case-studies/2026-04-24-avifenesh-agentsys.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

agentsys 是一个**多平台 AI agent 运行时与编排系统**，口号是"AI 写代码，agentsys 自动化其他一切"。项目由 Avi Fenesh 维护，托管在 `agent-sh` 组织下，当前版本 6.0.0（审查时 5.8.3）。安装方式：通过 `npm install agentsys` 后运行 `agentsys setup` 自动将技能分发至 Claude Code、OpenCode、Codex CLI、Cursor 四个平台的对应目录。仓库在 Claude Code 生态中属于**超大规模 skill 套件**的代表：32 个审查工件、27 个当前 `.kiro` skill（较审查时 30 个略有减少）。

关键事实：
- 741 stars，81 forks（2026-04-06 审查数据）
- Kiro / Claude / OpenCode / Codex 四平台并发维护
- 含 8 个协同 perf-\* skills 组成 pipeline
- 含 `meta/skills/maintain-cross-platform/SKILL.md` 这一特殊自维护元技能

### 1.2 架构剖析

```
agentsys/
├── .claude-plugin/plugin.json       ← manifest（当前版本 6.0.0）
├── CLAUDE.md                         ← 项目记忆（Critical Rules 置顶）
├── AGENTS.md                         ← agent 总索引
├── .kiro/skills/                     ← 27 个 Kiro skill（*.SKILL.md）
│   ├── perf-code-paths/
│   ├── perf-theory-gatherer/
│   ├── perf-benchmarker/
│   ├── perf-profiler/
│   ├── perf-theory-tester/
│   ├── perf-baseline-manager/
│   ├── perf-analyzer/
│   ├── perf-investigation-logger/    ← 8 个 perf-* 形成有序 pipeline
│   ├── enhance-skills/
│   ├── enhance-agent-prompts/
│   ├── enhance-orchestrator/
│   ├── deslop/ consult/ debate/ ...
│   └── web-auth/ web-browse/         ← 审查时存在，已从仓库移除（路径 bug 已消灭）
├── meta/skills/maintain-cross-platform/ ← 跨平台兼容性元技能（1023 行）
├── scripts/                          ← dev-install.js 等安装脚本
└── package.json                      ← postinstall → prepare 自动装钩
```

**文件类型分布**：27 个 SKILL.md / 1 个 plugin.json manifest / 1 个 CLAUDE.md / 多个 JS 安装脚本

**编排关系**：skills 之间彼此独立（平列），但 perf-\* 8 个在 `docs/perf-requirements.md` 约定下形成隐式管道。`maintain-cross-platform/SKILL.md` 是全局元技能，包含 Kiro/Claude/Codex 转换规则。

**跨件契约**：plugin.json 版本与 package.json 保持同步；`consult/SKILL.md` 和 `debate/SKILL.md` 共用外部工具引用节保持一致；perf-\* skills 共同引用 `docs/perf-requirements.md` 作为不变式契约。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：约定优于配置。所有 skill 遵守统一 frontmatter 格式（name / description / trigger-phrase），安装脚本则负责路径变换——同一份 SKILL.md 通过 transformer 输出到不同平台目录。
- **解决什么问题**：开发者在多个 AI 工具之间切换时需要重复配置相同的 skill；agentsys 的安装脚本解决一次写、多平台生效的分发问题。
- **做了什么 trade-off**：安装时自动修改用户 `~/.claude/settings.json`（见 `dev-install.js`）。这换来了开箱即用的体验，但也引入了用户主目录写入和 npm postinstall 静默执行的安全面。
- **反映什么认知模型**：作者认为 NL skill 是**可移植的文本协议**，应该对工具平台无感知——只要 transformer 处理路径和格式差异，skill 本身保持纯粹。这和 "Write once, deploy everywhere" 的 Java 理念在 NL 层面的同构。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「Transformer 分发 + 单一源 skill」

核心思路：skill 文本仅有一份，安装时由脚本（transformer）按目标平台做路径替换和格式适配，分发至 `.kiro/`、`.claude/`、`.codex/` 等目录。

特征清单：
- 特征 1：Skill 本身不含平台特定路径，由安装时注入（DI 思想）
- 特征 2：perf-\* 多 skill 共享外部约定文件（`docs/perf-requirements.md`）作为稳定契约
- 特征 3：元技能（`maintain-cross-platform`）以 NL 描述跨平台兼容性规则，形成自文档化
- 特征 4：manifest（plugin.json）版本和 package.json 版本严格同步
- 特征 5：CLAUDE.md Critical Rules 置顶，避免"中间丢失"问题

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要同时支持多个 AI 工具平台的 skill 套件 | ✅ 高度适用 | Transformer 分发是这个问题的最优解之一 |
| 单人/小团队维护、追求长期可维护性的项目 | ✅ 适用 | 单一源减少 drift |
| 纯脚本/数据处理工具（不跨 AI 平台） | ❌ 不适用 | transformer 是过度工程 |
| 需要平台特定高度定制（如 Kiro 的 steer 协议、Claude 的 hooks） | ⚠️ 谨慎 | transformer 只处理路径替换，平台特定语义需另行处理 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Transformer 分发（本仓库） | agentsys | 单一源，一次改、多处生效 | transformer 本身需要维护；安装逻辑复杂 |
| 备选 A：独立仓库各平台维护 | 大多数 skill 作者 | 简单直接、平台定制灵活 | 内容 drift 不可避免，修复需要多个 PR |
| 备选 B：Runtime 统一层（如 agentsys 未来方向） | 尚未出现成熟案例 | 彻底消除安装步骤 | 需要所有平台支持统一 API，目前不现实 |

### 2.4 改进空间

1. **当前问题**：`maintain-cross-platform/SKILL.md` 约 1023 行，远超建议 500 行上限，认知负荷高。**改进做法**：将"安装原理"、"平台差异速查表"、"常见 pitfall"拆分为 `reference/` 子目录的三个文件，主 SKILL.md 只保留触发条件和高层决策树。**预期收益**：主文件可读性提升，深度内容不丢失。

2. **当前问题**：postinstall 静默写入用户 `~/.claude/settings.json`，用户无感知。**改进做法**：在 npm install 结束后打印一行清单，列出修改了哪些文件（已写入的 plugin paths）。**预期收益**：用户信任度提升，减少"为什么我的 settings 变了"的 issue。

3. **当前问题**：perf-\* 管道依赖 `docs/perf-requirements.md` 但这个契约文件不在 NLPM 扫描范围。**改进做法**：在每个 perf-\* SKILL.md frontmatter 中加 `references: ["../../docs/perf-requirements.md"]` 显式声明依赖。**预期收益**：工具链可以验证契约文件存在，broken-reference 风险降低。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **97/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.kiro/skills/web-auth/SKILL.md` | 90 | 硬编码路径 `/Users/avifen/.agentsys/` |
| `.kiro/skills/web-browse/SKILL.md` | 90 | 同上 |
| `.claude-plugin/plugin.json` | 90 | 非 NL 工件，内容最少 |
| `CLAUDE.md` | 92 | "Non-trivial changes"轻微模糊 |
| `meta/skills/maintain-cross-platform/SKILL.md` | 95 | 约 1000 行，建议上限 500 |
| 其余 27 个 skills | 97–98 | 干净 |

### 3.2 当时值得借鉴的模式

1. **完整 frontmatter 覆盖率 100%**：30 个 skills 无一遗漏 `name`/`description`，这在 30+ 工件的仓库中极为罕见。→ 根本原因：作者把 frontmatter 当作 skill 能被注册的门槛，不是可选项。→ 如何借鉴：在自己的 skill 模板里写明"缺 frontmatter = 不可见"。

2. **perf-\* 8 技能有序管道**：`code-paths → theory-gatherer → benchmarker → profiler → theory-tester → baseline-manager → analyzer → investigation-logger`，每个 skill 都有单一入口、明确退出条件，且全部引用同一契约文件。→ 根本原因：将复杂任务分解为可独立测试的单元，同时用共享契约保持一致性。→ 如何借鉴：`drama-workshop-skills` 的创作流程（选题→策划→分镜）可以参考这种显式管道设计。

3. **CLAUDE.md Critical Rules 置顶**：规则紧接 heading 之后，避免 AI 在长上下文中"中间丢失"。→ 如何借鉴：检查自己所有 CLAUDE.md，确保最重要规则在前 30 行内。

4. **plugin.json 与 package.json 版本严格同步**：两处版本号（5.8.3）完全一致。→ 如何借鉴：每次发版后用 `jq` 校验两者是否一致。

5. **consult/debate 共享外部工具节**：两个相关 skill 的外部工具引用节内容保持同步，避免分叉。→ 如何借鉴：同类 skill 提取共用参考为 `references/common-tools.md`。

### 3.3 当时的缺陷

1. **硬编码开发者路径**：`web-auth/SKILL.md` 和 `web-browse/SKILL.md` 中所有命令示例都使用 `/Users/avifen/.agentsys/plugins/web-ctl/scripts/web-ctl.js`。→ 根本原因：安装时 transformer 对 Kiro 路径替换逻辑缺失，Codex 版本已有替换但 Kiro 版本未跟进——两套安装路径分叉了。→ 自查：我的 skill 中有没有绝对路径？运行 `grep -rn "/Users/\|/home/" ~/.claude/skills/`。

2. **元技能 1000 行过载**：`maintain-cross-platform/SKILL.md` 约 1000 行，是建议上限的两倍。→ 根本原因：作者把所有参考材料塞进了同一个文件，便于自维护但牺牲可读性。→ 自查：`wc -l` 检查自己最大的 SKILL.md。

3. **模糊量词残留**：`orchestrate-review/SKILL.md` 中 "typically indicates"；`repo-intel/SKILL.md` 中 "for better analysis"。→ 根本原因：在优秀的技能中，作者放松了警惕，认为"显然能理解"。→ 自查：`grep -rn "typically\|better\|appropriate" ~/.claude/skills/`。

### 3.4 当时的优化机会

1. 为 `web-auth/SKILL.md` 和 `web-browse/SKILL.md` 添加 Kiro transformer 路径替换（PR 已提交）
2. 将 `maintain-cross-platform/SKILL.md` 的深度参考材料迁移至 `reference/` 子目录
3. 替换 "typically indicates" 为精确阈值（如"20+ 个文件跨模块变更时触发 cross-module review"）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| web-auth/SKILL.md 硬编码路径 | `ls .kiro/skills/web-auth/` | **已消灭**：整个 web-auth/ 目录不存在 | 作者可能将 web-auth/web-browse 迁移至另一仓库或彻底重构 |
| web-browse/SKILL.md 硬编码路径 | `ls .kiro/skills/web-browse/` | **已消灭**：同上 | 同上 |
| maintain-cross-platform 1023 行 | `wc -l meta/skills/maintain-cross-platform/SKILL.md` | **仍存在**：当前仍为 1023 行 | 作者未拆分；仍是已知技术债 |

### 4.2 架构演进

- **版本跳跃**：5.8.3 → 6.0.0，主版本号增加，表明有 breaking change
- **skill 数量**：`.kiro/skills/` 从 30 个降至 27 个，web-auth/web-browse 两个 Kiro skill 消失，另有变化
- **新文档层**：增加了 `agent-docs/` 目录（MULTI-AGENT-SYSTEMS-REFERENCE.md, AI-AGENT-ARCHITECTURE-RESEARCH.md, CLAUDE-CODE-REFERENCE.md）—— 作者开始系统化记录架构决策，说明项目在规模化
- **AGENTS.md 增加**：根目录和 `docs/reference/` 下均有 AGENTS.md，agent 索引文档化程度提升

### 4.3 新增的可学习模式

**`agent-docs/` 研究笔记目录**：将多 agent 系统参考资料、AI agent 架构研究和 Claude Code 参考文档单独归档，与 skill 代码分离。这是一种"知识侧车"（sidecar）模式——可执行的 skill 与参考学习材料分层管理，互不干扰。适合维护大型系统的团队。

---

## 五、校准

### 5.1 我已经在做对的

1. **`echo-sleuth-for-claude` 有完整 frontmatter**：agents/ 下的 recall.md、analyze.md 等均有 name/description，和 agentsys 的 100% 覆盖率一致。
2. **`drama-workshop-skills` 的 SKILL.md 置顶关键约束**：short-drama/SKILL.md 开头即有创作边界，类似 CLAUDE.md Critical Rules 置顶模式。
3. **`claude-for-legal` 的 skill 引用结构**：ip-legal/ 下的 skill 有 `## Examples` 节，做到了示例驱动。

### 5.2 挑战 / 验证

**挑战**：我之前认为"一个仓库只服务一个 AI 平台"是合理的。agentsys 通过 transformer 分发验证了"多平台共源"是可行的工程实践，不是额外复杂度——关键是把路径变换从 skill 内容中解耦出去。这改变了我对"多平台支持 = 多倍工作量"的预设。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的 skill 中有没有绝对路径（开发者机器特定）
grep -rn "/Users/\|/home/" /tmp/my-repos/MarkQWu-*/  2>/dev/null | grep -v ".git"

# 检查最大 SKILL.md 的行数
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" -exec wc -l {} \; 2>/dev/null | sort -n

# 检查 plugin.json 和 package.json 版本是否同步（如果有的话）
jq -r '.version' /tmp/my-repos/MarkQWu-*/plugin.json 2>/dev/null
```

命中绝对路径后：替换为变量或文档化"需要安装脚本注入"。
SKILL.md 超过 500 行：将参考材料提取至 `references/` 子目录。

### 6.2 灵感 → 实施路径

1. **想法**：为 `drama-workshop-skills` 设计显式编排管道（选题 → 策划 → 分镜 → 导出），类似 perf-\* pipeline。
   - **为何可行**：drama-workshop-skills 当前命令是平列的，用户需要自行知道顺序；pipeline 设计可以让 AI 自动推进下一步。
   - **第一步**：在 short-drama/SKILL.md 中新增 `## Pipeline` 节，声明阶段顺序和每阶段退出条件。预计 30 分钟。

2. **想法**：给 `echo-sleuth-for-claude` 添加 `agent-docs/` 类似的研究笔记侧车目录，记录设计决策。
   - **为何可行**：该项目是 xiaolai 插件生态的一部分，文档化架构决策有助于未来贡献者理解。
   - **第一步**：创建 `docs/design-decisions.md`，记录"为何选择 ~/.claude/projects/ 而非数据库"这类决策。预计 20 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 avifenesh/agentsys 的核心目的**：提供一套高质量、跨平台分发的 AI agent skill 套件，覆盖性能分析、代码 review、文档同步等开发者日常工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 都是面向特定场景的 skill 套件 | 垂直领域（短剧 vs 通用开发）；单平台 vs 多平台 | 高 |
| MarkQWu/claude-for-legal | 中 | 都是多 skill 的系统性套件 | 法律垂直 vs 开发工具；无 transformer 分发 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 agent + skill 组合 | 单一功能（会话挖掘）vs 综合工作流套件 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 硬编码绝对路径 | `grep -rn "/Users/\|/home/" /tmp/my-repos/MarkQWu-*/` | 未发现命中（0 处） | 无 |
| SKILL.md 行数超限（>500行） | `find /tmp/my-repos/ -name "SKILL.md" -exec wc -l {} \;` | drama-workshop-skills/short-drama/SKILL.md 较长，需人工确认 | 低 |
| 模糊量词（typically / better） | `grep -rn "typically\|better\|appropriate" /tmp/my-repos/MarkQWu-*/` | claude-for-legal 中有若干命中，集中在注释性描述中 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/claude-for-legal` 中模糊量词 → 找到对应 SKILL.md 行，将 "appropriate legal analysis" 改为具体描述，如"包含：事实摘要 + 3条相关先例 + 风险等级（低/中/高）"

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：多 skill 协同管道设计
   - **本案例做法**：8 个 perf-\* skills 通过共享 `docs/perf-requirements.md` 形成有序管道，每个 skill 明确输入/输出边界（`avifenesh-agentsys/.kiro/skills/perf-code-paths/SKILL.md` 等）
   - **我的项目现状**：`drama-workshop-skills` 的各命令（/开始、/策划、/分镜）是平列的，无显式管道契约
   - **如何借鉴**：在 short-drama/SKILL.md 新增 `## Phase Contract` 节，声明各阶段输出格式作为下一阶段的输入规范

2. **领域**：manifest 版本同步纪律
   - **本案例做法**：plugin.json 版本与 package.json 严格同步，`6.0.0 == 6.0.0`
   - **我的项目现状**：`claude-for-legal` 有 plugin.json，但版本同步是否有自动检查未知
   - **如何借鉴**：在 CI 或 pre-commit 中加 `jq -e '.version' plugin.json == package.json` 检查

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：法律垂直场景的 skill 中，`claude-for-legal` 包含完整的 `## Examples` 节（input/output 对）
- **我的做法**：`MarkQWu/claude-for-legal/ip-legal/skills/cease-desist/SKILL.md` 等有具体示例块
- **本案例做法**（弱在哪）：agentsys 的 skill 示例多为命令占位符，缺乏真实 input→output 对
- **意义**：示例驱动是 agentsys 可以改进的方向，也是我项目值得保持的优势

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才知道这个 skill 叫什么、怎么注册。缺少 frontmatter = 对 AI 工具不可见。

### <a name="transformer"></a>transformer（安装变换器）
> 安装脚本中的一段逻辑，在把 skill 文件"分发"到不同 AI 工具目录时，自动替换文件中的平台特定内容（如路径、命令格式）。类比：同一份菜谱，在中餐馆翻译成中文菜单，在西餐馆翻译成英文菜单，菜谱本身不变。

### <a name="manifest"></a>manifest（清单文件）
> 告诉系统"这个插件包含哪些组件"的配置文件，在 agentsys 中是 `.claude-plugin/plugin.json`。如果 manifest 里漏写了某个 skill 路径，那个 skill 即使在磁盘上存在也不会被加载。

### <a name="perf-pipeline"></a>perf-\* pipeline（性能分析管道）
> agentsys 中 8 个以 `perf-` 开头的 skill 的有序集合：code-paths → theory-gatherer → benchmarker → profiler → theory-tester → baseline-manager → analyzer → investigation-logger。每个 skill 负责性能分析流程的一个阶段，共同引用 `docs/perf-requirements.md` 作为不变式契约。
