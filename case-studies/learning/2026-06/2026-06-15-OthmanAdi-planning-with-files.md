# OthmanAdi/planning-with-files — 学习案例

**仓库**：https://github.com/OthmanAdi/planning-with-files
**Stars**：⭐未收录 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-15（基于当前 HEAD）
**主题标签**：`template-design`, `manifest-discipline`, `security-gate`, `cross-reference`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
planning-with-files 是一个「Manus 式」的文件化任务规划插件，当前版本 3.1.1，核心思路是：让 AI 编程助手（Claude Code、Cursor、Codex、Kiro 等）通过维护三个 Markdown 文件（`task_plan.md`/`findings.md`/`progress.md`）来实现跨 context 重置的持久化规划状态。即使用户执行 `/clear` 或 context 被压缩，计划状态都不会丢失。更重要的是，这套 [SKILL.md](#skill-md-标准) 规范可以被 11 种以上的 AI 工具读取，是目前跨 IDE 兼容性最广的规划插件之一。

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── .claude-plugin/plugin.json      # Claude Code manifest
  ├── skills/                          # 规范主体（6 种语言）
  │   ├── planning-with-files/SKILL.md # 英文主版本
  │   ├── planning-with-files-ar/      # 阿拉伯语
  │   ├── planning-with-files-de/      # 德语
  │   ├── planning-with-files-zh/      # 简体中文
  │   ├── planning-with-files-zht/     # 繁体中文
  │   └── planning-with-files-es/      # 西班牙语
  ├── commands/                        # 7 个命令（plan, start, status + 多语言变体）
  ├── scripts/                         # 15+ 个 shell/PowerShell/Python 脚本
  │   ├── attest-plan.sh / .ps1       # v2.37.0 新增：SHA-256 计划完整性证明
  │   ├── init-session.sh / .ps1      # 初始化规划会话
  │   ├── session-catchup.py           # 上下文恢复脚本
  │   ├── resolve-plan-dir.sh          # 多计划目录解析
  │   └── inject-plan.sh               # 计划内容注入钩子
  ├── .claude/   .cursor/  .gemini/   # 各 IDE 的规范副本（hooks + SKILL.md 变体）
  ├── .mastracode/ .codex/ .opencode/ # 更多 IDE 变体
  ├── .kiro/   .continue/  .codebuddy/# 更多 IDE 变体
  └── .pi/                             # TypeScript 扩展（v3.x 重新设计，不再是 SKILL.md）
  ```
- **文件类型分布**：14 个 SKILL.md（6 语言 × 多 IDE）/ 7 个 command / 15+ 个 shell/Python 脚本 / 1 个 manifest
- **编排关系**：commands 直接调用对应 skill（`plan.md` → `planning-with-files:planning-with-files`），skill 通过 hooks（PreToolUse/Stop/SessionStart）自动注入 task_plan.md 内容到模型上下文，无需用户手动触发
- **跨件契约**：`scripts/sync-ide-folders.py` 负责把 canonical `skills/planning-with-files/SKILL.md` 同步到所有 IDE 变体目录；所有 hook 脚本通过 `${CLAUDE_PLUGIN_ROOT:-$HOME/.claude/plugins/planning-with-files}` 动态解析脚本路径

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「文件即状态」—— 不依赖 AI 的 context window 或记忆来维持规划状态，所有状态写到文件里，任何 AI 工具在任何时刻读这些文件都能接续工作
- **解决什么问题**：解决复杂任务（多阶段、多天、需要多次对话）在 context 重置后「AI 忘了在做什么」的问题
- **做了什么 trade-off**：每次工具调用前都注入 task_plan.md 头 30 行（PreToolUse hook）——增加了每次调用的 token 消耗，换来了 AI 始终知道当前计划状态；v2.37.0 引入的 `attest-plan.sh` 在注入前验证文件哈希，防止注入被篡改的计划内容
- **反映什么认知模型**：作者把 AI agent 视为「无记忆的执行者」，规划状态不应依赖 AI 的连续性，而应外化到文件系统

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「规范驱动 + 跨 IDE 同步」**：一份规范（canonical SKILL.md）通过同步脚本分发到 11+ 个 IDE 专用格式，每个 IDE 变体只改 hook 格式，核心内容保持一致。

模式特征清单：
- 特征 1：Canonical SKILL.md 是单一真实来源，IDE 变体通过脚本派生，不手动维护
- 特征 2：Hook 层（PreToolUse/Stop/SessionStart）把规划状态自动注入 AI 上下文，对用户透明
- 特征 3：三文件状态分离：task_plan.md（计划）/ findings.md（研究发现）/ progress.md（会话日志）
- 特征 4：多语言本地化（6 种语言）内置，通过命令变体（plan-zh、plan-ar 等）激活
- 特征 5：v3.x 引入 SHA-256 attestation 防止计划内容被恶意注入（安全硬化）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多天多会话的复杂开发任务 | ✅ 高度适用 | 文件化状态让 context 重置不影响进度 |
| 多 agent 并发规划（PLAN_ID 特性）| ✅ 适用 | v3.x 的 PLAN_ID 支持多个 agent 共享同一个计划目录 |
| 简单单次问答任务 | ❌ 不适用 | 维护三个文件的开销大于收益 |
| 高安全要求场景（防止提示注入）| ⚠️ 谨慎 | PreToolUse 每次调用都注入 task_plan.md 内容，如果该文件来自外部（如网络爬取），存在提示注入放大风险；attest-plan 可以缓解 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 文件化状态 + 跨 IDE（本仓库）| planning-with-files | 无 AI 状态依赖、跨工具可用、可审计 | PreToolUse 增加 token 消耗、三文件维护有学习成本 |
| 内存型规划（依赖 context）| 大多数规划 prompt | 简单、无文件 | context 重置后丢失全部状态 |
| 数据库型状态（SQLite/JSON）| claude-task-master 类 | 结构化、可查询 | 需要代码支持，无法跨 AI 工具 |

### 2.4 改进空间
1. **当前问题**：所有 command（plan.md、plan-ar.md 等 7 个）仍然缺少 `allowed-tools` 声明。**改进做法**：在每个 command frontmatter 加 `allowed-tools: [bash, read, write, edit]`（planning 任务需要读写文件）。**预期收益**：权限边界明确，NLPM 评分提升
2. **当前问题**：session-start hook 把 session-catchup.py 的完整输出（可能含前次会话的外部网络内容）无截断注入模型 context。**改进做法**：加 `head -100` 截断，或在输出前加 `[PRIOR SESSION CONTEXT - potentially untrusted]` 前缀。**预期收益**：降低前次会话内容中的恶意指令被当前 session 执行的风险
3. **当前问题**：各 `## Examples` 块缺失（25 个 artifact 全部缺少）。**改进做法**：在主 SKILL.md 添加一个完整示例：用户触发 `/plan` → Claude 创建三文件 → PreToolUse hook 注入 task_plan.md → Claude 按计划执行。**预期收益**：NLPM 分数从 91 升至约 96

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **91/100**（当时版本 2.33.0）

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .pi/skills/planning-with-files/SKILL.md | 82/100 | 缺 allowed-tools、PowerShell 语法错误（缺 `$` 前缀）|
| .mastracode/skills/.../SKILL.md | 85/100 | `version: "2.33.0"` 重复 YAML 键 |
| .cursor/.codebuddy/.codex/.opencode 各 SKILL.md | 85/100 | 同上，重复 version 键 |
| commands/status.md | 90/100 | 缺 allowed-tools |
| .kiro/skills/SKILL.md | 92/100 | 版本比 canonical 低一个 minor version（2.32.0-kiro vs 2.33.0）|
| skills/planning-with-files/SKILL.md（主版本）| 95/100 | 无专用 `## Examples` section（内联示例存在但格式非标准）|

### 3.2 当时值得借鉴的模式
1. **三文件状态分离设计** → task_plan.md（计划）/ findings.md（研究）/ progress.md（日志）职责清晰、不混用 → 可借鉴到 claude-for-legal：把监管变化追踪、分析发现、日志分开存放
2. **Security Boundary 自我文档** → SKILL.md 里有专门的 Security Boundary 段，声明 task_plan.md 可能含外部内容的风险 → 这种「把安全风险写在 skill 里」的做法是负责任的透明度
3. **sync-ide-folders.py 规范同步** → 维护 11+ 个 IDE 变体时，通过脚本而不是手工维护，单一真实来源 → 可借鉴到多平台插件开发
4. **多语言 command 变体** → plan/plan-zh/plan-ar/plan-de 等命令让非英语用户有本地化入口 → echo-sleuth 目前只有英文命令
5. **完整的 CONTRIBUTORS.md + CITATION.cff + CHANGELOG.md** → 开源项目规范完备，降低贡献门槛 → 我的插件完全缺这些

### 3.3 当时的缺陷
1. **问题**：5 个 IDE 变体 SKILL.md 有重复的 `version: "2.33.0"` YAML 键 → **根本原因**：sync-ide-folders.py 在同步时引入了 bug，或者手工编辑时复制了两行 → YAML 严格解析器会报错或静默取最后一个值，造成版本读取不一致 → **自查**：我的 SKILL.md 有没有重复的 YAML 键？
2. **问题**：`.pi/skills/SKILL.md` 里 PowerShell 路径缺少 `$` 前缀，语法错误 → **根本原因**：`.pi` 平台的特殊 PowerShell 调用语法没有经过测试，sync 脚本没有覆盖这个平台 → **自查**：我有没有针对没测试过的平台写脚本？
3. **问题**：session-start hook 把前次会话的完整 catchup 输出无截断注入模型 → **根本原因**：追求「最完整的上下文恢复」却忽略了注入内容可能含恶意指令 → **自查**：我的 hooks 有没有无截断注入外部内容？

### 3.4 当时的优化机会
1. 修复 5 个 SKILL.md 的重复 YAML version 键（机械修复，脚本可自动化）
2. 给 session-start hook 的 catchup 输出加 `head -100` 截断
3. 在 plan.md 等 7 个 command 文件添加 `allowed-tools`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 5 个 SKILL.md 有重复 version 键 | `grep -c "version" .mastracode/skills/planning-with-files/SKILL.md` | **已修复** ✅（v3.1.1 的 mastracode SKILL.md 只有 1 处 version 出现）| sync 脚本在 v3.x 重构中修复了这个问题 |
| .pi PowerShell 语法错误 | `find .pi -name "SKILL.md"` | **已重新设计** ✅（.pi 目录现在是 TypeScript 扩展：index.ts + attestation.ts，不再是 SKILL.md）| 维护者选择彻底重新设计 .pi 集成，而不是修 bug |
| Commands 缺 allowed-tools | `grep "allowed-tools" commands/plan.md` | **仍未修复**（plan.md frontmatter 仍只有 `description`）| 这是低优先级质量问题，维护者专注于功能迭代 |

### 4.2 架构演进
从 2.33.0 到 3.1.1 是一次重大演进：
- 新增 **attest-plan.sh**（v2.37.0 引入）：用 SHA-256 哈希锁定 task_plan.md 内容，PreToolUse hook 在注入计划前验证哈希，防止计划被篡改后被注入执行——这是对 audit 安全发现的正面回应
- 新增 **PLAN_ID 多代理共享状态**：通过环境变量 `PLAN_ID` 指定计划目录，多个 agent 可以共享同一个规划状态，支持真正的多 agent 并发规划
- 新增 **ledger-append.sh**（会话账本）：记录每次会话的操作流水，比 progress.md 更细粒度
- **.pi 重新设计**：从 SKILL.md 变成 TypeScript 扩展，反映作者对 Pi 平台的深入了解
- **版本跳跃巨大**（2.33.0 → 3.1.1）：说明这个插件在积极迭代，是生态中的活跃项目

### 4.3 新增的可学习模式
**attest-plan 的 SHA-256 证明机制**是这次最值得学习的新设计：
- 用户跑 `/plan-attest` 命令后，系统计算 task_plan.md 的 SHA-256 并存起来
- 每次 PreToolUse hook 注入计划内容前，先验证哈希，发现不匹配就拒绝注入
- 这把「AI 只执行你亲手批准的计划」变成了可验证的保证，而不只是靠约定

这个思路可以推广：任何「hook 注入外部文件内容」的场景都可以加 attestation 机制。

---

## 五、校准

### 5.1 我已经在做对的
1. **单一真实来源设计**：echo-sleuth 的 MEMORY.md 是索引，单个记忆文件是内容，职责分离
2. **状态外化到文件**：echo-sleuth 把经验、决策、反馈存成 .md 文件——和 planning-with-files 的「文件即状态」思路一致
3. **CLAUDE.md 说明插件配置路径**：claude-for-legal 的 CLAUDE.md 清楚说明了配置文件的位置和读取顺序，与 planning-with-files 的文档清晰度类似

### 5.2 挑战 / 验证
这次案例最大的认知更新是**安全对抗的具体化**。我之前知道「hook 注入外部内容有风险」，但觉得这是理论风险。planning-with-files 实际实现了 SHA-256 attestation 来对抗这个风险，把抽象的安全建议变成了具体的技术措施。这让我意识到：对于会持续注入文件内容的 hook，attestation 是应该加的，不是「可选的好实践」。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 有没有重复的 YAML 键
python3 -c "
import yaml, glob
for f in glob.glob('/tmp/my-repos/MarkQWu-echo-sleuth-for-claude/**/*.md', recursive=True):
    try:
        with open(f) as fp:
            content = fp.read()
        if content.startswith('---'):
            fm = content.split('---')[1]
            yaml.safe_load(fm)
    except yaml.YAMLError as e:
        print(f'{f}: {e}')
"

# 检查我的 hooks 有没有无截断注入外部文件
find /tmp/my-repos -name "hooks.json" 2>/dev/null \
  | xargs grep -n "cat\|head\|session-catchup" 2>/dev/null

# 检查我的 commands 缺 allowed-tools
find /tmp/my-repos -name "*.md" -path "*/commands/*" 2>/dev/null \
  | xargs grep -L "allowed-tools" 2>/dev/null
```
命中后怎么办：
- YAML 重复键：删除重复行，只保留一个
- hook 无截断注入：加 `| head -100` 限制行数，或在注入内容前加 `[EXTERNAL CONTEXT]` 标签
- command 缺 allowed-tools：加 `allowed-tools: [bash, read, write]`

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 借鉴 attest-plan 思路，实现「记忆完整性检查」
   - **为何可行**：echo-sleuth 在 SessionStart 时可能注入 MEMORY.md 内容，如果记忆文件被意外修改，attestation 能保护
   - **第一步**：在 `scripts/` 下创建 `attest-memory.sh`，计算 MEMORY.md 的 SHA-256，存到 `.memory-hash`，在读取记忆前验证；约 20 分钟
2. **想法**：给 claude-for-legal 加「文件化规划状态」（参考 planning-with-files 的三文件模式）
   - **为何可行**：法律 due diligence 任务往往跨多天多会话，三文件模式完全适用
   - **第一步**：在 `regulatory-legal/CLAUDE.md` 添加「Session State」段，要求 AI 在开始工作前检查 `.legal-plan/` 目录下是否有未完成的计划文件；约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 OthmanAdi/planning-with-files 的核心目的**：让 AI agent 在多次会话间保持规划状态连续性，通过文件化状态对抗 context 重置
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都在解决「AI 跨 session 记住东西」的问题，都把状态外化到文件 | echo-sleuth 挖掘历史、planning-with-files 维护当前状态 | **高** |
| MarkQWu/claude-for-legal | 中 | 都涉及多步骤、多天的复杂工作流 | claude-for-legal 无 hook 驱动的自动状态维护 | **高** |
| MarkQWu/drama-workshop-skills | 低 | — | 完全不同领域 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 缺 `allowed-tools` | `grep -rL "allowed-tools" */commands/*.md` | claude-for-legal：regulatory-legal 的 commands 无 `allowed-tools` | **中** |
| Hook 注入外部内容无截断 | 人工检查 hooks 配置 | echo-sleuth 无 hooks（无风险）；claude-for-legal 无 hooks（无风险）| N/A |
| 缺 `## Examples` | `grep -rL "## Example" skills/*/SKILL.md` | echo-sleuth：4/4 skills 命中 | **高** |

**命中后的具体行动建议**：
- `claude-for-legal/regulatory-legal/CLAUDE.md` → 参考 planning-with-files 的三文件模式，建议 AI 把监管追踪结果写到 `.legal-state/task_plan.md` → 落地 30 分钟
- `echo-sleuth/skills/*/SKILL.md` → 每个 skill 添加 1 个 `## Examples` 块 → 约 10 分钟/个

### 7.3 别人的更优方案

1. **领域**：SHA-256 attestation 防止 hook 注入被篡改内容
   - **本案例做法**：`scripts/attest-plan.sh` 对 task_plan.md 计算哈希并存储，PreToolUse hook 在注入前验证哈希，不匹配则拒绝注入
   - **我的项目现状**：echo-sleuth 如果将来加 hook 注入 MEMORY.md，目前没有任何完整性保护
   - **如何借鉴**：在 echo-sleuth 的 `scripts/` 下参照 `attest-plan.sh` 实现 `attest-memory.sh`，hook 注入前先 verify

2. **领域**：sync-ide-folders.py 的「单一真实来源 + 脚本派生」
   - **本案例做法**：canonical SKILL.md 一份，通过 `sync-ide-folders.py` 分发到 11 个 IDE 目录，手动改 canonical 自动同步到所有 IDE
   - **我的项目现状**：claude-for-legal 的各子插件是手工独立维护，共同的配置结构（如 company-profile 引用）可能出现漂移
   - **如何借鉴**：把 claude-for-legal 的 12 个子插件的共同配置片段提取到 canonical，写脚本同步

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：配置版本升级兼容（前向迁移）
- **我的做法**：claude-for-legal 的 CLAUDE.md 包含「plugin 更新后从旧 cache 路径迁移配置」的逻辑，确保用户数据不丢失
- **本案例做法**：planning-with-files 有 MIGRATION.md，但插件更新时的用户状态迁移不如 claude-for-legal 自动化（需要用户手动参照 MIGRATION.md 操作）
- **意义**：自动配置迁移是 UX 亮点，在给上游 PR 时可以提这个思路

---

## 八、术语表

### <a name="skill-md-标准"></a>SKILL.md 标准
> 一个开放的 Markdown 文件格式约定，用于描述 AI agent 的技能和行为规范。不同的 AI 工具（Claude Code、Cursor、Codex 等）都能读取 SKILL.md 文件，按里面的指令执行任务。planning-with-files 通过为每个 IDE 维护一个适配版本的 SKILL.md，实现了跨工具兼容。

### <a name="hook"></a>hook
> Claude Code 等 AI 工具里的自动触发器，在特定事件（如 SessionStart、工具调用前 PreToolUse、会话结束 Stop）发生时，自动运行一段 shell 命令，把结果注入 AI 的对话上下文。

### <a name="attestation"></a>attestation（证明/公证）
> 用密码学哈希（SHA-256）为文件内容生成一个「指纹」，存储起来。下次使用该文件前，重新计算哈希并与存储值对比——匹配说明文件未被篡改，不匹配说明内容已更改（可能是恶意修改）。

### <a name="plan-id"></a>PLAN_ID
> 一个环境变量，指向当前活跃的计划目录名（如 `2026-01-10-backend-refactor`）。多个 AI agent 通过读取同一个 PLAN_ID 对应的目录，实现对同一份规划状态的共享访问。

### <a name="context-reset"></a>context 重置
> 用户执行 `/clear` 命令或 AI 对话窗口被关闭后，AI 失去当前对话的所有记忆的状态。file-based planning 通过把状态写入文件而不是依赖 AI 记忆，避免了 context 重置带来的进度丢失。

### <a name="manus-式"></a>Manus 式规划
> 参考 Manus AI agent 的规划方式——把任务拆分成阶段，每个阶段的状态写入文件，agent 崩溃或重启后能从文件恢复状态继续执行。与「每次都让 AI 重新规划」相比，Manus 式更适合长时运行的复杂任务。
