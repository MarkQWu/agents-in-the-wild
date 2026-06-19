# lackeyjb/playwright-skill — 学习案例

**仓库**：https://github.com/lackeyjb/playwright-skill
**Stars**：2607 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-12（历史快照）| **生成日期**：2026-06-19（基于当前 HEAD）
**主题标签**：`examples-driven`, `manifest-discipline`, `single-purpose`, `nl-binary-hybrid`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

lackeyjb/playwright-skill 是目前评分最高的 Claude Code 技能之一（98/100），2607 Stars。仓库非常精简：**一个技能**（browser automation）+ **一个 Node.js 执行器**（run.js）+ **完整的 plugin [manifest](#manifest)**。它解决的问题是：让 Claude 能够自主编写并立即执行 Playwright 浏览器自动化脚本，实现"我帮你点网页"的效果——无需用户手写代码。作者 lackeyjb（Jordan Lackey）是前端测试领域的实践者，将多年 Playwright 经验蒸馏成了这个技能。

### 1.2 架构剖析

**目录结构**：
```
playwright-skill/
├── skills/
│   └── playwright-skill/
│       ├── SKILL.md          ← 技能定义（454 行，接近 R05 的 500 行上限）
│       ├── run.js            ← 通用 Playwright 执行器（接受脚本路径作为参数）
│       ├── lib/
│       │   └── helpers.js    ← dev server 自动发现逻辑
│       ├── API_REFERENCE.md  ← Playwright API 参考文档
│       └── package.json      ← playwright 依赖声明
└── .claude-plugin/
    ├── plugin.json           ← Claude Code plugin manifest（100/100）
    └── marketplace.json      ← Claude marketplace 注册信息
```

**文件类型分布**：1 个 SKILL.md，0 个独立 agent，0 个 hook，2 个 [manifest](#manifest) 文件，2 个 JS 执行文件。

**编排关系**：单一技能，无 orchestrator，无跨件调用。SKILL.md 是控制层，run.js 是执行层——典型的 [NL 表皮 + 原生二进制核心](#nl-binary-hybrid)模式（NL 表皮 = SKILL.md 描述行为，原生执行层 = Node.js/Playwright）。

**跨件契约**：`SKILL.md` 明确引用 `API_REFERENCE.md`（位于同目录），`run.js` 接受脚本路径参数，`helpers.js` 提供 dev server 发现能力。三者形成一个紧密的执行单元，内聚性高。

### 1.3 设计思路 / 方法论

**核心设计哲学**：「/tmp 隔离 + 可见浏览器」——Claude 生成的测试脚本写入 `/tmp` 目录（不污染项目），默认使用非 headless（用户可以看到浏览器在操作什么）。这两个设计决定直接降低了"AI 乱改我的项目"的焦虑感。

**解决什么问题**：浏览器自动化脚本编写门槛高（Playwright API 复杂、选择器易碎、异步处理容易出错），且传统 E2E 测试框架需要复杂的项目配置。本技能让用户用自然语言描述想要测试的场景，Claude 负责生成和执行脚本。

**做了什么 trade-off**：选择了"AI 动态生成脚本"而非"提供固定脚本模板"——灵活性高但需要 Claude 有足够的 Playwright 上下文（这也是为什么 `API_REFERENCE.md` 和 SKILL.md 中有大量示例的原因）。

**反映什么认知模型**：作者把 AI 定位为"脚本编写者"而非"测试框架"——Claude 不维护状态，不管理测试套件，每次请求是独立的"帮我做这件浏览器操作"任务。这与传统 QA 工具的设计哲学完全不同。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「NL 表皮 + 原生执行核心（单焦点型）」**

SKILL.md 描述行为和约束，具体执行委托给同目录下的 Node.js 执行器（run.js）。关键特征是：NL 层（SKILL.md）是可读的行为规范，原生层（run.js）是确定性的执行机器，两者通过文件路径参数传递脚本内容。

模式特征清单：
- **单一职责**：整个仓库只解决一个问题（browser automation），深度而非广度
- **执行器封装**：run.js 作为通用执行入口，接受 LLM 生成的脚本路径，避免每次都用 `node -e "..."` 内联执行
- **/tmp 隔离**：所有生成的脚本写入 `/tmp`，不影响用户项目目录
- **环境感知启动**：helpers.js 的 dev server 自动发现在工作流开始前收集运行时信息，避免 Claude 猜测端口号
- **454 行技能体**：接近 NLPM 的 500 行上限，说明作者把大量 context（API 示例、工作流步骤、边界条件处理）都内联在 SKILL.md 中

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 一次性 UI 验证（冒烟测试） | ✅ 高度适用 | 无需维护测试套件，一次性脚本正好 |
| 持续集成（CI）回归测试 | ❌ 不适用 | 生成脚本不持久化、每次需要 LLM 生成，成本高且不稳定 |
| API 响应时间性能测试 | ❌ 不适用 | Playwright 侧重 UI 自动化，不是性能工具 |
| 爬虫 / 数据采集 | ⚠️ 有限适用 | 技术上可以，但技能设计面向测试场景，非采集场景 |
| 无障碍（a11y）检查 | ✅ 适用 | 可以要求 Claude 添加 Axe 等 a11y 工具到生成脚本中 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮 + 原生执行核心（本仓库） | lackeyjb/playwright-skill | 技能描述和执行引擎分离，各自演进；执行稳定 | 需要用户机器有 Node.js 和 Playwright 依赖 |
| 纯 NL 技能（无执行器） | 大多数 SKILL.md | 零依赖安装；跨平台 | 执行细节依赖 LLM 在会话中生成，不稳定 |
| 外部 MCP 服务 | playwright-mcp | 服务器端执行，用户机器无需安装 | 需要 MCP 服务器运行，配置更复杂 |

### 2.4 改进空间

1. **当前问题**：marketplace.json 的 `metadata.version` 停留在 "1.0.0"，而实际版本是 "4.1.0"，registry 查询时会返回错误的版本号。**改进做法**：建立 `npm version patch/minor/major` 钩子，自动同步 plugin.json 和 marketplace.json 中的所有 version 字段。**预期收益**：消除版本漂移，让 marketplace 索引显示正确版本。

2. **当前问题**：`package.json` 用 `^1.57.0` [宽松版本范围](#semver)允许 Playwright 自动升级。**改进做法**：改为精确版本 `"playwright": "1.57.0"` + 提交 `package-lock.json` + 文档中说明"升级 Playwright 步骤"。**预期收益**：防止 Playwright 小版本更新引入 breaking change 或供应链风险。

3. **当前问题**："comprehensive Playwright API documentation"（line 373）和 "clean test scripts"（line 3）是主观描述。**改进做法**：替换为 "API_REFERENCE.md covers selectors, auth, network, CI patterns"；"writes self-contained test scripts"（去掉 clean）。

---

## 三、过去审查发现（2026-05-12 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-05-12 当时得分 **98/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/playwright-skill/SKILL.md | 97 | "comprehensive"（borderline R01 模糊量词，-2）；"clean"（主观，-1） |
| .claude-plugin/plugin.json | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **「描述即触发条件（R04 范例）」** → `description` 字段列出了 8 个具体的用户触发场景（"test pages, fill forms, take screenshots, check responsive design, validate UX, test login flows, check links, automate any browser task"），每个都是用户的自然语言用词。借鉴点：`description` 不是技能摘要，是触发条件列表，越具体越好。路径：`skills/playwright-skill/SKILL.md` 第 2-3 行。

2. **「CRITICAL WORKFLOW 编号步骤（R14 精神）」** → SKILL.md 中的核心工作流用 "CRITICAL WORKFLOW - Follow these steps in order" 标题 + 编号步骤（1. Auto-detect → 2. Write scripts → 3. Use visible browser → 4. Parameterize URLs），为 Claude 提供了明确的执行顺序约束。借鉴点：复杂技能必须用显式的步骤编号声明执行顺序，而不是用自然段落描述。

3. **「多形态安装路径适配」** → SKILL.md 开头说明了 3 种安装位置（Plugin system / Manual global / Project-specific）并让 Claude 在执行前先发现实际路径。借鉴点：面向多种部署环境的技能，应在技能体内处理路径不确定性，而非假设固定路径。

4. **「plugin.json 完整字段（100/100）」** → `name`、`version`、`description`、`author`、`license`、`repository`、`keywords` 全部填写，且 `keywords` 中包含 `"model-invoked"` 表明这是运行时激活的技能。借鉴点：manifest 字段宁多勿少，每个字段都是 marketplace 索引和用户发现的机会。

5. **「示例驱动的 6 个代码模式 + 2 个交互示例」** → `## Example Usage` 部分提供了 6 个具体的 Playwright 代码片段（基础测试、表单填写、截图、响应式检查、登录、链接验证），直接降低了 Claude 生成质量不稳定的风险。借鉴点：给 Claude 展示"好的输出长什么样"比用文字描述"应该怎么做"更有效。

### 3.3 当时的缺陷

1. **marketplace.json 版本漂移（BUG）** → `metadata.version: "1.0.0"` vs `plugins[0].version: "4.1.0"` vs `plugin.json version: "4.1.0"`。根本原因：版本升级时更新了 plugin.json 和 plugins[] 数组内的 version，但忘记同步外层 metadata.version。这是典型的"多处维护版本号"的脆弱性——只要有多个版本字段，就会有漂移风险。自查：echo-sleuth 的 marketplace.json 是否有类似漂移？

2. **package.json 使用 `^` 宽松版本范围** → 允许 Playwright minor 版本自动升级，任何包含 breaking change 或供应链攻击的 minor 版本都会静默传播到所有用户。

3. **"clean test scripts" 和 "comprehensive Playwright API"** → 主观形容词，违反 R01 精神。"clean" 没有告诉 Claude 任何可测量的标准。

### 3.4 当时的优化机会

1. 将 `metadata.version` 从 "1.0.0" 改为 "4.1.0"（单行修复，1 分钟）。
2. 在 `package.json` 中将 `"^1.57.0"` 改为 `"1.57.0"`，并提交 `package-lock.json`。
3. 将 "comprehensive Playwright API documentation" 改为 "For selectors, network interception, auth, and CI/CD patterns, see API_REFERENCE.md"。

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| marketplace.json metadata.version 漂移 | `cat .claude-plugin/marketplace.json \| grep version` | **仍存在**：metadata.version = "1.0.0"，plugins[0].version = "4.1.0" | 这个 1 行修复超过 5 周未被合并，说明作者或许不知道 metadata.version 字段有意义 |
| package.json 宽松版本 `^1.57.0` | `cat skills/playwright-skill/package.json` | **仍存在**：`"playwright": "^1.57.0"` | 安全风险依然存在 |
| 主观形容词 "clean" / "comprehensive" | 读 SKILL.md 描述行 | **仍存在**：描述第 3 行 "writes clean test scripts to /tmp" 未改 | 低优先级，对功能无影响 |

### 4.2 架构演进

结构与 audit 时完全一致，无新文件新增，无目录重组。这说明作者将仓库视为"稳定版本"而非持续演进，与 2607 stars 说明的用户规模相符——一个稳定工作的高质量单技能产品。

### 4.3 新增的可学习模式

暂无。当前 HEAD 与 audit 快照一致，无结构性变化。

---

## 五、校准

### 5.1 我已经在做对的

1. **/tmp 隔离原则**：在涉及文件生成的技能中，将临时文件写入 `/tmp` 而非项目目录。drama-workshop-skills 的输出写入 `~/Documents/notes/`（用户文档目录）而非项目内，符合相同的隔离精神。✓

2. **描述触发场景而非技术实现**：drama-workshop-skills 的 description 写的是"当用户说 /开始 或 '我想写一部都市爱情剧'"，而非"使用 AI 生成剧本内容"。✓

3. **API 参考文档分离**：echo-sleuth-for-claude 有独立的 `references/` 目录存放 skill 引用的规范文档，与本仓库的 `API_REFERENCE.md` 思路一致。✓

### 5.2 挑战 / 验证

**验证了什么**：一个仓库只要把一件事做到极致（98/100），就能获得 2607 stars——市场对"深度 > 广度"的回报是真实的。我的 drama-workshop-skills 和 echo-sleuth-for-claude 都采用了"只做一件事、做好"的策略，方向正确。

**挑战了什么假设**：我以为 2607 stars 的仓库一定是维护非常活跃的——但这个仓库在发布后几乎没有更新，连 1 行的版本号修复都没有合并。说明"质量足够高的时候，不活跃也能维持口碑"。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 marketplace.json 中是否有版本漂移
for repo in ~/projects/my-skills/*/; do
  if [ -f "$repo/.claude-plugin/marketplace.json" ]; then
    echo "=== $repo ==="
    python3 -c "
import json
d = json.load(open('${repo}/.claude-plugin/marketplace.json'))
meta_ver = d.get('metadata', {}).get('version', 'N/A')
plugin_ver = d.get('plugins', [{}])[0].get('version', 'N/A')
if meta_ver != plugin_ver:
    print(f'DRIFT: metadata={meta_ver} plugins[0]={plugin_ver}')
else:
    print(f'OK: {meta_ver}')
"
  fi
done
# 命中后：手动将 metadata.version 同步到与 plugins[0].version 相同

# 检查 package.json 中是否有 caret 宽松版本
find . -name "package.json" -not -path "*/node_modules/*" | \
  xargs grep -n '"\^' 2>/dev/null
# 命中后：决策是否改为精确版本 + 提交 lock file
```

### 6.2 灵感 → 实施路径

1. **想法**：将 drama-workshop-skills 的核心技能拆出一个"执行器"脚本（类似 run.js），负责将生成的剧本内容以标准格式写入 `~/Documents/notes/` 并返回文件路径。
   - **为何可行**：drama-workshop 已经有固定的输出路径约定，只差一个标准的写入脚本让输出可测试。
   - **第一步**：创建 `short-drama/assets/export.sh`，接受标题和内容作为参数，按命名约定写入目标目录并打印结果路径。约 20 分钟完成。

2. **想法**：参照 playwright-skill 的"CRITICAL WORKFLOW + 编号步骤"格式，重写 drama-workshop-skills 的主技能的工作流部分。
   - **为何可行**：目前 drama-workshop 的工作流描述是连续段落，缺乏强制顺序标注。添加 "CRITICAL: Follow these steps in order" + 编号列表即可。
   - **第一步**：找到 `short-drama/SKILL.md` 中的工作流描述段，加 `## 核心工作流（必须按序执行）` 标题 + 编号 1-6。约 15 分钟完成。

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度

- **本案例 lackeyjb/playwright-skill 的核心目的**：让 Claude Code 能够代替用户编写和执行 Playwright 浏览器自动化脚本，用于 UI 测试、页面验证和浏览器交互自动化。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 低 | 都是单一领域的深度技能集 | drama 是内容创作，playwright 是技术测试 | 高（架构模式借鉴） |
| MarkQWu/echo-sleuth-for-claude | 中 | 都有 NL 表皮 + 原生执行层的思路（echo-sleuth 依赖 git log 等命令） | echo-sleuth 的执行层是系统命令而非 Node.js 脚本 | 中 |
| MarkQWu/claude-for-legal | 低 | 都是单领域深度工具 | claude-for-legal 无执行层（纯 NL） | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| marketplace.json 版本漂移 | 比对 metadata.version vs plugins[0].version | **echo-sleuth-for-claude**：marketplace.json 无 metadata.version 字段（空字段），plugins[0].version = "0.4.0"；无漂移问题但有字段缺失 | 低 |
| package.json 宽松版本 `^` | `grep '"\^' */package.json` | **drama-workshop-skills 未命中**（无 package.json） | 暂无 |
| 主观形容词 "clean" / "comprehensive" | `grep -n "clean\|comprehensive\|robust\|efficient" */skills/*/SKILL.md` | **需要进一步检查** | 低 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/.claude-plugin/marketplace.json` → 补充 `"metadata": { "version": "0.4.0" }` 字段，与 plugins[0].version 一致 → 约 2 分钟

### 7.3 别人的更优方案（值得借鉴的）

1. **领域**：SKILL.md 中的 6 个具体代码示例
   - **本案例做法**：`skills/playwright-skill/SKILL.md` 的 `## Example Usage` 提供了 6 个完整的 Playwright 代码片段（每个对应一个测试场景），Claude 可以直接参照生成同类代码
   - **我的项目现状**：drama-workshop-skills 的 SKILL.md 无任何 `<example>` 块，用户和 Claude 都需要靠经验摸索输出格式
   - **如何借鉴**：为 `short-drama/SKILL.md` 添加 2-3 个完整示例：User: `/开始 [都市爱情剧，女主是厨师]` → Assistant: [标准的策划输出格式]。约 30 分钟完成

2. **领域**：plugin.json 的 `keywords` 字段
   - **本案例做法**：9 个关键词（`claude-skill`, `playwright`, `browser-automation`, `testing`, `e2e`, `web-testing`, `automation`, `model-invoked`），每个都是用户搜索可能用的词
   - **我的项目现状**：echo-sleuth-for-claude 的 plugin.json 有 keywords 但数量较少
   - **如何借鉴**：按照"用户会怎么搜索这个技能"的思路，为 drama-workshop 和 echo-sleuth 各添加 5-8 个 keywords

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：版本号集中维护
  - **我的做法**：drama-workshop-skills 使用 `release-manifest.json` 作为版本权威来源，理论上可以从单一文件同步到其他文件
  - **本案例做法**：版本号分散在 plugin.json 和 marketplace.json 两个文件的 3 个位置，导致了实际发生的漂移（metadata.version vs plugins[0].version vs plugin.json version）
  - **意义**：版本号单一来源原则是正确的设计方向；但需要脚本来强制执行同步，否则手动维护同样会漂移

---

## 八、术语表

### <a name="manifest"></a>manifest（清单文件）
> 插件的"注册表"，告诉 Claude Code 这个插件包含什么。`plugin.json` 是面向 Claude Code 本地加载的 manifest；`marketplace.json` 是面向云端 marketplace 索引的 manifest。两者都包含 version 字段，若不同步就会造成用户看到错误版本。

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心（NL-binary hybrid）
> 一种 Claude Code 技能架构模式：SKILL.md（自然语言描述）是用户可读的"使用手册"，而实际执行由同目录下的原生代码（JavaScript、Python、Shell）完成。NL 层负责告诉 Claude"什么时候用、怎么调用"，原生层负责实际执行逻辑。优点：执行稳定、可单元测试；缺点：需要依赖环境（Node.js、Python 等）。

### <a name="semver"></a>semver 版本号约定
> Semantic Versioning（语义化版本）：版本号格式 `X.Y.Z`（主版本.次版本.补丁版本）。`^1.57.0` 表示"允许自动安装 1.x.x 中任何比 1.57.0 新的版本"（`^` 叫 caret 符号）。问题在于 Playwright 的 minor 版本更新有时包含 breaking change，且恶意的 minor 版本可以在用户不知情的情况下被自动安装。精确版本 `1.57.0`（无前缀）则只安装这个精确版本。
