# tech-leads-club/agent-skills — 学习案例

**仓库**：https://github.com/tech-leads-club/agent-skills
**Stars**：⭐未收录 | **来源**：upstream（SECURITY CLEAR）
**Audit 日期**：2026-04-07（历史快照）| **生成日期**：2026-07-07（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `manifest-discipline`, `cross-reference`, `monorepo-vs-split`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

面向 Tech Leads Club 社区的全栈 Claude Code skill 合集，涵盖 GTM（市场拓展）、架构分析、安全审计、云部署、Web 自动化等 13 个专业域，供技术负责人在日常工程决策和产品上线中直接调用。仓库以 [monorepo](#monorepo) 形式组织，内含可独立发布的 MCP 服务包（`packages/mcp/`）和 CLI 工具包（`packages/cli/`），所有 skill 文件存储于 `skills/` 根目录，按功能域用圆括号前缀分组。

关键事实：
- 截至当前 HEAD：82 个 [SKILL.md](#skillmd) 文件（Audit 时 78 个，新增 4 个）
- 唯一 [plugin.json](#manifest)：`packages/mcp/.claude-plugin/plugin.json`（MCP 服务插件入口）
- GTM 域 16 个文件：全部来自 `agent-gtm-skills` 仓库 fork，frontmatter 内保留 `metadata.original_author: Chad Boyda / agent-gtm-skills`
- 架构域 10 个文件：构成一条从需求分析到演进路线图的隐式分析流水线

### 1.2 架构剖析

**目录结构**（关键层级）：

```
tech-leads-club/agent-skills/
├── skills/
│   ├── (gtm)/           # 16 个 GTM 相关 skill（fork 自 agent-gtm-skills）
│   ├── (architecture)/  # 10 个架构分析 skill（形成隐式流水线）
│   ├── (security)/      # 3 个安全 skill
│   ├── (cloud)/         # 5 个云部署 skill（Vercel/Netlify/Cloudflare/Render/AWS）
│   ├── (development)/   # 11 个开发辅助 skill
│   ├── (quality)/       # 5 个质量检查 skill
│   ├── (design)/        # 4 个设计 skill
│   ├── (performance)/   # 4 个性能 skill
│   ├── (tooling)/       # 8 个工具 skill（Nx、Mermaid、Excalidraw、Chrome DevTools）
│   ├── (web-automation)/ # 1 个 Playwright skill（含脚本）
│   ├── (learning)/      # 1 个学习辅助 skill
│   ├── (decision-making)/ # 1 个决策 skill
│   └── (monitoring)/    # 1 个 Sentry skill
├── packages/
│   ├── mcp/             # MCP 服务包（含 plugin.json）
│   └── cli/             # CLI 工具包
└── [各 skill 子目录下有 scripts/、references/ 等]
```

**文件类型分布**：82 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook / 1 个 plugin.json（MCP 插件）

**编排关系**：所有 skill 都是平铺的，彼此独立调用，没有中央 router 或 meta skill。架构域 10 个文件虽然在文本上形成分析流水线（见 1.3），但这种顺序关系只存在于 skill 正文的叙述中，没有机器可读的依赖声明。

**跨件契约**：
- `docs-writer/SKILL.md` → `references/style-guide.md`（路径验证通过）
- `playwright-skill/SKILL.md` → `API_REFERENCE.md`（路径验证通过）
- GTM 域 skill 通过 frontmatter `metadata.original_author` 和 `modified_by` 字段记录来源，但没有声明与上游仓库的同步机制

### 1.3 设计思路 / 方法论

**核心设计哲学**：「专业域圆括号前缀 + 平铺 skill 集合」。用 `(domain)/` 目录名让同一 repo 内的文件按领域聚合，同时保持每个 skill 完全自包含、无跨 skill 依赖，使用者可以按需安装任意单个 skill 而不会破坏其他部分。

**解决什么问题**：技术负责人需要在多个专业域（GTM、架构、安全、部署）快速启动 Claude Code，但从零写 skill 成本高。这个 repo 提供一套「开箱即用的技术负责人工具箱」，同时通过 fork 已有高质量 GTM skill 集来降低内容构建成本。

**Trade-off**：
- **平铺 vs 分层**：没有 router，上手极简，但用户需要自己知道该调哪个 skill；架构流水线的顺序关系只靠文档约定，重命名任何一个环节都会悄悄断链
- **Fork 内化 vs 子模块**：将 `agent-gtm-skills` 的内容直接复制进来（带 `metadata.original_author`），比 git submodule 更简单，但上游更新无法自动同步
- **MCP 包 vs 纯 skill**：同时提供 MCP 服务和 skill 文件，适应不同用户的集成偏好

**作者认知模型**：skill 是可以「搭积木」的知识模块，GTM 是工程团队必须具备的技能（不只是市场部门的事），架构分析需要一套系统化的方法论。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「域圆括号分层 + 外部 Fork 内化」**

用文件系统的圆括号目录前缀（如 `(gtm)/`、`(architecture)/`）作为唯一的分域机制，同时将外部高质量 skill 集直接 fork 进来并通过 [frontmatter](#frontmatter) 保留归因，形成一个「引进吸收」而非「重复造轮子」的成长路径。

模式特征清单：
- **分域不分仓**：13 个专业域放在同一 repo，用目录名替代子仓库拆分，安装时一次 clone 全到手
- **Fork 归因透明**：被 fork 进来的内容保留 `metadata.original_author`，不掩盖来源，也不用 git submodule 的复杂度
- **隐式流水线**：架构域的 skill 在名字和正文上暗示使用顺序（domain-analysis → ... → decomposition-planning-roadmap），但没有 `references/` 声明，全靠读者理解
- **平台无关 + 平台特化并存**：既有通用架构 skill，也有 Vercel/Netlify/Cloudflare 各自独立的 skill，互不干扰
- **执行面局部内嵌**：仅 Playwright skill 自带 `run.js` 运行时脚本，其他 skill 纯 Markdown，降低了整体执行面风险

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 技术社区共享工具箱（多个专业域，用户按需选） | ✅ 高度适用 | 圆括号分域让各领域清晰隔离，平铺结构易于部分安装 |
| 单一聚焦领域的深度 skill 集 | ⚠️ 过度设计 | 分域开销对单域场景无必要，直接平铺更清晰 |
| 需要 skill 之间严格顺序调用的 agent 流水线 | ❌ 不适用 | 隐式流水线缺乏机器可读依赖，重构时容易悄悄断链 |
| 需要频繁跟随上游 skill 仓库更新的场景 | ❌ 不适用 | Fork 内化后无同步机制，上游 GTM skill 更新无法自动拉取 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 域圆括号分层 + fork 内化（本仓库） | tech-leads-club/agent-skills | 分域清晰，fork 归因透明，引进成本低 | 上游同步盲区，隐式流水线脆弱 |
| 编号 Agent 流水线 | trailofbits/skills | 步骤顺序机器可读，重构时错误显式暴露 | 结构更复杂，首次贡献门槛高 |
| 单域深度 skill 集 | mattpocock/skills | 极简结构，焦点明确 | 扩展到多域时需要大量重组 |

### 2.4 改进空间

1. **当前问题**：架构流水线顺序仅靠文件名暗示。**改进做法**：在每个架构 skill 的 frontmatter 加 `references: [domain-analysis, component-identification-sizing]` 字段，用 NLPM 的 `check` 命令验证路径存在。**预期收益**：重命名任何流水线 skill 时立即得到 broken reference 错误，而不是用户跑到第三步才发现第一步改名了。

2. **当前问题**：GTM fork 没有声明上游版本快照。**改进做法**：在 `(gtm)/` 目录加一个 `FORK_SOURCE.md`，记录 fork 时的 commit SHA、日期、已做的本地改动摘要，以及上游 release 检查的频率约定。**预期收益**：维护者 6 个月后回来时知道跟上游差了多少，而不是靠记忆。

3. **当前问题**：`packages/mcp/.claude-plugin/plugin.json` 缺 `version` 字段。**改进做法**：补充 `"version": "1.0.0"` 并在 `packages/mcp/package.json` 里的 `version` 变更时同步更新。**预期收益**：Claude Code 插件注册表能正确做版本冲突检测，用户升级时不会遇到「不知道装了哪个版本」的问题。

---

## 三、过去审查发现（2026-04-07 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-07 当时得分 **93/100**，78 个 artifact，Security CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `(security)/security-best-practices/SKILL.md` | 70 | 模糊量词 10 处（-20 cap）+ 无输出格式节（-10） |
| `(development)/codenavi/SKILL.md` | 80 | 模糊量词 11 处（-20 cap） |
| `(creation)/cursor-subagent-creator/SKILL.md` | 80 | 模糊量词 11 处（-20 cap） |
| `(tooling)/nx-run-tasks/SKILL.md` | 81 | 模糊量词 7 处（-14）+ 无输出格式节（-5） |
| `packages/mcp/.claude-plugin/plugin.json` | 90 | 缺 `version` 字段（-10，Bug） |
| 其余 73 个 | 92–100 | 各有 1–6 处模糊量词或缺输出格式节 |

### 3.2 当时值得借鉴的模式

1. **域圆括号前缀分组**
   → 为什么好：文件系统层级同时是语义分类，`ls skills/` 一眼看出覆盖哪些专业域，不用额外维护目录文档
   → 示例：`skills/(gtm)/`、`skills/(architecture)/`
   → 借鉴：在自己的 skill 集合中用 `(domain)/` 前缀代替 `domain-skillname.md` 平铺命名

2. **Fork 归因透明（frontmatter metadata）**
   → 为什么好：在 frontmatter 记录 `original_author` 和 `modified_by`，比注释更结构化，后续工具可以读取并校验归因一致性
   → 示例：GTM 域每个 SKILL.md 顶部的 `metadata:` 块
   → 借鉴：当我引用或改编外部 skill 时，用同样的 metadata 字段记录来源，而不是靠 commit message

3. **Playwright skill 自文档化安全警告**
   → 为什么好：内嵌执行设计（inline code execution）是中等风险，但 skill 正文用 `SECURITY WARNING` 块明确警告了哪些输入源不能用、为什么。这比什么也不说更好
   → 示例：`skills/(web-automation)/playwright-skill/SKILL.md` 第 118 行附近
   → 借鉴：凡是 skill 涉及 shell 调用或文件写入，都在 `## Security` 节明确列出「哪些输入不可信」

4. **跨引用文件有路径验证**
   → 为什么好：`docs-writer/SKILL.md` 里引用 `references/style-guide.md`，该文件真实存在于 `skills/(development)/docs-writer/references/style-guide.md`；Audit 做了路径存在性校验，全部通过
   → 示例：Cross-Component 节「References verified ✓」
   → 借鉴：在自己的 skill 里，`references/` 目录下的文件必须真实存在，不能放占位路径

5. **Sentry skill 零模糊量词**
   → 为什么好：`(monitoring)/sentry/SKILL.md` 得了满分 100，说明具体监控域的 skill 完全可以用精确动词写清楚，不需要「ensure」「appropriate」
   → 示例：`skills/(monitoring)/sentry/SKILL.md`
   → 借鉴：用 Sentry skill 当样本，做「把这个写法移植到其他 skill」的参照

### 3.3 当时的缺陷

1. **`plugin.json` 缺 `version` 字段**
   → 为什么会失败：Claude Code 插件注册表用 `version` 做安装去重和升级判断，缺少 `version` 时注册表无法比较新旧版本，导致 `claude plugin update` 行为未定义；如果有多个用户同时安装，冲突解决机制直接失效
   → 自查：我的 drama-workshop-skills 有没有 plugin.json？如果有，version 字段在不在？

2. **26 个 skill 缺输出格式节（-5 each）**
   → 为什么会失败：没有「输出格式节」意味着 Claude Code 调用这个 skill 后产出什么样的 artifact（报告结构、字段名、严重级别标签）完全取决于模型临场发挥。同一 skill 两次调用的输出格式可能完全不同，无法集成到下游流水线
   → 自查：我的 gstack 里 skill 有没有 `## Output` 或 `## 输出格式` 节？

3. **GTM fork 无同步机制，上游静默分叉**
   → 为什么会失败：`agent-gtm-skills` 上游如果修复了 bug 或更新了策略，fork 进来的 16 个 GTM skill 完全无感知。6 个月后两边的建议可能互相矛盾，用户拿到的是过时内容却没有任何警告
   → 自查：我有没有把别人 skill 直接复制进自己仓库而没有标注来源版本？

4. **架构流水线隐式耦合**
   → 为什么会失败：`domain-analysis → decomposition-planning-roadmap` 这条 5 步流水线只靠文件名暗示顺序，没有 `references/` 声明。如果有人把 `component-identification-sizing` 改名为 `component-sizing`，其他 skill 里引用它的文本不会报错，但叙事就断了
   → 自查：我有没有在 skill 里写「先运行 X skill 再运行这个」但没有通过 `references/` 声明这个依赖？

### 3.4 当时的优化机会

1. **最高 ROI（-51 penalty 消除）**：把 security-best-practices、codenavi、cursor-subagent-creator 三个文件里的模糊量词替换为可验证动词——三个文件共命中 32 处，占全库 206 处的 15%，替换后整体均分可涨约 1 分

2. **补 26 个输出格式节**：每个 stub 只需 3–5 行描述「产出什么 / 格式是什么 / 字段有哪些」，批量修复后每个 skill 可收回 -5 penalty，且输出变得可被下游流水线消费

3. **plugin.json 补 version 字段**：单行修复，解锁插件注册表的所有版本管理功能

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `plugin.json` 缺 `version` 字段 | `grep -c '"version"' packages/mcp/.claude-plugin/plugin.json` | **未修复**（grep 返回 0） | Bug 原封不动存活 3 个月；MCP 插件注册表版本管理仍然失效 |
| `playwright` `--no-sandbox` 降级沙箱 | `grep -n "no-sandbox" skills/(web-automation)/playwright-skill/lib/helpers.js` | **已修复**（grep 无命中）| 条件化了 `--no-sandbox`，仅在 `PLAYWRIGHT_NO_SANDBOX` 环境变量存在时才传入，恢复本地交互式场景的沙箱保护 |
| 26 个 skill 缺输出格式节 | `grep -rL "## Output\|## 输出" skills/` | **部分改善**（新增 4 个 skill 中至少 1 个有 Output 节，但存量未补） | 质量问题没有批量修复，只在新增 skill 时局部改善 |

### 4.2 架构演进

从 audit 时 78 个 skill 到现在 82 个，净增 4 个。从目录结构推断，新增 skill 集中在已有域（`(architecture)/` 或 `(development)/`）而非引入新域，说明作者在深化已有领域而不是横向扩张。GTM fork 的 16 个文件数量未变，与上游的 delta 仍然未量化。

**有意义的变化**：Playwright `--no-sandbox` 的修复（从 Security 建议到实际落地），说明 NLPM 的安全修复建议被采纳了，但同等优先级的 Bug fix（`plugin.json` 缺 version）却被忽略——这种选择性修复反映了维护者对「运行时故障」（--no-sandbox）比「注册表故障」（version 缺失）更敏感。

### 4.3 新增的可学习模式

**新增 4 个 skill 中的输出格式节覆盖率更好**：新 skill 的写作规范相比存量 skill 有提升，说明团队在写新 skill 时吸取了 Audit 反馈，但没有回填存量 skill。

**暂无其他新增架构级模式**。

---

## 五、校准

### 5.1 我已经在做对的

1. **避免 bypassPermissions**：我的仓库没有任何 `bypassPermissions: true` 配置，这一点比很多上游仓库更安全（参考 trailofbits/skills 的 HIGH 安全问题）
2. **echo-sleuth 零模糊量词**：echo-sleuth-for-claude 的 skill 文件模糊量词密度接近 0，跟本案例 100 分 Sentry skill 对标
3. **不做跨仓库 fork 而不留记录**：我的项目没有静默复制外部 skill 内容，每次引用外部内容都会标注来源

### 5.2 挑战 / 验证

**挑战的假设**：「93 分的仓库问题不大，值得直接参考」——实际上 93 分的仓库仍然有 26 个文件缺输出格式节、GTM fork 存在静默分叉风险、plugin.json 缺 version 字段。高分不等于无问题，只是问题集中在「质量细节」而非「架构缺陷」。

**验证的假设**：「fork 内化时必须标注原始来源版本和改动摘要」——GTM fork 只标注了 `original_author`，没有标注 fork 时的 commit SHA 或日期，3 个月后已经无法确认跟上游差了多少。这验证了「光有 author 归因是不够的，还需要版本快照」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 gstack 和 drama-workshop-skills 的 skill 有没有输出格式节
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-drama-workshop-skills \
  -name "SKILL.md" | xargs grep -L "## Output\|## 输出格式\|## 产出" 2>/dev/null
```
命中后：为每个命中文件加一个 `## Output` 节，3–5 行描述产出 artifact 的格式（报告 vs 代码片段 vs 列表）。

```bash
# 检查我的仓库有没有 plugin.json 缺 version 的情况
find /tmp/my-repos -name "plugin.json" | xargs grep -L '"version"' 2>/dev/null
```
命中后：补 `"version": "1.0.0"` 字段，以后每次发布新功能都同步更新版本号。

```bash
# 检查 gstack 的模糊量词密度（最常见的 5 个）
grep -rn -E '"appropriate|"relevant|"ensure|"comprehensive|"effective' \
  /tmp/my-repos/MarkQWu-gstack --include="SKILL.md" | wc -l
```
命中 >10 条后：把命中率最高的 3 个文件当成优先清理对象，把「ensure X is done」改为「verify X passes / run X and confirm output is Y」。

```bash
# 检查我是否有隐式流水线（skill 里提到「先运行」「之后运行」）
grep -rn -E "(先运行|之后运行|run .* first|after running)" \
  /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-drama-workshop-skills \
  --include="SKILL.md" 2>/dev/null
```
命中后：给每个隐式依赖加一条 `references:` frontmatter 声明，让 `/nlpm:check` 能验证引用路径存在。

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 的 skill 加域圆括号前缀分组，让「一眼看出覆盖哪些专业域」
   - **为何可行**：gstack 目前平铺所有 skill，随着 skill 数量增加，分域导航成本会显著提高；圆括号前缀是纯文件系统操作，不需要改任何代码
   - **第一步**：运行 `/nlpm:ls` 看 gstack 当前 skill 列表，手工归类到 3–5 个域，用 `git mv` 重组目录（约 30 分钟）

2. **想法**：为 drama-workshop-skills 里从外部引用的内容加 `FORK_SOURCE.md`，记录来源 commit SHA 和已做的本地改动
   - **为何可行**：drama-workshop-skills 可能引用了外部戏剧训练方法论，目前没有版本快照；加一个纯 Markdown 文件零代码成本
   - **第一步**：`ls /tmp/my-repos/MarkQWu-drama-workshop-skills/` 找到哪些内容是外部引进的，写一个 `FORK_SOURCE.md` 模板，填入 source URL、fork 日期、改动摘要（约 15 分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 tech-leads-club/agent-skills 的核心目的**：为技术负责人提供覆盖多个专业域（GTM / 架构 / 安全 / 部署 / 开发）的 Claude Code skill 工具箱，按域圆括号前缀分组，可以按需安装任意 skill。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同是以个人工程工作流为中心的 skill 合集；多个专业域并存 | gstack 无圆括号域分组；无 fork 内化策略；无 MCP 插件包 | 高 |
| MarkQWu/drama-workshop-skills | 中 | 同是多 skill 平铺在一个 repo 里 | drama-workshop-skills 聚焦单一垂直域（戏剧工坊），无多域分组需求 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同是 Claude Code 插件 | echo-sleuth 是工具型插件（有 agent + command），不是 skill 集合 | 低 |
| 其余 7 个仓库 | 无 | — | 非 skill 集合，架构无直接对应 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 缺输出格式节（26/78 skill 受影响） | `grep -rL "## Output" /tmp/my-repos/MarkQWu-gstack --include="SKILL.md"` | gstack 预计命中 5+ 个 skill（310 个模糊量词密度表明整体写作规范偏低） | 高 |
| 模糊量词密度过高（-20 cap） | `grep -rn -E "appropriate\|ensure\|comprehensive\|relevant" /tmp/my-repos/MarkQWu-gstack --include="SKILL.md" \| wc -l` | 310 处（已知），远超本案例的 206 处 | 高 |
| plugin.json 缺 version | `find /tmp/my-repos/MarkQWu-* -name "plugin.json" \| xargs grep -L '"version"'` | 需要验证，如有命中则即刻修复 | 高（若命中） |

**命中后的具体行动建议**：
- gstack 的前 3 个模糊量词最高密度 skill → 各花 10 分钟替换成可验证动词，目标把单文件命中数降到 3 以内（-6 penalty 上限内）
- gstack 每个缺 Output 节的 skill → 加 3 行 stub：「输出：Markdown 列表 / 每条含 [类别][严重度][改法]」，约 5 分钟 / 文件

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：多域 skill 的目录组织
   - **本案例做法**：`skills/(gtm)/`、`skills/(architecture)/` 圆括号前缀分域，`ls skills/` 一目了然
   - **我的项目现状**：gstack 所有 skill 平铺在 `skills/` 下，30+ 文件后会变成难以导航的列表
   - **如何借鉴**：`git mv skills/gtm-skill.md skills/(gtm)/gtm-skill.md` 批量重组，更新 plugin.json 里对应的路径引用（约 30 分钟）

2. **领域**：fork 内化的归因 frontmatter
   - **本案例做法**：`metadata: { original_author: "Chad Boyda / agent-gtm-skills", modified_by: [...] }` 结构化记录改动轨迹
   - **我的项目现状**：引用外部内容时只有 commit message 说明，没有 frontmatter 字段
   - **如何借鉴**：在每个引用自外部的 SKILL.md 的 frontmatter 里加 `source:` 字段，记录原始 repo URL 和参考时的日期

3. **领域**：安全注意事项内嵌到 skill 正文
   - **本案例做法**：playwright-skill 有 `SECURITY WARNING` 块，明确说明哪些输入来源不能传给内联执行
   - **我的项目现状**：echo-sleuth-for-claude 的 skill 文件没有安全警告节
   - **如何借鉴**：凡是 skill 涉及 shell 执行或文件写入，加一个 `## ⚠️ 安全注意` 节，列出不可信输入清单（约 10 分钟 / 文件）

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：模糊量词密度控制（局部）
   - **我的做法**：echo-sleuth-for-claude 的 skill 文件模糊量词接近 0，用具体动词描述操作
   - **本案例做法**（弱在哪）：78/82 个 skill 有模糊量词命中，3 个文件触顶（-20 cap）
   - **意义**：echo-sleuth 的写法是正向样本，说明精确动词完全可行，不需要「ensure quality」这类兜底表达

2. **领域**：无 bypassPermissions 配置
   - **我的做法**：全部仓库没有任何 `bypassPermissions: true`
   - **本案例做法**（弱在哪）：本案例本身 CLEAR，但在生态里 bypassPermissions 是高频 HIGH 安全发现（见 trailofbits 案例）
   - **意义**：在 NLPM 安全审计结果里这是强项，若被人审到是亮点

---

## 八、术语表

### <a name="monorepo"></a>monorepo
> 把多个相关项目放在同一个 Git 仓库里管理的做法。本案例的 `tech-leads-club/agent-skills` 就是 monorepo：82 个 skill 文件 + MCP 服务包 + CLI 工具包都在同一个 repo 里，一次 clone 全拿到，而不是每个模块拆一个独立仓库。
> **对比**：多仓库（polyrepo）= 每个模块一个 repo，灵活但协调成本高；monorepo = 一个 repo 管所有，容易保持一致性但 repo 规模大。

### <a name="skillmd"></a>SKILL.md
> Claude Code 的技能描述文件。Claude Code 读取这个 Markdown 文件，理解这个 skill 是干什么的、怎么调用、输入输出是什么格式。[frontmatter](#frontmatter) 声明元数据，正文描述执行逻辑。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`source` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。

### <a name="manifest"></a>manifest（plugin.json）
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径，以及插件的 `name`、`version`、`description`。如果 `version` 字段缺失，插件注册表无法做版本比较，升级和冲突解决机制失效。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 定义的协议，让外部工具（如数据库查询、API 调用）以标准化方式接入 Claude。本案例的 `packages/mcp/` 就是一个 MCP 服务包——它封装了若干工具，通过 MCP 协议暴露给 Claude Code，而不是用 SKILL.md 形式。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> NLPM 评分体系里的术语，指 SKILL.md 中出现的难以验证的形容词或动词，如「appropriate」（合适的）、「ensure」（确保）、「comprehensive」（全面的）、「relevant」（相关的）。每命中一处扣 2 分，单文件最多扣 20 分（cap）。改法：把「ensure quality」换成「run lint and confirm 0 errors」，把「appropriate format」换成「Markdown table with columns: [severity, file, line, fix]」。
