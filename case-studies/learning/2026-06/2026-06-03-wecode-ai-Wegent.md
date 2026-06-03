# wecode-ai/Wegent — 学习案例

**仓库**：https://github.com/wecode-ai/Wegent
**Stars**：521 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-20（历史快照）| **生成日期**：2026-06-03（基于当前 HEAD）
**主题标签**：`vague-quantifier`, `security-gate`, `manifest-discipline`, `single-purpose`, `nl-binary-hybrid`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览
Wegent 是一个开源的 AI 原生操作系统，面向企业和个人开发者，允许用户以可视化方式定义、组织并运行多个 AI 智能体协同工作。关键事实：
- 全栈应用（FastAPI 后端 + Next.js 前端），依赖 Docker 部署
- Claude Code 技能层作为附加组件（`backend/init_data/skills/`），不是主产品
- 支持 Claude 和 Gemini 双模型后端
- 当前版本 1.0.20，有独立文档站点（wecode-ai.github.io/wegent-docs）
- 属于"AI 操作系统"赛道，与 Dify、Coze 同类但以 Claude Code 生态为特色

### 1.2 架构剖析
```
wecode-ai/Wegent/
├── backend/                    # FastAPI 后端（Python）
│   └── init_data/skills/       # Claude Code 技能定义（NL 层）
│       ├── browser/SKILL.md
│       ├── mermaid-diagram/SKILL.md
│       ├── prompt-optimization/SKILL.md
│       ├── sandbox/SKILL.md
│       ├── skill-creator/SKILL.md  ← 核心：教用户建技能
│       └── ...（共 11 个技能）
├── frontend/                   # Next.js 前端
├── executor/                   # Python 执行器（代码沙箱）
├── scripts/                    # 运维脚本（13 个 Shell + 8 个 hook 脚本）
├── docker-compose.yml          # 容器编排
└── tests/CLAUDE.md             # 测试说明文档
```
- **文件类型分布**：11 个 SKILL.md，0 个 agent 定义文件，0 个 command 文件，1 个 tests/CLAUDE.md
- **编排关系**：技能是平列关系，没有 router 或 meta skill；`skill-creator` 地位特殊（引导用户生成其他技能）
- **跨件契约**：技能之间无显式依赖；`sandbox/SKILL.md` frontmatter 包含 `api_key: "xxxxx"` 占位符，暗示运行时需要用户自行填充凭证

### 1.3 设计思路 / 方法论
- **核心设计哲学**：将 Claude Code 技能作为 Wegent 平台的"分发单元"——用户安装 Wegent 时，技能随着平台一起部署，成为平台功能的自然延伸
- **解决什么问题**：在企业多智能体场景中，单个 Claude 对话不够用；Wegent 提供了持久化任务状态、团队协作、可视化调试的基础设施层
- **做了什么 trade-off**：把"可以运行 Claude Code"当作附加功能，而不是产品核心。这导致技能文件的质量管理不是主线优先级，6 个技能缺少 `name` frontmatter 这类基础错误才得以长期存在
- **反映什么认知模型**：作者把技能看作"附赠教程"而非"一等公民"，与 autopilot 和 addyosmani 把技能当作核心产品的态度形成鲜明对比

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类
**模式名：NL 技能层 + 全栈应用内核**

主产品是一个完整的 Web 应用（有 DB、有 API、有前端），Claude Code 技能只是这个应用提供的一种工具/接口形式。

模式特征清单：
- 特征 1：技能不是独立存在的，它们的生命周期与主应用绑定
- 特征 2：技能文件存放在应用的数据初始化目录（`init_data/`），像配置文件而非独立工件
- 特征 3：最复杂的逻辑在 Python/JS 代码中，技能只做"入口说明"
- 特征 4：`skill-creator` 作为元技能引导用户生成其他技能，形成平台飞轮

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要持久化状态的多智能体系统 | ✅ 高度适用 | 全栈基础设施支持任务队列、数据库、会话管理 |
| 纯粹的 Claude Code 插件（给个人用） | ❌ 不适用 | 过度工程，99% 的用例不需要 Docker + 前端 |
| 企业内部 AI 工具平台 | ✅ 适用 | 有权限管理、可视化界面和部署方案 |
| NL 技能质量要求高的项目 | ⚠️ 需谨慎 | 技能是二等公民，容易疏于维护 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL 技能层 + 全栈应用内核（本仓库） | Wegent | 功能完整，适合企业交付 | 技能质量往往被忽视 |
| 纯 NL 插件（无应用层） | addyosmani/agent-skills | 技能是核心，质量高，部署简单 | 无持久化，无多用户 |
| NL 表皮 + 原生二进制核心 | 777genius/claude-notifications-go | 性能好，跨平台 | 开发门槛高，NL 层易变薄 |

### 2.4 改进空间
1. **当前问题**：`skill-creator` 文档里教用户的 frontmatter 只有 `name` + `description` 两个字段，但 Wegent 自己的技能用 8+ 个字段（`displayName`, `version`, `bindShells` 等）。这是一个"误导性文档"陷阱。**改进做法**：更新 `skill-creator/SKILL.md` 中的 frontmatter 模板，展示实际的完整 8 字段 schema。**预期收益**：用户创建的技能可被 Wegent 技能加载器正确解析。
2. **当前问题**：6 个技能只有 `displayName` 没有 `name`，但代码里查 SKILL.md 的工具是按 `name` 字段查找的。**改进做法**：批量添加 `name: xxx` 字段（按目录名取 kebab-case 值）。**预期收益**：技能注册不再静默失败。
3. **当前问题**：`sandbox/SKILL.md` 把 `api_key: "xxxxx"` 这样的占位符写在 frontmatter 里，会引导用户把真实密钥放进配置文件提交到 git。**改进做法**：改为在正文用 `{{ VARIABLE_NAME }}` 风格说明要从环境变量注入。

---

## 三、过去审查发现（2026-04-20 历史快照）

### 3.1 当时质量评分（NLPM）
该仓库 2026-04-20 当时得分 **81/100**，安全扫描结论 **BLOCKED**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `backend/init_data/skills/mermaid-diagram/SKILL.md` | 67 | 缺 `name` frontmatter（-25） |
| `backend/init_data/skills/wiki_submit/SKILL.md` | 68 | 缺 `name` frontmatter（-25） |
| `backend/init_data/skills/sandbox/SKILL.md` | 69 | 缺 `name` frontmatter（-25） |
| `backend/init_data/skills/skill-creator/SKILL.md` | 83 | 模糊量词（"effective", "appropriate"）密集出现 |
| `backend/init_data/skills/subscription-manager/SKILL.md` | 96 | 轻微模糊语言 |
| `backend/init_data/skills/interactive-form-question/SKILL.md` | 98 | 最小问题 |

### 3.2 当时值得借鉴的模式
1. **`skill-creator` 元技能**：把"如何创建技能"本身做成一个技能，这是平台飞轮设计。借鉴：我的 `legal-builder-hub` 已经在做类似的事（`skills-qa/SKILL.md`），但可以更明确地引导用户创建新技能。
2. **`interactive-form-question/SKILL.md`（98 分）**：该技能是最高质量的，说明交互式多步骤表单类技能很容易写干净，因为交互步骤天然有明确的输入输出。
3. **`subscription-manager/SKILL.md`（96 分）**：尽管其他技能质量差，Wegent 的订阅管理技能写得很好，说明作者对高频交互场景有清晰认知。

### 3.3 当时的缺陷
1. **6 个技能缺 `name` frontmatter**（根本原因：把 `displayName` 误当成了注册用的字段，作者没仔细读 Claude Code 的 SKILL.md schema）。自查：我的技能用 `name:` 还是别的字段注册？→ 需要 grep 验证。
2. **`skill-creator` 教错了 frontmatter 模式**：作者文档和实际技能用的 schema 不一致，会误导下游用户写出不兼容的技能。自查：我的 `legal-builder-hub/skills/skill-installer` 是否也有类似的 schema 文档和实际不符的情况？
3. **CRITICAL 安全问题 `scripts/run-e2e-local.sh:122`**：`curl -LsSf https://astral.sh/uv/install.sh | sh`，无 checksum 验证，在 CI/测试脚本中执行。根本原因：测试脚本被视为"只有开发者运行"而忽略了供应链安全。
4. **HIGH `scripts/build-standalone.sh:94`**：`eval $BUILD_CMD`，BUILD_CMD 由 CLI 参数组装，存在命令注入风险。

### 3.4 当时的优化机会
1. 批量修复 6 个 SKILL.md 的 `name:` 缺失（5 分钟可完成）
2. 更新 `skill-creator` 的 frontmatter 模板，对齐实际 schema（10 分钟）
3. 去除 `sandbox/SKILL.md` 中的 `api_key: "xxxxx"` 占位符凭证（2 分钟）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| 6 个技能缺 `name` frontmatter | `grep -rL "^name:" backend/init_data/skills/*/SKILL.md` | **仍然存在（6 个文件命中）** | 2 个月过去，Audit 的 bug 报告未被修复 |
| `curl \| sh` in run-e2e-local.sh:121 | `grep "curl.*\|.*sh" scripts/run-e2e-local.sh` | **仍然存在（line 121）** | 安全 CRITICAL 未处理 |
| `eval $BUILD_CMD` in build-standalone.sh:94 | `grep "eval.*BUILD_CMD" scripts/build-standalone.sh` | **仍然存在** | 安全 HIGH 未处理 |
| 硬编码密码 `123456` | `grep -rn "123456" scripts/` | **仍然存在（2 处）** | `fix-missing-project-id.sh` 和 `run-e2e-local.sh` 都还是默认密码 |

### 4.2 架构演进
SKILL.md 文件数量和结构无明显变化（仍为 11 个技能，目录结构一致）。README 显示版本号已升至 1.0.20（2026-04-20 时未知，但目录中已有的 scripts 数量没变）。这说明项目在持续迭代主应用功能，但 Claude Code 技能层维护频率极低。

### 4.3 新增的可学习模式
`skill-creator` 现在有完整的 `name:` 和 `description:` frontmatter（`name: skill-creator`），是 11 个技能中少数正确的之一。这可能是作者为了让 `skill-creator` 本身可以被注册而手工修了这一个，但忘了统一修其他技能——暗示"先修自己最常用的那个"的临时修复模式。

---

## 五、校准

### 5.1 我已经在做对的
1. 我的所有 SKILL.md 文件均有 `name:` 字段（grep 验证：`find /tmp/my-repos/ -name "SKILL.md" | xargs grep -L "^name:"` 返回空）
2. 我的脚本没有 `eval $VAR` 这类明显的命令注入写法
3. 我的技能文件不包含硬编码凭证占位符

### 5.2 挑战 / 验证
本案例挑战了我一个假设：**"企业级、高 Stars 项目的 NL 技能质量会更高"**。Wegent 有 521 星、1.0.20 版、完整文档站，但 NL 技能层 81 分且 2 个月后所有缺陷仍然存在。验证：NL 技能质量与项目知名度无关，只与维护者是否把技能当作"核心交付物"有关。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的技能是否有 name: 字段
find ~/.claude/skills -name "SKILL.md" | xargs grep -L "^name:"
# 命中后：立即添加 name: 字段，用目录名的 kebab-case 形式

# 检查我的脚本是否有 eval $VAR 模式
find . -name "*.sh" | xargs grep -n "eval \$[A-Z_]*" 2>/dev/null
# 命中后：替换为数组形式 cmd=($VAR); "${cmd[@]}"

# 检查技能文档中的 schema 是否与实际技能一致
grep -r "frontmatter\|schema\|fields:" */skills/skill-*.md 2>/dev/null
# 命中后：对比实际使用的字段清单，更新文档中的模板
```

### 6.2 灵感 → 实施路径

1. **想法**：在我的 `legal-builder-hub/skills/skills-qa` 里加一个 frontmatter schema 检查清单
   - **为何可行**：`skills-qa` 本来就是审计技能质量的，添加 "required fields" 检查是自然延伸
   - **第一步**：打开 `MarkQWu-claude-for-legal/legal-builder-hub/skills/skills-qa/SKILL.md`，在输出格式里增加一行 `- [ ] name: field present`，10 分钟可完成

2. **想法**：建立一个"技能 schema 约定"文档放在每个插件仓库根目录
   - **为何可行**：Wegent 的 `skill-creator` 教错了 schema，说明没有统一的参考文档
   - **第一步**：在 `MarkQWu-echo-sleuth-for-claude/` 根目录创建 `SKILL-SCHEMA.md`，列出必填字段清单

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **wecode-ai/Wegent 的核心目的**：多智能体操作系统平台，Claude Code 技能是其功能扩展层
- **我的对应项目**：

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/drama-workshop-skills | 低 | 同样有 SKILL.md 文件 | drama 是纯 NL 插件，Wegent 是全栈平台 | 低 |
| MarkQWu/claude-for-legal | 中 | 都是面向特定专业领域的工具套件 | claude-for-legal 纯 NL，Wegent 有完整应用层 | 中 |
| MarkQWu/echo-sleuth-for-claude | 低 | 同样有 commands + skills | echo-sleuth 是单一插件，Wegent 是平台 | 低 |

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| 技能缺 `name:` frontmatter | `find /tmp/my-repos -name "SKILL.md" \| xargs grep -L "^name:"` | **未命中**（我的全部 SKILL.md 有 name 字段）| N/A |
| 模糊量词泛滥 | `find /tmp/my-repos -name "SKILL.md" \| xargs grep -lE '\b(appropriate\|relevant\|sufficient)\b'` | **命中 100+ 个文件**（echo-sleuth 4/4 技能，claude-for-legal 几乎全部命中）| 高 |
| 命令缺 `allowed-tools` | `for f in commands/*.md; do grep -c "allowed-tools" "$f" \|\| echo "MISSING: $f"; done` | **echo-sleuth 4 个 commands 命中（recall.md, recap.md, timeline.md, lessons.md 缺 allowed-tools）** | 中 |

**命中后的具体行动建议**：
- `MarkQWu-echo-sleuth-for-claude/commands/recall.md` → 在 frontmatter 添加 `allowed-tools: Bash, Read`，5 分钟
- `MarkQWu-echo-sleuth-for-claude/commands/recap.md` → 同上
- `MarkQWu-echo-sleuth-for-claude/commands/timeline.md` → 需要先确认该 command 实际调用哪些工具，再补充
- `MarkQWu-claude-for-legal/*/skills/*.md` 中的 `relevant`（关联性）实例 → 逐一替换为具体的选择标准，如"如果问题涉及 GDPR 第 6 条合法依据，则..."

### 8.3 别人的更优方案

1. **领域**：`skill-creator` 元技能设计
   - **本案例做法**：Wegent 提供了一个专门引导用户创建技能的元技能（`skill-creator/SKILL.md`），将平台的扩展机制本身文档化为可交互的工具
   - **我的项目现状**：`MarkQWu-claude-for-legal/legal-builder-hub/skills/skill-installer/SKILL.md` 有类似想法但更侧重"安装"而不是"创建引导"
   - **如何借鉴**：给 `skill-installer` 增加一个"创建新技能"的交互路径，引导用户回答 5 个问题后生成一个技能草稿

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：技能 frontmatter 完整性
- **我的做法**：`MarkQWu-echo-sleuth-for-claude/skills/` 下所有 SKILL.md 均有完整的 `name:` 和 `description:` 字段
- **本案例做法（弱在哪）**：Wegent 6/11 技能缺 `name:` 字段，导致技能注册静默失败
- **意义**：这是基础合规点，若有人审查我的仓库，这是正向亮点

---

## 八、术语表

### <a name="eval注入"></a>eval 注入
> Shell 的 `eval` 命令把一个字符串当作代码来执行。当字符串由用户输入或可变量拼接而成时，攻击者可以在字符串里插入恶意命令，让系统在 eval 时执行它。类比：把一张"请读出这张纸上的内容"的纸条传给一个会高声朗读任何内容的机器人——如果纸条内容是"打开前门"，机器人就真的去开门了。

### <a name="curl-pipe-bash"></a>curl | bash 模式
> 一种安装脚本写法：`curl https://example.com/install.sh | bash`，即从互联网下载一个脚本并立刻用 bash 执行它。方便，但危险——你无法在执行前检查脚本内容，如果托管地址被攻击者控制，任意代码都会在你的机器上运行。

### <a name="供应链攻击"></a>供应链攻击
> 不直接攻击目标系统，而是攻击目标系统依赖的上游工具、库或服务器。比如攻击者污染了 npm 上的一个热门包，凡是安装这个包的项目都会"中招"。`pip install git+https://github.com/xxx/yyy.git`（从 git HEAD 安装，不固定版本）是典型的供应链风险：仓库任何时候被攻击，下次安装就会拉到恶意代码。

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，声明文件的元数据。Claude Code 读 SKILL.md 时先解析 frontmatter 获取技能名称、描述和权限配置。如果 `name:` 字段缺失，Claude Code 找不到这个技能。
