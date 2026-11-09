# court - OpenCode skill (fork / adaptation)

Put your idea on trial in front of a jury of 12 independent AI reviewers. A prosecutor argues it fails, a defense argues it works, they rebut each other, 12 jurors vote independently, and a judge reads the verdict + conditions for acquittal. Never touches your files except `.court/` in the folder you run it in. MIT.

**Upstream original:** [alexyc9381/court-skill](https://github.com/alexyc9381/court-skill) by Alex Chen [@nocodealex](https://instagram.com/nocodealex). This repo is an adaptation for OpenCode (model-agnostic jury wording, `.opencode/skills/court` layout with `scripts/` + `references/`). Logic, prompts, `court.py` bookkeeping, and MIT license unchanged.

## What changed vs upstream
- Jury renamed from vendor-specific to "independent AI reviewers" - works on any model (OpenCode, Claude Code, etc.), no false provenance.
- OpenCode-native layout: `.opencode/skills/court/SKILL.md` + `scripts/court.py` + `references/jurors.json`, plus `court.py`/`jurors.json` beside `SKILL.md` for relative-path compat.
- SKILL.md `The tool` section updated to OpenCode base-directory + `skill` tool (`id="court"`), not Claude-specific env vars.
- Compat copies kept at `.agents/skills/court/` and `.claude/skills/court/` (OpenCode auto-discovers these too).

## Install

**Global OpenCode (all projects):**
```powershell
Copy-Item -Recurse .opencode\skills\court "$env:USERPROFILE\.config\opencode\skills\court"
```

**Project-local (one repo):**
```powershell
Copy-Item -Recurse .opencode\skills .\.opencode\skills -Force
```
Or add to `opencode.jsonc`:
```jsonc
{ "skills": ["./.opencode/skills"] }
```

**Compatibility locations** (also auto-discovered by OpenCode): `.agents/skills/court/`, `.claude/skills/court/` / `~/.agents/skills/court/` / `~/.claude/skills/court/`.

## Use

In OpenCode, load / invoke the skill:
```
/court Quit my job to sell candles on Etsy full time. I have $8,000 saved and 40 sales so far.
```
Or mention it: `@court Quit my job...`

| flag | meaning |
| --- | --- |
| `--jury N` | 3 to 12 jurors. Default 12. |
| `--quick` | 6 jurors. |
| `--seed S` | Fixes which jurors sit on a smaller jury. |

A full trial is 17 subagent calls (2 openings, 2 rebuttals, 12 jurors, 1 judge); `--quick` is 11. Every subagent counts toward usage, so start with `--quick` if near limit.

Outputs `.court/<run>/VERDICT.md`: vote (e.g. GUILTY, 9 to 3), every juror's one-line reason, and what to change before you try again.

## Test
```powershell
python .opencode\skills\court\court.py plan
python .opencode\skills\court\court.py plan --quick
```

## Credits / disclaimer

- Original: Alex Chen (@nocodealex), MIT - see `LICENSE`. Keep `VERDICT.md` credit line.
- Adaptation: JOEL D'SOUZA / Crouton Alpha for OpenCode use.
- Not made by Anthropic. The jury is made of language models, so treat the verdict as a structured second opinion, not legal, financial or professional advice.
