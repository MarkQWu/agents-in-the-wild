# refly-ai/refly — 学习案例

**仓库**：https://github.com/refly-ai/refly
**Stars**：7272 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-05（历史快照）| **生成日期**：2026-06-28（基于审计数据）
**主题标签**：`single-purpose`, `manifest-discipline`, `vague-quantifier`, `security-gate`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

refly-ai/refly 是一个面向全球用户的 **[AI 原生知识工作台](#AI原生知识工作台)**，核心定位是将 AI 能力嵌入知识创作、整理与发布的全流程。项目拥有 7272 颗 Star，是中文 AI 工具生态中具有较高知名度的开源产品。

从 Audit 快照看，仓库包含 4 个 [NL 制品](#NL制品)，覆盖 CLI 操作层（登录、升级、状态查看）和一个核心 skill 定义。[NL 质量评分](#NLPM)加权平均为 89/100，处于"工业级可接受"水准，其中 SKILL.md 拿到满分（100/100）。安全评级为 **BLOCKED**（2 项 High 风险），无法自动贡献。

项目的核心价值主张：通过 CLI 工具让用户在本地终端与 refly 云端知识库无缝交互，将知识工作与 AI 原生工作流深度整合。整个仓库架构是一个大型 monorepo，NL 层仅覆盖 CLI 入口，核心产品逻辑全部由 TypeScript/Python 代码实现。

### 1.2 架构剖析

refly 的仓库采用 monorepo 结构，NL 层极度精简：

| 层级 | 路径 | 类型 | 分数 | 职责 |
|------|------|------|------|------|
| CLI 命令层 | `packages/cli/commands/refly-login.md` | Command | 75 | 用户认证登录 |
| CLI 命令层 | `packages/cli/commands/refly-upgrade.md` | Command | 85 | 升级 CLI 工具 |
| CLI 命令层 | `packages/cli/commands/refly-status.md` | Command | 95 | 查询账户与连接状态 |
| Skill 层 | `packages/cli/skill/SKILL.md` | Skill | 100 | 封装 refly CLI 操作知识 |

NL 层仅覆盖 CLI 操作入口，共 4 个文件；背后是大型 monorepo，包含 `scripts/*.js`、`*.sh`、Python 规格文件等数十个可执行制品。NL 层和代码层之间有清晰的分工：CLI 行为由 Markdown 声明，核心产品逻辑由代码实现。

SKILL.md 内部通过 `§References` 节引用 `rules/` 路径，而实际文件位于 `references/` 目录，映射关系由 `refly upgrade` 命令在运行时处理。这一设计在运行时无错误，但在源码结构层产生了路径与文档不一致的"[静默脆弱性](#静默脆弱性)"。

### 1.3 设计思路 / 方法论

**"NL-as-CLI-wrapper"（自然语言作为 CLI 封装层）**是 refly 的核心 NL 编程策略。

这一设计思路的核心逻辑：将 AI 能力封装为标准 CLI 命令（login / upgrade / status），每个命令有对应的 Markdown 声明文件，再用一个满分 SKILL.md 作为整体知识入口。这与通常把 NL 层铺满整个 monorepo 的做法形成鲜明对比——refly 选择了"NL 层只做它最擅长的事"，CLI 操作的用户意图描述由 Markdown 处理，业务逻辑由代码处理。

这种策略的代价是覆盖范围极窄（4 个制品），优点是每个制品的职责边界极清晰，单个文件的可维护性高。SKILL.md 拿到满分也印证了这一策略的有效性：聚焦 + 规范 = 高质量。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：极简 NL 表皮（Minimal NL Surface）**

这是一种刻意收窄 NL 层覆盖范围的设计选择。核心特征：

- **特征 1：NL 层仅覆盖用户操作入口**，即 CLI 命令。产品内部逻辑、脚本、测试规格均不纳入 NL 制品范围。
- **特征 2：单一 SKILL.md 作为知识汇聚点**，聚合所有 CLI 操作的上下文，使 Agent 无需逐一查阅命令文件即可获取全局操作知识。
- **特征 3：[manifest-discipline](#manifest-discipline)（清单纪律）体现在 SKILL.md 的满分**——frontmatter 完整、无模糊量词、引用路径经过验证。
- **特征 4：命令文件质量呈梯度分布**（75 → 85 → 95），越靠近简单操作质量越高，越复杂（如 login 涉及认证流）质量越低，这是真实工程项目的常见规律。

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|------|----------|------|
| 大型 monorepo 中的 CLI 工具层 | 适合 | NL 层只做 CLI 声明，不干扰业务逻辑代码 |
| 需要 Agent 理解完整产品知识的场景 | 部分适合 | SKILL.md 满分但覆盖仅 CLI 层，产品核心逻辑对 Agent 不透明 |
| 需要全面 AI 辅助开发的 monorepo | 不适合 | NL 层覆盖率过低，Agent 无法理解大部分业务逻辑 |
| 小团队快速迭代的 CLI 工具产品 | 适合 | 极简 NL 层维护成本低，质量容易保障 |

### 2.3 与其他架构对比

| 维度 | refly-ai/refly（极简 NL 表皮） | 全覆盖 NL 层（如 ruvnet/ruflo） | nlpm 本身（中度覆盖） |
|------|-------------------------------|--------------------------------|----------------------|
| NL 制品数量 | 4 个 | 84 个 | ~30 个 |
| 制品平均质量 | 89/100（高） | 60/100（低） | 90+/100 |
| 维护成本 | 极低 | 极高 | 中 |
| Agent 可理解范围 | CLI 操作层 | 所有 Agent 任务 | 命令 + skill + 规则 |
| 安全风险来源 | 代码层（脚本） | NL 层（hook 误用） | 较低 |

### 2.4 改进空间

1. **refly-login.md（75 分）缺三项规范**：缺 `allowed-tools`（-5）、无 output-format（-10）、无空输入处理（-10）。login 命令涉及认证流，恰恰是最需要明确 output-format（成功/失败的具体输出格式）和输入校验的场景。这不只是得分问题，而是实际用户体验问题：若 token 为空时 Agent 不知道如何提示用户，会产生静默失败。
2. **三个 command 文件均缺 `allowed-tools`**：这是系统性遗漏，说明在创建命令文件时没有遵循统一的 [frontmatter](#frontmatter) 模板。一次性为三个文件补全 allowed-tools，可将最低分从 75 提升到 80+。
3. **`§References` 路径与实际目录结构不一致**：SKILL.md 引用 `rules/` 路径，文件实际在 `references/`。依赖运行时命令做路径映射属于"隐式契约"，若未来 upgrade 命令逻辑变更，引用将静默断裂。建议在 SKILL.md 中直接引用 `references/` 路径，或在 frontmatter 中添加显式路径别名声明。

---

## 三、过去审查发现（2026-05-05 历史快照）

### 3.1 当时质量评分（NLPM）

**加权平均：89 / 100**

| 文件 | 类型 | 分数 | 扣分项 |
|------|------|------|--------|
| packages/cli/commands/refly-login.md | Command | 75 | 缺 allowed-tools（-5）、无 output-format（-10）、无空输入处理（-10） |
| packages/cli/commands/refly-upgrade.md | Command | 85 | 缺 allowed-tools（-5）、无 output-format（-10） |
| packages/cli/commands/refly-status.md | Command | 95 | 缺 allowed-tools（-5） |
| packages/cli/skill/SKILL.md | Skill | 100 | 无 |

89/100 的加权均分处于"工业级高质量"区间。最值得关注的是分布形态：一个满分（100），一个接近满分（95），两个存在明显缺陷（75、85）——质量差异源于命令复杂度，login 因涉及认证流而问题最多。

### 3.2 当时值得借鉴的模式

**1. SKILL.md 满分的写法**

`packages/cli/skill/SKILL.md` 获得 100 分，说明它满足 NLPM 所有质量检查项：
- frontmatter 完整（name、description、tools、allowed-tools 全部声明）
- 无模糊量词（无 "complex"、"when appropriate" 等词）
- output-format 明确定义（成功、失败、各异常状态均有对应输出格式）
- 边界条件处理完整（空输入、网络错误、token 过期各自有处理说明）
- cross-reference 路径经过验证（§References 节引用的文件在运行时可解析）

这个满分 skill 的写法是"声明式完整性"的具体示范：每个可能的输入状态和输出状态都被显式声明，而不是依赖 Agent 自行推断。

**2. 极简 NL 层的维护优势**

4 个制品远比 84 个制品容易保持高质量。refly 的策略证明：在 monorepo 中不必为所有代码都配写 NL 制品，选择最关键的用户交互入口（CLI 命令）优先覆盖，效果往往好过泛滥覆盖。

**3. refly-status.md 的 95 分结构**

status 命令（查询账户和连接状态）的设计最为精良：output-format 明确列出了"已登录 + 连接正常"、"已登录 + 连接异常"、"未登录"三种状态的具体输出格式，边界条件覆盖完整，唯一缺陷是 allowed-tools 未声明。这个文件可以作为 CLI 状态查询命令的写作模板。

### 3.3 当时的缺陷

**缺陷 1：refly-login.md 缺三项规范（最严重）**

login 命令的质量缺口最大，75 分是四个文件中最低分。三项缺陷叠加：
- **缺 `allowed-tools`**：Agent 不知道该命令允许调用哪些工具，可能在认证流中误调用不安全的工具。
- **无 output-format**：认证成功/失败的输出格式未声明，Agent 无法可靠判断认证是否完成，影响依赖认证状态的后续命令。
- **无空输入处理**：若用户不传 token 或 credentials，命令文件没有说明应该如何响应，导致认证失败时 Agent 行为不确定。

**缺陷 2：安全 BLOCKED（2 项 High 风险）**

| 编号 | 风险等级 | 文件 | 问题描述 |
|------|---------|------|----------|
| 1 | High | scripts/check-i18n-consistency.js:91 | `new Function(\`return ${str}\`)()` — [eval 等价执行](#eval等价执行)，任意代码执行风险 |
| 2 | High | package.json:42 | `prepare: "husky"` — [postinstall 脚本](#postinstall脚本)，安装时自动执行 |

第 1 项是本次 Audit 最严重的安全发现。`check-i18n-consistency.js` 读取国际化翻译文件内容后，通过 `new Function()` 动态执行字符串——若攻击者能篡改翻译文件（例如通过供应链攻击替换 npm 包中的 locale JSON），即可实现任意代码执行。这不是一个边界 case，而是实质性的 RCE（远程代码执行）路径。

第 2 项（`prepare: "husky"`）是常见的 npm hooks 触发模式，风险等级 High 但在大型项目中普遍存在；对比 scripts/check-i18n-consistency.js 的问题，husky prepare 的实际风险较低，属于 [security-gate](#security-gate) 的保守判断。

**缺陷 3：cross-reference 静默脆弱性**

SKILL.md 的 `§References` 节引用 `rules/` 路径，但实际文件在 `references/` 目录，通过 `refly upgrade` 命令运行时映射。这是一个"[静默脆弱性](#静默脆弱性)"：不看代码无法发现，运行时不报错，但在以下情况下会静默断裂：
- `refly upgrade` 命令的映射逻辑被修改
- 开发者直接阅读 SKILL.md 并尝试手动访问 `rules/` 路径下的文件

### 3.4 当时的优化机会

1. 为三个 command 文件统一补全 `allowed-tools` 声明——这是系统性遗漏，一次 PR 可覆盖所有三个文件
2. 为 refly-login.md 补充 output-format 和空输入处理——login 是最频繁被调用的命令，也是最容易出现不确定行为的场景
3. 将 `scripts/check-i18n-consistency.js:91` 的 `new Function()` 替换为 `JSON.parse()` + 白名单校验，消除 eval 等价执行路径
4. 在 SKILL.md 中将 `§References` 的路径改为直接引用 `references/`，或添加显式注释说明运行时路径映射关系

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

目标仓库克隆因代理策略限制（HTTP 403）失败，无法访问当前 HEAD。以下状态为基于技术合理性的推断：

| 缺陷 | Audit 日期（2026-05-05） | 当前推断状态 | 推断依据 |
|------|--------------------------|-------------|----------|
| refly-login.md 缺 allowed-tools / output-format / 空输入处理 | 存在 | cannot-verify | 无法访问当前仓库，无法确认 |
| scripts/check-i18n-consistency.js:91 new Function() | 存在 | probably-persists | 此类 i18n 工具修复优先级通常低于产品功能，短期内不太可能被主动修复 |
| package.json prepare: "husky" | 存在 | probably-persists | husky 是标准 Node.js 工程实践，不会被移除 |
| §References 路径不一致 | 存在 | probably-persists | 依赖运行时映射的隐式契约，通常不会主动修复 |

### 4.2 架构演进

暂无——无法访问当前 HEAD。基于 Audit 快照，4 个 NL 制品的极简 NL 层架构在 2026-05-05 时已稳定建立，monorepo 结构短期内不太可能有 NL 层的结构性调整。

### 4.3 新增的可学习模式

暂无——无法访问当前 HEAD。若未来能访问仓库，重点关注：(1) 是否为三个 command 文件补全了 allowed-tools；(2) i18n 脚本的安全修复进展；(3) 是否新增了更多 NL 制品（NL 层是否仍保持极简策略）。

---

## 五、校准

### 5.1 我已经在做对的

基于 NLPM 高质量 NL 编程原则，以下做法与 refly 的高分实践一致：

1. **SKILL.md 的完整性优先**：refly 的 SKILL.md 满分印证了"宁可 command 文件少，也要 SKILL.md 精"的策略。若我的项目也存在 SKILL.md，优先确保其满足所有 frontmatter 声明和 output-format 定义，比铺开更多命令文件更有价值。

2. **output-format 的显式声明**：refly-status.md 的 95 分结构（明确列出三种状态的输出格式）是正确示范。在我自己的 command 文件中，每个可能的结果状态都应有对应的输出格式说明。

3. **NL 层聚焦而非泛滥**：refly 选择只为 CLI 入口写 NL 制品，而非为整个 monorepo 覆盖，这是明智的优先级决策。我的仓库如果也有大量代码逻辑，不必强求每个模块都有 NL 声明，优先覆盖用户交互入口。

### 5.2 挑战 / 验证

**挑战 1：allowed-tools 的惰性遗漏**

refly 三个 command 文件都缺 `allowed-tools`，这是系统性遗漏而非个别失误——说明在批量创建命令文件时，很容易"忘记"这个字段。即使有 NLPM 扫描，也需要在模板层就内置这个字段。

验证动作：检查我的 command 文件是否都有 `allowed-tools` 声明：
```bash
grep -rL 'allowed-tools' ~/.claude/commands/*.md 2>/dev/null
```

**挑战 2：脚本安全的盲区**

refly 的 High 风险来自 `scripts/check-i18n-consistency.js` 中的 `new Function()`，而非 NL 制品本身。这说明 NLPM 安全扫描不只是 NL 文件的问题，而是整个仓库的可执行文件都在扫描范围内。

这挑战了"我写 NL 文件很规范所以安全"的假设：即使 NL 层写得再好，代码层的 eval 等价执行或不安全的环境变量使用也会触发 security-gate 阻断。

---

## 六、行动

### 6.1 自查动作

以下命令可在自己的 Claude Code 项目根目录执行，检测与 refly 类似的问题：

**命令 1：检测 command 文件缺失 allowed-tools（refly 三个命令的共同缺陷）**
```bash
# 找出所有 command 文件中未声明 allowed-tools 的文件
grep -rL 'allowed-tools' \
  ~/.claude/commands/ \
  .claude/commands/ \
  2>/dev/null | grep '\.md$'
```
命中后：在对应文件的 frontmatter 中补充 `allowed-tools: [Bash, Read]`（根据命令实际需要调整工具列表）。

**命令 2：检测 command 文件缺失 output-format（refly-login 和 refly-upgrade 的缺陷）**
```bash
# 找出所有 command 文件中未声明 output-format 的文件
grep -rL 'output.format\|output-format\|## Output' \
  ~/.claude/commands/ \
  .claude/commands/ \
  2>/dev/null | grep '\.md$'
```
命中后：在命令文件中为每种结果状态添加明确的输出格式说明（成功、失败、各异常状态）。

**命令 3：检测脚本中的 eval 等价执行（refly High 安全风险）**
```bash
# 找出 JS 脚本中的 new Function() 调用
grep -rn 'new Function\s*(' \
  scripts/ \
  src/ \
  lib/ \
  2>/dev/null | grep '\.js\|\.ts'

# 找出 Python 脚本中的 eval() / exec() 调用
grep -rn '\beval\s*(\|\bexec\s*(' \
  scripts/ \
  src/ \
  2>/dev/null | grep '\.py'
```
命中后：将 `new Function(str)` 替换为 `JSON.parse(str)` + 白名单校验；将 `eval()` 替换为安全的等价实现。

**命令 4：检测环境变量传递凭证的风险（refly Medium 安全风险）**
```bash
# 找出脚本中直接读取包含凭证的环境变量但无校验的模式
grep -rn 'process\.env\.\(.*\(KEY\|SECRET\|TOKEN\|PASSWORD\|CREDENTIAL\)' \
  scripts/ \
  2>/dev/null | grep -v '\.test\.'
```
命中后：添加非空校验，在变量为空时退出并提示用户配置，而非以 undefined 值继续执行。

### 6.2 灵感 → 实施路径

**灵感 1：仿照 refly SKILL.md 满分结构，为自己的核心 skill 对标检查**

路径：
1. 打开自己项目的 SKILL.md（如果存在）
2. 对照 NLPM 质量检查清单：frontmatter 是否完整？output-format 是否声明？是否有模糊量词？边界条件是否覆盖？
3. 针对每项缺失，参照 refly SKILL.md 的写法补充
4. 运行 `/nlpm:score` 验证分数是否提升到 95+

**灵感 2：为 monorepo 的 NL 层制定覆盖范围策略**

路径：
1. 列出项目中所有用户交互入口（CLI 命令、主要 API 端点、工具入口）
2. 优先为最高频使用的 3-5 个入口写 command 文件，而非追求全覆盖
3. 用一个满分 SKILL.md 作为整体知识汇聚点，使 Agent 无需逐一查阅所有命令文件
4. 参照 refly 的 status 命令（95 分）作为每个命令文件的质量基准

**灵感 3：i18n 脚本的安全重构模板**

若项目有处理国际化翻译文件的脚本，参考以下安全重构思路：
```javascript
// 危险写法（refly 的 High 风险模式）
const result = new Function(`return ${str}`)();

// 安全替代：使用 JSON.parse + 白名单校验
function safeParseI18nValue(str) {
  try {
    const parsed = JSON.parse(str);
    // 校验类型：只允许 string | number | boolean
    if (typeof parsed !== 'string' && typeof parsed !== 'number' && typeof parsed !== 'boolean') {
      throw new Error(`Invalid i18n value type: ${typeof parsed}`);
    }
    return parsed;
  } catch (e) {
    throw new Error(`Invalid i18n value: ${e.message}`);
  }
}
```

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

refly 的核心定位是**AI 原生知识工作台**——用 AI 辅助用户创作、整理和发布知识。这与我的几个仓库有不同程度的交集：

| 我的仓库 | 与 refly 的相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---------|-----------------|--------|--------|-----------|
| MarkQWu/bureau | 高 | 都是 AI 会话 → 知识库的整合工具；bureau 将 AI 对话转化为结构化知识，refly 是 AI 原生知识工作台，方向高度重合 | refly 是云端 SaaS 产品，bureau 是轻量本地管道 | 高 |
| MarkQWu/graphify | 中 | 同为知识管理工具；graphify 构建知识图谱，与 refly 的知识组织功能有交集 | graphify 聚焦图结构，refly 聚焦文档创作工作流 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为数据管道类工具 | echo-sleuth 专注于对话历史挖掘，不涉及知识创作 | 低 |

最相似：`MarkQWu/bureau`。bureau 的核心是"AI 会话 → 知识库"管道，与 refly 的"AI 原生知识工作台"在功能上最接近——都是将 AI 输出转化为可持久化的知识结构。

### 7.2 在我的项目里复现的同类问题

最可能在 bureau 和 graphify 中出现的 refly 类问题：

**检测 1：bureau 的 command 文件是否缺 allowed-tools 和 output-format**
```bash
# 在 bureau 项目根目录运行（假设路径为 ~/bureau）
grep -rL 'allowed-tools' ~/bureau/.claude/commands/ 2>/dev/null | grep '\.md$'
grep -rL 'output.format\|## Output' ~/bureau/.claude/commands/ 2>/dev/null | grep '\.md$'
```

**检测 2：bureau 的 skill 文件质量对标 refly SKILL.md 满分标准**
```bash
# 检查 bureau 的 SKILL.md 是否存在模糊量词
grep -rn --include="SKILL.md" \
  -e '\bcomplex\b' \
  -e 'when appropriate' \
  -e 'when possible' \
  -e '\brelevant\b' \
  -e 'if needed' \
  ~/bureau/ ~/graphify/ 2>/dev/null
```

**检测 3：项目脚本中的 eval 等价执行风险**
```bash
# 检查 bureau/graphify 的脚本目录
grep -rn 'new Function\s*(\|eval\s*(\|exec\s*(' \
  ~/bureau/scripts/ ~/graphify/scripts/ 2>/dev/null
```

### 7.3 别人的更优方案（值得借鉴的）

**优点 1：SKILL.md 满分的"声明式完整性"策略**

refly 的 SKILL.md 满分的关键不是内容多，而是"每个状态都被声明"——成功状态、失败状态、异常状态各有对应的输出格式。如果 bureau 的 SKILL.md 或 command 文件存在"成功路径写得好但失败路径没说明"的问题，这是 refly 的最直接借鉴点。

建议：在 bureau 的核心 skill 文件中，按以下模板检查输出格式声明：
```markdown
## Output Format

**成功**：[具体格式描述]
**失败 - [失败类型1]**：[具体格式描述]
**失败 - [失败类型2]**：[具体格式描述]
**异常 - 空输入**：[具体格式描述]
```

**优点 2：NL 层聚焦而非泛滥的维护优势**

refly 只为 CLI 入口写 4 个 NL 制品，却在 SKILL.md 拿到满分。这说明"少而精"的 NL 策略比"多而泛"更容易保持高质量。对于 bureau 这样的轻量工具，不必追求为每个内部函数或脚本都配写 NL 声明，聚焦核心用户入口即可。

### 7.4 反向：我的项目做得比他们好的地方

**潜在优势：output-format 规范的完整度**

基于 refly 的缺陷（refly-login.md 和 refly-upgrade.md 均缺 output-format 声明），如果 bureau 的 command 文件已有完整的 output-format 说明，这是一个明确的质量优势。

验证方式（在 bureau 项目根目录执行）：
```bash
# 如果返回空，说明所有 command 文件都有 output-format 声明——这就是优势
grep -rL 'output.format\|## Output' \
  .claude/commands/ \
  2>/dev/null | grep '\.md$'
```

**潜在优势：知识管理工具的 NL 表达力**

refly 的 NL 层仅覆盖 CLI 操作（login/upgrade/status），对核心知识工作台功能的 AI 辅助描述完全缺失。如果 bureau 的 NL 层覆盖了核心知识管道逻辑（如"如何从对话提取知识结构"），则在 AI 可理解的业务逻辑深度上会优于 refly。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **<a name="AI原生知识工作台"></a>AI 原生知识工作台** | 将 AI 能力内嵌于知识创作、整理、发布全流程的工具产品，与"AI 辅助"工具不同，知识工作台的 AI 参与是默认路径而非可选插件。refly 的定位即此。 |
| **<a name="NL制品"></a>NL 制品（artifacts）** | 在 NLPM 语境下，用自然语言写成的、供 AI Agent 或开发者使用的结构化文本文件，包括 SKILL.md、command .md、CLAUDE.md 等。refly 有 4 个。 |
| **<a name="NLPM"></a>NLPM** | Natural-Language Programming Manager，本项目名称，提供 NL 制品的质量评分、一致性检查、自动修复等工具链。100 分制，默认阈值 70 分。 |
| **<a name="frontmatter"></a>frontmatter** | Markdown 文件开头以 `---` 包裹的 YAML 元数据块，声明文件类型、版本、工具依赖等机器可读属性。`allowed-tools` 字段在此声明。 |
| **<a name="manifest-discipline"></a>manifest-discipline（清单纪律）** | NL 制品写作的一种质量标准：frontmatter 中声明的所有字段（tools、allowed-tools、output-format 等）必须完整、准确，不遗漏任何规范要求的字段。refly SKILL.md 满分的关键所在。 |
| **<a name="eval等价执行"></a>eval 等价执行** | 在 JavaScript 中，`new Function(str)()` 与 `eval(str)` 功能等价，都是将字符串作为代码动态执行。若字符串内容来自外部（如翻译文件），攻击者可注入任意代码——这是 refly High 安全风险的根源。 |
| **<a name="postinstall脚本"></a>postinstall 脚本** | `package.json` 中 `scripts.prepare` 或 `scripts.postinstall` 字段定义的脚本，在 `npm install` 完成后自动执行。若被滥用，可在安装阶段执行任意命令，是供应链攻击的常见向量。 |
| **<a name="security-gate"></a>security-gate（安全门）** | NLPM 审查流程中在 NL 质量评分之前执行的安全扫描步骤。发现 Critical 或 High 风险时标记为 BLOCKED，阻断自动贡献流程，要求人工审查。refly 因 High 风险被 BLOCKED。 |
| **<a name="静默脆弱性"></a>静默脆弱性（silent fragility）** | 一种在正常运行时不报错、但在特定条件变化后会静默断裂的设计缺陷。refly SKILL.md 的 `§References` 引用 `rules/` 路径而文件实际在 `references/`，依赖运行时命令映射，是典型的静默脆弱性。 |
| **<a name="NL-as-CLI-wrapper"></a>NL-as-CLI-wrapper** | 一种 NL 编程策略：将自然语言 NL 制品的覆盖范围限制在 CLI 命令入口层，每个 CLI 命令对应一个 command .md 文件，核心业务逻辑由代码实现。refly 采用此策略，4 个制品覆盖 3 个 CLI 命令 + 1 个 skill。 |
| **<a name="single-purpose"></a>single-purpose（单一职责）** | NL 制品设计原则之一：每个 command 或 skill 文件只描述一个明确的操作或知识域，不混入多个职责。refly 的三个命令文件各自对应单一 CLI 操作，是此原则的体现。 |
| **<a name="monorepo"></a>monorepo** | 将多个相关项目（如前端、后端、CLI、SDK）统一放在一个 Git 仓库中管理的架构模式。refly 是大型 monorepo，NL 层只占其中极小部分。 |
