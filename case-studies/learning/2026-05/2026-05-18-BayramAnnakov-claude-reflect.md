# BayramAnnakov/claude-reflect — 学习案例

**仓库**：https://github.com/BayramAnnakov/claude-reflect
**Stars**：1000 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-18（基于当前 HEAD）
**主题标签**：experience-accumulation, manifest-discipline, security-gate, single-purpose, fallback-chain

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
claude-reflect 是一个「AI 自学习系统」：通过 4 个自动触发的 hook 检测用户纠错信号（「不，用 X」「实际上…」），将纠错队列化，然后由 `/reflect` 命令在人工审核后写入 CLAUDE.md。这是目前生态中最完整的「跨会话记忆工程」实现之一，1000 颗星验证了市场需求。作者 Bayram Annakov 以 v3.1.0 的版本号和 Python stdlib-only 脚本库体现了认真的工程投入。

### 1.2 架构剖析

```
claude-reflect/
├── SKILL.md              # 主知识文件（所有概念说明）
├── commands/             # 6 个 command（reflect, reflect-skills, skip-reflect, view-queue, ...）
├── hooks/
│   └── hooks.json        # 4 个 hook（SessionStart, UserPromptSubmit, PreCompact, PostToolUse:Bash）
├── scripts/
│   ├── capture_learning.py        # UserPromptSubmit → 检测纠错信号
│   ├── check_learnings.py         # PreCompact → 提醒用户
│   ├── post_commit_reminder.py    # PostToolUse:Bash git commit → 提醒 reflect
│   ├── session_start_reminder.py  # SessionStart → 恢复未处理队列
│   ├── extract_session_learnings.py
│   ├── extract_tool_errors.py
│   ├── extract_tool_rejections.py
│   ├── compare_detection.py
│   ├── read_queue.py
│   └── lib/
│       ├── reflect_utils.py       # 多目标文件发现工具
│       └── semantic_detector.py   # 语义分类（调用 Claude CLI）
├── scripts/legacy/       # 已弃用 bash 版本（保留供参考）
└── .claude-plugin/
    └── plugin.json
```

- **文件类型分布**：0 个 agent / 6 个 command / 1 个 skill / 4 个 hook / 12 个 Python 脚本
- **编排关系**：Hook → Script（自动捕获） → 队列文件 → Command（人工审核） → CLAUDE.md；两阶段设计（自动采集 vs 人工确认）是核心架构决策
- **跨件契约**：hooks.json → scripts/*.py（`${CLAUDE_PLUGIN_ROOT}` 路径协议）；scripts/lib/ 提供 `find_claude_files()` 工具函数，被多个 command 复用；队列文件 `~/.claude/learnings-queue.json` 是跨组件共享状态

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「捕获不打扰，确认有掌控」——自动捕获做到透明无感，人工确认保留用户主权；Stage 1（自动）与 Stage 2（手动）的清晰分离体现了对「AI 自主性边界」的深刻思考
- **解决什么问题**：每次会话结束后，用户对 Claude 的纠错行为就消失了；没有持久化机制，Claude 下次会话重蹈覆辙
- **Trade-off**：语义检测精度 vs 成本——`semantic_detector.py` 调用 Claude CLI 本身做语义分析，是「用 AI 解决 AI 行为问题」的递归设计，但引入了 API 调用成本和提示注入风险
- **认知模型**：作者将 CLAUDE.md 视为「AI 的程序性记忆」，将纠错信号视为「用户对 AI 的反馈训练数据」，这是一种把 Claude Code 当作可持续学习系统来运营的世界观

---

## 二、过去审查发现（2026-04-06 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **99/100**，Security: CLEAR（1 个 MEDIUM + 2 个 LOW）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/reflect.md | 94/100 | 3 个模糊量词（-6） |
| commands/view-queue.md | 97/100 | Read 声明但未使用（-3） |
| commands/reflect-skills.md | 98/100 | 模糊量词（-2） |
| commands/skip-reflect.md | 100/100 | 无 |
| SKILL.md | 100/100 | 无 |
| hooks/hooks.json | 100/100 | 无 |
| .claude-plugin/plugin.json | 100/100 | （跨件问题单独标注） |

### 2.2 当时值得借鉴的模式

1. **双阶段自学习架构** → 捕获（自动 hook）+ 处理（人工 command）的分离是 99/100 高分的基础。这个模式适用于任何「实时采集 + 批量处理」场景，避免了自动写入带来的信任问题 → 借鉴点：我的记忆相关 skill 可以引入「队列化」而非「即时写入」
2. **Python stdlib-only 脚本** → 无任何 pip 依赖，确保在任何有 Python 3.11+ 的环境即可运行。这是一个约束即设计的决策——牺牲第三方库的便利性，换取零依赖安装体验 → 适用于面向广泛用户的插件脚本
3. **多目标导出（Multi-Target Export）** → reflect.md 中定义了完整的记忆写入目标层级（Global CLAUDE.md / Project CLAUDE.md / CLAUDE.local.md / 子目录 CLAUDE.md / Rules / AGENTS.md），通过 `find_claude_files()` 动态发现，而非硬编码路径 → 这是「约定优于配置」的优秀示范
4. **Legacy 保留策略** → bash 脚本不删除而是移到 `scripts/legacy/`，并标注 DEPRECATED。这比直接删除更友好——用户可以查看算法演化，开发者可以回滚参考 → 版本迁移的文明做法
5. **CLAUDE_PLUGIN_ROOT 约定**：大多数 command 使用 `${CLAUDE_PLUGIN_ROOT}` 而非绝对路径，保证插件在不同安装路径下可以正常运行

### 2.3 当时的缺陷

1. **plugin.json 缺 `hooks` 字段** → 这是 HIGH 级别 bug：plugin.json 中没有指向 hooks/hooks.json 的字段，但 CLAUDE.md 明确说「manifest 指向 hooks」。根本原因：作者可能假设 Claude Code 能自动发现 hooks 目录，但这一行为并未在文档中明确保证，依赖隐式约定会在不同版本的 Claude Code 中产生不一致行为。**自查：我的 plugin.json 是否明确声明了 hooks 路径？**
2. **reflect.md 中使用 `$0` 而非 `CLAUDE_PLUGIN_ROOT`** → context 注入行 `!python3 "$(dirname "$(dirname "$(readlink -f "$0")")")/scripts/read_queue.py"` 依赖 `$0` 解析插件根目录。在 Claude Code 的 `!` 上下文中 `$0` 通常是 shell 解释器路径，不是技能文件路径，导致静默失败（通过 `|| echo "[]"` 掩盖错误）。**自查：我的 context injection 是否使用了 CLAUDE_PLUGIN_ROOT 约定？**
3. **vague quantifier 三连** → reflect.md 中「appropriate heading」「relevant section header」「make semantic sense」三处模糊量词均在描述 learning 写入位置的决策逻辑，根本原因是作者用自然语言表达了一个需要枚举决策树的场景。**自查：我的 command 中有没有用「appropriate/relevant/makes sense」描述应该用枚举覆盖的场景？**

### 2.4 当时的优化机会

1. 在 plugin.json 中添加 `"hooks": "../hooks/hooks.json"` 字段（或查阅 Claude Code 最新 plugin schema 确认正确字段名）
2. 将 reflect.md 中的 `$0` 路径解析替换为 `${CLAUDE_PLUGIN_ROOT}/scripts/read_queue.py`，与 skip-reflect.md 保持一致
3. 将「写入哪个章节」的模糊描述替换为决策表：「如果 learning 类型是 X，写入第 Y 节；如果是 Z，写入第 W 节」

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 缺 `hooks` 字段 | `cat .claude-plugin/plugin.json \| python3 -c "import json,sys; print(json.load(sys.stdin).get('hooks','MISSING'))"` | **仍缺失**（hooks 字段未添加） | 7 周后仍未修复；说明作者可能经过验证确认 Claude Code 能自动发现 hooks，或者用户未反馈此问题 |
| reflect.md 使用 `$0` | `grep "\$0" commands/reflect.md` | **仍存在**（第 20 行，与 audit 时完全相同） | 因为有 `|| echo "[]"` 静默回退，用户感知不到失败，所以不会被报告为 bug |
| view-queue.md 多余的 Read 声明 | `grep "allowed-tools" commands/view-queue.md` | **已修复**：当前为 `allowed-tools: Bash, Read`（audit 时建议移除 Read，但实际保留了——可能是有意为之） | 有趣：未完全按建议修复，而是保留了 Read，暗示作者可能在计划添加 Read 用法 |

### 3.2 架构演进

当前 HEAD 相比 audit 时的结构基本稳定，无重大重组。版本号从未记录的状态升至 v3.1.0，说明有持续迭代。SKILL.md 现在记录了更完整的 Multi-Target Export 表格，包含 AGENTS.md（Codex/Cursor/Aider 行业标准格式）的导出目标——这是 audit 后新增的跨工具兼容性特性，体现作者认识到 Claude Code 只是 AI 编程工具生态的一部分。

### 3.3 新增的可学习模式

当前 SKILL.md 中「Target Discovery」章节的 Python 示例代码（`find_claude_files()` 调用）是 audit 后补充的，展示了如何用代码示例内联在 SKILL.md 中说明工具调用方法——这是一种「代码即文档」的 skill 写法：不是描述「如何找文件」，而是直接给出可执行的 Python 代码。

---

## 四、校准

### 4.1 我已经在做对的
- 两阶段人机协作：我的工作流同样区分「自动采集」和「人工确认」，避免 AI 擅自修改重要文件
- CLAUDE_PLUGIN_ROOT 约定：我的脚本引用使用环境变量而非硬编码路径
- Python stdlib only：我的辅助脚本避免非必要的第三方依赖
- 多目标记忆：我知道 CLAUDE.md / rules / AGENTS.md 是不同层次的记忆载体

### 4.2 挑战 / 验证

这次 audit 验证了一个直觉：**「静默失败比显式报错更危险」**。`$0` 路径解析失败后 `|| echo "[]"` 掩盖了错误，用户看到「空队列」而非报错——这是一种认知欺骗。99/100 的高分掩盖了这个运行时缺陷，提示了「高 NL 分数 ≠ 无运行时 bug」。NLPM 评分系统的有效范围是静态分析，无法替代实际运行测试。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查我的 command 中是否使用了 $0 进行路径解析
grep -rn '\$0' ~/.claude/commands/ .claude/commands/ 2>/dev/null
# 命中后：替换为 ${CLAUDE_PLUGIN_ROOT}/<相对路径> 或 $(dirname "$(readlink -f "${BASH_SOURCE[0]}")")

# 检查 plugin.json 是否声明了 hooks 路径
cat .claude-plugin/plugin.json 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); print('OK' if 'hooks' in d else 'MISSING hooks field')"
# 命中 MISSING 后：添加 "hooks": "路径/hooks.json" 字段
```

```bash
# 检查我的 command 是否用模糊语言描述「写入位置」决策
grep -rn -E '"(appropriate|relevant|makes? sense|suitable)" ' ~/.claude/commands/ 2>/dev/null
# 命中后：将该段落改为枚举决策表（如果 X 则 Y；如果 Z 则 W）
```

### 5.2 灵感 → 实施路径

1. **想法**：实现一个轻量版「纠错队列」机制，用 PostToolUse hook 记录用户否定信号
   - **为何可行**：claude-reflect 的 1000 颗星证明用户愿意为「AI 记住自己的偏好」付费；核心机制（检测「No,」「Actually,」→ 写入 JSON）本身不复杂，Python stdlib 可完成
   - **第一步**：创建 `scripts/capture_correction.py`，解析 hook 输入中的用户消息，用正则匹配否定信号模式（`r'\b(no,? use|actually[,.]|don\'t|instead use)\b'`）；将匹配结果追加到 `.claude/corrections-queue.json`；10 行代码，30 分钟可完成

2. **想法**：为我的 CLAUDE.md 建立「分层记忆架构」（参考 Multi-Target Export 表格）
   - **为何可行**：目前我用单一 CLAUDE.md 存放所有规则，随着规则增长会变臃肿；claude-reflect 的分层设计（global / project / local / rules/）提供了清晰的信息架构模板
   - **第一步**：将当前 CLAUDE.md 中的项目特定规则移到 `.claude/rules/<category>.md`；Global 级别的规则保留在 `~/.claude/CLAUDE.md`；创建索引说明每层的职责；1 小时可完成
