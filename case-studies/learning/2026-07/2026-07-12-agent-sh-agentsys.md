# agent-sh/agentsys — 学习案例

**仓库**：https://github.com/agent-sh/agentsys
**Stars**：N/A（注册时未记录）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-07-12（基于当前 HEAD）
**主题标签**：`single-purpose`, `experience-accumulation`, `fallback-chain`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`agentsys`（agent-sh/agentsys）是一个面向 Claude Code 和 Kiro IDE 的"agent 增强与优化系统"——提供 30+ 个高质量技能，覆盖 agent 增强（enhance-*）、性能分析（perf-*）、学习与反思（learn / drift-analysis）等场景，并附带一个 [原生二进制核心](#原生二进制核心)（`bin/` 目录下的 CLI 工具）。

关键事实：
- Audit 时 32 个 artifact，当前仍为 `.kiro/skills/` 目录下约 30 个技能
- 安全评级：CLEAR（0 Critical / 0 High）——在所有被 NLPM 审计的仓库中属极少数无高风险问题的
- 与 `avifenesh/agentsys`（741 stars）是关联仓库，后者有 xiaolai 案例：`case-studies/2026-04-24-avifenesh-agentsys.md`
- 分发方式：`npm install agentsys` + 自包含 CLI

### 1.2 架构剖析

**目录结构（关键层级）**：
```
agentsys/
├── .kiro/skills/       # 30 个 Kiro IDE 技能（每个 skill 一个目录）
│   ├── consult/        # 技能：咨询顾问模式
│   ├── debate/         # 技能：辩论反驳
│   ├── deslop/         # 技能：去除 AI 回复废话
│   ├── enhance-*/      # 系列：增强各类 NL artifact
│   ├── learn/          # 技能：自适应学习
│   ├── drift-analysis/ # 技能：检测代码质量漂移
│   └── (perf-* 系列已从此目录移除)
├── meta/skills/        # 跨平台维护技能（maintain-cross-platform）
├── bin/                # 原生 CLI
├── scripts/            # 工具脚本（detect.js, dev-install.js 等）
├── adapters/           # 平台适配器
├── __tests__/          # 测试套件（perf-* 测试已迁移到此处！）
├── lib/                # 核心库
├── package.json        # npm 包定义（版本 5.x）
└── CLAUDE.md           # 项目规则文件
```

**文件类型分布**：
- SKILL.md（.kiro/skills/）：约 30 个
- meta/skills/：1 个（maintain-cross-platform，1024 行超大型跨平台维护指南）
- agent：0 个（纯 skill 设计，无独立 agent）
- command：0 个
- hook：0 个（现已移除）

**编排关系**：
- `.kiro/skills/` 下的技能是独立平列的，没有 router 或 meta skill
- `enhance-orchestrator/` 是例外：它引用其他 enhance-* 技能（enhance-agent-prompts / enhance-prompts 等），扮演轻量编排角色
- `meta/skills/maintain-cross-platform/SKILL.md` 作为全局"安装指南"，被所有技能隐式依赖
- `perf-*` 系列（原 .kiro/skills/ 下）已从 NL 技能层移除，**迁移到 `__tests__/` 目录**（JavaScript 测试），说明作者决定用测试代替 NL 描述来定义性能行为

**跨件契约**：
- 所有技能通过 `../../scripts/` 相对路径引用工具脚本，由 `maintain-cross-platform/SKILL.md` 统一规范
- `docs/perf-requirements.md` 作为 perf-* 系列的"规格书"，虽未在 audit 集合中，但在各 skill 中被一致引用

### 1.3 设计思路 / 方法论

- **核心设计哲学**："让 AI 更像一个优秀的思考伙伴"——每个技能都在训练模型做某类高质量推理（辩论、反驳、去废话、自我漂移检测）
- **解决什么问题**：默认的 Claude Code 回复容易"和稀泥"，agentsys 的技能给模型设定了更高的认知标准
- **Trade-off**：
  - 技能数量少而精（30 个 vs affaan-m 的 934 个），但单个质量极高（平均 91 分）
  - 专注 Kiro IDE，牺牲了 Claude Code 原生适配面
  - 保留原生 CLI（`bin/`）而非纯 NL，实现了 NL 与原生的混合架构
- **认知模型**：作者认为"AI agent 的行为可以通过 NL 技能系统性地升级"——不是给 Claude 更多数据，而是给它更好的"思维框架"

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「NL 表皮 + 原生 CLI 核心」(NL surface + native binary core)

模式特征清单：
- 特征 1：**NL 层薄而精**——30 个技能，每个聚焦单一认知任务（辩论 / 去废话 / 漂移检测）
- 特征 2：**原生 CLI 作为骨架**——复杂的算法逻辑（性能分析、性能基线管理）放在 JS/Node.js 层
- 特征 3：**跨平台维护是一等公民**——`maintain-cross-platform/SKILL.md` 作为专门技能存在，体现对工程可维护性的重视
- 特征 4：**版本对齐强约束**——`package.json` 和 `.claude-plugin/plugin.json` 共同维护版本号（5.8.3）
- 特征 5：**无 agent / 无 command**——只有 skill，每个 skill 是一个"思维增强器"而非"任务执行器"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|------|---------|------|
| 需要提升 AI 推理质量（去废话/辩证思考）| ✅ 高度适用 | 每个 skill 专注于一类认知升级 |
| 需要原生 CLI 与 NL 协同工作 | ✅ 适用 | bin/ 提供 CLI，SKILL.md 提供 NL 交互层 |
| 需要大量任务型命令（如代码生成）| ❌ 不适用 | 无 command，无 agent，不是任务型设计 |
| 只用 Claude Code（非 Kiro）| ❌ 不适用 | 主要面向 Kiro IDE |
| 希望覆盖多个技术垂类 | ❌ 不适用 | 聚焦认知增强，无技术垂类 skill |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|------|---------|------|------|
| NL 表皮 + CLI 核心（本仓库）| agent-sh/agentsys | 质量极高，可测试，可 npm 安装 | 覆盖面窄，不面向 Claude Code 原生 |
| 全域 NL 工具箱 | affaan-m/everything-claude-code | 覆盖 80+ 垂类 | 质量不均，安全风险 |
| 垂类精品仓库 | alirezarezvani/claude-skills | 垂类深度极高 | 无 CLI 支持，依赖纯 NL |

### 2.4 改进空间

1. **当前问题**：.kiro/skills/web-auth 和 web-browse 中历史上存在硬编码用户路径（已修复），但修复方式未记录。**改进做法**：在 `maintain-cross-platform/SKILL.md` 中明确禁止硬编码绝对路径，加入"禁止 `/Users/` 或 `/home/用户名/`"的验证 step。**预期收益**：将路径安全性文档化，防止后续贡献者重蹈覆辙
2. **当前问题**：`perf-*` NL 技能被移除，改为 JS 测试，但 NL 层的使用说明随之消失。**改进做法**：保留一个 `perf-overview/SKILL.md` 作为入口，指向 `docs/perf-requirements.md` 和测试文件。**预期收益**：用户不需要读 JS 测试代码就能理解性能功能

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-12 当时得分 91/100（批量扫描 32 个 artifact）

| 文件 | 当时分数 | 主要问题 |
|------|---------|---------|
| .kiro/skills/sync-docs/SKILL.md | 83 | 模糊量词 + 缺示例 |
| .kiro/skills/perf-*/SKILL.md × 12 | 85 | 缺示例块 |
| meta/skills/maintain-cross-platform/SKILL.md | 95 | 超过 500 行（实 1024 行） |
| .kiro/skills/consult/SKILL.md | 96 | 近完美 |
| .kiro/skills/orchestrate-review/SKILL.md | 98 | 近完美 |

### 3.2 当时值得借鉴的模式

1. **`deslop` / `debate` 系列：功能单一的认知技能** → 每个技能只做一件事，名字直接描述功能。根本原因：单职责原则让技能组合灵活、测试容易。借鉴：我的技能应避免"万能型"描述

2. **`orchestrate-review/SKILL.md` 98 分** → 引用其他具体技能名，有清晰的触发时机和输出格式。借鉴：多技能编排要明确"在哪种条件下触发哪个技能"

3. **`maintain-cross-platform/SKILL.md` 作为工程规范技能** → 把"如何跨平台维护"本身写成 SKILL.md，而非散落在 README 里。根本原因：技能系统把工程知识变成可执行的规范。借鉴：可以为自己的仓库写一个 `how-to-contribute/SKILL.md`

4. **版本双重锁定**：`package.json` 和 `.claude-plugin/plugin.json` 都声明 `5.8.3`——单一真相来源。借鉴：版本号必须在所有 manifest 中保持一致

5. **无 broken references**：所有内部相对路径（`../../scripts/detect.js` 等）均被 `maintain-cross-platform/SKILL.md` 规范化，audit 期间 0 broken reference。借鉴：相对路径约定要写入文档

### 3.3 当时的缺陷

1. **12 个 perf-* 技能缺示例块** → 为什么失败：没有 input → output 示例，用户不知道"调用这个技能后会得到什么"。自查：echo-sleuth 的 4 个 skills 全部缺示例，同等问题

2. **web-auth / web-browse 技能中硬编码 `/Users/avifen/` 绝对路径** → 为什么失败：在任何非原开发者机器上，命令会因路径不存在而静默失败。根本原因：技能在单一机器上开发，没有跨机器测试流程

3. **`prepare` 脚本在 `npm install` 时自动安装 git hooks** → 为什么失败：`prepare` lifecycle 在 `npm install` 时执行，用户不知情地被安装了 git 钩子，有侵入性。自查：我的 `package.json` 是否有类似隐式副作用？

### 3.4 当时的优化机会

1. 为 12 个 perf-* 技能批量添加 `<examples>` 块（每个 -15 分的主因）
2. 将 hardcoded paths 替换为 `~/.agentsys/...` 或 `$AGENTSYS_DIR` 变量
3. 将 `prepare` 脚本重命名为 `setup-hooks`（仅开发者手动调用）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---------|---------|------|------|
| web-auth / web-browse 硬编码 `/Users/avifen/` | `grep -rn '/Users/avifen' .kiro/skills/web-*/SKILL.md` | ✅ **已修复**：0 次命中 | 跨机器路径问题已解决 |
| `prepare` 脚本自动安装 git hooks | `grep -n 'prepare' package.json` | ✅ **已修复**：脚本已更名为 `setup-hooks`，不再自动执行 | 侵入性问题消除 |
| perf-* 技能缺示例块 | `find .kiro/skills -name 'perf-*'` | ⚠️ **架构变更**：perf-* NL 技能从 .kiro/skills/ 完全移除，迁移到 `__tests__/` JS 测试 | 作者选择"用测试代替 NL 描述"，原缺陷通过删除而非修复解决 |

### 4.2 架构演进

最重要的变化：**perf-* NL 技能全部从 `.kiro/skills/` 目录移除**，相应的测试逻辑移到 `__tests__/` 目录（JS 测试：perf-schemas.test.js / perf-optimization-runner.test.js 等）。这说明作者意识到：
- 复杂算法行为用 NL 描述不够精确，用测试更可靠
- NL 层应该专注于"认知增强"（辩论 / 去废话 / 漂移检测），而非"算法规格"

这是一个罕见的"从 NL 退化到原生"的架构演进，说明 NL 编程也有其适用边界。

### 4.3 新增的可学习模式

**测试驱动的 NL 边界**：当某类行为需要精确规格时（如性能基准测试），作者选择了 JavaScript 单元测试而非 NL 描述——这是对"NL 编程适用边界"的清醒认识。我应该问自己：哪些场景应该用 NL 描述，哪些场景应该用测试？

---

## 五、校准

### 5.1 我已经在做对的

1. **单功能技能命名**：echo-sleuth 的技能名（experience-synthesis / git-mining / jsonl-core）都是单功能的，符合 agentsys 的单职责风格
2. **有 tests/ 目录**：echo-sleuth 有测试目录，体现质量文化
3. **无侵入性安装副作用**：我的仓库没有在 `npm install` 时自动执行有副作用的脚本

### 5.2 挑战 / 验证

本案例**挑战**了我"NL 技能可以描述任何行为"的假设。perf-* 系列的迁移告诉我：**当行为需要精确定量规格时，测试比 NL 更可靠**。未来设计技能时，我会先问："这个行为是定性的（适合 NL）还是定量的（适合测试代码）？"

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 文件是否有 examples 块
grep -rL '<examples>\|## Example\|example' /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md

# 检查我的 package.json 是否有侵入性 lifecycle scripts
grep -A3 '"scripts"' /tmp/my-repos/MarkQWu-*/package.json 2>/dev/null | grep -E 'prepare|postinstall|preinstall'

# 检查我的技能是否有硬编码绝对路径
grep -rn '/Users/\|/home/[a-z][a-z]*/' /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null
```

命中后怎么办：
- 缺 examples：参考 agentsys 的 `consult/SKILL.md`（96 分），加至少 2 个 `<example>` 块
- `prepare` 或 `postinstall`：将其重命名，加 `# 手动运行` 注释，告知贡献者
- 硬编码路径：替换为 `$HOME/` 或环境变量 `$MY_REPO_DIR`

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 4 个 skills 各加一个 `deslop`-style 的精简示例
   - **为何可行**：agentsys 的 `deslop/SKILL.md` 展示了"AI 回复前 vs 后"的对比格式，简洁有力
   - **第一步**：打开 echo-sleuth 的 `experience-synthesis/SKILL.md`，仿照 deslop 格式加 1 个 before/after 示例——20 分钟

2. **想法**：为 bureau 仓库写一个 `how-to-contribute/SKILL.md`（参考 agentsys 的 maintain-cross-platform）
   - **为何可行**：bureau 有 crew/ hooks/ commands/ 多个目录，新贡献者需要维护指南
   - **第一步**：在 bureau 的 skills/ 目录下新建 `how-to-contribute/SKILL.md`，描述各目录用途和命名约定——30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 agent-sh/agentsys 的核心目的**：通过 NL 技能和原生 CLI 提升 Claude 的认知质量（去废话、辩证推理、性能分析）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|-------|-------|-------|----------|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code 插件；有 skills + agents | 目标是挖掘历史会话，非认知增强 | 高 |
| MarkQWu/bureau | 低 | 同为 Claude Code 插件 | 知识库管理，无认知增强目标 | 低 |
| MarkQWu/gstack | 低 | 都有原生 CLI | gstack 是 NL + 二进制混合，目标是浏览器操控，非认知增强 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---------|---------|--------------|------|
| skills 缺 `<examples>` 块 | `grep -rL 'example' .../skills/*/SKILL.md` | **echo-sleuth 全中 4/4**（experience-synthesis / git-mining / jsonl-core / memory-management）| 高 |
| 硬编码绝对路径 | `grep -rn '/Users/\|/home/' .../skills/` | echo-sleuth: 0 命中（良好）| 无 |
| package.json 侵入性脚本 | `grep 'prepare\|postinstall'` | bureau / echo-sleuth：无此类脚本（良好）| 无 |

**命中后的具体行动建议**：
- `MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md` → 加 2 个 `<example>` 块（input: "Session log content", output: "提取出的经验"）→ 20 分钟
- `MarkQWu/echo-sleuth-for-claude/skills/git-mining/SKILL.md` → 加 1 个 `<example>` 展示 git log 解析前后 → 15 分钟

### 8.3 别人的更优方案

1. **领域**：`maintain-cross-platform` 工程维护技能
   - **本案例做法**：`meta/skills/maintain-cross-platform/SKILL.md` 是专门描述"如何维护这个仓库的跨平台约定"的技能，1024 行，非常详细
   - **我的项目现状**：echo-sleuth 无任何维护文档，只有 README
   - **如何借鉴**：在 echo-sleuth 的 `.claude/` 目录下写一个 `CONTRIBUTING-SKILL.md`，描述 skill / agent / command 的写作规范——参考 agentsys 的结构，但控制在 200 行以内

2. **领域**：版本双重锁定
   - **本案例做法**：`package.json` 和 `.claude-plugin/plugin.json` 都声明同一版本号
   - **我的项目现状**：echo-sleuth 有 `nlpm-badge.json`，但无 `plugin.json`——版本信息不完整
   - **如何借鉴**：在 echo-sleuth 中加 `.claude-plugin/plugin.json`，与 `nlpm-badge.json` 保持版本一致

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：同时覆盖 Claude Code 和标准 agents
- **我的做法**：echo-sleuth 有 `agents/` 目录，可在 Claude Code 和通过 agent protocol 使用
- **本案例做法（弱在哪）**：agentsys 主要面向 Kiro IDE（.kiro/skills/），Claude Code 原生支持有限
- **意义**：我的跨平台策略更广，若 Kiro 生态未达主流，echo-sleuth 的覆盖面更稳定

---

## 八、术语表

### <a name="原生二进制核心"></a>原生二进制核心
> 用 JavaScript / TypeScript / Go / Rust 这类语言写出来、编译成可执行文件的程序。"原生"意思是不依赖 Claude 就能直接运行，"二进制"（或"原生 CLI"）意思是可以在命令行直接调用，如 `agentsys enhance`。**对比**：SKILL.md 是纯文本，只能在 Claude Code 会话中被读取执行；原生 CLI 可以在 shell 脚本、CI 流水线中独立运行。
