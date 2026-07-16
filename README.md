# cowork-stack

The plugin bundle behind [protoAgent](https://github.com/protoLabsAI/protoAgent)'s
**Cowork archetype** (ADR 0083) — pick "Cowork" in the setup wizard or
new-agent picker and this is what gets installed. For knowledge workers
graduating from Claude Cowork: the same file-first workflow, self-hosted,
with cron-grade scheduling, unified memory, and model choice.

## Composition

| Member | Ships | Why it's here |
|---|---|---|
| `cowork` | [cowork-plugin](https://github.com/protoLabsAI/cowork-plugin) | Document skills (docx/xlsx/pptx/pdf) + schedule/memory/writing-voice habits + `/setup-cowork` |
| `claude_bridge` | [claude-bridge-plugin](https://github.com/protoLabsAI/claude-bridge-plugin) | Explore + import your existing Claude Code/Cowork state |
| `google` | [google-plugin](https://github.com/protoLabsAI/google-plugin) | Gmail / Calendar / Drive — the connector layer |
| `artifact` | builtin | The deliverable surface |
| `notes` | builtin | Notes surface |
| `execute_code` | builtin | The runtime the document skills drive |

No config defaults are seeded on purpose: folder access is consented during
`/setup-cowork` (the same opt-in folder model Cowork uses), and Google OAuth
is a console flow.

## Install

Pick **Cowork** in the protoAgent setup wizard / new-agent picker, or:

```
python -m server plugin install https://github.com/protoLabsAI/cowork-stack
```

After install: enable the suggested list, run install-deps for the document
libraries, then run `/setup-cowork`.

## Pin lifecycle (ADR 0049)

Member refs are release tags; `plugins.lock` records resolved SHAs;
`verified_against` is the core version this pin set was last verified on.
`scripts/check_bundle_updates.py` + the verify workflow keep pins moving only
through passing verification.
