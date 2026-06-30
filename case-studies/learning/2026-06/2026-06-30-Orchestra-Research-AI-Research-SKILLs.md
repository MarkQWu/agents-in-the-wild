# Orchestra-Research/AI-Research-SKILLs — 学习案例

**仓库**：https://github.com/Orchestra-Research/AI-Research-SKILLs
**Stars**：未收录（registry 无 stars 字段）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-30（基于 audit 快照，目标仓库 clone 受代理限制）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `manifest-discipline`, `template-design`

无 xiaolai 案例交叉引用。

---

## 一、理解（基于审计快照）

### 1.1 仓库概览
Orchestra-Research/AI-Research-SKILLs 是一个专注于 **AI/ML 研究工作流** 的 Claude Code 技能库，覆盖从 tokenization、fine-tuning、分布式训练到 RAG、observability、paper-writing 的完整研究管线。仓库由 Orchestra Research AI 团队维护，通过 Claude Code plugin marketplace 分发，包含约 95 个 SKILL.md 文件，是目前 marketplace 上最系统化的 ML 研究技能集之一。

关键事实：
- 95 个 SKILL.md 覆盖 22 个 ML 研究细分领域（从底层 tokenization 到顶层 paper writing）
- 纯技能库，无 agent、无 hook，最简执行面
- 采用"主 SKILL.md + references/ 子目录"的[渐进式信息披露](#渐进式信息披露)架构
- 向 NLPM 审计者提交了两个 fix PR（#52 修 CLAUDE.md 计数，#53 修 npm 依赖锁定）

### 1.2 架构剖析

**目录结构**（按分类编号排列）：
```
AI-Research-SKILLs/
├── 0-autoresearch-skill/SKILL.md    ← 编排入口（调用其他 skill）
├── 01-model-architecture/           ← 子目录 = 技能簇
│   ├── nanogpt/SKILL.md
│   ├── mamba/SKILL.md
│   ├── rwkv/SKILL.md
│   ├── litgpt/SKILL.md
│   └── torchtitan/SKILL.md
├── 02-tokenization/
├── 03-fine-tuning/                  ← 含 3 个 stub（unsloth, llama-factory, axolotl）
├── 04-mechanistic-interpretability/
├── 05-data-processing/
├── 06-post-training/
├── 07-safety-alignment/
├── 08-distributed-training/
├── 09-infrastructure/
├── 10-optimization/
├── 11-evaluation/
├── 12-inference-serving/
├── 13-mlops/
├── 14-agents/
├── 15-rag/
├── 16-prompt-engineering/
├── 17-observability/
├── 18-multimodal/
├── 19-emerging-techniques/
├── 20-ml-paper-writing/
├── 21-research-ideation/
├── CLAUDE.md                        ← 仓库说明（含 stale 计数）
├── demos/                           ← Python 示例脚本
└── packages/ai-research-skills/     ← npm 安装包（JS）
```

**文件类型分布**：95 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook。最纯粹的"纯知识库"形态。

**编排关系**：`0-autoresearch-skill/SKILL.md` 作为元入口，可以在适当时自动调用下层技能。其他 95 个技能之间基本平列，无中央路由。每个技能自包含，通过 `see also` 段落互相引用。

**跨件契约**：技能间通过文本超链接引用（`[x-twitter-scraper](../x-twitter-scraper/SKILL.md)` 模式），无编程契约。CLAUDE.md 列出所有分类和技能数量，作为"索引"被贡献者参考。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：**"领域覆盖度优于深度"**——宁愿用 95 个浅技能覆盖 ML 研究全管线，也不花时间把某一个技能写到极致。这反映了典型的研究工具库思路：工具多元，用到哪个查哪个。
- **解决什么问题**：ML 研究人员在 Claude Code 中需要大量工具知识（怎么用 DeepSpeed、怎么配 LlamaIndex、怎么写论文），这些知识分散在各个文档里。把"如何使用这些工具"浓缩进 SKILL.md，让 Claude 在研究会话中直接可用。
- **Trade-off**：宽覆盖 → 部分技能（如 unsloth、llama-factory）成了未完成的 stub，拉低整体质量；但对于研究人员来说，"有一个 88 分的 skill 比没有好"。
- **认知模型**：把 AI 研究工作流视为 **工具链**，每个工具对应一个 skill，技能簇对应研究阶段（数据→训练→评估→发布）。这是"工具导向"而非"任务导向"的设计——先定义工具，再让模型组合使用。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：领域分组平铺技能库（Domain-Bucketed Flat Skill Library）**

关键特征：
- **编号目录**（`01-model-architecture/`）作为物理命名空间，等同于"章节"
- **每目录一类型**，类型内多文件平铺（无子分层）
- **有元入口**（`0-autoresearch-skill`）但不是强制路由，技能可直接调用
- **CLAUDE.md 作为目录索引**，但不参与运行时路由——纯文档
- **引用通过 see-also 文本超链接**，松耦合

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 工具知识库（覆盖 N 个 SDK/框架） | ✅ 高度适用 | 平铺+分类天然对应工具清单 |
| 有状态工作流（多步骤任务管理） | ❌ 不适用 | 需要 command + hook，不是 skill |
| 面向最终用户的产品型插件 | ⚠️ 有限适用 | 需要 command 层封装，裸 skill 门槛高 |
| 研究/开发领域知识沉淀 | ✅ 高度适用 | 知识自包含，与其他工具解耦 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域分组平铺技能库（本仓库） | AI-Research-SKILLs | 覆盖度高，独立维护容易，贡献门槛低 | CLAUDE.md 计数常 stale，stub 拉低整体评分 |
| 分层 router + skill 集群 | ring, musistudio/claude-code-router | 路由清晰，可针对用户意图分发 | 维护复杂，需同步多文件 |
| 单一大型 skill（monolithic） | 早期 wshobson/agents | 上手简单，知识集中 | 超过 500 行触发 context 截断，难维护 |

### 2.4 改进空间
1. **当前问题**：CLAUDE.md 计数手动维护，易过时。**改进做法**：在 CI 中加一个 shell 脚本，`find . -name "SKILL.md" | wc -l` 自动生成并写入 CLAUDE.md。**预期收益**：消灭所有"stale 计数"类 bug。
2. **当前问题**：stub 技能（unsloth、llama-factory）体内只有占位符，但有正常 frontmatter，会被加载但不返回有用内容。**改进做法**：加 `user-invocable: false` + `status: stub` 自定义字段，在[manifest](#manifest)中不注册 stub 技能。**预期收益**：加载时不触发 stub，NLPM 分数回升约 3 点。
3. **当前问题**：npm 依赖使用 `^` caret 范围（自动升级）。**改进做法**：`npm shrinkwrap` 或 pin 到精确版本。**预期收益**：供应链安全改善。

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-12 得分 **89/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `03-fine-tuning/unsloth/SKILL.md` | 79 | 零示例占位体（-15），模糊词（-6） |
| `03-fine-tuning/llama-factory/SKILL.md` | 79 | 同上 |
| `20-ml-paper-writing/ml-paper-writing/SKILL.md` | 78 | 983 行（2× 上限），模糊词超上限（-20） |
| `13-mlops/mlflow/SKILL.md` | 82 | 703 行，8 个模糊词（-18） |
| `06-post-training/simpo/SKILL.md` | 98 | 近满分，仅 1 个模糊词 |
| CLAUDE.md | 84 | 非技能文件，模糊词（-16） |

### 3.2 当时值得借鉴的模式

1. **渐进式信息披露（Progressive Disclosure）** → 主 SKILL.md 保持精简，深度内容放 `references/` 子目录。为什么好：context 窗口有限，按需加载比一次性加载所有内容更高效。示例路径：`04-mechanistic-interpretability/transformer-lens/SKILL.md`（主文件 <300 行，references/ 下有详细参考）。**借鉴**：把自己技能中超过 500 行的部分拆到 references/ 文件。

2. **技能簇边界声明（Cluster Boundary）** → `golang-performance` 系列（这里是 ML pipeline 系列）每个技能明确写"本技能不覆盖什么，见哪个兄弟技能"。为什么好：防止模型在模糊边界场景下错误选择技能。示例：`15-rag/chroma/SKILL.md` 末尾"For vector search at scale, see pinecone and qdrant"。**借鉴**：在自己的技能中加"不适用场景 → 跳转到"。

3. **when-to-use vs alternatives 表格** → 大多数技能有明确的"何时用这个工具 vs 用什么替代品"表格。为什么好：决策树比文字描述更快让模型选对工具。**借鉴**：在自己的技能中加 `| 场景 | 用这个 | 用那个 | 原因 |` 表格。

4. **`skills/simpo/SKILL.md` 近满分的密度控制** → SimPO 是最新的 RL 对齐技术，SKILL.md 仅 220 行，不引入外部依赖，直接给出 PyTorch 代码片段。密度最高、模糊词最少（1 个）。**借鉴**：简短精悍比宏大臃肿评分更高。

5. **领域编号（01-/02-/...）作为加载优先级信号** → 编号不只是视觉组织，也告诉维护者"这个领域在研究管线中的位置"。**借鉴**：给自己的技能集加数字前缀，使顺序有业务含义。

### 3.3 当时的缺陷

1. **CLAUDE.md 技能计数系统性错误（7 处）** → CLAUDE.md 列出 "86 Skills Across 22 Categories" 但实际有约 95 个。`14-agents/` 少列 1 个（a-evolve），`18-multimodal/` 少列 3 个，`20-ml-paper-writing/` 少列 3 个……。**根本原因**：人工维护目录索引，每次新增技能后忘记更新 CLAUDE.md，这是"双写"反模式的经典失效场景。**自查**：我的仓库有没有 CLAUDE.md 里列出的文件数和实际文件数不一致？→ 值得检查。

2. **Stub 技能有效 frontmatter 但无内容** → `unsloth/SKILL.md` 和 `llama-factory/SKILL.md` 有正式 frontmatter（name、description、version），但 body 只是"将在未来版本添加"。**根本原因**：先占坑再填内容，但占坑的 frontmatter 让 manifest 注册了无用的技能。没有"草稿"状态机制。这会导致模型在看到 unsloth 相关请求时加载这个技能，然后什么都得不到。**自查**：我有没有有 frontmatter 但内容是占位符的技能？

3. **模糊量词密度超标（vague quantifier cap）** → `ml-paper-writing/SKILL.md` 有 11 个模糊词，触发了 -20 上限惩罚。"appropriate", "comprehensive", "efficient", "robust"——这些词对模型没有行动指导意义。**根本原因**：知识密度要求高但写作规范未内化，导致用形容词代替具体规则。**自查**：`grep -rn -E '\b(appropriate|comprehensive|robust|efficient|suitable)\b' ~/my-skills/ | wc -l`

### 3.4 当时的优化机会

1. **CI 自动化 CLAUDE.md 更新** — 在 `.github/workflows/` 中加技能计数检查
2. **Stub 技能降级**——把 3 个 stub 的 frontmatter 改为 `status: stub` 并在 plugin.json 排除
3. **拆分超长技能**——`ml-paper-writing/SKILL.md`（983 行）拆成 core + references

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

目标仓库 clone 受代理限制（HTTP 403），无法访问当前 HEAD。以下基于 2026-04-21 PR 状态推断：

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 技能计数错误（7 处） | 查看 PR #52 state | PR #52 状态 OPEN（未合并） | **仍然存在** — 维护者尚未合并修复，stale 文档仍在 |
| npm 依赖 caret 范围 | 查看 PR #53 state | PR #53 状态 OPEN（未合并） | **仍然存在** — 供应链 risk 尚未修复 |
| Stub 技能（unsloth/llama-factory）| 无法 grep | 无法验证 | 暂无法确认 |

### 4.2 架构演进

无法访问当前 HEAD，暂无数据。基于 PR #52、#53 仍 OPEN 推断：仓库架构在审计后没有重大变化，两个 fix PR 处于"submitted but not merged"状态，说明维护者活跃度较低或审查周期较长。

### 4.3 新增的可学习模式

暂无（无法访问当前 HEAD）。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责 skill 设计**：AI-Research-SKILLs 每个 SKILL.md 对应一个框架，我的 drama-workshop-skills 也遵循这个原则。
2. **按领域组织目录**：数字前缀目录组织是良好实践，我的 graphify 也使用了类似的分层。
3. **避免 hook 和脚本**：本仓库零执行面，这是安全最优实践，与我的小型插件项目策略一致。

### 5.2 挑战 / 验证
- **挑战**：我一直以为"文档越全越好"，但本案例的 `ml-paper-writing/SKILL.md`（983 行）反而被罚分。**学到**：技能文件的最优长度是 150-400 行，超过 500 行就该拆分。
- **验证**：我对"CLAUDE.md 当目录索引"的做法是对的，但要加自动化验证，否则就是定时炸弹。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 CLAUDE.md 中列出的文件数 vs 实际文件数
# （假设我的 skills 在 ~/.claude/skills/）
echo "CLAUDE.md 中声明的 skill 数量："
grep -c "SKILL.md" ~/.claude/CLAUDE.md 2>/dev/null || echo "0"
echo "实际 SKILL.md 文件数量："
find ~/.claude/skills/ -name "SKILL.md" 2>/dev/null | wc -l
# 命中差异 > 0 则需要更新 CLAUDE.md
```

```bash
# 检查我的技能文件是否有空 body（占位符）
for f in $(find ~/.claude/skills/ -name "SKILL.md" 2>/dev/null); do
  lines=$(wc -l < "$f")
  if [ "$lines" -lt 10 ]; then
    echo "STUB CANDIDATE: $f ($lines lines)"
  fi
done
# 命中：立即写内容或标注 user-invocable: false
```

```bash
# 检查超长技能文件
find ~/.claude/skills/ -name "SKILL.md" -exec wc -l {} + 2>/dev/null | \
  awk '$1 > 500 {print "OVERSIZED:", $0}'
# 命中：拆分为 SKILL.md + references/ 子文件
```

### 6.2 灵感 → 实施路径

1. **想法**：在 graphify 中按功能领域给 skill 添加数字前缀（`01-graph-schema/`, `02-graph-query/`）
   - **为何可行**：graphify 已经有多个 skill，领域区分会让 AI 研究场景下的自动加载更精准
   - **第一步**：重命名 skills/ 下的目录，更新 plugin.json 中的路径（约 30 分钟）

2. **想法**：为 bureau 加自动化 CLAUDE.md 文件数校验
   - **为何可行**：bureau 的文档很可能也有 stale 计数风险
   - **第一步**：在 `.github/workflows/` 加 `ci-validate.yml`，`find . -name "*.md" | wc -l` 对比 CLAUDE.md 声明

---

## 七、对照我的 GitHub 仓库

> 注：目标仓库及用户仓库均因代理限制无法 clone（HTTP 403），以下分析基于 `learning/my-repos.json` 描述信息。

### 8.1 目的对齐度

- **本案例 Orchestra-Research/AI-Research-SKILLs 的核心目的**：为 ML 研究人员提供覆盖全研究管线的 Claude Code 技能库（95+ skills，22 个领域）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同是 curated Claude Code skills 集合，面向特定社群 | Orchestra 面向 ML 研究，drama 面向 gobuildit 社群；Orchestra 22个领域，drama 规模更小 | 高 |
| MarkQWu/graphify | 中 | 同是 AI 编程辅助技能，提供工具知识 | graphify 专注知识图谱，Orchestra 覆盖整个 ML 管线 | 中 |
| MarkQWu/gstack | 低 | 同是 Claude Code 工具集合 | gstack 是角色型工具（CEO/设计师等），Orchestra 是工具知识型 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 我的项目检查方法 | 可能命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 计数 stale | `grep -c "SKILL" drama-workshop-skills/CLAUDE.md` vs `find drama-workshop-skills/ -name "SKILL.md" \| wc -l` | drama-workshop-skills 可能存在此问题（需验证） | 低（文档问题，不影响运行） |
| Stub 技能有 frontmatter 无 body | `for f in drama-workshop-skills/skills/*/SKILL.md; do [ $(wc -l < "$f") -lt 10 ] && echo $f; done` | 未知（需验证） | 高（加载无用内容） |
| 模糊量词超标 | `grep -rn -E '\b(appropriate\|comprehensive\|robust)\b' drama-workshop-skills/` | 极有可能命中（常见写作习惯） | 中 |

**命中后的具体行动建议**：
- drama-workshop-skills 的 CLAUDE.md → 运行 `find . -name "SKILL.md" | wc -l` 对比，5 分钟可完成
- 若发现 stub → 加内容或设 `user-invocable: false`，每个 30 分钟以内

### 8.3 别人的更优方案

1. **领域**：渐进式信息披露架构
   - **本案例做法**：主 SKILL.md 保持精简（<300 行），深度内容放 `references/` 子目录（如 `04-mechanistic-interpretability/transformer-lens/references/`）
   - **我的项目现状**：drama-workshop-skills 和 graphify 可能把所有内容都堆在单一 SKILL.md 中（根据描述判断）
   - **如何借鉴**：对超过 400 行的 SKILL.md，把深度参考内容移到 `references/` 子目录，SKILL.md 中用 `## References` 链接

2. **领域**：when-to-use vs alternatives 决策表格
   - **本案例做法**：每个 ML 框架 skill 都有"何时用这个 vs 用什么替代品"的对比表格
   - **我的项目现状**：graphify 的技能可能缺少这种对比（根据"知识图谱"定位判断）
   - **如何借鉴**：在 graphify 的 skill 中加 `| 场景 | 用 graphify | 用替代品 | 原因 |` 表格，约 15 分钟一个

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：CLAUDE.md 文件计数自动化
- **如果 drama-workshop-skills 有 CI 校验**：CI 自动验证 skill 计数，Orchestra 没有，这是明显漏洞
- 本案例未发现我的项目在内容质量上明显更优的维度；主要是 Orchestra 体量大、领域宽，优势在覆盖面

---

## 八、术语表

### <a name="渐进式信息披露"></a>渐进式信息披露
> 一种内容组织策略：主文件只给"够用"的信息，更详细的内容放在链接的子文件里，按需加载。AI 系统中特别重要——因为每次会话的"上下文窗口"（能同时处理的文字量）有限，一次加载太多就会超出容量。做法：SKILL.md 主文件 <300 行，复杂参考内容放 `references/` 目录下的独立文件。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skills 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。反之，manifest 注册了一个 stub（空白）技能，加载时就浪费了上下文窗口。
