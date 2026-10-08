# Mini Project 4

Track: A

Cybersecurity Pentest

FLAG_SEED = g_giffin

## About this repo

This repo is an authorized, black-box penetration test of **Vuln Hub** (the
instructor-provided Flask target), driven by Claude Code as an agent. The pieces fit
together as a pipeline:

- **`claude-workflow.md`** — the instruction file that defines the agentic loop
  (`goal → tool calls → review`). The pentest is run from it: the agent works only
  from the six public hints and discovers each exploit in the loop.
- **`submission.txt`** — the captured results. Each `FLAG{...}` the loop produces is
  appended here, one per line, and verified with `check_flags.py`.
- **`Proof-of-Concept/`** — one folder per finding. As each flag is captured, the
  loop writes the details there (`finding.md` with the request/payload/command and a
  screenshot) so the evidence lives outside the chat.
- **Pentest report** — generated at the end from the combined findings: the
  `Proof-of-Concept/` notes are assembled into one report (scope + authorization,
  methodology, findings with severity, evidence, remediation).

```text
claude-workflow.md  ──drives──▶  the loop  ──▶  submission.txt      (flags)
                                           └──▶  Proof-of-Concept/  (per-finding detail)
                                                        └──▶  pentest report
```

## How to run / reproduce

The target, **Vuln Hub**, is instructor-provided and is **not** redistributed in this
repo. Get it from the MP4 starter zip (`track_a_vulnhub/`) on Blackboard. All flags in
this repo were produced with `FLAG_SEED=g_giffin`, so reproduce with the same seed.

**1. Start the target** (from the starter's `track_a_vulnhub/` folder), using your seed:

```bash
# Docker (recommended by the starter)
export FLAG_SEED=g_giffin
docker compose up --build
```

```bash
# Podman (Docker-compatible; what I used on Fedora)
podman build -t inf601-vulnhub .
podman run -d --name inf601-vulnhub -p 127.0.0.1:5000:5000 -e FLAG_SEED=g_giffin inf601-vulnhub
```

```bash
# Or a plain Python venv
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
FLAG_SEED=g_giffin python app.py
```

Then open <http://127.0.0.1:5000>. There should be **no** red "DEFAULT seed" banner — if
there is, the seed isn't set and the flags won't match. A public test account
(`alice / password123`) is used where a logged-in session is needed.

**2. Reproduce the findings.** Follow the loop in `claude-workflow.md`; each captured
`FLAG{...}` is appended to `submission.txt` and documented under `Proof-of-Concept/`.

**3. Verify the flags** (instructor's checker, from the starter folder):

```bash
python check_flags.py submission.txt --username g_giffin
```

It prints `VALID` / `NOT VALID` per line (it awards no points — credit requires the PoC).

## Scope & authorization

This penetration test was performed **only** against the instructor-provided Vuln Hub
target, running locally on my own machine and bound to `127.0.0.1`, as an authorized
exercise for INF601 Mini Project 4. No other systems were scanned, accessed, or tested.
All testing was self-contained on localhost. The full scope and authorization statement
is restated at the top of the pentest report.

— Gage Giffin, INF601 (Fall 2026)

## AI Usage

Below is a overview of the use of artificial intelligence through this project. 

### What I used Claude Code for

- Claude Code (Opus 4.8) - Created empty `submission.txt` and the Proof-of-Concept folder structure for each vulnerability.
- Claude Code (Opus 4.8) - Drafted the repo overview, run/reproduce, and scope/authorization sections of the README.

### What I wrote myself

Base README and organization. 

### What I changed in AI-generated code
