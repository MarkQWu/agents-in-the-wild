# jnuyens/gsd-plugin — 学习案例

**仓库**：https://github.com/jnuyens/gsd-plugin
**Stars**：9 | **来源**：upstream audit
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-08-01（基于当前 HEAD）
**主题标签**：`model-pinning`, `cross-reference`, `single-purpose`, `fallback-chain`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
jnuyens/gsd-plugin 是一个以「Get Stuff Done」为名的大型 Claude Code 插件，包含 **33 个 agents + 82 个 skills**，实现从项目规划到代码实现、调试、审查、文档生成的完整 AI 工作流。版本 v2.38.8，活跃维护中。仅 9 ★，但内容规模和设计质量超过多数高星仓库。

关键事实：
1. 33 agents + 82 skills = 115 个 NL 文件，全部通过 NLPM 质量阈值（最低分 75/100）
2. 所有 33 个 agents 无一声明 `model:` 字段——唯一的普遍性系统缺陷
3. 有完整 hook 系统（SessionStart / PreToolUse / PostToolUse / PreCompact）
4. GSD 工作流设计：从 sketch → plan → implement → review → ship 的完整生命周期
5. 调试链精心设计：skill → session-manager agent → debugger agent，三层状态驱动

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── agents/               # 33 agents（gsd-* 前缀）
  ├── skills/               # 82 skills（gsd-workflow 约定）
  │   ├── debug/SKILL.md    # 100分，多子命令、安全边界
  │   ├── complete-milestone/SKILL.md  # 100分，8步流程
  │   └── thread/SKILL.md  # 100分，五种模式、代码示例
  ├── hooks/hooks.json      # 4个 hook（SessionStart + 3个工具钩子）
  ├── workflows/            # 跨 agent 工作流定义
  ├── templates/            # 项目模板
  ├── references/           # 参考材料（`@${CLAUDE_PLUGIN_ROOT}` 引用）
  ├── bin/gsd-tools.cjs     # hook 加载的 Node.js CJS 模块
  └── package.json          # AJV schema 验证依赖
  ```
- **文件类型分布**：33 agents、82 skills、4 hooks、0 commands（所有入口通过 skills 实现）
- **编排关系**：skill 触发 → agent 执行 → 产出文件 → 下一个 agent 消费；没有 router skill，靠 GSD 生命周期阶段约定编排
- **跨件契约**：`gsd-project-researcher` 产出 6 个文件（SUMMARY/STACK/FEATURES 等），供 `gsd-research-synthesizer` 消费——按节标题（section headers）而非正式 schema 传递

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一个工作流框架，不是工具箱」——所有组件按阶段（phase）编排，agent 是流程里的角色，skill 是阶段的执行说明书
- **解决什么问题**：开发者在复杂项目中丢失上下文、无法从中断处续做、缺乏跨 session 状态管理
- **做了什么 trade-off**：115 个文件换来完整的生命周期覆盖，但维护成本极高；选择不声明 model 换来简单统一，但丧失成本控制能力
- **反映什么认知模型**：作者把 Claude 视为"流程执行者"，而非"随机对话者"——每个 skill 是一个有确定性 I/O 的操作步骤，整体像一个 DAG（有向无环图）工作流

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「分阶段流水线」大型 AI 工作流架构**

33 agents 按 GSD 生命周期分工，通过文件系统传递状态（`.planning/` 目录），每个 agent 是流水线的一个节点。模式名：**「多 agent 流水线 + 文件驱动状态」**。

模式特征清单：
- 特征 1：agents 以 `gsd-` 前缀统一命名，对应 GSD 阶段（gsd-planner/gsd-executor/gsd-verifier...）
- 特征 2：通过文件系统（`.planning/`）在 agents 间传递状态，而非通过 context 直接传递
- 特征 3：hook 系统监控所有工具调用（SessionStart + PreToolUse + PostToolUse + PreCompact）
- 特征 4：`@${CLAUDE_PLUGIN_ROOT}` 变量在所有 agent/skill 中引用外部 workflows/templates
- 特征 5：调试链三层分工：skill（入口）→ session-manager（状态管理）→ debugger（执行）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 复杂长周期项目（月级开发） | ✅ 高度适用 | 文件状态持久化，支持 session 间续做 |
| 简单脚本/单次任务 | ❌ 过度设计 | 115 个文件的框架对单次任务而言是工程过剩 |
| 团队标准化工作流 | ✅ 适用 | GSD 阶段提供统一的开发语言 |
| 教学/学习 AI 工作流设计 | ✅ 适用 | 覆盖最完整的设计模式，是绝佳参考 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多 agent 流水线（本仓库） | jnuyens/gsd-plugin | 生命周期完整、状态持久、阶段清晰 | 维护复杂、无 model 声明、成本不可控 |
| Spartan/GSD 双层（源+编译）| c0x12c/ai-toolkit | 分发成熟、skill 质量高 | 镜像维护负担 |
| 轻量 skill-only | MarkQWu/gstack | 简单、无编排复杂度 | 缺乏跨 session 状态 |

### 2.4 改进空间
1. **当前问题**：33 agents 全部无 `model:` 声明，默认用 Sonnet。**改进做法**：按复杂度分类——gsd-doc-classifier → `haiku`，gsd-debugger → `sonnet`，gsd-planner → `sonnet`，恢复单模型 per agent 声明。**预期收益**：agent 平均分从 88.7 → ~94，成本可控。
2. **当前问题**：PostToolUse hook 匹配所有工具包括 Read/Grep/Glob，每次文件读取都触发钩子延迟。**改进做法**：把 matcher 缩窄为 `Bash|Edit|Write|MultiEdit|NotebookEdit`。**预期收益**：消除重度文件扫描场景的 3s hook 延迟。
3. **当前问题**：producer-consumer 间隐式文件格式约定（section headers），无 schema。**改进做法**：在 `references/` 加一个 STATE_SCHEMA.md 文档，定义每种状态文件的必选和可选节。**预期收益**：agent 版本演进时降低 schema 漂移风险。

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-28 当时得分 **90.3/100**。安全 CLEAR，Bugs 0，Quality Issues 较多但全部在阈值以上。

| 层级 | 当时均分 | 最低分文件 | 最高分文件 |
|---|---|---|---|
| Agents（33 个） | 88.7 | 76（gsd-advisor-researcher） | 95（gsd-nyquist-auditor/gsd-roadmapper/gsd-security-auditor） |
| Skills（82 个） | 90.9 | 75（9 个薄委托 skill） | 100（多个） |

### 3.2 当时值得借鉴的模式
1. **调试三层链（skill → session-manager → debugger）** → 把"调试状态持久化"（session-manager）和"调试执行"（debugger）拆分为独立 agents，各司其职。根本原因：状态管理和执行逻辑混在一起会导致上下文污染。借鉴：bureau 中可以类似分离"写操作审计"和"写操作执行"。
2. **`reapply-patches/SKILL.md` 100 分——Hunk Verification Table** → 定义了明确的三方合并逻辑和"Hunk Verification Table"格式，让 AI 能结构化地处理补丁冲突。根本原因：操作型 skill 需要给出完整的成功判据。借鉴：我的 skill 加 "成功判据" 段落。
3. **gsd-security-auditor 内置输入边界** → skill 中有明确的安全输入校验说明（SQL 注入/XSS 检查）。根本原因：安全审计 skill 需要自带防御性设计。借鉴：涉及用户输入的 skill 加输入边界说明。
4. **hooks 有版本感知 fallback** → hook 命令先查 canonical 路径，失败则扫描 semver 最高版本目录。体现了 defensive loading 模式。

### 3.3 当时的缺陷
1. **33 agents 全无 model 声明（系统性、165 点罚分）** → 所有 agent 默认 Sonnet，但 gsd-doc-classifier（枚举分类，90 行）和 gsd-debugger（科学调试法，1453 行）对资源的需求完全不同。为什么会失败：过度用 Sonnet 浪费金钱；under-用 Haiku 会让复杂推理失准。自查：我的 agents 有没有按复杂度声明 model？
2. **9 个薄委托 skill 得 75 分（零示例 + 无输出格式）** → `cleanup`、`do`、`fast` 等 skill 只有目标声明和 `@workflow` 引用，无任何具体示例。为什么会失败：薄 skill 对 AI 来说是"黑盒委托"，无法预测行为，也无法调试。自查：我的 skill 有没有只有一两行目标说明？
3. **PostToolUse 匹配 Read/Grep/Glob，性能影响大** → 每次文件读取都触发 3s timeout 的 hook。为什么这么设计会失败：file-heavy 的 plan/research 阶段会变慢 10 倍以上。自查：我的 hooks 有没有匹配了不需要的只读工具？

### 3.4 当时的优化机会
1. 为 33 agents 加 model 声明（一行 per agent，最高 ROI 改动）
2. 为 9 个薄 skill 加输出格式 + 一个示例
3. 收窄 PostToolUse hook matcher

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 33 agents 无 model 声明 | `grep -rL "^model:" agents/*.md \| wc -l` | **持续**（33/33 无 model）| 核心质量缺陷 3 个月未处理 |
| 薄 skill（9 个，75分） | `wc -l skills/cleanup/SKILL.md`（23 行）| **持续**（仍在 22-37 行范围内）| 内容未扩充 |
| PostToolUse 匹配 Read/Grep/Glob | `cat hooks/hooks.json`（matcher 字段）| **未检查，逻辑预期持续**（设计未变）| 性能问题持续 |

### 4.2 架构演进
从 audit 到当前 HEAD，文件结构基本稳定（33 agents / 82 skills），说明核心 GSD 工作流设计已成熟，近期迭代侧重功能而非质量修复。CHANGELOG 显示 v2.38.8 处于活跃迭代中（有 changelog），但 model 声明等质量改进未被纳入任何版本。

### 4.3 新增的可学习模式
**`set-profile/SKILL.md` 有 `model: haiku`**——唯一一个声明了 model 的 skill（非 agent）。这说明作者知道这个功能，却没有在 agents 层应用。这个模式本身是正确的：profile 查询是枚举型操作，Haiku 完全够用，这是成本优化的正确示范。如果能推广到其他简单 agents，效果会很显著。

---

## 五、校准

### 5.1 我已经在做对的
1. bureau/auditor 有 `model: sonnet` 声明——在 agents model 声明上优于 jnuyens 整个 33 个 agent 群
2. bureau 的 skills 有示例块，比 gsd-plugin 的 9 个薄 skill 更规范
3. bureau 没有匹配只读工具的 PostToolUse hook——无此性能问题

### 5.2 挑战 / 验证
这个案例**挑战了"大量文件 = 高质量"的直觉**。115 个文件，NLPM 平均 90.3，但最基础的 `model:` 字段 0/33 agents 声明。规模和质量不相关——系统性缺陷往往是"被第一个项目决策锁定的"，后来再改需要修改 33 个文件。这验证了"第一次做对"的重要性：在写第一个 agent 时就加 `model:`，而不是等到 33 个都写完才发现问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否有 model 声明
find /tmp/my-repos/ -name "*.md" \( -path "*/agents/*" -o -path "*/.claude/agents/*" \) \
  | while read f; do
    if ! grep -q "^model:" "$f" 2>/dev/null; then
      echo "MISSING model: $f"
    fi
  done
```
命中后怎么办：根据 agent 的任务复杂度选择 model tier——分类/枚举 → `haiku`；推理/规划 → `sonnet`；复杂多步骤 → `sonnet` 或 `opus`。

```bash
# 检查我的 hooks 是否匹配了 Read/Grep/Glob 等只读工具
cat ~/.claude/settings.json 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
hooks = data.get('hooks', {})
for event, handlers in hooks.items():
    if isinstance(handlers, list):
        for h in handlers:
            matcher = h.get('matcher', '')
            if any(t in matcher for t in ['Read', 'Grep', 'Glob']):
                print(f'WARNING: {event} matches read-only tool: {matcher}')
"
```
命中后怎么办：从 matcher 中移除 Read/Grep/Glob，只保留 Bash/Edit/Write/MultiEdit。

```bash
# 检查我的 skills 是否有薄委托问题（少于 20 行）
find /tmp/my-repos/ -name "SKILL.md" | while read f; do
  lines=$(wc -l < "$f")
  if [ "$lines" -lt 20 ]; then
    echo "THIN ($lines lines): $f"
  fi
done
```
命中后怎么办：至少加一个 `## Example` 块和一个 `## Output` 段落，让 AI 知道这个 skill 预期产出什么。

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau/compile command 引入 session 状态文件（参考 gsd-plugin 的 `.planning/` 模式）
   - **为何可行**：bureau 的 compile 操作有多步骤，状态持久化能支持断点续做
   - **第一步**：设计一个 `.bureau/compile-state.json` schema，定义"已处理页面"和"待处理页面"列表，修改 compile command 读写此文件

2. **想法**：将 bureau skills 的输出格式明确化（参考 gsd-plugin 的 100 分 skills）
   - **为何可行**：bureau/recall/SKILL.md 的输出结构在 3 次调用中表现不一致
   - **第一步**：在 recall/SKILL.md 加 `## Output Format` 段落，给出固定 JSON schema 或 Markdown 表格模板

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 jnuyens/gsd-plugin 的核心目的**：开发全生命周期 AI 工作流管理（从规划到上线的完整阶段框架）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件，都有 agents+skills | bureau 专注知识库管理，gsd-plugin 专注开发生命周期 | 高（model 声明、状态文件模式） |
| MarkQWu/gstack | 中 | 都是多 skill 工具集，都聚焦开发流程 | gstack 较简单，gsd-plugin 有完整编排 | 中（可学习 hook 设计） |
| MarkQWu/- | 无 | — | 完全不同场景 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agents 无 model 声明 | `grep -rL "^model:" */agents/*.md` | 未命中（bureau/auditor 有 model）| — |
| 薄 skill（< 20 行）| `wc -l */skills/*/SKILL.md` | 未检查，预计 1-2 个 | 低 |
| PostToolUse 匹配只读工具 | `cat ~/.claude/settings.json \| grep matcher` | 需手动检查 | 中 |

**命中后的具体行动建议**：
- 若发现薄 skill：`MarkQWu/bureau/skills/capture/SKILL.md` → 加 `## Example` 和 `## Output` 段落 → 15 分钟

### 7.3 别人的更优方案

1. **领域**：调试工作流的三层分离（skill → session-manager → debugger）
   - **本案例做法**：debug skill 创建状态文件 → gsd-debug-session-manager 管理跨 session 状态 → gsd-debugger 执行具体调试步骤
   - **我的项目现状**：bureau 的 review 和 compile 操作没有状态分离，每次 session 重新开始
   - **如何借鉴**：为 bureau/review command 设计状态文件（.bureau/review-state.json），让 review 可以从上次中断处继续

2. **领域**：`set-profile/SKILL.md` 的 `model: haiku` 声明
   - **本案例做法**：枚举型操作（profile 查询）明确降级到 haiku，节省 token
   - **我的项目现状**：所有 gstack skills 无 model 声明
   - **如何借鉴**：识别 gstack 中的简单枚举/查询 skill（如 `context-save`），加 `model: haiku` 降低成本

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：agent model 声明
- **我的做法**：bureau/.claude/agents/auditor.md 有 `model: sonnet`，成本和行为都可预测
- **本案例做法（弱在哪）**：33/33 个 agents 无 model 声明，全部默认 Sonnet，无差异化
- **意义**：bureau 的 agent 设计比 gsd-plugin 更精细，即使规模小也做到了 model tier 有意识选择

---

## 八、术语表

### <a name="model-pinning"></a>model 声明（model pinning）
> 在 agent 或 skill 的 frontmatter 里用 `model: haiku|sonnet|opus` 指定这个组件用哪个 AI 模型。不声明时默认用 Sonnet（当前 session 默认模型）。声明的好处：成本可控（分类任务用 Haiku 比 Sonnet 便宜约 10 倍）、行为可预期。

### <a name="薄委托"></a>薄委托 skill
> 只有一两行目标说明然后引用外部工作流（`@workflow`）的 skill，没有具体步骤、示例或输出格式描述。对 AI 来说是"黑盒"——知道要做什么，但不知道怎么做、做完了什么样子算成功。

### <a name="PostToolUse"></a>PostToolUse hook
> 在 Claude 调用工具"之后"自动执行的脚本。`matcher` 字段决定哪些工具触发此 hook。如果 matcher 包含 Read/Grep/Glob 这种只读工具，每次文件读取都会触发 hook，在文件密集型操作（如代码分析）中显著降低速度。

### <a name="文件驱动状态"></a>文件驱动状态（File-driven State）
> 用文件系统（而不是 Claude context）在不同 session 或不同 agent 之间传递状态信息。好处：跨 session 持久、可查看、可调试。坏处：需要定义文件格式约定，producer 和 consumer 必须遵守相同的文件结构。
