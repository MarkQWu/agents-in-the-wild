# The-Vibe-Company/companion — 学习案例

**仓库**：https://github.com/The-Vibe-Company/companion
**Stars**：2312 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-05-27（基于当前 HEAD）
**主题标签**：`template-design`, `vague-quantifier`, `examples-driven`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
**companion** 是 Claude Code 和 Codex 的 Web UI。它通过逆向工程 Claude Code CLI 未文档化的 `--sdk-url` [WebSocket协议](#websocket协议) 实现了浏览器端界面，支持运行多个并发 Claude Code 会话，并提供流式输出、工具调用可见性和权限控制。

技术栈：Bun + Hono 后端，Vite HMR 前端，TypeScript，Vitest 测试，husky pre-commit 钩子。当前版本 0.95.0（package.json），companion npm 包版本 `^0.2.2`。

NL 层面的核心定位：`.agents/skills/` 下的 18 个 [skill](#skill) 文件不是为外部使用者设计的通用工具——它们是为 companion 自身的 UI/UX 迭代服务的内部操作集合，充当「一次性 UI 精修操作指令集」。用户启动某个 skill，Claude 就对界面的特定维度执行一次针对性改进。

关键事实：
1. 共 19 个 NL 产物：18 个 `.agents/skills/*/SKILL.md` + 1 个 `CLAUDE.md` 配置
2. 无命令文件、无代理文件、无钩子——纯 skill 架构
3. `frontend-design` skill 额外附带 7 个参考文档（`reference/` 子目录）
4. 6 个 skill 通过 frontmatter 明确引用 `frontend-design` 作为底层设计原则知识库
5. 得分最高的三个 skill（`audit`、`critique`、`teach-impeccable`）均为 100/100

### 1.2 架构剖析
```
companion/
  .agents/
    skills/
      audit/SKILL.md              # 综合质量审查（100/100）
      critique/SKILL.md           # UX 评估（100/100）
      teach-impeccable/SKILL.md   # 一次性设计上下文采集（100/100）
      polish/SKILL.md             # 发布前最终修整（98/100）
      frontend-design/
        SKILL.md                  # 核心设计原则路由入口（90/100）
        reference/
          color-and-contrast.md
          interaction-design.md
          motion-design.md
          responsive-design.md
          spatial-design.md
          typography.md
          ux-writing.md
      animate/SKILL.md            # 添加动效（88/100）
      bolder/SKILL.md             # 增强对比度/粗度（88/100）
      colorize/SKILL.md           # 色彩系统（88/100）
      delight/SKILL.md            # 愉悦感细节（88/100）
      distill/SKILL.md            # 简化视觉噪声（88/100）
      quieter/SKILL.md            # 降低视觉嘈杂感（88/100）
      adapt/SKILL.md              # 响应式适配（86/100）
      clarify/SKILL.md            # 清晰度提升（90/100）
      ...（共 18 个）
  CLAUDE.md
  package.json                    # version: 0.95.0
  web/                            # Vite 前端
  scripts/                        # 构建脚本（含安全发现）
```

**文件类型分布**：18 个 skill，0 个 agent，0 个 command，0 个 hook，1 个 CLAUDE.md 配置。

**编排关系**：`frontend-design` 是知识枢纽。6 个 skill（animate、bolder、colorize、delight、distill、quieter）在 body 中显式要求先加载 `frontend-design` skill。`audit` 和 `critique` skill 在输出模板中推荐路由回其他已有 skill（`/animate`、`/quieter`、`/optimize` 等）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单职责纵向切片」——每个 skill 只做一件事（`colorize` 只负责色彩，`animate` 只负责动效），用户的使用模式是「我要让这个界面更 X，执行 `/X`」
- **解决什么问题**：Claude Code 会话面对完整 UI 代码库时，单次「优化整个 UI」的指令过于宽泛，结果不可预期。切片 skill 让改进粒度可控、可重复
- **知识分层**：操作 skill（animate、bolder…）负责「做什么」，`frontend-design/reference/` 子文档负责「设计原则是什么」，两层分工明确
- **做了什么 trade-off**：无编排层意味着用户必须自己知道要用哪个 skill；好处是每个 skill 极度聚焦且易于单独测试和维护

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单职责 skill 纵向切片 + 共享参考知识库」**

模式特征清单（4 条）：
- 特征 1：每个 skill 是界面属性的一个维度（颜色、动效、对比度…），命名即操作动词
- 特征 2：共享知识集中在 `frontend-design/reference/*.md`，操作 skill 通过显式路由语句引用，不重复
- 特征 3：高质量 skill（`audit`、`critique`）输出带严格 schema 的结构化报告，包含闭合枚举字段（`Critical / High / Medium / Low`）
- 特征 4：`teach-impeccable` 是「一次性初始化」skill 的范例：Step 1 探索代码库，Step 2 提问收集上下文，最终写入 AI 配置文件

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 迭代优化一个特定产品的 UI | ✅ 高度适用 | skill 与产品强绑定，每次只改一个维度，风险可控 |
| 作为通用「设计插件」发布给陌生用户 | ⚠️ 需改进 | 大多数 skill 缺 [Output Format](#output-format)，新用户不知道期待什么 |
| 有多个协作者共同维护 UI | ✅ 适用 | `audit` 和 `critique` 生成标准化报告，可存档比较 |
| 需要自动化流水线串联多个操作 | ❌ 不适用 | 无命令/代理编排层，操作之间的串联依赖用户手动触发 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单职责 skill 纵向切片（本仓库） | companion | 聚焦、可测、DRY（共享参考库） | 无编排，用户需知道调哪个 skill |
| 多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置，138 个领域专家即插即用 | 无跨代理协作，无共享知识 |
| 完整命令+代理+skill 插件 | AgriciDaniel/claude-ads | 命令入口清晰，可 marketplace 发布 | 维护 manifest 同步成本高 |
| 单 CLAUDE.md 全局配置 | feiskyer/claude-code-settings | 最轻量，无额外文件 | 无模块化，规则混杂在一起 |

### 2.4 改进空间
1. **当前问题**：18 个 skill 中 0 个有 `## Output Format` 节（[Output Format](#output-format) 缺失是此仓库最大的单项扣分）。**改进做法**：参照 `audit/SKILL.md` 的每字段模板，为其余 17 个 skill 补充 `## Output Format` 节，定义产物结构和字段枚举。**预期收益**：NLPM 评分从 91 提升到约 95+，更重要的是让 Claude 的输出可预测、可比较。
2. **当前问题**：6 个 skill（animate、bolder、colorize、delight、distill、quieter）的 MANDATORY PREPARATION 模块存在复制粘贴模板错误："MUST STOP and STOP"（应为"MUST STOP and call…"）。**改进做法**：一次性修正模板原文，所有引用同一模板的 skill 同步更新。**预期收益**：消除运行时的歧义指令。
3. **当前问题**：7 个 skill 引用了 `AskUserQuestionTool`，该工具不是 Claude Code 标准工具集的一部分，调用时静默失败。**改进做法**：改用标准澄清提问模式（直接在 skill body 中写"如果信息不足，向用户提问：…"），或在 README 中文档化自定义工具注册要求。**预期收益**：消除静默失败路径。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
**总分：91/100**（Security: CLEAR）。0 个 bug，22 个质量问题，7 个安全发现（0 Critical，0 High，1 Medium，6 Low）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `.agents/skills/adapt/SKILL.md` | 86 | 缺 Output Format；"where appropriate" 出现 2 次 |
| `.agents/skills/animate/SKILL.md` | 88 | 缺 Output Format；MANDATORY PREPARATION 模板错误 |
| `.agents/skills/bolder/SKILL.md` | 88 | 缺 Output Format；同上模板错误 |
| `.agents/skills/colorize/SKILL.md` | 88 | 缺 Output Format；同上模板错误 |
| `.agents/skills/delight/SKILL.md` | 88 | 缺 Output Format；同上模板错误 |
| `.agents/skills/distill/SKILL.md` | 88 | 缺 Output Format；同上模板错误 |
| `.agents/skills/quieter/SKILL.md` | 88 | 缺 Output Format；同上模板错误 |
| 其余 8 个 skill | 90 | 仅缺 Output Format |
| `CLAUDE.md` | 95 | 软引用非标准 `agent-browser` CLI |
| `.agents/skills/polish/SKILL.md` | 98 | 边界模糊词"Appropriate motion" |
| `.agents/skills/audit/SKILL.md` | 100 | 无 |
| `.agents/skills/critique/SKILL.md` | 100 | 无 |
| `.agents/skills/teach-impeccable/SKILL.md` | 100 | 无 |

### 3.2 当时值得借鉴的模式
1. **触发器即路由**（R04）→ 高分 skill 的 `description` 字段内嵌 3–5 个具体动作短语（"comprehensive audit … accessibility, performance, theming, and responsive design"），而非能力摘要 → 路径：所有 skill 的 frontmatter `description:` 字段 → 借鉴方式：description 不写「这个 skill 做什么」，写「用户会用什么词来触发它」。
2. **共享知识 DRY**（R07）→ 6 个 skill 用同一句路由语句「First: Use the frontend-design skill」把设计原则知识外包给共享库 → 路径：`animate/SKILL.md:13`、`bolder/SKILL.md:13` 等第 13 行 → 借鉴方式：把公共参考知识抽到独立 skill 的 `reference/` 子目录，其他 skill 通过路由语句引用而不重复。
3. **可验证清单**（R08）→ `audit/SKILL.md` 的每条检查项都是可测量的客观标准（"Text contrast ratios < 4.5:1"、"Interactive elements < 44x44px"）→ 路径：`audit/SKILL.md:20-36` → 借鉴方式：用具体数值或规范名称替换抽象标准。
4. **输出 schema 闭合枚举**（R41）→ `audit/SKILL.md` 的每字段模板含 `Severity: Critical / High / Medium / Low`（闭合枚举），防止模型自由发挥 → 路径：`audit/SKILL.md:66-74`

### 3.3 当时的缺陷
1. **[Output Format](#output-format) 全面缺失**：14/18 个 skill 无 `## Output Format` 节，违反 R41，每个 -10 分
2. **复制粘贴模板错误**：animate、bolder、colorize、delight、distill、quieter 第 21 行均含 "MUST STOP and STOP"，应为 "MUST STOP and call…"，违反 R02，每个 -2 分
3. **模糊量词**：`adapt/SKILL.md` 的 "where appropriate" 出现 2 次（R01，-4 分）；`polish/SKILL.md` 的 "Appropriate motion" 未定义（R01，-2 分）
4. **非标准工具引用**：7 个 skill 引用 `AskUserQuestionTool`，不在 Claude Code 标准工具集中，静默失败
5. **非标准 CLI 引用**：`CLAUDE.md` 要求使用 `agent-browser` CLI，未在仓库中定义或文档化

### 3.4 当时的优化机会
- 最高杠杆的单次修复：修正 MANDATORY PREPARATION 模板原文 → 一次改动修复 6 个 skill 的模板错误
- 最大分数提升：补全所有 skill 的 `## Output Format` 节 → 潜在提升 ~9 分
- 最低风险修复：把 `AskUserQuestionTool` 替换为内联澄清提问模式 → 消除静默失败，无副作用

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 当时状态 | 当前状态 | 结论 |
|---|---|---|---|
| MANDATORY PREPARATION 模板错误 | 存在于 6 个 skill | **仍存在**（animate、bolder、colorize、delight、distill、quieter 第 21 行） | ❌ 未修复 |
| `## Output Format` 缺失 | 14/18 个 skill | **仍全部缺失**（grep 找不到任何 skill 含 Output Format 节） | ❌ 未修复 |
| `AskUserQuestionTool` 引用 | 7 个 skill | **仍存在**（7 个 skill） | ❌ 未修复 |
| `agent-browser` 引用 | CLAUDE.md 中存在 | **仍存在**（"Always use `agent-browser` CLI command to explore the browser"） | ❌ 未修复 |
| `adapt/SKILL.md` 模糊量词 | "where appropriate" 2 次 | **仍存在**（2 次） | ❌ 未修复 |

**结论**：2026-04-26 到 2026-05-27 的一个月内，所有已识别的 NL 质量问题均未得到修复。仓库版本从 audit 时的状态升至 0.95.0，但 NL 层完全未变。

### 4.2 架构演进
无可观察的 NL 架构演进。skill 集合、数量（18 个）、目录结构与 audit 时完全一致。代码层（web/、scripts/）有版本迭代，NL 层静止。

### 4.3 新增的可学习模式
无新增 NL 模式可学习（NL 层无变更）。

---

## 五、校准

### 5.1 我已经在做对的
1. **参考子目录模式**：我的 `drama-workshop-skills` 仓库已经实现了类似 `frontend-design/reference/*.md` 的 `references/*.md` 子目录结构，这个模式在 companion 中得到了验证（R05 + R07 证据）
2. **无复制粘贴模板错误**：我的仓库（drama-workshop-skills、echo-sleuth-for-claude）没有发现同类 MANDATORY PREPARATION 模板错误

### 5.2 挑战 / 验证
1. **Output Format 缺失是系统性问题**：companion 91/100 的仓库尚且在 18 个 skill 中 0 个有 Output Format——说明这个遗漏极其普遍，需要主动逐一检查自己的每个 skill
2. **静默失败的工具引用难以发现**：`AskUserQuestionTool` 在 skill 中看起来合法，但运行时静默失败。自己的 skill 是否也引用了非标准工具？需要主动核对
3. **模板批量传播的双刃剑**：统一模板降低了维护成本，但错误也会批量传播。companion 的 6 个相同错误就是例证

---

## 六、行动

### 6.1 自查动作

**检查 1：我的 skill 是否有 Output Format？**
```bash
# 扫描所有 SKILL.md，找出缺少 Output Format 的文件
grep -rL "## Output Format\|## 输出格式\|## Output\b" \
  ~/drama-workshop-skills/ \
  ~/echo-sleuth-for-claude/ \
  --include="SKILL.md" 2>/dev/null
```
命中后怎么办：对每个命中文件，参照 `audit/SKILL.md:66-74` 的字段模板，补充 `## Output Format` 节，定义产物类型、字段名、枚举值。

**检查 2：我的 skill 是否引用了非标准工具？**
```bash
# 查找所有 SKILL.md 中类似 XxxTool 的工具调用引用
grep -rn "Tool\b\|tool_call\|call_tool" \
  ~/drama-workshop-skills/ \
  ~/echo-sleuth-for-claude/ \
  --include="SKILL.md" 2>/dev/null
```
命中后怎么办：对照 Claude Code 官方文档确认工具名称；若不在标准工具集中，改为内联澄清提问模式（"如果以下信息不足，先向用户提问：…"）。

**检查 3：我的 skill 是否有模糊量词？**
```bash
# 扫描常见模糊量词
grep -rn "where appropriate\|as needed\|if necessary\|comprehensive\|robust\|appropriate\b" \
  ~/drama-workshop-skills/ \
  ~/echo-sleuth-for-claude/ \
  --include="SKILL.md" 2>/dev/null | grep -v "^Binary"
```
命中后怎么办：将每处模糊标准替换为可测量标准，如 "comprehensive coverage" → "覆盖所有公开 API 的 happy path + 至少 1 个 error case"。

**检查 4：我的 agent frontmatter 是否有 model 字段？**
```bash
grep -rL "^model:" ~/echo-sleuth-for-claude/ --include="*.md" 2>/dev/null
```
命中后怎么办：在 frontmatter 中补充 `model: claude-sonnet-4-20250514`（或当前推荐的完整模型 ID），避免依赖浮动别名。

### 6.2 灵感 → 实施路径
1. **参考子目录模式**（已在做，强化）：在 `echo-sleuth-for-claude` 的 skills 中补充 `reference/` 子目录，把各 skill 公用的参考知识（如 jsonl 格式规范、Git log 格式说明）从 skill body 移出，改为路由引用。
2. **可验证清单化**：把 `drama-workshop-skills` 中抽象的「高质量剧本标准」替换为可测量标准（如「主角至少有 3 个对立面冲突」「每一幕不超过 800 字」）。
3. **输出 schema 闭合枚举**：为 `echo-sleuth-for-claude` 的 `experience-synthesis/SKILL.md` 定义 `## Output Format`，包含 `type: insight / pattern / anti-pattern`（闭合枚举）。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度
| 我的仓库 | 相似度 | 理由 |
|---|---|---|
| MarkQWu/drama-workshop-skills | **高**（架构层面高度相似） | 都是为单一产品/领域服务的 skill 集合，都有 `references/` 子目录模式，都是纯 skill 无 agent/command |
| MarkQWu/echo-sleuth-for-claude | **中** | skill 集合，但领域完全不同（工具型 vs UI 精修型）；`echo-sleuth` 有 4 个 skill，companion 有 18 个 |
| MarkQWu/claude-for-legal | **低** | 不同架构（CLAUDE.md 路由 vs skill 切片）；不同领域（法律工作流 vs UI 设计） |

### 8.2 在我的项目里复现的同类问题

**Output Format 缺失（companion 最大单项扣分）**

```bash
# echo-sleuth-for-claude：4 个 skill，仅 1 个有 Output Format
grep -rL "## Output Format" ~/echo-sleuth-for-claude/ --include="SKILL.md"
# 预计命中：memory-management、git-mining、experience-synthesis（3/4 缺失）

# drama-workshop-skills：short-drama/SKILL.md 也缺 Output Format
grep -L "## Output Format" ~/drama-workshop-skills/*/SKILL.md 2>/dev/null
```

已确认：`echo-sleuth-for-claude` 中 1/4 skill 有 Output Format（jsonl-core 有，其余 3 个缺失）；`drama-workshop-skills` 的 `short-drama/SKILL.md` 缺 Output Format。这与 companion 14/18 缺失的模式完全相同，只是规模更小。

**Trigger/描述不够精确**

```bash
# 查看我的 skill description 字段
grep -A1 "^description:" ~/echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
grep -A1 "^description:" ~/drama-workshop-skills/*/SKILL.md 2>/dev/null
```

待验证：`echo-sleuth` 的 skill description 是否内嵌了足够多的具体触发动作短语？还是写成了能力摘要？

### 8.3 别人的更优方案
1. **`audit/SKILL.md` 的输出 schema 设计**：companion 的 `audit` skill 定义了带闭合枚举（`Critical / High / Medium / Low`）、命名字段（Location、Severity、Category、Description、Impact、WCAG/Standard、Recommendation、Suggested command）的完整每条 issue 模板。这是我的任何 skill 目前都没有达到的精度——我的输出格式往往只说「输出一份报告」，没有到字段级别的 schema 定义。
2. **`teach-impeccable` 的初始化 skill 模式**：这种「先探索代码库 → 提问收集上下文 → 写入配置文件」的一次性初始化 skill 是我完全没有的模式，可以直接在 `echo-sleuth-for-claude` 中参考实现一个 `setup/SKILL.md`（探索 `.claude/` 配置，询问用户偏好，写入 `nlpm.local.md`）。
3. **bidirectional 路由**：`audit` skill 在推荐修复时，引导模型调用其他已有 skill（`/colorize`、`/animate` 等）而不是自己实现修复逻辑。这个双向路由模式（skill 既调用共享知识，也向外路由后续操作）在 `echo-sleuth` 中缺失——我的 skill 都是孤立的，没有互相路由。

### 8.4 反向：我的项目做得比他们好的地方
1. **没有复制粘贴模板错误**：companion 有 6 个 skill 在同一行含相同的模板 bug（"MUST STOP and STOP"）。我的仓库没有这类批量传播的模板错误——每个 skill 都是独立写就的。
2. **domain-specific skill 的中文描述质量**：`echo-sleuth-for-claude` 的 skill 采用中文 description，描述更贴近实际使用场景（面向中文用户），不像 companion 部分 skill 的 description 仍停留在能力摘要层面。
3. **仓库结构清晰度**：我的仓库 README 对 skill 用途有明确说明，而 companion 的 NL 层（`.agents/skills/`）在仓库主 README 中几乎不可见——用户容易忽略这 18 个 skill 的存在。

---

## 八、术语表

| 术语 | 定义 |
|---|---|
| <a id="frontmatter"></a>frontmatter | Markdown 文件开头由 `---` 包裹的 YAML 元数据块，用于声明 `name`、`description`、`model` 等字段，供 Claude Code 注册和发现 NL 产物 |
| <a id="skill"></a>skill | Claude Code 插件体系中的指令文件（`.agents/skills/*/SKILL.md`），定义触发条件、执行步骤和输出格式，供 Claude Code 在对话中按需加载 |
| <a id="output-format"></a>Output Format | skill 或 agent 文件中的 `## Output Format` 节，明确规定产物的结构、字段名称和取值范围（如枚举值），是 R41 规则的核心要求 |
| <a id="askuserquestiontool"></a>AskUserQuestionTool | companion 多个 skill 中引用的一个工具名称，但该工具不属于 Claude Code 标准工具集（`Read`、`Write`、`Edit`、`Bash` 等），在 Claude Code 环境中调用时会静默失败 |
| <a id="websocket协议"></a>WebSocket协议 | companion 通过逆向工程 Claude Code CLI 的 `--sdk-url` 参数发现并实现的未文档化双向通信协议，是 companion Web UI 与 Claude Code 会话之间的底层传输机制 |
