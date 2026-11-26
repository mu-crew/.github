# Install

## Multiplexer (recommended: vtmux >= 3.8-rc3)

We recommend installing [**vtmux**](https://github.com/martintrojer/vtmux) as your `tmux` binary and enabling its OSC 7501/7577 options in `~/.tmux.conf`:

```tmux
%if "#{vtmux_version}"
set -s vtmux-program-status on
set -s vtmux-program-resume on
%endif
```

**Why vtmux:** pi, Claude Code, and Codex report their own state over OSC 7501 natively, and vtmux exposes it directly in `#{*_program_state}` formats with in-process focus clearing — so `murmur` 3.0, `mu`, `tsesh`, and `dotfiles` need zero harness state hooks, zero murmur pi extension, and zero focus-clear shell-outs. Staying on vanilla tmux 3.8? Install `@mu-crew/murmur@2` (branch `release/2.x`) instead of murmur 3.0.

## CLI tools

Install the CLI tools you want (each works alone):

```sh
npm i -g \
  @mu-crew/mu \
  @mu-crew/murmur \
  @mu-crew/mule \
  @mu-crew/muzzle \
  @mu-crew/museum
```

Initialize each tool you installed:

```sh
murmur init                      # this node's identity (no harness hooks needed on vtmux/Rex)
mu init                          # mu's Stop hook entry and skill per harness
muzzle init                      # muzzle's hook entries: queues heavy shell calls
museum init devbox:/data/museum  # museum's store, hook entries, skill and sync timer
mule init                        # seeds config and registers mule's plugin with mu
```

Each `init` writes entries for every harness set up on the machine (Claude
Code, Codex, Cursor, Muse Code) and touches only its own. `museum init` takes
the store (`host:/path`, or a local `/path`).

## pi extensions

If you use [pi](https://github.com/earendil-works/pi), install the matching
extensions:

```sh
pi install npm:@mu-crew/mu       # mu's control socket, delegate tool, skill and recipes
pi install npm:@mu-crew/mule     # remote workspace tools (`pi --mule host:tree`)
pi install npm:@mu-crew/muzzle   # queues heavy shell calls and skill
pi install npm:@mu-crew/museum   # session backup and search skill
```

## tsesh (tmux plugin)

With [TPM](https://github.com/tmux-plugins/tpm), add to `~/.tmux.conf`:

```tmux
set -g @plugin 'mu-crew/tmux-session-picker'
```

## Updating

```sh
npm update -g @mu-crew/mu @mu-crew/murmur @mu-crew/mule @mu-crew/muzzle @mu-crew/museum
# rerun the inits you use, then:
pi update --extensions
```
