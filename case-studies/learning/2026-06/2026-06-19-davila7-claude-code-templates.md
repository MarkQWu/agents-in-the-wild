# davila7/claude-code-templates — 学习案例

**仓库**：https://github.com/davila7/claude-code-templates
**Stars**：24685 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-19（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `template-design`, `cross-reference`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

davila7/claude-code-templates 是目前社区最大的 Claude Code [组件](#component)模板库之一，24685 stars 说明它命中了"我想要一个开箱即用的技能包"这个高需求。项目本质是两层架构：**① CLI 工具**（Node.js，`npm install -g claude-code-templates`），帮助用户挑选和安装组件；**② 静态网站**（GitHub Pages），提供可视化的组件浏览器。组件本身按领域分类：business-marketing（市场营销）、security（安全测试）、media（媒体处理）、workflow-automation、devops。作者是 Daniel Dávila，活跃的 Claude 社区贡献者。

### 1.2 架构剖析

**目录结构**（关键层级）：
```
claude-code-templates/
├── cli-tool/                         ← NPM 包核心
│   └── components/
│       ├── agents/
│       │   ├── devops/               ← 6 个 DevOps agent
│       │   │   ├── deploy-assistant/AGENT.md
│       │   │   ├── ci-cd-agent/AGENT.md
│       │   │   └── ...（共6个）
│       │   └── expert-advisors/
│       │       └── droid.md          ← 含第三方 curl-pipe-sh
│       └── skills/
│           ├── business-marketing/   ← 主力区，20+ skills
│           ├── security/             ← 含攻防技能（reverse shells）
│           ├── media/                ← 音视频处理
│           └── workflow-automation/
├── .github/workflows/                ← CI/CD 自动化
│   ├── skill-security-scan.yml       ← 每次 PR 触发安全扫描
│   └── component-security-validation.yml
├── database/                         ← Supabase 迁移文件
├── CLAUDE.md
└── SECURITY.md
```

**文件类型分布**：~100 个 SKILL.md，6 个 AGENT.md（DevOps 系），多个 hook 配置，7 个 MCP 配置文件，1079 个脚本文件（含安全技能中的攻防 payload）。

**编排关系**：平铺结构，无 router 层，用户通过 CLI 工具挑选并安装单个组件到自己项目。技能之间不互相调用，是"组件超市"而非"技能工作流"。

**跨件契约**：组件之间无依赖，最大跨件风险是 `brand-guidelines` 重名（同一 `name:` 值的两个/三个技能会在安装时互相覆盖）。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「超市模型」——提供大量高质量单品，用户自选搭配，CLI 工具是购物车。每个组件尽量独立，无依赖关系。

**解决什么问题**：用户想给特定领域（营销、安全、DevOps）的 Claude Code 会话增加专业能力，但从零写技能成本高、质量不稳定。

**做了什么 trade-off**：选择了「广度优先」而非「深度优先」——~100 个技能覆盖多个领域，但每个技能的深度（示例、边界条件、错误处理）参差不齐。安全技能的存在是刻意的教育选择，但也带来了自动化安全扫描的误报成本。

**反映什么认知模型**：作者把技能库理解为"知识组件市场"，每个技能是一个可插拔的领域专家，而非执行固定 SOP 的流程机器人。这与 NLPM 的单职责原则高度一致。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「组件超市 + CLI 购物车」**

将大量独立 NL 组件按领域分类，配套一个 CLI 工具让用户按需安装。核心特征是去中心化（组件之间无依赖）+ 基础设施化（CI/CD 自动验证每个 PR 的安全性）。

模式特征清单：
- **独立无依赖**：每个 SKILL.md 可以单独安装，不需要其他技能一起工作
- **领域分类索引**：`cli-tool/components/skills/` 按领域建子目录，降低发现成本
- **CI 安全门禁**：`.github/workflows/skill-security-scan.yml` 在每个 PR 合并前扫描新增技能，防止恶意 payload 混入
- **平台适配层**：配备网站（可视化浏览）+ CLI（命令行安装）双入口，覆盖不同用户习惯
- **版本管理基础设施**：通过 npm 发布管理版本，提供 `npm install -g` 的标准安装体验

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 社区驱动的技能市场 | ✅ 高度适用 | 去中心化的组件架构天然支持多人贡献 |
| 单一领域深度工具集 | ❌ 不适用 | 广度优先的设计不适合需要深度集成的场景 |
| 企业内部 skill 分发 | ⚠️ 有限适用 | 缺乏权限控制和私有化部署支持 |
| 教学/培训场景 | ✅ 适用 | 组件按领域分类，学习者可以逐类研究 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 组件超市（本仓库） | davila7/claude-code-templates | 覆盖广、发现容易、独立安装 | 组件质量参差不齐、无跨组件工作流 |
| 单一深度插件 | lackeyjb/playwright-skill | 深度优化、文档完整、评分高 | 只解决一个问题 |
| 个人工具集 | lijigang/ljg-skills | 高度定制、内聚性强 | 难以推广给陌生用户 |

### 2.4 改进空间

1. **当前问题**：`brand-guidelines` 重名技能（现在有 3 个文件声明了 `name: brand-guidelines`）在安装时会发生静默覆盖，用户无法预测安装了哪一个版本。**改进做法**：用 `brand-guidelines-community`、`brand-guidelines-anthropic`、`brand-guidelines-enterprise` 分别命名，并在 CI 中添加重名检查。**预期收益**：消除安装歧义，且 CI 可以阻止未来出现同名冲突。

2. **当前问题**：DevOps agent 文件用 VS Code Copilot 工具名（`codebase`、`editFiles`）而非 Claude Code 工具名，但 audit 发现这些 bug 在当前 HEAD 已不存在——可能有人修复了。值得保留的学习点是：**跨平台复用 agent 时，工具名是最容易遗漏的破坏性差异**，需要在 CI 中对工具名做白名单验证。

3. **当前问题**：安全技能（pentesting）中的 reverse shell payload 会触发任何基于正则的自动扫描。**改进做法**：在 `skills/security/` 目录根部添加 `SECURITY-EDUCATION.md` + 在每个进攻性技能顶部加 `audience: security-professional` [frontmatter](#frontmatter) 字段，允许扫描器按字段白名单跳过这些技能。

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 当时得分 **84/100**。

| 文件（代表） | 当时分数 | 主要问题 |
|---|---|---|
| devops/deploy-assistant/AGENT.md | 52 | VS Code Copilot 工具名（codebase、editFiles 等）无效 |
| devops/ci-cd-agent/AGENT.md | 52 | 同上 |
| brand-guidelines-community/SKILL.md | 75 | name: brand-guidelines 与 brand-guidelines-anthropic 重复 |
| business-marketing/ai-product/SKILL.md | 82 | Sharp Edges 解决方案列为空注释 |
| business-marketing/ceo-advisor/SKILL.md | 85 | 非标准 frontmatter（license, metadata 嵌套块） |
| business-marketing/launch-strategy/SKILL.md | 92 | 干净 |
| business-marketing/content-research-writer/SKILL.md | 94 | 干净 |

### 3.2 当时值得借鉴的模式

1. **「CI 安全门禁」** → `.github/workflows/skill-security-scan.yml` 在每个 PR 前自动扫描新增组件，防止恶意 payload 混入。借鉴点：规模化 skill 仓库需要自动化安全门禁，手工审查不可持续。

2. **「高质量营销 skill 示例」** → `business-marketing/content-research-writer/SKILL.md`（94分）是示例驱动（examples-driven）的优秀样板，描述清晰、无模糊量词、有完整输入→输出示例。借鉴点：为自己的 skill 库树立"94分基线"，每个新 skill 对照检查。

3. **「双入口分发」** → 网站浏览 + CLI 安装双通道，覆盖不同使用习惯的用户。借鉴点：如果 skill 库的目标受众超过个人，值得投入分发基础设施。

4. **「领域分类目录」** → `skills/business-marketing/`、`skills/security/` 等按领域建子目录，无需 README 索引用户就能通过目录结构发现组件。借鉴点：skill 命名前缀 + 目录分类双重索引，胜过单纯 README 列表。

### 3.3 当时的缺陷

1. **B1-B6：6 个 DevOps agent 使用 VS Code Copilot 工具名** → 根本原因：批量复制了 GitHub Copilot 的 agent 定义格式，未针对 Claude Code 适配工具名。`codebase`、`editFiles`、`runCommands` 等工具在 Claude Code 运行时不存在，agent 会静默收到空工具列表，表现为无法执行任何操作。自查：是否有从其他平台移植的 agent 定义？

2. **B7-B8（现在扩展为 B7-B9）：`name: brand-guidelines` 重名冲突** → 根本原因：不同贡献者独立提交了功能相似的技能，没有命名注册机制防止冲突。安装时第二个技能会静默覆盖第一个，用户无法察觉自己装了哪个版本。

3. **B9/SF1：`droid.md` 中的 unsigned curl-pipe-sh** → `curl -fsSL https://app.factory.ai/cli | sh`，从第三方域名拉取并立即执行脚本，无任何完整性验证。这不是教育性的安全技能，而是出现在通用用途 agent 中，用户触发时可能无意识地执行了外部代码。

### 3.4 当时的优化机会

1. 在 `ai-product/SKILL.md` 和 `zapier-make-patterns/SKILL.md` 的 Sharp Edges 表格中填写真实的解决方案内容，而非留空注释（`# Always validate output:`）。
2. 统一 `ceo-advisor`、`cto-advisor` 等 skill 的 frontmatter 格式，去除非标准字段（`license`, `metadata`）。
3. 为 7 个引用了不存在 Python 脚本的 skill 要么添加脚本文件，要么去掉脚本引用。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| DevOps agent VS Code 工具名（B1-B6） | `grep -r "codebase\|editFiles\|runCommands" cli-tool/components/agents/devops/` | **已修复**：grep 返回空，全部 6 个 agent 的 VS Code 工具名已被移除 | 社区 PR 或作者自修，约 2 个月内完成修复 |
| brand-guidelines 重名（B7-B8） | `grep -rn "^name: brand-guidelines" cli-tool/` | **恶化**：现在有 3 个文件声明了 `name: brand-guidelines`（新增了 `enterprise-communication/brand-guidelines/SKILL.md`） | 修复了当时的 2 个，但后续新增时又引入了第 3 个 |
| droid.md curl-pipe-sh（B9/SF1） | `grep -n "factory.ai\|curl.*sh" cli-tool/components/agents/expert-advisors/droid.md` | **仍存在**：第 21 行和第 238 行均有 `curl -fsSL https://app.factory.ai/cli \| sh`，无任何用户警告 | 高优先级安全问题持续超过 2 个月未修复 |

### 4.2 架构演进

新增了 `enterprise-communication/` 子目录（说明仓库在向企业沟通场景扩展），以及更完善的 GitHub Actions 工作流（`component-security-validation.yml` 是新增的）。这说明作者意识到了安全门禁的重要性并投入了基础设施——但 CI 扫描显然没有捕获 `droid.md` 中的问题，可能是因为 `droid.md` 早于 CI 门禁存在。

### 4.3 新增的可学习模式

**「component-pr-welcome.yml」**：为每个新组件 PR 自动发送欢迎 comment 并指引贡献者完成标准化提交，是社区运营的良好实践。规模化 skill 仓库需要让新贡献者自助完成质量检查。

---

## 五、校准

### 5.1 我已经在做对的

1. **单技能单职责**：drama-workshop-skills 的每个技能只做剧本创作的一个子任务，与本仓库的「独立无依赖」原则一致。✓
2. **版本号手动维护**：echo-sleuth-for-claude 有明确的版本号在 plugin.json 中，虽然没有 npm 发布，但版本追踪的意识是对的。✓

### 5.2 挑战 / 验证

**挑战了什么假设**：我以为"大型 skill 库"意味着"组件之间深度协同"，但本仓库证明"大而独立"（组件超市）是一个成功的商业模式——24685 stars 的大部分来自用户对独立可用组件的需求，而非对工作流编排的需求。

**验证了什么**：CI 安全门禁是规模化 skill 库的必需基础设施，不是可选的——否则任何贡献者都可能在无意中引入安全风险。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 agent/skill 文件中是否有非 Claude Code 的工具名
grep -rn "codebase\|editFiles\|runCommands\|terminalCommand\|githubRepo" \
  ~/.claude/skills/ ~/.claude/agents/ 2>/dev/null
# 命中后：替换为 Claude Code 有效工具名（Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Agent）

# 检查我的多技能仓库中是否有重名的 name 字段
find . -name "SKILL.md" -exec grep "^name:" {} \; | sort | uniq -d
# 命中后：重命名以消除冲突，加前缀区分（如 project-brand-guidelines）
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 添加类似的 GitHub Actions 安全扫描（检查 SKILL.md 中是否有 `curl|bash`、`eval`、未声明的外部请求）。
   - **为何可行**：drama-workshop-skills 目前只有 2 个 SKILL.md，扫描规则简单；随着 skills 增加，CI 门禁越来越值得。
   - **第一步**：复制本仓库的 `.github/workflows/skill-security-scan.yml`，修改扫描路径到 `skills/*/SKILL.md`。约 30 分钟完成。

2. **想法**：为 claude-for-legal 建立领域分类目录（`regulatory/`、`product/`、`corporate/` 等子目录），参考本仓库的 `business-marketing/`、`security/` 分类。
   - **为何可行**：claude-for-legal 已经有 `regulatory-legal/` 和 `product-legal/` 两个顶级目录，方向正确，深化分类即可。
   - **第一步**：在 `regulatory-legal/skills/` 下按技能功能建子目录，移动现有 skill 文件。约 20 分钟完成。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 davila7/claude-code-templates 的核心目的**：为多领域用户提供开箱即用的 Claude Code 组件库，通过 CLI 工具降低安装门槛，通过 CI 保证安全质量。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是多 skill 集合；都有领域分类（本案 marketing/security，我的 regulatory/product） | claude-for-legal 有工作流深度编排；本案是平铺组件库 | 高（CI 基础设施借鉴） |
| MarkQWu/drama-workshop-skills | 低 | 都有 skill 系统 | drama-workshop 是单一深度工具集；本案是广度组件超市 | 低 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都用 Claude Code plugin 格式 | 功能完全不同 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| brand-guidelines 重名（3 个文件同名） | `grep -rn "^name:" my-skills/ \| sort \| uniq -d` | **claude-for-legal 未命中**（每个 skill 名唯一）；但未来新增时风险存在 | 中（预防性）|
| Sharp Edges 留空注释占位符 | `grep -n "^# [A-Z].*:" skills/*/SKILL.md \| grep -v "##"` | **未检查到同类问题** | 暂无 |
| 脚本引用但文件不存在 | `grep -rn "scripts/.*\.py" skills/*/SKILL.md` | **claude-for-legal 有外部 MCP 引用**（.mcp.json 指向外部服务，但这是设计意图） | 低 |

**命中后的具体行动建议**：
- 为 claude-for-legal 添加 CI 检查：扫描所有 SKILL.md 中的 `name:` 字段，若有重复则 PR 失败。约 1 小时完成。

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：CI 安全门禁基础设施
   - **本案例做法**：`.github/workflows/skill-security-scan.yml` 在每个 PR 合并前自动运行安全扫描，阻止恶意 payload 进入主分支
   - **我的项目现状**：claude-for-legal 和 drama-workshop-skills 均无自动化安全扫描 CI，依赖人工审查
   - **如何借鉴**：复制本仓库的安全扫描 workflow，修改扫描路径，添加到自己的 `.github/workflows/` 中。关键扫描规则：`curl.*|.*sh`、`eval(`、`exec(`、`subprocess.shell=True`

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：工作流深度和内聚性
  - **我的做法**：claude-for-legal 中每个领域（regulatory/product/corporate）的技能形成完整的工作流链，技能之间通过共享参考文档（`references/` 目录）保持内聚
  - **本案例做法**：组件彼此独立，无法组合成端到端的法律工作流；缺少领域专家所需的深度（如合规检查清单、条款对比等）
  - **意义**：claude-for-legal 的深度定制是差异化竞争力，不应为了"组件超市"模式而削弱内聚性

---

## 八、术语表

### <a name="component"></a>组件（Component）
> 在本案例中，"组件"指 Claude Code 中的单个可安装单元，包括 SKILL.md（技能）、AGENT.md（代理）、hooks.json（钩子）、MCP 配置等。davila7/claude-code-templates 的 CLI 工具将这些单元打包管理，用户用命令行选择并安装到自己项目的 `.claude/` 目录中。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的 YAML 配置段，声明文件元数据。Claude Code 解析 SKILL.md 时先读 frontmatter，里面的 `name`、`description`、`model`、`allowed-tools` 等字段控制技能如何被注册和调用。非标准字段（如 `license`、`metadata` 嵌套块）会被 Claude Code 静默忽略。

### <a name="curl-pipe-sh"></a>curl-pipe-sh 风险
> `curl URL | sh` 是从互联网下载脚本并立即以当前用户权限执行的命令。风险在于：没有机会审查脚本内容，且若下载服务器被攻击，任何恶意代码都会在用户系统上运行。`droid.md` 中使用这个模式来安装第三方 CLI 工具，在一个通用 agent 中这是不可接受的风险。

### <a name="silent-override"></a>静默覆盖（Silent Override）
> 当两个 skill 声明了相同的 `name` 字段时，后安装的 skill 会在 Claude Code 的 skill 注册表中覆盖先安装的，且用户不会收到任何警告或错误提示。这是"品牌指南 brand-guidelines 重名"问题的核心危害。
