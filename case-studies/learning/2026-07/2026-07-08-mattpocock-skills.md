# mattpocock/skills — 学习案例

**仓库**：https://github.com/mattpocock/skills
**Stars**：69,816 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-11（历史快照）| **生成日期**：2026-07-08（基于当前 HEAD）
**主题标签**：manifest-discipline, vague-quantifier, single-purpose, template-design

**xiaolai 案例**：[../../2026-05-11-mattpocock-skills.md](../../2026-05-11-mattpocock-skills.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Matt Pocock 的个人 Claude Code skill kit，由 TypeScript 生态最知名的独立教育者之一维护。截至 xiaolai 案例撰写时（2026-05-11）拥有 **69,816 星、6,039 fork**，是 GitHub 上关注度最高的个人 skill kit（不含 Anthropic 官方）。口号是"Skills for Real Engineers. Straight from my .claude directory."。

5 个关键事实：
1. 发布于 2026-02-03，三个月内积累近 7 万星，靠 Matt 的 Twitter/YouTube 受众群放大——是"个人品牌 + 优质内容"的典型案例
2. 当前（2026-07-08）仓库可能已转为私有（GitHub 搜索返回 0 结果），但本地 shallow clone 成功，说明仍可访问（或有镜像）
3. 技能覆盖工程（TDD、triage、prototype）、效率（handoff、teach）、写作（writing-great-skills）三大领域，是典型的"个人工作流数字化"
4. `plugin.json` 自审计后经历了完全重写——新增 ask-matt、implement、domain-modeling、codebase-design、code-review、grilling、teach 等技能，旧的 zoom-out、triage（独立技能）等被替换或重命名
5. 审计后 NLPM 提交了 4 个 bug 修复 PR；xiaolai 案例标记 `status=complete`，但实地核查显示 4 个 misc 技能 bug **仍未修复**

### 1.2 架构剖析

- **目录结构**（当前 HEAD）：
  ```
  skills/
    engineering/       ← 14 个技能（包括 ask-matt, diagnosing-bugs, implement...）
    productivity/      ← 6 个技能（handoff, teach, writing-great-skills...）
    misc/              ← 4 个技能（git-guardrails-claude-code, setup-pre-commit...）
    deprecated/        ← 归档技能（不在 manifest 中）
  .claude-plugin/
    plugin.json        ← 只列出 engineering/ + productivity/ 共 20 个技能
  scripts/
    link-skills.sh
    list-skills.sh
  docs/
  package.json
  CLAUDE.md
  CONTEXT.md
  CHANGELOG.md
  ```
- **文件类型分布**：全部为 SKILL.md（约 24 个活跃技能），无 agents 或 commands
- **编排关系**：完全平列——用户在工作时手动选择调用哪个 skill，无自动路由。`link-skills.sh` 通过创建符号链接的方式安装技能（区别于 `claude plugin install`）
- **跨件契约**：部分技能（如 `grill-with-docs`）有捆绑的辅助文档（CONTEXT-FORMAT.md、ADR-FORMAT.md）。技能之间通过"推荐一起使用"的文字说明建立软连接，无代码级依赖

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**"个人工作流文档化"**——把 Matt 自己每天用的思维框架和操作步骤，原汁原味地编码为 Claude 可执行的 skill。不追求通用性，而追求"这是我实际在用的"
- **解决什么问题**：有编程背景但无 AI 助手调教经验的工程师，拿来即用地获得"Matt 日常工作流"的 AI 加持
- **做了什么 trade-off**：技能极度个人化（ask-matt、"grilling"等词汇是 Matt 自创的工作方法论），高 star 量说明这种真实性本身就是吸引力；代价是其他人可能需要理解 Matt 的工作方式才能充分利用这些 skill
- **反映什么认知模型**：作者把 Claude Code 当作"忠实执行我工作流的助理"，而非"解决通用编程问题的工具"。skill = 个人 SOP 的 AI 实现

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「个人工作流 SOP 数字化」模式**：不追求通用性，直接把作者本人的日常操作流程编码为 skill，通过个人品牌背书获得广泛传播。

模式特征清单：
- 特征 1：技能名称反映作者个人方法论（ask-matt、grilling、grill-with-docs 等，普通人不一定理解字面意思）
- 特征 2：无 agent/command 层，全部是平列 SKILL.md；用户始终是主动发起者
- 特征 3：有专属的安装脚本（link-skills.sh 创建符号链接），不依赖 `claude plugin install`
- 特征 4：CHANGELOG.md 详细记录技能演进，像产品迭代日志一样维护

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人工作流标准化（自用） | ✅ 高度适用 | 最自然的 personal skill kit 形式，无需考虑通用性 |
| 团队工具集（多人使用） | ⚠️ 谨慎 | 个人化方法论对他人可能不透明，需要额外文档 |
| 供陌生人安装的公开插件 | ⚠️ 谨慎 | 69,816 星说明这是个例外；一般个人 skill kit 受众有限 |
| 需要自动化 orchestration 的复杂流程 | ❌ 不适用 | 纯 skill 平列结构，无法自动路由 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 个人 SOP 数字化（本仓库） | mattpocock/skills | 真实性强，用户信任"这是作者本人在用的" | 个人化词汇对外人不透明；manifest 容易漂移 |
| 产品线专化（多 manifest） | hashicorp/agent-skills | 按领域精确安装 | 需要用户理解产品线结构 |
| 单职责工具集 | trailofbits/skills | 每个 skill 边界清晰 | 安全风险较高 |

### 2.4 改进空间

1. **当前问题**：4 个 misc/ 技能（git-guardrails-claude-code、setup-pre-commit、migrate-to-shoehorn、scaffold-exercises）在 plugin.json 中缺失，用户用 `claude plugin install` 安装后无法调用这些技能。**改进做法**：向 plugin.json 的 skills 数组添加 4 行。**预期收益**：消除"README 有、安装后没有"的体验落差，零成本修复
2. **当前问题**：旧技能名称被重命名（如 diagnose → diagnosing-bugs），但 CLAUDE.md 中可能仍有旧引用。**改进做法**：在 `scripts/` 中添加 `check-manifest-sync.sh`，自动对比 `skills/**/SKILL.md` 的存在情况和 `plugin.json` 中的声明。**预期收益**：下次重组时自动发现漂移
3. **当前问题**：技能里的"relevant"等模糊量词（16 处）。**改进做法**：每处将"relevant modules"替换为"modules whose name matches the error stack trace or the file being edited"等可验证描述。**预期收益**：AI 执行时行为更一致

---

## 三、过去审查发现（2026-05-11 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-05-11 当时得分 **98/100**，安全状态为 **CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude-plugin/plugin.json | 82 | 4 个 misc skill 不在 skills 数组中 |
| skills/in-progress/handoff/SKILL.md | 90 | 没有指定交接文档格式/模板 |
| skills/personal/edit-article/SKILL.md | 90 | 流程截断在第 2a 步，无完成信号 |
| skills/deprecated/qa/SKILL.md | 94 | "relevant" ×3 |
| 14 个其他 SKILL.md | 96-100 | "relevant" 出现 1-2 次 |

### 3.2 当时值得借鉴的模式

1. **全部脚本通过安全验证** → scripts/ 里的 list-skills.sh 和 link-skills.sh 无网络调用、无 eval，link-skills.sh 甚至有防循环引用的 guard → 说明即使是安装脚本也应该有防御性设计
2. **bundled reference docs** → grill-with-docs、prototype、tdd 等技能把厚重的参考文档（CONTEXT-FORMAT.md、LOGIC.md、tests.md 等）放在技能目录内 → 用户安装插件时这些文档自动一并到位，SKILL.md 保持简洁
3. **CLAUDE.md 自我约束** → CLAUDE.md 里明确写了"每个 engineering/productivity/misc 下的技能必须在 plugin.json 里有对应条目"这条规则 → 把规范写进仓库自身的文档，让后续贡献者有章可循

### 3.3 当时的缺陷

1. **manifest 自我违约** → `CLAUDE.md` 规定 misc/ 技能必须在 plugin.json 中注册，但 4 个 misc 技能均不在其中。**根本原因**：规范只写在文档里，没有对应的自动化检查；技能新增时没有同步更新 manifest 的流程约束。**自查**：我的 CLAUDE.md 或仓库规范里有没有类似"写了但没执行"的规则？
2. **"relevant"模糊量词 ×14 处** → 16 个 SKILL.md 里都有类似"trace the relevant code"、"pick the relevant skill"的表达，让 AI 需要自行判断什么是"相关的"。**根本原因**：从人类操作文档迁移过来的语言习惯；对人类读者直觉上清楚，对 AI 执行者却是模糊约束。**自查**：我写 SKILL.md 时有没有使用"relevant/appropriate/meaningful"？
3. **edit-article 流程截断** → `skills/personal/edit-article/SKILL.md` 步骤在 2a 处突然结束，没有"完成"信号，AI 不知道什么时候停止。**根本原因**：草稿没完成就发布，或写作过程中被打断后未补完。**自查**：我的 SKILL.md 每个流程都有明确的终止条件吗？

### 3.4 当时的优化机会

1. `plugin.json` skills 数组添加 4 行（git-guardrails-claude-code/setup-pre-commit/migrate-to-shoehorn/scaffold-exercises）——5 分钟可完成的修复
2. `handoff/SKILL.md` 添加具体的交接文档模板（标题 / 项目背景 / 未完成事项 / 环境变量 / 下一步）
3. 14 个技能里的"relevant"替换为可验证的具体描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 4 个 misc 技能未在 plugin.json 注册 | `cat .claude-plugin/plugin.json \| python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('skills', []))"` | **仍存在**：plugin.json 列出 20 个技能（14 engineering + 6 productivity），misc/ 下的 4 个技能仍然不在列表中 | NLPM PR 未被合并，或 PR 被拒/漏处理 |
| "relevant"模糊量词 ×14 处 | 未在 HEAD 中单独核查（skill 名称已重构，可能部分已改） | 不确定（技能已大幅重命名，无法直接对应旧文件） | — |

**plugin.json 重构情况**：

当前 plugin.json 的技能列表与审计时相比完全不同，新增了大量技能（ask-matt、implement、domain-modeling、codebase-design、code-review、grilling、handoff、teach、writing-great-skills）；部分旧技能已消失（zoom-out、triage 作为独立条目）。这是一次大型重构，但 misc/ 的 bug 在重构中依然被忽略。

### 4.2 架构演进

从审计（2026-05-11）到现在（2026-07-08），不到 2 个月的时间内：
- **重构**：engineering/ 下从 ~10 个技能扩展到 14 个，新增 ask-matt（问 Matt 本人思路）、implement（代码实现）、domain-modeling、codebase-design、code-review、research
- **restructured productivity**：handoff 和 teach 从 in-progress/ 毕业到 productivity/
- **misc/ 未动**：4 个 misc 技能仍在目录里，仍未进 plugin.json
- **意味着**：Matt 仍在积极迭代技能集内容，但 manifest 维护是他的盲区——添加新技能时会更新 manifest，但修复现有技能的 manifest 缺失则被跳过

### 4.3 新增的可学习模式

1. **「ask-matt」元技能模式**：有一个专门的技能 `ask-matt.md`，用途是让 Claude 模拟 Matt 本人的思维方式来回答工程问题。这是个有趣的设计——把"向作者提问"本身也变成技能，让用户在没有 Matt 在场时也能获得"Matt 视角"
2. **`writing-great-skills.md` 元写作技能**：技能集里有教人"如何写好技能"的技能，这是一种递归的自我文档化——用同一个工具和标准来创建新的工具

---

## 五、校准

### 5.1 我已经在做对的

1. **plugin.json 有 skills/agents 数组**：虽然 bureau 和 echo-sleuth 的 plugin.json 目前只有元数据（name/version/description），但…等等，这正是一个需要修复的问题（见下文）
2. **SKILL.md 有明确终止条件**：echo-sleuth 的每个 SKILL.md 都有"完成"步骤，不会像 edit-article 那样截断
3. **脚本安全性**：echo-sleuth 的所有脚本通过了 NLPM 安全扫描，无 eval、无 curl-pipe-bash

### 5.2 挑战 / 验证

本案例验证了一个已知但容易忘记的原则：**"写下来的规范，不被机器检查就会腐烂"**。

Matt 在 CLAUDE.md 里写了"每个 misc/ 技能必须在 plugin.json 中"，4 个技能却不在——这不是 Matt 不知道这条规则，而是没有任何机器在每次 commit 时强制执行它。

对应到 xiaolai 案例的总结："The author wrote it, then drifted from it as the kit grew."（作者写了它，然后随着代码库增长而偏离了它。）

**对我的校准**：检查我在 CLAUDE.md 或 README 里写下的所有"规范性要求"，逐条确认是否有对应的自动化检查——没有的，优先级提高为"下次修改相关文件时顺手加"。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否注册了所有 SKILL.md 和 agent 文件
python3 << 'EOF'
import json, os, glob

for pj_path in glob.glob('/tmp/my-repos/MarkQWu-*/.claude-plugin/plugin.json'):
    d = json.load(open(pj_path))
    base = os.path.dirname(os.path.dirname(pj_path))
    repo = os.path.basename(base)
    
    declared_skills = set(d.get('skills', []))
    declared_agents = set(d.get('agents', []))
    
    disk_skills = set(
        os.path.relpath(os.path.dirname(f), base)
        for f in glob.glob(os.path.join(base, 'skills/**/SKILL.md'), recursive=True)
    )
    disk_agents = set(
        os.path.relpath(f, base)
        for f in glob.glob(os.path.join(base, 'agents/*.md'))
    )
    
    if not declared_skills and disk_skills:
        print(f'[{repo}] plugin.json 无 skills 数组，但磁盘有: {disk_skills}')
    if not declared_agents and disk_agents:
        print(f'[{repo}] plugin.json 无 agents 数组，但磁盘有: {disk_agents}')
EOF
```
命中后（预期 bureau 和 echo-sleuth 均会命中）：向各自 plugin.json 添加 skills 和 agents 数组。

```bash
# 检查 CLAUDE.md 里的规范是否有对应的 CI 脚本
grep -n "must\|should\|every\|must have\|required\|必须\|要有" \
  /tmp/my-repos/MarkQWu-bureau/CLAUDE.md \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/CLAUDE.md 2>/dev/null
```
命中后：对每条规范，检查 `.github/workflows/` 里是否有对应的自动化检查；没有的列入 backlog。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 和 echo-sleuth 各自添加"manifest 完整性检查"脚本，CI 中每次 push 时运行
   - **为何可行**：mattpocock 的案例直接示范了"手动 manifest 维护会漂移"——我现在就有相同的漂移风险（bureau 和 echo-sleuth 的 plugin.json 根本没有 skills/agents 数组）
   - **第一步**：写 `scripts/check-manifest.py`，比较磁盘上所有 SKILL.md 路径和 plugin.json 里声明的路径，约 30 分钟；然后在 `.github/workflows/ci.yml` 调用它

2. **想法**：参考 mattpocock 的"ask-matt 元技能"设计，为 echo-sleuth 添加一个"ask-echo"元技能
   - **想法具体化**：当用户问"过去我遇到这类问题时是怎么处理的"，echo-sleuth 不仅返回历史记录，还能模拟"基于历史记录的我会怎么处理"
   - **第一步**：在 `skills/experience-synthesis/SKILL.md` 里增加一个"模式 → 建议"的输出阶段，从已提取的行为模式生成"如果是过去的我"的具体建议，约 1 小时

3. **想法**：模仿 CHANGELOG.md 的习惯，给 bureau 和 echo-sleuth 加版本日志
   - **为何可行**：echo-sleuth 有 `version: 0.4.0` 但无对应 CHANGELOG，用户不知道 0.4.0 vs 0.3.x 有什么变化
   - **第一步**：新建 CHANGELOG.md，从 git log 中倒推最近 5 次有意义的变更，约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 mattpocock/skills 的核心目的**：把个人工程工作流（TDD 实践、代码 triage、重构、交接等）编码为 Claude Code 技能
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 均为个人工作流工具，有 skills + plugin.json | echo-sleuth 聚焦对话挖掘，mattpocock 聚焦工程工作流 | 高 |
| MarkQWu/bureau | 高 | 均为 Claude Code 插件，有类似的技能分层 | bureau 是知识库管理，mattpocock 是工作流辅助 | 高 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| misc skills 未在 plugin.json 注册 | `cat .claude-plugin/plugin.json` | **严重命中**：bureau 的 plugin.json 无 skills/agents 数组，7 个 SKILL.md 和 1 个 agent 均未注册；echo-sleuth 的 plugin.json 同样无 skills/agents 数组，5 个 agents 和 4 个 skills 均未注册 | **高**（与 mattpocock 同类 bug） |
| "relevant"模糊量词 | `grep -n "relevant" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | 命中：experience-synthesis/SKILL.md:118、git-mining/SKILL.md:93 | 中 |
| SKILL.md 流程截断（无终止条件） | 手动查看 skills/*/SKILL.md 末尾是否有"完成"步骤 | 未命中（echo-sleuth 技能均有明确结束步骤） | 无 |

**命中后的具体行动**（这是本次最高优先级行动项）：

**bureau**（`MarkQWu/bureau/.claude-plugin/plugin.json`）：
```json
// 需要添加：
"skills": [
  "./skills/capture",
  "./skills/compile",
  "./skills/guide",
  "./skills/lint",
  "./skills/recall",
  "./skills/review",
  "./skills/scribe"
],
"agents": ["./agents/auditor.md"]
```
约 10 分钟完成，影响极大（所有技能对用户不可见的 bug）。

**echo-sleuth**（`MarkQWu/echo-sleuth-for-claude/.claude-plugin/plugin.json`）：
```json
// 需要添加：
"skills": [
  "./skills/experience-synthesis",
  "./skills/git-mining",
  "./skills/jsonl-core",
  "./skills/memory-management"
],
"agents": [
  "./agents/analyze.md",
  "./agents/file-historian.md",
  "./agents/memory-auditor.md",
  "./agents/recall.md",
  "./agents/schema-scout.md"
]
```
约 15 分钟完成，同等重要。

### 8.3 别人的更优方案

1. **领域**：技能目录结构与 CHANGELOG 结合
   - **本案例做法**：CHANGELOG.md 记录每次技能的"升级/新增/弃用"，用户可以了解技能集的演进路径（skills 目录下有 `deprecated/` 归档层）
   - **我的项目现状**：echo-sleuth 有 `version: 0.4.0` 但无 CHANGELOG；bureau 有 `version: 0.5.2` 但无 CHANGELOG。用户不知道现在和上个版本有什么不同
   - **如何借鉴**：echo-sleuth 新建 CHANGELOG.md，格式参考 mattpocock 的 CHANGELOG.md，从 git log 整理最近 5 次有意义的变化，约 20 分钟

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：plugin.json 格式完整性（相对讽刺）
- 等等——实际上我的 bureau 和 echo-sleuth 的 plugin.json 甚至比 mattpocock 更差：他们至少有 skills 数组（只是漏了 4 个 misc），而我的 plugin.json 根本没有任何 skills/agents 数组。

**修正**：在 manifest discipline 维度，我的项目比 mattpocock 做得更差（他漏了 4/30，我漏了 all/all）。本节无优势可写。

将此认知转化为优先级最高的行动项：**本次案例学习完成后，第一件事是修复 bureau 和 echo-sleuth 的 plugin.json**。

---

## 八、术语表

### <a name="manifest"></a>manifest（清单文件，plugin.json）
> Claude Code 插件的"零件清单"，告诉系统这个插件里有哪些技能（skills）、代理（agents）、命令（commands）。如果 manifest 里没有列入某个技能文件，即使文件在磁盘上存在，`claude plugin install` 之后用户也无法调用它——相当于把菜写在菜单上但厨房实际没做。

### <a name="SOP"></a>SOP（Standard Operating Procedure，标准操作规程）
> 一套明确定义的步骤，用于完成某类重复任务。本案例中 mattpocock 的每个 SKILL.md 本质上是他个人的 SOP——"当我遇到这类问题，我通常按这几步处理"被编码为 Claude 可执行的指令。

### <a name="link-skills"></a>符号链接安装（link-skills.sh）
> 一种不通过 `claude plugin install` 的安装方式：把技能文件夹以符号链接（symlink）的形式放到 `~/.claude/skills/` 下，Claude Code 在启动时会读取这个目录里的所有技能。优势是修改原始文件时立即生效（不需要重新安装）；劣势是依赖具体的文件路径，在不同机器间移植比较麻烦。

### <a name="vague-quantifier-in-skill"></a>模糊量词（skill 上下文）
> 在 SKILL.md 步骤描述中无法客观验证的修饰词，例如"relevant"（相关的）、"appropriate"（适当的）、"meaningful"（有意义的）。对人类读者来说这些词语直觉上清晰，但 Claude 在执行时需要自行判断"什么算相关"，导致行为不可预测。修复方法：用具体条件替换——"relevant modules"→"modules that appear in the error stack trace or import the file being edited"。

### <a name="deprecated-skill"></a>deprecated/（归档层）
> 本仓库中不再推荐使用、但保留以供参考的技能文件夹。技能"弃用"而非"删除"的好处：历史版本可以追溯，其他人可以看到某个功能被废弃的原因，也可以 fork 来复活某个旧版设计。类比：就像 git 里打 `deprecated` tag 而不是强制删除 commit。
