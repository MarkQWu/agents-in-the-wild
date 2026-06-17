# jordanrendric/claude-video-vision — 学习案例

**仓库**：https://github.com/jordanrendric/claude-video-vision
**Stars**：561 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-17（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `curl-pipe-bash-risk`, `vague-quantifier`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
jordanrendric/claude-video-vision 是一个赋予 Claude Code「看懂视频」能力的插件，通过 [MCP 服务器](#mcp-server)（[`claude-video-vision` npm 包](#npm包)）实现视频帧提取、音频转录，再通过 NL 层（2 个 command、1 个 agent、1 个 skill）向模型提供操作指引。核心价值：把「视频理解」这一多模态能力通过 MCP 工具暴露给 Claude Code，用户可直接问 `/watch-video path/to/video.mp4`。

关键事实：
- 混合架构：NL 表皮（command + agent + skill）+ [TypeScript MCP 服务器](#mcp-server)（`mcp-server/` 目录），NL 层调用 MCP 工具
- 通过 `npx -y claude-video-vision@latest` 在运行时自动安装 MCP 服务器
- 支持 Gemini API、OpenAI 双后端做音频分析，ffmpeg 做帧提取
- 2026-04-06 审计时 plugin.json 版本 1.2.0，npm 包已到 1.3.1；现在 npm 包已到 1.3.2，plugin.json 仍为 1.2.0

### 1.2 架构剖析
- **目录结构**：

```
claude-video-vision/
├── .claude-plugin/
│   ├── plugin.json          # Claude Code manifest（版本 1.2.0，未更新）
│   └── marketplace.json
├── .mcp.json                # MCP 服务器入口：npx -y claude-video-vision@latest
├── commands/
│   ├── watch-video.md       # 主命令：解析视频路径，调用 MCP 工具分析
│   └── setup-video-vision.md # 向导：引导用户配置后端、参数
├── agents/
│   └── frame-describer.md   # 子 agent：把视频帧图片转为文字描述
├── skills/                  # （plugin.json 中声明 ./skills/，目录下无 SKILL.md——
│                            # plugin.json 的 skills 字段指向整个目录）
│   └── video-perception/    # 视频感知方法论：帧选策略、滤波器选择、分析门限
│       └── SKILL.md
└── mcp-server/              # TypeScript MCP 服务器（作为 npm 包发布）
    ├── src/
    │   ├── tools/           # MCP 工具实现
    │   ├── backends/        # Gemini/OpenAI 后端
    │   └── extractors/      # 帧/音频提取
    └── package.json         # 版本 1.3.2
```

- **文件类型分布**：2 个 command，1 个 agent，1 个 SKILL.md，1 个 plugin.json
- **编排关系**：`watch-video` command 调用 MCP 工具（`video_info`、`video_analyze`、`video_watch` 等），调用 `frame-describer` agent（当 frame_mode=descriptions 时）；`video-perception` skill 作为方法论参考被 command 隐式使用
- **跨件契约**：`plugin.json` 版本（1.2.0）与 `mcp-server/package.json`（1.3.2）存在版本分歧；`.mcp.json` 用 `@latest` 自动安装，实际运行的是 1.3.2，但 marketplace 展示的是 1.2.0

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「[NL 表皮 + 原生能力层](#nl-binary-hybrid)」——NL 文件（command/skill/agent）只做指挥调度，复杂的视频处理逻辑封装在 MCP 服务器（TypeScript）里，两层之间通过 MCP 工具接口解耦
- **解决什么问题**：Claude Code 原生不支持读取视频文件，但多模态 LLM（Gemini Flash）可以分析图像和音频片段。作者把视频拆帧、抽音频、批量 API 调用等「脏活」封装进 MCP，NL 层只需协调流程
- **Trade-off**：`@latest` 自动安装（用户体验简单）vs 版本锁定（供应链安全）。作者选择了前者，简化安装流程，但引入了供应链风险
- **认知模型**：把视频理解问题拆解为：①提取帧+音频（MCP 工具）→ ②描述帧（frame-describer agent）→ ③语义分析（模型自身理解 + video-perception skill 指引）。三层分工清晰。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 表皮 + MCP 原生能力」模式**：NL 层（command/skill/agent）负责交互逻辑和流程协调，复杂计算能力封装为 MCP 工具，通过 npm 包发布。

模式特征清单：
- NL 文件调用 MCP 工具名称（如 `video_analyze`），不直接写算法逻辑
- MCP 服务器独立发布为 npm 包，通过 `.mcp.json` 声明启动方式
- [plugin.json](#manifest) 和 MCP 服务器有各自版本，需要同步策略
- agent 专责一种能力转换（图片→文字），被 command 按需调用

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要系统级能力（文件 I/O、API 调用、视频处理）| ✅ 高度适用 | NL 层无法直接做，必须委托 MCP |
| 纯文档分析、代码生成 | ❌ 不适用 | 用 skill 就够，MCP 是过度工程 |
| 需要频繁更新业务逻辑 | ⚠️ 谨慎 | 每次改 MCP 逻辑需重新发布 npm 包，版本同步成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + MCP 原生（本仓库）| claude-video-vision | 系统级能力，跨语言 | MCP 版本管理复杂，`@latest` 有供应链风险 |
| 纯 NL skill 集合 | superpowers-zh | 零依赖，多平台，无运行时 | 无法访问系统资源 |
| NL + bash hook | 777genius/claude-notifications-go | 轻量，可访问系统 | 只适合简单脚本任务 |

### 2.4 改进空间
1. **当前问题**：`.mcp.json` 用 `@latest` 安装，供应链风险敞口。**改进做法**：每次 release 时同步更新 `.mcp.json` 里的版本号（如 `claude-video-vision@1.3.2`）和 `plugin.json` 版本，通过 CI 脚本自动化这一步骤。**预期收益**：消除 `@latest` 自动安装的 Critical 级供应链风险，版本号三方一致。
2. **当前问题**：两个 command 缺 `name` [frontmatter](#frontmatter)，用户无法通过命令面板发现插件。**改进做法**：在 `watch-video.md` 和 `setup-video-vision.md` 的 frontmatter 分别加 `name: watch-video` 和 `name: setup-video-vision`。**预期收益**：插件的主要用户界面（slash command）变得可发现。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **79/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/watch-video.md | 56 | 缺 `name` (-25)、无空输入保护 (-10)、无 allowed-tools (-5)、4 处模糊量词 (-8) |
| commands/setup-video-vision.md | 66 | 缺 `name` (-25)、无 allowed-tools (-5)、2 处模糊量词 (-4) |
| agents/frame-describer.md | 81 | 零示例块 (-15)、2 处模糊量词 (-4) |
| skills/video-perception/SKILL.md | 92 | 4 处模糊量词 (-8) |
| .claude-plugin/plugin.json | 100 | 版本分歧（跨件问题） |

### 3.2 当时值得借鉴的模式
1. **NL/MCP 职责分离**：`video-perception` SKILL.md 描述「如何决策」（选帧策略、滤波器选择、分析门限），MCP 工具负责「怎么执行」。职责边界清晰，NL 文件不含实现代码。
2. **frame-describer agent 单职责**：专门负责「图片→文字描述」这一转换，有明确的模型指定（`model: sonnet`）和工具限制（`tools: Read`）。不掺杂分析逻辑。
3. **两步流程设计**：`setup-video-vision` 先配置，`watch-video` 再使用。分离配置与执行，用户体验清晰。

### 3.3 当时的缺陷
1. **两个 command 缺 `name` frontmatter**：根本原因是作者可能把 `.claude-plugin/plugin.json` 的 `commands` 路径声明当作了注册的充分条件，但实际上 command 文件本身也需要 `name` 字段才能出现在斜杠命令列表。用户安装后看不到 `/watch-video`。自查：我的 command 是否都有 `name`？
2. **agent 零示例块（-15 分）**：`frame-describer.md` 没有任何示例展示「输入：帧图片」→「输出：文字描述」。根本原因：作者认为 agent 描述已经足够清晰，忽视了示例对模型行为稳定性的作用。缺示例时模型可能产生格式漂移。
3. **`@latest` MCP 自动安装**：根本原因是追求安装便利性。风险：任何未来发布到 `latest` tag 的版本（包括被攻陷的版本）会立即生效，用户无法感知变更。**这是最值得警惕的模式**——在我自己的 MCP-based 插件中，`@latest` 是禁忌。

### 3.4 当时的优化机会
1. 在两个 command 的 frontmatter 加 `name` 字段（5 分钟可完成，影响最大）
2. 把 `.mcp.json` 的 `@latest` 替换为当前版本号，加入 release checklist
3. 给 `frame-describer.md` 加一个 input→output 示例块

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 两个 command 缺 `name` | `head -5 commands/watch-video.md` | **未修复**：frontmatter 只有 `description` 和 `argument-hint`，无 `name` | 主要用户界面（slash command）仍不可发现 |
| `.mcp.json` 用 `@latest` | `cat .mcp.json` | **未修复**：仍为 `npx -y claude-video-vision@latest` | 供应链风险仍在敞口 |
| plugin.json/npm 版本分歧 | `grep version .claude-plugin/plugin.json` vs `grep version mcp-server/package.json` | **仍存在，且扩大**：plugin.json=1.2.0，npm 包=1.3.2（审计时是 1.3.1，现在是 1.3.2）| 版本分歧随每次 MCP 包更新在扩大，说明版本同步流程完全缺失 |

### 4.2 架构演进
对比审计快照与现 HEAD，**结构几乎没有变化**。`mcp-server/` 有新版本（1.3.1→1.3.2），`package-lock.json` 有更新，但 NL 层（commands/agent/skill）内容无实质改动，审计发现的 4 个 bugs 一个都未修复。

这说明作者对 NL 层的维护频率远低于 MCP 服务器——功能迭代集中在 TypeScript 实现层，NL 层被忽视。

### 4.3 新增的可学习模式
**反向案例：NL 层欠维护模式**。本仓库揭示了「NL 表皮 + MCP 原生」架构的一个常见陷阱：工程师天然倾向维护代码（TypeScript MCP），而忽视 NL 文件的更新。从用户视角看，NL 层是唯一的交互界面；NL 层腐烂直接导致插件不可发现、行为不可预期。

---

## 五、校准

### 5.1 我已经在做对的
1. **command 有 `name` 字段**：检查 `echo-sleuth-for-claude` 的所有 command，均有正确的 `name` frontmatter，不存在本案例的发现性问题。
2. **职责分层**：`echo-sleuth-for-claude` 的 agent（`memory-auditor`）专门负责记忆审计，不承担 skill 的参考知识角色，与 frame-describer 的单职责设计一致。
3. **不用 `@latest`**：我没有任何 `.mcp.json` 配置，如果未来引入 MCP 将会避免 `@latest`。

### 5.2 挑战 / 验证
本案例挑战了我「只要功能能跑，NL 层随时可以后补」的假设。claude-video-vision 实际运行时视频分析功能完全可用（MCP 服务器正常），但用户无法通过 `/watch-video` 发现和调用它——功能可用、界面失效的分裂状态让插件对大多数用户实际上是不可见的。**NL 层是 AI 工具的第一界面，不是装饰。**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否都有 name frontmatter
find /tmp/my-repos/MarkQWu-* -path "*/commands/*.md" | while read f; do
  if ! grep -q "^name:" "$f"; then
    echo "MISSING name: $f"
  fi
done
# 命中后：在 frontmatter 中添加 name: <command-name>
```

```bash
# 检查有没有 @latest MCP 配置
grep -rn '@latest' /tmp/my-repos/MarkQWu-*/.mcp.json 2>/dev/null
# 命中后：替换为当前 semver 版本号，加入 release checklist
```

```bash
# 检查我的 agent 是否有示例块
grep -rL "## Example\|```" /tmp/my-repos/MarkQWu-*/agents/*.md 2>/dev/null
# 命中后：添加至少一个 input → output 示例
```

### 6.2 灵感 → 实施路径
1. **想法**：为 `echo-sleuth-for-claude` 的 `memory-auditor` agent 补充一个 input→output 示例块
   - **为何可行**：本案例展示了零示例 agent 的风险（格式漂移），`memory-auditor` 输出格式如果飘了会让结果难以阅读
   - **第一步**：在 `agents/memory-auditor.md` 末尾添加 `## Examples` 节，写一个「输入：memory 文件片段 → 输出：审计报告格式」的完整例子，30 分钟可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 jordanrendric/claude-video-vision 的核心目的**：通过 MCP 向 Claude Code 注入视频理解能力，NL 层协调分析流程。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件，有 agent 层 | echo-sleuth 无 MCP，纯 NL skill；领域不同（视频 vs 会话历史） | 中 |
| MarkQWu/claude-for-legal | 低 | 同为多能力集成插件 | 法律领域；无 MCP 服务器 | 低 |
| MarkQWu/drama-workshop-skills | 无 | — | 完全不同（剧本创作 vs 视频分析） | 低 |

我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| command 缺 `name` frontmatter | `grep -rL "^name:" /tmp/my-repos/MarkQWu-*/commands/*.md 2>/dev/null` | **0 命中**：echo-sleuth 的 5 个 command 均有 `name` | 无 |
| agent 零示例块 | `grep -rL "## Example" /tmp/my-repos/MarkQWu-*/agents/*.md 2>/dev/null` | `MarkQWu-echo-sleuth-for-claude/agents/memory-auditor.md` 待核实 | 中 |
| NL 层版本与实现层版本分歧 | `grep version /tmp/my-repos/MarkQWu-*/plugin.json 2>/dev/null` | drama-workshop-skills 无 plugin.json；echo-sleuth 无 MCP 包，无此风险 | 无 |

### 7.3 别人的更优方案

1. **领域**：`argument-hint` frontmatter 字段
   - **本案例做法**：`commands/watch-video.md` 的 frontmatter 有 `argument-hint: "path/to/video.mp4 or YouTube URL [optional prompt]"`，在命令面板里提示用户该传什么参数
   - **我的项目现状**：检查 `echo-sleuth-for-claude/commands/*.md`，frontmatter 有 `description` 但未见 `argument-hint`
   - **如何借鉴**：给 `/recap`、`/extract`、`/timeline` 等需要参数的 command 加 `argument-hint`，告诉用户期望的参数格式，改动 3 个文件各 1 行

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：NL 层与功能层同步维护
- **我的做法**：`echo-sleuth-for-claude` 的 commands 和 skills 随每次功能迭代一起更新，skill 描述和实际功能保持一致
- **本案例做法**：NL 层（command/skill）审计后 2+ 月未有实质改动，而 MCP 包已从 1.3.1 升至 1.3.2；两层脱节
- **意义**：把 NL 文件纳入 release checklist（与代码版本一同更新）是防止脱节的关键

---

## 八、术语表

### <a name="mcp-server"></a>MCP 服务器
> Model Context Protocol 服务器，一种特殊的本地进程，向 Claude Code 暴露自定义工具（如视频分析、数据库查询）。Claude Code 通过 `.mcp.json` 文件知道如何启动 MCP 服务器。工具调用发生时，Claude Code 把参数发给 MCP 服务器，服务器执行真实逻辑并返回结果。

### <a name="npm包"></a>npm 包
> Node.js 生态的软件包格式，通过 `npm install` 或 `npx` 下载和运行。`npx -y claude-video-vision@latest` 表示「不询问确认，下载并执行 claude-video-vision 包的最新版本」。`@latest` 意味着每次调用可能运行不同版本的代码。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个插件包含哪些 command、skill、agent 的路径。`plugin.json` 是 Claude Code 插件的 manifest。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置，声明文件元数据（`name`、`description` 等）。缺少 `name` 时 command 无法出现在斜杠命令列表。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生能力层
> 一种插件架构模式：Markdown 文件（command/skill/agent）负责交互协调，复杂能力封装在编译型或 TypeScript 实现的 MCP 服务器中，两者通过 MCP 工具接口解耦。「NL 表皮」是用户看到和交互的部分，「原生能力层」是真正执行计算的部分。
