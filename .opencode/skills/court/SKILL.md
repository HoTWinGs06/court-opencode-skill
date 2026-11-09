---
name: court
description: >-
  Put an idea on trial in front of a jury of 12 independent AI reviewers. A prosecutor reviewer
  argues it fails, a defense reviewer argues it works, they rebut each other,
  12 juror reviewers with different backgrounds vote on their own, and a judge
  reviewer reads the verdict and the conditions that would flip it. Use when the
  user wants honest pushback instead of agreement, gets too much agreement, asks "is this a good idea", wants an idea, plan, pitch or claim
  stress-tested, or says "court" or "put it on trial".
license: MIT
---

# court

For when you get agreement with everything you say. Instead of one polite answer, the idea goes on trial:
a prosecutor builds the case against it, a defense builds the case for it, and a jury of independent reviewers who
never see each other votes. You are the clerk of the court. You never argue, never vote, and never
soften the verdict.

What the user typed after `/court`: `$ARGUMENTS`

If that is blank, or still reads like a placeholder, take the case from the conversation: the idea,
plan or claim the user most recently wanted an opinion on.

> Upstream: alexyc9381/court-skill by Alex Chen (@nocodealex). OpenCode-native adaptation by JOEL D'SOUZA: jury renamed to independent AI reviewers (model-agnostic), kept court.py logic unchanged, restructured to `.opencode/skills/court/` for OpenCode discovery.

## The tool

Every piece of bookkeeping goes through `court.py` in this skill's base directory (provided when you load this skill via the `skill` tool with `id="court"`):

```bash
# <skill-dir> is the base directory printed after loading the skill, e.g. ~/.config/opencode/skills/court
python3 "<skill-dir>/court.py" <command>
# on Windows, python3 may be just: python
python "<skill-dir>/court.py" <command>
```

Supporting files beside this `SKILL.md`:
- `court.py` - bookkeeping (also in `scripts/court.py` for OpenCode layout)
- `jurors.json` - 12 juror definitions (also in `references/jurors.json`)

Below, `COURT` means exactly that command. If the path looks unexpanded, use the "Base directory for this skill" from the skill load. The state lives in `.court/<run>/state.json` in the current directory, and every command after `init` finds it through `.court/LATEST`.

## Step 1: size it

Read the flags out of the request. Everything that is not a flag is the case.

| flag | meaning |
| --- | --- |
| `--jury N` | N jurors, 3 to 12. Default 12. |
| `--quick` | 6 jurors. |
| `--seed S` | Fixes which jurors sit on a smaller jury. Default: random, and recorded. |

Run `COURT plan` with the same flags and tell the user in one line how big the trial is, for example
"12 jurors, 17 sub-agent calls", then start. If this skill fired on its own (the user never asked for
a trial), ask once first and offer `--quick`.

Sub-agents write into `.court/` in the current directory. In OpenCode default permissions this may prompt per file; suggest approving or using an auto-approve permission setup for the trial. Do not change the user's settings yourself.

## Step 2: write the case file

**Sub-agents cannot see this conversation.** The prosecutor, the defense, every juror and the judge
know only what is in the case file, so write `.court/case.md` to stand on its own:

- First line: the case in one sentence, in the user's own words where you can ("Quit my job to sell
  candles on Etsy full time").
- Then everything the user told you that matters: numbers, audience, budget, timeline, what they have
  already tried, constraints.
- Do not add facts the user never gave, and do not write your own opinion of the idea into it. A case
  file that leans one way decides the trial before it starts.

## Step 3: open the trial

```bash
COURT init --case-file .court/case.md [--jury N | --quick] [--seed S]
```

## Step 4: the trial

Always drive it with `COURT next`. It reads the files on disk and tells you the next phase and the
exact command. The phases run in this order: `opening`, `rebuttal`, `jury`, `judge`.

Every phase works the same way:

1. `COURT prompts <phase>` writes one brief per job and lists the jobs still to run, in waves.
2. Launch **one wave at a time**: a single message with one subagent call per job in that wave.
   - OpenCode: use the `task` subagent tool with a general agent, `description="court <job id>"`
   - Claude Code compat: Agent (Task) tool, `subagent_type=general-purpose`

   Each call prompt is:
   - `Read <brief path> and follow it exactly. It is your whole brief.`

   Wait until every subagent in the wave has replied before you launch the next wave.
3. After the last wave, run `COURT next`. If an output is missing it sends you back to the same phase,
   and `prompts` then lists only the missing jobs. Re-run those once. If a juror fails twice, write
   the single line `NO VOTE` into its file (`COURT check jury` lists it) and move on. Never write a
   vote, a statement or a judgment yourself.

## Step 5: the verdict

When `COURT next` says the trial is over:

```bash
COURT render
```

It counts the votes, writes `VERDICT.md` into the run folder and prints it. Show the user the verdict
line, the vote table and the judge's sections exactly as written. Then add at most two lines of your
own: the single condition for acquittal you would start with, and the path to the full trial.

Rules for you, the clerk:

- Never change, soften or re-count the verdict. A guilty verdict is useful information, not bad news
  to cushion.
- Never add praise the jury did not give.
- If the user disagrees with the verdict, offer a retrial with the changes from "Conditions for
  acquittal" written into a new case file. Do not argue for either side yourself.

---

*Adapted from [/court](https://github.com/alexyc9381/court-skill) by Alex Chen (@nocodealex). MIT. The jury is made of language models, so treat the verdict as a structured second opinion, not legal, financial or professional advice. Not made by Anthropic. OpenCode skill by JOEL D'SOUZA / Crouton Alpha.*
