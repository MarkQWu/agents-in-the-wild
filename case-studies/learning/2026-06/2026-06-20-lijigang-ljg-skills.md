# lijigang/ljg-skills — 学习案例

**仓库**：https://github.com/lijigang/ljg-skills
**Stars**：4860 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-13（历史快照）| **生成日期**：2026-06-20（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `security-gate`, `vague-quantifier`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

李继刚的个人 Claude Code skills 集合（ljg = 作者名缩写），覆盖他个人学习、写作、研究工作流。4860 stars，是中文社区里关注度最高的个人 skills 仓库之一。仓库分两个分支：master（org-mode 输出）和 md（markdown 输出），当前 HEAD 在 master 分支，输出格式为 org-mode。

关键事实：
1. 22 个 skill（audit 时 21 个，现在增加了 `ljg-book` 和 `ljg-library`）
2. 所有 skill 都有 `ljg-` 前缀，强约定命名
3. 输出到 `~/Documents/notes/`，有跨 skill 共同的输出格式约定（org-mode 粗体 `*text*`，时间戳 `date +%Y%m%dT%H%M%S`）
4. 维护者本人大量使用 emacs org-mode，整个 skill 集合服务于他的 emacs 工作流

### 1.2 架构剖析

```
ljg-skills/
├── skills/
│   ├── ljg-book/         # 新：拆书分析
│   ├── ljg-card/         # 内容→PNG（含 Playwright）
│   │   ├── SKILL.md
│   │   ├── assets/       # HTML 模板 + capture.js
│   │   └── references/   # 每种模式（poster/infograph/long/big/whiteboard）的 md
│   ├── ljg-learn/        # 概念八维解剖（有 example）
│   ├── ljg-library/      # 新：书库管理 + ASCII 图（含 Python render 脚本）
│   ├── ljg-paper/        # 论文分析（无 example ← 已知缺陷）
│   ├── ljg-paper-flow/   # paper + card 组合
│   ├── ljg-push/         # git push 工作流（含 Workflows/Push.md + Push.sh）
│   ├── ljg-qa/           # QA 提炼（含 Workflows/ + References/）
│   ├── ljg-roundtable/   # 多角色圆桌（有 example）
│   ├── ljg-skill-map/    # 扫描已安装 skills（含 scripts/scan.sh）
│   ├── ljg-word/         # 英语单词深挖（example 引用已废弃 skill 名）
│   └── ... 另外 11 个 skill
├── CLAUDE.md             # 架构说明（技能清单只列 7 个 ← 已知缺陷）
├── README.md
└── scripts/              # install.sh, sync-push.sh（含安全漏洞）
```

- **文件类型分布**：22 个 SKILL.md、0 个 agent、0 个 command、0 个 hook
- **编排关系**：平铺结构。每个 SKILL.md 独立，无路由 skill、无 meta skill。`ljg-word-flow` 和 `ljg-paper-flow` 依赖 `ljg-card`（内部引用），这是唯一的 skill→skill 组合依赖，但未在 CLAUDE.md 中记录
- **跨件契约**：`ljg-qa` 和 `ljg-push` 各自引用 `Workflows/Extract.md`、`Workflows/Push.md`（按技能目录内 `Workflows/` 子目录存放），确认存在。`ljg-rank` 和 `ljg-paper-river` 引用 `references/template.org`，也确认存在

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「约定优于配置」——`ljg-` 前缀约定、输出到 `~/Documents/notes/`、统一 org-mode 格式，这些约定让不同 skill 的输出可以被同一个 emacs 工作流无缝处理
- **解决什么问题**：把作者个人的"大量阅读 + 思考 + 输出"工作流自动化。ljg-learn（解剖概念）→ ljg-think（深度思考）→ ljg-rank（领域降秩）→ ljg-paper（论文分析）→ ljg-card（输出为 PNG）构成一条完整的知识加工流水线
- **Trade-off**：
  - 选 org-mode 输出而非 markdown，满足作者自己的工具偏好，但对不用 emacs 的用户毫无意义
  - skills 完全平铺，不分层，上手简单但组合依赖（如 ljg-paper-flow 依赖 ljg-card）无法被工具检查
- **认知模型**：作者把每个 skill 看作"一种思考工具"，对应一种认知动作（拆书、解剖概念、降秩、圆桌讨论）

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「个人工作流自动化 + 约定优于配置」**

所有 skill 服务于同一个人的工作流，共享输出目录、输出格式约定，不需要对外兼容。这与"公共插件"的设计目标完全不同。

模式特征清单：
- 强约定命名（`ljg-` 前缀），所有 skill 一眼认出
- 输出目录约定（`~/Documents/notes/`），org-mode 格式统一
- 无 router 或 meta skill——用户通过名字直接选择对应 skill
- reference 文件按 skill 子目录存放（`skills/ljg-qa/References/`），不共享，各自独立

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人工作流自动化（一个人的工具集） | ✅ 高度适用 | 约定不需要被外人理解，只需作者本人用 |
| 团队共享 skill 库 | ❌ 不适用 | org-mode 输出、emacs 路径约定，其他人无法直接用 |
| 作为其他插件的依赖 | ⚠️ 有限适用 | skill 间存在隐式依赖（ljg-paper-flow → ljg-card），无声明契约 |
| 教学示例（中文社区） | ✅ 适用 | skill 设计清晰，中文注释，适合国内社区学习参考 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：个人工作流平铺 | lijigang/ljg-skills | 灵活，每个 skill 独立，可随时增删 | 缺乏跨 skill 组合的声明式管理 |
| 分层命令编排 | 本 repo 的 nlpm | 可以做复杂多步工作流，依赖关系明确 | 架构成本高，维护复杂 |
| 领域专项插件 | google-gemini/gemini-skills | 对齐产品边界，易于定向分发 | 修改需要协调团队，不灵活 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 技能清单只列了 7 个 skill，实际 22 个，新来的贡献者或工具扫描到的是严重不完整的画面。**改进做法**：用 `scripts/scan.sh`（`ljg-skill-map` 的扫描脚本）在 CI 或 pre-commit 时自动输出 skill 列表，写入 CLAUDE.md skill inventory 表。**预期收益**：CLAUDE.md 永远与实际一致，不需要手动维护。

2. **当前问题**：`ljg-push/SKILL.md` 和 `ljg-qa/SKILL.md` 里的 `curl -s -X POST http://localhost:31337/notify` 硬编码了 voice notification，任何进程都可以绑定 31337 端口。**改进做法**：加 `[ "${LJG_NOTIFY:-0}" = "1" ] && curl ...` 的环境变量保护，只有明确设置 `LJG_NOTIFY=1` 时才发出通知请求。**预期收益**：不影响 99% 的使用场景，消除安全隐患。

3. **当前问题**：14 个 user_invocable 的 skill 没有 `<example>` 块。**改进做法**：每个 skill 加 1-2 个 `<example>` 块，格式：`User: /ljg-xxx <输入>\nAssistant: [输出格式说明]`。**预期收益**：Claude 执行时有参考输出格式，减少格式错误。

---

## 三、过去审查发现（2026-05-13 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-05-13 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| 14 个 skill（ljg-paper 等） | 85/100 | 无 `<example>` 块（−15） |
| CLAUDE.md | 85/100 | Skill inventory 只列 7 个（−15） |
| ljg-push/SKILL.md | 97/100 | Workflows/Push.md 路径待确认（−3） |
| ljg-word/SKILL.md | 97/100 | Example 引用了已废弃 skill 名（−3） |
| 4 个 skill（ljg-learn 等） | 100/100 | — |
| .claude-plugin/plugin.json | 100/100 | — |

### 3.2 当时值得借鉴的模式

1. **`ljg-learn` 八维概念解剖框架** → 为什么好：把"学一个概念"分解为历史/辩证/现象/语言/形式/存在/美感/元反思八刀，每刀的产出有明确格式（2-3 句，只留筋骨），消除了模糊输出的可能 → 原文：`skills/ljg-learn/SKILL.md` 的"八刀"章节 → 借鉴：把复杂认知任务分解为有限数量的固定切面，每个切面给出字数约束

2. **按 skill 存放 reference 文件** → 为什么好：`skills/ljg-qa/Workflows/Extract.md`、`skills/ljg-qa/References/QuestionDesign.md`——把参考文档放在用它的 skill 目录里，自包含，避免全局共享文件引发的依赖乱象 → 借鉴：reference 文件应该放在依赖它的 skill 目录里，不要放在全局 references/

3. **`ljg-roundtable` 多角色圆桌示例** → 为什么好：有 `<example>` 块展示具体输入→输出，格式规范，Claude 可以直接参照 → 借鉴：任何有特定输出格式的 skill 都需要 `<example>` 块

4. **`ljg-card` 多渲染模式 + 模式文件** → 为什么好：每种渲染模式（poster/infograph/long/whiteboard/big）有独立的 `references/mode-xxx.md`，修改一种模式不影响其他模式 → 借鉴：多模式 skill 的各模式应该有独立的 reference 文件

### 3.3 当时的缺陷

1. **14 个 user_invocable skill 无 `<example>` 块** → 根本原因：作者本人对这些 skill 非常熟悉，不需要示例就能调用；他写 skill 是给自己用的，不是给社区用的，所以没想到要加示例 → 自查：我的 echo-sleuth 的 4 个 SKILL.md 全部没有 `<example>` 块——这是同一个问题

2. **CLAUDE.md skill inventory 严重过期** → 根本原因：CLAUDE.md 只在项目初期写过一次，后来每次新增 skill 都忘了更新清单；skill 列表在两个地方维护（CLAUDE.md 和实际文件系统），没有自动化保证同步 → 自查：我的 drama-workshop-skills 的 README.md 的功能列表有没有同步？

3. **`scripts/sync-push.sh` 路径穿越漏洞** → 根本原因：`SKILL="$1"` 直接用在 rsync 目标路径里，调用者传入 `../../sensitive-dir` 就能把文件 rsync 到 skills 目录外 → 这是输入没有在系统边界处验证的经典错误 → 自查：我的 shell 脚本里有没有把外部输入直接拼接进路径或命令？

### 3.4 当时的优化机会

1. **`ljg-word/SKILL.md` Example 中 `[Calls ljg-explain-words with "Serendipity"]`**：skill 已改名为 `ljg-word`，示例还用旧名，直接引起混乱
2. **`ljg-qa` 和 `ljg-push` 的 localhost:31337 curl**：加 opt-in 环境变量保护
3. **`scripts/sync-push.sh` 参数验证**：加 `[[ "$SKILL" =~ ^ljg-[a-z][a-z0-9-]*$ ]] || exit 2`

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| ljg-word stale example（叫 ljg-explain-words） | `grep "ljg-explain-words" skills/ljg-word/SKILL.md` | **仍存在**：line 12 仍然是旧名 | 1+ 个月了未修复，misleads users |
| CLAUDE.md 只列 7 个 skill | `grep -c "ljg-" CLAUDE.md` 数 inventory 行数 | **仍存在**：还是只列 7 个，但实际有 22 个 | 差距从 14 个变成 15 个（新增了 ljg-book 和 ljg-library） |
| ljg-push/ljg-qa localhost:31337 | `grep "31337" skills/ljg-push/SKILL.md` | **仍存在**：line 72 完全未变 | security issue 未修复 |

### 4.2 架构演进

Audit 时 21 个 skill，现在 22 个：
- 新增 `ljg-book`（拆书：提炼 delta = 作者的贡献对比已有共识的认知位移，最后画 ASCII 参考系图）
- 新增 `ljg-library`（书库：管理已读书目，含 Python render.py 生成 ASCII 可视化）

这说明作者在持续扩展他的知识加工工具集，从"读论文"（ljg-paper）扩展到"读书"（ljg-book）再到"管理书库"（ljg-library）——是知识工作流的纵向深化。

但 CLAUDE.md 没有随之更新，说明文档维护仍是最低优先级。

### 4.3 新增的可学习模式

1. **`ljg-book` 的"认知位移框架"**：把一本书的价值定义为 delta（作者挪动了什么，从共识的 X 到新的 Y），并用 ASCII 参考系图把这个 delta 可视化。这是一种非常有原创性的"书的价值评估方法"。

2. **`ljg-library` 的 Python render 脚本**：`skills/ljg-library/assets/render.py` 用纯 Python 生成 ASCII 可视化——不依赖 npm、Playwright 等重工具，比 ljg-card 的 Playwright 路径更轻量，值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **reference 文件按模块存放**：我的 drama-workshop-skills 的 `references/` 放在各 skill 目录里，不是全局共享
2. **强约定命名前缀**：我的 drama-workshop-skills 有命名规范（short-drama、short-drama-remake），与 ljg- 约定异曲同工
3. **输出路径约定**：我的 skill 输出到固定的 `~/Downloads/`，与 ljg 的 `~/Documents/notes/` 是同一种"约定输出目录"的思路
4. **skill 自包含**：我的每个 skill 目录包含它需要的所有 reference 文件

### 5.2 挑战 / 验证

这次案例**挑战**了我对"不加 example 也没关系"的假设。ljg-skills 14 个 skill 无 example，在 90/100 高分下也被扣了 15 分——这说明 example 缺失是结构性缺陷，不是小问题。我的 echo-sleuth 所有 4 个 SKILL.md 同样没有 example，这次确认要修。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 里有多少个有 example 块
for f in $(find . -name "SKILL.md"); do
  count=$(grep -c "<example>" "$f" 2>/dev/null || echo 0)
  echo "$count $f"
done | sort -n
```
命中 0 的文件：每个加至少 1 个 `<example>` 块，格式：`<example>\nUser: /skill-name <input>\nAssistant: [输出格式说明]\n</example>`。

```bash
# 检查我的 shell 脚本里有没有不安全的参数拼接
grep -rn '\$1\|\$2\|\$@\|\$\*' scripts/ --include="*.sh" | grep -v "^#"
```
命中后怎么办：在使用 `$1` 之前加 pattern 验证 `[[ "$1" =~ ^[a-z][a-z0-9-]*$ ]] || exit 1`。

```bash
# 检查我的 CLAUDE.md skill inventory 与实际 SKILL.md 文件是否同步
actual=$(find . -name "SKILL.md" | wc -l)
listed=$(grep -c "SKILL" CLAUDE.md 2>/dev/null || echo "0")
echo "实际: $actual, CLAUDE.md 里提到: $listed"
```
命中后怎么办：在 CLAUDE.md 里补全 skill inventory 表。

### 6.2 灵感 → 实施路径

1. **想法**：把 `ljg-learn` 的八维解剖框架引入 echo-sleuth 的 experience-synthesis skill
   - **为何可行**：echo-sleuth 的 experience-synthesis 目前只有"分类经验"的粗框架，八维解剖框架更系统
   - **第一步**：在 `experience-synthesis/SKILL.md` 里加一个"概念深化"的分支，输入一个从会话里提取出的"关键决策"，走历史→辩证→现象三刀压缩成一条教训；预计 20 分钟

2. **想法**：给 drama-workshop-skills 加类似 ljg-book 的"创作delta分析"功能
   - **为何可行**：参考剧本拆解的核心就是找出这个剧本"挪动了什么"（类型认知的 delta）
   - **第一步**：在 `short-drama-remake/SKILL.md` 里加"Delta 提炼"章节，格式：旧类型共识 X / 本剧创新点 Y / X→Y 的距离；预计 30 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 lijigang/ljg-skills 的核心目的**：把个人知识加工工作流（阅读/思考/写作）自动化，输出格式符合作者个人的 emacs+org-mode 工具链

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为个人知识管理型 skill，有多个 skill 平铺，处理文本分析 | echo-sleuth 面向会话历史；ljg-skills 面向读书/写作 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为内容生产工具，有 reference 分层 | 内容创作 vs 知识加工，领域不同 | 低 |
| MarkQWu/claude-for-legal | 无 | — | 法律工作 vs 个人学习，无交集 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 无 `<example>` 块 | `grep -L "<example>" */SKILL.md` | **echo-sleuth 命中全部 4 个 SKILL.md** | 高 |
| CLAUDE.md skill inventory 过期 | `diff <(find . -name "SKILL.md" \| ...) <(grep 表格行 CLAUDE.md)` | echo-sleuth CLAUDE.md 列出了全部 4 个 skill，**暂时同步** | 暂无 |
| localhost 硬编码网络调用 | `grep -rn "localhost\|curl.*POST"` | **echo-sleuth 无此问题** | 无 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/skills/jsonl-core/SKILL.md` → 加 `<example>User: 如何解析 JSONL\nAssistant: [展示 extract-messages.sh 的用法和输出格式]</example>` → 5 分钟
- `echo-sleuth-for-claude/skills/memory-management/SKILL.md` → 加 `<example>User: /audit\nAssistant: [扫描结果展示：哪些 memory.md 超过 90 天未访问]</example>` → 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：skill 内部的 reference 文件按功能分子目录（`Workflows/`、`References/`、`references/`）
   - **本案例做法**：`skills/ljg-qa/Workflows/Extract.md`（工作流参考）、`skills/ljg-qa/References/QuestionDesign.md`（领域参考）——两种 reference 用不同目录名区分类型
   - **我的项目现状**：echo-sleuth 的 skills/ 下的 reference 都混在一起，没有区分"工作流"和"领域知识"
   - **如何借鉴**：把 `skills/experience-synthesis/` 下的参考文档拆成 `workflows/`（how to run）和 `domain/`（what to extract）两个子目录

2. **领域**：`ljg-card` 的 HTML 模板 + Playwright 截图
   - **本案例做法**：用 HTML 模板 + `assets/capture.js` 把文本内容截图为 PNG，支持多种排版模式
   - **我的项目现状**：drama-workshop-skills 目前只输出文本剧本，没有可视化能力
   - **如何借鉴**：如果未来需要输出分镜可视化，可以直接参考 ljg-card 的 HTML+Playwright 架构

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：多层编排（commands → agents → skills）
  - **我的做法**：echo-sleuth 有 commands 分发任务给 agents，agents 再用 skills——有明确的调用链和职责边界
  - **本案例做法**：全部 22 个 skill 平铺，无 router、无 agent，ljg-paper-flow → ljg-card 的依赖是隐式的（靠 SKILL.md 文字描述，不是声明式）
  - **意义**：当 skill 数量超过 10 个时，明确的编排层能防止"隐式依赖迷宫"。ljg-skills 现在 22 个 skill，已经出现了 ljg-word-flow 和 ljg-paper-flow 依赖 ljg-card 但 CLAUDE.md 没有记录的问题

---

## 八、术语表

### <a name="org-mode"></a>org-mode
> emacs 编辑器里的一种文件格式（`.org`），类似 Markdown 但功能更强。可以做任务管理、文档写作、编程笔记。`*text*` 是 org-mode 里的粗体（Markdown 里是 `**text**`）。如果你不用 emacs，ljg-skills 的输出格式对你来说可能需要转换。

### <a name="路径穿越"></a>路径穿越
> Path Traversal 攻击。假设脚本用 `$1` 做 rsync 目标路径，攻击者传入 `../../etc/passwd`，rsync 就会把文件写到 `/etc/passwd`（如果有权限）。防御方法：在使用用户输入之前，验证它只包含合法字符（比如只允许 `a-z`、`0-9`、`-`）。

### <a name="user_invocable"></a>user_invocable
> SKILL.md frontmatter 里的一个字段。`true` 表示用户可以用 `/skill-name` 命令直接调用；`false` 表示只有 Claude 自己可以在执行任务时调用。如果 `user_invocable: true` 但没有 `<example>` 块，用户不知道怎么用这个 skill。
