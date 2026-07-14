# pbakaus/impeccable — 学习案例

**仓库**：https://github.com/pbakaus/impeccable
**Stars**：46,376 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-10（历史快照）| **生成日期**：2026-07-14（基于当前 HEAD）
**主题标签**：`template-design`, `security-gate`, `manifest-discipline`, `nl-binary-hybrid`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
impeccable 是 Peter Bakaus（前 Google 工程师）开发的「AI 前端设计语言」插件，定位为「让 AI harness 在前端设计上更好」的 Claude Code 插件，以 46,376 颗星成为目前最受欢迎的 Claude Code 插件之一。包含一个核心 SKILL.md（`/impeccable`）、23 个子命令（`/impeccable polish`、`/impeccable audit`、`/impeccable critique` 等）和一套反模式检测库。

- **创建时间**：2025-11-16
- **核心语言**：JavaScript（构建脚本）+ Bun（包管理）
- **获取方式**：`claude plugin install impeccable@pbakaus --scope project`
- **生态位置**：设计工具类插件的标杆，官方网站 [impeccable.style](https://impeccable.style) 独立运营

### 1.2 架构剖析
- **目录结构**：
  ```
  pbakaus/impeccable
  ├── .claude-plugin/
  │   ├── plugin.json        ← manifest（17 个 skill）
  │   └── marketplace.json   ← 市场元数据
  ├── CLAUDE.md              ← 开发者指南
  ├── scripts/               ← 构建脚本（14 个）
  │   ├── build.js
  │   ├── build-extension.js
  │   └── lib/sub-pages-data.js
  ├── extension/             ← Chrome 扩展
  ├── server/                ← 开发服务器
  ├── bin/                   ← CLI 工具
  └── bun.lock               ← 锁定版本
  ```
- **文件类型分布**：17 个 skill / 0 个 agent / 2 个 [manifest](#manifest) / 8 个构建脚本 / 4 个浏览器扩展文件
- **编排关系**：单一顶层 skill（`/impeccable`）派生 23 个子命令，形成「主 skill + 子命令」的分层架构。构建脚本在部署时自动提取 ANTIPATTERNS 数组，注入到多个平台（浏览器扩展、CLI、网站）。
- **跨件契约**：`arrange/SKILL.md` 和 `typeset/SKILL.md` 通过相对路径引用 `reference/spatial-design.md` 和 `reference/typography.md`。由 `scripts/build.js` 中的 `generateCounts()` 验证 manifest 与实际文件数量是否一致。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：[「NL 表皮 + 原生代码核心」](#nl-表皮原生代码核心)——CLAUDE.md 和 skill 是 AI 可读的「界面」，但真正的逻辑（反模式库、构建流程）由 JavaScript/Bun 实现，AI 不接触核心逻辑。
- **解决什么问题**：AI 生成的前端设计常常违反基础设计规范（间距不一、字体随意、颜色混乱）。impeccable 通过将设计规范编码为 skill 步骤，让 AI 按设计系统行事。
- **Trade-off**：选择独立仓库 + 独立网站，而非在某个大仓库内作为子目录——这带来了更好的品牌认知和专注度，代价是用户需要单独安装。
- **认知模型**：作者把 AI skill 视为「专业角色扮演协议」——`/impeccable audit` 不是「运行一段代码」，而是「成为一个有设计专业背景的审查员」。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「NL 表皮 + 原生代码核心」** — [frontmatter](#frontmatter) 和 SKILL.md 作为 AI 可读层，真正的领域知识（反模式库、构建脚本）由原生代码维护，两层通过构建时注入连接。

模式特征清单：
- 特征 1：CLAUDE.md / SKILL.md 是 AI 的交互界面，但不包含核心业务逻辑
- 特征 2：构建脚本（`build.js`）在编译时将核心数据（ANTIPATTERNS）注入到多个目标（浏览器扩展、CLI、网站）
- 特征 3：`plugin.json` 和 `marketplace.json` 双 [manifest](#manifest)，分别面向运行时和市场展示
- 特征 4：独立网站（impeccable.style）作为文档和推广渠道，与 GitHub 仓库分离
- 特征 5：Chrome 扩展 + CLI + Claude Code skill 三端覆盖，同一套反模式库服务三个入口

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有独立域知识库的设计/规范工具 | ✅ 高度适用 | 知识库需要版本控制和 CI 保护 |
| 多端（浏览器+CLI+AI）统一规则 | ✅ 高度适用 | 构建注入确保规则一致性 |
| 纯文档类 skill | ❌ 不适用 | 过度工程，直接写 SKILL.md 更简单 |
| 个人项目 | ❌ 不适用 | 维护两套 manifest + 构建脚本成本高 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生代码核心（本仓库） | impeccable | 知识库有 CI 保护，多端一致 | 上手成本高，需要构建步骤 |
| 纯 NL 平铺式 | openai/symphony | 零依赖，最简单 | 反模式库只能写在 Markdown 里，难以版本化 |
| 多 plugin 聚合式 | ccplugins | 覆盖场景广 | 无构建保障，质量参差 |

### 2.4 改进空间
1. **当前问题**：`plugin.json` 描述说「1 skill with 23 commands」但实际 plugin.json 显示 17 个 skill（0 个 commands）——数量口径不一致 **改进做法**：在 `generateCounts()` 中同时校验描述文本中的数字 **预期收益**：市场展示与实际功能一致，用户不会因为数量差异困惑
2. **当前问题**：`arrange/SKILL.md` 和 `typeset/SKILL.md` 引用的相对路径指向不存在的位置（应为 `impeccable/reference/`） **改进做法**：在构建步骤中校验所有相对路径 **预期收益**：消除 AI 因找不到引用文件而静默失败

---

## 三、过去审查发现（2026-04-10 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-10 当时得分 **93/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| plugin.json | 100 | 无问题 |
| marketplace.json | 100 | 无问题 |
| CLAUDE.md | 80 | 引用了 gitignored 的 `evals/AGENT.md` |

### 3.2 当时值得借鉴的模式
1. **`plugin.json` 和 `marketplace.json` 均 100 分** → 为什么好：每个 manifest 字段完整、版本合法、描述语义清晰。原文：`.claude-plugin/plugin.json` → 借鉴：manifest 文件应当首先写、最后审，是插件质量的基线
2. **构建时数量校验（`generateCounts()`）** → 为什么好：自动检测 manifest 声明的 skill 数量与实际文件数量是否一致，防止漂移。原文：`scripts/build.js` → 借鉴：有 manifest 的项目应加 CI 校验「manifest 里的 skill 数 == 目录里的 SKILL.md 数」
3. **`land_watch.py` 风格的 `sanitize_terminal_output`**（来自被引用的 symphony 模式）→ 为什么好：impeccable 的浏览器扩展 `panel.js` 已用 `JSON.stringify()` 转义——这个审计建议被采纳

### 3.3 当时的缺陷
1. **`build.js` 中 `new Function(动态字符串)`** → 为什么失败：构建时从源码提取 ANTIPATTERNS 数组并用 `new Function()` 求值，等同于在构建机器上执行动态代码，如果 ANTIPATTERNS 数据被污染则攻击者可以在 `bun run build` 时执行任意代码 → 自查：我的构建脚本有没有 `new Function()`/`eval()`？
2. **CLAUDE.md 引用 gitignored 的 `evals/AGENT.md`（第 101、138 行）** → 为什么失败：AI 读 CLAUDE.md 时会尝试打开 `evals/AGENT.md`，但文件不存在（gitignored），导致 AI 得到错误指引 → 自查：我的 CLAUDE.md 有没有引用 gitignored 文件？
3. **命令数量口径不一致**（manifest 说 18，实际 21）→ 为什么失败：用户在市场看到「18 commands」，安装后发现「21 commands」，轻微的信任损耗 → 自查：我的 plugin.json 描述中的数字与实际文件数量一致吗？

### 3.4 当时的优化机会
1. 将 `new Function()` 替换为直接 ESM import（ANTIPATTERNS 已经是 export 的）
2. 修复 CLAUDE.md 中的 stale reference（删除或替换 evals 相关内容）
3. 用 `generateCounts()` 同步描述文本中的数字与实际 skill 数量

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `new Function()` 动态代码执行 | `grep -n "new Function" scripts/build.js` | **已修复**：build.js 中不再有 `new Function` | 高安全风险在 3 个月内被修复 |
| `evals/AGENT.md` stale reference | `grep -n "evals/AGENT.md" CLAUDE.md` | **已修复**：CLAUDE.md 中无 `evals/AGENT.md` 引用（改为引用私有仓库路径） | 清理了不存在的文件引用 |
| 命令数量不一致 | `jq '.skills | length' .claude-plugin/plugin.json` | **仍存在**：plugin.json 显示 17 个 skill，但描述说「23 commands」 | 数量口径仍不一致（单位从 commands 变 skills） |

### 4.2 架构演进
从审计（2026-04-10）到现在（2026-07-14），两个高严重度安全问题均已修复——`new Function()` 和 stale CLAUDE.md 引用。CLAUDE.md 中 evals 相关内容改为引用私有仓库路径 `~/code/impeccable-evals/`，说明 evals 框架已移到本地私有仓库，不再随主仓库分发。仓库持续活跃（最新 push 2026-07-13）。

### 4.3 新增的可学习模式
暂无——仓库保持稳定演进，无架构层面的新增模式。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的构建脚本中没有 `new Function()`/`eval()` 模式（我的项目多为纯 Markdown 插件）
2. `MarkQWu/bureau` 和 `MarkQWu/gstack` 的 CLAUDE.md 不引用 gitignored 文件
3. `MarkQWu/bureau` 的 plugin.json 描述语义清晰，无截断

### 5.2 挑战 / 验证
- **验证**：impeccable 在 3 个月内修复了两个高安全风险（`new Function()`），证明「REVIEW 状态」的安全问题在活跃仓库中确实会被跟进修复。这说明 NLPM 的安全审计 → PR 流程在高质量仓库里是有效的。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的构建脚本有没有 eval/new Function
grep -rn "new Function\|eval(" ~/*/scripts/ ~/.claude/scripts/ 2>/dev/null | grep -v "node_modules"
```
命中后：将动态执行替换为静态 import（如果是数据文件）或安全解析（如果是配置）。

```bash
# 检查 CLAUDE.md 有没有引用 gitignored 文件
if [ -f .gitignore ]; then
  grep -n -f .gitignore CLAUDE.md 2>/dev/null | head -10
fi
```
命中后：更新 CLAUDE.md，删除对 gitignored 文件的引用，或改为「见私有仓库 ~/xxx」的说明。

```bash
# 检查 plugin.json 描述中的数字与实际 skill 数是否一致
python3 -c "
import json, re
d = json.load(open('.claude-plugin/plugin.json'))
desc = d.get('description', '')
nums = re.findall(r'\d+', desc)
actual = len(d.get('skills', d.get('commands', [])))
print(f'desc numbers: {nums}, actual: {actual}')
" 2>/dev/null
```
命中后：更新 description 中的数字，或在构建脚本中自动生成。

### 6.2 灵感 → 实施路径
1. **想法**：给 `MarkQWu/bureau` 的 `plugin.json` 描述添加 examples（模仿 impeccable 的「1 skill with X commands」模式）
   - **为何可行**：bureau 有 7 个 skill，描述格式可以借用 impeccable 的「7 skills: recall/capture/compile/review/lint/guide/scribe」
   - **第一步**：读 `bureau/.claude-plugin/plugin.json`，更新 description 字段

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度
- **本案例 pbakaus/impeccable 的核心目的**：将专业设计规范编码为 AI skill，让 AI 在前端开发时遵守设计系统

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 都是工作流增强插件，都有 CLAUDE.md | gstack 面向工程工作流，impeccable 面向设计 | 中 |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件 | bureau 是知识管理，impeccable 是设计规范 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都有 skill 集合 | drama-workshop 是特定领域，不是通用工具 | 低 |

若目的对齐度低，本节仅做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 引用 gitignored 文件 | `grep -n -f .gitignore CLAUDE.md` | gstack/CLAUDE.md 未发现此类引用 | 低（未命中） |
| plugin.json 描述中数字不一致 | 目视检查 + python3 脚本 | bureau plugin.json 描述可能未声明 skill 数量 | 中 |
| 构建脚本 eval/new Function | `grep -rn "new Function\|eval(" scripts/` | 我的项目基本无构建脚本，未命中 | 低（未命中） |

**命中后的具体行动建议**：
- `MarkQWu/bureau/.claude-plugin/plugin.json` → 更新 description 加入「7 skills: ...」的数量说明，并考虑加一个构建步骤校验数量 → 10 分钟

### 7.3 别人的更优方案

1. **领域**：`manifest-discipline`（双 manifest 100 分）
   - **本案例做法**：`plugin.json` 和 `marketplace.json` 均 100 分，字段完整、描述语义清晰、version 合法（`.claude-plugin/plugin.json`，`.claude-plugin/marketplace.json`）
   - **我的项目现状**：gstack 的 plugin.json（如果存在）可能未检查是否 100 分
   - **如何借鉴**：运行 `/nlpm:score ./` 对我的 manifest 文件评分，目标 100 分

2. **领域**：`nl-binary-hybrid`（NL 层与代码层分离）
   - **本案例做法**：ANTIPATTERNS 库在 JavaScript 中维护，通过构建时注入到 SKILL.md——业务逻辑有 CI 保护，AI 看到的是注入后的结果（`scripts/build.js`）
   - **我的项目现状**：我的 skill 把所有逻辑都直接写在 SKILL.md 里，无构建保护
   - **如何借鉴**：对于有规则库/数据库支撑的 skill，考虑把规则数据移到 JSON/JS 文件，通过简单脚本生成 SKILL.md，获得 CI 校验能力

### 7.4 反向：我的项目做得比他们好的地方
本案例未发现我的项目更优的维度。impeccable 在 manifest 质量、安全修复速度、多端统一方面均优于我目前的项目。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据（如 `name`、`description`、`model`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。impeccable 有两个：`plugin.json`（Claude Code 运行时用）和 `marketplace.json`（Claude Code 插件市场用），各自面向不同受众，内容略有差异，需要保持同步。

### <a name="nl-表皮原生代码核心"></a>NL 表皮原生代码核心
> 一种插件架构模式：Markdown 文件（CLAUDE.md、SKILL.md）是「NL 表皮」，AI 通过它理解任务；而真正的业务逻辑（规则库、算法）由 JavaScript/Python/Go 等原生代码实现，AI 不直接接触。两层通过构建脚本在编译时连接。优点：业务逻辑可以做单元测试、CI 保护、版本化；缺点：开发流程更复杂，需要构建步骤。
