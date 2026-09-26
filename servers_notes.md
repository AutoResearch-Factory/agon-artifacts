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
