# 2389-research/review-squad — 学习案例

**仓库**：https://github.com/2389-research/review-squad
**Stars**：1 | **来源**：upstream（exemplar_published，SECURITY CLEAR）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-09（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `manifest-discipline`, `cross-reference`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Review Squad 是一个 Claude Code 插件，向项目派遣一组专业亚智能体从不同角度审查代码库——专家审计、普通用户视角、任务完成流程测试、吹毛求疵的语法审查（well-actually 风格）。每个技能对应一类审查员角色，插件核心设计理念是「同一项目，多角度并行审查」。

关键事实：
- 作者 2389-research 同时维护 simmer（本次第二个案例）等多个精品插件
- 4 个技能（experts / normies / regulars / well-actually）全部在 [plugin.json](#manifest) 中声明
- 无钩子、无脚本、无二进制——纯 [Markdown](#frontmatter) 插件，零执行面
- NLPM 评分 96/100，唯一缺陷在 CLAUDE.md 层而非技能层

### 1.2 架构剖析
```
review-squad/
├── CLAUDE.md               ← 插件总说明（无 frontmatter）
├── .claude-plugin/
│   └── plugin.json         ← manifest，声明 4 个 skills，commands: []
└── skills/
    ├── experts/SKILL.md    ← 并行模式：多专家同时审查
    ├── normies/SKILL.md    ← 顺序模式：浏览器 MCP 模拟普通用户
    ├── regulars/SKILL.md   ← 顺序模式：任务完成验证
    └── well-actually/SKILL.md ← 顺序模式：语法/细节挑剔
```

- **文件类型分布**：4 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：平级并列，无路由层；experts 内部并行派遣多个子智能体，另外三个顺序执行
- **跨件契约**：CLAUDE.md 记录了"代码类项目可并行，浏览器类必须顺序"的约定，各 SKILL.md 与此一致；plugin.json 的 `commands: []` 是刻意设计（只有 skills，无 slash commands）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「角色即技能，每个 skill 对应一类审查视角」——没有路由层，用户直接选择想要的视角
- **解决什么问题**：单一 code review 视角不够——QA 找 bug、UX 看可用性、安全工程师看漏洞、普通用户看流程，作者把这四类视角做成可按需召唤的技能
- **Trade-off**：4 个独立技能 vs. 1 个带参数的统一技能——选择了前者，用户体验更直接，代价是无法组合调用
- **反映什么认知模型**：把「不同角色的 AI 专家」封装为可组合的技能原语，每次调用等于请来一批审查员

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「角色面板插件」模式**：一个插件包含 N 个独立的角色视角技能，每个技能代表一类利益相关者的审查方式，互相没有依赖。

模式特征清单：
- 特征 1：技能数量有限（≤5），每个职责单一明确
- 特征 2：plugin.json 只有 skills，没有 slash commands
- 特征 3：技能之间无数据流，不互相调用
- 特征 4：同一项目可以跑多个技能，结果独立
- 特征 5：部分技能共用同一技术前提（如 browser MCP），CLAUDE.md 统一说明

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 不同视角的代码/内容审查 | ✅ 高度适用 | 每个视角独立，平行价值清晰 |
| 需要跨步骤传递上下文的流程 | ❌ 不适用 | 技能间无数据流，无法协作 |
| 想让用户一键触发全流程 | ❌ 不适用 | 用户需要逐个选择视角 |
| 轻量快速交付（纯 Markdown） | ✅ 高度适用 | 零构建成本，安装即用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 角色面板（本仓库） | review-squad | 角色语义清晰，零学习成本 | 无法组合，无法自动依次执行 |
| 路由 + 子技能 | simmer（本次 Case 2） | 可组合，流程自动化 | 较复杂，需要路由逻辑 |
| 单技能带参数 | 大多数简单插件 | 接口统一 | 角色切换需用户理解参数 |

### 2.4 改进空间
1. **当前问题**：CLAUDE.md 无 [frontmatter](#frontmatter)，NLPM 扫描器无法机读 **改进做法**：加 `name: review-squad` + `description:` 的 YAML 头 **预期收益**：NLPM 评分从 96 → 100，且在插件市场展示正确元数据
2. **当前问题**：CLAUDE.md 无调用示例，用户不知道输入什么触发技能 **改进做法**：加一个 `## 示例` 段落，每个技能各一行触发短语示例 **预期收益**：新用户上手时间从"自己摸索"→"30 秒"

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 得分 96/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 80 | 无 frontmatter，无调用示例，描述过于简短 |
| skills/experts/SKILL.md | 98 | "relevant" 模糊量词（L95） |
| skills/normies/SKILL.md | 99 | 无 |
| skills/regulars/SKILL.md | 99 | 无 |
| skills/well-actually/SKILL.md | 99 | 无 |
| .claude-plugin/plugin.json | 100 | 无 |

### 3.2 当时值得借鉴的模式
1. **plugin.json 满分设计** → manifest 完整声明所有字段，无遗漏 → `plugin.json` → 借鉴：每次新插件先写 manifest 再写 skills
2. **跨文件约定记录在 CLAUDE.md** → "Sequential for browser-using agents; code-only agents can run in parallel" → CLAUDE.md 行为约束段落 → 借鉴：把调度模式的决策依据写进 CLAUDE.md
3. **No-Code Guards 一致性** → CLAUDE.md 文档了 guard，技能文件逐字实现 → 跨件一致性范例 → 借鉴：记录约定后，每个文件都要验证实现
4. **极简执行面** → 无 hook、无脚本、零安全风险 → 纯 Markdown → 借鉴：能用 Markdown 实现的，不要引入 shell

### 3.3 当时的缺陷
1. **CLAUDE.md 缺少 frontmatter** → 为什么失败：NLPM 扫描器解析 Markdown 文件时，依赖 YAML frontmatter 识别文件性质；没有这层，文件对工具链不可见 → 自查：我的 gstack、bureau、echo-sleuth、shiji-kb 的 CLAUDE.md 全都缺 frontmatter
2. **无调用示例** → 为什么失败：用户拿到插件不知道输入什么——文字描述"它能审查你的项目"不如"输入 'review-squad this project as a normie user'"直观 → 自查：bureau 的 CLAUDE.md 同样缺具体触发示例
3. **"relevant" 无定义标准** → 为什么失败：L95 写的是 "always suggest what's relevant"——什么是 relevant？对 SEO 技能来说是关键词密度，对安全技能来说是 CVE，两者完全不同；LLM 会自行推断，导致不同对话结果差异巨大 → 自查：需要检查我的 skills 中是否有类似问题

### 3.4 当时的优化机会
1. CLAUDE.md 加 YAML frontmatter（ROI 最高，5 分钟可完成，分数 +16 分）
2. 加触发短语示例（让新用户 30 秒上手）
3. experts/SKILL.md L95 "relevant" → 改为具体建议标准（如"建议直接覆盖检测到的技术栈缺口的审查员"）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 无 frontmatter | `head -3 CLAUDE.md`：首行为 `# Review Squad Plugin` | **持续** | 4 个月过去，维护者未修 |
| "relevant" L95 无定义标准 | `grep -n "relevant" skills/experts/SKILL.md` → L95 仍为原文 | **持续** | 低优先级，语义勉强可接受 |
| 无调用示例 | `grep -n "示例\|example\|trigger" CLAUDE.md` → 无结果 | **持续** | 文档只有描述，仍无示例 |

### 4.2 架构演进
当前 HEAD 与 audit 时结构完全相同：4 技能 + plugin.json + CLAUDE.md，未见任何重组。作者选择了保持稳定而不是持续迭代——这对一个"纯审查工具"来说是合理策略。

### 4.3 新增的可学习模式
暂无。当前 HEAD 无架构变化，audit 时已经完整覆盖了设计要点。

---

## 五、校准

### 5.1 我已经在做对的
1. **极简执行面意识**：bureau 和 echo-sleuth 也是纯 Markdown 插件，没有无谓引入 shell 脚本
2. **plugin.json manifest 完整**：与 review-squad 类似，我的插件 manifest 覆盖了所有 skill
3. **跨文件约定文档化**：bureau 的 BUREAU.md 记录了 `canon/` 管辖约定，与 CLAUDE.md 的行为约束记录思路一致
4. **单职责技能**：bureau 的 capture/compile/review 等技能各自职责清晰，与 review-squad 的角色分离对齐

### 5.2 挑战 / 验证
这个案例**验证了**：插件不一定需要复杂架构——4 个各 99 分的单职责技能比 1 个 90 分的全能技能更有价值。我之前在设计 bureau 时犹豫是否需要路由层，这个案例告诉我：如果视角之间没有数据流，平铺就是最优架构。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 CLAUDE.md 是否有 frontmatter
for f in ~/projects/*/CLAUDE.md; do
  head -1 "$f" | grep -q "^---" && echo "OK: $f" || echo "MISSING frontmatter: $f"
done
```
命中后：为对应 CLAUDE.md 开头加 `---\nname: <插件名>\ndescription: <一句话描述>\n---`

```bash
# 检查 skills 中是否有 "relevant" 缺标准
grep -rn "\brelevant\b" ~/.claude/plugins/*/skills/*/SKILL.md | grep -v "#"
```
命中后：把 "relevant" 替换为具体的选择标准（如"与检测到的错误模式直接对应的"）

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 加一个"角色面板"层——capture/review/lint 已经是角色了，但 CLAUDE.md 没有显式说明每个角色的调用时机
   - **为何可行**：review-squad 的 CLAUDE.md 里有一段"何时用哪个技能"说明，且经过验证能有效引导用户
   - **第一步**：在 `bureau/CLAUDE.md` 加 `## 何时用哪个技能` 段落，5 分钟
2. **想法**：给 gstack 的每个技能加触发示例
   - **为何可行**：gstack 有 50+ 技能，用户很难记住触发短语；review-squad 模式证明在 CLAUDE.md 层统一列出示例比每个 SKILL.md 各自写更高效
   - **第一步**：找 `gstack/CLAUDE.md`，在每个技能描述后加一行 `> 触发：xxx`，30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：派遣多角色审查员从不同视角审查代码库，每个视角独立

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 多个独立技能，每个对应一类工作角色（CEO/设计师/工程师） | gstack 有路由（plan-* 技能系列）；review-squad 纯平铺 | 高 |
| MarkQWu/bureau | 中 | 多技能平铺，各自职责清晰 | bureau 是知识管理流水线，技能有顺序依赖；review-squad 无依赖 | 中 |
| MarkQWu/drama-workshop-skills | 中 | 纯技能集合，无路由 | drama 是戏剧领域专用；review-squad 是通用审查工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 无 frontmatter | `head -1 */CLAUDE.md \| grep "^---"` | **gstack / bureau / echo-sleuth / shiji-kb 全部命中** | 高 |
| 无调用示例 | `grep -n "example\|示例\|trigger" */CLAUDE.md` | gstack、bureau 均无触发示例段落 | 中 |
| 模糊量词"relevant" | `grep -rn "\brelevant\b" */skills/*/SKILL.md` | 需在各项目逐一验证 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/gstack/CLAUDE.md` → 文件顶部加 3 行 frontmatter → 5 分钟
- `MarkQWu/bureau/CLAUDE.md` → 同上 + 加 `## 何时用哪个技能` 段落 → 15 分钟

### 8.3 别人的更优方案

1. **领域**：技能内并行调度文档化
   - **本案例做法**：CLAUDE.md 明确写出"experts 并行，browser 类顺序"的原因 → `CLAUDE.md` 第 11-12 行
   - **我的项目现状**：gstack 的 plan-* 系列与 ios-* 系列是否可并行，CLAUDE.md 完全没说明
   - **如何借鉴**：在 `gstack/CLAUDE.md` 加一节"并行 vs 顺序"，标注哪些技能需要独占上下文

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：gstack/CLAUDE.md 有完整的 build/run/test 命令段落
- **我的做法**：`gstack/CLAUDE.md` 顶部列出了 `bun install` / `bun test` 等命令
- **本案例做法（弱在哪）**：review-squad 的 CLAUDE.md 无任何安装命令或测试命令说明
- **意义**：这是我 gstack 的一个亮点；如果给 review-squad 贡献 PR，可以提这个改进

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model` 等）。Claude Code 读 SKILL.md 或 CLAUDE.md 时先解析 frontmatter 才能知道这个文件的性质和注册方式。
> **对比**：没有 frontmatter 的 Markdown = 纯文本文档；有 frontmatter 的 Markdown = 可被工具链机读的结构化配置文件。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="模糊量词"></a>模糊量词（vague quantifier）
> 在技能文件里出现的、没有明确衡量标准的形容词或副词，如 "appropriate"、"relevant"、"meaningful"、"comprehensive" 等。这类词语会让 AI 每次都自己决定标准，导致同一技能在不同对话中表现不一致。
> **修复方式**：把"推荐相关的内容"改成"推荐与检测到的技术栈缺口直接对应的内容（如检测到无 i18n 则推荐 i18n 审查员）"。
