# AgriciDaniel/claude-obsidian — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-obsidian
**Stars**：976 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-17（基于当前 HEAD）
**主题标签**：security-gate, vague-quantifier, manifest-discipline, experience-accumulation

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-obsidian 是一个将 Claude Code 插件与 Obsidian 知识库管理深度融合的混合型仓库，当前 HEAD（75d3b6feb77b96c6bb16599c4550cc9703553d87）共包含 19 个自然语言制品：4 个命令、9 个技能、2 个代理，以及 `hooks.json`、`plugin.json`、`CLAUDE.md`。该插件以 976 颗星成为同类项目中关注度较高的实践，核心价值在于让 Claude Code 直接读写 Obsidian vault 并维护自动生长的知识图谱。审查后新增文件包括 `skills/wiki-fold/SKILL.md`、`GEMINI.md`、`WIKI.md`、`AGENTS.md`、`wiki/canvases/` 目录、`.vault-meta/` 目录以及 `bin/` 脚本集。

### 1.2 架构剖析

项目采用"Claude Code 插件 + Obsidian Vault 同目录"的混合布局，目录职责如下：

- **`.raw/`**：不可变原始资料存放目录，由 Claude 摄取（ingest）但不直接修改。
- **`wiki/`**：Claude 生成的知识库目录，是插件写出结构的主要目标位置。
- **`wiki/hot.md`**：会话缓存文件，SessionStart 钩子通过 `cat wiki/hot.md` 将其内容注入上下文。
- **`wiki/canvases/`**（审查后新增）：存放 Obsidian Canvas 格式的可视化笔记。
- **`.vault-meta/`**（审查后新增）：vault 元数据目录。
- **`bin/`**（审查后新增）：辅助脚本集合。

技能层由 9 个技能组成，形成清晰的职责划分：
- `wiki/SKILL.md`：顶层编排技能，统一调度其余技能。
- `wiki-ingest/SKILL.md`：将 `.raw/` 中的原始资料摄取进知识库。
- `wiki-query/SKILL.md`：查询知识库，**声明** `allowed-tools: Read Glob Grep`，但实际在第 44、56、153 行执行写操作。
- `wiki-lint/SKILL.md`：对知识库进行质量检查与交叉引用维护。
- `autoresearch/SKILL.md`：多轮深度研究流程。
- `save/SKILL.md`：将对话内容持久化到 vault。
- `canvas/SKILL.md`：生成 Obsidian Canvas 可视化。
- `defuddle/SKILL.md`：网页内容提炼。
- `obsidian-markdown/SKILL.md` 与 `obsidian-bases/SKILL.md`：Obsidian 格式规范参考。

代理层包含 2 个代理：`wiki-lint.md`（负责 lint 流程编排）和另一名代理，均通过 `hooks.json` 中的生命周期钩子与 vault 联动。

### 1.3 设计思路 / 方法论

- **Vault 即数据库**：将 Obsidian vault 作为 Claude 的持久化存储后端，通过 `.raw/`（只读源）→ `wiki/`（结构化输出）的单向数据流维护信息可信度。
- **热缓存注入**：`wiki/hot.md` 充当会话级上下文缓存，通过 SessionStart 钩子在每次对话开始时自动注入，减少重复检索开销。
- **钩子驱动自动提交**：Stop 钩子通过检测 `wiki/` 目录变化，条件性地输出提交指令，尝试将知识库变更自动纳入 git 历史。
- **技能编排树**：顶层 `wiki/SKILL.md` 统一调度下层技能，符合单入口、多出口的编排模式。
- **生态扩展**：审查后新增 `GEMINI.md` 和 `AGENTS.md`，说明项目正在向多模型、多代理环境演进。

---

## 二、过去审查发现（2026-04-20 历史快照）

### 2.1 当时质量评分（NLPM）

**综合评分：91 / 100**（安全状态：**BLOCKED** — 2 Critical, 1 High, 4 Medium）

| 文件 | 得分 | 主要扣分项 |
|------|------|-----------|
| commands/autoresearch.md | 60 | 缺少 `name` 字段（-25）、无 `allowed-tools`（-5）、多步骤流程未编号（-10） |
| commands/save.md | 60 | 缺少 `name` 字段（-25）、无 `allowed-tools`（-5）、多步骤流程未编号（-10） |
| commands/canvas.md | 70 | 缺少 `name` 字段（-25）、无 `allowed-tools`（-5） |
| commands/wiki.md | 70 | 缺少 `name` 字段（-25）、无 `allowed-tools`（-5） |
| skills/wiki-query/SKILL.md | 85 | `Write` 未列入 `allowed-tools`，但技能在第 44、56、153 行执行写操作（-15） |
| agents/wiki-lint.md | 97 | 声明 Bash 工具但从未使用（-3） |
| skills/autoresearch/SKILL.md | 98 | 模糊量词："major contradictions"（-2） |
| skills/save/SKILL.md | 98 | 模糊量词："most valuable content"（-2） |
| skills/wiki-lint/SKILL.md | 98 | 模糊量词："significant cross-references"（-2） |
| skills/wiki/SKILL.md | 98 | 模糊量词："significant query exchange"（-2） |
| skills/wiki-ingest/SKILL.md | 96 | 模糊量词："significant ideas"、"relevant domain"（-4） |
| hooks/hooks.json | 90 | 自动提交 `.raw/` 目录（-5）、Stop 钩子通过 `echo` 注入多句 LLM 指令（-5） |

### 2.2 当时值得借鉴的模式

- **单向数据流设计**：`.raw/`（只读）→ `wiki/`（写出）的分层存储模式，清晰区分原始资料与加工结果，防止原始资料被意外覆写。
- **热缓存机制**：`wiki/hot.md` 作为会话级索引缓存，通过 SessionStart 钩子自动注入，在大型知识库场景下显著降低检索延迟。
- **技能编排树**：`wiki/SKILL.md` 作为唯一编排入口，下游技能各司其职，体现了良好的模块化思维。
- **整体得分保持高位（91/100）**：在混合型仓库（插件 + vault）的复杂约束下，技能层平均分接近 97，体现出作者对 NL 制品质量规范的基础掌握。

### 2.3 当时的缺陷

1. **4 个命令全部缺少 `name` 字段**：`commands/autoresearch.md`、`commands/save.md`、`commands/canvas.md`、`commands/wiki.md` 的前置元信息均未声明 `name:`，导致插件清单无法正确索引命令，每命令扣 25 分。
2. **4 个命令全部缺少 `allowed-tools` 声明**：工具权限不透明，用户无法预判命令会调用哪些工具，存在意外权限扩散风险。
3. **`commands/autoresearch.md` 和 `commands/save.md` 多步骤流程未编号**：无序步骤使 Claude 难以按预期顺序执行，降低流程可重现性。
4. **`skills/wiki-query/SKILL.md` 实际执行写操作但 `allowed-tools` 未包含 `Write`**：声明与行为不一致，第 44、56、153 行均调用写操作，而 `allowed-tools: Read Glob Grep` 中无 `Write`。
5. **6 处模糊量词分散在 5 个技能文件**：`major contradictions`、`most valuable content`、`significant cross-references`、`significant query exchange`、`significant ideas`、`relevant domain` 等表达缺乏可测量标准。

### 2.4 当时的优化机会

- 为全部 4 个命令的前置元信息补充 `name:` 字段，使插件清单可正确索引。
- 为全部 4 个命令添加 `allowed-tools` 声明，列出实际使用的工具集合。
- 将 `commands/autoresearch.md` 和 `commands/save.md` 中的无序步骤替换为编号列表（1. 2. 3. …）。
- 在 `skills/wiki-query/SKILL.md` 的 `allowed-tools` 中补充 `Write`，或重构技能以避免写操作。
- 将 6 处模糊量词替换为具体的可测量标准，例如以条数、字数或阈值表达代替形容词。
- 修复 `hooks.json` 的安全问题（见安全审查）。

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 缺陷 | 审查时状态 | 当前 HEAD 状态 |
|------|-----------|--------------|
| 命令缺少 `name` 字段（4 个） | 全部缺失 | **仍未修复**：`grep -rL "^name:" commands/` 返回全部 4 个文件 |
| 命令缺少 `allowed-tools`（4 个） | 全部缺失 | **仍未修复** |
| `autoresearch.md` / `save.md` 步骤未编号 | 存在 | **仍未修复** |
| `wiki-query` `Write` 未在 `allowed-tools` 中 | `allowed-tools: Read Glob Grep` | **仍未修复**：当前 `allowed-tools` 仍为 `"Read Glob Grep"`，无 `Write` |
| 6 处模糊量词 | 6 处 | **仍未修复**：`grep -rn "major contradictions\|most valuable\|significant cross-references\|significant query exchange\|significant ideas\|relevant domain" skills/` 仍返回 6 条 |
| Stop 钩子提示注入 | `hooks.json` 第 46 行，直接 `echo` 多句 LLM 指令 | **部分改善**：Stop 钩子现改为 `git diff --name-only HEAD \| grep -q '^wiki/'` 后条件性 `echo`，攻击面缩小但条件性提示注入漏洞仍存在 |
| SessionStart `cat wiki/hot.md` 间接注入 | `hooks.json` 第 9 行 | **仍未修复**：`cat wiki/hot.md` 内容仍无校验地注入上下文 |
| `agents/wiki-lint.md` 声明 Bash 但未使用 | 存在 | 未见修复 |

### 3.2 架构演进

审查后仓库在原有技能树基础上进行了有意义的扩展：

- **新增 `skills/wiki-fold/SKILL.md`**：功能尚不明确，推测为知识折叠/摘要技能，扩展了知识库的层次管理能力。
- **新增 `wiki/canvases/` 目录**：支持 Obsidian Canvas 可视化笔记，与 `canvas/SKILL.md` 形成完整闭环。
- **新增 `.vault-meta/` 目录**：vault 元数据独立存放，体现对 vault 状态跟踪的工程化思考。
- **新增 `bin/` 脚本集**：辅助脚本的引入表明项目正向工程化工具链演进。
- **新增 `GEMINI.md` 和 `AGENTS.md`**：向多模型、多代理环境的显式扩展，说明作者在探索跨模型知识库工作流。
- **Stop 钩子条件化**：将原来无条件的 `echo` 指令注入改为基于 `git diff` 的条件触发，减少了不必要的提示注入频率，但未从根本上消除漏洞。

### 3.3 新增的可学习模式

- **Canvas 技能与 vault 目录协同**：`skills/canvas/SKILL.md` 与 `wiki/canvases/` 目录形成"技能定义写出位置"的配对模式，是技能与存储约定显式绑定的良好示例。
- **`.vault-meta/` 元数据分离**：将 vault 运行时状态与用户笔记内容分离，避免元数据污染知识库，是混合型仓库中值得借鉴的目录组织策略。
- **`GEMINI.md` 多模型配置**：为不同模型提供独立的配置文件，体现模型配置与内容配置解耦的工程意识。

---

## 四、校准

### 4.1 我已经在做对的

- **单向数据流（只读源 → 写出目标）**是防止原始资料被覆写的可靠模式，适用于任何需要区分"输入区"与"输出区"的知识管理插件。
- **顶层编排技能统一入口**（`wiki/SKILL.md` 调度下层技能）是管理多技能复杂度的成熟方式，在技能数量超过 5 个时尤为必要。
- **会话级热缓存注入**（SessionStart 钩子读取索引文件）是减少大型知识库检索延迟的实用技巧，但需要配合内容校验以防止注入风险。

### 4.2 挑战 / 验证

claude-obsidian 案例提出了两个值得深入思考的问题：

**挑战一：声明与行为不一致的隐蔽代价**

`skills/wiki-query/SKILL.md` 声明 `allowed-tools: Read Glob Grep`，但第 44、56、153 行实际执行写操作。这不仅是安全评分扣分项，更会在运行时触发 Claude Code 的权限校验失败。更危险的是，这种不一致在代码审查中极难发现——读者看到 `allowed-tools` 声明后往往不再逐行检查实际调用。这一问题历经审查后 27 天仍未修复，说明作者可能并不清楚其存在。**验证点**：在自己项目的每个技能文件中，`allowed-tools` 是否与实际调用的工具集合完全吻合？是否有自动化手段（如 grep 比对）检验这一一致性？

**挑战二：安全漏洞与功能设计的权衡**

Stop 钩子的提示注入漏洞（`hooks.json` 第 46 行）和 SessionStart 的间接注入漏洞（第 9 行）是功能设计与安全边界的直接冲突：`wiki/hot.md` 的自动注入是该插件的核心价值之一，但若 hot.md 被污染，所有会话均会受影响。作者在审查后将 Stop 钩子改为条件触发，表明已意识到问题，但未彻底修复。**验证点**：自己项目中的钩子是否有外部内容（文件、网络）注入上下文？注入前是否有格式校验或长度限制作为安全边界？

---

## 五、行动

### 5.1 自查动作

以下命令可在任何含 NL 制品的项目中直接运行：

```bash
# 检查所有命令是否声明 name 字段
grep -rL "^name:" commands/ && echo "以上命令文件缺少 name 字段"

# 检查所有命令是否声明 allowed-tools
grep -rL "allowed-tools:" commands/ && echo "以上命令文件缺少 allowed-tools 声明"

# 检查技能文件中 allowed-tools 声明与实际 Write 调用是否一致
for f in $(grep -rL "Write" skills/*/SKILL.md 2>/dev/null); do
  if grep -q "^\s*Write\b" "$f" 2>/dev/null; then
    echo "⚠️  $f: 实际调用 Write 但未在 allowed-tools 中声明"
  fi
done

# 统计模糊量词出现次数
grep -rn "major\|significant\|most valuable\|relevant domain\|appropriate\|suitable" skills/ | wc -l

# 检查钩子文件中是否有 echo 注入 LLM 指令
grep -n "echo" hooks/hooks.json

# 检查 SessionStart 是否存在不校验的文件内容注入
grep -n "cat " hooks/hooks.json

# 检查多步骤命令是否使用了编号列表
grep -L "^[0-9]\+\." commands/*.md 2>/dev/null && echo "以上命令可能包含未编号的多步骤流程"
```

### 5.2 灵感 → 实施路径

**灵感一：补全命令前置元信息 `name` 字段 → 今日可改**

为 `commands/autoresearch.md`、`commands/save.md`、`commands/canvas.md`、`commands/wiki.md` 的前置元信息添加 `name:` 字段。格式参考：

```yaml
---
name: autoresearch
description: 对给定主题执行多轮深度研究并写入知识库
allowed-tools: Read, Write, Glob, Grep, WebSearch
---
```

`name` 字段是 Claude Code 命令清单索引的必要字段，缺失时插件无法被 `/nlpm:ls` 正确发现，且每命令扣 25 分是所有单项扣分中最高的。

**灵感二：修复 `wiki-query` allowed-tools 声明 → 本周可完成**

选择以下两种方案之一：

- **方案 A（推荐）**：在 `skills/wiki-query/SKILL.md` 的 `allowed-tools` 中补充 `Write`，将声明更新为 `allowed-tools: Read Glob Grep Write`，使声明与第 44、56、153 行的实际行为一致。
- **方案 B**：重构技能，将写操作从 `wiki-query` 中剥离，移入专职的 `wiki-write` 技能，保持 `wiki-query` 纯读的语义。

方案 B 更符合单一职责原则，但需要更新 `wiki/SKILL.md` 中的调度逻辑。

**灵感三：将模糊量词替换为可测量标准 → 本周可完成**

参考以下替换示例，将 6 处模糊量词逐一修订：

| 原文（模糊） | 建议替换（具体） |
|-------------|----------------|
| `major contradictions` | `与已有条目直接矛盾（同一事实，不同结论）的矛盾` |
| `most valuable content` | `包含独立论点或具体数据的段落（排除过渡句和引言）` |
| `significant cross-references` | `被 3 个以上其他条目引用的节点` |
| `significant query exchange` | `包含 5 轮以上有效问答的对话片段` |
| `significant ideas` | `原文中包含独立论点、数据或案例的概念单元` |
| `relevant domain` | `与当前 vault 主题标签（tags: 字段）匹配的领域` |

**灵感四：修复钩子安全漏洞 → 优先级最高**

针对 `hooks/hooks.json` 的两处安全问题：

1. **Stop 钩子（第 46 行）条件性提示注入**：将 `echo` 输出的 LLM 指令改为从固定本地文件读取（而非动态生成），并限制输出长度（如最多 200 字符），防止外部攻击者通过构造 `wiki/` 目录下的文件内容劫持提示。

2. **SessionStart 钩子（第 9 行）`cat wiki/hot.md` 间接注入**：在注入前添加内容校验步骤：
   ```bash
   # 限制 hot.md 注入长度，防止超大文件淹没上下文
   head -n 50 wiki/hot.md
   # 或添加格式校验，仅注入以 # 或 - 开头的行
   grep "^[#\-]" wiki/hot.md | head -n 30
   ```
   同时在 `wiki/hot.md` 的写入端（更新该文件的技能）添加内容清洗步骤，过滤可能的注入载荷（如包含 `SYSTEM:` 或 `IGNORE PREVIOUS` 的行）。

**灵感五：单向数据流模式 → 立即可复用**

claude-obsidian 的 `.raw/`（只读源）→ `wiki/`（写出目标）分层模式可直接迁移到任何知识管理场景。在自己的项目中，若技能同时读写同一目录，可按此模式重构：

1. 创建 `sources/` 目录存放原始输入（文件、网页抓取、API 响应），设为只读约定。
2. 创建 `output/` 或 `kb/` 目录作为 Claude 的唯一写出目标。
3. 在 `CLAUDE.md` 中明确声明两个目录的读写约定，防止技能误写原始资料。
4. 在 `allowed-tools` 中通过工具集差异强化约束：读取技能仅声明 `Read Glob Grep`，写出技能显式声明 `Write`。
