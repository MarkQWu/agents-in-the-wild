# jordanrendric/claude-video-vision — 学习案例

**仓库**：https://github.com/jordanrendric/claude-video-vision
**Stars**：561 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-03（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `examples-driven`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`claude-video-vision` 是一个让 Claude Code 能"看懂视频"的插件——通过 ffmpeg 提取帧、多后端处理音频，把视觉帧和带时间戳的音频转写一起送给 Claude，使其具备视频感知能力。关键事实：
- 定位是"感知层（perception layer）"而非"解释层"——插件只负责把视频拆解成 Claude 能读的数据，解释由 Claude 完成
- 支持三种音频后端：Gemini API（推荐）、本地 Whisper、OpenAI API
- 支持 YouTube URL（通过 `yt-dlp` 下载并利用 YouTube 字幕）
- 版本漂移：`plugin.json` 声明 1.2.0，但 `mcp-server/package.json` 是 1.3.1，`.mcp.json` 用 `@latest` 所以运行时实际是最新版
- 有完整的开源基础设施（CONTRIBUTING.md、CODE_OF_CONDUCT.md、SECURITY.md、CHANGELOG.md）

### 1.2 架构剖析
```
jordanrendric/claude-video-vision/
├── commands/
│   ├── setup-video-vision.md     # 配置向导 command（缺 name + allowed-tools）
│   └── watch-video.md            # 视频分析 command（缺 name + allowed-tools）
├── agents/
│   └── frame-describer.md        # 帧描述 agent（零示例块）
├── skills/
│   └── video-perception/SKILL.md # 视频感知技能（4 个模糊量词）
├── mcp-server/                   # TypeScript MCP 服务器
│   ├── package.json              # 版本 1.3.1
│   └── src/                     # 实际视频处理逻辑
├── .mcp.json                     # MCP 配置（用 @latest 自动安装）
└── .claude-plugin/plugin.json   # 插件清单（版本 1.2.0）
```
- **文件类型分布**：1 个 SKILL.md，1 个 agent，2 个 command，1 个 MCP 服务器
- **编排关系**：用户先运行 `/setup-video-vision` 配置后端，再运行 `/watch-video <path>` 分析视频；`watch-video` 内部依次调用 `video_info` → `video_analyze` → `video_watch` → `video_detail`（按需）
- **跨件契约**：SKILL.md 中描述的工具调用顺序和阈值（30 秒分析门槛，2 分钟短/长视频分界）与 `watch-video.md` command 完全一致，无矛盾

### 1.3 设计思路 / 方法论
- **核心设计哲学**："Claude 是分析者，MCP 是工具"——插件不让 MCP 服务器做任何内容理解，只做机械的帧提取和音频转写，Claude 拿到数据后自己决定怎么理解
- **解决什么问题**：Claude Code 原生没有视频处理能力；本插件打通了多媒体文件到 Claude 的"最后一公里"
- **做了什么 trade-off**：选择"多后端 + 灵活配置"而不是"单一依赖"，牺牲了配置简单性，换来了用户可选择本地或云端处理
- **反映什么认知模型**：作者清楚地把"数据采集"和"数据理解"分层，这是良好的系统设计直觉；但 NL 文档层（command 文件）的质量管理意识较弱（缺 `name` 和 `allowed-tools` 这种基础问题未发现）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：MCP 感知层 + NL 命令协同**

MCP 服务器承担所有 IO 密集操作（ffmpeg 帧提取、whisper 转写、yt-dlp 下载），NL command 文件提供人性化的交互脚本（向用户询问配置、解释分析步骤、格式化输出），两层职责分离。

模式特征清单：
- 特征 1：MCP 工具名语义清晰（`video_info`, `video_analyze`, `video_watch`, `video_detail`），自描述性强
- 特征 2：NL command 里有明确的"必须按顺序执行"注释（"do NOT skip step 2"）
- 特征 3：SKILL.md 负责携带跨 session 的"策略记忆"（哪个后端，阈值是多少）
- 特征 4：配置向导（`setup-video-vision.md`）用单选交互引导用户逐步完成复杂配置

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多媒体处理（视频/音频） | ✅ 高度适用 | MCP 服务器天然适合处理 IO 密集型任务 |
| 纯文本分析 | ❌ 过度设计 | 没必要引入 MCP + TypeScript 构建链 |
| 需要多步配置的功能 | ✅ 适用 | 配置向导 command 模式很好地处理了多选项的引导 |
| 对延迟敏感的实时场景 | ⚠️ 需谨慎 | `@latest` 安装 + 自动下载 Whisper 模型在首次运行时很慢 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| MCP 感知层 + NL 命令协同（本仓库） | claude-video-vision | 职责分离清晰，可扩展多后端 | command frontmatter 质量风险，@latest 供应链 |
| 纯脚本调用（无 MCP） | 直接在 command 里调 Bash ffmpeg | 无额外依赖 | Claude 无法"交互式"调整提取策略 |
| 全功能 MCP（解释也在服务端） | 假设的 video-analyzer | 一致性强 | 失去 Claude 的理解能力，输出死板 |

### 2.4 改进空间
1. **当前问题**：两个 command 文件都缺 `name` frontmatter，导致命令无法通过 slash-command 面板发现。**改进做法**：在 `watch-video.md` 添加 `name: watch-video`，在 `setup-video-vision.md` 添加 `name: setup-video-vision`。**预期收益**：用户可以直接在命令面板输入 `/watch-video` 调用，无需手动发现。
2. **当前问题**：`.mcp.json` 使用 `claude-video-vision@latest`，每次 MCP 服务器启动时拉取最新版。**改进做法**：固定到具体版本 `claude-video-vision@1.3.1`，并在每次发版时手动更新。**预期收益**：消除供应链风险，确保运行时版本可预期。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **79/100**，安全扫描结论 **CLEAR**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/watch-video.md` | 56 | 缺 `name` frontmatter（-25），无空输入处理（-10），缺 allowed-tools（-5） |
| `commands/setup-video-vision.md` | 66 | 缺 `name` frontmatter（-25），缺 allowed-tools（-5），模糊量词（-4） |
| `agents/frame-describer.md` | 81 | 零示例块（-15），模糊量词（-4） |
| `skills/video-perception/SKILL.md` | 92 | 四个模糊量词（"smart", "relevant", "insufficient", "complete"）各 -2 |
| `.claude-plugin/plugin.json` | 100 | 完美 |

### 3.2 当时值得借鉴的模式
1. **感知/解释分层**：`watch-video.md` 明确注释"Step 2 是必须执行的，不能跳过"——把工作流约束直接写进 NL 文档，而不是依赖用户理解。借鉴：我的 `echo-sleuth/commands/timeline.md` 是否也有"必须按序"的步骤？如果有，需要显式标注。
2. **配置向导 command 模式**：`setup-video-vision.md` 用"每次只问一个问题，选项用字母标记"的方式降低配置复杂度。这是复杂初始化场景的最佳实践。借鉴：我的 `cold-start-interview` 类技能已经在用类似模式，验证了这个方向。
3. **`plugin.json` 100 分**：manifest 完整，字段不缺。这说明作者对 plugin.json 的规范很清楚，但对 command frontmatter 规范不熟悉——两个知识盲区共存。

### 3.3 当时的缺陷
1. **命令文件缺 `name` frontmatter**：根本原因是作者不知道 Claude Code 的 command discovery 依赖 `name` 字段，`plugin.json` 里有 `name` 不代表子文件里可以省略。这个错误导致 `/watch-video` 和 `/setup-video-vision` 在命令面板里根本不可见。自查：我的所有 command 文件是否有 `name:` 字段？
2. **`watch-video.md` 无空输入处理**：命令需要视频路径，但如果用户不带参数调用，没有任何错误提示或引导。根本原因：作者假设用户"肯定知道要传参数"，没有从失败路径考虑。
3. **`frame-describer.md` 零示例**：Agent 定义里没有任何输入/输出示例，Claude 在使用时缺少对齐点。根本原因：作者把示例视为"可选装饰"而不是"约束对齐工具"。

### 3.4 当时的优化机会
1. 给两个 command 文件各加 `name:` 字段（5 分钟，最高 ROI）
2. 给 `watch-video.md` 加空输入处理："如果用户未提供视频路径，询问：Which video file or YouTube URL would you like to analyze?"（2 分钟）
3. 给 `frame-describer.md` 加一个完整示例（输入：30 秒 GoPro 视频，输出：场景描述）（15 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `watch-video.md` 缺 `name:` | `grep "^name:" commands/watch-video.md` | **仍然缺失**（frontmatter 只有 `description` 和 `argument-hint`）| Bug 2 个月未修 |
| `setup-video-vision.md` 缺 `name:` | `grep "^name:" commands/setup-video-vision.md` | **仍然缺失**（frontmatter 只有 `description`）| 同上 |
| `.mcp.json` 用 `@latest` | `grep "latest" .mcp.json` | **仍然存在**（`"args": ["-y", "claude-video-vision@latest"]`）| 供应链风险持续 |

### 4.2 架构演进
仓库自 audit 以来结构无变化（相同的 5 个 NL 工件：2 commands、1 agent、1 skill、1 manifest）。CHANGELOG.md 的存在说明 MCP 服务器端（`mcp-server/`）有更新（1.2.0 → 1.3.1），但 NL 层未同步维护。这印证了"NL 层和代码层的维护频率不对称"的普遍现象。

### 4.3 新增的可学习模式
暂无——NL 层无变化。MCP 服务器的 1.3.1 更新可能增加了新功能，但未被 SKILL.md 或 command 文件反映。

---

## 五、校准

### 5.1 我已经在做对的
1. 配置向导模式（`cold-start-interview` 类技能）——本案例验证这是对的
2. 感知/解释分层的认知——`echo-sleuth` 的 MCP 工具只做文件读写，理解由技能文档和 Claude 协作完成
3. `plugin.json` 字段完整性

### 5.2 挑战 / 验证
本案例**验证**了一个教训：**`plugin.json` 知道规范不等于 command 文件也知道规范**——这两种文件的格式要求来自不同文档，作者有认知盲区。我需要确认自己的 command 文件确实全部有 `name:` 字段。（实际上 echo-sleuth 有 4 个 commands 缺 allowed-tools，说明我也有类似的盲区。）

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 command 文件是否都有 name: 字段
find . -path "*/commands/*.md" | while read f; do
  name=$(grep -c "^name:" "$f" 2>/dev/null)
  [ "$name" -eq 0 ] && echo "缺 name: $f"
done
# 命中后：添加 name: 字段，用文件名（去掉 .md 后缀）作为值

# 检查我的 command 文件是否都有空输入处理
find . -path "*/commands/*.md" | while read f; do
  if ! grep -qi "no argument\|without argument\|未提供\|如果用户没有\|如果没有参数\|empty" "$f" 2>/dev/null; then
    echo "可能缺空输入处理: $f"
  fi
done
# 命中后：在 command 正文开头加"若用户未提供 $ARGUMENTS，提示用户..."

# 检查我的 agent 文件是否有示例
find . -path "*/agents/*.md" | while read f; do
  if ! grep -qi "## example\|## 示例\|input.*output\|### sample" "$f" 2>/dev/null; then
    echo "无示例: $f"
  fi
done
# 命中后：至少加一个完整的 input→output 示例
```

### 6.2 灵感 → 实施路径

1. **想法**：给 `echo-sleuth` 的 4 个缺 `allowed-tools` 的 commands 批量补齐
   - **为何可行**：这是今天发现的确定性问题，修复成本极低
   - **第一步**：打开 `recall.md`，查看正文里调用了哪些 Claude Code 工具，加 `allowed-tools:` frontmatter，5 分钟/文件

2. **想法**：参考 `watch-video.md` 的"明确工作流步骤顺序"做法，给 `echo-sleuth/commands/audit.md` 加步骤顺序约束
   - **为何可行**：`audit` 命令涉及多个 agents 的顺序调用，存在步骤依赖，但文档没有强调顺序
   - **第一步**：在 `audit.md` 正文里，对每个步骤添加"Step N:"前缀，并对必须顺序执行的步骤标注"必须在 Step N-1 完成后执行"，20 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **jordanrendric/claude-video-vision 的核心目的**：给 Claude Code 添加视频理解能力（多媒体感知层）
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 同样是"给 Claude 添加新感知能力"的插件，同样有 commands + agent + skill + MCP 层 | echo-sleuth 挖历史对话，video-vision 处理视频文件 | 高 |
| MarkQWu/claude-for-legal | 低 | 都有多个 skill 文件 | 领域和技术栈完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 `name:` frontmatter | `find commands/ -name "*.md" \| xargs grep -L "^name:"` | 未命中（echo-sleuth 的 commands 有 name 字段） | N/A |
| Command 缺 `allowed-tools` | `for f in commands/*.md; do grep -c "allowed-tools" "$f" \|\| echo "MISSING: $f"; done` | **命中 4/8**：recall.md, recap.md, timeline.md, lessons.md | 中 |
| Agent 无示例块 | `find agents/ -name "*.md" \| xargs grep -L "example\|示例"` | **命中全部 5 个** echo-sleuth agents（memory-auditor, file-historian, schema-scout, recall, analyze）| 高 |

**命中后的具体行动建议**：
- `echo-sleuth/agents/memory-auditor.md` → 加一个完整示例：输入"分析会话 2026-05-15"，输出"发现 3 个决策点 + 2 个错误模式"，20 分钟
- `echo-sleuth/agents/recall.md` → 加示例：输入"查找我上周提到 GraphQL 的对话"，输出"找到 2 个会话，关键引用..."，15 分钟
- `echo-sleuth/commands/recall.md` 等 → 补 `allowed-tools`，5 分钟/文件

### 8.3 别人的更优方案

1. **领域**：工作流步骤顺序的显式约束
   - **本案例做法**：`watch-video.md` 在 Step 2 旁边标注 "**REQUIRED for videos > 30s:** ... This is NOT optional"，用大写 + 粗体强调顺序约束
   - **我的项目现状**：`echo-sleuth/commands/audit.md` 有多步骤工作流，但步骤没有顺序约束的显式标注，用户可能乱序执行
   - **如何借鉴**：在 `audit.md` 里对"必须先跑 extract，才能跑 analyze"这类依赖关系加粗体提示

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Command `name:` frontmatter 完整性
- **我的做法**：`echo-sleuth` 所有 command 文件均有 `name:` 字段
- **本案例做法（弱在哪）**：`claude-video-vision` 两个 command 文件都缺 `name:` frontmatter，导致 slash-command 面板里根本找不到
- **意义**：这是我做对的基础点，说明对 Claude Code command 规范的理解比本案例作者更扎实

---

## 八、术语表

### <a name="感知层"></a>感知层（Perception Layer）
> 在 AI 系统架构中，专门负责把原始数据（视频、音频、图像）转换为 AI 可以处理的格式（文本描述、时间戳序列）的一层。它不做理解，只做转换——就像眼睛只负责把光转换为神经信号，大脑才负责理解。本插件的 MCP 服务器就是视频的感知层：ffmpeg 提取帧，whisper 转写音频，但不判断"视频里发生了什么"。

### <a name="slash-command"></a>slash-command（斜杠命令）
> 在 Claude Code 里，以 `/` 开头的快捷命令，如 `/watch-video`。Claude Code 通过扫描 `.claude/commands/` 目录下的 `.md` 文件来注册这些命令。如果文件里没有 `name:` frontmatter，Claude Code 不知道命令叫什么，用户就无法通过命令面板发现和调用它。

### <a name="allowed-tools"></a>allowed-tools
> SKILL.md 或 command 文件 frontmatter 里声明当前文件允许 Claude 使用的工具列表（如 `Bash`, `Read`, `Write`）。如果 command 文件调用了某工具但 `allowed-tools` 里没声明，在严格权限模式下该工具调用会被阻止。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据。Claude Code 读 command 文件时先解析 frontmatter 获取命令名称（`name`）、描述（`description`）、权限（`allowed-tools`）等。如果 `name:` 字段缺失，该 command 不会出现在斜杠命令面板中。
