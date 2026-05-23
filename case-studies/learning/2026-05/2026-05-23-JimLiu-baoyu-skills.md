# JimLiu/baoyu-skills — 学习案例

**仓库**：https://github.com/JimLiu/baoyu-skills
**Stars**：16,303 | **来源**：upstream audit
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-23（基于当前 HEAD）
**主题标签**：`single-purpose`, `manifest-discipline`, `examples-driven`, `experience-accumulation`, `fallback-chain`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
baoyu-skills 是 JimLiu（AI 工具布道者，YouTube/微信公众号"宝玉"）打造的 Claude Code [插件](#插件) 集合，专注于**内容创作者的自动化发布流水线**：图文生成、社交平台发布（微信/微博/X/小红书）、格式转换、视频摘要。当前版本 **v1.117.3**，22 个面向用户的 skill + 1 个内部 release skill，通过 TypeScript/Bun 运行时驱动外部工具（CDP、图像 API、本地二进制）。作者在 GitHub 有约 10.5 万 followers，是中文 AI 工具社区的重要布道者，这个仓库是其实际使用的个人工作流。

### 1.2 架构剖析
- **目录结构**：
  ```
  baoyu-skills/
  ├── skills/              # 22 个面向用户的 skill（SKILL.md + scripts/）
  ├── .claude/skills/      # 1 个内部 skill（release-skills）
  ├── packages/            # 共享 TS 工具包
  ├── scripts/             # 仓库维护脚本（sync、hooks、publish）
  ├── docs/                # 作者参考文档（不进 SKILL.md）
  └── .claude-plugin/      # marketplace.json（插件清单）
  ```
- **文件类型分布**：22 个面向用户 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook（运行时全走 TypeScript scripts）
- **编排关系**：完全平铺，每个 skill 自包含（有 SKILL.md + scripts/main.ts）。技能之间无直接编排，由用户在对话中手动组合。`baoyu-image-gen` 和 `baoyu-xhs-images` 已废弃，指向后继者 `baoyu-imagine` 和 `baoyu-image-cards`。
- **跨件契约**：`CLAUDE.md` 明确要求"每个 skill 必须自包含，不得引用 docs/ 或兄弟 skill 的路径"。[frontmatter](#frontmatter) 字段含 `name`、`version`、`metadata.openclaw.homepage`、`metadata.openclaw.requires`。版本与主仓库保持同步（CLAUDE.md：每次 tag 时所有 skill 同步版本号）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：约定优于配置 + 技能自包含发布。每个 skill 是一个独立的"能力单元"，可以从仓库中拆出单独使用，这推动了 CLAUDE.md 中严格的自包含约束。
- **解决什么问题**：内容创作者的发布工作流分散、平台 API 复杂、每次都要重复相同的操作序列。baoyu-skills 把这些重复工序封装成一句话触发的 skill。
- **做了什么 trade-off**：选择了 TypeScript + Bun 而非纯 Markdown prompt，换来了强大的系统集成能力（CDP 浏览器自动化、本地二进制调用），但代价是 skill 必须有运行时，纯 AI-only 环境无法使用。
- **反映什么认知模型**：作者把 AI agent 视为工具调度器，而非纯语言推理引擎。SKILL.md 是"工具说明书"，核心逻辑在 TypeScript 里，AI 负责解析用户意图和错误处理，程序负责稳定执行。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[NL 表皮 + 原生二进制核心](#nl-表皮--原生二进制核心)」变体 — "NL 表皮 + TypeScript 运行时核心"**

每个 SKILL.md 是简短的意图声明和 trigger 规则，核心业务逻辑（CDP 自动化、API 调用、图像处理）在 TypeScript scripts 里。SKILL.md 只告诉 AI"何时用、如何调用脚本、如何解释结果"。

模式特征清单：
- **意图层（SKILL.md）**：自然语言描述 + trigger 词 + 脚本调用方式
- **执行层（scripts/main.ts）**：强类型 TS，处理所有 I/O、错误、重试
- **运行时（Bun）**：统一的脚本执行环境，比 node 启动更快
- **自包含分发**：每个 skill 目录可独立提取，包含 SKILL.md 和所有依赖脚本
- **EXTEND.md 扩展点**：用户级个性化配置（API key、偏好）通过 EXTEND.md 注入，不污染 skill 本体

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要调用系统 API 或外部工具（CDP、REST API、本地二进制） | ✅ 高度适用 | NL 表皮解决意图解析，TS 核心解决可靠执行 |
| 内容创作 / 社交媒体自动化 | ✅ 高度适用 | 正是本仓库的甜点 |
| 纯语义推理任务（代码审查、文档生成） | ❌ 不适用 | 用纯 Markdown skill 更轻量，不需要 TS 运行时 |
| 需要跨 skill 编排、有复杂依赖的工作流 | ⚠️ 部分适用 | 此仓库没有 orchestrator，跨 skill 组合靠用户手动，适合独立任务但不适合强依赖链 |
| 离线或受限网络环境 | ❌ 不适用 | 多个 skill 依赖外部 API，`baoyu-imagine` 无 API key 无法运行 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + TS 运行时（本仓库） | baoyu-skills | 系统集成能力强，执行可靠 | 需要 Bun 运行时，非 AI-only |
| 纯 NL skill（无脚本） | cavekit、drama-workshop-skills | 零运行时依赖，部署极简 | 依赖 AI 的文本能力，无法稳定调用外部工具 |
| agent + skill 分层 | wshobson/agents | 多 agent 协作可处理复杂任务 | 设计复杂，调试困难 |

### 2.4 改进空间
1. **当前问题**：废弃 skill（baoyu-image-gen、baoyu-xhs-images）仍存在于 repo，且 baoyu-xhs-images 的 EXTEND.md 路径错误引用了 baoyu-image-cards 目录。**改进做法**：将废弃 skill 迁移到 `deprecated/` 目录并在 [manifest](#manifest) 中移除注册，或添加 `deprecated: true` frontmatter 字段让 loader 自动过滤。**预期收益**：消除配置命名空间污染，用户不会意外读写错误路径。
2. **当前问题**：package.json 的运行时依赖使用 `^` semver 范围。**改进做法**：用 `npm ci` 强制锁文件安装，或 CI 中定期 `npm audit`。**预期收益**：减少供应链风险，版本可重复。
3. **当前问题**：多个 skill 没有完成汇报步骤（post-to-x、post-to-weibo 等）。**改进做法**：统一在 scripts/main.ts 末尾输出 JSON 格式的完成报告，SKILL.md 里 step N："脚本完成后会输出：平台名称/操作结果/下一步用户行动"。**预期收益**：用户知道 skill 成功了什么，AI 有可以引用的结构化输出。

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/baoyu-image-gen/SKILL.md | 82 | 废弃标记嵌入 description；无完成汇报步骤 |
| skills/baoyu-xhs-images/SKILL.md | 82 | 废弃标记；EXTEND.md 路径引用错误 skill 目录 |
| skills/baoyu-post-to-weibo/SKILL.md | 87 | 无完成汇报步骤 |
| skills/baoyu-post-to-x/SKILL.md | 87 | 无完成汇报步骤 |
| skills/baoyu-youtube-transcript/SKILL.md | 87 | Workflow 步骤编号 1,2,3,3,4（重复） |
| .claude/skills/release-skills/SKILL.md | 96 | "meaningful description" 模糊量词×2 |

### 3.2 当时值得借鉴的模式
1. **技能自包含原则** → 根本原因：技能可被单独提取使用，不依赖仓库其他文件。示例：CLAUDE.md 明确写"Never link from SKILL.md to files outside the skill's own directory" → 借鉴：在自己项目中为每个 skill 写同样的约束，防止开发时习惯性引用 `../../shared/`
2. **EXTEND.md 用户配置扩展点** → 根本原因：将用户个性化配置（API key、偏好）从 SKILL.md 中分离，避免 skill 核心逻辑被用户配置污染。→ 借鉴：凡是需要用户配置的 skill，增加 EXTEND.md 机制
3. **运行时选择降级链** → CLAUDE.md 里 `if bun → bun; elif npx → npx -y bun; else error`。明确的 [降级路径](#降级路径)，不让 AI 猜。→ 借鉴：每个需要运行时的 skill 写明 fallback 顺序
4. **废弃 skill 指向后继者** → `baoyu-image-gen` 的 description 里写明 "use baoyu-imagine"。虽然有更优雅的做法，但明确指路好过无声废弃。
5. **`metadata.openclaw.requires` 声明硬依赖** → frontmatter 里声明 `anyBins`，让 skill loader 在安装时提前检测依赖，而不是运行时报错。

### 3.3 当时的缺陷
1. **问题：Workflow 步骤编号重复（baoyu-youtube-transcript）** → 为什么失败：AI 遵照数字顺序执行，步骤 3 出现两次时会跳过保存逻辑或错序执行。根本原因是手写步骤列表没有自动验证机制，改一行时影响了后续编号而没有发现。→ 自查：我的 skill 里有没有手写步骤列表？有多少个 skill 最近有过步骤插入/删除？
2. **问题：废弃 skill 的 EXTEND.md 路径指向错误 skill 目录** → 为什么失败：`baoyu-xhs-images` 的 Step 0 里搜索 `.baoyu-skills/baoyu-image-cards/EXTEND.md`（后继者的目录），安装了 xhs-images 的用户会意外创建/读取 image-cards 的配置文件夹，即使没安装 image-cards。这是"自包含原则"与"迁移方便性"之间 trade-off 处理不当。→ 自查：我有没有在废弃 skill 里引用新 skill 的路径？
3. **问题：多个 skill 缺少完成汇报步骤** → 为什么失败：`baoyu-post-to-x` 运行后脚本打开了浏览器，但 AI 不知道是否成功，用户也没有结构化反馈。没有完成步骤 = agent 无法判断"任务完成了吗"。根本原因是脚本执行即结束，没有回调给 SKILL.md 的约定。→ 自查：我的发布类/执行类 skill 有没有明确的"完成后输出什么"规范？

### 3.4 当时的优化机会
1. **废弃 skill 处理**：用 `deprecated: true` frontmatter 字段代替在 description 里嵌入废弃通知，保持 description 字段干净
2. **统一完成汇报格式**：所有 skill 添加 Step N（完成汇报）：列出操作对象、结果状态、用户下一步
3. **步骤编号自动验证**：在 CI 里加一个简单脚本，检查所有 SKILL.md 的 numbered list 是否连续递增

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| baoyu-youtube-transcript 步骤编号重复（1,2,3,3,4） | `grep -n "^[0-9]\." skills/baoyu-youtube-transcript/SKILL.md` | **仍存在**（第 141-142 行仍为两个"3."） | 作者 1 个月内未 fix，此 bug 优先级低但影响 agent 执行 |
| baoyu-xhs-images EXTEND.md 路径错误 | `grep "baoyu-image-cards" skills/baoyu-xhs-images/SKILL.md` | **仍存在**（第 315-317 行仍引用 baoyu-image-cards 路径） | config 命名空间污染问题未解决 |
| package.json unpinned semver（^） | `grep '"\^"' package.json` | **仍存在**（sharp、pdf-lib、pptxgenjs 等均用 ^） | 供应链风险依然存在 |

### 4.2 架构演进
当时审计时有 22 个 skill，现在目录里仍是 22 个——但出现了一个全新的 `baoyu-wechat-summary`（v1.117.3），而 CHANGELOG 中未显示有旧 skill 删除。版本号从 audit 时约 v1.56.x 到现在 v1.117.3，说明几周内有大量迭代，但迭代集中在功能增加（新增 wechat-summary）而非 bug 修复。

### 4.3 新增的可学习模式
**baoyu-wechat-summary** 展示了一个很好的新模式：`metadata.openclaw.requires.anyBins: [wx]`——用 frontmatter 声明"需要本地二进制 `wx`"，让 skill loader 在安装时就提示用户安装 wx-cli，而不是运行时才报错。这种"硬依赖前置声明"模式在当时审计的文件里也有，但在这个新 skill 里写得最清晰：`anyBins` 字段 + frontmatter 描述里的 URL 说明在哪里安装依赖。

---

## 五、校准

### 5.1 我已经在做对的
1. **echo-sleuth-for-claude 的技能自包含**：每个 skill 目录下只引用自己目录内的文件，没有跨 skill 引用——和 baoyu-skills 的自包含原则一致
2. **drama-workshop-skills 的运行时降级处理**：scripts/export_docx.py 中有明确的依赖安装检测和错误信息，类似 baoyu-skills 的 Bun 检测降级链
3. **claude-for-legal 的 trigger 词描述**：每个 SKILL.md 有清晰的 "Use when..." 触发条件——与 baoyu-skills 的 description 触发词设计相同

### 5.2 挑战 / 验证
这次案例**验证**了一个我曾犹豫的做法：把废弃 skill 保留在仓库里但标注 deprecated，而不是立即删除。baoyu-skills 保留了 baoyu-image-gen 和 baoyu-xhs-images，并通过 description 指向后继者，确保了老用户的配置兼容性。但 baoyu-xhs-images 的 EXTEND.md 路径问题说明，这种"指向后继者"的迁移方案必须保持配置路径隔离——不能直接复用新 skill 的配置目录。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 是否有步骤编号断续或重复
grep -rn "^[0-9]\+\." ~/.claude/skills/*/SKILL.md /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null \
  | awk -F: '{print $1}' | sort -u \
  | while read f; do
      prev=0; grep -n "^[0-9]\+\." "$f" | while IFS=: read line num rest; do
        cur=${num%%.*}
        if [ "$cur" != "$((prev+1))" ] && [ "$prev" -ne 0 ]; then
          echo "BROKEN: $f line $line (expected $((prev+1)), got $cur)"
        fi
        prev=$cur
      done
    done
# 命中后：手动修正步骤编号，确保 1,2,3... 递增无重复
```

```bash
# 检查我的技能是否引用了 skill 目录外的路径
grep -rn "\.\./\.\." /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null
# 命中后：将引用的共享内容 inline 进 SKILL.md，或说明引用为仓库作者专用（不进分发路径）
```

```bash
# 检查 claude-for-legal 中有多少 SKILL.md 有 allowed-tools 声明
grep -rl "allowed-tools" /tmp/my-repos/MarkQWu-claude-for-legal/*/skills/*/SKILL.md 2>/dev/null | wc -l
# 命中后：为未声明的 skill 添加 allowed-tools frontmatter（Read/Write/Edit/Bash 按需）
```

### 6.2 灵感 → 实施路径

1. **想法**：为 claude-for-legal 的 skill 加 `metadata.requires` 声明（类似 `anyBins`）
   - **为何可行**：legal skill 里有依赖本地工具（如 pandoc）或特定 MCP 的 skill，提前声明比运行时报错好
   - **第一步**：挑选 1 个最常因依赖缺失失败的 skill（如 `chronology`），在 frontmatter 加 `metadata.requires.anyBins: [pandoc]`，测试是否影响 loader——约 15 分钟

2. **想法**：为 drama-workshop-skills 的执行类命令加完成汇报格式规范
   - **为何可行**：目前 `/导出` 命令结束后 AI 说"完成了"，但没有结构化输出（导出路径、集数、格式）
   - **第一步**：在 `short-drama/SKILL.md` 的 `/导出` 步骤末尾加"输出格式：文件路径 | 集数 | 格式 | 文件大小"——约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 JimLiu/baoyu-skills 的核心目的**：内容创作者自动化工作流（图文生成、社交平台发布、格式转换）
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为内容创作工具；同用 skill 分发；同有脚本执行层 | drama-workshop 聚焦剧本，baoyu 聚焦多平台发布；baoyu 用 TS/Bun，drama 用 Python | 高 |
| MarkQWu/claude-for-legal | 低 | 同为 skill 集合；同有多个独立 skill 并列 | 垂直领域完全不同；legal 无脚本执行层 | 低 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 Claude Code 插件；同有 skill 自包含设计 | 用途差异大（会话挖掘 vs 内容发布） | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 缺少完成汇报步骤 | `grep -c "完成\|complete\|summary" short-drama/SKILL.md` | drama-workshop-skills：`/导出`、`/分镜` 命令缺少结构化完成输出规范 | 中 |
| 模糊量词（"relevant", "obvious"） | `grep -n "relevant\|obvious\|sometimes" echo-sleuth-for-claude/skills/*/SKILL.md` | echo-sleuth：experience-synthesis/SKILL.md:118 "relevant sessions"；git-mining/SKILL.md:93 "relevant git commits" | 低 |
| 命令 frontmatter 缺 `allowed-tools` | `grep -L "allowed-tools" claude-for-legal/*/skills/*/SKILL.md` | claude-for-legal：151 个 SKILL.md 中只有 4 个声明了 allowed-tools，147/151 未声明 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/drama-workshop-skills` 的 `short-drama/SKILL.md`（`/导出`、`/分镜` 步骤结尾）→ 各加一行"完成输出：[文件路径] [集数] [操作结果]" → 约 15 分钟
- `MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md:118` → "relevant sessions" → 改为 "sessions matching the time range and keywords specified in the input" → 5 分钟
- `MarkQWu/claude-for-legal` → 挑 5 个最常用 skill 各加 `allowed-tools: Read Write Edit Bash`（按实际用量）→ 30 分钟

### 7.3 别人的更优方案

1. **领域**：`metadata.openclaw.requires` 硬依赖前置声明
   - **本案例做法**：frontmatter 里 `metadata.openclaw.requires.anyBins: [wx]`，让 skill loader 安装时检测 → `baoyu-wechat-summary/SKILL.md` 第 9-11 行
   - **我的项目现状**：`claude-for-legal` 的 skill 无任何依赖声明，依赖缺失只在运行时报错（`litigation-legal/skills/chronology/SKILL.md`）
   - **如何借鉴**：在 claude-for-legal 的需要外部工具的 skill frontmatter 加 `metadata.requires.anyBins: [...]`，参考 baoyu 格式

2. **领域**：EXTEND.md 用户配置扩展点机制
   - **本案例做法**：每个 skill 定义三级 EXTEND.md 搜索路径（项目级/XDG/用户 home），用户修改 EXTEND.md 定制行为，不改 SKILL.md 本体
   - **我的项目现状**：`drama-workshop-skills` 没有这种机制，用户配置只能改 SKILL.md 本体，影响升级
   - **如何借鉴**：在 `short-drama/SKILL.md` 的 Step 0 加 EXTEND.md 查找逻辑，把用户可定制项（默认集数、出海 vs 国内模式等）移到 EXTEND.md

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：版本号管理
- **我的做法**：`drama-workshop-skills` 有 `release-manifest.json`，CI 里有 release-gate 流程，版本号变更有自动验证
- **本案例做法（弱在哪）**：baoyu-skills 的版本同步依赖人工：CLAUDE.md 要求"每次 tag 时所有 skill 同步版本号"，但没有 CI 强制验证，实际已出现 `baoyu-image-gen`（v1.56.4）和 `baoyu-imagine`（v1.58.0）版本漂移
- **意义**：drama-workshop-skills 的 release-gate 机制是真实优势，若有机会给上游 PR，可以建议添加版本一致性 CI 检查

---

## 八、术语表

### <a name="插件"></a>插件（Plugin）
> Claude Code 里的"技能包"。一个插件可以包含多个 skill、command 和 agent，通过 `claude plugin install` 一次性安装，用户之后用斜杠命令触发里面的功能。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据（name、description、version 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 决定如何注册这个 skill。

### <a name="manifest"></a>manifest
> 项目清单文件（如 `marketplace.json`、`plugin.json`），告诉插件系统这个项目包含哪些 skill/command/agent。如果 manifest 里漏写了某个文件，那个文件即使在磁盘上也不会被加载。

### <a name="nl-表皮--原生二进制核心"></a>NL 表皮 + 原生二进制核心
> 一种设计模式：自然语言的 SKILL.md 只负责意图解析和触发规则，真正的执行逻辑交给用 Go/Rust/C 或 TypeScript 写的程序完成。好处是执行可靠、可测试；坏处是依赖运行时环境。

### <a name="降级路径"></a>降级路径（Fallback Chain）
> 当首选方法不可用时，系统按顺序尝试备选方案的逻辑。如 baoyu-skills 的"先找 bun，没有就用 npx，都没有就报错安装说明"。明确的降级路径让 skill 在不同环境下都能给出有意义的反馈而不是神秘报错。

### <a name="EXTEND.md"></a>EXTEND.md
> baoyu-skills 发明的用户级配置文件机制。安装 skill 后，系统会在三个位置按优先级查找 EXTEND.md（项目级 > XDG 配置目录 > 用户 home），用来存储用户个人偏好（API key、输出格式等），不需要修改 SKILL.md 本体。
