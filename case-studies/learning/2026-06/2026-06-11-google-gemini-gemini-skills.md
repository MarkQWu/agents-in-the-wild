# google-gemini/gemini-skills — 学习案例

**仓库**：https://github.com/google-gemini/gemini-skills
**Stars**：3,330 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-06-11（基于当前 HEAD）
**主题标签**：`examples-driven`, `vague-quantifier`, `model-pinning`, `template-design`, `single-purpose`

**xiaolai 案例**：[../../case-studies/2026-04-28-google-gemini-gemini-skills-learnings.md](../../case-studies/2026-04-28-google-gemini-gemini-skills-learnings.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`gemini-skills` 是 Google Gemini 团队官方出品的 **Claude Code skill 集合**，专为帮助开发者用 Gemini API / Vertex AI 构建应用而设计。每个 skill 是一份精炼的「API 使用指南 + 最佳实践 + 多语言代码示例」，直接注入 Claude Code 的工作上下文。

关键事实：
1. 官方背书，由 Google 员工 `markmcd` 等维护，质量标准极高
2. 当前 3 个 SKILL.md（gemini-api-dev、gemini-interactions-api、gemini-live-api-dev）；审计时 4 个（原有 vertex-ai-api-dev 已被移除）
3. 审计得分 98/100，是迄今 NLPM 审计中最高分之一
4. 最高分 skill（gemini-live-api-dev）得分 100，是正面教学的标杆工件

### 1.2 架构剖析

```
skills/
├── gemini-api-dev/
│   ├── SKILL.md          # Gemini REST API 开发指南
│   └── references/       # 扩展参考（embedding、function-calling 等）
├── gemini-interactions-api/
│   ├── SKILL.md          # 思维签名、上下文缓存等高级交互
│   └── references/       # 迁移指南、模型对比等
└── gemini-live-api-dev/
    └── SKILL.md          # 实时双向流式 API（满分工件）
README.md
```

- **文件类型分布**：3 个 SKILL.md，~12 个 reference 文档
- **编排关系**：3 个 skill 相互独立，无调用关系。用户按需加载对应 skill；gemini-live-api-dev 内部有完整的降级路径（Live API → 标准 generate → 文本模拟）
- **跨件契约**：3 个 skill 共享「Documentation Lookup」部分的 MCP 工具调用约定（`search_documentation`），通过此约定保持一致性

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「每个 skill 是一个独立的、完整的 API 知识库」——无需依赖其他 skill，无需外部文件，即插即用
- **解决什么问题**：Claude Code 的 AI 训练数据落后于 Gemini API 快速迭代（新模型名、新 SDK 方法），skill 用显式约束覆盖训练数据
- **Trade-off**：每个 skill 覆盖面窄（单一 API 领域），但深度极高；breadth 牺牲换 depth
- **认知模型**：把 skill 看作「权威覆盖层」——`> [!IMPORTANT] These rules override your training data.` 这句话直接写在每个 skill 最顶部

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「权威覆盖型单职责 skill」**

关键特征：
- 每个 skill 开头用 `> [!IMPORTANT]` 声明「这些规则覆盖训练数据」，明确优先级
- skill 体系结构化：快速参考表 → 何时使用哪个功能 → 代码示例 → 已弃用/禁用模式 → 降级路径
- [frontmatter](#frontmatter) 的 description 精确描述适用场景（when to use this skill）
- 无 agent、无 command、无 hook——极简架构，skill 是唯一工件

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| API 官方知识注入（覆盖训练数据） | ✅ 极度适用 | 设计目的就是解决这个问题 |
| 单一 SDK/工具的最佳实践 | ✅ 适用 | 每个 skill 是完整的最佳实践文档 |
| 需要 agent 协作的复杂工作流 | ❌ 不适用 | 没有编排层，只有知识层 |
| 需要跨多个 skill 协调的任务 | ❌ 不适用 | skill 之间完全独立，无协作协议 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 权威覆盖型单职责 | gemini-skills | 质量极高，即插即用，无依赖 | 覆盖面窄，无编排 |
| 厚 skill + 薄编排 | Vibe-Skills | 覆盖广（258 个） | 结构不规范，质量参差 |
| NL 触发 + 多厂商适配 | oh-my-agent | 功能全面，跨平台 | 安全风险，依赖外部文件 |

### 2.4 改进空间

1. **当前问题**：vertex-ai-api-dev 已从仓库移除，但 Vertex AI 用户仍需要特定指导。**改进做法**：为 Vertex AI 更新的 Live API 模型（gemini-3.1-flash-live-preview）新建一个 `vertex-ai-live/SKILL.md`。**预期收益**：覆盖已删除文件的用户需求。
2. **当前问题**：README 中「thought circulation」链接文字仍存在（Bug3 未修复）。**改进做法**：把链接文字改为「thought signatures」（1 行改动）。**预期收益**：文档准确性，避免读者困惑。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）

得分 **98/100**（4 SKILL.md 加权均值 96/100）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| gemini-live-api-dev/SKILL.md | 100 | 无问题，标杆工件 |
| gemini-interactions-api/SKILL.md | 98 | 编号重复 bug（第 283-284 行） |
| gemini-api-dev/SKILL.md | 97 | `fetch_url` 不是 Claude 原生工具名 |
| vertex-ai-api-dev/SKILL.md | 96 | MCP 工具名漂移（`search_documents` vs `search_documentation`） |

### 3.2 当时值得借鉴的模式

1. **gemini-live-api-dev 的完整降级路径** → Live API 不可用时，skill 明确写出降级步骤（WebSocket → HTTP → 文本模拟），任何情况都有可执行路径。路径：`skills/gemini-live-api-dev/SKILL.md`。如何借鉴：每个 skill 都加 `## Fallback` 节，列出功能不可用时的替代方案。
2. **「这些规则覆盖训练数据」声明** → 放在 skill 最顶部，明确 AI 优先级。路径：所有 3 个 SKILL.md。如何借鉴：当 skill 包含「特定版本」知识时，用 `> [!IMPORTANT]` 声明覆盖优先级。
3. **弃用/禁用模式的显式列表** → gemini-live-api-dev 有 `## Deprecated & Forbidden Patterns` 节，列出不要做什么和为什么。路径：`skills/gemini-live-api-dev/SKILL.md`。如何借鉴：为每个 skill 加「常见错误」或「禁止模式」节。

### 3.3 当时的缺陷

1. **问题**：vertex-ai-api-dev 的 reference 文件与 gemini-live-api-dev 的 SKILL.md 在 3 个地方矛盾（deprecated 模型、`send_client_content` 用法、`media` vs `audio`/`video` key）。**根本原因**：vertex-ai-api-dev 的 references 文件没有跟着新的 Live API 模型迁移更新，形成「内部文档不同步」。**自查**：我的项目是否有「参考文档比主文档旧」的问题？→ 需检查 claude-for-legal 的 references/ 目录。
2. **问题**：gemini-api-dev 使用 `fetch_url` 这个不存在的 Claude 工具名。**根本原因**：作者用了一个「看起来合理」的工具名，但 Claude Code 的实际工具名是 `WebFetch`。**自查**：我的 skill 里有没有引用不存在的工具名？→ 用 grep 检查。
3. **问题**：README 链接文字「thought circulation」与链接目标锚点 `#signatures` 不匹配。**根本原因**：术语在 SDK 迭代中改名（从「thought circulation」到「thought signatures」），README 未同步更新。**自查**：我的 README 链接有没有文字与目标不匹配的？

### 3.4 当时的优化机会

1. 统一 4 个 skill 的 MCP 工具名（`search_documents` vs `search_documentation`）
2. 修复 gemini-api-dev 的 `fetch_url` → `WebFetch`（或 agent-neutral phrasing）
3. README 链接文字「thought circulation」→「thought signatures」

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| model_id 未定义变量（vertex-ai-api-dev reference） | `grep -n "model_id"` in vertex-ai-api-dev | **无法验证** — vertex-ai-api-dev 目录已从仓库移除 | 文件被删，bug 自然消失，但可能用户需求仍存在 |
| `fetch_url` 错误工具名（gemini-api-dev） | `grep -n "fetch_url"` in gemini-api-dev/SKILL.md | **已修复** ✓ — 未找到 `fetch_url`，已改为 agent-neutral phrasing | PR #37 合并生效 |
| README「thought circulation」→「thought signatures」| `grep -n "circulation"` in README.md | **仍存在** — 第 13-14 行仍为「thought circulation」链接文字 | PR #38 虽合并，但 README 之后似乎被回滚或有新的变更 |

**注**：关于 PR #38 合并后 README Bug3 仍存在的情况：当前 HEAD 的 README.md 第 13-14 行仍显示「thought circulation」。可能是后续 commit 引入了新版本 README，覆盖了修复。

### 4.2 架构演进

最显著的变化：**vertex-ai-api-dev 整个目录被移除**。这说明 Google Gemini 团队决定不再在这个仓库维护 Vertex AI 特定的 skill，或者把 vertex-ai-api-dev 的内容合并到了其他渠道。结合 xiaolai 案例中描述的 PR #36 被维护者以「No longer needed」关闭，可以推断这次删除是有意为之——当 Vertex AI Live API 的新模型文档发布后，旧的 references 已经过期，直接删除比修复成本更低。

### 4.3 新增的可学习模式

- vertex-ai-api-dev 被删除后，gemini-interactions-api 新增了 `references/migration.md`，这是一种「主动文档化迁移路径」的做法——不只说「用新的」，还提供「从哪里来到哪里去」的指导。

---

## 五、校准

### 5.1 我已经在做对的

1. **skill frontmatter 完整**：echo-sleuth 的 SKILL.md 都有 name + description + version，与 gemini-skills 一致。
2. **专注单一职责**：echo-sleuth 的每个 skill 专注一个领域（memory-management 不涉及 git-mining 的内容），与 gemini-skills 的设计哲学一致。
3. **description 描述「何时使用」**：echo-sleuth/skills/git-mining/SKILL.md 的 description 写明了「when the user asks to check git history...」触发条件，与 gemini-skills 的 description 写法一致。

### 5.2 挑战 / 验证

xiaolai 的案例（2026-04-28）记录了 Google 维护者 markmcd 的直接反馈，挑战了我对「贡献者和上游仓库关系」的假设。核心教训：**「无法自己验证的发现不应该提交给维护者」**。这不只是礼节问题，而是对认知负担（cognitive load）的直接尊重。在我自己给别人提 bug 或建议时，需要先自己复现，确认。这个教训对我的日常代码审查行为同样适用。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 是否引用了不存在的工具名
grep -rn "fetch_url\|search_documentation\|search_documents" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ \
  /tmp/my-repos/MarkQWu-claude-for-legal/ 2>/dev/null
# 命中后：替换为 Claude Code 实际工具名（WebFetch、Bash 等）
```

```bash
# 检查我的 README 链接是否有文字与锚点不匹配的
grep -n "\[.\+\](#" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/README.md 2>/dev/null | head -20
# 手工核对每个链接的文字是否与锚点标题对应
```

```bash
# 检查 references/ 目录文件是否比主 SKILL.md 旧
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/ -name "*.md" -newer \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/memory-management/SKILL.md 2>/dev/null
# 命中文件多于预期时：检查主 SKILL.md 是否需要同步更新
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 SKILL.md 添加 `## Forbidden Patterns` 节
   - **为何可行**：gemini-live-api-dev 证明这个模式能预防 AI 犯已知错误
   - **第一步**：在 skills/memory-management/SKILL.md 末尾加「禁止在 body 中使用嵌套 YAML」「禁止写超过 200 行」两条禁止规则，10 分钟

2. **想法**：为 claude-for-legal 的 skill 添加「知识截止日期」声明
   - **为何可行**：法律法规有时效性，gemini-skills 的「覆盖训练数据」模式用于 API 版本，法律 skill 应用于「监管变化」
   - **第一步**：在 reg-feed-watcher/SKILL.md 顶部加 `> [!NOTE] This skill reflects regulations as of 2026-06. Check for updates before use.`，5 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：用 skill 注入 Gemini API 特定知识，覆盖 AI 训练数据中的过时信息
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 都用 SKILL.md 格式；都是 Claude Code 插件 | gemini-skills 是 API 知识库；echo-sleuth 是 JSONL 数据挖掘工具 | 中（借鉴文件结构） |
| MarkQWu/claude-for-legal | 低 | 都是特定领域的知识注入 | 领域不同（AI API vs 法律），但模式相似 | 高（模式直接可用） |
| MarkQWu/drama-workshop-skills | 低 | 无明显相同点 | 完全不同目的 | 低 |

若全部相似度「低」，则按「技术模式对照」处理：我的项目中「知识注入 + 覆盖训练数据」的模式可以向 gemini-skills 看齐。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Reference 文件与主 SKILL.md 不同步 | `find skills/ -name "*.md" -newer skills/*/SKILL.md` | echo-sleuth: 无 references 子目录，问题不适用 | N/A |
| 工具名称使用了不存在的名字 | `grep -rn "fetch_url\|search_docs"` | echo-sleuth: 命中 0（commands 用 Bash/Read/Edit 等标准名） | N/A |
| README 链接文字与锚点不匹配 | `grep -c "\[.\+\](#"` | 需手工检查，未 grep | 低 |

### 8.3 别人的更优方案

1. **领域**：弃用模式的显式声明（Deprecated & Forbidden Patterns）
   - **本案例做法**：gemini-live-api-dev 用专门的 section 列出「不要这样做 + 为什么 + 正确做法」（路径：`skills/gemini-live-api-dev/SKILL.md`）
   - **我的项目现状**：echo-sleuth 的 skills 只说「应该这样做」，没有「不要这样做」的负向约束
   - **如何借鉴**：在 memory-management/SKILL.md 末尾加 `## Forbidden Patterns`，列 3 条禁止事项

2. **领域**：单文件完整性（skill 不依赖任何外部文件）
   - **本案例做法**：3 个 SKILL.md 都可以独立使用，reference 文件是可选的深化材料（路径：任意 SKILL.md）
   - **我的项目现状**：echo-sleuth/skills/git-mining/SKILL.md 引用了 bash 脚本（通过 Bash tool 调用 scripts/）
   - **如何借鉴**：在 git-mining/SKILL.md 中内联关键命令示例，让 skill 可以在 scripts/ 不存在时也能退而求其次地工作

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：多语言代码示例
- **我的做法**：echo-sleuth/skills/jsonl-core/SKILL.md 使用单一语言（bash + python），风格一致
- **本案例做法**（弱在哪）：gemini-skills 提供 Python + JavaScript/TypeScript 双语言示例，在维护时需要保持两套代码同步，存在漂移风险（vertex-ai-api-dev 就是因为维护难度高被删除的）
- **意义**：单语言 skill 维护成本低，减少「参考文件不同步」的系统性风险

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置。gemini-skills 的 frontmatter 的 description 字段非常典型：它不只描述 skill 做什么，而是明确说「在什么情况下使用这个 skill」（when to use），帮助 Claude Code 在多个 skill 可用时做出正确选择。

### <a name="覆盖训练数据"></a>覆盖训练数据
> AI 模型（比如 Claude）的知识来自训练数据，训练数据有截止日期。如果一个 API 在训练数据截止后更新了（比如新增模型名、改变方法签名），AI 就会用旧的错误方式调用它。skill 文件通过在顶部写 `> [!IMPORTANT] These rules override your training data.` 来告诉 AI「不管你学过什么，先看这个文件说的」，相当于给 AI 发了一份最新的操作说明书。

### <a name="降级路径"></a>降级路径
> 当首选方案不可用时的备用执行路径。gemini-live-api-dev 写明：如果 Live API 不可用，降级到标准 generate；再不可用，降级到文本模拟。这样 AI 在任何环境下都能完成任务，只是方式不同。类比：飞机被延误时，你有备用路线：改乘高铁，再不行就汽车，最差打出租车。
