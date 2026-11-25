<p align="center">
  <img src="https://raw.githubusercontent.com/mu-crew/.github/main/logo/mu-crew.png" alt="mu-crew logo: a tmux window with μ and two agent panes" width="128">
</p>

<h1 align="center">mu-crew</h1>

<p align="center">
Run a crew of coding agents in <a href="https://github.com/martintrojer/vtmux">tmux</a>, each placed where it fits: the loop where your credentials are, the work where the code is, one view of who needs you.
</p>

## The tools

Each tool works alone; pick what you need.

| Tool | What it does |
| --- | --- |
| [**mu**](https://github.com/mu-crew/mu) | Plans work across a crew: task DAG, per-agent workspaces, audit log |
| [**murmur**](https://github.com/mu-crew/murmur) | Shows which agent, on which host, is waiting on you; `murmur jump` or tsesh takes you there |
| [**mule**](https://github.com/mu-crew/mule) | Runs remote builds, file edits, and your agent's tools over one SSH connection it never holds |
| [**muzzle**](https://github.com/mu-crew/muzzle) | Queues heavy commands from all your agents on each machine, sized from what they actually used |
| [**museum**](https://github.com/mu-crew/museum) | Backs up your agents' sessions from every machine to one store your agents can search |
| [**tsesh**](https://github.com/mu-crew/tmux-session-picker) | One popup picker for tmux sessions and the agents in them, on every host |
| [**dotfiles**](https://github.com/mu-crew/dotfiles) | Wires them into tmux: agent state in tabs and borders, status pill, keys |

## Where agents run

Every agent has four places: its **loop** (harness, credentials, control socket), **workspace**, **shell**, and **DB** (`mu.db`, always local). Placement is per agent across rungs 0–5 — from a local worker, to a local pi editing a remote tree (`--mule`), to leased jobs and on-demand provider hosts.

See **[LADDER.md](https://github.com/mu-crew/.github/blob/main/LADDER.md)** for the full rung table, harness/multiplexer matrix, and limits.

## Quick install

```sh
npm i -g @mu-crew/mu @mu-crew/murmur @mu-crew/mule @mu-crew/muzzle @mu-crew/museum
```

Full `init` commands, `pi install` extensions, TPM config for `tsesh`, and updates: **[INSTALL.md](https://github.com/mu-crew/.github/blob/main/INSTALL.md)**.

## Docs

| Doc | Covers |
| --- | --- |
| [**LADDER.md**](https://github.com/mu-crew/.github/blob/main/LADDER.md) | The four places, rungs 0–5, capability matrix, and limits |
| [**ARCHITECTURE.md**](https://github.com/mu-crew/.github/blob/main/ARCHITECTURE.md) | Adoption by need and how the tools compose without shared code or DB |
| [**STANCE.md**](https://github.com/mu-crew/.github/blob/main/STANCE.md) | What we promise, harness/multiplexer limits, and what we do not build |
| [**ZEN.md**](https://github.com/mu-crew/.github/blob/main/ZEN.md) | Design rules |

## Built by agents, for agents

AI coding agents wrote most of this code, running on this stack, with a human reviewing what ships. The CLIs print next steps and use distinct exit codes so an agent can drive them too. Published as-is; open an issue before a pull request.
