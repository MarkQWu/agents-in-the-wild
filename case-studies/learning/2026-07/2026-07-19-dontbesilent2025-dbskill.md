# dontbesilent2025/dbskill — 学习案例

**仓库**：https://github.com/dontbesilent2025/dbskill
**Stars**：0（registry，exemplar_published=true）| **来源**：upstream
**Audit 日期**：2026-04-13（历史快照）| **生成日期**：2026-07-19（基于当前 HEAD）
**主题标签**：`cross-reference`, `router-channels`, `security-gate`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

面向中文创业者与内容创作者的商业决策 AI 技能集，由 dontbesilent（Twitter: @dontbesilent）创建，当前版本 v2.18.0。作者从 16,152 条公开推文中筛选出 4,176 个「知识原子」，将其中的方法沉淀为 29 个可直接调用的技能。支持 Claude Code、豆包、WorkBuddy、Codex 等多种 Agent 平台，核心用户是需要用 AI 做商业诊断、内容创作、行动推进的中文用户。

### 1.2 架构剖析

- **目录结构**：

```
dbskill/
├── skills/
│   ├── dbs/SKILL.md          # 路由器：智能分发入口（三模式：新手/前置路由/后续导航）
│   ├── dbs-action/           # 行动推进
│   ├── dbs-agent-migration/  # Agent 迁移（新）
│   ├── dbs-ai-check/         # AI 工具评估
│   ├── dbs-benchmark/        # 标杆测试
│   ├── dbs-bridge/           # 跨平台桥接（新）
│   ├── dbs-chatroom/         # 商业讨论室（新，拆自 chatroom-austrian）
│   ├── dbs-chatroom-austrian/# 奥派思维框架
│   ├── dbs-content/          # 内容创作
│   ├── dbs-content-system/   # 内容系统（新）
│   ├── dbs-decision/         # 决策分析（新）
│   ├── dbs-deconstruct/      # 拆解分析
│   ├── dbs-diagnosis/        # 商业诊断
│   ├── dbs-goal/             # 目标设定（新）
│   ├── dbs-good-question/    # 提问优化（新）
│   ├── dbs-hook/             # 内容钩子
│   ├── dbs-knowledge/        # 知识库管理（新）
│   ├── dbs-learning/         # 学习追踪（新）
│   ├── dbs-report/           # 报告生成（新）
│   ├── dbs-resonate/         # 共鸣设计（新）
│   ├── dbs-restore/          # 状态恢复（新）
│   ├── dbs-save/             # 状态保存（新）
│   ├── dbs-script-flow/      # 脚本流程（新）
│   ├── dbs-skill-cleaner/    # 技能清理（新）
│   ├── dbs-slowisfast/       # 慢即是快
│   ├── dbs-spread/           # 传播设计（新）
│   ├── dbs-update/           # 内容更新（新）
│   ├── dbs-wechat-html/      # 微信 HTML 排版（新）
│   └── dbs-xhs-title/        # 小红书标题
├── scripts/record-demo.sh    # GIF 录制工具（开发者工具）
├── docs/skill-link-map.svg   # 技能关联图可视化
└── .claude-plugin/marketplace.json
```

- **文件类型分布**：29 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook
- **编排关系**：`dbs` 是路由器，读取对话上下文后将用户路由到最合适的子技能。三种模式：新手引导（首次使用）、前置路由（任务开始前）、后续导航（任务结束后推荐下一步）。路由完成后，后续技能可以通过末尾的「下一步建议」反向调用 `dbs` 进行续航导航。
- **跨件契约**：每个技能的「下一步建议」节引用其他 dbs-* 技能；所有技能通过 `/技能名` 格式互相调用；路由表维护在 `dbs/SKILL.md` 的主菜单中。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「用户不需要先学会用哪个工具」—— 把选择复杂度完全封装在路由器里，用户只需描述处境，系统选择工具。
- **解决什么问题**：创业者和内容创作者面临的决策和执行问题是模糊且多维度的（商业诊断 + 内容执行 + 行动推进往往交织），用户无法预知该用哪个工具。
- **做了什么 trade-off**：路由逻辑在 NL 层（`dbs/SKILL.md`）而非代码层，可读性强但无法用 unit test 验证路由正确性；选择了可维护性优先。
- **反映什么认知模型**：作者把知识积累（推文 → 知识原子 → 方法 → 技能）视为有损压缩过程，强调技能内容需要有「可以立刻执行的下一步」而非仅仅分析框架。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「三模式路由器 + 技能矩阵」架构**

核心特征：
- **特征 1**：路由器 (`dbs`) 有明确的「模式判断」逻辑：新手/前置路由/后续导航三种行为截然不同
- **特征 2**：路由器读取对话历史来判断模式，而不只看当前输入——「本次对话里有没有任何 dbs-* 技能的输出？」
- **特征 3**：每个子技能末尾有「下一步建议」，形成技能链的自然闭环
- **特征 4**：知识基础层（4,176 个知识原子）是所有技能的数据来源，技能是知识的「精华萃取」而非凭空写就
- **特征 5**：多平台支持（`claude plugin install` + 豆包 + Codex），通过单仓库维护多目标发布

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 用户群体对 AI 技能调用不熟悉 | ✅ 高度适用 | 路由器将选择负担从用户转移到系统 |
| 业务场景有多个高度相关但不同的工作流 | ✅ 适用 | 技能矩阵覆盖宽度 + 路由器做深度 |
| 需要跨会话持续推进某个项目 | ✅ 适用（有 dbs-save/restore） | 状态保存/恢复技能形成了跨会话工作流 |
| 工作流场景固定且明确 | ❌ 过度设计 | 直接用单技能即可，路由器增加了复杂度 |
| 技术类工作（代码、调试、测试） | ❌ 不适用 | dbskill 定位是商业/内容，非开发工具 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 三模式路由器（本仓库） | dontbesilent2025/dbskill | 覆盖新手/老手/续航三种使用模式，无感知 | 路由逻辑在 NL 层，准确性难以量化验证 |
| Hook 自动触发 | czlonkowski/n8n-skills | 完全无需用户主动调用 `/dbs` | 需要 hooks 基础设施，复杂度更高 |
| 平铺技能 + 用户自选 | addyosmani/agent-skills | 简单透明 | 用户认知负担高，技能数量超过 8 个后很难记住 |

### 2.4 改进空间

1. **当前问题**：无 hooks 层，用户仍需记得输入 `/dbs` **改进做法**：添加 `hooks.json` + SessionStart hook，在会话开始时自动注入 `dbs` 路由器 **预期收益**：消除用户首次使用时「我该用哪个命令」的困惑
2. **当前问题**：`docs/skill-link-map.svg` 是手绘 SVG，需手动更新 **改进做法**：用 `docs/skill-link-map.mmd`（已有）生成 SVG，集成到 CI **预期收益**：架构图与代码自动同步
3. **当前问题**：路由器没有显式的路由测试 **改进做法**：在 evaluations/ 目录添加路由判断的 JSON 测试用例（参考 czlonkowski 的 evaluations 目录结构）**预期收益**：路由准确性变成可量化的指标

---

## 三、过去审查发现（2026-04-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-13 当时得分 90/100，SECURITY REVIEW（HIGH × 3）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/dbskill-upgrade/SKILL.md | 78/100 | 内嵌破坏性 shell 脚本；非标准 frontmatter；仅中文 description |
| skills/dbs-ai-check/SKILL.md | 86/100 | 引用了不存在的 `/文风分析` 技能 |
| skills/dbs-benchmark/SKILL.md | 90/100 | 「合理时间内」模糊量词 |
| skills/dbs-diagnosis/SKILL.md | 90/100 | 「合适的时机」模糊量词 |
| skills/dbs-xhs-title/SKILL.md | 90/100 | 无重大问题 |
| skills/dbs/SKILL.md | 91/100 | chatroom-austrian 未在主菜单列出 |
| skills/dbs-deconstruct/SKILL.md | 91/100 | 无重大问题 |

**HIGH 安全问题**（3 项）：`dbskill-upgrade/SKILL.md` 内嵌了 `rm -rf ~/.claude/skills/dbs*`、`git clone https://github.com/...`、`cp` 三条组合命令，实质上是一个无完整性校验的自动更新器，若上游仓库被攻陷则形成供应链攻击向量。

### 3.2 当时值得借鉴的模式

1. **三模式路由器** → `dbs/SKILL.md` 明确描述三种模式（新手/前置/后续）的判断逻辑，并要求路由器「优先提取对话上下文，能判断时直接路由，不重复向用户索取已提供的信息」。根本原因：避免路由器成为信息收集机器而不是决策机器。借鉴：我的 bureau 路由技能（guide）可参考这种三模式判断框架。

2. **状态保存/恢复技能对** → `dbs-save` + `dbs-restore` 形成跨会话状态延续能力。根本原因：对话结束后状态默认清零，这两个技能是一种最小可行的「会话记忆」补丁。借鉴：bureau 的 capture + recall 技能对与此类似，但我的版本有更强的知识库支撑。

3. **多语言 README** → 仓库有简中/繁中/英/日/韩 5 个 README 版本，对应多平台多用户群。根本原因：中文创作者工具面向全球华人圈，语言一致性降低首次使用摩擦。借鉴：如果我的工具面向多语言用户，这是很低成本的用户覆盖扩展。

4. **技能链末尾「下一步建议」** → 每个子技能执行完成后，提供 2-3 个有上下文感知的下一步路由建议。根本原因：单次技能执行结束后的断层感是用户流失的高风险点；显式的「如果你想继续，可以...」降低了续航摩擦。借鉴：我的 bureau 技能执行完成后，可以在末尾加「下一步可运行 /recall 查看历史快照」。

### 3.3 当时的缺陷

1. **`dbskill-upgrade` 内嵌破坏性命令（HIGH × 3）** → 在一个 NL 技能文件里内嵌了 `rm -rf`、`git clone`、文件覆盖三条组合操作，且没有哈希校验。为什么失败：上游仓库一旦被攻陷或被 force-push 替换，用户执行 `/dbskill-upgrade` 后 Claude 会直接安装被篡改的代码。自查：我的技能文件中有没有嵌入 shell 命令的地方？→ 检查后确认没有（我的技能只描述应该做什么，不直接执行系统命令）。

2. **`dbs-ai-check` 引用不存在的 `/文风分析`** → 在「下一步建议」中引用了一个仓库里不存在的技能。为什么失败：用户跟着建议走到死胡同，产生信任损耗。自查：我的技能中「下一步建议」里引用的技能名是否都实际存在？→ 值得检查。

3. **`dbskill-upgrade` 仅有中文 description** → 所有其他技能都是中英双语 description。为什么失败：这意味着该技能在英文环境下不会被正确触发，且与整体风格不一致。自查：我的技能 description 有没有语言一致性要求？

### 3.4 当时的优化机会

1. `dbskill-upgrade/SKILL.md` 整体重新设计：移除内嵌 shell，改为说明「请用户手动运行 `claude plugin install dontbesilent2025/dbskill`」
2. `dbs-ai-check/SKILL.md` 下一步建议中删除 `/文风分析`
3. 为所有 description 字段补充英文版本

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `dbskill-upgrade` HIGH 安全问题 | `ls skills/dbskill-upgrade/` | **已修**：该目录已被完全删除 | 作者选择了最彻底的修复方式：移除整个升级技能 |
| `dbs-ai-check` 引用不存在的 `/文风分析` | `grep -rn "文风分析" skills/` | **已修**：grep 无输出，引用已删除或被重新设计 | 死链接已清理 |
| 模糊量词「合理时间内」「合适的时机」 | `grep -rn "合理\|合适" skills/dbs-benchmark/SKILL.md skills/dbs-diagnosis/SKILL.md` | 需要实地确认（工具限制） | 暂无确认数据 |

### 4.2 架构演进

从 audit 时的 12 个技能 → 现在的 29 个技能，增加了 17 个新技能，版本从未知升级到 v2.18.0。

最重要的演进信号：
- **移除了 `dbskill-upgrade`**：从激进的内嵌自动更新器，退回到依赖平台原生安装的保守模式。这是对安全边界的重新认知。
- **引入了 `dbs-save`/`dbs-restore`**：说明作者意识到跨会话状态是用户的真实需求，而不只是当次对话的问题解决。
- **引入了 `dbs-knowledge`**：知识库管理技能出现，意味着用户数据规模大到需要专门的技能来管理，而不只是依赖临时对话。
- **新增多个内容创作技能**（dbs-content-system, dbs-wechat-html, dbs-spread）：说明用户从「业务诊断」扩展到了「内容分发执行」，产品目标扩大了。

### 4.3 新增的可学习模式

**dbs-skill-cleaner**：一个用于清理本地已安装的旧版或冗余技能的维护工具。这是一个「技能维护」元技能，解决用户随着技能积累产生的版本混乱问题。在技能生态成熟后，维护工具本身也需要作为技能来分发——这是 dbskill 从「工具箱」到「工具箱维护系统」的演进。

---

## 五、校准

### 5.1 我已经在做对的

- **路由器技能设计**：bureau 的 `guide` 技能承担类似 `dbs` 的路由功能，覆盖了不同使用场景的分发
- **无破坏性系统操作**：我的技能文件从不内嵌 shell 命令，这个 HIGH 安全问题我不存在
- **跨会话状态设计**：bureau 的 capture + recall 技能对，与 dbs-save/restore 的目标相近

### 5.2 挑战 / 验证

**这次案例挑战了我的一个假设**：我以前认为「知道要用哪个技能」是用户应该承担的认知负担。dbskill 的三模式路由器和 17 个技能的快速扩张说明：当技能库规模超过 15 个时，路由器的作用不是「便利」而是「必须」。

另一个被验证的判断：内嵌 shell 命令到 NL 技能文件是高风险的设计——即使意图是自动更新，代价是潜在的供应链攻击面。这个教训适用于我自己：任何「让 Claude 自动帮你做系统级操作」的设计，都需要在技能文件外有一层独立的确认机制。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能的「下一步建议」中引用的技能名是否都实际存在
grep -rn "\/[a-z_-]*" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md | \
  grep -v "^Binary" | grep "下一步\|next step\|建议" | head -20

# 检查我的技能文件中是否有内嵌 shell 命令
grep -rn '`rm\|`git clone\|`curl\|`cp -r' \
  /tmp/my-repos/MarkQWu-bureau/skills/ \
  /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null | head -10
```

命中后怎么办：
- 引用不存在的技能名 → 删除或替换为实际存在的技能
- 内嵌 shell 命令 → 重构为「请用户手动运行 XXX」的说明，不让技能直接执行

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的 guide 技能添加三模式判断逻辑（新手/前置路由/后续导航）
   - **为何可行**：bureau 已有路由技能，三模式分离会让路由更精准，减少「每次都问一遍」的冗余
   - **第一步**：打开 `bureau/skills/guide/SKILL.md`，在「如何判断模式」节参照 dbskill 的三条判断优先级（对话历史是否有 bureau-* 输出）；约 20 分钟

2. **想法**：添加 `dbs-save`/`dbs-restore` 类似的状态快照技能对到 gstack
   - **为何可行**：gstack 的长任务（如 land-and-deploy）经常跨多个会话，状态丢失是实际痛点
   - **第一步**：在 gstack 目录创建 `context-save/SKILL.md` 和 `context-restore/SKILL.md`，描述将当前任务进度保存到特定文件的标准化流程；约 30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 dontbesilent2025/dbskill 的核心目的**：把创业者的商业知识和内容方法论结构化为 AI 技能，通过路由器降低使用门槛，让用户只描述处境即可获得可执行建议

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是多技能 Claude Code 插件，有路由入口，以知识管理为核心目标 | bureau 管理的是 AI 会话知识，dbskill 管理的是个人业务知识；bureau 有更强的知识库基础设施 | 高 |
| MarkQWu/gstack | 中 | 多技能工具箱，有路由设计 | gstack 面向开发工作流，dbskill 面向商业内容场景 | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 下一步建议引用不存在的技能 | `grep -rn "\/[a-z]" bureau/skills/*/SKILL.md` | 需实地验证 | 中 |
| 路由器无多模式分支逻辑 | 阅读 bureau/skills/guide/SKILL.md 的判断流程 | guide 技能未明确区分新手/续航模式 | 低 |
| 内嵌破坏性命令 | `grep -rn "rm -rf\|git clone" */SKILL.md` | 未命中（0 结果）| 无 |

**命中后的具体行动建议**：
- bureau/skills/guide/SKILL.md → 在「判断逻辑」节添加「本次对话是否有 bureau-* 技能的输出」作为优先判断条件 → 15 分钟可完成

### 8.3 别人的更优方案

1. **领域**：三模式路由器（新手/前置/续航）
   - **本案例做法**：`dbs/SKILL.md` 明确定义三种模式的触发条件，模式 A/B/C 互不重叠，判断顺序有优先级
   - **我的项目现状**：bureau/guide 技能的模式判断较模糊，没有明确区分「首次使用者」和「续航使用者」
   - **如何借鉴**：在 guide/SKILL.md 的前置判断节，增加「本次对话是否存在 bureau-* 技能的输出」→ 有则进入续航模式，无则进入引导模式

2. **领域**：知识来源可溯（4,176 个知识原子）
   - **本案例做法**：作者在 README 明确说明技能内容是从 16,152 条推文中结构化出来的，给了技能内容一个「数据血统证明」
   - **我的项目现状**：gstack 和 bureau 的技能内容来源未在文档中说明
   - **如何借鉴**：在 bureau 的 README 中说明「技能内容基于 X 个实际 AI 会话的提炼」，增加可信度

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：知识库基础设施（持久化 + 版本化）
- **我的做法**：bureau 有完整的 capture → compile → review → query 管道，知识以 logbook 形式持久化，通过信任层门控
- **本案例做法**（弱在哪）：dbskill 的 dbs-save 技能本质上是在当次对话保存状态到文件，没有 bureau 这样的知识管理管道
- **意义**：如果和 dbskill 的作者交流，bureau 的知识持久化设计是一个清晰的差异化点；如果我要给上游 PR，可以提议「用 bureau 的 compile 模式重构 dbs-save 的持久化机制」

---

## 八、术语表

### <a name="知识原子"></a>知识原子

> 作者将推文中的单个观点、方法或洞见拆解到不可再分的最小单元，称为「知识原子」。例如「提高转化率的关键是减少决策步骤」是一个知识原子。通过 4,176 个知识原子的结构化，才能提炼出 29 个「方法技能」。

### <a name="供应链攻击"></a>供应链攻击（Supply Chain Attack）

> 通过破坏软件的「上游来源」（如 GitHub 仓库、npm 包）而不是直接攻击用户，让用户在正常使用工具时执行恶意代码。`dbskill-upgrade` 的 `git clone` 设计是一个典型的供应链攻击面——如果 GitHub 仓库被攻陷，任何运行 `/dbskill-upgrade` 的用户都会安装恶意代码。

### <a name="frontmatter"></a>frontmatter

> Markdown 文件顶部用 `---` 包裹的 YAML 配置，声明技能的 `name`、`description` 等元数据。Claude Code 通过解析 frontmatter 来注册技能。`dbskill-upgrade` 有非标准的 `trigger` 字段，不被 Claude Code 标准 schema 识别。

### <a name="router-skill"></a>路由器技能

> 专门负责「理解用户处境 → 选择最合适的子技能 → 分发执行」的技能。自身不执行任何业务逻辑，只做「谁来处理这个问题」的决策。`dbs` 是本仓库的路由器技能，类比于 API 网关中的路由层。
