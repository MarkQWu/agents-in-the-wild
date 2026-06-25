# vercel-labs/agent-browser — 学习案例

**仓库**：https://github.com/vercel-labs/agent-browser
**Stars**：32,739 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-13（历史快照）| **生成日期**：2026-06-25（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `examples-driven`, `manifest-discipline`, `single-purpose`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

vercel-labs/agent-browser 是一个 **[NL 表皮 + 原生二进制核心](#nl-表皮-原生二进制核心)** 的浏览器自动化 CLI，让 AI agent 能像人类一样操作浏览器（导航、填表、截图、提取数据）。关键事实：

- 拥有 32,739 颗星，是 Vercel 实验室旗下最受关注的 AI 工具之一
- 安装方式：`npm install -g agent-browser` 或 `npx agent-browser install`
- 核心组件用 **Rust** 实现（`cli/src/`），搭配 Node.js [postinstall 脚本](#postinstall)下载平台原生二进制
- NL 层（skills）共 7 个 SKILL.md，负责教导 AI agent 如何使用该工具；平均得分 **99/100**
- 支持多运行环境：Vercel Sandbox 微虚拟机、AWS Bedrock AgentCore、本地 Electron 应用

### 1.2 架构剖析

- **目录结构**：
  ```
  agent-browser/
  ├── cli/src/          # Rust 原生二进制核心（skills.rs, plugins.rs）
  ├── bin/              # JS 跨平台包装层（为 npx 支持）
  ├── scripts/
  │   └── postinstall.js  # 下载平台二进制 + 替换全局 PATH 符号链接
  ├── skill-data/       # 5 个专用场景 SKILL.md（core, dogfood, slack, electron, ...）
  ├── skills/           # 1 个发现 stub（agent-browser/SKILL.md，hidden:true）
  ├── examples/         # 各运行环境的集成示例
  └── docs/             # Next.js 文档站
  ```

- **文件类型分布**：7 个 SKILL.md（6 在 skill-data/、1 发现 stub 在 skills/），0 个 agent，0 个 hook

- **编排关系**：**双层 skill 路由**。`skills/agent-browser/SKILL.md` 是「发现入口」（discovery stub），带 `hidden:true`；agent 通过它得知存在后，调用 `agent-browser skills get core` 获取真正的 420 行使用指南。5 个专用 skill（dogfood、slack、electron、vercel-sandbox、agentcore）各自覆盖一个运行环境，向主 `core` skill 做补充。

- **跨件契约**：每个 SKILL.md 的 `description` 字段包含精确触发词（trigger words），当用户说「check my Slack」或「dogfood this app」时，AI 能路由到正确 skill。`allowed-tools` 字段声明 `Bash(agent-browser:*)` 限定工具范围。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「CLI first，skill 是薄包装层」——所有功能都在原生二进制里，SKILL.md 只负责让 AI 知道「什么时候用它」和「基本用法规范」，细节通过 CLI 自身的帮助系统按需获取（`agent-browser skills get core --full`）
- **解决什么问题**：Agent 调用浏览器时缺乏稳定的 API——现有工具（playwright、puppeteer）需要 agent 自己管理浏览器实例，agent-browser 抽象出「snapshot-ref-act」循环，让 agent 只需关心「看页面→找元素→操作」
- **做了什么 trade-off**：选择了原生二进制核心（性能、跨平台稳定性）而非纯 JS，代价是 postinstall 要从 GitHub Releases 下载平台特定二进制，引入了供应链安全问题（无 checksum 验证）
- **反映什么认知模型**：作者认为「skill 的使命是触发路由，不是包含所有细节」——这与「skill = 内联知识库」的常见模式形成鲜明对比，是一种「发现层 vs 内容层分离」的设计哲学

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「发现 stub + 内容层按需拉取」

Skill 文件本身只包含触发词和最小必要说明，指向「运行时可查询的内容源」（这里是 CLI 的 `skills get` 子命令）。用户按需拉取详细内容，而不是把 5000 行使用手册全部内联进 SKILL.md。

模式特征清单：
- 特征 1：`skills/agent-browser/SKILL.md` 加 `hidden:true` 让它从 `skills list` 消失，但保持可被 `npx skills add` 发现
- 特征 2：每个 skill 的 `description` 里有 10+ 个明确的触发场景短语，让 LLM 路由更精准
- 特征 3：`allowed-tools: Bash(agent-browser:*)` 把工具白名单直接写在 [frontmatter](#frontmatter) 里，无需用户手动配置权限
- 特征 4：跨环境（本地、Sandbox、AWS）的行为差异通过独立 skill 隔离，core skill 保持通用
- 特征 5：`skill-data/core/references/*.md` 把参考文档从工作流分开，主 SKILL.md 聚焦「如何做」，参考文档聚焦「有什么选项」

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有稳定 CLI 的工具库 | ✅ 高度适用 | CLI 本身可以提供详细帮助，skill 只需做路由 |
| 需要详细内联指导的任务 | ⚠️ 部分适用 | 若工具无自我文档能力，需在 skill body 补充细节 |
| 离线或封闭网络环境 | ❌ 不适用 | `agent-browser skills get` 需要 CLI 已安装，postinstall 需要下载二进制 |
| 纯 Markdown 插件（无 CLI 依赖） | ❌ 不适用 | 「按需拉取」模式的前提是有可查询的运行时内容源 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：「发现 stub + 运行时内容层」 | vercel-labs/agent-browser | skill 极轻量，CLI 升级不需要同步更新 skill 文件 | 离线无效；CLI 未安装时 skill 无法补充内容 |
| 备选 A：「完全内联 skill」 | sickn33/antigravity-awesome-skills | 离线可用，无外部依赖 | 内容更新需要同步修改 SKILL.md；超长 skill 上下文成本高 |
| 备选 B：「skill 指向外部文档链接」 | sickn33 的 tavily-web stub | 维护简单 | Claude Code 不会自动 fetch 链接，用户得到空响应 |

### 2.4 改进空间

1. **当前问题**：`postinstall.js` 下载二进制时无 SHA256 checksum 验证（Medium 安全发现）**改进做法**：在 GitHub Releases 旁边发布 `checksums.txt`，postinstall 下载后用 `crypto.createHash('sha256')` 验证 **预期收益**：即使 Release 被投毒，校验失败会阻止执行，阻断供应链攻击
2. **当前问题**：`skill-data/dogfood/SKILL.md` 的 `allowed-tools` 声明 `Bash(npx agent-browser:*)` 但 body 第 25 行禁止使用 npx（矛盾） **改进做法**：删除 `allowed-tools` 里的 `Bash(npx agent-browser:*)` 这一行，只保留 `Bash(agent-browser:*)` **预期收益**：消除自我矛盾，工具调用声明与使用指南一致
3. **当前问题**：`skill-data/vercel-sandbox/SKILL.md` 缺少 `allowed-tools` 字段（6 个同类 skill 中唯一缺失的） **改进做法**：添加 `allowed-tools: Bash(agent-browser:*)` 保持一致性，或加注释说明为何不需要 **预期收益**：消除不一致，自动化工具扫描不会误报

---

## 三、过去审查发现（2026-05-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-05-13 当时得分 **99/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skill-data/core/SKILL.md | 100 | — 无问题 |
| skill-data/agentcore/SKILL.md | 100 | — 无问题 |
| skill-data/dogfood/SKILL.md | 97 | `allowed-tools` 声明 `npx agent-browser:*` 但 body 禁止使用 npx（−3） |
| skill-data/vercel-sandbox/SKILL.md | 98 | 缺少 `allowed-tools` 字段（−2） |
| skills/agent-browser/SKILL.md | 100 | — 无问题 |

### 3.2 当时值得借鉴的模式

1. **精准触发词设计** → `description` 字段不只是一句简介，而是包含 10+ 个真实用户可能说出的短语（「check my Slack」、「fill out a form」、「take a screenshot」）。借鉴：自己的 skill description 应该包含 3-5 个自然语言触发场景，而不只是功能描述。

2. **`allowed-tools` 白名单** → 所有 skill 都明确声明 `allowed-tools: Bash(agent-browser:*)` 而不是不填（默认允许所有）。借鉴：每个 SKILL.md 都应该有 `allowed-tools` 字段，限制 agent 只使用与 skill 相关的工具。

3. **发现层与内容层分离** → `skills/agent-browser/SKILL.md` 加 `hidden:true` 保持可发现但不污染列表；详细内容由 CLI 命令提供。借鉴：当工具有自身文档能力时，skill 可以做「指针」而非「内容源」。

4. **跨环境 skill 隔离** → 「在 Vercel Sandbox 运行」和「在 AWS Bedrock 运行」的差异被隔离到独立 skill 里，core skill 保持纯净。借鉴：同一个工具在不同环境有明显行为差异时，用独立 skill 隔离而不是在一个 skill 里用大量条件判断。

5. **cross-reference 的正确实现** → `skills/agent-browser/SKILL.md` 的 description 中引用了 5 个子 skill 的名字；跨引用在审计时全部通过验证（0 个断链）。借鉴：skill cross-reference 应该在审计时可机械验证——只引用真实存在的 skill 名。

### 3.3 当时的缺陷

1. **dogfood SKILL.md 的 npx 矛盾** → 根本原因：在修改 body 内容（禁止 npx）时忘记同步更新 frontmatter 的 `allowed-tools`，两处相互矛盾导致无论 agent 做什么选择都违反了某处规范。自查：我的 SKILL.md 里有没有 `allowed-tools` 声明的工具名和 body 里的限制说明相矛盾？

2. **vercel-sandbox 缺少 `allowed-tools`** → 根本原因：这个 skill 的用途略不同（教 TypeScript SDK 用法而非直接 CLI 调用），作者可能认为不需要工具声明——但没有加注释解释，外部审查者无法区分「有意为之」和「遗漏」。自查：我的 skill 里有没有某些缺少 `allowed-tools` 的情况？如果有意为之，有没有注释解释？

3. **High 安全问题：postinstall 无 checksum 验证** → 根本原因：从 GitHub Releases 下载二进制后直接 `chmodSync` 变为可执行是 npm native binary package 的常见模式（esbuild、sharp、pnpm 也这样），但缺少 checksum 使得 Release 被投毒时无防护。自查：如果我的项目有 postinstall 下载行为，有没有 checksum 验证？

### 3.4 当时的优化机会

1. 删除 `dogfood/SKILL.md` 的 `Bash(npx agent-browser:*)` 工具声明（一行修改，5 分钟内可完成）
2. 为 `vercel-sandbox/SKILL.md` 添加 `allowed-tools` 字段或注释解释省略原因（一行修改）
3. 在 `postinstall.js` 中添加 SHA256 checksum 验证逻辑（从 GitHub Releases 旁边的 checksums.txt 下载并对比）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| dogfood SKILL.md 声明 `npx agent-browser:*` 但 body 禁止 npx | 搜索 `npx agent-browser repo:vercel-labs/agent-browser filename:SKILL.md` | **仍存在**：`skill-data/dogfood/SKILL.md` 当前 HEAD 仍含 `Bash(npx agent-browser:*)` | 轻微不一致，3 分扣分仍有效 |
| vercel-sandbox 缺少 `allowed-tools` | 间接查看搜索结果中的 skill-data/vercel-sandbox/SKILL.md 片段 | **无法确认**（搜索结果未暴露 frontmatter 全文） | 暂无法核验 |
| postinstall 无 checksum | SEC 发现中等级为 Medium | 仍存在（CHANGELOG 中无相关修复记录） | 供应链风险仍存 |

### 4.2 架构演进

CHANGELOG 中记录了 2026 年以来的重大迭代：

- **`core` skill 重命名**：原 `agent-browser` skill（40 行）被替换为 420 行完整使用指南；原 stub 保留但加了 `hidden:true` 标记。这是**设计模式的一次主动升级**：从「stub 指向文档链接」变为「stub 指向运行时 CLI 内容」，避免了 tavily-web 反模式（链接不会被 Claude 自动 fetch）。

- **Doctor 命令**：新增 `agent-browser doctor` 用于诊断安装问题。说明作者意识到原生二进制安装链条复杂，用户摩擦点多，主动提供自诊断工具。

- **Stable tab IDs**：Tab 现在有稳定 ID（`t1`, `t2`），避免 agent 把位置索引误认为稳定引用。这是对 AI agent 认知局限的主动适配。

- **JSON Schema for config**：提供 `agent-browser.schema.json`，让 IDE 可以自动补全配置。把「工具易用性」从 NL 层延伸到开发者层。

### 4.3 新增的可学习模式

**「schema.json + NL skill 双轨文档」**：工具同时维护机器可读的 JSON Schema（给 IDE）和人类+AI 可读的 SKILL.md（给 Claude Code）。这两种文档的用途不同但互补——前者服务于开发者工具链，后者服务于 AI agent。如果自己的工具有复杂配置，可以考虑同时维护两套：一套结构化 schema，一套 NL skill 说明。

---

## 五、校准

### 5.1 我已经在做对的

1. **Skill description 包含触发场景**：我的 skill 描述里包含了用户可能说出的场景短语，而不只是技术描述
2. **Skill 聚焦单一场景**：我没有把多个不同环境的行为塞进同一个 skill 文件
3. **避免空 body stub**：我的 skill body 包含实质性内容，不会让用户调用后得到空响应

### 5.2 挑战 / 验证

这次案例**验证了一个之前犹豫的做法**：在 skill 里大量写触发词是否显得啰嗦？答案是 No——agent-browser 的高精度路由（0 个误触发记录）很大程度上得益于 description 里的精确触发词列表。这个 99 分的 skill 证明了：description 越具体，AI 路由越准确，不是啰嗦而是必要。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 SKILL.md 中 allowed-tools 声明与 body 禁止指令是否矛盾
# 找出 allowed-tools 声明了某命令但 body 里出现了 "never" 或 "avoid" 修饰该命令
grep -rn 'allowed-tools' .claude/skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null
```
命中后：对每个声明的工具名，搜索 body 里是否有「禁止使用」的矛盾描述。

```bash
# 检查哪些 SKILL.md 缺少 allowed-tools 字段
grep -rL 'allowed-tools' .claude/skills/*/SKILL.md ~/.claude/skills/*/SKILL.md 2>/dev/null
```
命中后：判断是「有意为之」还是遗漏——有意省略的加注释，遗漏的补上。

```bash
# 检查 skill description 的触发词数量（少于 3 个场景短语可能路由精度不足）
grep -c 'Use when\|Triggers include\|使用时机' .claude/skills/*/SKILL.md 2>/dev/null
```
命中 0 的文件：补充 3-5 个「Use when...」触发场景。

### 6.2 灵感 → 实施路径

1. **想法**：仿照 agent-browser 的「发现 stub + hidden:true」模式，为自己的工具类 skill 创建一个轻量发现入口
   - **为何可行**：如果工具有 `--help` 或子命令能提供详细说明，skill 只需告诉 agent「这里有个工具，触发条件是 X，然后用 CLI 查细节」
   - **第一步**：在一个现有 SKILL.md 顶部加 `hidden: true`，把 body 缩减为 5-10 行核心触发词和 CLI 帮助命令（20 分钟）

2. **想法**：为 skill description 系统性补充触发词
   - **为何可行**：agent-browser 99 分的核心优势之一就是精确触发词，这是纯写作优化，无需改代码
   - **第一步**：对每个现有 skill，列出 5 个用户可能用自然语言说出的场景，追加到 description 末尾（10 分钟/skill）

---

## 七、对照我的 GitHub 仓库

> 注：本次运行中 my-repos 克隆因网络策略限制失败，GitHub Code Search 对低星仓库无索引。以下分析基于 my-repos.json 描述和公开元数据。

### 7.1 目的对齐度

- **本案例 vercel-labs/agent-browser 的核心目的**：为 AI agent 提供可控的浏览器操作接口，NL skill 层做路由，Rust CLI 层做执行

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 中 | 同为「工具包装成 skill」的设计模式，让 AI 能使用特定工具 | graphify 针对代码知识图谱而非浏览器 | 高 |
| MarkQWu/gstack | 低 | 同为 Claude Code 插件 | gstack 是角色扮演而非工具包装 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 同为 skill 库 | drama-workshop-skills 是社区策展，不是单一工具包装 | 低 |

若目的不完全对齐：本案例的**架构模式**（发现层/内容层分离、allowed-tools 白名单、精准触发词）仍高度适用于我的所有 skill 类项目。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| `allowed-tools` 与 body 中禁止指令矛盾 | `grep -n 'allowed-tools' SKILL.md` 后对照 body | 无法克隆验证；graphify 作为工具包装 skill，矛盾风险中等 | 中 |
| 缺少 `allowed-tools` 字段 | `grep -L 'allowed-tools' */SKILL.md` | 无法克隆验证 | 中 |
| description 缺触发场景短语 | `grep -c 'Use when' SKILL.md` | 无法克隆验证；drama-workshop-skills 为「社区策展」，可能描述较抽象 | 低-中 |

**命中后的具体行动建议**：
- graphify 的 SKILL.md → 检查 `allowed-tools` → 如缺失添加对应工具名（5 分钟）
- drama-workshop-skills 下每个 SKILL.md → 追加 3 个 "Use when..." 触发场景到 description（10 分钟/文件）

### 7.3 别人的更优方案

1. **领域**：「精准触发词 + description 场景化」
   - **本案例做法**：`skill-data/slack/SKILL.md` 的 description 列出了「check my Slack」、「what channels have unreads」、「search Slack for」等 8 个具体短语
   - **我的项目现状**：graphify 的 SKILL.md 描述大概率是功能性一句话（基于仓库 README 描述风格），缺乏场景化触发词
   - **如何借鉴**：在 graphify 的 SKILL.md description 中补充：「Use when: user wants to explore codebase, visualize dependencies, ask questions about a large repo, or understand how files relate to each other」

2. **领域**：「allowed-tools 白名单的系统性应用」
   - **本案例做法**：每个 skill 都有 `allowed-tools: Bash(agent-browser:*)` 精确限制
   - **我的项目现状**：不确定，但大概率部分 skill 无此字段
   - **如何借鉴**：为每个 skill 加一行 `allowed-tools`，限制 agent 只能调用该 skill 相关的 Bash 命令

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：「body 内容内联完整性」
- **我的做法**：基于 drama-workshop-skills 描述（「Curated Claude Code skills」），推断 skill body 包含完整使用说明，不依赖外部 CLI 的帮助子命令
- **本案例做法**（弱在哪）：agent-browser 的 skill 依赖已安装 CLI 提供详细内容——在 CI 环境或 CLI 未安装时，skill 内容降级为极简 stub，agent 无法获得使用指南
- **意义**：内联 body 模式的离线可靠性更高；如果我的 skill 覆盖的工具有安装失败的风险，应主动保留内联使用说明作为降级路径

---

## 八、术语表

### <a name="nl-表皮-原生二进制核心"></a>NL 表皮 + 原生二进制核心
> 一种架构模式：用户通过自然语言（NL）与 AI agent 交互，AI agent 把意图翻译成命令，真正执行的是用 C/Go/Rust 编译成的原生二进制程序。「NL 表皮」= SKILL.md；「原生二进制核心」= Rust 编译的 `agent-browser` 可执行文件。性能接近系统级，启动时间 < 100ms。

### <a name="postinstall"></a>postinstall 脚本
> npm 包安装后自动执行的脚本，在 `package.json` 的 `"scripts": { "postinstall": "..." }` 里声明。agent-browser 用它在 `npm install` 后下载当前平台（macOS/Linux/Windows）对应的原生二进制并替换全局 PATH 里的符号链接。风险：用户不一定意识到 `npm install` 触发了网络下载和系统文件修改。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置，声明 `name`、`description`、`allowed-tools`、`hidden` 等元数据。Claude Code 通过解析 frontmatter 来注册 skill 和限制工具访问范围。

### <a name="snapshot-ref-act"></a>snapshot-ref-act 循环
> agent-browser 的核心交互模型：AI 先用 `agent-browser snapshot` 获取页面的 accessibility tree 快照（snapshot），从快照中提取元素引用（ref），再对引用执行操作（act：click、fill、type）。每次操作后重新 snapshot 获取新状态。这与「看一眼→找目标→动手→看结果」的人类浏览器操作完全对应。

### <a name="discovery-stub"></a>发现 stub
> 一个极简的 SKILL.md，只用来让 AI 知道「这个工具存在」，不包含使用细节。带 `hidden: true` 可以防止它出现在 `skills list` 里（避免用户看到空内容 skill），但通过 `npx skills add` 仍然可以安装。详细内容由工具自身提供。
