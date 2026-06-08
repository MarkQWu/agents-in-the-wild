# 0xmariowu/Autosearch — 学习案例

**仓库**：https://github.com/0xmariowu/Autosearch
**Stars**：13 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-08（基于当前 HEAD）
**主题标签**：`router-channels`, `experience-accumulation`, `fallback-chain`, `security-gate`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Autosearch 是一个面向 AI Coding Agent 的开源深度研究插件，核心价值是"自动选渠道 → 并行搜索 → 合成引用报告 → 跨会话学习"。作者 0xmariowu 采用 discovery_strategy: `awesome-list-citation` 进入 NLPM 审计池，可能是从某个 awesome-claude-plugins 列表引流发现的。截至审计时 13 stars，体量较小但架构复杂度在 Claude Code 插件生态中属于顶端。发布方式：`npx autosearch-ai` 安装 npm wrapper，内部用 pipx/pip 装 Python 核心。

关键事实：
- 70 个 SKILL.md 文件，分 channels/（40+ 个搜索渠道）、meta/（13 个元策略）、tools/（5 个工具）、router/（1 个路由器）
- 1 个命令文件：`commands/autosearch.md`
- 有完整的自学习闭环：每个 skill 目录旁有 `experience.md`，通过 [frontmatter](#frontmatter) 中的 `experience_digest` 字段被加载

### 1.2 架构剖析

**目录结构**（关键层级）：
```
autosearch/
  skills/
    router/SKILL.md          ← 唯一路由器，loads 14 个 group index
    channels/                 ← 40+ 个搜索渠道（每个是独立 SKILL.md）
      github/SKILL.md
      arxiv/SKILL.md
      zhihu/SKILL.md
      twitter/SKILL.md
      ...
    meta/                     ← 13 个元策略（图搜索计划、经验压缩、渠道选择…）
      graph-search-plan/SKILL.md
      experience-capture/SKILL.md
      channel-selection/SKILL.md
      ...
    tools/                    ← 5 个工具（fetch-jina、fetch-playwright…）
    references/groups/        ← 14 个 group index 文件（router 的 loads 目标）
commands/
  autosearch.md               ← 唯一用户入口命令
npm/bin/
  autosearch-ai.js            ← npm 安装器 wrapper
scripts/                      ← 安装脚本、验证脚本、E2B 基准测试
.claude-plugin/plugin.json    ← manifest
```

**文件类型分布**：70 个 SKILL.md / 1 个 command / 0 个 agent / 7 个 shell 脚本 / 38 个 Python 脚本 / 1 个 JS 脚本

**编排关系**：三层结构。
1. 用户输入触发 `commands/autosearch.md`（唯一命令）
2. 命令调用 `router/SKILL.md`，router 通过 `loads:` 加载 14 个 group index 文件，并按查询意图选择最相关的 group
3. 命令再按需加载对应的 channel/meta/tool SKILL.md，执行搜索并合成结果

**跨件契约**：
- `frontmatter` 中的 `experience_digest` 字段 → 指向同目录下 `experience.md` 文件 → 由 `autosearch/skills/experience.py` 的 `load_experience_digest()` 函数在运行时加载
- `router/SKILL.md` 的 `loads:` 字段 → 14 个 group index 文件（路径相对于 skill 目录）
- `commands/autosearch.md` 声明 `allowed-tools:` 列举所有 MCP 工具（`mcp__autosearch__*`）

### 1.3 设计思路 / 方法论

**核心设计哲学**：「懒加载路由 + 经验侧车」。70+ 个 SKILL.md 不可能在每次对话开始时全量加载（token 成本极高），所以引入轻量 router 先按意图缩小范围，只加载命中的 group 内的 skill。

**解决什么问题**：AI Agent 搜索时不知道"该用哪个平台"——用 xiaohongshu 还是 reddit？用 arxiv 还是 pubmed？Autosearch 通过结构化的渠道 skill（每个渠道有自己的 When to Choose / How To Search / Known Quirks / fallback_chain）把这个判断从模型的隐性知识变成显式文档。

**做了什么 trade-off**：
- 70 个文件 vs 统一大文件：可维护性胜出，但带来了一致性维护成本（experience_digest 路径、方法声明规范需要全局一致）
- npm wrapper（跨平台）vs Python 直装：降低了安装门槛，但 npm 包中的 curl|bash 引入了安全风险（见第三章）

**反映什么认知模型**：作者把每个搜索渠道看作一个「有专业知识的工具人」——它知道自己的强项（学术论文 vs 中文社区 vs 短视频），有自己的失败模式（quirks），有自己的降级路径（fallback_chain）。这种拟人化建模让 AI 能做出更靠近领域专家的渠道选择。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名**：Router + Channels 分层架构（带经验侧车）

Router 是轻量入口，只做路由判断，不做实质工作；Channel skill 是领域专家，只做特定渠道的搜索。两者通过 group index 文件解耦。经验侧车（`experience_digest`）让每个渠道 skill 能从历史使用中学习，跨会话积累。

模式特征清单：
- **单入口单命令**：用户只看到 `autosearch` 一个命令，内部复杂性全隐藏
- **Router 先于 Channel**：router 按意图缩小范围，避免全量加载
- **Skill 内聚**：每个 channel skill 封装「选择条件 + 搜索方法 + 已知坑 + 降级路径 + 经验摘要」
- **方法声明制**：frontmatter 中明确声明 `methods` 和 `fallback_chain`，体内描述必须和声明一致
- **经验侧车分离**：学习数据（experience.md）和规范文档（SKILL.md）分离，互不污染

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多渠道/多来源的搜索或数据聚合 | ✅ 高度适用 | Router+Channels 天然适配"选渠道"的决策结构 |
| 需要跨会话积累经验的工具 | ✅ 高度适用 | experience_digest 字段即插即用 |
| 渠道数 < 5 的简单插件 | ❌ 不适用 | 过度工程，直接平铺技能即可 |
| 安全敏感环境（企业内网） | ⚠️ 慎用 | npm wrapper 含 curl 安装路径，需替换 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Router + Channels（本仓库） | Autosearch | Token 效率高；渠道扩展无需改 router | 一致性维护成本高（70 个文件需同步规范） |
| 平铺多 skill | review-squad | 简单直观；文件少好维护 | 渠道多后加载 token 激增 |
| 单体大 skill | 很多简单插件 | 极简；无跨件一致性问题 | 文件超 500 行后可读性崩溃 |

### 2.4 改进空间
1. **当前问题**：experience_digest 路径规范（`experience.md` vs `experience/experience.md`）靠人工维护，已出现 2 处偏差。**改进做法**：在 CI 中加一条 `grep -rn 'experience_digest' autosearch/skills | grep -v ': experience.md$'` 检查，自动拦截偏差。**预期收益**：消除运行时 experience 加载失败的隐患。
2. **当前问题**：commands/autosearch.md 没有 `name:` 字段（NLPM 无法注册命令）。**改进做法**：在 [manifest](#manifest) 的 CI 验证步骤中加 `name:` 字段检查。**预期收益**：防止未来新命令也出现同样遗漏。
3. **当前问题**：5 个 channel skill（github, youtube, zhihu, arxiv, bilibili）的 How To Search 标注了"(Planned)"，实际实现未完成。**改进做法**：要么实现，要么用 `status: planned` frontmatter 字段标注，从文档中移除虚假的行为描述。**预期收益**：用户不会对功能预期落空感到困惑。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**。安全评定：**BLOCKED**（高危 curl|bash）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 50 | 非 NL artifact 格式，无 frontmatter |
| commands/autosearch.md | 70 | 缺少 `name` 字段 |
| channels/twitter/SKILL.md | 88 | 缺 When to Choose / How To Search / Known Quirks |
| channels/tieba/SKILL.md | 88 | How To Search 仅一句话，中英混用 |
| channels/linkedin/SKILL.md | 88 | 正文极简，三句话 |
| router/SKILL.md | 97 | Clean |
| meta 类 skill | 96-97 | Clean |

### 3.2 当时值得借鉴的模式

1. **`model_tier: Fast` 字段** → router skill 明确声明用轻量模型做路由判断，不浪费 Sonnet/Opus 算力在分流决策上。根本原因：路由是分类问题，不需要推理能力。示例路径：`autosearch/skills/router/SKILL.md`。借鉴：凡是"判断该用哪个 skill"的逻辑，都可以是 Fast 模型。
2. **fallback_chain 声明** → frontmatter 中显式列出降级路径（如 `via_tikhub` 失败后用 `via_cookie`），而不是在正文里含糊说"如果失败就换一种方法"。根本原因：可验证的降级路径让 NLPM 能检测体内实现和声明的一致性。借鉴：任何有网络依赖的 skill 都应该声明 fallback_chain。
3. **group index 分层** → 14 个 group index 文件把 40+ channel skill 按领域（学术/中文/代码包/视频…）分组，router 只加载命中的 group，不加载全部。根本原因：避免每次对话加载 70 个文件导致的 token 浪费。借鉴：当 skill 数超过 10 个时，考虑引入 group index 层。
4. **experience_digest + experience.py 闭环** → 每个 channel skill 运行后可以把经验写入 `experience.md`，下次加载 skill 时 experience.py 自动注入。根本原因：让 AI 从实际使用中学习渠道特性，而不只靠静态文档。借鉴：任何频繁运行且结果有规律的工具都可以引入经验侧车。
5. **test fixture 明确标注** → `tests/fixtures/skills/` 下的 SKILL.md 在审计时被标记为"test fixture，有意省略 Quality Bar"，不计入主评分。根本原因：测试文件不应被当作产品文件评分。借鉴：测试 fixture 中可以明确写 `# This is a test fixture` 避免误判。

### 3.3 当时的缺陷

1. **commands/autosearch.md 缺 `name:` 字段** → 根本原因：写命令时漏掉了 frontmatter 必填字段，NLPM 扫描器无法注册此命令。自查：我的 echo-sleuth 命令文件中 `recall.md`、`recap.md`、`timeline.md`、`lessons.md` 都有完整 frontmatter，但确实存在其他遗漏。
2. **experience_digest 路径不统一**（instagram/wechat_channels 用 `experience/experience.md`，其他 50+ 个 skill 用 `experience.md`）→ 根本原因：新增 skill 时手工复制 frontmatter，忘记对齐路径规范。自查：我的仓库目前无 experience_digest 机制，不存在此问题，但提醒我如果引入类似机制需要在创建时 CI 检验。
3. **npm wrapper 中 curl|bash 安全漏洞** → `npm/bin/autosearch-ai.js` 原来在失败时会执行 `bash -c "curl ... | bash"` 安装 autosearch，没有哈希验证。根本原因：为降低安装门槛（npm 一行命令）而妥协了安全性，供应链风险不可忽视（GitHub CDN 被劫持 = 用户系统被入侵）。自查：我的 drama-workshop-skills 的 `install.sh` 同样需要检查是否有类似远程执行行为。

### 3.4 当时的优化机会

1. **channels/twitter、tieba、linkedin SKILL.md 补全** → 这三个渠道没有"When to Choose"/"How To Search"章节，信息量不足以让 AI 正确选择和使用它们。
2. **将"(Planned)"标注的 5 个渠道从 How To Search 中移除** → github、youtube、zhihu、arxiv、bilibili 的 How To Search 标了"(Planned)"，但未实现的功能出现在正文中会让用户产生虚假期望。
3. **对齐 experience_digest 路径** → 2 个偏差文件需要修正到 `experience.md`，同时在 CI 中加检查防止回归。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| commands/autosearch.md 缺 `name:` 字段 | `head -10 commands/autosearch.md` 查 frontmatter | **仍存在**：frontmatter 有 description 和 allowed-tools，但无 name 字段 | 命令注册 bug 未修复，NLPM 仍无法扫描到此命令 |
| experience_digest 路径不统一 | `grep -n "experience_digest" instagram/SKILL.md wechat_channels/SKILL.md` | **仍存在**：两个文件 L23 仍为 `experience/experience.md` | 运行时 experience 加载可能静默失败 |
| curl\|bash 安全漏洞（High） | `grep -n "curl" npm/bin/autosearch-ai.js` | **部分修复**：curl\|bash 现在只出现在 `printAutosearchNotFoundHint()` 的错误提示字符串中（作为给用户看的安装建议文本），不再作为执行路径；但字符串仍在代码里，不能排除未来回归 | 安全风险已缓解，但代码中仍存在 curl\|bash 字符串，需确认执行路径是否彻底移除 |

### 4.2 架构演进

从 audit 快照到当前 HEAD，主要变化：
- `commands/autosearch.md` 的 `allowed-tools` 被大幅扩充（现在包含 `mcp__autosearch__*` 系列工具），说明作者在工具接口层做了重构，把搜索功能从 Claude 直接执行转向了 MCP 协议
- npm wrapper 的安全模型有所调整（curl|bash 变为提示文本而非执行路径），说明作者意识到了安全问题
- plugin.json 版本号更新到 `2026.04.26.2`，说明有持续迭代

### 4.3 新增的可学习模式

当前 HEAD 引入但 audit 未覆盖的新模式：
- **MCP 工具协议化**：`commands/autosearch.md` 的 `allowed-tools` 中全部是 `mcp__autosearch__*` 格式的 MCP 工具，而非直接调用 bash 或 python。这是一种把搜索能力封装成 MCP 服务再通过命令调用的设计——安全边界更清晰，工具接口更可测试。

---

## 五、校准

### 5.1 我已经在做对的

1. **命令文件有完整 frontmatter**：echo-sleuth 的所有命令文件都有 `name`、`description`、`argument-hint` 字段，这个基本卫生 Autosearch 反而没做到。
2. **CLAUDE.md 作为架构文档**：我的仓库 CLAUDE.md 描述了整体架构（命令 → agents → skills 层级），和 Autosearch 的思路相似。
3. **Skills 有独立目录**：echo-sleuth 每个 skill 在独立目录下，和 Autosearch 的 channels/meta/tools 分组一致。
4. **argument-hint 字段完整**：echo-sleuth 命令有 argument-hint，Autosearch 反而在命令上缺少对空输入的显式处理。

### 5.2 挑战 / 验证

这次案例**验证**了我此前犹豫的一个做法：experience_digest / 经验侧车机制。我一直觉得"让 AI 把经验写回 skill 目录"太复杂，这个案例证明它的确有工程代价（路径一致性需要 CI 保障）。但它同时也证明了这个机制在大型多渠道插件中是必要的。结论：如果 skill 数 < 10，暂时不引入；如果规模上去了，从一开始就要把路径规范放进 CI。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令文件是否有遗漏 name 字段（对应 Autosearch 的 bug）
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands -name "*.md" -exec grep -L "^name:" {} \;
```
命中后怎么办：在命中的命令文件 frontmatter 中补充 `name:` 字段。

```bash
# 检查我的 SKILL.md 是否有模糊量词（对应 experience_digest 等规范性问题的前体）
grep -rn -E '\b(appropriate|comprehensive|relevant)\b' \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
```
命中后怎么办：替换为具体条件描述（比如"relevant → 包含命中关键词的会话"）。

```bash
# 检查 drama-workshop-skills 的安装脚本是否有 curl|bash 模式
grep -n "curl.*|.*bash\|bash.*curl" /tmp/my-repos/MarkQWu-drama-workshop-skills/install.sh 2>/dev/null
```
命中后怎么办：改用 SHA256 校验或 pipx/pip 直装，移除远程执行路径。

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 中为 skills 目录下的每个 skill 引入 group index 文件，当 skill 数超过 8 个时启用 router 层。
   - **为何可行**：echo-sleuth 目前有 2 个 skill，尚不需要，但未来如果扩展到多个 skill（如不同类型的会话分析），这个模式直接可复用。
   - **第一步**：先把 2 个现有 skill 的功能定位写清楚（When to Choose），再看是否需要 router。30 分钟内。

2. **想法**：drama-workshop-skills 的 `install.sh` 应该加 SHA256 验证。
   - **为何可行**：Autosearch 的反面教训证明 curl|bash 是供应链风险，我的 drama-workshop-skills 也有 install.sh。
   - **第一步**：打开 `install.sh`，确认是否有远程下载步骤，如果有，加 `sha256sum --check` 验证。30 分钟内。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 0xmariowu/Autosearch 的核心目的**：多渠道深度搜索 + 跨会话学习，帮助 AI Agent 自动选最优搜索渠道。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是"找信息"工具（echo-sleuth 找历史对话，Autosearch 找网络资料）；都有多个信息来源 | Autosearch 面向网络搜索，echo-sleuth 面向本地 JSONL；Autosearch 有 router+channels，echo-sleuth 平铺 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 都有安装脚本 | 目的完全不同（创意内容 vs 信息搜索） | 低 |
| MarkQWu/claude-for-legal | 低 | 都有多个专业域 skill | 目的不同（法律工作流 vs 搜索研究） | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| commands 缺 `name:` 字段 | `find commands -name "*.md" -exec grep -L "^name:" {} \;` | echo-sleuth 的 `recap.md`、`timeline.md`、`recall.md`、`lessons.md` 有 `name:` 字段（通过检查） | 无问题 |
| allowed-tools 缺失 | `grep -L "allowed-tools" commands/*.md` | echo-sleuth 的 `recap.md`、`timeline.md`、`recall.md`、`lessons.md` 确认无 `allowed-tools`（这 4 个只读 JSONL，应加 `allowed-tools: Bash`） | 中 |
| 模糊量词 "relevant" | `grep -rn "relevant" skills/*/SKILL.md` | echo-sleuth 的 `experience-synthesis/SKILL.md:118` 和 `git-mining/SKILL.md:93` 各 1 处 | 低 |

**命中后的具体行动建议**：
- `echo-sleuth/commands/recap.md` → 在 frontmatter 加 `allowed-tools: Bash`（因为要读取 ~/.claude/projects/ JSONL 文件）→ 5 分钟
- `echo-sleuth/commands/timeline.md` → 同上 → 5 分钟
- `echo-sleuth/skills/experience-synthesis/SKILL.md:118` → 将"relevant sessions"改为"包含该关键词或标签的历史会话" → 10 分钟

### 8.3 别人的更优方案

1. **领域**：懒加载路由（Router + group index）
   - **本案例做法**：router/SKILL.md 通过 `loads:` 声明 14 个 group index 文件，每个 group index 描述该组 skill 的功能，router 按意图只加载命中的 group，单次对话 token 消耗大幅降低。
   - **我的项目现状**：echo-sleuth 当前只有 2 个 skill，直接平铺，无 router 层；claude-for-legal 有大量 skill 但也是平铺（每个 practice 域独立加载）。
   - **如何借鉴**：当 claude-for-legal 的某个 practice 域 skill 数超过 5 个时，在该域入口引入一个 group-index.md；当 echo-sleuth 扩展到 5+ skill 时，用 router SKILL.md 做意图路由。

2. **领域**：`fallback_chain` 在 frontmatter 中显式声明
   - **本案例做法**：每个 channel skill 的 frontmatter 声明 `fallback_chain: [via_tikhub, via_cookie]`，body 中的方法描述必须和声明一致，NLPM 可以机械检测偏差。
   - **我的项目现状**：claude-for-legal 的 skill 有 fallback 描述，但全在 body 里，没有 frontmatter 声明。
   - **如何借鉴**：在 claude-for-legal 中对有网络/API 依赖的 skill 添加 `fallback_chain:` frontmatter 字段，并在 NLPM check 中验证 body 和声明的一致性。

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：命令的 argument-hint 文档
  - **我的做法**：echo-sleuth 每个命令都有详细的 `argument-hint:`（如 `[N-sessions-or-days] [--detail low|medium|high]`）
  - **本案例做法**（弱在哪）：`commands/autosearch.md` 没有 `name:` 字段，更没有 `argument-hint:` 字段
  - **意义**：命令接口文档做得更完整，用户体验和 NLPM 扫描结果都更优。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`allowed-tools`、`experience_digest` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能知道这个 skill 怎么注册和调用。少了 `name:` 字段，NLPM 扫描器就找不到这个文件。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，这里指 `.claude-plugin/plugin.json`——告诉 Claude Code 这个插件包含哪些 skill、命令、版本等。如果 manifest 里没有列出某个文件，那个文件即使在硬盘上也不会被插件系统加载。

### <a name="experience_digest"></a>experience_digest
> Autosearch 自定义的 frontmatter 字段，值是同目录下 `experience.md` 文件的相对路径。运行时由 `experience.py` 解析，把历史使用经验注入到当前对话的 skill context 中，实现跨会话学习。

### <a name="fallback_chain"></a>fallback_chain
> frontmatter 中声明的"降级路径列表"。例如 `fallback_chain: [via_tikhub, via_cookie]` 表示首选 `via_tikhub` 方法，失败后尝试 `via_cookie`。声明在 frontmatter 里比写在 body 里的优势：可以被工具机械验证，防止实现与声明不一致。
