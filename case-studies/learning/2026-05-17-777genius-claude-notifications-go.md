# 777genius/claude-notifications-go — 学习案例

**仓库**：https://github.com/777genius/claude-notifications-go
**Stars**：528 | **来源**：xiaolai 上游审查
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD a73421739cad）
**主题标签**：manifest-discipline、security-gate、single-purpose、fallback-chain

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`claude-notifications-go` 是一个为 Claude Code 提供桌面通知的插件，核心用 Go 实现，支持 macOS / Linux / Windows 三平台。功能描述相当完整：6 种通知类型（任务完成、代码审查、问题询问、计划生成、会话限制、API 错误）、点击通知聚焦终端、Webhook 集成（ntfy、Slack、Telegram）、可定制音效与音量控制、Git 分支显示。版本号已到 1.39.1，说明项目曾经历相当长的迭代周期。

然而仓库根目录存在一个 `.orphaned_at` 文件，内容是时间戳 `1766603417062`（对应约 2025 年 12 月下旬），意味着作者在 2026 年 4 月的审查发生之前就已将该项目标记为废弃。528 颗星是用户规模的见证，也是遗留问题最值得研究的原因——一个被广泛安装的插件，其安全漏洞会在用户机器上长期存在。

### 1.2 架构剖析

该仓库是一个典型的"NL 表皮 + 原生二进制核心"架构：

```
NL 表面层（Claude Code 可见）
  commands/init.md              ← 下载并安装二进制
  commands/settings.md          ← 交互式配置向导
  commands/sounds.md            ← 列出并预览音效（未注册到 plugin.json）
  commands/notifications-init.md     ← init 的别名
  commands/notifications-settings.md ← settings 的别名
  commands/notifications-sounds.md   ← sounds 的别名（未注册）
  hooks/hooks.json              ← 5 个生命周期钩子

执行层（Claude Code 不可见）
  bin/hook-wrapper.sh           ← 钩子触发点，调用 Go 二进制
  bin/install.sh                ← 安装脚本（从 main 分支动态拉取）
  bin/bootstrap.sh              ← 引导脚本（同样从 main 分支动态拉取）
  cmd/claude-notifications/     ← Go 主程序，macOS 侧使用 swift-notifier
  sounds/                       ← 5 个 MP3 文件（question / error /
                                   task-complete / review-complete / plan-ready）
```

`hooks.json` 配置了 5 种钩子事件（`PreToolUse`、`Notification`、`Stop`、`SubagentStop`、`TeammateIdle`），统一通过 `${CLAUDE_PLUGIN_ROOT}/bin/hook-wrapper.sh` 转发给 Go 二进制，超时均设为 30 秒。这个钩子设计是整个项目中最规范的部分。

### 1.3 设计思路 / 方法论

该项目遵循"命令是薄包装"原则——真正的逻辑全部下沉到 shell 脚本和 Go 二进制，Claude Code 命令只负责触发安装或配置流程。这在理论上是合理的分层，但实践中暴露了两个系统性问题：

**别名泛滥**：`notifications-init`、`notifications-settings`、`notifications-sounds` 完全复制了 `init`、`settings`、`sounds` 的功能，没有增量价值。这使命令数量从 3 膨胀到 6，plugin.json 的维护负担加倍，且二者同步时极易漂移。

**安全边界模糊**：命令文件本身看似无害（`disable-model-invocation: true`），但它们调用的 shell 脚本在运行时从 `main` 分支拉取另一个脚本再立即执行，把"静态已知代码"变成了"动态未知代码"。这是一条从 NL 层到执行层的安全漏洞传导链。

---

## 二、过去审查发现（2026-04-17 历史快照）

### 2.1 当时质量评分（NLPM）

**综合得分：63 / 100**（安全状态：BLOCKED）

| 文件 | 得分 | 主要扣分项 |
|---|---|---|
| commands/init.md | 53 | 缺少 `name` 字段（-25）、无输出格式规范（-10）、无空输入处理（-10）|
| commands/notifications-settings.md | 53 | 缺少 `name` 字段（-25）、4 个声明但未使用的工具（-12）、无输出格式（-10）|
| commands/sounds.md | 55 | 缺少 `name` 字段（-25）、未注册到 plugin.json（-10）、无输出格式（-10）、无空输入处理（-10）|
| commands/settings.md | 60 | 缺少 `name` 字段（-25）、无空输入处理（-10）、输出格式不规范（-5）|
| commands/notifications-init.md | 62 | 缺少 `name` 字段（-25）、Bash 工具声明但未使用（-3）、无输出格式（-10）|
| commands/notifications-sounds.md | 62 | 缺少 `name` 字段（-25）、未注册到 plugin.json（-10）、Bash 未使用（-3）|
| plugin.json | 70 | `sounds.md` 和 `notifications-sounds.md` 均不在 commands 数组中 |
| hooks/hooks.json | 90 | 结构规范，扣分仅为细节 |

安全发现：

- **CRITICAL（2个）**：`commands/init.md` 第 38、45 行从 `raw.githubusercontent.com/main` 下载 `install.sh` 后直接用 `bash` 执行，无任何完整性校验；`bin/bootstrap.sh` 第 756–769 行同样默认从 `main` 分支拉取并执行，无校验和验证。
- **HIGH（1个）**：`commands/settings.md` 第 250–265 行将用户输入的 `sound_name` 直接拼接为文件路径 `${PLUGIN_ROOT}/sounds/${sound_name}.mp3`，存在路径穿越风险。
- **MEDIUM（3个）/ LOW（2个）**：未详细列出。

### 2.2 当时值得借鉴的模式

**hooks.json 设计（得分 90）**：5 个钩子事件的配置结构清晰、分工明确，统一走 `hook-wrapper.sh` 中转，每个事件对应一个语义明确的处理函数名（`handle-hook PreToolUse`、`Stop`、`SubagentStop`、`TeammateIdle`）。30 秒超时设置合理，避免了无限挂起。

**钩子与二进制解耦**：`hooks.json` 不直接调用 Go 二进制，而是通过 `hook-wrapper.sh` 这一中间层。这使得二进制路径、参数格式的变化不会直接影响 hooks.json 的结构，降低了 NL 层的脆弱性。

**多平台 fallback 链**：`sounds.md` 在找不到插件内置 `list-sounds` 二进制时，会回退到手动列出系统音效目录（macOS：`/System/Library/Sounds/`，Linux：`/usr/share/sounds/`）。这种"主路径 + 降级路径"的设计使命令在部分安装失败的情况下仍有基本可用性。

**功能描述精准**：README 清楚列出 6 种通知类型和触发时机，用户安装前即可预判插件行为，减少了"装了不知道干什么"的问题。

### 2.3 当时的缺陷

**全部 6 个命令文件缺少 `name` 字段**：这是单个影响最大的机械错误，每个文件扣 25 分。`name` 字段缺失后，Claude Code 无法在 UI 中正确注册命令，用户以 `/claude-notifications-go:xxx` 调用时行为未定义。这 6 个错误性质完全相同，显然是批量复制 frontmatter 模板时遗漏了这个字段。

**`sounds.md` 和 `notifications-sounds.md` 未注册到 plugin.json**：两个命令文件存在于磁盘上，但 plugin.json 的 `commands` 数组中只有 `init.md`、`settings.md`、`notifications-init.md`、`notifications-settings.md`。结果是这两个音效命令对用户完全不可达，相当于死代码。

**别名命令无增量价值**：`notifications-*` 三个命令与原始三个命令的区别仅在于名称前缀，内容上看不出任何差异化逻辑。别名使总命令数翻倍，但同时只有 4 个注册到 plugin.json，制造了一个令人困惑的不对称结构。

**`notifications-settings.md` 声明了 4 个未使用的工具**：frontmatter 中列出了 `Bash`、`AskUserQuestion`、`Write`、`Read`，但实际命令体中只用到了部分工具。未使用的工具声明会让 Claude Code 请求不必要的权限，增加用户的安全顾虑。

### 2.4 当时的优化机会

1. 为所有 6 个命令文件的 frontmatter 补充 `name` 字段，格式参考 `name: claude-notifications-go:init`——这是可在几分钟内完成的机械修复，却能消除 25 分/文件的最大单项扣分。
2. 将 `sounds.md` 和 `notifications-sounds.md` 加入 `plugin.json` 的 `commands` 数组，或明确删除这两个文件（如果别名策略已废弃）。
3. 合并或删除 `notifications-*` 别名系列，保持命令集合的最小化。
4. 为下载-执行模式引入 SHA-256 完整性校验，将校验和固定在版本发布中，而非每次运行时从 `main` 分支拉取。
5. 对 `sound_name` 做白名单过滤（仅允许 `[a-z0-9\-]+`），消除路径穿越风险。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

基于当前 HEAD（`a73421739cad6fd9fe45153d173d5be9096c6eb8`）的逐项核查：

| 缺陷 | 2026-04-17 状态 | 2026-05-17 状态 | 变化 |
|---|---|---|---|
| 6 个命令文件缺少 `name` 字段 | 未修复 | **仍未修复** | 无变化 |
| `sounds.md` 未注册到 plugin.json | 未修复 | **仍未修复** | 无变化 |
| `init.md` 从 `main` 下载并执行（无校验）| 存在 | **仍存在**（第 33 行 URL 仍指向 main）| 无变化 |
| `bootstrap.sh` 从 `main` 下载并执行（无校验）| 存在 | **仍存在**（第 21 行默认值仍为 main）| 无变化 |
| `sound_name` 路径穿越风险 | 存在 | 未确认修复 | 无明确变化 |

结论：`.orphaned_at` 文件时间戳（2025-12-29）早于审查日期（2026-04-17），作者在审查发现之前就已停止维护。因此所有缺陷均处于冻结状态，审查结论对当前用户完全适用。

### 3.2 架构演进

无。仓库在被标记废弃后未发生任何可见的架构变化。plugin.json 版本仍停留在 1.39.1，commands 数组结构与审查时一致。

这本身就是一个值得记录的状态：**一个被 528 人加星的安全漏洞，在被正式标记为废弃后，以固化的形态继续存在于所有已安装用户的机器上**。与一个从未流行的项目不同，高星仓库的遗留漏洞具有长尾效应——`main` 分支 URL 在 hooks 触发时仍然可用，意味着理论上攻击者在 `main` 分支被篡改或通配域名劫持时可以推送恶意载荷。

### 3.3 新增的可学习模式

无新增模式。但废弃状态本身提供了一个反面教材：**项目生命周期结束时，应当采取安全降级措施**，例如：
- 将 `main` 分支 URL 替换为指向最后一个稳定 release tag 的 URL，防止 main 分支未来被意外写入；
- 在 README 顶部显著标注废弃状态和推荐替代项；
- 归档仓库（GitHub 的 Archive 功能会阻止新的 push，从根本上固定代码内容）。

---

## 四、校准

### 4.1 我已经在做对的

**NL 命令只做一件事**：该项目中 settings.md 的配置向导混合了二进制验证、音效选择、音量配置、设备选择、Webhook 配置共 8 个步骤，本质上是一个过于宽泛的瑞士军刀。相比之下，限定每个命令的职责范围（如分别提供 `configure-sound`、`configure-webhook`、`configure-volume`）会让每个命令更易于测试和独立维护。

**hooks.json 的高分（90）证明：NL 钩子层应该保持轻薄**。钩子文件不含任何业务逻辑，只做"转发给二进制"这一件事。这个分层是该项目最值得保留的设计决策。我在自己的钩子设计中也应坚持这一原则。

**manifest 与文件系统的一致性是基础**：plugin.json 中 commands 数组与磁盘上实际存在的 `.md` 文件不一致，是一个低成本、高危害的问题——用户无法调用未注册的命令，但他们不会看到任何错误提示。这种静默失效比显式报错更难排查。我应在每次新增或删除命令文件时同步检查 manifest。

### 4.2 挑战 / 验证

**挑战一：下载-执行模式的安全边界在哪里？**

该项目使用 `curl | bash` 模式安装二进制。这在命令行工具生态中极为普遍（Homebrew、rustup 都用类似机制），但风险点在于"从哪个 ref 下载"和"是否校验完整性"。

验证方案：检查自己的任何安装脚本或命令文件——是否有从 `main`/`master` 分支动态拉取并执行的操作？如果有，将下载 URL 改为指向固定 tag 或 commit SHA，并在执行前校验 `sha256sum`。

**挑战二：声明的工具与实际使用的工具是否一致？**

`notifications-settings.md` 的 frontmatter 声明了 `Bash`、`AskUserQuestion`、`Write`、`Read`，但并非所有工具都实际被用到。Claude Code 会根据 `allowed-tools` 字段向用户请求权限，多余的声明相当于申请了不需要的权限——这违反最小权限原则，也会降低用户对插件的信任。

验证方案：逐行审查自己命令文件的 `allowed-tools` 字段，对每一个工具名，在命令体中找到至少一处对应的使用场景，否则从列表中删除。

**挑战三：别名命令是否真的必要？**

`notifications-init` 等别名的存在暗示作者曾考虑过命名空间问题——短命令名可能与其他插件冲突，带前缀的长命令名更安全。但实现方式是复制全部内容而非使用重定向机制，导致维护成本加倍。

验证方案：如果我有类似的别名命令，考虑改用单一 `name` 字段配置多个触发词（若 Claude Code 支持），或明确选择一个规范名称并通过文档引导用户，而不是维护两份内容相同的文件。

---

## 五、行动

### 5.1 自查动作

1. **Manifest 一致性检查**：打开自己项目的 `plugin.json`，逐行核对 `commands` 数组中的每个路径是否对应磁盘上真实存在的文件；反向检查 `commands/` 目录下的每个 `.md` 文件是否都已注册。这个检查可以用一行 shell 完成：`diff <(jq -r '.commands[]' plugin.json | sort) <(ls commands/*.md | sort)`。

2. **Frontmatter 完整性检查**：检查所有命令文件的 frontmatter 是否包含 `name`、`description`、`allowed-tools` 三个必填字段。`name` 字段的格式应为 `<plugin-name>:<command-name>`，与 plugin.json 中的插件名一致。

3. **最小权限审查**：对每个 `allowed-tools` 列表中的工具，确认命令体中有对应的使用场景。去掉所有未使用的工具声明。

4. **下载-执行扫描**：在整个仓库中搜索 `curl.*\|.*sh`、`bash.*<(curl`、`wget.*\|.*sh` 等模式。发现任何从 `main`/`HEAD` 分支拉取并执行的模式时，评估是否可以改为固定 tag + 校验和。

5. **用户输入路径穿越检查**：搜索所有将用户输入直接拼接为文件路径的 shell 代码（如 `${VAR}/${user_input}`），确认在拼接前做了白名单过滤或字符集限制。

### 5.2 灵感 → 实施路径

**灵感 A：hooks.json 的"统一转发"模式**

该项目 hooks.json 的核心价值在于：所有钩子事件都通过同一个 `hook-wrapper.sh` 入口，用事件名作为参数区分处理函数。这使 hooks.json 本身保持极度简洁（无业务逻辑），所有复杂处理下沉到 shell 脚本或二进制。

实施路径：
- 在自己的钩子设计中，将 `hooks.json` 视为"路由表"而非"处理器"
- 钩子 command 只写 `wrapper.sh <EventName>`，wrapper 内部再做分发
- 好处：hooks.json 结构变化频率极低（接近只读），wrapper 可以自由迭代而无需修改 NL 层

**灵感 B：多平台 fallback 链作为健壮性设计**

`sounds.md` 的 fallback 链（插件二进制 → 系统音效目录 → 提示用户手动操作）提供了一个分层降级模板：主路径失败时不直接报错，而是尝试次级方案，最终才给出无法继续的提示。

实施路径：
- 识别自己命令中依赖外部资源（二进制、网络、文件）的操作
- 为每个依赖点设计 fallback：外部二进制不存在 → 用内置 shell 实现简化版本；网络不通 → 使用缓存；文件不存在 → 提示用户安装步骤
- 在命令文档中明确记录每条 fallback 路径，使用户知道部分失败时会发生什么

**灵感 C：废弃时的安全降级清单**

该项目废弃处理的缺失是一个反面教材。正确的废弃流程应包含：

实施路径（当决定废弃一个有用户的插件时）：
1. 立即将所有动态 URL（`main` 分支）替换为最后一个稳定 release tag 的 URL，防止 main 分支未来的意外写入被用户自动执行
2. 在 README 顶部用显著标记（如 `> [!CAUTION] 该项目已停止维护`）告知用户
3. 使用 GitHub 的"Archive repository"功能冻结代码库，阻止任何新的 push
4. 在 plugin.json 的 `description` 字段加入废弃提示，使 Claude Code UI 中也可见
5. 如果有明确的替代项目，在 README 和 plugin.json 中均添加链接
