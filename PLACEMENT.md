# Placement

Where each part of an agent runs, and which tool covers it. Each tool's docs
cover only its own part; mu's [placement guide](https://github.com/mu-crew/mu/blob/main/docs/guide/placement.md)
has the verbs, flags and refusals in full. [STANCE.md](STANCE.md) lists the
limits; [ZEN.md](ZEN.md) the rules.

## Four places

An agent has four places:

- **loop**: the harness process (pi, claude, codex, cursor-agent or muse),
  with its model calls, credentials and control socket;
- **workspace**: the worktree it edits;
- **shell**: where bash, git and make run;
- **DB**: the `mu.db` that holds its row, tasks and recovery.

**The DB is always here, and so is your pane.** Every placement opens its
pane on your multiplexer, on this machine; a remote loop puts ssh, a carrier
or a plugin view in it. A pane is where an agent is reached; mu's row is who
it is. Placement is per agent: one crew can mix placements, as long as every
pane lives on one multiplexer.

## Three axes

| Axis | Values | Meaning |
| --- | --- | --- |
| **placement**: which places are remote | local · remote workspace · remote loop | your choice |
| **transport**: how a remote loop is reached | session · lease · plugin job | support differs |
| **host**: where the machine comes from | your host · provisioned host | who makes and releases it |

- **local**: loop, workspace and shell here (`mu agent spawn`).
- **remote workspace**: loop here; workspace and shell there
  (`mu agent spawn --mule <host>`, or `pi --mule host:tree`).
- **remote loop**: the loop there too, and so the workspace and shell
  (`mu agent spawn --host <host>`): over a **session** template, a **lease**
  (a lease template or mule's plugin lease), or another **plugin job**.

Transport and host are independent. mule's lease is a plugin job on your
host, the ssh example plugin's job runs on a host you already have, and a
provider plugin's job runs on a provisioned host.

`--mule <host>` means remote workspace; `--host <host>` means remote loop,
even through mule's lease.

A **remote command** is not a placement, since no agent runs: `mu host exec
<host> -- cmd` runs one command there, through the host's link.

## Pick by need

- Stay **local** unless something forces you off.
- The code, data or hardware is over there, and you want your session, UI and
  extensions here: **remote workspace**.
- The agent must keep working when your laptop sleeps or the link drops, and
  the host runs the harness: **remote loop** over a lease, or a plugin job
  whose job outlives the drop.
- The host runs the harness and the link is stable, and losing the run on a
  drop is acceptable: **remote loop** over a session. A session is attached
  to its ssh tty.
- No suitable machine: **remote loop** on a **provisioned host**.
- Just a build or test there, no agent: **remote command**.

## What each supports

Support depends on placement and transport.

| Placement | Transport | Harnesses | Multiplexer (your pane) |
| --- | --- | --- | --- |
| local | none | all five | tmux, Rex |
| remote workspace | mule | pi only (today) | tmux, Rex |
| remote loop | session | pi; the other four on tmux via remote hook events, on Rex polled (no hook receipt or hook-based close) | tmux, Rex |
| remote loop | lease (a lease template or mule's plugin lease) | pi only: a lease carrier forwards no stdin | tmux |
| remote loop | other plugin job (on your host or a provisioned one) | what the plugin and host offer | tmux; Rex refused |
| remote command | the host's link | none: no agent | any (no pane) |

On an sshd with `MaxSessions 1`, mu's SSH session starter opens its own
connection per pane, so it needs no lease; a template that shares one master
needs a free session on it. mule runs every op over the one master you opened.

## Which tool provides what

| Tool | Provides |
| --- | --- |
| mu | every placement: spawns, tracks and recovers agents; `mu host exec` for a remote command; session templates, and plugins for plugin jobs and provisioned hosts |
| mule | the remote workspace (its pi extension), the lease (its plugin's `lease` job kind), and a link over the one SSH master per host that you open |
| murmur | who waits on you, for agents in tmux panes in any placement; it reads tmux/vtmux and Rex panes, local and remote |
| muzzle | queues several agents' heavy commands on one machine; runs on that machine |

## Limits

- The remote workspace is pi only. claude and codex there are on mu's
  [roadmap](https://github.com/mu-crew/mu/blob/main/docs/ROADMAP.md), not
  shipped.
- mule needs tmux on the job host for every job, exec included, and opens no
  SSH master itself: a human opens it, and mule exits 3 when one is missing.
- A dropped master is not a lost host. Sends fail, nothing is resent, and the
  agent's row and tree stay; reopen the master and retry.
- A remote fact carries its age. One that cannot be read is `unknown`, never
  gone.
- mu opens no SSH. Every hop to a host goes through the user's link: a stored
  template or a registered plugin.

## What we do not build

- **A remote agent runtime.** mule moves commands and files over one
  connection; the agent loop runs wherever its harness runs it. So no
  remote codemode: it would need a JS runtime on the host, which needs only
  tmux today, and its tool semantics would have to match pi's. Reopen only on
  measured chain latency, hit twice.
- **A mule daemon.** One process that holds the only session for every caller
  outlives them and needs lifecycle, discovery and auth: the machinery a
  transport already gives. `mule serve` coalesces a burst into about one
  round trip per caller. Reopen only if several remote-workspace agents on one
  capped host hit ticket waits twice.
- **A second admission counter.** mule admits ops on its own master; mu
  takes no slot for them.
- **A mixed-multiplexer crew.** Placement can mix within one crew; managed
  pane backends cannot.
