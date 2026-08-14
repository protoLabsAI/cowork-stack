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

### Pin-bump PR lifecycle (explicit-approval model, [#2645][issue-2645])

The scheduled `bump` job pushes to a single, stable branch — `bump-pins` — and keeps **at
most one** open pin-bump PR at a time. A later scheduled run that finds more bumps
force-pushes that same branch, updating the PR in place instead of piling up duplicates.
Treat `bump-pins` as bot-owned: it's rewritten wholesale every run, so hand edits to it
don't survive the next bump.

GitHub does **not** auto-start a `pull_request` workflow run for a PR opened with the
repository `GITHUB_TOKEN` — it's held `action_required` until someone with write access
clicks **Approve and run workflows** on the Actions tab (recursion-prevention; see
[GitHub's docs][gh-token-docs]). This repo has no GitHub App installation or PAT
provisioned to avoid that, so it deliberately runs the **explicit-approval model** instead:

- **Approving is a documented, one-click operator responsibility, not a bug.** Watch the
  repo's Actions tab (or PR notifications) for the pin-bump PR and approve its run so
  `verify` actually runs before merge.
- **The `bump` job makes a stall visible instead of silent.** After pushing, it polls
  (bounded wait) for the `verify` run it should have queued. If that run comes back
  `action_required` — or never shows up at all, which is worse — the job **fails**,
  comments on the PR, and adds a `needs-approval` label. An unapproved pin-bump PR then
  shows up as a red weekly schedule, not a PR quietly rotting for weeks.
- **ADR 0049's invariant still holds either way:** `verify` still has to pass before merge
  — this only makes sure someone notices it needs to be *started*.

[issue-2645]: https://github.com/protoLabsAI/protoAgent/issues/2645
[gh-token-docs]: https://docs.github.com/en/actions/concepts/security/github_token#when-github_token-triggers-workflow-runs
