# 777genius/claude-notifications-go — 学习案例

**仓库**：https://github.com/777genius/claude-notifications-go
**Stars**：528 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-17（基于审计数据）
**主题标签**：security-gate, manifest-discipline, template-design, model-pinning

---

## 一、理解（基于审计报告与注册表数据）

### 1.1 仓库概览
这是一个为 Claude Code 提供桌面推送通知能力的 Go 语言插件，通过 macOS 原生 API（`terminal-notifier`）把 Claude 的各类生命周期事件（Stop、SubagentStop、TeammateIdle 等）转换成系统级提示音和通知。528 颗星表明有相当数量的开发者认为"Claude 做完任务主动通知我"是刚需功能。插件依赖自动构建的 Go 二进制文件，通过 hooks.json 挂载到 5 种钩子事件上。安装方式是执行 curl 一键脚本。

关键事实：
- 纯系统集成类插件，NL 层（markdown 指令文件）数量少（6 个 command）但执行层（shell 脚本、Go 程序）规模大
- 从 plugin.json 维护状态看有若干遗漏注册（两个 command 文件不在 commands 数组中）
- hooks.json 覆盖 5 种钩子，是少数同时钩挂 PreToolUse + 通知类事件的插件
- 安全状态：BLOCKED（两个 CRITICAL 级别安全问题）

### 1.2 架构剖析
```
claude-notifications-go/
├── .claude-plugin/
│   └── plugin.json          # 插件清单（有注册遗漏）
├── commands/
│   ├── init.md              # 初始化命令
│   ├── settings.md          # 设置命令
│   ├── sounds.md            # 声音命令（★未注册）
│   ├── notifications-init.md     # 别名
│   ├── notifications-settings.md # 别名
│   └── notifications-sounds.md   # 别名（★未注册）
├── hooks/
│   └── hooks.json           # 5 种钩子，均调用 hook-wrapper.sh
└── bin/
    ├── hook-wrapper.sh      # 钩子包装器，有自动更新逻辑
    ├── install.sh           # 安装脚本（CRITICAL 安全问题）
    └── bootstrap.sh         # 引导脚本（CRITICAL 安全问题）
```

- **文件类型分布**：6 个 command / 1 个 hooks.json / 1 个 plugin.json / 多个 shell 脚本与 Go 源码
- **编排关系**：所有 5 个钩子 → hook-wrapper.sh → Go 二进制（管理通知逻辑）；command 文件调用 Bash 工具执行设置操作
- **跨件契约**：plugin.json 应列出所有 command 文件路径，但漏掉了 sounds.md 和 notifications-sounds.md；alias 系列命令应透传到主命令，但 allowed-tools 声明与实际行为不符

### 1.3 设计思路 / 方法论
- **核心设计哲学**："最小 NL 层 + 最大原生层"——尽可能把复杂逻辑下推到 Go 程序，NL 文件只做薄薄的引导层
- **解决什么问题**：开发者跑长任务时需要盯着终端，这个插件把"Claude 完成"变成一个不需要窗口焦点的感知信号
- **Trade-off**：Go 二进制提供了跨版本稳定性和 macOS 系统 API 访问能力，但引入了自动更新逻辑；自动更新让 hook-wrapper.sh 在每次触发时发起网络请求（silently），这在 hook 调用频繁的场景下会显著拖慢响应
- **认知模型**：把 Claude Code 看成一个异步执行引擎，通知系统是其"完成信号"抽象层

---

## 二、过去审查发现（2026-04-17 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **63/100**，安全状态 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/init.md | 53 | 缺 `name` 字段（-25）、无输出格式（-10）、无空输入处理（-10） |
| commands/notifications-settings.md | 53 | 缺 `name`（-25）、4 个工具声明但均未使用（-12） |
| commands/sounds.md | 55 | 缺 `name`（-25）、未注册到 plugin.json、无输出格式（-10） |
| commands/settings.md | 60 | 缺 `name`（-25）、无空输入处理（-10） |
| commands/notifications-init.md | 62 | 缺 `name`（-25）、Bash 声明但未使用（-3） |
| commands/notifications-sounds.md | 62 | 缺 `name`（-25）、未注册到 plugin.json |
| .claude-plugin/plugin.json | 70 | sounds.md 和 notifications-sounds.md 缺失于 commands 数组 |
| hooks/hooks.json | 90 | 结构完整，无结构性问题 |

**加权平均**：63/100

### 2.2 当时值得借鉴的模式

1. **Hooks 覆盖完整性**：hooks.json 钩挂了 5 种不同类型的钩子（PreToolUse、Notification、Stop、SubagentStop、TeammateIdle），是实战中事件驱动设计较完整的案例。每种事件都有独立的处理逻辑。  
   → 原文路径：`hooks/hooks.json`  
   → 借鉴：设计通知类插件时，主动列出所有相关钩子事件，而不只钩挂 Stop

2. **钩子包装器模式**：hook-wrapper.sh 统一处理所有钩子调用，主二进制只做业务逻辑，分离关注点。  
   → 原文路径：`bin/hook-wrapper.sh`  
   → 借鉴：钩子逻辑复杂时，抽出一个 wrapper 做鉴权、版本检测、日志的交叉切面

3. **Alias 命令组织**：用 `notifications-*` 前缀给命令做"旧名字兼容"，思路有价值；但执行不到位（tools 声明错误）。这提示我们 alias 命令需要单独审计，不能直接继承主命令的 frontmatter。  
   → 借鉴：alias 命令如果只是 redirect stub，应声明 `allowed-tools: []` 或直接省略

### 2.3 当时的缺陷

1. **全量 `name` 字段缺失**（6 个 command 均无 `name`）  
   → 根本原因：作者可能误以为 plugin.json 里的 command 路径已足够作为命令标识，没有意识到 markdown frontmatter 里的 `name` 是 Claude Code 注册命令时读取的元数据  
   → 自查：我的每个 command/agent/skill 文件的 YAML frontmatter 是否都有 `name` 字段？

2. **plugin.json 注册遗漏**（sounds.md 和 notifications-sounds.md 不在 commands 数组）  
   → 根本原因：manifest 文件和实际文件目录之间没有任何自动同步机制，手动维护时出现遗漏。这是"manifest discipline"问题的典型表现——新增文件后忘记更新清单  
   → 自查：我的 plugin.json 是否与 commands/ 目录保持完全对应？

3. **允许工具声明与实际使用严重不符**（notifications-settings.md 声明了 Bash、AskUserQuestion、Write、Read 四个工具但一个都没用）  
   → 根本原因：alias 命令是从主命令复制粘贴 frontmatter 而来，没有根据 stub 的实际功能裁剪；工具过声明扩大攻击面  
   → 自查：我的每个 NL artifact 的 allowed-tools 是否与正文实际调用严格对应？

4. **钩子自动更新引发安全盲点**：hook-wrapper.sh 在版本不匹配时静默运行 `run_install --force`，所有输出重定向到 `/dev/null 2>&1 || true`  
   → 根本原因：作者把"用户无感升级"当成好的用户体验，但在安全上这是一个无法被用户察觉的后门入口  
   → 自查：我的 hooks 是否有任何静默网络访问或静默安装行为？

### 2.4 当时的优化机会

1. **统一 alias 命令的 allowed-tools**：所有 `notifications-*` 的 redirect stub 应该声明空工具列表，或完全移除工具声明，减少攻击面

2. **manifest 同步脚本**：在 CI 或安装脚本中加一步检查：`commands/` 目录下所有 `.md` 文件是否都出现在 `plugin.json` 的 `commands` 数组中——一行 bash 就能做到

3. **输出格式标准化**：5 个 command 缺少 `## Output Format` 节，用统一的模板（成功状态 / 错误状态 / 无操作状态三态）补全

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 6 个 command 全部缺 `name` 字段 | GitHub API 直读 frontmatter | **无法验证**（API 限速，无认证 token） | 暂缺 |
| sounds.md 未注册到 plugin.json | 读 plugin.json commands 数组 | **无法验证** | 暂缺 |
| notifications-settings.md 工具声明与实际不符 | 比对 allowed-tools vs 正文 | **无法验证** | 暂缺 |

> 注：本案例生成时 GitHub REST API 处于未认证限速状态，无法实时拉取目标仓库文件。上述状态基于 2026-04-17 审计快照，未来可通过配置 `GITHUB_TOKEN` 重新核查。

### 3.2 架构演进

审计报告确认了 `.claude-plugin/plugin.json` 存在，6 个 command 文件存在，hooks.json 存在，bin/ 下有多个 shell 脚本。从 registry 看该仓库状态为 `audited`（非 `contributed`，因安全门被 BLOCK）。说明自 2026-04-17 以来 NLPM 未向该仓库提交 PR，架构演进无外部干预，完全取决于作者自身迭代。

### 3.3 新增的可学习模式

暂无（live 检查不可用，无法识别新增设计）。

---

## 四、校准

### 4.1 我已经在做对的

1. **Hooks 事件覆盖**：如果我的插件也需要感知多种 Claude Code 生命周期事件，我已养成习惯逐一列举钩子事件类型，而不是只钩 Stop
2. **Wrapper 分层**：在钩子逻辑复杂的情况下，我已用 wrapper 脚本分离业务逻辑和系统逻辑
3. **Manifest 同步意识**：我已经知道 plugin.json 和文件系统需要手动保持一致，并在写新 command 时同步更新清单
4. **空工具声明**：alias/stub 类命令我会显式声明 `allowed-tools: []` 或完全省略，不从主命令复制粘贴

### 4.2 挑战 / 验证

**挑战**：这次案例让我重新审视"静默自动更新"这个设计决策。hook-wrapper.sh 的自动更新看似便利，但实际上是把一个完整的软件更新周期压缩进了每次钩子触发。这在频繁触发的场景下（PreToolUse 每次工具调用都触发）意味着每次操作都可能触发网络下载，完全不透明。

**验证**：`name` 字段缺失在 6 个 command 中全部复现，说明这确实是一个最容易被忽视的"一次遗漏、全量失效"型错误。我之前在自己的 command 文件里也曾经犯过类似的从模板创建新文件后忘记改 `name` 的错误。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查本项目所有 command/agent/skill markdown 文件是否都有 name 字段
find ~/.claude -name "*.md" -path "*/commands/*" -o -name "*.md" -path "*/agents/*" | \
  xargs grep -L "^name:" 2>/dev/null
# 命中后：为缺失 name 字段的文件添加 `name: <命令名称>` 到 YAML frontmatter
```

```bash
# 检查 plugin.json 中注册的 commands 与实际 commands/ 目录是否同步
PLUGIN_JSON=".claude-plugin/plugin.json"
if [ -f "$PLUGIN_JSON" ]; then
  REGISTERED=$(jq -r '.commands[]' "$PLUGIN_JSON" 2>/dev/null | sort)
  ACTUAL=$(find commands/ -name "*.md" | sort)
  diff <(echo "$REGISTERED") <(echo "$ACTUAL")
fi
# 命中差异后：在 plugin.json 的 commands 数组中补全遗漏的文件路径
```

```bash
# 检查 hooks 中是否有静默网络访问或静默安装行为
grep -rn ">/dev/null 2>&1" hooks/ bin/ scripts/ 2>/dev/null | grep -E "curl|wget|pip|npm|install"
# 命中后：评估该静默操作是否需要暴露给用户（加日志或改成交互式确认）
```

### 5.2 灵感 → 实施路径

1. **想法**：在 CI 中加 manifest 一致性检查，防止新增 command 后忘记注册  
   **为何可行**：一行 bash diff 就能对比目录和 JSON 数组，零依赖  
   **第一步**：在 `.github/workflows/` 中新建一个步骤，执行 `diff <(jq -r '.commands[]' .claude-plugin/plugin.json | sort) <(find commands/ -name '*.md' | sort)`；命中差异时 exit 1

2. **想法**：为所有 command 文件添加输出格式模板（三态：成功 / 错误 / 无操作）  
   **为何可行**：三态输出是最小可行的结构化输出约定，复用成本低  
   **第一步**：在 `commands/` 下的每个 `.md` 中加 `## Output Format` 节，用三个 code block 分别描述成功、错误、无操作场景的输出样例；30 分钟内可完成全量补全
