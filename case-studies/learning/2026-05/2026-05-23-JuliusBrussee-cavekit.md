# JuliusBrussee/cavekit — 学习案例

**仓库**：https://github.com/JuliusBrussee/cavekit
**Stars**：564 | **来源**：upstream audit
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-23（基于当前 HEAD）
**主题标签**：`single-purpose`, `template-design`, `vague-quantifier`, `manifest-discipline`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Cavekit（版本 v4.0.0，插件名 "ck"）是 JuliusBrussee/caveman 的精简配套工具包，专注于**规格驱动开发（spec-driven development）**：用一个 `SPEC.md` 文件管理整个项目的规格、构建、检查三个动作。核心理念来自作者的 caveman 语言系统——极致压缩，以 `backprop`（规格反向传播）机制为创新点。9 个 NL 工件，**97/100 的最高分**是本批次4个仓库中质量最高的。

### 1.2 架构剖析
- **目录结构**：
  ```
  cavekit/
  ├── commands/     # 3 个命令（/ck:spec, /ck:build, /ck:check）
  ├── skills/       # 5 个技能（spec, build, check, caveman, backprop）
  ├── plugin.json   # 插件清单
  └── FORMAT.md     # 共享格式约定
  ```
- **文件类型分布**：0 个 SKILL.md 重命名，3 个 command / 5 个 skill（含 backprop、caveman）/ 1 个 manifest / 1 个 FORMAT.md
- **编排关系**：每个 command 对应一个同名 skill，command 是入口层（用户触发），skill 是实现层（被 command 调用）。`backprop` skill 被 build command 和 build skill 交叉引用。没有 router，没有 meta skill，完全 1:1 映射。
- **跨件契约**：`FORMAT.md` 在 repo 根目录，被 `commands/spec.md`、`commands/build.md`、`skills/spec/SKILL.md` 三个文件显式引用——这是罕见的"共享格式文档"模式，把输出格式约定从各文件中提取出来统一维护。

### 1.3 设计思路 / 方法论
- **核心设计哲学**："规格是唯一真相"。所有构建输出都要与 `SPEC.md` 比对；偏差不是 bug，是规格本身的问题——`backprop` skill 的目的就是把发现的偏差写回规格。
- **解决什么问题**：开发过程中规格和代码经常产生漂移，代码实现了"某种行为"但规格还停留在过去。cavekit 强制让规格成为动态文档而非静态愿望清单。
- **做了什么 trade-off**：简单到极致（3 个命令，1 个 SPEC.md，无外部依赖）换来了普适性和零门槛，代价是没有多项目管理能力，适合单 SPEC.md 的小型项目。
- **反映什么认知模型**：作者认为 AI agent 最适合做"契约守护者"：人写规格意图，AI 验证代码是否兑现，并把未兑现的偏差用结构化方式写回——而不是无约束地生成代码。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「[契约驱动三件套](#契约驱动三件套)」（Command-Spec-Backprop 架构）**

一个 SPEC.md 作为项目契约，三个命令对应"写契约→执行契约→修复契约"的循环。Backprop 是这个模式的独特创新：当执行和契约不一致时，不是抛出错误让人去改代码，而是更新规格以反映现实或标记分歧供人类决策。

模式特征清单：
- **单一真相文件**（SPEC.md）：整个项目的规格、状态、§V（学习日志）都在一个文件
- **1:1 command-skill 映射**：每个命令精确对应一个技能，无冗余
- **FORMAT.md 共享约定**：输出格式与业务逻辑分离，format 变化不需改多个文件
- **Backprop 反向传播**：执行发现的偏差写回 SPEC.md §I（不变量）或 §V（学习日志）
- **零外部依赖**：纯 Markdown，无运行时，任何 Claude Code 环境即可使用

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 单人小型项目，有明确功能规格 | ✅ 高度适用 | 零配置，立即可用 |
| 需要频繁重构、保持规格一致性的项目 | ✅ 高度适用 | backprop 机制正是为此设计 |
| 多个并行子项目（multi-repo / monorepo） | ⚠️ 部分适用 | 每个项目可以有自己的 SPEC.md，但 cavekit 没有跨 SPEC 管理能力 |
| 团队协作、需要 issue tracker 集成 | ❌ 不适用 | SPEC.md 的并发编辑会有冲突，设计为单人 |
| 需要自动化测试驱动的项目 | ❌ 不适用 | check 是 AI 语义检查，不是可运行的单元测试 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 契约驱动三件套（本仓库） | cavekit | 零依赖，规格-代码一致性内建，backprop 创新 | 单 SPEC.md 限制，无测试自动化 |
| NL-TDD spec（像 NLPM 的 .nlpm-test） | NLPM 本体 | 可自动运行测试，结果可重复 | 需要 tester agent，配置复杂 |
| 纯命令型插件（无 skill 层） | 简单 CLAUDE.md 里的 slash command | 最简单 | 无法被其他 command 复用 |

### 2.4 改进空间
1. **当前问题**：三个 command 的 [frontmatter](#frontmatter) 缺少 `allowed-tools` 声明。**改进做法**：`build.md` 加 `allowed-tools: Read Write Edit Bash`，`check.md` 加 `allowed-tools: Read Bash`，`spec.md` 加 `allowed-tools: Read Write`。**预期收益**：用户看到权限声明时知道这个命令会做什么系统操作，提高信任度；部分 Claude Code 配置中需要显式声明工具才能使用。
2. **当前问题**：`skills/backprop/SKILL.md` 里的 "sometimes"/"usually" 让 AI 在边界情况下行为不确定。**改进做法**：将 "§V entry (usually)" 改为 "§V entry, unless the failure is a mechanical typo with no spec implication—skip §V only when the fix takes under 10s and affects no invariants"。**预期收益**：agent 在不确定时有明确判断标准，行为更一致。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **97/100**——本批次最高分。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/check.md | 93 | 缺 allowed-tools；"relevant files" 模糊 |
| commands/build.md | 95 | 缺 allowed-tools |
| commands/spec.md | 95 | 缺 allowed-tools |
| skills/backprop/SKILL.md | 96 | "sometimes"、"usually" 模糊量词 |
| skills/check/SKILL.md | 98 | "relevant files" 模糊 |
| plugin.json | 100 | 无问题 |
| skills/build/SKILL.md | 100 | 无问题 |
| skills/caveman/SKILL.md | 100 | 无问题 |
| skills/spec/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **FORMAT.md 共享格式文档** → 根本原因：输出格式是多个文件关心的横切关注点，抽出到共享文件避免同步问题。示例：`commands/spec.md`、`commands/build.md`、`skills/spec/SKILL.md` 都引用 `FORMAT.md` → 借鉴：自己项目中多个 skill 共用的输出格式，提取到 `references/format-conventions.md`
2. **Command-Skill 完美 1:1 对应** → 根本原因：command 是用户入口（触发器），skill 是可复用实现（逻辑），分离使得 skill 可以被多个 command 或其他 skill 复用。→ 借鉴：设计时先定 skill（业务逻辑），再包一层 command（用户体验）
3. **backprop 规格反向传播** → 根本原因：规格漂移是项目最常见的维护问题，把"修 bug 的同时更新规格"变成明确步骤，避免规格文档越来越假。→ 借鉴：在自己的 CLAUDE.md 或 skill 里加一条："每次修复 bug 后，同步更新 SPEC.md 或 CLAUDE.md 中对应的约束描述"
4. **plugin.json 命名简洁（name: "ck"）** → 短插件名让触发更快：`/ck:spec` 比 `/cavekit:spec` 少打字。值得在设计插件时考虑别名或短名。
5. **skills/caveman/SKILL.md 满分** → caveman 技能本身是 cavekit 和 caveman 两个仓库共享的核心，100 分说明设计极其清晰：明确的压缩规则、多级示例、auto-clarity 规则。

### 3.3 当时的缺陷
1. **问题：三个 command 都缺少 `allowed-tools` 声明** → 为什么失败：command 调用了 Bash（build）和 Read/Grep（check），但没有在 frontmatter 声明，用户安全审查时看不到工具依赖。根本原因是 `allowed-tools` 是相对新的 Claude Code 约定，作者可能写这个 command 时这个字段还不流行。→ 自查：我的所有 command 有没有声明 allowed-tools？
2. **问题："relevant files" 在 check.md 和 skills/check/SKILL.md 里同时出现** → 为什么失败："relevant" 没有给 AI 任何判断标准，执行时可能遗漏也可能过度扫描。相同的模糊措辞在两个文件里出现，说明二者是 copy-paste 而不是 DRY 引用，未来修一个会漏改另一个。→ 自查：我有没有在不同文件里写了相同的模糊描述？
3. **问题：`backprop/SKILL.md` 里 "sometimes" 和 "usually"** → 为什么失败：agent 遇到 "sometimes add §V" 时不知道阈值在哪，会在每次 run 时随机决定，导致相同场景下行为不一致。根本原因是作者用了日常英语表达习惯，没有转化为可判断的条件。→ 自查：我的 skill 里有多少 "sometimes/usually/often/typically"？

### 3.4 当时的优化机会
1. 为三个 command 加 `allowed-tools` frontmatter（每个不超过 5 分钟）
2. 在 check.md 和 skills/check/SKILL.md 里把 "relevant files" 改为 "files cited in §V invariants and §I constraints"
3. 在 backprop/SKILL.md 里把 "sometimes"/"usually" 改为带条件的精确描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 三个 command 缺 `allowed-tools` | `grep -c "allowed-tools" commands/*.md` | **仍存在**（全部返回 0） | 6 周内未修复，作者可能认为此字段非必须 |
| `check.md` + `skills/check/SKILL.md` 里 "relevant files" | `grep -n "relevant" commands/check.md skills/check/SKILL.md` | **仍存在**（两个文件各一处） | 同步性问题：两处相同模糊词未同步修复 |
| `backprop/SKILL.md` 里 "sometimes"/"usually" | `grep -n "sometimes\|usually" skills/backprop/SKILL.md` | **仍存在**（第 33、84 行） | 边界行为仍不确定 |

### 4.2 架构演进
比较 plugin.json：当时 version 是 4.0.0，现在仍然是 4.0.0——说明几周内无结构性变化，这是一个稳定、成熟的小工具。目录结构与 audit 时完全一致，没有新增文件，也没有删除。这个仓库的"静止"反而说明作者认为 cavekit 已经完整，主要精力在 caveman 仓库（stars 多 39 倍）。

### 4.3 新增的可学习模式
暂无——当前 HEAD 与 2026-04-06 audit 状态完全一致，无新增文件或模式变化。

---

## 五、校准

### 5.1 我已经在做对的
1. **echo-sleuth 的 skill 1:1 命名**：skills/memory-management、skills/git-mining 等与 commands/ 下的命令名一致，和 cavekit 的 1:1 映射原则相同
2. **drama-workshop-skills 的 FORMAT 约定**：references/output-templates-core.md 和 output-templates-aux.md 把输出格式抽出来，和 FORMAT.md 模式相同
3. **claude-for-legal 的自包含 skill 设计**：每个 skill 目录下不引用兄弟 skill 的内部文件——和 cavekit 的跨件一致性原则相同

### 5.2 挑战 / 验证
这次案例**挑战**了一个假设："97 分的仓库应该没什么值得改的了"——但实际上 3 个 -5 分的 `allowed-tools` 缺失和 2 个模糊量词都是明显可改的机械问题，就是没改。这说明高分≠作者关注 NL 质量问题，作者可能只是没有 NLPM 这类工具在工作流中自动提醒。正向启示：**自动化质量门禁比人工审查更能持续保证质量**，哪怕是高质量作者也会忽略小细节。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 command 文件有没有 allowed-tools 声明
find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/commands/*" | while read f; do
  if ! grep -q "allowed-tools" "$f"; then
    echo "MISSING allowed-tools: $f"
  fi
done
# 命中后：根据 command 实际使用的工具添加 allowed-tools frontmatter
```

```bash
# 检查我的 skill 里有多少模糊量词
grep -rn -E '\b(sometimes|usually|often|typically|generally|appropriate|relevant)\b' \
  /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null
# 命中后：对每处判断：能否改为"if X then Y else Z"的条件句？能则改，不能则加限定词
```

```bash
# 检查是否有相同段落出现在多个文件（copy-paste 而非 DRY 引用）
# 这里检查 "relevant" 这类词是否在多文件同步更新
grep -rn "relevant" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md
# 命中后：确认是否两处都需要同样修改，避免只改一处
```

### 6.2 灵感 → 实施路径

1. **想法**：为 claude-for-legal 的 commands 层（如果有）添加 `allowed-tools` 声明
   - **为何可行**：claude-for-legal 目前 SKILL.md 里只有 4/151 声明了 allowed-tools，很可能也缺少 command 层声明
   - **第一步**：`find /tmp/my-repos/MarkQWu-claude-for-legal -name "*.md" -path "*/commands/*"` 检查是否有 command 文件，若有，逐一补 allowed-tools——约 30 分钟

2. **想法**：为 drama-workshop-skills 实现一个 backprop 等价机制
   - **为何可行**：drama-workshop 的三层控制模型（地基/骨架/血肉）相当于 cavekit 的 SPEC.md §I 不变量，但没有把"发现的规格漂移写回 SKILL.md"的机制
   - **第一步**：在 `short-drama/SKILL.md` 的质量检查步骤结尾加一句："若发现新的规则违反，更新 SKILL.md 对应限制描述"——约 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 JuliusBrussee/cavekit 的核心目的**：规格驱动开发，强制 AI 与单一 SPEC.md 保持一致
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 中 | 同为 command+skill 分层；同有"规格类"约束文档（references/） | drama 是内容创作工具，cavekit 是开发工具；drama 无 backprop 机制 | 中 |
| MarkQWu/claude-for-legal | 中 | 同为 skill 集合，同有结构化输出约定 | legal 无明确的规格-实现一致性检查机制 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为插件；command 层设计类似 | 完全不同的领域和使用场景 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Command 缺 `allowed-tools` | `find /tmp/my-repos/MarkQWu-* -name "*.md" -path "*/commands/*" -exec grep -L "allowed-tools" {} \;` | echo-sleuth-for-claude：commands/ 目录下文件均未做检查（需进一步验证） | 中 |
| 模糊量词 "relevant/sometimes/usually" | `grep -rn "relevant\|sometimes\|usually" /tmp/my-repos/MarkQWu-*/*/SKILL.md` | echo-sleuth：experience-synthesis:118 "relevant sessions"；git-mining:93 "relevant git commits"（合计 2 处）| 低 |

**命中后的具体行动建议**：
- `echo-sleuth/skills/experience-synthesis/SKILL.md:118` → "relevant sessions" → 改为 "sessions within the specified date range and matching the user's search keywords"→ 5 分钟
- `echo-sleuth/skills/git-mining/SKILL.md:93` → "relevant git commits" → 改为 "commits that touch the specified files or match the keyword filter"→ 5 分钟

### 7.3 别人的更优方案

1. **领域**：FORMAT.md 共享格式约定文件
   - **本案例做法**：`FORMAT.md` 在 repo 根，被 commands 和 skills 共同引用，格式约定只有一份 → cavekit `FORMAT.md`
   - **我的项目现状**：`drama-workshop-skills` 的输出格式分散在 `references/output-templates-core.md` 和 `output-templates-aux.md` 两个文件，且每个命令在 SKILL.md 里还有一些行内格式约定，三处不同步
   - **如何借鉴**：把 drama-workshop 的行内格式约定抽出，合并到 `references/output-templates-core.md`，各命令改为引用这一个文件

2. **领域**：Plugin 简短名称（`ck` vs `drama-workshop`）
   - **本案例做法**：`plugin.json` 的 name 是 `ck`，触发命令为 `/ck:spec` — 极短
   - **我的项目现状**：drama-workshop-skills 命令较长（中文斜杠命令如 `/开始`，实际不走插件 namespace），但若有 namespace，应考虑缩写
   - **如何借鉴**：设计新插件时，将 plugin name 限制在 2-5 个字符，方便用户记忆和输入

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：自动化版本一致性检查
- **我的做法**：`drama-workshop-skills` 有 `release-gate/` 脚本和 CI 验证版本号一致性
- **本案例做法（弱在哪）**：cavekit 无 CI 验证，`plugin.json` 版本号是否与 CHANGELOG.md 同步完全依赖人工——尽管目前 4.0.0 是稳定版本，但若未来频繁迭代会出现漂移
- **意义**：这是 drama-workshop 的可见优势，如有机会参与 cavekit PR，可建议加简单的 version-consistency CI check

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置块，声明文件的元数据。Claude Code 读 command 或 SKILL.md 时先解析 frontmatter 决定如何注册和路由。

### <a name="allowed-tools"></a>allowed-tools
> Claude Code command frontmatter 里的一个字段，声明这个命令需要使用哪些工具（如 `Read`, `Bash`, `Write`）。未声明的工具不会被自动许可执行，用户也无法在安装时预先批准权限。

### <a name="契约驱动三件套"></a>契约驱动三件套
> cavekit 的架构模式：SPEC.md（契约）+ build（执行契约）+ check（验证契约）+ backprop（修复契约）。"契约"是指规格文档，"三件套"指三个命令。

### <a name="backprop"></a>backprop（规格反向传播）
> 借用机器学习的反向传播（backpropagation）概念：当代码和规格出现偏差时，不一定是代码错了，也可能是规格写错了。backprop skill 帮助判断应该改代码还是改规格，并把学到的教训写回 SPEC.md 的 §V（Violations log）。

### <a name="DRY"></a>DRY（Don't Repeat Yourself）
> 编程原则：同一段信息只在一个地方维护，其他地方引用它而不是复制它。cavekit 的 FORMAT.md 就是对输出格式这段信息应用 DRY 原则。
