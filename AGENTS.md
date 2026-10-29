# Setupdots agent instructions

Setupdots owns new-machine coordination. Read [README.md](README.md) for the
current implementation status and
[ADR-0001](docs/adr/0001-coordinate-repository-installation.md) before changing
repository ownership or installation order. Global instructions remain owned
by Agentdots; this file adds only Setupdots-specific rules.

## Where work belongs

- Extend the owning repository's installer for its packages, configuration,
  backups, registration, and diagnostics. Setupdots invokes those entry points;
  it does not copy their implementation or repair their files directly.
- Read [the implementation brief](docs/plans/new-machine-install.md) before
  implementing the new-machine workflow. Keep accepted requirements separate
  from open decisions and proposed designs.
- Use [GLOSSARY.md](GLOSSARY.md) for domain terms,
  `docs/adr/` for significant architectural decisions, and `docs/plans/` for
  task requirements. Repository-specific procedures belong in `.agents/skills/`
  only when there is an actual reusable procedure to document.
- The [manifest contract](config/README.md) governs `config/repositories.tsv`.
  Record existing entry points until their replacements actually exist. Do not
  duplicate package lists or Hyprdots' theme/plugin source pins here.

## Implementation rules

- Start from an installed Omarchy system. Keep OS installation, personal-data
  restoration, firmware flashing, and routine maintenance outside this
  coordinator's execution path.
- Treat manifests as data. Validate repository names, sources, and script
  paths; use argument arrays. Never source a manifest or execute it with `eval`.
- Reuse existing checkouts without implicit pulls, resets, branch changes, or
  overwrites. Detect unexpected origins and explain how to resolve them.
- Store run records outside Git, under the user's state directory. Revalidate
  their postconditions on retry; a recorded success is not current proof.
- Propagate failures. A missing required component, deferred activation, or
  failed doctor cannot become an overall success. Optional skips must be
  visible, and selecting an optional component makes its installation required
  for that run.
- `doctor.sh` must be read-only. Invoke component doctors directly; never call
  `home-doctor` from it, which could recurse through the installed registry.
- Use Bash with `set -euo pipefail` and `IFS=$'\n\t'` when adding shell scripts.
  Keep the initial implementation small; add shared helpers only for repeated
  behavior that actually exists.

## Verification

This foundation has no executable installer, `test.sh`, or `doctor.sh` yet.
For documentation and manifest changes, check relative links, manifest rows
against the contract and available checkouts, the relative `CLAUDE.md` symlink,
and whitespace. Do not report missing future scripts as passing checks.

When executable behavior is introduced, provide `./test.sh` with isolated
fixtures. Accept explicit checkout, state, and configuration roots in testable
helpers; never repurpose `HOME`. Stub privileged, network, package, and desktop
commands so fixtures cannot change the workstation.

Run the owning repositories' tests for changed behavior and their read-only
doctors when relevant. A real installation belongs in a disposable fresh
Omarchy VM, followed by a rerun and failure/recovery checks. Never run bootstrap,
an OS update, or a firmware flash on the working machine merely to test code.
Record actual evidence against the brief's criterion IDs; use NOT VERIFIED
until the relevant implementation and checks exist.
