# ZeframLou/call-me — 学习案例

**仓库**：https://github.com/ZeframLou/call-me
**Stars**：2590 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-06（历史快照）| **生成日期**：2026-05-27（基于当前 HEAD）
**主题标签**：`nl-binary-hybrid`, `vague-quantifier`, `manifest-discipline`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

ZeframLou/call-me 是一个让 Claude Code 在停止时主动给用户打电话的插件。核心场景：你让 Claude 做一个耗时任务然后离开，Claude 完成后或遭遇阻塞时拨打你的手机，通过语音与你交互，而不是等你回来看屏幕。

关键事实：

1. **NL + 二进制混合架构**：插件包含自然语言层（`skills/phone-input/SKILL.md` + `hooks/hooks.json`）和 TypeScript/Bun MCP 服务器（`server/`），两层各司其职
2. **双电话运营商支持**：同时支持 Twilio 和 Telnyx 两家电话 API，用户可按需选择
3. **双 TTS 引擎**：支持 OpenAI 云端 TTS 和 Kokoro 本地 TTS，后者在离线环境下仍可使用
4. **4 个 MCP 工具**：`initiate_call`、`continue_call`、`speak_to_user`、`end_call`，覆盖完整通话生命周期
5. **Stop [钩子](#hook)驱动**：`hooks/hooks.json` 注册一个 `Stop` 类型 prompt 钩子，在 Claude 即将停止时评估是否应打电话

插件版本：plugin.json 1.0.3 / server 1.0.2

### 1.2 架构剖析

目录结构：
```
call-me/
├── .claude-plugin/
│   └── plugin.json          # 清单：注册技能 + MCP 服务器配置
├── skills/
│   └── phone-input/
│       └── SKILL.md         # 自然语言层：何时打电话 + 工具说明
├── hooks/
│   └── hooks.json           # Stop 钩子：触发打电话的决策逻辑
└── server/
    ├── package.json         # Bun 运行时依赖
    ├── bun.lock             # 锁定文件（已提交，保证可复现性）
    ├── src/
    │   ├── index.ts         # MCP 服务器入口，暴露 4 个工具
    │   └── providers/
    │       ├── phone-twilio.ts      # Twilio 电话提供者
    │       ├── phone-telnyx.ts      # Telnyx 电话提供者
    │       ├── tts-openai.ts        # OpenAI TTS 提供者
    │       ├── tts-kokoro.ts        # Kokoro 本地 TTS 提供者
    │       └── stt-openai-realtime.ts  # OpenAI 实时 STT 提供者
    └── .env.example         # 配置示例（展示 CALLME_* 命名空间）
```

**编排流程**：用户任务结束 → `hooks/hooks.json` 中的 Stop 钩子触发 → Claude 根据 SKILL.md 中的判断规则评估是否应打电话 → 若是，调用 `initiate_call` → [MCP 服务器](#mcp-server)通过 Twilio/Telnyx API 发起呼叫 → ngrok 隧道将回调路由到本地服务器 → 持续通话直至 `end_call`

**凭据命名规范**：`plugin.json` 中所有环境变量使用 `CALLME_*` 前缀（如 `${CALLME_TWILIO_ACCOUNT_SID}`），防止与其他插件的变量名冲突。

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「NL 层做决策，二进制层做执行」——Stop 钩子和 SKILL.md 负责判断「该不该打」「什么时候打」，MCP 服务器负责「怎么打」。两层职责不重叠
- **解决什么问题**：长任务「完成了没人知道」的问题。传统方案是桌面通知，但用户可能不在电脑旁；本插件把通知渠道拓展到手机语音通话，且支持双向交互（用户说话 → STT → Claude 继续执行）
- **Trade-off**：引入 [ngrok](#ngrok) 隧道意味着每次启动服务器都会开一个新的公开 URL，外部可达但地址会变。这是 Twilio webhook 回调的架构需求，属于「有意为之的安全风险」而非疏忽
- **认知模型**：Stop 钩子是「守门人」，决定是否要打电话；SKILL.md 是「规则书」，提供打电话的场景判断准则；MCP 服务器是「电话机器」，执行实际通话

---

## 二、架构作为可学习模式（横向）

### 2.1 模式归类

**模式名：NL 决策层 + 二进制执行层（NL-Binary-Hybrid）**

与「纯 NL 插件」不同，这个仓库把「决策逻辑」和「系统操作」严格分层：SKILL.md + hooks.json 用自然语言描述「什么情况下做什么」，MCP 服务器用 TypeScript 实现「怎么做」。NL 层无需关心电话 API 细节，服务器层无需关心打电话的语义判断。

模式特征清单（4 条）：

- 特征 1：hooks.json 使用 `type: "prompt"` 而非 `type: "command"`，钩子逻辑完全在 LLM 推理层，无 shell 执行面
- 特征 2：SKILL.md 通过文档化 4 个 MCP 工具的用途和参数，让 Claude 知道「能调用什么」，而不是「如何实现」
- 特征 3：providers/ 目录将电话运营商（Twilio/Telnyx）和 TTS/STT 引擎解耦，新增提供者只需实现同一接口
- 特征 4：`plugin.json` 使用 `${CALLME_*}` 环境变量插值传递凭据，MCP 服务器配置与安装时凭据完全分离

### 2.2 适用场景

| 场景 | 适不适用 | 原因 |
|---|---|---|
| 需要触达用户「不在电脑旁」的场景 | ✅ 高度适用 | 电话通知优于桌面通知，覆盖离开状态 |
| 任务耗时 >30 分钟、结果不确定 | ✅ 适用 | Claude 阻塞时能主动求助，避免无声等待 |
| 需要「执行层」调用外部 API | ✅ 适用 | NL-Binary-Hybrid 模式适合所有需要 API 调用的插件 |
| 无法启动本地服务器的环境（纯云 IDE）| ❌ 不适用 | MCP 服务器 + ngrok 需要本地进程，无法在无本地运行时的环境中使用 |
| 不愿暴露 ngrok 公开 URL 的高安全环境 | ⚠️ 慎用 | ngrok 隧道每次启动都开一个新的公开端点，需评估可接受性 |

### 2.3 与其他架构对比

| 架构 | 典型代表 | 优势 | 劣势 |
|---|---|---|---|
| NL-Binary-Hybrid（本仓库）| ZeframLou/call-me | NL 层专注语义决策；执行层能调用任意外部 API；providers/ 解耦 | 需要本地服务器运行时；ngrok 隧道引入外部依赖 |
| 纯 NL 无服务器 | BayramAnnakov/claude-reflect | 零依赖安装；无运行时 | 无法执行系统级或外部 API 操作 |
| 纯 NL + 系统脚本 | SimoneAvogadro/android-reverse | 脚本可做系统操作；无长驻进程 | 无法维持状态；无法做双向通信 |

### 2.4 改进空间

1. **当前问题**：`skills/phone-input/SKILL.md` 缺少 [frontmatter](#frontmatter) 块，文件直接以 `# Phone Call Input Skill` 标题开头，没有 `---` 分隔符，也没有 `name` 和 `description` 字段。**改进做法**：在文件第 1 行插入完整 frontmatter（详见 §3.3）。**预期收益**：Claude Code 技能加载器能正确索引该技能，`[[skill-ref]]` 交叉引用和 `/help` 技能发现均恢复正常

2. **当前问题**：`hooks/hooks.json` 中存在 5 个模糊量词（"significant work"、"genuinely blocked"、"Trivial tasks"等），导致 Stop 钩子在模糊边界上行为不一致。**改进做法**：替换为可操作的具体描述（详见 §3.4 优化建议）。**预期收益**：钩子触发条件可预测，减少「该打没打 / 不该打却打」的误判

---

## 三、过去审查发现（2026-04-06 历史快照）

### 3.1 当时质量评分（NLPM）

该仓库 2026-04-06 审查得分 **80/100**（3 个文件，加权平均）。安全扫描：CLEAR。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| `skills/phone-input/SKILL.md` | 46/100 | 缺 `name` frontmatter（−25）+ 缺 `description` frontmatter（−25）+ 2 个模糊量词（−4）|
| `hooks/hooks.json` | 94/100 | 3 个 R01 模糊量词（−6）|
| `.claude-plugin/plugin.json` | 100/100 | 无问题 |

加权计算：(46 + 94 + 100) / 3 = **80/100**

### 3.2 当时值得借鉴的模式

1. **plugin.json 100/100 完美清单** → `.claude-plugin/plugin.json` 的所有必需字段（name、description、version、skills、mcpServers）均正确填写，mcpServers 配置规范，凭据全部通过环境变量传入 → `.claude-plugin/plugin.json` → 借鉴：清单文件是插件的「对外契约」，应该是插件中质量最高的文件，优先做到满分

2. **prompt 类型钩子无 shell 执行面** → `hooks/hooks.json` 使用 `"type": "prompt"` 而非 `"type": "command"`，整个 Stop 决策逻辑在 LLM 推理层完成，没有任何 shell 命令执行 → `hooks/hooks.json` → 借鉴：当钩子逻辑可以用自然语言描述时，优先用 prompt 类型，避免引入 shell 执行面带来的安全风险

3. **环境变量命名空间化（CALLME_\* 前缀）** → `plugin.json` 中所有凭据变量统一用 `CALLME_` 前缀（如 `CALLME_TWILIO_ACCOUNT_SID`、`CALLME_NGROK_AUTHTOKEN`），防止变量名与其他插件或系统变量冲突 → `.claude-plugin/plugin.json` → 借鉴：MCP 服务器插件的环境变量应使用插件专属前缀，建议格式 `<PLUGIN_SLUG>_<SERVICE>_<KEY>`

4. **bun.lock 已提交，可复现性有保障** → `server/bun.lock` 已提交到仓库（当前 HEAD），确保每次 `bun install` 安装相同版本集 → `server/bun.lock` → 借鉴：有本地服务器的插件应将锁定文件（package-lock.json / bun.lock / poetry.lock）提交到仓库，让安装结果可复现

### 3.3 当时的缺陷

1. **SKILL.md 缺少 `name` frontmatter**（影响：技能索引失败，无法被 `[[skill-ref]]` 引用，无法在 `/help` 中显示）→ 根本原因：作者可能认为文件名（`SKILL.md`）已足够标识技能，但 Claude Code 的技能加载器依赖机器可读的 frontmatter 字段 → 自查：检查我的所有 SKILL.md 是否都有 `name` 字段

2. **SKILL.md 缺少 `description` frontmatter**（影响：触发词解析失败，用户问「帮我打电话」时插件不会被触发）→ 根本原因：同上，缺少对 frontmatter 规范的了解 → 两个 bug 都可用一次 edit 修复：

```yaml
---
name: phone-input
description: "Call the user for real-time voice input, status reporting, or blocked-task clarification via Twilio phone calls."
---
```

3. **`hooks/hooks.json` 的 3 个模糊量词**（"significant work"、"genuinely blocked"、"Trivial tasks"）→ 根本原因：钩子提示词描述了「意图」但没有给出「判断标准」，LLM 对「significant」的理解因上下文而异，导致触发条件不稳定 → 自查：我的 hooks.json（若有）里有没有同类模糊描述？

4. **SKILL.md 的 2 个模糊量词**（SKILL.md 第 9 行 "completed a significant task"、第 12 行 "complex decisions"）→ 根本原因：与 hooks.json 相同，用主观形容词代替了可量化的判断标准

### 3.4 当时的优化机会

1. **钩子提示词模糊量词替换**（优先级高，影响行为可靠性）：
   - "significant work" → "wrote or modified at least one file, or executed a command with observable side-effects"
   - "genuinely blocked" → "cannot proceed without a decision the user must make (e.g., choosing between two implementation approaches)"
   - "Trivial tasks" → "read-only tasks (file reads, searches, status checks)"

2. **服务器安全加固**（优先级中）：
   - 移除 `server/package.json` 中的 `"prestart": "bun install --silent"`（当前每次服务器启动都会自动拉包，结合 `^` 范围依赖，版本会漂移）
   - 将 `bun-types` 从 `"latest"` 改为具体版本号（如 `"^1.1.38"`）

---

## 四、现在 vs 过去对比

### 4.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| SKILL.md 缺 `name` frontmatter | `head -3 skills/phone-input/SKILL.md` | **仍存在**（文件以 `# Phone Call Input Skill` 开头，无 `---` 块）| 技能索引至今仍失效 |
| SKILL.md 缺 `description` frontmatter | 同上 | **仍存在** | 触发词解析至今仍失效 |
| `prestart` 自动安装 | `grep prestart server/package.json` | **仍存在**（`"prestart": "bun install --silent"`）| 每次服务器启动都拉包，可能引入版本漂移 |
| `bun-types: "latest"` 未固定 | `grep bun-types server/package.json` | **仍存在**（`"latest"`）| 安装时版本完全不可预测 |
| hooks.json 模糊量词 | `grep "significant\|genuinely\|Trivial" hooks/hooks.json` | **仍存在**（同一提示文本）| 钩子触发边界仍不明确 |

**结论**：2026-04-06 审查发现的全部 bug 和主要质量问题，4 个月后均未被修复。

### 4.2 架构演进

与 audit 时相比，仓库有三项实质性改进：

1. **新增 Telnyx 提供者**（`server/src/providers/phone-telnyx.ts`）：从单一 Twilio 扩展到双运营商支持，providers/ 目录的抽象层被验证了——新增提供者只需实现同一接口，无需改动 MCP 入口
2. **新增 Kokoro 本地 TTS**（`server/src/providers/tts-kokoro.ts`）：离线可用，用户不必依赖 OpenAI TTS API，降低了运行成本和外部依赖
3. **`bun.lock` 已提交**：相比 audit 时，当前 HEAD 已将锁定文件纳入版本控制，可复现性从「可能漂移」提升到「锁定」

### 4.3 新增的可学习模式

- **Providers 目录扩展模式**：`server/src/providers/` 以「服务类型-提供商」命名（`tts-openai.ts`、`tts-kokoro.ts`、`phone-twilio.ts`、`phone-telnyx.ts`），接口统一、实现隔离，新增第三个运营商或 TTS 引擎时只需新建一个文件。这是有外部 API 依赖的 MCP 服务器的优秀可扩展性设计，值得借鉴。

---

## 五、校准

### 5.1 我已经在做对的

1. **我的 SKILL.md 有 frontmatter**（对比：call-me 的最高代价 bug）：echo-sleuth 的 `skills/memory-management/SKILL.md` 等文件均有 frontmatter 块，`name` 和 `description` 字段存在。这个细节直接影响技能是否能被正确加载，echo-sleuth 做对了
2. **无 prestart 自动安装**：echo-sleuth 没有 MCP 服务器，也没有 npm/bun 脚本，因此不存在 prestart 自动拉包的风险；claude-for-legal 同理
3. **没有 ngrok 类的常驻网络隧道**：我的三个仓库都不依赖 ngrok 或类似工具，不引入公开网络端点

### 5.2 挑战 / 验证

- **挑战了什么假设**：我之前以为 frontmatter 缺失是「低影响」问题——用户能看到技能内容，只是元数据缺失而已。事实是 NLPM 对缺少 `name` 和 `description` 各扣 25 分，直接把 SKILL.md 从满分打到 46/100，把整体分从 96+ 拉低到 80。frontmatter 不是「锦上添花」，是「有没有注册成功」的关键
- **这给我带来认知**：NL 层的 bug 不一定在内容里，而可能在结构里。一个缺失的 `---` 块可以让精心撰写的技能文档完全失效。这类「结构性 bug」比内容错误更难被肉眼发现，但影响最大。

---

## 六、行动

### 6.1 自查动作

```bash
# 检查我的所有 SKILL.md 是否都有 frontmatter（以 --- 开头）
for f in $(find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "SKILL.md"); do
  first=$(head -1 "$f")
  if [ "$first" != "---" ]; then
    echo "MISSING FRONTMATTER: $f (starts with: $first)"
  fi
done
# 命中后：为缺少 frontmatter 的 SKILL.md 文件添加 --- 块，补充 name 和 description 字段
```

```bash
# 检查我的 SKILL.md 是否有 name 和 description 字段
for f in $(find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "SKILL.md"); do
  has_name=$(grep -c "^name:" "$f" 2>/dev/null || echo 0)
  has_desc=$(grep -c "^description:" "$f" 2>/dev/null || echo 0)
  [ "$has_name" -eq 0 ] && echo "MISSING name: $f"
  [ "$has_desc" -eq 0 ] && echo "MISSING description: $f"
done
# 命中后：在 frontmatter 块内补充缺失字段；name 与文件夹名一致，description 一句话说清技能用途
```

```bash
# 检查我的 SKILL.md 是否含有常见模糊量词
grep -rn "significant\|genuinely\|trivial\|complex\|important\|meaningful" \
  /tmp/my-repos/MarkQWu-echo-sleuth-for-claude/skills/ 2>/dev/null
# 命中后：将模糊量词替换为可量化描述，例如「significant」→「包含至少一次文件写入或命令执行」
```

```bash
# 检查我的 hooks.json（若有）是否含有模糊量词
find /tmp/my-repos/MarkQWu-echo-sleuth-for-claude -name "hooks.json" -exec \
  grep -n "significant\|genuinely\|trivial\|complex" {} \;
# 命中后：参照 §3.4 优化建议替换为可操作描述
```

### 6.2 灵感 → 实施路径

1. **想法**：为 echo-sleuth 的 skills 添加 NLPM 风格的 frontmatter 完整性检查脚本
   - **为何可行**：echo-sleuth 有 4 个技能，当前不确定 `trigger` 字段是否完整，一个脚本可以一次性检查所有 SKILL.md 的 frontmatter 字段
   - **第一步**：在 `scripts/` 创建 `check-skill-frontmatter.sh`，检查每个 SKILL.md 的 `name`/`description`/`trigger` 字段存在性，输出 `MISSING:<field>:<file>` 格式，约 10 分钟

2. **想法**：如果 echo-sleuth 将来加入 MCP 服务器，采用 call-me 的 providers/ 抽象模式
   - **为何可行**：echo-sleuth 当前有 4 个不同知识来源（git 挖掘、session 解析等），如果封装为 API，providers/ 模式可以隔离不同数据源的实现
   - **第一步**：在技术方案设计时，在 `server/src/` 下规划 `providers/` 目录，先实现一个 provider，再扩展第二个，而不是把所有逻辑塞进 `index.ts`

---

## 七、对照我的 GitHub 仓库

### 8.1 目的对齐度

- **本案例 ZeframLou/call-me 的核心目的**：手机语音通话通知与交互，在 Claude 完成任务或阻塞时主动联系用户

| 我的仓库 | 相似度 | 相同点 | 不同点 | 借鉴优先级 |
|---|---|---|---|---|
| MarkQWu/echo-sleuth-for-claude | 低 | 都是 Claude Code 插件；都有多组件架构 | echo-sleuth 挖掘会话历史，无语音通话 | 低（架构模式可借鉴） |
| MarkQWu/claude-for-legal | 低 | 都有清单文件（plugin.json）| 法律工作流 vs 语音通话，功能无重叠 | 低（frontmatter 规范可借鉴）|
| MarkQWu/drama-workshop-skills | 无 | — | 创意写作 vs 语音通话 | 无 |

所有三个仓库与 call-me 的目的完全不同（无电话/语音功能），但 call-me 的**设计原则**（frontmatter 规范、环境变量命名、providers/ 抽象）普遍适用。

### 8.2 在我的项目里复现的同类问题

| 本案例缺陷 | 检查方法 | 我的项目命中情况 | 严重度 |
|---|---|---|---|
| SKILL.md 缺 frontmatter | `head -1 skills/*/SKILL.md` | 需按 §6.1 脚本实测，基于已知情况：echo-sleuth 有 frontmatter，但 `trigger` 字段和 `model` 字段完整性未验证 | 待验证 |
| 模糊量词（significant/complex）| `grep -n "significant\|complex" skills/*/SKILL.md` | echo-sleuth 4 个技能**未发现**严重模糊量词（无 significant/genuinely/trivial 命中）| 低（暂无问题） |
| 环境变量无命名空间 | `grep "env:" .claude-plugin/plugin.json` | echo-sleuth 无 MCP 服务器，不涉及 env 变量 | 不适用 |

### 8.3 别人的更优方案

1. **领域**：凭据环境变量命名空间化（`CALLME_*` 前缀模式）
   - **本案例做法**：所有凭据统一以 `CALLME_` 开头，在 `plugin.json` 中通过 `${CALLME_*}` 插值使用
   - **我的项目现状**：echo-sleuth 无 MCP 服务器，当前无环境变量需求；但 claude-for-legal 的 CLAUDE.md 路由模式若将来接入外部 API，没有预设的命名规范
   - **如何借鉴**：若 claude-for-legal 将来加入外部 API 调用，从第一天起使用 `CLEGAL_*` 前缀，避免后期因命名冲突被迫改变

2. **领域**：prompt 类型钩子（无 shell 执行面）
   - **本案例做法**：`hooks/hooks.json` 全部使用 `"type": "prompt"`，决策逻辑在 LLM 层，零 shell 命令
   - **我的项目现状**：echo-sleuth 当前无 hooks.json；若将来添加钩子，有参考先例
   - **如何借鉴**：编写钩子时优先使用 prompt 类型，只在必须执行 shell 命令时才使用 command 类型，减少安全风险

### 8.4 反向：我的项目做得比他们好的地方

- **领域**：SKILL.md frontmatter 完整性（最核心的结构规范）
- **我的做法**：echo-sleuth 的技能文件均有 frontmatter 块（基于已知信息），而 call-me 在 2590 星规模下仍有 SKILL.md 完全缺失 frontmatter 的 bug，4 个月未修复
- **本案例弱在哪**：`skills/phone-input/SKILL.md` 直接以 `# Phone Call Input Skill` 开头，NLPM 因此扣 50 分（25+25），把整体分从 ~96 拉到 80。这是一个「高影响、低成本修复」的 bug，却长期存在
- **意义**：即使是 2590 stars 的热门插件，最基础的结构规范也可能缺失。frontmatter 规范是「最容易验证、影响最大」的质量检查项，应该是任何 NL artifact 的第一道检查

---

## 八、术语表

### <a name="frontmatter"></a>frontmatter
> Markdown 文件最顶部用 `---` 包起来的一段 YAML 配置，用来声明文件的元数据。Claude Code 读取 SKILL.md 或 command 文件时先解析 frontmatter。对于 SKILL.md，`name` 字段用于技能索引，`description` 字段用于触发词解析，两者缺一不可。缺少 frontmatter 会导致技能「注册失败」——文件内容存在但插件系统无法识别该技能。NLPM 对缺少 `name` 和 `description` 各扣 25 分。

### <a name="mcp-server"></a>MCP 服务器
> Model Context Protocol Server，是一个独立进程，通过标准化协议向 Claude Code 暴露一组「工具」（tools）。Claude 可以调用这些工具执行本地或远程操作（如发起电话、调用外部 API）。在 call-me 中，MCP 服务器用 TypeScript/Bun 实现，在 `server/src/index.ts` 中注册 4 个工具（`initiate_call`、`continue_call`、`speak_to_user`、`end_call`），通过 `plugin.json` 的 `mcpServers` 字段自动启动。

### <a name="ngrok"></a>ngrok
> 一个网络隧道工具，在本地服务器和公开互联网之间创建一个临时的 HTTPS 端点。在 call-me 中，Twilio/Telnyx 发起通话后需要将语音回调（webhook）发送到 MCP 服务器，但服务器运行在用户本地机器上，没有公开 IP。ngrok 解决这个问题：本地服务器启动时同时启动 ngrok，获得一个类似 `https://abc123.ngrok.io` 的公开 URL，电话运营商把回调发到这里，ngrok 转发给本地服务器。代价是每次启动都生成新 URL，且该 URL 对外公开可达。

### <a name="hook"></a>钩子（hook）
> Claude Code 的钩子系统允许在特定事件（如任务停止、工具调用前后）时触发自定义逻辑。钩子有两种类型：`"type": "prompt"` 在 LLM 层执行（Claude 读取钩子提示词后决策），`"type": "command"` 在 shell 层执行（直接运行命令行指令）。call-me 的 `Stop` 钩子使用 prompt 类型：Claude 即将停止时，钩子提示词被注入到上下文，Claude 根据提示词判断是否应该调用 `initiate_call`，而非直接执行 shell 命令。
