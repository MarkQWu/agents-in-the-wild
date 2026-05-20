# 777genius/claude-notifications-go — 学习案例

**仓库**：https://github.com/777genius/claude-notifications-go
**Stars**：528 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-20（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `security-gate`, `manifest-discipline`, `curl-pipe-bash-risk`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Code 的 Go 语言实现智能通知插件。当 Claude Code 完成任务、需要等待输入、子代理停止时，自动发送系统通知（macOS 原生通知、Linux 通知、Windows Toast）和声音提示。版本 1.39.1，作者 777genius（Iliya Zelenko）。用户通过 `claude plugin install` 或 one-liner `bash` 脚本安装。

关键事实：
1. 架构核心是一个 Go 编译的二进制文件，Markdown 命令只是「下载/配置」层，不是业务逻辑层
2. 有 `.orphaned_at` 文件（时间戳 1766603417062，约 2026-03），说明本仓库曾被 NLPM auditor 标记为孤儿仓库
3. Security 状态：BLOCKED（Critical 级别：curl-download-execute 模式）
4. plugin.json 中有 4 个命令，但 `sounds.md` 和 `notifications-sounds.md` 未注册
5. 所有 6 个命令文件均缺少 `name` [frontmatter](#frontmatter) 字段

### 1.2 架构剖析
- **目录结构**：
```
claude-notifications-go/
  .claude-plugin/
    plugin.json           # 插件 manifest（v1.39.1）
  commands/
    init.md               # 下载 Go 二进制
    settings.md           # 通知设置
    sounds.md             # 声音列表（未注册！）
    notifications-init.md # 别名
    notifications-settings.md # 别名
    notifications-sounds.md   # 别名（未注册！）
  hooks/
    hooks.json            # 5 个 hook 事件 → hook-wrapper.sh
  bin/
    install.sh            # Go 二进制安装脚本
    hook-wrapper.sh       # Hook 入口，自动更新二进制
    bootstrap.sh          # 跨平台安装
  cmd/
    claude-notifications/ # Go 源码
  sounds/                 # .mp3 音效文件
```
- **文件类型分布**：6 个 command，1 个 hooks config，N 个 shell/Python 脚本，0 个 skill，Go 源码包
- **编排关系**：NL 层（commands/）负责下载配置 → hooks/hooks.json 触发 hook-wrapper.sh → Go 二进制执行实际通知逻辑。是典型的「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」分层
- **跨件契约**：`hook-wrapper.sh` 在每次钩子触发时检查二进制版本，版本不匹配时自动调用 `install.sh` 更新。NL 命令层不直接处理通知，只负责下载二进制和配置用户偏好

### 1.3 设计思路 / 方法论
- **核心设计哲学**：把性能敏感的通知投递逻辑移到 Go 二进制里（低延迟、跨平台 API 调用），把用户交互层保留在 Markdown（可读、可定制）——NL 和原生代码各司其职
- **解决什么问题**：Claude Code 没有内置通知机制，用户不知道后台任务何时完成。这个插件在 5 个 hook 事件上（PreToolUse、Notification、Stop、SubagentStop、TeammateIdle）注入通知，填补这个体验空白
- **做了什么 trade-off**：Go 二进制带来了跨平台优势和低延迟，但引入了安装复杂性（需要下载预编译二进制）和安全风险（curl-bash 下载链）。Markdown 命令层极薄，业务逻辑全在二进制里，这意味着 NL 层几乎没有可测试的业务逻辑
- **反映什么认知模型**：作者把 Claude Code 插件视为「胶水层」——NL 命令负责配置体验，原生代码负责实际功能。这和传统 CLI 工具的设计思路一致，只是把 CLI 入口替换成了 Markdown 命令

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 表皮 + 原生二进制核心」（NL-Binary Hybrid）**

关键特征：Claude Code 的 Markdown 组件只负责协调和配置，真正的业务逻辑在 Go/Rust/C 编译产物里。

模式特征清单（4 条）：
- 特征 1：commands/ 文件是「安装/配置向导」，而不是「执行业务逻辑」
- 特征 2：hooks/ 触发原生二进制，而非 Claude 模型
- 特征 3：hook-wrapper.sh 承担自动版本管理，把 NL 层和二进制版本解耦
- 特征 4：`disable-model-invocation: true` 在 init 命令中关闭模型推理，因为下载操作不需要 AI

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 系统通知、声音播放等需要 OS API 的功能 | ✅ 高度适用 | Python/Shell 调 OS API 麻烦，Go/Rust 天然优势 |
| 高频 hook（每次工具调用触发）的操作 | ✅ 适用 | 二进制启动延迟 <10ms，避免 Python 解释器开销 |
| 纯 NL 推理任务（分析、生成、审查） | ❌ 不适用 | 引入二进制的安装和更新复杂性得不偿失 |
| 需要快速迭代业务逻辑的场景 | ❌ 不适用 | 每次改业务逻辑都要重新编译发布二进制 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制核心（本仓库） | claude-notifications-go | 高性能，跨平台 OS API，低延迟 hook | 安装复杂，curl-bash 安全风险，debug 困难 |
| 纯 Markdown 命令 | 大多数 command 插件 | 无安装负担，透明可读 | 无法调 OS API，依赖 Claude 模型执行 |
| Python 脚本 + Markdown | AgriciDaniel/claude-ads 的 scripts/ | 比二进制易改，跨平台 | 比二进制慢，需要 Python 环境 |

### 2.4 改进空间
1. **当前问题**：curl-bash 下载链没有完整性验证（init.md 直接 curl main 分支）。**改进做法**：把下载 URL 固定到 release tag 版本（如 `v${VERSION}/bin/install.sh`），并在下载后用 SHA256 校验 script。**预期收益**：消除 Critical 安全风险，能开放 PR 提交。
2. **当前问题**：sounds.md 和 notifications-sounds.md 不在 plugin.json 的 commands 数组里，用户无法调用。**改进做法**：在 plugin.json 的 `commands` 数组里加上 `"./commands/sounds.md"` 和 `"./commands/notifications-sounds.md"` 两条。**预期收益**：修复 bug #1 和 #2，声音相关命令对用户可见。
3. **当前问题**：所有 6 个命令文件缺 `name` 字段，UI 展示可能错误。**改进做法**：在每个命令 [frontmatter](#frontmatter) 里加 `name: claude-notifications-go:init` 等。**预期收益**：命令在 Claude Code UI 里正确显示名称。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **63/100**，Security 状态：**BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/init.md | 53 | 缺 name（-25），无输出格式（-10），无空输入处理（-10） |
| commands/notifications-settings.md | 53 | 缺 name（-25），4 个未使用工具（-12），无输出格式（-10） |
| commands/sounds.md | 55 | 缺 name（-25），未注册到 plugin.json，无输出格式（-10） |
| commands/settings.md | 60 | 缺 name（-25），无空输入处理（-10） |
| .claude-plugin/plugin.json | 70 | sounds.md 和 notifications-sounds.md 未注册 |
| hooks/hooks.json | 90 | 结构完好，无问题 |

### 3.2 当时值得借鉴的模式
1. **`disable-model-invocation: true`**（单职责）→ 为什么好：下载二进制是确定性操作，不需要 AI 推理，显式关闭模型调用节省 token、降低延迟、避免 AI「发挥创意」 → 原文路径：`commands/init.md` frontmatter → 如何借鉴：对所有纯机械操作（文件下载、脚本执行、格式转换）的命令，考虑设置 `disable-model-invocation: true`
2. **hooks.json 结构规范**（template-design）→ 为什么好：5 个 hook 事件统一注册到同一个 hook-wrapper.sh 入口，逻辑集中，维护成本低 → 原文路径：`hooks/hooks.json` → 如何借鉴：如果需要对多个 hook 事件做类似处理，用一个 wrapper script 作为统一入口，而不是为每个事件写单独的处理脚本
3. **别名命令模式**（fallback-chain）→ 为什么好：`notifications-init.md` 作为 `init.md` 的别名，允许用户用旧习惯的命名前缀调用 → 原文路径：commands/notifications-*.md 系列 → 如何借鉴：在插件命名策略改变时用别名保持向后兼容，但要记得把别名也注册到 plugin.json

### 3.3 当时的缺陷
1. **所有命令缺 `name` 字段**：6 个命令文件都没有 `name` frontmatter 字段。为什么会失败：Claude Code 用 `name` 字段来注册命令入口，缺失时要么显示路径名（难看）要么注册失败（依版本而定）。**自查**：我的 commands/SKILL.md 里有没有 `name` 字段？
2. **声音命令孤儿化**：`sounds.md` 和 `notifications-sounds.md` 存在于磁盘但不在 plugin.json 里。为什么会失败：Claude Code 不扫描 commands/ 目录，只加载 plugin.json 里列出的路径。没在 manifest 里的文件对用户完全不可见。**自查**：我的 plugin.json 里是否列出了所有 commands/ 和 skills/ 文件？
3. **别名文件声明了从不使用的工具**：notifications-init.md 声明了 `allowed-tools: Bash`，但命令体只是告诉用户运行另一个命令，Bash 从未被调用。为什么会失败：声明了但不用的工具会引发 NLPM 的「工具声明不匹配」惩罚（-3/工具），更重要的是让 Claude 误以为这个命令会执行 Bash 操作，可能影响权限判断。**自查**：我的命令文件 allowed-tools 声明是否和实际使用一致？

### 3.4 当时的优化机会
1. **plugin.json 里加上两个缺失的命令路径**（Bug 级别，立即可提 PR）
2. **给所有 6 个命令文件加上 `name: claude-notifications-go:<cmd>`**（Bug 级别）
3. **把 init.md 里的下载 URL 固定到 release tag**（Medium 安全修复，能解封 BLOCKED 状态）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| init.md 缺 name 字段 | `grep "^name:" commands/init.md` | **持续存在**：仍无 `name:` 字段，frontmatter 只有 description/disable-model-invocation/allowed-tools | 命令注册仍依赖路径名显示 |
| sounds.md 未在 plugin.json | `grep "sounds" .claude-plugin/plugin.json` | **持续存在**：plugin.json 只列 init.md、settings.md、notifications-init.md、notifications-settings.md，无 sounds | 声音命令对安装用户仍不可见 |
| curl-bash Critical 风险 | `grep "curl.*bash" commands/init.md` | **持续存在**：init.md 仍包含直接下载执行逻辑，未固定到 release tag | BLOCKED 状态未解，无法提 PR |

### 4.2 架构演进
版本从 audit 时约 v1.x 到现在 v1.39.1（微版本号很高），说明在 Go 二进制层（cmd/ 目录）有持续迭代，功能不断扩展。NL 层（commands/）结构完全没变，所有 audit 发现的 bug 均未修复。仓库根目录增加了 `.orphaned_at` 时间戳文件，这是 NLPM auditor 标记「该仓库 PR 长期未回应」的追踪机制——说明至少有一个 PR 被提出但没有得到维护者响应。

**作者后来意识到的**：Go 层功能迭代是优先项，NL 层的结构合规性维护不在作者关注范围内。

### 4.3 新增的可学习模式
当前 HEAD 引入了 `.orphaned_at` 追踪文件。这不是作者自己加的，而是 NLPM auditor pipeline 在检测到「PR 提出后长期无响应」时写入的标记——这是一个有趣的生态观察：工具链在观察者效应层面影响被观察对象的元数据。这对我有什么意义：当一个仓库有 `.orphaned_at` 标记时，说明维护者可能已不活跃，对该仓库的依赖要谨慎。

---

## 五、校准

### 5.1 我已经在做对的
1. **manifest 同步**：我在每次新增 command 或 skill 时都会同步更新 plugin.json，不会出现「文件在磁盘但不在 manifest」的问题
2. **工具声明精确**：我的 allowed-tools 声明和实际调用一致，不声明冗余工具
3. **命令有 name 字段**：我的 commands 文件都有正确的 `name` frontmatter
4. **安全意识**：我的命令文件不包含 curl-bash 直接下载执行模式

### 5.2 挑战 / 验证
- **这次案例挑战了我的一个假设**：「Go 二进制插件比纯 NL 插件更「专业」、质量更高」。实际上本仓库的 Go 层写得很好（版本 1.39.1 说明功能成熟），但 NL 层（commands/）质量极低（63/100），两层质量完全脱钩。这说明：一个插件的「功能层」质量和「NL 接口层」质量是独立维度，都需要分别关注。
- **验证了一个对我有意义的认知**：plugin.json 是 NL 层的「唯一真相源」（single source of truth），任何不在里面的文件，无论磁盘上存不存在，对用户都是透明的。这强化了我「每次改 commands/ 必须同步改 plugin.json」的习惯。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查 plugin.json 里列出的命令文件是否和 commands/ 目录一致
comm -23 \
  <(ls commands/*.md 2>/dev/null | sort) \
  <(python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); print('\n'.join(d.get('commands',[])+d.get('skills',[])+d.get('agents',[])))" | sed 's|^\./||' | sort)
# 命中后（有不在 plugin.json 里的 .md 文件）：把它加进 plugin.json

# 检查所有命令文件是否有 name 字段
for f in commands/*.md skills/*/SKILL.md; do
  grep -q "^name:" "$f" || echo "MISSING name: $f"
done
# 命中后：在 frontmatter 里加 name 字段

# 检查 allowed-tools 和实际使用是否匹配（检查声明了 Bash 但没有调用的情况）
for f in commands/*.md; do
  if grep -q "Bash" "$f" && ! grep -q "^\`\`\`bash\|Bash(" "$f"; then
    echo "POSSIBLY UNUSED Bash: $f"
  fi
done
# 命中后：确认命令体是否真的需要 Bash，不需要就移出 allowed-tools
```

### 6.2 灵感 → 实施路径
1. **想法**：对自己的钩子驱动插件，在 hook-wrapper 脚本里加版本检查日志（打印到 stderr 而非 /dev/null），让用户能看到自动更新
   - **为何可行**：本仓库的 hook-wrapper.sh 把自动更新静默压制（`>/dev/null 2>&1 || true`），这是一个调试盲点；保留 stderr 日志但不打印到 Claude Code 主界面，是个平衡方案
   - **第一步**：把 hook-wrapper.sh 里的 `>/dev/null 2>&1 || true` 改为 `>> ~/.claude/logs/plugin-update.log 2>&1 || true`，大约 5 分钟
2. **想法**：仿照本仓库的别名命令模式，给自己的插件命令加向后兼容别名，然后在 plugin.json 里同时注册
   - **为何可行**：命名约定会变，别名让旧用户不需要更新肌肉记忆
   - **第一步**：每个改名的命令，创建一个新的 `old-name.md` 文件，文件体只有一句「此命令已重命名为 `/plugin:new-name`」并 `disable-model-invocation: true`；同步更新 plugin.json

---

## 七、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心（NL-Binary Hybrid）
> 插件架构的一种模式：Claude Code 的 Markdown 命令（NL 层）负责用户交互和配置，实际的功能逻辑放在用 Go、Rust、C 等编译型语言写的二进制文件里。用户看到的是 Markdown 命令，但背后调用的是高性能原生程序。类比：餐厅的菜单（NL 层）是给顾客看的，厨房里的炉灶和锅（原生二进制）才是实际做菜的地方。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools` 等）。Claude Code 读命令/技能/代理文件时先解析 frontmatter 才能知道如何注册和调用。`---` 必须严格从行首（第 1 列）开始。

### <a name="hook"></a>hook（钩子）
> Claude Code 在特定事件发生时自动触发的外部脚本。例如 `Stop`（任务完成）、`Notification`（需要用户注意）等。`hooks/hooks.json` 定义了每个事件触发哪个脚本。本仓库用 5 个 hook 事件来触发通知，是把 Claude Code 任务状态暴露给操作系统的核心机制。
