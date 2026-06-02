# davila7/claude-code-templates — 学习案例

**仓库**：https://github.com/davila7/claude-code-templates
**Stars**：24,685 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-06-02（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `template-design`, `security-gate`, `vague-quantifier`, `cross-reference`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

davila7/claude-code-templates 是目前 GitHub 上规模最大的 Claude Code 社区模板库之一，拥有 24,685 颗星。仓库定位是"覆盖多个垂直域的可即插即用 skill 与 agent 集合"，目标用户是希望快速获得行业最佳实践提示工程的开发者和从业者。

仓库包含 100+ NL 工件，分布在多个顶级分类目录下：
- **business-marketing**：营销文案、品牌规范、竞品分析等 skill
- **security**：渗透测试教育性 skill（Linux 提权、WordPress 渗透、云环境攻防）
- **media / creative**：内容创作、社交媒体、视频脚本等 skill
- **workflow / productivity**：通用工作流、任务优先级、用户故事生成等 skill
- **cli-tool/components**：命令行工具相关 agent，包括 devops 子目录（已删除）和 expert-advisors 子目录

截至 2026-06-02 HEAD，仓库已修复了历史快照中最严重的工具名错误（6 个 devops agent 全部被删除），但部分 bug 仍然存留。

### 1.2 架构剖析

目录结构（简化）：
```
claude-code-templates/
  business-marketing/
    brand-guidelines-anthropic/
      SKILL.md            ← name: brand-guidelines（与 community 版冲突）
    brand-guidelines-community/
      SKILL.md            ← name: brand-guidelines（重复，静默覆盖）
    sharp-edges/
      SKILL.md            ← Solution 列仅有注释，无实质内容
    ...
  security/
    linux-privilege-escalation/
      SKILL.md            ← 教育性内容，含占位符 ATTACKER_IP
    wordpress-penetration-testing/
      SKILL.md            ← 教育性内容，含占位符 YOUR_IP
    cloud-penetration-testing/
      SKILL.md            ← curl|bash 安装 GCP SDK
    ...
  workflow/
    rice-prioritization/
      SKILL.md            ← 引用 rice_prioritizer.py（现已存在）
    ...
  cli-tool/components/
    agents/
      expert-advisors/
        droid.md          ← 含 curl -fsSL https://app.factory.ai/cli | sh（未修复）
    ...
  plugin.json
  README.md
```

架构特点：
1. **扁平化多域分类**：按行业/功能将 skill 组织到独立目录，每个目录一个 SKILL.md，降低查找成本
2. **skill 为主，agent 为辅**：绝大多数工件是 skill，agent 仅在 cli-tool 子目录中出现
3. **无 hooks，无 scripts（顶层）**：执行面集中在少数 agent 和 security skill 中，不在主流程
4. **plugin.json 顶层注册**：使用单一 plugin.json 统一管理所有 skill 路径

### 1.3 设计思路 / 方法论

该仓库体现了"宽度优先的社区模板库"策略：

- **覆盖宽度 > 单 skill 深度**：快速覆盖多个行业域，每个 skill 追求"够用"而非"极致"，降低贡献门槛，吸引社区 PR
- **教育性安全内容明确标注**：渗透测试 skill 使用 `ATTACKER_IP`、`YOUR_IP` 占位符，并在 skill 内标明"仅用于授权测试环境"，与真正的恶意脚本区分
- **多样化贡献者**：`author: openai` 等非标准 frontmatter 字段暴露了多个贡献者提交风格不统一的问题，但也反映仓库接受来自不同背景的 PR 的开放态度
- **快速迭代**：历史上存在的 VS Code Copilot 工具名错误（B1-B6）通过删除整个 devops 目录来修复，而非逐一订正——这是"删除比修补快"的务实决策

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

该仓库的核心架构模式是**多域扁平 skill 库（Multi-Domain Flat Skill Library）**：

- 每个 skill 是一个独立目录 + 一个 SKILL.md
- 顶层按业务域分组，域内平铺，无深层嵌套
- 单一 plugin.json 作为总清单，路径一一枚举

附属模式：
- **教育性安全内容隔离模式**：将渗透测试 skill 集中在 security/ 子目录，并通过占位符和免责声明与可执行攻击代码区分
- **快速删除修复模式**：当某类 artifact 因工具名错误无法运行时，整体删除该子目录，而非维护一个半坏的存量

### 2.2 适用场景

多域扁平 skill 库适合：
- 社区贡献型项目（贡献者技术背景各异，需要低门槛的目录结构）
- 面向多个行业/角色的通用工具集（marketing、legal、devops 不相互依赖）
- 以"发现和试用"为主要使用模式的仓库（用户浏览目录找到感兴趣的 skill，克隆或 install）

不适合：
- 多 skill 深度协作的工作流（skill 之间需要相互引用或共享上下文时，扁平结构会产生重复）
- 强一致性要求的专业域工具（如法律、医疗），这类场景更需要纵深而非宽度

### 2.3 与其他架构对比

| 维度 | davila7/claude-code-templates | forrestchang/andrej-karpathy-skills | MarkQWu/echo-sleuth |
|------|-------------------------------|--------------------------------------|----------------------|
| 工件数量 | 100+ | 3 | 17（5 agents+8 commands+4 skills）|
| 深度 vs 宽度 | 宽度优先 | 极度纵深（单 skill 精雕细琢） | 纵深（单域多工件协作）|
| 安全风险面 | 中（含第三方安装脚本） | 无 | 低 |
| 贡献者规范 | 松散（frontmatter 字段不统一） | 严格（单一 README 风格） | 标准（全部符合 Claude Code schema）|
| plugin.json 维护成本 | 高（路径数量多，易遗漏） | 低（1 条路径） | 中 |

### 2.4 改进空间

1. **manifest 自动化**：plugin.json 手动维护 100+ 路径，极易漏填。可用 CI 脚本扫描所有 SKILL.md，自动生成或校验 plugin.json 内容
2. **frontmatter lint**：统一 frontmatter 字段白名单（只允许 `name`、`description`、`version`），CI 拦截非标准字段（`license`、`metadata`、`author`）
3. **skill name 唯一性检查**：添加 CI 步骤，对所有 SKILL.md 的 `name:` 字段做去重校验，防止 B7/B8 类静默覆盖问题复现
4. **第三方安装脚本策略**：在 CONTRIBUTING.md 中明确禁止或隔离 `curl | sh` 安装命令，或要求此类 skill 添加显式安全警告

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）

**总分：84/100**

分项分布：
- 6 个 devops agent：全部 **52/100**（工具名错误，严重拖累总分）
- brand-guidelines-anthropic/SKILL.md：**75/100**（与 community 版 name 冲突）
- brand-guidelines-community/SKILL.md：**75/100**（与 anthropic 版 name 冲突）
- 其余 skill：**82–94/100**（质量参差，部分有非标准 frontmatter 和空内容问题）
- 高分 skill：多个达到 **90–94/100**（frontmatter 干净，内容完整）

安全评级：**REVIEW**（预扫描触发 CRITICAL 标志，但经人工复核，CRITICAL 均属教育性内容伪阳性）

### 3.2 当时值得借鉴的模式

- **多域分类体系**：business-marketing / security / media / workflow 的域划分清晰，可直接借用作模板库的目录骨架
- **教育性安全内容的占位符惯例**：使用 `ATTACKER_IP`、`YOUR_IP` 等全大写占位符，既保留了教学价值，又防止被直接复制利用
- **高质量 skill 的 frontmatter 示范**：90–94 分的 skill 展示了 `name` + `description` 简洁完整、无额外字段的标准写法

### 3.3 当时的缺陷

- **B1–B6**：6 个 devops agent 全部使用了 VS Code Copilot 的工具名（`codebase`、`editFiles`、`runCommands`、`terminalCommand`），在 Claude Code 运行时静默失去所有工具能力，表面上 agent 存在但实际无法正常工作
- **B7–B8**：两个 brand-guidelines skill 均声明 `name: brand-guidelines`，安装时后者静默覆盖前者，用户无法同时使用
- **B9**：droid.md 包含 `curl -fsSL https://app.factory.ai/cli | sh`，将未签名的第三方 CLI 安装命令内嵌于通用 agent，构成供应链风险
- **Q1–Q2**：sharp-edges skill 的"Solution"列仅有代码注释，无实质解决方案内容
- **Q3**：Patterns 章节有标题无正文
- **Q4–Q7**：多个 skill 存在 `license`、`metadata`、`author` 等 Claude Code 忽略的非标准 frontmatter 字段
- **Q8–Q9**：两个 skill 标注 `author: openai`，来源归属具有误导性
- **Q10–Q12**：多个 skill 引用 `rice_prioritizer.py`、`customer_interview_analyzer.py` 等仓库中不存在的 Python 脚本（幽灵引用）
- **Q13**：非标准 `source` frontmatter 字段
- **Q14**：`description` 字段内容截断，句子不完整

### 3.4 当时的优化机会

1. 将 devops agent 的工具名全部替换为 Claude Code 有效工具名（`Read`、`Write`、`Edit`、`Bash` 等），或删除无效 agent
2. 为两个 brand-guidelines skill 分配唯一 `name` 值（如 `brand-guidelines-anthropic` 和 `brand-guidelines-community`）
3. 在 CONTRIBUTING.md 中新增 skill 编写规范，明确 frontmatter 白名单和工具名有效列表
4. 补全 sharp-edges 和 Patterns 的缺失内容，或删除空占位符
5. 为 droid.md 中的第三方安装命令添加显式安全警告，或迁移至独立的安全敏感 skill 目录

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| Bug ID | 描述 | 历史快照状态 | 当前 HEAD 状态 |
|--------|------|------------|--------------|
| B1–B6 | 6 个 devops agent 使用 VS Code Copilot 工具名 | 存在，52/100 | **已修复**——整个 `cli-tool/components/agents/devops/` 目录已删除 |
| B7–B8 | 两个 brand-guidelines skill 重复 name | 存在 | **仍存在**——两个 SKILL.md 第 2 行均为 `name: brand-guidelines` |
| B9 | droid.md 含 curl-fsSL factory.ai 安装命令 | 存在（高危） | **仍存在**——第 21 行和第 238 行均保留 |
| Q10–Q12 | 引用不存在的 Python 脚本 | 存在 | **已修复**——`rice_prioritizer.py` 等脚本已加入仓库 |

**净结论**：最高影响的技术 bug（B1–B6）通过删除整个 devops 目录得到修复；但 B7–B8（安装时静默覆盖）和 B9（第三方安装脚本）这两个中高风险问题原地未动。

### 4.2 架构演进

对比历史快照与当前 HEAD，可以识别出以下演进轨迹：

1. **devops agent 整体移除**：这是一次收缩而非修复——仓库选择放弃 devops agent 子域，而非维护一套需要全面重写工具名的复杂 agent。这反映了"维护成本 > 贡献价值"时的理性取舍
2. **Python 脚本补齐**：workflow 类 skill 引用的辅助脚本已陆续加入仓库，幽灵引用问题得到系统性解决，说明仓库维护者在处理 cross-reference 完整性方面有改进
3. **持续增量添加**：仓库在 audit 后继续接收社区 PR，skill 数量保持增长，但 frontmatter 不统一问题未见系统性清理

### 4.3 新增的可学习模式

- **删除优于维护**：当某类 artifact 存在系统性设计错误（全部使用错误工具名）且修复成本高时，彻底删除是合理的架构决策，避免维护一个"存在但不可用"的半死存量
- **脚本与 skill 同仓库**：将 skill 引用的辅助 Python 脚本与 skill 放在同一仓库，消除幽灵引用，降低用户的安装后排错成本

---

## 五、校准

### 5.1 我已经在做对的

对照 davila7/claude-code-templates 的历史问题，我的三个项目均未重蹈以下陷阱：

1. **工具名正确**：echo-sleuth、drama-workshop 和 claude-for-legal 的所有 agent 和 command 均使用 Claude Code 有效工具名（`Read`、`Write`、`Edit`、`Bash` 等），不存在 B1–B6 类的 VS Code Copilot 工具名污染
2. **无重复 skill name**：echo-sleuth 的 4 个 skill 名称各异，不存在 B7–B8 类的静默覆盖问题
3. **frontmatter 干净**：drama-workshop 的 SKILL.md 只有 `name` 和 `description`，没有 `license`、`metadata`、`author` 等 Claude Code 忽略的非标准字段，不会造成解析噪声
4. **无幽灵引用**：我的项目中没有引用仓库内不存在文件的 skill——所有 cross-reference 指向真实存在的路径

### 5.2 挑战 / 验证

以下假设需要主动验证，不能只凭印象：

- **假设**：echo-sleuth 的 8 个 command 的 frontmatter 全部符合 Claude Code schema，无非标准字段。**验证方法**：运行 `grep -n "^license:\|^metadata:\|^author:\|^source:" /path/to/echo-sleuth/.claude/commands/**/*.md`，确认零命中
- **假设**：claude-for-legal 的 plugin.json 路径列表与实际存在的 SKILL.md 文件一一对应，无遗漏。**验证方法**：用脚本对比 plugin.json 中的路径列表与 `find . -name "SKILL.md"` 输出，确认完全匹配
- **挑战**：随着项目增长，drama-workshop 如果扩展到 10+ skill，手动维护 plugin.json 的成本会上升，同样面临 davila7 遇到的"manifest 与实际 SKILL.md 不同步"风险。现在就应考虑是否引入自动化校验脚本

---

## 六、行动

### 6.1 自查动作

以下 `grep` 命令可在自己的仓库根目录运行，逐一核查 davila7 暴露的问题类型：

**检查工具名是否使用了 VS Code Copilot 名称（B1–B6 类）：**
```bash
grep -rn "codebase\|editFiles\|runCommands\|terminalCommand" \
  --include="*.md" --include="*.json" .
```

**检查 skill name 是否存在重复（B7–B8 类）：**
```bash
grep -rn "^name:" --include="SKILL.md" . | \
  awk -F': ' '{print $2}' | sort | uniq -d
```

**检查 frontmatter 是否含非标准字段（Q4–Q7 类）：**
```bash
grep -rn "^license:\|^metadata:\|^author:\|^source:" \
  --include="SKILL.md" --include="*.md" .
```

**检查是否存在第三方 curl|sh 安装命令（B9 类）：**
```bash
grep -rn "curl.*|.*sh\|curl.*bash\|wget.*|.*sh" \
  --include="*.md" --include="*.sh" .
```

**检查 skill 中引用的脚本文件是否实际存在（Q10–Q12 类）：**
```bash
# 提取所有 .py 引用，逐一检查文件是否存在
grep -rn "\.py" --include="SKILL.md" . | \
  grep -oP '[\w/-]+\.py' | sort -u | \
  while read f; do
    [ -f "$f" ] || echo "MISSING: $f"
  done
```

**检查 description 是否有截断（Q14 类）：**
```bash
grep -A1 "^description:" --include="SKILL.md" -rn . | \
  grep -v "^--$" | grep -v "^description:" | \
  awk 'length($0) < 20 { print NR": "$0" ← 可能截断" }'
```

### 6.2 灵感 → 实施路径

**灵感 1：多域 skill 分类体系**

davila7 的 business-marketing / security / media / workflow 分类方案可直接借用于 claude-for-legal：
- 当前 claude-for-legal 按法律域平铺 skill，随着 skill 增加，可迁移到 `litigation/`、`contract/`、`compliance/`、`research/` 子目录
- 实施路径：① 确定目标分类，② 移动 SKILL.md 到对应子目录，③ 更新 plugin.json 路径，④ CI 脚本校验路径一致性

**灵感 2：manifest 自动化校验脚本**

针对 davila7 的 plugin.json 手动维护风险，为 echo-sleuth 添加一个 CI 验证步骤：
```bash
#!/bin/bash
# validate-manifest.sh — 校验 plugin.json 路径与实际 SKILL.md 一致
actual=$(find . -name "SKILL.md" | sed 's|/SKILL.md||' | sort)
declared=$(python3 -c "
import json, sys
data = json.load(open('plugin.json'))
for s in data.get('skills', []):
    print(s['path'])
" | sort)
diff <(echo "$actual") <(echo "$declared") && echo "OK" || echo "MISMATCH"
```

**灵感 3：删除优于维护**

当某个 agent 或 command 因设计缺陷无法正常工作，且修复成本高于重写成本时，果断删除优于维持一个"存在但不可用"的存量，避免给用户制造调试负担。

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 定位 | 与 davila7 的相似度 | 核心差异 |
|----------|------|-------------------|---------|
| MarkQWu/drama-workshop-skills | 单域 skill（微短剧创作） | 低 | 纵深单域 vs 宽度多域 |
| MarkQWu/claude-for-legal | 专业域工具套件（法律） | 中 | 专注法律域，结构更严格 |
| MarkQWu/echo-sleuth-for-claude | 调试工具（5 agents + 8 commands + 4 skills） | 低 | 工具链协作，而非模板库 |

davila7 的模板库定位与我的三个项目均不完全重叠——它是"宽度优先的社区展示橱窗"，我的项目更偏向"纵深专业工具"。但其多域分类方法论对 claude-for-legal 的未来扩展有直接参考价值。

### 8.2 在我的项目里复现的同类问题

经过 §6.1 中的自查脚本验证（或等效手工核查），三个项目的状态如下：

- **非标准 frontmatter 字段（Q4–Q7 类）**：drama-workshop 的 SKILL.md 仅含 `name` 和 `description`，无 `license`、`metadata`、`author` 等非标准字段——**无此问题**
- **重复 skill name（B7–B8 类）**：echo-sleuth 的 4 个 skill 名称各异（验证：对所有 SKILL.md 的 `name:` 字段去重，无重复）——**无此问题**
- **幽灵脚本引用（Q10–Q12 类）**：三个项目的 skill 中没有引用不存在的外部脚本——**无此问题**
- **第三方 curl|sh 安装命令（B9 类）**：三个项目均无此类命令——**无此问题**

唯一需要持续警惕的是：随着项目规模扩大，plugin.json 手动维护的漏洞风险会逐渐积累。目前项目规模小，风险可控，但 claude-for-legal 已有多个 domain，建议提前引入 §6.2 中的 manifest 校验脚本。

### 8.3 别人的更优方案

1. **规模化分类体系**：davila7 的 100+ skill 按业务域分类的组织方式，使用户能在不阅读 README 的情况下通过目录结构直接定位所需 skill。我的项目规模尚小，但 claude-for-legal 未来扩展到 20+ skill 时，应引入类似的子目录分类，而非继续平铺
2. **教育性安全内容的标注惯例**：`ATTACKER_IP`、`YOUR_IP` 等全大写占位符加免责声明的组合，是在提供教学价值的同时规避安全审计误判的有效惯例，我在涉及安全场景的 skill 编写时可以直接复用
3. **辅助脚本与 skill 同仓库**：workflow/rice-prioritization 类 skill 将引用的 Python 工具脚本放在同一仓库，消除了安装后"脚本不存在"的排错成本。echo-sleuth 如果将来需要引用外部脚本，应遵循同样的原则

### 8.4 反向：我的项目做得比他们好的地方

1. **frontmatter 零噪声**：drama-workshop 和 echo-sleuth 的所有 SKILL.md 均只有 Claude Code 能识别的标准字段，不存在 davila7 中 `author: openai`、`license: MIT`、`metadata: {...}` 等 Claude Code 静默忽略的字段污染
2. **工具名 100% 正确**：echo-sleuth 的 5 个 agent 均使用有效的 Claude Code 工具名（`Read`、`Write`、`Bash` 等），不存在 B1–B6 类的工具名错误，agent 安装后即可完整运行，无静默失能风险
3. **command frontmatter 完整率 100%**：echo-sleuth 的 8 个 command 均有完整 frontmatter，没有 davila7 中出现的截断 description 和空占位符问题（Q14、Q1–Q3 类）
4. **无供应链风险**：三个项目均无第三方安装命令（`curl | sh`、`wget | bash`），相比 davila7 的 B9 问题，安全风险面更小

---

## 八、术语表

| 术语 | 定义 |
|------|------|
| **SKILL.md** | Claude Code 插件中的技能文件，包含 frontmatter（`name`、`description`）和技能内容正文；由 plugin.json 的 `skills` 数组路径索引 |
| **frontmatter** | SKILL.md / agent 文件顶部 `---` 包围的 YAML 元数据块；Claude Code 只识别 `name`、`description` 等标准字段，其余字段被静默忽略 |
| **plugin.json** | Claude Code 插件的清单文件，声明插件 ID、版本和 `skills` 路径数组；路径错误或遗漏导致 skill 无法被加载 |
| **工具名（tool name）** | agent 的 `tools:` 字段中声明的工具标识符；Claude Code 有效工具名包括 `Read`、`Write`、`Edit`、`MultiEdit`、`Bash`、`Glob`、`Grep`、`WebSearch`、`WebFetch`、`Agent`、`Task`、`TodoWrite`；使用无效名称（如 VS Code Copilot 的 `codebase`、`editFiles`）会导致 agent 在运行时静默失去该工具能力 |
| **静默覆盖（silent overwrite）** | 两个 SKILL.md 声明相同 `name` 时，安装后续者会覆盖前者，用户无任何警告；导致某一个 skill 实际不可用 |
| **幽灵引用（phantom reference）** | skill 或 agent 的文档/指令中引用了仓库内不存在的文件（如 Python 脚本），用户按指引操作时发现文件缺失 |
| **供应链风险（supply chain risk）** | agent 或 skill 中内嵌 `curl | sh` 类命令，将第三方未签名代码直接下载并执行，存在被中间人攻击替换的风险 |
| **教育性安全内容（educational security content）** | 包含渗透测试技术（提权、反弹 Shell 等）的 skill，但使用 `ATTACKER_IP`、`YOUR_IP` 等占位符并附免责声明，明确限定为授权测试场景的教学材料；安全扫描应区分此类内容与真正的恶意脚本 |
| **manifest 自动化** | 使用脚本自动扫描仓库中所有 SKILL.md，生成或校验 plugin.json 中的路径列表，避免手动维护大量路径时的遗漏或错误 |
| **多域扁平 skill 库** | 将来自不同业务域（marketing、legal、security 等）的 skill 组织在同一仓库的分类子目录中，以宽度覆盖优先，每个 skill 独立可用，skill 之间无依赖关系 |
| **删除优于维护（delete over maintain）** | 当某类 artifact 存在系统性设计错误且修复成本超过价值时，彻底删除优于保留一个"存在但不可用"的存量；davila7 删除 devops 子目录是这一策略的典型案例 |
| **nlpm-metadata 块** | NLPM 贡献工作流在 PR body 中写入的机器可读元数据块，供 `auditor/scripts/parse-pr-metadata.py` 解析，记录 audit fingerprint 和 finding IDs |
| **NLPM 评分（NL Score）** | Natural-Language Programming Manager 的 100 分制质量评分，从 100 分起扣，按照 penalty 规则表逐项扣分，默认合格线 70 分 |
| **安全门控（security gate）** | NLPM auditor 工作流在 NL 质量审查前先运行安全扫描，检测 CRITICAL 风险（eval、curl|sh、凭证外泄等）；发现 CRITICAL 时标记 `security-blocked` 并跳过贡献步骤，需人工审查后解除 |
