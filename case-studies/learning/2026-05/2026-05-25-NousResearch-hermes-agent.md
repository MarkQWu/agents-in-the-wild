# NousResearch/hermes-agent — 学习案例

**仓库**：https://github.com/NousResearch/hermes-agent
**Stars**：94,145 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-05-25（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `vague-quantifier`, `model-pinning`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Hermes Agent 是 NousResearch 出品的生产级 AI agent 平台——不只是一个 Claude Code 插件，而是一个完整的 AI 代理运行时（带 CLI、TUI、MCP 服务器、Python 包）。它的 skill 库涵盖 89 个核心技能（apple/creative/data-science/devops/email/gaming/mlops/productivity/social-media 等 20+ 类目）+ 81 个 optional-skills，是目前 NLPM 审计过的**体量最大**的 skill 仓库。

关键事实：
- 94,145 stars，在 Claude Code 生态中是头部项目之一
- NousResearch 是知名的开源 AI 研究机构（以 Hermes 系列微调模型著名）
- 用户通过 `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash` 安装——这也是核心安全问题所在
- 在生态中是「全能型 AI 代理框架」定位，对标 Claude Code 的使用场景做了完整的封装和扩展

### 1.2 架构剖析
- **目录结构**：
  ```
  hermes_cli/         ← Python CLI 核心（hermes_cli 包）
  skills/             ← 89 个核心 SKILL.md（按类目分目录）
  optional-skills/    ← 81 个可选 SKILL.md（需单独激活）
  scripts/            ← 安装/引导/Node.js bootstrap 脚本
  web/                ← Web 界面
  ui-tui/             ← 终端 UI
  gateway/            ← API 网关
  providers/          ← 多 LLM 提供商适配
  plugins/            ← 插件系统
  ```
- **文件类型分布**：89 个核心 SKILL.md + 81 个 optional SKILL.md + Python CLI + 多个 bash/PowerShell 安装脚本；无 hook-config（直接用 Claude Code hooks？）
- **编排关系**：hermes-agent 是平铺型 skill 库，每个 SKILL.md 独立，无集中 router。optional-skills 需要用户手动激活。Python CLI 处理 agent 运行时逻辑
- **跨件契约**：SKILL.md 通过 [frontmatter](#frontmatter) 里的 `metadata.hermes.tags` 和 `metadata.hermes.related_skills` 实现技能间的引用关系。但存在三种不同的 [frontmatter](#frontmatter) 方言（见§3.3）——这是最大的跨件一致性问题

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一个 framework，所有领域」——hermes-agent 想成为 AI 工作的操作系统，而不只是一个工具集合
- **解决什么问题**：Claude Code 开箱即用没有领域专业知识（apple 自动化、论文写作、MLOps 等），hermes-agent 通过 skill 库填补这个空缺
- **做了什么 trade-off**：
  - 选择大量技能而非精选少量——覆盖广但维护成本极高，导致部分 skill 是自动生成的占位符
  - 选择 curl|bash 一键安装而非包管理——降低用户门槛，但引入了严重安全风险
  - 选择 exec() over HERMES_HOME 路径加载 godmode——灵活但危险（env 注入即可执行任意代码）
- **反映什么认知模型**：作者把 hermes-agent 理解为一个「AI 技能市场的载体」——用户可以按需激活 optional-skills，类似 App Store 的概念

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「大型平铺 Skill 库 + 多类目组织」**

这是一种「水平扩展」架构：主干是运行时（CLI + MCP），枝干是无数独立的 SKILL.md，每个技能是叶节点，彼此通过 `related_skills` 松耦合。整体像一棵扁平的树，根深叶茂。

模式特征清单：
- **特征 1**：按领域分 20+ 类目，每类目下多个 SKILL.md，无集中路由层
- **特征 2**：标准 frontmatter schema（metadata.hermes.tags/related_skills/homepage）作为技能间的「目录索引」
- **特征 3**：optional-skills 目录提供「付费功能」或「高风险功能」的隔离区
- **特征 4**：安装时 curl|bash 一次拿全，no partial install（这是安全和分发的核心矛盾）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖广泛领域的全能平台 | ✅ 适用 | 平铺架构便于横向扩展 |
| 企业内部工具（高安全要求） | ❌ 不适用 | curl\|bash 安装 + eval 模式不可接受 |
| 个人工具箱（接受安全风险） | ⚠️ 谨慎 | 功能丰富但安全门槛要求用户自主评估 |
| 需要精选、高质量 skill 的专业插件 | ❌ 不适用 | 批量生成的占位符 skill 会污染质量 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 大型平铺 skill 库（本仓库） | hermes-agent | 覆盖广；横向扩展容易 | 质量难以统一；frontmatter 方言分裂 |
| 精选单职责 skill 集 | echo-sleuth-for-claude | 质量可控；每个 skill 都经过认真设计 | 覆盖有限；扩展慢 |
| 按领域拆分多仓库 | claude-for-legal | 每个领域独立版本化；各领域团队自治 | 跨领域搜索难；安装复杂 |

### 2.4 改进空间
1. **当前问题**：3 种 [frontmatter](#frontmatter) 方言导致基于 `metadata.hermes.tags` 的搜索只能找到 ~80% 的 skill。**改进做法**：用一个 CI 脚本（类似 NLPM 的 `bin/nlpm-check`）在 PR 合并前校验所有 SKILL.md 的 frontmatter schema 合规性。**预期收益**：一次性消灭方言问题，且以后新增 skill 自动被检查

2. **当前问题**：占位符 skill（unsloth、axolotl，当前版本已是空文件）混入主 skill 库，用户激活后得不到任何有用信息。**改进做法**：在 PR 合规检查里加「content 长度 ≥ 500 字节」的规则，空文件或占位符无法合入。**预期收益**：防止未来再出现同类问题

3. **当前问题**：shell=True 在 tools_config.py:721 仍然存在，plugin 提供的 install_cmd 可注入任意 shell 命令。**改进做法**：用 `shlex.split(install_cmd)` 替换 `shell=True`，并加白名单校验 install_cmd 只包含可信命令格式。**预期收益**：消除命令注入向量

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **80/100**（PASS），Security: **BLOCKED**（8 个 CRITICAL 安全发现）。

| 评分区间 | 文件数 | 代表性问题 |
|---|---|---|
| 90-100（优秀） | 12 个 | software-development/* 系列——subagent-driven-development、TDD、systematic-debugging |
| 80-89（良好） | 62 个 | 占主体，基本合格但有小问题（missing author/license/related_skills） |
| 70-79（可接受） | 17 个 | 缺 pitfalls 节、缺 homepage |
| 60-69（BUG） | 10 个 | frontmatter 不完整或使用方言 B/C |
| 50-59（严重 BUG） | 4 个 | 自动生成占位符（unsloth、axolotl）+ 极度不完整 |

### 3.2 当时值得借鉴的模式
1. **软件工程类 skill 的「极致深度」** → software-development/ 下的 subagent-driven-development、test-driven-development、systematic-debugging 得分 90 分，全部有完整的「4步流程 + 红旗列表 + 示例工作流」→ 示例：`skills/software-development/subagent-driven-development/SKILL.md` → 借鉴：高质量 skill 的 body 应该有：流程步骤、反模式/红旗、验证方法

2. **baoyu-infographic 的「矩阵式示例」** → 21×21 的排版/风格矩阵，每个格子都是可执行的 input→output 对 → 示例：`skills/creative/baoyu-infographic/SKILL.md` → 借鉴：对于有大量参数组合的 skill，矩阵表格比文字列表信息密度高 10 倍

3. **google-workspace SKILL 的完整性** → OAuth 设置、故障排除、安全规则全部涵盖，score 88 → 借鉴：生产用 skill 必须有「踩坑记录」节（pitfalls / troubleshooting）

4. **Secret Safety 节（xurl）** → 即使这个 skill 自己有安全问题（curl-pipe-bash），它在 SKILL.md 里专门有 `## Secret Safety` 节教用户怎么处理敏感信息 → 借鉴：凡是涉及 API key / 密码的 skill，专门加一个 Secret Safety 节

### 3.3 当时的缺陷
1. **16 个 BUG 级文件（frontmatter 缺字段）** → 为什么失败：批量添加 skill 时没有 CI 校验，新贡献者不熟悉 schema，或用自动生成工具没有正确填写所有字段 → 根本原因是**无自动化门槛** → 自查：我的 claude-for-legal 有 151 个 SKILL.md，是否也有类似问题？

2. **3 种 frontmatter 方言并存** → 方言 B 用顶层 `tags`/`triggers` 代替 `metadata.hermes.tags`，方言 C 只有 name+description → 为什么会这样：仓库成长过快，schema 本身在演进，早期贡献没有被追溯更新 → 自查：我的 skill 是否也有多套 schema 并存？

3. **shell=True 命令注入风险（memory_setup.py）** → `subprocess.run(check_cmd, shell=True)` 其中 check_cmd 来自用户安装的 plugin 的 metadata → 为什么危险：恶意 plugin 可以把任意 shell 命令放进 check_cmd，一旦被加载就会执行 → 自查：我的代码里有没有 `shell=True` + 外部输入？

### 3.4 当时的优化机会
1. 用 shlex.split 替换 shell=True（P0 安全修复，5 分钟工作量）
2. 给 lodash override 改为 4.17.21（当前写了一个不存在的 4.18.1）
3. 批量修复 16 个 BUG 文件的 frontmatter（P1，约 2 小时）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| shell=True in memory_setup.py | `grep -n "shell=True" hermes_cli/` | **仍然存在**，但挪到了 `tools_config.py:721`；`memory_setup.py` 里变成了注释说明而非实际调用 | 部分修复：旧位置改了，但新位置引入同样模式 |
| lodash override 4.18.1 | `grep "lodash" package.json` | **仍然存在**：`"lodash": "4.18.1"` 未改 | 供应链风险持续存在，4.18.1 在 npm 上仍然不存在 |
| 占位符 skill（unsloth/axolotl） | `cat skills/mlops/training/unsloth/SKILL.md` | **已修复**：文件为空（0字节）→ 实际上是从仓库移除了 skill 内容，目录结构仍在 | 作者选择清空而非填写，效果等同删除 |

### 4.2 架构演进
从 audit（2026-04-19）到现在：
- 添加了大量 RELEASE_v*.md 文件（v0.2.0 到 v0.14.0），说明项目发版节奏很快
- optional-skills 目录从 audit 时的 ~40 个扩展到 81 个，说明社区贡献活跃
- 添加了 `acp_adapter/`、`acp_registry/` 目录，说明项目在接入 Agent Communication Protocol 生态

**作者后来意识到什么**：optional-skills 隔离策略是一个好的设计决策——高风险的 curl|bash 安装说明被集中在 optional-skills 目录，可以单独审查。

### 4.3 新增的可学习模式
- `README.zh-CN.md` 中文文档表明项目有意进入中文市场
- `AGENTS.md` 文件出现，说明项目正在对齐「适合 AI agent 工作的仓库规范」
- ACP（Agent Communication Protocol）接入是新兴模式——agent 之间的标准化通信协议，值得关注

---

## 五、校准

### 5.1 我已经在做对的
1. **不用 curl|bash 安装**：echo-sleuth 通过 `claude plugin install` 分发，没有自制 shell 安装脚本，规避了最大的安全风险
2. **小规模但质量优先**：echo-sleuth 只有 4 个 skill，每个都有完整 frontmatter，比 hermes-agent 的 16 个 BUG 文件好得多
3. **本地优先**：我的插件不需要 exec() 加载外部代码，无 HERMES_HOME 类环境变量风险

### 5.2 挑战 / 验证
- **被挑战的假设**：「star 数高 = 质量好」。hermes-agent 有 94,145 stars，但安全评级是 BLOCKED（8 个 CRITICAL 发现），还有 16 个 BUG 文件。star 数反映的是知名度和功能覆盖，不是 NL 质量或安全性
- **被验证的做法**：「CI 自动校验 frontmatter」的价值——hermes-agent 的问题恰好是「没有自动化门槛」导致的。我在 claude-for-legal 里应该加上类似检查

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 claude-for-legal 里有多少 SKILL.md 缺必要的 frontmatter 字段
find /tmp/my-repos/MarkQWu-claude-for-legal -name "SKILL.md" | while read f; do
  if ! python3 -c "
import sys
content = open('$f').read()
required = ['name:', 'description:', 'version:']
missing = [r for r in required if r not in content[:500]]
if missing: print('$f MISSING:', missing)
" 2>/dev/null; then
    echo "error: $f"
  fi
done | head -20
```
命中后怎么办：批量补全缺失字段，优先处理 `name` 和 `description` 缺失的文件。

```bash
# 检查我的代码里有没有 shell=True + 外部输入的组合
grep -rn "shell=True" /tmp/my-repos/MarkQWu-*/  --include="*.py" 2>/dev/null
```
命中后怎么办：把外部字符串输入换成 `shlex.split()`，或加白名单验证。

```bash
# 检查 claude-for-legal 的 SKILL.md 里模糊量词
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|proper|relevant)\b' \
  /tmp/my-repos/MarkQWu-claude-for-legal --include="SKILL.md" | wc -l
```
命中后怎么办：把每个模糊词替换为「具体标准」，比如 `appropriate` → 「符合[具体条件]时」。

### 6.2 灵感 → 实施路径
1. **想法**：给 claude-for-legal 加 CI 脚本检查 SKILL.md frontmatter 合规性
   - **为何可行**：hermes-agent 的教训是没有 CI 门槛导致 16 个 BUG 文件。claude-for-legal 有 151 个 SKILL.md，更需要自动化检查
   - **第一步**：复用 `agents-in-the-wild/bin/nlpm-check`，在 claude-for-legal 的 `.github/workflows/` 里加一个 `nlpm-check.yml`（约 20 分钟）

2. **想法**：借鉴 software-development/* skill 的「4步流程 + 红旗列表」写法，提升 claude-for-legal 的 skill 质量
   - **为何可行**：法律工作流（合同审查、尽调）天然适合「步骤化」描述，和软件工程 TDD 的结构类似
   - **第一步**：挑一个最常用的法律 skill（如 `regulatory-legal/skills/policy-redraft`），对照 `hermes-agent/skills/software-development/test-driven-development/SKILL.md` 的结构重写（约 45 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 NousResearch/hermes-agent 的核心目的**：提供覆盖 20+ 领域的 AI skill 库 + 完整 agent 运行时

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是大型 skill 库（claude-for-legal 有 151 个 SKILL.md） | hermes通用；legal专领域 | 高（规模管理经验） |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 SKILL.md | echo-sleuth 是精选小工具集，不是全能平台 | 中 |
| MarkQWu/drama-workshop-skills | 无 | — | 完全不同领域 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| frontmatter 缺 `author`/`license` | `grep -rL "^author:" skills/*/SKILL.md` | claude-for-legal 随机抽样 5 个：`regulatory-legal/skills/reg-feed-watcher/SKILL.md` 无 author 字段 | 中 |
| 缺 `## Pitfalls` / 故障排查节 | `grep -rL "Pitfall\|pitfall\|Troubleshoot" skills/*/SKILL.md` | claude-for-legal 多数 skill 无此节（目测 60%+ 缺失） | 高 |
| vague quantifiers | `grep -rn -E '\b(appropriate|comprehensive)\b' --include="SKILL.md"` | claude-for-legal 命中大量（legal 写作里这些词天然多） | 中 |

**命中后的具体行动建议**：
- `claude-for-legal/regulatory-legal/skills/policy-redraft/SKILL.md` → 加 `## Pitfalls` 节，列出 3 个法律 AI 常见失误（约 15 分钟）
- 在 claude-for-legal 的 contributing guide 里加一条：「每个 SKILL.md 必须有 author、license、pitfalls 字段」

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：软件工程 skill 的「流程步骤 + 红旗列表」写法
   - **本案例做法**：`skills/software-development/subagent-driven-development/SKILL.md` 有「4阶段流程图 + 5个红旗场景 + 完整示例工作流」
   - **我的项目现状**：`claude-for-legal/litigation-legal/skills/chronology/SKILL.md` 只有简单的步骤列表，无红旗场景
   - **如何借鉴**：给 chronology skill 加「常见失误（Red Flags）」节，列出「时间线构建失败的 3 个信号」

2. **领域**：optional-skills 隔离高风险内容
   - **本案例做法**：高安全风险或需要特殊依赖的 skill 放在 optional-skills/ 目录，默认不激活
   - **我的项目现状**：claude-for-legal 所有 skill 放在一起，用户一次性全部激活，没有分级
   - **如何借鉴**：把 claude-for-legal 里需要外部 API key 的 skill（如 reg-feed-watcher）移到 `optional/` 子目录，并在 README 里说明

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安装方式的安全性
- **我的做法**：echo-sleuth 和 claude-for-legal 都通过 `claude plugin install` 标准渠道分发，无自定义 shell 安装脚本，无 exec() 加载外部路径
- **本案例做法**（弱在哪）：curl|bash 主安装路径 + exec() over HERMES_HOME，是 BLOCKED 的直接原因
- **意义**：这是一个应该坚持的设计原则。如果未来 claude-for-legal 需要安装 CLI 工具，绝对不用 curl|bash，改用 `pip install` 或 `brew install` 等带完整性验证的包管理器

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块。hermes-agent 的 SKILL.md 要求标准的 frontmatter 里必须有 `name`、`description`、`version`、`author`、`license`、以及嵌套的 `metadata.hermes` 对象（含 `tags`、`related_skills`、`homepage`）。

### <a name="frontmatter方言"></a>frontmatter 方言
> 同一个项目里存在多种不同格式的 frontmatter。hermes-agent 有 3 种：(A) 标准的 `metadata.hermes.tags`；(B) 直接在顶层写 `tags:` 和 `triggers:`；(C) 只有 `name` 和 `description`，其余全无。方言 B/C 会让基于标准 schema 的搜索和工具失效。

### <a name="shell=True命令注入"></a>shell=True 命令注入
> Python 的 `subprocess.run(cmd, shell=True)` 会把 `cmd` 字符串传给系统 shell（bash/sh）执行。如果 `cmd` 来自外部输入（如用户安装的插件提供的字符串），攻击者可以在 `cmd` 里注入 `;rm -rf ~` 之类的命令，导致任意代码执行。修复方法：用 `subprocess.run(shlex.split(cmd))` 替代。

### <a name="供应链攻击"></a>供应链攻击
> 攻击者不直接攻击你的代码，而是攻击你的「依赖」——你使用的第三方库、安装脚本、CDN 等。lodash@4.18.1 在 npm 上不存在，如果有人发布了这个版本，所有使用 hermes-agent 并运行 `npm install` 的用户都会安装攻击者的包。这就是供应链攻击中的「typosquatting/version squatting」变体。

### <a name="curl-pipe-bash风险"></a>curl-pipe-bash 风险
> `curl URL | bash` 的安装方式：从网上下载一个脚本，立刻用 bash 执行，没有任何检查。风险在于：(1) 如果 URL 被劫持（DNS 污染、CDN 被攻击），用户执行的是攻击者的代码；(2) 没有校验和（checksum）验证，无法确认下载内容的完整性。更安全的做法是先下载，校验 SHA256 后再执行。

### <a name="exec注入"></a>exec() 注入
> Python 的 `exec()` 函数可以执行任意字符串作为代码。hermes-agent 的 godmode skill 用 `exec(open(HERMES_HOME + "/path/to/script").read())` 加载脚本，其中路径通过 `HERMES_HOME` 环境变量控制。如果攻击者能设置 `HERMES_HOME`（在 CI 环境或多租户环境里这是可能的），就能让 exec() 运行任意代码。
