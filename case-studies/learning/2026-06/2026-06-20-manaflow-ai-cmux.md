# manaflow-ai/cmux — 学习案例

**仓库**：https://github.com/manaflow-ai/cmux
**Stars**：15339 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-20（基于当前 HEAD）
**主题标签**：`cross-reference`, `manifest-discipline`, `security-gate`, `single-purpose`, `nl-binary-hybrid`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

cmux 是一个 macOS 原生终端应用，专门为 AI coding agent 工作流设计——带垂直标签、通知系统、macOS Sparkle 自动更新。15339 stars，活跃开发中。

关键事实：
1. **主体是 Swift/Xcode 项目**（macOS 原生 app），NL 工件（skills + commands）只是开发辅助工具
2. 当前有 19 个 skill（audit 时 5 个，大幅扩展）+ 6 个 slash command
3. 仓库极大：含 Xcode 项目、多个 Swift package、web 前端（Next.js）、Ghostty 集成、多语言 README（20 种语言）
4. 有 Homebrew 公式（`homebrew-cmux/`）——用户通过 `brew install` 安装

### 1.2 架构剖析

```
cmux/
├── .claude/
│   └── commands/           # 6 个 slash command（全部缺 YAML frontmatter）
│       ├── cleanup-builds.md  # 新增（audit 后）
│       ├── pull.md
│       ├── release.md        # ← 含 changelog.mdx 双重更新矛盾
│       ├── release-local.md  # ← 同上
│       ├── release-nightly.md
│       └── sync-branch.md
├── CLAUDE.md               # 项目开发指南（98/100，vague: "minimal"）
├── AGENTS.md               # 多 agent 协作说明
├── skills/                 # 19 个 skill（audit 时 5 个）
│   ├── cmux/               # 核心 skill（99/100）
│   ├── cmux-architecture/  # 新
│   ├── cmux-backend/       # 新（含 references/effect-boundaries.md）
│   ├── cmux-browser/       # 原有（98/100）
│   ├── cmux-custom-sidebar/# 新
│   ├── cmux-customization/ # 新
│   ├── cmux-debugging/     # 新
│   ├── cmux-dev-workflow/  # 新
│   ├── cmux-diagnostics/   # 原 cmux-debug-windows 重命名
│   ├── cmux-ghostty/       # 新
│   ├── cmux-keyboard-shortcuts/ # 新
│   ├── cmux-localization/  # 新（含 references/audit-workflow.md）
│   ├── cmux-markdown/      # 原有（98/100）
│   ├── cmux-release/       # 原有（原 release/）
│   ├── cmux-settings/      # 新
│   ├── cmux-shared-behavior/# 新
│   ├── cmux-socket-policy/ # 新
│   ├── cmux-testing/       # 新
│   └── cmux-workspace/     # 新
├── scripts/                # 24 个 shell 脚本 + 6 个 Python + 2 个 JS（含安全漏洞）
├── Sources/                # Swift 主体代码
├── web/                    # Next.js 网站
└── ...（Xcode 项目文件、多语言 README 等）
```

- **文件类型分布**：19 个 SKILL.md、0 个 agent、6 个 command、0 个 hook
- **编排关系**：command（用户调用 `/release`）→ skill（Claude 使用 `cmux-release/SKILL.md` 的知识执行）。CLAUDE.md 是所有 Claude 工作的顶层约束文档。skills 之间相互独立，各自聚焦一个开发域
- **跨件契约**：每个 skill 有自己的 `agents/openai.yaml`（OpenAI 兼容 agent 配置）和 `references/` 子目录——表明 cmux team 在多 AI agent 环境（Claude + OpenAI）下使用同一套 skill

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「领域 skill + 知识注入」——每个 skill 对应 cmux 开发的一个领域（调试、本地化、发布、架构...），Claude 工作时加载对应 skill 获得领域知识，而不是把所有知识都塞进 CLAUDE.md
- **解决什么问题**：开发 cmux 时 AI agent（Claude 或 Codex）需要大量上下文（Sparkle 更新机制、macOS 签名流程、Ghostty 集成、本地化规范...），单靠 CLAUDE.md 装不下。skill 分散了上下文负担
- **Trade-off**：
  - commands（slash）和 skill 分离——commands 是动词（"做什么"），skills 是名词（"这个领域的规则"）。但 commands 缺少 frontmatter，导致 Agent 自选时没有 description 可依赖
  - 每个 skill 下的 `agents/openai.yaml` 暗示团队同时使用 Claude Code 和 OpenAI Codex，skill 要对两个 runtime 兼容
- **认知模型**：作者把 AI agent 看作"领域知识加载的消费者"——agent 从 CLAUDE.md 获取顶层规则，从 skill 获取领域细节，commands 触发 agent 执行具体任务

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「原生应用 + 领域 skill 矩阵 + 多 Agent 兼容」**

主体是原生 macOS app（Swift），NL 工件（skills/commands）是开发团队的 AI 辅助工具，不是对外的用户功能。每个开发领域一个 skill，skill 内有 `agents/openai.yaml` 确保 Claude 和 OpenAI agent 都能使用。

模式特征清单：
- skill 是开发工具（团队内部用），不是插件（不对外发布）
- skill 数量与领域复杂度成正比（原生 macOS 开发的 Sparkle/签名/Ghostty 等各占一个 skill）
- commands（slash）触发开发工作流，skills 提供领域知识
- 多 AI agent 兼容：每个 skill 有 `agents/openai.yaml` 声明 OpenAI 侧的 agent 行为

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 复杂原生 app 开发（有签名、发布、本地化等多个域）| ✅ 高度适用 | 每个域一个 skill，上下文负担分散 |
| 简单 CRUD web 应用开发 | ❌ 不适用 | 领域简单，CLAUDE.md 就够了，不需要 19 个 skill |
| 多 AI agent 协作（Claude + OpenAI Codex）| ✅ 适用 | `agents/openai.yaml` 提供了 OpenAI 侧的兼容声明 |
| 公开 Claude Code 插件（面向外部用户）| ❌ 不适用 | commands 无 frontmatter，用户无法通过插件市场发现和使用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：领域 skill 矩阵 | manaflow-ai/cmux | 上下文分散，每个 skill 专注一域 | commands 无 frontmatter，自动化调用困难 |
| CLAUDE.md 单文件兜底 | 小型项目通用做法 | 维护成本低，新人直接读一个文件 | 知识量大时 CLAUDE.md 超长，影响 token 效率 |
| 命令驱动 + Router skill | nlpm 模式 | 自动路由，用户不需要知道每个子工具 | 架构成本高 |

### 2.4 改进空间

1. **当前问题**：6 个 slash command 全部缺少 YAML [frontmatter](#frontmatter)（`name`、`description` 字段），Agent 在需要自动选择工具时没有 description 可读，只能靠文件名猜测。**改进做法**：给每个命令文件加 3 行 frontmatter：`---\nname: xxx\ndescription: xxx\nallowed-tools: [Bash, Write]\n---`。**预期收益**：Agent 自选命令时能看到语义描述，减少误调用。

2. **当前问题**：release.md 和 release-local.md 都有"Also update `docs-site/content/docs/changelog.mdx`"的指令，但 CLAUDE.md 明确说 `web/app/docs/changelog/page.tsx` 从 `CHANGELOG.md` 直接渲染，`docs-site/content/docs/changelog.mdx` **不应手动编辑**。**改进做法**：从 release.md 和 release-local.md 里删除对 `changelog.mdx` 的更新步骤，只保留 `CHANGELOG.md` 的更新。**预期收益**：Agent 执行发布命令不会产生多余的 stale 文件更改。

3. **当前问题**：`scripts/build-sign-upload.sh` line 85 把 `SPARKLE_PRIVATE_KEY` 作为 CLI 参数传给 swift 脚本，私钥在 `ps aux` 里短暂可见。**改进做法**：改为通过环境变量或 temp file 传递（`SPARKLE_PRIVATE_KEY="$key" swift scripts/derive_sparkle_public_key.swift`）。**预期收益**：私钥不再出现在进程列表里，消除 High 安全风险。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **74/100**，**安全门状态：BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.claude/commands/release.md` | 43/100 | 无 frontmatter（name -25, desc -25），"relevant" 模糊量词（-2） |
| `.claude/commands/pull.md` 等 4 个 | 45/100 | 无 frontmatter（name -25, desc -25），无 allowed-tools（-5） |
| `CLAUDE.md` | 98/100 | "minimal" 模糊量词（-2） |
| 5 个 skill（cmux, cmux-browser 等）| 98-99/100 | 各有 1 个模糊量词（-2） |

### 3.2 当时值得借鉴的模式

1. **CLAUDE.md 作为项目顶层规则权威来源** → 为什么好：CLAUDE.md 明确声明了"changelog.mdx 不应手动编辑"等关键约束，作为所有 Agent 行为的权威来源，即使 commands 里的说明与之矛盾，也有一个地方可以仲裁 → 原文：CLAUDE.md Release 章节 → 借鉴：项目应该有一个"规则权威文件"（CLAUDE.md 或 AGENTS.md），所有其他工件里的指令不能高于它

2. **`cmux` 核心 skill 99/100** → 为什么好：`cmux/SKILL.md` 覆盖了 cmux 的核心概念、工作流程、常见操作，是 Agent 的第一参考，达到了近满分 → 借鉴：项目的"核心 skill"（介绍这个项目是什么、怎么工作）应该先于其他 skill 写，且写到 100 分

3. **每个 skill 有独立 references 子目录** → 为什么好：`skills/cmux-backend/references/effect-boundaries.md`、`skills/cmux-localization/references/audit-workflow.md`——把该领域的参考知识放在 skill 目录里，不污染全局 → 借鉴：长文档、规范文件应该放在用到它的 skill 目录的 references/ 里

4. **AGENTS.md 单独声明多 agent 协作规则** → 为什么好：与 CLAUDE.md（给 Claude 用）分开，有专门的 AGENTS.md（给所有 agent 用），关注点分离 → 借鉴：如果项目使用多种 AI agent，应该分开维护各自的 context 文件

### 3.3 当时的缺陷

1. **5 个 commands 全部缺少 frontmatter** → 根本原因：作者写 commands 时把它们当做"人类可读的操作手册"而不是"Agent 可机器读的工具声明"；frontmatter 是给 Agent 自动发现和选择工具用的，作者忽略了这个需求 → 自查：我的 slash commands 有没有 YAML frontmatter？

2. **Changelog 双重更新矛盾** → 根本原因：项目早期 `docs-site/content/docs/changelog.mdx` 是手动维护的；后来改成从 `CHANGELOG.md` 自动渲染，但 release commands 里的旧指令没有同步删除；两套"规范"并存，Agent 不知道该听谁的 → 这是"规范漂移"的典型例子——在两个地方维护同一件事，有一个迟早落后 → 自查：我的 CLAUDE.md 和 commands 里有没有矛盾的指令？

3. **`SPARKLE_PRIVATE_KEY` 作为 CLI 参数（High 安全风险）** → 根本原因：作者把私钥通过 `"$SPARKLE_PRIVATE_KEY"` 直接传给 swift 脚本，图方便，但 `ps aux` 在命令执行期间可以看到所有命令行参数 → 私钥在系统级进程列表里短暂可见，任何有本机访问权限的进程都能读到 → 自查：我的发布脚本有没有把 secret 放在命令行参数里？

### 3.4 当时的优化机会

1. **最高优先级**：给 5 个 commands 加 frontmatter（name + description），每个约 5 分钟
2. **次高优先级**：从 release.md 和 release-local.md 里删除对 `changelog.mdx` 的更新步骤
3. **安全修复**：`scripts/build-sign-upload.sh` line 85 改为环境变量传递私钥

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 5 个 commands 无 frontmatter | `head -3 .claude/commands/*.md` | **仍存在**：所有 5 个原有 command（+新增 cleanup-builds.md）全部以 `# Title` 开头，无 `---` frontmatter | 新增 command 也沿用了错误模式 |
| Changelog 双重更新矛盾 | `grep "changelog.mdx" .claude/commands/release.md` | **仍存在**：release.md line 34、release-local.md line 35 仍要求更新 changelog.mdx | 2+ 个月未修复 |
| SPARKLE_PRIVATE_KEY 作为 CLI 参数 | `grep "SPARKLE_PRIVATE_KEY" scripts/build-sign-upload.sh` | **仍存在**：line 85 `SPARKLE_PUBLIC_KEY_DERIVED=$(swift scripts/derive_sparkle_public_key.swift "$SPARKLE_PRIVATE_KEY")` | High 安全风险持续，私钥仍暴露在进程列表 |

### 4.2 架构演进

这是本批次 4 个案例中**演进最大**的仓库：

- **Skill 数量**：5 → 19（增加了 14 个）
- **重命名**：`cmux-debug-windows` → `cmux-diagnostics`（更准确的名字）
- **新增 command**：`cleanup-builds.md`（管理 Tagged Dev Build 占用的磁盘空间）
- **多 agent 支持**：每个 skill 增加了 `agents/openai.yaml`，表明团队开始同时使用 OpenAI agent
- **AGENTS.md**：新增顶层多 agent 协作说明文件

这说明：一旦团队发现 skill 分域有效，就会快速扩展——从"核心 5 个"到"覆盖所有开发域的 19 个"。同时团队开始支持多 AI agent，为此统一了 skill 格式。

但**三个核心 bug**（commands 无 frontmatter、changelog 矛盾、SPARKLE 私钥）在 2 个月里一个都没修复，这反映了：功能扩展的优先级远高于质量修复。

### 4.3 新增的可学习模式

1. **每个 skill 一个 `agents/openai.yaml`**：统一格式让 skill 可以被 Claude 和 OpenAI Codex 都使用。这是一种"AI 无关"的 skill 设计——skill 内容用 Markdown 写，各 agent 的具体调用方式在 `agents/` 子目录里各自声明。

2. **AGENTS.md 与 CLAUDE.md 分离**：`CLAUDE.md` 是 Claude 专属指南，`AGENTS.md` 是所有 AI agent（包括 Claude 和 Codex）的共同规则。这种分离在多 AI agent 团队里是值得借鉴的架构选择。

3. **skill 命名加 `cmux-` 前缀**：所有 skill 都以产品名 `cmux-` 开头，明确标识这些 skill 是这个项目的内部工具，不会与其他项目的 skill 混淆。

---

## 五、校准

### 5.1 我已经在做对的

1. **commands 有 frontmatter**：我的 echo-sleuth 的 commands/ 目录（`/recall`、`/recap` 等）都有 YAML frontmatter 声明 name 和 description
2. **单一权威规范来源**：drama-workshop-skills 的 `SKILL.md` 是规范的唯一权威，没有在多个文件里写矛盾的指令
3. **skill 前缀约定**：echo-sleuth 的 agents 都有清晰的功能名（recall, file-historian, analyze），与 cmux-xxx 的约定命名思路一致
4. **reference 子目录**：drama-workshop-skills 的 references/ 按功能分放，与 cmux 的做法类似

### 5.2 挑战 / 验证

这次案例**挑战**了我的假设："高 star 仓库 = 高 NL 工件质量"。cmux 有 15339 stars，但 NL Score 只有 74/100，3 个 bug 持续 2 个月未修。高 star 只说明产品本身受欢迎，不说明 NL 工件的质量。NL 工件质量需要独立审查。

另一个发现：**团队规模不重要，纪律重要**。cmux 是多人团队维护的知名项目，但 commands 全部缺少 frontmatter——这是一个每个人写 command 时都会触及的问题，但 2 个月里没有人修。说明没有"写 command 时必须加 frontmatter"的团队纪律或 CI 检查。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 slash commands 是否有 YAML frontmatter
for f in $(find . -path "*/.claude/commands/*.md" -o -path "*/commands/*.md" | grep -v ".git"); do
  if ! head -1 "$f" | grep -q "^---"; then
    echo "MISSING FRONTMATTER: $f"
  fi
done
```
命中后怎么办：在文件顶部加：
```yaml
---
name: <command-name>
description: <一句话描述，说明何时调用这个 command>
allowed-tools: [Bash, Write, Read]
---
```

```bash
# 检查我的不同文件里有没有矛盾的指令（找出同一操作在不同地方的不同说法）
# 例：CLAUDE.md 说"不要更新 X"，但某个 command 说"更新 X"
grep -rn "不要\|禁止\|must not\|do not" CLAUDE.md | while read line; do
  key=$(echo "$line" | grep -oP '(?<=不要|禁止|must not|do not )\S+')
  echo "Checking '$key' in commands..."
  grep -rn "$key" .claude/commands/ 2>/dev/null | head -3
done
```
命中后怎么办：在两处中删除一处，以 CLAUDE.md 的声明为权威。

```bash
# 检查发布脚本里有没有 secret 在命令行参数里
grep -rn '\$[A-Z_]*KEY\|\$[A-Z_]*SECRET\|\$[A-Z_]*TOKEN\|\$[A-Z_]*PASSWORD' scripts/ --include="*.sh" | grep -v "^#" | grep -v "export "
```
命中后怎么办：改为通过环境变量或 temp file 传递（`SOME_KEY="$key" cmd` 或 `echo "$key" > /tmp/key; cmd --key-file /tmp/key; rm /tmp/key`）。

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 的每个 skill 加 `agents/openai.yaml`，实现 Claude + Codex 双 runtime 兼容
   - **为何可行**：参考 cmux 的做法，`agents/openai.yaml` 里的内容是该 skill 的 OpenAI 侧 system prompt，可以从 SKILL.md 内容精简而来
   - **第一步**：看 `cmux/skills/cmux/agents/openai.yaml` 的格式，给 `echo-sleuth/skills/jsonl-core/agents/openai.yaml` 写一个；预计 15 分钟

2. **想法**：在 drama-workshop-skills 里加 AGENTS.md，与 SKILL.md（Claude 专用）分离
   - **为何可行**：drama-workshop-skills 可能未来被其他 AI agent 使用，早分离早受益
   - **第一步**：新建 `AGENTS.md`，把 CLAUDE.md 里"不同 AI agent 的注意事项"单独提出来；预计 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 manaflow-ai/cmux 的核心目的**：为 AI coding agent（Claude/Codex）提供领域 skill 矩阵，让 Agent 在开发 cmux 这个复杂原生应用时能加载对应领域的知识

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是项目内部 AI 工具（skills 服务于项目本身的开发/使用），有 commands + skills + CLAUDE.md 结构 | echo-sleuth 是公开插件，cmux skills 是内部工具 | 高 |
| MarkQWu/drama-workshop-skills | 中 | 都有多个平行 skill 处理不同任务域 | drama 是内容创作工具（外部用户用）；cmux skills 是开发辅助（团队用） | 中 |
| MarkQWu/claude-for-legal | 中 | 都有 commands + practice-area skills 的分层结构，都有多个 plugin.json | claude-for-legal 是面向外部律师用户的产品；cmux skills 是内部工具 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Commands 无 frontmatter | `head -1 commands/*.md \| grep -v "^---"` | echo-sleuth 命中 0（commands 都有 frontmatter）| 无 |
| 不同文件间矛盾指令 | `grep -rn "不要\|禁止" CLAUDE.md` 后交叉检查 | 未检测到明显矛盾 | 暂无 |
| Secret 在 CLI 参数里 | `grep "\$[A-Z]*KEY\|\$[A-Z]*SECRET" scripts/*.sh` | drama-workshop-skills 和 echo-sleuth **无发布脚本，无此风险** | 无 |
| Skill 扩展后 CLAUDE.md inventory 未更新 | `diff <(ls skills/) <(grep "skill" CLAUDE.md)` | **drama-workshop-skills 无 skill inventory 表**（CLAUDE.md 描述了 short-drama 和 short-drama-remake 这两个 skill 目录，与实际一致） | 暂无 |

**命中后的具体行动建议**：
- 暂无高优先级命中，本案例的典型缺陷在我的项目里尚未出现

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：每个 skill 的 `references/` 子目录存放领域参考文档
   - **本案例做法**：`skills/cmux-backend/references/effect-boundaries.md`（Effect.ts 的边界条件文档）、`skills/cmux-localization/references/audit-workflow.md`（本地化审查工作流）——把领域参考文档就近放在用到它的 skill 里
   - **我的项目现状**：echo-sleuth 的 `skills/` 下各 skill 目录里**没有 `references/` 子目录**，所有参考知识都写在 SKILL.md 正文里，导致 SKILL.md 偏长
   - **如何借鉴**：把 `skills/experience-synthesis/SKILL.md` 里"经验分类体系"那节单独提出到 `skills/experience-synthesis/references/taxonomy.md`，SKILL.md 里只保留如何使用分类体系的说明；预计 15 分钟

2. **领域**：AGENTS.md 与 CLAUDE.md 分离
   - **本案例做法**：`CLAUDE.md` 是 Claude 专属指南（工具调用、格式要求）；`AGENTS.md` 是所有 AI agent 的共同规则（安全边界、不能做的事）
   - **我的项目现状**：echo-sleuth 只有 `CLAUDE.md`，把 Claude 特有的规则和所有 agent 共用的规则混在一起
   - **如何借鉴**：新建 `AGENTS.md`，把"所有 AI agent 不能修改的文件列表"、"所有 AI agent 不能执行的危险操作"提出来；`CLAUDE.md` 保留 Claude 专属的格式和工具偏好

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：commands 的 frontmatter 完整性
  - **我的做法**：echo-sleuth 的 8 个 slash commands（`/recall`、`/recap`、`/timeline` 等）全部有 YAML frontmatter，包含 name、description 字段
  - **本案例做法**：cmux 的 6 个 commands 全部缺少 frontmatter，2+ 个月未修复
  - **意义**：这是一个"我已经做对了但容易被忽视"的点——保持这个习惯，未来如果扩展 commands 时要坚守这个约定，不能因为"先写功能再补 frontmatter"的心态逐渐退化

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools` 等）。Claude Code 读 slash command 的 `.md` 文件时，先解析 frontmatter 才能知道这个命令是什么、何时调用、可以用哪些工具。缺少 frontmatter，Agent 在自动选择工具时只能靠文件名猜测用途。

### <a name="Sparkle"></a>Sparkle
> macOS 应用的自动更新框架，开源的。应用在启动时检查 Sparkle 服务器上的 appcast（XML 文件），如果有新版本就提示用户更新。`SPARKLE_PRIVATE_KEY` 是用来给 appcast 签名的私钥——有了私钥才能发布被应用信任的更新。如果私钥泄露，攻击者可以伪造更新包。

### <a name="领域-skill"></a>领域 skill
> 聚焦于一个开发领域（如"本地化"、"发布流程"、"调试"）的 skill。Agent 工作时加载对应领域的 skill，获取该领域的专业知识和约束规则，而不是把所有知识塞进 CLAUDE.md。领域 skill 的好处是：每个 skill 保持在 500 行以内，上下文窗口利用率高；缺点是用户需要知道该加载哪个 skill。

### <a name="规范漂移"></a>规范漂移
> 当同一件事的规范分散在多个文件里维护，随着时间推移这些文件的说法开始出现矛盾的现象。本案例的例子：CLAUDE.md 说"不要手动编辑 changelog.mdx"，而 release.md 说"更新 changelog.mdx"。防御方法：确保规范只有一个权威来源，其他文件引用它而不是复制它。
