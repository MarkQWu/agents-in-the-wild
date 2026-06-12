# hamelsmu/claude-review-loop — 学习案例

**仓库**：https://github.com/hamelsmu/claude-review-loop
**Stars**：655 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-12（基于当前 HEAD）
**主题标签**：`fallback-chain`, `security-gate`, `cross-reference`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`claude-review-loop` 是一个**极简的 Claude Code 插件**，实现了「AI-to-AI 代码审查循环」：用户通过 `/review-loop` 命令发起一个任务，Claude 负责实现代码，任务完成后 stop hook 自动触发 Codex（OpenAI）对实现结果做独立审查，Claude 再根据 Codex 的发现进行修订。整个循环自动闭合，无需人工介入。

关键事实：
1. 作者 Hamel Husain — 知名 ML 从业者与教育者，以大量高质量教程和开源贡献著称
2. 655 stars，结构极其精简：当前仓库可见的 NL 工件仅 4 个
3. 核心价值主张：**两个独立 AI 模型互相制衡**，消除单一模型的盲点
4. 插件于 2026 年初重构，将全部内容迁移至 `plugins/review-loop/` 目录下

### 1.2 架构剖析

**审计时（2026-04-06）的结构**：
```
commands/
  review-loop.md          # 主命令：初始化状态文件，发起任务
  cancel-review.md        # 取消正在进行的审查循环
hooks/
  hooks.json              # Stop 事件绑定 stop-hook.sh
scripts/
  stop-hook.sh            # 生成 Codex 审查脚本并执行
plugins/review-loop/
  .claude-plugin/plugin.json   # 插件清单
```

**当前 HEAD（2026-06-12）的结构**：
```
README.md
plugins/
  review-loop/
    CLAUDE.md
    AGENTS.md
    commands/
      cancel-review.md
      review-loop.md
```

- **文件类型分布**：2 个命令文件、1 个 CLAUDE.md、1 个 AGENTS.md；hooks/ 和 scripts/ 已在重构中整合（或内化为 AGENTS.md 中的 agent 定义）
- **编排关系**：用户 → `/review-loop <任务>` → 状态文件写入 `.claude/review-loop.local.md` → Claude 执行任务 → Stop hook 触发 → Codex 审查脚本运行 → Claude 读取审查结果并修订
- **跨件契约**：cancel-review.md 负责删除 `.claude/review-loop-run-codex.sh` 和 `.claude/review-loop-codex-prompt.txt`，这两个文件均由 stop-hook.sh 生成，形成完整的状态生命周期管理

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「一个命令，一个循环，两个 AI」——尽可能少的文件，尽可能清晰的职责划分
- **解决什么问题**：单一 AI 模型在自我审查时存在固有盲点，无法发现自己的假设错误；引入独立模型作为外部审查者突破这一局限
- **Trade-off**：要求用户系统中安装 `@openai/codex`，增加了依赖；但作者选择以「功能完整性」换取「零依赖」，因为独立审查者本身就是核心价值
- **认知模型**：把 AI 代码审查看作「结对程序员协议」——一人写代码，一人独立 review，两者的分歧才是真正需要被解决的地方

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「Stop Hook 触发的 AI-to-AI 审查循环 + 状态文件协议」**

关键特征：
- 使用 Claude Code 的 Stop 事件作为触发器，在 AI 完成工作后自动启动下一阶段
- 通过临时状态文件（`.claude/review-loop.local.md`）在命令与 hook 之间传递上下文
- 「实施者」与「审查者」由不同 AI 模型担任，保证独立性
- 配套提供 cancel-review.md 管理状态清理，形成完整的生命周期
- Review ID 格式 `YYYYMMDD-HHMMSS-hexhex` 做到确定性、可排序、人类可读

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要高质量代码审查、但团队人力不足 | ✅ 高度适用 | Codex 独立审查相当于额外一双眼睛，无需人工等待 |
| 关键业务逻辑的安全性/正确性验证 | ✅ 适用 | 双模型交叉验证能发现单模型的假设漏洞 |
| 快速原型、不追求代码质量的场景 | ❌ 不适用 | 审查循环会显著延长每次任务的总耗时 |
| 无法访问 OpenAI API 的环境 | ❌ 不适用 | Codex 审查步骤依赖 OpenAI 网络访问 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Stop Hook + AI-to-AI 循环 | hamelsmu/claude-review-loop | 自动闭环，无需人工介入；独立模型消除盲点 | 强依赖外部 AI 服务；空输入 bug 未修复 |
| 多 agent 并行编排 | 2389-research/review-squad | 多专家同时工作，速度快 | 协调复杂，agent 间需要明确协议 |
| 人工 review + AI 辅助 | 大多数 PR review 工具 | 保留人类判断力 | 速度慢，无法做到全自动化 |

### 2.4 改进空间

1. **当前问题**：`review-loop.md` 中 `$ARGUMENTS` 未做空值校验，用户执行 `/review-loop` 不带参数时会静默启动一个没有任务描述的循环。**改进做法**：在 bash 脚本头部加 `[ -z "$ARGUMENTS" ] && { echo "Error: task description required"; exit 1; }` 校验并提前终止。**预期收益**：避免让 Claude 消耗 token 执行一个目标不明的空任务。

2. **当前问题**：AGENTS.md 引入后，stop hook 的逻辑位置不明确（原 scripts/stop-hook.sh 是否已移入或整合）。**改进做法**：在 CLAUDE.md 或 AGENTS.md 中加一节「Component Map」，明确说明每个文件的职责及相互引用关系。**预期收益**：降低新贡献者理解成本。

3. **当前问题**：`@openai/codex` 使用 `npm install -g` 安装，未锁定版本。**改进做法**：在命令注释中说明推荐版本，或使用 `npm install -g @openai/codex@x.y.z` 锁定。**预期收益**：防止 Codex 更新引入破坏性变更导致 hook 失败。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 得分 **89/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/review-loop.md | 64 | 无编号步骤，无空输入处理，无输出格式声明，3 处模糊量词 |
| commands/cancel-review.md | 90 | 4 步操作流以散文写成，未编号 |
| plugin.json | 100 | 满分，各字段完整规范 |
| hooks/hooks.json | 100 | 满分，事件绑定清晰 |

加权平均：**89/100**

### 3.2 当时值得借鉴的模式

1. **plugin.json 100/100 满分结构** → 为什么好：字段完整（name, version, description, author, license, keywords 全部填写），description 精准描述功能，keywords 语义准确可搜索。路径：`plugins/review-loop/.claude-plugin/plugin.json`。如何借鉴：对照 NLPM 检查清单逐字段核对自己的 plugin.json，特别检查 `author.url` 和 `license` 是否遗漏。

2. **hooks.json 100/100 满分结构** → 为什么好：事件类型（Stop）、匹配器、脚本路径三者清晰对应，无歧义。路径：`hooks/hooks.json`。如何借鉴：hooks.json 中的 `command` 字段应使用绝对路径或 `${CLAUDE_PLUGIN_ROOT}` 变量，避免相对路径依赖执行目录。

3. **状态文件协议（State File Protocol）** → 为什么好：`.claude/review-loop.local.md` 作为命令与 hook 之间的唯一通信渠道，生命周期由 `cancel-review.md` 明确管理（创建 → 读取 → 删除）。如何借鉴：凡是涉及跨组件通信的插件，设计一个单一的状态文件而非散落的环境变量，并配套提供清理命令。

4. **cancel 命令配套主命令** → 为什么好：`cancel-review.md` 明确列出了所有需要清理的生成文件，用户有清晰的退出路径。如何借鉴：每个有副作用（生成文件、写配置）的命令都应配套一个 undo/cancel 命令。

### 3.3 当时的缺陷

1. **问题**：`review-loop.md` 中 `$ARGUMENTS` 未做空值校验——用户运行 `/review-loop` 不加参数时，状态文件被写入空 body，循环静默启动。**根本原因**：命令内联 bash 脚本是从独立的 `scripts/setup-review-loop.sh` 复制而来，但复制时遗漏了 `[ -z "$ARGUMENTS" ]` 校验逻辑；而两份实现并存本身也是设计风险。**自查**：我的 echo-sleuth 命令文件中，有没有使用 `$ARGUMENTS` 但未做校验的命令？

2. **问题**：`review-loop.md` 无输出格式声明（Output Format section），Claude 不知道任务完成后的产物应该是什么样的。**根本原因**：作者专注于触发机制的设计，对「Claude 执行阶段」的指令缺乏约束。**自查**：我的命令文件有没有声明 `## Output Format`？

3. **问题**：3 处模糊量词（如「some files」「a few issues」），导致 AI 无法判断「多少」算满足条件。**根本原因**：自然语言写作习惯，没有意识到模糊量词对 AI 执行的危害。**自查**：运行 `/nlpm:score` 扫描自己的命令文件中的模糊量词。

### 3.4 当时的优化机会

1. `review-loop.md` 加 `[ -z "$ARGUMENTS" ]` 空值校验（3 行代码，高性价比）
2. `cancel-review.md` 把 4 步流程改为编号列表（1/2/3/4 步骤）
3. `hooks/stop-hook.sh` 头部加 `set -euo pipefail`，并将 `$REVIEW_LOOP_CODEX_FLAGS` 用双引号包裹防止 word splitting

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `$ARGUMENTS` 空值未校验 | 查看 review-loop.md 中 bash 脚本是否有 `[ -z "$ARGUMENTS" ]` | **仍未修复** ❌ — `argument-hint: "<task description>"` 仅作提示，bash 脚本第 24 行无校验 | 沉积 bug，用户体验缺口持续存在 |
| `cancel-review.md` 无编号步骤 | 查看当前文件的步骤写法 | **状态未知** — 重构后文件位于 `plugins/review-loop/commands/cancel-review.md`，无法从浅克隆确认 | 可能通过重构改善，也可能原样保留 |
| `stop-hook.sh` 安全问题 | 查看 hook 脚本中 `REVIEW_LOOP_CODEX_FLAGS` 引用方式 | **状态未知** — 重构后 hooks/ 不在浅克隆可见路径内，无法直接验证 | 架构重组可能同时迁移或重写了 hook |

### 4.2 架构演进

从审计时的「扁平式：commands/ + hooks/ + scripts/ + plugin.json 分散在根目录」→ 现在的「包式：所有内容归入 `plugins/review-loop/` 子目录，新增 CLAUDE.md 和 AGENTS.md」。

这一重构的意义：
- **AGENTS.md 的引入**：说明作者把 Codex 审查者抽象为一个独立的 agent 声明，而不仅仅是 hook 脚本中的一行命令。这是从「脚本化」走向「声明化」的进化。
- **CLAUDE.md 的引入**：为整个插件提供顶层上下文，让 Claude 在插件加载时就了解设计意图——这符合 Claude Code 的最佳实践。
- **包式结构**：`plugins/review-loop/` 作为自包含单元，方便在 monorepo 中管理多插件。

### 4.3 新增的可学习模式

- **AGENTS.md 声明审查者 agent**：把 Codex 审查者作为独立 agent 声明，而不是隐藏在 bash 脚本里的一行命令。这让整个审查循环的「参与者」在代码层面可见可管理。
- **plugins/ 子目录结构**：把插件内容与仓库根目录分离，便于一个仓库托管多个插件（如未来增加 `plugins/review-loop-lite/` 等变体）。

---

## 五、校准

### 5.1 我已经在做对的

1. **plugin.json 字段完整性**：echo-sleuth 的 plugin.json 填写了 name、version、description 和 keywords。参照本案例 100/100 的标准，还需要检查 `author.url` 和 `license` 字段是否存在。
2. **命令文件声明 allowed-tools**：echo-sleuth 的命令文件有 `allowed-tools` 声明，与本案例 review-loop.md 的做法一致（review-loop.md 列出了 Bash/Read/Write/Edit/Glob/Grep）。
3. **专注单一职责**：echo-sleuth 的每个命令对应一个明确的功能（挖掘对话历史、提取决策等），与本案例「一个命令做一件事」的设计哲学一致。

### 5.2 挑战 / 验证

本案例验证了一个我之前只是隐约知道但没有明确执行的原则：**内联 bash 脚本必须与独立脚本的功能完全对齐**。hamelsmu 的案例清楚展示了「两处实现」的危险：`scripts/setup-review-loop.sh` 有空值校验，命令内的内联 bash 没有，结果 bug 在内联版本中存活了数月。这迫使我重新审视：我的命令文件中是否也有「复制自某处但遗漏了关键逻辑」的内联脚本？

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件中所有使用 $ARGUMENTS 的位置
grep -rn "\$ARGUMENTS" /home/user/my-repos/MarkQWu-echo-sleuth-for-claude/commands/ 2>/dev/null
# 命中后：确认每处 $ARGUMENTS 上方是否有 [ -z "$ARGUMENTS" ] 校验

# 检查 plugin.json 是否包含 author.url 和 license
cat /home/user/my-repos/MarkQWu-echo-sleuth-for-claude/plugin.json | python3 -c "
import json,sys
d=json.load(sys.stdin)
for f in ['author','license']:
    print(f, ':', d.get(f,'MISSING'))
if isinstance(d.get('author'),dict):
    print('author.url:', d['author'].get('url','MISSING'))
"
# 命中后：补填缺失字段，参照 hamelsmu 的 100/100 格式

# 检查命令文件是否有 Output Format 声明
grep -rL "## Output Format\|## 输出格式" /home/user/my-repos/MarkQWu-echo-sleuth-for-claude/commands/ 2>/dev/null
# 命中后：为缺少该 section 的命令文件补充输出格式声明
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 添加 cancel 命令对应每个有副作用的主命令
   - **为何可行**：echo-sleuth 的 `/extract` 等命令会生成临时文件，但目前没有对应的清理命令
   - **第一步**：列出所有会写文件的命令（如 extract、audit），为每个创建 `cancel-<command>.md`，明确列出需要删除的文件路径

2. **想法**：仿照 AGENTS.md 模式，将 echo-sleuth 中的 agent 角色显式声明
   - **为何可行**：echo-sleuth 已有 recall、file-historian 等 agent，当前分散在 agents/ 目录
   - **第一步**：创建顶层 `AGENTS.md`，汇总所有 agent 的职责、触发条件和输出格式，作为整个插件的 agent 注册表

3. **想法**：引入 Stop Hook 实现某种自动后处理
   - **为何可行**：echo-sleuth 的某些操作（如会话历史挖掘）完成后可以自动触发摘要写入
   - **第一步**：参考本案例的 hooks.json 格式，在 echo-sleuth 的 hooks/ 目录创建一个最简单的 Stop hook，验证触发机制是否正常工作

---

## 七、对照我的 GitHub 仓库

> 数据源：user 仓库（echo-sleuth-for-claude、drama-workshop-skills、claude-for-legal）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 hamelsmu/claude-review-loop 的核心目的**：通过 AI-to-AI 审查循环提升代码质量，Claude 实现 + Codex 审查 + Claude 修订

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是 Claude Code 插件（有 plugin.json）；都有命令 + hook 架构 | echo-sleuth 无 AI-to-AI 循环；核心目的是挖掘历史而非代码审查 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件形式 | drama-workshop 是创作辅助工具，无 hook 机制 | 低 |
| MarkQWu/claude-for-legal | 低 | 都有领域专业知识封装 | claude-for-legal 无 hook，无状态文件协议 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `$ARGUMENTS` 无空值校验 | `grep -rn "\$ARGUMENTS"` in commands/ | echo-sleuth: 需要检查 `/recall` 命令是否接受空参数并静默执行 | 高 |
| 命令文件无 Output Format 声明 | `grep -rL "Output Format"` in commands/ | echo-sleuth: 多个命令文件疑似缺少该 section | 中 |
| plugin.json 缺 author.url | `cat plugin.json \| jq '.author'` | echo-sleuth: plugin.json 中 author 字段仅填写了名称，无 URL | 低 |

**命中后的具体行动建议**：
- echo-sleuth/commands/recall.md → 检查命令头部 bash，加入空参数提前退出逻辑
- echo-sleuth/commands/ 下所有文件 → 统一补充 `## Output Format` section，明确文件路径和内容结构

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：Stop Hook 触发自动后处理
   - **本案例做法**：`hooks/hooks.json` 绑定 Stop 事件，Claude 完成任务后自动运行 `stop-hook.sh`，不需要用户手动触发 Codex 审查（路径：`hooks/hooks.json` → `scripts/stop-hook.sh`）
   - **我的项目现状**：echo-sleuth 无 Stop hook，所有操作需要用户手动触发下一步命令
   - **如何借鉴**：在 echo-sleuth 中添加 Stop hook，当 `/audit` 命令完成后自动将结果写入 `evals/latest-audit.md`，免去用户手动触发写入步骤

2. **领域**：状态文件协议 + 对应的清理命令
   - **本案例做法**：`review-loop.local.md` 是唯一的状态载体；`cancel-review.md` 明确列出所有需删除的生成文件（`.claude/review-loop-run-codex.sh`、`.claude/review-loop-codex-prompt.txt`）
   - **我的项目现状**：echo-sleuth 的副作用文件散落各处，无统一的清理命令
   - **如何借鉴**：为每个产生临时文件的命令制定「状态文件协议」：生成文件名固定、路径固定、配套 cancel 命令

3. **领域**：插件清单（plugin.json）的完整度
   - **本案例做法**：plugin.json 100/100，所有字段填写，包括 `author.url`、`license`、`keywords`（语义准确）
   - **我的项目现状**：echo-sleuth 的 plugin.json 缺少 `author.url`，`license` 字段填写但较简略
   - **如何借鉴**：逐字段对照本案例的 plugin.json，补填缺失字段（10 分钟改完）

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：命令参数的 argument-hint 与实际校验的一致性
   - **我的做法**：echo-sleuth 在有 argument-hint 的命令中同时做了参数存在性检查（未静默接受空参数）
   - **本案例做法**（弱在哪）：review-loop.md 有 `argument-hint: "<task description>"`，但实际 bash 脚本没有对应的空值校验，`argument-hint` 沦为装饰性文本，不能防止误用
   - **意义**：提示声明与运行时行为必须一致，「有提示但无校验」比「无提示无校验」更危险，因为它给用户虚假的安全感

2. **领域**：agent 架构的显式分层
   - **我的做法**：echo-sleuth 有明确的 commands/、agents/、skills/ 三层分离，每层职责清晰
   - **本案例做法**（弱在哪）：审计时的 review-loop 没有独立的 agents/ 层，Codex 审查逻辑藏在 bash 脚本中，不够声明化（重构后引入 AGENTS.md 才改善）
   - **意义**：从项目起始就做架构分层，比后期重构成本低得多

---

## 八、术语表

### stop hook
> Claude Code 的「完成后触发器」。当 Claude 完成一轮回答、准备停止时，Claude Code 会自动执行 hooks.json 中绑定到 Stop 事件的脚本。hamelsmu 用这个机制实现了「Claude 写完代码 → 自动触发 Codex 审查」的自动化闭环。与 UserPromptSubmit hook（用户发消息时触发）相对，Stop hook 是「AI 说完话」的监听器。类比：快递员送完货签字离开时，系统自动发送「包裹已送达」短信——不需要快递员手动操作，完成动作本身就是触发条件。

### REVIEW_LOOP_CODEX_FLAGS
> `stop-hook.sh` 中读取的环境变量，允许用户向 Codex 审查命令传递额外的 CLI 标志（如 `--model gpt-4o`）。审计发现的安全问题：该变量在脚本内被「未加引号」地展开进了一段动态生成的 shell 脚本字符串。未加引号意味着变量值中的空格会触发 word splitting，如果变量值包含特殊字符（如 `; rm -rf /`）则会导致命令注入。安全修复：将所有变量展开改为 `"${REVIEW_LOOP_CODEX_FLAGS}"` 双引号包裹形式，并在脚本头部添加 `set -u` 使未定义变量报错而非静默展开为空。

### 孤儿组件（Orphaned Component）
> 插件中存在一个文件，但没有其他组件引用它（或它所引用的文件已被删除）。本案例中，审计时 `scripts/setup-review-loop.sh` 是独立的安装脚本，与 `commands/review-loop.md` 内的内联 bash 逻辑平行存在。当命令文件的内联 bash 做了修改但 `setup-review-loop.sh` 未同步更新时，两者就出现了逻辑分歧。重构后 `setup-review-loop.sh` 疑似被删除，孤儿问题通过「消灭孤儿」而非「修复一致性」解决。自查方法：`grep -rn "setup-review-loop" .`——如果结果为零，说明该脚本已成孤儿或已被删除。

### 状态文件协议（State File Protocol）
> 一种通过临时文件在插件不同组件之间传递上下文的设计模式。在本案例中：(1) `review-loop.md` 将任务描述写入 `.claude/review-loop.local.md`；(2) stop-hook.sh 读取该文件并生成 `.claude/review-loop-run-codex.sh` 和 `.claude/review-loop-codex-prompt.txt`；(3) `cancel-review.md` 负责清理上述所有生成文件。关键要素：状态文件路径固定、格式约定明确、存在配套的清理机制。反模式：使用环境变量传递多步状态（进程退出后丢失）或散落在随机路径的临时文件（无清理路径）。

### AI-to-AI 审查循环（AI-to-AI Review Loop）
> 由两个独立 AI 模型组成的代码审查协议：第一个 AI（Claude）负责根据需求实现代码，完成后由第二个 AI（Codex/GPT）独立审查实现质量，并将发现的问题反馈给第一个 AI 进行修订。「独立」是核心价值：两个 AI 使用不同的基础模型、不同的训练数据，对「好代码」有不同的先验假设，因此能互相发现对方的盲点。类比：飞机的双飞行员制度——不是因为一个飞行员不够能干，而是因为两套独立判断能捕捉单人操作时的系统性漏洞。与「AI 自我审查」（同一模型先写后检）的本质区别在于：独立模型不共享相同的假设，无法对相同的错误视而不见。
