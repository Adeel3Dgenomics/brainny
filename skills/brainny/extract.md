# brainny — extract skill (mine ideas out of existing repos)

Every other capture skill works from a *live* session — watching a
conversation as it happens. This one works backward instead: given one
or more already-existing repositories (code someone already wrote,
possibly years ago, with no AI session attached to it at all), read
through them cold and mine out the techniques, precautions, solutions,
skills, and insights already sitting there — unwritten anywhere else,
same as everything else brainny catches, just discovered retroactively
instead of in the moment.

You run when the user types `/brainny-extract <path>` (optionally with a
focus hint, e.g. `/brainny-extract ~/code/gwas-pipeline focus on the QC
steps`), where `<path>` is either:
- a single local repo, or
- a directory containing several repos (e.g. `~/Documents/mygithub/`) —
  process every subdirectory that's actually a git repo (has its own
  `.git`), skipping anything that isn't one (data folders, loose files,
  spreadsheets) rather than trying to mine non-code directories, unless
  the user names specific ones.

A bare GitHub URL instead of a local path means it isn't cloned yet —
ask whether to clone it (`git clone <url>`) before proceeding, rather
than doing it unprompted; this skill's default territory is repos the
user already has locally.

This is a **direct, explicit, user-invoked** request, same contract as
`/brainny-catch-this` and `/brainny-synthesize` — always respond with
what you found (or that you found nothing and why), never silently.

## This is expensive — the user knows, budget accordingly

Reading real repositories costs real tokens and time, and the user
explicitly acknowledged that cost when asking for this. That's not
license to read every file top to bottom, though — it's the opposite
reason to be surgical:

- Don't do exhaustive per-file reads. Start with the highest-density
  places knowledge actually gets written down on purpose:
  README/CONTRIBUTING/docs, a CHANGELOG, `OPERATIONS.md`/`SEED.md`-style
  design docs if the repo has one, then commit history for signal.
- `git log --oneline | grep -iE 'fix|workaround|revert|gotcha|bug'` (or
  the platform-appropriate equivalent) surfaces exactly the commits most
  likely to encode a precaution someone actually paid for.
- `Grep` across source for comment markers that flag hard-won knowledge
  on purpose: `NOTE:`, `WARNING:`, `HACK:`, `IMPORTANT:`, `workaround`,
  `gotcha`, `must not`, `careful`. These are far higher-signal than
  reading arbitrary implementation code start to finish.
- Sample a handful of the most-substantial or most-changed source files
  (`git log --stat` churn, or just the largest/most-central modules)
  rather than the whole tree. A few well-chosen files beat exhaustive
  coverage for this purpose — the goal is real signal, not completeness.
- For a directory of many repos, this budget applies **per repo** — don't
  let repo count multiply effort linearly without checking in; if asked
  to process a large number of repos, it's fine to say so and confirm
  scope before diving into all of them.

## What counts as "unique and informative" (the GATE, adapted for cold reading)

Same discipline as every other capture skill — default answer is **NO**,
most files and even most whole repos should yield **zero** entries. The
adaptation for reading finished code instead of a live conversation:

- A **technique**: a genuinely clever or non-obvious implementation
  approach visible in the actual logic — not "uses a for loop," but a
  specific, transferable way of solving something (a particular retry/
  backoff shape, an unusual-but-effective data structure choice, a real
  performance trick).
- A **precaution**: something the code or its history reveals was a real
  gotcha — a defensive check whose existence implies a past bug, a
  comment explaining an ordering requirement, a commit message that says
  what broke and why.
- A **solution**: a specific, concrete fix to a hard problem — a
  workaround for a library's own bug, a working answer to a fiddly
  integration.
- An **insight**: a design rationale spelled out in a comment, docstring,
  or README that reveals *why*, not just what.
- A **skill**: a whole reusable procedure or project structure (a
  pipeline shape, a specific analysis's steps) — attach real evidence
  (a script, a small config, a small table) via `brainny attach` the same
  way `/brainny-catch-skill` does, so it can actually be followed again,
  not just read about.

Reject anything that's just "this project uses library X" or restates
what a docstring already says without adding judgment — boilerplate and
common patterns aren't unique just because you found them by reading
code instead of watching a session. The bar is the same as everywhere
else: reusable beyond this one repo, AND non-obvious enough that
re-deriving it would cost real effort.

## Origin

Unlike a live session, you generally can't tell whether a human or an AI
wrote any given piece of code just by reading it. Leave `origin` unset
(unclassified) by default. Only set it when there's real, specific
signal — an explicit AI-generated-code disclaimer, a commit message or
`git blame` authorship pattern you can actually point to — never guess
from code style alone.

## Output, per repo

For each repo that yields anything, write the same `EntryInput` JSON
array shape every other capture skill uses (`brainny/schema.py`) and
shell out from a directory where `brainny` can see it:
```
brainny capture <path-to-entries.json> --project <repo-name> --session extract-<yyyy-mm-dd>
```
Use the repo's own directory name as `--project` unless the user names
something else. For a multi-repo pass, do this once per repo that
actually yielded something — skip the call entirely for repos that
yielded nothing, same as any other empty pass. Each call mirrors to the
configured central folder automatically, same as every other capture —
that's the actual point of this skill: feeding repos that never had a
live AI session into the same shared idea network everything else lands
in, so `brainny central`/the Network tab pick them up like any other
project.

Report back per repo: which ones yielded entries (title + kind for each)
and which ones yielded nothing (briefly — a list is fine, this doesn't
need individual explanations for a whole batch of nothing). This is a
direct request with a real cost behind it, so silence on the whole
operation would read as broken — but "checked repo X, found nothing" for
one repo among many is fine to fold into a one-line summary rather than
narrated individually.

## What this must never do

- Never run ambiently or on a schedule — this only ever runs because the
  user explicitly typed `/brainny-extract` right now, given how
  expensive it is.
- Never invent an idea that isn't actually grounded in something real in
  the repo (a specific file, comment, or commit) — no filling in
  plausible-sounding techniques just because a repo "probably" has them.
- Never capture just to have something to report for a repo that
  genuinely has nothing noteworthy — an empty result for a given repo is
  correct, not a miss.
- Never guess `origin` from code style or naming conventions alone.
- Never `git push`, modify, or commit anything in the repo being read —
  this is read-only mining; the only writes are brainny's own
  `brainny capture` calls into `brainny-out/`.
- Never touch a repo the user didn't point you at, even if it's sitting
  right next to one they did (e.g. in the same parent directory) —
  process exactly what was named.
