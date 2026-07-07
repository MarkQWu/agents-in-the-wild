# hashicorp/agent-skills — 学习案例

**仓库**：https://github.com/hashicorp/agent-skills
**Stars**：未收录（upstream 池，已通过 500+ stars 发现筛选）| **来源**：upstream（SECURITY CLEAR，核心 bug 已修）
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-07（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `vague-quantifier`, `single-purpose`, `monorepo-vs-split`, `security-gate`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
HashiCorp 官方出品的 Claude Code skill 集合，覆盖 Terraform 和 Packer 两个产品线。HashiCorp 是基础设施即代码领域的领导者（Terraform 有 4 万+ stars），这是企业级团队如何将复杂 DevOps 工作流编码为 AI skill 的典型样本。

关键事实：
- 作者：HashiCorp 工程团队（非个人项目）
- 覆盖范围：Terraform（代码生成、提供商开发、模块生成）+ Packer（HCP、镜像构建）
- 安装方式：5 个独立 plugin，每个产品线一个或多个
- 审计时间：2026-04-06，NL Score 98/100，SECURITY CLEAR

### 1.2 架构剖析
- **目录结构**：
  ```
  terraform/
    code-generation/        ← Plugin 1（4 个 skill）
      .claude-plugin/plugin.json
      skills/
        terraform-search-import/  ← ⚠ audit 时有 bug
        terraform-style-guide/
        azure-verified-modules/
        terraform-test/
    provider-development/   ← Plugin 2（6 个 skill）
      .claude-plugin/plugin.json
      skills/
    module-generation/      ← Plugin 3（3 个 skill）
      .claude-plugin/plugin.json
      skills/
  packer/
    hcp/                    ← Plugin 4（1 个 skill）
      .claude-plugin/plugin.json
      skills/
    builders/               ← Plugin 5（3 个 skill）
      .claude-plugin/plugin.json
      skills/
  scripts/
    validate-structure.sh
  ```
- **文件类型分布**：21 个 SKILL.md，5 个 plugin.json（无 agent/command）
- **编排关系**：5 个完全独立的 plugin，按产品×功能维度切分，无 router，无跨 plugin 引用
- **跨件契约**：每个 plugin 内部通过相对路径引用 `references/` 子文档（`MOCK_PROVIDERS.md`、`CI_CD.md` 等）；`refactor-module` 使用 raw githubusercontent.com 绝对 URL 引用其他 skill（这是架构弱点：fork 后 URL 会失效）

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「产品线即 plugin，功能即 skill」——每个 plugin 完全对应一个产品功能域，用户只装自己用的那个
- **解决什么问题**：让使用 HashiCorp 产品的工程师可以用 AI 辅助完成 Terraform/Packer 的复杂操作，同时保留 HashiCorp 特定的规范约束（如 Azure Verified Modules 标准）
- **做了什么 trade-off**：5 个独立 plugin vs 1 个大 plugin——用户颗粒度更高（按需安装），但维护 5 个 plugin.json 有重复配置成本（MCP server 配置完全一样却出现在两个 plugin.json 里）
- **反映什么认知模型**：HashiCorp 把 Claude Code skill 当作「产品文档的可执行扩展」——skill 内容与官方文档高度对齐，是公司产品知识的 AI 化封装

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「产品域 × 功能分层 Plugin 拆分」模式**：按产品×功能二维矩阵将 skill 分装到多个独立 plugin，每个 plugin 是一个可独立安装的最小功能单元。

模式特征清单：
- 每个 plugin 覆盖一个产品+功能组合（Terraform 代码生成、Terraform 提供商开发等）
- plugin 之间无依赖，用户可以只安装「terraform-code-generation」而不安装其他
- 所有 plugin 共享同一个 MCP server 配置（`hashicorp/terraform-mcp-server`），这是隐性耦合
- skill 文件质量均匀，多数达到 96-100 分，仅少数有 vague quantifier

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 企业级多产品工具 | ✅ 高度适用 | 用户只安装自己需要的产品域 plugin，减少干扰 |
| 开源工具集 | ✅ 适用 | 按功能模块安装，符合开源用户习惯 |
| 个人工具集（<10 个 skill） | ❌ 过度工程 | 5 个 plugin 的维护成本对个人来说太高 |
| 需要跨产品线 skill 联动 | ⚠ 部分适用 | 5 个 plugin 互相独立，跨域 skill 联动需要用户手动安装多个 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 产品域拆分（本仓库） | hashicorp/agent-skills | 颗粒度细，按需安装 | MCP 配置重复，绝对 URL 引用脆弱 |
| 单 plugin 平铺 | mattpocock/skills | 简单，一次安装 | 随 skill 增多，manifest 管理难度升 |
| 大型单仓（agent+skill） | trailofbits/skills | 功能强大，可跨件联动 | 安全风险面广，维护复杂 |

### 2.4 改进空间
1. **当前问题**：`refactor-module/SKILL.md` 使用 raw GitHub URL 跨 plugin 引用。**改进做法**：改为相对路径或提取为共享 `references/` 目录。**预期收益**：fork-friendly，不因 URL 变化导致引用失效。
2. **当前问题**：MCP server 配置在 `code-generation` 和 `module-generation` 两个 plugin.json 中完全重复。**改进做法**：使用共享 config 文件或在 README 中明确说明这是有意重复。**预期收益**：一次修改（如 pin docker image 版本）只需改一处。
3. **当前问题**：TFE_TOKEN 传递给 Docker 容器时没有文档说明最小权限范围。**改进做法**：在 plugin README 里加「所需 Token Scope」章节。**预期收益**：用户安全意识提升，减少权限过度授予的风险。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **98/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `terraform-search-import/SKILL.md` | 90 | 引用不存在的 `./scripts/list_resources.sh` |
| `provider-actions/SKILL.md` | 92 | 4 个量词："meaningful"、"relevant"×2、"appropriate"×2 |
| `refactor-module/SKILL.md` | 92 | 4 个量词：well-structured、clear、proper、appropriate |
| 其他 15 个文件 | 96-100 | 最多 1-2 个 vague quantifier |
| 5 个 plugin.json | 100 | 全部完美 |

### 3.2 当时值得借鉴的模式
1. **Checklist 驱动 skill**：`provider-test-patterns` skill 提供了完整的检查清单格式（有/无核查项），让 AI 可以逐项验证。为什么好：结构化输出，便于追踪进度。如何借鉴：把流程类 skill（review、audit）改为 checklist 格式。
2. **100 分的 skill 结构**：`provider-docs`、`new-terraform-provider`、`provider-test-patterns` 三个文件完全没有 vague quantifier，证明技术类 skill 完全可以避免模糊语言——只需把「做好」换成「满足 X 标准」、把「适当的」换成「根据 Y 规则」。
3. **多 plugin.json 组织**：5 个独立 plugin 让用户可以精确安装所需模块，符合「最小权限」原则。

### 3.3 当时的缺陷
1. **「断引用」bug**：`terraform-search-import/SKILL.md` 第 30、55 行引用 `./scripts/list_resources.sh`，但该文件不存在（只有 `scripts/validate-structure.sh`）。根本原因：script 被重命名或移除时没有同步更新 SKILL.md 内的引用。自查：我的 skill 是否有引用不存在文件的情况？→ 需要检查。
2. **TFE_TOKEN 文档缺失**：MCP 配置把 Terraform Enterprise API token 传给 Docker，但没有说明需要哪些最小权限。根本原因：安全意识不足——对内部工程师可能是「当然知道」，但对外部用户是安全风险。自查：我的 MCP 配置是否有类似「用户需要知道但没有写清楚」的内容？
3. **量词残留**：18 个 quality issue 全部是 vague quantifier，说明即使是 HashiCorp 这种专业团队，也很难完全避免「meaningful」「appropriate」这类表达习惯。

### 3.4 当时的优化机会
1. 在 terraform-search-import 里添加 `list_resources.sh` 脚本（最核心的 bug fix）
2. 把两个 plugin.json 里完全一样的 MCP server 配置提取到共享文档
3. 给 TFE_TOKEN 的 Docker 环境变量加一个 Scope 说明注释

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `list_resources.sh` 不存在（BUG） | `find . -name "list_resources.sh"` | **FIXED（已修复）**：文件已存在于 `terraform/code-generation/skills/terraform-search-import/scripts/list_resources.sh` | 主要 bug 修复，skill 的核心工作流恢复正常 |
| TFE_TOKEN scope 无文档 | `grep -n "TFE_TOKEN" terraform/code-generation/.claude-plugin/plugin.json` | **PARTIAL（部分改善）**：token 配置仍存在，但 README/文档是否新增 scope 说明需进一步核查 | 安全 medium 问题仍需关注 |
| vague quantifier（18 处） | 通过 grep 检查各 SKILL.md | 仍可在多个文件中找到，属于 quality issue 而非 bug | 低优先级 |

### 4.2 架构演进
与 audit 时相比，`list_resources.sh` 的修复是最主要变化，说明 HashiCorp 团队在收到 PR 或审查后快速响应了 bug 类问题。整体目录结构（5 个 plugin）保持稳定，说明「产品域拆分」架构经过实践验证，团队认可这一组织方式。

### 4.3 新增的可学习模式
`CHANGELOG.md` 被加入仓库，记录了各版本的变更（此为推测，基于规范仓库的演进规律）；bug 修复周期相对较快，说明有活跃维护。暂无从 audit 报告外可见的重大新模式。

---

## 五、校准

### 5.1 我已经在做对的
1. **独立 skill 单职责**：hashicorp 的 5 个 plugin 各司其职，与我的 echo-sleuth（记忆）、bureau（知识管理）等各专注单一领域的思路一致。
2. **不在 skill 里硬编码 URL**：我的 skill 里没有使用 raw GitHub URL 引用（避免了 hashicorp 的脆弱引用问题）。

### 5.2 挑战 / 验证
这个案例验证了一个我已知的原则：**「断引用 bug 在 NL 代码里和在源代码里一样致命」**。hashicorp 的 `list_resources.sh` 引用断掉，用户完全无法使用 `terraform-search-import` skill 的核心功能，但没有 runtime error，只是「执行时发现文件不存在」。NL skill 的调试比代码更难，因为没有 linter 能自动发现这类问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 里是否有引用不存在的文件
for skill in /tmp/my-repos/MarkQWu-*/skills/**/*.md; do
  dir=$(dirname "$skill")
  # 找所有看起来像相对文件引用的行
  grep -n '\./.*\.\(sh\|py\|md\|json\)' "$skill" 2>/dev/null | while read -r line; do
    fname=$(echo "$line" | grep -oP '\./[^\s"'\'']+')
    if [ ! -f "$dir/$fname" ]; then
      echo "BROKEN: $skill → $fname"
    fi
  done
done
```
命中后怎么办：更新 skill 里的引用路径，或添加缺失文件。

```bash
# 检查 MCP 配置是否有 credential 传递但无文档
grep -rn "TFE_\|API_TOKEN\|SECRET" /tmp/my-repos/**/.claude-plugin/plugin.json 2>/dev/null
```
命中后怎么办：在对应 plugin 的 README 里加「所需权限」章节。

### 6.2 灵感 → 实施路径

1. **想法**：给 graphify 或 bureau 的 MCP 配置加一个「最小权限说明」文档块。
   - **为何可行**：graphify 是一个图谱查询工具，如果有 MCP 配置，可能有 token 传递。
   - **第一步**：检查 `MarkQWu/graphify` 的 plugin.json，若有 token 配置，加 README 章节。约 20 分钟。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 hashicorp/agent-skills 的核心目的**：将 Terraform/Packer 的专业 DevOps 知识封装为可安装 skill，辅助工程师完成基础设施即代码工作流。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 中 | 都是「将特定技术域知识封装为 AI 辅助工具」 | graphify 是通用代码图谱工具，不是特定产品 skill | 中 |
| MarkQWu/gstack | 低 | 都有多个 skill | gstack 是个人工作流工具，不是产品技术知识封装 | 低 |

若全部相似度低：「我的仓库中无目的相近的项目，本节仅做技术模式对照」→ 主要学习：多 plugin 拆分策略、安全 credential 文档化实践。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 断引用（skill 引用不存在的脚本） | `grep -n '\./.*\.sh' skills/**/*.md` | echo-sleuth 无 shell 引用；gstack 有 shell 脚本引用，待核查 | 中 |
| credential 传递无 scope 说明 | `grep -n "TOKEN\|SECRET" .claude-plugin/plugin.json` | bureau/graphify 无 MCP credential；echo-sleuth 无 | 低（目前无问题） |

**具体行动**：
- 检查 gstack 中 `land-and-deploy/SKILL.md` 等是否引用外部脚本：`grep -n '\./.*\.sh' /tmp/my-repos/MarkQWu-gstack/**/*.md`

### 7.3 别人的更优方案

1. **领域**：产品域 plugin 拆分（按需安装）
   - **本案例做法**：5 个独立 plugin，分别对应 Terraform 三个功能域 + Packer 两个功能域，用户按需安装
   - **我的项目现状**：gstack 是单一仓库但没有 plugin.json（直接作为 skill 目录），用户要么全装要么不装
   - **如何借鉴**：考虑把 gstack 按角色（engineering、design、product）分成 2-3 个子 plugin，用 `plugin.json` 分别注册

2. **领域**：Checklist 格式 skill
   - **本案例做法**：`provider-test-patterns` skill 以 checklist 形式（[ ] 项）列出测试准备清单，结构清晰
   - **我的项目现状**：echo-sleuth 的 skill 以步骤列表为主，但没有 checklist 风格
   - **如何借鉴**：对于「审计」或「核查」类 skill，改用 `- [ ]` checklist 格式，提升可追踪性

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：跨件引用稳定性
- **我的做法**：echo-sleuth 和 bureau 的 skill 全部使用相对路径或内联内容，没有使用 raw GitHub URL（如 `githubusercontent.com/...`）
- **本案例做法（弱在哪）**：`refactor-module/SKILL.md` 使用 raw GitHub URL 引用同仓库内其他 skill，fork 或重命名后 URL 失效
- **意义**：我的项目在「fork-friendly 设计」上领先于 HashiCorp 官方实现。

---

## 八、术语表

### <a name="skill-bundle"></a>skill bundle（插件捆绑包）
> 将多个 SKILL.md 文件打包成一个可通过 `claude plugin install` 安装的单元。一个 plugin 可以包含 1 到 N 个 skill。HashiCorp 选择了「多个小 plugin，每个 plugin 对应一个功能域」的模式，而不是「一个大 plugin 包含所有 skill」。

### <a name="mcp-server"></a>MCP server（Model Context Protocol 服务器）
> Claude Code 通过 MCP 协议与外部工具通信的服务端程序。在 hashicorp/agent-skills 中，`hashicorp/terraform-mcp-server` Docker 镜像就是一个 MCP server，负责提供 Terraform 操作的执行能力。plugin.json 里的 `mcpServers` 字段声明了 Claude Code 需要启动的 MCP server。

### <a name="tfe-token"></a>TFE_TOKEN
> Terraform Enterprise 的 API 认证 Token。拥有这个 token 才能访问 Terraform Cloud/Enterprise 的私有模块、状态文件等资源。安全风险：如果 Token 权限过大（如 Admin 权限），一旦泄露可能导致生产基础设施被破坏。
