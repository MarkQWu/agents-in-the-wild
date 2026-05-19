# JuliusBrussee/caveman — 学习案例

**仓库**：https://github.com/JuliusBrussee/caveman
**Stars**：22505 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-19（基于当前 HEAD）
**主题标签**：`security-gate`, `single-purpose`, `experience-accumulation`, `cross-reference`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

JuliusBrussee/caveman 是「穴居人写作风格」压缩技能的原始仓库，指导 Claude 用极简英语写作（去掉冠词、填充词、客套话），把自然语言输出压缩至 60-70% 体积，同时保留所有技术内容。22505★ 使其成为 Claude Code 生态中最高星技能之一。配套的 cavekit（564★）建立在 caveman 基础上，提供了规范驱动的 TDD 工作流。当前版本已扩展为多平台生态：Claude Code、Cursor、Windsurf、OpenAI Codex、以及自定义 `.agents/` 协议。

### 1.2 架构剖析

```
caveman/
├── .agents/                   # 其他 agent 平台的技能分发
│   ├── plugins/marketplace.json
│   └── skills/cavecrew/SKILL.md
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── .codex/
│   └── hooks.json             # OpenAI Codex 平台集成（SessionStart hook）
├── skills/
│   ├── cavecrew/SKILL.md      # 新增：子代理委托决策指南
│   ├── caveman/SKILL.md       # 核心：caveman 写作风格
│   ├── caveman-commit/SKILL.md  # Git commit 消息压缩
│   ├── caveman-compress/SKILL.md # 记忆文件压缩（CLAUDE.md/todos）
│   ├── caveman-help/SKILL.md  # 帮助卡片
│   ├── caveman-review/SKILL.md  # 代码审查
│   └── caveman-stats/SKILL.md   # 新增：caveman 使用统计
└── .claude-plugin/plugin.json
```

- **文件类型分布**：7 个 SKILL.md（相比 audit 时增加了 cavecrew 和 caveman-stats）、多平台分发文件夹、.codex hooks
- **编排关系**：cavecrew 作为元技能，指导何时委托给三种子代理（investigator/builder/reviewer）；其他技能平列
- **跨件契约**：caveman 写作风格是所有技能的底层约定；cavecrew 的输出格式也是 caveman 压缩格式，形成自我指涉的闭环

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「压缩即哲学」——caveman 不只是一个写作技巧，而是一个认知模型：把 AI 输出的冗余当成噪音，把简洁当成品质信号。这个哲学渗透到所有衍生技能中（commit、review、compress 都用 caveman 输出）
- **解决什么问题**：长会话中的 context 窗口耗尽问题——通过压缩 AI 输出（约 60%）和 CLAUDE.md 记忆文件，延长有效工作窗口
- **Trade-off**：caveman 输出对非英语母语用户友好度下降（碎片化句子可能更难理解），但对英语开发者效率收益显著
- **认知模型**：作者把 AI context 视为稀缺资源——每个 token 都有成本，因此写作风格本身可以成为一种资源优化手段

---

## 二、过去审查发现（2026-04-17 历史快照）

### 2.1 当时质量评分（NLPM）

该仓库 2026-04-17 当时得分 **92/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| caveman-compress/SKILL.md | 85 | 硬编码 `cd caveman-compress`，CWD 不是仓库根时失败 |
| plugins/caveman/skills/compress/SKILL.md | 88 | 占位符 `<directory_containing_this_SKILL.md>` 需要模型推理 |
| skills/compress/SKILL.md | 88 | 与 plugins 版本重名不一致 |
| skills/caveman-help/SKILL.md | 90 | 外部 URL 无降级方案 |
| skills/caveman/SKILL.md | 95 | 自动同步副本，内容优秀 |
| skills/caveman-commit/SKILL.md | 95 | 结构清晰，双层示例 |
| skills/caveman-review/SKILL.md | 95 | 良好的 ❌/✅ 示例，严重性前缀清晰 |

### 2.2 当时值得借鉴的模式

1. **多平台同步分发**（CI 驱动）→ `skills/caveman/SKILL.md` 是唯一真实源，CI 自动同步到 `.cursor/`、`.windsurf/`、`caveman/`、`plugins/caveman/skills/caveman/`，5 个副本。根本原因是一次写作、多平台复用，维护成本归于单一文件。示例：CLAUDE.md 中的同步规则说明。借鉴：核心 skill 做「主文件 + 同步副本」架构，不手动维护多份

2. **严重性前缀系统**（caveman-review）→ review 输出用 `❌ CRITICAL:` / `⚠️ WARNING:` / `💡 NOTE:` 前缀区分优先级，让接收方（人或 agent）无需阅读全文即可判断紧急程度。借鉴：输出中的多级问题使用统一的严重性前缀

3. **安全钩子设计**（hooks 目录，已在当前 HEAD 移除，但值得记录）→ 当时的 `safeWriteFlag` 使用 O_NOFOLLOW、原子 temp+rename、0600 权限、符号链接检查；`readFlag` 强制 64 字节大小上限和 VALID_MODES 白名单；statusline.sh 过滤非 `[a-z0-9-]` 字符。这是 OSS 插件中罕见的防御性安全设计

4. **双层示例**（caveman-commit）→ commit 消息 skill 同时提供「反例（❌）」和「正例（✅）」，让对比更直观。借鉴：技能示例总是成对提供

5. **EXTEND.md 模式变体（名为 FORMAT.md）** → caveman 用 FORMAT.md 作为共享格式规范（在 cavekit 中），把输出格式从技能实现中分离

### 2.3 当时的缺陷

1. **caveman-compress 硬编码 `cd caveman-compress`**：根本原因是作者假设 agent 调用时 CWD 是仓库根目录，但实际 agent 的 CWD 依赖于调用上下文，这个假设不可靠。自查：我的技能中是否有任何相对路径依赖于特定的 CWD？

2. **compress 技能名称分裂（caveman-compress vs compress）**：audit 时插件版本用 `name: compress`，源文件用 `name: caveman-compress`。这对按名称去重的 skill loader 会创建两个注册项。根本原因是发布插件时「短名更方便」但缺乏文档说明。自查：我的插件在不同分发路径下的技能 name 字段是否一致？

3. **安装脚本无完整性校验**：`install.sh` 用 `curl` 下载 JS hook 文件到 `~/.claude/hooks/` 但没有 SHA-256 校验，REPO_URL 固定在 `main` 分支而非 tagged release。根本原因是快速发布时优先考虑用户安装便利性，忽略了供应链风险。自查：我的安装脚本是否从可变 URL 下载可执行文件？

### 2.4 当时的优化机会

1. **caveman-compress Process 步骤改为 robust 路径解析**：使用 `<directory_containing_this_SKILL.md>` 模式或动态搜索 `scripts/__main__.py`，而非硬编码目录名
2. **install.sh 添加 sha256sum 校验**：发布 `hooks/checksums.sha256`，下载后校验，REPO_URL 改为 tagged release
3. **compress 名称统一**：在 CLAUDE.md 中文档化名称分裂的理由，或统一为单一名称

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| caveman-compress 硬编码 `cd caveman-compress` | `grep -n "cd caveman" skills/caveman-compress/SKILL.md` | **已修复** — Process 步骤改为「search for scripts/__main__.py next to this SKILL.md」，不再硬编码目录 | 作者意识到路径依赖问题，用动态搜索取代硬编码 |
| hooks/install.sh 无完整性校验 | `ls hooks/ && grep -n "sha256\|integrity" hooks/install.sh` | **无法验证** — `hooks/` 目录在当前 HEAD 中已不存在 | 作者完全移除了 hooks 目录，可能转移到了单独仓库或不再维护安装脚本 |
| compress 名称分裂（caveman-compress vs compress） | `grep "^name:" skills/caveman-compress/SKILL.md` | **无法验证** — `plugins/` 和旧的 `skills/compress/` 目录均已移除，只剩 `skills/caveman-compress/SKILL.md`（`name: caveman-compress`） | 通过删除问题文件解决了名称分裂问题；架构简化 |

### 3.2 架构演进

这是 4 个仓库中演进最显著的一个：

**移除了什么**：
- `hooks/` 目录（含 install.sh、caveman-activate.js、caveman-mode-tracker.js、caveman-config.js）
- `plugins/` 目录
- `caveman-compress/` 顶层目录（技能移至 `skills/caveman-compress/`）
- 旧的 `skills/compress/` 路径

**新增了什么**：
- `skills/cavecrew/SKILL.md` — 多子代理委托决策指南
- `skills/caveman-stats/SKILL.md` — 使用统计技能
- `.agents/` 目录 — 其他 agent 平台支持
- `.codex/hooks.json` — OpenAI Codex 平台 SessionStart hook 集成

**这说明作者意识到了什么**：
1. Hook 机制的维护成本（JS 文件 + 安装脚本 + 安全校验）超过了其价值，决定用更轻量的 `.codex/hooks.json` 方式替代
2. caveman 生态从「单一工具」演进为「多平台压缩写作标准」，`.agents/` 的引入是跨平台扩展的体现
3. cavecrew 的出现说明作者在考虑「context 压缩」的下一层：不只压缩输出，还要压缩子代理调用的 context 开销

### 3.3 新增的可学习模式

1. **cavecrew — 子代理委托决策指南**：cavecrew 是一个「元技能」，专门告诉 main thread 什么时候应该委托给子代理（investigator/builder/reviewer），以及委托的 context 压缩收益。这是一个显式的「委托策略即技能」的新模式——把多代理工作流的决策本身文档化。

2. **`.codex/hooks.json` 跨平台 hook 声明**：用 JSON 格式声明 SessionStart hook（通过 echo 命令加载 caveman 规则），比 JS 文件更轻量、更可移植、更易审查。这是比 claude-code 原生 hooks.json 更简洁的同类实现。

---

## 四、校准

### 4.1 我已经在做对的

1. **避免硬编码 CWD 依赖**：我的技能中使用动态路径解析，不假设 CWD
2. **审查类工具声明只读**：我在审查型技能描述中标注「不写文件」
3. **双层示例（反例 + 正例）**：我在技能示例中提供对比对
4. **严重性前缀系统**：我在输出中使用类似的优先级标注
5. **核心技能单一真实源**：我的跨平台分发也遵循「源文件 + 同步副本」模式

### 4.2 挑战 / 验证

**挑战**：我之前认为「hooks 是插件能力的核心组成部分」，但 caveman 完全移除了 hooks 目录（包括曾被 audit 赞扬为「高于平均水准」的安全钩子实现），改用更轻量的 `.codex/hooks.json`。这说明当维护成本（JS + 安装脚本 + 安全审查）超过功能价值时，移除是更好的选择。精简本身也是一种架构决策。

**验证**：`cavecrew` 的出现验证了「委托策略也可以作为技能文档化」这个想法——我之前只把「做什么」写成技能，但「何时委托给谁」也可以写成可查询的技能文档。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查技能中是否有硬编码的相对路径
grep -rn "cd \./\|cd \.\." ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：改为动态路径解析或绝对路径，或使用「搜索脚本所在目录」的模式

# 检查安装脚本是否从可变 URL 下载可执行文件
grep -rn "curl.*\.(sh\|js\|py)" ~/.claude/hooks/*.sh 2>/dev/null
# 命中后：添加 sha256sum 校验步骤，把 URL 固定到 tagged release 而非 main 分支

# 检查多平台分发的技能名称是否一致
find ~/.claude -name "SKILL.md" -exec grep "^name:" {} \; | sort | uniq -c | sort -rn | head -10
# 命中后：对 count > 1 的同名技能，检查是否是预期的同步副本
```

### 5.2 灵感 → 实施路径

1. **想法**：写一个「委托策略技能」（参考 cavecrew），明确描述何时委托给子代理
   - **为何可行**：cavecrew 验证了「委托决策」可以被文档化并作为可查询的技能；对于复杂多代理工作流，这能减少 main thread 的决策负担
   - **第一步**：在 `~/.claude/skills/` 下新建 `agent-delegation/SKILL.md`，列出 3-5 种常见任务类型和对应的委托策略

2. **想法**：用 `.codex/hooks.json` 模式取代重型 JS hooks
   - **为何可行**：JSON 声明 + echo 命令的组合更易审查、更易移植、零安全风险；caveman 从完整 JS hooks 退回到此模式说明轻量够用
   - **第一步**：把当前最简单的 SessionStart hook 改写为 `.codex/hooks.json` 格式，验证功能等价性后移除对应的 JS 文件

3. **想法**：在技能示例中统一使用「❌ 反例 + ✅ 正例」对比格式
   - **为何可行**：对比格式让代理（和人）更快理解边界，比单独正例的理解效率高
   - **第一步**：在 5 个最常用的技能中，把现有示例改为对比对格式（约 10 分钟/技能）
