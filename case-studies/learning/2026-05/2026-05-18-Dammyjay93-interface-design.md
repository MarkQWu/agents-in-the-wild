# Dammyjay93/interface-design — 学习案例

**仓库**：https://github.com/Dammyjay93/interface-design
**Stars**：4630 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-05-18（基于当前 HEAD）
**主题标签**：cross-reference, template-design, manifest-discipline, single-purpose, vague-quantifier

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
这是一个专注于「界面设计工程」的 Claude Code 插件，4630 颗星使其成为同类 UI 方向插件中最受关注的之一。作者 Damola Akinleye 将插件定位为设计工程师的思维伙伴，而非代码生成器——它帮助 Claude 在整个项目生命周期内维持设计意图一致性。仓库极其简洁：7 个 NL artifact，零脚本，零 hook，零 MCP 配置。

### 1.2 架构剖析

```
interface-design/
├── .claude/
│   ├── commands/           # 5 个 command（audit, critique, extract, init, status）
│   └── skills/
│       └── interface-design/
│           ├── SKILL.md    # 主技能文件（极其详尽）
│           └── references/ # 深度参考材料（audit 后新增！）
│               ├── principles.md
│               ├── validation.md
│               ├── critique.md
│               └── example.md
├── .claude-plugin/
│   └── plugin.json
├── reference/              # 参考示例（非 Claude Code 核心路径）
└── website/                # 配套网站
```

- **文件类型分布**：0 个 agent / 5 个 command / 1 个 skill（极简设计）
- **编排关系**：`init.md` 是唯一入口 command，加载 SKILL.md 到上下文；其他 4 个 command（audit/critique/extract/status）围绕 SKILL.md 中建立的 `system.md` 记忆文件操作，形成「加载 → 记忆建立 → 循环检查/改进」的生命周期
- **跨件契约**：所有 command 通过 `system.md`（项目本地文件）传递设计决策状态；SKILL.md 是唯一知识源，command 是其操作层

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「意图持久化」——通过 `system.md` 这一跨会话记忆载体，让设计决策不随上下文压缩而消失
- **解决什么问题**：LLM 的无状态性导致设计风格在长项目中发生漂移；这个插件用结构化记忆文件对抗默认值侵蚀
- **Trade-off**：极度专注（只做界面设计）vs 通用性缺失——SKILL.md 明确写道「不用于 landing page/营销页，重定向到 `/frontend-design`」，但 `/frontend-design` 这个命令并不在此插件中，依赖用户额外安装
- **认知模型**：作者认为 AI 产出「正确」（correct）与「精心设计」（crafted）之间存在鸿沟，并相信这道鸿沟可以通过强迫 AI 在生成代码前先做设计审视来弥合——这是一种「强制减速」的 AI 使用哲学

---

## 二、过去审查发现（2026-04-27 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-27 当时得分 **94/100**，Security: CLEAR，0 个安全发现。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| SKILL.md | 98/100 | 模糊量词 "some"（-2） |
| .claude/commands/critique.md | 85/100 | 缺 empty input handling（-10）+ 缺 allowed-tools（-5） |
| .claude/commands/audit.md | 95/100 | 缺 allowed-tools（-5） |
| .claude/commands/extract.md | 95/100 | 缺 allowed-tools（-5） |
| .claude/commands/init.md | 95/100 | 缺 allowed-tools（-5） |
| .claude/commands/status.md | 95/100 | 缺 allowed-tools（-5） |
| .claude-plugin/plugin.json | 95/100 | CalVer 版本格式（信息性） |

### 2.2 当时值得借鉴的模式

1. **意图先于代码**（Intent First 章节）→ SKILL.md 要求 Claude 在写第一行代码前必须回答「用户是谁？他们在做什么？他们会感受到什么？」。这迫使生成结果锚定人类体验而非技术惯例 → 适用于任何 UI 生成型 skill：将设计问题前置，代码后置
2. **「默认值隐藏在哪里」的明确化**（Where Defaults Hide 章节）→ SKILL.md 系统性地列举了 typography、navigation、data 三类「看起来是实现细节实际上是设计决策」的区域，并解释为什么 AI 会在这里滑向默认值 → 这是元认知能力的文档化——告诉 AI「你容易在哪里偷懒」
3. **跨会话记忆文件（system.md 模式）** → 通过在用户项目中维护一个 `system.md` 记录设计签名（design signature）、调色板、字体选择、间距原则，实现跨会话设计一致性 → 可直接借鉴到任何需要跨会话持久化决策的插件场景
4. **命令专职化**：5 个 command 各司其职（init 初始化 / audit 验证合规 / critique 主观审视 / extract 提取模式 / status 查看记忆），无功能重叠 → 符合单职责原则，每个命令的使用场景清晰

### 2.3 当时的缺陷

1. **3 个 broken reference**：SKILL.md 第 381-383 行指向 `references/principles.md`、`references/validation.md`、`references/critique.md`，但这三个文件不存在。根本原因：作者在写 SKILL.md 时规划了「深度参考」功能，但没有同步创建文件，形成 dead link。**自查：我的 SKILL.md 中有没有引用但未创建的文件？**
2. **所有 5 个 command 缺 `allowed-tools`**：命令读取/写入文件但未声明工具访问权限。根本原因：作者未来得及熟悉 Claude Code 的 frontmatter schema 完整要求，或认为默认权限已经足够。**自查：我的 command 是否全部声明了 `allowed-tools`？**
3. **`critique.md` 缺 empty input handling**：该命令假设用户刚刚完成了一个 UI 构建，但没有说明如果没有最近构建上下文时应该如何处理。根本原因：作者在设计 command 时默认了一个特定工作流场景，忽略了边缘调用情况。**自查：我的 command 有没有说明「如果没有输入/上下文时做什么」？**

### 2.4 当时的优化机会

1. 在所有 5 个 command frontmatter 中添加 `allowed-tools`：audit → `[Read, Glob]`；critique → `[Read, Edit, Write]`；extract → `[Read, Glob, Write]`；init → `[Read, Write]`；status → `[Read]`
2. 创建或内联 references/ 文件：将 SKILL.md 中承诺的深度参考材料落实为实际文件，或改为内联内容
3. 在 critique.md 开头添加：「如果没有最近完成的 UI 文件在上下文中，询问用户指定要审视的文件路径」

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 3 个 broken reference（principles/validation/critique） | `ls .claude/skills/interface-design/references/` | **已修复**：references/ 目录现在存在，包含 principles.md、validation.md、critique.md、example.md 共 4 个文件 | 最高优先级 bug 在 3 周内完成修复，说明作者响应了反馈或自己发现了问题 |
| 5 个 command 缺 allowed-tools | `grep "allowed-tools" .claude/commands/*.md` | **仍缺失**：5 个 command 全部没有 `allowed-tools` 声明 | 质量类问题修复优先级低于功能性 bug；但这也意味着功能本身可以运行，未修复不影响用户 |
| critique.md 缺 empty input handling | 读取 critique.md 开头 | **仍缺失**：命令依然假设存在最近的 UI 构建上下文 | 同上，作者选择性修复影响功能的 bug，忽略边缘情况处理 |

### 3.2 架构演进

audit 时 `references/` 目录根本不存在，现在已包含 4 个文件（还多出了一个 `example.md`）。这是直接的内容补全，而非架构重组。`reference/` 目录（仓库根目录，非 Claude 插件路径）下也出现了 `examples/system-warmth.md` 和 `examples/system-precision.md`——这两个文件展示了「温暖系」和「精准系」两种不同设计签名的 system.md 示例，说明作者在拓展教学材料，帮助用户理解如何初始化 system.md。

### 3.3 新增的可学习模式

`references/example.md` 是 audit 时不存在的新增内容。它提供了一个完整的 `system.md` 示例（具体的 bakery management tool 设计决策），作为 init 命令的参考输出。这是「输出物示例化」模式的延伸——不只是 SKILL.md 有示例，连深度参考文档也提供了具体输出样本。

---

## 四、校准

### 4.1 我已经在做对的
- 跨会话记忆机制：我已经有类似 system.md 的项目状态文件设计理念
- 命令专职化：我的 command 设计遵循单职责，功能不重叠
- 明确指出「不适用场景」：像 SKILL.md 的 Scope 章节一样，我的 skill 也有明确的适用边界说明
- Bug 优先修复策略：这次对比验证了「先修 bug，再修质量问题」的优先级顺序是真实的

### 4.2 挑战 / 验证

这次 audit 验证了一个之前不确定的做法：**在 SKILL.md 中引用不存在的深度参考文件是否值得**。Dammyjay 的案例说明「先声明再实现」的写法是可接受的——用 audit 工具发现它、然后修复，这本身构成一个有效的开发循环。更有趣的是：`example.md` 的增加（audit 时没有规划的文件）说明作者在填补 references/ 时超额完成，加入了一个系统示例。这验证了「承诺 + 外部审视 → 超额交付」的心理机制对开源作者同样有效。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查我的 SKILL.md 中的 references 是否都实际存在
grep -oE '\[.*?\]\((.*?)\)' ~/.claude/skills/*/SKILL.md 2>/dev/null | \
  grep -oE '\(.*?\)' | tr -d '()' | while read path; do
    [ -f "$path" ] || echo "DEAD LINK: $path"
  done
# 命中后：创建对应文件，或将引用改为内联内容
```

```bash
# 检查我的 skill 是否有描述「不适用场景」的 Scope 章节
grep -rL "Not for:\|不适用\|适用范围" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：为该 skill 添加明确的适用/不适用边界声明
```

### 5.2 灵感 → 实施路径

1. **想法**：借鉴 system.md 模式，为我的代码风格 skill 建立「风格签名文件」
   - **为何可行**：Dammyjay 的 4630 颗星证明了「跨会话记忆 + 一致性维护」是用户真实痛点，代码风格领域同样存在漂移问题
   - **第一步**：在我最常用的项目根目录创建 `.claude/style.md`，记录：命名约定、注释风格、错误处理模式；将 skill 的第一步改为「读取 .claude/style.md（如存在）并在后续所有建议中遵循」；30 分钟可完成

2. **想法**：为 critique 类 command 补充「无上下文时的降级路径」
   - **为何可行**：Dammyjay 的 critique.md 被扣 10 分正是因为缺少此处理——这个问题普遍存在于假设「用户刚完成某事」的 command 中
   - **第一步**：在 critique.md（或类似命令）开头加一段：「如果对话中没有最近生成的文件，先用 Glob 工具找出最近修改的文件，列给用户选择，再继续」；10 分钟可完成
