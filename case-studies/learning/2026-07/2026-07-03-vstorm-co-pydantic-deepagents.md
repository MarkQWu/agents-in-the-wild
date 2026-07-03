# vstorm-co/pydantic-deepagents — 学习案例

**仓库**：https://github.com/vstorm-co/pydantic-deepagents
**Stars**：⭐687 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-03（基于当前 HEAD）
**主题标签**：`curl-pipe-bash-risk`, `security-gate`, `monorepo-vs-split`, `single-purpose`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

**一句话**：pydantic-deepagents（也叫 pydantic-deep）是一款 Python AI agent 框架，将 pydantic-ai SDK、CLI 工具、深度研究（deepresearch）、以及 Docker 容器化 agent 执行环境（harbor）打包在一起，并为 Claude Code 附带了一套配套技能（11 个 SKILL.md）。

关键事实：
- 定位是「面向开发者的 agent 开发工具包」，本体是 Python 代码（`apps/` 下），Claude Code skill 是附加产品
- 仓库含有真实的 Benchmark 评测数据（`jobs/terminal-bench/`）——这说明作者在认真做 agent 能力评测
- 安全政策完备：`SECURITY.md` 存在，有专用漏洞报告邮箱（info@vstorm.co），说明作者有安全意识
- 使用方式：通过 `install.sh` 安装（但此脚本是安全隐患所在），或通过 pip/uv 安装 `pydantic-deep`

### 1.2 架构剖析

```
pydantic-deepagents/
├── apps/
│   ├── cli/skills/           # CLI 配套技能（11 个 SKILL.md）
│   │   ├── code-review/      # 100分，示例 - 无输出格式
│   │   ├── test-writer/      # 100分
│   │   ├── skill-creator/    # 100分，内嵌模板
│   │   ├── verification-strategy/  # 88分，缺输出格式
│   │   └── ...（共 11 个）
│   ├── deepresearch/skills/  # 深度研究技能（3 个 SKILL.md）
│   │   ├── diagram-design/
│   │   ├── research-methodology/
│   │   └── report-writing/
│   └── harbor/agent.py       # Docker 容器 agent（⚠️ curl|sh 在此）
├── pydantic_deep/bundled_skills/  # 与 apps/cli/skills/ 完全同步的副本（11 个）
├── examples/
│   ├── skills/code-review/   # 示例技能（含 example_review.md）
│   └── full_app/skills/      # 完整示例 app 技能（含 example_review.md）
├── install.sh               # ⚠️ curl|sh 在第 38 行
├── pyproject.toml           # Python 包配置
└── SECURITY.md              # 安全政策（漏洞报告邮箱）
```

- **文件类型分布**：31 个 SKILL.md（11 个 cli + 11 个 bundled 副本 + 3 个 deepresearch + 2×3 个 examples）+ 1 个 CLAUDE.md
- **编排关系**：skills 是独立平铺，无路由层，无 agent 定义，无 commands；技能库与 Python 框架分离
- **跨件契约**：`apps/cli/skills/` 和 `pydantic_deep/bundled_skills/` 保持字节对齐同步（含同步脚本），frontmatter 中无跨件引用

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「最佳实践即产品」——把技能库作为框架文档的一部分，用户安装 pydantic-deep 就顺带获得一套 Claude Code 最佳实践技能；无需单独维护
- **解决什么问题**：pydantic-ai 开发者在写 agent 代码时需要配套的 AI 辅助（代码审查、测试生成、性能优化）；与其让用户自己写 skill，不如框架自带
- **做了什么 trade-off**：把技能打进框架（统一发布/更新）换来了技能和框架版本耦合——框架大版本升级时，技能可能也需要更新，不能独立演进
- **反映什么认知模型**：作者把「开发工具」和「AI 辅助提示」视为同等重要的产品组件，而不是把 AI 提示当作「文档的一部分」随手写

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「框架伴随技能库（Bundled Skill Library）」

Python 框架的正式发行版附带一套 Claude Code skill，技能知识与框架版本同步发布、同步更新。技能不是独立产品，而是框架使用者的「标配 AI 辅助包」。

模式特征清单：
- 特征 1：技能内容与框架知识强绑定（skill-creator 知道 pydantic-deep 的技能格式；code-review 知道 pydantic-ai 的 API 模式）
- 特征 2：技能库有两份「物理副本」（apps/cli/ + bundled/），随框架包分发
- 特征 3：0 个 command，0 个 agent——只有 skill；用户自己决定何时调用
- 特征 4：`skill-creator/SKILL.md` 内嵌一个新技能的模板，实现「自举」——用自己的 skill 格式来教用户写新技能
- 特征 5：examples/ 目录下有「比生产质量略低的示例技能」，帮助用户学习（付出了 examples 需要单独维护的代价）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有明确 API/框架知识点需要 AI 辅助理解的 SDK | ✅ 高度适用 | 技能知识与框架深度绑定，效果比通用 skill 好 |
| 纯业务逻辑应用（无框架知识要求） | ❌ 不适用 | 通用 skill 更合适，无需捆绑 |
| 需要独立发布/安装 skill 的场景 | ⚠️ 谨慎 | 技能随框架发布，无法单独安装 |
| 需要频繁迭代技能的场景 | ⚠️ 谨慎 | 每次技能更新需要发一个框架版本 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 框架伴随技能库（本仓库） | pydantic-deepagents | 技能知识精准，与框架版本同步 | 技能无法独立发布，更新耦合 |
| 独立 skill 插件 | swiftui-pro | 独立安装更新，与框架解耦 | 需要单独维护，知识可能滞后框架 |
| 多平台发行版 | autoresearch | 一套逻辑多平台覆盖 | 同步脚本维护成本 |
| 中央化技能市场 | claude-code-best-practice | 社区贡献，覆盖广 | 质量参差不齐 |

### 2.4 改进空间

1. **当前问题**：`apps/cli/skills/` 和 `pydantic_deep/bundled_skills/` 是两份字节对齐的物理副本，任何改动需要改两处（或依赖同步脚本）。**改进做法**：用 symlink 或 build step（`cp -r apps/cli/skills pydantic_deep/bundled_skills`）替代手工维护。**预期收益**：消除 16 个「重复fix」的维护负担，单一事实源。

2. **当前问题**：`install.sh` 第 38 行仍有 `curl -LsSf https://astral.sh/uv/install.sh | sh`（当前 HEAD 确认）。**改进做法**：改用「下载→验证 checksum→执行」三步，或直接用 `pip install uv`（不依赖 curl|sh）。**预期收益**：消除供应链攻击面，通过 NLPM 安全审查，有资格打开对外 PR。

3. **当前问题**：`apps/deepresearch/` 下的 3 个 SKILL.md 缺 `version` 和 `tags` frontmatter 字段（`diagram-design` 还缺 `auto_load`）。**改进做法**：统一对齐到 `apps/cli/skills/` 的 frontmatter 模式。**预期收益**：消除跨 skill 组的 frontmatter 不一致。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **93/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `apps/cli/skills/verification-strategy/SKILL.md` | 88/100 | 缺输出格式；"reasonable"（-12）|
| `apps/cli/skills/{build-and-compile,...7 个}/SKILL.md` | 90/100 | 缺输出格式节（-10）|
| `examples/full_app/skills/code-review/SKILL.md` | 92/100 | 坏引用 `example_review.md`；模糊量词（-8）|
| `CLAUDE.md` | 95/100 | "sensible defaults"（-2）|
| `apps/cli/skills/{code-review,test-writer,skill-creator}/SKILL.md` | 100/100 | 完美 |
| Security | **BLOCKED** | 2 处 CRITICAL：install.sh + harbor/agent.py 各 1 处 curl\|sh |

### 3.2 当时值得借鉴的模式

1. **skill-creator 自举**：`apps/cli/skills/skill-creator/SKILL.md` 内嵌了「新建 pydantic-deep 技能」的完整模板，用工具自身来教人写工具。这是「示例驱动（examples-driven）」模式的极致——最好的文档就是能用的模板。

2. **100 分的 code-review/SKILL.md**：`apps/cli/skills/code-review/SKILL.md` 得了满分——精确的输出格式、无模糊量词、完整 frontmatter。是整个仓库 31 个技能中的标杆，可直接作为写 code-review skill 的参考模板。

3. **SECURITY.md + 专用邮箱**：有正式的安全漏洞报告渠道（info@vstorm.co），说明仓库有安全意识。这是开源项目的基础设施，应成为所有 NL artifact 仓库的标配。

4. **frontmatter 字段完整性（cli 技能）**：`apps/cli/skills/` 的所有技能都有 `version: "1.0.0"`、`tags: [...]`、`auto_load`，形成了统一的 frontmatter 规范，通过标准化字段集提高了可维护性。

5. **examples/ 与生产 skills/ 分离**：示例技能放在 `examples/` 下，明确标记为教学用途而非生产质量，避免用户误把示例当生产技能用。

### 3.3 当时的缺陷

1. **2 处 CRITICAL 级 curl|sh**：`install.sh:38` 和 `harbor/agent.py:320` 各有一处 `curl | sh`，下载远程脚本直接执行，无 checksum 校验。根本原因：`uv`（一款 Rust 写的 Python 包管理器）官方安装脚本就是 curl|sh 格式，作者直接引用了上游的做法。危险在于：如果 astral.sh（uv 提供商）的 CDN 被攻击，或网络被中间人攻击，任意代码可在用户机器上执行。自查：我的仓库无 install.sh，这个问题不存在，但如果未来要写安装脚本，必须加 checksum 验证。

2. **9 个 cli skill 缺输出格式节**：`verification-strategy`、`build-and-compile` 等 9 个技能缺 `## Output Format` 节，AI 输出什么格式由模型自行决定，每次运行结果不一致。根本原因：作者在写这些技能时关注了「做什么」，忽略了「输出什么」的显式约定。自查：我的 gstack 也是相同问题（52 个 SKILL.md 全部缺 Output Format）。

3. **examples/ 技能质量低于 apps/**：`examples/code-review/SKILL.md` 比 `apps/cli/skills/code-review/SKILL.md` 质量差（有更多模糊量词，有坏引用），用户学习时可能从「低质量示例」开始。根本原因：示例和生产版本是两份独立文件，示例的更新优先级低，导致两者分化。自查：我的 graphify 有 `skill.md` 和 `skill-opencode.md` 两份，需要检查是否存在类似质量分化。

### 3.4 当时的优化机会

1. **安全修复**（私下报告）：fix `install.sh:38` 和 `harbor/agent.py:320` 的 curl|sh，改为 checksum 验证安装
2. **DRY**：将 `apps/cli/skills/` 和 `pydantic_deep/bundled_skills/` 合并为单一事实源
3. **为 9 个技能添加 `## Output Format` 节**：参考仓库自己的 100 分 `code-review/SKILL.md` 的格式

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `install.sh:38` curl\|sh | `grep -n "curl.*sh" install.sh` | **仍存在**（第 38 行）；注释里（第 5 行）还有另一处引用 | CRITICAL 安全问题未解决；仓库自加了 SECURITY.md 但核心问题未动 |
| `harbor/agent.py:320` curl\|sh | `grep -n "curl.*sh" apps/harbor/agent.py` | **仍存在**（第 320 行，在 Docker init 模板字符串内） | 同上；影响所有 harbor 容器实例 |
| `examples/code-review/` 缺 `example_review.md` | `ls examples/full_app/skills/code-review/` | **已修复**：`example_review.md` 现在存在 | 技术 bug 被修复；但 CRITICAL 安全问题未动，说明作者有维护但有优先级选择 |
| 重复技能树 | `diff apps/cli/skills/ pydantic_deep/bundled_skills/` | **仍存在**（diff 输出无差异，仍为物理副本） | 架构问题通常难以快速解决 |

### 4.2 架构演进

- **新增**：`SECURITY.md`（安全政策），`jobs/terminal-bench/` Benchmark 评测结果文件
- **变化**：版本升至 0.3.x（`SECURITY.md` 声明只支持 0.3.x 的安全修复）
- **不变**：技能树结构、curl|sh 问题、重复技能副本

Benchmark 数据的存在是一个积极信号：作者在做系统化评测（而非只靠直觉）。这类「评测驱动开发」是 agent 框架成熟的标志。

### 4.3 新增的可学习模式

**Benchmark 评测数据随仓库保存**（audit 时未覆盖）：

`jobs/terminal-bench/` 下保存了 terminal-bench 的评测 run 数据（config.json + trial.log + ctrf.json），说明作者把评测结果作为仓库的一部分，而非存在外部服务。这种「评测即文档」的做法让用户/贡献者可以复现评测，验证框架能力。

---

## 五、校准

### 5.1 我已经在做对的

1. **无 curl|sh 安装脚本**：我的所有仓库均无 `install.sh`，没有这个风险
2. **单一事实源**：gstack 每个技能只有一份 SKILL.md，没有 cli/ 和 bundled/ 双份同步的问题
3. **SECURITY.md**：暂无（这是我需要补的，参见 §6.2）

### 5.2 挑战 / 验证

这次案例挑战了一个假设：**「高 NLPM 分（93/100）不等于安全」**。pydantic-deepagents 的技能质量相当好（多个 100 分技能），但同时有 CRITICAL 级安全问题——这两个维度是独立的，不能用 NL 质量分来代替安全扫描。

另一个反思：audit 时发现的 `example_review.md` 坏引用已被修复，但 curl|sh 的 CRITICAL 问题未动。这告诉我：**维护者修 bug 的优先级不是按严重度，而是按修复难度**——坏引用是一个文件的事，curl|sh 需要重设计安装流程，所以坏引用先修了。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的仓库是否有安装脚本含 curl|sh
find /tmp/my-repos/MarkQWu-gstack \
     /tmp/my-repos/MarkQWu-bureau \
     /tmp/my-repos/MarkQWu-graphify \
     -name "install.sh" -o -name "setup.sh" | \
     xargs grep -l "curl.*sh" 2>/dev/null
# 命中后：重写安装步骤，用 checksum 验证替代直接 pipe to sh

# 检查我的 skill 是否有 examples/ 和生产版质量分化
diff /tmp/my-repos/MarkQWu-graphify/graphify/skill.md \
     /tmp/my-repos/MarkQWu-graphify/graphify/skill-opencode.md 2>/dev/null | head -20
# 命中后：哪个是生产版？哪个是 OpenCode 适配版？确保两者主体内容一致

# 检查我的 repo 是否有 SECURITY.md
for repo in /tmp/my-repos/MarkQWu-*; do
  if [ ! -f "$repo/SECURITY.md" ]; then
    echo "缺 SECURITY.md: $(basename $repo)"
  fi
done
# 命中后（几乎全部）：至少在最活跃的仓库（gstack/bureau）加 SECURITY.md
```

### 6.2 灵感 → 实施路径

1. **想法**：为 graphify 检查 `skill.md`（claude版）与 `skill-opencode.md`（opencode版）的质量是否分化
   - **为何可行**：graphify 和 pydantic-deepagents 一样，有多发行版的技能文件
   - **第一步**：diff 两个文件，找出哪些内容在 OpenCode 版本里缺失或降级，统一到 Claude 版的质量水平（10-15 分钟）

2. **想法**：为 bureau 添加 `SECURITY.md`
   - **为何可行**：bureau 有 agent 会读写用户文件，有漏洞报告渠道是基础设施
   - **第一步**：新建 `SECURITY.md`，参考 vstorm 的格式（声明支持版本 + 漏洞报告邮箱），5 分钟内完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：为 Python agent 框架配套提供 Claude Code 技能集，知识与框架版本绑定

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | 同为 AI coding assistant 技能，都有多版本（claude/opencode）分发 | graphify 不是 Python 框架的伴随技能，是独立插件 | 高 |
| MarkQWu/bureau | 中 | 都有 skills/ 目录 + plugin.json | bureau 无多发行版问题 | 低 |
| MarkQWu/gstack | 低 | 都是 SKILL.md 集合 | gstack 是个人工具集，无框架绑定 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 缺 `## Output Format` 节 | `grep -L "## Output Format" gstack/*/SKILL.md` | **gstack 52/52 命中**（全部缺少） | 高 |
| 多发行版质量分化 | `diff graphify/skill.md graphify/skill-opencode.md` | **graphify 需检查**：两文件内容不同，需确认是否有质量分化 | 中 |
| 无 SECURITY.md | `ls bureau/SECURITY.md` | **bureau、gstack、graphify、echo-sleuth 均无** | 低（但基础设施）|

**命中后的具体行动建议**：
- `MarkQWu-gstack/review/SKILL.md` 等最常用的 10 个 → 优先追加 `## Output Format` 节（每个 5-10 分钟）
- `MarkQWu-graphify/graphify/skill-opencode.md` → diff 比对 claude 版，补齐缺失内容（10 分钟）
- `MarkQWu-bureau/SECURITY.md` → 新建，写支持版本 + 邮箱（5 分钟）

### 8.3 别人的更优方案

1. **领域**：skill-creator 自举模板
   - **本案例做法**：`apps/cli/skills/skill-creator/SKILL.md` 内嵌了一个完整的「新建技能模板」作为输出示例，AI 读完这个 skill 就知道如何生成符合格式的新 SKILL.md
   - **我的项目现状**：gstack 无 `skill-creator` 类工具；bureau 有 `skills/scribe/SKILL.md` 负责写作，但没有「生成新 SKILL.md」的专用 skill
   - **如何借鉴**：在 gstack 或 bureau 下新建 `skills/skillify/SKILL.md`，内嵌符合 gstack 规范的 SKILL.md 模板作为 Output Format 的 `## Example` 部分（30 分钟可完成）

2. **领域**：满分 code-review SKILL.md 作为格式标杆
   - **本案例做法**：`apps/cli/skills/code-review/SKILL.md` 100/100 分——无模糊量词、明确输出格式、完整 frontmatter；整个仓库把它当内部标准
   - **我的项目现状**：gstack 的 `review/SKILL.md` 有 18 处模糊量词命中，且无 Output Format 节
   - **如何借鉴**：以 vstorm 的 code-review SKILL.md 为模板，重写 gstack/review/SKILL.md（明确审查维度，加 Output Format 节，去模糊量词）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：单一事实源（无重复技能副本）
- **我的做法**：gstack 每个技能只有一份 SKILL.md，bureau 同理；不存在 apps/cli/ 和 bundled/ 的双份同步问题
- **本案例做法（弱在哪）**：31 个 SKILL.md 中有 11 个是 cli 版的字节对齐副本，任何修复需要改两处（或依赖同步脚本），且同步脚本本身有安全隐患
- **意义**：「单一事实源」是软件工程的基本原则，在 NL artifact 管理中同样适用

---

## 八、术语表

### <a name="curl-pipe-bash"></a>curl|bash 反模式
> `curl https://example.com/install.sh | sh` 这个命令组合：先从网络下载一个脚本，再立即执行。危险在于：下载和执行之间没有任何检查（内容是什么、是否被篡改），如果服务器被攻击、CDN 被污染、或网络被中间人攻击，任意代码都可以在你的机器上以你的权限运行。安全做法是：下载 → 验证 checksum（SHA256 对比）→ 人工确认 → 执行。

### <a name="supply-chain-attack"></a>供应链攻击
> 不直接攻击你，而是攻击你**依赖的东西**（一个 npm 包、一个安装脚本、一个 CDN 上的文件），让你在运行「正常操作」时中招。curl|bash 的风险就是一种供应链攻击面：攻击者只需要攻陷 astral.sh 或劫持 DNS，就能在所有运行 install.sh 的机器上执行任意代码。

### <a name="dry"></a>DRY（Don't Repeat Yourself）
> 软件工程原则：同样的知识或逻辑只应该在一个地方存在。pydantic-deepagents 里 `apps/cli/skills/` 和 `pydantic_deep/bundled_skills/` 是 DRY 的违反——同一份技能存了两份，任何修复需要改两次。Symlink 或 build 脚本可以恢复 DRY。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 元数据。`version`、`tags`、`auto_load` 等字段写在这里。pydantic-deepagents 的问题之一是 `apps/deepresearch/` 的 3 个技能缺少 `version` 和 `tags`，与 `apps/cli/` 的技能 frontmatter 不一致。

### <a name="benchmarking"></a>Benchmark 评测
> 用标准化测试集系统地量化 agent 能力。pydantic-deepagents 的 `jobs/terminal-bench/` 目录保存了 TerminalBench 评测结果（config.json 定义测试，trial.log 记录执行，reward.txt 记录得分）。把评测结果放进仓库让能力可复现、可比较——这是 agent 框架成熟的标志。
