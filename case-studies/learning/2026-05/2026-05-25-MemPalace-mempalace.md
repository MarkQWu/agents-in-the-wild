# MemPalace/mempalace — 学习案例

**仓库**：https://github.com/MemPalace/mempalace
**Stars**：51,982 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-13（历史快照）| **生成日期**：2026-05-25（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `experience-accumulation`, `security-gate`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
MemPalace 是一个本地优先的 AI 对话记忆系统，核心承诺是「100% 完全召回」——存储用户每一句原话，绝不摘要，绝不改写，随时可检索。技术上它是一个 MCP 服务器（19 个工具），配合 Claude Code 插件（hooks + commands + skill）在每次会话结束时自动归档对话。

关键事实：
- 51,982 stars，属于 AI 工具生态明星项目
- 作者 milla-jovovich，设计哲学直接对标古代「记忆宫殿（Method of Loci）」和 Zettelkasten 卡片盒
- 用户通过 `uv tool install mempalace` 或 `pip install mempalace` 安装，随即通过 [Claude Code 插件](#claude-code-插件)获取 AI 交互界面
- 在生态中占据「AI 跨会话记忆」这一罕见定位——绝大多数 AI agent 框架没有解决这个问题

### 1.2 架构剖析
- **目录结构**：
  ```
  mempalace/           ← Python 核心（PyPI 包）
  .claude-plugin/      ← Claude Code 插件接入层
    commands/          ← 5 个命令（help/init/mine/search/status）
    skills/mempalace/  ← 单体 SKILL.md
    hooks/             ← mempal-stop-hook.sh + mempal-precompact-hook.sh
    plugin.json        ← manifest（commands: [] 注意！）
    .mcp.json          ← MCP 服务器声明
  .codex-plugin/       ← Codex 接入层
    skills/            ← 5 个按命令拆分的独立 SKILL.md
  integrations/openclaw/  ← OpenClaw 适配
  hooks/               ← 根目录 hooks（原始版本，带安全加固）
  website/             ← 文档网站
  ```
- **文件类型分布**：1 个单体 SKILL.md（claude-plugin）/ 5 个独立 SKILL.md（codex-plugin）/ 5 个 command / 1 个 hook-config / 5 个 hook 脚本
- **编排关系**：用户触发 `/mempalace:mine` → command 调用 mempalace CLI → CLI 调用 MCP 工具 → hook 在会话结束时自动触发记忆归档。claude-plugin 侧是「1个skill覆盖全部功能」，codex-plugin 侧是「每命令1个skill」。两套并列，分别服务不同的 LLM 平台
- **跨件契约**：SKILL.md 里的用法指引指向 `mempalace instructions <command>` CLI 动态获取操作说明——skill 本身不硬编码具体步骤，而是委托给 CLI，这是一种「动态文档」模式

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「原生二进制核心 + [NL 表皮](#nl-表皮)」——Python [原生二进制核心](#原生二进制核心)做真正的存储和检索工作，NL 文件（SKILL.md、command）只做人机接口
- **解决什么问题**：AI 无跨会话记忆，每次对话都是白板。用户在一次会话里说的事情，下次对话 AI 完全不记得
- **做了什么 trade-off**：
  - 选择「绝对不摘要」而非「智能摘要」——牺牲存储效率，换取零信息损失
  - 选择本地优先而非云同步——牺牲跨设备方便，换取隐私保证
  - 选择双平台插件（claude-plugin + codex-plugin）而非只支持一个——增加维护负担，换取更广覆盖
- **反映什么认知模型**：作者认为 AI 最大的能力差距不是「生成」，而是「记忆」。技术路线是把 LLM 的推理能力与文件系统的持久性组合——用 LLM 理解对话，用文件系统保存原文

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「NL 表皮 + 原生二进制核心」的双平台适配架构**

核心逻辑是：把「做事情的代码」全部放进一个普通的 Python 包里，SKILL.md/command 只做两件事：(1) 告诉 AI 什么时候用它，(2) 告诉 AI 怎么调用 CLI。这样 NL 层极薄，核心逻辑完全在 Python 里，可独立测试、独立发布、独立升级。

模式特征清单：
- **特征 1**：AI 接口（SKILL.md）不包含业务逻辑，只包含触发条件和 CLI 调用指令
- **特征 2**：不同 LLM 平台（Claude Code、Codex）用不同的适配层，共享同一个 Python 核心
- **特征 3**：版本化在 PyPI，不依赖 Claude Code 插件版本——用户 `uv tool install mempalace` 升级和 `claude plugin install` 升级是解耦的
- **特征 4**：动态文档——SKILL.md 调用 `mempalace instructions <cmd>` 获取最新操作说明，避免 NL 文件与 CLI 文档漂移
- **特征 5**：hook 做背景归档，不占用对话 token

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要系统级权限或本地文件操作的工具 | ✅ 高度适用 | 原生二进制处理文件最自然，NL 层只是调用者 |
| 需要跨多个 LLM 平台发布的插件 | ✅ 适用 | 核心逻辑一份，adapter 多份 |
| 纯 AI 提示工程的内容生成类工具 | ❌ 不适用 | 没有「重逻辑」需要放进二进制，全 NL 就够了 |
| 需要频繁迭代 AI 提示的早期实验 | ⚠️ 谨慎 | 双层结构增加调试复杂度，早期可以先用纯 NL |
| 团队共享插件、有版本管理需求 | ✅ 适用 | PyPI 提供了天然的版本管理和分发机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL表皮+原生二进制核心（本仓库） | MemPalace | 核心可独立测试、发布；AI层极薄；跨LLM适配容易 | 维护两套东西；版本同步是负担 |
| 纯NL单仓库 | drama-workshop-skills | 零门槛维护；所有逻辑对 AI 可见 | 复杂逻辑难用 NL 描述；无独立测试 |
| 单仓 Python + 单 SKILL.md | echo-sleuth-for-claude | 简单 | 不适合跨平台 |

### 2.4 改进空间
1. **当前问题**：`commands: []` 空数组，5 个 command 文件没有注册进 plugin.json。**改进做法**：把 `commands` 数组填上，或在 CLAUDE.md 里明确写明「Claude Code 依赖目录自动发现，不需要显式注册」。**预期收益**：消除歧义，让 NLPM 类工具不误报 BUG
2. **当前问题**：`integrations/openclaw/SKILL.md` 的 version 3.3.0 对比 plugin.json 的 3.3.6，每次发新版忘记同步。**改进做法**：在 pyproject.toml 的 `[tool.hatch.build.hooks]` 或 Makefile 里加一个版本同步脚本，发布前自动检查。**预期收益**：彻底消除版本漂移

---

## 三、过去审查发现（2026-05-13 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-13 当时得分 **90/100**，Security: CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude-plugin/commands/mine.md | 70 | 多步操作无编号步骤；无输出格式；无空输入处理 |
| .claude-plugin/commands/init.md | 80 | 多步操作无编号步骤；无输出格式 |
| .claude-plugin/commands/search.md | 80 | 无输出格式；无空输入处理 |
| .claude-plugin/plugin.json | 85 | commands:[] 但有5个command文件 |
| .claude-plugin/commands/help.md | 90 | 无输出格式 |
| .claude-plugin/commands/status.md | 90 | 无输出格式 |
| .claude-plugin/skills/mempalace/SKILL.md | 94 | 模糊词 "short"/"often"/"significant" |
| integrations/openclaw/SKILL.md | 94 | 版本 3.3.0 vs plugin.json 3.3.5（当时）；模糊量词 |
| CLAUDE.md | 94 | 模糊词 "real understanding"/"instant"/"vast amounts" |

### 3.2 当时值得借鉴的模式
1. **Hook 安全加固** → 根本原因：作者理解「hook 是系统边界」，是真正的外部输入入口，必须做防护 → 示例：根目录 `hooks/mempal_save_hook.sh` 用 mapfile + 单遍 Python 清洗，不用 eval，文件名用 sys.argv 传参不内插 → 如何借鉴：检查自己所有 hook 是否有类似防护

2. **动态文档委托** → 根本原因：避免 NL 层和 CLI 实现的「文档漂移」—— SKILL.md 调用 `mempalace instructions` 而不硬编码步骤 → 示例：`.claude-plugin/skills/mempalace/SKILL.md` 的 Usage 节 → 如何借鉴：对于有 CLI 的插件，SKILL.md 调用 CLI 的 `--help` 或 `instructions` 子命令来获取最新说明

3. **双平台适配但共享核心** → 根本原因：Claude Code 和 Codex 的 SKILL.md 规范略有不同，共享 Python 核心避免代码重复 → 示例：`.claude-plugin/` 里单体 SKILL vs `.codex-plugin/` 里 5 个细粒度 SKILL → 如何借鉴：设计插件时，先想清楚核心逻辑放哪，AI 层只是 adapter

4. **本地优先的架构承诺** → CLAUDE.md 里明确写「BYOK 永远不是默认，永远不是静默 fallback」 → 这是一个可检验、可执行的设计原则，不是空话

### 3.3 当时的缺陷
1. **`commands: []` 空 manifest** → 为什么会失败：如果 Claude Code 要求显式注册，这会导致 5 个 command 静默失效。根本原因可能是维护者觉得「Claude 会自动发现目录」但没在文档里说明 → 自查：我的 plugin.json 里有没有遗漏注册的文件？

2. **openclaw SKILL.md 版本漂移** → 版本 3.3.0（当时）而 plugin.json 3.3.5。根本原因：手动维护多处版本号，发版时容易漏一处 → 自查：我的插件有没有多处写版本号的情况？

3. **command 文件缺输出格式说明** → mine/search/init 都没有「输出是什么格式」的说明，用户和 AI 无法预知结果形态 → 为什么重要：AI 在组合工具时需要知道每个命令的输出，没有输出格式说明等于缺乏接口契约 → 自查：我的 command 有没有写输出格式？

### 3.4 当时的优化机会
1. commands 数组填入 5 个 command 的相对路径，或在 README/CLAUDE.md 里明确声明「依赖自动发现」
2. openclaw/SKILL.md 版本改为从 plugin.json 读取（或 CI 检查两者一致）
3. mine.md / search.md 加 `## Output` 节，说明命令输出什么格式、什么情况下为空

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `commands: []` 空 manifest | `cat .claude-plugin/plugin.json \| python3 -c "import json,sys; print(json.load(sys.stdin)['commands'])"` | **仍然存在**：命令行确认返回 `[]` | 维护者可能确实依赖目录自动发现，或一直没人触发 |
| openclaw 版本漂移 | `grep "version:" integrations/openclaw/SKILL.md` | **仍然存在**：openclaw 写 3.3.0，plugin.json 写 3.3.6（版本差距从 3.3.5→3.3.5 变为 3.3.0→3.3.6） | 漂移反而更大了，说明没有自动同步机制 |
| vague quantifiers in SKILL.md | `grep -n "short\|often\|significant" .claude-plugin/skills/mempalace/SKILL.md` | 需要实时核查（audit 后可能改动过）；结合 CLAUDE.md 中「instant」等词仍在 | 模糊词很难被机械修复，通常需要作者主动重写 |

### 4.2 架构演进
从 audit 时的文件清单到当前 HEAD，主要变化：
- plugin.json version 从 3.3.5 升到 3.3.6，说明有过一次发版
- 根目录 hooks 依然存在，说明不打算移除旧 hook 目录（兼容旧版安装的用户）
- `.codex-plugin/` 结构未变，说明双平台适配是稳定的长期策略

**作者后来意识到什么**：暂无明显大重组，项目处于平稳维护状态，核心架构决策已固化。

### 4.3 新增的可学习模式
当前 HEAD 中多了 `MISSION.md`、`CONTRIBUTING.md`、`SECURITY.md`、`AGENTS.md` 等文件，说明项目在向更正式的开源项目规范演进。`AGENTS.md` 是一个有趣的新增——专门写给 AI agent 的贡献指南，比 CLAUDE.md 更面向 agent 视角。

---

## 五、校准

### 5.1 我已经在做对的
1. **本地优先设计**：echo-sleuth-for-claude 同样是本地文件操作，不调用外部 API，这个方向是对的
2. **plugin.json 维护**：echo-sleuth 有 plugin.json，基础信息完整（name/version/description/author/license）
3. **单职责 skill 拆分**：echo-sleuth 里 4 个 skill 各有明确职责（memory-management / git-mining / experience-synthesis / jsonl-core）

### 5.2 挑战 / 验证
- **被挑战的假设**：我以为「SKILL.md 里应该写清楚所有操作步骤」。MemPalace 的做法正相反——SKILL.md 调用 `mempalace instructions` 动态获取，避免硬编码。这对有 CLI 核心的插件来说是更健壮的做法
- **被验证的做法**：「hook 是系统边界，要做输入校验」——MemPalace 的 hook 安全加固代码验证了这个判断

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 里 commands/skills 是否有遗漏注册
cat ~/.claude/plugins/*/plugin.json 2>/dev/null | python3 -c "
import json, sys
for line in sys.stdin:
    try:
        d = json.loads(line)
        print('commands:', d.get('commands', 'MISSING'))
    except: pass
"
```
命中后怎么办：对照文件系统确认是否需要填写，或在 README 里说明「依赖自动发现」。

```bash
# 检查 echo-sleuth 的 SKILL.md 是否缺输出格式
grep -rL "## Output\|output format\|Output:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
```
命中后怎么办：为每个命中的 SKILL.md 加 `## Output` 节，写明「输出什么、格式是什么、空时输出什么」。

```bash
# 检查版本号是否多处维护（漂移风险）
grep -rn "version:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude --include="*.md" --include="*.json" 2>/dev/null
```
命中后怎么办：如果有多处版本号，加一个发布前检查脚本确保一致。

### 6.2 灵感 → 实施路径
1. **想法**：在 echo-sleuth 的 SKILL.md 里用「调用 CLI 的 `--help`」代替硬编码操作步骤
   - **为何可行**：echo-sleuth 也有 Python 脚本，如果抽出 CLI 入口就可以做动态文档
   - **第一步**：给 echo-sleuth 的 Python 脚本加 `argparse` 入口，在 SKILL.md 里用 `python3 echo-sleuth.py --help` 获取说明（约 30 分钟）

2. **想法**：给 claude-for-legal 的 hook 加输入校验（path traversal 防护）
   - **为何可行**：claude-for-legal 有 scripts/ 目录，里面如果有 hook 脚本应该做防护
   - **第一步**：检查 `ls /tmp/my-repos/MarkQWu-claude-for-legal/scripts/`，如果有脚本，对照 MemPalace 的 mempal_save_hook.sh 加验证（约 20 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 MemPalace/mempalace 的核心目的**：给 AI 提供跨会话记忆，存储用户原话，随时可检索

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是"挖掘对话历史以积累经验/记忆" | MemPalace存原话+检索；echo-sleuth挖决策/错误/模式 | 高 |
| MarkQWu/claude-for-legal | 低 | 都是 Claude Code 插件 | 法律领域，无记忆/挖掘功能 | 低 |
| MarkQWu/drama-workshop-skills | 无 | 都有 SKILL.md | 短剧创作，功能完全不同 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺输出格式 | `grep -rL "## Output" skills/*/SKILL.md` | echo-sleuth 命中 3/4（memory-management/git-mining/jsonl-core 均无 ## Output） | 高 |
| 无 model 声明 | `grep -rn "^model:" skills/*/SKILL.md` | echo-sleuth 命中 4/4（全部无 model 字段） | 中 |
| 版本漂移风险 | 手动对比 plugin.json 与各 SKILL.md 里的 version | echo-sleuth plugin.json 写 0.4.0，skills 里写 0.2.0 (memory-management)——**漂移已发生** | 高 |

**命中后的具体行动建议**：
- `echo-sleuth/skills/memory-management/SKILL.md` → 在文件末尾加 `## Output` 节，描述成功时输出什么、失败时输出什么（5 分钟）
- `echo-sleuth/plugin.json` 与 skills 里的 version 不一致 → 统一到 0.4.0 并写发布脚本（15 分钟）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：动态文档委托（SKILL.md 调 CLI 而非硬编码）
   - **本案例做法**：`.claude-plugin/skills/mempalace/SKILL.md` 里写 `mempalace instructions <command>` 获取实时操作说明
   - **我的项目现状**：`echo-sleuth/skills/git-mining/SKILL.md` 直接在 SKILL.md 里写了所有操作步骤，当 Python 脚本改动时需要手动同步
   - **如何借鉴**：给 echo-sleuth 的 Python 脚本加 `--instructions` 子命令，在 SKILL.md 里调用它

2. **领域**：hook 安全加固（path validation + no eval）
   - **本案例做法**：`hooks/mempal_save_hook.sh` 用 mapfile 读 JSON，用 Python sys.argv 传路径，做 `..` 路径遍历检查
   - **我的项目现状**：echo-sleuth 暂无 hook，claude-for-legal 的 scripts 未经检查
   - **如何借鉴**：在下次写 hook 时，直接复制 MemPalace 的校验模式

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：plugin.json 的 `commands/skills` 显式注册
- **我的做法**：echo-sleuth 的 plugin.json 里明确列出了 keywords，但没有 commands/skills 字段（因为使用目录自动发现）——和 MemPalace 一样的做法
- 本案例未发现我的项目更优的维度（echo-sleuth 规模更小，缺陷更多）

---

## 八、术语表

### <a name="claude-code-插件"></a>Claude Code 插件
> Claude Code（Anthropic 的命令行工具）支持安装第三方「插件」来扩展功能。插件本质上是一组 Markdown 文件（commands、skills）+ 配置文件（plugin.json），告诉 Claude Code 这个插件有哪些命令和技能。用户通过 `claude plugin install` 安装，安装后就可以在对话里用 `/pluginname:command` 触发。

### <a name="原生二进制核心"></a>原生二进制核心
> 用 Python / Go / Rust 等编译型或打包型语言写出来、最终变成可独立运行的程序（如 CLI 工具）的代码。「原生」意思是不依赖 Claude Code 环境就能跑。MemPalace 的「原生二进制核心」是指 `mempalace` Python 包——它可以脱离 Claude Code 独立通过 `pip install mempalace` 安装并使用。

### <a name="nl-表皮"></a>NL 表皮
> Natural Language 表皮，即 SKILL.md、command.md 这些用自然语言写的文件。它们是 AI 看的「使用说明」，本身不执行任何代码，只告诉 AI「什么时候该调用什么工具、怎么调用」。

### <a name="manifest"></a>manifest
> 项目的「清单文件」——`plugin.json` 告诉 Claude Code 这个插件包含什么（commands、skills、MCP 服务器等）。如果 manifest 里遗漏了某个文件，那个文件可能无法被自动加载。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，用来声明文件的元数据（如 `name`、`description`、`version` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 来知道这个 skill 叫什么名字、什么时候触发。

### <a name="MCP服务器"></a>MCP 服务器
> Model Context Protocol 服务器。一个标准化协议，让 AI 能调用外部工具（函数）。MemPalace 通过 MCP 暴露 19 个内存操作工具给 Claude，Claude 调用这些工具完成「存记忆、取记忆、搜索记忆」等操作。
