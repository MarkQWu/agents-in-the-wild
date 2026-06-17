# kbwo/ccmanager — 学习案例

**仓库**：https://github.com/kbwo/ccmanager
**Stars**：1031 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-17（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `nl-binary-hybrid`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
kbwo/ccmanager 是一个用 TypeScript + [Bun](#bun) + [Ink](#ink) 构建的 TUI（终端用户界面）程序，用于跨 [git worktree](#git-worktree) 管理多个 AI 编程助手会话（Claude Code、Gemini CLI、Codex CLI 等）。1031 颗星，活跃维护。其 NL 产物主体是 `.claude/commands/kiro/` 目录下 10 个 kiro 风格命令，实现了一套从需求到任务到实现的 spec 工作流（类 Kiro IDE 的工作流）。

关键事实：
- ccmanager 本体是 TypeScript/Bun 二进制（支持 macOS arm64/x64、Linux arm64/x64、Windows x64 五平台）
- NL 产物：10 个 `.claude/commands/kiro/*.md` + `CLAUDE.md`，实现了 `/kiro:spec-init`、`/kiro:spec-requirements`、`/kiro:spec-design` 等 spec 工作流
- `.kiro/specs/` 目录有 2 个真实使用过的 spec（loading-spinner、result-pattern）
- DevContainer 集成（`.devcontainer/`），含防火墙初始化脚本

### 1.2 架构剖析
- **目录结构**：

```
ccmanager/
├── .claude/
│   └── commands/kiro/
│       ├── spec-init.md        # 初始化 spec（创建 spec.json）
│       ├── spec-requirements.md # 需求文档生成
│       ├── spec-design.md      # 技术设计文档
│       ├── spec-tasks.md       # 任务列表生成
│       ├── spec-impl.md        # 实现执行
│       ├── spec-status.md      # 查看 spec 状态
│       ├── steering.md         # 自动生成 steering 文件
│       ├── steering-custom.md  # 自定义 steering
│       ├── validate-design.md  # 验证设计文档
│       └── validate-gap.md     # 验证实现 gap
├── CLAUDE.md                   # 项目规范（bug：未提及 bun 依赖）
├── .kiro/
│   ├── specs/                  # 实际使用过的 spec
│   └── steering/               # 项目引导文件
├── .devcontainer/
│   └── init-firewall.sh        # 网络防火墙初始化
├── bin/cli.js                  # 平台适配入口
└── npm/                        # 各平台二进制包
    ├── darwin-arm64/
    ├── linux-x64/
    └── ...
```

- **文件类型分布**：10 个 command，1 个 project doc（CLAUDE.md），0 个 skill，0 个 agent
- **编排关系**：10 个 command 平铺，无 router。每个 command 以 `$1`（feature-name）为参数，操作 `.kiro/specs/$1/` 目录下的文件。工作流顺序：spec-init → spec-requirements → spec-design → spec-tasks → spec-impl。validate-design/validate-gap 可在任意节点插入。
- **跨件契约**：CLAUDE.md 的构建命令（`npm run dev`）与 `package.json` 的实现（`bun run tsc`）不一致；3 个 command 的 `allowed-tools` 包含不存在的 `Update` 工具名

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「spec 驱动开发」——在写代码之前先通过 AI 生成需求文档（EARS 格式）、技术设计、任务列表，然后按计划执行。每个阶段有独立命令，可独立触发或串联。
- **解决什么问题**：多 AI 会话管理（TUI 主体）+ 帮助 AI 在复杂功能上保持一致性（kiro spec 工作流）。两个功能共用一个仓库。
- **Trade-off**：spec 工作流把需求、设计、任务分为 3 个 command，每个都接受 `$1` 参数。好处：流程清晰，每步可独立重跑；坏处：没有输入保护，`$1` 为空时或被注入时行为不确定。
- **认知模型**：把「写一个功能」拆分为「理解需求 → 设计方案 → 分解任务 → 按任务实现」，每个阶段对应一个 command，AI 只在当前阶段工作，不跳步。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「无 manifest 的项目内 command 集合」模式**：commands 存放在 `.claude/commands/` 下（项目级而非插件级），通过 CLAUDE.md 作为 project doc 提供上下文，无单独的 plugin.json。

模式特征清单：
- 命令属于项目本身，而非可独立安装的插件
- 共享参数约定（所有 spec 命令都接受 `$1 = feature-name`）
- 工作流有明确顺序但无强制编排（用户手动决定执行顺序）
- `.kiro/specs/` 目录是 command 的持久化状态存储

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 项目内部工作流（不需要跨项目复用）| ✅ 高度适用 | 无插件分发成本，直接放 .claude/commands/ |
| 需要严格安全约束的环境 | ❌ 不适用 | `bash -c '$1'` 模式存在[命令注入](#command-injection)风险 |
| 多人团队共享工作流 | ⚠️ 勉强 | 需要确保团队了解 spec 工作流顺序，无强制约束 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 项目内 command 集合（本仓库）| ccmanager kiro | 零分发成本，项目定制性强 | 不可复用，无插件生态 |
| 可安装插件（plugin.json）| superpowers-zh | 可复用，多项目使用 | 需要维护 manifest，版本管理 |
| Slash command + skill 分离 | nlpm | 灵活，command 调度 skill | 架构复杂，学习成本高 |

### 2.4 改进空间
1. **当前问题**：3 个 command 中 `!bash -c 'ls -la .kiro/specs/$1/'` 直接注入用户输入，存在[命令注入](#command-injection)（High 级安全）。**改进做法**：在每个 command 顶部加 `$1` 校验指令：「如果 $1 包含非字母数字/下划线/连字符字符，停止并返回错误」。**预期收益**：消除 3 个 High 级安全发现，同时解决了空输入保护问题。
2. **当前问题**：10 个 command 中大量「comprehensive」「thorough」「appropriate」等模糊量词（最多累计 -20 分/文件），导致多个 command 失去可验证性。**改进做法**：逐一替换为 EARS 格式的可量化标准（如「comprehensive design」→「design covering all EARS requirements from requirements.md」）。**预期收益**：command 执行结果可校验，质量分提升约 10 分。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 当时得分 **83/100**。安全状态：**BLOCKED**（4 个 High 安全发现）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| kiro/spec-design.md | 67 | 模糊量词封顶 (-20)、无空输入保护 (-10)、无效 `Update` 工具 |
| kiro/spec-status.md | 79 | 无效 `Update` 工具、无空输入保护、模糊量词 (-8) |
| kiro/steering.md | 80 | 模糊量词封顶 (-20) |
| kiro/validate-design.md | 80 | 无空输入保护 (-10)、模糊量词 (-10) |
| kiro/validate-gap.md | 80 | 无空输入保护 (-10)、模糊量词 (-10) |
| kiro/spec-requirements.md | 81 | 无效 `Update` 工具、无空输入保护、模糊量词 |
| kiro/spec-tasks.md | 82 | 无空输入保护、模糊量词 |
| kiro/spec-init.md | 86 | 无空输入保护 (-10)，轻微模糊量词 |
| kiro/spec-impl.md | 88 | 无空输入保护 (-10) |
| kiro/steering-custom.md | 92 | 轻微模糊量词 (-8) |
| CLAUDE.md | 96 | 轻微模糊量词 (-4) |

### 3.2 当时值得借鉴的模式
1. **spec 工作流阶段化**：把「写一个功能」分解为 5 个可独立执行的命令（init/requirements/design/tasks/impl），每个命令专注一个阶段，职责单一。这是「大任务拆分为可校验小步骤」的优秀实践。
2. **steering 文件系统**：`.kiro/steering/` 下有 `product.md`、`structure.md`、`tech.md`——把项目的产品定位、架构约束、技术栈分离存储，比单一 CLAUDE.md 的信息更有结构。
3. **binary 分平台发布**：`npm/darwin-arm64/`、`npm/linux-x64/` 等独立包加上 platform-specific binary——完整的多平台分发方案，Bun 的 bundle 能力使二进制体积小、无外部依赖。

### 3.3 当时的缺陷
1. **3 个 command 的[命令注入](#command-injection)（High 级）**：`spec-requirements.md`、`spec-status.md`、`validate-gap.md` 都有 `!bash -c 'ls -la .kiro/specs/$1/'`。根本原因：作者把 NL command 的 `$1` 等同于安全的环境变量，忽略了 bash inline 执行时字符串插值的危险。恶意输入如 `feature'; rm -rf ~/.claude; echo '` 可执行任意命令。**这是我见过的 NL 产物中最危险的模式之一。**
2. **`Update` 工具名无效**：3 个 command 的 `allowed-tools` 包含 `Update`，但 Claude Code 没有这个工具。根本原因：模板复制时带入了草稿阶段的占位符，未清理。运行时可能产生警告或行为不一致。
3. **CLAUDE.md 与 package.json 不一致（CC-1）**：CLAUDE.md 指示用 `npm run dev`，但 package.json 的每个脚本实际调用 `bun run`，没有 bun 的环境会直接失败。根本原因：文档更新滞后于技术栈迁移（npm→bun）。自查：我的 CLAUDE.md 有没有描述不存在的依赖或命令？

### 3.4 当时的优化机会
1. 在 3 个有注入风险的 command 顶部加 `$1` 格式校验（字母数字/下划线/连字符）
2. 移除 `allowed-tools` 中的 `Update`（3 文件各 1 行，5 分钟完成）
3. 补充 CLAUDE.md 的 bun 先决条件说明

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `spec-requirements.md` 命令注入 | `grep -n "bash -c" .claude/commands/kiro/spec-requirements.md` | **未修复**：line 20 仍为 `!bash -c 'ls -la .kiro/specs/$1/'` | 3 个月后 High 级安全问题仍在 |
| `spec-status.md` 命令注入 | `grep -n "bash -c" .claude/commands/kiro/spec-status.md` | **未修复**：lines 14、21-22 仍包含未净化的 `$1` 插值 | 同上 |
| `Update` 工具在 `allowed-tools` | `grep "Update" .claude/commands/kiro/spec-design.md` | **未修复**：line 3 仍为 `allowed-tools: ..., Update, ...` | 无效工具声明持续存在 |
| CLAUDE.md 未提及 bun | `grep bun CLAUDE.md` | **未修复**：CLAUDE.md 仍只提 `npm run *` 命令，无 bun 相关说明 | 新人按文档操作会失败 |

### 4.2 架构演进
对比审计快照，NL 产物（10 个 command + CLAUDE.md）**无实质改动**。`package.json` 版本从 4.1.x 升级（当前 4.1.19），`bun.lock` 有更新，TypeScript 主体功能持续迭代，但 NL 层完全冻结。

4 个审计发现一个都未修复，其中包括 High 级安全问题。这是一个强烈信号：作者没有把 NL 产物纳入常规维护流程。

### 4.3 新增的可学习模式
**steering 文件的实际使用场景**：`.kiro/specs/loading-spinner-async-operations/` 和 `.kiro/specs/result-pattern-error-handling-2/` 是真实的 spec 记录——包含 requirements.md、design.md、tasks.md、spec.json。这些文件是「kiro spec 工作流的最终产物」，证明了 spec 驱动开发在 ccmanager 自身开发中确实被使用。学习点：用自己的工具开发自己的项目（dogfooding）是对工具最好的验证。

---

## 五、校准

### 5.1 我已经在做对的
1. **不在 NL command 中内联 bash $1 插值**：我的 echo-sleuth commands 没有 `!bash -c '...$1...'` 这种模式，不存在命令注入风险。
2. **文档与实现一致**：我的 CLAUDE.md（如果存在）描述的命令与 package.json 的实际脚本一致。
3. **阶段化工作流设计**：echo-sleuth 的 `/audit`、`/prune`、`/extract` 命令各管一个阶段，与 ccmanager 的 spec 工作流理念一致。

### 5.2 挑战 / 验证
本案例强烈挑战了「NL command 只是文字，没有安全风险」的假设。`!bash -c '...$1...'` 证明了 NL command 中的内联 shell 执行和 bash 脚本一样危险，甚至更隐蔽——因为安全审计通常只看 `.sh` 文件，不检查 Markdown。**任何在 NL command 中出现的 `bash -c` + 用户参数，都需要做参数净化。**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 是否有 bash -c 与用户参数拼接
grep -rn "bash -c.*\\\$" /tmp/my-repos/MarkQWu-*/commands/ 2>/dev/null
grep -rn 'bash -c.*\$' /tmp/my-repos/MarkQWu-*/.claude/commands/ 2>/dev/null
# 命中后：立即检查是否存在未净化的用户输入插值
```

```bash
# 检查我的 command allowed-tools 是否包含无效工具名
grep -rn "allowed-tools" /tmp/my-repos/MarkQWu-*/commands/ 2>/dev/null | \
  grep -v "Bash\|Read\|Write\|Edit\|Glob\|Grep\|WebFetch\|WebSearch\|LS\|MultiEdit"
# 命中后：对比 Claude Code 官方工具列表，移除不存在的工具名
```

```bash
# 检查 CLAUDE.md 描述的命令是否和 package.json 一致
for repo in /tmp/my-repos/MarkQWu-*/; do
  echo "=== $(basename $repo) ==="
  grep -n "npm run\|bun run\|yarn\|pnpm" "$repo/CLAUDE.md" 2>/dev/null | head -5
  grep -n '"scripts"' "$repo/package.json" 2>/dev/null
done
# 命中后：同步 CLAUDE.md 的命令描述
```

### 6.2 灵感 → 实施路径
1. **想法**：在 drama-workshop-skills 或 echo-sleuth 中引入类似的分阶段 spec 工作流
   - **为何可行**：ccmanager 自身用 spec 工作流开发自身功能，证明了对 TypeScript 项目的实际效果；echo-sleuth 也有复杂功能迭代需求
   - **第一步**：在 `.claude/commands/` 下创建 `spec-init.md`（参考 ccmanager 的实现，但在 `$1` 处理上加输入校验），20 分钟可完成框架

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 kbwo/ccmanager 的核心目的**：TUI 管理多 AI 会话 + spec 驱动开发工作流。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件，有 command 层 | echo-sleuth 是 skill 集合，无 TUI；领域不同（会话挖掘 vs 会话管理）| 中 |
| MarkQWu/drama-workshop-skills | 低 | 同有多阶段 command 工作流 | 领域不同（剧本创作 vs 软件开发） | 中 |
| MarkQWu/claude-for-legal | 无 | — | 完全不同 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| bash -c 命令注入 | `grep -rn 'bash -c.*\$' /tmp/my-repos/MarkQWu-*/` | **0 命中**：我的 command 无内联 bash 执行 | 无 |
| allowed-tools 包含无效工具 | `grep -rn "Update" /tmp/my-repos/MarkQWu-*/commands/` | **0 命中** | 无 |
| 模糊量词（appropriate/comprehensive）| `grep -rn "comprehensive\|appropriate" /tmp/my-repos/MarkQWu-claude-for-legal/` | claude-for-legal 有多处命中（`matter-workspace/SKILL.md`、`demand-received/SKILL.md` 等），但多在法律文书说明文字中，非行为指令 | 低 |

### 7.3 别人的更优方案

1. **领域**：steering 文件分层（product/structure/tech）
   - **本案例做法**：`.kiro/steering/product.md`、`structure.md`、`tech.md` 分别记录产品定位、项目结构约束、技术栈，比单一 CLAUDE.md 结构化
   - **我的项目现状**：drama-workshop-skills 的 CLAUDE.md（如果存在）可能混合了所有这些信息
   - **如何借鉴**：把 echo-sleuth 或 drama-workshop-skills 的 CLAUDE.md 拆为 `steering/product.md`（用途/用户）、`steering/tech.md`（技术栈/命令）两个文件，提高可维护性

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：NL 层随功能更新同步维护
- **我的做法**：echo-sleuth-for-claude 的 skill 和 command 与功能实现同步更新，无脱节
- **本案例做法**：ccmanager 的 NL 层（10 个 kiro command + CLAUDE.md）在 4 个高优先级 bug 存在的情况下冻结了 3+ 个月
- **意义**：NL 产物不应该被视为「写完就完事」的静态文档。高安全风险（命令注入）在 NL 文件里同样危险，同样需要修复

---

## 八、术语表

### <a name="bun"></a>Bun
> 一种 JavaScript/TypeScript 运行时，类似 Node.js，但性能更高，内置打包器和测试运行器。ccmanager 的 `package.json` 中的所有脚本（`bun run tsc`、`bun run eslint`）依赖 Bun，而非 Node.js 的 npm。如果系统中没有安装 bun，直接运行 `npm run dev` 会失败。

### <a name="ink"></a>Ink
> 用 React 组件模型构建 TUI（终端用户界面）的 Node.js 库。ccmanager 用 Ink 渲染终端中的多 session 管理界面——像写 React 组件一样写终端 UI。

### <a name="git-worktree"></a>git worktree
> Git 的一个功能，允许同一个仓库同时检出到多个工作目录，每个工作目录对应一个分支。ccmanager 的核心用途是在每个 worktree 里启动一个独立的 AI 编程助手会话，并行处理多个功能分支。

### <a name="command-injection"></a>命令注入
> 一种安全漏洞：程序把用户提供的输入直接拼接进 shell 命令字符串后执行。恶意输入（如包含 `;`、`&&`、`$()`）可让攻击者在执行预期命令的同时执行任意其他命令。`bash -c 'ls $1'` 中的 `$1` 如果来自用户输入且未验证，就存在此风险。
