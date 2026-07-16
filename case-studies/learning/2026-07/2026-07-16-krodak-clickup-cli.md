# krodak/clickup-cli — 学习案例

**仓库**：https://github.com/krodak/clickup-cli
**Stars**：57（exemplar_published）| **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-16（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `manifest-discipline`, `security-gate`, `single-purpose`, `examples-driven`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
clickup-cli 是波兰工程师 Krzysztof Rodak 开发的 ClickUp 命令行工具（TypeScript 实现），同时附带 Claude Code skill 集，让 AI 代理可以通过 skill 来驾驶这个 CLI 完成任务管理工作。当前版本 1.38.3（审计时 1.25.2），含 4 个 skill（其中 1 个是审计后新增的 `using-clickup-cli`）。这是典型的「[NL 表皮 + 原生二进制核心](#nl-binary-hybrid)」架构：真正的能力在 TypeScript CLI 里，skill 是该 CLI 的 AI 驾驶手册。

- **创建时间**：2025 年
- **作者背景**：波兰工程师，ClickUp 重度用户，同时为 OpenCode 发布（多平台 skill）
- **获取方式**：`npm install -g @krodak/clickup-cli`（CLI）+ Claude Code plugin
- **生态位置**：ClickUp 任务管理与 Claude Code/OpenCode 的桥梁，小众但实用

### 1.2 架构剖析

```
krodak/clickup-cli
├── skills/
│   └── clickup-cli/SKILL.md    ← 主 skill（公开）：完整 CLI 参考手册
├── .agents/skills/              ← 内部 skill（不在主菜单中）
│   ├── releasing-clickup-cli/SKILL.md  ← 发布流程（internal: true）
│   ├── testing-clickup-cli/SKILL.md    ← 测试规范（internal: true）
│   └── using-clickup-cli/SKILL.md      ← 使用指南（新增）
├── .claude-plugin/
│   └── plugin.json              ← 含 name/version/author，但无 skills 枚举
├── src/
│   ├── index.ts                 ← CLI 入口
│   ├── api.ts                   ← ClickUp API 封装
│   ├── commands/                ← 20+ 命令实现（TypeScript）
│   │   ├── tasks.ts
│   │   ├── delete.ts
│   │   ├── comment.ts
│   │   └── ...
│   └── ...
├── package.json                 ← TypeScript CLI 包（npm 发布）
├── tsup.config.ts
└── eslint.config.mjs
```

- **文件类型分布**：4 个 SKILL.md（1 个公开 + 3 个内部）/ 1 个 [plugin.json](#manifest)（无 skills 枚举）/ 20+ TypeScript 命令文件 / package.json
- **编排关系**：主 skill（`clickup-cli/SKILL.md`）是独立的 AI 驾驶手册，无需引用其他 skill。`.agents/skills/` 里的 3 个内部 skill 标有 `metadata.internal: true`，是开发者维护工具，不对外暴露。
- **跨件契约**：版本号在 `plugin.json`（1.38.3）、`skills/clickup-cli/SKILL.md`（版本头）、`package.json`（1.38.3）三处同步，由 `releasing-clickup-cli/SKILL.md` 步骤 3 的发布脚本强制执行。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「skill 是 CLI 的 AI 驾驶手册」——CLI 本身不关心 AI，skill 专门为 AI 解释如何调用 CLI。两者独立演化：CLI 版本跟用户功能需求走，skill 跟 AI 最佳实践走。
- **解决什么问题**：ClickUp 有大量 CLI 命令和 flag，AI 直接调用容易选错命令或遗漏必要 flag；skill 提供了「预编译」的最佳实践路径，避免 AI 每次都从头探索。
- **Trade-off**：TypeScript CLI 是真正的能力核心，维护成本高（需要编译、测试、npm 发布）；而 skill 只是文档，维护成本低。这种分离让 AI 友好性不拖累 CLI 的迭代速度。
- **认知模型**：把 AI 视为「需要操作手册的新员工」，skill 是操作手册，CLI 是工具本身。新员工（AI）通过手册（skill）学会用工具（CLI），而不是靠猜测。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「NL 表皮 + 原生二进制核心」**——用编译型语言（TypeScript/Go/Rust）实现核心功能，打包为 CLI 二进制；NL skill 作为 AI 接口层，描述如何调用这个 CLI。两层独立演化，各自有自己的版本发布流程。

模式特征清单：
- 特征 1：核心功能在编译/打包的原生程序（TypeScript → `cup` binary），不在 NL 文件里
- 特征 2：SKILL.md 是「操作手册」，不包含任何业务逻辑
- 特征 3：内部 skill（`metadata.internal: true`）隐藏在 `.agents/skills/`，与公开 skill 分离
- 特征 4：版本号通过发布脚本（`releasing-clickup-cli/SKILL.md` 步骤 3）三处强制同步
- 特征 5：DELETE SAFETY、OUTPUT MODES 等关键操作在 SKILL.md 中有专门章节

### 2.2 适用场景
| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要系统级权限的桌面工具 | ✅ 高度适用 | CLI 可调用系统 API，NL 层只负责指令翻译 |
| 已有 CLI 工具需要 AI 化 | ✅ 适用 | 只需添加 skill 层，无需改造现有 CLI |
| 需要低延迟/高吞吐的数据处理 | ✅ 适用 | 编译型核心比 Python/Node.js 解释器快 |
| 纯知识类 skill（无实际工具调用） | ❌ 不适用 | 没有 CLI 时，引入编译型核心是过度设计 |

### 2.3 与其他架构对比
| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 表皮+原生二进制核心（本仓库） | krodak/clickup-cli | 性能、系统集成能力强，skill 轻量 | 需维护两套发布流程（npm + plugin）|
| 纯 NL skill 集 | czlonkowski/n8n-skills | 维护成本低，无构建步骤 | 受限于 NL 表达能力 |
| NL 表皮+解释型脚本 | kazukinagata/shinkoku | 无编译步骤，部署简单 | 不能调用系统级 API，性能一般 |

### 2.4 改进空间
1. **当前问题**：`plugin.json` 缺少 `skills` 枚举数组 **改进做法**：添加 `"skills": ["skills/clickup-cli"]`（`.agents/skills/` 内的 internal skill 不需要列入）**预期收益**：`claude plugin install` 可验证 skill 文件完整性，自动化工具可枚举暴露的 skill
2. **当前问题**：`name: clickup` vs 包名 `clickup-cli` 不一致 **改进做法**：在 `skills/clickup-cli/SKILL.md` 的 frontmatter 中添加注释 `# Short alias for the clickup-cli package`，或与 `releasing-clickup-cli` 同步，确保发布脚本检查这两者 **预期收益**：贡献者不会误以为这是另一个无关 skill
3. **当前问题**：生产依赖 `@inquirer/prompts`、`chalk`、`commander` 使用 `^` caret 范围 **改进做法**：通过 `package-lock.json`（若非 monorepo）或 `.npmrc` 的 `save-exact=true` 固定安装版本 **预期收益**：CI/CD 环境安装结果完全可复现

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-06 当时得分 **95/100**（4 skill，SECURITY CLEAR）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| .claude-plugin/plugin.json | 90 | 无 `entries`/`skills` 数组，自动化无法枚举 (-10) |
| .agents/skills/testing-clickup-cli/SKILL.md | 95 | "edge cases" 模糊 (-2) |
| .agents/skills/releasing-clickup-cli/SKILL.md | 97 | 无显著问题 |
| skills/clickup-cli/SKILL.md | 97 | 无显著问题 |

### 3.2 当时值得借鉴的模式
1. **`DELETE SAFETY` 专门章节** → 为什么好：`skills/clickup-cli/SKILL.md` 末尾有独立的 "DELETE SAFETY" 章节，明确列出所有不可逆操作及操作前的必要确认步骤。这防止 AI 「误操作」删除任务、评论等不可恢复的数据。原文：`skills/clickup-cli/SKILL.md:DELETE SAFETY` 章节 → 借鉴：凡是 skill 中涉及不可逆操作（删除、覆盖、发布），都单独列一节说明
2. **`OUTPUT MODES` 明确 TTY vs piped 行为** → 为什么好：CLI 在 TTY 终端和管道（`cup tasks | jq`）下输出格式不同（彩色 vs 纯文本），skill 明确声明了这个行为，防止 AI 在管道场景误解输出。原文：`skills/clickup-cli/SKILL.md:Output Modes` 章节 → 借鉴：当 skill 描述的工具有 TTY 敏感的输出行为时，在 skill 中明确说明
3. **内部 skill 用 `metadata.internal: true` 隐藏** → 为什么好：`releasing-clickup-cli` 和 `testing-clickup-cli` 是开发者维护工具，不应暴露给最终用户。通过 `metadata.internal: true` 和 `.agents/skills/` 目录将其与公开 skill 分离。原文：`.agents/skills/*/SKILL.md` 的 frontmatter → 借鉴：维护类/内部类 skill 用独立目录 + 特殊标记与公开 skill 区分
4. **三处版本强制同步** → 为什么好：`releasing-clickup-cli/SKILL.md` 步骤 3 里有同步脚本 (`scripts/sync-command-docs.ts`)，确保 `plugin.json`、`SKILL.md`、`package.json` 三处版本号同时更新。原文：`scripts/sync-command-docs.ts` → 借鉴：当版本号需要在多处声明时，引入同步脚本而非靠人工记忆

### 3.3 当时的缺陷
1. **问题**：`plugin.json` 缺 `skills` 数组（`-10 分` 最大单项扣分）**根本原因**：作者可能依赖 Claude Code 的文件系统发现机制（自动扫描 `skills/` 目录）而不是显式声明；这在 CLI 工具生态中很常见（package.json 也不需要手动列出所有文件）**自查**：我的 plugin.json 有没有 skills 枚举？
2. **问题**：生产依赖用 `^` caret 范围（3 个 LOW 安全发现）**根本原因**：npm 的 `^` 是默认行为，大多数 Node.js 开发者习惯用 `^` 允许次版本更新；但对 CLI 工具而言，次版本更新可能改变命令行 API，影响 skill 的调用说明**自查**：我的 package.json 生产依赖是否用了 `^` 范围？
3. **问题**：`name: clickup` vs 包名/目录名 `clickup-cli` 命名漂移 **根本原因**：作者希望 skill 名保持短（`/clickup` 比 `/clickup-cli` 好输入），但没有在代码中注释这个意图，造成歧义 **自查**：我的 skill `name` 字段和它的目录名/包名是否一致？不一致时是否有说明？

### 3.4 当时的优化机会
1. 在 `plugin.json` 添加 `"skills": ["skills/clickup-cli"]`（单行修改，消除 -10 扣分）
2. 在 `package.json` 把 3 个生产依赖的 `^` 去掉，或通过 `npm shrinkwrap` 锁定
3. 在 `skills/clickup-cli/SKILL.md` frontmatter 加注释说明 `name: clickup` 是 `clickup-cli` 的短别名

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| plugin.json 缺 skills 枚举 | `cat .claude-plugin/plugin.json` | **持续存在** — v1.38.3 仍无 skills 数组 | 历经 13 个次版本（1.25.2→1.38.3）未添加 |
| 生产依赖 caret 范围 | `grep "commander" package.json` → `"^15.0.0"` | **持续存在**（commander 从 `^14.0.3` 升级到 `^15.0.0`，仍是 caret）| 次版本锁定策略未变，但 commander 完成了一次主版本升级 |
| name drift（clickup vs clickup-cli） | `grep "^name:" skills/clickup-cli/SKILL.md` → `name: clickup` | **持续存在** | 未添加说明注释 |

### 4.2 架构演进
从 3 skill（v1.25.2：clickup-cli + releasing + testing）到 4 skill（v1.38.3：新增 `using-clickup-cli`）：

- **新增** `using-clickup-cli/SKILL.md`（位于 `.agents/skills/`）：入门使用指南，从 `releasing-clickup-cli/SKILL.md` 步骤 9 可见，作者现在将该 skill 同步到 OpenCode 的 `~/.config/opencode/skills/clickup/SKILL.md` 路径，实现多平台支持。
- **版本升级**：1.25.2 → 1.38.3（13 个次版本），说明 CLI 功能在持续扩展，但 skill 层只新增了 1 个 skill，保持了「CLI 核心演化快、NL 层更新慢」的分离节奏。

### 4.3 新增的可学习模式
- **多平台 skill 同步**：`releasing-clickup-cli/SKILL.md` 步骤 9 中包含把 skill 文件复制到 OpenCode 路径（`~/.config/opencode/skills/clickup/SKILL.md`），这是「一次写，多平台部署」的发布流程实践。与 agentsys 的跨平台维护 skill 思路相近，但更轻量（只是一个 cp 命令）。

---

## 五、校准

### 5.1 我已经在做对的
1. **bureau 的 4 个 skill 全部有清晰的 frontmatter**（name/description 字段），与 clickup-cli skill 的规范前置信息一致
2. **gstack 没有涉及 ClickUp/第三方任务系统的 skill**，没有重叠设计
3. **bureau 的 release 流程在 CLAUDE.md 中有说明**，与 `releasing-clickup-cli` 的发布 skill 设计理念一致（把发布流程本身文档化为 NL 指令）

### 5.2 挑战 / 验证
- **验证了的认知**：`metadata.internal: true` + 独立目录（`.agents/skills/`）是区分「内部维护 skill」和「用户使用 skill」的有效模式。我此前想过在 bureau 里加一个 `maintenance/SKILL.md` 但犹豫于是否该让用户看到——这个案例给出了答案：通过目录分离和 `internal` 标记解决。
- **认知更新**：skill 的 `name` 字段和实际命令名可以刻意不同（短别名），但需要在 frontmatter 里注释清楚意图，而不是让读者猜。这是一个易被忽视的「文档化意图」细节。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 plugin.json 是否有 skills 枚举
for repo in /tmp/my-repos/MarkQWu-bureau /tmp/my-repos/MarkQWu-gstack; do
  pj=$(find "$repo" -name "plugin.json" -path "*/.claude-plugin/*" 2>/dev/null | head -1)
  if [ -n "$pj" ]; then
    echo "$pj:"
    python3 -c "import json,sys; d=json.load(open('$pj')); print('  has skills/entries:', 'skills' in d or 'entries' in d)"
  fi
done
```
命中后：添加 `"skills": ["skills/<name1>", "skills/<name2>", ...]` 到 plugin.json。

```bash
# 检查我的 package.json 生产依赖 caret 范围数量
for pj in /tmp/my-repos/MarkQWu-gstack/package.json /tmp/my-repos/MarkQWu-bureau/package.json; do
  echo "$pj:"
  grep '"dependencies"' -A 999 "$pj" | grep '"\^' | wc -l
done
```
命中后（数量 > 0）：评估每个 caret 依赖是否为生产关键依赖；若是 CLI 工具或插件的核心依赖，考虑用 `npm shrinkwrap` 或 `.npmrc save-exact=true` 固定。

```bash
# 检查我的 skill name 字段与目录名是否一致
find /tmp/my-repos/MarkQWu-bureau/skills /tmp/my-repos/MarkQWu-gstack \
  -name "SKILL.md" | while read f; do
  dir=$(basename $(dirname "$f"))
  name=$(grep "^name:" "$f" | head -1 | sed 's/name: //')
  if [ "$dir" != "$name" ]; then
    echo "MISMATCH: dir=$dir name=$name file=$f"
  fi
done
```
命中后：确认是否为刻意的短别名；若是，在 frontmatter 加注释 `# Short alias for <full-name>`；若不是，对齐 name 与目录名。

### 6.2 灵感 → 实施路径

1. **想法**：给 bureau 的维护 skill 用 `.agents/skills/` 隐藏模式
   - **为何可行**：bureau 有 `lint/SKILL.md` 和 `guide/SKILL.md` 两个本质上是维护工具的 skill，不应暴露给最终用户
   - **第一步**：创建 `.agents/skills/` 目录，将 `lint/SKILL.md` 迁移到 `.agents/skills/bureau-lint/SKILL.md`，在 frontmatter 加 `metadata.internal: true`；更新 plugin.json 的 skills 枚举（去掉 lint）；约 15 分钟

2. **想法**：在 bureau/gstack 的重要 skill 中添加 "DESTRUCTIVE OPERATIONS" 专节
   - **为何可行**：gstack 的 `land-and-deploy/SKILL.md` 包含 `git push --force`、部署命令等高风险操作，类比 clickup-cli 的 DELETE SAFETY 章节
   - **第一步**：在 `land-and-deploy/SKILL.md` 末尾添加「不可逆操作警告」章节，列出：`--force push`、`npm publish`、`TestFlight 提交`，每条注明确认前必须执行的检查；约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 krodak/clickup-cli 的核心目的**：让 AI 代理能够通过 skill 驾驶 ClickUp CLI，实现任务管理的自动化

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 都是「工具 + skill」组合，skill 是工具的 AI 驾驶手册 | bureau 的工具层是简单脚本，clickup-cli 是完整的 TypeScript CLI | 高（发布流程 skill、内部 skill 模式）|
| MarkQWu/gstack | 中 | gstack 有类似「NL 描述 + 外部工具调用」的模式 | gstack 的 skill 更多是操作流程，clickup-cli 的 skill 更侧重 CLI 手册 | 中 |
| 其余仓库 | 低/无 | — | — | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| plugin.json 缺 skills 枚举 | `python3 -c "import json; d=json.load(open('bureau/.claude-plugin/plugin.json')); print('skills' in d)"` | **bureau 命中**：`False`，无 skills 数组 | 中 |
| package.json caret 范围生产依赖 | `grep '"\^' gstack/package.json` | **gstack 命中**：9 处 caret 范围 | 低（须评估各依赖类型）|
| skill name 与目录名不一致 | `grep "^name:" bureau/skills/*/SKILL.md` | **bureau 未命中**（name 字段与目录名一致）| 无 |

**命中后的具体行动建议**：
- `MarkQWu-bureau/.claude-plugin/plugin.json` → 添加 `"skills": ["skills/capture", "skills/compile", "skills/recall", "skills/review", "skills/scribe", "skills/guide"]`（不含 lint，迁移到内部 skill）→ 5 分钟可完成
- `MarkQWu-gstack/package.json` → 检查 9 个 caret 依赖中哪些是发布时的生产关键依赖（如 `@anthropic-ai/sdk`），将其固定 → 30 分钟

### 7.3 别人的更优方案

1. **领域**：发布流程 skill（releasing skill）
   - **本案例做法**：`.agents/skills/releasing-clickup-cli/SKILL.md` 包含 9 步发布流程（typecheck → lint → test → build → version sync → changelog → tag → publish → OpenCode 同步），hidden from 最终用户（路径：`.agents/skills/releasing-clickup-cli/SKILL.md`）
   - **我的项目现状**：bureau 的发布流程分散在 CLAUDE.md 的「开发规范」章节，没有独立的发布 skill
   - **如何借鉴**：创建 `.agents/skills/bureau-release/SKILL.md`，把发布检查清单（版本号同步、changelog 更新、git tag、npm publish）集中管理；diff 思路：从 CLAUDE.md 中提取发布相关章节迁移过去

2. **领域**：DELETE SAFETY 专节
   - **本案例做法**：`skills/clickup-cli/SKILL.md` 末尾有专门的 "DELETE SAFETY" 章节（路径：`skills/clickup-cli/SKILL.md`）
   - **我的项目现状**：gstack 的 `land-and-deploy/SKILL.md` 有 `--force push` 等高风险操作，但没有专门的"危险操作"警告章节
   - **如何借鉴**：在 `land-and-deploy/SKILL.md` 末尾添加 `## 不可逆操作` 章节，列出所有需要额外确认的操作

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：plugin.json skills 枚举约定
- **我的做法**（bureau）：bureau 虽然也缺 skills 数组，但 bureau 的 CLAUDE.md 明确记录了所有 skill 的名称和功能，形成了另一种形式的「人类可读清单」
- **本案例做法（弱在哪）**：clickup-cli 的 plugin.json 既无 skills 数组，CLAUDE.md 也无对应清单，只能靠目录扫描发现 skill；且 `-10` 的分值是本仓库最大的单项扣分
- **意义**：bureau 的 CLAUDE.md skill 清单是一个可指向的优势，即使 plugin.json 还没有 skills 数组，清单本身也降低了新贡献者的理解成本。

---

## 八、术语表

### <a name="nl-binary-hybrid"></a>NL 表皮 + 原生二进制核心
> 一种 Claude Code 插件架构：核心功能由编译型语言（TypeScript/Go/Rust 等）实现，打包为可直接运行的程序（CLI 命令行工具）；NL skill 只是「如何驾驶这个工具」的 AI 指令文档，不包含实际业务逻辑。类比：汽车（原生二进制）+ 驾驶手册（NL skill）。

### <a name="manifest"></a>plugin.json（manifest）
> Claude Code 插件的清单文件。声明插件的 name、version、author、homepage，以及可选的 skills 枚举数组（列出所有 skill 路径）。缺少 skills 数组时，Claude Code 需要扫描文件系统才能发现有哪些 skill，自动化工具无法直接枚举。

### <a name="caret-semver"></a>Caret 范围（`^`）
> npm 版本范围记法。`^8.3.0` 表示允许安装 `8.3.0` 到 `<9.0.0` 之间的任意版本（次版本和补丁版本可以自动更新）。对于 CLI 工具的生产依赖，次版本更新可能改变 API，导致 skill 的命令示例失效。

### <a name="metadata-internal"></a>metadata.internal: true
> 一个非标准但被部分插件使用的 SKILL.md frontmatter 字段。标记为 `internal: true` 的 skill 不对外暴露给最终用户，只供开发者/维护者调用。clickup-cli 用它区分「用户使用 skill」（`skills/`）和「维护工具 skill」（`.agents/skills/`）。
