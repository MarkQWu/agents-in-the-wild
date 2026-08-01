# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-08-01（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `template-design`, `single-purpose`, `cross-reference`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
c0x12c/ai-toolkit 是一个以「Spartan / GSD（Get Stuff Done）」为名的 Claude Code 全栈开发工具包，包含 10 个 agents、25 个 commands、34 个 skills，覆盖全栈开发、基础设施、设计、研究、创业运营等场景。61 ★。核心亮点是**「双目录镜像架构」**：`toolkit/` 是源代码，`.codex/` 是发布编译产物（通过 `compile-packs.js` 生成）。插件版本 v1.22.1，是成熟商业级工具包。

关键事实：
1. 10 agents + 25 commands（全部以 `spartan:` 前缀） + 34 skills（全在 `toolkit/skills/`）
2. 双目录架构：`.codex/` 是 npm publish 产物，`toolkit/` 是作者维护的源头
3. skills 平均分 98.6/100——整个工具包质量最高的一层
4. commands 平均分 94.8/100——唯一的系统性问题是全部缺 `allowed-tools`
5. 有 `bridges/` 目录：Telegram bot bridge、engine.js 等——将 Spartan 接入聊天机器人

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── toolkit/
  │   ├── agents/               # 10 agents（源）
  │   ├── commands/spartan/     # 25 commands（源）
  │   ├── skills/               # 34 skills（源）
  │   └── rules/                # 设计/架构/命名等规则文档（未注册为 skill）
  ├── .codex/
  │   ├── agents/               # 5 agents（编译产物，未含全部 10 个）
  │   ├── commands/spartan/     # 62 commands（含额外扩展）
  │   └── skills/               # 26 skills（编译产物，未含全部 34 个）
  ├── bridges/
  │   └── core/engine.js        # bypassPermissions 默认开启（Medium 安全风险）
  └── CLAUDE.md, AGENTS.md      # 项目文档
  ```
- **文件类型分布**：10 agents、25 commands、34 skills，另有 5 agents / 8 skills 仅在 `.codex/` 存在的镜像差异
- **编排关系**：commands 以 `spartan:` 前缀通过 slash command 触发；agents 以 `spartan:gsd` 为核心编排器；skills 通过 `/use-skill` 加载
- **跨件契约**：`gsd-project-researcher` agent 产出 6 个文件供 `gsd-research-synthesizer` 消费——隐式文件名约定，无正式 schema 声明

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「Spartan 哲学——做事而非聊天（Get Stuff Done）」，每个 skill 是一个可独立执行的任务单元，不是参考文档
- **解决什么问题**：全栈开发团队用 AI 加速从需求到部署的全链路，统一工作流标准（GSD 框架）
- **做了什么 trade-off**：双目录镜像（开发体验 vs 维护一致性）；bypassPermissions 默认开启（用户体验 vs 安全性）
- **反映什么认知模型**：作者把 Claude Code 视为"可以有自己 SDK 和工具链的平台"——`compile-packs.js` 是他们的构建工具，`.codex/` 是"npm 包"，`toolkit/` 是"源代码"

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「源码 + 编译产物双目录」发布型架构**

维护 `toolkit/`（人工编辑）和 `.codex/`（脚本生成），通过 `prepublishOnly` 钩子在发布时自动同步。

模式特征清单：
- 特征 1：有明确的"源"和"产物"分离，发布时自动重新生成产物
- 特征 2：产物目录（.codex/）与源目录（toolkit/）部分不同步（8 skills、5 agents 仅在源）
- 特征 3：skills 质量最高（98.6），agents 次之（94.3），commands 略低（94.8，因缺 allowed-tools）
- 特征 4：rules/ 下有丰富规则文档，但未注册为 skill（发现的改进空间）
- 特征 5：bridges/ 提供额外集成层（Telegram），体现"工具包 → 平台"的扩展性意图

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要分发给多用户的团队标准工具包 | ✅ 高度适用 | npm publish 模式便于安装和版本控制 |
| 个人项目/小团队 | ⚠️ 过度设计 | 双目录架构维护成本高，个人用 toolkit/ 直接引用即可 |
| 需要动态扩展的命令集 | ✅ 适用 | bridges/ 架构支持接入不同平台 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 双目录发布型（本仓库） | c0x12c/ai-toolkit | 版本化分发、构建链成熟 | 镜像不完整风险、维护双份 |
| 单目录直接使用型 | MarkQWu/bureau | 简单、无同步问题 | 不适合 npm 分发 |
| 大型多插件型 | jnuyens/gsd-plugin | 功能丰富（115 个文件） | 无模型声明、薄 skill 多 |

### 2.4 改进空间
1. **当前问题**：25 个 commands 全部缺 `allowed-tools`，62 个 `.codex/commands/` 也是如此。**改进做法**：在 `toolkit/commands/spartan/` 为每个 command 加 `allowed-tools`，重新 compile。**预期收益**：commands 从 94.8 → ~99.8，同时提升安全性。
2. **当前问题**：`toolkit/rules/` 内容丰富但未注册为 skills。**改进做法**：为每个规则文档加 SKILL.md 包装（name, description, allowed-tools 字段），让它们可以通过 `/use-skill` 加载。**预期收益**：Claude 可以动态引用设计规范，不只是开发者手动查阅。
3. **当前问题**：`.codex/` 比 `toolkit/` 少 8 个 skills 和 5 个 agents（镜像不完整）。**改进做法**：在 `compile-packs.js` 中确保所有 toolkit/ 文件都被镜像，或在 README 明确说明哪些文件不应出现在 `.codex/`。**预期收益**：消除"silent miss"的混淆。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-25 当时得分 **96/100**。安全 CLEAR（无 Critical/High）。

| 层级 | 当时平均分 | 主要问题 |
|---|---|---|
| Agents（10 个） | 94.3 | research-planner/idea-killer/ai-designer 无示例（-15 each） |
| Commands（25 个） | 94.8 | 全部缺 `allowed-tools`（-5 each） |
| Skills（34 个） | 98.6 | 部分缺 Gotchas section（-3 each） |

### 3.2 当时值得借鉴的模式
1. **skills 98.6 分——「Gotchas 段落」标准化** → 每个 skill 都有一个专门段落写"陷阱/常见错误"，告诉 Claude 什么情况下不要用这个方法。根本原因：负面示例和边界条件说明比"最佳实践"更能定型 AI 行为。借鉴：我的每个 skill 加 `## Gotchas` 段落。
2. **dual-agent review 模式（phase-reviewer + builder）** → 用两个 agent 相互制衡：builder 写代码，phase-reviewer 做质量门控。根本原因：单 agent 缺乏客观性；双 agent 模拟了真实代码审查流程。借鉴：在 bureau 中引入专门的 auditor → reviewer 双重验证链。
3. **spartan 命令有清晰的 argument-hint** → 如 `argument-hint: "[theme or problem space]"` 告诉用户如何调用。根本原因：用户发现命令时知道怎么用，降低认知负担。借鉴：所有有参数的 command 加 `argument-hint`。
4. **agents 有 model 声明** → phase-reviewer 明确 `model: sonnet`，让成本和质量都可预测。

### 3.3 当时的缺陷
1. **phase-reviewer agent 中 cat/ls 无 Bash 工具声明（高危 bug）** → agent body 写了 `cat .spartan/config.yaml` 和 `ls rules/`，但 tools 只有 Read/Grep/Glob，无 Bash。运行时 tool call 会被拒绝。为什么会失败：Claude Code 严格执行 tools 白名单，任何不在声明内的工具调用都会报错。自查：我的 agent body 有没有用了 Bash 命令但没在 tools 里声明？
2. **全部 25 个 commands 缺 allowed-tools（系统性缺陷）** → 没有一个 command 声明它需要哪些工具。为什么会失败：Claude 可能请求不必要的工具权限，或工具访问变得不可审计。自查：我的 commands 是否有 allowed-tools？
3. **engine.js bypassPermissions 默认开启（Medium 安全风险）** → 用户不主动查看文档就会在无权限确认模式下运行 bridge。为什么这么设计会失败：用户期望有确认步骤，静默绕过会产生信任危机。自查：我的 hooks 或 bridge 有没有默认高权限模式？

### 3.4 当时的优化机会
1. 为全部 25 个 commands 批量加 `allowed-tools` 字段（一个 PR，最高价值）
2. 为 research-planner/idea-killer/ai-designer 三个 agents 加示例块
3. 为 terraform-best-practices skill 加 `allowed_tools` [frontmatter](#frontmatter) 字段
4. 为 `toolkit/rules/` 下的规则文档加 SKILL.md 包装

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| phase-reviewer 缺 Bash 工具 | `grep "tools:" toolkit/agents/phase-reviewer.md` | **部分改善**：agent 新增了 model、examples，但仍无 `tools:` 字段声明 | 核心 bug 未完全修复，tools 字段缺失 |
| 全部 commands 缺 allowed-tools | `grep -L "allowed-tools" .codex/commands/spartan/*.md \| wc -l` | **持续**（62/62 个命令无 allowed-tools） | 系统性缺陷未处理 |
| terraform-best-practices 缺 allowed_tools | `grep "allowed_tools" toolkit/skills/terraform-best-practices/SKILL.md` | **持续**（frontmatter 只有 name/description）| 同上 |

### 4.2 架构演进
phase-reviewer agent 有显著改善：新增了 `model: sonnet` 声明，description 中加了两个 `<example>` 块（从 audit 的 -15 提升为质量改善）。这说明作者在部分修复——选择了最容易的改动（加 model 和 description examples），跳过了最难的改动（给全部 commands 加 allowed-tools）。

.codex/ 目录文件数量从审计时的 "5 agents / 26 skills" 扩展至更多命令（62 个 command 文件），说明产品功能继续迭代。

### 4.3 新增的可学习模式
**双 agent 协作（phase-reviewer v2）**：phase-reviewer 现在 description 中包含两个具体的调用场景示例（Gate 3.5 场景），作为 description-level examples 的最佳实践——description 中的 `<example>` 块既是用户文档，也是 AI 理解 agent 用途的上下文。这是"description 即文档"模式的优化实现。

---

## 五、校准

### 5.1 我已经在做对的
1. bureau 的 skills 有示例块（recall 等有 3+ 示例）——接近 ai-toolkit skill 层质量
2. bureau/auditor agent 有 `model: sonnet` 声明——与 phase-reviewer 改善后对齐
3. 我没有使用 bypassPermissions 默认模式——比 bridges/engine.js 更安全

### 5.2 挑战 / 验证
这个案例**验证了"批量改动阻力最大"的经验**：phase-reviewer 的单点 bug 3 个月内修了，但 25 个 commands 的系统性 allowed-tools 缺失没有触动。大规模改动（改 N 个文件）往往比精确改动（改 1 个文件）的阻力大得多——这与"技术债会积累"的常识一致，但这次看到了具体数字：62 个文件的系统性问题原地不动 3 个月。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 是否有 allowed-tools 声明
find /tmp/my-repos/MarkQWu-bureau/commands/ -name "*.md" | while read f; do
  if ! grep -q "allowed-tools\|allowed_tools" "$f" 2>/dev/null; then
    echo "MISSING allowed-tools: $(basename $f)"
  fi
done
```
命中后怎么办：对每个 command 分析它使用的工具（Read/Write/Bash/Glob 等），加对应的 `allowed-tools:` 行。

```bash
# 检查我的 agents body 是否使用了未声明的工具
# （比如 body 里有 cat/ls 这种 Bash 命令但 tools 没有 Bash）
grep -rn "^cat \|^ls \|^find \|^grep " /tmp/my-repos/MarkQWu-bureau/.claude/agents/*.md 2>/dev/null
```
命中后怎么办：把 shell 命令改为 Glob/Grep/Read 工具调用，或在 `tools:` 加 `Bash`。

### 6.2 灵感 → 实施路径
1. **想法**：为所有 bureau commands 批量加 allowed-tools
   - **为何可行**：bureau 有 11 个 commands，比 c0x12c 少，一个 PR 能完成
   - **第一步**：读每个 command 的 body，判断它用哪些工具（大多数只需 Read/Write/Glob/Bash），加对应的 `allowed-tools: [...]` 行

2. **想法**：在 bureau/skills/ 加 Gotchas 段落
   - **为何可行**：ai-toolkit 的 skills 98.6 分证明 Gotchas 段落价值高
   - **第一步**：对 bureau 的 7 个 skills 逐个加 `## Gotchas` 段落，写"什么时候不要用这个 skill / 哪些错误要避免"

3. **想法**：将 bureau 规则文档注册为 skills
   - **为何可行**：ai-toolkit 的 rules/ 值得包装为 skills——bureau 也有类似的规则描述散落在 CLAUDE.md 中
   - **第一步**：把 bureau CLAUDE.md 中的约束提炼为独立 SKILL.md，实现"可调用的项目规范"

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 c0x12c/ai-toolkit 的核心目的**：全栈开发团队的 AI 工作流标准化工具包（Spartan/GSD 框架）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件，都包含 agents+skills | bureau 专注知识库管理，ai-toolkit 专注全栈开发流程 | 高（skills 质量和 allowed-tools 实践） |
| MarkQWu/gstack | 中 | 都是多 skill 工具集 | gstack 较简单，ai-toolkit 有 25 commands + 10 agents 复杂编排 | 中 |
| MarkQWu/- | 无 | — | 完全不同场景 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 allowed-tools | `grep -rL "allowed-tools" bureau/commands/*.md` | bureau 命中 11/11 个 commands | 高 |
| agents body 使用未声明工具 | `grep -n "^cat \|^ls " bureau/.claude/agents/*.md` | 未命中 | — |
| skills 缺 Gotchas 段落 | `grep -rL "## Gotchas\|Gotcha" bureau/skills/*/SKILL.md` | 未检查，预计多数缺失 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/commands/lint.md` → 加 `allowed-tools: [Bash, Glob, Read]` → 5 分钟
- `MarkQWu/bureau/commands/review.md` → 加 `allowed-tools: [Read, Grep, Glob]` → 5 分钟
- 一个 PR 批量修复 11 个 commands → 约 25 分钟

### 7.3 别人的更优方案

1. **领域**：Skills Gotchas 标准化
   - **本案例做法**：`toolkit/skills/terraform-review/SKILL.md` 等 skills 有专门的 `## Gotchas` 段落，列举"哪些情况下不要用此 skill / 会出错的边界案例"
   - **我的项目现状**：bureau/skills/ 无 Gotchas 段落，只有 description 和 process steps
   - **如何借鉴**：为 bureau 7 个 skills 各加 `## Gotchas` 段落，内容聚焦"哪些场景会出错 / 哪些误用需要提前警告"

2. **领域**：description-level examples（agent 可发现性）
   - **本案例做法**：phase-reviewer 的 description 字段内嵌 `<example>` 块，让用户在 `/agents` 列表就能看到使用场景
   - **我的项目现状**：bureau/auditor description 是纯文字描述，无示例
   - **如何借鉴**：在 auditor description 中加一个示例，展示典型调用场景

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：commands 数量控制
- **我的做法**：bureau 只有 11 个 commands，职责边界清晰
- **本案例做法（弱在哪）**：62 个 commands（.codex 中）导致维护负担重，allowed-tools 缺失问题也因此放大
- **意义**：命令数量适中、职责单一是比"大而全"更可维护的选择

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据。Claude Code 解析 skill 的 `allowed_tools`、agent 的 `model` 和 `tools` 都从这里读取。

### <a name="allowed-tools"></a>allowed-tools
> command 或 agent 声明的"可使用工具白名单"。告诉 Claude Code 这个指令可以调用哪些工具（如 `Read`、`Write`、`Bash`、`Glob`）。声明后 Claude 只能使用白名单内的工具，超出范围的调用会被拒绝。缺少此字段意味着工具访问不受限制，带来安全和审计风险。

### <a name="bypassPermissions"></a>bypassPermissions
> Claude Code 的一种运行模式，跳过所有工具权限确认弹窗，直接允许所有操作。在自动化 CI/CD 场景中有用，但在用户机器上默认启用会让用户失去对 AI 操作的控制权。

### <a name="Gotchas"></a>Gotchas 段落
> skill 文件中专门记录"易错点"和"反面案例"的段落。标准写法是 `## Gotchas`，内容是"什么情况下不要用这个 skill"或"用了会出错的场景"。对 AI 来说，负面约束（不能做什么）比正面描述（应该做什么）更有助于精确控制行为。
