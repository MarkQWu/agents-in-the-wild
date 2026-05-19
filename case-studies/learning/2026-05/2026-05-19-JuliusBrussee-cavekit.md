# JuliusBrussee/cavekit — 学习案例

**仓库**：https://github.com/JuliusBrussee/cavekit
**Stars**：564 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-19（基于当前 HEAD）
**主题标签**：`template-design`, `single-purpose`, `vague-quantifier`, `manifest-discipline`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

JuliusBrussee/cavekit 是 caveman 插件的配套工具包，提供一套基于 SPEC.md 的 TDD 工作流（spec → build → check），并集成了 caveman 压缩风格写作。564★ 在精品小体量插件中属于较高水准。作者 JuliusBrussee 同时维护 caveman（22505★），两个仓库共享设计哲学：极简、单职责、FORMAT.md 驱动的规范化输出。

### 1.2 架构剖析

```
cavekit/
├── .claude-plugin/
│   ├── plugin.json          # 满分 100/100 的清洁 manifest
│   └── marketplace.json
├── FORMAT.md                # 共享规范文档，被 3 个 artifact 引用
├── CHANGELOG.md
├── LAUNCH-POST.md
├── SECURITY.md
├── UPGRADE.md
├── commands/
│   ├── build.md             # /build — 实现 SPEC.md 中的任务
│   ├── check.md             # /check — 漂移检测，只读不写
│   └── spec.md              # /spec — SPEC.md 的唯一变更者
└── skills/
    ├── backprop/SKILL.md    # 把 bug 反向传播回 §I/§V
    ├── build/SKILL.md       # 100/100
    ├── caveman/SKILL.md     # 100/100
    ├── check/SKILL.md       # 漂移检测技能
    └── spec/SKILL.md        # 100/100
```

- **文件类型分布**：3 个 command + 5 个 SKILL.md + 1 个 FORMAT.md 共享规范 + 满分 manifest
- **编排关系**：command（用户调用）→ skill（实现逻辑）一对一映射；`/build` 还调用 backprop skill 处理 bug 反向传播
- **跨件契约**：FORMAT.md 是多方共享的格式规范，被 `commands/spec.md`、`commands/build.md`、`skills/spec/SKILL.md` 三处引用；每处都在 LOAD 步骤第一条显式加载 FORMAT.md

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「SPEC.md 作为唯一真实源」——spec 是规范，build 是实现，check 是验证，三者职责清晰不重叠。SPEC.md 是系统的不变量，check 只读验证，build 不改 spec，spec 是 spec 的唯一变更者
- **解决什么问题**：解决代码和规范漂移问题——开发过程中规范与实现逐渐脱节，check 提供一个可随时执行的漂移检测器
- **Trade-off**：工作流有明确的调用顺序约束（先 /spec，再 /build，再 /check），牺牲了灵活性换取了状态一致性；不适合没有 SPEC.md 的项目
- **认知模型**：作者把 AI agent 视为可以严格遵守预定义协议的状态机——FORMAT.md 定义了输出契约，skill 只需要按契约执行，不需要「理解意图」

---

## 二、过去审查发现（2026-04-06 历史快照）

### 2.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **97/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/check.md | 93 | 缺 allowed-tools；"relevant" 模糊量词 |
| commands/build.md | 95 | 缺 allowed-tools |
| commands/spec.md | 95 | 缺 allowed-tools |
| skills/backprop/SKILL.md | 96 | "sometimes"、"usually" 模糊量词 |
| skills/check/SKILL.md | 98 | "relevant" 模糊量词 |
| .claude-plugin/plugin.json | 100 | 无问题 |
| skills/build/SKILL.md | 100 | 无问题 |
| skills/caveman/SKILL.md | 100 | 无问题 |
| skills/spec/SKILL.md | 100 | 无问题 |

### 2.2 当时值得借鉴的模式

1. **FORMAT.md 作为多 artifact 共享规范** → 三个 command 和一个 skill 都显式加载同一个 FORMAT.md，确保输出格式一致性。根本原因：把「格式约定」从各文件中抽出来，集中管理，修改一处即全局生效。示例：`commands/build.md` LOAD 步骤第 2 行。借鉴方式：自己的多技能套件可以抽取一个共享的 FORMAT.md

2. **命令 ↔ 技能一对一映射** → build/check/spec 各有对应的 SKILL.md，command 是触发器，skill 是逻辑实现，清晰分层。借鉴方式：设计命令时先写 skill，命令只做调度

3. **plugin.json 满分 100/100** → manifest 字段完整，无遗漏，说明作者对 frontmatter 规范理解深入。借鉴方式：新建 manifest 时对照 NLPM 规范逐字段检查

4. **/check 只读不写的单职责约束** → check 命令在开头就声明「Pure diagnostic. Reports violations. Writes nothing.」，把副作用边界刻在文档中，避免审查工具意外改写文件。借鉴方式：对任何只读工具在描述中明确「不写文件」

5. **backprop 技能的反向传播模式** → build 执行中发现 bug 时，不是默默修复，而是把 bug 的根因反向传播到 §I（接口规范）或 §V（不变量），更新规范本身。这是「规范即真实源」哲学的最后一块砖。借鉴方式：建立一个「缺陷反馈到规范」的维护流程

### 2.3 当时的缺陷

1. **三个 command 都缺 allowed-tools 声明**：根本原因是 Claude Code 早期命令文件对 allowed-tools 的规范意识不足；实际上 build 用了 Bash 和 Write/Edit，check 用了 Read 和 Grep，但均未声明。这让工具依赖对用户不透明，在受限环境下会静默失败。自查：我的 command frontmatter 是否声明了所有实际使用的工具？

2. **"sometimes"/"usually"/"relevant" 等模糊量词**：根本原因是这些词在日常英语中是完全合理的，但对于需要确定性行为的 AI 代理来说，「有时」和「通常」无法转化为明确的执行决策。自查：我的 SKILL.md 中是否有类似「可能」「一般来说」「有时候」这类词？

3. **empty-argument 处理依赖隐式默认**：check 命令的无参数情况（默认执行 §V）是隐式约定而非显式文档。根本原因是作者认为「默认行为显而易见」，但对于不熟悉系统的用户，隐式默认是一个认知负担。自查：我的命令是否清楚描述了无参数时的行为？

### 2.4 当时的优化机会

1. **在所有 command frontmatter 中添加 `allowed-tools`**：`build.md` 加 `[Bash, Write, Edit]`，`check.md` 和 `spec.md` 加 `[Read, Grep]`
2. **替换 "relevant files" 为具体描述**：例如改为「§V 不变量中引用的文件」
3. **把 "sometimes"/"usually" 替换为条件分支**：`"if §I shape mismatch"` 代替 `"sometimes"`

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 三个 command 缺 allowed-tools | `grep "allowed-tools" commands/*.md` | **仍缺失** — grep 无输出，三个命令 frontmatter 均无该字段 | 6 周内未修复；作者可能未收到 PR，或认为优先级低 |
| backprop "sometimes"/"usually" | `grep -n "sometimes\|usually" skills/backprop/SKILL.md` | **仍缺失** — 第 33、84 行仍保留原词 | 模糊量词在小体量高分仓库中也是低优先级问题 |
| check "relevant files" | `grep -n "relevant" skills/check/SKILL.md commands/check.md` | **仍缺失** — 两处均保留 "relevant files" | 跨件重复的同一措辞，修复需要同时改两个文件 |

### 3.2 架构演进

当前 HEAD 相比 audit 时（2026-04-06）新增了 `LAUNCH-POST.md`、`SECURITY.md`、`UPGRADE.md` 三个文档，说明仓库进入正式发布后阶段，开始补充发布生命周期文档。核心技能和命令结构无变化——plugin.json、FORMAT.md、5 个 SKILL.md 的内容均稳定。这说明作者对核心设计已经满意，变化集中在外围文档。

### 3.3 新增的可学习模式

**SECURITY.md 的引入**：仓库在 audit 后新增了专门的安全文档，尽管这个仓库没有高风险的执行表面（无 hooks、无脚本）。这是一个「先建规范，再扩展」的前瞻性实践——有了 SECURITY.md，后续添加 hooks 时有了安全分析的文档框架。借鉴意义：即使当前无安全风险，提前建立 SECURITY.md 模板，为未来扩展做准备。

---

## 四、校准

### 4.1 我已经在做对的

1. **命令和技能分层设计**：我的命令只做调度，逻辑放在 skill，与 cavekit 的实践一致
2. **manifest 字段完整性检查**：我在创建插件时会检查 plugin.json 是否缺字段
3. **只读工具声明不写操作**：我的审查类工具描述中包含「不修改文件」的约束
4. **FORMAT.md 抽取公共格式规范**：我已在多技能套件中使用共享格式文档
5. **无参数行为显式文档**：我的命令在 argument-hint 中注明默认行为

### 4.2 挑战 / 验证

**验证**：cavekit 97/100 的得分验证了「命令 ↔ 技能一对一映射 + FORMAT.md 共享规范」的架构能接近满分。这个组合在实践中可以达到极高的质量水准，证明我的方向是对的。

**挑战**：我之前认为「allowed-tools 声明是可选的」——cavekit 说明即使是 97 分的仓库也会被扣分于此，而且 6 周内作者都没有主动修复，可见这个字段对作者来说不影响功能运行。但对用户透明度和受限环境兼容性有实质影响，值得认真补充。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查 command 文件是否有 allowed-tools 声明
find ~/.claude/commands -name "*.md" -exec grep -l "allowed-tools" {} \;
# 命中后：对照 LOAD 步骤中实际使用的工具，确认 allowed-tools 列表完整

# 检查 skill 中是否有模糊量词
grep -rn -E '\b(sometimes|usually|often|relevant|appropriate|reasonable)\b' ~/.claude/skills/*/SKILL.md
# 命中后：把每个命中替换为条件分支（if X then Y）或具体描述

# 检查 command 是否声明了无参数时的默认行为
grep -rn "argument-hint\|default" ~/.claude/commands/*.md | head -20
# 命中后：对无 argument-hint 的 command 补充默认行为说明
```

### 5.2 灵感 → 实施路径

1. **想法**：为多技能套件引入 FORMAT.md 共享规范
   - **为何可行**：cavekit 的 FORMAT.md 被 3 个 artifact 引用，修改一处全局生效，减少格式漂移
   - **第一步**：从现有最规范的一个技能中提取输出格式部分，写成单独的 FORMAT.md，在其他技能的 LOAD 步骤加引用

2. **想法**：建立「规范反向传播」流程（backprop 模式）
   - **为何可行**：代码修复了但规范没更新，是长期维护的最大隐患；backprop 强制把 bug 的根因记入规范
   - **第一步**：在 CLAUDE.md 中添加一条规则：「发现技能执行失误时，先更新 SKILL.md 中对应步骤的描述，再修复行为」

3. **想法**：补全所有命令的 allowed-tools 字段
   - **为何可行**：这是 3 分的扣分项，且是影响受限环境兼容性的实质问题，修复成本极低（每个命令加一行）
   - **第一步**：遍历所有 command 文件，对照 LOAD/执行步骤中实际调用的工具名称，在 frontmatter 中补充 allowed-tools 列表
