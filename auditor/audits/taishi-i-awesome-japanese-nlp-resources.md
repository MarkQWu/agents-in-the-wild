# NLPM Audit: taishi-i/awesome-japanese-nlp-resources
**Date**: 2026-05-17  |  **Artifacts**: 5  |  **Strategy**: single
**NL Score**: 79/100
**Security**: CLEAR
**Bugs**: 0  |  **Quality Issues**: 5  |  **Security Findings**: 2

## NL Score Summary
| File | Type | Score | Top Issue |
|------|------|-------|-----------|
| plugins/awesome-japanese-nlp-resources/skills/search/SKILL.md | Skill | 72 | Missing `name` in frontmatter (−25); no cross-references to sibling skills (R07, −3) |
| plugins/awesome-japanese-nlp-resources/skills/find-new-resources/SKILL.md | Skill | 75 | Missing `name` in frontmatter (−25) |
| plugins/awesome-japanese-nlp-resources/skills/research-issues/SKILL.md | Skill | 75 | Missing `name` in frontmatter (−25) |
| plugins/awesome-japanese-nlp-resources/skills/research-trends/SKILL.md | Skill | 75 | Missing `name` in frontmatter (−25) |
| plugins/awesome-japanese-nlp-resources/.claude-plugin/plugin.json | plugin.json | 100 | No issues |

**Weighted average**: (100 + 75 + 75 + 75 + 72) / 5 = **79/100**

### Per-file scoring detail

**plugin.json** — 100/100
All required fields present (`name`, `description`). No `version` field to evaluate for semver validity. No vague quantifiers. No issues.

**find-new-resources/SKILL.md** — 75/100
- −25: `name` field absent from frontmatter (Skills penalty table, unnamed rule). The directory name `find-new-resources` implicitly registers the skill, but the frontmatter should be self-describing.
- Output format: comprehensive bilingual templates with numbered steps ✓
- Empty input handling: Step 0 covers the empty-arguments case ✓
- Cross-references: links to `search` skill in output template ✓ (R07 not triggered)
- Embedded Python code blocks serve as executable examples ✓ (R06 not triggered)
- Body length: 278 lines, under 400-line threshold ✓
- No R01 vague quantifiers found

**research-issues/SKILL.md** — 75/100
- −25: `name` field absent from frontmatter
- Output format: bilingual templates with 6 structured sections ✓
- Empty input handling: Step 0 handles blank input ✓
- Cross-references: Step 6 references `find-new-resources` skill ✓
- Body length: 279 lines ✓
- No R01 vague quantifiers found

**research-trends/SKILL.md** — 75/100
- −25: `name` field absent from frontmatter
- Output format: bilingual templates with 5 structured sections ✓
- Empty input handling: Step 0 handles blank input ✓
- Cross-references: Step 6 references `find-new-resources` skill ✓
- Body length: 269 lines ✓
- No R01 vague quantifiers found

**search/SKILL.md** — 72/100
- −25: `name` field absent from frontmatter
- −3 (R07): No cross-references to sibling skills (`find-new-resources`, `research-trends`, `research-issues`). The `find-new-resources` empty-result template tells users to run `/awesome-japanese-nlp-resources:search`, but `search` itself never points outward.
- Empty input handling: Step 0 includes explicit stop-and-output block ✓
- Output format: well-defined table + selection guide template ✓
- Body length: 257 lines ✓
- No R01 vague quantifiers found

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
| Scripts (sh/py/js) | 0 |
| MCP configs | 0 |
| package.json | 0 |
| requirements.txt | 0 |
| Embedded Bash/Python in SKILL.md files | 4 (inline, not installed) |

No installed hooks, scripts, MCP servers, or package manifests exist in the plugin. All Bash/Python code lives inside SKILL.md instruction bodies — it is authored text that Claude executes at runtime via the Bash tool, not pre-installed automation.

### Security Findings
| # | Severity | File | Line | Pattern | Description |
|---|----------|------|------|---------|-------------|
| 1 | Medium | plugins/awesome-japanese-nlp-resources/skills/find-new-resources/SKILL.md | 124 | file-write-outside-repo | Embedded Python writes `/tmp/awesome_ja_nlp_existing_urls.txt` — a file outside the project directory. Benign temp-file pattern but persists across invocations and could leak URL data if /tmp is shared. |
| 2 | Low | plugins/awesome-japanese-nlp-resources/skills/find-new-resources/SKILL.md | 58 | env-var-access | `find "${HOME}/.claude/plugins"` accesses the user's home directory and Claude plugin store. Pattern repeated in research-issues (line 42) and research-trends (line 40) and search (line 68). Expected for plugin path resolution; no exfiltration vector. |

## Bugs (PR-worthy)
No bugs found. All five files parse and register correctly. No broken cross-references detected.

## Security Fixes (PR-worthy, Medium/Low only)
| # | File | Issue | Suggested Fix |
|---|------|-------|---------------|
| 1 | skills/find-new-resources/SKILL.md | Temp file `/tmp/awesome_ja_nlp_existing_urls.txt` persists and is world-readable on shared systems | Use `tempfile.mkstemp()` or `tempfile.NamedTemporaryFile(delete=True)` and pass the path to subsequent steps instead of a hardcoded `/tmp/` path |

## Quality Issues (informational)
| # | File | Issue | Penalty |
|---|------|-------|---------|
| 1 | skills/find-new-resources/SKILL.md | Missing `name` field in frontmatter | −25 |
| 2 | skills/research-issues/SKILL.md | Missing `name` field in frontmatter | −25 |
| 3 | skills/research-trends/SKILL.md | Missing `name` field in frontmatter | −25 |
| 4 | skills/search/SKILL.md | Missing `name` field in frontmatter | −25 |
| 5 | skills/search/SKILL.md | No cross-references to sibling skills (R07); search is the logical entry point but never directs users to `find-new-resources`, `research-issues`, or `research-trends` for follow-on workflows | −3 |

## Cross-Component
- **Skill registration**: Claude Code derives skill names from directory paths, so the four missing `name` frontmatter fields do not prevent registration. However, omitting `name` makes each skill file non-self-describing and inconsistent with the NLPM skills convention.
- **Inter-skill references**: `find-new-resources`, `research-issues`, and `research-trends` all reference `find-new-resources` or `search` in their output templates — these references are valid and point to real installed skills.
- **Inbound-only search**: `search` receives references from three siblings but makes no outbound references. Adding a "Related skills" scope note or R07 cross-reference at the top would close the discovery loop for users who arrive at search first.
- **plugin.json**: does not enumerate skills (they are discovered by convention); no stale path references.
- No contradictions or orphaned components detected.

## Recommendation
CLEAR — submit PRs for the Medium security fix (temp-file hardening) and the five quality issues. No critical or high security findings; the plugin is safe to contribute to.

Priority order:
1. Add `name:` to all four `SKILL.md` frontmatter blocks (single-line fix, four files).
2. Harden the `/tmp/` write in `find-new-resources` with `tempfile` (security fix).
3. Add a scope note to `search/SKILL.md` pointing to the three research/discovery sibling skills.
