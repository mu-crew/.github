# How the tools fit

The tools share no code and no database. Each works alone, and they meet only
in environment variables, tmux options, CLI output, and — for mule — mu's
plugin protocol (`mu-plugin-mule`).

## Pick by need

- **Capped SSH or jobs that must survive drops:** [mule](https://github.com/mu-crew/mule).
- **Local agent, remote codebase:** pi with mule's extension (`pi --mule host:tree`).
- **Several agents, need to see who waits on you:** add [murmur](https://github.com/mu-crew/murmur) for the cross-machine view, tmux badges, and jump.
- **Plan work across a crew:** add [mu](https://github.com/mu-crew/mu), the center of the crew. It hands work to agents of all five harnesses, drives pi through a control socket and the other four by paste confirmed through murmur hooks. `mu agent spawn --mule` sets up a remote workspace; `--host` runs a remote loop over a session, a lease or a provisioned host ([PLACEMENT.md](PLACEMENT.md)).
- **Builds competing on one machine:** add [muzzle](https://github.com/mu-crew/muzzle).
- **Sessions worth keeping and searching:** add [museum](https://github.com/mu-crew/museum).
- **One popup to switch sessions and jump to agents:** add [tsesh](https://github.com/mu-crew/tmux-session-picker).

## Composition contracts

- **Crew identity:** mu sets `MU_AGENT_NAME` and `MU_WORKSTREAM` in every pane it spawns. murmur marks those agents as crew and shows them only when they need you.
- **pi control & state:** mu's pi extension serves a per-agent control socket (`MU_CTL_SOCK`); murmur's extension publishes `@murmur_*` tmux options. Both load in the same pi and do not depend on each other.
- **Other harnesses:** Claude Code, Codex, Cursor, and Muse Code run `murmur hook <harness>` from their own hooks. Each call drops one event file; murmur turns those into state, and mu reads it to confirm sends and detect turn ends.
- **Remote jobs & workspaces:** mule runs each job as `mule-<id>`, so remote jobs appear in murmur too. mule's pi extension keeps pi's loop local while its workspace tools run on the host over mule's single master connection (`mu agent spawn --mule` starts it for you).
- **Placement:** an agent's loop, workspace, and shell each run locally or on a host; mu's DB always stays local. Every hop to a host goes through your link (a stored template or a registered plugin such as `mu-plugin-mule`), never mu's own SSH.
- **Session archive:** museum copies each machine's pi, Claude Code, Codex, and cursor-agent session files into one store over plain rsync, started by a pi extension, a tmux status line, or a timer.
- **Admission:** muzzle needs nothing from mu, but respects what mu sets. A build you start in your own shell goes ahead of builds queued by mu's workers, and `muzzle status` shows which agent holds each slot.
- **UI & switching:** murmur publishes agent state as `@murmur_*` tmux options for [dotfiles](https://github.com/mu-crew/dotfiles) and anything else. tsesh reads `murmur status --json --cached` and hands jumps to `murmur jump <jump_target>`, falling back to `@murmur_*` without murmur.
