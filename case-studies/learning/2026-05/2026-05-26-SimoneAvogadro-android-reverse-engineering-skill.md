# SimoneAvogadro/android-reverse-engineering-skill — 学习案例

**仓库**：https://github.com/SimoneAvogadro/android-reverse-engineering-skill
**Stars**：5485 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-26（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `examples-driven`, `manifest-discipline`, `offline-capable`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
SimoneAvogadro/android-reverse-engineering-skill 是一个专注于 [Android](#android) APK 逆向工程的深度领域插件。用户安装后可以让 Claude Code 反编译 APK/XAPK/JAR/AAR 文件、提取 HTTP API 端点（Retrofit、OkHttp、Volley）、追踪调用链。

关键事实：
1. **单一深度插件**：只有 1 个 SKILL.md + 1 个 command + 5 个 reference 文档 + 4 组脚本（.sh + .ps1 各一套，共 8 个脚本文件），专注做好一件事
2. **跨平台完整**：每个 shell 脚本都有对应的 PowerShell 版本（Linux/macOS 用 .sh，Windows 用 .ps1）
3. **双引擎反编译**：同时支持 jadx（广度覆盖）和 Fernflower/Vineflower（Java 还原质量更高），可结合使用
4. **5 份 reference 文档**：`setup-guide.md`、`jadx-usage.md`、`fernflower-usage.md`、`api-extraction-patterns.md`、`call-flow-analysis.md`，是该仓库质量最高的部分
5. 5,485 stars，是本批 4 个案例中 star 最高的，说明 Android 安全/分析社区有强需求

### 1.2 架构剖析
- **目录结构**：
```
android-reverse-engineering-skill/
└── plugins/
    └── android-reverse-engineering/
        ├── .claude-plugin/
        │   └── plugin.json            # 注册 skill + command
        ├── skills/
        │   └── android-reverse-engineering/
        │       ├── SKILL.md           # 核心技能文档
        │       ├── references/        # 5 份深度参考文档
        │       │   ├── setup-guide.md
        │       │   ├── jadx-usage.md
        │       │   ├── fernflower-usage.md
        │       │   ├── api-extraction-patterns.md
        │       │   └── call-flow-analysis.md
        │       └── scripts/
        │           ├── check-deps.sh / .ps1
        │           ├── decompile.sh / .ps1
        │           ├── find-api-calls.sh / .ps1
        │           └── install-dep.sh / .ps1
        └── commands/
            └── decompile.md           # 入口命令
```
- **文件类型分布**：1 个 SKILL.md / 1 个 command / 5 个 reference .md / 8 个脚本（4 sh + 4 ps1）/ 1 个 plugin.json
- **编排关系**：`commands/decompile.md` 是用户入口 → 调用 SKILL.md 的知识 → 执行 `scripts/*.sh` 进行实际工作 → 输出报告给用户。SKILL.md 通过 `${CLAUDE_PLUGIN_ROOT}` 路径变量引用脚本，保证安装后路径正确
- **跨件契约**：`${CLAUDE_PLUGIN_ROOT}/skills/android-reverse-engineering/scripts/` 是 SKILL.md 和 command 共同引用的脚本根路径，两者保持一致（已验证）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「工具知识文档化 + 脚本自动化 + 结构化输出」——让 Claude 读懂逆向工程工具的使用规律（通过 references/），然后通过标准化脚本执行实际操作
- **解决什么问题**：Android APK 逆向需要熟悉多个命令行工具（jadx、dex2jar、Fernflower），并组合分析。把这个知识固化为技能，让不熟悉工具的用户也能完成逆向分析
- **Trade-off**：`install-dep.sh` 使用 sudo 安装系统包，降低了安装门槛，但引入了权限风险；PATH 修改写入 shell 配置文件，对用户透明但有持久化副作用
- **认知模型**：Claude 是逆向分析的「操作员」，脚本是工具，references/ 是知识库。Claude 读 references 来决定用哪个工具，读 SKILL.md 来执行分析流程

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：深度单技能 + 参考库分层（Deep-Single + Reference-Layered）**

与「广度平铺多技能」相反，这个仓库把所有精力集中在一个领域（Android 逆向），但在这个领域内做到极深：1 个技能、5 份参考文档、8 个脚本，每一份文档都是独立的知识模块，可以被技能按需引用。

模式特征清单（4 条）：
- 特征 1：单一 SKILL.md 涵盖整个工作流，不拆分成多个子技能
- 特征 2：`references/` 目录存储工具特化知识（jadx-usage.md vs fernflower-usage.md），按工具而非按阶段组织
- 特征 3：脚本成对出现（.sh + .ps1），覆盖 Linux/macOS/Windows 三平台
- 特征 4：`check-deps.sh` 提供机器可读的依赖状态输出（`INSTALL_REQUIRED:<dep>` 格式），方便 Claude 解析后自动安装

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要深入一个专业工具生态 | ✅ 高度适用 | 参考库分层设计能容纳大量工具知识 |
| 跨平台用户群（Windows + Linux + macOS）| ✅ 适用 | .sh + .ps1 双版本脚本开箱即用 |
| 需要频繁跨多个领域切换 | ❌ 不适用 | 单技能深度设计不适合做通用工具箱 |
| 不想管理任何脚本依赖 | ❌ 注意 | 安装脚本有 sudo 权限要求，需用户有管理员权限 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 深度单技能 + 参考库分层（本仓库）| SimoneAvogadro/android-reverse | 领域深度极高；参考库可独立维护 | 覆盖面窄；脚本依赖复杂（sudo）|
| 域内平铺多技能 | RKiding/Awesome-finance-skills | 多工具组合使用；独立安装 | 每个技能相对浅 |
| 纯 NL 无脚本 | BayramAnnakov/claude-reflect | 零依赖安装；无权限问题 | 无法执行系统级操作 |

### 2.4 改进空间
1. **当前问题**：`find-api-calls.sh` 的 `run_grep` 函数调用时第一个参数传 `-i`（大小写不敏感标志），导致 grep 把 `-i` 当作搜索模式，实际的认证 credential 和 URL 搜索完全失效。**改进做法**：修改 `run_grep` 函数签名为 `run_grep(flags, pattern)`，或者在调用时用 `grep -i "$pattern"`。**预期收益**：`--auth` 模式的 API 搜索恢复正常
2. **当前问题**：`commands/decompile.md` 的 `allowed-tools` 包含 `Write` 和 `Edit`，但命令步骤里从未使用这两个工具。**改进做法**：把 `Write` 和 `Edit` 从 allowed-tools 里移除。**预期收益**：减少不必要的权限提示，遵循最小权限原则

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**（3 个文件，single 策略）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/decompile.md | 83 | 缺 `## Output` section；Write+Edit 工具声明冗余；4 个模糊比较词 |
| SKILL.md | 92 | 4 个模糊比较词（「best results」等）|
| plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **机器可读的依赖检查输出** → `check-deps.sh` 输出 `INSTALL_REQUIRED:<dep>` 和 `INSTALL_OPTIONAL:<dep>` 格式，Claude 可以解析后自动触发安装 → `scripts/check-deps.sh` → 借鉴：设计带依赖的脚本时，把依赖检查结果格式化为 AI 可解析的结构化输出，而不是只打印人类阅读的文字

2. **Phase-based 工作流（分阶段执行）** → SKILL.md 把逆向工程分为 Phase 1（依赖检查）→ Phase 2（文件分析）→ Phase 3（选择引擎）→ Phase 4（执行反编译），清晰的阶段划分防止跳步 → SKILL.md L1-50 → 借鉴：多步骤工作流技能应该明确列出阶段，并规定每个阶段的前置条件

3. **参考文档分工明确** → `jadx-usage.md` 只讲 jadx，`fernflower-usage.md` 只讲 Fernflower，`api-extraction-patterns.md` 专注 API 提取，职责完全不重叠 → `references/` 目录 → 借鉴：当技能需要涵盖多个工具或方法时，把每个工具的详细知识独立成 reference 文件，技能主体只做决策逻辑

4. **中英文双语触发词** → SKILL.md frontmatter 的 `trigger` 字段同时包含英文（`decompile APK`）和中文（`反编译APK`、`安卓逆向`），覆盖中文用户群 → `SKILL.md` L3 frontmatter → 借鉴：面向中英双语用户的技能可以在 `trigger:` 里同时列出两种语言的关键词

### 3.3 当时的缺陷

1. **模糊比较词**（「best results」「better Java output」「best quality」「broad sweep」）→ 根本原因：描述工具比较时用了主观形容词，没有给出可量化标准（如「jadx 对 .dex 格式支持更好；Fernflower 对复杂 Java 类的还原质量更高，但速度更慢 3-5 倍」）→ 自查：我的 echo-sleuth SKILL.md 有无类似模糊比较词？有 `relevant` 等，但程度较轻

2. **`commands/decompile.md` 声明了不需要的工具权限**（Write + Edit 在 allowed-tools 里，但从未使用）→ 根本原因：作者在初始设计时保守地列了所有「可能会用到」的工具，没有做实际使用验证 → 自查：我的 commands 有没有类似的多余权限声明？需要逐个检查

3. **`install-dep.sh` 使用 sudo + 永久修改 shell 配置文件**（无用户确认，直接 append `export PATH=...` 到 `~/.zshrc`/`~/.bashrc`）→ 根本原因：在 install 脚本里追求「零摩擦安装」，但跳过了用户同意步骤 → 自查：我的安装脚本有无类似的持久化操作？echo-sleuth 无安装脚本 ✓

### 3.4 当时的优化机会

1. 把模糊比较词（「best results」等）替换为具体性能/场景描述（「对 100K+ 类的大型 APK，jadx 内存消耗低 40%；Fernflower 对匿名内部类的还原更准确」）
2. 从 `commands/decompile.md` 的 `allowed-tools` 里移除 Write 和 Edit
3. 修复 `find-api-calls.sh` 的 `run_grep -i` 调用方式（把 `-i` 移入函数内部，而非作为第一个参数传入）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Write+Edit 多余工具权限 | `grep "allowed-tools" commands/decompile.md` | **仍存在**（`allowed-tools: Bash, Read, Glob, Grep, Write, Edit`）| 4 个月未修复，每次运行此命令都会触发不必要的 Write/Edit 权限提示 |
| `find-api-calls.sh` grep -i bug | `grep "run_grep -i" scripts/find-api-calls.sh` | **仍存在**（Line 112、114 传 `-i` 作为模式参数）| `--auth` 模式功能完全失效 4 个月，没有用户报 bug 说明使用率低 |
| sudo 调用（16 处）| `grep -c "sudo" scripts/install-dep.sh` | **仍存在**（16 处 `sudo apt-get`/`sudo dnf`/`sudo pacman`）| 仅是 HIGH 级安全发现，非功能 bug，maintainer 有意为之 |

### 4.2 架构演进
与 audit 时相比，仓库增加了 `.ps1` PowerShell 脚本（每个 .sh 都有对应 .ps1），说明 Windows 支持是后来补充的。`references/` 文档内容有所丰富（`call-flow-analysis.md` 和 `api-extraction-patterns.md` 内容更详细）。架构本质未变，是持续精炼而非重构。

### 4.3 新增的可学习模式
- **PowerShell 配对模式**：每个 shell 脚本都有 PowerShell 版本，两者功能对等但实现独立，用 Claude Code 技能统一调度（根据操作系统调用对应版本）。这是跨平台脚本技能的成熟做法，值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的
1. **allowed-tools 最小化**：echo-sleuth 的 audit.md 和 dashboard.md 只声明实际需要的工具
2. **无 sudo 依赖**：echo-sleuth 脚本全部以当前用户权限运行，无 sudo 调用
3. **工作流有明确阶段**：echo-sleuth 的命令（如 `/recall`）有清晰的「查找 → 筛选 → 分析」步骤，与此仓库的 Phase-based 设计一致

### 5.2 挑战 / 验证
- **挑战了什么假设**：我之前认为 5,485 stars 的仓库应该有完善的 bug 修复流程。事实是：`find-api-calls.sh` 的 grep 功能缺失（`--auth` 模式失效）从 audit 到现在 4 个月都没有被修复，说明即使高 star 仓库也存在功能性 bug 长期未被发现的情况——如果没人使用 `--auth` 功能，就没有 bug report。
- **这给我带来认知**：issue tracker 上没有 bug 不等于没有 bug。关键功能需要有主动测试，而不是等用户反馈。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 commands 里是否有声明但从未在正文使用的 allowed-tools
for f in /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/*.md; do
  tools=$(grep "^allowed-tools:" "$f" 2>/dev/null | sed 's/allowed-tools: //' | tr ',' '\n' | tr -d ' ')
  for tool in $tools; do
    if ! grep -q "$tool" "$f" 2>/dev/null; then echo "UNUSED TOOL: $f → $tool"; fi
  done
done
# 命中后：从 allowed-tools 里移除未使用的工具
```

```bash
# 检查我的脚本是否有类似 run_grep 这样的函数调用方式问题
grep -rn "function.*grep\|_grep\|run_grep" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/scripts/ 2>/dev/null
# 命中后：验证函数调用时参数顺序是否正确
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的脚本添加机器可读的依赖检查输出（仿照 `check-deps.sh` 的 `INSTALL_REQUIRED:` 格式）
   - **为何可行**：echo-sleuth 有 `scripts/echolib.py` 依赖 Python 3.9+，目前没有自动检查机制
   - **第一步**：在 `scripts/` 创建 `check-deps.sh`，检查 Python 版本和必要的包，输出 `INSTALL_REQUIRED:` 格式，约 15 分钟

2. **想法**：为 drama-workshop-skills 或 claude-for-legal 添加 PowerShell 脚本配对（如果有 sh 脚本）
   - **为何可行**：这两个仓库面向职业用户，Windows 用户比例可能较高
   - **第一步**：检查 drama-workshop-skills 是否有 install.sh，若有，用 PowerShell 重写对应逻辑，约 30-60 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 SimoneAvogadro/android-reverse-engineering-skill 的核心目的**：深度专业领域工具（Android 逆向工程），包含脚本化安装和分析流程

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是深度垂直领域技能；都有专业用户群（安全研究员 vs 律师）；都需要外部工具 | 法律分析 vs 二进制逆向；claude-for-legal 可能无系统脚本 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有脚本后端 | Echo Sleuth 是日常开发工具，android-re 是安全专业工具 | 低 |
| MarkQWu/drama-workshop-skills | 无 | - | 创意 vs 安全技术 | 无 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 包含多余 allowed-tools | 见 §6.1 自查命令 | echo-sleuth 的 recall.md、recap.md 等缺少 allowed-tools 声明（而非多余）| 中（性质相反） |
| SKILL.md 有模糊比较词（「best」「better」）| `grep -n "best results\|better\|best quality" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | 暂未发现严重的模糊比较词 | 低 |
| 脚本无 check-deps 环节 | `find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "check-deps*"` | 未找到，echo-sleuth 没有依赖检查脚本 | 低（可优化） |

**命中后的具体行动建议**：
- echo-sleuth 的 4 个缺 `allowed-tools` 的命令文件 → 逐一添加（参见 §7.2 of Q00/ouroboros 案例中的相同发现）

### 7.3 别人的更优方案

1. **领域**：机器可读的依赖检查脚本（结构化输出供 AI 解析）
   - **本案例做法**：`check-deps.sh` 输出 `INSTALL_REQUIRED:jadx` 等格式，SKILL.md 明确告诉 Claude「解析这些行后自动安装」
   - **我的项目现状**：echo-sleuth 没有依赖检查脚本；用户如果缺少依赖只能看到 Python 报错，没有引导
   - **如何借鉴**：在 `echo-sleuth/scripts/` 创建 `check-deps.sh`，检查 Python 版本、`echolib.py` 所需包，输出 `INSTALL_REQUIRED:` 行，并在 agent 里加「运行检查后按提示安装」的步骤

2. **领域**：References 目录的工具特化分层
   - **本案例做法**：5 份 reference 文档各自专注一个知识点（jadx / fernflower / call-flow / api-extraction / setup），SKILL.md 通过 `${CLAUDE_PLUGIN_ROOT}/...` 路径引用按需加载
   - **我的项目现状**：echo-sleuth 的 `skills/jsonl-core/` 把所有 JSONL 知识堆在一个 SKILL.md 里（较长），没有拆分成 references/
   - **如何借鉴**：把 `skills/jsonl-core/SKILL.md` 里的「事件类型参考表」移到 `references/event-types.md`，把「提取命令参考」移到 `references/extraction-commands.md`，SKILL.md 精简为决策逻辑

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：allowed-tools 最小权限实践（按命令实际需求声明）
- **我的做法**：echo-sleuth 的 `commands/audit.md` 声明 `allowed-tools: Bash, Read`，与命令实际使用的工具精确匹配；`commands/extract.md` 声明 `allowed-tools: Bash, Read, Write`，因为确实需要写入内存文件
- **本案例做法（弱在哪）**：`commands/decompile.md` 声明 `Write` 和 `Edit` 但从不使用——audit 时发现，4 个月后仍未修复
- **意义**：echo-sleuth 的 allowed-tools 声明更准确，减少了不必要的用户权限提示。这是好习惯，应该继续保持并推广到 claude-for-legal。

---

## 八、术语表

### <a name="android"></a>Android APK（安卓应用安装包）
> APK 是 Android 应用的分发格式（类似 Windows 的 .exe 文件）。内部是一个 ZIP 压缩包，包含编译后的 Dalvik 字节码（.dex 文件）、资源文件、清单文件等。「逆向工程」就是把这些字节码反编译回可读的 Java/Kotlin 代码，用于安全分析或研究。

### <a name="jadx"></a>jadx
> 一个开源的 Android 反编译工具，直接把 .dex 字节码反编译为 Java 源码。特点是支持大多数 APK 类型，有图形界面和命令行两种模式，对初学者友好。相比之下，Fernflower/Vineflower 擅长还原复杂的 Java 泛型和匿名类。

### <a name="sudo"></a>sudo（超级用户执行）
> Linux/macOS 的提权命令，让当前用户以 root（超级管理员）权限执行某条命令。用于安装系统级软件包（如 `sudo apt-get install jadx`）。风险：脚本里包含 sudo 意味着安装者必须有管理员权限，且所有安装操作都以 root 权限执行，一旦脚本有漏洞，影响范围扩大到整个系统。

### <a name="allowed-tools"></a>allowed-tools（允许工具列表）
> Claude Code command frontmatter 里的一个字段，声明这个命令在执行过程中被允许使用哪些工具（如 Bash、Read、Write、Edit）。Claude Code 在每次使用列表外的工具时会向用户请求额外权限。列表越精确，用户受到的「意外权限提示」越少，安全性越高。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`trigger`、`allowed-tools` 等）。Claude Code 读 command 或 SKILL.md 文件时先解析 frontmatter。
