# AgriciDaniel/claude-blog — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-blog
**Stars**：562 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-17（基于审计数据）
**主题标签**：vague-quantifier, model-pinning, examples-driven, security-gate, single-purpose

---

## 一、理解（基于审计报告与注册表数据）

### 1.1 仓库概览
这是 AgriciDaniel 系列插件中专注于博客写作全流程的一款，用 4 个专职 agent 覆盖研究→写作→审核→SEO 优化的完整流水线，配套 22 个细粒度 skill 文件。562 颗星说明内容创作者把这套工作流当成了一个完整的"AI 编辑室"。整体 NL 质量在同类插件中属于上游水平（92/100），但安全层面被 BLOCKED，原因包括 curl-pipe-sh 安装脚本和一个 npx 自动安装的 MCP 依赖。

关键事实：
- 4 个 agent 分工精确（researcher / writer / reviewer / SEO），是典型的流水线型多 agent 架构
- 22 个 skill + 4 个 agent = 26 个 NL artifact（含 plugin.json 和 CLAUDE.md 共 28 个）
- 最大扣分项：4 个 agent 全部缺少 model 声明（-5 × 4）和 examples 示例（-15 × 4）
- 存在一个跨件破损引用：`skills/blog-write/SKILL.md` 引用了不存在的 `references/cta-placement.md`
- `.mcp.json` 自动运行 `npx -y @ycse/nanobanana-mcp`——High 级安全风险

### 1.2 架构剖析
```
claude-blog/
├── .claude-plugin/
│   └── plugin.json          # 95分
├── CLAUDE.md                # 88分（含curl-pipe-sh安全问题）
├── agents/
│   ├── blog-researcher.md   # 72分（无model、无examples、vague）
│   ├── blog-writer.md       # 76分（无model、无examples）
│   ├── blog-reviewer.md     # 77分（无model、无examples、Bash未使用）
│   └── blog-seo.md          # 80分（无model、无examples）
├── skills/
│   ├── blog/SKILL.md        # 核心路由 skill
│   ├── blog-write/SKILL.md  # 88分（vague量词×6、引用破损）
│   ├── blog-strategy/SKILL.md
│   ├── blog-brief/SKILL.md
│   ├── blog-rewrite/SKILL.md
│   └── blog-{factcheck,image,audio,notebooklm,google,calendar,...}/SKILL.md  # ×17 个
├── scripts/
│   └── *.py                 # 分析脚本（含路径穿越漏洞）
├── .mcp.json                # npx 自动安装 MCP（High 安全风险）
├── install.sh               # curl-pipe-sh（Critical）
└── requirements.txt         # pip 无哈希验证
```

- **文件类型分布**：4 个 agent / 22 个 SKILL / 1 个 plugin.json / 1 个 CLAUDE.md / 多个 Python 脚本
- **编排关系**：4 个 agent 构成流水线（researcher 输出供 writer 消费，writer 输出供 reviewer 审核，reviewer 输出供 seo 优化）；skill 文件作为知识库被 agent 按需引用
- **跨件契约**：`blog-write/SKILL.md` 引用 `references/cta-placement.md`，但该文件可能不存在；CLAUDE.md 声明有 14 个 reference 文件，只列出了 13 个

### 1.3 设计思路 / 方法论
- **核心设计哲学**："分工到人、知识到 skill"——每个 agent 只做一件事，深度知识（如 SEO 规则、CTA 规范）沉淀在对应 skill 文件中
- **解决什么问题**：博客从选题到发布的完整 AI 辅助流程，让内容团队的每个角色（研究员、撰稿人、编辑、SEO 专家）都有对应的 AI 同事
- **Trade-off**：22 个细粒度 skill 意味着高度专业化，但也带来维护负担；跨件引用越多，破损风险越高（本案中已出现引用缺失）
- **认知模型**：用 agent 建模"角色"，用 skill 建模"角色的专业知识"，命令是"项目启动"，结果是"发布物"

---

## 二、过去审查发现（2026-04-06 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**，安全状态 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/blog-researcher.md | 72 | 无 model 声明、无 examples、vague 量词×5 |
| agents/blog-writer.md | 76 | 无 model 声明、无 examples、vague 量词×2 |
| agents/blog-reviewer.md | 77 | 无 model 声明、无 examples、Bash 声明但未使用 |
| agents/blog-seo.md | 80 | 无 model 声明、无 examples |
| skills/blog-write/SKILL.md | 88 | vague 量词×6（appropriate、relevant、suitable） |
| skills/blog-strategy/SKILL.md | 92 | vague 量词×4 |
| skills/blog-brief/SKILL.md | 93 | vague 量词×2 |
| skills/blog/SKILL.md | 93 | vague 量词×1 |
| skills/blog-rewrite/SKILL.md | 94 | vague 量词×3 |
| skills/blog-{factcheck...analyze}/SKILL.md | 95-98 | 轻微 vague |
| .claude-plugin/plugin.json | 95 | 结构正确，轻微版本不一致 |
| CLAUDE.md | 88 | 含 curl-pipe-sh 安全问题 |

**加权平均**：92/100

### 2.2 当时值得借鉴的模式

1. **专用 runner 脚本模式**：`skills/blog-notebooklm/scripts/run.py`、`blog-audio/scripts/run.py`、`blog-google/scripts/run.py` 都有独立的脚本目录，每个 skill 把复杂逻辑封装在 Python 脚本里。这是"NL 指令调度、代码执行"的正确分层。  
   → 原文路径：`skills/blog-{notebooklm,audio,google}/scripts/run.py`  
   → 借鉴：把复杂的数据处理逻辑从 NL 文件中剥离，放入 skill 目录下的 scripts/ 子目录

2. **术语统一性**：`answer-first formatting`、`citation capsule`、`information gain marker`、`TL;DR box` 这些术语在 blog-write、blog-rewrite、blog-analyze、blog-geo 和 4 个 agent 中使用完全一致，无术语漂移。  
   → 借鉴：建立一个内部词汇表（可以是 CLAUDE.md 的一节），用统一术语规范跨件引用

3. **版本一致性管理**：`plugin.json` 和 `skills/blog/SKILL.md` 都报告 version `1.6.9`，保持同步。子技能有独立版本（如 blog-image `1.4.0`）这也是合理设计。  
   → 借鉴：主 skill 的版本和 plugin.json 保持同步，子组件版本可以独立演化

4. **精细到字段的 bug 描述**：审计报告指出 `blog-write/SKILL.md` 在第 161 行和第 165 行引用了 `cta-placement.md`，这种精确到行号的跨件引用追踪，说明充分的文档化使 bug 可精确定位。  
   → 借鉴：在 skill 文件中引用外部 reference 时，用统一格式标注（如 `<!-- ref: references/xxx.md -->`），便于静态分析

### 2.3 当时的缺陷

1. **全部 4 个 agent 无 model 声明**  
   → 根本原因：作者可能认为默认模型足够好，或者觉得 model 声明是可选的。实际上无 model 声明的 agent 会继承调用者的当前模型，在升级或切换默认模型时行为改变，难以重现和调试  
   → 自查：我的每个 agent 文件的 frontmatter 中是否有 `model:` 字段？

2. **全部 4 个 agent 无 examples 块**  
   → 根本原因：examples 是最费力气的部分（需要构造具体输入输出）。author 可能认为详细的 workflow 描述已经足够，但缺少具体例子时，Claude 对边界情况的处理会退化到"我猜你想要什么"  
   → 自查：我的 agent 文件是否至少有一个 `<example>` 块，包含 input 和 expected output？

3. **`blog-write/SKILL.md` 跨件引用缺失**（Bug #1）  
   → 根本原因：在 skill 迭代过程中，写了一个 `references/cta-placement.md` 的引用，但忘记创建或提交该文件。没有静态分析工具来检测 "文件引用但文件不存在" 这类错误  
   → 自查：我的 skill 文件中所有 `references/xxx.md` 引用的文件是否都真实存在于仓库中？

4. **vague 量词在 skill 中普遍存在**（appropriate、relevant、suitable 共出现数十次）  
   → 根本原因：这些词是英文写作中最自然的形容词，作者在写 skill 时处于"说明文档"模式而非"可执行规范"模式，没有意识到这些词在 AI 指令中会产生模糊执行  
   → 自查：我的 skill 文件中有没有 appropriate / relevant / suitable / comprehensive / robust 等词，以及它们是否可以替换为有具体阈值或条件的描述？

### 2.4 当时的优化机会

1. **为 4 个 agent 补 model 声明和 examples**：按照每 agent 加 `model: claude-sonnet-4-6` 和 2 个 `<example>` 块的标准，4 个 agent 大约需要 2-3 小时全量补全

2. **把 vague 量词系统化替换**：`appropriate` → 给出具体标准（如字数范围、内容类型枚举）；`relevant` → 给出相关性定义（如"与目标关键词语义相似度 >0.8"）；`suitable` → 给出选择条件

3. **补全 `references/cta-placement.md`**：从 `blog-write/SKILL.md` 中关于 CTA 的上下文出发，构建一个具体的 CTA 放置规范文档

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 4 个 agent 无 model 声明 | grep frontmatter 中 model 字段 | **无法验证**（API 限速） | 暂缺 |
| 4 个 agent 无 examples 块 | grep `<example>` | **无法验证** | 暂缺 |
| blog-write 引用 cta-placement.md 但文件缺失 | 读 references/ 目录 | **无法验证** | 暂缺 |
| install.sh curl-pipe-sh（CRITICAL） | 读 CLAUDE.md 第 119 行 | **无法验证** | 暂缺 |

> 注：GitHub API 限速，无认证 token，无法实时读取目标仓库。状态基于 2026-04-06 审计快照。

### 3.2 架构演进

Registry 显示该仓库状态为 `audited`，未进入 `contributed` 状态（CRITICAL 安全问题阻断了 NLPM 的 PR 提交）。从 2026-04-06 至今，外部无 NLPM 驱动的变更。

### 3.3 新增的可学习模式

暂无（live 检查不可用）。

---

## 四、校准

### 4.1 我已经在做对的

1. **model 声明**：我的每个 agent 文件都有显式的 `model:` 字段
2. **examples 块**：我在每个 agent 文件中至少有 2 个 `<example>` 块，覆盖典型和边界场景
3. **术语一致性**：我已在 CLAUDE.md 中建立了内部词汇表，用于规范跨件术语
4. **vague 量词意识**：我在写 skill 时会主动问"这个词可以测量吗？"——已有意识但需要持续检查

### 4.2 挑战 / 验证

**验证**：这次 audit 验证了我对"examples 是最低成本高收益投资"的判断。4 个 agent 因为缺少 examples 平均损失了 15 分（25% 的可用分），而添加 examples 并不需要设计新功能，只是把 agent 的预期行为具体化。这是最值得优先修复的 gap。

**挑战**：我之前觉得"vague 量词只是轻微扣分（-2 到 -4 分）"，这次案例让我重新计算：blog 系列 skill 中 vague 量词累计扣分超过 40 分（分散在多个 skill 中），如果这些 skill 是同一个评分对象，总扣分就很可观了。规模化时 vague 量词的代价被放大。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查所有 agent 文件是否有 model 声明
find . -path "*/agents/*.md" | xargs grep -L "^model:" 2>/dev/null
# 命中后：在 YAML frontmatter 中添加 `model: claude-sonnet-4-6`（或其他意图模型）

# 检查所有 agent 文件是否有 examples 块
find . -path "*/agents/*.md" | xargs grep -L "<example>" 2>/dev/null
# 命中后：至少添加一个 <example> 块，包含具体的 input 描述和 expected output

# 批量扫描 skill 文件中的 vague 量词
grep -rn -E '\b(appropriate|relevant|suitable|comprehensive|robust|efficient|significant|major|valuable|important)\b' \
  $(find . -name "SKILL.md") 2>/dev/null | grep -v "^Binary"
# 命中后：逐条评估是否可替换为可验证的标准（数字范围、枚举列表、条件判断）
```

```bash
# 检查 skill 中的 reference 引用是否都有对应文件
for skill in $(find . -name "SKILL.md"); do
  dir=$(dirname "$skill")
  # 提取形如 references/xxx.md 的引用
  refs=$(grep -oP 'references/[a-zA-Z0-9_.-]+\.md' "$skill" 2>/dev/null | sort -u)
  for ref in $refs; do
    if [ ! -f "$dir/$ref" ] && [ ! -f "./$ref" ]; then
      echo "MISSING: $skill → $ref"
    fi
  done
done
# 命中后：创建缺失的 reference 文件，或移除失效引用
```

### 5.2 灵感 → 实施路径

1. **想法**：实现一个静态检查脚本，检测 skill 文件中所有 `references/xxx.md` 引用是否存在对应文件  
   **为何可行**：纯文件系统操作，不需要解析 markdown AST，20 行 bash 即可完成  
   **第一步**：把上方的 reference 检查脚本集成到 `bin/nlpm-check` 或 pre-commit hook；预计 30 分钟

2. **想法**：构建一个 examples 生成辅助 workflow：给定 agent 定义，让 Claude 自动生成 2 个典型 example 块  
   **为何可行**：Claude 理解 agent 定义，可以从 workflow 描述反推示例  
   **第一步**：在 CLAUDE.md 里写一个"如何为 agent 补 examples"的操作规程，然后手动为 blog 系列 agent 执行一次；这样既补了缺口，又验证了操作规程是否可复用
