# kazukinagata/shinkoku — 学习案例

**仓库**：https://github.com/kazukinagata/shinkoku
**Stars**：335（registry，exemplar_published=true）| **来源**：upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-19（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `cross-reference`, `manifest-discipline`, `security-gate`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

日本税务申报自动化 Claude Code 插件，作者 kazukinagata，当前版本 v0.6.5。针对会社員 + 副業（事業所得・青色申告）场景，将确定申告（日本年度纳税申报）的全流程——从帐簿管理、税额计算，到在申告书等作成コーナー（日本国税局官网系统）输入——全部自动化。技术栈：Claude Skills（NL 层）+ Python 3.11 CLI（计算层）+ SQLite WAL 数据库（持久化层）+ browser automation（e-Tax 表单填写层）。

### 1.2 架构剖析

- **目录结构**：

```
shinkoku/
├── skills/                    # 25 个 SKILL.md（NL 对话层）
│   ├── setup/                 # 环境初始化（入口）
│   ├── capabilities/          # 能力查询（路由起点）
│   ├── gather/                # 资料收集协调
│   ├── reading-receipt/       # OCR 票据读取
│   ├── reading-invoice/       # OCR 发票读取
│   ├── reading-withholding/   # OCR 源泉徴収票读取
│   ├── reading-deduction-cert/# OCR 控除证明读取
│   ├── reading-payment-statement/ # OCR 支払调书读取
│   ├── income-tax/            # 所得税计算
│   ├── consumption-tax/       # 消费税计算
│   ├── settlement/            # 结算
│   ├── journal/               # 帐簿管理
│   ├── e-bookkeeping-compliance/ # 电子帐簿保存法合规
│   ├── invoice-system/        # 发票系统
│   ├── furusato/              # 故乡纳税
│   ├── tax-housing-loan-context/ # 住宅贷款控除上下文
│   ├── tax-legal-context/     # 税法上下文
│   ├── tax-ebookkeeping-context/ # 电子帐簿上下文
│   ├── tax-invoice-credit-context/ # 发票信用上下文
│   ├── tax-advisor/           # 税务顾问综合
│   ├── assess/                # 税额评估
│   ├── submit/                # 申告提交协调
│   ├── incorporation/         # 法人化
│   └── e-tax/                 # e-Tax 系统操作
├── src/shinkoku/               # Python CLI（计算层）
│   ├── cli/                   # 命令行入口
│   ├── tools/                 # 业务逻辑（纯函数）
│   └── (models, db, masters)  # 数据层
├── pyproject.toml             # Python 依赖管理
├── uv.lock                    # 依赖锁文件
└── .claude-plugin/plugin.json # 插件清单
```

- **文件类型分布**：25 个 SKILL.md / 0 个 agent / 0 个 command / 0 个 hook（技能之间互相引用形成编排，无 hook 层）
- **编排关系**：`capabilities` 技能是路由起点，用户通过它了解系统边界。实际工作流按「gather → reading-* → journal → income-tax/consumption-tax → settlement → submit → e-tax」线性推进，每个技能通过「次のステップ（下一步）」引导用户进入下一环节。`reading-*` 系列（OCR 读取技能）作为通用工具被其他技能的「下一步」引用。
- **跨件契约**：所有 SKILL.md 在 References 节统一引用 `references/` 目录下的 Markdown 文档（如税法、常见错误、操作模式），唯一例外是 `tax-advisor/` 使用 `reference/`（单数）而非 `references/`（复数），属于历史命名不一致，两个目录都实际存在。Python CLI 通过 `shinkoku <command> <subcommand>` 调用，输入/输出均为 JSON，与 SKILL.md 的对话层解耦。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「Claude 做对话协调，Python 做数值计算」—— NL 技能负责引导用户收集资料、解释结果，不做任何税额计算；所有数值由 Python CLI 独立计算，保证可验证性。
- **解决什么问题**：日本确定申告流程复杂（多种收入类型 × 多种控除 × 政府在线系统的 OS 限制），且高度个人化，传统税务软件难以覆盖副业 + 会社員的混合场景。
- **做了什么 trade-off**：NL 层 + Python 二进制层 vs 纯 NL。选择了混合架构：税额计算涉及法律合规，不能由 LLM 估算；但表单填写、资料收集的对话流程适合 NL 驱动。
- **反映什么认知模型**：作者把 Claude 看作「流程协调者」而非「计算引擎」——Claude 负责用自然语言引导用户提供正确输入、解释计算结果、操作浏览器；Python 负责保证计算结果的正确性。这是对 LLM 局限性的诚实认知。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「NL 表皮 + [原生二进制核心](#原生二进制核心)」架构**

核心特征：
- **特征 1**：技能层纯对话，不做任何数值计算；计算全部在 Python CLI 中
- **特征 2**：NL 技能通过自然语言告知用户「请运行 `shinkoku income-tax calculate`」，而不是自己计算
- **特征 3**：OCR 读取技能群（reading-* × 5）形成一个「文档解析子系统」，被上层技能复用
- **特征 4**：上下文文档技能（tax-*-context × 4）作为法律知识的「引用锚点」，被计算技能引用而不被用户直接调用
- **特征 5**：SQLite WAL 模式提供事务级安全，保证帐簿数据不因会话中断而损坏

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 有精确计算要求的合规场景（税务、会计、法律） | ✅ 高度适用 | NL 负责引导，Python 保证计算正确性，两层各司其职 |
| 流程线性且步骤固定的专家任务 | ✅ 适用 | 技能链的线性推进配合「下一步」引导，流畅度高 |
| 需要 OCR 批量处理文档的场景 | ✅ 适用 | reading-* 子系统可独立使用，也可被其他技能引用 |
| 需要实时网络查询的场景 | ❌ 不适用 | 技能内嵌的知识是静态的，不自动更新税法规定 |
| 多用户共享数据的企业场景 | ❌ 不适用 | SQLite 本地数据库，非多用户架构 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL + 原生二进制（本仓库） | kazukinagata/shinkoku | 计算可验证、合规性强、离线可用 | 用户需要安装 Python 环境，上手成本高 |
| 纯 NL 技能（无计算层） | addyosmani/agent-skills | 零安装、开箱即用 | 无法保证数值计算准确性，不适合合规场景 |
| NL + MCP 工具调用 | czlonkowski/n8n-skills | 数据实时，无需本地安装工具 | 依赖 MCP 服务器在线，有网络单点故障风险 |

### 2.4 改进空间

1. **当前问题**：`skills/setup/SKILL.md` 中的安装命令 `uv tool install git+https://github.com/kazukinagata/shinkoku` 没有版本锁定 **改进做法**：添加版本 tag：`uv tool install git+https://github.com/kazukinagata/shinkoku@v0.6.5`，并在每次发布时更新 **预期收益**：消除供应链攻击面，用户安装的版本与测试过的版本一致
2. **当前问题**：`skills/e-tax/SKILL.md` 超过 25,000 token，audit 无法完整读取 **改进做法**：按表单类型拆分（e-tax-assessment/、e-tax-submit/ 等子技能），保持每个 SKILL.md 在 8,000 token 以内 **预期收益**：提高可审计性；NLPM 等工具可以完整评分
3. **当前问题**：`tax-advisor/reference/` vs 其他技能的 `references/`（复数）命名不一致 **改进做法**：统一重命名为 `references/`，更新 SKILL.md 中的路径引用 **预期收益**：贡献者不再需要记住例外

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 94/100，SECURITY CLEAR（但有 medium × 1、low × 2）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| CLAUDE.md | 85/100 | 无 NL frontmatter（设计如此，不是 skill 文件） |
| skills/e-tax/SKILL.md | 88/100 | 文件超过 25k token，无法完整审查 |
| skills/income-tax/SKILL.md | 92/100 | 873 行，最长技能；复杂度成本 |
| skills/settlement/SKILL.md | 96/100 | 「妥当か」模糊量词 ×2 |
| skills/furusato/SKILL.md | 97/100 | 「適切に設定されているか」×1 |
| skills/tax-advisor/SKILL.md | 96/100 | 「網羅的に」×1；`reference/` vs `references/` 不一致 |
| 其余 19 个技能 | 95-100/100 | 无重大问题 |

**安全发现（medium/low）**：
- `skills/setup/SKILL.md`：git URL 安装无版本锁定（medium）
- `pyproject.toml`：依赖使用 `>=` floor 约束而非精确版本锁定（low）
- `skills/e-tax/scripts/etax-stealth.js`：浏览器 user-agent 伪装（low，但有合理理由：NTA e-Tax 网站拒绝 Linux）

### 3.2 当时值得借鉴的模式

1. **所有内部引用零死链接** → 25 个 SKILL.md 中引用的每一个辅助文档（`DATA_ACCESS.md`, `COMMON_PATTERNS.md` 等）均在仓库中实际存在。根本原因：维护者在每次重构后都会更新引用，未用 CI 自动化验证但执行到位。借鉴：我的技能中有没有引用实际不存在的文件？→ 用 `grep -rE '\[.*\]\(.*\.md\)' skills/*/SKILL.md | xargs -I{} test -f {}` 检查。

2. **语境文档技能群（tax-*-context × 4）** → 将法律条文、政府指南等静态上下文抽取到独立技能，其他技能通过引用加载而不是直接复制。根本原因：法律文本会更新，集中维护避免多点更新负担。借鉴：bureau 的技能可以建立一个「上下文技能」层，将用户档案（写作风格、项目背景）抽出为独立可引用的上下文技能。

3. **能力边界声明（capabilities 技能）** → 独立的 `capabilities/SKILL.md` 列出系统能做什么、不能做什么（Out 对应的场景）。根本原因：明确能力边界是防止用户误用的第一道防线，也是可信度的来源。借鉴：bureau 可以有一个 `limits/SKILL.md` 明确声明「哪些信息不会被 bureau 捕获/推理」，减少用户的错误期望。

4. **版本同步校验** → `plugin.json` v0.6.5 与 `pyproject.toml` v0.6.5 一致，且两者在 CI 中会被验证。根本原因：版本是隐式合同，两个独立文件必须有机制保持同步。借鉴：bureau 的 plugin.json 和代码版本是否同步？→ 检查。

5. **etax-stealth.js 的合理伪装** → 对 NTA 网站的 OS 检测进行 user-agent 欺骗，但在文件头加了注释说明理由（NTA 歧视 Linux 用户）。根本原因：技术上的「欺骗」行为在有合法理由时需要在代码中记录，防止被误解为恶意代码。借鉴：我的代码中是否有类似「看起来奇怪但有合理理由」的做法需要在注释中记录？

### 3.3 当时的缺陷

1. **`setup/SKILL.md` git URL 无版本锁定** → `uv tool install git+https://github.com/kazukinagata/shinkoku` 没有 `@version` 后缀。为什么会失败：用户安装的版本可能是最新提交而非经过测试的发布版，导致意外的兼容性问题；更严重的是，若仓库被攻陷，用户会安装被篡改的代码。自查：我的 bureau/setup 相关文档中有没有类似的无版本锁定安装命令？

2. **`e-tax/SKILL.md` 超过 25k token** → 单个文件过大，无法被工具完整审查，也影响 Claude 的 context 利用效率。为什么会失败：审查盲区 → 质量问题无法被发现；context 占用高 → 同时加载多技能时性能下降。自查：我的 bureau 技能中最长的文件有多少行？

3. **模糊量词（日语）** → 「妥当か」（是否妥当）、「適切に設定されているか」（是否适当设置）等表达模糊。为什么会失败：Claude 不知道什么是「妥当」，无法生成可验证的检查结果。自查：我的技能中有没有「合适的」「适当的」等模糊判断标准？

### 3.4 当时的优化机会

1. `setup/SKILL.md` 中的安装命令加版本 tag：`@v0.6.5`
2. `e-tax/SKILL.md` 按表单类型拆分为多个子技能
3. 「妥当か」替换为「残高がマイナスでないか」等具体可验证标准

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `setup/SKILL.md` git URL 无版本锁定 | `grep "uv tool install git+https" skills/setup/SKILL.md` | **持续**：line 21 仍为 `uv tool install git+https://github.com/kazukinagata/shinkoku`，无 @tag | 3 个月未修，说明作者认为这是低优先级风险 |
| `tax-advisor/reference/` 命名不一致 | `ls skills/tax-advisor/` | 暂无数据（工具限制） | 暂无确认 |
| `e-tax/SKILL.md` 超大文件 | `wc -l skills/e-tax/SKILL.md` | 暂无数据（工具限制） | 暂无确认 |

### 4.2 架构演进

当前 HEAD 的技能数量与 audit 时相同（25 个），版本维持 v0.6.5，说明仓库在 audit 后处于稳定维护而非快速扩张阶段。新增了：
- `docs/wsl-os-detection-workaround.md`：记录 WSL 用户使用 e-Tax 网站的变通方案，说明 etax-stealth.js 的 Linux 兼容性问题引发了用户需求
- `.github/pull_request_template.md`：说明开始接受外部贡献
- `SECURITY.md`：显式的安全策略文档，与 audit 中 security 发现的响应一致

### 4.3 新增的可学习模式

**`SECURITY.md` 的出现**：在 audit 记录了安全发现后，仓库新增了 `SECURITY.md`，说明作者开始正式化安全策略。对于任何有 CLI 安装、网络调用或系统权限操作的插件，显式的安全策略文档是专业度的信号，也是接受外部贡献的前提。

---

## 五、校准

### 5.1 我已经在做对的

- **NL 与计算分离**：bureau 和 gstack 的技能都不做数值计算，只做流程协调和内容引导
- **内部链接意识**：我的技能文件中相互引用的文档我在写作时会验证存在性
- **版本同步意识**：bureau 的 plugin.json 版本号我会有意识地与内容变更同步

### 5.2 挑战 / 验证

**这次案例验证了「NL + 原生二进制」混合架构的适用边界**：计算必须正确的场景（税务、金融、法律合规）不能纯靠 LLM 估算，必须有独立的计算层。这个判断适用于我的 graphify 仓库——知识图谱的图数据库查询必须由 Python/Rust 来执行，而不是让 Claude 「猜测」图中有哪些节点。

另一个被验证的认知：**能力边界声明**（capabilities 技能）是减少用户期望错误的低成本高价值投入。我的 bureau 缺少明确的「我不做什么」声明——当用户期望 bureau 能做到的事情实际不在设计范围内时，没有明确的拒绝锚点。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能文件中最大的文件（是否超过 500 行）
find /tmp/my-repos/MarkQWu-bureau/skills \
     /tmp/my-repos/MarkQWu-gstack \
     -name "SKILL.md" -exec wc -l {} \; 2>/dev/null | sort -rn | head -10

# 检查安装指南中是否有无版本的 git clone / install 命令
grep -rn "git clone\|npm install\|pip install\|uv tool install" \
  /tmp/my-repos/MarkQWu-bureau/ \
  /tmp/my-repos/MarkQWu-gstack/ 2>/dev/null | grep -v ".git" | head -10

# 检查日语/中文模糊量词
grep -rn "合理\|适当\|合适\|恰当\|适时" \
  /tmp/my-repos/MarkQWu-bureau/skills/ 2>/dev/null | head -10
```

命中后怎么办：
- SKILL.md 超过 500 行 → 考虑按子功能拆分，每个子技能聚焦一个操作流程
- 无版本安装命令 → 锁定到具体版本号（`@v1.2.3`）
- 模糊量词 → 替换为可验证的具体描述（如「X 字以内」「Y 秒以内」）

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 添加 `capabilities/SKILL.md`，明确声明 bureau 能做什么、不能做什么
   - **为何可行**：bureau 已有 7 个技能，用户经常问「bureau 能帮我做 X 吗」，一个明确的能力声明减少这类对话
   - **第一步**：在 `bureau/skills/capabilities/` 创建 `SKILL.md`，列出 bureau 支持的场景（logbook、知识编译、查询）和不支持的场景（实时数据、外部 API 调用）；约 20 分钟

2. **想法**：给 graphify 或 shiji-kb 的技能文件拆分超大 SKILL.md
   - **为何可行**：shiji-kb 有多个 procedural skill 文件，某些可能已超过合理大小
   - **第一步**：`find /tmp/my-repos/MarkQWu-shiji-kb -name "SKILL.md" -exec wc -l {} \;` 找到最大的，评估是否按子任务拆分；约 30 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 kazukinagata/shinkoku 的核心目的**：NL + Python 二进制混合架构，将日本税务申报的复杂合规流程自动化，Claude 负责协调对话，Python 负责保证计算正确性

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 高 | NL 技能 + Python 核心二进制的混合架构；有复杂的领域知识需要 NL 层引导 | graphify 的计算层是图数据库查询，shinkoku 是税务计算；graphify 面向开发者，shinkoku 面向最终用户 | 高 |
| MarkQWu/bureau | 中 | 都是 Claude Code 插件；都有跨会话状态管理需求 | bureau 无 Python 二进制层，纯 NL | 中 |
| MarkQWu/shiji-kb | 低 | 都有 SKILL.md | shiji-kb 是知识库，shinkoku 是流程自动化 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 安装命令无版本锁定 | `grep -rn "git clone.*github" */skills/` | 需实地检查 | 中 |
| SKILL.md 超大文件（>500 行） | `find . -name "SKILL.md" -exec wc -l {} \; \| sort -rn` | shiji-kb 的 procedural skills 可能有超大文件 | 低 |
| 模糊量词（合适/适当） | `grep -rn "合理\|适当" bureau/skills/` | 需实地检查 | 低 |

**命中后的具体行动建议**：
- 如果 graphify 技能中有无版本锁定的安装命令 → 锁定到 GitHub Release 的具体 tag → 10 分钟

### 8.3 别人的更优方案

1. **领域**：上下文文档技能群（context skills）
   - **本案例做法**：`tax-*-context/` × 4 将法律条文抽为独立技能，其他计算技能引用而不复制
   - **我的项目现状**：bureau 每个技能各自包含背景信息，没有共享的上下文技能层
   - **如何借鉴**：在 bureau 创建 `bureau-context/SKILL.md`（记录用户的工作方式偏好和项目背景），其他技能通过 `See also: /bureau-context` 引用它，减少重复的上下文加载

2. **领域**：能力边界声明（Out / 对象外 清单）
   - **本案例做法**：CLAUDE.md 中有完整的「対象外ペルソナ / 対象外機能」表格，明确告知用户什么不在系统支持范围
   - **我的项目现状**：bureau 和 gstack 的 README 只描述能做什么，没有说明不能做什么
   - **如何借鉴**：在 bureau README 中添加「Bureau 不做什么」节：不支持实时数据、不替代人工审阅、不自动推送到外部系统

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：持久化知识管理管道（knowledge pipeline）
- **我的做法**：bureau 有 capture → compile → review → query 的完整知识管道，会话结束后知识通过人工审阅门控后持久化到知识库，支持后续查询
- **本案例做法**（弱在哪）：shinkoku 的数据层用 SQLite 存储税务数据，但没有「AI 会话内容 → 结构化知识」的提炼流程，每次使用相对独立
- **意义**：如果和 kazukinagata 交流，bureau 的知识编译模式（将 AI 会话内容归纳为持久知识）是一个有趣的补充设计；shinkoku 的申告历史记录可以通过类似机制变成可查询的税务知识库

---

## 八、术语表

### <a name="原生二进制核心"></a>原生二进制核心

> 用 Python、Go、Rust 这类编译型或有解释器的语言写出来的独立可执行程序，在本案例中指 `shinkoku` 命令行工具。「二进制」层面表示它以非 Markdown 的程序形式运行，与 NL 技能层（SKILL.md）形成「对话协调 vs 数值计算」的分工。shinkoku 通过 `shinkoku income-tax calculate` 这样的命令运行，输出 JSON，与 Claude 的 SKILL.md 层完全解耦。

### <a name="SQLite-WAL"></a>SQLite WAL

> SQLite 数据库的「Write-Ahead Logging」写入模式。普通模式下写入失败可能损坏数据库；WAL 模式先写日志再更新数据文件，中断后可以从日志恢复，相当于给数据库加了事务安全保障。对税务数据（不可重算）来说，WAL 模式是必要的选择。

### <a name="frontmatter"></a>frontmatter

> Markdown 文件顶部用 `---` 包裹的 YAML 配置，声明技能的 `name`、`description` 等元数据。shinkoku 的 CLAUDE.md 没有 frontmatter（因为 CLAUDE.md 是项目说明文件，不是技能文件），这是设计决策而非疏漏。

### <a name="e-Tax"></a>e-Tax

> 日本国税局的网络申告系统（国税電子申告・納税システム）。该系统原本只支持 Windows/Mac，并在服务器端检测用户 OS，拒绝 Linux 用户访问。`etax-stealth.js` 通过修改浏览器的 `navigator.platform` 等属性来绕过这个歧视性限制，是合理的无障碍访问实践。

### <a name="uv"></a>uv

> Astral（Rust 团队）开发的高速 Python 包管理器，替代 pip/conda。`uv tool install` 类似 `pip install` 但更快、支持隔离环境。`uv.lock` 是精确的依赖锁文件，保证在不同机器上安装完全相同的依赖版本。
