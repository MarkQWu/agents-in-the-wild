# jnMetaCode/superpowers-zh — 学习案例

**仓库**：https://github.com/jnMetaCode/superpowers-zh
**Stars**：1001 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-06-17（基于当前 HEAD）
**主题标签**：`template-design`, `experience-accumulation`, `cross-reference`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
jnMetaCode/superpowers-zh 是 [obra/superpowers](https://github.com/obra/superpowers)（159k+ ⭐）的简体中文社区版，在完整汉化 14 个原版 skill 的基础上新增了 6 个中国原创 skill，支持 Claude Code / Copilot CLI / Hermes / Cursor / Windsurf / Kiro / Gemini CLI 等 18 款 AI 编程工具。其核心价值主张是「让 AI 编程工具真正会干活」——把实战验证的开发方法论（TDD、子 agent 并发、代码审查话术）封装成可复用的[skill](#skill)。

关键事实：
- 2025 年从 obra/superpowers fork 而来，作者 jnMetaCode 专注于中文开发者社区
- 通过[SessionStart hook](#sessionstart-hook)自动在会话开始时注入 `using-superpowers` skill 内容
- 同时发布 npm 包（`superpowers-zh`）和适配多平台的 `.claude-plugin/`、`.codex-plugin/`、`.cursor-plugin/` 三套[manifest](#manifest)

### 1.2 架构剖析
- **目录结构**：

```
superpowers-zh/
├── skills/                    # 20 个 skill（14 汉化 + 6 中国原创）
│   ├── using-superpowers/     # 元 skill：会话入口，SessionStart hook 注入
│   ├── brainstorming/         # 头脑风暴
│   ├── chinese-code-review/   # 中文 code review 话术（原创）
│   ├── chinese-commit-conventions/  # 中文提交规范（原创）
│   ├── chinese-documentation/ # 中文技术文档（原创）
│   ├── chinese-git-workflow/  # 中文 git 工作流（原创）
│   ├── mcp-builder/           # MCP 服务器构建（原创）
│   ├── systematic-debugging/  # 系统化调试（含 3 个伴生 .md）
│   ├── writing-skills/        # 编写 skill 指南（含 4 个伴生 .md）
│   └── ...其余 11 个
├── hooks/
│   ├── session-start          # bash 脚本，读取 using-superpowers SKILL.md 注入会话
│   └── hooks.json             # 注册 SessionStart hook
├── .claude-plugin/            # Claude Code manifest
├── .cursor-plugin/            # Cursor manifest
├── .codex-plugin/             # Codex manifest
└── package.json               # Node engine constraint
```

- **文件类型分布**：20 个 SKILL.md，1 个 SessionStart hook，3 套 manifest（Claude/Cursor/Codex）
- **编排关系**：平铺结构。各 skill 独立加载，无 router。`using-superpowers` 作为元入口由 SessionStart hook 强制注入，指示 AI 在适用时调用对应 skill。内部存在有向依赖：`requesting-code-review` → `agents/code-reviewer`（审计时存在，现已移除）；`subagent-driven-development` → `finishing-a-development-branch`、`using-git-worktrees`、`writing-plans` 等。
- **跨件契约**：部分 skill 通过 `@filename` 语法内联加载同目录下的伴生 .md 文件（如 `systematic-debugging` 加载 `root-cause-tracing.md`、`defense-in-depth.md`、`condition-based-waiting.md`）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「方法论封装」——把经过实战验证的开发流程（TDD、并发子 agent、PR 请求/接受流程）抽象成 skill，每个 skill 对应一个具体场景，不跨界。
- **解决什么问题**：默认 AI 工具缺乏项目级方法论。作者通过 skill 向模型注入「下一步怎么做」的结构化指引，而不是让用户每次重新描述工作流程。
- **Trade-off**：平铺 skill 结构 vs 路由分层。选择了平铺——上手简单，用户直接 `/skill-name` 调用，但如果 skill 数量再多，发现成本会上升。
- **认知模型**：作者把 AI 编程工具视为「有记忆和方法论的结对程序员」——skill 是向其传授工作习惯的载体，不是命令行工具的封装。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「元入口 + 平铺 skill 集合」模式**：设置一个 `using-superpowers`（或类似）元 skill 作为会话入口，通过 SessionStart hook 自动注入，告知模型 skill 生态的存在和使用规则；其余 skill 平铺独立，各管一个场景。

模式特征清单：
- 每个 skill 对应一个场景，不跨界
- 有一个元 skill 作为发现入口，其余 skill 被动等待调用
- SessionStart hook 确保元 skill 在每次会话开始时存在于上下文
- 多平台共用同一份 skill 内容，只有 manifest 文件不同
- 子 skill 可通过 `@` 语法加载伴生参考文档

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 10-30 个开发工作流 skill 集 | ✅ 高度适用 | skill 数量适中，发现成本低，平铺清晰 |
| 需要根据上下文动态选择 skill 的复杂系统 | ⚠️ 勉强适用 | 超过 30 个 skill 后 `using-superpowers` 元入口会变得臃肿 |
| 多工具兼容（Claude/Cursor/Kiro 等） | ✅ 高度适用 | skill 内容与平台无关，只需多套 manifest |
| 有复杂工作流编排需求（if/else/router）| ❌ 不适用 | 平铺结构不支持编排逻辑，需要引入 agent 或 command |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 元入口 + 平铺 skill（本仓库） | superpowers-zh | 简单、多平台、上手快 | skill 多时发现困难，无编排能力 |
| Router + Channels（路由层 + 渠道层）| nlpm（本项目） | 可编排，支持并发调度 | 架构复杂，学习曲线高 |
| Command 驱动（单命令入口）| kbwo/ccmanager kiro 命令 | 用户无需了解 skill 体系 | 每个 command 自给自足，难以共享逻辑 |

### 2.4 改进空间
1. **当前问题**：`using-superpowers` 元 skill 随着 skill 增多会越来越长，注入上下文占用 token 增加。**改进做法**：只注入一个「skill 目录索引」（名称 + 一句描述），把详细规则留给被调用时懒加载。**预期收益**：减少每次会话开销约 60%，不影响发现性。
2. **当前问题**：多平台 manifest 文件（.claude-plugin、.cursor-plugin、.codex-plugin）内容重复，手动同步容易遗漏。**改进做法**：维护一个 `plugin-source.json` 作为单一真相，构建脚本自动生成各平台 manifest。**预期收益**：版本号一致，减少维护负担。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-20 当时得分 **95/100**。加权均值 94.75，24 个 NL 产物中 20 个满分。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/brainstorm.md | 60 | 缺 `name` [frontmatter](#frontmatter)，deprecated stub 缺 allowed-tools 和空输入保护 |
| commands/write-plan.md | 60 | 同上 |
| commands/execute-plan.md | 60 | 同上 |
| agents/code-reviewer.md | 94 | 3 处模糊量词「恰当/适当」（-6 分） |
| 其余 20 个 SKILL.md | 100 | 无 |

### 3.2 当时值得借鉴的模式
1. **skill 全文 JSON 转义注入**：`hooks/session-start` 用纯 bash 参数替换（`${s//\\/\\\\}` 等）对 SKILL.md 内容做 JSON 转义，而不是借助 `jq` 或 Python。优点：零外部依赖，速度快一个数量级。自查：我的 hook 有没有类似的零依赖 JSON 转义？
2. **伴生 .md 文件**：复杂 skill（systematic-debugging、writing-skills 等）把深度参考内容拆到同目录的伴生 .md，主 SKILL.md 保持简洁。优点：主文件可读性好，详细内容按需通过 `@` 加载。
3. **多平台单一内容**：20 个 SKILL.md 内容完全平台无关，三套 manifest 只有路径声明不同。优点：维护成本低，内容一致。
4. **中文原创 skill 不依赖英文骨架**：`chinese-code-review`、`chinese-commit-conventions` 等 6 个原创 skill 完全自足，不引用任何英文假设。优点：可被没有安装上游 superpowers 的用户单独使用。

### 3.3 当时的缺陷
1. **3 个 deprecated 命令缺 `name` [frontmatter](#frontmatter)**：根本原因是 deprecated stub 被认为是「过渡性文件」而放宽了规范检查，但即使是过渡文件，缺 `name` 意味着用户无法通过命令面板发现它们，迁移引导失效。自查：我的 deprecated 文件有没有同样问题？
2. **code-reviewer agent 使用模糊量词「恰当/适当」**：根本原因是翻译时保留了原文语气词而未量化。「检查错误处理是否恰当」无法给模型提供可操作的标准。自查：我的 SKILL.md 有没有类似的翻译腔量词？
3. **孤立的 `@`-引用**：审计时约 10 个伴生 .md 文件（`root-cause-tracing.md`、`anthropic-best-practices.md` 等）仅在上游存在，未随汉化一起迁移。根本原因：汉化只移植了 SKILL.md 主文件，伴生参考文件被遗漏。自查：我的 skill 有没有引用了不存在的文件？

### 3.4 当时的优化机会
1. `agents/code-reviewer.md` 的 3 处模糊量词替换为可量化标准（如「检查错误处理：验证所有外部调用有显式 try/catch 或 Result 类型包装」）
2. 三套 manifest 合并为单一源脚本生成
3. 补齐约 10 个未迁移伴生文件，或替换为内联内容

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个命令缺 `name` frontmatter | `ls commands/` | **已修复**：整个 `commands/` 目录已不存在（deprecated 命令被彻底删除） | 作者选择删除过渡文件而非补全 frontmatter |
| Node 版本约束过宽 `>=14.0.0` | `grep node package.json` | **已修复**：已升级为 `>=20.0.0`（比建议的 18 更激进） | 积极维护，EOL 版本风险消除 |
| 约 10 个孤立 `@`-引用 | `ls skills/systematic-debugging/` | **已修复**：伴生文件已全部补齐（root-cause-tracing.md、defense-in-depth.md、anthropic-best-practices.md 等均存在） | 翻译工作已补完，@-链路完整 |

### 4.2 架构演进
相较审计快照，当前 HEAD 主要变化：
- `commands/` 目录整体消失（3 个 deprecated stub 删除）
- `agents/` 目录整体消失（`code-reviewer` agent 被删除，可能功能被合并进某个 skill）
- 新增 `.codex-plugin/`、`.opencode/`、`.cursor-plugin/` 等额外平台适配目录
- 18 款支持工具（README 里列出：Kiro、Gemini CLI、DeerFlow、Qwen Code 等均为审计后新增）

作者后来意识到：与其维护 deprecated 桩文件和 agent，不如直接删除，用更纯粹的 skill 平铺结构取代。

### 4.3 新增的可学习模式
**多平台统一 INSTALL 指南**：新增 `.codex/INSTALL.md`、`.opencode/INSTALL.md` 等平台专属安装说明，与 manifest 文件共存于同一目录。模式：「每个平台适配包包含 manifest + INSTALL，skill 内容共用」——非常干净的多平台发布策略。

---

## 五、校准

### 5.1 我已经在做对的
1. **skill 单职责**：我的 echo-sleuth-for-claude 的每个 skill（`git-mining`、`experience-synthesis`、`jsonl-core`）各管一件事，与 superpowers-zh 的单 skill 对应单场景完全一致。
2. **伴生参考文件**：`echo-sleuth-for-claude` 的 `skills/git-mining/references/git-commands.md`、`skills/jsonl-core/references/record-types.md` 等都是伴生参考文件的正确用法。
3. **多平台无关内容**：我的 SKILL.md 内容不包含平台特定语法，可以被 Cursor/Codex 等工具复用。

### 5.2 挑战 / 验证
这次案例**验证了我之前犹豫的一个做法**：SessionStart hook 直接嵌入 SKILL.md 全文内容是可行的，而不是只注入一个路径引用。superpowers-zh 实战证明了这一方案的稳定性（最低 Node 要求、JSON 转义的可靠性）。我之前对 hook 注入完整内容是否可靠有顾虑，本案例解除了这个顾虑。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否有模糊量词（含中文）
grep -rn -E '\b(appropriate|comprehensive|robust|efficient)\b|恰当|适当|合理' \
  /tmp/my-repos/MarkQWu-*/skills/ --include="SKILL.md"
# 命中后：把模糊量词替换为可量化标准
```

```bash
# 检查我的 skill 中有没有 @ 引用不存在的文件
grep -rn "^@" /tmp/my-repos/MarkQWu-*/skills/ --include="SKILL.md" | while IFS=: read f l ref; do
  target="$(dirname $f)/${ref#@}"
  [ ! -f "$target" ] && echo "ORPHAN: $f:$l → $target"
done
# 命中后：要么补充缺失文件，要么改为内联内容
```

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 增加一个 SessionStart hook，自动在会话开始时注入 `memory-management` skill 内容
   - **为何可行**：superpowers-zh 实战验证了 bash JSON 转义注入方案的可靠性
   - **第一步**：参考 superpowers-zh 的 `hooks/session-start` 脚本，复制其 JSON 转义函数，15 分钟可完成

2. **想法**：把 drama-workshop-skills 的多个命令 reference 文件迁移为 skill 下的伴生 .md，用 `@` 语法按需加载
   - **为何可行**：superpowers-zh 已验证这种结构在 18 款工具上稳定工作
   - **第一步**：挑选一个命令（如 `/策划`）把其内联参考拆到伴生 .md，测试是否正常加载，30 分钟可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（含 MarkQWu/drama-workshop-skills、MarkQWu/claude-for-legal、MarkQWu/echo-sleuth-for-claude）

### 7.1 目的对齐度

- **本案例 jnMetaCode/superpowers-zh 的核心目的**：向 AI 编程工具注入开发方法论（TDD、子 agent、代码审查流程），以 skill 集合形式分发，支持多工具平台。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为 skill 集合，平台无关内容，单职责 skill | 领域不同（剧本 vs 编程），drama 有更复杂的 command 层 | 高 |
| MarkQWu/echo-sleuth-for-claude | 高 | 同为 skill 集合，有伴生参考 .md，支持多工具 | echo-sleuth 有 agent 层，superpowers-zh 无 | 高 |
| MarkQWu/claude-for-legal | 低 | 同为 skill 集合 | 法律领域，有复杂 practice-area 层级，不是纯粹编程工作流 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| code-reviewer agent 中模糊量词「适当/恰当」 | `grep -rn '适当\|恰当\|appropriate' /tmp/my-repos/MarkQWu-*/skills/*.md` | claude-for-legal 命中多处（`matter-workspace/SKILL.md:180`、`litigation-legal/skills/matter-workspace/SKILL.md:184` 等），但这些出现在法律文书的客观描述中，语境合理，非模型指令场景 | 低 |
| 孤立 @-引用 | `grep -rn "^@" /tmp/my-repos/MarkQWu-*/skills/` | **0 命中**：我的 skill 未使用 @-语法 | 无 |

**命中后具体行动**：`claude-for-legal/regulatory-legal/skills/matter-workspace/SKILL.md:180` 中「Decide whether cross-matter is appropriate」虽然用了 appropriate，但语境是给用户看的说明文字，不是给模型的行为指令，可暂时保留。

### 7.3 别人的更优方案

1. **领域**：SessionStart hook 自动注入元 skill
   - **本案例做法**：`hooks/session-start` 读取 `using-superpowers/SKILL.md` 全文，JSON 转义后作为 `pre_prompt` 注入每次会话（`skills/using-superpowers/SKILL.md`）
   - **我的项目现状**：`echo-sleuth-for-claude` 无 SessionStart hook，用户必须手动 `/memory-management` 才能激活 skill
   - **如何借鉴**：新建 `hooks/session-start`（参考 superpowers-zh 的脚本），读取 `skills/memory-management/SKILL.md`，JSON 转义注入；在 `plugin.json` 中注册 hook

2. **领域**：伴生参考文件 + @ 语法按需加载
   - **本案例做法**：`skills/systematic-debugging/` 下有 `root-cause-tracing.md`、`defense-in-depth.md` 等，主 SKILL.md 通过 `@root-cause-tracing.md` 按需加载
   - **我的项目现状**：`echo-sleuth-for-claude/skills/git-mining/references/` 有参考文件但未用 @ 语法加载，只靠 skill 正文内联引用
   - **如何借鉴**：在 `git-mining/SKILL.md` 中添加 `@references/git-commands.md` 加载语句，把当前内联的 git 命令速查表移出主文件

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：[agent](#agent) 层分离
- **我的做法**：`echo-sleuth-for-claude/agents/memory-auditor.md` 把需要跨 skill 综合判断的任务（内存审计）专门封装为 agent，与各 skill 清晰分工
- **本案例做法**：审计时有 `agents/code-reviewer.md`，但当前版本已删除，功能被合并（或弃用）。superpowers-zh 最终选择了纯 skill 平铺，没有 agent 层
- **意义**：对于需要读多个文件、综合推理的任务，专属 agent 比 skill 更合适；我的 echo-sleuth 在这个维度上更完整

---

## 八、术语表

### <a name="skill"></a>skill
> Claude Code 的知识单元，一个 Markdown 文件（SKILL.md）包含给 AI 的具体操作指南。通过 `claude skill add` 安装到 `~/.claude/skills/`，AI 在对话中可通过工具调用激活。类比：像插件，但内容是自然语言说明书而非代码。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 如何注册和调用。缺少 `name` 字段时，该 skill 无法通过名称被发现。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉 AI 工具这个插件包含哪些 skill、command、agent。`plugin.json` 是 Claude Code 插件的 manifest。manifest 里漏写的文件不会被加载，即使文件在磁盘上存在。

### <a name="sessionstart-hook"></a>SessionStart hook
> Claude Code 的 hook 类型之一，在每次新会话开始时自动执行一段 bash 脚本。superpowers-zh 用它在每次会话开始时把 `using-superpowers` 的内容注入上下文，确保模型知道 skill 体系的存在，无需用户手动提示。

### <a name="agent"></a>agent
> 一种特殊的 Claude Code 产物，用于封装需要多步骤、多工具调用的复杂任务。与 skill 的区别：skill 是参考知识，agent 是任务执行者，有自己的 `model`、`tools` 声明，可被其他 command 或 skill 调度。
