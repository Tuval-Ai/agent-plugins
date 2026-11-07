# Tuval AI agent plugins

One plugin per product, one folder each, for the AI agent marketplaces: Grok Build, Claude's
directory, Antigravity.

| Product | Folder | What it does |
|---|---|---|
| [ONDA](https://onda.tuval-ai.co.il) | [`onda/`](onda/) | Read meetings ONDA recorded, from the recording folders on your own machine |

**Nothing in this repository is product code.** It holds skills, manifests and documentation that
tell an agent how to work with a Tuval AI product. The products themselves are closed source and
live elsewhere.

---

# Adding a product

## 1. Where it goes

One folder at the repository root, named after the product in lowercase, matching the plugin
`name`. Inside:

```
<product>/
  .grok-plugin/plugin.json      required - the manifest
  README.md                     required - this is the documentation URL the catalogs ask for
  skills/<skill-name>/SKILL.md  at least one skill, or an MCP server, or both
  .mcp.json                     only if the plugin ships an MCP server
  assets/logo.svg               the catalog listing's logo
```

`.claude-plugin/plugin.json` is accepted in place of `.grok-plugin/plugin.json` for the Claude
ecosystem. A plugin that targets both can carry both.

## 2. Every folder stands alone

This is the rule the rest of the file exists to protect.

A catalog entry points at **this repository at one commit**, and scopes the plugin to one folder
with `path`. So a user installing product B gets a clone of the whole tree at B's pinned commit,
and the agent then loads only `B/`.

Two consequences:

**No shared files between products.** No `shared/`, no `common/`, no skill in one folder that
reads a file in another, no relative path that climbs out of the folder. If two products need
the same text, each keeps its own copy. A shared file is read at whatever commit that product
happens to be pinned to, which is not necessarily the commit its owner last tested.

**A reviewer judges the folder, not the repository.** Grok's review asks "what does this plugin
actually ship". Everything the plugin provides has to be visible inside its own folder, and
nothing outside it should be needed to understand it.

## 3. Pin each entry on its own

A catalog entry in xAI's `.grok-plugin/marketplace.json` looks like this. Every field matters:

```json
{
  "name": "onda",
  "description": "…",
  "category": "productivity",
  "source": {
    "source": "url",
    "url": "https://github.com/<org>/agent-plugins.git",
    "sha": "0000000000000000000000000000000000000000",
    "path": "onda"
  },
  "homepage": "https://onda.tuval-ai.co.il",
  "keywords": ["onda", "onda recording"],
  "domains": ["onda.tuval-ai.co.il"]
}
```

`sha` is a **full 40-character lowercase commit SHA**, per entry. A branch, a tag or an
abbreviated SHA is rejected by their validator, because a moving reference would let a later
force-push ship new code to everyone silently.

**Each entry carries its own `sha`, so releasing one product does not move another.** Update
`onda` and its entry's SHA changes while every other entry keeps pointing at the commit it was
reviewed at. This is what makes one repository safe for many products.

It also means entries drift apart over time, which is fine and is exactly why §2 forbids shared
files. (For reference, `mongodb/agent-skills` serves two entries this way and currently keeps
both on the same SHA. Either is valid. Independent pinning is what we do, so one product's
release never waits on another's.)

Get the SHA to pin:

```bash
git ls-remote https://github.com/<org>/agent-plugins.git HEAD
```

**To release an update, change one entry's `sha` and nothing else.** Never open a second entry
for a product that already has one.

## 4. Keywords and domains must not collide

These two fields drive Grok Build's plugin **CTA**, the prompt that proactively offers a plugin.
Generic terms are rejected on submission, because they would fire the CTA on unrelated requests.

- **Brand-scope every term.** `onda`, `onda transcript`, `onda meeting`. Not `meeting`,
  `transcript`, `recording`, `audio`, `notes`.
- **Only domains the product actually owns.** No third-party domains.
- **No term may appear under two products in this repository.** Before adding a product, grep the
  existing manifests. Two of our own plugins competing for the same prompt is worse than either
  of them missing it, because the user cannot tell which one answered.

Where two products genuinely overlap, qualify both sides rather than letting one win: not `onda`
and `tuval`, but `onda meeting` and `<product> <its own noun>`.

## 5. What never enters this repository

Grok's review is a static security audit and it rejects on exactly these. They are also simply
true for a public repository:

- **No secrets.** No tokens, no API keys, no `.env`, no credentials, no private endpoints.
- **No product code.** Skills and manifests only. The ONDA applications are not open source.
- **Nothing that executes on install.** No `postinstall`, no `curl | bash`, no downloading and
  running a binary, no `eval` of remote content.
- **Nothing that reads secrets and sends them anywhere.** A skill that touches `~/.ssh`, `.env`
  or environment variables and reaches the network is rejected.
- **No obfuscation.** No base64 blobs, no minified bundles without source.
- **No over-broad hooks.** A lifecycle hook that runs a shell on every write, with no matcher, is
  rejected. Scope it.
- **No instructions aimed at the installing agent** inside a `SKILL.md` or a description. That
  reads as prompt injection.

Declare in the product's `README.md` every network endpoint its plugin calls and every credential
it needs. If the answer is none, say so - it is the strongest thing a reviewer can read.

## 6. Validate before opening a PR

The catalog lives in xAI's repository, not this one, so validation runs there. Clone it, vendor or
point at your plugin, and run the two scripts their CI runs:

```bash
git clone https://github.com/xai-org/plugin-marketplace.git
cd plugin-marketplace
python3 scripts/generate-plugin-index.py
python3 scripts/validate-catalog.py
python3 scripts/generate-plugin-index.py --check
```

All three must pass, and the last one must exit 0. It is the staleness gate: it fails if
`marketplace.json` was edited without regenerating `plugin-index.json`.

**Edit `marketplace.json` as text, by hand.** Do not load and re-dump it with a JSON library: a
round-trip reformats the entire catalog and turns a 12-line addition into a diff of hundreds of
lines, which reads as a botched merge and will be sent back. Their style keeps `keywords` and
`domains` on one line.

For Claude and Antigravity, validate with their own tools before submitting:

```bash
claude plugin validate ./<product>
agy plugin validate --strict ./<product>     # must be 0 errors and 0 warnings
```

## 7. Submission routes, as of 7 November 2026

| Catalog | How | Needs this repository public |
|---|---|---|
| Grok Build | PR to [`xai-org/plugin-marketplace`](https://github.com/xai-org/plugin-marketplace) adding one entry | Yes, for a `url` source. A `local` source vendors the files into their repository instead and needs no repository of ours |
| Claude | [`claude.ai/directory/manage`](https://claude.ai/directory/manage), **Plugin bundle**, with the product folder as the plugin path | Yes, before the listing goes live. It may stay private during review |
| Antigravity | Partner intake form, after `agy plugin validate --strict` passes | Not documented |

Claude reads a plugin from a GitHub repository and takes a **plugin path**, so one product folder
here is one Claude submission. Source the plugin from the official organisation: a branded plugin
coming from a personal account reads as possible impersonation and will be questioned.

## Licence

[MIT](LICENSE). Every product's `plugin.json` states `"license": "MIT"`, and a new product's
manifest must state it too - the catalogs check for a stated licence, by human review rather than
in CI.

MIT does not grant any right to the Tuval AI name or to a product name. The files may be used,
copied and modified freely; the brands may not.
