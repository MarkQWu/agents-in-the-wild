# peterkrueck/Claude-Code-Development-Kit — 学习案例

**仓库**：https://github.com/peterkrueck/Claude-Code-Development-Kit
**Stars**：1342 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-23（基于当前 HEAD）
**主题标签**：dev-kit, security-hooks, image-skills, command-template, security-blocked

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

Claude-Code-Development-Kit 是 Peter Krueck 为个人开发工作流打造的 Claude Code 增强套件。它的核心定位是"工程师配套工具箱"：通过 hooks 建立安全网、通过 skills 扩展 AI 的图像创作和部署能力、通过 commands 标准化 merge 和 context 注入流程。1342 颗星，定位介于"配置集合"和"完整开发工具链"之间。

关键事实：
- 4 个 shell hook 脚本（notify、review-on-stop、security-scan、snapshot-baseline）+ 2 个安装脚本（install.sh、setup.sh）
- 5 个 SKILL.md（bg-remove、deploy、image-edit、image-gen、review-work、second-opinion、update-docs）和 2 个 command（merge、prime）
- 使用 install.sh 的 curl-pipe-bash 设计（CRITICAL）
- security-scan hook 本身存在 JSON 注入漏洞（HIGH 讽刺性反转）
- 所有内部 cross-reference 审计通过（无悬挂引用）

### 1.2 架构剖析

```
Claude-Code-Development-Kit/
├── install.sh           # ⚠️ CRITICAL: curl-pipe-bash 安装脚本
├── setup.sh             # 实际安装逻辑（被 install.sh 调用）
├── commands/
│   ├── merge.md         # ⚠️ 缺 name + description frontmatter
│   └── prime.md         # ⚠️ 缺 name + description frontmatter
├── hooks/
│   ├── security-scan.sh # ⚠️ HIGH: $file 字符串插值 JSON 注入
│   ├── review-on-stop.sh # ⚠️ MEDIUM: 可预测 /tmp/ 路径
│   ├── snapshot-baseline.sh # ⚠️ MEDIUM: 同上
│   ├── notify.sh
│   └── config/
│       ├── pipeline.json    # 配置 review_command + docs_command
│       └── sensitive-patterns.json  # 敏感文件模式列表
├── skills/
│   ├── bg-remove/SKILL.md   # 96/100（图像背景移除）
│   ├── deploy/SKILL.md      # 88/100（含 [PLACEHOLDER] 命令）
│   ├── image-edit/SKILL.md  # 90/100
│   ├── image-gen/SKILL.md   # 91/100
│   ├── review-work/SKILL.md # 96/100
│   ├── second-opinion/SKILL.md # 92/100（双模型 review）
│   └── update-docs/SKILL.md # 93/100
├── templates/
│   └── CLAUDE.md            # 82/100（多空白 section stub）
└── settings/
    └── settings.local.json  # Bash(gemini:*) 权限配置
```

- **文件类型分布**：7 个 SKILL.md / 2 个 command / 4 个 hook 脚本 / 2 个安装脚本 / 1 个 CLAUDE.md 模板
- **编排关系**：`pipeline.json` 声明 `review_command: "/review-work"` 和 `docs_command: "/update-docs"`，review-on-stop hook 读取该配置后调用对应 command。hooks 是核心编排层。
- **跨件契约**：所有内部引用（commands → skills、hooks → config 文件、skills → setup.sh 安装的 Python/TypeScript 脚本）均验证通过，零悬挂引用。这是本仓库质量最高的维度。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：工具链补丁——Claude Code 停止时自动 review 工作、捕获敏感文件泄露、拍基线快照。AI 的工作后门上锁，人手不离键盘但享受自动化护栏。
- **解决什么问题**：Claude Code 在停止前不会自动检查：是否修改了文档、是否意外暴露了密钥、这个会话相比上次做了多少工作。这套 hooks 补全了这些"退出检查"。
- **trade-off**：hooks 提供了自动化护栏，但 shell 脚本的安全性反而成了最大的风险来源——security-scan.sh 本身有 JSON 注入漏洞，讽刺性十足。
- **认知模型**：作者把 hooks 视为"AI 工作的后门检查员"，把 skills 视为"AI 的创作工具箱"（图像生成/编辑是独特定位），把 commands 视为"标准工作流的自动化"。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「钩子安全网 + 图像创作技能套件」

核心特征：
- 特征 1：PostToolUse + StopHook 组合——工具调用后验证，停止时汇报。双层 hook 防御网
- 特征 2：pipeline.json 配置驱动——hook 读配置决定调用哪个 command，避免硬编码
- 特征 3：图像创作 skills（bg-remove、image-edit、image-gen）是生态中罕见的视觉创作能力
- 特征 4：`second-opinion/SKILL.md` 对接 Gemini，实现双模型交叉审查——工具层面的多样性
- 特征 5：`deploy/SKILL.md` 使用 `[YOUR_DEPLOY_COMMAND]` 占位符——模板化设计，鼓励定制

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要会话结束后自动 review 的开发工作流 | ✅ 核心用例 | review-on-stop hook 专为此设计 |
| 图像生成和编辑工作流 | ✅ 独特优势 | 生态中罕见，3 个 image skill 完整覆盖 |
| 敏感代码仓库（需防止密钥泄露） | ✅ 设计意图 | security-scan hook 检测敏感文件模式 |
| 安全审计严格的企业环境 | ❌ 不适用 | CRITICAL install.sh 安全风险未修复 |
| 零安装依赖的轻量配置 | ❌ 不适用 | setup.sh 安装 Python 脚本和 TypeScript，有依赖 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 钩子安全网（Dev Kit 模式） | peterkrueck/Claude-Code-Development-Kit | 退出检查自动化，密钥保护有 hooks | hooks 本身有安全漏洞，讽刺性反转 |
| 纯 skill 集合 | numman-ali/n-skills | 无 hook 副作用，安装简单 | 无自动化护栏 |
| 多 agent 分层 | nyldn/claude-octopus | 覆盖面更广，多模型路由 | 维护复杂度高 |

### 2.4 改进空间

1. **当前问题**：`install.sh` 设计为 curl-pipe-bash 安装，无内容验证。**改进做法**：改为提供两步安装（download → review → execute），在 README 中给出 SHA256 校验步骤。**预期收益**：消除 CRITICAL 安全发现，unblock contribution。
2. **当前问题**：`hooks/security-scan.sh` 在 JSON 输出时直接嵌入 `$file` 变量，含有 `"` 或 `\` 的文件名会破坏 JSON 结构（line 141）。**改进做法**：用 `jq -n --arg f "$file" --arg r "message" '{"decision":"block","reason":$r}'` 替换字符串拼接。**预期收益**：消除 HIGH 安全发现，修复"安全扫描器本身可被绕过"的讽刺性漏洞。
3. **当前问题**：`commands/merge.md` 和 `commands/prime.md` 均缺 `name` 和 `description` frontmatter（合计 4 个必填字段缺失）。**改进做法**：加 YAML frontmatter 块（6 行）。**预期收益**：NLPM 分数从 33/35 提升到 73/75，command 可被自动发现和索引。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **82/100**（Security: BLOCKED，Bugs: 4，Quality Issues: 14，Security Findings: 7）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/merge.md | 33 | 缺 name + description frontmatter（-50 组合） |
| commands/prime.md | 35 | 同上 |
| templates/CLAUDE.md | 82 | 4/6 sections 是空白 stub |
| skills/deploy/SKILL.md | 88 | 所有命令是 [PLACEHOLDER] |
| skills/image-edit/SKILL.md | 90 | 缺 output step |
| skills/image-gen/SKILL.md | 91 | 模糊词："good enough"、"best variant" |
| skills/second-opinion/SKILL.md | 92 | 模糊词："genuinely"、"routine"、"real consequences" |
| skills/update-docs/SKILL.md | 93 | 缺 output format；"meaningful" 未定义 |
| skills/bg-remove/SKILL.md | 96 | 清理步骤只在 Important Rules 里，不在 workflow |
| skills/review-work/SKILL.md | 96 | 轻微模糊词 |

### 3.2 当时值得借鉴的模式

1. **pipeline.json 配置驱动 hook**：hook 读取 `hooks/config/pipeline.json` 决定调用哪个 slash command，解耦了 hook 逻辑和 command 名称。修改 command 名字只需改配置，不用动 hook 代码。
2. **sensitive-patterns.json 外置敏感文件规则**：把敏感文件模式（`.env`、`*.key` 等）放在 JSON 配置里而不是硬编码在 shell 脚本中——规则可维护，可扩展，hook 逻辑简洁。
3. **`second-opinion` 双模型审查 skill**：通过 `settings.local.json` 的 `Bash(gemini:*)` 权限，让 Claude 调用 Gemini CLI 进行第二方审查——工具层面的多样性，低成本实现。
4. **snapshot-baseline + review-on-stop 组合**：会话开始时拍 git diff 基线，停止时 diff 显示本次所有变更并自动 review——自动化的工作汇报机制。
5. **跨件引用零悬挂**：所有 command → skill、hook → config、skill → script 的引用均验证通过，说明作者在维护时保持了引用完整性。

### 3.3 当时的缺陷

1. **install.sh 的 curl-pipe-bash 设计**：注释文档（line 6）就说用法是 `curl ... | bash`，随后 line 90-188 下载 tarball 然后直接执行 setup.sh，无校验。根本原因：开发者优先考虑安装便利性，没有意识到 GitHub 仓库本身也可能被攻击（compromised → RCE on all users）。**我有没有犯？** drama-workshop-skills 有 `install.sh`——需要检查。
2. **security-scan.sh 中 `$file` 注入到 JSON echo**（讽刺性反转）：一个用于防止安全问题的 hook 自己存在 JSON 注入漏洞。含 `"` 的文件名可以注入 `"decision":"allow"` 字段，绕过安全阻断。根本原因：shell 脚本字符串拼接生成 JSON 是常见的不安全模式，应始终用 `jq` 构造 JSON。**我有没有犯？** 我的 hook 里如果有 JSON 输出需要检查。
3. **commands 缺 frontmatter**：merge.md 和 prime.md 直接以 `# Merge Command` 正文开始，没有 frontmatter。导致 NLPM 分数 33/35，无法被自动发现。根本原因：作者可能把这两个 command 当作普通 Markdown 文档写，没有意识到 Claude Code 需要 frontmatter 才能正确识别 command。**我有没有犯？** 我需要检查我的 command 文件是否都有 frontmatter。

### 3.4 当时的优化机会

1. **merge.md / prime.md**：各加 6 行 YAML frontmatter（name、description、allowed-tools、$ARGUMENTS 处理）→ 分数从 33 → 73
2. **security-scan.sh:141**：`echo "{...\"$file\"...}"` → `jq -n --arg f "$file" --arg r "reason" '{"decision":"block","reason":$r}'`
3. **hooks/review-on-stop.sh**：`/tmp/claude-stop-${SESSION_ID}.state` → `mktemp` 创建随机临时文件

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| merge.md 缺 frontmatter | `head -5 commands/merge.md` | **持续存在**（以 `# Merge Command` 正文开头） | 77 天无修复，command 仍无法被 NLPM 识别 |
| security-scan.sh JSON 注入 | `sed -n '138,145p' hooks/security-scan.sh` | **持续存在**（`echo "{...\"$file\"...}"` 仍在 line 141） | 高风险漏洞：保护者本身有漏洞 |
| install.sh curl-pipe-bash | `head -10 install.sh` | **持续存在**（line 6 注释说明 `curl ... | bash` 用法，line 90 下载 tarball 直接执行） | CRITICAL 安全模式在所有用户安装时触发 |

### 4.2 架构演进

当前 HEAD 目录结构与 audit 时一致（`commands/`、`hooks/`、`skills/`、`templates/`、`settings/`），没有发现架构层面的显著变化。`CHANGELOG.md` 存在但内容需要细读才能确认演进细节。

这是一个"稳定但有债"的仓库状态——没有新功能扩展，但历史安全债也没有还清。

### 4.3 新增的可学习模式

暂无。当前 HEAD 与 audit 时状态高度一致，未发现新模式。

---

## 五、校准

### 5.1 我已经在做对的

1. drama-workshop-skills 的 `install.sh` 需要进一步检查，但 bureau 和 graphify 无安装脚本，不存在 curl-pipe-bash 风险
2. 我的 skill 文件都有完整 frontmatter（从 graphify 的测试 `assert "allowed-tools:" in head` 可以看出强制要求）
3. 我没有在 shell 脚本里用字符串拼接生成 JSON（无 echo 拼接 JSON 的模式）

### 5.2 挑战 / 验证

这个案例最大的认知冲突是**"保护者本身的脆弱性"**：一个专门防止安全问题的 hook（security-scan.sh）自己有 JSON 注入漏洞，且这个漏洞允许攻击者绕过该 hook 的保护。这提醒我：安全机制本身需要被同等严格地审查——不能因为是"保护代码"就信任它。shell 脚本生成 JSON 时必须用 `jq`，这是一条我需要固化为个人规范的做法。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 hook 里有没有字符串拼接生成 JSON
find /tmp/my-repos/MarkQWu-bureau/hooks/ /tmp/my-repos/MarkQWu-gstack/scripts/ -name "*.sh" 2>/dev/null | \
  xargs grep -l 'echo.*{.*}' 2>/dev/null
```
命中后怎么办：把 `echo "{...}"` 改为 `jq -n --arg x "$var" '{"key": $x}'`。

```bash
# 检查我的 command 文件有没有 frontmatter
for f in $(find /tmp/my-repos/MarkQWu-bureau/commands/ -name "*.md" 2>/dev/null); do
  head -1 "$f" | grep -q "^---" || echo "MISSING frontmatter: $f"
done
```
命中后怎么办：加 YAML frontmatter 块（name、description、allowed-tools 三个字段）。

```bash
# 检查 drama-workshop-skills 的 install.sh
grep -n "curl" /tmp/my-repos/MarkQWu-drama-workshop-skills/install.sh 2>/dev/null | head -5
```
命中后怎么办：改为两步安装（下载 + SHA256 校验 + 执行），或提供 git clone 安装路径。

### 6.2 灵感 → 实施路径

1. **想法**：借鉴 pipeline.json 配置驱动 hook 的模式，给 bureau 的会话退出 hook 加一个 `bureau-pipeline.json`，声明退出时应调用哪个 command（如 `/bureau:lint`）
   - **为何可行**：bureau 有 lint command，只需一个 hook 读取配置后调用它
   - **第一步**：在 bureau/hooks/config/ 新建 `bureau-pipeline.json`，声明 `"on_stop": "/bureau:lint"`；在 hooks/review-on-stop.sh 里读取该配置。约 20 分钟可完成。

2. **想法**：借鉴 sensitive-patterns.json 的外置配置模式，给 gstack 的安全检查 hook 加一个可配置的敏感文件模式列表
   - **为何可行**：gstack 已有 checklist.md 里列出了敏感文件类型，只需提取为 JSON 配置
   - **第一步**：从 gstack/review/checklist.md 提取 `.env`、`*.key` 等模式到 `hooks/config/sensitive-patterns.json`。约 15 分钟可完成。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 peterkrueck/Claude-Code-Development-Kit 的核心目的**：通过 hooks 自动化开发工作流护栏 + 图像创作 skills 套件

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是开发工具套件，都有 review 相关 skill | gstack 无图像 skill，无 security-scan hook | 高 |
| MarkQWu/bureau | 中 | 都有 hooks，都有 stop 时的触发逻辑 | bureau 聚焦知识库管理而非代码 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 都有 install.sh | 领域完全不同 | 低（但安全检查优先级高） |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| command 缺 frontmatter | `head -1 /tmp/my-repos/MarkQWu-bureau/commands/*.md` | bureau/commands 下所有 md 以 `#` 开头（无 frontmatter）→ **全部命中** | 高 |
| install.sh curl-pipe-bash | `grep "curl" /tmp/my-repos/MarkQWu-drama-workshop-skills/install.sh` | 需要检查 drama-workshop-skills/install.sh | 待查 |
| shell 脚本 JSON 字符串拼接 | `grep -n 'echo.*{' /tmp/my-repos/MarkQWu-bureau/hooks/*.sh 2>/dev/null` | bureau hooks 目录当前无 .sh 文件 → 未命中 | N/A |

**命中后的具体行动建议**：
- bureau/commands/ 下所有 .md 文件 → 逐一加 YAML frontmatter（name、description、allowed-tools）→ 约 45 分钟可完成（10 个 command）
- drama-workshop-skills/install.sh → 检查并确认是否有 curl-pipe-bash 模式，如有则改为两步安装

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：pipeline.json 解耦 hook 逻辑与 command 名称
   - **本案例做法**：`hooks/config/pipeline.json` 声明 `review_command` 和 `docs_command`，hook 脚本从配置读取，不硬编码 command 名（`hooks/review-on-stop.sh` + `hooks/config/pipeline.json`）
   - **我的项目现状**：gstack 的 hook（如有）直接硬编码 command 名，修改 command 需要改 hook 脚本
   - **如何借鉴**：在 gstack 和 bureau 的 hooks/config/ 下各加一个 JSON 配置文件，声明 hook 触发时调用的 command，hook 脚本从配置读取

2. **领域**：`second-opinion` 双模型审查
   - **本案例做法**：`skills/second-opinion/SKILL.md` 通过 `Bash(gemini:*)` 权限让 Claude 调用 Gemini CLI 进行第二方审查，2 个 AI 各自独立分析代码（skills/second-opinion/SKILL.md）
   - **我的项目现状**：gstack 有 review skill 但是单模型审查
   - **如何借鉴**：在 gstack/review/ 下加一个 `second-opinion.md` skill，配置 `Bash(gemini:*)` 权限并说明何时调用（针对关键架构变更或安全相关代码）

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：Skill 文件 frontmatter 完整性
- **我的做法**：graphify 的测试套件（`tests/test_skillgen.py:508`）强制断言 `"allowed-tools:" in head`，所有生成的 skill 都通过测试确保有 frontmatter
- **本案例做法**：merge.md 和 prime.md 完全没有 frontmatter，且 77 天无修复
- **意义**：graphify 的"通过测试强制 frontmatter"模式是比人工检查更可靠的保障。如果 peterkrueck 有类似的 CI 检查，这两个 command bug 早就被发现了。这是一个可以作为"最佳实践示例"提给上游的模式。

---

## 八、术语表

- **[PostToolUse hook](#posttooluse-hook)**：Claude Code hook 事件类型，在 AI 调用工具（如 Bash、Write、Edit）之后触发，用于验证工具调用的结果或副作用。
- **[StopHook](#stophook)**（review-on-stop）：Claude Code 会话停止时触发的 hook，本案例用于自动 review 本次会话的工作并生成报告。
- **[pipeline.json](#pipeline-json)**：peterkrueck 定义的 hook 配置文件格式，用键值对声明 hook 触发时调用的 slash command，把配置从代码中分离出来。
- **[curl-pipe-bash](#curl-pipe-bash)**：`curl URL | bash` 模式，在 install.sh 中提示用户用这种方式安装，等同于从互联网下载未经验证的代码直接执行。一旦仓库被攻击，所有安装用户都受影响（供应链攻击）。
- **[JSON 注入](#json-injection)**：在 shell 脚本中用字符串拼接构造 JSON，当变量值包含 `"`、`\` 等特殊字符时，构造出的 JSON 结构被破坏或被额外字段覆盖。应始终用 `jq` 构造 JSON。
- **[jq](#jq)**：命令行 JSON 处理工具，`jq -n --arg key "$value" '{"key": $key}'` 方式构造 JSON 时会对特殊字符自动转义，是 shell 脚本安全构造 JSON 的正确方法。
- **[TOCTOU](#toctou)**：Time-of-Check Time-of-Use，一种竞态条件漏洞。当攻击者能在"检查"和"使用"之间的窗口期替换文件或符号链接时，检查结果就失效了。可预测的 `/tmp/` 路径是 TOCTOU 的常见攻击面。
- **[mktemp](#mktemp)**：创建随机命名临时文件的命令，`mktemp` 生成的路径无法被预测，消除 TOCTOU 风险。
- **[printf '%b'](#printf-b)**：bash 中解释 `\n`、`\t` 等转义序列的 printf 变体。当字符串来自外部（如 git diff 的文件名），若文件名包含 `\n`，`printf '%b'` 会把它解释为换行，破坏 JSON 输出。应用 `printf '%s'` 替代。
- **[frontmatter](#frontmatter)**：Markdown 文件顶部 `---` 包围的 YAML 元数据块。对于 Claude Code command 文件，缺少 frontmatter 导致 NLPM 无法识别 `name` 和 `description`，合计扣 50 分（-25 × 2）。
- **[second-opinion skill](#second-opinion-skill)**：本案例中通过 `Bash(gemini:*)` 权限让 Claude 调用 Gemini CLI 对代码进行第二方审查的 skill，实现了双模型交叉验证。
