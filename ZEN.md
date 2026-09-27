# Zen of mu-stack

Six rules for designing and reviewing the tools. Cite them by number in a
review: "this breaks #1".

1. **Report state, don't scrape it.**
   State comes from inside the agent or from a typed CLI verb. It is never
   parsed out of a terminal. murmur stores nothing read from a screen, and
   every mu read path that matters has `--json`.

2. **Absence means absence; fail loudly.**
   Show the current snapshot and never fill a gap with a guess. A failure must
   not look like success: mule exits 3 when a human must open the SSH master,
   124 on timeout and 137 on kill, and an orphaned job shows `-` for runtime
   because its runtime is unknown.

3. **Local-first, no daemon.**
   Each machine keeps its own state in SQLite or plain files. Peers pull
   snapshots over ssh. Nothing runs in the background to keep the stack
   working.

4. **Small binaries, thin contract.**
   Each tool works alone. They integrate only through environment variables:
   `MU_MANAGED_AGENT`, `MU_AGENT_NAME`, `MU_WORKSTREAM`. A new integration
   extends that contract before it adds a dependency between tools.

5. **Write down the no.**
   Every README says what the tool is not. A rejected flag or feature gets its
   reason recorded. Known gaps are listed publicly, not left for users to find.

6. **Agents read the docs too.**
   Much of the stack is driven by agents. Print a `next:` hint after a state
   change, give each failure a distinct exit code, and write instructions an
   automated caller can follow: "exit 3: stop and ask the operator".
