# kangraemin/claude-inspector — 学习案例

**仓库**：https://github.com/kangraemin/claude-inspector
**Stars**：115 | **来源**：upstream audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-08-01（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `template-design`, `examples-driven`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-inspector 是一个**面向 macOS/Electron 桌面应用的 AI 辅助调试工具**，以韩文（한국어）为主要开发语言（README.ko.md 为主文档）。核心功能：用 3 个 Claude Code agent 对 Electron 应用进行代理分析、UI 调试和代码审查。打包为 Homebrew 安装的桌面应用，支持 Playwright e2e 测试。115 ★。

关键事实：
1. 韩文开发团队，CLAUDE.md 和 agent 正文以韩文书写
2. 基于 Electron + Playwright 技术栈，非传统服务端应用
3. 3 个 agent + 3 个 skill（build/deploy/e2e），架构极简
4. 有 Sentry 错误收集（analytics.js）但未向用户披露
5. skills 平均分 100/100，agents 平均分 47.7/100——两层质量极端分化

### 1.2 架构剖析
- **目录结构**：
  ```
  /
  ├── .claude/
  │   ├── agents/               # 3 个 agents（proxy-analyzer, reviewer, ui-debugger）
  │   └── skills/
  │       ├── e2e/SKILL.md      # 100分
  │       ├── build/SKILL.md    # 100分
  │       └── deploy/SKILL.md   # 100分
  ├── scripts/notarize.js       # 代码签名脚本
  ├── main.js                   # Electron 主进程
  ├── public/index.html         # 渲染进程
  └── package.json              # 含 Sentry + Playwright 依赖
  ```
- **文件类型分布**：3 个 agent、3 个 skill、0 个 command、1 个 shell-equivalent script（notarize.js）
- **编排关系**：agents 独立工作，无 router；skills 被 agents 隐式使用（build/deploy/e2e 可被 reviewer 调用）
- **跨件契约**：reviewer agent 引用 `~/.claude/rules/review-rules.md`（用户级文件，不在仓库中）；ui-debugger 和 proxy-analyzer 引用 `public/index.html`（仓库内文件，一致）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「工具层 100 分，角色层 43 分」——作者精心设计了 build/deploy/e2e 三个 skill，却几乎没有投入在 agent 质量上
- **解决什么问题**：Electron 桌面应用的调试周期长（打包→测试→分析），用 AI agent 加速这一循环
- **做了什么 trade-off**：选择 Electron（成熟 desktop UI 框架）而非 web 应用；选择 Playwright 做 e2e（自动化测试能力强）
- **反映什么认知模型**：作者对"工具如何使用"（skill 层）理解深刻，对"角色如何描述自己"（agent 层）理解较浅——skill 的质量告诉我们作者知道正确的写法，但 agent 的质量差异说明这些知识没有迁移过去

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「工具精密 + 角色粗糙」的不均衡双层架构**

skills 达到 100/100，agents 平均 47.7/100。这是"作者知识在不同类型文件间未均匀分布"的典型案例。模式名：**「skill 精 / agent 粗」不均衡型**。

模式特征清单：
- 特征 1：3 个 skill 全部 100 分，覆盖 build/deploy/e2e 三条黄金路径
- 特征 2：3 个 agent 全部缺失 `name` [frontmatter](#frontmatter) 字段，无法注册
- 特征 3：reviewer agent 引用了仓库外的 `review-rules.md`（孤儿引用）
- 特征 4：agent 无示例块、无输出格式段落
- 特征 5：质量差距反映"工具知识 ≠ 角色定义知识"

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 桌面应用 QA/调试辅助 | ✅ 适用 | e2e skill 精度高，Playwright 集成完善 |
| 需要 agent 复杂编排 | ❌ 不适用 | agents 无法注册（缺 name），编排失效 |
| 纯 skill 工具库 | ✅ 适用 | skill 层质量优秀，可单独提取复用 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 不均衡双层（本仓库） | claude-inspector | skill 质量高，值得参考 | agents 质量低导致整体评分拖累 |
| 均衡全层（agents+skills 同等质量）| c0x12c/ai-toolkit | 整体评分高、可用性强 | 维护成本高 |
| 纯 skill 架构 | MarkQWu/gstack | 简单，无 agent 注册风险 | 缺乏复杂推理能力 |

### 2.4 改进空间
1. **当前问题**：3 个 agents 缺 `name` 字段，无法注册。**改进做法**：为每个 agent 的 [frontmatter](#frontmatter) 加 `name:` 字段（proxy-analyzer、reviewer、ui-debugger）。**预期收益**：agents 立即可用，5 分钟修复，0 风险。
2. **当前问题**：reviewer 依赖用户目录的 `review-rules.md`，新用户 clone 后失效。**改进做法**：将 review-rules.md 提交到仓库（`.claude/rules/review-rules.md`）并改为相对路径引用。**预期收益**：消除孤儿引用，让项目开箱即用。
3. **当前问题**：agents 无示例块。**改进做法**：为每个 agent 加至少一个 `<example>` 块，展示典型输入和期望输出格式。**预期收益**：agent 行为更可预测。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **76/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude/agents/proxy-analyzer.md | 43 | 缺 `name` + 无示例 + 无输出格式 |
| .claude/agents/ui-debugger.md | 45 | 缺 `name` + 无示例 + 无输出格式 |
| .claude/agents/reviewer.md | 55 | 缺 `name` + 无示例 |
| CLAUDE.md | 88 | 内容清晰，略简洁 |
| .claude/skills/e2e/SKILL.md | 100 | 无问题 |
| .claude/skills/build/SKILL.md | 100 | 无问题 |
| .claude/skills/deploy/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **skill 定义精度（100 分）** → 3 个 skill 都清楚描述了 npm script 的调用方式和 package.json 的对应关系。根本原因：作者把 skill 理解为"可执行的操作说明书"而非"模糊描述"。借鉴：我的每个 skill 应明确说明它调用哪些工具、期望哪些文件存在。
2. **package.json 与 skill 对齐** → build/deploy/e2e skill 引用的 npm scripts 全部在 package.json 中存在。根本原因：skill 与代码同步维护。借鉴：当 skill 引用脚本时，PR 检查阶段应同步验证脚本存在。
3. **CLAUDE.md 简洁有效** → 内容少但精准，88 分仅因为略简。借鉴：CLAUDE.md 不需要长，关键是内容不能模糊。

### 3.3 当时的缺陷
1. **3 个 agents 缺 `name` frontmatter 字段（注册破坏性 bug）** → agent 在 Claude Code 中无法通过名称调用。为什么会失败：`name` 是 agent 注册的 primary key，缺少后整个 agent 功能失效。自查：我的所有 agent 是否都有 `name:` 字段？
2. **reviewer 引用 `~/.claude/rules/review-rules.md`（孤儿引用）** → 该文件不在仓库中，新 clone 用户的 reviewer agent 在步骤 1 就会失败。为什么会失败：用户级文件路径对其他人来说不存在，且无 fallback。自查：我的 agents/skills 有没有引用不在仓库内的文件？
3. **agents 无 model 声明** → 所有 agent 默认用 Sonnet，但 proxy-analyzer（网络流量分析）和 ui-debugger（UI 视觉分析）需要更强的上下文理解能力，可能更适合 Opus。为什么设计会失败：成本和质量都不可预测。自查：我的 agents 是否根据复杂度选择了合适的 model tier？

### 3.4 当时的优化机会
1. 为 3 个 agents 加 `name:` 字段（5 分钟，最高优先级）
2. 将 review-rules.md 提交到仓库，修改路径为相对路径
3. 为 agents 加示例块和输出格式段落

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| proxy-analyzer 缺 `name` 字段 | `grep "^name:" .claude/agents/proxy-analyzer.md` | **持续**（0 个 name 字段） | 约 3.5 个月，核心 bug 未修 |
| reviewer 缺 `name` 字段 | `grep "^name:" .claude/agents/reviewer.md` | **持续** | 同上 |
| ui-debugger 缺 `name` 字段 | `grep "^name:" .claude/agents/ui-debugger.md` | **持续** | 同上 |
| reviewer 引用 `~/.claude/rules/review-rules.md` | `grep "review-rules" .claude/agents/reviewer.md` | **持续**（lines 4, 17, 25） | 用户体验问题未解决 |

### 4.2 架构演进
对比当前 HEAD 和 audit 时文件：结构几乎无变化。所有 agents 和 skills 仍存在，路径未变。这个仓库在过去 3.5 个月中**几乎没有针对 NL 质量问题的任何更新**。

这说明维护者要么不了解这些缺陷，要么将维护优先级放在 Electron 应用本身而非 Claude 集成。

### 4.3 新增的可学习模式
暂无。当前 HEAD 与 audit 快照几乎完全相同，无新增可学习模式。

---

## 五、校准

### 5.1 我已经在做对的
1. bureau/auditor agent 有 `name:` 字段 ✓
2. bureau 的 skills 有示例块（recall/SKILL.md 有 3 个示例）✓
3. 我的 agents 所引用的文件都在仓库内 ✓
4. bureau/auditor 有 `model: sonnet` 声明 ✓

### 5.2 挑战 / 验证
这个案例**验证了"skill 质量和 agent 质量是独立知识域"的假设**。claude-inspector 的作者清楚地知道如何写好 skill（100 分），却在 agent 层犯了最基础的错误（缺 name 字段）。这意味着：即使在某个组件类型上写得很好，也不能假设同样的知识会自动迁移到另一种组件类型。每种组件类型需要独立的质量检查。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我所有 agents 是否有 name 字段
find /tmp/my-repos/ -name "*.md" -path "*/agents/*" | while read f; do
  if ! grep -q "^name:" "$f" 2>/dev/null; then
    echo "MISSING name: $f"
  fi
done
```
命中后怎么办：在 frontmatter 最顶部加 `name: <agent-name>` 行，立即可用。

```bash
# 检查我的 agents/skills 是否引用了用户目录文件
grep -rn "~/.claude\|~/\." /tmp/my-repos/MarkQWu-bureau/.claude/agents/*.md 2>/dev/null
grep -rn "~/.claude\|~/\." /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md 2>/dev/null
```
命中后怎么办：将引用的文件提交到仓库，改为相对路径引用；或加 fallback 指引告诉用户如何自建该文件。

### 6.2 灵感 → 实施路径
1. **想法**：建立 agent 质量 checklist，防止 skill/agent 质量脱节
   - **为何可行**：claude-inspector 证明即使有高质量 skills，也可能忽视 agents
   - **第一步**：在项目的 CONTRIBUTING.md 或 CLAUDE.md 加一段"新增 agent 必须包含：name, model, 至少 1 个 example"

2. **想法**：将外部规则文件（如 review-rules.md）提交到仓库
   - **为何可行**：仓库内引用才能保证新贡献者开箱即用
   - **第一步**：检查 bureau 的 agents 和 skills 是否有任何 `~/.claude/` 路径引用，发现后提交对应文件

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 kangraemin/claude-inspector 的核心目的**：Electron 桌面应用的 AI 辅助调试工具

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 低 | 都是 Claude Code 插件 | bureau 管理知识库，inspector 调试桌面应用 | 低（仅 agent 质量教训有参考价值） |
| MarkQWu/gstack | 低 | 都有 skills | gstack 是 skill 工具集，inspector 面向桌面 app 调试 | 低 |

若全部「低」，我的仓库中无目的相近的项目，本节仅做技术模式对照。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查命令 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| agent 缺 `name` 字段 | `grep -L "^name:" */agents/*.md` | 未命中（bureau/auditor 有 name）| — |
| 引用仓库外文件 | `grep -rn "~/.claude" */skills/*/SKILL.md` | 未命中 | — |
| agents 缺示例块 | `grep -rL "<example\|## Example" */agents/*.md` | 未验证，需手动检查 | 中 |

**命中后的具体行动建议**：
- 若 bureau/auditor.md 缺示例块 → 加一个"输入：某个 canon 页面路径 → 输出：包含 trust tier 和 contradiction 列表的报告"示例 → 15 分钟可完成

### 7.3 别人的更优方案

1. **领域**：skill 与 package.json 完全对齐
   - **本案例做法**：build/deploy/e2e skill 中引用的 npm script 名称与 package.json 完全一致（可验证）
   - **我的项目现状**：gstack/SKILL.md 引用的工具/命令未经系统验证
   - **如何借鉴**：建立一个 checklist：每次 skill 引用外部工具/脚本时，PR 时验证该工具/脚本存在

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：agent model 声明和注册完整性
- **我的做法**：bureau/.claude/agents/auditor.md 有完整的 name、model、tools、description
- **本案例做法（弱在哪）**：3 个 agents 全部缺 name，且无 model 声明
- **意义**：bureau 的 agent 定义更完整，这是一个亮点。如果未来给上游做 PR 参考，可以把这种完整 frontmatter 作为示例

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据。Claude Code 读 agent 文件时先解析 frontmatter 来注册这个 agent——`name`（名称，必填）、`model`（使用哪个 AI 模型）、`tools`（允许使用的工具）都在这里声明。缺了 `name` 字段，Claude Code 就找不到这个 agent，就好像这个文件不存在一样。

### <a name="孤儿引用"></a>孤儿引用
> 文件 A 引用了文件 B，但文件 B 不存在（或在别人的机器上不存在）。这种引用会在运行时静默失败——Claude 读不到 B 的内容，但也不会报错，只是行为变得不可预测。常见于引用用户主目录（`~/.claude/...`）下的文件。

### <a name="e2e测试"></a>e2e 测试（End-to-End Test）
> 模拟真实用户操作、覆盖完整流程的测试。比如：打开浏览器 → 输入内容 → 点击按钮 → 验证结果。`Playwright` 是目前流行的 e2e 测试框架，可以控制 Chromium/Firefox/WebKit。
