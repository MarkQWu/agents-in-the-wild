# 我的学习案例索引

从 NLPM audit 报告生成的个人学习笔记。与 [上一级目录](../INDEX.md) 中 xiaolai 的案例互补：

- **xiaolai 案例**（`case-studies/*.md`）= engagement 故事，记录 xiaolai 跟仓库维护者的互动叙事
- **我的学习案例**（`case-studies/learning/<YYYY-MM>/*.md`）= 自查笔记，重点在「我有没有犯同样的错」和「能从他们设计里学到什么」

📚 **系统总纲**：[../../learning/README.md](../../learning/README.md)
📋 **进度看板**：[PROGRESS.md](PROGRESS.md)

---

## 全部案例（按生成日期降序）

> 🔄 v2 模板正在回填中。原有 14 篇 v0/v1 案例已于 2026-05-19 删除，由 Routine 按 4 篇/天的节奏用 v2 模板（含「架构作为可学习模式」+「术语表」）重新生成。预计 4 天内补齐。

| 仓库 | Stars | NLPM | 来源 | 生成日期 | 案例 |
|---|---|---|---|---|---|
| pbakaus/impeccable | ⭐46,376 | 93/100 | upstream（SECURITY REVIEW→已修，new Function() 已修，stale CLAUDE.md 引用已修，命令计数不一致持续） | 2026-07-14 | [2026-07/2026-07-14-pbakaus-impeccable.md](2026-07/2026-07-14-pbakaus-impeccable.md) |
| openai/symphony | ⭐25,947 | 89/100 | upstream（SECURITY CLEAR，5 skills 缺 ## Output 持续，AGENTS.md path 前缀缺失，模糊量词 judgment/brief/minimal/complex 持续） | 2026-07-14 | [2026-07/2026-07-14-openai-symphony.md](2026-07/2026-07-14-openai-symphony.md) |
| ccplugins/awesome-claude-code-plugins | ⭐878 | 78/100 | upstream（SECURITY CLEAR，10 个命名错误 bug 持续，~30 truncated description 持续，agent-in-wrong-dir 持续） | 2026-07-14 | [2026-07/2026-07-14-ccplugins-awesome-claude-code-plugins.md](2026-07/2026-07-14-ccplugins-awesome-claude-code-plugins.md) |
| agent-sh/agentsys | ⭐892 | 91/100 | upstream（SECURITY CLEAR，hardcoded /Users/avifen/ paths 已修/删除，12 perf skills 缺 examples 持续） | 2026-07-14 | [2026-07/2026-07-14-agent-sh-agentsys.md](2026-07/2026-07-14-agent-sh-agentsys.md) |
| xiaolai/grill-for-claude | ⭐N/A | 97/100 | upstream（SECURITY CLEAR，3 agent 缺 Output Format 未修，版本 drift 持续，新增 codex/ 双运行时支持） | 2026-07-10 | [2026-07/2026-07-10-xiaolai-grill-for-claude.md](2026-07/2026-07-10-xiaolai-grill-for-claude.md) |
| xiaolai/codex-toolkit-for-claude | ⭐N/A | 96/100 | upstream（SECURITY CLEAR，status.md $ARGUMENTS 已修，系统性无 allowed-tools 持续） | 2026-07-10 | [2026-07/2026-07-10-xiaolai-codex-toolkit-for-claude.md](2026-07/2026-07-10-xiaolai-codex-toolkit-for-claude.md) |
| openai/codex-plugin-cc | ⭐N/A | 93/100 | upstream（SECURITY REVIEW→已修，xiaolai案例存在，新增 transfer.md 命令） | 2026-07-10 | [2026-07/2026-07-10-openai-codex-plugin-cc.md](2026-07/2026-07-10-openai-codex-plugin-cc.md) |
| slavingia/skills | ⭐N/A | 97/100 | upstream（SECURITY CLEAR，plugin.json skills 字段 3 个月仍未修，模糊量词持续） | 2026-07-10 | [2026-07/2026-07-10-slavingia-skills.md](2026-07/2026-07-10-slavingia-skills.md) |
| coreyhaines31/marketingskills | ⭐0 | 96/100 | upstream（exemplar_published，SECURITY CLEAR，Output Format 缺陷从13扩至23） | 2026-07-09 | [2026-07/2026-07-09-coreyhaines31-marketingskills.md](2026-07/2026-07-09-coreyhaines31-marketingskills.md) |
| agenticnotetaking/arscontexta | ⭐0 | 96/100 | upstream（exemplar_published，SECURITY CLEAR，8/9 model bug 持续未修） | 2026-07-09 | [2026-07/2026-07-09-agenticnotetaking-arscontexta.md](2026-07/2026-07-09-agenticnotetaking-arscontexta.md) |
| 2389-research/simmer | ⭐4 | 93/100 | upstream（exemplar_published，SECURITY CLEAR，CLAUDE.md -20分持续未修） | 2026-07-09 | [2026-07/2026-07-09-2389-research-simmer.md](2026-07/2026-07-09-2389-research-simmer.md) |
| 2389-research/review-squad | ⭐1 | 96/100 | upstream（exemplar_published，SECURITY CLEAR，全部缺陷持续） | 2026-07-09 | [2026-07/2026-07-09-2389-research-review-squad.md](2026-07/2026-07-09-2389-research-review-squad.md) |
| zubair-trabzada/geo-seo-claude | ⭐5411 | 88/100 | upstream（SECURITY CLEAR，全部bug已修） | 2026-07-06 | [2026-07/2026-07-06-zubair-trabzada-geo-seo-claude.md](2026-07/2026-07-06-zubair-trabzada-geo-seo-claude.md) |
| zhukunpenglinyutong/jetbrains-cc-gui | ⭐3355 | 98/100 | upstream（SECURITY BLOCKED，高危未修） | 2026-07-06 | [2026-07/2026-07-06-zhukunpenglinyutong-jetbrains-cc-gui.md](2026-07/2026-07-06-zhukunpenglinyutong-jetbrains-cc-gui.md) |
| zebbern/claude-code-guide | ⭐3921 | 86/100 | upstream（SECURITY CLEAR） | 2026-07-06 | [2026-07/2026-07-06-zebbern-claude-code-guide.md](2026-07/2026-07-06-zebbern-claude-code-guide.md) |
| zhsama/claude-sub-agent | ⭐576 | 72/100 | upstream（SECURITY CLEAR，核心bug未修） | 2026-07-06 | [2026-07/2026-07-06-zhsama-claude-sub-agent.md](2026-07/2026-07-06-zhsama-claude-sub-agent.md) |
| ykdojo/claude-code-tips | ⭐7505 | 93/100 | upstream（SECURITY REVIEW→HIGH已修） | 2026-07-05 | [2026-07/2026-07-05-ykdojo-claude-code-tips.md](2026-07/2026-07-05-ykdojo-claude-code-tips.md) |
| wuji-labs/nopua | ⭐1229 | 80/100 | upstream（SECURITY CLEAR，全部bug已修） | 2026-07-05 | [2026-07/2026-07-05-wuji-labs-nopua.md](2026-07/2026-07-05-wuji-labs-nopua.md) |
| wecode-ai/Wegent | ⭐521 | 81/100 | upstream（SECURITY BLOCKED） | 2026-07-05 | [2026-07/2026-07-05-wecode-ai-Wegent.md](2026-07/2026-07-05-wecode-ai-Wegent.md) |
| wasp-lang/open-saas | ⭐14349 | 87/100 | upstream（SECURITY CLEAR） | 2026-07-05 | [2026-07/2026-07-05-wasp-lang-open-saas.md](2026-07/2026-07-05-wasp-lang-open-saas.md) |
| wanshuiyin/Auto-claude-code-research-in-sleep | ⭐6387 | 81/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-07-04 | [2026-07/2026-07-04-wanshuiyin-Auto-claude-code-research-in-sleep.md](2026-07/2026-07-04-wanshuiyin-Auto-claude-code-research-in-sleep.md) |
| vstorm-co/pydantic-deepagents | ⭐687 | 93/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-07-04 | [2026-07/2026-07-04-vstorm-co-pydantic-deepagents.md](2026-07/2026-07-04-vstorm-co-pydantic-deepagents.md) |
| uditgoenka/autoresearch | ⭐3612 | 88/100 | xiaolai upstream | 2026-07-04 | [2026-07/2026-07-04-uditgoenka-autoresearch.md](2026-07/2026-07-04-uditgoenka-autoresearch.md) |
| twostraws/SwiftUI-Agent-Skill | ⭐3806 | 89/100 | xiaolai upstream | 2026-07-04 | [2026-07/2026-07-04-twostraws-SwiftUI-Agent-Skill.md](2026-07/2026-07-04-twostraws-SwiftUI-Agent-Skill.md) |
| tw93/Waza | ⭐3301 | 79/100 | xiaolai upstream（SECURITY BLOCKED→REVIEW） | 2026-07-02 | [2026-07/2026-07-02-tw93-Waza.md](2026-07/2026-07-02-tw93-Waza.md) |
| tirth8205/code-review-graph | ⭐12954 | 95/100 | xiaolai upstream | 2026-07-02 | [2026-07/2026-07-02-tirth8205-code-review-graph.md](2026-07/2026-07-02-tirth8205-code-review-graph.md) |
| timescale/pg-aiguide | ⭐1680 | 94/100 | xiaolai upstream（SECURITY REVIEW） | 2026-07-02 | [2026-07/2026-07-02-timescale-pg-aiguide.md](2026-07/2026-07-02-timescale-pg-aiguide.md) |
| team-attention/plugins-for-claude-natives | ⭐729 | 92/100 | xiaolai upstream | 2026-07-02 | [2026-07/2026-07-02-team-attention-plugins-for-claude-natives.md](2026-07/2026-07-02-team-attention-plugins-for-claude-natives.md) |
| vectorize-io/hindsight | ⭐10609 | 94/100 | xiaolai upstream | 2026-06-27 | [2026-06/2026-06-27-vectorize-io-hindsight.md](2026-06/2026-06-27-vectorize-io-hindsight.md) |
| rohitg00/awesome-claude-code-toolkit | ⭐1214 | 46/100 | xiaolai upstream | 2026-06-27 | [2026-06/2026-06-27-rohitg00-awesome-claude-code-toolkit.md](2026-06/2026-06-27-rohitg00-awesome-claude-code-toolkit.md) |
| stablyai/orca | ⭐1130 | 96/100 | xiaolai upstream | 2026-06-27 | [2026-06/2026-06-27-stablyai-orca.md](2026-06/2026-06-27-stablyai-orca.md) |
| study8677/antigravity-workspace-template | ⭐1128 | 62/100 | xiaolai upstream | 2026-06-27 | [2026-06/2026-06-27-study8677-antigravity-workspace-template.md](2026-06/2026-06-27-study8677-antigravity-workspace-template.md) |
| ruvnet/ruflo | ⭐33166 | 60/100 | xiaolai upstream | 2026-06-25 | [2026-06/2026-06-25-ruvnet-ruflo.md](2026-06/2026-06-25-ruvnet-ruflo.md) |
| vercel-labs/agent-browser | ⭐32739 | 99/100 | xiaolai upstream | 2026-06-25 | [2026-06/2026-06-25-vercel-labs-agent-browser.md](2026-06/2026-06-25-vercel-labs-agent-browser.md) |
| sickn33/antigravity-awesome-skills | ⭐32502 | 82/100 | xiaolai upstream | 2026-06-25 | [2026-06/2026-06-25-sickn33-antigravity-awesome-skills.md](2026-06/2026-06-25-sickn33-antigravity-awesome-skills.md) |
| santifer/career-ops | ⭐31983 | 91/100 | xiaolai upstream | 2026-06-25 | [2026-06/2026-06-25-santifer-career-ops.md](2026-06/2026-06-25-santifer-career-ops.md) |
| nexu-io/open-design | ⭐20558 | 91/100 | xiaolai upstream | 2026-06-22 | [2026-06/2026-06-22-nexu-io-open-design.md](2026-06/2026-06-22-nexu-io-open-design.md) |
| nicobailon/visual-explainer | ⭐7692 | 66/100 | xiaolai upstream | 2026-06-22 | [2026-06/2026-06-22-nicobailon-visual-explainer.md](2026-06/2026-06-22-nicobailon-visual-explainer.md) |
| musistudio/claude-code-router | ⭐32875 | 79/100 | xiaolai upstream | 2026-06-22 | [2026-06/2026-06-22-musistudio-claude-code-router.md](2026-06/2026-06-22-musistudio-claude-code-router.md) |
| muratcankoylan/ralph-wiggum-marketer | ⭐720 | 90/100 | xiaolai upstream | 2026-06-22 | [2026-06/2026-06-22-muratcankoylan-ralph-wiggum-marketer.md](2026-06/2026-06-22-muratcankoylan-ralph-wiggum-marketer.md) |
| muratcankoylan/Agent-Skills-for-Context-Engineering | ⭐15277 | 83/100 | xiaolai upstream | 2026-06-21 | [2026-06/2026-06-21-muratcankoylan-Agent-Skills-for-Context-Engineering.md](2026-06/2026-06-21-muratcankoylan-Agent-Skills-for-Context-Engineering.md) |
| mukul975/Anthropic-Cybersecurity-Skills | ⭐4317 | 79/100 | xiaolai upstream | 2026-06-21 | [2026-06/2026-06-21-mukul975-Anthropic-Cybersecurity-Skills.md](2026-06/2026-06-21-mukul975-Anthropic-Cybersecurity-Skills.md) |
| mksglu/context-mode | ⭐9822 | 87/100 | xiaolai upstream | 2026-06-21 | [2026-06/2026-06-21-mksglu-context-mode.md](2026-06/2026-06-21-mksglu-context-mode.md) |
| mikeyobrien/ralph-orchestrator | ⭐2710 | 91/100 | xiaolai upstream | 2026-06-21 | [2026-06/2026-06-21-mikeyobrien-ralph-orchestrator.md](2026-06/2026-06-21-mikeyobrien-ralph-orchestrator.md) |
| manaflow-ai/cmux | ⭐15339 | 74/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-manaflow-ai-cmux.md](2026-06/2026-06-20-manaflow-ai-cmux.md) |
| m1heng/claude-plugin-weixin | ⭐556 | 97/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-m1heng-claude-plugin-weixin.md](2026-06/2026-06-20-m1heng-claude-plugin-weixin.md) |
| lijigang/ljg-skills | ⭐4860 | 90/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-lijigang-ljg-skills.md](2026-06/2026-06-20-lijigang-ljg-skills.md) |
| lackeyjb/playwright-skill | ⭐2607 | 98/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-lackeyjb-playwright-skill.md](2026-06/2026-06-20-lackeyjb-playwright-skill.md) |
| kubesphere/kubesphere | ⭐16912 | 89/100 | xiaolai upstream | 2026-06-17 | [2026-06/2026-06-17-kubesphere-kubesphere.md](2026-06/2026-06-17-kubesphere-kubesphere.md) |
| kbwo/ccmanager | ⭐1031 | 83/100 | xiaolai upstream | 2026-06-17 | [2026-06/2026-06-17-kbwo-ccmanager.md](2026-06/2026-06-17-kbwo-ccmanager.md) |
| jordanrendric/claude-video-vision | ⭐561 | 79/100 | xiaolai upstream | 2026-06-17 | [2026-06/2026-06-17-jordanrendric-claude-video-vision.md](2026-06/2026-06-17-jordanrendric-claude-video-vision.md) |
| jnMetaCode/superpowers-zh | ⭐1001 | 95/100 | xiaolai upstream | 2026-06-17 | [2026-06/2026-06-17-jnMetaCode-superpowers-zh.md](2026-06/2026-06-17-jnMetaCode-superpowers-zh.md) |
| iannuttall/claude-agents | ⭐2046 | 88/100 | xiaolai upstream | 2026-06-16 | [2026-06/2026-06-16-iannuttall-claude-agents.md](2026-06/2026-06-16-iannuttall-claude-agents.md) |
| htdt/godogen | ⭐2839 | 91/100 | xiaolai upstream | 2026-06-16 | [2026-06/2026-06-16-htdt-godogen.md](2026-06/2026-06-16-htdt-godogen.md) |
| itsmostafa/aws-agent-skills | ⭐1076 | 99/100 | xiaolai upstream | 2026-06-16 | [2026-06/2026-06-16-itsmostafa-aws-agent-skills.md](2026-06/2026-06-16-itsmostafa-aws-agent-skills.md) |
| jeremylongshore/claude-code-plugins-plus-skills | ⭐1917 | 73/100 | xiaolai upstream | 2026-06-16 | [2026-06/2026-06-16-jeremylongshore-claude-code-plugins-plus-skills.md](2026-06/2026-06-16-jeremylongshore-claude-code-plugins-plus-skills.md) |
| OthmanAdi/planning-with-files | ⭐未收录 | 91/100 | xiaolai upstream | 2026-06-15 | [2026-06/2026-06-15-OthmanAdi-planning-with-files.md](2026-06/2026-06-15-OthmanAdi-planning-with-files.md) |
| Lum1104/Understand-Anything | ⭐未收录 | 86/100 | xiaolai upstream | 2026-06-15 | [2026-06/2026-06-15-Lum1104-Understand-Anything.md](2026-06/2026-06-15-Lum1104-Understand-Anything.md) |
| IvanMurzak/Unity-MCP | ⭐未收录 | 82/100 | xiaolai upstream | 2026-06-15 | [2026-06/2026-06-15-IvanMurzak-Unity-MCP.md](2026-06/2026-06-15-IvanMurzak-Unity-MCP.md) |
| Jeffallan/claude-skills | ⭐未收录 | 92/100 | xiaolai upstream | 2026-06-15 | [2026-06/2026-06-15-Jeffallan-claude-skills.md](2026-06/2026-06-15-Jeffallan-claude-skills.md) |
| ComposioHQ/awesome-claude-plugins | ⭐未收录 | 88/100 | xiaolai upstream | 2026-06-14 | [2026-06/2026-06-14-ComposioHQ-awesome-claude-plugins.md](2026-06/2026-06-14-ComposioHQ-awesome-claude-plugins.md) |
| Donchitos/Claude-Code-Game-Studios | ⭐未收录 | 82/100 | xiaolai upstream | 2026-06-14 | [2026-06/2026-06-14-Donchitos-Claude-Code-Game-Studios.md](2026-06/2026-06-14-Donchitos-Claude-Code-Game-Studios.md) |
| Ibrahim-3d/orchestrator-supaconductor | ⭐336 | 89/100 | xiaolai upstream | 2026-06-14 | [2026-06/2026-06-14-Ibrahim-3d-orchestrator-supaconductor.md](2026-06/2026-06-14-Ibrahim-3d-orchestrator-supaconductor.md) |
| Imbad0202/academic-research-skills | ⭐未收录 | 20/100 | xiaolai upstream | 2026-06-14 | [2026-06/2026-06-14-Imbad0202-academic-research-skills.md](2026-06/2026-06-14-Imbad0202-academic-research-skills.md) |
| gotalab/cc-sdd | ⭐3099 | 60/100 | xiaolai upstream | 2026-06-12 | [2026-06/2026-06-12-gotalab-cc-sdd.md](2026-06/2026-06-12-gotalab-cc-sdd.md) |
| guanyang/antigravity-skills | ⭐653 | 86/100 | xiaolai upstream | 2026-06-12 | [2026-06/2026-06-12-guanyang-antigravity-skills.md](2026-06/2026-06-12-guanyang-antigravity-skills.md) |
| hamelsmu/claude-review-loop | ⭐655 | 89/100 | xiaolai upstream | 2026-06-12 | [2026-06/2026-06-12-hamelsmu-claude-review-loop.md](2026-06/2026-06-12-hamelsmu-claude-review-loop.md) |
| hellowind777/hello2cc | ⭐651 | 84/100 | xiaolai upstream | 2026-06-12 | [2026-06/2026-06-12-hellowind777-hello2cc.md](2026-06/2026-06-12-hellowind777-hello2cc.md) |
| google-labs-code/stitch-skills | ⭐4860 | 96/100 | xiaolai upstream | 2026-06-11 | [2026-06/2026-06-11-google-labs-code-stitch-skills.md](2026-06/2026-06-11-google-labs-code-stitch-skills.md) |
| google-gemini/gemini-skills | ⭐3330 | 98/100 | xiaolai upstream | 2026-06-11 | [2026-06/2026-06-11-google-gemini-gemini-skills.md](2026-06/2026-06-11-google-gemini-gemini-skills.md) |
| foryourhealth111-pixel/Vibe-Skills | ⭐1570 | 79/100 | xiaolai upstream | 2026-06-11 | [2026-06/2026-06-11-foryourhealth111-pixel-Vibe-Skills.md](2026-06/2026-06-11-foryourhealth111-pixel-Vibe-Skills.md) |
| first-fluke/oh-my-agent | ⭐663 | 86/100 | xiaolai upstream | 2026-06-11 | [2026-06/2026-06-11-first-fluke-oh-my-agent.md](2026-06/2026-06-11-first-fluke-oh-my-agent.md) |
| EveryInc/compound-engineering-plugin | 未记录 | 84/100 | xiaolai upstream | 2026-06-10 | [2026-06/2026-06-10-EveryInc-compound-engineering-plugin.md](2026-06/2026-06-10-EveryInc-compound-engineering-plugin.md) |
| AgriciDaniel/claude-seo | 未记录 | 94/100 | xiaolai upstream | 2026-06-10 | [2026-06/2026-06-10-AgriciDaniel-claude-seo.md](2026-06/2026-06-10-AgriciDaniel-claude-seo.md) |
| ChrisWiles/claude-code-showcase | 未记录 | 81/100 | xiaolai upstream | 2026-06-10 | [2026-06/2026-06-10-ChrisWiles-claude-code-showcase.md](2026-06/2026-06-10-ChrisWiles-claude-code-showcase.md) |
| CloudAI-X/claude-workflow-v2 | 未记录 | 93/100 | xiaolai upstream | 2026-06-10 | [2026-06/2026-06-10-CloudAI-X-claude-workflow-v2.md](2026-06/2026-06-10-CloudAI-X-claude-workflow-v2.md) |
| upstash/context7 | ⭐53665 | 82/100 | xiaolai upstream | 2026-06-09 | [2026-06/2026-06-09-upstash-context7.md](2026-06/2026-06-09-upstash-context7.md) |
| shanraisshan/claude-code-best-practice | ⭐46008 | 88/100 | xiaolai upstream | 2026-06-09 | [2026-06/2026-06-09-shanraisshan-claude-code-best-practice.md](2026-06/2026-06-09-shanraisshan-claude-code-best-practice.md) |
| safishamsi/graphify | ⭐37391 | 58/100 | xiaolai upstream | 2026-06-09 | [2026-06/2026-06-09-safishamsi-graphify.md](2026-06/2026-06-09-safishamsi-graphify.md) |
| wshobson/agents | ⭐33764 | 82/100 | xiaolai upstream | 2026-06-09 | [2026-06/2026-06-09-wshobson-agents.md](2026-06/2026-06-09-wshobson-agents.md) |
| 0xmariowu/Autosearch | ⭐13 | 92/100 | xiaolai upstream | 2026-06-08 | [2026-06/2026-06-08-0xmariowu-Autosearch.md](2026-06/2026-06-08-0xmariowu-Autosearch.md) |
| 2389-research/review-squad | ⭐1 | 96/100 | xiaolai upstream | 2026-06-08 | [2026-06/2026-06-08-2389-research-review-squad.md](2026-06/2026-06-08-2389-research-review-squad.md) |
| 2389-research/simmer | ⭐4 | 93/100 | xiaolai upstream | 2026-06-08 | [2026-06/2026-06-08-2389-research-simmer.md](2026-06/2026-06-08-2389-research-simmer.md) |
| 888wing/codetape | ⭐0 | 88/100 | xiaolai upstream | 2026-06-08 | [2026-06/2026-06-08-888wing-codetape.md](2026-06/2026-06-08-888wing-codetape.md) |
| googleworkspace/cli | ⭐25330 | 99/100 | xiaolai upstream | 2026-06-07 | [2026-06/2026-06-07-googleworkspace-cli.md](2026-06/2026-06-07-googleworkspace-cli.md) |
| kepano/obsidian-skills | ⭐26283 | 100/100 | xiaolai upstream | 2026-06-07 | [2026-06/2026-06-07-kepano-obsidian-skills.md](2026-06/2026-06-07-kepano-obsidian-skills.md) |
| luongnv89/claude-howto | ⭐27150 | 77/100 | xiaolai upstream | 2026-06-07 | [2026-06/2026-06-07-luongnv89-claude-howto.md](2026-06/2026-06-07-luongnv89-claude-howto.md) |
| mem0ai/mem0 | ⭐55456 | 91/100 | xiaolai upstream | 2026-06-07 | [2026-06/2026-06-07-mem0ai-mem0.md](2026-06/2026-06-07-mem0ai-mem0.md) |
| gmickel/flow-next | ⭐568 | 93/100 | xiaolai upstream | 2026-06-06 | [2026-06/2026-06-06-gmickel-flow-next.md](2026-06/2026-06-06-gmickel-flow-next.md) |
| glitternetwork/pinme | ⭐3188 | 93/100 | xiaolai upstream | 2026-06-06 | [2026-06/2026-06-06-glitternetwork-pinme.md](2026-06/2026-06-06-glitternetwork-pinme.md) |
| firetiger-oss/claude-plugin | ⭐508 | 84/100 | xiaolai upstream | 2026-06-06 | [2026-06/2026-06-06-firetiger-oss-claude-plugin.md](2026-06/2026-06-06-firetiger-oss-claude-plugin.md) |
| feiskyer/claude-code-settings | ⭐1434 | 85/100 | xiaolai upstream | 2026-06-06 | [2026-06/2026-06-06-feiskyer-claude-code-settings.md](2026-06/2026-06-06-feiskyer-claude-code-settings.md) |
| hesreallyhim/awesome-claude-code | ⭐38367 | 89/100 | xiaolai upstream | 2026-06-05 | [2026-06/2026-06-05-hesreallyhim-awesome-claude-code.md](2026-06/2026-06-05-hesreallyhim-awesome-claude-code.md) |
| github/awesome-copilot | ⭐31146 | 77/100 | xiaolai upstream | 2026-06-05 | [2026-06/2026-06-05-github-awesome-copilot.md](2026-06/2026-06-05-github-awesome-copilot.md) |
| iOfficeAI/AionUi | ⭐22516 | 75/100 | xiaolai upstream | 2026-06-05 | [2026-06/2026-06-05-iOfficeAI-AionUi.md](2026-06/2026-06-05-iOfficeAI-AionUi.md) |
| jarrodwatts/claude-hud | ⭐21673 | 92/100 | xiaolai upstream | 2026-06-05 | [2026-06/2026-06-05-jarrodwatts-claude-hud.md](2026-06/2026-06-05-jarrodwatts-claude-hud.md) |
| fcakyon/claude-codex-settings | ⭐590 | 79/100 | xiaolai upstream | 2026-06-04 | [2026-06/2026-06-04-fcakyon-claude-codex-settings.md](2026-06/2026-06-04-fcakyon-claude-codex-settings.md) |
| eyaltoledano/claude-task-master | ⭐26807 | 67/100 | xiaolai upstream | 2026-06-04 | [2026-06/2026-06-04-eyaltoledano-claude-task-master.md](2026-06/2026-06-04-eyaltoledano-claude-task-master.md) |
| expo/skills | ⭐1784 | 92/100 | xiaolai upstream | 2026-06-04 | [2026-06/2026-06-04-expo-skills.md](2026-06/2026-06-04-expo-skills.md) |
| evo-hq/evo | ⭐645 | 98/100 | xiaolai upstream | 2026-06-04 | [2026-06/2026-06-04-evo-hq-evo.md](2026-06/2026-06-04-evo-hq-evo.md) |
| disler/claude-code-hooks-mastery | ⭐3566 | 77/100 | xiaolai upstream | 2026-06-02 | [2026-06/2026-06-02-disler-claude-code-hooks-mastery.md](2026-06/2026-06-02-disler-claude-code-hooks-mastery.md) |
| backnotprop/plannotator | ⭐4561 | 93/100 | xiaolai upstream | 2026-06-02 | [2026-06/2026-06-02-backnotprop-plannotator.md](2026-06/2026-06-02-backnotprop-plannotator.md) |
| czlonkowski/n8n-mcp | ⭐18662 | 81/100 | xiaolai upstream | 2026-06-02 | [2026-06/2026-06-02-czlonkowski-n8n-mcp.md](2026-06/2026-06-02-czlonkowski-n8n-mcp.md) |
| davila7/claude-code-templates | ⭐24685 | 84/100 | xiaolai upstream | 2026-06-02 | [2026-06/2026-06-02-davila7-claude-code-templates.md](2026-06/2026-06-02-davila7-claude-code-templates.md) |
| obra/superpowers | ⭐166779 | 92/100 | xiaolai upstream | 2026-06-01 | [2026-06/2026-06-01-obra-superpowers.md](2026-06/2026-06-01-obra-superpowers.md) |
| forrestchang/andrej-karpathy-skills | ⭐125813 | 99/100 | xiaolai upstream | 2026-06-01 | [2026-06/2026-06-01-forrestchang-andrej-karpathy-skills.md](2026-06/2026-06-01-forrestchang-andrej-karpathy-skills.md) |
| nextlevelbuilder/ui-ux-pro-max-skill | ⭐70122 | 85/100 | xiaolai upstream | 2026-06-01 | [2026-06/2026-06-01-nextlevelbuilder-ui-ux-pro-max-skill.md](2026-06/2026-06-01-nextlevelbuilder-ui-ux-pro-max-skill.md) |
| shareAI-lab/learn-claude-code | ⭐59868 | 88/100 | xiaolai upstream | 2026-06-01 | [2026-06/2026-06-01-shareAI-lab-learn-claude-code.md](2026-06/2026-06-01-shareAI-lab-learn-claude-code.md) |
| centminmod/my-claude-code-setup | ⭐2210 | 49/100 | xiaolai upstream | 2026-05-31 | [2026-05/2026-05-31-centminmod-my-claude-code-setup.md](2026-05/2026-05-31-centminmod-my-claude-code-setup.md) |
| code-yeongyu/oh-my-openagent | ⭐51912 | 74/100 | xiaolai upstream | 2026-05-31 | [2026-05/2026-05-31-code-yeongyu-oh-my-openagent.md](2026-05/2026-05-31-code-yeongyu-oh-my-openagent.md) |
| codeaholicguy/ai-devkit | ⭐1128 | 96/100 | xiaolai upstream | 2026-05-31 | [2026-05/2026-05-31-codeaholicguy-ai-devkit.md](2026-05/2026-05-31-codeaholicguy-ai-devkit.md) |
| avifenesh/agentsys | ⭐741 | 97/100 | xiaolai upstream | 2026-05-31 | [2026-05/2026-05-31-avifenesh-agentsys.md](2026-05/2026-05-31-avifenesh-agentsys.md) |
| alirezarezvani/claude-code-tresor | ⭐697 | 81/100 | xiaolai upstream | 2026-05-29 | [2026-05/2026-05-29-alirezarezvani-claude-code-tresor.md](2026-05/2026-05-29-alirezarezvani-claude-code-tresor.md) |
| alirezarezvani/claude-code-skill-factory | ⭐721 | 80/100 | xiaolai upstream | 2026-05-29 | [2026-05/2026-05-29-alirezarezvani-claude-code-skill-factory.md](2026-05/2026-05-29-alirezarezvani-claude-code-skill-factory.md) |
| alinaqi/claude-bootstrap | ⭐576 | 80/100 | xiaolai upstream | 2026-05-29 | [2026-05/2026-05-29-alinaqi-claude-bootstrap.md](2026-05/2026-05-29-alinaqi-claude-bootstrap.md) |
| alexgreensh/token-optimizer | ⭐646 | 98/100 | xiaolai upstream | 2026-05-29 | [2026-05/2026-05-29-alexgreensh-token-optimizer.md](2026-05/2026-05-29-alexgreensh-token-optimizer.md) |
| addyosmani/agent-skills | ⭐22697 | 94/100 | xiaolai upstream | 2026-05-28 | [2026-05/2026-05-28-addyosmani-agent-skills.md](2026-05/2026-05-28-addyosmani-agent-skills.md) |
| addyosmani/web-quality-skills | ⭐1804 | 97/100 | xiaolai upstream | 2026-05-28 | [2026-05/2026-05-28-addyosmani-web-quality-skills.md](2026-05/2026-05-28-addyosmani-web-quality-skills.md) |
| aaron-he-zhu/seo-geo-claude-skills | ⭐1050 | 91/100 | xiaolai upstream | 2026-05-28 | [2026-05/2026-05-28-aaron-he-zhu-seo-geo-claude-skills.md](2026-05/2026-05-28-aaron-he-zhu-seo-geo-claude-skills.md) |
| a5c-ai/babysitter | ⭐562 | 70/100 | xiaolai upstream | 2026-05-28 | [2026-05/2026-05-28-a5c-ai-babysitter.md](2026-05/2026-05-28-a5c-ai-babysitter.md) |
| Yeachan-Heo/oh-my-claudecode | ⭐29671 | 80/100 | xiaolai upstream | 2026-05-27 | [2026-05/2026-05-27-Yeachan-Heo-oh-my-claudecode.md](2026-05/2026-05-27-Yeachan-Heo-oh-my-claudecode.md) |
| SuperClaude-Org/SuperClaude_Framework | ⭐22474 | 84/100 | xiaolai upstream | 2026-05-27 | [2026-05/2026-05-27-SuperClaude-Org-SuperClaude_Framework.md](2026-05/2026-05-27-SuperClaude-Org-SuperClaude_Framework.md) |
| The-Vibe-Company/companion | ⭐2312 | 91/100 | xiaolai upstream | 2026-05-27 | [2026-05/2026-05-27-The-Vibe-Company-companion.md](2026-05/2026-05-27-The-Vibe-Company-companion.md) |
| ZeframLou/call-me | ⭐2590 | 80/100 | xiaolai upstream | 2026-05-27 | [2026-05/2026-05-27-ZeframLou-call-me.md](2026-05/2026-05-27-ZeframLou-call-me.md) |
| SimoneAvogadro/android-reverse-engineering-skill | ⭐5485 | 92/100 | xiaolai upstream | 2026-05-26 | [2026-05/2026-05-26-SimoneAvogadro-android-reverse-engineering-skill.md](2026-05/2026-05-26-SimoneAvogadro-android-reverse-engineering-skill.md) |
| ReScienceLab/opc-skills | ⭐776 | 92/100 | xiaolai upstream | 2026-05-26 | [2026-05/2026-05-26-ReScienceLab-opc-skills.md](2026-05/2026-05-26-ReScienceLab-opc-skills.md) |
| RKiding/Awesome-finance-skills | ⭐1907 | 92/100 | xiaolai upstream | 2026-05-26 | [2026-05/2026-05-26-RKiding-Awesome-finance-skills.md](2026-05/2026-05-26-RKiding-Awesome-finance-skills.md) |
| Q00/ouroboros | ⭐3637 | 69/100 | xiaolai upstream | 2026-05-26 | [2026-05/2026-05-26-Q00-ouroboros.md](2026-05/2026-05-26-Q00-ouroboros.md) |
| PeonPing/peon-ping | ⭐4595 | 77/100 | xiaolai upstream | 2026-05-25 | [2026-05/2026-05-25-PeonPing-peon-ping.md](2026-05/2026-05-25-PeonPing-peon-ping.md) |
| MiniMax-AI/skills | ⭐11187 | 89/100 | xiaolai upstream | 2026-05-25 | [2026-05/2026-05-25-MiniMax-AI-skills.md](2026-05/2026-05-25-MiniMax-AI-skills.md) |
| NousResearch/hermes-agent | ⭐94145 | 80/100 | xiaolai upstream | 2026-05-25 | [2026-05/2026-05-25-NousResearch-hermes-agent.md](2026-05/2026-05-25-NousResearch-hermes-agent.md) |
| MemPalace/mempalace | ⭐51982 | 90/100 | xiaolai upstream | 2026-05-25 | [2026-05/2026-05-25-MemPalace-mempalace.md](2026-05/2026-05-25-MemPalace-mempalace.md) |
| caliber-ai-org/ai-setup | ⭐698 | 100/100 | xiaolai upstream | 2026-05-30 | [2026-05/2026-05-30-caliber-ai-org-ai-setup.md](2026-05/2026-05-30-caliber-ai-org-ai-setup.md) |
| breaking-brake/cc-wf-studio | ⭐4893 | 87/100 | xiaolai upstream | 2026-05-30 | [2026-05/2026-05-30-breaking-brake-cc-wf-studio.md](2026-05/2026-05-30-breaking-brake-cc-wf-studio.md) |
| axtonliu/axton-obsidian-visual-skills | ⭐2669 | 90/100 | xiaolai upstream | 2026-05-30 | [2026-05/2026-05-30-axtonliu-axton-obsidian-visual-skills.md](2026-05/2026-05-30-axtonliu-axton-obsidian-visual-skills.md) |
| antfu/skills | ⭐4700 | 93/100 | xiaolai upstream | 2026-05-30 | [2026-05/2026-05-30-antfu-skills.md](2026-05/2026-05-30-antfu-skills.md) |
| Manavarya09/design-extract | ⭐2482 | 94/100 | xiaolai upstream | 2026-05-24 | [2026-05/2026-05-24-Manavarya09-design-extract.md](2026-05/2026-05-24-Manavarya09-design-extract.md) |
| OpenRaiser/NanoResearch | ⭐690 | 73/100 | xiaolai upstream | 2026-05-24 | [2026-05/2026-05-24-OpenRaiser-NanoResearch.md](2026-05/2026-05-24-OpenRaiser-NanoResearch.md) |
| KhazP/vibe-coding-prompt-template | ⭐2278 | 80/100 | xiaolai upstream | 2026-05-24 | [2026-05/2026-05-24-KhazP-vibe-coding-prompt-template.md](2026-05/2026-05-24-KhazP-vibe-coding-prompt-template.md) |
| LigphiDonk/Oh-my--paper | ⭐506 | 65/100 | xiaolai upstream | 2026-05-24 | [2026-05/2026-05-24-LigphiDonk-Oh-my--paper.md](2026-05/2026-05-24-LigphiDonk-Oh-my--paper.md) |
| K-Dense-AI/scientific-agent-skills | ⭐19356 | 83/100 | xiaolai upstream | 2026-05-23 | [2026-05/2026-05-23-K-Dense-AI-scientific-agent-skills.md](2026-05/2026-05-23-K-Dense-AI-scientific-agent-skills.md) |
| JuliusBrussee/caveman | ⭐22505 | 92/100 | xiaolai upstream | 2026-05-23 | [2026-05/2026-05-23-JuliusBrussee-caveman.md](2026-05/2026-05-23-JuliusBrussee-caveman.md) |
| JuliusBrussee/cavekit | ⭐564 | 97/100 | xiaolai upstream | 2026-05-23 | [2026-05/2026-05-23-JuliusBrussee-cavekit.md](2026-05/2026-05-23-JuliusBrussee-cavekit.md) |
| JimLiu/baoyu-skills | ⭐16303 | 90/100 | xiaolai upstream | 2026-05-23 | [2026-05/2026-05-23-JimLiu-baoyu-skills.md](2026-05/2026-05-23-JimLiu-baoyu-skills.md) |
| anthropics/claude-code | ⭐125529 | 88/100 | 本地 audit | 2026-05-22 | [2026-05/2026-05-22-anthropics-claude-code.md](2026-05/2026-05-22-anthropics-claude-code.md) |
| taishi-i/awesome-japanese-nlp-resources | ⭐962 | 79/100 | 本地 audit | 2026-05-22 | [2026-05/2026-05-22-taishi-i-awesome-japanese-nlp-resources.md](2026-05/2026-05-22-taishi-i-awesome-japanese-nlp-resources.md) |
| FlorianBruniaux/claude-code-ultimate-guide | ⭐3604 | 80/100 | xiaolai upstream | 2026-05-22 | [2026-05/2026-05-22-FlorianBruniaux-claude-code-ultimate-guide.md](2026-05/2026-05-22-FlorianBruniaux-claude-code-ultimate-guide.md) |
| Dammyjay93/interface-design | ⭐4630 | 94/100 | xiaolai upstream | 2026-05-22 | [2026-05/2026-05-22-Dammyjay93-interface-design.md](2026-05/2026-05-22-Dammyjay93-interface-design.md) |
| BayramAnnakov/claude-reflect | ⭐1000 | 99/100 | xiaolai upstream | 2026-05-22 | [2026-05/2026-05-22-BayramAnnakov-claude-reflect.md](2026-05/2026-05-22-BayramAnnakov-claude-reflect.md) |
| AgriciDaniel/claude-obsidian | ⭐976 | 91/100 | xiaolai upstream | 2026-05-22 | [2026-05/2026-05-22-AgriciDaniel-claude-obsidian.md](2026-05/2026-05-22-AgriciDaniel-claude-obsidian.md) |
| nexu-io/html-anything | ⭐3378 | 80/100 | 本地 audit | 2026-05-20 | [2026-05/2026-05-20-nexu-io-html-anything.md](2026-05/2026-05-20-nexu-io-html-anything.md) |
| AgriciDaniel/claude-blog | ⭐562 | 92/100 | xiaolai upstream | 2026-05-20 | [2026-05/2026-05-20-AgriciDaniel-claude-blog.md](2026-05/2026-05-20-AgriciDaniel-claude-blog.md) |
| AgriciDaniel/claude-ads | ⭐2377 | 99/100 | xiaolai upstream | 2026-05-20 | [2026-05/2026-05-20-AgriciDaniel-claude-ads.md](2026-05/2026-05-20-AgriciDaniel-claude-ads.md) |
| 777genius/claude-notifications-go | ⭐528 | 63/100 | xiaolai upstream | 2026-05-20 | [2026-05/2026-05-20-777genius-claude-notifications-go.md](2026-05/2026-05-20-777genius-claude-notifications-go.md) |
| 0xfurai/claude-code-subagents | ⭐859 | 74/100 | xiaolai upstream | 2026-05-20 | [2026-05/2026-05-20-0xfurai-claude-code-subagents.md](2026-05/2026-05-20-0xfurai-claude-code-subagents.md) |
| lackeyjb/playwright-skill | ⭐2607 | 98/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-lackeyjb-playwright-skill.md](2026-06/2026-06-20-lackeyjb-playwright-skill.md) |
| lijigang/ljg-skills | ⭐4860 | 90/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-lijigang-ljg-skills.md](2026-06/2026-06-20-lijigang-ljg-skills.md) |
| m1heng/claude-plugin-weixin | ⭐556 | 97/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-m1heng-claude-plugin-weixin.md](2026-06/2026-06-20-m1heng-claude-plugin-weixin.md) |
| manaflow-ai/cmux | ⭐15339 | 74/100 | xiaolai upstream | 2026-06-20 | [2026-06/2026-06-20-manaflow-ai-cmux.md](2026-06/2026-06-20-manaflow-ai-cmux.md) |

| numman-ali/n-skills | ⭐974 | 96/100 | xiaolai upstream | 2026-06-23 | [2026-06/2026-06-23-numman-ali-n-skills.md](2026-06/2026-06-23-numman-ali-n-skills.md) |
| nyldn/claude-octopus | ⭐2575 | 79/100 | xiaolai upstream | 2026-06-23 | [2026-06/2026-06-23-nyldn-claude-octopus.md](2026-06/2026-06-23-nyldn-claude-octopus.md) |
| parcadei/Continuous-Claude-v3 | ⭐1200 | 78/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-06-23 | [2026-06/2026-06-23-parcadei-Continuous-Claude-v3.md](2026-06/2026-06-23-parcadei-Continuous-Claude-v3.md) |
| peterkrueck/Claude-Code-Development-Kit | ⭐1342 | 82/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-06-23 | [2026-06/2026-06-23-peterkrueck-Claude-Code-Development-Kit.md](2026-06/2026-06-23-peterkrueck-Claude-Code-Development-Kit.md) |

| qwibitai/nanoclaw | ⭐27917 | 81/100 | xiaolai upstream（SECURITY CRITICAL） | 2026-06-28 | [2026-06/2026-06-28-qwibitai-nanoclaw.md](2026-06/2026-06-28-qwibitai-nanoclaw.md) |
| refly-ai/refly | ⭐7272 | 89/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-06-28 | [2026-06/2026-06-28-refly-ai-refly.md](2026-06/2026-06-28-refly-ai-refly.md) |
| rohitg00/pro-workflow | ⭐1912 | 90/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-06-28 | [2026-06/2026-06-28-rohitg00-pro-workflow.md](2026-06/2026-06-28-rohitg00-pro-workflow.md) |
| rtk-ai/rtk | ⭐28660 | 88/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-06-28 | [2026-06/2026-06-28-rtk-ai-rtk.md](2026-06/2026-06-28-rtk-ai-rtk.md) |
| Orchestra-Research/AI-Research-SKILLs | ⭐未收录 | 89/100 | xiaolai upstream | 2026-06-30 | [2026-06/2026-06-30-Orchestra-Research-AI-Research-SKILLs.md](2026-06/2026-06-30-Orchestra-Research-AI-Research-SKILLs.md) |
| SukinShetty/Nemp-memory | ⭐95 | 92/100 | xiaolai upstream | 2026-06-30 | [2026-06/2026-06-30-SukinShetty-Nemp-memory.md](2026-06/2026-06-30-SukinShetty-Nemp-memory.md) |
| Xquik-dev/x-twitter-scraper | ⭐55 | 97/100 | xiaolai upstream | 2026-06-30 | [2026-06/2026-06-30-Xquik-dev-x-twitter-scraper.md](2026-06/2026-06-30-Xquik-dev-x-twitter-scraper.md) |
| samber/cc-skills-golang | ⭐1362 | 87/100 | xiaolai upstream | 2026-06-30 | [2026-06/2026-06-30-samber-cc-skills-golang.md](2026-06/2026-06-30-samber-cc-skills-golang.md) |
| sangrokjung/claude-forge | ⭐648 | 82/100 | xiaolai upstream（SECURITY REVIEW） | 2026-07-01 | [2026-07/2026-07-01-sangrokjung-claude-forge.md](2026-07/2026-07-01-sangrokjung-claude-forge.md) |
| softaworks/agent-toolkit | ⭐1629 | 90/100 | xiaolai upstream | 2026-07-01 | [2026-07/2026-07-01-softaworks-agent-toolkit.md](2026-07/2026-07-01-softaworks-agent-toolkit.md) |
| superset-sh/superset | ⭐9749 | 64/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-07-01 | [2026-07/2026-07-01-superset-sh-superset.md](2026-07/2026-07-01-superset-sh-superset.md) |
| tanweai/pua | ⭐16701 | 90/100 | xiaolai upstream | 2026-07-01 | [2026-07/2026-07-01-tanweai-pua.md](2026-07/2026-07-01-tanweai-pua.md) |

| anthropics/claude-plugins-official | ⭐31759 | 96/100 | xiaolai upstream（SECURITY REVIEW） | 2026-07-08 | [2026-07/2026-07-08-anthropics-claude-plugins-official.md](2026-07/2026-07-08-anthropics-claude-plugins-official.md) |
| hashicorp/agent-skills | ⭐702 | 98/100 | xiaolai upstream（SECURITY CLEAR） | 2026-07-08 | [2026-07/2026-07-08-hashicorp-agent-skills.md](2026-07/2026-07-08-hashicorp-agent-skills.md) |
| trailofbits/skills | ⭐6020 | 95/100 | xiaolai upstream（SECURITY BLOCKED） | 2026-07-08 | [2026-07/2026-07-08-trailofbits-skills.md](2026-07/2026-07-08-trailofbits-skills.md) |
| mattpocock/skills | ⭐69816 | 98/100 | xiaolai upstream | 2026-07-08 | [2026-07/2026-07-08-mattpocock-skills.md](2026-07/2026-07-08-mattpocock-skills.md) |
