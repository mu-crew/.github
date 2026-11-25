# The ladder

Where each part of an agent runs, and which tool covers it. Each tool's docs
cover only its own part; mu's [remote ladder guide](https://github.com/mu-crew/mu/blob/main/docs/guide/remote-ladder.md)
has the verbs, flags and refusals in full. [STANCE.md](STANCE.md) lists the
limits; [ZEN.md](ZEN.md) the rules.

## Four places

An agent has four places:

- **loop**: the harness process, with its model calls, credentials and
  control socket;
- **workspace**: the worktree it edits;
- **shell**: where bash, git and make run;
- **DB**: the `mu.db` that holds its row, tasks and recovery.

**Local** means the DB's machine. The DB is always local. Placement is per
agent: one crew can mix placements, as long as every pane lives on one
multiplexer. A tmux pane is where an agent is reached; mu's row is who it is.

## Rungs

A rung names one combination of the four places. The numbers are names, not
a ranking: climb only as far as the host makes you.

| Rung | What it is | Loop | Workspace, shell | Tools |
| --- | --- | --- | --- | --- |
| 0 | a local worker | local | local | mu; murmur, muzzle |
| 1 | one command on a host, no agent (`mu host exec`) | none | host | mu, through the host's link (mule's, or your template) |
| 2 | local pi, remote tree (`pi --mule host:tree`, or `mu agent spawn --mule`) | local | host | pi with mule's pi extension; mu to manage it |
| 3 | a session pane on the host | host | host | mu, with a session template or the ssh example plugin |
| 4 | a lease: a job that outlives its connection | host | host | mu with mule's plugin (the `lease` job kind) |
| 5 | a provider host, made on demand | host | host | mu with a provider plugin |

## What each rung supports

| Rung | Harnesses | Multiplexer (local pane) | One SSH session per connection |
| --- | --- | --- | --- |
| 0 | all five | tmux, Rex, herdr | n/a |
| 1 | none: no agent | any (no pane) | fine over a link that serialises itself, as mule's does |
| 2 | pi only | tmux or Rex; herdr refused | fine: mule admits each op on its one master |
| 3 | pi, plus remote hook events for the other four on tmux | tmux; Rex for session templates; herdr refused | fine: each pane opens its own connection |
| 4 | pi only: a lease carrier forwards no stdin | tmux | fine: one master |
| 5 | what the provider's plugin and host offer | tmux | the provider's |

## Reach for what

| You have | Reach for |
| --- | --- |
| a build or test to run on a host, no agent | rung 1 |
| a host that caps SSH sessions, or a job that must survive a dropped connection | mule: rung 1 through its link, rung 4 for a worker |
| a host with no credentials or model API for the loop | rung 2 |
| a host that runs the loop and takes a connection per pane | rung 3 |
| no machine yet | rung 5 |
| several agents, and you need to see who waits on you | murmur, on every rung that has a pane |
| several agents' builds on one machine | muzzle, on that machine |

## Limits

- Rung 2 is pi only. claude and codex on rung 2 are on mu's
  [roadmap](https://github.com/mu-crew/mu/blob/main/docs/ROADMAP.md) as
  "Ladder reach", not shipped.
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
  round trip per caller. Reopen only if several rung-2 agents on one capped
  host hit ticket waits twice.
- **A second admission counter.** mule admits ops on its own master; mu
  takes no slot for them.
- **A mixed-multiplexer crew.** Placement can mix within one crew; managed
  pane backends cannot.
