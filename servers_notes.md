# Servers notes

This file complements `servers_manual.md`.

- `servers_manual.md` should describe stable facts: hardware, access rules, storage layout, scheduler rules, shell defaults, and common environment/cache conventions.
- This file should record short project-local notes that are not stable enough for the manual yet, or pitfalls that repeatedly affect multiple projects.

Do not put credentials, private hostnames, private usernames, internal paths, or full incident logs here. If a note only matters to one workspace, put it in that workspace's `STATE.md`, `experiment-log.md`, or local runbook instead.

## Entry format

Prepend new entries under the relevant section, newest first.

```md
#### YYYY-MM-DD - Short title
[3-5 lines: reusable symptom / likely cause / fix. Prefer a command or checklist that another user can copy after filling in their own paths.]
```

Stale entries can be marked with `~~strikethrough~~`. Delete them only after they are clearly obsolete.

---

## Global notes

### Configuration

#### YYYY-MM-DD - Title
Template entry.

### Pitfalls

#### 2026-09-26 - `ssh host bash -s -- a b ""` silently loses or re-splits your last arguments
Symptom: a remote launcher built by `ssh host 'bash -s' -- "$@" <<'REMOTE'` dies on `$8: unbound variable` under `set -u`, or receives only the first word of an argument (`--stage represent` arrives as `--stage`, and the job exits 2 with an argparse usage message). Cause: ssh does not pass an argv vector; it joins everything into one string that the *remote* shell re-splits, so a trailing empty argument vanishes and any argument containing a space becomes two. Fix: never pass empty or space-containing positionals - send a sentinel (`NONE`) and translate it back on the remote side, use single-token flag forms (`--stage=represent`), or pass values as `KEY=VAL` env assignments before the command. Check right after launch: `tail -5 results/<run>/train.log` and `cat results/<run>/EXIT_CODE` - an arg-parse failure shows up within seconds, not at the end. Related bash trap in the same script: `${1:?usage ... {a|b} ...}` ends the parameter expansion at the first inner `}`, so the remainder of your message runs as a command; keep `{}` out of `:?` messages.

#### 2026-09-26 - Do not `git commit --amend` in a worktree other agents are committing to
Symptom: you amend "your" small fix and the commit you rewrote turns out to belong to someone else who committed in the seconds between your commit and your amend; their hash changes and your file is folded into their message. Cause: `--amend` targets whatever HEAD is *now*, not the commit you made. Fix: in any shared checkout, make a new commit instead - the "prefer amend for small changes" habit assumes a single writer. Recovery: do not rewrite again. `git diff <old-hash> <new-hash>` to prove no content was lost, note the hash substitution where the other agent will read it, and leave the old object reachable via the reflog.

#### 2026-09-26 - The FIRST CUDA replay in a process differs in the last bits from every later one
Symptom: you replay the same optimizer segment three ways in one process to prove two drivers agree; whichever mode runs *first* ends on different parameters (we saw max|dw| 1.8e-3 to 3.1e-3 after 400 L-BFGS iterations in float64), and reordering the modes moves the discrepancy to whatever is now first. Cause: the first matmul in a fresh CUDA context picks different cuBLAS workspace/algorithm than later calls, so a last-bit difference enters step 1; a chaotic iteration then amplifies it. Fix: before the comparison, run a throwaway warm-up of the same shapes, or put a sacrificial replay at position 1 and ignore it; always report the *first iteration at which parameter hashes differ*, not only the final distance. Check: repeat one mode at two positions in the run order - if the two copies of the same mode disagree, it is position, not mode.

#### 2026-09-26 - Optional external time binary absent on RHEL
A launcher using `/usr/bin/time` may exit 127 before Python starts. Use bash's built-in `time` with `TIMEFORMAT='real %R user %U sys %S'`, redirect its stderr separately, and retain `PIPESTATUS[0]` when piping to tee. Sample Python PIDs for RSS; bash time includes child CPU but does not report memory.
#### 2026-09-26 - Frozen numerical coordinates versus cross-CPU libm
Re-running a trig-based coordinate generator on another CPU/libm may change a few final bits even with identical seeds. Once an asset is canonical, transfer its bytes and verify its receipt hashes; do not regenerate it on load, overwrite it, or relax the hash check. Record cross-platform regeneration as a separate diagnostic.

#### 2026-09-25 - `screen -dmS <s> bash launcher.sh` keeps NO log unless the launcher itself redirects
Symptom: the job runs to completion (`DONE` says `EXIT_0`, every result file is there) but `results/<run>/train.log` does not exist, and by the time you look the screen session is already gone — `screen -S <s> -X hardcopy` answers `No screen session found`, so the stdout is unrecoverable. Cause: `screen -dmS` exits with its command, taking the scrollback with it; a launcher that runs `timeout N python train.py` with no redirect writes only to that dead pty. This bites hardest on short jobs, where the session is gone before you check. Fix: redirect *inside* the launcher — `python train.py ... > results/<run>/train.log 2>&1` — not via `tee` (see the SIGTTOU note below) and not via `screen -L` (its `screenlog.N` lands wherever screen's cwd was). Check right after launch: `test -s results/<run>/train.log || echo "NO LOG - fix the launcher before it finishes"`. Recovery is only cheap if the job is deterministic and short: re-running from scratch gave a byte-identical `solves.jsonl`, which then doubled as a determinism receipt — but that is luck, not a plan.

#### 2026-09-25 - A torch/OMP *thread* is not a physical core, and the ratio differs between two machines in the same rack
Symptom: a cost ledger billed at `$/physical-core-hour` is silently 2x too high for some runs and right for others, with no way to tell which from the receipts. Cause: receipts record `torch_threads` / worker count, and whoever computes dollars assumes one thread per core — but SMT is a per-machine BIOS setting. Two otherwise-identical hosts in our pool disagree: one reports `thread_siblings_list = 0,56` (2 hardware threads per core, 112 logical = 56 physical), the other reports `0` (SMT off, 128 logical = 128 physical). Fix: record `cat /sys/devices/system/cpu/cpu0/topology/thread_siblings_list` per host in the run receipt and divide threads by its length; also record `getrusage(RUSAGE_SELF).ru_maxrss` so the RAM term is a measurement rather than a guess. Check: `nproc` alone cannot tell you — `lscpu | grep -i 'thread.*core'` or the sysfs file can. A hard-coded `threads / 2` helper (we had one) is wrong on exactly half the pool.

#### 2026-09-25 - `rsync --delete-excluded` DELETES your excludes at the destination (remote `.venv` and `results/` wiped)
Symptom: a routine code push, `rsync -az --delete-excluded --exclude='.venv' --exclude='results' ./ host:/path/`, reports a normal transfer; minutes later `host:/path/.venv/bin/python` is `No such file or directory` and `results/` is gone. Cause: the exclude list is the list of things you do NOT want to send; `--delete-excluded` then means "and delete those at the destination too" (and it implies `--delete`). The excludes we all copy around — `.venv`, `.git`, `results`, `checkpoints`, `__pycache__` — are exactly the things that only exist remotely, so the flag deletes precisely what must survive a code push. Fix: for a code push use plain `--delete` (prunes only files that *were* under a synced path) or no delete flag at all; reserve `--delete-excluded` for a mirror you genuinely want byte-identical. Check before pushing: `rsync -n --delete-excluded ... | grep '^deleting'`. Recovery on UMD is cheap if your evidence is already local — `uv sync --frozen` rebuilt a 2.5 GB venv in 1.1 s off the warm `/export/<user>/.uv_cache`, and pushing 95 MB of results back took 10 s — but only because every recorded output hash still verified locally. Verify that first: walk each `results/<run>/manifest.json`'s `outputs` and re-hash, do not assume.

#### 2026-09-25 - screen + pytest/tee writing to the pty SIGTTOU-stops the job (STAT T)
Symptom: `screen -dmS ... bash launch.sh` starts, `ps` shows python as `T` (stopped) at ~0% CPU, and no log file appears. Cause: pytest (or `tee`) writes to the screen pty; with tostop on, the kernel sends SIGTTOU and the child never runs. Fix: redirect stdout/stderr to files inside the launch script (`> results/<run>/train.log 2>&1`), do not pipe through `tee` in screen; `stty -tostop` is extra insurance. Check with `cat /proc/$pid/status | grep State`.

#### 2026-09-25 - A CPU-usage sampler that pgreps the script name samples `timeout`, not python
Symptom: a self-sampling loop like
`P=$(pgrep -u "$USER" -f "my_script.py" | head -1); ps -o pcpu=,rss= -p $P`
logs `0.0` CPU and ~1 MB RSS for a job that is clearly running. Cause: the job was started
as `timeout 1800 .venv/bin/python my_script.py ...`, and `timeout`'s own argv repeats the
whole python command line, so `pgrep -f` matches the wrapper first and `head -1` picks it
(lower pid). Fix: match the interpreter too and take the last pid, e.g.
`pgrep -f "[.]venv/bin/python .*my_script.py" | tail -1`, or `pgrep -x python` and filter.
Check with `ps -o pid,comm,args -p $P` before trusting a utilisation log.

#### 2026-09-25 - A stray module in /tmp shadows the stdlib for scripts run from /tmp
Symptom: a one-off probe script placed in `/tmp` dies on an unrelated import, e.g.
`import sympy` -> `sympy/utilities/timeutils.py: import timeit` -> `ModuleNotFoundError`
naming a module you have never heard of. Cause: Python prepends the *script's own
directory* to `sys.path`, so any leftover `/tmp/<stdlib-name>.py` from another project
shadows the real module for every script run from there. Fix: run probe and throwaway
scripts from inside the project directory, not `/tmp`; or `python -P script.py` (3.11+) /
`PYTHONSAFEPATH=1` to stop the script directory being added. Check with
`ls /tmp/*.py` before blaming the venv.

#### YYYY-MM-DD - Title
Template entry.

## Server group: <name>

### Configuration

#### YYYY-MM-DD - Title
Template entry.

### Pitfalls

#### YYYY-MM-DD - Title
Template entry.

## External APIs / hosted compute

### Provider: <name>

#### YYYY-MM-DD - Title
Template entry.
