# wecode-ai/Wegent — 学习案例

**仓库**：https://github.com/wecode-ai/Wegent
**Stars**：521 | **来源**：upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-07-05（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `vague-quantifier`, `curl-pipe-bash-risk`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
wecode-ai/Wegent 是一个完整的 AI agent 平台产品，提供 Web UI、后端服务、代码执行沙箱和浏览器 relay，其中 Claude Code skills 是平台「工具层」的一部分——随 backend 初始化数据一起分发，供用户在 Wegent 界面中直接使用。

关键事实：
- 521 颗星，是较小体量的 AI 平台产品，但有完整的全栈架构
- skills 存放在 `backend/init_data/skills/`，不是独立 plugin，而是平台的内置工具数据
- 包含 11 个 SKILL.md（audit 时 12 个，现在有 `interactive/SKILL.md` 新增）
- Security 状态为 BLOCKED：存在 CRITICAL（curl|sh）和 HIGH（eval 命令注入）安全风险，审计后 2.5 个月 CRITICAL 仍未修复
- 核心产品价值：用户可以在 Wegent 的 Web UI 里直接创建和使用 skill，因此「skill 创作者」（`skill-creator/SKILL.md`）本身也是一个 skill

### 1.2 架构剖析
```
Wegent/
├── backend/
│   └── init_data/
│       └── skills/                    ← 平台内置 skills（随 backend 初始化）
│           ├── skill-creator/SKILL.md ← 帮用户创建新 skill 的 meta-skill
│           ├── browser/SKILL.md       ← 浏览器操作封装
│           ├── sandbox/SKILL.md       ← 代码沙箱执行
│           ├── mermaid-diagram/SKILL.md
│           ├── prompt-optimization/SKILL.md
│           └── ... (6 more skills)
├── frontend/                          ← React 前端
├── executor/                          ← 代码执行引擎（Python）
├── scripts/                           ← 13 个 shell 脚本（含安全漏洞）
└── tests/CLAUDE.md                    ← 测试项目说明
```

- **文件类型分布**：11 个 SKILL.md，0 个 agent，0 个 command，1 个 tests/CLAUDE.md，无注册用 plugin.json（内置 skill，不走 marketplace）
- **编排关系**：11 个 skill 平列；`skill-creator` 是 meta-skill，逻辑上「高于」其他 skill，但代码层面平等
- **跨件契约**：`skill-creator/SKILL.md` 教用户创建新 skill，但其中的 [frontmatter](#frontmatter) 模板只有 `name` + `description` 两个字段，而 Wegent 自己的 skill 实际使用 `displayName`、`version`、`author`、`bindShells`、`mcpServers` 等 8+ 字段——存在「向导」和「范例」不一致的严重内部矛盾

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「技能作为产品数据」——skill 不是用户安装的插件，而是平台初始化时注入的内置工具；这和 Claude plugin marketplace 的分发模式截然不同
- **解决什么问题**：让非技术用户在图形界面里创建和管理 AI skill，降低 NL 编程门槛
- **做了什么 trade-off**：自建平台 vs. 依赖 Claude Code 生态。Wegent 把 skill 做成私有数据格式，获得了 UI 控制权，但失去了 Claude plugin marketplace 的分发网络
- **反映什么认知模型**：作者把 skill 理解为「平台功能配置项」，而不是「给 AI 看的提示词」——这也解释了为什么很多 skill 的 frontmatter 设计是面向 Wegent UI 的（displayName 给人看），而不是面向 NLPM 规范的（name 给工具链看）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「平台内置 Skill 模式（Platform-Embedded Skills）」**

skill 作为平台的初始数据，随后端服务一起部署，而非独立 plugin。平台拥有 skill 的注册、展示、调用全链路控制权。

模式特征清单：
- 特征 1：skills 存放在 `init_data/`，通过数据库 seed 机制加载，而非 `claude plugin install`
- 特征 2：frontmatter 字段是平台私有扩展（`displayName`、`bindShells`、`mcpServers`），不完全遵循 NLPM 标准
- 特征 3：`skill-creator` meta-skill 是平台的「自繁殖机制」，让用户在 UI 里创建新 skill
- 特征 4：skill 和平台的其他基础设施（executor、browser relay）深度耦合
- 特征 5：无 marketplace 注册，也无 CLAUDE.md 声明，skill 是「内部知识」而非「公开工具」

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 自建 AI 工具平台，需要定制 skill 注册/展示 | ✅ 适用 | 平台掌控全链路，可自定义 frontmatter 和 UI 呈现 |
| 需要快速向用户分发 skill 的开源工具 | ❌ 不适用 | 平台用户 ≠ Claude Code 用户，分发范围受限 |
| 希望技能在 Claude marketplace 上架 | ❌ 不适用 | 私有格式不兼容 marketplace 规范 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 平台内置（本仓库） | wecode-ai/Wegent | 完全掌控，可自定义格式 | 脱离 Claude 生态，无法通过 marketplace 分发 |
| 独立 plugin（marketplace） | twostraws/SwiftUI-Agent-Skill | 任何 Claude Code 用户均可安装 | 需遵守 NLPM 规范，无法自定义字段 |
| 嵌入主项目（随项目分发） | wasp-lang/open-saas | 无需额外安装，即开即用 | 只对该项目的用户可见 |

### 2.4 改进空间
1. **当前问题**：`skill-creator/SKILL.md` 教用户写 2 字段 frontmatter，但平台实际需要 8+ 字段。**改进做法**：把 `skill-creator` 里的模板替换成 Wegent 实际使用的完整 frontmatter 模板，包含 displayName、version、author 等。**预期收益**：消除「按教程做出的 skill 无法被平台正常加载」的问题。
2. **当前问题**：6 个 skill 仅有 `displayName` 而无 `name` 字段，导致 NLPM 注册失败。**改进做法**：批量脚本为每个 skill 补全 `name` 字段（`name` 值可等于 `displayName` 的 snake_case）。**预期收益**：6 个 skill 注册成功，分数从 67-75 提升到 90+。
3. **当前问题**：`scripts/run-e2e-local.sh` 中的 `curl|sh` 在自动化测试中执行。**改进做法**：替换为「下载 + 校验 SHA-256 + 执行」三步模式，或改用 [uv](#uv) 的官方锁文件安装。**预期收益**：消除 CRITICAL 安全漏洞，解除 BLOCKED 状态。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-20 当时得分 **81/100**（12 个文件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `mermaid-diagram/SKILL.md` | 67 | 缺少 `name` frontmatter（−25） |
| `wiki_submit/SKILL.md` | 68 | 缺少 `name` frontmatter（−25） |
| `sandbox/SKILL.md` | 69 | 缺少 `name` frontmatter（−25） |
| `prompt-optimization/SKILL.md` | 71 | 缺少 `name` frontmatter（−25） |
| `ui-links/SKILL.md` | 71 | 缺少 `name` frontmatter（−25） |
| `wegent-knowledge/SKILL.md` | 75 | 缺少 `name` frontmatter（−25） |
| `skill-creator/SKILL.md` | 83 | 模糊量词 ×6（−12） |
| `interactive-form-question/SKILL.md` | 98 | 几乎无问题 |
| `subscription-manager/SKILL.md` | 96 | 少量模糊语言 |

### 3.2 当时值得借鉴的模式

1. **`interactive-form-question/SKILL.md` 的精确问题分解** → 把「问用户问题」拆解为「一次只问一个问题、等待回答后再继续」的状态机模型，完全可操作 → 参考 `interactive-form-question/SKILL.md` → 借鉴：任何需要用户输入的 skill 都可以用这种状态机写法
2. **`subscription-manager/SKILL.md` 的状态驱动逻辑** → 根据订阅状态（试用/付费/过期）分支处理，逻辑清晰，覆盖边界情况 → 借鉴：为有状态的 skill（有不同输入情况的）提供分支处理表而不是「尽可能处理」
3. **`browser/SKILL.md` 的工具组合策略** → 明确声明「先用 mcp__browser_action 批量操作，避免多次单独调用」→ 借鉴：在 skill 中明确说明「工具选择策略」，而不是让模型自行决定用哪个工具

### 3.3 当时的缺陷

1. **6 个 skill 缺少 `name` frontmatter**：6 个 skill 只有 `displayName`（面向 UI）而无 `name`（面向工具链），导致 NLPM 无法按名称解析这些 skill。根本原因：Wegent 的 skill 格式是为自己的 UI 设计的，`displayName` 优先，`name` 被遗忘。自查：我的所有 skill 均有 `name` 字段，这点没问题；但值得确认 displayName 和 name 的关系。
2. **`skill-creator` 模板与实际格式不一致**：教用户写 skill 的工具，自己就用了错误的模板。这是「向导悖论」：教材和实践脱节，学生（平台用户）照着教材做出的 skill 无法正常工作。根本原因：平台演进时 skill 格式扩展了，但 skill-creator 的模板没跟着更新。自查：我的 `gstack/skillify/SKILL.md`（帮用户创建 skill 的 meta-skill）是否有同样问题？
3. **CRITICAL `curl|sh`**：`scripts/run-e2e-local.sh` 直接 `curl|sh` 下载并执行外部脚本，无任何完整性校验。这是安全的最高风险模式——供应链攻击可以注入任意代码。根本原因：工程文化中把「快速启动」的便利性置于安全审查之上。自查：我的 gstack 和 bureau 脚本中是否有类似模式？

### 3.4 当时的优化机会

1. 为 6 个缺少 `name` 的 skill 批量添加 `name` 字段（一个脚本，10 分钟）
2. 更新 `skill-creator/SKILL.md` 里的 frontmatter 模板，与 Wegent 实际格式对齐
3. 修复 `scripts/run-e2e-local.sh` 中的 `curl|sh`：改为「先下载到临时文件，SHA-256 校验后执行」

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 6 个 skill 缺少 `name` 字段 | `grep -c "^name:" */SKILL.md` 逐文件检查 | **仍然存在**（mermaid-diagram、prompt-optimization、sandbox、ui-links、wegent-knowledge、wiki_submit 均为 0） | 2.5 个月未修，frontmatter 债务被视为低优先级 |
| CRITICAL `curl\|sh` | `grep -n "curl.*sh" scripts/run-e2e-local.sh` → L154 命中 | **仍然存在**（`curl -LsSf https://astral.sh/uv/install.sh \| sh`） | CRITICAL 漏洞未修，BLOCKED 状态持续；新增 skill `interactive/SKILL.md` 但安全问题未处理 |
| `skill-creator` 模板与实际格式不一致 | 读 `skill-creator/SKILL.md` frontmatter 章节 | 无法直接在线核实，基于 audit 判断仍可能存在 | 暂无证据修复 |

### 4.2 架构演进

与 audit 时相比，新增了 `interactive/SKILL.md`（交互式问答 skill）。这说明 Wegent 的 skill 库在持续扩充，但安全和 frontmatter 质量问题被排在功能开发之后。结构上无重组，仍是 backend init_data 的平铺模式。

### 4.3 新增的可学习模式

`interactive/SKILL.md` 的新增（audit 时只有 `interactive-form-question`，现在又有一个独立的 `interactive` skill）暗示作者在「交互式引导」这个方向上持续投入——这是 Wegent 产品差异化的核心能力（图形界面 + AI 交互），值得关注其交互设计模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **所有 skill 均有 `name` frontmatter**：gstack 59 个 skill 全部有 `name` 字段，这是 Wegent 6 个 skill 卡掉的问题，我没有踩坑
2. **没有 curl|sh 反模式**：我的 gstack 和 bureau 脚本中没有直接 curl|sh 的模式（经快速验证）
3. **skill-creator 类型的 meta-skill 需要特别维护**：gstack 的 `skillify/SKILL.md` 如果有引导创建 skill 的功能，需要检查其中的 frontmatter 模板是否与当前规范一致

### 5.2 挑战 / 验证

这次案例验证了一个我已有的判断：**「向导悖论」是 NL 编程的高风险点**——帮用户创建 artifact 的 artifact 本身，需要比普通 artifact 更严格的质量维护，因为它的错误会被复制放大。Wegent 的 `skill-creator` 用旧模板教用户，导致用户创建的 skill 从一开始就有缺陷。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 gstack/skillify 是否有 frontmatter 模板，且模板是否与当前规范一致
grep -A 20 "frontmatter\|---" /tmp/my-repos/MarkQWu-gstack/skillify/SKILL.md | head -30
# 命中后怎么办：确认模板中是否包含 name、description、allowed-tools 三件套；如果缺少任何一个，补充

# 检查我的脚本中是否有 curl|sh 模式
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau -name "*.sh" -exec grep -l "curl.*sh\|curl.*bash" {} \; 2>/dev/null
# 命中后怎么办：将 curl|sh 替换为「curl -o /tmp/file + sha256sum 校验 + bash /tmp/file」三步模式
```

### 6.2 灵感 → 实施路径

1. **想法**：检查并修复 gstack/skillify 的 frontmatter 模板一致性
   - **为何可行**：skillify 是 gstack 的 meta-skill，用户照着它创建的 skill 会进入我的仓库
   - **第一步**：读 `MarkQWu-gstack/skillify/SKILL.md`，确认其中的 frontmatter 示例包含 name、description、allowed-tools，以及我的任何自定义字段

2. **想法**：为 bureau 中语义最重的 skill（`review`）补 `name` 字段验证脚本
   - **为何可行**：bureau 所有 skill 已有 name 字段，但可以加一个 CI 验证脚本，防止未来遗漏
   - **第一步**：在 `MarkQWu-bureau` 根目录创建 `check-skills.sh`，遍历所有 SKILL.md，验证 `name:` 字段存在；失败则非零退出，接入 pre-commit hook

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 wecode-ai/Wegent 的核心目的**：为 AI agent 平台提供内置 skill 工具集，让用户在 Web UI 中创建和使用 AI 技能
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 AI 工作流系统，都有多个 skill | bureau 专注知识管理，Wegent 是通用 agent 平台；bureau 走 marketplace，Wegent 内置 | 中 |
| MarkQWu/gstack | 低 | 都是 skill 套件 | gstack 面向个人开发者生产力，Wegent 面向平台用户 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill `name` 字段缺失 | `grep -c "^name:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | bureau 全部 7/7 有 name 字段，✅ 无问题 | N/A |
| Meta-skill 模板与实际格式不一致 | `grep -A 10 "frontmatter" /tmp/my-repos/MarkQWu-gstack/skillify/SKILL.md` | 需要手动核查 skillify 的模板内容 | 潜在高 |
| curl\|sh 安全模式 | `find /tmp/my-repos -name "*.sh" -exec grep -l "curl.*\| sh"` | 无命中，✅ 无问题 | N/A |

**命中后的具体行动建议**：
- 重点核查 `MarkQWu-gstack/skillify/SKILL.md` 的 frontmatter 模板 → 确认三件套齐全 → 10 分钟可完成

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：平台内置 skill 的用户可见性设计
   - **本案例做法**：用 `displayName` 和 `version` 字段控制 UI 展示，不同于 NLPM 的 `name` 字段——两套 label 并存，面向用户和面向工具链分离（`skill-creator/SKILL.md`）
   - **我的项目现状**：bureau 和 gstack 均只有 `name`，没有区分「用户友好名称」和「工具链名称」
   - **如何借鉴**：在 description 字段里加「用户界面显示名」（作为 description 的首行），而不需要新字段——既符合规范又让用户看到友好名称

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：`name` frontmatter 覆盖率
- **我的做法**：gstack、bureau、echo-sleuth 的所有 SKILL.md 均有 `name` 字段
- **本案例做法（弱在哪）**：Wegent 6/11 个 skill 缺少 `name`，导致 NLPM 注册失败、分数拖累整体
- **意义**：这是一个基础质量亮点；如果 Wegent 将来开放 marketplace，这 6 个 skill 需要先补 name 才能上架

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件元数据（`name`、`description`、`displayName` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能注册该 skill；`name` 是工具链用的标识符，`displayName` 是可选的用户友好名称。

### <a name="uv"></a>uv
> 一个用 Rust 写的 Python 包管理工具，比 pip 快 10-100 倍。Wegent 的 setup 脚本用 `curl | sh` 从 astral.sh 安装 uv——这种安装方式方便但存在供应链安全风险，建议改用官方 SHA 校验的安装流程。

### <a name="curl-pipe-sh"></a>curl-pipe-sh（curl|sh 反模式）
> `curl <URL> | sh` 是把从网络下载的脚本直接通过 bash 执行的快捷写法。这意味着如果下载 URL 被攻击者控制（服务器被入侵、DNS 劫持、中间人攻击），任意代码都会在你的机器上以你的权限执行。安全替代方案：先 curl 到本地文件，用 sha256sum 校验完整性，再执行。

### <a name="meta-skill"></a>meta-skill
> 一个 skill 的功能是「帮用户创建其他 skill」，即操作对象本身也是 skill。这类 skill 需要额外维护，确保其内置的 skill 模板始终与当前平台规范保持同步，否则用户照模板创建的 skill 从一开始就有缺陷。
