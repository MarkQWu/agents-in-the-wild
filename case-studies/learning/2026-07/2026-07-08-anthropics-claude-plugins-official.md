# anthropics/claude-plugins-official — 学习案例

**仓库**：https://github.com/anthropics/claude-plugins-official
**Stars**：31,759 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-08（基于当前 HEAD）
**主题标签**：manifest-discipline, examples-driven, security-gate, monorepo-vs-split, single-purpose

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Anthropic 官方维护的 Claude Code [插件](#插件) 目录，定位为"高质量官方插件展示柜"。创建于 2025-11-20，描述为"Official, Anthropic-managed directory of high quality Claude Code Plugins"。

5 个关键事实：
1. 当前 37 个核心插件（`plugins/`）+ 15 个外部社区集成（`external_plugins/`），持续活跃增长（审计时 2026-04-06 至今已新增 code-modernization、ralph-loop 等多个插件）
2. 涵盖多语言 [LSP](#LSP) 支持（gopls、rust-analyzer、pyright、kotlin-lsp 等 10+ 种）、AI 工作流（feature-dev、code-modernization、skill-creator）、MCP 服务开发（mcp-server-dev、mcp-tunnels）、学习风格（learning-output-style、explanatory-output-style）
3. 用户通过 `claude plugin install <name>@anthropics` 安装，仓库本身即为 Claude Code 插件注册表（marketplace）
4. 外部插件包括 Discord、Telegram、iMessage、GitHub、GitLab、Linear 等主流集成
5. 政策约束：外部 PR 被拒（NLPM 贡献被 policy_denied，因 `anthropics` 在 DENY_OWNERS 列表）

### 1.2 架构剖析

- **目录结构**：
  ```
  plugins/
    feature-dev/
      .claude-plugin/plugin.json
      agents/            ← code-architect、code-explorer、code-reviewer
      commands/
    skill-creator/
      skills/
        skill-creator/
          agents/        ← analyzer、comparator、grader（当前无 YAML frontmatter）
    hookify/
      hooks/
        hooks.json       ← 动态读取用户 .local.md 配置
        pretooluse.py
        posttooluse.py
    code-modernization/  ← 7个 agents：legacy-analyst、architecture-critic...
    mcp-server-dev/
      skills/            ← 3个技能：build-mcp-server、build-mcpb、build-mcp-app
    ... (37 个插件)
  external_plugins/
    discord/  telegram/  imessage/  github/  ...
  ```
- **文件类型分布**：审计时约 103 个 [NL artifact](#NL-artifact)；当前因新增插件已超过 120 个
- **编排关系**：每个插件高度独立，无跨插件调用。`feature-dev` 插件内部三个 agents 构成三角协作（explorer → architect → reviewer）。`skill-creator` 使用 blind comparator 模式进行技能评估
- **跨件契约**：`hookify` 读取用户 `.local.md` 文件来动态配置 hook 行为，实现"用户配置 → 插件执行"的分离

### 1.3 设计思路 / 方法论

- **核心设计哲学**：每个插件单职责（hookify 只做 hook 框架，skill-creator 只做技能创建，LSP 插件只做语言服务器集成）。插件间完全解耦——用户可按需安装任意组合
- **解决什么问题**：提供官方背书的高质量 Claude Code 插件参考实现，同时作为插件生态的事实标准（其他开发者参考这里的格式）
- **做了什么 trade-off**：选择多插件目录式 [monorepo](#monorepo)（plugins/ 下每个子目录独立有 plugin.json），而非单一大型插件。代价是用户需要单独安装多个插件；收益是职责清晰、可独立发布、测试、迭代
- **反映什么认知模型**：作者认为 Claude Code 生态应该是"插件集市"而非"全家桶"——每个插件解决特定问题，用户自行组合

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「多插件平铺 + 外部集成分层」模式**：核心特征是将第一方插件（plugins/）和第三方集成（external_plugins/）物理分离，每个插件拥有完整的独立 manifest，互不依赖。

模式特征清单：
- 特征 1：每个插件目录自包含（plugin.json + agents/ + skills/ + commands/），删除任一不影响其他
- 特征 2：两层目录（plugins/ vs external_plugins/）在目录级别表达信任等级
- 特征 3：无 meta-router（用户直接按名称安装插件，无统一入口）
- 特征 4：`hookify` 插件本身是框架，不直接提供功能，而是为用户提供配置 hook 的能力

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 覆盖多语言/多工具链的官方工具集 | ✅ 高度适用 | 每种工具独立安装，用户只装自己需要的 |
| 个人日常工作流 skill kit | ❌ 过于复杂 | 单个 plugin.json 统一管理更省事 |
| 需要插件间共享状态的工作流 | ❌ 不适用 | 各插件完全解耦，无法跨插件传递上下文 |
| 作为第三方插件的参考规范 | ✅ 高度适用 | 官方背书 + 格式规范 = 最权威参照 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多插件平铺（本仓库） | anthropics/claude-plugins-official | 高度模块化、可独立迭代 | 安装多步，用户需手动选择 |
| 单 manifest 全家桶 | mattpocock/skills | 一键安装全部技能 | 用户无法精选，manifest 容易漂移 |
| 产品线 manifest 分拆 | hashicorp/agent-skills | 按产品线隔离，可独立发版 | 多个 plugin.json 同步成本高 |

### 2.4 改进空间

1. **当前问题**：skill-creator 的三个 agents（analyzer/comparator/grader）缺 YAML [frontmatter](#frontmatter)，Claude Code 无法正确注册。**改进做法**：为每个 agent 添加标准 `---name/description/model/tools/color---` 块。**预期收益**：agents 可被正确调用，也消除了违反自家规范的尴尬
2. **当前问题**：feature-dev 和 skill-creator 的 plugin.json 均未声明 agents 数组，用户安装后无法发现这些 agents。**改进做法**：在 plugin.json 的 `agents:` 数组中显式列出所有 agent 路径。**预期收益**：安装后 Claude 能自动感知这些 agents
3. **当前问题**：外部插件（discord/telegram 等）在 package.json 里写 `"start": "bun install --no-summary && bun server.ts"`，使用版本范围符 `^`，启动时拉取未固定依赖。**改进做法**：锁定到具体版本 + 添加 lockfile 校验。**预期收益**：消除供应链攻击面

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **96/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| 多个 plugin-dev agents | 80 | 关闭代码块 ``` 之后附有陈旧对话文本 |
| skill-creator agents | 低 | 缺 YAML frontmatter，无法注册 |
| feature-dev agents | 较低 | 零示例（no examples） |
| hookify/hooks.json | 低 | PreToolUse 无 matcher，触发所有工具调用 |

### 3.2 当时值得借鉴的模式

1. **单职责插件设计** → 每个插件只解决一个问题，强制分层 → feature-dev.md 只写特性开发流程 → 自己的 skill 也应避免"大而全"
2. **多语言 LSP 统一接入** → 同一插件接口规范覆盖 10+ 语言 → 各 lsp 插件结构一致 → 说明良好的规范可以横向扩展而不需要改设计
3. **外部集成物理隔离** → 官方/社区插件目录级别隔离 → external_plugins/ 与 plugins/ 分开 → 信任层级通过目录结构表达，无需文档说明

### 3.3 当时的缺陷

1. **Agent 缺 YAML frontmatter** → skill-creator 的三个 agents 以 `# Title` 开头而无 `---`。Claude Code 读取 agent 时需要解析 frontmatter 才知道 name/model/tools；没有 frontmatter 意味着 agent 无法正确注册，用户安装后调用这些 agents 会失败。**自查**：检查自己的 agents 是否都有完整 `---name/description/model---` 块
2. **外部 MCP 服务 bun install 版本未锁定** → 4 个外部 MCP 服务在启动脚本中 `bun install` 拉取未固定依赖（`^`范围符）。这意味着每次用户启动 MCP 服务都可能拉取不同版本。**根本原因**：方便性优于安全性，没把供应链安全当一等公民。**自查**：我的 MCP 服务有没有用范围版本而非固定版本？
3. **hookify 全工具触发** → hooks.json 没有 matcher，PreToolUse 会在每一次工具调用时触发，性能开销大且增加误触发风险。**根本原因**：hook 框架早期实现，没有细化触发条件。**自查**：我写的 hook 是否有精确的 matcher 限制触发范围？

### 3.4 当时的优化机会

1. feature-dev agents 需要至少 2 个 input→output 示例，否则 Claude 难以理解使用场景
2. plugin-dev agents 结束代码块后有陈旧对话文本，应清理
3. skill-creator plugin.json 应显式声明所包含的 agents 路径

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| skill-creator agents 缺 frontmatter | `head -5 plugins/skill-creator/skills/skill-creator/agents/*.md` | **仍存在**：analyzer/comparator/grader 均以 `# Title` 开头，无 `---` YAML 块 | 9 个月过去，核心 bug 未修 |
| type-design-analyzer color:pink 无效值 | `find . -name "type-design-analyzer.md"` | **已消失**：整个文件从仓库中移除 | 通过删除消除 bug，非修复 |
| feature-dev agents 缺 frontmatter/零示例 | `head -10 plugins/feature-dev/agents/*.md` | **已修复**：code-architect、code-explorer、code-reviewer 现均有完整 `---name/tools/model/color---` frontmatter | 主动修复，且新增了丰富的 agent 定义 |
| bun install 无版本锁定 | `grep "bun install" external_plugins/*/package.json` | **仍存在**：discord/telegram/imessage/fakechat 的 package.json 均有 `bun install --no-summary && bun server.ts` | Medium 安全风险持续 |

### 4.2 架构演进

从审计到现在（约 3 个月），仓库变化显著：
- **新增**：code-modernization（7 agents 的大型工作流）、ralph-loop、agent-sdk-dev（含 2 个 verifier agents）、security-guidance、learning-output-style/explanatory-output-style、mcp-tunnels、claude-md-management、session-report 等 10+ 新插件
- **重组**：skill-creator 从扁平结构变为 `skills/skill-creator/` 嵌套结构，agents 移到其中
- **新增 external_plugins**：从 4 个扩展到 15 个（新增 asana、context7、firebase、gitlab、linear、playwright、serena、greptile 等）
- **意味着**：仓库正在从"少数几个工作流插件"向"完整插件生态"演进，速度很快；与此同时，audit bug 修复速度远落后于新功能添加速度

### 4.3 新增的可学习模式

1. **hookify 2.0 — 用户可配置 hooks**：hookify 现在不是硬编码 hook 逻辑，而是通过 pretooluse.py/posttooluse.py 读取用户 `.local.md` 文件动态配置。这是"把 hook 框架本身做成可配置插件"的优雅设计
2. **code-modernization 多 agent 流水线**：7 个专职 agents 各负责一段遗留代码现代化流程（legacy-analyst → architecture-critic → scaffolder → test-engineer 等），比单 agent 拆解更清晰
3. **外部插件目录标准化**：15 个 external_plugins 结构一致（package.json + .claude-plugin/plugin.json），说明社区 MCP 集成逐渐走向规范化

---

## 五、校准

### 5.1 我已经在做对的

1. **插件单职责**：echo-sleuth-for-claude 专注"挖掘过去对话"，bureau 专注"知识库管理"，每个仓库解决单一问题——与 anthropics 官方插件理念一致
2. **Agent 有 name/description frontmatter**：echo-sleuth 的 agents（analyze/recall 等）均有完整 frontmatter，比 skill-creator 的三个 agents 做得更好
3. **示例驱动的 agent description**：echo-sleuth 的 recall.md 在 description 中提供了多个触发场景和例子，与官方最佳实践对齐

### 5.2 挑战 / 验证

本案例挑战了我的这个假设：**"Anthropic 官方仓库应该是最规范、零 bug 的"**。

事实是：审计时 skill-creator 的三个核心 agents 没有 YAML frontmatter，这意味着这些 agents 在用户安装后无法正常工作。9 个月过去这个 bug 还在。这说明：

1. **规模即盲区**：仓库规模越大，越容易有角落里的 bug 没人注意
2. **新功能 > bug 修复**：团队在快速添加插件（3 个月 +10 个插件），但基础 bug 没有被同等优先级处理
3. **自我教训**：即使是官方仓库，都需要自动化 manifest 校验（`validate-frontmatter.yml` 存在，但显然没覆盖 agent frontmatter 的必填字段检查）

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agents 是否都有 frontmatter
for f in ~/.claude/skills/*/agents/*.md; do
  python3 -c "
import sys
content = open('$f').read()
if not content.startswith('---'):
    print('MISSING FRONTMATTER:', '$f')
" 2>/dev/null
done

# 检查我的 plugin.json 是否声明了所有 agents
python3 -c "
import json, os, glob
for pj in glob.glob('**/.claude-plugin/plugin.json', recursive=True):
    d = json.load(open(pj))
    declared = set(d.get('agents', []))
    disk = set(glob.glob(os.path.join(os.path.dirname(pj), '../agents/*.md')))
    if disk - declared:
        print(f'{pj}: agents on disk not in manifest: {disk - declared}')
"
```
命中后：为每个 agent 文件添加 `---name/description/model/tools---` 块，并将路径添加到 plugin.json 的 agents 数组。

```bash
# 检查外部 MCP 服务的 bun/npm 版本是否锁定
grep -rn '"start"' external_plugins/*/package.json | grep -E '(install|bun|npm)' | grep -v 'lockfile'
```
命中后：将版本范围符 `^` 替换为精确版本，添加 bun.lock 或 package-lock.json 并提交到仓库。

### 6.2 灵感 → 实施路径

1. **想法**：为 bureau 添加 hookify 风格的"用户可配置 hook"——用户在 `.local.md` 里声明想在哪些 tool 后触发什么动作
   - **为何可行**：bureau 已有 hooks/ 目录；这个设计不增加 bureau 自身复杂度，而是把配置权还给用户
   - **第一步**：参考 hookify 的 pretooluse.py，在 bureau/hooks/ 下创建 user-config-hook.py，10 分钟可搭出框架

2. **想法**：在 CI 中添加 agent frontmatter 校验（检查所有 agents/*.md 是否有合法 YAML frontmatter）
   - **为何可行**：anthropics 官方都翻车了，手动检查不可靠
   - **第一步**：在 .github/workflows/ 里新建 validate-nl-artifacts.yml，用 Python 解析所有 agent/skill 文件的 frontmatter，2小时可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 anthropics/claude-plugins-official 的核心目的**：提供高质量、官方背书的 Claude Code 插件参考实现集合
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 均为 Claude Code 插件，有 plugin.json + skills + agents | bureau 是单插件，anthropics 是多插件目录 | 高 |
| MarkQWu/echo-sleuth-for-claude | 高 | 均有 commands/agents/skills 三层结构 | echo-sleuth 单一功能，anthropics 是生态级 | 高 |
| MarkQWu/gstack | 低 | 均有 agents | gstack 不是 Claude Code 插件，agents 是 YAML 格式（OpenAI风格） | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 未声明 agents | `python3 -c "import json; d=json.load(open('.claude-plugin/plugin.json')); print(d.get('agents', 'MISSING'))"` | bureau 和 echo-sleuth 的 plugin.json 均只有元数据（name/version/description），无 skills/agents/commands 数组 | **高** |
| agent 缺 YAML frontmatter | `head -3 agents/*.md` | echo-sleuth 的 agents 均有完整 frontmatter — 无命中 | 无 |

**命中后的具体行动**（bureau 和 echo-sleuth 均受影响）：
- `MarkQWu/bureau/.claude-plugin/plugin.json` → 添加 `"skills": ["./skills/capture", "./skills/compile", ...]` 和 `"agents": ["./agents/auditor.md"]` 数组 → 约 10 分钟
- `MarkQWu/echo-sleuth-for-claude/.claude-plugin/plugin.json` → 添加 `"skills": ["./skills/experience-synthesis", ...]` 和 `"agents": ["./agents/analyze.md", ...]` → 约 15 分钟

### 8.3 别人的更优方案

1. **领域**：多 agent 工作流设计
   - **本案例做法**：feature-dev 将探索/架构/审查三个职责分给三个专职 agent，每个 agent 只有 Read 类工具，互不越权（`plugins/feature-dev/agents/code-architect.md`）
   - **我的项目现状**：echo-sleuth 的 analyze.md 一个 agent 做了所有分析，职责边界模糊
   - **如何借鉴**：把 echo-sleuth 的 analyze 拆为 `pattern-extractor.md`（提取模式）和 `insight-synthesizer.md`（生成洞察）两个 agent，各自只声明需要的工具

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：examples 质量
- **我的做法**：echo-sleuth 的 recall.md 在 description 里提供了 10+ 个触发句式示例和具体 input/output 对（`MarkQWu/echo-sleuth-for-claude/agents/recall.md`）
- **本案例做法（弱在哪）**：feature-dev agents 在审计时零示例（0 examples），即使现在 frontmatter 已修复，示例密度仍不及 echo-sleuth
- **意义**：这是 echo-sleuth 的一个亮点；如果考虑贡献，可以给 feature-dev agents 提 PR 补充示例

---

## 八、术语表

### <a name="插件"></a>插件（Claude Code Plugin）
> Claude Code 的"扩展包"，通过一个 `plugin.json` 声明包含哪些技能（skills）、代理（agents）、命令（commands）和钩子（hooks）。用户运行 `claude plugin install <name>` 后，Claude Code 会读取这个清单文件并加载其中声明的所有组件。类比：就像浏览器扩展，你安装后获得新功能，但不同扩展互不干扰。

### <a name="LSP"></a>LSP（Language Server Protocol）
> 一种协议标准，让代码编辑器（VS Code、JetBrains 等）和语言服务器（提供代码补全、跳转定义、错误检查等能力）之间通信。Claude Code 的 LSP 插件让 AI 也能"懂"这个协议，从而在处理代码时获取语言级别的精准信息（如"这个函数定义在哪里"）。

### <a name="NL-artifact"></a>NL artifact（自然语言制品）
> 用自然语言（而非代码）编写的 AI 指令文件。包括 SKILL.md（技能描述）、agent .md（代理定义）、command .md（命令文件）等。这些文件告诉 Claude 该做什么、怎么做，就像给新员工写的操作手册。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 YAML 配置段，用来声明文件的元数据。对 agent 文件来说，frontmatter 里必须有 `name`（名字）、`description`（描述）、`model`（使用哪个模型）等字段，Claude Code 读取这些信息才能注册和调用 agent。没有 frontmatter，文件相当于"匿名"，系统不知道怎么使用它。

### <a name="monorepo"></a>monorepo
> 把多个独立项目放在同一个 git 仓库里管理的策略。本案例的 `plugins/` 目录就是 monorepo 风格——37 个插件在同一仓库，但各自独立可安装。对比"多仓库"方案（每个插件单独开一个 repo）的优势是：统一的 CI/CD 流程、跨插件共享测试基础设施、更方便统一格式审查。

### <a name="manifest"></a>manifest（清单文件）
> 项目的"成分表"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载，就像超市卖的没贴标签的商品——不会出现在收银台扫描结果里。
