# AgriciDaniel/claude-obsidian — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-obsidian
**Stars**：976 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `security-gate`, `curl-pipe-bash-risk`, `template-design`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude + Obsidian 双模插件——同一目录既是 Claude Code 插件，也是 Obsidian vault。核心思想：把每次对话产生的知识写进一个持久增长的 wiki，下次会话开始时注入热点摘要，让 Claude 带着「记忆」继续工作。

关键事实：
1. 976 Stars，作者 AgriciDaniel，同为 claude-ads（2377 Stars）作者，作品风格一贯：精美 CLAUDE.md、结构化 wiki 目录、精心设计的 hook 编排
2. NL Score 91/100——整体高分，但 4 个 commands 均被扣掉 -30 分以上，反映出一个典型的「头重脚轻」问题：SKILL.md 精细，commands 粗糙
3. Security BLOCKED（2 Critical）：两个 [hook](#hook) 均涉及向模型上下文注入内容，是生态中极少见的架构级安全问题
4. 当前 HEAD 已新增 GEMINI.md（多模型支持）、WIKI.md、AGENTS.md 文档，以及 wiki-fold skill 和 bin/ 目录，说明项目仍在活跃演化
5. 安装方式：`claude plugin install claude-obsidian@AgriciDaniel --scope project`，无额外依赖

### 1.2 架构剖析
- **目录结构**：
```
claude-obsidian/
  commands/
    autoresearch.md     # 自动研究：拉网页 → 提炼 → 写入 wiki
    save.md             # 会话知识保存到 wiki
    wiki.md             # wiki 查询 / 更新
    canvas.md           # 生成 Obsidian canvas 可视化图
  skills/
    wiki/SKILL.md       # wiki 读写规范
    wiki-query/SKILL.md # 语义查询 + 反写 wiki
    wiki-lint/SKILL.md  # wiki 质量检查
    wiki-ingest/SKILL.md  # 批量导入外部文档
    wiki-fold/SKILL.md  # (新增) 折叠整理
    autoresearch/SKILL.md
    save/SKILL.md
    canvas/SKILL.md     # 生成 canvas JSON 图
    defuddle/SKILL.md   # 网页正文提取
    obsidian-bases/SKILL.md
    obsidian-markdown/SKILL.md
  agents/
    wiki-lint.md        # wiki 质量代理
    wiki-ingest.md      # 批量导入代理
  hooks/
    hooks.json          # SessionStart / PostToolUse / Stop 三类 hook
  wiki/
    hot.md              # 滚动热点摘要（由 Claude 写入）
    log.md              # 会话日志
    getting-started.md
    index.md / overview.md
  .raw/                 # 不可变原始文档（source of truth）
  .claude-plugin/plugin.json
  CLAUDE.md / WIKI.md / AGENTS.md / GEMINI.md / CHANGELOG.md
```
- **文件类型分布**：11 个 skill，2 个 agent，4 个 command，1 个 hooks.json，1 个 [manifest](#manifest)
- **编排关系**：用户触发 command → command 调用对应 skill（如 `autoresearch.md` 调用 `skills/autoresearch/SKILL.md`）→ skill 可调用 `defuddle` 提取网页正文；复杂导入任务 → `wiki-ingest.md` 代理并行批处理；会话结束时 Stop hook 检测 wiki 变化并提示更新 `wiki/hot.md`。整体是「命令层 + 技能层 + 代理层」的标准三层结构，但三层之间的 [allowed-tools](#allowed-tools) 声明残缺
- **跨件契约**：`commands/wiki.md` 引用 `skills/wiki/references/plugins.md`、`references/modes.md` 等路径，但这些文件未在 audit 时确认存在；`skills/wiki-ingest/SKILL.md` 引用 `references/frontmatter.md`；`skills/canvas/SKILL.md` 要求「先读 canvas-spec.md」，但 canvas-spec.md 不在 audit 范围内。跨件引用链多且未被 manifest 完整追踪

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「热缓存 + 不可变原料」——`.raw/` 存原始文档永不修改，`wiki/` 是 Claude 可持续写入的知识库，`wiki/hot.md` 是每次会话开始时注入的浓缩摘要。这套设计的根基是「跨会话知识复利」：每次交互都使知识库更丰富，下次会话的起点比上次更高
- **解决什么问题**：Claude 每次会话都从零开始，不记得上次讨论了什么。作者用 wiki + SessionStart hook 绕过了这个限制——不是依赖模型本身的记忆，而是用文件系统构建外置记忆
- **做了什么 trade-off**：外置记忆 vs. 安全隔离。热缓存注入的便利性是真实的，但代价是向模型上下文开放了一个 Claude 可写、也因此可被污染的文件。作者选择了便利，牺牲了隔离
- **反映什么认知模型**：作者把 Claude 看作「需要带着笔记本上班的专家」——没有持久记忆的专家会反复犯同样的错误，所以要给他一本每次上班前必读的热点摘要。这个比喻准确，但忽略了「笔记本也可能被人篡改」

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「热缓存滚动注入 + 不可变源仓」（Hot-Cache Rolling Injection + Immutable Source Store）**

外置记忆模式的一种：把 Claude 生成的知识沉淀到文件系统，会话开始时把最新摘要注入上下文，形成「知识复利」循环。

模式特征清单（4 条）：
- 特征 1：`.raw/` 目录存放原始文档，Claude 只读不写，保持来源不可变
- 特征 2：`wiki/hot.md` 是滚动摘要，由 Claude 在每次会话末尾更新，下次会话开头被 SessionStart hook 注入
- 特征 3：`wiki/log.md` 记录每次会话操作，提供审计轨迹
- 特征 4：Obsidian vault 和 Claude plugin 共享同一目录，同一批 wiki Markdown 文件既可被 Claude 读写，也可在 Obsidian 中可视化浏览

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人知识管理 / 笔记增强（Obsidian 用户） | ✅ 高度适用 | 双模共享目录是核心卖点，Obsidian 生态用户直接受益 |
| 长期持续的研究项目（跨周、跨月） | ✅ 适用 | 知识复利在时间跨度长的项目里价值最显著 |
| 多用户 / 团队协作环境 | ⚠️ 谨慎 | hot.md 由单个 Claude 会话写入，多人环境下容易冲突 |
| 对安全隔离有要求的生产环境 | ❌ 不适用 | SessionStart 注入是架构级安全风险，生产环境禁用 |
| 一次性任务 / 不需要记忆延续的场景 | ❌ 过度设计 | 维护 wiki 的成本高于收益 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 热缓存滚动注入（本仓库） | claude-obsidian | 知识复利效应强，人类也能在 Obsidian 浏览 wiki | SessionStart 注入是安全风险；hot.md 被污染后无感知 |
| 独立记忆插件（命令触发） | MarkQWu/echo-sleuth-for-claude | 用户主动触发，无自动注入风险；JSONL 解析更结构化 | 知识访问需要主动调用，不「自动带记忆」 |
| 无状态大型 Skill | 许多单会话工具 | 零安全风险，维护简单 | 每次会话从零开始，无法利用历史积累 |

### 2.4 改进空间
1. **当前问题**：SessionStart hook 用 `cat wiki/hot.md` 注入全文，没有长度上限。**改进做法**：改为读取 hot.md 的前 N 行（如 `head -50 wiki/hot.md`）并在 hook 命令里加 token 预算注释，超过阈值时触发 `wiki/hot.md` 的自动摘要压缩。**预期收益**：控制注入 token 消耗，避免 hot.md 膨胀后拖慢每次会话启动
2. **当前问题**：4 个 commands 全部缺少 `name:` [frontmatter](#frontmatter) 字段，导致命令无法被 `/nlpm:ls` 正确注册。**改进做法**：在每个 command 文件头部的 `---` 块里加 `name: <command-name>`，5 分钟可全部修完。**预期收益**：修复注册 bug，命令在插件目录中可见
3. **当前问题**：Stop hook 用 `echo` 向模型输出多行 LLM 指令，即使改成条件触发（当前 HEAD 已是 `git diff` 条件下才 echo），仍属于 hook → 模型上下文的可控注入面。**改进做法**：改用 `prompt` 类型 hook 或改成 `description` 字段声明意图，不在 Stop hook 里输出 LLM 指令字符串。**预期收益**：消除 Critical #1 安全发现

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-20 当时得分 **91/100**（19 个文件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/autoresearch.md | 60 | 缺 `name:` frontmatter（-25），无 allowed-tools（-5），多步工作流未编号（-10） |
| commands/save.md | 60 | 缺 `name:` frontmatter（-25），无 allowed-tools（-5），多步工作流未编号（-10） |
| commands/canvas.md | 70 | 缺 `name:` frontmatter（-25），无 allowed-tools（-5） |
| commands/wiki.md | 70 | 缺 `name:` frontmatter（-25），无 allowed-tools（-5） |
| skills/wiki-query/SKILL.md | 85 | `Write` 工具未声明在 allowed-tools 里，但 skill 实际会写 wiki 页面 |
| agents/wiki-lint.md | 97 | `Bash` 声明在 tools 但 agent body 里无实际 bash 命令 |
| skills/autoresearch/SKILL.md | 98 | 模糊量词："major contradictions"（-2） |
| skills/save/SKILL.md | 98 | 模糊量词："most valuable content"（-2） |
| skills/wiki-lint/SKILL.md | 98 | 模糊量词："significant cross-references"（-2） |
| skills/wiki/SKILL.md | 98 | 模糊量词："significant query exchange"（-2） |
| skills/wiki-ingest/SKILL.md | 96 | 模糊量词："significant ideas"、"relevant domain"（-4） |
| hooks/hooks.json | 90 | 自动 commit `.raw/`（含潜在凭据）；Stop hook 用 echo 注入 LLM 指令 |
| agents/wiki-ingest.md | 100 | — |
| .claude-plugin/plugin.json | 100 | — |
| CLAUDE.md | 100 | — |
| skills/canvas/SKILL.md | 100 | — |
| skills/defuddle/SKILL.md | 100 | — |
| skills/obsidian-bases/SKILL.md | 100 | — |
| skills/obsidian-markdown/SKILL.md | 100 | — |

### 3.2 当时值得借鉴的模式
1. **不可变源仓**（`experience-accumulation`）→ 为什么好：`.raw/` 目录设计为「只写一次，永不覆盖」的来源仓，Wiki 是 `.raw/` 内容的派生物。这确保了即使 wiki 出错或被污染，原始资料仍可信 → 原文路径：`CLAUDE.md` 中 `.raw/ = immutable source documents` 描述 + `hooks/hooks.json` PostToolUse 规则（auto-commit 把 wiki 和 .raw 分开维护）→ 如何借鉴：设计外置记忆系统时，将「来源文档」和「生成内容」严格分层，生成内容出错可以从来源重新生成
2. **wiki/hot.md 热缓存设计**（`experience-accumulation`）→ 为什么好：与其在会话开始让 Claude 读全量 wiki（上下文爆炸），不如维护一个由 Claude 自己精炼的滚动摘要。每次会话末尾更新 hot.md，下次会话头部注入，形成「压缩 → 注入 → 更新」的紧凑循环 → 原文路径：`hooks/hooks.json` SessionStart + Stop hook 组合，`wiki/hot.md` 文件本身 → 如何借鉴：在 echo-sleuth 类系统里，可以把 `experience-synthesis` 的输出维护成一个滚动摘要文件，而不是每次全量重算
3. **skill 命名与目录对齐**（`manifest-discipline`）→ 为什么好：每个 skill 目录名（`wiki-query/`）和 `SKILL.md` 里的 `name:` 字段保持一致，plugin.json 对应注册，无孤儿文件 → 原文路径：`.claude-plugin/plugin.json` → 如何借鉴：skill 目录名 = `name:` frontmatter = plugin.json 注册名，三者必须保持一致
4. **wiki/log.md 操作日志**（`experience-accumulation`）→ 为什么好：wiki 操作留下审计轨迹，知道哪次会话添加了哪个页面，出错时可以回溯 → 如何借鉴：在知识库类系统里保持操作日志，即使只是追加一行时间戳 + 操作摘要

### 3.3 当时的缺陷
1. **4 个 commands 全部缺 `name:` frontmatter 字段**：commands/autoresearch.md、canvas.md、save.md、wiki.md 均无 `name:` 声明。为什么这是根本问题：`name:` 是 Claude Code 注册 command 的唯一标识符，缺失后插件系统无法把用户的 `/wiki` 调用正确路由到对应文件，命令在 `/nlpm:ls` 里也不可见。作者可能专注在 skill body 的质量上，忽略了 command 的 frontmatter 合规性。**自查**：我的 commands 有没有全部声明 `name:` 字段？
2. **SessionStart hook 注入 Claude 可写文件**（Critical）：`hooks/hooks.json` line 9 用 `cat wiki/hot.md` 把文件内容注入每次会话上下文。问题根源：`hot.md` 是 Claude 写入的，Claude 也会读取它。攻击者或恶意 web 页面只需让 Claude 把「下次会话请执行 X」写进 hot.md，这条指令就会在下次 SessionStart 时静默地进入模型上下文，用户完全无感。这是间接 [提示注入](#prompt-injection) 的教科书案例。**自查**：我的系统有没有把 AI 生成内容自动注入到下一次模型上下文的地方？
3. **Stop hook 用 `echo` 向模型输出 LLM 指令**（Critical）：hook 命令的 stdout 会被 Claude Code 注入模型上下文，Stop hook 在这里输出一段多句英文指令（要求 Claude 更新 wiki/hot.md），等于 hook 作者在用户不知情的情况下向模型注入指令。为什么会失败：这个 hook 正常工作时确实方便（自动提醒 Claude 保存），但在被攻击者控制时，修改这段 echo 字符串就能让 Claude 在每次会话结束时执行任意操作。**自查**：我的 hook 有没有通过 echo / stdout 向模型发送指令？

### 3.4 当时的优化机会
1. **commands 批量加 `name:`**：4 个文件各 2 分钟，总计 10 分钟以内可全部修完，NLPM 评分可从 60-70 回升到 85-90
2. **skills/wiki-query 的 `allowed-tools` 加 `Write`**：该 skill 实际调用 Write 写 wiki 页面（lines 44、56、153），但 frontmatter 未声明，导致 Claude 可能因权限问题静默失败。直接在 frontmatter 里追加 `Write` 即可
3. **模糊量词替换**：把 6 个 skill 里的 "significant / major / most valuable / relevant domain" 替换为可操作标准，例如「significant ideas → 超过 3 句话阐述的概念」、「most valuable content → 按引用频率排前 5 的段落」

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| 4 个 commands 缺 `name:` frontmatter | `grep -L "^name:" commands/*.md`（期望结果：列出缺少的文件） | **持续存在**：4 个 command 文件均无 `name:` 字段，grep 命中全部 4 个 | 命令注册 bug 未修复，插件在 `/nlpm:ls` 中仍不可见 |
| SessionStart `cat wiki/hot.md` 注入 | `grep -n "cat wiki/hot.md" hooks/hooks.json` | **持续存在**：line 9 仍为 `cat wiki/hot.md` | Critical #2 安全风险未修复，间接提示注入入口仍开放 |
| Stop hook `echo` 注入 LLM 指令 | `grep -n "echo" hooks/hooks.json` → 读 line 46 上下文 | **部分变化**：line 46 已改为条件触发（`git diff` 检测到 wiki 变化才 echo），内容变为 `echo 'WIKI_CHANGED: Wiki pages were modified this session. Please update wiki/hot.md...'`；echo 机制本身未移除 | Risk 从「每次会话结束都注入」降为「有 wiki 变化才注入」，但 hook→上下文注入的架构风险仍存在 |

### 4.2 架构演进
Audit 时（2026-04-20）：4 commands、11 skills（无 wiki-fold）、2 agents、无 GEMINI.md / WIKI.md / AGENTS.md、无 bin/ 目录。

当前 HEAD（2026-05-22）：
- **新增**：GEMINI.md（Gemini 模型支持文档）、WIKI.md、AGENTS.md 分拆文档，wiki-fold skill，bin/ 目录（可能含脚本工具），docs/ 目录
- **未变**：commands 层 4 个文件，核心 hooks 逻辑，wiki/ 目录结构
- **未修复**：所有 audit 缺陷

作者后来意识到的：项目文档需要按受众分拆（CLAUDE.md 给 Claude 读，WIKI.md 给 wiki 使用者读，AGENTS.md 给代理系统读，GEMINI.md 给 Gemini 用户读）——这是一个「文档按读者分层」的意识进化，值得借鉴。但核心代码质量问题（name 字段、hook 安全）未被优先处理，说明作者把功能扩展排在缺陷修复前面。

### 4.3 新增的可学习模式
**文档按读者分层**：CLAUDE.md（给 Claude）、WIKI.md（给 wiki 操作者）、AGENTS.md（给代理系统）、GEMINI.md（给 Gemini 用户）——四个文档各自面向不同读者，避免把所有内容塞进一个巨型 CLAUDE.md。对比大多数插件把所有文档挤在单个 CLAUDE.md 里，这个分层做法让每个读者只看到与自己相关的内容，减少噪声。

---

## 五、校准

### 5.1 我已经在做对的
1. **commands 都有 `name:` 字段**：echo-sleuth 的 8 个 commands 全部有 `name:` 声明，不存在本案例的注册 bug
2. **没有自动向模型上下文注入 AI 生成文件**：echo-sleuth 的 skills 和 commands 不使用 hook 把 Claude 写入的文件自动注入回上下文，避免了间接提示注入
3. **manifest 同步**：plugin.json 完整注册所有 skill / command，无孤儿文件
4. **操作日志意识**：echo-sleuth 的 memory-management skill 有会话日志追踪，和 wiki/log.md 的设计理念一致

### 5.2 挑战 / 验证
- **这次案例挑战了我的一个假设**：「高 Stars 仓库的基础 bug 应该已经被社区发现并 PR 修复」。本仓库 976 Stars，4 个 commands 的 `name:` 字段缺失这种显而易见的 bug 在 audit 之后至少 32 天（2026-04-20 → 2026-05-22）仍未被修复，说明：一、许多用户并不依赖 `/nlpm:ls` 发现问题；二、「高 Stars = 高质量」的直觉是不可靠的
- **验证了一个正确做法**：「把 AI 生成内容和触发模型的上下文严格隔离」。本案例是这个原则被破坏后的反例——hot.md 由 Claude 写入并被 hook 注入回 Claude，形成「AI 可写 + AI 必读」的闭环。这个设计便利性高但安全边界模糊，在我的 echo-sleuth 设计中，明智地选择了「用户主动触发 recall / recap」而非「自动注入历史摘要」

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 是否都有 name: 字段
grep -rL "^name:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
# 命中（文件被列出）后：在缺失文件的 frontmatter 里加 name: <command-name>

# 检查我的 commands 是否都有 allowed-tools 声明（本案例的 bug）
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md
# 命中（文件被列出）后：在 frontmatter 里加 allowed-tools，列出该 command 实际使用的工具

# 检查是否有 hook 通过 stdout 注入模型上下文
grep -n "echo\|cat " ~/.claude/hooks/*.json 2>/dev/null
# 命中后：评估是否为「AI 可写文件 → hook → 模型上下文」链路，若是，重构为 prompt 类型 hook

# 检查 skill 里的模糊量词
grep -rn -E '\b(significant|major|most valuable|relevant domain|comprehensive|appropriate)\b' /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
# 命中后：把模糊形容词替换为可测量标准，如「超过 N 次出现」「引用频率排前 K」
```

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 里引入类 hot.md 的「精华摘要」机制，但用安全方式实现——不用 hook 自动注入，改为 `/recall --hot` 子命令主动加载
   - **为何可行**：echo-sleuth 已有 `experience-synthesis` skill 可以生成摘要；`recall.md` command 已有基础查询能力；只需加一个 `--hot` 模式读取并呈现摘要文件，无安全风险
   - **第一步**：在 `skills/memory-management/SKILL.md` 里定义 `hot-cache.md` 的格式规范（最多 50 行，固定结构：Top Decisions / Active Threads / Next Steps），然后在 `commands/recall.md` 添加「如果用户说"带热点启动"则加载 hot-cache.md」的触发条件，约 1-2 小时

2. **想法**：仿照本仓库的「来源不可变」设计，给 echo-sleuth 加 `.raw-sessions/` 目录存放用户指定要保留的原始会话快照，与经过 `prune` 处理后的 memory 文件严格分层
   - **为何可行**：echo-sleuth 已有 `prune` 命令做清理，但当前没有「已归档原始来源」的概念；参考 `.raw/` 的语义，`.raw-sessions/` 可作为不可变的审计底稿
   - **第一步**：在 `CLAUDE.md` 里定义 `.raw-sessions/` 语义，更新 `skills/memory-management/SKILL.md` 里的路径约定，约 30 分钟

3. **想法**：给 echo-sleuth 的 4 个缺失 `allowed-tools` 的 commands 补全声明（lessons.md、timeline.md、recap.md、recall.md）
   - **为何可行**：这是直接可执行的 5 分钟修复，完全参考本案例的 bug 教训
   - **第一步**：打开 4 个文件，在 frontmatter 里加 `allowed-tools`，列出实际使用的工具（Bash、Read 等），运行 `/nlpm:score` 验证分数提升

---

## 七、对照我的 GitHub 仓库

> 数据源：`/tmp/my-repos/`（含 MarkQWu/drama-workshop-skills、MarkQWu/claude-for-legal、MarkQWu/echo-sleuth-for-claude）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 AgriciDaniel/claude-obsidian 的核心目的**：通过 wiki + 热缓存注入实现 Claude 跨会话知识积累，让每次会话都能站在前一次的肩膀上

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同为「挖掘 Claude 会话历史、提取知识」的记忆类插件；都有 experience-synthesis 类 skill；都面向个人知识积累 | echo-sleuth 主动触发（用户 /recall），claude-obsidian 被动注入（hook 自动）；echo-sleuth 解析 JSONL，claude-obsidian 写 Markdown wiki；echo-sleuth 无 Obsidian 集成 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 插件，都有 SKILL.md | 目的完全不同：drama-workshop 面向剧本创作，无记忆积累逻辑 | 低 |
| MarkQWu/claude-for-legal | 低 | 同为多 skill 插件 | 领域完全不同，无记忆/wiki 逻辑 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 `allowed-tools` 声明 | `grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **命中**：lessons.md、timeline.md、recap.md、recall.md 共 4 个文件缺 allowed-tools | 高 |
| skill 模糊量词 | `grep -rn "relevant\|significant" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | **命中**：git-mining/SKILL.md line 93（"relevant git commits"）、experience-synthesis/SKILL.md line 118（"relevant sessions"）| 中 |
| commands 缺 `name:` 字段 | `grep -rL "^name:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **未命中**：8 个 commands 均有 `name:` | — |
| AI 生成文件通过 hook 自动注入模型上下文 | `grep -n "echo\|cat " ~/.claude/hooks/*.json` | **未命中**：echo-sleuth 无 hook | — |

**命中后的具体行动建议**：
- `commands/lessons.md` → 在 frontmatter 里加 `allowed-tools: [Bash, Read]`（该命令实际调用 scripts/ 下的 bash 脚本）→ 5 分钟
- `commands/timeline.md` → 在 frontmatter 里加 `allowed-tools: [Bash, Read]` → 5 分钟
- `commands/recap.md` → 在 frontmatter 里加 `allowed-tools: [Bash, Read]` → 5 分钟
- `commands/recall.md` → 在 frontmatter 里加 `allowed-tools: [Bash, Read, WebSearch]`（如有网络查询）→ 5 分钟
- `skills/git-mining/SKILL.md line 93` → 把 "relevant git commits" 改为 "git commits in the specified time range" → 2 分钟
- `skills/experience-synthesis/SKILL.md line 118` → 把 "relevant sessions" 改为 "sessions touching the specified project directory" → 2 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：跨会话知识热缓存（hot.md 设计）
   - **本案例做法**：维护 `wiki/hot.md`——Claude 每次会话结束后把当次新增知识的摘要 append 进去，SessionStart 时自动注入，形成「知识入账 → 下次提取」的循环。原文件路径：`hooks/hooks.json`（SessionStart + Stop 组合）+ `wiki/hot.md`
   - **我的项目现状**：echo-sleuth 没有 hot-cache 机制，每次会话需要用户主动运行 `/recall` 才能取回历史知识，没有「自动带着上次结论上班」的效果
   - **如何借鉴**：在 `skills/memory-management/SKILL.md` 里定义 `hot-cache.md` 格式（50 行上限，三段式：Active Decisions / Open Threads / Next Actions）；在 `commands/recall.md` 里加「启动时如果 hot-cache.md 存在且在 7 天内，自动展示」的逻辑；**不要**用 hook 自动注入，改为用户 `/recall` 触发时主动展示

2. **领域**：文档按读者分层（CLAUDE.md / WIKI.md / AGENTS.md 分拆）
   - **本案例做法**：当前 HEAD 已把说明文档拆成 4 个按读者定向的文件，CLAUDE.md 只讲「Claude 怎么用这个插件」，WIKI.md 讲「wiki 操作者怎么管理 wiki」，AGENTS.md 讲「代理系统怎么交互」
   - **我的项目现状**：echo-sleuth 的 CLAUDE.md 把「用户操作指南」「架构说明」「开发者注意事项」全部混在一起，随着功能扩展会变成难以维护的大文件
   - **如何借鉴**：把 echo-sleuth 的 CLAUDE.md 拆成 CLAUDE.md（仅保留 Claude 需要的：架构速览 + 命令路由）+ DEVELOPERS.md（开发者注意事项）；约 30 分钟重组

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：commands 的 `allowed-tools` 合规性与 `name:` 字段完整性
   - **我的做法**：echo-sleuth 的 8 个 commands 均有 `name:` 字段（`grep -c "^name:" commands/*.md` 全部命中），且 4/8 有完整的 `allowed-tools` 声明（commands/audit.md、dashboard.md、extract.md、prune.md）
   - **本案例做法**（弱在哪）：4 个 commands 全部缺 `name:`，全部缺 `allowed-tools`，是本仓库 NL Score 从 91 下拉的主要原因
   - **意义**：若被人审到，这是 echo-sleuth 的合规亮点；若考虑给上游 PR，补 `name:` 是一个极低风险、极易被接受的首次 PR 切入点

2. **领域**：安全隔离——无 hook 向模型自动注入 AI 生成内容
   - **我的做法**：echo-sleuth 完全无 hook，所有历史知识访问均由用户主动触发，AI 生成内容（memory files）不会被自动注入回模型上下文
   - **本案例做法**（弱在哪）：SessionStart + Stop 两个 hook 均向模型上下文注入内容，且注入的 hot.md 是 Claude 自己写的，构成间接提示注入闭环
   - **意义**：echo-sleuth 的「被动记忆访问」设计是正确的安全选择，本案例是反例教材；在给他人介绍「跨会话记忆」设计时，可以用 claude-obsidian 的 Security BLOCKED 状态说明为什么不能用 hook 自动注入 AI 生成文件

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读 SKILL.md 或 command 文件时先解析 frontmatter 才能知道这个 skill / command 怎么注册和调用。**缺少 `name:` 字段 = 插件系统不知道这个文件叫什么，无法注册**。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`.claude-plugin/plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="hook"></a>hook
> Claude Code 提供的事件触发机制，可以在特定时机（SessionStart=会话开始、PostToolUse=工具调用后、Stop=会话结束）自动执行一段 shell 命令。命令的 stdout 输出会被注入模型上下文，这是本仓库两个 Critical 安全问题的根本原因。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md 或 command 文件 frontmatter 里的一个列表字段，声明这个 skill / command 允许 Claude 使用哪些工具（如 `Bash`、`Read`、`Write`）。未声明的工具即使 Claude 想用也会被权限系统拒绝。如果 skill 的 body 里写了「用 Write 保存文件」但 allowed-tools 里没有声明 `Write`，这个操作会静默失败。

### <a name="prompt-injection"></a>提示注入（Prompt Injection）
> 攻击者通过某种途径（如恶意网页内容、被污染的文件）向模型上下文插入指令，让模型执行攻击者意图的操作，而不是用户意图的操作。**间接提示注入**：攻击者不直接和模型对话，而是污染模型会读取的数据（如 wiki/hot.md），让模型在后续会话中自动执行恶意指令。本仓库的 SessionStart hook 注入 Claude 可写的 hot.md，就创建了间接提示注入的完整攻击链路。

### <a name="experience-accumulation"></a>经验积累（Experience Accumulation）
> 一类插件设计模式：把每次 AI 会话产生的洞察、决策、模式系统性地保存到文件，使后续会话可以站在前一次的基础上继续推进，而不是每次从零开始。代表实现：本仓库的 wiki + hot.md，以及 MarkQWu/echo-sleuth-for-claude 的 memory files。两者的核心差异在于「是否允许 AI 生成内容自动注入回下一次模型上下文」。
