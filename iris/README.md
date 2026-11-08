# Iris for AI agents

Lets an AI agent answer questions about a business's customer and staff conversations, from the
archive [Iris](https://iris-app.tuval-ai.co.il) keeps on your own Mac.

Ask "what did Dana ask about the quote last week?" and the agent searches the archive, opens the
conversation and answers. Nothing is uploaded, and no account is created.

## Iris works with official WhatsApp channels

- **A business number registered with Meta.** Iris connects to it through Meta's official tools, with the
  permissions granted in the owner's Meta account. Customer messages to the business number arrive in the
  same archive and work inbox, the business replies from the number inside Iris, and Iris can answer
  automatically from knowledge the owner approved, with a person able to take over at any time. Replies
  follow Meta's rules: text, within 24 hours of the customer's last message.
- **The official WhatsApp app on a Mac.** Iris can build its archive from the copy of the conversations
  that the app already keeps on that computer.

**This plugin reads the archive on the Mac**, which comes from the second channel. Conversations on the
business number registered with Meta are kept in Iris's cloud archive, and this plugin does not read them.

## What you need before it works

1. **Iris on a Mac, with its archive built from the WhatsApp app.** Iris is a work archive for small
   businesses by Tuval AI, mostly used in Hebrew. On a Mac it builds its archive from the copy of the
   conversations that the official WhatsApp app already keeps on that computer. It is distributed on
   request at `iris@tuval-ai.co.il`, not from a public download page.

   This needs **Iris 0.2.19 or later**, with **WhatsApp on the Mac** (וואטסאפ במק) switched on in Iris's settings.
   **Iris 0.2.20 or later is recommended**, including the latest local-copy cleanup fix.
   It is off until the business turns it on, because turning it on asks macOS for access to another
   app's data. The first import runs in the background and can take a quarter of an hour on a large
   history.
2. **The agent has to run on that Mac.** This reads a local archive, so it works in a terminal agent,
   a desktop agent or an IDE agent on the same computer. It cannot work in a browser-only chat, which
   has no access to your disk.
3. **macOS only, for now.** Windows is not supported by this plugin.

## What it does

The plugin is one skill. It teaches the agent to use a read-only command that Iris installs inside
the app (`connector_read.py`), so the agent can:

- **List the conversations it may read**: those the business marked in Iris as a customer or an
  employee
- **Search across them** for a topic, a request or a name
- **Read one conversation**, with the business's own messages told apart from the other side's
- **Answer in your language**, keeping Hebrew quotes in Hebrew

## What it does not do

- **It does not see unmarked conversations.** A conversation nobody marked as a customer or an
  employee is outside what Iris shares, and the command refuses to open it.
- **It does not send, delete or change anything.** There is no command for it. Replies are drafted in
  the agent's answer, and a person sends them.
- **It does not see phone numbers.** Every conversation has an identifier of Iris's own, and a contact
  saved only under a number appears as `unnamed contact`.
- **It does not see files.** Photos, voice notes and documents appear as a type and a caption.
- **It does not reach a server.** There is no Iris server involved. If the archive is not on the
  machine, the agent has nothing to read and says so.

## Privacy

Everything the plugin does is a local read through Iris's own command:

- **Network endpoints called: none.**
- **Credentials needed: none.** No account, no sign-in, no API key.
- **Telemetry: none.** Nothing is sent to Tuval AI.

The rules about what the agent may see are enforced in that command, not left to the agent: only
conversations marked customer or staff, only messages from the WhatsApp app's own copy, and payment
card numbers, ID numbers and one-time codes masked before any text leaves it. Every request is
recorded in the business's own Iris log on the Mac, with the action and the number of results.

What does leave your machine is whatever the agent itself sends to its own model provider in the
course of answering you, as with any other file you ask an agent to read. These are conversations
with real customers and employees, so the skill tells the agent to read only what the question needs
and never to copy or aggregate the archive.

- [Privacy policy](https://iris-app.tuval-ai.co.il/privacy-en.html) (English). The Hebrew original is at
  [privacy.html](https://iris-app.tuval-ai.co.il/privacy.html).
- [Terms of use](https://iris-app.tuval-ai.co.il/terms-en.html) (English). The Hebrew original is at
  [terms.html](https://iris-app.tuval-ai.co.il/terms.html).

Iris is a product of Tuval AI. It is not affiliated with, endorsed by or sponsored by WhatsApp or
Meta.

## Install

### Claude Code

In Claude Code 2.1.275 or later, run this in the agent's conversation:

```text
/plugin install tuval-iris --marketplace Tuval-Ai/agent-plugins
```

Or, in Claude Code 2.1.292 or later, run the equivalent command in a terminal:

```sh
claude plugin install tuval-iris --marketplace Tuval-Ai/agent-plugins
```

This installs from Tuval AI's own source marketplace. Approval and publication in Anthropic's
official directory are a separate process and are not needed for this source installation.

### Antigravity

Install just the Iris folder from a local copy of the public repository:

```sh
git clone https://github.com/Tuval-Ai/agent-plugins.git
agy plugin install ./agent-plugins/iris
```

This direct installation does not mean the plugin has been approved for Google's official
Marketplace. The partner submission is a separate review.

### Grok Build

The [official catalog submission](https://github.com/xai-org/plugin-marketplace/pull/1942) is awaiting
review. Once it is approved and published, look for Iris in Grok Build's plugin directory.

The plugin contains one skill and no executables, no hooks and no network calls. Iris must already
be installed and its Mac archive enabled as described above, but the plugin itself needs no account
or configuration.

## Support

`iris@tuval-ai.co.il` - Tuval AI. Please say which agent you are using and what you asked.

## Licence

MIT. See [LICENSE](LICENSE) beside this file. The same licence at the repository root covers every
plugin in it.

This folder contains only the agent plugin: one skill, written in Markdown, and a logo. The Iris
application, including the command the skill calls, is not open source and no application code is
published here.
