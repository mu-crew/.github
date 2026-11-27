# How the tools fit

The tools share no code and no database. Each works alone, and they meet only
in environment variables, tmux options, CLI output, and — for mule — mu's
plugin protocol (`mu-plugin-mule`).

## Pick by need

- **Capped SSH or jobs that must survive drops:** [mule](https://github.com/mu-crew/mule).
- **Local agent, remote codebase:** pi with mule's extension (`pi --mule host:tree`).
- **Several agents, need to see who waits on you:** add [murmur](https://github.com/mu-crew/murmur) for the cross-machine view, tmux badges, and jump.
- **Plan work across a crew:** add [mu](https://github.com/mu-crew/mu), the center of the crew. It hands work to agents of all five harnesses, drives pi through a control socket and reads the other four's native OSC 7501 status directly from vtmux or Rex (falling back to murmur 2.x hooks on vanilla tmux). `mu agent spawn --mule` sets up a remote workspace; `--host` runs a remote loop over a session, a lease or another plugin job ([PLACEMENT.md](PLACEMENT.md)).
- **Builds competing on one machine:** add [muzzle](https://github.com/mu-crew/muzzle).
- **Sessions worth keeping and searching:** add [museum](https://github.com/mu-crew/museum).
- **One popup to switch sessions and jump to agents:** add [tsesh](https://github.com/mu-crew/tmux-session-picker).

## Composition contracts

- **Crew identity:** mu sets `@mu_agent`, `@mu_workstream`, and `@mu_role` pane options (plus `MU_*` env vars) on every pane it spawns. murmur reads those pane options directly in its `list-panes` pass and marks them as crew.
- **Agent state (OSC 7501 + 7577):** pi, Claude Code, and Codex emit OSC 7501 natively (Cursor via a thin TTY bridge); vtmux (`>= 3.8-rc3`) and Rex keep per-pane state and resume records. mu reads `#{pane_program_*}` directly via `paneStatus()` (pi also keeps its `MU_CTL_SOCK` control socket for exact turn counts, abort, and steering), and murmur 3.0 reads local panes in a single pass with no pi extension and no harness hooks.
- **Rex bridge (`murmur rex-feed`):** murmur reads Rex panes directly (one `rex do` per collect; local and remote via `peer --rex-host`), but Rex.app only shows chrome for programs that report to Rex itself. `murmur rex-feed` forwards murmur's view back into Rex tabs as OSC 7501 records. You need it for local tmux-in-Rex tabs and remote vtmux peers (tmux keeps the record, so Rex.app sees nothing), for `mu` attention filtering (only human agents and `blocked`/`error`/`crashed` crew workers, not every `done` worker), and for the `crashed` rollup. `--exclude-rex` bridges only tmux panes in hybrid setups. A pure-Rex fleet connected in Rex.app does not need it.
- **Remote jobs & workspaces:** mule runs each job as `mule-<id>`, so remote jobs appear in murmur too. mule's pi extension keeps pi's loop local while its workspace tools run on the host over mule's single master connection (`mu agent spawn --mule` starts it for you).
- **Placement:** an agent's loop, workspace, and shell each run locally or on a host; mu's DB always stays local. Every hop to a host goes through your link (a stored template or a registered plugin such as `mu-plugin-mule`), never mu's own SSH.
- **Session archive:** museum copies each machine's pi, Claude Code, Codex, and cursor-agent session files into one store over plain rsync, started by a pi extension, a tmux status line, or a timer.
- **Admission:** muzzle needs nothing from mu, but respects what mu sets. A build you start in your own shell goes ahead of builds queued by mu's workers, and `muzzle status` shows which agent holds each slot.
- **UI & switching:** [dotfiles](https://github.com/mu-crew/dotfiles) reads vtmux's native `#{window_program_state}` and `#{pane_program_state}` directly (zero forks, native focus clearing); murmur 3.0 publishes only global `@murmur_count_*` for the multi-machine pill. tsesh reads `#{session_program_state}` natively for local sessions and `murmur status --json --cached` + `murmur jump` for multi-machine agents.
