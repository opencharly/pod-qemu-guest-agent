# AGENTS.md — pod-qemu-guest-agent

Standalone candy repo for the `qemu-guest-agent` candy — the QEMU guest agent with
the full host↔guest RPC surface and an application-consistent `fsfreeze` snapshot
hook. The entire candy lives in `charly.yml` at the repo root. There is no source
tree — the hook dispatcher and config are authored inline in the plan.

Canonical files:

- `charly.yml` — the `qemu-guest-agent:` candy entity (description, `package`,
  `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:qemu-guest-agent` — the owning skill: the full RPC surface, the
  `fsfreeze` hook, and the virtio-serial channel contribution. Load before
  editing, building, deploying, or troubleshooting this candy.
- `/charly-vm:vm` — VM lifecycle and the QEMU-user-net caveat for the agent.
- `/charly-internals:libvirt-renderer` — the renderer that injects this candy's
  channel snippet into `<devices>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `write:` / `check:`, service declarations,
  `libvirt.snippets:`).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert the `/usr/bin/qemu-ga` binary, the
  installed package, the `fsfreeze` dispatcher and its drop-in dir, a clean
  `freeze` exit, and the config's hook and no-blocked-RPC content.

## Modify this repo

- Edit the `qemu-guest-agent:` candy entity in `charly.yml`; the `skill:` entity
  in the same file is the owning skill's source — a candy change and its skill
  change land together.
- The `fsfreeze-hook` dispatcher and `/etc/qemu/qemu-ga.conf` are authored inline;
  keep their paths and content in step with the `check:` steps.
- The guest-side virtio-serial channel is declared on the VM entity, not here —
  do not add a channel to this candy.
- The `skill:` entity is the source for `/charly-distros:qemu-guest-agent`; never
  edit the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
