# a5c-ai/babysitter — 学习案例

**仓库**：https://github.com/a5c-ai/babysitter
**Stars**：562 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-28（基于当前 HEAD）
**主题标签**：`security-gate`, `examples-driven`, `vague-quantifier`, `curl-pipe-bash-risk`, `model-pinning`

**xiaolai 案例**：[../../2026-04-28-a5c-ai-babysitter.md](../../2026-04-28-a5c-ai-babysitter.md)

---

## 一、理解

### 1.1 仓库概览

a5c-ai/babysitter 是一个面向 Claude Code 的**确定性、事件溯源式多代理编排框架**。它的核心主张是：让 AI 代理的行为变得可预测、可重现、无幻觉——通过事件日志（effect-sourced journal）取代随机对话历史来驱动代理间协作。

项目当前结构已经从最初的"四个钩子脚本 + 4189 个领域代理定义"演进为一个多包 monorepo：

```
packages/
├── catalog/          # 代理目录管理
├── observer-dashboard/  # 运行状态可视化
├── babysitter/       # 核心运行时
└── sdk/              # 开发者 SDK

plugins/
├── babysitter/           # Claude Code 插件（主体）
├── babysitter-codex/     # Codex 适配器
├── babysitter-cursor/    # Cursor 适配器
├── babysitter-gemini/    # Gemini 适配器
├── babysitter-github/    # GitHub Actions 适配器
├── babysitter-omp/       # OMP 适配器
├── babysitter-opencode/  # OpenCode 适配器
├── babysitter-pi/        # PI 适配器
└── a5c/                  # a5c 主插件

library/
├── SPEC-KIT.md           # 规格套件
└── methodologies/        # 方法论文档

.claude-plugin/marketplace.json
```

项目的野心是多编辑器、多 AI 提供商兼容的代理编排层，不再局限于 Claude Code 一个平台。

### 1.2 架构剖析

Babysitter 的核心设计哲学是**事件溯源（event-sourced）编排**：

1. **Run 生命周期**：通过 `babysitter run:create`（初始化运行上下文）和 `babysitter run:iterate`（推进状态）来驱动工作流，而非让代理在对话历史中漂移。
2. **Effect Journal（效果日志）**：每个代理的输出被记录为一条"效果"（effect），新代理读取 journal 而非整个对话历史，实现隔离执行。
3. **钩子层（Hook Layer）**：四个 Claude Code 钩子（`session-start`、`stop`、`pre-tool-use`、`user-prompt-submit`）负责在 SDK 未就绪时完成初始化、注入上下文。
4. **SDK 下沉**：真正的业务逻辑在 `@a5c-ai/babysitter-sdk`（npm 包）里，NL 文件只做调度。

在 4189 个自然语言代理定义中，领域覆盖范围惊人：量子计算、生物医学工程、土木工程、电气工程、机械工程（ruflo 系列）、飞行器壳体（pilot-shell）等。这些领域代理定义构成一个"专家代理库"，可被编排框架按需调用。

### 1.3 设计思路 / 方法论

Babysitter 的方法论建立在三个核心支柱上：

**支柱 1：确定性优先（Determinism First）**

传统多代理系统依赖对话历史来传递上下文，但对话历史是非结构化的、顺序敏感的。Babysitter 用 effect journal 替代这种"上下文扩散"，每个代理只看到它需要看到的效果记录，不受无关对话干扰。

**支柱 2：多平台适配（Multi-Platform Adaptation）**

`plugins/` 下的 8 个子插件（babysitter-codex、babysitter-cursor、babysitter-gemini 等）是同一编排逻辑在不同 AI 平台上的薄层适配。这体现了"核心逻辑与平台适配分离"的架构思想。

**支柱 3：大规模专家库（Domain Expert Library at Scale）**

4189 个领域代理定义不是为了让用户逐一使用，而是作为编排器可以**按需路由**的专家库。编排器根据任务语义选择合适的领域代理，类似于"动态团队组建"。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名称 | 描述 | 在仓库中的体现 |
|---------|------|--------------|
| **Event-Sourced 编排模式** | 用不可变效果日志驱动代理状态转移，替代对话历史 | `babysitter run:create` / `run:iterate` 语义 |
| **钩子层初始化模式** | 在 `session-start` 钩子中完成 SDK 注入和上下文准备 | 四个 `*-hook.sh` 脚本 |
| **运行时 npm 安装模式** | 钩子在执行时从 npm 注册表拉取 SDK | `npx -y @a5c-ai/babysitter-sdk` |
| **版本锁定文件模式** | `versions.json` 固定 npm 包版本，收窄供应链攻击窗口 | `versions.json` |
| **多平台适配器模式** | 核心编排逻辑不变，为每个平台写薄层适配插件 | `plugins/babysitter-*/` |
| **大规模领域代理库模式** | 为不同专业领域预写大量代理定义，供编排器按需路由 | `library/` 下 4189 个 `.md` 文件 |

### 2.2 适用场景

**Event-Sourced 编排模式**适用于：
- 多个并行代理需要共享状态但又必须隔离执行的场景
- 工作流需要可重现（同样的 journal 总产出同样结果）的场景
- 调试时需要"回放"代理行为的场景

**运行时 npm 安装模式（反面警示）**：
- **不适用**于任何生产级插件。每次会话拉取 npm 包意味着：1）网络故障即失败；2）供应链风险窗口始终开放。
- 对于 SDK 需要频繁更新的**开发阶段**原型，可以接受，但必须在发布前替换为预安装方案。

**大规模领域代理库模式**适用于：
- 有明确路由逻辑（可以根据任务自动选择领域代理）的框架
- 代理库质量要求高于覆盖广度的场景（反例：4189 个但质量参差不齐）

**版本锁定文件模式**适用于：
- 任何依赖运行时拉取外部包的场景，是最低安全基线
- 但版本锁定只是"缩小攻击窗口"，不是"消除供应链风险"

### 2.3 与其他架构对比

| 架构维度 | babysitter（Event-Sourced） | 纯 NL 平铺（如 SuperClaude） | NL+MCP 混合（如 ouroboros） |
|---------|--------------------------|---------------------------|--------------------------|
| 状态管理 | Effect journal（结构化、可回放）| 对话历史（非结构化）| MCP 后端维护跨 session 状态 |
| 安装复杂度 | 高（npm SDK + 钩子初始化）| 低（纯 Markdown）| 高（pip install + MCP 配置）|
| 供应链风险 | **极高**（每次 session 拉取 npm）| 无 | 中（一次性 pip install）|
| 多平台兼容 | **最佳**（8 个平台适配器）| 无 | 无 |
| 代理发现方式 | 编排器路由 + `library/` 静态定义 | Claude Code 扫描 frontmatter | SKILL.md 显式引用 agent |
| NL 制品质量 | 中（70/100，有高低分化）| 中（84/100）| 中（69/100）|

### 2.4 改进空间

1. **彻底消除运行时 npm 拉取**：将 `@a5c-ai/babysitter-sdk` 预安装为插件的一部分（类似于 SuperClaude 的 Python 包模式），或至少提供离线回退路径。预期收益：消除 Medium × 8 安全发现，同时降低网络依赖。

2. **为领域代理库补充三要素**：100 个被抽样的代理文件全部缺少 examples 块、64 个缺少 output format 节、91 个缺少 model 字段。可以用一个脚本批量检查并提示人工补充，从结构上保证新代理定义符合质量基线。

3. **`export PATH` 风险迁移**：将 `export PATH="$HOME/.local/bin:$PATH"` 从钩子脚本移到 `.env` 文件或在安装时一次性配置用户 shell profile，而非每次钩子执行时修改。

4. **补充 `spec-guard/AGENT.md`（已修复，见第四节）**：2026-04-20 快照中 `pilot-shell/spec-guard/README.md` 存在但无伴生 `AGENT.md`，导致代理无法注册。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：70/100**（刚过阈值）
**安全等级：REVIEW**（无 Critical，有 High 发现，需人工审查）
**制品数量**：4189 个 NL 制品（抽样评分 100 个）

| 制品类型 | 当时得分范围 | 典型文件 | 主要扣分项 |
|---------|------------|---------|----------|
| `pilot-shell/spec-guard/README.md` | **20** | 唯一最低分 | 无伴生 AGENT.md、无 frontmatter、无示例 |
| `ruflo/*.README.md`（7 个存根）| 60 | 3–4 行内容 | 无人设、无专业描述、近零文档价值 |
| `quantum/*.AGENT.md`（20 个）| 66 | `quantum/hybrid-system-architect` | 无 model、无输出格式、无示例、模糊语言泛滥 |
| `civil/*.AGENT.md`（22 个）| 70 | `civil/structural-engineer` | 无 model、无输出格式、无示例 |
| `biomed/*.AGENT.md`（22 个）| 66–70 | `biomed/orthopedic-implant-tester` | 同上 |
| `elec/*.AGENT.md`（部分）| **90** | `elec/test-measurement-expert` | **（优秀）** 有输出格式 + 参考表 |

得分分布（100 个抽样）：

```
≥ 90       ████ 3 个（电气工程优秀代理）
80–89      ████████ 12 个
75–79      ████████ 8 个
70–74      ██████████████████████████████████████████████ 46 个（刚过阈值密集带）
60–69      ██████████████████████████████ 30 个
< 60       █ 1 个（spec-guard README，孤立最低分）
```

*注：被抽样的 100 个是 4189 个中表现最差的部分，整体分布上尾未被表征。*

**安全发现汇总：**

| 严重级别 | 数量 | 来源模式 |
|---------|------|---------|
| High | 4 | 四个钩子脚本均含 `export PATH="$HOME/.local/bin:$PATH"` |
| Medium | 8 | 四个钩子的 `npx -y` 运行时安装（×2 per hook：install + execute）+ `eval-compression.sh` heredoc 变量插值 |
| Low | 2 | `rollback-release.sh` 未校验 `$TAG`；钩子读取 `~/.claude/projects/`（目的合理但值得注意）|

**关键 Bug：**

`pilot-shell/spec-guard/README.md`（第 20 分文件）存在但没有伴生 `AGENT.md`。Claude Code 无法注册该代理；任何引用 `spec-guard` 的编排流程都会静默失败或报错。这不是文档问题，而是**代理不可用**问题。

**跨组件矛盾（最具实践价值的发现）：**

`CLAUDE.md` 声称存在一个 PostToolUse 钩子，在检测到 TypeScript 文件编辑时自动运行 `npm run lint`。但 `.claude/settings.json` 的内容是 `{}`，空配置。贡献者读 `CLAUDE.md` 会以为 lint 自动运行，实际上什么都没有发生——**文档超越了实现**。

### 3.2 当时值得借鉴的模式

**1. 版本锁定文件（`versions.json`）**

在运行时拉取 npm 包时，babysitter 通过 `versions.json` 文件固定要安装的包版本。钩子脚本先读取该文件，再执行 `npm install <package>@<version>`。这是一个值得借鉴的安全习惯：哪怕是运行时安装，也应当版本化。

```bash
# 实际调用模式（简化）
VERSION=$(cat versions.json | jq -r '.["@a5c-ai/babysitter-sdk"]')
npm install "@a5c-ai/babysitter-sdk@$VERSION" --prefix "$LOCAL_PATH"
```

**2. 电气工程代理（得分 90）作为同库对比基准**

`elec/test-measurement-expert.AGENT.md`、`elec/reliability-engineer.AGENT.md`、`elec/hardware-validation-engineer.AGENT.md` 是同库内的优秀代理：它们有**输出格式节**和**参考表格**。这证明"高质量代理定义在这个代码库里是可实现的"——其他领域的代理并非因为主题困难而缺失这些元素，而是因为**批量生成时省略了**。

**3. 多适配器插件架构**

`plugins/babysitter-github`、`plugins/babysitter-cursor` 等证明了"同一编排核心，多平台薄层适配"的可行性。每个适配器都是独立的 Claude Code 插件，用户按需安装。这是未来 Claude Code 生态插件设计的参考模式。

### 3.3 当时的缺陷

**Critical 缺陷（无，但 High 缺陷影响安全）：**

**High — PATH 劫持向量（4 处）**

所有四个钩子脚本（`babysitter-session-start-hook.sh`、`babysitter-stop-hook.sh`、`babysitter-pre-tool-use-hook.sh`、`babysitter-user-prompt-submit-hook.sh`）均含：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

在每次钩子执行时将用户本地目录注入 PATH 的前端位置。如果 `~/.local/bin/` 对非特权进程可写（在默认 Linux/macOS 配置下是真实的），任何能写入该目录的进程（恶意依赖、CI 构建产物）都可以用同名文件劫持后续命令执行路径。`pre-tool-use` 和 `user-prompt-submit` 钩子执行频率最高，攻击面最宽。

**Medium — 运行时供应链依赖（8 处）**

`npx -y @a5c-ai/babysitter-sdk` 在每次 `session-start`、`stop`、`pre-tool-use`、`user-prompt-submit` 时执行。`npx -y` 会在包不存在时自动安装——这意味着每次 Claude Code 会话都在 npm 注册表上建立一个实时依赖。`versions.json` 版本锁定缩小了窗口，但不能消除风险：被锁定版本的包本身也可能被投毒（npm 包的不可变性是最终发布策略，不是绝对保证）。

**Medium — Heredoc 变量插值（`eval-compression.sh`）**

`scripts/eval-compression.sh` 通过 heredoc 构造内联 Node.js 代码并执行：

```bash
node -e "$(cat <<EOF
const path = '$SOME_PATH';  # 文件系统派生路径流入可执行代码
...
EOF
)"
```

如果 `$SOME_PATH` 包含特殊字符（如反引号、`$()`），heredoc 内的变量展开会将其注入为代码。

**Low — 未校验 `$TAG`（`rollback-release.sh`）**

`scripts/rollback-release.sh` 将 `$TAG` 参数直接传入 `git push origin :refs/tags/$TAG`（删除远端 tag），未进行格式校验或白名单过滤。

**质量缺陷（系统性，跨全部被抽样代理）：**

1. **全部 100 个文件无示例块（examples block）**：零个代理定义包含输入 → 输出的具体示例。NLPM 每个文件扣 15 分。
2. **64 个代理缺少输出格式节（output format section）**：量子（20 个）、土木（22 个）、生物医学（22 个）领域全军覆没。
3. **91 个文件无 `model` 字段**：运行时路由完全依赖平台默认值，无法为高复杂度代理（如 `quantum/hybrid-system-architect`）保证模型能力。
4. **量子计算领域（20 个文件）：模糊量词泛滥**：动词 "support"、"analyze"、"optimize" 在文件间高频重复，无可验证的承诺。NLPM 每处模糊量词扣分。

### 3.4 当时的优化机会

按优先级排序：

| 优先级 | 改动 | 预估得分提升 |
|--------|------|-----------|
| P1（Bug）| 为 `pilot-shell/spec-guard` 添加 `AGENT.md` | spec-guard 从 20 → ≥ 65 |
| P2（Medium 安全）| 钩子改为预安装 SDK，移除 `npx -y` | 消除 8 个 Medium 发现 |
| P2（Medium 安全）| 修复 `eval-compression.sh` heredoc 变量插值 | 消除 2 个 Medium 发现 |
| P2（Low 安全）| `rollback-release.sh` 校验 `$TAG` 格式 | 消除 1 个 Low 发现 |
| P3（质量）| 为全部代理添加 `model` 字段 | +5 分/文件 × 91 = +455（归一化） |
| P3（质量）| 为全部代理添加 output format 节 | +10 分/文件 × 64 = +640（归一化）|
| P3（质量）| 为全部代理添加 examples 块 | +15 分/文件 × 100 = +1500（归一化）|
| P4（私下建议）| 将 `export PATH` 移出钩子 | 消除 4 个 High 发现 |

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-20 快照 | 2026-05-28 当前 HEAD | 状态 |
|------|----------------|---------------------|------|
| `spec-guard` 缺伴生 `AGENT.md` | 存在（Bug P1）| `spec-guard/AGENT.md` 现已存在 | ✅ **已修复** |
| `export PATH` 在四个钩子 | 全部 4 个钩子 | 仍在全部 4 个钩子脚本中 | ❌ 未修复 |
| `npx -y` 运行时安装 | 4 个钩子 | `babysitter-github/hooks` 脚本中仍可见 | ❌ 未修复 |
| `.claude/settings.json = {}` | 存在 | 未额外验证 | 未知 |
| 代理无 `model` 字段（91 个）| 存在 | 架构重组但质量规律预计不变 | 未验证 |
| 代理无 examples 块（100 个）| 存在 | 架构重组但质量规律预计不变 | 未验证 |

**spec-guard 修复** 是审查期间最直接可操作的 Bug，如今已修复。这一改变意味着 `pilot-shell` 下的编排流程现在可以正确注册并调用 spec-guard 代理，不再静默失败。

**PATH 修改仍未解决**是最值得关注的信号：从 2026-04-20 到 2026-05-28 超过 5 周时间，High 级别安全发现依然存在于全部四个钩子。这可能反映维护者认为这是"已知且可接受的权衡"，而非疏忽。

```bash
# 验证 PATH 修改是否仍存在：
grep -r "export PATH" plugins/babysitter/hooks/ 2>/dev/null
grep -r "export PATH" plugins/babysitter-github/hooks/ 2>/dev/null
```

```bash
# 验证 spec-guard AGENT.md 现在存在：
find . -path "*/spec-guard/AGENT.md" -type f 2>/dev/null
```

### 4.2 架构演进

2026-04-20 → 2026-05-28 之间，最重大的架构变化是：

**从"单体 Claude Code 插件"向"多平台编排框架"演进**

- 原来：一个 Claude Code 插件 + 四个 hook 脚本 + 4189 个领域代理定义
- 现在：`packages/` 内的 monorepo + `plugins/` 内 8 个平台适配器 + `library/` 内的领域代理库 + `packages/observer-dashboard`（运行状态可视化）

这次演进体现了从"Claude Code 专属工具"向"编辑器无关的 AI 代理编排层"的战略转型。`packages/observer-dashboard` 的加入表明框架开始关注可观测性（observability）——这是成熟代理编排系统的必备能力。

`library/SPEC-KIT.md` 和 `library/methodologies/` 的出现暗示项目开始将方法论提炼为可学习的框架知识，而非只是工具代码。

### 4.3 新增的可学习模式

**1. Observer Dashboard 模式（可观测性优先）**

`packages/observer-dashboard` 专门用于可视化代理运行状态。这是一个信号：规模达到一定程度后，编排框架必须提供运行时可见性，否则调试成本会急剧上升。

**学习点**：如果你的插件涉及多个代理并行执行，从一开始就规划一个简单的状态输出机制（哪怕只是一个 Markdown 表格），而非在复杂度上来后才追加。

**2. SPEC-KIT 方法论沉淀**

`library/SPEC-KIT.md` 将规格编写方法提炼为可复用的工具包。这类"方法论文档化"（把如何用框架的知识写成可引用的参考）是高质量插件的特征。

**学习点**：规模化插件应该有独立的 `library/` 或 `docs/` 目录存放方法论，而非全部混在 `CLAUDE.md` 或 `README.md` 里。

---

## 五、校准

### 5.1 我已经在做对的

1. **无运行时 npm 安装**：我的三个仓库（`drama-workshop-skills`、`claude-for-legal`、`echo-sleuth-for-claude`）均不在钩子或脚本中执行 `npx -y` 或运行时 `npm install`。这避免了 babysitter 的 Medium × 8 供应链发现。

2. **无 PATH 修改**：我的钩子不修改 `PATH` 环境变量。babysitter 的案例证明这一"便利性操作"会被评为 High 安全发现。

3. **体量适配规格**：我的三个仓库代理数量分别是 5、<10、<5 个，不存在 babysitter 的"4189 个文件导致质量无法逐一审查"的规模问题。质量控制在小体量时更易执行。

4. **`claude-for-legal` 有钩子实现与文档的一致性**：`claude-for-legal` 的每个子插件下 `hooks/hooks.json` 与 `CLAUDE.md` 中描述的钩子行为保持同步，没有 babysitter 的"文档说有钩子、配置是空的"的跨组件矛盾。

### 5.2 挑战 / 验证

**反直觉发现 1：70/100 的"刚过"意味着什么？**

Babysitter 在 4189 个制品上整体得分 70，处于门槛边缘。但被抽样的 100 个（最差的部分）有 30 个在 60–69 之间。这意味着整体"通过"实际上掩盖了底部有 30% 的制品低于阈值。

**学习**：当制品数量极大时，平均分可能迷惑性地高于阈值，而实际上有大量"拉低平均"的文件在线上运行。对于自己的仓库，不仅要看总分，还要看最低分是多少、有多大比例低于 70。

**反直觉发现 2：电气工程代理 90 分 vs 量子计算代理 66 分——同一仓库**

同一个仓库里，电气工程（3 个代理达到 90 分）和量子计算（20 个代理仅 66 分）之间存在 24 分的稳定差距。这个差距完全是**结构性的**（examples、output format、model 字段的有无），而非内容质量差距。这验证了 NLPM 的核心命题：结构缺陷可以被机械地检测和修复，与内容准确性无关。

**校准动作**：审查自己的代理文件时，先跑结构检查（有无 frontmatter 的 model、有无 output format 节、有无 examples 块），这些检查不需要理解领域知识，可以完全自动化。

**反直觉发现 3：spec-guard Bug（20 分）在 5 周内被修复，High 安全发现没有**

优先级逻辑：容易修复（加一个 AGENT.md 文件）的 Bug 比难修复（重新设计 SDK 交付机制）的安全发现更快被解决。这是**技术债的惰性**——不是不重要，而是改动成本高、影响面广，维护者优先解决了"摘到的低垂果实"。

对自己的校准：不要因为某个安全发现"知道存在"就假设它会被修复。周期性地重新检查高优先级发现是否仍然存在。

---

## 六、行动

### 6.1 自查动作

**自查 1：确认钩子不修改 PATH**

```bash
# 检查自己所有 hooks 相关脚本里是否有 PATH 修改
find ~/drama-workshop-skills ~/claude-for-legal ~/echo-sleuth-for-claude \
  -name "*.sh" -o -name "*.bash" 2>/dev/null \
  | xargs grep -l "export PATH" 2>/dev/null \
  && echo "FOUND PATH modification in hooks" \
  || echo "OK: No PATH modification in hooks"
```

**自查 2：确认钩子 JSON 中引用的脚本真实存在**

```bash
# 对每个 hooks.json，验证其引用的 command 脚本存在
for hooks_file in $(find ~/drama-workshop-skills ~/claude-for-legal ~/echo-sleuth-for-claude \
  -name "hooks.json" 2>/dev/null); do
  dir=$(dirname "$hooks_file")
  echo "=== Checking $hooks_file ==="
  python3 -c "
import json, os, sys
data = json.load(open('$hooks_file'))
for hook_type, hooks in data.items():
  if isinstance(hooks, list):
    for hook in hooks:
      if 'command' in hook:
        cmd_parts = hook['command'].split()
        script = cmd_parts[0] if cmd_parts else ''
        if script.startswith('./') or script.startswith('/'):
          full = os.path.join('$dir', script) if not script.startswith('/') else script
          if not os.path.exists(full):
            print(f'MISSING: {full}')
"
done
```

**自查 3：检查代理文件的三大结构要素**

```bash
# 对自己所有代理文件检查三个结构要素
for repo_dir in ~/drama-workshop-skills ~/claude-for-legal ~/echo-sleuth-for-claude; do
  echo "=== $repo_dir ==="
  
  # 1. 检查 model 字段
  echo "--- 缺少 model 字段的代理 ---"
  find "$repo_dir" -name "*.md" -path "*/agents/*" \
    | xargs grep -rL "^model:" 2>/dev/null
  
  # 2. 检查 output format 节（大小写不敏感）
  echo "--- 缺少 Output Format 节的代理 ---"
  find "$repo_dir" -name "*.md" -path "*/agents/*" \
    | xargs grep -rL -i "output.format\|输出格式\|## Output" 2>/dev/null
  
  # 3. 检查 examples 块
  echo "--- 缺少 Examples 块的代理 ---"
  find "$repo_dir" -name "*.md" -path "*/agents/*" \
    | xargs grep -rL -i "example\|示例\|## Example" 2>/dev/null
done
```

**自查 4：确认 CLAUDE.md 中声明的钩子在 settings.json 中真实存在（跨组件一致性检查）**

```bash
# 检查 CLAUDE.md 声明的钩子是否在实际配置里能找到
for repo_dir in ~/claude-for-legal ~/echo-sleuth-for-claude; do
  echo "=== $repo_dir ==="
  claude_md="$repo_dir/CLAUDE.md"
  
  # 找出 CLAUDE.md 中提到的 hook 类型
  grep -i "PostToolUse\|PreToolUse\|SessionStart\|UserPromptSubmit" "$claude_md" 2>/dev/null \
    | sed 's/.*\(PostToolUse\|PreToolUse\|SessionStart\|UserPromptSubmit\).*/\1/' \
    | sort -u
    
  # 找出实际 hooks.json 中配置的 hook 类型
  find "$repo_dir" -name "hooks.json" \
    | xargs python3 -c "
import json, sys
for f in sys.argv[1:]:
  data = json.load(open(f))
  print(f'{f}: {list(data.keys())}')
" 2>/dev/null
done
```

### 6.2 灵感 → 实施路径

**灵感 1：版本锁定文件（`versions.json` 模式）**

Babysitter 用 `versions.json` 固定 npm 包版本。即使我的仓库当前不需要运行时 npm 安装，这个模式可以扩展应用于任何"依赖外部资源版本"的场景。

**实施路径（针对 `echo-sleuth-for-claude`）**：
- 创建 `versions.json`，记录 scripts/ 下脚本所依赖的工具版本（如 `jq`、`python3`）
- 在安装文档中注明推荐版本
- 将其纳入 CI 检查（如果将来添加 CI）

**灵感 2：Observer / 运行状态可视化（Observer Dashboard 模式）**

`echo-sleuth-for-claude` 有 `memory-auditor` 代理，用于审查内存状态。但审查结果只输出到对话，没有持久化的状态视图。

**实施路径**：
1. 新建 `commands/status.md`：调用多个代理后收集摘要，写入 `.echo-sleuth/status.md`
2. `memory-auditor` 代理在执行后追加一条记录到 `.echo-sleuth/audit-log.jsonl`
3. `commands/dashboard.md`（如果已有）从该 jsonl 生成 Markdown 视图

**灵感 3：领域代理的质量分层策略（对抗批量生成的质量退化）**

Babysitter 的量子代理批量生成，结果全部停在 66 分。电气工程代理手工精调，达到 90 分。这个对比教给我一个实践原则：**批量生成后必须有质量分层验收**。

**实施路径（针对 `claude-for-legal`）**：
- 每次添加新的法律领域代理后，跑一次 NLPM score
- 设置一个本地检查脚本，强制每个代理有 output format 节和至少一个 example
- 对于"核心代理"（如 `regulatory-agent`、`litigation-agent`），额外要求 model 字段明确指定 Sonnet 而非 Haiku

```bash
# 法律代理质量验收脚本（在 claude-for-legal 根目录运行）
#!/bin/bash
FAILED=0
for agent_file in $(find . -name "*.md" -path "*/agents/*"); do
  name=$(basename "$agent_file")
  
  # 检查 model 字段
  if ! grep -q "^model:" "$agent_file"; then
    echo "WARN: $name 缺少 model 字段"
    FAILED=$((FAILED+1))
  fi
  
  # 检查 output format 节
  if ! grep -qi "output.format\|## Output" "$agent_file"; then
    echo "WARN: $name 缺少 Output Format 节"
    FAILED=$((FAILED+1))
  fi
  
  # 检查 examples 块
  if ! grep -qi "example\|示例" "$agent_file"; then
    echo "WARN: $name 缺少 Examples 块"
    FAILED=$((FAILED+1))
  fi
done

echo "共发现 $FAILED 个结构缺陷"
[ $FAILED -eq 0 ] && echo "PASS" || exit 1
```

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | babysitter 的目的 | 对齐度 | 对齐说明 |
|---------|-----------------|--------|---------|
| MarkQWu/drama-workshop-skills | 确定性多代理编排框架 | **低** | drama-workshop-skills 是垂直技能库（短剧生成），没有编排层。两者定位完全不同。 |
| MarkQWu/claude-for-legal | 确定性多代理编排框架 | **中** | claude-for-legal 有多代理协作（sub-plugins 间通过 hooks 协调），有编排意图，但规模和复杂度远低于 babysitter。 |
| MarkQWu/echo-sleuth-for-claude | 确定性多代理编排框架 | **低** | echo-sleuth 是会话历史分析工具，5 个代理但各自独立运行，不涉及代理间编排。 |

### 8.2 在我的项目里复现的同类问题

**问题 1：`echo-sleuth-for-claude` 代理无 `model` 字段（与 babysitter 91/100 代理相同）**

Babysitter 有 91 个代理没有声明 model 字段，echo-sleuth-for-claude 的 5 个代理同样可能缺失。

```bash
# 立即验证
find ~/echo-sleuth-for-claude/agents -name "*.md" \
  | xargs grep -rL "^model:" 2>/dev/null \
  | while read f; do echo "需要添加 model 字段: $f"; done
```

若命中：根据代理复杂度决定 model 值——`recall`、`file-historian` 等查询类代理用 `claude-haiku-4-5`；`analyze`、`memory-auditor` 等推理类代理用 `claude-sonnet-4-5`。

**问题 2：`echo-sleuth-for-claude` skills 含模糊量词（与 babysitter 量子计算领域相同）**

据 PROGRESS.md 记录，echo-sleuth 的 skills 目录有 2 处模糊量词命中（"significantly"、"various" 等英文或中文等价词）。Babysitter 量子计算领域的 "support"、"analyze"、"optimize" 就是模糊量词的典型——这些词告诉 Claude 要做什么，但没有说**做到什么程度**算完成。

```bash
# 检查 echo-sleuth skills 中的模糊量词
grep -r "\beffectively\b\|\befficiently\b\|\bsignificantly\b\|\bvarious\b\|\bappropriate\b" \
  ~/echo-sleuth-for-claude/skills/ 2>/dev/null
```

修复方向：将 "effectively summarize" → "提取前 5 个关键主题，每个附一句话描述"；将 "analyze patterns" → "列出出现频率最高的 3 类操作，按次数降序排列"。

**问题 3：`claude-for-legal` 可能存在 CLAUDE.md 与 hooks.json 不一致（与 babysitter 跨组件矛盾相同）**

Babysitter 的 CLAUDE.md 描述了一个 PostToolUse lint 钩子，但 `.claude/settings.json` 是空的。`claude-for-legal` 有多个子插件，每个子插件下都有 `hooks/hooks.json`，同时 CLAUDE.md 中描述了 Managed Agents API 的钩子行为。这两者是否一致？

```bash
# 快速检查 CLAUDE.md 中描述的钩子类型与 hooks.json 实际配置的钩子类型
for plugin_dir in ~/claude-for-legal/*/; do
  [ -f "$plugin_dir/CLAUDE.md" ] && [ -f "$plugin_dir/hooks/hooks.json" ] || continue
  echo "=== $plugin_dir ==="
  echo "CLAUDE.md 提到的钩子:"
  grep -i "hook\|PostToolUse\|PreToolUse\|SessionStart\|UserPromptSubmit" \
    "$plugin_dir/CLAUDE.md" 2>/dev/null | head -5
  echo "hooks.json 实际配置:"
  python3 -c "import json; d=json.load(open('${plugin_dir}hooks/hooks.json')); print(list(d.keys()))"
done
```

### 8.3 别人的更优方案

**1. 版本锁定文件（babysitter 优于我的仓库）**

Babysitter 用 `versions.json` 固定运行时依赖版本，尽管运行时安装本身是个反模式，但版本锁定是正确思路。我的三个仓库均没有记录脚本依赖的工具版本。对于 `echo-sleuth-for-claude` 中依赖 `jq`、`python3` 的 shell 脚本，至少应在 README 中记录测试通过的工具版本。

**2. 电气工程代理的 Output Format + 参考表（babysitter `elec/` 系列优于我的代理）**

`elec/test-measurement-expert.AGENT.md` 包含参考表格（测量单位、容差范围、标准代号），是领域知识的结构化沉淀。我的 `claude-for-legal` 的法律代理可以借鉴这个模式，为每个代理加入参考表（如"适用法条速查表"、"诉讼时效对照表"），使代理即使在 Claude 无法访问外部知识库时也能提供有价值的结构化参考。

**3. 多平台适配器架构（babysitter 优于我的仓库）**

Babysitter 的 `plugins/babysitter-github`、`plugins/babysitter-cursor` 等展示了如何将同一套核心逻辑适配到不同 AI 工具生态。`claude-for-legal` 目前只适配 Claude Code，但法律工作流在 GitHub Copilot、Cursor 等工具里同样有需求。

### 8.4 反向：我的项目做得比他们好的地方

1. **无运行时供应链依赖（我的全部仓库优于 babysitter）**

   我的三个仓库均为纯 Markdown 插件，不依赖 `npx -y` 或运行时 `npm install`。任何用户安装后的行为都是完全可审计的——代码库里有什么就运行什么。babysitter 的 `npx -y @a5c-ai/babysitter-sdk` 在每个 session 引入了一个实时的外部信任链。这是我最应该保持的优势。

2. **代理数量与质量的平衡（我的仓库优于 babysitter）**

   babysitter 有 4189 个代理定义，其中抽样的 100 个全部缺少 examples 块，64 个缺少 output format 节。质量与数量严重失衡。我的仓库代理数量少（5–10 个），每个代理可以精心打磨。规模小是约束，也是质量可控的保障。

3. **钩子与文档的一致性（`claude-for-legal` 优于 babysitter）**

   Babysitter 的 CLAUDE.md 声明了一个不存在于 `.claude/settings.json` 的 PostToolUse lint 钩子——这是一个"文档承诺但代码没有兑现"的典型跨组件矛盾。`claude-for-legal` 的每个子插件下 `hooks/hooks.json` 与其 CLAUDE.md 描述保持一致，不存在"空 settings 假装有钩子"的问题。

4. **`drama-workshop-skills` 的结构化质量控制（3 层控制模型）**

   据项目描述，`drama-workshop-skills` 有 7 维度质量管控（剧本生成的 7 个质量维度）。这是一个比 babysitter 更内聚的质量设计——每个代理知道它要满足哪些具体的质量标准（如"情节逆转次数不少于 2 次"），而非模糊的"generate high-quality scripts"。

---

## 八、术语表

| 术语 | 说明 |
|------|------|
| <a name="event-sourced">事件溯源（Event-Sourced）</a> | 一种状态管理模式，系统状态由不可变事件序列（event log）派生，而非直接修改当前状态。babysitter 用 effect journal 记录每个代理的输出效果，新代理读取 journal 而非整个对话历史，确保状态转移的可重现性。 |
| <a name="effect-journal">Effect Journal（效果日志）</a> | babysitter 中记录每个代理执行结果的结构化日志。替代对话历史作为代理间上下文传递的媒介，实现代理隔离执行和运行回放。 |
| <a name="供应链攻击">供应链攻击（Supply Chain Attack）</a> | 攻击者通过污染上游依赖包（如 npm 包），在目标机器上执行恶意代码的攻击方式。babysitter 在每次钩子执行时运行 `npx -y @a5c-ai/babysitter-sdk`，每次都依赖 npm 注册表——这是一个持续开放的供应链风险窗口。 |
| <a name="npx-y">npx -y（无确认安装）</a> | `npx -y` 在包不存在于本地缓存时，自动从 npm 注册表下载并执行，无需用户确认。在钩子脚本中使用此模式意味着每次 Claude Code 会话启动都会触发一次潜在的外部包下载。 |
| <a name="path-hijacking">PATH 劫持（PATH Hijacking）</a> | 通过将攻击者控制的目录放在 `PATH` 前面，使系统在执行命令时优先找到恶意同名程序。babysitter 的 `export PATH="$HOME/.local/bin:$PATH"` 将用户本地目录注入到搜索路径前端，若该目录可被低权限进程写入则产生此风险。 |
| <a name="frontmatter">Frontmatter（前置元数据）</a> | Markdown 文件开头由 `---` 包围的 YAML 元数据块。Claude Code 用它解析命令/代理的 `name`、`description`、`model`、`allowed-tools` 等属性。babysitter 91 个代理文件缺少 `model:` 字段，导致运行时路由依赖平台默认值。 |
| <a name="model-pinning">model-pinning（模型固定）</a> | 在代理或命令的 frontmatter 中通过 `model:` 字段显式指定使用的 Claude 模型 ID。未固定 model 时，运行时选用平台默认值，可能导致复杂代理使用能力不足的模型。babysitter 91/100 个被抽样代理缺少此字段。 |
| <a name="output-format-section">Output Format 节</a> | 代理定义文档中的一个专用节（通常以 `## Output Format` 标题），描述代理输出的结构（JSON schema、Markdown 表格、纯文本等）。缺少此节意味着调用者必须实际运行代理才能了解输出格式。babysitter 64 个代理缺此节。 |
| <a name="examples-block">Examples 块</a> | 代理或技能文档中的具体输入→输出示例，展示代理在真实场景下的行为。NLPM 对缺少 examples 的文件每处扣 15 分。babysitter 全部 100 个被抽样文件均无此块。 |
| <a name="vague-quantifier">模糊量词（Vague Quantifier）</a> | 在代理描述中使用无法验证的抽象动词，如 "support"（支持）、"analyze"（分析）、"optimize"（优化）、"effectively"（有效地）。模糊量词无法告诉 Claude 什么情况算"完成"，导致输出不可预期。babysitter 量子计算领域 20 个代理文件中此类词高频出现。 |
| <a name="heredoc-interpolation">Heredoc 变量插值（Heredoc Variable Interpolation）</a> | 在 Bash heredoc 中使用 `$VAR` 变量展开，当 heredoc 内容是被执行的代码（如内联 Node.js）时，若变量值来自文件系统或用户输入且未经转义，可能注入任意代码片段。babysitter `eval-compression.sh` 中存在此模式。 |
| <a name="cross-component-contradiction">跨组件矛盾（Cross-Component Contradiction）</a> | NL 制品间的不一致：一个文件（如 CLAUDE.md）声明某行为存在，而负责实现该行为的另一个文件（如 `settings.json`）不包含对应配置。babysitter 的"CLAUDE.md 声明 PostToolUse lint 钩子、settings.json 为空"是典型案例。 |
| <a name="versions-json">versions.json（版本锁定文件）</a> | babysitter 用于固定运行时安装的 npm 包版本的 JSON 文件。钩子脚本先读取此文件中的版本号，再执行带版本约束的 `npm install`，将供应链攻击窗口从"任意最新版"缩小到"特定版本"。 |
| <a name="observer-dashboard">Observer Dashboard（可观测性面板）</a> | babysitter 的 `packages/observer-dashboard`，用于可视化代理运行状态的独立包。代表了复杂代理编排系统从"能运行"到"可观测"的成熟度演进。 |
| <a name="spec-kit">SPEC-KIT（规格套件）</a> | babysitter `library/SPEC-KIT.md` 中沉淀的规格编写方法论。将如何正确定义代理规格（spec）的知识工具化，是大规模代理库质量管理的基础设施。 |
| <a name="multi-platform-adapter">多平台适配器模式（Multi-Platform Adapter Pattern）</a> | 将核心编排逻辑与平台绑定完全解耦，为每个目标平台（Claude Code、Cursor、GitHub Actions 等）编写一个独立的薄层适配插件。babysitter 的 `plugins/` 目录下 8 个子插件实现了此模式。 |
| <a name="monorepo">Monorepo（单仓多包）</a> | 将多个相关包（packages）放在同一个 git 仓库管理的组织方式。babysitter 2026-05 架构将核心运行时（`packages/babysitter`）、SDK（`packages/sdk`）、观测面板（`packages/observer-dashboard`）放在同一仓库，共享 CI/CD 和版本管理。 |
| <a name="nlpm">NLPM（Natural-Language Programming Manager）</a> | 用于评分、检查、修复自然语言制品（如 Claude Code 的命令、代理、技能定义）的质量管理工具。本案例研究基于 NLPM audit 数据生成。评分从 100 分起，按确定性惩罚表扣分，下限 0，通过阈值默认 70。 |
