# nexu-io/open-design — 学习案例

**仓库**：https://github.com/nexu-io/open-design
**Stars**：20558 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-04（历史快照）| **生成日期**：2026-06-22（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `examples-driven`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

open-design 是 nexu-io 开源的一个 Claude Code skills 仓库，专注于用自然语言生成富 UI 产物：HTML 幻灯片、网页原型、海报、数据仪表盘等视觉设计作品。20558 stars，是 Claude Code 生态里关注度最高的 skills 仓库之一。

关键事实：
1. 当前（2026-06-22 HEAD）共 **157 个 SKILL.md**，audit 时（2026-05-04）约 63 个——6 周内技能数量增长 2.5 倍
2. 已发布 RELEASE-NOTES-0.10.0.md，完成了一次主版本里程碑
3. 所有 skill 统一输出视觉 artifact（HTML/CSS/JS 文件），无文本型 skill
4. 使用 `od` 自定义 frontmatter 块声明每个 skill 的输出契约（mode、scenario、platform、preview、design_system）

### 1.2 架构剖析

```
open-design/
├── skills/                     # 当前 157 个技能目录，扁平排列
│   ├── html-ppt/               # 主 html-ppt skill（被 14 个 focused-entry skill 调用）
│   ├── html-ppt-dark-theme/    # focused-entry skill（主题变体）
│   ├── html-ppt-presenter-mode-reveal/  # audit 时存在，现已删除
│   ├── blog-post/              # 100 分示范技能
│   ├── dashboard/              # 100 分示范技能
│   ├── dating-web/             # 100 分示范技能
│   ├── gsap-react/             # 新增（0.10.0 版本后）
│   ├── canvas-design/          # 新增
│   ├── shader-dev/             # 新增
│   ├── ai-music-album/         # 新增
│   ├── article-magazine/       # 新增
│   ├── video-downloader/       # 新增
│   ├── saas-landing/           # BUG-02：与 docs/examples/ 存在重复 name
│   └── ... 共 157 个
├── docs/
│   └── examples/
│       └── saas-landing-skill/ # BUG-02：仍存在，name: saas-landing 与上面重复
├── scripts/
│   └── render.sh               # Chrome headless 渲染（含 --no-sandbox）
├── generate_pet_images.py       # 用 curl 调用 OpenAI API
├── finalize_pet_run.py          # 写入 ~/.codex/
├── CLAUDE.md                    # 仅一行，重定向到 AGENTS.md
└── AGENTS.md                    # 实际项目说明
```

- **文件类型分布**：157 个 SKILL.md、0 个 agent、0 个 command、1 个 CLAUDE.md（单行重定向）
- **编排关系**：平铺结构为主。核心例外：14 个 focused-entry html-ppt skill 在 prompt 里显式声明"delegate to master `html-ppt` skill"，形成轻量的主从调用模式
- **frontmatter 结构**：每个 SKILL.md 含标准 `od` 块，声明 `mode`（生成模式）、`scenario`（使用场景）、`platform`（目标平台）、`preview`（预览方式）、`design_system`（设计系统）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「每个技能一种视觉产物，产物契约前置声明」——所有 skill 都生成可以直接在浏览器里打开的 artifact，输出类型在 `od` 块里提前约定好
- **解决什么问题**：把"我想要一个……风格的幻灯片/海报/原型"这类设计需求，拆解成可以被 Claude 精确执行的结构化指令
- **Trade-off**：
  - 选择扁平目录而非分域子目录，找 skill 靠名字前缀区分（`html-ppt-*`），简单直接但 157 个技能在一个目录里浏览不方便
  - `od` 块增加了 frontmatter 维护成本，但换来了工具和人类都可以读懂的输出契约
  - focused-entry 模式（14 个主题变体 skill 各自委托给 master）让主题扩展极低成本，但主 skill 变成了单点故障——master html-ppt 修改会影响所有 14 个变体
- **认知模型**：作者把每个视觉产物类型映射为一个 skill，skill 就是"产物规格书"，Claude 是"按规格书施工的设计师"

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「输出契约前置 + focused-entry 委托」**

两个相互配合的模式：
1. 每个 skill 在 frontmatter 的 `od` 块里声明输出的完整规格（产物类型、场景、平台、预览、设计系统），Claude 在生成前知道"产物是什么样的"
2. focused-entry skill 做单一变体声明（比如"黑暗主题的 html-ppt"），然后把实际生成工作委托给 master skill，自身不含生成逻辑

模式特征清单：
- 每个 skill 的 `od` 块是可机器解析的输出规格（YAML 结构）
- focused-entry skill 的 prompt 极短，核心是"use master skill X"
- master skill 的 prompt 详尽，是真正的生成逻辑
- 100 分 skill（blog-post、dashboard、dating-web）展示了"一个完整 skill 应该是什么样的"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 输出类型明确、有结构的工具集（幻灯片/海报/原型） | ✅ 高度适用 | 输出类型固定，`od` 契约可以完整描述 |
| 探索性、开放结果的工作流 | ❌ 不适用 | 契约前置要求输出类型已知，不适合"随机探索"类需求 |
| 大量主题/样式变体 | ✅ 适用 | focused-entry + master 模式让变体扩展成本接近零 |
| 需要多步骤 agent 编排的复杂任务 | ⚠️ 有限适用 | 当前架构无 agent/command 层，不适合需要多轮决策的工作流 |
| 设计团队共享 skill 库 | ✅ 适用 | `od` 契约是团队对齐的共同语言 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：输出契约前置 + focused-entry 委托 | nexu-io/open-design | 输出类型明确，变体扩展成本低 | 无编排层，复杂多步任务难处理 |
| 分层命令编排（commands → agents → skills） | nlpm（本项目） | 支持复杂多步工作流，依赖关系明确 | 架构成本高，小需求过设计 |
| 个人工作流约定优于配置 | lijigang/ljg-skills | 灵活，每个 skill 独立，可随时增删 | 无输出契约，依赖隐式约定 |
| 领域专项插件 | google-gemini/gemini-skills | 对齐产品边界，定向分发 | 修改需协调团队 |

### 2.4 改进空间

1. **当前问题**：157 个 skill 全部扁平堆在 `skills/` 下，无分类索引。**改进做法**：在 AGENTS.md（或新建 skills/README.md）里按产物类型（幻灯片类、原型类、海报类、工具类）分表列出，每个 skill 一行，注明 `od.mode` 值。**预期收益**：新用户 5 分钟内找到对应 skill，不需要扫描全部 157 个目录名。

2. **当前问题**：BUG-02（`saas-landing` 重名）持续存在，导致两个 skill 互相遮蔽，用户调用 `saas-landing` 时实际触发哪个取决于 Claude 的内部排序，结果不可预测。**改进做法**：把 `docs/examples/saas-landing-skill/SKILL.md` 的 `name:` 改为 `saas-landing-example`（与目录语义一致），或删除该目录（示例目的已由主 skill 的 `<example>` 块满足）。**预期收益**：消除 silent misdispatch，示例目录回归纯文档身份。

3. **当前问题**：CLAUDE.md 是单行重定向文件（score 80），没有项目说明。**改进做法**：在 CLAUDE.md 里加入项目概览（产物类型、技能数量、`od` 块格式说明、focused-entry 使用指引），引导新用户快速上手。**预期收益**：减少"为什么有 14 个 html-ppt-*"的迷惑感。

---

## 三、过去审查发现（2026-05-04 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-05-04 当时得分 **91/100**。

| 技能/文件 | 当时分数 | 主要问题 |
|---|---|---|
| blog-post、dashboard、dating-web | 100/100 | — |
| editorial-collage-deck | 78/100 | 输出契约不完整，缺少 platform 声明 |
| pptx-html-fidelity-audit | 78/100 | 工具类 skill 混入视觉产物集合，定位模糊 |
| CLAUDE.md | 80/100 | 仅一行重定向，无项目说明 |
| 其余大多数 skill | ~90/100 | — |
| 整体 | 91/100 | 63 个 artifact（61 SKILL.md + 1 CLAUDE.md + 1 example SKILL.md） |

### 3.2 当时值得借鉴的模式

1. **`od` 块——机器可读的输出契约** → 为什么好：每个 SKILL.md frontmatter 里的 `od` 块（mode/scenario/platform/preview/design_system）让工具和人都能解析输出规格，不需要阅读全文才知道这个 skill 产出什么 → 原文：各高分 skill 的 frontmatter → 借鉴：任何输出类型固定的 skill 都应该有结构化的输出契约声明

2. **14 个 focused-entry html-ppt skill 委托给 master** → 为什么好：每个主题变体（深色/演讲者模式/大字/极简……）只声明自己的差异点，共同逻辑在 master 里维护一处，修改不需要同步 14 个文件 → 借鉴：有多个变体的 skill 集合，应该提取 master skill，变体用委托模式

3. **100 分 skill 作为范例（blog-post、dashboard、dating-web）** → 为什么好：三个 100 分 skill 的结构（完整 `od` 块 + 清晰 prompt + `<example>` 块）是整个仓库其他 skill 的质量标杆 → 借鉴：在 skill 库里留 1-2 个"完美示范 skill"，作为新 skill 的模板参考

4. **`od.preview` 字段** → 为什么好：`preview: browser-live` 明确告诉 Claude 如何预览产物（直接在浏览器打开），避免了生成 skill 但不知道如何验证输出的问题 → 借鉴：输出 artifact 的 skill 应该声明验证/预览方式

### 3.3 当时的缺陷

1. **BUG-01：`skills/html-ppt-presenter-mode-reveal/SKILL.md` 的 `name:` 字段是 `html-ppt-presenter-mode`（缺少 `-reveal` 后缀）** → 根本原因：重命名目录时忘记同步更新 SKILL.md 里的 `name:` 字段——目录叫 `presenter-mode-reveal`，name 声明还是 `presenter-mode`，master skill 按 `presenter-mode-reveal` 调用时找不到匹配项，触发 silent misdispatch → 自查：我的 skill 仓库里有没有 `name:` 字段与目录名不一致的情况？

2. **BUG-02：`skills/saas-landing/SKILL.md` 与 `docs/examples/saas-landing-skill/SKILL.md` 都声明 `name: saas-landing`** → 根本原因：示例目录里的 SKILL.md 沿用了主 skill 的 name，导致两个不同文件声明同一个名字，Claude 会随机触发其中一个 → 自查：我的项目里有没有示例或文档目录里存放了同名 skill？

3. **CLAUDE.md 只有一行重定向** → 根本原因：项目快速增长时文档维护优先级最低，CLAUDE.md 没有随 skill 数量扩张而更新 → 自查：我的项目 CLAUDE.md 是否还能准确反映当前架构？

### 3.4 当时的优化机会

1. **editorial-collage-deck 和 pptx-html-fidelity-audit 输出契约不完整**：`od` 块缺少 `platform` 和 `preview` 字段，与仓库其他 skill 格式不一致
2. **示例目录 `docs/examples/` 的 SKILL.md 是可被调用的真实 skill**：如果示例目录的目的是展示"怎么写 skill"，那里面的 SKILL.md 不应该有可调用的 `name:` 声明——应该改名或去掉 `name:` 字段
3. **`render.sh` 里 Chrome `--no-sandbox`**：在 CI/Docker 环境里确实需要，但应该加注释说明为什么，否则后来者会误认为是安全疏忽

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| BUG-01：html-ppt-presenter-mode-reveal name 字段错误 | `ls skills/html-ppt-presenter-mode-reveal/` | **已解决**：该目录已删除（技能整体移除或合并） | 通过删除解决问题；master skill 对应的调用引用也已清理 |
| BUG-02：saas-landing 重名 | `grep "name: saas-landing" skills/saas-landing/SKILL.md docs/examples/saas-landing-skill/SKILL.md` | **仍存在**：两个文件都声明 `name: saas-landing` | 持续暴露 silent misdispatch 风险，6 周未修复 |
| CLAUDE.md 单行重定向 | `wc -l CLAUDE.md` | **仍存在**：仍然是单行，指向 AGENTS.md | AGENTS.md 是实际说明，CLAUDE.md 作为跳转层存在 |

### 4.2 架构演进

audit 时（2026-05-04）约 63 个 skill，现在（2026-06-22）157 个——6 周内增长 +94 个，增幅 149%：

主要新增方向：
- **前端框架技术类**：gsap-react、canvas-design、shader-dev（从"生成静态 HTML"扩展到"生成框架代码"）
- **多媒体类**：ai-music-album（音乐专辑视觉）、video-downloader（视频工具）
- **内容型视觉产物**：article-magazine（杂志排版）

这说明 open-design 正在从"UI 幻灯片/原型"扩展到更广义的"任何视觉/多媒体产物生成"，边界在持续拓宽。

RELEASE-NOTES-0.10.0.md 的存在说明团队已有版本管理意识，major version 里程碑对用户是明确的稳定性信号。

### 4.3 新增的可学习模式

1. **技术栈专项 skill（gsap-react、shader-dev）**：不再只是"生成外观"，而是生成特定技术栈的可运行代码。这扩展了`od` 契约的语义——`platform: react` 意味着产物是 React 组件，不是独立 HTML 文件。

2. **跨域产物扩展（ai-music-album）**：把"视觉 artifact"的定义拓展到"音乐专辑封面+元数据"，展示了 `od` 块能容纳多维产物描述（视觉 + 内容）的弹性。

---

## 五、校准

### 5.1 我已经在做对的

1. **无模糊量词**：对 drama-workshop-skills 的 grep 确认，`appropriate/comprehensive/robust/efficient` 等模糊量词在我的 skill 文件里命中为零——这与 open-design 的高分 skill 的语言精确度一致
2. **单一输出类型约定**：drama-workshop-skills 的每个 skill 都产出特定格式的剧本（固定结构化输出），与 open-design 的"每个 skill 一种产物"的核心原则一致
3. **质量控制 hooks**：drama-workshop-skills 已有 pre-commit quality hooks，open-design 没有——这一点我的项目比对标仓库更完善

### 5.2 挑战 / 验证

这次案例**挑战**了我对"示例目录无害"的假设。`docs/examples/saas-landing-skill/` 里的 SKILL.md 因为声明了 `name: saas-landing`，从"文档"变成了"可调用的竞争 skill"——示例目录不是安全区，里面的 SKILL.md 会被 Claude Code 当作真实技能加载。

这对我的 claude-for-legal 有直接意义：如果我有任何 `docs/` 或 `examples/` 目录里存放了演示用的 SKILL.md，需要检查它们的 `name:` 字段是否和真实 skill 冲突。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 库里有没有 name 字段重复的 SKILL.md（BUG-02 同类问题）
find . -name "SKILL.md" -exec grep -h "^name:" {} \; | sort | uniq -d
```
命中后怎么办：找到重名的两个 SKILL.md，确定哪个是"真实 skill"哪个是"示例"，把示例的 `name:` 改为 `<name>-example` 或删除 `name:` 字段。

```bash
# 检查 docs/ 或 examples/ 目录下有没有带 name 字段的 SKILL.md（安全区检查）
find docs/ examples/ -name "SKILL.md" -exec grep -l "^name:" {} \; 2>/dev/null
```
命中后怎么办：这些文件会被 Claude Code 当作真实技能加载，与主 skill 重名时触发 silent misdispatch。应去掉 `name:` 字段或重命名。

```bash
# 检查目录名与 SKILL.md 里 name 字段是否一致（BUG-01 同类问题）
for d in skills/*/; do
  skill_name=$(basename "$d")
  declared_name=$(grep "^name:" "$d/SKILL.md" 2>/dev/null | head -1 | sed 's/name: *//')
  if [ -n "$declared_name" ] && [ "$declared_name" != "$skill_name" ]; then
    echo "MISMATCH: dir=$skill_name declared=$declared_name"
  fi
done
```
命中后怎么办：把 `name:` 字段改为与目录名一致（通常目录名是正确的，因为调用时用的是目录路径）。

### 6.2 灵感 → 实施路径

1. **想法**：在 claude-for-legal 的 SKILL.md 里引入 `od` 块（改造为法律专项版本）
   - **为何可行**：法律 skill 的输出同样有固定格式（合同/备忘录/分析报告/意见书），用 `od` 块声明 `mode: contract-draft` / `scenario: commercial` / `output_format: markdown` 等字段，可以让使用者一眼知道产物规格
   - **第一步**：选 claude-for-legal 里得分最高的一个 skill（比如 `commercial-legal/`），在它的 frontmatter 加 `od:` 块，测试 Claude Code 能否正确解析；预计 15 分钟

2. **想法**：给 drama-workshop-skills 的短剧 skill 提取 master + focused-entry 结构
   - **为何可行**：drama-workshop-skills 的 `short-drama` 和 `short-drama-remake` 是同一个基础逻辑的两个变体，目前各自有完整 prompt，改成 `short-drama`（master）+ `short-drama-remake`（focused-entry，仅声明"改编"差异点）可以减少维护重复
   - **第一步**：识别两个 skill 中完全重复的 prompt 段落（生成格式约定、输出结构规范），把这些段落提取到 `short-drama/SKILL.md`，`short-drama-remake/SKILL.md` 改为"use short-drama as master, apply remake guidelines below"；预计 40 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 nexu-io/open-design 的核心目的**：用 skill 集合把"描述一种视觉产物"的自然语言需求转换成可在浏览器里打开的 HTML/CSS/JS artifact

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为输出导向的 skill 库（产出结构化文档），有主变体关系（short-drama / short-drama-remake） | 输出是文字剧本而非视觉 artifact；无 `od` 契约块 | 高 |
| MarkQWu/claude-for-legal | 中 | 同为多技能平铺结构，按域组织，每个 skill 对应固定格式的法律文档 | 按法律领域分子目录（commercial/corporate/employment/…），比 open-design 更有层次 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code plugin | 会话挖掘工具，不产出视觉 artifact，架构完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| BUG-02：多个 SKILL.md 声明同一 `name:` | `find . -name "SKILL.md" -exec grep -h "^name:" {} \; \| sort \| uniq -d` | **待检查**：claude-for-legal 有多个域（5 个子目录），如果不同域的 skill 碰巧同名会触发同类问题 | 高 |
| 示例目录的 SKILL.md 可被 Claude 加载 | `find docs/ examples/ -name "SKILL.md" \| xargs grep -l "^name:"` | **待检查**：drama-workshop-skills 和 claude-for-legal 是否有 docs/ 目录存放示例 SKILL.md | 高 |
| 输出契约仅靠 prompt 文字描述，无结构化字段 | 检查 frontmatter 有没有 `od:` 块 | **drama-workshop-skills 命中**：skill 依赖 prompt 内嵌格式说明，无机器可解析的输出契约 | 中 |
| CLAUDE.md 无项目说明 | `wc -l CLAUDE.md` | **待确认**：我的项目 CLAUDE.md 内容是否足够说明架构 | 低 |

**命中后的具体行动建议**：
- 对 claude-for-legal，运行 `find . -name "SKILL.md" -exec grep -h "^name:" {} \; | sort | uniq -d`，确认无重名——如有，立即修复，因为法律文档错触发错误 skill 会产生实质性伤害（生成了错误的法律文件类型）
- 对 drama-workshop-skills，在 `short-drama/SKILL.md` 的 frontmatter 加 `od:` 块示范：`mode: drama-script`、`scenario: short-form-video`、`output_format: structured-markdown`；10 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：`od` 块——结构化输出契约前置声明
   - **本案例做法**：每个 SKILL.md 的 frontmatter 里有 `od:` 块，声明 `mode/scenario/platform/preview/design_system`，Claude 在生成前就知道产物规格
   - **我的项目现状**：drama-workshop-skills 和 claude-for-legal 的输出格式描述全部内嵌在 prompt 文字里，工具无法解析，人工对齐成本高
   - **如何借鉴**：在 drama-workshop-skills 的每个 SKILL.md 加 `od:` 块，字段改为法律/剧本专项：`mode: script-draft`、`output_format: structured-markdown`、`scenario: short-drama-production`；不需要与 open-design 的字段完全一致，关键是机器可读的结构

2. **领域**：focused-entry + master skill 委托模式
   - **本案例做法**：14 个 `html-ppt-*` 变体 skill 各自声明差异点，共同逻辑委托给 `html-ppt` master，维护一处即可
   - **我的项目现状**：drama-workshop-skills 的 `short-drama` 和 `short-drama-remake` 有大量重复 prompt（格式约定、输出结构规范）
   - **如何借鉴**：提取重复段落到 `short-drama/SKILL.md`，`short-drama-remake` 改为 focused-entry

3. **领域**：100 分示范 skill 作为质量标杆
   - **本案例做法**：blog-post、dashboard、dating-web 是整个仓库的"模板参考 skill"，新 skill 按这三个的结构写就能达到高分
   - **我的项目现状**：我没有明确指定哪个 skill 是"范例"
   - **如何借鉴**：在 CLAUDE.md 里注明"写新 skill 时参考 `commercial-legal/contract-draft/SKILL.md`（最完整的 skill）"

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：manifest discipline（清单纪律）
   - **我的做法**：drama-workshop-skills 的 skill 命名遵循强约定（`short-drama-*` 前缀），且 README 中对每个 skill 有功能说明，skill 数量也在可管理范围（不到 20 个）
   - **本案例做法**：157 个 skill 扁平堆在 `skills/` 目录，无分类索引，BUG-02 重名问题持续存在 6 周未修复，manifest discipline 随规模增长在退化
   - **意义**：skill 数量超过 100 个后，没有自动化重名检查和分类索引，维护质量会快速下滑。我的项目当前规模小，应该趁规模小时建立命名唯一性自动检查（pre-commit hook），而不是等到 150+ 再修

2. **领域**：无模糊量词
   - **我的做法**：drama-workshop-skills 所有 SKILL.md 中 `appropriate/comprehensive/robust/efficient` 命中为零
   - **本案例做法**：部分低分 skill（editorial-collage-deck，78 分）包含模糊指令，是扣分原因之一
   - **意义**：清晰的语言精确度从一开始就保持是最低成本的——事后修改模糊量词比新写一个 skill 更费时

3. **领域**：质量控制 hooks
   - **我的做法**：drama-workshop-skills 有 pre-commit quality hooks，能在写入前拦截质量问题
   - **本案例做法**：open-design 无 quality hooks，BUG-01（name 字段错误）、BUG-02（重名）都是可以被机械检查拦截的问题，但没有被拦截
   - **意义**：质量控制的最低成本是在 write 时拦截，不是在 audit 时发现。open-design 的两个 bug 都是"写 SKILL.md 时可检查的机械问题"，一个简单的 hook 就能预防

---

## 八、术语表

### <a name="od-block"></a>od 块
> open-design 仓库发明的 SKILL.md frontmatter 自定义 YAML 块，格式为：
> ```yaml
> od:
>   mode: html-presentation
>   scenario: business-pitch
>   platform: browser
>   preview: browser-live
>   design_system: tailwind
> ```
> 作用是在 SKILL.md 开头用机器可读的结构声明"这个 skill 生成什么"，让工具扫描和人工审查都不需要阅读全文就能理解产物规格。类比：就像 npm 的 `package.json` 声明包的入口和依赖，`od` 块声明 skill 的输出契约。

### <a name="focused-entry-skill"></a>focused-entry skill
> 一种只声明"我是 X 的特定变体"的轻量 skill，自身不包含生成逻辑，而是把实际工作委托给 master skill。open-design 里 14 个 `html-ppt-*` 变体（如 `html-ppt-dark-theme`、`html-ppt-minimal`）就是 focused-entry skill，它们的 prompt 核心只有一句"use master html-ppt skill with these theme constraints"。优点：主题变体扩展成本接近零；缺点：master skill 成为单点故障。

### <a name="silent-misdispatch"></a>silent misdispatch（静默误触发）
> 当两个 SKILL.md 声明了相同的 `name:` 字段，Claude Code 在调用该名字时会随机选择其中一个，用户没有任何错误提示。这比"找不到 skill"更危险，因为用户以为调用成功了，实际上触发的是另一个 skill。open-design 的 BUG-02（`saas-landing` 重名）就是典型的 silent misdispatch 场景。预防方法：在 pre-commit hook 里检查 `find . -name "SKILL.md" -exec grep -h "^name:" {} \; | sort | uniq -d`，有输出即报错。

### <a name="artifact"></a>artifact（产物）
> Claude Code 执行 skill 后生成的输出文件。open-design 的所有 skill 产物都是可以在浏览器里打开的 HTML/CSS/JS 文件。更广义地说，artifact 是 skill 的"交付物"——可以是文本文档、代码文件、图像、配置文件等任何格式。与 Claude Code 内置的 Artifacts 功能同名但不同概念：这里的 artifact 特指 skill 生成并保存到磁盘的文件，不是 Claude 界面里的内联预览块。
