# expo/skills — 学习案例

**仓库**：https://github.com/expo/skills
**Stars**：1784 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-04（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `cross-reference`, `vague-quantifier`, `template-design`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
expo/skills 是 Expo 官方维护的 Claude Code 插件集合，专为构建 Expo 移动应用提供 AI 辅助技能。Expo 是业内主流的 React Native 开发框架（Meta/Google 推荐的跨平台 App 开发工具），约 100 万开发者在用。这个仓库是 Expo 官方在 Claude Code 生态内的 AI 辅助「官方插件」，权威性高，但维护方式偏「自动化持续集成」而非手工精细打磨。

关键事实：
- 13 个 SKILL.md 涵盖 UI 构建、部署、CI/CD、SDK 升级、EAS 分析等 Expo 开发全流程
- 单一插件（`plugins/expo/`）结构，通过 `claude plugin install expo@expo` 安装
- CLAUDE.md 里的目录结构说明严重过时（描述三个独立插件，实际只有一个）

### 1.2 架构剖析

**目录结构**：
```
plugins/expo/
  .claude-plugin/plugin.json        # 插件清单（95/100分）
  skills/
    eas-update-insights/SKILL.md    # EAS 更新分析（98/100，标杆文件）
    expo-module/SKILL.md            # 原生模块开发（97/100）
    upgrading-expo/SKILL.md         # SDK 升级指导（93/100）
    expo-dev-client/SKILL.md        # Dev Client 使用（95/100）
    expo-tailwind-setup/SKILL.md    # Tailwind 配置（95/100）
    use-dom/SKILL.md                # DOM 组件（95/100）
    expo-api-routes/SKILL.md        # API 路由（96/100）
    native-data-fetching/SKILL.md   # 原生数据获取（96/100）
    building-native-ui/SKILL.md     # 原生 UI 构建（92/100）
    expo-cicd-workflows/SKILL.md    # CI/CD 工作流（91/100）
    expo-ui-jetpack-compose/SKILL.md # Jetpack Compose（88/100）
    expo-ui-swift-ui/SKILL.md       # SwiftUI（80/100，有 Bug）
    expo-deployment/SKILL.md        # 部署流程（85/100，有 Bug）
    expo-cicd-workflows/scripts/    # fetch.js, validate.js（唯二可执行文件）
CLAUDE.md                           # 严重过时的仓库结构说明
```

**文件类型分布**：13 个 SKILL.md / 0 个 command / 0 个 agent / 1 个 plugin.json / 2 个 JS 脚本

**编排关系**：完全平列——13 个 skill 相互独立，没有 router、没有 orchestrator。用户根据任务类型直接触发对应 skill。是典型的「技能手册」模式，不是流水线。

**跨件契约**：expo-cicd-workflows SKILL.md 里引用 `{baseDir}` 变量，期望 Claude Code 在运行时替换，但没有文档说明这是自动替换还是需要用户手动填写（这是一个 bug 级别的含糊）。deployment SKILL.md 的顶层 References 用 `./references/`，正文里用 `./reference/`（少了个 s），一半链接实际失效。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「官方框架文档的 AI 适配层」——每个 skill 对应 Expo 官方文档的一个核心场景，把文档里「适合人类阅读」的内容重构为「适合 AI 执行」的步骤化指令。

**解决什么问题**：Expo 文档齐全但分散，开发者在实际开发中需要频繁切换文档场景（部署、CI/CD、升级、原生模块……），每次都要重新在文档里定位。这些 skills 把「我要部署我的 Expo App」变成一个直接可执行的 Claude 技能。

**做了什么 trade-off**：宽度 vs 深度——13 个 skill 覆盖面广，但每个 skill 的平均质量参差不齐（最高 98，最低 80）。相比每个 skill 都精打细磨，Expo 选择了快速覆盖所有场景。

**反映什么认知模型**：作者（Expo 团队）把 skills 看作「文档 API」——覆盖的场景越多越好，精度是其次。这个认知模型在「官方维护、用户基数大」的场景下合理，但在「小团队、资源有限」的场景下会造成质量债。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「框架专属技能手册」模式**：一个框架对应一个插件，插件里所有 skills 都围绕该框架的核心使用场景展开，skill 之间无依赖，用户按需触发。

模式特征清单：
- 特征 1：skill 数量 ≥ 10，每个覆盖框架的一个典型场景
- 特征 2：skill 命名格式为 `{框架名}-{场景}`（如 `expo-deployment`、`expo-cicd-workflows`）
- 特征 3：有一两个「标杆 skill」质量接近满分（eas-update-insights 98/100）
- 特征 4：不需要 router/orchestrator——用户自己知道该用哪个技能
- 特征 5：参考文件（references/）存在于部分 skill 的子目录里，但不强制所有 skill 都有

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 某个技术框架/平台的官方 AI 辅助工具 | ✅ 高度适用 | 用户按场景查找，无需了解内部依赖 |
| 跨框架通用技能（如代码审查、文档生成） | ❌ 不适用 | 通用场景需要 router 把用户路由到正确 skill |
| 技能间有明确调用顺序的工作流 | ❌ 不适用 | 平列结构无法表达「先做 A 再做 B」的依赖 |
| 需要快速覆盖大量场景、允许质量参差 | ✅ 适用 | 宽度优先的合理场景 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 框架专属技能手册（本仓库） | expo/skills | 用户体验简单，按需查找；官方维护权威性高 | skill 间无协同；质量参差不齐 |
| 路由+渠道分层（Router） | 0xmariowu/Autosearch | AI 自动选择渠道，减少用户认知负担 | 结构复杂，维护成本高 |
| 单一深度 skill | antfu/skills | 每个 skill 打磨充分 | 场景覆盖有限 |

### 2.4 改进空间

1. **当前问题**：CLAUDE.md 描述了不存在的三个独立插件目录 **改进做法**：更新 CLAUDE.md 的 Repository Structure 章节，描述实际的 `plugins/expo/` 单插件结构（10 分钟） **预期收益**：新贡献者不会因为按旧文档找文件而困惑

2. **当前问题**：expo-deployment 里 `./references/` 和 `./reference/` 混用，实际只有 `references/`（有 s），导致 5 个 in-body 链接全部失效 **改进做法**：全局 `sed -i 's|./reference/|./references/|g' plugins/expo/skills/expo-deployment/SKILL.md`（2 分钟） **预期收益**：Claude 能正确读取引用文件，部署指导质量直接提升

3. **当前问题**：expo-ui-swift-ui 的代码示例 import path 是 `@expo-ui/swift-ui`，与 frontmatter 里的 `@expo/ui/swift-ui` 不一致 **改进做法**：改代码示例第 27 行 → `import { Host, VStack, RNHostView } from "@expo/ui/swift-ui";` **预期收益**：开发者复制示例代码不会遇到模块找不到的错误

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-26 当时得分 92/100。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 80 | 仓库结构说明已过时，安装命令指向不存在的插件名 |
| expo-ui-swift-ui/SKILL.md | 80 | import path 错误（Bug #1） |
| expo-deployment/SKILL.md | 85 | references 路径不一致（Bug #2） |
| expo-ui-jetpack-compose/SKILL.md | 88 | name 字段不是 kebab-case |
| building-native-ui/SKILL.md | 92 | 4 处模糊量词 |
| eas-update-insights/SKILL.md | 98 | 标杆（无问题） |

### 3.2 当时值得借鉴的模式

1. **`eas-update-insights` 作为标杆 skill** → 该文件在指令清晰度、示例丰富度、边界条件处理上都接近满分，是「这个仓库应该怎么写」的样板 → 写多 skill 插件时，先把一个 skill 打磨到 95+，再用它作为格式/风格模板

2. **Script 文件放在 skill 子目录里（expo-cicd-workflows/scripts/）** → 脚本跟着 skill 走，不放在全局 scripts/，保持 skill 自包含 → `expo-cicd-workflows/scripts/fetch.js`、`validate.js`

3. **每个 skill 有独立的 `references/` 子目录** → 支持文档的参考材料（workflows.md、testflight.md 等）与 skill 主体分开，保持 SKILL.md 简洁 → `expo-deployment/references/` 下的 5 个 Markdown 文件

4. **plugin.json 评分 95/100，结构规范** → 字段齐全、格式正确，manifest 本身不拖累总分

### 3.3 当时的缺陷

1. **CLAUDE.md 和实际目录结构严重不符** → 当 CLAUDE.md 说「这里有三个插件」但实际只有一个时，新贡献者会花时间找不存在的文件；说明文档和代码不同步是「积累式技术债」，越晚修越贵 → 自查：我的 claude-for-legal 有一个 CLAUDE.md，需要验证它的目录树是否和实际一致

2. **`expo-ui-swift-ui` 的 import 路径错误（Bug #1）** → `@expo-ui/swift-ui` vs `@expo/ui/swift-ui` 是两个完全不同的 npm 包；代码示例里的错误会直接导致开发者的项目编译失败 → 根本原因：技术文档在快速迭代时经常不同步，frontmatter 更新了 import 路径，但代码示例没有跟着改 → 自查：我有没有 skill 里的代码示例和 frontmatter 描述不一致的情况？

3. **模糊量词密度在 building-native-ui 里偏高**（"almost always" ×2、"Whenever possible"、"frequently"）→ 4 处模糊指令让模型无法做出确定性决策 → 根本原因：从人类文档「建议」改写为 AI「指令」时，自然语言的「通常应该」没有转换为「具体条件下应该」

### 3.4 当时的优化机会

1. 修复 expo-ui-swift-ui import 路径（最高用户影响，一行代码）
2. 修复 expo-deployment 的 references/reference 路径不一致（5处链接全部失效，一次 sed 可解决）
3. CLAUDE.md 更新仓库结构描述

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| expo-ui-swift-ui import 路径错误（L26） | `grep -n "expo-ui/swift-ui" skills/expo-ui-swift-ui/SKILL.md` | **仍存在**（L26 仍为 `@expo-ui/swift-ui`） | Audit 后 2 个月未修，说明维护节奏不追踪 NLPM 报告 |
| expo-deployment references/reference 混用 | `grep -n "reference" skills/expo-deployment/SKILL.md` | **仍存在**（L16-20 用 `./references/`，L123+ 用 `./reference/`） | 5个失效链接仍未修复；开发者使用部署 skill 时 Claude 读不到参考文档 |
| CLAUDE.md 仓库结构过时 | `head -25 CLAUDE.md` | **仍存在**（说明三个独立插件目录，实际只有一个） | 至今未更新，积累了 2 个月的信息债 |

### 4.2 架构演进

Audit 时的 skill 集合和当前 HEAD 基本相同（13 个 skill），没有新增也没有删除。说明 Expo 团队目前以「维护现有 skills 质量」为主，不在快速扩展场景覆盖。主要变化：
- `plugins/expo/.mcp.json` 新增（MCP 配置文件），说明 Expo 开始集成 MCP 服务器来增强某些 skill 的工具访问能力

### 4.3 新增的可学习模式

`.mcp.json` 文件出现在 `plugins/expo/` 目录下：插件可以随附 MCP 配置，让 skill 在执行时能访问特定的 MCP 服务器工具。这是 「skill + MCP 双层增强」的新模式——NL 指令（skill）和工具权限（MCP）分别声明，组合使用。

---

## 五、校准

### 5.1 我已经在做对的

1. **echo-sleuth 所有命令的 `name` frontmatter 完整**：expo/skills 没有 command，但对比之下 echo-sleuth 命令层的规范度更高
2. **skill description 使用 "This skill should be used when..." 格式**：echo-sleuth 的 skills 基本符合触发格式，expo 的 expo-ui-jetpack-compose 的 name 不是 kebab-case 我没有犯这个错误
3. **references/ 目录结构已经在部分 skill 里使用**：echo-sleuth 的 jsonl-core、git-mining 都有 references/ 目录

### 5.2 挑战 / 验证

这次案例挑战了我「官方仓库一般质量有保证」的假设——expo 是一线框架官方出品，但 Bug #1（import 路径错误）和 Bug #2（links 全部失效）在 audit 后 2 个月都没修，说明「官方」不等于「自我审查严格」。

验证了：NLPM 发现的 bug（即使是 medium/low 级别）如果没有人跟进，会无限期存留。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 里代码示例的 import 路径和 frontmatter 描述是否一致
grep -n "import\|from " /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null | head -20
```
命中后：对比 frontmatter 的 `description` 提到的包名，确认代码示例里的 import 路径是否匹配。

```bash
# 检查我的 SKILL.md 里 references 路径是否存在
for f in $(find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "SKILL.md"); do
  dir=$(dirname "$f")
  grep -o '\./references/[^)]*' "$f" 2>/dev/null | while read ref; do
    [ ! -f "${dir}/${ref#./}" ] && echo "BROKEN: $f -> $ref"
  done
done
```
命中后：修正路径或补充缺失的参考文件。

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 补充一个「标杆 skill」（类似 eas-update-insights 的角色），作为其他 skill 的格式参考  
   **为何可行**：当前 echo-sleuth 的 jsonl-core skill 质量最高，可以显式标注为「格式参考」  
   **第一步**：在 CLAUDE.md 里加一句 "jsonl-core/SKILL.md 是本插件的格式标杆"（5分钟）

2. **想法**：建立一个「link checker」脚本，在每次写完 SKILL.md 后自动检查 references/ 里的链接是否存在  
   **为何可行**：expo 的 Bug #2 完全可以被一个 10 行 shell 脚本在提交前检测出来  
   **第一步**：写 `scripts/check-references.sh`，循环检查所有 SKILL.md 里的 `./references/` 路径是否对应真实文件（30分钟）

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 expo/skills 的核心目的**：为 Expo 框架开发提供 AI 辅助技能集合，覆盖从 UI 构建到部署的全流程
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 都是多 skill 插件，技能间相互独立 | expo 面向框架开发；echo-sleuth 面向历史挖掘 | 中 |
| MarkQWu/claude-for-legal | 高 | 都是「领域专属技能手册」，多技能覆盖同一领域的不同场景 | expo 是技术框架；claude-for-legal 是法律工作流 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 都是 Claude Code 插件 | 不同领域 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 命令缺少 `allowed-tools` | `find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/commands -name "*.md" \| xargs grep -L "allowed-tools"` | echo-sleuth 命中 4/8：recap.md、timeline.md、recall.md、lessons.md | 中 |
| 模糊量词 "almost always" | `grep -rn "almost always" /tmp/my-repos/MarkQWu-claude-for-legal` | claude-for-legal 命中 3 处 | 低 |

**命中后的具体行动建议**：
- `echo-sleuth/commands/recap.md` → 添加 `allowed-tools: [Read]`（5分钟，重复 4 个文件）
- `echo-sleuth/commands/recall.md` → 确认它是否用了 Write 工具，若是加 `allowed-tools: [Read, Write]`

### 8.3 别人的更优方案

1. **领域**：单个 skill 文件作为「标杆」的显式定位
   - **本案例做法**：audit 中明确指出 `eas-update-insights/SKILL.md` 是「model skill for the collection」；该文件 98/100，在整个集合里作为格式参考
   - **我的项目现状**：echo-sleuth 没有明确的格式标杆，各 skill 格式稍有差异（memory-management 有 `version` 字段，其他没有）
   - **如何借鉴**：在 echo-sleuth README 里注明哪个 skill 是格式参考，保持一致性

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：命令的 `allowed-tools` 覆盖率
- **我的做法**：echo-sleuth 8 个命令中有 4 个带 `allowed-tools`（50%），drama-workshop-skills 没有命令，claude-for-legal 也有类似声明
- **本案例做法**：expo/skills 没有 command 文件，无法对比；但对比 eyaltoledano/claude-task-master（47 个命令全部缺 allowed-tools）我的 echo-sleuth 做得更好
- **意义**：allowed-tools 声明是 Claude Code 安全性的基础——声明了才知道这个命令应该有哪些权限，未来 audit 时是加分项

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包起来的 YAML 配置块，声明 skill 的名称、描述、工具权限等元数据。

### <a name="kebab-case"></a>kebab-case
> 单词之间用连字符 `-` 连接的命名风格，例如 `expo-ui-swift-ui`。Claude Code plugin 规范要求 skill `name` 字段必须使用 kebab-case 小写格式，不能用空格或大写（如 "Expo UI SwiftUI" 是错误的）。

### <a name="SSRF"></a>SSRF
> Server-Side Request Forgery（服务端请求伪造）。如果一个脚本接受用户输入的 URL 然后直接发起 HTTP 请求，攻击者可能提供内部网络地址（如 `http://169.254.169.254/` AWS 元数据接口），让脚本代替他访问原本无法访问的资源。expo-cicd-workflows 的 `fetch.js` 就存在这个风险。

### <a name="EAS"></a>EAS
> Expo Application Services。Expo 提供的云端构建和发布服务，包括 EAS Build（云端构建 App）和 EAS Update（OTA 更新）。`eas-update-insights` skill 专门帮助分析 EAS Update 的发布数据。
