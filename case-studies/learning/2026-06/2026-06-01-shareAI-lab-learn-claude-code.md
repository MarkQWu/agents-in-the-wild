# shareAI-lab/learn-claude-code — 学习案例

**仓库**：https://github.com/shareAI-lab/learn-claude-code
**Stars**：59,868 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-01（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `examples-driven`, `curl-pipe-bash-risk`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

这是 GitHub 上星标数最高的 Claude Code 教程仓库之一，59,868 颗星。由 shareAI-lab 维护，定位是「从零开始手写 Claude Code Agent」的渐进式教程。核心主张是：**Agency 来自模型训练，而非外部代码编排**——开发者构建的是 harness（执行环境），而非 agent 本身。

仓库横跨中文、日文、英文三语 README（`README.md`、`README-zh.md`、`README-ja.md`），面向全球华语社区，也是目前少数提供完整 20 节渐进式 agent 教程的公开仓库。

关键事实：
1. **20 节渐进式课程**（s01→s20），每节专注一个能力层：从最基础的 agent loop 到子代理、记忆系统、MCP 集成、综合实战
2. **双轨结构**：`agents/` 目录（13+ Python 脚本，可直接运行）+ `skills/` 目录（4 个 NL [SKILL.md](#skill-md)，供 Claude Code 加载）
3. **无 plugin.json**：不走 marketplace 发布渠道，直接通过目录结构接入 Claude Code
4. **安全评级 BLOCKED**：`agents/s01_agent_loop.py` 第 70 行 `subprocess.run(command, shell=True)` 触发 HIGH 级安全发现，贡献流程被安全门控拦截，未能推送改进 PR
5. **质量评分 88/100**：4 个 skill 文件中 3 个缺少 `## Output Format` 节，多处[模糊量词](#模糊量词)

### 1.2 架构剖析

目录结构（当前 HEAD）：
```
learn-claude-code/
  agents/
    s01_agent_loop.py          # 基础 agent loop（含 shell=True ← 安全问题）
    s02_tool_use.py
    ...
    s12_mcp_integration.py
    s_full.py                  # 完整参考实现（共 14 个 Python 脚本）
  skills/
    agent-builder/SKILL.md     # 评分 88/100
    code-review/SKILL.md       # 评分 94/100（最优）
    mcp-builder/SKILL.md       # 评分 84/100
    pdf/SKILL.md               # 评分 84/100
  s01_agent_loop/              # 每节对应的完整示例目录
  s02_tool_use/
  ...
  s19_mcp_plugin/
  s20_comprehensive/
  web/                         # Web 可视化界面
  requirements.txt             # anthropic>=0.25.0 等（使用 >= 宽松固定）
  README.md / README-zh.md / README-ja.md
```

- **文件类型分布**：4 个 skill，0 个 agent（NL 定义），0 个 command，0 个 hook，0 个 manifest；14 个 Python 可执行脚本
- **编排关系**：Python 脚本与 NL skill 之间无硬性依赖。脚本演示底层实现原理，skill 提供 Claude Code 可加载的高层行为指南，两者通过教育语义联系而非代码引用
- **跨件契约**：无正式契约。skills 路径不在任何 plugin.json 中注册，依赖用户手动加载或 Claude Code 自动发现

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「教程式渐进架构」（Tutorial Progressive Architecture）——每个编号模块（s01→s20）只引入一个新能力，前一节的代码是后一节的基础，学习曲线平滑可控
- **解决什么问题**：大多数 Claude Code 教程停留在「用法介绍」层面，缺少对底层机制的拆解。本仓库通过让学习者亲手实现 agent loop 的每一层，建立完整的心智模型
- **做了什么 trade-off**：为了教学清晰，`agents/` 脚本刻意使用 `subprocess.run(command, shell=True)` 展示「裸 bash_tool 实现」，代价是引入了真实的安全风险——而且仓库没有用醒目的免责声明说明「仅限本地可信环境」
- **反映什么认知模型**：作者把 agent 技术教学分为两个层次：Python 层（原理实现）和 NL 层（生产配置），分别对应不同读者——前者面向想理解机制的开发者，后者面向想直接接入 Claude Code 的使用者

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「教程式渐进双轨」（Tutorial Progressive Dual-Track）**

这个模式的核心特征：同一仓库同时维护两条并行轨道——代码实现轨（Python 脚本，用于原理教学）和 NL 配置轨（SKILL.md，用于 Claude Code 集成），两条轨道通过统一的领域主题（agent 构建）在教育语义层面对齐。

模式特征清单（4 条）：
- **特征 1**：编号模块递进（s01→s20），每节专注单一能力扩展，形成可验证的学习路径
- **特征 2**：双轨并行——Python 实现轨展示「how it works」，NL skill 轨展示「how to use」，分层服务不同读者
- **特征 3**：三语 README（zh + ja + en），主动覆盖多语言社区
- **特征 4**：零 manifest 配置，直接依赖 Claude Code 原生目录发现机制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 技术教程仓库：逐步讲解 agent 实现原理 | ✅ 高度适用 | 渐进编号模块 + 代码示例是该模式的核心优势 |
| 学习者本地动手实验 | ✅ 适用 | 脚本直接可运行，每节独立，可随时重来 |
| 面向多语言社区的开源知识传播 | ✅ 适用 | 多语言 README 已验证此路径 |
| 生产环境 agent 部署 | ❌ 不适用 | `shell=True` + 宽松版本固定不符合生产安全要求 |
| 通过 marketplace 发布的插件 | ❌ 不适用 | 缺少 plugin.json，无法走正式发布渠道 |
| 需要跨代理协作的复杂 workflow | ❌ 不适用 | 无命令定义，无跨件编排机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 教程式渐进双轨（本仓库） | shareAI-lab/learn-claude-code | 学习路径清晰，原理与配置并举，社区覆盖广 | 安全边界模糊，无 manifest，生产不可直用 |
| 纯 skill 合集 | forrestchang/andrej-karpathy-skills | 零执行面，安全评级 CLEAR，单一职责清晰 | 无原理演示，学习深度有限 |
| 完整插件（带 manifest） | MarkQWu/echo-sleuth-for-claude | 可发布 marketplace，生产就绪，Python+skill 协作 | 需维护 plugin.json 同步，新增成本较高 |
| 单仓多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置，138 个专家代理，新增成本极低 | 无教程路径，无 skill，无 Python 实现层 |

### 2.4 改进空间

1. **当前问题**：`agents/` 脚本的安全免责声明缺失。**改进做法**：在 README 显眼位置（第一屏）和每个 `agents/*.py` 文件头部加注 `# ⚠️ 仅限本地可信环境。shell=True 会将模型输出直接传入 shell，生产环境请改用 allowlist 架构`。**预期收益**：消除用户对安全边界的误解，同时保留教学意图的完整性。

2. **当前问题**：[blocklist](#blocklist) 给了「已安全」的错误信号。`["rm -rf /", "sudo", "shutdown"]` 可被 `rm -rf / `（末尾空格）、`sudo sh` 等轻易绕过。**改进做法**：将 blocklist 替换为明确的信任模型文档：「此脚本假设执行者信任 LLM 输出；blocklist 仅为演示目的，不构成安全保障」。**预期收益**：消除 HIGH 安全发现，解除 BLOCKED 状态。

3. **当前问题**：3 个 skill 缺失 `## Output Format` 节（agent-builder、mcp-builder、pdf）。**改进做法**：参照 code-review/SKILL.md 的结构化模板，为每个 skill 补充具体的输出格式定义。**预期收益**：每个 skill 各恢复 10 分，理论质量评分上限从当前加权均值升至 93+。

4. **当前问题**：`requirements.txt` 使用 `>=` 宽松固定（`anthropic>=0.25.0`）。**改进做法**：锁定精确版本（`anthropic==0.49.0`）并附带 `requirements-dev.txt` 供开发使用，或使用 `~=` 兼容发布符。**预期收益**：消除 Medium 安全发现，保证教程可重现性。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

| 文件 | 得分 | 主要扣分项 |
|---|---|---|
| skills/code-review/SKILL.md | 94/100 | 模糊量词：thorough, comprehensive, Meaningful（-6） |
| skills/agent-builder/SKILL.md | 88/100 | 缺失 Output Format（-10），模糊量词：relevant（-2） |
| skills/mcp-builder/SKILL.md | 84/100 | 缺失 Output Format（-10），模糊量词：meaningful, sensitive, safe（-6） |
| skills/pdf/SKILL.md | 84/100 | 缺失 Output Format（-10），模糊量词：preferred, recommended, various（-6） |
| **整体加权均值** | **88/100** | 安全评级：**BLOCKED**（2 HIGH 安全发现） |

### 3.2 当时值得借鉴的模式

1. **code-review/SKILL.md 的结构化输出模板**：明确定义了 `## Code Review: [file]` → `### Critical Issues` → `### Improvements` → `### Verdict` 的完整输出结构，是 4 个 skill 中唯一达到 90+ 的文件，可作为其他 skill 的参照基准
2. **渐进编号模块**（s01→s12，当时尚未扩展到 s20）：每节单一能力聚焦，已在 audit 时被识别为可学习的教学组织模式
3. **双语 README**（zh + en）：当时已具备，体现了面向华语社区的主动设计意识

### 3.3 当时的缺陷

1. **HIGH 安全发现 ①**：`agents/s01_agent_loop.py` 第 70 行 `subprocess.run(command, shell=True)` 将 LLM 生成的命令字符串直接传入 shell，影响全部 13 个 agent 文件（s01-s12 + s_full）及 2 个参考脚本
2. **HIGH 安全发现 ②**：[blocklist](#blocklist) 仅包含字面字符串匹配（`"rm -rf /"`, `"sudo"` 等），可被末尾空格、大小写变体、命令拼接等方式轻易绕过，造成虚假安全感
3. **3 个 skill 缺失 Output Format**：agent-builder、mcp-builder、pdf 均无 `## Output Format` 节，导致 Claude 在使用这些 skill 时无法对输出结构形成确定性预期
4. **13 处模糊量词**：thorough, comprehensive, Meaningful, relevant, meaningful, sensitive, safe, preferred, recommended, various——这些词在 skill 上下文中无法被机器或人类验证，削弱了 skill 的可测试性
5. **`Load it when relevant` 标准未定义**：agent-builder/SKILL.md 中出现的这一表述没有任何可操作的触发条件

### 3.4 当时的优化机会

1. **最高优先级**：修复 blocklist 设计，改为明确的信任边界声明，解除 BLOCKED 状态（否则任何其他改进都不会被贡献回上游）
2. **中优先级**：为 3 个缺失 Output Format 的 skill 补充结构化输出定义（参照 code-review/SKILL.md）
3. **低优先级**：批量替换模糊量词（"thorough" → "覆盖所有 Critical Issues 条目"，"comprehensive" → "列出 5 个以上检查项" 等）
4. **维护优先级**：将 `requirements.txt` 中的 `>=` 改为精确版本固定，配套 `CHANGELOG.md` 记录版本变更

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-06 状态 | 2026-06-01 状态 | 变化 |
|---|---|---|---|
| `subprocess.run(command, shell=True)` | 存在，agents/s01_agent_loop.py:70 | **仍存在**，同一位置 | 未修复 |
| Blocklist 虚假安全感 | 存在，["rm -rf /", "sudo", ...] | **仍存在**，逻辑未变 | 未修复 |
| agent-builder/SKILL.md 缺失 Output Format | 存在 | **仍存在**（audit BLOCKED，未贡献） | 未修复 |
| mcp-builder/SKILL.md 缺失 Output Format | 存在 | **仍存在** | 未修复 |
| pdf/SKILL.md 缺失 Output Format | 存在 | **仍存在** | 未修复 |
| requirements.txt >= 宽松固定 | 存在 | **仍存在** | 未修复 |
| 模糊量词（13 处） | 存在 | 未经验证，推测仍存在 | 未修复 |

> **说明**：所有缺陷均未修复，根本原因是安全评级 BLOCKED——贡献流程在安全门控阶段被拦截，改进 PR 从未被打开。这意味着即使质量问题有解决方案，安全问题不先修复，就没有任何改进能够回流到仓库。

### 4.2 架构演进

从 s12（audit 时）扩展到 s20（当前 HEAD），新增了：
- `s19_mcp_plugin/`：MCP 插件集成专题
- `s20_comprehensive/`：综合实战课程
- `agents/` 目录从 13 个 Python 脚本增长到 14 个
- `web/` 目录保留（但 Web 平台仍渲染旧版 12 节轨道，存在版本不一致）

核心架构没有变化：双轨结构（Python + NL skill）、零 manifest、无命令定义。扩展方式是在已有模式上追加节数，而非重构设计。

### 4.3 新增的可学习模式

1. **MCP 集成专题（s19）**：将 MCP（Model Context Protocol）服务器的搭建过程拆解为独立一节，配合 `skills/mcp-builder/SKILL.md`，形成「原理实现 + NL 配置」的完整闭环，是双轨模式在新技术点上的延伸
2. **综合实战节（s20）**：作为课程终点提供完整 agent 系统的组合演示，隐式确认了「s01→s19 的每一节都是一个可测试的能力单元」这一设计约定

---

## 五、校准

### 5.1 我已经在做对的

1. **Python 脚本不使用 `shell=True` 传入 LLM 输出**：`echo-sleuth-for-claude` 的 `scripts/` 目录使用 shell 脚本但没有将 LLM 生成内容直接传入 shell，绕开了本仓库最严重的安全风险
2. **有 plugin.json manifest**：`echo-sleuth-for-claude` 正确配置了 plugin.json，可走 marketplace 发布渠道；shareAI-lab 完全缺失这一层
3. **skills 整体更干净**：用户仓库只有 2 处模糊量词（两处 "relevant"），而 shareAI-lab 有 13 处——从 vague quantifier 密度来看，用户仓库的 NL 质量更高

### 5.2 挑战 / 验证

1. **挑战**：`echo-sleuth-for-claude` 中 3 个 skill 同样缺少 `## Output Format` 节——这与 shareAI-lab 的 agent-builder、mcp-builder、pdf 是同类问题，只是数量相当。需要验证哪些 skill 缺失，并逐一补充
2. **挑战**：用户仓库没有渐进式教程结构。如果未来需要为用户写技术博客或教程，缺少 s01→sNN 这类有序的能力分解路径会增加读者的认知负担
3. **验证**：`echo-sleuth-for-claude` 的 `scripts/` 是否有任何位置间接使用 LLM 输出？需要 grep 确认没有隐式的 shell 注入风险

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查自己的仓库中是否有 shell=True 且使用了外部输入
grep -rn "shell=True" ~/projects/ --include="*.py" | grep -v "test_" | grep -v "#"

# 2. 检查是否有 subprocess 直接使用变量（潜在注入风险）
grep -rn "subprocess.run\|subprocess.call\|os.system" ~/projects/ --include="*.py" | grep -v "#"

# 3. 检查 skills/ 目录下哪些 SKILL.md 缺失 Output Format 节
for f in $(find ~/projects -name "SKILL.md"); do
  if ! grep -q "## Output Format" "$f"; then
    echo "MISSING Output Format: $f"
  fi
done

# 4. 检查模糊量词密度（前 10 个高频词）
grep -rnioh "thorough\|comprehensive\|meaningful\|relevant\|appropriate\|various\|preferred\|recommended\|sensitive\|safe" \
  ~/projects --include="SKILL.md" | wc -l

# 5. 检查 requirements.txt 是否使用宽松版本固定
grep -rn ">=" ~/projects --include="requirements.txt"

# 6. 检查 plugin.json 是否存在且格式正确
find ~/projects -name "plugin.json" | xargs -I{} python3 -c "import json,sys; json.load(open('{}'))" && echo "OK" || echo "INVALID JSON"
```

### 6.2 灵感 → 实施路径

| 灵感来源 | 具体实施步骤 | 预期收益 |
|---|---|---|
| code-review/SKILL.md 的结构化输出模板 | 为 `echo-sleuth-for-claude` 中缺失 Output Format 的 skill 补充 `## Output Format` 节，参照 `## [Review Name]: [target]\n### Summary\n...` 的层级结构 | 每个 skill 恢复 10 分；Claude 输出更可预测 |
| shareAI-lab 的渐进编号模式 | 若创建新教程类仓库，采用 `s01_xxx/`、`s02_xxx/` 命名约定，每节一个 `README.md` + 一段可运行代码 | 学习路径可验证，贡献者可精准定位要改进的节 |
| 安全边界明确化 | 在任何含可执行脚本的仓库 README 第一屏加注信任模型说明；避免 blocklist 模式，改为 allowlist 或明确免责 | 消除安全门控风险，保持贡献渠道畅通 |
| blocklist → 信任边界声明 | 如果 `scripts/` 有接受外部输入的脚本，添加 `SECURITY.md` 明确说明信任假设和使用边界 | 安全评级从潜在 BLOCKED 保持在 CLEAR |

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 与 shareAI-lab/learn-claude-code 的目的对齐度 | 关键共同点 | 关键差异 |
|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | **高**（Python + skill 双轨，最相似） | 都有 Python 脚本 + NL skill 组合；都面向 Claude Code 生态 | 用户仓库有 plugin.json，有生产就绪意图；shareAI-lab 是纯教程，无发布意图 |
| MarkQWu/drama-workshop-skills | **低**（纯 skill，无 Python 脚本） | 都使用 SKILL.md 格式 | 无执行面，安全风险天然为零；无教程路径 |
| MarkQWu/claude-for-legal | **低**（领域 skill，无 Python 脚本） | 都使用 SKILL.md 格式 | 垂直领域应用，无 agent 实现层 |

### 8.2 在我的项目里复现的同类问题

| 问题类型 | shareAI-lab 实例 | 我的项目实例 | 严重程度 |
|---|---|---|---|
| 缺失 Output Format 节 | agent-builder、mcp-builder、pdf（3 个） | echo-sleuth-for-claude/skills/ 中 3/4 个 skill 缺失 | 中（-10 分/个） |
| 模糊量词 | 13 处（thorough, comprehensive 等） | echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md:118 和 git-mining/SKILL.md:93 各一处 "relevant" | 低（2 处，密度远低于 shareAI-lab） |
| 宽松版本固定 | requirements.txt 中 `anthropic>=0.25.0` | 需自查 requirements.txt 是否存在 | 待确认 |

### 8.3 别人的更优方案

1. **渐进编号模块（s01→s20）**：shareAI-lab 通过编号目录将学习路径具象化，每一节可被独立测试和引用。这是在教程类仓库中建立「可验证学习路径」的最低成本方式——只需目录命名约定，无需额外工具。
2. **三语 README**：shareAI-lab 覆盖 zh + ja + en，面向三个不同语言社区。这比单语 README 多花约 30% 的维护成本，但显著扩大了受众基础，对开源影响力有直接贡献。
3. **双轨设计**（实现轨 + 配置轨）：将「理解原理」和「直接使用」分离成两条独立的读者路径，避免了一份文档同时服务两类读者导致的深度不足。这个设计思路可以移植到任何「既想教原理又想提供工具」的仓库。

### 8.4 反向：我的项目做得比他们好的地方

1. **plugin.json manifest**：`echo-sleuth-for-claude` 正确配置了 plugin.json，可走 marketplace 发布渠道，且 `skills` 数组路径与实际目录一致。shareAI-lab 完全缺失这一层，无法被正式发现和安装。
2. **无 `shell=True` 安全风险**：用户脚本不将 LLM 输出传入 shell，安全评级维持 CLEAR，贡献渠道保持畅通。这是最重要的差异——shareAI-lab 因安全问题永久处于 BLOCKED 状态，所有质量改进都无法通过自动化渠道回流。
3. **更低的模糊量词密度**：2 处 vs. 13 处（echo-sleuth-for-claude vs. shareAI-lab），NL 文本质量更高，NLPM 扣分更少。

---

## 八、术语表

| 术语 | 解释 |
|---|---|
| <a name="shell-true安全风险"></a>shell=True 安全风险（shell injection） | Python `subprocess.run(cmd, shell=True)` 将字符串 `cmd` 交给 `/bin/sh -c` 执行。若 `cmd` 来自 LLM 输出或用户输入，攻击者可通过 `; rm -rf ~` 等 shell 元字符注入任意命令。正确做法：传入 `list` 参数而非字符串，或在受信任边界内使用 `shell=False` |
| <a name="blocklist"></a>blocklist（黑名单） | 禁止特定模式的访问控制策略。在 shell 命令过滤场景中，blocklist 枚举「不允许的命令」（如 `rm -rf /`）。其根本缺陷是遗漏面（false negatives）不可穷举——攻击者只需找到一个未列入黑名单的等效方式即可绕过 |
| <a name="allowlist"></a>allowlist（白名单） | 只允许特定模式的访问控制策略。与 blocklist 相反，allowlist 枚举「允许的命令」（如仅允许 `ls`、`cat`、`grep`），未列入的一律拒绝。安全强度远高于 blocklist，但需要更精确的业务需求定义 |
| <a name="output-format"></a>output format（输出格式） | SKILL.md 中 `## Output Format` 节，定义 Claude 在执行该 skill 时应产出的具体文本结构（如 Markdown 模板、JSON schema 等）。缺失此节会导致 Claude 的输出结构不可预测，NLPM 评分扣 10 分 |
| <a name="semver-pinning"></a>semver pinning（版本固定） | 在 `requirements.txt` 或 `package.json` 中锁定依赖的精确版本（`==1.2.3`）或兼容范围（`~=1.2`）。`>=` 宽松固定允许任意高版本，可能导致 breaking change 悄无声息地进入构建，影响教程可重现性 |
| <a name="模糊量词"></a>vague quantifier（模糊量词） | 在 NL 工件中无法被机器或人类客观验证的程度词，如 "thorough"、"comprehensive"、"relevant"、"appropriate"。NLPM 每处扣 2 分，上限 6 分。解决方式：替换为可测量标准（"≥80% 分支覆盖" 而非 "thorough test coverage"） |
| <a name="skill-md"></a>SKILL.md | Claude Code skill 的定义文件，放置于 `skills/<skill-name>/SKILL.md`。包含 YAML frontmatter（name、description 等元数据）和 Markdown 正文（背景、能力边界、输出格式、示例）。由 agent 的 `skills:` frontmatter 声明后自动加载 |
