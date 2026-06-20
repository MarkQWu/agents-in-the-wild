# lackeyjb/playwright-skill — 学习案例

**仓库**：https://github.com/lackeyjb/playwright-skill
**Stars**：2607 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-12（历史快照）| **生成日期**：2026-06-20（基于当前 HEAD）
**主题标签**：`single-purpose`, `examples-driven`, `manifest-discipline`, `nl-binary-hybrid`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

一个专门给 Claude Code 做浏览器自动化的[单职责 skill](#单职责-skill)。安装后 Claude 能自己写 Playwright 代码、在 `/tmp` 执行，并自动侦测本地 dev server，无需用户干预路径或 URL。作者是 @lackeyjb，2607 stars，定位"模型自调用"（`model-invoked`），不是用户主动 `/skill-name` 那种。

三个关键事实：
1. 代码极少——1 个 SKILL.md + 1 个 run.js executor + 1 个 helpers.js（dev server 侦测） + 1 个 API_REFERENCE.md
2. 不需要任何上游 AI skill 知识，纯浏览器自动化领域
3. 安装方式走 Claude Code 插件市场（有 `plugin.json` + `marketplace.json`），也支持手动全局安装

### 1.2 架构剖析

```
playwright-skill/
├── .claude-plugin/
│   ├── plugin.json         # 插件注册元数据（version 4.1.0）
│   └── marketplace.json    # 市场清单（metadata.version 1.0.0 ← 未同步！）
├── skills/
│   └── playwright-skill/
│       ├── SKILL.md        # 唯一 NL artifact（454 行）
│       ├── run.js          # 通用执行器：node run.js /tmp/test.js
│       ├── lib/
│       │   └── helpers.js  # detectDevServers() 扫描本地端口
│       ├── API_REFERENCE.md # Playwright API 备查
│       └── package.json    # playwright ^1.57.0（未固定版本）
└── README.md
```

- **文件类型分布**：1 个 SKILL.md、0 个 agent、0 个 command、0 个 hook
- **编排关系**：单层，无路由。SKILL.md 直接告诉 Claude 如何调用 `run.js`。SKILL.md 是唯一入口，`run.js` 是唯一执行器，不存在多 skill 编排
- **跨件契约**：SKILL.md 内部用 `$SKILL_DIR` 占位符，运行时动态推导安装路径。API_REFERENCE.md 从 SKILL.md 第 373 行引用，路径相对，确认存在

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「写到 /tmp，永不污染项目」——所有测试脚本写到 `/tmp/playwright-test-*.js`，由 OS 自动清理，对用户项目零侵入
- **解决什么问题**：用户描述浏览器任务 → Claude 生成 Playwright 代码 → 自动执行——这个链条在没有 skill 时需要用户手动处理路径、安装、执行，skill 把全流程标准化
- **Trade-off**：
  - 选择 `headless: false`（可见浏览器）作为默认值，牺牲速度换调试体验
  - 选择动态路径发现（`$SKILL_DIR` 在运行时从 SKILL.md 加载路径推导）而非硬编码，牺牲简单性换可移植性
  - 选择单 SKILL.md（不拆子 agent），牺牲扩展性换极低维护成本
- **认知模型**：作者把 skill 看作"工具箱说明书"——不是 Claude 的规划层，是 Claude 的"做这件事的标准操作程序"

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：「极简执行器 + 动态路径发现」**

SKILL.md 只做两件事：(1) 告诉 Claude 怎么推导出 `$SKILL_DIR`；(2) 告诉 Claude 怎么调用 `node run.js <script>`。JS 层做真正的工作。这是最小化 NL 复杂度、最大化代码可测试性的架构。

模式特征清单：
- NL 层（SKILL.md）只含工作流程，不含业务逻辑
- 通过"先侦测 dev server，再写脚本"消除了用户手动输入 URL 的步骤
- 测试脚本写到系统临时目录（`/tmp`），天然隔离
- 路径通过 SKILL.md 加载路径反向推导，不依赖全局变量或环境变量

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 开发者想自动化测试本地 web 应用 | ✅ 高度适用 | dev server 自动侦测 + 写到 /tmp 不干扰项目 |
| 非技术用户做重复性表单填写 | ✅ 适用 | model-invoked，用户不需要知道命令行 |
| 需要高并发、CI/CD 集成场景 | ⚠️ 有限适用 | 无 headless 优先模式、无输出报告文件保存 |
| 需要跨浏览器测试（Firefox、Safari） | ❌ 不适用 | 当前只安装 Chromium，代码层未见其他 browser 支持 |
| 需要持久化测试用例（存入项目）| ❌ 不适用 | /tmp 写法设计上就是临时的 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：极简执行器 | lackeyjb/playwright-skill | 维护成本极低；NL 层清晰 | 扩展难，所有逻辑挤在 run.js |
| 多 skill 分层：不同阶段不同 skill | addyosmani/web-quality-skills | 每个检查维度独立，可组合 | 新用户上手成本高，文件多 |
| 命令驱动：/test, /screenshot 各一个命令 | 假设场景 | 用户意图明确，输出可预期 | 需要用户了解每个命令的边界 |

### 2.4 改进空间

1. **当前问题**：marketplace.json 的 `metadata.version` 是 `1.0.0`，与 plugin.json 的 `4.1.0` 不同步。**改进做法**：在 CONTRIBUTING.md 或发布 checklist 中加一行"更新 marketplace.json metadata.version = plugin.json version"，或用 `npm version` 钩子自动同步。**预期收益**：消除 registry 看到旧版本、用户看到更新失败的混乱。

2. **当前问题**：`playwright ^1.57.0` 使用[语义化版本范围](#语义化版本范围)，允许 minor 版本自动升级。**改进做法**：固定为 `"playwright": "1.57.0"` 并提交 `package-lock.json`，在 README 里加"升级 Playwright 时改这个版本号"的说明。**预期收益**：供应链风险清零，`npm ci` 在任意环境结果可复现。

3. **当前问题**：`API_REFERENCE.md` 被 SKILL.md 第 373 行引用为"comprehensive Playwright API documentation"，但 "comprehensive" 没有量化。**改进做法**：改为具体描述："覆盖 Page、Locator、ElementHandle 三类核心对象的 API 速查表"。**预期收益**：Claude 知道去找什么，避免自己在网上搜。

---

## 三、过去审查发现（2026-05-12 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-05-12 当时得分 **98/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `skills/playwright-skill/SKILL.md` | 97/100 | "comprehensive" 边缘性模糊量词（−2），"clean test scripts" 主观（−1） |
| `.claude-plugin/plugin.json` | 100/100 | 无 |

### 3.2 当时值得借鉴的模式

1. **明确路径发现协议** → 为什么好：跨安装环境（插件市场/手动全局/项目本地）路径不同，SKILL.md 列出全部可能路径并要求 Claude 动态发现，避免硬编码失效 → 原文：SKILL.md 头部"Path Resolution"章节 → 借鉴：任何依赖外部文件的 skill 都应加路径发现章节

2. **CRITICAL WORKFLOW 用编号步骤** → 为什么好：Claude 执行多步任务时容易乱序，编号步骤提供强约束 → 原文：SKILL.md "CRITICAL WORKFLOW - Follow these steps in order" → 借鉴：只要有顺序依赖就用编号，不要用 bullet

3. **6 个代码示例 + 2 个交互示例** → 为什么好：覆盖了常见用例的 input→output，Claude 可以直接参照生成 → 原文：SKILL.md "Example Usage" 章节 → 借鉴：示例数量 ≥ 3 时考虑按场景分组

4. **写到 /tmp 不写项目目录** → 为什么好：避免测试文件污染用户仓库 → 原文：步骤 2 "Write scripts to /tmp" → 借鉴：任何生成临时文件的 skill 都应指定输出到 /tmp

### 3.3 当时的缺陷

1. **marketplace.json metadata.version 未同步** → 根本原因：marketplace.json 里的 `metadata.version` 和 `plugins[0].version` 是两个独立字段，作者升级插件时只更新了 plugin.json 和 plugins[0].version，忘了同步 metadata.version——这两个字段在视觉上不在同一层级，容易遗漏 → 自查：我的 marketplace.json 里有无同样的两层版本字段不同步？

2. **"comprehensive" 是模糊量词（R01 精神）** → 根本原因：描述 API 文档时用了"comprehensive"，但没有说明它覆盖哪些类/方法，Claude 无法知道该文档能不能回答它的具体问题 → 自查：我的 SKILL.md 里有没有"完整的""全面的"这类形容词？

3. **unpinned semver** → 根本原因：package.json 用 `^1.57.0`，允许自动拉取新 minor 版本，任何 Playwright minor 版本的 breaking change 都会悄无声息地破坏 skill → 自查：我的 package.json 里有多少个 `^`？

### 3.4 当时的优化机会

1. **优化机会**：在 plugin.json 里显式声明 `"skills"` 数组，不依赖 auto-discovery（当前 plugin.json 没有 `skills` 字段，靠约定自动发现）
2. **优化机会**：`marketplace.json` 的 `owner.email` 是 `"github@lackeyjb"`，不是合法 email 格式，可以改为 `"lackeyjb@users.noreply.github.com"` 或删除
3. **优化机会**：README 里没有卸载说明，用户不知道怎么移除这个 skill

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| marketplace.json metadata.version=1.0.0 vs plugin.json 4.1.0 | `python3 -c "import json; d=json.load(open('.claude-plugin/marketplace.json')); print(d['metadata']['version'])"` | **仍存在**：metadata.version=1.0.0，plugin.json=4.1.0 | 版本漂移未修复，2+ 个月了 |
| playwright ^1.57.0 unpinned | `grep playwright skills/playwright-skill/package.json` | **仍存在**：`"playwright": "^1.57.0"` | 供应链风险持续，lockfile 未提交 |
| "comprehensive" 模糊量词 | `grep "comprehensive" skills/playwright-skill/SKILL.md` | **仍存在**：line 373 unchanged | 低优先级，不影响功能 |

### 4.2 架构演进

从 audit 到现在，**目录结构无变化**。文件列表和 audit 时完全一致（plugin.json、marketplace.json、SKILL.md、run.js、helpers.js、API_REFERENCE.md、package.json）。作者没有重组，没有新增 agent 或 command。

这说明：作者对这个极简架构是满意的，或者没有时间维护。Bug 没有修复说明这个仓库处于"功能完成，不活跃维护"状态。

### 4.3 新增的可学习模式

暂无——当前 HEAD 与 audit 时完全相同，未发现 audit 之后的新设计。

---

## 五、校准

### 5.1 我已经在做对的

1. **写到临时目录**：我的 drama-workshop-skills 输出到 `~/Downloads/`，不写入项目目录，与 playwright-skill 的 `/tmp` 模式一致
2. **CRITICAL WORKFLOW 编号步骤**：我的 SKILL.md 里也有编号的强约束步骤
3. **model-invoked vs user-invocable 明确区分**：我的 skills 都有 `user_invocable` frontmatter，与 playwright-skill 的 `model-invoked` keyword 是同一种意图的不同表达
4. **单职责**：我的每个 skill 只做一件事，与 playwright-skill 的设计哲学一致

### 5.2 挑战 / 验证

这次案例**验证**了一个我之前不确定的做法：动态路径发现是必要的。playwright-skill 明确列出"Plugin system / Manual global / Project-specific"三种安装路径，说明 Claude Code 插件生态里路径是真的不固定的。我之前在 echo-sleuth 里写死了一些路径，这次确认这是一个需要修复的问题。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 marketplace.json 版本字段是否与 plugin.json 同步
for pj in $(find . -name "plugin.json" -path "*/.claude-plugin/*"); do
  dir=$(dirname "$pj")
  mp="$dir/marketplace.json"
  if [ -f "$mp" ]; then
    pv=$(python3 -c "import json; print(json.load(open('$pj'))['version'])")
    mv=$(python3 -c "import json; d=json.load(open('$mp')); print(d.get('metadata',{}).get('version','NOT_FOUND'))")
    echo "$pj: plugin=$pv, marketplace_meta=$mv"
  fi
done
```
命中后怎么办：把 marketplace.json 的 `metadata.version` 改成与 plugin.json 一致。

```bash
# 检查 package.json 里的 ^ 版本范围
find . -name "package.json" ! -path "*/node_modules/*" | xargs grep -n "\"\\^" 2>/dev/null
```
命中后怎么办：把 `^x.y.z` 改为 `x.y.z`，然后 `npm install` 生成 lockfile 并提交。

### 6.2 灵感 → 实施路径

1. **想法**：给 echo-sleuth 的 scripts/ 加一个"先侦测安装路径"的 helpers 模块，像 playwright-skill 的 helpers.js 那样
   - **为何可行**：echo-sleuth 的 scripts 需要找到 `~/.claude/projects/` 等路径，当前硬编码
   - **第一步**：在 `scripts/` 下新建 `detect-paths.sh`，输出 `PROJECTS_DIR`、`SKILLS_DIR`；在 bash 脚本里 source 它；预计 15 分钟

2. **想法**：给 drama-workshop-skills 加路径发现章节（参考 SKILL.md 头部"Path Resolution"）
   - **为何可行**：drama-workshop-skills 目前假设在特定路径下运行，移植到其他机器可能失败
   - **第一步**：在 `short-drama/SKILL.md` 顶部加"安装路径侦测"说明，列出 3 种安装方式；预计 10 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 lackeyjb/playwright-skill 的核心目的**：让 Claude 能自主完成浏览器自动化任务，无需用户配置环境或指定路径

- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 Claude Code skill，有执行器脚本（echolib.py），model-invoked 场景 | echo-sleuth 有多 skill 多 agent 分层；playwright-skill 是极简单 skill | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 user_invocable skill | drama 是内容生产型；playwright 是工具型 | 低 |
| MarkQWu/claude-for-legal | 无 | — | 法律文书 vs 浏览器自动化，毫无关联 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| marketplace.json metadata.version 未同步 | `find . -name "marketplace.json" \| xargs grep "metadata"` | claude-for-legal: 各 plugin.json 均 1.0.2，无 marketplace.json，暂无此问题 | 暂无 |
| unpinned semver `^` | `find . -name "package.json" ! -path "*/node_modules/*" \| xargs grep "\\^"` | echo-sleuth 无 package.json（纯 stdlib Python），drama 无 package.json，暂无 | 暂无 |
| SKILL.md 无 example 块 | `grep -L "<example>" */SKILL.md` | **echo-sleuth 命中全部 4 个 SKILL.md（0 example blocks）** | 高 |

**命中后的具体行动建议**：
- `echo-sleuth-for-claude/skills/memory-management/SKILL.md` → 在最后加 `<example>` 块：`User: /audit\nAssistant: [扫描所有 project 的 memory.md，输出 stale 列表]` → 5 分钟可完成
- `echo-sleuth-for-claude/skills/git-mining/SKILL.md` → 同上，示例为：`User: 这个文件被谁改过？\nAssistant: [调用 git log -- path/to/file 并格式化输出]` → 5 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：dev server 自动侦测
   - **本案例做法**：`helpers.js` 中 `detectDevServers()` 扫描常用端口（3000, 8080 等），一旦发现即自动使用，无需用户输入 URL
   - **我的项目现状**：echo-sleuth 的 `list-sessions.sh` 需要用户手动指定 `~/.claude/projects/` 路径，没有自动发现
   - **如何借鉴**：在 `detect-paths.sh` 里用 `ls ~/.claude/projects/` 自动枚举 project 目录，不再要求用户传入路径参数

2. **领域**：路径发现文档在 SKILL.md 头部明确展示
   - **本案例做法**：SKILL.md 顶部用"Path Resolution"章节列出 3 种安装路径，要求 Claude 在运行时推导
   - **我的项目现状**：echo-sleuth SKILL.md 没有路径发现章节，假设路径固定
   - **如何借鉴**：在每个 SKILL.md 顶部加一个"路径发现"章节，2-3 行，列出插件市场路径、手动全局路径

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：多 skill 分层 + agent 编排
  - **我的做法**：echo-sleuth 有 commands → agents → skills 三层，各层职责清晰
  - **本案例做法**：只有 1 个 SKILL.md，没有 agent 层，复杂任务编排能力为零
  - **意义**：lackeyjb/playwright-skill 是极简单场景的极简方案；echo-sleuth 的分层设计在功能扩展时更有优势

---

## 八、术语表

### <a name="单职责-skill"></a>单职责 skill
> 只做一件事的 skill——只有 1 个 SKILL.md，不拆 agent，不拆子命令。好处是维护成本极低，坏处是功能扩展时所有逻辑都挤在一个文件里。

### <a name="语义化版本范围"></a>语义化版本范围
> `^1.57.0` 这个写法叫"caret range"——它允许 npm 自动升级到任意 `1.x.x` 版本（只要不跨 major）。好处是自动拿到安全补丁，坏处是 minor 版本的 breaking change 会悄悄破坏你的代码，而且两台机器上的 `npm install` 结果可能不同。固定版本（去掉 `^`）+ 提交 lockfile 才能保证可复现。

### <a name="manifest"></a>manifest
> 项目的"清单文件"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出插件名称、版本、作者、描述。`marketplace.json` 是插件市场专用的更丰富的 manifest。如果两个文件的版本号不同步，注册表会看到错误信息。
