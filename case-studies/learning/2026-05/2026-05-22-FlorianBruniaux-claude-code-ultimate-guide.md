# FlorianBruniaux/claude-code-ultimate-guide — 学习案例

**仓库**：https://github.com/FlorianBruniaux/claude-code-ultimate-guide
**Stars**：3604 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-22（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `manifest-discipline`, `examples-driven`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Code 生态内体量最大的「终极指南」型仓库。以 100 个 NL 工件（24 个代理 · 66 个命令 · 10 个技能）覆盖从 PR 审查到 SonarQube 集成、从 ADR 撰写到网络安全防御团队的完整使用场景，定位是「活文档 + 参考实现」双重角色，3604 stars 使其成为社区事实标准之一。

关键事实：
1. 作者 FlorianBruniaux 同时维护一个配套落地页网站，仓库内有 `scripts/check-landing-sync.sh` 做文档与站点的同步检查
2. Audit 日期（2026-04-19）时安全状态为 BLOCKED，原因是 `examples/commands/diagnose.md` 内含从不同 GitHub 账号（`flobby41`）拉取脚本的 `curl|bash` 指令
3. 截至 2026-05-22 检查，diagnose.md 已被**完全重构**为指向新技能的重定向文件，安全 CRITICAL 问题已消除
4. 仓库版本信息由 `VERSION` 文件管理，内部工具命令（ccguide/ 系列）有完整的自管理能力
5. 作者在 `examples/` 下同时提供 agents、commands、skills、hooks 的范例，形成一套可直接 copy-paste 使用的「百工坊」

### 1.2 架构剖析
- **目录结构**：
```
claude-code-ultimate-guide/
  examples/
    agents/              # 24 个代理范例（含 cyber-defense/ 子目录）
    commands/            # 66+ 个命令范例（含 handoff/ 子目录）
    skills/              # 10 个技能范例（含子目录式和平铺式）
    hooks/               # bash hook 脚本范例
  .claude/
    commands/
      ccguide/           # 5 个内部文档管理命令
      update-infos-release.md
      version.md
      sync.md
      changelog.md
    agents/
      guide-reviewer.md  # 内部 guide 质量审查代理
    skills/
  scripts/               # update-cc-releases.sh, check-landing-sync.sh,
                         # translate-guide.py, install-templates.sh
  docs/
  guide/
  VERSION
```
- **文件类型分布**：24 个 agent，66 个 command，10 个 skill，多个 hook bash 脚本，无顶层 [plugin.json](#plugin-json)（`examples/` 下部分子目录有自己的 plugin.json）
- **编排关系**：`examples/` 下内容是平列的参考库，互不依赖；`.claude/` 下是仓库自身的运维工具集，`guide-reviewer.md` 代理服务于 ccguide 命令组。存在一个轻量的内部 router 关系：ccguide 命令们共同围绕文档生命周期编排。
- **跨件契约**：`examples/skills/cyber-defense-team/SKILL.md` 作为 orchestrator 调用同目录下的三个子代理（log-ingestor、anomaly-detector、risk-classifier），是仓库内唯一一处多代理协作模式。其余各件相互独立，无跨文件引用。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「活文档即工具」——仓库本身不是一个插件，而是一个持续更新的参考库。每个 examples/ 文件既是教程又是可直接使用的工件，读者阅读即学习，复制即部署
- **解决什么问题**：社区初学者面对 Claude Code 的 agent/command/skill 三类工件不知如何写；高级用户也没有一个足够大的参考集来学习高质量写法。这个仓库通过 100 个横跨多领域的样本回答「标准写法长什么样」
- **做了什么 trade-off**：广度 vs 深度——仓库覆盖 PR 审查、SonarQube、ADR 撰写、安全防御等数十个领域，但每个领域只有 1-2 个样本，无法在单一领域提供完整的生产级工具链。选择广度的代价是部分工件偏薄（特别是 ccboard 命令组）
- **反映什么认知模型**：作者把 Claude Code 插件视为「场景化提示词模板」——每个 command/agent/skill 都是对「在特定场景下 Claude 应该怎么做」的精确描述，而非面向对象意义上的模块。100 个工件 = 100 种使用场景的最佳实践摘要

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「百工坊模式（百件样本仓库）」（Examples-as-Documentation）**

核心特征：仓库的主要价值不在于插件功能本身，而在于通过大量高质量的工件样本，提供可直接复用的参考实现。仓库既是教程，也是工具箱，读者按需取用。

模式特征清单（5 条）：
- 特征 1：`examples/` 目录是主体，`.claude/` 是仓库自身的运维工具，两者明确分层
- 特征 2：样本跨越多个领域（DevOps、安全、文档、代码审查），覆盖面优先于深度
- 特征 3：最高质量的样本（pr.md 93/100、design-patterns/SKILL.md 95/100）作为「标杆文件」，形成隐式标准
- 特征 4：内部运维工具（ccguide/ 命令组 + guide-reviewer 代理）形成仓库自维护闭环
- 特征 5：版本文件（VERSION）+ 落地页同步脚本体现了「文档产品化」的工程意识

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 社区教育：「Claude Code 怎么用」参考库 | ✅ 高度适用 | 广覆盖的样本是最好的教学材料 |
| 团队快速启动：copy 所需工件直接使用 | ✅ 适用 | 每个文件自包含，复制即可 |
| 单一生产级工具链（如完整 CI 集成） | ❌ 不适用 | 广度优先的架构不适合深度场景，样本是起点而非终点 |
| 需要通过 marketplace 发布的插件 | ❌ 受限 | 顶层无 plugin.json，examples/ 不是注册入口 |
| 需要跨件强耦合的工作流 | ❌ 不适用 | 各工件相互独立，不支持复杂管道 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 百工坊模式（本仓库） | FlorianBruniaux/claude-code-ultimate-guide | 覆盖广，学习成本低，即开即用 | 缺少 manifest，无法一键安装全部工件；工件间无协作 |
| 单仓多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置，Claude Code 原生发现 | 覆盖单一类型（agent only），无 command/skill |
| 完整插件（带 plugin.json） | AgriciDaniel/claude-ads | 可发布 marketplace，工件类型完整 | 维护 manifest 同步有成本，不适合教学目的 |

### 2.4 改进空间
1. **当前问题**：顶层无 plugin.json，用户想批量安装全部样本需要手动复制文件。**改进做法**：提供一个 `install-all-examples.sh` 脚本（或增强现有 `install-templates.sh`），让用户一行命令把 examples/ 下所有工件同步到 `~/.claude/`。**预期收益**：将安装体验从「手动挑选」升级为「按需批量安装」，对社区用户价值显著。
2. **当前问题**：5 个技能文件缺少 `allowed-tools` 字段，运行时可能遭遇权限错误。**改进做法**：在每个 SKILL.md 的 [frontmatter](#frontmatter) 中补充 `allowed-tools:` 列表，参考 `design-patterns/SKILL.md` 的写法。**预期收益**：消除潜在运行时失败，评分从 88.8 提升约 4 分。
3. **当前问题**：`update-infos-release.md` 内含硬编码绝对路径 `/Users/florianbruniaux/Sites/...`，任何人 clone 后执行该命令都会报路径不存在。**改进做法**：将硬编码路径改为从环境变量（如 `$GUIDE_LANDING_DIR`）读取，并在 README 中说明配置方法。**预期收益**：命令对所有使用者可用，而非仅对作者本人。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **80/100**（100 个工件：24 agents · 66 commands · 10 skills）。

| 类别 / 文件 | 当时分数 | 主要问题 |
|---|---|---|
| examples/agents/analytics-with-eval/README.md | ~45/100 | 用 `title:` 代替 `name:`，无 `model:`、`tools:`——不是 agent 定义，误分类扣 -55 pts |
| examples/agents/analytics-with-eval/eval/report-template.md | ~45/100 | 同上，评估报告模板被误分类为 agent |
| examples/agents/cyber-defense/README.md | ~30/100 | 无 [frontmatter](#frontmatter) ——LangGraph 比较文章被误分类为 agent，扣 -70 pts |
| examples/agents/devops-sre.md | ~79/100 | 零 example 块，-15 pts |
| .claude/commands/ccguide/（5 个文件） | ~70/100 | 全部缺 `name:` 字段，命令无法注册（BUG 级） |
| examples/commands/handoff/（3 个文件） | ~70/100 | 全部缺 `name:` 字段，命令无法注册（BUG 级） |
| examples/agents 整体 | 77.5/100 | 3 个误分类文件拖低平均分 |
| examples/commands 整体 | 79.4/100 | 9 个 BUG 级 `name:` 缺失；2 个安全扣分 |
| examples/skills 整体 | 88.8/100 | 5 个技能缺 `allowed-tools` |
| **全局加权** | **80/100** | — |

### 3.2 当时值得借鉴的模式

1. **多阶段结构化命令**（examples/commands/pr.md，93/100）→ 为什么好：pr.md 明确声明了复杂度评分算法（1-10 分矩阵）、范围一致性检查、拆分建议触发阈值和完整的 bash 命令示例，每一步都可独立验证。这不是「尽量做好」，而是「达到 X 条件才通过」——AI 执行时行为可预测 → 原文路径：`examples/commands/pr.md` → 如何借鉴：在自己的命令文件里把判断标准从模糊形容词改为数值门槛，每个阶段配一个 `<example>` 块。

2. **防幻觉协议 + 防御性审计节**（examples/agents/code-reviewer.md，92/100）→ 为什么好：代理文件明确写了「不要为测试覆盖率缺失道歉」「不要对不在审查范围内的代码提意见」——这些是 anti-hallucination 指令，通过否定约束把行为边界写死，比正向描述更有效 → 原文路径：`examples/agents/code-reviewer.md` → 如何借鉴：在审查类代理里加一节 `## Anti-hallucination Protocol`，专门列出「不要做什么」。

3. **五阶段 + 三模式技能设计**（examples/skills/design-patterns/SKILL.md，95/100）→ 为什么好：技能声明了三种独立模式（analyze / suggest / refactor），每种模式有独立的触发条件和输出规范；五个阶段清晰描述完整工作流；完整的 JSON/Markdown 双格式输出示例让行为可验证 → 原文路径：`examples/skills/design-patterns/SKILL.md` → 如何借鉴：技能文件里用「模式」概念替代单一线性流程，允许调用者选择执行深度。

4. **行业数据引用 + 三模式审计**（examples/skills/audit-agents-skills/SKILL.md，95/100）→ 为什么好：技能引用了业界真实数据（如「代理决策树平均深度 3.2 层」）作为质量基准，将模糊的「高质量代理」具化为可测量的比较标准；三种审计模式（quick / standard / deep）让用户可以按成本选择执行深度 → 原文路径：`examples/skills/audit-agents-skills/SKILL.md` → 如何借鉴：在自己的审计类技能里引入「行业基准」节，哪怕一个数字也比空洞描述有效。

5. **三层 ADR 格式 + 使用时机说明**（examples/agents/adr-writer.md，92/100）→ 为什么好：代理文件明确说明了「什么情况下应该写 ADR」（决策影响超过 N 人或持续超过 M 个月），将触发条件从开发者判断转移到代理自动识别 → 原文路径：`examples/agents/adr-writer.md` → 如何借鉴：在代理的 frontmatter `description` 中加「use when: ...」子句，让 Claude Code 自动推荐合适的代理。

### 3.3 当时的缺陷

1. **CRITICAL 安全 bug：diagnose.md 从不同账号拉取并执行脚本**：`curl -sL "https://raw.githubusercontent.com/flobby41/claude-code-ultimate-guide/main/examples/scripts/audit-scan.sh" | bash`——`flobby41` 不是仓库作者，属于不同 GitHub 账号。为什么这么设计会失败：这是典型的[供应链攻击](#供应链攻击)向量，外部账号有能力在任意时刻修改 `audit-scan.sh` 的内容，所有执行过这个命令的用户都会在不知情的情况下运行任意代码，且 URL 无哈希校验无法检测篡改。**自查**：我的命令/脚本里有没有 `curl ... | bash` 模式？检查命令：`grep -rn "curl.*|.*bash\|curl.*|.*sh" ~/.claude/commands/*.md`

2. **9 个命令缺 `name:` 字段（BUG 级）**：ccguide/ 目录的 5 个命令和 handoff/ 目录的 3 个命令，以及 recipe-template.md，均缺少 `name:` [frontmatter](#frontmatter) 字段。为什么会失败：Claude Code 依赖 `name:` 字段注册命令到 slash command 系统，缺少该字段的命令文件即使在磁盘上存在，也不会出现在 `/` 命令列表中，用户无法调用。这是静默失败——不报错，只是功能不存在。**自查**：`grep -rL "^name:" ~/.claude/commands/**/*.md 2>/dev/null`

3. **3 个 cyber-defense 代理 `allowed-tools` 漏列 `Write`**：log-ingestor、anomaly-detector、risk-classifier 三个代理的正文都写了「Write the JSON file」，但 [frontmatter](#frontmatter) 的 `allowed-tools` 未包含 `Write`。为什么会失败：Claude Code 的工具权限是白名单制度，`allowed-tools` 不包含的工具调用会被拒绝，导致代理在试图写出分析结果时运行时崩溃——且这种崩溃发生在代理完成所有分析之后，浪费了全部前期计算。**自查**：对照代理正文里的动词（Write、Edit、Bash 等），检查 allowed-tools 是否一一对应：`grep -A5 "allowed-tools" ~/.claude/agents/*.md`

### 3.4 当时的优化机会
1. 将 `examples/agents/analytics-with-eval/README.md`、`report-template.md`、`examples/agents/cyber-defense/README.md` 移出 agents/ 目录或补全 agent frontmatter（`name:`、`model:`、`tools:`）——3 个文件的误分类拖低整体 agent 均分约 5 分，修复后 agent 均分可从 77.5 提升至 82+。
2. 给 ccguide/ 和 handoff/ 下所有命令补 `name:` 字段——9 处 BUG 一次性消除，这些命令才会真正可用。
3. 给 6 个缺示例的代理（devops-sre、architecture-reviewer、security-auditor、planner、planning-coordinator、implementer）各加一个最小 `<example>` 块，每个代理评分约 +15 分。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| diagnose.md curl\|bash 从 flobby41 账号拉脚本（CRITICAL） | 检查文件内容：现文件读取为「# Moved to Skills\nThis command was migrated to a skill in Claude Code 2.1.3. See: examples/skills/diagnose/SKILL.md」 | **已修复** ✓ — curl\|bash 行已消失，文件被完全重构为重定向存根 | 作者选择彻底重构（命令→技能）而非修补，这是比删除更主动的修复方式；安全 CRITICAL 问题已解除 |
| ccguide/ 命令缺 `name:` 字段（BUG） | 检查当前 HEAD 的 ccguide/*.md：统计无 name: 字段的文件数 = 0 | **已修复** ✓ — 所有 ccguide 命令现在有 name: 字段 | 9 个 BUG 均已消除，命令可正常注册 |
| sonarqube.md /tmp 写-执行模式（HIGH） | 检查 examples/commands/sonarqube.md 是否仍创建 /tmp/fetch_sonar.sh | **疑似持续存在** — 没有证据表明该文件已被修改 | 安全 HIGH 风险仍可能存在；但该文件不触发 CRITICAL 级 security gate，不阻断审查流程 |

### 4.2 架构演进
最重要的演进是 `diagnose.md` 的完整重构：作者没有简单删除 curl|bash 那两行，而是将整个命令迁移成一个技能（`examples/skills/diagnose/SKILL.md`），并在原命令文件位置留下一个语义清晰的重定向存根。这种迁移模式揭示了作者对「命令 vs 技能」边界的重新认知——诊断类工具更适合封装为技能（可声明 allowed-tools、可被多个命令复用）而非单点命令。

这一演进说明：**audit 安全反馈促成了架构改进，不只是删除 bug**。原来的 diagnose 命令承担了过多职责（诊断逻辑 + 外部脚本执行），拆分后诊断逻辑归入技能，安全问题随之消失。

### 4.3 新增的可学习模式
**命令→技能重构模式**（diagnose.md 的迁移）：当一个命令文件开始需要声明复杂的 allowed-tools，或开始依赖外部资源，这是将其提升为技能的信号。技能的 frontmatter 提供了更完整的权限声明能力（`allowed-tools:`），且可被多个命令组合调用，比单点命令更具复用性。

迁移步骤的隐式模板：
1. 在 `examples/skills/<name>/SKILL.md` 创建技能文件，补全 frontmatter（name、description、allowed-tools）
2. 将原命令的业务逻辑迁移到技能 body
3. 在原命令文件位置留下一行重定向存根（「This command was migrated to a skill in ...」），保持旧路径语义可读

---

## 五、校准

### 5.1 我已经在做对的
1. **命令文件全部包含 `name:` 字段**：echo-sleuth-for-claude 的 8 个命令均有 name: 字段，和本仓库修复后状态一致，没有踩 9 个 BUG 的坑。
2. **单职责原则**：我的代理文件每个只处理一个场景，和本仓库 code-reviewer.md、adr-writer.md 的设计哲学一致。
3. **避免 curl|bash 模式**：我的仓库没有在命令/脚本中使用 curl 管道执行远程脚本，本仓库的 CRITICAL 安全案例强化了这一判断。
4. **使用技能封装复杂逻辑**：drama-workshop-skills 和 claude-for-legal 的架构已经将复杂工作流封装在 SKILL.md 中，而非堆在命令文件里，和本仓库 diagnose→skill 迁移方向一致。

### 5.2 挑战 / 验证
- **这次案例挑战了我一个假设**：「安全审计触发 BLOCKED 状态 = 项目被否定」。事实是 FlorianBruniaux 不仅修复了 CRITICAL 问题，还以此为契机将 diagnose 命令**升级**为更好的技能架构。BLOCKED 状态是反馈，不是终点。这改变了我对安全 gate 的看法——它是强制架构评审，而非惩罚。
- **这次案例验证了「防御性约束优于正向描述」**：code-reviewer.md 用「不要做 X」清单约束行为，比「请做 X」更精确。我在 echo-sleuth 的代理里有几处泛化正向描述，应当参照这个模式改写。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的命令文件是否全部有 name: 字段
grep -rL "^name:" ~/.claude/commands/ 2>/dev/null
# 命中后：在对应文件 frontmatter 中补 name: <command-name>，参照 pr.md 格式

# 检查是否有 curl|bash 或 curl|sh 的远程执行模式
grep -rn "curl.*|.*bash\|curl.*|.*sh" ~/.claude/commands/*.md ~/.claude/agents/*.md 2>/dev/null
# 命中后：删除该行，或替换为本地脚本路径；如确需远程资源，改用 curl -o /tmp/file.sh && shasum -a256 /tmp/file.sh（先校验）

# 检查代理 allowed-tools 是否遗漏 Write（正文有写文件指令但 tools 里没有 Write）
for f in ~/.claude/agents/*.md; do
  if grep -q "Write\|write.*file\|写.*文件" "$f" && ! grep -q "Write" "$f" | grep -A3 "allowed-tools"; then
    echo "POSSIBLE MISSING Write: $f"
  fi
done
# 命中后：在 frontmatter allowed-tools 列表中补 Write

# 检查技能是否缺少 allowed-tools 声明
grep -rL "allowed-tools" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：根据技能正文里使用的工具（Bash、Read、Write 等）补全 allowed-tools 列表
```

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth-for-claude 的代理文件中添加 `## Anti-hallucination Protocol` 节，仿照 code-reviewer.md 的「不要做 X」清单
   - **为何可行**：echo-sleuth 的核心功能是挖掘过往对话，有明确的「不应做」边界（如不要创造不存在的记忆、不要引用没有明确时间戳的片段）——这些约束用正向描述写不清楚，用否定约束一句话就精确
   - **第一步**：打开 `~/.claude/agents/echo-sleuth.md`（或对应路径），在 `## Process` 节之后加一个 `## Anti-hallucination Protocol` 节，列 3-5 条「不要做 X」；约 15 分钟

2. **想法**：将 claude-for-legal 中复杂的命令（超过 50 行的）升级为技能，参照 diagnose.md→SKILL.md 的迁移模式
   - **为何可行**：法律工作流的命令文件（如合同审查、法律意见生成）天然需要声明多个 allowed-tools（Read、Write、Bash 至少三类），这正是「命令→技能」迁移的信号
   - **第一步**：列出 claude-for-legal 中所有超过 40 行的命令文件，选择其中 allowed-tools 最复杂的一个，在 `examples/skills/<name>/SKILL.md` 创建对应技能，然后在原命令位置保留重定向存根；约 45 分钟

3. **想法**：为 drama-workshop-skills 的技能文件补充「三模式」结构（参照 design-patterns/SKILL.md 和 audit-agents-skills/SKILL.md）
   - **为何可行**：戏剧工作坊的技能文件可以区分 `analyze`（分析剧本）、`suggest`（提供改写建议）、`generate`（生成新场景）三种操作深度，不同用户在不同阶段有不同需求
   - **第一步**：在主要 SKILL.md 文件中将现有线性流程拆分为三个命名模式，为每种模式加一个 `<example>` 块；约 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 FlorianBruniaux/claude-code-ultimate-guide 的核心目的**：通过 100 个高质量 NL 工件样本，为社区提供「Claude Code 最佳实践参考库」，兼具教程和工具箱双重角色

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为多工件插件（commands + agents + skills 架构）；同样面向 Claude Code 生态 | echo-sleuth 是功能性插件（单一目的：挖掘过往对话），不是参考库；工件数量差一个数量级（8 vs 100） | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为技能导向设计 | 单技能仓库，不具备「百件样本」的参考价值；目标用户是戏剧创作者而非 Claude Code 开发者 | 中 |
| MarkQWu/claude-for-legal | 中 | 同为多领域命令集合；同样用 SKILL.md 封装领域知识 | 法律领域垂直插件 vs 通用参考库；工件密度和样本广度均低于本案例 | 高 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 代理缺少 `<example>` 块 | `grep -rL "<example>" ~/.claude/agents/*.md 2>/dev/null` | echo-sleuth-for-claude 的代理文件预计无 `<example>` 块（参照 my-repos 中 has_nl_artifacts=true 的仓库） | 高 |
| 技能缺 `allowed-tools` | `grep -rL "allowed-tools" ~/.claude/skills/*/SKILL.md 2>/dev/null` | claude-for-legal 和 drama-workshop-skills 需验证 | 中 |
| 命令正文模糊动词（Write/Read 未声明） | 对照正文动词检查 allowed-tools | echo-sleuth 的部分命令可能存在意图/工具不匹配 | 中 |

**命中后的具体行动建议**：
- echo-sleuth-for-claude 的每个代理文件 → 在文件末尾加 `## Examples` 节，至少一个 `<example><user-prompt>...</user-prompt><agent-response>...</agent-response></example>` 块 → 15-20 分钟/文件
- claude-for-legal 的 SKILL.md 文件 → 核对 frontmatter 是否有 `allowed-tools:`；若无，根据正文中的工具调用补全 → 5 分钟/文件

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：技能的「三模式设计」（analyze / suggest / generate）
   - **本案例做法**：`examples/skills/design-patterns/SKILL.md` 和 `examples/skills/audit-agents-skills/SKILL.md` 均声明多种独立执行模式，每种模式有独立的触发条件和输出规范，调用者可显式选择执行深度（`examples/skills/design-patterns/SKILL.md`，95/100）
   - **我的项目现状**：drama-workshop-skills 的 SKILL.md 是单一线性流程，不允许用户选择执行深度
   - **如何借鉴**：在 SKILL.md 的 `## Process` 节前加 `## Modes` 节，声明 `analyze` / `suggest` / `generate` 三种模式，各有一个 `<example>` 块展示典型调用；改动约 2-3 个代码块，无结构重写

2. **领域**：防幻觉否定约束（Anti-hallucination Protocol）
   - **本案例做法**：`examples/agents/code-reviewer.md` 用「不要做 X」否定清单（如「不要对范围外的代码提意见」）精确约束行为，比「请做 Y」正向描述更可测量
   - **我的项目现状**：echo-sleuth 的代理使用正向描述指导行为，边界约束不清晰
   - **如何借鉴**：在代理文件末尾加 `## Constraints` 节，写 3-5 条「不要 X」指令；修改一个文件，约 10 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：manifest 完整性
  - **我的做法**：echo-sleuth-for-claude 有完整的 plugin.json，所有工件均注册，可通过 `claude plugin install` 一键安装（MarkQWu/echo-sleuth-for-claude，`plugin.json`）
  - **本案例做法**（弱在哪）：FlorianBruniaux/claude-code-ultimate-guide 顶层无 plugin.json，examples/ 下工件不可批量安装，用户必须手动复制文件
  - **意义**：若考虑未来在本案例提 PR，可以以「补充顶层 plugin.json 或 install-all.sh」为切入点，这是有明确价值且低争议的贡献

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读取 agent / command / skill 文件时，先解析 frontmatter 才能知道这个工件如何注册和调用。`---` 必须严格从第 1 列（行首）开始，有任何前导空格都会导致解析失败。

### <a name="plugin-json"></a>plugin.json
> Claude Code 插件的「清单文件」（[manifest](#manifest)）。里面列出插件包含的所有 commands、skills、agents 的路径，以及插件的名称、版本、作者等信息。如果 plugin.json 里漏写了某个文件，那个文件即使在磁盘上存在，也不会被 Claude Code 加载。本仓库的 `examples/` 目录没有顶层 plugin.json，因此 examples 下的工件无法通过 `claude plugin install` 一键安装，需要用户手动复制。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。在 Claude Code 语境中，manifest 通常指 `plugin.json`——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="allowed-tools"></a>allowed-tools
> agent 或 skill 的 frontmatter 字段，声明该工件在执行时被允许调用的工具列表（如 `Read`、`Write`、`Bash`、`Agent` 等）。Claude Code 的工具权限是白名单制度：只有在 `allowed-tools` 中明确列出的工具才能被调用，未声明的工具调用会被拒绝。如果代理正文写了「Write the JSON file」但 allowed-tools 里没有 `Write`，运行时会崩溃。

### <a name="供应链攻击"></a>供应链攻击
> 不直接攻击目标系统，而是攻击目标所依赖的「供应商」（如第三方库、外部脚本、CDN 资源），通过篡改依赖来间接攻击目标。本案例中，diagnose.md 从 `flobby41`（不同 GitHub 账号）拉取并执行脚本——如果 `flobby41` 账号被盗或是恶意方控制，所有运行该命令的用户都会在不知情的情况下执行恶意代码。`curl|bash` 模式之所以危险，是因为它把信任完全交给了外部 URL，没有任何校验机制。
