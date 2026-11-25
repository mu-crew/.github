# Zen of mu-crew

Seven rules for designing and reviewing the tools. Cite them by number in a
review: "this breaks #1".

1. **Get out of the model's way.**
   The tools persist state and coordinate handoffs; the model decides what to
   do. None of them picks a model, provider or thinking effort, and none of
   them rewrites what an agent says or does. mu launches whatever
   `$MU_<KEY>_COMMAND` names, so your shell rc owns the agent command.

2. **Report state, don't scrape it.**
   State comes from inside the agent or from a typed CLI verb. It is never
   parsed out of a terminal. murmur stores nothing read from a screen, and
   every mu read path that matters has `--json`. A mirror (a tmux option, a
   cached peer snapshot) is for display and is never the state; it carries
   its age.

3. **Absence means absence; fail loudly.**
   Show the current snapshot and never fill a gap with a guess. A failure must
   not look like success: mule exits 3 when a human must open the SSH master,
   124 on timeout and 137 on kill, and an orphaned job shows `-` for runtime
   because its runtime is unknown. A remote fact carries its age, and one
   that cannot be read is unknown, never absent.

4. **No daemons, no complicated setup.**
   Install one package, run one init command, and the tool works. Nothing runs
   in the background to keep it working. A process that lives exactly as long
   as its caller (pi's control socket, a `mule serve --stdio` child, a lease
   carrier in its pane) is not a daemon; one that is shared between callers or
   outlives them is. Each machine keeps its own state in
   SQLite or plain files, and peers pull snapshots over ssh when asked.

5. **Standalone tools that compose.**
   Each tool is useful alone, and each does one job. They share no code and no
   database; they compose through environment variables (`MU_MANAGED_AGENT`,
   `MU_AGENT_NAME`, `MU_WORKSTREAM`), tmux options, CLI output and, for
   mule, mu's plugin protocol. A new integration extends
   that contract before it adds a dependency between tools.

6. **Write down the no.**
   Every README says what the tool is not. A rejected flag or feature gets its
   reason recorded. Known gaps are listed publicly, not left for users to find.

7. **Agents read the docs too.**
   Much of the stack is driven by agents. Print a `next:` hint after a state
   change, give each failure a distinct exit code, and write instructions an
   automated caller can follow: "exit 3: stop and ask the operator".
