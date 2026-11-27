<p align="center">
  <img src="https://raw.githubusercontent.com/mu-crew/.github/main/logo/mu-crew.png" alt="mu-crew logo: a tmux window with μ and two agent panes" width="128">
</p>

<h1 align="center">mu-crew</h1>

<p align="center">
Run a crew of coding agents in <a href="https://github.com/martintrojer/vtmux">vtmux</a> or <a href="https://www.superlogical.com/rex">Rex</a>, each placed where it fits: the loop where your credentials are, the work where the code is, one view of who needs you.
</p>

## The tools

Each tool works alone; pick what you need.

| Tool | What it does |
| --- | --- |
| [**mu**](https://github.com/mu-crew/mu) | Plans work across a crew: task DAG, per-agent workspaces, audit log |
| [**murmur**](https://github.com/mu-crew/murmur) | Shows which agent, on which host and in tmux or Rex, is waiting on you; `murmur jump`, tsesh or Rex's `cmd+j` takes you there |
| [**mule**](https://github.com/mu-crew/mule) | Runs remote builds, file edits, and your agent's tools over one SSH connection it never holds |
| [**muzzle**](https://github.com/mu-crew/muzzle) | Queues heavy commands from all your agents on each machine, sized from what they actually used |
| [**museum**](https://github.com/mu-crew/museum) | Backs up your agents' sessions from every machine to one store your agents can search |
| [**tsesh**](https://github.com/mu-crew/tmux-session-picker) | One popup picker for tmux sessions and the agents in them, on every host |
| [**dotfiles**](https://github.com/mu-crew/dotfiles) | Wires them into tmux and Rex: agent state in tabs and borders, status pill, keys, the Rex agent switcher |

## Where agents run

Every agent has four places: its **loop** (harness, credentials, control socket), **workspace**, **shell**, and **DB** (`mu.db`, always local). Each agent's placement is **local**, **remote workspace** (loop here, workspace and shell on a host: `--mule`), or **remote loop** (loop, workspace and shell on a host: `--host`, over a session, a lease or another plugin job). The DB and your pane stay here.

See **[PLACEMENT.md](https://github.com/mu-crew/.github/blob/main/PLACEMENT.md)** for the axes, picking by need, the support matrix, and limits.

## Two first-class terminals: vtmux and Rex

Agents report their state and how to resume them with two open terminal protocols, **OSC 7501** (program status) and **OSC 7577** (program resume), which pi, Claude Code and Codex emit natively. Both terminals below keep those records per pane, so mu and murmur read agent state straight from the terminal: no per-harness hooks, inboxes or state extensions.

- **[vtmux](https://github.com/martintrojer/vtmux)** (`>= 3.8-rc3`, a drop-in `tmux`): set `set -s vtmux-program-status on` and `set -s vtmux-program-resume on`. Runs anywhere tmux does, including Linux servers and over ssh. Every placement works, including remote loops over a lease or a plugin job, and tsesh's picker. The default for remote and server work.
- **[Rex](https://www.superlogical.com/rex)** (a terminal with its own multiplexer: Rex.app, and Rex servers on macOS and Linux): mu spawns crews into Rex sessions; murmur reads every Rex pane in one call and jumps, prompts, stops and glances local and remote ones (`murmur peer set <peer> --rex-host <label>`); dotfiles' `cmd+j` switcher jumps to any agent, Rex or tmux, on any host; Rex restores agent sessions through OSC 7577. Limits: Rex is a preview release; lease and plugin job views are tmux-only; tsesh is tmux-only; `murmur rex-feed` bridges tmux-hosted agents into Rex.app's tabs.

On **vanilla tmux 3.8**, which drops both protocols, mu's pi workers, mule, muzzle and museum still work, without glyphs or murmur's local agent state; stay on `@mu-crew/murmur@2` (`release/2.x`) if you need local agent state on stock tmux.

## Quick install

```sh
npm i -g @mu-crew/mu @mu-crew/murmur @mu-crew/mule @mu-crew/muzzle @mu-crew/museum
```

Full `init` commands, `pi install` extensions, `vtmux` options, TPM config for `tsesh`, and updates: **[INSTALL.md](https://github.com/mu-crew/.github/blob/main/INSTALL.md)**.

## Docs

| Doc | Covers |
| --- | --- |
| [**PLACEMENT.md**](https://github.com/mu-crew/.github/blob/main/PLACEMENT.md) | The four places, the three axes, support matrix, and limits |
| [**ARCHITECTURE.md**](https://github.com/mu-crew/.github/blob/main/ARCHITECTURE.md) | Adoption by need and how the tools compose without shared code or DB |
| [**STANCE.md**](https://github.com/mu-crew/.github/blob/main/STANCE.md) | What we promise, harness/multiplexer limits, and what we do not build |
| [**ZEN.md**](https://github.com/mu-crew/.github/blob/main/ZEN.md) | Design rules |

## Built by agents, for agents

AI coding agents wrote most of this code, running on this stack, with a human reviewing what ships. The CLIs print next steps and use distinct exit codes so an agent can drive them too. Published as-is; open an issue before a pull request.
