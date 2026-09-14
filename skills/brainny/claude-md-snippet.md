# CLAUDE.md snippet — enables ambient/automatic brainny

Installing the skill files (`skills/brainny/*.md` → `~/.claude/skills/`)
only makes the **manual** slash commands work (`/brainny-catch`,
`/brainny-catch-this`, `/brainny-synthesize`, `/brainny-extract`, ...).
None of the **automatic** behavior — onboarding on first use, catching
ideas at the end of a turn, checking sync drift, proposing opportunities,
recalling relevant past ideas — happens on its own until Claude Code is
actually told to trigger those skills without being asked. That trigger
lives in your global `~/.claude/CLAUDE.md`, not in this repo, because
Claude Code only reads instructions from there (or a project's own
`CLAUDE.md`) at the start of a session — nothing in a skill file or a
pip install can make itself ambient on its own.

**To enable it: copy the block below into `~/.claude/CLAUDE.md`**
(create the file if it doesn't exist yet). Do this once, on each machine
where you want ambient capture. Without this step, brainny still works
fully by hand — you just have to type the slash commands yourself.

```markdown
# brAInny ambient capture

At the start of every session, in every project, silently check that the
`brainny` CLI is available (e.g. `brainny --version`). If it is:
1. Run the `brainny-onboarding` skill once (not a loop, not every
   session — it self-guards via `onboarding-done` in
   `~/.brainny/config.json` so it only ever actually asks anything on
   the very first session, on a fresh install, before a central folder
   exists). Full behavior in `~/.claude/skills/brainny-onboarding/SKILL.md`.
2. **Append** the `brainny-catch` skill to the end of every task/turn —
   once whatever was actually asked is fully done, before handing control
   back — so techniques, precautions, solutions, and insights worth
   keeping get captured without being asked, and without ever
   interrupting active work. Never schedule this as a recurring
   background loop/timer (`Skill({skill: "loop", ...})`) — a loop fires
   as a scheduled wakeup that can land mid-task and take over that turn,
   with the original task not reliably resuming. Skip the pass entirely
   for a turn too trivial to plausibly contain anything catch-worthy.
   Full behavior in `~/.claude/skills/brainny-catch/SKILL.md`.
3. Separately, run the `brainny-sync-check` skill once (not on a loop —
   just this one time, now, at session start) to check whether this
   project's ideas have drifted from its configured central folder
   (asking permission before `brainny sync`), and whether the central
   folder itself — if GitHub-backed — is overdue for a push (asking
   permission before `brainny sync --push`). Full behavior in
   `~/.claude/skills/brainny-sync-check/SKILL.md`.
4. Also once per session, run the `brainny-propose-check` skill: the
   ambient sibling of `/brainny-synthesize`, so a project's already-
   captured ideas get one automatic look per session for genuine
   combinations worth proposing as an Opportunity. Most sessions should
   propose nothing, and that's correct. Full behavior in
   `~/.claude/skills/brainny-propose-check/SKILL.md`.
5. Right after the user's first real message in the session, run the
   `brainny-recall` skill once: pull a few concrete keywords out of what
   they just said, run `brainny recall <terms>` (searches the current
   project *and* every project synced into the central folder), and
   mention it briefly if something genuinely relevant turns up. Full
   behavior in `~/.claude/skills/brainny-recall/SKILL.md`.

None of the above replace the user directly telling you what to keep.
When the user types `/brainny-catch-this <description>`, run the
`brainny-catch-this` skill instead: searches the whole session for what
the description points at, always responds with what got captured. Full
behavior in `~/.claude/skills/brainny-catch-this/SKILL.md`.

When the user types `/brainny-catch-skill <description>` — pointing at a
reusable *procedure*, not just a fact — run the `brainny-catch-skill`
skill instead: same user-directed contract, but captures `kind: "skill"`
and attaches real evidence (code/plot/table) via `brainny attach`. Full
behavior in `~/.claude/skills/brainny-catch-skill/SKILL.md`.

When the user types `/brainny` — a full end-of-session review, not a
quick catch — run the `brainny` skill: the heavier, two-pass version of
`/brainny-catch`, meant to run once, deliberately, at session end. Full
behavior in `~/.claude/skills/brainny/SKILL.md`.

When the user types `/brainny-synthesize` — asking whether already-
captured ideas combine into something bigger — run the
`brainny-synthesize` skill: reviews the project's real captured ideas for
genuine combinations, proposing each with an honest confidence weight via
`brainny propose`. Most passes should propose nothing. Full behavior in
`~/.claude/skills/brainny-synthesize/SKILL.md`.

When the user types `/brainny-extract <path>` — pointing at existing
repo(s), not a live session — run the `brainny-extract` skill: mines
ideas out of code someone already wrote (README/CHANGELOG/commit history
first, never exhaustive per-file reads), then `brainny capture`s per repo
whatever genuinely clears the bar. Expensive and on-demand only — never
runs on its own. Full behavior in `~/.claude/skills/brainny-extract/SKILL.md`.

If `brainny` isn't installed in this environment, skip all of these
silently — don't mention it, don't nag, don't offer to install it unless
asked.

**Guardrails, non-negotiable:**
- Stay silent when a catch cycle, sync-check, or propose-check finds
  nothing to report — that's the common, correct outcome, never announce
  "checked, nothing found."
- The onboarding offer speaks up at most once, ever, per machine.
- Recall and propose-check each run at most once per session; sync-check
  runs at most once per session too.
- Never take a destructive or user-visible action beyond writing to the
  *current* project's own local `brainny-out/` — no `git push`, no
  messages sent, nothing that needs approval. `brainny sync --push` is a
  separate, explicit, user-invoked command.
- Catch runs at the end of a completed task, never mid-task — it must
  never be scheduled as a separate interrupting wakeup.

Full skill instructions: `~/.claude/skills/brainny-onboarding/SKILL.md`,
`~/.claude/skills/brainny-catch/SKILL.md`,
`~/.claude/skills/brainny-sync-check/SKILL.md`,
`~/.claude/skills/brainny-catch-this/SKILL.md`,
`~/.claude/skills/brainny-catch-skill/SKILL.md`,
`~/.claude/skills/brainny-recall/SKILL.md`,
`~/.claude/skills/brainny/SKILL.md`,
`~/.claude/skills/brainny-synthesize/SKILL.md`,
`~/.claude/skills/brainny-propose-check/SKILL.md`, and
`~/.claude/skills/brainny-extract/SKILL.md` (installed from, and kept in
sync with, the brainny repo's own `skills/brainny/*.md`). The tool
itself: see its `OPERATIONS.md` for the full picture.
```

That's the whole thing — one paste, one time. `brainny install-skills`
(see the main README's Install section) copies the skill files into
`~/.claude/skills/` for you, but it does **not** touch
`~/.claude/CLAUDE.md` on its own — that file is yours, hand-edited
global config, and brainny will never write to it without you doing the
paste yourself.
