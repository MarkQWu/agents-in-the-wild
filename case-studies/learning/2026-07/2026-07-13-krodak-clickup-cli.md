# krodak/clickup-cli — 学习案例

**仓库**：https://github.com/krodak/clickup-cli
**Stars**：57 | **来源**：upstream audit（exemplar_published=true）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-13（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `examples-driven`, `single-purpose`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概留览
`krodak/clickup-cli`（`cup`）是波兰开发者 Krzysztof Rodak 开发的 ClickUp 任务管理 CLI 工具，同时附带 Claude Code skill 支持。核心价值：把 ClickUp API 的 50+ 个接口包装为 `cup` 命令，并通过 skill 文件教 Claude 如何用 `cup` 做 AI 驱动的任务管理。

截至 2026-07-13 当前 HEAD：
- **版本 1.38.3**（审计时 1.25.2；3 个月内 13 个次版本更新）
- **3 个 skill 文件**（与 audit 时一致）：主 skill + 2 个内部 agent skill
- **新增**：AGENTS.md、`docs/superpowers/`（含 plans/ 和 specs/ 子目录）、demos/（gif 演示）、dependabot.yml
- **package.json**：commander 从 ^14.0.3 升级到 ^15.0.0（仍保持 caret 范围）
- TypeScript 源码，npm 发布，Homebrew tap 支持

### 1.2 架构剖析
- **目录结构**：
  ```
  clickup-cli/
  ├── skills/clickup-cli/SKILL.md       # 主 skill（389 行，面向用户）
  ├── .agents/skills/
  │   ├── testing-clickup-cli/SKILL.md   # 内部 skill（132 行，测试专用）
  │   └── releasing-clickup-cli/SKILL.md # 内部 skill（132 行，发布专用）
  ├── .claude-plugin/plugin.json          # 插件 manifest
  ├── AGENTS.md                           # 新增：Agent 辅助指南
  ├── docs/
  │   ├── commands.md                     # 自动同步的命令文档
  │   └── superpowers/                    # 新增：功能规划文档
  │       ├── plans/2026-04-29-chat-support.md
  │       └── specs/2026-04-29-chat-support-design.md
  ├── demos/                              # 新增：演示 gif
  ├── scripts/sync-command-docs.ts        # 命令文档自动同步脚本
  ├── src/                               # TypeScript 源码
  ├── package.json
  └── package-lock.json
  ```
- **文件类型分布**：3 个 SKILL.md / 0 个 agent / 0 个 command / 1 个 TS 脚本（sync-command-docs.ts）/ 1 个 JS CLI 主体（npm 发布）
- **编排关系**：主 skill 覆盖所有面向用户的 `cup` 命令；两个内部 skill 标注了 `metadata: {internal: true}`（非标准但无害），只在 CLI 开发/发布场景下被 agent 调用。三个 skill 共享同一个版本号（1.38.3），通过 `releasing-clickup-cli` skill 的 step 3 自动同步。
- **跨件契约**：`releasing-clickup-cli/SKILL.md` step 3 描述了版本同步脚本，确保 plugin.json + SKILL.md header + package.json 版本号三方一致；`scripts/sync-command-docs.ts` 自动同步 docs/commands.md，确保文档不腐烂。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「API 包装层 + 意图驱动 skill」——`cup` 把 ClickUp API 的技术复杂度隐藏在 CLI 后，skill 把 CLI 的操作模式结构化为 Agent 可以意图驱动执行的工作流（"帮我总结今天的进度" → agent 用多个 `cup` 命令组合）。
- **解决什么问题**：AI agent 直接调用 ClickUp REST API 需要处理 pagination、authentication、error 等大量技术细节。`cup` + skill 把这些细节封装掉，让 agent 只需关心"我要做什么"，而不是"API 端点是什么、参数格式是什么"。
- **做了什么 trade-off**：选择"NL 表皮 + 原生二进制核心"——好处是 skill 极简（不需要解释 HTTP 细节），agent workflow examples 可以给出具体可执行的 shell 命令；代价是必须先 `npm install -g clickup-cli`，增加了一个外部依赖。
- **反映什么认知模型**：作者把 AI agent 视为"高层意图执行者"，CLI 视为"意图的精确实现层"——两者分工：AI 理解用户意图并选择合适的命令序列，CLI 精确执行并返回结构化输出。这与"工具调用"范式完全契合。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「NL 表皮 + CLI 核心 + 三 Skill 分层」**

面向用户的主 skill 覆盖所有日常操作场景；两个内部 skill 分别服务"测试"和"发布"两个维护场景；CLI 二进制提供精确的执行层，skill 只描述如何组合 CLI 命令。

模式特征清单：
- **特征 1**：主 skill 的 description 包含 30 个触发短语，覆盖所有 ClickUp 使用意图
- **特征 2**：内部 skill 用 `metadata.internal: true` 与面向用户的 skill 区分，不出现在用户可见的 skill 列表中
- **特征 3**：`Agent Workflow Examples` 章节按任务类型（而非命令类型）组织工作流示例
- **特征 4**：`DELETE SAFETY` 块明确列出所有不可逆操作，要求用户明确确认
- **特征 5**：版本号由发布 skill 的步骤脚本自动同步，不依赖人工维护一致性

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有明确 CLI 的第三方服务（ClickUp / GitHub / Jira） | ✅ 高度适用 | CLI 封装 API 复杂度；skill 只描述工作流 |
| 需要跨平台运行（macOS/Linux/Windows）的工具 | ✅ 适用 | npm 包提供跨平台支持；skill 无 OS 依赖 |
| 纯 NL 任务（写作、搜索、分析） | ❌ 不适用 | 这个模式的核心是 CLI 封装层，纯 NL 场景不需要 |
| 需要离线运行的工具 | ⚠️ 有限适用 | CLI 调用 ClickUp API，必须联网；但 skill 本身无网络依赖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + CLI 核心（本仓库） | krodak/clickup-cli | Skill 极简；CLI 提供精确执行 | 需要额外安装 CLI；CLI 版本和 skill 版本必须同步 |
| 纯 NL skill（无 CLI） | dontbesilent2025/dbskill | 无安装依赖，即开即用 | 无法执行系统操作；只能给建议 |
| NL 表皮 + 原生二进制（Go/Rust） | 777genius/claude-notifications-go | 零运行时依赖；性能高 | 编译成本高；跨平台更复杂 |

### 2.4 改进空间
1. **当前问题**：`plugin.json` 没有 `entries`、`commands` 或 `skills` 数组声明，自动化工具无法枚举插件表面，需要靠文件系统扫描。**改进做法**：在 plugin.json 中加 `"skills": ["skills/clickup-cli/SKILL.md"]` 数组。**预期收益**：NLPM 得分 +10；marketplace 工具可直接索引 skill 列表。
2. **当前问题**：`package.json` 的 `@inquirer/prompts`、`chalk`、`commander` 均使用 `^` caret 范围，允许自动安装 minor 版本更新，有潜在兼容性风险。**改进做法**：改为精确版本（去掉 `^`）或在 CI 中定期检查 `npm audit`。**预期收益**：消除低安全风险；fresh install 结果一致。
3. **当前问题**：`testing-clickup-cli/SKILL.md` step 3 说"Test happy path, error cases, edge cases"——"edge cases"是测试词汇，但本仓库语境下未定义哪些是 edge case。**改进做法**：替换为"Test with missing optional flags, invalid task IDs, and API error responses"。**预期收益**：NLPM 得分 +2；测试人员知道具体测什么。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
95/100（4 个文件）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude-plugin/plugin.json | 90 | 无 entries/skills 数组声明（-10 分） |
| .agents/skills/testing-clickup-cli/SKILL.md | 95 | "edge cases" 轻微模糊（-2 分） |
| .agents/skills/releasing-clickup-cli/SKILL.md | 97 | 无显著问题 |
| skills/clickup-cli/SKILL.md | 97 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **30 个触发短语穷举用户意图** → `skills/clickup-cli/SKILL.md:3` 的 description 列出了 30 个具体动作短语（task queries, status updates, sprint tracking, creating subtasks, posting comments...），前置 "Triggers:" 标签表明这是触发器列表而非功能描述。**为何好**：30 个短语几乎穷举了用户的所有 ClickUp 使用意图，Claude 从任意角度描述需求都能匹配到这个 skill。**如何借鉴**：主 skill 的 description 不是"功能摘要"而是"触发器列表"——多写 10-20 个用户实际会说的短语。
2. **DELETE SAFETY 块** → skill 末尾有专门的 `DELETE SAFETY` 章节，列出所有不可逆操作及其要求（用户明确确认、先列出受影响任务再删除）。**为何好**：不可逆操作是 AI agent 最大的风险点，集中声明比分散在每个命令说明里更醒目。**如何借鉴**：任何有写入/删除/修改操作的 skill，加一个专门的 `## 不可逆操作` 或 `## DELETE/DESTRUCTIVE SAFETY` 章节。
3. **Workflow Examples 按任务类型组织** → `Agent Workflow Examples` 章节不是按命令分类（`cup task --list`、`cup comment --add`），而是按用户任务分类（"Daily standup summary"、"Sprint closure"、"Bug report flow"）——每个任务类型给出完整的多命令 shell 序列。**为何好**：agent 思考的是"我要完成什么"而不是"我有什么命令"；按任务分类的示例更容易被 agent 检索和复用。**如何借鉴**：skill 的示例章节按用户任务/场景分组，而不是按命令/功能分组。

### 3.3 当时的缺陷
1. **plugin.json 无 entries 声明（-10 分）** → `plugin.json` 有 name、version 等标准字段，但没有 `entries`/`skills`/`commands` 数组。根本原因：作者依赖 Claude Code 的文件系统扫描发现 skill，没有意识到 manifest 声明是 NLPM 的评分维度。自查：我的 plugin.json 是否有 skills/commands 数组？
2. **metadata.internal: true 是非标准字段** → 两个内部 skill 都有 `metadata: {internal: true}`，但这个字段不在 Claude Code 的已知 skill schema 中。根本原因：作者发明了一个自定义字段来标记"仅内部使用"的 skill，但 Claude Code 可能忽略这个字段，无法保证内部 skill 对用户不可见。自查：我的 skill 是否有自定义的非标准 frontmatter 字段？
3. **package.json 使用 caret 依赖范围（Low 安全）** → 生产依赖（`@inquirer/prompts`、`chalk`、`commander`）均使用 `^` 范围，fresh install 会自动拉取 latest minor 版本，有兼容性和供应链风险。根本原因：npm 默认行为是 caret 范围，作者没有主动改为精确版本。自查：我的 npm 项目（如 bureau 的 press/）是否也用了 caret 范围？

### 3.4 当时的优化机会（仅供学习）
1. `plugin.json` 加 `"skills": ["skills/clickup-cli/SKILL.md"]`
2. 生产依赖从 `^` 改为精确版本或加 npm shrinkwrap
3. `testing-clickup-cli/SKILL.md:111` "edge cases" 改为具体场景列举

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 无 skills 数组 | `cat .claude-plugin/plugin.json` → 只有 name/description/version/author 等，无 skills 数组 | **仍然存在** | NLPM -10 分问题 3 个月未修复 |
| caret 依赖范围 | `grep '"commander"' package.json` → `"^15.0.0"`（仍 caret） | **仍然存在**（commander 升到 ^15，但未去掉 ^） | 低安全风险持续 |
| metadata.internal 非标准字段 | `grep "internal" .agents/skills/releasing-clickup-cli/SKILL.md` → `metadata:\n  internal: true` | **仍然存在** | 非标准字段，行为不确定 |

### 4.2 架构演进
从 2026-04-06 audit 到 2026-07-13（版本 1.25.2 → 1.38.3，13 个次版本）：
- **最大变化 1**：新增 `docs/superpowers/` 目录（plans/ + specs/）——这是把功能设计文档版本化的做法，符合"NL TDD"思路：先写 spec，再实现。这说明作者开始用 Claude 做功能设计文档，然后用这些文档指导实现。
- **最大变化 2**：新增 `AGENTS.md`——Agent 辅助指南（在 .claude-plugin/plugin.json 之外提供 agent 补充上下文）。
- **最大变化 3**：`.github/dependabot.yml`——引入 Dependabot 自动检查依赖更新，与"手动发现更新"相比，更接近"供应链安全自动化"。
- **commander 版本**：^14 → ^15，这是一个包含 breaking changes 的 major 版本升级，说明作者有在维护依赖。

### 4.3 新增的可学习模式
1. **docs/superpowers/ = NL TDD 的功能规划层**：新增的 `2026-04-29-chat-support.md`（plan）和 `2026-04-29-chat-support-design.md`（spec）把功能规划文档存入 docs/superpowers/，日期前缀命名，按功能组织。这是把"规划 → 实现"的工作流文档化的好实践。
2. **AGENTS.md 作为 agent 上下文补充**：AGENTS.md 不是 CLAUDE.md，是专门针对 AI agent 调用这个 CLI 时的补充说明（与 skill 互补）。这提供了一种"agent 专属文档"和"开发者文档"分离的模式。

---

## 五、校准

### 5.1 我已经在做对的
1. **描述字段写触发短语**：我的 gstack 的 skill description 也包含多种触发场景，与本仓库的"30 触发短语"方向一致。
2. **版本自动同步**：我的 bureau 没有多处版本号，所以无同步问题；但对于 echo-sleuth（有 plugin.json），应该参考本仓库的"releasing skill step"来确保版本一致性。
3. **Workflow Examples 按场景分组**：echo-sleuth 的 `/recall` command 按用户意图（"找上次做的事"、"分析 session 质量"）给出不同路径，与本仓库的"按任务类型给工作流示例"一致。

### 5.2 挑战 / 验证
本案例验证了：**"plugin.json 的 manifest 声明比 CLAUDE.md 中的说明更重要"**。krodak/clickup-cli 的 CLAUDE.md 或 README 可能有 skill 说明，但 NLPM 评分是看 plugin.json 中是否有明确的 `skills` 声明。对于我的 echo-sleuth-for-claude（有 plugin.json），应该检查 skills 数组是否已声明——如果没有，NLPM 会扣 10 分。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否有 skills/entries/commands 数组
cat /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/plugin.json 2>/dev/null | \
  python3 -c "import json,sys; d=json.load(sys.stdin); print('skills:', d.get('skills')); print('commands:', d.get('commands'))"
# 如果输出 None，补充 skills 数组到 plugin.json

# 检查我的 npm 项目（bureau）依赖是否用了 caret 范围
cat /tmp/my-repos/MarkQWu-bureau/package.json 2>/dev/null | \
  python3 -c "import json,sys; d=json.load(sys.stdin); deps=d.get('dependencies',{}); [print(k,v) for k,v in deps.items() if '^' in str(v)]"
# 命中后：评估是否需要改为精确版本（生产依赖应精确；devDependencies 可保持 caret）
```

```bash
# 检查我的 skill 中是否缺少不可逆操作的专门警告块
grep -rn "DELETE\|destroy\|remove\|rm -rf\|irreversible" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ \
  /tmp/my-repos/MarkQWu-bureau/commands/ 2>/dev/null | head -10
# 如果没有任何命中，但 skill 中确实有破坏性操作，说明需要补充安全警告块
```

```bash
# 检查我的 skill 的 Agent Workflow Examples 是否按任务分类（而非按命令分类）
grep -n "## Workflow\|## Example\|## 示例\|## Usage" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/**/*.md 2>/dev/null | head -5
# 如果没有按任务分类的 workflow 示例，考虑重组示例章节
```

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth-for-claude 的 plugin.json 加 `"skills"` 数组声明，列出所有 skill 路径。**为何可行**：本仓库审计发现 -10 分的根本原因；echo-sleuth 如果有 plugin.json 而无 skills 声明，同理扣 10 分。**第一步**：`cat echo-sleuth-for-claude/plugin.json`，找到或创建 skills 数组，把 `skills/*/SKILL.md` 路径列进去；改动 1 个 JSON 文件，5 分钟。
2. **想法**：给 echo-sleuth 加 `docs/superpowers/` 目录，用日期+功能名的文件存放功能设计文档（如 `2026-07-13-session-diff.md`）。**为何可行**：本仓库证明"功能设计文档版本化"能帮助 AI agent 理解"为什么这么设计"，也有助于 NL TDD。**第一步**：在 echo-sleuth 根目录建 `docs/features/` 目录，把"还没实现但已规划"的功能从 TODO 注释迁移为独立 spec 文件，30 分钟。
3. **想法**：给 gstack 中最重要的 skill（如 `land-and-deploy/SKILL.md`）的 description 做触发短语穷举，从现有的 5-6 个触发场景扩展到 15+ 个，覆盖用户可能用不同措辞表达同一意图的情况。**为何可行**：30 触发短语的 description 直接解决"用户措辞 ≠ skill 关键词"的漏载问题。**第一步**：打开 `land-and-deploy/SKILL.md`，在 description 字段末尾追加 10 个同义触发短语，10 分钟。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 krodak/clickup-cli 的核心目的**：通过 CLI 封装 ClickUp API，配合 3 个 skill，让 AI agent 能用 `cup` 命令实现任务管理自动化，并通过 releasing/testing skill 支持工具本身的维护流程。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是 NL 表皮 + 原生工具核心；都有"面向用户"和"面向开发者"两类 skill | gstack 的 binary 核心是 TypeScript 构建的 browser/CLI 工具；krodak 的核心是 TypeScript npm CLI | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 plugin.json；都有 skill + scripts 的组合架构 | echo-sleuth 主要是 NL skill，没有 CLI 核心 | 中 |
| MarkQWu/bureau | 低 | 都有 commands/ 入口层 | bureau 是知识管理系统；krodak 是任务管理 CLI | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 无 skills 数组 | `python3 -c "import json; d=json.load(open('/tmp/my-repos/MarkQWu-echo-sleuth-for-claude/plugin.json')); print(d.get('skills'))"` | `plugin.json` 不存在（echo-sleuth 当前无 plugin.json 文件）；**不适用** | 不适用 |
| npm caret 依赖范围 | `cat /tmp/my-repos/MarkQWu-bureau/package.json 2>/dev/null \| python3 -c "import json,sys; d=json.load(sys.stdin); [print(k,v) for k,v in d.get('dependencies',{}).items()]"` | bureau 有 package.json；依赖项以 `file:press` 为主，无 caret 范围问题 | 无 |
| skill 缺 DELETE SAFETY 块 | `grep -rn "DELETE\|destroy\|警告\|危险" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null` | echo-sleuth skills 无破坏性操作，不需要 DELETE SAFETY 块 | 无 |

**命中后的具体行动建议**：本次扫描均未命中关键缺陷。如果 echo-sleuth 未来添加 plugin.json，应立即加 `"skills"` 数组声明，避免 NLPM -10 分。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：30 触发短语 description + "Triggers:" 前缀明确标记
   - **本案例做法**：`skills/clickup-cli/SKILL.md:3` 的 description 以 "Use when..." 引导意图，然后 "Triggers:" 标签引出 30 个具体动作短语，完全枚举了用户使用 ClickUp 的所有可能意图。
   - **我的项目现状**：gstack 的 SKILL.md description 字段有触发场景描述，但没有明确的 "Triggers:" 前缀；echo-sleuth 的 skill description 也没有穷举触发短语。gstack/land-and-deploy/SKILL.md 的 description 估计在 5-8 个场景。
   - **如何借鉴**：给 gstack 的 `land-and-deploy/SKILL.md` 加 "Triggers: 合并代码、部署上线、push to production、land change、trigger CI/CD..." 等 15+ 触发短语，直接编辑 frontmatter description 字段。

2. **领域**：releasing skill 中的版本同步步骤（版本号三方一致性）
   - **本案例做法**：`.agents/skills/releasing-clickup-cli/SKILL.md` step 3 明确说"update version in plugin.json AND SKILL.md header AND package.json"，step 9 还有 OpenCode skill 路径同步。通过 NL 描述强制维护者在发布时检查三处版本号。
   - **我的项目现状**：gstack 有 VERSION 文件但不确定 gstack 发布时是否有明确的版本同步步骤检查。
   - **如何借鉴**：给 gstack 加一个 `gstack-upgrade/SKILL.md`（本身已存在），确认里面是否有"更新版本号"的具体步骤——如果没有，补充一条"Update VERSION file, plugin.json, and SKILL.md version header"。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skill 的 manifest 声明完整性
- **我的做法**：MarkQWu/gstack 有详细的 CLAUDE.md 组件清单，列出了所有 skill 的路径和名称；MarkQWu/echo-sleuth-for-claude 的 CLAUDE.md 也列出了 commands/agents/skills 的分层结构和路径。
- **本案例做法**（弱在哪）：krodak/clickup-cli 的 plugin.json 没有 `skills` 数组，只有 name/version/description/author，自动化工具无法枚举 skill 表面。虽然 CLAUDE.md 可能有说明，但 manifest 是机器可读的正式接口——缺失会扣 10 分。
- **意义**：如果未来 echo-sleuth 有 plugin.json，第一件事是加 skills 数组；这是 NLPM 最容易避免的扣分点，也是 marketplace 工具依赖的元数据。

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种架构模式：把复杂的技术操作封装为一个命令行程序（二进制/CLI），然后用自然语言 skill 文件教 Claude 如何调用这个程序。"NL 表皮"是 skill（自然语言描述）；"原生二进制核心"是 CLI 程序（如 `cup`）。这种分工让 skill 保持简洁，复杂逻辑在 CLI 层处理。

### <a name="caret-range"></a>caret 范围（^ 版本范围）
> npm package.json 中 `"chalk": "^5.6.2"` 表示"允许安装 5.6.2 及以上的任何 5.x.x 版本"。`^` 是 npm 默认的版本范围符号。好处是自动获得 bug 修复；坏处是 minor 版本可能引入 API 变更导致项目崩溃，且不同机器 fresh install 可能得到不同版本。

### <a name="manifest"></a>manifest（plugin.json）
> 插件的"清单文件"，告诉 Claude Code 这个插件包含哪些组件。`skills` 数组声明了插件暴露的 skill 文件路径；缺少这个数组时，Claude Code 只能靠文件系统扫描来发现 skill，自动化工具（如 NLPM）无法直接枚举。

### <a name="metadata-internal"></a>metadata.internal: true
> krodak/clickup-cli 在内部 skill 的 frontmatter 中加了 `metadata: {internal: true}` 来标记"仅供内部使用"。这是作者自定义的非标准字段——Claude Code 的 skill schema 没有定义 `metadata` 字段，所以 Claude Code 可能会忽略它。真正的隔离方案应该是把内部 skill 放在用户不会直接调用的路径（如 `.agents/skills/`），而不是依赖非标准字段。

### <a name="trigger-phrase"></a>触发短语（trigger phrase）
> skill description 中写给 Claude 的匹配关键词，描述用户可能使用的原话（如"check overdue items"、"post comment"、"sprint tracking"）。Claude 在匹配用户意图和 skill 时做语义匹配；触发短语越贴近用户原话，匹配精度越高。30 个触发短语覆盖了 ClickUp 几乎所有使用场景。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的 `name`、`description`、`metadata` 等字段。Claude Code 读 SKILL.md 时先解析 frontmatter 才能注册和路由这个 skill。

### <a name="dependabot"></a>Dependabot
> GitHub 提供的自动依赖更新机器人。`.github/dependabot.yml` 配置文件告诉 GitHub 多久检查一次依赖更新，并自动提 PR 升级依赖。krodak/clickup-cli 新增了 dependabot.yml，说明作者开始自动化依赖维护。
