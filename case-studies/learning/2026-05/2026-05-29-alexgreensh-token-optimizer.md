# alexgreensh/token-optimizer — 学习案例

**仓库**：https://github.com/alexgreensh/token-optimizer
**Stars**：646 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-29（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `examples-driven`, `manifest-discipline`, `vague-quantifier`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Token Optimizer 是一款 Claude Code / Codex 插件，专注于审计和优化 AI 代码助手的上下文窗口占用——找到"幽灵 token"并回收它们。作者 Alex Greenshpun 以 5.7.13 这个版本号表明该插件已高度成熟。插件的独特定位是**多客户端原生支持**：同一套逻辑同时适配 Claude Code（`.claude-plugin/`）、Codex（`.codex-plugin/`）、OpenClaw（`openclaw/`）和 opencode（`opencode/`）四个运行时，不是简单的"兼容"而是各有专属的 [manifest](#manifest) 入口点。

关键事实：
1. **版本 5.7.13**：高迭代速度，说明该工具在生产中被高频使用
2. **许可证**：PolyForm-Noncommercial-1.0.0，禁止商业使用，但允许个人和开源学习
3. **PRIVACY.md 存在**：明确声明零网络调用、零数据收集，是少数主动处理隐私透明度的插件之一
4. **30+ Python 脚本**：这是少见的"厚后端"插件，NL 层只是入口，真正的工作由 Python 完成

### 1.2 架构剖析

```
token-optimizer/
├── .claude-plugin/plugin.json       # Claude Code manifest
├── .codex-plugin/plugin.json        # Codex manifest
├── openclaw/                        # OpenClaw variant
├── opencode/                        # opencode variant
├── commands/
│   ├── health.md                    # 健康检查命令
│   └── quick.md                     # 快速优化命令
├── skills/
│   ├── token-optimizer/SKILL.md     # 核心技能（含 Python 脚本引用）
│   ├── fleet-auditor/SKILL.md       # 多项目审计
│   ├── token-coach/SKILL.md         # 教练技能
│   └── token-dashboard/SKILL.md     # 仪表盘
├── hooks/
│   ├── hooks.json                   # 10+ 事件注册
│   ├── python-launcher.sh           # hooks 启动器
│   └── run.py                       # hooks 主逻辑
└── skills/token-optimizer/scripts/  # 15+ Python 脚本（measure.py 等）
```

- **文件类型分布**：4 个 SKILL.md / 2 个 command / 1 个 hook config / 30+ Python 脚本
- **编排关系**：command（health/quick）→ SKILL.md（token-optimizer）→ references/ 子文件 → Python 脚本。SKILL.md 不内联详细提示词，而是用 `Read references/xxx.md` 将复杂 prompt 外包给引用文件
- **跨件契约**：fleet-auditor/SKILL.md 引用 `references/fleet-systems.md` 和 `references/waste-patterns.md`；token-coach/SKILL.md 引用 4 个参考文件 + 3 个示例文件；hooks.json → 6 个 Python 脚本。Audit 时全部引用均可解析

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**参考文件分离**——SKILL.md 只做流程控制，把所有"重型"内容（Prompt 模板、参数表格、示例输入输出）外包给 `references/` 子目录。这样 SKILL.md 保持简洁，同时不限制参考内容的体量
- **解决什么问题**：上下文窗口是 AI 编程助手的"内存"，当窗口被 CLAUDE.md / 无效 memory / 过多 skill 填满时，模型"忘性变大"或被迫触发 autocompact，导致体验变差甚至成本上升
- **Trade-off**：厚 Python 后端换来精准度量（毫秒级 token 计数），但对 NL artifact 的可移植性有代价——别的 AI 平台无法直接 fork 使用，必须先有 Python 运行时
- **多客户端设计**：作者明确面向整个 AI agent 生态编写，而不是绑定单一平台，这是一种"生态位最大化"思维

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「NL 编排层 + Python 度量核心」**：用 Markdown/SKILL.md 描述工作流和决策逻辑，把所有需要精确计算的工作（token 统计、diff 比对、结构扫描）交给 Python 脚本。NL 层不做计算，只做编排。

模式特征清单：
- **特征 1**：SKILL.md 中的 Phase 描述全部是"指令型"而非"实现型"，例如 `Run measure.py` 而不是把 Python 逻辑内联进 Markdown
- **特征 2**：参考文件（references/）作为"可插拔知识库"，NL 层只引用不重复
- **特征 3**：量化目标在 SKILL.md 第一段就出现："5-15% context recovery... up to 25%+"，告诉模型成功标准
- **特征 4**：多客户端通过独立 manifest 隔离，共享同一套 skills/
- **特征 5**：hooks 注册超过 10 个事件，覆盖整个 session 生命周期（SessionStart / PreCompact / PostToolUse 等）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要精准 token 度量的工具 | ✅ 高度适用 | Python 脚本可做精确计数，NL 无法独立完成 |
| 跨 AI 客户端发布的插件 | ✅ 适用 | 多 manifest 架构天然支持 |
| 纯文字处理 / 内容生成类 skill | ❌ 过度工程 | Python 后端是不必要的复杂度 |
| 快速原型 / 单人使用 | ❌ 不适用 | 维护 30+ Python 文件成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 编排 + Python 度量核心（本案例） | token-optimizer | 精准度高、跨客户端 | 依赖 Python 运行时、维护成本高 |
| 纯 NL skill | addyosmani/agent-skills | 零依赖、易移植 | 无法做精确计量 |
| NL 表皮 + 原生二进制 | 777genius/claude-notifications-go | 性能最高、跨平台 | 需要编译工具链 |

### 2.4 改进空间

1. **当前问题**：commands/health.md 和 commands/quick.md 缺少 `allowed-tools` [frontmatter](#frontmatter)，截止当前 HEAD 仍未修复 **改进做法**：在两个 command 文件中加入 `allowed-tools: [Bash, Read]` **预期收益**：符合 R11 最小权限原则，允许 Claude Code 在无提示的情况下执行

2. **当前问题**：skills/fleet-auditor/scripts/fleet.py 生成的仪表盘 HTML 硬编码了 Google Fonts CDN 链接，每次查看仪表盘都向 Google 发送请求 **改进做法**：将 JetBrains Mono 字体 woff2 文件本地化到 `assets/fonts/`，或改用系统字体栈 **预期收益**：符合 PRIVACY.md 的"零网络调用"承诺，减少信息泄露

3. **当前问题**：SKILL.md 中的 `references/` 引用路径假设固定安装位置，不同客户端的安装路径可能不同 **改进做法**：通过 `$CLAUDE_PLUGIN_ROOT` 或运行时检测变量动态解析路径 **预期收益**：提高跨环境可靠性

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **98/100**。这是本批次中得分最高的仓库。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/health.md | 95 | 缺少 allowed-tools (-5) |
| commands/quick.md | 93 | 缺少 allowed-tools (-5)、模糊量词 "some bloat" (-2) |
| skills/token-optimizer/SKILL.md | 96 | 模糊量词 "Some optimizations"、"appropriate models" (-4) |
| .claude-plugin/plugin.json | 100 | 无问题 |
| skills/fleet-auditor/SKILL.md | 100 | 无问题 |
| skills/token-coach/SKILL.md | 100 | 无问题 |
| skills/token-dashboard/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **量化目标模式** → SKILL.md 开头就给出"5-15% 回收率... 最高 25%+"的具体数字，让模型有明确的成功判断标准 → `skills/token-optimizer/SKILL.md` 第 1 行 → 在我的技能里，把"效果描述"从定性改为定量
2. **参考文件委托模式** → SKILL.md 不直接包含大段 prompt，而是 `Read references/xxx.md`，保持主文件简洁 → `skills/token-optimizer/SKILL.md` Phase 0-3 → 适合 skill 体积较大时的模块化设计
3. **并行 Agent 声明** → Phase 1 明确给出 6 个 agent 并行表格（文件、输出路径、模型、任务），不模糊地说"分析一下" → `skills/token-optimizer/SKILL.md` Phase 1 → 任何需要并行工作的 skill 都应该用这种显式表格
4. **PRIVACY.md 主动透明** → 提前声明插件的数据政策，是建立用户信任的重要工件 → `.github/CLA.md`、`PRIVACY.md` → 我的插件发布前也应该有 PRIVACY.md

### 3.3 当时的缺陷

1. **commands 缺 `allowed-tools`** → 问题根因：command 文件在编写时关注的是"做什么"，忽略了"用什么工具做"的声明。没有 allowed-tools，Claude Code 无法执行 [最小权限](#最小权限) 策略 → 自查：我的 echo-sleuth 的 commands 全部有 allowed-tools（从 frontmatter 检查结果看 HAS_FM 全部为 true），这一点我做得比这个仓库好

2. **模糊量词** → SKILL.md 出现"Some optimizations have side effects"（哪些？）和"appropriate models"（什么叫 appropriate？）→ 问题根因：描述性语言习惯于用形容词，没有意识到 LLM 需要可执行的边界条件 → 自查：drama-workshop-skills 中的 SKILL.md 是否也有类似措辞？需要 grep 确认

3. **Google Fonts CDN 硬编码** → 在生成的仪表盘 HTML 里引用 `fonts.googleapis.com`，违背了 PRIVACY.md 所声明的"零网络调用" → 问题根因：动态生成 HTML 时 CDN 引用是"默认好看方案"，没有意识到与 privacy 承诺的矛盾 → 自查：我的插件有没有在运行时生成 HTML 或动态内容？如果有，要检查外链

### 3.4 当时的优化机会

1. commands 中加入 `allowed-tools: [Bash]` 两行，成本极低，收益明显
2. `skills/token-optimizer/SKILL.md` 中"Some optimizations"改为明确枚举：把下方已有的列表（analyze/compress/archive）提前到那句话前面
3. 生成仪表盘时改用本地 base64 内联字体或系统 sans-serif

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| commands 缺 `allowed-tools` | `grep -l "allowed-tools" commands/*.md` | **仍存在**（grep 无结果） | 作者优先级不高，认为可选 |
| SKILL.md 中"Some optimizations"模糊量词 | `grep -n "Some" skills/token-optimizer/SKILL.md` | **已修复**（grep 无命中） | 说明作者接受了该类建议 |
| 无 PRIVACY.md | `ls PRIVACY.md` | **已修复**（文件存在） | 主动透明度提升 |

### 4.2 架构演进

当前 HEAD 相比 audit 时多了 `openclaw/`（OpenClaw 客户端支持）和 `opencode/`（opencode 支持），说明作者在向多客户端方向持续扩展。`signatures/cla.json` 和 `.github/CLA.md` 的出现说明项目已开始接收外部贡献，成熟度提升。

### 4.3 新增的可学习模式

**多 manifest 多客户端模式**：`.claude-plugin/`、`.codex-plugin/`、`openclaw/`、`opencode/` 四个目录各有独立 manifest，共享同一套 skills。这是一种"多平台发布"的组织方式，在 audit 时只有前两个，现在扩展为四个，反映了 AI 生态多元化趋势。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 全部有 frontmatter**：echo-sleuth 的所有 command 文件（recap.md、timeline.md 等）frontmatter 完整，优于 token-optimizer
2. **agents 有 examples**：memory-auditor.md 中有两个完整的 `<example>` 块，包含 Context / user / assistant 三段格式
3. **SKILL.md 有明确触发条件**：experience-synthesis 的 description 直接列出触发短语（"learn from past sessions"、"extract lessons" 等）
4. **空输入处理**：recall.md 有明确的 "If zero sessions are found, report..." 空状态处理

### 5.2 挑战 / 验证

**验证**：量化目标确实有效。token-optimizer SKILL.md 开头的"5-15%"和"25%+"这两个数字告诉模型什么叫"成功"，比我的 SKILL.md 里"improve token efficiency"这种描述更精准。今后写 SKILL 时应该先问自己：**"成功是什么样的？能给出数字吗？"**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 是否有量化成功标准
grep -rn "%" /tmp/my-repos/MarkQWu-*/skills/**/*.md 2>/dev/null
grep -rn "success\|succeed\|完成" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/**/*.md 2>/dev/null
```
命中后怎么办：如果某个 skill 没有量化目标，加一行具体的成功描述，例如把"summarize sessions"改为"produce a summary under 500 words per session"。

```bash
# 检查我的 commands 是否都有 allowed-tools
grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
```
命中后怎么办：为命中的 command 添加 `allowed-tools:` frontmatter 字段，列出实际使用的工具。

### 6.2 灵感 → 实施路径

1. **想法**：drama-workshop-skills 的 SKILL.md 应该也有 `references/` 子目录，把大段的"规则知识"（爽点矩阵、出海适配规则等）从 SKILL.md 主体移出去
   - **为何可行**：token-optimizer 证明这个模式在大型 skill 中有效，drama-workshop 的 SKILL.md 目前较长
   - **第一步**：创建 `short-drama/references/` 目录，把现有的 ai-live-rules.md 等文件整理进去，在 SKILL.md 中改用 `Read references/ai-live-rules.md`，预计 30 分钟

2. **想法**：echo-sleuth 加 PRIVACY.md
   - **为何可行**：插件读取用户的 `~/.claude/projects/` 历史，涉及隐私，PRIVACY.md 应该明确说明不上传任何数据
   - **第一步**：参考 token-optimizer 的 PRIVACY.md 结构，写一个类似文件，约 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 alexgreensh/token-optimizer 的核心目的**：审计并优化 Claude Code 的上下文窗口占用，追踪"幽灵 token"来源

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是辅助 Claude Code 自我管理的 meta 工具；都有 Python 脚本；都关注 session/memory | token-optimizer 关注 token 消耗，echo-sleuth 关注历史知识提取 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code skill | 完全不同域（创作 vs 工具） | 低 |
| MarkQWu/claude-for-legal | 低 | 同为 skill 集合 | 领域和规模都不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词（"Some"、"appropriate"） | `grep -rn "Some\|appropriate\|comprehensive" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/**/*.md` | 未命中——experience-synthesis 的 SKILL.md 无这类词 | N/A |
| commands 缺 allowed-tools | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | 未命中——所有 command 均有 frontmatter 且含 allowed-tools | N/A |

**结论**：echo-sleuth 在这两个具体问题上比 token-optimizer 做得更好。

### 8.3 别人的更优方案

1. **领域**：量化 SKILL 成功指标
   - **本案例做法**：SKILL.md 第一段写明 "Target: 5-15% context recovery... up to 25%+" (`skills/token-optimizer/SKILL.md` 第 11 行)
   - **我的项目现状**：echo-sleuth 的 experience-synthesis 描述效果为 "surface values, patterns, mistakes, and decisions worth remembering"，是定性描述
   - **如何借鉴**：在 SKILL.md 的成功描述中加入具体数字，例如 "extract 3-10 reusable insights per session, covering at least values, decisions, and mistakes categories"

2. **领域**：多 manifest 多平台发布
   - **本案例做法**：一个 skills/ 目录，四套独立 manifest（.claude-plugin、.codex-plugin、openclaw、opencode）
   - **我的项目现状**：echo-sleuth 只有 `.claude-plugin/`，不支持 Codex 等平台
   - **如何借鉴**：添加 `.codex-plugin/plugin.json`，script paths 可复用，约 15 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：commands 完整度
- **我的做法**：MarkQWu/echo-sleuth-for-claude 所有 command 文件均有 frontmatter（name、description、allowed-tools 均完整）
- **本案例做法**：token-optimizer 的两个 commands 截至当前 HEAD 仍缺 `allowed-tools`
- **意义**：这说明我的插件在 R11 合规性上达到了比 646 星仓库更高的标准；如果未来在社区被审查，这是一个可展示的优势

---

## 八、术语表

### <a name="manifest"></a>manifest
> Markdown 或 JSON 格式的"清单文件"，告诉运行时（Claude Code / Codex 等）这个插件包含哪些组件（skill、command、agent）。`plugin.json` 是 Claude Code 插件的 manifest——里面列出所有组件的路径。如果 manifest 里漏写了某个文件，那个文件即使存在也不会被加载。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`allowed-tools`）。Claude Code 读 command 或 SKILL.md 时先解析 frontmatter 才能注册和调用该组件。

### <a name="最小权限"></a>最小权限
> 安全原则：任何组件只申请它实际需要的权限，不多申请。在 Claude Code 中体现为 `allowed-tools` 字段——只声明这个 command 真正会用到的工具。声明得越具体，越不容易出现意外的工具调用。
