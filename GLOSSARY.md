# Glossary

**Component**: an owning repository selected for a setup, invoked through its
own installation and verification entry points.

**Build selection**: the desired components for a new machine. The repository
manifest supplies defaults; explicit choices determine the components required
for a particular run.

**Installed repository registry**: `~/.dotfiles-env.sh`, populated by the owning
repositories to record their installed locations and directory aliases. It is
an observed machine result, separate from the build selection.

**Stage**: a named step with prerequisites, an owning command, and an observable
success condition, such as completing the base update or installing Appdots.

**Run record**: local progress information used to explain or resume a run.
It is not a configuration source or a substitute for verifying current state.

Theme ownership terms remain defined in
[Hyprdots' glossary](https://github.com/ChazBeaver/hyprdots/blob/main/GLOSSARY.md).
