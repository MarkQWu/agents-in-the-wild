# vstorm-co/pydantic-deepagents — 学习案例

**仓库**：https://github.com/vstorm-co/pydantic-deepagents
**Stars**：687 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-04（基于当前 HEAD）
**主题标签**：`security-gate`, `curl-pipe-bash-risk`, `vague-quantifier`, `template-design`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
`pydantic-deepagents` 是一个基于 [pydantic-ai](https://github.com/pydantic/pydantic-ai) 的 AI agent 框架库，同时为自己的 CLI 和研究工具内置了一套 Claude Code skill 套件。本质是「一个 Python 库 + 附带的 NL 工件」：核心价值是 Python agent framework，skill 是辅助开发者用 Claude Code 配合使用这个框架的工具。

关键事实：
- 31 个 NL 工件（全部是 skill，无 command/agent/manifest）
- skill 分布在三个路径：`apps/cli/skills/`、`pydantic_deep/bundled_skills/`（与 cli 完全相同）、`examples/`
- `apps/cli/` 和 `pydantic_deep/bundled_skills/` 是字节级相同的两棵 skill 树（11 对完全相同文件）
- 安全状态：**BLOCKED**——`install.sh` 和 `apps/harbor/agent.py` 中有 2 个 `curl | sh` 模式

### 1.2 架构剖析
```
pydantic-deepagents/
├── apps/
│   ├── cli/
│   │   └── skills/              ← 9 个 skill（code-review/test-writer/refactor/...）
│   ├── deepresearch/
│   │   └── skills/              ← 3 个 skill（diagram-design/research-methodology/report-writing）
│   └── harbor/                  ← Docker 容器化 agent（含 install.sh）
├── pydantic_deep/
│   └── bundled_skills/          ← 与 cli/skills/ 完全相同的 11 个 skill（字节级复制）
├── examples/
│   ├── full_app/skills/         ← 示例版 skill（质量低于生产版）
│   └── skills/                  ← 独立示例版 skill（与 full_app 相同）
├── install.sh                   ← ⚠️ 含 curl | sh（Critical）
└── CLAUDE.md                    ← 开发文档
```

- **文件类型分布**：30 个 Skill + 1 个 CLAUDE.md（被作为 context 工件评分）
- **编排关系**：skills 之间平级，无 router/命令/agent。所有 skill 直接由用户调用，框架不做编排
- **跨件契约**：`cli/code-review` 和 `examples/code-review` 是两个版本的同一 skill，但 examples 版本质量更低（有断链 reference 和更多模糊量词）——这是一个反向契约违反

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「库 + skill 一起发布」——开发者安装这个 Python 库时，同时获得配套的 AI skill
- **解决什么问题**：pydantic-ai 的 API 有学习曲线，配套 skill 让 Claude 了解框架约定，减少开发者写 AI prompt 的时间
- **做了什么 trade-off**：为了在 CLI 安装和 `pydantic_deep` Python 包两个路径下都能加载 skill，选择了完全复制而非符号链接。代价是任何修改都要同步两处
- **反映什么认知模型**：作者把 Claude Code skill 理解为「框架文档的可执行形态」——用户调用 skill 就像查阅框架最佳实践文档，同时 Claude 会在当前项目语境中应用这些实践

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「框架库附带型 NL 工件（Library-Embedded Skills）」**

Python/JS 库随包发布配套 Claude Code skill，作为框架使用文档的可执行替代。

模式特征清单：
- 特征 1：skill 内容即框架使用规范（`code-review/SKILL.md` 告诉 Claude 如何做 pydantic-ai 项目的代码审查）
- 特征 2：skill 随 Python 包安装路径自动部署（`pydantic_deep/bundled_skills/` 随 pip 安装）
- 特征 3：无 manifest，无 plugin.json——skill 不是以 Claude plugin 形式分发，而是随代码库存在
- 特征 4：`CLAUDE.md` 充当高层项目说明，引导 Claude 了解仓库架构
- 特征 5：examples/ 目录的 skill 是教学材料，质量低于生产版本（反模式：示例版本更差）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| Python/JS 框架库，想随包提供 AI 辅助 | ✅ 高度适用 | skill 随包安装，用户零配置获得 AI 加持 |
| 独立 Claude Code plugin 分发 | ❌ 不适用 | 没有 plugin.json，不能通过 marketplace 安装 |
| 需要跨版本 skill 管理 | ⚠️ 慎用 | 两套相同文件的同步成本高，容易出现版本分叉 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 框架库附带型（本仓库） | vstorm-co/pydantic-deepagents | 随包安装，零配置 | 无 plugin 机制，更新需重装包 |
| 独立 plugin | twostraws/SwiftUI-Agent-Skill | plugin 机制管理版本和更新 | 用户需单独安装 |
| 仓库级 CLAUDE.md | 多数开源项目 | 极简，无学习成本 | 功能受限，无法按需加载 skill |

### 2.4 改进空间
1. **当前问题**：`install.sh` 第 38 行 `curl -LsSf https://astral.sh/uv/install.sh | sh` 无完整性校验，仍然存在。**改进做法**：改用固定版本下载 + sha256 校验，如 `curl -LsSf https://astral.sh/uv/0.x.y-installer.sh | INSTALLER_HASH=<sha256> sh` 或使用官方提供的验证方式。**预期收益**：消除 MitM 和供应链攻击风险。
2. **当前问题**：`apps/cli/skills/` 和 `pydantic_deep/bundled_skills/` 字节相同，但各自维护。**改进做法**：用符号链接或 Makefile copy 步骤替代手动双份维护，消除 DRY 违反。**预期收益**：任何修复只需改一处。
3. **当前问题**：`examples/` 下的 skill 质量低于生产版（有断链 reference、更多模糊量词）。**改进做法**：examples skill 直接 symlink 到 cli 版本，或加 CI 检查两者 diff 应为零。**预期收益**：新手从 examples 学到的是最佳实践而非次优版本。

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **93/100**（31 个文件加权平均）。

| 文件组 | 当时分数 | 主要问题 |
|---|---|---|
| `apps/cli/skills/verification-strategy/SKILL.md` | 88/100 | 无 Output Format、`reasonable` 模糊量词（-12） |
| `apps/cli/skills/{build/refactor/performant/...}`（7 个） | 90/100 | 无 Output Format（-10 each） |
| `examples/*/code-review/SKILL.md`（2 个） | 92/100 | 断链 `example_review.md`、4 个模糊量词 |
| `apps/cli/skills/{code-review,test-writer,skill-creator}`（3 个） | 100/100 | 无问题 |
| CLAUDE.md | 95/100 | `sensible defaults` 轻微模糊（-2） |

### 3.2 当时值得借鉴的模式
1. **100 分 skill 的结构** → `apps/cli/skills/code-review/SKILL.md` 得满分：精确 Output Format（`File:line — category — description — severity`）、无模糊量词、清晰分类 checklist → 可以直接作为 skill 写作的对照模板
2. **`skill-creator/SKILL.md` 内嵌 skill 模板** → skill body 里包含完整的 skill 创建模板，Claude 执行 skill 时直接生成符合规范的新 skill → `apps/cli/skills/skill-creator/SKILL.md` → 工具类 skill 可以嵌入自己的输出模板
3. **feature-grouped 目录结构** → `CLAUDE.md` 明确说明「按 feature 组织，不按类型平铺」（`organize by feature, not by kind`）→ 避免 `toolsets/`、`capabilities/` 这种按类型平铺的反模式
4. **垂直切片（vertical slice）功能包** → 每个 feature 下有 `capability.py + toolset.py + service.py`，老的 `toolsets/` 等目录保留为 deprecation shim 重导出 → 迁移时的向后兼容策略值得借鉴

### 3.3 当时的缺陷
1. **`curl | sh` 无完整性校验（CRITICAL）**：`install.sh` 和 `apps/harbor/agent.py` 中直接 `curl ... | sh`，不验证下载文件的 sha256 或签名 → 根本原因：作者把安装便捷性放在安全性之上，没有意识到 HTTPS 并不保证内容完整性（DNS 劫持、CDN 被入侵等场景下 HTTPS 依然可能返回恶意内容）→ 自查：我的任何脚本是否有类似模式？
2. **examples/skill 质量低于 cli/skill**：`examples/code-review/SKILL.md` 引用了不存在的 `example_review.md`，同名词还有更多模糊量词 → 根本原因：examples 目录是早期版本，cli 版本迭代后 examples 没有同步更新 → 自查：我的仓库是否存在「示例版本比生产版本更老」的情况？
3. **Output Format 缺失（9 个 skill）**：7 个 cli skill 无 `## Output Format` 节 → 根本原因：作者关注的是「做什么」（checklist），没有规定「输出长什么样」，导致 Claude 每次格式自由发挥 → 自查：我的 skill 是否都有 Output Format？

### 3.4 当时的优化机会
1. 修复 `example_review.md` 断链（在 examples/code-review/ 创建文件，或改为引用存在的文件）
2. 合并重复 skill 树（cli 和 bundled_skills 用 symlink 或 Makefile 统一管理）
3. 为 9 个缺少 Output Format 的 skill 添加 `## Output Format` 节

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `example_review.md` 断链 | `ls examples/full_app/skills/code-review/` | ✅ **已修复**（`example_review.md` 文件已创建） | 作者修复了这个可见 bug |
| `install.sh` 的 `curl \| sh`（Critical #1） | `grep -n "curl.*\|.*sh" install.sh` | ❌ **仍存在**（第 38 行 + 第 569 行） | Critical 安全问题未修复 |
| `apps/harbor/agent.py` 的 `curl \| sh`（Critical #2） | `grep -n "curl.*\|.*sh" apps/harbor/agent.py` | **部分变化**（原第 320 行代码被重构，但 harbor 文件第 569 行附近仍存在 curl\|sh 模式） | 代码重构但根本风险未消除 |
| `examples/skill` 与 `cli/skill` 分叉 | `diff apps/cli/skills/code-review/SKILL.md examples/skills/code-review/SKILL.md` | **仍存在差异** | 双 skill 树问题持续 |

### 4.2 架构演进
新增了 `SECURITY.md`（说明安全政策和支持版本）、`SYSTEM_REVIEW.md`、`CODE_QUALITY_REPORT.md`，说明作者在 audit 后增加了文档层面的「安全意识展示」，但 `install.sh` 中的实际 curl|sh 代码仍未修改。这是一个有趣的分裂：文档层面做了安全声明，代码层面安全问题仍在。

### 4.3 新增的可学习模式
`examples/security_gate/` 目录新增了一个安全门控示例，展示如何在 pydantic-ai agent 中添加安全检查 hook。这是 audit 后新增的、与安全相关的教学材料——可能是作者对安全问题的一种回应方式（新增 examples 而非修复现有问题）。

---

## 五、校准

### 5.1 我已经在做对的
1. **Output Format 显式声明**：gstack 的 skill 普遍有 `## Output Format` 或等效的输出规范节
2. **无 curl|sh 安装脚本**：我的仓库脚本不使用 curl|sh 模式
3. **code-review 满分 skill 的结构**：gstack 的 `spec/SKILL.md` 使用类似的精确 Output Format（`File:line` 格式）
4. **feature-grouped 目录**：bureau 按功能组织（`skills/recall/`、`skills/capture/` 等），不按文件类型平铺

### 5.2 挑战 / 验证
**验证**：之前我对「example 质量可以低一些」有一定容忍度——毕竟是示例。但这个案例反面证明：用户往往先看 examples 再看生产版，如果 examples 有断链、质量更低，反而会给用户留下错误印象，甚至让用户学到错误的写法。**结论：example 版本质量应该 ≥ 生产版本，或者直接 symlink。**

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的脚本是否有 curl|sh 模式
grep -rn -E "curl.*\|.*(sh|bash)|wget.*\|.*(sh|bash)" \
  /tmp/my-repos/MarkQWu-bureau /tmp/my-repos/MarkQWu-gstack \
  --include="*.sh" 2>/dev/null | head -10
```
命中后：替换为固定版本下载 + sha256 校验，或改用包管理器安装。

```bash
# 检查我的 examples/skill 与生产 skill 是否存在质量差异
# 以 gstack 为例，检查 examples 目录（如果存在）
find /tmp/my-repos/MarkQWu-gstack -path "*/examples/*.md" 2>/dev/null | head -10
```
命中后：对比 examples 版本和生产版本，如有差异则将 examples 改为 symlink 或 CI 检查两者 diff。

### 6.2 灵感 → 实施路径
1. **想法**：把 vstorm 的 `apps/cli/skills/code-review/SKILL.md`（100 分）作为参照，对照我的 bureau `skills/review/SKILL.md` 做改进
   - **为何可行**：同样是代码 review skill，100 分版本有精确的 Output Format、零模糊量词；我的版本缺少 Output Format
   - **第一步**：`cat /tmp/my-repos/MarkQWu-bureau/skills/review/SKILL.md`，对照 vstorm 的 `## Output Format`（`File:line — category — description — severity`），在 bureau 的 review skill 末尾添加同样格式的输出规范 → 预计 10 分钟

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 vstorm-co/pydantic-deepagents 的核心目的**：pydantic-ai agent 框架库 + 配套 Claude Code skill 套件，辅助框架使用者用 AI 配合开发

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/graphify | 中 | 都是「AI 工具 + 配套 NL 工件」 | graphify 是知识图谱查询工具，pydantic-deepagents 是 Python agent 框架 | 中 |
| MarkQWu/bureau | 低 | 都有 skill 套件 | bureau 是知识管理，目标不同 | 低 |
| MarkQWu/gstack | 低 | 都是多 skill 套件 | gstack 面向工程工作流 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 无 `## Output Format` 节 | `grep -rL "Output Format" /tmp/my-repos/MarkQWu-bureau/skills/*/SKILL.md` | **bureau 7/7 SKILL.md 无 Output Format 节** | 中 |
| examples 版本低于生产版本 | 检查各仓库 examples/ 目录 | graphify 和 echo-sleuth 无 examples 目录，不存在此问题 | 无 |

**命中后的具体行动建议**：
- `bureau/skills/recall/SKILL.md` → 末尾加 `## Output Format` 节（格式：`**[来源文件]**：引用块 + 信任层级标注`） → 10 分钟
- `bureau/skills/review/SKILL.md` → 加 `## Output Format`（格式：`## [版本/日期] 审查摘要 + 每条发现的文件:行 → 建议`） → 10 分钟

### 8.3 别人的更优方案

1. **领域**：满分 skill 的输出格式设计
   - **本案例做法**：`apps/cli/skills/code-review/SKILL.md` 的 Output Format：`For each issue found: **File:line** — category — description — suggested fix · Severity: critical / warning / suggestion`（精确、可解析、无歧义）
   - **我的项目现状**：bureau `skills/review/SKILL.md` 无 Output Format 节，Claude 自由发挥格式
   - **如何借鉴**：直接复用这个格式约定，修改 category 适配 bureau 的场景

2. **领域**：CLAUDE.md 项目上下文文档
   - **本案例做法**：`CLAUDE.md` 清晰说明架构、命令、设计原则（feature-grouped、vertical slice），给 Claude 全局视图
   - **我的项目现状**：gstack 的 CLAUDE.md 很完整；bureau 的 CLAUDE.md 内容较少，缺少架构说明
   - **如何借鉴**：为 bureau 的 `CLAUDE.md` 补充「目录结构说明」和「核心设计哲学」两个节

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：plugin.json manifest 的维护
- **我的做法**：bureau 有完整的 `plugin.json`，每个 skill 都被注册，有版本号和 semver
- **本案例做法**：pydantic-deepagents 的 30 个 skill 全部无 plugin.json（不以 Claude plugin 形式分发），不能通过 marketplace 安装，更新依赖重新安装 Python 包
- **意义**：我的仓库以 plugin 形式分发，用户可以通过 `claude plugin install` 直接安装和更新，分发机制更规范。

---

## 八、术语表

### <a name="curl-pipe-bash"></a>curl|bash（curl-pipe-bash）
> 一种常见但不安全的安装模式：`curl <URL> | bash` —— 直接从网络下载一段脚本并立即执行，不做任何完整性校验。风险：如果下载 URL 被劫持（DNS 劫持、CDN 被黑），用户会在不知情的情况下执行攻击者的代码。更安全的替代方案：下载到本地文件 → 验证 sha256 → 执行。

### <a name="vertical-slice"></a>垂直切片（vertical slice）
> 一种代码组织方式：按功能（feature）而非按层（layer）组织代码。例如 `features/research/` 目录下包含该功能的所有层（API、业务逻辑、数据访问），而非把所有 API 放在 `controllers/`、所有逻辑放在 `services/`。Claude Code 的 skill 组织也适用同样原则：每个 skill 目录自包含它需要的所有文件。

### <a name="deprecation-shim"></a>deprecation shim（废弃垫片）
> 为了向后兼容而保留的「空转」文件，内容只是重导出（re-export）新路径下的内容。用户代码在迁移时不需要立即修改 import 路径，但系统内部已经重组。vstorm 的 `toolsets/` 目录保留了旧路径的 shim，实际代码已迁移到 `features/`。

### <a name="output-format"></a>Output Format 节
> SKILL.md 末尾的专门节，明确规定 Claude 执行 skill 后的输出结构（字段、顺序、格式）。缺少此节时 Claude 每次自由发挥格式，导致输出不可预期、难以被下游程序解析。
