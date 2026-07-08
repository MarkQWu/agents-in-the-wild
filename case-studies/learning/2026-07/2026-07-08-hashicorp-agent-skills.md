# hashicorp/agent-skills — 学习案例

**仓库**：https://github.com/hashicorp/agent-skills
**Stars**：702 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-07-08（基于当前 HEAD）
**主题标签**：vague-quantifier, security-gate, single-purpose, cross-reference

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

HashiCorp 官方维护的 Claude Code agent skills 集合，面向使用 Terraform 和 Packer 的 DevOps 工程师。创建于 2025-11-08，描述为"A collection of Agent skills and Claude Code plugins for HashiCorp products"。

5 个关键事实：
1. 702 星、85 fork，是 HashiCorp 产品线中首个公开的 Claude Code 集成——对 Terraform 生态用户具有参考价值
2. 覆盖两大产品线：Terraform（基础设施即代码）和 Packer（镜像构建），每条产品线下分多个子域，共 5 个独立 [manifest](#manifest)
3. 最近一次推送为 2026-05-28，距今约 6 周，活跃度中等
4. 仓库顶部有 `scripts/validate-structure.sh`，说明团队有自动化结构校验意识
5. 许可证为 MPL-2.0（Mozilla Public License），HashiCorp 标准许可证

### 1.2 架构剖析

- **目录结构**：
  ```
  terraform/
    code-generation/
      .claude-plugin/plugin.json   ← 含 MCP server config (TFE_TOKEN docker)
      skills/
        terraform-search-import/
          scripts/
            list_resources.sh      ← 2026-04-06 审计时缺失，现已存在
          SKILL.md
        terraform-style-guide/SKILL.md
        terraform-test/SKILL.md
        azure-verified-modules/SKILL.md
    module-generation/
      .claude-plugin/plugin.json   ← 同样含 TFE_TOKEN docker config
      skills/
        refactor-module/SKILL.md
        terraform-stacks/SKILL.md
    provider-development/
      .claude-plugin/plugin.json
      skills/
        provider-actions/SKILL.md
        provider-resources/SKILL.md
        provider-docs/SKILL.md
        new-terraform-provider/SKILL.md
        provider-test-patterns/SKILL.md
        run-acceptance-tests/SKILL.md
  packer/
    hcp/
      .claude-plugin/plugin.json
      skills/push-to-registry/SKILL.md
    builders/
      .claude-plugin/plugin.json
      skills/
        aws-ami-builder/SKILL.md
        azure-image-builder/SKILL.md
        windows-builder/SKILL.md   ← 含 chocolatey iex 示例
  scripts/
    validate-structure.sh
  ```
- **文件类型分布**：21 个 NL artifact，全部为 SKILL.md，无 agents 或 commands。5 个 plugin manifest
- **编排关系**：平列结构——每个 skill 独立，无跨件调用。用户在 Terraform/Packer 任务中手动决定使用哪个技能
- **跨件契约**：几个技能有 `references/` 子目录（MOCK_PROVIDERS.md、CI_CD.md、EXAMPLES.md 等），通过相对路径内联参考文档

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「产品线隔离 + 多 [manifest](#manifest)」——Terraform 的 code-generation 和 module-generation 分别有自己的 plugin.json，用户可以只安装需要的子集
- **解决什么问题**：让 Claude Code 能够辅助复杂的 Terraform/Packer 工作流（资源导入、模块重构、provider 开发测试等），把 HashiCorp 的最佳实践内嵌到 AI 提示中
- **做了什么 trade-off**：选择多 manifest（5 个）而非单 manifest，代价是同步成本高（同一个 MCP server 配置块在两个 plugin.json 里重复）；收益是用户可以精确安装所需产品线
- **反映什么认知模型**：作者把 skills 视为"领域知识封装容器"，每个 SKILL.md 相当于把一个领域专家的操作手册翻译成 AI 可执行的指令

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**「产品线隔离多 manifest + 内联参考文档」模式**：按产品/子域分拆 manifest，同时通过 `references/` 目录把厚重的专业知识从主 SKILL.md 中剥离。

模式特征清单：
- 特征 1：目录层级映射产品边界（terraform/ vs packer/ → code-generation/ vs module-generation/）
- 特征 2：每个产品子域有独立 plugin.json，可单独发版和安装
- 特征 3：SKILL.md 保持简洁，厚重内容外置到 `references/` 子目录
- 特征 4：顶层 `scripts/` 目录有结构校验脚本，自动守护仓库规范

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型工具生态（多产品/多子域）的官方集成 | ✅ 高度适用 | 按产品线隔离避免"全家桶"膨胀 |
| 个人技能集合（<20 个 skill） | ❌ 过度设计 | 单 manifest 就够，多 manifest 维护成本不合算 |
| 需要 MCP server 集成的企业级工具 | ✅ 适用 | 各产品线可独立配置自己的 MCP server |
| 需要频繁跨 skill 共享状态 | ❌ 不适用 | 平列结构无法跨 skill 传递上下文 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 产品线多 manifest（本仓库） | hashicorp/agent-skills | 精确安装，产品边界清晰 | 公共配置重复（如 MCP server config） |
| 单 manifest 全局 skill | trailofbits/skills | 一键获取全部能力 | 用户需要全部工具，无法精选 |
| 外部参考型（references/子目录） | 本仓库中的 provider-development skills | SKILL.md 简洁，知识分层 | 参考文档丢失后指令静默失败 |

### 2.4 改进空间

1. **当前问题**：两个 plugin.json（code-generation 和 module-generation）含相同的 terraform MCP server 配置块，修改要改两处。**改进做法**：抽取共用 MCP 配置到根目录 `terraform/.claude-plugin/plugin.json`，子目录通过继承或引用。**预期收益**：单点修改，避免配置漂移
2. **当前问题**：TFE_TOKEN 通过 `-e TFE_TOKEN` 传入 Docker 容器，但未文档化最小权限范围，镜像未锁定到 digest。**改进做法**：在 plugin.json 旁添加 README 说明最小权限，镜像改为 `hashicorp/terraform-mcp-server@sha256:<digest>`。**预期收益**：消除供应链风险
3. **当前问题**：`refactor-module/SKILL.md` 使用绝对 githubusercontent.com URL 引用同仓库内的其他 skill 文件。**改进做法**：改为相对路径（`../../code-generation/skills/terraform-style-guide/SKILL.md`）。**预期收益**：fork 后仍可使用，不会因 URL 失效而断链

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 当时得分 **98/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| terraform/provider-development/skills/provider-actions/SKILL.md | 92 | 4 个模糊量词：meaningful、when relevant、appropriate (×2) |
| terraform/module-generation/skills/refactor-module/SKILL.md | 92 | 4 个模糊量词：well-structured、clear、proper、appropriate |
| terraform/code-generation/skills/terraform-style-guide/SKILL.md | 96 | 2 个模糊量词：meaningful、where applicable |
| terraform/module-generation/skills/terraform-stacks/SKILL.md | 96 | 2 个模糊量词：descriptive、logical |
| terraform/code-generation/skills/terraform-search-import/SKILL.md | 90 | 引用不存在的 `./scripts/list_resources.sh` |

### 3.2 当时值得借鉴的模式

1. **产品线边界清晰** → 把 terraform/packer 的不同子域放在独立目录，每个 plugin.json 只负责一个子域 → `terraform/code-generation/.claude-plugin/plugin.json` → 自己的仓库如果有多个功能域，可以考虑类似分拆
2. **脚本配套** → `terraform-search-import` skill 配套 shell 脚本辅助资源发现 → `skills/terraform-search-import/scripts/list_resources.sh` → 纯文本 SKILL.md 配合可执行脚本，能力边界清晰
3. **满分 manifest** → 5 个 plugin.json 全部得满分 100，说明 manifest 格式非常规范 → 可以直接参考这里的 plugin.json 格式来规范自己的 manifest

### 3.3 当时的缺陷

1. **引用不存在的脚本** → `terraform-search-import/SKILL.md` 在第 30-31 行和 55-57 行都让用户运行 `./scripts/list_resources.sh`，但这个文件不存在。用户照着做会立即碰壁。**根本原因**：文档先于实现，或文件在重构时被意外删除后没有同步更新 SKILL.md。**自查**：我的 SKILL.md 有没有引用不存在的文件或脚本？
2. **模糊量词泛滥** → provider-actions SKILL.md 里"meaningful progress messages"、"when relevant"等词让 AI 无法知道具体该做什么。**根本原因**：从人类文档迁移过来的描述习惯，没有意识到 AI 需要可验证的标准而非审美判断。**自查**：我的 SKILL.md 有没有"appropriate/relevant/meaningful"这类词？
3. **绝对 URL 引用同仓库** → `refactor-module/SKILL.md` 用 githubusercontent.com URL 引用同仓库的其他文件，fork 后 URL 失效。**根本原因**：快速复制链接的习惯，没考虑可移植性。**自查**：我的文档有没有用绝对 URL 引用同仓库内容？

### 3.4 当时的优化机会

1. `provider-actions/SKILL.md` 中的"Use appropriate error types"改为列举具体错误类型（ErrNotFound/ErrConflict 等）
2. `windows-builder/SKILL.md` 的 chocolatey 安装示例加 integrity 注释（验证签名或 checksum）
3. 两个 plugin.json 的重复 MCP server 配置块抽取到共同位置

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `list_resources.sh` 不存在 | `ls terraform/code-generation/skills/terraform-search-import/scripts/` | **已修复**：`list_resources.sh` 已存在，脚本引用不再断链 | 主要 workflow bug 已修；推测是在 NLPM PR 或内部 issue 后修复 |
| TFE_TOKEN 未文档化、镜像未锁定 | `grep "TFE_TOKEN\|digest\|sha256" terraform/code-generation/.claude-plugin/plugin.json` | **仍存在**：`-e TFE_TOKEN` 传 Docker 但无文档说明最小权限，无 digest 固定 | Medium 安全风险持续 |
| windows-builder chocolatey iex | `grep "iex\|DownloadString" packer/builders/skills/windows-builder/SKILL.md` | **仍存在**：第 106 行依然是 `iex (New-Object System.Net.WebClient).DownloadString(...)` 无 integrity 说明 | Low 安全风险持续 |

### 4.2 架构演进

从审计到现在（约 3 个月），仓库变化相对平稳：
- **修复**：`list_resources.sh` 已补全，这是影响最大的 bug
- **无大型重组**：产品线目录结构未变，说明初始架构决策稳定
- **意味着**：这是一个"功能稳定、持续小修"的仓库，区别于 anthropics 的快速扩张。外部 PR 政策（CLA 要求）可能减慢了社区贡献速度

### 4.3 新增的可学习模式

当前 HEAD 与审计时无重大结构变化，暂无新增可学习模式。主要变化是 `list_resources.sh` 的补全——这个补全本身说明了一个教训：SKILL.md 中的脚本引用应该与 CI 结构校验绑定（`scripts/validate-structure.sh` 应该验证所有被引用的文件都存在）。

---

## 五、校准

### 5.1 我已经在做对的

1. **避免绝对 URL 引用同仓库文件**：echo-sleuth 的 SKILL.md 文件使用相对路径引用其他文件，不用 githubusercontent.com 链接
2. **SKILL.md 尽量可验证**：echo-sleuth 中步骤描述具体到命令级别（`./scripts/list-sessions.sh`），而非"use appropriate command"
3. **引用文件前确认存在**：echo-sleuth 的技能引用的脚本（list-sessions.sh、extract-messages.sh）均已在仓库中存在

### 5.2 挑战 / 验证

本案例验证了一个假设：**"模糊量词确实会降低 skill 质量"**。

provider-actions 里 4 个模糊量词（meaningful、when relevant、appropriate×2）的代价是：AI 遇到这些词时会依赖自己的主观判断，导致行为不可复现。这不是风格问题，是"AI 能不能可靠执行指令"的可靠性问题。

具体验证：比较 provider-test-patterns/SKILL.md（得 100 分、无模糊量词）和 provider-actions/SKILL.md（92 分、4 个模糊量词）的写法差异——前者每条指令都有可验证结果，后者依赖 AI 的审美判断。这个对比是"模糊量词有代价"的直接证据。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 中的模糊量词
grep -rn -E '\b(appropriate|relevant|meaningful|well-structured|proper|descriptive|logical|adequate|sufficient|specific enough)\b' \
  ~/.claude/skills/*/SKILL.md ~/projects/*/skills/*/SKILL.md 2>/dev/null | \
  grep -v "^Binary"
```
命中后：将每个模糊词替换为可验证标准。例："meaningful labels"→"labels must include version (semver), team name, and compliance tag"；"when relevant"→"when the error is not a 4xx HTTP status code"。

```bash
# 检查我的 SKILL.md 有没有引用不存在的文件
python3 -c "
import re, os, glob
for skill in glob.glob('**/*.md', recursive=True):
    content = open(skill).read()
    # 匹配 ./scripts/xxx、./references/xxx 等相对路径引用
    refs = re.findall(r'\./[\w/.-]+\.(sh|py|md|json)', content)
    base = os.path.dirname(skill)
    for ref in refs:
        path = os.path.join(base, ref)
        if not os.path.exists(path):
            print(f'BROKEN REF in {skill}: {ref}')
" 2>/dev/null
```
命中后：检查被引用文件是否应该存在（如果是，补全它；如果 SKILL.md 写错了引用路径，修正路径）。

### 6.2 灵感 → 实施路径

1. **想法**：在 echo-sleuth 中为每个需要调用脚本的 SKILL.md 添加 CI 校验，确保引用的脚本都存在
   - **为何可行**：hashicorp 已有 `validate-structure.sh` 的先例，说明这个模式可行
   - **第一步**：在 `.github/workflows/ci.yml` 中添加一步：`python3 scripts/validate-refs.py`，扫描所有 SKILL.md 中的相对路径引用并验证其存在，约 1 小时

2. **想法**：将 bureau 的 references/ 文档模式引入 echo-sleuth——把每个 skill 的厚重背景知识放到 `skills/<name>/references/` 而非直接内联
   - **为何可行**：echo-sleuth 的 experience-synthesis 技能正在变长，已经有需要外置知识的迹象
   - **第一步**：把 `skills/experience-synthesis/SKILL.md` 中超过 50 行的"背景说明"部分移到 `skills/experience-synthesis/references/context.md`，SKILL.md 改为引用，约 20 分钟

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 8.1 目的对齐度

- **本案例 hashicorp/agent-skills 的核心目的**：把 HashiCorp 产品线的最佳实践内嵌为 Claude Code skills，辅助 DevOps 工程师使用 Terraform/Packer
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/bureau | 中 | 均为工具辅助型 skill 集合，有 references 风格的补充文档 | bureau 是知识库管理，不是 DevOps 工具 | 中（架构参考） |
| MarkQWu/echo-sleuth-for-claude | 低 | 均有 skills 目录 | echo-sleuth 是对话挖掘，无产品线概念 | 低 |
| MarkQWu/gstack | 无 | — | gstack 是 AI builder framework，架构完全不同 | 无 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 模糊量词 | `grep -rn "appropriate\|relevant\|meaningful" /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/*/SKILL.md` | 命中：`experience-synthesis/SKILL.md` 第 118 行 "identify **relevant** sessions"；`git-mining/SKILL.md` 第 93 行 "Find the **relevant** git commits" | 中 |
| 引用不存在的脚本 | `python3 -c "..."` 扫描 | 未命中（echo-sleuth 所有脚本引用均实际存在） | 无 |

**命中后的具体行动**：
- `MarkQWu/echo-sleuth-for-claude/skills/experience-synthesis/SKILL.md:118`："relevant sessions"→"sessions from the past 90 days that contain at least one of the search keywords" → 约 5 分钟
- `MarkQWu/echo-sleuth-for-claude/skills/git-mining/SKILL.md:93`："relevant git commits"→"git commits whose message or diff contains the target keyword or file path" → 约 5 分钟

### 8.3 别人的更优方案

1. **领域**：脚本与 SKILL.md 捆绑
   - **本案例做法**：`terraform-search-import/scripts/list_resources.sh` 与 SKILL.md 放在同一个 skill 目录下，SKILL.md 直接引用 `./scripts/list_resources.sh`（`terraform/code-generation/skills/terraform-search-import/SKILL.md`）
   - **我的项目现状**：echo-sleuth 的脚本放在仓库根目录的 `scripts/`，SKILL.md 用绝对路径或环境变量引用，耦合较松
   - **如何借鉴**：把 echo-sleuth 中每个 skill 专属的脚本移到 `skills/<name>/scripts/` 下，SKILL.md 改为相对路径引用 `./scripts/<name>.sh`，大约 30 分钟重构

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：CI 结构校验
- **我的做法**：echo-sleuth 有 `tests/` 目录，包含对脚本的 unittest（`tests/test_nlpm_check.py` 风格）
- **本案例做法**：`scripts/validate-structure.sh` 存在，但 `list_resources.sh` 的缺失还是让主要 workflow 断掉了，说明校验脚本覆盖不完整
- **意义**：echo-sleuth 的测试覆盖面比 hashicorp 更全，这是一个保持的优势；继续保持每个新脚本/引用都有对应测试的习惯

---

## 八、术语表

### <a name="manifest"></a>manifest（清单文件）
> 项目的"成分表"，告诉系统这个项目包含哪些组件。`plugin.json` 就是 Claude Code 插件的 manifest。类比：就像乐高套装的零件清单，告诉你盒子里有什么；如果清单漏了几块，那些零件即使在盒子里也不会被计入。

### <a name="MCP-server"></a>MCP server（模型上下文协议服务器）
> 一个独立运行的服务，通过标准协议（Model Context Protocol）向 Claude Code 提供额外的工具能力。例如 Terraform 的 MCP server 让 Claude 能直接查询 Terraform 状态、调用 provider API。类比：MCP server 相当于给 Claude 添了一个"翻译官"，让它能和特定系统（如 Terraform Enterprise）说话。

### <a name="TFE_TOKEN"></a>TFE_TOKEN（Terraform Enterprise API 密钥）
> 访问 Terraform Enterprise（HashiCorp 的企业版 Terraform SaaS）所需的认证令牌。本案例将其通过环境变量 `-e TFE_TOKEN` 传入 Docker 容器，安全风险在于：如果容器镜像被篡改，令牌可能被泄露。

### <a name="vague-quantifier"></a>模糊量词（vague quantifier）
> 在 NL artifact（自然语言指令）中无法被客观验证的形容词或副词，例如"适当的"（appropriate）、"有意义的"（meaningful）、"相关的"（relevant）、"结构良好的"（well-structured）。NLPM 把这类词视为质量缺陷，因为 AI 执行这类指令时会依赖主观判断，导致行为不可复现。

### <a name="supply-chain-attack"></a>供应链攻击（supply chain attack）
> 通过攻击软件的"上游供应商"（如某个 npm 包、Docker 镜像）来间接攻击使用这个供应商的下游用户。本案例中的风险是：如果 `bun install` 拉取的某个版本范围内的包被恶意方入侵，用户的 MCP server 在启动时就会运行恶意代码。
