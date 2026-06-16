# jeremylongshore/claude-code-plugins-plus-skills — 学习案例

**仓库**：https://github.com/jeremylongshore/claude-code-plugins-plus-skills
**Stars**：⭐1917（审计时约 2009）| **来源**：xiaolai upstream（已有案例文章）
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-06-16（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `vague-quantifier`, `examples-driven`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-code-plugins-plus-skills` 是 Claude Code 生态里规模最大的公共插件仓库之一，由 [jeremylongshore](https://github.com/jeremylongshore)（intentsolutions.io）维护，并配有独立 marketplace 网站（tonsofskills.com）和 CLI 包管理器（`ccpi`）。审计时收录 **423 个插件、2,849 个 skill、177 个 agent**，3,000+ 总制品。当前 Stars 1917，fork 数 270+。

仓库横跨多个完全独立的子生态：Geepers（14 个 AI 辅助 UX agent）、FairDB（PostgreSQL 运维 runbook）、Freshie（食品库存追踪）、schema-optimization lab（实验性 BigQuery 优化 agent）等。这种规模使得"整体质量"本身就是一个统计问题——任何单次审计都只能抽样，就像读一本 3000 页书的每第十页然后评价整本书。

### 1.2 架构剖析

- **目录结构（当前 HEAD）**：
  ```
  /
  ├── .claude/
  │   └── agents/
  │       └── skill-auditor.md         # 自身的 auditor agent（已修复 frontmatter）
  ├── plugins/
  │   ├── community/
  │   │   └── geepers-agents/
  │   │       └── agents/              # 社区 agents 目录（重新组织后）
  │   │           ├── geepers_orchestrator_games.md
  │   │           ├── geepers_snippets.md
  │   │           ├── geepers_code_checker.md
  │   │           ├── geepers_critic.md
  │   │           ├── geepers_react.md
  │   │           ├── geepers_fullstack_dev.md
  │   │           └── ... （多个新增 agent）
  │   └── ...（423 个插件目录）
  ├── workspace/
  │   └── lab/
  │       └── schema-optimization/
  │           └── agents/
  │               ├── phase_1.md       # 已添加 frontmatter（BigQuery 上下文）
  │               ├── phase_2.md
  │               ├── phase_3.md
  │               ├── phase_4.md
  │               └── phase_5.md
  ├── backups/                          # 时间戳版本快照（审计后已重组/清理）
  ├── scripts/
  │   └── quick-test.sh                # 已修复 npm install 问题
  └── plugin.json / marketplace.json
  ```

- **文件类型分布**：3000+ 制品（agent + command + skill + manifest + 脚本）混合分布
- **编排关系**：Geepers agent 群通过 `geepers_*` 前缀命名空间统一，FairDB 运维 runbook 是线性 step-by-step 流程，schema-optimization 是五阶段顺序 pipeline（phase_1 → phase_5）
- **跨件契约**：FairDB commands 调用真实 PostgreSQL 集群；webhook 集成通过环境变量 URL 参数化

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「规模优先、多生态并行」—— 一个仓库承载完全异质的多个插件生态（游戏 UX、数据库运维、食品库存），每个子生态有自己的设计语言和深度
- **解决什么问题**：成为 Claude Code 用户的一站式插件市场，通过 `ccpi install` 让用户像 npm 一样装插件
- **做了什么 trade-off**：广度 vs. 深度——同时维护 400+ 插件意味着平均质量不可能和专注单一领域的作者相比；部分制品（backup 归档、lab 实验）不可避免地以低于 production 标准的状态存在于同一仓库
- **反映什么认知模型**：作者把这个仓库视为「生态圈」而非「产品」——有精品（Geepers 91-93 分）、有工作草稿（schema-optimization lab 原来 30 分）、有历史归档（backups/），三者并存，一起对外开放

> **与 xiaolai 案例的关系**：xiaolai 已于 2026-04-24 发布了一篇详细的英文审计案例 [《The Validator's Blind Spot: What 73 Points Found — and Fixed — in a 2,009-Star Plugin Marketplace》](../../2026-04-24-jeremylongshore-claude-code-plugins-plus-skills.md)，覆盖了完整的审计-修复-再审计循环。本学习案例基于那次审计，重点从**个人学习视角**提炼设计模式和自查项。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多生态单仓 + 社区目录结构」**：用命名空间前缀（`geepers_*`）和 community 子目录隔离不同来源的 agent，共用同一个 manifest 和 CI 体系。

模式特征清单：
- 特征 1：命名空间前缀（`geepers_*`）—— 14 个相关 agent 统一前缀，在插件列表里一眼辨识、不与其他插件冲突
- 特征 2：社区目录（`plugins/community/`）—— 把社区贡献集中隔离，与核心插件区分，降低管理复杂度
- 特征 3：运维 runbook 风格（FairDB commands）—— 每步有编号、有安全 callout、有回滚说明，适合真实生产环境执行
- 特征 4：实验室路径（`workspace/lab/`）—— 把在研 pipeline 和 production artifacts 物理分离，虽然 CI 标准仍统一应用
- 特征 5：CI 扩展扫描路径（PR #578）—— `find_agent_files()` 覆盖 `.claude/agents/` 和 `workspace/**/agents/`，不仅扫 `plugins/`

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型多领域社区插件市场 | ✅ 高度适用 | 命名空间前缀 + 社区目录使得大量异质插件可以同仓管理 |
| 个人/小团队单一领域插件 | ❌ 过度设计 | 结构复杂性超出需要，用扁平结构即可 |
| 生产运维 runbook | ✅ 适用 | FairDB 风格的 step-by-step + 安全 callout 大幅降低误操作风险 |
| 实验性 lab artifact | ⚠️ 谨慎 | lab 路径下的 artifact 仍被 CI 和 NLPM 扫描，需要明确是否要达到 production 标准 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多生态单仓（本仓库） | jeremylongshore/claude-code-plugins-plus-skills | 一站式、CI 统一、插件互相发现 | 规模大时质量控制难度指数级上升 |
| 单领域深度插件 | google-gemini/gemini-skills（98分） | 质量高度一致，易于维护 | 生态宽度有限，需要用户装多个插件 |
| 社区 awesome 列表 | ComposioHQ/awesome-claude-plugins | 零维护成本、覆盖广 | 质量完全依赖被引用仓库，自身不承载制品 |

### 2.4 改进空间

1. **当前问题**：33 个 backup devops/testing commands 缺少 `allowed-tools` 字段，每个评分仅 72/100。**改进做法**：为每个 command 在 frontmatter 里声明 `allowed-tools`，列出实际使用的 bash 命令、MCP 工具。**预期收益**：AI 执行时权限范围明确，静态验证工具可以检查工具调用合法性。
2. **当前问题**：部分 command 有「ensure robust connection」「comprehensive backup strategy」等模糊量词。**改进做法**：把「comprehensive」改成「covering: [列出具体覆盖的场景]」，把「robust」改成「with retry=3 on connection timeout」。**预期收益**：prompt 可测试，AI 输出稳定性提升。
3. **当前问题**：schema-optimization lab agents 被 NLPM 当成 production artifact 审计，评分拖累全局。**改进做法**：在 lab 目录加 `.nlpm-ignore` 或在 `nlpm.local.md` 里配置扫描排除规则。**预期收益**：整体 NL 分数更准确反映真实 production 质量。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-17 审计 105 个制品（31 agents + 74 commands），整体得分 **73/100**。

```
分布：
90-100 分（14 个，13%）—— 全为 Geepers agents
80-89 分（26 个，25%）—— FairDB + Freshie commands
70-79 分（22 个，21%）—— 标准 devops commands
55-69 分（36 个，34%）—— backup 归档 + 缺 allowed-tools
0-54 分（7 个，7%）  —— 无 YAML frontmatter 或模板未评估
```

最低分制品：

| 制品 | 分数 | 主要惩罚 |
|---|---|---|
| `.claude/agents/skill-auditor.md` | 30/100 | 完全没有 YAML frontmatter（-70）|
| `workspace/lab/schema-optimization/agents/phase_1.md` 到 `phase_5.md` | 30/100 各 | 完全没有 YAML frontmatter（-70）|
| `backups/.../backup-strategy.md` | 20/100 | YAML description 字段含 shell 替换表达式（-50+）|
| `backups/.../sync-agent-context.md` | 40/100 | 完全没有 YAML frontmatter（-50）|

### 3.2 当时值得借鉴的模式

1. **Geepers 命名空间前缀**：14 个 agent 统一用 `geepers_*` 前缀，在插件列表中形成视觉聚类，无冲突。得分 91-93，是仓库里最强的部分 → 可借鉴到 echo-sleuth 的多 agent 命名设计
2. **FairDB 运维 runbook 风格**：编号步骤 + 安全 callout + 回滚说明，得分 85-88 → claude-for-legal 的法律程序 agent 可以借鉴这种「操作手册」格式
3. **manifest 精准声明**：plugin.json 统一声明多个子目录的制品路径，一个 manifest 管理 400+ 插件 → 多域插件仓库的标准做法
4. **CI 体系存在**（PR #578 前）：虽然 CI 有盲区，但仓库有 CI 基础，使得修复可以通过 PR + 验证流程推进

### 3.3 当时的关键缺陷

1. **「验证者的盲点」**：`.claude/agents/skill-auditor.md` 是仓库自己的 auditor agent，却没有 YAML frontmatter，得 30 分，无法被 Claude Code 加载。**根本原因**：CI 只扫 `plugins/` 目录，没有扫描 `.claude/agents/`。**教训**：自动化工具的扫描路径如果不覆盖自身所在目录，就是系统性盲区而非个别失误。
2. **模板未求值就提交**：`backup-strategy.md` 的 YAML `description` 字段里有 `$(echo "$description"|cut -c1-100)` 这样的 shell 替换表达式。**根本原因**：批量生成脚本的输出没有经过验证就提交，模板占位符未被替换。CI 只检查字段是否存在，不检查字段内容的合法性。**教训**：字段存在 ≠ 字段内容正确，CI 验证必须检查内容格式。
3. **`sudo rm -rf` 无保护执行**：`incident-p0-disk-full.md` 包含 `sudo rm -rf /var/lib/postgresql/16/main/pgsql_tmp/*` 且无任何前置验证（如检查挂载点、目录是否存在）。**根本原因**：运维 runbook 直接存储可执行命令，风险高于普通配置。**教训**：runbook 里的破坏性命令必须有前置 guard clause。
4. **security MEDIUM：npm install -g 在 CI 里**：`scripts/quick-test.sh` 在测试运行时执行 `npm install -g pnpm@9.15.9`，是供应链安全风险。**根本原因**：依赖安装和测试运行混在同一步。

### 3.4 当时的安全发现总结

| 级别 | 描述 | 文件 |
|---|---|---|
| HIGH | YAML description 字段含 shell 替换（解析器混淆风险）| backup-strategy.md |
| HIGH | `sudo rm -rf` PostgreSQL 数据目录，无 guard | incident-p0-disk-full.md |
| MEDIUM | `npm install -g pnpm@9.15.9` 在测试运行时（供应链）| scripts/quick-test.sh |
| MEDIUM | `PGPASSWORD` 可能在进程表可见 | fairdb-setup-backup.md |
| MEDIUM | webhook curl 使用环境变量 URL，无前置 `[[ -n ]]` 验证 | fairdb-onboard-customer.md |
| LOW（×2） | 其他低风险配置问题 | 多处 |

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `skill-auditor.md` 无 frontmatter | `grep "^name:" .claude/agents/skill-auditor.md` | **已修复** ✅（`name: skill-auditor`，有完整 description）| PR #535，审计后约 28 小时内完成 |
| 5 个 schema-optimization phase agents 无 frontmatter | `grep "^name:" workspace/lab/schema-optimization/agents/phase_*.md` | **全部已修复** ✅（名称如 `phase-1-schema-analysis`，含 BigQuery 上下文描述）| PR #536，同一修复窗口 |
| `backup-strategy.md` shell 替换在 YAML 里 | 路径已变化 | **文件已重组/找不到原路径**（backups/ 目录在当前 clone 中已重组）| 问题已解决，具体通过文件删除或内容修复 |
| `npm install -g pnpm@9.15.9` 在 quick-test.sh | `grep "npm install" scripts/quick-test.sh` | **已修复** ✅（替换为明确的依赖缺失报错提示）| PR #538 |
| webhook curl 无 guard | 检查 FairDB 相关文件 | **已修复** ✅（加了 `[[ -n "${FAIRDB_MONITORING_WEBHOOK:-}" ]]` 前置检查）| PR #539 |
| Geepers agents 路径 | `ls plugins/community/geepers-agents/agents/` | **已迁移并扩充** ✅（新增 geepers_orchestrator_games、geepers_snippets、geepers_code_checker 等多个 agent）| 社区目录重组，生态继续扩展 |

### 4.2 最重要的演进：CI 硬化（PR #578）

这是这次审计最有价值的长期产出。维护者不只是修了 bug，而是把整个验证基础设施重新画了一遍：

- `find_agent_files()` 现在扫描 `.claude/agents/` 和 `workspace/**/agents/`——关闭了「审计者自身」那类盲区
- 新增 `check_yaml_shell_substitution()` 函数，检测 `$(...)``、backtick、`${VAR}` 在 YAML 字符串值里的出现
- PR 触发器扩展到 `.claude/**` 和 `workspace/**`
- 新增 `secret-scan.yml`：gitleaks（每次 PR）+ trufflehog（每周全历史扫描）
- 新增 `.gitleaks.toml`：在上游默认规则基础上添加 Anthropic、Groq、Firebase/GCP 凭证 pattern

这次演进的逻辑是：**一个 NLPM 发现的具体 bug，触发了维护者对整类问题的系统性修复**。原始修复是「把这个文件加上 frontmatter」，真正的价值是「以后所有 PR 里的同类问题都会被 CI 拦截」。

### 4.3 发布记录

2026-04-23，仓库标记了 `v4.28.0` release。changelog 明确列出「External audit response (NLPM/xiaolai) — validator + CI expansion」作为发布亮点之一。commit 信息里有一句：*"Jeremy made me do it / -claude"*——一个安静的注脚，说明这次修复是维护者的 Claude Code 实例写的，是人机协作的产物。

---

## 五、校准

### 5.1 我已经在做对的

1. **纯只读脚本**：echo-sleuth 的所有脚本都是只读分析（git log、grep 等），没有 `sudo`、没有 `rm -rf` 模式，天然没有这类运维 runbook 的安全风险
2. **单生态聚焦**：echo-sleuth 只做一件事（挖掘 Claude Code 对话记忆），规模小意味着全量审计可行，不存在抽样偏差问题
3. **所有 agent 有 frontmatter**：通过 grep 确认 echo-sleuth 所有 agent 文件都有正确 frontmatter，未发生「验证者盲点」问题

### 5.2 挑战 / 验证

这次案例验证了两个之前不够清晰的认知：

- **「字段存在」不等于「字段正确」**：YAML description 里可以有 shell 替换表达式，一般的「有没有 description 字段」检查通不过去，需要内容验证。我的 CI 有没有做字段内容格式验证？
- **CI 扫描路径的盲区是系统性风险**：如果我的 CI 只扫 `skills/` 但忘了扫 `.claude/agents/`，我的 auditor agent 可能也有同样的盲点。要明确 CI 的扫描边界。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查我的所有 agent/skill 是否有 name 字段（包括 .claude/agents/）
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "*.md" \
  -path "*agent*" -o -path "*skill*" | \
  xargs grep -L "^name:" 2>/dev/null

# 2. 检查是否有 YAML description 字段含 shell 替换（仿 check_yaml_shell_substitution）
grep -rn -E 'description:.*\$\(|description:.*`' \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null

# 3. 检查我的脚本里有没有 sudo rm -rf 类破坏性命令
grep -rn "sudo rm -rf\|rm -rf /" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null

# 4. 检查 CI 的 find_agent_files() 或等价逻辑是否覆盖了 .claude/agents/
# （查看 CI 配置文件）
grep -rn "\.claude/agents\|\.claude\/agents" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/.github/ 2>/dev/null

# 5. 检查我的 commands/agents 是否有 allowed-tools 声明
grep -rL "allowed-tools" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null | \
  grep "\.md$"

# 6. 检查模糊量词
grep -rn -E '\b(comprehensive|robust|appropriate|carefully|thorough|ensure)\b' \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ \
  /tmp/my-repos/MarkQWu-claude-for-legal/ 2>/dev/null | head -30
```

命中后怎么办：
- YAML 里有 `$(...)` 表达式 → 立即修复，替换为字面值；同时检查是否是批量生成管道的遗留
- 没有 `.claude/agents` 的 CI 覆盖 → 在 CI 脚本里添加该路径到 find 命令
- `sudo rm -rf` 出现 → 添加前置 guard：`[[ -d "$TARGET_DIR" ]] && [[ -n "$TARGET_DIR" ]] || exit 1`
- 缺 `allowed-tools` → 根据 agent/command 实际调用的工具添加声明

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 的多个 agent 添加 `geepers_*` 风格的命名空间前缀
   - **为何可行**：echo-sleuth 有 `recall`、`memory-manager`、`synthesize` 等 agent，统一前缀后（如 `sleuth_recall`）在用户的插件列表里更清晰
   - **第一步**：在 frontmatter 里把 `name: recall` 改为 `name: sleuth_recall`，在 plugin.json 里更新注册名，约 10 分钟/个

2. **想法**：给 claude-for-legal 的法律程序 agent 采用 FairDB runbook 风格
   - **为何可行**：legal 插件的 reg-change-monitor 等 agent 描述的是有明确步骤的合规流程，和 FairDB 的 PostgreSQL 运维流程同构
   - **第一步**：在每个 legal agent 的正文里，把无序列表改为编号步骤，高风险步骤前加「⚠️ 安全检查：[具体检查项]」callout

3. **想法**：在 echo-sleuth 的 CI 里添加 YAML 内容格式验证（防止 shell 替换表达式进入 description 字段）
   - **为何可行**：如果未来引入批量生成工具，这类问题可能在不注意时发生
   - **第一步**：在 `.github/workflows/` 里加一步 `grep -rn 'description:.*\$(' skills/ agents/`，命中就报错，约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 jeremylongshore/claude-code-plugins-plus-skills 的核心目的**：成为 Claude Code 生态的一站式多域插件市场，覆盖游戏 UX、数据库运维、食品库存等跨领域场景

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中高 | 同为多域插件（多个法律子领域），有 agents + commands | 规模差异极大（400+ vs 约 20），legal 无 community 子目录结构 | **高** |
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code plugin，有多个 agent，有 CI | echo-sleuth 单一聚焦，无多生态并行问题 | **中** |
| MarkQWu/drama-workshop-skills | 低 | 都有 skill 合集 | drama-workshop 是内容创作类，结构更简单 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `.claude/agents/` 目录下的 agent 无 frontmatter | `grep -L "^name:" .claude/agents/*.md` | 需检查 echo-sleuth 和 claude-for-legal 的 `.claude/` 目录 | **高** |
| YAML description 含 shell 替换 | `grep -n 'description:.*\$(' **/*.md` | 如有批量生成流程则需排查 | **高** |
| CI 扫描路径不覆盖 `.claude/` | 查看 CI 配置 | 大概率未覆盖（多数小型仓库忽略此路径）| **高** |
| 运维 runbook 缺 guard clause | `grep -n "sudo rm" **/*.md` | echo-sleuth 无此问题（纯只读），claude-for-legal 需检查 | 中 |
| command 缺 `allowed-tools` | `grep -rL "allowed-tools" **/*.md` | echo-sleuth 和 claude-for-legal 均可能命中 | 中 |

**命中后的具体行动建议**：
- 如果 `.claude/agents/` 有 agent 文件缺 frontmatter → 参考 PR #535 的修复方式，添加 `name:` + `description:` + `model:` 三字段最低要求
- 如果发现任何 YAML 字段含 `$(...)` → 立即修复，并回查该文件的生成来源

### 7.3 别人的更优方案

1. **领域**：命名空间前缀统一相关 agent
   - **本案例做法**：`geepers_orchestrator_web`、`geepers_react`、`geepers_a11y` 等 14 个 agent 统一 `geepers_*` 前缀，在 marketplace 里天然聚类，用户一眼识别出同系列
   - **我的项目现状**：echo-sleuth 有 `recall`、`memory-manager`、`synthesize-experience` 等 agent，没有统一命名前缀，在用户的插件列表里散落各处
   - **如何借鉴**：把所有 echo-sleuth agent 的 `name:` 改为 `sleuth_*` 前缀，同步更新 plugin.json 和文档中的引用名

2. **领域**：FairDB 运维 runbook 风格（编号步骤 + 安全 callout）
   - **本案例做法**：FairDB 每个 command 是一个有编号步骤的执行手册，高风险操作前有明确的「Warning: this action is irreversible」类 callout，平均分 85-88
   - **我的项目现状**：claude-for-legal 的合规流程 agent（如 reg-change-monitor）描述步骤时用无序列表，没有编号，没有风险提示，用户按照 agent 输出操作时缺乏明确的安全意识
   - **如何借鉴**：在 legal agent 里，把「分析监管变更」等步骤改为「1. 确认目标法规范围 → 2. 检索最近 90 天变更 → ⚠️ 步骤 3 将生成正式通知文件，请确认合规官已审阅 → 3. 生成通知」

### 7.4 反向：我的项目做得比他们好的地方

1. **领域**：脚本安全（无破坏性命令）
   - **我的做法**：echo-sleuth 的所有脚本（`git log`、`grep`、`jq` 等）都是只读操作，天然没有 `sudo rm -rf` 类风险，无需 guard clause
   - **本案例做法**：jeremylongshore 的 FairDB runbook 包含真实 PostgreSQL 运维操作（`sudo rm -rf` 磁盘清理、备份恢复），存在需要特别防范的执行风险
   - **意义**：这是「选择合适的工具范围」带来的安全红利——如果 plugin 只做信息查询和分析，危险命令执行风险天然为零

2. **领域**：单生态聚焦带来的可测试性
   - **我的做法**：echo-sleuth 聚焦「对话记忆挖掘」单一场景，所有 agent 的行为都可以用 NL-TDD spec 完整覆盖
   - **本案例做法**：3000+ 制品的 NL-TDD 测试覆盖几乎不可能做到，NLPM 本次也只审计了 105/3000+ 个制品（3.5%）
   - **意义**：规模是 jeremylongshore 的优势，也是其测试覆盖的最大挑战；小仓库的全量可测试性是大市场无法轻易获得的质量保证

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明 artifact 的元数据。Claude Code 加载 agent 时先解析 frontmatter，如果没有 frontmatter，agent 文件在磁盘上存在，但在 Claude Code 里完全不可见、不可调用——就像一本书没有书名和目录。

### <a name="allowed-tools"></a>allowed-tools
> frontmatter 里声明该 agent/command 可以调用哪些工具（如 `bash`、`mcp__postgre__query`）的字段。缺少这个字段相当于「不限制工具访问」，静态分析工具无法验证权限边界，AI 执行时可能使用未预期的工具。

### <a name="shell-substitution-in-yaml"></a>YAML 里的 shell 替换
> 在 YAML 字符串值（如 `description: $(echo "$x" | cut ...)` ）里嵌入 shell 替换表达式。YAML 解析器不执行 shell，所以这个值会被原样保留为字符串字面量——但 NLPM 的 `check_yaml_shell_substitution()` 会把它标记为安全风险，因为它暗示这个文件是一个「未被正确渲染的模板」，可能包含其他未求值的占位符。

### <a name="guard-clause"></a>guard clause（保护子句）
> 在执行高风险操作前加的前置检查，例如 `[[ -n "${TARGET_DIR}" ]] || { echo "TARGET_DIR is empty, aborting"; exit 1; }`。如果没有 guard，`sudo rm -rf $EMPTY_VAR/...` 可能扩展为 `sudo rm -rf /...`，导致灾难性误操作。

### <a name="namespace-prefix"></a>命名空间前缀
> 给同系列 agent 统一加的名称前缀（如 `geepers_*`），使得同一生态的 artifact 在列表里聚类显示，便于用户识别和管理。类似于 Python 包里的模块前缀或 npm 的 `@scope/` 组织名。

### <a name="runbook"></a>runbook（运维手册）
> 把生产环境操作流程（如数据库备份、事故响应）写成有编号步骤、带有安全提示的执行手册。FairDB commands 是典型的 AI-assisted runbook：用户调用 agent，agent 按 runbook 格式输出可执行的步骤序列。与普通 prompt 的区别在于：runbook 假设指令最终会在真实生产系统上执行，因此安全 callout 和回滚说明是必要的，而不是可选的。

### <a name="ci-blind-spot"></a>CI 扫描盲区
> CI 验证脚本的 `find` 命令未覆盖的目录路径。本案例的经典案例：CI 只扫 `plugins/` 但不扫 `.claude/agents/`，导致 `skill-auditor.md`——仓库自己的 auditor agent——长期无 frontmatter 而不被 CI 发现。系统性盲区不是偶然的 bug，而是 CI 配置设计时的遗漏，修复需要扩展扫描路径，而不是修一个文件。
