# Bun Experiment: `murmur` & `mu`

**Date:** 2026-11-27  
**Environment:** macOS arm64, Node `v26.13.1`, Bun `1.4.5`  
**Targets:** `@mu-crew/murmur` `3.0.1`, `@mu-crew/mu` `5.6.0`

## Decision

**Stay all-in on Node.**

Switching to Bun is an all-in-or-nothing move (straddling both runtimes adds double-driver and test-harness maintenance), and the measured CLI wins are too small to justify it:
- **`mu` CLI:** no win (0.92x–1.08x vs Node).
- **`murmur` CLI:** ~15–20 ms faster (1.20x–1.68x), with ~35 ms of `status`/`export` spent shelling out to `tmux` regardless of JS runtime.
- **Dev tooling:** `bun check` (66 ms vs `tsc`/TS 7 native 399 ms) and `bun build` (31 ms vs `tsup` 802 ms) save ~1 s combined, which is a rounding error next to the 13 s+ test suite.

Existing Node optimizations (`tsup` code-splitting keeping `ink` lazy, V8 compile cache, Node 26, TS 7 native, Biome) already captured almost all of the practical latency gains.

---

## Benchmarks

> **Measurement note:** Invoking `bun` through `/Users/mtrojer/.local/share/mise/shims/bun` added ~40 ms per call (making Bun appear 0.57x–0.81x slower than direct `node`). All numbers below use the direct binaries (`.../mise/installs/node/26/bin/node` and `.../mise/installs/bun/1.4.5/bin/bun`).

### 1. `murmur` CLI latency (median of 25 runs, 3 warmups)

| Command | Node `dist/cli.js` (cached) | Bun `dist/cli.js` | Bun `src/cli.ts` (raw TS) | Bun `dist` vs Node |
| --- | ---: | ---: | ---: | ---: |
| `--version` | 39.2 ms | 25.2 ms | 29.3 ms | 1.55x (-14.0 ms) |
| `peer list` | 47.4 ms | 28.3 ms | 33.2 ms | 1.68x (-19.1 ms) |
| `status --cached` | 78.3 ms | 65.2 ms | 66.1 ms | 1.20x (-13.1 ms) |
| `status --json --cached` | 77.6 ms | 61.8 ms | 65.2 ms | 1.25x (-15.8 ms) |
| `status` (live tmux) | 78.2 ms | 62.7 ms | 65.8 ms | 1.25x (-15.5 ms) |
| `export` (live tmux) | 83.0 ms | 62.4 ms | 66.9 ms | 1.33x (-20.6 ms) |

### 2. `mu` CLI latency (median of 20 runs, 3 warmups)

| Command | Node `dist/cli.js` | Bun `dist/cli.js` | Bun vs Node |
| --- | ---: | ---: | ---: |
| `mu --version` | 43.8 ms | 43.4 ms | 1.01x (-0.4 ms) |
| `mu --help` | 47.2 ms | 43.6 ms | 1.08x (-3.6 ms) |
| `mu sql "SELECT count(*) FROM tasks"` | 58.4 ms | 57.0 ms | 1.02x (-1.4 ms) |
| `mu host list` | 61.9 ms | 58.8 ms | 1.05x (-3.1 ms) |
| `mu state --all --json` | 114.4 ms | 119.7 ms | 0.96x (+5.3 ms) |
| `mu state --all` | 108.4 ms | 117.3 ms | 0.92x (+8.8 ms) |

### 3. `murmur dash` TUI cold start to first frame (10 runs in PTY)

| Runtime | Median | Min |
| --- | ---: | ---: |
| Node `dist/cli.js dash` | 337.0 ms | 329.0 ms |
| Bun `dist/cli.js dash` | 318.7 ms | 292.2 ms |

Renders (~15.4 KB initial frame) and exits cleanly on `q` in both runtimes.

### 4. `murmur` in-process SQLite (`openStore` on temp DB, median of 3 runs)

| Operation | Node (`better-sqlite3`) | Bun (`bun:sqlite`) | Comparison |
| --- | ---: | ---: | ---: |
| `openStore()` (init schema) | 6.5 ms | 9.7 ms | Bun 1.5x slower |
| 5,000 × `store.peers()` reads | 65.2 ms | 53.4 ms | Bun 1.22x faster |
| 500 × `store.syncLocal()` (12 panes) | 43.8 ms | 85.6 ms | Bun 1.95x slower |

### 5. `murmur` toolchain (typecheck, build, format, tests)

| Role | Current | Current Time | Bun 1.4.5 | Bun Time | Notes |
| --- | --- | ---: | --- | ---: | --- |
| Typecheck | `tsc --noEmit` (`@typescript/native` TS 7) | 399 ms | `bun check --no-script` | 66 ms (44 ms check) | Full TS check, 6x faster; `--no-script` needed because `package.json` defines `"check"`. |
| Build | `tsup` (esbuild + DTS) | 802 ms (27 ms JS + 494 ms DTS) | `bun build --splitting --target node --packages=external` | 31 ms (JS only) | Doesn't emit `dist/index.d.ts`. |
| Format / Lint | `biome format` / `biome check` | 67 ms / 100 ms | `bun fmt --check` | 50 ms | Format only (Prettier-style, flagged 134/144 files vs `biome.json`); no linter. |
| Tests (Vitest) | `vitest run` (Node) | 13.4 s | `bun --bun vitest run` | 15.5 s | 64/69 files, 895/905 passed (10 failed). |
| Tests (native) | `vitest run` (Node) | 13.4 s | `bun test --preload ./test/setup.ts` | 11.7 s | 707 pass / 145 fail; real tmux spawns dominate wall time. |

---

## Compatibility Notes (`murmur` under Bun 1.4.5)

- `murmur/src/store.ts` already switches to `bun:sqlite` under Bun; CLI output (`status --cached`, `status --json --cached`, `peer list`) is byte-for-byte identical to Node.
- **2 real `bun:sqlite` edge cases** surfaced by `vitest` on Bun:
  1. `test/dash-store.test.ts`: `SQLiteError: disk I/O error` when reading an open handle after `state.db` / `-wal` / `-shm` are unlinked during schema rebuild (`better-sqlite3` keeps reading the unlinked inode).
  2. `test/store-concurrency.test.ts`: `SQLITE_BUSY` (`database is locked`) on `PRAGMA` inside `openStore()` during concurrent upgrade bursts.
- **Test-harness Node assumptions** (not `murmur` runtime bugs):
  - `test/dash-register.test.ts` & `test/sidepanel-register.test.ts`: rely on `process.moduleLoadList` and `--experimental-strip-types`.
  - `test/legacy-state.test.ts`: mutates `process.env.HOME` in-process, which Bun's `os.homedir()` does not pick up.
  - `bun test`: lacks `vi.stubEnv` / `vi.unstubAllEnvs` and runs files in one process instead of `vitest`'s `pool: "forks"` + serial `tmux-rig` project.
- **`bun build --compile`**: bundles to an 86 MB binary in ~0.6 s, but fails at runtime because `ink/build/devtools.js` eagerly imports optional `react-devtools-core`.
