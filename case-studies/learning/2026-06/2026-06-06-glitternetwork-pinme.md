# glitternetwork/pinme — 学习案例

**仓库**：https://github.com/glitternetwork/pinme
**Stars**：3188 | **来源**：xiaolai upstream
**Audit 日期**：2026-05-04（历史快照）| **生成日期**：2026-06-06（基于当前 HEAD）
**主题标签**：`security-gate`, `vague-quantifier`, `curl-pipe-bash-risk`, `single-purpose`, `template-design`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

PinMe 是一个 CLI/npm 包，为 [IPFS](#ipfs)（星际文件系统）提供 API 层，让开发者可以通过云服务平台进行去中心化的文件固定（pinning）。配套的 Claude Code 插件提供了 5 个专项技能（skill），覆盖文件存储、分享预览生成、邮件发送、LLM 模型配置和身份验证等集成场景。整个项目以 TypeScript/Node.js 为技术核心，同时以 `pinme` 命令发布在 [npm](#npm) 上供用户安装使用。

关键事实：
- **5 个专项 skill + 1 个 CLAUDE.md**：每个 skill 覆盖一个集成领域，职责边界清晰
- **API 统一基准**：所有 skill 均指向 `https://pinme.cloud`，没有跨文件 URL 不一致的问题
- **双语 skill 正文**：`pinme-auth/SKILL.md` 使用中英双语正文，是面向中文开发者社区的示范
- **安全状态：BLOCKED**：`build.js` 存在将所有环境变量注入构建产物的 HIGH 级别安全漏洞

### 1.2 架构剖析

```
pinme/
├── CLAUDE.md                         ← 项目上下文（95/100；明确指出 rollup.config.js 已废弃）
├── build.js                          ← 构建脚本（SECURITY ISSUE：全量 env 注入）
├── rollup.config.js                  ← 废弃，CLAUDE.md 明确标注勿用
├── package.json                      ← npm 包（prepublishOnly → build.js）
├── skills/
│   ├── pinme/SKILL.md                ← 主 skill：Cloudflare Workers + 全栈架构指南（92/100）
│   ├── pinme-auth/SKILL.md           ← 认证集成（95/100；中英双语）
│   ├── pinme-email/SKILL.md          ← 邮件 API（95/100）
│   ├── pinme-llm/SKILL.md            ← LLM 模型配置（95/100；含陈旧 model ID）
│   └── pinme-share/SKILL.md          ← HTML/CSS 分享页生成（86/100；7 个模糊量词）
└── example/
    ├── supabase/package.json         ← 示例项目（Low: 所有依赖使用 ^ 未固定版本）
    ├── pinme-blog/                   ← 博客示例
    └── docs/                         ← 文档
```

- **文件类型**：5 个 skill（NL 制品核心）、1 个 CLAUDE.md、构建脚本与示例项目
- **编排关系**：`pinme/SKILL.md` 作为「路由器」处理全栈架构，专项 skill 为各领域提供深度指南，层级分明
- **跨件契约**：API base URL（`https://pinme.cloud`）在所有 skill 中保持一致，但主 skill 未引用 pinme-auth → 用户可能不知道身份验证有专门 skill

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「一 skill 一领域」——每个 skill 文件只负责一个集成方向，不混合邮件和认证，不把 LLM 配置混入分享页逻辑
- **解决什么问题**：IPFS 本身 API 复杂，PinMe 的云 API 层封装了底层细节，Claude Code skill 进一步让 AI 知道「如何用 PinMe 的 API 构建功能」
- **做了什么 trade-off**：细粒度 skill 分包（5 个文件）vs 维护成本（email 逻辑在主 skill 和 pinme-email 中重复出现）
- **反映什么认知模型**：作者理解「领域专项化」的价值——用户只需要邮件功能时只加载 pinme-email，不必加载整个主 skill

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名**：「技能分层分包」（skill domain decomposition）

特征清单：
- **1 个主 skill 做总览**：`pinme/SKILL.md` 涵盖平台全景和全栈架构入口
- **N 个专项 skill 做纵深**：`pinme-auth`、`pinme-email`、`pinme-llm`、`pinme-share` 各自只管一个领域
- **每个 skill 可独立安装和使用**：用户按需加载，不需要把全部 skill 都装上
- **API 基准统一**：所有 skill 共用同一个 base URL，不存在「哪个才是对的」的歧义

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 多领域集成平台（云存储、邮件、认证、AI） | ✅ 高度适用 | 各领域 API 差异大，拆分后每个 skill 更聚焦，token 消耗更低 |
| 单一 API 的简单封装 | ❌ 过度设计 | 3 个端点以内用一个 SKILL.md 就够，拆成多文件反而难以维护 |
| 面向中文开发者社区的插件 | ✅ 适用 | pinme-auth 的双语正文证明了中英双语可以共存且不影响质量评分 |
| 需要高频更新内容的领域 | ⚠️ 谨慎 | 分包后每次 API 变更需要同步修改多个文件，维护负担高于单文件 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 技能分层分包（本仓库） | glitternetwork/pinme | 职责清晰、按需加载、token 高效 | 跨 skill 的逻辑容易重复，主 skill 未引用专项 skill 时用户难以发现 |
| 单一大 SKILL.md | shanraisshan/claude-code-best-practice | 内容完整、一次加载全覆盖 | 文件臃肿、AI 需要处理大量无关上下文 |
| 命令 + skill 混合 | jarrodwatts/claude-hud | 命令做引导、skill 做知识、职责完全分离 | 需要设计命令和 skill 的边界，初学者容易混淆 |

### 2.4 改进空间

1. **当前问题**：`pinme/SKILL.md` 未在 scope notes 中引用 `pinme-auth`  
   **改进做法**：在主 skill 末尾加「See Also」节，列出所有专项 skill 的路径和适用场景  
   **预期收益**：用户加载主 skill 后就能发现所有配套 skill，不会因为「不知道有 pinme-auth」而手写认证逻辑

2. **当前问题**：`handleSendEmail` 逻辑在 `pinme/SKILL.md` 和 `pinme-email/SKILL.md` 中同时存在  
   **改进做法**：主 skill 只保留 `import` 级别的调用示例，完整实现移至 pinme-email  
   **预期收益**：消除两个真相来源（two sources of truth），维护时只需改一处

3. **当前问题**：`pinme-llm/SKILL.md` 示例中包含 `openai/gpt-5.2`（陈旧 model ID）  
   **改进做法**：改为 `openai/gpt-4o` 或使用占位符 `<YOUR_MODEL_ID>` 并注明「请替换为当前最新模型 ID」  
   **预期收益**：示例代码不会因 model ID 失效而导致用户困惑

---

## 三、过去审查发现（2026-05-04 历史快照）

### 3.1 当时质量评分表

整体 NL 评分 **93/100**，Security: **BLOCKED**（High×1, Medium×1, Low×1）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `skills/pinme-share/SKILL.md` | 86 | 7 个模糊量词："relevant"、"clean"、"restrained"、"sensible"、"short"、"unrelated"、"valuable" |
| `skills/pinme/SKILL.md` | 92 | 4 个模糊量词："simple"、"some"、"clear"、"truly needed"；缺少 scope notes 引用兄弟 skill |
| `CLAUDE.md` | 95 | 内容完整，无问题 |
| `skills/pinme-auth/SKILL.md` | 95 | 无问题；中英双语正文 |
| `skills/pinme-email/SKILL.md` | 95 | 无问题 |
| `skills/pinme-llm/SKILL.md` | 95 | 陈旧 model ID（openai/gpt-5.2），仅作信息提示 |

**安全状态**：BLOCKED——HIGH 1 个（build.js 全量 env 注入 + prepublishOnly 链路）、Medium 1 个（SECRET_KEY 显式注入）、Low 1 个（示例项目未固定 semver）。

### 3.2 值得借鉴的模式

1. **`pinme-auth/SKILL.md` 的双语正文**：该 skill 正文同时包含中英文说明，NLPM 审查不将双语内容视为违规，评分仍为 95/100。这证明了中英双语 skill 是**完全合规的**实践——面向中文开发者社区时不必牺牲质量分数来照顾两种语言

2. **API base URL 统一**：5 个 skill 全部使用 `https://pinme.cloud` 作为 base，没有一个 skill 使用不同的 URL 或旧版 URL。这是「多 skill 套件保持一致性」的教科书示例——跨文件契约靠的是纪律，不是工具链强制

3. **CLAUDE.md 的废弃声明**：CLAUDE.md 明确写明「构建请用 build.js，不要用 rollup.config.js（已废弃）」，这防止了 AI 在看到两个构建文件时产生歧义。主动把「过时文件为什么还在」的原因记录在 CLAUDE.md 里，是值得复制的信息卫生做法

### 3.3 当时的缺陷

1. **HIGH 安全漏洞（最重要）**：`build.js` 第 5-7 行包含 `for (const key in process.env)` 循环，将构建机器上的**所有环境变量**注入到 [esbuild](#esbuild) 的 `define` 配置中；同时 `package.json` 的 `prepublishOnly` 脚本会在 `npm publish` 时自动触发 build.js  
   **为什么危险**：这是一个典型的**供应链攻击**载体——CI 机器上通常有 `NPM_TOKEN`、`GITHUB_TOKEN`、数据库密码等敏感环境变量，运行 `npm publish` 时这些 secret 会被 esbuild 以 `process.env.XXX = "明文值"` 的形式硬编码到 `dist/index.js` 里，而 `dist/index.js` 会随 npm 包公开发布。任何安装了这个 npm 包的人都可以在 `node_modules` 里找到这些 CI secret。这是一个「通过 npm 包泄露 CI 凭据」的完整攻击链  
   **自查**：我的构建脚本有没有用 `process.env` 的枚举或全量注入？

2. **Medium 安全漏洞**：`build.js` 第 9-10 行显式注入 `process.env.SECRET_KEY`——即使修复了全量 env 注入，这个字段也单独硬编码进了构建产物  
   **根本原因**：开发者可能把「把密钥注入到客户端代码」当成了常规做法，没有意识到 `dist/index.js` 发布后对全世界可见

3. **模糊量词（vague quantifier）问题**：`pinme-share/SKILL.md` 里有 7 个模糊量词，其中「clean」「restrained」「sensible」是**设计审美语言**——这类词在面向人类设计师时是有意义的描述，但对 AI 来说缺乏操作性，是一种特殊的模糊量词陷阱：用设计美学词汇代替具体的视觉规则

### 3.4 当时的优化机会

1. 修复 `build.js`：将 `for (const key in process.env)` 改为显式白名单，只注入业务需要的公开配置键
2. 删除或隔离 `process.env.SECRET_KEY` 注入——SECRET_KEY 不应该进入 npm 发布产物
3. 将 `pinme-share` 的 7 个模糊量词替换为可量化的设计规则（例：「clean」→「单页不超过 3 种字体、2 种主色」）
4. 在 `pinme/SKILL.md` 添加 scope notes，列出所有专项 skill 的引用路径

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷状态表格

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `build.js` 全量 env 注入（HIGH） | `grep "for.*in.*process.env" build.js` → 第 5-8 行命中 | **PERSISTS（CRITICAL）** | npm 发布时 CI secret 仍会被烘焙进 dist/index.js，供应链风险未消除 |
| `build.js` SECRET_KEY 显式注入（Medium） | `grep "SECRET_KEY" build.js` → 第 13 行命中 | **PERSISTS** | 即使修复了全量注入，SECRET_KEY 仍会单独泄露 |
| `pinme-share` 模糊量词（7 个） | `grep -c "relevant\|clean\|restrained\|sensible\|short" skills/pinme-share/SKILL.md` → 10 命中 | **PERSISTS（likely）** | 模糊量词仍在，评分提升空间未兑现 |
| example 未固定 semver（Low） | `grep "\^" example/supabase/package.json` | 未再验证 | 低风险，但依赖漂移风险仍存在 |

**重点**：距审查已过去一个月（2026-05-04 → 2026-06-06），HIGH 安全漏洞**仍未修复**。由于安全状态为 BLOCKED，NLPM auditor-contribute 工作流程拒绝向此仓库发起 PR——这是安全门控机制的直接体现。

### 4.2 架构演进

对比审查快照到当前 HEAD，架构结构没有实质变化：5 个 skill + CLAUDE.md 的格局保持不变，`build.js` 的安全问题依然存在。这与 jarrodwatts/claude-hud 的模式相似——**NL 制品层静止，代码层问题未受关注**。

### 4.3 新增模式

无新增可学习模式。主要变化缺失：高分 skill 的结构稳定，低分 skill（pinme-share）的模糊量词也未修复——说明作者目前没有把 NLPM 质量分数作为迭代信号。

---

## 五、校准

### 5.1 我已经在做对的

1. **单文件单职责意识**：echo-sleuth 的 4 个 skill（memory-management、git-mining、experience-synthesis、jsonl-core）与 pinme 的 5 个 skill 结构一致，都做到了领域拆分
2. **CLAUDE.md 提供项目上下文**：我的项目有 CLAUDE.md，pinme 的 CLAUDE.md 95/100 说明只要写出关键约定就能得高分，这个投入是值得的
3. **API base URL 纪律**：我理解跨文件保持 URL 一致性的重要性

### 5.2 挑战 / 验证

本案例挑战了一个假设：「安全问题会被作者迅速修复，尤其是在有公开 audit 记录之后。」

结果是：**一个月过去，HIGH 级别的 npm 供应链安全漏洞仍然存在**。这说明：

1. 安全发现到修复之间存在认知断层——作者可能不知道 audit 结果，也可能不理解这个漏洞的严重性
2. NLPM 的安全门控（BLOCKED 状态 → 不发 PR）在此案例中发挥了预期作用，但「不发 PR」本身无法驱动作者修复——修复需要主动告知

对我自己的启示：如果我的构建脚本里有任何 `process.env` 的使用，**都应该用白名单模式**，而不是全量枚举。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的构建脚本中是否有全量 env 枚举注入（供应链漏洞模式）
grep -rn "for.*in.*process\.env\|Object\.keys(process\.env\|Object\.entries(process\.env" \
  ~/projects/ --include="*.js" --include="*.ts" --include="*.mjs" | head -20
```
命中后怎么办：把全量枚举改为显式白名单——只注入明确需要在运行时可读的公开配置键（如 `process.env.PINME_API_URL`），绝不注入包含 SECRET/TOKEN/PASSWORD/KEY 的键。

```bash
# 检查构建脚本中是否有显式的敏感变量注入
grep -rn "SECRET_KEY\|API_SECRET\|PRIVATE_KEY\|NPM_TOKEN\|GITHUB_TOKEN\|DATABASE_URL" \
  ~/projects/ --include="build.js" --include="*.config.js" --include="*.config.ts" | head -20
```
命中后怎么办：从 esbuild/rollup/webpack 的 `define` 配置中移除该键；如果运行时真的需要 secret，改用服务端注入或环境变量运行时读取，不要硬编码进 npm 发布产物。

```bash
# 检查我的 skill 文件中的模糊量词（对 AI 没有操作意义的设计美学词汇）
grep -rn "\brelevant\b\|clean\|restrained\|sensible\|truly needed\|simple\|some\b" \
  ~/.claude/skills/ --include="*.md" | head -20
```
命中后怎么办：把「clean」替换为可量化规则（「最多 3 种字体，主色不超过 2 种」）；把「relevant」替换为「符合以下条件之一的」并列出具体条件；把「simple」替换为「不超过 N 步」或「不使用 X 技术」。

### 6.2 灵感

1. **想法**：为 echo-sleuth 的 skill 套件添加 scope notes，从主 skill 引用 3 个专项 skill  
   - **为何可行**：pinme 案例暴露了「主 skill 不引用专项 skill → 用户不知道有 pinme-auth」的问题，echo-sleuth 的主 skill 也可能有同样问题  
   - **第一步**：打开 `skills/experience-synthesis/SKILL.md`，在末尾加「See Also」节，列出 git-mining 和 memory-management 的路径，约 10 分钟

2. **想法**：建立构建脚本安全自查 checklist，固化到我的项目 CLAUDE.md 里  
   - **为何可行**：pinme 的 HIGH 漏洞模式（`for key in process.env` + `prepublishOnly`）是可检测的，在 CLAUDE.md 里写明「每次修改 build.js 后检查以下内容」可以预防  
   - **第一步**：在相关项目的 CLAUDE.md 里加「构建脚本安全规则」节，约 15 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度（这个仓库跟我的哪个项目最相似？）

- **本案例 glitternetwork/pinme 的核心目的**：面向特定云 API 平台的多领域 Claude Code skill 套件

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 中 | 同为多 skill Claude Code 插件，有相似的领域拆分结构 | echo-sleuth 是分析工具，pinme 是 API 集成套件 | 高 |
| MarkQWu/claude-for-legal | 中 | 同为领域特定 NL skill 套件，多文件多领域 | claude-for-legal 是法律知识型，pinme 是 API 调用型 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 同为 Claude Code skill | 创作类 skill 与 API 集成 skill 差异较大 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| skill 中出现模糊量词 "relevant" | `grep -rn "\brelevant\b" ~/projects/echo-sleuth-for-claude/skills/` | **命中 2 次**（已确认：echo-sleuth skills 中出现 "relevant"） | 中 |
| 主 skill 未引用专项 skill 的 scope notes | 手动检查各 skill 末尾是否有 See Also / scope notes | 需人工核对 | 中 |
| 构建脚本全量 env 枚举 | `grep "for.*in.*process.env" ~/projects/*/build.js` | 无已知 TypeScript 构建项目，可能 N/A | 视情况 |

**针对 echo-sleuth 的"relevant"命中的具体行动**：
- 找到 2 处"relevant"在哪个 skill 文件的哪一行
- 分析上下文：「relevant」用来描述什么？是「相关的记忆」还是「相关的 git 提交」？
- 替换为可操作的标准：「满足以下条件之一的」+具体条件列表，约 20 分钟

### 8.3 别人的更优方案（值得借鉴的）

1. **领域**：中英双语 skill 正文  
   - **本案例做法**：`pinme-auth/SKILL.md` 使用中英双语正文，评分 95/100，不受惩罚  
   - **我的项目现状**：echo-sleuth 和 claude-for-legal 的 skill 全部为英文正文  
   - **如何借鉴**：对于明确面向中文开发者的 skill（如 echo-sleuth 的 `memory-management/SKILL.md`），可以在关键步骤下方补充中文说明，不必担心影响质量评分

2. **领域**：CLAUDE.md 中的废弃声明  
   - **本案例做法**：CLAUDE.md 明确标注 `rollup.config.js` 已废弃，防止歧义  
   - **我的项目现状**：如果项目里有旧版脚本或废弃文件，未必在 CLAUDE.md 里注明  
   - **如何借鉴**：每次废弃一个文件时，在 CLAUDE.md 里加一行：「`[filename]` 已废弃，原因是 X，请使用 Y」，避免 AI 在看到两个类似文件时选错

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：安全构建实践  
- **我的做法**：echo-sleuth 不涉及 esbuild 发布流程，claude-for-legal 是纯 NL 制品，不存在把 secret 注入 npm 产物的风险  
- **本案例弱点**：pinme 的 HIGH 漏洞本质上是「不应该存在的功能」——`for key in process.env` 全量注入没有任何业务必要性，是开发者对 esbuild `define` 配置的误用  
- **意义**：「没有构建步骤」本身就是安全优势。纯 NL 制品套件（如 claude-for-legal、echo-sleuth）不存在供应链注入风险，这个优势容易被忽视

---

## 八、术语表

### <a name="ipfs"></a>IPFS（InterPlanetary File System，星际文件系统）
> 一种去中心化的文件存储协议。传统文件存储靠的是「服务器地址 + 文件路径」（location-based），IPFS 用的是「文件内容的哈希值」（content-based）——同一份文件在全网只有一个唯一 ID，任何存有该文件的节点都能响应请求。「Pinning」（固定）是指主动告诉某个 IPFS 节点「请一直保留这个文件，不要垃圾回收它」。PinMe 就是把 IPFS pinning 操作包装成简洁的云 API，让开发者不用自己跑 IPFS 节点。

### <a name="esbuild"></a>esbuild
> 用 Go 编写的极速 JavaScript 打包器，比 Webpack/Rollup 快 10-100 倍。它的 `define` 配置项允许在打包时把特定字符串（如 `process.env.NODE_ENV`）替换为字面值，这样生产代码里就不会出现 `process.env` 的动态查找。pinme 的漏洞就发生在这里：开发者用了一个循环把**所有** env 变量都传给 `define`，结果 CI 上的任何 secret 都被当成「要替换的字符串」烘焙进了发布产物。

### <a name="prepublishOnly"></a>prepublishOnly
> `package.json` 的生命周期钩子，在执行 `npm publish` 前**自动**触发。pinme 的 `prepublishOnly` 设置了 `node build.js`，意味着每次发包时构建脚本都会自动运行——在 CI 环境中，这一步会把所有 CI secret 注入打包产物，然后产物随 npm 包公开发布。危险之处在于「自动触发」——开发者可能没意识到 publish 会先跑 build，更没意识到 build 会读 env。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件开头用 `---` 包围的 YAML 元数据块。在 Claude Code skill 文件里，frontmatter 通常声明 `name`、`description`、`allowed-tools`、`version` 等字段，是 Claude Code 解析 skill 的入口。NLPM 审查器通过检查 frontmatter 完整性来判断 skill 是否有 bug（如缺少必填字段）。

### <a name="npm"></a>npm（Node Package Manager）
> Node.js 的官方包管理器和公开注册表。开发者可以把自己的 JavaScript/TypeScript 库打包后发布到 `npmjs.com`，任何人都可以用 `npm install <包名>` 安装使用。一旦包发布到 npm，全球用户都能下载——这使得「把 secret 打进 npm 包」成为一个严重的公开泄露事件，而不只是本地安全问题。

### <a name="skill"></a>skill（技能文件）
> Claude Code 插件系统中的一种制品类型，通常是一个 Markdown 文件（SKILL.md），告诉 AI「如何完成特定领域的任务」。skill 不执行代码，只提供知识和指导——AI 在收到相关请求时读取 skill 内容，据此生成操作建议或代码。pinme 的 5 个 skill 文件共同构成了一个「API 集成知识套件」，每个文件覆盖一个集成领域。
