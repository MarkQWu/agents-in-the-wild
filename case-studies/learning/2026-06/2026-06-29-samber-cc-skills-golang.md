# samber/cc-skills-golang — 学习案例

**仓库**：https://github.com/samber/cc-skills-golang
**Stars**：1362 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-06-29（目标仓库不可访问，仅基于 audit 快照）
**主题标签**：`single-purpose`, `manifest-discipline`, `cross-reference`, `examples-driven`, `vague-quantifier`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
samber/cc-skills-golang 是一套专为 Claude Code 设计的 Go 语言领域 [skill](#skill) 集合，作者是法国开源开发者 samber（samber/lo、samber/do 等热门 Go 库的作者）。该插件将 samber 本人在 Go 开发中的知识体系直接编码为 38 个 SKILL.md 文件，供 Claude Code 按需加载。用户通过 `claude plugin install` 一键安装，即可让 Claude Code 在 Go 相关对话中自动获得对应专业知识。

关键事实：
- 38 个 Go 领域 skill，涵盖并发、测试、性能、安全、DI、GraphQL 等主题
- 作者本人是多个被 skill 所引用的开源库的创建者（samber/lo、samber/do、samber/hot 等）
- 总体 NL 评分 87/100，是高水准的单领域 skill 集
- 安全级别 CLEAR：纯 Markdown 仓库，零可执行面

### 1.2 架构剖析
- **目录结构**：
```
cc-skills-golang/
├── .claude-plugin/
│   └── plugin.json          # manifest，注册全部 38 个 skill
├── skills/
│   ├── golang-benchmark/SKILL.md
│   ├── golang-cli/SKILL.md
│   ├── golang-concurrency/SKILL.md
│   ├── golang-context/SKILL.md
│   ├── golang-database/SKILL.md
│   ├── golang-dependency-injection/SKILL.md
│   ├── golang-error-handling/SKILL.md
│   ├── golang-graphql/SKILL.md      ← 空 stub
│   ├── golang-performance/SKILL.md  ← 最高分 91
│   ├── golang-security/SKILL.md     ← 最高分 91
│   ├── golang-uber-dig/SKILL.md     ← 悬空引用
│   ├── golang-uber-fx/SKILL.md      ← 悬空引用
│   └── ... (38 个)
└── CLAUDE.md                # 开发者指南
```
- **文件类型分布**：38 个 SKILL.md，1 个 plugin.json [manifest](#manifest)，1 个 CLAUDE.md
- **编排关系**：平铺式，每个 skill 独立，无 router 或 meta-skill。[frontmatter](#frontmatter) 的 `cross-references` 字段建立 skill 间的显式依赖图
- **跨件契约**：通过 `samber/cc-skills-golang@golang-xxx` 格式的跨引用字符串连接相关 skill；性能 skill 集群（performance、benchmark、troubleshooting、observability）有文档化的边界说明，互不重叠

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「一个 skill = 一个 Go 知识子域」——每个 skill 有 persona、modes、examples、cross-references、common-mistakes 五要素，且范围严格限定在单一子域内
- **解决什么问题**：把作者在 Go 开发（尤其是 samber 生态自己的库）中的专家知识固化为机器可读格式，降低用户从零写 prompt 的成本
- **做了什么 trade-off**：选择「平铺 + manifest 注册」而非「router 分发」，用 plugin.json 的字段列表代替运行时 dispatch 逻辑；换来了结构简单、每个 skill 独立可替换，但代价是跨 skill 之间的动态 context 传递能力为零
- **反映什么认知模型**：作者把 AI skill 类比成"标准库手册"——一个话题一篇文章，文章之间通过 cross-references 形成知识图谱

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「单领域技术栈平铺 Skill 库」**

38 个 skill 全部属于同一技术栈（Go），按知识子域平铺组织，通过 plugin manifest 批量注册，skill 间通过 cross-reference 字段形成隐式图谱。

模式特征清单：
- 特征 1：每个 skill 严格单职责，文件名 = 子域名（`golang-concurrency`、`golang-error-handling`）
- 特征 2：Skill 间通过 cross-references 字符串而非代码逻辑连接
- 特征 3：有记录在案的「skill 集群」——相关 skill 声明边界，防止内容重复
- 特征 4：Manifest 是唯一的注册入口，Manifest 是否同步直接决定 skill 是否可用
- 特征 5：作者即领域专家，skill 体现的是个人专有知识而非通用 AI prompt 技巧

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 单一编程语言 / 框架的深度知识包 | ✅ 高度适用 | 平铺结构在单领域内清晰可维护 |
| 跨领域综合工具包（前端+后端+DB+运维） | ❌ 不适用 | 子域过多会导致 manifest 臃肿，缺 router 难以按需激活 |
| 需要动态上下文传递的 workflow | ❌ 不适用 | 平铺 skill 之间没有运行时通信机制 |
| 个人知识体系/专家笔记 | ✅ 适用 | 个人对某个领域的深度覆盖，正好匹配平铺结构 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：单领域平铺 Skill 库 | samber/cc-skills-golang | 结构简单，各 skill 独立可测试，零运行时复杂度 | 无法动态组合，新 skill 要手动加 manifest |
| Router + Channels 分层 | musistudio/claude-code-router | 支持跨领域动态分发，单入口 | 维护 router 规则成本高，调试困难 |
| NL 表皮 + 原生二进制核心 | 777genius/claude-notifications-go | 性能优秀，可系统集成 | 技术门槛高，非 Go/Rust 作者难复制 |

### 2.4 改进空间
1. **当前问题**：golang-google-wire skill 被两处 cross-reference 引用但未实现。**改进做法**：补充 `skills/golang-google-wire/SKILL.md` 覆盖 Wire 编译期 DI 模式。**预期收益**：消除两处悬空引用，完善 DI 知识图谱（现有 dig/fx/samber-do/samber-lo 均已覆盖，Wire 是明显缺口）
2. **当前问题**：golang-graphql 是空 stub（45/100）但 `user-invocable: false`，导致既不能手动调用也不能自动激活。**改进做法**：要么补写正文（gqlgen + graphql-client），要么改 `user-invocable: true` 明确标为待完成占位符。**预期收益**：消除最大单文件扣分项，整体评分可从 87 升至约 89

---

## 三、过去审查发现（2026-04-29 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-29 当时得分 87/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/golang-graphql/SKILL.md | 45 | 空 stub，仅有占位文本 |
| skills/golang-stay-updated/SKILL.md | 77 | 仅资源列表，无代码示例 |
| skills/golang-popular-libraries/SKILL.md | 82 | 正文过薄，依赖未加载的引用文件 |
| skills/golang-uber-dig/SKILL.md | 82 | 悬空 cross-reference（golang-google-wire 不存在）|
| skills/golang-uber-fx/SKILL.md | 82 | 同上 |
| skills/golang-performance/SKILL.md | 91 | 最高分之一，ultrathink 指令 + decision tree |
| skills/golang-security/SKILL.md | 91 | 最高分之一，parallel audit agents 设计 |

### 3.2 当时值得借鉴的模式

1. **Skill 集群边界声明** → 为什么好：集群内的 performance/benchmark/troubleshooting/observability 四个 skill 各自的 description 中明确写出了与相邻 skill 的边界，防止重叠内容相互干扰 → 路径：skills/golang-performance/SKILL.md 的 Cross-References 部分 → 借鉴：在自己的 skill 集合中，若有功能临近的 skill，应在各自 frontmatter 的 description 里声明「本 skill 不覆盖 X，X 见 @xxx」

2. **ultrathink 指令 + decision tree 组合** → 为什么好：golang-performance 的 91 分得益于触发 ultrathink 模式 + 提供 decision tree，让模型在性能问题中按图索骥而非漫游 → 路径：skills/golang-performance/SKILL.md 的 Modes 和 Decision Tree 部分 → 借鉴：高复杂度 skill（如 architecture、debugging）应加 decision tree 引导推理路径

3. **parallel audit agents 模式** → 为什么好：golang-security 指示模型并行运行安全审查 agent，而非串行扫描 → 路径：skills/golang-security/SKILL.md → 借鉴：安全、review、audit 类 skill 可显式声明并发 subagent 策略

4. **common-mistakes 表格** → 为什么好：每个高分 skill 都有 common mistakes 部分，让模型知道哪些做法是陷阱 → 路径：几乎所有 89+ 分 skill → 借鉴：在 skill 正文末加一节「常见错误」，减少模型重蹈覆辙

5. **persona 声明一致性** → 为什么好：38 个 skill 在 frontmatter 的 persona 字段描述一致（均是对应子域的 Go 专家角色），减少角色漂移风险 → 借鉴：skill 集合中所有 skill 应保持 persona 风格一致，避免某些有 persona、某些没有的割裂感

### 3.3 当时的缺陷

1. **空 stub 但 user-invocable: false 死锁** → 根本原因：作者在 golang-graphql 写了 frontmatter（含 `user-invocable: false`）后搁置了正文，导致 skill 既不能手动调用（`user-invocable: false` 禁止斜杠命令），也不能自动激活（空正文没有触发信号），变成一个「存在但不可触达」的孤儿 → 自查：我的 skill 有没有 `user-invocable: false` 但正文为空的情况？

2. **悬空 cross-reference（golang-google-wire）** → 根本原因：作者在规划阶段写了引用 `golang-google-wire` 的 cross-reference，但 Wire skill 始终未实现；两个 skill（uber-dig、uber-fx）用了相同的模板文本，说明是复制粘贴自规划草稿 → 加载 golang-uber-dig 时，模型会看到「参见 golang-google-wire」的提示然后加载一个不存在的 skill，结果是静默失败 → 自查：我有没有在 cross-references 中引用了实际不存在的 skill？

3. **golang-stay-updated 仅资源列表** → 根本原因：把「工具/网站推荐清单」当成了 skill 正文，缺少任何过程性指导（怎么检查 release notes？怎么跑 govulncheck？怎么更新 go.mod？）→ 这种 skill 加载后，模型收到的是一份书单而非可操作的指导 → 自查：我的 skill 正文有没有以「参考资料列表」代替「行动指导」？

### 3.4 当时的优化机会

1. **golang-popular-libraries 需要内联决策矩阵**：现在正文只列了库类别名字，把所有深度内容推给引用文件。如果引用文件没被加载，skill 正文几乎没有独立价值。修法：在 SKILL.md 里加一个「快速选型」对照表，让 skill 在无引用时也能独立运作
2. **批量消除模糊量词**：35 个 89 分 skill 都扣了「minor vague quantifiers」分，主要是一些「appropriate」「efficient」等词。可以批量 `sed` 替换为可量化描述
3. **golang-graphql stub 补全或明确占位**：这是拉低平均分最大的单文件问题（45/100）

---

## 四、现在 vs 过去对比

> 目标仓库（samber/cc-skills-golang）在本运行环境中无法访问（HTTP 403），以下分析基于 audit 快照，无法进行实时 grep 验证。

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| golang-graphql 空 stub | `cat skills/golang-graphql/SKILL.md \| wc -c` → 若 < 200 则仍为空 | 无法验证（仓库不可访问） | 若仍未填写，该 skill 一年多来始终不可激活 |
| golang-google-wire 悬空引用 | `ls skills/golang-google-wire/` → 若目录不存在则仍悬空 | 无法验证 | 社区用 uber-dig/uber-fx 时持续收到静默失败 |
| golang-stay-updated 仅书单 | `grep -c '#' skills/golang-stay-updated/SKILL.md` → 看是否有实质性 section | 无法验证 | — |

### 4.2 架构演进
无法从当前 HEAD 与 audit 快照的目录清单进行对比（仓库不可访问）。从 audit 记录看，2026-04-29 时共有 40 个制品（38 skill + plugin.json + CLAUDE.md），结构已相当成熟，预期未有大规模重组。

### 4.3 新增的可学习模式
暂无（无法访问当前 HEAD）。

---

## 五、校准

### 5.1 我已经在做对的
1. **Skill 单职责**：我的 skill 集合（如 MarkQWu/graphify）也遵循一个 skill 管一个知识域的原则
2. **Plugin manifest 同步**：我在修改 skill 目录时会同步更新 plugin.json，与 samber 的做法一致
3. **Cross-reference 显式声明**：在有关联 skill 时写 cross-references 字段，而非把所有内容堆在一个 skill 里

### 5.2 挑战 / 验证
**被这次案例验证的做法**：skill 要有 `## Examples` 节才能把知识传递给模型。本案例 35 个高分 skill 全部有 examples，而 3 个低分 skill（graphql/stay-updated/popular-libraries）都缺少具体示例。这验证了「示例驱动」是高分 skill 的共同特征，而不只是加分项。

---

## 六、行动

### 6.1 自查动作

```bash
# 1. 检查我的 skill 有没有 user-invocable: false 但正文为空（空 stub 死锁）
for f in ~/.claude/skills/*/SKILL.md; do
  if grep -q 'user-invocable: false' "$f" && [ $(wc -c < "$f") -lt 300 ]; then
    echo "可能的空 stub: $f"
  fi
done
# 命中后：把 user-invocable 改为 true（标记为手动占位符），或补写正文

# 2. 检查我的 skill cross-references 是否有悬空引用
grep -rn 'cross-references' ~/.claude/skills/*/SKILL.md | grep '@' | while IFS= read -r line; do
  ref=$(echo "$line" | grep -oP '(?<=@)[a-z0-9-]+')
  if [ -n "$ref" ] && [ ! -d ~/.claude/skills/$ref ]; then
    echo "悬空引用: $line"
  fi
done
# 命中后：创建对应 skill 目录或删除悬空引用

# 3. 检查是否有正文只有链接列表的 skill（书单 skill）
grep -rL '```\|##.*步骤\|##.*流程\|##.*示例\|##.*Examples' ~/.claude/skills/*/SKILL.md | head -10
# 命中后：在对应 SKILL.md 加入至少一个操作性段落
```

### 6.2 灵感 → 实施路径

1. **想法**：给 MarkQWu/graphify 的 skill 加 skill 集群边界声明
   - **为何可行**：graphify 有多个功能临近的 skill（code-graph、db-schema、dependency-map），加边界声明可减少模型混淆
   - **第一步**：在每个 skill 的 frontmatter description 末尾加「本 skill 不覆盖 X，X 见 @graphify@xxx」，约 10 分钟

2. **想法**：为复杂 skill 加 decision tree
   - **为何可行**：graphify 的 query-graph skill 逻辑分支多，decision tree 可让模型按图推理
   - **第一步**：在 skills/query-graph/SKILL.md 的 Modes 节后加 `## Decision Tree` 节，用 `- If X → Step Y` 格式写，约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 注：本运行环境中无法访问外部 GitHub 仓库（HTTP 403），以下分析基于 `learning/my-repos.json` 元数据和已知结构。

### 8.1 目的对齐度

- **本案例 samber/cc-skills-golang 的核心目的**：将单一编程语言（Go）领域的专家知识编码为 Claude Code skill 集合，让模型在 Go 编程对话中自动获得专业指导

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同为「知识领域 skill 集合」，面向特定社区（gobuildit） | drama-workshop-skills 更泛用，不局限于单一语言 | 高 |
| MarkQWu/graphify | 中 | 都是 Claude Code plugin，含多个 skill | graphify 是工具型（生成知识图谱），samber 是知识型（编码专家经验） | 中 |
| MarkQWu/bureau | 低 | 都是 Claude Code plugin | bureau 是 session 知识管理，与语言知识无关 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| cross-reference 悬空引用 | `grep -rn '@' skills/*/SKILL.md \| grep cross` | 无法验证（仓库不可访问），但从已知结构推测：drama-workshop-skills 如有 cross-references 需核查 | 中 |
| user-invocable: false + 空正文 | `wc -c skills/*/SKILL.md` 配合 grep frontmatter | 无法验证 | 高（若命中则 skill 完全不可激活） |
| 仅资源列表型正文（书单 skill）| `grep -L '##.*步骤\|##.*示例'` | 无法验证，但 echo-sleuth 类 skill 存在此风险 | 中 |

**命中后的具体行动建议**：
- drama-workshop-skills 的任何 SKILL.md → 检查 cross-references 中引用的 skill 是否都存在对应目录 → 5 分钟可完成

### 8.3 别人的更优方案

1. **领域**：Skill 集群边界文档化
   - **本案例做法**：skills/golang-performance/SKILL.md 的 Cross-References 节明确写「本 skill 不覆盖 profiling setup（见 golang-troubleshooting）」
   - **我的项目现状**：MarkQWu/graphify 中功能临近的 skill（如代码分析类）之间没有显式边界声明，用户可能不知道选哪个
   - **如何借鉴**：在各 skill 的 description 或 Cross-References 中加一行「本 skill 不处理 X」，5 分钟/文件

2. **领域**：common-mistakes 专节
   - **本案例做法**：几乎每个 89+ 分 skill 都有 `## Common Mistakes` 节，列出已知陷阱
   - **我的项目现状**：drama-workshop-skills 中的 skill 以「最佳实践」为主，缺少「常见错误」视角
   - **如何借鉴**：在每个 skill 末尾加 `## 常见错误` 节，列 3-5 个陷阱，各自一句话说明后果 → 10 分钟/文件

### 8.4 反向：我的项目做得比他们好的地方

本案例未发现我的项目更优的维度（samber 的 skill 质量普遍高于我的项目现状）。

---

## 八、术语表

### <a name="skill"></a>skill
> Claude Code 中的「技能文件」——一个 SKILL.md 文件，描述模型在特定场景下应具备的知识、角色和行为方式。类比于人类专家的「说明书」：告诉 Claude「当用户问 Go 并发时，你应该以这种方式思考和回答」。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明 skill 的元数据（name、description、user-invocable、model 等）。Claude Code 读 SKILL.md 时先解析 frontmatter 才知道这个 skill 怎么注册和调用。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，即 plugin.json。告诉 Claude Code 这个插件包含哪些 skill 的路径。Manifest 里没有列出的 skill 文件不会被加载，即使文件在硬盘上存在。

### <a name="cross-reference"></a>cross-reference
> Skill 之间的显式引用。格式如 `samber/cc-skills-golang@golang-dependency-injection`，表示「当前 skill 建议同时加载另一个 skill」。若引用的 skill 不存在，模型会静默失败（收到引用提示但加载不到内容）。

### <a name="stub"></a>stub
> 占位文件——有 frontmatter 和文件名，但正文内容尚未填写（通常只有一句「Content will be added later」）。stub 是开发过程中的临时状态，但若与 `user-invocable: false` 结合，会造成 skill 完全不可激活的死锁。
