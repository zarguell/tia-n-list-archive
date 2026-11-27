# Tia N. List — cold store

Aged-out stories from [Tia N. List](https://zarguell.github.io/tia-n-list/),
moved here by the retention policy so the published site stops growing without
losing the record.

**Data only.** Nothing here is built, rendered, served or published. There is
no workflow, no dependency, and no `index.html`. If you are looking for the
site, it is [zarguell/tia-n-list](https://github.com/zarguell/tia-n-list).

## Why this exists

The published site keeps a story page for 3 years if the story was ever
`hottest` (`peak_score >= 7.5`) or cited by an immutable artifact (a CTI case
and the STIX snapshot built from it), 3 years if it was digest-cited or ever
`hot`, and 6 months otherwise. That policy is written down in the main repo's
`retention.md`. When a horizon runs out, the *data* comes here and the *page*
leaves the site — the alternative would be a site that grows without limit or
an archive that quietly deletes its own history.

## Layout

```
manifest.jsonl                  one line per archived story (append-only)
stories/<id>.json               the story record as it left the hot store
events/<event-id>.json          only events no live story references
events/<event-id>.md
analysis/<id>.md                only when the story brought one
```

`manifest.jsonl` is the index of record. One JSON object per line:

```json
{"ts": "2026-11-27T04:10:00Z",
 "id": "<story-id>",
 "tier": "C",
 "events": ["<event-id>", "..."],
 "files": [{"path": "stories/<story-id>.json", "sha256": "..."},
           {"path": "events/<event-id>.md",   "sha256": "..."}]}
```

`sha256` is per file, so drift is detectable without trusting the checkout.

## Written by

`engine/archive.py` in the main repo. It copies every file, appends the
manifest line, and only then unlinks the originals — an interrupted run leaves
a complete archive and an untouched store, and re-running is a no-op because
the manifest already names the id. It refuses to move anything a live page
still depends on: redirect targets, digest-cited stories, stories with an
analysis, and stories sharing an event with a story that stays.

```bash
# from the tia-n-list checkout
python3 engine/archive.py --archive-root ~/repos/tia-n-list-archive            # dry run
python3 engine/archive.py --archive-root ~/repos/tia-n-list-archive --verify   # integrity
```

## Ground rules

- **Append-only.** Never rewrite or delete a manifest line; a correction is a
  new line plus a note in the commit message.
- **No hand edits.** Files are byte copies of the hot store at the moment of
  archiving; editing one breaks its recorded `sha256`.
- **Not a backup.** This is the cold tier of one store, not a copy of the live
  repo. The live store is the source of truth for everything still published.
- Every byte here was public before it arrived: the story and event records are
  tracked in the main repo, which is public. This repo is public for the same
  reason — it adds no exposure.
