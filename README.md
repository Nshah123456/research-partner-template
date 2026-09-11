# research-partner

A running, cross-linked research journal on companies, markets, and ideas.
See `CLAUDE.md` for the full contract: the entity file format, how capture
works for articles/Granola/conversations, and the rules for how the
frontmatter schema is meant to evolve.

## Getting started

1. Point a Claude Code session at this folder — `CLAUDE.md` loads
   automatically and tells it how to behave.
2. Give it a URL and your own reasoning for why it's interesting, or tell it
   which Granola call to pull, or ask it to save a conversation. It writes
   entities into `kb/notes/` as you go — there's nothing to set up first.
3. Optional: open this folder as an Obsidian vault to see the `[[wikilinks]]`
   between entities as a live graph. Nothing here depends on Obsidian —
   it's just plain markdown either way.
4. Optional — proactive research: flag an entity `watch: true` in its
   frontmatter, then ask your Claude Code session to set up a local
   recurring scheduled task (every couple of days is a reasonable default)
   that checks watched entities for genuinely new developments and pushes
   you a notification when it finds something. This has to be created
   per-machine/per-person — it doesn't come with the repo, since it's tied
   to your own Claude Code install, not to these files. See CLAUDE.md's
   "Proactive watch" section for exactly how it's meant to behave once set
   up. Keep the watch list small (~3-5) — `bin/kb stats` will nudge you if
   it grows past that.

## Quick start

```bash
bin/kb list                       # every entity, type, entry count, latest date
bin/kb search moat "heat pump"    # keyword search across all notes
bin/kb search --type market defensibility   # restrict to one type
bin/kb show Airform                 # one entity's full note
bin/kb stats                       # coverage: stubs, entry counts by source type
bin/kb check                       # validate wikilinks resolve + filenames match name:
```

`grep -ri "term" kb/notes/` works fine too, and often better. No
dependencies, no API keys, no network — `kb/notes/` is plain markdown on
disk.
