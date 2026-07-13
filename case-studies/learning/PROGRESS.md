# 学习进度

**总量**：210 audit reports（207 upstream + 3 local）
**已学**：175（截至 2026-07-13）
**<500 星跳过**：68（注：0xmariowu/Autosearch 等低星仓库在 upstream 池中，已生成案例；SukinShetty/Nemp-memory、Xquik-dev/x-twitter-scraper 因 exemplar_published=true 加入案例）
**待处理**：35 个 upstream（≥500 stars 或 exemplar_published；2026-07-13 批次完成 4 个 exemplar_published 仓库：czlonkowski/n8n-skills、dontbesilent2025/dbskill、kazukinagata/shinkoku、krodak/clickup-cli）
**最后更新**：2026-07-13

---

## ✅ 已完成（按生成日期降序）

### 2026-07-13 (4 篇)
- [x] czlonkowski/n8n-skills · ⭐N/A · NLPM 93/100 · upstream（exemplar: R04/R06/R07/R08；9→15 skills；新增 hooks 执法层；vague words 持续；hooks post-tool-use 路由值得借鉴） · [案例](2026-07/2026-07-13-czlonkowski-n8n-skills.md)
- [x] dontbesilent2025/dbskill · ⭐N/A · NLPM 90/100 · upstream（exemplar: R04/R05/R07/R08；12→27 skills；dbskill-upgrade HIGH 安全已移除；/文风分析 死引用已修；record-demo.sh sed 注入已修；新增 scaffold-as-a-skill 模式） · [案例](2026-07/2026-07-13-dontbesilent2025-dbskill.md)
- [x] kazukinagata/shinkoku · ⭐335 · NLPM 94/100 · upstream（exemplar: R01/R04/R06/R07/R08；整数公式精度设计；setup unpinned git URL 持续；妥当か 持续；新增 e-tax/research/ 逐屏研究文档） · [案例](2026-07/2026-07-13-kazukinagata-shinkoku.md)
- [x] krodak/clickup-cli · ⭐57 · NLPM 95/100 · upstream（exemplar: R01/R02/R04/R05/R06/R08；NL+CLI 混合架构；30 触发短语；plugin.json 无 skills 数组持续；caret 依赖持续；新增 docs/superpowers/ + AGENTS.md） · [案例](2026-07/2026-07-13-krodak-clickup-cli.md)

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

## 📋 待处理（按 stars 降序，共 63 个 upstream ≥500 stars）

> 优先级高（≥3000 stars）：

- [ ] wasp-lang/open-saas · ⭐14349 · upstream
- [ ] ykdojo/claude-code-tips · ⭐7505 · upstream
- [ ] wanshuiyin/Auto-claude-code-research-in-sleep · ✅ 已完成 (2026-07-04)
- [ ] zubair-trabzada/geo-seo-claude · ⭐5411 · upstream
- [ ] zebbern/claude-code-guide · ⭐3921 · upstream
- [ ] twostraws/SwiftUI-Agent-Skill · ✅ 已完成 (2026-07-04)
- [ ] uditgoenka/autoresearch · ✅ 已完成 (2026-07-04)
- [ ] zhukunpenglinyutong/jetbrains-cc-gui · ⭐3355 · upstream
- [ ] tw93/Waza · ✅ 已完成 (2026-07-02)
- [ ] timescale/pg-aiguide · ✅ 已完成 (2026-07-02)

> 中等优先级（500-3000 stars）：

- [ ] wuji-labs/nopua · ⭐1229 · upstream
- [ ] timescale/pg-aiguide · ✅ 已完成 (2026-07-02)
- [ ] tirth8205/code-review-graph · ✅ 已完成 (2026-07-02)
- [ ] team-attention/plugins-for-claude-natives · ✅ 已完成 (2026-07-02)
- [ ] vstorm-co/pydantic-deepagents · ✅ 已完成 (2026-07-04)
- [ ] zhsama/claude-sub-agent · ⭐576 · upstream
- [ ] wecode-ai/Wegent · ⭐521 · upstream
- [ ] josstei/maestro-orchestrate · ⭐370 · upstream（<500，低优先）

...（其余 48 个 <500 stars 或 N/A，视情况处理）

---

## ⏭️ 永久跳过（<500 stars 或 registry 无记录，共 68 个）

- ~~LerianStudio/ring · ⭐174~~
- ~~aaronmaturen/claude-plugin · ⭐1~~
- ~~agent-sh/agentsys · ⭐N/A~~
- ~~agent-sh/agnix · ⭐207~~
- ~~agent-sh/enhance · ⭐3~~
- ~~agiletec-inc/airis-mcp-gateway · ⭐151~~
- ~~alirezarezvani/claude-skills · ⭐N/A~~
- ~~c0x12c/ai-toolkit · ⭐61~~
- ~~eroslifestyle/Claude-Orchestrator-Plugin · ⭐0~~
- ~~jnuyens/gsd-plugin · ⭐9~~
- ~~kangraemin/claude-inspector · ⭐115~~
- ~~kazukinagata/shinkoku · ⭐335~~ ✅ 已完成（2026-07-13，exemplar_published=true）
- ~~krodak/clickup-cli · ⭐57~~ ✅ 已完成（2026-07-13，exemplar_published=true）
- ~~leowux/pony · ⭐1~~
- ~~levnikolaevich/claude-code-skills · ⭐423~~

...（其余 53 个同理）
