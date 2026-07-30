# agent-sh/enhance — 学习案例

**仓库**：https://github.com/agent-sh/enhance
**Stars**：3 | **来源**：upstream audit
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-07-30（基于当前 HEAD）
**主题标签**：`single-purpose`, `manifest-discipline`, `examples-driven`, `nl-binary-hybrid`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`agent-sh/enhance` 是一个「NL 表皮 + [原生二进制核心](#原生二进制核心)」架构的 Claude Code 插件，其核心功能是分析和改进 Claude Code 插件本身的质量。插件通过一个主命令 `/enhance` 并行运行 8 个专用 enhancer agent，覆盖 prompt、agent、plugin、docs、hooks、skills、cross-file 和 CLAUDE.md 八个维度。底层由 `lib/binary/index.js` 在首次运行时从 GitHub release 下载原生 `agent-analyzer` 二进制文件（附 SHA-256 校验和可选 SLSA 认证）。

关键事实：
- 20 个 NL 工件（8 agent + 9 skill + 1 command + 1 manifest + 1 CLAUDE.md）
- 审计时 86/100，SECURITY CLEAR（无 Critical/High 安全问题）
- 架构特色：单一入口命令 + 8 并行 enhancer agent，自动分析任意 Claude Code 插件
- 版本问题：`plugin.json` 声明 5.1.0，`package.json` 声明 1.0.0（至今未修复）
- 自我参照：插件包含 `skills/enhance-plugins/SKILL.md`，其中明确记录了这个版本不一致是「HIGH certainty structural bug」

### 1.2 架构剖析
```
enhance/
├── .claude-plugin/
│   └── plugin.json          ← manifest（版本 5.1.0）
├── commands/
│   └── enhance.md           ← 唯一入口命令（现已有 frontmatter）
├── agents/                  ← 8 个专用 enhancer agent
│   ├── agent-enhancer.md
│   ├── claudemd-enhancer.md
│   ├── cross-file-enhancer.md
│   ├── docs-enhancer.md
│   ├── hooks-enhancer.md
│   ├── plugin-enhancer.md
│   ├── prompt-enhancer.md
│   └── skills-enhancer.md
├── skills/                  ← 9 个专用 skill（每 agent 对应一 skill）
│   ├── enhance-agent-prompts/SKILL.md
│   ├── enhance-claude-memory/SKILL.md
│   ├── enhance-cross-file/SKILL.md
│   ├── enhance-docs/SKILL.md
│   ├── enhance-hooks/SKILL.md
│   ├── enhance-orchestrator/SKILL.md
│   ├── enhance-plugins/SKILL.md
│   ├── enhance-prompts/SKILL.md
│   └── enhance-skills/SKILL.md
├── lib/
│   └── binary/index.js      ← 二进制下载器 + SLSA 验证逻辑
└── package.json             ← 版本 1.0.0（与 plugin.json 不一致）
```

- **文件类型分布**：1 command / 8 agents / 9 skills / 1 manifest / 92+ JS 文件（lib/）
- **编排关系**：`enhance.md` 命令触发 8 个 agent 并行执行，每个 agent 加载对应 skill；`enhance-orchestrator/SKILL.md` 描述了 8 路并行的协调逻辑；存在 `--fix` 参数触发自动应用模式
- **跨件契约**：每个 enhancer agent 与对应 skill 形成 1:1 绑定；agent 通过 `tools: [Skill]` 加载 skill，但未声明 `Write/Edit`（这导致 `--fix` 自动应用模式在 agent 层可能无法生效）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「AI 工具自我改进」——用 Claude Code 插件来改进 Claude Code 插件，具有明显的自参照性；每个维度（docs/hooks/agents/skills/plugins）有独立专家 agent，避免单 agent 既分析又改进的职责混乱
- **解决什么问题**：插件开发者在扩展功能时引入质量退化（vague quantifiers 积累、frontmatter 遗漏、版本不一致等），`enhance` 提供自动化质量看门人
- **Trade-off**：「1 command + 8 parallel agents」减少了用户学习成本（只需记一条命令），但 8 个 agent 各自独立，跨 agent 的发现无法自然汇聚；同时，agent 未声明 Write/Edit 工具，使得 `--fix` 模式存在工具权限缺口
- **认知模型**：作者将 skill 视为 agent 的「专业知识载体」，agent 本身保持轻量（仅含调度逻辑），知识深度全在 skill 里——这是单职责原则的清晰体现

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单入口 + 并行专家 agent + 知识下沉 skill」**

一个主命令作为唯一用户接口，内部 fan-out 到 N 个专家 agent，每个 agent 绑定一个深度 skill，skill 包含具体知识，agent 只负责调度。

模式特征清单：
- 特征 1：用户界面极简（1 个命令），内部实现复杂（8 并行 agent）
- 特征 2：知识与调度分离——知识在 skill，调度在 agent，命令只负责触发
- 特征 3：agent-skill 一一对应，职责边界清晰，不存在 skill 被多 agent 共享的模糊情况
- 特征 4：原生二进制作为性能敏感部分的实现，NL 层只处理用户交互和分析逻辑

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要对同一对象做多维度分析 | ✅ 高度适用 | 并行分析，每维度独立专家，互不干扰 |
| 单一直接任务 | ❌ 过度设计 | 8 agent 并行对简单任务是过度开销 |
| 需要性能密集计算 | ✅ 适用 | 二进制核心可处理大量文件扫描 |
| 工具链自动化（CI/pre-commit） | ✅ 适用 | 有 `bin/nlpm-check` 思路，可以独立运行 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单入口并行专家（本仓库） | agent-sh/enhance | 用户接口简洁，分析全面 | 工具权限缺口，跨 agent 发现难聚合 |
| 多命令任务矩阵 | aaronmaturen/claude-plugin | 用户自主选择分析维度 | 需要记住多个命令名 |
| 单 agent 全能分析 | 简单分析类插件 | 实现简单 | 容易出现职责混乱、输出冗长 |

### 2.4 改进空间
1. **当前问题**：8 个 enhancer agent 均缺 `Write/Edit` 工具声明，`--fix` 模式可能静默失败 **改进做法**：在每个 enhancer agent 的 `tools:` 中显式声明 `Write` 和 `Edit`，并在 body 中加「何时允许修改」的约束文档 **预期收益**：--fix 模式行为可预期，不再依赖 skill 能否绕过 agent tool 限制
2. **当前问题**：`plugin.json` 5.1.0 与 `package.json` 1.0.0 长期不一致 **改进做法**：在 CI 加 `jq -r '.version' plugin.json | diff - <(jq -r '.version' package.json)` 校验 **预期收益**：版本不一致立刻 CI 失败，不会进入 release
3. **当前问题**：8 个 agent 无 examples 块，行为不可测试 **改进做法**：为每个 agent 添加 1-2 个 input/output examples **预期收益**：NLPM tester 可以运行，也给用户提供了期待输出的参考

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-27 得分 **86/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/claudemd-enhancer.md | 73 | 无 examples（-15）；无输出格式（-10）；描述模糊 |
| agents/cross-file-enhancer.md | 73 | 无 examples（-15）；无输出格式（-10）；"complex" 模糊 |
| agents/hooks-enhancer.md | 73 | 无 examples（-15）；无输出格式（-10）；"cautious" 模糊 |
| commands/enhance.md | 93 | 缺 allowed-tools（-5）；"relevant" 模糊 |
| .claude-plugin/plugin.json | 95 | 版本 5.1.0 与 package.json 1.0.0 不同步 |
| skills/enhance-prompts/SKILL.md | 98 | "if needed" 模糊（-2） |

### 3.2 当时值得借鉴的模式
1. **1:1 agent-skill 绑定** → 每个 enhancer agent 对应一个同名 skill，职责边界极清晰；skill 文档化了所有知识，agent 只做调度。借鉴：我的项目如果要做多维分析，可以用这个模式避免「知识和逻辑混在一个 agent 里」
2. **`enhance-plugins/SKILL.md` 的自我文档化** → 这个 skill 不只描述如何分析插件，还明确列出了「HIGH certainty structural bugs」包括版本不一致——相当于把 linting 规则写进 skill 本身。这是一种「可执行的质量标准」思路
3. **SLSA + SHA-256 双重验证** → 原生二进制下载时先做 SHA-256 校验，再做可选 SLSA 认证；`AGENT_ANALYZER_REQUIRE_ATTESTATION=1` 可以强制要求 SLSA——供应链安全设计比大多数工具严格

### 3.3 当时的缺陷
1. **8 个 enhancer agent 均无 examples** → 根本原因：作者把所有知识放在 skill 里，认为 skill 的 examples 就够了；但 agent 层的 examples 是为了说明「这个 agent 在面对 X 类输入时应该输出什么格式」，与 skill 层的知识 examples 不同。没有 agent examples，AI 在边缘情况下输出格式不稳定。**自查：bureau 的 NL 工件目前无 agent 文件，暂不受影响。**
2. **agent 声明 `--fix` apply 行为但无 Write/Edit 工具** → 根本原因：作者可能假设 skill 调用可以绕过 agent tool 限制，但 Claude Code 的工具权限机制未文档化此行为。结果：用户使用 `--fix` 时，所有自动修复静默失败。**自查：我的 bureau 命令如果需要 `--apply` 类功能，必须在 frontmatter 中声明 Write/Edit。**
3. **CLAUDE.md 的 8 条关键规则无 WHY 解释** → 根本原因：作者记得规则本身，忘记了规则的理由。无 WHY 的规则对新用户、未来的自己、或 AI 执行都是黑盒——AI 遇到边缘情况时无法判断「在这种情况下，这条规则的底层原则是什么」。

### 3.4 当时的优化机会
1. 为所有 8 个 enhancer agent 批量添加 examples 块（最高杠杆：平均分从 73 提升到 88）
2. 修复 `commands/enhance.md` 的 `allowed-tools`（已在 PR 中提交）
3. 同步 plugin.json 和 package.json 的版本号（高确定性 bug，enhance-plugins skill 自身标记为 HIGH certainty）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `commands/enhance.md` 缺 `allowed-tools` | `head -10 commands/enhance.md` | **已修复**：当前 frontmatter 中有完整 `allowed-tools`、`argument-hint` 等字段 | NLPM PR 被接受；这是 bug #1 的修复 |
| plugin.json 5.1.0 vs package.json 1.0.0 | `jq '.version' plugin.json package.json` | **未修复**：plugin.json 仍为 5.1.0，package.json 仍为 1.0.0 | 3 个月后此 bug 依然存在，说明维护者认为不影响功能或不了解如何修复 |
| 8 agent 无 examples | `grep -l "example" agents/*.md` | **未修复**：agents 目录无任何 examples 块 | 系统性问题，需要批量修复但维护者未处理 |

### 4.2 架构演进
从 audit 时到现在，架构无重组。`commands/enhance.md` 添加了 frontmatter（包括 `codex-description` 字段——新增的 OpenCode 适配），说明作者在扩展多 IDE 支持。`agent-knowledge/` 新目录出现，包含持久化的 enhancement 知识，这是「经验积累」设计模式的体现。

### 4.3 新增的可学习模式
`agent-knowledge/` 目录是 audit 未覆盖的新模式：enhance 命令运行后，学到的模式被持久化到这个目录，下次运行时加载。这是一种跨会话学习机制（[经验积累](#经验积累)设计模式），能让工具随使用次数增加而变得更精准。

---

## 五、校准

### 5.1 我已经在做对的
1. **bureau commands 现在有 frontmatter**（有 description 字段），避免了 enhance 命令 audit 时零 frontmatter 的问题
2. **bureau skills 有完整示例** → 与 enhance 的 skill 质量相当（98/100 级别）
3. **没有工具权限缺口**：bureau 命令不声明 `--fix` 类自动修改行为，避免了「声明了但无法执行」的问题

### 5.2 挑战 / 验证
本案例挑战了我对「agent 和 skill 是同等地位的两层文档」的假设。实际上，**agent 的 examples 和 skill 的 examples 解决不同问题**：skill examples 说明「知识如何应用」，agent examples 说明「agent 在这个上下文中的行为模式」。两者都需要，不能互相替代。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 plugin.json 是否与 package.json 版本一致
for dir in /tmp/my-repos/*/; do
  plugin_v=$(jq -r '.version // "N/A"' "$dir/.claude-plugin/plugin.json" 2>/dev/null)
  pkg_v=$(jq -r '.version // "N/A"' "$dir/package.json" 2>/dev/null)
  if [ "$plugin_v" != "$pkg_v" ] && [ "$plugin_v" != "N/A" ] && [ "$pkg_v" != "N/A" ]; then
    echo "MISMATCH: $dir plugin=$plugin_v pkg=$pkg_v"
  fi
done
# 命中后：对齐两个版本号，推荐 plugin.json 跟 package.json 用同一个值

# 检查我的命令是否声明了某种「apply」行为但无 Write/Edit
grep -rn "\-\-apply\|--fix" /tmp/my-repos/*/commands/*.md 2>/dev/null
# 命中后：确认 frontmatter 中声明了 Write 或 Edit
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 添加 `agent-knowledge/` 式跨会话学习目录
   - **为何可行**：bureau 本身就是知识管理工具，一个专门存放「bureau 自己学到的改进模式」的目录高度自洽
   - **第一步**：在 bureau 仓库中创建 `canon/bureau-learnings/` 目录，并在 `commands/lint.md` 结尾加一步「将发现的模式追加到此目录」，20 分钟

2. **想法**：为 gstack/bureau 的 commands 添加 WHY 解释到 CLAUDE.md
   - **为何可行**：enhance-CLAUDE.md 的教训是明确的——规则不解释 WHY 会导致 AI 在边缘情况下误判
   - **第一步**：读取 bureau CLAUDE.md 中每条关键规则，为每条添加一行 WHY 注释，15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例核心目的**：自动分析和改进 Claude Code 插件质量的元工具

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同为 Claude Code 插件，有 skill + command 结构 | bureau 是知识管理，enhance 是质量分析；两者目的不同 | 高（架构模式可借鉴） |
| MarkQWu/gstack | 中 | 同为插件，多 skill 结构 | gstack 聚焦开发工具，无自参照质量分析 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| version-mismatch（plugin.json vs package.json） | `diff <(jq -r .version .claude-plugin/plugin.json) <(jq -r .version package.json)` | **bureau**：plugin.json 0.5.2，无 package.json→**不适用**；**gstack**：无 plugin.json | 不适用 |
| CLAUDE.md 关键规则无 WHY | `cat CLAUDE.md \| grep -A1 "^-\s"` | **需要检查 bureau 和 gstack 的 CLAUDE.md** | 中 |
| agent 无 examples（若有 agent） | `grep -L "example" agents/*.md` | **bureau/gstack 无 agent 文件**，暂不适用 | 暂不适用 |

### 7.3 别人的更优方案

1. **领域**：`argument-hint` 字段（命令参数提示）
   - **本案例做法**：`enhance.md` frontmatter 中有 `argument-hint: "[target-path] [--apply] [--focus=TYPE] ..."` 完整参数文档
   - **我的项目现状**：`MarkQWu-bureau/commands/lint.md` 有 `argument-hint` 字段，这点我做对了
   - **结论**：本维度我与 enhance 同等水平，均使用了 argument-hint

2. **领域**：单一入口命令降低认知负荷
   - **本案例做法**：只有 1 个 `enhance` 命令，内部 8 个 agent 并行——用户不需要知道内部复杂性
   - **我的项目现状**：bureau 有 10 个命令，用户需要记住 capture/compile/review/query 等各个步骤
   - **如何借鉴**：考虑添加一个 `bureau:run` 全流程命令，内部按顺序执行 capture→compile→review，减少用户操作步骤

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 示例覆盖率
- **我的做法**：`MarkQWu-bureau/skills/*/SKILL.md` 均有 3 个 examples 块（通过 NLPM 测试）
- **本案例做法**（弱在哪）：8 个 agent 零 examples（audit 后 3 个月仍未修复）
- **意义**：我的 NL 工件在可测试性上明显优于 enhance；如果被 NLPM 审计，这是加分项

---

## 八、术语表

### <a name="原生二进制核心"></a>原生二进制核心
> 用 C / Go / Rust 这种编译型语言写出来的可执行文件。`agent-sh/enhance` 的底层分析引擎 `agent-analyzer` 是一个需要从 GitHub release 下载的原生二进制——不依赖 Node.js 解释器，直接在 CPU 上运行，扫描速度比纯 JS 快一个数量级。「NL 表皮 + 原生二进制核心」是这类插件的常见混合架构：用户看到的是 Markdown 命令，背后跑的是高性能二进制。

### <a name="经验积累"></a>经验积累
> 工具在每次运行后将学到的新模式写入磁盘（如 `agent-knowledge/` 目录），下次运行时读取并应用这些积累——类似人工经验的跨会话传递。这种设计让工具随使用次数增加而变得更精准，但也需要管理「旧经验失效」的问题（环境变化后旧知识可能误导新判断）。

### <a name="SLSA认证"></a>SLSA 认证
> Supply-chain Levels for Software Artifacts，软件制品供应链安全等级规范。GitHub Actions 可以为构建产物生成 SLSA provenance（来源证明），消费者可以用 `gh attestation verify` 验证二进制确实是从特定 commit 的 CI 流水线构建而来，而不是攻击者替换的版本。SHA-256 只验证内容完整性，SLSA 还验证构建来源，是更强的安全保证。
