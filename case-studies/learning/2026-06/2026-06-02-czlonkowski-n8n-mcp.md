# czlonkowski/n8n-mcp — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-mcp
**Stars**：18,662 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-02（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `model-pinning`, `manifest-discipline`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

czlonkowski/n8n-mcp 是一个为 n8n 工作流自动化平台提供 MCP（Model Context Protocol）服务器的工具集，同时内置了一套 AI agent 辅助开发配置。仓库有 18,662 颗星，说明它在 n8n 生态与 Claude Code 社区中有相当广泛的采用。

项目定位非常具体：让 AI agent（Claude Code）能够通过标准化的 MCP 工具接口查询、验证和测试 n8n 节点配置，从而在 n8n 工作流开发过程中提供精准的智能辅助。这是一个典型的「工具增强型 agent 套件」——仓库主体是 MCP 服务器，NL 工件（CLAUDE.md + 8 个 agent 定义）是其开发辅助层。

核心 NL 工件：
1. `CLAUDE.md` — 项目级主配置，质量评分 92 分，内容相当详细
2. `.claude/agents/` 下共 8 个专用 agent：`code-reviewer.md`、`context-manager.md`、`debugger.md`、`deployment-engineer.md`、`mcp-backend-engineer.md`、`n8n-mcp-tester.md`、`technical-researcher.md`、`test-automator.md`

可执行面（安全关注点）：
- `scripts/test-http.sh` — 含 `eval "$cmd"` 构造，存在 shell 注入风险
- `test-n8n-integration.sh` — 含 `sudo apt-get` 等提权链
- `deploy-to-vm.sh` — `source .env` 前未做验证

### 1.2 架构剖析

```
n8n-mcp/
  CLAUDE.md                          # 主配置（92分）
  .claude/
    agents/
      code-reviewer.md               # 有 model 声明（claude-sonnet-4-5）
      n8n-mcp-tester.md              # 有 model 声明，但有工具引用 bug
      context-manager.md             # 无 model 声明
      debugger.md                    # 无 model 声明
      deployment-engineer.md         # 无 model 声明
      mcp-backend-engineer.md        # 无 model 声明，无输出格式声明
      technical-researcher.md        # 无 model 声明
      test-automator.md              # 无 model 声明
  scripts/
    test-http.sh                     # CRITICAL: eval shell 注入
    test-n8n-integration.sh          # HIGH: sudo 提权链
    deploy-to-vm.sh                  # MEDIUM: .env 未验证即 source
  src/                               # MCP 服务器主体代码（TypeScript）
  package.json                       # "prepare": "husky"，版本号未钉死
```

8 个 agent 的职责分工非常清晰，体现了单一职责设计：
- `mcp-backend-engineer` — MCP 后端工程师，负责核心开发
- `n8n-mcp-tester` — 通过 MCP 工具接口测试 n8n 节点
- `code-reviewer` — 代码审查，规范执行
- `debugger` — 调试辅助
- `deployment-engineer` — 部署流程
- `context-manager` — 上下文管理与记忆维护
- `technical-researcher` — 技术调研
- `test-automator` — 自动化测试生成

### 1.3 设计思路 / 方法论

n8n-mcp 的设计遵循一条清晰的主线：**将 AI agent 能力与 MCP 工具接口深度绑定，让 agent 通过结构化工具调用而非自由文本查询来辅助 n8n 开发**。这种思路有以下几个值得关注的方法论特点：

**职责分层**：CLAUDE.md 描述整体架构（什么是 n8n-mcp、目录结构、构建方式），各 agent 文件只描述单一角色的行为规则。不同于把所有指令堆在 CLAUDE.md 里的做法，这里的分层是刻意设计的。

**测试 agent 与 MCP 服务器共生**：`n8n-mcp-tester.md` 是整个项目最有特色的设计——它的 `tools:` 列表声明了 MCP 服务器暴露的测试工具，理论上 agent 的测试行为完全通过 MCP 工具接口发生，而不是靠随意调用 CLI。这种设计在大型工具类仓库中并不常见。

**品牌声明嵌入技术文档**：CLAUDE.md 第 210 行要求所有提交和 PR 包含"Concieved by Romuald Członkowski"及链接。这是一个有争议的设计选择——将营销声明硬编码进技术规范文件，会造成技术读者的认知混乱，也给代码审查和提交历史带来噪音。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**工具增强型多角色 agent 套件（Tool-Augmented Multi-Role Agent Suite）**

核心特征：
- 主体是一个具体工具（MCP 服务器），NL 工件是其开发辅助层，不是独立产品
- 多个 agent 各司其职，通过 `tools:` 字段显式声明允许使用的工具子集
- agent 的行为约束通过工具白名单而非文字叙述来实现

这种模式与纯粹的「技能集合型」仓库（如 karpathy-skills）不同——后者不涉及可执行工具，前者每个 agent 都在一个受约束的工具调用空间内行动。

### 2.2 适用场景

适合复制这种模式的项目特征：
1. 项目主体是一个 MCP 服务器或具有明确工具接口的 CLI 工具
2. 需要多个 agent 分别负责不同开发阶段（研究、开发、测试、部署）
3. 工具集较大，不同阶段需要使用不同的工具子集，用白名单防止误用
4. 测试行为需要与主工具深度集成（如 `n8n-mcp-tester` 直接调用被测试的 MCP 工具）

不适合的场景：
- 工具接口不稳定或频繁变更（tools 白名单会成为维护负担）
- 项目本身是纯内容/纯文本工作流，没有明确的工具接口

### 2.3 与其他架构对比

| 维度 | n8n-mcp（工具增强型） | karpathy-skills（技能集合型） | claude-for-legal（领域工具型） |
|------|----------------------|------------------------------|-------------------------------|
| NL 工件角色 | 工具开发辅助层 | 独立知识产品 | 领域流程编排 |
| agent 数量 | 8（按角色分工） | 0（仅 SKILL.md） | 多个（按领域分工） |
| 工具绑定 | 强（tools 白名单） | 无 | 中等 |
| 安全面 | 宽（scripts 目录） | 零（无执行面） | 中等 |
| 可移植性 | 低（强依赖 n8n 生态） | 高（通用指南） | 中（法律领域专用） |

### 2.4 改进空间

1. **tools 白名单完整性**：`n8n-mcp-tester.md` 中引用了 `get_node_info`、`get_node_essentials`、`validate_node_config`、`get_node_example` 这 4 个工具名，但 `tools:` frontmatter 中没有声明这些名称（最接近的声明是 `mcp__n8n-mcp-testing__validate_node`，并非精确匹配）。这类 manifest-body 不一致会导致 Claude Code 的工具权限检查失效。

2. **model 字段补全**：8 个 agent 中只有 2 个声明了 `model` 字段。未声明 model 的 agent 会使用运行时默认值，无法做到行为可预期，也无法在发布时控制成本。

3. **安全脚本重构**：`eval "$cmd"` 是 shell 脚本中最危险的构造之一，可以用 `"$cmd"` 直接执行或改用 `case` 语句分发来消除注入面。

4. **MCP 依赖文档化**：`n8n-mcp-tester.md` 依赖 `n8n-mcp-testing`、`supabase-telemetry`、`plugin_postgres-best-practices_supabase` 三个 MCP 服务器，但 CLAUDE.md 的安装/配置章节完全没有提及这些依赖。这是典型的「文档盲区」。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**综合 NL 分数：81/100**
**安全状态：BLOCKED（CRITICAL 级别）**

各工件得分（从低到高）：

| 工件 | 得分 | 主要扣分原因 |
|------|------|------------|
| mcp-backend-engineer.md | 65 | 缺少 model (-5)、缺少输出格式 (-10)、模糊限定词封顶 (-20) |
| context-manager.md | 75 | 缺少 model (-5)、模糊限定词封顶 (-20) |
| deployment-engineer.md | 75 | 缺少 model (-5)、模糊限定词封顶 (-20) |
| technical-researcher.md | 75 | 缺少 model (-5)、模糊限定词封顶 (-20) |
| test-automator.md | 75 | 缺少 model (-5)、模糊限定词封顶 (-20) |
| debugger.md | 91 | 缺少 model (-5)、少量模糊词 (-4) |
| n8n-mcp-tester.md | 91 | 工具引用未声明 (-5)、少量模糊词 (-4) |
| CLAUDE.md | 92 | 少量模糊词 (-8) |
| code-reviewer.md | 94 | 少量模糊词 (-6) |

### 3.2 当时值得借鉴的模式

1. **CLAUDE.md 的详细程度**：92 分的 CLAUDE.md 覆盖了项目整体架构、目录结构说明、构建命令、开发规范。这是本仓库 NL 质量最高的部分，值得作为项目配置文件的参考标准。

2. **agent 职责单一化**：8 个 agent 没有任何功能重叠，每个文件只描述一个角色的行为。这种设计使得单个 agent 的维护成本极低，且故障范围可控。

3. **code-reviewer.md 的高质量**：94 分，是除 CLAUDE.md 外得分最高的工件，说明代码审查类 agent 的编写相对规范——审查规则具体，无大量模糊词。

4. **测试 agent 与被测系统的工具绑定**：`n8n-mcp-tester.md` 通过 `tools:` 声明尝试将测试 agent 的能力范围限定在 MCP 工具接口上，思路正确，只是执行存在缺陷。

### 3.3 当时的缺陷

**Bug 类（n8n-mcp-tester.md）**：
- `get_node_info` 出现在 body 中，未在 `tools:` 里声明
- `get_node_essentials` 出现在 body 中，未在 `tools:` 里声明
- `validate_node_config` 出现在 body 中，未在 `tools:` 里声明（最接近的声明是 `mcp__n8n-mcp-testing__validate_node`）
- `get_node_example` 出现在 body 中，未在 `tools:` 里声明
- step 编号错误：两个连续步骤都标为"6."

**系统性质量问题**：
- 6/8 个 agent 缺少 `model:` 字段
- 大量模糊限定词：`"comprehensive knowledge"`、`"proper TypeScript types"`、`"appropriate comments"`、`"relevant"`、`"clean, maintainable"` 等
- `mcp-backend-engineer.md` 缺少 `output_format:` 声明

**结构性问题**：
- CLAUDE.md 第 210 行嵌入品牌营销声明，要求每个提交/PR 包含 `"Concieved by Romuald Członkowski"` 及个人链接

**安全问题**：
- CRITICAL：`eval "$cmd"` shell 注入（`scripts/test-http.sh` 第 51 行）
- HIGH：sudo 提权链（`test-n8n-integration.sh`）
- MEDIUM：`source .env` 前未验证（`deploy-to-vm.sh`）
- LOW：husky postinstall 钩子，`^` 版本号未钉死

### 3.4 当时的优化机会

1. 将 6 个缺少 `model:` 的 agent 补全，统一指定模型版本，消除运行时不确定性
2. 将 `n8n-mcp-tester.md` 中 4 个工具引用与 `tools:` 声明对齐（或统一命名规范）
3. 用 `case` 语句或直接执行替换 `eval "$cmd"` 消除 shell 注入
4. 在 CLAUDE.md 中补充 `n8n-mcp-testing` 等 MCP 依赖的安装说明
5. 将营销声明从 CLAUDE.md 移至 README 或 CONTRIBUTING.md

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

经过 2026-06-02 的现状核查（基于当前 HEAD），结论如下：

**未修复（全部持续存在）**：

| 缺陷 | 审查时状态 | 当前状态 |
|------|-----------|---------|
| `get_node_info` 未在 tools 中声明 | 存在（body 第 31 行） | **持续存在** |
| `get_node_essentials` 未在 tools 中声明 | 存在（body 第 32 行） | **持续存在** |
| `validate_node_config` 未在 tools 中声明 | 存在（body 第 33/65/67 行） | **持续存在** |
| `get_node_example` 未在 tools 中声明 | 存在（body 第 35 行） | **持续存在** |
| 6/8 agent 缺少 model 字段 | context-manager, deployment-engineer, mcp-backend-engineer, technical-researcher, test-automator, debugger | **全部持续缺失** |
| `eval "$cmd"` shell 注入 | `scripts/test-http.sh` 第 51 行 | **持续存在**（`local response=$(eval "$cmd")`） |

**已修复**：
- 无明确证据显示任何缺陷已被修复

**变化**：
- agent 文件集合与审查时相同（仍是 8 个，无增删）
- `code-reviewer.md` 和 `n8n-mcp-tester.md` 仍是唯二有 model 字段的 agent

### 4.2 架构演进

从审查日（2026-04-06）到生成日（2026-06-02），约两个月时间内，n8n-mcp 的 NL 工件层基本没有演进。agent 文件集合、工具声明方式、model 字段缺失情况均与审查时一致。

这是一个常见现象：当 NL 工件本身不是仓库的核心产品（主产品是 MCP 服务器代码）时，维护者倾向于把开发重心放在可执行代码上，NL 工件的质量改进优先级偏低。

### 4.3 新增的可学习模式

鉴于两个月内 NL 层无实质性改动，本节记录的是审查中已发现但更值得深入分析的持续性模式：

**「工具声明漂移」反模式的形成机制**：`n8n-mcp-tester.md` 的 tools-body 不一致很可能源于以下流程：先写 body（直接用人类可读的简短工具名），后补 `tools:` 声明（用 MCP 完整限定名），两者命名规范不同，未做对齐检查。这种漂移在工具接口迭代时尤为常见——MCP 服务器改了工具名，但 agent body 里的引用没有同步更新。

**教训**：`tools:` 声明中的工具名是机器执行时的权限依据，body 中的工具名是人类可读的意图描述，两者必须严格对齐，不能各自演化。

---

## 五、校准

### 5.1 我已经在做对的

1. **MarkQWu/claude-for-legal 的 model 字段合规性**：claude-for-legal 中所有 agent 都声明了 `model:` 字段，这一点比 n8n-mcp 的 6/8 缺失要好。这说明在 agent 定义的完整性上，我有基础规范意识。

2. **NL 工件与工具绑定的分离意识**：在 drama-workshop-skills 中，SKILL.md 不涉及工具调用，避免了 tools-body 不一致问题。对于不需要工具的场景，不引入工具声明是正确的。

3. **避免可执行面的滥用**：drama-workshop-skills 没有 scripts 目录，echo-sleuth-for-claude 也没有 `eval` 类危险构造，执行面相对干净。

### 5.2 挑战 / 验证

1. **echo-sleuth 的 model 字段问题是系统性的**：5/5 个 agent 全部缺少 model 字段，和 n8n-mcp 的问题一模一样。n8n-mcp 两个月内未修复这个问题，说明仅靠意识提醒不够，需要 CI 检查或 pre-commit hook 来强制约束。

2. **模糊限定词在 echo-sleuth 中同样存在**：`appropriate`、`comprehensive`、`robust`、`efficient`、`relevant` 各出现至少一次。这说明即使知道这是问题，在实际编写时依然容易滑入「用形容词代替规范」的惯性。

3. **能否做到 tools-body 严格对齐**：当项目引入 MCP 工具时，我是否有机制确保 agent body 里引用的工具名与 `tools:` 声明精确一致？目前没有任何自动检查。

---

## 六、行动

### 6.1 自查动作

以下 grep 命令可在任意仓库根目录执行，排查与 n8n-mcp 相同类型的问题：

**检查 agent 是否缺少 model 字段**：
```bash
# 列出 .claude/agents/ 下所有缺少 "^model:" 的文件
grep -rL "^model:" .claude/agents/
```

**检查 agent 是否缺少 description 字段**：
```bash
grep -rL "^description:" .claude/agents/
```

**扫描模糊限定词（常见 vague quantifier）**：
```bash
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|relevant|proper|clean|maintainable|reasonable|suitable|adequate|thorough|clear|meaningful)\b' .claude/agents/
```

**检查 tools frontmatter 与 body 的一致性（列出 body 中出现的工具调用模式）**：
```bash
# 找出 body 中形如 `tool_name(` 或 `use tool_name` 等引用
grep -n -E '(use |call |invoke |run )?\b[a-z][a-z0-9_]+\b\(' .claude/agents/
```

**检查脚本中的 eval 构造**：
```bash
find . -name "*.sh" -not -path "./.git/*" | xargs grep -n '\beval\b' 2>/dev/null
```

**检查 sudo 使用（高权限操作）**：
```bash
find . -name "*.sh" -not -path "./.git/*" | xargs grep -n '\bsudo\b' 2>/dev/null
```

**检查 .env 的不安全 source**：
```bash
grep -rn 'source .*\.env' . --include="*.sh"
```

**检查 package.json 中未钉死的版本号**：
```bash
grep -E '"[^"]+": "\^' package.json 2>/dev/null
```

**针对我自己的 echo-sleuth 仓库的具体检查**：
```bash
# 假设仓库在 ~/projects/echo-sleuth-for-claude
grep -rL "^model:" ~/projects/echo-sleuth-for-claude/agents/
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|relevant)\b' ~/projects/echo-sleuth-for-claude/agents/
```

### 6.2 灵感 → 实施路径

**修复 echo-sleuth 的 model 字段缺失（优先级：高）**：

1. 确定项目预期使用的 Claude 模型（如 `claude-sonnet-4-5`）
2. 对 agents/ 目录下所有缺少 `model:` 的文件，在 frontmatter 中添加该字段
3. 提交前用 `grep -rL "^model:" agents/` 验证无遗漏
4. 可选：在 pre-commit hook 或 CI 步骤中加入该检查，防止未来遗漏

**消灭模糊限定词（优先级：中）**：

对每个命中的模糊词，按以下步骤替换：
- `"appropriate comments"` → `"每个公开函数头部一行说明注释"`
- `"comprehensive knowledge"` → `"熟悉 X 的具体机制（列举 3 项）"`
- `"relevant context"` → `"最近 5 次对话的工具调用记录"`
- 原则：用可验证的具体标准替代形容词判断

**建立 tools 白名单维护规范（优先级：中，适用于有 MCP 工具的项目）**：

1. 在每个 agent 文件顶部 `tools:` 列表中，工具名必须使用完整的 MCP 限定名（如 `mcp__server-name__tool-name`）
2. body 中引用工具时，使用与 `tools:` 完全一致的名称，不使用简短别名
3. 每次 MCP 服务器接口变更时，同步检查所有 agent 的 `tools:` 声明

**对 scripts 目录的安全整改（适用于有脚本的项目）**：

- 将 `eval "$cmd"` 改为：先用 `case` 语句枚举允许的命令，再执行；或直接用 `"$cmd"` 避免二次解析
- 将 `source .env` 改为：先检查文件存在性和权限，再 source，或改用显式 `export KEY=VALUE` 读取

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

n8n-mcp 是「n8n 工作流自动化的 MCP 服务器 + AI agent 辅助开发工具集」。与我的三个仓库对比：

| 仓库 | 对齐度 | 说明 |
|------|-------|------|
| MarkQWu/echo-sleuth-for-claude | 中（最接近） | 同为工具使用增强型 AI agent 套件，有多个专用 agent，面向特定工具链 |
| MarkQWu/claude-for-legal | 低 | 领域工作流工具，不是 MCP 服务器，agent 结构相似但场景完全不同 |
| MarkQWu/drama-workshop-skills | 无 | 内容创作领域，技能集合型，无 MCP 工具绑定 |

echo-sleuth-for-claude 是最值得对比学习的项目：同样是多 agent 套件，同样有工具绑定需求，同样存在 model 字段缺失和模糊限定词问题。

### 8.2 在我的项目里复现的同类问题

**echo-sleuth-for-claude（5/5 agent 缺少 model 字段）**：

与 n8n-mcp 的 6/8 问题完全相同。以下命令可以验证：
```bash
grep -rL "^model:" agents/
```
预期输出：5 个 agent 文件全部列出。

**echo-sleuth-for-claude（4 处模糊限定词命中）**：

```bash
grep -rn -E '\b(appropriate|comprehensive|robust|efficient|relevant)\b' agents/
```
预期输出：至少 4 处命中，分布在不同 agent 文件中。

这两个问题在 n8n-mcp 中存在且两个月未修复，说明它们在没有外部检查机制的情况下具有强烈的「自然稳定性」——不会自动消失，需要主动干预。

### 8.3 别人的更优方案

**n8n-mcp 的 CLAUDE.md 质量（92 分）远优于我的同类文件**：

n8n-mcp 的 CLAUDE.md 覆盖：整体架构说明、目录结构注释、构建命令、开发规范、agent 职责索引。相比之下，echo-sleuth-for-claude 的 CLAUDE.md 质量有待验证，但根据该项目其他 NL 工件的质量水平，很可能也存在模糊词和不完整结构。

行动：参照 n8n-mcp CLAUDE.md 的结构，检查并补完 echo-sleuth-for-claude 的 CLAUDE.md，至少包含：项目目的一段话、目录结构注释表、agent 职责索引、常用命令列表。

**n8n-mcp 的 agent 职责分工极度清晰**：

8 个 agent 各司其职，无功能重叠。我的仓库是否达到同样的职责单一性，需要逐一审查每个 agent 的 `description:` 字段是否存在功能交集。

### 8.4 反向：我的项目做得比他们好的地方

**MarkQWu/claude-for-legal 的 model 字段 100% 合规**：

所有 agent 都声明了 `model:` 字段，这直接避免了 n8n-mcp 最严重的系统性质量问题。在 claude-for-legal 中建立了正确习惯，后续需要把这个习惯迁移到 echo-sleuth-for-claude。

**drama-workshop-skills 的零安全面**：

无 scripts 目录、无 MCP 配置、无 hooks 文件，安全评级天然是 CLEAR。对比 n8n-mcp 的 CRITICAL 安全问题，drama-workshop-skills 在安全性上零风险。这不是刻意设计的结果，但结果是正确的——对于纯内容创作工具，不引入可执行面本身就是最佳实践。

**没有营销声明嵌入技术文档**：

我的任何仓库都没有在 CLAUDE.md 或 agent 文件中嵌入品牌归属声明，保持了技术文档的纯粹性。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **MCP（Model Context Protocol）** | Anthropic 定义的模型上下文协议，允许 AI agent 通过标准化接口调用外部工具服务。`tools:` frontmatter 中声明的是 agent 有权调用的 MCP 工具列表。 |
| **n8n** | 一个开源的工作流自动化平台，类似 Zapier，允许通过节点连接各类服务。n8n-mcp 为其提供了 MCP 服务器层。 |
| **agent（NL agent / Claude Code agent）** | `.claude/agents/` 目录下的 Markdown 文件，定义了一个专用子代理的行为规范。由 `model`、`description`、`tools` 等 frontmatter 字段和 body 正文组成。 |
| **frontmatter** | Markdown 文件顶部两行 `---` 之间的 YAML 结构，用于声明机器可读的元数据（如 model、description、tools）。frontmatter 字段是 Claude Code 运行时的权限和行为依据。 |
| **tools 白名单（tools frontmatter）** | agent 文件中 `tools:` 字段声明的允许工具列表。Claude Code 只允许 agent 调用白名单内的工具，body 中引用的工具名必须与之精确匹配。 |
| **tools-body 不一致（manifest-body drift）** | `tools:` frontmatter 中声明的工具名与 agent body 正文中引用的工具名不一致的状态。常见原因：MCP 服务器改名后未同步更新 agent 文件，或编写时使用了简短别名而非完整限定名。 |
| **model 字段缺失（missing model field）** | agent frontmatter 中未声明 `model:` 字段，导致 Claude Code 使用运行时默认模型。缺陷：行为不可预期，版本升级时无法控制兼容性，成本无法约束。 |
| **模糊限定词（vague quantifier）** | agent 或 skill 文件中使用了无法量化验证的形容词短语，如 `"comprehensive"`、`"appropriate"`、`"relevant"`、`"clean"`。NLPM 评分规则对此单独扣分，累计封顶 -20 分。 |
| **NLPM 评分** | Natural Language Programming Manager 的 100 分制质量评分体系。从 100 分起扣，按规则表应用确定性惩罚，下限 0 分，上限 100 分。默认合格线 70 分。 |
| **shell 注入（shell injection）** | 脚本中将外部输入（如 `$1` 命令行参数）直接传入 `eval` 执行，攻击者可通过构造特殊输入执行任意 shell 命令。NLPM 安全扫描将此列为 CRITICAL 级别。 |
| **安全门控（security gate）** | NLPM auditor 流程中，在 NL 质量审查之前强制执行的安全扫描步骤。若发现 CRITICAL 级别安全问题（如 `eval` 注入），仓库被标记为 `security-blocked`，后续的贡献（contribute）流程拒绝运行。 |
| **工具增强型 agent 套件** | 本文归纳的架构模式：仓库主体是具体工具（MCP 服务器、CLI 等），NL 工件（CLAUDE.md + agents）是其开发辅助层，agent 行为通过工具白名单约束而非自由文本叙述。 |
| **单一职责原则（single-purpose）** | 每个 agent 文件只描述一个角色、一类任务，不与其他 agent 的职责重叠。n8n-mcp 在 agent 职责分工上遵循了这一原则，8 个 agent 各司其职。 |
| **CLAUDE.md** | Claude Code 项目级主配置文件，位于仓库根目录，每次会话自动加载。包含项目架构说明、开发规范、常用命令等。相当于「给 Claude 的项目级说明书」。 |
| **vague quantifier 封顶（-20 cap）** | NLPM 评分规则：模糊限定词的总扣分上限为 -20 分，无论实际命中多少处。这意味着一个满是模糊词的 agent 文件，仅此一项最多被扣去 20 分，其他问题独立计算。 |
| **品牌声明嵌入（attribution injection）** | 将作者署名、营销声明等非技术内容硬编码进技术规范文件（如 CLAUDE.md）的做法。n8n-mcp CLAUDE.md 第 210 行的 `"Concieved by Romuald Członkowski"` 要求是典型案例。这会污染提交历史，增加 PR 审查噪音。 |
