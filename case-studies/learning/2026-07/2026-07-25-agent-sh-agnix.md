# agent-sh/agnix — 学习案例

**仓库**：https://github.com/agent-sh/agnix
**Stars**：207 | **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-25（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `vague-quantifier`, `security-gate`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`agnix` 是一个面向多 AI 工具（Claude Code、GitHub Copilot、Cursor、Zed、Kiro 等 10+ 工具）的 NL [配置文件](#配置文件) 验证器。它以[原生二进制](#原生二进制)（Rust 编写）为核心，同时提供 Claude Code 插件接口（plugin.json + skills）作为 AI 可调用的外壳。

关键事实：
- 工具定位：NL 配置 lint 工具，类比代码世界的 ESLint/Prettier
- 当前版本 v0.40.0，拥有 437 条验证规则
- 多编辑器支持：VS Code 扩展、JetBrains 插件、Neovim、Zed
- 核心架构：NL 表皮（skill/command 描述规则）+ Rust 原生二进制执行引擎

### 1.2 架构剖析
**目录结构**：
```
agnix/
├── plugin/                    # Claude Code 插件入口
│   ├── .claude-plugin/plugin.json
│   ├── commands/agnix.md
│   ├── skills/agnix/SKILL.md
│   └── agents/agnix-agent.md
├── skills/agnix/SKILL.md     # 根路径 skill（独立安装路径）
├── tests/fixtures/            # 80% 文件是测试夹具（故意包含缺陷的样例）
│   ├── valid/                 # 合规样例
│   ├── invalid/               # 故意违规样例
│   ├── copilot/               # Copilot 格式样例
│   ├── copilot-invalid/       # Copilot 故意违规样例
│   ├── cross_platform/        # 跨工具兼容性测试
│   └── per_client_skills/     # 每个 AI 工具的特有字段测试
├── crates/agnix-rules/rules.json  # 规则数据库
├── npm/                       # npm 包（含自动安装脚本）
├── editors/                   # 编辑器集成（vscode/jetbrains/neovim/zed）
└── scripts/                   # 维护脚本
```

**文件类型分布**：2 个 skill（plugin + root）/ 1 个 command / 1 个 agent / 大量测试夹具

**编排关系**：Claude Code 调用 `agnix.md` 命令 → 触发 `agnix` 二进制 → 输出 lint 结果 → Claude 解析并提供修复建议。Skill 和 agent 是可选的上层界面。

**跨件契约**：plugin/ 路径的 skill 和 root skills/ 路径的 skill 内容几乎重复，各有互补缺失（历史上一个有 allowed-tools 但无 argument-hint，另一个相反）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「规则即文档」——437 条规则本身就是完整的规范文档，`rules.json` 是单一真实来源
- **解决什么问题**：AI 工具的 NL 配置文件（SKILL.md, agent.md, hooks.json）缺乏统一的 lint/验证标准，质量良莠不齐
- **Trade-off**：选择原生二进制（Rust）而非纯 NL 实现，获得性能和确定性，但增加了分发复杂度（npm install 需要下载二进制）
- **认知模型**：把 AI agent 配置文件当作「代码」来对待，需要静态分析工具检查质量

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：「NL 表皮 + [原生二进制核心](#原生二进制)」

核心特征：
- 特征 1：Rust/Go/C++ 等编译型语言实现核心逻辑，性能高、行为确定
- 特征 2：NL 层（skill/command）只做调用桥接，不包含业务规则
- 特征 3：[manifest](#manifest) 文件（plugin.json）严格映射到实际二进制工具
- 特征 4：测试夹具文件与生产文件明确分离，避免 linter 误判

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要高速处理大量文件 | ✅ 高度适用 | 二进制比脚本快 10-100× |
| 跨平台分发（Windows/Mac/Linux） | ✅ 适用 | Rust 跨编译成熟 |
| 纯 NL 任务（写作、分析） | ❌ 不适用 | 过度工程，直接用 SKILL.md 即可 |
| 个人快速原型 | ❌ 不适用 | 二进制需要构建 CI，维护成本高 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生二进制（本仓库） | agnix, 777genius/claude-notifications-go | 性能优、行为确定 | 分发复杂，需要 CI/CD 自动化 |
| 纯 NL skill | mattpocock/skills, addyosmani/agent-skills | 零构建步骤 | 依赖 LLM 解释，行为不确定 |
| NL + Python/JS 脚本 | wecode-ai/Wegent | 开发快、调试容易 | 需要运行时（pip/node），安装体验差 |

### 2.4 改进空间
1. **当前问题**：npm postinstall 下载二进制无 SHA256 校验 **改进做法**：在 package.json 中硬编码每个版本的 sha256 哈希，install.js 下载后验证 **预期收益**：消除高风险安全发现，符合 npm 最佳实践
2. **当前问题**：plugin/ 和 root skills/ 两个 skill 文件各有互补缺失字段 **改进做法**：统一合并到一个位置，或用脚本从单一源同步两者 **预期收益**：消除双路径维护的版本漂移风险

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
2026-04-06 当时得分 **64/100**（注：大量测试夹具文件拉低了整体均分）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| plugin/.claude-plugin/plugin.json | 100 | 干净 |
| plugin/commands/agnix.md | 95 | 规则数量陈旧（405 vs 实际 414） |
| plugin/skills/agnix/SKILL.md | 90 | 无 model (-5)、示例部分 (-5) |
| plugin/agents/agnix-agent.md | 85 | 无示例 (-15) |
| 测试夹具文件（80% 体量） | 20-70 | 故意违规，设计如此 |

### 3.2 当时值得借鉴的模式
1. **测试夹具作为规范文档** → 通过「故意错误」的样例文件精确定义什么是「错误」→ `tests/fixtures/invalid/` 中每个文件名就是错误类型名 → 可借鉴到自己的 validation 工具
2. **双路径分发（plugin/ + skills/）** → 支持「完整插件安装」和「只安装 skill」两种使用路径 → 增加了分发灵活性
3. **规则计数同步脚本** → `scripts/sync-rule-bookkeeping.js` 自动同步 skill 描述中的规则计数 → 「数字作为断言」的概念值得借鉴

### 3.3 当时的缺陷
1. **plugin.json 版本（0.16.1）落后于 npm 版本（0.20.0）四个小版本**：根本原因：两个 [manifest](#manifest) 文件手动维护，没有自动化同步机制。自查：我是否有多个版本文件需要同步？
2. **规则数量在三个文件中均陈旧（405 vs 实际 414）**：根本原因：规则新增后忘记更新描述文本；`sync-rule-bookkeeping.js` 脚本存在但未在 CI 中强制执行。这是「数字作为文档」的维护陷阱。
3. **HIGH 安全风险：npm postinstall 无校验直接执行下载的二进制**：根本原因：快速迭代的项目常见问题，验证逻辑往往被视为「以后再加」。

### 3.4 当时的优化机会
1. 在 CI 中强制运行 `sync-rule-bookkeeping.js` 确保规则数量始终准确
2. 为 npm 二进制下载添加 SHA256 校验
3. 统一 plugin/skills 和 root/skills 的字段配置

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Bug #1：plugin.json 版本落后 | grep version plugin/.claude-plugin/plugin.json | v0.40.0，与 npm 版本一致 → **已修复** | 版本同步问题解决 |
| Bug #2-4：规则数 405（陈旧） | grep "rules" plugin/commands/agnix.md | 现在显示「437 rules」→ **已更新** | 数量同步，但 rule count 需持续维护 |
| Bug #5：plugin skill 缺 allowed-tools | grep "allowed-tools" plugin/skills/agnix/SKILL.md | 现有 `allowed-tools: Bash(agnix:*), Bash(cargo:*), Read, Glob, Grep` → **已修复** | 两个 skill 路径现在均有 allowed-tools |

**5 个生产 bug 全部修复**（参见我们 2026-04-26 提交的 PR #828，最终 applied_separately 方式修复）。

### 4.2 架构演进
从 v0.20.0 → v0.40.0，有重大扩展：
- 规则数从 414 增加到 437（新增 23 条规则）
- 新增编辑器集成：Neovim、Zed（之前只有 VS Code、JetBrains）
- `schemas/` 目录新增 JSON Schema 文件，用于编辑器 schema 验证
- 新增跨平台测试夹具（`tests/fixtures/cross_platform/`、`tests/fixtures/per_client_skills/`）
- 安全文档（`SECURITY.md`）新增

这说明作者已将 agnix 定位从「Claude Code 专属 linter」升级为「通用 AI 工具配置 linter」。

### 4.3 新增的可学习模式
- **按客户端划分测试夹具**（`per_client_skills/.cline/`、`per_client_skills/.cursor/` 等）：每个 AI 工具有自己的 skill 格式差异，用文件夹名称编码这种差异，极大提升了测试的可读性
- **JSON Schema 校验**（`schemas/agnix.json`）：提供机器可读的 schema，让编辑器可以在用户编写配置文件时实时提示错误

---

## 五、校准

### 5.1 我已经在做对的
1. **测试夹具分离**：我的 agents-in-the-wild 仓库也将测试规范（.nlpm-test/）与生产代码分离
2. **规则即文档**：我的 skills/nlpm/rules/ 目录用文件名编码规则，类似 agnix 的 rules.json 思路
3. **多路径 skill 分发**：我的 nlpm 插件也支持从 marketplace 和直接路径两种安装方式

### 5.2 挑战 / 验证
这个案例验证了「**规则数量是文档的一部分，必须受到版本控制**」。agnix 花了大量时间在 CI 中强制同步 rule count——这个教训也适用于 NLPM 自身：每次添加规则时，scoring 文档中的总数是否需要自动更新？目前是手动维护的，这个案例提示了一个潜在的技术债。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 版本字段是否与 plugin.json 同步
cat /tmp/my-repos/MarkQWu-bureau/.claude-plugin/plugin.json | python3 -c "import sys,json;d=json.load(sys.stdin);print('version:', d.get('version'))"
grep "version:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null | head -5
```

命中后：确认两处版本号一致，不一致则修改 plugin.json。

```bash
# 检查我的 skills 中是否有数字断言（「N 条规则」「M 个功能」等易过时的数字）
grep -rn -E '[0-9]+ (rules|commands|skills|agents|steps)' /tmp/my-repos/MarkQWu-bureau/ 2>/dev/null
```

命中后：用相对表述替代，或加入 CI 检查自动更新。

### 6.2 灵感 → 实施路径
1. **想法**：为 bureau 建立一个最小版本的「测试夹具」目录，放置「故意违规」的 SKILL.md 样例 **为何可行**：有助于测试 nlpm:score 在我的 bureau 上的行为 **第一步**：新建 `bureau/tests/fixtures/invalid/missing-model/SKILL.md`，只含 `name:` 字段，用于验证 score 工具能检测缺失 model
2. **想法**：给 gstack 的 skills 添加 JSON Schema 校验支持 **为何可行**：gstack 已有 name 字段但没有自动化校验 **第一步**：在 gstack 根目录加 `.vscode/settings.json` 指向 agnix 提供的 JSON Schema

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 agent-sh/agnix 的核心目的**：验证 AI 工具 NL 配置文件的质量，类似于代码 linter

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/agents-in-the-wild（即本仓库） | 高 | 都是 NL 质量工具 | agnix 是执行型验证，nlpm 是 AI 评分 | 高 |
| MarkQWu/bureau | 低 | 都是 Claude Code 插件 | 目的完全不同 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skills 无 model 声明 | `grep -rL "^model:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | **全部 7 个命中** | 中 |
| 规则数量陈旧（文档中有具体数字） | `grep -rn "[0-9]+ rules" /tmp/my-repos/` | 暂无发现 | 无 |
| npm 依赖未固定版本 | `grep "\^" /tmp/my-repos/MarkQWu-bureau/package.json 2>/dev/null` | 无 package.json，不适用 | 无 |

**命中后的具体行动建议**：
- `MarkQWu/bureau/skills/capture/SKILL.md` 等 7 个文件 → 在 frontmatter 加 `model: claude-haiku-4-5-20251001` 或 `claude-sonnet-4-5` → 每个文件 5 分钟，共 35 分钟

### 8.3 别人的更优方案

1. **领域**：NL 表皮 + 原生二进制双层架构
   - **本案例做法**：Rust 二进制执行实际验证，NL skill 只做 prompt 调用，两者职责完全分离
   - **我的项目现状**：gstack 的所有 skills 都是纯 NL，没有可执行后端
   - **如何借鉴**：对于 gstack 中需要精确执行的场景（如 benchmark），可以抽取一个 Python 脚本作为后端，skill 只做调用桥接

2. **领域**：测试夹具 = 规范文档
   - **本案例做法**：每个「错误类型」都有一个故意违规的夹具文件，文件名即错误类型
   - **我的项目现状**：bureau 没有测试夹具
   - **如何借鉴**：在 bureau 新建 `tests/fixtures/` 目录，按错误类型命名夹具文件

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：单一 skill 文件（无双路径维护问题）
- **我的做法**：bureau/skills/ 每个 skill 只有一个 SKILL.md，无 plugin/ vs root/ 双路径分歧
- **本案例做法**（弱在哪）：`plugin/skills/agnix/SKILL.md` 和 `skills/agnix/SKILL.md` 历史上各有互补缺失字段，增加了维护负担
- **意义**：单一真实来源（Single Source of Truth）是优势，避免了版本漂移

---

## 八、术语表

### <a name="原生二进制"></a>原生二进制
> 用 Rust / Go / C++ 这种「编译型语言」写出来、最终变成可执行文件的程序。不依赖额外的解释器（如 Python、Node.js）就能直接运行，性能高一个数量级。agnix 的核心验证逻辑就是一个 Rust 二进制，安装时 npm 会下载对应平台的预编译版本。

### <a name="配置文件"></a>配置文件
> 在这里特指 AI 工具（Claude Code、Cursor、GitHub Copilot 等）读取的 Markdown 格式配置文件，如 `SKILL.md`、`agent.md`、`hooks.json`。这些文件告诉 AI 如何行动，agnix 的任务就是验证这些文件格式是否正确。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。agnix 的 `plugin.json` 列出了所有 commands、skills、agents。npm 的 `package.json` 则是 npm 世界的 manifest，描述包名、版本、依赖。
