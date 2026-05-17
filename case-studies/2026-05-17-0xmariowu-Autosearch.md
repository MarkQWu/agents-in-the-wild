# 0xmariowu/Autosearch — 学习案例

**仓库**：https://github.com/0xmariowu/Autosearch
**分析日期**：2026-05-17
**NLPM 评分**：92/100

## 仓库概览

Autosearch 是一个开源的深度研究工具，专为编程代理设计，支持跨 50+ 信息源（Zhihu、Bilibili、arXiv、HackerNews、Reddit、LinkedIn、GitHub 等）并发搜索与聚合。核心 NL 工件是按渠道划分的 SKILL.md 文件（channel skills）和跨渠道的 meta skills（路由、引用索引、体验积累、搜索反思循环等），另有一个 `.github/agents/` 目录定义 CI 用代理。整体设计风格是"协议化"——每个渠道 skill 遵循统一的 frontmatter schema（methods/fallback_chain/experience_digest 等），meta skills 定义跨渠道的编排协议。安全评级为 **BLOCKED**（npm 安装脚本存在 High 安全问题）。

## 质量评分

| 文件 | 分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 50 | 非 NL 工件格式，无 frontmatter，无调用示例 |
| commands/autosearch.md | 70 | 缺少必需的 `name` frontmatter 字段 |
| .claude-plugin/plugin.json | 75 | JSON 配置，无 NL prose body |
| autosearch/skills/channels/twitter/SKILL.md | 88 | 缺 When to Choose / How To Search / Known Quirks 区块 |
| autosearch/skills/channels/tieba/SKILL.md | 88 | body 部分中英混排，结构指导最小化 |
| autosearch/skills/channels/linkedin/SKILL.md | 88 | 正文三段话，Known Quirks 仅两个占位 bullet |
| autosearch/skills/channels/hackernews/SKILL.md | 93 | How To Search 三个 bullet 完全重复同一方法名 |
| autosearch/skills/channels/douyin/SKILL.md | 94 | body 描述了 `via_douyin_mcp` 方法但 frontmatter methods 未声明 |
| autosearch/skills/meta/channel-selection/SKILL.md | 97 | Clean |
| autosearch/skills/router/SKILL.md | 97 | Clean |

评分维度：触发词清晰度高（router/meta skills 普遍 96-97），channel skills 体验区块设计系统化；主要扣分项是部分渠道 skill 缺少结构化区块，以及 CLAUDE.md 完全未接入 NLPM schema。

## 值得借鉴的模式

**渠道 skill 协议化 frontmatter** → 每个渠道 skill 用统一的 `methods/fallback_chain/experience_digest/requires_token` 字段声明能力边界，而不是在 prose 里自由描述。AI 读取 frontmatter 就知道这个渠道支持哪些搜索路径、哪个作备用、是否需要 token，不依赖解析自然语言。→ 自己的 skill 文件应尽可能把结构化元数据放进 frontmatter，而非埋在正文里。

**体验积累机制（experience_digest）** → skill frontmatter 声明 `experience_digest: experience.md`，在每次搜索后将经验摘要写入该文件并在下次调用时加载。这是一种轻量的"跨会话学习"机制，让代理随着使用变得更智能。→ 对于需要积累知识的工作场景（如代码审查、SEO 分析），可以借鉴这个模式设计一个 experience 侧车文件。

**Meta skill 作为跨渠道编排层** → `channel-selection/SKILL.md` 和 `router/SKILL.md` 把选择逻辑从各渠道 skill 中抽离，形成独立的路由层，各渠道 skill 只负责"如何搜索"而不负责"何时选择我"。→ 当有多个同类 skill 时，应设计一个选择器 skill 而非在每个 skill 里都写"什么时候用我"。

**Fallback chain 显式声明** → `fallback_chain` 字段明确定义当主方法失败时的降级顺序，而不是让 AI 自行决定。这避免了 AI 在搜索失败时随机尝试或直接放弃。→ 任何涉及外部服务的 skill 都应声明失败降级路径。

**CI 代理专门化** → `.github/agents/test-sufficiency.md` 是专门用于 CI 的代理，而非在通用代理里加条件分支。→ CI 场景的 NL 工件应与交互式场景的分开，避免混淆上下文。

## 缺陷分析

**commands/autosearch.md 缺少 `name` 字段** → 根本原因：command 文件的 frontmatter 是手写的，没有从 plugin 系统生成，也没有 lint 检查必需字段。`name` 是 NLPM scanner 注册命令的必需字段，缺失导致该命令对 NLPM 不可见。→ 命令文件不同于 skill 文件，有不同的必需字段，应有独立的 schema 验证。**自查：我的 command 文件有没有写 `name` 字段？**

**experience_digest 路径不一致** → `instagram/SKILL.md` 和 `wechat_channels/SKILL.md` 写的是 `experience_digest: experience/experience.md`（嵌套路径），而其他 50+ 个 skill 都用 `experience_digest: experience.md`（同级文件）。根本原因是这两个 skill 是后加的，没有检查已有的路径约定，也没有 schema 验证路径格式。运行时行为取决于加载器如何解析路径，存在静默失败风险。**自查：我的项目有没有类似的"路径格式惯例"未被 schema 保护？**

**npm 安装脚本 curl|bash 无校验** → `npm/bin/autosearch-ai.js` 在用户运行 `npx autosearch-ai` 时，如果检测到本地未安装，会执行 `bash -c "curl -fsSL ... | bash"` 下载并运行远程安装脚本，没有任何 checksum 验证。根本原因是为了"一键安装"体验牺牲了安全性。如果 GitHub CDN 或 install.sh URL 被劫持，用户机器会执行任意代码。这是导致安全评级 BLOCKED 的直接原因。**自查：我的安装脚本有没有从远程下载代码直接执行？**

## 优化机会

1. **问题**：commands/autosearch.md 缺 `name` 字段导致命令无法注册 **修法**：在 frontmatter 加 `name: autosearch`（或对应 slug） **预期影响**：命令从 NLPM 不可见变为可发现，高影响、零风险

2. **问题**：experience_digest 路径两个文件不一致 **修法**：将 `instagram/SKILL.md` 和 `wechat_channels/SKILL.md` 的 `experience_digest` 值改为 `experience.md`，与其他 50+ skill 保持一致 **预期影响**：消除运行时静默失败风险，中等影响

3. **问题**：npm curl|bash 无 checksum 验证（安全 High） **修法**：对 install.sh 做 SHA-256 校验再执行，或改为通过 PyPI 分发，完全移除 curl|bash 路径 **预期影响**：解除 BLOCKED 状态，解锁 PR 贡献通道

## 对照我的项目

**自查：我有没有犯同样的错？**

- **路径惯例保护**：检查自己项目里有没有"大家都约定用这个路径格式但没有 schema 校验"的地方——比如 experience.md 路径、图片引用路径、技能索引文件路径。
- **命令文件 vs. skill 文件的必需字段差异**：命令文件的 frontmatter 必需字段和 skill 文件不同，是否在项目里对两类文件分别做了 lint？
- **curl|bash 模式**：检查自己的安装脚本或 CI 脚本，是否有从远程下载代码直接 pipe 给 bash/sh 的模式？如果有，是否做了 checksum 验证？
- **体验积累机制**：Autosearch 的 `experience_digest` 模式值得借鉴——哪些自己常用的工作流可以从跨会话积累中受益？
- **Fallback chain 声明**：自己的 skill 如果依赖外部服务，是否显式声明了降级路径？

## 灵感

- **"渠道协议"设计模式**：在自己的多渠道/多工具 skill 集里引入 frontmatter schema 协议（methods/fallback/requires），把能力边界从 prose 里提取出来，让路由层可以机械地读取而不是解析自然语言。
- **体验侧车文件**：为长期使用的 skill（如代码审查、SEO 分析）设计一个 `experience.md` 侧车，每次执行后追加学到的规律，让 skill 随使用越来越准确。
- **安装安全审查**：在发布可安装的插件/工具前，检查安装脚本是否符合最小权限原则，是否有 curl|bash 模式——这类安全问题会直接阻断外部贡献者提 PR。
