# tanweai/pua — 学习案例

**仓库**：https://github.com/tanweai/pua
**Stars**：16701 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-07-01（基于 audit 快照，目标仓库克隆受限）
**主题标签**：`experience-accumulation`, `cross-reference`, `vague-quantifier`, `security-gate`, `template-design`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
tanweai/pua（探微安全实验室出品）是一个以「正向激励」为核心设计理念的 Claude Code 插件，16701 个 star 使其跻身中文 Claude Code 生态的头部产品。关键特征：
- 规模适中，质量扎实：45 个 NL 工件，NLPM 得分 90/100，安全评级 CLEAR
- **多平台兼容性**是显著差异点：同一套 skill 同时为 Claude Code、Kimi、Hermes、CodeBuddy、Codex 发布（每平台独立目录），产生了 12 组跨平台 SKILL.md 镜像
- 有精心设计的「激励系统」：`/pua:pua-loop` 自动化循环激励、`/pua:flavor` 风格切换、`/pua:pro` 排行榜注册
- 含运行时 hook：8 个 shell hook 脚本处理会话反馈、沮丧检测、会话恢复等行为
- 4 个 command 得满分 100：`cancel-pua-loop`、`reap-orphans`、`team-status`、`teardown-all` 是标杆

### 1.2 架构剖析
- **目录结构**（推断自 audit 报告）：
```
pua/
├── agents/
│   ├── tech-lead-p9.md      # P9 技术负责人 agent
│   ├── cto-p10.md           # P10 CTO agent
│   └── senior-engineer-p7.md # P7 高级工程师 agent
├── commands/                # 15 个命令（4 个满分）
├── skills/
│   ├── pua/SKILL.md         # 核心激励 skill（中文）
│   ├── pua-en/SKILL.md      # 英文版
│   ├── pua-ja/SKILL.md      # 日文版
│   ├── p7/SKILL.md          # 职级 stub skill
│   ├── p9/SKILL.md          # 职级 stub skill
│   ├── p10/SKILL.md         # 职级 stub skill
│   ├── pro/SKILL.md         # 专业版 skill（满分 100）
│   ├── shot/SKILL.md        # 浓缩版
│   ├── mama/SKILL.md        # 妈妈激励版
│   └── pua/references/      # p7/p9/p10 协议文件
├── codex/                   # Codex 平台版本镜像
├── kimi/                    # Kimi 平台版本镜像
├── hermes/                  # Hermes 平台版本镜像
├── codebuddy/               # CodeBuddy 平台版本镜像
├── hooks/                   # 8 个 shell hook（stop-feedback、failure-detector 等）
├── scripts/setup-pua-loop.sh
└── plugin.json              # 满分 100
```
- **文件类型分布**：3 个 agent + 15 个 command + 17 个 skill（含多平台镜像）+ 8 个 hook 脚本 + 1 个 plugin manifest
- **编排关系**：command 层（如 `/pua:pua-loop`）→ 加载 skill 层（`pua:pua-loop` skill）→ 引用 hook 层（`pua-loop-hook.sh`）。三层解耦，每层职责清晰
- **跨件契约**：agent 通过 Glob 模式加载 skill（`Glob **/pua/skills/pua/SKILL.md`），但 agents/ 内部的相对路径引用协议文件存在 bug（见 §3.3）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「行为状态机」——不只是给 Claude 规则，而是根据 Claude 的情绪状态（沮丧/激励/放弃）触发不同响应。hook 是状态感知的执行层
- **解决什么问题**：Claude 在长任务中会"放弃"或"变懒"——pua 设计了全自动的激励闭环，当 hook 检测到沮丧信号时自动触发激励响应
- **做了什么 trade-off**：多平台镜像（5 个平台 × 3 个语言 = 15 组）是显著的维护成本，换取了在更广泛工具链中的兼容性
- **反映什么认知模型**：作者把 Claude 当成一个有情绪状态的「团队成员」来管理，而不是一个无状态的 API——这种「AI 拟人化管理」的设计哲学在中文开源社区很有影响力

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「行为闭环激励系统」（Behavioral Feedback Loop）**

关键特征：
- 特征 1：hook 层作为「传感器」（检测沮丧、失败信号）
- 特征 2：skill 层作为「策略库」（不同激励风格 flavor）
- 特征 3：command 层作为「控制面」（用户手动触发 /pua:flavor 切换策略）
- 特征 4：跨 session 状态持久化（instinct 积累、session restore）
- 特征 5：多平台兼容层（同一逻辑在 5 个 AI 工具中分发）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 长时间自动化任务（AI 持续运行 > 1 小时） | ✅ 高度适用 | 激励闭环能有效防止 AI 在长任务中行为退化 |
| 短对话式交互（< 10 轮） | ❌ 不适用 | hook 成本高，激励系统发挥不了作用 |
| 多平台分发（商业产品） | ✅ 适用 | 跨平台镜像模式是成熟的多平台策略 |
| 个人快速工作流搭建 | ❌ 过度工程 | 8 个 hook 脚本 + 17 个 skill 对个人使用偏重 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：行为闭环激励 | tanweai/pua | 运行时状态感知，防 AI 行为退化 | 维护成本高，hook 脚本有安全边界 |
| 备选 A：静态规则约定 | BayramAnnakov/claude-reflect | 零运行时开销 | 无法响应 AI 实时状态变化 |
| 备选 B：多层 agent 验证 | sangrokjung/claude-forge | 验证链路完整 | 无激励机制，AI 仍可能在长任务中退化 |

### 2.4 改进空间
1. **当前问题**：3 个 agent 用 `cat 同目录下的 references/xxx-protocol.md` 相对路径引用，但文件不在 agents/ 目录下 **改进做法**：改用 `Glob **/pua/skills/pua/references/p9-protocol.md`（senior-engineer-p7 已正确使用 Glob 加载 skill，只需对 protocol 文件做同样处理）**预期收益**：协议文件正确加载，agent 行为与预期一致
2. **当前问题**：17 个跨平台 SKILL.md 是手动维护的精确副本，修改 skills/pua/SKILL.md 后需同步 4 个平台 **改进做法**：用 jinja2/mustache 模板 + build script 自动生成各平台版本 **预期收益**：减少同步遗漏风险，更新时只改一处
3. **当前问题**：`/tmp/pua-plugin-root` 临时文件路径可被本地攻击者利用（race condition）**改进做法**：改用 `${CLAUDE_PLUGIN_ROOT}` 环境变量直接引用，两行代码修复

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-25 得分 **90/100**（高质量）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/tech-lead-p9.md | 70 | 零示例（-15），无 model（-5），无输出格式（-10） |
| agents/cto-p10.md | 75 | 零示例（-15），无输出格式（-10） |
| commands/survey.md | 75 | 无编号步骤（-10），无输出格式（-10），无错误路径（-5） |
| commands/cancel-pua-loop.md | 100 | 满分标杆 |
| plugin.json | 100 | 满分标杆 |
| skills/pro/SKILL.md | 100 | 满分标杆 |

### 3.2 当时值得借鉴的模式

1. **4 个满分 command 的共同特征** → 精准的单操作+完整的输出格式+明确的错误路径 → `commands/cancel-pua-loop.md`、`commands/reap-orphans.md` → 给自己的 command 加「成功时输出什么」「失败时输出什么」两个 section
2. **plugin.json 满分（完整 manifest 规范）** → 根本好处：插件可被正确安装、版本管理、权限声明 → `plugin.json` → 每次给插件新增工件时同步更新 plugin.json（而非事后补）
3. **`skills/pro/SKILL.md` 满分（完整技术写作范例）** → 同时做到：示例密集、输出格式明确、Scope 清晰、无模糊量词 → 作为自己写 skill 的参照标准
4. **跨平台镜像策略（codex/kimi/hermes/codebuddy/）** → 根本好处：同一产品覆盖更广泛的用户群 → 从多平台目录结构中学习命名约定和目录组织
5. **`skills/yes/SKILL.md` 的 scope note 缺失对比** → pua 系列 skill 都缺少 scope note，导致 R07 扣分；这提醒我 scope 不是只给自己看的，是给 AI 做上下文锚定的

### 3.3 当时的缺陷

1. **3 个 agent 的 `cat 同目录下的 references/` 相对路径引用失效** → 根本原因：作者在 skill 目录下测试时路径是对的（`skills/pua/references/` 在 skill 同目录），但 agent 放在 `agents/` 目录下时相对路径完全不同——路径上下文切换后忘记更新引用 → **自查**：我有没有 agent 引用了一个「只在 skill 目录下才有效」的相对路径？
2. **agent 零示例（tech-lead-p9、cto-p10、senior-engineer-p7 共 -15 × 3）** → 根本原因：角色型 agent 最容易跳过示例——写 "P9 技术负责人" 的定义比写具体对话例子容易得多 → **自查**：我的 gstack 里有 CEO、设计师等角色 agent，有没有缺示例？
3. **agent 缺 `model:` 声明（tech-lead-p9、senior-engineer-p7）** → 根本原因：相比 skill，agent 的 model 字段更重要——P9 级别的编排 agent 需要推理能力，haiku 不够用，但未声明就会随机降级 → **自查**：我的 role agent 有没有指定 model？
4. **`/tmp/pua-plugin-root` race condition（Medium）** → 根本原因：使用了可预测路径的临时文件作为状态传递媒介，而非 `$CLAUDE_PLUGIN_ROOT` 环境变量 → **自查**：我的 hook 脚本有没有用 /tmp 临时文件做路径传递？
5. **PII 收集无透明度（邮件+电话发送到第三方端点）** → 根本原因：排行榜功能需要身份信息，但收集时没有充分告知保留政策 → 学习：任何收集用户信息的 skill 都要在 UI 里明确告知「存哪里、用于什么、怎么删除」

### 3.4 当时的优化机会

1. **给 3 个 agent 加 `<example>` 块**：每个各 2 个示例，大幅提升触发可靠性（从 70/75 → 90+）
2. **给 tech-lead-p9 和 senior-engineer-p7 加 `model:`**：分别加 `model: sonnet` 和 `model: sonnet`（P9 编排）
3. **把 agent 协议文件引用改为 Glob 模式**：三行修改，消除静默失败风险

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
> 注：git clone 受限，以下基于推断。

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个 agent 协议文件相对路径错误 | `grep "同目录下" agents/*.md` | **无法验证** | 16701 star 的活跃仓库，PR 修复可能性高 |
| agent 零示例 | `grep -L "<example>" agents/*.md` | **无法验证** | -15 × 3 = -45 分的损失足以驱动作者修复 |
| `/tmp/pua-plugin-root` race condition | `grep "pua-plugin-root" hooks/stop-feedback.sh` | **无法验证** | CLEAR 安全评级后可能已有社区 PR |

### 4.2 架构演进
audit 时 45 个工件，跨平台 × 3 语言的矩阵结构显示作者有商业化思维。16701 star 表明产品在快速增长期，预期演进方向：增加更多 flavor（激励风格），可能增加新职级（P6/P8），以及优化 hook 的平台兼容性。

### 4.3 新增的可学习模式
暂无（无法访问当前仓库状态）。

---

## 五、校准

### 5.1 我已经在做对的
1. 在 agent 中用 Glob 而非相对路径引用外部文件
2. 给角色型 agent 声明 `model:` 字段（避免随机降级）
3. 在 hook 脚本中使用环境变量而非 /tmp 临时文件做状态传递
4. 为 command 提供完整输出格式和错误路径声明
5. plugin.json 与工件同步更新（manifest 纪律）

### 5.2 挑战 / 验证
- **挑战的假设**：我之前认为「示例越多越好，越通用越好」——但 pua 的最佳实践是「从满分 command 学模式」。cancel-pua-loop 满分的原因是：单操作、完整格式、无歧义。不是「示例多」，而是「范围收窄、职责单一」。
- **新认知**：跨平台多语言镜像是成熟商业产品的必然选择，但需要配合 build script 自动同步，否则维护成本会在版本迭代中急速累积。这是从「个人项目」升级到「产品」的关键转型点。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查所有 agent 中是否有「相对路径」引用非 agent 目录的文件
grep -rn "同目录下\|\.\./" ~/.claude/agents/*.md 2>/dev/null
# 命中后：改用 Glob 模式，如 Glob **/skills/<skill-name>/SKILL.md

# 检查角色型 agent 是否声明了 model:
grep -rL "^model:" ~/.claude/agents/*.md 2>/dev/null
# 命中后：根据 agent 职责选择：orchestration → sonnet，execution → haiku 可选

# 检查 hook 脚本中是否有用 /tmp 做路径传递的写法
grep -rn "/tmp/" ~/.claude/hooks/*.sh 2>/dev/null | grep -v "#"
# 命中后：改用 ${CLAUDE_PLUGIN_ROOT} 或 ${HOME}/.claude/state/ 等可控路径

# 检查 agent 中零示例情况
grep -rL "<example>\|## Example\|## 示例" ~/.claude/agents/*.md 2>/dev/null
# 命中后：为每个命中 agent 加 2 个具体对话示例（输入→触发→输出格式）
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 项目中引入「behavior feedback hook」——当 AI 在 knowledge capture 阶段停滞（如连续 3 次空输出）时，hook 触发提示"请总结当前状态并继续"
   - **为何可行**：bureau 有长任务（会话知识提取），非常适合行为闭环模式
   - **第一步**：在 bureau hooks 目录新建 `stall-detector.sh`，检测连续空 Bash 输出，触发恢复提示，参考 pua 的 `failure-detector.sh` 逻辑，30 分钟初稿

2. **想法**：在 gstack 的 CEO/DesignManager agent 中加入 `<example>` 块，借鉴 pua 角色 agent 的教训
   - **为何可行**：gstack 的角色 agent 越具体，触发准确率越高
   - **第一步**：找 gstack 中使用最频繁的一个角色 agent（如 EM），写一个「具体任务 → agent 响应格式」的示例，20 分钟完成

3. **想法**：为 graphify skill 增加 Scope Note（第一行说明适用范围）
   - **为何可行**：graphify 有多个 skill，没有 scope note 会导致 AI 在不相关场景下也加载它
   - **第一步**：在每个 SKILL.md 第一段加一句「仅在用户明确要求分析代码图谱关系时激活」，5 分钟批量完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **tanweai/pua 的核心目的**：AI 行为持久化与激励系统——让 Claude 在长任务中保持动力，跨 session 积累工作模式

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为跨 session 积累/挖掘机制，关注 AI 历史行为 | echo-sleuth 挖掘过去决策，pua 激励当前行为 | 高 |
| MarkQWu/bureau | 中 | 同为持久化会话状态的 Claude Code 插件 | bureau 聚焦知识结构化，pua 聚焦行为激励 | 中 |
| MarkQWu/gstack | 中 | 同为多角色 Claude Code 套件 | gstack 用角色模拟决策，pua 用激励调控行为 | 中 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 我的项目推测情况 | 严重度 |
|---|---|---|
| agent 相对路径引用外部文件（静默失败） | gstack 的角色 agent 若引用其他 skill 则需排查路径 | 高 |
| 角色型 agent 零示例 | gstack CEO/EM/Designer agent 可能缺示例（角色 agent 通病） | 高 |
| hook 脚本用 /tmp 临时文件 | echo-sleuth 若有 hook 则需排查 | 中 |
| skill 无 Scope Note | graphify 和 drama-workshop-skills 的 skill 可能无触发范围声明 | 中 |

**命中后的具体行动建议**：
- gstack：`grep -n "Glob\|cat " .claude/agents/*.md | grep -v "Glob \*\*"` 找出所有非 Glob 的文件引用，每个 5 分钟改为 Glob 模式
- echo-sleuth：`grep -rL "<example>\|## Example" .claude/agents/*.md` 找零示例 agent，优先处理触发频次高的

### 7.3 别人的更优方案

1. **领域**：多 Flavor（激励风格）的动态切换
   - **pua 做法**：`/pua:flavor` command 让用户在运行时切换激励风格（温柔版/严格版/妈妈版），通过修改本地 config 文件实现，不需要重新安装
   - **我的项目现状**：gstack 的角色是静态的（CEO 就是 CEO），用户无法在 session 中动态切换工作风格
   - **如何借鉴**：在 gstack 中添加 `/gstack:mode` command，允许用户在 `focus`（专注模式）/ `explore`（探索模式）/ `review`（审查模式）之间切换，修改 `.claude/gstack-config.json`，20-30 分钟可实现基础版

2. **领域**：Plugin Manifest（plugin.json）的规范完整性
   - **pua 做法**：`plugin.json` 满分 100，版本号、作者、描述、工件列表全部完整声明，安装时零歧义
   - **我的项目现状**：bureau 和 echo-sleuth 若 plugin.json 有遗漏字段则安装体验会有问题
   - **如何借鉴**：用 `npx claude-plugin validate plugin.json`（如有此工具）或手动对照 pua 的 plugin.json 检查每个字段，15 分钟完成

### 7.4 反向：我的项目做得比他们好的地方

若 echo-sleuth-for-claude 的 agent 已正确使用 Glob 模式加载 skill，则在「文件引用路径安全性」上优于 pua——pua 的 3 个 role agent 全部使用了会失败的相对路径引用协议文件。这是一个具体可量化的优势点：跨目录文件引用要用 Glob，不用假设当前工作目录。

---

## 八、术语表

### <a name="pua-loop"></a>pua-loop
> pua 插件的核心命令，全称 Positive Unlimited Acceleration Loop。启动后自动以激励语言持续给 Claude 正向反馈，帮助 Claude 在长时间任务中保持「激励状态」而不退化为消极/偷懒行为。背后由 `pua-loop-hook.sh` 实现状态检测。

### <a name="flavor"></a>Flavor（激励风格）
> pua 中的一个可切换参数，控制激励语言的风格。通过 `/pua:flavor` command 在运行时切换，配置保存在本地 JSON 文件。类似于为 Claude 设置「工作语气」——严格导师、温柔教练、妈妈鼓励等不同角色。

### <a name="behavior-feedback-loop"></a>行为反馈闭环
> pua 的核心架构模式。hook 层检测 Claude 的行为信号（如沮丧词汇、停滞、失败），触发 skill 层的激励策略，激励策略通过 command 层暴露给用户控制。三层形成一个持续运行的反馈闭环，使 AI 的工作状态可被动态调控。

### <a name="Glob-pattern"></a>Glob 模式（文件引用）
> 一种路径匹配写法，如 `Glob **/pua/skills/pua/SKILL.md`。在 Claude Code 的 agent 中，用 Glob 引用文件比相对路径（如 `./references/file.md`）更安全——相对路径依赖于当前工作目录，而 Glob 在整个仓库中搜索，不受执行上下文影响。

### <a name="race-condition"></a>Race Condition（竞态条件）
> 并发场景下的 bug 类型。pua 的 `/tmp/pua-plugin-root` 问题是竞态条件的一个例子：脚本写入 /tmp 文件和读取该文件之间存在窗口期，恶意用户可以在这个窗口内替换文件内容，导致执行了意外的脚本路径。正确做法：用不可预测的路径（如 `${CLAUDE_PLUGIN_ROOT}`）传递状态。

### <a name="scope-note"></a>Scope Note（触发范围声明）
> SKILL.md 中的一段说明，告诉 AI（和 NLPM 评分器）「这个 skill 在什么情况下应该被激活，什么情况下不应该」。缺少 scope note 的 skill 可能在不相关的场景中被错误加载，导致 AI 行为漂移。R07 规则要求所有 skill 都有 scope note。
