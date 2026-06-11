# google-labs-code/stitch-skills — 学习案例

**仓库**：https://github.com/google-labs-code/stitch-skills
**Stars**：4,860 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-11（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `vague-quantifier`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`stitch-skills` 是 Google Labs 为其 AI UI 设计工具 **Google Stitch**（`labs.google.com/stitch`）出品的官方 Claude Code skill 集合。Stitch 是一个把自然语言转化为 UI 设计稿的 AI 工具，而这些 skill 是「如何用 AI 编程工具驱动 Stitch」的操作手册。

关键事实：
1. Google Labs 官方出品，围绕单一外部工具（Stitch）构建，定位清晰
2. 当前 14 个 SKILL.md（审计时 8 个），新增 react-native 等技术栈
3. 仓库结构从审计时的「平铺 skills/」演进为「3 个独立 plugin 包」架构
4. 4,860 stars，是本批次最高关注度仓库

### 1.2 架构剖析

```
plugins/
├── stitch-design/      # 设计工作流（代码→设计、生成屏幕、管理设计系统等）
│   ├── plugin.json     # 独立插件清单
│   └── skills/
│       ├── code-to-design/SKILL.md
│       ├── extract-design-md/SKILL.md
│       ├── extract-static-html/SKILL.md
│       ├── generate-design/SKILL.md
│       ├── manage-design-system/SKILL.md
│       └── upload-to-stitch/SKILL.md
├── stitch-utilities/   # 辅助工具（设计规范文件、循环工作流等）
│   ├── plugin.json
│   └── skills/
│       ├── design-md/SKILL.md      # 生成 DESIGN.md
│       ├── enhance-prompt/SKILL.md # 提示词优化
│       ├── stitch-loop/SKILL.md    # 持续迭代循环
│       └── taste-design/SKILL.md  # 语义设计系统生成
└── stitch-build/       # 前端构建（React、React Native、Remotion、shadcn/ui）
    ├── plugin.json
    └── skills/
        ├── react-components/SKILL.md
        ├── react-native/SKILL.md   # 新增
        ├── remotion/SKILL.md
        └── shadcn-ui/SKILL.md
```

- **文件类型分布**：14 个 SKILL.md，3 个 plugin.json，4 个 shell/JS scripts，1 个 package.json
- **编排关系**：3 个 plugin 相互独立，用户按需安装。stitch-design 负责「创造设计」，stitch-utilities 负责「维护设计规范」，stitch-build 负责「把设计变成代码」。stitch-loop 是三者的粘合剂（持续迭代循环）
- **跨件契约**：多数 skill 通过 `stitch*:*` glob 模式声明 Stitch MCP 工具访问权限，但 taste-design 仍使用字面量 `StitchMCP`（潜在不一致）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「每个 plugin 是独立的功能单元」——用户只安装需要的 plugin，不强迫全量安装
- **解决什么问题**：AI 工具（Claude Code）与设计工具（Stitch）之间的协作断点——开发者在写代码时需要在两个工具间切换
- **Trade-off**：多 plugin 架构增加了发现成本（用户需要找到对的 plugin），但降低了依赖爆炸（不需要 React 技术栈的用户不用安装 stitch-build）
- **认知模型**：把 Stitch MCP 工具视为「设计层 API」，skill 是「如何操作这个 API」的操作手册，与 gemini-skills 的 API 知识库模式高度相似

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「领域分层多 plugin 架构」（Domain-Layered Multi-Plugin）**

关键特征：
- 按「设计 → 辅助 → 构建」职责将 skill 分组到 3 个独立 plugin
- 每个 plugin 有独立的 plugin.json（可单独安装）
- skill 内 allowed-tools 全覆盖（14/14 声明 allowed-tools）
- 核心 MCP 工具用 glob 模式声明（`stitch*:*`），兼容工具名前缀变化

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 围绕单一外部服务构建多 skill 集合 | ✅ 高度适用 | 多 plugin 分层让用户按需选择，设计清晰 |
| 功能差异明显的 skill 需要分组管理 | ✅ 适用 | 「设计/辅助/构建」三个维度正交 |
| 只有 2-3 个 skill 的小型项目 | ❌ 过度设计 | plugin 架构增加维护开销 |
| 需要 skill 之间强引用的复杂编排 | ❌ 不完全适用 | plugin 间独立，跨 plugin 协调靠 stitch-loop |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 领域分层多 plugin | stitch-skills | 按需安装，职责清晰，allowed-tools 全覆盖 | 发现成本高，stitch-loop 跨 plugin 协调靠约定 |
| 单仓所有 skill | Vibe-Skills | 安装简单，skill 间可引用 | 大量 skill 缺 allowed-tools，编排层薄弱 |
| 精简单一职责 | gemini-skills | 质量极高，即插即用 | 覆盖面窄 |

### 2.4 改进空间

1. **当前问题**：taste-design 仍用 `StitchMCP` 而非 `stitch*:*`，与其他 13 个 skill 不一致。**改进做法**：把 taste-design/SKILL.md 中 `allowed-tools: - "StitchMCP"` 改为 `- "stitch*:*"`。**预期收益**：消除跨组件不一致，防止 Stitch MCP 工具名变化时只有这一个 skill 失效。
2. **当前问题**：react-components 和 shadcn-ui 的 allowed-tools 中有 `web_fetch`，但 skill body 中没有任何地方调用它（网络操作全通过 shell scripts 完成）。**改进做法**：从 allowed-tools 中删除 `web_fetch`，工具声明与实际使用对齐。**预期收益**：最小权限原则，降低工具滥用风险。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

得分 **96/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| taste-design/SKILL.md | 99 | 无显著问题 |
| stitch-loop/SKILL.md | 98 | 「if appropriate」模糊量词 |
| enhance-prompt/SKILL.md | 98 | 「appropriate」模糊量词 |
| design-md/SKILL.md | 97 | Overview 和 The Goal 内容近乎重复 |
| remotion/SKILL.md | 96 | 「appropriate」出现两次（timing + font sizes）|
| react-components/SKILL.md | 94 | 无 Output Format section；`web_fetch` 声明但未使用 |
| shadcn-ui/SKILL.md | 94 | 同上 |
| stitch-design/SKILL.md | 88 | 无 output format；「intelligently route」模糊 |

### 3.2 当时值得借鉴的模式

1. **allowed-tools 全覆盖（当时 5/8 声明）** → 为什么好：明确工具边界，防止 AI 随意使用未声明工具。路径：`skills/stitch-loop/SKILL.md`（当时已声明 `stitch*:*`）。如何借鉴：每个 skill 都声明 allowed-tools，哪怕是空集（表示不需要外部工具）。
2. **stitch*:* glob 模式** → 为什么好：用 glob 而不是具体工具名，当 MCP 工具名前缀变化时 skill 无需修改。路径：`skills/stitch-loop/SKILL.md`。如何借鉴：凡是声明 MCP 工具的 allowed-tools，用 `<prefix>*:*` 格式。
3. **skill 职责边界清晰** → react-components 只管「把 Stitch 设计稿变成 React 组件」，remotion 只管「制作 Remotion 视频」，单一职责做到了极致。如何借鉴：一个 skill 只做一件事，所有「顺便做的」功能分出去。

### 3.3 当时的缺陷

1. **问题**：stitch-design/SKILL.md 无 Output Format section，作为路由 skill 不知道它产出什么。**根本原因**：路由 skill 的作者把路由逻辑（分发给哪个 workflow）当成了唯一职责，忽视了用户需要知道最终产出物是什么。**自查**：我的 echo-sleuth 命令文件是否声明了 output 格式？→ 命令文件有步骤但没有显式 output 声明节。
2. **问题**：remotion/SKILL.md 引用了 `examples/walkthrough/` 目录，但该目录不存在。**根本原因**：重构目录结构时只更新了文件，忘记更新文档中的路径引用。**自查**：我的 skill 有没有引用不存在的相对路径？→ 用 `grep -rn "references/\|examples/"` 检查再逐一验证。
3. **问题**：stitch-design 和 taste-design 用 `StitchMCP` 而非 `stitch*:*`，与其他 skill 不一致。**根本原因**：不同 skill 由不同作者或在不同时间编写，没有统一的工具声明约定。**自查**：我的项目不同 skill 的 allowed-tools 格式是否一致？→ 需检查 claude-for-legal 的多个 skill。

### 3.4 当时的优化机会

1. 修复 remotion/SKILL.md 中 `examples/walkthrough/` → `examples/`（路径修复，1 行改动）
2. 统一 `StitchMCP` → `stitch*:*`（stitch-design 和 taste-design）
3. 为 stitch-design 添加 Output Format section

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| remotion `examples/walkthrough/` 路径错误 | `grep -rn "examples/walkthrough"` | **已修复** ✓ — 全仓库未找到该路径引用 | 最明显的 bug 已被修复 |
| `StitchMCP` vs `stitch*:*` 不一致 | `grep -rn "StitchMCP"` | **部分残留** — taste-design 仍用 `StitchMCP`；原有 stitch-design 已改用 `stitch*:*` | 从 2 个不一致减少到 1 个，有改善但未完成 |
| 所有 skill 缺少 allowed-tools | `grep -rL "allowed-tools"` in SKILL.md | **已全面修复** ✓ — 14/14 SKILL.md 均有 allowed-tools 声明 | 这是此次演进最重要的改进 |

### 4.2 架构演进

最重大变化：**从单层 `skills/` 目录演进为 3 个独立 plugin 包（stitch-design / stitch-utilities / stitch-build）**。这是一次显著的「模块化重构」。意味着维护者意识到：随着 skill 数量增长，平铺结构会让用户找不到需要的 skill，而按「设计/辅助/构建」分组更符合用户工作流。新增 react-native skill，说明移动端开发是增长方向。

design-md 和 taste-design 现在都有完整的 `## Output Format (DESIGN.md Structure)` section（审计时缺失），这与 stitch-design 的 output format 修复是一脉相承的。

### 4.3 新增的可学习模式

- **每个 plugin 有独立的 plugin.json**：允许用户只安装需要的功能（只用设计功能就装 stitch-design，不需要 React 技术栈就不装 stitch-build）。这是「按需加载」的架构思路，对于大型插件集合非常实用。
- **allowed-tools 全面覆盖**：从审计时部分覆盖到当前 100% 覆盖，说明维护团队把「工具声明规范化」作为一次专门的技术债偿还。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责原则**：echo-sleuth 的每个 agent/skill 只做一件事（memory-management 不管 git-mining），与 stitch-skills 一致。
2. **Commands 有 allowed-tools**：echo-sleuth/commands/ 的工具声明比审计时的 stitch-skills 更规范。
3. **frontmatter 完整**：echo-sleuth 的 SKILL.md 有 name + description，符合规范。

### 5.2 挑战 / 验证

本案例验证了一个实践：**技术债要单独集中偿还，而不是混在功能迭代里**。stitch-skills 的演进中，allowed-tools 从部分覆盖到全面覆盖，明显是一次专门的「补全 allowed-tools」commit，而不是在添加新功能时顺手修的。这种专注的技术债偿还方式效果更好——一次把所有 skill 都修了，而不是修了几个、漏了几个。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否都有 allowed-tools（参考 stitch-skills 100% 覆盖目标）
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ -name "SKILL.md" | \
  xargs grep -L "allowed-tools"
find /tmp/my-repos/MarkQWu-claude-for-legal/ -name "SKILL.md" | \
  xargs grep -L "allowed-tools" | head -20
# 命中后：为每个 skill 根据 body 中实际调用的工具声明 allowed-tools
```

```bash
# 检查我的 skill 中 allowed-tools 声明的工具是否真的在 body 中用到（ghost tool 问题）
# 手工检查：看 allowed-tools 里的工具，在 skill body 里是否有对应操作
grep -A2 "allowed-tools:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
# 命中 web_fetch/fetch_url 但 body 中无网络操作后：删除该工具声明
```

```bash
# 检查 skill 中是否有引用不存在路径
grep -rn "references/\|examples/\|resources/" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null
# 对每个路径用 ls 验证是否实际存在
```

### 6.2 灵感 → 实施路径

1. **想法**：将 claude-for-legal 按「监管/公司/诉讼」重组为 3 个子 plugin（对应 stitch-skills 的分层模式）
   - **为何可行**：claude-for-legal 已有明显的子域分组（regulatory-legal、corporate-legal、litigation-legal），与 stitch-skills 的设计/辅助/构建三层对应
   - **第一步**：创建 3 个 plugin.json，每个指向对应子域的 skills/ 目录，不改动任何 skill 内容，30 分钟

2. **想法**：为 echo-sleuth 的所有 SKILL.md 做一次「allowed-tools 集中补全」
   - **为何可行**：4 个 SKILL.md 全部缺失，是一次小而专注的技术债偿还（类比 stitch-skills 的补全策略）
   - **第一步**：逐一打开 skills/*/SKILL.md，看 body 中有哪些工具操作，对应补充 allowed-tools，4 个文件约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：为 Google Stitch AI 设计工具提供 Claude Code 集成 skill，覆盖设计→构建全链路
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是「围绕单一专业领域 + 多个垂直 skill + 分层组织」模式 | stitch-skills 围绕外部 AI 工具；claude-for-legal 是领域知识库 | 高（架构模式） |
| MarkQWu/echo-sleuth-for-claude | 低 | 都是 Claude Code 插件，都有 SKILL.md | 目的不同，规模不同 | 中（allowed-tools 规范） |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件 | 无明显重叠 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 allowed-tools | `grep -rL "allowed-tools" skills/*/SKILL.md` | echo-sleuth: 4/4 命中；claude-for-legal regulatory skill 需检查 | 中 |
| allowed-tools 中有「幽灵工具」（声明但不用）| 手工核对 allowed-tools 与 body | echo-sleuth: commands 有 `Task` 工具，需确认 audit.md 是否真的用了 Task | 低 |
| skill 引用不存在的路径 | `grep -rn "references/"` + 逐一 ls 验证 | 未发现明显问题（echo-sleuth skill 没有 references 子目录引用） | N/A |

**命中后的具体行动建议**：
- echo-sleuth/skills/memory-management/SKILL.md → 添加 `allowed-tools:\n  - Read\n  - Write\n  - Edit`，5 分钟
- echo-sleuth/skills/git-mining/SKILL.md → 添加 `allowed-tools:\n  - Bash`，5 分钟

### 8.3 别人的更优方案

1. **领域**：多 plugin 分层（按使用场景分组，各自独立安装）
   - **本案例做法**：3 个独立 plugin，用户按工作流段落选择安装（`stitch-design` / `stitch-utilities` / `stitch-build`）
   - **我的项目现状**：claude-for-legal 有多个子域（regulatory、corporate、litigation）但都放在单一仓库根目录，没有 plugin 层分组
   - **如何借鉴**：为 claude-for-legal 在根目录创建 3 个 plugin.json（各指向对应子域 skills/），不需要移动任何文件

2. **领域**：Output Format section 的明确声明
   - **本案例做法**：design-md 和 taste-design 都有 `## Output Format (DESIGN.md Structure)` 节，明确说明输出什么
   - **我的项目现状**：echo-sleuth commands 没有显式 Output Format 声明（只有步骤描述）
   - **如何借鉴**：在 commands/extract.md 末尾加 `## Output Format`，说明提取结果写到哪个 JSONL 文件、格式是什么

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：技术栈描述 skill 中避免 ghost tools（声明但未使用的工具）
- **我的做法**：echo-sleuth/commands/extract.md 的 allowed-tools 都是 body 中实际用到的（Bash 对应脚本调用，Read/Edit/Write 对应文件操作，AskUserQuestion 对应用户确认）
- **本案例做法**（弱在哪）：react-components 和 shadcn-ui 在 allowed-tools 声明了 `web_fetch`，但 skill body 不直接调用网络——网络操作通过 scripts/fetch-stitch.sh 完成。这是一个声明与实现不匹配的例子。
- **意义**：工具声明与实际使用对齐是最小权限原则的体现，echo-sleuth 在这一点上比 stitch-skills 更规范。

---

## 八、术语表

### <a name="plugin.json"></a>plugin.json（manifest 清单）
> 插件的「目录」。Claude Code 通过这个 JSON 文件知道这个插件有哪些 skill、哪些 agent、名字叫什么。stitch-skills 有 3 个 plugin.json，意味着它是 3 个独立的插件——你可以只安装 stitch-build（前端构建），不安装 stitch-design（设计工作流）。就像超市的分区：你只需要买肉类区的东西，不需要强制走完所有区域。

### <a name="glob-pattern"></a>glob 模式
> 一种用通配符匹配多个名字的写法。`stitch*:*` 的意思是「所有以 stitch 开头、后面跟冒号再跟任意字符的工具名」。好处：当 Stitch MCP 服务器改版，工具名从 `stitch_generate` 变成 `stitch2_generate` 时，`stitch*:*` 依然能匹配，但写死的 `stitch_generate` 就失效了。对比：固定字符串是认死规矩，glob 是认大方向。

### <a name="allowed-tools"></a>allowed-tools
> skill 的「工具白名单」。声明这个 skill 被允许使用哪些工具（如 `Read`、`Bash`、`stitch*:*`）。作用：①限制 skill 权限，防止 AI 随意调用高危工具；②告知 Claude Code 这个 skill 需要哪些工具，便于权限审批。如果声明了 `Bash` 但 skill body 里没有任何 Bash 操作，这就是「幽灵工具」——声明了但没用，浪费权限配额，也让读者困惑。
