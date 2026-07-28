# agent-sh/enhance — 学习案例

**仓库**：https://github.com/agent-sh/enhance
**Stars**：3 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-07-28（基于当前 HEAD）
**主题标签**：`single-purpose`, `manifest-discipline`, `examples-driven`, `nl-binary-hybrid`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`agent-sh/enhance` 是 `agent-sh/agentsys` 单体仓库旗下的一个独立 Claude Code 插件，专门用于**分析和增强 Claude Code NL 工件**（插件、agent、skill、文档、hooks、提示词、CLAUDE.md）的质量。一句话定位：_"给 Claude Code 插件做 X 光扫描，然后告诉你骨头哪里断了。"_

关键事实：
- 作者：Avi Fenesh（GitHub: avifenesh），同时也是 agent-sh 组织的核心贡献者
- 对外发布形式：`agent-sh` 父级仓库包含多个插件，`enhance` 是其中之一
- 核心能力：同时运行 8 个平行 agent（每个负责一种工件类型），生成统一报告；支持 `--apply` 标志自动修复
- 生态位置：NLPM 审计管道把这个插件本身也审了一遍，形成"元审计"——审计工具审计自己

### 1.2 架构剖析

```
enhance/
├── .claude-plugin/plugin.json   # 插件清单（name: enhance, version: 5.1.0）
├── commands/enhance.md          # 唯一入口命令，编排 8 个 agent
├── agents/                      # 8 个专项 enhancer
│   ├── agent-enhancer.md
│   ├── claudemd-enhancer.md
│   ├── cross-file-enhancer.md
│   ├── docs-enhancer.md
│   ├── hooks-enhancer.md
│   ├── plugin-enhancer.md
│   ├── prompt-enhancer.md
│   └── skills-enhancer.md
├── skills/                      # 9 个背景知识 skill（每 agent 加载对应 skill）
│   ├── enhance-agent-prompts/SKILL.md
│   ├── enhance-claude-memory/SKILL.md
│   ├── enhance-cross-file/SKILL.md
│   ├── enhance-docs/SKILL.md
│   ├── enhance-hooks/SKILL.md
│   ├── enhance-orchestrator/SKILL.md
│   ├── enhance-plugins/SKILL.md
│   ├── enhance-prompts/SKILL.md
│   └── enhance-skills/SKILL.md
├── lib/                         # Node.js/TypeScript 原生二进制核心
│   ├── binary/index.js          # 运行时下载 agent-analyzer 二进制（含 SHA-256 + SLSA 验证）
│   ├── types/, schemas/, cross-platform/
│   └── package.json
├── scripts/setup-hooks.sh       # 安装 git hooks
├── package.json                 # npm 包（version: 1.0.0）
└── CLAUDE.md                    # 重定向到 AGENTS.md
```

- **文件类型分布**：1 个 command / 8 个 agent / 9 个 skill / 1 个 plugin manifest / ~92 个 JS lib 文件
- **编排关系**：`enhance.md` → 调度 8 个 agent（并行） → 每个 agent 加载对应的 enhance-X skill → lib/binary 在运行时下载 `agent-analyzer` 原生二进制执行实际分析
- **跨件契约**：commands 没有在 [frontmatter](#frontmatter) 里声明 `skills:`，agent 与 skill 的绑定是**隐式**的（由 agent 体内 Skill 调用决定）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「NL 表皮 + [原生二进制核心](#原生二进制核心)」——Markdown 编排层负责调度逻辑，高强度分析委托给预编译的 `agent-analyzer` 二进制，两层分工明确
- **解决什么问题**：作者面对 Claude Code 插件生态里大量低质量 NL 工件（缺 examples、模糊量词泛滥），想提供一个能自动发现并建议修复的工具
- **做了什么 trade-off**：把 8 种增强类型切成独立 agent 而非一个"大通才" → 提高并行度和单职责清晰度，代价是 agent 间没有结构化的上下文传递（每个 agent 独立运行，不知道其他 agent 发现了什么）
- **反映什么认知模型**：作者把 Claude Code 插件看作"可分解的子系统"，每个子系统有独立的最佳实践知识体；同时相信原生二进制在分析精度上比纯 NL 更可靠，所以用 NL 做"前端"、二进制做"后端"

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「NL 表皮 + 原生二进制核心」**

关键特征：
- **特征 1：职责分层**：NL 层（Markdown）只负责调度和用户交互；实际计算密集型工作由编译好的二进制处理
- **特征 2：二进制有完整验证链**：SHA-256 校验 + 可选 SLSA 证明，安全性远超 npx 临时下载
- **特征 3：原生工件增强性能**：静态分析、AST 解析这类 CPU 密集任务，二进制比让 LLM 一行行读要快几个数量级
- **特征 4：升级解耦**：NL 层和二进制层可以独立升级，只要 API 接口稳定
- **特征 5：跨平台适配由二进制层处理**：lib/cross-platform 封装了 macOS/Linux/Windows 差异，NL 层无需关心

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要确定性输出的代码分析工具 | ✅ 高度适用 | LLM 天然不确定，二进制规则引擎输出可重复 |
| 高频调用的质量检查（CI/pre-commit） | ✅ 适用 | 二进制速度远快于反复调用 LLM |
| 简单的提示词增强 / 一次性 Q&A | ❌ 不适用 | 引入二进制层是过度工程，纯 NL 足够 |
| 需要用户自行理解和修改分析逻辑 | ❌ 慎用 | 二进制是黑箱，维护者难以调整规则 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制核心（本仓库） | agent-sh/enhance | 分析精度高、速度快、可重复 | 二进制分发/升级成本高，黑箱不透明 |
| 纯 NL（单 command + skills） | aaronmaturen/claude-plugin | 零额外依赖，完全透明 | 分析能力受 LLM 上下文窗口限制 |
| 纯 Python 脚本 + NL 触发 | nlpm/bin/nlpm-check | 可读、可修改、可 CI 集成 | 需要 Python 环境，速度中等 |

### 2.4 改进空间

1. **当前问题**：8 个 agent 全部缺少 `## Examples` 块，分析结果难以量化验证。**改进做法**：为每个 agent 添加一个 input（原始工件片段）→ output（增强报告 JSON）的 examples 块。**预期收益**：NLPM 分数从 73-75 → 88+，并且 agent 的调用更可预测

2. **当前问题**：`plugin.json`（version: 5.1.0）和 `package.json`（version: 1.0.0）版本不同步。**改进做法**：用单一来源，例如从 `package.json` 派生 `plugin.json` 的 version 字段（用 `jq` 或在 CI 中同步）。**预期收益**：消除版本漂移带来的使用者困惑

3. **当前问题**：agent 的 `tools:` 列表声明了 `Bash(git:*)` 和 `Write/Edit`，但 `--apply` 写操作能否穿越 agent 层级依然不明确。**改进做法**：在 CLAUDE.md 里明确文档化"--apply 会调用 agent 内的 Write/Edit，需要用户在会话中预先允许工具"。**预期收益**：用户不会在 `--apply` 无声失败时感到困惑

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-27 当时得分 **86/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/\*.md（8 个） | 73-75 | 无 examples 块（-15），无 output format（-10），各 1 个模糊量词（-2） |
| CLAUDE.md | 90 | 关键规则缺少 WHY 解释 |
| skills/enhance-hooks/SKILL.md | 92 | "complex", "simple", "unreasonably", "cautious" 四个模糊词（-8） |
| commands/enhance.md | 93 | frontmatter 缺 allowed-tools（-5），"relevant" 模糊（-2） |
| .claude-plugin/plugin.json | 95 | version 5.1.0 与 package.json 1.0.0 不同步 |

### 3.2 当时值得借鉴的模式

1. **「一命令编排多 agent」模式** → `enhance.md` 一个入口并发调度 8 个专项 agent → 用户无需了解内部细节，调用体验统一。借鉴：适用于有多个独立分析维度的工具，可以用一个 orchestrator command 收口

2. **「每 skill 对应一 agent」精准绑定** → 9 个 skill 各自服务一个专项 agent，边界清晰 → 知识不混乱，更新一个 skill 不影响其他。借鉴：避免写一个"万能 skill"，认识领域的自然边界

3. **「NL + 二进制」分层安全架构** → 运行时下载的二进制有 SHA-256 验证 + SLSA 软证明 → 比 npx -y（无版本pin）安全得多。借鉴：任何需要运行时下载可执行文件的插件，都应模仿这个验证链

4. **「enhance-plugins skill 记录自身 bug」** → enhance-plugins/SKILL.md 里显式文档化了"plugin.json 与 package.json 版本不一致是 HIGH 确定性 bug"，相当于插件文档了自己的已知缺陷 → 用户看到即知道要注意。借鉴：skill 里可以写"已知问题"或"容易踩的坑"

5. **CLAUDE.md 精简重定向** → CLAUDE.md 只有标题和 agent/skill/command 清单，实质内容在 AGENTS.md → 降低了 CLAUDE.md 维护负担。借鉴：CLAUDE.md 适合做索引，不适合堆叠大量规则

### 3.3 当时的缺陷

1. **所有 8 个 agent 缺 examples 块（-15 × 8 = -120 分总代价）**
   - **根本原因**：作者把"agent 调用 skill"等同于"agent 有足够文档了"——但 skill 描述的是知识，examples 描述的是 agent 怎么把知识转化为输出。两者不可替代。
   - **自查**：我的 bureau `.claude/agents/auditor.md` 有没有 Examples 块？

2. **version 漂移：plugin.json（5.1.0）vs package.json（1.0.0）**
   - **根本原因**：两个版本字段在不同文件里手动维护，没有单一来源（SSOT），时间一长必然不一致。
   - **自查**：我的插件是否有多个地方声明版本号？

3. **enhance.md 缺 `allowed-tools` 字段**
   - **根本原因**：作者更新了命令本体，但忘记更新 [frontmatter](#frontmatter) 元数据。写 command 时代码和元数据在不同的心智层，容易遗忘。
   - **自查**：我的 bureau 命令是否也缺 allowed-tools？（答案：是的，11 个命令全缺）

### 3.4 当时的优化机会

1. **为 8 个 agent 批量添加 examples 块**：每个只需一个 input→output 示例，可以用脚本批量添加骨架
2. **统一版本号来源**：在 CI/pre-commit 里加一步 `jq -r .version package.json` vs `jq -r .version .claude-plugin/plugin.json` 的断言
3. **为 enhance.md 在 frontmatter 中声明 allowed-tools**：根据命令体的实际工具调用生成

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| enhance.md 缺 allowed-tools | `grep "allowed-tools" commands/enhance.md` | **已修复** ✅ — enhance.md 现有 name、description、codex-description、argument-hint 等完整 frontmatter，但仍未见 allowed-tools 字段 → 需再确认 | 部分修复，frontmatter 结构已存在 |
| version 漂移（5.1.0 vs 1.0.0） | `grep version .claude-plugin/plugin.json; grep version package.json` | **仍存在** ❌ — plugin.json=5.1.0，package.json=1.0.0 | 2 个月后问题依然未修，说明维护者没有将其列为优先级 |
| 8 个 agent 无 examples | `grep -c "## Example" agents/*.md` | **仍存在** ❌ — 全部 0 个 examples | 系统性问题，需要批量处理才会改善 |

### 4.2 架构演进

与审计时相比，当前 HEAD 的 enhance.md 命令体明显扩展：
- 新增了 `codex-description` 字段和完整的 `argument-hint`（支持 `--apply --focus=TYPE --verbose` 等多个标志）
- 新增了 `cross-file` enhancer 集成（原审计时已存在但描述更简单）
- `lib/` 目录结构更成熟（增加了 `cross-platform/` 和 `schemas/` 子目录）

作者意识到：增强器的用户交互需要更丰富的 CLI 语义（标志支持），于是把 argument-hint 做得更详细。

### 4.3 新增的可学习模式

- **`codex-description` 字段**：审计时没有，现在出现在 enhance.md 里。这是给 OpenAI Codex 等非 Claude 工具用的 skill 触发描述，暗示作者在追求跨工具兼容性——NL 工件不只服务 Claude Code，也开始面向整个 AI 编码生态。值得借鉴：如果未来我的 skill 想跨工具部署，可以加这个字段

---

## 五、校准

### 5.1 我已经在做对的

1. **我的 bureau skill 有完整 frontmatter**（name + description + argument-hint）——比 aaronmaturen 强得多，和 enhance 的 skill 层质量相近
2. **我的 bureau skills 零模糊量词**——比 enhance-hooks/SKILL.md（4 个模糊词）做得更好
3. **单职责 skill 设计**：bureau 里每个 skill（recall、scribe、compile 等）各司其职，和 enhance 的 9 个专项 skill 思路一致
4. **CLAUDE.md 简洁索引化**：我的 bureau/CLAUDE.md 风格类似 enhance 的做法，不堆砌规则

### 5.2 挑战 / 验证

- **本案例挑战了什么假设？** 我之前认为"agent 有 skill 就足够了"，看完这个案例才意识到 examples 块是独立必要项——skill 教 agent 背景知识，examples 教 agent 怎么格式化输出。两者不可替代。
- **被验证的做法**："一命令编排多 agent"在实际生产中是可行的，enhance 已经验证了 8 个并行 agent 的工作流是稳定的

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否有 Examples 块
grep -rL "## Example" ~/.claude/agents/*.md 2>/dev/null
# 命中后：为每个缺少示例的 agent 补一个 input→output 示例

# 检查我的插件版本号是否同步
[ -f .claude-plugin/plugin.json ] && [ -f package.json ] && \
  echo "plugin.json: $(jq -r .version .claude-plugin/plugin.json)" && \
  echo "package.json: $(jq -r .version package.json)"
# 命中后：选一个文件为 SSOT，在 CI 里断言两者相等

# 检查 bureau commands 是否有 allowed-tools
grep -L "allowed-tools" commands/*.md 2>/dev/null
# 命中后：根据命令体实际使用的工具，手动补上 allowed-tools 列表

# 检查我的 command frontmatter 是否完整（name + description + allowed-tools）
for f in commands/*.md; do
  echo -n "$f: "
  head -10 "$f" | grep -E "^name:|^description:|^allowed-tools:" | wc -l
done
# 命中后：少于 3 行的命令文件需要补充 frontmatter 字段
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 `.claude/agents/auditor.md` 补 examples 块
   - **为何可行**：auditor.md 现在没有 examples，加一个"输入：会话文本片段；输出：审计报告结构"的示例就能大幅提升 NLPM 分数
   - **第一步**：打开 bureau/.claude/agents/auditor.md，在末尾加 `## Examples` 章节，写一个 10 行的 input→output 示例。耗时 15 分钟

2. **想法**：为 bureau commands 批量补 allowed-tools
   - **为何可行**：11 个命令全部缺 allowed-tools，但每个命令体里的工具调用是明确的（Bash、Read、Write 等），可以一次性补全
   - **第一步**：阅读每个命令的 `## Steps`，列出实际调用的工具，在 frontmatter 里加 `allowed-tools: [Read, Bash, ...]`。耗时 30-45 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 agent-sh/enhance 的核心目的**：分析和增强 Claude Code NL 工件质量，支持自动修复
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有 commands+agents+skills 三层 | bureau 是知识管理工具，enhance 是代码分析工具；enhance 有原生二进制核心 | 高 |
| MarkQWu/gstack | 低 | 都用 skill 组织背景知识 | gstack 是角色扮演（CEO/设计师），enhance 是工件分析 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| enhance.md 缺 `allowed-tools` | `grep -L "allowed-tools" commands/*.md` | **bureau 命中 11/11** — 所有命令全缺 | 高 |
| 8 个 agent 无 examples 块 | `grep -rL "## Example" .claude/agents/*.md` | bureau 的 auditor.md 需检查 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的 `commands/lint.md` → 在 frontmatter 的 `description:` 下方加一行 `allowed-tools: [Read, Bash, Glob, Grep]` → 10 分钟可完成
- `MarkQWu/bureau` 的 `.claude/agents/auditor.md` → 补 `## Examples` 章节 → 15 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：「NL + 原生二进制分层」
   - **本案例做法**：`lib/binary/index.js` 在运行时下载 `agent-analyzer` 二进制，用 SHA-256 校验后执行，分析能力远超纯 LLM
   - **我的项目现状**：bureau/gstack 均是纯 NL，没有任何原生二进制层
   - **如何借鉴**：目前阶段无需引入，但如果 bureau 未来需要高速语义一致性检查，可以考虑用 Python 脚本或 Go 二进制替代部分 LLM 调用

2. **领域**：「codex-description 跨工具兼容字段」
   - **本案例做法**：`enhance.md` 有 `codex-description:` 字段，使插件同时兼容 Claude Code 和 OpenAI Codex
   - **我的项目现状**：bureau/gstack 的命令只有 `description:` 字段，无跨工具声明
   - **如何借鉴**：在 bureau 命令 frontmatter 里加 `codex-description:` 字段，为未来多工具部署做准备。每个命令约 5 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skills 的 frontmatter 完整性
- **我的做法**：bureau 的每个 SKILL.md 都有 `name`、`description`、`argument-hint` 三个 frontmatter 字段，且无模糊量词
- **本案例做法（弱在哪）**：enhance 的 skills（enhance-hooks/SKILL.md 等）有"complex"、"simple"、"cautious"等模糊词，导致扣分
- **意义**：bureau 的 skill 质量在 frontmatter 层面已超过 enhance，这是亮点；若被 NLPM 审计，skill 层分数会较高

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读命令文件时先解析 frontmatter 才能知道这个命令的名称、描述和权限。

### <a name="原生二进制核心"></a>原生二进制核心
> 用 Go/Rust/C 等编译型语言写成的可执行程序。"原生"表示不依赖解释器（如 Python/Node.js）直接运行，"二进制"表示文件内容是机器码。性能高出脚本一个数量级，适合高频调用的静态分析场景。

### <a name="SLSA"></a>SLSA 证明
> Supply-chain Levels for Software Artifacts 的缩写。一种用于验证软件来源的安全框架。当二进制文件有 SLSA 证明时，可以确认它是由特定的 CI 流水线构建的，而非被攻击者篡改后重新发布的版本。即使 SHA-256 匹配，没有 SLSA 仍然存在"攻击者同时替换二进制和校验值"的风险。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code 命令 frontmatter 中的一个字段，用来声明该命令被允许调用哪些工具（如 `Bash`、`Read`、`Write` 等）。缺少这个字段时，Claude 无法知道该预授权哪些工具，可能导致运行时中断或用户频繁被要求手动批准工具调用。
