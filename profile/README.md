<p align="center">
  <img src="https://raw.githubusercontent.com/mu-crew/.github/main/logo/mu-crew.png" alt="mu-crew logo: a tmux window with μ and two agent panes" width="128">
</p>

<h1 align="center">mu-crew</h1>

<p align="center">
Run a crew of <a href="https://github.com/earendil-works/pi">pi</a> agents in <a href="https://github.com/tmux/tmux">tmux</a>.<br>
See which one needs you, on any machine.<br>
Keep long jobs off your SSH session.<br>
Keep every session they ever ran.
</p>

## The tools

| Tool | What it does |
| --- | --- |
| [**mu**](https://github.com/mu-crew/mu) | Plans the work and hands it out: task DAG, per-agent workspaces, audit log |
| [**murmur**](https://github.com/mu-crew/murmur) | Shows which agent, on which host, is waiting on you, and jumps you there |
| [**mule**](https://github.com/mu-crew/mule) | Runs the long build on a remote host without holding your SSH session |
| [**museum**](https://github.com/mu-crew/museum) | Backs up every pi session from every machine to one store your agents can search |
| [**tsesh**](https://github.com/mu-crew/tmux-session-picker) | Picks or creates a tmux session, with each session's agent state |
| [**dotfiles**](https://github.com/mu-crew/dotfiles) | Wires them into tmux: agent state in tabs and borders, a status pill, keys |

```sh
npm i -g \
  @mu-crew/mu \
  @mu-crew/murmur \
  @mu-crew/mule
```

Each tool works alone. Start with mu for parallel agents on one machine. Add
murmur when agents run on more than one machine. Add mule when a remote host
caps SSH sessions or a command must survive a dropped connection. Add museum
when your sessions are worth keeping.

## How they fit

The tools share no code and no database. They meet in environment variables
and tmux options:

- mu sets `MU_AGENT_NAME` and `MU_WORKSTREAM` in every pane it spawns. murmur
  marks those agents as crew and shows them only when they need you.
- mule runs each job as `mule-<id>`, so a remote job shows up in murmur too.
- museum copies each machine's pi session files into its own folder in one
  store, over plain rsync. Agents read the store with a skill.
- murmur publishes agent state as `@murmur_*` tmux options. The dotfiles and
  tsesh read them, and so can anything else.

## What we believe

- **Stay out of the model's way.** The tools coordinate; the model decides.
- **No daemons, no complicated setup.** State is SQLite and files on each machine.
- **Standalone tools that compose.** Each is useful alone.

Built on tmux and pi, with narrower support for herdr, Codex, Cursor and
opencode. The full stance is in [STANCE.md](https://github.com/mu-crew/.github/blob/main/STANCE.md)
and the design rules in [ZEN.md](https://github.com/mu-crew/.github/blob/main/ZEN.md).

## Built by agents, for agents

AI coding agents wrote most of this code, running on this stack, with a human
reviewing what ships. The CLIs print next steps and use distinct exit codes so
an agent can drive them too.

Published as-is. Issues are welcome; open one before a pull request.
