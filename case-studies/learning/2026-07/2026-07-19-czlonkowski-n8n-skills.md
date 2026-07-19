# czlonkowski/n8n-skills — 学习案例

**仓库**：https://github.com/czlonkowski/n8n-skills
**Stars**：暂无（registry null，exemplar_published=true）| **来源**：upstream
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-19（基于当前 HEAD）
**主题标签**：`cross-reference`, `examples-driven`, `single-purpose`, `template-design`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

一个专门教 Claude 如何通过 [n8n-mcp](#n8n-mcp) 服务器构建 n8n 工作流的 Claude Code 插件，由波兰 AI 顾问 Romuald Członkowski 创建。当前版本 v1.25.0，包含 15 个互补技能 + hooks 强制层 + JSON 格式的评估测试。通过 `claude plugin install` 安装，主要用户是需要用 AI 自动构建 n8n 自动化流程的开发者和运维人员。

### 1.2 架构剖析

- **目录结构**：

```
n8n-skills/
├── skills/                    # 15 个技能实现
│   ├── using-n8n-mcp-skills/  # 路由器技能（SessionStart hook 加载）
│   ├── n8n-mcp-tools-expert/  # MCP 工具专家
│   ├── n8n-workflow-patterns/ # 工作流架构模式
│   ├── n8n-expression-syntax/ # 表达式语法
│   ├── n8n-code-javascript/   # JS 代码节点
│   ├── n8n-code-python/       # Python 代码节点
│   ├── n8n-code-tool/         # Code Tool 节点
│   ├── n8n-error-handling/    # 错误处理
│   ├── n8n-binary-and-data/   # 二进制数据
│   ├── n8n-subworkflows/      # 子工作流
│   ├── n8n-agents/            # AI Agent 节点
│   ├── n8n-multi-instance/    # 多实例管理
│   ├── n8n-self-hosting/      # 自托管部署
│   ├── n8n-node-configuration/ # 节点配置
│   └── n8n-validation-expert/ # 验证专家
├── hooks/                     # 强制层
│   ├── hooks.json             # hook 注册
│   ├── session-start.sh       # 注入路由器技能
│   ├── pre-tool-use/          # 工具调用前的 hint 脚本
│   └── post-tool-use/         # 工具调用后的路由脚本
├── evaluations/               # 每个技能的 JSON 评估测试
├── .claude-plugin/plugin.json # 插件清单
└── build.sh                   # 打包脚本
```

- **文件类型分布**：15 个 SKILL.md / 0 个 agent / 0 个 command / 4 个 hook 脚本
- **编排关系**：`using-n8n-mcp-skills` 是路由器，由 SessionStart hook 每次会话注入。PreToolUse hook 在特定 MCP 工具调用前触发技能提示，PostToolUse hook 解析 `validate_workflow` 输出并路由到对应技能。其余 14 个技能是平行专家层。
- **跨件契约**：所有 SKILL.md 在 `description` 字段列出完整触发词集合，支持"即使用户没提到技能名"也能被路由触发。每个技能的 `Resources` 节都相互引用兄弟技能。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「MCP 提供数据，Claude 技能提供如何用数据的智慧」—— 技能内容是专家级 HOW-TO，而不是 n8n 节点的文档镜像。
- **解决什么问题**：n8n MCP 服务器提供了访问 800+ 节点的能力，但 Claude 在没有上下文时经常选错 nodeType 格式、用错参数结构。技能层解决的是"哪个工具，怎么用，为什么这样用"。
- **做了什么 trade-off**：独立技能仓库 vs. 打包进 n8n-mcp。选择了独立仓库，因为技能是 Claude 端的认知增强，而 MCP 是 n8n 端的数据访问，两者生命周期和维护主体不同。
- **反映什么认知模型**：作者把 Claude 看作一个需要「场景路由 → 技能激活 → 执行指导」三阶段的系统，hooks 强制层确保这个管道在每次工具调用时都能触发，而不依赖用户主动输入技能命令。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「MCP 数据层 + 技能指导层 + Hook 强制层」三层分离架构**

这个架构的关键特征：
- **特征 1**：技能内容与数据访问层完全解耦——技能不操作数据，只教 Claude 如何正确操作
- **特征 2**：路由器技能 (`using-n8n-mcp-skills`) 是会话级"协调人"，其余技能是"专家"，职责单一
- **特征 3**：Hook 层实现「无感加载」—— 用户无需知道 skill 名，系统在正确的时机自动触发
- **特征 4**：evaluations/ 目录将测试用例与技能一对一对应，NL 层面的 TDD
- **特征 5**：技能描述中的 IMPORTANT 哨兵句和"即使没提到关键词"子句，减少触发漏洞

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要配合特定 MCP 服务器的工作流自动化 | ✅ 高度适用 | 技能层天然与 MCP 层分工，两边独立更新 |
| 面向最终用户的自助式工具（无须记命令名） | ✅ 适用 | Hook 强制层消除了"知道用哪个技能"的认知负担 |
| 单一领域的专家技能集（如报税、代码审查） | ✅ 适用 | 可按领域独立维护，不互相干扰 |
| 简单的单技能场景 | ❌ 过度设计 | 三层架构只在有 MCP + 多技能时才值回维护成本 |
| 需要跨会话状态的复杂工作流 | ❌ 不适用 | 技能层是无状态的，跨会话记忆需要额外机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 三层分离（本仓库） | czlonkowski/n8n-skills | 用户无感、触发精准、可独立演进 | 维护成本高：hook 脚本需与 MCP API 保持同步 |
| 纯平铺技能 | addyosmani/agent-skills | 简单直接，易于贡献 | 用户需要知道哪个技能名，容易遗漏 |
| 路由器 + 子技能（无 hook） | dontbesilent2025/dbskill | 路由逻辑在 NL 层，可读性强 | 依赖用户主动触发 `/dbs`，有首次使用门槛 |

### 2.4 改进空间

1. **当前问题**：`plugin.json` 缺少 `entries`/`skills` 数组，自动化工具无法枚举插件暴露的技能面 **改进做法**：添加 `"skills": ["skills/n8n-workflow-patterns", ...]` 列表 **预期收益**：支持 NLPM 等工具的无 filesystem scan 枚举
2. **当前问题**：vague quantifiers（comprehensive、effectively）仍在 3 个文件中出现 **改进做法**：替换为具体描述，如「covers X, Y, Z three areas」 **预期收益**：NLPM 评分从 93 提升到 95+
3. **当前问题**：hook 脚本路径硬编码 `${CLAUDE_PLUGIN_ROOT}`，在某些安装路径下可能失效 **改进做法**：增加路径存在性检查和降级逻辑 **预期收益**：增强跨平台安装稳定性

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 93/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 90/100 | "effectively"、"expert guidance" 模糊措辞 |
| n8n-code-javascript/SKILL.md | 92/100 | "comprehensive"(×2)、"many" |
| n8n-code-python/SKILL.md | 92/100 | "comprehensive"/"complete"(×3) |
| n8n-mcp-tools-expert/SKILL.md | 92/100 | "effectively" 在 description 字段 |
| n8n-node-configuration/SKILL.md | 94/100 | "comprehensive" in reference section |
| n8n-validation-expert/SKILL.md | 94/100 | 同上 |
| n8n-expression-syntax/SKILL.md | 95/100 | 无重大问题 |
| n8n-workflow-patterns/SKILL.md | 95/100 | 无重大问题 |
| .claude-plugin/plugin.json | 95/100 | 无重大问题 |

### 3.2 当时值得借鉴的模式

1. **触发升级哨兵（IMPORTANT 句）** → 在 description 末尾加「IMPORTANT — Always consult this skill before calling any n8n-mcp tool」，告知 Claude 这是加载顺序指令，不只是触发词。根本原因：大语言模型倾向于先行动后查参考，哨兵句打断这个默认行为。原文：`skills/n8n-mcp-tools-expert/SKILL.md:3`。借鉴：对有「先加载后操作」需求的技能（如安全检查技能），添加此句型。

2. **「即使没提到关键词」的覆盖句** → n8n-workflow-patterns 在 description 末尾加「even if they don't explicitly mention 'patterns'」。根本原因：用户语言与技能名词汇不对齐是触发失败最常见原因。原文：`skills/n8n-workflow-patterns/SKILL.md:3`。借鉴：所有技能描述末尾，为最容易被用户绕过的触发场景加兜底句。

3. **✅/❌ 运行对比示例** → n8n-expression-syntax 将错误语法和正确语法并排展示，无任何散文描述。根本原因：具体代码比文字描述快 10 倍传递"什么是对"。原文：`skills/n8n-expression-syntax/SKILL.md:20-27`。借鉴：重构我的技能中任何以文字描述「不要做 X」的地方，改为代码对比。

4. **版本锁定一致性** → plugin.json v1.4.0 与 build.sh `VERSION="1.4.0"` 一致，无人工维护漂移。根本原因：版本是隐式合同，不一致时用户无法信任任何一个。原文：`build.sh:VERSION` 与 `plugin.json:version`。借鉴：任何多文件版本声明，在 CI 中加版本一致性检查。

5. **全部内部引用可解析** → 每个 SKILL.md 的 Resources 节引用的所有 `.md` 文件均在仓库中实际存在。根本原因：死链接是对用户信任的慢性损耗。审查中零死链接。借鉴：在 CI 中加内部链接扫描（`grep -rE '\[.*\]\(.*\.md\)' | xargs -I{} test -f`）。

### 3.3 当时的缺陷

1. **模糊量词「comprehensive」「effectively」** → 在 description 和资源链接中反复出现，总共 -7 分扣减。为什么会失败：Claude 读到"comprehensive guide"无法从中推断应该加载哪些具体内容，只是知道这是个"全面"的指南。自查：我的 bureau 技能中有类似措辞吗？→ 有，bureau/SKILL.md 中的 description 字段待检查。

2. **plugin.json 缺少 entries/skills 数组** → 自动化工具无法枚举技能，需要 filesystem scan。为什么这么设计会失败：当插件生态工具（如 NLPM、marketplace）期望从 manifest 读取技能列表时，会得到空对象，造成"无技能"的误判。自查：我的 bureau/plugin.json 也缺少 skills 字段（已确认）。

3. **CLAUDE.md npm 安装路径陈旧** → CLAUDE.md 描述用 `npm install @anthropic/claude-code-plugin-n8n-skills` 安装，但仓库无 package.json。为什么这么设计会失败：首次使用者跟着文档操作会立即失败，形成第一印象伤害。自查：我的仓库文档中有没有已失效的安装步骤？

### 3.4 当时的优化机会

1. 所有 description 字段替换「comprehensive」→「covers X, Y, Z」形式的具体清单
2. `plugin.json` 添加 `skills` 数组，支持无 filesystem 的自动化枚举
3. CLAUDE.md 中删除 npm 安装路径，只保留 `claude plugin install` 的安装方式

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| vague quantifiers（comprehensive/effectively/many） | `grep -rn "comprehensive\|effectively\|many" skills/n8n-code-javascript/SKILL.md` | **持续**：n8n-code-javascript:270、340 仍有；CLAUDE.md:11、58 仍有 | 4 个月内此类问题未引起重视，说明作者认为这是可接受的措辞 |
| plugin.json 缺 entries/skills | `cat .claude-plugin/plugin.json \| python3 -c "import json,sys; print(json.load(sys.stdin).get('skills','MISSING'))"` | **持续**：字段不存在 | 生态工具集成仍然受限 |
| CLAUDE.md npm 安装路径 | `grep "npm install" CLAUDE.md` | **已修**：CLAUDE.md 已更新，不再提 npm 安装 | 说明作者关注了用户首次体验 |

### 4.2 架构演进

从 audit 时的 9 个技能 → 现在的 15 个技能，增加了：
- `n8n-code-tool`（Code Tool 节点专家）
- `n8n-error-handling`（错误处理）
- `n8n-binary-and-data`（二进制数据处理）
- `n8n-subworkflows`（子工作流）
- `n8n-agents`（AI Agent 工作流）
- `n8n-multi-instance`（多实例管理）
- `using-n8n-mcp-skills`（路由器，由 hook 加载）

**最重大演进**：新增了 `hooks/` 目录，引入了 SessionStart/PreToolUse/PostToolUse 三类 hook，将技能从「用户主动调用」转变为「系统自动路由」。这说明作者意识到，即使技能写得再好，如果用户不知道该调用哪个，系统就无法发挥全部价值。

`evaluations/` 目录的出现，意味着作者开始用 JSON 测试场景对技能进行 NL 层面的验证，是向 NL-TDD 演进的信号。

### 4.3 新增的可学习模式

**Hook 强制层（Enforcement Layer）**：这是当前 HEAD 中最重要的新设计。`session-start.sh` 在每次会话时自动将路由器技能注入上下文；PreToolUse hook 在用户调用特定 MCP 工具前，主动推送技能提示；PostToolUse hook 解析工具输出并路由到对应技能。

这个模式的核心价值：**技能的激活从依赖用户行为转移到了系统触发**，彻底消除了「用户不知道该调哪个技能」的问题。

---

## 五、校准

### 5.1 我已经在做对的

- **版本一致性意识**：我的 bureau/plugin.json 中版本与实际内容保持同步，没有版本漂移
- **单职责技能**：我的 bureau 的每个技能（capture, recall, review, compile...）都有明确单一职责，不混叠
- **内部链接可解析**：我的技能中内部引用的文件均实际存在
- **详细操作步骤**：我的 gstack 技能普遍有编号步骤，避免模糊说「do X」

### 5.2 挑战 / 验证

**这次案例验证了一个我之前犹豫的做法**：是否值得为技能集写 hooks。n8n-skills 的演进路径（先纯技能，后加 hooks）说明：当技能数量超过 5-7 个时，用户的认知负担会到达临界点，hooks 这时候才是真正值得投入的。

我的 gstack 目前有约 15 个技能，已经超过了这个临界点——我应该认真考虑添加 SessionStart hook 来自动注入路由器逻辑。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能中是否有 comprehensive/effectively 等模糊量词
grep -rn -E "\b(comprehensive|effectively|appropriate|robust)\b" \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null

# 检查我的 plugin.json 是否有 skills/entries 字段
cat /tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json | python3 -c "
import json,sys; d=json.load(sys.stdin); 
print('skills:', d.get('skills','MISSING')); 
print('entries:', d.get('entries','MISSING'))"
```

命中后怎么办：
- `comprehensive` → 替换为「covers X, Y, Z」
- plugin.json 缺 skills → 在 bureau/.claude-plugin/plugin.json 中添加 `"skills": ["skills/capture", "skills/recall", ...]`

### 6.2 灵感 → 实施路径

1. **想法**：给 gstack 或 bureau 添加 SessionStart hook，自动注入路由器技能
   - **为何可行**：gstack 已有 15 个技能，用户记不住哪个技能做什么；hooks 可以将选择成本从用户转移到系统
   - **第一步**：在 bureau 或 gstack 目录下创建 `hooks/hooks.json`，添加 SessionStart hook 来注入 `guide` 技能；参考 czlonkowski `hooks/session-start.sh` 的结构；约 30 分钟

2. **想法**：在 bureau/plugin.json 中添加 `skills` 数组，列出所有技能路径
   - **为何可行**：bureau 已有 7 个 SKILL.md，添加 skills 数组只是机械性更新
   - **第一步**：打开 `bureau/.claude-plugin/plugin.json`，在末尾加 `"skills": ["skills/capture", "skills/recall", "skills/scribe", "skills/lint", "skills/compile", "skills/review", "skills/guide"]`；约 5 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 czlonkowski/n8n-skills 的核心目的**：通过技能层 + hook 强制层，让 Claude 在使用 n8n MCP 服务器时始终做出正确决策

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 多技能 + CLAUDE.md 路由 + 有 MCP 工具使用场景 | gstack 无 hooks 层，技能触发依赖用户主动输入 | 高 |
| MarkQWu/bureau | 中 | 有 plugin.json + 多技能 + 跨技能协作链 | bureau 无 hook，且无 entries 字段 | 中 |
| MarkQWu/graphify | 低 | 也是 Claude Code 技能 | graphify 是单技能，无路由问题 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺 skills/entries 字段 | `cat bureau/.claude-plugin/plugin.json \| python3 -c "..."` | **bureau 命中**：字段不存在 | 中 |
| 技能无 hooks 自动路由 | `ls gstack/hooks/ bureau/hooks/ 2>/dev/null` | **两者均无 hooks 目录** | 中 |
| description 中模糊量词 | `grep -rn "comprehensive\|effectively" bureau/skills/` | 需实地检查 | 低 |

**命中后的具体行动建议**：
- bureau/.claude-plugin/plugin.json → 添加 `"skills"` 数组 → 5 分钟可完成
- gstack 技能目录 → 创建 `hooks/hooks.json` + `hooks/session-start.sh` → 参考 czlonkowski 的 session-start.sh 结构 → 30 分钟可完成

### 8.3 别人的更优方案

1. **领域**：Hook 强制层（无感触发）
   - **本案例做法**：`hooks/session-start.sh` 在每次会话自动注入路由器技能；PreToolUse hook 在 MCP 调用前主动推送 hint（`hooks/hooks.json`）
   - **我的项目现状**：gstack 和 bureau 均无 hooks 目录，用户每次需要记住对应技能名或手动输入
   - **如何借鉴**：在 bureau 添加 `hooks/hooks.json` + `hooks/session-start.sh`，SessionStart 时加载 `guide` 技能；gstack 中添加同样机制加载路由技能；参考 czlonkowski `hooks/hooks.json` 的 json schema

2. **领域**：触发升级哨兵句型
   - **本案例做法**：`IMPORTANT — Always consult this skill before calling any n8n-mcp tool` —— 这是加载顺序指令，不只是触发词
   - **我的项目现状**：bureau 技能 description 只有普通触发词，无优先级指令
   - **如何借鉴**：对 bureau 的 `lint` 技能（应在每次提交前运行），在 description 末尾加 IMPORTANT 哨兵句

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：详细的标准输出格式定义
- **我的做法**：bureau 每个技能有明确的 `## Output` 节，规定输出格式和结构
- **本案例做法**（弱在哪）：n8n-skills 的技能输出格式定义分散在各 SKILL.md 中，没有统一的 `## Output` 节
- **意义**：如果有人审查我的 bureau，Output 节的存在会是一个明显的亮点

---

## 八、术语表

### <a name="n8n-mcp"></a>n8n-mcp

> n8n 官方提供的 [MCP](#MCP) 服务器，让 Claude 可以直接调用 API 来访问 n8n 平台的 800+ 个集成节点的元数据、验证工作流配置、读取模板等。相当于给 Claude 装了一个"n8n 数据库的查询接口"。

### <a name="MCP"></a>MCP

> Model Context Protocol，Anthropic 制定的一个开放协议，让 AI 助手可以安全地连接到外部数据源和工具。`mcp__xxx__yyy` 格式的工具调用就是通过 MCP 调用外部服务的方式。

### <a name="hook"></a>Hook

> Claude Code 的事件钩子机制，允许在特定事件（如会话开始、工具调用前/后）触发自定义 shell 脚本。`hooks.json` 文件声明哪个事件触发哪个脚本。

### <a name="frontmatter"></a>frontmatter

> Markdown 文件最顶部用 `---` 包起来的 YAML 配置区，用于声明元数据（如 `name`、`description`）。Claude Code 通过解析 SKILL.md 的 frontmatter 来注册和匹配技能。

### <a name="路由器技能"></a>路由器技能（Router Skill）

> 一个专门负责「看上下文、选最合适的子技能」的技能，本身不做具体任务。相当于一个总调度，收到用户的模糊请求后，分配给对应的专家技能处理。
