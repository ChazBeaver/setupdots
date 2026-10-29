# Repository manifest contract

[`repositories.tsv`](repositories.tsv) is the initial build selection, not an
installed-machine inventory. It records current, verified entry points. No
installer consumes it yet; implement and test this contract before using it
to run commands.

## Fields

Rows contain exactly five tab-separated fields. Blank lines and lines starting
with `#` are ignored. Row order is installation order; there is no second
numeric ordering field.

| Field | Meaning |
| --- | --- |
| `repository` | Unique checkout name under `~/Projects/home`. |
| `source` | Credential-free Git source used for a missing checkout. |
| `selection` | `required` or `optional`; optional components require an explicit choice. |
| `install` | Executable script path relative to that checkout. |
| `doctor` | Read-only executable script path, or `-` if no standalone doctor exists. |

Repository names must match `[a-z][a-z0-9-]*`. Script paths must begin `./`,
contain only ordinary relative path components, and resolve inside the checkout
to executable files. Reject empty components, `.`/`..` components after the
leading `./`, shell syntax, embedded arguments, absolute paths, and symlinks
that escape the checkout. A script path is one argument, never a shell command.

Initially allow GitHub SSH sources of the form
`git@github.com:OWNER/REPOSITORY.git` and HTTPS sources of the form
`https://github.com/OWNER/REPOSITORY.git`, with owner/repository names limited to
letters, digits, dots, underscores, and hyphens. Reject embedded credentials,
query strings, fragments, control characters, and other source forms. Changes
to this small allowlist should have a demonstrated requirement and tests.
SSH and HTTPS forms of the same owner/repository are equivalent when checking
an existing origin.

Reject duplicate repositories, unknown selection values, malformed rows,
missing required commands, and self-inclusion of `setupdots`. Validate the
whole selection before executing any component. Do not infer an unlisted
repository from the installed registry or recursively run an aggregate doctor.

## Execution and verification

Run each script with its checkout as the working directory. An exit status of
zero is necessary for success, but final success also requires the selected
component's doctor and documented postconditions. Stop dependent stages when
a prerequisite fails. Do not treat interactive cancellation as completion.

An optional component may be skipped before the run. Once selected, its
failure makes that run incomplete. Display explicit skips in the final report.
For `doctor = -`, verify only the documented postcondition that Setupdots can
establish, such as checkout and registration. Report this limited coverage;
do not claim that the repository passed a doctor it does not have.

The manifest does not lock component revisions. New clones use the remote's
default branch. Existing valid checkouts are reused at their current revisions,
without implicit updates or resets. Record revisions locally for troubleshooting.
Hyprdots continues to own its theme/plugin pins; an installed OS/package snapshot
is not promised by this design.

Authentication is a preflight prerequisite, including access to private sources
referenced by the owning installers. Never put keys or tokens in the manifest.
The SSH URLs below match the current repository origins; this does not prove
that a fresh machine already has access.

## Current entry points

- Appdots and Hyprdots use `bootstrap.sh` for first installation.
- Agentdots currently uses `sync.sh` plus `doctor.sh`. Change its manifest
  entry to `install.sh` only after the thin wrapper in the implementation brief
  exists and passes its checks.
- Wikinotes and the editable theme vault use `install.sh` for registration.
  Neither has a standalone root doctor today. The editable vault checkout is
  optional; Hyprdots still requires access to sources used by its pinned themes.
- Cyberdots is outside the initial build selection. Adding optional security
  tooling later requires its own integration requirements; it must not silently
  become part of the default setup.
