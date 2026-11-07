# ONDA for AI agents

Lets an AI agent answer questions about meetings [ONDA](https://onda.tuval-ai.co.il) recorded, by
reading the recording folders on your own machine.

Ask "what did we decide in the meeting with Dani last Tuesday?" and the agent finds the folder,
reads the transcript and answers. Nothing is uploaded, and no account is created.

## What you need before it works

1. **ONDA installed, with at least one recording.** ONDA is a meeting recorder and transcriber by
   Tuval AI, for macOS and Windows, used mainly for Hebrew. It is distributed on request at
   `onda@tuval-ai.co.il`, not from a public download page.
2. **The agent has to run on the machine that holds the library.** This reads local files, so it
   works wherever the agent can see your filesystem: a terminal agent, a desktop agent, or an IDE
   agent. It cannot work in a browser-only chat, which has no access to your disk.

If a recording folder sits in a Google Drive or iCloud folder that the sync app mirrors locally,
that counts as local and works the same way.

## What it does

ONDA writes every meeting as a folder on disk: the audio, a structured `transcript.json`, and a
readable `Transcript-*.md`. This plugin teaches the agent that layout, so it can:

- **Find a meeting** by date, by title, or by who was in it
- **Search across the whole library** for a topic, a decision or a name
- **Read a transcript** and attribute lines to the right speaker by name
- **Read a generated summary** or your own notes on a meeting
- **Answer in your language**, keeping Hebrew quotes in Hebrew

It finds your libraries the way ONDA itself does, from `~/.onda/fs-roots.json` and the Mac app's
own list of save folders, so it sees exactly the folders ONDA shows you and nothing else.

## What it does not do

- **It does not record, transcribe or edit.** Those happen in ONDA. If you ask the agent to change
  a transcript it will tell you to do it in ONDA, because `transcript.json` is shared by four ONDA
  apps and a partial rewrite loses data.
- **It does not play audio**, and it does not read `recording.mp3`. It works from the text.
- **It does not reach a server.** There is no ONDA server. If the files are not on the machine, the
  agent has nothing to read and will say so.

## Privacy

Everything this plugin does is a local file read. It sends nothing to Tuval AI, it has no
telemetry, and it collects nothing. There is no account and no sign-in.

**A recording you marked private stays private.** ONDA gives every meeting folder a privacy
level, and the strictest one, חשאי (stealth), means the recording does not leave your computer.
Answering a question puts what the agent read into a request to its model provider, which is
leaving your computer - so this plugin checks the level before it opens anything, and refuses a
private folder outright. It does not quote it, summarise it, or describe it from its name.

The check fails safe. A privacy marker that is damaged, half-synced from Drive, or written by a
newer ONDA than the plugin knows **may** be private, so it is treated as private. A folder you
never configured is local, which is readable - ONDA never treats an unset folder as cloud.

What does leave your machine is whatever the agent itself sends to its own model provider in the
course of answering you, exactly as with any other file you ask an agent to read. These are
recordings of real meetings with named people, so the skill tells the agent to read only what the
question needs and never to copy or aggregate a library.

Your personal notes (`Notes-*.md`) are treated as private: read when you ask about them, and not
quoted into anything that leaves the machine.

- [Privacy policy](https://onda.tuval-ai.co.il/privacy)
- [Terms of use](https://onda.tuval-ai.co.il/terms)

## Install

Install it from your agent's plugin directory, or point your agent at this folder directly. The
plugin contains one skill and no executables, no hooks and no network calls, so there is nothing
to configure after installing.

## Support

`onda@tuval-ai.co.il` - Tuval AI. Please say which agent you are using and what you asked.

## Licence

<!-- TODO (Harel's decision, not set yet): every marketplace requires a stated licence.
     MIT is the usual choice for a plugin and is what the examples in each directory use.
     Add a LICENSE file with the chosen licence and name it here before submitting. -->

This folder contains only the agent plugin: one skill, written in Markdown. The ONDA applications
themselves are not open source and no application code is published here.
