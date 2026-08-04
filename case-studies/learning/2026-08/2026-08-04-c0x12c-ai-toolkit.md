# c0x12c/ai-toolkit — 学习案例

**仓库**：https://github.com/c0x12c/ai-toolkit
**Stars**：61 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-08-04（基于当前 HEAD v1.27.0）
**主题标签**：`manifest-discipline`, `examples-driven`, `template-design`, `single-purpose`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

c0x12c/ai-toolkit 是一个面向全栈工程师和创业团队的 Claude Code 插件，以 **Spartan/GSD**（Get Stuff Done）工作流为核心，打包了研发全生命周期所需的 agent、command 和 skill。当前版本 v1.27.0（审计时为 v1.22.1），由 c0x12c 工程组主导维护，定位是「高质量出厂即用」的工程提效套件，npm 包名为 `@c0x12c/ai-toolkit`。

关键事实：
- 从 audit 时的 10 agent + 25 command + 34 skill → 当前 9 agent + 70 command + 35 skill（command 数量翻近 3 倍）
- 设有 `bridges/` 目录（Telegram 等外部集成），是少见的「多运行时桥接」架构
- `toolkit/` 是主开发目录，`.codex/` 是编译产物镜像，由 `compile-packs.js` 在 prepublish 阶段自动生成
- NLPM 历史得分 96/100，属于生产级高分仓库

### 1.2 架构剖析

**目录结构（当前 HEAD）：**
```
c0x12c/ai-toolkit/
├── toolkit/
│   ├── agents/         (9 agents)
│   ├── commands/spartan/  (70 commands)
│   ├── skills/         (35 skills)
│   ├── rules/          (设计原则、架构规范 — 人类可读文档)
│   ├── hooks/          (JS 钩子：版本更新检测等)
│   ├── codex/          (目录级镜像 — toolkit 内的 .codex 镜像)
│   └── VERSION
├── bridges/
│   ├── core/engine.js  (桥接引擎)
│   └── telegram/       (Telegram 集成)
├── CLAUDE.md
└── package.json        (npm 发布入口)
```

**文件类型分布**：9 个 agent / 70 个 command / 35 个 skill

**编排关系**：
- `spartan:gsd` 是「超级入口」，包含完整项目从想法到上线的多阶段编排
- 各 agent 独立运行，无显式 router；command 通过名称空间 `spartan:*` 组织
- Skills 由 agent 和 command 通过 description 引用，并非通过代码显式调用

**跨件契约**：
- [frontmatter](#frontmatter) 协议化，所有 skill 有 `allowed-tools` 字段，但 command 层普遍缺失
- `toolkit/rules/` 中有 DESIGN_PROCESS.md、ARCHITECTURE.md、GIT_COMMIT.md 等，但这些文件没有 SKILL.md [manifest](#manifest) 包装，无法被 NLPM 扫描器发现

### 1.3 设计思路 / 方法论

**核心设计哲学**：「Spartan」= 去除一切不必要的步骤，直接执行。每个 command 对应工程生命周期的一个阶段（`spartan:architect` → `spartan:implement` → `spartan:review` → `spartan:pr`），语义清晰、正交。

**解决什么问题**：工程团队在使用 Claude Code 时，要么重复写 prompt，要么忘记带上正确的上下文约束（安全检查、架构规范、测试要求）。ai-toolkit 把「公司工程文化」固化为 skill 和 command，让新工程师用同一套流程。

**Trade-off**：
- 独立 npm 包 vs 直接在项目里放 .claude/ 文件：选了 npm，换来版本管理和团队共享，代价是发布周期约束
- compile-packs.js 自动镜像 vs 手动维护：选了自动化，但当前镜像产物（`.codex/`）仍有 8 个 skill 和 5 个 agent 未同步（详见 §3.3）
- 70 个 command 大规模扩张：覆盖面提升，但维护成本（`allowed-tools` 字段缺失）呈线性增长

**认知模型**：作者把 AI agent 当「专业岗位」看待——`phase-reviewer` 是代码评审工程师，`sre-architect` 是 SRE 团队负责人，`ai-designer` 是设计师。每个 agent 对应一个明确的「职责边界」，这与传统角色定义一脉相承。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：编译镜像式发布（Compiled Mirror Publishing）**

`toolkit/` 是人类维护的开发目录，`.codex/` 是面向 Claude Code 运行时的产物。`compile-packs.js` 在 npm prepublish 阶段把 `toolkit/` 的内容同步到 `.codex/`，实现「写在一处，运行在另一处」。

模式特征清单：
- **单一真实来源**：toolkit/ 是 SoT，.codex/ 永远不手动编辑
- **发布时同步**：只在发布时跑编译，避免开发期产物污染
- **选择性镜像**：并非全部文件都进 .codex/，这是当前的已知缺陷
- **版本号从 VERSION 文件读取**：确保 plugin.json 和代码始终对齐
- **npm 作为分发通道**：用成熟的包管理生态替代手动 git clone

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨团队共享 NL 工作流 | ✅ 高度适用 | npm 发布 + 版本管理解决多团队同步问题 |
| 企业内需要审批流程的插件 | ✅ 适用 | publish 前可加 CI 门禁 |
| 个人单机使用 | ❌ 过度工程 | 直接放 .claude/ 即可，不需要 npm |
| 需要运行时动态更新 skill | ❌ 不适用 | 编译产物是静态的，更新需重新发布版本 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 编译镜像式（本仓库） | c0x12c/ai-toolkit | 版本化、团队共享、产物与源码隔离 | 发布流程较重；镜像完整性需要测试保证 |
| 直接平铺（Direct Flat） | MarkQWu/bureau | 零构建步骤，改完即用 | 多仓库维护成本高；无版本管理 |
| 动态注册（Dynamic Registry） | ccplugins/awesome-claude-code-plugins | 可以从外部 URL 加载新 skill | 依赖网络；安全风险高 |

### 2.4 改进空间

1. **当前问题**：mirror gap（8 个 skill + 5 个 agent 在 .codex/ 缺失）。**改进做法**：在 compile-packs.js 结束时加一个 assert，确保 toolkit/ 和 .codex/ 的文件清单完全对齐；或在 CI 中跑 `diff <(ls toolkit/skills/) <(ls .codex/skills/)`。**预期收益**：消除「用户安装后发现部分 skill 无法调用」的 bug。

2. **当前问题**：70 个 command 全部缺 `allowed-tools`（截至当前 HEAD，5/70 已修）。**改进做法**：在 compile-packs.js 中加一个 validator，发布前自动检查所有 command 是否有 `allowed-tools` 字段。**预期收益**：彻底防止这类系统性缺失重复出现。

3. **当前问题**：`toolkit/rules/*.md` 是高质量参考文档，但没有 SKILL.md 包装，NLPM 扫描器看不到。**改进做法**：用 `SKILL.md.tmpl` 模板为每个 rule 文件生成一个 SKILL.md 包装（仿照 jnuyens/gsd-plugin 的做法）。**预期收益**：DESIGN_PROCESS.md、GIT_COMMIT.md 等可以被 `/use-skill` 调用，team 知识库真正可被 AI 消费。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）

c0x12c/ai-toolkit 在 2026-04-25 得分 **96/100**。

| 类别 | 得分 | 主要问题 |
|---|---|---|
| Agents（10个）| 94.3/100 | 3 个 agent 零示例（-15 × 3）；phase-reviewer 缺 Bash 工具（BUG） |
| Commands（25个）| 94.8/100 | 全部 25 个缺 allowed-tools（-5 × 25）；guard 缺空参数保护 |
| Skills（34个）| 98.6/100 | terraform-best-practices 缺 allowed_tools（BUG）；少数缺 Gotchas section |

### 3.2 当时值得借鉴的模式

1. **Skill 100分标准参考实现** → `toolkit/skills/deep-research/SKILL.md`、`toolkit/skills/startup-pipeline/SKILL.md` 等 20+ 个 skill 得了满分。根本原因：字段齐全、有 Gotchas section、有 input→output 示例。→ 直接 cat 这些文件作为自己写 skill 的模板。

2. **Agent 职责命名** → `phase-reviewer`、`sre-architect`、`idea-killer` 的名称完全对应工程团队里的真实岗位。根本原因：名称即约束，AI 读到名字就知道自己的边界。→ 给 agent 起名时用「人类岗位名词」而非「功能描述」。

3. **版本管理 + prepublish 编译** → `package.json` 的 `prepublishOnly` 钩子确保每次发布前都重新生成 `.codex/`。根本原因：发布即测试，防止产物落后于源码。→ 对有构建步骤的 NL 项目，在 CI 里加 `npm run compile && npm run test` 串联。

4. **ethos 文档化** → `toolkit/ETHOS.md` 明确写出「Spartan = 够用即可，拒绝过度工程」。根本原因：文档化 trade-off 比写代码注释更持久。→ 在自己项目的 CLAUDE.md 里写一段「我们的设计哲学是什么，什么该做什么不该做」。

### 3.3 当时的缺陷

1. **BUG-1：phase-reviewer.md 引用了 cat/ls 但 tools 不含 Bash**
   - **根本原因**：agent 作者在写 instructions 时用了 shell 习惯（cat config, ls dir），但 Claude Code 要求在 frontmatter 中显式声明 tools 白名单，不声明就无法调用 Bash。这是「文档和配置不一致」的典型模式——人类写注释用的语言和机器执行用的语言不同。
   - **自查**：我的 bureau/commands/ 中所有 command 都没有 allowed-tools；如果某个 command 在 body 里隐式假设了 Bash 可用，同样会出现这个问题。

2. **Q-1：25 个 command 全部缺 allowed-tools**
   - **根本原因**：`allowed-tools` 是 Claude Code 为 command 运行时设定工具白名单的机制。当没有声明时，Claude 会用默认行为（可能更宽松也可能更严格，取决于版本）。系统性缺失说明作者在创建 command 时没有对照 schema checklist，属于「批量遗忘」模式。
   - **自查**：我的 bureau 有 11 个 command，全部缺 allowed-tools。❗同样问题。

3. **Mirror Gap：8 个 skill + 5 个 agent 在 .codex/ 中缺失**
   - **根本原因**：`compile-packs.js` 的 include 列表是手动维护的。每次新增 skill/agent，都需要同步更新 include list，否则新文件不会进产物。这是「双重维护」问题的经典场景——源码和配置清单各一份，人工同步必然出错。
   - **自查**：我的 gstack 有 `codex/` 目录。需要检查是否有类似遗漏。

### 3.4 当时的优化机会

1. **为 toolkit/rules/ 加 SKILL.md 包装**：让 DESIGN_PROCESS.md 等文档变成 NLPM 可扫描、AI 可调用的 skill。
2. **batch PR 给 25 个 command 加 allowed-tools**：单次修改，影响所有 command，性价比最高。
3. **M-2 安全风险：bridges/core/engine.js 默认 bypassPermissions**：明确文档化或改默认值，避免用户无意间启用最大权限。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-1：phase-reviewer 缺 Bash 工具 | `grep "^tools:" toolkit/agents/phase-reviewer.md` | **持续** — tools 字段仍不存在，cat/ls 还在 body 中 | 高优先级 bug 放置 3+ 个月未修，可能作者不知道 NLPM 的检测逻辑 |
| BUG-2：terraform-best-practices 缺 allowed_tools | `head -15 toolkit/skills/terraform-best-practices/SKILL.md` | **持续** — frontmatter 仍无 allowed_tools 字段 | 同上，与 BUG-1 同一个 audit PR，均未合并 |
| Q-1：commands 缺 allowed-tools | `grep -l "allowed-tools" toolkit/commands/spartan/*.md` | **部分修复** — 70 个 command 中 5 个已加，65 个仍缺 | Command 数量从 25 增长到 70，修复率从 0% 升至 7%，趋势向好但基数更大了 |
| Q-2：3 个 agent 零示例 | `grep -l "example" toolkit/agents/*.md` | **部分修复** — 6/9 agent 现在有示例，ai-designer/idea-killer/research-planner 仍缺 | phase-reviewer 在 frontmatter description 里加了示例，但剩余 3 个仍为零 |

### 4.2 架构演进

- **Commands 大扩张（25 → 70）**：新增了 `spartan:fundraise`、`spartan:gate-review`、`spartan:ux` 等全新命令，覆盖了融资、UX、专项评审流程。说明作者意识到原有 25 个 command 覆盖面不足，把「工作流编排」的边界扩大到了整个创业公司生命周期。
- **Team-coordinator agent 被移除**：从 10 个 agent 降到 9 个，team-coordinator 消失。可能是因为其功能被更具体的 `spartan:gate-review` command 取代，避免 agent 和 command 职责重叠。
- **`toolkit/codex/` 目录出现**：当前结构中多了 `toolkit/codex/` 子目录（而非根目录的 `.codex/`），镜像组织方式有所调整。

### 4.3 新增的可学习模式

`spartan:gate-review` command 实现了「双 agent 评审闸门（Dual-Agent Gate）」模式：builder agent 完成一个阶段后，必须通过 gate-review 才能进入下一阶段。这是一种**强制 QA 检查点**，防止积累性技术债。在 audit 时这个 command 还不存在，是 v1.22.1 到 v1.27.0 期间新增的。

---

## 五、校准

### 5.1 我已经在做对的

1. **skill 有 allowed-tools 字段**：gstack/SKILL.md 有 `allowed-tools: [Bash, Read, AskUserQuestion]`，符合规范。
2. **命令语义化命名**：bureau 的 `bureau:lint`、`bureau:review`、`bureau:serve` 命名清晰，对应具体操作。
3. **CLAUDE.md 存在**：gstack 和 bureau 都有 CLAUDE.md 提供上下文约束。
4. **技能引用链路清晰**：bureau commands 明确引用对应 skill（如 `Follow the protocol in the lint skill`），与 ai-toolkit 的做法一致。

### 5.2 挑战 / 验证

本案例验证了一个之前犹豫的判断：**「allowed-tools 字段到底有多重要？」**

结论是非常重要——c0x12c 这个 96 分的高质量仓库，截至今天 command 层仍有 65/70 缺失这个字段，且这是导致命令类得分从 100 下降到 94.8 的主因。我的 bureau 11 个 command 全部缺失，需要作为 P0 优先修复。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 bureau commands 是否有 allowed-tools
grep -rL "allowed-tools" /tmp/my-repos/MarkQWu-bureau/commands/*.md 2>/dev/null || \
grep -rL "allowed-tools" ~/.claude/plugins/*/commands/*.md 2>/dev/null
# 命中后：按 c0x12c 的做法，为每个 command 添加 allowed-tools 字段
```

```bash
# 检查自己的 agent 文件是否缺 Bash 工具但在 body 里用了 shell 命令
grep -rn "cat \|ls \|grep \|find \|echo " ~/.claude/plugins/*/agents/*.md 2>/dev/null | \
  grep -v "^--"
# 命中后：检查对应 agent 的 tools 字段是否包含 Bash
```

```bash
# 检查 gstack codex 目录是否有镜像缺失
diff <(ls /tmp/my-repos/MarkQWu-gstack/agents/ 2>/dev/null | sort) \
     <(ls /tmp/my-repos/MarkQWu-gstack/codex/ 2>/dev/null | sort)
# 命中后：把缺失文件加入 codex/ 或更新编译脚本
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 的 11 个 command 批量添加 allowed-tools
   - **为何可行**：bureau 的 command 大多读写文件，工具集明确（Read/Write/Edit/Glob/Grep 为主，serve command 需要 Bash）
   - **第一步**：打开 `commands/lint.md`，在 frontmatter 加 `allowed-tools: [Read, Write, Glob, Grep]`；用同样模式处理其余 10 个文件；约 30 分钟

2. **想法**：借鉴 compile-packs.js 思路，为 bureau 的多仓库部署添加 prepublish 校验
   - **为何可行**：bureau 目前没有 npm publish 流程，但可以在 CLAUDE.md 里加一条「发布前运行的检查清单」作为替代
   - **第一步**：在 bureau/CLAUDE.md 末尾加 `## 发布检查清单` section，列出必须通过的 nlpm-check 断言

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（含 NL 工件的公开仓库）

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 c0x12c/ai-toolkit 的核心目的**：把全栈工程团队的工作流固化为 Claude Code plugin，涵盖从需求到上线的完整阶段。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同样是「工程团队工作流」场景，有 agent + SKILL.md | gstack 专注于 Garry Tan 的 CEO+CTO+PM 角色套件；ai-toolkit 是 Spartan 编码流水线 | 高 |
| MarkQWu/bureau | 中 | 同样有 commands/ 结构 | bureau 是知识库管理工具，ai-toolkit 是编码工作流 | 中 |
| MarkQWu/graphify | 低 | 都是 Claude Code 工具 | graphify 是 Python 数据工具，无 NL plugin 结构 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Q-1：commands 缺 allowed-tools | `grep -rL "allowed-tools" MarkQWu-bureau/commands/*.md` | **全部命中（11/11）**，bureau 所有 command 均缺 allowed-tools | 高 |
| BUG-1：agent body 用 shell 命令但 tools 无 Bash | `grep -n "cat \|ls \|grep " */agents/*.md` | gstack 仅 1 个 agent（openai.yaml），未命中 | 低 |
| Mirror Gap：编译产物目录与源码不同步 | `diff gstack/agents/ gstack/codex/ 2>/dev/null` | gstack/codex/ 存在但只有 README 和 spartan.zsh，无 agent/skill 镜像 | 需进一步确认 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/commands/lint.md` → 在 frontmatter 加 `allowed-tools: [Read, Write, Glob, Grep]` → 5 分钟
- `MarkQWu-bureau/commands/serve.md` → 需要加 `allowed-tools: [Bash, Read, Write]`（serve 需要启动服务器） → 5 分钟
- 其余 9 个 bureau command → 同样模式，每个 2 分钟，共 18 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：skill 的 Gotchas section
   - **本案例做法**：`toolkit/skills/terraform-best-practices/SKILL.md`、`toolkit/skills/deep-research/SKILL.md` 等均有明确的「What to Avoid / Gotchas」section，列出常见误用模式
   - **我的项目现状**：`MarkQWu-bureau/skills/lint/SKILL.md` 无 Gotchas section（未验证，需进一步检查）
   - **如何借鉴**：在每个 skill 末尾加 `## Gotchas` section，列出「用户最容易误用这个 skill 的 3 种场景」

2. **领域**：命令覆盖整个工作流的完整性
   - **本案例做法**：ai-toolkit 的 70 个 command 把「idea → architect → implement → review → pr → deploy」所有阶段都纳入，用户不会「找不到对应命令」
   - **我的项目现状**：bureau 有 `capture/compile/review` 但缺少 `audit/export/migrate` 等操作
   - **如何借鉴**：做一张「我的知识库管理全生命周期」流程图，对照检查哪些关键步骤还没对应 command

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：commands 的参数处理
- **我的做法**：`MarkQWu-bureau/commands/lint.md` 在 body 里有完整的「如果 --apply 参数存在则执行修复，否则只报告」逻辑，分支路径清晰
- **本案例做法（弱在哪）**：ai-toolkit 的 `spartan:guard` 缺少空参数保护（BUG-3，无 Jinja default），用户忘记传参会直接报错
- **意义**：bureau 的参数防御性处理是一个亮点，未来可以在 NLPM audit 时作为正面参考

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools`、`model` 等）。Claude Code 读 skill/agent/command 文件时先解析 frontmatter 才能知道这个工件怎么注册和调用。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="compile-packs"></a>compile-packs.js
> c0x12c/ai-toolkit 的构建脚本。运行后把 `toolkit/` 目录里的源文件复制到 `.codex/` 产物目录，生成用于 npm 发布的最终文件集合。类似于前端项目的「webpack build」，只不过处理的是 Markdown 而非 JavaScript。

### <a name="prepublish"></a>prepublishOnly
> npm 的一个生命周期钩子：`npm publish` 执行前自动运行。c0x12c 用它在每次发布前自动跑 compile-packs.js，确保产物与源码同步。
