# safishamsi/graphify — 学习案例

**仓库**：https://github.com/safishamsi/graphify
**Stars**：37,391 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-09（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `curl-pipe-bash-risk`, `vague-quantifier`, `template-design`
**xiaolai 案例**：[../../2026-05-03-safishamsi-graphify.md](../../2026-05-03-safishamsi-graphify.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

graphify 是由 Safi（safishamsi）维护的 Python CLI 工具，可以将任意文件夹（代码、SQL 模式、文档、图像、视频）转化为可查询的知识图谱。它同时作为 AI skill 分发给 15+ AI 工具使用：Claude Code、Codex、OpenCode、Cursor、Aider、GitHub Copilot CLI、Kiro、VSCode、Windows、Factory Droid、Trae、OpenClaw，以及 2026-04-29 审计后新增的 Amp、Devin、Kilo、Pi 等变体。通过 PyPI（`pip install graphify`）分发，Skill 文件位于 `graphify/skill*.md`，而非 Claude Code 规范路径 `skills/<name>/SKILL.md`——这是一个刻意的产品决策，但同时导致了 NLPM 自动发现失效。

关键数字（2026-04-29 审计快照 → 当前 HEAD）：

- 审计时 NLPM 综合评分：**58/100**（12 个工件）
- 安全评级：**REVIEW**（0 Critical，0 High，3 Medium，3 Low）
- Skill 变体数量：11 个（审计时）→ **15 个**（当前 HEAD，新增 skill-amp、skill-devin、skill-kilo、skill-pi）
- 审计后 33 小时内独立修复：**11/12 个发现**（25 次 commits，部分由 Claude Sonnet 4.6 / Opus 4.7 协作）
- xiaolai 流水线提交的 PR：**1 个**（PR #603，trae SyntaxError 修复，已合并）
- 当前 HEAD 3 个原始 Bug 状态：**全部已修复** ✓

### 1.2 架构剖析

**目录结构（Skill 库部分）**：

```
graphify/
  skill.md                  ← 主变体（Claude Code，基础参考版本）
  skill-aider.md            ← Aider 变体
  skill-amp.md              ← Amp 变体（审计后新增）
  skill-claw.md             ← OpenClaw 变体
  skill-codex.md            ← Codex 变体
  skill-copilot.md          ← GitHub Copilot CLI 变体（审计时 ≈ skill.md）
  skill-devin.md            ← Devin 变体（审计后新增）
  skill-droid.md            ← Factory Droid 变体
  skill-kilo.md             ← Kilo 变体（审计后新增）
  skill-kiro.md             ← Kiro 变体（审计时 ≈ skill-copilot.md）
  skill-opencode.md         ← OpenCode 变体
  skill-pi.md               ← Pi 变体（审计后新增）
  skill-trae.md             ← Trae 变体（原含 SyntaxError，已修复）
  skill-vscode.md           ← VSCode 变体（原削减 70% 功能，已修复）
  skill-windows.md          ← Windows/PowerShell 变体（最佳得分 61/100）
  __main__.py               ← CLI 入口
  security.py               ← 安全防护层（URL 校验、SSRF 阻断、DNS 重绑定防护）
  hooks.py                  ← Git hook 集成
  ...（其他 Python 模块）
AGENTS.md                   ← Agent 指令（审计得分 72/100，含过时 "graphify update" 指令）
pyproject.toml              ← Python 包清单
```

**变体维护模式——全复制（Full-Copy）而非共享基础**：

```
skill.md（主版本）
  ├── 功能 A + 功能 B + 功能 C + 功能 D（全量）
  ├── step-1: writes graphify-out/.graphify_python
  └── step-3A: reads graphify-out/.graphify_python（修复后一致）

skill-copilot.md（复制版，无差异化）
  └── 内容 ≈ skill.md（无 Copilot CLI 专属特性说明）

skill-kiro.md（复制版，无差异化）
  └── 内容 ≈ skill-copilot.md（完全冗余）

skill-aider.md / skill-claw.md 等（过时复制版）
  └── 缺少 skill.md 新增的功能：--wiki、--obsidian-dir、GitHub URL 克隆、uv 检测
```

**审计时架构问题核心**：15 个 Skill 文件全部独立维护（全复制），每次向 skill.md 添加新功能都需要手动同步到 14 个其他变体，而实际上从未系统性地同步，导致「功能漂移」从 11 个变体扩展到 15 个变体。

**NL 与二进制混合分发（nl-binary-hybrid）**：

```
PyPI 包
  ├── graphify/ 目录下的 Python 源代码（可执行逻辑）
  ├── graphify/skill*.md（NL artifacts，随包一起分发）
  └── 安装：pip install graphify / uv add graphify
```

这种分发模式使 Claude Code 规范路径（`skills/<name>/SKILL.md`）失效——PyPI 包维护者无法控制用户将包安装到哪里，因此 skill 文件随包源码存放而非放到规范位置。NLPM 的自动发现扫描器因此找不到这些 NL 工件，需要人工干预才能触发审计。

### 1.3 设计思路 / 方法论

**核心设计哲学：「以 AI 工具命名变体，降低每个 AI 用户的认知摩擦」**

graphify 的每个 skill 变体都以目标 AI 工具命名，使 Kiro 用户直接找到 `skill-kiro.md`，而不是在 `skills/` 下阅读通用文件。这种设计以维护成本换取用户的即时发现体验（搜索"graphify kiro skill"可以直接命中）。变体数量从 11 增长到 15，说明维护者认可这种设计哲学并持续扩展。

**解决什么问题**：知识图谱构建天然是跨工具场景——开发者在不同阶段使用不同 AI 工具（用 Cursor 写代码，用 Aider 重构，用 Kiro 做 PR）。graphify 希望无论用户在哪个工具中工作，都能把「把代码库转成知识图谱」这件事用同一套工具完成。每个 AI 工具都有不同的调用上下文（shell 命令格式、Python 解释器位置、系统路径约定），因此需要工具专属变体。

**security.py 的防御深度**：值得注意的是，security.py 的设计体现了相当的安全意识——URL 模式校验（scheme 白名单）、重定向重验证、DNS 重绑定防护（socket 打补丁）、响应大小上限。3 个 Medium 安全发现都是增量加固，而非根本性设计缺陷。这说明维护者对安全问题有主动意识，而非被动应付。

**trade-off**：
- **全复制 vs 共享基础**：选择了全复制（可读性、独立部署），代价是维护成本随变体数线性增长
- **非标准路径 vs 规范路径**：选择了随 PyPI 包分发（用户安装即用），代价是 NLPM 等工具无法自动发现
- **快速迭代 vs 变体同步**：选择了在主变体 skill.md 快速添加新功能，代价是 14 个复制变体持续落后于主版本

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：多 AI 工具适配的全复制 Skill 变体模式（NL-Binary Hybrid Distribution）**

一个工具的 NL artifacts 随二进制包一起通过包管理器（PyPI/npm）分发，同时为每个主流 AI 工具生成专属命名的 skill 副本，以降低每个 AI 用户的认知摩擦为代价换取维护成本的线性增长。

模式特征清单：
- **NL 与二进制混合打包**：skill 文件和 Python 源码在同一 PyPI 包内，安装后同步可用
- **AI 工具按名称索引**：每个变体文件名直接对应一个 AI 工具品牌，而非功能描述
- **全复制而非模板化**：每个变体是完整独立的 skill 文件，不通过 `loads:` 或 include 共享基础
- **变体增长驱动力**：每次支持新 AI 工具就新建一个完整文件（15 个变体仍在增长）

### 2.2 适用场景

| 场景 | 是否适用 | 原因 |
|---|---|---|
| PyPI/npm 包附带的 AI skill（跨工具分发） | ✅ 高度适用 | 本案例即为此场景，非标准路径是分发约束而非设计缺陷 |
| 单一 AI 工具的深度集成 skill | ❌ 不适用 | 单工具无需多变体，全复制成本无意义 |
| 工具差异明显（如 Windows PowerShell vs Linux bash） | ✅ 适用 | 差异足够大时，全复制可避免变体内大量 if-else 判断 |
| 工具差异极小（如 skill-copilot ≈ skill-kiro） | ❌ 不适用 | 差异不足以支撑独立文件，全复制只产生冗余维护成本 |
| Claude Code 插件（canonical paths 可用） | ⚠️ 避免 | 规范路径下可用 plugin.json + SKILL.md 单一来源，无需多变体复制 |

### 2.3 与其他架构对比

| 架构 | 代表性仓库 | 维护成本 | 发现体验 | 适合规模 |
|---|---|---|---|---|
| 全复制变体（本案例） | graphify | 随变体数线性增长 | 极优（工具名直接索引） | 3-5 个差异明显的变体 |
| 共享基础 + 覆盖层 | 无典型 Claude Code 案例 | 常数级基础 + 小覆盖 | 需要用户理解基础/覆盖概念 | 5+ 个变体，差异集中在少数段落 |
| 规范单一 SKILL.md | 大多数小型 Claude Code 插件 | 极低 | 标准路径自动发现 | 1-2 个 AI 工具 |
| 四层抽象梯度 | googleworkspace/cli | 中等（按层维护） | 需要理解分层命名约定 | 大型多域 API 覆盖 |

### 2.4 改进空间

**改进点 1：从全复制迁移到「共享基础 + 工具专属覆盖」**

- 当前问题：15 个 skill 文件中约 85% 内容重复（核心安装步骤、路径约定、知识图谱构建流程）。每次向 skill.md 添加功能都需要手动同步 14 个文件，实际上几乎没有同步（skill-vscode.md 曾削减 70% 功能即为证据）。
- 改进方式：建立 `graphify/skill-base.md`（基础层，包含所有工具共用的内容），各工具变体只保留差异段（如 PowerShell 命令替换、Codex 特定参数等），通过 `loads: graphify/skill-base` 加载基础。
- 预期收益：从 O(n) 维护成本降至 O(1) 基础 + O(k) 差异（k 为真实差异段数量）。对于 skill-copilot 和 skill-kiro 这类几乎无差异的变体，差异段可能只有 3-5 行。

**改进点 2：为所有变体添加 `<example>` 块**

- 当前问题：审计时全部 11 个 skill 变体（现 15 个）均无 `<example>` 块（-15 分/文件）。这是单个规则扣分最大的问题。
- 改进方式：在 skill-base.md（或每个变体）添加 2-3 个具体的调用示例：`graphify /path/to/repo --wiki`（生成 wiki）、`graphify /path/to/repo --obsidian-dir ~/Notes`（输出到 Obsidian）、`graphify https://github.com/user/repo`（克隆并分析）。
- 预期收益：每个变体得分从约 56-61 → 71-76（单规则 +15 分）。

**改进点 3：在 AGENTS.md 中添加示例并更新过时指令**

- 当前问题：AGENTS.md 得分 72/100，主要扣分来自无 `<example>` 块（-15）和「graphify update」指令过时（与实际 CLI 不符）。
- 改进方式：添加 3 个 `<example>` 块展示典型工作流，并将「graphify update」替换为当前正确的更新指令（`pip install --upgrade graphify` 或 `uv add --upgrade graphify`）。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

综合评分：**58/100**（12 个工件，加权平均）。安全评级：REVIEW（3 Medium，3 Low，0 Critical/High）。

| 文件 | 当时得分 | 主要问题 |
|---|---|---|
| AGENTS.md | 72 | 无 `<example>` 块（-15）；「graphify update」指令过时（-5）|
| graphify/skill-windows.md | 61 | 最佳变体——完整 PowerShell 移植；无 `<example>` 块（-15）；无 `version` 字段（-5）；非标准路径 R12（-10）|
| graphify/skill.md | 58 | 非标准路径 R12（-10）；解释器缓存路径漂移 Bug #2（-5）；无 `<example>` 块（-15）；无 `version` 字段（-5）|
| graphify/skill-codex.md | 57 | 功能落后（缺少 --wiki、--obsidian-dir 等）|
| graphify/skill-droid.md | 57 | 过时 + 路径漂移 |
| graphify/skill-copilot.md | 57 | 与 skill.md 功能完全相同，无差异化说明 |
| graphify/skill-kiro.md | 57 | 与 skill-copilot.md 内容完全一致 |
| graphify/skill-aider.md | 56 | 过时 + 解释器缓存路径漂移 |
| graphify/skill-claw.md | 56 | 过时 + 路径漂移 |
| graphify/skill-opencode.md | 56 | 与 skill-claw.md 几乎相同 |
| graphify/skill-trae.md | 56 | 第 829 行 Python SyntaxError（`'output':': 0}`）|
| graphify/skill-vscode.md | 52 | 最严重退化：削减约 70% 功能；第 154 行空循环占位符 |

**加权平均：58/100**

### 3.2 当时值得借鉴的模式

**模式 1：security.py 的防御深度设计**

- 为何有效：security.py 实现了多层防御——URL scheme 白名单（只允许 http/https）、重定向重验证（每次重定向后重新校验目的 URL）、DNS 重绑定防护（通过 socket 打补丁在解析时检查 IP 范围）、响应大小上限（防止 zip bomb）。在 NL 质量只有 58 分的情况下，安全层的设计是真正的亮点。
- 根本原因：维护者有意识地将安全逻辑从 `__main__.py` 分离到专用模块 `security.py`，使其可单独测试和审查。
- 借鉴：即使是以 NL artifacts 为主的 AI 工具插件，如果含有可执行代码（hooks/scripts/MCP），也应将安全校验逻辑集中到独立模块，而非散落在入口文件中。

**模式 2：skill-windows.md 的平台完整移植**

- 为何有效：skill-windows.md 是 11 个变体中得分最高的（61 分），因为它做了真正的平台适配——将所有 Linux/macOS 命令完整移植为 PowerShell 等效命令，而不是简单复制 Unix 命令并加上 Windows 前缀。这使 Windows 用户能无障碍执行。
- 根本原因：Windows 用户与 Linux 用户的执行环境差异足够大，值得一个独立且完整维护的变体。
- 借鉴：当目标平台差异导致命令行语法完全不同时（PowerShell vs bash），独立维护的全功能变体是合理的——这是全复制模式真正发挥价值的场景。

**模式 3：NL + PyPI 的混合分发策略**

- 为何有效：`pip install graphify` 后，skill 文件即在本地，AI 工具可以直接引用。用户无需单独下载或配置 skill 文件，降低了从「安装工具」到「AI 工具可用 skill」的摩擦。
- 借鉴：如果开发的是 PyPI/npm 分发的工具，将 skill 文件随包一起打包是减少用户配置步骤的合理选择。需要注意的代价是 NLPM 等工具的自动发现会失效，需要在文档中明确说明 skill 文件路径。

**模式 4：快速响应审计的维护文化**

- 为何有效：xiaolai 审计（2026-04-29）后，维护者 Safi 在 33 小时内通过 25 次 commits 独立修复了 11/12 个发现，部分与 Claude Sonnet 4.6 / Opus 4.7 协作完成。这种快速响应模式说明维护者有能力在短时间内消化大量质量反馈。
- 根本原因：维护者本身就频繁使用 Claude 协作（commits 记录可见 co-authored-by），AI 协作工作流已经内化。

### 3.3 当时的缺陷

**缺陷 1（BUG-syntax-error）：skill-trae.md 第 829 行 Python SyntaxError**

- 具体内容：`'output':': 0}` — 多了一个冒号，Python 解析器会直接抛出 SyntaxError。
- 根因：这是典型的「复制粘贴引入的字符误差」，极可能发生在将 Python 代码示例从 skill.md 复制到 skill-trae.md 的过程中。全复制维护模式使每次复制都是潜在的错误引入点。
- 影响：使用 Trae 调用 `--cluster-only` 选项的用户会得到回溯（traceback）而非结果，整个功能不可用。

**缺陷 2（BUG-path-mismatch）：skill.md（及 7 个变体）路径不一致**

- 具体内容：Step 1 写入 `graphify-out/.graphify_python`（含 `graphify-out/` 前缀），Step 3A 读取 `.graphify_python`（无前缀）——同一文件的写入路径和读取路径不同，首次运行必然失败（FileNotFoundError）。
- 根因：两个步骤由不同的编辑会话完成，没有做跨步骤的路径一致性检查。
- 影响：首次执行图谱构建时 Step 3A 失败，用户需要手动调试路径才能继续。

**缺陷 3（BUG-empty-loop-placeholder）：skill-vscode.md 第 154 行空循环占位符**

- 具体内容：`for chunk_json in []:` 配上「PASTE each subagent response here」注释——空列表的 for 循环永远不执行，图谱节点数为零。
- 根因：skill-vscode.md 是所有变体中退化最严重的（52 分），这个占位符说明维护者在开发 VSCode 变体时留下了未完成的模板代码，从未填充实际逻辑。
- 影响：所有通过 VSCode 变体调用 graphify 的用户会得到一个空图谱，完全无用。

**缺陷 4（CC-duplicate-variants）：skill-copilot.md ≈ skill-kiro.md ≈ skill.md**

- 具体内容：skill-copilot.md 与 skill.md 功能完全相同，无任何 GitHub Copilot CLI 专属说明；skill-kiro.md 与 skill-copilot.md 内容完全一致。3 个文件提供零差异化价值，却产生 3 倍维护成本。
- 根因：扩展 AI 工具支持时，维护者选择了「复制 + 重命名」而非「评估是否需要差异化」。

**缺陷 5（CC-stale-variants）：5 个变体功能落后于 skill.md**

- 具体内容：skill-aider.md、skill-claw.md、skill-codex.md、skill-droid.md、skill-opencode.md 均缺少 skill.md 后期新增的功能：GitHub URL 直接克隆、`--wiki` 标志、`--obsidian-dir` 标志、`uv` 包管理器检测。
- 根因：功能在 skill.md 添加后未同步到其他变体，全复制模式的固有代价。
- 影响：使用这些工具的用户无法发现和使用 graphify 的最新功能，体验劣于直接查看 skill.md 的用户。

### 3.4 当时的优化机会

1. **最高 ROI（单行修复，消除 SyntaxError）**：skill-trae.md 第 829 行删除多余的冒号（`'output':': 0}` → `'output': 0}`），立即恢复 Trae 用户的 `--cluster-only` 功能。
2. **高 ROI（路径统一，消除首次运行失败）**：在 skill.md 及 7 个变体中将所有 Step 3A 的路径引用统一为 `graphify-out/.graphify_python`（与 Step 1 一致）。
3. **高 ROI（VSCode 空循环，恢复功能）**：在 skill-vscode.md 中用实际的 chunk 处理逻辑替换 `for chunk_json in []:`，补充 VSCode 专属的 subagent 响应处理方式。
4. **中等 ROI（添加 `<example>` 块）**：所有 12 个文件均无示例块，是单规则扣分最大项（-15 分/文件）。为 skill.md 和 AGENTS.md 各添加 2-3 个具体示例，可提升整体评分约 8-12 分。
5. **低 ROI（合并冗余变体）**：将 skill-copilot.md 和 skill-kiro.md 标记为对 skill.md 的别名（或直接删除，在 AGENTS.md 中注明两个工具使用 skill.md）。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 验证方法 | 当前状态 | 备注 |
|---|---|---|---|
| Bug #1：skill-trae.md 第 829 行 SyntaxError `'output':': 0}` | `grep -n "'output':':" graphify/skill-trae.md` | ✅ **已修复**（PR #603 合并，xiaolai 流水线提交） | 唯一由 xiaolai 提交并合并的修复 |
| Bug #2：skill.md 路径前缀漂移（Step 1 写 `graphify-out/.graphify_python`，Step 3A 读 `.graphify_python`） | 查看第 96、103、108、168、206、267 行 | ✅ **已修复**（全部引用现为 `graphify-out/.graphify_python`，维护者独立修复） | — |
| Bug #3：skill-vscode.md 第 154 行 `for chunk_json in []:` | `grep -n "for chunk_json in" graphify/skill-vscode.md` | ✅ **已修复**（维护者独立修复） | xiaolai 案例注记：re-audit.md 标记为未修复，diff 表标记为已修复，两者冲突；此处采信 diff 表 |
| CC：skill-copilot.md ≈ skill-kiro.md（冗余变体） | 对比两个文件内容 | ✅ **已修复**（维护者独立修复，现有差异化内容） | — |
| CC：5 个过时变体缺少新功能 | 搜索 `--wiki\|--obsidian-dir` 各变体 | ✅ **已修复**（维护者独立修复） | — |
| 全部无 `<example>` 块（-15 分/文件） | `grep -c "<example>" graphify/skill*.md` | 状态待确认（当前 HEAD 数据未包含此项直接验证） | 如仍未添加，是得分最大的单项改进机会 |
| 全部无 `version` frontmatter 字段（-5 分/文件） | `grep -n "^version:" graphify/skill*.md` | 状态待确认 | — |

**整体结论**：原审计 12 个发现，11 个由维护者在 33 小时内独立修复，1 个（BUG-syntax-error）通过 xiaolai 的 PR #603 修复。当前 HEAD 已无原始 3 个 Bug。

### 4.2 架构演进

**从 11 个变体增长到 15 个变体**：当前 HEAD 新增了 skill-amp.md、skill-devin.md、skill-kilo.md、skill-pi.md。这说明：

1. 维护者认可「每个 AI 工具一个命名 skill」的设计哲学，并持续扩展
2. 全复制维护模式没有因为维护成本问题而被放弃，而是在扩大规模
3. 变体数量从 11 → 15，潜在的维护负担增加了 36%（若不引入共享基础机制）

**非标准路径约束未变**：skill 文件仍在 `graphify/skill*.md` 而非 `skills/<name>/SKILL.md`。NLPM 扫描器的路径发现限制在当前 HEAD 仍然存在，未来审计仍需人工干预。这是 PyPI 分发模式的结构性约束，不是疏忽。

### 4.3 新增的可学习模式

**快速维护突击模式（Burst-Fix Pattern）**：

维护者在审计后 33 小时内用 25 次 commits 修复 11 个发现，部分由 Claude Opus 4.7（1M context window）协作。这种「高密度短期修复冲刺」与传统的「渐进式 PR 合并」形成对比。对于维护者自己的仓库，当有可信审计报告（如 NLPM 输出）时，直接提交 commits 比走 PR 流程更高效——这解释了为什么 11/12 的修复都是直接提交而非 PR。

**外部审计的「认同效应」**：xiaolai 案例的核心洞察：维护者可能已经知道这些问题或正在处理相关代码区域，审计的价值在于「提供了一份有序的问题清单」而非「发现了维护者未知的缺陷」。外部审计是信号，维护者的快速响应是能力。

---

## 五、校准

### 5.1 我已经在做对的

1. **drama-workshop-skills 有初步的 skill 路径规范意识**：drama-workshop-skills 在 `short-drama/` 和 `short-drama-remake/` 下放置 skill 相关文件，虽然也是非标准路径，但相比 graphify 的 `graphify/skill*.md` 有更明显的 skill 组织意图（有子目录）。在路径规范性上与 graphify 面临类似问题，但出于不同原因（创作平台 vs PyPI 分发）。

2. **echo-sleuth 使用规范路径**：`agents/`、`commands/`、`skills/` 均符合 Claude Code 规范路径，NLPM 自动发现无障碍。这是 graphify 架构上缺失的优势。

3. **没有全复制变体问题**：我的仓库目前没有 15 个近乎相同 skill 文件的维护负担。drama-workshop-skills 的 short-drama vs short-drama-remake 是两个不同用途的工作流，而非同一工具针对不同 AI 工具的重复文件。

### 5.2 挑战 / 验证

**挑战 1：非标准路径的双刃剑**

graphify 的案例挑战了「规范路径是唯一正确选择」的直觉。对于 PyPI 分发的工具，非标准路径是合理的工程决策，不应被无条件评为缺陷。同样的逻辑适用于 drama-workshop-skills——如果工作流 skill 随创作项目一起打包分发，非标准路径可能是合理的。**我需要问自己：drama-workshop-skills 的非标准路径是分发约束（如 graphify）还是疏忽（没有意识到规范路径）？**

**验证：全复制的维护成本是可量化的**

graphify 审计明确量化了全复制的成本：skill.md 新增功能后，5 个过时变体仍缺失这些功能（2026-04-29 时）。从 11 个变体到 15 个变体，这个同步失败的概率在增大。这验证了「共享基础 + 覆盖层」方案的价值——drama-workshop-skills 的 short-drama vs short-drama-remake 如果不建立共享基础，未来也会出现类似的功能漂移。

**验证：security.py 级别的安全基础设施**

graphify 有专用 security.py，实现了 URL scheme 白名单、SSRF 阻断、DNS 重绑定防护。我的 3 个仓库都没有类似的安全基础设施。echo-sleuth 和 claude-for-legal 是纯 NL 工件（无可执行脚本），安全风险极低。但如果未来为 drama-workshop-skills 添加 hook 或 MCP，需要参照 graphify 的 security.py 设计安全层。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查 drama-workshop-skills 的 skill 文件是否有路径一致性问题（类比 Bug #2）
# 查找所有 skill 相关文件中引用的路径，检查 write 和 read 是否一致
grep -rn "graphify-out\|\.python\|cache" \
  /tmp/my-repos/MarkQWu-drama-workshop-skills/ 2>/dev/null
```
命中后怎么办：逐一核查写入路径和读取路径是否一致，修复不一致项。

```bash
# 2. 检查 drama-workshop-skills 两个变体是否存在功能漂移（类比 CC-stale-variants）
# 对比 short-drama/SKILL.md 和 short-drama-remake/SKILL.md 的内容差异
diff \
  /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/SKILL.md \
  /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama-remake/SKILL.md \
  2>/dev/null | head -60
```
输出差异较大后怎么办：评估差异是「有意的功能差异」还是「遗忘同步」——前者是合理设计，后者需要修复。

```bash
# 3. 检查所有 NL 工件是否有 <example> 块（graphify 最大单项扣分 -15/文件）
for f in /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md; do
  count=$(grep -c "<example>" "$f" 2>/dev/null || echo 0)
  echo "$count $f"
done
```
命中 0 后怎么办：为每个 SKILL.md 添加至少 2 个 `<example>` 块，展示典型触发短语和预期输出。

```bash
# 4. 检查 claude-for-legal 的模糊量词积累（所有仓库共同问题）
grep -rn "relevant\|appropriate\|some\|a few\|several" \
  /tmp/my-repos/MarkQWu-claude-for-legal/skills/*/SKILL.md 2>/dev/null | wc -l
```
命中超过 10 处后怎么办：按文件逐一排查，将「relevant documents」替换为具体文档类型列表，将「appropriate format」替换为具体格式名称。

```bash
# 5. 检查 drama-workshop-skills 的非标准路径是否是刻意决策
ls /tmp/my-repos/MarkQWu-drama-workshop-skills/short-drama/ 2>/dev/null
ls /tmp/my-repos/MarkQWu-drama-workshop-skills/skills/ 2>/dev/null
```
如果没有 `skills/` 目录：评估是否应迁移到规范路径（如果是 Claude Code 插件），或接受非标准路径（如果随其他系统打包分发）并在 CLAUDE.md 中明确说明路径位置。

### 6.2 灵感 → 实施路径

**灵感 1：为 drama-workshop-skills 建立「共享基础 + 覆盖层」防漂移机制**

- **为何可行**：drama-workshop-skills 的 short-drama 和 short-drama-remake 两个工作流共用大量基础概念（角色定义格式、场景描述规范、对白质量标准）。如果这些公共内容只在一个文件中维护，另一个发生功能漂移的风险与 graphify 的 skill-aider vs skill.md 完全一致。
- **第一步**：在 drama-workshop-skills 中建立 `shared/base-conventions.md`，将两个工作流文件中重复的内容（创作规范、格式约定）提取至此，并在两个工作流文件的 frontmatter 中添加 `loads: shared/base-conventions`。
- **时间预估**：内容提取 1-2 小时；验证两个变体功能不回退 30 分钟。

**灵感 2：为 echo-sleuth 的安全 hook 预留 security 模块设计**

- **为何可行**：echo-sleuth 目前是纯 NL 工件，但未来可能添加用于会话数据读取的 hook 或 MCP 配置。graphify 的 security.py 展示了当 NL 工具接触外部资源时应有的安全基础设施。
- **第一步**：在 echo-sleuth 的 CLAUDE.md 中添加「安全原则」节，声明「如未来添加 hook 或 MCP，需要：URL scheme 白名单、路径遍历防护、无凭证硬编码」——这是比事后修复更低成本的预防性文档。
- **时间预估**：文档写作 20 分钟。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

graphify 的核心目的：将任意代码库/文档库转化为可查询知识图谱，通过 PyPI 分发，以专属命名 skill 文件降低各 AI 工具用户的上手摩擦。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| **MarkQWu/drama-workshop-skills** | **高** | 两者都有多变体 skill 文件（graphify 15 个、drama-workshop 2 个）；都面临非标准路径问题；都有潜在的变体内容漂移风险 | graphify 是工具类（功能复杂，有可执行代码）；drama-workshop 是创作类（纯工作流，无二进制）；graphify 变体差异驱动力是 AI 工具适配，drama-workshop 差异驱动力是创作类型不同 | **高**（最直接的结构类比） |
| **MarkQWu/echo-sleuth-for-claude** | 低 | 都有面向 AI 工具的 skill 文件；都关注 Claude Code 生态 | echo-sleuth 使用规范路径，单一 skill，无变体；graphify 有 15 个变体和可执行二进制 | 中（安全模块设计可借鉴） |
| **MarkQWu/claude-for-legal** | 低 | 两者都需要处理多步骤工作流 | claude-for-legal 是法律领域专用，无 PyPI 分发，无多 AI 工具适配需求 | 低（主要借鉴：补全 `<example>` 块） |

### 8.2 在我的项目里复现的同类问题

| graphify 缺陷类型 | drama-workshop-skills 对应风险 | 验证方法 | 严重度 |
|---|---|---|---|
| CC-stale-variants（5 个过时变体缺少新功能） | short-drama-remake 未同步 short-drama 后期添加的新规范 | `diff short-drama/SKILL.md short-drama-remake/SKILL.md` | **高**（直接复现风险） |
| 无 `<example>` 块（-15 分/文件） | drama-workshop、echo-sleuth、claude-for-legal 的所有 SKILL.md | `grep -c "<example>" skills/*/SKILL.md` | **高**（所有仓库共同问题） |
| BUG-path-mismatch（写入路径 ≠ 读取路径） | drama-workshop-skills 中如有文件路径操作步骤 | `grep -n "write\|read\|path" short-drama/SKILL.md` | 中（待验证） |
| CC-duplicate-variants（无差异化的重复变体） | drama-workshop 的两个变体是否有真正的功能差异 | 对比两文件内容，识别「有意差异」vs「遗忘同步」 | 中（视实际差异而定） |
| 无 `version` frontmatter 字段（-5 分/文件） | 三个仓库的所有 SKILL.md | `grep -n "^version:" */skills/*/SKILL.md` | 低（易修复） |

**重要认知（来自 graphify 案例）**：drama-workshop-skills 的非标准路径问题在 NLPM 自动发现层面与 graphify 完全相同——扫描器找不到 `short-drama/SKILL.md`，因为规范路径是 `skills/<name>/SKILL.md`。这意味着 drama-workshop-skills 的 NLPM 评分需要人工干预，并且「NLPM 自动评分为 0 缺陷」很可能是「扫描器找不到 artifacts」而非「真正没有问题」。

### 8.3 别人的更优方案

**方案 1：PyPI 分发随包 skill 的组织方式**

- **graphify 做法**：`graphify/skill*.md`（与 Python 模块同目录，所有变体平铺）
- **我的 drama-workshop-skills 现状**：`short-drama/` 和 `short-drama-remake/` 子目录，层次更清晰，但没有统一前缀命名约定
- **graphify 的优势**：`skill-*.md` 前缀命名使所有变体在文件系统中一目了然（`ls graphify/skill*.md` 即可列出所有 skill）；drama-workshop 的分散在多个子目录中的 SKILL.md 需要递归搜索才能找到全部 skill 文件
- **如何借鉴**：为 drama-workshop-skills 建立命名约定——如果要保持子目录组织，则在每个子目录名中体现类型（`workshop-main/`、`workshop-remake/`），并在 CLAUDE.md 中明确列出所有 skill 路径

**方案 2：维护者快速响应的 AI 协作工作流**

- **graphify 做法**：在收到审计报告后，使用 Claude Sonnet 4.6 / Opus 4.7 协作，33 小时内完成 25 次 commits，修复 11 个发现
- **我的现状**：缺乏「接收外部质量报告 → 系统性修复」的 AI 协作工作流
- **如何借鉴**：建立「定期跑 NLPM 评分 → 将低分项整理为清单 → 与 Claude 协作批量修复 → 验证评分恢复」的周期性维护流程

### 8.4 反向：我的项目做得比他们好的地方

**1. echo-sleuth 和 claude-for-legal 使用规范路径（NLPM 自动可发现）**

graphify 因非标准路径导致 NLPM 扫描器完全无法发现，每次审计都需要人工干预。xiaolai 案例中「re-audit 给出 100/100」实际上是「0 artifacts found」的假完美分。

echo-sleuth 使用 `agents/`、`commands/`、`skills/` 规范路径，NLPM 可以自动发现所有工件，无需任何人工干预。这使得 CI/CD 中接入 NLPM 质量检查变得简单直接——不需要写任何特殊路径配置。

**2. drama-workshop-skills 的变体数量仍然可控（2 个 vs 15 个）**

graphify 的 15 个变体已经产生了可量化的维护负担（5 个过时变体、3 个无意义重复变体）。drama-workshop-skills 的 2 个变体在维护成本上远未到达临界点——即使完全独立维护，两个文件的同步也只需要几分钟人工检查。这是 drama-workshop-skills 目前的结构优势，但也是需要在扩展变体前建立共享基础机制的窗口期。

**3. 无可执行代码降低安全风险基线**

drama-workshop-skills 和 claude-for-legal 是纯 NL 工件（无 hooks、scripts、MCP），安全攻击面接近零。graphify 有 `__main__.py`、`hooks.py`、`security.py`，面临 URL 注入、路径遍历、SSRF 等真实安全风险（3 个 Medium 发现）。我的纯 NL 工件仓库天然规避了这个风险类别，但代价是功能受限（无法自动执行外部操作）。

---

## 八、术语表

| 术语 | 解释 |
|---|---|
| **PyPI** | Python Package Index，Python 官方包管理仓库（https://pypi.org）。通过 `pip install <package>` 或 `uv add <package>` 从 PyPI 安装包。graphify 通过 PyPI 分发，使得 skill 文件随 Python 包一起安装到用户本地环境，无需单独配置 skill 路径。但这也导致 skill 文件不在 Claude Code 的规范 `skills/<name>/SKILL.md` 路径下，NLPM 扫描器无法自动发现。 |
| **知识图谱（Knowledge Graph）** | 将信息（代码、文档、SQL 模式等）表示为实体（节点）和关系（边）的数据结构，支持结构化查询（「这个函数被哪些模块调用？」「这个表与哪些其他表有关联？」）。graphify 的核心功能是把任意文件夹转化为可查询的知识图谱，让 AI 工具能以结构化方式理解代码库。 |
| **skill variant（变体 Skill 文件）** | 针对同一工具功能，为不同目标 AI 工具（Claude Code、Codex、Aider、VSCode 等）单独创建的 skill 文件版本。变体可能只有命令语法、路径约定或平台命令的差异（如 Windows PowerShell vs bash），也可能几乎完全相同（如 skill-copilot.md ≈ skill.md）。变体数量越多，维护同步的成本越高。 |
| **non-canonical path（非标准路径）** | Skill 文件存放位置偏离 Claude Code 规范（`skills/<name>/SKILL.md`）的情况。graphify 将 skill 文件存放在 `graphify/skill*.md`（Python 包目录内），这是为了随 PyPI 包一起分发的刻意设计。非标准路径不一定是缺陷，但会导致 NLPM 等依赖规范路径的自动化工具（如扫描器）无法发现这些工件，需要人工干预才能触发审计或评分。 |
| **SyntaxError** | Python 解释器在解析源代码时发现语法不合法时抛出的异常。与运行时错误不同，SyntaxError 在代码被执行之前就会触发——即用户一旦尝试调用包含 SyntaxError 的代码块，立即得到回溯（traceback），功能完全不可用。graphify Bug #1（skill-trae.md 第 829 行 `'output':': 0}`）即为此类问题，由 xiaolai PR #603 修复。 |
| **uv（Python 包管理工具）** | Astral 开发的高速 Python 包管理器（https://docs.astral.sh/uv/），用于替代 pip + virtualenv 的组合。graphify 的 skill.md 在后期更新中添加了 `uv` 检测逻辑（自动判断用户使用 pip 还是 uv 来安装），但 5 个过时变体（skill-aider、skill-claw 等）在审计时仍缺少此功能，导致使用 uv 的用户在这些变体中得不到正确的安装指令。 |
