# kubesphere/kubesphere — 学习案例

**仓库**：https://github.com/kubesphere/kubesphere
**Stars**：16912 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-27（历史快照）| **生成日期**：2026-06-17（基于当前 HEAD）
**主题标签**：`security-gate`, `template-design`, `cross-reference`, `examples-driven`, `manifest-discipline`

**xiaolai 案例**：[../2026-05-07-kubesphere-kubesphere.md](../2026-05-07-kubesphere-kubesphere.md)

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
kubesphere/kubesphere 是一个面向 Kubernetes 多云、数据中心和边缘管理的开源容器平台（16912 颗星，2735 forks）。其 NL 产物层——`skills/` 目录下 21 个 SKILL.md——是整个 KubeSphere 生态的 AI 接入面：AI agent 理解 KubeSphere API、DevOps 流程、可观测性栈时所仰赖的结构化指引。

关键事实：
- kubesphere 主体是 Go 微内核（LuBan 架构：ks-apiserver + ks-controller-manager + ks-console）
- NL 产物专注于 KubeSphere 扩展生态，覆盖 DevOps（Jenkins/ArgoCD/Pipeline）、可观测性（Whizard 日志/事件/告警/追踪）、多租户、存储（Fluid/OpenSearch/Vector）等 10+ 子系统
- 21 个 skill，其中 14 个得分 90+，最低分 77（devops-tenant）
- xiaolai 在 2026-05-07 提交 5 个 PR（#6632-#6636），修复了 5 个机械 bug

### 1.2 架构剖析
- **目录结构**：

```
kubesphere/
├── skills/                                # 21 个 SKILL.md
│   ├── kubesphere-core/                   # 路由 skill：分发到具体子系统 skill
│   │   └── scripts/ks_api.py             # OAuth token 工具脚本
│   ├── kubesphere-devops-*/              # 5 个 DevOps 相关 skill
│   ├── whizard-*/                         # 5 个可观测性 skill（日志/事件/告警/追踪/审计）
│   ├── kubesphere-multi-tenant-management/
│   │   └── scripts/ks_api.py             # 与 kubesphere-core 完全重复（245 行 byte-for-byte）
│   ├── opensearch/                        # 有 Output Contract 节，跨 skill 数据共享范例
│   ├── kubesphere-fluid/                  # Fluid 存储扩展
│   └── ...
├── config/ks-core/charts/ks-crds/scripts/
│   └── post-delete.sh                    # Helm post-delete hook（含 xargs sh -c 安全问题）
└── ...（Go 主体代码）
```

- **文件类型分布**：21 个 SKILL.md，多个 Python 脚本（ks_api.py）、bash 脚本（generate-config.sh）、Helm hook 脚本
- **编排关系**：`kubesphere-core` 作为路由 skill，其 frontmatter description 明确列出分发规则（多集群→cluster-management，多租户→multi-tenant-management，扩展→extension-management）。其余 20 个 skill 各管一个子系统，相对独立。`opensearch` skill 有 `Output Contract` 节，定义跨 skill 数据格式。
- **跨件契约**：`whizard-telemetry/scripts/generate-config.sh` 运行时依赖 `vector` 扩展创建的 `vector-sinks` Kubernetes Secret，但此依赖未在任何 skill 中文档化。`ks_api.py` 在 2 个不同 skill 目录下存在完全相同的拷贝。

### 1.3 设计思路 / 方法论
- **核心设计哲学**：「平台即 skill 集合」——把每个 KubeSphere 扩展的操作知识（安装、配置、排错）封装为一个 SKILL.md，让 AI agent 在触碰真实集群之前先掌握正确的 API 使用方式和约束条件。
- **解决什么问题**：KubeSphere 的 API 有很多平台特定约束（如 DevOps 凭证必须用 `credential.devops.kubesphere.io/*` 类型，不能用 `Opaque`）。没有 skill，AI agent 很可能用通用 K8s 知识操作，产生「静默失败」的错误（凭证创建成功但 Jenkins 同步失败）。
- **Trade-off**：深度（每个 skill 覆盖子系统全细节）vs 广度（覆盖所有子系统）。kubesphere 选择深度优先——每个 skill 包含完整的安装步骤、API 示例、常见错误表，但这导致单个 skill 文件超长（devops-pipeline SKILL.md 超过 1300 行）。
- **认知模型**：作者把 AI agent 视为「需要本地知识的操作员」——skill 是 AI 版本的运维手册，不是通用教程，而是针对 KubeSphere 特定 API 版本和约束的精确操作指引。

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**「主体代码仓 + 内嵌 AI 接入层」模式**：在一个大型工程项目的主代码仓里直接维护 `skills/` 子目录作为 AI 接入层，与平台代码共存。

模式特征清单：
- AI 接入层（skills）与被它描述的系统（KubeSphere）共生在同一仓库
- 有路由 skill（kubesphere-core）作为入口，明确分发规则
- 每个子系统对应一个 skill，有明确的「What/When/How/Troubleshooting」结构
- 跨 skill 的数据合约通过「Output Contract」节显式化（opensearch skill 为典范）

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 大型平台/框架需要 AI 接入 | ✅ 高度适用 | skill 和平台代码共生，更新同步，平台特定知识准确率高 |
| 跨公司通用 skill 集（如 superpowers）| ❌ 不适用 | 平台特定知识无法跨项目复用 |
| 需要 AI 安全执行敏感操作 | ⚠️ 谨慎 | 文档中出现示例密码可能被 AI 直接使用（P@88w0rd 问题） |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| 主仓内嵌 AI 接入层（本仓库）| kubesphere/kubesphere | 知识与代码同步，平台特定 | 不可复用，skill 质量依赖主仓维护者 |
| 独立 skill 仓库 | superpowers-zh | 专注 AI 工作流，多平台，可独立安装 | 与平台代码脱节，更新滞后 |
| 插件 marketplace 插件 | KhazP/vibe-coding-prompt-template | 有分发生态 | 维护者需要同时懂平台和 NL 规范 |

### 2.4 改进空间
1. **当前问题**：`ks_api.py` 在 2 个 skill 目录下完全重复（245 行 × 2），安全修复必须同步应用两次，实际上已出现遗漏（chmod 修复未一致应用）。**改进做法**：提取到 `skills/shared/ks_api.py`，两个 skill 的脚本通过符号链接或脚本内 `import` 引用。**预期收益**：单一修复点，消除同步遗漏风险。
2. **当前问题**：文档示例密码 `P@88w0rd` 外观真实，可能被 AI 直接填入生产配置。**改进做法**：替换为 `<YOUR_SECURE_PASSWORD>` 或 `REPLACE_ME_CHANGEME`（格式强调）。**预期收益**：消除 Medium 安全发现，降低 AI 误用风险。

---

## 三、过去审查发现（2026-04-27 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-27 当时得分 **89/100**。安全状态：**REVIEW**（1 High + 4 Medium）。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| kubesphere-devops-tenant/SKILL.md | 77 | 硬编码密码 `P@88w0rd` (-5)，重复 Authorization header (-2) |
| kubesphere-devops-credentials/SKILL.md | 78 | API 示例用 `Opaque` 类型与文内警告矛盾 |
| kubesphere-devops-overview/SKILL.md | 84 | ASCII 架构图重复，`Key Resources` 标题出现两次 |
| kubesphere-devops-pipeline/SKILL.md | 86 | 缺闭合 `**`，Common Mistakes 表格重复 |
| kubesphere-fluid/SKILL.md | 87 | YAML 模板语法错误：`low: "{{low}}` 缺闭合引号 |
| 其余 16 个 SKILL.md | 88-93 | 多数 90 分以上 |

### 3.2 当时值得借鉴的模式
1. **Output Contract 节（opensearch SKILL.md）**：定义了 skill 向调用方返回的数据格式（字段名、类型、含义），是跨 skill 数据共享的显式契约。**极少见的高质量模式**——大多数 skill 不声明输出格式，导致下游 skill 无法可靠解析。
2. **Out of Scope 节（kubesphere-cluster-management SKILL.md）**：明确列出「本 skill 不负责的事项」，防止 AI 把不相关问题错误路由到这里。
3. **Prerequisites 链式声明（whizard-events SKILL.md）**：按步骤编号列出先决条件，每条有具体命令验证。AI 在执行前可逐一确认环境就绪。
4. **路由 skill 模式（kubesphere-core）**：单入口 skill，frontmatter description 就是路由表，直接告知模型「遇到多集群问题去 cluster-management，遇到多租户问题去 multi-tenant-management」。对于大型 skill 集合，这比让模型自行判断 skill 更可靠。

### 3.3 当时的缺陷
1. **API 示例与警告自相矛盾（devops-credentials）**：同一个文件第 56-70 行警告「`Opaque` 类型会导致 Jenkins 凭证同步失败」，第 111 行的 API 示例却用了 `"type": "Opaque"`。根本原因：示例代码和说明文字由不同时间写成，未互相校对。**这是最危险的缺陷类型**——文件本身看起来完整，但 AI 会优先跟随代码示例（而非文字警告），产生静默失败的凭证。自查：我的 skill 有没有代码示例和说明文字互相矛盾的情况？
2. **硬编码示例密码**：`P@88w0rd` 外观像真实密码（大写+特殊符号+数字），AI 可能直接用于生产。根本原因：复制粘贴了 KubeSphere 内部测试账号。自查：我的 skill 有没有放了看起来真实的示例凭证？
3. **`ks_api.py` 文件完全重复**：`skills/kubesphere-core/scripts/ks_api.py` 和 `skills/kubesphere-multi-tenant-management/scripts/ks_api.py` 字节级相同。根本原因：快速添加新 skill 时直接复制，未建立共享机制。后果：chmod 安全修复只对其中一个文件评估，另一个未得到修复（当前 HEAD 已验证）。

### 3.4 当时的优化机会
1. 把 `kubesphere-devops-credentials/SKILL.md` 第 111、134、469 行的 `"type": "Opaque"` 替换为正确的 `credential.devops.kubesphere.io/*` 类型（PR #6632 已执行）
2. 提取 `ks_api.py` 到共享位置，同时修复 `os.chmod(TOKEN_FILE, 0o600)`
3. 全局替换 `P@88w0rd` 为 `<YOUR_SECURE_PASSWORD>`

---

## 四、现在 vs 过去对比

> xiaolai 在 2026-05-07 提交了 5 个 PR（#6632–#6636），修复了 5 个机械 bug。xiaolai 案例有详细分析：[../2026-05-07-kubesphere-kubesphere.md](../2026-05-07-kubesphere-kubesphere.md)

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `Opaque` 类型与警告矛盾 | `grep Opaque skills/kubesphere-devops-credentials/SKILL.md` | **已修复（PR #6632）**：现在第一行明确警告 `CRITICAL: Must use credential.devops.kubesphere.io/* type, NOT Opaque!` | Jenkins 凭证同步问题已消除 |
| YAML 模板缺闭合引号（fluid）| `grep '{{low}}' skills/kubesphere-fluid/SKILL.md` | **已修复（PR #6633）**：现为 `low: "{{low}}"` | InstallPlan 不再解析失败 |
| Step 3 错误引用（whizard-logging）| `grep "Step 3\|Step 2" skills/whizard-logging/SKILL.md` | **已修复（PR #6634）**：安装节只有 Step 1/Step 2，无错误 Step 3 引用 | 无 |
| 缺闭合 `**`（devops-pipeline）| `grep "Step 3b" skills/kubesphere-devops-pipeline/SKILL.md` | **已修复（PR #6635）**：`**Step 3b: ...**` 闭合正确 | 无 |
| `Project Components` 重复（devops-overview）| `grep -c "Project Components" skills/kubesphere-devops-overview/SKILL.md` | **已修复（PR #6636）**：grep 计数 = 1 | 无 |
| 硬编码密码 `P@88w0rd` | `grep -c "P@88w0rd" skills/kubesphere-devops-tenant/SKILL.md` | **未修复**：仍有 4 处（lines 120、126、794、858）| 该 Medium 安全问题不在 5 个 PR 覆盖范围内 |
| `ks_api.py` 缺 `chmod 0o600` | `grep chmod skills/kubesphere-core/scripts/ks_api.py` | **未修复**：两个 ks_api.py 均无 chmod 调用 | token 文件仍对其他用户可读 |
| `xargs -n 3 sh -c` 注入（Helm hook）| `grep xargs config/ks-core/charts/ks-crds/scripts/post-delete.sh` | **未修复**：line 33/35 仍为 xargs sh -c 模式 | High 级 shell 注入模式仍存在 |

### 4.2 架构演进
审计快照 → 现 HEAD，NL 产物（21 skills）的主体结构未变，但：
- 新增了 3 个 skill：`frontend-forge-fe-operations`、`nodegroup`、`openpitrix`（审计时未覆盖）
- `kubesphere-network-extension-operations`、`kubesphere-servicemesh`、`whizard-notification`、`whizard-telemetry-ruler`、`wiztelemetry-tracing` 也是新加的 skill（现在共 30 个 skill 目录，审计只对 21 个评分）

说明：KubeSphere 的 AI 接入层在持续扩张，但现有 skill 的质量问题修复优先级低于新 skill 的添加。

### 4.3 新增的可学习模式
**多个新 skill 延续了「Output Contract」模式**：新增的 skill 中，部分也有类似 `opensearch/SKILL.md` 的输出契约声明——说明这个模式已经成为了 KubeSphere team 的内部规范。横向传播速度慢（只有少数 skill 实践），但方向正确。

---

## 五、校准

### 5.1 我已经在做对的
1. **示例密码不用真实格式**：我的 skill（如 claude-for-legal 的 cold-start-interview）在示例中用 `[YOUR_VALUE]` 格式而非外观真实的密码，不会被 AI 直接使用。
2. **不重复文件**：我的 skill 目录中没有完全相同的文件拷贝，避免了「修一处漏一处」的维护陷阱。
3. **说明文字与代码示例一致性**：我的 skill 在有代码示例的地方，说明文字和示例代码描述的是同一件事。

### 5.2 挑战 / 验证
本案例让我关注到一个之前没有意识到的模式：**「Output Contract」节**。我的 echo-sleuth/skills/jsonl-core/SKILL.md 定义了 JSONL 记录格式，但没有显式的「Output Contract」节——读者（和 AI）需要自己从正文中推断。加一个显式的 Output Contract 节会让跨 skill 的数据传递更可靠。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的 SKILL.md 中有没有真实格式的示例密码
grep -rn -E "(password|passwd|secret|token)\s*[:=]\s*['\"][^<{][^'\"]{5,}['\"]" \
  /tmp/my-repos/MarkQWu-*/skills/ 2>/dev/null | grep -iv "placeholder\|example\|your_\|changeme"
# 命中后：替换为 <YOUR_*> 格式的占位符
```

```bash
# 检查我的 skill 有没有说明文字和代码示例矛盾的情况
# （无法自动检测，手工 review 每个有 ⚠️WARNING 的 skill）
grep -rn "WARNING\|CRITICAL\|注意\|警告" /tmp/my-repos/MarkQWu-*/skills/ --include="SKILL.md" | \
  while IFS=: read f l text; do echo "手工检查 $f 附近是否有冲突代码示例"; done
```

```bash
# 检查我的 skill 有没有 Output Contract 节
grep -rL "Output Contract\|## Output\|## 输出" /tmp/my-repos/MarkQWu-*/skills/*/SKILL.md 2>/dev/null
# 命中后：考虑是否需要补充输出契约声明（对跨 skill 传递数据的场景尤其重要）
```

### 6.2 灵感 → 实施路径
1. **想法**：给 echo-sleuth 的 `jsonl-core` skill 添加「Output Contract」节，明确 JSON 输出的字段格式
   - **为何可行**：kubesphere 的 opensearch skill 证明了 Output Contract 对跨 skill 数据交换的价值；echo-sleuth 的 skills 之间也有数据传递（`git-mining` 的输出被 `experience-synthesis` 消费）
   - **第一步**：在 `skills/jsonl-core/SKILL.md` 末尾添加 `## Output Contract` 节，列出 JSONL 记录的必填字段名和类型，15 分钟可完成

2. **想法**：给 drama-workshop-skills 中有警告注释的地方做说明文字 vs 代码示例一致性检查
   - **为何可行**：本案例揭示了「警告文字在代码示例中被覆盖」是一个高频且高危的模式
   - **第一步**：找到所有包含「⚠️」「注意」「WARNING」的 skill，手工验证后面的代码示例是否与警告一致，30 分钟可完成

---

## 七、对照我的 GitHub 仓库

> 数据源：`learning/my-repos.json`

### 7.1 目的对齐度

- **本案例 kubesphere/kubesphere 的核心目的**：为 KubeSphere 多集群 K8s 平台的 AI 运维提供精确的平台特定 skill，覆盖 DevOps/可观测性/多租户/扩展管理。

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/claude-for-legal | 中 | 同为领域专精的 skill 集合（有大量平台特定知识），skill 数量多（20+） | 法律领域 vs K8s 运维；claude-for-legal 有 practice-area 分层，kubesphere 按子系统分层 | 高 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同为 skill 集合，有跨 skill 数据传递 | 通用工具 vs 领域专精 | 中 |
| MarkQWu/drama-workshop-skills | 低 | 同有多 skill 复杂结构 | 创意领域 vs 技术运维 | 低 |

### 7.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 代码示例与说明文字矛盾 | 手工检查含 WARNING 的 skill | `claude-for-legal` 的 `written-consent/SKILL.md` 有 `[None / Director Name]` 占位符在代码块旁——待核实是否与说明一致 | 中 |
| 硬编码真实格式密码/凭证 | `grep -rn "P@\|password.*=" /tmp/my-repos/MarkQWu-*/skills/` | **0 命中**：我的 skill 无此类凭证 | 无 |
| 完全重复的文件 | `find /tmp/my-repos/MarkQWu-* -name "*.py" \| xargs md5sum \| sort \| uniq -d` | **0 命中**：无重复文件 | 无 |

**命中后具体行动建议**：
- `claude-for-legal/corporate-legal/skills/written-consent/SKILL.md` → 核实 `[None / Director Name]` 所在段落的说明文字是否与代码示例一致，5 分钟可完成

### 7.3 别人的更优方案

1. **领域**：路由 skill（routing skill）模式
   - **本案例做法**：`kubesphere-core/SKILL.md` 的 description 就是路由表，直接告知模型「多集群问题→cluster-management，多租户问题→multi-tenant-management」，模型无需猜测
   - **我的项目现状**：`claude-for-legal` 有多个 practice area（in-house/regulatory/litigation 等），但没有一个路由 skill 声明「遇到合同问题用哪个 skill，遇到隐私问题用哪个 skill」
   - **如何借鉴**：新建 `claude-for-legal/skills/legal-router/SKILL.md`，description 明确列出场景→skill 的映射表；或在主 `CLAUDE.md` 中增加一个 skill 路由索引节

2. **领域**：Output Contract 节
   - **本案例做法**：`opensearch/SKILL.md` 有专门的 Output Contract 节，定义返回格式的字段名和类型
   - **我的项目现状**：`echo-sleuth-for-claude/skills/jsonl-core/SKILL.md` 定义了 JSONL 格式但没有专门的 Output Contract 节
   - **如何借鉴**：在 `jsonl-core/SKILL.md` 末尾添加 `## Output Contract`，声明下游 skill 可以依赖的输出字段

### 7.4 反向：我的项目做得比他们好的地方
- **领域**：安全敏感 skill 的凭证处理
- **我的做法**：`echo-sleuth-for-claude` 和 `claude-for-legal` 中所有涉及 API 密钥、密码的示例均用 `<YOUR_API_KEY>`、`[REPLACE_ME]` 格式，不出现外观真实的凭证
- **本案例做法**：`P@88w0rd` 出现 4 次（audit 后 3 个月仍未修复），外观真实的示例密码给 AI 错误信号
- **意义**：法律领域的 skill 会涉及敏感的 API endpoint 和认证信息（如 Ironclad、DocuSign），我的凭证占位符风格可以防止类似风险

---

## 八、术语表

### <a name="output-contract"></a>Output Contract
> Skill 文件中用一个独立章节声明「本 skill 产出的数据格式」：字段名、类型、必填 / 选填状态。目的是让调用本 skill 输出结果的下游 skill 或 command 可以可靠地解析数据，而不是自己猜测格式。类比编程中的接口定义（interface），但是自然语言版本。

### <a name="routing-skill"></a>路由 skill
> 一种特殊的 skill，其主要功能是「判断用户的问题属于哪个子系统，然后指向对应的专属 skill」。本身不包含深度技术内容，只有分发规则。对于大型 skill 集合（10+ skill），路由 skill 避免了模型在不确定时随机选一个 skill 的问题。

### <a name="kubectl"></a>kubectl
> Kubernetes 的命令行工具，用于与 K8s 集群交互（查询资源、修改配置、删除对象等）。kubesphere 的多个 skill 脚本中出现的 `kubectl get`、`kubectl patch` 等都是 kubectl 命令。

### <a name="xargs-injection"></a>xargs sh -c 注入
> 一种 shell 安全风险。`xargs -n N sh -c '...$0...'` 把上游命令（此处为 kubectl）的输出作为位置参数传递给 shell 字符串。如果上游输出的内容（如 Kubernetes 资源名）包含 shell 特殊字符（分号、引号等），可能改变 shell 命令的语义，执行非预期操作。通常用 `while read` 循环替代，避免字符串插值。

### <a name="helm-hook"></a>Helm hook
> Kubernetes 的 Helm 包管理器在特定事件（install/delete/upgrade）发生时自动执行的脚本，存放在 `templates/` 下带有 `helm.sh/hook` 注解的资源中。kubesphere 的 `post-delete.sh` 是在 Helm chart 卸载后自动清理 CRD finalizer 的 hook 脚本。
