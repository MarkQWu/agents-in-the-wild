# uppinote20/claude-dashboard — 学习案例

**仓库**：https://github.com/uppinote20/claude-dashboard
**Stars**：404（exemplar_published 覆盖星数门槛）| **来源**：upstream（exemplar_published，SECURITY REVIEW）
**Audit 日期**：2026-04-28（历史快照）| **生成日期**：2026-07-23（基于当前 HEAD）
**主题标签**：`template-design`, `examples-driven`, `security-gate`, `manifest-discipline`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-dashboard 是一个 Claude Code 插件，为终端状态行提供模块化 Widget 系统，支持 AI Token 消耗跟踪、API 限速显示、Cost 追踪、Codex/Gemini/z.ai 等多 CLI 工具用量统计。作者是「uppinote20」，韩国开发者（仓库含韩文文档）。插件用 TypeScript + esbuild 构建，发布编译后的 JS。

关键事实：
- 4 个命令（`setup`、`setup-alias`、`check-usage`、`update`），对应 4 个 Markdown 命令文件
- Widget 系统目前支持 30+ 个 Widget（见 `scripts/widgets/`）
- 附带完整韩/英双语 Astro 文档网站（`website/`）
- 被 NLPM 评选为 exemplar，例示 R14/R15/R16/R18/R35/R38 六条规则

### 1.2 架构剖析

```
uppinote20/claude-dashboard/
├── .claude-plugin/
│   ├── plugin.json              # manifest：注册 4 个命令
│   └── marketplace.json         # marketplace 元数据
├── commands/
│   ├── setup.md                 # 主设置命令，带 interactive/direct 双模式
│   ├── setup-alias.md           # Shell alias 安装命令
│   ├── check-usage.md           # AI CLI 用量查询命令
│   └── update.md                # 版本更新命令
├── scripts/
│   ├── statusline.ts            # 状态行入口（主产品）
│   ├── check-usage.ts           # check-usage CLI 入口
│   ├── widgets/                 # 30+ 个 Widget 实现（每个独立 .ts 文件）
│   └── ...
├── dist/                        # 编译后的 JS（已 commit 入仓库）
├── locales/                     # 国际化 JSON
├── CLAUDE.md                    # 项目 AI 指导文件（含 Widget 参考表）
└── website/                     # 英/韩双语文档网站
```

- **文件类型分布**：4 个命令文件，1 个 plugin.json，1 个 CLAUDE.md，30+ 个 TS Widget
- **编排关系**：4 个命令相互独立，无 router；`setup` 是最复杂的，带 interactive 和 direct 两条路径
- **跨件契约**：CLAUDE.md 作为 AI 指导文件，含完整 Widget ID 参考表和 `DISPLAY_PRESETS` TypeScript 块，指导 Claude 在修改代码时知道改哪里

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「命令即协议——每个 Claude 命令是一份可执行的合同，精确描述输入格式、分支逻辑和输出模板」
- **解决什么问题**：Claude Code 的状态行配置本来需要用户手动编辑 JSON 文件。这个插件把复杂配置流程变成两个命令（`/claude-dashboard:setup` 交互式引导，`/claude-dashboard:update` 一键更新）
- **Trade-off**：dist/ 编译后的 JS commit 入仓库（避免用户需要 Node.js 构建）vs 仓库体积变大、PR diff 难读
- **认知模型**：作者把每个命令文件当做「用户操作指南 + AI 执行脚本」合二为一——同一个文件，人读懂流程，Claude 读懂步骤

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「分支标签命令 + 输出模板三元组」**

每个命令明确标出 if/else 分支（用粗体标题），每条分支给出对应的输出模板（逐字照抄格式），再配合 `argument-hint` 提示用户可选参数。三者合在一起让命令做到「零歧义」。

模式特征清单：
- 特征 1：empty/non-empty arguments 用粗体标题显式分支（**If no arguments provided** / **If arguments provided**）
- 特征 2：每条分支有对应的逐字输出模板（含 emoji、空格、换行都写死）
- 特征 3：`argument-hint` 在 [frontmatter](#frontmatter) 中声明所有可选参数和分隔符
- 特征 4：CLAUDE.md 包含完整的 Widget 参考表和 TypeScript 类型块，让 AI 改代码时知道边界
- 特征 5：`AskUserQuestion` 调用限定「每次最多 4 个问题」，减少往返

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 配置安装类命令，用户选项多 | ✅ 高度适用 | 双模式（interactive/direct）覆盖两类用户 |
| 业务逻辑复杂、需要多步决策 | ✅ 适用 | 标签化分支让 Claude 不会走错路 |
| 简单单步命令（如读一个文件输出摘要） | ❌ 过度工程 | 不需要这么多结构 |
| 需要用户实时输入（如代码 review） | ⚠️ 部分适用 | interactive 模式适合，但 batch 4 问上限需要权衡 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：分支标签 + 输出模板 | uppinote20/claude-dashboard | 零歧义，Claude 输出高度一致 | 模板维护成本高，每次改格式要改多处 |
| 备选 A：纯散文命令 | 大多数简单命令 | 写起来快 | Claude 可能自行发挥，输出不一致 |
| 备选 B：only `AskUserQuestion` | 交互式向导 | 用户体验好 | 往返多，不支持 direct 模式 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 项目结构树（第 21-23 行）只列了 `setup.md` 和 `check-usage.md`，漏掉 `setup-alias.md` 和 `update.md`。**改进做法**：在 CLAUDE.md 树中补全所有 4 个命令文件。**预期收益**：新贡献者（包括 Claude）不会以为只有两个命令。

2. **当前问题**：`commands/check-usage.md` 第 39 行 `$ARGUMENTS` 未加引号（`node ... $ARGUMENTS`）。**改进做法**：改为 `"$ARGUMENTS"`（加双引号）。**预期收益**：防止用户输入含空格或特殊字符时触发 shell 注入。这是 High 严重度安全问题，应优先修复。

3. **当前问题**：`commands/setup.md` 的 `allowed-tools` 声明了 `Bash(cat:*)` 但无任何步骤用到 `cat`，所有文件操作走 `Write` 和 `jq`。**改进做法**：删除 `Bash(cat:*)` 声明。**预期收益**：减少不必要的权限声明，提升安全审计清晰度。

---

## 三、过去审查发现（2026-04-28 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-28 当时得分 99/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/setup.md | 97 | `Bash(cat:*)` 无用声明（-3） |
| CLAUDE.md | 98 | "comprehensive" 模糊量词（-2） |
| commands/setup-alias.md | 98 | 步骤 2 标题含 "appropriate"（-2） |
| .claude-plugin/plugin.json | 100 | — |
| commands/check-usage.md | 100 | — |
| commands/update.md | 100 | — |

### 3.2 当时值得借鉴的模式

**1. R14：步骤必须编号**（`commands/update.md`）：3 个编号步骤，每个附有可直接复制的 shell 块，无连接散文——顺序无歧义。

**2. R15：空输入显式处理**（`commands/setup.md:83-131`）：两条路径用粗体标题明确标出，先写 interactive 模式（default），再写 direct 模式——Claude 不可能猜错分支。

**3. R16：输出格式逐字定义**（`commands/setup-alias.md:103-130`）：already-installed、newly-added、Windows 三种情况，每种都有逐字输出模板（含 ✓ emoji 和缩进空格）。

**4. R18：argument-hint 声明可选参数**（`commands/setup.md:3`）：`"[displayMode] [language] [plan] | custom \"widgets\""` 在一行内展示所有参数和特殊路径，用户无需翻 body。

**5. R35：CLAUDE.md 含架构组件图**：36 行 Widget 参考表（Widget ID → 数据源 → 描述）+ `DISPLAY_PRESETS` TypeScript 块，构成完整的组件地图。

### 3.3 当时的缺陷

**1. CLAUDE.md 结构树缺失两个命令（Bug #1）**
- **根本原因**：结构树是手动维护的，作者在后续添加 `setup-alias.md` 和 `update.md` 时忘记更新文档。典型的「代码改了，文档没跟上」问题
- **自查**：我的 `MarkQWu/bureau/CLAUDE.md` 是否及时同步了所有命令？值得检查

**2. check-usage.md 中 `$ARGUMENTS` 未加引号（High 安全问题）**
- **根本原因**：在 Claude 命令文件中写 shell 命令时，开发者直接复制了可能在交互 shell 中工作的写法，没有考虑 `$ARGUMENTS` 可能包含空格或 shell 特殊字符
- **自查**：凡是在命令文件中用 `$ARGUMENTS` 拼接 shell 命令的，都需要加引号

**3. `Bash(cat:*)` 无用声明（-3分）**
- **根本原因**：作者在 `allowed-tools` 声明了最初版本中用到的工具，后来重构去掉了 `cat` 的使用，但 `allowed-tools` 没有清理
- **自查**：我的命令文件里 `allowed-tools` 是否有声明但从未使用的工具？

### 3.4 当时的优化机会

1. 补全 CLAUDE.md 结构树（低风险，5 分钟）
2. 给 `$ARGUMENTS` 加引号（一字之差，消除 High 安全风险）
3. 删除 `Bash(cat:*)` 声明，清洁 allowed-tools

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| CLAUDE.md 结构树缺少 setup-alias.md、update.md | `grep -n "setup-alias\|update\.md" CLAUDE.md` | **持续**（第 21-23 行仅列 setup.md 和 check-usage.md） | Bug #1 至今未修复 |
| $ARGUMENTS 未加引号 | `grep "ARGUMENTS" commands/check-usage.md` | **持续**（第 39 行：`node ... $ARGUMENTS`） | High 安全问题持续，shell 注入风险未消除 |
| Bash(cat:*) 无用声明 | `grep "cat" commands/setup.md` | **持续**（第 4 行仍有 `Bash(cat:*)`） | 三项质量问题全部未修复 |

**结论**：当前 HEAD（commit `1281b7d`）的所有 Audit 缺陷均原样保留。NLPM 如有提交 PR，均未被合并。

### 4.2 架构演进

当前 HEAD 与 2026-04-28 审计时比较：

- **命令文件**：仍为 4 个（setup、setup-alias、check-usage、update），无新增
- **Widget 数量**：从审计时的基础 Widget 扩展到 30+ 个（含 Codex/Gemini/z.ai/budget/forecast 等多个新 Widget），说明作者主要精力在扩展功能层
- **文档**：新增了完整的 Astro 网站（韩/英双语），显示用户获取重心从 Claude 内置说明转向外部文档
- **CLAUDE.md**：变得更长（包含 30+ Widget 参考表），架构指导更完整

作者在「功能扩展 + 文档丰富」两个方向用力，但 NL 质量修复基本被忽略。

### 4.3 新增的可学习模式

**CLAUDE.md 作为 Widget 查找表**：现在的 CLAUDE.md 有一个 30+ 行的 Widget 参考表（Widget ID → 数据源 → 描述），还有 `DISPLAY_PRESETS` TypeScript 块直接嵌入配置代码。这是 R35（架构概览）的进化版——不只描述组件，而是把组件 ID 和 TypeScript 类型都直接放进 AI 上下文，让 Claude 改 Widget 时无需猜测。

---

## 五、校准

### 5.1 我已经在做对的

1. **命令步骤编号**：我的 `MarkQWu/bureau/commands/` 下的命令文件（lint.md、status.md 等）都有编号步骤，符合 R14
2. **输出格式定义**：bureau 的命令文件定义了输出格式（如 logbook 入口的格式），符合 R16 方向
3. **CLAUDE.md 维护**：我的仓库都有 CLAUDE.md，且尝试保持同步

### 5.2 挑战 / 验证

**这次案例挑战了我的一个假设**：我以为「文档写得很好的仓库不会有安全问题」。uppinote20/claude-dashboard 是整个审计库中得分最高的（99/100），设计极其精良，但它有一个 High 安全问题（`$ARGUMENTS` 注入）从 2026-04-28 至今持续未修。这证明 NL 质量高不等于安全无虞——两个维度需要独立审查。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件中是否有 $ARGUMENTS 未加引号拼接 shell 命令
grep -rn 'Bash.*ARGUMENTS\|ARGUMENTS.*Bash\|\$ARGUMENTS' /tmp/my-repos/*/commands/*.md 2>/dev/null | grep -v '".*\$ARGUMENTS.*"'
# 命中后怎么办：立即加双引号，或改为通过 allowed-tools + $ARGUMENTS 直接传递而非 shell 插值
```

```bash
# 检查我的 CLAUDE.md 结构树是否遗漏命令文件
for repo in /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau; do
  echo "=== $repo ==="
  cmd_count=$(find "$repo/commands" -name "*.md" 2>/dev/null | wc -l)
  claude_list=$(grep -c '\.md' "$repo/CLAUDE.md" 2>/dev/null || echo 0)
  echo "commands/ 文件数: $cmd_count，CLAUDE.md 中 .md 引用数: $claude_list"
done
# 命中后怎么办：在 CLAUDE.md 中补全所有命令文件的路径
```

```bash
# 检查 allowed-tools 中是否有声明但实际未使用的工具
for f in /tmp/my-repos/MarkQWu-bureau/commands/*.md; do
  tools=$(grep "^allowed-tools:" "$f" | sed 's/allowed-tools://g')
  echo "=== $f ==="
  echo "声明工具: $tools"
done
# 命中后怎么办：删除从未在 body 步骤中出现的工具声明
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `MarkQWu/bureau` 的 setup 命令添加 interactive/direct 双模式（仿 setup.md 的分支标签模式）
   - **为何可行**：bureau 目前只支持 interactive setup，power user 想用命令行参数跳过问询
   - **第一步**：在 `commands/` 下找最复杂的那个命令，加一个 `**If no arguments provided:**` / `**If arguments provided:**` 的分支标签

2. **想法**：在 `MarkQWu/gstack` 的 CLAUDE.md 中加入 Widget 类的「skill 参考表」——哪些 skill 对应哪些场景，像 claude-dashboard 的 Widget ID 表
   - **为何可行**：gstack 有 59 个 skill，新手（包括 Claude 在新会话中）很难知道用哪个
   - **第一步**：写一个 skill-reference.md 或在 CLAUDE.md 中加 2-3 列表格（skill 名 | 触发场景 | 主要工具）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 uppinote20/claude-dashboard 的核心目的**：通过 Claude Code 命令简化 Claude 状态行配置 + AI CLI 用量追踪

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是多命令 Claude Code 插件，有 setup 流程 | bureau 面向知识管理，dashboard 面向状态显示 | 高 |
| MarkQWu/gstack | 低 | 同样有很多 skill/命令 | gstack 是工具集合，非单一工具安装器 | 中 |
| 其余仓库 | 无 | — | — | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| CLAUDE.md 结构树遗漏命令 | `grep -c '\.md' CLAUDE.md` vs 实际 `commands/` 文件数 | 未验证（需手动对照），但 bureau 结构经常变动，高概率有漂移 | 中 |
| $ARGUMENTS 未加引号 | `grep '\$ARGUMENTS' commands/*.md` | bureau commands 中未发现（当前 11 个命令）| 低 |
| allowed-tools 声明了实际未用的工具 | 逐文件对照声明与 body 步骤 | 未系统检查，可能存在 | 低 |

**命中后的具体行动建议**：
- 对 `MarkQWu/bureau/CLAUDE.md` 的结构树做一次手动核对，补全所有命令 → 15 分钟

### 7.3 别人的更优方案

1. **领域：输出模板三元组（已安装/新安装/Windows）**
   - **本案例做法**：`commands/setup-alias.md:103-130` 给三种情况各写了逐字输出模板，包括 emoji 和缩进
   - **我的项目现状**：`bureau/commands/` 的命令输出格式定义不够具体，只说「展示结果」
   - **如何借鉴**：在 bureau 最重要的命令（如 `compile.md`）中，为「成功/部分失败/完全失败」三种情况各写一个逐字输出模板

2. **领域：AskUserQuestion 上限声明（max 4 per call）**
   - **本案例做法**：`commands/setup.md:87` 明确写 "Batch independent questions into a single AskUserQuestion call (max 4 per call)"
   - **我的项目现状**：我的交互式命令没有声明问题批次上限，Claude 可能一次问一个问题造成多轮往返
   - **如何借鉴**：在 bureau 的 setup/init 类命令中加一行「Batch related questions, max 4 per AskUserQuestion call」

### 7.4 反向：我的项目做得比他们好的地方

- **领域：文档同步机制**：我的 `MarkQWu/bureau` 有 hooks 来监控文件变化并提醒同步文档，而 claude-dashboard 的 CLAUDE.md 与实际命令目录完全手动维护，至今仍漂移
- **意义**：自动化文档一致性检查（如用 PostToolUse hook 检测 commands/ 变更后提醒更新 CLAUDE.md）比手动更可靠

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（如 `allowed-tools`、`argument-hint`）。Claude Code 读命令文件时先解析 frontmatter，再执行 body 中的步骤。

### <a name="argument-hint"></a>argument-hint
> frontmatter 中的一个字段，声明命令接受的参数格式。显示在 `/help` 输出中，帮助用户知道如何调用命令，格式示例：`"[displayMode] [language] [plan] | custom \"widgets\""` 表示三个可选参数加一个特殊模式。

### <a name="allowed-tools"></a>allowed-tools
> frontmatter 中的字段，白名单声明该命令可以使用的 Claude 工具。未列在此处的工具即使在 body 中被调用也会要求用户手动批准。常见值：`Read`、`Write`、`Bash`、`AskUserQuestion`。
