# AgriciDaniel/claude-blog — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-blog
**Stars**：562 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-20（基于当前 HEAD）
**主题标签**：`security-gate`, `cross-reference`, `template-design`, `fallback-chain`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Code 博客创作与优化技能，「Tier 4」复杂技能，同一作者（AgriciDaniel）在 claude-ads 之前发布的作品。28 个子技能 + 5 个专项代理 + 12 个内容模板，双重优化目标：Google 排名（2025 年 12 月 Core Update，E-E-A-T）和 AI 引用（GEO/AEO）。版本 1.7.0（Pro Hub Challenge 社区版）。

关键事实：
1. NLPM 评分 92/100，Security 状态 **BLOCKED**（audit 时）——高 NL 质量 + 安全问题并存
2. 包含 FLOW 框架集成（`scripts/sync_flow.py` 同步外部 FLOW 参考文件）
3. 采用 pyproject.toml 现代 Python 打包，是少见的把 Python 包管理带入 Claude Code 插件的案例
4. 相比 audit 时新增了 blog-translator 代理（第 5 个代理）和 blog-localize 子技能
5. `.mcp.json` 已从直接执行配置改为 `.mcp.example.json` 模板文件，是重大安全修复

### 1.2 架构剖析
- **目录结构**：
```
claude-blog/
  .claude-plugin/
    plugin.json / marketplace.json
  .mcp.example.json      # MCP 配置模板（.mcp.json 被 gitignore）
  pyproject.toml         # Python 包管理（3.11+）
  skills/
    blog/                # 主入口编排技能
      SKILL.md           # 路由表 + 全局规则
      references/        # 14 个按需知识文件
      templates/         # 12 个内容模板
      scripts/           # Python 分析脚本
    blog-write/SKILL.md  # 从头写文章
    blog-rewrite/SKILL.md # 优化现有博文
    blog-analyze/SKILL.md # 100 分评分
    blog-seo-check/SKILL.md
    blog-localize/SKILL.md # 新增：多语言发布
    ... (共 28 个)
  agents/
    blog-researcher.md   # 研究代理（已修复 name/model）
    blog-writer.md       # 写作代理
    blog-reviewer.md     # 审查代理
    blog-seo.md          # SEO 代理
    blog-translator.md   # 翻译代理（新增）
  scripts/
    sync_flow.py         # FLOW 框架同步
```
- **文件类型分布**：28 个 skill，5 个 agent，1 个 [manifest](#manifest)，35 个 Python 脚本，1 个 MCP 配置模板
- **编排关系**：`blog/SKILL.md` 路由 → 专项子技能 → 可选启动代理并行执行；Python 脚本通过 `skills/*/scripts/run.py` 统一调用（dispatcher 模式）
- **跨件契约**：`blog-write/SKILL.md` 通过路径引用 `references/cta-placement.md`（该文件 audit 时可能不存在）；`blog-analyze` 提供的 100 分评分框架被 blog-write 和 blog-rewrite 交叉引用作为质量目标

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「答案优先格式 + AI 引用优化」——文章结构不仅要让人读懂，还要让 AI 系统能引用（GEO/AEO 策略），这是 2025-2026 年 SEO 进化的体现
- **解决什么问题**：博客写作工作流碎片化：选题 → 研究 → 大纲 → 写作 → SEO 优化 → 图片生成 → 多平台发布，每个环节都有独立工具，这个插件把全流程整合到一个技能包里
- **做了什么 trade-off**：集成 nanobanana-mcp（Google Gemini 图像生成）给功能加了很大价值，但引入了外部依赖（npm 包）和安全风险。解法是把 .mcp.json 改为 .mcp.example.json——用户自己复制并配置，而非自动执行
- **反映什么认知模型**：作者把技能文件视为「专业编辑团队的工作手册」——不只是提示词，而是带有评分体系（100 分量表）、样式指南（6 大优化支柱）、参考文献列表的完整作业规范

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「Router + 代理池 + 脚本执行层」（Router-Agent-Script Three Tier）**

在 Router + Channels 基础上，额外引入了「代理池」（可调用的专项代理）和「脚本执行层」（Python 脚本处理无法纯靠 LLM 的操作，如 SEO 分析、图片下载）。

模式特征清单（4 条）：
- 特征 1：三层分离：NL 路由层 → 代理编排层 → Python 执行层
- 特征 2：统一 dispatcher（`scripts/run.py`）作为脚本调用入口，避免路径散乱
- 特征 3：MCP 配置模板化（.mcp.example.json）而非直接提交，敏感配置不进仓库
- 特征 4：评分体系内嵌（blog-analyze 的 100 分量表）作为质量约束跨技能引用

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要 LLM + 外部 API（图像生成、SEO 分析）的混合工作流 | ✅ 高度适用 | 脚本执行层正是为此设计 |
| 需要 MCP 工具但不想把 API 密钥暴露给仓库读者 | ✅ 适用 | .mcp.example.json 模式可复用 |
| 纯 LLM 推理的技能 | ❌ 过度设计 | 不需要 Python 脚本层和 MCP 层 |
| 小型个人工具（<5 个子技能） | ❌ 过度设计 | 维护成本和复杂度不成比例 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Router + 代理池 + 脚本层（本仓库） | claude-blog | 功能完整，LLM+代码混合，MCP 安全配置 | 最高维护成本，Python 环境依赖 |
| Router + Channels 纯 NL（同作者） | claude-ads | 无运行时依赖，部署简单 | 无法调 OS/网络 API |
| 单仓多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置 | 无工作流编排 |

### 2.4 改进空间
1. **当前问题**：blog-notebooklm、blog-audio、blog-google 下的 `run.py` 脚本通过 `sys.argv[1]` 接收脚本名称并直接构造路径，虽然有 `.py` 后缀和 `scripts/` 前缀的过滤，但没有 `basename()` 处理，理论上仍存在路径遍历风险。**改进做法**：在构造路径前加 `script_name = os.path.basename(script_name)` 一行。**预期收益**：完全消除路径遍历向量，代码更健壮。
2. **当前问题**：agents/ 下的代理文件仍缺少 `model:` frontmatter 字段（blog-researcher 有 name 和 tools，但无 model）。**改进做法**：在所有代理 frontmatter 里加 `model: claude-sonnet-4-20250514`。**预期收益**：NLPM 评分从当前各代理的 72-80 跳升 5 分，确保模型一致性。
3. **当前问题**：CLAUDE.md 第 14 行仍写 `.mcp.json` 但实际文件是 `.mcp.example.json`。**改进做法**：更新 CLAUDE.md 描述，说明 `.mcp.json` 已 gitignore，用户需从 `.mcp.example.json` 复制。**预期收益**：文档和实际结构一致，避免混淆。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**，Security 状态：**BLOCKED**（Critical + High 安全发现）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/blog-researcher.md | 72 | 无模型声明，无示例，模糊量词（suitable×2, appropriate×2） |
| agents/blog-writer.md | 76 | 无模型声明，无示例，模糊量词 |
| agents/blog-reviewer.md | 77 | 无模型声明，无示例，Bash 声明但未使用 |
| agents/blog-seo.md | 80 | 无模型声明，无示例 |
| skills/blog-write/SKILL.md | 88 | 模糊量词 6 个（appropriate, relevant, suitable） |
| skills/blog-strategy/SKILL.md | 92 | 模糊量词 4 个 |
| 大多数 skills/ | 93-98 | 少量模糊量词 |

### 3.2 当时值得借鉴的模式
1. **内嵌评分体系**（template-design）→ 为什么好：`blog-analyze/SKILL.md` 的 100 分评分体系（5 类 × 20 分）被其他技能交叉引用作为质量目标，创造了技能间的「契约」——写作必须符合分析的评分标准 → 原文路径：`skills/blog-analyze/SKILL.md` → 如何借鉴：给自己的技能组设计一个内部评分标准，让各子技能都以达到这个标准为目标
2. **answers-first + citation capsule 一致性术语**（cross-reference）→ 为什么好：「answer-first formatting」、「citation capsule」、「information gain marker」等专有术语在 blog-write、blog-rewrite、blog-analyze、blog-geo 和 4 个代理里保持一致，零术语漂移 → 原文路径：多文件 → 如何借鉴：在项目开始时建立一个「术语表」，所有文件引用同一个定义，不要在不同文件里用不同的词表达同一概念
3. **pyproject.toml 现代 Python 包管理**（offline-capable）→ 为什么好：用 pyproject.toml 而非 requirements.txt-only 方式管理 Python 依赖，支持版本约束和依赖解析，适合长期维护 → 原文路径：`pyproject.toml` → 如何借鉴：如果插件需要 Python 脚本，从一开始就用 pyproject.toml + requirements.lock

### 3.3 当时的缺陷
1. **`.mcp.json` 自动执行 `npx -y @ycse/nanobanana-mcp`**（High 安全）：`.mcp.json` 里的 `npx -y` 标志在 MCP 服务器每次启动时自动下载执行 npm 包，无任何用户确认。为什么会失败：`@ycse/nanobanana-mcp` 如果被同名恶意包替换（typosquatting），会在 Claude Code 进程上下文里静默执行恶意代码，且 `-y` 绕过了 npm 的确认提示。**自查**：我的配置文件里有没有 `npx -y` 或等价的「自动执行」模式？
2. **4 个代理无 `model:` 声明**：模型版本游离，每次升级 Claude Code 默认模型都可能改变代理行为。为什么会失败：不固定模型意味着代理的输出风格和能力可能随工具版本隐性变化，难以复现和调试。**自查**：我的代理文件 frontmatter 里是否都有 `model: claude-sonnet-4-20250514`（或当前最新版）？
3. **blog-write/SKILL.md 引用 `references/cta-placement.md` 但该文件可能不存在**：技能被调用时静默缺失 CTA 定位指导，写作代理会自行「发挥」。为什么会失败：引用了不存在的文件不会报错，Claude 会用训练数据填补空缺，结果不可预测。**自查**：我的技能文件里有没有路径引用？全部验证是否实际存在？

### 3.4 当时的优化机会
1. 给 blog-researcher、blog-writer、blog-reviewer、blog-seo 的 frontmatter 加上 `model:` 字段
2. 给 4 个代理各加一个 `<example>` 块（哪怕是最简单的输入/输出对）
3. `skills/blog-notebooklm/scripts/run.py` 等三处路径遍历：在构造路径前加 `os.path.basename()` 一行

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `.mcp.json` npx -y 自动执行 | `ls .mcp.json` + `cat .mcp.example.json` | **已修复**：`.mcp.json` 已从仓库移除（gitignore），替换为 `.mcp.example.json` 注释模板，`npx` 命令不再自动执行 | High 安全风险消除，MCP 配置模板化是最佳实践 |
| 代理无 `name` 字段 | `grep "^name:" agents/blog-researcher.md` | **已修复**：blog-researcher 等代理现已有 `name:` 字段 | 命令注册正常 |
| 代理无 `model:` 声明 | `grep "^model:" agents/blog-researcher.md` | **持续存在**：blog-researcher 有 name 和 tools 但仍无 model | 模型版本仍未固定，NLPM 评分代理项仍被扣 5 分 |

### 4.2 架构演进
从 audit 时 4 个代理增长到 5 个（新增 blog-translator），从 22 个技能增长到 28 个（新增 blog-localize、blog-audio 等），说明作者在向「多语言发布工作流」方向扩展——符合「Pro Hub Challenge 社区版」的国际化目标。最大的架构变化是 .mcp.json → .mcp.example.json，这是一个安全架构层面的成熟度提升，也说明作者意识到了把可执行配置提交到仓库的风险。

**作者后来意识到的**：MCP 配置不应该随仓库分发（因为包含 API 密钥），应该是用户本地配置文件；把模板（.example.json）和实际配置（.json，gitignored）分离是正确实践。

### 4.3 新增的可学习模式
**MCP 配置模板化**：`.mcp.example.json` 里的注释说明了模板用途，并用 `${GOOGLE_AI_API_KEY}` 环境变量占位符，提示用户「密钥从环境变量读，不要硬编码」。这是一个「配置即文档」（Config-as-Documentation）的模式——模板文件本身就是安全使用说明书。

---

## 五、校准

### 5.1 我已经在做对的
1. **MCP 配置模板化**：我不把 `.mcp.json`（含密钥的实际配置）提交到仓库，用 `.mcp.example.json` + `.gitignore` 的分离模式——和本仓库修复后的状态一致
2. **避免 `npx -y` 自动执行**：我的配置文件里没有 `-y` 这种绕过确认的标志
3. **术语一致性**：我在跨文件引用同一个概念时使用同一术语，不混用同义词
4. **代理有 name 字段**：我的代理 frontmatter 都有 `name:` 字段

### 5.2 挑战 / 验证
- **这次案例挑战了一个我之前模糊的假设**：「92/100 的技能 security 状态是什么？」答案令人意外：92/100 的 NL 质量评分完全无法反映安全状态，audit 时这个仓库是 Critical + High 安全风险 → BLOCKED，但 NL 评分极高。这说明 NLPM 的两个维度（NL 质量分 + Security 状态）是正交的，高 NL 评分不等于安全。**这强化了一个行动**：我在审查自己的插件时，必须把 NL 质量检查和安全扫描作为两个独立的检查清单，不能用一个替代另一个。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查是否有 .mcp.json 被提交到 git（应该在 .gitignore 里）
git ls-files .mcp.json 2>/dev/null && echo "WARNING: .mcp.json is tracked by git (should be gitignored)"
# 命中后：git rm --cached .mcp.json && echo ".mcp.json" >> .gitignore

# 检查 MCP 配置里是否有 npx -y 自动执行
grep -rn "npx.*-y\|\"args\".*\"-y\"" .mcp*.json 2>/dev/null
# 命中后：移除 -y 标志，或改为用户手动安装后再引用

# 检查 Python 脚本里的路径构造是否安全
grep -rn "Path.*argv\|sys\.argv.*Path\|path.*script_name" scripts/ skills/*/scripts/ 2>/dev/null
# 命中后：在路径构造前加 os.path.basename(sys.argv[1]) 去除目录分隔符

# 检查代理 frontmatter 有没有 model 字段
for f in agents/*.md; do
  grep -q "^model:" "$f" || echo "MISSING model: $f"
done
# 命中后：加 model: claude-sonnet-4-20250514 到每个代理 frontmatter
```

### 6.2 灵感 → 实施路径
1. **想法**：把 blog-analyze 的「内嵌 100 分量表跨技能引用」模式移植到自己的审查类技能组
   - **为何可行**：本仓库的评分体系让写作技能（blog-write）有了明确的质量目标，而不是模糊的「写一篇好文章」；跨技能契约让整个技能组形成闭环
   - **第一步**：创建一个 `quality-checklist/SKILL.md`，定义 5 个维度各 20 分的评分量表，然后在 write 和 review 类技能里引用这个文件；预计 2 小时
2. **想法**：仿照 `.mcp.example.json` 模式，给所有可能包含敏感配置的文件建立「模板/实例」分离规范
   - **为何可行**：本仓库的修复证明这是正确实践；我目前有几个配置文件可能存在同样问题
   - **第一步**：检查自己仓库里所有 `.json` 配置文件，对含有 API 密钥、webhook URL 等敏感信息的文件：`cp config.json config.example.json && sed -i 's/<key>/${ENV_VAR}/g' config.example.json && echo "config.json" >> .gitignore`；约 20 分钟

---

## 七、术语表

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 提出的开放协议，让 AI 模型能安全地调用外部工具服务器（MCP servers）。`.mcp.json` 是配置文件，定义了哪些 MCP server 可用、如何启动它们。本仓库用 MCP 连接 nanobanana-mcp 来调用 Google Gemini 图像生成 API。

### <a name="typosquatting"></a>typosquatting（域名抢注）
> 恶意方注册一个和合法包名高度相似（比如差一个字母）的包名，希望用户或系统错误安装恶意包。例如合法包 `@ycse/nanobanana-mcp`，攻击者可能注册 `@ycse/nanabanana-mcp`（调换两个字母）。配合 `npx -y`（自动安装不确认），一旦误安装就会在用户机器上执行恶意代码。

### <a name="GEO-AEO"></a>GEO / AEO（AI 引用优化）
> **GEO**（Generative Engine Optimization）：让文章内容被 ChatGPT、Perplexity 等 AI 引擎引用的优化策略；**AEO**（Answer Engine Optimization）：让内容成为 AI 问答引擎直接引用答案来源的优化策略。与传统 SEO（让 Google 爬虫收录）不同，GEO/AEO 面向 AI 摘要生成器，要求内容结构更像「可引用的段落」。

### <a name="E-E-A-T"></a>E-E-A-T
> Google 对内容质量的评判标准：**Experience**（经验）、**Expertise**（专业知识）、**Authoritativeness**（权威性）、**Trustworthiness**（可信度）。Google 2025 年 12 月 Core Update 对 E-E-A-T 评分权重进行了调整，AI 写作内容需要更明显的「人的经验」信号才能保持排名。
