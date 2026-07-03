# twostraws/SwiftUI-Agent-Skill — 学习案例

**仓库**：https://github.com/twostraws/SwiftUI-Agent-Skill
**Stars**：⭐3806 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-05（历史快照）| **生成日期**：2026-07-03（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `manifest-discipline`, `vague-quantifier`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

**一句话**：Paul Hudson（iOS 开发领域知名作者、Hacking with Swift 创始人）为 Claude Code 写的 SwiftUI 代码审查技能插件，审查范围覆盖 API、视图、数据流、导航、设计、无障碍、性能、Swift 语言本身、代码整洁度九个维度。

关键事实：
- 作者 Paul Hudson 是 iOS 开发社区最有影响力的技术作者之一，⭐3806 的星数反映了他的受众基础
- 使用方式：`claude plugin install swiftui-pro@twostraws`，安装后调用 `/swiftui-pro [focus area]`
- 技能启动时按需加载 9 个参考文件，不做全量加载（context 优化）
- 仓库内含 OpenAI Responses API 的 agent.yaml 定义，说明作者已在多平台发布

### 1.2 架构剖析

```
SwiftUI-Agent-Skill/
├── .claude-plugin/marketplace.json    # marketplace 元数据（非 plugin.json）
├── swiftui-pro/
│   ├── .claude-plugin/plugin.json    # 插件注册文件（声明 skills: ./skills/）
│   ├── SKILL.md                      # ⚠️ 根目录副本（version 1.1，未注册）
│   ├── agents/openai.yaml            # OpenAI Responses API agent 定义
│   ├── references/
│   │   ├── api.md                    # deprecated API + 替换列表
│   │   ├── views.md                  # 视图/修饰符/动画最佳实践
│   │   ├── data.md                   # 数据流（@State / @Binding / etc.）
│   │   ├── navigation.md             # NavigationStack 等导航模式
│   │   ├── design.md                 # HIG 合规设计规则
│   │   ├── accessibility.md          # Dynamic Type / VoiceOver / Reduce Motion
│   │   ├── performance.md            # 渲染性能优化
│   │   ├── swift.md                  # Swift 6.2 并发最佳实践
│   │   └── hygiene.md               # 代码整洁度规则
│   └── skills/swiftui-pro/SKILL.md  # ✅ 唯一注册的技能（version 1.0）
```

- **文件类型分布**：1 个 [SKILL.md](#SKILL.md)（canonical）+ 1 个 SKILL.md（root，未注册）+ 9 个 references + 1 个 plugin.json + 1 个 openai.yaml
- **编排关系**：平铺结构。技能收到调用后，根据「partial review」时按需加载对应 reference 文件，无路由层
- **跨件契约**：SKILL.md 用 `${CLAUDE_SKILL_DIR}/references/xxx.md` 引用参考文件；`plugin.json` 的 `"skills": "./skills/"` 决定哪个 SKILL.md 被加载

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「领域知识外置到参考文件，技能主体只保留流程」——9 个 reference 文件将 SwiftUI 各维度知识作为「可替换组件」独立维护，技能主文件只规定检查顺序和输出格式
- **解决什么问题**：SwiftUI 审查范围广（API 弃用/并发/可访问性等），把所有知识塞进一个 SKILL.md 会让文件膨胀到无法维护；分拆到 reference 还能实现「partial review 只加载相关文件」的 context 优化
- **做了什么 trade-off**：控制粒度高（每个维度独立文件）换来了维护成本——9 个文件需要协同更新，某个维度升级时容易漏改；相对地，入门门槛降低（只读 SKILL.md 就能理解流程）
- **反映什么认知模型**：作者把 AI skill 当「专家系统」——检查清单驱动，每一步 reference 文件是一本「参考手册」，AI 在每步骤时翻阅对应手册，而非把所有知识灌入提示

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「步骤化流程 + 领域知识侧车（Reference Sidecar）」

技能主文件定义固定的 N 步审查流程，每步对应一个独立的参考文件，AI 按需加载（partial review 时只加载涉及的维度）。知识更新只改 reference 文件，流程更新只改 SKILL.md。

模式特征清单：
- 特征 1：技能主文件行数极短（~50 行），不含具体审查规则
- 特征 2：每个 reference 文件是「自包含的专业手册」，可独立升级
- 特征 3：输出格式固定（按文件 → 行 → 规则 → before/after 的层次），便于消费方解析
- 特征 4：「partial review」模式通过加载子集降低 context 消耗
- 特征 5：单技能设计——不做 router，不分 subagent，AI 就是审查者本身

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 针对单一技术栈的代码审查工具 | ✅ 高度适用 | 知识维度可预先穷举，流程固定 |
| 每次调用只需部分知识的场景 | ✅ 适用 | partial review 直接跳过不相关 reference |
| 需要跨技术栈泛化的通用工具 | ❌ 不适用 | 9 个 reference 文件需为每个栈单独维护 |
| 知识更新频繁（每周变化）的领域 | ⚠️ 谨慎 | 9 个文件的同步更新成本高 |
| 需要跨多个工件编排（multi-agent）的复杂任务 | ❌ 不适用 | 单技能无法编排子任务 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 步骤化流程 + Reference Sidecar（本仓库） | swiftui-pro | context 可控，文件职责单一，entry 简洁 | 维护多文件同步负担 |
| 知识全内联 | 多数简单 SKILL.md | 无需维护多文件 | SKILL.md 臃肿，context 全量加载 |
| 路由层 + 子技能分发 | autoresearch（14 命令） | 功能扩展性强 | 复杂度高，新用户学习曲线陡 |
| 外部数据库 + 技能调用 | 知识图谱型 skill | 知识量无上限 | 需要外部服务，部署复杂 |

### 2.4 改进空间

1. **当前问题**：根目录 `SKILL.md`（version 1.1）与 canonical（version 1.0）并存，版本错位。**改进做法**：删除根目录副本或将 1.1 的变更反向合并到 canonical。**预期收益**：消除贡献者困惑，确保 plugin.json 加载的始终是最新版本。

2. **当前问题**：所有 8 个模糊量词（"correctly", "relevant" ×2, "Comprehensively", "optimally", "consistent", "brief", "modern"）依然存在于 canonical SKILL.md 第 3/11/16/17/25/31/35/42/43 行。**改进做法**：按 audit 建议的具体表达替换（如 "iOS 26+ API usage" 替换 "modern API usage"）。**预期收益**：AI 执行时不需猜"relevant"/"brief"的边界，审查结果更一致。

3. **当前问题**：9 个 reference 文件没有版本号/更新日期。**改进做法**：在每个 reference 文件 frontmatter 中加 `updated: "2026-xx-xx"`。**预期收益**：使用者可判断参考资料时效性，尤其重要——SwiftUI API 更新很快。

---

## 三、过去审查发现（2026-05-05 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-05 当时得分 **89/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `swiftui-pro/skills/swiftui-pro/SKILL.md`（canonical） | 84/100 | 8 个模糊量词（-16）|
| `swiftui-pro/SKILL.md`（root，未注册） | 84/100 | 相同 8 个模糊量词（-16）|
| `swiftui-pro/.claude-plugin/plugin.json` | 100/100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **Reference Sidecar 分层**：9 个参考文件按审查维度独立存放，每步骤 SKILL.md 只引用需要的文件，context 消耗可控。原文见 `swiftui-pro/skills/swiftui-pro/SKILL.md` 步骤 1-9。

2. **Output Format 模板化**：输出格式写得非常具体——「按文件 → 行号 → 规则名 → before/after」，带有完整示例（ContentView.swift 那段）。这是学习"怎么写 Output Format"的教科书级案例。

3. **Argument Hint 正确使用**：`argument-hint: "[focus area]"` 让用户知道可以指定聚焦维度，配合「partial review 只加载相关 reference」，形成闭环。

4. **plugin.json 简洁完整**：只声明必要字段，version semver 合规，`"skills": "./skills/"` 路径清晰。

5. **「有话则报，无话则止」原则**：`Skip files with no issues.` 这句话避免了大量无意义输出，是大模型审查 skill 的重要设计细节。

### 3.3 当时的缺陷

1. **模糊量词密度过高**（8 个）：`correctly`、`relevantly`、`comprehensively` 等词让 AI 无法知道「多少算多」、「哪些算 relevant」。根本原因：作者用写人类文档的习惯写 AI 指令，忘了 AI 不会「理解上下文」自己补充边界。自查：我的 gstack 技能中，`design-review/SKILL.md` 有 27 处命中，`cso/SKILL.md` 有 26 处——我确实犯了同样的错误，而且比这严重 3 倍以上。

2. **根目录 SKILL.md 未注册却存在**：`plugin.json` 声明 `"skills": "./skills/"`，因此根目录的 `swiftui-pro/SKILL.md` 被静默忽略。根本原因：开发过程中文件移动了，旧文件没清理。**危险在于**：贡献者可能修改根目录文件以为在改「主文件」，实际改的是一个从未被加载的副本——这是静默失效。自查：我的 gstack 各 SKILL.md 直接放在技能目录下，由 plugin.json 统一声明，没有这个问题。

3. **版本号错位**：根目录 1.1、canonical 1.0。根本原因：缺乏 lint 或 CI 校验两个文件的版本一致性。类似 monorepo 里多个 package.json 版本不同步。自查：我的 gstack 没有版本号字段，问题不存在，但如果加了要注意。

### 3.4 当时的优化机会

1. **删除根目录 SKILL.md** 或将 1.1 变更反向同步到 canonical，消除双头蛇。
2. **替换 8 个模糊量词**：按审查规则给出具体操作定义（"iOS 26+ API usage" / "≤5-line before/after" 等）。
3. **在 references/ 文件中加版本日期**：SwiftUI API 变化快，使用者需要判断参考资料是否过时。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 8 个模糊量词 | `grep -n -E '\b(correctly\|relevant\|comprehensively\|optimally\|consistent\|brief\|modern)\b' canonical SKILL.md` | **仍存在**（同样 8 处，行号未变） | 作者未处理，这类「软质量问题」通常没有紧迫感 |
| 根目录 SKILL.md（version 1.1）与 canonical（version 1.0）并存 | 直接检查 `swiftui-pro/SKILL.md` 是否存在 | **仍存在**，版本差异不变 | 仓库快照约 2 个月未变动——可能是稳定期，也可能是维护减少 |
| version skew | `grep version` in both files | **仍存在**（root 1.1 / canonical 1.0） | 两缺陷相互关联，一起修或一起留 |

### 4.2 架构演进

与 2026-05-05 audit 时对比，**目录结构几乎无变化**。当前 HEAD 与快照一致：
- `swiftui-pro/` 子目录结构不变
- 新增了 `CODE_OF_CONDUCT.md`（行为准则）
- 9 个 reference 文件都在，无删改信号

这说明：仓库处于「稳定发布态」，内容定型，作者没有在活跃迭代功能。对学习者的意义是：这是一个「完成品快照」，适合作为参考架构，但不是活跃演进中的设计。

### 4.3 新增的可学习模式

无新增。当前 HEAD 与 audit 快照基本一致。

---

## 五、校准

### 5.1 我已经在做对的

1. **allowed-tools 声明**：我的 gstack 所有 SKILL.md 都有 `allowed-tools` 字段（如 `setup-deploy/SKILL.md:10`），swiftui-pro 没有（技能默认继承全部工具）——这点我做得更规范。

2. **单一注册路径**：gstack 的每个技能目录就是一个 SKILL.md，由 plugin.json 统一声明，没有多个副本并存的风险。

3. **References 概念**：我的 graphify 也使用了 `references/` 子目录（`graphify/skills/kilo/references/`），说明我已经在实践「知识外置」模式。

### 5.2 挑战 / 验证

这次案例强化了一个认知：**「软质量问题」（模糊量词）在高星仓库里一样普遍且长期存在**。Paul Hudson 这样的大神写的 skill 也有 8 个量词问题，且 2 个月后还在。

挑战了的假设：原来以为「知名作者」写的 NL artifact 质量会很高，实际上对模糊量词的敏感度并不比普通开发者高——这类问题需要专门的 linter 才能系统解决，不能靠「写得好」自然规避。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查 gstack 所有 SKILL.md 中的模糊量词
grep -rn -E '\b(correctly|relevant|comprehensively|optimally|consistent|brief|modern|appropriate|efficient|robust|ensure)\b' \
  /tmp/my-repos/MarkQWu-gstack/*/SKILL.md 2>/dev/null | wc -l
# 命中后：按文件逐一替换，参考 twostraws 的改法：
# "correctly" → 加具体结构规则；"relevant" → 明确指名是哪些文件/场景；"brief" → 给行数上限

# 检查是否有重复 SKILL.md（根目录 vs canonical 双头）
find ~/.claude/skills /tmp/my-repos/MarkQWu-gstack -name "SKILL.md" | \
  xargs -I{} dirname {} | sort | uniq -d
# 命中后：确认哪个被 plugin.json 注册，删除未注册的副本
```

### 6.2 灵感 → 实施路径

1. **想法**：为 gstack 中最重的 skill（`design-review/SKILL.md`，27 处模糊命中）做一次"去模糊化"重写
   - **为何可行**：该文件是 Garry Tan 的设计审查 skill，使用频率高，修复带来的一致性收益最大
   - **第一步**：打开文件，把每个 `appropriate` 替换成具体标准（如 "for target iOS 16+ users"），预计 20-30 分钟

2. **想法**：为 gstack 引入 Reference Sidecar 模式
   - **为何可行**：gstack 的 `design-review` 和 `ios-design-review` 有大量 HIG 规则，适合抽取到 `references/hig.md`
   - **第一步**：新建 `design-review/references/hig.md`，把 HIG 相关规则移入，主文件改为 `${CLAUDE_SKILL_DIR}/references/hig.md` 引用

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例核心目的**：单一技术栈（SwiftUI）的 AI 辅助代码审查，领域知识分拆到参考文件

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 同为 Claude Code skill 集合，都有多个审查/分析类技能 | gstack 是多技术栈泛用集合，swiftui-pro 是单技术栈深度专精 | 高 |
| MarkQWu/bureau | 中 | 都有技能文件，都用 plugin.json 组织 | bureau 专注于知识捕获与查询，不是审查工具 | 中 |
| MarkQWu/graphify | 低 | 都有 references/ 子目录 | 定位是代码理解，不是审查 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词过多 | `grep -rn -E '\b(appropriate\|correct\|relevant\|comprehensive\|modern)\b' MarkQWu-gstack/*/SKILL.md` | **gstack 全员命中**：49/52 个 SKILL.md 有命中，最严重的 design-review (27处)、cso (26处)、office-hours (22处) | 高 |
| 缺少 `## Output Format` 节 | `find MarkQWu-gstack -name "SKILL.md" \| xargs grep -L "## Output Format"` | **gstack 52/52 个 SKILL.md 均无 Output Format 节** | 高 |
| 根目录 SKILL.md 与 canonical 并存 | 手动检查 | gstack 各技能目录下只有一个 SKILL.md，无此问题 | 无 |

**命中后的具体行动建议**：
- `MarkQWu-gstack/design-review/SKILL.md` → 27 处命中，优先修复，把每个 `appropriate` 改为具体约束（5-30 分钟）
- `MarkQWu-gstack/*/SKILL.md`（全部）→ 统一追加 `## Output Format` 节，参考 swiftui-pro 的「文件 → 行 → 规则 → before/after」格式（可批量脚本辅助）

### 8.3 别人的更优方案

1. **领域**：Output Format 模板化
   - **本案例做法**：`swiftui-pro/skills/swiftui-pro/SKILL.md` 第 40-52 行：先说结构（文件 → 行 → 规则名 → before/after），再给完整示例（ContentView.swift 代码块）
   - **我的项目现状**：gstack 所有 52 个 SKILL.md 均无 `## Output Format` 节，AI 的输出格式由大模型自行决定，每次不同
   - **如何借鉴**：为最常用的 10 个 gstack skill（design-review、qa、review、spec 等）添加 Output Format 节；格式参考 swiftui-pro，先列结构约束，再给 5-10 行示例

2. **领域**：Reference Sidecar（知识侧车文件）
   - **本案例做法**：9 个独立 reference 文件，技能按需加载，避免 context 全量膨胀
   - **我的项目现状**：gstack 的 `ios-design-review/SKILL.md` 把所有 HIG 规则全部内联（文件很长），context 负担重
   - **如何借鉴**：新建 `ios-design-review/references/hig.md`，抽取 HIG 部分；主 SKILL.md 改用 `${CLAUDE_SKILL_DIR}/references/hig.md` 引用

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 声明
- **我的做法**：gstack 所有 SKILL.md（`setup-deploy/SKILL.md:10`、`ios-design-review/SKILL.md:6` 等）均有显式 `allowed-tools` 声明
- **本案例做法**：swiftui-pro SKILL.md 无 `allowed-tools` 字段，意味着技能加载时默认继承全部工具权限
- **意义**：工具权限的显式收窄是安全最小权限原则的体现；若 swiftui-pro 在某个限制工具环境（企业 Claude Code 部署）下使用，可能因工具不可用而静默失败

---

## 八、术语表

### <a name="SKILL.md"></a>SKILL.md
> Claude Code plugin 体系中「技能定义文件」的标准文件名。里面用 YAML [frontmatter](#frontmatter) 声明技能的 name/description/version 等元数据，正文是 Markdown 格式的使用说明和执行流程。Claude 加载 plugin 时会按 `plugin.json` 指定的路径找到这个文件并解析。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`version`、`allowed-tools` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter，才能知道这个 skill 如何注册和调用。

### <a name="manifest"></a>manifest（plugin.json）
> 插件的「清单文件」，列出这个插件包含哪些组件（skills、commands、agents 的路径）。`plugin.json` 里声明 `"skills": "./skills/"` 意思是：到 `./skills/` 子目录下找所有 SKILL.md 来加载；根目录的 SKILL.md 不在这个路径里，因此被静默忽略。

### <a name="context"></a>context（上下文窗口）
> 大语言模型每次处理请求时「能看到的内容总量」是有上限的（如 200k tokens）。所有加载的 skill 文件、reference 文件、对话历史都消耗这个上限。Reference Sidecar 模式中「按需加载」的意义：不用到的 reference 文件不加载，节省 context 给真正需要的内容。

### <a name="partial-review"></a>partial review
> 只审查 SwiftUI 代码的某些维度，而不是全部 9 个。例如「只检查数据流」时，SKILL.md 里的 `If doing a partial review, load only the relevant reference files.` 指令让 AI 只加载 `data.md`，跳过其他 8 个文件，大幅减少 context 消耗。
