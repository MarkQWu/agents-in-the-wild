# NLPM Audit: nexu-io/html-anything
**Date**: 2026-05-19  |  **Artifacts**: 76  |  **Strategy**: progressive
**NL Score**: 80/100
**Security**: CLEAR
**Bugs**: 2  |  **Quality Issues**: 22  |  **Security Findings**: 2

## NL Score Summary
| File | Type | Score | Top Issue |
|------|------|-------|-----------|
| CLAUDE.md | config | 50 | Missing name & description frontmatter (-25 each) |
| next/src/lib/templates/skills/web-proto-soft/SKILL.md | skill | 65 | Missing output format; 4-line body provides no technical spec |
| next/src/lib/templates/skills/motion-frames/SKILL.md | skill | 70 | Missing output format; 9-line body, "流畅" is vague |
| next/src/lib/templates/skills/web-proto-editorial/SKILL.md | skill | 70 | Missing output format; 8-line body with no HTML/CSS anchors |
| next/src/lib/templates/skills/hr-onboarding/SKILL.md | skill | 70 | Missing output format; body lists content sections only |
| next/src/lib/templates/skills/pm-spec/SKILL.md | skill | 70 | Missing output format; body lists content sections only |
| next/src/lib/templates/skills/social-media-dashboard/SKILL.md | skill | 70 | Missing output format; body lists content areas, no tech spec |
| next/src/lib/templates/skills/pricing-page/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/dating-web/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/wireframe-sketch/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/dashboard/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/live-dashboard/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/team-okrs/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/meeting-notes/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/web-proto-brutalist/SKILL.md | skill | 72 | Very brief body; no output format |
| next/src/lib/templates/skills/deck-simple/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/deck-tech-sharing/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/docs-page/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/deck-course-module/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/deck-product-launch/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/kanban-board/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/digital-eguide/SKILL.md | skill | 75 | Brief body; "lifestyle / creator brand 调子" is vague |
| next/src/lib/templates/skills/deck-graphify-dark/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/gamified-app/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/social-media-matrix/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/mobile-app/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/flowai-team-dashboard/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/mobile-onboarding/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/sprite-animation/SKILL.md | skill | 75 | Brief body; no output format explicit |
| next/src/lib/templates/skills/magazine-poster/SKILL.md | skill | 78 | No explicit single-file HTML output spec |
| next/src/lib/templates/skills/deck-magazine-web/SKILL.md | skill | 78 | Brief body; mentions HTML deck implicitly |
| next/src/lib/templates/skills/deck-xhs-post/SKILL.md | skill | 78 | Brief body; dimensions present but no explicit HTML |
| next/src/lib/templates/skills/deck-presenter-mode/SKILL.md | skill | 78 | Brief body; popup structure described |
| next/src/lib/templates/skills/deck-hermes-cyber/SKILL.md | skill | 78 | Brief body; hex colors present |
| next/src/lib/templates/skills/deck-dir-key-nav/SKILL.md | skill | 78 | Brief body; color palette present |
| next/src/lib/templates/skills/deck-xhs-pastel/SKILL.md | skill | 78 | Brief body; SVG donut mentioned |
| next/src/lib/templates/skills/deck-safety-alert/SKILL.md | skill | 78 | Brief body; design elements specific |
| next/src/lib/templates/skills/weekly-update/SKILL.md | skill | 78 | Brief body; keyboard nav mentioned |
| next/src/lib/templates/skills/deck-obsidian-claude/SKILL.md | skill | 78 | Brief body; hex colors present |
| next/src/lib/templates/skills/deck-xhs-white/SKILL.md | skill | 78 | Brief body; rainbow bar spec present |
| next/src/lib/templates/skills/invoice/SKILL.md | skill | 78 | Brief body; @media print implies HTML |
| next/src/lib/templates/skills/waitlist-page/SKILL.md | skill | 78 | Brief body; SVG gradient mesh mentioned |
| next/src/lib/templates/skills/deck-blueprint/SKILL.md | skill | 80 | Brief body; SVG mentioned, no explicit HTML |
| next/src/lib/templates/skills/deck-pitch/SKILL.md | skill | 80 | Brief body; 10-page structure present |
| next/src/lib/templates/skills/poster-hero/SKILL.md | skill | 80 | Tailwind classes present; no explicit single-file HTML |
| next/src/lib/templates/skills/finance-report/SKILL.md | skill | 80 | Chart.js/ECharts spec; no explicit HTML |
| next/src/lib/templates/skills/eng-runbook/SKILL.md | skill | 80 | Mono code block mentioned; no explicit HTML |
| next/src/lib/templates/skills/email-marketing/SKILL.md | skill | 82 | `table role='presentation'` spec; good email HTML implied |
| next/src/lib/templates/skills/card-twitter/SKILL.md | skill | 82 | Tailwind container dimensions present |
| next/src/lib/templates/skills/blog-post/SKILL.md | skill | 82 | Good section structure; no explicit single-file HTML |
| next/src/lib/templates/skills/social-carousel/SKILL.md | skill | 82 | 1080×1080 dimensions; brief body |
| next/src/lib/templates/skills/saas-landing/SKILL.md | skill | 82 | Glassmorphism cards and responsive breakpoints mentioned |
| next/src/lib/templates/skills/card-xiaohongshu/SKILL.md | skill | 85 | Example present; N-card system clear; no "单文件 HTML" explicit |
| next/src/lib/templates/skills/article-magazine/SKILL.md | skill | 85 | Example present; section structure detailed |
| next/src/lib/templates/skills/resume-modern/SKILL.md | skill | 85 | Example present; A4 dimensions explicit |
| next/src/lib/templates/skills/prototype-web/SKILL.md | skill | 85 | Example present; full sections described |
| next/src/lib/templates/skills/deck-replit/SKILL.md | skill | 85 | Example present; theme system clear |
| next/src/lib/templates/skills/video-hyperframes/SKILL.md | skill | 88 | Example present; frame structure and JS auto-play clear |
| next/src/lib/templates/skills/ppt-keynote/SKILL.md | skill | 88 | Example present; keyboard nav + hash sync explicit |
| next/src/lib/templates/skills/data-report/SKILL.md | skill | 88 | Example present; Chart.js/ECharts + fixed-height container rule |
| next/src/lib/templates/skills/deck-swiss-international/SKILL.md | skill | 93 | Example present; 22 layouts; single-file HTML explicit |
| next/src/lib/templates/skills/deck-guizang-editorial/SKILL.md | skill | 93 | Example present; 10 layouts; single-file HTML explicit |
| next/src/lib/templates/skills/social-reddit-card/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/social-spotify-card/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/frame-light-leak-cinema/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/frame-liquid-bg-hero/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/frame-logo-outro/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/vfx-text-cursor/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/frame-data-chart-nyt/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/frame-macos-notification/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/deck-open-slide-canvas/SKILL.md | skill | 93 | Example present; full spec; single-file HTML explicit |
| next/src/lib/templates/skills/social-x-post-card/SKILL.md | skill | 95 | Full spec; example; single-file HTML; detailed constraints |
| next/src/lib/templates/skills/frame-flowchart-sticky/SKILL.md | skill | 95 | Full spec; example; single-file HTML; node/edge rules precise |
| next/src/lib/templates/skills/doc-kami-parchment/SKILL.md | skill | 95 | Full spec; example; single-file HTML; 8 doc-type variants |
| next/src/lib/templates/skills/frame-glitch-title/SKILL.md | skill | 95 | Full spec; example; single-file HTML; SVG filter spec |
| next/src/lib/templates/skills/mockup-device-3d/SKILL.md | skill | 95 | Full spec; example; single-file HTML; 3D transform rules |

## Security Scan
| Severity | Count |
|----------|-------|
| Critical | 0 |
| High | 0 |
| Medium | 1 |
| Low | 1 |

### Execution Surface Inventory
| Surface | Files |
|---------|-------|
| Hooks | 0 |
| Scripts (.sh/.py/.js) | 0 |
| MCP configs (.mcp.json) | 0 |
| Package manifests | 3 (package.json, next/package.json, e2e/package.json) |
| Requirements.txt | 0 |

### Security Findings
| # | Severity | File | Line | Pattern | Description |
|---|----------|------|------|---------|-------------|
| 1 | Medium | next/package.json | 27 | SEC-unpinned-semver | `xlsx@^0.18.5` is the last open-source SheetJS release (community fork); this version has known unresolved CVEs and no security backports since the 2023 license change. The `^` range allows auto-upgrade within the 0.18.x line only, mitigating promotion to newer proprietary builds, but the library itself is effectively unmaintained. |
| 2 | Low | next/package.json | 7–29 | SEC-unpinned-semver | All production dependencies use `^` semver ranges; while pnpm lockfile pins resolved versions, the ranges allow minor-version drift on `pnpm update`, including for `marked@^18` and `modern-screenshot@^4.7` which execute in the rendering pipeline. |

## Bugs (PR-worthy)
| # | File | Issue | Impact |
|---|------|-------|--------|
| 1 | CLAUDE.md | Missing `name` frontmatter field | NLPM scanner cannot register this file as a named artifact; discovery and scoring automation skip it |
| 2 | CLAUDE.md | Missing `description` frontmatter field | NLPM scanner cannot classify or summarize this artifact; tooling that requires description will error |

## Security Fixes (PR-worthy, Medium/Low only)
| # | File | Issue | Suggested Fix |
|---|------|-------|---------------|
| 1 | next/package.json | `xlsx@^0.18.5` — deprecated open-source SheetJS, effectively unmaintained | Replace with actively maintained alternative (`exceljs`, `@e965/xlsx`, or the commercial SheetJS Pro if acceptable); audit any file-parsing codepaths for path-traversal risks after migration |
| 2 | next/package.json | Broad `^` semver ranges on rendering-pipeline deps (`marked`, `modern-screenshot`) | Pin exact versions in package.json or use pnpm's `catalog:` protocol to centralize version governance; prioritize `marked@^18` and `modern-screenshot@^4.7` |

## Quality Issues (informational)
| # | File | Issue | Penalty |
|---|------|-------|---------|
| 1 | next/src/lib/templates/skills/web-proto-soft/SKILL.md | Body is 4 lines; no output format (single-file HTML, dimensions, or rendering context) specified | -10 |
| 2 | next/src/lib/templates/skills/motion-frames/SKILL.md | Body is 9 lines; no output format; "流畅可循环" (smooth loop) is vague without timing spec | -10 |
| 3 | next/src/lib/templates/skills/hr-onboarding/SKILL.md | Body lists content sections only; no HTML/CSS/dimension anchors to establish output format | -10 |
| 4 | next/src/lib/templates/skills/pm-spec/SKILL.md | Body lists content sections only; no HTML/CSS/dimension anchors to establish output format | -10 |
| 5 | next/src/lib/templates/skills/social-media-dashboard/SKILL.md | Body lists content areas only; no technical spec implies what the AI should produce | -10 |
| 6 | next/src/lib/templates/skills/web-proto-editorial/SKILL.md | Body is 8 lines; "Ambient micro-motion" and "warm monochrome canvas" are vague without hex/timing | -10 |
| 7 | next/src/lib/templates/skills/digital-eguide/SKILL.md | "lifestyle / creator brand 调子, 柔和米色" — no hex values or constraints on color | -4 |
| 8 | next/src/lib/templates/skills/motion-frames/SKILL.md | "电影感调色 + 1 个霓虹 accent" — "电影感" (cinematic) and "霓虹" (neon) without hex range | -2 |
| 9 | next/src/lib/templates/skills/web-proto-soft/SKILL.md | "Spring motion" without easing function, duration, or CSS property spec | -2 |
| 10 | next/src/lib/templates/skills/gamified-app/SKILL.md | "醒目的 quest tile 渐变" (eye-catching gradient) — no color or direction constraint | -2 |
| 11 | next/src/lib/templates/skills/social-carousel/SKILL.md | "颜色统一一套调色板, 卡片之间渐进切换" — no palette specified; agent must guess | -4 |
| 12 | next/src/lib/templates/skills/live-dashboard/SKILL.md | "Notion-callout / toggle / 数据库表配色风格" — design reference without concrete values | -2 |
| 13 | CLAUDE.md | File contains only `@AGENTS.md`; no standalone NL content; missing both frontmatter fields | -50 (scoring impact) |
| 14 | next/src/lib/templates/skills/web-proto-brutalist/SKILL.md | Body is 9 lines; no output dimensions or explicit format | -5 |
| 15 | next/src/lib/templates/skills/wireframe-sketch/SKILL.md | "不要规规矩矩的对齐" (not neatly aligned) — provides intent but no concrete layout constraint | -2 |
| 16 | next/src/lib/templates/skills/deck-simple/SKILL.md | "干净通用" (clean and general) — vague without a reference visual or style constraint | -2 |
| 17 | next/src/lib/templates/skills/docs-page/SKILL.md | Body is 9 lines; three-column layout implied but scroll-spy and routing behaviour unspecified | -5 |
| 18 | next/src/lib/templates/skills/dashboard/SKILL.md | "1-2 张主图表 (折线 / 柱 / 区域)" — no chart library or rendering approach specified | -2 |
| 19 | next/src/lib/templates/skills/dating-web/SKILL.md | "editorial typography + 克制的高亮色" — no font or accent hex given | -4 |
| 20 | next/src/lib/templates/skills/sprite-animation/SKILL.md | "复古调色板: 红 / 米 / 墨绿" — hex values missing for all three colors | -4 |
| 21 | next/src/lib/templates/skills/mobile-onboarding/SKILL.md | Body is 8 lines; no guidance on phone frame rendering technique or dimensions | -5 |
| 22 | next/src/lib/templates/skills/team-okrs/SKILL.md | Body is 9 lines; progress bar and pill implementation not specified | -5 |

## Cross-Component
- **example_source_url references**: 18 SKILL.md files reference external URLs (`hyperframes.heygen.com/catalog`, `github.com/op7418/guizang-ppt-skill`, `github.com/1weiho/open-slide`, `github.com/heygen-com/hyperframes`, `replit.com/slides`, `github.com/tw93/kami`). These are informational attribution links in `example_source_label` fields, not functional imports — no broken-reference risk.
- **CLAUDE.md → @AGENTS.md**: AGENTS.md exists at repo root and was accessible. Reference is valid.
- **Tier split within skills**: 16 of 75 skill files include `example_id` / `example_name` metadata, creating two quality tiers in a single collection. The 59 skills without examples have shallower test coverage and could diverge from platform behavior invisibly.
- **Custom Next.js version (16.2.6)**: AGENTS.md explicitly warns this is a non-standard build with breaking API changes. Skills that reference platform behavior (e.g., how HTML is rendered into an iframe-like div in `mockup-device-3d`) may silently mismatch if the rendering layer changes without updating the skill files.
- **No orphaned components detected**: All skill names match their directory names. No agent, command, or hook files reference skills that don't exist.

## Recommendation
CLEAR — submit PRs for all bugs and medium/low security fixes.

Priority order:
1. Replace `xlsx@^0.18.5` with a maintained alternative (Medium security).
2. Pin `marked` and `modern-screenshot` to exact versions (Low security).
3. Add `name` and `description` frontmatter to CLAUDE.md (Bug).
4. Expand body content of `web-proto-soft`, `motion-frames`, `hr-onboarding`, `pm-spec`, `social-media-dashboard`, `web-proto-editorial` with output format and concrete design constraints (Quality).
