# vladikk/modularity — 学习案例

**仓库**：https://github.com/vladikk/modularity
**Stars**：337 | **来源**：upstream audit（exemplar_published=true，覆盖 <500 星门槛）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-22（基于当前 HEAD）
**主题标签**：`template-design`, `cross-reference`, `single-purpose`, `examples-driven`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

`vladikk/modularity` 是一个四 skill Claude Code 插件，核心是把软件作者 Vlad Khononov 自己写的书《Balancing Coupling in Software Design》里的「[Balanced Coupling 模型](#balanced-coupling)」编码进可执行的 AI skill，让 Claude 能帮用户做模块化架构设计、评审现有代码的耦合度、生成设计文档。

关键事实：
- 仅 4 个 SKILL.md + 1 个 plugin.json，**迄今零 bug，零安全发现**
- 作者 Vlad Khononov 是 DDD（Domain-Driven Design）领域知名作家，`balanced-coupling` 是他自己提出的理论框架
- NLPM 得分 **98/100**，唯一扣分是 3 个措辞问题（「good」「particularly」缺 model:）
- 在 NLPM exemplar 库以 **R04/R05/R07/R08** 四条规则正向示范入选
- 版本 `1.5.0`——经过多次迭代，结构稳定

### 1.2 架构剖析

**目录结构**（当前 HEAD，与审计时基本相同）：
```
modularity/
├── .claude-plugin/
│   ├── plugin.json        # manifest，含 name/description/version/author
│   └── marketplace.json   # 市场元数据
├── skills/
│   ├── balanced-coupling/ # 参考资料 skill（user-invocable: false）
│   │   └── SKILL.md       # 330 行，全量 Balanced Coupling 理论
│   ├── design/            # 高层设计 skill（入口之一）
│   │   └── SKILL.md       # 248 行
│   ├── review/            # 代码审查 skill（入口之一）
│   │   └── SKILL.md       # 89 行
│   └── document/          # 文档生成 skill（内部依赖，non-invocable）
│       ├── SKILL.md        # 154 行
│       └── assets/
│           └── template.html
├── ai.txt                 # 给 AI 的简介文档
└── README.md
```

**文件类型分布**：4 个 SKILL.md，无 command，无 agent，无 hook
**编排关系**：**依赖注入（skills: frontmatter）** 实现跨件加载：
- `design` 和 `review` 都在 frontmatter 里声明 `skills: [balanced-coupling]`，让 Claude 在调用前自动预加载框架知识
- `review` 声明 `skills: [balanced-coupling, document]`，同时注入框架和文档生成能力
- `document` 和 `balanced-coupling` 都是 `user-invocable: false`，只能被其他 skill 引用，不暴露给用户

**跨件契约**：通过 `user-invocable: false` 明确区分「公开 API」和「内部实现」。这是 [frontmatter](#frontmatter) 约定的典范——用元数据而非文档约定来强制边界。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「知识注入优于上下文复制」——把 Balanced Coupling 理论写在单独的参考 skill 里，被需要的 skill 按需注入，而不是在每个 skill 里重复写理论
- **解决什么问题**：软件架构师需要一个能够理解并应用 Balanced Coupling 模型的 AI 助手——这个模型有专门术语、公式和决策逻辑，普通 Claude 没有被训练过
- **核心 trade-off**：把领域理论完整写进 `balanced-coupling/SKILL.md`（330 行）换来了 `design` 和 `review` 的简洁（89-248 行）。代价是 `balanced-coupling` 文件长，但由于是参考文档而非指令，可读性依然好
- **反映的认知模型**：作者把 Claude Code skill 视为「把自己的领域专业知识做成 Claude 可调用的 API」——书里的框架直接变成了 Claude 的内置知识

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「公开入口 + 私有知识库注入」

将领域知识封装为 non-invocable 的参考 skill，通过 frontmatter 的 `skills:` 字段按需注入到任意入口 skill。

模式特征清单：
- 特征 1：有明确的「公开 API」（user-invocable: true）和「私有知识库」（user-invocable: false）区分
- 特征 2：知识库 skill 纯描述性，无指令步骤（参考手册），入口 skill 纯指令性，无知识重复（执行步骤）
- 特征 3：frontmatter 的 `skills:` 字段实现 [依赖注入](#dependency-injection)，明确声明依赖关系
- 特征 4：入口 skill 专注自己的流程，假设领域知识已被注入（不重复解释模型）
- 特征 5：所有 skill 都有 `argument-hint` 字段，引导用户提供正确格式的输入

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有成体系的领域框架需要灌入 Claude | ✅ 高度适用 | 知识库 skill 是最干净的框架注入方式 |
| 多个入口 skill 共享同一知识基础 | ✅ 高度适用 | 一次写，多处注入，无重复 |
| 临时性、一次性任务 | ❌ 不适用 | 建 plugin 的启动成本超过收益 |
| 知识本身变动频繁 | ❌ 不适用 | frontmatter 注入是静态加载，更新不实时 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 公开入口 + 私有知识注入（本案例） | vladikk/modularity | 无重复、边界清晰、可组合 | 需要规划依赖图，新 skill 需要更新 frontmatter |
| 单 SKILL.md 大文件 | 多数个人 skill | 零依赖，无需考虑注入 | 文件变长后维护困难 |
| Command + Agent 分离 | ooiyeefei/ccc product-management | 支持多步骤异步操作 | 复杂度高，适合有状态工作流 |

### 2.4 改进空间

1. **当前问题**：`design/SKILL.md` 没有 `model:` 字段，作为一个多步骤交互式 workflow skill，模型选择会直接影响输出质量 **改进做法**：加 `model: claude-sonnet-5` 或按需选择，明确声明 **预期收益**：防止用户切换到低能力模型后 skill 退化

2. **当前问题**：两处措辞（「good」「particularly」）让 Claude 有无约束的解释空间 **改进做法**：把「good Balanced Coupling decisions」改为「design decisions that satisfy the three-dimensional Balanced Coupling balance rule (STRENGTH XOR DISTANCE)」 **预期收益**：NL 得分从 98 → 100，更重要的是减少 Claude 的模糊自由裁量

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

得分 **98/100**，SECURITY: **CLEAR**，零 bug，零安全问题。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/design/SKILL.md | 93 | 无 model: 字段；"good" 措辞模糊 |
| skills/review/SKILL.md | 98 | "particularly" 措辞模糊 |
| 其余 3 个文件 | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **触发地图 description（R04）** → `balanced-coupling` skill 的 description 枚举了 10 个不同的用户触发场景，每个场景解决不同的设计问题 → `skills/balanced-coupling/SKILL.md:3-11` → 真正的广度：「设计新系统」「评审现有代码」「决定是否拆分模块」「理解分布式单体」是不同用户、不同阶段的真实触发词

2. **体长控制（R05）** → 最长的 `balanced-coupling`（330 行）通过 XOR/AND 布尔表达式 + ASCII 表格压缩复杂框架，而非用散文重复解释 → 「MODULARITY = STRENGTH XOR DISTANCE」两行替代了两页解释 → 把自己的决策逻辑压缩成数学表达式或查找表

3. **`user-invocable: false` 的依赖注入模式（R07 + 新模式）** → `balanced-coupling` 和 `document` 被声明为不可直接调用，只能通过其他 skill 的 `skills:` 注入 → 这是「内部实现 vs 公开 API」的 NL 版本 → 任何一个知识库型 skill（参考资料、领域模型）都应该考虑设为 non-invocable

4. **`skills:` frontmatter 的依赖注入** → `review/SKILL.md` 的 frontmatter 声明 `skills: [balanced-coupling, document]`，Claude 在运行时会先加载这两个 skill 的内容 → 比在每个 skill 里复制粘贴知识更优雅 → 我的任何需要「背景知识 + 执行步骤」的 skill 都可以拆成两层

### 3.3 当时的缺陷

1. **`design/SKILL.md` 缺 `model:` 字段** → 这个 skill 有多步骤交互（AskUserQuestion + Write + Read），事实上是一个需要较强推理能力的 agent，但没有声明应该用哪个模型。根本原因：作者可能假设用户会用默认模型。自查：我有没有需要高推理能力的 skill 没有声明 model？→ **影响：用户切换到 haiku 后质量断崖式下降**

2. **「good Balanced Coupling decisions」** → 第 43 行 "make good Balanced Coupling decisions" 中的 "good" 是典型模糊量词——什么叫 good？没有 Claude 可以验证的标准。根本原因：native speaker 的惯用表达，没有做 NL 量词审查。自查：我的 skill 里有没有「好的」「合适的」「重要的」这类无法验证的形容词？→ **影响：Claude 有自由发挥空间，可能产生不一致的输出**

3. **「particularly dangerous」** → `review/SKILL.md` 第 62 行 "implicit coupling … is particularly dangerous"——「particularly」是比较级修饰词，但没有给出基准（比什么 particular？比多少算 dangerous？）。根本原因：同上，正常的技术写作习惯被带入 NL 工件。自查：我有没有类似「尤其是」「特别是」「非常重要」这种相对表达而非绝对标准？

### 3.4 当时的优化机会

1. `design/SKILL.md` 加 `model: claude-sonnet-5`（或当前最优 Sonnet 模型）
2. 把「good」改为「satisfying the three-dimensional balance rule」
3. 把「particularly dangerous」改为具体的风险分级描述（如「ranks as Severity-3 coupling risk」）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `model:` 缺失于 design/SKILL.md | `grep "^model:" skills/design/SKILL.md` | **持续** ❌ — 无 model: 字段 | 3+ 个月，作者对这个质量问题无感知 |
| "good" 措辞模糊 | `grep "make good" skills/design/SKILL.md` | **持续** ❌ — 第 43 行仍为原文 | 同上，小改动但未优先处理 |
| "particularly" 措辞模糊 | `grep "particularly" skills/review/SKILL.md` | **持续** ❌ — 第 62 行仍为原文 | 同上 |

### 4.2 架构演进

3 个月内文件数量没有变化，plugin.json 版本升至 `1.5.0`。**结构完全稳定**——这本身就是一个信号：作者认为功能已经完整，专注质量而非扩张。新增 `marketplace.json` 说明加入了 Claude Code 市场发布流程。

`skills/document/assets/template.html` 文件验证仍然存在，那条文档链路没有断裂。

### 4.3 新增的可学习模式

无实质性架构变化。`1.5.0` 的改动主要集中在 marketplace 相关元数据，NL 工件本身没有新增内容。

---

## 五、校准

### 5.1 我已经在做对的

1. **单职责 skill**：我的 bureau skills 每个都专注单一任务，与 vladikk 的设计思路一致
2. **`user-invocable` 区分**：我的 bureau 也有部分 skill 是内部调用型（比如 lint skill），思路一致
3. **依赖声明**：我知道 `skills:` frontmatter 可以用来声明依赖，虽然还没完全系统使用
4. **知识与执行分离**：我的 bureau `recall` skill 承担知识查询，`compile` 承担执行，已经有分层意识

### 5.2 挑战 / 验证

这次案例验证了一个我之前模糊意识到的事情：**`model:` 字段不应该是可选的，它应该是每个需要推理的 skill 的必填项**。vladikk/modularity 98 分中唯一被扣的最大一项就是 design skill 缺 model:。我的所有 skill 都缺这个字段——这是一个系统性未注意到的问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 里有没有模糊量词
grep -rn -E '\b(good|appropriate|particularly|especially|very important|crucial)\b' \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | head -20
```
命中后：把每处模糊词改为可验证标准（如「good PR」→「PR that passes all checks, has ≤400 lines changed, and includes test coverage」）。

```bash
# 检查我的 skill 有没有声明 model:
grep -rL "^model:" \
  /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null
```
命中后：对每个多步骤或需要推理的 skill，加 `model: claude-sonnet-5`（或当前最佳 Sonnet）。简单查询型 skill 可以用 `model: claude-haiku-4-5-20251001`。

```bash
# 检查有没有可以拆出 non-invocable 知识库 skill 的候选
# 找正文超过 200 行的 SKILL.md
find /tmp/my-repos/MarkQWu-bureau /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" -exec \
  wc -l {} + 2>/dev/null | sort -rn | head -10
```
命中后（超过 200 行的 skill）：考虑把「背景知识」部分提取为 non-invocable 的参考 skill，主 skill 通过 `skills:` 注入。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的所有 skill 补充 `model:` 字段
   - **为何可行**：这是 frontmatter 修改，5 行可搞定一个文件，完全机械操作
   - **第一步**：打开 `bureau/skills/compile/SKILL.md`，在 frontmatter 里加 `model: claude-sonnet-5`，然后批量处理其他 skill —— 30 分钟完成全部 7 个

2. **想法**：把 bureau 的「trust tier 判断规则」提取为 non-invocable 知识库 skill
   - **为何可行**：bureau 的 review/capture/recall 都依赖同一套 trust tier 规则，目前可能分散在各文件里
   - **第一步**：检查 `bureau/skills/review/SKILL.md` 里是否有「trust tier」定义，如果有，提取到 `skills/trust-model/SKILL.md` 并设置 `user-invocable: false`，其他 skill 通过 `skills: [trust-model]` 注入 —— 1 小时可完成

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 vladikk/modularity 的核心目的**：把「Balanced Coupling」领域框架编码进 Claude，让 AI 能辅助做模块化架构决策

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 高 | 都是 Claude Code plugin，都用 skills: frontmatter 依赖注入 | bureau 服务知识管理，modularity 服务软件设计 | 高（依赖注入模式完全可借鉴）|
| MarkQWu/gstack | 中 | 都有架构/设计类工作流 | gstack 有 ios-design-review skill，与 modularity 设计审查方向接近 | 高（gstack 的设计类 skill 可参考 vladikk 的结构）|

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 缺 `model:` 声明 | `grep -rL "^model:" skills/*/SKILL.md` | **MarkQWu-bureau 命中 7/7 个 skills**；**MarkQWu-gstack 命中 10+/10+ 个 skills** | 高 |
| 模糊量词 | `grep -n "good\|particularly\|appropriate" skills/*/SKILL.md` | 待验证（大概率有） | 中 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/skills/recall/SKILL.md` → 加 `model: claude-sonnet-5` → 1 分钟
- `MarkQWu-bureau/skills/compile/SKILL.md` → 同上 → 1 分钟
- 批量处理所有 skills: `for f in $(find /path/to/bureau/skills -name SKILL.md); do grep -q "^model:" "$f" || echo "model: claude-sonnet-5" >> "$f"; done`

### 7.3 别人的更优方案

1. **领域**：non-invocable 知识库 skill + frontmatter 依赖注入
   - **本案例做法**：`balanced-coupling/SKILL.md` 设为 `user-invocable: false`，被 `design` 和 `review` 通过 `skills: [balanced-coupling]` 注入
   - **我的项目现状**：bureau 的所有 skills 都是独立的，共享知识（如 trust tier 规则）可能分散在各文件里
   - **如何借鉴**：提取 bureau 的 `trust-model` 知识为独立 skill，在其他 skills 的 frontmatter 里声明依赖

2. **领域**：布尔表达式压缩决策逻辑
   - **本案例做法**：`MODULARITY = STRENGTH XOR DISTANCE`，两行替代两段解释
   - **我的项目现状**：gstack 或 bureau 里可能有「判断是否执行某操作」的条件逻辑，用散文描述
   - **如何借鉴**：在 `ios-design-review/SKILL.md` 里把「什么是好的设计评审」改写为几行布尔逻辑或真值表

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：发布与 marketplace.json 的同步维护
  - **我的项目现状**：MarkQWu/bureau 有明确的版本管理并同步更新 marketplace.json（版本 0.5.2）
  - **本案例做法（弱在哪）**：vladikk/modularity 直到 v1.5.0 才加入 marketplace.json，说明早期版本的市场元数据缺失
  - **意义**：我在发布规范上的实践比这个 exemplar 要早，值得继续保持

---

## 八、术语表

### <a name="balanced-coupling"></a>Balanced Coupling 模型
> Vlad Khononov 提出的软件模块化评估框架，用三个维度评估模块之间的耦合：Integration Strength（集成强度）、Distance（物理/组织距离）、Volatility（变化频率）。核心规则：好的模块化设计要求 STRENGTH XOR DISTANCE（强度和距离不能同时高）。

### <a name="dependency-injection"></a>依赖注入（Skill 层面的 DI）
> 在 Claude Code 中，SKILL.md 的 frontmatter 里可以声明 `skills: [skill-a, skill-b]`，Claude Code 在加载该 skill 时会自动先加载声明的 skill 文件。这类似程序语言里的依赖注入——被依赖的知识被「注入」到当前 skill 的上下文中，无需复制粘贴。

### <a name="user-invocable"></a>user-invocable
> SKILL.md frontmatter 里的布尔字段。`false` 表示这个 skill 不能被用户直接调用（不出现在 skill 列表里），只能被其他 skill 通过 `skills:` 声明间接加载。用于区分「对外 API」和「内部实现」。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块。在 SKILL.md 里声明 `name`、`description`、`model`、`allowed-tools`、`skills`（依赖）、`user-invocable` 等元数据。Claude Code 先解析 frontmatter 再读正文。

### <a name="r04-r05"></a>R04 / R05 规则
> NLPM 两条核心规则：R04 要求 `description` 字段写成触发地图（列举用户可能说出的 4-10 个激活短语）而非一句摘要；R05 要求 SKILL.md 正文不超过 500 行，超出时用查找表和公式代替散文解释。
