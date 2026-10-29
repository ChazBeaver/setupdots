# Setupdots

Setupdots coordinates a personal setup on a freshly installed Omarchy Quattro
system. It brings together the repositories under `~/Projects/home`, runs their
own installers in dependency order, and checks whether the requested setup is
complete.

**Current status: architecture and implementation plan.** This repository does
not yet contain an executable installer or doctor. The files here establish
their scope and contract; they do not claim a fresh-machine installation has
been tested.

## Start here

- [Architecture decision](docs/adr/0001-coordinate-repository-installation.md):
  why Setupdots exists and what each repository owns.
- [Implementation brief](docs/plans/new-machine-install.md): requirements,
  installation order, outstanding decisions, and acceptance checks.
- [Repository manifest](config/repositories.tsv) and its
  [contract](config/README.md): the initial component selection and existing
  entry points.
- [Agent instructions](AGENTS.md): how to work in this repository.
- [Glossary](GLOSSARY.md): the small shared vocabulary used by the plan.

## Intended workflow

Start with Omarchy already installed and a working user session. Clone
Setupdots, review the selected components, and run its future `install.sh`.
The installer will check access and prerequisites, complete the supported base
update, invoke the owning repositories, and finish with an overall diagnostic.
It must support rerunning after an interrupted stage or a required reboot.

The initial selection is Appdots, Hyprdots, and Agentdots. Wikinotes and an
editable theme-vault checkout are optional. Hyprdots already installs its pinned
themes without requiring a separate editable vault checkout; access to the
pinned sources is still required.

Setupdots will own `install.sh` for new-machine coordination and `doctor.sh`
for read-only verification. It will not add a routine `sync.sh` that repeats
installation. Existing repository sync commands and the `home-*` maintenance
commands remain the normal maintenance workflow.

Application configuration, package lists, desktop settings, themes, and agent
skills stay in their owning repositories. The manifest describes the desired
build; `~/.dotfiles-env.sh` records repositories actually installed on a
machine. Each repository remains responsible for registering itself there.

## Private data

Personal files, credentials, browser profiles, and bookmarks stay on external
storage and are restored manually. Setupdots may print a reminder, but it does
not discover, copy, export, or upload those payloads. Any private launcher kept
on the external device should invoke this repository rather than carry a second
implementation.

The [implementation brief](docs/plans/new-machine-install.md) records Jellyfish,
FiraCode, keyboard flashing access, safe package upgrades, and local Git
identities. It also identifies the two preferences still to settle before
implementing their dependent desktop behavior.
