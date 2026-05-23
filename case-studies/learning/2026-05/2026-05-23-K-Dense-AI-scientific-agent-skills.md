# K-Dense-AI/scientific-agent-skills — 学习案例

**仓库**：https://github.com/K-Dense-AI/scientific-agent-skills
**Stars**：19,356 | **来源**：upstream audit
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-05-23（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `manifest-discipline`, `security-gate`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
K-Dense-AI/scientific-agent-skills 是 K-Dense Inc. 的旗舰产品：一个面向**科研工作者**的 Claude Code skill 集合（当前 139 个 SKILL.md），覆盖生物信息学（scanpy、anndata、biopython）、化学（pyopenms、cobrapy）、物理（fluidsim、cirq）、临床（treatment-plans、clinical-reports）、科研写作（scientific-writing、peer-review、literature-review）等领域。19,356 stars 说明它打中了科研社区"让 AI 真正懂我的专业工具"的需求。K-Dense Inc. 是一家 AI-first 的科研工具公司，本仓库是其商业产品生态的一部分——包含专有的"Nano Banana"图像生成产品的调用入口。

### 1.2 架构剖析
- **目录结构**：
  ```
  scientific-agent-skills/
  ├── scientific-skills/       # 139 个 skill 子目录
  │   ├── scanpy/              # 每个 skill 含 SKILL.md + scripts/ + 可选 references/
  │   │   ├── SKILL.md
  │   │   └── scripts/
  │   │       ├── generate_schematic.py
  │   │       └── generate_schematic_ai.py
  │   ├── autoskill/           # 新增：从屏幕录制自动生成 skill
  │   └── ... (137 others)
  ├── docs/                    # 文档
  ├── scan_skills.py           # skill 质量扫描工具
  ├── scan_pr_skills.py        # PR 检查工具
  ├── pyproject.toml           # Python 项目配置
  └── SECURITY.md              # 安全报告（审计工具生成）
  ```
- **文件类型分布**：139 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook / 233+ 个 Python scripts
- **编排关系**：完全平铺，139 个 skill 互相独立，没有 router 或编排层。共有模式是每个 skill 附带 `generate_schematic.py`（调用 Nano Banana API 生成图像）和 `generate_schematic_ai.py`（实际的 OpenRouter API 调用）。
- **跨件契约**：[frontmatter](#frontmatter) 约定有 `name`、`description`、`license`、`metadata.skill-author`。`license` 字段有大量不一致（Unknown、URL 格式、Proprietary 等）。多个 skill 由第三方贡献（`metadata.skill-author` 显示不同作者），说明仓库接受外部贡献。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：领域专家知识封装为标准化 skill。每个 skill 是一个科学计算工具的"操作手册"，让 Claude 不用从头学习每个工具的 API 和最佳实践。
- **解决什么问题**：科研软件（bioinformatics 工具、化学分析库）的 API 复杂、文档分散、初学者门槛极高。把这些知识封装成 skill 后，用户可以用自然语言驱动专业工具。
- **做了什么 trade-off**：把"生成科学示意图"（通过 Nano Banana Pro API）集成到多个 skill 的标准工作流里，换来了更丰富的输出，但代价是引入了不透明的付费依赖，且很多 skill 在没有 `OPENROUTER_API_KEY` 的情况下会静默失败。
- **反映什么认知模型**：作者认为科研 AI agent 的价值在于"专业工具的知识前沿"——不只是通用 AI 能力，而是把每个科学领域最新的最佳实践作为 skill 的知识库。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「领域百科 skill 平铺集合」**

一个仓库包含某个垂直领域（科研）里所有工具的独立 skill，每个 skill 是一个工具的操作手册，用户按需单独加载。仓库本身是一个"商店货架"而不是"编排系统"。

模式特征清单：
- **领域专用**：所有 skill 服务同一垂直领域（科研），形成内聚的知识库
- **工具 1:1**：每个 skill 对应一个科学计算工具或任务类型，边界清晰
- **外部贡献机制**：`metadata.skill-author` 记录贡献者，支持社区 PR，形成知识众包
- **脚本伴随**：每个 skill 目录含配套 Python scripts，AI 调用脚本而非自己计算
- **商业集成**：通过 `generate_schematic.py` 集成专有产品，skill 是产品生态入口

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要深度垂直领域知识的专业工具集合 | ✅ 高度适用 | 正是本仓库的定位 |
| 社区众包知识积累 | ✅ 高度适用 | metadata.skill-author 机制天然支持 |
| 离线/无 API key 环境 | ❌ 不适用 | 多个 skill 依赖付费 API（OPENROUTER_API_KEY） |
| 需要 skill 之间编排、有复杂依赖流 | ❌ 不适用 | 平铺架构无编排能力 |
| 追求高 NL 质量评分的项目 | ⚠️ 部分适用 | 139 个 skill 难以统一维护质量，实际平均 83 分 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域百科平铺（本仓库） | scientific-agent-skills | 覆盖广，社区贡献友好，每个 skill 独立可用 | 质量参差不齐，维护成本 O(n)，无法 skill 编排 |
| 分层 command+skill | cavekit | 质量均匀，易维护 | 功能数量受限 |
| 垂直工作流 | baoyu-skills | 深度，脚本集成 | 宽度受限 |

### 2.4 改进空间
1. **当前问题**：14+ 个 skill 的工作流步骤里硬编码"Nano Banana Pro"品牌调用，但 `description` 字段未说明需要付费 API key。**改进做法**：在所有依赖 `OPENROUTER_API_KEY` 的 skill 的 `description` 里加 "(requires OPENROUTER_API_KEY)"，并参考 baoyu-skills 的 `metadata.openclaw.requires` 在 frontmatter 声明 API key 依赖。**预期收益**：用户安装时就知道前置条件，不在运行时静默失败。
2. **当前问题**：`uv uv pip install` 的双前缀 bug 在多个 skill 里存在（当前已修 scikit-learn 等，但 gget 仍存在）。**改进做法**：在 `scan_skills.py` 里加一条正则检测规则 `r"uv uv pip"` 作为必过测试，CI 强制执行。**预期收益**：新贡献者提交 PR 时自动被拦截，防止同类 bug 反复出现。
3. **当前问题**：`license: Unknown` 从审计时的 9 个增加到现在的 15 个。**改进做法**：在 `scan_pr_skills.py` 的 PR 检查里加 `license: Unknown` 为阻断项（而不只是警告），强制贡献者查明 license 再合并。**预期收益**：license 问题不再积累，已有的 Unknown 逐步清理。

---

## 三、过去审查发现（2026-04-25 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-25 当时得分 **83/100**（加权平均，100 个 skill）。

| 层级 | 文件数 | 分数区间 | 代表文件 |
|---|---|---|---|
| Clean | ~48 | 86–88 | zarr-python, anndata, pymc, networkx |
| Minor issues | ~27 | 82–85 | scikit-learn (bug), pyopenms (bug), tiledbvcf |
| Quality issues | ~17 | 73–80 | venue-templates, scientific-writing, peer-review |
| Significant issues | ~8 | 70–72 | treatment-plans, clinical-reports, latex-posters |

### 3.2 当时值得借鉴的模式
1. **领域专家知识封装为独立 skill** → 每个生物信息学工具（scanpy、anndata、seurat）有自己的 SKILL.md，不做大而全的"all-in-one bioinformatics" skill。分而治之，每个 skill 聚焦一个工具的最佳实践。→ 借鉴：自己的 skill 集合里，按工具而不是按任务类型切分 skill 边界
2. **`metadata.skill-author` 第三方贡献归因** → 每个 skill 记录实际的贡献者（可以是公司外部人员），保证归因清晰，维护责任明确。→ 借鉴：在接受外部贡献的仓库里，强制要求 `metadata.skill-author` 字段
3. **Python scripts 和 SKILL.md 同目录分发** → 每个 skill 的 scripts 放在 skill 自己的目录里，不共享也不依赖全局 scripts/，保持自包含。→ 借鉴：skill 的配套脚本放在 skill 目录内，不要放在仓库根目录
4. **`scan_skills.py` 内建质量扫描工具** → 仓库自带一个 Python 质量检查脚本，可以在提交前主动发现问题，不依赖外部审计工具。
5. **`autoskill` 模式感知自生成 skill** → 新增的 autoskill 观察用户屏幕（通过 screenpipe），发现重复的研究工作流，自动生成对应的 skill 草稿——是目前见过的最激进的 [经验侧车文件](#经验侧车文件) 思路的演进。

### 3.3 当时的缺陷
1. **问题：`uv uv pip install` 双前缀 bug（5 个 skill）** → 为什么失败：批量生成 skill 时用了错误的模板，模板里的 install 命令是 `uv uv pip install`（比正确多一个 `uv`），AI 生成了 100 个 skill 但没有对 install 命令做验证测试。根本原因是**缺少 install 命令的机械验证步骤**——这种 bug 用 regex 一行就能检测，但没有集成到 CI。→ 自查：我的 skill 里的安装命令有没有经过实际执行验证？
2. **问题：14 个 skill 嵌入 "Nano Banana Pro" 品牌调用而不披露 API key 需求** → 为什么失败：作者把自家产品的调用直接写进 skill 工作流步骤，但没有在 description 或 frontmatter 里说明前置条件。用户下载并运行时才发现需要付费 API key，体验差且降低社区信任。根本原因是**商业利益和用户透明度之间没有平衡**。→ 自查：我的 skill 里有没有隐含的付费依赖或账号前置条件？
3. **问题：`license: Unknown` 积累** → 为什么失败：贡献者提 PR 时填了 `Unknown`，CI 没有拦截，主维护者 merge 时也没有追问。随时间积累，"Unknown" license 变成了仓库的技术债务。→ 自查：我的 skill 里有没有 `license` 字段缺失或值不规范的情况？

### 3.4 当时的优化机会
1. 在 `scan_pr_skills.py` 里加 `uv uv` 和 `license: Unknown` 的阻断检测规则
2. 为依赖 `OPENROUTER_API_KEY` 的 skill 统一在 description 和 frontmatter 里声明 API key 需求
3. 对 9 个 `license: Unknown` 的 skill 逐一查明对应库的 SPDX license 标识符（大多数是知名 OSS 库，5 分钟可查明）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `uv uv pip install` 双前缀 bug（5 个 skill） | `grep -rn "uv uv pip" scientific-skills/` | **部分修复**：scikit-learn、pyopenms、fluidsim、geniml 已修，但 `gget/SKILL.md` 第 23 行仍有 `uv uv pip install gget` | SECURITY.md 里记录了此 bug，说明被外部安全扫描发现，但仍有遗漏 |
| 14 个 skill 嵌入 Nano Banana Pro 品牌（未披露 API key） | `grep -rl "Nano Banana" scientific-skills/*.md` | **大幅减少但仍存在**：从 14 个减少到约 5 个（markitdown、generate-image、peer-review 等仍有）；大多数已删除品牌引用 | 作者在审计后有清理，但清理不彻底 |
| `license: Unknown` 积累 | `grep -rn "license: Unknown" scientific-skills/` | **恶化**：从 9 个增加到 15 个（+6 个）| 新增的 skill 没有遵守 license 规范，说明 PR 检查没有强制此字段 |

### 4.2 架构演进
skill 数量从 100 增长到 139（+39 个），说明过去 1 个月里持续有新 skill 合并。最重要的新增是：
- **`autoskill`**：从用户屏幕录制（screenpipe）自动检测重复工作流，生成 skill 草稿——这是一个颠覆性的自进化机制，把 skill 的生成从人工知识录入变为自动观察+生成
- **`clinical-decision-support`**：临床决策支持，说明在保持科研工具广度的同时，开始往临床应用深化
- **`uv.lock` 文件**：Python 依赖锁定，配合 `pyproject.toml` 说明 Python 工具层在走成熟化路线

`license: Unknown` 增加是一个警示：新增 39 个 skill 里有部分贡献者没有认真填写 license 字段，且 PR review 没有拦截。这和审计时的质量参差不齐问题是同一根源。

### 4.3 新增的可学习模式
**`autoskill` 的屏幕录制驱动的 skill 生成**：通过 screenpipe 监听用户的实际操作轨迹，聚类识别重复的研究工作流，与现有 skill 库对比后提出新 skill 草稿。关键安全设计：所有检测在本地运行，只有"去标识化的聚类摘要"发送给 LLM；明确不允许其他数据 egress。这是"[经验侧车文件](#经验侧车文件)"模式的激进版本——不依赖用户主动记录，而是被动观察。

---

## 五、校准

### 5.1 我已经在做对的
1. **echo-sleuth 无隐含付费依赖**：所有 skill 使用 Claude Code 本地资源（`~/.claude/projects/`），不需要额外 API key——避免了 scientific-agent-skills 的透明度问题
2. **claude-for-legal 的 `metadata.skill-author` 使用**：legal skill 里维护了贡献者信息，和 scientific-skills 的归因机制一致
3. **drama-workshop-skills 的 install 命令有实际验证**：每次发版时有人工测试安装流程，不像 scientific-skills 那样通过模板批量生成而缺乏验证

### 5.2 挑战 / 验证
这次案例**挑战**了"大仓库 = 高质量维护"的假设。19,356 stars 的仓库，`license: Unknown` 数量在 1 个月内不降反升，说明增长速度超过了质量维护能力。对我的项目的启示：**规模扩张时，质量门禁必须先于规模扩张**——要先把 `scan_pr_skills.py` 的 license 检测从 warning 升级为 blocking，再合并新 skill。这一点在自己的 claude-for-legal 里同样重要：随着 skill 数量增长，需要自动化质量门禁而不是人工 review。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 有没有隐含的付费 API 依赖（但未在 description 中说明）
grep -rn "API_KEY\|api_key\|OPENROUTER\|openai" /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null
# 命中后：在 description 里加 "(requires XXX_API_KEY)" 提示
```

```bash
# 检查我的 skill 的 license 字段规范性
grep -rn "license:" /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null | grep -E "Unknown|unknown|http|url"
# 命中后：将 URL 格式改为 SPDX 标识符（如 MIT、Apache-2.0、BSD-3-Clause）
```

```bash
# 检查我的 skill 里有没有安装命令重复词问题（类似 uv uv pip）
grep -rn "pip install\|npm install\|bun install" /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null | grep -E "\b(\w+) \1\b"
# 命中后：删除重复的词，并加入 CI 正则检测防止复现
```

### 6.2 灵感 → 实施路径

1. **想法**：为 claude-for-legal 创建类似 `scan_skills.py` 的质量扫描工具
   - **为何可行**：claude-for-legal 有 151 个 SKILL.md，人工 review 成本高，自动化扫描可以持续保证质量
   - **第一步**：基于 nlpm 的 `bin/nlpm-check` 工具，在 `.github/workflows/` 里加一个 PR 检查 workflow，对新提交的 SKILL.md 运行 nlpm-check——约 1 小时

2. **想法**：研究 autoskill 的屏幕录制模式，为 echo-sleuth 添加"被动学习"能力
   - **为何可行**：echo-sleuth 的核心是从过去会话提取知识，autoskill 是从当前操作实时提取——两者可以互补
   - **第一步**：阅读 autoskill/SKILL.md，评估 screenpipe 是否可以集成到 echo-sleuth 的 experience-synthesis skill 里——约 30 分钟研究，再决定是否实施

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 K-Dense-AI/scientific-agent-skills 的核心目的**：科研工作者垂直领域工具的 AI 技能百科，降低专业科学软件的使用门槛
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 高 | 同为垂直领域（法律 vs 科研）专业工具 skill 集合；同有多领域子目录；同有第三方贡献机制 | 领域不同；legal 有 managed-agent 支持；legal 无专有产品集成 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 skill 集合 | 完全不同领域和架构 | 低 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为工具类 skill | 无直接关联 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `license` 字段缺失或不规范 | `find /tmp/my-repos/MarkQWu-claude-for-legal -name "SKILL.md" -exec grep -L "^license:" {} \;` | claude-for-legal：随机抽查 3 个 skill，均无 `license` 字段（整个仓库无一个 skill 有 license 字段） | 高 |
| 隐含 API key 依赖未披露 | `grep -rn "API_KEY" /tmp/my-repos/MarkQWu-claude-for-legal/` | claude-for-legal：`regulatory-legal/skills/reg-feed-watcher/SKILL.md` 可能需要外部 API，未查证 | 中 |
| `metadata.skill-author` 缺失 | `grep -rL "skill-author" /tmp/my-repos/MarkQWu-claude-for-legal/*/skills/*/SKILL.md 2>/dev/null \| wc -l` | 需进一步检查，但基于 echo-sleuth 的模式判断 legal skill 的 metadata 字段可能不完整 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/claude-for-legal`：选 5 个最常用的 skill（如 `product-legal/skills/launch-review/SKILL.md`），补充 `license: MIT`（或对应的实际 license）和 `metadata.skill-author: MarkQWu`，建立规范后通过 PR template 强制新贡献者填写——约 30 分钟建立模板，1 小时补现有 5 个
- `MarkQWu/claude-for-legal`：在 `.github/pull_request_template.md` 里加一条 checklist："[ ] 已在 frontmatter 填写 license 和 metadata.skill-author"

### 7.3 别人的更优方案

1. **领域**：`scan_skills.py` 内建质量扫描工具
   - **本案例做法**：仓库自带 Python 质量扫描脚本，可以本地运行检测 skill 问题 → `scan_skills.py`（repo 根）
   - **我的项目现状**：`claude-for-legal` 无任何质量扫描工具，依赖人工 review；151 个 SKILL.md 人工 review 成本极高
   - **如何借鉴**：基于 `nlpm-check`（本项目已有的独立工具）为 claude-for-legal 写一个简单的 pre-commit hook 或 GitHub Actions workflow

2. **领域**：按工具/任务类型的细粒度 skill 切分
   - **本案例做法**：每个科学计算工具有自己的 skill（scanpy/SKILL.md、anndata/SKILL.md），粒度细、职责明确
   - **我的项目现状**：`claude-for-legal` 的 `product-legal/skills/` 里的 skill 粒度有时较粗（如 `is-this-a-problem/SKILL.md` 覆盖范围模糊），可以参考 K-Dense 按工具而非按"问题大类"切分
   - **如何借鉴**：重新审视 claude-for-legal 里粒度较粗的 skill，考虑是否可以按具体法律工具（合同审查、隐私影响评估等）做更细的切分

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：API key 依赖透明度
- **我的做法**：`claude-for-legal` 的 QUICKSTART.md 明确列出所有前置条件（Claude 账号、特定 plugin 安装），`README.md` 有明确的 "⚠️ 每个输出都需要律师审查" 免责说明
- **本案例做法（弱在哪）**：14 个 skill 的 description 里没有说明需要 `OPENROUTER_API_KEY`，用户运行时才发现付费门槛，信任度受损
- **意义**：提前披露前置条件是用户体验的基本素养，claude-for-legal 在这方面比 scientific-agent-skills 做得好，是一个可以在社区分享的最佳实践

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包围的 YAML 配置块。scientific-agent-skills 的 frontmatter 包含 `name`、`description`、`license`、`metadata.skill-author` 等字段，用于 skill 注册和元信息追踪。

### <a name="SPDX"></a>SPDX 标识符
> Software Package Data Exchange：一套软件许可证的标准缩写系统（如 `MIT`、`Apache-2.0`、`BSD-3-Clause`）。用 SPDX 标识符而不是自由文本（如 "MIT License"、URL）填写 license 字段，可以被工具链自动解析。

### <a name="经验侧车文件"></a>经验侧车文件（Experience Sidecar）
> 一种设计模式：在 AI 会话之外，维护一个记录经验、决策、错误的文档，下次 AI 会话时加载这个文档，让 AI "记住"之前的教训。echo-sleuth 的 lesson-extraction 功能就是在生成这种文件。autoskill 把这个模式推进了一步：不是人工记录，而是自动观察用户操作来生成。

### <a name="screenpipe"></a>screenpipe
> 一个开源工具（https://github.com/screenpipe/screenpipe），在本地运行并录制用户的屏幕活动，提供 HTTP API 供其他程序查询历史操作。K-Dense 的 autoskill 通过 screenpipe 的本地 API（localhost:3030）获取用户的工作流数据，全程不出本机。

### <a name="uv"></a>uv（Python 包管理器）
> Astral 开发的极速 Python 包管理器，`uv pip install` 相当于 `pip install`，但速度快 10-100 倍。scientific-agent-skills 把 `uv pip install` 定为标准安装命令，错写为 `uv uv pip install` 是一个打字错误，导致 shell 命令报错。
