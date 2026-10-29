# Stance

mu-crew is tools for running coding agents: mu, murmur, mule and muzzle. This
page says what they are built on, what they support beyond that, and what the
org promises. [ZEN.md](ZEN.md) has the design rules.

## Built on tmux and pi

**tmux is the runtime.** Agents live in [tmux](https://github.com/tmux/tmux)
panes, and a tmux server plus a pane id is an agent's address. mu spawns crews
into tmux sessions, murmur jumps to panes, and mule runs TUI jobs under a
private tmux server. We build nothing tmux already does: panes, sessions,
attach, detach.

**pi is the agent.** mu drives
[pi](https://github.com/earendil-works/pi), and murmur reports
from inside it as an extension (`pi install npm:@mu-crew/murmur`). We extend pi through
extensions, skills and CLIs, and we never fork or vendor it. pi picks its own
model, provider and effort; so does your shell rc through
`$MU_<KEY>_COMMAND`.

Every feature ships on tmux and pi first, and that path is the one we test.

## Beyond tmux and pi

Some support reaches further. Each entry below works today, and each has a
limit we state instead of hiding.

| Beyond tmux and pi | Where | Limit |
| --- | --- | --- |
| herdr | mu spawns crews into herdr panes | murmur does not read herdr panes. herdr's own sidebar and `herdr machine` do that job. |
| Codex, Cursor, opencode | murmur records their attention requests through `murmur notify` hooks | Attention only: no ownership, no `running`/`idle`, no crash detection |
| Any agent command | mu `--cli <key>` launches `$MU_<KEY>_COMMAND` | mu's skill and prompts are written for pi |
| Any command | mule runs whatever you give it | TUI jobs need tmux on the remote host |
| Claude Code, Codex | muzzle rewrites their Bash calls through a `PreToolUse` hook | The harness's own tool timeout still counts the wait; Claude gets `allow` only when a rule matches at the command word |
| opencode, Cursor | muzzle: an example opencode plugin; Cursor can't rewrite a command | opencode's plugin is untested; on Cursor, wrap by hand |
| Containers (distrobox, podman exec) | muzzle admits and queues container gates | Admission only: no memory cap or CPU figures inside the container |
| macOS | muzzle runs there | Advisory: no memory cap or CPU weight (no systemd scope) |

A new entry needs a named limit before it goes in this table.

## What we do not build

- A terminal multiplexer or terminal runtime. tmux is ours; herdr is its own.
- An agent, a model router or a fork of pi.
- A hosted service or a daemon. State is SQLite and files on each machine.
- Agent-to-agent chat. mu coordinates through a task graph and a log.
- A single merged binary. The tools stay separate and share only the `MU_*`
  environment contract.

## What the org promises

The tools are published as-is. Issues are welcome. Open an issue before a pull
request, so we can agree on the change first. There is no roadmap commitment,
and a feature request can be declined with a reason.
