# agent-sh/agentsys — 学习案例

**仓库**：https://github.com/agent-sh/agentsys
**Stars**：892 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-12（历史快照）| **生成日期**：2026-07-14（基于当前 HEAD）
**主题标签**：`examples-driven`, `vague-quantifier`, `security-gate`, `cross-reference`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
agentsys 是 agent-sh 组织开源的「AI 代码代理模块化运行时和编排系统」，兼容 Claude Code、OpenCode、Codex CLI 和 Cursor，提供 24 个插件、49 个 agent、44 个 skill。2026 年 1 月上线，版本 6.0.0，892 颗星。它的核心价值是让 AI 不只是「改代码」，而是接管整个开发工作流（代码分析、性能基准测试、文档同步、hook 维护）。

- **创建时间**：2026-01-15
- **核心语言**：JavaScript（Node.js 工具链）
- **获取方式**：`claude plugin install agentsys@agent-sh --scope project`
- **生态位置**：多平台兼容（Claude Code + OpenCode + Codex + Cursor），属于最具野心的多运行时插件之一

### 1.2 架构剖析
- **目录结构**：
  ```
  agent-sh/agentsys
  ├── .kiro/skills/          ← Kiro IDE 的 skill 目录（27 个 skill）
  │   ├── consult/SKILL.md
  │   ├── debate/SKILL.md
  │   ├── deslop/SKILL.md
  │   ├── enhance-*/SKILL.md  ← 8 个增强类 skill
  │   ├── perf-*/SKILL.md     ← 8 个性能分析 skill
  │   └── ...
  ├── meta/skills/           ← 元 skill（跨平台维护参考）
  ├── .claude-plugin/
  │   └── plugin.json        ← Claude Code manifest
  ├── scripts/               ← 23 个 JS 工具脚本
  ├── package.json
  └── CLAUDE.md
  ```
- **文件类型分布**：27 个 SKILL.md / 多个 agent / 1 个 [manifest](#manifest) / 23 个脚本
- **编排关系**：`enhance-orchestrator` 是核心编排 skill，内置与 `enhance-prompts`、`enhance-skills`、`enhance-plugins`、`enhance-agent-prompts` 的差异化表，防止混淆。`perf-*` 系列 8 个 skill 共同引用一个 `docs/perf-requirements.md` 契约文档，形成有中心化契约的扁平编排。
- **跨件契约**：所有跨平台脚本路径统一由 `meta/skills/maintain-cross-platform/SKILL.md` 定义，这是一个 1024 行的「平台维护元 skill」，作为单一事实来源。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「多平台一次编写，统一路径约定」——通过 `maintain-cross-platform/SKILL.md` 作为所有平台路径的 single source of truth，减少各平台之间的路径漂移。
- **解决什么问题**：跨 4 个 AI 平台（Claude/OpenCode/Codex/Cursor）维护相同的 agent 和 skill，面临路径约定不一致的挑战。
- **Trade-off**：覆盖场景广（performance 分析、文档同步、代码增强）但质量参差不齐——高分 skill（97-98 分）和低分 skill（83-85 分）并存。
- **认知模型**：把 AI skill 视为「可组合模块」，大量 `enhance-*` 系列 skill 按职责细分，类似 Unix 工具哲学。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「契约驱动的平铺扩展式」** — 通过一个中央契约文档（`perf-requirements.md`）和一个元 skill（`maintain-cross-platform`）维护大量 skill 之间的一致性，允许 skill 数量无限增长而不互相矛盾。

模式特征清单：
- 特征 1：存在中央契约文档（`docs/perf-requirements.md`），所有 perf-* skill 引用它
- 特征 2：元 skill（`maintain-cross-platform`）作为路径命名约定的 single source
- 特征 3：差异化表内置于编排 skill（`enhance-orchestrator` 区分自己与相似 skill）
- 特征 4：skill 按语义前缀分族（`perf-*`、`enhance-*`），便于批量管理
- 特征 5：版本号在 `package.json` 和 `plugin.json` 双重声明且保持对齐

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多平台兼容的大型 skill 库 | ✅ 高度适用 | 中央契约防止各平台路径漂移 |
| 性能基准测试系列任务 | ✅ 适用 | 共享 perf-requirements.md 契约，8 个 skill 高度协调 |
| 小型单平台项目 | ❌ 过度设计 | 维护 1024 行元 skill 成本高于收益 |
| 需要频繁添加新 skill 的项目 | ⚠️ 有条件适用 | 新 skill 必须先读元 skill，上手成本高 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 契约驱动平铺扩展式（本仓库） | agentsys | 跨平台一致，按族分组 | 质量参差，入门成本高 |
| 参考实现锚定式 | openai/symphony | 质量极高，易审查 | skill 数量少 |
| 多 plugin 聚合式 | ccplugins | 功能极丰富 | bug 率高，缺乏统一约定 |

### 2.4 改进空间
1. **当前问题**：12 个 perf-* 和运营类 skill 缺示例 **改进做法**：为每个 skill 添加一个「典型场景 → 输出」的 `<example>` 块，不需要全面覆盖 **预期收益**：NL 分数从 85 提升到 95+，AI 在首次调用时输出格式更稳定
2. **当前问题**：`maintain-cross-platform/SKILL.md` 长达 1024 行 **改进做法**：拆出「添加新功能」和「安装器深潜」两节到独立 `reference.md` **预期收益**：核心 SKILL.md 降至 500 行以内，保留速查功能
3. **当前问题**：`CLAUDE.md` 的 8 条规则只说「做什么」，不说「为什么」 **改进做法**：每条规则后加一句 WHY（如「Rule 3: Tests must pass → 因为 AI 没有上下文来判断是否影响其他功能，测试是唯一客观验证」）**预期收益**：AI 在规则边缘情况时做出更好的判断

---

## 三、过去审查发现（2026-04-12 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-12 当时得分 **91/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| sync-docs/SKILL.md | 83 | 缺示例 + vague quantifier |
| perf-*/SKILL.md（12 个） | 85 | 缺示例 |
| CLAUDE.md | 88 | 缺 WHY，负面措辞 |
| web-auth/SKILL.md | 90 | 硬编码路径 `/Users/avifen/` |
| web-browse/SKILL.md | 90 | 同上（40+ 处） |
| maintain-cross-platform | 95 | 超长（1024 行） |
| enhance-orchestrator | 98 | 近满分 |

### 3.2 当时值得借鉴的模式
1. **`enhance-orchestrator` 的差异化表** → 为什么好：明确区分自己与其他相似 skill（`enhance-skills`、`enhance-prompts`），防止 AI 调用错误的 skill。原文：`.kiro/skills/enhance-orchestrator/SKILL.md` → 借鉴：当项目有多个相似功能的 skill 时，在编排 skill 中加一张"什么时候用我 vs 其他 skill"的对比表
2. **`perf-requirements.md` 作为 perf-* skill 族的中央契约** → 为什么好：8 个 skill 共享同一份需求文档，修改时只需改一处。原文：所有 `perf-*/SKILL.md` 的 "canonical contract" 引用 → 借鉴：系列 skill（如 `bureau` 的 capture/compile/review 系列）可以共享一份"知识库约定文档"
3. **package.json 与 plugin.json 版本号双重校验** → 为什么好：两处版本号对齐（均 5.8.3），消除版本漂移风险。原文：`package.json` line 1 + `.claude-plugin/plugin.json` → 借鉴：多 manifest 项目应将版本号检查加入 CI

### 3.3 当时的缺陷
1. **12 个 skill 缺示例**（尤其 perf-* 系列）→ 为什么失败：perf skill 需要 AI 「知道一个典型基准场景长什么样」，没有示例则 AI 只能靠猜 → 自查：我的 gstack 的 perf/benchmark skill 也缺示例
2. **`web-auth`/`web-browse` 硬编码 `/Users/avifen/` 路径** → 为什么失败：这两个 skill 对除原作者以外的所有用户完全失效，属于「对 892 个 star 用户不可用」的严重缺陷 → 自查：我的 skill 中有没有硬编码本机路径？
3. **CLAUDE.md 规则无 WHY** → 为什么失败：AI 在规则边缘情况下无法判断优先级，会随机选择 → 自查：我的 CLAUDE.md 规则有 WHY 吗？

### 3.4 当时的优化机会
1. 为所有 `perf-*` skill 添加示例（优先级最高，影响 12 个文件）
2. 替换 `web-auth`、`web-browse` 中的硬编码路径为 `~/.agentsys/...`
3. 将 `prepare` 脚本移出 npm 生命周期，改为需要显式调用的 `setup-hooks` 命令

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态
| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `web-auth`/`web-browse` 硬编码路径 | `find .kiro/skills/ -name "web-*"` | **已修复**：两个 skill 已从仓库中完全删除 | 通过删除而非修复路径解决问题 |
| `prepare` 自动安装 git hook | `grep "prepare" package.json` | **已修复**：`prepare` 脚本已被移除 | npm install 不再自动修改用户的 git hook |
| 12 个 perf-* skill 缺示例 | `grep -c "## Example\|<example" .kiro/skills/perf-*/SKILL.md` | **仍存在**：8 个 perf-* skill 全部 0 示例 | 历经 3 个月，质量问题未改善 |

### 4.2 架构演进
从审计时到现在（v6.0.0），`web-auth` 和 `web-browse` 两个最有安全风险的 skill 被彻底删除。`prepare` 脚本移除降低了安装时风险。但 `.kiro/skills/` 仍保持 27 个 skill，perf-* 系列无变化。作者选择「删除问题文件」而非「修复路径」，体现了「维护负担 > 修复成本」时的务实取舍。

### 4.3 新增的可学习模式
暂无——该仓库近期无新增文件。

---

## 五、校准

### 5.1 我已经在做对的
1. `MarkQWu/bureau` 的 skill 有示例（3 个 `<example>` 块），比 agentsys 的 perf-* 系列做得更好
2. 我没有在 skill 中硬编码本机路径（如 `/Users/xxxx/`）
3. `MarkQWu/gstack` 的 CLAUDE.md 有较清晰的规则组织，部分规则有 WHY 暗示

### 5.2 挑战 / 验证
- **验证**：agentsys 删除 `web-auth`/`web-browse` 而非修复——这验证了一个原则：**含有不可移植硬编码路径的 skill，最好直接删掉重写，而不是试图打补丁**。我在审查自己的 skill 时如果发现同类问题，应该大胆删除。

---

## 六、行动

### 6.1 自查动作
```bash
# 检查我的 skill 中有没有硬编码本机路径
grep -rn "/Users/$(whoami)\|/home/$(whoami)" \
  ~/.claude/skills/ ~/*/skills/ 2>/dev/null
```
命中后：立即删除该 skill 并用平台无关路径重写（`~/.toolname/` 或环境变量）。

```bash
# 检查 package.json/npm 是否有 prepare 自动安装 hook
find ~/*/  -name "package.json" -maxdepth 3 | xargs grep "\"prepare\"" 2>/dev/null
```
命中后：将 `prepare` 重命名为 `setup-hooks`，并在 CONTRIBUTING.md 说明需显式运行。

```bash
# 检查 perf/benchmark 相关 skill 是否有示例
grep -L "## Example\|<example" ~/*/skills/*perf*/* ~/*/skills/*bench*/* 2>/dev/null
```
命中后：为每个 benchmark/perf skill 添加一个「输入场景 → 期望输出」的示例。

### 6.2 灵感 → 实施路径
1. **想法**：为 gstack 的 benchmark/performance 相关 skill 添加差异化表（区分 benchmark vs canary vs health skill）
   - **为何可行**：gstack 中有 `benchmark`、`benchmark-models`、`canary`、`health` 四个相似 skill，用户（和 AI）容易混淆
   - **第一步**：读 `agentsys/.kiro/skills/enhance-orchestrator/SKILL.md` 中的差异化表示例，在 gstack/benchmark/SKILL.md 中添加「什么时候用 benchmark vs canary vs health」的对比节

---

## 七、对照我的 GitHub 仓库

### 7.1 目的对齐度
- **本案例 agent-sh/agentsys 的核心目的**：多平台兼容的大型 skill 运行时，覆盖从性能分析到文档维护的完整开发工作流

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/gstack | 高 | 都是以 skill 驱动的开发工作流自动化 | gstack 是单平台(Claude)，agentsys 多平台 | 高 |
| MarkQWu/bureau | 低 | 都用 SKILL.md 格式 | bureau 是知识管理，不是代码工作流 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| perf-*/benchmark skill 缺示例 | `grep -L "<example" gstack/*/skills/*bench*/SKILL.md` | gstack 的 `benchmark`、`benchmark-models` 均无示例 | 高 |
| CLAUDE.md 规则无 WHY | `grep -c "because\|why\|due to" gstack/CLAUDE.md` | gstack CLAUDE.md WHY 出现次数 1（很少）| 中 |
| 多 skill 无差异化表 | 目视检查相似功能 skill | gstack 有 benchmark/canary/health 三个相似 skill，无区分说明 | 中 |

**命中后的具体行动建议**：
- `gstack/benchmark/SKILL.md`（假设路径） → 添加「适用场景 vs benchmark-models vs canary」对比表 → 20 分钟
- gstack CLAUDE.md → 为每条规则补一句「为什么」→ 30 分钟

### 7.3 别人的更优方案

1. **领域**：`cross-reference`（编排 skill 内嵌差异化表）
   - **本案例做法**：`enhance-orchestrator/SKILL.md` 包含完整的「什么时候用我 vs 其他 enhance-* skill」区分表（`.kiro/skills/enhance-orchestrator/SKILL.md`）
   - **我的项目现状**：gstack 中 `benchmark`、`benchmark-models`、`canary`、`health` 四个相似 skill 没有相互区分说明
   - **如何借鉴**：在 gstack 的 benchmark/SKILL.md 中添加一节「相似 skill 区分」

2. **领域**：`manifest-discipline`（多 [manifest](#manifest) 版本号对齐）
   - **本案例做法**：`package.json` 和 `plugin.json` 同时声明 `5.8.3`，且在 CI 中可验证
   - **我的项目现状**：gstack 如有多处版本声明，未明确对齐策略
   - **如何借鉴**：在 gstack 的 CLAUDE.md 中加规则「修改版本号时必须同步更新所有 manifest」

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：`examples-driven`（示例块）
- **我的做法**：`MarkQWu/bureau` 所有 7 个 skill 都有 `<example>` 块（最少 3 个）
- **本案例做法**：agentsys 的 12 个 perf-* skill 全部 0 示例，历经 3 个月未改善
- **意义**：bureau 的 skill 在「提供 AI 可学习的具体示例」这点上领先 agentsys；在 skill 写作时坚持「先有示例再有步骤」的做法是对的

---

## 八、术语表

### <a name="manifest"></a>manifest
> 项目的「清单文件」，告诉系统这个插件包含哪些组件。`plugin.json` 是 Claude Code 插件的 manifest——里面列出所有 commands、skills、agents 的路径。如果 manifest 里漏写了某个文件，那个文件即使在硬盘上也不会被 Claude Code 加载。agentsys 同时有 `package.json`（npm 生态的 manifest）和 `.claude-plugin/plugin.json`（Claude Code 的 manifest），需要保持两者版本号同步。
