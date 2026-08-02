# kangraemin/claude-inspector — 学习案例

**仓库**：https://github.com/kangraemin/claude-inspector
**Stars**：115 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-08-02（基于当前 HEAD spot-check）
**主题标签**：`single-purpose`, `examples-driven`, `vague-quantifier`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-inspector 是一个专为开发者设计的 [Electron](#electron) 桌面应用，用于检查和分析 Claude Code 的机制检测行为——包括代理流量分析、UI 调试和代码审查。几个关键事实：

- **韩语优先**：主 README 是 `README.ko.md`，英文文档为辅；作者 kangraemin 是韩国开发者
- **定位明确**：它不是通用 AI 工具，而是专门为「调试 Claude Code 本身行为」而生的单一目的工具
- **NL 层极薄**：7 个 NL [工件](#工件)——3 个 [agent](#agent)（代理分析 / UI 调试 / 代码审查）+ 3 个 [skill](#skill)（build / deploy / e2e 测试）+ 1 个 CLAUDE.md
- **双轨质量**：3 个 skill 全部 100/100，3 个 agent 全部 43–55/100，形成鲜明落差

### 1.2 架构剖析

- **目录结构**：
```
claude-inspector/
├── .claude/
│   ├── agents/
│   │   ├── proxy-analyzer.md    # 代理流量分析 agent（43/100）
│   │   ├── ui-debugger.md       # UI 调试 agent（45/100）
│   │   └── reviewer.md          # 代码审查 agent（55/100）
│   ├── settings.json
│   └── skills/
│       ├── build/SKILL.md       # 构建 skill（100/100）
│       ├── deploy/SKILL.md      # 发布 skill（100/100）
│       └── e2e/SKILL.md         # 端到端测试 skill（100/100）
├── main.js                      # Electron 主进程
├── package.json                 # npm 脚本中枢
├── playwright.config.ts
├── public/
├── scripts/
│   └── notarize.js              # macOS 公证脚本（含安全问题）
├── tests/
└── analytics.js
```

- **文件类型分布**：3 个 agent + 3 个 skill + 1 个 CLAUDE.md；无 command 文件；无 hook 脚本（`notarize.js` 是 [electron-builder](#electron-builder) 的 `afterSign` 钩子，不是 Claude Code hook）
- **编排关系**：三个 skill 和三个 agent 完全平列，没有 router 层、没有 meta skill、没有 agent 间调用链——每个工件独立触发
- **跨件契约**：skill 与 `package.json` 之间存在隐式契约——每个 skill 都锚定一个具体的 `npm run <script>`，可随时用 `npm run` 验证；agent 与 `CLAUDE.md` 之间引用了 `proxyDetailView` 结构体，内部一致；但 `reviewer.md` 第 17 行引用的 `~/.claude/rules/review-rules.md` 是用户级路径，不在仓库中，新克隆时会静默失效

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「操作类工件脚本化，分析类工件描述化」——build / deploy / e2e 这三件事有清晰的 npm 脚本对应，作者选择将 skill 写成对脚本的调用包装；而代理分析、UI 调试、代码审查这三件事没有脚本，作者选择写 agent 来描述流程——但描述写得不完整
- **解决什么问题**：让 Claude Code 在 Electron 应用的开发周期中承担具体的辅助角色（构建 / 发布 / 测试 / 调试），而不是通用 AI 助手
- **做了什么 trade-off**：选择写极少量 NL 工件（7 个），而非大而全的 AI 套件；代价是 agent 缺少示例和结构，一旦有 bug 就是灾难性的（见 §3.3）
- **反映什么认知模型**：作者把 skill 理解为「对已有脚本的 NL 包装」——这是正确的；但把 agent 理解为「对工作流程的文字描述」——这是不完整的，缺少了 input→output 契约的关键环节

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「操作锚定型 NL 辅助层」（Script-Grounded NL Wrapper）**

关键特征：
- **特征 1**：每个 skill 都与 `package.json` 中实际存在的 npm 脚本一一对应，可验证、可回退
- **特征 2**：NL 层极薄——只写「能落地到具体命令」的工件，没有抽象的编排层
- **特征 3**：工件数量与项目规模成比例——Electron 单页应用对应 7 个 NL 工件，不扩张
- **特征 4**：skill 和 agent 的分工由工件类型内在决定（操作类用 skill，分析类用 agent），而非刻意设计
- **特征 5**：无 router 层、无 meta skill，每个工件都是叶节点

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有清晰 npm/make 脚本的工程项目 | ✅ 高度适用 | skill 可以直接锚定脚本，验证成本为零 |
| 桌面应用 / 移动应用的构建发布流程 | ✅ 适用 | build/deploy/e2e 三件套是标准化需求 |
| 需要多 agent 协作的复杂分析任务 | ❌ 不适用 | 这个模式缺乏编排层，复杂任务无法分工协调 |
| 没有固定 CLI 命令的纯研究项目 | ❌ 不适用 | 没有脚本锚点时，skill 的「可验证性」优势消失 |
| 需要跨 session 积累经验的长期工程 | ❌ 不适用 | 无经验侧车机制，每次 session 从零开始 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：操作锚定型 NL 辅助层 | kangraemin/claude-inspector | skill 可验证性极高；维护量少 | agent 得不到脚本约束，容易写空 |
| 备选 A：多层验证流水线 | sangrokjung/claude-forge | 质量门显式化，能防止 AI 假完成 | 98 个工件，维护成本数十倍于此 |
| 备选 B：Router + Channels 分层 | 典型的大型 AI 套件 | 支持复杂多分支工作流，扩展性好 | 过度工程，小型项目完全不需要 |

### 2.4 改进空间

1. **当前问题**：3 个 agent 全部缺少 `name:` [frontmatter](#frontmatter)，完全无法注册 **改进做法**：在每个 agent 文件的 `---` 块中添加一行 `name: proxy-analyzer`（类比其他字段） **预期收益**：3 个 agent 立即从「不可用」变为「可用」，修复成本 < 3 分钟

2. **当前问题**：`reviewer.md` 第 17 行引用 `~/.claude/rules/review-rules.md`，是用户级路径，新克隆时静默失效 **改进做法**：将 `review-rules.md` 提交到仓库 `.claude/rules/` 目录下，或在 README 中明确写「需手动创建此文件」的安装步骤 **预期收益**：消除幽灵依赖，让仓库成为自包含的开箱即用工具

3. **当前问题**：proxy-analyzer 和 ui-debugger 没有任何示例块或输出格式声明 **改进做法**：为每个 agent 加一个 `## 示例` section，包含一次具体的输入（用户请求描述）和预期输出格式（JSON / Markdown 表格等） **预期收益**：agent 分数从 43/45 提升至 70+ 区间，Claude Code 调用时行为更稳定

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **76/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude/agents/proxy-analyzer.md` | 43 | 缺 `name` + 无示例 + 无输出格式 |
| `.claude/agents/ui-debugger.md` | 45 | 缺 `name` + 无示例 + 无输出格式 |
| `.claude/agents/reviewer.md` | 55 | 缺 `name` + 无示例（有输出格式，得分略高） |
| `CLAUDE.md` | 88 | 内容精炼、有具体结构体描述，仅扣分于篇幅精简 |
| `.claude/skills/e2e/SKILL.md` | 100 | 无问题 |
| `.claude/skills/build/SKILL.md` | 100 | 无问题 |
| `.claude/skills/deploy/SKILL.md` | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **脚本锚定 skill** → 根本好处：skill 调用的是真实存在的 `npm run dist:mac` / `npm run test:e2e` 等命令，任何人拿到仓库都能立刻验证 skill 的正确性，而不是靠记忆或文档 → 原文件路径：`.claude/skills/build/SKILL.md`、`.claude/skills/deploy/SKILL.md`、`.claude/skills/e2e/SKILL.md` → 如何借鉴：为我的每个 skill 找一个对应的可运行命令，在 skill 正文里明确写出，例如「执行 `make lint` 验证通过后继续」

2. **CLAUDE.md 精炼但具体** → 根本好处：88 分的高分来自内容的具体性——它描述了 `proxyDetailView` 这样的数据结构细节，而非「这是一个 Electron 应用」的空话 → 原文件路径：`CLAUDE.md`（88/100）→ 如何借鉴：CLAUDE.md 不应该写项目介绍文字，应该写「Claude 需要知道的技术事实」，如数据结构、关键路径、约束条件

3. **Skill 与 agent 在工件类型上的自然分工** → 根本好处：build/deploy/e2e 天然是「执行型」工件（有命令可跑），agent 是「分析型」工件（需要推理）；这套分工在认知上非常清晰 → 如何借鉴：设计 NL 辅助层时，先问「这个任务有没有一个可运行的命令来验证？」——有就写 skill，没有就写 agent

4. **Skill 篇幅克制** → 根本好处：每个 skill 只描述自己负责的那一件事（build 就是 build，e2e 就是 e2e），没有跨职责蔓延 → 如何借鉴：单职责原则——如果一个 skill 开始写「顺便也可以做 X」，就应该分拆

### 3.3 当时的缺陷

1. **所有 3 个 agent 缺少 `name` frontmatter 字段** → 为什么会失败：Claude Code 在加载 agent 时通过 `name:` 字段注册和查找 agent；没有这个字段，agent 文件存在于磁盘但永远不会被系统识别——这是「文件存在但工件不存在」的经典陷阱 → **自查**：我的 bureau 项目的 `auditor.md` 和 gstack 的各 agent 文件，有没有遗漏 `name:` 字段？这是最高优先级的检查

2. **reviewer.md 引用了仓库外的 `~/.claude/rules/review-rules.md`** → 为什么会失败：这是一个用户级路径，克隆仓库后不存在；Claude Code 调用 reviewer agent 时会静默跳过这条引用，导致 reviewer 在不知情的情况下缺少核心优先级规则，行为与作者预期不符 → **自查**：我有没有在 skill 或 agent 中引用 `~/.claude/` 开头的路径？这类引用对所有其他用户都是幽灵引用

3. **proxy-analyzer 和 ui-debugger 完全没有示例块** → 为什么会失败：Claude Code 调用 agent 时，示例块是最高优先级的行为约束——没有示例，agent 只能靠自然语言描述推断行为，输出格式和粒度高度不稳定 → **自查**：我的 bureau skill 里是否每个都有 `## Examples` section？缺少示例的 skill 应当列入优先修复队列

4. **所有 3 个 agent 都没有声明 `model:` 字段** → 为什么会失败：没有 model 声明时，Claude Code 使用默认模型；对于 proxy-analyzer 这类重分析任务，作者可能期望使用 Sonnet 级别的模型，但实际可能跑在更便宜的 Haiku 上，导致分析质量不符合预期 → **自查**：我的 agent 中需要复杂推理的（如 bureau 的 auditor），有没有显式指定模型？

5. **`reviewer.md` 中出现模糊量词「합리적」（合理的）** → 为什么会失败：「합리적」没有任何操作定义，Claude Code 无法用它作为可验证的判断标准；不同 session 里的输出会因语境不同而大幅漂移 → **自查**：我的 skill 和 agent 中有没有「合理」「适当」「足够」「相关」这类词？用 `grep -rn -E '合理|适当|足够|相关'` 扫一下

### 3.4 当时的优化机会

1. **三个 agent 各添加一行 `name:` 字段** → 零风险、零逻辑改动，纯 frontmatter 补全，每个文件 30 秒完成；这是最高 ROI 的修复，直接从「不可用」变为「可用」

2. **将 `review-rules.md` 纳入仓库** → 把用户级规则文件提交到 `.claude/rules/review-rules.md`，并在 reviewer.md 中将引用路径改为相对路径；让仓库自包含，任何人克隆即可用

3. **为 proxy-analyzer 和 ui-debugger 补充 `## Output Format` 和 `## 示例` section** → 以 build/SKILL.md（100/100）为参照，添加输入描述、输出格式（JSON / Markdown）、至少一个完整调用示例；预计将评分从 43/45 提升至 75+ 区间

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
（基于 2026-08-02 的 Phase 3 spot-check，对当前 HEAD 的直接检查）

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个 agent 缺 `name:` 字段 | 读取 proxy-analyzer.md / reviewer.md / ui-debugger.md 的 frontmatter | **持续存在**：三者的 frontmatter 只有 `description:`，没有 `name:` | 所有 3 个 agent 在当前版本仍不可用；这个 bug 已存在超过 3.5 个月未被修复 |
| reviewer.md 引用 `~/.claude/rules/review-rules.md` | 检查 reviewer.md 第 17 行 | **持续存在**：引用路径未变 | 幽灵依赖持续存在，reviewer agent 的优先级规则对所有新克隆者不可用 |
| `scripts/notarize.js` 密码明文传参 | 检查 notarize.js 的 xcrun notarytool 调用 | **持续存在**：`--apple-id`, `--password`, `--team-id` 模式未改变 | APPLE_APP_SPECIFIC_PASSWORD 在 macOS `ps` 输出中对本机所有用户可见，安全风险未解除 |

### 4.2 架构演进
基于 spot-check 结果，当前 HEAD 的目录结构与 2026-04-19 audit 时完全一致——3 个 agent、3 个 skill、`scripts/notarize.js` 均未发生变动。没有新增工件，没有删除工件，也没有重组。

这告诉我们两件事：
- **正面**：架构设计本身是稳定的，没有因为补丁冲动而引入新的混乱
- **负面**：3 个已知 bug（缺 `name:` 字段）在 3.5 个月内没有被修复，说明作者目前对这个仓库的 NL 层维护优先级较低

### 4.3 新增的可学习模式
暂无——当前 HEAD 与 audit 快照在 NL 层面完全一致，没有发现新的设计模式或改进迹象。

---

## 五、校准

### 5.1 我已经在做对的

1. **Examples 驱动**：bureau 的 skill 文件中有 `## Examples` section，这正是 kangraemin 的 agent 所缺乏的；这种做法使输出格式稳定，不依赖 Claude 自由发挥
2. **skill 覆盖实际操作**：我的 gstack 和 bureau 项目的 skill 都是围绕具体操作而非抽象概念设计的，这与 kangraemin 的 build/deploy/e2e skill 的思路一致
3. **避免模糊量词**：我在 skill 编写时有意避免「合理」「适当」这类词，与 kangraemin reviewer.md 中「합리적」的反例形成对比
4. **CLAUDE.md 精炼具体**：我的 CLAUDE.md 倾向于写技术约束而非项目介绍，与 kangraemin 的 88/100 CLAUDE.md 的得分路径一致
5. **工件数量与项目规模匹配**：我没有为小项目堆砌大量 NL 工件，这与 kangraemin 的克制策略一致

### 5.2 挑战 / 验证

**挑战的假设**：我之前以为「skill 和 agent 只要文件存在就会被 Claude Code 加载」。这个案例清晰说明：缺少 `name:` 字段时，文件在磁盘上存在，但对 Claude Code 来说这个工件不存在。3 个 agent 因为这一个字段的缺失，从「可用但质量差」变成了「完全不可用」——这是质量评分系统本身无法完全捕获的灾难性 bug（43/55 的得分掩盖了 0/100 的实际可用性）。

**验证的做法**：`name:` 字段是 frontmatter 中最不可缺少的字段，应该成为我写任何新 agent/skill 时的第一条检查项，而不是事后审查项。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent 是否缺少 name: 字段（最高优先级）
grep -rL "^name:" ~/.claude/agents/*.md 2>/dev/null
# 命中后：立即在该文件的 frontmatter 顶部添加 name: <agent-name> 一行

# 检查我的 skill 是否缺少 name: 字段
grep -rL "^name:" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：同上，添加 name: 字段

# 检查是否引用了仓库外的用户级路径（幽灵引用检测）
grep -rn "~/.claude/" ~/.claude/agents/*.md ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：将引用的文件提交到仓库对应相对路径，或在 README 中说明手动安装步骤

# 检查 agent 和 skill 中是否有模糊量词
grep -rn -E '合理|适当|足够|相关|reasonable|appropriate|sufficient' ~/.claude/agents/*.md ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：将模糊量词替换为可验证标准，如「≥3 个具体例子」「运行 npm test 通过」

# 检查 agent 是否有示例 section
grep -rL "## 示例\|## Example\|## 例子" ~/.claude/agents/*.md 2>/dev/null
# 命中后：参照 build/SKILL.md 的结构，添加至少一个完整的 input→output 示例
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 和 gstack 的每个 skill 里显式引用对应的可运行命令（「脚本锚定」模式）
   - **为何可行**：bureau 有 compile、capture 等操作都对应具体的命令；gstack 的 23 个工具中有相当一部分是对具体 CLI 命令的包装；锚定后 skill 的行为可验证
   - **第一步**：打开 bureau 的 `skills/compile/SKILL.md`，在正文中加一行「验证命令：`npm run compile` 或 `python3 scripts/compile.py`」，5 分钟完成

2. **想法**：建立一个 NL 工件「上线前检查清单」脚本，在新建 agent/skill 后自动检测 frontmatter 完整性
   - **为何可行**：kangraemin 的案例证明 `name:` 字段缺失是高频低成本 bug，完全可以用脚本检测；在提交前跑一次检查，3.5 个月的 bug 本可以在 30 秒内发现
   - **第一步**：在项目 scripts 目录写一个 20 行的 bash 脚本：`grep -rL "^name:" .claude/agents/*.md .claude/skills/*/SKILL.md`，如果有命中就以非零状态退出；接入 pre-commit hook，15 分钟完成

3. **想法**：为 bureau 的 auditor agent（分析类工件）补充输出格式声明和示例块
   - **为何可行**：bureau 的 auditor 与 kangraemin 的 reviewer 类型相近（都是分析型 agent）；kangraemin reviewer 的 55/100 得分主要就是因为有输出格式而缺示例——我的 auditor 可以做到更完整
   - **第一步**：打开 bureau 的 auditor agent，添加 `## Output Format` section（JSON schema 或 Markdown 表格）和 `## 示例` section（一次虚构但完整的调用示例），30 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`（2026-07-27 刷新，含 60 天内有 push 且有 NL 工件的公开仓库）

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **kangraemin/claude-inspector 的核心目的**：Electron 桌面开发工具，NL 层服务于开发流程（构建 / 发布 / 调试），而非通用 AI 工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 低 | 同为以「检查 / 审计」为核心任务的 Claude Code 插件 | bureau 是知识管理插件，claude-inspector 是 Electron 桌面应用；实现路径完全不同 | 中（agent 写法可借鉴） |
| MarkQWu/gstack | 低 | 同为多工件 Claude Code 套件 | gstack 覆盖角色模拟（CEO / 设计师 / EM），claude-inspector 覆盖 Electron 开发流程 | 低 |
| MarkQWu/- | 无 | — | 情报仪表盘，与 Electron 开发工具无交集 | 低 |

我的仓库中无目的相近的项目，本节主要做技术模式对照（尤其是 agent 写法）。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 缺 `name:` 字段 | `grep -rL "^name:" ~/.claude/agents/*.md` | 需手动检查 bureau 的 auditor.md 和 gstack 的所有 agent 文件 | 高（若命中则工件完全不可用） |
| 引用仓库外的 `~/.claude/` 路径 | `grep -rn "~/.claude/" ~/.claude/agents/*.md ~/.claude/skills/*/SKILL.md` | 需检查 bureau 和 gstack 的 skill 文件是否有此类路径 | 高（幽灵引用对所有用户静默失效） |
| agent 无示例块 | `grep -rL "## 示例\|## Example" ~/.claude/agents/*.md` | bureau 的 auditor agent 可能受影响 | 中（不影响注册，但影响输出稳定性） |
| 模糊量词 | `grep -rn -E '合理\|适当\|sufficient\|reasonable' ~/.claude/agents/*.md` | gstack 的角色 agent 中若有此类表述则需修正 | 低（影响输出一致性而非可用性） |

**命中后的具体行动建议**：
- bureau 的 `auditor.md` → 检查 frontmatter 是否有 `name:` 字段，没有则添加 `name: auditor`，2 分钟完成
- gstack 的所有 agent 文件 → 批量检查 `name:` 字段缺失，每个添加 1 分钟
- bureau 和 gstack 的 skill 文件 → 扫描 `~/.claude/` 路径引用，改为仓库相对路径，每处 5 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：skill 与可运行命令的显式绑定（脚本锚定）
   - **kangraemin 做法**：`.claude/skills/build/SKILL.md` 直接调用 `npm run dist:mac`；`.claude/skills/e2e/SKILL.md` 调用 `npm run test:e2e`——每个 skill 都有一个 `package.json` 中真实存在的 npm 命令作为验证锚点
   - **我的项目现状**：gstack 的 skill 描述的是角色行为（CEO 如何决策），操作锚点不如 kangraemin 明确；bureau 的 skill 有一定操作性但也可以更精确
   - **如何借鉴**：在 gstack 的每个 skill 末尾加「验证命令」一行（即使是简单的 `echo "Done"` 也比没有好）；bureau 的 compile skill 应直接写出 `python3 scripts/compile.py` 或等价命令

2. **领域**：CLAUDE.md 技术细节而非项目简介
   - **kangraemin 做法**：CLAUDE.md 描述了 `proxyDetailView` 这样的内部数据结构，是 Claude Code 实际需要知道的技术信息，而非「这是什么项目」的概述
   - **我的项目现状**：需要检查我的 CLAUDE.md 是否过于描述项目而非描述「Claude 需要知道的约束」
   - **如何借鉴**：审查 bureau 和 gstack 的 CLAUDE.md，删除所有「这个项目是一个……」式的描述句，改为「key 数据结构 X 的格式是……」「不允许直接操作 Y 文件，必须通过 Z 命令」这类技术约束

### 7.4 反向：我的项目做得比他们好的地方

1. **领域**：agent 示例块的完整性
   - **我的做法**：bureau 的 skill 和 agent 文件有 `## Examples` section，包含完整的 input→output 对
   - **kangraemin 做法**（弱在哪）：proxy-analyzer 和 ui-debugger 完全没有示例块，reviewer 有输出格式但无调用示例——这是 43/45/55 分的主要原因
   - **意义**：如果审到我的 bureau，示例块是明确的亮点；若考虑给类似仓库提 PR，补充示例块是最高 ROI 的贡献点

---

## 八、术语表

### <a name="工件"></a>工件
> 在 NLPM 语境中，指 `.claude/commands/`、`.claude/agents/`、`skills/` 等目录下的 Markdown 文件。这些文件是 Claude Code 能读取并执行的「指令模板」，是整个 NL 编程系统的基本单元。没有工件，Claude Code 不知道有哪些特定任务可以执行。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包裹的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读取 agent 和 skill 时先解析 frontmatter 才能知道这个工件叫什么、怎么注册。**缺少 `name:` 字段 = 文件存在但工件不存在**，是本案例最严重的 bug 根源。

### <a name="agent"></a>agent
> Claude Code 中的「专职助手」——一段带有 frontmatter 的 Markdown 文件，描述某类任务的处理流程。与 skill 的区别：agent 通常有独立的工具权限和执行逻辑，可以被 Claude Code 以任务模式调用；skill 更像参考知识。

### <a name="skill"></a>skill
> Claude Code 中的「参考知识包」——一段描述某种能力或操作步骤的 Markdown 文件。与 agent 的区别：skill 通常描述「怎么做」，不直接执行，而是被 agent 或 command 引用。本案例的 build/deploy/e2e 三个 skill 各对应一个具体的 npm 脚本。

### <a name="electron"></a>Electron
> 一个用 JavaScript / HTML / CSS 构建跨平台桌面应用的框架——本质上是把 Chromium（浏览器内核）和 Node.js 打包进一个桌面应用。Claude Desktop 本身也是 Electron 应用。claude-inspector 用 Electron 做 UI，用 main.js 作为主进程控制代理流量分析。

### <a name="electron-builder"></a>electron-builder
> 一个将 Electron 应用打包成可分发安装包（`.dmg`、`.exe`、`.AppImage` 等）的构建工具。`package.json` 中的 `afterSign` 钩子在打包完成后自动触发，用于 macOS 代码公证等签名流程——这是 `notarize.js` 被调用的入口。

### <a name="npm-scripts"></a>npm scripts
> `package.json` 文件里 `"scripts"` 字段下定义的命令快捷方式。例如 `"dist:mac": "electron-builder --mac"` 可以通过 `npm run dist:mac` 来调用。本案例的 3 个 skill 都锚定了 `package.json` 中真实存在的 npm script，这是它们能得 100 分的根本原因。

### <a name="sentry"></a>Sentry
> 一个云端错误监控和性能追踪服务（`sentry.io`）。应用集成 `@sentry/electron` 后，运行时发生的崩溃和异常会自动上报到 Sentry 服务器，包括设备信息、堆栈追踪等。本案例的 Medium 安全问题在于：这一数据收集行为对最终用户没有明确披露。

### <a name="semver"></a>semver
> 语义化版本号（Semantic Versioning）。`^1.2.3` 表示「接受 1.x.x 范围内的任意版本」。本案例的 Low 安全问题：所有依赖都用 `^` 范围，意味着每次 `npm install` 可能安装不同版本；若某个补丁版本被植入恶意代码（供应链攻击），会自动传播进构建产物。
