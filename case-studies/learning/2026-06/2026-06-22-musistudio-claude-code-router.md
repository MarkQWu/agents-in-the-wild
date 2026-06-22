# musistudio/claude-code-router — 学习案例

**仓库**：https://github.com/musistudio/claude-code-router
**Stars**：35,200+ | **来源**：xiaolai upstream
**Audit 日期**：2026-04-26（历史快照）| **生成日期**：2026-06-22（基于当前 HEAD）
**主题标签**：`template-design`、`cross-reference`、`vague-quantifier`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

claude-code-router 是一个本地 HTTP 中间件代理（middleware proxy），运行于 Claude Code 与各 LLM 提供商之间。它拦截 Claude Code 发出的 API 请求，依据任务类型（场景）将其路由到不同的模型或提供商（OpenAI、Google、Anthropic 等），从而让用户在不修改 Claude Code 本身配置的前提下，混合使用多家服务商、控制成本与性能。

- **语言**：TypeScript（96.3%）
- **包管理**：pnpm workspace monorepo
- **默认端口**：3456（`packages/cli/src/cli.ts` 第 143 行：`const port = globalConfig.PORT || 3456`）
- **配置主目录**：`~/.claude-code-router/`（含 config.json、logs/、presets/、plugins/）

### 1.2 架构剖析

monorepo 顶层分为四个主包：

| 包 | 职责 |
|----|------|
| `packages/cli` | 命令行入口，启动/停止/状态/备份/恢复预设等 CLI 子命令 |
| `packages/core` | 路由引擎、变换器管道（transformer pipeline）、提供商插件加载 |
| `packages/shared` | 跨包公共类型与工具函数 |
| `packages/ui` | 可选 Web UI |

核心数据流：

```
Claude Code → localhost:3456 → [Router] → [Transformer] → LLM Provider API
                                  ↑
                           config.json (Router 字段)
```

**Provider 模式**：每个提供商由 `name`、`api_base_url`、`api_key`、`models` 四个字段描述，运行时按需实例化。

**Transformer Pipeline**：请求与响应都经过可插拔的变换器，每个变换器只做一件事（格式转换、header 注入、流式响应适配等），彼此解耦。

**场景路由（Scenario-Based Routing）**：`Router` 对象将具名场景（`default`、`background`、`think`、`longContext`）映射到指定的提供商+模型组合。用户也可提供自定义 JS 脚本实现任意路由逻辑。

**环境变量插值**：config.json 中以 `$VAR_NAME` 引用外部环境变量，凭据不进版本库。

### 1.3 设计思路 / 方法论

1. **零侵入代理**：Claude Code 无需任何修改，只需将 `ANTHROPIC_BASE_URL` 指向本地代理即可。
2. **插件化提供商**：新增一家 LLM 提供商不需要修改核心代码，只需在 config.json 增加一个 provider 条目。
3. **场景驱动而非模型驱动**：用户配置的是"什么任务用什么策略"，而非"调用哪个模型"，解耦了使用意图与技术实现。
4. **本地优先**：所有路由逻辑在用户机器上运行，API Key 不经过第三方服务器。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

| 模式 | 描述 | 体现位置 |
|------|------|---------|
| **Proxy / Middleware** | 拦截并转发请求，对上下游透明 | packages/core 路由引擎 |
| **Strategy Pattern** | 将"选哪个提供商"的算法封装为可替换策略 | Router 场景映射 + 自定义 JS 脚本 |
| **Pipeline / Chain of Responsibility** | 请求/响应经过有序的变换器链 | Transformer Pipeline |
| **Plugin Architecture** | 提供商以统一接口注册，核心不感知具体实现 | packages/core provider 加载机制 |
| **Environment Injection** | 凭据通过环境变量注入，与配置结构分离 | `$VAR_NAME` 插值 |

### 2.2 适用场景

- **成本优化**：将耗时长但对质量要求低的后台任务（`background` 场景）路由到廉价模型。
- **能力互补**：将需要深度推理的任务（`think` 场景）路由到支持 CoT 的模型。
- **多区域合规**：按请求特征选择不同地区的 API 端点。
- **A/B 测试**：在不同场景下同时实验多个提供商，收集对比数据。

### 2.3 与其他架构对比

| 方案 | 路由层位置 | 侵入程度 | 灵活度 |
|------|-----------|---------|--------|
| claude-code-router | 本地 HTTP 代理 | 零侵入 | 高（自定义 JS） |
| LiteLLM Proxy | 独立服务进程 | 需改 base_url | 极高 |
| 直接在应用层切换 SDK | 应用内部 | 强耦合 | 低 |
| 云端网关（如 AWS Bedrock） | 云端 | 需账号/权限 | 中 |

claude-code-router 的独特优势在于与 Claude Code 生态的深度集成（识别场景标记）以及完全本地化运行。

### 2.4 改进空间

1. **文档一致性**：当前英文文档中端口仍有误（见第三节），多语言文档缺乏自动校验机制。
2. **发布安全**：`scripts/release.sh` 中 `npm publish` 无 `--provenance` 标志，无法提供构建溯源（provenance attestation）。
3. **Docker 镜像签名**：推送 `latest` 标签未启用 Docker Content Trust，下游用户无法验证镜像完整性。
4. **依赖锁定**：devDependencies 使用 `^` 版本范围，CI 构建结果在不同时间可能不同。

---

## 三、过去审查发现（2026-04-26 历史快照）

### 3.1 当时质量评分（NLPM）

- **总分**：79 / 100
- **安全状态**：CLEAR（4 项 Medium，1 项 Low）
- **被评分制品数**：15 个（主要为 Docusaurus CLI 参考文档，含 EN 和 zh-CN 双语）

> **重要背景**：被评分的文件是 CLI 参考文档，而非 NL skill/command 规范。评分中 `name`/`description` YAML 规则对文档文件属误报（false positive）。真正的质量问题是**内容准确性**与**跨语言一致性**。

各文件得分（最低优先）：

| 文件 | 得分 | 主要扣分原因 |
|------|------|------------|
| docs backup preset.md zh-CN | 65 | 路由格式写成 `openai:gpt-4`（应为 `openai,gpt-4`） |
| docs other.md EN | 70 | 5 条命令缺少输出格式说明 |
| docs other.md zh-CN | 70 | 同上 |
| docs start.md zh-CN | 72 | 默认端口与 EN 文档矛盾（3456 vs 8080） |
| docs status.md zh-CN | 72 | 同上端口矛盾 |

平均分：**79**

### 3.2 当时值得借鉴的模式

- **场景命名规范**：`default`、`background`、`think`、`longContext` 这四个场景名清晰传达了意图，无需额外解释。
- **config.json 注释文档**：配置文件结构通过文档而非代码注释传达，降低了阅读源码的门槛。
- **双语文档并行维护**：EN + zh-CN 双轨文档覆盖了主要用户群体，体现了对社区的重视。

### 3.3 当时的缺陷

1. **Bug 1（端口矛盾）**：zh-CN `start.md` 写默认端口 3456，EN `start.md` 写 8080——而源码实际默认是 3456，**英文文档有误**。
2. **Bug 2（端口矛盾）**：zh-CN `status.md` 同样存在 3456 vs 8080 矛盾。
3. **Bug 3（路由格式错误）**：`docs/i18n/zh-CN/.../cli/commands/preset.md` 第 151 行使用 `"openai:gpt-4"`（冒号），应为 `"openai,gpt-4"`（逗号）。
4. **Bug 4（本地化链接缺失）**：`statusline.md` 备份版本的 locale 链接缺少 `/zh` 前缀。
5. **Bug 5（本地化链接缺失）**：`statusline.md` 当前版本存在同样问题。

### 3.4 当时的优化机会

- M1：`scripts/release.sh` 中 `npm publish` 缺少 `--provenance` 标志
- M2：`@CCR/cli` 包的 `npm publish` 同样缺少 provenance
- M3：`docker push` 未启用 Docker Content Trust（无内容签名）
- M4：`latest` 标签推送同样无签名
- L1：devDependencies 使用 `^` 浮动版本范围

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 缺陷 | 2026-04-26 状态 | 2026-06-22 状态 | 变化 |
|------|----------------|----------------|------|
| 端口矛盾（start.md） | 存在 | **仍然存在** | 未修复 |
| 端口矛盾（status.md） | 存在 | **仍然存在** | 未修复 |
| 路由格式 `openai:gpt-4` | 存在于 zh-CN 备份 | **仍然存在**（`docs/i18n/zh-CN/docusaurus-plugin-content-docs.backup.20260101_205603/cli/commands/preset.md` 第 151 行） | 未修复 |
| locale 链接缺失 `/zh` | 存在 | 未深入验证，推测仍存在 | 可能未修复 |

**关键确认**：`packages/cli/src/cli.ts` 第 143 行当前仍为 `const port = globalConfig.PORT || 3456`，README 也一致使用 3456。因此**英文文档说 8080 是错误的**，不是 zh-CN 文档的问题。

### 4.2 架构演进

当前 HEAD 相较于 audit 时期：

- README 中的端口引用已统一为 3456（README 层面修复）
- monorepo 包结构（cli / core / shared / ui）保持稳定，无重大重构
- Stars 从 audit 时期增长至 35,200+，社区采用度显著提升

### 4.3 新增的可学习模式

- **README 双语并行**（README.md + README_zh.md）：比 Docusaurus i18n 更轻量的双语策略，适合早期项目。
- **插件目录约定**（`~/.claude-code-router/plugins/`）：将用户扩展点标准化为文件系统路径约定，无需注册中心。

---

## 五、校准

### 5.1 我已经在做对的

如果你的项目维护了任何形式的配置文档或 README：

- ✓ **单一真相源（single source of truth）**：源码是权威，文档跟随源码，不反过来
- ✓ **凭据外化**：API Key 通过环境变量传入，不硬编码在配置文件模板里
- ✓ **场景命名语义化**：用 `think`、`background` 这类业务语言命名，而非 `model_a`、`model_b`

### 5.2 挑战 / 验证

以下判断值得用你自己的项目来验证：

1. **文档与代码同步的代价**：claude-code-router 的端口错误在 audit 后近两个月仍未修复，说明即使对活跃项目，文档漂移（documentation drift）也是常态而非例外。你的项目是否有机制防止这种漂移？
2. **备份文件的质量负担**：`docs/i18n/zh-CN/docusaurus-plugin-content-docs.backup.*` 这类备份目录携带了过时错误，而评分工具会扫描它们。备份是否应该排除在文档质量扫描之外？
3. **误报成本**：NLPM 对 CLI 参考文档中的 `name`/`description` 字段触发 YAML 规则，产生误报。这提示：评分工具的覆盖范围设计与制品类型的匹配度需要持续校准。

---

## 六、行动

### 6.1 自查动作

针对你自己项目的具体检查动作：

1. **端口/URL 一致性检查**：
   ```bash
   # 找出所有文档中的端口引用，与源码对比
   grep -rn "localhost:[0-9]\+" docs/ README*.md
   grep -rn "PORT\|port\s*=" packages/cli/src/
   ```

2. **跨语言文档一致性检查**：
   ```bash
   # 统计 EN 和 zh-CN 文档的行数差异，差异过大说明内容不同步
   wc -l docs/cli/commands/*.md
   wc -l docs/i18n/zh-CN/docusaurus-plugin-content-docs/cli/commands/*.md
   ```

3. **路由格式引用检查**：
   ```bash
   # 检查是否存在冒号分隔（错误格式）
   grep -rn '"[a-z]*:[a-z]' docs/
   ```

4. **发布脚本安全检查**：
   ```bash
   grep -n "npm publish" scripts/release.sh
   # 确认是否含有 --provenance 标志
   ```

### 6.2 灵感 → 实施路径

| 灵感来源 | 对应实施动作 |
|---------|------------|
| 场景驱动路由命名 | 在你的项目 skill 文件中，用业务场景命名而非技术实现命名（如 `summarize-legal-brief` 而非 `call-claude-with-long-context`） |
| 零侵入代理模式 | 若需要在 Claude Code 之外接入自定义逻辑，优先考虑代理层而非修改 CLAUDE.md 中的大量指令 |
| 插件目录约定 | 为你的 Claude Code 插件定义约定式扩展点目录（如 `.claude/rules/`、`.claude/hooks/`），降低用户配置门槛 |
| 备份文件排除 | 在 `.nlpm-ignore` 或同类配置中排除 `*.backup.*` 目录，避免备份污染质量评分 |

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

| 我的仓库 | 类型 | 与 claude-code-router 的相关度 |
|---------|------|-------------------------------|
| `MarkQWu/drama-workshop-skills` | Claude Code skill 集合（微短剧写作） | 低——领域不同，但同为 Claude Code 生态插件 |
| `MarkQWu/echo-sleuth-for-claude` | Claude Code 会话挖掘插件 | 中——同为 Claude Code 工具链，但功能互补而非重叠 |
| `MarkQWu/claude-for-legal` | 法律工作流 | 低——垂直领域应用，无路由需求 |

claude-code-router 是**基础设施层**工具；我的三个仓库都是**应用层**工具。直接功能复用机会有限，但架构模式（特别是场景命名和文档一致性）有普遍借鉴价值。

### 8.2 在我的项目里复现的同类问题

**文档准确性漂移**：claude-code-router 的核心教训是文档与源码之间的漂移——一个端口号在三个地方（源码、EN 文档、zh-CN 文档）出现了分叉。

对应到我的项目：
- `echo-sleuth-for-claude`：CLAUDE.md 中的命令示例是否与实际脚本行为一致？安装说明中的路径是否仍然有效？
- `drama-workshop-skills`：skill 文件中引用的输出格式是否与实际 Claude 响应结构匹配？
- `claude-for-legal`：工作流文档中引用的 Claude Code 命令语法是否跟上了 Claude Code 版本更新？

**自查建议**：为每个仓库建立一个"文档准确性检查单"，列出所有需要与源码/工具版本保持同步的具体声明（版本号、端口、路径、命令语法）。

### 8.3 别人的更优方案

1. **场景化路由配置**：claude-code-router 用四个语义场景（`default`/`background`/`think`/`longContext`）覆盖了大多数 Claude Code 使用模式。我的 skill 文件目前没有类似的场景分层——所有 skill 对 Claude 来说权重相同。借鉴这一模式，可以在 skill 的 `description` 字段中显式声明适用场景，帮助模型更准确地选择 skill。

2. **插件目录约定**：`~/.claude-code-router/plugins/` 这一约定式扩展点比"在 README 里描述如何修改源码"更友好。`echo-sleuth-for-claude` 可以考虑为用户自定义过滤规则提供类似的约定目录。

3. **README 双语策略**：README.md（EN）+ README_zh.md（zh-CN）的平行文件策略比 Docusaurus i18n 机制轻量得多。对于早期项目（stars < 1000），这种方式的维护成本更低，且不会引入 Docusaurus 的备份文件质量问题。

### 8.4 反向：我的项目做得比他们好的地方

1. **文档范围聚焦**：我的项目目前没有维护多语言文档，因此不存在跨语言一致性问题。这既是局限（覆盖用户群窄），也是优势（无一致性维护负担）。在项目早期，单语言文档维护质量更高是正确的权衡。

2. **无发布脚本安全风险**：我的项目（skill 文件 + CLAUDE.md）没有 npm publish 流程，因此不存在 M1/M2（无 provenance）的风险。Claude Code 插件的"发布"通过 `claude plugin install` 直接从 GitHub 拉取，跳过了 npm 注册中心这一攻击面。

3. **备份文件污染风险为零**：Docusaurus 的备份机制（`.backup.YYYYMMDD_HHMMSS` 目录）是 claude-code-router 路由格式错误至今未被发现的原因之一——错误藏在备份文件里，不在主文档路径。我的项目无此类备份机制，质量扫描覆盖范围更可预测。

---

## 八、术语表

| 术语 | 解释 |
|------|------|
| **路由器（Router）** | 在网络或软件中，根据规则将请求分发到不同目标的组件。claude-code-router 中，Router 根据场景名将 API 请求分发到不同的 LLM 提供商。 |
| **Docusaurus** | Meta 开源的静态网站生成器，专为技术文档设计。支持 Markdown、MDX、多语言（i18n）、版本控制。claude-code-router 使用 Docusaurus 构建其 `docs/` 站点，i18n 目录下存放 zh-CN 翻译。 |
| **monorepo** | 将多个相关项目/包放在同一个 Git 仓库中管理的策略。claude-code-router 使用 pnpm workspace 管理 cli/core/shared/ui 四个包，共享 tsconfig 和 lint 配置。 |
| **provenance attestation** | 构建溯源证明。npm publish 加 `--provenance` 标志后，npm 会生成一份由 OIDC 签署的声明，记录"这个包是从哪个 Git commit、在哪个 CI 环境中构建的"，下游用户可验证包的来源可信度。claude-code-router 当前发布流程缺少此标志。 |
| **Docker Content Trust（DCT）** | Docker 的镜像签名机制。启用后（`DOCKER_CONTENT_TRUST=1`），`docker push` 会对镜像摘要进行加密签名，下游 `docker pull` 可验证镜像未被篡改。claude-code-router 的 CI 推送 `latest` 标签时未启用 DCT。 |
| **Transformer Pipeline** | 变换器管道。一种设计模式，将数据处理拆分为一系列独立的变换步骤（transformer），每个步骤只做一件事，通过管道串联。claude-code-router 用此模式处理 LLM 请求/响应的格式转换。 |
| **false positive（误报）** | 检测工具将正常情况错误地标记为问题。本案例中，NLPM 对 CLI 参考文档中的 `name`/`description` 字段触发 YAML 命名规则属于误报——这些字段是文档内容，而非 NL artifact 的元数据字段。 |
| **documentation drift（文档漂移）** | 随时间推移，文档内容与实际代码/系统行为之间出现偏差的现象。claude-code-router 的端口错误是典型案例：源码改了端口默认值，但部分文档未同步更新。 |
