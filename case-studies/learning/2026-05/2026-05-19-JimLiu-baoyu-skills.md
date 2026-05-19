# JimLiu/baoyu-skills — 学习案例

**仓库**：https://github.com/JimLiu/baoyu-skills
**Stars**：16303 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-29（历史快照）| **生成日期**：2026-05-19（基于当前 HEAD）
**主题标签**：`experience-accumulation`, `template-design`, `manifest-discipline`, `vague-quantifier`, `single-purpose`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

JimLiu/baoyu-skills 是一个面向中文内容创作者的 Claude Code 技能套件，涵盖图文生成、内容发布（微博/X/微信）、YouTube 字幕下载、图像压缩等 20+ 个技能。作者 JimLiu 是中国开发者社区知名技术博主，目前以 16303★ 成为 Claude Code 插件生态中最受欢迎的中文技能包之一。仓库于 2024 年随 Claude Code 插件体系兴起，以 npm monorepo 结构管理配套工具脚本，并通过 `scripts/sync-clawhub.sh` 保持与 ClawhHub 市场的同步。

### 1.2 架构剖析

```
baoyu-skills/
├── .claude-plugin/
│   ├── marketplace.json     # ClawhHub 市场元数据
│   └── (plugin.json)
├── CLAUDE.md                # 仓库级规范（代码风格 + 跨 skill 更新规则）
├── CHANGELOG.md / .zh.md    # 双语更新日志
├── docs/                    # 仓库内部参考文档（chrome-profile, image-gen 等）
├── packages/
│   ├── baoyu-chrome-cdp/    # Chrome DevTools Protocol 支持包
│   ├── baoyu-fetch/         # 网页抓取封装
│   └── baoyu-md/            # Markdown 处理工具
├── scripts/
│   └── sync-clawhub.sh      # 发布脚本
├── skills/
│   ├── baoyu-article-illustrator/SKILL.md
│   ├── baoyu-comic/SKILL.md
│   ├── baoyu-compress-image/SKILL.md
│   ├── baoyu-cover-image/SKILL.md
│   ├── baoyu-danger-gemini-web/SKILL.md  # 标记危险用途
│   ├── baoyu-danger-x-to-markdown/SKILL.md
│   ├── baoyu-diagram/SKILL.md
│   ├── baoyu-format-markdown/SKILL.md
│   ├── baoyu-image-cards/SKILL.md
│   ├── baoyu-image-gen/SKILL.md          # 已废弃 → baoyu-imagine
│   ├── baoyu-imagine/SKILL.md            # baoyu-image-gen 继任者
│   ├── baoyu-infographic/SKILL.md
│   ├── baoyu-markdown-to-html/SKILL.md
│   ├── baoyu-post-to-wechat/SKILL.md
│   ├── baoyu-post-to-weibo/SKILL.md
│   ├── baoyu-post-to-x/SKILL.md
│   ├── baoyu-slide-deck/SKILL.md
│   ├── baoyu-translate/SKILL.md
│   ├── baoyu-url-to-markdown/SKILL.md
│   ├── baoyu-xhs-images/SKILL.md         # 已废弃 → baoyu-image-cards
│   └── baoyu-youtube-transcript/SKILL.md
└── .claude/skills/release-skills/SKILL.md  # 内部发布流程技能
```

- **文件类型分布**：22 个 SKILL.md（20 个功能技能 + 2 个废弃技能）、1 个内部 release 技能、package.json monorepo
- **编排关系**：技能之间平列独立，无 router 或 meta skill。CLAUDE.md 规定跨 skill 同步规则（如 baoyu-image-gen 需跟随 baoyu-imagine 更新）
- **跨件契约**：每个技能通过 EXTEND.md 文件实现用户偏好覆写——技能运行时按优先级（项目级 → XDG 配置 → 用户主目录）搜索三个路径下的 EXTEND.md，允许用户在不修改技能本体的情况下注入个性化配置

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「EXTEND.md 用户配置插槽」——每个技能在 Step 0 预留一个标准化的配置读取步骤，使用户可以通过写一个本地文件覆盖技能默认行为，无需 fork 仓库
- **解决什么问题**：Claude Code 技能是只读安装的，用户无法直接修改已安装技能的行为。EXTEND.md 机制提供了一个无需 fork 的个性化出口
- **Trade-off**：EXTEND.md 机制带来了跨 skill 配置命名空间污染风险（见缺陷 2.3）；废弃技能保留在仓库中便于迁移参考，但增加了版本同步维护负担
- **认知模型**：作者将技能视为「有状态的工作流程模板」——不只是指令集，还包含用户偏好持久化、工具依赖声明、提供商切换逻辑等完整生命周期管理

---

## 二、过去审查发现（2026-04-29 历史快照）

### 2.1 当时质量评分（NLPM）

该仓库 2026-04-29 当时得分 **90/100**。

| 文件 | 当时分数 | 主要问题 |
|---|---|---|
| skills/baoyu-image-gen/SKILL.md | 82 | 废弃技能，description 嵌入废弃声明，无完成报告步骤 |
| skills/baoyu-xhs-images/SKILL.md | 82 | 废弃技能，EXTEND.md 路径指向 baoyu-image-cards |
| skills/baoyu-youtube-transcript/SKILL.md | 87 | 步骤编号重复（1,2,3,3,4） |
| skills/baoyu-post-to-weibo/SKILL.md | 87 | 无正式完成/输出摘要步骤 |
| skills/baoyu-post-to-x/SKILL.md | 87 | 同上 |
| CLAUDE.md | 88 | "short variable names" 量词模糊 |
| skills/baoyu-imagine/SKILL.md | 88 | 无完成/输出摘要步骤 |
| skills/baoyu-url-to-markdown/SKILL.md | 88 | 无正式完成摘要步骤 |
| .claude/skills/release-skills/SKILL.md | 96 | "meaningful description" 模糊量词 ×2 |

### 2.2 当时值得借鉴的模式

1. **EXTEND.md 用户偏好插槽** → 让技能在只读安装环境下支持个性化配置，根本原因是把「用户数据」和「技能逻辑」的关注点分离。示例路径：`skills/baoyu-image-cards/SKILL.md` Step 0。借鉴方式：在自己的技能 Step 0 加入三级路径搜索（项目 → XDG → home），约定 EXTEND.md 格式

2. **danger- 前缀显式标记高风险技能** → `baoyu-danger-gemini-web` 和 `baoyu-danger-x-to-markdown` 通过命名告知用户这些技能操控外部服务或有副作用，让用户在安装前知道风险级别。借鉴方式：对有网络写入、账号操作等副作用的技能用命名约定区分

3. **双语文档（CHANGELOG.md + CHANGELOG.zh.md）** → 中文用户基数大的开源项目，保持双语同步能扩大覆盖面。借鉴方式：README 提供 zh/en 两个版本

4. **技能版本号同步规则写入 CLAUDE.md** → CLAUDE.md 明确要求跨 skill 同步更新时需要保持版本号一致，把维护约束文档化。借鉴方式：在仓库 CLAUDE.md 中声明哪些文件需要联动更新

5. **废弃技能保留 + 接班人声明** → deprecated 技能在 description 中标注接班者，保持向后兼容的安装体验。借鉴方式：技能演进时不直接删除旧版，在 frontmatter 中加 deprecated + successor 字段

### 2.3 当时的缺陷

1. **baoyu-youtube-transcript 步骤编号重复（1,2,3,3,4）**：根本原因是编辑时手动维护步骤序号，没有自动校验机制。代理按编号顺序执行工作流时会跳过或错误排序保存步骤，静默失败。自查：我是否也有手动维护的有序列表？

2. **baoyu-xhs-images 的 EXTEND.md 路径指向了 baoyu-image-cards**：根本原因是从 baoyu-image-cards fork 过来时，没有全局替换配置路径，导致废弃技能「污染」了它本来不应该知道的邻居的配置目录。自查：我克隆/复用技能时是否全面替换了所有路径引用？

3. **多个技能缺少「完成摘要步骤」**：工作流结束后代理没有结构化的成功信号，只是静默退出。根本原因是作者将「技能执行成功」等同于「最后一步运行完」，忽略了代理需要一个明确的输出格式来向用户汇报。自查：我的技能最后一步是否有明确的输出描述？

### 2.4 当时的优化机会

1. **废弃技能 description 用专用 frontmatter 字段标注**：改为 `deprecated: true` + `successor: baoyu-imagine`，让 description 保持干净，不混入废弃元数据
2. **所有技能统一添加完成摘要步骤**：格式如「Step N: 输出摘要——列出文件路径、使用的提供商/模型、成功指标」
3. **baoyu-diagram 补充 version 和 metadata.openclaw 字段**：与其他技能保持一致，提高市场可发现性

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| baoyu-youtube-transcript 步骤编号重复（1,2,3,3,4） | `grep -n "^[0-9]\." skills/baoyu-youtube-transcript/SKILL.md` | **仍缺失** — 第 140-141 行仍是两个 "3." | 4 周内未修复，说明作者优先级低于新功能开发 |
| baoyu-xhs-images EXTEND.md 路径错误 | `grep -n "baoyu-image-cards" skills/baoyu-xhs-images/SKILL.md` | **仍缺失** — 第 315-317 行仍引用 baoyu-image-cards 路径 | 配置命名空间污染问题持续存在 |
| baoyu-image-gen vs baoyu-imagine 版本漂移 | `grep "^version:" skills/baoyu-image-gen/SKILL.md skills/baoyu-imagine/SKILL.md` | **仍缺失** — 1.56.4 vs 1.58.0，差距未缩小 | 废弃技能维护成本被低估，版本同步规则在实践中未被执行 |

### 3.2 架构演进

与 audit 时（2026-04-29）相比，仓库结构基本稳定——22 个技能、monorepo 结构、CLAUDE.md 规范均无大变化。CHANGELOG.md 记录了最近的更新，未发现新技能添加或旧技能删除。说明作者正处于维护期而非快速扩张期，重心可能转移到了脚本和工具层（packages/ 目录下的 npm 包）。

### 3.3 新增的可学习模式

当前 HEAD 中未发现相对 audit 时新增的显著设计模式。EXTEND.md 系统、废弃技能管理、双语文档均已在 audit 时覆盖。暂无新增内容值得额外记录。

---

## 四、校准

### 4.1 我已经在做对的

1. **单职责技能设计**：每个技能对应一个明确任务，baoyu 也是这样组织的——无跨技能调用的意大利面条式结构
2. **CLAUDE.md 记录跨文件维护规则**：我也在 CLAUDE.md 中声明哪些文件需要联动更新，与 baoyu 实践一致
3. **Frontmatter 完整性**：我在设计技能时注意保持 name/description/version 字段齐全
4. **危险操作显式标注**：对涉及外部写入的操作我也有类似的命名或注释提示

### 4.2 挑战 / 验证

**挑战**：我之前认为「技能废弃后只要加个 deprecated 声明就够了」，但 baoyu-xhs-images 的案例说明，废弃技能如果不做配置路径隔离，会实际污染用户环境——这不只是文档问题，而是一个运行时副作用。废弃技能需要做到：路径自包含 + 不写入任何不属于自己的目录。

**验证**：EXTEND.md 三级搜索路径（项目 → XDG → home）的优先级设计被验证是正确的——这是标准的配置优先级约定（XDG Base Directory），比单一固定路径灵活得多。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查所有 SKILL.md 中是否有重复的步骤序号
grep -rn "^[0-9]\. " ~/.claude/skills/*/SKILL.md | awk -F: '{print $1":"$2}' | sort | uniq -d
# 命中后：手动检查对应文件的有序列表，逐行确认步骤编号连续且不重复

# 检查废弃技能是否引用了非自身路径
grep -rn "deprecated\|Deprecated" ~/.claude/skills/*/SKILL.md | grep -v "^Binary"
# 命中后：检查对应文件的 EXTEND.md 路径是否包含其他技能名称

# 检查技能末尾是否有明确的完成摘要步骤
grep -rn "完成\|summary\|output\|结果\|报告" ~/.claude/skills/*/SKILL.md | grep -i "step\|步骤" | tail -5
# 命中后：确认最后一步是否有具体的输出格式（路径/数量/状态）
```

### 5.2 灵感 → 实施路径

1. **想法**：在自己的技能中引入 EXTEND.md 用户配置插槽
   - **为何可行**：这是已被 16303★ 仓库验证的模式，解决了「只读安装 + 用户个性化」的矛盾
   - **第一步**：在一个高频使用的技能的 Step 0 添加三级路径搜索代码块（约 15 行），定义 EXTEND.md 的格式说明

2. **想法**：为废弃技能创建标准化的迁移声明 frontmatter
   - **为何可行**：`deprecated: true` + `successor: <skill-name>` 字段让工具链可以自动检测并提示用户升级，避免在 description 里混入废弃声明
   - **第一步**：在自己最近要废弃的技能 frontmatter 中添加这两个字段，验证 NLPM 扫描工具是否识别

3. **想法**：建立步骤编号连续性检查脚本
   - **为何可行**：baoyu 的重复步骤 bug 4 周未修复，说明人工审核靠不住，自动化才能可靠发现
   - **第一步**：写一个 20 行 Python 脚本，解析 SKILL.md 中的有序列表，检测序号跳跃或重复，集成到 pre-commit 钩子
