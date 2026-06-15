# IvanMurzak/Unity-MCP — 学习案例

**仓库**：https://github.com/IvanMurzak/Unity-MCP
**Stars**：⭐未收录 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-06-15（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `security-gate`, `manifest-discipline`, `single-purpose`, `model-pinning`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Unity-MCP 是一个让 Claude Code 直接控制 Unity 编辑器的 MCP（Model Context Protocol）插件，通过 C# Unity 插件 + 自托管 MCP server 实现 AI 与 Unity 之间的双向通信。用户安装 Unity 插件后，Claude Code 就能直接操作 Unity 场景对象、资源、脚本、测试——无需切换窗口。整个系统是「NL 表皮（78 个 SKILL.md 告诉 Claude 怎么用）+ 原生二进制核心（C# MCP server 实际执行 Unity API）」的典型混合架构。

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── CLAUDE.md                     # 项目上下文
  ├── Unity-MCP-Plugin/             # Unity 侧插件（C#）
  │   ├── CLAUDE.md                 # Plugin-specific 上下文
  │   └── .claude/
  │       └── skills/               # 78 个 SKILL.md（Unity API 封装）
  │           ├── gameobject-create/SKILL.md
  │           ├── scene-open/SKILL.md
  │           ├── assets-find/SKILL.md
  │           └── ... (共 78 个，覆盖 GameObject/Scene/Assets/Script/Tests)
  ├── Unity-MCP-Server/             # MCP server（.NET/C#）
  ├── cli/                          # Node.js CLI 工具
  ├── commands/                     # 开发者工具脚本（PowerShell）
  │   ├── bump-version.ps1
  │   ├── generate-release.ps1
  │   ├── run-act-test.ps1          # ⚠️ 有 Invoke-Expression 注入风险
  │   └── run-unity-tests.ps1
  └── .claude/
      └── skills/                   # 2 个根级 skill（build-cli, github-pr-review-fix）
  ```
- **文件类型分布**：78 个 SKILL.md（Unity API 操作） / 2 个根级 SKILL.md / 3 个 CLAUDE.md context 文件 / 0 个 command（speckit 已被删除）
- **编排关系**：78 个 Unity SKILL.md 是平铺的独立 API 调用，用户通过自然语言触发，Claude 选择合适的 skill 调用对应 Unity MCP 工具
- **跨件契约**：SKILL.md 的 `name` 字段与 C# 侧 `[McpPluginTool]` attribute 的工具名对应，命名规则是 `category-action` kebab-case（如 `gameobject-create`、`scene-open`）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「每个 Unity API 对应一个 SKILL.md」—— 把 Unity Editor API 的调用方法、参数说明、注意事项封装成自然语言 skill，让 Claude 知道「如何正确调用这个 MCP 工具」
- **解决什么问题**：MCP 工具的 JSON schema 很精确但缺乏上下文——SKILL.md 补充了「什么时候用这个工具」「参数有哪些坑」「预期输出是什么」
- **做了什么 trade-off**：78 个独立 skill 让每个工具都有完整文档，但大量重复样板（每个 skill 都缺 model 声明、缺 allowed-tools）说明这些 skill 是从工具 schema 批量生成的，没有经过 NL 质量打磨
- **反映什么认知模型**：作者把 SKILL.md 看作 MCP 工具的「API 文档」而不是「行为规范」——重心在参数说明，轻行为约束

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「NL 表皮 + 原生二进制核心」**：自然语言层（SKILL.md）定义 AI 如何理解和调用工具，原生代码层（C# Unity 插件 + MCP server）执行真正的系统操作。

模式特征清单：
- 特征 1：SKILL.md 是人给 AI 看的文档，C# 是机器执行的代码——两层分开维护
- 特征 2：命名规则严格统一（category-action kebab-case），skill name = MCP tool name
- 特征 3：需要独立进程（MCP server）作为 AI 与系统之间的桥接层
- 特征 4：SKILL.md 的核心价值在于补充工具 schema 没有的「使用语境」
- 特征 5：运行时需要从 GitHub releases 下载二进制 server，而不是包管理器安装

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| IDE/编辑器控制（Unity/Blender/Figma）| ✅ 高度适用 | 需要系统级 API，原生二进制性能和权限优势明显 |
| 游戏开发自动化 | ✅ 适用 | 脚本和测试跑在同一个编辑器进程里 |
| 纯文本处理 / 数据分析 | ❌ 不适用 | 用 Python skill 就够，不需要 MCP server |
| 无法安装本地组件的环境（如 browser-only）| ❌ 不适用 | 依赖本地 .NET 运行时和 Unity Editor |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生核心（本仓库）| Unity-MCP | 系统级能力、性能优 | 安装复杂、下载二进制有安全风险 |
| 纯 SKILL.md + 脚本调用 | addyosmani/agent-skills | 安装简单、无额外进程 | 不能跨进程控制外部应用 |
| 云端 API wrapper | upstash/context7 | 无需本地安装、随时可用 | 网络依赖、API key 管理 |

### 2.4 改进空间
1. **当前问题**：运行时从 GitHub releases 下载 MCP server 二进制无 checksum 验证。**改进做法**：在 release 资产中发布 `.sha256` 文件，下载后用 `Get-FileHash` 或 `sha256sum` 验证后再执行。**预期收益**：防止中间人攻击或 release 篡改
2. **当前问题**：78 个 SKILL.md 批量生成，全部缺少 `model` 和 `allowed-tools`。**改进做法**：写一个脚本模板，批量为所有 skill 注入 `model: claude-haiku-4-5-20251001`（API 调用不需要大模型）和 `allowed-tools: [mcp__unity__<tool-name>]`。**预期收益**：NL 分数从 82 升到约 90，工具权限边界明确
3. **当前问题**：`commands/run-act-test.ps1` 用 `Invoke-Expression` 执行动态字符串。**改进做法**：改为 `& act @actArgs`（splatted array）。**预期收益**：消除命令注入攻击面

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-12 当时得分 **82/100**

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/commands/speckit.taskstoissues.md | 60/100 | 缺 `name` frontmatter，缺示例块 |
| .claude/commands/speckit.*.md（共9个）| 60–65/100 | 全部缺 `name`，全部缺 `allowed-tools` |
| Unity-MCP-Plugin/.claude/skills/editor-selection-get/SKILL.md | 76/100 | 6 个 bool 参数全部描述为空 |
| Unity-MCP-Plugin/.claude/skills/scene-set-active/SKILL.md | 80/100 | `sceneRef` 描述用了错误的通用 AssetObjectRef 模板文本 |
| 64 个 plugin skills | 83–85/100 | 全部缺 `model` 声明，全部缺 `allowed-tools` |
| .claude/skills/unity-skill-create/SKILL.md | 90/100 | 无问题——包含 C# 代码示例和模式文档，**当时最优文件** |

### 3.2 当时值得借鉴的模式
1. **unity-skill-create 的 C# 代码示例** → 在 SKILL.md 里直接嵌入代码示例，让 AI 知道「生成的代码应该长什么样」→ 比「参考代码风格」这种抽象描述有效得多 → 可借鉴到 echo-sleuth 的 skill（直接给一个好的 memory file 示例）
2. **tool name = skill name 统一命名** → `gameobject-create` skill 对应 MCP tool `gameobject_create`，零歧义 → 可借鉴到 echo-sleuth（skill name 应该和它实际调用的工具/行为完全对应）
3. **版本文件矩阵一致性检查** → `bump-version.ps1` 列出 7 个需要同步更新的版本文件路径 → 这种「版本一致性审计脚本」是个好实践
4. **speckit 工作流文件使用 `handoffs` 字段** → 即使在旧版中，命令间传递上下文的思路（handoffs）也是有价值的编排模式
5. **CLAUDE.md 分层（根 + 插件 + server）** → 每个子系统有自己的 CLAUDE.md，根级提供全局上下文 → 这种分层在 monorepo 里很必要

### 3.3 当时的缺陷
1. **问题**：9 个 speckit 命令全部缺 `name` 字段 → **根本原因**：frontmatter 中有 `description` 和 `handoffs` 但遗漏了 `name`，Claude Code 无法按名字注册这些命令 → **自查**：我的 echo-sleuth 的 agents 都有 `name` 字段吗？
2. **问题**：运行时下载二进制无完整性验证 → **根本原因**：McpServerManager.cs 只用 version 文件追踪版本，没有 checksum 对比 → **自查**：我有没有在 skill 里引导 AI 下载外部二进制或执行远程脚本？
3. **问题**：`run-act-test.ps1` 用 `Invoke-Expression` 执行动态字符串 → **根本原因**：开发者工具追求便捷，但 Invoke-Expression 会把字符串当 PowerShell 代码执行，.env 里的特殊字符可能触发意外命令 → **自查**：我的 hooks/scripts 有没有用字符串拼接然后 eval/exec？

### 3.4 当时的优化机会
1. 批量给 64 个 plugin skill 添加 `model` 和 `allowed-tools` 字段（脚本可生成）
2. 修复 `assets-shader-get-data`、`tool-list`、`editor-selection-get` 的参数描述（类型错误或描述为空）
3. 用 `& act @actArgs` 替换 `Invoke-Expression $actCommand`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 9 个 speckit 命令缺 `name` | `find .claude/commands -name "speckit*"` | **speckit 整体被删除** ✅（整个 `.claude/commands/` 里已无 speckit 文件）| 维护者选择删除有问题的 feature，而不是修复——这是一种「快速止损」策略 |
| 二进制下载无 checksum | `grep -r "checksum\|sha256" Unity-MCP-Plugin/` | **未修复**（仍无 checksum 验证逻辑）| 中等风险依然存在；维护者可能认为 GitHub release 本身提供足够安全保障 |
| Plugin skills 缺 model 声明 | `grep -L "^model:" Unity-MCP-Plugin/.claude/skills/*/SKILL.md` | **全部仍缺少**（78/78 个 skill 无 model 字段）| 批量生成的 skill 模板没有更新 |

### 4.2 架构演进
最重要的变化是 **speckit 被彻底删除**。过去 audit 时，仓库有 `.claude/commands/` 包含 9 个 speckit 命令（feature 规格管理工作流）。现在 `.claude/commands/` 目录已清空（只剩 `skills/` 子目录）。这表明维护者重新聚焦于核心定位——Unity MCP 控制——而不是扩展成通用项目管理工具。这是一个「减法」决策，值得尊重。

### 4.3 新增的可学习模式
speckit 被删除后，仓库变得更纯粹。这揭示了一个**「删除优于修复」的维护哲学**：当一个 feature 质量不达标且不在核心定位时，删掉比修 bug 更干净。这对我设计插件时的取舍有启发——如果某个 agent 的质量很差，不如先删掉，等设计成熟后再加回来。

---

## 五、校准

### 5.1 我已经在做对的
1. **NL 层与执行层分离**：echo-sleuth 的 SKILL.md 描述「如何处理记忆文件」，实际文件操作是 Claude 执行的工具调用——与 Unity-MCP 的 SKILL.md + C# 分层思路一致
2. **分层 CLAUDE.md**：我的 claude-for-legal 也为每个子插件写了独立 CLAUDE.md
3. **命名规则统一**：echo-sleuth 的 skill 都用 kebab-case slug（memory-management、git-mining）
4. **安全脚本意识**：echo-sleuth 没有使用 Invoke-Expression 或 eval 模式

### 5.2 挑战 / 验证
这次案例的最大认知冲击是**「删除也是一种正确决策」**。过去我总觉得 bug 要修，feature 要完善。Unity-MCP 的作者直接把有缺陷的 speckit 工作流整个删除，反而让项目更专注。这挑战了我「修复优于删除」的默认假设——对于非核心、低质量的组件，删除往往是更好的选择。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill/agent 是否有 name 字段
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude \
  -name "*.md" -path "*/skills/*" -o -name "*.md" -path "*/agents/*" \
  | xargs grep -L "^name:" 2>/dev/null

# 检查我的任何 skill 是否引导 AI 下载二进制或执行远程脚本（安全检查）
grep -rn "curl.*\|sh\|wget.*exec\|Invoke-Expression\|eval\b" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ 2>/dev/null

# 检查我的 plugin skills 是否有 model 声明
grep -rL "^model:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null
```
命中后怎么办：
- 缺 `name`：在 frontmatter 加 `name: <skill-slug>`
- 安全模式匹配：审查上下文，确认是否真有注入风险
- 缺 `model`：加 `model: claude-haiku-4-5-20251001`（轻量 skill 不需要大模型）

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 所有 skill 批量添加 `model` 声明
   - **为何可行**：echo-sleuth 的任务（内存审计、git 挖掘）都是中等复杂度，Haiku 就够
   - **第一步**：在 `skills/*/SKILL.md` frontmatter 第二行加 `model: claude-haiku-4-5-20251001`，4 个文件，约 5 分钟
2. **想法**：review echo-sleuth 的功能范围，删除质量差的组件而不是勉强维护
   - **为何可行**：echo-sleuth 有 5 个 agents，其中 schema-scout 可能用途较窄
   - **第一步**：评估每个 agent 的使用频率，对低价值 agent 写删除 PR，约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 IvanMurzak/Unity-MCP 的核心目的**：让 AI 直接控制 Unity 编辑器，MCP 工具 + SKILL.md 提供操作接口
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是 skill 形式 + 实际工具调用，有 plugin.json | echo-sleuth 操作的是 Claude 记忆文件（文本），Unity-MCP 操作的是 Unity API（系统级）| **高**（skill 结构可借鉴）|
| MarkQWu/claude-for-legal | 低 | 都有 agent + context 文件 | 法律 plugin 无 MCP server，只有 markdown 引导 | 低 |
| MarkQWu/drama-workshop-skills | 无 | — | 完全不同领域 | 无 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 `model` 声明 | `grep -rL "^model:" skills/*/SKILL.md` | echo-sleuth：4/4 skills 命中 | **中** |
| 参数描述用通用模板文本（非具体描述）| 人工审查 skill 参数说明 | echo-sleuth memory-management 的 `type` 参数描述不够具体 | **低** |
| 缺 `allowed-tools` | `grep -rL "allowed-tools" skills/*/SKILL.md` | echo-sleuth：4/4 全部命中 | **中** |

**命中后的具体行动建议**：
- `echo-sleuth/skills/memory-management/SKILL.md` → 添加 `model: claude-haiku-4-5-20251001` → 约 1 分钟
- `echo-sleuth/skills/git-mining/SKILL.md` → 检查参数描述是否具体（如 `since` 参数应写「ISO 8601 date string, e.g. 2026-01-01」而非「date string」）

### 7.3 别人的更优方案

1. **领域**：SKILL.md 与 MCP 工具 name 强绑定
   - **本案例做法**：`gameobject-create/SKILL.md` 的 `name: gameobject-create` 与 C# 侧 `[McpPluginTool("gameobject-create")]` 完全对应，零歧义
   - **我的项目现状**：echo-sleuth 的 skill name（`memory-management`）与它实际调用的 bash 工具之间没有显式绑定声明
   - **如何借鉴**：在 echo-sleuth skill frontmatter 加 `allowed-tools: [bash, read]` 说明这个 skill 实际用的工具

2. **领域**：代码示例嵌入 SKILL.md
   - **本案例做法**：`unity-skill-create/SKILL.md` 包含完整的 C# 代码示例，展示「生成的代码应该长什么样」
   - **我的项目现状**：echo-sleuth 无任何 `## Examples` 块
   - **如何借鉴**：给 `memory-management/SKILL.md` 添加一个完整示例：input（「audit memory files」）→ output（标注 stale 的文件列表 + 数学公式计算过程）

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：skill 内容的实质性文档质量
- **我的做法**：echo-sleuth 的 `memory-management/SKILL.md` 有完整的 Staleness Scoring 公式、claim extraction 规则、mutation 约束——是真正的行为规范，不只是参数列表
- **本案例做法**：Unity-MCP 的大多数 plugin skill 是从工具 schema 机械生成的，缺乏使用语境和约束描述
- **意义**：内容深度是 echo-sleuth 的优势，这正是被外部 PR 参考的切入点

---

## 八、术语表

### <a name="mcp-server"></a>MCP server
> Model Context Protocol 服务端，运行在本地的一个进程，负责把 AI 的「函数调用」翻译成系统操作（如 Unity API 调用）。Claude Code 通过 JSON-RPC 协议与 MCP server 通信。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种架构模式：自然语言文件（SKILL.md）告诉 AI 如何理解和使用工具，实际执行由原生代码（C#、Go、Rust 等编译型程序）完成。「NL 表皮」处理意图解析，「二进制核心」处理系统操作。

### <a name="checksum"></a>checksum
> 文件的「指纹」，通常是 SHA-256 哈希值。下载一个文件后，计算其 checksum 并与发布者公布的值对比，能验证文件没有被篡改或损坏。

### <a name="invoke-expression"></a>Invoke-Expression
> PowerShell 的命令，把一个字符串当作 PowerShell 代码执行（等价于 eval）。如果字符串里混入了用户输入或外部数据，可能执行意外代码——这是命令注入的经典来源。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 YAML 配置块，声明 name、description、model 等元数据，Claude Code 先读 frontmatter 才知道这个 skill 叫什么、什么时候用。
