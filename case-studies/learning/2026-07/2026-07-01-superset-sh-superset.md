# superset-sh/superset — 学习案例

**仓库**：https://github.com/superset-sh/superset
**Stars**：9749 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-19（历史快照）| **生成日期**：2026-07-01（基于 audit 快照，目标仓库克隆受限）
**主题标签**：`security-gate`, `vague-quantifier`, `manifest-discipline`, `curl-pipe-bash-risk`, `template-design`

---

## 一、理解（基于 audit 快照）

### 1.1 仓库概览
superset-sh/superset 是一个面向开发者的 AI 超级工作站套件，9749 个 star 说明它有相当大的用户基础。关键特征：
- 规模偏小但密度高：仅 13 个 NL 工件，但每个工件都是重度使用的工作流命令
- NLPM 得分仅 64/100，是本批次最低分（比 sangrokjung 低 18 分）
- 安全评级 **BLOCKED**：包含 1 个 Critical 级别安全问题（`curl | sh` 安装脚本无完整性校验）
- 架构特色：三个 CLAUDE.md（root、apps/desktop、apps/mobile）全部用 `@AGENTS.md` 单行导入，核心指引集中在 AGENTS.md 中
- 工作流命令而非 skill 套件：9 个 command 承担了大部分价值，定位为"打开就用"的操作工具

### 1.2 架构剖析
- **目录结构**（推断自 audit 报告）：
```
superset/
├── .agents/commands/    # 9 个工作流命令（ci-check、task、deslop、create-pr 等）
├── .claude/agents/      # 1 个 agent（project-structure-validator）
├── .claude/commands/    # （可能有附加命令）
├── apps/
│   ├── desktop/
│   │   ├── CLAUDE.md   # 仅含 @AGENTS.md
│   │   └── src/main/lib/agent-setup/templates/  # 4 个 hook 模板脚本
│   ├── mobile/
│   │   └── CLAUDE.md   # 仅含 @AGENTS.md
│   └── marketing/
│       └── public/cli/install.sh  # ⚠️ 含 curl-pipe-sh
├── .superset/lib/setup/steps.sh  # ⚠️ jq injection 风险
├── scripts/postinstall.sh        # 自动在 bun install 时执行
├── package.json                  # postinstall 钩子
└── CLAUDE.md                     # 仅含 @AGENTS.md
```
- **文件类型分布**：9 个 command + 1 个 agent + 3 个 CLAUDE.md（单行导入式）+ 21 个 shell 脚本
- **编排关系**：CLAUDE.md → `@AGENTS.md`（中央指引）→ 各 command 自治。无 router，command 之间有引用（refresh-compare-pages 引用 create-pr）
- **跨件契约**：AGENTS.md 是全局约定的唯一来源；3 个子目录 CLAUDE.md 通过 `@AGENTS.md` 完全委托，实现了"约定单一源"

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「单一权威约定」——不在每个子目录重复规则，全部委托给根目录的 AGENTS.md。三行 CLAUDE.md 代替三份重复文档
- **解决什么问题**：在多应用（desktop/mobile/docs）的 monorepo 中，如何避免不同目录的 CLAUDE.md 内容漂移导致 AI 理解不一致
- **做了什么 trade-off**：`@AGENTS.md` 委托模式让维护成本降至最低，代价是子目录没有任何本地化定制空间——如果 mobile 和 desktop 有不同的 AI 工作规范，就无法表达
- **反映什么认知模型**：作者将 AI 工作规范视为"代码规范"，和代码一样应有单一权威来源，而不是让 AI 自己拼接来自多处的指令

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「@Import 委托式 CLAUDE.md」（Single-Source CLAUDE.md via @Import）**

关键特征：
- 特征 1：根目录 CLAUDE.md 是唯一的实质性上下文文件；子目录 CLAUDE.md 仅一行 `@AGENTS.md`
- 特征 2：AGENTS.md 成为跨所有子应用的 AI 行为约定的单一真相来源
- 特征 3：变更约定只需改一个文件，不用同步多个 CLAUDE.md
- 特征 4：子目录可选择性覆写（通过 @import 后追加本地规则），但默认不追加

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多应用 monorepo（共享相同 AI 规范） | ✅ 高度适用 | 单一来源消除规范漂移 |
| 各子应用需要不同 AI 指令的项目 | ❌ 不适用 | @import 委托后无法分应用定制 |
| 单仓库单应用 | 中性 | 不需要这个模式，直接写 CLAUDE.md 即可 |
| 快速迭代期需要频繁改规范 | ✅ 适用 | 改一处生效处处，比多文件同步快 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 本仓库：@Import 委托 | superset-sh/superset | 单一来源，维护成本低 | 子目录无法本地化 |
| 备选 A：各目录独立 CLAUDE.md | 大多数 monorepo | 精细化控制 | 内容漂移，维护代价高 |
| 备选 B：完全在根 CLAUDE.md 覆盖 | 单 app 项目 | 简单直接 | 子目录规则与全局规则混合 |

### 2.4 改进空间
1. **当前问题**：9 个 command 全部缺 `name:` 字段，且 2 个完全无 frontmatter **改进做法**：给 create-plan.md 和 create-pr.md 先加完整 frontmatter（name/description/allowed-tools），其余 7 个只需加 `name:` **预期收益**：所有 command 可注册，分数从 64 升至约 78
2. **当前问题**：`create-plan.md` 和 `create-pr.md` 充斥模糊量词，达到扣分上限 **改进做法**：把"appropriate location"改为"与调用 command 同级目录"，把"thorough"改为"覆盖每个需求点，无一遗漏" **预期收益**：减少 AI 在边缘情况下的自由解释空间
3. **当前问题**：CRITICAL 安全问题（curl-pipe-sh）阻塞了所有外部 PR **改进做法**：在安装脚本中加 tarball SHA256 校验，使用 GitHub release asset 的已知哈希值对比 **预期收益**：清除安全门阻塞，恢复 PR 可贡献状态

---

## 三、过去审查发现（2026-04-19 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-19 得分 **64/100**（本批次最低分）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .agents/commands/create-plan.md | 15 | 零 frontmatter，模糊量词上限 |
| .agents/commands/create-pr.md | 25 | 零 frontmatter，模糊量词上限 |
| .agents/commands/task.md | 63 | 缺 name，无空输入处理 |
| CLAUDE.md | 85 | 单行 @AGENTS.md 委托（正确设计） |

### 3.2 当时值得借鉴的模式

1. **@AGENTS.md 单行委托的 CLAUDE.md** → 根本好处：多子应用共享一份规范，零漂移 → root/apps/desktop/apps/mobile 三处 CLAUDE.md → 在自己的 monorepo 里用 `@` 导入替代复制粘贴
2. **cross-reference 已验证（refresh-compare-pages 引用 create-pr）** → 根本好处：文件间引用有效，无死链 → audit 交叉引用检查结果 → 养成「写引用前先确认目标文件存在」的习惯
3. **`project-structure-validator.md` 的功能定位明确** → 一个 agent 只做一件事 → 单职责原则在 agent 层面的体现

### 3.3 当时的缺陷

1. **create-plan.md 和 create-pr.md 零 frontmatter** → 根本原因：这两个文件实际上是成长为"实质性工作流文档"的参考指南，从文档升格为 command 时没有完成 frontmatter 添加 → **自查**：我有没有从文档/README 片段直接改造成 command 的文件？
2. **9/9 command 缺 `name:` 字段** → 根本原因：与 softaworks 完全相同的系统性遗漏——作者认为文件名就是名字，不知道 Claude Code 要求 frontmatter 里显式声明 → **自查**：立即执行 `grep -rL "^name:" .claude/commands/`
3. **`jq` 注入风险（steps.sh line 113）** → 根本原因：将环境变量直接插值到 jq filter 字符串，违反了「用户输入永不内联到命令字符串」原则 → **自查**：我的 hook 或脚本里有没有把变量直接拼进命令字符串？
4. **`curl-pipe-sh` 无校验** → 根本原因：安装脚本设计遵循了易用性原则，却牺牲了完整性验证 → 这是整个仓库被 BLOCKED 的根源，所有外部 PR 均无法提交
5. **`postinstall` 自动执行脚本** → 根本原因：供应链安全意识不足——`bun install` 后自动执行 `postinstall.sh` 是隐式信任，任何依赖包妥协都会导致开发者机器被攻击

### 3.4 当时的优化机会

1. **先修安全 CRITICAL**：给 install.sh 的 tarball 下载加 SHA256 校验——两行 shell 代码，解除 BLOCKED 状态
2. **批量加 `name:` 字段到 9 个 command**：纯机械操作，解除最大面积的分数扣减
3. **给 project-structure-validator 加示例和 model 声明**：从 80 分到 90+ 分的两个关键缺失项

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
> 注：git clone 受限，以下基于推断。

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| install.sh curl-pipe-sh 无校验（CRITICAL） | `grep -A5 "curl" apps/marketing/public/cli/install.sh` | **无法验证** | BLOCKED 状态对贡献者有强激励修复此项 |
| create-plan/create-pr 零 frontmatter | `head -5 .agents/commands/create-plan.md` | **无法验证** | 若 9749 star 仓库有活跃维护则大概率已修 |
| jq 注入（steps.sh:113） | `grep "WORKSPACE_NAME" .superset/lib/setup/steps.sh` | **无法验证** | Medium 风险，修复方式明确（--arg 参数化） |

### 4.2 架构演进
audit 时仅 13 个工件，这是一个相对精简的配置。9749 star 的仓库在 2026-04 持有如此少的 NL 工件，说明 AI 工作流配置对该项目是附属功能而非核心产品。后续演进可能更关注 apps/ 的功能，而非 NL 工件数量。

### 4.3 新增的可学习模式
暂无（无法访问当前仓库状态）。

---

## 五、校准

### 5.1 我已经在做对的
1. 在跨目录项目中使用 CLAUDE.md 委托而非复制粘贴规范
2. 避免在 hook/脚本中直接插值用户输入到命令字符串（使用参数化）
3. 给 command 文件添加完整 frontmatter（name + description + allowed-tools）
4. 在安装脚本中做完整性校验而非直接执行 curl 输出
5. 将复杂工作流命令分解为有序步骤而非散落的操作列表

### 5.2 挑战 / 验证
- **挑战的假设**：我之前认为高 star 仓库质量必然高——但 9749 star 的 superset 得了 64/100，是本批次最低分，且被安全门 BLOCKED。Stars 衡量的是产品价值，不是 NL 工件质量。这两个维度相互独立。
- **另一个挑战**：低分不一定意味着作者水平低。superset 的 @AGENTS.md 委托模式是设计上的亮点，但 NL 工件的"形式合规性"（frontmatter 填写等）显然是低优先级。作者在产品功能上花了精力，在 NL artifact 规范上没有。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查自己的脚本中有没有把变量直接插值到 shell 命令字符串（jq/python 注入风险）
grep -rn 'jq.*"\$\|python3.*"\$' ~/.claude/hooks/*.sh ~/.claude/scripts/*.sh 2>/dev/null
# 命中后：改用 --arg 参数化（jq）或 argparse/os.environ（python）

# 检查有没有 curl-pipe-sh 或 curl 后直接执行输出的写法
grep -rn "curl.*|.*sh\|curl.*| bash\|eval.*curl" ~/bin/* ~/.claude/scripts/* 2>/dev/null
# 命中后：改为 curl + 存文件 + SHA256校验 + 执行三步分离

# 检查 monorepo 项目中各子目录的 CLAUDE.md 有没有不必要的重复规范
find . -name "CLAUDE.md" | while read f; do wc -l "$f"; done | sort -n
# 短文件（1-5行）是正常的委托文件；长文件（30行+）要检查是否可以改用 @import
```

### 6.2 灵感 → 实施路径

1. **想法**：在 bureau 项目中如果有多个应用场景（如 capture、review、query 各有子目录），用 @AGENTS.md 委托而非重复 CLAUDE.md
   - **为何可行**：bureau 各阶段共享同一套 AI 行为约定，委托模式天然合适
   - **第一步**：检查 bureau 是否已有子目录 CLAUDE.md；若有，用 `echo "@../AGENTS.md" > apps/subdir/CLAUDE.md` 替换内容，5 分钟完成

2. **想法**：给 gstack 的安装脚本（如果有）加 SHA256 校验
   - **为何可行**：gstack 有 23 个工具，如果有安装脚本供他人运行，安全是基础
   - **第一步**：`find . -name "install.sh" -o -name "setup.sh"` 找到脚本，在下载 tarball 后加 `sha256sum -c <<< "$EXPECTED_SHA  file.tar.gz"` 校验，10 分钟完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **superset-sh/superset 的核心目的**：多应用 monorepo 的 AI 工作流配置套件，重点是命令式工作流（创建 PR、运行 CI 检查等）

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 中 | 同为多工具工作流配置，有 CI/CD 类操作 | gstack 更多角色导向，superset 更多操作导向 | 中 |
| MarkQWu/bureau | 低 | 同为 Claude Code 插件 | bureau 专注知识管理，superset 专注开发操作 | 低 |
| MarkQWu/shiyun | 无 | — | 完全不同领域（诗词数据库） | 无 |

若本案例仓库类型与用户仓库不太接近，主要做技术模式对照（§7.3）。

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 我的项目推测情况 | 严重度 |
|---|---|---|
| command 缺 `name:` 字段（9/9 命中） | gstack 有 23 个工具 command，若批量创建则极可能有遗漏 | 高 |
| 脚本变量插值（jq/shell 注入） | 若 bureau 或 echo-sleuth 有 hook 脚本则需排查 | 中 |
| postinstall 自动执行第三方脚本 | 若有 package.json 则需检查 postinstall 字段 | 中 |

**命中后的具体行动建议**：
- gstack：`grep -rL "^name:" .claude/commands/*.md` 批量检查，每个缺失文件 2 分钟修复
- 所有含 hook/script 的仓库：`grep -n '"\$' .claude/hooks/*.sh 2>/dev/null` 排查变量插值

### 7.3 别人的更优方案

1. **领域**：多子目录 CLAUDE.md 的维护方式
   - **superset 做法**：三个 CLAUDE.md 全部是 `@AGENTS.md` 一行，变更只需改一个文件
   - **我的项目现状**：若我的 monorepo 项目有多个子目录 CLAUDE.md 且内容各不相同，则维护成本高且容易漂移
   - **如何借鉴**：找我最复杂的 monorepo 项目，把子目录 CLAUDE.md 改为 `@<相对路径>/AGENTS.md`，并把公共规范迁移到根目录 AGENTS.md

### 7.4 反向：我的项目做得比他们好的地方

若我的 command 文件有完整 frontmatter（name + description + allowed-tools），则在这一维度优于 superset-sh/superset——后者 9/9 command 全部缺 `name:` 字段，整体 NL 得分落到 64 分。这是一个基础规范上的明显差距。

---

## 八、术语表

### <a name="curl-pipe-sh"></a>curl-pipe-sh
> 一种安装模式：`curl https://xxx.sh | sh`，直接把远程脚本下载并执行，中间没有任何校验步骤。危险点在于：如果 CDN 或下载服务器被攻击者控制，用户会直接执行恶意代码，且无任何提示。NLPM 安全扫描器将其标记为 CRITICAL 风险。

### <a name="postinstall"></a>postinstall 脚本
> `package.json` 的 `"scripts": {"postinstall": "..."}` 字段。当用户运行 `npm install` 或 `bun install` 时，安装完依赖后**自动**执行该命令。危险点：这是一个自动触发的钩子，任何克隆该仓库的开发者执行安装都会运行它，是供应链攻击的常用入口。

### <a name="jq-injection"></a>jq 注入
> 在 shell 脚本中，把用户输入的变量直接拼接到 jq filter 表达式的字符串里（如 `jq ".[] | select(.name == \"$WORKSPACE_NAME\")"`）。如果 `$WORKSPACE_NAME` 包含 jq 特殊字符，攻击者可以破坏 filter 逻辑。正确做法：用 `jq --arg name "$WORKSPACE_NAME" '.[] | select(.name == $arg)'` 参数化传递。

### <a name="AGENTS.md"></a>AGENTS.md
> Claude Code 支持的一种约定文件，功能与 CLAUDE.md 类似，但通常放在项目根目录并通过 `@AGENTS.md` 语法被其他 CLAUDE.md 引入。实现了"单一权威 AI 行为规范"模式。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件顶部用 `---` 包裹的 YAML 配置块。Claude Code 读取 frontmatter 来注册 command/agent 的名字、描述和工具权限。没有 frontmatter 的文件对 Claude Code 来说是"不存在的"——它不会被注册也不会被触发。
