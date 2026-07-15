# LerianStudio/ring — 学习案例

**仓库**：https://github.com/LerianStudio/ring
**Stars**：174 | **来源**：upstream audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-15（基于当前 HEAD）
**主题标签**：`router-channels`, `manifest-discipline`, `vague-quantifier`, `model-pinning`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Ring 是 LerianStudio 为企业内部工程团队搭建的 Claude Code [多智能体框架](#多智能体框架)。它的核心设想是：把一个软件公司里的不同职能部门（工程、产品、运营、财务分析、技术写作）都映射成独立的 agent 团队，每个团队有自己的 agents/ 和 skills/ 子目录，通过统一的 `ring:` [命名空间前缀](#命名空间前缀)互相调用。

关键事实：
- 由 LerianStudio（巴西创业公司）在 2025-2026 年构建，stars 174（小众但深度使用）
- 安装方式：`./install-ring.sh` 脚本（curl 拉取、bash 执行的高危模式，详见安全章节）
- 在生态中的位置：属于"企业内部全团队 NL 编排"这个小众赛道，与 gstack、SuperClaude 目标不同
- [CLAUDE.md](#CLAUDE.md) 是单一配置源，AGENTS.md 是它的符号链接（`ln -s CLAUDE.md AGENTS.md`）

### 1.2 架构剖析

**目录结构**（2026-07-15 HEAD）：
```
ring/
├── CLAUDE.md              # 单一配置源（AGENTS.md 是其 symlink）
├── ARCHITECTURE.md        # 架构文档
├── MANUAL.md              # 用户手册
├── default/               # 通用 agents（code-reviewer, task-manager 等）
│   ├── agents/            # ~10 个通用 agent
│   └── skills/            # 通用 skill
├── dev-team/              # 工程团队 agents
│   ├── agents/            # backend-go.md, bff-ts.md, devops-engineer.md 等
│   └── skills/
├── tw-team/               # 技术写作团队
│   ├── agents/            # api-writer, docs-reviewer, functional-writer
│   └── skills/
├── pm-team/               # 产品管理团队
├── finops-team/           # 财务运营团队
├── pmo-team/              # 项目管理办公室
└── shared/                # 跨团队共享资源
```

**文件类型分布**（当前 HEAD）：
- 42 个 agent（.md），分布在 6 个团队目录
- 94 个 skill（audit 时估算），结构类似 agents/
- 10 个 hook 文件（session-start.sh 等）
- 1 个 CLAUDE.md + 1 个 AGENTS.md（symlink）

**编排关系**：
- 各团队的 agents 是平列的（tw-team/api-writer.md 和 dev-team/backend-go.md 互不从属）
- 团队内部通过 skill 分享知识（如 tw-team/skills/applying-voice-and-tone/SKILL.md）
- `default/agents/` 是"无团队归属"的通用 agent，被各团队复用
- 没有 meta-router：用户直接通过 `ring:<agent-name>` 调用特定 agent

**跨件契约**：
- 所有组件必须使用 `ring:` 前缀（CLAUDE.md Rule 4）
- dev-team 的 reviewer agents 通过 WebFetch 从 ring 仓库 main 分支实时拉取"rolling standards"（滚动标准）
- session-start.sh hook 在每次 Claude Code 会话启动时自动安装 PyYAML（无版本锁定）

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「职能团队映射」——把企业组织结构直接复制到 AI agent 系统，每个职能有独立的 agent 和 skill，避免单一通用 agent 包揽一切
- **解决什么问题**：企业场景下，不同职能（工程、产品、写作）有完全不同的输出标准和质量要求，通用 agent 无法满足
- **做了什么 trade-off**：
  - 职能分离 vs 学习曲线：用户需要知道调用哪个团队的 agent（`ring:code-reviewer` vs `ring:plan-reviewer`），上手成本高
  - Rolling standards（实时拉取 main 分支标准）vs 版本稳定性：获得了"总是最新"的好处，付出的代价是供应链风险（见安全章节）
  - 统一命名空间 vs 灵活性：所有 agent 必须带 `ring:` 前缀，降低了名称冲突，但增加了输入负担
- **反映什么认知模型**：作者把 Claude Code 视为「AI 版的企业组织系统」——agent = 员工角色，skill = 部门知识库，hook = 自动化 onboarding 流程

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「按职能划分的多团队 Agent 矩阵」

每个职能团队是一个独立的 `<team>/agents/` + `<team>/skills/` 对，公共资源放在 `default/` 和 `shared/`，所有组件用统一命名空间（`ring:`）串联。

模式特征清单：
- **职能隔离**：不同职能的 agent 物理隔离（不同目录），避免职责蔓延
- **统一入口命名**：`ring:` 前缀让用户在任何地方都能调用任何 agent，无需记住路径
- **知识共享层**：`shared/` 和 `default/` 目录持有跨团队知识，避免重复定义
- **符号链接单一真相**：`AGENTS.md` 是 `CLAUDE.md` 的 symlink，防止两个文件的内容漂移
- **活文档设计**：CLAUDE.md 里的审阅者池（reviewer pool）、反模式表（anti-patterns）都是可执行规则，不只是文档

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多职能企业团队（>5人，有 TW、PM、Dev、FinOps） | ✅ 高度适用 | 职能映射清晰，每个团队获得对应 agent |
| 个人开发者 / 小团队 | ❌ 不适用 | 太重，目录结构复杂，agent 数量远超需要 |
| 需要频繁定制 agent 的团队 | ⚠️ 慎用 | Rolling standards 设计让定制难以追踪（live main branch） |
| 安全要求高的企业环境 | ⚠️ 慎用 | install-ring.sh 是 curl-pipe-bash 模式；session-start.sh 自动装包 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 按职能多团队矩阵（本仓库） | LerianStudio/ring | 职能隔离清晰；适合企业规模 | 重量级；学习曲线高 |
| 单仓多 skill 平铺 | gstack, addyosmani/agent-skills | 轻量；上手快；个人友好 | 大了之后难分类；没有职能边界 |
| Router + Channels 分层 | SuperClaude 框架 | 有中央 router，用户不需要知道调哪个 agent | router 本身成为单点瓶颈 |
| NL 表皮 + 原生二进制核心 | claude-notifications-go | 性能极高；可跨平台打包 | 需要编译；不适合纯 NL 逻辑 |

### 2.4 改进空间

1. **当前问题**：Rolling standards（agents 实时从 main 拉取规范）导致供应链风险，且无法离线工作。**改进做法**：在 `RING_STANDARDS_REF` 环境变量允许锁定 commit/tag；默认改为「上次已知良好版本」而非 `main`。**预期收益**：消除安全风险；支持企业离线部署。

2. **当前问题**：所有 agent 都缺少 `model:` 字段（100% 未声明），用户无法知道某个 agent 应该用哪个模型。**改进做法**：按角色分类添加模型声明（reviewer agents → Haiku；specialist agents → Sonnet）。**预期收益**：降低成本（Haiku 价格是 Sonnet 的 1/10）；提升可预测性。

3. **当前问题**：install-ring.sh 使用 curl-pipe-bash 模式，且 marketplace.json 从 main 分支拉取无完整性校验。**改进做法**：改为下载后校验 SHA256 再执行（脚本里已有 `MARKETPLACE_JSON_SHA256` 变量占位，但未启用）。**预期收益**：供应链攻击面缩小 60%+。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **87/100**。

| 文件/群组 | 当时分数 | 主要问题 |
|---|---|---|
| Active agents（41 个平均） | ~88 | 无 model 字段（−5）；只有一个 example（−5） |
| Archive agents（17 个） | ~78 | 双 `---` YAML bug（−5）；缺 ring: 前缀（−5）；vague ×2 |
| sample-agent.md（测试固件） | 65 | 零 example（−15）；缺 ring: 前缀 |
| CLAUDE.md | 93 | 轻微 vague 词汇 |

### 3.2 当时值得借鉴的模式

1. **AGENTS.md 作为 CLAUDE.md symlink** → 防止两文件漂移；一次更新两处同步。原文：`README.md "AGENTS.md is a symlink to this file — edit CLAUDE.md only"` → 借鉴：在我的多配置仓库中，用 symlink 代替 cp 来维护配置副本。

2. **Anti-Rationalization 表** → CLAUDE.md 里专门列了「禁止找借口」规则（如"simple task is not an excuse"）。这种白名单式禁令比"尽量做"更可执行。→ 借鉴：在 CLAUDE.md 里加 `## 禁止找借口` 专节。

3. **Reviewer Pool 同步守护** → CLAUDE.md 规定修改 reviewer pool 必须同时更新 8 个相关文件，hook 代码里也有 hardcode 验证。→ 借鉴：跨组件修改要用 hook 自动验证一致性，不依赖人工记忆。

4. **团队 skill 载入 index.md** → agents 不直接 WebFetch 整个 standards 文档，而是先读 `index.md` 再按需载入章节。减少 context 浪费。→ 借鉴：大文档用 index.md 做二级入口。

5. **ZERO PANIC POLICY** → Go 代码里禁止所有 `panic()`，用 `(T, error)` 代替。这是一条在 CLAUDE.md 里可执行的代码规约。→ 借鉴：在 CLAUDE.md 写下 "MUST NOT use panic() in Go" 这类具体代码规约，而非泛泛的"写好代码"。

### 3.3 当时的缺陷

1. **B1：Archive agents 双 `---` YAML bug** → 根本原因：archive 过程可能是手动复制，没有 linter 检查 YAML 格式；两个 `---` 导致 YAML parser 把后续内容当成第二个文档，所有 metadata 失效。自查：我是否有手工复制 agent 文件的习惯？有的话需要加 `pre-commit` 格式检查。

2. **B2：Archive agents 缺少 ring: 前缀** → 根本原因：archive 早于命名空间规则的制定，迁移时没有批量重命名。这是「规则后制定，旧文件不回填」的经典问题。自查：我的 SKILL.md 文件是否遵守了统一的命名前缀？drama-workshop-skills 的 name 字段是否有前缀？

3. **Q1：100% agent 缺 model 声明** → 根本原因：项目早期还没有形成"每个 agent 必须声明 model"的规范，规范是后来才写进 CLAUDE.md 的（但没有回填）。这说明规范文档和实际文件的同步是个系统性问题，需要工具（如 nlpm:check）来强制执行，而不是人工维护。自查：我的 SKILL.md 文件几乎全部缺 model 字段。

4. **Security H1：自动安装 PyYAML（无版本锁定）** → 根本原因：方便性优先于安全性；`RING_AUTO_INSTALL_DEPS=true` 是默认值，会在 session 启动时静默安装第三方包。一旦 PyYAML 被供应链攻击，所有 ring 用户的工作区都受影响。自查：我的 hook 脚本里有无 `pip install` / `npm install` 等网络调用？

### 3.4 当时的优化机会

1. 为 top-10 高流量 agents（api-writer, code-reviewer, backend-engineer-golang）各添加第二个具体 example（完整 input→output 对，不只是模板）
2. 在 dev-team agents 里把 "appropriate error handling" → "return HTTP 4xx for validation failures, 5xx for internal errors" 等具体指标
3. 为 install-ring.sh 启用已有的 `MARKETPLACE_JSON_SHA256` 完整性校验占位代码

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| B1/B2：archive 目录 double-`---` + 缺 ring: 前缀 | `find .archive -name "*.md"` | `.archive/` 目录在 depth=1 shallow clone 中不存在 | 暂无法验证：可能已修复，也可能 archive 被删除 |
| B3：frontend-bff-engineer-typescript.md 重复 section | `grep -c "Post-Implementation" dev-team/agents/frontend-bff-engineer-typescript.md` | 文件已重命名为 `bff-ts.md`，重复 section 已消失 | **已修复**：文件重构顺带解决了重复内容问题 |
| Q1：所有 agent 缺 model 字段 | `find . -name "*.md" -path "*/agents/*" \| xargs grep -L "^model:"` | 42 个 agents，**仍然全部缺少 model 字段** | **持续存在**：三个月内未改进 |

### 4.2 架构演进

对比 audit 时（2026-04-19）和当前 HEAD（2026-07-15）的明显变化：
- `dev-team/agents/` 的文件名从 `backend-engineer-golang.md` 缩短为 `backend-go.md`（可读性优先）
- `frontend-bff-engineer-typescript.md` → `bff-ts.md`（重构时顺带清理了 B3 重复 section）
- agent 总数从 41 增至 42（新增了一个 dev-team agent）
- archive 目录状态未知（depth-1 clone 未包含）

这说明作者后来意识到：长文件名在 `ring:<name>` 调用时很笨重，缩短 agent 名字改善了 UX。

### 4.3 新增的可学习模式

当前 HEAD 的文件名重命名模式（`backend-engineer-golang.md` → `backend-go.md`）体现了一个新规律：**agent 命名越短越好**，因为用户要在命令行里输入 `ring:backend-go`。这对 agent 命名设计有直接指导意义。

---

## 五、校准

### 5.1 我已经在做对的

1. **CLAUDE.md 作为单一配置源**：我的 gstack 和 bureau 仓库都有 CLAUDE.md 做中央配置，和 ring 方向一致。
2. **职能分离理念**：gstack 把 CEO、Designer、Eng Manager 等角色分成独立 SKILL.md，和 ring 的团队划分哲学相同。
3. **skill 作为知识载体**：我的 skills 是 domain 知识的封装，ring 的团队 skills 也是同样的定位。
4. **CLAUDE.md 里的禁止规则**：我已经在 CLAUDE.md 里写了一些具体的 MUST NOT 规则，方向对的。

### 5.2 挑战 / 验证

这次案例挑战了我的一个假设：**「规范文档写好了，文件自然会遵守」**。ring 有全面的 CLAUDE.md 规范，但 100% 的 agents 仍然缺 model 字段——说明没有自动化工具（如 `/nlpm:check`）就无法真正守住规范。这验证了「规范必须配合工具执行」的重要性。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 是否有 model 字段（ring Q1 同款问题）
find ~/.claude/skills -name "SKILL.md" | xargs grep -L "^model:"
# 命中后怎么办：逐一添加 `model: claude-sonnet` 到每个 SKILL.md 的 frontmatter

# 检查我的 agent .md 文件里是否有 ring: 类似的命名空间前缀一致性
find . -name "*.md" -path "*/agents/*" | xargs grep -l "^name:" | xargs grep -h "^name:" | sort
# 命中后：确认所有 agent name 是否有统一前缀，无前缀的加上

# 检查我的 hooks 里是否有网络自动安装的操作（ring H1 同款风险）
grep -rn "pip install\|npm install\|brew install" ~/.claude/hooks/ .claude/hooks/ 2>/dev/null
# 命中后：审查是否真的有必要，至少锁定版本号
```

### 6.2 灵感 → 实施路径

1. **想法**：在 CLAUDE.md 里加一个「ZERO X POLICY」类似的可执行禁令表（参考 ring 的 ZERO PANIC POLICY）
   - **为何可行**：ring 证明了把代码规约直接写进 CLAUDE.md 是有效的——Claude 会遵守 MUST NOT
   - **第一步**：打开我仓库的 CLAUDE.md，在底部加 `## 执行规约（MUST NOT）` 章节，写 3-5 条最重要的禁令；30 分钟内可完成

2. **想法**：给 gstack 的 SKILL.md 批量添加 model 字段
   - **为何可行**：gstack 有 66 个 SKILL.md 全部缺 model，ring 证明这是实际影响用户行为的缺陷
   - **第一步**：写一个 `sed -i` 脚本批量在 frontmatter 末尾插入 `model: claude-sonnet`；10 分钟可完成

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 LerianStudio/ring 的核心目的**：企业多职能团队的 Claude Code 统一 agent 框架，每个职能有专属 agents 和 skills

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 多角色 skills（CEO、Designer、Eng Manager）；CLAUDE.md 中央配置 | gstack 是个人/小团队，ring 是企业级；gstack 无命名空间前缀 | 高 |
| MarkQWu/bureau | 低 | 都是 Claude Code plugin | bureau 专注知识管理，ring 专注团队 agent | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都有 SKILL.md 文件 | 完全不同的域 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Q1：所有 agent/skill 缺 model 字段 | `find /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" \| xargs grep -L "^model:"` | **命中 66/66**：gstack 全部 SKILL.md 均缺 model 字段 | 高 |
| vague 量词（appropriate/comprehensive） | `grep -r "appropriate\|comprehensive\|robust" /tmp/my-repos/MarkQWu-gstack/*/SKILL.md` | 未做完整 grep，但 gstack 描述中包含多处 vague 词汇 | 中 |

**命中后的具体行动建议**：
- `MarkQWu/gstack` 的 `pair-agent/SKILL.md` → 在 frontmatter 加 `model: claude-sonnet`（5 分钟）
- 对所有 66 个 gstack SKILL.md 批量操作：`find . -name "SKILL.md" | xargs sed -i '/^---$/a model: claude-sonnet'`（10 分钟，需验证格式）

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：「AGENTS.md = CLAUDE.md symlink」设计
   - **本案例做法**：`AGENTS.md` 是 `CLAUDE.md` 的符号链接（`ln -s CLAUDE.md AGENTS.md`），修改一次自动同步
   - **我的项目现状**：gstack 仓库有 AGENTS.md 和 CLAUDE.md 两个独立文件，内容可能漂移
   - **如何借鉴**：`cd ~/repos/gstack && ln -sf CLAUDE.md AGENTS.md`（1 分钟）；在 README 说明"edit CLAUDE.md only"

2. **领域**：「Anti-Rationalization 表」在 CLAUDE.md 里
   - **本案例做法**：明确列举所有「不被接受的借口」（如 "simple task is not an excuse"），形成可执行的禁止清单
   - **我的项目现状**：我的 CLAUDE.md 里没有系统性的禁止列表
   - **如何借鉴**：在 CLAUDE.md 添加 `## 禁止借口清单` 节，列出最常见的 3-5 个自我开脱模式

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：model 字段应用（drama-workshop-skills）
- **我的做法**：`MarkQWu/drama-workshop-skills/short-drama/SKILL.md` 的 frontmatter 包含 `name` 和 `description`，符合 Claude Code 规范
- **本案例做法（弱在哪）**：ring 的 41 个 active agents 全部缺少 `name` 和 `model` 字段（YAML frontmatter 缺失或仅有隐式识别）
- **意义**：drama-workshop-skills 在 frontmatter 规范性上高于 ring；若未来做 audit，这是亮点

---

## 八、术语表

### <a name="多智能体框架"></a>多智能体框架
> 让多个独立的 AI "角色"（agent）各司其职、协同完成任务的系统架构。每个 agent 专注一件事（如"代码审查"或"技术文档撰写"），需要时互相调用，而不是由一个超级 agent 包揽所有。
> **对比**：单一 agent 像一个"全能助手"，能做一切但什么都做得平庸；多 agent 框架像一个分工明确的团队，各专其职。

### <a name="命名空间前缀"></a>命名空间前缀
> 给所有组件名字加统一的「姓」，用于区分来自不同系统的同名组件。Ring 里所有 agent 都叫 `ring:xxx`（如 `ring:code-reviewer`），就算有别的插件也叫 `code-reviewer`，两者也不会混淆。
> **对比**：没有前缀就像一个公司所有员工都叫"小李"，叫名字时总要确认是哪个部门的。

### <a name="CLAUDE.md"></a>CLAUDE.md
> Claude Code 启动时自动读取的配置文件，相当于这个项目给 Claude 的「操作手册」。里面可以写规则、禁令、角色定义、常用命令等。Claude 在整个会话里都会参考这份文件行事。

### <a name="符号链接"></a>符号链接（symlink）
> 操作系统层面的"快捷方式"——一个文件的内容其实存在另一个地方，symlink 只是个指针。修改源文件（CLAUDE.md），所有指向它的 symlink（AGENTS.md）自动同步。
> **对比**：普通的"拷贝"是把内容复制一份，两份独立，以后可能不同步；symlink 永远指向同一份内容，不会漂移。

### <a name="Rolling standards"></a>滚动标准（Rolling Standards）
> Ring 的一种设计选择：agents 在每次运行时通过 WebFetch 从 GitHub `main` 分支实时拉取最新的编码规范，而不是用本地固定版本。好处是规范自动保持最新；坏处是 main 分支一旦被篡改，所有 agent 的行为都会受影响。
