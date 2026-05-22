# anthropics/claude-code — 学习案例

**仓库**：https://github.com/anthropics/claude-code
**Stars**：125,529 | **来源**：本地 audit（NOT in upstream manifest）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`router-channels`, `manifest-discipline`, `vague-quantifier`, `examples-driven`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
这是 Anthropic 官方发布的 Claude Code 插件参考实现——即 Claude Code 工具本身的作者，用自己的工具写的示例插件集合。125,529 颗星，是所有已审查插件仓库中 star 数最高的。

关键事实：
1. **作者即工具制造商**：Anthropic 是 Claude Code 的开发团队，这个仓库是他们展示「如何正确写插件」的官方示例，具有最高的参考权威性
2. **14 个独立插件，一个 monorepo**：所有插件存放在 `plugins/` 目录下，每个插件是独立的 `.claude-plugin/` 完整结构，可单独安装，互不依赖
3. **用户获取方式**：通过 `claude plugin install <name>@anthropics --scope project` 安装，插件名对应 `plugins/` 下的子目录名
4. **在生态中的位置**：这是 Claude Code [manifest](#manifest) 规范、[frontmatter](#frontmatter) 约定、命令/代理/技能/钩子结构的官方标杆，也是 NLPM 评分规则的隐含参照对象
5. **本次 audit 为本地独立审查**：未进入 xiaolai 上游 manifest，没有对应的 xiaolai 案例

### 1.2 架构剖析
- **目录结构**：
```
claude-code/
  plugins/
    agent-sdk-dev/          # SDK 开发辅助：verifier agents + new-sdk-app command
    claude-opus-4-5-migration/  # 模型迁移指南 skill（audit 后新增）
    code-review/            # 代码审查：单命令 + MCP 集成（待配置）
    commit-commands/        # Git 提交三件套：commit / commit-push-pr / clean_gone
    explanatory-output-style/   # 解释型输出风格 hook（SessionStart）
    feature-dev/            # 功能开发工作流：explorer + reviewer + architect agents
    frontend-design/        # 前端设计 skill
    hookify/                # Hook 生成框架：config_loader + rule_engine + 4 hook 类型
    learning-output-style/  # 学习型输出风格 hook（SessionStart）
    plugin-dev/             # 插件开发指南：7 个 skill + creator + validator agents
    pr-review-toolkit/      # PR 审查工具包：5 个专项 agents
    ralph-wiggum/           # AI 自主循环框架（ralph-loop）
    security-guidance/      # 安全提醒 hook
  .claude/
    commands/               # 仓库级别命令（triage-issue, commit-push-pr, dedupe）
  examples/                 # Hook 示例脚本
  scripts/                  # GitHub 自动化脚本
```
- **文件类型分布**：60 个 NL 工件（audit 时）——包含约 22 个 agent、12 个 command、9 个 skill、7 个 hook JSON 配置；另含多个 Python 和 Bash 执行脚本
- **编排关系**：每个插件内部是「命令 → 代理」或「命令 → 技能」的分层结构。插件之间互相独立（没有跨插件引用），不存在 router meta-plugin。hookify 是这 14 个中架构最复杂的，内部有「命令 → conversation-analyzer 代理（应该如此，但实际存在绕过）→ 规则引擎 → hook 输出」的四层结构
- **跨件契约**：同一插件内通过 [manifest](#manifest)（`.claude-plugin/plugin.json`）声明组件路径；代理通过 frontmatter 中的 `name:` 字段注册到全局命名空间；`allowed-tools` 字段是命令与底层工具之间的契约；`color:` 字段是代理的 UI 标识约定

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单仓多插件矩阵」——用一个 monorepo 托管 14 个功能正交、独立可安装的插件，每个插件是一个完整的「[manifest](#manifest) + 组件」单元，遵循统一的目录约定
- **解决什么问题**：演示 Claude Code 插件的全部能力维度（命令、代理、技能、钩子、Python 脚本集成、MCP 集成），以及多场景下的架构选择；同时作为社区开发者的参考实现
- **做了什么 trade-off**：
  - 选 monorepo 而非分仓：便于演示多种模式，代价是插件之间命名空间容易产生冲突（如两个 `code-reviewer` 代理）
  - 选「插件独立」而非「插件协作」：降低耦合，代价是无法展示跨插件工作流
  - hookify 的可配置规则引擎设计（`config_loader.py` + `rule_engine.py`）选择了工程化程度高的方案，代价是复杂度远超其他插件
- **反映什么认知模型**：Anthropic 把「插件」视为独立的功能单元，每个单元有明确的职责边界；把「代理」视为命令的执行核心，命令负责路由，代理负责专业判断

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「官方多插件宫殿」——单仓多插件矩阵（Official Multi-Plugin Palace）**

这个模式的核心特征：一个仓库内托管多个功能正交的完整插件，每个插件有独立的 manifest 和完整的命令/代理/技能/钩子层级，插件之间无依赖，可单独安装。

模式特征清单（5 条）：
- 特征 1：每个插件是「一个独立的 `.claude-plugin/plugin.json` + 完整组件目录」——manifest 是独立性的边界
- 特征 2：插件间功能正交（hookify 管钩子生成、feature-dev 管功能开发、pr-review-toolkit 管 PR 审查）——没有功能重叠
- 特征 3：共享 monorepo 仓库但独立安装——用户按需选择插件，不需要全部安装
- 特征 4：代理作为命令的专业执行核心——命令是路由入口，代理是领域专家（除了 hookify 的一个绕过 bug）
- 特征 5：有实际可执行的 Python/Bash 层——部分插件（hookify、security-guidance）在 NL 描述层之下有真实代码核心，是「NL 表皮 + 脚本核心」的混合模式

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 官方参考实现 / 标杆展示 | ✅ 高度适用 | 多插件覆盖多种模式，用户可按需参考对应插件 |
| 团队工具箱（多个独立工具） | ✅ 适用 | 统一仓库便于版本管理，独立安装便于按需使用 |
| 单一功能的社区插件 | ❌ 过度工程 | 14 个插件的 monorepo 结构对单一功能来说太重 |
| 插件之间需要紧密协作的工作流 | ❌ 不适用 | 插件独立设计，跨插件调用没有内置机制 |
| 个人快速实验 / 原型 | ❌ 不适用 | manifest 维护负担高，容易出现命名冲突 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单仓多插件矩阵（本仓库） | anthropics/claude-code | 统一维护，独立安装，覆盖多种模式 | manifest 同步负担高，跨插件命名空间易冲突 |
| 单插件单仓 | MarkQWu/echo-sleuth-for-claude | 结构简单，版本独立，README 清晰 | 多插件时需维护多个仓库，发现成本高 |
| 无 manifest 平铺 | 0xfurai/claude-code-subagents | 零配置，clone 即用 | 无法发布 marketplace，无法有命令/技能层 |

### 2.4 改进空间
1. **当前问题**：`hookify` 命令在步骤文字里声称「Launch the conversation-analyzer agent」，但实际用 `"subagent_type": "general-purpose"` 绕过了专用代理。**改进做法**：把 Task 调用里的 `subagent_type` 改为 `"subagent_type": "agent"` 并指定 `agent_name: "conversation-analyzer"`，让路由真正指向插件内的专用代理。**预期收益**：消除跨件契约的不一致，使 conversation-analyzer 的专业分析能力生效。
2. **当前问题**：`feature-dev` 和 `pr-review-toolkit` 两个插件各自有一个 `name: code-reviewer` 代理，同时安装时后者会遮蔽前者。**改进做法**：将 `feature-dev/agents/code-reviewer.md` 重命名为 `name: feature-code-reviewer`，并更新 feature-dev 命令的引用。**预期收益**：消除全局命名冲突，两个插件可以共存。
3. **当前问题**：`feature-dev`、`agent-sdk-dev` 中的 5 个代理全部缺少 `<example>` 块，导致代理无法自动触发。**改进做法**：给每个代理加一个最小示例对（用户输入 → 代理输出摘要），不需要复杂内容，3-5 行即可。**预期收益**：代理自动触发率提升，NLPM 评分从 73-77 跳到约 88-92。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **88/100**（60 个工件，14 个插件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agent-sdk-dev/agents/agent-sdk-verifier-ts.md | 73 | 无 `<example>` 块；缺 `color` 字段 |
| agent-sdk-dev/agents/agent-sdk-verifier-py.md | 73 | 无 `<example>` 块；缺 `color` 字段 |
| feature-dev/agents/code-explorer.md | 75 | 无 `<example>` 块；模糊量词 ×5 |
| feature-dev/agents/code-architect.md | 75 | 无 `<example>` 块 |
| feature-dev/agents/code-reviewer.md | 77 | 无 `<example>` 块 |
| pr-review-toolkit/agents/code-simplifier.md | 82 | 缺输出格式说明；无 `color` |
| plugin-dev/agents/plugin-validator.md | 83 | 文末遗留编辑对话文本，污染 system prompt |
| feature-dev/commands/feature-dev.md | 83 | 无 `allowed-tools`；模糊语言 |
| hookify/commands/hookify.md | 92 | 绕过 conversation-analyzer，用通用子代理替代 |
| 大多数其他工件 | 88-95 | 偶发模糊量词（appropriate/specific/relevant） |

### 3.2 当时值得借鉴的模式
1. **hookify 的四层规则框架**（router-channels）→ 为什么好：`config_loader.py` 解析 `.claude/hookify.*.local.md` 配置，`rule_engine.py` 执行规则匹配，4 种钩子类型（PreToolUse/PostToolUse/Stop/UserPromptSubmit）各有专用脚本——这是「NL 配置层 + Python 执行层」的最完整示范。→ 原文路径：`plugins/hookify/core/config_loader.py`、`plugins/hookify/core/rule_engine.py`、`plugins/hookify/hooks/` → 如何借鉴：当自己的钩子逻辑超过 50 行时，考虑拆出 config_loader 和 rule_engine 两个模块，而不是把所有逻辑堆在单个 hook 脚本里。

2. **security-guidance 的安全优先设计**（security-gate）→ 为什么好：在 PreToolUse 上挂载 `security_reminder_hook.py`，匹配 `Edit|Write|MultiEdit`——每次写文件前自动提示安全注意事项，把安全检查内嵌到工作流而不是依赖人工记忆。→ 原文路径：`plugins/security-guidance/hooks/hooks.json`、`plugins/security-guidance/hooks/security_reminder_hook.py` → 如何借鉴：法律、金融、医疗等高风险领域的插件可以用同样的 PreToolUse 钩子模式，在关键操作前强制弹出合规提醒。

3. **commit-commands 的极简命令设计**（single-purpose）→ 为什么好：三个命令（`commit`、`commit-push-pr`、`clean_gone`）各自只做一件事，`allowed-tools` 精确声明到 `Bash(git add:*)` 这个级别，权限最小化且意图清晰。→ 原文路径：`plugins/commit-commands/commands/commit.md` → 如何借鉴：`allowed-tools` 的值不应该只写 `Bash`，应该写到 `Bash(git commit:*)` 这种细粒度，既是文档也是安全约束。

4. **manifest 守护的插件隔离**（manifest-discipline）→ 为什么好：每个插件有独立的 `.claude-plugin/plugin.json`，声明了该插件所有组件的路径——这使得插件可以独立安装、独立卸载，不影响其他插件。→ 原文路径：所有 `plugins/*/.claude-plugin/plugin.json` → 如何借鉴：多插件仓库里一定要给每个插件单独维护 manifest，不要共用一个顶层 plugin.json。

5. **claude-opus-4-5-migration 的模型迁移 skill**（model-pinning）→ 为什么好：提供了从 Claude Opus 4.5 迁移的结构化指南，以 skill 形式封装，可以被任何命令按需加载——这说明「迁移路径」本身可以是一个可复用的知识单元。→ 原文路径：`plugins/claude-opus-4-5-migration/skills/claude-opus-4-5-migration/SKILL.md` → 如何借鉴：当模型版本升级时，不要修改现有代理，而是新增一个迁移 skill，保持向后兼容。

### 3.3 当时的缺陷
1. **plugin-validator.md 遗留对话文本**：文件末尾出现 "Excellent work! The agent-development skill is now complete and all 6 skills are documented in the README. Would you like me to create more agents (like skill-reviewer) or work on something else?" ——这是写文件时 Claude 对话残留，被当作 system prompt 内容加载进去。为什么这么设计会失败：agent 文件的全部内容都会作为 system prompt 注入，conversational 语气的结尾会让代理在每次调用时都倾向于生成类似的「完成确认」文本，污染实际输出。**自查**：我的 agent 或 skill 文件末尾有没有写作时留下的对话片段？→ 检查命令：`grep -n "Would you like\|Excellent work\|Feel free to\|Let me know" ~/.claude/agents/*.md ~/.claude/skills/*/SKILL.md 2>/dev/null`。

2. **代理缺少 `<example>` 块**：feature-dev 的 code-explorer、code-reviewer、code-architect 和 agent-sdk-dev 的两个 verifier 代理，全部没有 `<example>` 块。为什么会失败：Claude Code 的代理自动触发机制依赖 description 中的 `<example>` 来学习「在什么情况下调用这个代理」；没有示例的代理只能被用户显式 `@mention` 调用，丧失了自动路由能力。**自查**：`for f in ~/.claude/agents/*.md; do grep -q "<example>" "$f" || echo "MISSING: $f"; done`。

3. **invalid color 值**：`pr-review-toolkit/agents/type-design-analyzer.md` 的 frontmatter 里写了 `color: pink`，而 Claude Code 只支持 blue/cyan/green/yellow/magenta/red 六种颜色。为什么会失败：无效的 color 值可能导致插件注册失败或 UI 渲染降级，且这个错误不会在安装时抛出明显提示，只会在用 UI 查看代理时发现异常。**自查**：`grep -rn "^color:" ~/.claude/agents/*.md 2>/dev/null | grep -v "blue\|cyan\|green\|yellow\|magenta\|red"`。

4. **跨插件命名空间冲突**：feature-dev 和 pr-review-toolkit 两个插件各有一个 `name: code-reviewer`，同时安装时后安装的会遮蔽先安装的。为什么会失败：Claude Code 的代理注册是全局命名空间，同名代理会互相覆盖，用户无法预知哪个版本生效，调试困难。**自查**：`grep -rn "^name:" ~/.claude/agents/*.md 2>/dev/null | awk -F: '{print $3}' | sort | uniq -d`（找重复的 name 值）。

5. **hookify 命令绕过专用代理**：`hookify.md` 命令的步骤文字声称 "Launch the conversation-analyzer agent" 但 Task 调用里用的是 `"subagent_type": "general-purpose"` 内联提示，而非指向 `conversation-analyzer` 代理。为什么会失败：命令说明和实际行为不一致，产生文档欺骗；通用子代理不具备 conversation-analyzer 的专项分析能力，输出质量低于预期。**自查**：检查自己的命令文件里有没有「步骤文字说调用某代理，但 Task 实际用 general-purpose」的不一致。

### 3.4 当时的优化机会
1. **修复 plugin-validator.md 的尾部对话文本**：删掉最后那段 conversational 文字，一分钟能完成，立即消除 system prompt 污染。这是 7 个 bug 里影响最严重的——代理每次调用都会产生错误行为，而不只是静默失效。
2. **给 5 个缺 example 的代理各加一个最小 `<example>` 块**：每个 agent 加 3-5 行示例，5 个文件合计约 30 分钟工作，让这 5 个代理从「只能显式调用」变为「可自动触发」，直接影响可用性。
3. **把 feature-dev 的 code-reviewer 重命名为 feature-code-reviewer**：修改 frontmatter 的 `name:` 字段和 feature-dev 命令中的代理引用，消除跨插件命名冲突，使两个插件可以安全共存。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin-validator.md 尾部对话文本 | `tail -5 plugins/plugin-dev/agents/plugin-validator.md` | **持续存在**：末尾仍为 "Excellent work! The agent-development skill is now complete…Would you like me to create more agents" | audit 发出 6 周后仍未修复，说明 Anthropic 团队没有把这个 bug 列入近期迭代 |
| type-design-analyzer.md `color: pink` | `grep -n "color" plugins/pr-review-toolkit/agents/type-design-analyzer.md` | **持续存在**：第 5 行仍为 `color: pink` | 同样 6 周无修复，invalid color 继续存在 |
| code-explorer.md 无 example 块 | `grep -c "example" plugins/feature-dev/agents/code-explorer.md` | **持续存在**：count = 0，无任何 example 相关内容 | 5 个缺 example 的代理全部无变化 |
| 双 code-reviewer 命名冲突 | `grep -rn "name: code-reviewer" plugins/` | **持续存在且扩大**：feature-dev、pr-review-toolkit 各一个，另在 plugin-dev/skills/agent-development/examples/complete-agent-examples.md 有第三处引用 | 冲突范围比 audit 时认知的更广 |

### 4.2 架构演进
从 audit 时的文件清单 → 现在的目录结构：

**新增**：`claude-opus-4-5-migration` 插件（audit 时未见）。这是一个纯 skill 插件（`skills/claude-opus-4-5-migration/SKILL.md`），专门提供从 Claude Opus 4.5 向后续版本迁移的指南。这个新增说明 Anthropic 在 audit 后（2026-04-06）把「模型迁移支持」正式纳入了插件体系，侧面印证了 [模型版本固定](#model-pinning) 的重要性——版本会变，迁移路径需要有文档化的知识载体。

**未变**：其余 13 个插件结构无变化，7 个 bug 全部持续存在，3 个跨件问题无改动。

**意味着什么**：Anthropic 在这 6 周内优先级是「新增功能（模型迁移 skill）」而非「修复已知 bug」。这对我们的学习非常有启示：即使是工具的原作者，面对「修 bug vs 加功能」的取舍时，也会选择加功能——这不是坏事，只是说明 bug 修复需要外部压力（如用户反馈、CI 检查）才会被驱动。

### 4.3 新增的可学习模式
`claude-opus-4-5-migration` 引入了**「模型迁移 skill」模式**：当大语言模型版本升级时，不修改现有代理的 frontmatter，而是新增一个独立的 skill 文件，封装迁移注意事项、API 差异、prompt 适配建议——其他命令可以通过 `Skill` 工具按需加载这个知识单元。这个模式的关键价值是**向后兼容**：旧代理继续运行，新 skill 提供升级路径，不强迫所有用户立即迁移。

---

## 五、校准

### 5.1 我已经在做对的
1. **使用 manifest 声明组件**：我的 echo-sleuth 有完整的插件结构（虽然是单插件），manifest 同步是我已在维护的习惯，和本仓库的 manifest-discipline 模式一致。
2. **命令做路由、代理做专业判断的分层**：我的 echo-sleuth 里，commands 负责参数解析和流程控制，agents（analyze、recall 等）负责领域知识——和本仓库的分层设计哲学吻合。
3. **代理有 `<example>` 块**：echo-sleuth 的 5 个代理全部有 example 块（每个 2-4 个），这点比本仓库做得好——本仓库 feature-dev 和 agent-sdk-dev 的 7 个代理到今天仍然缺 example。
4. **无跨插件命名冲突**：单插件结构从根本上避免了本仓库的命名空间问题。
5. **命令文件内容与行为一致**：我没有「步骤说调用代理 X，实际用 general-purpose」这类不一致。

### 5.2 挑战 / 验证
**这次案例挑战了一个我的重要假设**：「Anthropic 官方仓库应该是最高质量的参照标准，所有做法都值得直接复制」。

实际上，Anthropic 官方仓库 88/100，有 7 个未修复 bug，其中包括 `color: pink` 这个肉眼可见的 invalid 值，6 周后仍持续存在。最低分的两个代理是 73/100——这比很多社区插件还低。

这个发现的意义不在于批评 Anthropic，而在于**验证了一个更重要的认知**：NL 工件的质量问题不是「初学者才犯的错误」，即使是工具的原作者在日常写作中也会留下对话残留文本、用 invalid 的 color 值、忘记加 example 块。这说明 NL 编程的质量问题是**结构性的**，需要自动化工具（如 NLPM）来系统性检测，不能依赖「只要作者足够专业就不会犯错」的假设。

对我的直接影响：我应该把 NLPM 评分检查加入自己项目的 pre-commit 或 CI 流程，而不是靠人工 review 来捕捉这类问题。

---

## 六、行动

### 6.1 自查动作
```bash
# 1. 检查我的 agent 或 skill 文件末尾是否有对话残留文本
grep -rn "Would you like\|Excellent work\|Feel free to\|Let me know if\|anything else" \
  ~/.claude/agents/*.md ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：找到对应文件，删除文件末尾的 conversational 语气段落，只保留指令性内容

# 2. 检查所有 agent 是否有 <example> 块
for f in ~/.claude/agents/*.md; do
  grep -q "<example>" "$f" || echo "MISSING EXAMPLE: $f"
done
# 命中后：给每个文件加至少一个 <example>...</example> 块，展示典型的用户输入 → 代理行为

# 3. 检查 color 字段是否使用合法值
grep -rn "^color:" ~/.claude/agents/*.md 2>/dev/null | \
  grep -v "blue\|cyan\|green\|yellow\|magenta\|red"
# 命中后：把非法 color 值替换为 blue/cyan/green/yellow/magenta/red 中的一个

# 4. 检查是否有重复 name 值（命名冲突）
grep -rn "^name:" ~/.claude/agents/*.md 2>/dev/null | \
  awk -F': ' '{print $2}' | sort | uniq -d
# 命中后：重命名其中一个，并更新引用它的命令文件

# 5. 检查 commands/ 里是否缺少 allowed-tools
for f in ~/.claude/commands/*.md; do
  grep -q "allowed-tools" "$f" || echo "MISSING allowed-tools: $f"
done
# 命中后：在 frontmatter 里加 allowed-tools，精确到 Bash(git commit:*) 级别

# 6. 检查 hookify 风格的「说调用代理 X，实际用 general-purpose」不一致
grep -rn "general-purpose" ~/.claude/commands/*.md ~/.claude/agents/*.md 2>/dev/null
# 命中后：如果步骤文字里写了调用某个具名代理，把 subagent_type 改为对应代理名
```

### 6.2 灵感 → 实施路径
1. **想法**：在 echo-sleuth 的 4 个缺 `allowed-tools` 的命令（lessons、timeline、recap、recall）里补全 `allowed-tools`
   - **为何可行**：这 4 个命令使用了 Bash（读取 session 文件）和 Read 工具，添加 `allowed-tools` 声明只需修改 frontmatter，不需要改功能逻辑；且本仓库的 commit-commands 展示了精确 `allowed-tools` 声明的最佳实践
   - **第一步**：读取每个命令文件，确认它实际使用了哪些工具（看 `!`bash 内联命令和 Tool 调用），然后在 frontmatter 加对应的 `allowed-tools: [...]`；4 个文件合计约 20 分钟

2. **想法**：为 echo-sleuth 添加一个「security-guidance 风格」的合规提醒钩子，在写入 `.jsonl` 会话文件前提醒用户注意隐私
   - **为何可行**：echo-sleuth 处理对话历史，可能涉及敏感内容；本仓库的 security-guidance 插件展示了一个完整的 PreToolUse 钩子实现（`hooks.json` + Python 脚本），可以直接仿写
   - **第一步**：在 echo-sleuth 仓库新建 `hooks/hooks.json`（匹配 Write 工具，过滤 `.jsonl` 文件路径），编写 `hooks/privacy_reminder.py`（检测写入目标是否为会话文件，是则打印隐私提醒）；约 45 分钟

3. **想法**：把 NLPM 评分检查加入 echo-sleuth 的 pre-commit hook
   - **为何可行**：本仓库的 6 周 bug 未修复说明没有 CI 压力就没有修复动力；`bin/nlpm-check` 是纯 Python stdlib 的独立二进制，可以直接用在 pre-commit 里，templates/pre-commit-nlpm.sh 是现成模板
   - **第一步**：把 `templates/pre-commit-nlpm.sh` 复制到 echo-sleuth 的 `.git/hooks/pre-commit`，设置 `chmod +x`；约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（最后刷新：2026-05-21）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 anthropics/claude-code 的核心目的**：提供 Claude Code 插件的官方参考实现，涵盖命令/代理/技能/钩子的完整设计模式，供社区开发者学习和复用

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同样有 commands+agents+skills 三层架构；同样用 manifest 管理组件；同样有命令→代理的路由模式 | echo-sleuth 是单插件；功能目的不同（挖掘会话历史 vs 展示插件模式）；echo-sleuth 代理有 example，anthropics 部分没有 | 高 |
| MarkQWu/claude-for-legal | 低 | 同样是多功能插件集合的思路 | claude-for-legal 没有代理层；法律领域 vs 开发工具领域 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同样使用 skill 文件格式 | drama-workshop-skills 是单一 skill，无命令/代理层 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 代理缺 `<example>` 块（7 个代理） | `for f in /tmp/echo-sleuth/agents/*.md; do grep -q "<example>" "$f" \|\| echo "MISSING: $f"; done` | **未命中**：echo-sleuth 5 个代理全部有 `<example>` 块（analyze: 4 个，recall: 4 个，其余各 2 个） | N/A |
| 命令缺 `allowed-tools` | `for f in /tmp/echo-sleuth/commands/*.md; do grep -q "allowed-tools" "$f" \|\| echo "MISSING: $f"; done` | **命中 4 个**：lessons.md、timeline.md、recap.md、recall.md 四个命令缺少 `allowed-tools` 声明 | 中 |
| 无效 color 值 | `grep -rn "^color:" /tmp/echo-sleuth/agents/*.md \| grep -v "blue\|cyan\|green\|yellow\|magenta\|red"` | **未命中**：echo-sleuth 代理的 color 值全部合法 | N/A |
| 跨插件命名冲突 | `grep -rn "^name:" /tmp/echo-sleuth/agents/*.md \| awk -F': ' '{print $2}' \| sort \| uniq -d` | **未命中**：单插件结构，5 个代理名称各不相同 | N/A |
| 文件末尾遗留对话文本 | `grep -rn "Would you like\|Excellent work" /tmp/echo-sleuth/agents/*.md /tmp/echo-sleuth/commands/*.md` | **未命中**：echo-sleuth 无此类残留 | N/A |

**命中后的具体行动建议**：
- `echo-sleuth/commands/lessons.md` → 在 frontmatter 加 `allowed-tools: ["Read", "Bash(ls:*)"]`（该命令用 Read 工具读 jsonl 文件和 ls 列出 session 目录）→ 约 5 分钟可完成
- `echo-sleuth/commands/timeline.md` → 加 `allowed-tools: ["Read", "Bash(git log:*)", "Bash(ls:*)"]`（该命令结合 Claude session 历史和 git log）→ 约 5 分钟
- `echo-sleuth/commands/recap.md` → 加 `allowed-tools: ["Read", "Bash(ls:*)"]`（读 session 文件汇总）→ 约 5 分钟
- `echo-sleuth/commands/recall.md` → 加 `allowed-tools: ["Read", "Bash(grep:*)"]`（搜索 session 文件内容）→ 约 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：精确 `allowed-tools` 声明（细粒度白名单）
   - **本案例做法**：`commit-commands/commands/commit.md` 的 `allowed-tools` 声明为 `Bash(git add:*), Bash(git status:*), Bash(git commit:*)`——精确到具体的 git 子命令，而不是宽泛的 `Bash`
   - **我的项目现状**：echo-sleuth 有 `allowed-tools` 的命令（audit.md、dashboard.md、extract.md）都只写到 `Bash` 级别，没有限制到具体子命令
   - **如何借鉴**：读取 echo-sleuth 每个命令实际用到的 bash 命令，把 `allowed-tools` 里的 `"Bash"` 改为 `"Bash(git log:*)"` 等细粒度形式；这既是安全约束也是使用文档

2. **领域**：Python 规则引擎作为 hook 的执行核心
   - **本案例做法**：hookify 的 `core/config_loader.py` + `core/rule_engine.py` 把 NL 配置文件（`.local.md`）解析为规则，由 Python 引擎执行匹配——NL 层负责配置表达，Python 层负责可靠执行
   - **我的项目现状**：echo-sleuth 目前没有 hook 层；如果加 privacy_reminder hook，大概会直接把逻辑写在单个 Python 脚本里
   - **如何借鉴**：如果 hook 逻辑超过 30 行，参考 hookify 的分层拆出 `config_loader.py`（读取 `.local.md` 配置）和 `rule_engine.py`（执行规则）；维护成本会明显降低

3. **领域**：模型迁移 skill 的独立封装
   - **本案例做法**：`claude-opus-4-5-migration` 是一个独立插件，把迁移知识封装为 skill，不修改现有代理
   - **我的项目现状**：echo-sleuth 目前没有显式的模型版本管理，升级模型时会直接修改各代理的 frontmatter `model:` 字段
   - **如何借鉴**：下次需要迁移模型时，新建 `skills/model-migration/SKILL.md`，记录迁移注意事项和 diff，让旧代理继续工作，需要升级的用户按需加载这个 skill

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：代理 `<example>` 块覆盖率
   - **我的做法**：echo-sleuth 的 5 个代理（analyze.md、recall.md、memory-auditor.md、file-historian.md、schema-scout.md）全部有 `<example>` 块，每个 2-4 个
   - **本案例做法**（弱在哪）：anthropics/claude-code 的 feature-dev（code-explorer、code-reviewer、code-architect）和 agent-sdk-dev（verifier-ts、verifier-py）5 个代理到 2026-05-22 仍然完全没有 `<example>` 块，audit 后 6 周无变化
   - **意义**：这是 echo-sleuth 相对于官方参考仓库的实际优势；在 NLPM 评分维度上，echo-sleuth 的代理平均分高于 feature-dev。如果考虑给 anthropics/claude-code 提 PR，这 5 个代理的 example 补全是最有把握、最容易被接受的贡献点——但 policy 门禁（`DENY_OWNERS` 包含 `anthropics`）会阻止自动提 PR，需要手动提交

---

## 八、术语表

### <a name="manifest"></a>manifest
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些组件。具体是每个插件目录下的 `.claude-plugin/plugin.json`，里面列出所有 commands、agents、skills、hooks 的文件路径。如果 manifest 里漏写了某个文件，那个文件即使在磁盘上存在也不会被 Claude Code 加载。多插件仓库里每个插件有独立的 manifest，是插件隔离的边界。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据。Claude Code 读代理/命令/技能文件时先解析 frontmatter，才知道这个文件的名字、描述、使用的模型、允许的工具等基本信息。`---` 必须严格从第 1 列（行首）开始，否则 YAML 解析器会把整个文件当作普通 Markdown 正文处理。

### <a name="allowed-tools"></a>allowed-tools
> 命令或代理 frontmatter 里的一个字段，声明这个组件被允许调用哪些工具。既是安全约束（防止误用危险工具），也是文档（告诉用户这个命令会做什么）。可以精确到子命令级别，如 `Bash(git commit:*)` 表示只允许 `git commit` 开头的 bash 命令，不允许任意 bash 命令。

### <a name="model-pinning"></a>模型版本固定
> 在代理或命令的 frontmatter 里用完整的模型 ID（如 `claude-sonnet-4-20250514`）而不是浮动别名（如 `claude-sonnet`）来指定使用的模型版本。「固定」的好处是行为稳定——模型升级时不会自动改变代理的行为；代价是需要主动维护，当旧模型被退役时要手动更新。

### <a name="system-prompt"></a>system prompt
> 传递给 AI 模型的「隐藏指令」，在对话开始前预先注入，定义 AI 的角色、限制和行为规范。Claude Code 把代理/技能的 Markdown 文件内容（去掉 frontmatter 后的正文）作为 system prompt 加载。如果文件末尾有对话残留文本，这些文本也会作为 system prompt 内容注入，影响 AI 的行为。

### <a name="conversation-analyzer"></a>conversation-analyzer 代理
> hookify 插件里的一个专用代理（`plugins/hookify/agents/conversation-analyzer.md`），专门负责分析 Claude Code 对话历史，找出用户想要避免的行为模式（如「你总是写很长的说明，我不想要这个」），然后把这些模式转化为 hook 规则。本仓库的 bug 之一是 hookify 命令在文字上声称调用这个代理，但实际代码里用了通用子代理。
