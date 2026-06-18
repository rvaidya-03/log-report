# log-report

A Harbor (Terminal-Bench) task: parse an HTTP access log into a small JSON traffic report.

## Overview
The agent is given an access log in Common Log Format at `/app/access.log` and must
write `/app/report.json` — a single JSON object with exactly three keys:

- `total_requests` (int): number of non-empty log lines.
- `unique_ips` (int): count of distinct client IPs (first whitespace field of each line).
- `top_path` (str): the most frequently requested path from the quoted request line.

## Layout
- `instruction.md` — the prompt handed to the agent (the only thing it sees).
- `solution/solve.sh` + `solution/solve.py` — the reference (oracle) solution.
- `environment/Dockerfile` + `environment/data/access.log` — the agent's pinned container and input.
- `tests/` — the separate-mode verifier image, `test.sh`, `test_outputs.py`, and a
  ground-truth copy of the log used to recompute the expected answer at verify time.
- `task.toml` — the manifest (artifacts, timeouts, resources, metadata).

## Verification
The verifier runs in its own container (`environment_mode = "separate"`). It recomputes
`total_requests`, `unique_ips`, and `top_path` from its own baked-in copy of the log,
then asserts the agent's `/app/report.json` is a JSON object with exactly those three
keys and matching values. It writes `reward.txt` (1/0) and `ctrf.json` to
`/logs/verifier/`. No expected values are hardcoded.

## Running
```
harbor run -p . --agent oracle   # reference solution -> reward 1.0
harbor run -p . --agent nop      # no-op agent        -> reward 0.0
```
