# Task brief: reproducible personal setup on fresh Omarchy

Status: implementation brief; coordinator ownership accepted on 2026-10-28.
Two desktop behavior decisions remain open. Runtime implementation evidence:
NOT VERIFIED. This foundation does not implement the requirements below.

## Outcome and inputs

A fresh Omarchy Quattro desktop can be configured through one Setupdots entry
point that invokes the owning repositories in the correct order, reports
incomplete work accurately, and can be rerun after a failure or reboot. The
result restores intentional configuration; it is not a disk image or data backup.

Inputs: the user's Appdots/Hyprdots installation audit request, subsequent
rebuild preferences, and acceptance of a dedicated Setupdots repository. This
brief supersedes the earlier cross-repository plan stored in Hyprdots.

Accepted requirements:

- Default appearance: the pinned `purple-jellyfish` theme, opaque windows,
  zero inner/outer gaps, and no window borders. No particular wallpaper inside
  that theme needs separate persistence.
- Hyprdots installation must install FiraCode system-wide. Hyprdots sync must
  restore the chosen font configuration. The breadth of "everything FiraCode"
  remains an open question below.
- Preserve the ability to flash the ZSA Voyager, preferably from Chromium.
- Provide a clear Agentdots installation entry point and use its diagnostics.
- Keep Neovim versions unpinned; no new application or plugin lock policy is
  requested for this rebuild workflow.
- Do not persist Cloudflare DNS or the default Omarchy agent selection.
- Personal data, browser bookmarks, credentials, and restore payloads remain
  on external storage and are transferred manually. They must not enter any
  public repository or GitHub workflow.
- Keep the main Git configuration generic. Prompt locally for home/work
  identities and generate separate identity files rather than versioning values.
- Correct Appdots' database-refresh-without-upgrade behavior.

Read [Setupdots' ownership ADR](../adr/0001-coordinate-repository-installation.md),
[glossary](../../GLOSSARY.md), and [manifest contract](../../config/README.md).
Theme behavior remains governed by
[Hyprdots' glossary](https://github.com/ChazBeaver/hyprdots/blob/main/GLOSSARY.md),
[community checkout ADR](https://github.com/ChazBeaver/hyprdots/blob/main/docs/adr/0001-community-theme-checkouts.md),
and [theme-sync plan](https://github.com/ChazBeaver/hyprdots/blob/main/docs/plans/theme-sync.md).
Read the owning checkout before implementation. Do not change the real-clone
versus linked-vault distinction or edit the theme lock by hand.

## Scope and authorization

Setupdots owns coordination and aggregate diagnostics. Component changes belong
in the repositories named by the ownership ADR. Cyberdots is outside the initial
selection. No desktop implementation or application configuration moves here.

The user authorized populating Setupdots with recommendations, agent guidance,
and architecture documents for its future implementation. This foundation does
not change the installed machine or claim that the runtime installer is built.
Live OS updates, reboots, firmware flashes, data restoration, commits, pushes,
and publication are not part of the foundation task.

Extend existing mechanisms rather than adding parallel installers. No new
global agent guidance, theme checkout scheme, package-manager framework, or
backup system is required.

## Entry points and execution order

Add `setupdots/install.sh` as the new-machine coordinator, `doctor.sh` for
read-only diagnostics, and `test.sh` for isolated fixtures. There is no routine
Setupdots `sync.sh`. Its doctor calls component doctors directly, never
`home-doctor`, which could recurse through Setupdots' registry entry.

The manifest currently records real entry points, including Agentdots' `sync.sh`.
Switch that row only after its proposed wrapper exists. Missing future scripts
are not successful installation steps.

1. Validate the manifest, Omarchy compatibility, required commands, usable
   session, network, repository locations, and access to pinned theme/plugin
   sources, including the private vault. Display the selected stages, expected
   interactions, and explicit optional skips. Clone missing selected repositories
   under `~/Projects/home`. Reuse valid existing checkouts without resetting
   changes, switching branches, or implicitly pulling a new revision. Validate
   origins and script paths; reject a conflicting directory rather than replace
   it. Handle authentication before backing up or replacing configuration.
2. Perform a complete supported Omarchy update before additional package
   installation. Stop on failure and honor reboot requirements. Allow rerunning
   after reboot; do not reboot automatically. Do not assume `omarchy update -y`
   makes every installed helper noninteractive; the audited restart helper
   still prompted. Recheck the updater contract during implementation.
3. Run Appdots bootstrap, including its owned Git identity setup. Do not repeat
   prompts that bootstrap has already completed.
4. Run Hyprdots bootstrap: desktop packages, pinned themes/plugins, keyboard
   permissions, font convergence, initial appearance, and plugin dependencies.
5. Run Agentdots' proposed `install.sh` once implemented. Its current first
   installation path is `sync.sh` followed by `doctor.sh`.
6. Register Wikinotes and an editable theme-vault checkout when selected.
   Neither is a prerequisite for displaying pinned themes. Access to the
   sources of Hyprdots' pins is still required even without an editable clone.
7. Run selected component doctors and report each stage. For components without
   a doctor, check documented registration postconditions and state that limited
   coverage. A selected optional component is required for that run. Register
   Setupdots itself using the established registry convention; never synthesize
   another repository's registration. Present manual data/bookmark follow-up
   separately from configuration checks.

Keep run records under the user's state directory, normally
`~/.local/state/setupdots`, honoring `XDG_STATE_HOME`. Record stage status,
selection, and checkout revisions without identity answers or credentials.
An aggregate doctor should use the last run's explicit selection; with no such
record, report the limitation and how to supply a selection. Do not infer it by
collecting every repository on the machine.

A run record is not proof that a stage remains correct. Recheck postconditions
and reuse idempotent owning commands. Interrupted work must not leave a success
receipt. Package updates must not run from ordinary sync or doctor. An unattended
run with missing required input must return an actionable incomplete result;
never fabricate answers or silently skip prerequisites.

## Component work

### Agentdots and themes

Agentdots already has `sync.sh`, `doctor.sh`, `backup.sh`, and `test.sh`. Its
sync supports first installation and installs targets even when the associated
agent client is not present. Add only a thin `install.sh` that checks
prerequisites, runs sync, then doctor. Retain sync's validation, backups,
collision handling, and registry behavior.

Hyprdots already installs and checks themes through `themes.sh`, root sync,
and doctor. The vault's `install.sh` only registers the editable checkout. Do
not add a competing theme installer or doctor there. Preserve the existing
source checkout used for private theme pins and the community checkout ADR.

### Desktop appearance lifecycle

Recommended, awaiting the user's answer: seed Jellyfish, opaque mode, and no
gaps/borders during initial bootstrap, preserving subsequent menu/key choices
during ordinary sync. This follows the existing theme-sync selection policy.
Enforcement on every sync would require an explicit change to that policy.

Bootstrap must apply the initial theme even though Omarchy has already seeded
its stock default. Repeated bootstrap must not reset later choices. Record
successful initialization only after its postconditions pass; absence of a
cached receipt alone must not reset an already configured workstation. Expose
an explicit reset-to-defaults operation if needed.

Use the existing opacity and Omarchy gap/border mechanisms so their normal
toggles remain usable. Do not create a second toggle or commit runtime state.

### One font preference, native Omarchy application

Audit findings to recheck during implementation:

- `ttf-firacode-nerd` supplies `FiraCode Nerd Font` system-wide. The Nerd Font
  variant preserves terminal glyphs; the plain FiraCode package was not installed.
- Appdots declares that Linux package and hardcodes the family in shared Ghostty
  configuration. macOS independently declares its Homebrew font package.
- Omarchy's font selection reads fontconfig. Its font setter writes the user's
  monospace preference, updates existing terminal configs, and refreshes the shell.
  A system-wide font package does not itself set every account's font preference.
- The setter edits Ghostty with `sed -i`. Appdots links the whole Ghostty
  directory into its checkout, so changing the family through that setter can
  edit tracked source. Fontconfig alone also cannot reveal app-specific drift.

Proposed design: one small desired-family file in Hyprdots, for example
`config/font.txt` containing `FiraCode Nerd Font`. The existing package manifest
owns package installation; the preference file owns selection. Do not put font
binaries in Git or install a second FiraCode variant. Move Linux package ownership
from Appdots to Hyprdots when the standalone application path is ready; preserve
macOS support.

Use Omarchy's command and fontconfig contract for application. Generated font
settings are installed results, not another desired-family declaration. One
physical config file cannot serve every application's syntax.

Prototype an integration that prevents runtime rewrites of tracked files before
calling the font setter from sync. Candidates are a real local app config merged
from intended settings or supported includes with generated font values. Prove
Ghostty include precedence and Omarchy menu behavior in isolation. Do not accept
dirty source checkouts as a side effect. Record the chosen layout in an accepted
owner ADR after resolving it; these alternatives are still proposals.

Sync repairs font drift and avoids unnecessary shell restarts. If the font
package is missing, return incomplete with the bootstrap repair command rather
than adding privileged package installation to ordinary sync. Preserve per-app
sizes, icons, emoji, and language fallbacks. Do not force proportional app or
website fonts before the scope decision below.

### ZSA Voyager

Normal typing, layouts, layers, and macros use onboard firmware without extra
software. ZSA supports Chromium-based Oryx flashing on Linux with appropriate
udev permissions; Keymapp is optional for that workflow. Sources:
[Voyager FAQ](https://www.zsa.io/voyager/faq/does-the-keyboard-require-software),
[flashing options](https://www.zsa.io/flash), and
[Linux rules](https://github.com/zsa/wally/wiki/Linux-install).

The user's [Oryx layout](https://configure.zsa.io/voyager/layouts/ZlvXJ/latest/2)
did not render through the research tool; no reset-key or firmware-revision
claim is made about that layout.

Add a validated rule for browser/bootloader access to Hyprdots' privileged
bootstrap work and a read-only doctor check. Prefer narrowly scoped access for
the active local user; verify HID and bootloader access separately. Reload rules
and explain when reconnection is required.

Do not copy the current machine's rules blindly. The audit's `udevadm verify`
failed on stray trailing commas, and `TAG+="uaccess"` assignments appeared on
unconditional lines separate from vendor matches. A syntax check alone does
not prove browser access. Never flash firmware as an installation or test step.

### Safe package upgrades

Remove Appdots' standalone `pacman -Sy`. Setupdots coordinates a complete
`omarchy update` before package stages; component package additions must not
independently refresh databases. Stop if the update fails. Standalone Appdots
bootstrap must establish the same prerequisite or explicitly stop with the
supported update instruction. It cannot rely on a hidden assumption that
Setupdots launched it.

If non-Omarchy Arch is an explicitly supported path, use a complete
`pacman -Syu` upgrade/install transaction there. Preserve macOS behavior and
avoid another Omarchy updater. Verify ordering and failure propagation in both
coordinated and standalone paths, including the mechanism used to avoid redundant
updates without trusting a stale success flag. Source:
[Arch package guidance](https://wiki.archlinux.org/index.php?title=Pacman#Installing_packages).

### Generic Git configuration and local identity prompts

Appdots owns the generic main config and a reusable local identity-setup command.
Remove placeholder name/email values, keep conditional home/work includes, and
set `user.useConfigOnly = true` so missing identity prevents inferred commits.

Use the user's requested `~/.gitconfig-home` and `~/.gitconfig-work` names.
Recognize the current dotted filenames on other machines and offer a preserving
migration; do not silently discard or overwrite existing identities. Prompt
for personal name/email and optional work identity using Git's config writer.
Keep answers only in local files outside Git, with no values in installer logs.

Bootstrap invokes this setup; ordinary sync must not prompt or overwrite it.
Reuse valid existing values. An unattended run with missing required input
reports the missing step instead of using placeholders. Doctor checks effective
identity in personal/work/worktree contexts and distinguishes an intentionally
unset optional work identity from a broken active setup. Use synthetic identities
in fixtures. Source: [Git config](https://git-scm.com/docs/git-config).

### External storage and bookmarks

Keep personal payloads and any private restore launcher on external storage.
The launcher may invoke Setupdots but should not copy its implementation.
The public installer may print a manual checklist; it must not enumerate,
upload, log, or commit external data.

Use a bookmark HTML export and the browser's supported import UI. Do not replace
a running browser profile or rewrite its Bookmarks JSON during installation.
Manual data transfer and bookmark import are outside configuration success.
Source: [Chrome bookmark import/export](https://support.google.com/chrome/answer/96816?hl=en-GB).

## Acceptance criteria and verification

| ID | Observable acceptance criterion | Required verification |
| --- | --- | --- |
| C1 | Setupdots validates selection/sources, calls owning installers in order, and retries interruption without resetting existing checkouts. | Manifest rejection and command-trace fixtures; fresh Quattro VM build and rerun. |
| C2 | A complete successful base update precedes package additions; no standalone database refresh remains; standalone Appdots is safe too. | Stubbed success/failure/order checks for both paths; VM package transaction. |
| C3 | Jellyfish, opaque mode, and no gaps/borders initialize according to the chosen lifecycle without premature success receipts. | Initial/retry/existing-machine fixtures; live Hyprland options and theme selection. |
| C4 | Hyprdots installs the Nerd Font system-wide and restores the agreed scope from one intended-family declaration. | Package check, `omarchy font current`, `fc-match`, effective terminal configs, UI observation. |
| C5 | Font convergence preserves app settings, fallbacks, macOS support, and clean checkouts; unchanged sync avoids restarts. | Symlink/merge/include fixtures; repository status before/after; change font through Omarchy, then sync. |
| C6 | Voyager rule syntax and intended browser/device access pass without flashing firmware. | `udevadm verify`, device-match tests, browser chooser after rule installation/reconnection. |
| C7 | Agentdots' thin installer retains sync behavior and propagates prerequisite/sync/doctor failures. | Existing Agentdots fixtures, wrapper ordering/failure fixture, doctor. |
| C8 | Hyprdots remains the theme installer; optional Wikinotes/vault registration works and its limited check coverage is explicit. | Coordinator/registry fixtures, existing theme fixtures and doctors. |
| C9 | Generic Git config uses local identities, safely handles legacy filenames, and accurately reports missing identities. | Synthetic home/work/worktree fixtures; doctor after local setup. |
| C10 | Personal data stays external/manual, never enters Git/logs, and no browser profile is implicitly replaced. | Inspect data boundaries and any synthetic launcher fixture; manual HTML import remains separate. |
| C11 | Selected stages report failures/deferred activation; skips are explicit; doctor never recurses, updates, or installs. | Failure injection and command traces; selected component doctors. |
| C12 | Invocation, ownership, recovery, and limitations match implemented behavior; unimplemented work is not called complete. | Document/implementation review, links, recorded owner checks and VM evidence. |

Run each changed owner's `./test.sh` and `./doctor.sh` after implementation.
Sync only when appropriate for authorized installed-config changes. Hyprland
changes require `hyprctl reload` and `hyprctl configerrors`. Never bootstrap
the production machine merely to test implementation.

## Open decisions

1. **Font scope:** all monospace contexts (recommended), proportional app UI as
   well, or app UI plus website text? Preserve icon/emoji/language fallbacks in
   every case. Awaiting the user's answer.
2. **Appearance lifecycle:** initial defaults preserving later choices
   (recommended), or restore Jellyfish/opaque/no-gaps on every sync? Awaiting
   the user's answer. Font enforcement on every sync is already requested.

The precise font adapter requires an isolated prototype governed by C4/C5.
That engineering choice and the two unanswered preferences do not block work
on manifest validation, safe package handling, or coordinator failure behavior.

## Implementation sequence

1. **Foundation:** establish the ownership ADR, agent instructions, manifest
   contract, and this brief. Runtime requirements remain NOT VERIFIED.
2. **Preflight:** implement parsing, selection, checkout validation, and a
   preview that does not mutate the machine. Add fixtures for malformed paths,
   unexpected origins, missing access, optional skips, and planned ordering.
3. **Owner prerequisites:** fix Appdots package safety and identities, add the
   Agentdots wrapper, and implement the agreed Hyprdots defaults/font/rules.
   Resolve open desktop decisions before their dependent changes.
4. **Execution and recovery:** wire the update, owning installers, interruption
   handling, local run records, direct doctors, and honest final reporting.
   Exercise selected-component failures and retries with stubbed commands.
5. **Fresh-system qualification:** run and rerun the build in a disposable
   Quattro VM. Record the tested Omarchy version and evidence. Verify physical
   keyboard access separately; do not flash it to prove access.

Preview must not claim that package resolution, privileged work, or activation
succeeded without the corresponding checks. Test helpers accept explicit roots;
do not redirect `HOME`, run real package commands, or require a live desktop.

## Completion evidence

| Criteria | Verdict | Evidence |
| --- | --- | --- |
| C1-C12 | NOT VERIFIED | Runtime installer and component changes are planned; the foundation supplies requirements, not a successful rebuild. |

Earlier component tests/doctors are baseline evidence only. The invalid ZSA
rules and font setter's writes through the Ghostty symlink are known reasons
to finish owner changes before relying on a new build. Recheck those findings
against the then-current implementation.

Separate guidance cleanup identified by the audit remains outside this task:
Hyprdots README package and shell-size prose differs from its manifests/config.
Those owning files are canonical. Obtain the user's decision before consolidating
that older guidance; moving this plan does not authorize unrelated cleanup.
