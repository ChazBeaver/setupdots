# ADR-0001: coordinate new-machine installation in Setupdots

Status: accepted. Decision and record date: 2026-10-28.

## Context

The personal repositories each own a distinct part of the workstation and
already provide installation or convergence commands. Rebuilding an Omarchy
machine also requires work across their boundaries: checking access, upgrading
the base system, obtaining repositories, ordering their installers, recovering
from interruption, and deciding whether the complete requested setup succeeded.

Putting that workflow in Hyprdots would make the desktop repository responsible
for application configuration, agent tooling, optional notes, and overall build
state. The user chose a separate repository for that responsibility and created
Setupdots. The name also avoids giving a directory alias the same name as the
system `install` command.

## Decision

Setupdots owns coordination starting from an already installed Omarchy Quattro
system. The initial implementation targets Omarchy only; Appdots' independent
macOS support does not make Setupdots a macOS installer.

| Repository | Owns | Boundary with Setupdots |
| --- | --- | --- |
| **setupdots** | Build selection, cross-repository preflight, base-update stage, checkout acquisition, ordering, retry/resume, and overall diagnostics. | Calls owning commands and reports their results. |
| **appdots** | Application configs, shell, application packages, and local Git identity setup for its supported operating systems. | Keeps standalone bootstrap safe and exposes its own checks. |
| **hyprdots** | Hyprland, Omarchy shell, desktop packages and defaults, Omarchy font intent, desktop hardware integration, and pinned plugins/themes. | Installs its own desktop layer; does not coordinate unrelated repositories. |
| **agentdots** | Global agent instructions, portable skills, target installation, and their diagnostics. | Keeps first installation and convergence in its own commands. |
| **wikinotes** | Notes, snippets, search helpers, and registration of its working checkout. | Optional registration; no notes or personal content copied into Setupdots. |
| **omarchy-theme-vault** | Editable personal theme content and wallpaper assets; registration of its working checkout. | Optional editable checkout. Hyprdots owns installation of pinned theme sources. |
| **cyberdots** | Optional, removable security and situational-awareness tooling. | Reserved for a later explicit opt-in integration, outside the initial selection. |

The table defines ownership. The current commands live in the
[repository manifest](../../config/repositories.tsv); detailed new behavior and
acceptance checks live in the [implementation brief](../plans/new-machine-install.md).

Each component remains independently installable. Setupdots must not be the
only place that makes an unsafe component installer safe. In particular, the
Appdots partial-upgrade correction belongs in Appdots even though Setupdots
also coordinates a complete base update.

The manifest records desired components. `~/.dotfiles-env.sh` records installed
repositories and remains written by their respective installers. Setupdots may
register its own location as `SETUP_DOTS_DIR` and its own `setupdots` directory
alias when its installer exists; it must not synthesize other repositories'
registry entries or derive the desired build by scanning the current machine.

Setupdots will expose a new-machine `install.sh` and a read-only `doctor.sh`.
Routine `home-pull`, `home-sync`, and `home-doctor` maintenance remains separate.
There is no Setupdots `sync.sh` that reruns an OS update or installation.
Its doctor delegates directly to selected component doctors and excludes
Setupdots itself, avoiding recursion through `home-doctor`.

Theme and plugin acquisition stays in Hyprdots. Preserve its real community
theme checkouts, linked vault themes, source pins, and existing lifecycle.
The optional editable vault clone is not a second installed-theme system.

Personal data, credentials, browser state, and restore payloads never become
repository inputs or outputs. An external private launcher may invoke
Setupdots, but restoration remains manual and outside configuration success.

## Alternatives considered

- **Coordinator inside Hyprdots:** convenient initially, but expands desktop
  ownership to unrelated repositories and whole-machine lifecycle.
- **Manual checklist only:** retains separation but leaves dependency ordering,
  failure reporting, and recovery to memory on every rebuild.
- **One combined dotfiles repository:** could centralize installation but would
  discard the existing independent scopes and entry points without a need.
- **Setupdots as a configuration copy or package-list aggregator:** would
  duplicate intent and allow the standalone repositories to drift from the
  coordinated installation.

## Consequences

There is one additional repository to maintain, with a small contract between
it and the owning installers. Component fixes may need coordinated changes,
but configuration and package intent stay in one place. A run must report
incomplete stages honestly and revalidate state when resumed.

The installation is reproducible in intended configuration, not a byte-for-byte
system image. Package and component versions can advance; existing theme/plugin
pins retain their current role. No new Neovim version-lock policy is introduced.

This decision accepts repository ownership and coordinator scope. It does not
settle the breadth of FiraCode selection, the lifetime of appearance defaults,
or the precise font adapter layout. Those remain explicit questions in the
implementation brief. No live installation or successful rebuild is claimed by
this foundation.
