# czlonkowski/n8n-skills — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-skills
**Stars**：N/A | **来源**：upstream audit（exemplar_published=true）
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-13（基于当前 HEAD）
**主题标签**：`template-design`, `examples-driven`, `cross-reference`, `router-channels`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`czlonkowski/n8n-skills` 是一套专门教 Claude 用 [n8n-mcp MCP 服务器](#mcp-server)构建 [n8n](#n8n) 工作流的技能包。作者 Romuald Członkowski 是波兰 AI 顾问（aiadvisors.pl）。截至 2026-07-13 当前 HEAD：

- **15 个技能**（审计时 9 个；新增 6 个：n8n-code-tool、n8n-error-handling、n8n-binary-and-data、n8n-subworkflows、n8n-agents、using-n8n-mcp-skills 路由器）
- **hooks 执法层**（审计时不存在；现在有 session-start.sh、pre-tool-use、post-tool-use 三类钩子）
- **evaluations/ 评测集**（新增）
- 版本 1.23.0，MIT 协议
- 安装方式：`claude plugin install n8n-mcp-skills`

### 1.2 架构剖析
- **目录结构**：
  ```
  n8n-skills/
  ├── skills/              # 15 个 SKILL.md（按功能域分目录）
  │   ├── using-n8n-mcp-skills/   # always-on 路由器
  │   ├── n8n-expression-syntax/
  │   ├── n8n-mcp-tools-expert/
  │   ├── n8n-workflow-patterns/
  │   ├── n8n-validation-expert/
  │   ├── n8n-node-configuration/
  │   ├── n8n-code-javascript/
  │   ├── n8n-code-python/
  │   ├── n8n-code-tool/          # 新增
  │   ├── n8n-error-handling/     # 新增
  │   ├── n8n-binary-and-data/    # 新增
  │   ├── n8n-subworkflows/       # 新增
  │   ├── n8n-agents/             # 新增
  │   └── n8n-self-hosting/       # 部署运维（不在路由主流）
  ├── hooks/               # 执法层（新增）
  │   ├── hooks.json
  │   ├── session-start.sh
  │   ├── pre-tool-use/
  │   └── post-tool-use/
  ├── evaluations/         # 评测场景（新增）
  ├── docs/
  └── .claude-plugin/plugin.json
  ```
- **文件类型分布**：15 个 SKILL.md / 0 个 agent / 0 个 command / 3 类 hook 脚本
- **编排关系**：`using-n8n-mcp-skills` 是 always-on 路由器（由 session-start.sh 在每次会话注入），其他 14 个 skill 是叶节点，在具体场景下被路由器调度。hooks 执法层在工具调用前后自动触发相关 skill 的提醒，实现"决策点即时执法"。
- **跨件契约**：路由器 SKILL.md 中列出所有叶节点 skill 的名称和触发场景；PostToolUse 钩子解析 `validate_workflow` 的节点 JSON，自动路由到对应 skill；所有 skill 在 CLAUDE.md 的 "The 14 Skills" 清单中交叉列举。

### 1.3 设计思路 / 方法论
- **核心设计哲学**："MCP 提供数据通道，skill 提供决策智慧"——两者分工明确，skill 不重复 MCP 已提供的数据查询能力，只提供 HOW（如何选择节点、如何构造参数、如何处理错误）。
- **解决什么问题**：n8n-mcp MCP 服务器有 800+ 个节点，Claude 没有领域知识时容易选错节点类型、写错参数格式。skill 把这些专家经验结构化为可检索的 NL 知识，并通过 hooks 在"刚好需要的时刻"注入上下文。
- **做了什么 trade-off**：选择"hooks 主动推送"而非"用户手动 /load skill"——好处是 Claude 不会漏用知识；代价是 hooks 只能在 Claude Code CLI 安装时生效，claude.ai 上传 zip 的用户无此执法层。作者在 CLAUDE.md 中明确记录了这个差异。
- **反映什么认知模型**：作者把 skill 视为"专家顾问"，hooks 视为"顾问的出场触发器"——顾问不会等你问了再出现，而是在你做出关键决策的那一刻主动到场。这是从"知识库"思维到"执法层"思维的进化。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「分域 Skill + 钩子执法层」**

多个领域专家 skill 各自聚焦一个技术域（代码/验证/表达式/模式），always-on 路由器在入口处路由，hooks 在工具调用的前后时机自动激活对应 skill。

模式特征清单：
- **特征 1**：每个 skill 单职责，边界清晰（n8n-code-javascript 只管 JS 代码，n8n-validation-expert 只管校验）
- **特征 2**：有显式路由器（using-n8n-mcp-skills），不依赖 Claude 自动选择
- **特征 3**：hooks 在工具调用生命周期中嵌入知识注入，非侵入式（fail open）
- **特征 4**：每个 skill 都有描述中的"catch-all 子句"（even if they don't explicitly mention...），降低漏载率
- **特征 5**：辅助文件（如 `docs/CODE_NODE_BEST_PRACTICES.md`）被 skill 内引用，实现知识模块化

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 复杂 API / SDK 工具（有大量子命令和参数格式） | ✅ 高度适用 | 分域 skill 可为每个子系统提供专家知识；hooks 在调用点激活 |
| 个人工作流加速（通用写作、搜索） | ❌ 过度工程 | 不需要多域 skill 分工；单一 skill 够用 |
| 需要用户手动切换上下文的工具 | ⚠️ 有限适用 | hooks 层在非 CLI 环境失效；需 fallback 到路由器 |
| 有明确 MCP 服务器做数据层的场景 | ✅ 理想场景 | skill 只做知识层，不做数据层，分工天然契合 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 分域 Skill + 钩子执法层（本仓库） | czlonkowski/n8n-skills | 决策点即时执法，覆盖率高 | hooks 仅 CLI 生效；维护 hooks 脚本增加复杂度 |
| 单一 monolith skill | 早期 addyosmani/web-quality-skills | 安装简单，无依赖 | 超 500 行后 Claude 难以检索；专家知识混杂 |
| 路由器 + 叶节点（无 hooks） | dontbesilent2025/dbskill | 无执行表面，安全性高 | 依赖用户/Claude 手动触发；覆盖率低于有 hooks 的方案 |

### 2.4 改进空间
1. **当前问题**：hooks 执法层仅在 CLI 下生效；claude.ai 用户完全依赖路由器和手动触发。**改进做法**：在 `using-n8n-mcp-skills` 路由器描述中明确说明"如果你看到 n8n 工具调用，主动建议用户先加载对应 skill"，弥补 hooks 缺失时的漏洞。**预期收益**：无需修改代码，纯 NL 降低非 CLI 环境的覆盖率损失。
2. **当前问题**：CLAUDE.md 提到 `npm install @anthropic/claude-code-plugin-n8n-skills` 但 package.json 不存在（stale 引用）。**改进做法**：删除 CLAUDE.md 中的 npm 引用，或在 README 中注明 npm 包尚未发布。**预期收益**：消除误导性安装指引。
3. **当前问题**：vague word "effectively" 仍在 CLAUDE.md 中出现 2 次（"Teaches how to use n8n-mcp MCP tools effectively for building n8n workflows"）。**改进做法**：替换为具体行为："Teaches how to select the right n8n-mcp tools, format parameters, and handle validation errors"。**预期收益**：NLPM 得分 +2。

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）
93/100（9 个文件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 90 | "effectively"、"expert guidance" 模糊描述 |
| n8n-code-javascript/SKILL.md | 92 | "comprehensive"（×2）、"many" |
| n8n-code-python/SKILL.md | 92 | "comprehensive"/"complete"（×3） |
| n8n-mcp-tools-expert/SKILL.md | 92 | "effectively" 在 description 字段 |
| n8n-node-configuration/SKILL.md | 94 | "comprehensive" 在参考节 |
| n8n-validation-expert/SKILL.md | 94 | "comprehensive" 在参考节 |
| n8n-expression-syntax/SKILL.md | 95 | 无显著问题 |
| n8n-workflow-patterns/SKILL.md | 95 | 无显著问题 |
| .claude-plugin/plugin.json | 95 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **触发器升级（Trigger Escalation）** → `n8n-mcp-tools-expert` description 使用 `IMPORTANT — Always consult this skill before calling any n8n-mcp tool` 哨兵语，把 skill 从"可选参考"升级为"强制前置步骤"。**为何好**：Claude 默认倾向于直接行动，哨兵语强制中断并加载知识。**原文路径**：`skills/n8n-mcp-tools-expert/SKILL.md:3`。**如何借鉴**：在关键 skill 的 description 中加 `IMPORTANT —` 前缀，专用于"不读就会犯错"的场景。
2. **Catch-all 子句** → `n8n-workflow-patterns` description 末尾有"even if they don't explicitly mention 'patterns'"，覆盖用户措辞和 skill 关键词不一致的情况。**原文**：`skills/n8n-workflow-patterns/SKILL.md:3`。**如何借鉴**：每个 skill 描述中加一句"即使用户没有明确提到 XXX，当..."。
3. **跨 skill 分发表格** → 每个 skill 内部有"When to use other skills"表格，列出何时应该切换到哪个 sibling skill。消除了 Claude 在边界情况下的歧义。

### 3.3 当时的缺陷
1. **"comprehensive" 滥用（-4～-6 分/文件）** → 出现在 resource link 描述中（"Comprehensive data access patterns"）。根本原因：作者在资料引用时习惯用 comprehensive 作修饰词，未意识到这是 NLPM 的模糊量词。自查：我的 skill 是否有类似的资料引用标题用了 comprehensive？
2. **"effectively" 在 description 字段（-2 分）** → `n8n-mcp-tools-expert` description 说"use n8n-mcp MCP tools effectively"——effectively 没有告诉 Claude 具体要做什么。根本原因：把"好的效果"写进了触发器描述，而不是把触发场景写进去。自查：我的 description 字段是否有效果词代替了行动词？
3. **CLAUDE.md 提到 npm 安装路径但 package.json 不存在** → 轻微的文档腐烂。根本原因：文档和代码并行演进，没有自动化检测 CLAUDE.md 中的引用是否仍然有效。自查：我的 CLAUDE.md 是否有陈旧的安装/使用指令？

### 3.4 当时的优化机会（仅供学习）
1. 每个 resource link description 中的 "comprehensive/complete" → 替换为具体章节标题（如"Data access patterns: list, filter, aggregate"）
2. `n8n_health_check()` 在 `n8n-mcp-tools-expert/SKILL.md` 中提到但未在 CLAUDE.md 中列出 → 补充到"Key MCP Tools"清单
3. npm 安装引用 → 删除或替换为实际可用路径

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| "comprehensive" 在 skills/ 中滥用 | `grep -rn "comprehensive" /tmp/target-czlonkowski-n8n-skills/skills/` → 96 hits | **仍然存在**，且命中数增加（新增了 6 个 skill，带入了新的 comprehensive） | 作者没有优先处理模糊量词 |
| "effectively" 在 CLAUDE.md | `grep -n "effectively" /tmp/target-czlonkowski-n8n-skills/CLAUDE.md` → 2 hits | **仍然存在** | 原始 CLAUDE.md 的 description 语言未更新 |
| npm 安装引用但无 package.json | `cat CLAUDE.md | grep npm` → 仍存在 | **仍然存在** | 文档腐烂持续 |

### 4.2 架构演进
从 9 skills（平铺）→ 15 skills + hooks 执法层 + evaluations：
- **新增的最大变化**：hooks 执法层。作者意识到 skill 只有被"触发"才有价值，于是加入了 session-start.sh（注入 router）、pre-tool-use（节点特定提醒）、post-tool-use（validation 结果路由）三层钩子。这说明作者从"构建知识库"迁移到了"构建执法系统"的认知。
- **新增 evaluations/**：测试场景文件夹，说明作者开始思考如何验证 skill 的实际效果。这和 NL-TDD 理念一致。
- **新增 6 个 skill**：覆盖 code-tool、error-handling、binary-and-data、subworkflows、agents 等高级主题——说明用户反馈推动了知识边界扩展。

### 4.3 新增的可学习模式
1. **Post-tool-use 路由器**：`hooks/post-tool-use/` 脚本解析 validate_workflow 的 JSON 输出，自动路由到对应 skill。这比"用户读懂 JSON 再选 skill"效率高一个数量级。
2. **明确的 dist/ 约定**：CLAUDE.md 注释 `dist/` 是 gitignored，zip 通过 GitHub Release 发布——防止了"提交了 zip 导致 Desktop/Cowork 插件安装失败"的陷阱（作者踩过）。

---

## 五、校准

### 5.1 我已经在做对的
1. **单职责 skill**：我的 gstack skill 也是按功能域分文件（setup-deploy/、land-and-deploy/、diagram/ 等），与本仓库理念一致。
2. **触发短语多元化**：我的 gstack SKILL.md description 字段也包含多个触发场景描述，不只写"what it does"。
3. **跨 skill 引用**：我的 echo-sleuth-for-claude 架构（commands → agents → skills → scripts）有明确的分层调用关系，与本仓库的路由器→叶节点模式相似。

### 5.2 挑战 / 验证
本案例挑战了我的假设：**"skill 自己的 description 够用，不需要 hooks"**。n8n-skills 用 3 类钩子证明：description 触发 ≠ 执法层保障。在关键决策点（调用特定 MCP 工具之前/之后），hooks 提供了 description 无法提供的"强制性"。如果我的 bureau 或 echo-sleuth 有"用户容易漏用某个 skill"的场景，值得考虑加 SessionStart hook 注入路由器。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill description 中是否有 effectively/comprehensive/complete 作修饰词
grep -rn -E '\b(effectively|comprehensive|complete|expert guidance)\b' \
  /tmp/my-repos/MarkQWu-gstack/**/*.md \
  /tmp/my-repos/MarkQWu-bureau/**/*.md 2>/dev/null | grep -v "No AI vocabulary"
# 命中后：把修饰词替换为具体动作描述
```

```bash
# 检查我的 CLAUDE.md 是否有陈旧安装路径引用（npm/pip 等但包不存在）
grep -n "npm install\|pip install\|brew install" \
  /tmp/my-repos/MarkQWu-*/CLAUDE.md 2>/dev/null
# 命中后：验证对应包是否发布，删除未发布的引用
```

```bash
# 检查我的 hooks/ 是否有未使用的执法机会
# 如：某个 skill 是"工具调用前必看"的，但我没有 PreToolUse hook 提醒用户
ls /tmp/my-repos/MarkQWu-bureau/hooks/ 2>/dev/null
# 如果缺少 pre-tool-use hook 而 bureau 有"必须先执行 init"这类需求，考虑添加
```

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 的 `/recall` command 加一个 PostToolUse hook，在 Bash 工具执行完 JSONL 解析脚本后自动提醒用户可用的 synthesis skill。**为何可行**：echo-sleuth 的 `scripts/echolib.py` 已经模块化，hook 可以检测 echolib 调用并路由。**第一步**：在 `commands/recall` 目录旁新建 `hooks/post-tool-use/echolib-router.sh`，先实现 echo 提示，5 分钟可完成原型。
2. **想法**：给 bureau 的关键 command（如 `lint.md`）在 description 中加 IMPORTANT 哨兵语，明确"必须在提交前运行"。**为何可行**：本仓库证明 IMPORTANT 哨兵可以把 skill 从可选变为强制。**第一步**：编辑 `/tmp/my-repos/MarkQWu-bureau/commands/lint.md` 的 description 字段，加一行 `IMPORTANT — Run before every commit to avoid corrupting the canon.`，改动 1 行。

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 czlonkowski/n8n-skills 的核心目的**：通过 15 个专域 skill + hooks 执法层，让 Claude 在 n8n MCP 工作流构建场景下具备专家级判断力。
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是多 skill 集合，功能域分文件，有 hooks.json | gstack 有 binary 核心（TypeScript CLI）；n8n-skills 纯 NL | 高 |
| MarkQWu/bureau | 中 | 都有 hooks 执法层；都有 always-on session 级上下文注入 | bureau 是知识管理系统；n8n-skills 是工具使用指南 | 中 |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 command → agent/skill 路由层 | echo-sleuth 有 scripts 数据层；n8n-skills 依赖 MCP | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| resource link description 中含 "comprehensive" | `grep -rn "comprehensive" /tmp/my-repos/MarkQWu-gstack/*.md 2>/dev/null` | gstack 的 "No AI vocabulary" 规则本身包含 "comprehensive"（作为禁用词举例），但 skill 正文未发现用作修饰词；**0 命中** | 低 |
| description 中含 "effectively" | `grep -n "effectively" /tmp/my-repos/MarkQWu-gstack/SKILL.md 2>/dev/null` | 0 命中 | 低 |
| 陈旧安装路径引用 | `grep -n "npm install" /tmp/my-repos/MarkQWu-*/CLAUDE.md 2>/dev/null` | bureau/CLAUDE.md: `node press/bin/gazette.mjs build`（有效路径）；**无陈旧引用** | 无 |

**结论**：我的项目在 vague word 控制上做得比本案例好（gstack 明确列出禁用词单，体系更严格）。

**命中后的具体行动建议**：本次扫描未命中，无需即时行动。下一步建议每次新增 skill 后运行上述 grep 作为 pre-commit 检查。

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：钩子执法层的分层设计（session / pre-tool-use / post-tool-use）
   - **本案例做法**：三类钩子分文件（`hooks/session-start.sh`, `hooks/pre-tool-use/`, `hooks/post-tool-use/`），职责清晰，按生命周期组织。
   - **我的项目现状**：bureau/hooks/ 只有 `hooks.json`，没有分生命周期的子目录结构；gstack 有 hooks 但未看到 pre/post-tool-use 分层。
   - **如何借鉴**：如果 bureau 需要"工具调用后触发复核"，可以参考本案例的 post-tool-use 目录结构，将各 hook 脚本按触发时机分子目录管理，而不是全部放在 hooks.json 中。

2. **领域**：evaluations/ 测试场景集
   - **本案例做法**：有独立的 `evaluations/` 目录存放 skill 测试场景。
   - **我的项目现状**：echo-sleuth-for-claude 有 `.nlpm-test/` 下的 NL-TDD 测试文件，但 gstack 和 bureau 未见评测集。
   - **如何借鉴**：给 gstack 的关键 skill 加 evaluations/ 目录，定义"正常路径 / 边界路径"两类测试场景，输入一个 prompt，期望 Claude 输出的工具选择顺序。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：vague word 的系统性禁止
- **我的做法**：MarkQWu/gstack 在每个 SKILL.md 末尾有明确的"No AI vocabulary"禁用词列表（delve, crucial, robust, comprehensive, nuanced...），相当于把 NLPM 规则内化为 skill 内的 linting 规则。
- **本案例做法**（弱在哪）：没有禁用词声明，依赖审计者事后发现。结果是 "comprehensive" 在 96 处命中、audit 发现后 3 个月仍未修复。
- **意义**：gstack 的"在 SKILL 内声明禁用词"是一种自防御机制，值得作为最佳实践推广——如果 PR reviewer 和审计工具都看到这个列表，违规率会大幅下降。

---

## 八、术语表

### <a name="n8n"></a>n8n
> 一个开源的工作流自动化平台，类似 Zapier / Make，但可以自托管。用户通过可视化界面连接不同服务（Slack、数据库、API 等）来构建自动化流程。"节点"是 n8n 的基本组成单元，每个节点代表一个动作（如"发 HTTP 请求"、"写数据库"）。

### <a name="mcp-server"></a>MCP 服务器
> Model Context Protocol（模型上下文协议）服务器，为 Claude 提供工具调用能力的外部服务。Claude 通过调用 MCP 工具来访问数据库、API、文件系统等资源。`n8n-mcp` 就是一个专门查询 n8n 平台数据（节点信息、模板、工作流）的 MCP 服务器。

### <a name="always-on-skill"></a>always-on 路由器
> 每次会话开始时自动加载的 skill，不需要用户手动触发。在本仓库中，`using-n8n-mcp-skills` 由 SessionStart hook 在每次会话开始时注入，确保 Claude 在任何 n8n 相关对话中都知道应该使用哪个专域 skill。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（如 `name`、`description`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能注册和路由这个 skill。

### <a name="catch-all-clause"></a>catch-all 子句
> skill description 中加入的容错语句，覆盖用户措辞与 skill 关键词不完全匹配的情况。例如"even if they don't explicitly mention 'patterns'"——让 Claude 在用户没说"pattern"这个词的时候也加载工作流模式 skill。
