# twostraws/SwiftUI-Agent-Skill — 学习案例

**仓库**：https://github.com/twostraws/SwiftUI-Agent-Skill
**Stars**：3806 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-05（历史快照）| **生成日期**：2026-07-04（基于当前 HEAD）
**主题标签**：`examples-driven`, `vague-quantifier`, `manifest-discipline`, `template-design`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Paul Hudson（Hacking with Swift 作者）为 Claude Code 写的 SwiftUI 专业代码审查 skill。安装后，让 Claude 对 SwiftUI 代码做「9 条规则集」级别的深度 review，包含 API 兼容性、视图写法、数据流、导航、无障碍、性能、Swift 语法、代码卫生共 9 个维度。

关键事实：
- Paul Hudson 是 iOS 开发界最知名的教育者之一（Swift on Sundays、Hacking with Swift），本 skill 体现了其「以参考文件代替规则记忆」的教学风格
- 通过 Claude plugin marketplace 分发，`plugin.json` 声明 `"skills": "./skills/"` 路径
- 仓库内有 9 个专门的 `references/` 参考文件（api.md、views.md、data.md 等），是 skill 的核心知识库
- 存在根目录 `SKILL.md`（版本 1.1）和注册用 `skills/swiftui-pro/SKILL.md`（版本 1.0）两份文件

### 1.2 架构剖析
```
SwiftUI-Agent-Skill/
└── swiftui-pro/
    ├── .claude-plugin/
    │   └── plugin.json          ← 注册入口，声明 skills: ./skills/
    ├── skills/
    │   └── swiftui-pro/
    │       └── SKILL.md         ← 注册用规范文件（version 1.0）
    ├── references/              ← 9 个领域知识文件
    │   ├── api.md               ← deprecated API 映射
    │   ├── views.md             ├── data.md
    │   ├── navigation.md        ├── design.md
    │   ├── accessibility.md     ├── performance.md
    │   ├── swift.md             └── hygiene.md
    └── SKILL.md                 ← 根目录冗余副本（version 1.1，未注册）
```

- **文件类型分布**：1 个 Skill（注册）+ 1 个冗余 Skill（根目录）+ 9 个 reference 文件 + 1 个 plugin.json
- **编排关系**：单 skill，在 skill body 中以 `${CLAUDE_SKILL_DIR}/references/xxx.md` 路径加载 9 个 reference 文件，每步 review 对应一个 reference
- **跨件契约**：每条审查步骤明确指向对应 reference（如 `1. Check deprecated API using .../references/api.md`），形成步骤→reference 的一一映射

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「把领域知识外置」——skill body 只写 9 步审查流程，具体判断标准全在 reference 文件里。好处是 reference 可以独立更新，不需要动 skill 本身
- **解决什么问题**：SwiftUI API 变化极快（iOS 15→16→17→18→26），规则不能硬写在 prompt 里，否则每次 Apple 发布新版本都要改 skill。外置 reference 让更新只需改 reference 文件
- **做了什么 trade-off**：深度（9 个 reference 文件全量加载）vs. 上下文消耗。作者的取舍是「部分 review 时只加载相关 reference」——`If doing a partial review, load only the relevant reference files`
- **反映什么认知模型**：作者把 skill 理解为「流程编排器」，把知识理解为「可加载文件」，两者分离。这与 Paul Hudson 教写书的思路一致：流程固定，内容持续更新

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「参考文件驱动型 Skill（Reference-File-Driven）」**

SKILL.md 只定义流程框架，全部判断标准外置到独立的 `references/` 文件中，由 skill 在执行时按需加载。

模式特征清单：
- 特征 1：SKILL.md body 极短（～100 行），所有「什么算对/错」在 reference 里
- 特征 2：每个审查维度对应一个 reference 文件（1:1 映射）
- 特征 3：reference 文件可独立更新，不需要改 skill 版本号
- 特征 4：`${CLAUDE_SKILL_DIR}/references/` 路径前缀确保 reference 从 skill 目录解析，不依赖项目结构
- 特征 5：`partial review` 降级路径：`load only the relevant reference files`（内容 × 上下文消耗的动态平衡）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域知识变化快（如 iOS API、法规、框架版本） | ✅ 高度适用 | reference 独立更新，skill body 不动 |
| 多维度审查（代码 review、安全检查） | ✅ 适用 | 每个维度对应一个 reference，彼此不干扰 |
| 简单单步任务（格式化、命名规范） | ❌ 过度设计 | 用不到外置 reference，直接写在 skill body 更简洁 |
| 需要跨 skill 共享知识库 | ⚠️ 需变体 | 原生 reference 路径是相对于当前 skill，不适合跨 skill 共享 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 参考文件驱动（本仓库） | twostraws/SwiftUI-Agent-Skill | 知识与流程分离，维护成本低 | reference 文件越多，首次加载上下文越大 |
| 规则内联型 | 多数简单 skill | 自包含，无外部依赖 | 知识更新需改 skill，版本管理复杂 |
| 外部 MCP 型 | upstash/context7 | 可查询海量实时知识 | 依赖网络 + MCP 服务，离线失效 |

### 2.4 改进空间
1. **当前问题**：根目录 `SKILL.md`（v1.1）与注册用 `skills/swiftui-pro/SKILL.md`（v1.0）版本不同步，新开发者容易改错文件。**改进做法**：删除根目录冗余副本，或在根目录放一个 `README.md` 说明「编辑 skills/ 下的规范文件」。**预期收益**：消除版本分叉风险。
2. **当前问题**：8 个模糊量词仍在规范文件中（`correctly`、`relevant`、`modern` 等），导致 Claude 无法确定判断边界。**改进做法**：参照 audit 建议，将 `modern API usage` → `iOS 26+ / Swift 6.2+ API usage`，`configured correctly` → `follows single-source-of-truth data flow`。**预期收益**：减少主观判断，提升 review 一致性。
3. **当前问题**：`If doing a partial review, load only the relevant reference files` 没有给出「哪些场景只加载哪些 reference」的具体指导。**改进做法**：在 Output Format 下方加一个表格，列出常见任务（新功能、bug fix、性能优化）对应加载哪几个 reference。

---

## 三、过去审查发现（2026-05-05 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-05 当时得分 **89/100**（3 个文件平均：两个 SKILL.md 各 84 分，plugin.json 100 分）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `swiftui-pro/skills/swiftui-pro/SKILL.md` | 84/100 | 8 个模糊量词（-16） |
| `swiftui-pro/SKILL.md`（根目录冗余） | 84/100 | 同上（内容几乎相同） |
| `swiftui-pro/.claude-plugin/plugin.json` | 100/100 | 无问题 |

### 3.2 当时值得借鉴的模式
1. **参考文件驱动** → 知识与流程分离，维护开销低 → `skills/swiftui-pro/SKILL.md` 每步骤直接引用 reference 文件 → 适合知识密集型 skill
2. **Output Format 模板示例** → 给出完整示例输出（文件名→问题→before/after 代码），消除歧义 → `swiftui-pro/SKILL.md` 第 41 行起的 `Example output` 块 → 我的 skill 可以加类似的 example block
3. **argument-hint 声明** → `argument-hint: "[focus area]"` 让用户在调用时知道可以指定聚焦范围 → `plugin.json` 注册配置 → 有参数的 skill 都应该声明 argument-hint
4. **明确 negative instruction** → `Report only genuine problems - do not nitpick or invent issues.` 一句话砍掉 false positive → SKILL.md 第 11 行 → 避免 Claude 刷存在感

### 3.3 当时的缺陷
1. **模糊量词**：`correctly`、`relevant`、`Comprehensively`、`optimally`、`modern` 等 8 个词语出现在 skill 正文中 → 根本原因：这些词对人类读者直觉清晰，但对 AI 来说没有操作边界，导致每次执行的 review 标准不一致 → 自查：我的 skill 文件中同样高频出现这类词（gstack 414 次命中）
2. **根目录冗余文件**：`swiftui-pro/SKILL.md` 存在但未被 `plugin.json` 注册，版本号（1.1）比注册版本（1.0）更新 → 根本原因：开发者在根目录修改了 skill 但忘记同步到 `skills/` 路径，plugin 系统只加载 `skills/` → 自查：如果我有多个 skill 副本，是否都指向同一规范文件？
3. **无 model 声明**（skills 不需要，这里是教育性说明）：skills 不需要 model 字段，但有些作者误以为需要而空填或不填，影响认知一致性

### 3.4 当时的优化机会
1. 把 8 个模糊量词替换为可测量表达（如 `correctly` → `follows single-source-of-truth data flow`）
2. 删除根目录冗余 SKILL.md，或将其变为指向 `skills/` 的说明文件
3. 在 `partial review` 降级路径中加具体场景→reference 对照表

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 8 个模糊量词 | `grep -n "correctly\|relevant\|modern\|Comprehensively\|optimally\|consistent\|brief" skills/swiftui-pro/SKILL.md` | **仍存在**（命中 9 行） | 作者未接受 NL 质量修复建议 |
| 根目录 SKILL.md 未注册 | `cat swiftui-pro/SKILL.md \| grep version` → 1.1；`cat skills/.../SKILL.md \| grep version` → 1.0 | **仍存在**，版本差距未消除 | 维护分叉风险持续 |
| reference 文件 `${CLAUDE_SKILL_DIR}` 前缀 | `grep "CLAUDE_SKILL_DIR" SKILL.md` | ✅ 已正确使用 | 路径解析正确 |

### 4.2 架构演进
与 audit 时相比，代码库基本未变化。目录结构、9 个 reference 文件、plugin.json 路径配置均一致。作者没有做结构性重组。这说明：对于已经运作良好的单 skill 仓库，维护者倾向于保持现状，NL 质量细节不是优先修复项。

### 4.3 新增的可学习模式
无新模式——仓库自 audit 以来几乎未变动。

---

## 五、校准

### 5.1 我已经在做对的
1. **参考文件外置**：gstack 的多个 skill（如 `ios-design-review`、`document-release`）已将部分领域知识放入 sections/ 子目录，与本仓库思路一致
2. **Output Format 有 example**：gstack 的多个 skill 包含具体示例输出，与 twostraws 的 `Example output` block 同样的做法
3. **Negative instruction**：gstack 的 `plan-devex-review` 有类似「不要虚报问题」的约束语
4. **argument-hint**：gstack 的部分 skill 已声明 argument-hint
5. **`${CLAUDE_SKILL_DIR}` 路径**：gstack skill 中正确使用了此变量

### 5.2 挑战 / 验证
**挑战**：我一直觉得 `modern` 这样的词在中文语境里更「自然」，不太当回事。但 twostraws 的案例提醒：即便是英文母语、专业技术作者写的 skill，同样会在这种词上失分，而且 **18 个月后仍未修复**。说明这不是翻译问题，而是人类写 AI 指令时的普遍认知盲区——我们对这类词有直觉，AI 没有。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 中的模糊量词（高优先级词）
grep -rn -E '\b(correctly|optimally|modern|appropriately|comprehensively|properly|efficiently|relevant)\b' \
  /tmp/my-repos/MarkQWu-gstack/ \
  /tmp/my-repos/MarkQWu-bureau/ \
  --include="SKILL.md" 2>/dev/null | head -30
```
命中后：逐条替换为可测量表达，参考 twostraws audit 建议的替换模式（加版本号、加具体数量、改成可 grep 的布尔条件）。

```bash
# 检查 skill 是否有未注册的根目录副本（plugin.json 只加载 skills/ 目录）
# 确保每个 plugin.json 的 skills 路径与实际 SKILL.md 位置一致
find /tmp/my-repos/MarkQWu-gstack /tmp/my-repos/MarkQWu-bureau \
  -name "plugin.json" -exec grep -l "skills" {} \; 2>/dev/null
```
命中后：对照 plugin.json 声明的 skills 路径，确认每个 SKILL.md 都在该路径下。

### 6.2 灵感 → 实施路径
1. **想法**：在 gstack 的 `ios-design-review` 中引入「Apple HIG reference 文件」，替代现有内联规则
   - **为何可行**：HIG 每年更新，外置文件能独立维护；当前 skill 里有硬编码的版本依赖
   - **第一步**：把 `ios-design-review/SKILL.md` 中「HIG checklist」那部分提取到 `references/hig.md`，加 `${CLAUDE_SKILL_DIR}/references/hig.md`，预计 20 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 twostraws/SwiftUI-Agent-Skill 的核心目的**：提供 iOS/SwiftUI 代码的专业 AI review，9 维度覆盖

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 包含 `ios-design-review`、`ios-sync` 等 iOS-specific skill | gstack 面向完整开发流程，SwiftUI-Skill 专注代码 review | 高 |
| MarkQWu/bureau | 低 | 同为 Claude Code plugin | bureau 是知识管理，与 iOS 无关 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词（correctly、relevant 等） | `grep -rn "correctly\|relevant\|modern" gstack/ --include="SKILL.md"` | **gstack 命中 414 次**，涉及 ios-design-review、document-release、sync-gbrain 等多个 SKILL.md | 高 |
| 未注册根目录副本 | 检查 plugin.json skills 路径 | gstack 无冗余副本（SKILL.md 在根目录是 template，不同情形） | 低 |

**命中后的具体行动建议**：
- `gstack/ios-design-review/SKILL.md` → 搜索 `correctly`、`appropriate` → 替换为可测量表达 → 10 分钟可完成
- `gstack/document-release/SKILL.md` → 同上 → 5 分钟

### 8.3 别人的更优方案

1. **领域**：参考文件驱动 + 阶梯化加载
   - **本案例做法**：9 个 reference 文件 + `If doing a partial review, load only the relevant reference files`，按需加载降低 context 消耗（`skills/swiftui-pro/SKILL.md` 第 25 行）
   - **我的项目现状**：`gstack/ios-design-review/SKILL.md` 将所有 HIG 知识内联，无 reference 外置，每次全量加载
   - **如何借鉴**：把 HIG 规则提取到 `references/` 下 3-4 个文件，skill body 按 review 范围按需加载

2. **领域**：Output Format 完整示例
   - **本案例做法**：提供 `### ContentView.swift` 格式的完整 before/after 代码块示例
   - **我的项目现状**：bureau 的 `recall/SKILL.md` 无 output format 示例
   - **如何借鉴**：在 `recall/SKILL.md` 末尾加 `## Output Format` + 一个完整示例，15 分钟可完成

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：`allowed-tools` 声明
- **我的做法**：`gstack/ios-design-review/SKILL.md` 明确声明 `allowed-tools: [Bash, Read, Glob, Grep, AskUserQuestion]`
- **本案例做法**：`skills/swiftui-pro/SKILL.md` 完全没有 `allowed-tools` 字段
- **意义**：不声明 allowed-tools 会让 Claude Code 在限制模式下运行时提示用户确认每个工具，降低体验。gstack 在这里比 twostraws 更规范。

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据（如 `name`、`description`、`version`）。Claude Code 读 SKILL.md 时先解析 frontmatter 才能注册 skill。

### <a name="CLAUDE_SKILL_DIR"></a>${CLAUDE_SKILL_DIR}
> Claude Code 在加载 skill 时自动设置的环境变量，值是当前 SKILL.md 所在目录的绝对路径。在 SKILL.md 里用 `${CLAUDE_SKILL_DIR}/references/xxx.md` 引用同目录文件，能确保路径在任何项目下都正确解析，而不依赖于用户的项目结构。

### <a name="argument-hint"></a>argument-hint
> SKILL.md frontmatter 中的可选字段，在用户调用 skill 时显示为提示文本（如 `[focus area]`），告诉用户可以传入什么参数。不影响功能，纯粹提升可发现性。

### <a name="manifest"></a>manifest（plugin.json）
> 插件的「清单文件」，告诉 Claude Code 这个插件包含哪些 skills、commands、agents，以及各自的路径。如果 manifest 里 `skills` 路径指向 `./skills/`，那么根目录的 SKILL.md 即使存在也不会被加载。
