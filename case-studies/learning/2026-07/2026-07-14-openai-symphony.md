# openai/symphony — 学习案例

**仓库**：https://github.com/openai/symphony
**Stars**：25,947 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-05（历史快照）| **生成日期**：2026-07-14（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `template-design`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Symphony 是 openai 开源的[自主 AI 代码实现系统](#自主实现系统)，专为 Codex CLI 设计，将项目工作拆解成隔离的自主运行单元，让团队管理任务而非监督 AI 编码。2026 年 2 月发布，以 Elixir 为核心后端语言，已获 25,947 颗星，是目前 OpenAI 在代码执行层最重要的开源工程之一。

- **创建时间**：2026-02-26
- **核心语言**：Elixir（后端）+ Python（PR 监控脚本）+ Bash（初始化）
- **获取方式**：GitHub 直接克隆，通过 Codex CLI 使用 `.codex/skills/` 目录中的 skill
- **生态位置**：与 Claude Code 的 `.claude/skills/` 平行，属于 OpenAI Codex 生态的 NL skill 层

### 1.2 架构剖析
- **目录结构**：
  ```
  openai/symphony
  ├── .codex/
  │   ├── skills/
  │   │   ├── commit/SKILL.md   ← 100分参考实现
  │   │   ├── debug/SKILL.md
  │   │   ├── land/SKILL.md     ← 最复杂（含 land_watch.py）
  │   │   ├── linear/SKILL.md
  │   │   ├── pull/SKILL.md
  │   │   └── push/SKILL.md
  │   └── worktree_init.sh
  └── elixir/                   ← 核心后端
      ├── AGENTS.md
      └── ...
  ```
- **文件类型分布**：6 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：`land/SKILL.md` 是编排核心，引用其他 5 个 skill（`commit`、`pull`、`push`），加上一个 Python 辅助脚本 `land_watch.py` 监控 PR 状态。各 skill 之间是**顺序依赖**关系，不是平列。
- **跨件契约**：skill 之间通过文件名相互引用（如 `land` 中 `"see .codex/skills/commit"`），依赖 `elixir/AGENTS.md` 声明项目级约定，依赖 `.github/pull_request_template.md` 结构化 PR。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一个 skill 做一件事，做到极致」。`commit/SKILL.md` 是标杆，包含 `## Goals`、`## Inputs`、`## Steps`（编号）、`## Output`、`## Template` 五段完整结构，100 分。
- **解决什么问题**：AI agent 执行代码任务时行为不稳定、输出格式不可预测。Symphony 通过标准化 skill [frontmatter](#frontmatter) 和完整的输出声明，让 AI 的行为边界清晰。
- **Trade-off**：6 个核心 skill 全部手工精写，不依赖生成器，保证质量但限制了扩展速度。
- **认知模型**：作者把 AI skill 视为「契约文档」——每个 skill 必须明确告诉 AI 它的输入是什么、输出是什么，而不仅仅是步骤列表。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「参考实现锚定式」** — 单一高分文件作为所有其他文件的规范参照；新 skill 被要求对齐这个参考实现的结构。

模式特征清单：
- 特征 1：存在一个「100 分参考实现」（`commit/SKILL.md`），显式承担规范角色
- 特征 2：`## Goals / Inputs / Steps / Output / Template` 五段结构为强约定
- 特征 3：skill 数量极少（6 个），每个 skill 独立覆盖一个工作流节点
- 特征 4：Land skill 是超级编排者，引用其他所有 skill，形成单向依赖树
- 特征 5：辅助脚本（`land_watch.py`）与 skill 配对存放，明确 skill 的外部依赖

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 核心工程工作流（commit、PR、merge） | ✅ 高度适用 | 流程固定，输出可验证，值得精写 |
| 快速扩展到 100+ skill | ❌ 不适用 | 手工精写成本高，质量难以持续 |
| 跨团队共享的 skill 库 | ✅ 适用 | 少而精的 skill 更容易评审和维护 |
| 探索性/一次性任务 | ❌ 不适用 | 投入产出比低，不值得精写 skill |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 参考实现锚定式（本仓库） | openai/symphony | 质量高、内聚、易评审 | 扩展慢、skill 数量受限 |
| 平铺多 skill | agent-sh/agentsys | 覆盖场景广（32 个 skill） | 质量参差不齐，示例缺失 |
| 多 plugin 聚合式 | ccplugins/awesome-claude-code-plugins | 功能丰富（100+ plugin） | 大量 bug（name mismatch、截断描述） |

### 2.4 改进空间
1. **当前问题**：5 个 skill 缺少 `## Output` 声明 **改进做法**：为每个 skill 添加正式输出声明（格式：「该 skill 产出什么，调用方应检查哪些字段」） **预期收益**：减少 AI 输出歧义，特别是 `debug/SKILL.md` 目前没有指定应该产出什么样的调试报告
2. **当前问题**：`pull/SKILL.md` 第 33 行的 `AGENTS.md` 引用没有路径前缀，从仓库根执行时找不到文件 **改进做法**：改为 `elixir/AGENTS.md` **预期收益**：防止 AI agent 在 workspace 根目录执行时找错文件
3. **当前问题**：`land/SKILL.md` 的「Use judgment」没有定义判断标准 **改进做法**：将模糊标准替换为可观察的信号（如「如果错误是 timeout 类型且重跑后不复现，则视为 flaky」）**预期收益**：让 AI 不依赖主观判断，行为更可预测

---

## 三、过去审查发现（2026-05-05 历史快���）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-05 当时得分 **89/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commit/SKILL.md | 100 | 无问题，参考实现 |
| linear/SKILL.md | 90 | 缺 `## Output` 节 |
| debug/SKILL.md | 88 | 缺 `## Output` 节 + vague "relevant" |
| push/SKILL.md | 86 | 缺 `## Output` 节 + vague "proper"/"clearly" |
| pull/SKILL.md | 86 | 缺 `## Output` 节 + vague "minimal"/"complex" |
| land/SKILL.md | 84 | 缺 `## Output` 节 + vague "judgment"/"brief"×2 |

### 3.2 当时值得借鉴的模式
1. **`commit/SKILL.md` 五段完整结构** → 为什么好：AI 能在任务开始前知道「我要完成什么（Goals）、有什么原材料（Inputs）、步骤是什么（Steps）、产出什么（Output）、格式模板是什么（Template）」，消除模糊。原文：`.codex/skills/commit/SKILL.md` → 借鉴方式：所有 skill 复制此结构，不省略任何节
2. **`land_watch.py` 使用 `asyncio.create_subprocess_exec` 代替 shell=True** → 为什么好：彻底消除 shell 注入风险。原文：`.codex/skills/land/land_watch.py` → 借鉴：凡是 Python 调用外部命令，优先 exec 而非 shell
3. **`sanitize_terminal_output` 过滤控制字符** → 为什么好：防止 PR comment 中的 ANSI 转义序列被当作命令执行。原文：`land_watch.py` 的 sanitize 函数 → 借鉴：任何处理外部输入（PR body、comment）的脚本都应加过滤层
4. **六个 skill 的 [frontmatter](#frontmatter) 全部有合法的 `name` 和 `description`** → 为什么好：零注册 bug。原文：`.codex/skills/*/SKILL.md` → 借鉴：每次新建 skill 先填写 frontmatter，最后写内容

### 3.3 当时的缺陷
1. **缺 `## Output` 声明**（5/6 个 skill）→ 为什么失败：AI 完成任务后不知道「产出什么格式的结果」，每次输出格式随机，调用方无法依赖 → 自查：我的 gstack 有 ~50 个 skill，其中 80%+ 同样缺 `## Output`，是高风险重叠点
2. **模糊量词**（`judgment`、`brief`、`minimal`、`complex`、`proper`、`clearly`）→ 为什么失败：这些词在 AI 解释时会变成「主观判断」，导致每次行为不一致。特别是「Use judgment to identify flaky failures」完全没有可观察的标准 → 自查：gstack skills 也有 1-2 处模糊词
3. **`pull/SKILL.md` 的 `AGENTS.md` 无路径前缀** → 为什么失败：从 workspace 根执行时找不到文件，AI agent 会静默失败或产生错误 → 自查：我的 skill 文件中有类似不含相对路径的引用吗？

### 3.4 当时的优化机会
1. 为 5 个缺 Output 的 skill 各添加一段正式输出声明（预计每个 10-15 分钟）
2. 将所有模糊量词替换为可测量标准（参考 `commit/SKILL.md` 的表达方式）
3. 修正 `pull/SKILL.md` 第 33 行的 `AGENTS.md` → `elixir/AGENTS.md`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 5 个 skill 缺 `## Output` | `grep -c "^## Output" .codex/skills/*/SKILL.md` | **仍存在**：commit=1，其余 5 个仍为 0 | openai 未接受 NLPM 的修复建议 |
| `land/SKILL.md` 模糊量词 | `grep -n "judgment\|brief"` | **仍存在**：judgment@121，brief@210+213 | 原作者认为模糊标准在此场景可接受 |
| `pull/SKILL.md` AGENTS.md 无路径 | `grep -n "AGENTS.md" pull/SKILL.md` | **仍存在**：第 33 行无 `elixir/` 前缀 | 未修复 |

### 4.2 架构演进
从审计时（2026-05-05）到现在（2026-07-14），目录结构几乎无变化：仍是 6 个 skill，结构不变。最近一次 push 是 2026-06-09。说明 openai 把这个仓库视为**稳定参考实现**而非活跃迭代产品。

### 4.3 新增的可学习模式
暂无——仓库在这段时间内无明显新增设计。

---

## 五、校准

### 5.1 我已经在做对的
1. `MarkQWu/bureau` 的所有 skills 都有 3 个 `<example>` 块——比 openai/symphony 的大多数 skill 都做得更好
2. bureau 和 gstack 的 skill [frontmatter](#frontmatter) 有合法的 `name` 和 `description`，无注册 bug
3. gstack 的 `ios-design-review`、`qa`、`qa-only`、`scrape`、`make-pdf`、`design-review` 有 `## Output` 节——部分 skill 已经遵守

### 5.2 挑战 / 验证
- **挑战**：我以前认为 `## Output` 是「锦上添花」，但 openai/symphony 的缺陷显示，即使是 89 分的高质量仓库，缺这个节就会导致 AI 产出格式随机。这验证了 `## Output` 是必须项，不是可选项。
- **验证**：bureau 的 skills 全部有 `<example>` 块——这个坚持是对的，但很多 skill 缺 `## Output`，bureau 也受到同样的问题。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的 skill 是否有 ## Output 节
find ~/.claude/skills ~/gstack ~/bureau -name "SKILL.md" 2>/dev/null | while read f; do
  has_out=$(grep -c "^## Output" "$f")
  [ "$has_out" -eq 0 ] && echo "缺Output: $f"
done
```
命中后：为每个缺 `## Output` 的 skill 补一段：「该 skill 产出什么，格式是什么，调用方应检查哪些字段。」

```bash
# 检查模糊量词
grep -rn -E '\b(appropriate|judgment|brief|minimal|complex|proper|clearly|relevant)\b' \
  ~/*/skills/*/SKILL.md 2>/dev/null
```
命中后：将模糊词替换为可测量标准（如「brief」→「一句话」，「minimal」→「只修改冲突行本身」）。

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 的所有 skill 补 `## Output` 节
   - **为何可行**：bureau 只有 7 个 skill，每个 10 分钟，总计 1 小时
   - **第一步**：读 `bureau/skills/capture/SKILL.md`，根据 skill 内容推断产出，添加 `## Output` 节
2. **想法**：把 `commit/SKILL.md` 的五段结构当模板，写一个 skill [frontmatter](#frontmatter) 脚手架
   - **为何可行**：只需要把这 5 个 `##` 节固定在模板里，每次新建 skill 填充
   - **第一步**：在 CLAUDE.md 里加一条「新建 skill 时必须包含 Goals / Inputs / Steps / Output / Template」规则

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度
- **本案例 openai/symphony 的核心目的**：用少量精心设计的 skill 规范 AI 在 git 工作流中的行为

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都用 SKILL.md 定义 AI 行为；都有 commit/PR/deploy 相关 skill | gstack 有 50+ skill，symphony 只有 6 个 | 高 |
| MarkQWu/bureau | 中 | 都有精心写的 skill，都追求 AI 行为可预测 | bureau 是知识管理，symphony 是代码工作流 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 缺 `## Output` 节 | `grep -L "^## Output" gstack/*/skills/*/SKILL.md` | gstack：~48/50 个 skill 缺 Output，bureau：6/7 缺 Output | 高 |
| 模糊量词 | `grep -rn "judgment\|brief\|minimal" gstack/openclaw/skills/` | gstack openclaw skills 命中 4 处 | 中 |
| 引用路径不含前缀 | `grep -rn "AGENTS.md" gstack/*/skills/` | 未在 gstack skills 发现（gstack 有 AGENTS.md 在根目录） | 低 |

**命中后的具体行动建议**：
- `gstack/openclaw/skills/gstack-openclaw-investigate/SKILL.md` → 在文末添加 `## Output` 节，声明输出格式（调查报告：根因、证据、建议行动）→ 15 分钟
- `bureau/skills/capture/SKILL.md` 等 6 个 → 各补 `## Output` 节 → 每个 10 分钟

### 7.3 别人的更优方案

1. **领域**：`template-design`（skill 结构标准化）
   - **本案例做法**：`commit/SKILL.md` 有完整五段结构（Goals/Inputs/Steps/Output/Template），显式承担参考实现角色（`.codex/skills/commit/SKILL.md`）
   - **我的项目现状**：gstack 的大多数 skill 只有 Steps，没有 Goals/Inputs/Output 节
   - **如何借鉴**：在 CLAUDE.md 里加规则「新建 skill 时必须包含五节」，逐步重构存量 skill

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：`examples-driven`
- **我的做法**：`MarkQWu/bureau` 的每个 skill 都有 3 个 `<example>` 块（`bureau/skills/recall/SKILL.md`）
- **本案例做法**：symphony 的所有 skill 均无示例块——输入输出之间没有具体例子
- **意义**：bureau 的 skill 在缺示例这点上实际上比 openai/symphony 做得更好；这是一个可以在社区分享的亮点

---

## 八、术语表

### <a name="自主实现系统"></a>自主实现系统
> 一种 AI 系统架构，AI 接收任务描述后，在隔离的环境中自主完成代码修改、测试、提交，全程不需要人类手把手监督每一步。Symphony 的核心价值就是把「AI 帮我改代码」变成「AI 自主完成一个完整任务直到 PR 合并」。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`）。Codex CLI 和 Claude Code 读 SKILL.md 时都先解析 frontmatter 才能正确注册这个 skill。
