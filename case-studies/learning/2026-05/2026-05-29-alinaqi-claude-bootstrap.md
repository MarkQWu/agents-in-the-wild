# alinaqi/claude-bootstrap — 学习案例

**仓库**：https://github.com/alinaqi/claude-bootstrap
**Stars**：576 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-29（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `vague-quantifier`, `experience-accumulation`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-bootstrap 是一个 Claude Code 项目模板和多智能体团队框架——相当于给 Claude Code 用户准备的"企业级工程师团队"开箱即用套件。作者 Ali Naqi 把它定位为"autonomous engineering scaffold"：clone 下来就能拥有一个有 TDD 工作流、合约管理、记忆系统、代码图谱的完整 AI 工程团队。

关键事实：
1. **82 个 NL 工件**（61 skills + 15 commands + 6 agents），是本批次中体量最大的仓库
2. **内嵌两套 Python 工具**：iCPG（代码图谱分析）和 Mnemos（记忆管理），作为 skill 的底层引擎
3. **安全状态：BLOCKED**——Audit 时因为 [curl-download-and-execute](#curl-download-and-execute) 模式被安全封锁，截至本次检查 initialize-project.md 中的二进制下载代码已不可见（可能已移除或重构）
4. **含 hooks**：pre-push 和 post-commit hooks 在推送前自动触发 code review，是少数把 Claude Code 集成进 git workflow 的仓库

### 1.2 架构剖析

```
claude-bootstrap/
├── CLAUDE.md                    # 全局偏好设置（TDD 政策、代码风格等）
├── commands/                    # 15 个斜杠命令
│   ├── spawn-team.md            # 核心：召唤多智能体团队
│   ├── initialize-project.md    # 项目初始化（含历史安全问题）
│   ├── icpg-*.md (5个)          # 代码图谱分析系列
│   ├── maggy*.md (2个)          # 工作空间管理
│   └── mnemos-*.md (2个)        # 记忆检查点
├── agents/                      # 6 个专业子智能体
│   ├── merger-agent.md          # 合并专家
│   ├── team-lead.md             # 团队领导
│   ├── feature-agent.md         # 功能实现
│   ├── code-review.md           # 代码审查
│   ├── security-agent.md        # 安全审计
│   └── quality-agent.md         # 质量保障
├── skills/                      # 61 个技能文件
│   ├── base/, security/         # 基础和安全
│   ├── agentic-development/     # Agent 开发模式
│   └── [50+ 领域 skills]        # 全栈覆盖
├── hooks/
│   ├── pre-push                 # 推送前代码审查
│   └── workspace/               # 工作空间钩子
└── scripts/
    ├── install-graph-tools.sh   # 安装脚本（历史安全问题所在）
    └── icpg/, mnemos/           # Python 工具脚本
```

- **文件类型分布**：61 个 SKILL.md / 15 个 command / 6 个 agent / 2 套 hooks / 大量 Python 脚本
- **编排关系**：`spawn-team` command → 6 个 agent 并行 → 各 agent 按职责调用对应 skill。是典型的"指挥官 + 专家团队"模式
- **跨件契约**：agents 直接执行 skills（通过 model 调用），hooks 直接调用 `claude --print "/code-review"` 命令，是强耦合的 git-workflow 集成

### 1.3 设计思路 / 方法论

- **核心设计哲学**："TDD 非谈判性"（在 CLAUDE.md 里显式标注 Non-Negotiable）——先写测试再实现，是整个 bootstrap 的工程文化基础
- **解决什么问题**：从零配置 Claude Code 项目太耗时；缺乏团队协作规范；记忆、代码图谱等工具的安装和集成成本高
- **Trade-off**：大而全（82 个工件）带来"即插即用"的体验，但 commands 全无 [frontmatter](#frontmatter) 导致它们无法真正被 Claude Code 注册——这是一个典型的"说一套做一套"矛盾：声称开箱即用，实际上核心功能无法加载
- **认知模型**：作者把 Claude Code 视为可以执行完整 SDLC（软件开发生命周期）的平台，不只是代码补全工具

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「TDD-First 多智能体 Bootstrap 模板」**：以工程规范（TDD、合约验证、代码图谱）为核心，用多智能体并行执行 SDLC 各阶段。特点是"规范即代码"——CLAUDE.md 把工程文化写死，不依赖团队默契。

模式特征清单：
- **特征 1**：CLAUDE.md 扮演"不可违反的工程政策"角色，而非只是"建议"
- **特征 2**：6 个专业 agent 覆盖整个 SDLC（feature / review / security / quality / merge / lead）
- **特征 3**：spawn-team 是"一键启动整个团队"的入口命令，而不是逐个调用 agent
- **特征 4**：hooks 自动触发 code review，把 AI 评审嵌入 git workflow
- **特征 5**：iCPG + Mnemos 提供持久化能力（代码图谱、记忆检查点），突破单会话限制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 全栈项目（后端+前端+测试全覆盖）| ✅ 高度适用 | 61 个领域 skills 几乎涵盖所有栈 |
| 团队协作 / onboarding | ✅ 适用 | CLAUDE.md 里的工程规范可作为团队契约 |
| 小型个人项目 | ❌ 过度工程 | 82 个工件对单人项目是负担 |
| 需要高度定制的特殊领域 | ⚠️ 部分适用 | 需要删减不相关的 61 个 skills |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| TDD-First Bootstrap 模板（本案例）| claude-bootstrap | 开箱即用、工程规范强制 | 体积大、commands 注册问题 |
| 单 skill 轻量包 | alexgreensh/token-optimizer | 轻量、专注 | 需要用户自己搭框架 |
| 超级框架 | SuperClaude_Framework | 功能最全 | 学习曲线更陡 |

### 2.4 改进空间

1. **当前问题**：15 个 commands 全无 frontmatter，无法被 Claude Code 注册 **改进做法**：为每个 command 加最小 frontmatter（name + description + allowed-tools 三行）**预期收益**：让用户真正能用 `/spawn-team` 而不是复制粘贴 markdown 内容

2. **当前问题**：安全 BLOCKED 状态拦截了贡献（虽然用户个人仍可使用）**改进做法**：为二进制下载加 SHA256 校验（一次性工作，约 30 分钟）**预期收益**：解除 BLOCKED，项目可以接受外部贡献

3. **当前问题**：iCPG / Mnemos 的 Python 工具没有顶层 `requirements.txt`，用户无法做依赖审计 **改进做法**：在 repo 根添加 `requirements.txt` 汇总 **预期收益**：支持 `pip-audit` 依赖安全扫描

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-20 当时得分 **80/100**，超过 70 的通过线，但分布极度不均。

| 类型 | 平均分 | 主要问题 |
|---|---|---|
| Agents (6) | 85/100 | 全部缺少 examples 块 (-15/个) |
| Commands (15) | 40/100 | 全部缺少 frontmatter (-55/个) |
| Skills (61) | 89/100 | 偶发模糊量词，整体高质量 |

### 3.2 当时值得借鉴的模式

1. **playwright-testing skill 的具体性** → 有 "dead-link detection requirement"、完整代码示例、逐步 TDD 工作流文档 → `skills/playwright-testing/SKILL.md` → 把这种"具体到可执行"的密度作为写 skill 的标准
2. **agent frontmatter 完整性** → 所有 6 个 agents 都有 `name, description, model, tools/disallowedTools, maxTurns, effort` 完整六字段 → `agents/` 目录下任意文件 → agent 的六字段应当视为最低标准
3. **TDD 作为不可协商的 CLAUDE.md 政策** → 不是"建议"而是"强制"，防止用户偷懒 → `CLAUDE.md` 第 30+ 行 → 把关键工程政策写成 CLAUDE.md 的强制规则
4. **spawn-team 的并行 agent 设计** → 一条命令触发 6 个专业 agent 并行执行 → `commands/spawn-team.md` → 学习用 command 编排多 agent 并行工作流

### 3.3 当时的缺陷

1. **15 个 commands 全无 frontmatter** → 问题根因：作者把 commands 当"文档"而非"可注册组件"来写，忘记 Claude Code 需要 frontmatter 才能识别。这是体量大的仓库最常见的问题——边界模糊导致"我以为加了" → 自查：我的 echo-sleuth 所有 commands 均有 frontmatter（已验证），这一点我做对了

2. **所有 agents 缺 examples** → 6 个 agents 得分 85 而非 100 的原因。问题根因：写 agent 时聚焦于"做什么"，忽略了"长什么样"——examples 让 Claude 理解触发场景 → 自查：echo-sleuth 的 memory-auditor.md 有两个 examples，这是优势

3. **安全 BLOCKED（curl-download-and-execute）** → `scripts/install-graph-tools.sh` 下载二进制后立即 `chmod +x && ./binary`，无校验 → 问题根因：最简单的"能用就行"安装思路，没有威胁建模 → 自查：我的仓库有安装脚本吗？如有，是否有类似 curl-pipe-execute 模式？

### 3.4 当时的优化机会

1. 为 15 个 commands 批量加 frontmatter（最高优先级，影响基本可用性）
2. 为 6 个 agents 各加 1 个 example block（中等优先级，影响触发精度）
3. 为 `mnemos-checkpoint.md` 和 `mnemos-status.md` 加空输入处理（"如无检查点，报告无激活会话"）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| commands 全无 frontmatter | `head -5 commands/spawn-team.md` | **仍存在**（spawn-team.md 直接以 `# /spawn-team` 开头，无 frontmatter） | 核心可用性问题至今未修复 |
| 安全 BLOCKED（二进制下载无校验）| `grep -n "chmod +x" commands/initialize-project.md` | **无法验证**（grep 无命中，可能已重构或移至其他文件）| 需要进一步检查 install 脚本 |
| mnemos-checkpoint 空输入无处理 | `grep -n "no checkpoint\|no.*session" commands/mnemos-checkpoint.md` | **仍存在**（grep 无命中） | 用户在无检查点时仍得不到清晰提示 |

### 4.2 架构演进

当前 HEAD 相比 audit 时多出了 `commands/polyphony-*.md`（3 个多智能体编排命令）、`commands/usage-summary.md`（用量统计）、`commands/build-in-public.md`（公开构建日志）等，说明作者在持续扩展命令集。同时增加了 `cortex-mcp/`（MCP 网关）和大量 `_project_specs/phases/` 文档，反映该项目本身正在用自己的工具来构建下一版本。

### 4.3 新增的可学习模式

**`_project_specs/` 目录作为公开 roadmap**：把产品的分阶段计划直接放在 repo 里，每个 `phase-XX-*.md` 文件是一个里程碑，这样 Claude Code 在执行任务时可以读取 spec 来了解全局背景。对于长期项目来说，这是"context engineering"的一种具体实践。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 有 frontmatter**：echo-sleuth 所有命令已注册
2. **agents 有 examples**：memory-auditor.md 有两个带 Context/user/assistant 的完整示例
3. **无 curl-download-and-execute**：我的安装方式是通过 `claude plugin install`，不需要用户执行任意二进制
4. **空输入处理**：recall.md 中有 "If zero sessions are found, report..." 的明确空状态处理

### 5.2 挑战 / 验证

**挑战了假设**：我一直以为"大仓库 = 高质量"，但 claude-bootstrap 的 80 分证明体量和质量不相关。15 个 commands 全无 frontmatter，说明**广度覆盖可以和注册可用性完全脱钩**。下次评估一个仓库时，先检查 commands 是否有 frontmatter，再看其他东西。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否全部有 frontmatter
for f in /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md; do
  if head -3 "$f" | grep -q "^---"; then
    echo "OK: $f"
  else
    echo "MISSING: $f"
  fi
done
```
命中后怎么办：为缺少 frontmatter 的 command 文件加 `---\nname: ...\ndescription: ...\nallowed-tools: [...]\n---`。

```bash
# 检查安装脚本是否有 curl-download-execute 风险
grep -rn "curl.*chmod\|chmod +x.*tmp\|download.*execute" \
  /tmp/my-repos/MarkQWu-*/ --include="*.sh" 2>/dev/null
```
命中后怎么办：加 SHA256 校验步骤，并把下载来源固定为特定版本 tag（不用 `latest`）。

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 agents 添加 example 块（参考 claude-bootstrap 的 agents 格式，加 `recall` agent 的 examples）
   - **为何可行**：现在 recall.md 有 frontmatter 但 examples 不够丰富，补充后可提升触发精度
   - **第一步**：在 `agents/recall.md` 的 frontmatter description 字段末尾加 `<example>...</example>`，包含一个 "搜索关键词" 场景，约 15 分钟

2. **想法**：借鉴 claude-bootstrap 的 `_project_specs/` 思路，在 echo-sleuth 里加 `specs/` 目录，记录计划中的功能
   - **为何可行**：echo-sleuth 目前只有 WHATSNEW 类文件，缺乏前瞻性计划文档
   - **第一步**：创建 `specs/v0.2-features.md`，列出下一版本的 2-3 个功能目标，5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 alinaqi/claude-bootstrap 的核心目的**：提供完整的 Claude Code 多智能体工程团队模板，含 TDD 政策、代码图谱、记忆管理

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 Python 脚本后端；都关注 session/memory 持久化；都是 Claude Code 增强工具 | claude-bootstrap 是全栈工程框架，echo-sleuth 是知识提取专用工具 | 高 |
| MarkQWu/claude-for-legal | 低 | 同为插件 | 目的和领域完全不同 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同为 skill 集合 | 创作领域 vs 工程工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 frontmatter | `grep -L "allowed-tools" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md` | **未命中**——echo-sleuth 所有 command 均有完整 frontmatter | N/A |
| agents 缺 examples | `grep -L "example" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/agents/*.md` | **部分命中**——analyze.md 和 schema-scout.md 的 examples 较少 | 中 |
| 模糊量词 | `grep -rn "comprehensive\|various\|appropriate" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/**/*.md` | **未命中**——experience-synthesis SKILL.md 无此类词 | N/A |

**命中后的具体行动建议**：
- `agents/analyze.md` 和 `agents/schema-scout.md` → 各增加 1 个 `<example>` 块，描述触发场景（用户询问什么时调用此 agent）→ 约 10 分钟/个

### 8.3 别人的更优方案

1. **领域**：TDD 作为不可协商的 CLAUDE.md 政策
   - **本案例做法**：CLAUDE.md 的 `## TDD — Non-Negotiable` 章节用大写+粗体标注，跟随 1-2-3 强制流程
   - **我的项目现状**：echo-sleuth 的 CLAUDE.md 只有简单的说明，没有明确的工程政策
   - **如何借鉴**：在 echo-sleuth 的 CLAUDE.md 中加 `## 使用原则` 节，明确"先验证再提取"等不可违反的规则

2. **领域**：spawn-team 式的单点入口多 agent 编排
   - **本案例做法**：一个 `spawn-team` command 召唤全部 6 个 agent，用户只需记住一个命令
   - **我的项目现状**：echo-sleuth 的每个命令对应一个独立功能，没有"全体出动"的编排层
   - **如何借鉴**：在 echo-sleuth 中考虑加 `/full-audit` command，一次性触发所有分析 agent（recall + schema-scout + memory-auditor）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：commands 注册可用性
- **我的做法**：MarkQWu/echo-sleuth-for-claude 所有 command 文件均有完整 frontmatter，可以被 Claude Code 直接注册使用
- **本案例做法**：15 个 commands 全无 frontmatter，用户无法通过 `/spawn-team` 等斜杠命令直接调用
- **意义**：在实际可用性上，echo-sleuth 的 8 个命令全部可用，而 claude-bootstrap 的 15 个命令在 Audit 时全部不可用。这是一个值得在社区分享的对比点。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`allowed-tools`）。Claude Code 读 command 文件时先解析 frontmatter 才能把这个命令注册到 `/command-name` 路径。没有 frontmatter，即使文件内容再好，用户也无法通过 `/` 前缀调用它。

### <a name="curl-download-and-execute"></a>curl-download-and-execute
> 一种安装模式：用 `curl` 从互联网下载一个文件，然后立即用 `chmod +x` 赋予执行权限并运行。问题在于没有任何校验——如果下载链接被劫持（DNS 污染、中间人攻击、供应链攻击），用户会在不知情的情况下运行恶意代码。正确做法是下载后先核对 SHA256 哈希值，确认与官方公布的值一致后再执行。

### <a name="SDLC"></a>SDLC
> Software Development Life Cycle（软件开发生命周期）的缩写，包括需求分析、设计、编码、测试、部署、维护等阶段。claude-bootstrap 的多智能体设计试图用不同 agent 覆盖 SDLC 的各个阶段。
