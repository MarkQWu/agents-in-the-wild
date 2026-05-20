# AgriciDaniel/claude-ads — 学习案例

**仓库**：https://github.com/AgriciDaniel/claude-ads
**Stars**：2377 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-20（基于当前 HEAD）
**主题标签**：`cross-reference`, `template-design`, `examples-driven`, `manifest-discipline`, `vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Claude Code 付费广告审计与优化技能，对标「Tier 4」复杂技能（Claude Code 技能生态中最高复杂度等级）。覆盖 Google、Meta、YouTube、LinkedIn、TikTok、Microsoft、Apple、Amazon 8 大平台，包含 250+ 加权审计检查项、6 个审计代理 + 4 个创意代理、22 个专项子技能，以及 Python 脚本执行层。版本 1.7.0，作者 AgriciDaniel，遵循「Agent Skills 开放标准」的 3 层架构。

关键事实：
1. NLPM 评分 99/100——生态中最高分段之一，是高质量技能的标杆
2. 采用「[3 层架构](#three-layer)」：指令层（ads/SKILL.md 路由表）→ 编排层（agents/）→ 执行层（skills/ 子技能）
3. 版本 1.7.0 相比 audit 时新增了 ads-attribution、ads-server-side-tracking、ads-amazon 三个子技能
4. Security 状态：REVIEW（无 Critical/High，有 3 个 Medium：curl-pipe-sh 文档注释、pip break-system-packages 回退、外部 API 调用）
5. 完整的工程配套：`requirements.txt`、`CHANGELOG.md`、`CODE_OF_CONDUCT.md`、`CONTRIBUTING.md`、`CITATION.cff`

### 1.2 架构剖析
- **目录结构**：
```
claude-ads/
  ads/
    SKILL.md              # 主入口：路由表 + 全局规则
    references/           # 按需加载知识文件（25 个）
  skills/                 # 22 个专项子技能
    ads-google/SKILL.md   # Google Ads 深度分析
    ads-meta/SKILL.md     # Meta/Facebook Ads
    ads-amazon/SKILL.md   # Amazon Ads（新增）
    ads-attribution/SKILL.md # 跨平台归因（新增）
    ads-server-side-tracking/SKILL.md # 服务端追踪（新增）
    ... (共 22 个)
  agents/                 # 10 个代理
    audit-google.md       # 80 检查项 Google 审计代理
    audit-meta.md         # 50 检查项 Meta 审计代理
    creative-strategist.md # 创意策划（100/100）
    visual-designer.md    # 图像生成代理（100/100）
    copy-writer.md        # 文案写作（1 个示例，待补）
    format-adapter.md     # 尺寸校验（1 个示例，待补）
    ...
  scripts/                # Python 执行脚本（analyze/capture/fetch）
  .claude-plugin/
    plugin.json           # manifest（同步完整）
  install.sh / install.ps1 # 跨平台安装脚本
```
- **文件类型分布**：22 个 skill，10 个 agent，1 个 [manifest](#manifest)，6 个 Python 脚本，0 个 command，0 个 hook
- **编排关系**：用户触发 → `ads/SKILL.md` 路由表识别请求类型 → 路由到对应子技能（如 ads-google）→ 子技能按需加载 references/ 里的知识文件；复杂审计任务 → 启动 audit-google 等代理并行执行。是「[Router + Channels 分层](#router-channels)」模式
- **跨件契约**：`ads/SKILL.md` 里的路由表明确列出每个子技能的触发关键词；`agents/audit-google.md` 通过 `ads/references/google-audit.md` 的路径引用加载检查清单；所有引用路径通过 `~/.claude/skills/ads/references/*.md` 解析

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「协议化 frontmatter + 路由表入口」——所有子技能通过唯一路由入口统一接入，用户只需记住一个顶层命令，细节路由对用户透明
- **解决什么问题**：广告行业的知识碎片化极严重（每个平台有独立的机器学习系统、竞价逻辑、创意规范），把这些拆分成 22 个专项子技能，让 Claude 在有限上下文窗口里获得精准的平台特有知识
- **做了什么 trade-off**：22 个子技能 + 25 个 references 文件 + 10 个代理，维护成本远高于简单插件，但覆盖面和精度无法用单一大文件实现；路由表依赖关键词触发，极端情况下可能误路由
- **反映什么认知模型**：作者把 Claude Code 视为「专业咨询助理」，技能文件是「结构化知识库」——不是让 Claude 凭训练数据泛化，而是把领域专家知识显式编码到 references 文件里，逼迫模型按文件行事

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「Router + Channels 分层」（Router-Channels Layered Architecture）**

顶层路由技能接收所有请求 → 路由到专项子技能频道 → 子技能按需加载知识文件。

模式特征清单（4 条）：
- 特征 1：单一入口（`ads/SKILL.md`）暴露给用户，内部分发对用户透明
- 特征 2：子技能（channels）高度专项化，每个只处理一个平台/功能
- 特征 3：知识文件（references/）按需加载而非预加载，节省上下文 token
- 特征 4：代理（agents/）独立于技能，只在需要并行审计时调用

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多平台/多领域的专业工具（如多平台广告、多语言代码审查） | ✅ 高度适用 | 路由分发是核心价值：用户不需要知道后面有 22 个子技能 |
| 单一功能的简单工具 | ❌ 过度设计 | 一个 SKILL.md 够用，不需要路由层 |
| 知识库深度重要但覆盖面不需要很广的场景 | ✅ 适用（简化版） | 可以只用 1-3 个子技能 + 1 个路由入口 |
| 需要频繁跨子技能协作的任务 | ⚠️ 谨慎 | 路由是单向的，子技能之间无法互相调用 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| Router + Channels（本仓库） | claude-ads | 用户体验简洁，专项知识深 | 维护成本高，路由关键词需精心设计 |
| 单仓多 agent 平铺 | 0xfurai/claude-code-subagents | 零配置，新增成本极低 | 用户必须知道代理名称才能调用 |
| 大 Monolith skill | 许多简单技能 | 维护简单，无路由逻辑 | 知识深度受上下文窗口限制 |

### 2.4 改进空间
1. **当前问题**：copy-writer 和 format-adapter 各只有 1 个 `<example>` 块。**改进做法**：增加第二个 `<example>` 块，展示不同广告类型（如 B2B vs. B2C）的对比示例。**预期收益**：NLPM 评分从 95 提到 100，更重要的是用户能更快校准预期。
2. **当前问题**：install.sh 第 287 行 echo 语句推荐用 `curl | bash` 安装 banana-claude，这是不安全的安装模式文档。**改进做法**：改为推荐 `claude plugin install banana-claude@AgriciDaniel --scope user`。**预期收益**：消除 Medium 安全发现，不再向用户传播不安全的安装模式。
3. **当前问题**：install.sh 有 `--break-system-packages` 静默回退，失败时不告知用户。**改进做法**：移除静默回退，改为打印错误信息并提示用户使用虚拟环境。**预期收益**：避免静默破坏系统 Python 包管理。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **99/100**（32 个文件）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| agents/audit-google.md | 93 | "74-check" 和 "80 Checks" 数字不一致 |
| agents/audit-meta.md | 93 | "46-check" 和 "50 Checks" 数字不一致 |
| agents/copy-writer.md | 95 | 只有 1 个示例 |
| agents/format-adapter.md | 95 | 只有 1 个示例 |
| CLAUDE.md | 95 | 架构树遗漏 4 个创意代理 |
| 所有 skills/ | 98-100 | 模糊量词（applicable, relevant）数量极少 |

### 3.2 当时值得借鉴的模式
1. **按需加载 references/**（cross-reference）→ 为什么好：25 个知识文件不全部加载到上下文，只在对应代理/技能被调用时才按路径引入，精确控制 token 消耗 → 原文路径：`ads/SKILL.md` 路由表中的 `load ... when` 指令 → 如何借鉴：将大体量知识文件拆成细粒度 references，在技能 frontmatter 里声明「何时加载哪个文件」，而非一股脑全加载
2. **100/100 的 creative-strategist 和 visual-designer**（examples-driven）→ 为什么好：这两个文件有完整的 input→output 示例，明确的工具声明，无模糊量词 → 原文路径：`agents/creative-strategist.md`、`agents/visual-designer.md` → 如何借鉴：把这两个文件作为「完美代理 frontmatter」的参考模板
3. **plugin.json 完整注册所有 22 个技能**（manifest-discipline）→ 为什么好：没有任何技能文件在磁盘上但不在 manifest 里，文件即注册，注册即存在 → 原文路径：`.claude-plugin/plugin.json` → 如何借鉴：每次新增技能/命令文件必须立即更新 manifest，不留孤儿文件

### 3.3 当时的缺陷
1. **check count 数字不一致**：audit-google.md 工作流第 1 步写「74-check 审计清单」，但类别表头写「80 Checks」。为什么会失败：维护者在扩充检查项时只更新了表格，忘记同步更新工作流步骤的数字描述，导致阅读者产生混淆，不确定「实际应该执行多少项检查」。**自查**：我的代理文件里有没有数字交叉引用？如果有，有没有保持同步？
2. **CLAUDE.md 架构树遗漏创意代理**：文件开头写「10 agents (6 audit + 4 creative)」，但架构树里只列了 6 个审计代理，4 个创意代理被遗漏。为什么会失败：新增代理时更新了数字（6+4=10）但没更新树状图。**自查**：我的 CLAUDE.md 或 README 里的架构图和实际文件是否同步？
3. **install.ps1 / uninstall 脚本 CLAUDE.md 里有引用但部分不存在**：CLAUDE.md 引用了跨平台安装脚本，但 audit 时部分不存在。为什么会失败：文档先于实现，或实现时遗漏了对应脚本的创建。**自查**：我的文档里引用的文件是否都实际存在？

### 3.4 当时的优化机会
1. audit-google.md 和 audit-meta.md：在工作流步骤的数字描述旁加注释「参见类别表」，或直接更新数字与表头一致
2. copy-writer.md 和 format-adapter.md：各增加一个示例 block，NLPM 从 95 升到 100
3. CLAUDE.md 架构树：把 4 个创意代理加进去，和「10 agents (6 audit + 4 creative)」的声明一致

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| audit-google "74-check" vs "80 Checks" 不一致 | `grep -n "74-check\|check audit" agents/audit-google.md` | **已修复**：工作流文本和类别表头现在都说 "80-check" / "80 Checks" | 数字文档一致性问题已解决 |
| CLAUDE.md 遗漏 4 个创意代理 | `grep "creative-strategist\|copy-writer" CLAUDE.md` | **已修复**：4 个创意代理（creative-strategist、visual-designer、copy-writer、format-adapter）均出现在架构树 | 架构文档现已准确 |
| copy-writer 只有 1 个示例 | `grep -c "<example>" agents/copy-writer.md` | **持续存在**：仍只有 1 个 `<example>` | 等待第二个示例的 PR |

### 4.2 架构演进
从 audit 时 22 个子技能增长到现在新增了 ads-attribution（跨平台归因）、ads-server-side-tracking（服务端追踪）、ads-amazon（Amazon Ads）3 个子技能，说明作者沿着「新兴重要广告平台/技术」的方向持续扩展。同时 install.ps1 和 uninstall 脚本也都补齐了（audit 时提到可能缺失），CLAUDE.md 的架构描述和实际文件现已同步。

**作者后来意识到的**：数字一致性（74-check vs. 80 Checks）是一个可维护性陷阱，已主动修复；架构文档和实现的同步是持续维护的一部分，而不是一次性任务。

### 4.3 新增的可学习模式
**结构化评估框架**：新增的 ads-amazon 子技能里引入了 ACOS/TACOS 比率的明确计算公式，不再用「优化广告花费效率」这种模糊表述，而是用可计算的 KPI 指标（`ACOS = 广告花费 / 广告带来销售额 × 100%`）。这是一个将抽象目标转化为可验证指标的具体示范。

---

## 五、校准

### 5.1 我已经在做对的
1. **manifest 同步**：我每次新增文件都会立即更新 plugin.json，没有孤儿文件
2. **使用 references/ 按需加载**：我的大型技能也用类似的 references 分离，不把所有知识塞进一个文件
3. **避免 install.sh 的 `--break-system-packages` 静默回退**：我的安装脚本对失败有明确的错误输出
4. **交叉引用数字保持同步**：我知道文件里如果提到数字（「N 个检查项」「M 个代理」），要和实际文件数保持一致

### 5.2 挑战 / 验证
- **这次案例验证了我之前正确的做法**：「用可计算 KPI 替换模糊目标」。本仓库最高分文件（creative-strategist 和 visual-designer 100/100，最低分 audit-* 93/100）的主要差异正是在于：是否有明确可验证的标准。我之前在设计 Quality Checklist 时就倾向于用具体指标，这次得到了高质量案例的正向确认。
- **认知挑战**：99/100 的评分让我想「这已经是极限了」，但深入看发现：check count 数字不一致、copy-writer 只有 1 个示例、install.sh 推荐 curl-bash 安装——这说明高评分和「完全正确」之间仍有距离，100 分是真实可及的目标，不是理论上限。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查自己技能文件里的数字交叉引用是否一致
# 比如检查 CLAUDE.md 里的代理数和实际代理文件数
DECLARED=$(grep -oE '[0-9]+ agent' CLAUDE.md | head -1 | grep -oE '[0-9]+')
ACTUAL=$(ls agents/*.md 2>/dev/null | wc -l | tr -d ' ')
echo "Declared: $DECLARED, Actual: $ACTUAL"
# 命中（数字不等）后：更新 CLAUDE.md 里的数字

# 检查示例数量
for f in agents/*.md; do
  count=$(grep -c "<example>" "$f" 2>/dev/null || echo 0)
  [ "$count" -lt 2 ] && echo "NEEDS MORE EXAMPLES ($count/2): $f"
done
# 命中后：给 example 数 <2 的代理文件补第二个示例

# 检查架构文档和实际文件的一致性
diff <(grep -oE 'skills/[a-z-]+/SKILL\.md' CLAUDE.md | sort) \
     <(ls skills/*/SKILL.md | sort)
# 命中后：更新 CLAUDE.md 里缺失或多余的路径引用
```

### 6.2 灵感 → 实施路径
1. **想法**：仿照 ads-amazon 的「可计算 KPI 指标」模式，给自己的审计类技能文件替换所有「优化 X」→「将 X 降至 Y 以下 / 提升 X 到 Z 以上」
   - **为何可行**：本仓库 93/100 的审计代理和 100/100 的创意代理最大差别就是「可测量性」；可测量的标准让 Claude 在审计时输出可比较的结果
   - **第一步**：在每个 Quality Checklist 条目旁加括号注明具体指标，例如「(ACOS < 25%)」「(加载时间 < 2s)」，约 30 分钟
2. **想法**：实现一个 references/ 目录的「按需加载追踪」——在技能 frontmatter 里显式声明每个场景应加载哪个 reference 文件
   - **为何可行**：本仓库 ads/SKILL.md 的路由表就是这个机制的成熟实现；我的技能目前把所有知识放在单文件里，随着规模增长会超出上下文
   - **第一步**：把现有技能文件里的「知识段落」拆出，创建 `references/` 目录，在 SKILL.md 路由部分写 `load references/xxx.md for Y requests`；预计 1-2 小时

---

## 七、术语表

### <a name="three-layer"></a>3 层架构（指令层 / 编排层 / 执行层）
> Agent Skills 开放标准定义的技能层级：**指令层**（Directive）= 顶层 SKILL.md，接收用户请求并路由；**编排层**（Orchestration）= agents/，负责协调多个执行单元并行工作；**执行层**（Execution）= skills/ 子技能，每个只处理一个具体平台或功能。类比：指令层是前台接待，编排层是项目经理，执行层是各专业工程师。

### <a name="router-channels"></a>Router + Channels 分层
> 插件架构模式：单一入口（router）接收所有请求，根据关键词或意图将请求分发到对应的专项频道（channels）。用户只和 router 交互，具体执行逻辑在各 channel 里。本仓库的 `ads/SKILL.md` 是 router，22 个 `skills/ads-*/SKILL.md` 是 channels。

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被加载。

### <a name="references"></a>references（按需加载知识文件）
> 存放在 `references/` 目录下的 Markdown 文件，包含详细的领域知识（如 Google Ads 的 80 条检查项列表）。这些文件不是代理/技能的主体，而是「字典」——只在对应场景下才被引入上下文，节省 token 消耗。对比：把所有知识塞进一个大技能文件 = 把整部词典随时带着读；用 references 按需加载 = 需要查某个词时才翻开词典。
