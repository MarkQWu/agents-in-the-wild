# jeremylongshore/claude-code-plugins-plus-skills — 学习案例

**仓库**：https://github.com/jeremylongshore/claude-code-plugins-plus-skills
**Stars**：1917 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-06-13（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `vague-quantifier`, `examples-driven`, `template-design`

**xiaolai 案例**：[../../case-studies/2026-04-24-jeremylongshore-claude-code-plugins-plus-skills.md](../../case-studies/2026-04-24-jeremylongshore-claude-code-plugins-plus-skills.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

jeremylongshore/claude-code-plugins-plus-skills 是目前 Claude Code 生态中规模最大的公开插件仓库之一：**423 个插件、2,849 个 skill、177 个 agent**，附带一个插件市场网站（tonsofskills.com）和 CLI 包管理器（`ccpi`）。一句话定位：**一站式 Claude Code 插件超市，覆盖 devops、数据库、安全、testing、SaaS 集成等几乎所有开发场景**。

关键事实：
- 1917 stars，270 forks；维护者 jeremylongshore（intentsolutions.io）
- 插件规模达到"库级"——不是几十个 skill，而是整个可浏览的生态系统
- 内含 `backups/` 目录，保留了批量增强管线处理前的历史版本快照
- `workspace/lab/` 目录用于实验性开发，不是生产级工件
- Geepers 插件（15 个 agent）是仓库中设计最完整、质量最高的单一插件

### 1.2 架构剖析

```
claude-code-plugins-plus-skills/
├── plugins/
│   ├── geepers/         # 多 agent web 开发团队（15 agents）
│   ├── database/        # 数据库相关插件（nosql, orm, validation, freshie）
│   ├── ...（400+ 其他插件）
├── skills/              # 独立 skill 集合
├── workspace/
│   └── lab/
│       └── schema-optimization/  # 实验性 BigQuery 优化 agent pipeline
├── templates/           # 新插件/agent 模板
├── backups/             # 历史版本快照（非生产）
├── .claude/
│   └── agents/
│       └── skill-auditor.md     # 内部 skill 审计 agent
├── scripts/             # 自动化测试和工具
├── freshie/             # freshie-inventory-manager 插件
└── marketplace/         # tonsofskills.com 市场相关
```

**文件类型分布**（按 audit 时）：105 个已审计工件（31 agents, 74 commands），实际总量远大于此

**编排关系**：仓库内分两类：
1. **插件级编排**（如 Geepers）：15 个 agent 形成多 agent 团队，`geepers_orchestrator_web` 作为协调者
2. **pipeline 型**（schema-optimization）：5 个 phase agent 按序执行，JSON I/O 传递

**跨件契约**：README 与插件命名一致，marketplace 和 ccpi 依赖插件 manifest 准确。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「平台型插件超市」——不止是一个仓库，而是想成为 Claude Code 插件的 npm registry
- **解决什么问题**：Claude Code 缺乏中央化的插件发现和安装机制，ccpi 填补这个空白
- **做了什么 trade-off**：广度 vs 深度。423 个插件覆盖面极广，但批量生成导致部分工件质量参差
- **反映什么认知模型**：作者把 Claude Code 生态视为类似 VS Code 插件市场，自己承担"生态建设者"角色

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「多 agent 专家团队」（Multi-Agent Expert Team）**

以 Geepers 插件为代表（仓库中质量最高的子模块）：

模式特征：
- 1 个 orchestrator agent（`geepers_orchestrator_web`）+ N 个专域 agent（tdd, schema, api, security, mobile...）
- orchestrator 按需调用专域 agent，用户只与 orchestrator 交互
- 每个 agent 3 个 `<example>` 块，覆盖触发场景
- agent 之间职责清晰不重叠

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 全栈 web 开发（多维度同时存在：TDD/schema/api/a11y） | ✅ 高度适用 | 多 agent 分工比单 agent 更聚焦 |
| 单一线性任务 | ❌ 过度设计 | orchestrator 层增加调用开销 |
| 需要明确 I/O 契约的 agent pipeline | ✅ 适用（schema-optimization 模式） | JSON I/O 契约确保 phase 间数据准确传递 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多 agent 专家团队（Geepers 模式） | jeremylongshore/geepers | 专域深度高，用户界面简单 | orchestrator 需要明确调度逻辑 |
| 单 agent 平铺 | iannuttall/claude-agents | 极简 | 无跨 agent 协作 |
| Pipeline agent（JSON I/O） | schema-optimization | 数据流可追溯，阶段可独立调试 | 需要严格的 JSON schema 契约 |

### 2.4 改进空间

1. **当前问题**：`backups/` 目录中仍有大量结构不完整的历史工件，会被 NLPM 误认为生产工件并拉低整体得分  
   **改进做法**：在 `.claude/nlpm.local.md` 中添加路径排除配置（`exclude: [backups/]`）  
   **预期收益**：NLPM 评分从 73/100 提升至接近 Geepers 的实际质量（91-93）

2. **当前问题**：`templates/full-plugin/agents/example.md` 中的 body 是占位符（"Agent instructions here."）  
   **改进做法**：用 Geepers agent 的结构作为模板示例，展示一个完整的"role + approach + output format + examples"结构  
   **预期收益**：用此模板创建的新 agent 质量显著提升

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-17 当时得分 **73/100**。

| 文件组 | 当时分数范围 | 主要问题 |
|---|---|---|
| schema-optimization agents (6) | 30 | 零[frontmatter](#frontmatter)（-70 每个） |
| backup-strategy.md | 20 | Shell 替换表达式在 YAML frontmatter 中 |
| sync-agent-context.md | 40 | 零 frontmatter |
| 10 个 backup commands | 55 | 缺 `name` 字段（-25 每个） |
| Geepers agents (15) | 91-93 | 2-3 个 example 块（低于推荐的 3 个）；缺 model |
| database plugin agents | 70-80 | 缺 example 块；缺 model |

### 3.2 当时值得借鉴的模式

1. **Geepers agent 的 3-element 结构** → name + description（含 2-3 example）+ model + 清晰的 output format section → 这是本 audit 中见到的最完整的 agent 结构示范 → 路径：`plugins/geepers/agents/geepers_tdd.md` → 借鉴：把这个结构用作自己 agent 的参考模板

2. **JSON I/O 契约（schema-optimization pipeline）** → 即使 phase agent 当时无 frontmatter，它们的 body 定义了严格的 JSON 输入输出格式，形成了"接口文档"→ 这种做法让 pipeline 各阶段可以独立验证 → 路径：`workspace/lab/schema-optimization/agents/phase_1.md`

3. **FairDB 命令的编号步骤 + 安全标注** → `backups/.../fairdb/commands/` 中的运维命令用编号步骤 + 明确的安全说明（"验证前请确认 $TARGET_DIR"）→ 这是运维类 command 的正确写法 → 借鉴：任何涉及破坏性操作的 command 都应有安全守卫说明

4. **freshie-inventory-manager agents 的分工清晰度** → `discovery-scanner`、`anomaly-detector`、`compliance-validator` 三个 agent 职责不重叠，形成完整的库存管理流水线 → 三 agent 分工设计是可借鉴的 pipeline 模式

5. **`skill-auditor.md` 的方法论详细度** → 虽然当时无 frontmatter（无法加载），body 中的审计方法论非常详细：多阶段分析 + 明确的评分标准 → 内容质量高，只是被 frontmatter 缺失"埋没"了

### 3.3 当时的缺陷

1. **6 个 agent 零 frontmatter**（schema-optimization phases + skill-auditor）  
   根本原因：`workspace/lab/` 路径暗示这些是实验性内容，作者可能未完成迁移到生产格式；skill-auditor 可能是内部工具，忘记添加 frontmatter  
   为什么失败：Claude Code agent 必须有 frontmatter 才能注册和加载，没有 frontmatter = 文件存在但永远不会被调用  
   **自查**：我的仓库有没有 body 写得很好但忘记加 frontmatter 的文件？

2. **backup-strategy.md 的 Shell 替换表达式在 YAML 中（HIGH 安全发现）**  
   根本原因：这是一个批量增强管线的未实例化模板残留，`$(echo "$description" | cut ...)` 从未被执行  
   为什么失败：YAML 解析器遇到 `$(...)` 表达式行为未定义；部分解析器会尝试解析为命令，存在安全风险；这是典型的"模板从未被测试过"反模式  
   **自查**：我的仓库里有没有未实例化的 shell 模板占位符？

3. **`incident-p0-disk-full.md` 中 `sudo rm -rf` 无守卫**（HIGH 安全发现）  
   根本原因：运维 SOP 命令直接写了 `sudo rm -rf /var/lib/postgresql/16/main/pgsql_tmp/*`，没有先验证目标路径  
   为什么失败：路径参数错误时（如 `pgsql_tmp/` → `pgsql_tmp`），可能删除整个 PostgreSQL 数据集群  
   **自查**：我的仓库里的运维类 command 有没有 `rm -rf` 类操作？

### 3.4 当时的优化机会

1. 修复 7 个零 frontmatter 文件（最高优先级）
2. 用具体路径守卫替换 `sudo rm -rf` 的裸写法
3. 在 templates/ 中用 Geepers agent 作为示范模板，替换"Agent instructions here."占位符

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| schema-optimization agents 零 frontmatter | `head -5 workspace/lab/schema-optimization/agents/phase_1.md` | **已修复** ✅：phase_1.md 现有完整 frontmatter（name + description，JSON 契约完整） | NLPM 提交的 PR 已被接受 |
| skill-auditor.md 零 frontmatter | `head -5 .claude/agents/skill-auditor.md` | **已修复** ✅：现有 `name: skill-auditor` + `description` | 此 agent 现可被正确加载 |
| 10 个 backup commands 缺 name | `head -5 backups/.../overnight-setup.md`（如仍存在） | **无法直接验证**：backups/ 目录结构已变化，部分文件可能已被移除或重命名 | 批量 PR 可能已处理 |

**结论（结合 xiaolai 案例）**：xiaolai 的 2026-04-24 案例记录了 NLPM 提交了 PR 并被合并，最关键的 7 个零 frontmatter bug 已在 PR 合并后修复。这是学习系列中第一个可以验证"PR 已合并 → 缺陷消失"的案例。

### 4.2 架构演进

对比 audit 时到现在，最显著的变化：
- **修复类变化**：7 个零 frontmatter agent 都已补全 frontmatter（验证了 phase_1 和 skill-auditor）
- **结构新增**：`AGENTS.md`（Codex 支持）、`CHANGELOG.md`（版本历史）、`freshie/` 目录从 `plugins/database/` 提升为一级目录
- **说明**：仓库规模本来就大，增量变化在百分比上不明显，但关键缺陷已修

### 4.3 新增的可学习模式

**Geepers 多 agent 团队当前状态**值得作为参照：15 个 agent 均有 2-3 个 example 块、清晰的专域职责，`geepers_orchestrator_web` 有 description 截断问题但 body 完整——这展示了一个"距满分 7 分"的多 agent 团队是什么样的。与 audit 时相比，各 geepers agent 的 model 字段仍然缺失，说明维护者接受了功能型 bug 修复（frontmatter）但未处理质量建议（model 字段）。

---

## 五、校准

### 5.1 我已经在做对的

1. **frontmatter 完整性检查**：echo-sleuth 和 drama-workshop 的所有 agent/command 文件都有完整 frontmatter
2. **避免在 YAML 中写 Shell 表达式**：我的仓库无模板残留占位符
3. **破坏性操作有说明**：drama-workshop 中导出命令有"执行前确认"的注意事项

### 5.2 挑战 / 验证

本案例（结合 xiaolai 案例）验证了**「批量自动化生成工件的质量下限」**问题：当 423 个插件通过批量管线创建时，frontmatter 错误会系统性地出现（6 个零 frontmatter、10 个缺 name），因为模板生成过程没有经过逐文件人工验证。这挑战了"自动化 = 更高质量"的假设——**自动化带来规模，但也会把质量问题批量复制**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库中零 frontmatter 或缺 name 字段的 agent/command 文件
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ \
     /tmp/my-repos/MarkQWu-claude-for-legal/ \
  -name "*.md" \( -path "*/agents/*" -o -path "*/commands/*" \) | while read f; do
  if ! head -3 "$f" | grep -q "^---"; then
    echo "MISSING frontmatter: $f"
  elif ! grep -q "^name:" "$f"; then
    echo "MISSING name: $f"
  fi
done
```
命中后：立即添加 frontmatter，这是功能性 bug 而非质量建议。

```bash
# 检查我的仓库中是否有 shell 表达式在 YAML frontmatter 里
find /tmp/my-repos/ -name "*.md" | while read f; do
  # 检查 --- 之间的区域是否有 $() 表达式
  awk '/^---/{c++} c==1 && /\$\(/{print FILENAME": "$0}' "$f"
done | head -10
```
命中后：这是高安全风险，立即清除。

### 6.2 灵感 → 实施路径

1. **想法**：以 Geepers agent 的格式为模板，统一我的 echo-sleuth agents 的结构
   **为何可行**：Geepers 是学习系列中设计最完整的多 agent 实例，其"name + description with examples + model + 清晰 output format"结构已被证明高效
   **第一步**：打开 `plugins/geepers/agents/geepers_tdd.md`，复制其结构框架；打开 echo-sleuth 最弱的 agent（analyze.md），按框架重写，约 30 分钟

2. **想法**：为 claude-for-legal 创建多 agent 协作编排层，类似 Geepers orchestrator
   **为何可行**：claude-for-legal 已有 10+ 专域 skill，但用户需要手动选择调用哪个；orchestrator 可以根据问题类型自动路由
   **第一步**：创建 `managed-agent-cookbooks/legal-orchestrator/` 目录，先写一个最简单的 `privacy-product-router.md` agent，约 1 小时

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 jeremylongshore 的核心目的**：Claude Code 插件超市，批量覆盖开发工作流的所有场景

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 同为多插件/多 skill 仓库；同样有领域分类 | legal 专注单领域，jeremylongshore 横跨所有开发场景 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件 | echo-sleuth 是单一功能工具 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code 插件 | 完全不同场景 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Agent 零 frontmatter | `head -3 agents/*.md \| grep -L "---"` | **未命中**：echo-sleuth 5 个 agent 均有 frontmatter | — |
| Agent 缺 model 字段 | `grep -L "^model:" agents/*.md` | **命中**：echo-sleuth 5 个 agent 全缺 model（同第二案例发现）| 高 |
| Geepers 风格：缺第 3 个 example 块 | `grep -c "<example>" agents/*.md` | **未检查**：echo-sleuth agents 的 example 数量需要人工核查 | 中 |

**命中后的行动**：
- 第一优先级仍是为 `MarkQWu/echo-sleuth-for-claude/agents/*.md` 5 个文件添加 `model:` 字段
- 第二步：检查每个 agent description 中的 example 数量，确保 ≥ 2 个（理想是 3 个）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：多 agent 团队的 orchestrator 模式（Geepers）
   - **本案例做法**：`geepers_orchestrator_web.md` 协调 15 个专域 agent，用户只需与 orchestrator 对话
   - **我的项目现状**：claude-for-legal 无 orchestrator，用户需要自行选择调用 privacy-legal 还是 product-legal 还是 employment-legal
   - **如何借鉴**：创建 `legal-intake-agent.md`，根据用户描述的问题类型路由到对应子域 skill，约 2 小时

2. **领域**：JSON I/O 契约的 pipeline agent（schema-optimization）
   - **本案例做法**：phase_1 定义严格的 JSON 输出格式，phase_2 以同格式为输入，数据流可验证
   - **我的项目现状**：echo-sleuth 的多步 skill（搜索 → 提取 → 写入 memory）无正式的 I/O 契约描述
   - **如何借鉴**：在 echo-sleuth 主 SKILL 的流程描述中添加各步骤的"期望输入/输出格式"说明，约 30 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：模板质量控制（templates 不用占位符）
- **我的做法**：`MarkQWu/drama-workshop-skills` 的 skill 模板文件（如分集模板）使用真实内容作为示范，而非"[在此填写内容]"
- **本案例做法（弱在哪）**：`templates/full-plugin/agents/example.md` body 是 "Agent instructions here." 占位符——这个模板会误导使用者以为 agent 只需要 frontmatter 加一行文字
- **意义**：模板应该是"活的示范"而非空占位符；drama-workshop 在这一点上更成熟

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明 agent/skill/command 的元数据（name、description、model、tools 等）。Claude Code 必须先读 frontmatter 才能注册和调用这个组件——零 frontmatter = 组件存在但永远无法被加载。

### <a name="YAML"></a>YAML
> 一种用于配置文件的格式，用缩进表示层级关系。例如：`name: skill-auditor` 是一条 YAML 键值对。YAML 格式敏感，在其中写 `$(echo ...)` 这样的 Shell 表达式会导致解析错误或安全问题。

### <a name="orchestrator"></a>orchestrator
> 多 agent 体系中的"协调者"角色，不直接处理具体任务，而是根据用户请求决定调用哪些专域 agent，并汇总结果。Geepers 中的 `geepers_orchestrator_web` 是典型实例——用户说"帮我审查这个 PR"，orchestrator 决定调用 security agent + tdd agent + a11y agent。

### <a name="pipeline-agent"></a>pipeline agent
> 以流水线方式串联的多个 agent，每个 agent 接受上一步的输出作为输入，最终完成一个复杂任务。schema-optimization 的 phase_1 到 phase_5 就是 pipeline agent 的例子，各阶段之间用 JSON 格式传递数据。

### <a name="供应链风险"></a>supply chain risk（供应链风险）
> 当依赖（npm 包、Python 包、Docker 镜像）版本不固定时，攻击者可以发布恶意新版本，使依赖它的项目自动"升级"到带有后门的代码。`npm install -g pnpm@9.15.9`（版本锁定）比 `npm install -g pnpm`（不锁定）安全得多。
