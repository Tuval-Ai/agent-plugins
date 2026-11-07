---
name: iris-archive
description: Use when the user asks about a business conversation Iris (Tuval AI) archived - what a customer asked, what was agreed with a supplier or an employee, when something was said, or searching across customer and staff conversations. Iris keeps the archive on this Mac and ships a read-only command for it, so answer from that command rather than asking the user to paste messages.
---

# Reading an Iris archive

Iris (Tuval AI) is a work archive for a small business. On a Mac it builds the archive from the
copy of the conversations that the official WhatsApp app already keeps on that computer, mostly in
Hebrew. There is no Iris server to call. Everything you need is on the machine you are running on,
behind one command that Iris installs.

## 1. The command

Iris ships a read-only command inside the app. Run it with `python3`:

```bash
python3 "/Applications/Iris.app/Contents/Resources/whatsapp-archive/bin/connector_read.py" conversations
```

If `/Applications/Iris.app` does not exist, try `~/Applications/Iris.app`. If neither exists, Iris
is not installed on this Mac. Say so. Do not search the disk for it.

Use the command and nothing else. The archive is a SQLite file, and the rules about what you may
see live in the command, not in the file: only conversations the business marked as a customer or
an employee, only messages from the WhatsApp app's own copy, payment card numbers, ID numbers and
one-time codes masked, and no phone numbers. Opening the file directly would skip all of that.

## 2. Three subcommands

All three print JSON.

**`conversations`** lists what you may read:

```json
{"conversations": [
  {"id": "3f9a0c1d2e4b", "name": "דנה כהן", "kind": "customer", "last": "2026-11-06T14:02:11+00:00", "messages": 214}
]}
```

`kind` is `customer` or `staff`. `id` is Iris's own identifier for the conversation. It is not a
phone number and it stays the same between calls.

**`search "<text>"`** searches the text of those conversations, newest first, up to 30 results:

```json
{"results": [
  {"conversation": "3f9a0c1d2e4b", "name": "דנה כהן", "date": "2026-11-02T09:15:40+00:00", "from": "דנה כהן", "text": "…"}
]}
```

`text` is a snippet of up to 400 characters. Add `--limit N` for fewer results.

**`read <id>`** reads one conversation, oldest of the batch first, the latest 50 messages by default
(`--limit N`, at most 200):

```json
{"conversation": "3f9a0c1d2e4b", "name": "דנה כהן", "messages": [
  {"date": "2026-11-02T09:15:40+00:00", "from": "דנה כהן", "type": "chat", "media": false, "text": "…"}
]}
```

`from` is `business` for the business's own messages. In a group it is the participant's name, or
`participant` when Iris has no name. `media: true` means a photo, video, voice note, document or
sticker: you get its caption when there is one, never the file.

## 3. Work in this order

1. **Search first.** Most questions name a topic, a product or a person. `search` is cheap and tells
   you which conversation to open.
2. **Then read only what matched.** Use `read` on the conversation the search pointed to. Raise
   `--limit` only when the answer is clearly further back.
3. **Use `conversations` to find someone by name**, or when the user asks what there is.

A business can have hundreds of conversations. Reading them all will exhaust your context before it
answers anything, and it is more than the question needs.

## 4. Errors, and what to say

| `error` | What it means | What to tell the user |
|---|---|---|
| `not_found` | No Iris archive on this Mac | Iris has not set up an archive on this computer |
| `no_app_local_archive` | The archive exists but was not built from the WhatsApp app on this Mac | Iris has not yet imported the conversations from the WhatsApp app here |
| `archive_not_ready` | Iris has never been opened since installing | Open Iris once, then try again |
| `not_allowed` | That conversation is not marked customer or staff, or the id is unknown | It is outside what Iris shares. The business can mark it in Iris |
| `query_too_short`, `query_is_only_wildcards` | The search text was empty or only `%` / `_` | Search for an actual word |

A conversation the user knows exists but `conversations` does not list is almost always unmarked.
Say that, rather than suggesting the archive is incomplete.

## 5. Hebrew and RTL

- Text can contain **bidi control characters** (U+200E/U+200F, U+202A-U+202E, U+2066-U+2069).
  They are invisible. Strip them before comparing names.
- When you quote a line that mixes Hebrew and a Latin term, keep the term as it is written. Do not
  translate names, product names or company names.
- Answer in the language the user asked in. Leave quotes in the language they were written in.
- Dates are UTC. Convert to the user's local time when you state a time of day.

## 6. Rules

**Read only.** The command cannot send, delete or change anything, and neither should you. If the
user asks you to reply to a customer, write the draft in your answer and tell them to send it from
WhatsApp or Iris.

**Masked values stay masked.** `[redacted:card]`, `[redacted:id]` and `[redacted:code]` mark where a
card number, ID number or one-time code was. Do not try to recover it from context.

**Do not copy the archive anywhere.** These are conversations with real customers and employees,
who did not connect you. Read what the question needs, answer, and do not aggregate, export or
upload them.

**Say when you cannot find it.** No Iris, no archive, an unmarked conversation, no search match:
say which. Do not answer about a conversation you did not read.
