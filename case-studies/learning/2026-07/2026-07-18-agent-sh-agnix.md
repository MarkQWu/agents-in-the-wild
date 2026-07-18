# agent-sh/agnix — 学习案例

**仓库**：https://github.com/agent-sh/agnix
**Stars**：207 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-18（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `security-gate`, `template-design`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
agnix 是「AI 编程助手配置文件的缺失 linter 与 [LSP](#lsp)」——用 Rust 编写的验证工具，检查 Claude Code、GitHub Copilot、Cursor、Codex CLI、OpenCode 等 10+ 工具的 SKILL.md、hooks、CLAUDE.md、MCP 配置、agent 定义是否合规。207 stars，由 agent-sh 团队维护，跨平台分发（macOS/Linux/Windows）。提供四种接入方式：CLI 工具、[LSP](#lsp)（编辑器实时高亮）、[MCP 服务器](#mcp)（供 Claude Code 调用）、[WASM](#wasm) 模块（供浏览器/CI 使用）。agnix 既是「被审计对象」也是「审计工具」，其 [NL 表皮](#nl-binary-hybrid)（plugin/、skills/）包裹了 Rust 核心，是 nl-binary-hybrid 模式的代表案例。

### 1.2 架构剖析
- **目录结构**：
  ```
  agent-sh/agnix/
  ├── crates/                    # Rust workspace
  │   ├── agnix-rules/           # 规则数据层（rules.json 源）
  │   ├── agnix-core/            # 核心验证逻辑库
  │   ├── agnix-cli/             # CLI 二进制
  │   ├── agnix-lsp/             # LSP 二进制
  │   ├── agnix-mcp/             # MCP 服务器
  │   └── agnix-wasm/            # WASM 绑定
  ├── knowledge-base/
  │   └── rules.json             # 437 条规则的机器可读源
  ├── VALIDATION-RULES.md        # 规则的人类可读文档
  ├── plugin/                    # Claude Code 插件 NL 表皮
  │   ├── .claude-plugin/plugin.json
  │   ├── commands/agnix.md
  │   └── skills/agnix/SKILL.md
  ├── skills/agnix/SKILL.md      # 不经插件的直接 skill 路径
  ├── tests/fixtures/            # ~80% 的 NL artifacts 是测试夹具
  │   ├── valid/                 # 合规示例夹具
  │   ├── invalid/               # 故意违规的夹具
  │   └── copilot-invalid/
  ├── editors/
  │   ├── vscode/                # VSCode 扩展
  │   ├── neovim/
  │   └── jetbrains/
  ├── npm/                       # npm 分发包
  └── scripts/                   # 版本同步等维护脚本
  ```
- **文件类型分布**：85 个 NL artifacts（含约 68 个测试夹具）/ 6 个 Rust crate / 1 个 plugin.json / 2 个 SKILL.md 路径（plugin/ 和 skills/）
- **编排关系**：Rust 核心（agnix-core）负责实际验证逻辑，NL 表皮（plugin/commands/agnix.md）是用户接口，两者通过二进制调用连接；`scripts/sync-versions.sh` 等脚本确保 plugin.json 和 npm/package.json 版本同步
- **跨件契约**：`knowledge-base/rules.json` 是规则的单一真相源（[Single Source of Truth](#sot)），CI 强制检验 rules.json 与 VALIDATION-RULES.md 的同步性；plugin/skills/agnix/SKILL.md 和 skills/agnix/SKILL.md 是两条分发路径，规则应互为镜像

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「在 NL 工件层引入工程严谨性」——像给代码写 linter 一样给 AI 配置文件写 linter，将隐性规范（frontmatter 必须有 name）变成可机器验证的规则（437 条）
- **解决什么问题**：AI 编程助手的配置文件（SKILL.md、hooks、CLAUDE.md）缺乏标准验证，导致「写了但无效」「引用了不存在的工具」等静默错误
- **做了什么 trade-off**：选择 Rust 实现核心（性能 + 跨平台一致性）而非纯 NL 实现（更简单但更慢、精度低）；选择独立仓库而非嵌入现有工具（可独立版本管理）；选择 tests/fixtures/ 内置测试夹具（可验证性高，但导致 NLPM 扫描时夹具文件得分拉低总分）
- **反映什么认知模型**：作者把 AI 配置文件视为「有语法的代码」，而非自由文本——这是一个更接近「静态分析工具」的认知框架，而非「提示词调优」框架

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」（NL Façade + Native Binary Core）**：NL 层（SKILL.md、commands/agnix.md）处理用户意图的自然语言接收，Rust 二进制处理实际的结构化验证逻辑。

模式特征清单：
- NL 层是薄接口：只负责解析用户自然语言请求、调用二进制、格式化输出
- 核心逻辑在编译型代码里：性能敏感或精度要求高的验证跑 Rust，不跑模型
- 规则以数据形式持久化（rules.json），而非嵌入 NL 文本
- 多端分发：同一核心通过 CLI/LSP/MCP/WASM 不同形态部署
- tests/fixtures/ 验证核心逻辑，不依赖 LLM

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要精确、高速验证的工具（linter、schema check） | ✅ 高度适用 | Rust 核心比 LLM 判断快 100× 以上，结果确定性高 |
| 跨平台 CLI 工具 | ✅ 高度适用 | Rust 编译产物可交叉编译，单二进制无依赖 |
| 纯内容生成型 skill（写作、总结） | ❌ 不适用 | LLM 本身就是最佳工具，加二进制核心是过度工程 |
| 快速原型或个人脚本 | ❌ 不适用 | Rust 工具链建设成本高，不适合一次性使用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制核心（本仓库） | agent-sh/agnix | 性能确定性高、规则精确、跨平台 | 技术门槛高，Rust + NL 双重维护成本 |
| 纯 NL 实现 | aaronmaturen/claude-plugin | 上手成本极低，无编译步骤 | 精确度受 LLM 影响，速度慢 |
| Python 纯 stdlib 实现 | markqwu/agents-in-the-wild bin/nlpm-check | 无依赖安装，跨平台，速度合理 | 比 Rust 慢，但对大多数 lint 场景够用 |

### 2.4 改进空间
1. **当前问题**：plugin/skills/agnix/SKILL.md 有 argument-hint 但缺 allowed-tools；skills/agnix/SKILL.md 有 allowed-tools 但缺 argument-hint——两条路径互补而不完整 **改进做法**：合并两个文件的优点，统一到 plugin/ 路径，在 plugin/skills/agnix/SKILL.md 同时声明 allowed-tools 和 argument-hint **预期收益**：消除两条分发路径的功能不对等
2. **当前问题**：postinstall 脚本下载二进制但不做 checksum 验证 **改进做法**：在 npm/install.js 里下载 .sha256 sidecar 文件，用 `crypto.createHash('sha256')` 验证后再 chmod +x **预期收益**：降低供应链攻击面

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **64/100**（均值，含夹具文件拉低）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| tests/fixtures/\*/missing-*.md (多个) | 20 | 故意缺 frontmatter（测试夹具，设计如此）|
| tests/fixtures/valid/skills/\*/SKILL.md | 70 | 无 model(-5)、无 output format(-10)、无 examples(-15）|
| plugin/agents/agnix-agent.md | 85 | 无 examples(-15) |
| skills/agnix/SKILL.md | 90 | 无 model(-5)，examples 不完整(-5) |
| plugin/commands/agnix.md | 95 | 规则数量陈旧（"405" vs 实际 414）|

### 3.2 当时值得借鉴的模式
1. **规则机器可读化**（rules.json 单一真相源）→ CI 强制 rules.json 与 VALIDATION-RULES.md 同步 → `knowledge-base/rules.json` → 借鉴：将规则/标准从 NL 文本提取为结构化数据，测试验证其完整性
2. **测试夹具作为验证锚点** → invalid/ 里故意违规的文件确保 linter 能检测到对应错误 → `tests/fixtures/invalid/` → 借鉴：NL 工件也可以有「反例单元测试」
3. **多路径分发** → plugin/ 和 skills/ 两条路径覆盖「通过插件安装」和「直接引用」两种使用场景 → 借鉴：设计时考虑安装路径多样性

### 3.3 当时的缺陷
1. **版本漂移（plugin.json 0.16.1 vs npm/package.json 0.20.0）** → 为什么失败：两个清单来自不同的更新流程，没有单一同步脚本绑定两者；版本不一致让工具链无法确定「哪个版本是准的」。自查：**我的 bureau 有相同问题**（plugin.json 0.5.2 vs package.json 0.0.1）
2. **规则数量陈旧（命令和 skill 里写"405 rules"，实际 414 条）** → 为什么失败：NL 文件里的数字是硬编码的，没有动态引用 rules.json 里的计数；每次加新规则都需要手动更新描述文本，容易漏。自查：我的 nlpm 里有类似的「描述里的数字」需要检查
3. **postinstall 无 checksum 验证** → 为什么失败：下载二进制后直接 chmod +x 执行，没有校验哈希；被攻击时用户无感知。自查：我的仓库目前无 postinstall 脚本，暂无此风险

### 3.4 当时的优化机会
1. 将 plugin/skills/agnix/SKILL.md 和 skills/agnix/SKILL.md 的优点合并到一个文件
2. 在 npm/install.js 里加 SHA-256 checksum 验证
3. 为 plugin/agents/agnix-agent.md 添加 examples block

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 版本漂移（0.16.1 vs 0.20.0） | `cat plugin/.claude-plugin/plugin.json \| python3 -c "...print(d['version'])"` 和 `cat npm/package.json` | **已修复**：两者均为 0.40.0 | `scripts/sync-versions.sh` 生效了；版本同步脚本被纳入发布流程 |
| 规则数量陈旧（"405 rules"） | `grep "rules" plugin/commands/agnix.md \| head -3` | **已修复**：现在写 "437 rules"，与实际规则数一致 | 发布流程中加入了规则数同步步骤 |
| plugin/skills 缺 allowed-tools | `grep "allowed-tools" plugin/skills/agnix/SKILL.md` | **未修复**：plugin/skills/agnix/SKILL.md 仍无 allowed-tools，root skills/agnix/SKILL.md 有 | 分裂状态持续存在 |

### 4.2 架构演进
从 Audit 时（v0.20.0）到当前 HEAD（v0.40.0），agnix 增加了：editors/ 子目录（VSCode + Neovim + JetBrains 编辑器支持）、Formula/ 目录（Homebrew 公式）、更完整的 per-client 测试夹具（`.roo/`、`.kiro/`、`.opencode/` 等新 AI 工具格式）。规则数从 414 增长到 437。这说明作者重心在「扩展覆盖的 AI 工具类型」而非「改善 NL 表皮质量」。

### 4.3 新增的可学习模式
- **per-client 测试夹具**：`tests/fixtures/per_client_skills/` 目录按工具（cline、cursor、kiro、opencode、windsurf）分组，测试每种工具支持的 frontmatter 字段集合。这是「跨工具兼容性测试矩阵」模式的具体实现，在 Audit 时该目录还不完整，现在扩展到 8 种工具。

---

## 五、校准

### 5.1 我已经在做对的
1. **单一真相源原则**：我的 nlpm 里 `auditor/SCHEMAS.md` 是 schema 定义的真相源，与 agnix 的 rules.json 异曲同工
2. **guard-protected-paths.sh**：保护关键目录不被自动化脚本误改，类似 agnix 的 CI parity tests
3. **二进制 + NL 混合设计意识**：我的 bin/nlpm-check（纯 Python stdlib）与 agnix 的思路相同——NL 层用于复杂推理，确定性验证用非 LLM 代码实现

### 5.2 挑战 / 验证
- **挑战**：测试夹具的计分问题——agnix 的 64/100 分大部分是测试夹具拖累的，而非生产代码质量低。这说明 NLPM score 在包含「故意错误的测试文件」的仓库里会失真。我的 .nlpm-test/ 也是测试文件，未来评分时需要考虑排除测试目录的配置
- **验证**：「规则数要动态引用而非硬编码」——agnix 用脚本同步，而不是手动改文字。验证了我之前的直觉：描述里的数字一旦硬编码就会腐化

---

## 六、行动

### 6.1 自查动作

```bash
# 检查版本一致性（仿 agnix 修复的那个 bug）
for repo in MarkQWu-gstack MarkQWu-bureau; do
  plugin_ver=$(find /tmp/my-repos/$repo -name "plugin.json" -not -path "*/node_modules/*" | \
    xargs python3 -c "import json,sys; d=json.load(open(sys.argv[1])); print(d.get('version','N/A'))" 2>/dev/null)
  pkg_ver=$(find /tmp/my-repos/$repo -name "package.json" -not -path "*/node_modules/*" | head -1 | \
    xargs python3 -c "import json,sys; d=json.load(open(sys.argv[1])); print(d.get('version','N/A'))" 2>/dev/null)
  echo "$repo: plugin=$plugin_ver pkg=$pkg_ver"
done
# 命中后（版本不一致）：写一个 sync-versions.sh 或把版本统一到同一来源

# 检查 NL 描述里的硬编码数字
grep -rn "[0-9]\+ rules\|[0-9]\+ skills\|[0-9]\+ commands" ~/.claude/skills/ 2>/dev/null
# 命中后：改为动态统计或备注「需随版本更新」
```

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau 写一个版本同步脚本（仿 agnix/scripts/sync-versions.sh）
   - **为何可行**：bureau 已有 plugin.json 和 package.json 版本不一致的问题（0.5.2 vs 0.0.1）
   - **第一步**：在 bureau/scripts/ 新建 sync-versions.sh，读取 package.json 的 version，写入 plugin.json，15 分钟完成

2. **想法**：给 nlpm 的 NLPM score 计算添加「排除测试目录」选项
   - **为何可行**：agnix 案例证明测试夹具会严重拉低分数，这是一个已知失真来源
   - **第一步**：在 `.claude/nlpm.local.md` 里研究是否支持 exclude 规则，或在评分时手动标注「含测试夹具」

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 agent-sh/agnix 的核心目的**：为 AI 编程助手配置文件提供静态 lint 验证（Rust 核心 + NL 表皮）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/agents-in-the-wild | 中 | 都有 NL 工件验证逻辑（bin/nlpm-check）| agents-in-the-wild 是 Python stdlib 实现，agnix 是 Rust；规模相差悬殊 | 高 |
| MarkQWu/gstack | 低 | 都有 NL 工件 | gstack 是使用工具的仓库，agnix 是验证工具的工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 版本漂移（plugin.json vs package.json） | `python3 -c "..."` 对比两文件 | **命中**：bureau plugin.json=0.5.2，package.json=0.0.1，不一致 | 中 |
| plugin skill 和 root skill 字段互补不完整 | `grep "allowed-tools\|argument-hint" /skills/*/SKILL.md` | 未在 my-repos 中发现此模式 | 低 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/.claude-plugin/plugin.json` 和 `MarkQWu-bureau/package.json` → 统一版本号到同一值 → 5 分钟完成

### 8.3 别人的更优方案

1. **领域**：版本同步自动化
   - **本案例做法**：`scripts/sync-versions.sh` 在发布时自动同步 plugin.json 和 npm/package.json
   - **我的项目现状**：bureau 手动维护两个版本号，已出现漂移
   - **如何借鉴**：在 bureau 发布流程里加版本同步步骤（可以是 1 行 sed 命令）

2. **领域**：测试夹具覆盖 per-client 工具兼容性
   - **本案例做法**：`tests/fixtures/per_client_skills/` 按工具分组，覆盖 8 种 AI 工具
   - **我的项目现状**：agents-in-the-wild 的 .nlpm-test/ 只测试 Claude Code 场景
   - **如何借鉴**：考虑添加针对 Codex CLI、Cursor 等工具的测试场景

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：测试与生产文件分离的评分意识
- **我的做法**（agents-in-the-wild）：.nlpm-test/ 专门存放测试 spec，不与生产 NL artifacts 混在同一目录树
- **本案例做法**（agnix）：tests/fixtures/ 混入大量故意违规文件，导致 NLPM score 64/100 严重失真
- **意义**：保持测试文件与生产文件的目录隔离，是评分公正性的前提

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种架构模式：外层用 Markdown 文件（SKILL.md、command.md）处理用户的自然语言请求，内层调用一个用 C/Go/Rust 等编译型语言写成的可执行文件做真正的计算。「NL 表皮」薄而灵活，「二进制核心」快而精确。agnix 的 agnix-cli、agnix-lsp 都是这个内层核心。

### <a name="lsp"></a>LSP
> Language Server Protocol（语言服务协议）。一种标准通信协议，让编辑器（VS Code、Neovim 等）与后端语言服务器交互，实现实时的代码高亮、错误提示、自动补全。agnix-lsp 让编辑器在你写 SKILL.md 时实时显示验证错误。

### <a name="mcp"></a>MCP 服务器
> Model Context Protocol Server，Claude Code 可以调用的外部服务，为模型提供额外工具能力。agnix-mcp 让 Claude Code 在会话中直接调用 agnix 的验证功能。

### <a name="wasm"></a>WASM
> WebAssembly，一种可在浏览器和 Node.js 里运行的低级字节码格式，由 Rust/C/C++ 代码编译而来。agnix-wasm 让 agnix 的验证逻辑在浏览器端或 CI 环境中运行，无需安装本地二进制。

### <a name="sot"></a>Single Source of Truth（单一真相源）
> 数据只在一个地方定义，其他地方引用这个来源，而不是分别维护多份拷贝。agnix 的 rules.json 是规则的单一真相源：其他文件（VALIDATION-RULES.md、skill 描述）通过脚本从 rules.json 生成，而不是手动同步。
