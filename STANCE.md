# Stance

mu-crew is tools for running coding agents: mu, murmur, mule, muzzle and
museum. This page says what they are built on, what they support beyond that,
and what the org promises. [ZEN.md](ZEN.md) has the design rules, and
[PLACEMENT.md](PLACEMENT.md) says where each part of an agent can run.

## Built on tmux, for five agent CLIs

**tmux (specifically [vtmux](https://github.com/martintrojer/vtmux) >= 3.8-rc3) is the runtime.** Agents live in tmux
panes. A pane is where an agent is reached; mu's row (workstream and name) is
who it is, and placement says where its loop, workspace and shell run. We recommend
vtmux because it is drop-in tmux 3.8 plus open OSC 7501 (program status) and
OSC 7577 (program resume) support — eliminating harness state hooks, murmur's
pi extension, and focus-clear shell-outs. On vanilla tmux 3.8 (which drops both
sequences), pi workers in mu, mule, muzzle, and museum still work; use `murmur`
2.x (`release/2.x`) if you need local state on stock tmux. We build nothing tmux
already does: panes, sessions, attach, detach.

**Five agent CLIs are first-class.** Claude Code, Codex, Cursor's CLI
(cursor-agent), Muse Code and [pi](https://github.com/earendil-works/pi). One
crew can mix them: an orchestrator of any of the five runs workers of any of
the five, locally and on remote hosts, within the limits in the table below. Each harness's own hooks report its
state, and each tool's `init` writes its hook entries and touches only the
entries it wrote. Each launcher keeps its own sandbox, auth, model and effort;
mu only starts it, and your shell rc names it through `$MU_<KEY>_COMMAND`.

**pi has the exact channel.** mu's pi extension serves a control socket, so
send, abort, wait and the final text are exact, and pi alone delegates through
`mu_delegate`. museum reports from inside pi as an extension; murmur 3.0 needs
no pi extension because pi emits OSC 7501 natively. We extend pi through
extensions, skills and CLIs, and we never fork or vendor it. mu reaches the
other four by paste and reads their OSC 7501 status from vtmux/Rex (or murmur
2.x hooks on vanilla tmux); what that still lacks is in the table below.

One contract suite runs every mu verb against each channel, and evals run
Claude Code, Codex and pi orchestrators.

## Beyond tmux and the five

Some support reaches further, and some limits remain. Each entry below works
today, and each has a limit we state instead of hiding.

| Beyond, or short of, the five on tmux | Where | Limit |
| --- | --- | --- |
| herdr | mu spawns crews into herdr panes | murmur does not read herdr panes. herdr's own sidebar and `herdr machine` do that job. mu refuses herdr for every remote workspace and remote loop, session templates included (before any effect); `mu host exec` still runs. |
| Rex | mu spawns crews into Rex sessions and murmur 3.0 reads local Rex panes directly via OSC 7501 | Attach switches Rex.app and needs its Remote Control setting. Remote Rex panes are visible in murmur, not remotely jumpable. A non-pi CLI that reports no OSC 7501 status reads `unknown`. Remote non-pi workers are polled, with no hook receipt or hook-based close; mu refuses a remote loop over a lease or a plugin job on Rex before any effect, so they are unsupported, not merely unsmoked; a Rex session template runs. |
| Claude Code, Codex, Cursor, Muse Code | mu pastes into their panes and reads their hook events | No mid-turn steering and no write fence: a send is confirmed by the turn count, never exactly, and never resent. A long tool call looks like a hung one. After you approve a permission in the pane, it reads `blocked` until the next event. |
| Claude Code | mu interrupts with Escape | Claude reports no interrupt: it reads `idle?` until its next event. |
| Remote Claude Code, Codex, Cursor, Muse Code | mu reads the host's hook events on demand, on tmux only | A send is confirmed on the next read, not as it lands. On Rex they are polled with no hook events. |
| Remote loop over a lease (a job outliving its connection) | mu runs a pi worker through a carrier | pi only: a non-pi worker has no input path on a lease. One managed backend per workstream, and the guard is advisory: two first spawns can race. |
| Remote workspace | `mu agent spawn --mule` runs pi locally on a remote worktree (mu 5.4.0; needs mule 0.7.0 or later) | pi only (mule's pi extension); claude and codex there are on mu's ROADMAP. The local pane runs on tmux or Rex; herdr is refused. |
| ssh `pane` plugin kind | the example plugin runs a pane over plain ssh | An example, not the supported path: one connection per pane, and stop covers only the recorded process group. |
| Checks on the four | Their own subagents, recorded with `mu task check` and `mu task verdict` | `mu_delegate` children need pi. |
| Cursor | murmur, mu, muzzle, museum | No permission hook, so a waiting dialog reads as unknown. museum backs up its agent-transcripts JSONL, not the live SQLite chats under `~/.cursor/chats/`. |
| Muse Code | mu, murmur | Its Stop hook cannot hand the model a reason, so it keeps driving by mu's skill rule, not by mu's Stop hook. |
| opencode | murmur records its attention requests through `murmur notify` | Attention only: no ownership, no `running`/`idle`, no crash detection |
| Any agent command | mu `--cli <key>` launches `$MU_<KEY>_COMMAND` | Outside the five, mu cannot send to it on tmux, and its state is attention only |
| Any command | mule runs whatever you give it | TUI jobs need tmux on the remote host and an operator-opened master connection. mule's plugin serves `exec` and the `lease` job kind. |
| Claude Code, Codex, Muse Code | muzzle rewrites their Bash calls through a `PreToolUse` hook | The harness's own tool timeout still counts the wait; Claude gets `allow` only when a rule matches at the command word |
| Cursor | muzzle denies a heavy command with the exact `muzzle run` line to run instead | Cursor can't rewrite a command, so the agent reruns it |
| opencode | muzzle: an example opencode plugin | Untested |
| Containers (distrobox, podman exec) | muzzle admits and queues container gates | Admission only: no memory cap or CPU figures inside the container |
| macOS | muzzle runs there | Advisory: no memory cap or CPU weight (no systemd scope) |

A new entry needs a named limit before it goes in this table.

## What we do not build

- A terminal multiplexer or terminal runtime. tmux is ours; herdr and Rex are their own.
- An agent, a model router or a fork of any agent CLI.
- A hosted service or a daemon. State is SQLite and files on each machine.
- A remote agent runtime. mule moves commands and files over one connection;
  the agent loop runs wherever its harness runs it.
- Agent-to-agent chat. mu coordinates through a task graph and a log.
- A single merged binary. The tools stay separate and share only the `MU_*`
  environment contract and, for mule, mu's plugin protocol.

## What the org promises

The tools are published as-is. Issues are welcome. Open an issue before a pull
request, so we can agree on the change first. There is no roadmap commitment,
and a feature request can be declined with a reason.
