# ooiyeefei/ccc — 学习案例

**仓库**：https://github.com/ooiyeefei/ccc
**Stars**：405 | **来源**：upstream audit（exemplar_published=true，覆盖 <500 星门槛）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-22（基于当前 HEAD）
**主题标签**：`template-design`, `examples-driven`, `manifest-discipline`, `cross-reference`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`ooiyeefei/ccc` 是一个多插件 Claude Code 技能集合，核心定位是「创始人日常工作套件」——覆盖产品管理、UAT 测试、启动页 GTM、Excalidraw 图表生成、Streak 习惯追踪、Google Analytics 配置、安全设计等横跨创业早期各阶段的工作流。

关键事实：
- 审计时（2026-04-06）有 27 个 artifacts，现在已扩展至 **21 个独立 skills + 8 个 plugins**
- 作者 ooiyeefei 是新加坡籍独立开发者，项目持续演进，半年内 skills 数量增长约 50%
- 最值得关注的亮点：`product-management` 插件把 WINNING 优先级框架完整编码进 [SKILL.md](#skill-md)
- 在 NLPM exemplar 库中以 **R04/R07/R08/R09/R12/R17** 六条规则的正向示范入选

### 1.2 架构剖析

**目录结构**（当前 HEAD）：
```
ccc/
├── skills/                  # 21 个独立 skills（无 plugin 包装）
│   ├── streak/              # 习惯追踪（含 Python bot + Docker）
│   ├── excalidraw/          # 图表生成
│   ├── google-analytics-setup/
│   ├── secure-by-design/    # 安全架构 skill
│   ├── pitch-deck/
│   ├── htmldrop/
│   └── ... (共 21 个)
└── plugins/                 # 8 个 plugin 包
    ├── product-management/  # 最成熟，含 5 commands + 3 agents + 1 skill
    ├── mvp-launch/
    ├── deckling/            # PPT 生成
    ├── daily-chief/
    ├── rethink-surveys/
    ├── agentic-toolkit/
    ├── bash-safety-guard/
    └── ...
```

**文件类型分布**：21 个 SKILL.md + 约 20 个 command + 6 个 agent + 多个 hook
**编排关系**：两层结构——独立 skills（无 plugin 包装，用户直接安装）和 plugins（含 manifest，多文件协作）。`product-management` 插件内部有三层：SKILL → command → agent，agents 之间无直接调用，每个 command 独立路由到对应 agent。
**跨件契约**：`product-management/SKILL.md` 第 163-174 行有一个显式的「与 spec-kit 集成」section，用表格形式声明边界和移交机制（GitHub Issue 是移交媒介）。这是跨件契约的教科书写法。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「触发地图」而非「标签」——每个 skill 的 `description` 字段被用作调度表，而非简短摘要。`product-management` skill 的 description 用 12 个带引号的用户短语枚举所有激活场景（exemplar R04 正是来自此处）
- **解决什么问题**：创业早期「AI 应该如何配合 PM/设计/销售等角色工作」——作者把自己的工作流直接编码成可复用 skill，人肉知识 → NL 工件
- **核心 trade-off**：独立 skills 和 plugins 共存，前者安装成本低（无需 plugin.json），后者功能完整（多文件协作）。代价是两套管理规则并行，维护者需要在两个维度同步更新
- **反映的认知模型**：作者把 Claude Code skill 看作「把领域专业知识封装成可调用 API」——WINNING 框架、Balanced Coupling 思想等都是先有领域模型，再把模型编码进 NL 工件

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「多域 skill 平铺 + 重点域 plugin 深耕」

将通用/轻量工作流编写为独立 skill，将核心业务域（产品管理）封装为完整 plugin，两者共存于同一仓库。

模式特征清单：
- 特征 1：`skills/` 和 `plugins/` 并列，各有不同的安装路径和使用场景
- 特征 2：核心 plugin 内部分层（skill → command → agent），边缘功能只有 skill 层
- 特征 3：description 字段用多个 quoted 触发短语构成调度表（R04 exemplar 模式）
- 特征 4：`references/` 子目录用于存放查找表、流程图、模板，SKILL.md 保持简洁
- 特征 5：跨件边界在 SKILL.md 中用专门 section + 表格显式声明

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 创始人/独立开发者个人工作流 | ✅ 高度适用 | 跨域覆盖，各模块独立可用，不需要全部安装 |
| 团队共享标准工作流 | ✅ 中度适用 | plugins 部分可团队共享，需要维护 plugin.json |
| 单一深度领域（如只做代码审查） | ❌ 不适用 | 架构复杂度超过需求，单 plugin 足矣 |
| 需要 agents 之间强依赖编排 | ❌ 不适用 | ccc 的 agents 是平行的，无跨 agent 调用机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多域 skill 平铺 + plugin 深耕（本案例） | ooiyeefei/ccc | 覆盖广，轻量域无负担，重点域完整 | 双轨维护，plugin.json 易漂移 |
| 单一 plugin 深耕 | vladikk/modularity | 结构清晰，一次安装全功能 | 范围窄，不同域要建多个仓库 |
| 聚合清单式（无 skill 内容，只列链接） | ComposioHQ/awesome-claude-plugins | 发现成本低 | 质量参差，无法直接使用 |

### 2.4 改进空间

1. **当前问题**：`plugin.json` 版本字段与 SKILL.md 中 `version:` 字段不同步（0.1.0 vs 0.2.0）**改进做法**：在 `product-management/` 目录增加一个 `bump-version.sh` 脚本，发版时同步修改两处；或用 CI 检查 **预期收益**：杜绝用户安装到版本号不匹配的插件

2. **当前问题**：`/pm:review` 命令在 SKILL.md 文档中被引用，但 `review.md` 文件至今不存在 **改进做法**：要么创建 `review.md`，要么从 SKILL.md 文档中删除对 `/pm:review` 的所有引用 **预期收益**：消除工作流文档与实际可用命令之间的断层

3. **当前问题**：21 个独立 skills 没有顶层 `skills/README.md` 索引描述每个 skill 的适用场景 **改进做法**：增加 `skills/README.md`，一行一个 skill + 触发短语示例 **预期收益**：大幅降低新用户发现成本

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

得分 **96/100**（27 个 artifacts）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| plugins/deckling/commands/deckling.md | 65 | 缺 allowed-tools；无步骤编号；引用不存在的 skill |
| commands/streak-switch.md | 85 | 缺 allowed-tools；无空参数处理 |
| plugins/product-management/.claude-plugin/plugin.json | 95 | 版本号与 SKILL.md 不匹配 |
| 其余 22 个文件 | 95-100 | 小问题（模糊量词等） |

### 3.2 当时值得借鉴的模式

1. **触发地图 description（R04）** → 把 description 字段当 dispatch table 用，枚举 8-12 个用户可能的精确短语 → `skills/product-management/SKILL.md:2-3` → 直接复制这种写法，把自己 skill 的 description 改成列举所有激活场景

2. **显式跨件边界（R07）** → 在 SKILL.md 中用专门 section + 表格说明「我处理什么，相邻 skill 处理什么」 → `product-management/SKILL.md:163-174` → 每个与其他 skill 有业务边界的 SKILL.md 都应该有「Integration with X」section

3. **references/ 子目录外置查找表** → 把冗长的流程/模板/枚举值放到 `references/*.md`，SKILL.md 保持简洁 → `skills/streak/references/*.md` → 自己的 skill 超过 200 行时考虑提取子文档

4. **WINNING 框架完整编码** → 把领域框架（Pain × Timing × Execution Capability）的每个维度都用评分表 + 示例写进 skill，变成可机械执行的决策树 → `product-management/SKILL.md:57-104` → 把自己的判断标准编码成表格，而非「请根据上下文判断」

### 3.3 当时的缺陷

1. **`skills/deckling-pptx.md` 引用断链** → deckling 命令依赖一个不存在的 skill，这是经典的「manifest 声明了但文件没写」问题。根本原因：重构删除了 skill 文件但忘记更新引用方。自查：我有没有 command 或 SKILL.md 里引用了不存在的文件？→ **这是高影响 bug，功能完全不可用**

2. **6 条 streak 命令全部缺 `allowed-tools`** → 不声明 allowed-tools 会导致 Claude 可能尝试使用任何工具，行为不可预测。根本原因：批量复制命令模板时漏掉了 frontmatter 更新。自查：我在批量复制 command 模板时是否总是更新 frontmatter？

3. **`/pm:review` 文档幽灵引用** → SKILL.md 明确记录了 `/pm:review` 作为标准工作流步骤，但命令文件不存在。根本原因：先写文档（规划），后写实现，规划变更后文档未同步删除。自查：我的文档里有没有「规划中」的功能被当做已实现功能描述？

### 3.4 当时的优化机会

1. `deckling.md` 需要增加 Output Format section，明确生成的 PPT 文件放在哪里
2. 6 个 streak 命令应该批量增加 `allowed-tools` frontmatter
3. `streak-notify.py` 的 Telegram Bot token 应该改用环境变量而非存在 markdown 文件里

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `skills/deckling-pptx.md` 不存在 | `ls skills/deckling-pptx.md` | **已修复** ✅ — 文件现在存在 | 修复速度快，可能是用户反馈触发 |
| `plugin.json` version 0.1.0 vs SKILL.md 0.2.0 | `cat plugin.json \| jq .version` | **持续** ❌ — 三个月后仍未修 | 版本同步不在作者的发版 checklist 里 |
| `/pm:review` 命令文件缺失 | `ls plugins/product-management/commands/` | **持续** ❌ — 仍无 review.md | 这个 workflow gap 可能已在作者计划中但未优先 |

### 4.2 架构演进

审计时 27 个 artifacts → 现在 21 个独立 skills + 8 个 plugins（artifacts 总数约 50+）。明显扩张方向：

- **新增领域 skills**：`secure-by-design`、`google-analytics-setup`、`pitch-craft`、`pitch-package`、`pitch-deck`、`demo-video`、`motion-video`、`self-improving-systems` — 从「产品管理」向「创业全周期」扩张
- **新增 plugins**：`agentic-toolkit`、`bash-safety-guard`、`rethink-surveys`、`daily-chief` — 工具类和习惯类的丰富
- **references/ 子目录普及**：几乎每个新 skill 都有 `references/` 子目录，说明作者已将外置查找表作为标准实践

这说明作者在六个月内意识到：单个 SKILL.md 装不下的内容应该拆分到 references/ 子目录，而不是堆进主文件。

### 4.3 新增的可学习模式

**外置知识库模式（References 子目录）**：所有新 skill 都使用 `skill-name/references/*.md` 结构，将模板、清单、案例等查找表外置，SKILL.md 保持精简可读。这是审计时已有但在新 skills 中标准化推广的模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **分层架构**：我的 bureau 项目也采用 skills + commands 分层，概念上与 ccc 的 plugin 内部结构一致
2. **SKILL.md 专注单一职责**：我的每个 SKILL.md 也聚焦一个具体任务，没有大杂烩
3. **README 索引**：我在各层级都有 README，比 ccc 当前状态要好
4. **文档驱动**：先写 SKILL.md 定义行为，再迭代优化——与 ccc 的设计思路一致

### 5.2 挑战 / 验证

这次案例挑战了我的一个假设：「description 字段写一句话摘要就够了」。ccc 的 R04 模式证明 description 更应该是触发地图——覆盖用户可能的各种表达方式而非只描述功能。这个认知值得我回去改写我的所有 SKILL.md description。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的命令是否引用了不存在的 skill 或 agent
# 先列出所有 skill 名称
find ~/.claude/skills /tmp/my-repos/MarkQWu-*/skills /tmp/my-repos/MarkQWu-*/plugins -name "SKILL.md" 2>/dev/null | \
  xargs grep -l "^name:" | head -20

# 检查 command 里是否有悬空引用
grep -rn "skills:\|agents:" /tmp/my-repos/MarkQWu-bureau/commands/ 2>/dev/null
```
命中后：逐条核查引用的 skill/agent 文件是否实际存在，不存在则要么创建要么删除引用。

```bash
# 检查 description 是否只有一句话摘要（而非触发地图）
grep -A2 "^description:" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | head -30
```
命中后：把摘要式 description 改写为列举 4-8 个用户触发短语的触发地图，参考 ooiyeefei/ccc 的 product-management SKILL.md。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的所有 skills 增加触发地图式 description
   - **为何可行**：bureau 的每个 skill 都有明确的使用场景，容易枚举 4-6 个触发短语
   - **第一步**：打开 `skills/recall/SKILL.md`，把现有 description 改为「Use when: 用户想查询历史知识、问 X 从哪来、有没有见过 Y、帮我找 Z 相关的记录…」—— 10 分钟可完成一个

2. **想法**：为 bureau 的 skills 增加 `references/` 子目录外置查找表
   - **为何可行**：bureau 的 capture/compile skills 有可能需要模板，目前可能内嵌在 SKILL.md 里
   - **第一步**：检查 `skills/capture/SKILL.md` 是否超过 150 行，如果是，把查找表部分提取到 `skills/capture/references/`

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 ooiyeefei/ccc 的核心目的**：用 Claude Code skills 覆盖创业早期创始人的全工作流（PM/GTM/安全/市场/习惯）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是 Claude Code plugin，都有 skill + command 分层 | bureau 聚焦知识管理，ccc 覆盖创业全域 | 高（架构模式可直接借鉴）|
| MarkQWu/gstack | 中 | 都是多 skill 集合，服务日常工作流 | gstack 更聚焦技术人员，ccc 更全栈创业视角 | 高（description 写法改进）|
| MarkQWu/graphify | 低 | 都有 AI 工具属性 | graphify 是知识图谱工具，与 ccc 领域无交集 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code skills 集合 | 域完全不同（短剧 vs 创业） | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 6 个命令缺 `allowed-tools` | `grep -L "allowed-tools" commands/*.md` | **MarkQWu/bureau 命中 11/11 个命令** — bureau 所有命令都没有 allowed-tools | 高 |
| `description` 只写摘要而非触发地图 | 检查 SKILL.md description 字段 | gstack 和 bureau 的 skills 大概率都是摘要式 | 中 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/commands/lint.md` 等 11 个命令 → 在 frontmatter 中增加 `allowed-tools: Bash, Read, Write, Edit`（根据实际使用工具填写）→ 每个 10 分钟，全部约 2 小时

### 7.3 别人的更优方案

1. **领域**：references/ 外置查找表
   - **本案例做法**：每个 skill 有 `references/*.md` 子目录，存放模板、流程图、枚举值（如 `skills/streak/references/types.md`, `flows-detailed.md`）
   - **我的项目现状**：bureau 的 SKILL.md 可能把所有内容都堆在一个文件里（待验证）
   - **如何借鉴**：检查 bureau 各 SKILL.md 行数，超过 150 行就考虑拆分 references 子目录

2. **领域**：WINNING 框架编码（领域框架 → 机械可执行决策表）
   - **本案例做法**：把抽象的优先级框架编码成评分表（Pain 1-5 × Timing 1-3 × Execution 1-4），Claude 可以机械执行
   - **我的项目现状**：gstack 的部分 skills 可能还在用「根据上下文判断」这类模糊指导
   - **如何借鉴**：把任何判断性指令（「好的 PR」「重要的 issue」）改写为可评分的多维表格

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 和 `model:` 声明的完整性
  - **我的项目现状**：虽然 bureau 的 commands 目前缺 allowed-tools（这是个 bug），但我的 SKILL.md 有 `allowed-tools` 声明（vladikk 案例对比后会更清楚）
  - **本案例做法（弱在哪）**：ccc 的 6 个 streak 命令都缺 allowed-tools，历史上 deckling 命令也有类似问题
  - **意义**：这提醒我，批量复制 command 模板是最高风险点，应建立 checklist 每次核查 frontmatter

---

## 八、术语表

### <a name="skill-md"></a>SKILL.md
> Claude Code 技能（Skill）的定义文件，用 Markdown + frontmatter 编写。frontmatter 声明名称、描述、允许的工具等元数据；正文是给 Claude 看的指令。Claude Code 在加载时解析这个文件并「学会」这个技能。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，用来声明文件的元数据（如 `name`、`description`、`model`、`allowed-tools`）。

### <a name="manifest"></a>manifest
> 插件的清单文件（`plugin.json`），告诉 Claude Code 这个插件包含哪些 commands、skills、agents。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="allowed-tools"></a>allowed-tools
> command 或 skill 的 frontmatter 字段，声明这个工件允许 Claude 使用哪些工具（如 `Bash`、`Write`、`Read`）。不声明会让 Claude 有权尝试使用任何工具，行为不可预测。

### <a name="trigger-map"></a>触发地图（Trigger Map）
> NLPM R04 规则要求的 description 写法：不是一句话描述功能，而是枚举用户可能说出的各种激活短语（如「分析我的产品」「研究竞争对手」「创建 PRD」）。这让 Claude 在判断是否调用这个 skill 时有明确的模式匹配依据。
