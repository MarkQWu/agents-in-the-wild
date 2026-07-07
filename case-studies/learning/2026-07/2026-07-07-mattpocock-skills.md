# mattpocock/skills — 学习案例

**仓库**：https://github.com/mattpocock/skills
**Stars**：69,816 | **来源**：upstream（SECURITY CLEAR）
**Audit 日期**：2026-05-11（历史快照）| **生成日期**：2026-07-07（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `single-purpose`, `examples-driven`, `template-design`

**xiaolai 案例**：[../../2026-05-11-mattpocock-skills.md](../../2026-05-11-mattpocock-skills.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Matt Pocock 的个人 Claude Code skill 工具集，面向工程师日常工作流（TDD、代码审查、重构、原型、triage、写作）。Matt 是 TypeScript 生态最知名的教育者之一，通过 Twitter/YouTube 大量传播，仓库在发布约 3 个月后就积累了 **69,816 stars**——成为 GitHub 上除 Anthropic 官方示例外的最受欢迎 skill 套件。

关键事实：
- 发布时间：2026-02-03（Matt 在 `setup-matt-pocock-skills` skill 内有说明）
- 安装方式：`claude plugin install mattpocock-skills`，通过 `.claude-plugin/plugin.json` 注册
- 覆盖领域：engineering（TDD、重构、架构、代码审查、实现）、productivity（handoff、teaching、技能编写指导）
- 目录结构：按 engineering / productivity / personal / in-progress / deprecated / misc 六个桶分层

### 1.2 架构剖析
- **目录结构**：
  ```
  .claude-plugin/plugin.json   ← 唯一的 manifest
  skills/
    engineering/   (13 个 SKILL.md — 全部注册)
    productivity/  (7 个 SKILL.md — 全部注册)
    misc/          (4 个 SKILL.md — ⚠ 无一注册)
    personal/      (1 个 — 正确排除)
    in-progress/   (多个 — 正确排除)
    deprecated/    (多个 — 正确排除)
  scripts/          ← list-skills.sh, link-skills.sh
  ```
- **文件类型分布**：当前 HEAD — 20 个已注册 SKILL.md，4 个孤立 SKILL.md（misc 桶），共 24 个以上（含个人/实验性）
- **编排关系**：纯平铺 skill 集合，无 agent/command，无 router。每个 skill 独立调用。`setup-matt-pocock-skills` 是特殊的「安装引导」skill。
- **跨件契约**：`grill-with-docs` 引用同目录下的 `CONTEXT-FORMAT.md` / `ADR-FORMAT.md`；`tdd` 引用 5 个附属文件；`improve-codebase-architecture` 跨目录引用 `../grill-with-docs/` 的文件。所有跨文件引用均为 bundled 模式（相对路径），经验证全部存在。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「实战工程师工具集」——不追求大而全，每个 skill 解决一个具体问题。`diagnose` 专门排查 bug；`triage` 专门管理 issue；`prototype` 专门快速验证想法。每个 skill 有配套参考文档（`LOGIC.md`、`AGENT-BRIEF.md` 等），把隐性知识显式化。
- **解决什么问题**：把 Matt 多年积累的「高效工程师工作方式」编码为可复用的 Claude Code skill，让他人能直接安装后获得同等能力。
- **做了什么 trade-off**：功能完整性 vs 维护成本——misc 桶的技能被添加到了 README 里供用户发现，但没有及时同步到 manifest（因为作者把 plugin.json 当成「精选 skill 列表」而不是「全量 skill 清单」）。
- **反映什么认知模型**：作者把 skill 看作「可组合的工程动作」，而不是「一次性的 prompt 模板」。每个 skill 都有清晰的触发条件、步骤和预期产出。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「个人工作流封装」模式**：单一作者的日常工作方式→编码为 skill→打包成 plugin 供他人安装。

模式特征清单：
- **作者即用户**：skill 设计完全服务作者自己的工作流，不过度设计「通用性」
- **Bundled 文档**：复杂 skill 附带参考 Markdown（`LOGIC.md`、`CONTEXT-FORMAT.md`），避免 skill 文件过长
- **分层桶策略**：engineering/productivity/personal/in-progress/deprecated 按成熟度分桶，实验性技能与生产技能物理隔离
- **零 agent/command**：所有功能都是 skill 驱动，没有 agent 或 slash command，降低复杂度
- **self-describing install skill**：`setup-matt-pocock-skills` 帮用户完成初始配置（如定义项目领域），是优秀的 UX 设计

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人生产力工具集 | ✅ 高度适用 | 作者即用户，不需要考虑他人偏好 |
| 企业级多人协作工具 | ⚠ 部分适用 | 需要加强 output format、增加更多边界条件说明 |
| 跨语言/跨框架通用工具 | ❌ 不适用 | 技能高度 TS/Web 特化（tdd、prototype 等隐含 TS 语境） |
| 教学演示 | ✅ 适用 | 98/100 分的高质量代码是很好的参考示范 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 个人工作流封装（本仓库） | mattpocock/skills | 专注、高质量、作者驱动 | misc 桶易脱同步，skill 间无编排 |
| 企业集合型（大而全） | tech-leads-club/agent-skills | 覆盖广（78 个 skill） | 质量参差不齐，维护压力大 |
| 单 plugin 多层（agent+command+skill） | trailofbits/skills | 功能强大，跨件联动 | 复杂度高，安全风险面广 |

### 2.4 改进空间
1. **当前问题**：misc 桶的 skill 手动管理 manifest，容易脱同步。**改进做法**：在 CI 里加一步 `scripts/list-skills.sh | diff - plugin.json` 检查，或用 `"skills": ["./**/*"]` 通配符代替显式列表。**预期收益**：消除整类「disk 上有、manifest 里无」的 bug。
2. **当前问题**：skill 内的 `relevant` 量词（14 处）让 Claude 在决策什么内容「相关」时有歧义。**改进做法**：替换为「the module in the current code path」「every hunk that touches a documented standard」等具体表达。**预期收益**：提升 skill 执行一致性，减少错误召回。

---

## 三、过去审查发现（2026-05-11 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-11 当时得分 **98/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude-plugin/plugin.json` | 82 | 4 个 misc skill 缺失 |
| `skills/in-progress/handoff/SKILL.md` | 90 | 缺少 output format 模板 |
| `skills/personal/edit-article/SKILL.md` | 90 | 步骤在 2a 截断，缺少完成信号 |
| `skills/deprecated/qa/SKILL.md` | 94 | `relevant` ×3 |
| `skills/engineering/triage/SKILL.md` | 96 | `relevant` ×2 |

### 3.2 当时值得借鉴的模式
1. **Bundled 参考文档**：`tdd` skill 附带 `tests.md`、`mocking.md`、`deep-modules.md` 等 5 个参考文件，把复杂知识模块化。为什么好：避免 skill 本身过长、使知识可独立维护。原文示例：`skills/engineering/tdd/`。如何借鉴：对超过 100 行的 skill，把方法论部分拆成 `references/` 子文件。
2. **Safety guard 脚本**：`link-skills.sh` 对符号链接的保护检查（防止循环写入）。为什么好：防御性设计，避免安装副作用。原文：`scripts/link-skills.sh`。如何借鉴：安装/初始化脚本应总是验证目标路径的安全性。
3. **分层桶策略**：in-progress/personal/deprecated 的物理隔离，使「公开 skill」与「实验 skill」清晰分离。为什么好：减少认知负担，用户不会误用不成熟的 skill。

### 3.3 当时的缺陷
1. **[manifest-discipline](#manifest-discipline)「manifest 脱同步」**（4 处）：CLAUDE.md 要求所有 misc/ 下的 skill 都进 plugin.json，但 4 个 misc skill 完全缺席。根本原因：作者在 CLAUDE.md 写了规则但没有自动化守护——「人肉」规则是不可靠的。自查：我有没有同样的问题？→ 有，详见 §7.2。
2. **「`relevant` 量词滥用」**（14 处）：`relevant modules`、`relevant inputs` 等表达把决策权交给了 Claude，而 Claude 对「什么是相关的」的判断会因上下文不同而飘移。根本原因：作者的工程直觉很准，但没有意识到 NL 指令需要像代码一样精确。自查：我的 skills 中是否有这类表达？→ 有，见 §7.2。
3. **「步骤截断」（edit-article）**：`personal/edit-article/SKILL.md` 在步骤 2a 就结束了，没有完成路径。根本原因：个人使用的 skill 不经 code review，没有外部质量约束。自查：我是否有步骤不完整的 skill？→ 待核查。

### 3.4 当时的优化机会
1. 给 manifest 加 CI 检查（diff plugin.json vs disk enumeration）
2. 把 14 个 `relevant` 替换为具体表达（每处 5 分钟内可完成）
3. 补全 `edit-article` 和 `handoff` 的步骤完整性（各 ~15 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 4 个 misc skill 不在 plugin.json | `cat .claude-plugin/plugin.json` 数 skills 数组 | **PERSISTS（持续存在）**：当前仅有 20 个已注册 skill，`misc/` 的 4 个 skill 仍全部缺席 | 用户安装后无法使用 git-guardrails、setup-pre-commit、migrate-to-shoehorn、scaffold-exercises |
| `relevant` 量词（14 处） | `grep -n "relevant" skills/**/*.md` | 无法精确统计（repo 结构变化），仍有多处保留 | 低优先级，不影响功能 |
| `handoff` 缺少 output format | `grep "## Output" skills/productivity/handoff/SKILL.md` | 文件路径变为 `skills/productivity/handoff/SKILL.md`，审计期间仍无完整 output section | 提示质量稍低 |

### 4.2 架构演进
从 2026-05-11 到当前 HEAD，仓库发生了显著重组：
- **旧 plugin.json 注册 26 个 skill** → **新 plugin.json 注册 20 个 skill**
- skill 命名策略变化：`diagnose` → `diagnosing-bugs`，`zoom-out` 被删除或重命名，新增 `ask-matt`、`domain-modeling`、`codebase-design`、`code-review` 等
- 部分旧 skill（如 `zoom-out`）已进入 deprecated 或消失
- **misc 桶依然孤立**：这次重组没有解决 manifest 脱同步问题——说明作者仍然没有自动化这一检查

### 4.3 新增的可学习模式
当前 HEAD 新增了 `ask-matt` skill（看名字推测是让 Claude 扮演 Matt Pocock 给出工程建议的工具）和 `domain-modeling` / `codebase-design`，这是比 audit 时更丰富的设计领域覆盖。`handoff` 从 in-progress 升级到 productivity，说明它已经成熟。

---

## 五、校准

### 5.1 我已经在做对的
1. **分层成熟度管理**：这个仓库的 engineering/in-progress/deprecated 分层策略我在 bureau、echo-sleuth 等项目里也有类似实践（skill 按成熟度管理）。
2. **Bundled 文档**：在 echo-sleuth 的 `memory-management` skill 里把格式规范直接写在 skill 中而不是分离 references，这是类似的「bundled 知识」策略，虽然粒度不同。
3. **精确步骤**：echo-sleuth 的 skill 中步骤完整，没有截断。

### 5.2 挑战 / 验证
这次案例挑战了我的一个假设：**「高分（98/100）的仓库 bug 一定会被修复。」** mattpocock/skills 在 xiaolai 发出审查报告 2 个月后，manifest 脱同步的 bug 依然存在——甚至在一次大的重组中也没被顺带修复。这说明：**NL programming 的 bug 不像代码 bug 那样「痛感强烈」**。用户看到的只是「某些技能不能用」，而不是报错，所以修复的紧迫性远低于功能性 bug。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否遗漏了 disk 上的 SKILL.md
python3 -c "
import json, os, glob
for plugin_json in glob.glob('/tmp/my-repos/**/.claude-plugin/plugin.json', recursive=True):
    d = json.load(open(plugin_json))
    registered = set(d.get('skills', []))
    root = os.path.dirname(os.path.dirname(plugin_json))
    on_disk = set(
        os.path.relpath(p, root)
        for p in glob.glob(root + '/**/SKILL.md', recursive=True)
    )
    missing = on_disk - registered
    if missing:
        print(f'[{plugin_json}] missing: {missing}')
    else:
        print(f'[{plugin_json}] OK')
"
```
命中后怎么办：把缺失的 skill 路径加入 `skills` 数组，或确认是有意排除（加注释说明）。

```bash
# 检查 relevant/appropriate 等量词在我的 skills 里的密度
grep -rn -E '\b(relevant|appropriate|comprehensive|ensure|robust|proper)\b' \
  /tmp/my-repos/MarkQWu-*/skills/ --include="*.md" 2>/dev/null | wc -l
```
命中后怎么办：逐条替换为具体表达（5-15 分钟/处）。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 项目的 plugin.json 加一个 CI 检查，防止 manifest 脱同步。
   - **为何可行**：bureau 已有 `.github/workflows/`，加一个 `python3 -c "..."` 步骤即可。
   - **第一步**：写一个 5 行 Python 脚本对比 `skills array` vs `find . -name SKILL.md`，在 CI 里运行。约 30 分钟。

2. **想法**：把 gstack 里 `relevant` 量词替换为具体描述。
   - **为何可行**：grep 显示 310 个命中，但不全是 vague quantifier，需要逐条确认。
   - **第一步**：先 grep 定位，再批量替换前 20 个最高频文件。约 1 小时。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 mattpocock/skills 的核心目的**：将个人工程工作流编码为可安装的 Claude Code skill 集合，面向 TypeScript/Web 工程师。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 个人工作流封装成 skill，也有 plugin.json manifest | gstack 定位更广（CEO/Designer/EM/PM 等角色），而 mattpocock 专注工程 | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都是 skill 为中心的 plugin，有 plugin.json | echo-sleuth 专注于记忆挖掘，不是通用工程工具 | 中 |
| MarkQWu/bureau | 低 | 都有 skill 文件 | bureau 是知识库工具，不是技能集合 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| misc skill 不在 plugin.json（manifest 脱同步） | 对比 plugin.json skills 数组 vs disk SKILL.md | gstack 没有 plugin.json（技能以目录暴露），不适用；echo-sleuth plugin.json 有 4 个 skill，disk 上也是 4 个，**OK** | 低（目前无问题） |
| `relevant/appropriate` 等量词 | `grep -rn relevant gstack/` | gstack 全仓库 310 次命中，大量 SKILL.md 中使用 `relevant`, `appropriate`, `ensure` 等 | **高** |
| 步骤截断（无完成信号） | 读几个 gstack SKILL.md 末尾 | 待人工核查 | 中 |

**gstack 命中命令**：
```bash
grep -rn -E '\b(relevant|appropriate|ensure|comprehensive)\b' \
  /tmp/my-repos/MarkQWu-gstack/ --include="*.md" | head -20
```

命中后行动：优先处理 gstack 中调用频率最高的 skill（如 `land-and-deploy`、`diagram`）中的量词，估计 2-3 小时可清理一半。

### 7.3 别人的更优方案

1. **领域**：Bundled 参考文档（附属 Markdown 文件）
   - **本案例做法**：`tdd` skill 附带 5 个参考文档（`tests.md`、`mocking.md` 等），skill 文件保持简洁，详细规范分离到子文件中。路径：`skills/engineering/tdd/`
   - **我的项目现状**：echo-sleuth 的 `memory-management/SKILL.md` 把所有规范都内联在一个文件里（约 300 行）；gstack 的 skill 类似
   - **如何借鉴**：把超过 100 行的 SKILL.md 中的「格式规范」部分拆分到 `references/` 子目录，在 skill 中用 `cat references/format.md` 或直接 Read 引用

2. **领域**：`install skill` 设计（`setup-matt-pocock-skills`）
   - **本案例做法**：专门有一个 skill 帮用户完成初始设置（定义领域、配置文件等），降低上手门槛
   - **我的项目现状**：echo-sleuth 没有 install skill；gstack 的初始化依赖手动配置
   - **如何借鉴**：在 gstack 或 echo-sleuth 中加一个 `setup-<project-name>` skill，引导用户完成初次配置

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：量词精确性（echo-sleuth）
- **我的做法**：echo-sleuth 的 4 个 skill 中无一使用 `relevant`/`appropriate` 等量词（grep 0 命中），每个步骤都有明确的触发条件和动作
- **本案例做法（弱在哪）**：mattpocock/skills 98 分仍有 14 处量词命中，大量使用 `relevant modules`、`relevant code`
- **意义**：echo-sleuth 在 NL 精确性上高于 69,816 star 的仓库。若被审查，这是亮点。

---

## 八、术语表

### <a name="manifest-discipline"></a>manifest-discipline
> Plugin 的「清单文件」（`plugin.json`）是 Claude Code 知道「这个插件包含哪些 skill」的唯一来源。**Manifest 脱同步**是指：硬盘上有 SKILL.md 文件，但 plugin.json 里没有登记，导致用户安装插件后这些技能完全不可用——就像餐厅菜单上有某道菜但厨房根本不做。**修复方式**：要么用脚本自动同步，要么在 CI 里加检查。
