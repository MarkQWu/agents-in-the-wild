# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：0 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-28（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `examples-driven`, `vague-quantifier`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`affaan-m/everything-claude-code`（简称 ECC）是迄今为止 NLPM 审计过的**规模最大的 Claude Code NL 工件仓库**，包含 934 个工件（命令、agent、skill），覆盖 20+ 编程语言审查、多语言国际化、GAN 风格自进化 harness、企业级运营工具等场景。一句话定位：_"Claude Code 的瑞士军刀——什么都有，但刀锋质量参差不齐"_。

关键事实：
- 作者：affaan-m，活跃的 Claude Code 社区贡献者，内容持续迭代
- 规模：934 个工件，是第二名的 5-10 倍；跨 8+ 语言国际化（zh-CN、zh-TW、ko-KR、pt-BR、tr、ja-JP 等）
- 审计时状态：Security **BLOCKED**（7 个安全发现），NL 84/100，24 个 bug，187 个质量问题
- 当前 install.sh：已从 shell 脚本重写为 **Node.js 委托架构**（`install.sh → scripts/install-apply.js`），安全态势大幅改善
- 独特组件：GAN 风格 harness（`scripts/gan-harness.sh` + `agents/gan-generator/evaluator.md`），可以让生成器 agent 和评估器 agent 互相对抗来提升内容质量

### 1.2 架构剖析

```
everything-claude-code/
├── commands/               # ~50 个命令（code-review, tdd, plan, learn, orchestrate…）
├── agents/                 # ~35 个专项 agent（语言审查、角色扮演、GAN）
├── skills/                 # ~80 个 skill（按语言/领域分类）
│   ├── coding-standards/   # 100/100
│   ├── continuous-learning/ # 100/100
│   ├── dart-flutter-patterns/  # 84/100（vague: "appropriate"×8）
│   ├── gan-style-harness/  # 100/100
│   ├── lead-intelligence/  # 包含子 agent 和多个 skill（嵌套）
│   └── ...（80+ 个）
├── .kiro/agents/           # AWS Kiro IDE 兼容 agent 定义（缺 model 字段）
├── docs/                   # 国际化文档（ko-KR, zh-TW, zh-CN, pt-BR, tr, ja-JP）
│   └── {lang}/commands/    # 各语言版本的命令文件
├── hooks/                  # Claude Code hooks
├── scripts/
│   ├── gan-harness.sh      # GAN 自进化 harness（shell 脚本）
│   └── install-apply.js    # 主安装逻辑（Node.js，当前版本）
├── install.sh              # 现为 Node.js 委托入口（已重写）
├── the-security-guide.md   # 新增的 agent 安全深度文章
├── SOUL.md                 # 作者的 AI 哲学文档
├── WORKING-CONTEXT.md      # 开发上下文说明
└── schemas/                # JSON Schema 定义
```

- **文件类型分布**：~50 个 command / ~35 个 agent / ~80 个 skill / 国际化副本约 200 个 / .kiro 兼容层 ~15 个
- **编排关系**：几乎没有显式编排，各组件相对平立。但有几条隐式调用链：orchestrate.md → 多个专项 agent；gan-harness.sh → gan-generator + gan-evaluator；chief-of-staff.md → 多个 agent
- **跨件契约**：国际化副本（docs/zh-CN/\*.md 等）与英文原版并行维护，但不是代码生成的，而是手动翻译 → 容易漂移

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「社区驱动的 NL 工件集合」——与其说是精心设计的系统，不如说是"持续贡献的 NL 知识库"。每个工件解决一个具体问题，整体通过命名约定松散耦合
- **解决什么问题**：大多数开发者的 Claude Code 配置是临时性的、碎片化的；ECC 提供了一个"现成的"高质量起点，覆盖从代码审查到 GAN 自进化的全栈场景
- **做了什么 trade-off**：深度 vs 广度。ECC 选择了广度（934 个工件）——对任何需求都有一个起点，但很少有工件做到极致（多数 agent 缺 examples，score 73-88）
- **反映什么认知模型**：作者把 NL 工件看作"可以持续演化的软件"——SOUL.md 和 WORKING-CONTEXT.md 反映了作者对 AI 自进化系统的思考，GAN harness 是这个认知的落地实验

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「大规模平铺聚合」（Flat Aggregation Monorepo）**

关键特征：
- **特征 1**：无显式编排层，所有工件直接面向用户，用户按需取用
- **特征 2**：依靠命名约定（`lang-review.md`、`lang-build-resolver.md`）而非 manifest 引用来组织
- **特征 3**：国际化作为关键差异化：同一工件在 6+ 语言里都有副本
- **特征 4**：内嵌子系统（GAN harness、instinct/持久记忆、PRP 流程）作为可选增强模块
- **特征 5**：通过 `hooks/` 加入行为拦截，补充标准工件无法覆盖的场景

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 作为个人或团队的 Claude Code starter kit | ✅ 高度适用 | 开箱即用，覆盖广，可以按需裁剪 |
| 构建高内聚的专用工具（如 airis 网关） | ❌ 不适用 | 平铺聚合缺乏内聚性，不适合有强依赖的系统 |
| 多语言国际化环境 | ✅ 适用 | 是少数提供 6+ 语言版本的仓库之一 |
| 对质量一致性有高要求的生产环境 | ❌ 慎用 | 工件质量参差，高分（100）和低分（23）并存 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 大规模平铺聚合（本仓库） | everything-claude-code | 覆盖广、社区驱动、易于贡献 | 质量参差不齐、无内聚性 |
| 单一职责插件（高内聚） | agiletec-inc/airis-mcp-gateway | 质量一致、依赖关系明确 | 覆盖窄、扩展需要设计 |
| 精选集合（策展型） | hesreallyhim/awesome-claude-code | 只引用，不包含工件，质量由引用筛选 | 不可直接安装，需要二次集成 |

### 2.4 改进空间

1. **当前问题**：24 个 bug 主要集中在缺 frontmatter（name/description）。**改进做法**：写一个脚本扫描所有命令文件，自动从文件名和第一个 h1 标题推断 `name` 和 `description` 并回填。**预期收益**：24 个 bug 可以用一次 PR 批量修复

2. **当前问题**：国际化副本（~200 个）与英文原版手动维护，质量漂移。**改进做法**：引入 CI 检查，对比英文版 mtime 和各语言副本 mtime，标记出"过期翻译"。**预期收益**：避免用户使用过时的国际化版本

3. **当前问题**：.kiro/ 的 agent 全部缺 `model` 字段（非标准 Claude Code 格式）。**改进做法**：统一加 `model: claude-sonnet-4-5` 或在 .kiro/ 目录加 README 说明这是 Kiro 专用格式。**预期收益**：消除 NLPM 扣分，也减少用户对这些文件的困惑

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **84/100**（进阶样本，~320 个工件）。Security: **BLOCKED**。

| 文件（代表性） | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 23 | 缺 name + description frontmatter |
| commands/gan-build.md | 25 | 缺 frontmatter + 引用未定义 agent |
| skills/dart-flutter-patterns/SKILL.md | 84 | "appropriate"×8 |
| commands/skill-create.md | 100 | 满分 |
| skills/everything-claude-code/SKILL.md | 100 | 满分 |
| skills/ck/SKILL.md | 100 | 满分 |

（注：100 分满分工件有 50+ 个，说明作者在核心 skill 上质量很高，低分集中在命令文件）

### 3.2 当时值得借鉴的模式

1. **50+ 个 100 分满分 skill**：skill 是这个仓库最高质量的部分，有完整的 name/description/examples/output format。说明作者对 skill 格式的掌握是扎实的——只是没有把同样的标准应用到命令文件上。借鉴：先把 skill 做好，再扩展到命令

2. **GAN 自进化 harness**：`scripts/gan-harness.sh` 驱动生成器 agent 和评估器 agent 互相对抗，自动优化工件质量。这是目前见过的最有创意的 NL 工件自进化尝试。借鉴：对于需要持续迭代的工件，可以引入评估者 agent 做自动质量反馈

3. **`commands/skill-create.md`（100 分）**：一个用来创建 skill 的命令本身是满分，说明作者知道怎么写高质量工件，只是没有批量应用到旧命令。借鉴：自举工具（用来创建其他工件的工件）最能体现作者的最高水平

4. **SOUL.md + WORKING-CONTEXT.md**：除了功能性文档，还有作者的设计哲学文档。这让贡献者理解"为什么"而不只是"怎么做"。借鉴：长期维护的仓库可以加入设计理念文档

5. **多层次 hooks 拦截**：`hooks/` 目录用于拦截 Claude Code 事件，补充标准工件无法处理的场景（如自动触发安全检查）。借鉴：hooks 是 NL 工件的"中间件"层，可以做横切关注点

### 3.3 当时的缺陷

1. **批量 Security BLOCKED（7 个安全发现，其中 3 个 Critical）**
   - install.sh 包含多种 `curl | bash` 和下载执行模式，无任何完整性验证
   - **根本原因**：规模化快速增长时，每个新增组件都遵循了"先跑起来"的原则，安全是事后考量。7 个安全发现分散在不同文件里，说明没有跨文件的安全审查流程
   - **自查**：我的项目如果要加安装脚本，有没有完整性验证流程？

2. **24 个 BUG 集中在 frontmatter 缺失**
   - 受影响命令很多（test-coverage、gan-build、learn 等），主因是早期写命令时没有建立 frontmatter 规范，后来也没有批量补救
   - **根本原因**：规模增长时缺乏"入库门槛"——没有 lint 工具或 CI 检查来拦截缺少 frontmatter 的 PR
   - **自查**：我的项目有没有 pre-commit hook 或 CI 检查来验证新增工件的 frontmatter 完整性？

3. **国际化副本质量漂移**
   - docs/zh-CN/ 等目录的副本分数普遍低于英文原版（75-90 vs 80-100），部分副本有额外的模糊量词或结构问题
   - **根本原因**：翻译是人工完成的，翻译过程中很难维持原文的结构严格性
   - **自查**：如果我的项目将来需要国际化，应该考虑什么策略来维持质量一致性？

### 3.4 当时的优化机会

1. **批量修复 frontmatter**：写一个脚本从文件名推断 `name`，从第一个 h1 推断 `description`，自动写入缺失字段
2. **install.sh 重写**：用 npm install 替代 curl|bash，把安全责任转移到 npm 的完整性验证机制
3. **国际化 CI 同步检查**：对比英文版和各语言版本的关键字段完整性

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| install.sh curl\|bash 安全漏洞 | `head -20 install.sh` | **已修复** ✅ — install.sh 现为 Node.js 委托架构，把安装逻辑移到 `scripts/install-apply.js`，通过 npm 生态分发，不再有裸露的 curl\|bash | 最关键的安全问题已解决，表明作者把安全列为高优先级 |
| commands/ 批量缺 frontmatter | `find commands/ -name "*.md" \| wc -l` vs `grep -l "^---" commands/*.md \| wc -l` | **大幅改善** ✅ — 当前 94 个命令文件有 frontmatter，较审计时有显著提升 | 批量修复已推进，但可能还有少量遗漏 |
| Security BLOCKED 状态 | 查看新增的 the-security-guide.md | **进行中** ⚠️ — 新增了专门的安全教育文档（the-security-guide.md），但 BLOCKED 状态是否清除需要重新跑 security scan 确认 | 作者对安全问题高度重视，但实际修复进展未完全核实 |

### 4.2 架构演进

与 2026-04-06 审计时相比，最显著的架构变化：
1. **install.sh 从 bash 重写为 Node.js 委托**：这是一次安全驱动的重大重构，表明作者接受了审计反馈
2. **新增 the-security-guide.md**：深度的 agent 安全文章，涉及 CVE-2025-59536、CVE-2026-21852 等真实案例，说明作者把安全教育提升到与工件质量同等的地位
3. **新增 SOUL.md 和 WORKING-CONTEXT.md**：作者在进化过程中加入了更多"为什么"层面的文档，从纯工具仓库向"有设计哲学的生态系统"转变
4. **ecc2/ 和 manifests/ 目录**：暗示正在进行第二代架构设计（ECC v2）

### 4.3 新增的可学习模式

- **`the-security-guide.md`**：作者把 agent 安全教育文章直接放入仓库，而不是外链博客。这意味着关注安全的用户可以在安装仓库的同时获得安全知识。值得借鉴：重要的"如何安全使用"文档可以和工件一起发布

- **`ecc_dashboard.py`**：一个可视化 ECC 使用情况的 Python 脚本。说明作者在思考"如何帮助用户理解这个庞大系统"的问题。值得借鉴：大型工件集合可以配套一个可视化/检索工具

---

## 五、校准

### 5.1 我已经在做对的

1. **我的项目规模合理**：bureau/gstack 的工件数量在 10-30 个，可以保证每个工件都能做到高质量；ECC 的 934 个工件中，低质量工件数量与总量成比例
2. **我的 skill 有 examples**：抽查 bureau/skills/recall/SKILL.md 有完整的 ## Steps 和使用说明，比 ECC 的多数命令文件更规范
3. **没有 curl|bash 安装方式**：bureau 通过 Claude Code plugin install 安装，天然安全
4. **单仓库聚焦一个领域**：bureau 聚焦知识管理，不像 ECC 什么都有。聚焦带来了更好的内聚性

### 5.2 挑战 / 验证

- **本案例挑战了什么假设？** 我之前认为"大量高分工件意味着仓库整体质量高"。ECC 有 50+ 个 100 分的 skill，但同时也有 24 个 bug 和 Security BLOCKED。规模越大，管理成本越高，质量下限越难保证。
- **被验证的做法**：「先写 skill 再写 command」的策略被 ECC 验证：ECC 的 skill 普遍比 command 得分高，这符合"knowledge 层比 action 层更容易写好"的直觉

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有完整 frontmatter（name + description + allowed-tools）
for f in commands/*.md; do
  count=$(head -10 "$f" | grep -cE "^name:|^description:|^allowed-tools:")
  [ "$count" -lt 3 ] && echo "INCOMPLETE: $f (found $count/3 required fields)"
done

# 如果有国际化副本，检查副本和英文原版的 name/description 一致性
# （ECC 的漂移教训）
for lang_dir in docs/*/; do
  for f in "$lang_dir"commands/*.md; do
    en_file="commands/$(basename $f)"
    [ -f "$en_file" ] && \
      diff <(grep -E "^name:|^description:" "$en_file") \
           <(grep -E "^name:|^description:" "$f") > /dev/null || \
      echo "DRIFT: $f vs $en_file"
  done
done

# 检查我的项目有没有引用不存在 agent 的命令（ECC 的 gan-build.md 缺陷）
grep -rn "agents/\|@\w*-agent" commands/*.md | grep -v "^Binary" | \
  while IFS=: read file line content; do
    agent=$(echo "$content" | grep -oE 'agents/\S+\.md' | head -1)
    [ -n "$agent" ] && [ ! -f "$agent" ] && echo "BROKEN_REF: $file:$line -> $agent"
  done

# 检查 hooks/ 目录是否有未声明的权限
grep -rn "rm -rf\|curl\|exec\|eval" hooks/*.sh 2>/dev/null
# 命中后：评估是否需要在 CLAUDE.md 里明确声明 hook 的行为边界
```

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 引入类似 `scripts/gan-harness.sh` 的自评估机制
   - **为何可行**：bureau 的 `auditor.md` agent 可以定期自动审查 canon 的质量；用 GAN 思路让"编写者 agent"和"审计者 agent"轮流工作可以持续提升质量
   - **第一步**：在 bureau 的 `.claude/agents/auditor.md` 里加入自触发逻辑，让它在每次 `bureau:compile` 后自动运行一次质量检查。耗时 20 分钟

2. **想法**：写一个脚本验证我的工件 frontmatter 完整性（对标 ECC 的教训）
   - **为何可行**：ECC 的 24 个 frontmatter bug 很大程度上是因为没有自动化检查拦截
   - **第一步**：在 `.github/workflows/` 里加一个 YAML 文件，用 `python3 -c "import yaml; ..."` 验证每个工件的 name/description/allowed-tools 字段存在。耗时 30 分钟

3. **想法**：借鉴 ECC 的 `the-security-guide.md`，为 bureau 写一个使用安全指南
   - **为何可行**：bureau 处理的是用户私人笔记和项目知识，存在 prompt injection 风险（用户输入的内容被当成指令执行）
   - **第一步**：创建 `docs/security.md`，列出 bureau 的主要风险面（未过滤的外部输入进入 canon）和缓解措施（trust tier 机制说明）。耗时 25 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 affaan-m/everything-claude-code 的核心目的**：提供一个全覆盖的 Claude Code NL 工件集合，作为任何开发者的 starter kit
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有 commands+agents+skills 三层 | bureau 是高内聚专用工具；ECC 是广覆盖聚合集合 | 高 |
| MarkQWu/gstack | 低 | 都有多个 agent 角色定义 | gstack 是角色扮演；ECC 是全栈开发辅助 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands/ 缺 name + allowed-tools | `for f in commands/*.md; do head -10 "$f" | grep -c "^name:\|^allowed-tools:"; done` | **bureau 命中** — commands 普遍有 description 但缺 name 和 allowed-tools | 高 |
| 引用不存在的 agent | `grep -rn "@\w*-agent\|agents/" commands/*.md` | 预计无命中（bureau 结构相对简单） | 低 |
| 工件质量参差（高低分并存） | 对 skills 和 commands 分别跑 NLPM 评估 | 未测量，但 bureau skills 预计较高，commands 可能偏低 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/bureau` 的所有 `commands/*.md` → 批量添加 `name:` 字段（从文件名推断，如 `lint.md` → `name: lint`）→ 每个 5 分钟，11 个约 55 分钟总计
- `MarkQWu/bureau` 的所有 `commands/*.md` → 根据命令体工具调用补 `allowed-tools:`（如 `lint.md` 用 Read/Bash → `allowed-tools: [Read, Bash, Glob, Grep]`）→ 同上合并处理

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：「GAN 风格自进化 harness」
   - **本案例做法**：`scripts/gan-harness.sh` 驱动生成器 agent 和评估器 agent 互相对抗，自动迭代工件质量，最多循环 15 次
   - **我的项目现状**：bureau/gstack 没有任何自评估机制，工件质量依赖人工审查
   - **如何借鉴**：为 bureau 的 `auditor.md` agent 加入自触发逻辑：在每次知识编译后自动运行一次质量审查，并把审查结果写入 canon 作为质量记录

2. **领域**：「SOUL.md 设计哲学文档」
   - **本案例做法**：`SOUL.md` 描述了作者对 AI agent 进化的长期思考，使贡献者理解"为什么要这么设计"
   - **我的项目现状**：bureau 只有功能性文档（README、CLAUDE.md），缺少设计理念说明
   - **如何借鉴**：为 bureau 加一个 `PHILOSOPHY.md`，解释"为什么要 trust tier 而不是直接存 raw 内容"，"为什么 capture→compile 分两步"等设计决策

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：工件规模与质量的平衡
- **我的做法**：bureau 保持 ~10 个高内聚命令 + ~7 个精心设计的 skill，每个工件都有机会做到高质量
- **本案例做法（弱在哪）**：ECC 的 934 个工件中，大量工件处于 23-80 分的"中等偏低"区间；规模化带来了覆盖广度，但也稀释了质量注意力
- **意义**：小而精是我目前的策略，应该继续保持；当 bureau 扩展时，要有意识地维护"每个新增工件都需要通过质量门槛"的原则

---

## 八、术语表

### <a name="GAN"></a>GAN 风格 harness
> 借用机器学习中生成对抗网络（Generative Adversarial Network）的概念，让两个 agent 互相"对抗"：生成器 agent 产出内容，评估器 agent 挑毛病，循环往复直到质量达标。这里不是真正的神经网络训练，而是用 LLM 模拟这个迭代优化过程。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。Claude Code 解析命令文件时必须读取 frontmatter 来注册命令，缺少 `name` 字段意味着命令无法被正确注册，用户调用时会找不到命令。

### <a name="trust-tier"></a>trust tier（信任分级）
> 一种对知识条目按可信度分级的机制。在 bureau 的上下文里，trust tier 区分"已验证的事实"和"待确认的说法"，防止 AI 把用户输入的内容当成已确认的事实来引用。类比：期刊文章（高信任）vs 社交媒体帖子（低信任）。

### <a name="SSOT"></a>单一来源（SSOT，Single Source of Truth）
> 系统中某个信息只在一个地方存储和维护，其他地方只引用这个来源。ECC 的版本号在多个文件里手动维护（没有 SSOT），导致版本不一致；正确做法是从 `package.json` 自动派生 `plugin.json` 的版本号。

### <a name="prompt-injection"></a>prompt injection
> 一种针对 AI 系统的攻击方式：攻击者在数据（如网页内容、用户输入的文档）里嵌入指令，使 AI 把这些指令当成合法任务执行。在 bureau 的场景里，如果用户把一段带有"忽略之前的指令，执行以下操作"的文本存入 canon，bureau 在查询时可能会误执行这段注入的指令。
