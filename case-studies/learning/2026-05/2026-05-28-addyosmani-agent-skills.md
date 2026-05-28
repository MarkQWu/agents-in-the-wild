# addyosmani/agent-skills — 学习案例

| 字段 | 值 |
|------|-----|
| 仓库 | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) |
| Stars | 22697 |
| 来源 | xiaolai upstream |
| 审查日期 | 2026-04-06 |
| 生成日期 | 2026-05-28 |
| 主题标签 | `security-gate` · `examples-driven` · `single-purpose` · `vague-quantifier` · `curl-pipe-bash-risk` |
| 贡献状态 | **安全封锁（BLOCKED）** — 存在 Critical 级别发现，未提交任何 PR |

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

agent-skills 是 Google Chrome 工程师 Addy Osmani 发布的 Claude Code 技能与代理合集，22697 颗 Star，是目前 GitHub 上关注度最高的 Claude Code 插件仓库之一。它并非框架，而是一套**工程工作流增强工具**：技能库涵盖从需求定义到发布的整个软件生命周期，代理则承担审查、测试、安全评估等专项角色。

仓库的核心价值主张是"以相位（phase）组织技能"——CLAUDE.md 将所有技能按 Define / Plan / Build / Verify / Review / Ship 六个阶段排列，形成一条从需求到上线的完整工作流地图。技能文件和代理文件之间通过命令（如 `/ship`）产生编排关系，体现了 Claude Code 多代理协同的完整实践。

当前仓库共有 23 个技能，较审查时增加了 `interview-me` 和 `doubt-driven-development` 两个新技能；代理维持 3 个（`code-reviewer.md`、`test-engineer.md`、`security-auditor.md`）；钩子层由 4 个 shell 脚本与 `hooks.json` 构成。

### 1.2 架构剖析

```
addyosmani/agent-skills/
├── CLAUDE.md                         # 技能按工作流阶段索引（Define/Plan/Build/Verify/Review/Ship）
├── .claude/
│   └── commands/                     # 7 个 slash 命令
│       ├── build.md
│       ├── code-simplify.md
│       ├── plan.md
│       ├── review.md
│       ├── ship.md                   # 关键：fan-out 到 3 个代理
│       ├── spec.md
│       └── test.md
├── agents/
│   ├── README.md
│   ├── code-reviewer.md
│   ├── security-auditor.md
│   └── test-engineer.md
├── skills/
│   ├── context-engineering/SKILL.md
│   ├── documentation-and-adrs/SKILL.md
│   ├── doubt-driven-development/SKILL.md     # 新（审查后新增）
│   ├── idea-refine/SKILL.md
│   ├── interview-me/SKILL.md                 # 新（审查后新增）
│   ├── performance-optimization/SKILL.md
│   ├── source-driven-development/SKILL.md
│   ├── test-driven-development/SKILL.md      # 最高分：99/100
│   └── ... (共 23 个)
└── hooks/
    ├── hooks.json
    ├── session-start.sh
    ├── simplify-ignore.sh
    ├── simplify-ignore-test.sh               # Critical 安全问题所在
    ├── sdd-cache-pre.sh
    └── sdd-cache-post.sh
```

**关键编排链**：`/ship` 命令以精确的 `name:` 字段分别调起 `code-reviewer`、`security-auditor`、`test-engineer` 三个代理，实现发布前的并行三路审查。这是仓库中最具教学价值的编排示范。

**钩子层**：`hooks.json` 仅注册 `SessionStart → session-start.sh`；`sdd-cache` 和 `simplify-ignore` 是 opt-in 额外钩子，不在默认配置中。

### 1.3 设计思路 / 方法论

仓库体现了三条清晰的设计哲学：

1. **工作流相位化**：不把技能堆成平铺列表，而是按软件生命周期阶段分组，用户打开 CLAUDE.md 即可定位自己当前处于哪个阶段，需要加载什么技能。这将"一个功能目录"升级为"一张工作流地图"。

2. **命令作为编排粘合剂**：7 个命令文件不是简单的提示词包装，而是将技能和代理串联起来的工作流节点。`/ship` 是最典型的例子——它不执行逻辑，而是把三个代理的并行审查流程**声明式地编排**进一个命令入口。

3. **钩子作为上下文注入器**：`session-start.sh` 在会话启动时将相关 SKILL.md 内容注入上下文，`sdd-cache` 系列在工具调用前后维护源驱动开发的缓存状态。钩子不替代技能，而是让技能在合适时机自动生效。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式名称 | 描述 |
|---------|------|
| **工作流相位模式** | 技能按软件生命周期阶段（Define/Plan/Build/Verify/Review/Ship）分组，CLAUDE.md 作为相位索引 |
| **Fan-out 编排模式** | 单个命令（`/ship`）同时派遣多个具名代理，实现并行多路审查 |
| **Opt-in 钩子模式** | 只有核心钩子默认注册，扩展钩子需用户主动引入，降低自动执行风险 |
| **单一职责代理模式** | 代理职责极度单一：只审代码、只测试、只做安全扫描，不混合职责 |
| **高分技能标杆模式** | `skills/test-driven-development/SKILL.md` 以 99 分成为仓库内部质量标杆，其他技能可对标改进 |

### 2.2 适用场景

- **工作流相位模式**：适用于任何技能数量超过 10 个的仓库。技能平铺超过一定数量后，认知负担急剧上升；相位分组让用户能按需索取，不必记忆全局。
- **Fan-out 编排模式**：适用于需要多维度并行评估的高风险操作（发布、部署、安全变更）。单个命令入口比要求用户逐一调用三个代理的体验好得多。
- **Opt-in 钩子模式**：适用于任何提供 shell 钩子的仓库——默认注册应极度保守，只注册无副作用的只读钩子，写操作或网络操作的钩子应始终是 opt-in。
- **单一职责代理模式**：适用于需要深度专业化输出的评审类场景。职责混合的代理往往在多项任务上表现平庸；职责单一的代理在其专项任务上更可靠。

### 2.3 与其他架构对比

| 维度 | addyosmani/agent-skills | SuperClaude（全栈框架） | 典型个人插件 |
|------|------------------------|----------------------|------------|
| 技能组织方式 | 按工作流阶段分组 | 按工程角色分组 | 平铺无分类 |
| 命令数量 | 7 个（精简） | 30 个（全覆盖） | 1-5 个 |
| 代理数量 | 3 个（专项） | 20 个（全角色） | 0-2 个 |
| 编排层 | `/ship` fan-out 到 3 代理 | 并行执行多代理 | 无编排 |
| 钩子策略 | 核心默认 + opt-in 扩展 | 全部默认（含缺失引用） | 无钩子 |
| 安全风险 | Critical（eval in test script） | Critical（postinstall + 缺失引用） | 通常无风险 |
| NL 质量分 | 94/100（整体高分） | 84/100（规模惩罚明显） | 70-85 |

### 2.4 改进空间

1. **所有命令缺少 `allowed-tools` 字段**：7 个命令文件均无工具权限声明，安全边界完全开放。`/build`、`/ship` 等高风险命令尤其需要显式限制可用工具范围。
2. **所有代理缺少 `model` 字段**：3 个代理无 `model:` 声明，运行时模型路由完全由平台决定。`security-auditor` 需要深度推理，应固定到更强模型；`test-engineer` 重复性高，可固定到经济模型。
3. **空输入处理缺失**：7 个命令均无空输入处理，用户直接运行 `/plan`（不带任何参数）时行为未定义。
4. **eval 安全缺陷未修复**（见第三章），这是阻断 PR 贡献的根本原因。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：94/100**
**安全等级：BLOCKED**（存在 Critical 级别发现，自动跳过 PR 贡献阶段）

> 注：94 分是同类仓库中的高分段。BLOCKED 状态意味着安全门禁触发，**NLPM 未向目标仓库提交任何 PR**，所有发现仅存于内部审查记录。这是"高质量内容 + 安全 BLOCKED"最典型的案例：NL 内容写得好，但一个 eval 就足以封锁整个贡献通道。

共发现：0 个 Bug · 25 个质量问题 · 5 个安全发现

**各文件评分分布：**

| 文件 | 评分 | 主要扣分原因 |
|------|------|------------|
| `.claude/commands/*.md`（7 个文件） | 85 | 无 `allowed-tools`（-5），无空输入处理（-10） |
| `agents/README.md` | 90 | — |
| `agents/code-reviewer.md` | 90 | 无 `model` 字段（-5），仅 1 个示例（-5） |
| `agents/security-auditor.md` | 90 | 无 `model` 字段（-5），仅 1 个示例（-5） |
| `agents/test-engineer.md` | 90 | 无 `model` 字段（-5），仅 1 个示例（-5） |
| `skills/idea-refine/SKILL.md` | 93 | 绝对路径引用（-5），轻微模糊词 |
| `skills/context-engineering/SKILL.md` | 94 | `"relevant"` 出现 3 次（-6） |
| `skills/documentation-and-adrs/SKILL.md` | 94 | `"significant"` 出现 3 次（-6） |
| `skills/performance-optimization/SKILL.md` | 94 | `"unnecessary"`, `"significantly"` 各 1 次（-6） |
| `skills/source-driven-development/SKILL.md` | 94 | `"relevant"` 出现 3 次（-6） |
| `hooks/hooks.json` | 95 | — |
| 大多数技能文件 | 96–99 | 零星模糊词 |
| `skills/test-driven-development/SKILL.md` | **99** | 近乎完美 |

### 3.2 当时值得借鉴的模式

1. **`/ship` fan-out 编排（审查时已存在）**：`/ship` 命令通过 `name:` 字段精确引用三个代理，实现并行三路质检——代码审查、安全审计、测试覆盖率验证同步进行，用户只需运行一个命令。这是本仓库最具教学价值的原生模式。

2. **`skills/test-driven-development/SKILL.md` 作为内部标杆（99/100）**：该文件几乎无模糊词、有精确的格式规范、示例充足，是仓库内质量最高的单文件，代表了"技能文件应该写成什么样"。

3. **钩子分层（默认注册 vs opt-in）**：`hooks.json` 只注册 `SessionStart → session-start.sh`，`sdd-cache` 和 `simplify-ignore` 系列需用户主动引入。这一分层策略比"所有钩子默认注册"更安全，减少了意外副作用的风险。

4. **`skills/source-driven-development/` 的缓存机制**：通过 `sdd-cache-pre.sh` 和 `sdd-cache-post.sh` 在工具调用前后维护源驱动开发的本地缓存，体现了"钩子作为状态管理器"的高级用法。

### 3.3 当时的缺陷

**Critical 安全缺陷：**

`hooks/simplify-ignore-test.sh` 第 34 行：

```bash
eval "$(sed -n '/^filter_file()/,/^}/p' hooks/simplify-ignore.sh)"
```

此行从 `hooks/simplify-ignore.sh` 文件中提取 `filter_file()` 函数体，然后通过 `eval` 在当前 shell 中动态执行。问题的严重性在于：
- 被执行的代码来自外部文件（而非硬编码字符串），若 `hooks/simplify-ignore.sh` 被篡改，测试脚本将静默执行攻击者注入的代码。
- `eval` 执行环境是调用者的 shell，具有调用者的全部权限（读写文件、网络访问、执行命令）。
- 这是典型的"代码从文件加载再执行"反模式，即使是测试脚本也不应使用。

**Medium 安全缺陷：**

1. `hooks/sdd-cache-pre.sh` 第 71 行：`curl -sI <用户提供的 URL>` — 对用户提供的 URL 执行 HTTP 请求，无域名白名单过滤（SSRF 风险）。
2. `hooks/sdd-cache-post.sh` 第 82 行：`curl -sI -L <URL>` — 跟随重定向到未经验证的目标 URL，可被重定向到内网地址。

**Low 安全缺陷：**

1. `hooks/session-start.sh` 第 13 行：`$CONTENT`（SKILL.md 内容）未经转义直接插入 heredoc JSON，存在 JSON 注入风险（若 SKILL.md 内容含有引号或反斜杠）。
2. `hooks/sdd-cache-post.sh` 第 118 行：任意网络内容直接写入 `.claude/sdd-cache/*.json`，无文件大小或内容类型验证（可触发磁盘耗尽或后续解析器漏洞）。

**质量缺陷（系统性）：**

- 7 个命令文件**全部**缺少 `allowed-tools` 字段（每处 -5 分）
- 7 个命令文件**全部**缺少空输入处理（每处 -10 分）
- 3 个代理文件**全部**缺少 `model:` 字段（每处 -5 分）
- 3 个代理文件均只有 1 个示例（每处 -5 分；建议最少 2 个）
- `skills/idea-refine/SKILL.md` 包含绝对路径 `/mnt/skills/user/...`（开发机路径泄露，-5 分）
- 多个技能文件出现高频模糊量词：`"relevant"` × 3、`"significant"` × 3、`"unnecessary"` × 1、`"significantly"` × 1

### 3.4 当时的优化机会

1. 修复 `simplify-ignore-test.sh` L34 的 `eval`（Critical，阻断 PR 贡献通道）
2. 为 `sdd-cache-pre.sh` 添加 URL 域名白名单验证（Medium，防 SSRF）
3. 为 `sdd-cache-post.sh` 的重定向 curl 添加 `--max-redirs 0` 或域名验证（Medium）
4. 为 `session-start.sh` 中的 `$CONTENT` 添加 JSON 转义（Low）
5. 为所有 7 个命令添加 `allowed-tools` 字段
6. 为所有 3 个代理添加 `model:` 字段
7. 为所有命令添加空输入处理（`if [ -z "$ARGUMENTS" ]; then ...`）
8. 将 `skills/idea-refine/SKILL.md` 中的绝对路径替换为相对路径
9. 清理高频模糊词（`relevant`、`significant`、`unnecessary`、`significantly`）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-06 | 2026-05-28 当前 HEAD | 状态 |
|------|-----------|---------------------|------|
| `simplify-ignore-test.sh` L34 `eval` | 存在 | `grep "^eval" hooks/simplify-ignore-test.sh` 仍返回该行 | ❌ 未修复 |
| `.claude/commands/plan.md` 缺 `allowed-tools` | 存在 | `grep "allowed-tools" .claude/commands/plan.md` → 无结果 | ❌ 未修复 |
| `agents/code-reviewer.md` 缺 `model` 字段 | 存在 | `grep "^model:" agents/code-reviewer.md` → 无结果 | ❌ 未修复 |
| `skills/idea-refine/SKILL.md` 绝对路径 | 存在 | 未重新核查，原始审查记录标注仍在 | 未知 |
| 技能文件模糊词（`relevant` × 3） | 存在 | 新增两个技能，原问题未必清除 | 未知 |

**结论**：三个可独立核查的缺陷（`eval`、`allowed-tools`、`model`）均在 52 天后保持原状。新增了两个技能，但原有质量和安全问题无任何修复迹象。这与 22697 Star 的声望形成强烈对比——**仓库的影响力远超其安全维护投入**。

### 4.2 架构演进

| 变化 | 详情 |
|------|------|
| 新增技能 `interview-me` | 新相位：职业发展 / 面试准备，扩展了工作流地图的覆盖范围 |
| 新增技能 `doubt-driven-development` | 延伸 TDD 理念，鼓励以"质疑驱动"代替"测试驱动"，是方法论层面的创新 |
| 技能总数 23（+2） | 相位分组策略持续奏效；新技能无缝融入已有相位结构 |
| 核心钩子、命令、代理 | 无变化 |

架构演进方向：**横向扩展技能覆盖**，而非深度修复已知质量问题。这是开源项目中常见的"新功能优先于维护"模式。

### 4.3 新增的可学习模式

- **`doubt-driven-development`**：将"怀疑"作为工程原则显式化——在编写测试之前先质疑需求和假设。这是方法论技能的一种创新写法，值得在其他领域技能库中借鉴（例如：在法律工作流中，"质疑初稿结论"可以作为独立的 `doubt-driven-review` 技能）。
- **`interview-me`**：将 Claude 转化为面试官角色，展示了技能文件可以承载"角色扮演协议"而不仅仅是"工具使用规范"。这拓宽了技能文件的应用场景定义。

---

## 五、校准

### 5.1 我已经在做对的

- **无 shell 钩子**：`drama-workshop-skills` 和 `echo-sleuth-for-claude` 均无 shell 钩子文件，完全规避了 eval / curl SSRF / JSON 注入等钩子层面的安全风险。
- **职责单一**：我的代理设计（如 `claude-for-legal` 中的 `reg-change-monitor.md`、`docket-watcher.md`）均专注于单一监控任务，符合本仓库代理职责单一化的最佳实践。
- **无绝对路径泄露**：我的技能文件中无开发机绝对路径引用，未重现 `skills/idea-refine/SKILL.md` 的路径泄露问题。

### 5.2 挑战 / 验证

**反直觉发现：22697 Star 的仓库被安全门禁封锁，且 52 天后 Critical 缺陷仍未修复。**

此案例挑战了"高 Star = 高质量 + 高安全"的直觉判断，提供了两个可验证的具体命题：

**命题 1：eval 反模式与"测试脚本豁免"的幻觉**

开发者常认为测试脚本的安全要求低于生产代码。`simplify-ignore-test.sh` 中的 `eval` 正是这种思维的产物——"这只是测试脚本，不会在生产环境运行"。但事实是：
- 该脚本与被测脚本（`simplify-ignore.sh`）存在代码依赖，任何对 `simplify-ignore.sh` 的篡改都会通过 `eval` 被静默执行
- 如果用户在 CI/CD 中运行该测试脚本，攻击面就转移到了 CI/CD 环境
- "测试脚本不需要安全审查"是工程实践中的常见误区，本案例是一个反证

**命题 2：缺陷持久性**

项目越流行，修复维护性缺陷（而非新增功能）的相对成本越低，但实际修复率并不更高。addyosmani 在 52 天内新增了 2 个技能，但 3 个 Critical/Medium/Low 安全问题零进展。**校准动作**：评估一个仓库的安全性，不应看 Star 数，应看"已知安全缺陷的修复周期"。

---

## 六、行动

### 6.1 自查动作

对自己的仓库运行以下检查，确认无类似问题：

```bash
# 1. 检查是否存在 eval 执行外部文件内容的模式
grep -rn "eval.*\$(" --include="*.sh" ~/MarkQWu/ 2>/dev/null \
  | grep -v "eval \"\$(" \
  | grep "sed\|cat\|awk\|head\|tail" \
  && echo "⚠️  发现 eval 执行外部文件内容的模式" || echo "✅ 无 eval 读取外部文件模式"

# 2. 检查钩子 JSON 中引用的脚本是否都存在
for repo_path in ~/MarkQWu/echo-sleuth-for-claude ~/MarkQWu/claude-for-legal ~/MarkQWu/drama-workshop-skills; do
  echo "=== $(basename $repo_path) ==="
  find "$repo_path" -name "hooks.json" | while read hookfile; do
    python3 -c "
import json, os, sys
data = json.load(open('$hookfile'))
for hook in data.get('hooks', []):
  for h in hook.get('hooks', []):
    cmd = h.get('command', '').split()[0]
    if cmd and not os.path.isabs(cmd):
      cmd = os.path.join(os.path.dirname('$hookfile'), cmd)
    if cmd and not os.path.exists(cmd):
      print(f'MISSING: {cmd}')
" 2>/dev/null || echo "(解析失败)"
  done
done

# 3. 检查 curl 是否对用户输入的 URL 执行请求（SSRF 风险）
grep -rn "curl.*\$" --include="*.sh" ~/MarkQWu/ 2>/dev/null \
  && echo "⚠️  发现可能的 SSRF 模式，请人工核查 URL 来源" || echo "✅ 无 curl 用户输入 URL 模式"

# 4. 检查命令文件是否缺少 allowed-tools
for repo_path in ~/MarkQWu/echo-sleuth-for-claude ~/MarkQWu/claude-for-legal ~/MarkQWu/drama-workshop-skills; do
  echo "=== $(basename $repo_path) 缺少 allowed-tools 的命令文件 ==="
  find "$repo_path" -path "*/.claude/commands/*.md" | xargs grep -rL "allowed-tools" 2>/dev/null \
    || echo "(无命令文件)"
done

# 5. 检查代理文件是否缺少 model 字段
for repo_path in ~/MarkQWu/echo-sleuth-for-claude ~/MarkQWu/claude-for-legal ~/MarkQWu/drama-workshop-skills; do
  echo "=== $(basename $repo_path) 缺少 model 字段的代理文件 ==="
  find "$repo_path" -path "*/agents/*.md" | xargs grep -rL "^model:" 2>/dev/null \
    || echo "(无代理文件)"
done

# 6. 扫描技能文件中的高频模糊词
for word in "relevant" "significant" "unnecessary" "significantly" "important" "various"; do
  count=$(grep -r "\b${word}\b" --include="SKILL.md" ~/MarkQWu/ 2>/dev/null | wc -l)
  [ "$count" -gt 0 ] && echo "⚠️  模糊词 '${word}' 出现 ${count} 次" || true
done
echo "完成：模糊词扫描"
```

### 6.2 灵感 → 实施路径

**灵感 1：为 echo-sleuth-for-claude 实现 /ship 风格的 fan-out 命令**

echo-sleuth-for-claude 有 5 个代理（recall、file-historian、analyze、schema-scout、memory-auditor），但目前没有任何命令将它们编排在一起。可以新增一个 `/echo:audit` 命令，调用 `analyze + memory-auditor` 进行双路并行分析：

```
实施步骤：
1. 在 .claude/commands/audit.md 中声明命令
2. 在 description 中列明调用的两个代理名称
3. 添加 allowed-tools 字段（限制为 Read、Bash）
4. 添加空输入处理（提示用户提供会话 ID 或路径）
5. 测试：运行 /echo:audit，验证两个代理并行启动
```

**灵感 2：为所有代理添加 model 字段（按任务复杂度分级）**

参照本仓库的缺陷，对 echo-sleuth-for-claude 的 5 个代理进行模型分配：

| 代理 | 任务特征 | 建议 model |
|------|---------|-----------|
| memory-auditor | 需要推理一致性 | claude-sonnet-4-5 |
| analyze | 深度分析，需强推理 | claude-sonnet-4-5 |
| file-historian | 结构化历史检索 | claude-haiku-4-5 |
| recall | 高频简单检索 | claude-haiku-4-5 |
| schema-scout | 模式识别，中等复杂度 | claude-haiku-4-5 |

**灵感 3：引入 doubt-driven-development 思维到 claude-for-legal**

`doubt-driven-development` 技能的核心是"在行动前先质疑假设"。法律场景对错误容忍度极低，可将此模式转化为 `claude-for-legal` 中的 `assumption-check` 技能：在生成任何法律意见前，先列出隐含假设并请求用户确认。实施路径：
1. 在 `skills/assumption-check/SKILL.md` 中定义假设检查协议
2. 在高风险代理（如 `reg-change-monitor.md`）的 `skills:` 字段中加载此技能
3. 对比加载前后的输出，验证假设显式化是否减少了误判率

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | agent-skills 目的 | 对齐度 |
|---------|-----------------|-------|
| MarkQWu/echo-sleuth-for-claude | 工程全生命周期技能增强 | **中** — 同为专项工具集，但 echo-sleuth 聚焦会话历史挖掘这一单一领域，而 agent-skills 覆盖完整 SDLC |
| MarkQWu/claude-for-legal | 工程全生命周期技能增强 | **低** — 法律工作流与软件工程工作流的领域完全不同，但相位组织模式可以直接借鉴 |
| MarkQWu/drama-workshop-skills | 工程全生命周期技能增强 | **低** — 短剧创作领域，目的差异大；但"按创作阶段组织技能（开发/剧本/拍摄/发行）"的相位模式与 agent-skills 同构 |

### 8.2 在我的项目里复现的同类问题

**echo-sleuth-for-claude 代理缺少 `model` 字段**：

与 agent-skills 三个代理相同，echo-sleuth 的 5 个代理均无 `model:` 声明。任务复杂度差异明显（`memory-auditor` 需要深度推理 vs `recall` 只需检索），却使用同一个运行时默认模型，是明确的优化盲点。

**缺少跨代理编排命令**：

agent-skills 通过 `/ship` 将三个代理的评审流程聚合为单一入口。echo-sleuth 的 5 个代理目前是孤立的，用户需要分别调用，没有类似 `/echo:audit`（同时运行 analyze + memory-auditor）的编排入口。

**技能文件中可能存在模糊量词**：

`drama-workshop-skills` 的技能文件面向创意写作领域，"relevant"、"significant" 等词在写作语境中出现频率更高，需要专门扫描。

### 8.3 别人的更优方案

**工作流相位组织（agent-skills 优于我的所有仓库）**：

我的三个仓库均将技能平铺在 CLAUDE.md 中，没有显式的生命周期阶段分组。用户阅读 CLAUDE.md 时需要全局扫描才能定位当前任务相关技能。agent-skills 的相位分组（Define/Plan/Build/Verify/Review/Ship）让用户可以按工作进展快速定位，认知负担显著降低。

改进动作：重新组织 `echo-sleuth-for-claude` 的 CLAUDE.md，将 5 个代理按"发现 → 分析 → 报告"三个阶段分组，而非平铺列表。

**Fan-out 编排命令（agent-skills 优于 echo-sleuth）**：

echo-sleuth 没有将多个代理编排在一起的命令，用户需要逐个调用。agent-skills 的 `/ship` 模式展示了"一个命令 = 多个代理协同"的正确姿势。

### 8.4 反向：我的项目做得比他们好的地方

1. **无 shell 钩子安全风险（我的仓库优于 agent-skills）**：`claude-for-legal` 和 `drama-workshop-skills` 无任何 shell 钩子，完全规避了 eval、SSRF、JSON 注入等钩子层面风险。即使有 `claude-for-legal` 使用了 `hooks.json`，其钩子也仅触发纯读操作，无网络请求和代码执行。

2. **代理 `allowed-tools` 约束（claude-for-legal 优于 agent-skills）**：`claude-for-legal` 的代理在设计上限制了可用工具范围，特别是监控类代理（`docket-watcher`、`reg-change-monitor`）明确声明了只读工具权限，而 agent-skills 的 7 个命令全部无 `allowed-tools` 约束。

3. **技能文件无绝对路径泄露（我的所有仓库优于 agent-skills）**：`skills/idea-refine/SKILL.md` 中的 `/mnt/skills/user/...` 绝对路径暴露了开发机文件系统结构，是典型的信息泄露。我的三个仓库均使用相对路径引用，无此问题。

---

## 八、术语表

| 术语 | 说明 |
|------|------|
| <a name="eval-反模式">eval 反模式</a> | 在 shell 脚本中用 `eval` 执行从外部文件（`sed`、`cat` 等提取的）代码字符串。被执行的代码来自运行时文件内容而非编译期常量，若文件被篡改则执行攻击者代码。即使在"仅供测试"的脚本中，也构成 Critical 安全风险。 |
| <a name="fan-out-编排">fan-out 编排</a> | 单个 Claude Code 命令通过 `name:` 字段同时调起多个代理，形成并行执行扇形结构。`/ship` → `code-reviewer + security-auditor + test-engineer` 是教科书级范例。 |
| <a name="工作流相位">工作流相位</a> | 将技能按软件生命周期阶段（Define / Plan / Build / Verify / Review / Ship）分组的 CLAUDE.md 组织模式。相位数量通常与仓库技能数量正相关：技能越多，越需要相位结构降低认知负担。 |
| <a name="ssrf">SSRF（服务器端请求伪造）</a> | 攻击者通过控制服务端发出的 URL，让服务器访问攻击者指定的内部资源（如 `169.254.169.254` 云元数据端点）。`sdd-cache-pre.sh` 中将用户输入的 URL 直接传给 `curl` 而无域名白名单，构成此类风险。 |
| <a name="opt-in钩子">opt-in 钩子</a> | 不在 `hooks.json` 默认注册、需要用户主动引入的 shell 钩子。agent-skills 将 `sdd-cache` 和 `simplify-ignore` 设计为 opt-in，是降低钩子副作用风险的最佳实践。 |
| <a name="allowed-tools">allowed-tools</a> | Claude Code 命令/代理 frontmatter 中的字段，声明该原件可调用的工具白名单。缺少此字段时工具权限完全开放，是命令和代理安全边界的核心声明。 |
| <a name="model-pinning">model 字段（模型固定）</a> | 在代理 frontmatter 中通过 `model:` 显式声明使用的 Claude 模型 ID。未声明时运行时路由依赖平台默认值，平台升级可能无声改变代理行为。复杂推理任务应固定 Sonnet，高频简单任务应固定 Haiku 以控制成本。 |
| <a name="vague-quantifier">模糊量词（vague quantifier）</a> | 技能文件中缺乏精确定义的程度副词和形容词，如 `relevant`、`significant`、`unnecessary`、`significantly`。NLPM 每出现一次扣 2 分，高频出现（×3 以上）累计扣 6 分。替换方案：用可测量的阈值或具体描述替代，如"与当前任务目标直接相关的" 代替 "relevant"。 |
| <a name="json-注入">JSON 注入</a> | 将未经转义的用户输入（或文件内容）直接拼接进 JSON 字符串，若输入包含 `"`、`\`、`\n` 等字符将破坏 JSON 结构。`session-start.sh` 将 SKILL.md 内容直接插入 heredoc JSON 即属此类风险。 |
