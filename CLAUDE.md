# research-partner

A running, cross-linked research journal on companies, markets, and ideas —
fed by articles you surface, Granola notes from team calls, and distilled
conversations you have with Claude. The goal is compounding context: what
gets written down today should make tomorrow's research faster, because
it's already connected to what came before.

## Layout

```
kb/
  notes/<slug>.md   one file per entity (company, market, or idea)
bin/kb              query CLI (stdlib only, no dependencies, no network)
```

Flat and un-typed at the folder level — every entity lives in `kb/notes/`
regardless of whether it's a company, a market, or an idea. The distinction
lives in a `type:` field in frontmatter, not in separate folders, because most
research doesn't respect that boundary anyway (an article about a market
thesis usually names two companies in the same breath). One field to set
beats three folders to choose between.

## The entity file contract

```markdown
---
name: Airform
type: company
aliases: [Airform Inc]
---

## 2026-09-08 [article] Why heat pumps still need installers
**Source:** https://example.com/heat-pump-installers
**Your take:** This is the installer-cost problem [[Airform]] is solving
head-on — connects to what the [[HVAC electrification]] piece said about
adoption bottlenecks not being the hardware.
**Summary:** <Claude's distillation of the article's actual content>

## 2026-08-30 [granola] Call with Airform (Josh, CEO)
**Source:** Granola — 2026-08-30, participants: Josh, You
**Summary:** <distilled from the transcript — direct quotes kept verbatim
where they matter>

## 2026-08-12 [conversation] Thinking through the counter-positioning argument
**Summary:** <distilled from that chat>
```

Rules for this shape:

- **Frontmatter:** `name` (required), `type` (`company` | `market` | `idea`),
  `aliases` (optional inline list — other names/spellings this entity gets
  referred to by, so lookups and mention-matching don't miss it), `watch`
  (optional boolean, `true`/`false` — marks an entity for the proactive
  research routine described below; absent means not watched).
- **The filename must exactly match `name`** — spaces, `+`, whatever's
  actually in the name — not a slugified version. Obsidian resolves
  `[[wikilinks]]` by filename, not by the `name` field, so a mismatch (e.g.
  `ai-physical-assays-for-drug-discovery.md` for a note titled "AI +
  Physical Assays for Drug Discovery") makes every link to it silently fail
  in Obsidian: it creates a new blank note at the vault root instead of
  linking to the real one.
- **Entries are reverse-chronological** (newest at the top) — re-opening a
  file should surface current thinking first, not the oldest context.
- **Every entry heading is exactly** `## YYYY-MM-DD [type] Title` where type
  is `article`, `granola`, `conversation`, or `watch` (the last one is written
  only by the automated proactive-watch routine described below, never by
  Claude in a live conversation). This is what `bin/kb` parses to count
  entries and find the latest date — don't drift from it.
- **Cross-link with `[[Entity Name]]`** wherever an entry mentions another
  company/market/idea. This is Obsidian's wikilink syntax — plain text
  without Obsidian installed, a clickable graph edge with it.
- **If a mentioned entity doesn't have a file yet, create a stub**: just the
  frontmatter plus a one-line note on where the mention came from (e.g. `_Stub
  — created from a mention in [[Airform]]. No dedicated research yet._`).
  Never fabricate content for a stub.
- **An article or conversation usually touches more than one entity.** Put
  the full entry in whichever entity is clearly primary (usually obvious from
  why you flagged it), and cross-link the others — don't duplicate the same
  entry across multiple files. Duplicated content drifts out of sync the
  moment one copy gets updated and the other doesn't.

## Evolving the frontmatter contract

`name`, `type`, `aliases` are deliberately minimal, not exhaustive. New fields
earn their way in from actual use, not speculative design:

- **Claude proposes a field** when the same kind of fact keeps recurring
  across entries/entities in a way nothing today captures — e.g. a status
  distinction, a conviction marker. Always flagged with the field name, its
  shape (scalar vs. list, valid values), and why — never added silently.
- **You can propose fields too**, any time, no justification needed.
- Either way, once adopted: update this file's contract, and update `bin/kb`
  too if the field should be listable/searchable/filterable.
- **Never backfilled by guessing.** A new field gets added to old entities
  only where the value is actually known — not inferred to make old data
  look as complete as new data.

## Capture behavior — how material actually gets in

**Articles.** You give a URL, plus your own reasoning for why it's
interesting (often referencing other entities already in the KB). Claude
fetches the URL and writes an `[article]` entry under the primary entity:
the source URL, your reasoning **kept close to verbatim** (it's your own
thinking, not something to paraphrase away), and a distillation of the
article itself. Any other entities the article or your reasoning mentions
get created/linked too.

**Granola calls.** You explicitly name which call to pull — this is never
automatic or scheduled. Claude pulls the transcript and writes a `[granola]`
entry. Context matters less here since the transcript carries the substance,
but your own framing gets captured if you give any.

**Conversations.** At the end of a conversation that substantively covered a
company/market/idea, Claude should offer to save it — *"want me to save this
to the KB?"* — rather than writing silently. It should only write after you
confirm, and confirm which entity it belongs to if that's not obvious. This
only works inside a Claude Code session with this repo in reach; there's no
ambient capture of conversations happening anywhere else.

**Proactive watch (scheduled, every couple of days).** A local scheduled
task — not a cloud routine, since it needs direct access to this KB on
disk — runs periodically and does fresh research (web search) on every
entity in `kb/notes/` whose frontmatter has `watch: true`. This is the one
exception to "conversations only get captured after you confirm": it runs
unattended, on its own schedule, without you in the loop turn-by-turn.
**This isn't created automatically just by cloning this repo** — ask your
Claude Code session to set it up (see README.md).

- **Only entities you (or Claude, with your sign-off) explicitly mark
  `watch: true`** get this treatment — never inferred from how much
  attention a topic happens to be getting. Claude can propose adding the
  flag to a note the same way it proposes new frontmatter fields generally
  (say why, wait for a yes), but never sets it silently.
- **No hard limit on how many entities can be watched, but keep it to
  ~3-5 at a time.** Every run does fresh research on *every* watched entity
  in one unattended pass — more topics means shallower coverage per topic
  and more chances of a marginal finding triggering a notification. `bin/kb
  stats` / `bin/kb list --watched` print a soft warning past 5.
- Each run: for every watched entity, search for genuinely new developments
  since the last run (new companies in the space, new funding, new articles/
  analysis) — not a re-summary of what's already in the note.
- **Findings reach you as a push notification**, not a silent KB write —
  that's the whole point of asking for it, so you don't have to remember to
  go check. The note itself can still get a new stub/article entry so the
  finding has a permanent home, but the notification is the primary delivery
  mechanism, not the write.
- Runs only while your machine and Claude Code are available (it's a local
  scheduled task, not a 24/7 cloud routine) — a missed cycle just means the
  next one covers a longer window.
- New findings get written as a `[watch]` entry (see above) — source(s) it
  found, a distillation, no fabricated "your take" since you weren't in the
  loop for this one. File edits are left uncommitted for you to review
  (`git diff`/`git status`) — the routine doesn't commit on its own behalf.

## Rules for anyone (human or agent) extending this

1. **Never guess a value.** If your reasoning or the source material doesn't
   say something, leave it out rather than inferring it.
2. **Your own reasoning stays verbatim.** It's your thinking, not raw
   material to summarize — that's the part that doesn't exist anywhere else.
3. **Always keep provenance.** Every entry names its source (a URL, a Granola
   call + date, or just the fact that it's a distilled conversation).

## Querying

```bash
bin/kb list                       # every entity, type, entry count, latest date
bin/kb list --watched              # only entities flagged watch: true
bin/kb search moat "heat pump"    # keyword search across all notes
bin/kb search --type market defensibility   # restrict to one type
bin/kb show Airform                # one entity's full note
bin/kb stats                       # coverage: stubs, entry counts by source type
bin/kb check                       # validate wikilinks resolve + filenames match name:
```

Run `bin/kb check` after adding any `[[wikilink]]` to a new entity — it catches both ways this
silently breaks (a `[[Link]]` with no matching file, and a `name:` that doesn't match its filename)
before Obsidian quietly creates a blank stub note at the vault root for either case.

`grep -ri "term" kb/notes/` works fine too, and often better. The corpus is
text on purpose.
