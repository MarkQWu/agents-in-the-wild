---
repo: yonatangross/orchestkit
audit_date: 2026-04-19
case_date: 2026-07-26
nl_score: 84
security_risk: CRITICAL
stars: 149
source: upstream
spot_check_date: 2026-07-26
tags: [security-critical, examplePrompts-deficit, mcp-mismatch, undeclared-tool, webhook-exfiltration, high-scale-growth]
---

# 学习案例：yonatangross/orchestkit

> **一句话定位**：一个成熟的大规模 Claude Code 插件（103→265+ artifacts，15→36 agents），NL 质量稳健（84/100），但脚本层埋着 CRITICAL `eval` 炸弹，webhook 转发器在所有 27 个事件上异步开枪，MCP 声明互相打架。安全是硬伤；NL 层的 examplePrompts 从 0/15 扩张到 36/36 则是三个月内最大的正向变化。

---

## §1 理解

### 这个仓库是什么

OrchestKit 是一个以 **前端/全栈工程师角色编排** 为核心的 Claude Code 插件生态：15 个专家 agent（security auditor、frontend developer、backend architect 等）+ 88 个跨领域 skill（storybook、langgraph、design tokens、distributed systems……），外加一套用 TypeScript + esbuild 打包的 hooks 管线。

架构分两层：
- `src/skills/`（74 个）：通用技能，面向所有场景
- `plugins/ork/skills/`（14 个）：插件专属技能，依赖 ork 命名空间前缀
- `src/agents/`（15 个）：专家 agent，每个都持有一批技能引用

### 为什么选这个仓库

- **星数低（149★）但 NLPM 评分高**：说明"是否有星"和"NL 质量"是独立维度
- **CRITICAL 安全风险**：`eval "$(...)"` 这类炸弹值得作为警示案例深读
- **大规模成长**：三个月后 agents 从 15 增到 36、skills 从 88 增到 229，如何在规模扩张中保持质量是真实问题

### 架构图（简化）

```
claude plugin install orchestkit@yonatangross
             │
     ┌───────┴────────┐
     │                │
  src/agents/      src/skills/  +  plugins/ork/skills/
  (36 agents)      (115 + 114 skills)
     │
  hooks pipeline
  (TypeScript/esbuild, 分事件类型打包)
     │
  lifecycle/webhook-forwarder (async, 27 events) ← HIGH 风险
```

---

## §2 架构作为可学习模式

### 模式一：双层 Skill 目录（通用 vs 专属）

`src/skills/` 存放可跨 agent 复用的通用知识（browser-tools、architecture-patterns、llm-integration）；`plugins/ork/skills/` 存放只在 ork 命名空间内有意义的专属知识（figma-design-handoff、github-operations、bare-eval）。

**学习要点**：两层分离的好处是通用技能不会因插件升级而失效；专属技能可以自由引用插件内部的 MCP 服务器，不会污染通用空间。缺点是 `plugins/ork/skills/` 里几个质量最差的 skill（bare-eval、release-sync）恰好是最专属、最难测试的那批——专属化越强，审查压力越低，质量越容易滑落。

### 模式二：TypeScript hooks + esbuild 打包

hooks 管线不是普通 bash 脚本，而是 TypeScript 模块用 esbuild 打包成 `dist/*.mjs` 再由 `run-hook.mjs` 按事件类型动态 `import()`。好处是有类型检查、可以做 tree-shaking；坏处是 bundlePath 的构造逻辑如果有漏洞（`argv[2]` 到 `bundleMap` 到 `distDir + bundleName + '.mjs'`），路径遍历风险就藏在编译期看不见的地方。

### 模式三：async webhook forwarder——全量事件订阅

`lifecycle/webhook-forwarder` 以 `async: true` 挂在所有 27 个 hook 事件上。这个模式的**意图**是全程可观测性（observability）——每个工具调用都会被转发到配置的 endpoint。**问题**是"全量"等于"零隐私"：一旦 endpoint URL 被 DNS 劫持或配置篡改，所有命令、路径、内容实时外泄，没有事件级别的过滤，没有 HMAC 验签。

这是一个"设计优秀但安全边界零防御"的反例：observability 和 security 的需求在这里正面冲突，且安全边界未被建立。

### 模式四：agent 的 examplePrompts 大规模补全

审计时（2026-04-19）：0/15 agents 有 `examplePrompts`。  
现状（2026-07-26）：36/36 production agents 有 `examplePrompts`。

这是三个月内最大的正向演化。`examplePrompts` 驱动 CC 的「建议输入」UX——没有它时，agent 在 CC 界面里几乎不可被发现（除非用户恰好知道 agent 名称）。从 0 到 36 的补全，理论上把所有 agent 的发现性提升到了标准水平。

---

## §3 过去审查

**审计日期**：2026-04-19  
**NLPM 分数**：84/100（15 agents 均分 78 + 74 src/skills 均分 85 + 14 plugin/skills 均分 86）  
**安全级别**：CRITICAL（2 个 CRITICAL + 3 HIGH + 2 MEDIUM + 2 LOW）

### 安全发现摘要

| 级别 | 编号 | 问题 | 文件 |
|------|------|------|------|
| CRITICAL | C-01 | `eval "$("$PROJECT_ROOT/bin/count-hooks.sh")"` 执行外部脚本完整 stdout | scripts/stamp-counts.sh:30 |
| CRITICAL | C-02 | 386 个脚本中 14 个 CRITICAL 模式匹配（eval、curl-pipe-sh、credential exfil 签名） | scripts/** |
| HIGH | H-01 | webhook-forwarder 挂在全部 27 个事件，异步无过滤 | src/hooks/hooks.json |
| HIGH | H-02 | `run-hook.mjs` 动态 `import()` 路径由 argv 派生，bundleMap 无 canonicalize | src/hooks/bin/run-hook.mjs:47-68 |
| HIGH | H-03 | component-curator：`mcpServers:[storybook-mcp]` vs `required_mcp_servers:[21st-dev-magic]` | src/agents/component-curator.md |

### NL 质量发现摘要

| 规则违反 | 次数 | 每次扣分 |
|---------|------|---------|
| 缺 `examplePrompts`（agents） | 15 | -5 |
| 缺 `disable-model-invocation` 字段 | 14 | -2 |
| 缺 `allowed-tools` | 8 | -5 |
| 缺 `model` 字段 | 7 | -3 |
| 缺 `user-invocable` | 5 | -5 |
| agent body 引用未声明工具（TaskGet） | 1 | -10 |
| MCP server frontmatter 矛盾 | 1 | -15 |
| disallowedTools 与 tools 矛盾（eval-runner） | 1 | -5 |

---

## §4 现在vs过去对比

*现场复查日期：2026-07-26，git clone --depth=1 HEAD*

### 缺陷追踪表

| 缺陷 | 审计时状态 | 现状 | 结论 |
|------|-----------|------|------|
| C-01：stamp-counts.sh:30 的 `eval "$(...)"` | CRITICAL | `eval "$("$PROJECT_ROOT/bin/count-hooks.sh")"` **原样保留** | ❌ 未修 |
| H-01：webhook-forwarder 全量 27 事件 | HIGH | `lifecycle/webhook-forwarder` 仍异步挂载，未见过滤 | ❌ 未修 |
| H-03：component-curator MCP 矛盾 | HIGH | `mcpServers: [storybook-mcp]` vs `required_mcp_servers: [21st-dev-magic]` 原样保留 | ❌ 未修 |
| infrastructure-architect 引用 `TaskGet` 但 tools 无声明 | HIGH | body 第 95 行仍见 `TaskGet`，tools 数组仍无 `TaskGet` | ❌ 未修 |
| 15 agents 全部缺 `examplePrompts` | MEDIUM | **36/36 production agents 已全部补全** | ✅ 已修 |
| agents 缺 `name:` 字段（NLPM PRs 主题） | MEDIUM | 36/36 agents 均有 `name:` | ✅ 已修 |

### 规模演化

```
              2026-04-19        2026-07-26      变化
agents:           15                36         +140%
src/skills:       74               115         + 55%
plugin/skills:    14               114         +714%
总 artifacts:    103               265+        +157%
```

在 265+ artifacts 中，整体 NL 质量（84/100 均分）基本保持稳定——说明新 agent/skill 添加时遵循了类似的 frontmatter 规范。这是值得肯定的**规模维持质量**能力，但安全层的两个 CRITICAL 问题完全没有移动。

### eval 原文引用

```bash
# scripts/stamp-counts.sh:28-36（HEAD）
SKILLS=$(find "$PROJECT_ROOT/src/skills" ...| wc -l | tr -d ' ')
AGENTS=$(find "$PROJECT_ROOT/src/agents" ...| wc -l | tr -d ' ')
eval "$("$PROJECT_ROOT/bin/count-hooks.sh")"   # ← CRITICAL：三个月后原样保留
HOOKS=$TOTAL
HOOKS_GLOBAL=$GLOBAL
```

**正确修法**（审计建议，一行也不嫌少）：
```bash
while IFS='=' read -r key val; do
  case "$key" in
    TOTAL)  HOOKS="$val" ;;
    GLOBAL) HOOKS_GLOBAL="$val" ;;
    AGENT)  HOOKS_AGENT="$val" ;;
    SKILL)  HOOKS_SKILL="$val" ;;
  esac
done < <("$PROJECT_ROOT/bin/count-hooks.sh")
```

---

## §5 校准

### 评分对照（NLPM 标准）

| 问题 | NLPM 扣分根据 |
|------|-------------|
| CRITICAL eval | 安全阻断；NL 层不直接扣分，但会导致 `security-blocked` 标签，NLPM 停止 PR 流程 |
| component-curator MCP mismatch | -15（agent MCP 声明矛盾，运行时必失败） |
| infrastructure-architect TaskGet 未声明 | -10（agent body 引用工具但 tools 数组缺失） |
| 缺 `examplePrompts` × 15 | -5 × 15 = -75（但有均摊缓解） |
| 缺 `allowed-tools` × 8 | -5 × 8 = -40 |
| 缺 `model` × 7 | -3 × 7 = -21 |

### eval 的危险程度再理解

`eval "$(...)"` 不只是"危险的 shell 写法"——它把**执行权**完全交给被求值的字符串：
- 如果 `count-hooks.sh` 被替换（依赖被污染、软链接攻击）
- 或其 stdout 包含意外内容（注入、边界条件）
- 执行结果是以调用者权限运行任意代码

在 CI 环境中，`stamp-counts.sh` 以 build 用户权限运行，且 `$PROJECT_ROOT/bin/` 可能来自 git clone——任何能推送恶意 commit 的人都可以触发这条路径。这不是理论风险，是实际可利用漏洞。

### webhook 数据面泄漏模型

```
用户在 CC 中调用 agent
     │
     ↓
PreToolUse / PostToolUse hook 触发
     │
     ↓（async: true）
lifecycle/webhook-forwarder 把事件发给 WEBHOOK_URL
     │
     ↓（如果 WEBHOOK_URL 被劫持）
攻击者收到：工具名、参数、文件路径、写入内容、环境变量快照
```

正确的 webhook 设计应该有三层防线：事件白名单（只转发 PostToolUse），工具黑名单（不转发处理 .env、SSH key、token 的工具），以及 HMAC 签名验证（防止中间人篡改响应）。

---

## §6 行动

### 对 orchestkit 本身（如需 PR）

**P0（安全，应立即提 PR）：**
```bash
# 替换 stamp-counts.sh:30
# 把 eval "$(...)" 改为 while IFS='=' read -r key val 循环
# 零行为变化，消除 CRITICAL
```

**P1（NL 质量，影响运行时正确性）：**
```bash
# component-curator.md: 统一 mcpServers 和 required_mcp_servers 为同一服务器名
# infrastructure-architect.md: 在 tools 数组加 TaskGet
```

**P2（NL 质量，影响可发现性）：**
```markdown
# 8 个缺 allowed-tools 的 skill 各加一行：
# allowed-tools: [Bash]  # 或对应工具
```

### 对我自己的学习行动

1. **Shell 脚本先做 `shellcheck`**：在任何脚本进入 CI 前，`shellcheck -S error` 是零成本的 CRITICAL 过滤器。本地几乎不需要 `eval "$(...)"` 来读取外部命令输出——`while IFS='=' read -r` 既安全又清晰。

2. **webhook hook 必须有事件白名单**：如果 bureau 或 gstack 未来需要 webhook 通知，默认应只挂 `PostToolUse`，并明确列出 `not_applicable_tools: [Write, Edit, Bash]`（或等价黑名单），避免内容泄漏面。

3. **MCP 声明用单一字段**：`mcpServers` 和 `required_mcp_servers` 是两个不同字段，用途不同——`mcpServers` 是 agent 运行时实际使用的服务器，`required_mcp_servers` 是 CC 安装时告知用户需要的服务器。二者**必须一致**，否则安装提示正确但 agent 运行时找不到服务器。

4. **examplePrompts 的 ROI 极高**：从 0→36 的修复效果说明这是"低成本高收益"的改进。在 bureau 和 gstack 中，任何有 agent 的地方都应补 examplePrompts，每个只需要 2-3 条自然语言句子。

5. **规模增长期间做质量基线**：orchestkit 在 3 个月内 artifacts 增长 157%，整体 NL 均分保持在 84/100，说明他们在 CONTRIBUTING.md 里有质量约束（有 `src/skills/CONTRIBUTING-SKILLS.md`）。在 gstack/bureau 增长时，应该对应维护 CONTRIBUTING 文档中的 frontmatter checklist。

---

## §7 对照我的 GitHub 仓库

### 复查范围

只对比 3 个有 NL artifacts 的仓库：**MarkQWu/gstack**、**MarkQWu/bureau**、**MarkQWu/graphify**

### Hooks 安全面对比

| 维度 | orchestkit | bureau | gstack | graphify |
|------|-----------|--------|--------|---------|
| 有 hooks.json | ✅ | ✅ | ❌ | ❌ |
| Hook 事件覆盖 | 27 个（全量） | 2 个（SessionEnd+SessionStart） | — | — |
| Webhook 转发器 | ✅（有，HIGH 风险） | ❌ | ❌ | ❌ |
| 工具级别过滤 | ❌ | N/A | N/A | N/A |
| HMAC 签名 | ❌ | N/A | N/A | N/A |

**结论**：bureau 的 hooks 范围极窄（仅 SessionEnd 做 capture-stub、SessionStart 做 scribe-checkpoint），安全面很小，但也缺乏中间工具调用的可观测性。这是保守的正确选择——在没有 HMAC 和过滤器的情况下，"不挂"比"全挂"更安全。

### examplePrompts 对比

| 仓库 | 总 agents | 有 examplePrompts | 比率 |
|------|---------|-------------------|------|
| orchestkit (审计时) | 15 | 0 | 0% |
| orchestkit (现在) | 36 | 36 | 100% |
| gstack | ~8 | ~8（有示例触发语句） | ~100% |
| bureau | 0 | N/A | N/A |
| graphify | 0 | N/A | N/A |

bureau 和 graphify 目前没有 agent 文件，不存在 examplePrompts 缺失问题。gstack 的 skill 文件里有类似「触发短语」的设计，功能等价。这一项三个仓库均不是问题。

### allowed-tools 对比

| 仓库 | skills with allowed-tools | commands with allowed-tools |
|------|--------------------------|----------------------------|
| orchestkit (缺失 8/88) | 91% 有 | N/A |
| gstack | skills 有（per-tool 精确声明） | N/A |
| bureau | skills 无（7 个 skill 均缺） | commands 无（10 个 command 均缺） |
| graphify | N/A | N/A |

**bureau 的 allowed-tools 缺失是优先级最高的待修问题**：10 个 commands 和 7 个 skills 均未声明 `allowed-tools`，这意味着 CC 无法在权限请求 UI 中正确展示可用工具，用户体验和安全审计都受影响。orchestkit 的缺失比率（9%）远低于 bureau（100%）。

### 工具声明完整性对比

orchestkit 的 `infrastructure-architect` 引用 `TaskGet` 但 tools 数组未声明——这类"代码体与声明不一致"的漏洞。

**在 bureau 是否存在类似问题**：bureau 的 10 个 commands 未读 tools 数组，但 commands 本身通常不严格声明 tools（由用户决定权限），风险较低。需关注的是 bureau 的 `recall.md` 命令是否在 body 中使用了 WebFetch/WebSearch 但未声明。

---

## §8 术语表

| 术语 | 定义 |
|------|------|
| `eval "$(...)"` | Shell 的命令替换+eval 组合：先执行括号内命令获取 stdout，再将 stdout 作为 shell 代码执行。存在任意代码执行风险，应用 `while IFS= read` 替代 |
| `async: true` (hook) | Claude Code hook 的异步执行模式：hook 在后台执行，不阻塞工具调用。适合日志/通知；不适合需要拦截的场景 |
| `mcpServers` vs `required_mcp_servers` | agent frontmatter 中两个不同字段：`mcpServers` 声明 agent 运行时实际使用的 MCP 服务器；`required_mcp_servers` 是 CC 安装提示告知用户需要预先配置的服务器。二者必须指向同一服务器 |
| webhook forwarder | 把 CC 工具调用事件转发到外部 HTTP endpoint 的 hook。如不加过滤和签名验证，构成数据外泄攻击面 |
| `examplePrompts` | agent frontmatter 字段：列举触发该 agent 的示例用户输入。CC 用它驱动「建议输入」自动补全 UI，缺失则 agent 在界面中几乎不可被发现 |
| bundleMap | run-hook.mjs 中的 hook 名到 bundle 文件名的映射表。作为 allowlist 防止 argv 注入，但 null 返回的处理仍需路径 canonicalize |
| `disable-model-invocation` | skill frontmatter 字段。纯知识/参考型 skill 应设为 `true`（读取时不触发 LLM 推理）；操作型 skill 设为 `false` |
| 双层 skill 布局 | `src/skills/`（通用）+ `plugins/ork/skills/`（插件专属）的分层设计。通用层可跨 agent 复用；专属层可自由引用插件内部服务 |
| CONTRIBUTING-SKILLS.md | orchestkit 中规范新 skill 贡献流程的文档，是维持大规模增长期间质量稳定的关键约束文件 |
