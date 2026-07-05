# ykdojo/claude-code-tips — 学习案例

**仓库**：https://github.com/ykdojo/claude-code-tips
**Stars**：7505 | **来源**：upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-07-05（基于当前 HEAD）
**主题标签**：`security-gate`, `curl-pipe-bash-risk`, `vague-quantifier`, `manifest-discipline`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
ykdojo/claude-code-tips 是 YouTube 创作者 ykdojo（约 10 万订阅）的个人 Claude Code skill 套件，7505 颗星。涵盖 GitHub Actions 辅助、会话克隆（clone/half-clone）、交接文档生成（handoff）、HN 摘要、版本推荐等实用工具。audit 后 2.5 个月，该仓库在安全方面有**重大进展**：两个 HIGH 安全漏洞均已修复。

关键事实：
- 高知名度个人 skill 仓库，创作者背景是 AI 教育内容
- `clone` 和 `half-clone` skill 实现了会话状态持久化（保存到 `~/.claude/history.jsonl`），是相对罕见的「跨会话记忆」实现
- audit 后新增 2 个高质量 skill：`hn-summarize`（HN 热门摘要）和 `version-check`（Claude Code 版本推荐）
- 整个 `.claude/commands/` 目录在当前 HEAD 中**已移除**，命令层完全消失

### 1.2 架构剖析
```
claude-code-tips/
├── .claude-plugin/
│   └── plugin.json              ← 注册 6 个 skill（audit 时 6 个，现仍 6 个但内容有变）
├── skills/
│   ├── clone/SKILL.md           ← 会话克隆（调用 clone-conversation.sh）
│   ├── half-clone/SKILL.md      ← 半克隆（保存当前窗口状态）
│   ├── handoff/SKILL.md         ← 生成下一个会话的交接文档
│   ├── gha/SKILL.md             ← GitHub Actions 错误分析
│   ├── reddit-fetch/SKILL.md    ← Reddit 内容获取
│   ├── review-claudemd/SKILL.md ← CLAUDE.md 质量审查
│   ├── hn-summarize/SKILL.md    ← 【新增】HN 热门摘要（使用 Firebase + Algolia API）
│   └── version-check/SKILL.md  ← 【新增】Claude Code 版本推荐
├── scripts/
│   ├── clone-conversation.sh   ← 会话克隆核心脚本
│   ├── half-clone-conversation.sh
│   ├── setup.sh                ← 安装脚本（安全已改善）
│   └── context-bar.sh          ← statusLine hook 脚本
└── CLAUDE.md                   ← 项目根文档
```

- **文件类型分布**：8 个 SKILL.md（audit 时 6 个）+ 0 个 command（audit 时 1 个）+ 6 个 shell 脚本 + 1 个 plugin.json
- **编排关系**：skill 调用脚本完成复杂操作（clone/half-clone → shell 脚本）；纯 NL skill 直接在 body 里声明步骤（gha、handoff、reddit-fetch）
- **跨件契约**：`clone/SKILL.md` 明确引用 `scripts/clone-conversation.sh`；`context-bar.sh` 通过 `setup.sh` 安装并注册为 statusLine hook——两处均有明确的文件路径契约

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「工具是真实工作流的产物」——每个 skill 来自创作者在实际使用 Claude Code 时遇到的痛点（跨会话记忆、版本选择焦虑、HN 日常浏览）
- **解决什么问题**：Claude Code 原生不支持跨会话历史记忆；`clone` 和 `half-clone` 用 [JSONL](#jsonl) 文件模拟了一个轻量级记忆层
- **做了什么 trade-off**：写到 `~/.claude/history.jsonl`（全局文件 vs. 项目隔离）。选择全局文件意味着所有项目的历史在一起，方便跨项目查询，但有路径安全隐患
- **反映什么认知模型**：作者把 skill 理解为「个人工作流的文档化」——先有工作流，再写 skill；而不是先设计 skill 生态，再找用例

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「工作流文档化型 Skill（Workflow-Derived Skills）」**

skill 是创作者实际工作流的直接映射，每个 skill 解决一个真实的、可重现的痛点；脚本处理复杂操作，SKILL.md 只声明调用时机和参数。

模式特征清单：
- 特征 1：每个 skill 对应一个明确的使用场景（克隆 vs 半克隆 vs 交接，各自独立）
- 特征 2：复杂操作封装进 shell 脚本，skill body 只调用脚本（`Run: scripts/clone-conversation.sh`）
- 特征 3：skill 之间无依赖关系，可单独安装使用
- 特征 4：新 skill 的诞生速度体现创作者的工作流演进（hn-summarize、version-check 都是新需求）
- 特征 5：setup.sh 一键安装，降低用户配置门槛

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人工作流工具包（解决自己的痛点） | ✅ 高度适用 | skill 来自真实使用，实用性天然有保证 |
| 面向特定用户群的工具（创作者、工程师） | ✅ 适用 | 用户群和作者工作流重叠度高 |
| 通用企业级 skill 套件 | ❌ 不适用 | 个人化细节（如 ~/.claude/ 路径）难以企业化 |
| 需要严格审计和权限控制的环境 | ❌ 不适用 | setup.sh 修改全局配置，不符合受控环境要求 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 工作流文档化（本仓库） | ykdojo/claude-code-tips | 实用性高，每个 skill 都有真实使用场景 | 个人化强，移植到他人工作流需调整 |
| 领域专家 skill 套件 | twostraws/SwiftUI-Agent-Skill | 系统性，覆盖领域完整 | 需要维护知识库同步 |
| 平台内置 skill | wecode-ai/Wegent | 与平台深度集成 | 脱离 Claude 生态 |

### 2.4 改进空间
1. **当前问题**：`hn-summarize/SKILL.md` 中内嵌了完整的 Python 脚本（约 40 行），skill body 承担了脚本的角色，过于臃肿。**改进做法**：把 Python 脚本提取到 `scripts/hn-fetch.py`，skill body 只调用 `python3 scripts/hn-fetch.py`，保持 skill 层和实现层分离。**预期收益**：skill body 聚焦「何时用、为何用」，脚本专注「如何做」，各司其职。
2. **当前问题**：多个 skill（gha、reddit-fetch）仍有少量模糊量词（「relevant」、「complex searches」）。**改进做法**：「relevant error messages」→「出错的 Action step 名称 + 最后 20 行日志」；「complex searches」→「含多个关键词的 Algolia 查询」。**预期收益**：消除扣分项，提升分数到 98+。
3. **当前问题**：整个 commands 层被删除后，用户没有 `/` 命令入口，只能通过 `@skill` 方式调用。**改进做法**：评估是否需要恢复一个简化的 command 文件，或在 README 中说明「推荐通过 skill 方式调用」。**预期收益**：避免老用户因习惯命令调用而困惑。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **93/100**（9 个文件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude/commands/upgrade-patches.md` | 80 | 硬编码绝对路径 `/Users/yk/Desktop/projects/safeclaw`；缺少 `allowed-tools` |
| `CLAUDE.md` | 85 | 私有 GCS URL 和 safeclaw 容器引用，限制可移植性 |
| `skills/clone/SKILL.md` | 90 | 无 example blocks |
| `skills/gha/SKILL.md` | 96 | `relevant` ×2（−4） |
| `skills/reddit-fetch/SKILL.md` | 96 | `adjust as needed`、`complex searches`（−4） |
| `skills/review-claudemd/SKILL.md` | 98 | `recent` 略模糊（−2） |
| `.claude-plugin/plugin.json` | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **会话克隆记忆模式（clone/half-clone）** → 用 JSONL 文件持久化会话摘要，实现跨会话记忆 → 为什么好：Claude Code 原生不保留跨会话上下文，这套方案用最小代价（一个文件）解决了记忆问题 → 借鉴：echo-sleuth 已有类似方向，可参考 clone-conversation.sh 的数据格式设计
2. **`handoff` skill 的交接文档标准化** → 每次会话结束时生成结构化的「交接文档」，供下一个会话快速上手 → 为什么好：解决了长期项目「下次继续时忘了昨天做到哪」的问题 → 借鉴：gstack 的 `context-save` 和 `context-restore` 是类似思路，可以对照看差异
3. **`version-check` 的反直觉推荐逻辑** → 不推荐「最新版」也不推荐「stable」，而是推荐「落后 bleeding edge 一天的版本」→ 为什么好：提供了实用的启发式规则，而不是通用的「update when stable」建议 → 借鉴：在 skill 里提供「违反直觉但有根据的建议」比「安全但无用的通用建议」更有价值

### 3.3 当时的缺陷

1. **硬编码用户私有路径（`/Users/yk/Desktop/projects/safeclaw`）**：`upgrade-patches.md` 命令里写死了作者自己的本地路径，任何其他用户运行时都会找不到文件。根本原因：命令从个人脚本直接复制进仓库，没有清理个人化内容。自查：我的 gstack 命令中是否有硬编码的本地路径？
2. **HIGH 安全：`curl | bash` 安装 context-bar.sh**：`setup.sh` 从 GitHub raw URL 下载脚本后直接 `chmod+x` 并在 statusLine hook 中自动执行，每次 Claude Code 启动时都会运行下载的代码。根本原因：追求安装便捷性，忽视了「下载 + 执行」组合的安全风险。自查：我的 setup 脚本中是否有类似下载后执行的模式？
3. **HIGH 安全：`sudo npm install -g cc-safe`（无版本固定）**：使用 root 权限安装一个未固定版本的 npm 包，供应链被污染时会以 root 权限执行恶意代码。根本原因：全局 CLI 工具习惯用 sudo npm 安装，但忽视了版本固定的重要性。自查：我的脚本中是否有 `sudo npm install -g` 模式？

### 3.4 当时的优化机会

1. 修复 Bug #1：从 `upgrade-patches.md` 移除或参数化硬编码路径（或直接删除该命令）
2. 修复 HIGH #1：将 `curl ... | chmod+x` 替换为「curl -o 临时文件 + sha256sum 校验 + chmod+x」
3. 修复 HIGH #2：固定 `cc-safe` 版本（`sudo npm install -g cc-safe@X.Y.Z`），或改用 npx 避免全局安装

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `upgrade-patches.md` 硬编码路径 | `find . -name "upgrade-patches.md"` | **已修复**——整个 `.claude/commands/` 目录已删除，包含 upgrade-patches.md | 采用「删除问题命令」而非「修复命令」的策略，果断 |
| HIGH：`curl \| sh`（context-bar.sh 下载）| `grep -n "curl" scripts/setup.sh` → L139 | **已修复**——L139 改为 `curl -sL ... -o $CLAUDE_DIR/scripts/context-bar.sh`（保存到文件，不再 pipe to sh） | curl 不再直接执行下载内容，HIGH 降为 LOW |
| HIGH：`sudo npm install -g cc-safe` | `grep -n "sudo npm" scripts/setup.sh` | **已修复**——`sudo npm` 不再出现 | 移除了 root 权限 npm 安装，HIGH 漏洞消除 |

### 4.2 架构演进

从 audit 时的「9 个文件，1 个命令目录」到现在的「8 个 skill，0 个命令目录，新增 hn-summarize 和 version-check」。架构演进的核心是：
1. **收缩命令层**：删除了有问题的 commands 目录，简化用户接口
2. **扩充 skill 层**：新增 2 个高质量 skill，说明作者工作流在持续演进
3. **安全修复**：两个 HIGH 漏洞均在 2.5 个月内修复，响应速度快——可能因为社区有安全讨论

### 4.3 新增的可学习模式

`hn-summarize/SKILL.md` 新增的技术亮点：明确说明「hckrnews.com 是 JavaScript 渲染页面，不要 curl 它」，转而使用 HN 官方 Firebase API 和 Algolia 搜索 API。这是一种「反踩坑型 skill 设计」——不只说「怎么做」，还明确说「不能这样做，以及为什么」。

`version-check/SKILL.md` 的核心价值：用启发式规则（「Claude Code 更新很频繁，stable 不代表更稳定，落后一天是最优解」）代替简单的「装最新版」建议，展示了高级 skill 的特征——**有独立判断，而不是复述官方文档**。

---

## 五、校准

### 5.1 我已经在做对的

1. **无 `curl|sh` 模式**：gstack 和 bureau 的脚本中无直接 `curl|sh`，这个 HIGH 风险在我的仓库里没有
2. **无 `sudo npm install -g` 无版本固定**：同样未发现此模式
3. **技能来自真实工作流**：gstack 的大多数 skill（如 `context-save`、`handoff` 类似的 `context-restore`）都来自实际需要，和 ykdojo 的「工作流文档化」理念一致

### 5.2 挑战 / 验证

这次案例**挑战了我的一个假设**：「HIGH 安全漏洞需要很长时间才能修复」。ykdojo 在 2.5 个月内修复了 2 个 HIGH 漏洞，而 wecode-ai/Wegent 的 CRITICAL 漏洞在同样时间内未修。**漏洞修复速度和仓库活跃度、维护者意识强相关，而不是漏洞严重程度。** ykdojo 作为个人维护者，对社区反馈响应更快；Wegent 是商业产品，决策链更长。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的脚本中是否有 curl 直接 pipe 到 sh 的模式
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau -name "*.sh" 2>/dev/null | xargs grep -l "curl.*|.*sh\|curl.*|.*bash" 2>/dev/null || echo "未发现 curl|sh 模式 ✅"

# 检查我的命令文件是否有硬编码路径（包含 /Users/ 或 /home/ 的绝对路径）
find /tmp/my-repos/MarkQWu-gstack -name "*.md" -path "*/commands/*" 2>/dev/null | xargs grep -n "/Users/\|/home/" 2>/dev/null || echo "未发现硬编码路径 ✅"
# 命中后怎么办：将路径参数化为 $HOME 或环境变量，或删除该命令

# 检查 clone/half-clone 类型的跨会话记忆 skill 是否已在我的仓库中实现
ls /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/
# 对比 ykdojo 的 history.jsonl 格式和 echo-sleuth 的存储格式，找出可互相借鉴的地方
```

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 `version-check` 的「反直觉推荐」模式，为 gstack 写一个 `model-select` skill
   - **为何可行**：Claude Code 用户经常纠结用哪个模型（sonnet vs haiku vs opus），一个像 `version-check` 那样提供启发式判断规则的 skill 会很有价值
   - **第一步**：写 `MarkQWu-gstack/model-select/SKILL.md`，规则包括「快速单步任务 → haiku；复杂推理 → sonnet；多轮 edit → sonnet；成本敏感 → haiku 做初稿 + sonnet 做 review」

2. **想法**：为 bureau 的 `capture` skill 加「反踩坑说明」（仿 hn-summarize 的「不要 curl JS 渲染页面」写法）
   - **为何可行**：`capture` 抓取网页内容，需要告知 Claude 哪些 URL 模式是 JS 渲染无法 curl 的（SPAs、动态加载等）
   - **第一步**：在 `MarkQWu-bureau/skills/capture/SKILL.md` 开头加一个「⚠️ 不要这样做」段，列出常见失效场景（`twitter.com`、动态加载的 dashboard 等）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 ykdojo/claude-code-tips 的核心目的**：为 Claude Code 用户提供实用的日常工作流 skill 套件
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是个人工作流 Claude Code skill 套件；都有跨会话记忆类 skill | gstack 面向 iOS 开发 + 全栈；ykdojo 更通用（HN、Reddit、GHA） | 高 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有跨会话历史记忆功能 | echo-sleuth 专注从历史挖掘规律，ykdojo 专注保存当前会话状态 | 高 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 硬编码绝对路径 | `grep -rn "/Users/\|/home/" gstack/*/SKILL.md` | 未执行；需手动核查 gstack 中与本机路径相关的 skill（如 ios 系列） | 中 |
| curl\|sh 安全模式 | `find -name "*.sh" -exec grep -l "curl.*\|.*sh"` | 无命中 ✅ | N/A |
| skill 缺少 example blocks | `grep -L "## Example\|## 示例" gstack/*/SKILL.md` | gstack 大多数 skill 无 example blocks（待核查） | 中 |

**命中后的具体行动建议**：
- gstack 的 `ios-fix`、`ios-qa` skill 可能引用 iOS Simulator 的本机路径 → 打开这两个文件核查 → 参数化为 `$HOME` 或 `$SIMULATOR_PATH` → 15 分钟可完成
- gstack 中无 example blocks 的 skill（预计多数）→ 优先为 `review/SKILL.md` 和 `qa/SKILL.md`（最高频）补一个具体 input→output 示例 → 每个 30 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：「反踩坑型 skill 设计」（hn-summarize 的 JS 渲染说明）
   - **本案例做法**：`hn-summarize/SKILL.md` 开头明确说明「hckrnews.com 是 JS 渲染页面，不要 curl 它」，然后给出正确替代方案（Firebase + Algolia API）
   - **我的项目现状**：`MarkQWu-bureau/skills/capture/SKILL.md` 和 `MarkQWu-gstack/browse/SKILL.md` 均无此类踩坑说明
   - **如何借鉴**：在 capture 和 browse 的 skill body 开头加「⚠️ 以下 URL 类型无法用 curl 获取有效内容：[JS SPA、动态加载页面]，改用 Playwright 或 MCP browser tool」

2. **领域**：启发式版本推荐（version-check 的「落后一天」原则）
   - **本案例做法**：`version-check/SKILL.md` 提供了具体的、可操作的启发式规则，而不是「参考官方文档」
   - **我的项目现状**：gstack 无 model-select 类型的 skill，用户需要自己判断用哪个模型
   - **如何借鉴**：参考 version-check 的写作风格，写一个 `model-select/SKILL.md`，给出「什么任务用什么模型」的启发式规则表

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Output Format 规范化程度
- **我的做法**：gstack 的 `review/SKILL.md` 有 49 处 output 相关内容，`qa/SKILL.md` 有 17 处——说明对输出格式有详尽的声明
- **本案例做法（弱在哪）**：ykdojo 的 `clone/SKILL.md` 和 `half-clone/SKILL.md` 各自无 example blocks，输出格式隐式而非显式
- **意义**：在输出格式规范化上，gstack 优于 ykdojo；若进行跨 skill 套件比较评审，这是我的优势点

---

## 八、术语表

### <a name="jsonl"></a>JSONL（JSON Lines）
> 每行是一个独立 JSON 对象的文本格式，常用于日志和数据流。ykdojo 的 `clone-conversation.sh` 把每次会话摘要追加到 `~/.claude/history.jsonl`，每行一条记录，易于按时间顺序读取和追加，同时不需要加载整个文件。

### <a name="sha256sum"></a>SHA-256 校验（sha256sum）
> 对文件内容计算一个 64 位的「指纹」字符串。在下载脚本后立即校验这个指纹，可以确认文件没有被篡改——如果校验值和预期不符，说明文件在传输中被修改了，应拒绝执行。这是 `curl | sh` 的安全替代方案中的关键步骤。

### <a name="statusline-hook"></a>statusLine Hook
> Claude Code 的一种 hook 类型，在终端底部的状态栏显示自定义信息。`context-bar.sh` 通过 statusLine hook 在每次操作后实时显示上下文用量。因为这个脚本在每次 Claude Code 事件时都会运行，如果脚本本身有问题，整个 Claude Code 工作流都会受影响，所以安全要求比普通脚本更高。

### <a name="bleeding-edge"></a>Bleeding Edge（前沿版本）
> 指刚刚发布、还没有经过社区大规模使用验证的最新版本。`version-check` 的建议是「避开只有几小时的版本」——因为 Claude Code 更新频繁（有时每天 1-2 次），同一天内可能有社区发现 regression（新功能破坏了旧功能）；落后一天能规避这类风险。

### <a name="curl-pipe-bash"></a>curl-pipe-bash（curl|bash）
> `curl <URL> | bash` 的缩写，从网络下载脚本并立即用 bash 执行。ykdojo 在 audit 后修复了此模式：不再 pipe 到 bash，而是 `curl -o <文件>` 先保存到本地，再分步处理。这是一个简单但有效的安全改进，不需要 SHA 校验也比直接 pipe 安全得多。
