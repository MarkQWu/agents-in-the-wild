# AgriciDaniel/claude-obsidian — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-obsidian
**Stars**：976 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-05-17（基于审计数据）
**主题标签**：security-gate, cross-reference, vague-quantifier, experience-accumulation, single-purpose

---

## 一、理解（基于审计报告与注册表数据）

### 1.1 仓库概览
这是一个把 Claude Code 深度整合进 Obsidian 知识库的插件。核心概念是用 Claude 作为知识工程师：自动从网页提取文章（defuddle 技术）、建立 wiki 关联图谱、自动提交到 git、跨会话维护"热点主题"列表。976 颗星反映了知识管理爱好者对 AI 辅助 PKM（个人知识管理）的强烈需求。审计时发现 NL 得分高达 91/100，但安全状态 BLOCKED，原因是 hooks.json 中存在两处 CRITICAL 级提示注入漏洞。

关键事实：
- NL 质量优秀（91/100），绝大多数 skill 满分
- 安全上是教科书级的提示注入反例：Stop 钩子通过 `echo` 向模型注入长指令字符串
- SessionStart 钩子读取 `wiki/hot.md` 并注入到模型上下文——形成跨会话持久污染向量
- 4 个 command 全部缺少 `name` 字段（与 777genius 案例同款问题）

### 1.2 架构剖析
```
claude-obsidian/
├── .claude-plugin/
│   └── plugin.json          # ★ 满分，结构完整
├── CLAUDE.md                # ★ 满分，架构文档清晰
├── commands/
│   ├── autoresearch.md      # 60分（缺name、无数字步骤）
│   ├── canvas.md            # 70分（缺name）
│   ├── save.md              # 60分（缺name、无数字步骤）
│   └── wiki.md              # 70分（缺name）
├── agents/
│   ├── wiki-ingest.md       # ★ 满分
│   └── wiki-lint.md         # 97分（Bash声明但未使用）
├── skills/
│   ├── autoresearch/SKILL.md
│   ├── canvas/SKILL.md      # ★ 满分（但有命令注入漏洞）
│   ├── defuddle/SKILL.md    # ★ 满分（但有路径穿越漏洞）
│   ├── obsidian-bases/SKILL.md
│   ├── obsidian-markdown/SKILL.md
│   ├── save/SKILL.md
│   ├── wiki/SKILL.md
│   ├── wiki-ingest/SKILL.md
│   ├── wiki-lint/SKILL.md
│   └── wiki-query/SKILL.md  # 85分（Write未声明但实际调用）
└── hooks/
    └── hooks.json           # ★ 结构分高但含2个CRITICAL漏洞
```

- **文件类型分布**：4 个 command / 2 个 agent / 9 个 SKILL / 1 个 hooks.json / 1 个 plugin.json / 1 个 CLAUDE.md
- **编排关系**：command 文件作为入口，调用相应 skill 和 agent；wiki-ingest agent 负责批量摄取，wiki-lint agent 负责质量检查；hooks 在 SessionStart 注入上下文，Stop 时写回 hot.md
- **跨件契约**：wiki-query/SKILL.md 实际调用 Write 工具写 wiki 页面，但 `allowed-tools` 未声明 Write——这是典型的工具声明遗漏

### 1.3 设计思路 / 方法论
- **核心设计哲学**："wiki 即真相来源"——所有知识最终沉淀到 Obsidian vault，Claude 是知识工人，不是对话机器人
- **解决什么问题**：跨会话知识积累。用 `wiki/hot.md` 作为"热点记忆"文件，在 SessionStart 时注入，让 Claude 在新会话中"记得"上次的焦点话题
- **Trade-off**：用 hooks 注入上下文比手动 @file 更方便，但把可信边界完全交给 hooks.json 一个文件，一旦该文件被篡改或 hot.md 被污染，整个会话上下文都可以被操控
- **认知模型**：Claude 是一个需要"喂上下文"才能有连续性的无记忆系统，hooks 是上下文的传送带

---

## 二、过去审查发现（2026-04-20 历史快照）

### 2.1 当时质量评分（NLPM）
该仓库 2026-04-20 当时得分 **91/100**，安全状态 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| commands/autoresearch.md | 60 | 缺 `name`（-25）、无allowed-tools（-5）、无编号步骤（-10） |
| commands/save.md | 60 | 缺 `name`（-25）、无allowed-tools（-5）、无编号步骤（-10） |
| commands/canvas.md | 70 | 缺 `name`（-25）、无allowed-tools（-5） |
| commands/wiki.md | 70 | 缺 `name`（-25）、无allowed-tools（-5） |
| skills/wiki-query/SKILL.md | 85 | Write 工具实际使用但未声明（-15） |
| agents/wiki-lint.md | 97 | Bash 声明但代理体中无 bash 命令（-3） |
| skills/autoresearch/SKILL.md | 98 | 模糊量词"major contradictions"（-2） |
| skills/save/SKILL.md | 98 | 模糊量词"most valuable content"（-2） |
| skills/wiki-lint/SKILL.md | 98 | 模糊量词"significant cross-references"（-2） |
| skills/wiki/SKILL.md | 98 | 模糊量词"significant query exchange"（-2） |
| skills/wiki-ingest/SKILL.md | 96 | 模糊量词"significant ideas"、"relevant domain"（-4） |
| hooks/hooks.json | 90 | 自动提交 .raw/；Stop 钩子注入指令 |
| agents/wiki-ingest.md | 100 | — |
| .claude-plugin/plugin.json | 100 | — |
| CLAUDE.md | 100 | — |

**加权平均**：91/100

### 2.2 当时值得借鉴的模式

1. **双层 Agent 分工**（wiki-ingest + wiki-lint）：一个 agent 负责批量写入，另一个负责质量审查，职责单一，互不耦合。wiki-ingest.md 满分，架构设计清晰。  
   → 原文路径：`agents/wiki-ingest.md`、`agents/wiki-lint.md`  
   → 借鉴：对"写入"和"检查"这两类操作，始终考虑分拆成两个独立 agent，而不是让一个 agent 又写又审

2. **CLAUDE.md 满分的架构文档**：CLAUDE.md 达到 100/100，说明该文档完整描述了插件架构、各组件职责、数据流向。这种文档质量是跨件引用可维护的基础。  
   → 原文路径：`CLAUDE.md`  
   → 借鉴：每个插件都应有一个 CLAUDE.md 作为"人类可读的架构图"

3. **plugin.json 满分**：manifest 文件完整注册了所有组件，无遗漏，与 777genius 案例形成鲜明对比。  
   → 借鉴：plugin.json 满分的关键是维护纪律——每次新增文件立即同步 manifest

4. **Skill 分层命名**（wiki / wiki-ingest / wiki-lint / wiki-query）：通过前缀 `wiki-` 组织相关 skill，命名本身就传达了组件间的亲缘关系。  
   → 借鉴：相关 skill 用统一前缀组织，比平铺命名更容易维护和发现

### 2.3 当时的缺陷

1. **Stop 钩子通过 `echo` 注入 LLM 指令**（CRITICAL）  
   → 根本原因：作者把"告诉 Claude 在 Stop 时做什么"的需求用最简单的 shell echo 实现——因为 Claude Code 会把 hook 的 stdout 注入模型上下文，所以这确实"能工作"。但作者没意识到这创造了一个任何能修改 hooks.json 的攻击者都可以利用的持久化提示注入接口  
   → 自查：我的 hooks.json 中有没有通过 `echo` 向 stdout 输出指令字符串的步骤？

2. **SessionStart 读取 hot.md 并注入上下文**（CRITICAL）  
   → 根本原因：hot.md 由 Claude 自身写入并跨会话积累。如果 Claude 曾经处理过恶意网页内容（间接提示注入），那些内容可能已经写入 hot.md，进而在每个新会话开始时被重新注入。这是一个自我强化的攻击链  
   → 自查：我的 SessionStart 钩子是否读取了任何由 Claude 或用户输入生成的文件内容？

3. **wiki-query SKILL.md 调用 Write 但 allowed-tools 未声明**  
   → 根本原因：skill 在开发过程中扩展了功能（从只读到读写），但 frontmatter 没有同步更新。这是典型的"功能漂移但元数据不更新"问题  
   → 自查：我的每个 skill 的 `allowed-tools` 是否与 skill 正文中实际调用的工具严格对应？

### 2.4 当时的优化机会

1. **把 Stop 钩子改为 `prompt` 类型**：使用 Claude Code 原生的 prompt hook（在 hooks UI 中可见），而不是通过 shell echo 注入不可见指令——这在功能上等价，但对用户透明

2. **hot.md 内容校验**：在 SessionStart 读取 hot.md 之前，先做格式检验（如只允许 YAML frontmatter + 无代码块的纯 Markdown），拒绝任何含 code block 或 HTML 的内容注入

3. **命令文件补全 `name` 字段和 `allowed-tools`**：这两个字段是最低成本、最高收益的机械修复

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Stop 钩子 echo 注入指令（CRITICAL） | 读 hooks.json 第 46 行 | **无法验证**（API 限速） | 暂缺 |
| SessionStart cat hot.md 注入（CRITICAL） | 读 hooks.json 第 9 行 | **无法验证** | 暂缺 |
| 4 个 command 缺 `name` 字段 | grep frontmatter | **无法验证** | 暂缺 |
| wiki-query Write 未声明 | 读 SKILL.md allowed-tools | **无法验证** | 暂缺 |

> 注：GitHub API 限速，无认证 token，无法实时读取目标仓库。状态基于 2026-04-20 审计快照。

### 3.2 架构演进

Registry 显示该仓库状态为 `audited`（非 `contributed`）——NLPM 因 CRITICAL 安全问题被阻断，未提交 PR。这意味着从 2026-04-20 至今，外部没有 NLPM 驱动的变更。

### 3.3 新增的可学习模式

暂无（live 检查不可用）。

---

## 四、校准

### 4.1 我已经在做对的

1. **Skill 分层命名**：我已经用前缀组织相关 skill，保持命名空间清晰
2. **allowed-tools 精确匹配**：我已养成在 skill 正文中新增工具调用后立即更新 frontmatter 的习惯
3. **agent 职责单一**：我的 agent 设计已遵循"一 agent 一职责"原则
4. **CLAUDE.md 架构文档**：我的插件已有 CLAUDE.md 描述组件结构
5. **plugin.json 同步纪律**：我已建立新增文件立即更新 manifest 的规范

### 4.2 挑战 / 验证

**这次 audit 挑战了我的一个假设**：我之前认为"hooks.json 中的 echo 输出是无害的调试信息"。这次案例让我意识到，Claude Code 会把 hook stdout 注入模型上下文，所以 echo 的任何内容都是对模型的隐式指令。这彻底改变了我对 hook stdout 的理解。

**验证**：SessionStart + cat 的模式（注入跨会话积累的文件）这个设计本身是有价值的（实现跨会话记忆），但需要严格的内容校验来防止间接注入。这验证了我之前的判断：跨会话记忆的正确实现需要在写入时校验、读取时过滤，而不是直接透传。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查 hooks.json 中是否有通过 echo 向 stdout 输出指令文本的步骤
find . -name "hooks.json" | xargs grep -l '"echo"' 2>/dev/null
# 命中后：审查 echo 输出的内容，如果是 LLM 指令，改用 prompt 类型 hook 或移到 CLAUDE.md

# 检查 SessionStart 钩子是否读取了 Claude 生成的文件内容
find . -name "hooks.json" | xargs python3 -c "
import json,sys
for f in sys.argv[1:]:
    hooks = json.load(open(f))
    for event, rules in hooks.items():
        if 'start' in event.lower() or 'session' in event.lower():
            for rule in (rules if isinstance(rules, list) else [rules]):
                cmd = rule.get('command','')
                if 'cat' in cmd or 'read' in cmd:
                    print(f'{f}:{event}: {cmd}')
" 2>/dev/null
# 命中后：评估被 cat 的文件是否由 Claude 写入（如 hot.md、memory.md），如果是则需要内容校验层
```

```bash
# 检查 allowed-tools 与 skill 正文的工具调用一致性
# 对每个 SKILL.md：提取 allowed-tools 声明 vs 正文中出现的工具调用
for skill in $(find . -name "SKILL.md"); do
  declared=$(grep "allowed-tools:" "$skill" | head -1)
  echo "=== $skill ==="
  echo "  Declared: $declared"
  # 检查正文是否有未声明的工具关键词
  grep -E "^(Write|Read|Bash|Edit|WebSearch|WebFetch|AskUser)" "$skill" | head -3
done
# 命中后：把实际使用的工具补充到 allowed-tools 声明中
```

### 5.2 灵感 → 实施路径

1. **想法**：实现安全的跨会话记忆注入（参考 claude-obsidian 的 hot.md 模式，但加防护）  
   **为何可行**：跨会话记忆是高价值功能，正确实现并不难  
   **第一步**：在写入 hot.md 的 Stop 钩子里加内容过滤——用 `sed` 删除所有 code block（` ```...``` `）再写入；在 SessionStart 读取时用 `grep -v '^\`\`\`'` 过滤掉残余代码块；30 分钟内可完成

2. **想法**：把 hooks 中的 LLM 指令从 echo 迁移到 prompt 类型  
   **为何可行**：Claude Code 原生支持 prompt 类型 hook，可见性更高  
   **第一步**：读 Claude Code hooks 文档，把 hooks.json 中 `"command": "echo '...' "` 形式的条目改写为 `"prompt": "..."` 形式；一个文件，10 分钟改完
