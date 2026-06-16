# htdt/godogen — 学习案例

**仓库**：https://github.com/htdt/godogen
**Stars**：2839 | **来源**：xiaolai upstream | 本地 audit
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-16（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `nl-binary-hybrid`, `model-pinning`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

godogen 是一个为 Godot/Bevy/Babylon.js 等游戏引擎提供 AI 辅助代码生成的 NL 插件。项目定位非常清晰——CLAUDE.md 明确写道「This repository is not a published game repo. It is the source for runtime skills and game-repo templates」。核心思路是：维护一套[发布时渲染](#publish-time-rendering)的模板仓库，通过 `publish.sh` 脚本将引擎（godot/bevy/babylon）与 AI 后端（claude/codex）排列组合，生成多个独立的运行时仓库，每个运行时仓库包含该引擎 + 该 AI 后端对应的 SKILL.md 集合。

这种设计回答了一个常见问题：当你需要同时支持多个游戏引擎和多个 AI 工具时，如何避免手工维护 N×M 个仓库？godogen 的答案是「只维护一份源，发布时派生出 N×M」。

关键事实：
- 2839 stars，NL 评分 91/100，安全评级 CLEAR（5 Medium + 1 Low 发现，均为预期的 AI API 调用）
- 历史审查发现 15 个质量问题，0 个 bug
- 架构在 2026-04-19 到 2026-06-16 之间发生了根本性重构（双树 → 多引擎 + 发布管道）

### 1.2 架构剖析

**当前目录结构（2026-06-16 HEAD）**：
```
/
├── CLAUDE.md                              # 插件定义：「源仓库，非发布仓库」
├── publish.sh                             # 发布脚本：--engine {godot,bevy,babylon} --agent {claude,codex}
├── godot/                                 # Godot 引擎专属资产
│   └── skills/
│       ├── godogen/SKILL.md              # frontmatter: name, display_name, default_prompt
│       ├── godot-api/SKILL.md
│       └── visual-qa/SKILL.md
├── bevy/                                  # Bevy 引擎专属资产
│   └── skills/
│       ├── godogen/SKILL.md
│       └── ...
├── babylon/                               # Babylon.js 引擎专属资产
│   └── skills/...
└── shared/                                # 引擎无关的共享资产
    └── skills/
        └── godogen/
            └── tools/
                └── requirements.txt       # Python 依赖（版本约束不完整）
```

**文件类型分布**：多个引擎目录各有 3 个 SKILL.md，shared/ 含工具脚本，publish.sh 是编排核心，CLAUDE.md 是项目自我描述。

**编排关系**：`publish.sh` 读取 `--engine` 和 `--agent` 参数，将对应引擎目录下的 SKILL.md 模板渲染为目标运行时仓库。SKILL.md 内部使用 `${GODOGEN_SKILL_DIR}` [环境变量](#env-var-interpolation)在运行时解析工具路径。

**跨件契约**：`godot/skills/godogen/SKILL.md` 的 [frontmatter](#frontmatter) 新增了 `name: godogen`、`display_name: Godogen`、`default_prompt` 字段，供 publish.sh 做模板替换。

### 1.3 设计思路 / 方法论

- **核心哲学**：「一次编写，N×M 次发布」——维护引擎和 AI 后端解耦的源树，发布时正交组合，避免代码漂移
- **解决什么问题**：游戏引擎特定的 AI 编程助手既需要深度了解引擎 API（godot-api/SKILL.md）又需要通用的 AI 代码生成技能（godogen/SKILL.md），单一仓库难以同时服务多个引擎生态
- **Trade-off**：源仓库（本仓库）对贡献者有更高的理解门槛（需要理解 publish.sh 的模板逻辑），换来运行时仓库对最终用户的简单性（直接 clone 即用）
- **认知模型**：作者把游戏引擎视为「执行环境维度」，把 AI 后端视为「工具链维度」，两个维度独立可配，体现了正交设计思想

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「发布时渲染的 NL 二进制混合体」**：源仓库存储模板和脚本（NL 规范 + Python 脚本），通过 publish.sh 派生多个纯运行时仓库，每个运行时仓库是单一引擎+AI组合的「二进制发布」。

模式特征清单：
- 特征 1：源仓库 ≠ 用户安装的仓库，存在明确的「发布层」分离
- 特征 2：SKILL.md 模板中使用环境变量 `${GODOGEN_SKILL_DIR}` 做路径占位，publish 时替换为绝对路径
- 特征 3：每个引擎目录有独立的 SKILL.md 集合（godogen + godot-api + visual-qa），引擎专属知识与通用技能共存
- 特征 4：Python 工具脚本（visual-qa、tools/）与 NL 规范（SKILL.md）并存，形成「NL 驱动 + Python 执行」的混合体
- 特征 5：Gemini/xAI/Tripo3D 等多模型 API 通过环境变量注入，支持运行时模型替换（[模型钉选](#model-pinning)在 skill 层实现）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要同时支持多引擎/多后端的工具集 | ✅ 高度适用 | publish 管道解决了 N×M 维护问题 |
| 游戏/3D 场景中需要 AI 视觉 QA | ✅ 适用 | visual-qa/SKILL.md 专门设计用于截图分析 |
| 单一技术栈的轻量插件 | ⚠️ 过度设计 | publish.sh 引入额外复杂度，收益不明显 |
| 需要严格依赖版本控制的生产环境 | ❌ 当前不适用 | requirements.txt 中大多数包未钉版本，生产环境不稳定 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 发布时渲染（本仓库）| godogen | 单一源，多目标发布；引擎/后端正交组合 | 贡献者门槛高，publish 步骤容易漏跑 |
| 多分支变体 | 大多数引擎插件 | 贡献者无需理解模板逻辑 | 分支间补丁漂移，维护成本随分支数线性增长 |
| 单一 SKILL.md（忽略平台差异）| 小型插件 | 最简单 | 引擎 API 差异无法建模，通用性差 |
| IDE-specific 同步脚本（推） | planning-with-files | canonical 清晰，sync 脚本可测试 | 同步脚本 bug 会批量污染所有 IDE 变体 |

### 2.4 改进空间

1. **当前问题**：godogen/SKILL.md（godot 和其他引擎）没有 `## Output` 格式声明。**改进做法**：在每个 godogen/SKILL.md 添加 `## Output` 段，声明「以有序步骤列表输出计划，每步含文件路径 + 操作类型 + 预期结果」。**预期收益**：消除「AI 不知道该以什么格式汇报进展」的歧义，评分从 84/86 升至 90+
2. **当前问题**：`shared/skills/godogen/tools/requirements.txt` 中除 `nvidia-cudnn-cu12==9.*` 外所有包均未钉版本。**改进做法**：运行 `pip freeze` 固定当前已验证的版本，写入 requirements.txt。**预期收益**：消除因上游包更新破坏 visual-qa 工具链的风险（这也是 2026-04-19 审查唯一建议的 PR）
3. **当前问题**：godogen/SKILL.md（godot 版）第 50 行 `"Show user a concise plan summary"` 保留了[模糊量词](#vague-quantifier) `concise`，未被修复。**改进做法**：替换为「以不超过 5 行的列表展示计划摘要，每行格式为 `步骤编号. 操作类型: 文件路径`」。**预期收益**：AI 输出可测试、可预期

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-19 当时整体得分 **91/100**。架构为双树（`claude/` 和 `codex/` 目录），各自包含一套 SKILL.md 集合。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| claude/skills/godogen/SKILL.md | 84/100 | 无 `## Output` 格式声明；`concise`（第 48 行）、`small`（第 101 行）、`major`（第 102 行）三个模糊量词 |
| codex/skills/godogen/SKILL.md | 86/100 | 无 `## Output` 格式声明；`concise`（第 51 行）、`major`（第 102 行）两个模糊量词 |
| claude/skills/godot-api/SKILL.md | 93/100 | 只有隐式输出格式，缺专用 `## Output` 段 |
| codex/skills/godot-api/SKILL.md | 91/100 | 只有隐式输出格式；`relevant`、`targeted` 两个模糊量词 |
| claude/skills/visual-qa/SKILL.md | 93/100 | `context: fork` 但无 `model:` 字段（[frontmatter](#frontmatter) 不完整）|
| codex/skills/visual-qa/SKILL.md | 98/100 | 只有一个模糊量词 `acceptable` |

安全评级：**CLEAR**（5 Medium + 1 Low）

| ID | 级别 | 描述 |
|---|---|---|
| M1–M4 | Medium | Gemini/xAI/Tripo3D API 调用，通过环境变量传入密钥——预期行为，非真实风险 |
| M3 | Medium | subprocess 调用 `nvidia-smi` 查询 GPU 状态——预期行为 |
| M5 | Medium | `git clone` 从 GitHub 拉取代码——预期行为 |
| L1 | Low | `requirements.txt` 缺失（当时），导致依赖版本不受控 |

### 3.2 当时值得借鉴的模式

1. **visual-qa 的多模型能力分层设计**：`visual-qa/SKILL.md` 把 Gemini（图像理解）和 xAI（场景分析）作为可替换的模型后端，通过环境变量切换——这种「功能驱动 + 模型可替换」的设计比直接在代码里硬编码模型名更健壮 → 可借鉴到任何需要多模型能力的 skill
2. **codex/skills/visual-qa/SKILL.md 的 98 分高质量**：接近满分说明作者在视觉 QA 领域的 NL 规范能力很强，只有 `acceptable` 一个模糊量词遗漏 → 这个文件可以作为「接近满分 SKILL.md」的范本参考
3. **godot-api/SKILL.md 的 API 覆盖深度**：93 分的得分背后是对 Godot API 的深度建模，skills 里包含引擎特定的函数签名和用法示例 → 体现了「domain knowledge as NL skill」的做法，比让 AI 自己猜 API 更准确
4. **双树结构的平台隔离**：claude/ 和 codex/ 完全独立，各自可以针对平台特性定制（比如 claude 版可以用 extended thinking，codex 版侧重补全）→ 平台隔离的代价是维护两份，但换来了更精准的平台适配

### 3.3 当时的缺陷

1. **问题**：godogen/SKILL.md（claude 版第 48 行、codex 版第 51 行）出现 `concise` 模糊量词 → **根本原因**：「简洁」是人类认知中的自然表达，但对 AI 来说「简洁」可以是 1 句话也可以是 10 段话，没有测试会暴露这个歧义 → **自查**：我的 SKILL.md 里有多少个 `concise`/`small`/`major` 这类词？
2. **问题**：所有 6 个 SKILL.md 都缺少 `## Output` 格式声明（visual-qa 有隐式格式，其余完全缺失）→ **根本原因**：输出格式容易被作者当成「显而易见」的事而忽略，但 AI 在没有输出规范时会即兴发挥 → **自查**：我的每个 SKILL.md 是否都有显式的 `## Output` 段？
3. **问题**：`requirements.txt` 完全缺失（L1 发现）→ **根本原因**：开发阶段默认用本地环境跑，忘记固化依赖 → 任何从 `pip install package-name` 开始的项目都会遇到这个问题 → **自查**：我的 Python 工具脚本有没有对应的 requirements.txt？
4. **问题**：`claude/skills/visual-qa/SKILL.md` 有 `context: fork` 但无 `model:` 字段 → **根本原因**：[frontmatter](#frontmatter) 字段不完整，`context: fork` 通常意味着需要指定用哪个模型来处理图像 → **自查**：我的 SKILL.md 中凡是有 `context: fork` 的，是否都配了 `model:` 字段？

### 3.4 当时的优化机会

1. 给 `shared/skills/godogen/tools/` 创建 `requirements.txt`，固化所有 Python 依赖版本（这是 2026-04-19 审查唯一判断「值得提 PR」的修复）
2. 在 godogen/SKILL.md（双树均需）添加 `## Output` 格式声明
3. 将第 48/51 行的 `concise` 替换为具体行数限制（如「不超过 5 行」）
4. 给 `claude/skills/visual-qa/SKILL.md` 补充 `model:` 字段（如 `model: claude-opus-4-5`）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| godogen/SKILL.md 缺 `## Output` | `grep -r "## Output" godot/skills/godogen/SKILL.md` | **仍未修复** ❌（grep 返回空）| 架构重构时未补充输出规范，这是结构性遗漏 |
| `concise` 模糊量词 | `grep -n "concise" godot/skills/godogen/SKILL.md` | **仍然存在** ❌（第 50 行命中）| 从 claude/ 第 48 行迁移到新 godot/ 第 50 行，未被修复 |
| requirements.txt 缺失 | `ls shared/skills/godogen/tools/requirements.txt` | **文件已存在** ✅（但内容问题见下）| 维护者响应了最紧迫的安全发现 |
| requirements.txt 内容未钉版本 | `grep "==" shared/skills/godogen/tools/requirements.txt` | **部分修复** ⚠️（只有 `nvidia-cudnn-cu12==9.*` 有版本约束，其余 8 个包仍无版本）| `9.*` 是通配符而非精确版本，其余包完全未约束 |
| visual-qa `context: fork` 缺 `model:` | `grep -A5 "context: fork" godot/skills/visual-qa/SKILL.md` | **架构重构后状态未知**（双树已消失，需重新检查新 visual-qa）| 重构改变了文件位置，原缺陷可能以新形式重现 |

### 4.2 架构演进

从 2026-04-19 到 2026-06-16，godogen 发生了一次根本性架构重构：

**变更前**：`claude/` 和 `codex/` 双树——每个 AI 平台有自己完整的 skills 副本，维护两份几乎相同的内容。

**变更后**：`godot/` + `bevy/` + `babylon/` + `shared/` 按引擎分树——AI 平台的差异由 `publish.sh` 在发布时渲染，不再在源码层保留两份副本。这是从「平台维度分割」到「引擎维度分割」的思路转变。

新增的关键能力：
- `${GODOGEN_SKILL_DIR}` 环境变量让 SKILL.md 在运行时动态解析工具路径，无需在源码中硬编码
- SKILL.md [frontmatter](#frontmatter) 新增 `name`、`display_name`、`default_prompt` 字段，供 publish.sh 做模板替换
- CLAUDE.md 明确声明「这是源仓库」，消除用户直接 clone 此仓库使用的误解

**代价**：部分历史质量问题（`concise` 模糊量词、缺 `## Output`）被带入了新架构，说明重构时没有做 NL 质量检查。

### 4.3 新增的可学习模式

**`${GODOGEN_SKILL_DIR}` 运行时路径解析**：这个模式解决了「SKILL.md 中的工具路径在不同用户机器上不同」的问题。通过在 SKILL.md 中写 `${GODOGEN_SKILL_DIR}/tools/script.py` 而非 `/home/user/.claude/plugins/godogen/tools/script.py`，让路径解析延迟到运行时。这是一个值得推广的模式——任何 SKILL.md 中引用本地文件路径的地方都可以用这个技巧。

**发布时渲染管道（publish.sh）**：用一个 shell 脚本把「引擎维度」×「AI 后端维度」的组合问题转化为「运行 publish.sh 两次」的操作问题。这比维护多个分支或多个仓库都更整洁。

---

## 五、校准

### 5.1 我已经在做对的

1. **agents 有明确的 `tools:` 约束**：echo-sleuth-for-claude 的 5 个 agents 全部声明了 `tools:` 字段，比 godogen（skills 不声明 tools，这是 skill vs agent 的概念差异）更明确地限制了每个组件的权限边界
2. **只有 1 个模糊量词**：echo-sleuth 的 `experience-synthesis/SKILL.md` 第 118 行有 `relevant`，但没有 `concise`、`small`、`major`——这比 godogen 的 godogen/SKILL.md（3 个模糊量词）要好
3. **无跨平台维护负担**：echo-sleuth 只面向 Claude Code 平台，不需要维护多份变体——这是范围控制带来的维护简洁性

### 5.2 挑战 / 验证

这次案例最大的认知更新是**「重构不自动修复 NL 质量问题」**。godogen 做了根本性架构重构，从双树变为多引擎树，这是很大的工程投入，但 `concise` 模糊量词和缺失的 `## Output` 段原封不动地被带入了新架构。这说明：NL 质量检查（`/nlpm:score`）需要明确集成到重构的 checklist 里，而不能指望架构变化自动带来质量改善。

另一个更新：requirements.txt 从无到有是进步，但 `9.*` 通配符和全未钉版本说明「文件存在」≠「依赖已固化」。`pip freeze` 固化和文件存在是两个独立的步骤。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 中有多少个模糊量词
grep -rn "concise\|small\|major\|relevant\|acceptable\|appropriate\|reasonable" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null

# 检查我的每个 SKILL.md 是否有 ## Output 段
for f in $(find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills -name "SKILL.md"); do
  if ! grep -q "## Output" "$f"; then
    echo "缺 ## Output: $f"
  fi
done

# 检查我的 Python 工具脚本目录有没有对应 requirements.txt
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "*.py" -not -name "test_*.py" \
  | xargs -I{} dirname {} | sort -u \
  | while read d; do
      if [ ! -f "$d/requirements.txt" ]; then
        echo "缺 requirements.txt: $d"
      fi
    done

# 检查 requirements.txt 内容（有文件不代表依赖已钉版本）
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "requirements.txt" \
  | xargs grep -L "==" 2>/dev/null \
  | while read f; do echo "requirements.txt 无钉版本: $f"; done

# 检查有没有 context: fork 但缺 model: 字段的 SKILL.md（扫描所有我的仓库）
for f in $(find /tmp/my-repos -name "SKILL.md"); do
  if grep -q "context: fork" "$f" && ! grep -q "^model:" "$f"; then
    echo "context:fork 缺 model: $f"
  fi
done
```

命中后怎么办：
- **模糊量词命中**：用具体约束替换，如 `concise` → 「不超过 5 行」，`relevant` → 「与当前任务直接相关的（即被当前命令引用或修改的）」
- **缺 `## Output` 段**：参考 godogen 的 visual-qa 高分版本（98/100），添加「以 `## Output` 开头的段落，声明返回格式、字段名称和类型」
- **requirements.txt 无钉版本**：在虚拟环境中运行 `pip freeze > requirements.txt` 覆盖现有文件，确保每个包都有精确版本号（非通配符）
- **context:fork 缺 model: 字段**：参考 Claude API 当前模型列表，补充 `model: claude-opus-4-5`（视觉任务建议 Opus）

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 引入 `${ECHO_SLEUTH_SKILL_DIR}` 环境变量模式，替代硬编码路径
   - **为何可行**：echo-sleuth 的 agents 当前通过相对路径或 scripts/ 目录引用工具，在不同安装路径下可能失效
   - **第一步**：在 CLAUDE.md 添加「安装说明」段，声明 `export ECHO_SLEUTH_SKILL_DIR=$(claude plugin path echo-sleuth)`，然后在 skills/SKILL.md 中用 `${ECHO_SLEUTH_SKILL_DIR}/scripts/` 替换硬编码路径；约 15 分钟

2. **想法**：给 echo-sleuth 的每个 SKILL.md 添加显式的 `## Output` 段（对齐 godogen visual-qa 98 分标准）
   - **为何可行**：4 个 skill 目前都没有 `## Output` 段，这是最快提升 NLPM 评分的机械性修复
   - **第一步**：先读 `experience-synthesis/SKILL.md` 确认当前有哪些隐式输出描述，然后将其提炼到 `## Output` 段中，格式为「字段名：类型 — 描述」；约 10 分钟/个 skill

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 htdt/godogen 的核心目的**：为游戏引擎（Godot/Bevy/Babylon.js）提供 AI 辅助代码生成能力，通过发布时渲染管道支持多引擎 × 多 AI 后端的正交组合

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 高 | 都是领域特定插件（godogen=游戏开发，echo-sleuth=会话挖掘）；都有 Python 脚本 + NL skills 的混合架构；都单平台定向（Claude Code）| godogen 有多引擎发布管道，echo-sleuth 无；echo-sleuth 有 agents，godogen 只有 skills | **高** |
| MarkQWu/claude-for-legal | 中 | 都是多域聚合插件（godogen=多引擎，legal=多法律子域）| claude-for-legal 无 Python 工具层；godogen 工具层更技术化 | **低** |
| MarkQWu/drama-workshop-skills | 低 | 都是领域技能包 | 完全不同领域，drama 无 Python 脚本 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 `## Output` 段 | `grep -rL "## Output" skills/*/SKILL.md` | echo-sleuth：4/4 skills 命中（jsonl-core、git-mining、experience-synthesis、memory-management 均无 `## Output`）| **高** |
| `relevant` 等模糊量词 | `grep -rn "relevant\|concise\|appropriate" skills/*/SKILL.md` | echo-sleuth：experience-synthesis/SKILL.md 第 118 行 1 处 `relevant` 命中 | **中** |
| requirements.txt 无钉版本 | `grep -L "==" */requirements.txt` | echo-sleuth：scripts/ 目录无 requirements.txt（Python 脚本使用了哪些第三方包未记录）| **高** |
| frontmatter 字段不完整 | 人工检查每个 SKILL.md frontmatter | echo-sleuth：4 个 skills 均无 `context:` 字段（可能是合理的默认值，但值得确认）| **低** |

**命中后的具体行动建议**：
- echo-sleuth 4 个 skills → 各添加 1 个 `## Output` 段，参考 godogen codex/visual-qa 98 分模板 → 预计每个 10 分钟
- echo-sleuth scripts/ → 梳理脚本 import 语句，生成 requirements.txt 并用 `pip freeze` 固化版本 → 约 20 分钟

### 8.3 别人的更优方案

1. **领域**：`${GODOGEN_SKILL_DIR}` 运行时路径解析
   - **本案例做法**：SKILL.md 内用环境变量 `${GODOGEN_SKILL_DIR}` 引用工具路径，publish.sh 在发布时注入正确的绝对路径
   - **我的项目现状**：echo-sleuth 的 agents 若需要引用 scripts/ 中的工具，目前依赖相对路径或 Claude Code 的工作目录假设
   - **如何借鉴**：在 CLAUDE.md 里声明 `ECHO_SLEUTH_SKILL_DIR` 的设置方法，在 agents/AGENT.md 和 skills/SKILL.md 中统一改用 `${ECHO_SLEUTH_SKILL_DIR}/scripts/`

2. **领域**：SKILL.md 中以流水线阶段表格替代散文描述
   - **本案例做法**：godogen 的 SKILL.md 在流程说明部分使用 Markdown 表格，每行是一个阶段，列为「阶段名/输入/输出/注意事项」
   - **我的项目现状**：echo-sleuth 的 agents 和 skills 用散文段落描述工作流程，层次不清晰
   - **如何借鉴**：把 echo-sleuth experience-synthesis/SKILL.md 的「工作流程」段改为表格形式

3. **领域**：在 CLAUDE.md 明确区分「源仓库 vs 发布仓库」
   - **本案例做法**：CLAUDE.md 第一段直接声明「This repository is not a published game repo. It is the source for runtime skills and game-repo templates」，避免用户误用
   - **我的项目现状**：echo-sleuth 的 CLAUDE.md 没有明确说明仓库的安装方式和发布关系

### 8.4 反向：我的项目做得比他们好的地方

1. **领域**：agents 有明确 `tools:` 声明
   - **我的做法**：echo-sleuth 的 5 个 agents 全部有 `tools:` 字段，明确限制每个 agent 可以调用的工具，权限边界清晰
   - **本案例现状**：godogen 只有 skills（不是 agents），skills 不声明 tools——这是 skill vs agent 的概念差异，但从权限最小化角度，agent-level 的 tools 约束比没有约束更安全
   - **意义**：在 NLPM scoring 体系里，agents 有 `tools:` 约束是加分项；如果 godogen 将来引入 agents，这是一个已知的差距

2. **领域**：模糊量词密度更低
   - **我的做法**：echo-sleuth 全库只有 1 处模糊量词（experience-synthesis 的 `relevant`），无 `concise`/`small`/`major`
   - **本案例现状**：godogen（迁移后的 godot/ 版）仍有 `concise`（第 50 行），历史 claude/ 版有 3 处
   - **意义**：echo-sleuth 的 NL 规范写作在量词精确度上已经优于一个 2839 stars 的高质量项目

---

## 八、术语表

### <a name="publish-time-rendering"></a>发布时渲染（publish-time rendering）
> 源仓库中的文件（SKILL.md 模板、脚本）不直接供用户使用，而是通过 `publish.sh` 脚本经过参数替换后，生成最终的运行时仓库。类似于前端的「构建步骤」——源码是模板，产物是用户安装的内容。godogen 用这个模式支持多引擎 × 多 AI 后端的正交组合，而不需要手工维护 N×M 个仓库。

### <a name="env-var-interpolation"></a>环境变量插值（env var interpolation）
> 在 SKILL.md 文本中写 `${VARIABLE_NAME}` 占位符，由运行时环境（shell 或 Claude Code 的 hook 机制）替换为实际值。godogen 用 `${GODOGEN_SKILL_DIR}` 让 SKILL.md 在任意安装路径下都能正确引用工具脚本，而不依赖硬编码的绝对路径。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> SKILL.md 中描述数量、程度或质量时使用的模糊词语，如 `concise`（简洁）、`small`（小）、`major`（重大）、`relevant`（相关）。这类词对人类读者直观，但对 AI 没有可执行的边界：`concise` 可以是 1 句话也可以是 1 段话，导致 AI 输出不可预测。NLPM 评分会对每个模糊量词扣分，修复方法是用具体的约束替换（如「不超过 5 行」）。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件开头由 `---` 包围的 YAML 元数据块。在 Claude Code 的 SKILL.md 规范中，frontmatter 声明 `name`、`display_name`、`model`、`context`、`tools`、`allowed-tools` 等字段，供 Claude Code 和发布脚本读取。frontmatter 字段不完整（如有 `context: fork` 但无 `model:`）是一类常见 NL 质量问题。

### <a name="model-pinning"></a>模型钉选（model pinning）
> 在 SKILL.md 或 agent 定义的 frontmatter 中指定 `model: <model-id>`，锁定该 skill/agent 使用的 AI 模型版本，而不是使用平台默认模型。godogen 在 visual-qa 等需要多模态能力的 skill 中使用模型钉选，确保即使平台默认模型切换，视觉 QA 能力也不受影响。与未钉选相比，模型钉选增加了维护成本（需要手动更新 ID），但换来了行为可预测性。

### <a name="nl-binary-hybrid"></a>NL 二进制混合体（NL-binary hybrid）
> 同时包含自然语言规范（SKILL.md / AGENT.md）和可执行代码（Python 脚本、shell 脚本）的插件架构。NL 层描述「做什么」，binary 层执行「怎么做」的具体步骤。godogen 是典型的混合体：SKILL.md 描述代码生成流程，`tools/` 下的 Python 脚本执行 GPU 检测、图像处理等无法用纯语言描述的操作。
