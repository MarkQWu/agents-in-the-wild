# RKiding/Awesome-finance-skills — 学习案例

**仓库**：https://github.com/RKiding/Awesome-finance-skills
**Stars**：1907 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-26（基于当前 HEAD）
**主题标签**：`single-purpose`, `vague-quantifier`, `security-gate`, `examples-driven`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
RKiding/Awesome-finance-skills 是一个面向量化投资和金融分析的 Claude Code 技能合集。核心品牌叫 **AlphaEar**，聚焦 A 股/港股/美股三市场的数据获取、预测、情绪分析和报告生成。

关键事实：
1. 10 个独立技能（`alphaear-stock`、`alphaear-predictor`、`alphaear-news`、`alphaear-sentiment`、`alphaear-reporter`、`alphaear-signal-tracker`、`alphaear-deepear-lite`、`alphaear-logic-visualizer`、`alphaear-search`）加 1 个元技能（`skill-creator`）
2. 每个技能都有 Python 后端脚本（`scripts/`），技能间存在未声明的跨技能依赖
3. `references/PROMPTS.md` 模式：多个技能在 `references/` 子目录里存放提示词参考，是该仓库独特的知识侧车设计
4. 三个技能（predictor、signal-tracker、reporter）共享相同的 Python 工具库，但通过物理复制而非导入共享
5. 有 1,907 stars，说明金融从业者或量化爱好者群体有真实需求

### 1.2 架构剖析
- **目录结构**：
```
Awesome-finance-skills/
├── skills/
│   ├── alphaear-stock/        # 股票搜索与历史数据
│   │   ├── SKILL.md
│   │   └── scripts/           # Python 后端
│   ├── alphaear-predictor/    # 市场趋势预测（ML）
│   │   ├── SKILL.md
│   │   ├── scripts/           # 含重复 evaluation.py
│   │   └── references/PROMPTS.md
│   ├── alphaear-sentiment/    # 情绪分析
│   ├── alphaear-news/         # 市场新闻聚合
│   ├── alphaear-reporter/     # 专业报告生成（含重复 evaluation.py）
│   ├── alphaear-signal-tracker/ # 信号追踪（含重复 evaluation.py）
│   ├── alphaear-logic-visualizer/ # Draw.io 图表生成
│   ├── alphaear-deepear-lite/ # 轻量级信号检测
│   ├── alphaear-search/       # 金融数据聚合搜索
│   └── skill-creator/         # 自动生成新技能的元技能
├── tests/                     # 测试套件
└── figs/                      # 说明图片
```
- **文件类型分布**：10 个 SKILL.md / ~139 个 Python 脚本 / 5 个 PROMPTS.md 参考文档 / 0 个 agent / 0 个 hook
- **编排关系**：所有技能平列，无路由层。`alphaear-signal-tracker` 依赖 `alphaear-search` 和 `alphaear-stock`，但这个依赖只在 SKILL.md body 里的文字中提及，不在 frontmatter 里声明
- **跨件契约**：`scripts/database_manager.py` 是跨技能的数据库封装；`scripts/utils/predictor/evaluation.py` 被三个技能物理复制

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「每个技能是一个独立的金融工具」——每个 SKILL.md 对应一个具体的金融分析任务，边界清晰（股票搜索、新闻、预测、报告生成各自独立）
- **解决什么问题**：金融分析需要调用多个异构数据源（AKShare、yfinance、newsnow.busiyi.world 等），通过技能把这些调用规范化，让 Claude 有一致的接口
- **Trade-off**：技能高度独立（方便单独安装/更新），但共享逻辑通过代码复制而非模块化共享，引入了维护成本
- **认知模型**：Claude 是金融分析任务的协调者，Python 脚本是工具，SKILL.md 是任务委托协议

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：域内平铺 + 知识侧车（Domain-Flat + Knowledge-Sidecar）**

所有技能平铺在 `skills/` 目录下，无层次关系。每个技能有一个 `references/PROMPTS.md` 知识侧车文件，用于存储该技能的专属提示词参考、分析框架和实例。侧车和 SKILL.md 并列，但不被 frontmatter 声明——是可选的上下文补充。

模式特征清单（4 条）：
- 特征 1：每个技能目录自闭合（SKILL.md + scripts/ + references/ 全部在一个子目录里）
- 特征 2：`references/PROMPTS.md` 侧车存储领域提示词，与技能主体分离，可以不同频率更新
- 特征 3：技能之间无 [manifest](#manifest) 级别的依赖声明，只有 SKILL.md body 里的文字描述
- 特征 4：技能共享的工具库通过物理复制（而非 Python import）分发到每个技能目录

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 领域内有 5-15 个相关但独立的工具 | ✅ 适用 | 平铺结构易于导航，每个工具可单独安装 |
| 技能间有复杂的调用链/组合 | ❌ 不适用 | 缺依赖声明，组合调用时容易失败 |
| 需要频繁迭代核心算法（如 ML 模型）| ❌ 不适用 | 物理复制导致修复需要同步到多个副本 |
| 需要清晰的安装成本 | ✅ 适用 | 每个技能的 requirements 自包含 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 域内平铺 + 知识侧车（本仓库）| RKiding/Awesome-finance-skills | 每个技能自包含、领域侧车可独立更新 | 共享逻辑复制漂移、跨技能依赖不透明 |
| 路由器 + 频道分层 | 0xmariowu/Autosearch | 统一路由层让跨频道调度有序 | 路由层变成单点故障 |
| 元技能生成器 | skill-creator（本仓库内）| 降低新建技能的脑力成本 | 生成出来的技能质量取决于模板质量 |

### 2.4 改进空间
1. **当前问题**：`evaluation.py` 在三个技能目录里各有一份，安全补丁（如 `torch.load` 的 `weights_only=True`）需要改三处。**改进做法**：把共享工具抽到根目录 `shared/utils/`，各技能通过 `sys.path.insert` 引用。**预期收益**：一处修复即全部生效
2. **当前问题**：跨技能依赖只在 body 文字里提及（「先用 alphaear-search 获取数据」），不在 frontmatter 里声明。**改进做法**：在 SKILL.md frontmatter 里加 `requires:` 字段，列出依赖的其他技能名。**预期收益**：安装检查和文档生成可以自动化

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **92/100**（10 个文件，single 策略）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/alphaear-predictor/SKILL.md | 88 | 缺输出格式；「subjectively adjust」模糊量词 |
| skills/alphaear-signal-tracker/SKILL.md | 88 | 缺输出格式；「necessary data」模糊量词 |
| skills/alphaear-deepear-lite/SKILL.md | 90 | 缺输出格式 |
| skills/alphaear-stock/SKILL.md | 100 | 无问题 |

### 3.2 当时值得借鉴的模式

1. **输出格式标准化（alphaear-stock）** → `alphaear-stock/SKILL.md` 明确指定 `search_ticker` 返回 `List[{code, name}]`，`get_stock_price` 返回包含 OHLCV 列的 DataFrame → `skills/alphaear-stock/SKILL.md` L9-30 → 借鉴：在每个 SKILL.md 里为每个核心函数声明具体返回类型和字段

2. **references/PROMPTS.md 侧车模式** → 把分析框架和提示词参考与技能指令分开存放，技能主体简洁，参考资料可独立维护 → `skills/alphaear-predictor/references/PROMPTS.md` → 借鉴：对于有复杂域知识的技能，把背景知识放到 `references/` 子目录，保持 SKILL.md 精简

3. **依赖声明（部分）** → `alphaear-stock/SKILL.md` 明确列出 `pandas`、`requests`、`akshare`、`yfinance` 作为依赖 → 借鉴：在 SKILL.md 的 `## Dependencies` 部分列出所有需要安装的 Python 包，方便用户排查环境问题

### 3.3 当时的缺陷

1. **六个技能缺输出格式说明**（alphaear-predictor、signal-tracker、reporter、news、deepear-lite、logic-visualizer）→ 根本原因：作者专注于描述「能做什么」而非「输出是什么」，混淆了能力宣传与使用契约 → 自查：echo-sleuth 的部分 skill 也有类似问题，需要核查

2. **Import 路径错误**（`from scripts.utils.kronos_predictor import KronosPredictorUtility` 但文件在 `scripts/kronos_predictor.py`）→ 根本原因：文件移动后没有同步更新文档里的示例代码 → 自查：我的 echo-sleuth SKILL.md 里有引用 scripts/ 路径的内容，需要确认路径准确

3. **代码复制引发的安全补丁问题**（`torch.load` 缺 `weights_only=True`，需要改三处）→ 根本原因：为了每个技能「自包含」而物理复制工具代码，导致安全修复需要多处同步 → 自查：我的项目无共享 Python 工具，暂无此问题

### 3.4 当时的优化机会

1. 为 6 个缺输出格式的技能添加 `## Output Format` section（最高 ROI，能把这 6 个文件从 88-90 提升至 95+）
2. 修复 `alphaear-predictor/SKILL.md` 里的 import 路径（把 `from scripts.utils.kronos_predictor` 改为 `from scripts.kronos_predictor`）
3. 把三处 `torch.load` 加上 `weights_only=True`（安全修复，防止恶意 .pt 文件执行任意代码）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| Import 路径错误（`scripts.utils.kronos_predictor`）| `grep "scripts.utils.kronos_predictor" skills/alphaear-predictor/SKILL.md` | **仍错误**（line 28 仍是 `from scripts.utils.kronos_predictor`）| 用户按文档跑示例代码必然 `ModuleNotFoundError` |
| `torch.load` 缺 `weights_only=True`（三处）| `grep -rn "torch.load" skills/` | **部分修复**：`kronos_predictor.py:79` 加了 `weights_only=True`，但三处 `evaluation.py:39` 仍未修复 | 最高风险路径（evaluation.py）仍是漏洞 |
| 重复 section heading | `grep -n "^### 1\. " skills/alphaear-predictor/SKILL.md` | **仍存在**（第 14、16 行重复 `### 1. Forecast Market Trends`）| 纯文档问题，不影响功能 |

### 4.2 架构演进
与 audit 时相比，现在目录结构基本未变（10 个技能 + tests/ + figs/），没有重大重组。`skill-creator/SKILL.md` 增加了描述，表明作者希望用这个元技能来批量生成更多金融技能。整体处于「维护而非演进」状态。

### 4.3 新增的可学习模式
暂无明显新增设计模式，HEAD 与 audit 时差异主要在内容补充而非结构变化。

---

## 五、校准

### 5.1 我已经在做对的
1. **无代码复制**：echo-sleuth 共享工具（`scripts/echolib.py`）是单一文件，被所有脚本 import，没有物理复制问题
2. **依赖列在 README** 中：echo-sleuth 有清晰的安装前提说明
3. **无 unsafe pickle**：echo-sleuth 不涉及 ML 模型加载，无此安全风险

### 5.2 挑战 / 验证
- **验证了什么做法**：「每个技能应该有明确的输出格式声明」——alphaear-stock 是该仓库最高分（100/100）的技能，其核心优势就是对每个函数的返回类型有精确描述。这验证了「输出格式是 SKILL.md 的一等公民」这个原则。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skills 是否有明确输出格式
grep -L "## Output\|## Returns\|Returns:\|return type" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md 2>/dev/null
# 命中后：为缺失的 SKILL.md 添加 ## Output Format 章节，说明 Claude 应该给用户展示什么格式的内容
```

```bash
# 检查 SKILL.md 里引用的脚本路径是否存在
grep -oE 'scripts/[a-zA-Z0-9_\-]+\.py' ~/.claude/skills/*/SKILL.md | while IFS=: read file path; do
  dir=$(dirname "$file")
  if [ ! -f "$dir/$path" ]; then echo "BROKEN PATH: $file → $path"; fi
done
# 命中后：修正路径或补充缺失脚本
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 skill 添加 `references/` 侧车（存储 JSONL 格式说明、Claude Code 事件类型参考等域知识）
   - **为何可行**：echo-sleuth 有复杂的 JSONL 解析知识，目前都堆在 SKILL.md 里，提取到 references/ 可以让主体 SKILL 更简洁
   - **第一步**：在 `skills/jsonl-core/` 创建 `references/` 目录，把现有的事件类型表格移入 `references/jsonl-format.md`，约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 RKiding/Awesome-finance-skills 的核心目的**：金融分析工具集（量化数据获取 + ML 预测 + 报告生成）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 都是「专业领域 + Claude Code 技能集合」模式；都有 Python 脚本后端；都有跨技能依赖 | 法律 vs 金融；claude-for-legal 技能更多 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 都有 Python 脚本后端 | 开发工具 vs 金融分析，领域完全不同 | 低 |
| MarkQWu/drama-workshop-skills | 无 | - | 创意内容 vs 结构化数据分析 | 无 |

若我的仓库 claude-for-legal 也有 Python 脚本驱动的技能，则「代码复制漂移」问题同样需要注意。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 缺输出格式说明 | `grep -L "## Output\|Returns:" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | 暂无完整路径可验证（克隆时无 skill 子目录）| 待查 |
| 跨技能依赖未声明 | `grep -L "requires:\|dependencies:" /tmp/my-repos/MarkQWu-claude-for-legal/*/SKILL.md 2>/dev/null` | claude-for-legal 的技能间依赖情况未知，需人工查 | 中 |

**命中后的具体行动建议**：
- 手动检查 `claude-for-legal` 是否有技能依赖另一技能，若有，在相关 SKILL.md frontmatter 加 `requires:` 字段

### 7.3 别人的更优方案

1. **领域**：`references/PROMPTS.md` 知识侧车模式
   - **本案例做法**：`skills/alphaear-predictor/references/PROMPTS.md` 存储预测分析的提示词框架，与 SKILL.md 分离
   - **我的项目现状**：echo-sleuth 的 `skills/jsonl-core/` 没有 references/ 子目录，所有知识都在 SKILL.md 主体里（导致 SKILL.md 较长）
   - **如何借鉴**：为 echo-sleuth 的 `skills/jsonl-core/` 和 `skills/experience-synthesis/` 创建 `references/` 目录，把「格式参考」和「分析框架」各自移入单独文件

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：共享工具的模块化（无物理复制）
- **我的做法**：echo-sleuth-for-claude 的 `scripts/echolib.py` 是单一共享库，所有脚本 import 它，无复制
- **本案例做法（弱在哪）**：三个技能各自有一份 `evaluation.py`，安全补丁需要改三处（audit 时发现，至今只部分修复）
- **意义**：共享工具单一化是我项目的明显优势，也是为什么 echo-sleuth 的安全修复成本低

---

## 八、术语表

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest——里面列出所有 skills、commands、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="unsafe-pickle"></a>unsafe pickle（不安全的序列化加载）
> Python 的 pickle 是一种「把 Python 对象存到文件里再读出来」的机制。问题在于：加载 pickle 文件时，文件里的代码会被直接执行。`torch.load`（PyTorch 模型加载）默认使用 pickle，如果有人替换了你的模型文件（.pt 文件），就可以让你的机器执行任意代码。`weights_only=True` 告诉 PyTorch「只读权重数组，不执行任何代码」，是安全实践。

### <a name="OHLCV"></a>OHLCV
> 股票价格数据的标准字段集合：Open（开盘价）、High（最高价）、Low（最低价）、Close（收盘价）、Volume（成交量）。金融分析的基础数据格式。

### <a name="AKShare"></a>AKShare
> 一个开源 Python 库，专门用于获取 A 股（中国大陆市场）和港股数据，数据来源主要是东方财富网等中国金融平台。
