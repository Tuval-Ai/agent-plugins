---
name: onda-library
description: Use when the user asks about a meeting ONDA recorded - what was decided, who said what, finding a recording, reading a transcript or a summary, or searching across meetings. ONDA stores every meeting as a folder on this machine (or in a synced Drive/iCloud folder), so answer from those files rather than asking the user to paste a transcript.
---

# Reading an ONDA library

ONDA (Tuval AI) records meetings, mostly in Hebrew, and writes each one as a **folder on
disk**. There is no ONDA server and no API to call: everything you need is in the filesystem of
the machine you are running on. Your job is to find the right folder and read it.

Recordings may sit in a Google Drive or iCloud folder that the sync app mirrors locally. From
your side that is still just a local path.

## 1. The rule that comes before reading anything

**A folder can be marked private, and a private folder's contents must not reach you.**

ONDA gives each folder a privacy level, and the strictest one, `stealth` (חשאי), means the
recording does not leave this computer. Answering a question puts whatever you read into a
request to your model provider, which is exactly leaving this computer. So the level is not
advice to pass on to the user - it decides whether you may open the file at all.

**Check before you read any content from a folder.** Read `<folder>/.onda/privacy.json`:

| What you find | Level | May you read the folder |
|---|---|---|
| no file at all | `local` | **Yes.** A folder nobody configured is local, never cloud |
| `{"level": "local"}` | `local` | Yes |
| `{"level": "cloud"}` | `cloud` | Yes |
| `{"level": "office"}` | `office` | Yes |
| `{"level": "stealth"}` | `stealth` | **No** |
| the file will not parse | `stealth` | **No** |
| no `level` key, or a level you do not recognise | `stealth` | **No** |

The last three matter as much as the explicit one. A `privacy.json` that is half-synced from
Drive, damaged, or written by a newer ONDA than the table you know **may** be stealth, so treat
it as stealth. Never resolve that doubt in the direction that sends a meeting outwards.

When you refuse, say so plainly: the recording is marked private and you cannot read it. Do not
quote any part of it, do not describe its contents from a filename, and do not offer a
workaround. The user can open it in ONDA.

## 2. Find the libraries

Never guess paths. ONDA registers every save folder in one file, the same path on macOS and
Windows:

```
~/.onda/fs-roots.json
```

A JSON array of `{ id, name, path, chromeId }`. Use `path`, and skip entries whose path is not an
existing directory (a save folder on an external disk or an unmounted Drive mount is normal).

On macOS the Mac app keeps a second list, and a user can have folders in one and not the other.
Read both and merge by path:

```bash
defaults export com.onda.mac - 2>/dev/null
```

The `workspaces` key holds a JSON string of `{ path, name }` entries.

If `~/.onda/fs-roots.json` does not exist, ONDA is not installed for this user, or has never been
configured. Say so instead of searching the whole disk.

## 3. A recording folder

A recording is a directory named `<yyyy-MM-dd_HHmm>__<title>`, where the stamp is the **local
time the meeting began**, not when it was saved:

```
2026-11-01_0930__ישיבת צוות
2026-11-01_0930__Weekly sync
2026-11-01_0930__פגישה עם הלקוח· מחיר · תנאים
```

Enough of the title rule to recognise and match a folder:

- Each of `\ / : * ? " < > |` in the title becomes **`·` (U+00B7)**, so a middle dot in a folder
  name usually stands for one of those. It is never removed.
- **Spaces inside the title are kept as spaces.** Always quote these paths.
- The title is cut to 60 characters, and invisible formatting characters are dropped.
- A title that cleans to nothing becomes `הקלטה`.
- A name that is taken gets ` (2)`, ` (3)` and so on, so **two folders with the same stamp and
  title are different meetings**, never duplicates of one.

**A library holds folders written under more than one rule.** The first Chrome extension used an
older scheme - hyphens for spaces, a shorter cut - and folders are never renamed just because the
rule changed. So **match on the `yyyy-MM-dd_HHmm__` stamp and treat the rest as opaque.** Do not
reconstruct a title from a folder name and do not assume the two match character for character;
the real title is in `transcript.json`.

Two things that trip up a naive walk, both common:

**Not every directory under a root is a recording, and recordings nest.** Users make their own
folders - a month, a person, a project - and move recordings into them. Match the stamp rather
than assuming depth, and **look at least one level below each root**:

```bash
find "<root>" -maxdepth 2 -type d -name '[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]_[0-9][0-9][0-9][0-9]__*'
```

A directory that does not match is the user's own filing. Do not treat it as a meeting, and do
not report it as broken.

**A `_Transcripts` folder is not a recording.** It is a flat export of the Markdown transcripts,
one `<folder-name>.md` per meeting. It is useful for *locating* a meeting, with limits - see §5.

### What is in the folder

`<dir>` below is the folder's own name.

| File | What it is |
|---|---|
| `recording.mp3` | the audio |
| `recording.original.mp3` | the audio before an edit, kept once. Only if the audio was edited |
| `transcript.json` | the structured transcript. The authoritative file |
| `transcript.js` | the same data wrapped for the offline player. Ignore it |
| `Transcript-<dir>.md` | the transcript as readable Markdown. **Start here** |
| `Summary-<dir>.md` | a generated summary: a short overview, then the full one |
| `Lessons-<dir>.md` | a coaching analysis from a template. Customer calls mostly |
| `Notes-<dir>.md` | the user's personal notes. See §7 |
| `.onda/privacy.json` | the privacy level. **Read this first**, see §1 |

Any other `<name>-<dir>.md` is the output of a user-defined template, worth reading when its name
matches the question.

**Expect folders that have less than this.** In a real library a meaningful share have no
`Transcript-*.md`, and some no `transcript.json` either:

- `recording.mp3` + `transcript.json` + `transcript.js` + `player.html`, no Markdown. An older or
  exported layout. Read the JSON.
- `live-draft.md` alone. A dictation or recording never finalised. Read it, and say it is an
  unfinished draft rather than presenting it as a transcript.
- An empty folder. Nothing to read. Say so.

## 4. Read in this order

**For a question about content, read `Transcript-<dir>.md` first.** It is readable, it names the
speakers, and it costs far fewer tokens than the JSON. If it is missing, fall back to
`transcript.json` and build the text from `segments[]`.

**Read `transcript.json` when you need structure:** exact timings, per-word data, speaker
confidence, which speaker was auto-detected rather than named by a person, calendar attendees, or
the generated summaries.

### The shape of `Transcript-<dir>.md`

YAML frontmatter, a title heading, then one line per turn:

```
---
title: "ישיבת צוות"
date: 2026-11-01T09:30:00+02:00
duration: "00:00:05"
language: he
speakers: ["דנה", "Speaker 2"]
engine: "ivrit-ai/whisper-large-v3-turbo"
diarization: "pyannote"
audio: "recording.mp3"
data: "transcript.json"
---

# ישיבת צוות — 2026-11-01

דנה [00:00] (93%): בוקר טוב לכולם
Speaker 2 [00:02] (91%): נתחיל עם ה-roadmap
```

Each turn is `<speaker label> [<mm:ss>] (<confidence>%): <text>`.

**The two timestamps are not the same format.** `duration` is `HH:MM:SS`. A turn's timestamp is
`MM:SS` with **minutes that are not capped at 59**, so `[65:05]` means one hour five minutes and
five seconds, not sixty-five hours. Treat the first field as minutes.

**To answer who was in a meeting, read the frontmatter alone.** `speakers: [...]` holds the
labels, with `date` and `duration` beside them. That is the first dozen lines, not the whole
transcript. Do not read a 40-minute transcript to list its speakers.

**Do not trust `(100%)` as high confidence.** A transcript written by the Mac or the iPhone may
carry no confidence at all, and `100%` is what gets written when the value is absent. So it means
either certainty or silence and you cannot tell which from the Markdown. Where confidence
matters, read `segments[].confidence` in the JSON, where an absent value is absent. The same goes
for `engine`, `audio` and `language`: a value like `"unknown"`, `"none"` or `auto` may be a
default rather than a fact.

### `transcript.json`, the fields that matter

```jsonc
{
  "version": "1.0",
  "title": "…",            // the real title. The folder name is a cleaned copy
  "createdAt": "…",        // ISO-8601 with timezone
  "language": "he-IL",
  "durationSec": 0,
  "speakers": {            // key -> speaker
    "S1": {
      "label": "דנה",      // the display name. What to call them
      "auto": false,       // true = ONDA guessed it, a person has not confirmed it
      "enrolledId": null,  // set when matched against a known voice
      "source": "manual"   // meet-dom | audio-cluster | enrolled | manual
    }
  },
  "segments": [            // the transcript, in order. This is what you read
    { "id": "…", "speaker": "S1", "startSec": 0, "endSec": 0,
      "confidence": 0, "text": "…", "words": [] }
  ],
  "words": [],             // every word flat, with `speaker` and `segId`.
                           // DERIVED from segments[].words - the same data, not extra data
  "calendar": { "participants": [] },   // attendees from the calendar event, if there was one
  "summaries": { "brief": "…", "full": "…" }
}
```

**`speakers[k].label` is the name, the key is not.** Say "דנה", never "S1".

**`auto: true` means nobody confirmed that name.** Where it matters who said something and the
label is automatic, attribute it with that caveat rather than as fact.

**`words[]` is derived from `segments[].words`.** Reading both doubles the tokens and tells you
nothing new. Read `segments`.

A file may carry keys not listed here. They belong to other parts of ONDA. Ignore what you do not
recognise, and do not report it as corruption.

## 5. Searching across meetings

Grep rather than loading. A library can hold several hundred meetings, and reading them all will
exhaust your context before it answers anything.

```bash
grep -ril "<term>" "<root>"/*/Transcript-*.md "<root>"/*/*/Transcript-*.md 2>/dev/null
```

`-l` matters: it returns filenames, so no content enters your context from a folder you have not
cleared under §1. Then, for each hit, check that folder's privacy level and only then read it.

**A `_Transcripts` export may be used to locate, never to read.** Those files sit outside any
recording folder, so **they carry no privacy level** and you cannot tell a private meeting's
transcript from any other. Use the export only to get candidate folder names:

```bash
grep -rl "<term>" "<root>"/_Transcripts/*.md 2>/dev/null | xargs -I{} basename {} .md
```

Then go to `<root>/<name>/`, check `.onda/privacy.json`, and read the folder's own
`Transcript-<name>.md`. Never read a file out of `_Transcripts` directly, and never quote from
one.

**To find a meeting by date, match the folder name stamp.** Do not open files to find the date.
Remember the nesting: a recording from September may sit inside a `2026-09` folder.

## 6. Hebrew and RTL

The content is mostly Hebrew, and that shows up in the data:

- Text can contain **invisible bidi characters**. Some are deliberate: a Latin word inside a
  Hebrew line is wrapped in isolate marks (U+2066/U+2069) so it displays correctly. Others arrive
  pasted from a calendar or a chat. Either way they break naive string matching, so strip them
  before comparing, and do not treat them as corruption.
- Folder and file names are Hebrew and contain spaces. **Always quote paths.**
- Keep Latin terms, names, product names and company names exactly as written. Do not translate
  them.
- Answer in the language the user asked in. An English question about a Hebrew meeting gets an
  English answer with the quotes left in Hebrew.

## 7. Rules

**Respect the privacy level, every time.** §1 is not a preamble. Check the folder before reading
its content, and treat anything unreadable or unfamiliar as private.

**Read only.** Do not write, rename, move or delete anything in a library. `transcript.json` is a
contract shared by every ONDA surface, and a partial rewrite destroys data: a save has to start
from the file as it was read and overwrite only the keys it owns, which is not something to
attempt from here. If the user asks you to change a transcript, tell them to do it in ONDA.

**Do not copy a library anywhere.** These are recordings of real meetings with named people. Read
what the question needs, answer, and do not aggregate, upload or move them.

**`Notes-<dir>.md` is the user's own writing.** Read it when the question is about their notes.
Do not quote it into anything that leaves this machine.

**Say when you cannot find it.** No library registered, no folder matching the date, no
`Transcript-*.md` in the folder, or a folder you may not read: say which of those it was. Do not
answer about a meeting you did not read.
