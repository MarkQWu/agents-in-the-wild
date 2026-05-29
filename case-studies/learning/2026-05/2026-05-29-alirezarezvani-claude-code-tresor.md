# alirezarezvani/claude-code-tresor — 学习案例

**仓库**：https://github.com/alirezarezvani/claude-code-tresor
**Stars**：697 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-29（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `cross-reference`, `template-design`, `router-channels`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-code-tresor（法语"宝藏"）是一套面向生产级别的 Claude Code 增强工具集，由 Alireza Rezvani 于 2025 年 9 月创建，v2.7.0（2025 年 11 月）。核心资产是 **8 个自主 skills + 8 个核心 agents + 133 个扩展 agents + 19 个斜杠命令**，围绕"研发团队工作流自动化"这一目标组织，覆盖从代码审查到部署验证的完整链路。同一作者也维护 claude-code-skill-factory（本批次第三个案例）。

关键事实：
1. **133 个扩展 agents**：分 10 个职能团队（工程 54 个、设计 7 个、营销 11 个等），是本批次中智能体数量最多的仓库
2. **CLEAR 安全**：安装脚本只做文件复制，无动态代码执行，是少见的"零安全风险"仓库
3. **v2.7.0 重大重组**：agents/ 目录变为废弃的符号链接，真实位置迁移到 subagents/（这次迁移引入了新缺陷）
4. **NAVIGATION.md + MIGRATION.md**：专门为用户编写的版本迁移指南，是成熟插件的标志

### 1.2 架构剖析

```
claude-code-tresor/
├── CLAUDE.md                        # 仓库开发指南（兼作用户手册）
├── skills/                          # 8 个自主 skills
│   ├── development/                 # code-reviewer, test-generator, git-commit-helper
│   ├── security/                    # security-auditor, secret-scanner, dependency-auditor
│   └── documentation/               # api-documenter, readme-updater
├── subagents/                       # 133 个专家子智能体（v2.7+ 主目录）
│   ├── core/                        # 8 个核心生产 agents（与 agents/ 符号链接）
│   ├── engineering/                 # 54 个工程专家
│   ├── marketing/ product/ ...      # 其余 9 个职能团队
├── agents/                          # [已废弃 v2.7] 指向 subagents/core/ 的符号链接
├── commands/                        # 19 个斜杠命令
│   ├── development/scaffold.md
│   ├── workflow/                    # 6 个工作流命令（含历史重复命令）
│   ├── testing/test-gen.md          # ✅ 当前已存在（Audit 时缺失）
│   ├── documentation/docs-gen.md   # ✅ 当前已存在（Audit 时缺失）
│   ├── security/ performance/       # 共 10 个编排命令
│   └── operations/ quality/
├── scripts/                         # install.sh, update.sh（安全 CLEAR）
└── prompts/                         # 20+ 提示词模板
```

- **文件类型分布**：8 个 SKILL.md / 24 个 command / 8 核心 agents + 133 扩展 agents
- **编排关系**：commands → agents（通过 allowed-tools 调用）→ skills（作为背景知识加载）。10 个编排命令使用复杂的多步伪代码调度 TodoWrite 和多个 agent
- **跨件契约**：`code-reviewer/SKILL.md` 引用了已重命名为 `@config-safety-reviewer` 的 agent（v2.7 前名为 `@code-reviewer`）；`dependency-auditor/SKILL.md` 引用了 `@architect`（已更名为 `@systems-architect`）——这是跨件命名漂移的典型案例

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**"一个仓库，全公司角色"**——从工程师到市场总监、从测试工程师到 CSM，133 个 agents 覆盖了软件公司的大部分职能，让 Claude 扮演任意角色
- **解决什么问题**：在不同任务之间切换 Claude 的"角色"时需要手动上下文切换，tresor 把这种切换变成 `@agent-name` 的简单调用
- **Trade-off**：广覆盖（133 agents）带来认知负担——用户不知道该调用哪个 agent；深度不足——每个 agent 的专业深度受制于文件体量
- **认知模型**：作者把 agent 视为"公司组织图中的角色"，比"工具"或"技能"的隐喻更接近企业软件的使用场景

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多职能 Agent 团队 + 工作流命令编排」**：核心是大量预定义的职能 agent（每个 agent = 一个角色），通过命令层（commands）进行工作流编排。Agent 是角色，command 是剧本，skill 是工具箱。

模式特征清单：
- **特征 1**：Agent 以职能命名（refactor-expert、security-auditor、docs-writer），而非以动作命名（do-refactor、run-security）
- **特征 2**：subagents/ 按组织职能分目录（engineering/、marketing/、product/），而非按技术功能
- **特征 3**：编排命令使用 JavaScript 伪代码描述多 agent 协作流，可读性高
- **特征 4**：CLAUDE.md 兼作开发者指南和用户手册（一鱼两吃），减少文档维护成本
- **特征 5**：skills 设计为"后台自动加载的专业知识"，而非用户主动调用的工具

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要频繁切换不同专业视角的项目 | ✅ 高度适用 | 133 个角色 agents 覆盖大多数视角 |
| 独立全栈开发者 | ✅ 适用 | 用 @security-auditor 做安全审查，用 @docs-writer 写文档 |
| 小型快速迭代项目 | ❌ 不适用 | 133 个 agents 需要时间熟悉 |
| 需要高度精准的单一任务 | ❌ 不适用 | agent 的广度换来了深度的牺牲 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多职能 Agent 团队（本案例）| claude-code-tresor | 角色覆盖广、角色切换直观 | Agent 数量多导致选择困难 |
| 单职责专家 agents | wshobson/agents | 每个 agent 深度高、职责清晰 | 需要用户知道该调哪个 |
| 技能驱动（无 agents）| addyosmani/agent-skills | 轻量、易上手 | 缺少角色记忆和连贯性 |

### 2.4 改进空间

1. **当前问题**：`config-safety-reviewer.md` 的 frontmatter 名称/描述与 body 不一致（body 仍是通用代码审查员的指令，未更新以匹配"配置安全专家"职责）**改进做法**：重写 body，聚焦数据库连接超时、环境变量验证、配置项差异等配置安全域 **预期收益**：用户在做配置安全审查时调用 @config-safety-reviewer 能获得真正的专业意见

2. **当前问题**：5 对重复命令（todo-add/add-to-todos、todo-check/check-todos 等）仍存在于当前 HEAD **改进做法**：删除旧名称别名（保留规范名称 add-to-todos、check-todos）**预期收益**：减少用户困惑，避免两套同功能命令在版本演进中产生分歧

3. **当前问题**：skills 中引用的 agent 名称（@code-reviewer、@architect）仍为 v2.4 旧名 **改进做法**：批量替换为 @config-safety-reviewer 和 @systems-architect **预期收益**：消除用户在 skill 文档中跟随指引后找不到 agent 的体验断裂

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **81/100**，安全等级 CLEAR。

| 文件类型 | 平均分 | 主要问题 |
|---|---|---|
| Agents (8) | 91/100 | config-safety-reviewer body/identity 严重不匹配 (-20) |
| Commands (24) | 75/100 | SlashCommand 无效工具 (-3)、TodoWrite 未声明 (-10)、缺失文件得 0 |
| Skills (8) | 89/100 | 模糊量词、stale agent 名称引用 |
| CLAUDE.md | 90/100 | 少量旧版本代码示例 |

### 3.2 当时值得借鉴的模式

1. **CLEAR 安全安装脚本** → `scripts/install.sh` 只做 `mkdir` + `cp` 文件复制，零网络调用，零动态执行 → `scripts/install.sh` → 安装脚本要"无聊得安全"，能用文件复制解决的绝不用 curl-execute
2. **核心 agents 的高质量** → refactor-expert、security-auditor 等 7 个核心 agent 得分 92-96，frontmatter 完整，numbered workflow 清晰 → `agents/refactor-expert.md` → 用编号步骤（1. Analyze → 2. Plan → 3. Execute）代替模糊指令是提升 agent 质量的最快方法
3. **CLAUDE.md 兼作用户手册** → CLAUDE.md 里的架构树状图让用户一眼看清仓库结构，不需要读 README → `CLAUDE.md` Architecture 章节 → 把架构图写进 CLAUDE.md，Claude 在工作时也能随时参考
4. **NAVIGATION.md + MIGRATION.md 的版本管理** → 每次大版本提供迁移指南，这是专业插件对用户的承诺 → `NAVIGATION.md`, `MIGRATION.md` → 我的插件在做破坏性变更时应该提供 MIGRATION.md

### 3.3 当时的缺陷

1. **`config-safety-reviewer.md` body/identity 严重不匹配** → frontmatter 说"配置安全专家"，body 里却是通用代码审查员的指令 → 问题根因：v2.5 重命名时只改了 frontmatter，没改 body，存在"表里不一"的欺骗性 → 自查：我的 agents 有没有 frontmatter 描述和 body 内容不一致的情况？

2. **两个 command 文件缺失（当时）** → `docs-gen.md` 和 `test-gen.md` 在 CLAUDE.md 中有文档记录但文件不存在，用户调用会报错 → 问题根因：先写文档后实现，实现滞后但文档没有标注"待实现" → 自查：我的 CLAUDE.md / README 中有没有描述了但还没实现的功能？

3. **`SlashCommand` 作为无效工具声明** → 10 个编排命令在 `allowed-tools` 中声明了 `SlashCommand`，这不是 Claude Code 的真实工具 → 问题根因：作者想表达"这个命令会调用其他斜杠命令"的概念，但选错了方式（`SlashCommand` 是概念抽象，不是 API 工具名）→ 自查：我的 allowed-tools 中有没有声明了不存在的工具？

### 3.4 当时的优化机会

1. 修复 `config-safety-reviewer.md` body（最高优先级，影响核心 agent 可信度）
2. 创建缺失的 `docs-gen.md` 和 `test-gen.md` 或从 CLAUDE.md 中移除它们的文档记录
3. 删除 `allowed-tools` 中的 `SlashCommand`，用 `TodoWrite` 替换

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| docs-gen.md 和 test-gen.md 文件缺失 | `ls commands/documentation/docs-gen.md commands/testing/test-gen.md` | **已修复**（两个文件均存在）| 作者完成了这两个功能的实现 |
| SlashCommand 在 allowed-tools 中 | `grep -r "SlashCommand" commands/` | **已修复**（grep 无命中）| 无效工具声明已清除 |
| config-safety-reviewer body/identity 不匹配 | `head -20 agents/config-safety-reviewer.md` | **仍存在**（body 仍描述通用代码审查员）| 这个 bug 存在已超过 7 个月 |
| 重复命令文件（todo-add/add-to-todos 等）| `ls commands/workflow/` | **仍存在**（todo-add、todo-check、prompt-run 等仍在）| 用户仍面临"选哪个"的困惑 |

### 4.2 架构演进

最显著的变化是 v2.7 重组：`agents/` 变为废弃的符号链接层，真正的 agents 迁移到了 `subagents/`，并且规模从 8 个扩展到 133 个（增加了 9 个职能团队的 125 个专家 agent）。同时新增了 `NAVIGATION.md`、`MIGRATION.md`、`WORKFLOW-GUIDE.md` 等大量文档。这说明作者在"扩大覆盖面"而非"修复已知问题"上投入了更多精力。

### 4.3 新增的可学习模式

**符号链接维持后向兼容**：`agents/` 目录保留为指向 `subagents/core/` 的符号链接，使得依赖旧路径（`@refactor-expert`）的用户不需要更新调用方式。这是一种"兼容即服务"的迁移策略，代价是目录结构变复杂，但用户体验平滑。

---

## 五、校准

### 5.1 我已经在做对的

1. **allowed-tools 声明合法工具**：echo-sleuth 的所有 commands 只声明了实际存在的工具
2. **无文件缺失问题**：plugin.json 中列出的所有工件在 repo 中都存在
3. **SKILL.md 和 agent frontmatter 一致**：experience-synthesis 的 body 内容与 description 所声称的功能一致
4. **安装无动态执行**：通过 `claude plugin install` 安装，不需要用户执行任意脚本

### 5.2 挑战 / 验证

**验证了教训**：config-safety-reviewer 的"改了名字没改 body"这个 bug 在 Audit（2026-04-29）到今天（2026-05-29）整整一个月都没被修复。说明**frontmatter 和 body 的一致性不会自动被用户或 CI 检测到**，必须在 review 或 NLPM `/nlpm:check` 中主动检查。今后每次重命名 agent 时，要把修改 frontmatter 和修改 body 作为一个原子操作。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否有 body 与 description 不一致的情况
# 提取每个 agent 的 description 字段和 body 第一段进行对比
for f in /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md; do
  echo "=== $f ==="
  grep "description:" "$f" | head -1
  # 读取 frontmatter 结束后的第一行非空内容
  awk '/^---/{found++; next} found==2 && NF{print; exit}' "$f"
done
```
命中后怎么办：如果某个 agent 的 description 描述的是一件事，body 第一段描述的是另一件事，立即重写 body 使其与 description 对齐。

```bash
# 检查我的 allowed-tools 中是否有无效工具
grep -rn "allowed-tools\|SlashCommand" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md \
  2>/dev/null
```
命中后怎么办：删除所有非 Claude Code 官方工具（如 SlashCommand、CustomTool 等自造名词）。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 agents 目录增加 README，说明每个 agent 的触发条件和使用场景（类似 tresor 的 agents/README.md）
   - **为何可行**：tresor 的 agents/README.md 让用户能快速找到合适的 agent，echo-sleuth 目前需要逐个打开 agent 文件才能理解
   - **第一步**：创建 `agents/README.md`，用表格列出 5 个 agents 的名称、描述、最佳调用时机，约 15 分钟

2. **想法**：借鉴 tresor 的 MIGRATION.md，在 drama-workshop-skills 每次大版本更新时提供迁移指南
   - **为何可行**：drama-workshop-skills 已到 v1.40，有多次破坏性变更（如 overseas-adaptation-map 的引入），但没有迁移文档
   - **第一步**：在下次版本发布时创建 `MIGRATION.md`，列出旧命令到新命令的映射，约 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 alirezarezvani/claude-code-tresor 的核心目的**：提供生产级 Claude Code 多职能 agent 团队和工作流命令集，自动化软件开发全流程

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是 multi-agent Claude Code 插件；都有 5+ agents；都有命令编排层 | tresor 面向软件开发工作流，echo-sleuth 面向知识提取 | 高 |
| MarkQWu/claude-for-legal | 中 | 都是多 skill + multi-domain 的插件集合 | 职能域不同（工程 vs 法律），架构相似 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code skill | 领域完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent body 与 frontmatter description 不一致 | 已逐个检查 echo-sleuth agents | **未命中**——所有 5 个 agents 的 body 与 description 一致 | N/A |
| allowed-tools 声明无效工具（SlashCommand）| `grep "SlashCommand" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/**/*.md` | **未命中** | N/A |
| 重复命令文件 | 检查 echo-sleuth commands/ 目录 | **未命中**（recall 和 recap 功能不同，不是重复）| N/A |
| Skills 引用 stale agent 名称 | `grep "@" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/**/*.md` | 需要检查 echo-sleuth 的 experience-synthesis SKILL.md | 低 |

**命中后的具体行动建议**：
- 当重命名任意 agent 时，立即用 `grep -rn "@old-name"` 搜索全 repo 并批量替换——这一步要和重命名本身作为同一个 commit

### 8.3 别人的更优方案

1. **领域**：NAVIGATION.md + MIGRATION.md 版本维护文档
   - **本案例做法**：每个大版本提供 `MIGRATION.md`，列出迁移路径（旧目录 → 新目录）；`NAVIGATION.md` 提供仓库结构快速导览
   - **我的项目现状**：echo-sleuth 和 drama-workshop-skills 只有 WHATSNEW.md（当前版本更新），没有迁移指南
   - **如何借鉴**：在 drama-workshop-skills v1.41 发布时创建 `MIGRATION.md`，记录从 v1.40 开始的命令变更，约 10 分钟

2. **领域**：CLAUDE.md 架构树状图
   - **本案例做法**：CLAUDE.md 中有完整的 ASCII 树状图展示仓库目录结构，含注释
   - **我的项目现状**：echo-sleuth 的 CLAUDE.md 没有目录树（只有文字描述）
   - **如何借鉴**：在 echo-sleuth/CLAUDE.md 中加一个简洁的 6-8 行目录树，约 5 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Agent identity 一致性（frontmatter vs body 一致）
- **我的做法**：MarkQWu/echo-sleuth-for-claude 所有 5 个 agents 的 frontmatter description 和 body 开头描述一致——每次重命名后同步更新了 body
- **本案例做法**：`config-safety-reviewer.md` 被重命名但 body 仍是旧代码审查员的通用指令，这个 bug 距今已超过 1 个月未修复
- **意义**：这是 NLPM `/nlpm:check` 能检测到的一类问题（cross-component consistency）。我的项目在这个维度更干净，可以作为案例证明"重命名时 body 同步"工作流的价值。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置，声明 `name`、`description`、`tools`、`model` 等元数据。对于 agents，frontmatter 是 Claude Code 识别和注册该 agent 的唯一依据；body 是 agent 的行为指令。两者必须一致，否则用户按名字调用 agent 时会得到意外的行为。

### <a name="allowed-tools"></a>allowed-tools
> command 文件 frontmatter 中的字段，声明这个斜杠命令在执行时被允许使用哪些 Claude Code 工具。只有在这个白名单里的工具，Claude 才会在执行该命令时不经额外确认地使用。声明不存在的工具（如 `SlashCommand`）不会导致错误，但该工具调用永远不会执行，形成静默失败。

### <a name="符号链接"></a>符号链接
> 文件系统中的一种"快捷方式"（symlink）：`agents/` 目录实际上是一个指向 `subagents/core/` 的链接，两个路径指向同一份文件。好处是旧的调用方式（`@refactor-expert`）不需要改；坏处是目录结构对不熟悉这种设计的用户来说会产生困惑。

### <a name="stale-reference"></a>stale reference
> "过期引用"——文件中引用了一个曾经存在但已被重命名/删除的目标，如 `@code-reviewer`（v2.4 名称）在 skill body 中仍然出现，但实际 agent 已被重命名为 `@config-safety-reviewer`。用户按指引操作时会得到"agent not found"错误。
