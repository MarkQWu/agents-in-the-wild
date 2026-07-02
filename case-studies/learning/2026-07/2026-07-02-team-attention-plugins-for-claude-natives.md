# team-attention/plugins-for-claude-natives — 学习案例

**仓库**：https://github.com/team-attention/plugins-for-claude-natives
**Stars**：729 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-02（基于当前 HEAD）
**主题标签**：`monorepo-vs-split`, `examples-driven`, `manifest-discipline`, `security-gate`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
**team-attention/plugins-for-claude-natives** 是一个 Claude Code 插件集合仓库，面向「Claude 重度用户」，通过统一的插件市场（`/plugin marketplace add team-attention/plugins-for-claude-natives`）分发 14 个独立插件，覆盖会话管理、AI 增强、内容处理、通信集成、开发者工具五大领域。

关键事实：
- 创建者为 team-attention 组织，运营「为 Claude 原生用户服务」的插件生态
- 每个插件独立目录，可单独安装，不相互依赖
- 集成了 Google 服务（Gmail、Calendar）、macOS 原生 API（KakaoTalk 无障碍 API）、外部 AI（Gemini、GPT、Codex）等异构系统
- MIT 协议，安全扫描结果为 CLEAR（无 Critical/High 级别问题）

### 1.2 架构剖析
- **目录结构**：
  ```
  plugins-for-claude-natives/
  ├── .claude-plugin/plugin.json    # 根 manifest（入口）
  ├── plugins/
  │   ├── agent-council/           # 多模型并行咨询
  │   ├── dev/                     # 4-子 agent 技术决策
  │   ├── session-wrap/            # 5-agent 会话归档流水线
  │   ├── gmail/ google-calendar/  # Google 集成
  │   ├── youtube-digest/ podcast/ # 内容处理
  │   ├── doubt/ clarify/          # AI 辅助思考
  │   ├── interactive-review/      # MCP 辅助审查
  │   ├── say-summary/             # macOS 语音摘要
  │   ├── kakaotalk/               # KakaoTalk 集成
  │   └── team-assemble/           # 动态专家团队
  └── （根级脚本和文档）
  ```
- **文件类型分布**：46 个 NL [工件](#工件)（8 个 plugin.json、9 个 skill、9 个 agent、2 个 hook config、多个 CLAUDE.md）
- **编排关系**：`dev/skills/tech-decision/SKILL.md` 是最复杂节点，编排 4 个并行子 agent（`codebase-explorer`、`docs-researcher`、`tradeoff-analyzer`、`decision-synthesizer`）+ 2 个外部 skill（`dev-scan`、`agent-council`）；`session-wrap` 是 5-agent 流水线（4 并行 + 1 顺序）；其余插件相互独立
- **跨件契约**：`dev` 插件跨插件调用 `agent-council` skill（按安装名引用），是唯一的跨插件 skill 调用；`docs-researcher` 声明了外部 MCP 工具（`mcp__context7__*`），但未在插件内声明先决条件

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「插件即原子单元」—— 每个插件完全自包含（拥有独立的 plugin.json、skills、agents、hooks），最小化跨插件耦合
- **解决什么问题**：Claude Code 默认能力对于高频、场景化任务（每日会话总结、多模型投票决策、实时内容处理）过于通用；作者通过专用插件将这些工作流固化
- **做了什么 trade-off**：选择 monorepo + 多 plugin.json 而非多独立仓库，牺牲了每个插件的独立版本管理，但换来了统一安装、统一发现和跨插件 skill 调用的便利
- **反映什么认知模型**：作者相信「单职责 skill + 独立 plugin 边界」比大而全的设计更可维护；多 agent 并行是该仓库的显著偏好（agent-council、dev、session-wrap 均使用并行模式）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「Monorepo 插件超市模式」（Multi-Plugin Monorepo Marketplace）**

作者将多个主题无关的插件放在同一个仓库中，用一个根级 [manifest](#manifest) 作为入口，每个子目录是一个独立插件（带自己的 plugin.json）。

模式特征清单：
- 特征 1：根 manifest 作为「菜单」，不含具体功能实现
- 特征 2：每个子插件目录结构自治（独立 plugin.json + skills + agents + hooks）
- 特征 3：跨插件 skill 调用依赖安装名而非路径（`agent-council` 被 `dev` 调用）
- 特征 4：质量分布不均——同一仓库中文件评分跨度达 30 分（70 ～ 100）
- 特征 5：外部依赖（Google OAuth、macOS Accessibility API、OpenAI TTS）散布在各插件，无统一声明

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 作者想同时维护多个主题不同的插件 | ✅ 高度适用 | 一个仓库管理所有插件，安装体验统一 |
| 插件间需要频繁共享 skill | ✅ 适用 | 跨插件 skill 调用比多仓库依赖管理更简单 |
| 插件需要独立版本发布和 changelog | ❌ 不适用 | Monorepo 没有 per-plugin 版本控制机制 |
| 团队成员分别维护不同插件 | ❌ 谨慎 | 权限边界模糊，一个 PR 可能影响多个插件 |
| 插件规模超过 20 个 | ❌ 不适用 | 根 manifest 会变得笨重，discovery 困难 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Monorepo 插件超市（本仓库） | team-attention/plugins-for-claude-natives | 统一安装、跨插件调用简单 | 版本管理、权限边界混淆 |
| 单一插件专属仓库 | lackeyjb/playwright-skill | 独立发布、明确 scope | 多个仓库运维成本高 |
| 单 plugin.json 平铺所有 skill | gstack（MarkQWu） | 最简单，无命名空间复杂度 | 随 skill 增多难以组织 |

### 2.4 改进空间
1. **当前问题**：外部依赖（Context7 MCP、Google OAuth、macOS API）散布各插件，用户安装后才发现缺依赖。**改进做法**：在每个 plugin.json 增加 `prerequisites` 数组，列出环境要求。**预期收益**：减少安装后失效的用户体验问题。
2. **当前问题**：session-wrap 的 5 个 agent 文件全部缺少 `<example>` 块，且 `duplicate-checker` 使用 haiku 做需要推理的 Layer 4 评估。**改进做法**：每个 agent 至少添加 1 个 `<example>` 块展示 input→output，并将 `duplicate-checker` model 升级为 sonnet。**预期收益**：评分从 77 升至 90+，重复检测误报率降低。
3. **当前问题**：根级 plugin.json 缺少 `license` 和 `keywords`，google-calendar/gmail 的 plugin.json 缺少 4 个 manifest 字段。**改进做法**：参照 session-wrap、agent-council 等 100 分 plugin.json 补全所有必填字段。**预期收益**：manifest 扫描工具能正确索引这些插件。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**（加权平均，46 个工件）。

| 文件（代表性） | 当时分数 | 主要问题 |
|---|---|---|
| plugins/google-calendar/.claude-plugin/plugin.json | 70 | 缺 author/repository/license/keywords（−30） |
| plugins/session-wrap/agents/followup-suggester.md | 77 | 零 example 块（−15），模糊量词（−12） |
| plugins/interactive-review/CLAUDE.md | 90 | 文件路径引用错误 |
| plugins/say-summary/hooks/hooks.json | 90 | Stop hook 缺 matcher 字段 |
| plugins/session-wrap/.claude-plugin/plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **分层 agent 编排** → tech-decision/SKILL.md 的 4-phase pipeline（并行收集 → 并行分析 → 汇总 → 决策）是教科书级多 agent 编排；根本原因：每个 agent 单职责且输出结构化。如何借鉴：设计多 agent 任务时先画出 DAG（有向无环图），明确哪些阶段可并行。

2. **模型选型意识** → session-wrap 的 duplicate-checker 使用 haiku（成本优化），其余 agent 使用 sonnet；根本原因：作者有意识地按任务复杂度匹配模型。如何借鉴：机械搜索/去重用 haiku，需要推理的评估/决策用 sonnet。

3. **100 分 plugin.json 模式** → session-wrap、agent-council 等多个 plugin.json 得 100 分，其共同特征是：`name`、`description`、`version`、`author`、`repository`、`license`、`keywords` 全部存在。如何借鉴：以 session-wrap 的 plugin.json 作为模板复制到新插件。

4. **verify-script 机制** → hooks 的 `doubt-detector.sh` 和 `doubt-validator.sh` 实现了基于状态文件的会话级「怀疑模式」；概念是正确的（用文件系统做持久状态），但实现有安全漏洞。如何借鉴：状态文件路径必须经过 sanitize（见 §3.3）。

5. **跨插件 skill 引用** → dev/docs-researcher.md 直接引用安装后可用的 `agent-council` skill，比复制粘贴内容更优雅。如何借鉴：自己的插件中提炼可复用的 skill，让其他插件按名引用。

### 3.3 当时的缺陷

1. **Bug #1 — CLAUDE.md 路径错误**（`interactive-review/CLAUDE.md`）→ 目录树显示 `skills/review.md`，实际路径是 `skills/review/SKILL.md`。**根本原因**：作者将 skill 从单文件重组为子目录后，忘记更新文档；CLAUDE.md 和代码未在同一 PR 中修改。**自查**：我的 CLAUDE.md 中所有文件路径引用是否也可能过时？

2. **Bug #2 — orphaned SKILL.md**（`agent-council/SKILL.md` 根级 vs `skills/agent-council/SKILL.md`）→ 两个内容相同的文件，plugin.json 只注册了嵌套版本，根级版本成为游离文件。**根本原因**：重构时只改了 plugin.json 指向，没有删除旧文件。**自查**：重组目录后是否总是 git rm 掉旧位置的文件？

3. **中级安全 — session_id 路径穿越**（`doubt-detector.sh`、`doubt-validator.sh`）→ 从不可信 JSON stdin 提取的 `session_id` 未经过滤直接拼接文件路径，`rm -f` 操作在污染路径上执行。**根本原因**：shell 脚本接受外部输入时缺乏「永远不信任输入」的意识。**自查**：我的 hook 脚本中是否有类似的未 sanitize 输入？

### 3.4 当时的优化机会

1. **session-wrap 的 5 个 agent 全部缺 `<example>` 块**（Quality Issue #1–5）：这是仓库中最集中的质量短板，每个 agent 扣 15 分。添加 1 个 input→output 示例可将每个 agent 从 77 提升至 92+。
2. **`baseDir` 未定义变量**（`session-analyzer/SKILL.md`）：skill 文件中引用了 `${baseDir}/scripts/...` 但该变量从未定义，运行时必然失败。正确做法是用 `$SKILL_DIR` 或在 skill 中写死脚本的绝对路径。
3. **docs-researcher.md 缺外部 MCP 声明**：声明了 `mcp__context7__*` 工具但没有任何先决条件说明，用户安装后遇到静默降级。应在 agent 文件顶部加 `> 前提：需要安装 Context7 MCP 服务器`。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：CLAUDE.md 路径错误（`skills/review.md`） | WebFetch interactive-review/CLAUDE.md，搜索 `review.md` | **依然存在** — 文件至今显示 `└── review.md` | 维护者 3 个月内未修复此文档 bug，可能是该文件很少被阅读 |
| session-wrap agents 零 example | WebFetch followup-suggester.md，检查 `<example>` | **部分改善** — agent 现有完整 frontmatter（name/description/model/color），但仍无 `<example>` 块，只有格式模板 | frontmatter 补齐说明维护者有持续改进意识，但 example 这个最难写的部分仍未完成 |
| session_id 路径穿越 | （无法直接验证，需读 doubt-detector.sh） | 无法验证（WebFetch 未覆盖此文件） | 安全问题修复通常优先级低于功能开发 |

### 4.2 架构演进
从 audit（2026-04-06）到当前 HEAD，session-wrap agents 已补充 [frontmatter](#frontmatter) 字段（`name`、`description`、`tools`、`model`、`color`），这说明作者意识到了 agent 元数据的重要性，但 example 驱动的质量提升尚未完成。其他结构性变化不可见（无法 clone）。

### 4.3 新增的可学习模式
agent frontmatter 扩充了 `color` 字段（如 `color: cyan`）—— 这是 Claude Code 界面中 agent 的视觉标识，帮助用户在多 agent 并行时区分输出来源。值得在自己的 agent 文件中也加上。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责 plugin** — 我的 bureau/gstack 都是独立仓库，没有把不相关功能塞进同一仓库，比 team-attention 的 monorepo 边界更清晰
2. **manifest 完整性** — bureau 的命令文件有 `description` 和 `argument-hint` 字段，符合 plugin.json 规范意识
3. **output format 明确** — bureau/query.md 有完整的 Output Format 节，比 session-wrap agents 的格式模板更正式
4. **避免跨插件隐式依赖** — 我的仓库之间不存在「调用对方的 skill」的隐式依赖，比 dev→agent-council 的跨插件调用更安全

### 5.2 挑战 / 验证
**挑战了我的假设**：我之前认为「monorepo 是插件开发的自然选择」，但这个仓库展示了 monorepo 的副作用 — 14 个插件的质量差异极大（70～100 分），而且旧文档 bug 因为「量太多」而长期未被修复。这验证了「小而精的单仓比大而全的 monorepo 更容易保证质量」的判断。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 CLAUDE.md 中的文件路径是否都实际存在
grep -rn '\.\/' ~/.claude/commands/ ~/.claude/skills/ 2>/dev/null | \
  grep -E '\.(md|json|sh|py)' | \
  awk -F: '{print $3}' | \
  grep -oE '\./[a-zA-Z0-9/_.-]+' | \
  while read path; do [ ! -f "$path" ] && echo "BROKEN: $path"; done
```
命中后怎么办：更新 CLAUDE.md 中的路径，或删除指向不存在文件的引用。

```bash
# 检查我的 agent 文件是否有至少 1 个 <example> 块
find ~/.claude/ -name "*.md" -path "*/agents/*" | \
  while read f; do
    grep -q '<example>' "$f" || echo "NO EXAMPLE: $f"
  done
```
命中后怎么办：为每个缺 example 的 agent 文件补充 1 个 `<example>` ... `</example>` 块，包含一次真实的输入→输出对。

```bash
# 检查 hook 脚本是否对外部输入做了 sanitize
grep -rn 'jq -r' ~/.claude/hooks/*.sh 2>/dev/null | grep -v '| tr -cd' | grep -v 'allowlist'
```
命中后怎么办：在每个从 JSON stdin 提取字符串后立即加 `| tr -cd '[:alnum:]-_.'` 过滤非法字符。

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 的复杂 command（如 /review）添加多 agent 并行架构，参照 tech-decision 的 4-phase pipeline
   - **为何可行**：/review 目前是单步执行，拆分为「代码探索 agent」+「风险分析 agent」+「建议综合 agent」可提高审查深度
   - **第一步**：在 gstack 的 commands/ 下新建 agents/ 子目录，把 review 的各阶段分离为独立 agent 文件，15 分钟可完成目录重组

2. **想法**：在我所有 agent 文件中加 `color:` 字段
   - **为何可行**：Claude Code 支持 agent 颜色标识，在多 agent 并行时输出来源一目了然
   - **第一步**：打开 bureau 的所有 agent 文件，逐个加 `color: cyan|purple|green|yellow` 等字段，5 分钟

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 team-attention/plugins-for-claude-natives 的核心目的**：为 Claude 重度用户提供场景化 AI 增强插件集（会话管理、多模型决策、第三方集成）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多命令 Claude Code 插件集合，都面向提升日常工作效率 | gstack 是单 plugin（命令式工作流），team-attention 是多 plugin（场景化集成） | 高 |
| MarkQWu/bureau | 中 | 都有多 agent 编排（bureau 的 capture/compile/review） | bureau 专注知识管理，team-attention 覆盖更广 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都涉及会话数据处理 | echo-sleuth 是会话挖掘（过去），session-wrap 是实时归档（当下） | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 文件缺 `<example>` 块 | `find ~/.claude/ -name "*.md" -path "*/agents/*"` 然后检查有无 `<example>` | bureau 的 query 命令和 agent 文件无法直接验证，但 bureau/query.md 已确认缺少 example | 高 |
| CLAUDE.md 中路径引用过时 | `grep -rn 'skills/.*\.md' CLAUDE.md` 然后验证路径是否存在 | 无法在本次运行验证（无法读取用户仓库），风险：有目录重组历史的仓库更易出现 | 中 |

**命中后的具体行动建议**：
- MarkQWu/gstack 的 commands/ 目录 → 为每个 command 文件检查是否有使用示例，无则按 `<example>` 格式添加 1 个，每个 10 分钟

### 7.3 别人的更优方案

1. **领域**：多 agent 编排可视化（agent frontmatter 的 `color` 字段）
   - **本案例做法**：session-wrap agents 每个都有 `color:` 字段（cyan、purple 等），Claude Code 界面中多 agent 并行时颜色区分
   - **我的项目现状**：MarkQWu/bureau 的 agent 文件中无 `color` 字段（从 bureau/query.md 结构推断）
   - **如何借鉴**：在每个 agent 文件的 frontmatter 加 `color: <color>` 字段，5 分钟搞定全部

2. **领域**：跨插件 skill 复用（dev→agent-council 引用）
   - **本案例做法**：`dev/docs-researcher.md` 直接在 `tools:` 中引用 `agent-council` skill，避免复制粘贴
   - **我的项目现状**：gstack 的命令之间无交叉引用，可能有重复逻辑
   - **如何借鉴**：识别 gstack 中多个 command 共用的逻辑（如「代码探索」），提取为独立 skill，其他 command 引用它

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：仓库边界清晰（单一职责）
- **我的做法**：MarkQWu/gstack、MarkQWu/bureau 各是独立仓库，每个仓库只做一件事
- **本案例做法（弱在哪）**：14 个主题差异极大的插件塞在同一仓库，导致质量参差不齐（70～100 分的跨度），维护者难以统一提升
- **意义**：单仓单职责是保证 NL 工件质量一致性的关键设计决策

---

## 八、术语表

### <a name="工件"></a>工件（NL artifact）
> Claude Code 生态中被 NLPM 扫描和评分的自然语言文件，包括 `SKILL.md`（技能说明）、agent 指令文件（`.md`）、`CLAUDE.md`（项目指令）、`plugin.json`（插件清单）、`hooks.json`（钩子配置）等。它们是「用自然语言写的程序」，不是给人看的文档，而是给 AI 执行的指令。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`、`tools`、`color` 等）。Claude Code 读 agent/SKILL.md 时先解析 frontmatter 才知道这个文件是什么、怎么注册和调用。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="orphaned-file"></a>游离文件（orphaned file）
> 存在于磁盘上、但没有被任何 manifest 或引用链接所指向的文件。就像图书馆里一本没有编入目录的书——物理上存在，但没人能通过正常途径找到它。这类文件会随着时间推移成为「版本漂移」的温床，因为没人有动力更新它。
