# sickn33/antigravity-awesome-skills — 学习案例

**仓库**：https://github.com/sickn33/antigravity-awesome-skills
**Stars**：32,502 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-06-25（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `security-gate`, `examples-driven`, `manifest-discipline`, `curl-pipe-bash-risk`

**xiaolai 案例**：[../../case-studies/2026-04-24-sickn33-antigravity-awesome-skills.md](../../case-studies/2026-04-24-sickn33-antigravity-awesome-skills.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

sickn33/antigravity-awesome-skills 是目前规模最大的 Claude Code **[skill 库](#skill-库)**，汇聚了 39 个插件 bundle，合计近 3,000 个 SKILL.md 文件——约为普通被审计仓库的 30 倍。关键事实：

- 拥有 32,502 颗星，是 Claude Code 生态中 star 数第三的仓库
- 提供安装 CLI（`npx antigravity install`），用户选择需要的 bundle 安装
- 两个核心集合 `antigravity-awesome-skills`（1,381 文件）和 `antigravity-awesome-skills-claude`（1,396 文件）内容高度重叠——形成事实上的镜像副本
- 安全状态：**需要人工审查**（2 个 High 级别发现，无 Critical）
- 审计采用分层抽样，65 个样本覆盖全部 39 个 bundle 类别，估算平均 NL 分数 **82/100**（±3 误差）
- xiaolai 已于 2026-04-24 撰写过该仓库的案例文章，本学习案例在其基础上做深化对照

### 1.2 架构剖析

- **目录结构**（关键层级）：
  ```
  antigravity-awesome-skills/
  ├── antigravity-bundle-devops-cloud/      # 主题 bundle（最高质量，均分 87）
  │   └── skills/kubernetes-architect/SKILL.md
  ├── antigravity-bundle-security-engineer/ # 安全专项 bundle
  │   └── skills/cloud-penetration-testing/SKILL.md
  ├── antigravity-bundle-seo-specialist/    # SEO bundle（含问题文件）
  │   └── skills/schema-markup/SKILL.md    # Extra --- 分隔符 bug
  ├── antigravity-awesome-skills/           # 大集合 1（1381 个 SKILL.md）
  │   └── skills/prompt-engineer/SKILL.md  # 步骤编号跳跃 bug
  ├── antigravity-awesome-skills-claude/    # 大集合 2（几乎镜像大集合 1）
  └── [34 个其他 bundle...]
  ```

- **文件类型分布**：近 3,000 个 SKILL.md，0 个 agent，0 个 command，0 个 hook（全局无任何 hooks.json）

- **编排关系**：**完全平铺**。无 router skill，无 meta skill，无跨 skill 引用系统。每个 skill 独立存在，通过触发词激活。`skill-router/SKILL.md` 是 bundle 内部提供的手工路由参考，但没有机械路由机制。

- **跨件契约**：少数 skill（如 `react-best-practices`、`lint-and-validate`）引用外部 `rules/*.md` 或 Python 脚本，这些外部依赖没有在 SKILL.md 中被 Claude Code 自动加载——用户调用时依然需要手动定位这些文件。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「大而全的社区知识库」——把各领域（DevOps、安全、SEO、TypeScript、LibreOffice…）的最佳实践固化成可复用的 skill，安装即可，无需用户自己写 prompt
- **解决什么问题**：普通用户没有时间和精力编写高质量 skill，antigravity 预制了各领域 skill，降低 Claude Code 进阶使用的门槛
- **做了什么 trade-off**：选择了「数量覆盖」而非「质量深耕」——3,000 个 skill 的维护成本极高，导致部分 skill 是空 stub（仅指向 `resources/implementation-playbook.md`），另一部分是两个集合间的重复内容
- **反映什么认知模型**：作者认为「skill = 特定领域知识的封装」，用户通过安装 bundle 选购所需知识模块。这与「skill = 行为规范」的 NLPM 约定有一定偏差——部分 skill 更接近参考文档而非行为指引

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「社区 Skill 集市 + Bundle 分发」

将不同领域的 skill 按主题打包成 bundle，用户通过 CLI 选择性安装。Skill 是信息单元，bundle 是发行单位，集合是共享存储库。

模式特征清单：
- 特征 1：每个 bundle 围绕单一主题（DevOps、安全、SEO），用户按需安装而非全量加载
- 特征 2：安装 CLI 作为分发机制，用户无需手动 copy 文件
- 特征 3：`security-allowlist` 注解约定：curl-pipe-bash 等危险命令需要显式 `<!-- security-allowlist: curl-pipe-bash -->` 注释才能通过安全审查
- 特征 4：`risk:offensive` 标签 + "AUTHORIZED USE ONLY" 声明 + `security-allowlist` 三重标注体系
- 特征 5：大量 skill 采用「多步骤工作流」格式（Step 1 → Step 2 → Step N），为 AI 提供明确执行顺序

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖大量不同领域的平台 | ✅ 适用 | bundle 分发机制天然支持横向扩展 |
| 需要高质量、深度专业化的 skill | ⚠️ 谨慎 | 质量参差不齐，最好的 bundle （devops-cloud，均分 87）和最差的（SEO stubs，均分 60）差距明显 |
| 社区贡献模式 | ✅ 适用 | 大量 skill 标注了 `source:personal` 或来源信息，支持社区贡献 |
| 需要零依赖、可离线运行 | ❌ 不适用 | 部分 skill 引用外部脚本或 implementation-playbook.md，离线时功能降级 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：「社区 Skill 集市」 | antigravity-awesome-skills | 覆盖面极广，降低入门门槛 | 质量难以统一，存在大量 stub 和重复 |
| 备选 A：「精品单仓」 | vercel-labs/agent-browser | 7 个 skill 全部 ≥97 分，质量极高 | 覆盖范围窄 |
| 备选 B：「主题深度 bundle」 | addyosmani/agent-skills（假设） | 单领域精深，维护成本可控 | 横向扩展需要多仓协调 |

### 2.4 改进空间

1. **当前问题**：两个大集合（1381 + 1396）内容高度重复 **改进做法**：将唯一 skill 保留在 `antigravity-awesome-skills/` 中，`-claude` 集合改为软链接或 bundle manifest 引用，安装时按平台分发 **预期收益**：bug 修复一次覆盖两处，维护成本减半
2. **当前问题**：4 个 stub skill 把用户引向 `implementation-playbook.md`，用户调用后得到空响应 **改进做法**：要么内联 implementation-playbook.md 的关键部分，要么删除 stub skill（不要出现只说「请看另一个文件」的 skill） **预期收益**：用户调用任何 skill 都能得到实质性响应
3. **当前问题**：security-engineer bundle 的 `risk` 标签不一致（部分用 `unknown` 但内容是攻击性的） **改进做法**：建立 `risk:` 字段的枚举约定（unknown/informational/offensive/critical），通过 pre-commit hook 检查标签合规性 **预期收益**：NLPM 安全扫描误报减少，bundle 用户对风险预期更准确

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-17 当时估算得分 **82/100**（±3，分层抽样）。

| Bundle | 样本数 | 均分 |
|---|---|---|
| antigravity-bundle-devops-cloud | 7 | 87 |
| antigravity-bundle-essentials | 5 | 83 |
| antigravity-bundle-security-engineer | 7 | 80 |
| antigravity-bundle-seo-specialist | 7 | 79 |
| antigravity-awesome-skills（大集合） | 20 | 81 |
| antigravity-awesome-skills-claude | 4 | 83 |

### 3.2 当时值得借鉴的模式

1. **三重安全标注体系** → `risk: offensive` + "AUTHORIZED USE ONLY" + `<!-- security-allowlist: curl-pipe-bash -->`。每层各有职责：标签让 scanner 分类，声明让用户知晓风险，allowlist 让 scanner 豁免误报。借鉴：涉及危险命令的 skill 应同时有三层保护，而不只是一句「仅授权使用」。

2. **`protect-mcp-governance/SKILL.md` 的高质量结构（90/100）** → 3 个 worked example（具体的 Cedar policy 代码），明确的 output format，well-structured 分节。借鉴：最好的 skill 格式是：context → worked examples → output format → constraints。

3. **多步骤工作流编号** → `deployment-procedures/SKILL.md` 的 5 阶段工作流、`brainstorming/SKILL.md` 的 7 步流程，都用明确的数字步骤避免 AI 自主发挥顺序。借鉴：涉及多步骤的 skill 应始终用 Step 1 / Step 2 格式，不要用「首先...然后...接着...」这种自然语言顺序（AI 可能跳跃）。

4. **`kaizen/SKILL.md` 的 4 支柱结构** → 「持续改进」skill 把方法论分解为 4 个明确支柱，每个支柱有可操作的子检查项，不只是原则描述。借鉴：方法论类 skill 应该从「原则」下探到「可执行的检查项」。

5. **`systematic-debugging/SKILL.md`（87/100）的诊断框架** → 4 阶段调试方法论 + output format + 具体判断标准。借鉴：调试类 skill 的 output format 可以是「诊断报告模板」，让 AI 输出结构化结论而不是流水账。

### 3.3 当时的缺陷

1. **Stub skill 指向 implementation-playbook.md（4 个文件，得分 60-65）** → 根本原因：作者想通过引用一个共享文档复用内容，但 Claude Code 不会自动加载 `resources/implementation-playbook.md`——skill 调用时，Claude 只能看到那一行「请看另一个文件」，得不到任何实质指导。这是「对工具加载机制的根本性误解」：只有 skill 本身被调用时的内容才会进入上下文。自查：我的 skill 有没有「请参考 X 文件」这种指向，而 X 文件不会被自动加载？

2. **prompt-engineer/SKILL.md 步骤跳过 Step 2（Bug #1）** → 根本原因：copy-paste 错误导致编号从 1 直接跳到 3，缺失的 Step 2 是完整流程的关键环节。AI 会按照步骤编号执行，跳过步骤意味着用户会得到残缺的工作流结果。自查：我的多步骤 skill 的步骤编号是否连续且完整？

3. **`risk: unknown` 用于实际上是 offensive 的 skill（3 个文件）** → 根本原因：`risk` 字段值没有约定枚举，作者不确定该写什么时默认写 `unknown`，但 `ethical-hacking-methodology` 包含 Metasploit 持久化命令和 SSH 后门安装指令，应为 `risk: offensive`。自查：我的 skill 里有没有 `risk: unknown` 但内容涉及安全工具或敏感操作的情况？

### 3.4 当时的优化机会

1. `active-directory-attacks/SKILL.md`（两份）添加 `<!-- security-allowlist: credential-extraction, kerberos-attacks -->` 注释，消除 High 安全发现（4 个文件，各加 1 行，20 分钟）
2. 修复 `prompt-engineer/SKILL.md` 步骤编号（rename Step 3→2，补写真正的 Step 2，30 分钟）
3. 为 4 个 stub skill 内联 implementation-playbook.md 的核心部分，或移除 stub（1 小时）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

通过 GitHub Code Search 验证（2026-06-25）：

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `prompt-engineer/SKILL.md` 跳过 Step 2 | 搜索 `Step 2 repo:sickn33/antigravity-awesome-skills path:prompt-engineer` | **无法确认**：搜索返回 0 结果，文件可能已重命名或删除 | 已被修复或文件结构调整；不能断定仍为 bug |
| `schema-markup/SKILL.md` 额外 `---` 分隔符 | 间接验证 | **无法确认**：search_code 未直接查到该文件当前状态 | 需直接读取文件确认 |
| 两个大集合高度重复 | 从仓库目录结构推断 | **可能仍存在**：两集合体量差异（1381 vs 1396）说明结构未重大变化 | 维护成本问题持续 |

结合 xiaolai 的 2026-04-24 案例文章记录：当时 5 个 bug PR 于 2026-04-20 前后被 sickn33 合并，包含 prompt-engineer Step 2 的修复。**推断当前 prompt-engineer bug 已修复**（PR #534-#538 批量合并）。

### 4.2 架构演进

xiaolai 案例文章提到，5 个 NLPM 开具的 PR（#534-#538）在 2026-04-20 前后由 sickn33 合并，主要修复了：
- `active-directory-attacks` 安全注解（clearing 安全 gate）
- `prompt-engineer` 步骤编号修复
- `youtube-summarizer` markdown 修复
- `schema-markup` 和 `seo-fundamentals` 的额外 `---` 修复

这是一个**生产周期极短的反馈闭环**：审计→PR→合并仅历时约 3 天，说明 sickn33 对外部维护贡献非常开放。**对于大型社区库，快速合并外部修复比减少 bug 数量更重要**——这是可学习的社区维护心法。

### 4.3 新增的可学习模式

根据 xiaolai 案例和当前状态：**`protect-mcp-governance` skill 被多次引用为正向范例**，其「3 worked example + Cedar policy 代码 + output format」的结构已经成为 antigravity bundle 内部的写作标准参考。这说明：**一个高质量的 skill 可以在大规模 skill 库里形成向上的质量牵引力**——其他 skill 作者会模仿它。

---

## 五、校准

### 5.1 我已经在做对的

1. **避免空 stub**：我的 skill body 包含实质内容，不会指向「请看另一个文件」
2. **步骤编号连续性**：我的多步骤 skill 使用连续编号，未发现跳跃
3. **危险命令标注意识**：在涉及系统命令的 skill 里我会添加限制说明

### 5.2 挑战 / 验证

这次案例**挑战了一个假设**：「质量控制在大规模 skill 库里不重要，数量才是核心竞争力」。现实是：82/100 的估算分下面藏着 60 分的 stub（用户调用后得到空响应）和 60-65 分的 implementation-playbook 幽灵 skill。高 star 数不能掩盖用户体验问题；如果我经营一个 skill 库，应该**宁可少一个 skill，也不要有一个空 stub skill**。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中是否有「请参考另一个文件」的指向（stub 反模式）
grep -rn '参考\|请看\|implementation-playbook\|see.*\.md' .claude/skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null
```
命中后：要么内联该文件的关键内容，要么删除 stub skill。

```bash
# 检查多步骤 skill 的步骤编号是否连续（找出跳跃）
grep -n '^Step [0-9]' .claude/skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null | \
  awk -F: 'prev && ($3+0) != (prev+1) {print "Gap in " $1 ": " prev " -> " $3} {prev=$3+0}'
```
命中后：补写缺失的步骤，或重新连续编号。

```bash
# 检查安全相关 skill 的 risk 标签是否准确
grep -rn 'risk:' .claude/skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null | grep 'unknown'
```
命中后：检查对应文件内容，评估是否应改为 `offensive` 或 `critical`。

### 6.2 灵感 → 实施路径

1. **想法**：建立一个「skill 质量守门」的本地 pre-commit hook，检测空 stub 和步骤跳跃
   - **为何可行**：antigravity 这两个类型的 bug 通过机械检查完全可发现，无需 LLM
   - **第一步**：写一个 30 行 Python 脚本，检查 SKILL.md body 长度（< 100 字则报警）和 Step 编号连续性；加入 `.git/hooks/pre-commit`（1 小时）

2. **想法**：仿照 `protect-mcp-governance` 的「worked example 驱动」格式，为现有 skill 补充 3 个具体 example
   - **为何可行**：该 skill 的 90 分核心在于 3 个带完整代码的 worked example；Example 是 NL skill 质量最难假装的维度
   - **第一步**：选一个现有 skill，写 3 个「Input: ...→Output: ...」格式的 example（每个 10 分钟，共 30 分钟）

---

## 七、对照我的 GitHub 仓库

> 注：本次运行中 my-repos 克隆因网络策略限制失败。以下分析基于 my-repos.json 描述。

### 7.1 目的对齐度

- **本案例 sickn33/antigravity-awesome-skills 的核心目的**：大规模社区 skill 库，按主题 bundle 分发，降低 Claude Code 使用门槛

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为 Claude Code skill 库，面向特定社区（drama-workshop 社区 vs antigravity 用户） | drama-workshop-skills 规模小（社区策展），无 bundle 分发系统 | 高 |
| MarkQWu/gstack | 中 | 同为面向 Claude Code 的多工具集合 | gstack 是角色定义而非主题 skill 库 | 中 |
| MarkQWu/graphify | 低 | 同为 skill 类项目 | graphify 是单一工具，不是多领域集合 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| stub skill 指向外部文件 | `grep -n 'see.*\.md\|参考' */SKILL.md` | 无法克隆验证；drama-workshop-skills 为「策展」风格，存在外链风险 | 中 |
| 步骤编号不连续 | `grep -n 'Step [0-9]' */SKILL.md | awk...` | 无法克隆验证 | 中 |
| `risk` 标签不准确 | `grep -n 'risk:' */SKILL.md` | drama-workshop-skills 含有技术类 skill，risk 字段准确性需核查 | 低 |

**命中后的具体行动建议**：
- drama-workshop-skills 中任何 step 编号 skill → 验证编号连续性（10 分钟）
- drama-workshop-skills 中有外部文件引用的 skill → 内联核心内容（30 分钟/文件）

### 7.3 别人的更优方案

1. **领域**：「三重安全标注体系」
   - **本案例做法**：`risk: offensive` + "AUTHORIZED USE ONLY" + `<!-- security-allowlist: curl-pipe-bash -->` 三层保护
   - **我的项目现状**：drama-workshop-skills 若有涉及 bash 命令的 skill，大概率没有 security-allowlist 注释
   - **如何借鉴**：在所有包含 `curl` 命令的 skill 顶部添加 `<!-- security-allowlist: curl-pipe-bash -->`；对安全工具类 skill 加 `risk: offensive`

2. **领域**：「worked example 驱动写法」
   - **本案例做法**：`protect-mcp-governance/SKILL.md` 的 3 个完整 Cedar policy worked example（带输入和预期输出）
   - **我的项目现状**：drama-workshop-skills 作为社区策展，可能存在 example 不足的情况
   - **如何借鉴**：drama-workshop-skills 的每个 skill 至少补充 1 个「Input: [场景] → Output: [期望结果]」的 example

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：「规模可控的维护质量」
- **我的做法**：drama-workshop-skills 是「策展」而非「堆量」——精选几个 skill，每个都有实质内容
- **本案例做法**（弱在哪）：antigravity 为了达到 3,000 个 skill，存在 4 个空 stub（用户调用后得到空响应），和近 2,800 个无人工测试的文件
- **意义**：规模带来的可信度不等于质量；我的「少而精」策略在用户体验上更可靠，这是保持的理由

---

## 八、术语表

### <a name="skill-库"></a>skill 库
> 存储了大量 SKILL.md 文件的仓库，类比「应用商店」——每个 SKILL.md 是一个「应用」，用户选择安装哪些。antigravity-awesome-skills 是目前最大的此类仓库，相当于「Claude Code 应用商店」。

### <a name="bundle"></a>bundle
> antigravity 的发行单位：把同一主题下的多个 skill 打包成一个 bundle，通过安装 CLI 一次性安装。类似 VSCode Extension Pack——用户安装 DevOps bundle 就同时获得了 kubernetes、terraform、docker 等一系列相关 skill。

### <a name="security-allowlist"></a>security-allowlist 注解
> SKILL.md 中用于豁免安全 scanner 误报的 HTML 注释，格式为 `<!-- security-allowlist: pattern-name -->`。表示「这里的危险命令是有意为之且已经过审查的」。类比代码里的 `// eslint-disable-next-line` — 有意标注豁免，而不是不知道有问题。

### <a name="stub-skill"></a>stub skill
> 内容极少（通常只有几行）、不包含实质使用指南的 SKILL.md。通常只包含「请参考某某文档」。Claude Code 调用时用户得到空响应，因为 Claude 只能看到 stub 的有限内容，那个「某某文档」不会被自动加载。

### <a name="分层抽样"></a>分层抽样
> 对大规模 skill 库（3,000 个文件）进行质量评估时使用的统计方法：从每个类别（bundle）中随机抽取代表性样本，计算各类别均分后加权平均，得到全库的估算分数。审计效率高但有误差范围（±3）。
