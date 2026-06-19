# lijigang/ljg-skills — 学习案例

**仓库**：https://github.com/lijigang/ljg-skills
**Stars**：4860 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-13（历史快照）| **生成日期**：2026-06-19（基于当前 HEAD）
**主题标签**：`single-purpose`, `experience-accumulation`, `examples-driven`, `manifest-discipline`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

lijigang/ljg-skills 是一套**个人生产力技能集**（4860 Stars），由中国开发者李继刚打造。定位是"把我自己的工作方式蒸馏成可重用的 Claude Code 技能"——每个技能都带有 `ljg-` 前缀，代表作者的个人品牌。仓库包含 22 个技能，覆盖学术论文阅读（ljg-paper、ljg-paper-river）、英文学习（ljg-word）、知识卡片生成（ljg-card）、Git 推送工作流（ljg-push）、圆桌讨论（ljg-roundtable）等场景。作者以 Emacs/Org-mode 为主要输出格式，体现了鲜明的个人风格。

### 1.2 架构剖析

**目录结构**：
```
ljg-skills/
├── skills/
│   ├── ljg-book/SKILL.md
│   ├── ljg-card/           ← 最复杂技能：HTML 模板 + Node.js/Playwright 截图
│   │   ├── assets/         ← HTML 模板（多种渲染模式）
│   │   ├── references/     ← 渲染模式指南（mode-*.md）
│   │   └── SKILL.md
│   ├── ljg-library/SKILL.md
│   ├── ljg-paper/          ← 学术论文分析
│   ├── ljg-paper-river/    ← 含 references/template.org（输出模板）
│   ├── ljg-present/        ← 含 assets/slogan_template.html
│   ├── ljg-push/           ← 含 Workflows/Push.md + Tools/Push.sh
│   ├── ljg-qa/             ← 含 Workflows/Extract.md + References/QuestionDesign.md
│   ├── ljg-rank/           ← 含 references/template.org
│   ├── ljg-roundtable/     ← 圆桌讨论技能
│   └── ... (共22个)
├── .claude-plugin/
│   └── plugin.json         ← manifest（100/100）
├── CLAUDE.md               ← 技能目录（仅列了7个，实际22个）
├── scripts/
│   ├── install.sh
│   └── sync-push.sh        ← 有路径注入风险
└── README.md
```

**文件类型分布**：22 个 SKILL.md，0 个独立 agent，0 个 hooks，多个辅助脚本（shell + Node.js）和参考文档。4 个技能（ljg-learn、ljg-roundtable、ljg-relationship、ljg-qa）已达 100/100，是整个仓库的质量锚点。

**编排关系**：技能间有少量依赖：`ljg-word-flow` 和 `ljg-paper-flow` 都调用 `ljg-card`（以不同模式生成卡片）。大多数技能是独立的，属于平铺结构。

**跨件契约**：`ljg-push` 依赖 `Workflows/Push.md` 和 `Tools/Push.sh`（已确认文件存在）；`ljg-qa` 依赖 `Workflows/Extract.md` 和 `References/QuestionDesign.md`（已确认存在）；`ljg-paper-river` 和 `ljg-rank` 共用 `references/template.org` 作为 Org-mode 输出模板。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「个人知识工作流具象化」——作者把自己多年形成的知识处理习惯（Org-mode 输出格式、圆桌讨论法、词汇深挖方式）写成 Claude 技能，让 AI 按照自己经过验证的流程工作，而不是让 AI 自由发挥。

**解决什么问题**：知识工作的高频重复操作（读论文、记单词、整理想法、写文章）各有一套最优 SOP，但手工执行效率低、质量不稳定。

**做了什么 trade-off**：选择了 Org-mode 作为输出格式（高度个人化，不面向大众），牺牲了普适性但获得了与作者个人工具链的完美集成。技能深度设计而非广度覆盖。

**反映什么认知模型**：作者相信"有结构的 AI 比自由发挥的 AI 更可靠"——每个技能都有精确的步骤和输出模板，Claude 的任务是按照预定格式执行，而非自由创作。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「个人工具链蒸馏型技能集」**

将一个人的高频知识工作操作（读文章、写笔记、整理思路）编码为带前缀的个人品牌技能集，每个技能是其对应工作流的具象化，输出格式与个人工具链深度集成（如 Org-mode）。

模式特征清单：
- **前缀命名品牌化**：所有技能用 `ljg-` 前缀，个人品牌清晰，避免与他人技能冲突
- **工具链集成**：输出格式（Org-mode）、输出目录（`~/Documents/notes/`）、时间戳命名规范统一
- **参考文档侧车**：复杂技能配套 `references/` 目录存放输出模板、设计规范等，保持 SKILL.md 简洁
- **渐进复杂度**：简单技能（ljg-word、ljg-plain）只有 SKILL.md；复杂技能（ljg-card）有 assets + references + 脚本

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人知识管理系统集成 | ✅ 高度适用 | 与特定工具链深度集成是优势不是缺陷 |
| 团队共用技能库 | ❌ 不适用 | Org-mode 输出格式、路径约定都是高度个人化的 |
| 作为模板学习 NL 编程模式 | ✅ 适用 | 100 分技能（ljg-learn、ljg-roundtable）是极好的学习样本 |
| 大规模分发（商业化） | ❌ 不适用 | 无抽象化、无配置化，用户需要大量修改才能适配自己工作流 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 个人工具链蒸馏（本仓库） | lijigang/ljg-skills | 高内聚、个人品牌强、质量上限高 | 可移植性差、前缀不适用大规模协作 |
| 组件超市（广度优先） | davila7/claude-code-templates | 覆盖广、易于发现 | 各组件质量参差不齐 |
| 工作流编排（线性流水线） | KhazP/vibe-coding-prompt-template | 跨会话状态保持、时间估算明确 | 固定流程不灵活 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 的技能目录列了 7 个（audit 时），现在仓库有 22 个技能，目录严重过期。**改进做法**：在安装脚本 `install.sh` 中添加一步，自动扫描 `skills/ljg-*/SKILL.md` 并生成 CLAUDE.md 的技能清单表格，消除手动维护的负担。**预期收益**：CLAUDE.md 永远准确，每次安装后自动更新。

2. **当前问题**：14 个 `user_invocable: true` 的技能无任何 `<example>` 示例块。**改进做法**：为每个技能添加 1 个完整示例（User 触发语 + 预期 Assistant 响应摘要），这是 NLPM 中 -15 分的最大单项扣分项。**预期收益**：评分从 85 提升到 100，且用户首次使用成本大幅降低。

3. **当前问题**：`scripts/sync-push.sh` 中 `SKILL="$1"` 未验证，可用 `../../outside-repo` 路径遍历到技能目录之外。**改进做法**：添加模式验证：`[[ "$SKILL" =~ ^ljg-[a-z][a-z0-9-]*$ ]] || exit 2`。**预期收益**：消除路径注入风险，约 1 行代码修复。

---

## 三、过去审查发现（2026-05-13 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-13 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| ljg-paper、ljg-invest 等 14 个技能 | 85 | 无 `<example>` 块（-15） |
| ljg-push/SKILL.md | 97 | Workflows/Push.md 引用路径未验证（-3） |
| ljg-word/SKILL.md | 97 | 示例中引用了废弃技能名 `ljg-explain-words` |
| ljg-learn/SKILL.md | 100 | — |
| ljg-roundtable/SKILL.md | 100 | — |
| CLAUDE.md | 85 | 技能清单列出 7 个（实际 21 个），严重过期 |

### 3.2 当时值得借鉴的模式

1. **「前缀命名命名空间」** → 所有技能用 `ljg-` 前缀，彻底消除与第三方技能的命名冲突，且建立了个人品牌。借鉴点：任何个人或组织的技能集，都应在技能名称中加上唯一标识符前缀。

2. **「template.org 侧车文件」** → `ljg-paper-river` 和 `ljg-rank` 在各自的 `references/` 目录中放置 `template.org`，定义 Org-mode 输出格式。SKILL.md 只说"参照 references/template.org 格式输出"，避免在 SKILL.md 内嵌大量格式规范。借鉴点：复杂的输出格式要求应该放在单独的参考文件中，SKILL.md 引用该文件而非内嵌。路径：`skills/ljg-paper-river/references/template.org`。

3. **「ljg-learn 100 分范例」** → `ljg-learn/SKILL.md` 同时做到了：清晰的 description（触发条件）、至少 1 个 `<example>` 块、明确的输出格式、无模糊量词。可以直接用来作为写 SKILL.md 的对照范本。

4. **「localhost:31337 通知端口统一」** → `ljg-qa` 和 `ljg-push` 用同一个端口通知，说明作者有"服务层统一"的意识——但这个服务没有被文档化，是个隐性依赖。借鉴点：如果多个技能依赖同一个外部服务，需要在 CLAUDE.md 中明确记录服务约定和启动方式。

### 3.3 当时的缺陷

1. **14 个技能无示例块** → 根本原因：示例块需要花时间构造真实的 input→output 对，是写技能中最费力的部分。-15 分/技能是 NLPM 最重的单项扣分，影响了整体分数从 90 到更高。自查：drama-workshop-skills 的 2 个 SKILL.md 均无示例块——**命中**。

2. **ljg-word 中的废弃技能名引用** → `SKILL.md` 示例第 12 行写 `[Calls ljg-explain-words with "Serendipity"]`，但技能已改名为 `ljg-word`。用户看到这个示例会试图调用不存在的技能。这是重构时没有同步更新的遗留问题——当你重命名技能时，所有引用该名称的地方都需要同步更新。

3. **CLAUDE.md 技能清单仅列 7/21 个** → 根本原因：随着技能数量增加，手工维护文档清单变得不可持续，作者跟不上更新节奏。这是"文档滞后代码"的经典问题。含义：**CLAUDE.md 中严重过期的信息比没有信息更危险**——新贡献者会根据错误清单做出错误的判断。

4. **sync-push.sh 路径注入** → `SKILL="$1"` 未验证，作为 rsync 目标路径的一部分使用。虽然实际被利用的场景有限（攻击者需要控制脚本调用者），但作为一个分发给他人使用的脚本，这个风险不容忽视。

### 3.4 当时的优化机会

1. 在 `ljg-word/SKILL.md` 第 12 行，将 `ljg-explain-words` 改为 `ljg-word`（1 行修复）。
2. 在 CLAUDE.md 技能清单中补全 14 个缺失的技能条目（约 30 分钟）。
3. 在 `scripts/sync-push.sh` 第 9 行后添加：`[[ "$SKILL" =~ ^ljg-[a-z][a-z0-9-]*$ ]] || { echo "Invalid skill name"; exit 2; }`。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| ljg-word 废弃技能名引用 | `grep "ljg-explain-words" skills/ljg-word/SKILL.md` | **仍存在**：第 12 行 `[Calls ljg-explain-words with "Serendipity"]` | 简单的 1 行修复超过 5 周未完成 |
| CLAUDE.md 技能清单过期 | `cat CLAUDE.md` 中技能清单 vs `ls skills/` | **仍存在且更严重**：CLAUDE.md 列出 7 个技能，当前仓库有 22 个（新增了 ljg-book、ljg-library 等） | 清单滞后从 14 条变为 15 条缺失 |
| ljg-qa 无示例块 | `grep "<example>" skills/ljg-qa/SKILL.md` | **ljg-qa 仍无示例块**（该技能 audit 时得 100 分是因为交叉引用验证通过，但示例块缺失的问题在 14 个技能中仍普遍存在） | 大量技能示例缺失问题未修复 |

### 4.2 架构演进

新增了 `ljg-book`（读书类技能）和 `ljg-library`（图书馆/知识库管理？），说明作者在持续扩展技能集。但 CLAUDE.md 更新频率落后于技能增加速度，这个问题在规模扩大后变得更明显。新增技能沿用了相同的目录约定（`ljg-` 前缀 + SKILL.md），说明架构规范稳定。

### 4.3 新增的可学习模式

**「ljg-library 技能」**：根据目录名推断，这可能是一个管理个人知识库的技能，有别于已有的 ljg-skill-map（扫描已安装技能）。如果其功能是管理文件系统中的参考资料，则与 echo-sleuth-for-claude 的 `experience-synthesis` 技能有功能重叠，值得深入阅读对比。

---

## 五、校准

### 5.1 我已经在做对的

1. **前缀命名**：echo-sleuth-for-claude 的技能名（`memory-management`、`git-mining`、`experience-synthesis`、`jsonl-core`）虽然无前缀，但都在独立仓库中，不会与第三方冲突。drama-workshop-skills 的技能在独立仓库中，命名空间隔离。✓

2. **参考文档侧车**：echo-sleuth-for-claude 的 `skills/git-mining/references/`、`skills/experience-synthesis/references/` 等都有独立的 references 目录，与 ljg-skills 的模式一致。✓

3. **技能目的专一**：drama-workshop-skills 的每个技能（`short-drama`、`short-drama-remake`）只做一件事，没有功能蔓延。✓

### 5.2 挑战 / 验证

**挑战了什么假设**：我以为"个人品牌技能集规模越大越好"——但本仓库的 CLAUDE.md 问题说明规模扩张需要对应的文档维护基础设施，否则文档质量会跟不上代码增长。技能数量增长应该触发文档更新机制，而不是靠人工记忆。

**验证了什么**：100 分技能（ljg-learn、ljg-roundtable）证明个人风格和技术规范不矛盾——Org-mode 输出格式是极度个人化的，但示例块、明确步骤、无模糊量词这些要求适用于任何技能。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能文件中是否有引用废弃名称（类似 ljg-explain-words 问题）
# 找到所有 SKILL.md 中的 Calls/Invokes 引用
grep -rn "Calls\|Invokes\|/[a-z]" ~/.claude/skills/*/SKILL.md 2>/dev/null | \
  grep -v "^Binary"
# 对每个引用，确认对应的技能名确实存在

# 检查我的 CLAUDE.md 技能清单是否覆盖所有技能
# 自动列出实际存在的技能
find . -name "SKILL.md" | sed 's/.*\/\([^/]*\)\/SKILL.md/\1/' | sort
# 对比 CLAUDE.md 中的清单，命中缺失的则补充

# 检查 Org-mode 类似的输出格式是否有 references/template 文件
find . -name "SKILL.md" -exec grep -l "template\|模板" {} \; | while read f; do
  dir=$(dirname "$f")
  if ! ls "$dir/references/" 2>/dev/null | grep -q "template"; then
    echo "MISSING template in references: $dir"
  fi
done
# 命中后：创建对应的 references/template.md 文件，将格式规范移入其中
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 的每个 SKILL.md 添加 1 个完整示例（参照 ljg-learn 100 分格式）。
   - **为何可行**：drama-workshop 已有固定的输出格式（中文剧本 + Org-mode 格式化），构造一个示例很直接。
   - **第一步**：读取 `ljg-learn/SKILL.md` 的 `<example>` 格式，按同样结构为 `short-drama/SKILL.md` 写 1 个：User: `/开始 [一部古装权谋短剧]` → Assistant: [策划方案的前 100 字摘要]。约 20 分钟。

2. **想法**：为 drama-workshop-skills 建立自动化 CLAUDE.md 技能清单生成脚本（参照 ljg-skills 的问题教训）。
   - **为何可行**：drama-workshop 目前只有 2 个技能，但将来可能扩展；趁少建立机制。
   - **第一步**：创建 `scripts/update-claude-md.sh`，扫描 `skills/*/SKILL.md` 的 `name:` 和 `description:` 字段，自动生成 CLAUDE.md 中的技能清单表格。约 45 分钟。

3. **想法**：为 echo-sleuth-for-claude 的 4 个 SKILL.md 补充 `<example>` 块（命中了本案例最严重的 -15 分问题）。
   - **为何可行**：echo-sleuth 的技能已经有完整的功能描述，构造 input→output 示例是最后一步。
   - **第一步**：优先为 `memory-management/SKILL.md` 添加 1 个示例（User: "帮我查找我上周做过的代码审查记录" → Assistant: [扫描 ~/.claude/projects/ 后的返回格式]）。约 30 分钟。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 lijigang/ljg-skills 的核心目的**：将作者个人的知识工作流（读论文、记单词、写文章、生成卡片）蒸馏成可重用的 Claude Code 技能，让 AI 按照经过验证的个人 SOP 工作。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 高 | 同样是"把个人专业工作流蒸馏成技能"；同样有 references/ 目录；同样面向中文用户 | drama 是内容创作；ljg 是知识管理 | 高 |
| MarkQWu/echo-sleuth-for-claude | 高 | 同样是个人知识管理工具；同样有多技能协作；同样有 references/ 侧车文件 | echo-sleuth 侧重会话历史挖掘；ljg 侧重内容生产 | 高 |
| MarkQWu/claude-for-legal | 低 | 都是 Claude Code plugin 形式 | claude-for-legal 是领域工具，不是个人工作流蒸馏 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 无 `<example>` 块（-15/技能） | `grep -L "<example>" skills/*/SKILL.md` | **drama-workshop-skills 命中 2/2**；**echo-sleuth 命中 4/4** | 高 |
| CLAUDE.md 技能清单过期 | 比对 CLAUDE.md 与实际技能数量 | **echo-sleuth CLAUDE.md 暂无**（无技能清单可比对）；drama-workshop 无 CLAUDE.md | 中（预防性） |
| 废弃技能名引用（类似 ljg-explain-words） | `grep -rn "旧技能名" skills/*/SKILL.md` | **未检测到**（drama 无内部技能间调用；echo-sleuth 的跨技能引用通过 agent 间接调用） | 暂无 |
| sync-push.sh 路径注入 | 检查脚本中 `$1` 的使用 | **drama-workshop-skills 有 install.ps1/install.sh**，需检查用户输入处理 | 中 |

**命中后的具体行动建议**：
- `drama-workshop-skills/short-drama/SKILL.md` → 添加 1 个 `<example>` 块 → 约 20 分钟
- `drama-workshop-skills/short-drama-remake/SKILL.md` → 添加 1 个 `<example>` 块 → 约 20 分钟
- `echo-sleuth-for-claude/skills/memory-management/SKILL.md` → 添加 1 个 `<example>` 块 → 约 15 分钟（余 3 个技能依次完成）

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：参考文档侧车（references/ 模式）
   - **本案例做法**：`ljg-paper-river/references/template.org` 定义了学术论文的 Org-mode 输出格式；`ljg-qa/References/QuestionDesign.md` 定义了问题设计的方法论——两者都将"格式规范"从 SKILL.md 中分离到独立文件
   - **我的项目现状**：`drama-workshop-skills` 有 `references/` 目录（18 个参考文件），已经采用了这个模式 ✓；但 `echo-sleuth-for-claude` 的 references 文件内容较薄，`jsonl-core/references/` 只有 2 个文件
   - **如何借鉴**：为 echo-sleuth 的 experience-synthesis 技能补充 `references/insight-taxonomy.md`（已存在）的内容深度，对标 ljg-qa 的 `QuestionDesign.md` 详细程度

2. **领域**：ljg-card 的多模式渲染架构（`mode-poster.md`, `mode-infograph.md` 等）
   - **本案例做法**：一个技能（ljg-card）支持多种输出模式，每个模式有独立的 `references/mode-*.md` 说明，SKILL.md 通过"模式参数"选择调用哪个参考文件
   - **我的项目现状**：drama-workshop-skills 的输出模式硬编码在 SKILL.md 中（国内模式/出海模式），没有外化为独立的模式文件
   - **如何借鉴**：将 drama-workshop 的国内/出海两种模式提取为 `references/mode-domestic.md` 和 `references/mode-overseas.md`，SKILL.md 引用对应文件

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：技能目录文档同步机制
  - **我的做法**：drama-workshop-skills 通过 `release-manifest.json` 作为版本和组件权威来源，且未来可以脚本化同步
  - **本案例做法**：CLAUDE.md 的技能清单靠手工维护，导致 22 个技能只列了 7 个；没有任何机制保证文档与代码同步
  - **意义**：我的方向是对的——版本/技能清单应该单一来源 + 脚本自动同步，而不是依赖人工记忆。这个差异在规模扩大时会更加明显。

---

## 八、术语表

### <a name="org-mode"></a>Org-mode 格式
> Emacs 文本编辑器中的一种结构化文本格式，以 `.org` 为扩展名。特点是纯文本、层级结构（用 `*` 标记标题层级）、支持任务管理、时间戳和表格。lijigang 用 Org-mode 作为所有知识管理输出的标准格式，与他的个人工作工具链深度集成。对 Word/Markdown 用户来说，Org-mode 相当于"结构更严格的 Markdown + 内置任务管理"。

### <a name="skill-prefix"></a>技能前缀（Skill Prefix）
> 为技能名称添加的短标识符前缀（如 `ljg-`），用于建立命名空间。好处：防止与第三方技能同名冲突；建立个人品牌；让用户知道这些技能属于同一套体系。Claude Code 中技能名唯一，若两个不同来源的技能同名，后安装的会静默覆盖前者。

### <a name="references-sidecar"></a>侧车文件（Sidecar Reference）
> 放在技能目录的 `references/` 子目录中的辅助文档，存放 SKILL.md 引用的复杂格式规范、输出模板、方法论等内容。SKILL.md 用相对路径引用侧车文件，保持 SKILL.md 自身简洁。类比：乐谱是主文件，乐器演奏指南是侧车文件。

### <a name="path-traversal"></a>路径遍历（Path Traversal）
> 一种安全漏洞：攻击者通过在文件路径参数中插入 `../../` 等相对路径符号，访问程序预期目录之外的文件或目录。`sync-push.sh` 中 `SKILL="$1"` 未经验证就用于 rsync 路径构建，允许调用者通过 `bash sync-push.sh "../../etc"` 这样的参数访问技能目录之外的位置。
