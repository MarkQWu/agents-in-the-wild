# wuji-labs/nopua — 学习案例

**仓库**：https://github.com/wuji-labs/nopua
**Stars**：1229 | **来源**：upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-05（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `single-purpose`, `template-design`, `monorepo-vs-split`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
wuji-labs/nopua（Not Prohibited — Universal Agency）是一个把「道德经」哲学引入 NL 编程的 skill 插件：不用威胁恐吓 AI（PUA 式 prompt），而用信任和内在动机引导 AI 工作。1229 颗星，在「反 PUA 提示词」话题上具有独特定位。

关键事实：
- 核心理念：「以道法自然驾驭 AI，而非以恐惧和威胁」——stop→observe→shift→act→reflect（停·观·转·行·悟）五步法
- 多平台策略：除 Claude Code skill 外，还提供 Codex（`codex/nopua/SKILL.md`）和 Kiro（`kiro/skills/nopua/SKILL.md`）版本
- 现状：audit 发现的 3 个 [manifest](#manifest) 缺陷在 2.5 个月内**全部修复**，是本批案例中修复率最高的仓库
- 核心 skill 质量极高：精确的行为标准、具体的动作表、无模糊量词——95 分以上

### 1.2 架构剖析
```
nopua/
├── .claude-plugin/
│   └── plugin.json              ← 注册 3 个 skill + 3 个 agent（已修复）
├── skills/
│   ├── nopua/SKILL.md           ← 英文主 skill（language: en）
│   ├── nopua-zh/SKILL.md        ← 中文 skill（language: zh-CN）
│   └── nopua-lite/SKILL.md      ← 精简版
├── agents/
│   ├── nopua-mentor.md          ← 中文 mentor agent
│   ├── nopua-mentor-en.md       ← 英文 mentor agent（已注册）
│   └── nopua-mentor-ja.md       ← 日文 mentor agent（已注册）
├── commands/
│   └── nopua.md / nopua-en.md / nopua-ja.md  ← 手动触发命令
├── codex/nopua/SKILL.md         ← Codex 平台版
├── kiro/skills/nopua/SKILL.md   ← Kiro 平台版
└── SKILL.md                     ← 根目录（zh-CN 内容，正式内容在 skills/nopua-zh/）
```

- **文件类型分布**：5 个 SKILL.md（3 个注册 + 2 个平台特化）+ 3 个 agent + 3 个 command + 1 个 plugin.json
- **编排关系**：command（`/nopua`）→ skill（`nopua-zh/SKILL.md`）→ mentor agent（可选升级）；三层递进，用户可按需选择深度
- **跨件契约**：plugin.json 现在准确声明了所有 3 个 skill 和 3 个 agent；commands 有描述性 frontmatter；平台版本（codex、kiro）与主 skill 内容一致

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「约束通过正向激励而非惩罚」——skill 不写「不要做 X」，而是写「当你注意到 X 倾向时，执行 Y 动作」，把行为塑造从「规则执行」变成「觉察练习」
- **解决什么问题**：AI 在遇到困难任务时会「表演忙碌」（继续执行无效操作）或「恐惧驱动」（写出大量免责声明），nopua 提供「停·观·转·行·悟」框架让 AI 识别并打破这些模式
- **做了什么 trade-off**：哲学深度 vs. 上手速度。`nopua-lite` 是这个 trade-off 的显性体现——精简版给想快速试用的用户，完整版给认可哲学前提的用户
- **反映什么认知模型**：作者把 AI 的「卡住」问题理解为心理状态问题（恐惧、惯性），而非能力问题——这个框架本身就是独特的认知贡献

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「多平台多语言平铺型 Skill（Multi-Platform Flat Layout）」**

一套核心内容，通过多个语言变体 + 多个平台适配版本平铺展开，由单一 plugin.json 统一注册。

模式特征清单：
- 特征 1：核心内容只维护一份（`skills/nopua/SKILL.md`），其他语言是翻译分支
- 特征 2：平台适配版（codex、kiro）单独存放，不混入主 skill 目录
- 特征 3：三层访问深度（command 快捷入口 → skill 完整版 → mentor agent 个性化指导）
- 特征 4：nopua-lite 作为「低门槛入口」，降低首次使用摩擦
- 特征 5：plugin.json 版本号（2.0.2）显示插件已经历过重大版本升级

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要覆盖多语言用户群 | ✅ 适用 | 多语言 skill 平铺策略简洁有效 |
| 跨平台部署（Claude Code + Codex + Kiro） | ✅ 适用 | 平台特化版本独立目录，互不干扰 |
| 核心内容需要频繁更新 | ⚠️ 需注意 | 多语言版本需同步更新，维护成本随语言数增加 |
| 超过 5 个语言变体 | ❌ 不适用 | 平铺策略难以管理，应改用 i18n 机制 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多平台平铺（本仓库） | wuji-labs/nopua | 结构直观，每个平台有独立目录 | 内容更新需同步多个文件 |
| 单 skill 全平台兼容 | 大多数 skill | 维护成本最低 | 平台特化逻辑无法表达 |
| 子模块分仓 | 高 star 框架插件 | 各语言版本独立演进 | 跨仓协作复杂度高 |

### 2.4 改进空间
1. **当前问题**：3 个 mentor agent 均无 example blocks 和 output format 声明（audit 打了 75 分）。**改进做法**：为 `nopua-mentor.md` 添加一个具体示例（用户说「我卡了两小时了」→ mentor 触发「观」模式的具体回应格式）。**预期收益**：agent 分数从 75 提升到 90+，用户明确知道 mentor 会怎么回应。
2. **当前问题**：commands 只有 `description` frontmatter，缺少 `allowed-tools` 声明。**改进做法**：确认 commands 实际使用的工具（可能是 Read），在 frontmatter 加 `allowed-tools: []`（如果纯对话则空列表）。**预期收益**：消除 NLPM 扣分，同时明确权限边界。
3. **当前问题**：多语言内容（nopua-en、nopua-ja）在命令层面有对应文件，但 skill 层面只有 zh 和 en，日文 skill 缺失。**改进做法**：要么补全 `skills/nopua-ja/SKILL.md`，要么从 command 中移除日文引用，保持一致。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **80/100**（13 个文件加权平均）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `commands/nopua*.md` (×3) | 65 | 无 YAML frontmatter（−25），无 `allowed-tools`（−5） |
| `.claude-plugin/plugin.json` | 60 | 声明了 6 个不存在的 skill 路径；nopua skill 语言标签错误 |
| `agents/nopua-mentor*.md` (×3) | 75 | 无 example blocks（−15），无 output format（−10） |
| `skills/nopua*.md` / `SKILL.md` | 90-95 | 总体优秀；轻微冗余问题 |
| `codex/nopua/SKILL.md` | 95 | 平台版，无问题 |

### 3.2 当时值得借鉴的模式

1. **精确行为标准（无模糊量词）** → skills/nopua-zh/SKILL.md 和 skills/nopua/SKILL.md 中几乎无模糊形容词，全是「当 X 出现时，执行 Y，结果为 Z」的格式 → 为什么好：给模型的指令是可执行的，不是感受描述 → 借鉴：审查自己所有 skill 中出现「appropriate」「thorough」等词的地方，替换为具体动作
2. **三层访问深度设计** → command（快速触发）→ skill（完整哲学）→ agent（个性化mentor）→ 为什么好：不同用户有不同需求，不强制深度，降低入门摩擦 → 借鉴：为 gstack 中复杂的 skill（如 `office-hours`）设计一个 lite 版本入口
3. **nopua-lite 作为显性入门版** → 明确提供精简版，而不是让用户自己判断「完整版是否太复杂」→ 借鉴：当一个 skill 全读完需要 5 分钟以上，就应该有一个 30 秒的 lite 版

### 3.3 当时的缺陷

1. **plugin.json 声明了 6 个不存在的 skill 路径**：manifest 说「7 个语言的 skill」，但只有 3 个语言的 skill 文件实际存在（en、zh-CN、lite）。根本原因：先写了「7 languages」的愿景描述，然后 manifest 按愿景写，但实现没跟上。这是「愿景驱动 manifest」的典型陷阱。自查：我的 plugin.json 中是否有声明了但文件不存在的路径？
2. **语言标签错误（zh-CN 标签指向 en 内容）**：`skills/nopua/SKILL.md` 被标记为 `"language": "zh-CN"`，但内容是英文。根本原因：复制粘贴时标签没有更新，或者 zh-CN 内容后来被换成了 en 内容但标签没改。自查：我的 manifest 中所有 language 字段是否准确反映了实际内容语言？
3. **EN/JA mentor agent 未注册**：2 个 agent 文件存在但未出现在 plugin.json 的 agents 数组里，用户无法通过 `/agent` 命令发现它们。根本原因：文件先于注册创建（典型的「先写代码后更新配置」工程流程问题）。自查：我的所有 agent 文件是否都在 manifest 中有对应注册？

### 3.4 当时的优化机会

1. 清理 plugin.json：删除不存在的 6 个语言 skill 路径，或补全对应的 SKILL.md 文件（选择前者更务实）
2. 修正 zh-CN 语言标签：将 `skills/nopua/SKILL.md` 标记为 `"language": "en"`，将 `skills/nopua-zh/SKILL.md` 标记为 `"language": "zh-CN"`
3. 注册 EN/JA mentor agents：在 plugin.json 的 agents 数组加入 nopua-mentor-en 和 nopua-mentor-ja

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 6 个不存在的 skill 路径 | 读 plugin.json skills 数组 | **已修复** — skills 数组仅剩 3 个实际存在的路径（nopua/en、nopua-zh/zh-CN、nopua-lite/en） | 采用「删除幻想条目」而非「补全文件」的策略，务实 |
| zh-CN 语言标签错误 | `grep language .claude-plugin/plugin.json` | **已修复** — nopua 标签现为 `"en"`，nopua-zh 标签为 `"zh-CN"` | 准确反映实际内容语言 |
| EN/JA mentor agent 未注册 | 读 plugin.json agents 数组 | **已修复** — agents 数组现包含全部 3 个 mentor agent | 三个缺陷全部修复 |

### 4.2 架构演进

从 audit 时的「manifest 有 6 个幻想条目」到现在的「manifest 只列实际存在的文件」，说明作者接受了「务实优于愿景」的修复方向——不是去实现 7 语言的大计划，而是让 manifest 如实反映现状。这是一个成熟的工程判断：**承认技术债，缩小 scope 比无限延期更好**。

### 4.3 新增的可学习模式

plugin.json 从「声明愿景」到「声明现实」的转变，本身就是一个值得记录的模式：**manifest 应该是当前状态的镜子，而不是未来规划的文档**。Roadmap 属于 README，不属于 plugin.json。

---

## 五、校准

### 5.1 我已经在做对的

1. **manifest 路径都存在**：gstack 和 bureau 的 plugin.json（如果有的话）中声明的所有路径应该都是真实存在的——这是 nopua 修复后才达到的标准
2. **language 字段准确**：echo-sleuth 的英文 skill 应该都标记为 `"en"`，没有误标问题
3. **所有 agent 均已注册**：echo-sleuth 的 4 个 skill 都有 frontmatter 且（预期）已注册

### 5.2 挑战 / 验证

这次案例**验证**了一个我认为很重要的原则：**「manifest as code」**——plugin.json 必须和磁盘上的文件实时对应，任何「先写声明再实现」的做法都是技术债。nopua 在 2.5 个月内把 3 个 manifest bug 全部修复，速度快，是因为这些 bug 影响实际用户安装体验（6 个 skill 无法加载），压力直接。**有用户压力的 bug 修复最快——这也说明功能可见性是驱动质量的最强杠杆。**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 plugin.json 声明的路径是否都真实存在
python3 - << 'EOF'
import json, os, glob
for plugin_json in glob.glob('/tmp/my-repos/MarkQWu-*/*/plugin.json') + glob.glob('/tmp/my-repos/MarkQWu-*/.claude-plugin/plugin.json'):
    repo_root = os.path.dirname(os.path.dirname(plugin_json))
    with open(plugin_json) as f:
        data = json.load(f)
    for skill in data.get('skills', []):
        path = os.path.join(repo_root, skill.get('path', ''))
        if not os.path.exists(path):
            print(f"MISSING: {plugin_json} → {skill.get('path')}")
EOF
# 命中后怎么办：要么补全缺失文件，要么从 plugin.json 删除幻想条目

# 检查 manifest 中的 language 字段是否准确
grep -n '"language"' /tmp/my-repos/MarkQWu-*/.claude-plugin/plugin.json 2>/dev/null
# 命中后怎么办：对每条 language 声明，打开对应 SKILL.md 确认内容语言一致
```

### 6.2 灵感 → 实施路径

1. **想法**：为 gstack 最复杂的 skill（`office-hours`，12 处模糊量词）设计 lite 版本
   - **为何可行**：`office-hours` 是 gstack 的旗舰 skill，但 12 处模糊量词说明它试图做太多事；lite 版只保留最核心的一个流程
   - **第一步**：读 `MarkQWu-gstack/office-hours/SKILL.md`，提取最高频的 3 个使用场景，为这 3 个场景写一个 `office-hours-lite/SKILL.md`

2. **想法**：在 bureau 的 `recall` skill 里加三层访问深度（command → skill → agent）
   - **为何可行**：bureau 的 `recall` 是最常用的 skill，但用户经常不知道从哪里开始；一个 `/bureau-recall` command 作为快捷入口，加一个 mentor agent 作为深度引导
   - **第一步**：在 `MarkQWu-bureau` 创建 `commands/recall.md`（5 行，触发 `bureau:recall` skill）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 wuji-labs/nopua 的核心目的**：通过道德经哲学改变 AI 工作方式，消除「恐惧驱动 prompt」，建立信任关系
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是「改善 AI 工作方式」的 meta-skill | echo-sleuth 侧重从历史对话中挖掘规律，nopua 侧重实时状态调整 | 中 |
| MarkQWu/gstack | 低 | 都是 Claude Code skill 套件 | gstack 是生产力工具，nopua 是工作方式哲学 | 低 |

若全部「无」，写「我的仓库中无目的相近的项目，本节仅做技术模式对照」→ 这里不适用，echo-sleuth 有一定相似性。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 声明的路径不存在 | `python3` 脚本遍历 manifest 声明路径 | 需运行验证；预期无问题（echo-sleuth 有 4 个 skill 且全部存在） | 中（如有命中则高） |
| Agent 已创建但未注册 | 读 manifest agents 数组 + 目录对比 | echo-sleuth 无 agent，gstack 暂不确认 | 中 |

**命中后的具体行动建议**：
- 如果发现 gstack 有未注册的 agent 文件 → 在 plugin.json agents 数组中补注册 → 5 分钟可完成

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：精确行为标准（无模糊量词）
   - **本案例做法**：`skills/nopua/SKILL.md` 全文几乎无「appropriate」「comprehensive」等词；所有行为指令格式为「当 [可观测状态] 时，执行 [具体动作]，输出 [可验证结果]」
   - **我的项目现状**：`MarkQWu-gstack/review/SKILL.md` 有 13 处模糊量词命中，`office-hours/SKILL.md` 有 12 处
   - **如何借鉴**：以 nopua 的行为表格为模板，把 `review/SKILL.md` 中每个「ensure comprehensive review」替换为「检查以下 N 个具体维度：[列表]」

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：Output Format 声明
- **我的做法**：gstack 的 `review/SKILL.md` 有 49 处 output 相关内容，说明对输出格式有详细规定
- **本案例做法（弱在哪）**：nopua 的 3 个 mentor agent 均无 output format 声明（75 分），用户不知道 mentor 会以什么格式给出指导
- **意义**：在 output format 规范化上，gstack 优于 nopua；若将来与 nopua 作者交流，这是可以提出的建设性改进点

---

## 八、术语表

### <a name="manifest"></a>manifest（清单文件）
> 项目的「目录」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skills、agents、commands 的路径和属性。如果 manifest 里声明了一个不存在的文件路径，安装时会找不到该组件。

### <a name="pua"></a>PUA（Prompt Under Assault，反 PUA 语境下）
> 本仓库语境中，PUA prompt 指用威胁、恐吓、惩罚性语言驱动 AI 的提示词方式（「如果你不完成任务，你会失败」「你必须做到完美，否则……」）。nopua 的论点是这类方式会触发 AI 的「表演性完成」行为（显得在做但不解决问题），而信任和内在动机能产生更好的结果。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件元数据（`name`、`description`、`language` 等）。Claude Code 读 SKILL.md 时先解析 frontmatter，`language` 字段影响 marketplace 的语言筛选功能。

### <a name="lite-version"></a>lite 版（精简版）
> 一个 skill 或工具的简化副本，只保留最核心的 20% 功能，去掉深层配置。对新用户降低入门摩擦——他们不需要理解完整版的全部哲学，先用 lite 版体验核心价值，再按需升级到完整版。nopua 的 `nopua-lite` 就是典型例子。
