# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-08-02（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

agiletec-inc/airis-mcp-gateway 是一个按需启动的本地 [MCP](#mcp) 网关，当前仅维护 Context7 进程服务器。它不是持续驻留的守护进程，而是在被调用时才激活，用完后退出，以此降低空闲资源消耗。151 Star，由 agiletec-inc 公司发布。

关键事实：
1. **定位**：运行时基础设施而非 NL 插件——用户获得的是一个 MCP 协议路由中间层，不是知识共享工具集
2. **技术栈**：Rust/TypeScript API 层 + 极薄的 Claude Code NL 层（3 个操作命令 + 2 个 AI 引导命令）
3. **文档语言**：README 和 CLAUDE.md 均以日语书写，目标受众为日语开发者社区
4. **安全状态**：BLOCKED（3 个 Critical 级 [curl|bash](#curl-pipe-bash) 安全发现）
5. **架构演进**：原有的 skills 子包已在当前 HEAD 中完全移除，NL 层比 audit 时更薄

### 1.2 架构剖析

**目录结构（当前 HEAD）**：
```
agiletec-inc/airis-mcp-gateway/
  .claude/
    commands/
      status.md          # 97/100：查看网关运行状态
      test.md            # 96/100：运行测试套件
      troubleshoot.md    # 90/100：故障排查入口
  config/
    bootstrap/
      assets/claude/
        commands/
          airis-capability-router.md   # 98/100：AI 能力路由器
          airis-research-first.md      # 78/100：研究优先预检策略
  CLAUDE.md              # 98/100，日语，定义严格架构边界
  install.sh             # CRITICAL：curl|bash 安装链
  scripts/
    quick-install.sh     # CRITICAL：同类安装模式
    airis-gateway        # HIGH/MEDIUM：下载执行 + shell 变量注入风险
    airis_bootstrap.py
```

**文件类型分布（当前 HEAD）**：5 个 command，0 个 skill，0 个 agent，0 个 hook，1 个 CLAUDE.md

**编排关系**：两个目录的命令面向不同受众。`.claude/commands/` 是**人工运维命令**（status/test/troubleshoot），供开发者管理本地网关进程；`config/bootstrap/assets/claude/commands/` 是**AI 引导命令**（capability-router/research-first），供 AI agent 在使用网关前做路由和预检。两层命令职责不混。

**跨件契约**：`airis-capability-router.md` 读取当前网关能力后把请求路由到正确处理流程；`airis-research-first.md` 在 AI 研究任务前执行预检。CLAUDE.md 以日语写明全局边界：哪些文件是"手写主文件"（不可添加生成标记），这是整个项目 NL 层的元契约。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：NL 层极小化 + 严格边界护城河。把 MCP 协议路由逻辑完全放在 Rust/TypeScript 运行时，Claude Code 命令只负责"观察和操控"，不参与业务路由决策
- **解决什么问题**：MCP 生态中各类服务器通常长驻内存浪费资源。按需启动机制让 MCP 进程只在被调用时激活，降低空闲成本
- **做了什么 trade-off**：
  - 按需启动 vs 常驻：首次调用有启动延迟，但资源利用率更高
  - NL 层极薄 vs 知识共享：选择把核心逻辑放在运行时，代价是 skill 层无处安放，最终选择移除
  - 日语文档 vs 国际化：聚焦日语社区降低受众规模，换来文档表达密度更高
- **反映什么认知模型**：作者把 Claude Code 视为"运维终端"而非"业务执行者"——AI 只是观察和操控服务，不承担路由判断。这和把 Claude 当成代码执行主体的设计哲学形成鲜明对比

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「MCP 接入层 + 严格边界护城河」**

Claude Code 命令只是进入运行时基础设施的操作界面，核心调度逻辑不暴露给 NL 层，CLAUDE.md 以精确边界文档作为架构护城河。技能（skill）层已完全退出，NL 层极度精简。

模式特征清单（4 条）：
- 特征 1：命令文件是"观察/操控"界面，不是业务决策者——所有判断逻辑在运行时代码里
- 特征 2：skills 子包已完全移除，NL 层收敛至仅"命令 + CLAUDE.md"
- 特征 3：CLAUDE.md 以最少文字写明"哪些文件是手写主文件"的硬边界，约束优先于功能描述
- 特征 4：两层命令目录面向不同用户（人工运维 vs 协作 AI），职责不混、互不侵犯

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 本地服务管理（启动/状态/测试） | ✅ 高度适用 | 命令层职责清晰，只需观察和操控 |
| MCP 中间件 / 网关场景 | ✅ 适用 | NL 层薄，核心逻辑在运行时，职责分离自然 |
| 需要向 AI 用户共享领域知识的工具 | ❌ 不适用 | skill 层已移除，NL 层无法承载知识传递 |
| 需要复杂 AI 推理参与的业务流程 | ❌ 不适用 | AI 介入点太少，不适合 skill/agent 驱动的流程 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 接入层 + 严格边界（本仓库） | airis-mcp-gateway | 职责清晰，运维高效，边界坚固 | NL 知识层缺失，技能可扩展性差 |
| NL 表皮 + 原生二进制核心 | claude-notifications-go | 功能层高性能 + NL 操控层兼得 | 安装链安全风险同样存在，debug 困难 |
| 全 skill 平铺型 | MiniMax-AI/skills | NL 知识共享能力强，可发现性好 | 缺乏架构边界，skill 膨胀后维护成本高 |

### 2.4 改进空间

1. **当前问题**：install.sh 文档化和实现均使用 curl|bash，无完整性验证。**改进做法**：在 release 时同步发布 SHA-256 校验文件，install.sh 下载后先 `sha256sum -c` 验证再执行；或迁移到包管理器分发（Homebrew Tap）。**预期收益**：消除 3 个 Critical 安全发现，解除 BLOCKED 状态，开放 PR 提交通道。

2. **当前问题**：`airis-research-first.md` 以无序列表展示 4 条多步工作流规则，无输出格式声明。**改进做法**：把"Rules:"下的 `- ` 改成 `1. 2. 3. 4.` 编号列表；增加 `## 输出格式` 节说明预检完成后产出什么形式的结果。**预期收益**：命令得分从 78 提升至 95+，AI 执行顺序可预测。

3. **当前问题**：`troubleshoot.md` 使用 `$ARGUMENTS` 但无空输入分支，空调用时"Issue Type: "留白无引导。**改进做法**：在命令开头加空值检测：当 `$ARGUMENTS` 为空时，列出常见 issue 类型（如 gateway-not-starting / mcp-timeout / context7-error），供用户选择后重新调用。**预期收益**：命令健壮性从 90 提升至 100。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-26 当时得分 **90/100**，Security 状态：**BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `config/.../airis-research-first.md` | 78 | 无序列表规则（-10）+ 无输出格式节（-10）+ 模糊"prefer"（-2） |
| `skills/skills/mcp-database/SKILL.md` | 83 | 无示例（-15）+ 模糊"appropriate caution"（-2） |
| `skills/skills/mcp-debugging/SKILL.md` | 85 | 无示例（-15） |
| `skills/skills/mcp-implementation/SKILL.md` | 85 | 无示例（-15） |
| `skills/skills/mcp-research/SKILL.md` | 85 | 无示例（-15） |
| `.claude/commands/troubleshoot.md` | 90 | 无空 $ARGUMENTS 处理（-10） |
| `.claude/commands/test.md` | 96 | 模糊"some servers"（-2）+ 模糊"actionable"（-2） |
| `.claude/commands/status.md` | 97 | `Bash(curl*)` 声明但命令体未使用（-3） |
| `config/.../airis-capability-router.md` | 98 | 模糊"briefly"（-2） |
| `CLAUDE.md` | 98 | 模糊"significant tokens"（-2） |
| `skills/.claude-plugin/plugin.json` | 100 | — |

### 3.2 当时值得借鉴的模式

1. **[plugin.json](#manifest) 满分（100/100）**（[manifest-discipline](#manifest-discipline)）→ 为什么好：所有命令路径注册完整，无孤儿文件，无遗漏声明。[manifest](#manifest) 是 Claude Code 插件的"唯一真相源"，完整的 manifest 从根本上避免了"文件在磁盘但用户不可见"的静默 bug → 原文路径：`skills/.claude-plugin/plugin.json` → 如何借鉴：每次增加或删除 command/skill 文件时，立即同步更新 plugin.json；可在 pre-commit hook 中加 manifest 完整性验证

2. **allowed-tools 精确声明**（[single-purpose](#single-purpose)）→ 为什么好：status.md 仅因 `Bash(curl*)` 声明但命令体未调用 curl 扣了 3 分；其他命令的 allowed-tools 声明与实际调用高度一致，说明作者有精确声明工具的意识 → 原文路径：`.claude/commands/*.md` → 如何借鉴：每个命令只声明命令体会实际调用的工具，声明与使用保持一一对应

3. **CLAUDE.md 约束优先写法**（[single-purpose](#single-purpose)）→ 为什么好：日语 CLAUDE.md 以极简散文表达"哪些文件是手写主文件（不可加生成标记）"这条边界规则，信息密度高，没有冗余说明段落 → 原文路径：`CLAUDE.md` → 如何借鉴：在自己的 CLAUDE.md 中增加"文件边界"节，先列"禁止修改"清单，再列"允许操作"说明

4. **双层命令目录分工**（[single-purpose](#single-purpose)）→ 为什么好：`.claude/commands/` 面向人工运维，`config/bootstrap/.../commands/` 面向 AI agent 引导，两类受众的命令分目录管理，不混用 → 原文路径：两个命令目录 → 如何借鉴：设计命令时先明确"这是人用的还是 AI agent 调用的"，不同受众的命令用不同目录隔离

### 3.3 当时的缺陷

1. **4 个 skill 全部缺少 `## Examples` 示例块**：mcp-database、mcp-debugging、mcp-implementation、mcp-research 四个 skill 有完整的操作步骤说明，但无任何具体场景示例。**为什么会失败**：skill 的核心价值是"给 AI 示范怎么做"——没有示例的 skill 相当于教学参考书只有目录没有正文，AI 只能读到抽象规则，无法从具体输入→输出的模式中学习执行风格，导致 AI 执行时自由发挥空间过大。**自查**：我的 skill 文件是否每个都有至少一个 `## Examples` 或 `## 示例` section？

2. **`mcp-debugging/SKILL.md` 声明了未注册的外部依赖**：skill 第 8 行指示用户"先调用 `superpowers:systematic-debugging`"，但 plugin.json 中没有声明 `superpowers` 为依赖，也没有该插件不存在时的降级路径。**为什么会失败**：用户按指示操作却发现 superpowers 插件不可用，AI 找不到该 skill 后可能静默跳过或报错，用户无法定位原因。这是一种"误导性指引比无指引更危险"的失败模式。**自查**：我的 skill 引用的外部 skill/command 是否在 plugin.json 中声明为依赖？

3. **`airis-research-first.md` 用无序列表表达有序工作流**：4 条工作流规则以 `-` 无序列表书写，但实际上是"先做 A、再做 B、然后做 C"的顺序步骤。**为什么会失败**：无序列表向 AI 传递错误信号——这些规则是平等可选的，不是必须按序执行的。AI 可能选择性执行其中几条或颠倒顺序，导致预检行为不可预测。**自查**：我的 command 中是否有用无序列表表达的、实际需要按序执行的步骤？

### 3.4 当时的优化机会

1. **给 4 个 skill 统一补充示例块**（最大单点提升，各 skill +15 分）：每个 skill 加一个 `## 示例` section，展示真实场景下的调用输入→执行步骤→预期输出的完整链路
2. **`airis-research-first.md` 无序列表改编号列表 + 加输出格式节**（+20 分）：把"Rules:"下的 `- ` 改为 `1. `，新增 `## 输出格式` 描述预检完成后产出什么
3. **`troubleshoot.md` 加空 $ARGUMENTS 分支**（+10 分）：命令头部增加"当无参数时列出可选 issue 类型"的引导逻辑

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法（grep / file read） | 现状 | 含义 |
|---|---|---|---|
| `airis-research-first.md` 无序列表 + 无输出格式 | 浅克隆后查看文件内容，检查"Rules:"段落格式和 `## 输出` 节是否存在 | **持续存在**：仍是无序列表规则，仍无输出格式节 | 4 个月内作者未修复这两个质量问题 |
| `troubleshoot.md` 无空参处理 | `grep -n "ARGUMENTS" .claude/commands/troubleshoot.md` | **持续存在**：仍有 `$ARGUMENTS` 但无空值分支 | 空调用仍会渲染"Issue Type: "空白行 |
| 4 个 skill 无示例 + 外部依赖未声明 | `ls skills/` | **通过删除解决**：`skills/` 目录已完全从仓库移除 | 作者选择移除 skill 子包而非修复质量问题 |
| `install.sh` curl\|bash | `grep -n "curl.*bash\|bash.*curl" install.sh` | **持续存在**：第 6 行仍有 curl-pipe-bash 安装示范 | BLOCKED 状态未解除，不可提 PR |

### 4.2 架构演进

从 audit 时（11 个 NL 工件：5 个命令 + 4 个 skill + 1 个 plugin.json + 1 个 CLAUDE.md）到当前 HEAD（5 个命令 + 1 个 CLAUDE.md，无 skills/ 目录），最重要的变化是：

**skills 子包整体移除**：原本通过独立 plugin.json 发布的 mcp-database、mcp-debugging、mcp-implementation、mcp-research 四个 skill，连同 `skills/.claude-plugin/` 目录，已从仓库中完全删除。

这个变化说明作者后来意识到了什么：
- skills 子包的维护成本（外部依赖声明不完整、缺示例）高于其带来的价值
- 核心定位回归：airis-mcp-gateway 是 MCP 网关基础设施，不是知识共享平台
- 宁可没有，不要有质量缺陷的：这是一个"移除比修复更干净"的实际案例

**README 和 CLAUDE.md 语言转为日语**：文档聚焦 Context7 MCP 网关这一单一用途，而非广泛的 MCP 工具集。目标受众更集中，文档表达更精炼。

### 4.3 新增的可学习模式

**约束优先的精简 CLAUDE.md**：当前 HEAD 的 CLAUDE.md 以日语写就，核心是"哪些文件是手写主文件、不要添加生成标记"这类硬约束。这种"约束优先于功能描述"的 CLAUDE.md 写法——先告诉 AI 不能做什么，再告诉 AI 能做什么——比冗长英语说明文字信息密度更高。

这不是语言问题（日语 vs 英语），而是**写作策略问题**：把最重要的架构边界放在文档最前面，用最少的字表达最硬的约束。得分 98/100，仅因"significant tokens"模糊量词扣 2 分，说明精炼约束型 CLAUDE.md 是高分写法。

---

## 五、校准

### 5.1 我已经在做对的

1. **skill 有 Examples section**：MarkQWu/bureau 的 skill 文件都有示例节，审计时 airis-mcp-gateway 的 skill 均缺失这一关键部分——我在这一维度领先
2. **manifest 同步**：我在增加/删除 command 或 skill 时同步更新 plugin.json，不会出现孤儿文件（airis-mcp-gateway 的 plugin.json 满分 100 是同一习惯的结果）
3. **无 curl|bash 安装链**：bureau 和 gstack 均无 shell 脚本安装流程，不存在 BLOCKED 级安全风险
4. **allowed-tools 精确对应**：命令文件的 allowed-tools 声明与实际调用一致，不声明冗余工具
5. **有序步骤用编号列表**：command 文件中的工作流步骤使用编号列表，不用无序列表表达有序步骤

### 5.2 挑战 / 验证

- **挑战了我的假设**："有缺陷的 skill 不如没有 skill"。airis-mcp-gateway 的 skills 子包移除提示了一个反直觉选择：当 skill 有外部依赖未声明、缺示例时，移除比留着更干净。这挑战了我"skill 越多越好"的直觉——误导性指引（指向不存在的外部插件）比无指引更危险，会让用户做出基于错误假设的操作。

- **验证了对 CLAUDE.md 角色的判断**：CLAUDE.md 的"约束优先"写法验证了 CLAUDE.md 应该是行为护栏，不是功能说明书。作者用日语写出"手写主文件不可添加生成标记"这条约束，和我在 bureau/CLAUDE.md 里写"禁止直接修改 trust-registry.json"是同一类思路：先把"不能做什么"钉死，才能安全地放开"能做什么"。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 文件是否每个都有 Examples section
grep -rL '## Examples\|## 示例\|## Example' \
  ~/MarkQWu/bureau/skills/ \
  ~/MarkQWu/gstack/skills/ \
  2>/dev/null
# 命中后：为该 skill 补充至少一个具体场景的"输入→步骤→输出"示例

# 检查 command 中是否有 $ARGUMENTS 但缺空值处理
grep -rln '\$ARGUMENTS' \
  ~/MarkQWu/bureau/.claude/commands/ \
  ~/MarkQWu/gstack/.claude/commands/ \
  2>/dev/null
# 命中后：在命令头部加空值检测逻辑，说明无参数时的行为或列出可选值

# 检查 command 中是否有用无序列表表达的有序步骤
grep -rn '^- ' \
  ~/MarkQWu/bureau/.claude/commands/ \
  ~/MarkQWu/gstack/.claude/commands/ \
  2>/dev/null | grep -v '^\s*$'
# 命中后：判断该列表条目是否需要顺序执行，如果是，改成 "1. 2. 3." 编号列表

# 检查 skill 是否引用了未在 plugin.json 声明的外部依赖
grep -rn 'invoke\|superpowers:\|use.*:.*skill\|call.*plugin' \
  ~/MarkQWu/bureau/skills/ \
  ~/MarkQWu/gstack/skills/ \
  2>/dev/null
# 命中后：确认被引用的外部 skill/command 是否在 plugin.json 依赖中声明，如无降级路径则补充
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau/CLAUDE.md 最顶部增加"文件边界"节，声明手写主文件清单
   - **为何可行**：airis-mcp-gateway 的 CLAUDE.md 以最少文字表达"不要给手写主文件加生成标记"，这是我的 bureau/CLAUDE.md 里目前缺少的防护，bureau 有多个手写配置文件需要类似保护
   - **第一步**：在 `bureau/CLAUDE.md` 最顶部增加 `## 不可改动的文件` 节，列出 trust-registry.json、config.yaml 等手写主文件路径，注明"AI 禁止添加生成标记或覆盖"；约 10 分钟

2. **想法**：对 bureau/gstack 里有 $ARGUMENTS 的命令，统一加空值处理分支
   - **为何可行**：`troubleshoot.md` 的案例说明空 $ARGUMENTS 是一个高频 bug，扣分 -10；在 command 体开头用"如果 $ARGUMENTS 为空，则..."一句话就能修复，ROI 极高
   - **第一步**：用上面的 grep 命令找出所有含 `$ARGUMENTS` 的命令文件，逐一在文件开头添加空值分支；每个文件约 5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（由 `.github/workflows/refresh-my-repos.yml` 每周一 01:00 UTC 自动刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 agiletec-inc/airis-mcp-gateway 的核心目的**：本地 MCP 协议网关 + Claude Code 运维命令，面向运行时基础设施管理

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 基础设施插件；都有 CLAUDE.md + commands；都是小型 NL 层 | bureau 侧重知识管理（capture/query），airis 侧重 MCP 协议路由；bureau 有 skill 层，airis 的 skill 层已移除 | 高 |
| MarkQWu/gstack | 低 | 都有 Claude Code commands；都关注架构边界 | gstack 是工具集合（59 个 NL 工件），airis 是单一用途 MCP 网关（5 个工件） | 低 |
| MarkQWu/- | 无 | — | 实时情报仪表盘，完全不同领域 | 无 |

### 8.2 在我的项目里复现的同类问题

对 §3.3「当时的缺陷」和 §4.1「现在仍存在的缺陷」逐条核查我的项目（基于 my-repos.json 中 priority=high 的仓库做 grep 验证）：

| 本案例缺陷 | 检查方法（grep / file） | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 缺 `## Examples` section | `grep -rL '## Examples' bureau/skills/*/SKILL.md` | 数据显示 bureau 有 Examples sections——此项我已覆盖 | 低（已覆盖） |
| `$ARGUMENTS` 无空值处理 | `grep -rln '\$ARGUMENTS' gstack/.claude/commands/` | gstack 有 59 个 NL 工件，troubleshoot 类命令存在该风险，需逐一核查 | 中 |
| 外部依赖未在 plugin.json 声明 | `grep -rn 'superpowers:\|invoke.*:' bureau/skills/` | 目前 bureau 无跨插件引用，风险低；但未来扩展时需注意 | 低 |
| 无序列表表达有序步骤 | `grep -rn '^- ' gstack/.claude/commands/*.md` | gstack 规模大（59 工件），此类笔误存在概率较高 | 中 |

**命中后的具体行动建议**：
- MarkQWu/gstack 中含 `$ARGUMENTS` 的 troubleshoot 类命令 → 在命令文件最顶部加空值检测段落（"当无参数时，列出可选问题类型，用户选择后重新调用"）→ 每个文件约 5 分钟
- MarkQWu/gstack 中以无序列表写就的多步流程 → 逐条判断是否需要顺序执行，如果是则改为编号列表 → 每处修改约 2 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：CLAUDE.md 约束优先写法
   - **本案例做法**：CLAUDE.md 用日语极简表达"手写主文件不可添加生成标记"，一句话钉死边界，放在文档最顶部，98/100 高分 → 路径：`CLAUDE.md`
   - **我的项目现状**：bureau/CLAUDE.md 的边界说明散落在功能描述段落中，没有独立的"禁止行为"或"文件边界"节，相比之下约束的可见性低
   - **如何借鉴**：在 bureau/CLAUDE.md 顶部加 `## 不可改动的文件` 节，列出所有手写主文件路径；改动量约 10 行，约 10 分钟

2. **领域**：双层命令目录分工（人工运维 vs AI agent 引导）
   - **本案例做法**：`.claude/commands/` 和 `config/bootstrap/.../commands/` 分别面向不同受众，路径即语义——进入 `.claude/` 就知道是运维命令，进入 `config/bootstrap/` 就知道是 AI 引导层
   - **我的项目现状**：bureau 所有命令都在 `.claude/commands/` 下，人工入口和 AI agent 调用入口混放，区分度低
   - **如何借鉴**：若 bureau 未来有专门给 AI agent 调用的引导命令，考虑用 `config/bootstrap/` 或 `commands/agents/` 子目录隔离，不与人工运维命令混放

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：skill 示例覆盖率
   - **我的做法**（MarkQWu/bureau）：skills 有 `## Examples` section，展示具体场景的输入→执行步骤→输出链路
   - **本案例做法**（弱在哪）：audit 时所有 4 个 skill 均缺 Examples section；作者最终选择删除 skill 子包而非补充示例——这说明维护成本估算时示例被认为负担过重
   - **意义**：bureau 在这一维度的质量高于 airis-mcp-gateway 的审计时状态；若未来 bureau 被外部审计，skill 示例覆盖率是可预期的高分亮点

---

## 八、术语表

这一节是给「不熟悉技术名词的读者」准备的。正文中第一次出现专有名词时用锚点链接到这里。

### <a name="mcp"></a>MCP（Model Context Protocol）
> 模型上下文协议，一套让 AI 大模型（如 Claude）和外部工具/数据源交互的标准接口协议。类比：MCP 相当于 AI 世界的"USB 标准"——有了这个标准，任何人写的工具只要遵守 MCP 协议，AI 就能直接调用，不需要每次都定制对接代码。airis-mcp-gateway 就是这个标准下的一个"路由器"，负责把 AI 的请求转发给正确的 MCP 工具服务。

### <a name="curl-pipe-bash"></a>curl|bash（curl-pipe-bash）
> 一种常见但有安全风险的安装模式：`curl -fsSL https://...安装脚本 | bash`，意思是"从网上下载一个 shell 脚本，不看内容，直接交给 bash 执行"。风险在于：如果下载地址被劫持（CDN 污染、GitHub 账号被盗、中间人攻击），用户在不知情的情况下执行了恶意代码。NLPM 安全扫描把这类模式标为 Critical 级别，会阻止自动提 PR。**对比**：更安全的做法是先下载文件，再用 SHA-256 校验文件完整性，确认无误后再执行。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读命令/技能/代理文件时先解析 frontmatter，才能知道如何注册和调用这个文件。`---` 必须严格从行首第 1 列开始。

### <a name="manifest"></a>manifest（plugin.json）
> 项目的"清单文件"，告诉 Claude Code 这个插件包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上存在也不会被加载，对用户完全不可见。

### <a name="manifest-discipline"></a>manifest-discipline（manifest 纪律）
> 一种开发习惯：每次增加、删除、重命名 command/skill/agent 文件时，立即同步更新 plugin.json，保持 manifest 和实际文件系统完全一致。manifest 不同步是 Claude Code 插件最常见的静默 bug 来源之一。

### <a name="single-purpose"></a>single-purpose（单职责）
> 一个设计原则：每个 command、skill 或 agent 只做一件事，不把多个职责混合在同一个文件里。单职责的 NL 工件更容易理解、测试和维护，AI 调用时也更可预测。airis-mcp-gateway 把"运维命令"和"AI 引导命令"分到不同目录，就是单职责在目录层面的体现。
