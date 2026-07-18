# affaan-m/everything-claude-code — 学习案例

**仓库**：https://github.com/affaan-m/everything-claude-code
**Stars**：N/A（注：官方 badge 显示 211.9K+，来自自定义 badge API，非 GitHub 原生星数）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-18（基于当前 HEAD v2.0.0）
**主题标签**：`security-gate`, `cross-reference`, `manifest-discipline`, `template-design`, `monorepo-vs-split`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
ECC（Everything Claude Code）是一个「跨工具 AI 编程助手配置平台」——不仅支持 Claude Code，同时支持 GitHub Copilot、Cursor、Codex CLI、OpenCode、Kiro、Cline、Windsurf、Roo Code 等 10+ AI 工具。包含 889 个 SKILL.md、424 个命令、multilingual 文档（11 种语言），提供 npm 包分发（`ecc-universal`、`ecc-agentshield`）和 GitHub App（ecc-tools）。v2.0.0 版本，有 Discord 社区、官网（ecc.tools）、SPONSORS 文件显示有商业赞助。作者 affaan-m 将其定位为「agent harness 操作系统」。

### 1.2 架构剖析
- **目录结构**（顶层）：
  ```
  affaan-m/everything-claude-code/
  ├── agents/              # 专门化子代理（code-reviewer、architect、planner等）
  ├── commands/            # 424 个斜线命令（/tdd、/plan、/code-review等）
  ├── skills/              # NL skills（coding-standards、patterns等）
  ├── hooks/               # trigger-based 自动化（session persistence、pre/post-tool）
  ├── rules/               # 始终遵守的规范（security、coding-style）
  ├── mcp-configs/         # MCP 服务器配置
  ├── .agents/             # 跨工具格式 agent（Codex CLI、Copilot等）
  ├── docs/                # 11 种语言文档 + 架构文档
  ├── scripts/ci/          # 全套 CI 验证脚本（validate-agents、validate-skills等）
  ├── manifests/           # 安装 manifest
  ├── contexts/            # 上下文注入文件
  ├── integrations/        # 外部集成
  ├── ecc2/                # v2.0 新架构目录
  └── CLAUDE.md            # 包含 Prompt Defense Baseline 的项目记忆
  ```
- **文件类型分布**：889 个 SKILL.md / 424 个 command / 多个 agent / hooks.json / 大量 scripts/ci/ 验证脚本
- **编排关系**：每个命令有独立 frontmatter 和执行逻辑；CI scripts 验证所有工件的合规性（`npm test` 跑 10+ 个 validate-*.js 脚本）；COMMAND-REGISTRY.json 自动生成并作为单一真相源
- **跨件契约**：`scripts/ci/catalog.js` 维护工件目录；`scripts/ci/generate-command-registry.js` 同步命令注册表；`CLAUDE.md` 里的 Prompt Defense Baseline 是所有代理的安全约束基础

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「跨工具统一接口」——把 AI 工作流标准化到 NL 层，让同一套知识（如 TDD 流程）在不同工具里都可用，而非每个工具单独维护
- **解决什么问题**：开发者在 Claude Code / Cursor / Copilot 之间切换时需要重学一套配置；ECC 提供一个「多工具公分母」的 NL 工件集
- **做了什么 trade-off**：选择极大规模（889 skills）而非精炼深度——覆盖面极广但维护难度极高；选择 [monorepo](#monorepo) 而非多仓库——便于 CI 统一验证但仓库体积膨胀（934+ 文件）
- **反映什么认知模型**：作者把 AI 编程助手看作可替换的「执行引擎」，而 NL 工件是跨引擎可移植的「程序」——这是比大多数插件更宏大的抽象层

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「平台级 NL 生态系统」（Platform-level NL Ecosystem）**：不是一个工具，而是一个平台——有 CI 验证、[monorepo](#monorepo) 管理、多工具适配、版本发布流水线的完整 NL 编程生态。

模式特征清单：
- CI 作为守护者：所有工件通过 `npm test` 里的 10+ 个 validate 脚本自动验证
- COMMAND-REGISTRY.json 自动生成（而非手动维护）
- 多工具适配层（`.agents/` 按工具分组 agent 格式）
- Prompt Defense Baseline 内嵌 CLAUDE.md 作为全局安全约束
- 商业化路径：GitHub App、npm 包、网站、Discord 社区

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖多种 AI 工具的企业内部工具包 | ✅ 高度适用 | 跨工具统一接口是核心价值主张 |
| 有完整 CI/CD 的中大型团队 | ✅ 适用 | CI 验证脚本显著提升工件质量 |
| 个人小项目（1-3 个 skill）| ❌ 不适用 | 维护 1000 个 skill 的 monorepo 基础设施成本远超个人项目需要 |
| 只用一种 AI 工具 | ❌ 不适用 | 跨工具能力是主要溢价，单工具用户难以榨取价值 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 平台级 NL 生态（本仓库）| affaan-m/ECC | CI 保证质量、跨工具覆盖、社区规模 | 维护成本极高、安全面极大、单仓库体积膨胀 |
| 专注单一工具的插件集 | aaronmaturen/claude-plugin | 专注、维护成本低 | 不可移植到其他工具 |
| NL 表皮 + 原生二进制核心 | agent-sh/agnix | 精确验证性能好 | 只做验证，不覆盖工作流 |

### 2.4 改进空间
1. **当前问题**：规模过大导致安全面过大（Audit 时 7 个安全发现）**改进做法**：将高风险脚本（外部 API 调用、二进制下载）隔离到单独包，不作为主包的依赖 **预期收益**：降低安全 BLOCKED 触发概率
2. **当前问题**：SKILL.md 超过 889 个，单仓库维护质量难以保证 **改进做法**：设立「精华层」（核心 30-50 个 skill）vs「社区层」（其余），精华层严格 review，社区层宽松接受 **预期收益**：核心工件质量可控

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **84/100**（均值，渐进式采样约 320 个 artifacts）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/test-coverage.md | 23 | 缺 name + description [frontmatter](#frontmatter) |
| commands/gan-build.md | 25 | 缺 frontmatter + 引用未定义的 agents |
| commands/learn.md | 29 | 缺 frontmatter |
| skills/customs-trade-compliance/SKILL.md | 55 | 模糊量词封顶 + 缺输出格式 |
| 多数高质量 skills | 85-95 | 少量模糊量词 |

安全状态：**BLOCKED**（7 个安全发现，含 3 个已确认风险）

| 发现 | 文件 | 严重度 | 当时状态 |
|---|---|---|---|
| `glm-extract.js` 将用户代码内容发送到第三方 API（api.z.ai）| scripts/glm-extract.js | HIGH | 已确认 |
| postinstall 下载二进制无 checksum | package.json | HIGH | 已确认 |
| curl 下载无 checksum 验证 | scripts/check-tool-releases.sh | MEDIUM | 已确认 |
| test fixture 里的 `curl \| bash` | tests/hooks/dangerous.json | CRITICAL | **误报**：测试夹具 |

### 3.2 当时值得借鉴的模式
1. **Prompt Defense Baseline** → CLAUDE.md 里 6 条明确的防注入规则（拒绝角色切换、拒绝泄露凭据、拒绝执行外部代码等）→ `CLAUDE.md §Prompt Defense Baseline` → 借鉴：在任何生产 NL 工件里都应有类似的安全基线声明
2. **CI 多层验证** → `npm test` 串行跑 validate-agents、validate-commands、validate-rules、validate-skills、validate-hooks、validate-install-manifests、validate-no-personal-paths + catalog check + command-registry check → 借鉴：CI 是 NL 工件质量的最后守门员
3. **COMMAND-REGISTRY.json 自动生成** → `generate-command-registry.js` 确保命令注册表永远与实际文件同步 → 借鉴：避免手动维护带来的陈旧问题

### 3.3 当时的缺陷
1. **glm-extract.js 向第三方 API 发送用户代码** → 为什么会引发安全 BLOCKED：用户使用插件时，他们的代码（可能含商业机密）被 stdin 传给一个中国第三方 AI 服务（api.z.ai）——用户可能完全不知情。这违反了「明确告知数据流向」的基本原则。自查：我的插件是否有任何工件会悄悄向外发送数据？
2. **postinstall 下载二进制无 checksum** → 为什么失败：安装时自动执行 postinstall，下载的二进制未经哈希验证直接执行，supply chain 攻击可以通过替换发布文件植入恶意代码。自查：我的仓库无 postinstall 脚本，暂无此风险
3. **部分命令缺 frontmatter** → 同 aaronmaturen 案例，但 ECC 只有少数命令有此问题（多数已修复）。自查：已在 gstack 检查，暂无命中

### 3.4 当时的优化机会
1. 完全移除 glm-extract.js 或将其改为 opt-in（明确提示用户数据将发送到第三方）
2. 修复 postinstall 的 checksum 验证
3. 为 20-30 个低分命令补全 frontmatter

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| glm-extract.js 向第三方发送代码 | `ls scripts/glm-extract.js` | **已修复**：文件不存在 | 主要安全 BLOCKED 来源已移除；v2.0 重构时清理了该脚本 |
| postinstall 下载二进制无 checksum | `cat package.json \| python3 -c "...scripts['postinstall']"` | **已修复**：package.json 无 postinstall 脚本 | v2.0 改变了分发方式，不再自动下载二进制 |
| 命令 frontmatter 不完整 | `head -3 commands/aside.md` 等 5 个样本 | **已修复**：所有抽查命令均有完整 frontmatter | CI validate-commands.js 强制执行 |

### 4.2 架构演进
从 Audit 时（v1.x）到当前 HEAD（v2.0.0），ECC 发生了重大架构迁移：
- 移除了高风险的 glm-extract.js 和 postinstall 二进制下载
- 新增 `ecc2/` 目录（v2.0 新架构），多 harness adapter 分离
- 新增 `scripts/ci/scan-supply-chain-iocs.js`（主动供应链安全扫描）
- CLAUDE.md 加入了 Prompt Defense Baseline 章节
- 版本从 1.x 跨越到 2.0.0，说明这是一次有意识的「安全 + 架构」双重重构

这说明 NLPM Audit 的安全 BLOCKED 可能确实推动了 v2.0 的安全重构（无法确认因果关系，但时序吻合）。

### 4.3 新增的可学习模式
1. **supply-chain-iocs 主动扫描**：`scripts/ci/scan-supply-chain-iocs.js` 在 CI 里主动扫描第三方依赖的恶意指标（IOC），这是从「被动合规」到「主动安全」的跃升
2. **Harness Adapter Compliance**：`scripts/harness-adapter-compliance.js` 验证每种工具的适配器是否符合 ECC 标准，支撑了真正的跨工具可移植性承诺

---

## 五、校准

### 5.1 我已经在做对的
1. **不向第三方发送数据**：我的所有 NL 工件都是纯本地操作，无外部 API 调用
2. **CI 验证意识**：agents-in-the-wild 的 auditor 工作流也有类似的「验证 → 修复」CI 模式
3. **frontmatter 强制**：gstack 和 bureau 的 SKILL.md 都有完整 frontmatter

### 5.2 挑战 / 验证
- **挑战**：规模与安全的张力——ECC 因为规模大（934 artifacts、10+ 第三方脚本），安全面变得无法手动管控，必须依赖 CI 扫描工具。这提醒我：**超过某个规模阈值，NL 工件集就需要自动化安全守卫，而不能靠人工审查**
- **验证**：「先被 BLOCKED 再修复」的生命周期——ECC 的安全 BLOCKED 状态在 v2.0 里被系统性修复了。说明安全审计 → 披露 → 修复是可以工作的反馈闭环，只要作者愿意处理

---

## 六、行动

### 6.1 自查动作

```bash
# 检查是否有任何外部 API 调用（防止意外数据外发）
grep -rn "fetch\|axios\|https\.get\|curl" ~/.claude ~/.claude-plugin ~/gstack ~/bureau \
  --include="*.js" --include="*.sh" --include="*.py" 2>/dev/null | \
  grep -v "github.com\|anthropic.com\|local\|localhost" | head -10
# 命中后：审查每一条，确认是否有用户数据流向第三方

# 检查 package.json 是否有 postinstall
grep -rn "postinstall" ~/gstack/package.json ~/bureau/package.json 2>/dev/null
# 命中后：确认 postinstall 是否有 checksum 验证
```

### 6.2 灵感 → 实施路径
1. **想法**：在 agents-in-the-wild 的 CI 里加一个「外部 API 调用扫描」步骤（仿 ECC 的 scan-supply-chain-iocs.js）
   - **为何可行**：auditor 工作流贡献 PR 到外部仓库，如果我们的脚本被污染，影响面很广
   - **第一步**：在 `.github/workflows/` 里新建一个检查脚本，grep 所有 .sh/.js/.py 文件里的非白名单域名调用，30 分钟完成

2. **想法**：借鉴 ECC 的 Prompt Defense Baseline，在 agents-in-the-wild 的 CLAUDE.md 里加入安全约束声明
   - **为何可行**：auditor 工作流读取外部仓库内容，存在提示注入风险（CLAUDE.md 里已有相关注意事项，但不如 ECC 系统化）
   - **第一步**：在 CLAUDE.md 里新增「Security Baseline」章节，列出 5 条防注入规则（拒绝角色切换、不执行外部代码、不泄露凭据等），20 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 affaan-m/ECC 的核心目的**：跨工具 AI 编程助手配置平台，提供工作流 NL 工件集

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是 Claude Code skill 集合，都面向工程师日常工作流 | gstack 只支持 Claude Code，ECC 跨 10+ 工具；gstack 23 个 skill，ECC 889 个 | 高 |
| MarkQWu/bureau | 低 | 无直接对应 | bureau 是知识库工具，ECC 是工作流工具 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷（历史）| 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令 frontmatter 缺失 | `head -3 commands/*.md \| grep "^---"` | 未命中：gstack 所有 SKILL.md 有 frontmatter | 低 |
| 外部 API 调用无告知 | `grep -rn "fetch\|curl" --include="*.js"` | 未在 my-repos 中发现未告知的外部调用 | 低 |
| CLAUDE.md 缺安全约束声明 | `grep -c "Defense\|prompt injection\|安全" CLAUDE.md` | **可能命中**：gstack CLAUDE.md 无系统化安全约束章节 | 中 |

**命中后的具体行动建议**：
- `MarkQWu-gstack/CLAUDE.md` → 在顶部或独立章节加 3-5 条防注入规则 → 15 分钟完成

### 8.3 别人的更优方案

1. **领域**：CI 全链路 NL 工件验证
   - **本案例做法**：`npm test` 里串行跑 10+ 个 validate-*.js 脚本，覆盖 agents/commands/rules/skills/hooks/manifests
   - **我的项目现状**：gstack 无自动化 NL 工件 CI 验证
   - **如何借鉴**：在 gstack 的 GitHub Actions 里加一步调用 `/nlpm:score` 或 `bin/nlpm-check` 验证所有 SKILL.md

2. **领域**：Prompt Defense Baseline 在 CLAUDE.md 里的系统化声明
   - **本案例做法**：CLAUDE.md 开头有 6 条明确的防注入规则，覆盖角色切换、数据泄露、恶意指令等场景
   - **我的项目现状**：gstack CLAUDE.md 无安全约束章节
   - **如何借鉴**：直接复制 ECC 的 Prompt Defense Baseline 结构（6 条），调整到 gstack 的语境

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：规模控制与深度
- **我的做法**（gstack）：23 个精选 skill，每个都经过深度设计（有 examples、有 triggers、有输出格式声明）
- **本案例做法**（ECC）：889 个 skill，覆盖面极广但质量参差（Audit 时部分 skill 分数仅 55）
- **意义**：「精炼 23 个」比「平铺 889 个」在个人使用场景下更可维护、更高质量。规模不是竞争力，深度才是

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明 name、description、allowed-tools 等元数据。Claude Code 用它注册和调用命令/skill。

### <a name="monorepo"></a>monorepo
> 「单仓库」策略：把多个相关项目（skills、commands、agents、docs、scripts）放在同一个 git 仓库里管理，而非每个组件独立一个仓库。优点：共享 CI 配置、统一 PR 流程、跨组件引用方便；缺点：仓库体积大、clone 慢、权限粒度粗。

### <a name="ioc"></a>IOC
> Indicator of Compromise（威胁指标）。在供应链安全语境下，指可能是恶意软件或被篡改代码的特征（如向未知 IP 发送数据、下载来路不明的可执行文件）。ECC 的 scan-supply-chain-iocs.js 自动检测依赖里的 IOC。

### <a name="prompt-injection"></a>提示注入
> 攻击者通过在 AI 会话的上下文（如用户提交的代码、外部网页内容）里嵌入「伪指令」，诱导 AI 绕过安全约束执行恶意操作（如泄露凭据、调用危险 API）。ECC 的 Prompt Defense Baseline 是对抗这类攻击的 NL 层防御。

### <a name="harness"></a>Harness
> 在 ECC 语境里，指 AI 编程助手工具（Claude Code、GitHub Copilot、Cursor 等）。ECC 提供「harness adapter」让同一套 NL 工件适配不同工具的格式要求。
