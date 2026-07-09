# agenticnotetaking/arscontexta — 学习案例

**仓库**：https://github.com/agenticnotetaking/arscontexta
**Stars**：0 | **来源**：upstream（exemplar_published，SECURITY CLEAR，status=contributed）
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-07-09（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `security-gate`, `vague-quantifier`, `template-design`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Ars Contexta 是一个基于 Claude Code 的个人知识管理（PKM）插件，把 AI 会话转化为有组织的持久化知识库。核心理念是「三空间架构」：notes（知识原子）/ thinking（思考过程）/ ops（系统操作）分层管理。插件具有 [钩子](#hook) 系统——SessionStart 时自动定向、PostToolUse(Write) 时自动提交——是本次 4 个案例中唯一有执行面的插件。

关键事实：
- 27 个 NL 产物（技能 + 智能体），最大规模系统（本次 4 案例中）
- 独创 `skill-sources/` 与 `skills/` 双目录结构——前者存原始技能文件，后者存安装生成文件
- 有 `generators/` 目录，用于动态生成技能文件（高级模式）
- 安全层面：5 个安全发现（全部 MEDIUM/LOW），无 CRITICAL/HIGH

### 1.2 架构剖析
```
arscontexta/
├── agents/
│   └── knowledge-guide.md          ← 主动引导智能体（model: sonnet，无 allowed-tools）
├── skill-sources/                  ← 原始技能源文件（9 个缺 model 声明）
│   ├── refactor/ reweave/ reflect/
│   ├── learn/ reduce/ rethink/ ralph/ verify/
│   └── remember/ seed/ graph/ stats/ tasks/
│       pipeline/ validate/ next/
├── skills/                         ← 安装生成的技能（使用 ${CLAUDE_PLUGIN_ROOT} 路径变量）
│   ├── add-domain/ architect/ reseed/
│   ├── setup/ recommend/ ask/ health/ upgrade/
│   └── tutorial/
├── hooks/
│   ├── hooks.json                  ← SessionStart + PostToolUse(Write) 钩子配置
│   └── scripts/                    ← session-orient.sh / write-validate.sh / auto-commit.sh
├── methodology/                    ← 方法论参考文件（被技能引用）
├── reference/                      ← 知识域参考文件
├── generators/                     ← 技能生成器
└── .claude-plugin/plugin.json
```

- **文件类型分布**：27 个 SKILL.md / 1 个 agent / 0 个 command / 3 个 hook 脚本
- **编排关系**：`ralph/SKILL.md` 是 orchestrator，通过 RALPH HANDOFF 协议调度其他子技能；`pipeline/SKILL.md` 是流水线编排器
- **跨件契约**：所有 27 个产物共享 `${vocabulary.*}` 模板变量和 `${CLAUDE_PLUGIN_ROOT}` 路径变量，工具集（`mcp__qmd__search` 等 6 个工具）在文件间保持一致

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「协议化知识管理」——笔记格式、图谱关系、工具调用都有协议规范（三空间架构 + [frontmatter](#frontmatter) 模板 + RALPH HANDOFF），AI 遵循协议而非即兴发挥
- **解决什么问题**：AI 会话结束后知识消失——Ars Contexta 通过钩子自动捕获、提交、建立连接，让每次 AI 会话都沉淀到持久知识库
- **Trade-off**：精细的协议体系 vs. 上手门槛高——`kernel.yaml`、`three-spaces.md`、`derivation-manifest.md` 等方法论文件需要用户先理解体系才能使用
- **反映什么认知模型**：把知识图谱当「真相源」，AI 产生的所有内容都要经过验证才能进入知识库；AI 是知识的加工者，而不是知识的存储者

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「三空间知识管理 + 自动提交钩子」模式**：通过 SessionStart 钩子定向工作上下文，通过 PostToolUse(Write) 钩子自动提交笔记，技能和智能体在写入时遵循知识图谱协议。

模式特征清单：
- 特征 1：笔记存储与 AI 工作流深度集成（钩子自动触发，无需手动操作）
- 特征 2：技能文件使用 `${CLAUDE_PLUGIN_ROOT}` 路径变量，而不是硬编码路径
- 特征 3：所有技能共享同一套词汇表和工具集（`${vocabulary.*}` 变量）
- 特征 4：`skill-sources/` 与 `skills/` 双目录分离——生成的文件与源文件不混在一起
- 特征 5：有明确的"验证关卡"——`verify/SKILL.md` + `write-validate.sh` 双层验证

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 长期个人知识积累 | ✅ 高度适用 | 专为此场景设计，钩子自动化运作 |
| 快速一次性任务 | ❌ 不适用 | 安装成本高，需要配置 vault 和方法论 |
| 团队共享知识库 | ⚠️ 部分适用 | auto-commit 会提交所有内容，多人协作有冲突风险 |
| 希望零运维的插件 | ❌ 不适用 | 钩子脚本需要定期维护和安全审查 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 三空间 + 钩子（本仓库） | arscontexta | 自动化程度高，会话结束即存档 | 架构复杂，安全面宽 |
| 简单 Bureau 模式（同类） | MarkQWu/bureau | 上手门槛低，无钩子 | 需要手动触发技能 |
| 纯 Markdown 笔记插件 | 大多数 PKM 插件 | 零维护成本 | 无自动化，完全靠人工 |

### 2.4 改进空间
1. **当前问题**：9 个 `skill-sources/` 文件缺 `model` 声明 **改进做法**：为每个文件加 `model: sonnet` 或 `model: opus`（深度推理类用 opus） **预期收益**：避免使用环境模型漂移，确保行为可预测
2. **当前问题**：`knowledge-guide.md` 智能体无 `allowed-tools` 声明 **改进做法**：加 `allowed-tools: Read` **预期收益**：智能体真正能读取 `reference/` 目录文件
3. **当前问题**：`auto-commit.sh` 用 `git add -A` 范围过宽 **改进做法**：改为 `git add notes/ ops/ self/ inbox/ templates/ .arscontexta` **预期收益**：避免意外提交敏感内容

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-12 得分 96/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/knowledge-guide.md | 86 | 无输出格式声明（-10），无 allowed-tools（-5），模糊量词（-4） |
| 9 个 skill-sources/*/SKILL.md | 91-95 | 缺 model 声明（每个 -5） |
| 其余技能文件 | 96-100 | 模糊量词轻微扣分 |

### 3.2 当时值得借鉴的模式
1. **词汇表模板变量** → `${vocabulary.note_type}` 等变量在 27 个文件中保持一致 → 全局词汇表 → 借鉴：多文件插件应提前定义共享词汇，防止术语漂移
2. **三空间分层** → notes/thinking/ops 各有不同的处理逻辑和验证规则 → 分层存储协议 → 借鉴：知识管理类工具应明确区分"内容层"与"操作层"
3. **MCP 工具集一致性** → 6 个 `mcp__qmd__*` 工具在所有相关技能中命名完全一致，无别名漂移 → 跨件工具一致性 → 借鉴：我的 bureau 应检查工具名称是否在所有技能中一致
4. **RALPH HANDOFF 协议** → orchestrator 和 pipeline 都引用同一协议名，语义一致 → 跨件协议命名 → 借鉴：设计子技能协作时给数据流起一个名字
5. **双验证层** → `verify/SKILL.md`（LLM 验证）+ `write-validate.sh`（脚本验证）形成互补 → 借鉴：对关键写入操作，用脚本验证格式合规，用 LLM 验证语义合规

### 3.3 当时的缺陷
1. **9 个 skill-sources 缺 model 声明（BUG 级，PR-worthy）** → 为什么失败：声明了 `allowed-tools` 但没有 `model`，Claude Code 会退回到当前会话的环境模型，不同用户使用不同模型会导致行为不可预测 → 自查：gstack 54 个技能全部缺 model；bureau 7 个技能全部缺 model
2. **knowledge-guide.md 缺 allowed-tools（BUG 级）** → 为什么失败：智能体声称要读取 `reference/` 目录的文件，但没有在 frontmatter 中声明 `allowed-tools: Read`，实际上没有权限读 → 自查：我的智能体有没有声明 allowed-tools？
3. **read_config.sh KEY 参数注入风险（MEDIUM 安全）** → 为什么失败：`KEY` 直接拼接进 `grep -E "^${KEY}:"` 正则，若调用方传入特殊字符可导致意外匹配 → 自查：我的 hooks 脚本有没有类似模式

### 3.4 当时的优化机会
1. 为 9 个 skill-sources 文件批量加 `model: sonnet` 声明（BUG，直接影响功能）
2. knowledge-guide.md 加 `allowed-tools: Read` 并加 `## Output Format` 段落
3. read_config.sh 改用 `grep -F "${KEY}:"` 字面量匹配替代正则

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 9 个 skill-sources 缺 model | `grep -rn "^model:" skill-sources/*/SKILL.md` → 只有 8 个有 model | **8 个仍缺 model**（learn/ralph/reduce/refactor/reflect/rethink/reweave/verify 均缺） | BUG 未修，PR 未被合并 |
| knowledge-guide.md 缺 allowed-tools | `grep "allowed-tools" agents/knowledge-guide.md` → 无结果 | **持续**，allowed-tools 仍未声明 | 智能体功能受限 |
| read_config.sh KEY 注入 | `grep "KEY" hooks/scripts/read_config.sh` → L30 仍用 `-E "^${KEY}:"` | **持续**，安全建议未采纳 | 低风险，但未修 |

### 4.2 架构演进
主要演进：`agents/knowledge-guide.md` 现在有完整的 frontmatter（`name: knowledge-guide` + `model: sonnet`），audit 时仅标注了"有 frontmatter 但缺 allowed-tools"——说明 model 字段已被加上，但 allowed-tools 仍未加。`skill-sources/` 新增了 remember/seed/graph/stats/tasks/pipeline/validate/next 等文件的 `model: sonnet` 声明（8 个已修），但仍有 8 个未修。

### 4.3 新增的可学习模式
`generators/` 目录是 audit 时未深入分析的部分——这是一个"技能工厂"模式，通过生成器动态创建技能文件。这种元层级设计（用 AI 生成 AI 的工作规范）是高级 NL 编程模式，值得单独研究。

---

## 五、校准

### 5.1 我已经在做对的
1. **钩子用于知识积累**：bureau 的 PostToolUse 钩子思路与 arscontexta 的 auto-commit 钩子一致
2. **路径变量规范**：bureau 使用 `${CLAUDE_PLUGIN_ROOT}` 路径，与 arscontexta 对齐
3. **双层验证意识**：bureau 有 lint 技能做格式验证 + review 技能做语义验证，与 arscontexta 双验证层类似
4. **方法论文档化**：bureau 的 canon/ 目录类似 arscontexta 的 methodology/ 目录，都有持久化的方法论参考

### 5.2 挑战 / 验证
这个案例**挑战了**我"加 allowed-tools 是可选的"的假设。arscontexta 的 knowledge-guide 智能体声称读 `reference/` 文件，但实际上因为没有 `allowed-tools: Read` 而读不到——这不是 style 问题，是 **功能性 bug**。凡是技能/智能体需要读文件的，`allowed-tools: Read` 是必填，不是选填。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的智能体有没有声明 allowed-tools
grep -rn "allowed-tools" ~/projects/*/agents/*.md ~/projects/*/.claude/agents/*.md 2>/dev/null
```
命中后（无结果=未声明）：为所有需要读文件的智能体加 `allowed-tools: Read`

```bash
# 检查 skills 中缺 model 声明的文件
for f in ~/projects/MarkQWu-gstack/*/SKILL.md ~/projects/MarkQWu-bureau/skills/*/SKILL.md; do
  grep -q "^model:" "$f" || echo "MISSING model: $f"
done
```
命中后：为每个文件加 `model: sonnet`（默认）或 `model: opus`（需要深度推理的技能）

```bash
# 检查我的 hooks 脚本是否有变量直接拼接进命令
grep -rn '\$[A-Z_]*[^}"]' ~/.claude/hooks/*.sh 2>/dev/null | grep -v "#"
```
命中后：把 `grep -E "^${VAR}:"` 改成 `grep -F "${VAR}:"` 或加白名单校验

### 6.2 灵感 → 实施路径
1. **想法**：给 bureau 的所有技能批量加 model 声明
   - **为何可行**：arscontexta 的教训证明缺 model 是 BUG，不是风格问题；bureau 7 个技能全部缺 model
   - **第一步**：`sed -i '/^---$/a model: sonnet' bureau/skills/*/SKILL.md` → 然后手动把 guide 改成 opus（因为它需要深度推理）→ 15 分钟
2. **想法**：给 gstack 批量加 model 声明（54 个技能）
   - **为何可行**：arscontexta 有 27 个技能，batch 修复是可行的；gstack 54 个全部缺
   - **第一步**：写一个简单脚本批量在每个 SKILL.md 的 frontmatter 里加 `model: sonnet`，10 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：把 AI 会话沉淀为持久化知识库，通过钩子自动化，通过技能体系管理

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 同样是把 AI 会话转为知识库（capture→compile→review 流水线） | bureau 是"会话外知识管理"，arscontexta 是"会话内知识同步"；arscontexta 有钩子自动化，bureau 需手动触发 | 高 |
| MarkQWu/shiji-kb | 中 | 都是知识库管理 | shiji-kb 是静态知识图谱，arscontexta 是动态会话知识 | 中 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都从 AI 会话中挖掘价值 | echo-sleuth 挖掘过去会话，arscontexta 实时捕获 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill-sources 缺 model 声明（9/9） | `grep -rL "^model:" gstack/*/SKILL.md` | **gstack 54/54 全部命中，bureau 7/7 全部命中，echo-sleuth 4/4 全部命中** | 高（功能 BUG） |
| 智能体缺 allowed-tools | `grep -rL "allowed-tools" */agents/*.md` | 需本地验证 | 高（功能 BUG） |
| git add -A 范围过宽 | `grep "git add -A" **/hooks/**/*.sh` | 需本地验证 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack/` → 批量为所有 SKILL.md 加 `model: sonnet` → 10 分钟（脚本处理）
- `MarkQWu/bureau/skills/` → 同上 + 将 `guide/SKILL.md` 改为 `model: opus` → 15 分钟

### 8.3 别人的更优方案

1. **领域**：技能词汇表全局一致性
   - **本案例做法**：定义了 `${vocabulary.*}` 模板变量系统，27 个文件统一使用，NLPM 审计发现零词汇漂移
   - **我的项目现状**：bureau 各技能对"条目"的称呼不统一（有时叫 entry，有时叫 record，有时叫 note）
   - **如何借鉴**：在 `bureau/CLAUDE.md` 加一节"术语规范"，然后在各 SKILL.md 统一使用，逐步消除漂移

2. **领域**：双层验证（脚本 + LLM）
   - **本案例做法**：`write-validate.sh` 在每次写入后做格式校验，`verify/SKILL.md` 做语义校验
   - **我的项目现状**：bureau 的 lint 技能只做语义验证，没有脚本层格式校验
   - **如何借鉴**：为 bureau 加一个 `hooks/scripts/validate-entry.sh`，检查新增条目是否有必填字段

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skills 层已有 model 声明
- **我的做法**：bureau 的 skills 层（相对于 skill-sources 层）某些文件有 model 声明
- **本案例做法（弱在哪）**：arscontexta 的 skill-sources 层 8 个核心技能缺 model，这是 BUG 级问题（BUG 状态持续了 3 个月以上）
- **意义**：从这个案例可以看出：即使是高质量（96/100）的插件也会有 model 声明遗漏；这提醒我要写脚本定期检查，而不是靠记忆

---

## 八、术语表

### <a name="hook"></a>hook（钩子）
> Claude Code 在特定事件发生时自动触发的脚本。`SessionStart` 在会话开始时触发（arscontexta 用它来定向工作上下文），`PostToolUse(Write)` 在每次写入文件后触发（arscontexta 用它来自动提交笔记）。钩子是双刃剑：自动化程度高，但安全面也更宽。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置。对于 SKILL.md 文件，frontmatter 中的 `model` 字段告诉 Claude Code 用哪个推理层级，`allowed-tools` 字段声明这个技能可以使用哪些工具。没有这两个字段，系统会使用默认值，导致行为不可预测。

### <a name="三空间架构"></a>三空间架构
> Ars Contexta 的核心概念：`notes/`（知识原子，永久存储的命题）、`thinking/`（思考过程，临时推导）、`ops/`（系统操作，钩子和脚本产物）。三类内容有不同的写入规则、验证逻辑和提交策略。

### <a name="skill-sources"></a>skill-sources vs skills 双目录
> arscontexta 独特的文件组织方式：`skill-sources/` 存放原始技能文件（由作者手写），`skills/` 存放经过生成器处理后的最终版本（使用 `${CLAUDE_PLUGIN_ROOT}` 路径变量）。这种分离使得技能可以被"编译"——源文件保持可读性，生成文件针对运行时优化。

### <a name="模糊量词"></a>模糊量词（vague quantifier）
> 在技能文件里无法量化的描述词。arscontexta 中最典型的例子：`very similar content`（多相似算 very similar？）、`recent sessions`（多近算 recent？）。修复方式：改为 `≥80% 词汇重叠` 和 `7 天内的会话`。
