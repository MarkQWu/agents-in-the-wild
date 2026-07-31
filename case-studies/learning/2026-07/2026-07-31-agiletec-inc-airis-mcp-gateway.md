# agiletec-inc/airis-mcp-gateway — 学习案例

**仓库**：https://github.com/agiletec-inc/airis-mcp-gateway
**Stars**：151 | **来源**：upstream audit
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-07-31（基于当前 HEAD）
**主题标签**：`security-gate`, `curl-pipe-bash-risk`, `examples-driven`, `template-design`, `offline-capable`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
airis-mcp-gateway 是一个「按需启动的本地 [MCP](#MCP) 端点」——Claude Code 需要时才唤醒，不用时保持休眠，以节省资源。当前版本管理的主要服务是 Context7 进程服务器。CLAUDE.md 全部以日语撰写（罕见），说明作者群体主要面向日本开发者。安装方式为 `curl | bash` 一行指令，这也是本案的核心安全争议点。

关键事实：
- 由 agiletec-inc 团队维护，日语 CLAUDE.md 暗示核心用户群在日本
- Python 后端（`apps/api/` 下约 55 个 Python 源文件）+ Claude Code NL 层（4 个 skills + 3 个 commands）
- 151 Stars，在 MCP gateway 品类内属中等规模
- 在 CLAUDE.md 中明确列出「推测不到的边界」（不能假设 MCP 常设全局、不能修改 native 文件等）

### 1.2 架构剖析

```
airis-mcp-gateway/
├── .claude/
│   └── commands/           # 3 个命令：test.md, status.md, troubleshoot.md
├── config/
│   └── bootstrap/assets/claude/
│       └── commands/       # 2 个 bootstrap 命令：airis-capability-router.md, airis-research-first.md
├── skills/
│   └── skills/             # 4 个 skill: mcp-database, mcp-debugging, mcp-implementation, mcp-research
│       └── .claude-plugin/
│           └── plugin.json # 100/100 的 manifest
├── apps/
│   └── api/                # Python FastAPI 后端（~55 个源文件）
├── scripts/                # 安装脚本：install.sh, quick-install.sh, airis-gateway（核心）
└── CLAUDE.md               # 全日语，列出「推测不到的边界」
```

- **文件类型分布**：4 个 SKILL.md + 5 个 command + 0 个 agent（NL 层轻量化）
- **编排关系**：commands 调用 MCP server；skills 提供领域知识（mcp-debugging skill 还交叉引用了外部 `superpowers` 插件）
- **跨件契约**：plugin.json 独立管理（得分 100），commands 和 skills 分布在两棵目录树下，bootstrap 目录下的命令用于初始化安装流程

### 1.3 设计思路 / 方法论

- **核心哲学**：「最小化常设」——MCP gateway 不以全局服务形式挂在后台，而是由 Claude Code skill 按需创建短命 session
- **解决的问题**：本地 MCP 端点管理的复杂性——多服务器配置、动态启停、连接调试
- **Trade-off**：极简 NL 层（仅 4 skills + 5 commands）换来了维护成本低；代价是每个 skill 缺乏示例，新用户学习曲线较陡
- **认知模型**：作者把 MCP gateway 视为一个「会话级工具」而非「长驻服务」，这与主流 MCP 部署思路形成反差

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「最小 NL 表皮 + 重型原生后端」**模式：Python/Go/Rust 实现的核心能力，Claude Code NL 层只做调用入口，不包含业务逻辑。

模式特征：
- NL 层文件数 ≤10（此处 9 个）
- NL 文件几乎不含判断逻辑，仅描述「调用哪个工具、传什么参数」
- 后端维护者与 NL 层作者可能是同一人
- 安全边界在 NL 层之外（后端脚本承担所有系统级操作）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| MCP server/bridge 管理工具 | ✅ 高度适用 | NL 层只是驱动层，后端是核心 |
| 跨平台系统工具（macOS/Linux）| ✅ 适用 | 安装脚本维护，NL 层无平台差异 |
| 纯 NL 工作流（如 research/skills）| ❌ 不适用 | NL 层过薄，缺乏 skill 复用能力 |
| 需要多人协作的 skill 库 | ❌ 不适用 | 没有 skill 组合机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 最小 NL + 重型后端（本案） | airis-mcp-gateway | 后端能力完整，NL 维护成本低 | NL 层缺示例，缺失给用户的「脚手架」 |
| 纯 NL skill 库 | addyosmani-agent-skills | 可移植性高，无安装步骤 | 复杂操作难以纯 NL 实现 |
| NL 驱动 + 脚本联动 | agent-sh/agnix | 兼顾 NL 表达力与系统级能力 | 脚本安全管理复杂 |

### 2.4 改进空间

1. **当前问题**：4 个 skills 零示例，用户不知道如何调用。**改进做法**：每个 skill 补一个 `## Example` block，展示典型的输入（用户意图）→ 输出（MCP 操作结果）。**预期收益**：新手上手时间从「读文档 + 摸索」降至「看示例秒懂」。

2. **当前问题**：install.sh 和 quick-install.sh 将 `curl | bash` 作为唯一官方安装路径。**改进做法**：提供 checksummed tarball 安装路径，并在 README 中将其设为首选方式，`curl | bash` 仅作为「快速体验」的标注风险选项。**预期收益**：安全敏感的企业/团队用户可以采用，覆盖面增加。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 当时得分 **90/100**，SECURITY **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| airis-research-first.md | 78 | 无输出格式 + 无序列表替代编号步骤 |
| mcp-database/SKILL.md | 83 | 无示例 + "appropriate caution" 模糊量词 |
| mcp-debugging/SKILL.md | 85 | 无示例 |
| mcp-implementation/SKILL.md | 85 | 无示例 |
| mcp-research/SKILL.md | 85 | 无示例 |
| troubleshoot.md | 90 | $ARGUMENTS 无空输入处理 |
| plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **plugin.json 满分** → 所有字段声明完整，路径正确，没有 manifest 漂移问题。根本原因：作者显然把 manifest 视为「合约」而非「可选元数据」。如何借鉴：插件 manifest 每次新增 skill 就同步更新。

2. **CLAUDE.md「推测不到的边界」列表** → 明确告诉 Claude「不要做什么」，防止 AI 自作主张地扩展权限（如不启动 global MCP、不修改生成标记）。根本原因：作者意识到 AI 有时会根据「看起来合理」的推断做超出授权的事。如何借鉴：在项目 CLAUDE.md 中写一个「禁止推断操作」小节。

3. **airis-capability-router.md** → 得分 98/100，唯一问题是「briefly」这个模糊量词。命令体清晰，路由逻辑显式。

4. **Bash(curl*) 声明过度** → test.md 声明了 `Bash(curl*)` 但命令体无 curl 直接调用。这是反模式——声明未用工具，NLPM 会扣 3 分（可见性/安全性都降低）。

### 3.3 当时的缺陷

1. **4 个 skills 无示例（-15 分×4）** → 为什么会失败：AI 缺乏输入/输出的具体参考，在实际调用时会模糊解读 skill 意图，生成不符合预期的 MCP 操作。**自查**：我的 skills 里有没有零示例？

2. **Critical: curl|bash 安装模式** → 为什么这么设计会失败：任何能向 GitHub push 的人都可以在下次 `curl | bash` 时执行任意代码于用户机器。这不是假设威胁，历史上已有 `event-stream`、`node-ipc` 供应链攻击案例。

3. **mcp-debugging/SKILL.md 交叉引用 `superpowers` 插件但未声明依赖** → 为什么会失败：用户看到「调用 superpowers:systematic-debugging」但该插件未安装时，guidance 静默失效。根本原因：跨插件引用未经版本锁定。

### 3.4 当时的优化机会

1. 4 个无示例 skills → 各补一个示例 block
2. $ARGUMENTS 无空输入处理（troubleshoot.md）→ 加 fallback 分支
3. test.md 未使用 Bash(curl*) → 从 allowed-tools 中移除

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| curl|bash 在 install.sh | `grep "curl.*bash" install.sh` | **持续**：第 6 行、第 266 行均命中 | 3 个月未修，作者认为可接受 |
| curl|bash 在 quick-install.sh | `grep "curl.*bash" scripts/quick-install.sh` | **持续**：第 6 行命中 | 同上 |
| 4 skills 无示例 | `find skills/ -name "SKILL.md"` 查看内容 | 目录结构未变，内容需人工验证 | 可能持续 |

### 4.2 架构演进

审计时（2026-04-26）与当前 HEAD 的目录结构无明显重组。CHANGELOG 和 git log 显示主要工作集中在 Python backend（apps/api/）和安装脚本优化，NL 层（skills/commands）变化不大。这说明作者的注意力在 MCP 功能扩展而非 NL 质量优化。

### 4.3 新增的可学习模式

暂无。CLAUDE.md 内容相对稳定，「推测不到的边界」列表是持续学习的参考点，但没有引入新的架构模式。

---

## 五、校准

### 5.1 我已经在做对的

1. **plugin.json 视为合约**：参考此案 100 分 manifest，我也应维护完整 manifest
2. **CLAUDE.md「边界声明」**：明确说「不要做什么」是好实践，比只说「应该做什么」更有效
3. **命令 allowed-tools 声明精确**：本案 test.md 因声明未用工具被扣分，提醒我保持声明的精确性

### 5.2 挑战 / 验证

**挑战我的假设**：我之前认为「NL 层轻薄 = 质量差」。但此案 plugin.json 满分、capability-router 98 分，证明轻薄的 NL 层同样可以做到高质量——只要核心 commands 清晰，skill 示例补全，就能达标。轻薄不等于粗糙。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 是否有零示例
for f in $(find . -name "SKILL.md"); do
  example_count=$(grep -c "## Example\|<example\|\`\`\`" "$f" 2>/dev/null)
  if [ "$example_count" -eq 0 ]; then
    echo "ZERO EXAMPLES: $f"
  fi
done
```
命中后怎么办：为每个命中的 SKILL.md 添加至少一个 `## Example` block（输入 → 输出格式）。

```bash
# 检查 allowed-tools 中有没有声明但未使用的工具
# （以 WebFetch 为例）
for f in $(find .claude/commands -name "*.md"); do
  if grep -q "WebFetch" "$f" && ! grep -q "WebFetch\|fetch\|curl" "$f"; then
    echo "POSSIBLY UNUSED TOOL: $f"
  fi
done
```
命中后怎么办：从 frontmatter allowed-tools 中移除未使用的工具。

### 6.2 灵感 → 实施路径

1. **想法**：在项目 CLAUDE.md 加「不得推断操作」列表
   - **为何可行**：airis 的「推测不到的边界」是明确的负向约束，比正向指令更难被 AI 误解
   - **第一步**：找出项目中曾经发生「AI 自作主张」的场景，写成「禁止…」条目，加入 CLAUDE.md 的独立小节（10-20 分钟）

2. **想法**：将安装脚本中的 `curl | bash` 改为带 checksum 验证的安装
   - **为何可行**：如果我的任何项目有 install.sh，添加 SHA-256 checksum 验证是标准安全实践
   - **第一步**：检查项目 install/setup 脚本，确认下载后执行路径，改为先下载、验证 checksum、再执行（30-60 分钟）

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例核心目的**：按需启动的本地 MCP endpoint 管理工具，面向开发者日常 AI 工具链管理

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/- (WorldMonitor) | 低 | 均有后端服务 + NL 技能层 | WorldMonitor 是情报仪表盘，而非工具链管理 | 低 |
| MarkQWu/graphify | 低 | 均为开发者工具 | graphify 是知识图谱工具，非 MCP 管理 | 低 |
| MarkQWu/gstack | 中 | 均为 Claude Code 扩展工具集 | gstack 是角色工具集，非 MCP gateway | 中 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skills 无示例 | `grep -rL "Example\|example" /tmp/my-repos/MarkQWu-gstack/openclaw/skills/*/SKILL.md` | gstack 的 4 个 openclaw skills 各有 1-2 个示例，基本达标 | 低 |
| 模糊量词（appropriate/briefly） | `grep -rn "appropriate\|sufficient\|briefly" /tmp/my-repos/MarkQWu--/public/.well-known/agent-skills/*/SKILL.md` | 需人工验证；WorldMonitor skills 以动词开头描述，风险低 | 低 |

**结论**：此案的主要问题（curl|bash、skill 无示例）在我的项目中不突出复现。

### 8.3 别人的更优方案

1. **领域**：明确的「负向约束」声明
   - **本案做法**：CLAUDE.md 有「推测不到的边界」列表，7 条负向约束，精确到技术细节（如「不能用 `async for ... break` 关闭 httpx stream」）
   - **我的项目现状**：WorldMonitor/CLAUDE.md 和 graphify/AGENTS.md 主要描述正向行为，缺少「禁止推断」约束列表
   - **如何借鉴**：在 CLAUDE.md 加独立「## 禁止推断的操作」小节，列出 3-5 条场景具体的约束

### 8.4 反向：我的项目做得比他们好的地方

MarkQWu/- 的 skills 均位于 `public/.well-known/agent-skills/` 目录，所有 25 个 skill 都有 `name:` 字段，规范程度高于本案（本案无 agent，无此问题，但 WorldMonitor skills 的规范性可作为参考标杆）。

---

## 八、术语表

### <a name="MCP"></a>MCP（Model Context Protocol）
> Anthropic 推出的一种协议，让 Claude 可以连接外部工具服务器（如数据库、文件系统、浏览器自动化服务）。一个「MCP server」是一个提供工具接口的小服务，Claude Code 通过配置文件（`.mcp.json`）知道有哪些服务器可用。MCP gateway 是管理多个这类服务器的中间层。

### <a name="curl-pipe-bash"></a>curl | bash（管道执行风险）
> `curl -fsSL url | bash` 的意思是「从网上下载一段脚本，立刻执行」。危险在于：你执行的是**下载当时**服务器返回的内容，而不是你事先审查过的某个固定版本。如果仓库被攻击或域名被劫持，用户机器会执行恶意代码。

### <a name="checksum"></a>SHA-256 checksum（校验和）
> 对一个文件计算出的 64 位十六进制「数字指纹」。下载文件后对比官方公布的 checksum，若不匹配说明文件被篡改。`sha256sum file.tar.gz` 可以计算，`echo "官方值  file.tar.gz" | sha256sum --check` 可以验证。
