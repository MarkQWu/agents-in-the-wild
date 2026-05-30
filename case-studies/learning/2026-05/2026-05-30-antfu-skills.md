# antfu/skills — 学习案例

**仓库**：https://github.com/antfu/skills
**Stars**：4700 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-30（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `examples-driven`, `manifest-discipline`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Anthony Fu（Vue/Vite/Nuxt 核心贡献者，GitHub 上约 60 万关注者）的个人 [Agent Skills](#agent-skills) 合集——以 `pnpx skills add antfu/skills --skill='*'` 一键安装全部技能。仓库本身是一个**工具**，用于从项目文档自动生成 skill 文件并与上游 vendor 保持同步；附带 17 个手工调优或生成的 SKILL.md，覆盖 Vue/Nuxt/Vite/pnpm/UnoCSS/Turborepo 等整个前端工具链。

关键事实：
- 创建于 2025 年，定位为"proof-of-concept：从源文档生成 skill 并持续同步"
- 安装方式：通过 `@vercel-labs/skills` CLI 工具，而非 Claude Code 的 `plugin install`
- 生态位：展示"文档驱动 skill 生成"的范式，比手写 skill 更具可持续性
- 4700 stars 表明这一范式引发了广泛共鸣

### 1.2 架构剖析
```
antfu/skills
├── scripts/cli.ts          # 主入口：生成/同步 skill
├── meta.ts                 # skill 源配置（submodule 路径、同步规则）
├── sources/                # Type 1：从 OSS 文档生成（Vue, Nuxt, Vite, UnoCSS）
│   └── {project}/docs/
├── vendor/                 # Type 2：从上游 skill 仓库同步（Slidev, VueUse）
│   └── {project}/skills/{skill-name}/
├── skills/                 # 最终产物：17 个手工 + 生成的 SKILL.md
│   ├── antfu/SKILL.md
│   ├── turborepo/SKILL.md  # 912行，R05 超限
│   ├── vueuse-functions/SKILL.md  # 419行，0个代码块
│   └── vue-best-practices/SKILL.md  # references/ vs reference/ 分歧
├── instructions/           # Claude Code 原生 instructions 格式（最新添加）
│   └── {nuxt,pinia,pnpm...}.md
├── AGENTS.md               # 面向 agent 的生成指令（audit 后新增）
└── package.json            # private: true，pnpm workspace
```

- **文件类型分布**：17 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：skills 之间平列，无 router 或 meta skill；`meta.ts` 控制生成流程
- **跨件契约**：每个 skill 通过 `references/` 子目录存放长参考文档，SKILL.md 主体通过表格行引用

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「文档即 skill 的单一数据源」—— 上游文档变了，重新生成；不手动维护两份知识
- **解决什么问题**：手写 skill 追不上框架更新速度；作者维护多个活跃 OSS 项目，skill 手写成本太高
- **Trade-off**：生成 skill 牺牲了精细化的 Examples 段（生成器不知道哪种用法最常见），换取了覆盖面广和可持续同步
- **认知模型**：把 skill 看作"已处理的文档缓存"而非"代理人行为规范"——更近 RAG 而非指令集

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「文档驱动生成 + Vendor 同步」**

特征清单：
- 特征 1：有两类 skill 来源（自生成 vs. 上游同步），在 meta.ts 中显式声明
- 特征 2：skills/ 目录是**产物**而非源码；源头是 sources/ 和 vendor/
- 特征 3：`scripts/cli.ts` 是 skill 工厂，支持 submodule 拉取 + 内容转换
- 特征 4：每个 skill 有独立 references/ 子目录，实现长文档分层
- 特征 5：AGENTS.md 提供面向 agent 的元指令，指导 skill 如何被生成和维护

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 覆盖多个活跃更新的 OSS 框架 | ✅ 高度适用 | 生成 + 同步解决手动追更问题 |
| 个人偏好和惯例（如代码风格）| ❌ 不适用 | 生成器无法推断作者主观取舍，需手写 |
| 需要丰富 input→output 示例的技能 | ⚠️ 部分适用 | 生成 skill 缺乏行为示例（R06 扣分） |
| 团队共享的工作流 skill | ❌ 不适用 | 依赖个人偏好（antfu 的代码风格），难以通用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 文档驱动生成（本仓库）| antfu/skills | 可持续更新，覆盖面广 | 缺例子，无法捕捉作者的"最佳判断" |
| 手工精写单仓平铺 | xiaolai/skills | 每个 skill 经作者深度打磨 | 维护成本高，追框架更新慢 |
| 多 agent 编排 | wshobson/agents | 支持复杂多步任务 | 对于单点知识传递是过度设计 |

### 2.4 改进空间
1. **当前问题**：`vueuse-functions/SKILL.md` 419 行，0 个代码块，读者无法从文字目录推断用法。**改进做法**：在生成器中为每个 VueUse 分类各注入一个典型 snippet（`useLocalStorage`/`useDraggable`/`useFetch`）。**预期收益**：R06 扣分从 -10 降为 0，更重要的是 agent 调用 skill 时能准确推断调用模式。
2. **当前问题**：`turborepo/SKILL.md` 912 行，生成后从未切割。**改进做法**：把 Critical Anti-Patterns 段（约 500 行）移入 `references/anti-patterns.md`，主文件保留指针行。**预期收益**：R05 扣分从 -10 降为 0，每次调用 skill 时 context 开销减半。
3. **当前问题**：`vue-router-best-practices` 和 `vue-testing-best-practices` 用 `reference/`（单数），其他 skill 用 `references/`（复数）。**改进做法**：统一为 `references/`，加 CONTRIBUTING note。**预期收益**：跨 skill 链接不再需要"记住哪个用单数"。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 得分 **93/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/vueuse-functions/SKILL.md | 77 | -10 无 inline 示例 (R06)，-8 模糊量词 (R01)，-5 体积 400-500 行 (R05) |
| skills/vue-best-practices/SKILL.md | 85 | -10 模糊量词 5 处 (R01)，-5 无 inline 示例 (R06) |
| skills/turborepo/SKILL.md | 86 | -10 体积 >500 行 (R05)，-4 模糊量词 (R01) |
| skills/web-design-guidelines/SKILL.md | 92 | -5 无 inline 示例 (R06)，-3 无 scope note (R07) |
| skills/slidev/SKILL.md | 100 | — |
| skills/vite/SKILL.md | 100 | — |
| skills/vue/SKILL.md | 100 | — |

### 3.2 当时值得借鉴的模式
1. **references/ 分层模式** → 长参考内容不塞进 SKILL.md 主体，而是拆到 `references/*.md` 用表格行链接。优点：主文件保持精简、可快速加载，详细内容按需读取。示例：`skills/nuxt/SKILL.md` + `references/best-practices-ssr.md`。借鉴：我的 skill 超过 200 行时就应拆分 references/。

2. **Do NOT use 条款** → `antfu/SKILL.md` 明确写出 skill **不**覆盖的范围，防止 agent 越权调用。借鉴：每个 skill 的 description 应加"Do NOT use for..."限定语。

3. **100 分 skill 结构** → `slidev/SKILL.md` 和 `vite/SKILL.md` 满分，对比得分低的 skill，差别在于**决策树而非描述清单**——告诉 agent"如果 X 则 Y"而非"X 很重要"。

4. **private:true + submodule 管理** → 把 OSS 源文档纳为 submodule，生成时 clone，不污染主分支。借鉴：有批量生成需求时用 submodule 而非直接 copy 文档。

5. **SKILL.md 与 instructions/*.md 双轨** → audit 后新增了 `instructions/` 目录，兼容 Claude Code 的 [frontmatter](#frontmatter) 格式和 anthropic docs 推荐的 `.claude/` 格式，同一知识库的双出口。

### 3.3 当时的缺陷
1. **R06：419 行的 vueuse-functions/SKILL.md 无一行代码示例** → 根本原因：文档生成器只能复制文字目录，无法推断"哪种调用最能帮助 agent"——内容丰富但对 agent 毫无示范价值。自查：**我的 echo-sleuth 的 `memory-management/SKILL.md` 同样是 0 个代码块**（见 §七.8.2）。

2. **R01：大量"appropriate/complex/truly/overly"等模糊量词** → 根本原因：作者行文自然流畅，但 agent 无法把"too complex"翻译成可执行判断——skill 越写越像给人看的指南，而不是给 agent 的协议。自查：我的 `claude-for-legal` 仓库多处使用"appropriate"，同类问题。

3. **安全：web-design-guidelines 在运行时 WebFetch 外部 URL** → 根本原因：把指南源文档链接写进 skill，运行时 agent 去拉内容——上游仓库被攻击即成为 agent 的指令注入向量。即便概率低，对安全敏感场景不可接受。自查：我的 skill 暂无此模式，但值得留意。

### 3.4 当时的优化机会（Top 3）
1. `vueuse-functions`：每个功能分类各添一个 snippet，让 catalog 有示范锚点
2. `turborepo`：Anti-Patterns 段拆入 references/，主文件保持 200 行以内
3. `scripts/cli.ts`：`rmSync` 前加 `path.resolve` 断言路径在 root 下，防止 `.gitmodules` 路径逃逸

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| turborepo >500 行 (R05) | `wc -l skills/turborepo/SKILL.md` | **持续** (914 行) | 作者有意保留完整内容，认为拆分带来的导航成本更高 |
| vueuse-functions 无代码示例 (R06) | `grep -c '^\`\`\`' skills/vueuse-functions/SKILL.md` → 0 | **持续** | 生成器没有加 snippet 逻辑，需要手动改进 |
| turborepo 模糊量词"truly" (R01) | `grep -n "truly" skills/turborepo/SKILL.md` | **持续**（第 77、403 行） | 这类措辞在作者笔法中根深蒂固，不被视为 bug |

### 4.2 架构演进
Audit 后的变化：
- 新增 `AGENTS.md`：面向 agent 的元指令，规定如何生成和维护 skill（Audit 后新增）
- 新增 `instructions/` 目录：输出 Claude Code 标准 instructions 格式（`.md` 文件，无 frontmatter 依赖）
- 新增 `vendor/` submodule 体系：VueUse 等上游 skill 仓库改为 vendor 同步而非手工维护

这说明作者在 audit 后意识到：**让 skill 生成流程本身对 agent 透明**，比单纯修复个别 skill 更有长期价值。AGENTS.md 是这一认知的落地。

### 4.3 新增的可学习模式
- **AGENTS.md 元指令模式**：在仓库根目录放一个专门给 agent 的文件，解释"这个仓库是什么，你（agent）在这里如何工作"。这让 agent 接手维护任务时有上下文锚点，而不是从零读代码。
- **instructions/ 双轨出口**：同一 skill 知识库，一份给 Claude Code 的 SKILL.md（带 [frontmatter](#frontmatter)），一份给 Claude Code 的 instructions/*.md（无 frontmatter，更轻量）。适合 skill 内容稳定后减少注册开销。

---

## 五、校准

### 5.1 我已经在做对的
1. **references/ 分层**：我的 `echo-sleuth` 在 `skills/jsonl-core/references/` 下放了 `extraction-patterns.md`——和 antfu 的做法完全一致
2. **Do NOT use 限定语**：我的 `claude-for-legal` 的 `launch-review/SKILL.md` 中有"不用于……"式约束，防止越权
3. **代码示例**：`drama-workshop-skills` 的 `short-drama/SKILL.md` 有 24 个代码块，示例覆盖充分

### 5.2 挑战 / 验证
这次案例挑战了我的一个假设：**高分 skill 一定有 inline 示例**。

`slidev/SKILL.md` 和 `vite/SKILL.md` 得了 100 分，但它们也没有示例——它们赢在**决策树结构**而非示例。`vueuse-functions` 扣分 -10 不是"没示例"本身，而是"419 行目录+0 示例"的组合——长度越大，无示例的代价越高。

结论：**示例是给长文档做锚的**，不是给短 skill 的必选项。短 skill（<100 行）用好决策树同样可以满分。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的 skill 是否有模糊量词（R01）
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|truly|overly|suitable|balanced)\b' \
  /tmp/my-repos/MarkQWu-*/  --include="SKILL.md" | grep -v "CODE_OF_CONDUCT\|LICENSE"
```
命中后怎么办：把命中行的模糊词替换为可量化的标准（数字阈值、文件名模式、明确条件）。

```bash
# 检查长 SKILL.md 是否有代码示例（R05 + R06 组合检测）
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" | while read f; do
  lines=$(wc -l < "$f")
  blocks=$(grep -c "^\`\`\`" "$f" 2>/dev/null || echo 0)
  if [ "$lines" -gt 200 ] && [ "$blocks" -lt 2 ]; then
    echo "RISK: $lines lines, $blocks code blocks — $f"
  fi
done
```
命中后怎么办：每 100 行添加至少 1 个代表性 snippet。

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 的 `memory-management/SKILL.md` 加 inline 示例
   - **为何可行**：该 skill 现在 0 个代码块，且功能是内存管理操作（有具体的调用模式可展示）
   - **第一步**：读 `skills/memory-management/SKILL.md`，找到 2-3 个最核心的操作，各写一个 5 行以内的伪代码 snippet，约 15 分钟

2. **想法**：给 drama-workshop-skills 添加 AGENTS.md 元指令
   - **为何可行**：该仓库有 NL 工件但没有给维护 agent 的上下文文件，下次让 Claude Code 维护时会缺少锚点
   - **第一步**：仿 antfu/skills 的 AGENTS.md，写 3 段：这个仓库是什么、skill 如何组织、如何添加新 skill，约 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **antfu/skills 的核心目的**：为个人前端技术栈（Vue/Nuxt/Vite 生态）提供可自动同步的 agent skill 集合
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 都是 skill 集合，有专属技术领域（短剧 vs 前端） | antfu 有自动生成流程，我全靠手写 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 SKILL.md | echo-sleuth 是工具 skill，antfu 是知识 skill | 中 |
| MarkQWu/claude-for-legal | 低 | 都是多领域 skill 集合 | 领域完全不同（法律 vs 前端） | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| R06：无 inline 示例的 SKILL.md | `grep -c '^\`\`\`' skills/memory-management/SKILL.md` | **echo-sleuth `memory-management/SKILL.md` 命中**：0 个代码块 | 中 |
| R01：模糊量词 "appropriate" | `grep -rn "appropriate" claude-for-legal/ --include="SKILL.md"` | **claude-for-legal 命中 3 处**（matter-workspace×3） | 低 |

**命中后的具体行动建议**：
- `echo-sleuth/skills/memory-management/SKILL.md` → 添加"保存记忆"和"检索记忆"各一个 snippet，展示调用格式 → 15 分钟可完成
- `claude-for-legal/*/skills/matter-workspace/SKILL.md` → 把"appropriate"替换为具体决策条件（如"当事项涉及多个当事方时"）→ 每处 5 分钟

### 8.3 别人的更优方案

1. **领域**：references/ 分层配合 skill 体积控制
   - **antfu 做法**：`skills/nuxt/SKILL.md`（主体精简）+ `references/best-practices-ssr.md`（详细参考）
   - **我的现状**：`drama-workshop-skills/short-drama-remake/SKILL.md` 251 行，没有 references/ 子目录
   - **如何借鉴**：把 short-drama-remake 的"技术规则"段提取到 `references/tech-rules.md`，主文件保留决策指针行

2. **领域**：AGENTS.md 元指令让 agent 接手时有锚点
   - **antfu 做法**：`AGENTS.md` 解释仓库结构和 skill 生成规则，agent 维护时开箱即用
   - **我的现状**：三个仓库都没有 AGENTS.md
   - **如何借鉴**：为 echo-sleuth 写一个 200 字的 AGENTS.md，说明 skill 的组织逻辑和扩展方式

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：代码示例密度
- **我的做法**：`drama-workshop-skills/short-drama/SKILL.md` 有 24 个代码块，示例覆盖"输入→输出"全链路
- **antfu 做法**（弱在哪）：`vue-best-practices/SKILL.md` 0 个代码块，仅有文字规范
- **意义**：我对"示例驱动 skill"的重视已经高于 antfu 在这类 skill 上的投入——这是可以保持的竞争优势

---

## 八、术语表

### <a name="agent-skills"></a>Agent Skills
> Claude Code 中的"技能文件"（SKILL.md）——告诉 AI agent 如何在特定领域工作的结构化指令文件。类比：程序员的"工作手册"，AI 读完后知道该怎么做某类任务。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置段，声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才知道如何注册和调用这个 skill。

### <a name="vendor"></a>vendor 同步
> 把上游项目（如 VueUse、Slidev）作为 git submodule 引入，定期拉取更新后自动提取其 skill 文件。相当于"订阅上游的 skill 更新"，比手动 copy-paste 可维护性高一个数量级。
