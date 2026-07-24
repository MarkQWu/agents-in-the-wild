# levnikolaevich/claude-code-skills — 学习案例

**仓库**：https://github.com/levnikolaevich/claude-code-skills
**Stars**：423 | **来源**：upstream（近 500 星，星数门槛临界）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-24（基于当前 HEAD）
**主题标签**：`template-design`, `manifest-discipline`, `vague-quantifier`, `single-purpose`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
levnikolaevich/claude-code-skills 是一套「交付全生命周期」的 Claude Code 插件套件，含 MCP servers（hex-line 哈希验证编辑、hex-graph 知识图谱、hex-ssh 远程 SSH）和 288+ NL 工件（审计时）。目标是从需求拆分→编码→测试→文档→发布，一套技能全覆盖。

关键事实：
- 2026 年 4 月审计时：288 工件，分布在 `skills-catalog/`（100+）、`plugins/`（68 个适配器）、`.claude/` 等
- **当前 HEAD（2026-07-24）**：重大重构——旧的 `skills-catalog/`、`shared/agents/prompt_templates/` 目录已不存在，仅剩 6 个插件目录 + 44 个文件总量、21 个 MD 文件、1 个 SKILL.md
- 现在的定位从「单一 monorepo NL 技能库」转变为「多插件套件分发平台」

### 1.2 架构剖析（当前 HEAD）

```
claude-code-skills/
├── plugins/
│   ├── codebase-audit-suite/
│   │   ├── .codex-plugin/plugin.json
│   │   └── skills/
│   │       └── ln-41-test-strategy-planner/SKILL.md  ← 全库唯一 SKILL.md！
│   ├── maintainer-suite/
│   ├── optimization-suite/
│   ├── product-discovery-suite/
│   ├── review-suite/
│   └── testing-suite/
├── site/                       # 新增：静态网站展示页
│   ├── index.html
│   └── plugins/*.html
├── .agents/plugins/            # Codex 全局 agent registry
├── .claude-plugin/marketplace.json
└── CLAUDE.md / AGENTS.md       # 项目入口（CLAUDE.md → AGENTS.md 重定向模式）
```

- **文件类型分布（当前）**：6 个插件 × 1 plugin.json = 6 个 manifest；1 个 SKILL.md；21 个 MD 文件（含 README/CHANGELOG/AGENTS.md 等）
- **对比审计时**：288 工件 → 21 MD 文件，缩减 93%——这不是缺少内容，而是**架构模型根本性转变**
- **编排关系**：每个 plugin 是独立可安装单元，插件间不直接调用

### 1.3 设计思路 / 方法论
- **当时（审计时）哲学**：「统一技能目录 + Codex 薄适配器」——`skills-catalog/` 是规范的真实来源，`plugins/*/skills/` 只是 1:1 映射的 wrapper。系统化但维护代价高。
- **现在（重构后）哲学**：「插件套件化」——不再维护庞大的技能目录，而是把 6 个完整插件各自打包，网站展示，用户按需安装。
- **Trade-off**：旧版本：规模化维护难（288 SKILL.md 无一有 examples，全靠适配器层）；新版本：减少了工件数量，但也减少了可查阅学习的内容。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**（审计时）模式名：「规范目录 + 薄适配器层」**

原版的设计是：一个经过精心设计的 `skills-catalog/`（带 ln-编号的阶段化技能）作为真实来源，`plugins/` 下的 SKILL.md 每个只有几行，内容是 `MANDATORY READ: ../../../skills-catalog/ln-xxx.md`。

特征清单（审计时）：
- 68 个适配器 SKILL.md 全部只做转发（"Codex adapter"）
- 共享 prompt_templates 用模板变量 `{mode_header}` 等组合
- 版本控制在 canonical skill，适配器不单独管理版本

**（当前）模式名：「独立插件套件」**

6 个自包含插件，网站展示，独立发布。单一真实来源的思路转变为「每个套件独立管理」。

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要维护 100+ skills 的大型项目 | ⚠️ 谨慎使用旧版模式 | 规范+适配器模式理论优雅，但无 examples 的维护成本会积累 |
| 分发给不同用户不同子集 | ✅ 新版插件套件模式适用 | 按需安装，不必全量 |
| 需要跨多个 AI 运行时（Claude/Codex/Qwen） | ⚠️ 参考 josstei/maestro 更成熟 | 本仓库当前更专注 Claude Code |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 规范目录 + 薄适配器（原版） | 本仓库（审计时） | 单一真实来源，理论无重复 | 无 examples，适配器版本漂移 |
| 插件套件（当前重构后） | 本仓库（当前） | 用户按需安装 | 内容大幅减少 |
| 平铺扩展（无分层） | agent-sh/agentsys | 简单直接，CI 验证容易 | 工件多时难以组织 |

### 2.4 改进空间
1. **当前问题**：全库仅剩 1 个 SKILL.md，6 个 plugin.json 里的 skills 字段可能为空 **改进做法**：给每个插件套件加入至少 1-3 个实质性 SKILL.md，每个带完整 examples **预期收益**：让插件对用户可见且可评估
2. **当前问题**：重构后 `shared/agents/prompt_templates/` 消失，审计时发现的模板变量契约问题（`{mode_header}` 等）无从追踪 **改进做法**：若要复用模板逻辑，改用显式的 frontmatter `extends:` 字段

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
77/100（288 工件，progressive 策略抽样 ~100 文件）。

| 文件类型 | 当时分数 | 主要问题 |
|---|---|---|
| shared/agents/prompt_templates/*.md（12 个） | 40-50 | 无 frontmatter（name/description）——NL 扫描器无法注册 |
| plugins/*/skills/*.md（68 个适配器） | 80 | 无 model 声明；无 examples |
| skills-catalog/*.md（~100+） | 80-88 | 无 model 声明；无 examples；部分有模糊量词 |
| .claude/commands/*.md | 83-88 | 无空输入处理；个别 allowed-tools 声明有瑕疵 |

### 3.2 当时值得借鉴的模式
1. **ln-编号系统** → 每个技能文件名有三位数编号（`ln-200-scope-decomposer`），代表在交付流水线中的阶段位置，一眼知道调用顺序。借鉴：在有多个串联技能的项目里引入编号前缀。
2. **Codex 适配器薄层** → 如果同一技能需要在不同平台发布，可以用极薄的 wrapper 做命名空间适配，而不是维护两份完整内容。
3. **CLAUDE.md → AGENTS.md 重定向** → 主入口只写一行 `See AGENTS.md`，用于控制上下文加载顺序。借鉴：分离「用户指引」与「agent 规范」。

### 3.3 当时的缺陷
1. **12 个 prompt_templates 无 frontmatter** → NL 扫描器看不到，这些文件等于不存在于 NLPM 注册表。**根本原因**：作者把这些当作「内部模板」而非「公开 skill」，但 NLPM 不区分，没有 frontmatter 就不可被发现。**我有没有？** 我的 gstack 或 bureau 里若有没有 frontmatter 的 `.md` 文件在 skills 目录下，同样会被忽略。
2. **模板变量契约不显式** → `review_base.md` 用 `{mode_header}` 等占位符但无任何文档。**根本原因**：「内部约定」随团队时间累积会成为「鬼文档」。**我有没有？** bureau 的 `compile.md` 里有类似的模板字符串拼接，需要检查。
3. **全部 100 个文件无 examples** → 扣分 -15/文件。**根本原因**：适配器层「内容轻薄」是设计决策，但评分规则不区分 canonical 和 adapter。

### 3.4 当时的优化机会
1. 给 12 个 prompt_templates 加最简 frontmatter（`name: scope-analysis-template` + `description: ...`）即可让扫描器发现
2. 给 canonical skills 各加 1 个 examples 块，适配器继承父技能的示例
3. `ln-820` 的两个运行时路径（`optimization-runtime/cli.mjs` vs `dependency-runtime/cli.mjs`）去重

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| prompt_templates 无 frontmatter | `find . -path '*/prompt_templates/*.md'` | **已消失** — 整个 prompt_templates 目录不复存在 | 作者选择删除而非修复——根本性架构决策 |
| ln-774 缺 Paths header | `grep -n 'Paths' skills-catalog/ln-774-healthcheck-setup/SKILL.md` | **已消失** — skills-catalog 目录不复存在 | 同上 |
| 68 个适配器无 examples | `find plugins/ -name 'SKILL.md' \| wc -l` | **从 68 减至 1** — 仅剩 1 个 SKILL.md（`ln-41-test-strategy-planner`） | 大规模重构，旧适配器全部删除 |

### 4.2 架构演进

**最显著变化**：整个 `skills-catalog/`（含 100+ SKILL.md）被删除。`plugins/` 下原来的适配器层也几乎全清，取而代之的是更轻量的 6 个套件包 + 一个展示网站（`site/`）。

**作者后来意识到什么？**：维护 288 个 NL 工件（全无 examples、全无 model）的代价高于收益。与其维护庞大的低质量目录，不如缩减规模、提高每个发布单元的质量。这是「宁少勿滥」的典型案例。

### 4.3 新增的可学习模式

当前新增 `site/` 目录：每个插件有独立的 HTML 展示页（`review-suite.html`、`codebase-audit-suite.html` 等），这是**「插件市场友好型」架构**的信号——作者在为未来的市场展示做准备，把插件做成可独立展示的产品单元。

---

## 五、校准

### 5.1 我已经在做对的
1. gstack 的命令文件是 Claude Code 官方推荐格式，frontmatter 完整
2. drama-workshop-skills 的技能数量适中（不超过 20 个），每个 SKILL.md 可以保持高质量
3. bureau 的插件结构保持了 manifest discipline（plugin.json 与实际文件对应）

### 5.2 挑战 / 验证
本案例验证了：**「大而全」并不比「小而精」好**。作者花了数月维护 288 个工件，最终全部删除从头来过。这挑战了我认为「功能越多越好」的假设。在设计 bureau 扩展时，应优先保证现有命令质量（有 examples、有 model 声明），而不是快速增加新命令数量。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 目录下有无缺 frontmatter 的 .md 文件
for f in $(find /tmp/my-repos/MarkQWu-bureau -name "*.md" -path "*/skills/*" -not -path "*/.git/*"); do
  if ! head -2 "$f" | grep -q "^---"; then
    echo "NO FRONTMATTER: $f"
  fi
done
# 命中后：至少加 ---\nname: xxx\ndescription: yyy\n--- 三行
```

```bash
# 检查我的项目里有无模板变量占位符无契约文档
grep -rn '{[A-Z_]\+}' /tmp/my-repos/MarkQWu-bureau/.claude/ 2>/dev/null
# 命中后：在 CLAUDE.md 的「变量约定」section 里列出所有占位符及其来源
```

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 6 个核心命令各加 1 个 `## 示例` 块
   - **为何可行**：bureau 目前命令无 examples，得分在 80 左右
   - **第一步**：从真实会话里挑一个 `/bureau:note` 的输入输出，复制为 `## 示例\n**输入**: ...\n**输出**: ...`（每个 10 分钟）

2. **想法**：参考 levnikolaevich 的 ln-编号前缀，给 bureau 的技能链显式标注执行顺序
   - **为何可行**：bureau 的 `init→note→compile→query` 是有序的
   - **第一步**：在 skills 目录里加 `00-session-init/`、`10-capture/`、`20-compile/`、`30-query/` 前缀，用数字表达执行阶段

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例核心目的**：插件化技能套件，覆盖完整软件交付生命周期

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 同样是 Claude Code 多工具套件 | gstack 是「模拟团队角色」，本案例是「流水线阶段化技能」 | 中 |
| MarkQWu/bureau | 低 | 同样有多个命令构成工作流 | bureau 是单一产品（知识库），本案例是全功能开发套件 | 低 |
| MarkQWu/graphify | 低 | 同样把底层逻辑下沉到脚本层 | graphify 专注知识图谱，非交付流程 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷（审计时） | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skills 目录下 MD 文件无 frontmatter | `grep -rL '^---' */skills/**/*.md` | gstack 下有 3 个工具 MD 无 YAML 头 | 中 |
| 所有 SKILL.md 无 examples | `grep -L '## 示例\|## Example' */skills/**/*.md` | bureau/graphify/drama 各有 0-2 个 SKILL 有 examples | 高 |
| 模糊量词密度高（appropriate/relevant） | `grep -c 'appropriate\|relevant' */skills/**/*.md` | gstack：78 处；graphify：66 处 | 中 |

**命中后的具体行动建议**：
- bureau `.claude/commands/review.md` → 加空输入守护 → 5 分钟
- drama-workshop-skills 的 `write-scene/SKILL.md` → 加 1 个具体的 input→output examples 块 → 15 分钟
- gstack 中 `appropriate` 最密集的 3 个工具文件 → 替换为可验证标准 → 30 分钟

### 7.3 别人的更优方案

1. **领域**：编号前缀表达执行阶段
   - **本案例做法**：`ln-200-scope-decomposer`、`ln-300-task-coordinator`……三位数前缀编码了流水线位置
   - **我的项目现状**：bureau 的 skills 目录是扁平的，`init.md`、`compile.md` 没有明确的执行顺序标注
   - **如何借鉴**：把 bureau skills 改为 `10-init/`、`20-capture/`、`30-compile/`、`40-query/` 四个子目录，README 里注明「按编号顺序执行」

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：每个工件质量
- **我的做法（drama-workshop-skills）**：v1.41.0，21 个 skills 里有 7 个带完整 examples，命令文件全部有 frontmatter
- **本案例（审计时）做法（弱在哪）**：288 个工件，0 个有 examples
- **意义**：规模不代表质量；drama 的 7/21 examples 覆盖率虽然不完美，但远优于本案例的 0/288

---

## 八、术语表

### <a name="Codex适配器"></a>Codex 适配器
> 一个极薄的 SKILL.md 文件，内容只有一行 `MANDATORY READ: <canonical SKILL 路径>`，它本身不含规则，只是把调用转发给真正的规范文件。就像插座转换头——形状变了但电流不变。

### <a name="规范目录"></a>规范目录（Canonical Skill Directory）
> 项目里的「真实来源」——所有规则只在这里写，其他地方通过引用使用。本案例中的 `skills-catalog/` 就是规范目录，已在重构中删除。

### <a name="ln编号系统"></a>ln 编号系统
> 本案例用三位数（ln-200、ln-300……）给技能编号，表示在「交付流水线」中的阶段位置。如 ln-200 是阶段 2 开头（范围分解），ln-301 是阶段 3 第一步（任务创建）。

### <a name="模板变量契约"></a>模板变量契约
> 当 prompt 模板用 `{变量名}` 占位符时，需要有文档说明这些变量从哪里来、谁负责填充。否则调用者传错参数，模板里会出现字面量 `{mode_header}` 而不是实际内容。
