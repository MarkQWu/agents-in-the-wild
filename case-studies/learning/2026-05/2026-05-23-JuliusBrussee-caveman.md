# JuliusBrussee/caveman — 学习案例

**仓库**：https://github.com/JuliusBrussee/caveman
**Stars**：22,505 | **来源**：upstream audit
**Audit 日期**：2026-04-17（历史快照）| **生成日期**：2026-05-23（基于当前 HEAD）
**主题标签**：`single-purpose`, `security-gate`, `manifest-discipline`, `fallback-chain`, `curl-pipe-bash-risk`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
caveman 是 JuliusBrussee 开发的 Claude Code 插件，解决一个简单却高价值的问题：**让 AI 用极度压缩的"穴居人语言"通信，把 token 消耗降低 ~75%**，同时保持完整技术准确性。22,505 stars 说明这个痛点打中了很多人。当前版本已大幅扩展：从最初的单一压缩 skill 演进为包含 cavecrew（子 agent 决策导引）、caveman-stats（token 统计）、caveman-compress（压缩 CLAUDE.md 等记忆文件）等 7 个 skill 的完整生态系统。支持 Claude Code、Cursor、Windsurf、Gemini 四个 IDE。

### 1.2 架构剖析
- **目录结构（当前 HEAD）**：
  ```
  caveman/
  ├── skills/              # 7 个 skill（caveman + 6 个扩展）
  │   ├── caveman/         # 核心：压缩通信模式
  │   ├── caveman-commit/  # 提交信息压缩
  │   ├── caveman-compress/ # 记忆文件压缩（含 Python scripts）
  │   ├── caveman-help/    # 快速参考卡片
  │   ├── caveman-review/  # 代码审查压缩
  │   ├── caveman-stats/   # token 用量统计
  │   └── cavecrew/        # 子 agent 决策导引
  ├── plugins/             # 其他 IDE 的适配层（含 compress 变体）
  ├── .cursor/skills/      # Cursor 自动同步副本
  ├── .windsurf/skills/    # Windsurf 自动同步副本
  ├── agents/              # 子 agent 定义
  ├── bin/                 # 编译后的可执行文件
  ├── src/                 # TypeScript 源码
  ├── tests/ + evals/      # 测试和评估
  ├── install.sh / install.ps1  # 安装脚本
  └── skills-lock.json     # skill 版本锁定文件
  ```
- **文件类型分布**：7 个 SKILL.md / 0 个 command / 若干 agent / 2 个 hook（activate + mode-tracker）/ 1 个 plugin.json
- **编排关系**：核心 `caveman/SKILL.md` 是主 skill，其他 skill 都是功能扩展，互相独立不相互调用。`cavecrew` 例外：它是"路由导引"skill，告诉主线程何时派生子 agent（investigator/builder/tester）。
- **跨件契约**：CI 同步机制确保 `skills/caveman/SKILL.md` 的内容在 `.cursor/`、`.windsurf/`、`caveman/`、`plugins/caveman/skills/caveman/` 等 5 个路径保持一致。`plugin.json` 声明了 2 个 [hook](#hook)（caveman-activate.js、caveman-mode-tracker.js）。`skills-lock.json` 锁定依赖的外部 skill 版本（含 SHA256 hash）。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：用极简语言实现等价语义传达。去掉冠词、填充词、礼貌用语，保留技术实质。不同场景有不同压缩强度（lite/full/ultra），给用户选择权。
- **解决什么问题**：Claude Code 会话 token 消耗巨大，长会话很快用完 context window。caveman 通过在 AI 端压缩输出文字（而非内容），让同样的会话可以持续更长。
- **做了什么 trade-off**：极度压缩输出对熟练用户体验好，但对新手不友好——这正是 `caveman-help` skill 和 `lite` 模式的来源，是对易用性的让步。
- **反映什么认知模型**：作者把 token 视为有限资源，把 AI 输出视为可压缩的信号，把读者（开发者）视为能容忍语法不完整但需要技术完整性的专家。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：「多 IDE 同步核心 + 扩展 Skill 星形布局」**

一个核心 skill（caveman）作为基础，通过 CI 同步到多个 IDE 适配路径，周围有独立的功能扩展 skill 形成星形。扩展 skill 互相独立，用户按需加载。

模式特征清单：
- **多 IDE 分发**：同一 SKILL.md 内容自动同步到 4 个 IDE 的各自配置路径
- **星形扩展**：核心 skill + N 个独立功能扩展，扩展之间无依赖
- **Hook 增强**：2 个 JS hook 在后台追踪状态（mode tracker），给 skill 提供运行时上下文
- **Skills-lock 版本控制**：`skills-lock.json` 锁定引用的外部 skill 的版本（含 hash），防止无声升级
- **安全优先的 hook 实现**：O_NOFOLLOW、原子写入、ANSI 剥离等安全工程实践嵌入 hook 代码

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 跨多个 AI IDE 的核心 skill 分发 | ✅ 高度适用 | 这正是本仓库的核心创新 |
| 有状态的 AI 模式切换（需要 hook 追踪状态） | ✅ 高度适用 | mode-tracker hook 专为此设计 |
| 纯文字处理 / token 优化 | ✅ 高度适用 | 原始用途 |
| 需要网络 API 调用的 skill | ❌ 不适用 | caveman 是无网络依赖的纯语言转换 |
| 内容创作 / 社交发布等场景 | ❌ 不适用 | 没有系统集成能力，不适合有副作用的操作 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 多 IDE 同步核心（本仓库） | caveman | 一套规则多处使用，CI 保证一致性 | 同步机制需要 CI，结构复杂，名称分裂问题 |
| 单一发布路径 | baoyu-skills | 简单直接，无同步负担 | 用户需要手动适配到不同 IDE |
| 纯钩子增强 | 无典型 | 最小侵入 | 钩子有安全风险 |

### 2.4 改进空间
1. **当前问题**：`caveman-compress/SKILL.md` 第 26 行的 `python3 -m scripts <absolute_filepath>` 没有 `cd` 到正确目录，若当前工作目录不是 skill 目录则报错。**改进做法**：参照同仓库 `plugins/caveman/skills/compress/SKILL.md` 的写法，改为"从包含本 SKILL.md 的目录执行：`cd <directory_containing_this_SKILL.md> && python3 -m scripts <absolute_filepath>`"。**预期收益**：skill 在任意 CWD 下都可以正确执行。
2. **当前问题**：`install.sh` 使用 `main` 分支 URL 下载 hook 文件，无 SHA256 校验。**改进做法**：发布 tagged release，把 REPO_URL 改为 `refs/tags/v<VERSION>`，并在下载后用 `sha256sum --check` 验证。**预期收益**：用户安装到经过验证的版本，main 分支推送不会影响既有安装。

---

## 三、过去审查发现（2026-04-17 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-17 当时得分 **92/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| caveman-compress/SKILL.md | 85 | 硬编码相对目录 `cd caveman-compress`；Process 步骤未用代码块 |
| plugins/caveman/skills/compress/SKILL.md | 88 | 占位符 `<directory_containing_this_SKILL.md>` |
| skills/compress/SKILL.md | 88 | name `compress` 与 canonical `caveman-compress` 不一致 |
| skills/caveman/SKILL.md（×5 副本） | 95 | 自动同步副本，内容优秀 |

### 3.2 当时值得借鉴的模式
1. **O_NOFOLLOW + 原子写入的 hook 安全实现** → `safeWriteFlag` 使用原子 temp+rename、0600 权限、symlink 检查。这是比大多数开源插件高 1-2 个档次的安全意识。→ 借鉴：自己写 hook 时参照这个实现，不要直接 `fs.writeFile()`
2. **ANSI 剥离 + 白名单验证防注入** → `caveman-statusline.sh` 剥离非 `[a-z0-9-]` 字符，`readFlag` 白名单校验模式值。防止 flag 文件内容注入终端控制序列。→ 借鉴：任何读取外部文件内容再展示的 skill，都要剥离控制字符
3. **VALID_MODES 白名单** → hook 里从环境变量读取模式值时，先对比 VALID_MODES 白名单再使用，注入变为无效。→ 借鉴：凡是从环境变量或用户输入读取枚举值的地方，显式白名单验证
4. **CI 同步多 IDE 副本** → 一处更新，CI 自动同步到 5 个路径，杜绝人工维护多份副本带来的漂移风险
5. **cavecrew 子 agent 决策导引** → 告诉主线程何时应该委托子 agent（investigator/builder/tester），而不是在主线程里做所有事，减少 token 浪费。

### 3.3 当时的缺陷
1. **问题：caveman-compress 的 CWD 假设（硬编码 `cd caveman-compress`）** → 为什么失败：skill 假设执行时 CWD 是仓库根目录，但用户可以从任意目录触发，导致 `cd caveman-compress` 找不到目录而静默失败。根本原因是写 skill 时只在自己的开发环境测试，没有考虑其他安装路径。→ 自查：我的 skill 里有没有任何相对路径的 `cd` 命令？
2. **问题：install.sh 下载 hook 无完整性校验** → 为什么失败：`curl -fsSL` 下载 JS 文件到 `~/.claude/hooks/` 没有验证 SHA256，如果 main 分支被篡改，用户静默安装了恶意 hook。虽然实际发生概率低，但攻击面真实存在。根本原因是开发者通常先考虑功能，安全加固是事后补丁。→ 自查：我有没有在 install 脚本里下载可执行文件？
3. **问题：compress skill 名称分裂（source 叫 `caveman-compress`，plugin 发布叫 `compress`）** → 为什么失败：两个名称在不同路径下注册，可能被 skill loader 视为两个独立 skill，或者因为名称短而被其他 `compress` skill 覆盖。根本原因是"plugin 里用短名更方便"的设计意图没有文档说明，未来维护者可能会无意中"修复"这个看似的不一致。→ 自查：我有没有同一个 skill 在不同路径下有不同名称？

### 3.4 当时的优化机会
1. `caveman-compress/SKILL.md` 的 Process 步骤改为：从 SKILL.md 所在目录执行 python3 命令
2. `install.sh` 增加 sha256 校验步骤，并发布 `hooks/checksums.sha256` 文件
3. 在 CLAUDE.md 里明确记录 `compress` vs `caveman-compress` 名称差异是有意为之（plugin 短名 vs 源码全名）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| caveman-compress CWD 假设（硬编码目录名） | `grep -n "cd\|python3 -m scripts" skills/caveman-compress/SKILL.md` | **部分改善**：已改为"从 scripts/ 所在目录执行，找不到时搜索 `scripts/__main__.py`"，但仍无明确 `cd` 命令，执行路径依赖 AI 推断 | 比原来好，但仍可能在非标准 CWD 下推断失败 |
| install.sh 下载无完整性校验 | `grep -n "sha256\|verify\|checksum" install.sh` | **仍存在**（无任何 sha256 校验），main 分支 URL 未改 | 安全风险仍在，作者未优先处理 |
| main 分支 URL（不固定版本） | `grep -n "main" install.sh \| head -5` | **仍存在**（第 8-9 行注释里仍引用 main 分支 URL） | 用户按文档做 curl-pipe-bash 安装时没有版本固定保证 |

### 4.2 架构演进
当时审计时 14 个 NL 工件，现在 skills/ 目录下 7 个 SKILL.md（去掉了当时审计的多个 IDE 副本计数方式，因为副本仍存在但在 `.cursor/`、`.windsurf/` 下）。**最大变化是新增了**：
- `skills/cavecrew/SKILL.md`：子 agent 决策导引 skill——从单模式扩展为多 agent 协作
- `skills/caveman-stats/SKILL.md`：真实 token 用量统计——从"声称省 75%"到"实测省多少"
- `skills-lock.json`：skill 依赖版本锁定（含 SHA hash）——成熟的依赖管理机制
- `bin/` 目录：预编译二进制——说明仓库从纯 NL 走向了 [NL 表皮 + 原生二进制核心](#nl-表皮原生二进制核心) 方向

这说明作者在审计后的几周里重点做了"从概念验证到生产工具"的演进：加入可验证性（stats）、可组合性（cavecrew）、可靠性（skills-lock）。

### 4.3 新增的可学习模式
1. **`skills-lock.json` 外部 skill 版本锁定**：锁定引用的外部 skill 的 `source`、`skillPath` 和 `computedHash`，防止上游悄悄改 skill 内容影响本项目——类似 `package-lock.json` 的哲学应用到 NL skill 层
2. **caveman-stats 实测数据反馈**：一个 skill 专门读取 Claude Code 会话日志，报告当前会话的真实 token 用量和省了多少——把"大概省 75%"变为可验证的具体数字，这是"自我可观测性"模式

---

## 五、校准

### 5.1 我已经在做对的
1. **echo-sleuth 的安全意识**：skills/jsonl-core/SKILL.md 里明确约束不直接 eval 任何 JSONL 内容，只做结构化解析——和 caveman 的 VALID_MODES 白名单思路相同
2. **drama-workshop-skills 没有 install 脚本下载可执行文件**：安装用 `claude plugin install`，不走 curl-pipe-bash 模式——避免了 caveman 审计中发现的 install.sh 安全问题
3. **drama-workshop-skills 的版本一致性 CI 检查**：和 caveman 新增的 skills-lock 机制在目标上一致（防版本漂移）

### 5.2 挑战 / 验证
这次案例**挑战**了"开源项目的安全问题被发现后会很快修复"的假设。audit 发现 install.sh 无完整性校验（Medium 级）后 1 个月，作者没有修复这个问题。这反映了一个现实：**功能开发（新增 cavecrew、stats）优先级总是高于安全加固**，除非有用户投诉。对我自己的项目同样适用：要主动维护安全清单，不能等到有人报告。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 skill 有没有假设 CWD 的相对路径 cd 命令
grep -rn "^cd \|&&.*cd\|cd.*&&" /tmp/my-repos/MarkQWu-*/*/SKILL.md 2>/dev/null
# 命中后：将相对 cd 改为"从本 SKILL.md 所在目录执行"的显式描述
```

```bash
# 检查我的 install 脚本有没有 curl 下载可执行文件
grep -rn "curl.*|.*sh\|wget.*|.*sh" /tmp/my-repos/MarkQWu-*/install*.sh 2>/dev/null
# 命中后：如果有 curl-pipe-bash 模式，添加 SHA256 校验步骤
```

```bash
# 检查是否有相同 skill 在不同路径下有不同 name 值
find /tmp/my-repos/MarkQWu-* -name "SKILL.md" | xargs grep -h "^name:" 2>/dev/null | sort | uniq -d
# 命中后：确认是否是有意为之（plugin 短名 vs 源码全名），若无意则统一
```

### 6.2 灵感 → 实施路径

1. **想法**：为 drama-workshop-skills 添加 `skills-lock.json` 式的依赖版本锁定
   - **为何可行**：drama-workshop 目前依赖 xiaolai 插件市场上的外部资源，如果上游悄悄改了，本项目行为会变
   - **第一步**：查看 `release-manifest.json` 是否已有版本 hash，若无，添加一个 skills-lock 记录当前引用的外部 skill 版本——约 20 分钟

2. **想法**：为 echo-sleuth-for-claude 添加 caveman 风格的 token 使用量自报告
   - **为何可行**：echo-sleuth 的核心价值之一是"减少 token 浪费"，如果能在命令结束时报告"本次会话节省了 N tokens"，增加可信度
   - **第一步**：研究 caveman-stats/SKILL.md 的读取日志逻辑，参考写一个简单的统计步骤——约 45 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 JuliusBrussee/caveman 的核心目的**：降低 AI 会话 token 消耗，提升 context window 利用效率
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为 token/memory 效率工具；同以 Claude Code 会话为操作对象 | echo-sleuth 是会话**分析**，caveman 是输出**压缩**；目标相似但机制不同 | 高 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code skill；同关注 AI 输出质量 | 完全不同领域 | 低 |
| MarkQWu/claude-for-legal | 低 | 同为 skill 集合 | 无直接关联 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| Skill 脚本 CWD 假设（相对目录 cd） | `grep -rn "cd " /tmp/my-repos/MarkQWu-*/scripts/*.py 2>/dev/null` | drama-workshop-skills：`scripts/export_docx.py` 使用相对导入，假设从特定目录执行 | 中 |
| 无完整性校验的脚本下载 | `grep -rn "curl\|wget" /tmp/my-repos/MarkQWu-*/install*.sh 2>/dev/null` | drama-workshop-skills：`install.sh` 使用 `claude plugin install` 而非 curl 直接下载——无此问题 | 无 |

**命中后的具体行动建议**：
- `drama-workshop-skills/scripts/export_docx.py` 的相对导入→ 在 script 顶部加 `os.chdir(os.path.dirname(os.path.abspath(__file__)))` 确保从脚本所在目录执行→ 约 10 分钟

### 7.3 别人的更优方案

1. **领域**：Hook 安全实现（O_NOFOLLOW、原子写入、ANSI 剥离）
   - **本案例做法**：`hooks/caveman-activate.js` 用 `safeWriteFlag()` 实现原子写入，`caveman-statusline.sh` 剥离非 `[a-z0-9-]` 字符防止终端注入 → `hooks/caveman-activate.js`
   - **我的项目现状**：`echo-sleuth-for-claude/scripts/` 里的 shell 脚本没有同等级别的安全加固（直接读文件后输出，未做控制字符过滤）
   - **如何借鉴**：在 echo-sleuth 的 `scripts/extract-messages.sh` 等输出脚本末尾，过滤掉 ANSI 控制字符再输出（`sed 's/\x1b\[[0-9;]*m//g'`）

2. **领域**：`skills-lock.json` skill 依赖版本锁定
   - **本案例做法**：`skills-lock.json` 记录 `computedHash` 防止上游 skill 静默变更 → repo 根目录 `skills-lock.json`
   - **我的项目现状**：`drama-workshop-skills` 和 `echo-sleuth` 无 skill 依赖锁定机制
   - **如何借鉴**：为引用外部 skill 的项目创建 `skills-lock.json`，记录引用时的 commit hash 或文件 hash

### 7.4 反向：我的项目做得比他们好的地方

- **领域**：安全安装方式（`claude plugin install` vs curl-pipe-bash）
- **我的做法**：`drama-workshop-skills` 和 `echo-sleuth` 都通过 `claude plugin install` 安装，不使用 curl 直接下载可执行脚本
- **本案例做法（弱在哪）**：`install.sh` 文档里推荐 `curl -fsSL ... | bash` 安装方式，并下载 JS hook 文件到本地，这是安全审计里的 Medium 风险项
- **意义**：选择 `claude plugin install` 作为分发方式是一个安全优势，未来如有 PR 机会，可以向这类仓库建议优先使用 plugin install 而非 curl 安装

---

## 八、术语表

### <a name="hook"></a>hook
> Claude Code 里的钩子：在特定事件发生时（如工具执行前后、会话开始时）自动运行的脚本。caveman 的 hook 在 Claude Code 会话开始时激活压缩模式（caveman-activate.js）并追踪当前模式（caveman-mode-tracker.js）。

### <a name="nl-表皮原生二进制核心"></a>NL 表皮 + 原生二进制核心
> SKILL.md 负责意图解析（NL 表皮），核心执行逻辑由编译后的二进制或 TypeScript 脚本完成（原生核心）。caveman 的 `bin/` 目录里的预编译文件是这个方向的信号。

### <a name="curl-pipe-bash"></a>curl-pipe-bash
> 指 `curl <URL> | bash` 这种安装方式：直接从网络下载脚本并执行。便利但有安全风险：下载内容可能被篡改，且用户没有机会检查就直接执行了。caveman 的 SECURITY.md 审计中这是一个 Medium 风险项。

### <a name="O_NOFOLLOW"></a>O_NOFOLLOW
> 操作系统文件打开标志，表示"不跟随符号链接"。用于防止攻击者创建一个符号链接指向敏感文件，然后诱导程序"覆写"该符号链接目标——加了 O_NOFOLLOW 后，操作系统发现目标是 symlink 就拒绝打开。

### <a name="skills-lock"></a>skills-lock.json
> caveman 引入的依赖锁定文件，记录引用的外部 skill 的来源、路径和内容 hash。类似 npm 的 package-lock.json：确保你引用的 skill 版本不会在你不知情的情况下发生变化。
