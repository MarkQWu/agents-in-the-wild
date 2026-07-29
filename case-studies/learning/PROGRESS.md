# 学习进度

**总量**：210 audit reports（207 upstream + 3 local）
**已学**：202（截至 2026-07-29）
**<500 星跳过**：57（注：2026-07-19 将 czlonkowski/n8n-skills、dontbesilent2025/dbskill、kazukinagata/shinkoku、krodak/clickup-cli 从跳过池移出，原因：exemplar_published=true 覆盖星数门槛；2026-07-21 将 pe-menezes-fin-claude-plugin、leowux-pony、tech-leads-club-agent-skills、shinpr-claude-code-workflows 从跳过池移出；2026-07-23 将 ooiyeefei-ccc、uppinote20-claude-dashboard、vladikk-modularity 从跳过池移出，同样原因；2026-07-24 处理 memvid-claude-brain（465★）、levnikolaevich-claude-code-skills（423★）、josstei-maestro-orchestrate（370★）、peterfei-ai-agent-team（336★）——均 <500 但为现存最接近门槛的未处理 upstream audit，≥500 池已连续耗尽；2026-07-25 处理 mbruhler-claude-orchestration（215★）、agent-sh-agnix（207★）、webdevtodayjason-sub-agents（189★）、LerianStudio-ring（174★）——同上；2026-07-26 处理 xiaolai-loc-guardian-for-claude（N/A★）、deanpeters-Product-Manager-Skills（N/A★）、ananddtyagi-cc-marketplace（N/A★）、yonatangross-orchestkit（149★）——≥500 池持续耗尽，选取 NLPM 学习价值最高的 4 个案例；2026-07-29 处理 agiletec-inc-airis-mcp-gateway（151★）、kangraemin-claude-inspector（115★）、c0x12c-ai-toolkit（61★）、navapbc-digital-service-orchestra（3★）——≥500 池持续耗尽，按星数降序选取剩余 upstream 中学习价值最高的 4 个）
**待处理**：8 个 upstream（均 <10 stars，≥500 池已完全耗尽）
**最后更新**：2026-07-29

---

## ✅ 已完成（按生成日期降序）

### 2026-07-29 (4 篇)
- [x] agiletec-inc/airis-mcp-gateway · ⭐151 · NLPM 90/100 · upstream（SECURITY BLOCKED，curl|bash 3 处 Critical 持续，shell 注入 Medium 持续，4 个 skill 零示例持续；scripts/airis-gateway→airis-mcp-gateway 改名；MCP 按需启动架构；CLAUDE.md 禁止区模式） · [案例](2026-07/2026-07-29-agiletec-inc-airis-mcp-gateway.md)
- [x] kangraemin/claude-inspector · ⭐115 · NLPM 76/100 · upstream（SECURITY REVIEW，3 个 agent 全部缺 name 字段持续导致无法注册，review-rules.md 外部引用持续；3 个 skill 100/100 完美；静默失效陷阱案例；无架构演进） · [案例](2026-07/2026-07-29-kangraemin-claude-inspector.md)
- [x] c0x12c/ai-toolkit · ⭐61 · NLPM 96/100 · upstream（SECURITY REVIEW，allowed-tools 全 69 命令缺失持续，phase-reviewer Bash 未声明持续，guard 无空值处理持续；v1.22.1→v1.27.0 +5版本活跃迭代，bridges/telegram 新增；Spartan GSD 路由器+角色化 agent 架构） · [案例](2026-07/2026-07-29-c0x12c-ai-toolkit.md)
- [x] navapbc/digital-service-orchestra · ⭐3 · NLPM 84.8/100 · upstream（SECURITY PASS，bot-psychologist taxonomy 17→16 不一致持续；agents 从 31 暴增到 63；skills 从 25→39；DSO v1.16.11 双渠道发布；Router+agent 矩阵+JSON schema 通信架构；drift injection 测试模式） · [案例](2026-07/2026-07-29-navapbc-digital-service-orchestra.md)

### 2026-07-26 (4 篇)
- [x] xiaolai/loc-guardian-for-claude · ⭐N/A · NLPM 97/100 · upstream（SECURITY CLEAR，example 已从 1 增到 2，allowed-tools/concise/obvious 持续；双 agent 管线+结构化数据契约+最小 skill 访问架构） · [案例](2026-07/2026-07-26-xiaolai-loc-guardian-for-claude.md)
- [x] deanpeters/Product-Manager-Skills · ⭐N/A · NLPM 89/100 · upstream（SECURITY CLEAR，best_for/scenarios 全部 70 skills 已修，allowed-tools+empty-input 持续，PR #8 安全修复已合并；链式 skills+稳定可差分 schema 架构） · [案例](2026-07/2026-07-26-deanpeters-Product-Manager-Skills.md)
- [x] ananddtyagi/cc-marketplace · ⭐N/A · NLPM 73/100 · upstream（SECURITY CLEAR，5 个 PR #44-#48 全部仍开放，5 个 bug 全部持续；多贡献者目录型仓库架构；PR 无响应本身是维护信号） · [案例](2026-07/2026-07-26-ananddtyagi-cc-marketplace.md)
- [x] yonatangross/orchestkit · ⭐149 · NLPM 84/100 · upstream（SECURITY CRITICAL，eval/webhook/MCP矛盾/TaskGet全部持续，examplePrompts 从 0/15→36/36 全补，agents 从 15→36，artifacts 从 103→265+） · [案例](2026-07/2026-07-26-yonatangross-orchestkit.md)

### 2026-07-25 (4 篇)
- [x] mbruhler/claude-orchestration · ⭐215 · NLPM 82/100 · upstream（SECURITY CLEAR，Write 工具未声明+文档死链+examples/ 目录 3 个 bug 持续，namespace 已加，hardcoded 路径已修，security-auditor 新 agent 已加；Socratic 工作流架构） · [案例](2026-07/2026-07-25-mbruhler-claude-orchestration.md)
- [x] agent-sh/agnix · ⭐207 · NLPM 64/100 · upstream（SECURITY REVIEW→curl|bash 系夹具误判，ALL 5 production bugs 已全修，v0.40.0 增至 437 rules，Neovim/Zed 编辑器支持新增；NL 表皮+原生二进制核心架构） · [案例](2026-07/2026-07-25-agent-sh-agnix.md)
- [x] webdevtodayjason/sub-agents · ⭐189 · NLPM 78/100 · upstream（SECURITY REVIEW→shell:true 已修，name:/model:/Bash 已全修，ElevenLabs MCP 语音通知新增，work-completion-summary agent 新增；v1.5.5 命令路由+岗位专家分层架构） · [案例](2026-07/2026-07-25-webdevtodayjason-sub-agents.md)
- [x] LerianStudio/ring · ⭐174 · NLPM 87/100 · upstream（SECURITY APPROVE WITH REQUIRED FIXES→PyYAML 未固定+model: 100% 缺失持续；.archive/ 整个删除；CLAUDE.md 法典 93/100；法典驱动多团队评审委员会+反理性化表架构） · [案例](2026-07/2026-07-25-LerianStudio-ring.md)

### 2026-07-24 (4 篇)
- [x] memvid/claude-brain · ⭐465 · NLPM 96/100 · upstream（SECURITY BLOCKED，HIGH shell injection，全部 3 bug 持续，hooks 生命周期架构，NL 表皮+JS 核心模式） · [案例](2026-07/2026-07-24-memvid-claude-brain.md)
- [x] levnikolaevich/claude-code-skills · ⭐423 · NLPM 77/100 · upstream（SECURITY BLOCKED，CRITICAL eval，288→21 MD 架构大重组，skills-catalog 全删，独立插件套件演进） · [案例](2026-07/2026-07-24-levnikolaevich-claude-code-skills.md)
- [x] josstei/maestro-orchestrate · ⭐370 · NLPM 87/100 · upstream（SECURITY BLOCKED，prepare 已修，Output Contract 模糊词扩散，25 存根 score 75，多运行时适配器架构） · [案例](2026-07/2026-07-24-josstei-maestro-orchestrate.md)
- [x] peterfei/ai-agent-team · ⭐336 · NLPM 82/100 · upstream（SECURITY BLOCKED，CRITICAL curl|bash+HIGH SQL注入，3 bug 全持续，v2.1.0 新增 fullstack，中文量词问题） · [案例](2026-07/2026-07-24-peterfei-ai-agent-team.md)

### 2026-07-23 (3 篇)
- [x] ooiyeefei/ccc · ⭐405 · NLPM 96/100 · upstream（exemplar_published，SECURITY CLEAR，Bug #1 deckling-pptx 已修复，/pm:review 缺失+streak allowed-tools 缺失持续，skills 扩展至 18+references 子目录全面落地） · [案例](2026-07/2026-07-23-ooiyeefei-ccc.md)
- [x] uppinote20/claude-dashboard · ⭐404 · NLPM 99/100 · upstream（exemplar_published，SECURITY REVIEW，$ARGUMENTS 注入漏洞持续，CLAUDE.md 结构树遗漏 2 个命令持续，Widget 扩展至 30+） · [案例](2026-07/2026-07-23-uppinote20-claude-dashboard.md)
- [x] vladikk/modularity · ⭐337 · NLPM 98/100 · upstream（exemplar_published，SECURITY CLEAR，无 bug，model: 声明缺失+模糊量词持续，参考注入架构，4 skills 无架构变化） · [案例](2026-07/2026-07-23-vladikk-modularity.md)

### 2026-07-21 (4 篇)
- [x] shinpr/claude-code-workflows · ⭐320 · NLPM 91/100 · upstream（exemplar_published，SECURITY CLEAR，24 agents 无 model 声明持续，technical-designer Grep bug 持续，架构重组：扁平→4个子包 monorepo，recipe-agent 二层编排架构） · [案例](2026-07/2026-07-21-shinpr-claude-code-workflows.md)
- [x] tech-leads-club/agent-skills · ⭐N/A · NLPM 93/100 · upstream（exemplar_published，SECURITY CLEAR，plugin.json 缺 version 字段持续，模糊量词系统性问题持续，78→84 skills，MCP 分发层，域分桶大型 skill 目录架构） · [案例](2026-07/2026-07-21-tech-leads-club-agent-skills.md)
- [x] leowux/pony · ⭐1 · NLPM 91/100 · upstream（exemplar_published，SECURITY CLEAR，run.md Agent 工具未声明 bug 持续，4 agents 零示例持续，.orphaned_at 标记已废弃，NL表皮+原生二进制核心架构） · [案例](2026-07/2026-07-21-leowux-pony.md)
- [x] pe-menezes/fin-claude-plugin · ⭐0 · NLPM 94/100 · upstream（exemplar_published，SECURITY CLEAR，零示例+无 model 声明持续，.mcp.json 版本未固定持续，1 agent+6 skills 财务编排架构，v0.5.0 新增 marketplace.json） · [案例](2026-07/2026-07-21-pe-menezes-fin-claude-plugin.md)

### 2026-07-19 (4 篇)
- [x] krodak/clickup-cli · ⭐57 · NLPM 95/100 · upstream（exemplar_published，SECURITY CLEAR，plugin.json 缺 entries 持续，内外 skill 分层+符号链接单一真实来源；版本 v1.25.2→v1.39.0） · [案例](2026-07/2026-07-19-krodak-clickup-cli.md)
- [x] kazukinagata/shinkoku · ⭐335 · NLPM 94/100 · upstream（exemplar_published，SECURITY CLEAR，setup/SKILL.md @version 缺失持续，NL 协调+Python CLI 混合架构；新增 SECURITY.md + PR 模板） · [案例](2026-07/2026-07-19-kazukinagata-shinkoku.md)
- [x] dontbesilent2025/dbskill · ⭐0 · NLPM 90/100 · upstream（exemplar_published，SECURITY REVIEW→dbskill-upgrade 删除+/文风分析 引用已修，三模路由+技能链架构；29 skills 已大幅扩张） · [案例](2026-07/2026-07-19-dontbesilent2025-dbskill.md)
- [x] czlonkowski/n8n-skills · ⭐null · NLPM 93/100 · upstream（exemplar_published，SECURITY CLEAR，plugin.json 缺 entries 持续，模糊量词持续；三层架构 MCP+skill+hook；15 skills 扩张+evaluations/） · [案例](2026-07/2026-07-19-czlonkowski-n8n-skills.md)

### 2026-07-14 (4 篇)
- [x] pbakaus/impeccable · ⭐46,376 · NLPM 93/100 · upstream（SECURITY REVIEW→已修，new Function() 已修，stale CLAUDE.md evals/AGENT.md 引用已修，命令计数不一致持续；NL表皮+原生代码核心架构） · [案例](2026-07/2026-07-14-pbakaus-impeccable.md)
- [x] openai/symphony · ⭐25,947 · NLPM 89/100 · upstream（SECURITY CLEAR，commit/SKILL.md 100 分参考实现，5 skills 缺 ## Output 持续，AGENTS.md path 前缀缺失持续，模糊量词持续；参考实现锚定式架构） · [案例](2026-07/2026-07-14-openai-symphony.md)
- [x] ccplugins/awesome-claude-code-plugins · ⭐878 · NLPM 78/100 · upstream（SECURITY CLEAR，10 个命名错误 bug 持续，~30 truncated description 持续，agent-in-wrong-dir 持续，97 质量问题持续；共享基础设施聚合式架构） · [案例](2026-07/2026-07-14-ccplugins-awesome-claude-code-plugins.md)
- [x] agent-sh/agentsys · ⭐892 · NLPM 91/100 · upstream（SECURITY CLEAR，hardcoded /Users/avifen/ paths 已修/删除，prepare script git hooks 已修，12 perf skills 缺 examples 持续；契约驱动平铺扩展式架构） · [案例](2026-07/2026-07-14-agent-sh-agentsys.md)

### 2026-07-10 (4 篇)
- [x] slavingia/skills · ⭐N/A · NLPM 97/100 · upstream（SECURITY CLEAR，plugin.json skills 字段 3 月仍未修，线性 skill 链极简架构） · [案例](2026-07/2026-07-10-slavingia-skills.md)
- [x] openai/codex-plugin-cc · ⭐N/A · NLPM 93/100 · upstream（SECURITY REVIEW→已修，xiaolai案例存在，$ARGUMENTS 已 merge，新增 transfer.md） · [案例](2026-07/2026-07-10-openai-codex-plugin-cc.md)
- [x] xiaolai/codex-toolkit-for-claude · ⭐N/A · NLPM 96/100 · upstream（SECURITY CLEAR，status.md $ARGUMENTS 已修，无 allowed-tools 持续，shared partial 架构） · [案例](2026-07/2026-07-10-xiaolai-codex-toolkit-for-claude.md)
- [x] xiaolai/grill-for-claude · ⭐N/A · NLPM 97/100 · upstream（SECURITY CLEAR，3 agent 缺 Output Format 持续，版本 drift 持续，新增 codex/ 双运行时支持） · [案例](2026-07/2026-07-10-xiaolai-grill-for-claude.md)

### 2026-07-09 (4 篇)
- [x] coreyhaines31/marketingskills · ⭐0 · NLPM 96/100 · upstream（exemplar_published，SECURITY CLEAR，Output Format 缺陷从 13 扩大至 23 个，product-marketing-context 前置模式值得借鉴） · [案例](2026-07/2026-07-09-coreyhaines31-marketingskills.md)
- [x] agenticnotetaking/arscontexta · ⭐0 · NLPM 96/100 · upstream（exemplar_published，SECURITY CLEAR，8/9 model 声明 BUG 持续未修，allowed-tools 缺失 bug 持续） · [案例](2026-07/2026-07-09-agenticnotetaking-arscontexta.md)
- [x] 2389-research/simmer · ⭐4 · NLPM 93/100 · upstream（exemplar_published，SECURITY CLEAR，CLAUDE.md 三项 -20 分持续未修，orchestrator 550 行超限未修） · [案例](2026-07/2026-07-09-2389-research-simmer.md)
- [x] 2389-research/review-squad · ⭐1 · NLPM 96/100 · upstream（exemplar_published，SECURITY CLEAR，CLAUDE.md 无 frontmatter/无示例持续，"relevant" 模糊量词持续） · [案例](2026-07/2026-07-09-2389-research-review-squad.md)

### 2026-07-08 (4 篇)
- [x] anthropics/claude-plugins-official · ⭐31759 · NLPM 96/100 · upstream（SECURITY REVIEW，bun install 未修；skill-creator agents frontmatter bug 持续） · [案例](2026-07/2026-07-08-anthropics-claude-plugins-official.md)
- [x] hashicorp/agent-skills · ⭐702 · NLPM 98/100 · upstream（SECURITY CLEAR，主要 bug list_resources.sh 已修；TFE_TOKEN pinning 仍未修） · [案例](2026-07/2026-07-08-hashicorp-agent-skills.md)
- [x] trailofbits/skills · ⭐6020 · NLPM 95/100 · upstream（SECURITY BLOCKED，HIGH 高危 3 个均未修，bypassPermissions 甚至被当功能宣传） · [案例](2026-07/2026-07-08-trailofbits-skills.md)
- [x] mattpocock/skills · ⭐69816 · NLPM 98/100 · upstream（SECURITY CLEAR，4 个 misc skill manifest bug 持续；plugin.json 大幅重构） · [案例](2026-07/2026-07-08-mattpocock-skills.md)

### 2026-07-06 (4 篇)
- [x] zubair-trabzada/geo-seo-claude · ⭐5411 · NLPM 88/100 · upstream（SECURITY CLEAR，全部bug已修） · [案例](2026-07/2026-07-06-zubair-trabzada-geo-seo-claude.md)
- [x] zhukunpenglinyutong/jetbrains-cc-gui · ⭐3355 · NLPM 98/100 · upstream（SECURITY BLOCKED，高危未修） · [案例](2026-07/2026-07-06-zhukunpenglinyutong-jetbrains-cc-gui.md)
- [x] zebbern/claude-code-guide · ⭐3921 · NLPM 86/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-06-zebbern-claude-code-guide.md)
- [x] zhsama/claude-sub-agent · ⭐576 · NLPM 72/100 · upstream（SECURITY CLEAR，核心bug未修） · [案例](2026-07/2026-07-06-zhsama-claude-sub-agent.md)

### 2026-07-05 (4 篇)
- [x] ykdojo/claude-code-tips · ⭐7505 · NLPM 93/100 · upstream（SECURITY REVIEW→HIGH已修） · [案例](2026-07/2026-07-05-ykdojo-claude-code-tips.md)
- [x] wuji-labs/nopua · ⭐1229 · NLPM 80/100 · upstream（SECURITY CLEAR，全部bug已修） · [案例](2026-07/2026-07-05-wuji-labs-nopua.md)
- [x] wecode-ai/Wegent · ⭐521 · NLPM 81/100 · upstream（SECURITY BLOCKED） · [案例](2026-07/2026-07-05-wecode-ai-Wegent.md)
- [x] wasp-lang/open-saas · ⭐14349 · NLPM 87/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-05-wasp-lang-open-saas.md)

### 2026-07-04 (4 篇)
- [x] wanshuiyin/Auto-claude-code-research-in-sleep · ⭐6387 · NLPM 81/100 · upstream（SECURITY BLOCKED） · [案例](2026-07/2026-07-04-wanshuiyin-Auto-claude-code-research-in-sleep.md)
- [x] vstorm-co/pydantic-deepagents · ⭐687 · NLPM 93/100 · upstream（SECURITY BLOCKED） · [案例](2026-07/2026-07-04-vstorm-co-pydantic-deepagents.md)
- [x] uditgoenka/autoresearch · ⭐3612 · NLPM 88/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-04-uditgoenka-autoresearch.md)
- [x] twostraws/SwiftUI-Agent-Skill · ⭐3806 · NLPM 89/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-04-twostraws-SwiftUI-Agent-Skill.md)

### 2026-07-02 (4 篇)
- [x] tw93/Waza · ⭐3301 · NLPM 79/100 · upstream（SECURITY BLOCKED→已部分修复） · [案例](2026-07/2026-07-02-tw93-Waza.md)
- [x] tirth8205/code-review-graph · ⭐12954 · NLPM 95/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-02-tirth8205-code-review-graph.md)
- [x] timescale/pg-aiguide · ⭐1680 · NLPM 94/100 · upstream（SECURITY REVIEW） · [案例](2026-07/2026-07-02-timescale-pg-aiguide.md)
- [x] team-attention/plugins-for-claude-natives · ⭐729 · NLPM 92/100 · upstream（SECURITY CLEAR） · [案例](2026-07/2026-07-02-team-attention-plugins-for-claude-natives.md)

### 2026-07-01 (4 篇)
- [x] sangrokjung/claude-forge · ⭐648 · NLPM 82/100 · upstream（SECURITY REVIEW） · [案例](2026-07/2026-07-01-sangrokjung-claude-forge.md)
- [x] softaworks/agent-toolkit · ⭐1629 · NLPM 90/100 · upstream · [案例](2026-07/2026-07-01-softaworks-agent-toolkit.md)
- [x] superset-sh/superset · ⭐9749 · NLPM 64/100 · upstream（SECURITY BLOCKED） · [案例](2026-07/2026-07-01-superset-sh-superset.md)
- [x] tanweai/pua · ⭐16701 · NLPM 90/100 · upstream · [案例](2026-07/2026-07-01-tanweai-pua.md)

### 2026-06-30 (4 篇)
- [x] Orchestra-Research/AI-Research-SKILLs · ⭐未收录 · NLPM 89/100 · upstream · [案例](2026-06/2026-06-30-Orchestra-Research-AI-Research-SKILLs.md)
- [x] SukinShetty/Nemp-memory · ⭐95 · NLPM 92/100 · upstream（exemplar_published） · [案例](2026-06/2026-06-30-SukinShetty-Nemp-memory.md)
- [x] Xquik-dev/x-twitter-scraper · ⭐55 · NLPM 97/100 · upstream（exemplar_published） · [案例](2026-06/2026-06-30-Xquik-dev-x-twitter-scraper.md)
- [x] samber/cc-skills-golang · ⭐1362 · NLPM 87/100 · upstream · [案例](2026-06/2026-06-30-samber-cc-skills-golang.md)

### 2026-06-28 (4 篇)
- [x] qwibitai/nanoclaw · ⭐27917 · NLPM 81/100 · upstream（SECURITY CRITICAL） · [案例](2026-06/2026-06-28-qwibitai-nanoclaw.md)
- [x] refly-ai/refly · ⭐7272 · NLPM 89/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-28-refly-ai-refly.md)
- [x] rohitg00/pro-workflow · ⭐1912 · NLPM 90/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-28-rohitg00-pro-workflow.md)
- [x] rtk-ai/rtk · ⭐28660 · NLPM 88/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-28-rtk-ai-rtk.md)

### 2026-06-27 (4 篇)
- [x] rohitg00/awesome-claude-code-toolkit · ⭐1214 · NLPM 46/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-27-rohitg00-awesome-claude-code-toolkit.md)
- [x] stablyai/orca · ⭐1130 · NLPM 96/100 · upstream · [案例](2026-06/2026-06-27-stablyai-orca.md)
- [x] study8677/antigravity-workspace-template · ⭐1128 · NLPM 62/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-27-study8677-antigravity-workspace-template.md)
- [x] vectorize-io/hindsight · ⭐10609 · NLPM 94/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-27-vectorize-io-hindsight.md)

### 2026-06-25 (4 篇)
- [x] ruvnet/ruflo · ⭐33166 · NLPM 60/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-25-ruvnet-ruflo.md)
- [x] vercel-labs/agent-browser · ⭐32739 · NLPM 99/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-25-vercel-labs-agent-browser.md)
- [x] sickn33/antigravity-awesome-skills · ⭐32502 · NLPM 82/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-25-sickn33-antigravity-awesome-skills.md)
- [x] santifer/career-ops · ⭐31983 · NLPM 91/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-25-santifer-career-ops.md)

### 2026-06-23 (4 篇)
- [x] numman-ali/n-skills · ⭐974 · NLPM 96/100 · upstream · [案例](2026-06/2026-06-23-numman-ali-n-skills.md)
- [x] nyldn/claude-octopus · ⭐2575 · NLPM 79/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-23-nyldn-claude-octopus.md)
- [x] parcadei/Continuous-Claude-v3 · ⭐1200 · NLPM 78/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-23-parcadei-Continuous-Claude-v3.md)
- [x] peterkrueck/Claude-Code-Development-Kit · ⭐1342 · NLPM 82/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-23-peterkrueck-Claude-Code-Development-Kit.md)

### 2026-06-22 (4 篇)
- [x] muratcankoylan/ralph-wiggum-marketer · ⭐720 · NLPM 90/100 · upstream · [案例](2026-06/2026-06-22-muratcankoylan-ralph-wiggum-marketer.md)
- [x] musistudio/claude-code-router · ⭐32875 · NLPM 79/100 · upstream · [案例](2026-06/2026-06-22-musistudio-claude-code-router.md)
- [x] nexu-io/open-design · ⭐20558 · NLPM 91/100 · upstream · [案例](2026-06/2026-06-22-nexu-io-open-design.md)
- [x] nicobailon/visual-explainer · ⭐7692 · NLPM 66/100 · upstream · [案例](2026-06/2026-06-22-nicobailon-visual-explainer.md)

### 2026-06-21 (4 篇)
- [x] muratcankoylan/Agent-Skills-for-Context-Engineering · ⭐15277 · NLPM 83/100 · upstream · [案例](2026-06/2026-06-21-muratcankoylan-Agent-Skills-for-Context-Engineering.md)
- [x] mukul975/Anthropic-Cybersecurity-Skills · ⭐4317 · NLPM 79/100 · upstream · [案例](2026-06/2026-06-21-mukul975-Anthropic-Cybersecurity-Skills.md)
- [x] mksglu/context-mode · ⭐9822 · NLPM 87/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-21-mksglu-context-mode.md)
- [x] mikeyobrien/ralph-orchestrator · ⭐2710 · NLPM 91/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-21-mikeyobrien-ralph-orchestrator.md)

### 2026-06-20 (4 篇)
- [x] lackeyjb/playwright-skill · ⭐2607 · NLPM 98/100 · upstream · [案例](2026-06/2026-06-20-lackeyjb-playwright-skill.md)
- [x] lijigang/ljg-skills · ⭐4860 · NLPM 90/100 · upstream · [案例](2026-06/2026-06-20-lijigang-ljg-skills.md)
- [x] m1heng/claude-plugin-weixin · ⭐556 · NLPM 97/100 · upstream · [案例](2026-06/2026-06-20-m1heng-claude-plugin-weixin.md)
- [x] manaflow-ai/cmux · ⭐15339 · NLPM 74/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-20-manaflow-ai-cmux.md)

### 2026-06-17 (4 篇)
- [x] jnMetaCode/superpowers-zh · ⭐1001 · NLPM 95/100 · upstream · [案例](2026-06/2026-06-17-jnMetaCode-superpowers-zh.md)
- [x] jordanrendric/claude-video-vision · ⭐561 · NLPM 79/100 · upstream · [案例](2026-06/2026-06-17-jordanrendric-claude-video-vision.md)
- [x] kbwo/ccmanager · ⭐1031 · NLPM 83/100 · upstream（SECURITY BLOCKED） · [案例](2026-06/2026-06-17-kbwo-ccmanager.md)
- [x] kubesphere/kubesphere · ⭐16912 · NLPM 89/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-17-kubesphere-kubesphere.md)

### 2026-06-16 (4 篇)
- [x] htdt/godogen · ⭐2839 · NLPM 91/100 · upstream · [案例](2026-06/2026-06-16-htdt-godogen.md)
- [x] iannuttall/claude-agents · ⭐2046 · NLPM 88/100 · upstream · [案例](2026-06/2026-06-16-iannuttall-claude-agents.md)
- [x] itsmostafa/aws-agent-skills · ⭐1076 · NLPM 99/100 · upstream · [案例](2026-06/2026-06-16-itsmostafa-aws-agent-skills.md)
- [x] jeremylongshore/claude-code-plugins-plus-skills · ⭐1917 · NLPM 73/100 · upstream（xiaolai case exists） · [案例](2026-06/2026-06-16-jeremylongshore-claude-code-plugins-plus-skills.md)

### 2026-06-15 (4 篇)
- [x] IvanMurzak/Unity-MCP · ⭐未收录 · NLPM 82/100 · upstream · [案例](2026-06/2026-06-15-IvanMurzak-Unity-MCP.md)
- [x] Jeffallan/claude-skills · ⭐未收录 · NLPM 92/100 · upstream · [案例](2026-06/2026-06-15-Jeffallan-claude-skills.md)
- [x] Lum1104/Understand-Anything · ⭐未收录 · NLPM 86/100 · upstream · [案例](2026-06/2026-06-15-Lum1104-Understand-Anything.md)
- [x] OthmanAdi/planning-with-files · ⭐未收录 · NLPM 91/100 · upstream · [案例](2026-06/2026-06-15-OthmanAdi-planning-with-files.md)

### 2026-06-14 (4 篇)
- [x] ComposioHQ/awesome-claude-plugins · ⭐未收录 · NLPM 88/100 · upstream · [案例](2026-06/2026-06-14-ComposioHQ-awesome-claude-plugins.md)
- [x] Donchitos/Claude-Code-Game-Studios · ⭐未收录 · NLPM 82/100 · upstream · [案例](2026-06/2026-06-14-Donchitos-Claude-Code-Game-Studios.md)
- [x] Ibrahim-3d/orchestrator-supaconductor · ⭐336 · NLPM 89/100 · upstream · [案例](2026-06/2026-06-14-Ibrahim-3d-orchestrator-supaconductor.md)
- [x] Imbad0202/academic-research-skills · ⭐未收录 · NLPM 20/100 · upstream · [案例](2026-06/2026-06-14-Imbad0202-academic-research-skills.md)

### 2026-06-12 (4 篇)
- [x] gotalab/cc-sdd · ⭐3099 · NLPM 60/100 · upstream · [案例](2026-06/2026-06-12-gotalab-cc-sdd.md)
- [x] guanyang/antigravity-skills · ⭐653 · NLPM 86/100 · upstream · [案例](2026-06/2026-06-12-guanyang-antigravity-skills.md)
- [x] hamelsmu/claude-review-loop · ⭐655 · NLPM 89/100 · upstream · [案例](2026-06/2026-06-12-hamelsmu-claude-review-loop.md)
- [x] hellowind777/hello2cc · ⭐651 · NLPM 84/100 · upstream · [案例](2026-06/2026-06-12-hellowind777-hello2cc.md)

### 2026-06-11 (4 篇)
- [x] foryourhealth111-pixel/Vibe-Skills · ⭐1570 · NLPM 79/100 · upstream · [案例](2026-06/2026-06-11-foryourhealth111-pixel-Vibe-Skills.md)
- [x] first-fluke/oh-my-agent · ⭐663 · NLPM 86/100 · upstream · [案例](2026-06/2026-06-11-first-fluke-oh-my-agent.md)
- [x] google-gemini/gemini-skills · ⭐3330 · NLPM 98/100 · upstream · [案例](2026-06/2026-06-11-google-gemini-gemini-skills.md)
- [x] google-labs-code/stitch-skills · ⭐4860 · NLPM 96/100 · upstream · [案例](2026-06/2026-06-11-google-labs-code-stitch-skills.md)

### 2026-06-10 (4 篇)
- [x] AgriciDaniel/claude-seo · ⭐未记录 · NLPM 94/100 · upstream · [案例](2026-06/2026-06-10-AgriciDaniel-claude-seo.md)
- [x] ChrisWiles/claude-code-showcase · ⭐未记录 · NLPM 81/100 · upstream · [案例](2026-06/2026-06-10-ChrisWiles-claude-code-showcase.md)
- [x] CloudAI-X/claude-workflow-v2 · ⭐未记录 · NLPM 93/100 · upstream · [案例](2026-06/2026-06-10-CloudAI-X-claude-workflow-v2.md)
- [x] EveryInc/compound-engineering-plugin · ⭐未记录 · NLPM 84/100 · upstream · [案例](2026-06/2026-06-10-EveryInc-compound-engineering-plugin.md)

### 2026-06-09 (4 篇)
- [x] safishamsi/graphify · ⭐37391 · NLPM 58/100 · upstream · [案例](2026-06/2026-06-09-safishamsi-graphify.md)
- [x] shanraisshan/claude-code-best-practice · ⭐46008 · NLPM 88/100 · upstream · [案例](2026-06/2026-06-09-shanraisshan-claude-code-best-practice.md)
- [x] upstash/context7 · ⭐53665 · NLPM 82/100 · upstream · [案例](2026-06/2026-06-09-upstash-context7.md)
- [x] wshobson/agents · ⭐33764 · NLPM 82/100 · upstream · [案例](2026-06/2026-06-09-wshobson-agents.md)

### 2026-06-08 (4 篇)
- [x] 0xmariowu/Autosearch · ⭐13 · NLPM 92/100 · upstream · [案例](2026-06/2026-06-08-0xmariowu-Autosearch.md)
- [x] 2389-research/review-squad · ⭐1 · NLPM 96/100 · upstream · [案例](2026-06/2026-06-08-2389-research-review-squad.md)
- [x] 2389-research/simmer · ⭐4 · NLPM 93/100 · upstream · [案例](2026-06/2026-06-08-2389-research-simmer.md)
- [x] 888wing/codetape · ⭐0 · NLPM 88/100 · upstream · [案例](2026-06/2026-06-08-888wing-codetape.md)

### 2026-06-07 (4 篇)
- [x] googleworkspace/cli · ⭐25330 · NLPM 99/100 · upstream · [案例](2026-06/2026-06-07-googleworkspace-cli.md)
- [x] kepano/obsidian-skills · ⭐26283 · NLPM 100/100 · upstream · [案例](2026-06/2026-06-07-kepano-obsidian-skills.md)
- [x] luongnv89/claude-howto · ⭐27150 · NLPM 77/100 · upstream · [案例](2026-06/2026-06-07-luongnv89-claude-howto.md)
- [x] mem0ai/mem0 · ⭐55456 · NLPM 91/100 · upstream · [案例](2026-06/2026-06-07-mem0ai-mem0.md)

### 2026-06-06 (4 篇)
- [x] feiskyer/claude-code-settings · ⭐1434 · NLPM 85/100 · upstream · [案例](2026-06/2026-06-06-feiskyer-claude-code-settings.md)
- [x] firetiger-oss/claude-plugin · ⭐508 · NLPM 84/100 · upstream · [案例](2026-06/2026-06-06-firetiger-oss-claude-plugin.md)
- [x] glitternetwork/pinme · ⭐3188 · NLPM 93/100 · upstream · [案例](2026-06/2026-06-06-glitternetwork-pinme.md)
- [x] gmickel/flow-next · ⭐568 · NLPM 93/100 · upstream · [案例](2026-06/2026-06-06-gmickel-flow-next.md)

### 2026-06-05 (4 篇)
- [x] github/awesome-copilot · ⭐31146 · NLPM 77/100 · upstream · [案例](2026-06/2026-06-05-github-awesome-copilot.md)
- [x] hesreallyhim/awesome-claude-code · ⭐38367 · NLPM 89/100 · upstream · [案例](2026-06/2026-06-05-hesreallyhim-awesome-claude-code.md)
- [x] iOfficeAI/AionUi · ⭐22516 · NLPM 75/100 · upstream · [案例](2026-06/2026-06-05-iOfficeAI-AionUi.md)
- [x] jarrodwatts/claude-hud · ⭐21673 · NLPM 92/100 · upstream · [案例](2026-06/2026-06-05-jarrodwatts-claude-hud.md)

### 2026-06-04 (4 篇)
- [x] evo-hq/evo · ⭐645 · NLPM 98/100 · upstream · [案例](2026-06/2026-06-04-evo-hq-evo.md)
- [x] expo/skills · ⭐1784 · NLPM 92/100 · upstream · [案例](2026-06/2026-06-04-expo-skills.md)
- [x] eyaltoledano/claude-task-master · ⭐26807 · NLPM 67/100 · upstream · [案例](2026-06/2026-06-04-eyaltoledano-claude-task-master.md)
- [x] fcakyon/claude-codex-settings · ⭐590 · NLPM 79/100 · upstream · [案例](2026-06/2026-06-04-fcakyon-claude-codex-settings.md)

### 2026-06-02 (4 篇)
- [x] backnotprop/plannotator · ⭐4561 · NLPM 93/100 · upstream · [案例](2026-06/2026-06-02-backnotprop-plannotator.md)
- [x] czlonkowski/n8n-mcp · ⭐18662 · NLPM 81/100 · upstream · [案例](2026-06/2026-06-02-czlonkowski-n8n-mcp.md)
- [x] davila7/claude-code-templates · ⭐24685 · NLPM 84/100 · upstream · [案例](2026-06/2026-06-02-davila7-claude-code-templates.md)
- [x] disler/claude-code-hooks-mastery · ⭐3566 · NLPM 77/100 · upstream · [案例](2026-06/2026-06-02-disler-claude-code-hooks-mastery.md)

### 2026-06-01 (4 篇)
- [x] forrestchang/andrej-karpathy-skills · ⭐125813 · NLPM 99/100 · upstream · [案例](2026-06/2026-06-01-forrestchang-andrej-karpathy-skills.md)
- [x] nextlevelbuilder/ui-ux-pro-max-skill · ⭐70122 · NLPM 85/100 · upstream · [案例](2026-06/2026-06-01-nextlevelbuilder-ui-ux-pro-max-skill.md)
- [x] obra/superpowers · ⭐166779 · NLPM 92/100 · upstream · [案例](2026-06/2026-06-01-obra-superpowers.md)
- [x] shareAI-lab/learn-claude-code · ⭐59868 · NLPM 88/100 · upstream · [案例](2026-06/2026-06-01-shareAI-lab-learn-claude-code.md)

### 2026-05-31 (4 篇)
- [x] avifenesh/agentsys · ⭐741 · NLPM 97/100 · upstream · [案例](2026-05/2026-05-31-avifenesh-agentsys.md)
- [x] centminmod/my-claude-code-setup · ⭐2210 · NLPM 49/100 · upstream · [案例](2026-05/2026-05-31-centminmod-my-claude-code-setup.md)
- [x] code-yeongyu/oh-my-openagent · ⭐51912 · NLPM 74/100 · upstream · [案例](2026-05/2026-05-31-code-yeongyu-oh-my-openagent.md)
- [x] codeaholicguy/ai-devkit · ⭐1128 · NLPM 96/100 · upstream · [案例](2026-05/2026-05-31-codeaholicguy-ai-devkit.md)

### 2026-05-30 (4 篇)
- [x] antfu/skills · ⭐4700 · NLPM 93/100 · upstream · [案例](2026-05/2026-05-30-antfu-skills.md)
- [x] axtonliu/axton-obsidian-visual-skills · ⭐2669 · NLPM 90/100 · upstream · [案例](2026-05/2026-05-30-axtonliu-axton-obsidian-visual-skills.md)
- [x] breaking-brake/cc-wf-studio · ⭐4893 · NLPM 87/100 · upstream · [案例](2026-05/2026-05-30-breaking-brake-cc-wf-studio.md)
- [x] caliber-ai-org/ai-setup · ⭐698 · NLPM 100/100 · upstream · [案例](2026-05/2026-05-30-caliber-ai-org-ai-setup.md)

### 2026-05-29 (4 篇)
- [x] alexgreensh/token-optimizer · ⭐646 · NLPM 98/100 · upstream · [案例](2026-05/2026-05-29-alexgreensh-token-optimizer.md)
- [x] alinaqi/claude-bootstrap · ⭐576 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-29-alinaqi-claude-bootstrap.md)
- [x] alirezarezvani/claude-code-skill-factory · ⭐721 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-29-alirezarezvani-claude-code-skill-factory.md)
- [x] alirezarezvani/claude-code-tresor · ⭐697 · NLPM 81/100 · upstream · [案例](2026-05/2026-05-29-alirezarezvani-claude-code-tresor.md)

### 2026-05-28 (4 篇)
- [x] a5c-ai/babysitter · ⭐562 · NLPM 70/100 · upstream · [案例](2026-05/2026-05-28-a5c-ai-babysitter.md)
- [x] aaron-he-zhu/seo-geo-claude-skills · ⭐1050 · NLPM 91/100 · upstream · [案例](2026-05/2026-05-28-aaron-he-zhu-seo-geo-claude-skills.md)
- [x] addyosmani/agent-skills · ⭐22697 · NLPM 94/100 · upstream · [案例](2026-05/2026-05-28-addyosmani-agent-skills.md)
- [x] addyosmani/web-quality-skills · ⭐1804 · NLPM 97/100 · upstream · [案例](2026-05/2026-05-28-addyosmani-web-quality-skills.md)

### 2026-05-27 (4 篇)
- [x] SuperClaude-Org/SuperClaude_Framework · ⭐22474 · NLPM 84/100 · upstream · [案例](2026-05/2026-05-27-SuperClaude-Org-SuperClaude_Framework.md)
- [x] The-Vibe-Company/companion · ⭐2312 · NLPM 91/100 · upstream · [案例](2026-05/2026-05-27-The-Vibe-Company-companion.md)
- [x] Yeachan-Heo/oh-my-claudecode · ⭐29671 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-27-Yeachan-Heo-oh-my-claudecode.md)
- [x] ZeframLou/call-me · ⭐2590 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-27-ZeframLou-call-me.md)

### 2026-05-26 (4 篇)
- [x] Q00/ouroboros · ⭐3637 · NLPM 69/100 · upstream · [案例](2026-05/2026-05-26-Q00-ouroboros.md)
- [x] RKiding/Awesome-finance-skills · ⭐1907 · NLPM 92/100 · upstream · [案例](2026-05/2026-05-26-RKiding-Awesome-finance-skills.md)
- [x] ReScienceLab/opc-skills · ⭐776 · NLPM 92/100 · upstream · [案例](2026-05/2026-05-26-ReScienceLab-opc-skills.md)
- [x] SimoneAvogadro/android-reverse-engineering-skill · ⭐5485 · NLPM 92/100 · upstream · [案例](2026-05/2026-05-26-SimoneAvogadro-android-reverse-engineering-skill.md)

### 2026-05-25 (4 篇)
- [x] MemPalace/mempalace · ⭐51982 · NLPM 90/100 · upstream · [案例](2026-05/2026-05-25-MemPalace-mempalace.md)
- [x] MiniMax-AI/skills · ⭐11187 · NLPM 89/100 · upstream · [案例](2026-05/2026-05-25-MiniMax-AI-skills.md)
- [x] NousResearch/hermes-agent · ⭐94145 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-25-NousResearch-hermes-agent.md)
- [x] PeonPing/peon-ping · ⭐4595 · NLPM 77/100 · upstream · [案例](2026-05/2026-05-25-PeonPing-peon-ping.md)

### 2026-05-24 (4 篇)
- [x] KhazP/vibe-coding-prompt-template · ⭐2278 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-24-KhazP-vibe-coding-prompt-template.md)
- [x] LigphiDonk/Oh-my--paper · ⭐506 · NLPM 65/100 · upstream · [案例](2026-05/2026-05-24-LigphiDonk-Oh-my--paper.md)
- [x] Manavarya09/design-extract · ⭐2482 · NLPM 94/100 · upstream · [案例](2026-05/2026-05-24-Manavarya09-design-extract.md)
- [x] OpenRaiser/NanoResearch · ⭐690 · NLPM 73/100 · upstream · [案例](2026-05/2026-05-24-OpenRaiser-NanoResearch.md)

### 2026-05-23 (4 篇)
- [x] JimLiu/baoyu-skills · ⭐16303 · NLPM 90/100 · upstream · [案例](2026-05/2026-05-23-JimLiu-baoyu-skills.md)
- [x] JuliusBrussee/cavekit · ⭐564 · NLPM 97/100 · upstream · [案例](2026-05/2026-05-23-JuliusBrussee-cavekit.md)
- [x] JuliusBrussee/caveman · ⭐22505 · NLPM 92/100 · upstream · [案例](2026-05/2026-05-23-JuliusBrussee-caveman.md)
- [x] K-Dense-AI/scientific-agent-skills · ⭐19356 · NLPM 83/100 · upstream · [案例](2026-05/2026-05-23-K-Dense-AI-scientific-agent-skills.md)

### 2026-05-22 (6 篇)
- [x] AgriciDaniel/claude-obsidian · ⭐976 · NLPM 91/100 · upstream · [案例](2026-05/2026-05-22-AgriciDaniel-claude-obsidian.md)
- [x] BayramAnnakov/claude-reflect · ⭐1000 · NLPM 99/100 · upstream · [案例](2026-05/2026-05-22-BayramAnnakov-claude-reflect.md)
- [x] Dammyjay93/interface-design · ⭐4630 · NLPM 94/100 · upstream · [案例](2026-05/2026-05-22-Dammyjay93-interface-design.md)
- [x] FlorianBruniaux/claude-code-ultimate-guide · ⭐3604 · NLPM 80/100 · upstream · [案例](2026-05/2026-05-22-FlorianBruniaux-claude-code-ultimate-guide.md)
- [x] anthropics/claude-code · ⭐125529 · NLPM 88/100 · 本地 audit · [案例](2026-05/2026-05-22-anthropics-claude-code.md)
- [x] taishi-i/awesome-japanese-nlp-resources · ⭐962 · NLPM 79/100 · 本地 audit · [案例](2026-05/2026-05-22-taishi-i-awesome-japanese-nlp-resources.md)

### 2026-05-20 (5 篇)
- [x] 0xfurai/claude-code-subagents · ⭐859 · NLPM 74/100 · upstream · [案例](2026-05/2026-05-20-0xfurai-claude-code-subagents.md)
- [x] 777genius/claude-notifications-go · ⭐528 · NLPM 63/100 · upstream · [案例](2026-05/2026-05-20-777genius-claude-notifications-go.md)
- [x] AgriciDaniel/claude-ads · ⭐2377 · NLPM 99/100 · upstream · [案例](2026-05/2026-05-20-AgriciDaniel-claude-ads.md)
- [x] AgriciDaniel/claude-blog · ⭐562 · NLPM 92/100 · upstream · [案例](2026-05/2026-05-20-AgriciDaniel-claude-blog.md)
- [x] nexu-io/html-anything · ⭐3378 · NLPM 80/100 · 本地 audit · [案例](2026-05/2026-05-20-nexu-io-html-anything.md)

---

## 📋 待处理（共 8 个 upstream，均 <10 stars——≥500 池已完全耗尽）

> ⚠️ 2026-07-29 起，所有 ≥10 stars 的 upstream audit 已全部完成。以下为剩余极低星数案例，学习价值有限，仅在无新 audit 时处理：

- [ ] jnuyens/gsd-plugin · ⭐9 · upstream
- [ ] navapbc/digital-service-orchestra · ✅ 已完成 (2026-07-29)（3★）
- [ ] agent-sh/enhance · ⭐3 · upstream
- [ ] aaronmaturen/claude-plugin · ⭐1 · upstream
- [ ] thedotmack/claude-mem · ⭐0 · upstream
- [ ] gadievron/raptor · ⭐0 · upstream
- [ ] eroslifestyle/Claude-Orchestrator-Plugin · ⭐0 · upstream
- [ ] diet103/claude-code-infrastructure-showcase · ⭐0 · upstream
- [ ] davepoon/buildwithclaude · ⭐0 · upstream
- [ ] alirezarezvani/claude-skills · ⭐N/A · upstream
- [ ] affaan-m/everything-claude-code · ⭐N/A · upstream

---

## ⏭️ 永久跳过（<500 stars 或 registry 无记录，共 64 个）

> 注：2026-07-14 将 pbakaus/impeccable、openai/symphony、agent-sh/agentsys、ccplugins/awesome-claude-code-plugins 从跳过池移出（registry null-star 数据陈旧，实际均≥500 stars）。

- ~~LerianStudio/ring · ⭐174~~ → 已完成（2026-07-25，<500 但≥500池耗尽后最接近门槛）
- ~~aaronmaturen/claude-plugin · ⭐1~~
- ~~agent-sh/agnix · ⭐207~~ → 已完成（2026-07-25，<500 但≥500池耗尽后最接近门槛）
- ~~agent-sh/enhance · ⭐3~~
- ~~agiletec-inc/airis-mcp-gateway · ⭐151~~
- ~~alirezarezvani/claude-skills · ⭐N/A~~
- ~~c0x12c/ai-toolkit · ⭐61~~
- ~~eroslifestyle/Claude-Orchestrator-Plugin · ⭐0~~
- ~~jnuyens/gsd-plugin · ⭐9~~
- ~~kangraemin/claude-inspector · ⭐115~~
- ~~kazukinagata/shinkoku · ⭐335~~ → 已完成（2026-07-19，exemplar_published=true）
- ~~krodak/clickup-cli · ⭐57~~ → 已完成（2026-07-19，exemplar_published=true）
- ~~leowux/pony · ⭐1~~
- ~~levnikolaevich/claude-code-skills · ⭐423~~ → 已完成（2026-07-24，<500 但≥500池耗尽后最接近门槛）

...（其余 53 个同理）
