# fcakyon/claude-codex-settings — 学习案例

**仓库**：https://github.com/fcakyon/claude-codex-settings
**Stars**：590 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-06-04（基于当前 HEAD）
**主题标签**：`security-gate`, `manifest-discipline`, `cross-reference`, `monorepo-vs-split`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
fcakyon/claude-codex-settings 是一个「多域名技能工厂」——单仓包含 25+ 个独立 Claude Code 插件，覆盖 GitHub 工作流、Stripe 支付、Cloudflare 部署、Supabase 数据库、Azure 云、GCP、MongoDB、LiveKit、Hetzner、React、Python 等技术栈。作者 Fatih Cakmak 是机器学习工程师，在 Claude Code 生态里维护这个「个人工具箱」型仓库。

关键事实：
- 25 个插件涵盖从 AI 工具到前后端框架的全栈技术
- 4 个 Hook 脚本（ultralytics-dev、github-dev、tavily-tools、claude-tools），其中 ultralytics-dev 有 High 安全漏洞
- 安全评级：**REVIEW**（有 High 安全发现，但不到 BLOCKED 级别）
- 103 个 NL 工件，NLPM 综合得分 79/100，主要拖分点是 supabase-cli 下的 10 个无 frontmatter 参考文档

### 1.2 架构剖析

**目录结构**：
```
plugins/
  github-dev/
    agents/ (pr-reviewer, pr-creator, pr-comment-resolver, commit-creator)
    hooks/ (git_commit_confirm.py, gh_pr_create_confirm.py)
    .claude-plugin/plugin.json
  ultralytics-dev/
    hooks/hooks.json              # ← High 安全漏洞所在
    hooks/scripts/ (python_code_quality.py, format_python_docstrings.py)
  supabase-skills/
    skills/supabase-cli/
      SKILL.md                    # 主 skill
      references/commands/       # 10 个无 frontmatter 的参考文档（各评 50/100）
  claude-tools/
    commands/ (sync-claude-md, load-claude-md, sync-allowlist, update-readme)
    hooks/scripts/sync_marketplace_to_plugins.py
  stripe-skills/, react-skills/, cloudflare-skills/ ...（其余 19 个插件）
.github/scripts/                  # 14 个维护脚本（开发者工具）
```

**文件类型分布**：30+ 个 SKILL.md / 8 个 command / 4 个 agent / 11 个 hook 脚本 / 5 个 MCP 配置文件

**编排关系**：多插件平列，每个插件独立自治。github-dev 内部有 4 个 agent，其中 pr-reviewer（审查）、pr-creator（创建）、commit-creator（提交）和 pr-comment-resolver（处理审查意见）构成 PR 工作流的四个分支，但没有统一的 orchestrator——用户按需触发对应 agent。

**跨件契约**：ultralytics-dev 的 hooks.json 通过 `${CLAUDE_PLUGIN_ROOT}/hooks/scripts/` 路径引用 Python 脚本，路径约定在所有 4 个 hook 插件里一致。supabase-cli 的 10 个 references/commands/ 文档本意是「参考资料」（不可直接触发），但因为放在 skills/ 目录下被 NLPM 当作 SKILL.md 工件评分，导致得分被严重拉低。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「单仓多技能，按需取用」——一个 repo 包含作者日常用到的所有技术栈的 AI 辅助工具，用户 `git clone` 一次就有全套配置。

**解决什么问题**：开发者切换技术栈时需要重新找对应的 Claude Code 配置；这个仓库把「各技术栈最佳实践」集中管理，既是个人工具箱，也是社区参考实现。

**做了什么 trade-off**：单仓覆盖广 vs 每插件质量不均——25 个插件里质量最高的 github-dev agents（88-95/100）和质量最低的 supabase-cli references（50/100）差距巨大；作者优先扩展覆盖面，对参差不齐的质量问题未作统一补全。

**反映什么认知模型**：「配置即代码，skill 是我的工具，不需要满足外部审查标准」——这个仓库更像个人的 `~/.config/`，不像要发布到市场的产品。这导致了 name 字段缺失、模糊量词、frontmatter 不完整等问题长期存在。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「个人全栈工具箱」单仓多插件模式**：一个 Git 仓库按技术栈分目录存放多个相互独立的插件，每个插件自治，用户可以选择性安装。

模式特征清单：
- 特征 1：每个插件放在 `plugins/<domain>/` 子目录，有独立的 `plugin.json`
- 特征 2：插件之间无 NL 级别的依赖（但 sync-allowlist 命令会拉取整个仓库内容）
- 特征 3：混合工件类型：有些插件只有 skill，有些有 skill + agent + hook + command
- 特征 4：有统一的维护脚本（`.github/scripts/`）管理多插件的同步和发布
- 特征 5：质量高度不均——精心设计的插件（github-dev）和快速堆砌的插件（supabase-cli references）共存

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 个人开发者管理自己的 AI 工具集 | ✅ 高度适用 | 便于版本管理和跨机器同步 |
| 团队共享 AI 工具集（多人 fork/使用） | ✅ 适用 | 单仓让用户一次安装全套 |
| 作为「各技术栈参考实现」对外发布 | ⚠️ 谨慎 | 质量参差不齐会影响整体可信度 |
| 频繁更新单个插件 | ❌ 不适用 | 单仓发布需要同步更新所有插件版本 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 单仓多插件（本仓库） | fcakyon/claude-codex-settings | 维护集中；用户单次 clone/install | 质量不均；单个插件发版影响整体 |
| 每插件独立仓库 | AgriciDaniel/claude-ads（单仓单插件） | 质量高度聚焦；独立版本发布 | 多仓管理成本高 |
| 单仓单插件（多 skill）| expo/skills | 结构清晰，单一焦点 | 无法覆盖多技术栈 |

### 2.4 改进空间

1. **当前问题**：supabase-cli/references/commands/ 的 10 个文档无 frontmatter，各评 50/100，拖累全仓得分约 5 分 **改进做法**：给每个文档加最小化 frontmatter（`name: supabase-cli-<cmd>`，`description: ...`）（15分钟批处理） **预期收益**：总分从 79 → ~84，消除最大的单一失分来源

2. **当前问题**：ultralytics-dev 的 hooks.json 用内联 bash 处理 file_path（High 安全漏洞）**改进做法**：把 hooks.json 的内联 bash 提取为 `trim_trailing_whitespace.py`（参照 `python_code_quality.py` 的做法），在 Python 里用 `pathlib.Path.resolve()` 验证路径 **预期收益**：消除 AI 工具输入流入 shell 操作的路径注入风险；安全评级从 REVIEW → CLEAR

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 79/100。

| 文件（代表） | 当时分数 | 主要问题 |
|---|---|---|
| supabase-cli references/commands/*.md (10 files) | 50 | 无 frontmatter block |
| setup.md（4个）、sync-allowlist.md、load-claude-md.md 等 | 70–75 | 缺 name；部分缺 allowed-tools |
| composition-patterns/SKILL.md | 82 | name 带 vendor 前缀；description 非触发格式 |
| github-dev agents（4个） | 88–95 | 轻微问题（未使用的 tool 声明，inline 示例格式） |

### 3.2 当时值得借鉴的模式

1. **github-dev 的四 agent PR 工作流** → pr-reviewer 审查 → pr-creator 创建 → commit-creator 提交 → pr-comment-resolver 处理意见；四个独立 agent 覆盖 PR 生命周期的不同阶段，但不强制按顺序——用户可以只用其中一个 → `github-dev/agents/`

2. **Hook 脚本与插件绑定（plugin-level hooks）** → ultralytics-dev 的 Python 代码质量钩子只在该插件激活时运行，不影响其他插件 → 「插件级钩子」vs「全局钩子」的正确分离

3. **`sync_marketplace_to_plugins.py` 的统一分发脚本** → 从中央 marketplace.json 同步到各插件目录，单点更新 + 批量分发 → `.github/scripts/sync_marketplace_to_plugins.py`

4. **`github-dev` 插件内 allowed-tools 声明更完整** → 4 个 agent 的工具声明清晰（即使有部分「潜在未使用」工具）

### 3.3 当时的缺陷

1. **ultralytics-dev hooks.json 内联 bash 处理用户/AI 控制的文件路径（High）** → AI 工具返回的 `file_path` 未经验证直接进入 shell `case` 和 `sed -i` 操作；精心构造的 file_path 可以触发意外的 sed 操作 → 根本原因：hooks.json 为了方便直接写 inline bash，跳过了「path validation → subprocess list form」这个安全惯例 → **自查：我的 hook 脚本是否有类似问题？** echo-sleuth 无 hook 脚本，暂无此风险

2. **`sync-allowlist.md` 里的 repo URL 写错**（`fcakyon/claude-settings` 而非 `fcakyon/claude-codex-settings`）→ 用户运行 `/claude-tools:sync-allowlist` 会从错误的 repo 拉取配置，静默行为但产生错误结果 → 根本原因：硬编码 URL 没有跟着仓库重命名更新 → **自查：我的 command 里是否有硬编码的 GitHub URL？**

3. **`composition-patterns` skill name 带 vendor 前缀（vercel-composition-patterns）** → skill name 耦合特定供应商，降低了在其他平台使用时的可读性 → 根本原因：从 Vercel 的 Next.js 最佳实践直接移植，没有去除供应商前缀

### 3.4 当时的优化机会

1. 给 10 个 supabase-cli references 文档加 frontmatter（最高 ROI，5分改进/每文件）
2. 提取 ultralytics-dev hooks.json 的内联 bash 为 Python 脚本
3. 修复 sync-allowlist.md 的 repo URL（已确认在当前 HEAD 中修复）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| supabase-cli references 无 frontmatter | `head -5 plugins/supabase-skills/skills/supabase-cli/references/commands/functions.md` | **仍存在**（文件以 `## supabase-functions` 开头，无 frontmatter） | Audit 后 2 个月，最简单的批量修复都未完成 |
| sync-allowlist.md 错误 repo URL | `grep "fcakyon/claude" plugins/claude-tools/commands/sync-allowlist.md` | **已修复** ✅（当前引用 `fcakyon/claude-codex-settings`，正确） | 这是 audit 中唯一被修复的 bug，说明作者只修了「会直接影响自己使用」的问题 |
| ultralytics-dev hooks.json 内联 bash（High） | `cat plugins/ultralytics-dev/hooks/hooks.json` | **仍存在**（hooks.json 第一个 hook 是内联 bash，处理 file_path） | High 安全漏洞 2 个月未修 |

### 4.2 架构演进

Audit 时：25 个插件，4 个 hook 插件，10 个 supabase references 无 frontmatter。当前 HEAD 变化：
- 新增插件：`intelligent-compact`、`claude-telemetry-hooks`、`dokploy-skills`、`anthropic-office-skills`、`openai-office-skills`、`overleaf-skills`、`polar-skills`、`openobserve-skills` 等（已从 audit 时的约 17 个增长到 25+ 个）
- 新增的插件大多是作者在新技术上的快速拓展，质量继续参差不齐

说明作者意识到：「横向扩展覆盖面」比「纵向提升已有插件质量」更符合这个仓库的定位（个人工具箱）。

### 4.3 新增的可学习模式

`intelligent-compact` 插件名称有意思——智能 context 压缩，说明作者在探索「在有限 context window 内做更多事」这个方向。这可能是一个新型的 meta-skill 类别（管理 AI 自身资源），值得关注这个方向的后续实现。

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth 命令的 `name` frontmatter 全部完整**：sync-allowlist 等命令全部缺 name，而我没有这个问题
2. **不在 hooks 脚本里用内联 bash 处理 AI 控制的路径**：echo-sleuth 和 claude-for-legal 目前没有 hook 脚本，规避了这类风险
3. **claude-for-legal 的 skill description 使用触发格式**：对比 fcakyon 的 stripe-skills/react-skills 用的命令式/描述式，我的格式更规范

### 5.2 挑战 / 验证

这次案例验证了一个重要认知：**「只修会直接影响自己的 bug」是开源维护者的普遍行为**——sync-allowlist 的 URL 错误被修了（因为作者自己用这个命令），而 supabase-cli frontmatter（影响 NLPM score 而非功能）2 个月没动。

对我的启示：NLPM bug 修复的优先级应该按「功能失效」→「用户体验降低」→「得分改进」排序，而不是按严重度等级排序。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 hook 脚本（如果有）是否用了内联 shell + 用户/AI 控制路径
find /tmp/my-repos -name "hooks.json" -exec cat {} \; 2>/dev/null | grep -n "command\|bash\|sh"
```
命中后：把内联 bash 提取为独立 Python 脚本，用 `pathlib.Path.resolve()` 验证路径。

```bash
# 检查我的 command 文件里是否有硬编码 GitHub URL
grep -rn "github.com/MarkQWu" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands/ 2>/dev/null
grep -rn "github.com/MarkQWu" /tmp/my-repos/MarkQWu-claude-for-legal 2>/dev/null | grep ".md" | head -10
```
命中后：验证 URL 是否与实际仓库名一致；如果仓库曾经重命名，需要更新所有引用。

### 6.2 灵感 → 实施路径

1. **想法**：给 claude-for-legal 的每个插件（ai-governance-legal、commercial-legal 等）各加一个 `setup.md` command，统一安装入口  
   **为何可行**：fcakyon 的 setup.md 模式（azure、gcloud、tavily、paper-search 都有 setup command）是一个好的用户引导模式  
   **第一步**：在 claude-for-legal 最常用的 `commercial-legal` 插件里加一个 `commands/setup.md`（15分钟）

2. **想法**：参照 fcakyon 的 github-dev PR 工作流，给 echo-sleuth 加一个 `github-audit.md` command，自动审查 PR 里的 NL artifact 变更  
   **为何可行**：echo-sleuth 的 git-mining skill 已经有 git 历史挖掘能力，结合 github-dev 的 PR 审查模式可以实现  
   **第一步**：用 pr-reviewer.md 为模板，写一个检查「PR 是否修改了 SKILL.md 且 frontmatter 是否完整」的 command（30分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 fcakyon/claude-codex-settings 的核心目的**：单仓集中管理跨技术栈的 AI 辅助工具，覆盖全栈开发日常所需
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 高 | 都是「领域内多插件/多技能集合」；都以单仓管理多个工作流场景 | fcakyon 是通用技术栈；claude-for-legal 是法律领域专属 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都是 Claude Code 插件 | 功能定位完全不同 | 低 |
| MarkQWu/drama-workshop-skills | 低 | 都是插件 | 不同领域 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill description 非触发格式 | `grep -n "^description:" /tmp/my-repos/MarkQWu-claude-for-legal/**/**/SKILL.md \| grep -v "should be used"` | 需手动验证；claude-for-legal 有部分 skill 可能用命令式描述 | 低 |
| reference 文档无 frontmatter | `find /tmp/my-repos/MarkQWu-claude-for-legal -name "*.md" -path "*/references/*" \| xargs head -3 2>/dev/null \| grep -L "^---"` | 需验证；claude-for-legal 的 references/ 文档是否有 frontmatter 未确认 | 中 |

**命中后的具体行动建议**：
- 如果 claude-for-legal 的 `references/` 目录下有无 frontmatter 的 Markdown 文件（类似 supabase-cli 的情况），要么给它们加最小化 frontmatter，要么把它们移出 `skills/` 目录以避免被 NLPM 当作 skill 评分

### 8.3 别人的更优方案

1. **领域**：github-dev 的四 agent PR 工作流
   - **本案例做法**：pr-reviewer、pr-creator、commit-creator、pr-comment-resolver 四个专职 agent，覆盖 PR 生命周期（审查→创建→提交→处理意见）
   - **我的项目现状**：echo-sleuth 没有 PR 相关工作流；claude-for-legal 也没有类似的代码工作流 agent
   - **如何借鉴**：如果将来 claude-for-legal 需要协作开发（PR 审查法律文档草稿），可以直接参照 github-dev 的 4-agent 模式

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：skill description 的触发格式规范性
- **我的做法**：echo-sleuth 的 skills 普遍使用 "This skill should be used when..." 或类似的触发格式；drama-workshop-skills 使用中文触发词列表，格式清晰
- **本案例做法**（弱在哪）：stripe-skills 的 `upgrade-stripe/SKILL.md` 用的是 "Guide for upgrading..."（命令式说明），`react-skills` 的 `composition-patterns/SKILL.md` 也类似；触发格式不规范影响 AI 的 skill 路由准确性
- **意义**：触发格式是 skill 能否被准确调用的关键——"This skill should be used when..." 告诉模型触发条件，命令式描述则不清晰；我在这个维度上做得更好

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明 name、description 等元数据。`plugins/supabase-skills/skills/supabase-cli/references/commands/*.md` 里的文件直接从 `## 标题` 开始，没有 frontmatter，所以 NLPM 给每个文件扣了 50 分。

### <a name="SRI"></a>SRI（Subresource Integrity，子资源完整性）
> 一种防止 CDN 被篡改的安全机制。在 `<script>` 标签里加 `integrity="sha384-..."` 属性，浏览器会验证加载的脚本文件哈希是否匹配，不匹配则拒绝执行。expo/skills 的 mermaid.js 加载和 fcakyon 的 dashboard 脚本都缺这个保护。

### <a name="path-injection"></a>路径注入
> 攻击者通过控制文件路径参数，让程序操作预期之外的文件。例如传入 `../../etc/passwd` 访问系统文件，或传入 `../important-file.txt` 覆盖关键配置。ultralytics-dev 的 hooks.json 把 Claude 工具返回的 `file_path` 直接传给 `sed -i` 运行，就存在这个风险。

### <a name="CORS"></a>CORS（Cross-Origin Resource Sharing，跨域资源共享）
> 浏览器的安全机制，控制网页是否能向不同域名的服务器发起请求。`Access-Control-Allow-Origin: *` 表示允许任意网页请求这个服务——这在本地 dashboard 里意味着任何打开的网页都可以读取你本地的代码历史数据（codetape 的安全问题同款）。
