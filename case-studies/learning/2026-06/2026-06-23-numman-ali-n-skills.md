# numman-ali/n-skills — 学习案例

**仓库**：https://github.com/numman-ali/n-skills
**Stars**：974 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-23（基于当前 HEAD）
**主题标签**：multi-plugin-monorepo, skill-aggregator, external-sync, security, typescript-skill

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

n-skills 是 Numman Ali 构建的 Claude Code [插件](#插件) 集合仓库。它的核心价值在于"聚合"：把来自不同上游仓库的 skill 用一套 YAML 配置 + CI 脚本同步进来，统一发布为一个可安装套件。截至 2026-06-23，仓库包含 5 个 plugin，974 颗星。用户可以通过 `claude plugin install n-skills@numman-ali` 一次性安装所有 skill。

关键事实：
- 作者维护一个 `sources.yaml` 文件，列出上游 skill 仓库和分支，脚本定期 clone 并把 skill 内容同步到本仓库
- TypeScript 脚本 (`triage.ts` + 14 个辅助模块) 打包在 `open-source-maintainer` skill 内部，由 Claude 代替用户执行
- AGENTS.md 明确列出 5 个 skill 的路径和用途，形成一份人机共读的 skill 地图
- 无 hook，无 MCP，执行面仅有 GitHub Actions CI 脚本和 skill 内置的 TypeScript

### 1.2 架构剖析

```
n-skills/
├── AGENTS.md              # skill 地图（机器可读 + 人可读）
├── CLAUDE.md              # 项目级约定
├── sources.yaml           # 外部 skill 同步来源声明
├── scripts/
│   ├── sync-external.mjs  # ⚠️ CI 同步脚本，execSync 注入点
│   └── update-registry.mjs
├── package.json           # 依赖（yaml ^2.9.0）
└── skills/
    ├── automation/dev-browser/   # 外部 skill（SawyerHood/dev-browser）
    ├── tools/gastown/            # 本地 skill
    ├── tools/zai-cli/            # 外部 skill
    ├── workflow/orchestration/   # 本地 skill（含 12 个 references/*.md）
    └── workflow/open-source-maintainer/  # 本地 skill（含 triage.ts + 14 模块）
        └── skills/open-source-maintainer/
            ├── SKILL.md
            └── scripts/triage.ts  # TypeScript，由 Claude 执行
```

- **文件类型分布**：5 个 SKILL.md / 5 个 plugin.json / 2 个 Node.js CI 脚本 / 1 个 TypeScript 运行时 skill
- **编排关系**：AGENTS.md 平铺列出 5 个 skill，无层级路由，用户自行选择。skill 之间无相互依赖
- **跨件契约**：AGENTS.md 中的 `<location>` 路径与实际 SKILL.md 文件路径一一对应（audit 验证通过）；`references/*.md` 文件全部存在

### 1.3 设计思路 / 方法论

- **核心设计哲学**：外部 skill 聚合优于复制粘贴——通过 YAML 声明依赖、脚本同步内容，让上游更新自动流入
- **解决什么问题**：单个开发者同时维护多个 skill 时，版本跟踪和发布流水线的重复劳动问题
- **trade-off**：自动同步降低维护成本，但引入了 YAML 来源作为信任边界——如果 `sources.yaml` 被篡改，CI 会执行恶意代码
- **认知模型**：作者把 skill 视为"可独立安装的工具"，AGENTS.md 是工具目录，plugin.json 是每个工具的产品卡片

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「YAML 声明 + CI 同步的外部 skill 聚合中枢」

核心特征：
- 特征 1：上游 skill 来源用 `sources.yaml` 声明，脚本自动 clone 同步——内容与配置分离
- 特征 2：每个 skill 有独立的 `plugin.json`，可单独安装，也可整包安装
- 特征 3：AGENTS.md 兼具人类文档和机器可读 skill 目录的双重角色
- 特征 4：TypeScript skill 脚本与 SKILL.md 共存，SKILL.md 描述调用协议，脚本实现逻辑

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要聚合多个上游 skill 并统一发布 | ✅ 高度适用 | 声明式同步减少手工维护 |
| skill 之间需要相互调用或共享上下文 | ❌ 不适用 | 平铺架构无路由机制，skill 彼此独立 |
| 需要在 skill 内执行复杂逻辑 | ✅ 适用 | triage.ts 模式证明 TypeScript 脚本可打包进 skill |
| 团队协作、多人同时维护不同 skill | ✅ 适用 | 每个 skill 是独立目录，合并冲突少 |
| 对 CI 安全有严格要求 | ⚠️ 谨慎 | sources.yaml 是信任边界，存在注入风险 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| YAML 声明同步（n-skills 模式） | numman-ali/n-skills | 上游更新自动流入，配置即文档 | 信任边界薄弱，execSync 注入风险 |
| 手工复制维护 | 大多数个人 skill 仓库 | 简单，无额外工具链 | 版本跟踪全靠人工，上游更新易漏掉 |
| npm 包管理分发 | 无标准 Claude Code 惯例 | 版本精确可重现，有签名 | 需要发布到 npm，流程重 |

### 2.4 改进空间

1. **当前问题**：`sync-external.mjs` 用字符串插值拼接 `execSync` 命令，`ref` 和 `repoUrl` 来自 `sources.yaml`，存在 shell 注入风险。**改进做法**：改用 `execFileSync('git', ['clone', '--depth', '1', '--branch', ref, repoUrl, repoDir])` 传参数组。**预期收益**：即使 sources.yaml 被污染，shell 元字符也无法执行。
2. **当前问题**：`dev-browser/SKILL.md` 只有 19 行，只说"跑 `--help`"，没有命令参考表和任何示例。**改进做法**：加一个 3-5 行的 Quick Start 示例 + 常用命令表。**预期收益**：NLPM 质量分从 80 提升到 90+，用户上手时间缩短。
3. **当前问题**：`yaml` 依赖用 caret 范围 `^2.9.0`，允许小版本自动升级。**改进做法**：锁定为精确版本 `2.9.0`，CI 用 `npm ci` 而非 `npm install`。**预期收益**：排除供应链静默升级风险。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **96/100**。加权平均：11 个 artifact，其中 5 个 SKILL.md 贡献分数拉低，6 个 plugin.json 均满分。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/automation/dev-browser/SKILL.md | 80 | 极薄：只有 Installation + "run --help"，无示例无行为指引 |
| skills/tools/zai-cli/SKILL.md | 93 | 缺 AI 行为层（何时用 vision vs search） |
| skills/workflow/open-source-maintainer/SKILL.md | 94 | 模糊量词：materially / long-term / low |
| skills/workflow/orchestration/SKILL.md | 94 | 模糊量词：interesting / smart / beautiful |
| skills/tools/gastown/SKILL.md | 98 | 轻微模糊语言 |
| 6 个 plugin.json + CLAUDE.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **AGENTS.md 作为机器可读 skill 目录**：列出 skill 路径（`<location>`）+ 适用场景，审计工具和 Claude 都能直接解析。我的 gstack 有 AGENTS.md 但格式不一致，这是一个可以对齐的点。
2. **plugin.json 与 SKILL.md 的双重声明**：`name`/`description` 在两个文件中同步，marketplace 索引和 Claude 上下文各取所需——字段不重复但都有效。
3. **reference 文件系统**：orchestration skill 引用 12 个 `references/*.md`，open-source-maintainer 引用 8 个——把长内容切分为独立文档，SKILL.md 保持精简。
4. **TypeScript skill 脚本打包**：triage.ts 和 14 个辅助模块放在 skill 目录内，SKILL.md 通过 `npx tsx` 调用——逻辑和协议分离，skill 不膨胀。
5. **跨件一致性良好**：AGENTS.md 路径、plugin.json 内容、references/*.md 文件存在性全部 audit 验证通过，零悬挂引用。

### 3.3 当时的缺陷

1. **dev-browser SKILL.md 极薄**：只有"安装 + run --help"，没有命令参考、没有示例、没有 AI 行为指引（何时截图 vs 何时提取文本）。根本原因：这是外部 skill 的同步副本，作者只做了 frontmatter，没有在聚合时补充行为层。这告诉我：聚合外部 skill 时，不能只 clone 原内容，还需要补充聚合方的"使用协议"。**我有没有犯？** 我的 graphify 里的部分 skill 也只有接口说明没有行为指引，有同类风险。
2. **execSync 字符串插值**：shell 注入风险在 CI 脚本中，`sources.yaml` 是攻击面。根本原因：Node.js `execSync` 用模板字符串比传参数组直觉更自然，但安全性相差悬殊。**我有没有犯？** gstack 的 `capture-parity-baseline.ts` 中有 `execSync('git rev-parse --short HEAD')` 但参数是硬编码字符串，风险低。值得学习的是用参数数组的好习惯。
3. **模糊量词（interesting / smart / beautiful）**：orchestration skill 里这些词作为 persona 描述词出现。根本原因：作者写 persona 时不自觉用了形容词，但从 NL 编程角度这些词给不了 Claude 可执行的指导。**自查**：我应该检查我的 skill 里是否有类似描述性语言代替行为规范。

### 3.4 当时的优化机会

1. **dev-browser SKILL.md**：加入核心命令列表（navigate、click、screenshot、eval 各 1 行示例），以及"何时用 dev-browser 而不用 WebFetch"的决策树
2. **zai-cli SKILL.md**：补充 AI 行为层——`vision analyze` vs `search` vs `read` 的选择规则，以及认证失败时的处理流程
3. **open-source-maintainer SKILL.md**：把"materially"（line 22）改为具体阈值，如"超过 3 个文件改动或逻辑分支改变时才重新规划"

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| dev-browser SKILL.md 极薄 | `wc -l skills/automation/dev-browser/skills/dev-browser/SKILL.md` | **持续存在**（仍 19 行，内容未变） | 77 天无改善，作者优先级低于其他 skill |
| execSync 字符串插值注入 | `grep -n "execSync" scripts/sync-external.mjs` | **持续存在**（lines 128, 134 仍是字符串模板） | 安全债未还，CI 流水线每次同步都承担风险 |
| yaml 依赖未固定 | `grep yaml package.json` | **部分改善**（`^2.8.3` → `^2.9.0`，caret 范围仍在） | 版本升级了但锁定策略未变，仍是范围依赖 |

### 4.2 架构演进

Audit 时 11 个 artifact；现在目录结构与当时一致，无新 skill 加入、无删除。作者似乎处于维护模式而非扩张期——稳定性高，但也意味着质量债在积累。

### 4.3 新增的可学习模式

暂无。当前 HEAD 与 audit 时的结构高度一致，未发现显著架构演进。

---

## 五、校准

### 5.1 我已经在做对的

1. bureau 的 `.claude/agents/auditor.md` 明确声明了 `tools: Read, Grep, Glob`，没有 n-skills droids 的"无工具声明"问题
2. gstack 的 `review/checklist.md` 明确写出了 `subprocess.run` + `shell=True` + f-string 是安全反模式——我已有这份意识
3. gstack 的 allowed-tools 覆盖率（129 处命中）远高于大多数同类仓库

### 5.2 挑战 / 验证

这个案例验证了一个我之前半信半疑的观点：**聚合外部 skill 时，聚合者有责任补充行为层**。n-skills 的 dev-browser 证明了只 clone 不补充的副作用——一个 80 分的 skill 被打包进了一个整体 96 分的套件，成为最弱的一环。如果我做类似的 skill 聚合，我必须在 sync 脚本之外加一层"行为层覆盖"机制。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 里有无只有"安装 + run --help"的薄 skill
for f in $(find ~/.claude/skills -name "SKILL.md" 2>/dev/null); do
  lines=$(wc -l < "$f")
  if [ "$lines" -lt 25 ]; then echo "THIN ($lines lines): $f"; fi
done
```
命中后怎么办：每个命中文件加一个 "## Quick Start" section（3 行示例）+ "## When to Use This vs Other Tools" 决策说明。

```bash
# 检查我的项目里有没有 execSync + 字符串模板（潜在注入）
grep -rn "execSync\`" ~/.claude/ ~/projects/ 2>/dev/null | grep -v "node_modules"
```
命中后怎么办：改为 `execFileSync(cmd, [...args])` 形式，参数数组化。

### 6.2 灵感 → 实施路径

1. **想法**：给 graphify 的 skill 同步机制加一个"行为层覆盖"文件（`skill-overlay.md`），在 clone 上游内容后 merge 本地行为指引
   - **为何可行**：n-skills 证明了问题所在，解法就是在聚合层加一个钩子
   - **第一步**：在 graphify 的 `tools/skillgen/` 下新建 `overlay/` 目录，放 `dev-behavior.md`，在 skillgen 生成时合并。约 30 分钟可完成。

2. **想法**：参考 n-skills 的 AGENTS.md 格式，给 bureau 加一个机器可读的 crew 目录文件
   - **为何可行**：bureau 已有 `crew/` 目录，缺的是一个统一的入口声明文件
   - **第一步**：在 bureau 根目录新建 `AGENTS.md`，列出所有 `crew/*/agent.md` 的路径和一句话描述。约 15 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 numman-ali/n-skills 的核心目的**：聚合多个 Claude Code skill 为统一可安装套件，外部 skill 通过 YAML 声明同步

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | 都是多 skill 聚合，有 AGENTS.md 入口 | graphify 是代码图谱工具，n-skills 是工具合集 | 高 |
| MarkQWu/drama-workshop-skills | 高 | 都是多 skill monorepo，有安装脚本 | drama 专注戏剧创作领域，n-skills 通用工具 | 高 |
| MarkQWu/bureau | 中 | 都有 agents/ 目录和声明文件 | bureau 是知识库系统，n-skills 是 skill 套件 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| dev-browser 极薄 SKILL.md | `wc -l graphify/skill*.md` | graphify/skill-devin.md 只有 `allowed-tools:` 一行内容 → 命中 | 高 |
| execSync 字符串插值 | `grep -n "execSync\`" /tmp/my-repos/MarkQWu-gstack/test/` | gstack 测试文件命中 3 处，但参数均为硬编码常量，不涉及外部输入 | 低 |
| yaml/npm 依赖未固定 | `grep -n "\\^" /tmp/my-repos/MarkQWu-gstack/package.json` | gstack package.json 使用 caret 范围依赖 | 低 |

**命中后的具体行动建议**：
- `MarkQWu/graphify` 的 `graphify/skill-devin.md` → 补充 Quick Start 示例（5 行）+ 何时使用此 skill 的决策说明 → 30 分钟可完成

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：外部 skill 同步声明化
   - **本案例做法**：`sources.yaml` 明确声明上游 skill 仓库 URL、分支、路径，脚本 clone 同步。文件即文档，PR diff 即变更记录（`sync-external.mjs`）
   - **我的项目现状**：drama-workshop-skills 的 `install.sh` 是手工复制模式，无上游来源声明；graphify skill 完全本地，无聚合机制
   - **如何借鉴**：在 drama-workshop-skills 根目录加 `sources.yaml`，列出每个 skill 的"灵感来源"或"上游参考"，即使不做自动同步也能记录溯源

2. **领域**：reference 文件系统
   - **本案例做法**：orchestration skill 引用 12 个 `references/*.md`，SKILL.md 精简为调用协议
   - **我的项目现状**：graphify/graphify/skills/kilo/ 已有类似的 references/ 目录，但 drama-workshop-skills 里 references 全在一个目录，没有分 skill 组织
   - **如何借鉴**：把 drama-workshop-skills 的 references/ 按 skill 归属分组到各自子目录

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：agent tools 声明的精确性
- **我的做法**：bureau/.claude/agents/auditor.md 明确声明 `tools: Read, Grep, Glob`，遵循最小权限原则
- **本案例做法**：n-skills 没有 agent 文件（所有入口是 skill 而非 agent），无对比面
- **意义**：bureau 的 agent 声明模式是比 n-skills 更成熟的封装，若未来 n-skills 加入 agent 编排，这是可以贡献的模式

---

## 八、术语表

- **插件**（plugin）：Claude Code 可以通过 `claude plugin install <name>@<owner>` 安装的 skill 包。一个仓库可以包含多个 plugin，每个 plugin 有对应的 `plugin.json` 描述文件。
- **[SKILL.md](#skill-md)**：Claude Code skill 的核心文件，用自然语言描述 AI 的行为协议——何时触发、做什么、不做什么、输出格式。
- **[plugin.json](#plugin-json)**：每个 Claude Code plugin 的 manifest（清单）文件，包含 `name`、`description`、`version` 等字段，供 marketplace 索引。
- **[AGENTS.md](#agents-md)**：部分仓库用于列出可用 agent/skill 及其路径的机器可读目录文件，Claude Code 在读取项目上下文时会解析它。
- **[frontmatter](#frontmatter)**：Markdown 文件顶部 `---` 之间的 YAML 块，NLPM 用它识别 artifact 类型和元数据（如 `name`、`description`、`allowed-tools`）。
- **execSync**：Node.js `child_process` 模块中同步执行 shell 命令的函数。字符串形式（ `` execSync(`git clone ${url}`) `` ）会把字符串交给 shell 解释器，存在 shell 注入风险；参数数组形式（`execFileSync('git', ['clone', url])`）不经过 shell，安全。
- **execFileSync**：Node.js 中安全执行外部程序的函数，接受命令和参数数组而非 shell 字符串，不触发 shell 元字符解析。
- **供应链风险**（supply chain risk）：通过依赖包或同步的上游代码引入恶意内容的风险。`npm install` 时不锁定版本、或从未验证的 URL 直接执行脚本，都是供应链风险的来源。
- **caret 范围**（caret range）：`package.json` 中以 `^` 开头的版本约束，如 `^2.9.0` 表示允许安装 2.x.x 范围内的任何版本。对比精确锁定（`2.9.0`）更易受静默升级影响。
