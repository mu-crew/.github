# mu-crew

**Run a crew of [pi](https://github.com/earendil-works/pi) agents
in [tmux](https://github.com/tmux/tmux). See which one needs you, on any
machine. Keep long jobs off your SSH session.**

mu plans the work and hands it out. murmur shows which agent, on which host, is
waiting on you, and jumps you there. mule runs the long build on a remote host
so it doesn't hold your SSH session.

They stay out of the model's way: the tools coordinate, the model decides.
There are no daemons and no complicated setup. Each tool stands alone, and
together they compose.

| Tool | Job | Install |
| --- | --- | --- |
| [mu](https://github.com/mu-crew/mu) | Coordinate a crew of agents: task DAG, per-agent VCS workspaces, audit log | `npm i -g @mu-crew/mu` |
| [murmur](https://github.com/mu-crew/murmur) | See every agent on every machine, and jump to the one that needs you | `npm i -g @mu-crew/murmur` |
| [mule](https://github.com/mu-crew/mule) | Run long remote jobs without holding an SSH session | `npm i -g @mu-crew/mule` |

mule is a Rust binary; the npm package ships it prebuilt for Linux (x64,
arm64) and macOS (arm64). `cargo install mule-cli` also works.

Each tool works alone. Start with mu for parallel agents on one machine. Add
murmur when agents run on more than one machine. Use mule when a remote host
caps SSH sessions (`MaxSessions 1`) or a command must survive a dropped
connection.

## How they fit

The tools share no code and no database. They integrate through environment
variables:

- mu sets `MU_MANAGED_AGENT`, `MU_AGENT_NAME` and `MU_WORKSTREAM` in every
  pane it spawns. murmur marks those agents as crew and hides them unless they
  are blocked or crashed.
- mule runs each job with `MU_AGENT_NAME=mule-<id>` and forwards the caller's
  `MU_WORKSTREAM`. A remote TUI job shows up in `murmur pick` as crew.

## Built on tmux and pi

tmux is the runtime: a tmux server plus a pane id is an agent's address. pi is
the agent: mu drives it and murmur reports from inside it as an extension. We
build nothing tmux already does, and we extend pi without forking it.

Support reaches further in places: mu also spawns into herdr panes, murmur
takes attention hooks from Codex, Cursor and opencode, and mule runs any
command. Each has a stated limit; see [STANCE.md](https://github.com/mu-crew/.github/blob/main/STANCE.md).

## What none of them do

No hosted service, no daemon, no agent-to-agent chat. None of them picks a
model or provider. State is SQLite and files on each machine.

The design rules are in [ZEN.md](https://github.com/mu-crew/.github/blob/main/ZEN.md).

The tools are published as-is. Issues are welcome; open one before a pull
request.
