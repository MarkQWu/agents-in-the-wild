# K-Dense-AI/scientific-agent-skills — 学习案例

**仓库**：https://github.com/K-Dense-AI/scientific-agent-skills
**Stars**：19356 | **来源**：xiaolai upstream
**Audit 日期**：2026-04-25（历史快照）| **生成日期**：2026-05-19（基于当前 HEAD）
**主题标签**：`manifest-discipline`, `security-gate`, `vague-quantifier`, `single-purpose`, `experience-accumulation`

---

## 一、理解（基于当前仓库状态）

### 1.1 仓库概览

K-Dense-AI/scientific-agent-skills 是 K-Dense Inc. 出品的科学计算 AI 技能集合，覆盖生物信息学、化学信息学、量子计算、数据科学、科研写作、临床报告等领域的 138 个 SKILL.md（audit 时为 100 个）。19356★ 使其成为科学计算方向最高星的 Claude Code 技能仓库。仓库的用户群主要是研究人员和数据科学家，技能覆盖了从 scanpy/anndata（单细胞分析）到 pennylane（量子计算）再到 clinical-reports（临床文档）的完整研究栈。

### 1.2 架构剖析

```
scientific-agent-skills/
├── scientific-skills/         # 所有技能（138 个子目录，每个含 SKILL.md）
│   ├── adaptyv/SKILL.md
│   ├── anndata/SKILL.md
│   ├── benchling-integration/SKILL.md
│   ├── clinical-reports/SKILL.md
│   ├── ...（共 138 个）
│   └── zarr-python/SKILL.md
├── docs/                      # 新增：文档目录
├── .github/                   # CI 工作流
├── SECURITY.md                # 新增：公开安全披露文档（含 PR 审查发现）
├── pyproject.toml             # 新增：Python 项目配置
├── uv.lock                    # 新增：锁文件
├── scan_skills.py             # 新增：技能自动扫描工具
└── scan_pr_skills.py          # 新增：PR 技能扫描工具
```

- **文件类型分布**：138 个 SKILL.md（全部在 `scientific-skills/` 子目录），Python 工具脚本，无 hooks/无 MCP configs
- **编排关系**：技能之间完全平列，无 router 或 meta skill；14 个技能有配套的 `scripts/generate_schematic*.py`（调用 OpenRouter API）
- **跨件契约**：部分技能依赖外部 API 密钥（OPENROUTER_API_KEY、PARALLEL_API_KEY），但大多数 SKILL.md 未在 description 中声明这一依赖

### 1.3 设计思路 / 方法论

- **核心设计哲学**：「领域专家知识封装」——每个技能对应一个具体的科学计算库，封装安装方法、核心 API 用法、最佳实践，让非专家能够快速上手
- **解决什么问题**：研究人员使用各种小众科学库时，Claude 通常缺乏足够的训练数据；技能注入最新的 API 知识和领域最佳实践
- **Trade-off**：100+ 技能的大体量带来了维护一致性难题——install 命令规范（`uv pip install`）、license 格式（SPDX）、metadata 字段在不同贡献者之间频繁漂移；部分技能（14 个）依赖私有 API（Nano Banana Pro）但未在 description 中披露
- **认知模型**：作者将技能视为「领域知识接口层」——把库文档和最佳实践打包成可注入的结构化知识，让 AI 具备专家级的库使用能力

---

## 二、过去审查发现（2026-04-25 历史快照）

### 2.1 当时质量评分（NLPM）

该仓库 2026-04-25 当时得分 **83/100**（100 个 SKILL.md 的加权平均）。

| 层级 | 数量 | 分数范围 | 代表技能 |
|---|---|---|---|
| 清洁 | ~48 | 86-88 | zarr-python, umap-learn, anndata, pennylane |
| 轻微问题 | ~27 | 82-85 | scikit-learn (bug), pyopenms (bug) |
| 质量问题 | ~17 | 73-80 | venue-templates, Nano Banana 系列 |
| 显著问题 | ~8 | 70-72 | treatment-plans, scientific-writing |

### 2.2 当时值得借鉴的模式

1. **领域覆盖系统化**：技能集从基础工具（numpy/pandas）到高度专业化的领域（geniml/单细胞多组学）一应俱全，体现了系统性规划而非碎片化积累。借鉴：建立技能集时先做领域地图，再填充覆盖

2. **第三方贡献者机制**：11 个外部贡献者的技能被收录（如 Kuan-lin Huang 的分子动力学/系统发育/糖工程），并在 `metadata.skill-author` 中保留归属。借鉴：技能集可以接受社区贡献，但需要规范 metadata 格式

3. **统一的 install 命令风格（uv pip install）**：大多数技能使用 `uv pip install`，统一了 Python 包管理风格，便于用户批量安装。借鉴：技能集内部统一包管理工具和调用格式

4. **大规模技能集的分层质量**：不同技能的质量有明显分层（86-72 分），说明在规模化技能集中保持全面均匀的质量需要专门的质量保证机制

5. **API 密钥依赖的透明声明（应有但缺失的）**：OPENROUTER_API_KEY 依赖在 14 个技能中存在但均未在 description 中声明——这是一个反模式，提示正确做法是在 frontmatter description 中明确声明外部依赖

### 2.3 当时的缺陷

1. **`uv uv pip install` 命令错误（5 个技能）**：根本原因是批量生成技能时字符串拼接错误，`uv pip install` 被写成了 `uv uv pip install`，这是一个立即失败的 shell 命令。自查：我是否在批量生成内容时有类似的系统性错误？

2. **14 个技能嵌入「Nano Banana Pro」私有产品品牌，且不披露 API 密钥需求**：根本原因是技能文档把内部工作流程直接暴露为用户指令，未考虑外部用户没有访问该私有服务的权限。从用户角度看，这些技能会在运行时静默失败（没有 OPENROUTER_API_KEY），而 description 完全没有提示。自查：我的技能是否隐式依赖了用户不一定有的外部资源？

3. **license 字段混乱（Unknown、URL 格式、非 SPDX 格式）**：根本原因是技能是由多个贡献者在不同时间写的，没有统一的 PR 模板或 CI 检查来强制 SPDX 格式。自查：我的技能 frontmatter 中 license 字段是否使用标准 SPDX 标识符？

### 2.4 当时的优化机会

1. **批量修复 `uv uv pip install` → `uv pip install`**：简单的 find-replace，影响 5+ 个技能的可用性
2. **在所有依赖 OPENROUTER_API_KEY 的技能 description 中添加前置声明**：「需要 OPENROUTER_API_KEY 环境变量，参见 xxx 获取」
3. **建立 CI 检查强制 license SPDX 格式和 metadata.skill-author 字段存在**

---

## 三、现在 vs 过去对比

### 3.1 关键缺陷在现仓库中的状态

| 过去缺陷 | 检查方法 | 现状 | 含义 |
|---|---|---|---|
| `uv uv pip install` 错误命令（5 个技能） | `grep -rn "uv uv pip" scientific-skills/*/SKILL.md` | **大部分已修复** — 仅剩 `scientific-skills/gget/SKILL.md` 第 23 行保留了错误命令；其他 4 个技能已修复 | 作者进行了批量修复但遗漏了 gget；4 周内解决了 4/5 |
| scanpy license 字段错误（SD-3-Clause） | `grep "license" scientific-skills/scanpy/SKILL.md` | **已修复** — 现为 `license: BSD-3-Clause` | 简单 typo 修复快速到位 |
| 14 个技能的 Nano Banana Pro 品牌（无 API 密钥声明） | `grep -rn "Nano Banana" scientific-skills/ \| wc -l` | **仍缺失** — 289 处 Nano Banana 引用，较 audit 时无减少 | 4 周后问题完全未处理，说明作者认为这是功能而非缺陷 |

### 3.2 架构演进

这是本批次架构演进幅度仅次于 caveman 的仓库：

**结构变化**：
- 技能从平铺根目录（如 `scikit-learn/SKILL.md`）迁移至 `scientific-skills/scikit-learn/SKILL.md` 子目录——这是一个规范化重组，说明作者在扩展到 100+ 技能后意识到需要统一的命名空间
- 技能数量从 100 增长到 138（新增约 38 个技能）
- 新增 `SECURITY.md`：包含外部 PR 审查发现的公开披露，内容来源于 GitHub Actions 安全扫描（包括对 `uv uv pip` typo 的记录和 Nano Banana 操控模式分析）
- 新增 Python 工具链：`pyproject.toml`、`uv.lock`、`scan_skills.py`、`scan_pr_skills.py`——说明仓库从「纯文档集合」向「有工具支持的技能平台」演进

**这说明作者意识到了什么**：规模化技能集（100+）必须配套自动化工具来维护质量一致性；手动维护 138 个技能的 frontmatter 格式已经超出人工能力上限。

### 3.3 新增的可学习模式

1. **SECURITY.md 作为公开安全披露文档**：仓库的 `SECURITY.md` 不只是漏洞报告模板，而是包含了实际的外部审查发现（含具体文件路径、行号、修复建议）。这是一种「把 PR review 发现持久化」的实践——安全审查结论不只活在 PR 评论里，而是归档到仓库文档中。

2. **scan_skills.py 技能自动扫描**：新增的 Python 工具可以扫描所有技能文件，用于质量检查、命令格式验证等。这是「工具辅助维护大规模技能集」的具体实现。

3. **子目录分组（scientific-skills/）**：从平铺根目录到 `scientific-skills/` 子目录的迁移，提供了清晰的命名空间，便于未来添加其他类型的文件（文档、工具）而不与技能混淆。

---

## 四、校准

### 4.1 我已经在做对的

1. **SPDX 标准 license 字段**：我的技能 frontmatter 使用标准 SPDX 标识符（MIT、Apache-2.0 等）
2. **外部依赖在 description 中声明**：我在技能 description 中说明需要的环境变量或外部服务
3. **单一技能对应单一库/任务**：我的技能集合遵循单职责原则，不在单个技能中混入多个不相关的工具
4. **metadata.skill-author 字段**：我在技能 frontmatter 中保留作者归属

### 4.2 挑战 / 验证

**挑战**：我之前认为「在技能中引用内部工具只要用户知道就行」，但 K-Dense 的 Nano Banana 案例说明，当技能描述中出现「Nano Banana Pro 会自动生成」这类语句时，用户在安装时无法判断这是一个付费服务，直到实际运行时才发现静默失败。这产生了对用户的误导——不只是不方便，而是创建了错误的期望。任何隐式的外部依赖都应该在 description 的第一句就明确。

**验证**：scan_skills.py 的出现验证了「规模化技能集必须配套自动化工具」的判断——138 个技能光靠人工审查是不可靠的。这印证了 NLPM 这类自动化质量扫描工具的必要性。

---

## 五、行动

### 5.1 自查动作

```bash
# 检查技能中是否有隐式外部 API 依赖（环境变量）未在 description 中声明
grep -rn "API_KEY\|api_key\|TOKEN" ~/.claude/skills/*/SKILL.md | grep -v "^.*:.*description" | head -20
# 命中后：在对应技能的 description 字段第一句加「需要 XXX 环境变量」声明

# 检查 license 字段格式是否为标准 SPDX
grep -rn "^license:" ~/.claude/skills/*/SKILL.md | grep -vE "(MIT|Apache-2\.0|BSD-[23]-Clause|GPL-[23]\.0|LGPL-[23]\.0|ISC|CC0|Proprietary)" | head -10
# 命中后：查找对应库的 SPDX 标识符（spdx.org/licenses）并替换

# 检查 install 命令是否有重复工具前缀的错误
grep -rn "uv uv\|pip pip\|npm npm" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：删除重复前缀，保留正确格式

# 检查 metadata.skill-author 是否存在
grep -rL "skill-author" ~/.claude/skills/*/SKILL.md 2>/dev/null
# 命中后：在对应技能的 metadata: 块中添加 skill-author 字段
```

### 5.2 灵感 → 实施路径

1. **想法**：建立技能集的 CI 质量检查脚本（参考 scan_skills.py）
   - **为何可行**：K-Dense 138 个技能手动维护质量已不可行，scan_skills.py 是正确方向；即使只有 20 个技能，自动化检查也能捕获批量错误
   - **第一步**：写一个 30 行 Python 脚本，检查所有 SKILL.md 的 frontmatter 必填字段（name/description/license/metadata.skill-author）和 install 命令格式，集成到 pre-commit 或 GitHub Actions

2. **想法**：建立「外部依赖声明规范」
   - **为何可行**：K-Dense 的 Nano Banana 案例是一个典型的反模式——隐式依赖导致用户体验崩溃。制定明确的声明格式可以避免自己重蹈覆辙
   - **第一步**：在 CLAUDE.md 中添加一条规则：「技能 description 字段必须在第一句声明所有外部 API 依赖（格式：需要 XXX_API_KEY，详见 [链接]）」

3. **想法**：把技能迁移到带命名空间的子目录结构（参考 scientific-skills/）
   - **为何可行**：随着技能数量增长（超过 20 个），平铺根目录会变得难以导航；子目录提供清晰的分组和扩展空间
   - **第一步**：把现有技能按功能领域分组（开发工具、内容创作、数据分析等），重组到对应的子目录，更新 plugin.json 中的路径引用
