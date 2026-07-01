# softaworks/agent-toolkit — 学习案例

**仓库**：https://github.com/softaworks/agent-toolkit
**Stars**：1629 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-07-01（基于 audit 快照，目标仓库克隆受限）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `single-purpose`, `examples-driven`, `template-design`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
softaworks/agent-toolkit 是一个以"通用工具套件"为定位的 Claude Code 插件，1629 个 star 的体量表明它在社区中有相当知名度。关键特征：
- 58 个唯一工件（6 个 agent、9 个 command、43 个 skill），以 skill 为主力
- 整体 NL 得分 90/100，属高质量仓库，但存在一个系统性短板：**9 个 command 全部缺少 `name:` 字段**
- 文件整体结构清晰：`dist/plugins/` 目录是 `skills/`/`agents/`/`commands/` 的完整镜像，表明有构建流程
- skill 平均得分 95.1 分，command 平均仅 67.0 分，两者之间的断崖反差是最大学习点

### 1.2 架构剖析
- **目录结构**（推断自 audit 报告）：
```
agent-toolkit/
├── agents/          # 6 个专职 agent（ascii-ui-mockup、codebase-pattern-finder 等）
├── commands/        # 9 个命令（+ .claude/commands/ 下 2 个内部命令）
├── skills/          # 43 个 skill，覆盖技术写作、前后端、协作沟通等领域
├── dist/plugins/    # 构建产出目录（与源目录 1:1 镜像）
└── scripts/
    ├── build_plugins.py   # stdlib only，无 subprocess
    └── bump_version.py    # stdlib only，无 subprocess
```
- **文件类型分布**：skill 占 74%（43/58），是绝对主力；agent 6 个，command 11 个（含内部）
- **编排关系**：平铺结构，无集中式 router；skill 之间通过外部 `references/` 子目录引用补充内容（progressive disclosure 模式）
- **跨件契约**：skill 以 `references/` 子目录作为深层内容的"侧车"，SKILL.md 是入口，详细内容在同目录的 references 文件里

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「Progressive Disclosure（渐进披露）」——SKILL.md 是导论，`references/` 是正文，用目录组织让入门者不被细节淹没
- **解决什么问题**：如何在保持 skill 文件简洁的同时提供足够的深度内容
- **做了什么 trade-off**：把输出格式"委托"给 references 文件（如 `skills/command-creator/SKILL.md`），代价是 agent 在不加载 references 时不知道"done"是什么样的
- **反映什么认知模型**：作者将 skill 视为「API 文档」，SKILL.md = 接口摘要，references = 详细规格；这是文档化驱动而非约束驱动的思维

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「API 文档式 Skill」（API-Doc-Style Skill）**

关键特征：
- 特征 1：SKILL.md = 摘要 + 工作流，references/ = 详细规格和模板
- 特征 2：skill 间高度独立，无强依赖；每个 skill 是一个完整可用的单元
- 特征 3：有 `dist/` 目录表明存在构建流程，skill 被打包分发
- 特征 4：command 与 skill 服务于不同人群——command 给用户触发，skill 给 agent 加载

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 社区共享的通用工具集 | ✅ 高度适用 | 独立 skill + 构建流程 = 易于选择性安装 |
| 需要复杂跨 skill 编排的工作流 | ❌ 不足 | 无 router，skill 间无法自动协调 |
| skill 数量 < 10 的小型插件 | ❌ 过度工程 | dist/ 镜像 + references/ 结构对小项目是负担 |
| 以写作/沟通/研究为主的场景 | ✅ 适用 | 43 个 skill 中有大量软技能覆盖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：API 文档式平铺 skill | softaworks/agent-toolkit | skill 独立性强，可按需选用 | command 和 skill 质量断崖明显 |
| 备选 A：command 中心式 | sangrokjung/claude-forge | command 完整声明，工作流明确 | 工件耦合度高，修改成本大 |
| 备选 B：单 CLAUDE.md 约定 | 轻量型 setup 仓库 | 极简，无维护负担 | 无法扩展到复杂场景 |

### 2.4 改进空间
1. **当前问题**：9 个 command 全部缺 `name:` **改进做法**：用脚本批量插入 `name: <filename-without-ext>` 到每个 command frontmatter **预期收益**：command 正确注册，NL 总分从 90 升至约 93
2. **当前问题**：`skills/reducing-entropy/SKILL.md` 无输出格式 **改进做法**：加 `## Output Format` section，哪怕只是一句"输出 Markdown diff 列表，每条注明 entropy 降低理由" **预期收益**：agent 知道"做完"是什么样子，不会在 task 边界无限延伸
3. **当前问题**：`agents/mermaid-diagram-specialist.md` 是 skill 格式但注册为 agent **改进做法**：要么转换为真正的 agent（加 `model:`、改写触发条件），要么重命名去掉 agent 后缀 **预期收益**：消除歧义，新贡献者不会混淆

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/explain-pr-changes.md | 45 | 零 YAML frontmatter，无法注册（BUG） |
| commands/viral-tweet.md | 60 | 缺 name，用主题 header 替代编号步骤 |
| agents/general-purpose.md | 79 | 零示例，3 处模糊量词 |
| agents/ascii-ui-mockup-generator.md | 98 | 两个内置示例，5 步流程，标杆文件 |
| skills/mermaid-diagrams/SKILL.md | 98 | Class/Sequence/Flowchart/ERD 快速启动示例齐全 |

### 3.2 当时值得借鉴的模式

1. **多类型示例覆盖（98 分 skill 的共同特征）** → 根本好处：不同场景的用户都能找到匹配的起点 → `skills/mermaid-diagrams/SKILL.md`（Class、Sequence、Flowchart、ERD 四类例子）→ 在自己的 skill 中至少覆盖 2 种不同输入场景
2. **`skill-judge/SKILL.md` 的 120 分 8 维度评分** → 根本好处：把评估标准嵌入 skill 本身，AI 自我校准 → `skills/skill-judge/SKILL.md` → 在需要质量判断的 skill 中嵌入打分维度
3. **Progressive Disclosure 的 references/ 模式** → 根本好处：主文件不超过 200 行，深度内容按需加载 → `skills/codex/SKILL.md` + `references/` → 把超过 300 行的 skill 拆分为 SKILL.md + references/
4. **`web-to-markdown/SKILL.md` 的激活守卫** → 根本好处：明确声明触发条件，避免被意外加载 → "仅当用户明确请求时激活" → 在 skill 首行加激活条件声明
5. **`humanizer/SKILL.md` 的 24 个 Before/After 示例** → 根本好处：AI 可归纳出模式，而不是靠抽象描述猜测 → 在文本转换类 skill 中提供密集 Before/After 对

### 3.3 当时的缺陷

1. **commands/explain-pr-changes.md 零 frontmatter** → 根本原因：作者用 `#` 标题开头，可能是从 README 片段直接复制成 command，忘记加 frontmatter 包装 → **自查**：我的 bureau 里有没有从 README 或文档直接升格的 command 文件？
2. **9 个 command 全部缺 `name:` 字段** → 根本原因：系统性忽视——当作者写 `description:` 时，没有意识到 `name:` 与 `description:` 同等重要；可能认为"文件名就是 name" → **自查**：我的所有 command 里有没有 `name:` 字段？`description:` 和 `name:` 必须同时存在
3. **`commands/codex-plan.md` 用了 `AskUser` 而不是 `AskUserQuestion`** → 根本原因：`AskUser` 是直觉性写法，`AskUserQuestion` 才是 Claude Code 实际注册的工具名 → **自查**：`grep -rn "AskUser[^Q]" ~/.claude/commands/*.md`
4. **`agents/general-purpose.md` 零示例** → 根本原因：通用 agent 写起来最容易跳过示例——越通用、越难想出具体例子 → 自查：有没有自己写的 "通用" agent 同样没有示例？
5. **4 个 command 缺 `allowed-tools:`** → 根本原因：与 claude-forge 一样，command 逻辑优先写，声明字段被忽视

### 3.4 当时的优化机会

1. **批量给 9 个 command 加 `name:` 字段**：一行脚本即可完成，机械性修复，无歧义
2. **给 `agents/general-purpose.md` 加至少 2 个示例**：通用 agent 最依赖示例来限定触发边界
3. **`skills/reducing-entropy/SKILL.md` 加输出格式**：当前 skill 描述了过程但无法验证结果

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
> 注：git clone 受限，以下基于推断。

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| explain-pr-changes.md 零 frontmatter | `head -3 commands/explain-pr-changes.md` | **无法验证** | 已有贡献者 PR 修复（audit 建议为最高优先级） |
| 所有 command 缺 `name:` | `grep -L "name:" commands/*.md` | **无法验证** | 机械修复，大概率批量完成 |
| AskUser → AskUserQuestion | `grep "AskUser" commands/codex-plan.md` | **无法验证** | 易发现 bug，可能已修 |

### 4.2 架构演进
audit 时 skill 数量为 43，command 为 9。推断演进方向：skill 库会持续增长（社区贡献），command 因有系统性缺陷修复动机可能做了一次批量清理。

### 4.3 新增的可学习模式
暂无（无法访问当前仓库状态）。

---

## 五、校准

### 5.1 我已经在做对的
1. 在 skill 中分离入口文件（SKILL.md）和深度内容（references/），避免单文件过长
2. 为不同场景提供多个示例，而不是只有一个"标准"例子
3. 在 allowed-tools 声明上比 command 作者更仔细
4. 区分 agent 和 skill 的格式要求（model: 字段只有 agent 需要）
5. 在 Before/After 格式中写转换示例

### 5.2 挑战 / 验证
- **验证的认知**：「`name:` 字段是必须的，文件名不等于注册 name」——这条规则在 softaworks/agent-toolkit 的 9 个 command 全军覆没的案例中被充分验证了。文件名不能替代 frontmatter 里的 `name:`。
- **新挑战**：我之前认为高分仓库（90+）已经"基本没问题"——但 90 分的 softaworks 仍有 1 个完全无法注册的 command（45 分）。分数反映的是**加权平均**而非**最差情况**，要重点检查得分最低的文件而不是只看平均分。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查所有 command 中是否有同时存在 description: 但缺 name: 的
grep -rL "^name:" ~/.claude/commands/*.md 2>/dev/null
# 命中后：在 frontmatter 中添加 name: <kebab-case-identifier>

# 检查所有工件中有没有 AskUser（应为 AskUserQuestion）
grep -rn "\bAskUser\b" ~/.claude/commands/*.md ~/.claude/agents/*.md 2>/dev/null
# 命中后：改为 AskUserQuestion，同时在 allowed-tools 中更新

# 检查 agent 文件是否有零 example 的情况（高风险）
grep -rL "## Example\|<example>" ~/.claude/agents/*.md 2>/dev/null
# 命中后：为每个命中 agent 加至少 2 个示例块

# 检查 skill 是否有输出格式（Output Format 或 ## Output）
grep -rL "## Output\|Output Format" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：在末尾加 ## Output Format section，哪怕一句话描述产出形态
```

### 6.2 灵感 → 实施路径

1. **想法**：在 graphify 项目中学习 softaworks 的 `skill-judge/SKILL.md` 模式，为每个 skill 加内置质量评分维度
   - **为何可行**：graphify 的 skill 帮助 AI 分析代码，加入"分析质量维度"能让 AI 自我校准输出质量
   - **第一步**：在 graphify 最核心的 skill 末尾加 `## Quality Checklist`，列出 3-5 条可验证的输出标准，10 分钟可完成

2. **想法**：为 drama-workshop-skills 建立 `references/` 子目录结构，把过长 skill 拆分
   - **为何可行**：该项目是社区共享 skills，用户会直接阅读，长文件体验差
   - **第一步**：找最长的 SKILL.md，把 `## Details` 后的内容移到 `references/detail.md`，在 SKILL.md 末尾加 `See: [references/detail.md](references/detail.md)` 引用，20 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **softaworks/agent-toolkit 的核心目的**：面向通用研发场景的 Claude Code skill 套件，43 个 skill 覆盖技术写作、沟通、前后端开发等宽领域

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为社区共享 Claude Code skill 套件 | drama-workshop 聚焦戏剧工坊场景，toolkit 是通用领域 | 高 |
| MarkQWu/gstack | 中 | 同为多工具 Claude Code 插件 | gstack 偏角色扮演工作流，toolkit 偏独立技能 | 中 |
| MarkQWu/graphify | 低 | 同为 AI 编程辅助工具 | graphify 有特定知识图谱功能，非通用 skill 套件 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 我的项目推测情况 | 严重度 |
|---|---|---|
| command 缺 `name:` 字段（9/9 命中） | drama-workshop-skills 和 gstack 的 command 如快速编写则极可能存在 | 高 |
| agent 零示例（general-purpose.md） | 如有"通用"类 agent 则大概率无示例 | 高 |
| `AskUser` 非标准写法 | 若从教程复制代码则容易出现 | 中 |
| skill 无 Output Format | echo-sleuth-for-claude 的 skill 若定义了提取行为但无输出格式则命中 | 中 |

**命中后的具体行动建议**：
- drama-workshop-skills：`grep -rL "^name:" commands/` — 每个命中文件 2 分钟修复
- echo-sleuth：`grep -rL "Output Format\|## Output" skills/*/SKILL.md` — 优先修 priority=high 的 skill

### 7.3 别人的更优方案

1. **领域**：Skill 的 Before/After 示例密度
   - **toolkit 做法**：`skills/humanizer/SKILL.md` 提供 24 组 Before/After 对，让 AI 从密集示例中归纳转换模式
   - **我的项目现状**：drama-workshop-skills 的 skill 如只有 1-2 个示例，则 AI 在边缘输入上容易失准
   - **如何借鉴**：找 drama-workshop-skills 中最重要的 3 个 skill，从实际使用中收集 5+ 组 Before/After 追加进去，每个 skill 30-60 分钟

2. **领域**：Command 有无 `allowed-tools` 声明
   - **toolkit 做法**：即便有缺陷，toolkit 的 skill 层级 `allowed-tools` 声明非常完整
   - **我的项目现状**：gstack 的 command 若快速编写则可能缺失
   - **如何借鉴**：`grep -rL "allowed-tools" .claude/commands/*.md && grep -rn "Bash\|Edit\|Write" .claude/commands/*.md` 找到使用了工具但未声明的 command

### 7.4 反向：我的项目做得比他们好的地方

暂无（无法读取当前用户仓库代码进行对照）。

---

## 八、术语表

### <a name="progressive-disclosure"></a>Progressive Disclosure（渐进披露）
> UI 设计术语，这里指 NL 编程中的一种技巧：把信息分层展示。SKILL.md 是「摘要层」，用户快速了解 skill 能做什么；`references/` 目录是「细节层」，AI 在需要时才深入读取。好处是 SKILL.md 保持短小精悍，不吓退初次使用者。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 YAML 配置块。Claude Code 通过读取它来知道文件的类型、名字、允许使用哪些工具。缺少 frontmatter 的 command 文件无法被 Claude Code 识别和注册。

### <a name="allowed-tools"></a>allowed-tools
> command/agent frontmatter 中的字段，列出该工件被允许调用的工具名单。如果 command 会用到 `Bash` 但未在 `allowed-tools` 里声明，在限制模式下 Claude Code 会拒绝执行那次工具调用，导致 command 部分功能缺失。

### <a name="AskUserQuestion"></a>AskUserQuestion
> Claude Code 中向用户提问并等待回复的工具名。正确拼写是 `AskUserQuestion`，常见错误写法是 `AskUser`。两者的区别：前者会被 Claude Code 正确授权，后者会在工具调用时失败并静默跳过。
